<!--
Sync Impact Report
─────────────────────────────────────────────────────────────────────
Version change: 0.0.0 → 1.0.0 (MAJOR: initial ratification)
Modified principles:
  - Added: I. React + Tailwind + TypeScript Stack
  - Added: II. Component Reusability
  - Added: III. Responsive Design (Mobile-First)
  - Added: IV. Accessibility (ARIA Standards)
  - Added: V. Performance & Lazy Loading
Added sections:
  - Technology Constraints
  - Development Workflow
Removed sections: None
Templates requiring updates:
  ✅ .specify/templates/plan-template.md - No update needed (generic template)
  ✅ .specify/templates/spec-template.md - No update needed (generic template)
  ✅ .specify/templates/tasks-template.md - No update needed (generic template)
Follow-up TODOs:
  - TODO(RATIFICATION_DATE): Set original adoption date when project formally adopts this constitution
─────────────────────────────────────────────────────────────────────
-->

# Spec-Demo Constitution

## Core Principles

### I. React + Tailwind + TypeScript Stack

All UI development MUST use React 18+ as the framework, Tailwind CSS for styling, and TypeScript with strict mode enabled. No plain JavaScript files are permitted in the codebase.

**Rationale**: Enforces type safety, modern component patterns, and utility-first styling for maintainability and developer experience.

### II. Component Reusability

Components MUST be designed for reuse across the application. Shared UI components belong in `/src/components/ui` following shadcn/ui conventions. Business logic MUST be extracted to `/src/lib` utilities when used by multiple components.

**Rationale**: Reduces duplication, ensures consistency, and accelerates feature development through composable building blocks.

### III. Responsive Design (Mobile-First)

All layouts MUST be designed mobile-first using Tailwind's responsive breakpoints. Desktop views are enhancements, not the baseline. Test on mobile viewports before desktop.

**Rationale**: Mobile traffic is primary; mobile-first design ensures usable experiences across all device sizes.

### IV. Accessibility (ARIA Standards)

Interactive elements MUST include appropriate ARIA labels, roles, and keyboard navigation support. Color contrast MUST meet WCAG AA standards minimum. No decorative images without alt text.

**Rationale**: Ensures the application is usable by everyone, including users with disabilities, and meets legal accessibility requirements.

### V. Performance & Lazy Loading

Non-critical code MUST be lazy-loaded using dynamic imports and React.lazy. Bundle size MUST be monitored; avoid heavy third-party libraries. Prefer vanilla JavaScript utilities over library dependencies.

**Rationale**: Faster initial load times and better user experience, especially on slower networks and devices.

## Technology Constraints

**Data Storage**: No backend services. All data persistence MUST use IndexedDB for client-side storage. Consider localStorage only for small, non-critical data.

**Dependencies**: Avoid heavy utility libraries (lodash, moment, etc.). Write vanilla JavaScript/TypeScript helpers in `/src/lib` instead. Only add dependencies that provide significant value and cannot be easily implemented.

**Styling**: Tailwind CSS is the primary styling solution. CSS variables defined in the theme manage colors and spacing. Avoid writing custom CSS unless absolutely necessary.

## Development Workflow

**Type Safety**: All code MUST pass TypeScript strict checks before commit. No `any` types without explicit justification and approval.

**Code Quality**: Lint (ESLint) and format (Prettier) checks MUST pass in CI/CD. Use absolute imports (`@/`) for all internal modules.

**Component Structure**: Follow shadcn/ui patterns for component organization. Export named components with proper TypeScript interfaces.

**Performance Review**: Identify and document lazy-load boundaries for routes and heavy components. Review bundle impact before merging new dependencies.

## Governance

This constitution supersedes all other development practices and guides for the Spec-Demo project. All feature specifications, implementation plans, and code reviews MUST verify compliance with these principles.

**Amendments**: Changes to this constitution require proposal documentation, team review, and explicit version bump. Breaking changes (removing or contradicting principles) increment MAJOR version. New principles or expanded guidance increment MINOR version. Clarifications and typo fixes increment PATCH version.

**Compliance**: Pull requests that violate principles require documented exceptions with clear justification and sunset timeline. Complexity that conflicts with principles (e.g., heavy dependencies) must be justified in plan documents.

**Versioning Policy**: This document follows semantic versioning (MAJOR.MINOR.PATCH).

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE) | **Last Amended**: 2026-01-07
