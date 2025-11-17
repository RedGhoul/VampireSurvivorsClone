# Level Creation Guide

This guide will walk you through creating a new level for the Vampire Survivors Clone game.

## Table of Contents
1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Creating a Level Blueprint](#creating-a-level-blueprint)
4. [Creating a Level Scene](#creating-a-level-scene)
5. [Configuration Reference](#configuration-reference)
6. [Testing Your Level](#testing-your-level)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## Overview

The game uses a **data-driven level system** consisting of two main components:

1. **LevelBlueprint (ScriptableObject)** - Defines level configuration (duration, monsters, spawn curves, loot, etc.)
2. **Level Scene** - Contains the game objects and systems that run during gameplay

### Key Systems

- **LevelManager** (`Assets/Scripts/Gameplay/LevelManager.cs:1`) - Orchestrates game flow, timers, and boss spawns
- **EntityManager** (`Assets/Scripts/Monsters/EntityManager.cs:1`) - Handles monster spawning and object pooling
- **MonsterSpawnTable** (`Assets/Scripts/Monsters/MonsterSpawnTable.cs:1`) - Time-based spawn curves

---

## Quick Start

**Creating a new level involves 3 main steps:**

1. Create a LevelBlueprint ScriptableObject asset
2. Duplicate and modify an existing level scene
3. Wire everything together and test

**Estimated time:** 30-60 minutes for a basic level

---

## Creating a Level Blueprint

A LevelBlueprint is a ScriptableObject that defines all the configuration for your level.

### Step 1: Create the Asset

1. In Unity Project window, navigate to `Assets/Blueprints/Levels/`
2. Right-click in the folder
3. Select **Create > ScriptableObjects > LevelBlueprint**
4. Name it (e.g., `Level 2.asset`)

### Step 2: Configure Basic Settings

Select your new LevelBlueprint and configure these fields in the Inspector:

#### Duration Settings
- **levelTime** - Total level duration in seconds (default: 600 = 10 minutes)

#### Background
- **backgroundTexture** - Texture2D to use as the repeating background

#### Initial Resources
- **initialGemCount** - Number of gems player starts with (default: 25)

#### Loot Settings
- **chestSpawnInterval** - How often chests spawn in seconds (default: 30)

### Step 3: Configure Abilities

The **abilitiesToSpawn** array defines which weapons/upgrades are available in this level.

1. Set the **Size** to the number of abilities you want (e.g., 19)
2. Drag ability prefabs from `Assets/Prefabs/Abilities/` into each slot

**Tips:**
- Include a good mix of weapons and passive upgrades
- Start with 10-20 abilities for variety
- Check `Level 1.asset` for a reference list

### Step 4: Configure Monsters

The **monstersContainers** array defines which monsters can spawn.

Each **MonstersContainer** holds:
- **monsterVariations** - Array of MonsterBlueprint assets (3 variants recommended)
- **spawnTime** - When this monster type becomes available (in seconds)

**Example Configuration:**
```
Size: 4 containers

Container 0 (Melee - spawns at 0s):
  - 初級小兵 (Easy Melee)
  - 中級小兵 (Medium Melee)
  - 高級小兵 (Hard Melee)

Container 1 (Ranged - spawns at 60s):
  - 射擊小兵 (Shooting Monster)

Container 2 (Throwing - spawns at 120s):
  - Gravity Monster

Container 3 (Boomerang - spawns at 180s):
  - Boomerang Monster
```

**Monster Blueprints Location:** `Assets/Blueprints/Monsters/`

### Step 5: Configure Bosses

#### Mini Boss Container
- **spawnTime** - When to spawn (typically 300s = 5 minutes)
- **bossBlueprint** - Drag the Mini Boss blueprint from `Assets/Blueprints/Monsters/`

#### Final Boss Container
- **bossBlueprint** - Drag the Final Boss blueprint
- Spawns automatically when levelTime is reached

**Available Boss Types:**
- Mini Boss
- Final Boss
- Boss variants: Charge, Grenade, Walk, Bullet Hell, Shotgun

### Step 6: Configure Spawn Table

The **spawnTable** defines how monster spawning changes over time using curves.

**Key Fields:**

#### spawnRateCurve (AnimationCurve)
- **Y-axis:** Enemies per second
- **X-axis:** Time (0.0 to 1.0 represents 0% to 100% of level duration)

**Example progression:**
```
Time 0.0 → 1 enemy/sec (start easy)
Time 0.5 → 6 enemies/sec (peak difficulty)
Time 1.0 → 2 enemies/sec (end slower for final boss)
```

#### monsterProbabilityCurves (Array)
- One curve per MonstersContainer
- **Y-axis:** Probability weight (0.0 to 1.0)
- **X-axis:** Time (0.0 to 1.0)

**Example:**
```
Container 0 (Melee):   Starts at 1.0, gradually decreases to 0.3
Container 1 (Ranged):  Starts at 0.0, ramps up to 0.5 at 30%
Container 2 (Throwing): Starts at 0.0, ramps up to 0.3 at 60%
Container 3 (Boomerang): Starts at 0.0, ramps up to 0.2 at 90%
```

#### hpMultiplierOverTime (AnimationCurve)
- Multiplier applied to monster HP as level progresses
- **Y-axis:** HP multiplier (0.0 to 2.0+)
- **X-axis:** Time (0.0 to 1.0)

**Example:**
```
Time 0.0 → 0.0x (monsters at base HP)
Time 0.5 → 1.0x (monsters at 2x HP)
Time 1.0 → 2.0x (monsters at 3x HP)
```

**Tips:**
- Use the curve editor to create smooth progressions
- Test different curves to find the right difficulty
- Look at `Test Level.asset` for simpler configurations
- The X-axis is normalized (0-1), so curves work for any duration

---

## Creating a Level Scene

### Step 1: Duplicate Existing Scene

1. Navigate to `Assets/Scenes/Game/`
2. Duplicate `Level 1.unity` (Ctrl+D or Cmd+D)
3. Rename to your level name (e.g., `Level 2.unity`)
4. Open the new scene

### Step 2: Configure LevelManager

1. Select the **LevelManager** GameObject in the Hierarchy
2. In the Inspector, find the **Level Manager** component
3. Drag your new LevelBlueprint asset into the **levelBlueprint** field

### Step 3: Customize the Scene (Optional)

You can customize various aspects of the scene:

#### Background
1. Select the **Background** or **InfiniteBackground** GameObject
2. Update the sprite/texture to match your theme
3. Alternatively, the background can be set via the LevelBlueprint's backgroundTexture field

#### Lighting & Colors
- Adjust the **Camera** background color
- Add directional lights for atmosphere
- Modify post-processing effects if present

#### Environment Objects (Optional)
- Add decorative sprites (trees, rocks, etc.)
- Ensure they don't interfere with gameplay
- Keep them on appropriate sorting layers

### Step 4: Required Components Checklist

Your scene must have these GameObjects (already present if duplicated from Level 1):

- ✅ **LevelManager** - Game orchestrator
- ✅ **EntityManager** - Monster spawning & pooling
- ✅ **Character/MainCharacter** - Player object
- ✅ **AbilityManager** - Weapon system
- ✅ **Main Camera** - Follows player
- ✅ **Canvas** - UI (health, exp, dialogs)
- ✅ **Background** - Visual backdrop
- ✅ **StatsManager** - Player stats tracking
- ✅ **Object Pools** - For monsters, projectiles, items
- ✅ **SpatialHashGrid** - Collision optimization

**Don't delete these!** The game will not function properly without them.

### Step 5: Add Scene to Build Settings

1. Go to **File > Build Settings**
2. Click **Add Open Scenes** to add your new level
3. Note the **Scene Index** number (you'll need this for loading)

---

## Configuration Reference

### LevelBlueprint Fields

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `levelTime` | float | Level duration in seconds | 600 |
| `backgroundTexture` | Texture2D | Background image | null |
| `initialGemCount` | int | Starting gems for player | 25 |
| `chestSpawnInterval` | float | Seconds between chest spawns | 30 |
| `abilitiesToSpawn` | GameObject[] | Available abilities/weapons | [] |
| `monstersContainers` | MonstersContainer[] | Monster groups with spawn times | [] |
| `miniBossContainer` | BossContainer | Mini-boss config | null |
| `finalBossContainer` | BossContainer | Final boss config | null |
| `spawnTable` | MonsterSpawnTable | Spawn rate & difficulty curves | null |

### MonsterSpawnTable Fields

| Field | Type | Description |
|-------|------|-------------|
| `spawnRateCurve` | AnimationCurve | Enemies per second over time |
| `monsterProbabilityCurves` | AnimationCurve[] | Spawn weight for each monster type |
| `hpMultiplierOverTime` | AnimationCurve | HP scaling over time |

### Spawn Mechanics

**Monster Spawn Position:**
- Spawns in a ring around the player
- Ring distance = screen diagonal / 2 + buffer
- Random direction, weighted toward player movement
- Always off-screen for fair gameplay

**Boss Spawn Triggers:**
- Mini-boss spawns when `currentTime >= miniBossContainer.spawnTime`
- Final boss spawns when `currentTime >= levelBlueprint.levelTime`

---

## Testing Your Level

### Quick Test Setup

1. **Temporary Main Menu Modification:**
   - Open `Assets/Scripts/Main Menu/CharacterSelector.cs:47`
   - Change the scene index to your new level's index
   - Example: `SceneManager.LoadScene(2);` for Level 2

2. **Play from Main Menu Scene:**
   - Open `Assets/Scenes/Game/Main Menu.unity`
   - Press Play
   - Select a character to test your level

3. **Revert Changes:**
   - Don't forget to change CharacterSelector back to index 1 when done!

### Using Test Level Blueprint

For rapid iteration:

1. Duplicate your LevelBlueprint
2. Name it `[Your Level Name] Test.asset`
3. Modify for faster testing:
   - Reduce `levelTime` to 60-120 seconds
   - Increase `chestSpawnInterval` to 5 seconds
   - Compress spawn curves to fit shorter duration
   - Lower mini-boss spawn time

### What to Test

- ✅ **Difficulty Curve** - Does it ramp smoothly?
- ✅ **Monster Variety** - Do different monsters appear at the right times?
- ✅ **Spawn Rate** - Is it too easy/hard?
- ✅ **Boss Timing** - Do bosses spawn when expected?
- ✅ **Performance** - Does FPS stay stable with many enemies?
- ✅ **Balance** - Can player survive with basic equipment?
- ✅ **Loot Rate** - Are chests spawning appropriately?

### Debug Tools

**In LevelManager.cs:**
- Current time displayed in UI
- Check `levelTime` variable during play
- Boss spawn messages in console (if logging enabled)

**In EntityManager.cs:**
- Spawned monster count visible in Inspector during play
- Object pool usage stats

---

## Best Practices

### Difficulty Design

1. **Start Easy** - First 1-2 minutes should be comfortable
2. **Gradual Ramp** - Increase difficulty smoothly, avoid sudden spikes
3. **Peak & Taper** - Peak difficulty around 70-80%, then ease off for final boss
4. **HP Scaling** - Use moderate HP multipliers (0x to 2x is good)

### Monster Composition

1. **Variety** - Include 3-5 different monster types
2. **Unlock Progression** - Introduce new types every 1-2 minutes
3. **Balance** - Mix melee, ranged, and special enemies
4. **Late Game** - All monster types active in final third

### Spawn Curves Tips

1. **Smooth Curves** - Avoid jagged progression
2. **Test Extremes** - Play first minute and last minute thoroughly
3. **Enemy Types** - Don't make any monster type too rare (<10%) or dominant (>50%)
4. **Reference Level 1** - Use existing curves as templates

### Performance

1. **Object Pooling** - System handles this, but avoid spawning >50 enemies/second
2. **Projectiles** - Be careful with projectile-heavy monster compositions
3. **Test on Target Hardware** - Performance varies by device

### Playtesting

1. **Multiple Runs** - Play through at least 3 times
2. **Different Characters** - Test with various character builds
3. **Get Feedback** - Have others playtest
4. **Iterate** - Tweak spawn curves based on data

---

## Troubleshooting

### "Monsters aren't spawning"

**Possible Causes:**
- ✅ LevelBlueprint not assigned to LevelManager
- ✅ monstersContainers array is empty
- ✅ spawnRateCurve is flat at 0
- ✅ Monster prefabs missing from containers
- ✅ EntityManager not present in scene

**Solution:** Check all fields in LevelBlueprint are populated.

### "Wrong monsters are spawning"

**Possible Causes:**
- ✅ monsterProbabilityCurves incorrectly configured
- ✅ Wrong MonsterBlueprints assigned to containers
- ✅ Container spawn times too high

**Solution:** Review probability curves and spawn times.

### "Boss doesn't spawn"

**Possible Causes:**
- ✅ bossBlueprint not assigned in container
- ✅ Mini-boss spawnTime > levelTime
- ✅ LevelManager not ticking time correctly

**Solution:** Verify boss container configuration and timing.

### "Level ends immediately"

**Possible Causes:**
- ✅ levelTime set to 0 or very low value
- ✅ Final boss spawning immediately

**Solution:** Set reasonable levelTime (600 for 10 minutes).

### "Performance is poor"

**Possible Causes:**
- ✅ Spawn rate too high (>50 enemies/sec)
- ✅ Too many projectiles on screen
- ✅ Object pools exhausted

**Solution:** Reduce spawn rate, optimize monster composition.

### "Level won't load"

**Possible Causes:**
- ✅ Scene not added to Build Settings
- ✅ Wrong scene index in CharacterSelector
- ✅ Missing required GameObjects in scene

**Solution:** Add scene to Build Settings and verify scene index.

---

## Advanced Topics

### Creating Custom Monster Types

See `docs/HOWTO.md` for information on creating new monster blueprints.

### Multiple Level Selection

Currently, the game is hardcoded to load Level 1 (scene index 1) from CharacterSelector.

To implement level selection:

1. Create a UI for level selection in Main Menu
2. Store selected level index in `CrossSceneData` (similar to character selection)
3. Modify `CharacterSelector.cs:47` to load the selected scene index
4. Add level unlock logic if desired

### Dynamic Background System

Instead of setting a static background texture:

1. Create a BackgroundManager script
2. Read backgroundTexture from LevelBlueprint
3. Apply it to the Background/InfiniteBackground sprite at runtime

---

## Example: Creating "Forest Level"

Let's walk through creating a complete level from scratch.

### 1. Create LevelBlueprint

- Name: `Forest Level.asset`
- Duration: 480 seconds (8 minutes - shorter than Level 1)
- Theme: Forest with denser enemy spawns

### 2. Configure Settings

```
levelTime: 480
backgroundTexture: ForestBackground (create/assign)
initialGemCount: 30 (slightly more generous)
chestSpawnInterval: 25 (more frequent)
```

### 3. Configure Abilities

Copy all 19 abilities from Level 1 for consistency.

### 4. Configure Monsters

```
Container 0 - Melee Pack (spawns at 0s):
  - 初級小兵
  - 中級小兵
  - 高級小兵

Container 1 - Ranged (spawns at 40s):
  - 射擊小兵

Container 2 - Special (spawns at 80s):
  - Gravity Monster
  - Boomerang Monster
```

### 5. Configure Spawn Table

**spawnRateCurve:**
```
0.0 (0%) → 1.5 enemies/sec (start slightly harder)
0.4 (3m) → 7.0 enemies/sec (higher peak)
1.0 (8m) → 3.0 enemies/sec (moderate end)
```

**monsterProbabilityCurves:**
```
Container 0: 1.0 → 0.4 (melee always present, decreases)
Container 1: 0.0 → 0.4 (ranged ramps up from 40s)
Container 2: 0.0 → 0.5 (special becomes dominant late)
```

**hpMultiplierOverTime:**
```
0.0 → 0.0x
0.5 → 1.5x (faster ramp)
1.0 → 2.5x (harder end game)
```

### 6. Configure Bosses

```
Mini Boss:
  spawnTime: 240 (4 minutes)
  bossBlueprint: Boss Walk

Final Boss:
  bossBlueprint: Boss Bullet Hell
```

### 7. Create Scene

1. Duplicate Level 1 → rename to `Forest Level.unity`
2. Assign Forest Level blueprint to LevelManager
3. Add forest background sprite
4. Adjust camera background color to forest green
5. Add decorative tree sprites (optional)

### 8. Test

1. Modify CharacterSelector to load Forest Level scene index
2. Play through multiple times
3. Adjust spawn curves based on difficulty feel
4. Revert CharacterSelector when done

---

## Additional Resources

- **Architecture Overview:** `docs/ARCHITECTURE.md`
- **System Modification Guide:** `docs/HOWTO.md`
- **Example Levels:** `Assets/Blueprints/Levels/Level 1.asset` and `Test Level.asset`
- **Monster Blueprints:** `Assets/Blueprints/Monsters/`
- **Ability Prefabs:** `Assets/Prefabs/Abilities/`

---

## Questions or Issues?

If you encounter problems not covered in this guide, check:

1. Unity Console for error messages
2. LevelManager and EntityManager Inspector values during play mode
3. Scene Hierarchy for missing GameObjects
4. Build Settings for scene registration

Happy level creating! 🎮
