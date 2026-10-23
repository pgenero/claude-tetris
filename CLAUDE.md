# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no `package.json`, no build step, no bundler/transpiler, no test suite or linter. User-facing text and the README are in Spanish; keep new UI strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (recommended):

```bash
python -m http.server 8000   # then open http://localhost:8000
npx serve .
```

There is nothing to build, lint, or test — verify changes by playing in the browser.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel HUD, `#next-canvas`, shared `#overlay`), `style.css` (dark/retro theme), and `game.js` (all logic). `game.js` is a classic script loaded at the end of `<body>` (not an ES module); it uses `'use strict'` and module-level mutable globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`, …) that `init()` resets. Restart = calling `init()` again.

Key conventions in `game.js`:

- **Piece type = color index = cell value.** `PIECES[t]` matrices are filled with the value `t`, `COLORS[t]` is its color, and `board[r][c]` stores that same value (0 = empty). Index 0 of `PIECES`/`COLORS` is `null`; keep the three in sync when adding/changing pieces. `randomPiece()` hardcodes 7 types.
- **Shapes are square matrices**; rotation is `rotateCW` (transpose + reverse) and `tryRotate` applies simple wall kicks `[0, -1, 1, -2, 2]` (horizontal only, no SRS).
- **`collide(shape, x, y)`** is the single source of truth for movement validity; cells with `y < 0` are allowed (above the board).
- **Game loop** (`loop`) uses `requestAnimationFrame`, accumulating `dt` into `dropAccum` and stepping the piece down once it reaches `dropInterval`. Gravity, soft drop and hard drop all end in `lockPiece()` → `merge()` → `clearLines()` → `spawn()`; `spawn()` triggers `endGame()` if the new piece collides immediately.
- **Scoring/level**: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row. Level = `floor(lines/10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms, recomputed only in `clearLines()`.
- **Rendering** is a full redraw each frame (`draw()`: grid, board, ghost at alpha 0.2, current piece). `drawBlock()` is shared by the board and the next-piece preview; `drawNext()` assumes a 4×4 grid of 30px cells in the 120×120 `#next-canvas`.
- The same `#overlay` is reused for both PAUSA and GAME OVER, with title/score text set in `togglePause()` / `endGame()`.

## Gotchas

- Board dimensions are duplicated: if you change `COLS`, `ROWS` or `BLOCK`, also update the `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
- Input is handled in a single `keydown` listener using `e.code` (`ArrowLeft/Right/Up/Down`, `KeyX`, `Space`, `KeyP`); keep the controls list in `index.html` and the README table in sync when changing bindings.
