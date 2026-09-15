# Hunt Dice

Single-file HTML page that randomizes multiplayer **Hunt or Die** settings for
*theHunter: Call of the Wild*, so two players stop arguing about which map,
weapon class, and bounty animal to play.

## Purpose

Rolls a random, **guaranteed-compatible** combination of:

- **Map** — from the maps you check (free maps default on, DLC off)
- **Time of day** — Day or Night
- **Armament class** — from the options you check
- **Bounty animal** — picked from the rolled map's roster, filtered to species
  that are legal with the rolled armament class

The constraint engine never rolls an impossible combo (e.g. small-game bow vs
moose, shotgun vs deer). If a selected map+armament pair has zero valid
bounties, that pair is simply excluded from the roll space; if *nothing* is
valid, the Roll button disables and a hint explains why.

## Usage

Open `index.html` in a browser (double-click works — no server, no build, no
dependencies). Or put the repo on GitHub Pages and share the URL; both of you
open the same page, you each toggle the maps you own, and one person rolls.

1. Tick the **maps you actually own** (DLC maps start unticked).
2. Tick the armament classes you're both willing to play.
3. Hit **ROLL**. Read the result aloud and build the lobby around it.

Your selections are saved to a cookie and restored on the next visit, so you
don't re-tick your DLC/maps each time. The cookie is per-browser, so on a
shared machine each person's own browser remembers its own toggles.

## File layout

| File | What |
|------|------|
| `index.html` | The entire app: styles, data, constraint engine, UI. ~600 lines, zero dependencies. |

That's the whole repo. One file so GitHub Pages needs no config.

Everything lives in three tables at the top of the `<script>` block, all
sourced from the official wiki:

- **`CLASS`** — every huntable species → its ammo class(es) in the game's
  1–9 scale (e.g. Hare `1-2`, Chamois `3-4`, Moose `8`, Elephant `9`).
  A few species have two legal ammo classes and are listed as such.
- **`MAPS`** — all 19 reserves (2 base game — Hirschfelden, Layton Lake District —
  + 17 DLC, per the wiki's Reserves list), each with its species roster.
- **`WEAPONS`** — the 8 armaments, each with its ammo-class ranges exactly
  as the game casts them (small game rifle 1-6, large game rifle 4-9,
  small game bow 1-3, large game bow 2-4, small game pistol 1-2,
  large game pistol 2-9, shotgun 1-9, bow any 1-9).

`canHunt(weapon, species)` = the species' class(es) intersect the
weapon's ranges. No special cases: the wiki says a shotgun legally
shells class 1 through 9, so it hunts everything, including elephant.

| Armament | Ammo classes |
|----------|--------------|
| Small game rifle | 1-6 |
| Small game bow | 1-3 |
| Large game rifle | 4-9 |
| Large game bow | 2-4 |
| Small game pistol | 1-2 |
| Large game pistol | 2-9 |
| Shotgun / fowling | 1-9 |
| Bow (any) | 1-9 |

Note: **"Free" in the game means base-game:** per the wiki, only Hirschfelden and
Layton Lake District ship with the base game; every other reserve is a DLC
pack (playable for free when a DLC-owning friend hosts). The tool defaults
the two base maps on and all 17 DLC off.

**African Safari is not a Call of the Wild reserve** — it belongs to
the original 2016 *theHunter*. COTW's African map is Vurhonga Savanna,
which is in the tool. The wiki page for "African Safari" doesn't even
exist.

Roll = pick a valid (map, armament) pair, then a valid species on that map,
then Day/Night.

## Editing the data

**Add a map** (new DLC release): append one object to `MAPS`:

```js
{ name:"New Preserve", free:false, species:["Species One","Species Two"] },
```

and add any new species to `CLASS` with its ammo class(es). If a species on
a map is missing from `CLASS`, the page still works — it just can't offer
that species as a bounty, and the result notes list every skipped species so
gaps are visible instead of silent.

**Change legality rules:** edit a weapon's `ranges` in `WEAPONS`. That's the
only place weapon legality is encoded; `canHunt` is just a range check.

## Verification

The constraint logic is validated headless (node, DOM stubbed): exhaustive
weapon × ammo-class (1-9) legality matrix, every one of the 127 roster
entries checked, per-map coverage, and 20,000 randomized rolls asserting
every bounty is on its map and legal for its weapon. All pass. The only
dead pair is Salzwiesen Park × large game rifle (all its species are
class 4 or below), which the engine reports honestly instead of rolling it.

## Data provenance

- Map rosters + species classes: theHunter: COTW wiki (fandom) — every one of
  the game's 134 animal pages was pulled via the MediaWiki API; each species'
  infobox `locations=` field is the game's own reserve placement, so rosters
  are built from it (cross-checked against per-reserve class tables — zero
  conflicts). Covers all 19 reserves including the 2025/2026 DLC
  (Askiy Ridge, Tòrr nan Sithean, Intisuyu). (the game's own species→map
  placement), cross-verified against each reserve page's animal table.
  The two sources agree 100% on classes; the infobox is the superset
  (it includes seasonal species some reserve tables omit).
- Species ammo classes: per-species wiki infoboxes (`class=` field), all
  75 species. Both wiki sources (infobox + reserve tables) agree with
  zero conflicts.
- Weapon legality: the wiki's weapon pages — each weapon's exact
  ammo-class range (e.g. Rifles: small game rifle 1-6, large game
  rifle 4-9).
- "African Safari" was removed: it is not a COTW reserve (no wiki page;
  it's a map from the original 2016 theHunter).
- Reserve emblems: 12 of 19 hex logos are the official in-game logos, hotlinked
  from the fandom wiki (see `MAP_ICONS` in the script). The other 7 reserves
  have no logo file on the wiki, so they render as a tinted hexagon monogram
  instead. Every emblem sits on a monogram fallback, so if a hotlinked wiki
  image ever 404s or gets taken down, the chip degrades to the monogram rather
  than showing a broken box.
