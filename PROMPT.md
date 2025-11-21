# Conway's Arcade - Game Creation Prompt

## 🎯 Task

Create a **[GAME_TYPE]** game using Conway's Game of Life (B3/S23) for visual aesthetics.
Modify the existing Dino Runner codebase to create your variant.

---

## 📦 Repository Reference

**GitHub Repository:** `https://github.com/brunobarran/conways-arcade`

**Live Demo:** `https://brunobarran.github.io/conways-arcade/`

---

## 📁 Files to Reference

All files are in the repository root. Visit the GitHub URLs below to read the source code.

### Core Files (READ THESE FIRST)

**1. game.js** - **MODIFY THIS FILE** to create your game
- URL: `https://github.com/brunobarran/conways-arcade/blob/main/game.js`
- Contains: Game configuration, entity setup, update loop, collision detection
- Lines to focus on:
  - `CONFIG` object (lines ~30-120) - Game parameters
  - `setupPlayer()` - Player initialization
  - `setupObstacles()` / `spawnObstacle()` - Obstacle creation
  - `updateGame()` - Main game loop
  - `checkCollisions()` - Collision logic

**2. index.html** - Game UI/layout (modify if UI changes needed)
- URL: `https://github.com/brunobarran/conways-arcade/blob/main/index.html`
- Contains: HTML structure, CSS styles, UI overlays

### Module Library (lib/ - DO NOT MODIFY)

Import these modules in your modified `game.js`:

| Module | Description | GitHub URL |
|--------|-------------|------------|
| `GoLEngine.js` | Conway's GoL B3/S23 engine | `https://github.com/brunobarran/conways-arcade/blob/main/lib/GoLEngine.js` |
| `SimpleGradientRenderer.js` | Animated gradients | `https://github.com/brunobarran/conways-arcade/blob/main/lib/SimpleGradientRenderer.js` |
| `GradientPresets.js` | Color presets | `https://github.com/brunobarran/conways-arcade/blob/main/lib/GradientPresets.js` |
| `Collision.js` | Hitbox detection | `https://github.com/brunobarran/conways-arcade/blob/main/lib/Collision.js` |
| `Patterns.js` | GoL patterns | `https://github.com/brunobarran/conways-arcade/blob/main/lib/Patterns.js` |
| `GoLHelpers.js` | Helper functions | `https://github.com/brunobarran/conways-arcade/blob/main/lib/GoLHelpers.js` |
| `ParticleHelpers.js` | Particle systems | `https://github.com/brunobarran/conways-arcade/blob/main/lib/ParticleHelpers.js` |
| `PatternRenderer.js` | Pattern rendering | `https://github.com/brunobarran/conways-arcade/blob/main/lib/PatternRenderer.js` |
| `GameBaseConfig.js` | Canvas config | `https://github.com/brunobarran/conways-arcade/blob/main/lib/GameBaseConfig.js` |
| `UIHelpers.js` | UI rendering | `https://github.com/brunobarran/conways-arcade/blob/main/lib/UIHelpers.js` |
| `HitboxDebug.js` | Debug tools | `https://github.com/brunobarran/conways-arcade/blob/main/lib/HitboxDebug.js` |

---

## 🎮 How to Modify game.js

### Step 1: Understand Current Structure

Read `game.js` from the repository. Key sections:

```javascript
// CONFIGURATION
const CONFIG = createGameConfig({
  gravity: 5.5,
  jumpForce: -70,
  obstacle: { speed: -15, ... }
})

// SETUP
function setupPlayer() { ... }
function setupObstacles() { ... }

// UPDATE LOOP
function updateGame() {
  updatePlayer()
  updateObstacles()
  checkCollisions()
}

// COLLISION
function checkCollisions() { ... }
```

### Step 2: Modify for Your Game Type

**Example: Flappy Bird**

```javascript
// Change CONFIG
const CONFIG = createGameConfig({
  gravity: 0.8,        // Constant gravity
  jumpForce: -12,      // Tap to jump
  pipeGap: 300,        // Gap between pipes
  pipeSpeed: -6        // Horizontal speed
})

// Modify setupObstacles (vertical pipes)
function spawnObstacle() {
  const gapY = random(200, CONFIG.height - 400)

  // Top pipe
  obstacles.push({
    x: CONFIG.width,
    y: 0,
    width: 120,
    height: gapY,
    type: 'pipe-top',
    gol: new GoLEngine(4, Math.floor(gapY / 30), 15),
    pattern: PatternName.BEEHIVE
  })

  // Bottom pipe
  obstacles.push({
    x: CONFIG.width,
    y: gapY + CONFIG.pipeGap,
    width: 120,
    height: CONFIG.height - gapY - CONFIG.pipeGap,
    type: 'pipe-bottom',
    gol: new GoLEngine(4, Math.floor((CONFIG.height - gapY - CONFIG.pipeGap) / 30), 15),
    pattern: PatternName.LOAF
  })
}

// Modify updatePlayer (apply gravity constantly)
function updatePlayer() {
  player.vy += CONFIG.gravity   // Constant gravity
  player.y += player.vy

  // Clamp to screen
  player.y = constrain(player.y, 0, CONFIG.height - player.height)

  // Update GoL
  player.gol.updateThrottled(state.frameCount)
  applyLifeForce(player)
}

// Modify touch/key input (jump on tap)
function touchStarted() {
  if (state.phase === 'PLAYING') {
    player.vy = CONFIG.jumpForce
  }
  return false
}
```

**Example: Space Invaders**

```javascript
// Change CONFIG
const CONFIG = createGameConfig({
  invaderRows: 3,
  invaderCols: 5,
  invaderSpeed: 2,
  playerSpeed: 10,
  bulletSpeed: -8
})

// Setup grid of invaders
function setupInvaders() {
  invaders = []
  for (let row = 0; row < CONFIG.invaderRows; row++) {
    for (let col = 0; col < CONFIG.invaderCols; col++) {
      invaders.push({
        x: col * 150 + 200,
        y: row * 120 + 100,
        width: 120,
        height: 120,
        gol: new GoLEngine(4, 4, 15),
        gradient: GRADIENT_PRESETS.ENEMY_HOT,
        alive: true
      })
      seedRadialDensity(invaders[invaders.length - 1].gol, 0.8, 0.0)
    }
  }
}

// Player shoots bullets
function shootBullet() {
  if (state.shootCooldown <= 0) {
    bullets.push({
      x: player.x + player.width / 2,
      y: player.y,
      width: 30,
      height: 60,
      vy: CONFIG.bulletSpeed,
      gol: new GoLEngine(1, 2, 30),
      gradient: GRADIENT_PRESETS.BULLET
    })
    state.shootCooldown = 15  // 0.25s cooldown
  }
}
```

---

## 🧬 Conway's Game of Life Rules (B3/S23)

**CRITICAL:** Understand these rules before using GoL patterns.

### The Rules

- **Birth (B3):** Dead cell with exactly 3 live neighbors becomes alive
- **Survival (S2/S3):** Live cell with 2-3 neighbors survives
- **Death:** Live cell with <2 neighbors (underpopulation) or >3 (overpopulation) dies

### Authenticity Guidelines

Use GoL where it enhances visuals without breaking gameplay:

| Entity Type | GoL Strategy | Example Code |
|-------------|--------------|--------------|
| **Player** | Modified GoL + `applyLifeForce()` | Stable core, organic edges |
| **Enemies** | Modified GoL or static patterns | BLINKER, PULSAR, LWSS |
| **Projectiles** | Static patterns (Visual Only) | Single cell or BLOCK |
| **Explosions** | Pure GoL + `seedRadialDensity()` | Emergent behavior |
| **Decorations** | Pure GoL patterns | Background elements |

**Key Functions:**

```javascript
// Seed GoL with radial density (natural look)
seedRadialDensity(entity.gol, 0.85, 0.0)  // 85% center, 0% edges

// Keep entity core alive (prevent death)
applyLifeForce(entity)  // Call in update loop

// Use canonical GoL patterns
entity.gol.setPattern(Patterns.BLINKER, 1, 1)
```

---

## 🎨 Visual Guidelines

### Google Brand Colors

Use `GRADIENT_PRESETS` for consistent Material Design colors:

```javascript
GRADIENT_PRESETS.PLAYER          // Blue (#4285F4)
GRADIENT_PRESETS.ENEMY_HOT       // Red-Orange (#EA4335)
GRADIENT_PRESETS.ENEMY_COLD      // Blue-Purple
GRADIENT_PRESETS.ENEMY_RAINBOW   // Multi-color
GRADIENT_PRESETS.BULLET          // Yellow (#FBBC04)
GRADIENT_PRESETS.EXPLOSION       // Red-Yellow gradient
```

### Standard Entity Sizes

Based on portrait orientation (1200×1920):

```javascript
// Main entities (player, large enemies)
{
  width: 180,
  height: 180,
  gol: new GoLEngine(6, 6, 12),  // 6×6 grid, 12fps update
  cellSize: 30
}

// Small entities (projectiles)
{
  width: 90,
  height: 90,
  gol: new GoLEngine(3, 3, 15),  // 3×3 grid, 15fps update
  cellSize: 30
}

// Explosions (fast animation)
{
  width: 180,
  height: 180,
  gol: new GoLEngine(6, 6, 30),  // 6×6 grid, 30fps update
  cellSize: 30
}
```

---

## 🚫 Common Mistakes (AVOID)

1. ❌ **Using npm/build tools**
   - This is vanilla JS + p5.js CDN only
   - No webpack, Vite, or bundlers

2. ❌ **Modifying lib/ files**
   - Copy lib/ code as-is
   - Only modify `game.js` and optionally `index.html`

3. ❌ **Using GoL cells for collision**
   - Collision uses **fixed hitboxes** (Collision.js)
   - GoL is for **visuals only**

   ```javascript
   // ✅ CORRECT
   Collision.rectRect(
     player.x, player.y, player.width, player.height,
     obstacle.x, obstacle.y, obstacle.width, obstacle.height
   )

   // ❌ WRONG
   // Checking if GoL cells overlap (unpredictable!)
   ```

4. ❌ **Desktop-only controls**
   - Must support both keyboard AND touch

   ```javascript
   // ✅ CORRECT
   function keyPressed() { ... }
   function touchStarted() { ... }

   // ❌ WRONG
   // Only keyPressed (mobile won't work)
   ```

5. ❌ **Inventing GoL patterns**
   - Use canonical patterns from `lib/Patterns.js`
   - Patterns: BLOCK, BEEHIVE, LOAF, BOAT, TUB, BLINKER, TOAD, BEACON, PULSAR, GLIDER, LWSS, etc.

6. ❌ **Static sprites for everything**
   - Use procedural GoL patterns (exception: brand logos if needed)

7. ❌ **Forgetting p5.js global mode**
   - No need for `this.` prefix
   - Functions like `setup()`, `draw()` must be global

   ```javascript
   // ✅ CORRECT
   window.setup = setup
   window.draw = draw

   // ❌ WRONG
   // Not exposing to window (p5 won't find them)
   ```

---

## ✅ Output Format

Provide the **complete modified `game.js` file**.

User will:
1. Replace `game.js` in their local copy
2. Test with `python -m http.server 8000`
3. Play at `http://localhost:8000`

**Optional:** If UI changes are needed, also provide modified `index.html`.

---

## 📚 Reference Examples

### Available GoL Patterns

From `lib/Patterns.js`:

```javascript
PatternName.BLOCK              // 2×2 still life
PatternName.BEEHIVE            // 4×3 still life
PatternName.LOAF               // 4×4 still life
PatternName.BOAT               // 3×3 still life
PatternName.TUB                // 3×3 still life
PatternName.BLINKER            // 3×1 oscillator (period 2)
PatternName.TOAD               // 4×2 oscillator (period 2)
PatternName.BEACON             // 4×4 oscillator (period 2)
PatternName.PULSAR             // 13×13 oscillator (period 3)
PatternName.GLIDER             // 3×3 spaceship (moves diagonally)
PatternName.LIGHTWEIGHT_SPACESHIP  // 5×4 spaceship (moves horizontally)
```

### Rendering Patterns

```javascript
// Static rendering (no animation)
const renderer = createPatternRenderer(
  PatternName.BLOCK,
  RenderMode.STATIC,
  0  // phase
)

// Animated oscillator
const renderer = createPatternRenderer(
  PatternName.BLINKER,
  RenderMode.LOOP,
  0  // starting phase
)

// Update and render
renderer.update(state.frameCount)
maskedRenderer.renderMaskedGrid(
  renderer.gol,
  x, y,
  30,  // cellSize
  GRADIENT_PRESETS.ENEMY_HOT
)
```

---

## 🎯 Example Game Ideas

**Easy:**
- **Pong** - Two paddles (GoL patterns), ball (single cell)
- **Catch** - Falling objects (GoL patterns), move basket to catch

**Medium:**
- **Flappy Bird** - Vertical pipes, constant gravity
- **Snake** - Growing snake body (GoL segments)
- **Breakout** - Paddle + ball + GoL brick patterns

**Advanced:**
- **Space Invaders** - Grid of enemies, shooting mechanics
- **Galaga** - Enemies with movement patterns
- **Asteroids** - Rotating ship, shooting asteroids (GoL rocks)

---

## 🚀 Quick Start Checklist

- [ ] Read `game.js` from repository
- [ ] Understand CONFIG, setup functions, update loop
- [ ] Identify what to change for your game type
- [ ] Modify CONFIG (gravity, speed, mechanics)
- [ ] Modify setup functions (entities, patterns)
- [ ] Modify update loop (game logic)
- [ ] Add input handling (keyboard + touch)
- [ ] Test collision logic
- [ ] Use GoL patterns from `lib/Patterns.js`
- [ ] Apply Google brand colors (GRADIENT_PRESETS)

---

**Ready?** Read the repository files and create the modified `game.js` for the requested game type.
