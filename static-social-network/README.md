# Static Social Network

A single-file HTML “social network” demo with **no login**.

- **Photos**: opens on a grid of placeholder photo posts.
- **Videos**: switch via the navbar to a scrolling feed of **autoplaying** placeholder videos (muted, loops, plays inline).

## Running locally

Any static server works:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/static-social-network/`.

## Notes

- The “other users” and their posts are **static placeholder content** rendered in-browser.
- Placeholder images use `picsum.photos`. Placeholder video uses a public MDN sample MP4.

