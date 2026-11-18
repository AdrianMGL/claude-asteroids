# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

No build, bundler, package manager, linter, or test suite. There are no dependencies (`package.json` does not exist).

- Run: open `index.html` directly in a browser, or serve the folder with `npx serve .` and visit `http://localhost:3000`.
- Tests and lint: none configured. Verify changes by running the game in a browser.

## Architecture

The whole game is two files: `index.html` (an 800x600 `<canvas id="canvas">`, inline CSS, loads `game.js`) and `game.js` (all logic, no modules).

`game.js` is organized in sections marked by `// ── … ──` comments: Input, Utils, Bullet, Asteroid, Ship, Particle, Estado del juego (game state), Update, Draw, and the main loop. Read these together to follow a frame.

**Frame loop.** `loop(ts)` computes `dt` in seconds, clamped to 0.05 so a stalled tab does not teleport objects. It then calls `update(dt)` and `draw()`. Everything is advanced with `dt`, so keep new movement frame-rate independent.

**Game state machine.** The global `state` is one of `'playing'`, `'dead'`, or `'gameover'`:
- `playing`: normal update. Ship fires, bullets and asteroids move, collisions resolve, then the level check runs.
- `dead`: the ship was hit and lives remain. Asteroids and particles keep moving for `deadTimer` (2 s), then `ship.reset()` puts the ship back with a 3 s invincibility window.
- `gameover`: all lives are gone. Space calls `initGame()`.

`killShip()` is the only place lives decrease. `nextLevel()` runs when `asteroids.length === 0`. It spawns `3 + level` asteroids and keeps score and lives.

**Input.** `keys` tracks held keys by `e.code`. `justPressed` plus `pressed(code)` gives one-shot presses. Use `pressed()` for actions that should fire once per press (Space) and `keys[...]` for continuous actions (rotate, thrust). `pressed()` clears the flag when read.

**Collisions.** Bullet vs asteroid uses center distance < `asteroid.radius`. Ship vs asteroid uses `ship.radius + a.radius * 0.82`, and is skipped while `ship.invincible > 0`. When an asteroid is hit, `split()` returns two smaller asteroids (size 3 → 2 → 1), and those replace it in the array.

**Size tables.** `RADII`, `SPEEDS`, and `POINTS` are indexed by asteroid `size` (1 small, 2 medium, 3 large). Index 0 is unused. Scoring: large 20, medium 50, small 100, as in the README.

**World wrap.** The playfield is a torus. Ship, bullets, and asteroids use `wrap(v, max)` on both axes. Particles do not wrap and are simply removed when they expire.

**Rendering.** Everything is drawn with `ctx` in immediate mode, every frame, after `fillRect` clears the canvas. Asteroids are randomly either a stroked polygon (`verts`, generated once per asteroid) or the `asteroid.png` image (`useImg`, chosen at construction).

## Caveats

- `asteroid.png` is referenced in `game.js` but is not in the repository. Asteroids silently fall back to polygons, because `draw()` checks `asteroidImg.complete` and `naturalWidth`.
- `README.md` is partly stale. It describes power-ups and a shooting star, and recent commits removed both from `game.js`. Trust the code over the README for gameplay.
