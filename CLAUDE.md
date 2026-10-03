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

- **Power-ups (triple shot + shield)**: `PowerUp` has a `type` (`'triple' | 'shield'`). Each level drops exactly one of each, at the position of the `atKill`-th asteroid destroyed that level (two distinct random targets stored in `dropPlan`, planned in `spawnAsteroids()`, always reachable since each large asteroid yields 7 kills). Active items live in the `powerUps` array. All asteroid destruction goes through `destroyAsteroid()` (score, explosion, split, drop check), used by both bullets and the shield.
  - Triple: pickup sets `tripleTimer` (5s); while > 0, `Ship.tryShoot()` fires 3 bullets in a fan.
  - Shield: pickup sets `shieldTimer` (5s); the first asteroid hit while > 0 zeroes it, destroys that asteroid via `destroyAsteroid()` and gives `SHIELD_GRACE` (1s) of `ship.invincible`. Drawn in `Ship.draw()` as a circle that blinks in the last second.
  - Both timers are lost on death and survive `nextLevel()`.

The shooting-star asteroid was removed from the game (see git history); the README still mentions it in its description.
