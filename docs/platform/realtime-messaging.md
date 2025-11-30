# Realtime Messaging for OpenHR

**Last updated:** November 30, 2025  
**Confidence:** 95%+ (Supabase Realtime + WebSocket patterns)

---

## Overview

Realtime messaging enables instant communication between co-founders, founding engineers, and startups once a match is made. OpenHR uses Supabase Realtime (PostgreSQL logical replication over WebSockets) to deliver messages with <1s latency while persisting data in PostgreSQL for search, analytics, and compliance.

The design supports 10K+ concurrent connections in MVP and scales horizontally. It integrates tightly with the Message Suggestion Agent and Match Scoring Agent to drive higher engagement and faster decision cycles.

---

## Architecture

### Components

- Supabase PostgreSQL with Realtime enabled on `conversations` and `messages` tables.
- Node.js/Express API Gateway for message CRUD and auth.
- Next.js frontend using Supabase JS client for WebSocket subscriptions.
- Redis for rate limiting and ephemeral presence state.
- Optional E2E encryption via client-side key management (Phase 2).

### Data Flow

1. Client authenticates via JWT from OpenHR auth.
2. Client subscribes to a conversation channel using Supabase Realtime.
3. New message is sent via REST API (`POST /v1/conversations/{id}/messages`).
4. Backend inserts into `messages` table.
5. Supabase Realtime broadcasts INSERT event to subscribed clients.
6. Clients update UI (append message, update read receipts).

---

## Database Schema

```sql
CREATE TABLE IF NOT EXISTS conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_by UUID REFERENCES profiles(id) ON DELETE SET NULL,
  title TEXT,
  type TEXT CHECK (type IN ('one_to_one', 'group', 'system')) DEFAULT 'one_to_one',
  match_id UUID REFERENCES matches(id),
  is_archived BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS conversation_participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  profile_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role TEXT CHECK (role IN ('member', 'admin')) DEFAULT 'member',
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  last_read_at TIMESTAMPTZ,
  is_muted BOOLEAN DEFAULT FALSE,
  UNIQUE (conversation_id, profile_id)
);

CREATE TABLE IF NOT EXISTS messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  sender_profile_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  content TEXT NOT NULL,
  content_type TEXT CHECK (content_type IN ('text', 'system', 'suggested', 'attachment')) DEFAULT 'text',
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ,
  is_edited BOOLEAN DEFAULT FALSE,
  is_deleted BOOLEAN DEFAULT FALSE,
  reply_to_message_id UUID REFERENCES messages(id),
  delivered_at TIMESTAMPTZ,
  read_at TIMESTAMPTZ
);

CREATE INDEX IF NOT EXISTS idx_messages_conversation_created_at
  ON messages(conversation_id, created_at DESC);

ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;
ALTER TABLE conversation_participants ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

CREATE POLICY conversation_visibility ON conversations
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM conversation_participants cp
      WHERE cp.conversation_id = conversations.id
      AND cp.profile_id = auth.uid()
    )
  );

CREATE POLICY message_visibility ON messages
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM conversation_participants cp
      WHERE cp.conversation_id = messages.conversation_id
      AND cp.profile_id = auth.uid()
    )
  );
```

---

## Backend API Design

### Endpoints

- `GET /v1/conversations` – List user conversations.
- `POST /v1/conversations` – Create new conversation (usually from a match).
- `GET /v1/conversations/{id}` – Get conversation details.
- `GET /v1/conversations/{id}/messages` – Paginated messages.
- `POST /v1/conversations/{id}/messages` – Send message.
- `PATCH /v1/messages/{id}` – Edit message.
- `DELETE /v1/messages/{id}` – Soft-delete message.
- `POST /v1/conversations/{id}/read` – Mark as read.

### Example: Send Message

```typescript
// Node.js/Express
router.post('/v1/conversations/:id/messages', authMiddleware, async (req, res) => {
  const { id } = req.params;
  const { content, content_type, reply_to_message_id } = req.body;
  const profileId = req.user.profile_id;

  // Check membership
  const { data: participant, error: participantError } = await supabase
    .from('conversation_participants')
    .select('id')
    .eq('conversation_id', id)
    .eq('profile_id', profileId)
    .single();

  if (participantError || !participant) {
    return res.status(403).json({ error: 'Not a participant' });
  }

  const { data: message, error } = await supabase
    .from('messages')
    .insert({
      conversation_id: id,
      sender_profile_id: profileId,
      content,
      content_type: content_type || 'text',
      reply_to_message_id
    })
    .select('*')
    .single();

  if (error) {
    return res.status(500).json({ error: 'Failed to send message' });
  }

  res.status(201).json(message);
});
```

### Pagination

```http
GET /v1/conversations/{id}/messages?limit=50&before=2025-11-30T10:00:00Z
```

- `limit` max 100.
- `before` for backward pagination.

---

## Frontend Integration (Next.js + Supabase JS)

### Subscription Setup

```tsx
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.NEXT_PUBLIC_SUPABASE_KEY!);

export function useConversationRealtime(conversationId: string) {
  const [messages, setMessages] = useState<any[]>([]);

  useEffect(() => {
    let isMounted = true;

    const channel = supabase.channel(`room:${conversationId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `conversation_id=eq.${conversationId}`
        },
        payload => {
          if (!isMounted) return;
          setMessages(prev => [payload.new, ...prev]);
        }
      )
      .subscribe();

    return () => {
      isMounted = false;
      supabase.removeChannel(channel);
    };
  }, [conversationId]);

  return { messages, setMessages };
}
```

### Presence & Typing Indicators

```tsx
useEffect(() => {
  const presenceChannel = supabase.channel(`presence:${conversationId}`, {
    config: { presence: { key: userId } }
  });

  presenceChannel
    .on('presence', { event: 'sync' }, () => {
      const state = presenceChannel.presenceState();
      // state = { userId: [{ online_at: ... }], ... }
      setOnlineUsers(Object.keys(state));
    })
    .subscribe(async status => {
      if (status === 'SUBSCRIBED') {
        await presenceChannel.track({ online_at: new Date().toISOString() });
      }
    });

  return () => {
    supabase.removeChannel(presenceChannel);
  };
}, [conversationId, userId]);
```

---

## Message Suggestion Agent Integration

When a new conversation is created from an accepted match, the backend triggers the Message Suggestion Agent to generate 3–5 context-aware openers.

### Flow

1. Match accepted → conversation created.
2. Event emitted to AI Orchestrator.
3. Agent calls LLM (e.g., Llama 3) with profiles + match context.
4. Suggestions saved as `content_type='suggested'` in `messages`.
5. Client displays suggestions as clickable chips.

### Example Internal API

```python
# FastAPI internal service
@app.post('/internal/message-suggestions')
async def generate_suggestions(request: SuggestionRequest):
    prompt = build_prompt(request)

    response = await completion(
        model="llama3-70b",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=256
    )

    suggestions = parse_suggestions(response)

    # Persist suggestions
    for s in suggestions:
        supabase.table('messages').insert({
            'conversation_id': request.conversation_id,
            'sender_profile_id': request.system_profile_id,
            'content': s,
            'content_type': 'suggested',
            'metadata': {'source': 'message_suggestion_agent'}
        }).execute()

    return { 'suggestions': suggestions }
```

---

## Security & Privacy

- RLS ensures only participants see messages.
- Content is stored encrypted at rest (PostgreSQL + disk encryption).
- Optional Phase 2: E2E encryption where server only sees ciphertext.
- Strict logging: no message bodies in logs.
- Rate limits: max 30 messages/minute/user to prevent spam.

### Abuse & Spam Controls

- Heuristic filters for abusive content (keywords + LLM classifier).
- Report flow: `/v1/messages/{id}/report` stores reports for moderator review.
- Temporary muting of abusive users in conversations.

---

## Performance & Scaling

### Targets

- p95 send→receive latency: <1s.
- 10K concurrent connections (MVP), 100K+ in Phase 2.
- 1M messages/day with PostgreSQL partitioning by month.

### Optimizations

- `idx_messages_conversation_created_at` for fast pagination.
- Archive old conversations to S3/Glacier after 1 year (Phase 2).
- Use Supabase Realtime filters to limit payload size.
- Compress WebSocket frames where supported.

---

## Monitoring

```yaml
realtime_messages_total:
  help: Total realtime messages sent
  type: counter
  labels:
    status: [sent, delivered, failed]

realtime_latency_seconds:
  help: Send-to-deliver latency
  type: histogram
  buckets: [0.1, 0.3, 0.5, 1, 2, 5]

realtime_connections:
  help: Active WebSocket connections
  type: gauge
  labels:
    region: [eu, us, asia]

message_spam_reports:
  help: Number of spam/abuse reports
  type: counter
  labels:
    severity: [low, medium, high]
```

Alerts:
- Latency >1s p95.
- Failed messages >1%.
- Spam reports spike.

---

## Roadmap

- Phase 1: Text-only messaging, suggestions, presence.
- Phase 2: Attachments, E2E encryption, message reactions.
- Phase 3: Voice notes, video calls (WebRTC integration).
