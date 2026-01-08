# Data Model: PhotoOrganizer

**Feature**: PhotoOrganizer  
**Created**: 2026-01-07  
**Purpose**: Define data structures, TypeScript interfaces, and storage schema

## Entity Definitions

### Album

Represents a collection of photos grouped by time period (typically one month).

**TypeScript Interface**:

```typescript
interface Album {
  id: string; // Unique identifier (e.g., "album-001")
  title: string; // Display name (e.g., "2026-01", "January 2026")
  createdDate: string; // ISO 8601 string (e.g., "2026-01-07T12:00:00.000Z")
  photoCount: number; // Computed: number of photos in this album
  sortOrder: number; // Custom sort position (0-indexed, user-defined)
}
```

**Attributes**:

- `id`: String-based UUID or incremental ID for stable references
- `title`: Defaults to YYYY-MM format; could be customized in future versions
- `createdDate`: ISO 8601 format for consistent date handling with date-fns
- `photoCount`: Derived from Photo entities' `albumMemberships` arrays; denormalized for quick display
- `sortOrder`: Integer position in custom order (0 = first, 1 = second, etc.); updated on drag-and-drop

**Relationships**:

- **Contains many Photos**: Implicit via Photo.albumMemberships array (many-to-many)

**Validation Rules**:

- `id` must be unique across all albums
- `title` must not be empty
- `photoCount` must be non-negative integer
- `sortOrder` must be non-negative integer

**Storage**:

- Stored in IndexedDB under key: `"albums"` (array of Album objects)
- Custom order stored separately under key: `"albumOrder"` (array of album IDs)

---

### Photo

Represents an individual image with metadata and album memberships.

**TypeScript Interface**:

```typescript
interface Photo {
  id: string; // Unique identifier (e.g., "photo-001")
  thumbnailUrl: string; // URL for thumbnail display (e.g., Unsplash 400x300)
  fullSizeUrl: string; // URL for full-resolution image (e.g., Unsplash 1920x1080)
  createdDate: string; // ISO 8601 string (photo creation/capture date)
  tags: string[]; // Array of tag strings (e.g., ["vacation", "family"])
  albumMemberships: string[]; // Array of album IDs this photo belongs to
}
```

**Attributes**:

- `id`: String-based UUID or incremental ID for stable references
- `thumbnailUrl`: External URL (Unsplash) or data URI; used for grid display
- `fullSizeUrl`: External URL or data URI; used for lightbox/full view (future enhancement)
- `createdDate`: ISO 8601 format; used for date range filtering (FR-008)
- `tags`: Array of lowercase strings; used for tag filtering (FR-008)
- `albumMemberships`: Array of album IDs; enables many-to-many relationship (photo can be in multiple albums)

**Relationships**:

- **Belongs to many Albums**: Explicit via `albumMemberships` array

**Validation Rules**:

- `id` must be unique across all photos
- `thumbnailUrl` and `fullSizeUrl` must be valid URLs or data URIs
- `tags` array can be empty but must not contain duplicates
- `albumMemberships` array references must correspond to existing album IDs

**Storage**:

- Stored in IndexedDB under key: `"photos"` (array of Photo objects)

---

### UserPreferences

Stores user-specific settings persisted across sessions.

**TypeScript Interface**:

```typescript
interface UserPreferences {
  darkMode: boolean; // Dark mode enabled (true) or light mode (false)
  lastViewedAlbumId: string | null; // ID of most recently viewed album, or null
}
```

**Attributes**:

- `darkMode`: Boolean toggle state; used to apply dark mode theme on app load
- `lastViewedAlbumId`: Optional string; could be used to restore last view on app reopen (out of scope for MVP)

**Relationships**:

- None (standalone settings entity)

**Validation Rules**:

- `darkMode` must be boolean
- `lastViewedAlbumId` must be null or valid album ID

**Storage**:

- Stored in IndexedDB under key: `"userPreferences"` (single object, not array)

---

## Derived/Computed Properties

### Album.photoCount

**Computation**: Count of Photo entities where `albumMemberships` includes this album's ID

**Update Triggers**:

- When photo is added to album (increment)
- When photo is removed from album (decrement)
- On initial data load (recalculate from Photo array)

**Implementation**:

```typescript
function calculatePhotoCount(albumId: string, photos: Photo[]): number {
  return photos.filter((photo) => photo.albumMemberships.includes(albumId))
    .length;
}
```

---

## IndexedDB Storage Schema

### Keys and Values

| Key                 | Type              | Description                             |
| ------------------- | ----------------- | --------------------------------------- |
| `"albums"`          | `Album[]`         | Array of all album objects              |
| `"photos"`          | `Photo[]`         | Array of all photo objects              |
| `"albumOrder"`      | `string[]`        | Array of album IDs in custom sort order |
| `"userPreferences"` | `UserPreferences` | Single preferences object               |

### Initial Seed Data Structure

**Sample Album Titles** (5 albums):

1. "2026-01" (January 2026) - 15 photos
2. "2025-12" (December 2025) - 12 photos
3. "2025-11" (November 2025) - 18 photos
4. "2025-10" (October 2025) - 10 photos
5. "2025-09" (September 2025) - 20 photos

**Sample Photo Tags Distribution**:

- `vacation`: 20 photos
- `family`: 18 photos
- `nature`: 15 photos
- `architecture`: 12 photos
- `food`: 10 photos

**Photo URL Pattern**:

```
thumbnailUrl: `https://source.unsplash.com/random/400x300?sig=photo-${id}`
fullSizeUrl: `https://source.unsplash.com/random/1920x1080?sig=photo-${id}`
```

---

## State Management Strategy

### React Context Structure

**AlbumContext**:

```typescript
interface AlbumContextType {
  albums: Album[];
  albumOrder: string[];
  reorderAlbums: (startIndex: number, endIndex: number) => void;
  refreshAlbums: () => Promise<void>;
}
```

**PhotoContext**:

```typescript
interface PhotoContextType {
  photos: Photo[];
  addPhotoToAlbum: (photoId: string, albumId: string) => Promise<void>;
  removePhotoFromAlbum: (photoId: string, albumId: string) => Promise<void>;
  getAlbumPhotos: (albumId: string) => Photo[];
  refreshPhotos: () => Promise<void>;
}
```

**ThemeContext**:

```typescript
interface ThemeContextType {
  darkMode: boolean;
  toggleDarkMode: () => void;
}
```

### Data Flow

1. **App Initialization** (`main.tsx`):

   - Context providers wrap `<App>` component
   - Each provider calls `useIndexedDB` hook to load data from idb-keyval
   - If keys don't exist, seed data is generated and stored

2. **Album Reordering** (HomePage):

   - User drags album card (react-beautiful-dnd)
   - `onDragEnd` handler calls `AlbumContext.reorderAlbums(startIndex, endIndex)`
   - Context updates `albumOrder` array and persists to IndexedDB

3. **Photo Management** (AlbumDetailPage):

   - User selects photos, clicks "Remove from Album"
   - Component calls `PhotoContext.removePhotoFromAlbum(photoId, albumId)`
   - Context updates Photo.albumMemberships array, recalculates Album.photoCount, persists to IndexedDB

4. **Dark Mode Toggle** (DarkModeToggle component):
   - User clicks toggle button
   - Component calls `ThemeContext.toggleDarkMode()`
   - Context updates local state and persists to IndexedDB under `userPreferences.darkMode`
   - Root `<html>` class toggles `"dark"` class for Tailwind dark mode

---

## Filtering Logic

### Date Range Filter

**Implementation**:

```typescript
import { isWithinInterval, parseISO } from "date-fns";

function filterPhotosByDateRange(
  photos: Photo[],
  startDate: string, // ISO 8601 string
  endDate: string // ISO 8601 string
): Photo[] {
  const start = parseISO(startDate);
  const end = parseISO(endDate);

  return photos.filter((photo) => {
    const photoDate = parseISO(photo.createdDate);
    return isWithinInterval(photoDate, { start, end });
  });
}
```

### Tag Filter

**Implementation**:

```typescript
function filterPhotosByTags(
  photos: Photo[],
  selectedTags: string[] // Array of tag strings
): Photo[] {
  if (selectedTags.length === 0) return photos;

  return photos.filter((photo) =>
    selectedTags.some((tag) => photo.tags.includes(tag))
  );
}
```

### Combined Filter

**Implementation**:

```typescript
function filterPhotos(
  photos: Photo[],
  dateRange: { start: string; end: string } | null,
  selectedTags: string[]
): Photo[] {
  let filtered = photos;

  if (dateRange) {
    filtered = filterPhotosByDateRange(
      filtered,
      dateRange.start,
      dateRange.end
    );
  }

  if (selectedTags.length > 0) {
    filtered = filterPhotosByTags(filtered, selectedTags);
  }

  return filtered;
}
```

---

## Type Exports

**`src/types/index.ts`**:

```typescript
export interface Album {
  id: string;
  title: string;
  createdDate: string;
  photoCount: number;
  sortOrder: number;
}

export interface Photo {
  id: string;
  thumbnailUrl: string;
  fullSizeUrl: string;
  createdDate: string;
  tags: string[];
  albumMemberships: string[];
}

export interface UserPreferences {
  darkMode: boolean;
  lastViewedAlbumId: string | null;
}

// Context types
export interface AlbumContextType {
  albums: Album[];
  albumOrder: string[];
  reorderAlbums: (startIndex: number, endIndex: number) => void;
  refreshAlbums: () => Promise<void>;
}

export interface PhotoContextType {
  photos: Photo[];
  addPhotoToAlbum: (photoId: string, albumId: string) => Promise<void>;
  removePhotoFromAlbum: (photoId: string, albumId: string) => Promise<void>;
  getAlbumPhotos: (albumId: string) => Photo[];
  refreshPhotos: () => Promise<void>;
}

export interface ThemeContextType {
  darkMode: boolean;
  toggleDarkMode: () => void;
}
```

---

## Sample Data Generation

**Total Entities**:

- 5 Albums
- 75 Photos (distributed across albums)

**Photo Distribution**:

- Album "2026-01": photo-001 through photo-015 (15 photos)
- Album "2025-12": photo-016 through photo-027 (12 photos)
- Album "2025-11": photo-028 through photo-045 (18 photos)
- Album "2025-10": photo-046 through photo-055 (10 photos)
- Album "2025-09": photo-056 through photo-075 (20 photos)

**Generation Script Location**: `src/lib/seedData.ts`

**Seed Data Initialization**:

- Called on app mount if IndexedDB keys `"albums"` and `"photos"` don't exist
- Generates deterministic data (same URLs, tags, dates on each initialization)
- Stores all entities to IndexedDB via idb-keyval in single transaction
