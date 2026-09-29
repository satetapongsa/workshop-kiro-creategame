# Implementation Plan: Flappy Kiro

## Overview

Implement Flappy Kiro as a single `index.html` file using plain HTML5, CSS, and JavaScript. The game loop is driven by `requestAnimationFrame`, rendering to a `<canvas>` element. Work proceeds in layers: scaffold and constants first, then physics and input, then pipes and scoring, then collision and game-over, then visual polish and audio, finishing with integration and wiring.

---

## Tasks

- [x] 1. Scaffold the HTML file and define all constants
  - Create `index.html` with the `<canvas id="gameCanvas">` element and an absolutely-positioned `<button id="restartBtn">` (initially hidden)
  - Add inline `<style>` that centers the canvas in the viewport and styles the restart button
  - Define the `CONFIG` object with all compile-time constants (`GRAVITY`, `FLAP_VELOCITY`, `MAX_FALL_SPEED`, `PIPE_SPEED`, `PIPE_INTERVAL`, `PIPE_WIDTH`, `GAP_HEIGHT`, `GAP_Y`, `GHOST_WIDTH`, `GHOST_HEIGHT`, `HUD_HEIGHT`, `CANVAS_WIDTH`, `CANVAS_HEIGHT`, `LS_KEY`, `CLOUD_COUNT`)
  - _Requirements: 9.4, 12.1, 12.2_

- [x] 2. Implement asset loading and the initial game state
  - [x] 2.1 Load `ghosty.png` image asset with an `onerror` fallback flag, and load `jump.wav` and `game_over.wav` into the `SFX` object
    - Implement `playSound(key)` with `try/catch` and `.catch(() => {})` on the play promise
    - _Requirements: 9.1, 11.1, 11.2, 11.3_

  - [x] 2.2 Implement `loadHighScore()` and `saveHighScore(score)` using `localStorage` with a `try/catch` guard
    - _Requirements: 10.1, 10.2, 10.3_

  - [x] 2.3 Initialize the `state` object (`mode`, `score`, `highScore`, `ghost`, `pipes`, `pipeX`, `clouds`) and the `initClouds()` helper that populates static cloud positions
    - _Requirements: 1.1, 1.3, 1.4, 1.5_

  - [x]* 2.4 Write property test for high score persistence (Property 12)
    - **Property 12: High score is loaded correctly from localStorage**
    - **Validates: Requirements 10.1, 10.2**

- [x] 3. Implement the physics and flap system
  - [x] 3.1 Write `applyPhysics()` that applies `GRAVITY` to `ghost.vy`, clamps to `MAX_FALL_SPEED`, and integrates `ghost.vy` into `ghost.y`
    - _Requirements: 3.1, 3.2, 3.3_

  - [x]* 3.2 Write property test for gravity accumulation (Property 1)
    - **Property 1: Gravity accumulation is linear and bounded**
    - **Validates: Requirements 3.1, 3.3**

  - [x]* 3.3 Write property test for position integration (Property 2)
    - **Property 2: Position integrates velocity correctly**
    - **Validates: Requirements 3.2**

  - [x] 3.4 Write `flap()` that sets `ghost.vy = CONFIG.FLAP_VELOCITY` and calls `playSound('jump')`
    - _Requirements: 2.1, 2.2, 2.3_

- [x] 4. Implement the pipe system
  - [x] 4.1 Write `spawnPipe()` that pushes a new pipe object `{ x: CANVAS_WIDTH, scored: false }` onto `state.pipes` and advances `state.pipeX` by `PIPE_INTERVAL`
    - _Requirements: 4.1, 4.3, 4.4_

  - [x] 4.2 Write `updatePipes()` that decrements each pipe's `x` by `PIPE_SPEED`, spawns when `state.pipeX` drops to the canvas edge, and culls pipes where `x + PIPE_WIDTH < 0`
    - _Requirements: 4.1, 4.2, 4.5_

  - [x]* 4.3 Write property test for pipe spacing invariant (Property 3)
    - **Property 3: Pipe spacing invariant**
    - **Validates: Requirements 4.1**

  - [x]* 4.4 Write property test for pipe movement and culling (Property 4)
    - **Property 4: Pipe movement is constant and pipes are culled**
    - **Validates: Requirements 4.2, 4.5**

  - [x]* 4.5 Write property test for gap geometry consistency (Property 5)
    - **Property 5: Gap geometry is consistent across all pipes**
    - **Validates: Requirements 4.3, 4.4**

- [x] 5. Implement scoring
  - [x] 5.1 Write `updateScore()` that iterates `state.pipes`, marks pipes as `scored` when the ghost's x passes the pipe's right edge, and increments `state.score`
    - _Requirements: 5.1, 5.2, 5.5_

  - [x]* 5.2 Write property test for score increments exactly once per pipe (Property 6)
    - **Property 6: Score increments exactly once per passed pipe**
    - **Validates: Requirements 5.1**

- [x] 6. Implement collision detection and game over transition
  - [x] 6.1 Write `rectsOverlap(a, b)` AABB helper and `getGhostRect()` that returns the ghost bounding box with a 4 px inset
    - _Requirements: 6.1, 6.2, 6.3_

  - [x] 6.2 Write `checkCollisions()` that tests the ghost rect against each pipe's top and bottom rects and against the top/bottom canvas boundaries; on any hit, calls `triggerGameOver()`
    - _Requirements: 6.1, 6.2, 6.3_

  - [x] 6.3 Write `triggerGameOver()` that sets `state.mode = 'game_over'`, calls `playSound('gameOver')`, compares and saves the high score via `saveHighScore`, and shows the restart button
    - _Requirements: 6.4, 6.5, 6.6, 5.3, 5.4_

  - [x]* 6.4 Write property test for collision triggering game over (Property 8)
    - **Property 8: Collision triggers game over for all overlapping positions**
    - **Validates: Requirements 6.1, 6.2, 6.3**

  - [x]* 6.5 Write property test for game over freezing positions (Property 9)
    - **Property 9: Game over freezes all positions**
    - **Validates: Requirements 6.4**

  - [x]* 6.6 Write property test for high score is running maximum (Property 7)
    - **Property 7: High score is the running maximum**
    - **Validates: Requirements 5.3, 5.4, 10.3**

- [x] 7. Checkpoint — Ensure core game loop logic is correct
  - Ensure all tests pass, ask the user if questions arise.

- [x] 8. Implement the game loop and state transitions
  - [x] 8.1 Write `startGame()` that resets `state` (score, ghost position, pipes array, pipeX), sets `state.mode = 'playing'`, and hides the restart button
    - _Requirements: 7.1, 7.2_

  - [x]* 8.2 Write property test for restart producing a clean state (Property 10)
    - **Property 10: Restart produces a clean playing state**
    - **Validates: Requirements 7.1, 7.2**

  - [x] 8.3 Implement the main `update()` function that chains `applyPhysics()`, `updatePipes()`, `updateScore()`, and `checkCollisions()` — only runs when `state.mode === 'playing'`
    - _Requirements: 3.1, 3.2, 4.2, 5.1, 6.1_

  - [x] 8.4 Implement the `loop(timestamp)` function using `requestAnimationFrame`, calling `update()` when playing and `render()` every frame; kick off the loop with the initial `requestAnimationFrame(loop)` call
    - _Requirements: 1.1_

- [x] 9. Implement the renderer
  - [x] 9.1 Write `drawBackground()` that fills the canvas with light blue and adds a few light cross-hatch or wavy sketch lines for the hand-drawn feel
    - _Requirements: 1.2, 9.4_

  - [x] 9.2 Write `drawClouds()` that renders each cloud object as overlapping white ellipses using `ctx.ellipse` or bezier curves
    - _Requirements: 1.3, 9.3_

  - [x] 9.3 Write `drawPipes()` that renders top and bottom green filled rects for each pipe with a dark stroke for the sketchy border effect
    - _Requirements: 4.3, 9.2_

  - [x] 9.4 Write `drawGhost()` that draws `ghostImg` centered on the ghost position; falls back to a white filled rect if the image failed to load
    - _Requirements: 9.1_

  - [x] 9.5 Write `drawHUD()` that fills a dark rect at the bottom of the canvas and renders `"Score: X | High: X"` in white text
    - _Requirements: 8.1, 8.2, 8.3_

  - [x]* 9.6 Write property test for HUD text reflects current values (Property 11)
    - **Property 11: HUD text always reflects current values**
    - **Validates: Requirements 8.2**

  - [x] 9.7 Write `drawIdleOverlay()` that renders a semi-transparent overlay with "Click or press Space to Start" prompt over the canvas
    - _Requirements: 1.1_

  - [x] 9.8 Write `drawGameOverOverlay()` that renders a semi-transparent overlay with the final score, high score, and the visible restart button positioned over the canvas
    - _Requirements: 6.6_

  - [x] 9.9 Write the main `render()` function that clears the canvas, then calls the layer draws in order: background → clouds → pipes → ghost → HUD → overlay (if idle or game_over)
    - _Requirements: 9.4_

- [ ] 10. Implement input handling and wire everything together
  - [ ] 10.1 Attach `canvas.addEventListener('click', handleInput)` and `window.addEventListener('keydown', ...)` for spacebar, implementing `handleInput()` to call `startGame()` from idle or `flap()` during play
    - _Requirements: 2.1, 2.2, 2.4, 2.5_

  - [ ] 10.2 Attach the `restartBtn` click listener to call `startGame()`; confirm the button is positioned absolutely over the canvas center and styled to match the retro theme
    - _Requirements: 7.1_

  - [ ] 10.3 Call `loadHighScore()` and `initClouds()` on page load, then start the `requestAnimationFrame` loop in `'idle'` mode — verifying the full pipeline works end-to-end
    - _Requirements: 1.1, 10.1_

- [ ] 11. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- The design has no external test framework — property tests can be implemented as inline JavaScript assertions in a separate `<script>` block or a companion `tests.js` file, executed in the browser console
- All constants live in `CONFIG`; no magic numbers should appear outside it
- Audio failures are silent — no `console.error` or UI indication
- The restart button is a real DOM `<button>` for keyboard accessibility
- Property tests validate universal correctness properties; unit tests validate specific examples and edge cases

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "2.2", "2.3"] },
    { "id": 2, "tasks": ["2.4", "3.1", "4.1", "9.1", "9.2", "9.3", "9.4", "9.5"] },
    { "id": 3, "tasks": ["3.2", "3.3", "3.4", "4.2", "5.1", "6.1", "9.6", "9.7", "9.8", "9.9"] },
    { "id": 4, "tasks": ["4.3", "4.4", "4.5", "5.2", "6.2", "8.1", "8.3"] },
    { "id": 5, "tasks": ["6.3", "6.4", "6.5", "6.6", "8.2", "8.4"] },
    { "id": 6, "tasks": ["10.1", "10.2"] },
    { "id": 7, "tasks": ["10.3"] }
  ]
}
```
