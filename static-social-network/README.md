# Parasocial Network

A single-file HTML “social network” demo with **no login**.

- **Text posts**: a scrolling feed of static placeholder posts (rendered in-browser).
- **TMDB-powered accounts**: paste a TMDB API key + a list of names to populate the “accounts” in the feed by searching TMDB’s Person API (highest popularity match wins). Names with no results are marked “no personality.”

## Running locally

Any static server works:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/static-social-network/`.

## Notes

- The “other users” and their posts are **static placeholder content** rendered in-browser.
- Avatars use `picsum.photos` (placeholder).

