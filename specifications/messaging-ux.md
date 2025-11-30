# specifications/messaging-ux.md

## Messaging UX Design

This document details the user experience for real-time messaging in OpenHR, focusing on seamless conversations, AI-assisted communication, and mobile-first interactions. The design prioritizes low latency (<1s message delivery), rich media support, and intuitive threading for co-founder collaboration.

### Design Objectives

- **Real-time Collaboration:** Instant message delivery with typing indicators and read receipts.
- **AI Assistance:** Context-aware message suggestions, auto-replies, and conversation summaries.
- **Mobile-First:** Touch-friendly interface with swipe gestures and offline queuing.
- **Contextual:** Messages include profile previews, skill highlights, and shared match data.
- **Privacy:** End-to-end encryption for sensitive discussions, with optional ephemeral messages.

**Tech Stack:** Supabase Realtime (WebSockets), Next.js 14, Tailwind CSS, shadcn/ui, Framer Motion, Zustand for state.

---

## Conversation Interface

The messaging system uses a familiar Slack/Discord-like layout with sidebar conversations and main chat area. Conversations are initiated from match connections and support 1:1 and small group chats (up to 4 co-founders).

### Layout Structure

```tsx
// components/messaging/conversations.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

import { ConversationList } from './conversation-list';
import { MessagesContainer } from './messages-container';
import { MessageComposer } from './message-composer';
import { Header } from '@/components/layout/header';

function ConversationsPage() {
  const [activeConversationId, setActiveConversationId] = useState<string | null>(null);
  const [searchTerm, setSearchTerm] = useState('');
  
  const { data: conversations } = useQuery({
    queryKey: ['conversations', searchTerm],
    queryFn: () => fetchConversations({ search: searchTerm }),
    staleTime: 30 * 1000,
  });
  
  const activeConversation = conversations?.find(c => c.id === activeConversationId);
  
  return (
    <div className="flex h-screen bg-gray-50">
      <Header />
      
      {/* Sidebar - Conversation List */}
      <div className="w-80 flex-shrink-0 border-r border-gray-200 bg-white">
        <ConversationList
          conversations={conversations || []}
          activeId={activeConversationId}
          searchTerm={searchTerm}
          onSearchChange={setSearchTerm}
          onSelect={setActiveConversationId}
        />
      </div>
      
      {/* Main Chat Area */}
      <div className="flex-1 flex flex-col overflow-hidden">
        {activeConversation ? (
          <>
            {/* Conversation Header */}
            <div className="border-b border-gray-200 bg-white p-4">
              <div className="flex items-center space-x-4">
                <div className="flex items-center space-x-3">
                  {activeConversation.isGroup ? (
                    <div className="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center">
                      <UsersIcon className="h-5 w-5 text-white" />
                    </div>
                  ) : (
                    <Avatar className="w-10 h-10">
                      <AvatarImage src={activeConversation.avatar} />
                      <AvatarFallback>{activeConversation.initials}</AvatarFallback>
                    </Avatar>
                  )}
                  <div className="min-w-0 flex-1">
                    <h2 className="text-lg font-semibold text-gray-900 truncate">
                      {activeConversation.name}
                    </h2>
                    <p className="text-sm text-gray-500 truncate">
                      {activeConversation.subtitle}
                    </p>
                  </div>
                </div>
                
                <div className="flex items-center space-x-2 ml-auto">
                  {activeConversation.unreadCount > 0 && (
                    <Badge className="text-xs">
                      {activeConversation.unreadCount}
                    </Badge>
                  )}
                  <Button variant="ghost" size="sm">
                    <Search className="h-4 w-4" />
                  </Button>
                  <Button variant="ghost" size="sm">
                    <MoreVertical className="h-4 w-4" />
                  </Button>
                </div>
              </div>
            </div>
            
            {/* Messages */}
            <MessagesContainer
              conversationId={activeConversationId}
              messages={activeConversation.messages}
              userId={user.id}
            />
            
            {/* Composer */}
            <MessageComposer
              conversationId={activeConversationId}
              onSend={sendMessage}
              suggestions={getMessageSuggestions(activeConversation)}
            />
          </>
        ) : (
          <div className="flex-1 flex items-center justify-center p-8">
            <div className="text-center">
              <MessageCircle className="h-12 w-12 text-gray-400 mx-auto mb-4" />
              <h3 className="text-lg font-medium text-gray-900 mb-2">Select a conversation</h3>
              <p className="text-gray-500">Choose from your recent messages to start chatting</p>
              <Button className="mt-4" onClick={() => startNewConversation()}>
                New Message
              </Button>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

### Conversation List Sidebar

```tsx
// components/messaging/conversation-list.tsx
import { useMemo } from 'react';
import { useRealtimeSubscription } from '@/hooks/use-realtime';

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';

function ConversationList({
  conversations,
  activeId,
  searchTerm = '',
  onSearchChange,
  onSelect
}: ConversationListProps) {
  const filteredConversations = useMemo(() => {
    return conversations.filter(conv => 
      conv.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      conv.lastMessage?.toLowerCase().includes(searchTerm.toLowerCase())
    ).sort((a, b) => new Date(b.lastMessageTime) - new Date(a.lastMessageTime));
  }, [conversations, searchTerm]);
  
  // Realtime updates for new messages
  useRealtimeSubscription('messages', (payload) => {
    if (payload.conversationId) {
      // Update conversation in list
      updateConversationList(payload);
    }
  });
  
  return (
    <div className="flex flex-col h-full">
      {/* Header */}
      <div className="p-4 border-b border-gray-200">
        <div className="flex items-center justify-between mb-3">
          <h2 className="text-lg font-semibold text-gray-900">Messages</h2>
          <Button variant="ghost" size="sm" onClick={() => setShowNewConversation(true)}>
            <Plus className="h-4 w-4" />
          </Button>
        </div>
        <Input
          placeholder="Search conversations"
          value={searchTerm}
          onChange={(e) => onSearchChange(e.target.value)}
          className="h-9"
        />
      </div>
      
      {/* Empty State */}
      {filteredConversations.length === 0 && !searchTerm && (
        <div className="flex-1 flex items-center justify-center p-8">
          <div className="text-center">
            <MessageCircle className="h-12 w-12 text-gray-400 mx-auto mb-4" />
            <h3 className="text-lg font-medium text-gray-900 mb-2">No conversations yet</h3>
            <p className="text-gray-500 mb-4">Start by connecting with a match</p>
            <Button variant="outline" onClick={() => navigateToMatches()}>
              Find Matches
            </Button>
          </div>
        </div>
      )}
      
      {/* Conversations */}
      <div className="flex-1 overflow-y-auto">
        {filteredConversations.map(conversation => (
          <div
            key={conversation.id}
            className={cn(
              'flex items-center p-4 border-b border-gray-100 cursor-pointer hover:bg-gray-50',
              activeId === conversation.id && 'bg-blue-50 border-r-2 border-blue-500'
            )}
            onClick={() => onSelect(conversation.id)}
            role="button"
            tabIndex={0}
            onKeyDown={(e) => e.key === 'Enter' && onSelect(conversation.id)}
          >
            <div className="flex items-center space-x-3 flex-1 min-w-0">
              {/* Avatar */}
              {conversation.isGroup ? (
                <div className="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center">
                  <UsersIcon className="h-5 w-5 text-white" />
                </div>
              ) : (
                <Avatar className="w-10 h-10 flex-shrink-0">
                  <AvatarImage src={conversation.avatar} alt={conversation.name} />
                  <AvatarFallback className="text-xs">
                    {conversation.initials}
                  </AvatarFallback>
                </Avatar>
              )}
              
              {/* Content */}
              <div className="min-w-0 flex-1">
                <div className="flex items-center justify-between">
                  <h3 className="text-sm font-medium text-gray-900 truncate">
                    {conversation.name}
                  </h3>
                  {conversation.unreadCount > 0 && (
                    <Badge className="ml-2 text-xs bg-blue-100 text-blue-800 border-blue-200">
                      {conversation.unreadCount > 99 ? '99+' : conversation.unreadCount}
                    </Badge>
                  )}
                </div>
                <p className="text-xs text-gray-500 truncate mt-0.5">
                  {conversation.preview}
                </p>
              </div>
            </div>
            
            {/* Timestamp and Status */}
            <div className="text-right text-xs text-gray-400 ml-3 min-w-[60px] flex flex-col items-end">
              <span>{formatTime(conversation.lastMessageTime)}</span>
              {conversation.hasNewMessages && !conversation.isRead && (
                <div className="w-2 h-2 bg-blue-500 rounded-full mt-1 animate-pulse" />
              )}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Message Composer

Rich text composer with AI suggestions, file attachments, and emoji reactions. Supports markdown formatting and @mentions.

### Core Features

- **Rich Text:** Bold, italic, code blocks, lists (GitHub Flavored Markdown).
- **AI Suggestions:** Context-aware icebreakers and follow-ups based on match profile.
- **File Uploads:** Images, PDFs, code snippets (<10MB total).
- **Emojis & Reactions:** Quick emoji picker and inline reactions.
- **Mentions:** @username autocomplete for group chats.
- **Threading:** Reply to specific messages with context.

```tsx
// components/messaging/message-composer.tsx
import { useState, useRef, useEffect } from 'react';
import { useRealtimeSubscription } from '@/hooks/use-realtime';

import { Textarea } from '@/components/ui/textarea';
import { Button } from '@/components/ui/button';
import { Avatar, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Command, CommandItem } from '@/components/ui/command';

import { Send, PaperClip, Smile, Sparkles, Image, FileText } from 'lucide-react';

function MessageComposer({
  conversationId,
  onSend,
  suggestions,
  participants
}: MessageComposerProps) {
  const [message, setMessage] = useState('');
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [showEmojiPicker, setShowEmojiPicker] = useState(false);
  const [isTyping, setIsTyping] = useState(false);
  const [files, setFiles] = useState<File[]>([]);
  const [mentionQuery, setMentionQuery] = useState('');
  const [showMentions, setShowMentions] = useState(false);
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);
  
  // Typing indicators
  useEffect(() => {
    const timeout = setTimeout(() => {
      if (message.length > 0) {
        broadcastTyping(conversationId, true);
      } else {
        broadcastTyping(conversationId, false);
      }
    }, 500);
    
    return () => clearTimeout(timeout);
  }, [message]);
  
  useRealtimeSubscription(`conversations.${conversationId}.typing`, (payload) => {
    if (payload.userId !== user.id) {
      setIsTyping(true);
      setTimeout(() => setIsTyping(false), 3000);
    }
  });
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!message.trim() && files.length === 0) return;
    
    onSend({
      content: message,
      files,
      conversationId,
      timestamp: new Date().toISOString(),
    });
    
    setMessage('');
    setFiles([]);
    setShowSuggestions(false);
    textareaRef.current?.focus();
  };
  
  const handleSuggestionClick = (suggestion: string) => {
    setMessage(suggestion);
    setShowSuggestions(false);
    textareaRef.current?.focus();
  };
  
  const handleFileSelect = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files) {
      const newFiles = Array.from(e.target.files).filter(file => {
        const sizeInMB = file.size / (1024 * 1024);
        return sizeInMB <= 10;
      });
      setFiles(prev => [...prev, ...newFiles]);
    }
  };
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    // Markdown shortcuts
    if (e.ctrlKey || e.metaKey) {
      if (e.key === 'b') {
        e.preventDefault();
        insertTextAtCursor('**bold**');
      } else if (e.key === 'i') {
        e.preventDefault();
        insertTextAtCursor('*italic*');
      } else if (e.key === 'k') {
        e.preventDefault();
        insertTextAtCursor('`code`');
      } else if (e.key === 'Enter') {
        e.preventDefault();
        handleSubmit(e);
      }
    }
    
    // Mention detection
    if (e.key === '@' && !mentionQuery) {
      e.preventDefault();
      setMentionQuery('@');
      setShowMentions(true);
      insertTextAtCursor('@');
    }
    
    // Tab for suggestions
    if (e.key === 'Tab' && showSuggestions && suggestions.length > 0) {
      e.preventDefault();
      handleSuggestionClick(suggestions[0]);
    }
  };
  
  const insertTextAtCursor = (text: string) => {
    const textarea = textareaRef.current;
    if (textarea) {
      const start = textarea.selectionStart;
      const end = textarea.selectionEnd;
      const newValue = message.slice(0, start) + text + message.slice(end);
      setMessage(newValue);
      
      // Position cursor after inserted text
      const newCursorPos = start + text.length;
      setTimeout(() => {
        textarea.setSelectionRange(newCursorPos, newCursorPos);
        textarea.focus();
      }, 0);
    }
  };
  
  const handleMentionSelect = (participant: Participant) => {
    const mentionText = `@${participant.name} `;
    insertTextAtCursor(mentionText);
    setMentionQuery('');
    setShowMentions(false);
  };
  
  const getMessageSuggestions = () => {
    if (!message || message.length < 3) return [];
    
    // Context-aware suggestions based on conversation
    const context = getConversationContext(activeConversation);
    return generateAISuggestions(message, context);
  };
  
  return (
    <form onSubmit={handleSubmit} className="border-t border-gray-200 bg-white">
      {/* AI Suggestions */}
      {showSuggestions && suggestions.length > 0 && (
        <div className="border-t border-gray-100 bg-gray-50 p-3">
          <p className="text-xs text-gray-500 mb-2">AI Suggestions:</p>
          <div className="flex flex-wrap gap-2">
            {suggestions.slice(0, 3).map((suggestion, index) => (
              <Button
                key={index}
                type="button"
                variant="ghost"
                size="sm"
                className="h-8 px-3 text-xs bg-white hover:bg-gray-100"
                onClick={() => handleSuggestionClick(suggestion)}
              >
                {suggestion.length > 30 ? `${suggestion.slice(0, 27)}...` : suggestion}
              </Button>
            ))}
            {suggestions.length > 3 && (
              <Button
                variant="ghost"
                size="sm"
                className="h-8 px-3 text-xs"
                onClick={() => setShowSuggestions(false)}
              >
                Dismiss
              </Button>
            )}
          </div>
        </div>
      )}
      
      {/* Typing Indicator */}
      {isTyping && (
        <div className="px-4 py-2 border-t border-gray-100">
          <div className="flex items-center space-x-2 text-sm text-gray-500">
            <div className="flex space-x-1">
              <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" />
              <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{animationDelay: '0.1s'}} />
              <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{animationDelay: '0.2s'}} />
            </div>
            <span>{activeConversation?.name} is typing...</span>
          </div>
        </div>
      )}
      
      {/* Composer */}
      <div className="p-4 max-w-4xl mx-auto">
        {/* Files Preview */}
        {files.length > 0 && (
          <div className="mb-3 p-2 bg-gray-50 rounded-lg">
            <div className="flex items-center space-x-2">
              {files.map((file, index) => (
                <div key={index} className="flex items-center space-x-2 p-2 bg-white rounded border">
                  {file.type.startsWith('image/') ? (
                    <Image className="h-8 w-8 object-cover rounded" src={URL.createObjectURL(file)} alt="" />
                  ) : (
                    <FileText className="h-8 w-8 text-gray-400" />
                  )}
                  <span className="text-xs truncate max-w-[120px]">{file.name}</span>
                  <Button
                    type="button"
                    variant="ghost"
                    size="sm"
                    className="h-6 w-6 p-0 ml-1"
                    onClick={() => setFiles(prev => prev.filter((_, i) => i !== index))}
                  >
                    <X className="h-3 w-3" />
                  </Button>
                </div>
              ))}
            </div>
          </div>
        )}
        
        {/* Main Input Area */}
        <div className="flex items-end space-x-3">
          {/* File Attachment */}
          <Popover open={showEmojiPicker} onOpenChange={setShowEmojiPicker}>
            <PopoverTrigger asChild>
              <Button
                type="button"
                variant="ghost"
                size="sm"
                className="h-10 w-10 p-0 hover:bg-gray-100"
                onClick={() => fileInputRef.current?.click()}
              >
                <PaperClip className="h-4 w-4" />
              </Button>
            </PopoverTrigger>
            <input
              ref={fileInputRef}
              type="file"
              multiple
              className="hidden"
              onChange={handleFileSelect}
              accept="image/*,.pdf,.doc,.docx,.txt,application/zip"
            />
          </Popover>
          
          {/* Mention Suggestions */}
          {showMentions && mentionQuery && (
            <div className="absolute bottom-16 left-12 z-50 bg-white border shadow-lg rounded-md max-h-48 overflow-y-auto">
              <Command>
                <CommandList>
                  {participants
                    .filter(p => p.name.toLowerCase().includes(mentionQuery.slice(1).toLowerCase()))
                    .slice(0, 5)
                    .map(participant => (
                      <CommandItem
                        key={participant.id}
                        onSelect={() => handleMentionSelect(participant)}
                        className="flex items-center space-x-3 px-3 py-2 cursor-pointer"
                      >
                        <Avatar className="w-8 h-8">
                          <AvatarImage src={participant.avatar} />
                          <AvatarFallback>{participant.initials}</AvatarFallback>
                        </Avatar>
                        <span className="text-sm font-medium">{participant.name}</span>
                      </CommandItem>
                    ))}
                </CommandList>
              </Command>
            </div>
          )}
          
          {/* Text Input */}
          <div className="flex-1 relative">
            <Textarea
              ref={textareaRef}
              value={message}
              onChange={(e) => setMessage(e.target.value)}
              onKeyDown={handleKeyDown}
              placeholder={suggestions.length > 0 ? 'Try our AI suggestions...' : 'Type a message...'}
              className="min-h-[44px] max-h-32 resize-none pr-12 focus-visible:ring-2 focus-visible:ring-blue-500 rounded-lg border border-gray-200"
              onFocus={() => {
                if (suggestions.length > 0 && !message.trim()) {
                  setShowSuggestions(true);
                }
              }}
              rows={1}
            />
            
            {/* Character count */}
            {message.length > 1000 && (
              <div className="absolute bottom-2 right-10 text-xs text-red-500">
                {message.length}/1000
              </div>
            )}
          </div>
          
          {/* Actions */}
          <div className="flex items-center space-x-1">
            {/* Emoji Picker */}
            <Popover open={showEmojiPicker} onOpenChange={setShowEmojiPicker}>
              <PopoverTrigger asChild>
                <Button
                  type="button"
                  variant="ghost"
                  size="sm"
                  className="h-10 w-10 p-0 hover:bg-gray-100"
                >
                  <Smile className="h-4 w-4" />
                </Button>
              </PopoverTrigger>
              <PopoverContent className="w-80 p-0" align="end">
                <EmojiPicker onEmojiClick={handleEmojiSelect} />
              </PopoverContent>
            </Popover>
            
            {/* AI Suggestions Toggle */}
            {suggestions.length > 0 && !showSuggestions && !message.trim() && (
              <Button
                type="button"
                variant="ghost"
                size="sm"
                className="h-10 w-10 p-0 hover:bg-gray-100"
                onClick={() => setShowSuggestions(true)}
                title="Get AI message suggestions"
              >
                <Sparkles className="h-4 w-4 text-purple-600" />
              </Button>
            )}
            
            {/* Send Button */}
            <Button
              type="submit"
              disabled={!message.trim() && files.length === 0}
              className={cn(
                'h-10 w-10 p-0 rounded-full shadow-sm transition-all',
                (message.trim() || files.length > 0) 
                  ? 'bg-blue-600 hover:bg-blue-700 text-white shadow-md hover:shadow-lg' 
                  : 'bg-gray-200 text-gray-400 cursor-not-allowed'
              )}
            >
              <Send className="h-4 w-4" />
            </Button>
          </div>
        </div>
      </div>
    </form>
  );
}
```

### Message Bubbles

Individual messages with reactions, replies, editing, and rich previews.

```tsx
// components/messaging/message-bubble.tsx
import { useState } from 'react';
import { motion } from 'framer-motion';

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Textarea } from '@/components/ui/textarea';

import { MessageCircle, Edit, Trash2, Reply, Heart, ThumbsUp, Laugh, Surprised, Sad } from 'lucide-react';

function MessageBubble({
  message,
  isOwn,
  isLastInGroup,
  onReply,
  onReact,
  onEdit,
  onDelete,
  participants
}: MessageBubbleProps) {
  const [isEditing, setIsEditing] = useState(false);
  const [editedText, setEditedText] = useState(message.content);
  const [showReactions, setShowReactions] = useState(false);
  const [isHovered, setIsHovered] = useState(false);
  
  const sender = participants.find(p => p.id === message.senderId);
  const canEdit = isOwn && Date.now() - new Date(message.createdAt).getTime() < 5 * 60 * 1000; // 5 min
  const canDelete = isOwn || isAdmin;
  
  const handleEdit = async () => {
    try {
      await onEdit(message.id, editedText);
      setIsEditing(false);
    } catch (error) {
      toast.error('Failed to edit message');
      setIsEditing(false);
    }
  };
  
  const formatMarkdown = (text: string) => {
    // Simple markdown rendering
    return text
      .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
      .replace(/\*(.*?)\*/g, '<em>$1</em>')
      .replace(/`(.*?)`/g, '<code class="bg-gray-100 px-1 py-0.5 rounded text-xs">$1</code>')
      .replace(/\n/g, '<br>');
  };
  
  const renderContent = () => {
    if (isEditing) {
      return (
        <div className="space-y-2 p-2 bg-gray-50 rounded-lg">
          <Textarea
            value={editedText}
            onChange={(e) => setEditedText(e.target.value)}
            className="min-h-[40px] resize-none border-0 bg-transparent focus-visible:ring-0"
            autoFocus
          />
          <div className="flex justify-end space-x-2 pt-1">
            <Button
              variant="ghost"
              size="sm"
              onClick={() => {
                setIsEditing(false);
                setEditedText(message.content);
              }}
            >
              Cancel
            </Button>
            <Button size="sm" onClick={handleEdit}>
              Save
            </Button>
          </div>
        </div>
      );
    }
    
    return (
      <div 
        className="prose prose-sm max-w-none break-words"
        dangerouslySetInnerHTML={{ __html: formatMarkdown(message.content) }}
      />
    );
  };
  
  const reactionEmojis = ['❤️', '👍', '😂', '😮', '😢', '🙌', '🚀', '💡'];
  
  return (
    <motion.div
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      className={cn(
        'flex mb-4 group',
        isOwn ? 'justify-end' : 'justify-start',
        isLastInGroup && 'mb-6'
      )}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      {!isOwn && (
        <Avatar className="w-8 h-8 flex-shrink-0 mt-1 mr-3">
          <AvatarImage src={sender?.avatar} alt={sender?.name} />
          <AvatarFallback className="text-xs bg-gray-100">
            {sender?.initials}
          </AvatarFallback>
        </Avatar>
      )}
      
      <div className={cn(
        'max-w-xs lg:max-w-md relative',
        isOwn ? 'order-2' : ''
      )}>
        {/* Message Bubble */}
        <div className={cn(
          'inline-block p-3 rounded-2xl shadow-sm border',
          isOwn 
            ? 'bg-gradient-to-r from-blue-500 to-blue-600 text-white border-blue-300 rounded-br-sm' 
            : 'bg-white text-gray-900 border-gray-200 rounded-bl-sm',
          isEditing && 'p-0 border-0'
        )}>
          {renderContent()}
          
          {/* Message Meta */}
          {!isEditing && (
            <div className={cn(
              'flex items-center space-x-2 mt-2 pt-1 border-t border-opacity-20',
              isOwn ? 'justify-end border-blue-300/20' : 'justify-start border-gray-200/20',
              isHovered ? 'opacity-100' : 'opacity-0 group-hover:opacity-100 transition-opacity'
            )}>
              <span className={cn('text-xs', isOwn ? 'text-blue-100' : 'text-gray-500')}>
                {formatRelativeTime(message.createdAt)}
                {message.isEdited && (
                  <span className="ml-1">(edited)</span>
                )}
              </span>
              
              {/* Actions */}
              {isHovered && (
                <div className="flex items-center space-x-1">
                  {canEdit && (
                    <Button
                      variant="ghost"
                      size="sm"
                      className={cn('h-6 px-1 text-xs', isOwn ? 'text-blue-100 hover:bg-blue-400/20' : 'text-gray-500 hover:bg-gray-100')}
                      onClick={() => setIsEditing(true)}
                    >
                      <Edit className="h-3 w-3 mr-1" />
                      Edit
                    </Button>
                  )}
                  
                  {canDelete && (
                    <Popover>
                      <PopoverTrigger asChild>
                        <Button
                          variant="ghost"
                          size="sm"
                          className={cn('h-6 px-1 text-xs', isOwn ? 'text-blue-100 hover:bg-blue-400/20' : 'text-gray-500 hover:bg-gray-100')}
                        >
                          <Trash2 className="h-3 w-3 mr-1" />
                          Delete
                        </Button>
                      </PopoverTrigger>
                      <PopoverContent className="p-2">
                        <div className="text-sm text-gray-600">
                          Are you sure you want to delete this message?
                        </div>
                        <div className="flex justify-end space-x-2 mt-2">
                          <Button
                            variant="outline"
                            size="sm"
                            onClick={() => onDelete(message.id)}
                          >
                            Delete
                          </Button>
                          <Button variant="ghost" size="sm" onClick={() => setShowDeleteConfirm(false)}>
                            Cancel
                          </Button>
                        </div>
                      </PopoverContent>
                    </Popover>
                  )}
                  
                  <Button
                    variant="ghost"
                    size="sm"
                    className={cn('h-6 px-1 text-xs', isOwn ? 'text-blue-100 hover:bg-blue-400/20' : 'text-gray-500 hover:bg-gray-100')}
                    onClick={() => onReply(message.id)}
                  >
                    <Reply className="h-3 w-3 mr-1" />
                    Reply
                  </Button>
                </div>
              )}
            </div>
          )}
        </div>
        
        {/* Reactions */}
        {!isEditing && (
          <div className="flex items-center mt-1 ml-1">
            {Object.entries(message.reactions || {}).map(([emoji, count]) => (
              count > 0 && (
                <motion.div
                  key={emoji}
                  whileHover={{ scale: 1.1 }}
                  className="flex items-center mr-1 text-xs"
                >
                  <span className={cn(
                    'px-1 py-0.5 rounded mr-0.5',
                    isOwn ? 'bg-blue-400/20 text-blue-100' : 'bg-gray-100 text-gray-700'
                  )}>
                    {emoji}
                  </span>
                  <span className={cn(isOwn ? 'text-blue-100' : 'text-gray-500')}>{count}</span>
                </motion.div>
              )
            ))}
            
            {isHovered && !isEditing && (
              <Popover open={showReactions} onOpenChange={setShowReactions}>
                <PopoverTrigger asChild>
                  <Button
                    variant="ghost"
                    size="sm"
                    className={cn('h-6 px-1 ml-1 text-xs', isOwn ? 'text-blue-100 hover:bg-blue-400/20' : 'text-gray-500 hover:bg-gray-100')}
                  >
                    Add reaction
                  </Button>
                </PopoverTrigger>
                <PopoverContent className="p-2 w-48" align="start">
                  <div className="grid grid-cols-6 gap-1">
                    {reactionEmojis.map(emoji => (
                      <Button
                        key={emoji}
                        variant="ghost"
                        size="sm"
                        className="h-8 w-8 p-0 justify-center rounded"
                        onClick={() => onReact(message.id, emoji)}
                      >
                        <span className="text-lg">{emoji}</span>
                      </Button>
                    ))}
                  </div>
                </PopoverContent>
              </Popover>
            )}
          </div>
        )}
      </div>
      
      {isOwn && (
        <Avatar className="w-8 h-8 flex-shrink-0 mt-1 ml-3 order-1">
          <AvatarImage src={user.avatar} alt={user.name} />
          <AvatarFallback className="text-xs bg-blue-100 text-blue-600">
            {user.initials}
          </AvatarFallback>
        </Avatar>
      )}
    </motion.div>
  );
}
```

### Threaded Replies

Support for replying to specific messages with context preservation.

```tsx
// components/messaging/message-thread.tsx
function MessageThread({ parentMessage, replies, onReply, participants }) {
  const [showReplyInput, setShowReplyInput] = useState(false);
  const [replyText, setReplyText] = useState('');
  
  return (
    <div className="ml-6 border-l-2 border-gray-200 pl-4">
      {/* Parent Message (Collapsed) */}
      <div className="flex items-start space-x-3 py-2 -mt-2">
        <div className="w-6 h-6 bg-gray-200 rounded-full flex-shrink-0 mt-2 border-2 border-white" />
        <div className="flex-1">
          <div className="text-xs text-gray-500 mb-1">
            {parentMessage.senderName} {formatRelativeTime(parentMessage.createdAt)}
          </div>
          <div className={cn(
            'p-2 bg-gray-100 rounded-r-lg rounded-bl-lg border border-gray-200',
            replies.length > 0 && 'border-b-0 rounded-b-none'
          )}>
            <div className="prose prose-sm max-w-none">
              {parentMessage.content}
            </div>
          </div>
        </div>
      </div>
      
      {/* Replies */}
      <div className="space-y-2 mb-2">
        {replies.map(reply => (
          <MessageBubble
            key={reply.id}
            message={reply}
            isOwn={reply.senderId === user.id}
            isLastInGroup={false}
            onReply={() => setShowReplyInput(true)}
            participants={participants}
          />
        ))}
      </div>
      
      {/* Reply Input */}
      {showReplyInput && (
        <div className="pb-2">
          <form onSubmit={(e) => {
            e.preventDefault();
            onReply(parentMessage.id, replyText);
            setReplyText('');
          }}>
            <div className="flex items-end space-x-2">
              <div className="w-6 h-6 bg-blue-100 rounded-full flex items-center justify-center flex-shrink-0">
                <span className="text-xs font-medium text-blue-600">{user.initials}</span>
              </div>
              <Textarea
                value={replyText}
                onChange={(e) => setReplyText(e.target.value)}
                placeholder="Write a reply..."
                className="flex-1 min-h-[32px] max-h-24 resize-none mr-2"
                rows={1}
              />
              <Button type="submit" size="sm" className="h-9 px-3">
                Reply
              </Button>
            </div>
          </form>
        </div>
      )}
      
      {/* Show More Replies */}
      {replies.length > 3 && !showAllReplies && (
        <Button
          variant="ghost"
          size="sm"
          className="text-xs text-gray-500 h-auto p-1"
          onClick={() => setShowAllReplies(true)}
        >
          Show {replies.length - 3} more replies
        </Button>
      )}
    </div>
  );
}
```

---

## AI Message Assistance

Context-aware suggestions that adapt to conversation flow and user profiles.

### Suggestion Types

1. **Icebreakers:** Initial messages based on shared skills/interests.
2. **Follow-ups:** Context-aware responses to keep conversation flowing.
3. **Questions:** Thoughtful questions about experience, availability, vision.
4. **Summaries:** Conversation recaps for long threads.
5. **Clarifications:** Polite ways to ask for more information.

```tsx
// lib/ai-message-suggestions.ts
import { OpenAI } from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

interface SuggestionContext {
  conversation: Conversation;
  lastMessage: string;
  userProfile: Profile;
  matchProfile: Profile;
  sharedSkills: string[];
  conversationHistory: Message[];
}

export async function generateMessageSuggestions(context: SuggestionContext): Promise<string[]> {
  const systemPrompt = `
You are an AI assistant helping co-founders and developers connect on OpenHR.

Context:
- User: ${context.userProfile.headline}
- Match: ${context.matchProfile.headline}
- Shared skills: ${context.sharedSkills.join(', ')}
- Last message: ${context.lastMessage}

Generate 3 helpful, professional message suggestions that:
1. Reference shared interests/skills when appropriate
2. Ask thoughtful questions about experience or vision
3. Keep the conversation moving toward collaboration
4. Sound natural and conversational
5. Are under 2 sentences each

Format as a numbered list:
1. Suggestion one
2. Suggestion two
3. Suggestion three
  `;
  
  try {
    const response = await openai.chat.completions.create({
      model: 'gpt-4o-mini',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: 'Generate 3 message suggestions' }
      ],
      temperature: 0.7,
      max_tokens: 200,
    });
    
    const suggestions = response.choices[0].message.content
      ?.split('\n')
      .filter(line => line.trim() && !line.startsWith('#'))
      .map(line => line.replace(/^\d+\.\s*/, '').trim()) || [];
    
    return suggestions.slice(0, 3);
  } catch (error) {
    console.error('AI suggestion error:', error);
    // Fallback suggestions
    return [
      `I noticed we both work with ${context.sharedSkills[0] || 'similar technologies'}. What's your favorite project using it?`,
      `What's the most exciting part of your current work?`,
      `Are you looking for something specific in a co-founder relationship?`
    ];
  }
}

// Usage in component
const [suggestions, setSuggestions] = useState<string[]>([]);

useEffect(() => {
  if (conversationId && lastMessage) {
    generateMessageSuggestions({
      conversation: activeConversation,
      lastMessage,
      userProfile: userProfile,
      matchProfile: matchProfile,
      sharedSkills: sharedSkills,
      conversationHistory: messages.slice(-5),
    }).then(setSuggestions);
  }
}, [lastMessage, conversationId]);
```

### Smart Reply Suggestions

Quick reply buttons that appear below messages.

```tsx
// components/messaging/smart-replies.tsx
function SmartReplies({ message, onReply, conversationContext }) {
  const [replies, setReplies] = useState<string[]>([]);
  
  useEffect(() => {
    generateSmartReplies(message.content, conversationContext).then(setReplies);
  }, [message.content]);
  
  if (replies.length === 0) return null;
  
  return (
    <div className="flex items-center mt-2 ml-10 space-x-1">
      {replies.slice(0, 3).map((reply, index) => (
        <Button
          key={index}
          variant="ghost"
          size="sm"
          className="h-8 px-3 text-xs bg-gray-100 hover:bg-gray-200 rounded-full"
          onClick={() => onReply(reply)}
        >
          {reply.length > 25 ? `${reply.slice(0, 22)}...` : reply}
        </Button>
      ))}
      {replies.length > 3 && (
        <Button
          variant="ghost"
          size="sm"
          className="h-8 px-3 text-xs text-gray-500"
          onClick={() => setShowAllReplies(true)}
        >
          +{replies.length - 3} more
        </Button>
      )}
    </div>
  );
}
```

---

## Mobile Messaging Experience

Optimized for touch interactions with gesture support and offline capabilities.

### Touch Targets & Gestures

- **Minimum Touch Target:** 44x44px for all interactive elements.
- **Swipe Left:** Archive conversation or delete message.
- **Swipe Right:** Mark as read or add reaction.
- **Long Press:** Show message actions (reply, react, copy, forward).
- **Pull to Refresh:** Update conversation list.
- **Double Tap:** Quick thumbs up reaction.

```tsx
// components/messaging/mobile-message-gestures.tsx
import { useSwipeable } from 'react-swipeable';

function MobileMessageBubble({ message, ...props }) {
  const leftSwipeHandlers = useSwipeable({
    onSwipedLeft: () => {
      if (isOwn) {
        showDeleteConfirm(message.id);
      } else {
        archiveConversation();
      }
    },
    trackMouse: true,
    preventScrollOnSwipe: ['vertical'],
    swipeDuration: 500,
    swipeVelocity: 0.5,
  });
  
  const rightSwipeHandlers = useSwipeable({
    onSwipedRight: () => {
      if (message.isRead) {
        markUnread(message.id);
      } else {
        markRead(message.id);
      }
    },
  });
  
  const handleLongPress = useCallback(() => {
    showMessageActions(message.id);
  }, [message.id]);
  
  const handleDoubleTap = useCallback(() => {
    reactToMessage(message.id, '👍');
  }, [message.id]);
  
  return (
    <div
      {...leftSwipeHandlers}
      {...rightSwipeHandlers}
      onTouchStart={(e) => {
        // Long press detection
        const timer = setTimeout(() => handleLongPress(), 500);
        e.currentTarget.dataset.timer = timer;
      }}
      onTouchEnd={(e) => {
        const timer = e.currentTarget.dataset.timer;
        if (timer) clearTimeout(Number(timer));
      }}
      onTouchMove={() => {
        const timer = e.currentTarget.dataset.timer;
        if (timer) clearTimeout(Number(timer));
      }}
      onDoubleClick={handleDoubleTap}
      className="message-bubble touch-manipulation"
    >
      <MessageBubbleContent message={message} {...props} />
      
      {/* Swipe Feedback */}
      <motion.div
        className="absolute inset-y-0 left-0 bg-red-500 flex items-center justify-center text-white text-sm"
        initial={{ x: '-100%' }}
        animate={{ x: swipingLeft ? 0 : '-100%' }}
      >
        Delete
      </motion.div>
      
      <motion.div
        className="absolute inset-y-0 right-0 bg-green-500 flex items-center justify-center text-white text-sm"
        initial={{ x: '100%' }}
        animate={{ x: swipingRight ? 0 : '100%' }}
      >
        Read
      </motion.div>
    </div>
  );
}
```

### Offline Support

Progressive Web App capabilities with message queuing.

```tsx
// lib/offline-messaging.ts
import { writable } from 'svelte/store'; // or zustand for React

interface OfflineMessage {
  id: string;
  content: string;
  conversationId: string;
  timestamp: Date;
  status: 'pending' | 'sent' | 'failed';
}

const offlineMessages = writable<OfflineMessage[]>([]);

// Service Worker registration
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js').then(registration => {
    console.log('SW registered');
  });
}

// Queue message when offline
export async function queueMessage(message: Omit<OfflineMessage, 'id' | 'status'>) {
  if (navigator.onLine) {
    try {
      const result = await sendMessage(message);
      return result;
    } catch (error) {
      // Network error, queue it
      const queuedMessage = {
        ...message,
        id: generateId(),
        status: 'pending' as const,
      };
      offlineMessages.update(msgs => [...msgs, queuedMessage]);
      return queuedMessage;
    }
  } else {
    const queuedMessage = {
      ...message,
      id: generateId(),
      status: 'pending' as const,
    };
    offlineMessages.update(msgs => [...msgs, queuedMessage]);
    return queuedMessage;
  }
}

// Sync when back online
window.addEventListener('online', async () => {
  const pendingMessages = offlineMessages.get();
  
  for (const message of pendingMessages.filter(m => m.status === 'pending')) {
    try {
      await sendMessage(message);
      offlineMessages.update(msgs => 
        msgs.map(m => m.id === message.id ? { ...m, status: 'sent' } : m)
      );
    } catch (error) {
      offlineMessages.update(msgs => 
        msgs.map(m => m.id === message.id ? { ...m, status: 'failed' } : m)
      );
    }
  }
});

// UI for pending messages
function PendingMessagesIndicator() {
  const pendingCount = offlineMessages.subscribe(msgs => 
    msgs.filter(m => m.status === 'pending').length
  );
  
  return pendingCount > 0 ? (
    <Badge className="ml-2">
      {pendingCount} messages queued
    </Badge>
  ) : null;
}
```

---

## Group Conversations

Support for small group chats (up to 4 participants) with role-based permissions.

### Group Features

- **Admin Controls:** Add/remove members, set conversation name, mute notifications.
- **@everyone:** Mention all participants in group.
- **Role Indicators:** Show co-founder vs developer roles in member list.
- **Shared Resources:** Pin important messages, share files with expiration.

```tsx
// components/messaging/group-conversation.tsx
function GroupConversationHeader({ conversation, onAddMember, onLeave }) {
  const memberCount = conversation.participants.length;
  const isAdmin = conversation.admins.includes(user.id);
  
  return (
    <div className="border-b border-gray-200 bg-white p-4">
      <div className="flex items-center justify-between">
        {/* Group Avatar & Name */}
        <div className="flex items-center space-x-3">
          <div className="w-10 h-10 bg-gradient-to-br from-purple-500 to-pink-600 rounded-full flex items-center justify-center">
            <UsersIcon className="h-5 w-5 text-white" />
          </div>
          <div>
            <h2 className="text-lg font-semibold text-gray-900 truncate max-w-[200px]">
              {conversation.name || `Group chat (${memberCount})`}
            </h2>
            <p className="text-sm text-gray-500">
              {memberCount} {memberCount === 1 ? 'member' : 'members'}
            </p>
          </div>
        </div>
        
        {/* Actions */}
        <div className="flex items-center space-x-2">
          {isAdmin && (
            <>
              <Button variant="ghost" size="sm" onClick={onAddMember}>
                <UserPlus className="h-4 w-4 mr-1" />
                Add Member
              </Button>
              <Button variant="outline" size="sm" onClick={() => setShowRename(true)}>
                Rename
              </Button>
            </>
          )}
          <Button variant="ghost" size="sm">
            <Bell className="h-4 w-4" />
          </Button>
          <Button variant="ghost" size="sm" onClick={onLeave}>
            Leave
          </Button>
        </div>
      </div>
      
      {/* Member List (Collapsed on Mobile) */}
      <div className="mt-3 pt-3 border-t border-gray-100">
        <div className="flex items-center space-x-3 overflow-x-auto pb-1">
          {conversation.participants.map(participant => (
            <div key={participant.id} className="flex-shrink-0">
              <Avatar className="w-8 h-8">
                <AvatarImage src={participant.avatar} alt={participant.name} />
                <AvatarFallback className="text-xs">
                  {participant.initials}
                </AvatarFallback>
              </Avatar>
              <div className="text-center text-xs mt-1 truncate w-16">
                {participant.name.split(' ')[0]}
                <div className="text-[10px] text-gray-500 capitalize">
                  {participant.role}
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Group member management
function GroupMemberList({ members, isAdmin, onRemoveMember, onPromoteAdmin }) {
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="outline" className="w-full justify-between">
          Members ({members.length})
          <ChevronDown className="h-4 w-4 ml-2" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-80 p-0" align="start">
        <div className="max-h-60 overflow-y-auto">
          {members.map(member => (
            <div key={member.id} className="flex items-center justify-between p-3 hover:bg-gray-50">
              <div className="flex items-center space-x-3">
                <Avatar className="w-8 h-8">
                  <AvatarImage src={member.avatar} />
                  <AvatarFallback>{member.initials}</AvatarFallback>
                </Avatar>
                <div>
                  <div className="font-medium text-sm text-gray-900">
                    {member.name}
                  </div>
                  <div className="text-xs text-gray-500 capitalize">
                    {member.role}
                  </div>
                </div>
              </div>
              
              {isAdmin && member.id !== user.id && (
                <div className="flex items-center space-x-2">
                  {member.isAdmin && (
                    <Badge variant="secondary" className="text-xs">Admin</Badge>
                  )}
                  <Button
                    variant="ghost"
                    size="sm"
                    className="h-6 px-2 text-xs"
                    onClick={() => {
                      if (member.isAdmin) {
                        onRemoveAdmin(member.id);
                      } else {
                        onPromoteAdmin(member.id);
                      }
                    }}
                  >
                    {member.isAdmin ? 'Remove Admin' : 'Make Admin'}
                  </Button>
                  <Button
                    variant="ghost"
                    size="sm"
                    className="h-6 px-2 text-xs text-red-600"
                    onClick={() => onRemoveMember(member.id)}
                  >
                    Remove
                  </Button>
                </div>
              )}
            </div>
          ))}
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

---

## Notifications & Alerts

In-app and push notifications with customizable preferences.

### Notification Types

1. **New Messages:** Real-time alerts with message preview.
2. **Match Connections:** When someone accepts your connection request.
3. **Profile Views:** When someone views your profile.
4. **Conversation Updates:** New members added, name changes.
5. **System:** Platform updates, verification status.

```tsx
// components/notifications/notification-center.tsx
function NotificationCenter() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  
  useRealtimeSubscription('notifications', (payload) => {
    setNotifications(prev => [payload, ...prev]);
    if (!payload.read) {
      setUnreadCount(prev => prev + 1);
    }
  });
  
  const markAsRead = async (notificationId: string) => {
    await markNotificationRead(notificationId);
    setUnreadCount(prev => Math.max(0, prev - 1));
    setNotifications(prev => 
      prev.map(n => n.id === notificationId ? { ...n, read: true } : n)
    );
  };
  
  const getNotificationIcon = (type: NotificationType) => {
    switch (type) {
      case 'message': return <MessageCircle className="h-5 w-5" />;
      case 'connection': return <UsersIcon className="h-5 w-5" />;
      case 'profile_view': return <EyeIcon className="h-5 w-5" />;
      case 'group_update': return <UsersIcon className="h-5 w-5" />;
      default: return <Bell className="h-5 w-5" />;
    }
  };
  
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="ghost" className="relative h-9 w-9 p-0">
          <Bell className="h-5 w-5" />
          {unreadCount > 0 && (
            <Badge className="absolute -top-1 -right-1 h-5 w-5 p-0 rounded-full text-xs">
              {unreadCount > 99 ? '99+' : unreadCount}
            </Badge>
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-80 p-0 max-h-96" align="end">
        <div className="flex items-center justify-between p-4 border-b border-gray-200">
          <h3 className="text-sm font-semibold text-gray-900">Notifications</h3>
          <Button
            variant="ghost"
            size="sm"
            onClick={() => markAllRead()}
            className="text-xs h-6 px-2"
          >
            Mark all read
          </Button>
        </div>
        
        <div className="max-h-72 overflow-y-auto">
          {notifications.length === 0 ? (
            <div className="p-8 text-center">
              <Bell className="h-8 w-8 text-gray-400 mx-auto mb-2" />
              <p className="text-sm text-gray-500">No notifications</p>
            </div>
          ) : (
            notifications.map(notification => (
              <motion.div
                key={notification.id}
                initial={{ opacity: 0, y: -10 }}
                animate={{ opacity: 1, y: 0 }}
                className={cn(
                  'flex items-start p-4 border-b border-gray-100 hover:bg-gray-50',
                  !notification.read && 'bg-blue-50 border-blue-200'
                )}
                onClick={() => markAsRead(notification.id)}
              >
                <div className="flex-shrink-0 mt-0.5">
                  <div className={cn(
                    'w-10 h-10 rounded-full flex items-center justify-center',
                    notification.read ? 'bg-gray-100' : 'bg-blue-100'
                  )}>
                    {getNotificationIcon(notification.type)}
                  </div>
                </div>
                <div className="ml-3 flex-1 min-w-0">
                  <div className="flex items-center justify-between">
                    <h4 className="text-sm font-medium text-gray-900 truncate">
                      {notification.title}
                    </h4>
                    <span className="text-xs text-gray-400 ml-2 whitespace-nowrap">
                      {formatRelativeTime(notification.createdAt)}
                    </span>
                  </div>
                  <p className="text-sm text-gray-600 mt-1 line-clamp-2">
                    {notification.body}
                  </p>
                  {notification.actionUrl && (
                    <div className="mt-2">
                      <Button
                        variant="link"
                        size="sm"
                        className="h-auto p-0 text-xs text-blue-600 hover:text-blue-700"
                        onClick={(e) => {
                          e.stopPropagation();
                          window.open(notification.actionUrl, '_blank');
                        }}
                      >
                        {notification.actionText || 'View'}
                      </Button>
                    </div>
                  )}
                </div>
              </motion.div>
            ))
          )}
        </div>
        
        <div className="p-4 border-t border-gray-200">
          <Button variant="outline" className="w-full text-xs">
            View all notifications
          </Button>
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

### Push Notifications

Web Push API integration for background messages.

```tsx
// lib/push-notifications.ts
async function registerPushNotifications() {
  if ('serviceWorker' in navigator && 'PushManager' in window) {
    const registration = await navigator.serviceWorker.register('/sw.js');
    
    const vapidPublicKey = process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY;
    const applicationServerKey = urlBase64ToUint8Array(vapidPublicKey);
    
    const permission = await Notification.requestPermission();
    if (permission === 'granted') {
      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey,
      });
      
      // Send subscription to backend
      await savePushSubscription(subscription.toJSON());
    }
  }
}

function urlBase64ToUint8Array(base64String: string) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/');
  
  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);
  
  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

// Notification handler
self.addEventListener('push', function(event) {
  const data = event.data?.json();
  
  const options = {
    body: data?.body,
    icon: '/icon-192x192.png',
    badge: '/badge-72x72.png',
    vibrate: [100, 50, 100],
    data: {
      url: data?.actionUrl,
      conversationId: data?.conversationId,
    },
    actions: [
      {
        action: 'reply',
        title: 'Quick Reply',
        icon: '/icon-reply.png',
      },
      {
        action: 'mark_read',
        title: 'Mark as Read',
        icon: '/icon-read.png',
      },
    ],
  };
  
  event.waitUntil(
    self.registration.showNotification(data?.title || 'OpenHR', options)
  );
});

// Notification click handler
self.addEventListener('notificationclick', function(event) {
  event.notification.close();
  
  if (event.action === 'reply') {
    // Open quick reply interface
    clients.openWindow(`/messages/quick-reply?conversation=${event.notification.data.conversationId}`);
  } else {
    event.waitUntil(
      clients.openWindow(event.notification.data.url || '/messages')
    );
  }
});
```

---

## Privacy & Security

### End-to-End Encryption

Messages are encrypted using Web Crypto API with per-conversation keys.

```tsx
// lib/message-encryption.ts
async function generateConversationKey(conversationId: string, participants: string[]) {
  // Derive key from conversation ID and participant IDs
  const encoder = new TextEncoder();
  const data = encoder.encode(conversationId + participants.sort().join(','));
  
  const keyMaterial = await crypto.subtle.importKey(
    'raw',
    data,
    { name: 'PBKDF2' },
    false,
    ['deriveBits', 'deriveKey']
  );
  
  return crypto.subtle.deriveKey(
    {
      name: 'AES-GCM',
      salt: new Uint8Array(16), // Use proper salt
      iterations: 100000,
      hash: 'SHA-256',
    },
    keyMaterial,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt']
  );
}

async function encryptMessage(message: string, key: CryptoKey) {
  const encoder = new TextEncoder();
  const iv = crypto.getRandomValues(new Uint8Array(12));
  
  const encrypted = await crypto.subtle.encrypt(
    {
      name: 'AES-GCM',
      iv,
    },
    key,
    encoder.encode(message)
  );
  
  return {
    iv: Array.from(iv),
    data: Array.from(new Uint8Array(encrypted)),
    tag: Array.from(new Uint8Array(encrypted).slice(-16)),
  };
}

async function decryptMessage(encrypted: any, key: CryptoKey) {
  const decoder = new TextDecoder();
  
  const combined = new Uint8Array(
    encrypted.iv.concat(encrypted.data).concat(encrypted.tag)
  );
  
  const decrypted = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv: new Uint8Array(encrypted.iv) },
    key,
    combined
  );
  
  return decoder.decode(decrypted);
}

// Usage in message sending
const conversationKey = await generateConversationKey(conversationId, participantIds);
const encryptedContent = await encryptMessage(message.content, conversationKey);

// Store encrypted message
const messageData = {
  ...message,
  content: encryptedContent, // Store as JSON
  encrypted: true,
};

// Decrypt on render
const decryptedContent = await decryptMessage(message.content, conversationKey);
```

### Ephemeral Messages

Self-destructing messages for sensitive discussions.

```tsx
// components/messaging/ephemeral-message.tsx
function EphemeralMessage({ message, expiresIn }: { message: Message; expiresIn: number }) {
  const [timeRemaining, setTimeRemaining] = useState(expiresIn);
  const [isExpired, setIsExpired] = useState(false);
  
  useEffect(() => {
    if (timeRemaining <= 0) {
      setIsExpired(true);
      deleteMessage(message.id);
      return;
    }
    
    const timer = setInterval(() => {
      setTimeRemaining(prev => prev - 1000);
    }, 1000);
    
    return () => clearInterval(timer);
  }, [timeRemaining]);
  
  if (isExpired) {
    return (
      <div className="p-3 bg-gray-100 rounded-lg text-center">
        <MessageCircle className="h-8 w-8 text-gray-400 mx-auto mb-2" />
        <p className="text-sm text-gray-500">This message has expired</p>
      </div>
    );
  }
  
  return (
    <div className="relative">
      <MessageBubble message={message} />
      <div className="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
        {Math.ceil(timeRemaining / 1000)}
      </div>
    </div>
  );
}

// Ephemeral message composer toggle
function EphemeralToggle({ isEnabled, onToggle, duration }: EphemeralToggleProps) {
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button
          variant={isEnabled ? 'default' : 'outline'}
          size="sm"
          className="h-9"
        >
          <Clock className="h-4 w-4 mr-1" />
          {isEnabled ? `${duration}s` : 'Normal'}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-48 p-2">
        <div className="space-y-1">
          {[30, 60, 300, 600, 3600].map(seconds => (
            <Button
              key={seconds}
              variant="ghost"
              className="justify-start w-full h-8"
              onClick={() => onToggle(seconds)}
            >
              <span className="mr-2">
                {seconds < 60 ? `${seconds}s` : seconds < 3600 ? `${Math.floor(seconds/60)}m` : '1h'}
              </span>
              {isEnabled && seconds === duration && (
                <Check className="h-4 w-4 ml-auto text-green-600" />
              )}
            </Button>
          ))}
          <Button
            variant="ghost"
            className="justify-start w-full h-8 text-gray-500"
            onClick={() => onToggle(0)}
          >
            Normal messages
          </Button>
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

---

## Search & History

Full-text search across all conversations with advanced filtering.

### Global Message Search

```tsx
// components/messaging/search-messages.tsx
function MessageSearch({ onSearch, searchResults }) {
  const [searchQuery, setSearchQuery] = useState('');
  const [searchFilters, setSearchFilters] = useState({
    conversation: '',
    dateFrom: '',
    dateTo: '',
    participants: [],
    hasFiles: false,
  });
  
  const debouncedSearch = useDebounce(searchQuery, 300);
  
  useEffect(() => {
    if (debouncedSearch.length >= 2) {
      onSearch(debouncedSearch, searchFilters);
    }
  }, [debouncedSearch, searchFilters]);
  
  return (
    <Card className="mb-6">
      <CardContent className="p-6">
        <div className="space-y-4">
          {/* Search Input */}
          <div className="relative">
            <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-gray-400" />
            <Input
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              placeholder="Search messages, files, @mentions..."
              className="pl-10"
            />
          </div>
          
          {/* Filters */}
          {searchResults?.conversations?.length > 1 && (
            <Select onValueChange={(value) => setSearchFilters(prev => ({ ...prev, conversation: value }))}>
              <SelectTrigger>
                <SelectValue placeholder="All conversations" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="">All conversations</SelectItem>
                {searchResults.conversations.map(conv => (
                  <SelectItem key={conv.id} value={conv.id}>
                    {conv.name}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          )}
          
          {/* Date Range */}
          <div className="grid grid-cols-2 gap-2">
            <Input
              type="date"
              placeholder="From date"
              onChange={(e) => setSearchFilters(prev => ({ ...prev, dateFrom: e.target.value }))}
            />
            <Input
              type="date"
              placeholder="To date"
              onChange={(e) => setSearchFilters(prev => ({ ...prev, dateTo: e.target.value }))}
            />
          </div>
          
          {/* Advanced Filters */}
          <div className="flex items-center space-x-4 text-sm">
            <label className="flex items-center space-x-2">
              <input
                type="checkbox"
                checked={searchFilters.hasFiles}
                onChange={(e) => setSearchFilters(prev => ({ ...prev, hasFiles: e.target.checked }))}
                className="rounded"
              />
              <span>Has attachments</span>
            </label>
            <label className="flex items-center space-x-2">
              <input
                type="checkbox"
                checked={searchFilters.unreadOnly}
                onChange={(e) => setSearchFilters(prev => ({ ...prev, unreadOnly: e.target.checked }))}
                className="rounded"
              />
              <span>Unread only</span>
            </label>
          </div>
          
          {/* Results Summary */}
          {searchResults && (
            <div className="text-sm text-gray-600">
              Found {searchResults.total} messages in {searchResults.conversations?.length || 0} conversations
              {searchResults.dateRange && ` from ${searchResults.dateRange}`}
            </div>
          )}
        </div>
      </CardContent>
    </Card>
  );
}

// Search results with conversation jump
function SearchResults({ results, onJumpToMessage }) {
  const groupedByConversation = useMemo(() => {
    return results.reduce((acc, message) => {
      if (!acc[message.conversationId]) {
        acc[message.conversationId] = {
          conversation: conversations.find(c => c.id === message.conversationId),
          messages: [],
        };
      }
      acc[message.conversationId].messages.push(message);
      return acc;
    }, {});
  }, [results]);
  
  return (
    <div className="space-y-4">
      {Object.values(groupedByConversation).map(group => (
        <Card key={group.conversation.id}>
          <CardHeader className="pb-3">
            <div className="flex items-center justify-between">
              <div className="flex items-center space-x-3">
                <Avatar className="w-8 h-8">
                  <AvatarImage src={group.conversation.avatar} />
                  <AvatarFallback>{group.conversation.initials}</AvatarFallback>
                </Avatar>
                <div>
                  <h3 className="font-medium text-gray-900">
                    {group.conversation.name}
                  </h3>
                  <p className="text-sm text-gray-500">
                    {group.messages.length} matches
                  </p>
                </div>
              </div>
              <Button
                variant="outline"
                size="sm"
                onClick={() => jumpToConversation(group.conversation.id)}
              >
                Jump to conversation
              </Button>
            </div>
          </CardHeader>
          <CardContent className="p-0">
            <div className="max-h-40 overflow-y-auto">
              {group.messages.slice(0, 5).map(message => (
                <div
                  key={message.id}
                  className="p-4 border-b border-gray-100 hover:bg-gray-50 cursor-pointer"
                  onClick={() => onJumpToMessage(message.id)}
                >
                  <div className="flex items-start space-x-3">
                    <div className="w-6 h-6 bg-gray-200 rounded-full flex-shrink-0 mt-1" />
                    <div className="flex-1 min-w-0">
                      <div className="text-xs text-gray-500 mb-1">
                        {message.senderName} • {formatRelativeTime(message.createdAt)}
                      </div>
                      <p className="text-sm text-gray-900 line-clamp-2">
                        {message.content}
                      </p>
                      {message.hasFiles && (
                        <div className="flex items-center space-x-2 mt-1 text-xs text-gray-500">
                          <PaperClip className="h-3 w-3" />
                          <span>{message.files?.length} attachments</span>
                        </div>
                      )}
                    </div>
                  </div>
                </div>
              ))}
              {group.messages.length > 5 && (
                <div className="p-4 text-center text-sm text-gray-500">
                  +{group.messages.length - 5} more messages in this conversation
                </div>
              )}
            </div>
          </CardContent>
        </Card>
      ))}
    </div>
  );
}
```

### Message Export & Backup

Export conversation history for records or migration.

```tsx
// components/messaging/export-conversation.tsx
function ExportConversation({ conversationId }: { conversationId: string }) {
  const [isExporting, setIsExporting] = useState(false);
  const [exportFormat, setExportFormat] = useState<'pdf' | 'json' | 'txt'>('pdf');
  
  const handleExport = async () => {
    setIsExporting(true);
    try {
      const conversation = await fetchConversationWithMessages(conversationId);
      const exportData = await generateExportData(conversation, exportFormat);
      
      // Download file
      const blob = new Blob([exportData], { type: getMimeType(exportFormat) });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `openhr-conversation-${conversation.name}-${new Date().toISOString().split('T')[0]}.${exportFormat}`;
      a.click();
      URL.revokeObjectURL(url);
      
      toast.success('Conversation exported successfully');
    } catch (error) {
      toast.error('Failed to export conversation');
    } finally {
      setIsExporting(false);
    }
  };
  
  const getMimeType = (format: string) => {
    switch (format) {
      case 'pdf': return 'application/pdf';
      case 'json': return 'application/json';
      case 'txt': return 'text/plain';
      default: return 'text/plain';
    }
  };
  
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="outline" size="sm">
          <Download className="h-4 w-4 mr-1" />
          Export
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-48">
        <div className="space-y-1">
          {['pdf', 'json', 'txt'].map(format => (
            <div key={format} className="flex items-center justify-between p-2 hover:bg-gray-100 rounded cursor-pointer" onClick={() => {
              setExportFormat(format as any);
              handleExport();
            }}>
              <span className="text-sm capitalize">{format} format</span>
              {format === exportFormat && <Check className="h-4 w-4 text-green-600" />}
            </div>
          ))}
        </div>
        {isExporting && (
          <div className="p-2 text-center text-sm text-gray-500">
            <Loader2 className="h-4 w-4 animate-spin mx-auto mb-1" />
            Exporting...
          </div>
        )}
      </PopoverContent>
    </Popover>
  );
}

async function generateExportData(conversation: Conversation, format: string) {
  const messages = conversation.messages.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
  
  switch (format) {
    case 'pdf':
      // Generate PDF with jsPDF or similar
      const { jsPDF } = await import('jspdf');
      const doc = new jsPDF();
      
      doc.setFontSize(16);
      doc.text(`${conversation.name} - Conversation Export`, 20, 20);
      doc.setFontSize(10);
      doc.text(`Exported: ${new Date().toLocaleString()}`, 20, 30);
      
      let yPos = 50;
      messages.forEach((message, index) => {
        if (yPos > 280) {
          doc.addPage();
          yPos = 20;
        }
        
        doc.setFontSize(8);
        doc.text(`${formatTime(message.createdAt)} - ${message.senderName}:`, 20, yPos);
        yPos += 5;
        
        const wrappedText = doc.splitTextToSize(message.content, 170);
        doc.text(wrappedText, 25, yPos);
        yPos += wrappedText.length * 4;
        
        if (message.files?.length > 0) {
          doc.text(`Attachments: ${message.files.map(f => f.name).join(', ')}`, 25, yPos);
          yPos += 5;
        }
        
        yPos += 5;
      });
      
      return doc.output('blob');
      
    case 'json':
      return JSON.stringify({
        conversation,
        exportDate: new Date().toISOString(),
        format: 'json',
      }, null, 2);
      
    case 'txt':
      return messages.map(msg => 
        `[${formatTime(msg.createdAt)}] ${msg.senderName}: ${msg.content}${
          msg.files?.length > 0 ? ` [Attachments: ${msg.files.map(f => f.name).join(', ')}]` : ''
        }`
      ).join('\n\n');
      
    default:
      throw new Error('Unsupported export format');
  }
}
```

---

## Analytics & Insights

Track conversation health and engagement metrics.

### Conversation Health Metrics

```tsx
// components/messaging/conversation-analytics.tsx
function ConversationAnalytics({ conversationId }: { conversationId: string }) {
  const { data: analytics } = useQuery({
    queryKey: ['conversation-analytics', conversationId],
    queryFn: () => fetchConversationAnalytics(conversationId),
  });
  
  if (!analytics) return null;
  
  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-lg">Conversation Insights</CardTitle>
        <p className="text-sm text-gray-600">Engagement and health metrics</p>
      </CardHeader>
      <CardContent className="space-y-6">
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
          <StatCard
            title="Response Time"
            value={`${analytics.avgResponseTime} min`}
            trend={analytics.responseTrend}
            icon={<Clock className="h-5 w-5 text-blue-600" />}
          />
          <StatCard
            title="Engagement"
            value={`${analytics.engagementScore}/100`}
            trend={analytics.engagementTrend}
            icon={<Activity className="h-5 w-5 text-green-600" />}
          />
          <StatCard
            title="Messages"
            value={analytics.totalMessages}
            trend={analytics.messageTrend}
            icon={<MessageCircle className="h-5 w-5 text-purple-600" />}
          />
          <StatCard
            title="Active Days"
            value={analytics.activeDays}
            trend={analytics.daysTrend}
            icon={<Calendar className="h-5 w-5 text-orange-600" />}
          />
        </div>
        
        {/* Suggestions */}
        {analytics.suggestions?.length > 0 && (
          <div className="space-y-3">
            <h3 className="text-sm font-medium text-gray-900">Suggestions</h3>
            <div className="space-y-2">
              {analytics.suggestions.map((suggestion, index) => (
                <div key={index} className="flex items-start space-x-3 p-3 bg-blue-50 rounded-lg">
                  <Sparkles className="h-5 w-5 text-blue-600 flex-shrink-0 mt-0.5" />
                  <div className="text-sm text-gray-900">
                    {suggestion.text}
                    {suggestion.action && (
                      <Button
                        variant="link"
                        size="sm"
                        className="h-auto p-0 ml-2 text-xs"
                        onClick={suggestion.action.onClick}
                      >
                        {suggestion.action.text}
                      </Button>
                    )}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}

interface ConversationAnalytics {
  avgResponseTime: number;
  engagementScore: number;
  totalMessages: number;
  activeDays: number;
  responseTrend: 'improving' | 'declining' | 'stable';
  engagementTrend: 'improving' | 'declining' | 'stable';
  messageTrend: 'increasing' | 'decreasing' | 'stable';
  daysTrend: 'increasing' | 'decreasing' | 'stable';
  suggestions: Array<{
    text: string;
    action?: { text: string; onClick: () => void };
  }>;
}

async function fetchConversationAnalytics(conversationId: string): Promise<ConversationAnalytics> {
  const messages = await fetchMessages(conversationId);
  const participants = await fetchConversationParticipants(conversationId);
  
  // Calculate response times
  const responseTimes = messages
    .filter((_, i) => i > 0)
    .map((msg, i) => ({
      senderId: msg.senderId,
      time: new Date(msg.createdAt).getTime() - new Date(messages[i - 1].createdAt).getTime(),
    }))
    .filter(r => r.time > 0 && r.time < 24 * 60 * 60 * 1000); // Within 24h
  
  const avgResponseTime = Math.round(
    responseTimes.reduce((sum, r) => sum + r.time, 0) / responseTimes.length / 1000 / 60
  ) || 0;
  
  // Engagement score (0-100)
  const messagesPerDay = messages.length / (Math.max(...messages.map(m => new Date(m.createdAt).getDate())) - Math.min(...messages.map(m => new Date(m.createdAt).getDate())) + 1);
  const engagementScore = Math.min(100, Math.max(0, messagesPerDay * 10));
  
  // Trends (simplified)
  const recentMessages = messages.slice(-10);
  const responseTrend = recentMessages.length > 5 && 
    recentMessages.slice(-5).reduce((sum, msg, i) => sum + (new Date(msg.createdAt).getTime() - new Date(recentMessages[i + 5]?.createdAt || 0).getTime()), 0) < 0 
    ? 'improving' : 'declining';
  
  // Generate AI suggestions
  const suggestions = await generateAnalyticsSuggestions(messages, participants);
  
  return {
    avgResponseTime,
    engagementScore,
    totalMessages: messages.length,
    activeDays: new Set(messages.map(m => new Date(m.createdAt).toDateString())).size,
    responseTrend,
    engagementTrend: 'stable',
    messageTrend: 'stable',
    daysTrend: 'stable',
    suggestions,
  };
}
```

This completes the comprehensive Messaging UX specification for OpenHR, providing a robust, secure, and engaging communication system for co-founder collaboration.

**References:**
- Supabase Realtime Documentation [web:269]
- Web Crypto API Guide [web:270]
- Push Notifications Best Practices [web:271]
- Markdown Rendering in React [web:272]