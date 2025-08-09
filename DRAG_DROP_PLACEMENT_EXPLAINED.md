# Why Sessions Sometimes Go to "Nearest Available Slot"

## The Simple Answer
**Yes, it absolutely depends on how long the block is!** Longer sessions are much harder to place exactly where you drop them.

## How It Works

### 1. **Exact Placement First** ✅
The system ALWAYS tries to place your session exactly where you drop it first.

### 2. **Conflict Check** 🔍
If the exact location doesn't work, it's because of conflicts:

#### **Duration-Based Conflicts** (Your Main Issue)
- **Short sessions (15-30 min)**: Easy to place exactly - lots of small gaps available
- **Medium sessions (1-2 hours)**: Harder to place - needs bigger continuous gaps  
- **Long sessions (3+ hours)**: Very hard to place exactly - needs large blocks of free time

#### **Other Conflicts**
- **Existing sessions**: Already have something scheduled there
- **Fixed commitments**: Classes, meetings, appointments
- **Buffer time**: If you have buffer time between sessions (Settings → Buffer Time)
- **Study window boundaries**: Can't place outside your study hours

### 3. **Smart Fallback** 🎯
When exact placement fails, the system finds the nearest available slot that can fit your entire session.

## Visual Examples

### Example 1: 30-minute session
```
Calendar:  [Free] [Free] [Meeting] [Free] [Free]
Drop here:        ↑
Result:    [Free] [SESSION] [Meeting] [Free] [Free] ✅ EXACT
```

### Example 2: 2-hour session  
```
Calendar:  [Free] [30min] [Free] [Free] [Free]
Drop here:        ↑
Result:    [Free] [30min] [2-HOUR SESSION] ❌ MOVED (not enough space)
```

## How to Get More Exact Placement

### 1. **Check Your Buffer Settings**
- Go to **Settings** → **Buffer Time Between Sessions**
- If it's set to 15+ minutes, reduce it to 0-5 minutes
- This gives you more placement flexibility

### 2. **Use Shorter Sessions**
- Break long sessions into smaller chunks
- Instead of one 3-hour block, try three 1-hour blocks
- Smaller sessions = more exact placement

### 3. **Choose Better Drop Zones**
- Look for larger empty areas on your calendar
- Avoid dropping right after existing sessions (buffer space needed)
- Drop at the start of empty time blocks

### 4. **Use Day View for Precision**
- Day view shows the most detail
- Easier to see exactly where conflicts are
- Best view for precise drag and drop

## Quick Test
1. **Try a 15-minute session** - should place exactly where you drop
2. **Try a 3-hour session** - more likely to move to nearest available slot
3. **Check the feedback message** - it tells you exactly what happened!

## The Feedback Colors Tell You Why
- ✅ **Green**: "Placed exactly at X:XX" - No conflicts!
- 📍 **Blue**: "Placed at X:XX (Ymin from target)" - Minor adjustment
- 🔄 **Orange**: "Moved to nearest available slot" - Conflicts found

This behavior is actually a feature, not a bug - it ensures your sessions never overlap or violate your scheduling rules!
