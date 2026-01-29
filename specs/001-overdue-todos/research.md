# Research: Overdue Todo Items

**Feature**: 001-overdue-todos  
**Date**: 2026-01-29  
**Phase**: Phase 0 - Research & Investigation

## Research Questions

Based on Technical Context unknowns and feature requirements, this document resolves all "NEEDS CLARIFICATION" items and investigates best practices.

## 1. Date Comparison in JavaScript

### Decision
Use native JavaScript Date objects for date comparison with ISO 8601 date strings stored in the database.

### Rationale
- **Simplicity**: JavaScript's Date object provides built-in comparison operators
- **Performance**: Date comparison is O(1) - creating Date objects from ISO strings is ~1-5μs per operation
- **Existing Codebase**: Project already stores dates; no schema changes needed
- **Cross-browser Compatibility**: Date parsing of ISO 8601 formats is universally supported
- **Testing**: Easy to mock with Jest's `jest.useFakeTimers()`

### Implementation Approach
```javascript
function isOverdue(dueDate, currentDate = new Date()) {
  if (!dueDate) return false;
  
  const due = new Date(dueDate);
  const current = new Date(currentDate);
  
  // Set both to midnight for date-only comparison
  due.setHours(0, 0, 0, 0);
  current.setHours(0, 0, 0, 0);
  
  return due < current;
}
```

### Alternatives Considered
- **date-fns library**: More features but adds 13KB bundle size - overkill for simple comparison
- **Moment.js**: Deprecated and heavy (67KB) - not recommended for new projects
- **Day.js**: Lightweight (2KB) but unnecessary for our simple use case
- **Manual string comparison**: Error-prone and doesn't handle edge cases

## 2. Server Time Delivery to Frontend

### Decision
Add a lightweight `/api/server-time` endpoint that returns current server timestamp on initial page load.

### Rationale
- **Accuracy**: Ensures all users see consistent overdue status regardless of local clock settings
- **Simplicity**: Single endpoint, simple implementation
- **Performance**: One-time call on app initialization, minimal overhead
- **Caching**: Frontend caches server time and uses client-side calculation for subsequent checks
- **Existing Pattern**: Project already uses REST API pattern with Express.js

### Implementation Approach
**Backend** (Express.js):
```javascript
app.get('/api/server-time', (req, res) => {
  res.json({ timestamp: Date.now() });
});
```

**Frontend** (React):
```javascript
// In App.js or TodoList.js
useEffect(() => {
  async function fetchServerTime() {
    const response = await fetch('/api/server-time');
    const { timestamp } = await response.json();
    setServerTime(timestamp);
  }
  fetchServerTime();
}, []);
```

### Alternatives Considered
- **Include timestamp in existing API responses**: Couples time sync with todo data, harder to test
- **Embed in HTML at render time (SSR)**: Project uses client-side React, would require major architecture change
- **NTP/time sync library**: Overcomplicated for single-user app
- **Client-side only**: Vulnerable to incorrect system clocks (rejected per clarifications)

## 3. Visual Styling for Overdue Status

### Decision
Implement overdue styling using:
1. **Text Badge**: "OVERDUE" label in danger color
2. **Border Treatment**: Distinct colored border (danger color) around todo card
3. **Severity Indicator**: Enhanced styling for 30+ days overdue

### Rationale
- **Accessibility**: Combination approach ensures visibility for users with color blindness
- **User Preference**: Explicitly chosen during clarification session (Option C + D)
- **Material Design**: Aligns with existing Halloween-themed design system
- **Semantic**: Text label is screen-reader friendly

### Implementation Approach
**CSS Variables** (theme.css):
```css
:root {
  /* Existing colors */
  --danger: #c62828;          /* light mode */
  --danger-dark: #ef5350;     /* dark mode */
  
  /* New overdue-specific */
  --overdue-border: 3px;
  --overdue-severe-border: 4px;
  --overdue-badge-bg: var(--danger);
  --overdue-badge-text: #ffffff;
}
```

**Component Styling** (TodoCard.js):
```javascript
<div className={`todo-card ${isOverdue ? 'overdue' : ''} ${isSeverelyOverdue ? 'overdue-severe' : ''}`}>
  {isOverdue && <span className="overdue-badge">OVERDUE</span>}
  {/* todo content */}
</div>
```

**CSS Classes**:
```css
.todo-card.overdue {
  border: var(--overdue-border) solid var(--danger);
}

.todo-card.overdue-severe {
  border-width: var(--overdue-severe-border);
  border-style: double;
}

.overdue-badge {
  background: var(--overdue-badge-bg);
  color: var(--overdue-badge-text);
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
}
```

### Alternatives Considered
- **Icon only**: Not accessible for screen readers
- **Color change only**: Not accessible for color-blind users
- **Animation**: Out of scope per UI guidelines (minimal motion)

## 4. Archive Prompt Implementation

### Decision
Implement archive prompt as a simple confirmation dialog triggered when viewing todos overdue by 30+ days.

### Rationale
- **Simplicity**: Reuse existing ConfirmDialog component pattern
- **Non-intrusive**: Only shown on user action, not automatically
- **Scope**: Archive feature itself is out of scope; this is just a prompt

### Implementation Approach
```javascript
function TodoCard({ todo, onDelete, onArchive }) {
  const severityInfo = getOverdueSeverity(todo.dueDate, currentDate);
  
  const handleArchivePrompt = () => {
    if (window.confirm('This todo is significantly overdue. Would you like to archive it?')) {
      onArchive(todo.id);
    }
  };
  
  return (
    <div className="todo-card">
      {/* todo content */}
      {severityInfo.isSevere && (
        <button onClick={handleArchivePrompt} className="archive-prompt-btn">
          Archive?
        </button>
      )}
    </div>
  );
}
```

### Alternatives Considered
- **Automatic archiving**: Too aggressive, user may still want old todos visible
- **Dismissible banner**: More complex UI, adds state management
- **Batch archive**: Out of scope (no bulk operations per spec)

## 5. Testing Strategy for Date-Dependent Logic

### Decision
Use Jest's fake timers and fixed date mocking for deterministic testing.

### Rationale
- **Determinism**: Tests produce same results regardless of when run
- **Coverage**: Can test future dates, past dates, edge cases
- **Existing Tooling**: Jest already configured in project

### Implementation Approach
```javascript
describe('isOverdue', () => {
  beforeEach(() => {
    jest.useFakeTimers();
    jest.setSystemTime(new Date('2026-01-29'));
  });
  
  afterEach(() => {
    jest.useRealTimers();
  });
  
  test('returns true for yesterday', () => {
    expect(isOverdue('2026-01-28')).toBe(true);
  });
  
  test('returns false for today', () => {
    expect(isOverdue('2026-01-29')).toBe(false);
  });
  
  test('returns false for tomorrow', () => {
    expect(isOverdue('2026-01-30')).toBe(false);
  });
});
```

### Alternatives Considered
- **Real-time testing**: Flaky tests that fail at midnight
- **Passing dates as parameters**: Good for unit tests, but integration tests still need mocking
- **Time-travel library**: Unnecessary complexity for this use case

## 6. Performance Considerations

### Research Finding
Date comparison performance is not a concern for this feature.

### Analysis
- **Operation Complexity**: O(1) per todo item
- **Scale**: <1000 todos per user (per Technical Context)
- **Benchmark**: Creating Date object + comparison: ~5μs on modern browsers
- **Total Overhead**: 1000 todos × 5μs = 5ms (well under 100ms constraint)
- **Re-render Optimization**: Can use React.useMemo if needed

### Decision
No special optimization needed. If profiling reveals performance issues, add memoization:

```javascript
const isOverdue = useMemo(
  () => calculateIsOverdue(todo.dueDate, currentDate),
  [todo.dueDate, currentDate]
);
```

## 7. Edge Case: Completed Todo Marked Incomplete

### Decision
When a completed todo is marked incomplete and its due date is in the past, immediately apply overdue styling.

### Rationale
- **Consistency**: Overdue status is derived, not stored - recalculation happens automatically
- **User Expectation**: If they're un-completing an old task, they should see it's overdue
- **No Special Handling**: Existing logic covers this case

### Implementation
No special code needed. The `isOverdue()` utility function checks:
1. Is there a due date? (if not, return false)
2. Is the todo completed? (if yes, return false - checked before calling utility)
3. Is due date < current date? (if yes, return true)

This naturally handles the edge case.

## Summary

All research questions resolved. Key decisions:
1. ✅ Use native JavaScript Date objects for comparison
2. ✅ Add `/api/server-time` endpoint for hybrid time approach
3. ✅ Implement badge + border styling per user preference
4. ✅ Simple archive prompt for 30+ day overdue todos
5. ✅ Use Jest fake timers for deterministic date testing
6. ✅ No performance optimization needed (date comparison is fast)
7. ✅ Completed→incomplete edge case handled by existing logic

**Next Phase**: Proceed to Phase 1 (Data Model & Contracts)
