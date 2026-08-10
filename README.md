# CYBER-SEC CON

A browser-based point-and-click / text-adventure hybrid set at a fictional hacker convention, grounded in real hacker-culture history (DEF CON, 2600/HOPE, ShmooCon, HackMiami) and current cybersecurity-industry facts.

No build tools, no backend, no API keys — a single self-contained HTML file with all art and logic embedded.

**[Play it live](https://malcolm1014.github.io/cybercon/)**

## Files

| File | What it is |
|---|---|
| `index.html` | The deployed copy GitHub Pages serves — regenerated from `cybercon.html` before each push |
| `cybercon.html` | **The file you edit.** Everything — scenes, rooms, puzzles, NPCs, items — lives in one `<script>` block |
| `ADDING_CONTENT.md` | Guide to adding rooms, puzzles, NPCs, items, and lore without writing new engine code |

## Running locally

Open `cybercon.html` directly in a browser. That's it — no server needed.

## Editing

See [`ADDING_CONTENT.md`](ADDING_CONTENT.md) for the full guide. The short version: everything is fill-in-the-blank editing inside `cybercon.html`, and typing `VALIDATE` in the game's terminal checks every room, exit, hotspot, and puzzle for you.

## Publishing an update

```sh
cp cybercon.html index.html
git add -A
git commit -m "..."
git push
```

GitHub Pages redeploys automatically within a minute or two of a push to `main`.
