# Task Frequency Preference vs Study Plan Modes Analysis

## Current State Analysis

### Task Frequency Preferences (in TaskInput.tsx)
- **Available Options**: `daily`, `weekly`, `3x-week`, `flexible`
- **Application**: Applied to ALL tasks, with frequency-deadline conflict detection
- **Current Logic**: Checks if frequency allows completion before deadline

### Study Plan Modes (in Settings.tsx)
- **Available Modes**: `eisenhower`, `even`, `balanced`
- **Current Logic**:
  - **Even**: Distributes hours evenly across available days
  - **Balanced**: Combines even distribution with priority-based ordering (Eisenhower Matrix)
  - **Eisenhower**: Pure priority-based scheduling (Q1→Q2→Q3→Q4)

## The Problem: Disconnected Systems

### 1. **Frequency is Ignored by Study Plan Modes**
```typescript
// In scheduling.ts - No frequency consideration in distribution logic
if (settings.studyPlanMode === 'even') {
  // Just distributes evenly - doesn't respect targetFrequency
  const sessionLengths = optimizeSessionDistribution(task, totalHours, daysForTask, settings);
}
```

### 2. **Conflicting Intentions**
- User sets task to "weekly" frequency → expects work 1-2 days per week
- Study plan mode "even" → forces daily distribution
- Result: User's frequency preference is completely ignored

### 3. **No Clear Hierarchy**
- Which takes precedence: frequency preference or study plan mode?
- No user education about how they interact

## Proposed Solution: Unified Frequency-Aware Scheduling

### Phase 1: Make Study Plan Modes Respect Frequency

#### A. Enhanced "Even" Mode → "Frequency-Aware Even"
```typescript
// NEW: Distribute evenly within frequency constraints
if (task.targetFrequency === 'weekly') {
  // Cluster sessions into 1-2 days per week instead of daily spread
  const targetDaysPerWeek = 2;
  const sessionDays = selectOptimalDaysInWeek(availableDays, targetDaysPerWeek);
}
```

#### B. Enhanced "Balanced" Mode → "Priority + Frequency"
```typescript
// NEW: Apply frequency preferences within priority tiers
const distributeTierTasks = (tierTasks: Task[]) => {
  tierTasks.forEach(task => {
    const frequencyConstrainedDays = getFrequencyAwareDays(task, availableDays);
    // Then apply even distribution within those days
  });
}
```

#### C. Enhanced "Eisenhower" Mode → "Priority-First + Frequency"
```typescript
// NEW: Schedule high priority first, then respect frequency for lower priority
if (task.importance && urgentDeadline) {
  // Override frequency for critical tasks
  distributeDaily(task);
} else {
  // Respect frequency preference
  distributeByFrequency(task);
}
```

### Phase 2: Intelligent Frequency Selection in Task Input

#### A. Smart Defaults Based on Task Properties
```typescript
const suggestFrequency = (task: TaskFormData) => {
  // Deadline-driven suggestions
  if (isUrgent(task.deadline)) return 'daily';
  if (task.estimatedHours < 2) return 'daily'; // Short tasks
  if (task.category === 'Learning') return '3x-week'; // Consistent practice
  if (!task.importance) return 'weekly'; // Low priority
  return 'flexible'; // Default
}
```

#### B. Dynamic Frequency Validation
```typescript
// Enhanced conflict detection
const validateFrequencyDeadlineCombo = (task, frequency, studyPlanMode) => {
  const simulation = simulateScheduling(task, frequency, studyPlanMode);
  return {
    canComplete: simulation.completionDate <= task.deadline,
    suggestedFrequency: simulation.optimalFrequency,
    modeSpecificAdvice: getModeSpecificAdvice(studyPlanMode, simulation)
  };
}
```

### Phase 3: Study Plan Mode Redesign for Clarity

#### A. Rename Modes for Better Understanding
- **Even** → **"Steady Progress"** (frequency-respecting even distribution)
- **Balanced** → **"Smart Balance"** (priority + frequency)  
- **Eisenhower** → **"Priority First"** (override frequency for urgent tasks)

#### B. Mode-Specific Frequency Behavior
```typescript
const getModeDescription = (mode: string) => {
  switch(mode) {
    case 'steady':
      return "Respects your frequency preferences while distributing work evenly";
    case 'smart':
      return "Balances priorities with your preferred work frequency";
    case 'priority':
      return "Prioritizes urgent tasks, may override frequency for deadlines";
  }
}
```

## Implementation Plan

### Step 1: Frequency-Aware Scheduling Utils
```typescript
// New utility functions
const getFrequencyAwareDays = (task: Task, availableDays: string[]) => {
  switch(task.targetFrequency) {
    case 'daily': return availableDays;
    case '3x-week': return select3DaysPerWeek(availableDays);
    case 'weekly': return selectWeeklyDays(availableDays, 1-2);
    case 'flexible': return availableDays; // Mode decides
  }
}

const selectOptimalDaysInWeek = (days: string[], targetDaysPerWeek: number) => {
  // Smart day selection within weeks
  // Prefer spreading across different days of week
  // Consider user's work days preference
}
```

### Step 2: Enhanced Mode Logic
Update each study plan mode to:
1. **Check task frequency preference first**
2. **Apply mode-specific distribution within frequency constraints**
3. **Override only when necessary (urgent deadlines)**

### Step 3: UI Improvements

#### A. Task Input Enhancements
- Show real-time preview of how frequency + current study mode will work
- Dynamic suggestions: "With your current 'Priority First' mode, daily frequency recommended for urgent tasks"
- Conflict resolution with specific recommendations

#### B. Settings Page Integration
- Show how study plan mode affects frequency preferences
- Preview scheduling behavior for each mode
- Clear explanations of when frequency is overridden

#### C. Feedback in Calendar
- Color-code sessions by frequency adherence
- Show when frequency was overridden and why
- Statistics: "This week: 85% of tasks followed preferred frequency"

## Expected Benefits

### 1. **User Intent Alignment**
- Users get the work rhythm they actually want
- Frequency preferences are respected, not ignored
- Clear communication when overrides are necessary

### 2. **Better Work-Life Balance**
- "Weekly" tasks actually happen weekly, not daily micro-sessions
- "3x-week" learning tasks maintain consistent practice rhythm
- More predictable schedule patterns

### 3. **Reduced Cognitive Load**
- Clear hierarchy: Mode sets the strategy, frequency sets the rhythm
- Predictable behavior across different scenarios
- Fewer "Why did this happen?" moments

### 4. **Enhanced Productivity**
- Tasks scheduled in optimal frequency patterns for learning/retention
- Less context switching from over-fragmented sessions
- Better alignment with natural work rhythms

## Questions for User

1. **Priority vs Frequency**: When should urgent deadlines override frequency preferences?
2. **Default Behavior**: Should study plan modes respect frequency by default, or override by default?
3. **User Education**: How much explanation/preview do users need when setting up these preferences?
4. **Granular Control**: Should users be able to set frequency preferences per study plan mode?

This analysis shows that the current system has two independent scheduling dimensions that don't work together well. The solution is to create a unified system where study plan modes become the "strategy" and frequency preferences become the "rhythm" within that strategy.
