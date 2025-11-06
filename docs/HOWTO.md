# How-To Guides

Step-by-step guides for modifying core elements of the Vampire Survivors Clone.

## Table of Contents

1. [Adding a New Character](#adding-a-new-character)
2. [Creating a New Weapon/Ability](#creating-a-new-weaponability)
3. [Adding a New Enemy Type](#adding-a-new-enemy-type)
4. [Creating a New Boss](#creating-a-new-boss)
5. [Modifying Level Settings](#modifying-level-settings)
6. [Adding a New Collectible](#adding-a-new-collectible)
7. [Creating Passive Upgrades](#creating-passive-upgrades)
8. [Adjusting Difficulty Scaling](#adjusting-difficulty-scaling)
9. [Adding Localization](#adding-localization)
10. [Creating a New Level](#creating-a-new-level)

---

## Adding a New Character

Characters define player stats, appearance, and starting abilities.

### Step 1: Create the Character Blueprint

1. Right-click in `Assets/Blueprints/Characters/`
2. Select **Create → Vampire → Character Blueprint**
3. Name it (e.g., "Warrior Character")

### Step 2: Configure Character Stats

Select the new blueprint and configure in Inspector:

```
Name: "Warrior"
Max HP: 150 (higher than default 100)
Armor: 10 (some starting armor)
Movement Speed: 4.5 (slightly slower than default 5)
Acceleration: 20
Luck: 1.0 (affects drop rates)
Recovery: 1.0 (health regen per second)
```

### Step 3: Set Character Sprites

**Animation Sprites:**
1. Assign sprites for walk cycle:
   - `Walk Right Sprites` - Array of sprites for walking right
   - `Walk Left Sprites` - Array for walking left
   - `Walk Up Sprites` - Array for walking up
   - `Walk Down Sprites` - Array for walking down

2. Set animation timing:
   - `Seconds Per Frame` - How fast to cycle through sprites (e.g., 0.1)

**Tip:** Use the Character Set Generator scene to create sprite sheets from individual sprites.

### Step 4: Assign Starting Abilities

1. In the blueprint, find `Starting Abilities` array
2. Set size to number of starting abilities (e.g., 2)
3. Drag ability prefabs from `Assets/Prefabs/Abilities/`:
   - Example: `Dagger`, `Garlic`, `Gun`

### Step 5: Add to Character Selection

1. Open `Assets/Scenes/Game/Main Menu.unity`
2. Find the `CharacterSelector` GameObject
3. In Inspector, find `Available Characters` array
4. Increase size by 1
5. Drag your new blueprint into the new slot

### Step 6: Test

1. Play the Main Menu scene
2. Your character should appear in selection
3. Select it and start a game
4. Verify stats and starting abilities work correctly

**Example Blueprint:** `Assets/Blueprints/Characters/Main Character.asset`

---

## Creating a New Weapon/Ability

Abilities can be weapons (active) or upgrades (passive).

### Creating a Ranged Weapon

Let's create a "Crossbow" weapon that shoots bolts.

#### Step 1: Create the Ability Script

Create new file: `Assets/Scripts/Character/Abilities/Crossbow.cs`

```csharp
using UnityEngine;
using Vampire.Projectiles;
using Vampire.Utilities;

namespace Vampire.Character.Abilities
{
    [CreateAssetMenu(menuName = "Vampire/Abilities/Crossbow")]
    public class Crossbow : Ability
    {
        [Header("Crossbow Settings")]
        [SerializeField] private Projectile projectilePrefab;

        // Upgradeable stats
        public Damage damage = new Damage(20f); // Base damage
        public Cooldown cooldown = new Cooldown(2f); // Seconds between shots
        public ProjectileSpeed projectileSpeed = new ProjectileSpeed(15f);
        public Pierce pierce = new Pierce(2); // Number of enemies to pierce

        private float cooldownTimer;

        public override void Use()
        {
            // Called when ability is first acquired
            cooldownTimer = 0f;
        }

        public override void UpdateAbility()
        {
            // Called every frame
            cooldownTimer -= Time.deltaTime;

            if (cooldownTimer <= 0f)
            {
                Fire();
                cooldownTimer = cooldown.Value;
            }
        }

        private void Fire()
        {
            // Find nearest enemy
            var nearbyEnemies = entityManager.SpatialHashGrid.GetNearby(
                character.transform.position,
                20f // Search radius
            );

            Monster nearestEnemy = null;
            float nearestDistance = float.MaxValue;

            foreach (var entity in nearbyEnemies)
            {
                if (entity is Monster monster)
                {
                    float dist = Vector2.Distance(
                        character.transform.position,
                        monster.transform.position
                    );
                    if (dist < nearestDistance)
                    {
                        nearestDistance = dist;
                        nearestEnemy = monster;
                    }
                }
            }

            if (nearestEnemy == null) return;

            // Calculate direction
            Vector2 direction = (nearestEnemy.transform.position - character.transform.position).normalized;

            // Spawn projectile
            var projectile = entityManager.ProjectilePool.Get();
            projectile.transform.position = character.transform.position;
            projectile.Init(
                direction,
                projectileSpeed.Value,
                damage.Value,
                pierce.Value,
                character
            );
        }

        public override string GetDescription()
        {
            return $"Shoots bolts at nearest enemy\nDamage: {damage.Value}\nCooldown: {cooldown.Value}s";
        }
    }
}
```

#### Step 2: Create the Ability Asset

1. Right-click in `Assets/Blueprints/Abilities/` (create folder if needed)
2. Select **Create → Vampire → Abilities → Crossbow**
3. Name it "Crossbow"

#### Step 3: Configure the Ability

Select the asset and configure:

```
Name: "Crossbow"
Icon: (Assign a sprite for the ability icon)
Rarity: Common (or Rare, Epic, Legendary)
Description: "Shoots powerful bolts"
Max Level: 5
Owned: False (will be set when player gets it)

Projectile Prefab: (Drag a projectile prefab from Assets/Prefabs/Projectiles/)
Damage: 20
Cooldown: 2
Projectile Speed: 15
Pierce: 2
```

#### Step 4: Add to Level's Available Abilities

1. Open the level blueprint: `Assets/Blueprints/Levels/Level 1.asset`
2. Find `Ability Prefabs` array
3. Increase size by 1
4. Drag your Crossbow ability into the new slot

#### Step 5: Create Prefab (Optional but Recommended)

1. Create an empty GameObject in the scene
2. Add your Crossbow script as a component
3. Configure the serialized fields
4. Drag to `Assets/Prefabs/Abilities/` to create prefab
5. Delete from scene
6. Reference this prefab in the level blueprint

#### Step 6: Test

1. Play the game
2. Level up
3. Your Crossbow should appear as an option
4. Select it and verify it fires correctly

### Creating a Passive Upgrade

Let's create a "Critical Hit Chance" upgrade.

#### Step 1: Create Upgrade Value Type

Add to `Assets/Scripts/Utilities/UpgradeableValues.cs`:

```csharp
[System.Serializable]
public class CriticalChance : UpgradeableFloat
{
    public CriticalChance(float value) : base(value) { }

    public override string GetDescription()
    {
        return $"Critical Chance: {Value * 100f:F0}%";
    }
}
```

#### Step 2: Create Upgrade Ability

Create file: `Assets/Scripts/Character/Abilities/CriticalChanceUpgrade.cs`

```csharp
using UnityEngine;

namespace Vampire.Character.Abilities
{
    [CreateAssetMenu(menuName = "Vampire/Abilities/Critical Chance Upgrade")]
    public class CriticalChanceUpgrade : Ability
    {
        [SerializeField] private float upgradeAmount = 0.05f; // +5% per level

        public override void Select()
        {
            owned = true;
            Upgrade();
        }

        public override void Upgrade()
        {
            // Apply upgrade to all CriticalChance values in all abilities
            abilityManager.UpgradeValue<CriticalChance, float>(upgradeAmount);
        }

        public override string GetDescription()
        {
            return $"Increase critical hit chance by {upgradeAmount * 100f:F0}%";
        }
    }
}
```

#### Step 3: Add Critical System to Weapons

Modify existing weapon abilities to use critical hits:

```csharp
public class Gun : Ability
{
    public Damage damage = new Damage(10f);
    public CriticalChance criticalChance = new CriticalChance(0.05f); // 5% base
    public CriticalMultiplier criticalMultiplier = new CriticalMultiplier(2f); // 2x damage

    private void Fire()
    {
        float finalDamage = damage.Value;

        // Roll for critical hit
        if (Random.value < criticalChance.Value)
        {
            finalDamage *= criticalMultiplier.Value;
            // Show critical hit effect
            ShowCriticalEffect();
        }

        // ... rest of fire logic
    }
}
```

#### Step 4: Create and Configure Asset

1. Create the ability asset as before
2. Add to level's available abilities
3. Test by leveling up and selecting the upgrade

---

## Adding a New Enemy Type

Let's create a "Flying Enemy" that moves differently.

### Step 1: Create Enemy Script

Create file: `Assets/Scripts/Monsters/FlyingMonster.cs`

```csharp
using UnityEngine;

namespace Vampire.Monsters
{
    public class FlyingMonster : Monster
    {
        [Header("Flying Behavior")]
        [SerializeField] private float hoverHeight = 2f;
        [SerializeField] private float bobSpeed = 2f;
        [SerializeField] private float bobAmount = 0.3f;

        private float bobTimer;

        protected override void UpdateMovement()
        {
            if (character == null) return;

            // Move toward player
            Vector2 targetPosition = character.transform.position;
            Vector2 currentPosition = transform.position;
            Vector2 direction = (targetPosition - currentPosition).normalized;

            // Apply movement
            Vector2 newPosition = currentPosition + direction * MovementSpeed * Time.deltaTime;

            // Add bobbing motion
            bobTimer += Time.deltaTime * bobSpeed;
            float bobOffset = Mathf.Sin(bobTimer) * bobAmount;
            newPosition.y += bobOffset;

            transform.position = newPosition;
        }

        protected override void UpdateAnimation()
        {
            // Flying enemies might use single sprite or gentle rotation
            float angle = Mathf.Sin(bobTimer) * 10f; // Tilt slightly
            transform.rotation = Quaternion.Euler(0, 0, angle);
        }
    }
}
```

### Step 2: Create Enemy Prefab

1. Create new GameObject in scene: "Flying Monster"
2. Add components:
   - `FlyingMonster` script
   - `SpriteRenderer` (assign flying enemy sprite)
   - `CircleCollider2D` (set radius, make it a trigger)
   - `Rigidbody2D` (set to Kinematic)
3. Drag to `Assets/Prefabs/Monsters/` to create prefab
4. Delete from scene

### Step 3: Create Monster Blueprint

1. Right-click in `Assets/Blueprints/Monsters/`
2. **Create → Vampire → Monster Blueprint**
3. Name it "Flying Monster Blueprint"

### Step 4: Configure Blueprint

```
Name: "Flying Monster"
HP: 30
Movement Speed: 3.5
Armor: 0
Knockback Resistance: 0.2
Contact Damage: 15
Attack Cooldown: 1.5

Walk Animation Sprites: (Array of flying sprites)
Seconds Per Frame: 0.15

Loot Table: (Assign a loot table asset)
Exp Reward: 5
```

### Step 5: Add to Spawn Table

1. Open `Assets/Blueprints/Levels/Level 1.asset`
2. Find `Monster Spawn Table`
3. In the spawn table asset:
   - Add your Flying Monster Blueprint
   - Set weight (higher = more common)
   - Set time range (when it can spawn)

Example spawn table entry:
```
Monster: Flying Monster Blueprint
Weight: 1.0 (same as other monsters)
Min Time %: 0.3 (appears after 30% of level)
Max Time %: 1.0 (can spawn until end)
```

### Step 6: Test

1. Play the game
2. Wait until the spawn time (30% through level)
3. Flying monsters should start appearing
4. Verify movement and behavior

---

## Creating a New Boss

Bosses are powerful enemies with special attack patterns.

### Step 1: Create Boss Ability

Create file: `Assets/Scripts/Monsters/Boss Abilities/LaserBossAbility.cs`

```csharp
using UnityEngine;

namespace Vampire.Monsters.BossAbilities
{
    [CreateAssetMenu(menuName = "Vampire/Boss Abilities/Laser")]
    public class LaserBossAbility : BossAbility
    {
        [SerializeField] private LineRenderer laserPrefab;
        [SerializeField] private float laserDamage = 50f;
        [SerializeField] private float laserDuration = 2f;
        [SerializeField] private float laserWidth = 0.5f;
        [SerializeField] private float laserRange = 20f;

        public override void Execute(BossMonster boss)
        {
            // Aim at player
            Vector2 direction = (boss.character.transform.position - boss.transform.position).normalized;

            // Create laser
            var laser = boss.entityManager.ProjectilePool.Get();
            laser.transform.position = boss.transform.position;

            // Raycast to find what laser hits
            RaycastHit2D[] hits = Physics2D.RaycastAll(
                boss.transform.position,
                direction,
                laserRange
            );

            foreach (var hit in hits)
            {
                if (hit.collider.TryGetComponent<Character>(out var character))
                {
                    character.TakeDamage(laserDamage, boss.transform.position);
                }
            }

            // Visual effect
            boss.StartCoroutine(ShowLaserEffect(boss.transform.position, direction));
        }

        private IEnumerator ShowLaserEffect(Vector2 startPos, Vector2 direction)
        {
            // Spawn line renderer
            var laser = Instantiate(laserPrefab);
            laser.SetPosition(0, startPos);
            laser.SetPosition(1, startPos + direction * laserRange);
            laser.startWidth = laserWidth;
            laser.endWidth = laserWidth;

            yield return new WaitForSeconds(laserDuration);

            Destroy(laser.gameObject);
        }

        public override float GetCooldown()
        {
            return 5f; // 5 seconds between laser attacks
        }
    }
}
```

### Step 2: Create Boss Monster Blueprint

1. Right-click in `Assets/Blueprints/Monsters/`
2. **Create → Vampire → Boss Monster Blueprint**
3. Name it "Laser Boss"

### Step 3: Configure Boss

```
Name: "Laser Boss"
HP: 5000 (much higher than normal enemies)
Movement Speed: 2.0 (slower than normal)
Armor: 50
Knockback Resistance: 0.9 (hard to knock back)
Contact Damage: 30

Boss Abilities Array:
├─ Laser Boss Ability (your new ability)
├─ Walk Boss Ability (movement)
└─ Charge Boss Ability (rush attack)

Min Ability Cooldown: 2.0
Max Ability Cooldown: 5.0
```

### Step 4: Create Boss Ability Assets

1. Right-click `Assets/Blueprints/Boss Abilities/` (create if needed)
2. **Create → Vampire → Boss Abilities → Laser**
3. Configure damage, duration, etc.

### Step 5: Add Boss to Level

Open `Assets/Blueprints/Levels/Level 1.asset`

Configure boss spawning:
```
Boss Configurations Array:
├─ Boss Blueprint: Laser Boss
├─ Spawn Time: 540 (9 minutes into 10-minute level)
├─ Spawn Position: (0, 20) (north of player spawn)
└─ Is Final Boss: True
```

### Step 6: Test

1. Play the game
2. Wait until 9 minutes (or modify spawn time to test sooner)
3. Boss should spawn with special abilities
4. Verify laser attack works

---

## Modifying Level Settings

Level settings control duration, enemy spawning, and difficulty.

### Location
`Assets/Blueprints/Levels/Level 1.asset`

### Key Settings

#### Basic Settings
```
Level Time: 600 (seconds = 10 minutes)
Background Texture: (2D texture for background)
```

#### Monster Spawning

**Monster Spawn Table:**

The spawn table uses animation curves to control enemy spawning over time:

1. Select the spawn table asset
2. Configure curves:

**Spawn Rate Curve:**
- X-axis: Time % (0 = start, 1 = end)
- Y-axis: Spawn rate multiplier
- Example curve:
  ```
  0.0 → 1.0   (normal spawning at start)
  0.5 → 2.0   (double spawning at midpoint)
  1.0 → 5.0   (5x spawning at end)
  ```

**HP Multiplier Curve:**
- X-axis: Time %
- Y-axis: HP multiplier
- Example:
  ```
  0.0 → 1.0   (normal HP at start)
  0.5 → 1.5   (50% more HP at midpoint)
  1.0 → 3.0   (3x HP at end)
  ```

#### Editing Curves

1. Double-click Level 1 blueprint
2. Find Monster Spawn Table
3. Double-click the spawn table asset
4. Click on "Spawn Rate Curve"
5. In the curve editor:
   - Right-click to add keyframes
   - Drag keyframes to adjust timing
   - Adjust tangents for smooth/sharp changes

### Example: Making Level Easier

```
Level Time: 600 → 420 (7 minutes instead of 10)
Spawn Rate Curve: Reduce all Y values by 50%
HP Multiplier Curve: Cap at 1.5x instead of 3x
Boss Spawn Time: 360 (6 minutes)
```

### Example: Making Level Harder

```
Level Time: 600 → 900 (15 minutes)
Spawn Rate Curve: Increase all Y values by 50%
HP Multiplier Curve: Go up to 5x
Add more mini-bosses at 5min, 10min marks
Initial exp gems: Reduce from 50 to 20
```

---

## Adding a New Collectible

Let's create a "Shield Pickup" that grants temporary invulnerability.

### Step 1: Create Collectible Script

Create file: `Assets/Scripts/Collectables/ShieldPickup.cs`

```csharp
using UnityEngine;
using System.Collections;

namespace Vampire.Collectables
{
    public class ShieldPickup : Collectable
    {
        [SerializeField] private float shieldDuration = 5f;
        [SerializeField] private GameObject shieldVisualPrefab;

        protected override void Collect()
        {
            base.Collect();

            // Grant shield to player
            character.StartCoroutine(ApplyShield());

            // Return to pool
            collectablePool.Return(this);
        }

        private IEnumerator ApplyShield()
        {
            // Make player invulnerable
            character.IsInvulnerable = true;

            // Spawn visual effect
            GameObject shieldVisual = Instantiate(
                shieldVisualPrefab,
                character.transform
            );

            // Wait for duration
            yield return new WaitForSeconds(shieldDuration);

            // Remove shield
            character.IsInvulnerable = false;
            Destroy(shieldVisual);
        }
    }
}
```

### Step 2: Create Prefab

1. Create GameObject: "Shield Pickup"
2. Add components:
   - `ShieldPickup` script
   - `SpriteRenderer` (shield icon sprite)
   - `CircleCollider2D` (trigger enabled)
3. Configure:
   - Shield Duration: 5
   - Shield Visual Prefab: (create a glowing circle sprite)
4. Drag to `Assets/Prefabs/Collectables/`

### Step 3: Create Blueprint

1. Right-click `Assets/Blueprints/CollectableTypes/`
2. **Create → Vampire → Collectable Type Blueprint**
3. Configure:
```
Name: "Shield"
Sprite: (shield icon)
Value: 0 (not used for shield)
Rarity: Rare
```

### Step 4: Add to Loot Tables

1. Open existing loot table: `Assets/Blueprints/Loot Tables/Standard Loot Table.asset`
2. Add shield pickup:
```
Loot Entries:
├─ Prefab: Shield Pickup
├─ Drop Chance: 0.02 (2% chance)
├─ Min Amount: 1
└─ Max Amount: 1
```

### Step 5: Add Character Invulnerability Support

Modify `Assets/Scripts/Character/Character.cs`:

```csharp
public class Character : MonoBehaviour, IDamageable
{
    public bool IsInvulnerable { get; set; }

    public void TakeDamage(float damage, Vector2 sourcePosition)
    {
        if (IsInvulnerable) return; // Ignore damage when invulnerable

        // ... rest of damage logic
    }
}
```

### Step 6: Test

1. Play the game
2. Kill enemies until shield drops (2% chance)
3. Collect shield
4. Verify you take no damage for 5 seconds
5. Verify visual effect appears

---

## Creating Passive Upgrades

Passive upgrades improve character stats globally.

### Example: Attack Speed Upgrade

#### Step 1: Define Upgradeable Value

Add to `Assets/Scripts/Utilities/UpgradeableValues.cs`:

```csharp
[System.Serializable]
public class AttackSpeed : UpgradeableFloat
{
    public AttackSpeed(float value) : base(value) { }

    public override string GetDescription()
    {
        return $"Attack Speed: +{(Value - 1f) * 100f:F0}%";
    }
}
```

#### Step 2: Create Upgrade Ability

Create `Assets/Scripts/Character/Abilities/AttackSpeedUpgrade.cs`:

```csharp
using UnityEngine;

namespace Vampire.Character.Abilities
{
    [CreateAssetMenu(menuName = "Vampire/Abilities/Attack Speed Upgrade")]
    public class AttackSpeedUpgrade : Ability
    {
        [SerializeField] private float speedIncreasePerLevel = 0.1f; // +10%

        public override void Select()
        {
            owned = true;
            Upgrade();
        }

        public override void Upgrade()
        {
            // Increase attack speed for all abilities
            abilityManager.UpgradeValue<AttackSpeed, float>(speedIncreasePerLevel);
        }

        public override string GetDescription()
        {
            return $"Increase attack speed by {speedIncreasePerLevel * 100f:F0}%";
        }
    }
}
```

#### Step 3: Apply to Weapons

Modify weapon abilities to use AttackSpeed:

```csharp
public class Dagger : Ability
{
    public Cooldown cooldown = new Cooldown(1.5f);
    public AttackSpeed attackSpeed = new AttackSpeed(1.0f); // 1.0 = normal speed

    public override void UpdateAbility()
    {
        // Apply attack speed to cooldown
        float effectiveCooldown = cooldown.Value / attackSpeed.Value;

        cooldownTimer -= Time.deltaTime;
        if (cooldownTimer <= 0f)
        {
            Attack();
            cooldownTimer = effectiveCooldown;
        }
    }
}
```

#### Step 4: Create and Add Asset

1. Create ability asset
2. Add to level abilities
3. Test

---

## Adjusting Difficulty Scaling

### Method 1: Modify Spawn Curves

Open `Assets/Blueprints/Levels/Level 1.asset` → Monster Spawn Table

**Make Easier:**
```
Spawn Rate Curve:
- Reduce Y values (less enemies)
- Make curve flatter (less ramp-up)

HP Multiplier Curve:
- Lower max multiplier
- Delay when HP scaling starts
```

**Make Harder:**
```
Spawn Rate Curve:
- Increase Y values
- Steeper curve (faster ramp-up)

HP Multiplier Curve:
- Higher multipliers
- Start scaling earlier
```

### Method 2: Modify Character Stats

Open `Assets/Blueprints/Characters/Main Character.asset`

**Make Easier:**
```
Max HP: 100 → 150
Armor: 0 → 10
Movement Speed: 5 → 6
Starting Abilities: Add an extra weapon
```

**Make Harder:**
```
Max HP: 100 → 75
Armor: 0 (no change)
Movement Speed: 5 → 4
```

### Method 3: Modify Monster Stats

Open monster blueprints: `Assets/Blueprints/Monsters/`

**Make Easier:**
```
HP: Reduce by 25%
Damage: Reduce by 25%
Movement Speed: Reduce by 20%
```

**Make Harder:**
```
HP: Increase by 50%
Damage: Increase by 50%
Movement Speed: Increase by 25%
Armor: Add 5-10 armor
```

### Method 4: Modify Exp Curve

Edit `Assets/Scripts/Character/Character.cs:330`

```csharp
// Easier (level up faster)
private int CalculateExpForNextLevel(int level)
{
    if (level < 10) return 5 + level * 5;      // Was 10
    if (level < 20) return 5 + level * 8;      // Was 13
    if (level < 30) return 5 + level * 10;     // Was 16
    return 5 + level * 12;                      // Was 20
}

// Harder (level up slower)
private int CalculateExpForNextLevel(int level)
{
    if (level < 10) return 5 + level * 15;
    if (level < 20) return 5 + level * 20;
    if (level < 30) return 5 + level * 25;
    return 5 + level * 35;
}
```

---

## Adding Localization

The game supports multiple languages using Unity's Localization system.

### Step 1: Add New Language

1. Open **Window → Asset Management → Localization Tables**
2. Select a String Table (e.g., "UI Strings")
3. Click **Add Locale**
4. Choose your language (e.g., Spanish, French, Japanese)

### Step 2: Add Translations

**For UI Strings:**
1. Open the String Table
2. Find existing entries (e.g., "GAME_OVER", "LEVEL_UP")
3. Add translations for your new locale

Example:
```
Key: GAME_OVER
English: "Game Over"
Spanish: "Fin del Juego"
French: "Jeu Terminé"
Japanese: "ゲームオーバー"
```

**For Ability Names:**
1. Open "Ability Strings" table
2. Add entries for each ability:
```
Key: ABILITY_DAGGER_NAME
English: "Dagger"
Spanish: "Daga"
French: "Dague"

Key: ABILITY_DAGGER_DESC
English: "Throws rapid daggers"
Spanish: "Lanza dagas rápidas"
French: "Lance des dagues rapides"
```

### Step 3: Use Localized Strings in Code

```csharp
using UnityEngine.Localization;
using UnityEngine.Localization.Settings;

public class MyScript : MonoBehaviour
{
    public LocalizedString localizedText;

    private void Start()
    {
        // Get localized string
        string translatedText = localizedText.GetLocalizedString();
        Debug.Log(translatedText);
    }
}
```

### Step 4: Use in UI

1. Add `Localize String Event` component to Text elements
2. Assign String Table and entry key
3. Text will automatically update when language changes

### Step 5: Change Language

In code:
```csharp
using UnityEngine.Localization.Settings;

public void SetLanguage(string localeCode)
{
    var locale = LocalizationSettings.AvailableLocales.GetLocale(localeCode);
    LocalizationSettings.SelectedLocale = locale;
}

// Example usage
SetLanguage("es"); // Spanish
SetLanguage("en"); // English
SetLanguage("zh-Hans"); // Simplified Chinese
```

---

## Creating a New Level

### Step 1: Duplicate Existing Level Scene

1. In Project, find `Assets/Scenes/Game/Level 1.unity`
2. Duplicate (Ctrl+D)
3. Rename to "Level 2.unity"

### Step 2: Create Level Blueprint

1. Right-click `Assets/Blueprints/Levels/`
2. **Create → Vampire → Level Blueprint**
3. Name it "Level 2"

### Step 3: Configure Level Blueprint

```
Level Time: 720 (12 minutes, longer than Level 1)
Background Texture: (different texture for variety)

Monster Spawn Table: (create new or duplicate Level 1's)
Boss Configurations:
├─ Mini-boss at 4 minutes
├─ Mini-boss at 8 minutes
└─ Final boss at 11 minutes

Ability Prefabs: (same as Level 1, or add new ones)

Initial Exp Gems: 30
Chest Settings: (configure chest spawning)
```

### Step 4: Modify Scene

1. Open Level 2.unity
2. Select `LevelManager` GameObject
3. In Inspector, assign "Level 2" blueprint
4. Modify background sprite/color
5. Add unique environmental objects (optional)

### Step 5: Add to Build Settings

1. **File → Build Settings**
2. Click **Add Open Scenes**
3. Level 2 should now be in the build

### Step 6: Create Level Selection

Modify Main Menu to allow level selection:

```csharp
public class MainMenu : MonoBehaviour
{
    [SerializeField] private LevelBlueprint[] availableLevels;
    private int selectedLevelIndex = 0;

    public void SelectLevel(int index)
    {
        selectedLevelIndex = index;
    }

    public void Play()
    {
        CrossSceneData.CharacterBlueprint = characterSelector.SelectedCharacter;
        CrossSceneData.LevelBlueprint = availableLevels[selectedLevelIndex];

        SceneManager.LoadScene(availableLevels[selectedLevelIndex].sceneName);
    }
}
```

### Step 7: Test

1. Play Main Menu
2. Select Level 2
3. Verify it loads with correct settings
4. Test monster spawning and bosses

---

## Advanced: Creating Synergy Systems

### Creating Weapon Combos

Some abilities could unlock when you have specific combinations:

```csharp
[CreateAssetMenu(menuName = "Vampire/Abilities/Holy Beam")]
public class HolyBeam : Ability
{
    public override bool RequirementsMet()
    {
        // Only available if player has both Bible and Garlic
        bool hasBible = abilityManager.HasAbility("Bible");
        bool hasGarlic = abilityManager.HasAbility("Garlic");
        return hasBible && hasGarlic;
    }

    public override void Use()
    {
        // Powerful ability that combines Bible and Garlic mechanics
    }
}
```

Add helper method to AbilityManager:

```csharp
public bool HasAbility(string abilityName)
{
    return abilities.Any(a => a.owned && a.name == abilityName);
}
```

---

## Debugging Tips

### Common Issues

**Issue: Ability not appearing in level-up**
- Check it's in Level Blueprint's Ability Prefabs array
- Verify RequirementsMet() returns true
- Check rarity weight settings

**Issue: Monster not spawning**
- Verify it's in Monster Spawn Table
- Check time range (Min/Max Time %)
- Verify weight > 0

**Issue: Projectiles not damaging**
- Check layers and collision matrix (Edit → Project Settings → Physics 2D)
- Verify colliders are set to triggers
- Check SpatialHashGrid is updating

### Enable Debug Logs

Add to key systems:

```csharp
#if UNITY_EDITOR
Debug.Log($"Spawning {monster.name} at {position}");
Debug.Log($"Player took {damage} damage");
Debug.Log($"Ability {ability.name} selected");
#endif
```

### Visualize Spatial Hash Grid

```csharp
private void OnDrawGizmos()
{
    if (!Application.isPlaying) return;

    Gizmos.color = Color.green;
    for (int x = 0; x < gridWidth; x++)
    {
        for (int y = 0; y < gridHeight; y++)
        {
            Vector2 cellPos = GetCellPosition(x, y);
            Gizmos.DrawWireCube(cellPos, Vector3.one * cellSize);
        }
    }
}
```

---

## Best Practices

1. **Always use ScriptableObjects for configuration** - No hardcoded values
2. **Test on multiple platforms** - PC and mobile have different controls
3. **Profile performance** - Use Unity Profiler to check frame times
4. **Use object pools** - For any frequently spawned objects
5. **Localize all user-facing text** - Don't hardcode strings
6. **Keep abilities focused** - One ability should do one thing well
7. **Balance iteratively** - Test, adjust, repeat
8. **Document your changes** - Update these guides when you add major features

---

## Next Steps

Now that you know how to modify core elements:

1. Experiment with values in ScriptableObjects
2. Create your own custom abilities
3. Design unique bosses with interesting attack patterns
4. Balance difficulty for your target audience
5. Add your own creative mechanics

Happy modding!
