# Implementation Plan: Frequency-Aware Study Plan Modes

## Quick Summary
**Goal**: Make study plan modes respect task frequency preferences while maintaining their core scheduling philosophy.

**Current Problem**: User sets "weekly" frequency → system ignores it and schedules daily anyway.

**Solution**: Each study plan mode will honor frequency preferences as the baseline, only overriding when necessary.

## Implementation Phases

### Phase 1: Core Frequency Logic (High Priority)

#### 1.1 Create Frequency-Aware Day Selection
```typescript
// New utility in scheduling.ts
const getFrequencyAwareDays = (
  task: Task, 
  availableDays: string[], 
  studyPlanMode: string
): string[] => {
  const { targetFrequency } = task;
  
  switch(targetFrequency) {
    case 'daily':
      return availableDays; // Use all available days
      
    case '3x-week':
      return selectDaysForFrequency(availableDays, 3, 'week');
      
    case 'weekly':
      return selectDaysForFrequency(availableDays, 1-2, 'week');
      
    case 'flexible':
      // Let study plan mode decide
      return getModeBehaviorForFlexible(availableDays, studyPlanMode);
  }
}

const selectDaysForFrequency = (
  days: string[], 
  targetDaysPerPeriod: number, 
  period: 'week'
): string[] => {
  // Group days by week
  const weekGroups = groupDaysByWeek(days);
  const selectedDays: string[] = [];
  
  weekGroups.forEach(weekDays => {
    // For each week, select optimal days based on:
    // - Existing commitments density
    // - Spread across week (Mon, Wed, Fri better than Mon, Tue, Wed)
    // - User's work day preferences
    const weekSelection = selectOptimalDaysInWeek(weekDays, targetDaysPerPeriod);
    selectedDays.push(...weekSelection);
  });
  
  return selectedDays;
}
```

#### 1.2 Update Each Study Plan Mode

##### A. Enhanced "Even" Mode
```typescript
// In generateStudyPlan() for 'even' mode
if (settings.studyPlanMode === 'even') {
  for (const task of tasksEven) {
    // NEW: Get frequency-aware days first
    const frequencyDays = getFrequencyAwareDays(task, availableDays, 'even');
    
    // Check if frequency allows completion before deadline
    const canCompleteOnTime = checkFrequencyDeadlineCompatibility(task, frequencyDays);
    
    if (!canCompleteOnTime && isUrgentDeadline(task)) {
      // Override frequency for urgent deadlines
      const daysForTask = getUrgentScheduleDays(task, availableDays);
      addSchedulingNote(task, 'frequency_overridden_urgent');
    } else {
      // Respect frequency preference
      const daysForTask = frequencyDays;
      addSchedulingNote(task, 'frequency_respected');
    }
    
    // Continue with existing even distribution logic
    const sessionLengths = optimizeSessionDistribution(task, totalHours, daysForTask, settings);
    // ... rest of existing logic
  }
}
```

##### B. Enhanced "Balanced" Mode
```typescript
// In balanced mode logic
const distributeTierTasks = (tierTasks: Task[], tierName: string) => {
  for (const task of tierTasks) {
    // NEW: Apply frequency awareness within priority tiers
    const frequencyDays = getFrequencyAwareDays(task, availableDays, 'balanced');
    
    // High priority tasks get more flexibility to override frequency
    const canOverrideFrequency = task.importance && isUrgentDeadline(task);
    
    if (canOverrideFrequency && !checkFrequencyDeadlineCompatibility(task, frequencyDays)) {
      const daysForTask = availableDays; // Use all days for high priority urgent tasks
      addSchedulingNote(task, 'frequency_overridden_priority');
    } else {
      const daysForTask = frequencyDays;
      addSchedulingNote(task, 'frequency_respected');
    }
    
    // Continue with existing distribution logic
  }
}
```

##### C. Enhanced "Eisenhower" Mode  
```typescript
// In Eisenhower mode
const scheduleByPriority = (tasksSorted: Task[]) => {
  tasksSorted.forEach(task => {
    const priority = getEisenhowerPriority(task);
    
    if (priority === 'Q1_urgent_important') {
      // Q1: Always override frequency - these are critical
      const daysForTask = availableDays;
      addSchedulingNote(task, 'frequency_overridden_critical');
    } 
    else if (priority === 'Q2_important_not_urgent') {
      // Q2: Try to respect frequency, but be flexible
      const frequencyDays = getFrequencyAwareDays(task, availableDays, 'eisenhower');
      const canComplete = checkFrequencyDeadlineCompatibility(task, frequencyDays);
      
      const daysForTask = canComplete ? frequencyDays : availableDays;
      addSchedulingNote(task, canComplete ? 'frequency_respected' : 'frequency_overridden_important');
    }
    else {
      // Q3, Q4: Always respect frequency unless impossible
      const frequencyDays = getFrequencyAwareDays(task, availableDays, 'eisenhower');
      const daysForTask = frequencyDays;
      addSchedulingNote(task, 'frequency_respected');
    }
  });
}
```

### Phase 2: UI Improvements (Medium Priority)

#### 2.1 Enhanced Task Input Feedback
```typescript
// In TaskInput.tsx - show frequency + mode interaction
const FrequencyModePreview = ({ frequency, studyPlanMode, task }) => {
  const preview = simulateFrequencyBehavior(frequency, studyPlanMode, task);
  
  return (
    <div className="bg-blue-50 p-3 rounded-lg mt-2">
      <h4>📅 Scheduling Preview</h4>
      <p>With your "{studyPlanMode}" mode:</p>
      <ul>
        <li>Work frequency: {preview.actualFrequency}</li>
        <li>Session pattern: {preview.sessionPattern}</li>
        {preview.willOverride && (
          <li className="text-orange-600">
            ⚠️ May work more frequently due to tight deadline
          </li>
        )}
      </ul>
    </div>
  );
}
```

#### 2.2 Settings Page Mode Descriptions
```typescript
// Enhanced descriptions in Settings.tsx
const getModeDescription = (mode: string) => {
  return {
    even: {
      title: "Steady Progress",
      description: "Distributes work evenly while respecting your frequency preferences",
      frequencyBehavior: "Honors frequency settings unless deadline is very tight"
    },
    balanced: {
      title: "Smart Balance", 
      description: "Balances task priority with your preferred work rhythm",
      frequencyBehavior: "Respects frequency for routine tasks, overrides for high-priority deadlines"
    },
    eisenhower: {
      title: "Priority First",
      description: "Schedules by urgency/importance, adjusting frequency as needed",
      frequencyBehavior: "Critical tasks may work daily regardless of frequency preference"
    }
  }
}
```

#### 2.3 Calendar Frequency Indicators
```typescript
// In CalendarView.tsx - show frequency adherence
const getFrequencyIndicator = (session: StudySession) => {
  if (session.schedulingMetadata?.frequencyOverridden) {
    return {
      indicator: "🔄",
      color: "orange",
      tooltip: "Frequency overridden due to deadline pressure"
    };
  }
  
  if (session.schedulingMetadata?.frequencyRespected) {
    return {
      indicator: "✅", 
      color: "green",
      tooltip: "Scheduled according to your frequency preference"
    };
  }
  
  return null;
}
```

### Phase 3: Smart Defaults & Education (Lower Priority)

#### 3.1 Intelligent Frequency Suggestions
```typescript
// In TaskInput.tsx
const suggestOptimalFrequency = (task: TaskFormData, userSettings: UserSettings) => {
  const deadline = new Date(task.deadline);
  const daysUntilDeadline = getDaysUntilDeadline(deadline);
  const estimatedHours = task.estimatedHours;
  
  // Smart suggestions based on task characteristics
  if (daysUntilDeadline <= 3) return 'daily'; // Urgent
  if (estimatedHours <= 2) return 'daily'; // Short tasks
  if (task.category === 'Learning') return '3x-week'; // Consistent practice
  if (!task.importance) return 'weekly'; // Low priority
  if (estimatedHours >= 10) return '3x-week'; // Large projects
  
  return 'flexible'; // Let mode decide
}
```

#### 3.2 Onboarding & Education
```typescript
// Interactive tutorial updates
const FrequencyModeEducation = () => (
  <div className="tutorial-step">
    <h3>🎯 Study Plan Modes + Work Frequency</h3>
    <p>Your study plan mode sets the <strong>strategy</strong>, 
       frequency sets the <strong>rhythm</strong>:</p>
    
    <div className="examples">
      <div>📚 "Learning + 3x-week" = Consistent practice 3 days per week</div>
      <div>🔥 "Priority First + daily" = Critical tasks every day</div>
      <div>📈 "Steady Progress + weekly" = Even progress 1-2 days per week</div>
    </div>
  </div>
);
```

## Implementation Priority

### ✅ Phase 1 (Essential - Do First)
- Create `getFrequencyAwareDays()` utility
- Update "even" mode to respect frequency
- Add basic scheduling notes for override reasons
- Test with existing tasks

### 🔄 Phase 2 (Important - Do Next)  
- Update balanced and eisenhower modes
- Add frequency preview in task input
- Enhance mode descriptions in settings
- Add calendar indicators

### 📋 Phase 3 (Nice to Have - Do Later)
- Smart frequency suggestions
- Advanced education/onboarding
- Detailed analytics on frequency adherence
- User customization of override thresholds

## Success Metrics

1. **User Intent Alignment**: Tasks with "weekly" frequency should actually be scheduled weekly (not daily)
2. **Deadline Safety**: Urgent tasks still complete on time even with frequency constraints
3. **Predictable Behavior**: Users understand when/why frequency is overridden
4. **Reduced Confusion**: Fewer support questions about scheduling behavior

## Questions for User Decision

1. **Override Threshold**: How urgent should deadlines be before overriding frequency? (3 days? 1 week?)
2. **Mode Defaults**: Should "Priority First" mode be more aggressive about overriding frequency?
3. **User Control**: Should users be able to set "never override frequency" as an option?
4. **Notification Level**: How much should users be notified about frequency overrides?

This plan creates a unified system where both features work together intelligently rather than conflicting with each other.
