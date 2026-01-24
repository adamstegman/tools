# Movie & TV Watchlist (TMDB)

A single-file website to **search movies and TV shows via the TMDB API** and **save a personal watchlist** (stored in your browser) so you can pick something later.

## How to use

1. Open `index.html` in a browser.
2. Paste your **TMDB credential**:
   - **v3 API key** (32-character key), or
   - **v4 access token** (long token / JWT-like string)
3. Search, then click **“Save to watchlist”** on anything you want to keep.
4. Use **“Pick something”** to choose a random unwatched item from your filtered list.

## Running locally (optional)

You can open the file directly, but a tiny local server is often nicer:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/tmdb-watchlist/`.

## Data & privacy

- **Stored locally**: your TMDB credential (if you choose to store it) and your watchlist are saved in your browser’s `localStorage`.
- **Network**: requests go directly from your browser to `api.themoviedb.org`.
- **Export/import**: use the built-in JSON export/import to back up or move your list.

## TMDB notice

This product uses the TMDB API but is not endorsed or certified by TMDB.

