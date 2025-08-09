# Drag and Drop Precision - Test Plan

## Quick Test Steps

### 1. Test Basic Precision
1. **Open the calendar** in Week or Day view
2. **Create some study sessions** if you don't have any
3. **Drag a session** to a specific time slot
4. **Verify**: The session should snap to the calendar grid and place exactly where intended

### 2. Test Different Time Intervals
1. **Change the time interval** using the dropdown in the calendar toolbar (try 15 min, 30 min, 60 min)
2. **Drag sessions** and verify they snap to the selected interval grid
3. **Expected**: Sessions should align with the visible time grid lines

### 3. Test Feedback Messages
1. **Drag a session** to an empty time slot
2. **Look for feedback** in the top-right corner
3. **Expected**: 
   - ✅ Green message if placed exactly
   - 📍 Blue message if placed within 15 minutes of target
   - 🔄 Orange message if moved to nearest available slot

### 4. Test Micro-Movement Prevention
1. **Drag a session** and drop it very close to its original position (< 5 minutes)
2. **Expected**: Session should return to original position with a gray "micro-movement ignored" message

### 5. Test Conflict Resolution
1. **Try to drag a session** to a time slot that's already occupied
2. **Expected**: Session should be placed in the nearest available slot with appropriate feedback

## What Should Be Fixed

### Before (Old Behavior):
- ❌ Sessions would drop randomly "near" the target
- ❌ Unpredictable snapping to large intervals (15+ minutes)
- ❌ Poor feedback about where sessions were placed
- ❌ Inconsistent with calendar grid

### After (New Behavior):
- ✅ Sessions snap precisely to calendar grid
- ✅ Snapping matches user's selected time interval
- ✅ Clear feedback about exact placement
- ✅ Micro-movement tolerance prevents accidents
- ✅ Consistent visual experience

## Visual Improvements

### Enhanced Drag Experience:
- **Better drag preview**: More visible dashed border and shadow
- **Improved drop zones**: Clear green highlighting for valid drops
- **Hover effects**: Sessions scale slightly when hovering
- **Smooth transitions**: Subtle animations during drag operations

### Better Feedback:
- **Color-coded notifications**: Different colors for different outcomes
- **Specific timing information**: Shows exact time and any adjustments
- **Longer display time**: Feedback visible for 4 seconds
- **Emoji indicators**: Visual cues for quick understanding

## Common Issues to Test

1. **Edge Cases**:
   - Dragging to study window boundaries (early morning/late evening)
   - Dragging to days with many existing commitments
   - Dragging very short sessions (< 30 minutes)

2. **Different Views**:
   - Test in Day view (most precise)
   - Test in Week view (standard use)
   - Verify Month view shows sessions correctly after moving

3. **Settings Integration**:
   - Test with different buffer times between sessions
   - Test with different study window hours
   - Test with different work days configured

## Troubleshooting

If drag and drop still seems imprecise:

1. **Check time interval**: Make sure you're using a reasonable interval (15-30 minutes work best)
2. **Look at feedback**: The notification will tell you exactly what happened
3. **Try Day view**: This provides the most precise control
4. **Check console**: Open browser dev tools to see debug information

## Report Issues

If you still experience precision issues after this fix:

1. **Note the specific behavior**: What time you dragged to vs where it ended up
2. **Check the feedback message**: What did the system tell you?
3. **Note your settings**: Time interval, buffer settings, study window hours
4. **Try different scenarios**: Does it happen consistently or only in specific situations?

The new system should be much more predictable and user-friendly!
