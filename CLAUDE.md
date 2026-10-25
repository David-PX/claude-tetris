# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build tools, no package manager — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

There is no build/lint/test tooling. To run it:

```bash
start index.html        # Windows: open directly in the browser
# or serve it locally, e.g.:
npx serve .
python3 -m http.server 8000
```

There are no automated tests. Verify changes by opening the game in a browser and playing it (see the golden-path checks below).

## Architecture

Everything lives in `game.js` (~300 lines), organized around a single game loop:

- **Board model**: a `ROWS × COLS` (20×10) matrix of cells; each cell is `0` (empty) or a color index `1–7` identifying which piece filled it.
- **Pieces**: the 7 standard tetrominoes defined as square matrices in `PIECES`. Rotation (`rotateCW`) is transpose + row-reverse.
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells.
- **Wall kicks** (`tryRotate`): on a rotation collision, retries the rotation shifted ±1/±2 columns before giving up.
- **Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates elapsed time in `dropAccum` and drops the piece one row once `dropInterval` is exceeded, then calls `lockPiece()`.
- **Line clearing** (`clearLines`): scans bottom-to-top, removes full rows, unshifts empty rows at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current level; hard drop adds 2 pts/cell traveled, soft drop adds 1 pt/row.
- **Leveling**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- Game over is triggered in `spawn()` when a freshly spawned piece immediately collides, firing `endGame()`.

Control flow: `init()` builds the board, seeds `next`, calls `spawn()` (which promotes `next` to `current` and generates a new `next`), then starts the `loop`. Keyboard input (arrows, `X`/rotate, space/hard-drop, `P`/pause) is handled by a single `keydown` listener that mutates `current`/game state directly — there is no separate input abstraction layer.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If you change `COLS`, `ROWS`, or `BLOCK`, also update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
