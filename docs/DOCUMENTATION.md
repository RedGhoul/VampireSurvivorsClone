# Vampire Survivors Clone - Developer Documentation

Welcome to the Vampire Survivors Clone documentation! This guide will help you understand the codebase structure, game architecture, and how to modify core elements.

## Table of Contents

1. [Quick Start Guide](QUICKSTART.md) - Get up and running quickly
2. [Architecture & Code Flow](ARCHITECTURE.md) - Understand how the game works
3. [How-To Guides](HOWTO.md) - Modify core game elements

## Project Overview

This is a Unity-based clone of Vampire Survivors, a wave-based survival roguelike game. The player must survive for 10 minutes against increasingly difficult waves of enemies while collecting experience, leveling up, and gaining new abilities.

### Key Technologies

- **Engine:** Unity 2021.3+
- **Language:** C#
- **Target Platforms:** PC, iOS, Android
- **Key Packages:**
  - Unity Input System (mobile and PC support)
  - Unity Localization (multi-language support)
  - TextMeshPro (text rendering)

### Project Statistics

- **140+ C# scripts**
- **~15,754 lines of code**
- **40+ unique abilities/weapons**
- **Multiple enemy types and bosses**
- **Data-driven design using ScriptableObjects**

## Core Game Systems

### 1. Player Character System
The player character with health, armor, movement, and leveling mechanics.
- **Location:** `Assets/Scripts/Character/`
- **Key Classes:** `Character.cs`, `MainCharacter.cs`

### 2. Ability/Weapon System
40+ weapons and upgrades with automatic and manual activation.
- **Location:** `Assets/Scripts/Character/Abilities/`
- **Key Classes:** `Ability.cs`, `AbilityManager.cs`

### 3. Monster/Enemy System
Various enemy types with different behaviors and attack patterns.
- **Location:** `Assets/Scripts/Monsters/`
- **Key Classes:** `Monster.cs`, `EntityManager.cs`, `BossMonster.cs`

### 4. Object Pooling System
High-performance object recycling for enemies, projectiles, and collectibles.
- **Location:** `Assets/Scripts/Gameplay/Pools/`
- **Key Classes:** `Pool.cs` and specialized pool types

### 5. Progression System
Experience collection, leveling, and ability selection.
- **Location:** `Assets/Scripts/Gameplay/`
- **Key Classes:** `LevelManager.cs`, `AbilitySelectionDialog.cs`

### 6. Spatial Collision System
Efficient broad-phase collision detection using spatial hash grid.
- **Location:** `Assets/Scripts/Spatial/`
- **Key Classes:** `SpatialHashGrid.cs`

## Directory Structure

```
Assets/
├── Blueprints/              # ScriptableObject data assets
│   ├── Characters/          # Character definitions
│   ├── Levels/              # Level configurations
│   ├── Monsters/            # Enemy blueprints
│   ├── Chests/              # Chest loot configurations
│   └── CollectableTypes/    # Loot types
├── Scripts/                 # Main gameplay code
│   ├── Character/           # Player character system
│   │   └── Abilities/       # Weapons and upgrades
│   ├── Monsters/            # Enemy AI and behavior
│   ├── Gameplay/            # Core game systems
│   ├── Projectiles/         # Weapon projectile types
│   ├── Collectables/        # Item pickup system
│   ├── UI/                  # Menu and dialog systems
│   ├── Spatial/             # Collision detection
│   └── Utilities/           # Helper classes
├── Scenes/                  # Unity scene files
│   └── Game/                # Main Menu and Level scenes
├── Prefabs/                 # Reusable components
├── Input/                   # Input action definitions
└── Localization/            # Translation files
```

## Game Flow Overview

```
Main Menu → Character Selection → Level Start
    ↓
Player Movement + Auto-attacks
    ↓
Enemies Spawn in Waves
    ↓
Collect Experience & Items
    ↓
Level Up → Choose Ability/Upgrade
    ↓
Bosses Spawn at Key Times
    ↓
Game End (Death or 10 Minutes Survived)
```

## Getting Help

- Check the [Quick Start Guide](QUICKSTART.md) for setup instructions
- Read the [Architecture Guide](ARCHITECTURE.md) to understand code flow
- Follow the [How-To Guides](HOWTO.md) to make specific modifications
- Review inline code comments for implementation details

## Contributing

When modifying the codebase:
1. Follow the existing naming conventions
2. Use ScriptableObjects for configuration data
3. Implement object pooling for frequently spawned objects
4. Add localization keys for user-facing text
5. Test on both PC and mobile platforms

## Code Conventions

- **Namespace:** All code uses the `Vampire` namespace
- **Manager Pattern:** Core systems use Manager classes (LevelManager, EntityManager, etc.)
- **Data-Driven:** Balance values in ScriptableObjects, not hardcoded
- **Interface Usage:** IDamageable, ISpatialHashGridClient, IUpgradeableValue
- **Dependency Injection:** Init() methods for passing dependencies

Happy coding!
