# Architecture Documentation

This document provides an overview of the Foxenstein3D (Ultra Nightmare) codebase architecture, design patterns, and key systems.

## Table of Contents
- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Core Architecture](#core-architecture)
- [Key Systems](#key-systems)
- [Design Patterns](#design-patterns)
- [Data Flow](#data-flow)

---

## Project Overview

**Foxenstein3D** (game name: **Ultra Nightmare**) is a retro-style first-person shooter inspired by Wolfenstein 3D. Built during the libGDX Game Jam (December 2020), it features:

- 3D raycasting-style rendering
- Enemy AI with multiple enemy types
- Keycard-based progression
- Retro pixelated graphics
- Cross-platform support (Desktop, HTML)

**Language**: Java
**Framework**: libGDX
**Engine Type**: 3D (pseudo-3D Wolfenstein style)

---

## Technology Stack

### Core Framework
- **libGDX 3.x** - Cross-platform game development framework
  - Graphics rendering (OpenGL)
  - Input handling
  - Asset management
  - Audio system

### Key Libraries
- **LWJGL 3** - Low-level OpenGL/input bindings (desktop)
- **GWT** - Google Web Toolkit (HTML5 platform)
- **Gradle** - Build system

### Asset Tools
- **Tiled Map Editor** - Level design (.tmx files)
- Custom texture atlases for sprites

---

## Project Structure

```
enstein3D/
├── core/                          # Shared game logic
│   └── src/mysko/pilzhere/fox3d/
│       ├── Foxenstein3D.java      # Main game class
│       │
│       ├── screens/               # Game screens
│       │   ├── GameScreen.java    # Base screen class
│       │   ├── MainMenuScreen.java
│       │   └── PlayScreen.java    # Main gameplay
│       │
│       ├── entities/              # Game objects
│       │   ├── Entity.java        # Base entity class
│       │   ├── player/
│       │   │   └── Player.java
│       │   ├── enemies/
│       │   │   ├── Enemy.java     # Base enemy class
│       │   │   ├── Skull.java
│       │   │   ├── Eye.java
│       │   │   ├── Fireball.java
│       │   │   └── ai/            # Enemy AI
│       │   │       ├── EnemyAI.java
│       │   │       ├── SkullAI.java
│       │   │       ├── EyeAI.java
│       │   │       └── FireballAI.java
│       │   ├── objects/
│       │   │   └── Key.java       # Collectible keycards
│       │   ├── Door.java          # Interactive doors
│       │   ├── Grid.java          # Debug grid
│       │   └── IUsable.java       # Interface for interactive objects
│       │
│       ├── cell/                  # 3D world cells
│       │   └── Cell3D.java        # Floor/ceiling/wall tiles
│       │
│       ├── maps/                  # Level management
│       │   └── MapBuilder.java    # Tiled map parser
│       │
│       ├── utils/                 # Utilities
│       │   └── EntityManager.java # Entity lifecycle management
│       │
│       ├── rect/                  # Collision system
│       │   ├── RectManager.java
│       │   ├── RectanglePlus.java
│       │   └── filters/
│       │       └── RectanglePlusFilter.java
│       │
│       ├── models/                # 3D model utilities
│       │   ├── ModelMaker.java
│       │   └── ModelInstanceBB.java  # Model with bounding box
│       │
│       ├── assets/                # Asset loading
│       │   └── managers/
│       │       └── AssetsManager.java
│       │
│       ├── input/                 # Input handling
│       │   └── GameInputProcessor.java
│       │
│       ├── filters/               # Rendering filters
│       │   └── OverlapFilterManager.java
│       │
│       └── constants/             # Game constants
│           └── Constants.java
│
├── desktop/                       # Desktop launcher
│   └── src/.../DesktopLauncher.java
│
├── html/                          # HTML5/WebGL launcher
│   └── src/.../HtmlLauncher.java
│
├── gradle/                        # Build configuration
├── readme/                        # Documentation assets
└── docs/                          # Documentation (this folder)
    ├── CODE_FLOW.md
    └── ARCHITECTURE.md
```

---

## Core Architecture

### 1. Game Class: `Foxenstein3D`

The central hub that manages:
- **Rendering resources** (SpriteBatch, ModelBatch, FrameBuffer)
- **Global managers** (AssetsManager, EntityManager, RectManager, MapBuilder)
- **Screen lifecycle** (MainMenuScreen, PlayScreen)
- **Game state** (pause, volume settings)

**Pattern**: Singleton-like (single instance passed around)

**Reference**: `Foxenstein3D.java`

---

### 2. Screen System

Uses libGDX's `Screen` interface for game states:

```
Screen (interface)
    └── GameScreen (abstract base)
            ├── MainMenuScreen
            └── PlayScreen
```

**Responsibilities**:
- `handleInput(delta)` - Process user input
- `tick(delta)` - Update game logic
- `render(delta)` - Draw frame
- `resize(width, height)` - Handle window resize
- `show()`, `hide()`, `pause()`, `resume()`, `dispose()` - Lifecycle hooks

**Pattern**: State Pattern (each screen is a game state)

**Reference**: `screens/GameScreen.java`

---

### 3. Entity System

Component-like entity system where all game objects inherit from `Entity`:

```
Entity (base class)
    ├── Player
    ├── Enemy
    │   ├── Skull
    │   ├── Eye
    │   └── Fireball
    ├── Door
    ├── Key
    ├── Cell3D
    └── Grid
```

**Entity Properties**:
- `id` - Unique identifier
- `tick` - Whether to update each frame
- `render2D` - Whether to render in 2D pass
- `render3D` - Whether to render in 3D pass
- `destroy` - Marked for removal

**Entity Methods**:
- `tick(delta)` - Update logic
- `render2D(delta)` - 2D rendering
- `render3D(mdlBatch, env, delta)` - 3D rendering
- `destroy()` - Cleanup
- `onCollision(otherRect)` - Handle collision

**Pattern**: Entity-Component System (lightweight)

**Reference**: `entities/Entity.java`

---

### 4. Entity Manager

Central registry for all entities:

**Responsibilities**:
- Assign unique IDs
- Add/remove entities
- Batch update all entities (`tickAllEntities()`)
- Batch render all entities (`render2DAllEntities()`, `render3DAllEntities()`)
- Entity lookup by ID

**Pattern**: Manager Pattern / Registry Pattern

**Reference**: `utils/EntityManager.java`

---

### 5. Collision System

Rectangle-based 2D collision for a 3D game:

```
RectManager
    └── Manages array of RectanglePlus objects
        └── RectanglePlus
            ├── Position (x, y)
            ├── Size (width, height)
            ├── Connected entity ID
            └── Filter type (WALL, ENEMY, PLAYER, etc.)
```

**Collision Detection**:
1. Separate X and Y axis checks
2. Move on X, check collision, revert if needed
3. Move on Y, check collision, revert if needed

**Pattern**: Axis-Aligned Bounding Box (AABB)

**Reference**: `rect/RectManager.java`, `rect/RectanglePlus.java`

---

### 6. Map System

Levels designed in **Tiled Map Editor**, loaded via `MapBuilder`:

**Map Layers**:
- `floor` - Floor tiles (different textures)
- `ceiling` - Ceiling tiles
- `rects` - Collision walls (invisible rectangles)
- `enemies` - Enemy spawn points with properties
- `doors` - Door locations with lock status
- `teleports` - Player spawn and exit points
- `items` - Collectible items (keycards, etc.)

**MapBuilder Process**:
1. Parse `.tmx` file (TiledMap)
2. Convert tiles to Cell3D entities
3. Determine which walls to render (neighbors check)
4. Create collision rectangles
5. Spawn entities (enemies, doors, items)
6. Set player spawn point

**Pattern**: Builder Pattern

**Reference**: `maps/MapBuilder.java`

---

### 7. Rendering Pipeline

Multi-pass rendering for retro aesthetic:

```
Pass 1: Render to FrameBuffer (FBO)
    ├── Draw skybox background (2D)
    └── Draw 3D world (ModelBatch)
        └── Frustum culling for performance

Pass 2: Post-process and UI
    ├── Draw FBO to screen (scaled, pixelated)
    ├── Apply blood overlay (if damaged)
    ├── Draw weapon sprite (centered)
    └── Draw HUD (health, inventory, keycards)

Pass 3: Menu/Overlays
    └── Pause menu, death screen, etc.
```

**FBO Benefits**:
- Fixed internal resolution (pixelated look)
- Post-processing effects
- Resolution-independent rendering

**Pattern**: Multi-pass rendering, Post-processing pipeline

**Reference**: `PlayScreen.java:182-294`

---

### 8. Asset Management

Centralized asset loading via `AssetsManager`:

**Asset Types**:
- Textures (atlases, backgrounds, UI)
- Models (3D meshes)
- Fonts (bitmap fonts)
- Audio (music, sound effects)
- Maps (Tiled .tmx files)

**Loading Strategy**:
- All assets loaded at startup (`finishLoading()`)
- Assets stored in libGDX AssetManager
- Accessed via getter methods

**Pattern**: Service Locator / Asset Manager Pattern

**Reference**: `assets/managers/AssetsManager.java`

---

## Key Systems

### Player System

**Components**:
- **Movement**: WASD + mouse look, collision detection
- **Camera**: First-person perspective camera (PerspectiveCamera)
- **Inventory**: 6 slots for weapons/items, keycards tracked separately
- **Combat**: Raycasting-based shooting, weapon animations
- **Health**: HP system with damage feedback (blood overlay)

**Reference**: `entities/player/Player.java`

---

### Enemy AI System

Each enemy type has a dedicated AI controller:

**AI Components**:
- **Detection**: Distance-based player detection
- **Pathfinding**: Simple direct movement toward player
- **Attack**: Close-range damage dealing
- **Animation**: Sprite flashing when hit

**Enemy Types**:
- **Skull**: Basic melee enemy
- **Eye**: Ranged projectile enemy
- **Fireball**: (Implementation details in FireballAI)

**Pattern**: Strategy Pattern (each AI is interchangeable)

**Reference**: `entities/enemies/ai/`

---

### Door System

**Features**:
- Locked/unlocked state
- Keycard requirements (4 types: red, green, blue, golden)
- Animation (sliding open)
- Collision (blocks player when closed)

**Interface**: `IUsable` - for interactable objects

**Reference**: `entities/Door.java`

---

### Input System

Custom input processor for game-specific controls:

**Handled Input**:
- Keyboard (WASD, E, ESC, 1-6, arrow keys, etc.)
- Mouse (movement, clicks)
- Scroll wheel (weapon switching)

**Pattern**: Command Pattern (input maps to actions)

**Reference**: `input/GameInputProcessor.java`

---

## Design Patterns

### 1. **State Pattern** (Screen System)
Different screens represent different game states (menu, gameplay, pause).

### 2. **Entity-Component System** (Lightweight)
All game objects extend `Entity` with tick/render flags.

### 3. **Manager Pattern**
- `EntityManager` - Entity lifecycle
- `AssetsManager` - Asset loading
- `RectManager` - Collision detection

### 4. **Strategy Pattern** (Enemy AI)
Each enemy type has swappable AI behavior.

### 5. **Builder Pattern** (MapBuilder)
Complex map construction from Tiled data.

### 6. **Observer Pattern** (Collision)
Entities notified via `onCollision()` when rectangles overlap.

### 7. **Service Locator** (Global managers)
`Foxenstein3D` instance passed around for manager access.

---

## Data Flow

### Typical Frame Data Flow

```
1. Input
   User Input → GameInputProcessor → Player/Screen

2. Update (Tick)
   EntityManager.tickAllEntities()
       ├── Player.tick() → Update position, check collisions
       ├── Enemy.tick() → AI updates, movement
       └── Door.tick() → Animation updates

3. Collision Detection
   RectManager.checkCollision()
       └── Entity.onCollision() callbacks

4. Render
   Screen.render()
       ├── Skybox (SpriteBatch)
       ├── 3D World (ModelBatch) → Frustum culling
       └── UI (SpriteBatch) → HUD, overlays, menus

5. Cleanup
   EntityManager removes destroyed entities
```

---

## Performance Considerations

### 1. Frustum Culling
Only render entities visible to the camera:
```java
mdlInst.setInFrustum(screen.frustumCull(camera, mdlInst));
if (mdlInst.isInFrustum()) {
    mdlBatch.render(mdlInst, env);
}
```

**Reference**: `models/ModelInstanceBB.java`, `GameScreen.java:78-87`

### 2. FrameBuffer Resolution
Low internal resolution (see `Constants.FBO_WIDTH_ORIGINAL`) for retro look and performance.

### 3. Entity Tick/Render Flags
Entities can disable tick/render to skip processing:
```java
entity.setTick(false);    // Stop updating
entity.setRender3D(false); // Stop rendering
```

### 4. Batch Rendering
All entities rendered in batches (ModelBatch, SpriteBatch) for efficiency.

---

## Common Gotchas

### 1. Entity ID Assignment
IDs assigned on construction, incremental. Don't create entities outside `EntityManager`.

### 2. Collision Rect Cleanup
Entities with collision must remove their rect in `destroy()`:
```java
screen.game.getRectMan().removeRect(rect);
```

### 3. Screen Disposal
Switching screens doesn't auto-dispose. Must manually call `removeAllEntities()` and `dispose()`.

### 4. Pause Handling
Check `game.gameIsPaused` before updating game logic in `tick()`.

### 5. Coordinate Systems
- Tiled map coordinates converted to world coordinates
- Map center = (0, 0) in world space
- Conversion: `worldPos = tilePos - mapSize/2`

---

## Extension Points

### Adding New Content

1. **New Enemy Type**
   - Extend `Enemy`
   - Create AI class extending `EnemyAI`
   - Add spawn logic in `MapBuilder`

2. **New Weapon**
   - Add texture to `AssetsManager`
   - Update `Player` inventory handling
   - Implement shooting logic

3. **New Interactable**
   - Implement `IUsable` interface
   - Handle in `Player` use logic

4. **New Level**
   - Design in Tiled Map Editor
   - Add to `AssetsManager`
   - Load via `MapBuilder`

5. **New Screen**
   - Extend `GameScreen`
   - Implement render/input logic
   - Transition via `game.setScreen()`

---

## Testing & Debugging

### Debug Features
- **Grid overlay**: Uncomment grid entity spawn in `PlayScreen.java:84`
- **FPS counter**: libGDX provides `Gdx.graphics.getFramesPerSecond()`
- **Entity count**: Check `game.getEntMan().entities.size`

### Useful Debug Keybinds
- **ESC**: Toggle pause/menu (useful for testing)
- **F**: (Commented) Toggle cursor catch
- **Q**: (Commented) Destroy last entity

### Console Output
Many classes have commented-out `System.out.println()` for debugging. Uncomment as needed.

---

## Resources

### External Documentation
- [libGDX Wiki](https://libgdx.com/wiki/)
- [Tiled Map Editor](https://www.mapeditor.org/)
- [LWJGL](https://www.lwjgl.org/)

### Internal Documentation
- [CODE_FLOW.md](CODE_FLOW.md) - Execution flow and game loop
- [TODO.md](../readme/TODO.md) - Planned features

### Contributors
See [README.md](../README.md) for contributor information.

---

## Summary

Foxenstein3D uses a **component-based entity system** with **manager pattern** for organization. The **screen system** handles game states, while **MapBuilder** transforms Tiled maps into playable 3D levels. Rendering uses a **multi-pass pipeline** with FrameBuffer for post-processing and retro aesthetics.

Key strengths:
- Clean separation of concerns (entities, managers, screens)
- Extensible AI system
- Data-driven level design (Tiled maps)
- Cross-platform support via libGDX

For step-by-step code flow, see [CODE_FLOW.md](CODE_FLOW.md).
