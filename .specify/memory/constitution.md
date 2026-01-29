<!--
SYNC IMPACT REPORT
==================
Version: NONE → 1.0.0
Change: Initial constitution ratification
Bump Rationale: MAJOR version for initial adoption of governance framework

Modified Principles:
- All principles newly established from existing documentation

Added Sections:
1. Core Principles (7 principles)
2. Code Quality Standards
3. Development Workflow
4. Governance

Templates Status:
✅ plan-template.md - Constitution Check section aligns with new principles
✅ spec-template.md - User Stories and Requirements sections align with testing and functional requirement principles
✅ tasks-template.md - Task categorization reflects principle-driven development (test-first, independent testing)

Follow-up TODOs: None
-->

# Copilot Bootcamp Todo App Constitution

## Core Principles

### I. Code Quality and Maintainability

**MUST**: Follow DRY (Don't Repeat Yourself), KISS (Keep It Simple), and SOLID principles in all code
**MUST**: Extract common code into shared functions and utilities - no repetition across modules
**MUST**: Each module, component, or function has a single, well-defined responsibility
**MUST**: Use descriptive naming (camelCase for variables/functions, PascalCase for components/classes, UPPER_SNAKE_CASE for constants)
**MUST**: Remove all trailing whitespace and use 2-space indentation consistently
**MUST**: Keep line length under 100 characters for code readability

**Rationale**: Maintainable code reduces technical debt, accelerates feature development, and enables team collaboration. These standards are derived from established engineering best practices and ensure consistency across the monorepo structure.

### II. Test-First Development (NON-NEGOTIABLE)

**MUST**: Write tests for new features as part of the development process
**MUST**: Achieve 80%+ code coverage across all packages (frontend and backend)
**MUST**: Test behavior, not implementation details
**MUST**: Each test must be independent with clear setup, action, and assertion (AAA pattern)
**MUST**: Use descriptive test names that explain what is being tested
**MUST**: Mock all external dependencies in unit tests

**Rationale**: Comprehensive testing ensures reliability, prevents regressions, and serves as living documentation. Test-first development catches issues early and validates requirements before implementation.

### III. Component Isolation and Reusability

**MUST**: Build UI components that are reusable across the application
**MUST**: Components receive data and behavior through props, not global state or hardcoded values
**MUST**: Follow React component contract consistently - no breaking of expected prop interfaces
**MUST**: Keep prop lists focused and minimal (Interface Segregation Principle)
**MUST**: File names must match component names (PascalCase)

**Rationale**: Isolated, reusable components reduce duplication, improve testability, and enable independent development of features. This aligns with the monorepo architecture and component-based design system.

### IV. Error Handling and User Feedback

**MUST**: Include try-catch blocks around all operations that can fail
**MUST**: Provide clear, actionable error messages to users
**MUST**: Log errors with context information for debugging
**MUST**: Inform users when operations fail with appropriate UI feedback
**MUST**: Validate input data at API boundaries

**Rationale**: Graceful error handling improves user experience and system reliability. Proper logging and validation prevent silent failures and enable rapid debugging.

### V. Import Organization and Module Structure

**MUST**: Organize imports in this order: (1) External libraries, (2) Internal modules, (3) Styles
**MUST**: Use named imports for multiple exports, default imports for single exports
**MUST**: Use relative paths for internal modules
**MUST**: No circular dependencies
**MUST**: Separate import groups with blank lines

**Rationale**: Consistent import organization improves code readability and prevents dependency issues. Clear module boundaries support the monorepo workspace structure.

### VI. Performance and Optimization

**MUST**: Avoid unnecessary React renders using appropriate hooks (useMemo, useCallback)
**MUST**: Load components and data lazily when appropriate
**MUST**: Use efficient algorithms and data structures
**MUST**: Keep component and bundle sizes reasonable
**MUST**: Optimize only when necessary - write clear code first

**Rationale**: Performance optimization should be deliberate and measurable. Premature optimization reduces code clarity; optimization should be applied where it delivers measurable user benefit.

### VII. Documentation and Comments

**MUST**: Comment only "why" the code does something, never "what" it does
**MUST**: Keep comments updated when code changes
**MUST**: Remove obvious comments that describe self-evident code
**MUST**: Use JSDoc for public functions and components with type annotations and descriptions
**MUST**: Include rationale for non-obvious design decisions

**Rationale**: Meaningful documentation explains intent and context that code alone cannot convey. Outdated or obvious comments create maintenance burden without adding value.

## Code Quality Standards

### Formatting and Style
- **Indentation**: 2 spaces for all file types (JavaScript, JSON, CSS, Markdown)
- **Line Endings**: LF (Unix-style) only
- **File Structure**: Imports → Constants → Utility functions → Main component/class → Helpers → Exports

### Naming Conventions Enforcement
- Variables and functions: `getUserId`, `calculateTotalPrice`, `isActive`
- Constants: `MAX_RETRIES`, `API_BASE_URL`, `DEFAULT_TIMEOUT`
- React components: `TodoCard.js`, `TodoList.js` (file names match component names)
- Classes: `ApiService`, `TodoRepository`

### Linting and Quality Gates
- ESLint rules must pass before commits
- No unused variables or undefined variables
- No console statements in production code (warnings only)
- Proper error handling enforced

### File Organization Requirements
- **Frontend**: `src/components/`, `src/services/`, `src/utils/`, `src/__tests__/`
- **Backend**: `src/routes/`, `src/controllers/`, `src/services/`, `src/__tests__/`
- Tests colocated in `__tests__/` directories next to source files
- Test files named `{filename}.test.js`

## Development Workflow

### Git Practices
- **Atomic Commits**: Each commit represents one logical change
- **Clear Commit Messages**: Explain the "why" of changes, not just "what"
- **Feature Branches**: Use pattern `feature/descriptive-name` for new work
- **Pull Requests**: Required for code review before merging to main branch
- **Example Format**: `feat: add ability to edit todo title and due date`

### Code Review Checklist
Before submitting code for review, verify:
- [ ] Code follows naming conventions
- [ ] Imports are organized correctly
- [ ] No linting errors or warnings
- [ ] Code is DRY and avoids repetition
- [ ] Functions/components have single responsibility
- [ ] Error handling is implemented
- [ ] Comments are clear and helpful
- [ ] Tests are written for new functionality (80%+ coverage)
- [ ] Git commits are atomic and well-described
- [ ] No console.log statements left in production code

### Testing Workflow
- Run tests locally before committing: `npm test`
- Run specific package tests: `npm test --workspace=packages/frontend`
- Generate coverage reports: `npm test -- --coverage`
- All tests must pass before opening pull requests
- Critical user workflows require 100% test coverage

### Test Organization
- Unit tests: Test individual components/functions in isolation
- Integration tests: Test component interactions and API communication
- Tests colocated with source code in `__tests__/` directories
- Use fixtures and mock data in `__mocks__/` directories for consistency
- Follow AAA pattern: Arrange → Act → Assert

## Governance

### Amendment Process
This constitution supersedes all other development practices and guidelines. Amendments require:
1. Documented rationale and impact analysis
2. Review and approval from project maintainers
3. Migration plan for affected code if breaking changes introduced
4. Version bump following semantic versioning rules:
   - MAJOR: Backward incompatible governance changes or principle removals/redefinitions
   - MINOR: New principles/sections added or materially expanded guidance
   - PATCH: Clarifications, wording fixes, non-semantic refinements

### Compliance and Enforcement
- All pull requests must be reviewed for constitutional compliance
- Complexity and deviations must be justified with documented rationale
- Regular reviews to ensure continued alignment with project goals
- Templates in `.specify/templates/` must align with constitutional principles

### Runtime Development Guidance
- Refer to `docs/coding-guidelines.md` for detailed coding standards
- Refer to `docs/testing-guidelines.md` for comprehensive testing practices
- Refer to `docs/functional-requirements.md` for feature specifications
- Refer to `docs/ui-guidelines.md` for design system and UI standards
- Refer to `.github/copilot-instructions.md` for AI assistant context

**Version**: 1.0.0 | **Ratified**: 2026-01-29 | **Last Amended**: 2026-01-29
