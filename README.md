# AdMover — 3D Browser Game

Crowd Runner + Zombie Survival + Mob Control. WebGL browser game built with **Three.js + TypeScript + Vite**.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Visual Style & Art Direction](#visual-style--art-direction)
3. [Project Structure](#project-structure)
4. [Architecture](#architecture)
5. [Rendering Pipeline](#rendering-pipeline)
6. [Character System](#character-system)
7. [Player & Army Controller](#player--army-controller)
8. [Gate System](#gate-system)
9. [Weapon System](#weapon-system)
10. [Enemy & Zombie System](#enemy--zombie-system)
11. [Boss System](#boss-system)
12. [Skin System](#skin-system)
13. [Army Progression & Upgrades](#army-progression--upgrades)
14. [Visual Effects (VFX)](#visual-effects-vfx)
15. [HUD & UI](#hud--ui)
16. [Performance Optimization](#performance-optimization)
17. [Audio System](#audio-system)
18. [Input System](#input-system)
19. [Level & Stage Manager](#level--stage-manager)
20. [Save & Persistence](#save--persistence)
21. [Build & Deploy](#build--deploy)

---

## Tech Stack

| Layer | Tool |
|---|---|
| Language | TypeScript 5+ |
| Bundler | Vite 5+ |
| 3D Engine | Three.js r160+ |
| Physics | rapier3d-compat (Rust→WASM) |
| Animation | Tween.js / GSAP |
| State | Zustand |
| UI Overlay | HTML/CSS (raw DOM for perf) |
| Audio | Howler.js |
| Shaders | Custom GLSL (PBR + toon hybrid) |
| Model Format | GLTF/GLB (for complex models), procedural geometry for crowd units |

---

## Visual Style & Art Direction

### Reference Art

All visual references stored in `E:/PROjECTS/Games/AdMover/`:

| File | Description | Used For |
|---|---|---|
| `kommandor.png` | Knight in full plate armor with musket, 3-view turnaround | **Soldier model reference** |
| `magicalgate + knight.png` | Stone magical portal + armored knight, 3-view | **Gate model + soldier variant** |
| `player.img.png` | Knight in weathered plate armor with shotgun/rifle | **Player commander model** |
| `undead king.png` | Skeletal undead king with crown, tattered cape, great sword, 3-view | **Boss: Undead King** |
| `zombie.png` | Hooded undead with beak mask, tattered robes, glowing eyes, 3-view | **Zombie type: Plague Walker** |
| `enemy.jpg` | Skeleton zombie with knight helmet and shield | **Zombie type: Armored Skeleton** |
| `skinversion.jpg` | Detailed female knight armor, dark ornate metal | **Skin: Dark Knightess** |
| `skinver.jpg` | Cat in chainmail coif with sword | **Skin: Chainmail Cat** |
| `skinvers.jpg` | Cat in blue knight armor | **Skin: Blue Paladin Cat** |
| `one of art armorandgun.jpg` | Ornate dark knight with gun | **DELETE — not used** |

### Art Direction Summary

**Genre feel**: Dark Souls meets mobile runner. Gritty medieval aesthetic with firearms.

**Core principles**:
- **Low-poly geometry** with **hand-painted/photorealistic textures** (not flat colors)
- **Dark muted palette**: steel grays, rust browns, dark greens, bone whites, deep reds
- **Weathered/damaged look**: cracked armor, torn cloth, rusted metal, stained leather
- **Glow accents**: only for magic (gate portals, boss eyes, enchantments) — orange/yellow/purple
- **NOT bright/cartoony** — dark, atmospheric, slightly horror-tinged

### Color Palette

```
Primary metals:    #8B8682 (weathered steel), #5C4A3A (rusted iron), #3D3D3D (dark steel)
Armor highlights:  #C4B8A8 (worn plate), #6B5B4F (leather straps)
Cloth/fabric:      #4A3728 (dark leather), #2C2C2C (black cloth), #5C4033 (torn brown)
Bone/undead:       #D4C5A9 (old bone), #8B7D6B (dark bone), #FF6600 (glowing eyes)
Magic/gate:        #4A3A8A (dark purple), #6B4A9A (arcane glow), #FF8C00 (orange magic)
Environment:       #1A1A2E (dark sky), #0D0D15 (fog), #2C1810 (dark ground)
Blood/damage:      #8B0000 (dark red), #CC2200 (bright blood)
UI accent:         #C4A35A (gold coins), #FF4444 (damage), #FFFFFF (text)
```

### Material Types

| Material | Shader | Use |
|---|---|---|
| Dark Metal | PBR with low roughness (0.3), metallic (0.9), dark albedo | Armor, weapons, gates |
| Weathered Metal | PBR with high roughness (0.7), metallic (0.6), noise in albedo | Old armor, shields |
| Leather | PBR roughness 0.8, metallic 0.0, brown albedo | Straps, bags, belts |
| Cloth | PBR roughness 0.9, metallic 0.0, subsurface hint | Capes, tunics, hoods |
| Bone | PBR roughness 0.6, metallic 0.0, off-white | Skeletons, undead |
| Stone | PBR roughness 0.85, metallic 0.0, gray with noise | Gates, environment |
| Magic/Glow | Emissive material, animated intensity | Gate portal, boss eyes, enchantments |
| Skin (undead) | PBR roughness 0.7, metallic 0.0, gray-green | Zombie flesh |

---

## Project Structure

```
src/
├── main.ts                    # Entry, bootstrap, game loop
├── config.ts                  # All tuning constants
│
├── core/
│   ├── Engine.ts              # Three.js scene, camera, renderer init
│   ├── GameLoop.ts            # requestAnimationFrame + fixed-step physics
│   ├── EventBus.ts            # Typed pub/sub
│   ├── ObjectPool.ts          # Generic pool<T>
│   └── ResourceManager.ts     # GLTF/texture loader with caching
│
├── rendering/
│   ├── MaterialLibrary.ts     # PBR material factory (metal, leather, bone, stone)
│   ├── OutlinePass.ts         # Post-processing outline for character silhouettes
│   ├── ShadowSetup.ts         # PCFSoft shadow map config
│   ├── PostProcessing.ts      # Bloom, FXAA, vignette, color grading
│   ├── LODManager.ts          # Level-of-detail for distant units
│   └── TextureAtlas.ts        # Shared texture atlas for crowd units
│
├── characters/
│   ├── CharacterFactory.ts    # Creates soldier/zombie meshes from GLTF or procedural
│   ├── Soldier.ts             # Soldier logic (class)
│   ├── Zombie.ts              # Zombie logic (class)
│   ├── Boss.ts                # Boss logic (class)
│   ├── models/
│   │   ├── soldier.geo.ts     # Procedural low-poly knight geometry
│   │   ├── zombie.geo.ts      # Procedural hooded undead geometry
│   │   ├── boss_king.geo.ts   # Undead King geometry
│   │   └── gate.geo.ts        # Stone magical portal geometry
│   └── skins/
│       └── SkinRegistry.ts    # Skin loader and swap logic
│
├── systems/
│   ├── ArmySystem.ts          # Soldier crowd management
│   ├── WeaponSystem.ts        # Weapon registry, projectiles, fire logic
│   ├── GateSystem.ts          # Gate spawning, choice logic
│   ├── EnemySystem.ts         # Zombie wave spawner, AI
│   ├── BossSystem.ts          # Boss spawn, phases, HP bar
│   ├── CoinSystem.ts          # Coin drop, collection, spending
│   ├── UpgradeSystem.ts       # Stats, multipliers, upgrades
│   ├── ProgressionSystem.ts   # Stage/level progression
│   └── DamageNumberSystem.ts  # Floating text pool
│
├── effects/
│   ├── ParticlePool.ts        # GPU particle system wrapper
│   ├── MuzzleFlash.ts         # Gun flash effect (orange/yellow)
│   ├── Explosion.ts           # Explosion burst (dark fire)
│   ├── BulletTrail.ts         # Trail renderer (smoke + spark)
│   ├── BloodEffect.ts         # Dark red hit splash
│   ├── ScreenShake.ts         # Camera shake controller
│   ├── CoinPopup.ts           # "+coins" floating text
│   ├── MagicPortal.ts         # Gate portal swirl effect (purple/orange)
│   └── DeathEffect.ts         # Undead dissolve (green soul particles)
│
├── camera/
│   ├── GameCamera.ts          # Follow cam with smoothing
│   └── CameraShake.ts         # Shake overlay
│
├── input/
│   ├── InputManager.ts        # Unified mouse/touch/keyboard
│   └── SwipeController.ts     # Drag/swipe detection
│
├── ui/
│   ├── HUD.ts                 # HTML overlay: level, coins, army, weapon
│   ├── GateChoiceUI.ts        # Gate selection overlay
│   ├── WeaponChooseUI.ts      # Post-boss weapon picker (3 options)
│   ├── UpgradeMenu.ts         # Between-stage upgrade screen
│   ├── PauseMenu.ts
│   ├── GameOverScreen.ts
│   └── SkinSelectUI.ts        # Skin selection screen
│
├── data/
│   ├── weapons.ts             # Weapon definitions
│   ├── zombies.ts             # Zombie type definitions
│   ├── bosses.ts              # Boss definitions
│   ├── gates.ts               # Gate pool definitions
│   ├── upgrades.ts            # Upgrade tree definitions
│   ├── skins.ts               # Skin definitions
│   └── stages.ts              # Stage difficulty curves
│
├── stores/
│   └── gameStore.ts           # Zustand global state
│
└── utils/
    ├── math.ts
    ├── pool.ts
    └── random.ts
```

---

## Architecture

### Game Loop

Fixed-timestep physics (120 Hz) decoupled from render (60 FPS). Three-phase per frame:

1. **Input** — read swipe delta
2. **Update** — physics step, AI, weapon fire, collision, gates, coins
3. **Render** — Three.js render, post-processing, HUD sync

### State Flow

```
InputManager → SwipeController → GameCamera offset
                                     ↓
                              ArmySystem (move crowd)
                                     ↓
                         WeaponSystem (auto-fire at nearest)
                                     ↓
                          EnemySystem (spawn waves, AI)
                                     ↓
                         CollisionSystem (rapier broadphase)
                                     ↓
                    DamageNumberSystem + ParticlePool + CoinSystem
                                     ↓
                              HUD update
```

### Event Bus

```ts
EventBus.on('enemy:kill', (data: { pos: Vector3, xp: number }) => { ... });
EventBus.on('gate:select', (data: { type: GateType, value: number }) => { ... });
EventBus.on('boss:defeated', (data: { bossId: string }) => { ... });
EventBus.on('weapon:fire', (data: { origin: Vector3, dir: Vector3, type: WeaponType }) => { ... });
EventBus.on('skin:change', (data: { skinId: string }) => { ... });
```

---

## Rendering Pipeline

### Three.js Setup

```
Renderer: WebGLRenderer
  - antialias: true
  - powerPreference: 'high-performance'
  - outputColorSpace: THREE.SRGBColorSpace
  - toneMapping: THREE.ACESFilmicToneMapping
  - toneMappingExposure: 0.9          // darker overall than default
  - shadowMap.enabled: true
  - shadowMap.type: THREE.PCFSoftShadowMap

Scene:
  - background: Color(0x0D0D15)       // near-black with slight blue
  - fog: THREE.FogExp2(0x0D0D15, 0.015) // dense dark fog
```

### Lighting

| Light | Config | Purpose |
|---|---|---|
| DirectionalLight | intensity 1.2, color 0xFFF5E6 (warm), castShadow | Main sun/moon light |
| HemisphereLight | skyColor 0x1A1A2E, groundColor 0x2C1810, intensity 0.4 | Ambient fill |
| AmbientLight | intensity 0.2, color 0x333344 | Base darkness |
| PointLight (gate) | intensity 2.0, color 0x6B4A9A, distance 15 | Portal glow |
| PointLight (boss eyes) | intensity 1.5, color 0xFF6600, distance 8 | Undead King eye glow |

Shadow: `PCFSoftShadowMap`, bias -0.002, normalBias 0.03, mapSize 2048 (1024 mobile).

### Post-Processing Chain

```
RenderPass
  → UnrealBloomPass(threshold 0.6, strength 0.4, radius 0.6)   // glow on magic/eyes
  → FXAAPass
  → ColorGradingShader(contrast 1.1, saturation 0.85)           // slightly desaturated
  → VignetteShader(intensity 0.4)                                // dark edges
```

### Material Library

```ts
// Dark metal — armor, weapons
function createDarkMetal(): MeshStandardMaterial {
  return new MeshStandardMaterial({
    color: 0x5C4A3A,
    roughness: 0.35,
    metalness: 0.9,
    envMapIntensity: 0.8,
  });
}

// Weathered steel — old armor
function createWeatheredSteel(): MeshStandardMaterial {
  return new MeshStandardMaterial({
    color: 0x8B8682,
    roughness: 0.65,
    metalness: 0.7,
  });
}

// Bone — skeletons, undead
function createBone(): MeshStandardMaterial {
  return new MeshStandardMaterial({
    color: 0xD4C5A9,
    roughness: 0.6,
    metalness: 0.0,
  });
}

// Stone — gates, environment
function createStone(): MeshStandardMaterial {
  return new MeshStandardMaterial({
    color: 0x4A4A4A,
    roughness: 0.85,
    metalness: 0.0,
  });
}

// Magic glow — portal, enchantments
function createMagicGlow(color: number = 0x6B4A9A): MeshStandardMaterial {
  return new MeshStandardMaterial({
    color: color,
    emissive: color,
    emissiveIntensity: 2.0,
    roughness: 0.3,
    metalness: 0.0,
    transparent: true,
    opacity: 0.85,
  });
}
```

---

## Character System

### Soldier Model — Knight with Firearm

Based on `kommandor.png` and `player.img.png` references.

**Geometry** (procedural low-poly, ~300 triangles per soldier):

```
Torso:      BoxGeometry(0.35, 0.45, 0.2)     — plate armor chest
Shoulders:  BoxGeometry(0.12, 0.08, 0.15) ×2 — pauldrons (rounded edges)
Head:       SphereGeometry(0.14, 8, 6)        — helmet base
Visor:      BoxGeometry(0.16, 0.1, 0.05)      — helmet visor plate
Arms:       CapsuleGeometry(0.06, 0.25, 4, 6) — armored gauntlets
Legs:       CapsuleGeometry(0.07, 0.3, 4, 6)  — greaves
Feet:       BoxGeometry(0.1, 0.05, 0.14)      — sabatons
Weapon:     BoxGeometry(0.04, 0.04, 0.5)      — musket/rifle barrel
            BoxGeometry(0.06, 0.08, 0.12)     — stock
Cape:       PlaneGeometry(0.3, 0.4) ×2        — tattered cloth (both sides)
Bag/Pouch:  BoxGeometry(0.08, 0.1, 0.06)      — satchel on hip
```

All parts merged via `BufferGeometryUtils.mergeGeometries()` — **one draw call per batch**.

**Materials per soldier**:
- Armor plates: `createWeatheredSteel()` with random rust tint variation (±10% hue)
- Cloth/cape: dark brown `MeshStandardMaterial` roughness 0.9
- Leather straps: `createLeather()` — dark brown
- Weapon: `createDarkMetal()` — dark iron
- Helmet visor: slightly more reflective than body armor

**Soldier variation** (to avoid monotonous crowd):
- Random rust tint on armor (±15% on R/G/B channels)
- Random cape length (0.3–0.5)
- Random pouch presence (60% chance)
- Slight size variation (0.9–1.1 scale)

### Procedural Animation (No Skeletal)

All animation via `Math.sin/cos` transforms — zero bone system, zero animation clips:

```ts
// Idle — subtle breathing
soldier.position.y += Math.sin(time * 1.5) * 0.01;
soldier.children.torso.rotation.x = Math.sin(time * 1.5) * 0.02;

// Run — leg swing + body bob
leg.rotation.x = Math.sin(time * 8 + offset) * 0.4;
body.position.y = Math.abs(Math.sin(time * 8 + offset)) * 0.03;
cape.rotation.x = Math.sin(time * 8 + offset) * 0.15 + 0.2; // wind effect

// Shoot — recoil + muzzle flash
arm.rotation.x -= 0.3; // kickback
arm.rotation.x = lerp(arm.rotation.x, 0, 0.1); // recovery
// trigger muzzle flash at gun tip
```

### Zombie Model — Hooded Undead

Based on `zombie.png` (Plague Walker) and `enemy.jpg` (Armored Skeleton).

**Geometry** (~250 triangles):

```
Torso:      BoxGeometry(0.3, 0.4, 0.18)      — tattered tunic
Head:       SphereGeometry(0.12, 6, 5)        — skull/hood base
Hood:       ConeGeometry(0.15, 0.2, 6)        — draped hood
Beak mask:  ConeGeometry(0.04, 0.12, 4)       — plague doctor beak (some types)
Arms:       CapsuleGeometry(0.04, 0.22, 3, 5) — thin, emaciated
Legs:       CapsuleGeometry(0.05, 0.25, 3, 5) — torn pants
Hands:      SphereGeometry(0.05, 4, 4)         — clawed fingers
Rope belt:  TorusGeometry(0.12, 0.01, 4, 8)   — rope tied at waist
```

**Materials**:
- Cloth/hood: `MeshStandardMaterial` color 0x5C5040, roughness 0.95 — filthy ragged cloth
- Skin: `MeshStandardMaterial` color 0x6B5D4F, roughness 0.7 — diseased flesh
- Eyes: `MeshStandardMaterial` emissive 0xFF6600, emissiveIntensity 3.0 — glowing orange
- Rope: `MeshStandardMaterial` color 0x8B7355, roughness 0.9

**Zombie variation**:
- Hood color variation (brown/gray/green tint)
- Random torn holes in cloth (via alpha map or geometry cuts)
- Some have beak mask (plague doctor), some don't
- Some have knight helmet (armored skeleton variant from `enemy.jpg`)
- Arm posture: some reaching forward, some hanging

### Undead King Boss Model

Based on `undead king.png`.

**Geometry** (~800 triangles — higher detail for boss):

```
Torso:      BoxGeometry(0.5, 0.6, 0.3)       — large armored chest
Crown:      CylinderGeometry(0.15, 0.18, 0.15, 8) — spiked crown
            ConeGeometry(0.02, 0.1, 4) ×5    — crown spikes
Head:       SphereGeometry(0.18, 8, 6)        — skeletal skull
Jaw:        BoxGeometry(0.12, 0.06, 0.1)      — exposed jaw
Shoulders:  BoxGeometry(0.18, 0.1, 0.2) ×2    — massive pauldrons
Arms:       CapsuleGeometry(0.08, 0.35, 5, 6) — armored
Hands:      BoxGeometry(0.1, 0.12, 0.08) ×2   — gauntlets
Legs:       CapsuleGeometry(0.09, 0.35, 5, 6) — armored greaves
Cape:       PlaneGeometry(0.6, 0.8) ×2        — long tattered cape
Sword:      BoxGeometry(0.04, 0.7, 0.02)      — great sword blade
            BoxGeometry(0.15, 0.04, 0.04)     — crossguard
            CylinderGeometry(0.02, 0.02, 0.2, 6) — grip
Belt:       TorusGeometry(0.22, 0.02, 4, 8)   — chain belt
```

**Materials**:
- Crown: `createDarkMetal()` with gold emissive hint
- Skull: `createBone()` with dark wash
- Eyes: emissive 0xFF4400, intensity 4.0 — bright burning orange
- Armor: `createDarkMetal()` with extra rust
- Cape: `MeshStandardMaterial` color 0x2C1810, roughness 0.95 — torn dark fabric
- Sword: `createDarkMetal()` with subtle blue tint

---

## Player & Army Controller

### Player (Commander)

Based on `player.img.png` — weathered knight with prominent rifle.

- Positioned at army front, 1.3× scale
- Distinct armor: slightly cleaner steel than soldiers
- Wears a short cape (gold-trimmed)
- Holds weapon more prominently (rifle at ready position)
- Subtle glow on helmet visor (player indicator)

### Crowd Follow Logic (Boids-like)

```ts
for each soldier in army:
  targetPos = leader.position + formationOffset(soldier.index)
  soldier.position.lerp(targetPos, 0.15)
  soldier.position.x = clamp(soldier.position.x, -ARENA_HALF_WIDTH, ARENA_HALF_WIDTH)
```

Formation: honeycomb grid around leader. Outer ring expands as army grows.

### Auto-Shoot

Each soldier finds nearest enemy within range → fires projectile. Fire rate based on weapon stats. Cooldown timer per soldier.

---

## Gate System

### Gate Structure — Magical Stone Portal

Based on `magicalgate + knight.png` — carved stone arch with dark magic portal.

**Geometry** (~500 triangles per gate):

```
Left pillar:    BoxGeometry(0.4, 3.0, 0.4)
Right pillar:   BoxGeometry(0.4, 3.0, 0.4)
Arch top:       BoxGeometry(1.2, 0.4, 0.4)    — curved via vertex displacement
Arch curve:     TorusGeometry(0.6, 0.2, 4, 8, PI) — half-torus for arch shape
Base:           BoxGeometry(1.4, 0.3, 0.6)     — stone foundation
Carvings:       BoxGeometry(0.05, 0.15, 0.05) ×8 — decorative runes/patterns
Statues:        CylinderGeometry(0.08, 0.1, 0.3, 6) ×2 — small figure carvings on pillars
Portal plane:   PlaneGeometry(1.0, 2.5)        — magic effect surface
```

**Materials**:
- Stone body: `createStone()` — gray-brown weathered stone
- Portal surface: `createMagicGlow(0x4A3A8A)` — animated purple swirl
- Runes: emissive 0x6B4A9A, pulsing intensity

### Portal Effect

```ts
// Animated shader for portal swirl
portalMaterial.emissiveIntensity = 1.5 + Math.sin(time * 2) * 0.5;
portalMaterial.uniforms.time.value = time;

// Particle swirl inside portal
// 20-30 purple/orange particles orbiting inside the arch
```

### Gate Pool

```ts
interface GateOption {
  type: 'add_soldiers' | 'multiply_soldiers' | 'add_damage' |
        'add_firerate' | 'add_bullet' | 'weapon' | 'special';
  value: number;
  label: string;
  weight: number;
}
```

### Strategic Gate Feature

Left = quantity (more soldiers), Right = quality (better weapon):

```
Left: +100 Soldiers  |  Right: Blunderbuss
Left: x3 Soldiers    |  Right: +50% Damage
Left: +200 Soldiers  |  Right: Hand Cannon
```

---

## Weapon System

### Medieval Firearms Theme

Based on art references — weapons are **muskets, blunderbusses, hand cannons, rifles** — not modern guns.

### Weapon Definitions

```ts
interface WeaponDef {
  id: string;
  name: string;
  damage: number;
  fireRate: number;
  bulletSpeed: number;
  bulletCount: number;
  spread: number;
  piercing: boolean;
  explosive: boolean;
  aoeRadius: number;
  projectileType: 'musket_ball' | 'cannonball' | 'scatter_shot' | 'flame';
  color: number;        // muzzle flash color
  trailColor: number;   // bullet trail color
  sound: string;
}
```

### Weapon Roster

| Weapon | Style | Damage | FireRate | Special |
|---|---|---|---|---|
| Musket | Long barrel, slow reload | 15 | 2/s | — |
| Blunderbuss | Flared barrel, close range | 10×6 | 1/s | scatter |
| Hand Cannon | Short, heavy | 40 | 1/s | knockback |
| Arquebus | Improved musket | 25 | 3/s | — |
| Pepperbox | Multi-barrel | 8 | 5/s | — |
| Bombard | Portable cannon | 100 | 0.3/s | explosive AOE |
| Fire Lance | Tube weapon | 8/tick | continuous | flame cone |
| Matchlock Rifle | Long range | 60 | 0.5/s | piercing |

### Projectile Visual

- **Musket ball**: small dark sphere (SphereGeometry 0.03), trailing smoke particles
- **Cannonball**: larger sphere (0.06), fire trail, explosion on impact
- **Scatter shot**: 6 small spheres in cone spread
- **Flame**: particle burst, no mesh (orange/red particles with gravity)

### Muzzle Flash

Orange-yellow flash at barrel tip:
```ts
// Muzzle flash: sprite billboard, 0.08s lifetime
// Color: 0xFF8C00 (orange) to 0xFFFF00 (yellow)
// Size: random 0.1-0.2 scale
// Additional: 3-5 spark particles shooting outward
```

---

## Enemy & Zombie System

### Zombie Types

| Type | Based On | HP | Speed | Special |
|---|---|---|---|---|
| Plague Walker | `zombie.png` hooded undead | 30 | 1.2 | — |
| Armored Skeleton | `enemy.jpg` skeleton+knight helmet | 80 | 0.9 | 40% bullet resist |
| Fast Ghoul | Thin, hunched variant | 20 | 2.8 | burst speed |
| Bloated Corpse | Fat variant, no hood | 60 | 0.6 | explodes on death |
| Plague Doctor | Beak mask variant | 50 | 1.0 | heals nearby |
| Dark Acolyte | Hooded, staff | 40 | 1.1 | ranged attack |

### Zombie AI

```
SPAWN → WALK_TO_ARMY → ATTACK → (DEAD)
         ↓               ↓
      Find target     Deal damage per second
      Move toward     Reduce army count
                      Play attack anim
```

Straight-line movement toward nearest soldier. No pathfinding.

### Zombie Visual Effects on Death

Based on undead theme:
- Green/teal soul particles float upward (5-10 particles)
- Body shrinks + dissolves (scale to 0 over 0.5s)
- Cloth tatters remain briefly then fade
- Dark smoke puff at death location

---

## Boss System

### Boss: Undead King

Based on `undead king.png`.

| Stat | Value |
|---|---|
| HP | 10000 |
| Phases | 4 |
| Scale | 3× soldier size |
| Arena | Open ground before the final gate |

### Boss Attacks

| Phase | HP Range | Attacks |
|---|---|---|
| 1 | 100-75% | Sword sweep (wide arc), Summon 5 skeletons |
| 2 | 75-50% | Slam (AoE circle), Summon 10 skeletons, Cape swirl (push soldiers) |
| 3 | 50-25% | Dark wave (projectile ring), Summon 15 skeletons, Life drain (heals) |
| 4 | 25-0% | Enrage (speed ×2, damage ×2), Continuous summon, Execute (one-shot soldiers) |

### Boss Visual

- Sword glows orange during attacks
- Eyes pulse brighter during enrage
- Cape flows with cloth simulation (vertex animation)
- Dark aura particles surround boss (20 particles orbiting)
- Ground cracks on slam (decal projection)
- Summoning: dark portal opens, skeletons emerge

### Boss Health Bar

Large bar at top of screen:
```
┌──────────────────────────────────────────────┐
│  👑 UNDEAD KING        ████████░░░░  65%     │
└──────────────────────────────────────────────┘
```
Color: dark red (#8B0000) with gold border. Phase transitions flash white.

### Boss Defeat

1. Slow-motion (0.3× time scale, 2 seconds)
2. Sword drops (physics)
3. Crown falls off
4. Body dissolves upward (green soul particles, 100+)
5. Dark explosion + screen shake
6. Coins burst (50-200)
7. Camera orbits boss location
8. Weapon choose screen appears

---

## Skin System

Based on cat knight references (`skinver.jpg`, `skinvers.jpg`) and `skinversion.jpg`.

### Skin Definitions

```ts
interface SkinDef {
  id: string;
  name: string;
  description: string;
  modelOverrides?: Partial<SoldierModel>;
  materialOverrides?: Partial<MaterialSet>;
  unlockCondition: string;
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
}
```

### Available Skins

| Skin | Based On | Rarity | Unlock |
|---|---|---|---|
| Default Knight | `kommandor.png` | Common | Start |
| Dark Knightess | `skinversion.jpg` | Rare | Stage 20 |
| Chainmail Cat | `skinver.jpg` | Epic | Stage 50 |
| Blue Paladin | `skinvers.jpg` | Epic | Buy 500 coins |
| Plague Doctor | `zombie.png` style | Rare | Kill 1000 zombies |
| Undead King (mini) | `undead king.png` style | Legendary | Stage 100 |

### Skin Swap Logic

Skins replace soldier model geometry and materials. InstancedMesh gets new geometry + material on skin change. All soldiers in army share the same skin.

```ts
function applySkin(soldierMesh: InstancedMesh, skin: SkinDef) {
  soldierMesh.geometry = getSkinGeometry(skin.id);
  soldierMesh.material = getSkinMaterials(skin.id);
}
```

---

## Army Progression & Upgrades

### Coin System

- Zombies drop coins on death (1-5 based on type)
- Coins: gold cubes (BoxGeometry 0.08, emissive 0xC4A35A, rotating)
- Float up with bounce animation
- Auto-collected when soldier walks over
- Boss drops 50-200 coins

### Upgrade Menu (Between Stages)

After every 5 stages:

```
┌─────────────────────────────────────────┐
│        ARMORY (500 gold coins)          │
├─────────────┬─────────────┬─────────────┤
│ Blade Power │ Reload Speed│ Army Cap +  │
│   +20% dmg  │   +15% rate │   +50 max   │
│   200 gold  │   300 gold  │   400 gold  │
├─────────────┼─────────────┼─────────────┤
│ Critical    │ March Speed │ Gold Finder │
│   +5% crit  │   +10% move │   ×1.5 gold │
│   250 gold  │   350 gold  │   500 gold  │
└─────────────┴─────────────┴─────────────┘
```

### Upgrade Tree

```ts
const UPGRADES: Upgrade[] = [
  { id: 'damage', name: 'Blade Power', maxLevel: 20, baseCost: 200, costScale: 1.5,
    effect: (lvl) => ({ stat: 'damage', mult: 1 + lvl * 0.2 }) },
  { id: 'firerate', name: 'Reload Speed', maxLevel: 15, baseCost: 300, costScale: 1.6,
    effect: (lvl) => ({ stat: 'fireRate', mult: 1 + lvl * 0.15 }) },
  { id: 'maxarmy', name: 'Army Cap', maxLevel: 10, baseCost: 400, costScale: 2.0,
    effect: (lvl) => ({ stat: 'maxArmy', add: lvl * 50 }) },
  { id: 'crit', name: 'Critical', maxLevel: 10, baseCost: 250, costScale: 1.8,
    effect: (lvl) => ({ stat: 'critChance', add: lvl * 0.05 }) },
  { id: 'speed', name: 'March Speed', maxLevel: 8, baseCost: 350, costScale: 1.7,
    effect: (lvl) => ({ stat: 'moveSpeed', mult: 1 + lvl * 0.1 }) },
  { id: 'coinmult', name: 'Gold Finder', maxLevel: 5, baseCost: 500, costScale: 2.5,
    effect: (lvl) => ({ stat: 'coinMult', mult: 1 + lvl * 0.5 }) },
];
```

---

## Visual Effects (VFX)

### Particle System (GPU-based)

`THREE.Points` with `BufferGeometry`. Pool of 5000 particles.

### Effects List

| Effect | Visual | Color |
|---|---|---|
| Bullet trail | Smoke line, 3-5 points, fading | 0x888888 |
| Muzzle flash | Billboard sprite, 0.08s | 0xFF8C00 → 0xFFFF00 |
| Explosion | 50-100 particles, outward burst | 0xFF4400, 0xFF8800 |
| Blood/damage | Red splash, 10-20 particles, low gravity | 0x8B0000, 0xCC2200 |
| Damage numbers | CSS div, float up + fade | White (normal), 0xFF4444 (crit) |
| Coin popups | "+5" text, float up, gold color | 0xC4A35A |
| Critical hits | Larger text, screen flash, ring | 0xFF0000 |
| Screen shake | Camera offset oscillation, 0.3s decay | — |
| Death dissolve | Green soul particles float up | 0x00FF88, 0x44AAFF |
| Gate portal | Purple swirl, orbiting particles | 0x6B4A9A, 0x4A3A8A |
| Boss aura | Dark particles orbit boss | 0xFF4400, 0x440000 |
| Victory | Dark confetti (gray/bone colored) | 0x888888, 0xD4C5A9 |

### Screen Shake

```ts
class CameraShake {
  private intensity: number = 0;
  private decay: number = 0.9;

  trigger(amount: number): void { this.intensity = amount; }

  update(camera: Camera): void {
    if (this.intensity > 0.01) {
      camera.position.x += (Math.random() - 0.5) * this.intensity;
      camera.position.y += (Math.random() - 0.5) * this.intensity;
      this.intensity *= this.decay;
    }
  }
}
```

### Damage Numbers

```ts
function showDamageNumber(worldPos: Vector3, damage: number, isCrit: boolean) {
  const screenPos = worldPos.clone().project(camera);
  const div = getPoolDiv();
  div.textContent = isCrit ? `${damage}!` : `${damage}`;
  div.style.left = `${(screenPos.x + 1) * 0.5 * window.innerWidth}px`;
  div.style.top = `${(-screenPos.y + 1) * 0.5 * window.innerHeight}px`;
  div.style.color = isCrit ? '#FF4444' : '#FFFFFF';
  div.style.fontSize = isCrit ? '28px' : '18px';
  div.style.fontFamily = "'MedievalSharp', serif";
  div.style.textShadow = '2px 2px 4px #000000';
  requestAnimationFrame(() => {
    div.style.transform = 'translateY(-50px)';
    div.style.opacity = '0';
  });
  setTimeout(() => returnToPool(div), 800);
}
```

---

## HUD & UI

### Font

**MedievalSharp** or **Uncial Antiqua** from Google Fonts — medieval/gothic feel.

### HUD Layout

```
┌─────────────────────────────────────────┐
│ Stage 5        1250 gold     Musket     │
│ ┌─────────────────────────────────────┐ │
│ │ ☠ UNDEAD KING  ████████░░░░  65%   │ │
│ └─────────────────────────────────────┘ │
│                                         │
│              [3D SCENE]                 │
│                                         │
│         Army: 347 souls                 │
└─────────────────────────────────────────┘
```

### CSS Styling

- Dark semi-transparent background panels
- Gold coin icon with pulse animation
- Gothic/medieval font
- Muted colors: grays, browns, golds
- Boss HP bar: dark red with gold border
- Army count: "347 souls" (not "soldiers")

---

## Performance Optimization

### Instanced Rendering

```ts
const soldierMesh = new InstancedMesh(
  soldierGeometry,
  soldierMaterial,
  MAX_SOLDIERS  // 2000
);

function updateSoldierInstances() {
  const dummy = new Object3D();
  for (let i = 0; i < aliveCount; i++) {
    dummy.position.copy(soldiers[i].position);
    dummy.quaternion.copy(soldiers[i].rotation);
    dummy.scale.copy(soldiers[i].scale);
    dummy.updateMatrix();
    soldierMesh.setMatrixAt(i, dummy.matrix);
  }
  soldierMesh.instanceMatrix.needsUpdate = true;
}
```

### Object Pooling

Pools: Projectiles (500), Particles (5000), Damage numbers (100), Coins (200), Muzzle flashes (50).

### LOD

| Distance | Detail |
|---|---|
| 0-25m | Full model, shadows, all materials |
| 25-50m | Simplified (fewer segments), no shadows |
| 50m+ | Billboard sprite |

### Target Performance

| Platform | Target |
|---|---|
| Desktop | 60 FPS, 1000+ soldiers, 1000+ zombies |
| Mobile | 60 FPS, 500+ soldiers, 500+ zombies |
| Low-end mobile | 30 FPS, 200+ soldiers, 200+ zombies |

---

## Audio System

| Sound | Style | Notes |
|---|---|---|
| Musket fire | Boom + echo | Reverb, pitch variation |
| Blunderbuss | Multiple impacts | Scatter sound |
| Cannon blast | Deep boom | Camera shake trigger |
| Sword slash | Metal ring | Boss attacks |
| Zombie groan | Low moan, multiple variants | Ambient loop |
| Coin collect | Gold coin jingle | Pitched up |
| Gate pass | Stone grind + magic whoosh | Portal activation |
| Boss roar | Distorted, deep | Echo + reverb |
| Death dissolve | Ethereal fade | Eerie tone |
| BGM | Dark orchestral, 90 BPM | Cello, drums, choir |

---

## Input System

### Desktop

- Mouse drag on canvas: swipe left/right to steer
- A/D keys

### Mobile

- Touch drag on screen: swipe left/right

### Swipe Detection

```ts
class SwipeController {
  private startX: number = 0;
  private isDragging: boolean = false;
  private sensitivity: number = 0.01;

  onStart(x: number): void { this.startX = x; this.isDragging = true; }
  onMove(x: number): number {
    if (!this.isDragging) return 0;
    const delta = (x - this.startX) * this.sensitivity;
    this.startX = x;
    return clamp(delta, -1, 1);
  }
  onEnd(): void { this.isDragging = false; }
}
```

---

## Level & Stage Manager

### Stage Structure

1. Army spawns at start
2. Dark road stretches forward (stone path with ruins on sides)
3. Zombies spawn in waves
4. Magical gates appear between waves
5. Stage ends after distance/wave clear
6. Every 5th stage = boss (Undead King)

### Road Generation

Infinite scrolling stone road with ruins:

```ts
const roadSegments = [
  createStoneRoad(0),      // stone path with cracks
  createStoneRoad(-50),
  createStoneRoad(-100),
];
// Sides: broken walls, tombstones, dead trees, bones
// Recycled as player moves forward
```

### Environment Decorations

- Broken stone walls
- Dead trees (bare branches)
- Tombstones and crosses
- Scattered bones and skulls
- Fog wisps near ground
- Distant ruined castle silhouettes

### Difficulty Scaling

```ts
function getDifficulty(stage: number) {
  return {
    zombieHpMult: 1 + stage * 0.1,
    zombieCountMult: 1 + stage * 0.15,
    zombieSpeedMult: 1 + stage * 0.02,
    gateFrequency: Math.max(30, 50 - stage),
    bossEvery: 5,
  };
}
```

---

## Save & Persistence

```ts
interface SaveData {
  coins: number;
  upgrades: Record<string, number>;
  bestStage: number;
  totalKills: number;
  selectedSkin: string;
  unlockedSkins: string[];
  settings: { music: boolean; sfx: boolean; };
}
// LocalStorage, auto-save after each stage
```

---

## Build & Deploy

### Development

```bash
npm install
npm run dev
```

### Production

```bash
npm run build
npm run preview
```

### Vite Config

```ts
export default defineConfig({
  base: './',
  build: {
    target: 'esnext',
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          three: ['three'],
          howler: ['howler'],
        },
      },
    },
  },
});
```

---

## Implementation Order

### Phase 1 — Core (Week 1)

1. Vite + Three.js boilerplate
2. Scene, camera, renderer, dark lighting
3. Material library (metal, bone, stone, cloth)
4. Stone road generation (infinite scroll)
5. Player commander model (from `player.img.png`)
6. Input system (swipe)
7. Camera follow
8. Basic army (10 soldiers, knight models, follow leader)

### Phase 2 — Combat (Week 2)

9. Zombie models (Plague Walker from `zombie.png`)
10. Zombie InstancedMesh + AI
11. Collision detection
12. Soldier auto-shoot (musket)
13. Projectile pool (musket balls)
14. Damage system
15. Coin drops + collection

### Phase 3 — Gates & Weapons (Week 3)

16. Gate model (magical stone portal from `magicalgate + knight.png`)
17. Portal shader effect (purple swirl)
18. Gate choice logic
19. Weapon definitions (medieval firearms)
20. Weapon switching
21. Projectile behavior per weapon
22. Post-boss weapon choose screen

### Phase 4 — Effects & Polish (Week 4)

23. GPU particle system
24. Muzzle flash (orange/yellow)
25. Explosions (dark fire)
26. Bullet trails (smoke)
27. Blood effects (dark red)
28. Death dissolve (green soul particles)
29. Damage numbers (gothic font)
30. Screen shake
31. Post-processing (bloom, color grading, vignette)

### Phase 5 — Boss & Progression (Week 5)

32. Undead King model (from `undead king.png`)
33. Boss HP bar
34. Boss attack patterns (4 phases)
35. Boss summon mechanics
36. Stage progression system
37. Difficulty scaling
38. Upgrade menu ("Armory")
39. Upgrade system

### Phase 6 — Skins, UI & Audio (Week 6)

40. Skin system (cat knights, dark knightess, etc.)
41. Skin selection UI
42. HUD overlay (medieval font)
43. Coin counter animation
44. Army count ("X souls")
45. Boss HP bar UI
46. Pause menu
47. Game over screen
48. Sound effects
49. Background music (dark orchestral)

### Phase 7 — Optimization & QA (Week 7)

50. LOD system
51. Performance profiling
52. Mobile touch testing
53. Memory leak audit
54. Save/load system
55. Final polish pass

---

*Total estimated: 7 weeks for a production-quality dark medieval 3D browser game.*
