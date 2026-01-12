# Parasocial Network

A single-file HTML “social network” demo with **no login**.

- **Text posts**: a scrolling feed of static placeholder posts (rendered in-browser).
- **TMDB-powered accounts**: for each account slot, search for a TV series, pick a result, then type a character name. The app fetches the series’ Aggregate Credits and fuzzy-matches your character name; if it can’t find one, the account is marked “no personality.”

## Running locally

Any static server works:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/static-social-network/`.

## Notes

- The “other users” and their posts are **static placeholder content** rendered in-browser.
- Avatars use `picsum.photos` (placeholder).

