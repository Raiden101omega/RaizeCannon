# Raizecannon 2.2

Biotechnology expansion for Mindustry (build 146+ / 154).

Drop this folder (or a zip of it) into Mindustry's `mods/` directory and enable **Raizecannon**.

## What was fixed

The 2.1 pack could load, but several systems were not actually valid Mindustry JSON:

- **Biophage** used the removed v6 turret fields (`shots`, `shotDelay`, `spread`) and `FlakBulletType`'s default `collidesGround: false`, so it neither salvoed nor damaged ground targets. It now uses a `ShootSpread` pattern and flak that hits air **and** ground.
- **Power** was declared as `consumePower` (not a JSON field). Every factory, reconstructor, and the culture vat now uses `consumes.power`.
- **Liquid rates** were written as if they were per-craft, but Mindustry consumes/outputs liquids **per tick**. The vat and unit line now use rates a pump can actually feed.
- **Reconstructors** never listed `upgrades` or item costs, so they could not promote units. Each tier now has explicit upgrade pairs and item consumption.
- **Display names** were stored in a non-existent `displayName` field. Names now come from `name` plus `bundles/bundle.properties`.
- **Custom `spore-pod` item** patched vanilla spore pods. The mod now uses the vanilla item.
- **Invalid sounds** (`shoot-small`) were replaced with real `Sounds` fields (`shootSnap`, `pew`, `spray`, `shootBig`).
- Apex units were larger than a 3x3 payload (`hitSize` 27), so Colossus Worm could not enter the reconstructor. Hit sizes are capped at 22.
- Weapon sprites were missing (pink boxes). Each unit now has a matching weapon region.

## Content

### Production
1. **Nanite Culture Vat** — spore pod + plastanium + water → Parasitic Nanites
2. **Nanite Constructor** — T1: Raize Larva, Raize Stalker, Raize Wasp
3. **Nanite Recombinator** — T2: Brood Crawler, Spine Hunter, Miasma Wasp
4. **Mutagenic Reconstructor** — T3: Carapace Beast, Plague Stalker, Venom Wing
5. **Apex Reconstructor** — T4: Colossus Worm, Necrotic Warden, Hive Mother

Feed reconstructors from the previous factory with payload (place the reconstructor on the factory output, or use payload conveyors).

### Turrets
- **Biophage** (item) — 3-round flak salvo. Ammo: spore pod, plastanium, surge alloy.
- **Nanite Thrower** (liquid) — only accepts Parasitic Nanites; applies Nanite Infection.

### Status
- **Spore Sludge** — slow / reload penalty (Biophage spore ammo).
- **Nanite Infection** — DoT, slow, reduced max health. Raize units are immune.
