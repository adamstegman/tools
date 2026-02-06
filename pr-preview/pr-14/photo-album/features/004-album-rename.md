# Feature Spec: Album Rename

**Status:** Implemented
**PR:** #10
**Date:** 2026-01-25

## Overview

Add the ability to rename an existing album without losing its photos or metadata.

## Problem Statement

Users create albums with names that may become outdated or incorrect:
- Typos in the original name
- Events that get renamed ("John's Party" → "John's 30th Birthday")
- Generic names that need specificity ("Vacation" → "Hawaii 2026")

Without rename functionality, users must:
1. Create a new album with the desired name
2. Export the old album
3. Import into the new album
4. Delete the old album

This is tedious and error-prone.

## Requirements

### Functional Requirements

1. Users can rename an album from within the album view
2. Rename preserves all photos and their order
3. Rename preserves album creation date
4. Cannot rename to a name that already exists
5. Cannot rename to an empty name
6. UI provides clear feedback on success/failure

### Non-Functional Requirements

1. Rename operation should be atomic (all-or-nothing)
2. If rename fails, original album remains intact
3. UI should be intuitive and discoverable

## Design Decisions

### Decision 1: UI Trigger Location

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Album list context menu | Right-click on album | Discoverable from list | Requires context menu infra |
| B. Album view header button | "Rename" button near title | Visible when viewing | Clutters header |
| C. Click on title to edit | Inline editing | Clean UI, intuitive | Not obviously editable |
| D. Edit icon next to title | Small pencil icon | Clear affordance, minimal | Extra element |

**Decision:** Option D - Edit icon button next to album title

**Rationale:**
- Clear visual affordance that the title is editable
- Doesn't clutter the UI (small icon)
- Follows common patterns (Google Docs, Notion, etc.)
- Works well on both desktop and mobile

### Decision 2: Edit Mode UI

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Modal dialog | Popup with input field | Clear separation | Disruptive, extra clicks |
| B. Inline text input | Replace title with input | Seamless, fast | Need save/cancel affordance |
| C. Separate page | Navigate to rename page | Clear workflow | Over-engineered |

**Decision:** Option B - Inline text input replacing the title

**Rationale:**
- Fastest path to rename (click, type, save)
- Keeps user in context
- Follows inline editing patterns users know
- Save/Cancel buttons provide clear actions

### Decision 3: Save/Cancel Affordance

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Enter to save, Escape to cancel | Keyboard only | Fast for power users | Not discoverable |
| B. Save/Cancel buttons | Explicit buttons | Clear, accessible | Takes space |
| C. Both keyboard and buttons | Full coverage | Best of both | Most implementation |

**Decision:** Option C - Both keyboard shortcuts and visible buttons

**Rationale:**
- Buttons make actions discoverable for all users
- Keyboard shortcuts enable power users
- Accessible (buttons are focusable and have labels)

### Decision 4: Data Migration Strategy

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Update album name in place | Modify existing record | Simple if supported | IndexedDB keyPath issue |
| B. Create new, migrate photos, delete old | Full migration | Works with keyPath | More operations, risk |
| C. Add separate "displayName" field | Keep key, show different name | No migration | Confusing data model |

**Decision:** Option B - Create new album, migrate photos, delete old

**Rationale:**
IndexedDB uses `name` as the keyPath for the albums store. You cannot change an object's keyPath after creation. Therefore:

1. Create new album record with new name
2. Update all photos' `albumName` field to new name
3. Delete old album record

This is done in a careful sequence to prevent data loss:
- If step 1 fails: no changes made
- If step 2 fails: new album exists but old still has photos (recoverable)
- If step 3 fails: duplicate album (old can be manually deleted)

### Decision 5: Handling Name Conflicts

**Options Considered:**

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A. Silent fail | Just don't save | No error UI needed | Confusing for user |
| B. Auto-increment | Add "(2)" suffix | Always succeeds | User didn't ask for that name |
| C. Error message | Show conflict error | Clear feedback | User must fix |
| D. Merge albums | Combine into existing | Powerful | Destructive, unexpected |

**Decision:** Option C - Show error message, keep user in edit mode

**Rationale:**
- User should know exactly why their action didn't work
- User can then choose: pick a different name or cancel
- No surprising automatic behavior

## Implementation

### HTML Structure

```html
<div style="display: flex; align-items: center; gap: 0.5rem;">
  <!-- View mode -->
  <h2 id="current-album-name">Album Name</h2>
  <button id="rename-album-btn" class="icon-btn" title="Rename album">✏️</button>

  <!-- Edit mode (hidden by default) -->
  <input type="text" id="rename-album-input" class="hidden" />
  <button id="save-rename-btn" class="hidden">Save</button>
  <button id="cancel-rename-btn" class="hidden secondary">Cancel</button>
</div>
```

### State Management

```javascript
function enterRenameMode() {
  el.currentAlbumName.classList.add("hidden");
  el.renameAlbumBtn.classList.add("hidden");
  el.renameAlbumInput.classList.remove("hidden");
  el.saveRenameBtn.classList.remove("hidden");
  el.cancelRenameBtn.classList.remove("hidden");

  el.renameAlbumInput.value = state.currentAlbum.name;
  el.renameAlbumInput.focus();
  el.renameAlbumInput.select();
}

function exitRenameMode() {
  el.currentAlbumName.classList.remove("hidden");
  el.renameAlbumBtn.classList.remove("hidden");
  el.renameAlbumInput.classList.add("hidden");
  el.saveRenameBtn.classList.add("hidden");
  el.cancelRenameBtn.classList.add("hidden");
}
```

### Database Operation

```javascript
async function renameAlbum(oldName, newName) {
  // Validate
  if (!newName.trim()) {
    throw new Error("Album name cannot be empty");
  }

  const existingAlbum = await getAlbum(newName);
  if (existingAlbum) {
    throw new Error("An album with this name already exists");
  }

  const oldAlbum = await getAlbum(oldName);
  if (!oldAlbum) {
    throw new Error("Album not found");
  }

  // Create new album
  const newAlbum = {
    name: newName,
    photoIds: oldAlbum.photoIds,
    createdAt: oldAlbum.createdAt  // Preserve original date
  };
  await createAlbumRecord(newAlbum);

  // Migrate photos
  const photos = await getPhotosForAlbum(oldName);
  for (const photo of photos) {
    photo.albumName = newName;
    await updatePhoto(photo);
  }

  // Delete old album
  await deleteAlbumRecord(oldName);
}
```

### Event Handlers

```javascript
el.renameAlbumBtn.addEventListener("click", enterRenameMode);
el.cancelRenameBtn.addEventListener("click", exitRenameMode);

el.saveRenameBtn.addEventListener("click", handleRename);
el.renameAlbumInput.addEventListener("keydown", (e) => {
  if (e.key === "Enter") handleRename();
  if (e.key === "Escape") exitRenameMode();
});

async function handleRename() {
  const newName = el.renameAlbumInput.value.trim();
  const oldName = state.currentAlbum.name;

  if (newName === oldName) {
    exitRenameMode();
    return;
  }

  try {
    await renameAlbum(oldName, newName);
    state.currentAlbum.name = newName;
    el.currentAlbumName.textContent = newName;
    exitRenameMode();
    showStatus(`Album renamed to "${newName}"`, "ok");
  } catch (error) {
    showStatus(error.message, "bad");
    el.renameAlbumInput.focus();
  }
}
```

## Testing Checklist

- [ ] Click edit icon enters rename mode
- [ ] Input is pre-filled with current name and selected
- [ ] Enter key saves the new name
- [ ] Escape key cancels without saving
- [ ] Save button saves the new name
- [ ] Cancel button exits without saving
- [ ] Photos remain after rename
- [ ] Photo order preserved after rename
- [ ] Album creation date preserved
- [ ] Cannot rename to empty string
- [ ] Cannot rename to existing album name
- [ ] Error message shown for conflicts
- [ ] Album list shows new name after returning
- [ ] Export after rename uses new name

## Accessibility

- Edit button has `title` attribute for tooltip
- Input is focusable and labeled
- Save/Cancel buttons are keyboard accessible
- Error messages are announced (could add `aria-live`)

## Future Considerations

- Undo functionality for accidental renames
- Rename from album list (without opening album)
- Bulk rename with pattern matching
- Rename history/audit log
