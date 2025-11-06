# Architecture & Code Flow

This document explains how the Vampire Survivors Clone is structured and how different systems interact with each other.

## Table of Contents

1. [High-Level Architecture](#high-level-architecture)
2. [Scene Flow](#scene-flow)
3. [Core Systems Deep Dive](#core-systems-deep-dive)
4. [Data Flow Examples](#data-flow-examples)
5. [Design Patterns](#design-patterns)
6. [Performance Optimizations](#performance-optimizations)

## High-Level Architecture

### Manager-Based Orchestration Pattern

The game uses a centralized manager pattern where the `LevelManager` orchestrates all major game systems:

```
SCENE HIERARCHY:
┌─────────────────────────────────────────────────────────┐
│ LevelManager (Main Orchestrator)                        │
├─────────────────────────────────────────────────────────┤
│ ├─ EntityManager (Monster spawning, pools, spatial)     │
│ ├─ AbilityManager (Ability selection, upgrades)         │
│ ├─ Character (Player state, health, movement)           │
│ ├─ Inventory (Item management)                          │
│ ├─ StatsManager (Telemetry/stats tracking)              │
│ ├─ InfiniteBackground (Parallax scrolling)              │
│ └─ GameTimer (Elapsed time display)                     │
└─────────────────────────────────────────────────────────┘
```

**Key Files:**
- `Assets/Scripts/Gameplay/LevelManager.cs` - Main game loop orchestrator
- `Assets/Scripts/Monsters/EntityManager.cs` - Monster spawning and pooling
- `Assets/Scripts/Character/Abilities/AbilityManager.cs` - Ability system
- `Assets/Scripts/Character/Character.cs` - Player character

## Scene Flow

### 1. Game Startup Flow

```
Application Start
    ↓
Main Menu Scene Loads
    ↓
MainMenu.Start() Initializes
    ├─ Load available characters from blueprints
    ├─ Display character selection UI
    └─ Setup character cards
    ↓
Player Selects Character
    ↓
CrossSceneData.CharacterBlueprint = selected
    ↓
SceneManager.LoadScene("Level 1")
    ↓
Level 1 Scene Loads
    ↓
LevelManager.Start() Initializes Game
```

**File:** `Assets/Scripts/Main Menu/MainMenu.cs`
```csharp
public void Play()
{
    CrossSceneData.CharacterBlueprint = characterSelector.SelectedCharacter;
    SceneManager.LoadScene("Level 1");
}
```

### 2. Level Initialization Flow

When Level 1 loads, `LevelManager.Start()` executes:

```
LevelManager.Start()
    ↓
Init(levelBlueprint)
    ├─ Initialize EntityManager
    │   ├─ Create all object pools
    │   ├─ Setup SpatialHashGrid
    │   └─ Prepare monster spawn tables
    ├─ Initialize Player Character
    │   ├─ Spawn character at (0,0)
    │   ├─ Load character blueprint stats
    │   ├─ Setup starting abilities
    │   └─ Initialize health/armor/movement
    ├─ Initialize AbilityManager
    │   ├─ Register all available abilities
    │   ├─ Setup upgrade tracking
    │   └─ Grant starting abilities
    ├─ Initialize UI Systems
    │   ├─ Setup health bar
    │   ├─ Setup exp bar
    │   └─ Initialize ability selection dialog
    ├─ Start Game Timer (0 → 600 seconds)
    └─ Begin monster spawning loop
```

**File:** `Assets/Scripts/Gameplay/LevelManager.cs:69`

### 3. Game Loop Flow

The game runs on Unity's Update loop with key systems updating each frame:

```
Every Frame (Update):
    ↓
LevelManager.Update()
    ├─ Update Game Timer
    ├─ Check for boss spawn times
    └─ Trigger boss spawns if needed
    ↓
EntityManager.Update()
    ├─ Rebuild SpatialHashGrid (if needed)
    ├─ Spawn monsters based on time
    ├─ Update all active monsters
    └─ Check spawn queue
    ↓
Character.Update()
    ├─ Read input (WASD/Joystick)
    ├─ Move player
    ├─ Update animations
    └─ Update abilities
    ↓
Ability.Update() (for each ability)
    ├─ Cooldown timer tick
    ├─ Execute ability logic
    │   ├─ Find targets (via SpatialHashGrid)
    │   ├─ Spawn projectiles
    │   └─ Apply effects
    └─ Cleanup expired effects
    ↓
Monster.Update() (for each monster)
    ├─ Find player position
    ├─ Move toward player
    ├─ Update animations
    ├─ Attack if in range
    └─ Execute AI behavior
    ↓
Projectile.Update() (for each projectile)
    ├─ Move in direction
    ├─ Check collisions (SpatialHashGrid)
    ├─ Apply damage on hit
    └─ Destroy/pool when expired
```

## Core Systems Deep Dive

### 1. Player Character System

**Location:** `Assets/Scripts/Character/`

#### Character Class Hierarchy
```
MonoBehaviour
    ↓
Character (abstract)
    ├─ Implements IDamageable
    ├─ Implements ISpatialHashGridClient
    └─ MainCharacter (concrete implementation)
```

#### Character Responsibilities
- **Health Management:** Current HP, max HP, armor, damage reduction
- **Experience System:** Current exp, level, level-up thresholds
- **Movement:** Input handling, velocity, acceleration, animation
- **Combat:** Damage taking, invulnerability frames, death handling
- **Stats:** All upgradeable stats (movement speed, armor, recovery, etc.)

**File:** `Assets/Scripts/Character/Character.cs`

#### Key Methods
```csharp
// Initialize character with dependencies
public void Init(EntityManager entityManager, AbilityManager abilityManager, StatsManager statsManager)

// Add experience points
public void AddExp(int amount)
    → Increments currentExp
    → Checks for level up
    → Triggers LevelUp() if threshold reached

// Handle level up
private void LevelUp()
    → Increments level
    → Recalculates next level exp requirement
    → Triggers ability selection dialog

// Take damage
public void TakeDamage(float damage, Vector2 damageSourcePosition)
    → Calculates damage after armor reduction
    → Applies knockback
    → Updates health
    → Triggers death if health <= 0
```

#### Experience Scaling

**File:** `Assets/Scripts/Character/Character.cs:330`

```csharp
Level  1-9:  BASE + (level * 10)     // e.g., Level 5 = 5 + 50 = 55 exp
Level 10-19: BASE + (level * 13)     // e.g., Level 15 = 5 + 195 = 200 exp
Level 20-29: BASE + (level * 16)     // e.g., Level 25 = 5 + 400 = 405 exp
Level 30+:   BASE + (level * 20)     // e.g., Level 35 = 5 + 700 = 705 exp
```

### 2. Ability System

**Location:** `Assets/Scripts/Character/Abilities/`

#### Ability Class Hierarchy
```
ScriptableObject + MonoBehaviour
    ↓
Ability (abstract base class)
    ├─ Weapon Abilities (automatic)
    │   ├─ MeleeWeaponAbility (Dagger, Slash, Stab, Garlic)
    │   ├─ RangedWeaponAbility (Gun, MachineGun, Shuriken)
    │   └─ SpecialWeaponAbility (Boomerang, Grenade, Molotov)
    └─ Upgrade Abilities (passive)
        ├─ Damage, DamageRate, Cooldown
        ├─ Area, Pierce, Projectiles
        ├─ MovementSpeed, Armor, MaxHealth
        └─ Knockback, Luck, Recovery
```

#### Ability Lifecycle

```
Ability Selected (from level-up dialog)
    ↓
Ability.Select() called
    ↓
If New Ability:
    ├─ owned = true
    ├─ Use() - Activate ability
    └─ If weapon: Start cooldown loop
    ↓
If Already Owned:
    ├─ Upgrade() - Increase stats
    └─ Update IUpgradeableValue fields
```

**File:** `Assets/Scripts/Character/Abilities/Ability.cs:85`

#### Ability Manager - Selection System

**File:** `Assets/Scripts/Character/Abilities/AbilityManager.cs:109`

The `AbilityManager` handles ability selection on level-up:

```csharp
public void SelectAbilities(int count = 3)
{
    // 1. Separate owned vs. new abilities
    List<Ability> ownedAbilities = abilities.Where(a => a.owned).ToList();
    List<Ability> newAbilities = abilities.Where(a => !a.owned && a.RequirementsMet()).ToList();

    // 2. Calculate weighted probabilities
    float ownedWeight = ownedAbilities.Count > 0 ? NEW_ABILITY_WEIGHT : 0;
    float newWeight = newAbilities.Count > 0 ? OWNED_ABILITY_WEIGHT : 0;

    // 3. Select 3-4 random abilities (weighted toward new abilities)
    List<Ability> selected = new List<Ability>();
    for (int i = 0; i < count; i++)
    {
        if (Random.value < newWeight / (ownedWeight + newWeight))
            selected.Add(newAbilities.RandomElement());
        else
            selected.Add(ownedAbilities.RandomElement());
    }

    // 4. Show selection dialog to player
    abilitySelectionDialog.Show(selected);
}
```

#### Upgrade System - Reflection-Based

The ability system uses reflection to automatically discover and upgrade stat fields:

**File:** `Assets/Scripts/Character/Abilities/AbilityManager.cs:198`

```csharp
// At initialization, scan all abilities for upgradeable fields
private void RegisterUpgradeableValues()
{
    foreach (Ability ability in abilities)
    {
        FieldInfo[] fields = ability.GetType().GetFields();
        foreach (FieldInfo field in fields)
        {
            if (typeof(IUpgradeableValue).IsAssignableFrom(field.FieldType))
            {
                IUpgradeableValue value = (IUpgradeableValue)field.GetValue(ability);
                RegisterUpgradeableValue(value);
            }
        }
    }
}

// When upgrade ability is selected, apply to all matching types
public void UpgradeValue<T, V>(float amount) where T : UpgradeableValue<V>
{
    foreach (IUpgradeableValue value in upgradeableValues)
    {
        if (value is T upgradeableValue)
        {
            upgradeableValue.Upgrade(amount);
        }
    }
}
```

**Example:** When "Damage Upgrade" ability is selected:
1. `DamageUpgrade.Select()` called
2. Calls `abilityManager.UpgradeValue<Damage, float>(0.1f)` (10% increase)
3. All `Damage` fields in all abilities get upgraded by 10%
4. Gun, Dagger, MachineGun, etc. all deal 10% more damage

### 3. Monster System

**Location:** `Assets/Scripts/Monsters/`

#### Monster Types

```
Monster (base class)
    ├─ MeleeMonster - Rushes player for contact damage
    ├─ RangedMonster - Shoots projectiles from distance
    ├─ ThrowingMonster - Lobs grenades at player
    ├─ BoomerangMonster - Throws returning weapons
    └─ BossMonster - Complex AI with multiple attack patterns
```

#### EntityManager - Monster Spawning

**File:** `Assets/Scripts/Monsters/EntityManager.cs`

The `EntityManager` is the largest and most complex system (~900 lines), handling:
- Monster spawning
- Object pooling
- Spatial hash grid management
- Monster AI updates

```csharp
// Monster Spawn Loop (called every frame)
private void SpawnMonsters()
{
    // 1. Calculate time percentage through level
    float timePercent = levelManager.ElapsedTime / levelManager.LevelTime;

    // 2. Evaluate spawn rate curve
    float spawnRate = monsterSpawnTable.spawnRateCurve.Evaluate(timePercent);

    // 3. Accumulate spawn timer
    spawnTimer += Time.deltaTime * spawnRate;

    // 4. Spawn monsters when timer threshold reached
    while (spawnTimer >= 1f)
    {
        SpawnRandomMonster();
        spawnTimer -= 1f;
    }
}

// Spawn individual monster
private void SpawnRandomMonster()
{
    // 1. Select monster type from weighted table
    MonsterBlueprint blueprint = monsterSpawnTable.GetRandomMonster(timePercent);

    // 2. Get monster from pool
    Monster monster = monsterPool.Get();

    // 3. Apply HP scaling for difficulty progression
    float hpMultiplier = monsterSpawnTable.hpMultiplierCurve.Evaluate(timePercent);

    // 4. Initialize monster
    monster.Init(blueprint, hpMultiplier);

    // 5. Spawn at random position in ring around player
    Vector2 spawnPos = GetSpawnPosition();
    monster.transform.position = spawnPos;

    // 6. Add to spatial hash grid
    spatialHashGrid.Add(monster);
}
```

**File:** `Assets/Scripts/Monsters/EntityManager.cs:456`

#### Boss Monster System

Bosses have special attack patterns defined in `Assets/Scripts/Monsters/Boss Abilities/`:

```
BossAbility (abstract base)
    ├─ BulletHellBossAbility - Spray attack (many projectiles)
    ├─ ChargeBossAbility - Ram attack (fast movement toward player)
    ├─ GrenadeBossAbility - AoE explosive attack
    ├─ ShotgunBossAbility - Spread shot pattern
    └─ WalkBossAbility - Basic movement
```

Bosses spawn at specific times configured in `LevelBlueprint`:
- **Mini-bosses:** Configured spawn times (e.g., 3 minutes, 6 minutes)
- **Final boss:** Spawns near level end (e.g., 9 minutes)

**File:** `Assets/Scripts/Gameplay/LevelManager.cs:141`

### 4. Object Pooling System

**Location:** `Assets/Scripts/Gameplay/Pools/`

Object pooling prevents garbage collection spikes by reusing objects instead of destroying/instantiating.

#### Pool Architecture

```csharp
public abstract class Pool<T> where T : MonoBehaviour
{
    protected Queue<T> availableObjects;
    protected HashSet<T> activeObjects;

    // Get object from pool (or create new if empty)
    public T Get()
    {
        T obj;
        if (availableObjects.Count > 0)
            obj = availableObjects.Dequeue();
        else
            obj = CreateNew();

        activeObjects.Add(obj);
        obj.gameObject.SetActive(true);
        return obj;
    }

    // Return object to pool
    public void Return(T obj)
    {
        obj.gameObject.SetActive(false);
        activeObjects.Remove(obj);
        availableObjects.Enqueue(obj);
    }
}
```

**File:** `Assets/Scripts/Gameplay/Pools/Pool.cs`

#### Pool Types

| Pool | Purpose | Typical Size |
|------|---------|--------------|
| MonsterPool | Enemies | 100-500 |
| ProjectilePool | Bullets | 200-1000 |
| ExplosiveProjectilePool | Explosive bullets | 50-200 |
| BoomerangPool | Returning weapons | 20-50 |
| ThrowablePool | Grenades | 20-50 |
| ExpGemPool | Experience drops | 500-2000 |
| CoinPool | Currency drops | 100-500 |
| ChestPool | Loot containers | 10-20 |
| DamageTextPool | Floating damage numbers | 50-100 |

### 5. Spatial Collision System

**Location:** `Assets/Scripts/Spatial/SpatialHashGrid.cs`

The spatial hash grid provides efficient broad-phase collision detection by partitioning the game world into a grid.

#### How It Works

```
Game World divided into grid cells:
┌─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  D  │
├─────┼─────┼─────┼─────┤
│  E  │  F  │  G  │  H  │  Cell size: 5x5 units
├─────┼─────┼─────┼─────┤
│  I  │  J  │  K  │  L  │  Grid size: 50x50 units
├─────┼─────┼─────┼─────┤
│  M  │  N  │  O  │  P  │
└─────┴─────┴─────┴─────┘

Objects stored in cells based on position.
Range queries only check nearby cells.
```

**File:** `Assets/Scripts/Spatial/SpatialHashGrid.cs:45`

#### Usage Example

```csharp
// Find all enemies within 10 units of player for area attack
List<ISpatialHashGridClient> targets = spatialHashGrid.GetNearby(
    playerPosition,
    radius: 10f
);

foreach (var target in targets)
{
    if (target is Monster monster)
    {
        monster.TakeDamage(damage, playerPosition);
    }
}
```

#### Dynamic Rebuilding

The grid automatically rebuilds when the player moves far from the grid center:

```csharp
public void Update()
{
    Vector2 playerPos = character.transform.position;

    // Rebuild if player is near grid edge
    if (Vector2.Distance(playerPos, gridCenter) > rebuildThreshold)
    {
        Rebuild(playerPos);
    }
}
```

This ensures the grid is always centered on the player, regardless of how far they move.

### 6. Progression System

**Location:** `Assets/Scripts/Gameplay/LevelManager.cs`, `Assets/Scripts/UI/AbilitySelectionDialog.cs`

#### Level-Up Flow

```
Player Collects ExpGem
    ↓
Character.AddExp(amount)
    ├─ currentExp += amount
    └─ Check if currentExp >= nextLevelExp
    ↓
Character.LevelUp()
    ├─ currentLevel++
    ├─ currentExp -= nextLevelExp
    ├─ Calculate new nextLevelExp
    ├─ Trigger level-up effects (visual/audio)
    └─ Call abilityManager.SelectAbilities()
    ↓
AbilityManager.SelectAbilities()
    ├─ Select 3-4 random abilities (weighted)
    ├─ Filter by requirements
    └─ Show AbilitySelectionDialog
    ↓
Player Chooses Ability
    ↓
Ability.Select()
    ├─ If new: Ability.Use()
    └─ If owned: Ability.Upgrade()
    ↓
Game Resumes
```

## Data Flow Examples

### Example 1: Player Damages Monster

```
1. Projectile.Update() detects collision via SpatialHashGrid
    ↓
2. Projectile calls monster.TakeDamage(damage, position)
    ↓
3. Monster.TakeDamage()
    ├─ Calculate damage after armor: finalDamage = damage / (1 + armor)
    ├─ Apply knockback: AddForce(awayFromSource)
    ├─ Reduce HP: currentHp -= finalDamage
    ├─ Spawn damage text: damageTextPool.Get().Show(finalDamage)
    └─ Check death: if (currentHp <= 0) Die()
    ↓
4. If died: Monster.Die()
    ├─ Trigger death animation
    ├─ Spawn loot (experience gems, coins)
    │   ├─ Roll luck table for drop rates
    │   ├─ expGemPool.Get() for each gem
    │   └─ Position gems around monster
    ├─ Call statsManager.OnMonsterKilled()
    └─ Return monster to pool: monsterPool.Return(this)
```

**Files:**
- `Assets/Scripts/Projectiles/Projectile.cs:124` - Collision detection
- `Assets/Scripts/Monsters/Monster.cs:189` - TakeDamage
- `Assets/Scripts/Monsters/Monster.cs:243` - Die and loot spawning

### Example 2: Player Levels Up and Chooses Ability

```
1. Player walks over ExpGem
    ↓
2. ExpGem.OnTriggerEnter2D(player)
    ↓
3. ExpGem.Collect()
    ├─ Play collection sound/animation
    ├─ character.AddExp(value)
    └─ Return to pool: expGemPool.Return(this)
    ↓
4. Character.AddExp(amount)
    ├─ currentExp += amount
    ├─ Update exp bar UI
    └─ if (currentExp >= nextLevelExp) LevelUp()
    ↓
5. Character.LevelUp()
    ├─ currentLevel++
    ├─ currentExp -= nextLevelExp
    ├─ nextLevelExp = CalculateNextLevelExp(currentLevel)
    ├─ Play level-up effects (sound, particle)
    ├─ Pause game: Time.timeScale = 0
    └─ abilityManager.SelectAbilities(3)
    ↓
6. AbilityManager.SelectAbilities(count)
    ├─ Separate owned vs new abilities
    ├─ Filter by RequirementsMet()
    ├─ Select 3 random abilities (weighted toward new)
    └─ abilitySelectionDialog.Show(selectedAbilities)
    ↓
7. AbilitySelectionDialog.Show(abilities)
    ├─ Display ability cards
    ├─ Show ability stats (damage, cooldown, etc.)
    └─ Wait for player input
    ↓
8. Player clicks ability card
    ↓
9. AbilitySelectionDialog.OnAbilitySelected(ability)
    ├─ Hide dialog
    ├─ Resume game: Time.timeScale = 1
    └─ ability.Select()
    ↓
10. Ability.Select()
     ├─ owned = true
     ├─ if (is new weapon) Use() - start attack loop
     └─ if (is upgrade) Upgrade() - increase stats
```

**Files:**
- `Assets/Scripts/Collectables/ExpGem.cs:67` - Collection
- `Assets/Scripts/Character/Character.cs:273` - AddExp
- `Assets/Scripts/Character/Character.cs:293` - LevelUp
- `Assets/Scripts/Character/Abilities/AbilityManager.cs:109` - SelectAbilities
- `Assets/Scripts/UI/AbilitySelectionDialog.cs:42` - Show dialog

### Example 3: Monster Spawn and Attack

```
1. LevelManager.Update()
    ├─ Update elapsed time
    └─ entityManager.Update()
    ↓
2. EntityManager.Update()
    ├─ Check spawn timer
    ├─ spawnTimer += deltaTime * spawnRate
    └─ if (spawnTimer >= 1) SpawnMonster()
    ↓
3. EntityManager.SpawnMonster()
    ├─ Evaluate spawn table based on time %
    ├─ Select random monster type from weighted table
    ├─ Get monster from pool: monsterPool.Get()
    ├─ Calculate HP multiplier for difficulty
    ├─ monster.Init(blueprint, hpMultiplier)
    ├─ Position at spawn point (ring around player)
    └─ spatialHashGrid.Add(monster)
    ↓
4. Monster.Update() (every frame)
    ├─ Find player position
    ├─ Calculate direction toward player
    ├─ Move: position += direction * speed * deltaTime
    ├─ Update animation based on movement
    ├─ if (is in attack range) Attack()
    └─ spatialHashGrid.UpdatePosition(this)
    ↓
5. Monster.Attack() (for RangedMonster example)
    ├─ if (cooldown > 0) return  // Still on cooldown
    ├─ Aim at player predicted position
    ├─ Spawn projectile: projectilePool.Get()
    ├─ projectile.Init(direction, speed, damage)
    ├─ Reset cooldown timer
    └─ Play attack animation/sound
    ↓
6. Monster Projectile.Update()
    ├─ Move in direction
    ├─ Check collisions via SpatialHashGrid
    ├─ if (hit player) player.TakeDamage(damage, position)
    └─ if (expired or hit) projectilePool.Return(this)
```

**Files:**
- `Assets/Scripts/Gameplay/LevelManager.cs:110` - Update loop
- `Assets/Scripts/Monsters/EntityManager.cs:281` - Spawn logic
- `Assets/Scripts/Monsters/Monster.cs:98` - Update and movement
- `Assets/Scripts/Monsters/RangedMonster.cs:45` - Attack logic

## Design Patterns

### 1. Dependency Injection Pattern

Dependencies are passed via `Init()` methods rather than finding them with `FindObjectOfType()`:

```csharp
// Good: Explicit dependency injection
public void Init(EntityManager entityManager, AbilityManager abilityManager)
{
    this.entityManager = entityManager;
    this.abilityManager = abilityManager;
}

// Bad: Implicit dependency lookup (avoided)
private void Start()
{
    entityManager = FindObjectOfType<EntityManager>(); // Don't do this!
}
```

**Benefits:**
- Testable (can inject mocks)
- Explicit dependencies (clear what class needs)
- No runtime lookups (faster)

### 2. Data-Driven Design

All balance values are stored in ScriptableObjects, not hardcoded:

```csharp
// Good: Data-driven
[SerializeField] private CharacterBlueprint blueprint;
private float maxHealth => blueprint.maxHealth;

// Bad: Hardcoded (avoided)
private float maxHealth = 100f; // Don't do this!
```

**Benefits:**
- Easy to balance without code changes
- Can create variants without duplication
- Designers can modify without programmer help

### 3. Object Pooling Pattern

Frequently spawned objects are pooled and reused:

```csharp
// Get from pool
Monster monster = monsterPool.Get();
monster.Init(blueprint);

// Return to pool when done
monsterPool.Return(monster);
```

**Benefits:**
- Reduces garbage collection
- Consistent performance
- No allocation spikes

### 4. Component-Based Entity Pattern

Entities are built from composable MonoBehaviour components:

```
GameObject "Monster"
    ├─ Monster (behavior)
    ├─ SpriteRenderer (visual)
    ├─ Collider2D (collision)
    └─ SpriteAnimator (animation)
```

**Benefits:**
- Reusable components
- Unity-friendly
- Inspector editing

### 5. Interface Segregation

Small, focused interfaces for specific capabilities:

```csharp
public interface IDamageable
{
    void TakeDamage(float damage, Vector2 sourcePosition);
}

public interface ISpatialHashGridClient
{
    Vector2 GetPosition();
    void OnSpatialHashGridRemove();
}
```

**Benefits:**
- Loose coupling
- Clear contracts
- Easy to implement

### 6. Strategy Pattern (for Boss Abilities)

Boss behaviors are swappable strategies:

```csharp
public abstract class BossAbility : ScriptableObject
{
    public abstract void Execute(BossMonster boss);
}

// Boss can have multiple abilities and switch between them
public class BossMonster : Monster
{
    [SerializeField] private List<BossAbility> abilities;

    private void PerformRandomAbility()
    {
        BossAbility ability = abilities.RandomElement();
        ability.Execute(this);
    }
}
```

**Benefits:**
- Easy to add new boss abilities
- Reusable between bosses
- Data-driven boss behavior

## Performance Optimizations

### 1. Object Pooling
- All frequently spawned objects use pools
- Prevents GC allocation spikes
- Consistent frame times

### 2. Spatial Hash Grid
- O(1) cell lookup instead of O(n) distance checks
- Only checks nearby cells for collisions
- Dramatically reduces collision checks

### 3. FastList<T>
Custom list implementation for GC-efficient iteration:

**File:** `Assets/Scripts/Utilities/FastList.cs`

```csharp
// Standard List creates garbage when iterating
foreach (var item in standardList) { } // Allocates enumerator

// FastList uses index-based iteration (no allocation)
for (int i = 0; i < fastList.Count; i++)
{
    var item = fastList[i]; // No garbage
}
```

### 4. Reflection Caching
Upgradeable fields are scanned once at initialization, not every frame:

```csharp
// Init phase: Cache all upgradeable fields
private void CacheUpgradeableFields()
{
    foreach (Ability ability in abilities)
    {
        FieldInfo[] fields = ability.GetType().GetFields();
        foreach (FieldInfo field in fields)
        {
            if (typeof(IUpgradeableValue).IsAssignableFrom(field.FieldType))
            {
                cachedUpgradeableFields.Add(field);
            }
        }
    }
}
```

### 5. Efficient Monster Updates
Monsters only update when active and on-screen (or nearby):

```csharp
private void Update()
{
    // Only update if monster is active
    if (!isActive) return;

    // Only update if near player (in spatial grid)
    float distToPlayer = Vector2.Distance(transform.position, playerPos);
    if (distToPlayer > maxUpdateDistance) return;

    // Perform update logic
    UpdateMovement();
    UpdateAnimation();
}
```

### 6. Batch Operations
Monsters spawn in batches rather than one per frame:

```csharp
while (spawnTimer >= 1f)
{
    SpawnMonster();
    spawnTimer -= 1f;
}
```

### 7. Coroutine Queues
Serializable coroutine execution for timed effects:

**File:** `Assets/Scripts/Utilities/CoroutineQueue.cs`

Allows pausing/resuming coroutines efficiently.

## Summary

The Vampire Survivors Clone uses a well-structured architecture with:
- **Manager Pattern** for system orchestration
- **Data-Driven Design** for easy balancing
- **Object Pooling** for performance
- **Spatial Partitioning** for efficient collision
- **Reflection-Based Upgrades** for flexible progression
- **Component-Based Entities** for reusability

Understanding these systems and their interactions will help you navigate and modify the codebase effectively. For specific modification guides, see [HOWTO.md](HOWTO.md).
