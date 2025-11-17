# Level Editor Guide

## Overview

Foxenstein3D uses **[Tiled Map Editor](https://www.mapeditor.org/)**, an industry-standard, open-source tool for creating game levels. This game does **not** have an in-game level editor - all level design is done externally in Tiled and then loaded into the game.

## Table of Contents

1. [Getting Started](#getting-started)
   - [Installing Tiled Map Editor](#installing-tiled-map-editor)
   - [Example Maps](#example-maps)
2. [Required Layer Structure](#required-layer-structure)
3. [Creating Your First Level](#creating-your-first-level)
4. [Object Types and Properties](#object-types-and-properties)
5. [Adding Levels to the Game](#adding-levels-to-the-game)
6. [Tips and Best Practices](#tips-and-best-practices)
7. [Troubleshooting](#troubleshooting)

---

## Getting Started

### Installing Tiled Map Editor

1. Download Tiled from [https://www.mapeditor.org/](https://www.mapeditor.org/)
2. Install the application for your operating system (Windows, macOS, or Linux)
3. Launch Tiled

### Example Maps

The game includes **2 example maps** that you can use as learning references. These are excellent starting points for understanding level structure and design.

#### map01.tmx - Main Gameplay Level

**Location**: `core/assets/maps/map01.tmx`
**Size**: 64x64 tiles (61KB)
**Purpose**: Full gameplay demonstration map

**Contains**:
- **56 Skull enemies** - Shows extensive enemy placement strategies
- **12 Doors** - Various lock configurations and orientations
- **4 Keycards** - Collectible items for unlocking doors
- **45 Wall collision rectangles** - Complete collision setup
- **3 Enemy types**: Skull, Eye, Fireball
- **Spawn and Exit points** - Complete level flow from start to finish
- Floor and ceiling layers with varied tiles

**Best for learning**: Complete level design, enemy placement strategies, keycard progression systems, complex layouts

#### mapTitle.tmx - Title Screen Level

**Location**: `core/assets/maps/mapTitle.tmx`
**Size**: 64x64 tiles (47KB)
**Purpose**: Simpler demonstration for title/menu screen

**Contains**:
- **1 Eye enemy** - Minimal enemy presence for background effect
- **5 Doors** - Basic door configuration examples
- **21 Wall collision rectangles** - Simpler collision layout
- **Spawn point** - Player starting position
- Floor and ceiling layers with basic design

**Best for learning**: Basic map structure, simple layout design, minimal complexity

#### How to Open Example Maps

**In Tiled Map Editor:**
1. Launch Tiled
2. Go to **File > Open**
3. Navigate to `core/assets/maps/`
4. Open either `map01.tmx` or `mapTitle.tmx`

**In a Text Editor:**
You can also examine the `.tmx` files directly in any text editor to see the XML structure.

#### What to Study in the Examples

1. **Layer Organization** - See how floor, ceiling, and all object groups are structured
2. **Door Properties** - Check how `direction`, `locked`, and `keytype` are set
3. **Enemy Placement** - Notice spacing, positioning strategies, and variety
4. **Collision Rectangles** - See how walls are defined in the `rects` layer
5. **Spawn Points** - Find the `MapSpawn` teleport object and its placement
6. **Item Placement** - See where keycards are positioned relative to locked doors
7. **Tileset Usage** - Observe how `atlas01.tsx` and `atlasEnemies.tsx` are used

**Recommendation**: Start with `mapTitle.tmx` since it's simpler and easier to understand, then move to `map01.tmx` to see a more complex level with full gameplay elements and advanced design patterns

---

## Required Layer Structure

Every level must have the following layers in this order:

### Tile Layers

1. **floor** (Tile Layer)
   - Base walkable surface
   - Uses `atlas01.tsx` tileset
   - Required for all areas players can walk

2. **ceiling** (Tile Layer)
   - Optional ceiling/roof tiles
   - Uses `atlas01.tsx` tileset
   - Can be left empty or marked invisible

### Object Groups

3. **zones** (Object Group)
   - Currently unused but should be included for future features
   - Can be left empty

4. **rects** (Object Group)
   - Invisible collision rectangles for walls
   - Draw rectangles where solid walls should be

5. **doors** (Object Group)
   - Interactive door objects
   - See [Door Properties](#doors)

6. **enemies** (Object Group)
   - Enemy spawn points
   - See [Enemy Properties](#enemies)

7. **teleports** (Object Group)
   - Player spawn and exit points
   - See [Teleport Properties](#teleports)

8. **items** (Object Group)
   - Collectible keycards
   - See [Item Properties](#items)

9. **decorations** (Object Group)
   - Currently unused but should be included for future features
   - Can be left empty

---

## Creating Your First Level

### Step 1: Create a New Map

1. In Tiled, go to **File > New > New Map**
2. Set the following properties:
   - **Orientation**: Orthogonal
   - **Tile layer format**: Base64 (uncompressed or zlib compressed)
   - **Tile render order**: Right Down
   - **Map size**: 64 x 64 tiles (or your desired size)
   - **Tile size**: 16 x 16 pixels

### Step 2: Add Tilesets

1. Go to **Map > Add External Tileset**
2. Add `atlas01.tsx` (main tiles for floor/ceiling/walls)
3. Add `atlasEnemies.tsx` (enemy sprite tiles)

These tileset files are located in `core/assets/maps/`

### Step 3: Create Required Layers

Create all the layers listed in [Required Layer Structure](#required-layer-structure):

1. Right-click in the Layers panel
2. Select **Add Layer** > **Tile Layer** (for floor and ceiling)
3. Select **Add Layer** > **Object Layer** (for all object groups)
4. Name each layer exactly as shown above

### Step 4: Design Your Floor

1. Select the **floor** layer
2. Choose tiles from the `atlas01` tileset
3. Paint your floor layout
4. Leave areas you don't want walkable empty

### Step 5: Add Collision Walls

1. Select the **rects** layer
2. Use the **Insert Rectangle** tool (R key)
3. Draw rectangles where walls should block player movement
4. These are invisible in-game but create collision boundaries

### Step 6: Place Objects

Add doors, enemies, items, and spawn points as needed. See [Object Types and Properties](#object-types-and-properties) below.

### Step 7: Save Your Map

1. Go to **File > Save As**
2. Save as `.tmx` format in `core/assets/maps/`
3. Use a descriptive name like `map02.tmx`

---

## Object Types and Properties

### Doors

Doors are interactive objects that can be locked and require specific keycards.

**How to Add a Door:**
1. Select the **doors** layer
2. Select the door tile from `atlas01` tileset (usually tile ID 14)
3. Use **Insert Tile** tool to place the door
4. Select the placed door object
5. In the Properties panel, add custom properties:

**Required Properties:**

| Property | Type | Values | Description |
|----------|------|--------|-------------|
| `direction` | string | north, south, east, west | Which direction the door faces |
| `locked` | bool | true, false | Whether the door requires a key |
| `keytype` | int | 0-4 | Which keycard type unlocks it (0 = no key) |

**Example Door Properties:**
```xml
<object id="18" name="door" gid="14" x="112" y="112" width="16" height="16">
  <properties>
    <property name="direction" value="north"/>
    <property name="keytype" type="int" value="1"/>
    <property name="locked" type="bool" value="true"/>
  </properties>
</object>
```

**Keycard Types:**
- `0` - No key required (unlocked door)
- `1` - Keycard type 1 (typically red)
- `2` - Keycard type 2 (typically blue)
- `3` - Keycard type 3 (typically yellow)
- `4` - Keycard type 4 (typically green)

---

### Enemies

Enemy objects define spawn points for different enemy types.

**How to Add an Enemy:**
1. Select the **enemies** layer
2. Select an enemy tile from `atlasEnemies` tileset
3. Use **Insert Tile** tool to place the enemy
4. Select the placed enemy object
5. Add the `EnemyType` property

**Required Properties:**

| Property | Type | Values | Description |
|----------|------|--------|-------------|
| `EnemyType` | string | Skull, Eye, Fireball | The type of enemy to spawn |

**Available Enemy Types:**
- **Skull** - Basic enemy type
- **Eye** - Eye enemy type
- **Fireball** - Fireball enemy type

**Example Enemy Properties:**
```xml
<object id="5" name="enemy" gid="65" x="256" y="256" width="16" height="16">
  <properties>
    <property name="EnemyType" value="Skull"/>
  </properties>
</object>
```

---

### Teleports

Teleport objects define player spawn points and level exits.

**How to Add a Teleport:**
1. Select the **teleports** layer
2. Use **Insert Rectangle** tool to place a small rectangle
3. Select the placed teleport object
4. Add properties for spawn or exit

**Required Properties:**

| Property | Type | Values | Description |
|----------|------|--------|-------------|
| `MapSpawn` | bool | true, false | Mark as player spawn point |
| `ExitSpot` | bool | true, false | Mark as level exit |

**Important Notes:**
- Every map **must have exactly one** `MapSpawn` set to `true`
- Exit spots are optional
- Only one property should be `true` per teleport object

**Example Spawn Point:**
```xml
<object id="1" x="32" y="32" width="16" height="16">
  <properties>
    <property name="MapSpawn" type="bool" value="true"/>
  </properties>
</object>
```

**Example Exit Point:**
```xml
<object id="2" x="480" y="480" width="16" height="16">
  <properties>
    <property name="ExitSpot" type="bool" value="true"/>
  </properties>
</object>
```

---

### Items

Item objects are collectible keycards that unlock doors.

**How to Add an Item:**
1. Select the **items** layer
2. Use **Insert Rectangle** or **Insert Tile** tool
3. Select the placed item object
4. Add the `KeyType` property

**Required Properties:**

| Property | Type | Values | Description |
|----------|------|--------|-------------|
| `KeyType` | int | 1-4 | Which keycard type this is |

**Keycard Types:**
- `1` - Keycard type 1 (typically red)
- `2` - Keycard type 2 (typically blue)
- `3` - Keycard type 3 (typically yellow)
- `4` - Keycard type 4 (typically green)

**Example Item:**
```xml
<object id="10" x="128" y="128" width="16" height="16">
  <properties>
    <property name="KeyType" type="int" value="1"/>
  </properties>
</object>
```

---

## Adding Levels to the Game

Once you've created a level in Tiled, you need to register it in the game code.

### Step 1: Register the Map in AssetsManager

Edit `core/src/mysko/pilzhere/fox3d/assets/managers/AssetsManager.java`:

1. Add a public field for your map:
```java
public final String mapYourLevel = "maps/mapYourLevel.tmx";
```

2. Load the map in the `loadMaps()` method:
```java
public void loadMaps() {
    setLoader(TiledMap.class, new TmxMapLoader());
    load(mapTitle, TiledMap.class);
    load(map01, TiledMap.class);
    load(mapYourLevel, TiledMap.class);  // Add this line
}
```

### Step 2: Load the Map in a Screen

To actually use your map, you need to load it in a game screen. The typical place is in `PlayScreen.java`:

```java
// In PlayScreen constructor or show() method
game.getMapBuilder().buildMap(
    (TiledMap) game.getAssMan().get(game.getAssMan().mapYourLevel)
);
```

### Step 3: Test Your Level

1. Compile and run the game
2. Navigate to where your map is loaded
3. Check that:
   - Floor tiles render correctly
   - Walls block movement
   - Doors work and keycards unlock them
   - Enemies spawn
   - Player spawn point is correct

---

## Tips and Best Practices

### Level Design

1. **Start Simple**: Begin with a small area and expand
2. **Test Frequently**: Load your map in the game regularly to catch issues early
3. **Use Layers Wisely**: Keep the layer structure consistent across all maps
4. **Plan Your Layout**: Sketch out your level design on paper first

### Performance

1. **Reasonable Map Sizes**: 64x64 is standard, but you can go larger or smaller
2. **Optimize Collision**: Use as few collision rectangles as possible
3. **Enemy Placement**: Don't spawn too many enemies in one area

### Visual Design

1. **Tile Variety**: Use different floor tiles to create visual interest
2. **Ceiling Optional**: Ceiling layer can be left empty for an open-sky feel
3. **Consistent Theme**: Use similar tile patterns throughout the level

### Gameplay

1. **Key Progression**: Create a logical path requiring keycards in sequence
2. **Enemy Difficulty**: Place harder enemies deeper in the level
3. **Multiple Paths**: Consider alternate routes for player exploration
4. **Test Playability**: Make sure the level is fun and not frustrating

### File Organization

1. **Naming Convention**: Use clear names like `map01.tmx`, `map02.tmx`
2. **Backup Maps**: Keep backup copies of your maps before major changes
3. **Documentation**: Comment your map properties in Tiled if needed

---

## Troubleshooting

### Map Doesn't Load

**Problem**: Game crashes or map doesn't appear

**Solutions**:
- Check that all required layers exist with exact names
- Verify tilesets are properly linked
- Ensure the map is registered in `AssetsManager.java`
- Check console for error messages

### Player Spawns in Wrong Location

**Problem**: Player starts in an unexpected position

**Solutions**:
- Verify exactly one `MapSpawn` property is set to `true` in the teleports layer
- Check the coordinates of your spawn point
- Make sure the spawn point is on a valid floor tile

### Doors Don't Work

**Problem**: Doors don't open or keycards don't unlock them

**Solutions**:
- Check that door properties are spelled exactly right (case-sensitive)
- Verify `keytype` matches the `KeyType` of collectible items
- Ensure `locked` is set to `true` for locked doors
- Check that `direction` is one of: north, south, east, west

### Collision Issues

**Problem**: Player walks through walls or gets stuck

**Solutions**:
- Verify collision rectangles in the `rects` layer cover all wall areas
- Check for overlapping rectangles causing weird collision
- Make sure collision rectangles align with visual wall tiles

### Enemies Don't Spawn

**Problem**: No enemies appear in the level

**Solutions**:
- Check that `EnemyType` property is spelled correctly
- Verify enemy type is one of: Skull, Eye, Fireball (case-sensitive)
- Make sure enemy is placed on a valid floor tile
- Check console for spawn errors

### Keycards Not Collectible

**Problem**: Can't pick up keycards

**Solutions**:
- Verify `KeyType` property exists and is set to 1-4
- Make sure item is on the `items` layer
- Check that the item is on a valid floor tile

---

## Understanding Coordinates

The game uses a coordinate transformation system:

```
worldPosition = tilePosition - (mapSize / 2)
```

This centers the map at world coordinates (0, 0). For a 64x64 map:
- Tile position (0, 0) becomes world position (-32, -32)
- Tile position (32, 32) becomes world position (0, 0)
- Tile position (64, 64) becomes world position (32, 32)

You generally don't need to worry about this when designing in Tiled - the conversion happens automatically.

---

## Code References

For advanced users who want to understand how maps are processed:

- **Map Loading**: `core/src/mysko/pilzhere/fox3d/assets/managers/AssetsManager.java`
- **Map Building**: `core/src/mysko/pilzhere/fox3d/maps/MapBuilder.java`
- **Map Usage**: `core/src/mysko/pilzhere/fox3d/screens/PlayScreen.java`
- **Architecture Documentation**: `docs/ARCHITECTURE.md` (lines 251-274)
- **Code Flow Documentation**: `docs/CODE_FLOW.md` (lines 172-229)

---

## Example Workflow

Here's a complete workflow for creating a new level:

1. Open Tiled Map Editor
2. Create new 64x64 orthogonal map with 16x16 tiles
3. Add external tilesets: `atlas01.tsx` and `atlasEnemies.tsx`
4. Create all required layers (floor, ceiling, zones, rects, doors, enemies, teleports, items, decorations)
5. Design floor layout on the **floor** layer
6. Add collision rectangles on the **rects** layer
7. Place one spawn point in **teleports** layer with `MapSpawn: true`
8. Add doors to **doors** layer with appropriate properties
9. Place enemies on **enemies** layer with `EnemyType` properties
10. Add keycards to **items** layer with `KeyType` properties
11. Save as `map02.tmx` in `core/assets/maps/`
12. Register in `AssetsManager.java`
13. Load in `PlayScreen.java` (or create new screen)
14. Test in game
15. Iterate and refine

---

## Additional Resources

- **Tiled Documentation**: [https://doc.mapeditor.org/](https://doc.mapeditor.org/)
- **Tiled Forums**: [https://discourse.mapeditor.org/](https://discourse.mapeditor.org/)
- **LibGDX Tiled Maps**: [https://libgdx.com/wiki/graphics/2d/tile-maps](https://libgdx.com/wiki/graphics/2d/tile-maps)

---

## Summary

Foxenstein3D uses a **data-driven level design system** powered by Tiled Map Editor. This approach gives you:

- Professional-grade level editing tools
- Visual WYSIWYG interface
- Easy iteration and testing
- Unlimited level creation potential
- Industry-standard workflow

Start by opening and studying `map01.tmx`, then create your own masterpiece!
