# Quick Start Guide

This guide will help you get the Vampire Survivors Clone project up and running quickly.

## Prerequisites

### Required Software

1. **Unity Hub** (latest version)
2. **Unity Editor 2021.3 LTS or later**
   - Android Build Support (for Android builds)
   - iOS Build Support (for iOS builds)
3. **Git** (for version control)
4. **IDE** (choose one):
   - Visual Studio 2019/2022 (recommended for Windows)
   - Visual Studio Code
   - JetBrains Rider

### System Requirements

- **OS:** Windows 10/11, macOS 10.13+, or Linux
- **RAM:** 8GB minimum (16GB recommended)
- **Storage:** 10GB free space

## Installation

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd VampireSurvivorsClone
```

### Step 2: Open in Unity

1. Launch Unity Hub
2. Click "Open" or "Add"
3. Navigate to the cloned project folder
4. Select the `VampireSurvivorsClone` directory
5. Unity will import all assets (this may take several minutes on first load)

### Step 3: Install Required Packages

Unity should automatically install all required packages from `Packages/manifest.json`:

- Unity Input System
- Unity Localization
- TextMeshPro
- 2D packages

If packages are missing:
1. Open **Window → Package Manager**
2. Install any missing packages listed in the console

### Step 4: Verify Project Setup

1. Open **File → Build Settings**
2. Ensure scenes are added to build:
   - `Assets/Scenes/Game/Main Menu.unity`
   - `Assets/Scenes/Game/Level 1.unity`
3. Select your target platform (PC, Mac, Linux / iOS / Android)
4. Click "Switch Platform" if needed

## Running the Game

### Play Mode in Editor

1. Open the **Main Menu** scene: `Assets/Scenes/Game/Main Menu.unity`
2. Click the Play button (or press Ctrl/Cmd + P)
3. Use the character selection screen to choose a character
4. Click "Start Game" to begin playing

### Controls

#### PC Controls
- **Movement:** WASD or Arrow Keys
- **Pause:** ESC
- **Item Use:** Z, X, C, V (or 1, 2, 3, 4)

#### Mobile Controls
- **Movement:** Left joystick (on-screen)
- **Item Use:** Right side buttons (on-screen)

### Testing Levels Directly

To test Level 1 directly without going through the main menu:

1. Open `Assets/Scenes/Game/Level 1.unity`
2. In the Hierarchy, find the `LevelManager` GameObject
3. In the Inspector, configure the `Level Blueprint` if needed
4. Press Play

**Note:** When testing Level 1 directly, make sure `CrossSceneData.CharacterBlueprint` is set, or the character won't spawn properly.

## Project Structure Overview

```
Assets/
├── Blueprints/         # Configuration data (enemies, characters, levels)
├── Scripts/            # All C# code
├── Scenes/             # Unity scenes
├── Prefabs/            # Reusable game objects
├── Input/              # Input system configuration
├── Localization/       # Translation files
└── Sprites/            # Visual assets
```

## Key Scenes

### 1. Main Menu (`Assets/Scenes/Game/Main Menu.unity`)
- Character selection
- Starting ability configuration
- Entry point for the game

### 2. Level 1 (`Assets/Scenes/Game/Level 1.unity`)
- Main gameplay scene
- 10-minute survival wave-based gameplay
- Contains all game systems

### 3. Character Set Generation (`Assets/Scenes/Character Set Generation/`)
- Editor-only utility scene
- Used for creating character sprite sheets from localized string tables

## Configuration Files

### Character Configuration
**Location:** `Assets/Blueprints/Characters/`

Defines character stats like:
- Health, Armor, Movement Speed
- Starting abilities
- Animation sprites

**Example:** `Main Character.asset`

### Level Configuration
**Location:** `Assets/Blueprints/Levels/`

Defines level settings like:
- Level duration
- Monster spawn tables
- Boss configurations
- Available abilities

**Example:** `Level 1.asset`

### Monster Configuration
**Location:** `Assets/Blueprints/Monsters/`

Defines enemy properties like:
- Health, Speed, Armor
- Attack patterns
- Loot tables
- Sprites and animations

**Examples:** `Melee Monster Level 1.asset`, `Boss Monster.asset`

## Common Issues & Solutions

### Issue: Input not working
**Solution:**
1. Ensure Unity Input System is installed
2. Check **Edit → Project Settings → Player → Active Input Handling** is set to "Input System Package" or "Both"
3. Restart Unity if you just changed this setting

### Issue: Sprites/assets missing
**Solution:**
1. Check the Console for import errors
2. Reimport assets: Right-click Assets folder → Reimport All
3. Ensure TextMeshPro essentials are imported: **Window → TextMeshPro → Import TMP Essential Resources**

### Issue: Localization not working
**Solution:**
1. Open **Window → Asset Management → Localization Tables**
2. Ensure all string tables are properly set up
3. Check **Edit → Project Settings → Localization** for correct locale settings

### Issue: Character not spawning in Level 1
**Solution:**
- When testing Level 1 directly, the character blueprint isn't set
- Either:
  - Start from Main Menu scene
  - Or set a default character in `LevelManager.cs` for testing

### Issue: Build errors
**Solution:**
1. Check **File → Build Settings** has correct scenes
2. Ensure target platform is selected and modules installed
3. Check Console for specific compilation errors

## Next Steps

Now that you have the project running:

1. **Understand the Architecture:** Read [ARCHITECTURE.md](ARCHITECTURE.md) to learn how the code is organized
2. **Make Modifications:** Follow [HOWTO.md](HOWTO.md) for step-by-step guides on modifying core elements
3. **Experiment:** Try changing values in ScriptableObjects to see how they affect gameplay

## Development Workflow

### Recommended Editor Setup

1. **Scene Setup:** Keep both Main Menu and Level 1 scenes in your Hierarchy (but only one active)
2. **Console Filters:** Enable "Collapse" to reduce duplicate log messages
3. **Inspector Lock:** Lock inspector windows when working with prefabs
4. **Layout:** Use a layout with Scene, Game, Console, and Inspector visible

### Testing Tips

1. **Quick Testing:** Test gameplay changes directly in Level 1 scene (remember to set character blueprint)
2. **Full Testing:** Always test complete flow from Main Menu before committing changes
3. **Mobile Testing:** Use Unity Remote or build to device for mobile input testing
4. **Performance:** Use Profiler (**Window → Analysis → Profiler**) to check performance

### Version Control Best Practices

1. **Commit Often:** Small, focused commits are better
2. **Test Before Commit:** Always test your changes in Play mode
3. **Avoid Scene Conflicts:** Communicate with team when editing scenes
4. **Prefab Workflow:** Make changes to prefabs rather than scene instances when possible

## Building the Game

### Build for PC

1. **File → Build Settings**
2. Select **PC, Mac & Linux Standalone**
3. Click **Build** or **Build and Run**
4. Choose output folder

### Build for Mobile

#### Android
1. **File → Build Settings**
2. Select **Android**
3. Configure **Player Settings:**
   - Set Package Name
   - Set minimum API level (21+)
   - Configure keystore for release builds
4. Click **Build** or **Build and Run**

#### iOS
1. **File → Build Settings**
2. Select **iOS**
3. Configure **Player Settings:**
   - Set Bundle Identifier
   - Set Target SDK
4. Click **Build**
5. Open generated Xcode project
6. Build and deploy from Xcode

## Getting Help

- Check the [Main Documentation](DOCUMENTATION.md) for overview
- Read [ARCHITECTURE.md](ARCHITECTURE.md) for code structure details
- Follow [HOWTO.md](HOWTO.md) for specific modification guides
- Check Unity Console for error messages and warnings

Happy developing!
