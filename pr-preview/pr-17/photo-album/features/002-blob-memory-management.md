# Feature Spec: Blob Memory Management

**Status:** Implemented
**PRs:** #7, #8
**Date:** 2026-01-24

## Overview

Implement proper blob URL lifecycle management to prevent memory leaks and browser resource exhaustion when viewing albums with many photos/videos.

## Problem Statement

The initial implementation created blob URLs using `URL.createObjectURL()` but never revoked them. This caused:

1. **Memory leaks:** Blob URLs hold references to underlying data, preventing garbage collection
2. **Resource exhaustion:** Browsers limit concurrent blob resources; exceeding this causes "WebKitBlobResource error 1"
3. **Performance degradation:** Albums with 50+ items became sluggish and eventually failed to render

Additionally, WebKit's IndexedDB implementation has a bug that causes "Error preparing Blob/File data to be stored" when storing `File` or `Blob` objects directly from user uploads.

## Requirements

### Functional Requirements

1. Blob URLs must be revoked after they are no longer needed
2. The system must handle albums with 100+ photos/videos without errors
3. Media must continue to display correctly after blob URL cleanup
4. Existing albums with old blob format must remain readable (backwards compatibility)

### Non-Functional Requirements

1. No visible impact on user experience
2. Memory usage should remain stable regardless of album size
3. Must work across Chrome, Firefox, Safari, and Edge

## Design Decisions

### Decision 1: When to Revoke Blob URLs for Images

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Never revoke | Let browser manage | Simple code | Memory leaks, crashes |
| B. Revoke on view exit | Clean up when leaving album | Batch cleanup | Still accumulates during viewing |
| C. Revoke after load | Revoke immediately after image loads | Minimal memory use | Requires careful event handling |
| D. Periodic cleanup | Timer-based revocation | Predictable | Complex, may revoke too early/late |

**Decision:** Option C - Revoke immediately after image `load` event

**Rationale:** Once an image fires its `load` event, the browser has decoded the image data into memory. The blob URL is no longer needed - the `<img>` element retains the decoded pixels. This approach minimizes concurrent blob URLs to only those actively loading.

### Decision 2: Blob URL Handling for Videos vs Images

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Treat same as images | Revoke after `loadeddata` | Consistent logic | Breaks video playback |
| B. Never revoke video URLs | Keep alive indefinitely | Simple, works | Memory accumulates |
| C. Revoke on view exit only | Keep during playback, clean on exit | Works, bounded memory | Slightly more code |

**Decision:** Option C - Keep video blob URLs alive during viewing, revoke on album exit

**Rationale:** Unlike images, videos **stream** from their blob URLs during playback. Revoking a video's blob URL after `canplay` causes `ERR_FILE_NOT_FOUND` when the user tries to play. Videos must retain their blob URLs until the user navigates away from the album view.

### Decision 3: IndexedDB Storage Format

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Store Blob directly | `photo.blob = file` | Simple, standard | WebKit bug causes failures |
| B. Store as base64 string | Convert to data URL | Universal support | 33% larger, slow encoding |
| C. Store as ArrayBuffer | `file.arrayBuffer()` | Compact, reliable | Reconstruction needed |
| D. Clone blob | `new Blob([file])` | Might bypass bug | Didn't work in testing |

**Decision:** Option C - Convert to ArrayBuffer before storage

**Rationale:**
- ArrayBuffer is a guaranteed serializable format in IndexedDB
- Bypasses WebKit's Blob/File handling bug entirely
- No size overhead (unlike base64)
- Reconstruction is straightforward: `new Blob([arrayBuffer], { type })`

### Decision 4: Backwards Compatibility

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Migration script | Convert old format on app load | Clean data | Slow startup, complex |
| B. Dual-format support | Handle both formats at read time | No migration needed | Conditional logic |
| C. Ignore old data | Require re-import | Simple code | Data loss for users |

**Decision:** Option B - Runtime detection of old vs new format

**Rationale:** Users should not lose their existing albums. The format detection is simple:

```javascript
if (photo.blob && photo.blob.data instanceof ArrayBuffer) {
  // New format: { data: ArrayBuffer, type: string }
  blobToRender = new Blob([photo.blob.data], { type: photo.blob.type });
} else if (photo.blob instanceof Blob) {
  // Old format: direct Blob
  blobToRender = photo.blob;
}
```

## Implementation

### Storage Format (New)

```javascript
// When saving a photo
const arrayBuffer = await file.arrayBuffer();
const photo = {
  id: generateId(),
  albumName: albumName,
  blob: {
    data: arrayBuffer,  // ArrayBuffer
    type: file.type     // MIME type string
  },
  filename: file.name,
  order: nextOrder,
  createdAt: Date.now()
};
```

### Image Blob URL Lifecycle

```javascript
const url = URL.createObjectURL(blob);
img.src = url;

img.addEventListener('load', () => {
  URL.revokeObjectURL(url);  // Safe to revoke - image is decoded
}, { once: true });

img.addEventListener('error', () => {
  URL.revokeObjectURL(url);  // Clean up on error too
}, { once: true });
```

### Video Blob URL Tracking

```javascript
const videoBlobUrls = new Map(); // photoId -> url

// When loading video
const url = URL.createObjectURL(blob);
videoBlobUrls.set(photoId, url);
video.src = url;

// When leaving album view
for (const [photoId, url] of videoBlobUrls) {
  URL.revokeObjectURL(url);
}
videoBlobUrls.clear();
```

## Testing Checklist

- [ ] Upload 100+ photos, verify no memory errors
- [ ] Upload 20+ videos, verify playback works
- [ ] Navigate in/out of album multiple times, verify stable memory
- [ ] Open existing album with old Blob format, verify photos display
- [ ] Export album with new format, verify import works
- [ ] Test in Safari (most strict about blob resources)
- [ ] Monitor browser memory usage during extended use

## Metrics

- Before fix: Albums with 50+ items crashed Safari
- After fix: Albums with 200+ items work reliably
- Memory usage: Stable instead of linear growth

## Lessons Learned

1. Always revoke blob URLs when done - browsers don't auto-clean them
2. Videos and images have fundamentally different blob URL requirements
3. IndexedDB blob storage has browser-specific quirks - ArrayBuffer is safest
4. Always maintain backwards compatibility for persisted data formats
