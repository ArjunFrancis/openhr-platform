# UI Wireframes for OpenHR MVP

**Last updated:** November 30, 2025  
**Scope:** High-level wireframe specifications for core screens: Onboarding, Dashboard, Matches, Profile, Messaging.

---

## Onboarding Flow Screens

### 1. Welcome & Role Selection

**Purpose:** Capture intent and create first impression.

**Layout (desktop):**
- Header: Minimal logo top-left, "Already have an account? Sign in" link top-right.
- Left column (≈60% width):
  - Hero title: "Find your ideal co-founder or startup".
  - Subtitle: One-line value prop.
  - Progress indicator: Step 1 of 5, horizontal bar.
  - Form block:
    - Social login buttons (GitHub, Google, LinkedIn) stacked or in 3-column grid.
    - Divider "or continue with email".
    - Inputs: Email, Password.
    - Role selection: 3–4 selectable cards with icon, label, and short description:
      - Technical Co-founder / Business Co-founder / Developer / Startup.
    - Primary CTA: "Continue" (full-width on mobile, right-aligned on desktop).
    - Secondary: "Skip for now" text button.
- Right column (≈40% width):
  - Vertical stack of 2–3 statistic cards: "Avg. time to first match", "Verified profiles", "Active weekly users".

**Key states:**
- Error inline under fields.
- Disabled CTA until required fields and terms checkbox complete.

**Mobile adaptations:**
- Single column stacked; role cards in 2-column grid or vertical list.
- Sticky bottom CTA bar with progress and "Continue".

---

### 2. Profile Basics

**Purpose:** Collect minimal viable profile data.

**Layout:**
- Top: Step indicator (dots or numbered steps) and short description.
- Main card centered (max-width ~560px):
  - Avatar placeholder with upload button.
  - Inputs stacked:
    - Full name.
    - Headline (e.g., "Fullstack engineer building AI tools").
    - Location (text) + timezone helper.
    - Years of experience (numeric or dropdown).
  - Optional bio textarea.
  - GitHub connect strip at bottom of card:
    - Left: GitHub icon + copy "Connect to auto-fill skills and experience".
    - Right: "Connect GitHub" button, state badge when connected.
- Footer: Back / Skip / Continue buttons aligned horizontally.

---

### 3. Experience & Skills

**Purpose:** Normalize structured skills and experience.

**Layout:**
- Two-column content within a single card:
  - Left (Skills):
    - Skill search input with tag suggestions.
    - Selected skill chips with removable "x" and small proficiency indicator bar.
    - Category clusters (Frontend, Backend, DevOps, AI/ML) as pill buttons.
  - Right (Experience):
    - Resume upload dropzone with file name + status.
    - Collapsed timeline preview (company, role, dates) with edit icons.
- Progress microcopy under card: e.g., "Adding 3+ skills improves match quality by 40%".

---

### 4. Preferences & Matching

**Purpose:** Define what the user wants from matches.

**Layout:**
- Single wide card:
  - Grid of multi-select chips: "Looking for" (technical co-founder, founding engineer role, etc.).
  - Sliders: Hours per week, equity vs salary preference.
  - Selects: Location preference, company stage, industries.
  - Small note: "These only influence recommendations, not hard filters".
- Primary CTA: "See my first matches".

---

### 5. Completion & First Matches Preview

**Purpose:** Show immediate value and transition into product.

**Layout:**
- Top: Confetti/celebration icon, "You're in" title.
- Profile completeness ring (circular progress) with percentage.
- 2–3 horizontal match cards preview (mini version of Match Card): avatar, role, score.
- Actions:
  - Primary: "Go to dashboard".
  - Secondary: "Review my profile".

---

## Dashboard Screen

**Route:** `/dashboard`

**Purpose:** Central hub summarizing matches, profile status, and messages.

**Layout (desktop, 3-column grid):**
- Global header (sticky):
  - Left: Logo + product name.
  - Center: Global search input (profiles/skills/startups).
  - Right: Icons for notifications, theme toggle, avatar dropdown.

- Main content grid:
  - Column 1 (≈35%):
    - Card: "Profile overview" with completeness bar, checklist (3–5 items with checkboxes, e.g., "Connect GitHub").
    - Card: Quick actions (buttons) "Edit profile", "Update preferences", "Import resume".
  - Column 2 (≈40%):
    - Card: "New matches" carousel or vertical list of 3–5 match cards with key info and connect button.
  - Column 3 (≈25%):
    - Card: "Recent conversations" showing last 3 chats with unread badges.
    - Small card: "Tips" or onboarding help.

**Mobile:**
- Stacked cards in one column (Profile → Matches → Messages → Tips).
- Header collapses into logo + search icon.

---

## Matches Screen

**Route:** `/matches`

**Purpose:** Explore, filter, and act on recommended profiles.

**Layout (desktop, 2-column):**
- Left sidebar (≈28% width): Filter panel.
  - Sections with headings and controls:
    - Role (radio list or select).
    - Skills (multi-select chips + typeahead).
    - Location (select).
    - Experience level (select).
    - Match score slider (min–max).
    - Sort buttons (Relevance, Recent, Score).
  - "Reset filters" text button at bottom.

- Right content (≈72% width): Match grid/list.
  - Header row: Title ("Recommended matches"), total count, view toggle (grid/list).
  - Grid view:
    - 2–3 cards per row depending on width.
    - Each card height consistent; primary elements: avatar, name/headline, role, location, score badge, shared skills row, CTA buttons.
  - Empty state:
    - Illustration + text "No matches yet".
    - Suggestions: "Relax filters", "Update skills", "Invite your network".

**Mobile:**
- Filters in slide-over drawer triggered by "Filters" button.
- Cards in single column; sort dropdown at top.

---

## Profile Screen

**Route:** `/profile` (own) and `/profile/[id]` (other)

**Purpose:** Show a rich, scannable profile with compatibility context.

**Layout (desktop):**
- Header region:
  - Left: Large avatar, name, headline, role.
  - Middle: Location, years of experience, industry tags.
  - Right: Match score badge (for other profiles) and primary CTA ("Connect" / "Message").

- Body split into vertical sections:
  1. About
     - Paragraph text; limited width for readability.
  2. Skills
     - Skill chips grouped by category with optional proficiency bars.
  3. Experience
     - Vertical timeline: company, role, duration, bullet summary stub.
  4. Projects / GitHub
     - List of 3–5 highlighted repos or projects with language, stars, link icon.
  5. Preferences & Availability
     - Compact cards summarizing availability, time zone, collaboration preferences.

- Editing (own profile):
  - Inline "Edit" icon per section → modal or in-place editable fields.
  - "Preview public view" toggle in header.

**Mobile:**
- Sections stacked; sticky bottom bar with "Edit" (own) or "Connect" (other).

---

## Messaging Screen

**Route:** `/messages`

**Purpose:** Enable real-time conversations between matched users.

**Layout (desktop, 3-pane):**
- Left sidebar (~22% width): Conversation list.
  - Header: "Messages" title + new conversation icon.
  - Search input for conversations.
  - Scrollable list items showing avatar, name, last message preview, time, and unread badge.

- Middle pane (~53% width): Conversation thread.
  - Header: Participant avatar + name + role + small match score.
  - Messages area:
    - Bubbles aligned left/right based on author.
    - Time stamps in small, light text.
    - Grouping by sender with compact avatar.
  - Bottom composer:
    - Row with attach button, multiline text area, and circular send button.
    - Optional chip row above with AI-suggested openers.

- Right pane (~25% width): Context panel.
  - "Match summary": shared skills, compatibility notes.
  - "Profile preview" link.
  - Conversation actions: mute, archive, report.

**Mobile:**
- Two main states:
  - Conversation list full-screen.
  - Conversation thread full-screen with back arrow to list.
- Context panel collapses into bottom sheet accessible from info icon.

---

## Global Patterns & Components

### Navigation

- Primary nav (desktop): top horizontal bar with links: Dashboard, Matches, Messages, Profile.
- Secondary nav elements: Notifications bell, theme toggle, user menu.

### Feedback & States

- Loading: skeleton placeholders for cards and lists.
- Errors: inline messages near controls + non-intrusive toast.
- Success: subtle toasts (e.g., "Message sent", "Profile updated").

### Accessibility Considerations

- All CTAs distinguishable via color and icon/position.
- Min 44×44 px targets for tap interaction.
- Logical heading hierarchy per screen (H1 → H2 → H3).

These wireframe specs define the structure and hierarchy for key OpenHR screens and can be used to produce low- and high-fidelity designs in Figma while keeping implementation in Next.js + Tailwind straightforward.