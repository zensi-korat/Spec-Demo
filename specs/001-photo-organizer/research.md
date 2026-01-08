# Research: PhotoOrganizer Technical Decisions

**Feature**: PhotoOrganizer  
**Created**: 2026-01-07  
**Purpose**: Document research findings and technical decisions for implementation

## Research Tasks

### 1. Drag-and-Drop Library Selection

**Decision**: react-beautiful-dnd (per user specification)

**Rationale**:

- Mature library with strong React integration and TypeScript support
- Handles touch gestures for mobile devices (FR-003 requirement)
- Provides smooth animations and visual feedback out of the box
- Widely adopted with extensive documentation and examples
- Supports accessible drag-and-drop with keyboard navigation (FR-012 requirement)

**Alternatives Considered**:

- **dnd-kit**: More modern, better TypeScript support, but react-beautiful-dnd is user-specified and meets all requirements
- **react-dnd**: Lower-level API, more complex setup for simple list reordering
- Native HTML5 Drag-and-Drop API: Inconsistent browser behavior, requires significant custom code for mobile

**Implementation Notes**:

- Wrap album grid in `<DragDropContext>` with `onDragEnd` handler
- Use `<Droppable>` for album container and `<Draggable>` for individual album cards
- Persist new order to IndexedDB on drag completion
- Mobile touch support built-in; test on actual devices to verify gesture handling

---

### 2. IndexedDB Abstraction Library

**Decision**: idb-keyval (per user specification)

**Rationale**:

- Minimal API surface area (~0.5KB gzipped) aligns with constitution's "avoid heavy libraries" principle
- Promise-based API simplifies async operations in React components
- Simple key-value store sufficient for album order, user preferences, and photo data
- TypeScript definitions included
- No learning curve compared to raw IndexedDB API

**Alternatives Considered**:

- **Raw IndexedDB API**: More verbose, requires transaction management, error handling boilerplate
- **Dexie.js**: More features (queries, indexes) but heavier (~15KB); overkill for simple key-value storage
- **localForage**: Larger bundle size, automatic fallback to localStorage not needed (IndexedDB is universally supported in target browsers)

**Implementation Notes**:

- Store separate keys for: `albums` (array), `photos` (array), `albumOrder` (array of IDs), `userPreferences` (object)
- Initialize sample data on first app load if keys don't exist
- Wrap idb-keyval calls in custom hooks: `useAlbums()`, `usePhotos()`, `usePreferences()`
- Handle IndexedDB quota errors gracefully per edge case requirement

---

### 3. Date Formatting Library

**Decision**: date-fns (per user specification)

**Rationale**:

- Tree-shakeable; only import functions actually used (e.g., `format`, `isWithinInterval`, `parse`)
- Immutable date objects (safer than Moment.js mutability)
- Smaller bundle impact than Moment.js (~5KB vs ~70KB)
- Strong TypeScript support with type-safe date operations
- Functions match use cases: `format(date, 'yyyy-MM')` for album titles, `isWithinInterval` for date range filtering

**Alternatives Considered**:

- **Vanilla JavaScript Date/Intl.DateTimeFormat**: Requires more code for formatting and parsing; date-fns provides cleaner API
- **Day.js**: Similar size/features, but date-fns has better TypeScript ergonomics and wider adoption
- **Moment.js**: Large bundle size, mutable API (deprecated for new projects)

**Implementation Notes**:

- Use `format(date, 'yyyy-MM')` for album title generation (FR-001)
- Use `parseISO` for parsing date strings from IndexedDB
- Use `isWithinInterval` for date range filtering (FR-008)
- Store dates as ISO 8601 strings in IndexedDB for consistency

---

### 4. Lazy Loading Image Strategy

**Decision**: Intersection Observer API with React ref + useEffect

**Rationale**:

- Native browser API, no external dependencies (aligns with constitution principle V)
- Efficient viewport detection with configurable thresholds and root margins
- Works across all modern browsers
- Custom implementation gives full control over loading behavior and error handling

**Alternatives Considered**:

- **react-lazyload**: External dependency (~2KB), but Intersection Observer is natively available
- **react-intersection-observer**: Wrapper library, but custom hook implementation is straightforward
- **Loading="lazy" attribute**: Simpler but less control over loading thresholds and error states

**Implementation Notes**:

- Create `useLazyImage` custom hook: `const { ref, loaded, error } = useLazyImage(imageUrl)`
- Hook sets up Intersection Observer on mount, loads image when element enters viewport
- Return `loaded` and `error` states for conditional rendering (placeholder → image → error icon)
- Apply to `<PhotoThumbnail>` component to satisfy FR-014 requirement
- Configure `rootMargin: '100px'` to preload images slightly before they enter viewport

---

### 5. React Context for State Management

**Decision**: React Context API (per user specification, no state library)

**Rationale**:

- No external dependencies; built into React 18
- Sufficient for app's state management needs (albums, photos, preferences, dark mode)
- Constitution specifies "No state lib—use React Context"
- Avoids bundle bloat from Redux/Zustand/MobX

**Alternatives Considered**:

- **Redux Toolkit**: Overkill for simple state tree; adds complexity and bundle size
- **Zustand**: Lightweight alternative but user explicitly requested no state library
- **Component prop drilling**: Would require passing props through multiple levels (HomePage → AlbumGrid → AlbumCard)

**Implementation Notes**:

- Create separate contexts for logical boundaries:
  - `<AlbumContext>`: Provides albums array, album order, reorder function
  - `<PhotoContext>`: Provides photos array, add/remove functions
  - `<ThemeContext>`: Provides dark mode state and toggle function
- Use custom hooks to consume contexts: `useAlbums()`, `usePhotos()`, `useTheme()`
- Initialize contexts with IndexedDB data on app mount
- Context providers wrap `<App>` in main.tsx

---

### 6. Sample Photo Data Source

**Decision**: Unsplash public API URLs + local fallback data

**Rationale**:

- Free, high-quality images without signup for demo purposes
- Predictable URL format: `https://source.unsplash.com/random/400x300?sig={seed}` generates consistent images
- No CORS issues (public API designed for hotlinking)
- Meets assumption: "Sample photo URLs will be publicly accessible (e.g., via Unsplash, Pexels...)"

**Alternatives Considered**:

- **Pexels API**: Requires API key, adds complexity
- **Placeholder.com**: Generic placeholders less visually appealing
- **Base64 data URIs in IndexedDB**: Increases storage usage, slower initial load

**Implementation Notes**:

- Generate 75 unique photo objects (5 albums × 15 photos average) in `seedData.ts`
- Use Unsplash with incrementing `sig` parameter for consistent images: `?sig=photo-001`, `?sig=photo-002`, etc.
- Add tags to each photo for filtering: `["vacation", "family", "nature", "architecture", "food"]`
- Store photo objects in IndexedDB on first load
- Handle image load failures with placeholder icon per edge case requirement

---

### 7. Responsive Breakpoints Strategy

**Decision**: Tailwind CSS default breakpoints (sm: 640px, md: 768px, lg: 1024px, xl: 1280px)

**Rationale**:

- Constitution Principle III: "Mobile-first using Tailwind's responsive breakpoints"
- Default breakpoints align with FR-011 requirements (320px+, 768px+, 1024px+)
- No custom configuration needed; developers already familiar with Tailwind defaults

**Implementation Notes**:

- Album grid: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4`
- Photo grid: `grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5`
- Spacing adjusts: `gap-4 md:gap-6 lg:gap-8`
- Test mobile experience first, then verify tablet/desktop enhancements

---

### 8. Keyboard Navigation Pattern

**Decision**: Standard web keyboard shortcuts with visible focus indicators

**Rationale**:

- FR-012 requires keyboard navigation for album/photo selection
- FR-013 requires ARIA labels and roles
- Constitution Principle IV: Accessibility mandatory

**Implementation Notes**:

- Album grid: Arrow keys move focus, Enter opens album, Escape returns to homepage
- Photo selection: Space toggles selection, Ctrl+A/Cmd+A selects all
- Drag-and-drop: react-beautiful-dnd provides keyboard alternative (Space to lift, arrows to move, Space to drop)
- Add `tabIndex={0}` to interactive elements
- Style focus states: `focus:ring-2 focus:ring-primary focus:outline-none`
- ARIA labels: `aria-label="Album for January 2026 with 15 photos"`

---

## Summary of Technology Stack

| Category         | Technology                     | Rationale                                                                |
| ---------------- | ------------------------------ | ------------------------------------------------------------------------ |
| Framework        | React 18                       | Constitution-mandated, hooks-based architecture                          |
| Build Tool       | Vite                           | Fast dev server, optimized production builds                             |
| Language         | TypeScript (strict mode)       | Constitution-mandated type safety                                        |
| Styling          | Tailwind CSS                   | Constitution-mandated utility-first styling                              |
| UI Components    | shadcn/ui                      | Constitution principle II (reusability), pre-built accessible components |
| Icons            | lucide-react                   | Consistent icon set, tree-shakeable, React-optimized                     |
| Drag-and-Drop    | react-beautiful-dnd            | User-specified, mature library with mobile support                       |
| Date Handling    | date-fns                       | User-specified, tree-shakeable, TypeScript-friendly                      |
| Data Storage     | IndexedDB via idb-keyval       | User-specified, minimal API, constitution-compliant (no backend)         |
| State Management | React Context API              | User-specified, no external state library                                |
| Testing          | Vitest + React Testing Library | User-specified, fast unit tests for components and hooks                 |
| Lazy Loading     | Intersection Observer (native) | Constitution principle V (avoid heavy libs), native API                  |

---

## Open Questions / Clarifications

None—all technical decisions align with user specifications and constitution principles.
