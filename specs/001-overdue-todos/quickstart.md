# Quickstart: Overdue Todo Items

**Feature**: 001-overdue-todos  
**Date**: 2026-01-29  
**Phase**: Phase 1 - Implementation Guide

## Overview

This guide provides step-by-step instructions for implementing the overdue todos feature. Follow in order for systematic development.

## Prerequisites

- ✅ Node.js 16+ installed
- ✅ Repository cloned and dependencies installed (`npm install`)
- ✅ Existing todo app running (`npm run start`)
- ✅ Tests passing (`npm test`)

## Implementation Phases

### Phase 1: Backend - Server Time Endpoint (30 min)

#### Step 1.1: Add Server Time Endpoint
**File**: `packages/backend/src/app.js`

Add the new endpoint after existing todo routes:

```javascript
// Add after todo routes
app.get('/api/server-time', (req, res) => {
  try {
    res.json({ timestamp: Date.now() });
  } catch (error) {
    console.error('Error getting server time:', error);
    res.status(500).json({ error: 'Failed to retrieve server time' });
  }
});
```

**Test**: `curl http://localhost:3030/api/server-time`  
**Expected**: `{"timestamp":1738166400000}` (actual number will vary)

#### Step 1.2: Write Backend Tests
**File**: `packages/backend/__tests__/app.test.js`

Add test suite for new endpoint:

```javascript
describe('GET /api/server-time', () => {
  test('should return current server timestamp', async () => {
    const response = await request(app).get('/api/server-time');
    
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('timestamp');
    expect(typeof response.body.timestamp).toBe('number');
    expect(response.body.timestamp).toBeGreaterThan(0);
  });
  
  test('timestamp should be within reasonable range of current time', async () => {
    const before = Date.now();
    const response = await request(app).get('/api/server-time');
    const after = Date.now();
    
    const { timestamp } = response.body;
    expect(timestamp).toBeGreaterThanOrEqual(before);
    expect(timestamp).toBeLessThanOrEqual(after);
  });
});
```

**Run**: `npm run test:backend`  
**Expected**: All tests pass

---

### Phase 2: Frontend - Date Utilities (45 min)

#### Step 2.1: Create Date Utility File
**File**: `packages/frontend/src/utils/dateUtils.js`

```javascript
/**
 * Date utility functions for overdue todo calculations
 */

// Threshold for "severely overdue" status (days)
const OVERDUE_THRESHOLD_DAYS = 30;

/**
 * Checks if a todo is overdue based on its due date
 * @param {string|null} dueDate - ISO 8601 date string (YYYY-MM-DD)
 * @param {Date} currentDate - Current date for comparison
 * @returns {boolean} True if overdue (past due and not completed)
 */
export function isOverdue(dueDate, currentDate = new Date()) {
  if (!dueDate) return false;
  
  try {
    const due = new Date(dueDate);
    const current = new Date(currentDate);
    
    // Normalize to midnight for date-only comparison
    due.setHours(0, 0, 0, 0);
    current.setHours(0, 0, 0, 0);
    
    // Overdue if due date is strictly before current date
    return due < current;
  } catch (error) {
    console.error('Invalid due date:', dueDate, error);
    return false; // Treat invalid dates as not overdue
  }
}

/**
 * Calculates overdue severity information
 * @param {string|null} dueDate - ISO 8601 date string
 * @param {Date} currentDate - Current date for comparison
 * @returns {Object} Overdue status with severity info
 */
export function getOverdueSeverity(dueDate, currentDate = new Date()) {
  if (!dueDate) {
    return { isOverdue: false, isSevere: false, daysOverdue: null };
  }
  
  try {
    const due = new Date(dueDate);
    const current = new Date(currentDate);
    
    due.setHours(0, 0, 0, 0);
    current.setHours(0, 0, 0, 0);
    
    const diffMs = current.getTime() - due.getTime();
    const daysOverdue = Math.floor(diffMs / (1000 * 60 * 60 * 24));
    
    if (daysOverdue < 1) {
      return { isOverdue: false, isSevere: false, daysOverdue: null };
    }
    
    return {
      isOverdue: true,
      isSevere: daysOverdue >= OVERDUE_THRESHOLD_DAYS,
      daysOverdue
    };
  } catch (error) {
    console.error('Invalid due date:', dueDate, error);
    return { isOverdue: false, isSevere: false, daysOverdue: null };
  }
}
```

#### Step 2.2: Write Date Utility Tests
**File**: `packages/frontend/src/utils/__tests__/dateUtils.test.js`

```javascript
import { isOverdue, getOverdueSeverity } from '../dateUtils';

describe('dateUtils', () => {
  describe('isOverdue', () => {
    const testDate = new Date('2026-01-29');
    
    test('returns false for null due date', () => {
      expect(isOverdue(null, testDate)).toBe(false);
    });
    
    test('returns false for undefined due date', () => {
      expect(isOverdue(undefined, testDate)).toBe(false);
    });
    
    test('returns true for yesterday', () => {
      expect(isOverdue('2026-01-28', testDate)).toBe(true);
    });
    
    test('returns false for today', () => {
      expect(isOverdue('2026-01-29', testDate)).toBe(false);
    });
    
    test('returns false for tomorrow', () => {
      expect(isOverdue('2026-01-30', testDate)).toBe(false);
    });
    
    test('returns true for far past date', () => {
      expect(isOverdue('2025-01-01', testDate)).toBe(true);
    });
    
    test('returns false for invalid date string', () => {
      expect(isOverdue('invalid-date', testDate)).toBe(false);
    });
  });
  
  describe('getOverdueSeverity', () => {
    const testDate = new Date('2026-01-29');
    
    test('returns not overdue for null date', () => {
      const result = getOverdueSeverity(null, testDate);
      expect(result).toEqual({
        isOverdue: false,
        isSevere: false,
        daysOverdue: null
      });
    });
    
    test('returns overdue but not severe for 1 day', () => {
      const result = getOverdueSeverity('2026-01-28', testDate);
      expect(result.isOverdue).toBe(true);
      expect(result.isSevere).toBe(false);
      expect(result.daysOverdue).toBe(1);
    });
    
    test('returns not overdue for today', () => {
      const result = getOverdueSeverity('2026-01-29', testDate);
      expect(result.isOverdue).toBe(false);
    });
    
    test('returns severely overdue for 30 days', () => {
      const result = getOverdueSeverity('2025-12-30', testDate);
      expect(result.isOverdue).toBe(true);
      expect(result.isSevere).toBe(true);
      expect(result.daysOverdue).toBe(30);
    });
    
    test('returns severely overdue for more than 30 days', () => {
      const result = getOverdueSeverity('2025-12-01', testDate);
      expect(result.isOverdue).toBe(true);
      expect(result.isSevere).toBe(true);
      expect(result.daysOverdue).toBeGreaterThan(30);
    });
  });
});
```

**Run**: `npm run test:frontend`  
**Expected**: All new tests pass

---

### Phase 3: Frontend - Server Time Service (30 min)

#### Step 3.1: Add getServerTime Function
**File**: `packages/frontend/src/services/todoService.js`

Add to existing service:

```javascript
/**
 * Fetches current server timestamp for time synchronization
 * @returns {Promise<Date>} Server's current date/time
 */
export async function getServerTime() {
  try {
    const response = await fetch('/api/server-time');
    
    if (!response.ok) {
      throw new Error(`Server time fetch failed: ${response.status}`);
    }
    
    const { timestamp } = await response.json();
    return new Date(timestamp);
  } catch (error) {
    console.error('Failed to fetch server time, using client time:', error);
    return new Date(); // Fallback to client time
  }
}
```

#### Step 3.2: Write Service Tests
**File**: `packages/frontend/src/services/__tests__/todoService.test.js`

Add to existing test suite:

```javascript
describe('getServerTime', () => {
  test('fetches and returns server time as Date', async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: async () => ({ timestamp: 1738166400000 })
      })
    );
    
    const date = await getServerTime();
    
    expect(date).toBeInstanceOf(Date);
    expect(date.getTime()).toBe(1738166400000);
    expect(global.fetch).toHaveBeenCalledWith('/api/server-time');
  });
  
  test('falls back to client time on fetch error', async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({ ok: false, status: 500 })
    );
    
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
    const beforeFetch = Date.now();
    const date = await getServerTime();
    const afterFetch = Date.now();
    
    expect(date).toBeInstanceOf(Date);
    expect(date.getTime()).toBeGreaterThanOrEqual(beforeFetch);
    expect(date.getTime()).toBeLessThanOrEqual(afterFetch);
    expect(consoleSpy).toHaveBeenCalled();
    
    consoleSpy.mockRestore();
  });
});
```

**Run**: `npm run test:frontend`  
**Expected**: All tests pass

---

### Phase 4: Frontend - Styling (30 min)

#### Step 4.1: Add Overdue CSS Variables
**File**: `packages/frontend/src/styles/theme.css`

Add overdue-specific variables:

```css
:root {
  /* Existing danger colors */
  --danger: #c62828;
  --danger-dark: #ef5350;
  
  /* Overdue-specific variables */
  --overdue-border: 3px;
  --overdue-severe-border: 4px;
  --overdue-badge-bg: var(--danger);
  --overdue-badge-text: #ffffff;
  --overdue-border-color: var(--danger);
}

[data-theme="dark"] {
  --overdue-badge-bg: var(--danger-dark);
  --overdue-border-color: var(--danger-dark);
}
```

#### Step 4.2: Add Overdue CSS Classes
**File**: `packages/frontend/src/components/TodoCard.js` (add styles section)

```css
/* Overdue styling */
.todo-card.overdue {
  border: var(--overdue-border) solid var(--overdue-border-color);
}

.todo-card.overdue-severe {
  border-width: var(--overdue-severe-border);
  border-style: double;
}

.overdue-badge {
  display: inline-block;
  background: var(--overdue-badge-bg);
  color: var(--overdue-badge-text);
  padding: 2px 8px;
  margin-left: 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
}

.archive-prompt-btn {
  margin-top: 8px;
  padding: 4px 12px;
  background: transparent;
  border: 1px solid var(--text-secondary);
  border-radius: 4px;
  color: var(--text-secondary);
  cursor: pointer;
  font-size: 12px;
}

.archive-prompt-btn:hover {
  border-color: var(--primary);
  color: var(--primary);
}
```

---

### Phase 5: Frontend - Component Updates (60 min)

#### Step 5.1: Update TodoList Component
**File**: `packages/frontend/src/components/TodoList.js`

Add server time state and pass to cards:

```javascript
import { useState, useEffect } from 'react';
import { getServerTime } from '../services/todoService';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [currentDate, setCurrentDate] = useState(new Date());
  
  // Fetch server time on mount
  useEffect(() => {
    getServerTime().then(serverDate => {
      setCurrentDate(serverDate);
    });
  }, []);
  
  // ... existing todo fetch logic ...
  
  return (
    <div className="todo-list">
      {todos.map(todo => (
        <TodoCard 
          key={todo.id}
          todo={todo}
          currentDate={currentDate}
          onUpdate={handleUpdate}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

#### Step 5.2: Update TodoCard Component
**File**: `packages/frontend/src/components/TodoCard.js`

Add overdue display logic:

```javascript
import { useMemo } from 'react';
import { getOverdueSeverity } from '../utils/dateUtils';

function TodoCard({ todo, currentDate, onUpdate, onDelete }) {
  // Calculate overdue status (memoized for performance)
  const overdueStatus = useMemo(() => {
    if (todo.completed) {
      return { isOverdue: false, isSevere: false, daysOverdue: null };
    }
    return getOverdueSeverity(todo.dueDate, currentDate);
  }, [todo.dueDate, todo.completed, currentDate]);
  
  const handleArchivePrompt = () => {
    const confirmed = window.confirm(
      'This todo is significantly overdue. Would you like to archive it?'
    );
    if (confirmed) {
      // Archive functionality to be implemented (future feature)
      console.log('Archive requested for todo:', todo.id);
    }
  };
  
  const cardClasses = [
    'todo-card',
    overdueStatus.isOverdue && 'overdue',
    overdueStatus.isSevere && 'overdue-severe'
  ].filter(Boolean).join(' ');
  
  return (
    <div className={cardClasses}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onUpdate({ ...todo, completed: !todo.completed })}
      />
      
      <div className="todo-content">
        <span className="todo-title">
          {todo.title}
          {overdueStatus.isOverdue && (
            <span className="overdue-badge">Overdue</span>
          )}
        </span>
        
        {todo.dueDate && (
          <span className="todo-due-date">
            Due: {new Date(todo.dueDate).toLocaleDateString()}
          </span>
        )}
        
        {overdueStatus.isSevere && (
          <button 
            className="archive-prompt-btn"
            onClick={handleArchivePrompt}
          >
            Archive this old todo?
          </button>
        )}
      </div>
      
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
}
```

#### Step 5.3: Update TodoCard Tests
**File**: `packages/frontend/src/components/__tests__/TodoCard.test.js`

Add overdue-specific tests:

```javascript
import { render, screen } from '@testing-library/react';
import TodoCard from '../TodoCard';

describe('TodoCard - Overdue Status', () => {
  const mockOnUpdate = jest.fn();
  const mockOnDelete = jest.fn();
  const testDate = new Date('2026-01-29');
  
  test('displays overdue badge for past due date', () => {
    const overdueTodo = {
      id: '1',
      title: 'Test Todo',
      dueDate: '2026-01-28',
      completed: false
    };
    
    render(
      <TodoCard 
        todo={overdueTodo} 
        currentDate={testDate}
        onUpdate={mockOnUpdate}
        onDelete={mockOnDelete}
      />
    );
    
    expect(screen.getByText('Overdue')).toBeInTheDocument();
  });
  
  test('does not display overdue badge for today', () => {
    const todayTodo = {
      id: '1',
      title: 'Test Todo',
      dueDate: '2026-01-29',
      completed: false
    };
    
    render(
      <TodoCard 
        todo={todayTodo} 
        currentDate={testDate}
        onUpdate={mockOnUpdate}
        onDelete={mockOnDelete}
      />
    );
    
    expect(screen.queryByText('Overdue')).not.toBeInTheDocument();
  });
  
  test('does not display overdue badge for completed todos', () => {
    const completedTodo = {
      id: '1',
      title: 'Test Todo',
      dueDate: '2026-01-28',
      completed: true
    };
    
    render(
      <TodoCard 
        todo={completedTodo} 
        currentDate={testDate}
        onUpdate={mockOnUpdate}
        onDelete={mockOnDelete}
      />
    );
    
    expect(screen.queryByText('Overdue')).not.toBeInTheDocument();
  });
  
  test('shows archive prompt for severely overdue todos', () => {
    const severelyOverdueTodo = {
      id: '1',
      title: 'Test Todo',
      dueDate: '2025-12-29', // 31 days ago
      completed: false
    };
    
    render(
      <TodoCard 
        todo={severelyOverdueTodo} 
        currentDate={testDate}
        onUpdate={mockOnUpdate}
        onDelete={mockOnDelete}
      />
    );
    
    expect(screen.getByText(/Archive this old todo/)).toBeInTheDocument();
  });
});
```

**Run**: `npm run test:frontend`  
**Expected**: All tests pass

---

## Verification Checklist

After completing all phases:

### Backend
- [ ] `GET /api/server-time` endpoint returns timestamp
- [ ] Backend tests pass (`npm run test:backend`)
- [ ] Server time is within reasonable range of actual time

### Frontend - Utilities
- [ ] `dateUtils.js` exports `isOverdue` and `getOverdueSeverity`
- [ ] All date utility tests pass
- [ ] Edge cases handled (null dates, invalid dates, today's date)

### Frontend - Service
- [ ] `getServerTime()` fetches server timestamp successfully
- [ ] Falls back to client time on error
- [ ] Service tests pass

### Frontend - UI
- [ ] Overdue todos show "OVERDUE" badge
- [ ] Overdue todos have distinct border
- [ ] Severely overdue todos show archive prompt
- [ ] Completed todos never show overdue styling
- [ ] Todos without due dates never show overdue styling
- [ ] Component tests pass

### Integration
- [ ] Full app runs without errors (`npm run start`)
- [ ] All tests pass (`npm test`)
- [ ] Coverage meets 80%+ threshold
- [ ] Manual testing: create todo with past date → shows as overdue
- [ ] Manual testing: complete overdue todo → overdue styling disappears

## Common Issues & Solutions

**Issue**: Overdue badge not showing  
**Solution**: Check that `currentDate` prop is being passed to TodoCard and `isOverdue()` is returning true

**Issue**: Tests failing with date comparison  
**Solution**: Use fixed dates in tests, not `new Date()` which changes

**Issue**: Server time fetch fails  
**Solution**: Check backend is running on port 3030, check CORS configuration

**Issue**: Styling not applied  
**Solution**: Verify CSS classes are in theme.css and className is correctly applied

## Next Steps

After implementation is complete:
1. Run full test suite: `npm test`
2. Check coverage report: `npm run test:frontend -- --coverage`
3. Manual testing: Test all acceptance scenarios from spec.md
4. Code review: Ensure constitution principles are followed
5. Proceed to `/speckit.tasks` for task breakdown

## Estimated Time

- Phase 1 (Backend): 30 min
- Phase 2 (Date Utils): 45 min
- Phase 3 (Service): 30 min
- Phase 4 (Styling): 30 min
- Phase 5 (Components): 60 min
- Testing & Verification: 45 min

**Total**: ~4 hours for implementation + testing
