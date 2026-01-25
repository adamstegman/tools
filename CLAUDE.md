# CLAUDE.md

This file provides guidance for AI assistants working with this codebase.

## Project Overview

**HTML Tools** is a collection of single-file HTML tools. Each tool is a self-contained, standalone HTML file that runs entirely in the browser with no build process, external dependencies, or server-side code required.

### Repository Structure

```
/
├── index.html                 # Landing page listing all tools
├── photo-album/               # Photo album manager with drag-and-drop
│   ├── index.html
│   └── README.md
├── sales-tax-estimator/       # CSV-based sales tax calculator
│   └── index.html
├── static-social-network/     # Demo social network with placeholder posts
│   ├── index.html
│   └── README.md
├── tmdb-watchlist/            # Movie/TV watchlist using TMDB API
│   ├── index.html
│   └── README.md
├── pr-preview/                # Generated PR preview deployments (do not edit)
└── .github/workflows/         # GitHub Actions
    └── deploy-pr-preview.yml  # Deploys PR previews to GitHub Pages
```

## Architecture Principles

### Single-File HTML Tools

Each tool is **one self-contained `index.html` file** containing:
- HTML structure
- `<style>` block with CSS
- `<script type="module">` block with JavaScript

This design ensures:
- No build process or bundling required
- Easy to run locally with any static server
- Simple to understand and modify
- Portable and shareable

### No External Dependencies

Tools should not rely on external JavaScript libraries, CSS frameworks, or CDNs. All code is inline within the HTML file.

### Privacy-First Local Storage

All tools store data locally in the user's browser:
- **IndexedDB**: For large data like photos/blobs (photo-album)
- **localStorage**: For smaller data like settings and watchlists (tmdb-watchlist)

No data is sent to external servers (except explicit API calls like TMDB).

## Code Conventions

### HTML Structure

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Tool Name</title>
    <meta name="description" content="Brief description of the tool" />
    <style>
      /* CSS here */
    </style>
  </head>
  <body>
    <!-- HTML here -->
    <script type="module">
      // JavaScript here
    </script>
  </body>
</html>
```

### CSS Conventions

- Use CSS custom properties (variables) for theming
- Support light and dark mode via `color-scheme: light dark`
- Use system color keywords (`Canvas`, `CanvasText`, `Highlight`, etc.)
- Responsive design with `clamp()` and media queries
- Common variable pattern:

```css
:root {
  color-scheme: light dark;
  --bg: Canvas;
  --fg: CanvasText;
  --muted: color-mix(in oklab, CanvasText 65%, Canvas 35%);
  --card: color-mix(in oklab, Canvas 92%, CanvasText 8%);
  --border: color-mix(in oklab, CanvasText 18%, Canvas 82%);
  --accent: Highlight;
  --accentText: HighlightText;
  --ok: #0b6b3a;
  --danger: #b00020;
  --shadow: 0 1px 2px rgba(0, 0, 0, 0.08);
  --radius: 12px;
}
```

### JavaScript Conventions

- Use `<script type="module">` for ES modules syntax
- Use modern JavaScript (async/await, optional chaining, etc.)
- Document functions with JSDoc comments
- Prefer `const` and `let` over `var`
- Use descriptive function and variable names

## Development Workflow

### Running Locally

Start any static file server from the repository root:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/` and click a tool, or go directly to a tool like `http://localhost:8000/photo-album/`.

### Making Changes

1. Edit the tool's `index.html` file directly
2. Refresh the browser to see changes
3. No compilation or build step needed

### Adding a New Tool

1. Create a new directory: `mkdir new-tool-name`
2. Create `new-tool-name/index.html` following the conventions above
3. Add the tool to the root `index.html` listing
4. Optionally create a `README.md` in the tool directory

## Git Conventions

### Commit Message Format

Use conventional commits with scope:

```
type(scope): description

Examples:
feat(photo-album): Add album rename functionality
fix(photo-album): Implement lazy loading to prevent blob resource exhaustion
```

Common types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `style`: Formatting changes

### Pull Request Workflow

- PRs automatically get preview deployments via GitHub Actions
- Preview URLs are posted as comments on PRs
- The `pr-preview/` directory contains generated preview files (managed by CI, do not edit manually)

## Tool-Specific Notes

### photo-album

- Uses IndexedDB for photo blob storage
- Implements lazy loading to prevent browser memory exhaustion
- Supports drag-and-drop reordering
- Export/import albums as JSON with base64-encoded images

### tmdb-watchlist

- Requires user's TMDB API key (v3) or access token (v4)
- Stores credentials and watchlist in localStorage
- Makes direct API calls to api.themoviedb.org

### static-social-network

- Demo/placeholder social network
- Uses TMDB for character avatars
- Posts are generated placeholder content

### sales-tax-estimator

- Processes CSV transaction files
- Applies category-specific tax rates
- All processing done client-side

## Testing

There is no automated test suite. Manual testing workflow:

1. Run local static server
2. Test the specific tool in browser
3. Verify functionality works across browsers (Chrome, Firefox, Safari, Edge)
4. Test both light and dark mode
5. Test responsive design at various viewport sizes
