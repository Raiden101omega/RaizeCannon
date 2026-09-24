# Raizecannon v2.1 — Full Audit Report

Every claim, file and field was checked against the **actual Mindustry v154
source code** (tag `v154` of Anuken/Mindustry): `ContentParser`, `Mods`,
`UnitType`, `Weapon`, bullet types, `StatusEffect`, `Turret`/`ItemTurret`/
`LiquidTurret`, `Reconstructor`, `UnitFactory`, `GenericCrafter`, `Block`,
`Consume*`, `Category`, `Sounds` (generated from `core/assets/sounds`), and
the tech-tree code.

## Verdict of the original zip

The mod **loads** (JSON valid, strict field checking is warning-only in v154,
`ignoreUnknownFields = true`), and its core architecture is sound. But it had
**9 real defects** — several of them silently breaking documented behaviour.

## Confirmed working (no action needed)

| Item | Evidence |
|---|---|
| Zip layout (`mod.hjson` at root, `content/`, `sprites/`) | Mindustry mod layout spec |
| `mod.hjson` fields (name, displayName, minGameVersion, version, author, description) | all real `ModMeta` fields |
| Unit-side `requirements` (`block` + `previous`) for reconstructor upgrades | `ContentParser` lines 602–617: `r.upgrades.add(new UnitType[]{req.previous, unit})` |
| Forward references (e.g. `brood-crawler` → `raize-larva`) | upgrade reads are deferred to `finishParsing()` |
| `UnitFactory.plans` with unit/time/requirements | real field, correct shape |
| Sprite sizes: 3×3 blocks 96×96, 2×2 blocks 64×64, 32-bit RGBA | measured every PNG |
| Reconstructor `-in`/`-out`/`-top` sprite names | `PayloadBlock.findFactoryRegion()` |
| Unit `type` values `crawl` / `mech` / `flying` | `ContentParser.unitType()` switch |
| Category values `units` / `turret` / `crafting` | v154 `Category` enum (note: `turret` is singular in v7) |
| Research parents `cultivator`, `scatter`, `wave` | vanilla Serpulo blocks |
| Liquid economy scale | vat outputs per-tick; consumers drain only while `efficiency > 0`; 1.0–3.0/tick draw matches vanilla reconstructors' cryofluid draw |
| `LiquidTurret` + `LiquidBulletType` with explicit `liquid` and `status` | no-arg constructor leaves JSON values intact |
| All item references (copper…surge-alloy, phase-fabric, spore-pod) | vanilla item names |
| Status/liquid/weapon field names (after fixes below) | source-verified |

## Defects found & fixed

1. **All sounds were invalid → completely silent weapons.**
   `shootSound: "shoot-small"` does not exist in v154 (sounds resolve by
   exact asset field name, e.g. `shootDuo`), and `ambientSound: "machine"`
   is not a sound either (the real one is `loopMachine`, which is already the
   `GenericCrafter` default — the bad value was *overriding* it with silence).
   *Fix:* mapped every unit to an existing sound (shootDuo / shootSalvo /
   shootTank / shootArtillerySmall / shootArtillery / shootOmura / shootSpectre),
   Biophage → `shootScatter`, removed the vat's bad `ambientSound`.

2. **No block consumed any power.** Top-level `consumePower` is a *method* in
   `Block`, not a field — v154 only reads power from `consumes.power`, so all
   five powered blocks ran for free.
   *Fix:* moved into `consumes` (constructor 1.5, recombinator 3, mutagenic 6,
   apex 12, vat 1.5 — calibrated against vanilla factories/reconstructors).

3. **In-game names showed raw ids ("raize-larva").** `displayName` is not a
   recognized JSON key; the localized name comes from the JSON `name` field
   (`readBundle` puts it into the bundle).
   *Fix:* set `"name": "Raize Larva"` etc. in all 21 content files.

4. **Nanite Infection did 10× the intended damage.** `damage` on a StatusEffect
   is *per frame* (0.8 → 48 DPS); `damageInterval` and `statusDuration` are not
   StatusEffect fields (ignored).
   *Fix:* `intervalDamage: 0.8` + `intervalDamageTime: 10` (4.8 DPS, as
   designed); removed the invalid keys from both statuses.

5. **The "custom" Spore Pod item never existed — the file was patching the
   vanilla item.** `locate()` resolves vanilla names *first*, so
   `content/items/spore-pod.json` was silently applied to vanilla `spore-pod`:
   recolored purple→green, flammability 1.15→0.1, +0.2 explosiveness, cost 0.
   Meanwhile every `"spore-pod"` reference in the mod already resolved to the
   vanilla item, and nothing could ever produce a mod-specific pod.
   *Fix:* deleted the override and its dead sprite; the mod now (correctly)
   uses the vanilla Spore Pod everywhere.

6. **Every unit rendered broken weapon sprites.** Weapons look up sprites
   strictly by `raizecannon-<weapon-name>` with no fallback (`Weapon.load()`),
   and no `-weapon.png` files shipped → error sprites on all 12 units.
   *Fix:* generated 12 palette-matched weapon sprites (fangs for crawlers,
   dart launchers for mechs, stingers for fliers) with baked outlines.

7. **Biophage's three-shot salvo didn't work.** `shots`/`shotDelay`/`spread`
   are not v154 `Turret` fields (moved to `ShootPattern` in v7).
   *Fix:* `"shoot": {"type": "ShootBarrel", "shots": 3, "shotDelay": 5}` and
   kept `inaccuracy` for the spread.

8. **Biophage wasted ammo on ground targets.** `FlakBulletType` sets
   `collidesGround = false`, but the turret targeted ground by default.
   *Fix:* `targetGround: false` (pure AA flak, consistent with its research
   parent `scatter`).

9. **The 9 upgraded units had no tech-tree nodes.** They only unlocked by
   being produced.
   *Fix:* added `"research": {parent: <previous tier unit>, requirements}` to
   each, producing a full chain rooted at the Nanite Constructor.

Also removed dead/invalid fields: `crushFragile` (not a `UnitType` field),
`density` (not a `Liquid` field in v154), `hasPower` (implied by the consumer).

## Balance notes (left as designed)

- Ground units are faster than vanilla ground units (0.72–1.35 vs 0.3–0.62)
  but comparable to vanilla crawl units (elude 1.8).
- T1 units are strong vs vanilla T1 (larva 420 hp/18 dmg vs dagger 150/9);
  the mod's T4 (4.2–6.8k hp) sits around vanilla T3. Internally consistent.
- Research costs equal the units' upgrade costs (vanilla units use a 0.5×
  multiplier on factory plans; mod T1 costs are the plan costs 1:1).

## Validation performed after fixes

- All JSON parses; every field name source-verified for v154.
- Every content reference resolves (mod-first vs vanilla-first rules applied).
- Every sound name exists in `core/assets/sounds` at v154.
- Block sprites exactly `size×32`; reconstructor variants present.
- Upgrade chains and the full research tree verified programmatically.
- Rebuilt zip: `mod.hjson` at root, 67 files, integrity-checked.
