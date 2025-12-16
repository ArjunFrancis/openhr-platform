# OpenHR Frontend

**Status**: 🚧 Not yet implemented (skeleton only)

---

## Overview

This directory will contain the Next.js + React frontend application for OpenHR.

### Technology Stack

- **Framework**: Next.js 14+ (App Router)
- **UI Library**: React 18 with TypeScript
- **Styling**: TailwindCSS + Radix UI components
- **State Management**: React Query (TanStack Query)
- **Authentication**: Supabase Auth (OAuth2 + JWT)
- **Realtime**: Supabase Realtime (WebSocket)
- **Forms**: React Hook Form + Zod validation
- **Testing**: Jest + React Testing Library + Playwright (E2E)

---

## Planned Directory Structure

```
frontend/
├── app/                      # Next.js App Router pages
│   ├── (auth)/               # Auth-related pages
│   │   ├── login/
│   │   └── signup/
│   ├── (dashboard)/          # Protected dashboard routes
│   │   ├── profile/
│   │   ├── matches/
│   │   ├── messages/
│   │   └── settings/
│   ├── (marketing)/          # Public marketing pages
│   │   ├── page.tsx          # Home page
│   │   ├── about/
│   │   └── pricing/
│   ├── layout.tsx            # Root layout
│   └── globals.css           # Global styles
├── components/              # Reusable React components
│   ├── ui/                   # Base UI components (buttons, inputs, cards)
│   ├── features/             # Feature-specific components
│   │   ├── match-card/
│   │   ├── profile-form/
│   │   └── message-thread/
│   └── layout/               # Layout components (header, footer, sidebar)
├── lib/                     # Utility functions and configurations
│   ├── api/                  # API client functions
│   ├── supabase/             # Supabase client configuration
│   ├── utils/                # Helper functions
│   └── validators/           # Zod schemas for validation
├── hooks/                   # Custom React hooks
│   ├── useAuth.ts
│   ├── useMatches.ts
│   └── useRealtime.ts
├── types/                   # TypeScript type definitions
├── public/                  # Static assets
│   ├── images/
│   ├── icons/
│   └── fonts/
├── tests/                   # Test files
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .env.example             # Example environment variables
├── .eslintrc.json           # ESLint configuration
├── .prettierrc              # Prettier configuration
├── next.config.js           # Next.js configuration
├── package.json             # Dependencies and scripts
├── tailwind.config.js       # Tailwind CSS configuration
├── tsconfig.json            # TypeScript configuration
└── README.md                # This file
```

---

## Key Features to Implement

### 1. Authentication Flow
- [ ] Login with GitHub OAuth
- [ ] Login with Google OAuth
- [ ] Session management with Supabase Auth
- [ ] Protected routes with middleware
- [ ] JWT token refresh

### 2. Onboarding Flow
- [ ] Progressive profile completion (multi-step form)
- [ ] GitHub profile import
- [ ] Skill selection and preferences
- [ ] Role and experience level input
- [ ] Timezone and location selection

### 3. Match Discovery UI
- [ ] Card-based match browsing (swipe interface)
- [ ] Grid view with filters
- [ ] Match explanation display
- [ ] Like/Pass actions
- [ ] Match score visualization

### 4. Profile Management
- [ ] View and edit profile
- [ ] Skill management (add, remove, edit)
- [ ] Preference updates
- [ ] Profile completeness indicator
- [ ] Public profile view

### 5. Messaging Interface
- [ ] Conversation list with unread counts
- [ ] Real-time chat (Supabase Realtime)
- [ ] Message composition with AI suggestions
- [ ] Typing indicators
- [ ] Online/offline status

### 6. Settings & Preferences
- [ ] Notification preferences
- [ ] Privacy settings
- [ ] Account management
- [ ] Data export (GDPR)
- [ ] Account deletion

---

## Environment Variables

Create a `.env.local` file with the following:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# Backend API
NEXT_PUBLIC_API_URL=http://localhost:4000

# ML Service
NEXT_PUBLIC_ML_SERVICE_URL=http://localhost:8000

# Analytics (optional)
NEXT_PUBLIC_GA_TRACKING_ID=your_google_analytics_id
```

---

## Installation (When Implemented)

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test

# Run E2E tests
npm run test:e2e

# Lint and format
npm run lint
npm run format
```

---

## Implementation Guide for Coding Agents

### Step 1: Project Setup
1. Initialize Next.js project with TypeScript: `npx create-next-app@latest`
2. Install dependencies:
   ```bash
   npm install @supabase/supabase-js @tanstack/react-query
   npm install @radix-ui/react-* tailwindcss
   npm install react-hook-form zod @hookform/resolvers
   ```
3. Configure Tailwind CSS and Radix UI
4. Set up Supabase client in `lib/supabase/client.ts`

### Step 2: Authentication
1. Create auth context provider (`lib/auth/AuthProvider.tsx`)
2. Implement login/signup pages with OAuth buttons
3. Add protected route middleware
4. Handle session refresh and token management

### Step 3: Core Pages
1. Build onboarding flow (refer to `specifications/onboarding-flow.md`)
2. Build match discovery UI (refer to `specifications/match-discovery-ui.md`)
3. Build messaging interface (refer to `specifications/messaging-ux.md`)
4. Build profile management pages

### Step 4: API Integration
1. Create API client functions in `lib/api/`
2. Set up React Query for data fetching and caching
3. Implement error handling and loading states
4. Add optimistic updates for better UX

### Step 5: Realtime Features
1. Set up Supabase Realtime channels
2. Subscribe to message events
3. Implement typing indicators
4. Add online/offline presence

### Step 6: Testing & Polish
1. Write unit tests for components and hooks
2. Write E2E tests for critical flows (Playwright)
3. Optimize performance (lazy loading, code splitting)
4. Add accessibility features (ARIA labels, keyboard navigation)

---

## Design Resources

Refer to these documents for design guidance:
- [Wireframes](../ui-design/wireframes.md)
- [Onboarding Flow Spec](../specifications/onboarding-flow.md)
- [Match Discovery UI Spec](../specifications/match-discovery-ui.md)
- [Messaging UX Spec](../specifications/messaging-ux.md)

---

## Related Documentation

- [System Architecture](../docs/architecture/system-architecture.md)
- [API Specification](../docs/architecture/api-specification.md)
- [Database Schema](../docs/architecture/database-schema.md)

---

**Status**: Ready for implementation by frontend coding agents. All specifications are complete.

**Questions?** Open a GitHub Discussion or issue.