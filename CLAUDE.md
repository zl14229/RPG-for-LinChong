# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"林冲·命运之择 — 风雪山神庙" is a Chinese-language interactive fiction (互动叙事) web game based on Chapter 10 of the classic novel 《水浒传》 (Water Margin). The player takes on the role of Lin Chong (林冲) and makes choices at key story nodes that lead to different endings.

**Tech stack**: Pure HTML/CSS/JavaScript, no frameworks or build tools. Each page is a standalone HTML file with all styles and scripts inlined. Deploy by opening any `.html` file in a browser or serving with any static HTTP server.

## Project Structure

```
index.html       — Landing page with story intro and start button
node1.html~node4.html — Story decision nodes (4 nodes total)
revenge.html     — Classic ending: 雪夜的复仇 (canonical per original novel)
rebel.html       — Alternate ending: 逼上梁山 (early rebellion)
death.html       — Bad ending: 命尽于此 (death)
background/      — Local background images (node1.png~node4.png, revenge.png, death_1B.png, etc.)
bgm/             — BGM audio files (soft.mp3 for nodes, high.mp3 for revenge ending, bgm.mp3)
节点.txt          — Node/choice diagram defining the branching logic
原文.txt          — Original novel chapter text (source material)
```

## Story Branching Logic

4 decision nodes, 3 ending types:

- **Death (死)**: Passive/safe choices lead to death (routes: 1B, 2B, 3B, 4B)
- **Rebel (反)**: Aggressive/early-rebellion choices lead to Liangshan (routes: 1C, 2C, 4C)
- **Revenge (复仇)**: Following the original novel path (1A→2A→3A→4A) leads to the classic ending

Node flow: `index → node1 → node2 → node3 → node4` with branches to endings at each step. Query parameters (`?from=1B`, `?from=2C`, etc.) track the player's path.

## Key Architecture Patterns

### Inline JS per page (duplicated across all HTML files)

- **Snow canvas**: Animated snowfall particle effect on `<canvas id="snow-canvas">`. Particle count varies by page (N=60~100). Canvas resized on window resize.
- **BGM via `<audio>` + `sessionStorage`**: Three MP3 tracks in `bgm/`. Cross-page playback continuity is achieved via `sessionStorage` keys `bgmSrc` and `bgmTime` — on load, if the stored source matches the current page's audio src, playback resumes from the stored time. `soft.mp3` plays during nodes, `high.mp3` plays on the revenge ending.
- **Typewriter effect**: `typeWriter(element, html, speed, callback)` renders text character by character. Clicking skips to the end.
- **Choice reveal**: Choices are hidden behind a click-to-reveal button (`reveal-btn`). After the typewriter finishes, the reveal button appears. Clicking it shows choices via `revealChoices()`.
- **Ending return handling**: Ending pages set `sessionStorage.setItem('fromEnding','1')`. If this key exists on a node page, choices are auto-revealed (skips the typewriter intro), allowing the player to go back and try other paths.
- **Music toggle button** (`#music-btn`): Toggles play/pause. Auto-plays on first interaction. State reflected via `.playing` class and ♫/♩ icon.

### CSS Conventions

- All units use `vh`/`vw`/`%` for responsive scaling
- Dark theme: background `#0a0a12`, text `#e0d5b8`/`#d4c9a8`, red accent `#d43a3a`, gold `#d4b050`
- `backdrop-filter: blur()` for glass-morphism on interactive elements
- Background images via `body::before` with `linear-gradient` overlays
  - `index.html` uses Unsplash URL; all other pages use local files from `background/`

### Common Page Layout

- Node pages: `.container` with `.header` → `.content-area` (status-box + choices) → back link
- Ending pages: `.header` → `.story-box` → `.epilogue` → `.actions` (prev node + home links)

## Development

No build tools, package managers, or compilation steps. Edit any `.html` file directly and refresh the browser.

- **Preview**: `python -m http.server 8000` or `npx serve .` from the project root
- **Adding a new node**: Copy an existing node file, update story text, choices, and badge title. Choices link to other nodes with `href="<file>.html"` or to endings with `?from=<route>` query param.
- **Adding a new ending**: Create a new HTML file following the pattern in `revenge.html` or `death.html`. Extract key text from `原文.txt`. Customize BGM source and snow particle count. Include `fromEnding` sessionStorage write in `beforeunload`.
- **Styling**: CSS is entirely in `<style>` tags. Each page has its own background image path.
