# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Vanilla Tetris implementation using HTML5 Canvas, CSS, and plain JavaScript (ES6+). No dependencies, no build step, no package.json. The entire game logic lives in a single file: `game.js` (~300 lines).

## Running the game

No install/build required. Either open `index.html` directly in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no test suite, linter, or build/bundling process in this repo — changes are verified by playing the game in a browser.

## Architecture

Three files cooperate directly via DOM element IDs (no module system, no bundler):

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600, the 10×20 grid at `BLOCK=30`px/cell), `<canvas id="next-canvas">` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and the pause/game-over `#overlay`.
- **`style.css`** — dark/retro arcade visuals only.
- **`game.js`** — all game logic, structured around this state and flow:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
  - **Pieces**: `PIECES` are square matrices; rotation is done via `rotateCW` (transpose + reverse), not stored per-orientation.
  - **Collision**: `collide(shape, ox, oy)` checks board bounds and cell overlap; it underlies movement, rotation, ghost-piece projection, and spawn-blocking (game over).
  - **Wall kicks**: `tryRotate()` rotates then tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found.
  - **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and advances the piece one row (or calls `lockPiece()`) once `dropAccum >= dropInterval`.
  - **Locking/scoring**: `lockPiece()` → `merge()` (bake piece into `board`) → `clearLines()` (scan bottom-up, splice full rows, unshift empty rows at top) → `spawn()`.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell traveled, soft drop adds 1 pt/row.
  - **Leveling/speed**: level increases every 10 lines cleared; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
  - **Game over**: triggered in `spawn()` when the newly spawned piece immediately collides.

All rendering goes through `drawBlock()` onto either the main `ctx` or the next-piece `nextCtx`; there is no separate renderer abstraction.

### Tunable constants (all in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
