# PhotoOrganizer Quickstart Guide

**Feature**: PhotoOrganizer  
**Created**: 2026-01-07  
**Purpose**: Developer setup, initialization, and sample data overview

---

## Prerequisites

- **Node.js**: v18.x or later
- **npm**: v9.x or later
- **Browser**: Modern browser with IndexedDB support (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)

---

## Installation

### 1. Install Dependencies

From the project root directory (`/Users/zensi/Spec-Demo`), run:

```bash
npm install
```

This installs existing dependencies from `package.json`:

- React 19.2.0 (compatible with React 18 APIs)
- Vite 7.2.4
- TypeScript 5.9.3
- Tailwind CSS 3.4.19
- shadcn/ui components (Button, Card)
- lucide-react icons

### 2. Install PhotoOrganizer-Specific Dependencies

Run the following command to add feature-specific libraries:

```bash
npm install react-beautiful-dnd date-fns idb-keyval
```

Install type definitions for react-beautiful-dnd:

```bash
npm install -D @types/react-beautiful-dnd
```

Install testing libraries:

```bash
npm install -D vitest @vitest/ui @testing-library/react @testing-library/jest-dom jsdom
```

**Dependency Summary**:

| Package                     | Version | Purpose                                                        |
| --------------------------- | ------- | -------------------------------------------------------------- |
| `react-beautiful-dnd`       | Latest  | Drag-and-drop for album reordering (touch-enabled, accessible) |
| `date-fns`                  | Latest  | Date formatting, parsing, and filtering (tree-shakeable)       |
| `idb-keyval`                | Latest  | IndexedDB wrapper (~0.5KB, promise-based key-value store)      |
| `vitest`                    | Latest  | Unit testing framework (Vite-native)                           |
| `@testing-library/react`    | Latest  | React component testing utilities                              |
| `@testing-library/jest-dom` | Latest  | Custom DOM matchers for assertions                             |
| `jsdom`                     | Latest  | DOM environment for Vitest                                     |

---

## Configuration

### Vitest Setup

Create `vitest.config.ts` in project root:

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: "./src/test/setup.ts",
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

Create `src/test/setup.ts`:

```typescript
import "@testing-library/jest-dom";
```

### Update `package.json` Scripts

Add testing scripts to `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "preview": "vite preview",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui"
  }
}
```

---

## Sample Data Overview

PhotoOrganizer initializes with **5 sample albums** containing **75 total photos**.

### Album Structure

| Album ID    | Title   | Created Date | Photo Count | Photos Range           |
| ----------- | ------- | ------------ | ----------- | ---------------------- |
| `album-001` | 2026-01 | 2026-01-15   | 15          | photo-001 to photo-015 |
| `album-002` | 2025-12 | 2025-12-15   | 12          | photo-016 to photo-027 |
| `album-003` | 2025-11 | 2025-11-15   | 18          | photo-028 to photo-045 |
| `album-004` | 2025-10 | 2025-10-15   | 10          | photo-046 to photo-055 |
| `album-005` | 2025-09 | 2025-09-15   | 20          | photo-056 to photo-075 |

### Photo Tag Distribution

Photos are tagged with combinations of the following:

- **vacation** (20 photos): travel/holiday photos
- **family** (18 photos): family gatherings and portraits
- **nature** (15 photos): landscapes, wildlife, outdoor scenes
- **architecture** (12 photos): buildings, structures, cityscapes
- **food** (10 photos): meals, recipes, culinary scenes

### Photo URL Format

Photos use **Unsplash Source API** for placeholder images:

- **Thumbnail URL**: `https://source.unsplash.com/random/400x300?sig=photo-{id}`
- **Full-size URL**: `https://source.unsplash.com/random/1920x1080?sig=photo-{id}`

The `sig` parameter ensures deterministic URLs (same photo ID always returns same image).

### Initial Album Order

Albums are sorted by creation date (newest first):

```
1. album-001 (2026-01)
2. album-002 (2025-12)
3. album-003 (2025-11)
4. album-004 (2025-10)
5. album-005 (2025-09)
```

---

## Development Workflow

### 1. Start Development Server

```bash
npm run dev
```

- Opens at: `http://localhost:5174`
- Hot Module Replacement (HMR) enabled
- IndexedDB data persists across page reloads
- Use browser DevTools > Application > IndexedDB to inspect stored data

### 2. Initialize Sample Data

On first app launch, the following occurs automatically:

1. **Check IndexedDB keys**: App checks if `"albums"` and `"photos"` keys exist
2. **Seed data if missing**: If keys don't exist, `seedDatabase()` function is called
3. **Generate entities**:
   - 5 `Album` objects with YYYY-MM titles
   - 75 `Photo` objects with Unsplash URLs and tags
   - `albumOrder` array with album IDs
   - `userPreferences` object (dark mode: false)
4. **Store to IndexedDB**: All entities saved via `idb-keyval`

**Seed Function Location**: `src/lib/seedData.ts`

**Manual Reset** (to regenerate sample data):

- Open browser DevTools > Application > IndexedDB
- Delete `keyval-store` database
- Refresh page to trigger re-seed

### 3. Run Tests

**Run all tests once**:

```bash
npm run test
```

**Watch mode** (re-runs on file changes):

```bash
npm run test:watch
```

**Vitest UI** (interactive test browser):

```bash
npm run test:ui
```

Opens at: `http://localhost:51204/__vitest__/`

### 4. Build for Production

```bash
npm run build
```

- Output directory: `dist/`
- Preview production build: `npm run preview`

---

## IndexedDB Storage Keys

PhotoOrganizer uses **idb-keyval** for simple key-value storage:

| Key                 | Type              | Description                     |
| ------------------- | ----------------- | ------------------------------- |
| `"albums"`          | `Album[]`         | Array of all album objects      |
| `"photos"`          | `Photo[]`         | Array of all photo objects      |
| `"albumOrder"`      | `string[]`        | Custom sort order (album IDs)   |
| `"userPreferences"` | `UserPreferences` | Dark mode and last viewed album |

**Storage Limit**: ~50MB quota (enforced by browser)

**Error Handling**:

- If storage quota exceeded, show user error message (FR-003)
- If IndexedDB unavailable, show error message (FR-012)

---

## Project Structure

After implementation, the project will have the following structure:

```
src/
├── components/
│   ├── ui/                   # shadcn/ui components
│   │   ├── button.tsx
│   │   └── card.tsx
│   ├── AlbumCard.tsx         # Album display card
│   ├── PhotoGrid.tsx         # Photo grid with lazy loading
│   ├── FilterBar.tsx         # Date/tag filter controls
│   └── DarkModeToggle.tsx    # Theme toggle button
├── contexts/
│   ├── AlbumContext.tsx      # Album state management
│   ├── PhotoContext.tsx      # Photo state management
│   └── ThemeContext.tsx      # Dark mode state
├── hooks/
│   ├── useAlbums.ts          # Album operations hook
│   ├── usePhotos.ts          # Photo operations hook
│   ├── useLazyImage.ts       # Lazy loading hook
│   └── useIndexedDB.ts       # IndexedDB persistence hook
├── lib/
│   ├── utils.ts              # Utility functions (cn, etc.)
│   └── seedData.ts           # Sample data generation
├── types/
│   └── index.ts              # TypeScript interfaces
├── App.tsx                   # Root component
├── main.tsx                  # App entry point
└── index.css                 # Global styles

public/
└── images/                   # Placeholder for future local images

tests/
├── components/
│   ├── AlbumCard.test.tsx
│   ├── PhotoGrid.test.tsx
│   └── FilterBar.test.tsx
├── hooks/
│   ├── useAlbums.test.ts
│   ├── usePhotos.test.ts
│   └── useLazyImage.test.ts
└── lib/
    └── seedData.test.ts
```

---

## Development Checklist

Before implementing components, ensure:

- [ ] All dependencies installed (`npm install` completed)
- [ ] Dev server running (`npm run dev` at localhost:5174)
- [ ] TypeScript interfaces defined in `src/types/index.ts`
- [ ] Sample data generation script created at `src/lib/seedData.ts`
- [ ] IndexedDB helper hook created at `src/hooks/useIndexedDB.ts`
- [ ] Context providers implemented in `src/contexts/`
- [ ] Dark mode CSS variables working (test toggle in demo App.tsx)

---

## Testing Strategy

### Unit Tests

**Coverage Targets** (per constitution):

- All utility functions: 100%
- Custom hooks: 90%+
- Components: 80%+

**Test Structure**:

```typescript
// Example: src/hooks/useAlbums.test.ts
import { renderHook, act } from "@testing-library/react";
import { useAlbums } from "@/hooks/useAlbums";

describe("useAlbums", () => {
  it("should load albums from IndexedDB on mount", async () => {
    const { result } = renderHook(() => useAlbums());

    await act(async () => {
      await result.current.refreshAlbums();
    });

    expect(result.current.albums).toHaveLength(5);
  });

  it("should reorder albums and persist to IndexedDB", async () => {
    const { result } = renderHook(() => useAlbums());

    await act(async () => {
      await result.current.reorderAlbums(0, 2);
    });

    expect(result.current.albumOrder[0]).toBe("album-003");
  });
});
```

### Integration Tests

Test complete user flows:

1. Load homepage → See 5 albums in grid
2. Drag album from position 0 to position 2 → Order persists on reload
3. Click album → Navigate to detail view → See photos filtered to that album
4. Toggle dark mode → Theme persists on reload

---

## Performance Targets

From [plan.md](plan.md) Technical Context:

| Metric              | Target                                    |
| ------------------- | ----------------------------------------- |
| **Initial Load**    | < 2 seconds (homepage with 5 albums)      |
| **Drag Feedback**   | < 100ms (visual response on drag start)   |
| **Filter Response** | < 500ms (date/tag filter applied)         |
| **Lazy Load**       | < 3 seconds (photos load as user scrolls) |

**Monitoring**:

- Use React DevTools Profiler to identify slow renders
- Use Chrome DevTools Lighthouse for performance audits
- Aim for Lighthouse Performance score: 90+

---

## Accessibility Requirements

Per constitution **Principle IV: Accessibility (ARIA Standards)**:

- [ ] All interactive elements have ARIA labels (`aria-label`, `aria-describedby`)
- [ ] Keyboard navigation works for all features:
  - Arrow keys to navigate album cards
  - Enter to open album
  - Escape to close modals/filters
  - Tab to focus elements in logical order
- [ ] Focus indicators visible (Tailwind `focus:ring-2 focus:ring-primary`)
- [ ] Color contrast meets WCAG AA standards (4.5:1 for text)
- [ ] Screen reader testing with VoiceOver (macOS) or NVDA (Windows)

**Testing Tools**:

- [axe DevTools](https://www.deque.com/axe/devtools/) browser extension
- Chrome Lighthouse Accessibility audit

---

## Dark Mode Implementation

**CSS Variables** (already configured in `src/index.css`):

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  /* ...other light mode colors */
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ...other dark mode colors */
}
```

**Toggle Logic**:

1. User clicks dark mode button
2. `ThemeContext.toggleDarkMode()` called
3. Context updates `darkMode` state
4. Effect adds/removes `"dark"` class on `<html>` element
5. Preference saved to IndexedDB under `userPreferences.darkMode`
6. On app reload, preference is read and theme restored

**Implementation Location**: `src/contexts/ThemeContext.tsx`, `src/components/DarkModeToggle.tsx`

---

## Next Steps

1. **Implement TypeScript Interfaces** (`src/types/index.ts`) ✅ (defined in data-model.md)
2. **Create Seed Data Script** (`src/lib/seedData.ts`)
3. **Build Context Providers** (`src/contexts/` — AlbumContext, PhotoContext, ThemeContext)
4. **Create Custom Hooks** (`src/hooks/` — useAlbums, usePhotos, useLazyImage, useIndexedDB)
5. **Implement Components** (`src/components/` — AlbumCard, PhotoGrid, FilterBar, DarkModeToggle)
6. **Wire Up Drag-Drop** (react-beautiful-dnd in HomePage)
7. **Write Unit Tests** (`tests/` — match component structure)
8. **Manual Testing** (verify all 15 functional requirements from spec.md)
9. **Accessibility Audit** (axe DevTools, keyboard navigation)
10. **Performance Optimization** (lazy loading, React.memo where needed)

**Phase 2 Planning** (separate command):
Run `/speckit.tasks` to generate `tasks.md` with detailed implementation tasks.

---

## Troubleshooting

### IndexedDB Not Working

**Symptom**: Data doesn't persist across page reloads

**Solutions**:

- Check browser DevTools > Console for IndexedDB errors
- Ensure browser supports IndexedDB (check `window.indexedDB` exists)
- Try incognito mode (some privacy extensions block IndexedDB)
- Check storage quota: `navigator.storage.estimate()`

### Drag-Drop Not Responding

**Symptom**: Album cards don't move when dragged

**Solutions**:

- Verify `react-beautiful-dnd` installed: `npm list react-beautiful-dnd`
- Check console for errors related to `DragDropContext`
- Ensure `<Droppable droppableId>` is unique
- Verify `<Draggable draggableId>` is unique per item

### Images Not Loading

**Symptom**: Broken image icons in photo grid

**Solutions**:

- Check network tab for failed Unsplash requests
- Verify Unsplash Source API URLs: `https://source.unsplash.com/random/400x300?sig=photo-001`
- Try different photo ID in `sig` parameter
- Check CORS policy (Unsplash allows cross-origin requests)

### Dark Mode Not Persisting

**Symptom**: Theme resets to light mode on page reload

**Solutions**:

- Check IndexedDB for `userPreferences` key
- Verify `ThemeContext` reads from IndexedDB on mount
- Ensure `<html>` class toggled correctly: `document.documentElement.classList.toggle('dark')`

---

## Resources

- **React 18 Docs**: https://react.dev
- **Vite Docs**: https://vite.dev
- **Tailwind CSS Docs**: https://tailwindcss.com
- **shadcn/ui Docs**: https://ui.shadcn.com
- **react-beautiful-dnd Docs**: https://github.com/atlassian/react-beautiful-dnd
- **date-fns Docs**: https://date-fns.org
- **idb-keyval Docs**: https://github.com/jakearchibald/idb-keyval
- **Vitest Docs**: https://vitest.dev
- **Testing Library Docs**: https://testing-library.com/react

---

**Next Document**: Generate `tasks.md` with Phase 2 implementation tasks (run `/speckit.tasks`)
