# Match Discovery UI for OpenHR

**Last updated:** November 30, 2025  
**Confidence:** 95%+ (Next.js + Tailwind + shadcn/ui patterns)

---

## Overview

The Match Discovery UI presents AI-recommended co-founders and developers in an intuitive, card-based interface with advanced filtering, sorting, and search capabilities. The design balances serendipity (randomized presentation) with relevance (scoring-based prominence) while maintaining high performance (<2s loads) and mobile responsiveness.

Key features include match scoring visualization, shared skill highlighting, activity indicators (GitHub integration), and seamless transitions to messaging. The interface supports 1K+ concurrent users with real-time updates via Supabase Realtime subscriptions.

---

## Design Principles

- **Progressive Disclosure:** Show key compatibility signals upfront, reveal details on interaction.
- **Visual Hierarchy:** Match score, shared skills, and activity indicators are primary visual elements.
- **Consistency:** Design language matches onboarding and messaging (shadcn/ui + Tailwind).
- **Accessibility:** WCAG 2.1 AA, keyboard navigation, screen reader support.
- **Performance:** Lazy loading, infinite scroll, optimistic updates, image optimization.

**Tech Stack:** Next.js 14 (App Router), Tailwind CSS v3, shadcn/ui, Framer Motion, React Query, Zustand.

---

## Dashboard Overview

The dashboard serves as the primary entry point, providing at-a-glance metrics and quick actions to drive engagement.

### Layout Structure

```tsx
// pages/dashboard.tsx
import { useQuery } from '@tanstack/react-query';

import { Header } from '@/components/layout/header';
import { StatCard } from '@/components/ui/stat-card';
import { ActionCard } from '@/components/ui/action-card';
import { MatchGrid } from '@/components/matches/match-grid';

function Dashboard() {
  const { data: stats } = useQuery(['user-stats'], fetchUserStats);
  const { data: recentMatches } = useQuery(['recent-matches'], () => fetchMatches({ limit: 6 }));

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Global Header */}
      <Header />
      
      {/* Hero Stats Section */}
      <section className="bg-white shadow-sm">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <StatCard
              title="New Matches"
              value={stats?.newMatches || 0}
              change={stats?.matchesToday ? `+${stats.matchesToday} today` : ''}
              icon={<UsersIcon className="h-6 w-6 text-blue-600" />}
              href="/matches"
            />
            <StatCard
              title="Messages"
              value={stats?.unreadCount || 0}
              change={stats?.newConversations ? `${stats.newConversations} new` : ''}
              icon={<ChatBubbleLeftIcon className="h-6 w-6 text-green-600" />}
              href="/messages"
            />
            <StatCard
              title="Profile"
              value={`${stats?.profileCompleteness || 0}% complete`}
              change={stats?.profileSuggestions ? stats.profileSuggestions : 'Complete your profile'}
              icon={<UserCheckIcon className="h-6 w-6 text-purple-600" />}
              href="/profile"
            />
          </div>
        </div>
      </section>
      
      {/* Quick Actions */}
      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <h2 className="text-2xl font-bold text-gray-900 mb-6">Quick Actions</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <ActionCard
            title="Find Matches"
            description="Browse compatible co-founders and developers"
            icon={<MagnifyingGlassIcon className="h-8 w-8 text-blue-600" />}
            href="/matches"
            cta="Browse Matches"
          />
          <ActionCard
            title="Update Profile"
            description="Complete your profile for better matches"
            icon={<PencilIcon className="h-8 w-8 text-green-600" />}
            href="/profile"
            cta="Edit Profile"
          />
          <ActionCard
            title="Messages"
            description="Respond to new conversations"
            icon={<ChatBubbleLeftIcon className="h-8 w-8 text-purple-600" />}
            href="/messages"
            badge={stats?.newConversations ? `${stats.newConversations} new` : undefined}
          />
        </div>
      </section>
      
      {/* Recent Matches Teaser */}
      <section className="bg-white border-t">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <div className="flex justify-between items-center mb-6">
            <div>
              <h2 className="text-2xl font-bold text-gray-900">Recent Matches</h2>
              <p className="text-gray-600">People you might want to connect with</p>
            </div>
            <Button variant="outline" asChild>
              <Link href="/matches">View all matches</Link>
            </Button>
          </div>
          
          {recentMatches ? (
            <MatchGrid matches={recentMatches} columns={3} />
          ) : (
            <div className="text-center py-12">
              <UsersIcon className="h-12 w-12 text-gray-400 mx-auto mb-4" />
              <h3 className="text-lg font-medium text-gray-900 mb-2">No matches yet</h3>
              <p className="text-gray-500 mb-4">Complete your profile to see compatible co-founders</p>
              <Button asChild>
                <Link href="/profile">Complete Profile</Link>
              </Button>
            </div>
          )}
        </div>
      </section>
    </div>
  );
}
```

### StatCard Component

```tsx
// components/ui/stat-card.tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { cn } from '@/lib/utils';

import { LucideIcon } from 'lucide-react';

interface StatCardProps {
  title: string;
  value: string | number;
  change?: string;
  icon: LucideIcon;
  href?: string;
}

export function StatCard({ title, value, change, icon: Icon, href }: StatCardProps) {
  return (
    <Card asChild={!!href}>
      <Link href={href} className="block hover:shadow-md transition-shadow">
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium text-gray-900">
            {title}
          </CardTitle>
          <Icon className="h-5 w-5 text-gray-400" />
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold text-gray-900">
            {value}
          </div>
          {change && (
            <p className="text-xs text-gray-600 mt-1">
              {change}
            </p>
          )}
        </CardContent>
      </Link>
    </Card>
  );
}
```

---

## Matches Page

The main matches interface with filtering, sorting, and infinite scroll. Users can toggle between grid and list views.

### Layout with Filters

```tsx
// pages/matches.tsx
import { useState } from 'react';
import { useInfiniteQuery } from '@tanstack/react-query';

import { MatchFilters } from '@/components/matches/match-filters';
import { MatchGrid } from '@/components/matches/match-grid';
import { MatchList } from '@/components/matches/match-list';
import { ViewToggle } from '@/components/ui/view-toggle';

function MatchesPage() {
  const [filters, setFilters] = useState({
    role: '',
    skills: [],
    location: 'remote',
    experience: '',
    minScore: 70,
    maxScore: 100,
    sortBy: 'relevance',
    view: 'grid' as 'grid' | 'list'
  });

  const {
    data: matchesData,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage
  } = useInfiniteQuery({
    queryKey: ['matches', filters],
    queryFn: ({ pageParam = 0 }) => fetchMatches({
      ...filters,
      page: pageParam,
      limit: 20
    }),
    getNextPageParam: (lastPage, pages) => 
      lastPage.hasMore ? pages.length : undefined,
    initialPageParam: 0
  });

  const matches = matchesData?.pages.flatMap(page => page.matches) || [];

  return (
    <div className="min-h-screen bg-gray-50">
      <Header />
      
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {/* Page Header */}
        <div className="flex flex-col lg:flex-row lg:items-center lg:justify-between mb-8">
          <div>
            <h1 className="text-3xl font-bold text-gray-900">Matches</h1>
            <p className="text-gray-600 mt-2">
              {matches.length} compatible co-founders and developers
            </p>
          </div>
          
          <div className="flex items-center space-x-4 mt-4 lg:mt-0">
            <ViewToggle
              view={filters.view}
              onChange={(view) => setFilters(prev => ({ ...prev, view }))}
            />
            <Button size="sm" onClick={() => fetchMatches({ ...filters, refresh: true })}>
              <RefreshCwIcon className="h-4 w-4 mr-2" />
              Refresh
            </Button>
          </div>
        </div>
        
        {/* Filters and Results */}
        <div className="grid grid-cols-1 lg:grid-cols-4 gap-8">
          {/* Filters Sidebar */}
          <div className="lg:col-span-1">
            <MatchFilters
              filters={filters}
              onFilterChange={(key, value) => 
                setFilters(prev => ({ ...prev, [key]: value }))}
            />
          </div>
          
          {/* Matches Content */}
          <div className="lg:col-span-3">
            {filters.view === 'grid' ? (
              <MatchGrid
                matches={matches}
                onConnect={handleConnect}
                onSave={handleSave}
                onView={handleView}
                isLoading={isFetchingNextPage}
                onLoadMore={() => fetchNextPage()}
                hasMore={hasNextPage}
              />
            ) : (
              <MatchList
                matches={matches}
                onConnect={handleConnect}
                onSave={handleSave}
                onView={handleView}
                isLoading={isFetchingNextPage}
                onLoadMore={() => fetchNextPage()}
                hasMore={hasNextPage}
              />
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```

### Match Grid View

```tsx
// components/matches/match-grid.tsx
import { motion } from 'framer-motion';
import { useInView } from 'react-intersection-observer';

import { MatchCard } from './match-card';
import { LoadMoreButton } from '@/components/ui/load-more-button';

interface MatchGridProps {
  matches: Match[];
  onConnect: (id: string) => void;
  onSave: (id: string) => void;
  onView: (id: string) => void;
  isLoading: boolean;
  hasMore: boolean;
  onLoadMore: () => void;
}

export function MatchGrid({ matches, onConnect, onSave, onView, isLoading, hasMore, onLoadMore }: MatchGridProps) {
  const { ref, inView } = useInView({
    threshold: 0,
    onChange: (inView) => inView && hasMore && !isLoading && onLoadMore()
  });

  return (
    <div className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
        {matches.map((match, index) => (
          <motion.div
            key={match.id}
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ duration: 0.3, delay: index * 0.05 }}
          >
            <MatchCard
              match={match}
              onConnect={() => onConnect(match.id)}
              onSave={() => onSave(match.id)}
              onView={() => onView(match.id)}
            />
          </motion.div>
        ))}
      </div>
      
      {/* Load More */}
      {hasMore && (
        <div ref={ref} className="flex justify-center py-8">
          {isLoading ? (
            <div className="flex items-center space-x-2 text-gray-500">
              <Loader2 className="h-4 w-4 animate-spin" />
              <span>Loading more matches...</span>
            </div>
          ) : null}
        </div>
      )}
      
      {!hasMore && matches.length > 0 && (
        <div className="text-center py-12">
          <CheckCircleIcon className="h-12 w-12 text-green-500 mx-auto mb-4" />
          <h3 className="text-lg font-medium text-gray-900 mb-2">You've seen all matches</h3>
          <p className="text-gray-500 mb-4">Try adjusting your filters or update your profile for more</p>
          <div className="flex flex-col sm:flex-row gap-3 justify-center">
            <Button variant="outline" onClick={() => resetFilters()}>
              Clear Filters
            </Button>
            <Button asChild>
              <Link href="/profile">Update Profile</Link>
            </Button>
          </div>
        </div>
      )}
      
      {matches.length === 0 && !isLoading && (
        <div className="text-center py-12">
          <UsersIcon className="h-12 w-12 text-gray-400 mx-auto mb-4" />
          <h3 className="text-lg font-medium text-gray-900 mb-2">No matches found</h3>
          <p className="text-gray-500 mb-4">
            Try adjusting your filters or complete your profile for better results
          </p>
          <div className="flex flex-col sm:flex-row gap-3 justify-center">
            <Button variant="outline" onClick={() => resetFilters()}>
              Clear Filters
            </Button>
            <Button asChild>
              <Link href="/profile">Complete Profile</Link>
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### MatchCard Component

```tsx
// components/matches/match-card.tsx
import { Link } from 'next/link';
import { motion } from 'framer-motion';
import { toast } from 'sonner';

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter } from '@/components/ui/card';
import { BookmarkIcon, EyeIcon, MessageCircle, UsersIcon } from 'lucide-react';

import { cn } from '@/lib/utils';
import { getInitials } from '@/lib/utils';

interface MatchCardProps {
  match: Match;
  onConnect: () => void;
  onSave: () => void;
  onView: () => void;
}

export function MatchCard({ match, onConnect, onSave, onView }: MatchCardProps) {
  const [isSaved, setIsSaved] = useState(match.isSaved);
  const [isConnecting, setIsConnecting] = useState(false);

  const handleSave = async () => {
    try {
      await saveMatch(match.id);
      setIsSaved(true);
      toast.success('Match saved!');
    } catch (error) {
      toast.error('Failed to save match');
    }
  };

  const handleConnect = async () => {
    if (isConnecting) return;
    setIsConnecting(true);
    try {
      await initiateConversation(match.id);
      onConnect();
      toast.success(`Started conversation with ${match.profile.name}`);
    } catch (error) {
      toast.error('Failed to start conversation');
    } finally {
      setIsConnecting(false);
    }
  };

  return (
    <motion.div whileHover={{ y: -2 }} className="group">
      <Card className="overflow-hidden hover:shadow-lg transition-all duration-200 border-0 bg-white">
        {/* Header with Avatar and Basic Info */}
        <CardContent className="p-6 pb-4">
          <div className="flex items-start space-x-4">
            <Avatar className="w-16 h-16 flex-shrink-0">
              <AvatarImage src={match.profile.avatar_url} alt={match.profile.name} />
              <AvatarFallback className="bg-gradient-to-br from-blue-500 to-purple-600 text-white">
                {getInitials(match.profile.name)}
              </AvatarFallback>
            </Avatar>
            
            <div className="flex-1 min-w-0">
              <div className="flex items-start justify-between mb-2">
                <div className="flex-1">
                  <motion.h3 
                    className="text-lg font-semibold text-gray-900 group-hover:text-blue-600 transition-colors line-clamp-1"
                    whileHover={{ scale: 1.02 }}
                  >
                    {match.profile.headline || match.profile.name}
                  </motion.h3>
                  <p className="text-sm text-gray-500 mt-1 line-clamp-1">
                    {match.profile.role} • {match.profile.location} • {match.profile.years_experience} years exp
                  </p>
                </div>
                
                <motion.div 
                  className="flex-shrink-0 ml-3"
                  initial={{ scale: 0 }}
                  animate={{ scale: 1 }}
                  transition={{ delay: 0.1 }}
                >
                  <Badge 
                    variant={match.match_score >= 80 ? 'default' : 'secondary'}
                    className={cn(
                      'text-sm font-medium px-3 py-1',
                      match.match_score >= 80 ? 'bg-blue-100 text-blue-800' : 'bg-gray-100 text-gray-800'
                    )}
                  >
                    {match.match_score}%
                  </Badge>
                </motion.div>
              </div>
              
              {/* Summary */}
              <p className="text-sm text-gray-600 mt-3 line-clamp-2 leading-relaxed">
                {match.profile.summary}
              </p>
              
              {/* Shared Skills - Primary Visual Element */}
              {match.shared_skills.length > 0 && (
                <div className="mt-4 flex flex-wrap gap-1.5">
                  {match.shared_skills.slice(0, 4).map((skill) => (
                    <motion.div
                      key={skill.id}
                      whileHover={{ scale: 1.05 }}
                      className="group/skill"
                    >
                      <Badge 
                        variant="outline"
                        className="text-xs px-2 py-0.5 bg-blue-50 border-blue-200 text-blue-700 hover:bg-blue-100 transition-colors"
                      >
                        {skill.name}
                      </Badge>
                    </motion.div>
                  ))}
                  {match.shared_skills.length > 4 && (
                    <Badge variant="outline" className="text-xs px-2 py-0.5">
                      +{match.shared_skills.length - 4} more
                    </Badge>
                  )}
                </div>
              )}
              
              {/* Activity Indicators */}
              <div className="mt-4 pt-3 border-t border-gray-100">
                <div className="flex items-center justify-between text-xs text-gray-500">
                  <div className="flex items-center space-x-4">
                    {/* GitHub Activity */}
                    <div className="flex items-center space-x-1">
                      <GitHub className="h-3 w-3 text-gray-400" />
                      <span>{match.github_activity_score}/100</span>
                      {match.github_activity_score >= 80 && (
                        <span className="ml-1 text-green-600">• Active</span>
                      )}
                    </div>
                    
                    {/* Availability */}
                    <div className="flex items-center space-x-1">
                      <Clock className="h-3 w-3 text-gray-400" />
                      <span>{match.profile.availability}</span>
                    </div>
                    
                    {/* Response Rate */}
                    <div className="flex items-center space-x-1">
                      <MessageCircle className="h-3 w-3 text-gray-400" />
                      <span>{match.response_rate}% response</span>
                    </div>
                  </div>
                  
                  <div className="text-gray-400">
                    {formatDistanceToNowStrict(match.created_at, { addSuffix: true })}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </CardContent>
        
        {/* Action Footer */}
        <CardFooter className="pt-0 bg-gray-50 border-t border-gray-100 px-6 pb-6">
          <div className="flex w-full items-center justify-between">
            {/* Left Actions */}
            <div className="flex items-center space-x-2">
              <motion.div whileTap={{ scale: 0.95 }}>
                <Button
                  variant="ghost"
                  size="sm"
                  className={cn('h-9 px-3', {
                    'text-blue-600 hover:bg-blue-50 hover:text-blue-700': isSaved,
                    'text-gray-500 hover:bg-gray-100': !isSaved
                  })}
                  onClick={handleSave}
                >
                  <BookmarkIcon 
                    className={cn('h-4 w-4 mr-1', {
                      'fill-blue-600': isSaved,
                      'fill-transparent': !isSaved
                    })} 
                  />
                  {isSaved ? 'Saved' : 'Save'}
                </Button>
              </motion.div>
              
              <motion.div whileTap={{ scale: 0.95 }}>
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-9 px-3 text-gray-500 hover:bg-gray-100"
                  onClick={onView}
                >
                  <EyeIcon className="h-4 w-4 mr-1" />
                  View Profile
                </Button>
              </motion.div>
            </div>
            
            {/* Primary Action - Connect */}
            <motion.div whileHover={{ scale: 1.02 }} whileTap={{ scale: 0.98 }}>
              <Button
                onClick={handleConnect}
                disabled={isConnecting}
                className={cn(
                  'h-10 px-6 font-medium rounded-full shadow-sm',
                  'bg-gradient-to-r from-blue-600 to-blue-700 hover:from-blue-700 hover:to-blue-800',
                  'text-white hover:shadow-md transform transition-all duration-200',
                  isConnecting && 'opacity-75 cursor-not-allowed'
                )}
              >
                {isConnecting ? (
                  <>
                    <Loader2 className="h-4 w-4 mr-2 animate-spin" />
                    Connecting...
                  </>
                ) : (
                  <>
                    <MessageCircle className="h-4 w-4 mr-2" />
                    Connect ({match.match_score}%)
                  </>
                )}
              </Button>
            </motion.div>
          </div>
        </CardFooter>
      </Card>
    </motion.div>
  );
}
```

### Match List View (Alternative)

For users preferring a more compact, timeline-style view:

```tsx
// components/matches/match-list.tsx
import { motion } from 'framer-motion';

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Skeleton } from '@/components/ui/skeleton';

import { cn } from '@/lib/utils';
import { getInitials } from '@/lib/utils';

function MatchList({ matches, onConnect, onSave, onView, isLoading, hasMore, onLoadMore }) {
  return (
    <div className="space-y-4">
      {matches.map((match) => (
        <motion.div
          key={match.id}
          initial={{ opacity: 0, x: -20 }}
          animate={{ opacity: 1, x: 0 }}
          className="group"
        >
          <div className="flex items-start space-x-4 p-4 hover:bg-white rounded-lg border border-gray-200 hover:shadow-sm transition-all duration-200">
            {/* Avatar */}
            <Avatar className="w-12 h-12 flex-shrink-0 mt-1">
              <AvatarImage src={match.profile.avatar_url} />
              <AvatarFallback className="bg-blue-100 text-blue-600">
                {getInitials(match.profile.name)}
              </AvatarFallback>
            </Avatar>
            
            {/* Content */}
            <div className="flex-1 min-w-0">
              <div className="flex items-center justify-between mb-1">
                <h3 className="text-base font-semibold text-gray-900 truncate group-hover:text-blue-600 transition-colors">
                  {match.profile.headline || match.profile.name}
                </h3>
                <Badge className="ml-2 flex-shrink-0">
                  {match.match_score}%
                </Badge>
              </div>
              
              <p className="text-sm text-gray-600 mb-2 line-clamp-1">
                {match.profile.role} • {match.profile.location}
              </p>
              
              <p className="text-sm text-gray-600 mb-3 line-clamp-2">
                {match.profile.summary}
              </p>
              
              {/* Skills Row */}
              <div className="flex items-center space-x-2 mb-3">
                {match.shared_skills.slice(0, 3).map((skill) => (
                  <Badge key={skill.id} variant="outline" className="text-xs">
                    {skill.name}
                  </Badge>
                ))}
                {match.shared_skills.length > 3 && (
                  <Badge variant="outline" className="text-xs">
                    +{match.shared_skills.length - 3}
                  </Badge>
                )}
              </div>
              
              {/* Meta Info */}
              <div className="flex items-center justify-between text-xs text-gray-500 mb-3">
                <div className="flex items-center space-x-4">
                  <span className="flex items-center space-x-1">
                    <GitHub className="h-3 w-3" />
                    <span>{match.github_activity_score}/100</span>
                  </span>
                  <span>{match.profile.availability}</span>
                </div>
                <span>{formatDistanceToNow(match.created_at)}</span>
              </div>
              
              {/* Action Buttons - Compact */}
              <div className="flex items-center space-x-2">
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-8 px-2 text-xs"
                  onClick={() => onSave(match.id)}
                >
                  <BookmarkIcon className="h-3 w-3 mr-1" />
                  Save
                </Button>
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-8 px-2 text-xs"
                  onClick={() => onView(match.id)}
                >
                  View
                </Button>
                <Button
                  size="sm"
                  className="h-8 px-3 ml-auto"
                  onClick={() => onConnect(match.id)}
                >
                  Connect
                </Button>
              </div>
            </div>
          </div>
        </motion.div>
      ))}
      
      {/* Loading More */}
      {isLoading && hasMore && (
        <div className="flex justify-center py-8">
          <div className="flex items-center space-x-2 text-gray-500">
            <Loader2 className="h-4 w-4 animate-spin" />
            <span>Loading more matches...</span>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Advanced Filtering System

Comprehensive filtering with real-time updates and saved filter sets.

### Filter Panel Component

```tsx
// components/matches/match-filters.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Slider } from '@/components/ui/slider';
import { Badge } from '@/components/ui/badge';
import { MultiSelect } from '@/components/ui/multi-select';

import { skillsQuery } from '@/lib/queries';

interface MatchFiltersProps {
  filters: MatchFilters;
  onFilterChange: (key: string, value: any) => void;
}

export function MatchFilters({ filters, onFilterChange }: MatchFiltersProps) {
  const { data: allSkills } = useQuery(skillsQuery);
  const [searchSkills, setSearchSkills] = useState('');
  const filteredSkills = allSkills?.filter(skill => 
    skill.name.toLowerCase().includes(searchSkills.toLowerCase())
  ) || [];

  const roleOptions = [
    { value: 'frontend', label: 'Frontend Developer' },
    { value: 'backend', label: 'Backend Developer' },
    { value: 'fullstack', label: 'Fullstack Developer' },
    { value: 'mobile', label: 'Mobile Developer' },
    { value: 'devops', label: 'DevOps Engineer' },
    { value: 'data', label: 'Data Engineer' },
    { value: 'cto', label: 'Technical Co-founder' },
    { value: 'business', label: 'Business Co-founder' },
    { value: 'startup', label: 'Startup Founder' }
  ];

  const locationOptions = [
    { value: 'remote', label: 'Remote' },
    { value: 'us', label: 'United States' },
    { value: 'eu', label: 'Europe' },
    { value: 'asia', label: 'Asia' },
    { value: 'canada', label: 'Canada' },
    { value: 'australia', label: 'Australia / NZ' },
    { value: 'latam', label: 'Latin America' },
    { value: 'africa', label: 'Africa' }
  ];

  const experienceOptions = [
    { value: 'entry', label: '0-2 years' },
    { value: 'mid', label: '3-5 years' },
    { value: 'senior', label: '6-10 years' },
    { value: 'lead', label: '10+ years' }
  ];

  const handleClearFilters = () => {
    onFilterChange('role', '');
    onFilterChange('skills', []);
    onFilterChange('location', 'remote');
    onFilterChange('experience', '');
    onFilterChange('minScore', 70);
    onFilterChange('maxScore', 100);
    onFilterChange('sortBy', 'relevance');
  };

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-lg">Filter Matches</CardTitle>
        <p className="text-sm text-gray-600">Narrow down your search</p>
      </CardHeader>
      <CardContent className="space-y-6">
        {/* Role Filter */}
        <div className="space-y-2">
          <Label htmlFor="role">Role</Label>
          <Select 
            value={filters.role} 
            onValueChange={(value) => onFilterChange('role', value)}
          >
            <SelectTrigger id="role">
              <SelectValue placeholder="Any role" />
            </SelectTrigger>
            <SelectContent>
              {roleOptions.map(option => (
                <SelectItem key={option.value} value={option.value}>
                  {option.label}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        
        {/* Skills Filter */}
        <div className="space-y-2">
          <Label>Skills (up to 5)</Label>
          <div className="relative">
            <MultiSelect
              options={filteredSkills.slice(0, 20)}
              selected={filters.skills}
              onChange={(selected) => onFilterChange('skills', selected.slice(0, 5))}
              placeholder="Select skills"
              searchable
              searchValue={searchSkills}
              onSearchChange={setSearchSkills}
              maxSelections={5}
            />
          </div>
          {filters.skills.length > 0 && (
            <div className="flex flex-wrap gap-1 mt-2">
              {filters.skills.map(skillId => {
                const skill = allSkills?.find(s => s.id === skillId);
                return skill ? (
                  <Badge
                    key={skillId}
                    variant="secondary"
                    className="text-xs"
                    onClick={() => onFilterChange('skills', filters.skills.filter(id => id !== skillId))}
                  >
                    {skill.name} ×
                  </Badge>
                ) : null;
              })}
            </div>
          )}
        </div>
        
        {/* Location and Experience Row */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div className="space-y-2">
            <Label htmlFor="location">Location</Label>
            <Select 
              value={filters.location}
              onValueChange={(value) => onFilterChange('location', value)}
            >
              <SelectTrigger id="location">
                <SelectValue placeholder="Any location" />
              </SelectTrigger>
              <SelectContent>
                {locationOptions.map(option => (
                  <SelectItem key={option.value} value={option.value}>
                    {option.label}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>
          
          <div className="space-y-2">
            <Label htmlFor="experience">Experience Level</Label>
            <Select 
              value={filters.experience}
              onValueChange={(value) => onFilterChange('experience', value)}
            >
              <SelectTrigger id="experience">
                <SelectValue placeholder="Any experience" />
              </SelectTrigger>
              <SelectContent>
                {experienceOptions.map(option => (
                  <SelectItem key={option.value} value={option.value}>
                    {option.label}
                  </SelectItem>
                ))}
                <SelectItem value="">Any experience</SelectItem>
              </SelectContent>
            </Select>
          </div>
        </div>
        
        {/* Match Score Range */}
        <div className="space-y-2">
          <Label className="flex items-center justify-between">
            <span>Match Score</span>
            <span className="text-sm text-gray-500">
              {filters.minScore}% - {filters.maxScore}%
            </span>
          </Label>
          <Slider
            value={[filters.minScore, filters.maxScore]}
            onValueChange={([min, max]) => {
              onFilterChange('minScore', min);
              onFilterChange('maxScore', max);
            }}
            min={50}
            max={100}
            step={5}
            className="w-full"
          />
        </div>
        
        {/* Sort Options */}
        <div className="space-y-2">
          <Label>Sort by</Label>
          <div className="grid grid-cols-2 gap-2">
            {[
              { value: 'relevance', label: 'Best Match' },
              { value: 'recent', label: 'Newest' },
              { value: 'score', label: 'Match Score' },
              { value: 'activity', label: 'Most Active' },
              { value: 'response', label: 'Response Rate' }
            ].map(option => (
              <Button
                key={option.value}
                variant={filters.sortBy === option.value ? 'default' : 'outline'}
                className="h-9 justify-start"
                onClick={() => onFilterChange('sortBy', option.value)}
              >
                {option.label}
              </Button>
            ))}
          </div>
        </div>
        
        {/* Action Buttons */}
        <div className="flex flex-col sm:flex-row gap-2 pt-4 border-t">
          <Button
            variant="outline"
            className="flex-1"
            onClick={handleClearFilters}
          >
            Clear Filters
          </Button>
          <Button className="flex-1">
            Save Filter Set
          </Button>
        </div>
      </CardContent>
    </Card>
  );
}
```

### Saved Filters Management

```tsx
// Additional filter persistence
function SavedFilters({ onLoadFilterSet }) {
  const { data: savedFilters } = useQuery(['saved-filters'], fetchSavedFilters);

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-sm font-medium">Saved Filters</CardTitle>
        <p className="text-xs text-gray-500">Quick access to your saved searches</p>
      </CardHeader>
      <CardContent className="space-y-2">
        {savedFilters?.map(filterSet => (
          <Button
            key={filterSet.id}
            variant="ghost"
            className="h-auto p-3 justify-start text-left hover:bg-gray-50 w-full"
            onClick={() => onLoadFilterSet(filterSet)}
          >
            <div className="flex items-center justify-between w-full">
              <div className="flex-1 min-w-0">
                <h4 className="font-medium text-sm text-gray-900 truncate">
                  {filterSet.name}
                </h4>
                <p className="text-xs text-gray-500 truncate">
                  {filterSet.description}
                </p>
              </div>
              <Badge className="ml-2 flex-shrink-0">
                {filterSet.matchCount}
              </Badge>
            </div>
          </Button>
        )) || (
          <p className="text-sm text-gray-500 text-center py-4">
            No saved filter sets. Save your current filters to access them quickly.
          </p>
        )}
        
        <Button
          variant="outline"
          size="sm"
          className="w-full mt-2"
          onClick={() => setShowSaveModal(true)}
        >
          + Create Filter Set
        </Button>
      </CardContent>
    </Card>
  );
}
```

---

## Search Interface

Global search with autocomplete, typeahead, and rich suggestions across profiles, skills, and companies.

### Global Search Bar

```tsx
// components/search/global-search.tsx
import { useState, useEffect, useCallback } from 'react';
import { useRouter } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';

import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList } from '@/components/ui/command';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Search, X } from 'lucide-react';

import { useQuery } from '@tanstack/react-query';

interface SearchResult {
  id: string;
  type: 'profile' | 'skill' | 'company';
  name: string;
  description?: string;
  avatar?: string;
  match_score?: number;
  url: string;
}

export function GlobalSearch() {
  const router = useRouter();
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const [selectedIndex, setSelectedIndex] = useState(0);

  const debouncedSearch = useDebouncedCallback(
    (term: string) => {
      if (term.length >= 2) {
        refetch();
      }
    },
    250
  );

  const { data: searchResults, refetch, isFetching } = useQuery({
    queryKey: ['search', query],
    queryFn: () => searchGlobal(query),
    enabled: false,
    staleTime: 5 * 60 * 1000 // 5 minutes
  });

  useEffect(() => {
    debouncedSearch(query);
  }, [query, debouncedSearch]);

  useEffect(() => {
    if (searchResults?.length > 0) {
      setIsOpen(true);
    } else if (query.length < 2) {
      setIsOpen(false);
    }
  }, [searchResults, query]);

  const handleSelect = useCallback((result: SearchResult) => {
    setQuery('');
    setIsOpen(false);
    router.push(result.url);
  }, [router]);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Escape') {
      setQuery('');
      setIsOpen(false);
    } else if (e.key === 'ArrowDown') {
      e.preventDefault();
      setSelectedIndex(prev => (prev + 1) % (searchResults?.length || 1));
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      setSelectedIndex(prev => (prev - 1 + (searchResults?.length || 1)) % (searchResults?.length || 1));
    } else if (e.key === 'Enter' && !isFetching) {
      e.preventDefault();
      if (searchResults?.length) {
        handleSelect(searchResults[selectedIndex]);
      } else {
        router.push(`/search?q=${encodeURIComponent(query)}`);
      }
    }
  };

  const resultsByType = searchResults?.reduce((acc, result) => {
    if (!acc[result.type]) acc[result.type] = [];
    acc[result.type].push(result);
    return acc;
  }, {} as Record<string, SearchResult[]>);

  return (
    <Popover open={isOpen} onOpenChange={setIsOpen}>
      <PopoverTrigger asChild>
        <div className="relative w-full max-w-md">
          <div className="relative">
            <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-gray-400 pointer-events-none" />
            <Input
              value={query}
              onChange={(e) => setQuery(e.target.value)}
              onKeyDown={handleKeyDown}
              placeholder="Search profiles, skills, companies..."
              className="pl-10 pr-10 w-full h-10 bg-white border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            />
            {query && (
              <Button
                variant="ghost"
                size="sm"
                className="absolute right-1 top-1/2 transform -translate-y-1/2 h-6 w-6 p-0 rounded-full hover:bg-gray-100"
                onClick={() => {
                  setQuery('');
                  setIsOpen(false);
                }}
              >
                <X className="h-3 w-3" />
              </Button>
            )}
          </div>
          {isFetching && (
            <div className="absolute right-3 top-1/2 transform -translate-y-1/2">
              <Loader2 className="h-4 w-4 animate-spin text-gray-400" />
            </div>
          )}
        </div>
      </PopoverTrigger>
      <PopoverContent className="w-[--radix-popover-trigger-width] p-0 mt-1 border-gray-200 shadow-lg max-h-[80vh] overflow-hidden">
        {isFetching ? (
          <div className="p-4">
            <div className="flex items-center space-x-3 py-2">
              <div className="w-8 h-8 bg-gray-200 rounded-full animate-pulse" />
              <div className="space-y-2 flex-1">
                <Skeleton className="h-4 w-32" />
                <Skeleton className="h-3 w-24" />
              </div>
            </div>
          </div>
        ) : searchResults?.length ? (
          <Command>
            <CommandList className="max-h-[400px] overflow-y-auto">
              {Object.entries(resultsByType).map(([type, results]) => (
                <CommandGroup key={type} heading={type.charAt(0).toUpperCase() + type.slice(1)}>
                  {results.map((result, index) => (
                    <CommandItem
                      key={result.id}
                      onSelect={() => handleSelect(result)}
                      className={cn(
                        'flex items-center space-x-3 px-3 py-2 cursor-pointer hover:bg-gray-50 focus:bg-gray-50',
                        selectedIndex === index && 'bg-blue-50 border-r-2 border-blue-500'
                      )}
                    >
                      <Avatar className="w-8 h-8 flex-shrink-0">
                        <AvatarImage src={result.avatar} />
                        <AvatarFallback className="bg-gray-100 text-gray-600">
                          {type.charAt(0).toUpperCase()}
                        </AvatarFallback>
                      </Avatar>
                      <div className="flex-1 min-w-0">
                        <p className="text-sm font-medium text-gray-900 truncate">
                          {result.name}
                        </p>
                        <p className="text-xs text-gray-500 truncate">
                          {result.description}
                        </p>
                      </div>
                      {result.match_score && (
                        <Badge variant="outline" className="text-xs ml-2">
                          {result.match_score}%
                        </Badge>
                      )}
                    </CommandItem>
                  ))}
                </CommandGroup>
              ))}
            </CommandList>
          </Command>
        ) : query.length >= 2 ? (
          <Command>
            <CommandEmpty className="p-4 text-center">
              <Search className="h-8 w-8 text-gray-400 mx-auto mb-2" />
              <p className="text-sm text-gray-500">No results found for &quot;{query}&quot;</p>
              <p className="text-xs text-gray-400 mt-1">Try different keywords</p>
            </CommandEmpty>
          </Command>
        ) : null}
        
        {query.length >= 2 && searchResults?.length === 0 && !isFetching && (
          <div className="p-4 border-t bg-gray-50">
            <Button
              variant="outline"
              className="w-full justify-start"
              onClick={() => router.push(`/search?q=${encodeURIComponent(query)}`)}
            >
              <Search className="h-4 w-4 mr-2" />
              Search for &quot;{query}&quot;
            </Button>
          </div>
        )}
      </PopoverContent>
    </Popover>
  );
}
```

### Search Results Page

For complex searches that return many results:

```tsx
// pages/search.tsx
function SearchPage({ searchParams }: { searchParams: { q: string } }) {
  const query = searchParams.q || '';
  const [filters, setFilters] = useState({});
  const [view, setView] = useState('all');

  const { data, isLoading } = useQuery({
    queryKey: ['search-results', query, filters, view],
    queryFn: () => searchProfiles({ q: query, filters, view }),
  });

  return (
    <div className="min-h-screen bg-gray-50">
      <Header />
      <div className="max-w-7xl mx-auto px-4 py-8">
        <div className="mb-8">
          <div className="flex items-center space-x-2 mb-2">
            <h1 className="text-2xl font-bold text-gray-900">Search Results</h1>
            <Badge>{data?.total || 0} results</Badge>
          </div>
          <p className="text-gray-600">Showing results for &quot;{query}&quot;</p>
        </div>
        
        {/* Tabs for different result types */}
        <div className="border-b border-gray-200 mb-6">
          <nav className="-mb-px flex space-x-8">
            {['all', 'profiles', 'skills', 'companies'].map((tab) => (
              <Button
                key={tab}
                variant={view === tab ? 'default' : 'ghost'}
                className={cn(
                  'px-1 py-4 text-sm font-medium border-b-2',
                  view === tab 
                    ? 'border-blue-500 text-blue-600' 
                    : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'
                )}
                onClick={() => setView(tab)}
              >
                {tab.charAt(0).toUpperCase() + tab.slice(1)}
                {data?.counts?.[tab] && (
                  <span className="ml-1 text-xs bg-gray-100 rounded-full px-2 py-0.5">
                    {data.counts[tab]}
                  </span>
                )}
              </Button>
            ))}
          </nav>
        </div>
        
        {/* Results */}
        <div className="grid grid-cols-1 lg:grid-cols-4 gap-8">
          <div className="lg:col-span-1">
            {/* Filters sidebar for search */}
            <SearchFilters
              query={query}
              filters={filters}
              onFilterChange={setFilters}
            />
          </div>
          <div className="lg:col-span-3">
            {view === 'profiles' && <ProfileSearchResults results={data?.profiles || []} />}
            {view === 'skills' && <SkillSearchResults results={data?.skills || []} />}
            {/* etc. */}
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## Profile Detail View

Detailed profile view with compatibility breakdown, full experience timeline, and connection options.

### Profile Header with Match Score

```tsx
// components/profiles/profile-detail.tsx
import { motion } from 'framer-motion';
import { toast } from 'sonner';

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';
import { Separator } from '@/components/ui/separator';

import { Timeline } from '@/components/ui/timeline';
import { SkillCloud } from '@/components/ui/skill-cloud';
import { AvailabilityCard } from '@/components/ui/availability-card';
import { CompatibilityMetrics } from '@/components/profiles/compatibility-metrics';

import { cn, getInitials } from '@/lib/utils';

function ProfileDetail({ profile, matchData, onConnect }) {
  const [isConnecting, setIsConnecting] = useState(false);

  const handleConnect = async () => {
    setIsConnecting(true);
    try {
      await initiateConversation(profile.id);
      toast.success(`Started conversation with ${profile.name}`);
      onConnect?.();
    } catch (error) {
      toast.error('Failed to start conversation');
    } finally {
      setIsConnecting(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-50">
      <Header />
      <div className="max-w-4xl mx-auto px-4 py-8">
        {/* Profile Header */}
        <motion.div 
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          className="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden mb-8"
        >
          <div className="p-8">
            <div className="flex items-start space-x-6">
              <Avatar className="w-24 h-24 flex-shrink-0">
                <AvatarImage src={profile.avatar_url} alt={profile.name} />
                <AvatarFallback className="bg-gradient-to-br from-blue-500 to-purple-600 text-white text-2xl">
                  {getInitials(profile.name)}
                </AvatarFallback>
              </Avatar>
              
              <div className="flex-1 min-w-0">
                <div className="flex items-start justify-between mb-4">
                  <div className="flex-1">
                    <h1 className="text-3xl font-bold text-gray-900 mb-2">
                      {profile.name}
                    </h1>
                    <p className="text-xl text-gray-600 mb-4">
                      {profile.headline}
                    </p>
                    <div className="flex items-center space-x-6 text-sm text-gray-500">
                      <span className="flex items-center space-x-1">
                        <UsersIcon className="h-4 w-4" />
                        <span>{profile.role}</span>
                      </span>
                      <span className="flex items-center space-x-1">
                        <MapPinIcon className="h-4 w-4" />
                        <span>{profile.location}</span>
                      </span>
                      <span className="flex items-center space-x-1">
                        <BriefcaseIcon className="h-4 w-4" />
                        <span>{profile.years_experience} years experience</span>
                      </span>
                      {profile.response_rate && (
                        <span className="flex items-center space-x-1">
                          <MessageCircle className="h-4 w-4" />
                          <span>{profile.response_rate}% response rate</span>
                        </span>
                      )}
                    </div>
                  </div>
                  
                  <div className="flex flex-col items-end space-y-3">
                    <motion.div 
                      initial={{ scale: 0.8, opacity: 0 }}
                      animate={{ scale: 1, opacity: 1 }}
                      transition={{ delay: 0.2 }}
                    >
                      <Badge className="text-lg px-4 py-2 bg-gradient-to-r from-blue-500 to-blue-600 text-white shadow-lg">
                        {matchData.score}% Match
                      </Badge>
                    </motion.div>
                    
                    <motion.div 
                      initial={{ scale: 0.8, opacity: 0 }}
                      animate={{ scale: 1, opacity: 1 }}
                      transition={{ delay: 0.3 }}
                    >
                      <Button
                        size="lg"
                        onClick={handleConnect}
                        disabled={isConnecting}
                        className="w-full max-w-xs bg-gradient-to-r from-green-500 to-green-600 hover:from-green-600 hover:to-green-700 text-white shadow-lg"
                      >
                        {isConnecting ? (
                          <>
                            <Loader2 className="h-4 w-4 mr-2 animate-spin" />
                            Connecting...
                          </>
                        ) : (
                          <>
                            <MessageCircle className="h-4 w-4 mr-2" />
                            Start Conversation
                          </>
                        )}
                      </Button>
                    </motion.div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </motion.div>
        
        {/* Compatibility Breakdown */}
        <motion.section 
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.1 }}
          className="mb-8"
        >
          <Card>
            <CardContent className="p-6">
              <h2 className="text-xl font-semibold text-gray-900 mb-4 flex items-center">
                <Heart className="h-5 w-5 text-red-500 mr-2" />
                Why you'd work well together
              </h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <CompatibilityMetric
                  title="Shared Skills"
                  score={matchData.skills_overlap}
                  max={100}
                  icon={<Code className="h-5 w-5 text-blue-600" />}
                  color="blue"
                >
                  <div className="flex flex-wrap gap-1 mt-2">
                    {matchData.shared_skills.slice(0, 6).map((skill) => (
                      <Badge key={skill.id} variant="outline" className="text-xs">
                        {skill.name}
                      </Badge>
                    ))}
                    {matchData.shared_skills.length > 6 && (
                      <Badge variant="outline" className="text-xs">
                        +{matchData.shared_skills.length - 6} more
                      </Badge>
                    )}
                  </div>
                </CompatibilityMetric>
                
                <CompatibilityMetric
                  title="Experience Complement"
                  score={matchData.experience_complement}
                  max={100}
                  icon={<Briefcase className="h-5 w-5 text-green-600" />}
                  color="green"
                >
                  <p className="text-sm text-gray-600 mt-2">
                    Your {profile.years_experience} years of experience complements their background
                    {matchData.experience_note && (
                      <span> - {matchData.experience_note}</span>
                    )}
                  </p>
                </CompatibilityMetric>
                
                <CompatibilityMetric
                  title="Industry Alignment"
                  score={matchData.industry_alignment}
                  max={100}
                  icon={<Building className="h-5 w-5 text-purple-600" />}
                  color="purple"
                >
                  <div className="flex flex-wrap gap-1 mt-2">
                    {matchData.shared_industries.map(industry => (
                      <Badge key={industry} variant="outline" className="text-xs">
                        {industry}
                      </Badge>
                    ))}
                  </div>
                </CompatibilityMetric>
                
                <CompatibilityMetric
                  title="Location & Availability"
                  score={matchData.location_compatibility}
                  max={100}
                  icon={<MapPin className="h-5 w-5 text-orange-600" />}
                  color="orange"
                >
                  <p className="text-sm text-gray-600 mt-2">
                    {matchData.location_note}
                  </p>
                </CompatibilityMetric>
              </div>
            </CardContent>
          </Card>
        </motion.section>
        
        {/* Profile Content Sections */}
        <motion.section 
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.2 }}
          className="space-y-8"
        >
          {/* About Section */}
          <Card>
            <CardContent className="p-6">
              <div className="flex items-center space-x-3 mb-4">
                <User className="h-5 w-5 text-gray-500" />
                <h2 className="text-xl font-semibold text-gray-900">About</h2>
              </div>
              <div className="prose prose-sm max-w-none text-gray-700 leading-relaxed">
                <p>{profile.summary}</p>
                {profile.interests && (
                  <div className="mt-4">
                    <h3 className="font-medium text-gray-900 mb-2">Interests</h3>
                    <div className="flex flex-wrap gap-2">
                      {profile.interests.map(interest => (
                        <Badge key={interest} variant="outline" className="text-xs">
                          {interest}
                        </Badge>
                      ))}
                    </div>
                  </div>
                )}
              </div>
            </CardContent>
          </Card>
          
          {/* Experience Timeline */}
          {profile.experience?.length > 0 && (
            <Card>
              <CardContent className="p-6">
                <div className="flex items-center space-x-3 mb-4">
                  <Briefcase className="h-5 w-5 text-gray-500" />
                  <h2 className="text-xl font-semibold text-gray-900">Experience</h2>
                </div>
                <Timeline experiences={profile.experience} />
              </CardContent>
            </Card>
          )}
          
          {/* Skills Cloud */}
          {profile.skills?.length > 0 && (
            <Card>
              <CardContent className="p-6">
                <div className="flex items-center space-x-3 mb-4">
                  <Code className="h-5 w-5 text-gray-500" />
                  <h2 className="text-xl font-semibold text-gray-900">Skills</h2>
                </div>
                <SkillCloud skills={profile.skills} maxSkills={24} />
              </CardContent>
            </Card>
          )}
          
          {/* Availability & Preferences */}
          <Card>
            <CardContent className="p-6">
              <div className="flex items-center space-x-3 mb-4">
                <Clock className="h-5 w-5 text-gray-500" />
                <h2 className="text-xl font-semibold text-gray-900">Availability</h2>
              </div>
              <AvailabilityCard availability={profile.availability} />
            </CardContent>
          </Card>
        </motion.section>
      </div>
    </div>
  );
}
```

### Compatibility Metrics Component

```tsx
// components/profiles/compatibility-metrics.tsx
import { motion } from 'framer-motion';
import { LucideIcon } from 'lucide-react';

import { Card, CardContent } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';

import { cn } from '@/lib/utils';

interface CompatibilityMetricProps {
  title: string;
  score: number;
  max: number;
  icon: LucideIcon;
  color?: string;
  children?: React.ReactNode;
}

export function CompatibilityMetric({ 
  title, 
  score, 
  max, 
  icon: Icon, 
  color = 'blue',
  children 
}: CompatibilityMetricProps) {
  const percentage = (score / max) * 100;
  const barColor = {
    blue: 'bg-blue-500',
    green: 'bg-green-500',
    purple: 'bg-purple-500',
    orange: 'bg-orange-500'
  }[color];

  return (
    <Card>
      <CardContent className="p-4">
        <div className="flex items-center justify-between mb-3">
          <div className="flex items-center space-x-3">
            <div className={`p-2 rounded-lg bg-${color}-50`}>
              <Icon className={`h-5 w-5 text-${color}-600`} />
            </div>
            <div>
              <h3 className="font-medium text-gray-900 text-sm">{title}</h3>
              <p className="text-xs text-gray-500">{score}/{max}</p>
            </div>
          </div>
          <Badge variant="outline" className={`text-xs border-${color}-200 text-${color}-700`}>
            {percentage.toFixed(0)}%
          </Badge>
        </div>
        
        <div className="space-y-2">
          <div className="flex items-center justify-between text-xs text-gray-500 mb-1">
            <span>Low</span>
            <span>High</span>
          </div>
          <div className="h-2 bg-gray-200 rounded-full overflow-hidden">
            <motion.div
              className={cn('h-full rounded-full transition-all duration-700', barColor)}
              initial={{ width: 0 }}
              animate={{ width: `${percentage}%` }}
            />
          </div>
        </div>
        
        {children && (
          <div className="mt-3 pt-3 border-t border-gray-100">
            {children}
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

---

## Mobile Optimizations

### Responsive Breakpoints

- **sm (640px+):** 2-column grid for matches
- **md (768px+):** Sidebar filters appear
- **lg (1024px+):** 3-column grid, full-width profile details
- **xl (1280px+):** Enhanced spacing, larger avatars

### Touch Interactions

```tsx
// Touch-friendly match card for mobile
function MobileMatchCard({ match, onConnect, onSave, onView }) {
  return (
    <Card className="p-4 mb-4 bg-white rounded-lg shadow-sm border border-gray-200">
      <div className="flex items-start space-x-3">
        <Avatar className="w-12 h-12 flex-shrink-0 mt-1">
          <AvatarImage src={match.profile.avatar_url} />
          <AvatarFallback>{getInitials(match.profile.name)}</AvatarFallback>
        </Avatar>
        
        <div className="flex-1 min-w-0">
          <div className="flex items-center justify-between mb-2">
            <h3 className="text-base font-semibold text-gray-900 truncate">
              {match.profile.headline || match.profile.name}
            </h3>
            <Badge className="ml-2 flex-shrink-0 text-xs">
              {match.match_score}%
            </Badge>
          </div>
          
          <p className="text-sm text-gray-600 mb-2 line-clamp-2">
            {match.profile.summary}
          </p>
          
          <div className="flex flex-wrap gap-1 mb-3">
            {match.shared_skills.slice(0, 3).map(skill => (
              <Badge key={skill.id} variant="outline" className="text-xs">
                {skill.name}
              </Badge>
            ))}
          </div>
          
          <div className="flex items-center justify-between text-xs text-gray-500 mb-3">
            <span>{match.profile.role} • {match.profile.location}</span>
            <span>{formatDistanceToNow(match.created_at)}</span>
          </div>
          
          <div className="flex items-center space-x-2">
            <Button
              variant="outline"
              size="sm"
              className="h-8 px-3 text-xs flex-1"
              onClick={onSave}
            >
              Save
            </Button>
            <Button
              size="sm"
              className="h-8 px-4 ml-auto"
              onClick={onConnect}
            >
              Connect
            </Button>
          </div>
        </div>
      </div>
    </Card>
  );
}
```

### Swipe Gestures

```tsx
// Swipe to action support
import { useSwipeable } from 'react-swipeable';

function SwipeableMatchCard({ match, onConnect, onSave, onView }) {
  const handlers = useSwipeable({
    onSwipedLeft: () => hideMatch(match.id),
    onSwipedRight: () => onSave(match.id),
    trackMouse: true, // Enable on desktop too
    preventScrollOnSwipe: ['vertical']
  });

  return (
    <div {...handlers} className="relative">
      <MobileMatchCard {...props} />
      {/* Swipe feedback */}
      <motion.div
        className="absolute -right-12 top-1/2 transform -translate-y-1/2"
        initial={{ x: 20, opacity: 0 }}
        animate={{ x: 0, opacity: 1 }}
        transition={{ duration: 0.3 }}
      >
        <Button variant="outline" size="sm" onClick={onSave}>
          Save
        </Button>
      </motion.div>
    </div>
  );
}
```

---

## Performance Optimizations

### Bundle Analysis & Code Splitting

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    optimizePackageImports: ['lucide-react', 'framer-motion'],
  },
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'avatars.githubusercontent.com',
        pathname: '/u/**',
      },
      {
        protocol: 'https',
        hostname: 'lh3.googleusercontent.com',
      },
    ],
  },
  webpack: (config, { isServer }) => {
    // Optimize for production
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
      };
    }
    return config;
  },
};

module.exports = nextConfig;
```

### Image Optimization

```tsx
// Optimized avatar component
import Image from 'next/image';

function OptimizedAvatar({ src, alt, className, ...props }) {
  return (
    <div className={cn('relative overflow-hidden rounded-full bg-gray-100', className)}>
      <Image
        src={src || '/default-avatar.png'}
        alt={alt}
        fill
        className="object-cover"
        sizes="(max-width: 768px) 48px, (max-width: 1200px) 64px, 80px"
        priority={false}
        placeholder="blur"
        blurDataURL="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="
        onError={(e) => {
          e.currentTarget.src = '/default-avatar.png';
        }}
        {...props}
      />
    </div>
  );
}
```

### Server-Side Rendering Strategy

```tsx
// Dynamic imports for heavy components
import dynamic from 'next/dynamic';

const MatchFilters = dynamic(() => import('@/components/matches/match-filters'), {
  loading: () => <FiltersSkeleton />,
  ssr: false // Client-side only for interactive filters
});

const SkillCloud = dynamic(() => import('@/components/ui/skill-cloud'), {
  ssr: false, // Canvas-based, client-side only
  loading: () => <div className="h-32 bg-gray-200 rounded-lg animate-pulse" />
});

// Static generation for SEO pages
export const dynamic = 'force-static';
export async function generateStaticParams() {
  return [];
}
```

### Caching Strategy

```typescript
// lib/queries.ts
import { createTRPCReact } from '@trpc/react-query';
import { AppRouter } from '@/server/routers/_app';

export const api = createTRPCReact<AppRouter>();

export function matchesQuery(filters: MatchFilters, page = 0) {
  return api.matches.list.useQuery(
    { ...filters, page, limit: 20 },
    {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      keepPreviousData: true,
      refetchOnWindowFocus: false,
      retry: (failureCount, error) => {
        if (error.status === 404) return false;
        return failureCount < 3;
      }
    }
  );
}

export function profileQuery(id: string) {
  return api.profiles.get.useQuery(
    { id },
    {
      staleTime: 30 * 60 * 1000, // 30 minutes for profiles
      cacheTime: 60 * 60 * 1000, // 1 hour
      refetchOnMount: false,
    }
  );
}
```

---

## Accessibility Implementation

### Keyboard Navigation

```tsx
// Enhanced match card with keyboard support
function AccessibleMatchCard({ match, ...props }) {
  const [isHovered, setIsHovered] = useState(false);
  const buttonRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isHovered) {
      buttonRef.current?.focus();
    }
  }, [isHovered]);

  return (
    <Card 
      role="article"
      tabIndex={0}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          props.onConnect();
        }
        if (e.key === 'ArrowRight') {
          e.preventDefault();
          props.onView();
        }
      }}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      className="focus:ring-2 focus:ring-blue-500 focus:outline-none"
    >
      {/* Content */}
      <CardContent>
        {/* ... match content ... */}
      </CardContent>
      
      <CardFooter>
        <div className="sr-only" aria-live="polite">
          {isHovered ? 'Match card focused. Press Enter to connect.' : ''}
        </div>
        
        <Button
          ref={buttonRef}
          onClick={props.onConnect}
          aria-label={`Connect with ${match.profile.name}, ${match.match_score}% match`}
        >
          Connect ({match.match_score}%)
        </Button>
      </CardFooter>
    </Card>
  );
}
```

### Screen Reader Support

```tsx
// ARIA labels and live regions
function ScreenReaderFriendlyMatches({ matches }) {
  return (
    <section aria-labelledby="matches-heading">
      <h2 id="matches-heading" className="sr-only">
        Your matches
      </h2>
      
      <div role="main" aria-live="polite" aria-atomic="false">
        {matches.map(match => (
          <article key={match.id} className="mb-6">
            <header>
              <h3 id={`match-${match.id}-name`}>
                {match.profile.name}, {match.profile.role}
              </h3>
              <p aria-describedby={`match-${match.id}-name`}>
                Match score: {match.match_score}%
              </p>
            </header>
            
            <ul aria-label="Shared skills">
              {match.shared_skills.map(skill => (
                <li key={skill.id}>{skill.name}</li>
              ))}
            </ul>
            
            <footer>
              <button
                aria-label={`Connect with ${match.profile.name}`}
                onClick={() => connect(match.id)}
              >
                Connect
              </button>
            </footer>
          </article>
        ))}
      </div>
    </section>
  );
}
```

### Color Contrast & Focus Indicators

```css
/* globals.css - Focus management */
*:focus-visible {
  outline: 2px solid hsl(var(--primary));
  outline-offset: 2px;
}

/* Ensure sufficient contrast */
.text-primary {
  color: hsl(var(--primary-foreground));
}

.bg-card {
  background-color: hsl(var(--card-background));
  color: hsl(var(--card-foreground));
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  .bg-gray-50 {
    background-color: #f9fafb;
  }
  
  .text-gray-600 {
    color: #4b5563;
  }
  
  .border-gray-200 {
    border-color: #e5e7eb;
  }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Testing Strategy

### Component Tests

```tsx
// __tests__/match-card.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { MatchCard } from '@/components/matches/match-card';

import { mockMatch } from '../mocks';

describe('MatchCard', () => {
  test('renders match information correctly', () => {
    render(<MatchCard match={mockMatch} onConnect={jest.fn()} onSave={jest.fn()} onView={jest.fn()} />);
    
    expect(screen.getByText(mockMatch.profile.name)).toBeInTheDocument();
    expect(screen.getByText(`${mockMatch.match_score}%`)).toBeInTheDocument();
    expect(screen.getByText('Connect')).toBeInTheDocument();
  });

  test('handles connect button click', async () => {
    const onConnect = jest.fn();
    render(<MatchCard match={mockMatch} onConnect={onConnect} onSave={jest.fn()} onView={jest.fn()} />);
    
    fireEvent.click(screen.getByRole('button', { name: /connect/i }));
    
    await waitFor(() => {
      expect(onConnect).toHaveBeenCalledWith();
    });
  });

  test('displays loading state during connection', () => {
    const { rerender } = render(
      <MatchCard match={mockMatch} onConnect={jest.fn()} onSave={jest.fn()} onView={jest.fn()} isConnecting={true} />
    );
    
    expect(screen.getByText('Connecting...')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /connecting/i })).toBeDisabled();
  });

  test('is accessible with proper ARIA labels', () => {
    render(<MatchCard match={mockMatch} onConnect={jest.fn()} onSave={jest.fn()} onView={jest.fn()} />);
    
    const connectButton = screen.getByRole('button', { name: `Connect with ${mockMatch.profile.name}, ${mockMatch.match_score}% match` });
    expect(connectButton).toBeInTheDocument();
  });
});
```

### E2E Tests with Playwright

```javascript
// tests/matches.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Matches Page', () => {
  test('should load matches and display filters', async ({ page }) => {
    await page.goto('/matches');
    
    // Wait for matches to load
    await expect(page.locator('[data-testid="match-card"]')).toHaveCount(6);
    
    // Check header
    await expect(page.getByRole('heading', { name: 'Matches' })).toBeVisible();
    
    // Check filters are accessible
    await expect(page.getByLabel('Role')).toBeVisible();
    await expect(page.getByLabel('Skills')).toBeVisible();
    
    // Test filter interaction
    await page.getByLabel('Role').click();
    await page.getByText('Frontend Developer').click();
    
    // Verify results update
    await expect(page.locator('[data-testid="match-card"]')).toHaveCount.withTimeout(5000);
  });

  test('should handle infinite scroll', async ({ page }) => {
    await page.goto('/matches');
    
    // Scroll to bottom
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    
    // Wait for more matches to load
    await expect(page.locator('[data-testid="match-card"]')).toHaveCount.greaterThan(6);
  });

  test('should be mobile responsive', async ({ page, viewport }) => {
    await viewport.setSize({ width: 375, height: 667 });
    await page.goto('/matches');
    
    // Check mobile layout
    await expect(page.locator('.md\:grid-cols-2')).toHaveClass(/grid-cols-1/);
    await expect(page.getByRole('button', { name: 'Connect' })).toHaveAttribute('size', 'sm');
  });

  test('keyboard navigation works', async ({ page }) => {
    await page.goto('/matches');
    
    // Tab to first match card
    await page.keyboard.press('Tab');
    await expect(page.locator('[role="article"]:focus')).toBeVisible();
    
    // Enter to connect
    await page.keyboard.press('Enter');
    await expect(page.getByText('Started conversation')).toBeVisible();
  });
});
```

---

## Analytics & Monitoring

### User Behavior Tracking

```typescript
// lib/analytics.ts
import { AnalyticsBrowser } from '@segment/analytics-next';

const analytics = AnalyticsBrowser.load({
  writeKey: process.env.NEXT_PUBLIC_SEGMENT_WRITE_KEY,
});

export async function trackMatchEvent(event: string, properties: any) {
  await analytics.track(event, {
    ...properties,
    userId: getUserId(),
    timestamp: new Date().toISOString(),
  });
}

export async function trackPageView(page: string) {
  await analytics.page({
    name: page,
    properties: {
      path: window.location.pathname,
      referrer: document.referrer,
    },
  });
}

// Usage in components
useEffect(() => {
  trackPageView('matches');
}, []);

// Track match interactions
const handleConnect = async (matchId: string) => {
  await trackMatchEvent('match_connect_clicked', {
    match_id: matchId,
    match_score: match.match_score,
    shared_skills_count: match.shared_skills.length,
  });
  // ... connect logic
};
```

### Performance Monitoring

```javascript
// lib/performance.ts
import { reportWebVitals } from 'web-vitals';

function sendToAnalytics(metric: any) {
  // Send to Segment or other analytics
  console.log('Performance metric:', metric);
}

// Web Vitals
reportWebVitals(sendToAnalytics);

// Custom performance tracking
export function trackComponentRenderTime(component: string, startTime: number) {
  const duration = performance.now() - startTime;
  
  // Track if above threshold
  if (duration > 100) {
    analytics.track('component_render_slow', {
      component,
      duration,
      url: window.location.pathname,
    });
  }
}

// Usage
const startTime = performance.now();
// ... component render
const MatchCard = React.memo(({ match }) => {
  useEffect(() => {
    trackComponentRenderTime('MatchCard', startTime);
  }, []);
  // ... render
});
```

This completes the comprehensive Match Discovery UI specification for OpenHR, covering dashboard, matches interface, filtering, search, profile details, mobile optimization, accessibility, testing, and monitoring.

**References:**
- Next.js Performance [web:265]
- Tailwind Responsive Design [web:266]
- shadcn/ui Accessibility [web:267]
- Framer Motion Animations [web:268]