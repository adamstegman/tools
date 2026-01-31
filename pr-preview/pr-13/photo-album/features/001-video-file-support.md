# Feature Spec: Video File Support

**Status:** Implemented
**PR:** #6
**Date:** 2026-01-24

## Overview

Extend the Photo Album tool to accept and display video files alongside images, allowing users to create mixed-media albums.

## Problem Statement

The initial Photo Album implementation only supported image files. Users often want to store related videos alongside photos (e.g., event recordings, short clips). Without video support, users must manage videos separately, fragmenting their media organization.

## Requirements

### Functional Requirements

1. Accept video file uploads through the same upload zone as images
2. Display videos inline with photos in the album gallery
3. Provide playback controls for videos
4. Support common video formats (MP4, WebM, MOV, etc.)
5. Videos should be reorderable like photos
6. Videos should be exportable/importable with albums

### Non-Functional Requirements

1. Videos should load and play smoothly
2. Large videos should not block the UI during upload
3. Video storage should use the same IndexedDB infrastructure as photos

## Design Decisions

### Decision 1: File Input Accept Attribute

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. `accept="image/*"` only | Keep images only | Simple, smaller storage | Doesn't solve the problem |
| B. `accept="image/*,video/*"` | Accept both types | Solves problem, familiar UX | Larger storage requirements |
| C. Separate upload zones | Different zones for images/videos | Clear separation | Confusing UX, more complexity |

**Decision:** Option B - Single upload zone accepting both `image/*` and `video/*`

**Rationale:** Users expect a unified upload experience. The mental model of "add media to album" is simpler than distinguishing between image and video uploads. Storage concerns are mitigated by IndexedDB's generous limits.

### Decision 2: Video Display Element

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. HTML5 `<video>` with controls | Native video element | Built-in controls, accessible | Basic UI |
| B. Custom video player | JavaScript-based player | Custom UI, more features | Complexity, maintenance |
| C. Thumbnail with modal | Show poster, play in overlay | Clean gallery look | Extra clicks to watch |

**Decision:** Option A - Native HTML5 `<video>` element with `controls` attribute

**Rationale:**
- Follows the project's "no external dependencies" principle
- Native controls are accessible and familiar to users
- Works consistently across browsers
- Minimal code addition (~20 lines)

### Decision 3: Video Detection Logic

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Check MIME type | Use `file.type.startsWith('video/')` | Accurate, standard | Relies on correct MIME |
| B. Check file extension | Parse filename for video extensions | Simple | Can be wrong, incomplete list |
| C. Check both | MIME type with extension fallback | Most robust | More code |

**Decision:** Option A - MIME type checking

**Rationale:** The File API provides reliable MIME type information from the browser. Since files come from the user's system (not uploaded from unknown sources), MIME types are trustworthy.

## Implementation

### File Type Detection

```javascript
const isVideo = file.type.startsWith("video/");
```

### Rendering Logic

```javascript
if (isVideo) {
  mediaHtml = `<video controls src="" style="..."></video>`;
} else {
  mediaHtml = `<img src="" alt="${escapeHtml(photo.filename)}" />`;
}
```

### Styling

Videos share the same container styling as images:
- Full width within container
- `max-height: 80vh` to prevent oversized videos
- `object-fit: contain` to preserve aspect ratio

## Testing Checklist

- [ ] Upload single video file
- [ ] Upload multiple videos at once
- [ ] Upload mixed images and videos
- [ ] Video playback works with controls
- [ ] Video can be reordered via drag-and-drop
- [ ] Video can be reordered via arrow buttons
- [ ] Video can be deleted
- [ ] Album with videos can be exported
- [ ] Album with videos can be imported
- [ ] Works in Chrome, Firefox, Safari, Edge

## Future Considerations

- Video thumbnail generation for gallery view
- Video compression before storage
- Playback position memory
- Video trimming/editing capabilities
