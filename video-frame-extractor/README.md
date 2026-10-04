# Video Frame Extractor

A local-first browser tool that splits MP4 video clips into constituent still images, enables fast visual comparison and triage in a Filmstrip Studio, and downloads selected favorites as a single `.zip` archive.

## Features

- **Progressive Frame Extraction**: Frames stream live into the filmstrip as they are extracted so you can begin inspecting immediately without waiting for long videos to finish.
- **Configurable Sampling & In/Out Trimming**: Extract at 1, 3, 6, 10, 15, or all frames (30 fps), with optional start and end trim handles to isolate key segments.
- **Filmstrip Studio**: Persistent large inspection stage on top paired with a horizontal scrubbable filmstrip of thumbnails on the bottom.
- **Fast Flipbook Navigation**: Use `←` / `→` arrow keys to rapidly flip between adjacent frames with zero lag to spot subtle expressions or motion differences.
- **Keyboard-Driven Selection**: Press `Spacebar` or `Enter` to keep/deselect the currently inspected frame.
- **Zero-Dependency `.zip` Export**: Packages all kept frames in memory using a lightweight, pure JavaScript ZIP binary encoder into a clean download.
- **Privacy-First & Local-Only**: All video decoding, canvas manipulation, and archive generation happens entirely in your browser with no network or server dependencies.
- **Light & Dark Mode**: Adapts automatically to system theme preferences.

## Usage

1. Open `index.html` in a modern web browser.
2. Drag and drop an MP4 video clip (or click to browse).
3. Set optional Start (In) and End (Out) trim points and select your desired frame rate (default: 3 fps).
4. Click **Start Extraction**.
5. Use `←` / `→` arrow keys to navigate frames, and press `Space` to keep the ones you like.
6. Click **Download Kept (.zip)** to save the selected images to your hard drive.

## Technical Details

- Single-file HTML tool with zero build steps or external dependencies.
- HTML5 Video element seeking paired with 2D Canvas extraction.
- Dual-tier rendering: 160px downsampled thumbnails for 60fps filmstrip DOM performance, full source resolution for the main inspection canvas and final export.
- Pure JS uncompressed PKZIP (Store method 0) binary generator using standard CRC-32 checksums.
