# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A vanilla JavaScript implementation of Tetris using HTML5 Canvas. No dependencies, no build step, no package manager — just `index.html`, `style.css`, and `game.js`. The full README (in Spanish) is in `README.md` and documents controls, mechanics, and tunable constants in detail.

## Running the game

There is no build/lint/test tooling in this repo. To run it, just serve or open the static files:

```bash
start index.html          # Windows: open directly in the browser
python3 -m http.server 8000   # or any static file server, then open localhost:8000
```

Changes to `game.js`/`style.css`/`index.html` take effect on browser reload — no compile step.

## Architecture

Everything lives in three files with no module system (`game.js` is loaded as a plain `<script>`, not a module). State is a set of top-level mutable variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) shared across all functions in `game.js`.

Key pieces, in `game.js`:

- **Board model**: a `ROWS × COLS` matrix (`createBoard`); each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: defined as square matrices in `PIECES`. Rotation is done generically via matrix transpose+reverse in `rotateCW` — there's no piece-specific rotation logic.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used both for movement and for projecting the ghost piece.
- **Wall kicks** (`tryRotate`): after rotating, tries x-offsets `[0, -1, 1, -2, 2]` in order and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded; calls `draw()` every frame regardless.
- **Line clear** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at top; recalculates `level` (every 10 lines) and `dropInterval` (`max(100, 1000 - (level-1)*90)`).
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level` for clears; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Ghost piece** (`ghostY`): projects the current piece straight down via repeated `collide` checks; drawn with `globalAlpha = 0.2`.
- **Spawn/Game over**: `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires and the overlay shows GAME OVER.

Control flow: `init()` resets all state and kicks off the `loop`. Keyboard input is handled by a single `keydown` listener that dispatches on `e.code` (arrows, `Space`, `KeyX`, `KeyP`) and is gated by `paused`/`gameOver`. `P` toggles pause by cancelling/restarting the `requestAnimationFrame` loop and toggling the shared `#overlay` element (also reused for the GAME OVER state).

## Tunable constants (`game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval` (initial drop speed). If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK` and `ROWS × BLOCK`).
