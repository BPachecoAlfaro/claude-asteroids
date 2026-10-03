# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no tests, no linter. UI text and code comments are in Spanish; keep that convention.

## Running

Open `index.html` in a browser, or serve statically: `npx serve .` (http://localhost:3000).

## Architecture

All game logic lives in `game.js`, loaded by `index.html` as a classic script (not a module). The canvas is fixed at 800×600 (`W`/`H` constants must match the `<canvas>` attributes in `index.html`).

- **Loop**: `loop()` → `update(dt)` → `draw()` via `requestAnimationFrame`. `dt` is in seconds and clamped to 0.05; all speeds are px/s, so keep physics dt-based.
- **State machine**: global `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it up front. `'dead'` is a 2s respawn timer (asteroids keep moving); `'gameover'` restarts with Space via `initGame()`.
- **Entities** (`Ship`, `Asteroid`, `Bullet`, `Particle`): classes with `update(dt)` / `draw()` and a `dead` flag. Dead entities are removed by reassigning arrays with `.filter()` each frame, not by splicing.
- **World is toroidal**: positions wrap with `wrap()`. Collision uses plain circle distance (`dist`) with no wrap-aware checks.
- **Asteroid size tables**: `RADII`, `SPEEDS`, `POINTS` are indexed by size 1–3 (index 0 unused). Smaller asteroids are worth more points. `split()` yields two asteroids of `size - 1`.
- **Input**: `keys` (held) and `justPressed` (edge-triggered, consumed by calling `pressed(code)`). Use `pressed` for one-shot actions like shooting; use `keys` for continuous ones.
- **Collisions**: ship is only vulnerable when `ship.invincible <= 0` (3s after reset, shown by blinking in `Ship.draw`). Ship-vs-asteroid uses `ship.radius + a.radius * 0.82` as a forgiving hitbox.
- Level progression: clearing all asteroids calls `nextLevel()`, which spawns `3 + level` large asteroids outside a safe radius around the center.

Power-ups and the shooting-star asteroid were removed from the game (see git history); the README still mentions them in its description.
