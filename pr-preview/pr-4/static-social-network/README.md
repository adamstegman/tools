# Parasocial Network

A single-file HTML “social network” demo with **no login**.

- **Text posts**: a scrolling feed of static placeholder posts (rendered in-browser).
- **TMDB-powered accounts**: for each account slot, search for a TV series, pick a result, then pick a character from the series’ Aggregate Credits. If you don’t pick one, the account is marked “no personality.”

## Running locally

Any static server works:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/static-social-network/`.

## Notes

- The “other users” and their posts are **static placeholder content** rendered in-browser.
- Avatars use `picsum.photos` (placeholder).

