<!--
Sync Impact Report:
- Version: 0.0.0 → 1.0.0 (Initial ratification)
- Principles Added: 7 principles across 3 categories (Engineering, Product, UX/UI)
- Sections Added: AI Collaboration Rules, Technology Constraints
- Templates: All templates assumed valid (new constitution initialization)
- Deferred Items: None
-->

# Bookmark Manager Constitution

## I. Engineering Principles

### 1. Code Quality (NON-NEGOTIABLE)

All code MUST meet the following standards:

- **TypeScript strict mode**: No implicit `any`, strict null checks, no unused variables
- **ESLint + Prettier**: Zero warnings in CI/CD pipeline
- **Code reviews mandatory**: Minimum one approval before merge
- **Self-documenting code**: Variable/function names clearly express intent; comments only for "why", not "what"

**Rationale**: Quality gates prevent technical debt accumulation and ensure maintainability over time.

### 2. Readability & Maintainability

Code MUST prioritize clarity:

- **Single Responsibility Principle**: Each function/component does one thing well
- **File size limits**: Components <200 lines; utilities <150 lines
- **Cyclomatic complexity**: Max 10 branches per function
- **DRY principle**: No logic duplicated more than twice without abstraction

**Rationale**: Readable code reduces onboarding time and minimizes bugs during maintenance.

### 3. Testing Discipline

Testing MUST be comprehensive:

- **Unit test coverage**: Minimum 80% for business logic and utilities
- **Component testing**: All interactive UI components tested with React Testing Library
- **Integration tests**: Critical user flows (add bookmark, search, sync) covered
- **Edge case handling**: Empty states, network errors, quota limits tested

**Rationale**: High test coverage catches regressions early and enables confident refactoring.

---

## II. Product Principles

### 4. Mobile-First Design

All features MUST be designed mobile-first:

- **Responsive breakpoints**: sm (640px), md (768px), lg (1024px) using Tailwind
- **Touch targets**: Minimum 44×44px for all interactive elements
- **Mobile testing**: Verify features on iOS Safari and Android Chrome before desktop
- **Progressive enhancement**: Core functionality works on mobile; desktop adds convenience features

**Rationale**: Mobile devices represent the majority of web traffic; degraded mobile experience is unacceptable.

### 5. Offline-First Behavior

Application MUST function without network:

- **Local-first storage**: IndexedDB stores all bookmarks, folders, tags
- **Service Worker caching**: Static assets and app shell cached for offline access
- **Optimistic UI updates**: UI reflects changes immediately; sync happens in background
- **Sync conflict resolution**: Last-write-wins with timestamp-based merge strategy

**Rationale**: Users expect bookmark access regardless of connectivity; network failures should not block productivity.

### 6. Performance Standards

Performance MUST meet measurable targets:

- **Initial load**: Time-to-Interactive (TTI) <2 seconds on 4G connection
- **Search response**: Results displayed <200ms after keystroke
- **Bulk operations**: Adding/deleting 100 bookmarks completes <1 second
- **Lighthouse scores**: Performance >90, Accessibility >95

**Rationale**: Fast interfaces feel responsive; slow interfaces feel broken. Performance is a feature.

---

## III. UX / UI Principles

### 7. Accessibility (WCAG 2.1 AA)

All interfaces MUST be accessible:

- **Keyboard navigation**: All features operable via keyboard (Tab, Enter, Escape, Arrow keys)
- **ARIA labels**: All interactive elements have descriptive labels
- **Color contrast**: Minimum 4.5:1 for normal text, 3:1 for large text
- **Screen reader testing**: Verify with VoiceOver (macOS) or NVDA (Windows) before release

**Rationale**: Accessibility is not optional; 15% of users rely on assistive technologies.

### 8. Dark/Light Mode Parity

Both themes MUST be first-class:

- **Feature parity**: All UI elements styled consistently in both modes
- **System preference**: Respect `prefers-color-scheme` media query on initial load
- **Manual override**: User preference stored in localStorage/IndexedDB
- **Contrast validation**: Both themes pass WCAG AA contrast ratios

**Rationale**: Users expect seamless switching; broken dark mode degrades user experience.

### 9. Touch-Friendly Interactions

Mobile interactions MUST be intuitive:

- **Swipe gestures**: Swipe right to archive, swipe left to delete (with visual feedback)
- **Long-press menus**: Hold bookmark card to reveal context menu
- **Drag-and-drop**: Reorder bookmarks/folders via touch drag (react-beautiful-dnd)
- **Pull-to-refresh**: Standard gesture to sync latest bookmarks

**Rationale**: Touch interfaces require gestural affordances beyond point-and-click paradigms.

---

## IV. AI Collaboration Rules

When working with AI assistants (GitHub Copilot, Claude, etc.), the following rules apply:

### 10. Specification Authority

AI MUST NOT invent features:

- **Specs are source of truth**: Only implement requirements explicitly documented in `specs/*/spec.md`
- **No assumptions**: If requirement is ambiguous, ask clarifying question before proceeding
- **No scope creep**: Resist temptation to add "nice-to-have" features not in spec

**Rationale**: Unplanned features introduce complexity, testing burden, and maintenance overhead.

### 11. Clarification Protocol

AI MUST seek clarity:

- **Ambiguous requirements**: Ask user to resolve conflicts or missing details
- **Design decisions**: Present 2-3 options with tradeoffs when multiple valid approaches exist
- **Technical constraints**: Verify assumptions about browser APIs, library capabilities, performance limits

**Rationale**: Guessing leads to rework; asking prevents wasted implementation effort.

### 12. Pattern Consistency

AI MUST follow existing conventions:

- **Component structure**: Match shadcn/ui patterns for reusable UI components
- **Hook naming**: Custom hooks prefixed with `use` (e.g., `useBookmarks`, `useSync`)
- **File organization**: Follow established folder structure (`src/components/`, `src/hooks/`, `src/lib/`)
- **Error handling**: Use consistent error boundaries and fallback UI patterns

**Rationale**: Consistency reduces cognitive load; developers should recognize patterns instantly.

---

## V. Technology Constraints

### Storage

- **Primary storage**: IndexedDB via `idb-keyval` for bookmarks, folders, tags, user preferences
- **Cache storage**: Service Worker caches for static assets and app shell
- **Quota management**: Monitor IndexedDB quota; warn users at 80% capacity

### Dependencies

- **React 18+**: Use concurrent features (Suspense, Transitions) where beneficial
- **Vite**: Development server and build tool
- **Tailwind CSS v4**: Utility-first styling with CSS variables for theming
- **TypeScript 5+**: Strict mode enabled
- **No heavy libraries**: Avoid lodash, moment.js; prefer native JS/date-fns when needed

### Architecture

- **Single-page application (SPA)**: Client-side routing with React Router
- **State management**: React Context for global state; no Redux/Zustand unless complexity justifies
- **Component library**: shadcn/ui for base components (Button, Card, Dialog, etc.)

---

## VI. Development Workflow

### Type Safety

- TypeScript strict checks MUST pass before commit
- No `any` types without documented justification
- All API responses typed with interfaces

### Code Quality

- ESLint + Prettier run in pre-commit hook
- Zero warnings in CI/CD build
- Use absolute imports (`@/components/...`) for better refactoring

### Testing

- Write unit tests before implementation (TDD preferred for complex logic)
- Component tests verify user interactions, not implementation details
- Integration tests cover critical paths (add bookmark, search, sync)

### Performance Monitoring

- Run Lighthouse audits before every release
- Monitor bundle size; flag increases >10KB without justification
- Use React DevTools Profiler to identify slow renders

---

## Governance

This constitution supersedes all other development practices. When conflicts arise between constitution principles and external guidance (blog posts, library recommendations, AI suggestions), constitution rules prevail.

### Amendment Process

1. **Proposal**: Document proposed change with rationale in GitHub issue
2. **Review**: Discuss tradeoffs with team/stakeholders
3. **Approval**: Minimum one approval required
4. **Version bump**: Apply semantic versioning (see below)
5. **Migration**: Update affected code/docs to comply within 2 sprints

### Versioning

- **MAJOR** (X.0.0): Breaking changes (principle removed, constraint violated by new rule)
- **MINOR** (0.X.0): New principle added or existing principle materially expanded
- **PATCH** (0.0.X): Clarifications, typo fixes, non-semantic wording improvements

### Compliance

- All pull requests MUST pass constitution checks (automated via CI/CD where possible)
- Code reviews MUST verify adherence to principles
- Violations require documented exception with expiration date

### Runtime Guidance

For agent-specific development instructions, see:

- GitHub Copilot: `.github/agents/copilot-instructions.md`
- Claude/other agents: `.specify/memory/agent-context.md`

---

**Version**: 1.0.0 | **Ratified**: 2026-01-08 | **Last Amended**: 2026-01-08
