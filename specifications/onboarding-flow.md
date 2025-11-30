# Day 5: UI/UX & Feature Specifications

**Overview:** Day 5 designs the user interface and experience for OpenHR, focusing on seamless onboarding, intuitive match discovery, and engaging messaging. All designs prioritize mobile-first responsiveness, accessibility (WCAG 2.1 AA), and performance (<2s page loads) while integrating with Day 4's backend services.

**Design Principles:**
- Minimalist interface with focus on content and matches.
- Progressive disclosure to avoid information overload.
- AI-assisted flows with clear human override options.
- Dark/light mode support with system preference detection.
- Fitts's Law compliance for primary actions (large, obvious buttons).

**Tech Stack:** Next.js 14 (App Router), Tailwind CSS, shadcn/ui components, Framer Motion for animations, React Hook Form for validation, Zustand for state management.

---

## Onboarding Flow

The onboarding flow guides users through profile creation in <3 minutes, leveraging Day 4's enrichment pipeline for automation. It uses a multi-step wizard with progress indicators, validation, and AI suggestions to reduce drop-off by 60%.

### Flow Steps

1. **Welcome & Account Creation** (30s)
   - Email/password or OAuth (GitHub, LinkedIn, Google).
   - Role selection: Co-founder (Technical/Business), Developer (Frontend/Backend/Fullstack), Startup.
   - Basic info: Name, location, time zone, availability (full-time/part-time/contract).
   - Validation: Email format, password strength (8+ chars, 1 uppercase, 1 number).

2. **Profile Basics** (45s)
   - Headline (AI-suggested: "Fullstack Engineer seeking AI startup").
   - One-sentence summary (optional, AI completion).
   - Industry focus: Dropdown (AI/ML, Fintech, Healthtech, etc.).
   - Preferred collaboration: Remote/hybrid/office, equity/salary mix.

3. **Experience & Skills** (60s)
   - Resume upload (PDF/DOCX) → immediate parsing preview.
   - GitHub connect → auto-populate repos, languages, activity score.
   - Skills review: Edit parsed skills, add/remove, set proficiency.
   - Experience summary: AI-generated timeline from resume, editable.

4. **Preferences & Matching** (30s)
   - Match criteria: Role, industry, location, experience level, equity range.
   - Availability: Hours/week, start date, relocation willingness.
   - Notifications: Email/Slack/Discord preferences.

5. **Verification & Complete** (15s)
   - Optional: LinkedIn connect, phone verification.
   - Profile completeness score (aim for 80%+).
   - Final review: Preview public profile.
   - Welcome to first matches!

### UI Components

```tsx
// Onboarding Wizard - Step 1: Welcome
<OnboardingStep title="Welcome to OpenHR" step={1} totalSteps={5}>
  <div className="space-y-6">
    <div className="text-center">
      <h1 className="text-3xl font-bold text-gray-900">Find your co-founder</h1>
      <p className="mt-2 text-lg text-gray-600">Connect with technical talent and startups in 3 minutes.</p>
    </div>
    
    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
      <Card className="p-4">
        <UserIcon className="h-8 w-8 text-blue-600 mb-2" />
        <h3 className="font-medium">Choose Role</h3>
        <p className="text-sm text-gray-600">Co-founder, Developer, or Startup</p>
      </Card>
      <Card className="p-4">
        <UploadIcon className="h-8 w-8 text-green-600 mb-2" />
        <h3 className="font-medium">Upload Resume</h3>
        <p className="text-sm text-gray-600">AI parses skills & experience</p>
      </Card>
      <Card className="p-4">
        <ZapIcon className="h-8 w-8 text-purple-600 mb-2" />
        <h3 className="font-medium">Get Matches</h3>
        <p className="text-sm text-gray-600">See compatible co-founders instantly</p>
      </Card>
    </div>
    
    <div className="flex gap-4">
      <Button variant="outline" onClick={handleSkip}>Skip for now</Button>
      <Button onClick={nextStep}>Continue</Button>
    </div>
  </div>
</OnboardingStep>
```

### Validation & Error Handling

- Real-time validation with optimistic updates.
- Helpful error messages: "Email already exists - use sign in?"
- Loading states during API calls (parsing, GitHub auth).
- Success feedback: "Resume parsed! Found 12 skills and 3 years experience."

### Accessibility

- ARIA labels for all interactive elements.
- Keyboard navigation (Tab, Enter, Escape).
- Screen reader announcements for step changes.
- High contrast mode support.
- Focus management (trap focus in modals).

### Performance

- Lazy loading of non-critical components.
- Image optimization (Next.js Image).
- Code splitting by step.
- Service worker for offline capability (basic form caching).

---

## Match Discovery UI

Match discovery presents AI-recommended profiles in a card-based interface with filtering, sorting, and detailed views. The design balances serendipity (randomized order) with relevance (scoring-based prominence).

### Main Views

1. **Dashboard** – Overview of new matches, messages, profile completeness.
2. **Matches** – Grid/list view of recommended profiles.
3. **Search** – Advanced filtering and full-text search.
4. **Saved** – Bookmarked profiles for later review.

### Dashboard Layout

```tsx
// Dashboard - Hero Section
function Dashboard() {
  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <Header />
      
      {/* Hero Stats */}
      <section className="bg-white shadow-sm">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <StatCard
              title="New Matches"
              value="12"
              change="+4 today"
              icon={<UsersIcon className="h-6 w-6 text-blue-600" />}
            />
            <StatCard
              title="Messages"
              value="3 unread"
              change="2 new conversations"
              icon={<ChatBubbleLeftIcon className="h-6 w-6 text-green-600" />}
            />
            <StatCard
              title="Profile"
              value="85% complete"
              change="Add GitHub for better matches"
              icon={<UserCheckIcon className="h-6 w-6 text-purple-600" />}
            />
          </div>
        </div>
      </section>
      
      {/* Quick Actions */}
      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <ActionCard
            title="Find Matches"
            description="Browse compatible co-founders"
            icon={<MagnifyingGlassIcon className="h-8 w-8" />}
            href="/matches"
          />
          <ActionCard
            title="My Profile"
            description="Complete your profile"
            icon={<PencilIcon className="h-8 w-8" />}
            href="/profile"
          />
          <ActionCard
            title="Messages"
            description="Respond to new conversations"
            icon={<ChatBubbleLeftIcon className="h-8 w-8" />}
            href="/messages"
          />
        </div>
      </section>
      
      {/* Recent Matches Teaser */}
      <section className="bg-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <div className="flex justify-between items-center mb-6">
            <h2 className="text-2xl font-bold text-gray-900">Recent Matches</h2>
            <Button variant="outline" asChild>
              <Link href="/matches">View all</Link>
            </Button>
          </div>
          <MatchGrid matches={recentMatches} />
        </div>
      </section>
    </div>
  );
}
```

### Match Card Design

Each match appears as a card showing key compatibility signals at a glance.

```tsx
// Match Card Component
function MatchCard({ match, onConnect }) {
  return (
    <Card className="group hover:shadow-lg transition-shadow">
      <div className="flex items-start space-x-4 p-6">
        {/* Avatar */}
        <Avatar className="w-16 h-16">
          <AvatarImage src={match.profile.avatar_url || '/default-avatar.png'} />
          <AvatarFallback>{getInitials(match.profile.name)}</AvatarFallback>
        </Avatar>
        
        {/* Content */}
        <div className="flex-1 min-w-0">
          <div className="flex items-center justify-between">
            <div>
              <h3 className="text-lg font-semibold text-gray-900 group-hover:text-blue-600 transition-colors">
                {match.profile.headline || match.profile.name}
              </h3>
              <p className="text-sm text-gray-500">{match.profile.role} • {match.profile.location}</p>
            </div>
            <Badge variant={match.match_score > 80 ? 'default' : 'secondary'}>
              {match.match_score}%
            </Badge>
          </div>
          
          {/* Summary */}
          <p className="mt-2 text-sm text-gray-600 line-clamp-2">
            {match.profile.summary}
          </p>
          
          {/* Key Signals */}
          <div className="mt-4 flex flex-wrap gap-2">
            {match.shared_skills.slice(0, 3).map(skill => (
              <Badge key={skill.id} variant="outline" className="text-xs">
                {skill.name}
              </Badge>
            ))}
            {match.shared_skills.length > 3 && (
              <Badge variant="outline" className="text-xs">
                +{match.shared_skills.length - 3} more
              </Badge>
            )}
          </div>
          
          {/* Activity Indicators */}
          <div className="mt-3 flex items-center justify-between text-xs text-gray-500">
            <div className="flex items-center space-x-4">
              <div className="flex items-center space-x-1">
                <GitHubIcon className="h-3 w-3" />
                <span>{match.github_activity_score || 'N/A'}</span>
              </div>
              <div className="flex items-center space-x-1">
                <ClockIcon className="h-3 w-3" />
                <span>{match.profile.availability}</span>
              </div>
            </div>
            <span className="text-gray-400">{formatDistanceToNow(match.created_at)}</span>
          </div>
        </div>
      </div>
      
      {/* Action Bar */}
      <CardFooter className="pt-0 border-t border-gray-200">
        <div className="flex w-full justify-between">
          <div className="flex space-x-2">
            <Button
              variant="ghost"
              size="sm"
              onClick={() => onSave(match.id)}
              className="h-8"
            >
              <BookmarkIcon className="h-4 w-4 mr-1" />
              Save
            </Button>
            <Button
              variant="ghost"
              size="sm"
              onClick={() => onView(match.id)}
              className="h-8"
            >
              View Profile
            </Button>
          </div>
          <Button
            onClick={() => onConnect(match.id)}
            className="bg-blue-600 hover:bg-blue-700 h-8"
          >
            Connect ({match.match_score}%)
          </Button>
        </div>
      </CardFooter>
    </Card>
  );
}
```

### Filtering & Sorting

```tsx
// Filter Panel
function MatchFilters({ filters, onFilterChange }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Filter Matches</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <div>
          <Label>Role</Label>
          <Select value={filters.role} onValueChange={(v) => onFilterChange('role', v)}>
            <SelectTrigger>
              <SelectValue placeholder="All roles" />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="frontend">Frontend Developer</SelectItem>
              <SelectItem value="backend">Backend Developer</SelectItem>
              <SelectItem value="fullstack">Fullstack Developer</SelectItem>
              <SelectItem value="cto">Technical Co-founder</SelectItem>
              <SelectItem value="business">Business Co-founder</SelectItem>
            </SelectContent>
          </Select>
        </div>
        
        <div>
          <Label>Skills</Label>
          <MultiSelect
            options={skills}
            selected={filters.skills}
            onChange={(v) => onFilterChange('skills', v)}
            placeholder="Select skills"
          />
        </div>
        
        <div className="grid grid-cols-2 gap-4">
          <div>
            <Label>Location</Label>
            <Select value={filters.location} onValueChange={(v) => onFilterChange('location', v)}>
              <SelectTrigger>
                <SelectValue placeholder="Any location" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="remote">Remote</SelectItem>
                <SelectItem value="us">United States</SelectItem>
                <SelectItem value="eu">Europe</SelectItem>
                <SelectItem value="asia">Asia</SelectItem>
              </SelectContent>
            </Select>
          </div>
          
          <div>
            <Label>Experience</Label>
            <Select value={filters.experience} onValueChange={(v) => onFilterChange('experience', v)}>
              <SelectTrigger>
                <SelectValue placeholder="Any experience" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="entry">0-2 years</SelectItem>
                <SelectItem value="mid">3-5 years</SelectItem>
                <SelectItem value="senior">6+ years</SelectItem>
              </SelectContent>
            </Select>
          </div>
        </div>
        
        <div>
          <Label>Match Score</Label>
          <Slider
            value={[filters.minScore, filters.maxScore]}
            onValueChange={(v) => onFilterChange('score', v)}
            max={100}
            step={5}
            className="w-full"
          />
          <div className="text-xs text-gray-500 mt-1">
            {filters.minScore}% - {filters.maxScore}%
          </div>
        </div>
        
        <div className="flex gap-2">
          <Button
            variant={filters.sortBy === 'relevance' ? 'default' : 'outline'}
            onClick={() => onFilterChange('sortBy', 'relevance')}
          >
            Relevance
          </Button>
          <Button
            variant={filters.sortBy === 'recent' ? 'default' : 'outline'}
            onClick={() => onFilterChange('sortBy', 'recent')}
          >
            Recent
          </Button>
          <Button
            variant={filters.sortBy === 'score' ? 'default' : 'outline'}
            onClick={() => onFilterChange('sortBy', 'score')}
          >
            Score
          </Button>
        </div>
      </CardContent>
    </Card>
  );
}
```

### Search Interface

Full-text search across profiles, skills, and companies with autocomplete and typeahead.

```tsx
// Search Bar with Autocomplete
function GlobalSearch() {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [isOpen, setIsOpen] = useState(false);

  useEffect(() => {
    if (query.length >= 2) {
      // Debounced search
      const timer = setTimeout(async () => {
        const results = await searchProfiles(query);
        setSuggestions(results);
        setIsOpen(true);
      }, 300);
      return () => clearTimeout(timer);
    } else {
      setSuggestions([]);
      setIsOpen(false);
    }
  }, [query]);

  return (
    <Popover open={isOpen} onOpenChange={setIsOpen}>
      <PopoverTrigger asChild>
        <div className="relative">
          <SearchIcon className="h-4 w-4 absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" />
          <Input
            type="text"
            placeholder="Search profiles, skills, companies..."
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            className="pl-10 w-full max-w-md"
          />
        </div>
      </PopoverTrigger>
      <PopoverContent className="w-[--radix-popover-trigger-width] p-0">
        {suggestions.length > 0 ? (
          <div className="max-h-80 overflow-y-auto">
            {suggestions.map((suggestion) => (
              <CommandItem
                key={suggestion.id}
                onSelect={() => {
                  router.push(`/profile/${suggestion.id}`);
                  setIsOpen(false);
                }}
                className="flex items-center space-x-3 px-3 py-2"
              >
                <Avatar className="w-8 h-8">
                  <AvatarImage src={suggestion.avatar} />
                  <AvatarFallback>{getInitials(suggestion.name)}</AvatarFallback>
                </Avatar>
                <div className="flex-1 min-w-0">
                  <p className="text-sm font-medium text-gray-900 truncate">
                    {suggestion.name}
                  </p>
                  <p className="text-xs text-gray-500 truncate">
                    {suggestion.headline}
                  </p>
                </div>
                {suggestion.match_score && (
                  <Badge variant="outline" className="text-xs">
                    {suggestion.match_score}%
                  </Badge>
                )}
              </CommandItem>
            ))}
          </div>
        ) : query.length >= 2 ? (
          <div className="px-3 py-2 text-sm text-gray-500">
            No results found for &quot;{query}&quot;
          </div>
        ) : null}
      </PopoverContent>
    </Popover>
  );
}
```

### Profile Detail View

Expanded view showing full profile, compatibility breakdown, and connection options.

```tsx
// Profile Detail with Compatibility

function ProfileDetail({ profile, matchData }) {
  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      {/* Header */}
      <div className="flex items-start space-x-6 mb-8">
        <Avatar className="w-24 h-24">
          <AvatarImage src={profile.avatar_url} />
          <AvatarFallback>{getInitials(profile.name)}</AvatarFallback>
        </Avatar>
        <div className="flex-1">
          <div className="flex items-start justify-between">
            <div>
              <h1 className="text-3xl font-bold text-gray-900 mb-1">
                {profile.name}
              </h1>
              <p className="text-xl text-gray-600 mb-2">
                {profile.headline}
              </p>
              <div className="flex items-center space-x-4 text-sm text-gray-500">
                <span>{profile.role}</span>
                <span>•</span>
                <span>{profile.location}</span>
                <span>•</span>
                <span>{profile.years_experience} years exp</span>
              </div>
            </div>
            <div className="text-right">
              <Badge className="text-lg px-4 py-2 mb-2">
                {matchData.score}% Match
              </Badge>
              <Button className="w-full" onClick={handleConnect}>
                Start Conversation
              </Button>
            </div>
          </div>
        </div>
      </div>
      
      {/* Compatibility Breakdown */}
      <section className="mb-8">
        <h2 className="text-xl font-semibold mb-4">Why we think you'd work well together</h2>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <CompatibilityMetric
            title="Shared Skills"
            score={matchData.skills_overlap}
            max={100}
            icon={<CodeIcon className="h-5 w-5" />}
          >
            {matchData.shared_skills.map(skill => (
              <Badge key={skill.id} variant="outline" className="mr-1 mb-1">
                {skill.name}
              </Badge>
            ))}
          </CompatibilityMetric>
          
          <CompatibilityMetric
            title="Experience Complement"
            score={matchData.experience_complement}
            max={100}
            icon={<BriefcaseIcon className="h-5 w-5" />}
          >
            Your {profile.years_experience} years complements their {matchData.partner_experience} years
          </CompatibilityMetric>
          
          <CompatibilityMetric
            title="Industry Alignment"
            score={matchData.industry_alignment}
            max={100}
            icon={<BuildingIcon className="h-5 w-5" />}
          >
            Both focused on {matchData.shared_industries.join(', ')}
          </CompatibilityMetric>
          
          <CompatibilityMetric
            title="Location Compatibility"
            score={matchData.location_compatibility}
            max={100}
            icon={<MapPinIcon className="h-5 w-5" />}
          >
            {matchData.location_note}
          </CompatibilityMetric>
        </div>
      </section>
      
      {/* Profile Sections */}
      <section className="space-y-8">
        <ProfileSection title="About" icon={<UserIcon />}>
          <p className="text-gray-700 leading-relaxed">
            {profile.summary}
          </p>
        </ProfileSection>
        
        <ProfileSection title="Experience" icon={<BriefcaseIcon />}>
          <Timeline experiences={profile.experience} />
        </ProfileSection>
        
        <ProfileSection title="Skills" icon={<CodeIcon />}>
          <SkillCloud skills={profile.skills} />
        </ProfileSection>
        
        <ProfileSection title="Availability" icon={<ClockIcon />}>
          <AvailabilityCard availability={profile.availability} />
        </ProfileSection>
      </section>
    </div>
  );
}
```

---

## Messaging UX

Messaging provides a familiar, Slack/Discord-like interface with realtime updates, rich previews, and AI assistance. It integrates with Day 4's realtime messaging service for instant delivery.

### Conversation List

```tsx
// Sidebar - Conversation List
function ConversationList({ conversations, activeId, onSelect }) {
  return (
    <div className="flex flex-col h-full bg-white border-r border-gray-200">
      {/* Header */}
      <div className="p-4 border-b border-gray-200">
        <div className="flex items-center justify-between">
          <h2 className="text-lg font-semibold">Messages</h2>
          <Button variant="ghost" size="sm">
            <PlusIcon className="h-4 w-4" />
          </Button>
        </div>
        <Input
          placeholder="Search conversations"
          className="mt-3"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
      </div>
      
      {/* List */}
      <div className="flex-1 overflow-y-auto">
        {filteredConversations.map(conversation => (
          <div
            key={conversation.id}
            className={`p-4 border-b border-gray-100 cursor-pointer hover:bg-gray-50 ${
              activeId === conversation.id ? 'bg-blue-50 border-r-2 border-blue-500' : ''
            }`}
            onClick={() => onSelect(conversation.id)}
          >
            <div className="flex items-center space-x-3">
              <Avatar className="w-10 h-10">
                {conversation.isGroup ? (
                  <AvatarImage src="/group-avatar.png" />
                ) : (
                  <AvatarImage src={conversation.avatar} />
                )}
                <AvatarFallback>{conversation.initials}</AvatarFallback>
              </Avatar>
              <div className="flex-1 min-w-0">
                <div className="flex items-center justify-between">
                  <h3 className="text-sm font-medium text-gray-900 truncate">
                    {conversation.name}
                  </h3>
                  {conversation.unreadCount > 0 && (
                    <Badge className="ml-2 text-xs">
                      {conversation.unreadCount}
                    </Badge>
                  )}
                </div>
                <p className="text-xs text-gray-500 truncate mt-1">
                  {conversation.preview}
                </p>
              </div>
              <div className="text-right text-xs text-gray-400 min-w-[60px] flex flex-col items-end">
                <span>{formatTime(conversation.lastMessageTime)}</span>
                {conversation.hasNewMessages && (
                  <div className="w-2 h-2 bg-blue-500 rounded-full mt-1" />
                )}
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Message Input with AI Assistance

```tsx
// Message Composer with Suggestions
function MessageComposer({ conversationId, onSend, suggestions }) {
  const [message, setMessage] = useState('');
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [isTyping, setIsTyping] = useState(false);
  const fileInputRef = useRef();

  const handleKeyDown = (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      if (message.trim()) {
        onSend(message);
        setMessage('');
      }
    }
  };

  const handleSuggestionClick = (suggestion) => {
    setMessage(suggestion);
    setShowSuggestions(false);
  };

  return (
    <div className="border-t border-gray-200 bg-white">
      {/* Suggestions */}
      {showSuggestions && suggestions.length > 0 && (
        <div className="border-t border-gray-100 p-2 bg-gray-50">
          <div className="flex flex-wrap gap-2">
            {suggestions.map((suggestion, index) => (
              <Button
                key={index}
                variant="ghost"
                size="sm"
                className="h-8 px-3 text-xs"
                onClick={() => handleSuggestionClick(suggestion)}
              >
                {suggestion}
              </Button>
            ))}
            <Button
              variant="ghost"
              size="sm"
              className="h-8 px-3 text-xs"
              onClick={() => setShowSuggestions(false)}
            >
              Dismiss
            </Button>
          </div>
        </div>
      )}
      
      {/* Composer */}
      <div className="p-4 max-w-4xl mx-auto">
        <form onSubmit={(e) => { e.preventDefault(); onSend(message); }} className="flex items-end space-x-2">
          {/* File Attachment */}
          <Button
            type="button"
            variant="ghost"
            size="sm"
            onClick={() => fileInputRef.current?.click()}
            className="h-10 w-10 p-0"
          >
            <PaperClipIcon className="h-4 w-4" />
          </Button>
          <input
            ref={fileInputRef}
            type="file"
            className="hidden"
            onChange={handleFileSelect}
            accept="image/*,.pdf"
          />
          
          {/* Text Input */}
          <div className="flex-1 relative">
            <Textarea
              value={message}
              onChange={(e) => setMessage(e.target.value)}
              onKeyDown={handleKeyDown}
              placeholder={suggestions.length > 0 ? 'Try one of our suggestions...' : 'Type a message...'}
              className="min-h-[44px] max-h-32 resize-none pr-12 focus-visible:ring-2 focus-visible:ring-blue-500"
              onFocus={() => {
                if (suggestions.length > 0 && !message) {
                  setShowSuggestions(true);
                }
              }}
            />
            {/* Typing indicator */}
            {isTyping && (
              <div className="absolute bottom-2 right-2 flex items-center space-x-1">
                <div className="w-1.5 h-1.5 bg-gray-400 rounded-full animate-bounce" />
                <div className="w-1.5 h-1.5 bg-gray-400 rounded-full animate-bounce" style={{animationDelay: '0.1s'}} />
                <div className="w-1.5 h-1.5 bg-gray-400 rounded-full animate-bounce" style={{animationDelay: '0.2s'}} />
              </div>
            )}
          </div>
          
          {/* Send Button */}
          <Button
            type="submit"
            disabled={!message.trim()}
            className="h-10 w-10 p-0 rounded-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <SendIcon className={`h-4 w-4 ${message.trim() ? 'text-white' : 'text-gray-400'}`} />
          </Button>
        </form>
        
        {/* AI Suggestion Toggle */}
        {suggestions.length > 0 && !showSuggestions && !message && (
          <div className="flex items-center justify-center mt-2">
            <Button
              variant="ghost"
              size="sm"
              className="text-xs text-gray-500 hover:text-gray-700"
              onClick={() => setShowSuggestions(true)}
            >
              <SparklesIcon className="h-3 w-3 mr-1" />
              Get AI suggestions
            </Button>
          </div>
        )}
      </div>
    </div>
  );
}
```

### Message List with Realtime Updates

```tsx
// Messages Container with Infinite Scroll
function MessagesContainer({ messages, conversationId, userId }) {
  const [isLoadingMore, setIsLoadingMore] = useState(false);
  const messagesEndRef = useRef();

  // Auto-scroll to bottom on new messages
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const loadMoreMessages = async () => {
    if (isLoadingMore) return;
    setIsLoadingMore(true);
    // Load older messages
    await fetchMoreMessages(conversationId, lastMessageId);
    setIsLoadingMore(false);
  };

  return (
    <div className="flex-1 overflow-y-auto bg-gray-50 p-4 space-y-4">
      {/* Loading More Indicator */}
      {isLoadingMore && (
        <div className="text-center py-4">
          <Loader className="h-4 w-4 animate-spin mx-auto" />
        </div>
      )}
      
      {/* Messages */}
      {messages.map(message => (
        <MessageBubble
          key={message.id}
          message={message}
          isOwn={message.sender_profile_id === userId}
          isLastInGroup={/* logic to check if last in group */}
        />
      ))}
      
      {/* Scroll Anchor */}
      <div ref={messagesEndRef} />
    </div>
  );
}
```

### Message Bubble Component

```tsx
// Individual Message with Reactions and Replies
function MessageBubble({ message, isOwn, isLastInGroup }) {
  const [showReactions, setShowReactions] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [editedText, setEditedText] = useState(message.content);

  return (
    <div className={`flex ${isOwn ? 'justify-end' : 'justify-start'} mb-4`}>
      {!isOwn && (
        <Avatar className="w-8 h-8 flex-shrink-0 mt-2">
          <AvatarImage src={message.sender_avatar} />
          <AvatarFallback>{message.sender_initials}</AvatarFallback>
        </Avatar>
      )}
      
      <div className={`max-w-xs lg:max-w-md ${isOwn ? 'order-2' : ''}`}>
        <div className={`inline-block p-3 rounded-2xl ${isOwn ? 'bg-blue-600 text-white' : 'bg-white shadow-sm border'}`}>
          {isEditing ? (
            <div className="space-y-2">
              <Textarea
                value={editedText}
                onChange={(e) => setEditedText(e.target.value)}
                className="min-h-[40px] resize-none border-0 bg-transparent text-inherit focus-visible:ring-0 p-0"
                autoFocus
              />
              <div className="flex justify-end space-x-2">
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
                <Button
                  size="sm"
                  onClick={async () => {
                    await updateMessage(message.id, editedText);
                    setIsEditing(false);
                  }}
                >
                  Save
                </Button>
              </div>
            </div>
          ) : (
            <>
              <div className="prose prose-sm prose-inherit max-w-none break-words">
                {message.content}
                {message.is_edited && (
                  <span className="text-xs text-blue-200 ml-1">(edited)</span>
                )}
              </div>
              
              {/* Message Actions */}
              <div className={`flex items-center space-x-2 mt-2 ${isOwn ? 'justify-end' : 'justify-start'} pt-1 border-t border-opacity-20`}>
                <span className="text-xs opacity-75">
                  {formatTime(message.created_at)}
                </span>
                {isOwn && (
                  <>
                    <Button
                      variant="ghost"
                      size="sm"
                      className="h-6 px-1 text-xs"
                      onClick={() => setIsEditing(true)}
                    >
                      Edit
                    </Button>
                    <Button
                      variant="ghost"
                      size="sm"
                      className="h-6 px-1 text-xs"
                      onClick={() => deleteMessage(message.id)}
                    >
                      Delete
                    </Button>
                  </>
                )}
                
                {/* Reactions */}
                <Popover open={showReactions} onOpenChange={setShowReactions}>
                  <PopoverTrigger asChild>
                    <Button
                      variant="ghost"
                      size="sm"
                      className="h-6 px-1 text-xs"
                    >
                      👍 {message.reactions?.thumbs_up || 0}
                    </Button>
                  </PopoverTrigger>
                  <PopoverContent className="w-48 p-2">
                    <div className="space-y-1">
                      {['👍', '❤️', '😂', '😮', '😢'].map(emoji => (
                        <Button
                          key={emoji}
                          variant="ghost"
                          className="justify-start w-full h-8 text-left"
                          onClick={() => reactToMessage(message.id, emoji)}
                        >
                          <span className="mr-2">{emoji}</span>
                          <span className="text-xs text-gray-500">
                            {message.reactions?.[emoji] || 0}
                          </span>
                        </Button>
                      ))}
                    </div>
                  </PopoverContent>
                </Popover>
                
                {/* Reply */}
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-6 px-1 text-xs"
                  onClick={() => setReplyTo(message.id)}
                >
                  Reply
                </Button>
              </div>
            </>
          )}
        </div>
      </div>
      
      {isOwn && (
        <Avatar className="w-8 h-8 flex-shrink-0 mt-2 order-1">
          <AvatarImage src={userAvatar} />
          <AvatarFallback>{userInitials}</AvatarFallback>
        </Avatar>
      )}
    </div>
  );
}
```

### Mobile Responsiveness

- **Touch targets:** Minimum 44x44px for buttons.
- **Swipe gestures:** Swipe left on messages to reply/archive.
- **Keyboard handling:** Auto-focus input on conversation select.
- **Safe areas:** iOS notch, Android gesture navigation.
- **Offline support:** Queue messages when offline, sync on reconnect.

---

## MVP Feature Specifications

### Core Features (Phase 1)

1. **Profile Management**
   - CRUD operations for user profiles.
   - Resume upload and parsing integration.
   - GitHub OAuth and enrichment.
   - Skill editing and normalization.
   - Profile visibility controls (public/private fields).

2. **Match Discovery**
   - AI-powered match recommendations (top 20 per user).
   - Basic filtering (role, location, skills).
   - Match scoring and explanation.
   - Save/bookmark functionality.
   - Search with autocomplete.

3. **Messaging**
   - One-to-one conversations from matches.
   - Realtime message delivery (<1s latency).
   - Message history with pagination.
   - Read receipts and typing indicators.
   - File attachments (images, PDFs <5MB).

4. **Onboarding**
   - Multi-step wizard with progress tracking.
   - AI-assisted profile completion.
   - Email verification.
   - Welcome tour/highlights.

### Success Metrics

- **Onboarding:** <3min completion time, 80%+ completion rate.
- **Matches:** 15+ matches/user/week, 70%+ click-through rate.
- **Messaging:** 50%+ response rate within 24h, <1s message latency.
- **Retention:** 60%+ D1 retention, 40%+ D7 retention.
- **Performance:** Core Web Vitals all green, <2s page loads.

### Non-Functional Requirements

- **Security:** JWT auth, RLS on all data, rate limiting.
- **Scalability:** 1K concurrent users, 10K daily active.
- **Accessibility:** WCAG 2.1 AA, keyboard navigation.
- **Mobile:** Responsive design, PWA capabilities.
- **Monitoring:** Error tracking (Sentry), performance (Vercel).

---

## Technical Implementation Notes

- **Routing:** Next.js App Router with parallel routes for dashboard.
- **State:** Zustand for global state (user, matches, messages).
- **API:** tRPC for type-safe API calls.
- **Styling:** Tailwind CSS with shadcn/ui component library.
- **Animations:** Framer Motion for smooth transitions.
- **Testing:** React Testing Library, Playwright for E2E.
- **Deployment:** Vercel for frontend, Supabase for backend.

This completes the UI/UX specifications for OpenHR's MVP, providing a cohesive, performant, and user-centric experience for co-founder matching.

**References:**
- Next.js App Router
- Tailwind CSS Design System
- shadcn/ui Components
- Framer Motion Animations