# Code Flow Documentation

This document explains the execution flow of Foxenstein3D (Ultra Nightmare), a Wolfenstein3D-style first-person shooter built with libGDX.

## Table of Contents
- [Application Startup](#application-startup)
- [Game Loop](#game-loop)
- [Screen Flow](#screen-flow)
- [Entity Lifecycle](#entity-lifecycle)
- [Map Loading Flow](#map-loading-flow)
- [Player Input Flow](#player-input-flow)
- [Combat Flow](#combat-flow)

---

## Application Startup

### 1. Platform Launcher
The game starts from a platform-specific launcher:
- **Desktop**: `DesktopLauncher.java`
- **HTML**: `HtmlLauncher.java`

### 2. Main Game Initialization (`Foxenstein3D.create()`)

The initialization happens in this order:

```
Foxenstein3D.create()
├── 1. Create rendering resources
│   ├── SpriteBatch (for 2D/UI rendering)
│   ├── ModelBatch (for 3D rendering)
│   └── FrameBuffer (for post-processing effects)
│
├── 2. Initialize managers
│   ├── AssetsManager (loads all game assets)
│   ├── OverlapFilterManager (manages collision filters)
│   ├── ModelMaker (builds 3D models)
│   ├── EntityManager (manages all game entities)
│   ├── RectManager (manages collision rectangles)
│   └── MapBuilder (builds levels from Tiled maps)
│
├── 3. Setup input
│   └── GameInputProcessor (handles keyboard/mouse input)
│
└── 4. Set initial screen
    └── MainMenuScreen (game starts here)
```

**Reference**: `Foxenstein3D.java:49-71`

---

## Game Loop

The game loop follows libGDX's standard render cycle:

### Main Render Loop (`Foxenstein3D.render()`)

```
Every Frame:
├── 1. Update time tracker
│   └── timeSinceLaunch += deltaTime
│
├── 2. Clear screen
│   └── glClearColor(red) and glClear()
│
├── 3. Render current screen
│   └── getScreen().render(delta)
│       ├── handleInput(delta)
│       ├── tick(delta)
│       │   └── EntityManager.tickAllEntities(delta)
│       │       └── For each entity: entity.tick(delta)
│       │
│       └── render(delta)
│           ├── 2D rendering (SpriteBatch)
│           ├── 3D rendering (ModelBatch)
│           └── UI rendering (SpriteBatch)
│
└── 4. Reset input state
    └── gameInput.resetScrolled()
```

**Reference**: `Foxenstein3D.java:146-155`

---

## Screen Flow

The game uses libGDX's Screen system for different game states:

### Screen Hierarchy

```
GameScreen (base class)
├── Methods:
│   ├── handleInput(delta)  - Process user input
│   ├── tick(delta)          - Update game logic
│   ├── render(delta)        - Draw everything
│   └── checkOverlaps()      - Collision detection
│
├── MainMenuScreen
│   └── Entry point, shows menu options
│
└── PlayScreen (main gameplay)
    ├── Extends GameScreen
    ├── Manages player, enemies, level
    └── Handles game-over and pause menus
```

### Screen Transition Example

```
MainMenuScreen
    └── User selects "Start Game"
        └── game.setScreen(new PlayScreen(game))
            ├── PlayScreen constructor runs
            ├── Loads map via MapBuilder
            ├── Creates player entity
            └── Starts background music
```

**Reference**:
- `GameScreen.java:15-162`
- `PlayScreen.java:50-94`

---

## Entity Lifecycle

All game objects (player, enemies, walls, items) are entities:

### Entity Creation Flow

```
1. Create Entity
   └── new Entity(screen)
       └── Assigned unique ID by EntityManager

2. Add to EntityManager
   └── EntityManager.addEntity(entity)
       └── Added to entities array

3. Entity Update Loop (every frame if not paused)
   └── EntityManager.tickAllEntities(delta)
       └── For each entity:
           ├── if (shouldTick()) entity.tick(delta)
           ├── if (shouldRender2D()) entity.render2D(delta)
           └── if (shouldRender3D()) entity.render3D(mdlBatch, env, delta)

4. Entity Destruction
   └── entity.setDestroy(true)
       └── entity.destroy()
           ├── Remove from RectManager (if has collision)
           └── EntityManager.removeEntity(id)
```

### Entity Types

- **Cell3D**: Floor/ceiling/wall tiles
- **Player**: Player character with camera
- **Enemy**: Base class for enemies (Skull, Eye, Fireball)
- **Door**: Interactive doors with keycards
- **Key**: Collectible keycard items
- **Grid**: Debug grid overlay

**Reference**:
- `Entity.java:9-87`
- `EntityManager.java:10-84`

---

## Map Loading Flow

Maps are created using Tiled Map Editor and loaded at runtime:

### MapBuilder.buildMap() Process

```
MapBuilder.buildMap(TiledMap)
├── 1. Read map properties
│   ├── Width and height
│   └── Map layers
│
├── 2. Build floor and ceiling cells
│   ├── Read "floor" layer
│   ├── Read "ceiling" layer
│   ├── Create Cell3D for each tile
│   └── Assign textures based on tile ID
│
├── 3. Determine walls
│   ├── Compare adjacent cells
│   └── Mark which walls to render
│       (only render walls with no neighbor)
│
├── 4. Create collision rectangles
│   ├── Read "rects" layer
│   ├── Create RectanglePlus for each
│   └── Add to RectManager
│
├── 5. Spawn doors
│   ├── Read "doors" layer
│   ├── Create Door entities
│   └── Set locked state and keycard type
│
├── 6. Spawn enemies
│   ├── Read "enemies" layer
│   ├── Create appropriate enemy type
│   │   (Skull, Eye, Fireball)
│   └── Add to EntityManager
│
├── 7. Set spawn point
│   ├── Read "teleports" layer
│   └── Store spawn position
│
└── 8. Spawn items
    ├── Read "items" layer
    └── Create Key entities
```

**Map Layers**:
- `floor` - Floor tiles
- `ceiling` - Ceiling tiles
- `rects` - Collision walls
- `enemies` - Enemy spawn points
- `doors` - Door placements
- `teleports` - Player spawn/exit points
- `items` - Item pickups

**Reference**: `MapBuilder.java:40-289`

---

## Player Input Flow

Input is processed every frame in a specific order:

### Input Processing Order

```
PlayScreen.handleInput(delta)
├── 1. Check game state
│   ├── If player is dead → show menu
│   └── If ESC pressed → toggle pause menu
│
├── 2. Handle menu navigation (if menu shown)
│   ├── UP/DOWN keys → change selection
│   ├── ENTER/SPACE → activate option
│   │   ├── Continue → resume game
│   │   └── Quit → return to MainMenuScreen
│   └── game.gameIsPaused = true
│
└── 3. Handle player controls (if not paused and alive)
    └── player.handleInput(delta)
        ├── WASD/Arrow keys → movement
        │   ├── Calculate velocity vector
        │   ├── Check collision with RectManager
        │   └── Update position if valid
        │
        ├── Mouse movement → look around
        │   └── Rotate player camera
        │
        ├── Left click → shoot
        │   ├── Raycast from camera
        │   ├── Check if hit enemy
        │   └── Deal damage
        │
        ├── E key → use/interact
        │   └── Open doors (if has keycard)
        │
        └── 1-6 keys → select weapon/item
            └── Change current inventory slot
```

### Collision Detection Flow

```
Player Movement:
├── 1. Calculate new position
│   └── newPosition = oldPosition + velocity * delta
│
├── 2. Check X-axis collision
│   ├── Temporarily move on X-axis
│   ├── RectManager.checkCollision(playerRect)
│   ├── If collision → revert X position
│   └── else → apply X movement
│
└── 3. Check Y-axis collision
    ├── Temporarily move on Y-axis
    ├── RectManager.checkCollision(playerRect)
    ├── If collision → revert Y position
    └── else → apply Y movement
```

**Reference**:
- `PlayScreen.java:102-167`
- `GameScreen.java:33-71`

---

## Combat Flow

### Player Attacks Enemy

```
1. Player clicks (shoots)
   └── Player.handleInput()
       └── if (leftClick && canShoot)

2. Perform raycast
   └── Cast ray from camera forward
       └── Check intersection with enemies

3. If hit enemy
   └── enemy.subHp(damage)
       ├── currentHp -= damage
       ├── checkIfDead()
       │   └── if (currentHp == 0) isDead = true
       └── destroyIfDead()
           ├── destroy = true
           └── destroy()
               ├── Remove collision rect
               └── Remove from EntityManager
```

### Enemy AI Flow (General Pattern)

```
Enemy.tick(delta)
├── 1. Check player distance
│   └── Calculate distance to player
│       └── if (distance < detectionRange)
│           └── isPlayerInRange = true
│
├── 2. AI decision making
│   └── EnemyAI.update(delta)
│       ├── If player in range:
│       │   ├── Face player
│       │   ├── Move toward player
│       │   └── Attack if close enough
│       │
│       └── Else:
│           └── Idle or patrol
│
└── 3. Update collision rect
    └── rect.setPosition(enemyPosition)
```

### Enemy Attacks Player

```
1. Enemy in attack range
   └── EnemyAI detects player proximity

2. Trigger attack
   └── Deal damage to player
       └── player.subHp(damage)
           ├── currentHp -= damage
           ├── Show blood overlay effect
           ├── Play hurt sound
           └── if (currentHp <= 0)
               ├── isDead = true
               └── Show death screen
```

**Reference**:
- `Enemy.java:12-104`
- `Player.java` (for player-side combat)

---

## Rendering Pipeline

### Frame Rendering Order

```
PlayScreen.render(delta)
├── 1. Setup
│   └── currentCam.update()
│
├── 2. Render to FrameBuffer (FBO)
│   ├── fbo.begin()
│   ├── Clear with fog color
│   │
│   ├── Draw skybox (2D background)
│   │   └── SpriteBatch.draw(skyBg)
│   │
│   ├── Render 3D world
│   │   └── ModelBatch.begin(camera)
│   │       └── EntityManager.render3DAllEntities()
│   │           └── For each entity:
│   │               ├── Frustum culling check
│   │               └── mdlBatch.render(entity.model)
│   │
│   └── fbo.end()
│
├── 3. Render to screen
│   ├── SpriteBatch.begin()
│   │
│   ├── Draw blood overlay (if damaged)
│   │
│   ├── Draw FBO to screen
│   │   └── draw(fbo.getColorBufferTexture())
│   │
│   ├── Draw player weapon (HUD)
│   │
│   ├── Draw HUD elements
│   │   ├── Health bar
│   │   ├── Inventory slots
│   │   └── Keycards
│   │
│   ├── Draw menu (if paused)
│   │   ├── "YOU DIED!" (if dead)
│   │   ├── Continue option
│   │   └── Quit option
│   │
│   └── SpriteBatch.end()
│
└── Done!
```

**Reference**: `PlayScreen.java:182-294`

---

## Key Takeaways

1. **Entry Point**: Platform launcher → `Foxenstein3D.create()` → `MainMenuScreen`

2. **Game Loop**: `Foxenstein3D.render()` → `currentScreen.render()` → handle input, tick entities, render

3. **Entity System**: All game objects inherit from `Entity`, managed by `EntityManager`

4. **Map Loading**: Tiled maps parsed by `MapBuilder` into entities (cells, enemies, doors, items)

5. **Collision**: `RectanglePlus` rectangles managed by `RectManager`, checked on X/Y axes separately

6. **Rendering**: 3D world → FrameBuffer → Screen with 2D UI overlay

7. **Screen Management**: libGDX Screen interface handles different game states

---

## Common Development Tasks

### Adding a New Enemy Type

1. Create class extending `Enemy` (e.g., `NewEnemy.java`)
2. Create corresponding AI class extending `EnemyAI`
3. Add case in `MapBuilder.buildMap()` to spawn from map
4. Add enemy type to Tiled map editor enum

### Adding a New Weapon

1. Add weapon texture to asset manager
2. Update `Player.java` to handle new inventory slot
3. Add shooting logic in `Player.handleInput()`
4. Update HUD rendering in `PlayScreen.render()`

### Creating a New Level

1. Open Tiled Map Editor
2. Create map with required layers (floor, ceiling, rects, enemies, doors, teleports, items)
3. Export as `.tmx` file
4. Load in `AssetsManager`
5. Call `MapBuilder.buildMap()` with new map

---

For more information on the overall architecture, see [ARCHITECTURE.md](ARCHITECTURE.md).
