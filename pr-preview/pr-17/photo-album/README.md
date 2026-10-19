# Photo Album

A browser-based photo album manager that lets you create multiple albums, upload photos, and organize them with drag-and-drop reordering.

## Features

- **Multiple Albums**: Create and manage multiple photo albums, each with its own name
- **Photo Upload**: Upload multiple photos at once via file picker or drag-and-drop
- **Full-Width Display**: Photos displayed vertically at full width for easy viewing
- **Reordering**: Drag-and-drop photos to reorder them, or use up/down arrow buttons
- **Photo Removal**: Delete individual photos from albums
- **Album Management**: Create and delete entire albums
- **Export/Import**: Export albums as JSON files to share or backup, import them on other devices
- **Local Storage**: All photos stored locally in your browser using IndexedDB (no server required)
- **Privacy-First**: No data sent to any server - everything stays in your browser
- **Responsive Design**: Works on desktop and mobile devices
- **Dark Mode**: Automatically adapts to your system's light/dark mode preference

## Usage

1. Open `index.html` in a modern web browser
2. Create a new album by entering a name and clicking "Create Album"
3. Click on an album to open it
4. Upload photos by clicking the upload zone or dragging files onto it
5. Reorder photos by dragging them or using the ↑/↓ buttons
6. Delete photos using the × button that appears when hovering over a photo
7. Export an album to share or backup using the "Export Album" button (downloads a JSON file)
8. Import an album on another device/browser using the "Import Album" button
9. Return to the album list with the "Back to Albums" button
10. Delete entire albums with the "Delete Album" button

## Technical Details

- Single-file HTML tool (no build process required)
- Uses IndexedDB for storing photo Blobs and album metadata
- Progressive enhancement with graceful degradation
- Accessible (keyboard navigation, ARIA labels, semantic HTML)
- No external dependencies

## Data Storage

Photos are stored locally in your browser's IndexedDB. The storage limit varies by browser but is typically:
- Chrome/Edge: Several hundred MB to several GB
- Firefox: Up to 2GB
- Safari: Up to 1GB

**Important**: Browser data can be cleared if you clear your browsing data. Consider exporting important albums regularly.

## Browser Support

Requires a modern browser with support for:
- IndexedDB
- HTML5 Drag and Drop API
- File API
- CSS Grid and Flexbox

Tested in Chrome, Firefox, Safari, and Edge.
