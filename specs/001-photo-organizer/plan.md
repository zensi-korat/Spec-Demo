# Implementation Plan: PhotoOrganizer

**Branch**: `001-photo-organizer` | **Date**: 2026-01-07 | **Spec**: [spec.md](spec.md)  
**Input**: Feature specification from `/specs/001-photo-organizer/spec.md`

## Summary

PhotoOrganizer is a frontend photo album manager enabling users to browse date-grouped albums, reorder them via drag-and-drop, manage photo memberships, and filter by date/tags. Technical approach: React 18 SPA with Tailwind styling, IndexedDB persistence via idb-keyval, react-beautiful-dnd for drag interactions, and Intersection Observer for lazy-loaded thumbnails. No backend; all data stored client-side with 5 sample albums pre-populated on first load.

## Technical Context

**Language/Version**: TypeScript 5.x (strict mode enabled) with React 18.2+  
**Primary Dependencies**: react-beautiful-dnd (drag-drop), date-fns (date formatting), idb-keyval (IndexedDB storage), lucide-react (icons)  
**Storage**: IndexedDB (via idb-keyval) for albums, photos, album order, and user preferences; no backend services  
**Testing**: Vitest + React Testing Library for component and hook unit tests  
**Target Platform**: Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+) supporting IndexedDB and Intersection Observer  
**Project Type**: Single-page application (SPA) - all source under `src/` at repository root  
**Performance Goals**: <2s initial load, <100ms drag feedback, <500ms filter application, <3s lazy-load complete  
**Constraints**: IndexedDB-only storage (no backend), ~50MB storage limit, mobile-first responsive design, WCAG AA contrast  
**Scale/Scope**: 5 sample albums, 75 total photos (10-20 per album), single-user local app

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

### Initial Check (Pre-Research)

✅ **Principle I - React + Tailwind + TypeScript Stack**: Feature uses React 18, Tailwind CSS, TypeScript strict mode (no plain JS)  
✅ **Principle II - Component Reusability**: shadcn/ui components in `/src/components/ui`, utilities in `/src/lib`  
✅ **Principle III - Responsive Design (Mobile-First)**: FR-011 specifies mobile (320px+), tablet (768px+), desktop (1024px+) breakpoints  
✅ **Principle IV - Accessibility (ARIA Standards)**: FR-012 (keyboard nav), FR-013 (ARIA labels/roles), SC-007 (WCAG AA contrast)  
✅ **Principle V - Performance & Lazy Loading**: FR-014 (lazy-load thumbnails), date-fns/idb-keyval are lightweight  
✅ **Technology Constraints - IndexedDB Only**: FR-004 specifies IndexedDB persistence, no backend mentioned  
✅ **Technology Constraints - Minimal Dependencies**: idb-keyval (~0.5KB), date-fns (tree-shakeable), Intersection Observer (native API)  
✅ **Development Workflow - Type Safety**: TypeScript strict mode enforced, no `any` types without justification

**Status**: ✅ ALL GATES PASSED - Feature aligns with all constitution principles

### Post-Design Check (After Phase 1)

✅ **Principle I**: Technical stack confirmed (React 18 + Vite + Tailwind + TypeScript strict)  
✅ **Principle II**: Reusable components identified: AlbumCard, PhotoThumbnail, DarkModeToggle, FilterPanel (all in `/src/components/ui`)  
✅ **Principle III**: Mobile-first grid layouts defined in data-model.md with Tailwind responsive classes  
✅ **Principle IV**: ARIA labels specified for all interactive elements (album cards, photo thumbnails, drag handles)  
✅ **Principle V**: Lazy loading via Intersection Observer (native API), code-splitting for album detail route  
✅ **Dependencies Review**: react-beautiful-dnd (12KB), date-fns functions (~5KB tree-shaken), idb-keyval (0.5KB) - all justified and minimal  
✅ **Type Safety**: All entities in data-model.md include TypeScript interfaces with strict types

**Status**: ✅ ALL GATES PASSED - Design adheres to constitution; no violations requiring justification

## Project Structure

### Documentation (this feature)

```text
specs/001-photo-organizer/
├── plan.md              # This file (Phase 1 output)
├── research.md          # Phase 0 output (technical decisions)
├── data-model.md        # Phase 1 output (entities, interfaces)
├── quickstart.md        # Phase 1 output (dev setup, sample data)
├── spec.md              # Feature specification (input)
└── checklists/
    └── requirements.md  # Specification quality checklist
```

### Source Code (repository root)

```text
src/
├── components/
│   ├── ui/              # Reusable UI components (shadcn/ui style)
│   │   ├── AlbumCard.tsx
│   │   ├── PhotoThumbnail.tsx
│   │   ├── DarkModeToggle.tsx
│   │   ├── FilterPanel.tsx
│   │   ├── Button.tsx   # shadcn button (already exists)
│   │   └── Card.tsx     # shadcn card (already exists)
│   ├── HomePage.tsx     # Album grid view
│   └── AlbumDetailPage.tsx  # Photo grid view
├── contexts/
│   ├── AlbumContext.tsx      # Album data and reorder logic
│   ├── PhotoContext.tsx      # Photo data and management
│   └── ThemeContext.tsx      # Dark mode state
├── hooks/
│   ├── useAlbums.ts          # Consume AlbumContext
│   ├── usePhotos.ts          # Consume PhotoContext
│   ├── useTheme.ts           # Consume ThemeContext
│   ├── useLazyImage.ts       # Intersection Observer for lazy load
│   └── useIndexedDB.ts       # idb-keyval wrapper
├── lib/
│   ├── utils.ts              # Existing utility (cn function)
│   ├── seedData.ts           # Generate 5 sample albums + 75 photos
│   ├── storage.ts            # IndexedDB operations via idb-keyval
│   └── dateUtils.ts          # date-fns helpers (format, parse, filter)
├── types/
│   └── index.ts              # TypeScript interfaces (Album, Photo, UserPreferences)
├── App.tsx                   # Root component with context providers
├── main.tsx                  # Vite entry point
└── index.css                 # Tailwind imports + CSS variables (already exists)

public/
└── images/                   # Placeholder images for failed loads
    └── image-placeholder.svg

tests/
├── components/
│   ├── AlbumCard.test.tsx
│   ├── PhotoThumbnail.test.tsx
│   └── FilterPanel.test.tsx
└── hooks/
    ├── useAlbums.test.ts
    └── useLazyImage.test.ts
```

**Structure Decision**: Single project structure selected (default). All source code under `src/` at repository root. Frontend-only app with no backend separation needed. Tailwind configuration, TypeScript setup, and Vite build tool already configured in existing project structure.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

_No complexity violations detected. All dependencies justified in research.md and align with constitution principles._

| Violation | Why Needed | Simpler Alternative Rejected Because |
| --------- | ---------- | ------------------------------------ |
| N/A       | N/A        | N/A                                  |
