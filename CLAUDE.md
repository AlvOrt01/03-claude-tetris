# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vanilla JavaScript Tetris implementation using HTML5 Canvas and CSS. No dependencies, no build process—just open and play. The game includes standard Tetris mechanics: 7-piece types, rotation with wall kicks, scoring system, level progression, pause/game-over states.

## Running the Game

**No installation or build step required.**

```bash
# Option 1: Direct open (macOS)
open index.html

# Option 2: Local server (recommended — fixes any CORS issues)
python3 -m http.server 8000    # Python 3
npx serve .                     # Node.js
php -S localhost:8000           # PHP
```

Then navigate to `http://localhost:8000` (or just open the file).

## Architecture & Key Concepts

**Game Loop**: `init()` → `requestAnimationFrame(loop)` → `draw()` each frame. The loop accumulates elapsed time and triggers piece drops when `dropAccum >= dropInterval`.

**Board Model**: 2D array (`ROWS × COLS = 20 × 10`) where each cell holds `0` (empty) or a color index (1–7) identifying a piece type.

**Piece Representation**: Pieces are 4×4 matrices with color indices at occupied cells, 0 elsewhere. Rotation is implemented via matrix transpose + row reversal (`rotateCW`).

**Collision Detection** (`collide`): Checks if a piece at position (ox, oy) overlaps the board boundary or existing blocks.

**Wall Kicks** (`tryRotate`): After rotation fails collision check, tries up to 5 horizontal offsets (0, ±1, ±2 columns) to find a valid rotation state.

**Cleanup** (`clearLines`): Scans from bottom up for full rows; removes them and inserts blank rows at the top. Rebuilds level/speed/score based on lines cleared.

**Ghost Piece**: Rendered at 20% opacity showing where the current piece will land.

**Scoring**: Uses classic Tetris formula—points per cleared lines (1:100, 2:300, 3:500, 4:800) multiplied by current level. Hard drop adds 2 points per cell; soft drop adds 1 per row.

**Level & Speed**: Level increases every 10 lines. Drop interval decreases: `max(100, 1000 − (level − 1) × 90)` ms.

## Customization Points

All configurable constants are at the top of `game.js`:
- `COLS` / `ROWS`: Board dimensions (default 10 × 20)
- `BLOCK`: Pixel size per cell (default 30px)
- `COLORS`: RGB hex palette for each piece type
- `LINE_SCORES`: Point table for line clears
- `dropInterval`: Initial drop speed in ms

**Important**: If you change `COLS`, `ROWS`, or `BLOCK`, also update the `<canvas id="board">` width/height in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## File Layout

```
├── index.html       # DOM: canvas (#board, #next-canvas), HUD (score, lines, level), overlay
├── style.css        # Dark/retro arcade theme; flexbox layout
├── game.js          # All game logic: board, pieces, collision, rendering, input
└── README.md        # User-facing documentation (Spanish)
```

## Input Handling

Keyboard events are captured at document level (`keydown`). Special handling:
- `P`: Toggle pause (works even during game-over to restart cycle)
- During pause/game-over: arrow keys and hard/soft drop are blocked
- Hard drop prevents default space behavior (`preventDefault`)

Restart is triggered via button click on the overlay, which calls `init()`.
