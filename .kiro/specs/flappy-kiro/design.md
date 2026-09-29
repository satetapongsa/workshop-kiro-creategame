# Design Document — Flappy Kiro

## Overview

Flappy Kiro is a single-file browser game (`index.html`) built with plain HTML5, CSS, and JavaScript. All game logic runs inside a `requestAnimationFrame` loop that updates a shared state object and redraws a full-screen `<canvas>` element each tick. Assets (`ghosty.png`, `jump.wav`, `game_over.wav`) are loaded from the existing `/assets/` directory. No build tools, frameworks, or CDN dependencies are used.

---

## Architecture

The game follows a **state-machine / game-loop** pattern:

```
┌───────────────────────────────────────────────────┐
│                    index.html                     │
│                                                   │
│  ┌─────────┐   ┌──────────┐   ┌───────────────┐  │
│  │  State  │   │  Input   │   │ Asset Loader  │  │
│  │ Machine │◄──│ Handler  │   │ (img + audio) │  │
│  └────┬────┘   └──────────┘   └───────┬───────┘  │
│       │                               │           │
│  ┌────▼────────────────────────────────▼───────┐  │
│  │              Game Loop (rAF)                │  │
│  │   update(dt)  ──►  render(ctx)              │  │
│  └──────────────────────────────────────────── ┘  │
│                                                   │
│  Subsystems:                                      │
│   • PhysicsSystem   (gravity, velocity, position) │
│   • PipeSystem      (spawn, scroll, cull)         │
│   • CollisionSystem (AABB tests)                  │
│   • ScoreSystem     (increment, persist)          │
│   • AudioSystem     (play, graceful fail)         │
│   • Renderer        (layers: bg → clouds →        │
│                      pipes → ghost → HUD)         │
└───────────────────────────────────────────────────┘
```

### State Machine

```
         ┌──────────────────────────────────────────┐
         │                                          │
   ┌─────▼──────┐   input    ┌──────────────┐       │
   │    IDLE    ├───────────►│   PLAYING    │       │
   └────────────┘            └──────┬───────┘       │
                                    │ collision /   │
                                    │ boundary      │
                             ┌──────▼───────┐       │
                             │  GAME_OVER   │       │
                             └──────┬───────┘       │
                                    │ restart click │
                                    └───────────────┘
```

States: `'idle'` | `'playing'` | `'game_over'`

---

## Components and Data Models

### Constants (compile-time configuration)

```js
const CONFIG = {
  GRAVITY:        0.4,      // px/frame² downward acceleration
  FLAP_VELOCITY: -8,        // px/frame upward impulse on flap
  MAX_FALL_SPEED: 12,       // px/frame terminal downward velocity
  PIPE_SPEED:     3,        // px/frame leftward scroll
  PIPE_INTERVAL: 200,       // px between consecutive pipe spawn x positions
  PIPE_WIDTH:    60,        // px
  GAP_HEIGHT:   150,        // px vertical gap between top and bottom pipe
  GAP_Y:        200,        // px from top to the top edge of the gap (fixed)
  GHOST_WIDTH:   50,        // px
  GHOST_HEIGHT:  50,        // px
  HUD_HEIGHT:    40,        // px
  CANVAS_WIDTH:  480,       // px
  CANVAS_HEIGHT: 640,       // px
  LS_KEY: 'flappyKiroHigh', // localStorage key
  CLOUD_COUNT:   5,         // number of decorative clouds
};
```

### GameState object

```js
let state = {
  mode:      'idle',   // 'idle' | 'playing' | 'game_over'
  score:     0,
  highScore: 0,        // loaded from localStorage on init

  ghost: {
    x:  80,            // fixed horizontal position
    y:  260,           // vertical position (px from top)
    vy: 0,             // vertical velocity (positive = downward)
  },

  pipes:  [],          // array of Pipe objects (see below)
  pipeX:  CONFIG.CANVAS_WIDTH, // x position where next pipe spawns

  clouds: [],          // array of Cloud objects (static, set on init)
};
```

### Pipe object

```js
{
  x:       Number,  // left edge of the pipe pair
  scored:  Boolean, // true once this pipe's gap has been counted
}
// Derived geometry:
//   top pipe:    rect(x, 0, PIPE_WIDTH, GAP_Y)
//   bottom pipe: rect(x, GAP_Y + GAP_HEIGHT, PIPE_WIDTH,
//                     CANVAS_HEIGHT - GAP_Y - GAP_HEIGHT - HUD_HEIGHT)
```

### Cloud object

```js
{
  x:      Number,  // center x
  y:      Number,  // center y
  rx:     Number,  // horizontal radius
  ry:     Number,  // vertical radius
}
```

---

## Game Loop

The loop is driven by `requestAnimationFrame`:

```js
function loop(timestamp) {
  if (state.mode === 'playing') {
    update();
  }
  render();
  requestAnimationFrame(loop);
}
```

### `update()` — called only in `'playing'` state

1. **PhysicsSystem** — apply gravity, clamp velocity, update ghost position.
2. **PipeSystem** — advance pipeX tracker, spawn new pipes, scroll existing pipes, cull off-screen pipes.
3. **CollisionSystem** — test ghost AABB against each pipe AABB and boundaries; trigger game over if hit.
4. **ScoreSystem** — for each pipe not yet scored where ghost.x > pipe.x + PIPE_WIDTH, increment score and mark pipe as scored.

### `render()` — called every frame regardless of state

1. Clear canvas.
2. Draw background (light blue fill, optional hand-drawn border lines).
3. Draw clouds (white ellipses/bezier shapes).
4. Draw pipes (green filled rects, dark outline for hand-drawn effect).
5. Draw ghost (ghosty.png image centered on ghost position).
6. Draw HUD (dark bottom bar, "Score: X | High: X" text).
7. If `'idle'`: draw start prompt overlay.
8. If `'game_over'`: draw game over overlay (score, high score, restart button).

---

## Interfaces

### Input Handler

```js
// Attach once to window / canvas
canvas.addEventListener('click',    handleInput);
window.addEventListener('keydown',  e => { if (e.code === 'Space') handleInput(); });

function handleInput() {
  if (state.mode === 'idle' || state.mode === 'game_over') {
    // Restart button in game_over is handled separately via its own click listener
    if (state.mode === 'idle') startGame();
  } else if (state.mode === 'playing') {
    flap();
  }
}
```

The game over overlay's **Restart** button is a DOM `<button>` element positioned absolutely over the canvas. Its `click` handler calls `startGame()` directly so the restart path is unambiguous and keyboard-accessible.

### Audio System

```js
const SFX = {
  jump:     new Audio('assets/jump.wav'),
  gameOver: new Audio('assets/game_over.wav'),
};
// Graceful failure: if load fails, play() rejects silently
function playSound(key) {
  try {
    SFX[key].currentTime = 0;
    SFX[key].play().catch(() => {});
  } catch (_) {}
}
```

### Score Persistence

```js
function loadHighScore() {
  const stored = localStorage.getItem(CONFIG.LS_KEY);
  state.highScore = stored ? parseInt(stored, 10) : 0;
}
function saveHighScore(score) {
  localStorage.setItem(CONFIG.LS_KEY, String(score));
}
```

---

## Physics Formulas

Per-frame update (executed once per `requestAnimationFrame`):

```
ghost.vy = min(ghost.vy + GRAVITY, MAX_FALL_SPEED)
ghost.y  = ghost.y + ghost.vy
```

Flap event:

```
ghost.vy = FLAP_VELOCITY   // overwrite velocity (not additive)
```

---

## Collision Detection

Axis-aligned bounding box (AABB) intersects helper:

```js
function rectsOverlap(a, b) {
  return a.x < b.x + b.w &&
         a.x + a.w > b.x &&
         a.y < b.y + b.h &&
         a.y + a.h > b.y;
}
```

Ghost bounding rect (with 4 px inset on each side for forgiveness):

```js
{ x: ghost.x + 4, y: ghost.y + 4, w: GHOST_WIDTH - 8, h: GHOST_HEIGHT - 8 }
```

Boundary checks:

```
ghost.y < 0                                  → game over (top)
ghost.y + GHOST_HEIGHT > CANVAS_HEIGHT - HUD_HEIGHT → game over (bottom)
```

---

## Rendering Layers (draw order)

| Layer | Content | Notes |
|---|---|---|
| 0 | Background | Light-blue `fillRect` + optional cross-hatch lines |
| 1 | Clouds | White ellipses, static positions |
| 2 | Pipes | Green `fillRect` pairs, dark stroke for hand-drawn look |
| 3 | Ghost | `drawImage(ghostImg, ...)` |
| 4 | HUD | Dark `fillRect` bar at bottom + white text |
| 5 | Overlays | Start prompt (idle) or Game Over overlay (game_over) |

---

## DOM Structure

```html
<body>
  <canvas id="gameCanvas" width="480" height="640"></canvas>
  <button id="restartBtn" style="display:none; position:absolute; ...">Restart</button>
</body>
```

The canvas fills the viewport using CSS. The restart button is a real DOM element for accessibility; it is shown/hidden via `display: block / none`.

---

## Error Handling

| Scenario | Behavior |
|---|---|
| `ghosty.png` fails to load | Ghost rendered as a fallback white rectangle |
| Audio asset fails to load | `play()` promise rejection caught silently; gameplay continues |
| `localStorage` unavailable | `try/catch` around all storage calls; high score treated as 0 |

---

## File Structure

```
index.html          ← single deliverable (all HTML + CSS + JS inline)
assets/
  ghosty.png
  jump.wav
  game_over.wav
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Gravity accumulation is linear and bounded

*For any* number of frames N elapsed in the playing state without a flap, the ghost's vertical velocity after N frames must equal `min(initialVy + N × GRAVITY, MAX_FALL_SPEED)`.

**Validates: Requirements 3.1, 3.3**

---

### Property 2: Position integrates velocity correctly

*For any* sequence of frames in the playing state, the ghost's vertical position after each frame must equal its previous position plus its velocity for that frame (after gravity is applied), never deviating from the cumulative sum of velocities.

**Validates: Requirements 3.2**

---

### Property 3: Pipe spacing invariant

*For any* number of pipe spawn events during a playing session, every consecutive pair of pipes must be separated by exactly `PIPE_INTERVAL` pixels measured from pipe leading edge to pipe leading edge.

**Validates: Requirements 4.1**

---

### Property 4: Pipe movement is constant and pipes are culled

*For any* active pipe, after N frames its x position must equal `initialX − N × PIPE_SPEED`, and no pipe with `x + PIPE_WIDTH < 0` must remain in the active pipe set.

**Validates: Requirements 4.2, 4.5**

---

### Property 5: Gap geometry is consistent across all pipes

*For any* pipe in the active set, the top pipe height must equal `GAP_Y`, the bottom pipe must start at `GAP_Y + GAP_HEIGHT`, and their combined heights plus the gap must equal the playable canvas height. Every pipe must have the same `GAP_Y` value.

**Validates: Requirements 4.3, 4.4**

---

### Property 6: Score increments exactly once per passed pipe

*For any* pipe that the ghost passes through without collision, the score must increase by exactly 1, and the pipe must be marked as scored so subsequent frames do not increment the score again for the same pipe.

**Validates: Requirements 5.1**

---

### Property 7: High score is the running maximum

*For any* final session score and any previously stored high score value, the value written to `localStorage` after game over must equal `max(finalScore, storedHighScore)`.

**Validates: Requirements 5.3, 5.4, 10.3**

---

### Property 8: Collision triggers game over for all overlapping positions

*For any* ghost bounding rectangle and pipe bounding rectangle that satisfy the AABB overlap condition, or for any ghost position outside the vertical canvas boundaries, the game state must transition to `'game_over'` on the next update tick.

**Validates: Requirements 6.1, 6.2, 6.3**

---

### Property 9: Game over freezes all positions

*For any* game over state, advancing any number of frames must leave the ghost position and all pipe positions unchanged.

**Validates: Requirements 6.4**

---

### Property 10: Restart produces a clean playing state

*For any* game over state (arbitrary score, arbitrary pipe positions, arbitrary ghost position), invoking restart must produce: `score = 0`, `pipes = []`, ghost at starting position, `state.mode = 'playing'`, and the high score must remain equal to `max(allPreviousScores)`.

**Validates: Requirements 7.1, 7.2**

---

### Property 11: HUD text always reflects current values

*For any* (score, highScore) pair at any point during the game loop, the HUD text rendered on the canvas must equal `"Score: {score} | High: {highScore}"`.

**Validates: Requirements 8.2**

---

### Property 12: High score is loaded correctly from localStorage

*For any* value stored in `localStorage` under `CONFIG.LS_KEY` (including absent/null, which maps to 0), `state.highScore` after initialization must equal that value.

**Validates: Requirements 10.1, 10.2**
