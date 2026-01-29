# Implementation Plan: Overdue Todo Items

**Branch**: `001-overdue-todos` | **Date**: 2026-01-29 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-overdue-todos/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Users need visual identification of todos that are past their due date. The system will add overdue status calculation (comparing todo due date to current date) and apply visual styling (text badge + border treatment) to incomplete todos with past due dates. A hybrid approach uses server time on initial load and client time for UI updates. Enhanced severity indicators and archive prompts appear for todos overdue by 30+ days.

## Technical Context

**Language/Version**: JavaScript (Node.js 16+, ES6+ for frontend)  
**Primary Dependencies**: React 18.2.0, Express.js 4.18.2, axios 1.6.2  
**Storage**: better-sqlite3 11.10.0 (SQLite in-memory database)  
**Testing**: Jest 29.7+ with React Testing Library, supertest for backend  
**Target Platform**: Web application (browser + Node.js server)  
**Project Type**: Web (monorepo with frontend/backend workspaces)  
**Performance Goals**: Date comparison <1ms per todo, UI updates <100ms  
**Constraints**: Client-side date calculation must not block rendering  
**Scale/Scope**: Single-user todo app, expected <1000 todos per user

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Code Quality and Maintainability ✅
- Extract date comparison logic into shared utility function (DRY)
- Single responsibility: TodoCard renders, utility calculates overdue status
- Descriptive naming: `isOverdue()`, `getOverdueSeverity()`, `OVERDUE_THRESHOLD_DAYS`
- 2-space indentation, <100 char lines, no trailing whitespace

### II. Test-First Development ✅
- Write tests for overdue calculation logic before implementation
- Test TodoCard rendering with overdue styling
- Test edge cases: today's date, no due date, completed todos
- Target 80%+ coverage for new utility functions and component updates
- Mock Date.now() for deterministic testing

### III. Component Isolation and Reusability ✅
- TodoCard receives overdue status via props (or calculates from props)
- Date utility is pure function, easily testable
- No global state for current date - passed as prop or calculated
- Keep TodoCard interface minimal - only add necessary overdue-related props

### IV. Error Handling and User Feedback ✅
- Gracefully handle invalid/null due dates (treat as no due date)
- Handle server time fetch failures (fallback to client time)
- No user-facing errors expected (date comparison is safe operation)

### V. Import Organization and Module Structure ✅
- New utility: `src/utils/dateUtils.js` (frontend)
- Import order: React → Internal utils → Styles
- No circular dependencies introduced

### VI. Performance and Optimization ✅
- Date comparison is O(1) operation - no optimization needed
- useMemo for overdue calculation if TodoCard re-renders frequently
- No lazy loading required for this feature

### VII. Documentation and Comments ✅
- JSDoc for `isOverdue()` and `getOverdueSeverity()` functions
- Comment rationale for 30-day threshold
- Explain hybrid time approach (server + client) in code comments

**GATES RESULT**: ✅ PASS - No violations. Feature aligns with all constitution principles.

---

### POST-PHASE 1 RE-CHECK ✅

**Data Model**: No schema changes, derived state only - aligns with KISS principle ✅

**API Contract**: Single simple endpoint, minimal payload, clear error handling ✅

**Component Design**: TodoCard remains focused, date utilities are pure functions, reusable across app ✅

**Testing Strategy**: Comprehensive test coverage planned (backend, utilities, services, components) ✅

**Documentation**: Clear quickstart guide, API contracts, data model documentation ✅

**FINAL RESULT**: ✅ ALL GATES PASS - Ready to proceed to Phase 2 (Task Breakdown)

## Project Structure

### Documentation (this feature)

```text
specs/001-overdue-todos/
├── spec.md              # Feature specification (already exists)
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
│   └── api.md           # API contract for server time endpoint
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
packages/
├── backend/
│   ├── src/
│   │   ├── app.js                  # Add /api/server-time endpoint
│   │   ├── index.js
│   │   └── services/
│   │       └── todoService.js      # Existing - no changes needed
│   └── __tests__/
│       └── app.test.js             # Add tests for server-time endpoint
│
└── frontend/
    ├── src/
    │   ├── components/
    │   │   ├── TodoCard.js         # Update: render overdue styling
    │   │   ├── TodoList.js         # Update: pass server time to cards
    │   │   └── __tests__/
    │   │       ├── TodoCard.test.js    # Add overdue styling tests
    │   │       └── TodoList.test.js    # Update tests
    │   ├── services/
    │   │   └── todoService.js      # Add getServerTime() function
    │   ├── utils/
    │   │   ├── dateUtils.js        # NEW: overdue calculation logic
    │   │   └── __tests__/
    │   │       └── dateUtils.test.js   # NEW: date utility tests
    │   └── styles/
    │       └── theme.css            # Add overdue styling variables
    └── __tests__/
        └── App.test.js
```

**Structure Decision**: Web application monorepo (Option 2). The existing project already uses this structure with separate `packages/backend/` and `packages/frontend/` workspaces. This feature adds utility functions, updates existing components, and adds one new backend endpoint. No new packages or major structural changes required.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations - this section is not applicable.
