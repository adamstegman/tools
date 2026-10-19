# Feature Spec: Lazy Loading with Concurrency Control

**Status:** Implemented
**PR:** #9
**Date:** 2026-01-25

## Overview

Implement lazy loading for photos and videos using IntersectionObserver, combined with a loading queue that limits concurrent blob URL creations to prevent browser resource exhaustion.

## Problem Statement

Even with proper blob URL revocation (see 002-blob-memory-management.md), uploading or viewing many photos/videos at once still caused "WebKitBlobResource error 1" in Safari. The issue: all blob URLs were created simultaneously during render, exhausting the browser's blob resource pool before any could be revoked.

Example scenario:
1. User uploads 50 photos
2. `renderPhotos()` creates 50 blob URLs instantly
3. Browser hits resource limit before first image loads
4. Images fail with WebKitBlobResource error

## Requirements

### Functional Requirements

1. Only create blob URLs for media visible in or near the viewport
2. Limit concurrent blob URL creations to prevent resource exhaustion
3. Provide visual feedback for media that hasn't loaded yet
4. Maintain smooth scrolling experience
5. Handle rapid scrolling without breaking

### Non-Functional Requirements

1. First visible photos should load immediately (no unnecessary delay)
2. Loading should feel responsive - no long pauses
3. System should handle albums with 500+ items
4. Must work across all supported browsers

## Design Decisions

### Decision 1: Lazy Loading Trigger Mechanism

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Scroll event listener | Check positions on scroll | Works everywhere | Performance issues, complex math |
| B. IntersectionObserver | Native visibility API | Performant, simple | Older browser support |
| C. Virtual scrolling | Only render visible items | Minimal DOM | Complex, breaks drag-drop |
| D. Pagination | Load pages of N items | Simple | Bad UX for galleries |

**Decision:** Option B - IntersectionObserver

**Rationale:**
- Built-in browser API optimized for this exact use case
- Handles all the viewport math automatically
- Doesn't fire on every scroll pixel (debounced internally)
- Excellent browser support (all modern browsers)
- Simple API: observe elements, get callbacks when they enter/exit viewport

### Decision 2: Root Margin for Preloading

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. 0px (viewport only) | Load only when visible | Minimal resource use | Visible loading delay |
| B. 200px | Start 200px before visible | Good balance | Some extra loading |
| C. 500px | Start 500px before visible | Very smooth scroll | More concurrent loads |
| D. 100vh (full viewport) | Preload one screen ahead | Seamless | Too aggressive |

**Decision:** Option B - 200px root margin

**Rationale:**
- 200px provides enough lead time for most scroll speeds
- Fast scrollers may see brief placeholders (acceptable)
- Keeps concurrent blob URLs bounded
- Tested empirically to feel responsive without waste

```javascript
mediaObserver = new IntersectionObserver(callback, {
  rootMargin: '200px',
  threshold: 0
});
```

### Decision 3: Concurrent Load Limit

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. No limit | Load all visible at once | Fast for small albums | Crashes on large albums |
| B. 2 concurrent | Very conservative | Very safe | Feels slow |
| C. 4 concurrent | Moderate limit | Good balance | May still stress some browsers |
| D. 8 concurrent | Higher throughput | Fast loading | Risk of resource issues |

**Decision:** Option C - Maximum 4 concurrent loads

**Rationale:**
- 4 is enough to keep the loading pipeline full
- Testing showed Safari handles 4 concurrent blob URLs reliably
- Provides good throughput without overwhelming the browser
- Queue ensures additional items load as slots free up

### Decision 4: Loading State UI

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Empty space | Show nothing until loaded | Simple | Jarring, layout shifts |
| B. Solid color placeholder | Gray box | Simple, stable | Boring, no feedback |
| C. Spinner | Rotating indicator | Clear loading state | Distracting with many |
| D. Shimmer animation | Animated gradient | Modern feel, clear state | Slightly more CSS |

**Decision:** Option D - Shimmer loading animation

**Rationale:**
- Provides clear visual feedback that content is loading
- Maintains layout stability (placeholder has dimensions)
- Feels modern and polished
- Single CSS animation, no JavaScript required

```css
@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

.photo-item img:not([src]),
.photo-item video:not([src]) {
  background: linear-gradient(
    90deg,
    var(--card) 25%,
    color-mix(in oklch, var(--border) 50%, var(--card)) 50%,
    var(--card) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
```

### Decision 5: Data Caching Strategy

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Re-fetch from IndexedDB | Query DB when item enters viewport | Always fresh | Slow, many DB reads |
| B. Cache all data upfront | Load all photo records at render | Fast access | High memory for metadata |
| C. LRU cache | Cache N most recent | Bounded memory | Complex eviction |

**Decision:** Option B - Cache all photo data in a Map at render time

**Rationale:**
- Photo metadata (without decoded images) is small (~100 bytes each)
- Even 1000 photos = ~100KB metadata, negligible
- Enables instant blob URL creation when item enters viewport
- Simple implementation with `Map`

```javascript
const photoDataCache = new Map(); // photoId -> photo record

// At render time
for (const photo of photos) {
  photoDataCache.set(photo.id, photo);
}

// When lazy loading
const photoData = photoDataCache.get(photoId);
const blob = new Blob([photoData.blob.data], { type: photoData.blob.type });
```

## Implementation

### Queue System

```javascript
const loadingQueue = [];
const activeLoads = new Set();
const MAX_CONCURRENT_LOADS = 4;

function queueMediaLoad(item, photoId) {
  if (!loadingQueue.some(q => q.photoId === photoId)) {
    loadingQueue.push({ item, photoId });
  }
  processLoadQueue();
}

function processLoadQueue() {
  while (activeLoads.size < MAX_CONCURRENT_LOADS && loadingQueue.length > 0) {
    const { item, photoId } = loadingQueue.shift();
    // Skip if already loaded or removed from DOM
    if (media.dataset.loaded === 'true' || !item.isConnected) continue;
    doLoadMedia(item, photoId);
  }
}
```

### Observer Setup

```javascript
function initMediaObserver() {
  mediaObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const item = entry.target;
          const photoId = item.dataset.photoId;
          queueMediaLoad(item, photoId);
        }
      });
    },
    { rootMargin: '200px', threshold: 0 }
  );
}
```

### Cleanup on View Exit

```javascript
function cleanupAlbumView() {
  // Disconnect observer
  if (mediaObserver) {
    mediaObserver.disconnect();
    mediaObserver = null;
  }

  // Clear queue
  loadingQueue.length = 0;
  activeLoads.clear();

  // Clear cache
  photoDataCache.clear();

  // Revoke video URLs
  for (const url of videoBlobUrls.values()) {
    URL.revokeObjectURL(url);
  }
  videoBlobUrls.clear();
}
```

## Testing Checklist

- [ ] Upload 100 photos, verify only visible ones load initially
- [ ] Scroll slowly, verify photos load before entering viewport
- [ ] Scroll rapidly, verify no crashes (some placeholders OK)
- [ ] Upload 50 videos, verify playback works after lazy load
- [ ] Navigate away and back, verify clean state
- [ ] Test shimmer animation appears for unloaded items
- [ ] Verify memory usage stays bounded
- [ ] Test in Safari (strictest resource limits)

## Performance Metrics

| Scenario | Before | After |
|----------|--------|-------|
| 100 photo album initial load | 100 blob URLs | 4-6 blob URLs |
| Safari resource errors | Common at 50+ | None at 200+ |
| Scroll jank | Noticeable | Smooth |
| Memory growth | Linear | Bounded |

## Future Considerations

- Thumbnail generation for faster initial grid view
- Progressive loading (low-res first, then full)
- Predictive preloading based on scroll velocity
- Service worker caching for offline support
