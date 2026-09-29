# Requirements Document

## Introduction

Flappy Kiro is a browser-based endless-scroller game implemented as a single HTML file using plain HTML, CSS, and JavaScript. The player guides a ghost character through gaps in pipes by clicking or pressing spacebar to flap upward, while gravity pulls the ghost down continuously. The game features a retro hand-drawn visual style, sound effects, score tracking, and persistent high score storage. The game supports three states: idle/start screen, playing, and game over.

## Glossary

- **Game**: The Flappy Kiro browser application running in a single HTML file.
- **Ghost**: The player-controlled sprite character rendered using `ghosty.png`.
- **Pipe**: A pair of green rectangular obstacles (one from the top, one from the bottom) with a fixed-height gap between them.
- **Gap**: The vertical opening between the top pipe and bottom pipe through which the Ghost must pass.
- **Canvas**: The HTML5 `<canvas>` element on which all game visuals are rendered.
- **Score**: The count of Pipe gaps the Ghost has successfully passed through in the current session.
- **High Score**: The highest Score achieved across all sessions, persisted in `localStorage`.
- **HUD**: The dark bottom bar overlaid on the Canvas displaying the current Score and High Score.
- **Idle State**: The initial state shown when the page loads, displaying a start screen prompt.
- **Playing State**: The active gameplay state where the Ghost moves and Pipes scroll.
- **Game Over State**: The state entered when the Ghost collides with a Pipe or boundary, displaying an overlay.
- **Flap**: A single upward velocity impulse applied to the Ghost in response to player input.
- **Gravity**: The constant downward acceleration applied to the Ghost during the Playing State.
- **Pipe Speed**: The constant leftward scroll velocity of all Pipes.
- **Cloud**: A decorative white shape rendered in the background layer of the Canvas.

---

## Requirements

### Requirement 1: Game Initialization and Idle State

**User Story:** As a player, I want to see a start screen when the page loads, so that I know how to begin the game.

#### Acceptance Criteria

1. WHEN the page finishes loading, THE Game SHALL display the Canvas in the Idle State showing the Ghost, background, Clouds, and a "Click or press Space to Start" prompt.
2. THE Game SHALL render the background as a light blue color with a sketchy/hand-drawn visual style.
3. THE Game SHALL render at least three Cloud shapes as static decorative white shapes in the background layer.
4. WHILE in the Idle State, THE Game SHALL keep the Ghost stationary at its starting position.
5. WHILE in the Idle State, THE Game SHALL display no Pipes on the Canvas.

---

### Requirement 2: Player Input and Flap Mechanic

**User Story:** As a player, I want to flap the ghost upward by clicking or pressing spacebar, so that I can navigate through pipe gaps.

#### Acceptance Criteria

1. WHEN the player clicks the Canvas during the Playing State, THE Game SHALL apply a Flap impulse to the Ghost, setting its vertical velocity to an upward value.
2. WHEN the player presses the Spacebar key during the Playing State, THE Game SHALL apply a Flap impulse to the Ghost, setting its vertical velocity to an upward value.
3. WHEN a Flap is triggered, THE Game SHALL play the `jump.wav` audio asset.
4. WHEN the player clicks the Canvas during the Idle State, THE Game SHALL transition the Game from the Idle State to the Playing State.
5. WHEN the player presses the Spacebar key during the Idle State, THE Game SHALL transition the Game from the Idle State to the Playing State.

---

### Requirement 3: Physics — Gravity and Ghost Movement

**User Story:** As a player, I want the ghost to fall under gravity continuously, so that the game requires active input to stay airborne.

#### Acceptance Criteria

1. WHILE in the Playing State, THE Game SHALL apply a constant downward Gravity acceleration to the Ghost on every animation frame.
2. WHILE in the Playing State, THE Game SHALL update the Ghost's vertical position each frame by adding its current vertical velocity.
3. THE Game SHALL cap the Ghost's maximum downward velocity to prevent it from falling faster than the visible Canvas height per second.

---

### Requirement 4: Pipe Scrolling and Spawning

**User Story:** As a player, I want pipes to scroll toward me endlessly, so that the game presents a continuous challenge.

#### Acceptance Criteria

1. WHILE in the Playing State, THE Game SHALL spawn a new Pipe at the right edge of the Canvas at a fixed horizontal interval of 200 pixels.
2. WHILE in the Playing State, THE Game SHALL move all active Pipes leftward at the constant Pipe Speed on every animation frame.
3. THE Game SHALL render each Pipe as a green rectangle extending from the top of the Canvas and a green rectangle extending from the bottom of the Canvas, with a Gap of fixed vertical height between them.
4. THE Game SHALL position the Gap at a fixed vertical position on every Pipe.
5. WHEN a Pipe moves entirely off the left edge of the Canvas, THE Game SHALL remove that Pipe from the active Pipe set.

---

### Requirement 5: Scoring

**User Story:** As a player, I want to earn a point each time I pass through a pipe gap, so that I have a clear measure of my progress.

#### Acceptance Criteria

1. WHEN the Ghost's horizontal position passes the right edge of a Pipe's Gap without collision, THE Game SHALL increment the Score by one.
2. THE Game SHALL display the current Score in the HUD throughout the Playing State.
3. WHEN the Game transitions from the Playing State to the Game Over State, THE Game SHALL compare the final Score to the stored High Score.
4. IF the final Score exceeds the stored High Score, THEN THE Game SHALL update the High Score in `localStorage` with the final Score value.
5. THE Game SHALL display the High Score in the HUD throughout the Playing State and Game Over State.

---

### Requirement 6: Collision Detection and Game Over

**User Story:** As a player, I want the game to end immediately when I hit a pipe or boundary, so that collisions have clear consequences.

#### Acceptance Criteria

1. WHEN the Ghost's bounding rectangle intersects the bounding rectangle of any Pipe, THE Game SHALL transition from the Playing State to the Game Over State immediately.
2. WHEN the Ghost's vertical position causes its bounding rectangle to intersect the bottom boundary of the Canvas, THE Game SHALL transition from the Playing State to the Game Over State immediately.
3. WHEN the Ghost's vertical position causes its bounding rectangle to intersect the top boundary of the Canvas, THE Game SHALL transition from the Playing State to the Game Over State immediately.
4. WHEN the Game transitions to the Game Over State, THE Game SHALL freeze the Ghost and all Pipes at their current positions.
5. WHEN the Game transitions to the Game Over State, THE Game SHALL play the `game_over.wav` audio asset.
6. WHEN the Game transitions to the Game Over State, THE Game SHALL display a Game Over overlay containing the final Score, the High Score, and a "Restart" button.

---

### Requirement 7: Restart

**User Story:** As a player, I want to restart the game from the game over screen, so that I can try to beat my high score without reloading the page.

#### Acceptance Criteria

1. WHEN the player clicks the "Restart" button in the Game Over overlay, THE Game SHALL reset the Score to zero, remove all active Pipes, return the Ghost to its starting position, and transition to the Playing State.
2. WHEN the Game restarts, THE Game SHALL retain the current High Score value from `localStorage`.

---

### Requirement 8: HUD Display

**User Story:** As a player, I want to always see my current score and high score on screen, so that I know how I am performing.

#### Acceptance Criteria

1. THE Game SHALL render the HUD as a dark-colored bar anchored to the bottom of the Canvas.
2. THE Game SHALL display the text "Score: X | High: X" in the HUD, where each X is replaced with the corresponding numeric value.
3. WHILE in the Idle State, THE Game SHALL display a Score of zero and the stored High Score in the HUD.

---

### Requirement 9: Visual Rendering

**User Story:** As a player, I want the game to look like a retro hand-drawn style, so that it feels cohesive and charming.

#### Acceptance Criteria

1. THE Game SHALL render the Ghost using the `ghosty.png` image asset loaded from the `/assets/` directory.
2. THE Game SHALL render Pipes in green using filled rectangles with a hand-drawn border effect.
3. THE Game SHALL render Clouds as white rounded shapes in the background layer behind the Ghost and Pipes.
4. THE Game SHALL render all game elements on a single HTML5 Canvas element sized to fill the game viewport.

---

### Requirement 10: Persistence

**User Story:** As a player, I want my high score to persist between browser sessions, so that my best result is always remembered.

#### Acceptance Criteria

1. WHEN the page loads, THE Game SHALL read the stored High Score from `localStorage` under a defined key.
2. IF no High Score value exists in `localStorage`, THEN THE Game SHALL treat the High Score as zero.
3. WHEN the High Score is updated, THE Game SHALL write the new High Score value to `localStorage` under the same defined key immediately.

---

### Requirement 11: Audio

**User Story:** As a player, I want to hear sound effects for flapping and game over, so that the game feels responsive and engaging.

#### Acceptance Criteria

1. WHEN a Flap is triggered during the Playing State, THE Game SHALL play the `jump.wav` audio asset from the `/assets/` directory.
2. WHEN the Game transitions to the Game Over State, THE Game SHALL play the `game_over.wav` audio asset from the `/assets/` directory.
3. IF an audio asset fails to load, THEN THE Game SHALL continue gameplay without audio and SHALL NOT display an error to the player.

---

### Requirement 12: Single-File Delivery

**User Story:** As a developer, I want the game delivered as a single HTML file, so that it is easy to open and distribute.

#### Acceptance Criteria

1. THE Game SHALL be implemented within a single `index.html` file containing all HTML structure, CSS styles, and JavaScript logic inline.
2. THE Game SHALL reference only the existing assets in the `/assets/` directory (`ghosty.png`, `jump.wav`, `game_over.wav`) and SHALL NOT require any external libraries, CDN resources, or build tools.
