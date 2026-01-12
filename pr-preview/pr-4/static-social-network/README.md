# Parasocial Network

A single-file HTML “social network” demo with **no login**.

- **Text posts**: a scrolling feed of static placeholder posts (rendered in-browser).
- **TMDB-powered accounts**: add accounts one-by-one by searching for a TV series, picking a result, then picking a character from the series’ Aggregate Credits. Paste a **TMDB v3 API key** or **v4 access token**; the account name is the character name.

## Running locally

Any static server works:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/static-social-network/`.

## Notes

- The “other users” and their posts are **static placeholder content** rendered in-browser.
- Avatars use `picsum.photos` (placeholder).

