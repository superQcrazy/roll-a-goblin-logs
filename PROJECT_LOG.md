# Project log — Roll A Goblin

This file has two parts:

1. **Game overview**: the full state of the game. Update it when something big changes.
2. **Log**: one entry per finished prompt, newest at the top (see `CLAUDE.md`).

---

# 1. Game overview (checked in Studio on 2026-10-05)

## The game in short
**Roll A Goblin!** is a Roblox pet-RNG game, placeId 106895515727117. Working names were "Spin a Goblin" and "skiavivada;f". The place is a Team Create.

The core loop: press ROLL → the server picks a goblin by rarity → it goes into your inventory → you equip up to 6 goblins, which follow you as pets in a formation.

Around that loop there are coins, luck, a Zone 2 gate, an index, settings and teleport. The map is Zone 1, Fernwood Forest.

- **Source of truth:** Roblox Studio (Edit). The local `.luau` copies in `Goblin_Pets/` are older.
- **Font:** Montserrat everywhere (weight may vary).
- **References:** never copy an outside reference 1:1. The exception is when the user explicitly asks for a literal 1:1 copy of their own mock-up, as for the HUD on 2026-10-06 (`HUD_Mockup/`).

## Goblins (42 designs)
- **Catalog:** `ReplicatedStorage.GoblinConfig`, which holds rarities, odds, colours and all designs.
  - Each design has: Id, Name, Rarity, Image, RevealImage, Description, EquipEffect and Theme colour.
- **Models:** `ReplicatedStorage.GoblinModels`, plus `FollowMotion`, a StringValue holding the Idle/Walk/Run/Wave motion sampled from Blender.
- **Rarities and odds:**

  | Rarity | Odds | Goblins |
  |---|---|---|
  | Common | 1/2 | Basic, Leaf, Angry, Sleepy, Pebble, Shy, Muddy |
  | Uncommon | 1/4 | Scout, Mushroom, Tinkerer, Bramble, Crystal |
  | Rare | 1/8 | Alchemist, Hoarder, Brawler, Mystic |
  | Epic | 1/16 | Knight, Pyro, Bandit, Shaman, Chef, Engineer |
  | Legendary | 1/32 | King, Cyborg, Necromancer, Assassin, DragonRider |
  | Mythic | 1/64 | Demigod, ShadowStalker, Volcanic, CrystalLord, Stormbringer |
  | Divine | 1/100K | Seraph, Zeus, Eternal, Phoenix |
  | Celestial | 1/200K | Nebula, GalaxyWalker, Supernova, Eclipse, Starlight, VoidWatcher |
  | Astral | 1/300K | Wayfinder, Skyweaver, Nacre, Runescribe |
  | Cosmic | 1/400K | Polaris, Farwatch, Deepwake, Outrider |
  | Transcendent | 1/500K | Resonant, Silkenveil, Ambrosia, Zephyr |
  | Primordial | 1/600K | Resinborn, Ivory, Kilnborn, Flintshade |
  | Eternal | 1/700K | Moorbound, Tollkeeper, Signet, Vigil |
  | Infinite | 1/800K | Helix, Labyrinth, Flowbound, Recursor |
  | Omniversal | 1/900K | Riftwarden, Puppeteer, Manyfold, Ringmaster |
  | Absolute | 1/1M | Bulwark, Arbiter, Caliber, Cipher |
  | Beyond | 1/1.1M | Veilkeeper, Nullborn, Liminal, Unwritten |

  (odds set by the user 2026-10-10.) Divine and rarer have their own text gradient + outline (`GoblinConfig.RarityGradient` / `RarityStroke`). Astral → Beyond only hold v4 goblins (`V4Only`): they are rolled only while the v4 models are in the game (now: the Studio test).
- **Model rules (`AGENTS.md`):**
  - Every goblin is a re-skin of the Basic anatomy (`Common_Goblin/common_goblin.blend`) with the same rig of 18 bones.
  - No pickaxe in the templates. The old pickaxes are in `ServerStorage.GoblinMiningTools`.
  - Capes and wings attach to the back.
- **Blender:**
  - **Since 2026-10-10 the library is split** (one file with 12,400 objects lagged; the user wanted about 9 goblins per file):
    - `Goblin designs.blend` = the **base library**: the original designs (= the current Roblox goblins, `<Name>_Goblin`), `Basic_Goblin`, `Leaf_Goblin`, `Preview_Lights` and the shared Actions (Walk / Wave / …). About 2,400 objects.
    - `Goblin_Library/` holds the rows. Each file also has `Basic_Goblin`, `Preview_Lights` and the Actions:

      | File | Rows | Goblins | Objects |
      |---|---|---|---|
      | `00_concepts_archive` | Concepts v2 + v3 | 10 | 487 |
      | `01_v4_batch1_Uncommon` | `Goblin_Redesign_v4` (Mushroom, Scout, Tinkerer, Bramble) | 4 | 497 |
      | `02_v4_batch2_Common` | `_B2` (Basic, Leaf, Angry, Sleepy, Pebble, Shy, Muddy, Crystal, Alchemist) | 9 | 647 |
      | `03_v4_batch3_Rare_Epic` | `_B3` (Hoarder … Engineer) | 9 | 1,852 |
      | `04_v4_Legendary_Mythic` | `_B4` + `_B5` | 10 | 1,437 |
      | `05_v4_Divine_Celestial` | `_B6` + `_B7` | 10 | 1,343 |
      | `06_new_Astral_Cosmic` | `Goblin_New_Astral` + `_Cosmic` | 8 | 910 |
      | `07_new_Transcendent_Primordial` | `_Transcendent` + `_Primordial` | 8 | 966 |
      | `08_new_Eternal_Infinite` | `_Eternal` + `_Infinite` | 8 | 806 |
      | `09_new_Omniversal_Absolute` | `_Omniversal` + `_Absolute` | 8 | 803 |
      | `10_new_Beyond` | `_Beyond` | 4 | 351 |
    - The table lives in `Goblin_Concepts/goblin_files.py`.
    - `Design()` (in `redesign_v4.py`, used by every v4 and new-goblin script) **refuses to build a row in the wrong file** (`check_file`).
    - It **appends a missing source design** (e.g. `Knight_Goblin`, `Basic_Goblin`) from `Goblin designs.blend` (`ensure_collection`). So to rebuild a goblin, open its `Goblin_Library` file and run its script there.
    - `export_v4_specs.py` now merges into the existing `physics_parts.json`: run it in every file that changed.
    - The split is made by `Goblin_Concepts/split_library.py` in a background Blender. Full backup of the old single file: `Goblin designs_backup_before_split.blend`.
  - `Goblin_Variants/<name>/` holds a README per variant.
  - `Leaf_Goblin/` and `Common_Goblin/` hold the build scripts.
- **v4 redesigns** (2026-10-10; all 42 existing goblins done; exported as FBX, not imported in Studio yet):
  - Each goblin is a copy of its design: the same body and anatomy, plus its own armature copy with extra accessory bones.
  - New detailed pieces are separate objects, skinned 100 % to one bone.
  - Rows `Goblin_Redesign_v4` (batch 1), `_B2` … `_B7` (one row per rarity up to Celestial). They live in `Goblin_Library/01…05_*.blend` (see Blender above).
  - Scripts: `Goblin_Concepts/redesign_v4.py`, `redesign_v4_batch2.py` … `redesign_v4_batch7.py`, with the shared `redesign_v4_kit.py`.
  - Hand-off for Roblox (generated by `export_v4_specs.py`, which covers the v4 and the new goblins, 78 in total):
    - `Goblin_Concepts/physics_parts.json` (version 2): bones, physics candidates, held arm poses (`arm_hold`), held bone rotations (`leg_pose`: legs / head), hover and tilt poses, `FX_*` effect bones;
    - `Goblin_Concepts/V4_GOBLIN_SPECS.md`: the same, readable.
  - Exported as FBX (merged per bone + colour; see "FBX export" below). Roblox still has to do: import, hold the arm / leg / head poses, add the secondary motion / physics and the effects.
- **36 NEW goblins** (2026-10-10, all built; exported as FBX, not imported in Studio yet), for the rarities Astral → Beyond:
  - Astral: Wayfinder, Skyweaver, Nacre, Runescribe.
  - Cosmic: Polaris, Farwatch, Deepwake, Outrider.
  - Transcendent: Resonant, Silkenveil, Ambrosia, Zephyr.
  - Primordial: Resinborn, Ivory, Kilnborn, Flintshade.
  - Eternal (rarity; separate from the Divine "Eternal" goblin): Moorbound, Tollkeeper, Signet, Vigil.
  - Infinite: Helix, Labyrinth, Flowbound, Recursor.
  - Omniversal: Riftwarden, Puppeteer, Manyfold, Ringmaster.
  - Absolute: Bulwark, Arbiter, Caliber, Cipher.
  - Beyond: Veilkeeper, Nullborn, Liminal, Unwritten.
  - Each is a copy of `Basic_Goblin`: the same body, 19-bone rig and shared Actions, with its own armature. It has its own collection `<Name>_Goblin` with `<Name>_Rig`, `<Name>_Body` and the upright display parent `<Name>_Goblin_DisplayOffset`.
  - Rows `Goblin_New_<Rarity>` (y −29.4 … −53.4, 3 apart), in `Goblin_Library/06…10_new_*.blend` (two rarities per file). Scripts `Goblin_Concepts/goblin_new_<rarity>.py` (single goblin: `ONLY = ("Name",)`), with the shared kit `goblin_new_kit.py`:
    - `new_design`;
    - garment helpers `panel` (use `side="back"` for back panels!), `face_y`, `sleeve`, `coat_skirt` (F/B/S parts, `trim2`), `torso_sides`, `boots`;
    - `bend_knees`, `finish`.
  - Rig custom properties:
    - `v4_hold`: held arms;
    - `v4_legs`: held bone rotations for legs and head;
    - `v4_pose`: hover / tilt;
    - `v4_magic`: deliberately floating bones;
    - `v4_fx`: effect bones.
  - Goblins with a held pose have no Action on their display rig. The others keep `Goblin_Idle`.
- **FBX export for Roblox** (2026-10-10): `Goblin_Concepts/export_goblins_fbx.py` (background Blender) writes `Goblin_Export/v4/*_v4.fbx` (42), `new/*.fbx` (36) and `admin/Sprout.fbx` + `Pip.fbx`.
  - Sprout and Pip are the concepts `Concept_Sprout` / `Concept_Pip` from `00_concepts_archive`, meant as **admin pets**.
  - It merges pieces per bone + colour (64×64 palette PNG per goblin; atlases kept), keeps all bones including `FX_*`, and uses the original rig convention (origin, bind pose).
  - It also writes `Goblin_Export/export_manifest.json`. Per goblin, this file includes `transparent` (part → Transparency) and `glow` (part → emission colour), which have to be applied in Studio.
  - **In Studio (2026-10-10): `ReplicatedStorage.GoblinModelsV4`** holds all 80 as ready templates, named like `GoblinModels` (`Basic`, `Leaf`, …, `Wayfinder`, …, `Sprout`, `Pip`).
    - Each is built the same way as the current templates: `ScaleTo(GoblinModels.Basic:GetScale())` (0.03) with the pivot at the origin.
    - Model attributes: `Rarity`, `Kind` (v4 / new / admin) and `SourceFbx`. Glow parts have the attribute `GlowColor` (Color3); nothing glows yet. The Alchemist flask has Transparency 0.82.
    - Part names match the current export exactly. The first import had cut-off names (Studio truncates names over 50 characters, e.g. `Spine_Void Watcher Gobl... iris segment 0 emission`). They were mapped back by size and position.
  - The raw imports are in `ServerStorage.GoblinImport_Raw`, with extra copies in `Duplicates`. They were moved out of the Workspace, where they floated huge above the map; they are kept so Studio can re-import them.
  - **Test in Play:** the ReplicatedStorage attribute `GoblinV4Test` = true (Studio only; it is on now) makes `GoblinPets.GetTemplate` take models from `GoblinModelsV4` first. Without it, the game uses the old `GoblinModels`; the live game is unchanged. `GoblinConfig.V4Active()` = that check (used by the icons and mutation layers below).
  - **v4 icons (2026-10-10), all 80:** `GoblinConfig.V4_ICONS[Id] = { Image, RevealImage, IconBright }`, applied when `V4Active()` (and to the test designs of the new goblins / admin pets). The roll result = RevealImage + the shared GoldImage (`FinalImage` nil).
    - Made by `Goblin_Concepts/render_v4_icons.py` (background Blender per library file, OptiX GPU; the old roll camera / lights / pose relative to each goblin's display parent, 2404 px whose centre 1600 = the old frame, calibrated on the base Basic) and `Goblin_Concepts/make_v4_icons.py` (old shared inventory crop → 512 px `inventory/`, old character-layer transform → 1024 px `reveal/`, Shiny light sweep 512 px `bright/`). Output + ids: `Goblin_Icons_V4/` (`asset_ids.json`, `icons_report.json`, `preview/inventory_sheet.png`).
    - Goblins whose accessories stick out of the old crop are scaled down around their feet (Seraph 0.78, Scout / Skyweaver / Liminal / Puppeteer / Resonant ~0.8, …; the rest 0.85–1); the admin pets are scaled up to Basic's height (~1.43).
  - **Mutations on v4 models:** `ReplicatedStorage.GoblinMutationAssetsV4` (built in Edit from `Goblin_Icons_V4/v4_mutation_assets.luau`):
    - `Shiny.<Id>` = a Folder with one SurfaceAppearance per texture the model uses (body atlas + colour palette; attribute `Source` = the TextureID it replaces) and the attribute `IconBright`;
    - `Rainbow.<Id>` = the skin layer (TintMask) of the body atlas (`Source` = its TextureID); none for Sprout / Pip.
    - Textures: `Goblin_Concepts/make_v4_mutation_textures.py` (same Shiny filter / Rainbow holo as the old ones; skin colours of the 42 from their old Rainbow masks, of the 36 new from their head faces via `Goblin_Concepts/v4_skin_colours.py`). The metal / roughness maps are the shared uniform 4×4 ones.
    - `GoblinMutationFX`: on models with `Kind`, Shiny goes on every mesh by its texture, Rainbow on every mesh with the skin atlas (each with its own wave height); `bodyOf` picks `<Id>_Body` (admin pets: the biggest mesh). `RollClient` reads the v4 `IconBright` when `V4Active()`. Huge needs nothing extra.

## Rolling
- **Server: `ServerScriptService.RollServer`**
  - Uses the RemoteFunction `ReplicatedStorage.RollGoblinFunction`.
  - Cooldown of 3 seconds.
  - Picks a rarity by weight. Luck multiplies the weight of every non-Common rarity.
  - Then picks a design within that rarity and awards it through GoblinInventory.
  - Can refuse a roll: cooldown, still loading, or inventory full.
- **Client: `StarterGui.RollGui.RollClient`**
  - The ROLL button is styled like the HUD mock-up:
    - 272 × 86, deep green (88,188,62) → (2,143,42) with a (0,66,12) edge;
    - a see-through white dice and off-white "ROLL";
    - a bounce and a shine sweep.
  - **Auto Roll / Hide Rolls** are dark round buttons with a coloured ring (green / blue) and a glowing icon, at ±189 next to ROLL. They only exist in the full-screen roll (`spotlight`, RollGui attribute `Spotlight`).
    - ROLL opens the full-screen roll with both buttons.
    - Hide Rolls (`setHidden(true)`) leaves it: Auto keeps rolling small at the top, the buttons go away, and ROLL is light green again (no busy shade).
    - ROLL then brings the full-screen roll back (`setHidden(false)`) without starting an extra roll while one is running or Auto is on, so Auto / Hide can be switched off.
    - A Rainbow result switches Hide Rolls off for that roll (full screen). The next roll switches it back on (`MutReveal.restoreHidden`); pressing Hide Rolls yourself clears that flag.
  - **Speed:** `BASE_SPEED = 2` times the player's `RollSpeed` (1–20). All roll timings are divided by it, so a new player rolls at what used to be 2x (~3.7 s per roll). All columns land at the same moment (`SLOT_STAGGER = 0`).
  - **Auto Roll:** the icon spins while it is on.
  - **Hide Rolls:** shows the roll small at the top, 0.54x size.
  - **Roll animation:**
    - Slots cycle through goblin icons with a black outline (2.2 px since 2026-10-09) and a themed aura. All icons are kept in GPU memory by the `TextureKeeper` (1×1 labels in RollGui); the cycle only shows icons whose keeper `IsLoaded`.
    - The odds ("1/32", the same text as the world tags) sit at the top left of the icon, tilted, in the rarity colour; the name sits under the icon.
    - They land on the server result with a flash and the gold glow layer.
    - The goblin then flies into the Inventory button.
    - **Mutation reveals** (`MutReveal` in RollClient):
      - Shiny: a light beam with a shimmer sound (`ShinyShimmer1` = 9116394545), then a ding + chime cluster when the glow appears (`ShinyTing1/2` = 9126073001 / 9116395089, since 2026-10-08).
      - Default styles (`MutReveal.DEFAULT_STYLE`, when the player attribute is not set): Huge 4 Titaan-zoom, Rainbow 2 Supernova (since 2026-10-08, was 3).
      - Odds (2026-10-09): each reveal counts the odds up to the mutations shown so far. A Shiny Rainbow (or Huge Shiny Rainbow) first counts to the Shiny (Huge + Shiny) odds, and only the Rainbow reveal brings the final odds (`shownMutations`, `slot.oddsAt` in RollClient).
  - `GoblinPets.ROLLS_PER_SPIN = 1` sets how many goblins one roll gives (it can be raised).
  - While a roll plays, `RollGui` has the attribute `Rolling = true`, and menu panels cannot open.
  - **Clover Coin / Coin Rain (2026-10-06):** a roll result can be a Clover Coin instead of a goblin.
    - The chance is a fixed 1/300; luck does not change it.
    - 3 coins fill the COIN RAIN bar right of ROLL, and the 3rd coin starts a 15 s Coin Rain with 30x luck.
    - A coin rolled during the rain is a BONUS: the rain's luck goes up to 100x and its time starts over.
    - See the log entry "Clover Coin and Coin Rain".
  - **Luck clover:** sits centred above ROLL (UIListLayout, centred) and shows `1x` from the player attribute `Luck`. When luck goes up, a big clover animation counts up to the new value. The clover is hidden while a panel is open.
  - **New Best Roll cutscenes (2026-10-07, 8 more on 2026-10-08):** when a roll beats the player's rarest roll ever and is at least 1/100k (`GoblinPets.CUTSCENE_MIN_ODDS`), a cutscene plays (13 to choose from, plus the Mix).
    - Server: `GoblinInventory.OddsOf` / `RecordRoll` (`bestOdds` in the profile); RollServer sets `result.cutscene` and `result.prevBest`.
    - Client: `ReplicatedStorage.RecordCutscenes`. It is a module because RollClient is close to Studio's 200,000-character script limit; RollClient sets it up with its helpers, in a pcall.
    - Styles 7–14 live in `ReplicatedStorage.RecordCutscenesExtra`. RecordCutscenes passes it a helper table (camera director, puppet + poses, title card, sounds, stage pieces) in a pcall, adds them as `CUT[7..14]` and appends their names to `MutReveal.CUTSCENES`.
    - It plays right after the result lands, on a stage built locally high above the map, with its own lighting that is restored afterwards.
    - The goblin is a real animated pet (`GoblinPetRenderer.Puppet`).
    - The title card shows NEW BEST ROLL!, the name, the odds counting up, and the previous best.
    - After it, only the roll's end picture shows (`MutReveal.finale`).
    - **Tiers (the user, 2026-10-08, locked):** how many times rarer the record is than the previous best (`MutReveal.ratioOf`: rarity odds × `MutationOdds` / `prevBest`) picks the cutscene, whatever the style setting:
      - 20x → Toverportaal (9), 50x → Jackpot (13), 100x → Steencirkel (14), 200x → Supernova (12), 300x or more → Kristalgeode (2).
      - A first record without a previous best counts as 300x+.
      - Below 20x the `CutsceneStyle` plays (default: the Mix).
      - The table is `TIERS` in `MutReveal.cutscene` (RecordCutscenes).
    - **Settings > Filter Cutscenes** (slider Low … Max, 7 snapped steps, no numbers shown; replaces the Hide Cutscenes switch):
      - Low plays every record; each step up hides the next smaller tier (below 20x, 20x, 50x, 100x, 200x).
      - Max = no cutscenes, and the normal Huge / Rainbow / Shiny reveals play.
      - Player attributes `CutsceneFilter` (1–7) + `HideCutscenes`; saved as `settings.cutsceneFilter` (old saves with `hideCutscenes` become 7).
    - The style is the player attribute `CutsceneStyle`, 1–14 (the test panel "New Best Roll (test)" lists them in two columns):
      1. Schatkist
      2. Kristalgeode
      3. Rode loper
      4. Krachtexplosie
      5. UFO-bezorging
      6. Mix (the default): a random pick of 3 or 4 (since 2026-10-08 without the geode, which is the 300x tier).
      7. Meteoorinslag
      8. Drakenei
      9. Toverportaal
      10. Raketlancering
      11. Parelschelp
      12. Supernova
      13. Jackpot
      14. Steencirkel
    - The sounds come from `CUT_SFX` in four roles (build / reveal / title / count). The defaults are the user's picks 7 / 8 / 8 / 2; the test panel "Cutscene geluid (test)" can override them (`CutSfx_*` attributes).
    - The roll's end picture is built under the cutscene's last 0.9 s, so it already stands there when the cutscene ends.

## Inventory and equipping
- **Server: `ServerScriptService.GoblinInventory`** (ModuleScript)
  - Saves to DataStore `GoblinInventory_v1`, key `u_<UserId>`, with a session lock, retries and autosave.
  - In Studio without API access it falls back to a memory-only profile, so nothing is saved.
  - **Profile contents (v2 since 2026-10-07):**
    - stacks: one counter per design + mutations (`stacks[key] = { g, mu, n, t, f }`, key `"Zeus"` / `"King|Huge,Shiny"`). v1 item lists are converted on load.
    - the equipped list (max 6 stack keys; slot ids `"<key>#<k>"` for the client)
    - settings: music, sound, showPets
    - coins
    - luck
    - zones
    - index: every design ever discovered
  - **API:** Award, AwardMany, SetEquipped, UnequipAll, EquipBest, Delete, FreeSlots, PickDesign, GetSnapshot, SetSettings, SetLuck, AddCoins, HasZone, UnlockZone, SaveAll, autosave/shutdown.
  - **Limits:**
    - `MAX_EQUIPPED = 6`
    - `MAX_INVENTORY = 20000000` (total goblins; the data stays small because it is counters)
  - **Replication to the client:** the player attributes `EquippedGoblins`, `Coins`, `Luck` and `Zones`.
- **Remotes:** `ServerScriptService.GoblinInventoryServer` creates them in `ReplicatedStorage.GoblinInventoryRemotes`: GetInventory, SetEquipped, InventoryChanged, SaveSettings, DeleteGoblins, EquipBest and ZoneResult. Requests are rate-limited.

## Pets in the world (client-side)
- **`ReplicatedStorage.GoblinPetRenderer`:**
  - Every client renders the pets of every player itself.
  - Follow logic uses smoothDamp, ground raycasts and a hold-still that keeps the pet behind the player.
  - Slots are assigned to formation positions by shortest distance (`GoblinPets.Formation`, 1–6 pets).
  - The model tilt is corrected.
  - Animation is driven bone by bone from `FollowMotion`, including random waves.
  - Each design has its own equip effect (e.g. Basic = Gust, Leaf = Leaves); new goblins appear only after the inventory closes.
  - "Show Other Pets" can turn off rendering of other players' pets.
  - **Ground rays ignore enemies** (every model with `EnemyId`), so goblins never climb onto them.
  - **Ground = the visible surface (`groundAt`, `visParams`).** Foliage, rocks and invisible floors are skipped by name / transparency (`SKIP_NAMES`, `skipPart`). Since 2026-10-09 decor that streams in later is added too (debounced `refreshRayFilter`).
    - Parts with the attribute `GoblinGround` are always ground. These are the path tiles `V4_*_Pavers_NoCollide` (CanCollide + CanQuery, so the player also walks on them) and the stepping stones `V4_*_PathStones_NoCollide` (CanQuery only). Both use PreciseConvexDecomposition collision. Every ring, shadow and FX of a goblin follows `pet.groundY`.
    - **After a map re-import** these flags reset: set CanQuery / CollisionFidelity / the `GoblinGround` attribute again (and CanCollide on the Pavers).
  - **While fighting:** a path through the enemy's hitbox (+1.6) becomes a detour round it (`detour`).
  - **Knockback hop:** `pet.knock`, set by `GoblinAttacks.Hurt`, moves the goblin.
  - **No snapping (since 2026-10-06):**
    - Running to an enemy and back is always animated.
    - `TELEPORT_DIST` 250 is only a safety net (it used to be 60, so goblins far from their spot snapped).
    - Far from its spot a goblin sprints: up to 46 studs/s (`CATCHUP_FROM` / `CATCHUP_GAIN` / `CATCHUP_MAX`).
    - A real teleport of the player (≥ 40 studs in one frame, `PLAYER_JUMP`) brings the goblins along at once, in formation.
  - **Goblins walk through each other** (since 2026-10-06; the old pet-to-pet push `separate` is gone). They only keep off their own player (`keepOffPlayer`, `PLAYER_GAP` 2.6).
  - **After a fight** every goblin takes the formation slot closest to where it stands (`reslot`), so the paths back don't cross.
  - **Performance (2026-10-07):**
    - A pop-in goblin is built off-world and parented once, on reveal (`newPet(..., hidden)`).
    - Models move with one `BulkMoveTo` per pet (`moveModel`, equal to PivotTo).
    - The pop-in growth resizes at 30 Hz in shared frames (`REVEAL_SCALE_HZ`): any MeshPart size change near the camera costs ~10 ms of render CPU per frame.
    - Hidden goblins are built one per frame; `permute` uses branch and bound.
    - The pet hover's GUI hit test only runs when a goblin is under the mouse.
- **Mutations (since 2026-10-06), only Shiny is built so far:**
  - **Rules** in `GoblinPets`:
    - `GoblinPets.MUTATIONS` = { Shiny, Chance 200 }; each chance is fixed, luck never changes it, and the odds multiply with the rarity (a Shiny Rare is 1/1,600).
    - Functions: `CleanMutations`, `HasMutation`, `MutationOdds`, `RollMutations` (server), `DisplayName` ("Shiny Mystic Goblin"), `RatioText` / `OddsText(rarity, mutations)` with the total odds.
    - Now built: Huge 1/800 (1.6×), Rainbow 1/5,000 (since 2026-10-07, was 1/12,800). Planned: Glitched 1/3,200.
  - **Data:** each item has `mu` (a comma list, e.g. "Shiny").
    - `Inventory.Award(player, id, mutations)`; the snapshot items have `mu`; the equipped attribute entries have `m`.
    - EquipBest sorts by total odds.
  - **Rolling:** RollServer rolls the mutations per goblin and sends `mutations` in the results.
  - **Test hooks:**
    - `StudioGrantGoblins` accepts "Mystic:Shiny".
    - `ServerStorage` attribute `StudioForceMutation` = "Shiny" makes every roll Shiny.
  - **World look: `ReplicatedStorage.GoblinMutationFX`** (client, called by GoblinPetRenderer: Attach / Step / EquipBurst / Detach):
    - **Colour:** a white-gold layer over the goblin's own colours plus a bit of metal shine. It is a prepared SurfaceAppearance per design, `ReplicatedStorage.GoblinMutationAssets.Shiny.<Id>`, made in Studio because its maps cannot be set by a running game.
      - Textures come from `HUD_Mockup/make_shiny_textures.py` (each colour 30 % towards a warm white, 6 % brighter); metalness 71567097059396, roughness 75294398209894.
    - **Glint (since the lag fix):** every ~2.2 s the goblin lights up softly white (the rim Highlight's fill, 0.55 s, eased in and out, peak 0.62) and a big four-point star plus a few sparkles flash on its body. Only within 90 studs of the camera.
      - The first version, a band of Textures sliding over the body, made the game lag. **Rule:** never animate Texture offsets on the walking pets. Every change costs Roblox ~5 ms of render CPU per animated goblin (measured with `Stats.RenderCPUFrameTime`). Static textures, Highlight fills and particles cost nothing measurable.
    - **Extras:**
      - rising white-gold glitter (since 2026-10-09 it doesn't glow: LightEmission 0, 3/s);
      - **no light** (removed 2026-10-09, the user: "really minimal");
      - a faint white rim (Highlight, outline 0.78, at most 20 at once).
    - **Glint since 2026-10-09:** every ~3.6 s, fill peak 0.93 (barely visible). The ting star is smaller, and the ting and footstep sparkles hardly glow (LightEmission 0.1–0.15).
    - **Equip moment:** a glockenspiel "ting" plus a crystal shimmer, a soft flash (no Neon and no light since 2026-10-09), a ring of sparkles and a rain of stars.
  - **Light budget (`StarterPlayerScripts.GoblinPetLightBudget`, 2026-10-09; local copy `Goblin_Pets/GoblinPetLightBudget.client.luau`):**
    - Only the first `LIGHT_PETS` (2) goblins of each player, by model name, keep their PointLights; the rest are switched off. Six Celestial Rainbow Huge Shiny goblins had 24 lights (aura 1.2 / range 11, personality lights, Rainbow 0.42 / 8, Shiny) that washed everything out white.
    - It only switches lights off, and only turns back on what it switched off itself (attribute `_LightBudgetOff`). Particles and looks stay the same.
    - **Footstep sparkles:** at every step (pet.phase halves) a few twinkles stay on the ground.
  - **Nametag (GoblinTagFX):**
    - The name is "Shiny …", with "Shiny" in pale gold (#F5E3A1) and its own glint (a copy where only "Shiny" is visible).
    - The odds are the total.
    - `T_MAX` went from 8.8 to 12, so the longer name fits the plate.
  - **Inventory:**
    - A Shiny goblin is its own stack (key `g|mu`) and sorts before its normal version.
    - The card odds and the equipped-bar odds show the total; the tooltip shows "Shiny …" with the total.
    - **No Shiny visuals on the icons yet**, and the roll animation is unchanged: the user wants to design both together next.
  - **Hover cursor:** hovering your own goblin shows Roblox's own button hand (`UserInputService.MouseIcon` = `rbxasset://textures/Cursors/KeyboardMouse/ArrowCursor.png`) plus a black outline.
- **`ReplicatedStorage.GoblinV4Motion`** (2026-10-10): the animation layer for the v4 / new / admin models (`GoblinModelsV4`, model attribute `Kind`). Local copy: `Goblin_Pets/GoblinV4Motion.luau`.
  - **Data:** `ReplicatedStorage.GoblinV4MotionData` (local `Goblin_Pets/GoblinV4MotionData.luau`). It is generated by `Goblin_Concepts/export_v4_motion_data.py` from `physics_parts.json` plus the held poses read from the library rigs (scratch `held_poses.py`). Per goblin it holds:
    - hover and tilt;
    - held bone quaternions: `r` replaces the clip (held arms, and the legs of hovering goblins); `a` is added on top (wide stance / bent knees on grounded goblins);
    - secondary-motion hints.
  - **Renderer hooks** (`GoblinPetRenderer`):
    - `V4Motion.Attach` runs after FX.Attach and before the Huge ScaleTo.
    - `poseBones` takes the held poses. Hovering goblins glide: Idle clip only, no step hop.
    - Every frame: `V4Motion.Pre` (anatomy extras merged with `fx.extra`, plus `lift`), then `V4Motion.Post` after moveModel (the extra bones' Transforms).
    - Each goblin has its own stride (Ivory, Silkenveil, Vigil, Kilnborn), and `pet.noWave = true`: no random wave on the new models.
  - **Secondary motion** is classified per bone by name and data, then cached per template:
    - `skirt` = `<Coat|Robe|…>_<F|B|S><L|R><n>`. Pushed by the thighs; front panels never swing back into the legs, back panels never forward.
    - `chain` = capes `<Cape|Mantle|…>_<col><seg>`, tails, ribbons, hood tips. These are springs: inertia, air drag, breeze and a kick per step; lower links whip after their parent; limits depend on front/back position; cape columns sway together.
    - `hang` = pendulums. `wobble` = things that stand up. `wing` = flap and fold. `flicker` = flames.
    - Per-goblin signatures in the module's `SIG` table: `spin`, `revolve`, `hinge`, `clapper`, `open`, `tick`, `slide`, `flip`, `bob`, `pulse`, `scan`, `wave`; `blink` / `jaw` / `brow` for the admin pets' faces.
  - **Rarity flair** (anatomy):
    - Everyone breathes. From Uncommon they glance around; from Rare, grounded goblins shift their weight.
    - Body signatures: Farwatch spine scan, Outrider head scan, Runescribe writing, Signet stamping, Puppeteer string tugs, Unwritten / Deepwake drift.
    - **No idle flourishes and no spinning round** (removed 2026-10-10, the user: "niet zo rond draaien"; Helix's slow self-spin is gone too).
  - **Choreography** (`Motion.CHOREO` = table `K`, one entry per goblin key): a `run(c, t)` poses the goblin through the context `c`:
    - `c:rot(bone, x, y, z)` anatomy in degrees (+X raises a hanging arm forward / tips the head back, +Y turns to the goblin's LEFT, +Z swings the right arm out); `c:look`, `c:lean` (− = forward), `c:shift` (−Z = forward / towards the enemy), `c:up`, `c:crouch`, `c:arms`, `c:boost` (0..1: every moving part goes wild);
    - accessories: `c:spin(item, axis, deg)` round its centre, `c:swing(item, axis, deg)` round its pivot, `c:move(item, x, y, z)`; on top of the item's own motion, rigid items are reset afterwards.
    - Every term must be multiplied by an envelope (`W`, `Bl`, `U`) so it starts and ends at 0 (no pops). Throws with the right arm: wind-up turns the spine right (−Y), the throw turns it left (+Y).
  - **Click animation** (`V4Motion.Click`, from `GoblinPetFX.Click`): every goblin has its own themed click (`K.<Key>.click`, 2–3 s), e.g. Basic bindle swing, Angry tantrum, Ivory paw + tusk charge, Tollkeeper rings the bell, Unwritten conducts. Fallback `HOP`.
  - **Attacks** (`V4Motion.Attack`, from `GoblinAttacks.Play`, 2026-10-10): every goblin (all 80) fights with its own accessories, cooler with rarity (`attack(key, dur, hit, run)` section of the module). Examples: Ivory gores with the tusks on its back, Sleepy smacks with its pillow, Scout bonks with the backpack pan, Knight brings the shield round + overhead cut, DragonRider's dragon breathes fire, Volcanic's shoulder volcanoes erupt, Tollkeeper's bell bongs, Kilnborn's pot lid blasts off.
    - At most 0.95 s: all goblins strike together once a second (`CombatConfig.ATTACK_SPEED` = 1). The returned `hit` time drives the effects.
    - **Effects come out of the accessory:** `GoblinAttacks` `ANCHORS` table (`field = { bone, studs towards the enemy, studs up }`) + `liveAnchors(c, pet)` make `c.hand` / `c.chest` / `c.back` / … read the bone's live world position when the effect fires (e.g. Pyro's fireball from `FlameCore`, Cyborg's beam from `Cannon`, DragonRider fire from `DragonHead`, Crystal's lamp beam, Stormbringer bolts from its `Cloud`, Deepwake bubbles from both hoses). `c.hand` = the real right hand on every v4 model. Backup: `ServerStorage.UI_Backup.GoblinAttacks_before_accessory_fx`.
  - **Hover styles** for goblins with hover > 0 (the user still has to pick one; default `Motion.HOVER_STYLE = 1`):
    1. DRIFT: calm bob and sway; arms swing back loosely, legs kick in turn.
    2. GLIDE: superhero glide; arms sweep back, legs stream back together.
    3. SWIM: springy bob; breaststroke arms, pedalling legs.
    - Each pet's model attribute `HoverStyle` overrides; then workspace `V4HoverStyle`; then the default.
  - **GoblinPetFX on v4 models (`fx.v4`):**
    - Off: the old cape / wing / hat systems, `Fx:rig` (it returns an empty rig and detaches nothing), `def.hold`, `def.idle`, and the random personality gestures.
    - Kept: glows, emitters, aura, tag. A click goes to V4Motion.
  - **Cost** (measured 2026-10-10 with the 6 heaviest: Vigil, Tollkeeper, ShadowStalker, Riftwarden, Silkenveil, Phoenix): the layer takes ~0.3 ms Lua per frame; render CPU went 3.7 → 4.1 ms; FPS stayed at 120. LOD: every frame within 70 studs, 30 Hz up to 160, nothing beyond. In a fight with 6 goblins: 0.29 ms.
  - **Safety:** Pre / Post are pcall'd; an error switches the layer off for that one pet (warned once).
  - **Studio test hooks (workspace attributes):** `V4ClickTest` / `V4AttackTest` = a pet model name or goblin key (`*` = all, repeating) plays its click / attack motion (attack without effects); `V4ChoreoT` = seconds pins a running click / attack at that moment (pose check); `V4MotionOff`, `V4MotionMs`, `V4HoverStyle`.
  - **Follow tuning** (`GoblinPetRenderer`, 2026-10-10, smoother and quicker to react): `SMOOTH_TIME` 0.23 → 0.16, `START_DIST` 0.7 → 0.35, `LEAD` 0.5 → 0.85, `LOCO_BLEND_RATE` 12.6 → 16, `MOVE_TURN_RATE` 18 → 22, `PLAYER_DIR_RATE` 10 → 14, new `LEAD_VEL_RATE` 12 (was a fixed 6).
- **`ReplicatedStorage.GoblinPetFX`** with **`GoblinPersonalitiesA`** (Common–Epic) and **`GoblinPersonalitiesB`** (Legendary–Celestial):
  - Personality gestures, played only while the pet stands still.
  - Always-on effects per design.
  - A rarity aura under each pet.
  - One shared cape system and one shared wing system.
  - Test: `workspace:SetAttribute("GoblinFXTest", "Pyro")` (client, Studio only). More in `Goblin_Pets/PERSONALITIES_NL.md`.
- **`ReplicatedStorage.GoblinTagFX`** with **`GoblinSignaturesA/B/C`:**
  - One nametag style for every goblin: a dark plate with rarity colour, and the name and odds in Montserrat.
  - The overall size is set by `TagFX.SIZE`, which has been 1 / 1.3 since 2026-10-06.
  - Above it, a small signature-effect zone per goblin.
  - Tests: `GoblinTagTest`, `GoblinTagLevel` and `GoblinTagCrowd`.
  - Sprites are built by `Goblin_Variants/Nametag/make_nametag_sprites.py`.

## HUD and menus (`StarterPlayerScripts.GoblinPetClient` + `ReplicatedStorage.GoblinUI`)
- **Shared UI pieces live in `ReplicatedStorage.GoblinUIKit` (since 2026-10-09).** An element used on more than one page is ONE piece there, and each page passes only its texts. So far: `Kit.SortPill` (Goblins + Powers pages).
- **All texts live in `ReplicatedStorage.GameText` (since 2026-10-09).** Scripts use `UI.Text.<Section>.<Key>` (`UI.T`), set up in GoblinUI.
  - The user edits texts there.
  - New UI must put its texts there too, never hard-coded.
  - Content names stay in their configs (GoblinConfig, PowerConfig).
- **Theme:** bright cartoon panels, Montserrat, and hover/press animations.
  - Image IDs and helpers live in `GoblinUI`, including `UI.tiles`, `UI.aura`, `UI.outline` and particles.
- **Left HUD (`GoblinMenuGui.Dock`): a 1:1 copy of the user's mock-up (2026-10-06, sources in `HUD_Mockup/`).**
  - Six painted icons in a 2 × 3 grid: Inventory, Shop, Index, Settings, Teleport and Ascend.
    - All of them come from one atlas, `rbxassetid://128455311894694` (the gear has a solid dark hole): the top half holds the 2× icons and the bottom half their white silhouettes, used for the glint.
    - Each name sits under its icon in white Montserrat Heavy 18 with a black outline.
    - "x/6" sits over the backpack and pops when it changes.
  - Laid out in design px for a 772 px high screen (`DESIGN_H`), scaled down on smaller screens. It slides off screen while a roll plays full screen.
  - **Hover:** grow + tilt, a glow behind the icon, a glint and sparkles.
  - **Click:** squash + bounce with a burst per icon (stars, coins from the chest, book flap, a small gear kick of −28° that springs back, pin ripple, rising gold).
  - **Idle:** every few seconds one icon does a small animation.
  - RollClient flies rolled goblins to `Dock > … > "Inventory"` (the icon) and pops its `HoverScale`, which gives a sparkle burst.
- **Coin counter** (left, above the dock): "$amount" in flat gold, Montserrat Heavy 40 with a black outline.
  - Behind it is a fade that starts in the middle of the $ and reaches the right edge of the Shop icon; very long amounts make it longer.
  - Since 2026-10-09 the fade and the black "$" behind the amount are a lighter black (`COIN_BLACK` 40, 40, 47) and sit in a clipping frame `Coins.Shadow` (52 px high), so the bottom edge runs straight under the $ too.
  - It pops on change and a glint runs over it now and then.
- **Panels:**
  - One panel is open at a time, with a rotating rainbow edge.
  - Only the X or the dock icon closes a panel; clicking next to it does not.
  - Panels sit above the ROLL button, and the dim also covers the Roblox top bar.
  - Keyboard: G opens the inventory and Esc closes.
  - **Inventory** (layout since 2026-10-06, after the user's "My Pets" reference, not 1:1; panel 780 × 630 + an 84 px tab column):
    - **Tabs on the left** (`Stage.Inventory.<Key>Tab`, inside the panel's CanvasGroup; `makePanel(..., { left = INV_TABS_W })` makes the holder wider, the body keeps its size):
      - Goblins on top with the user's goblin-in-a-box icon (`rbxassetid://87311180912818`), then Potions, Titles and Items (placeholder icons made by `HUD_Mockup/make_inventory_icons.py`).
      - The open tab has the panel colour and runs into the panel (the `TabJoin` patch hides the panel outline there). The others are darker and tucked behind the panel edge. Hover = grow + tilt, click = bounce.
      - Items has no content yet: it shows `Body.SoonPage` ("Coming soon" + one line per tab, floating icon). Potions and Titles have their pages since 2026-10-10 (ext tabs, `GoblinPotionsInvClient` / `GoblinTitlesInvClient`, see "Potions" / "Titles").
      - The inventory always opens on Goblins with an empty search (`invUI.reset`).
    - **Title:** the dock backpack before "Inventory" (a wiggle + glint every few seconds).
    - **Search** (top right, `Body.GoblinsPage.Search`): filters the grid by goblin name or rarity (cell attribute `Search`). Shows "No goblins found" when nothing matches; the × clears it.
    - **Bottom bar** (`Body.GoblinsPage.BottomBar`):
      - Left: the **grid-view button** (`GridView`) and the pill "owned / 10,000" (backpack).
        - The icon is the user's **four cyan diamonds** (images/69.png). One diamond is cut 1:1 by `HUD_Mockup/cut_grid_gem.py` (asset 84985184296228) and shown four times.
        - Grid view makes the cards smaller (81 × 78, 8 per row instead of 6); the equipped bar keeps its size.
        - **Click:** a small ripple, where one after the other the diamonds pop and slide to their place.
        - **Grid view on:** the diamonds pack closer together (× 0.86), and a dark-blue plate with a breathing cyan rim stays behind the button.
        - `invUI.gridView`; it is not saved, and it starts off.
      - Right, one **Equip All / Unequip All** button: green "Equip All" while nothing is equipped (EquipBest remote), red "Unequip All" as soon as anything is equipped (`SetEquipped("*", false)` = UnequipAll).
    - Everything of the Goblins tab lives in `Body.GoblinsPage`. The newer parts are kept in the `invUI` and `eqBar` tables (the script is close to the 200 locals limit).
    - **Performance rules (2026-10-07):**
      - Card animations run only while the inventory is open and only for on-screen cards (`CARD.vis`, the pass "GoblinCardVisibility").
      - Never read `AbsolutePosition` / `AbsoluteSize` in a loop between writes (each read re-computes the whole GUI layout); batch the reads in a BindToRenderStep pass before First.
      - Cards are built off-tree, and big rebuilds continue over several frames (`invUI.buildGen`).
      - IconFX measures new icons in its read pass (`fresh`).
      - Off-screen cards have their content detached (`CARD.detach` / `CARD.attach`, pass "GoblinCardVirtual"). Code that looks inside a card later must allow for that: use `CARD.owner`, or keep references made at build time instead of `cell:FindFirstChild` later.
    - **Equipped bar** (`GoblinsPage.EquipBar`, 716 × 129): a **1:1 copy of the user's reference bar** (2026-10-06, `HUD_Mockup/make_equip_bar.py`, laid out in reference px × `K` = 716 / 1212).
      - One image holds the frame, the dark "x/6" pill and the cyan line.
      - "Equipped" and the count are TextLabels.
      - Each slot has a numbered badge (1–6) and a cyan pedestal.
      - Like the reference, **a goblin stands in its pedestal up to the hips**:
        - the image is 118 px, with the head top where the reference has it;
        - the pedestal is drawn in front;
        - everything below bar y `CUT` = 109.5 is clipped away (`eqBar.clip`, a ClipsDescendants frame), so the legs never show.
      - Equipped goblin:
        - a thin black outline and its own icon effect (`IconFX`);
        - a rarity glow behind it (`iconGlow(..., noSparkles)`; the star sparkles are gone);
        - the odds tilted -12° over the head, between the left ear and the head (`oddsTag`);
        - the gem top right.
      - **Gentle hover:** the goblin grows to 1.04 and tilts 1.5°, and the pedestal stays put. The user found the normal bounce too big: it lifted the goblin out of its pedestal.
      - Empty slots show a dark silhouette, clipped the same way.
      - Layers: Glows < Shadows < Goblins < Pedestals < Badges.
      - Icons are kept between refreshes and slide when the slots shift (`makeEquippedIcon(item, slot)`).
    - A grid where duplicates are stacked per design ("x5"), **always sorted by rarity** (rarest first, no sort button).
      - Equipping, deleting or a new roll keeps the grid's scroll position; opening starts at the top.
      - Each cell is a card (since 2026-10-06, the user's reference):
        - blue frame and lavender → mint background;
        - the goblin big with a thin black outline, from the head to the belt;
        - a hill in the rarity colour with "1/4";
        - top left: "x2". **Since 2026-10-10 equipped copies leave the grid.** The count shows only the free copies (34 owned, 1 equipped → "x33"), and a stack whose every copy is equipped disappears from the grid. The tick and green outline are no longer used (`anyEquipped = false`). Clicking a card equips one more copy.
        - **top right: the gem** (`CARD.gem`).
        - Pixel sizes scale with `gk` (0.75) in grid view.
        - Code: `CARD` + `makeCell` in GoblinPetClient.
    - **Gem** (`CARD.gem(parent, _, pos, size, z)`): the gold 4-point star with a blue stone, the same for every goblin.
      - Cut 1:1 out of the user's card reference (images/65.png) by `HUD_Mockup/cut_gem_ref.py`; the tips the reference cut off are closed again. Asset `136161945361156`.
      - It sways a little, and every few seconds a sparkle flashes on it (one RenderStepped loop over `CARD.gems`).
    - **Icon effects (`ReplicatedStorage.GoblinIconFX`):** every goblin's world effect drawn small on its icons.
      - Used in the grid, the equipped row, the index and the roll reveal (`IconFX.Attach(icon, goblinId, {frame = "inventory" | "final"})`).
      - New goblin: add `DEFS.<Id>`.
      - Visibility boosts sit at the top of the module: `SCALE_BOOST`, `PART_BOOST`, `ALPHA_BOOST`, `RATE_BOOST`, `MIN_PART_PX`, `MIN_PIECE_PX`, and a dark shadow under light particles.
    - Hover tooltip: name, rarity, odds, stats, description and owned count.
      - Only the button that owns the tip may hide it (`tipOwner`). Overlapping buttons, like the equipped goblins' boxes, fire the old button's MouseLeave after the new one's MouseEnter.
      - The equipped bar's glow, shadow, pedestal and badge layers have `Interactable = false`.
    - Trash (delete mode): the button is hidden (storage is unlimited); the code is kept. Click a goblin, then a slider sets how many copies to delete. Equipped goblins are never deleted.
    - An empty state with bobbing silhouettes.
  - **Shop:** "Coming soon".
  - **Ascend:** "Coming soon" (the button comes from the mock-up; its contents are still to be decided).
  - **Index:** every design, discovered or shown as a silhouette, with a rarity backdrop.
  - **Settings:** Music and Sound sliders, the Show Other Pets toggle and the Filter Cutscenes slider (Low … Max, since 2026-10-08; it replaced the Hide Cutscenes toggle). These are saved in the profile (`settings.cutsceneFilter` + `settings.hideCutscenes`; the client sets the player attributes `CutsceneFilter` and `HideCutscenes`).
    - GoblinPetClient is now ~193k characters, close to Studio's 200k limit: put bigger new parts in modules.
    - **Replaced on 2026-10-09** by the orange Settings panel `StarterPlayerScripts.GoblinSettingsClient` (local copy `Goblin_UI/GoblinSettingsClient.client.luau`; the old `settingsBody` is left in GoblinPetClient but no longer opened).
      - **External panels (how Settings / Teleport plug in, and how to add a new panel the same way):**
        - The client builds its panel as a CanvasGroup with the attribute `ExternalPanel` = key (+ `PanelColor`, a UIScale named `Pop`, a button named `Close`), and parents it into `GoblinMenuGui.Stage`.
        - GoblinPetClient registers it (the block after the `CloseMenus` handler) as `panels[key]`, so it behaves like Inventory / Index:
          - the same stage scale;
          - one panel open at a time, with the dim;
          - the halo of twinkling stars behind it;
          - the dock highlight;
          - closing with X / the dock icon / Esc / a full-screen roll;
          - `StudioOpenPanel` works.
        - The client refreshes on `PanelOpen` == key.
        - Sizes are in Stage px, where the Inventory body is 780 × 630: Settings 600 × 540, Teleport 780 × 540.
        - **GoblinPetClient is at ~198.9k characters** (Studio's limit is 200k; 2026-10-09). To open an external panel from any script, set the `GoblinMenuGui` attribute `OpenPanel` = key. Inventory tabs with `ext = true` draw their own page (see Powers): new panels must use this route, not code in GoblinPetClient.
      - Tabs: Audio / Gameplay / Display. Slim sliders (10 px track, 22 px knob) and small switches (60 × 30), a scrollbar with arrows.
      - Settings: Master / Music / SFX volume, Hide Other Goblins, Auto Attack, Manual Luck + Luck Cap (red warning when on), Hide Global Rolls, Filter Cutscenes, UI Scale 50–150 % (applied on release, scroll position kept), Performance Mode, Less VFX.
      - Saved via `SaveSettings` → `GoblinInventory.CleanSettings`. Player attributes: `MasterVolume`, `MusicVolume`, `SfxVolume`, `AutoAttack`, `ManualLuck`, `LuckCap`, `HideGlobalRolls`, `CutsceneFilter`, `HideCutscenes`, `UIScale`, `PerformanceMode`, `LessVFX`.
      - Server side: `RollServer` caps luck at `LuckCap` while `ManualLuck` is on; `CombatServer` only auto-targets when `AutoAttack` is not false.
      - The dock's Settings click sets `GoblinMenuGui.OpenSettings`; a second click closes it (DisplayOrder 8, under the dock).
  - **Teleport (since 2026-10-09): `StarterPlayerScripts.GoblinTeleportClient`** (local copy `Goblin_UI/GoblinTeleportClient.client.luau`), after the user's reference.
    - A blue panel (an external panel, see Settings) with the dock's own Teleport icon and filter tabs All Zones / Unlocked / Locked.
    - Zone cards: the zone icon, name, "Zone n • …" and a GO or LOCKED button. The price of the next zone comes from the gate button's `ZoneCost` and is cached, because of streaming.
    - The preview on the right: the zone icon big (bobbing, soft glow), name, text and a big GO.
    - **Zone icons** are painted tiles, not screenshots (the user). They are made by `Goblin_UI/make_zone_icons.py` → `extras/zoneicon_01..10.png`, one motif per zone: flag, pines, crystals, cactus, palm, peaks, temple, tombstones, toxic pool, halo + clouds. The ids are in the `ZONES` table.
    - Zones: Spawn, Fernwood Forest, Crystal Hideout, Extinct Desert, Beach Hideout, Ice Peaks, Jungle Temple, Abandoned Graveyard, Toxic Swamp, Heaven.
    - **Numbering (2026-10-09):** internal 1 = Spawn (no mobs, "Spawn • No mobs"); internal 2 Fernwood = "Zone 1" (always owned, `GoblinInventory.publish` sets `zones["2"]`). Shown number = internal − 1 everywhere: Teleport, gate screen titles, the unlock message. Internal numbers in attributes / data are unchanged.
    - Owned = zone 1, or listed in the player attribute `Zones`. GO streams in the area and puts the player on the ground at `V4WorldData.ZONES[i].c` (zone 1: the SpawnLocation).
    - It opens through `GoblinMenuGui.OpenTeleport` (dock click) and closes on a second click. The old `teleportBody` is no longer opened.
- **Toasts:** in their own `GoblinToastGui` (DisplayOrder 30). While rolls are visible, they appear just above ROLL.
- **EXTRAS (`StarterPlayerScripts.GoblinExtrasClient`, 2026-10-09, after the user's reference, not 1:1; local copy `Goblin_UI/GoblinExtrasClient.client.luau`):**
  - A small orange EXTRAS button (68 px, 2 × 2 cream tiles) left of ROLL, inside `RollGui.RollHud` (scales with ROLL), its bottom level with ROLL's.
  - While a roll plays full screen (`Spotlight`, Hide Rolls off) it slides down out of view, and it comes back after. With Hide Rolls on it stays.
  - It opens a pop-up list in its own ScreenGui `GoblinExtrasGui` (DisplayOrder 10) right above the button, with its tail (`rbxassetid://123089578597470`, open top) on the button. The list has an orange frame, an "EXTRAS" title and a red X, and cream rows with an icon, a white outlined label and an orange ">".
    - Placing note: `AbsolutePosition` does not count the 58 px top bar, while `Position` in a ScreenGui with IgnoreGuiInset does, so the inset is added.
  - Rows: Profile Stats, Bingo, Achievements, Codes, Mastery, Playtime Rewards. Exactly 4 show at a time; the rest scroll (wheel / touch, drag the orange thumb, click the track).
    - `CanvasPosition` is in screen pixels after the UIScale, so the scroll range comes from `AbsoluteCanvasSize − AbsoluteWindowSize`.
  - Every row shows "…: coming soon!" for now. Bingo would open a bingo card through `GoblinBingoGui` attribute `OpenRequest` (with a green dot while `Claimable`), but that UI was removed (see below).
  - It closes with X / the button / Esc / a full-screen roll / a menu panel opening; opening it closes the open menu panel (`CloseMenus`).
  - Icons (`Goblin_UI/extras/`, ids in `asset_ids.json`): Codes / Mastery / Playtime Rewards are the user's own; Profile Stats / Achievements / Bingo and the tail were drawn in the same style by `Goblin_UI/make_extras_icons.py`.
- **Bingo UI removed (the user, 2026-10-09):** `StarterPlayerScripts.GoblinBingoClient` (the "Bingo" icon under the dock + the 3×3 card panel) is gone; backup `ServerStorage.UI_Backup.GoblinBingoClient_removed_20261009` (Disabled). The server side still exists and counts progress: `ServerScriptService.GoblinBingoService`, `ReplicatedStorage.GoblinBingo`, `GoblinBingoRemotes`, and the hook in RollServer.

## Best roll ever podium (spawn, 2026-10-09)
- **The rarest roll ever over ALL servers** stands on the spawn podium as a giant goblin, with "1 / <full number>", "Rolled by @name" and the crown above it.
- Server: `ServerScriptService.GlobalBestRoll`.
  - DataStore `GlobalBestRoll_v1` / `record`; UpdateAsync keeps only better records.
  - MessagingService topic `GlobalBestRoll`.
  - The record goes to clients through `ReplicatedStorage.GlobalBestRoll` (StringValue JSON).
  - Fed by RollServer (`GlobalBest.Submit` with each roll's rarest result).
  - **`rev` (admin override, 2026-10-10):** a record with a higher `rev` always wins; within one rev the higher odds win.
    - `GlobalBest.AdminSet(admin, req)` sets any record (any goblin, mutations, odds, player by username or UserId), or clears it, with rev + 1. It replaces the record in every server and in the DataStore, even with a lower one.
    - Normal rolls keep the current rev. After a reset, the next roll becomes the record.
  - **Studio is isolated** (`LIVE = not IsStudio()`): no DataStore writes, no MessagingService publish or subscribe. MessagingService works from Studio, and Studio tests on 2026-10-10 reached other servers.
- Client: `StarterPlayerScripts.GlobalBestPodium`.
  - Puppet 7.5× with a cyan outline; its own walking ring off.
  - Electric cyan FX: floor circle + cracks, a lightning cage, a bolt pool, comets, streaks, pillar, particles, a surge every 5 s.
  - The sign follows the goblin's visible top. A new record plays a lightning-storm swap.
  - Odds text: one line, never shortened (`full()` keeps 15 significant digits past 1e15). The sign widens for long numbers, up to 77 studs.
  - Odds animation: a molten-gold gradient that breathes, a white/cyan shine sweep every 2.6 s (`OddsShine`), and a punch on every surge.
  - Zoom: bolt, beam and trail widths and particle sizes scale with camera distance (`zoomK` = dist / 55, clamped 1..3.2), so the show stays visible at max zoom.
  - **The record holder's avatar template:** the server (`GlobalBestRoll.buildAvatar`) builds an R15 model with `CreateHumanoidModelFromDescription` into `ReplicatedStorage.GlobalBestAvatar`.
    - Attributes: UserId, Name, Count. It is rebuilt only when the userId changes.
    - The client waits until all `Count` descendants have replicated.
    - The small dancing avatar beside the stairs was removed on 2026-10-10 (the user).
  - **Colossal record holder behind the back cliffs** (the `Giant` block, 2026-10-10), cloned from the `GlobalBestAvatar` template:
    - **The arm is the same for every avatar** (2026-10-10). Arm lengths at scale 1 range from 1.46 (classic) to 2.25 (Rthro), so each avatar gets its own scale:
      - `sc = ARM / armLength` (ARM 105 studs, clamped 40..100);
      - the body is moved so its right shoulder joint sits on `SHOULDER` (TOP + BACK·178 + right·60 + up·238);
      - the arm reaches along `ARM_DIR`: forward, 24° out to its right, 24° down, clear of the body.
      - Tested on 5 avatars: hand within ~10 studs, waists all at y 310–330 behind the wall.
    - Pose, solved in code (no animation asset):
      - every body part is anchored, CanCollide/Query/Touch off, no shadow;
      - parts are placed by forward kinematics over the rig's joint attachments. These avatars use AnimationConstraint joints; Motor6D also works;
      - 5° hips + 16° waist lean, 6° twist; the head looks down at the goblin (max 50°);
      - the right arm is near-straight (two-bone IK); the hand's own axes are used (-Y fingers, -X palm), with the palm facing the statue's head; the left arm is relaxed.
    - Animation: only the wrist moves (slow ±4–5° wobble). New holder: it rises from behind the cliffs (−320 → 0 in 2.8 s, `Giant.wantRise` from `setRecord`).
    - **Two points carry the effect** (the user):
      - `cAtt`: the middle of the palm, `HAND_GAP` 5 studs off it, so the circle never sits in the hand;
      - `gAtt`: the statue's head, `goblinHead` = the puppet's Head bone (`TransformedWorldCFrame`), lifted halfway to its visible top.
      - Every stream effect connects them.
    - Hand FX: a gold rune circle (`hand_circle`) lying flat against the palm (it tilts with the wrist), the cyan rune ring counter-rotating, purple glow.
    - The stream from the hand point to the statue's head (gold → purple → cyan):
      - beams: glow, 2× energy_flow, white core, 2 crackling bolts;
      - 6 spiral comet trails that wind down and then round the goblin onto the floor circle;
      - rune rings sliding down, streaks, sparkles, sparks, floating Futhark runes along it.
    - On the podium: rising rune particles and light streaks (the fed aura).
    - Eyes (bigger, 2026-10-10), per eye:
      - a billboard of `S*0.46` studs: glow, flare, a long anamorphic streak and a turning star;
      - two flowing light trails back past the temple;
      - rising energy wisps and sparks;
      - small crackling bolts round the eye.
    - On every pulse the eyes swell, a ring bursts out, and twin gaze bolts strike the goblin.
    - Face: small lightning flashes along the face sides; a faint purple halo; star twinkles; a cyan/purple Highlight outline.
    - Sound (3D, swells as you walk up):
      - loops: hum at the stream ("Magic Force Field Constant 2", 210 studs), arcing at the podium ("Electric Arcing 2", 120), a cosmic drone at the hand circle ("Hollow Rumble 1", 340);
      - every pulse: a whoosh, then an impact at the goblin;
      - crackles with the face lightning; thunder + "herrijzen" when a new giant rises.
      - The loops restart on `Ended` instead of `Looped`, so the Settings count them as effects, not music.
    - Pulse: 0.9 s before every podium surge the circle flares and a burst runs down the stream; a big pulse on a new record.
    - Culling: effects follow the podium cull (320 studs); the body, eyes and halo stay up to 1100 studs (`FAR`).
    - Textures: `Goblin_UI/make_giant_fx.py` → hand_circle 118206231131185, rune_sheet 124422866588666 (4×4), energy_flow 135724489714047, streak 112861688695412, eye_flare 75351308265274.
  - Culled beyond 320 studs; PerformanceMode / LessVFX lighter.
  - Test: ServerStorage attribute `StudioGlobalBest` = "Goblin|Mutations|odds" or "reset".
- Texts: GameText `Podium`. Textures: `Goblin_UI/make_podium_fx.py`, `make_crown.py`.

## Sprint (`StarterPlayerScripts.SprintClient`, 2026-10-09)
- Walk for 2 s (W, any walk key, the thumbstick: `Humanoid.MoveDirection`) and the player sprints: WalkSpeed × the player attribute `SprintMultiplier` (default 1.2; meant for later upgrades), FOV +7 (tween 0.35 s), and a rumble trail at the feet: dust puffs (`SprintDust`, fx_puff), grit (`SprintGrit`) and a short ground streak (`SprintTrail`), only on the ground.
- Stopping (> 0.25 s) ends it: speed and FOV go back. It never overrides a WalkSpeed someone else set (cutscenes freeze with 0) and leaves a Scriptable camera alone.
- Test tip: `Humanoid:MoveTo` does not set `MoveDirection`; a test calls `hum:Move(dir)` from `RunService.Stepped`.

## Map: Zone 1 Fernwood Forest
- **Map model:** `Workspace.RAG_TEXTURED_EXPORT`, imported from `Roll_A_Goblin_Zone1/`. It contains:
  - spawn hub
  - paths
  - 83 trees and plants
  - rocks
  - lanterns
  - a collision folder
- **Leaderboards:** `Leaderboard_Rolls`, `Leaderboard_Playtime` and `Leaderboard_Coins`. These are props only; there is no leaderboard script yet.
- **Signs:** `Workspace.SignTexts` holds Montserrat SurfaceGuis for "FERNWOOD FOREST / ZONE 1" and the Zone 2 sign. The old lettering meshes are hidden.
- **Zone 2 gate:**
  - The UNLOCK button is a part with `ZoneUnlock=2` and `ZoneCost=5000`.
  - **`ServerScriptService.ZoneServer`** handles the ClickDetector: it pays the coins and saves the zone.
  - **`StarterPlayerScripts.ZoneGateClient`** handles the feedback:
    - hover highlight
    - hint above the button
    - a press animation
    - a red shake when coins are short
    - a coin burst on success
    - the sign switching to UNLOCKED / OPENING SOON
  - **Collision:** the original `BarrierBox_GateTemporary` is switched off. `Zone2Barrier` sits just behind the button.
  - **Zone 2 itself is not built in this place.** The gate does not open yet.
- **Forest effects:** `StarterPlayerScripts.ForestAmbience`, client-only and streaming-proof:
  - falling leaves from the canopies
  - swaying trees and shrubs
  - wind gusts with leaves
  - pollen
  - fireflies and a light at the lanterns
  - spores at the toadstools
  - butterflies at the flowers

## Map in Blender: `GoblinRNG_MAP.blend` (10 zones, 3 restyled versions, 2026-10-08)
- **File:** `C:\Users\qmarc\Downloads\GoblinRNG_MAP.blend`.
- **In Studio (since 2026-10-08):** V4 as `Workspace.MR_Versie_4_Mix_A_zones1-5` and `…_B_zones6-10`.
  - Mapping: Studio = 3.571 × (-x, z, y)_Blender + (3679.80, 150.10, -31.98).
  - The new zone 1 stands above the old zone 1, with its floor at Y 150.
  - SpawnLocation on the new plaza; outer barriers `Workspace.V4_InvisibleBarriers`.
  - Gates:
    - `V4_Gate_<a>_<b>_Barrier` (invisible) and `_Wall` block until the player owns zone `<b>`; then they open client side (`ZoneGateClient`).
    - **Gate screens (2026-10-09, the user's reference, in Montserrat; built by `ServerStorage.ZoneScreenBuilder`, run `require(fresh clone)(zone)` for zone 2–10):**
      - **No shadows:** `Workspace.V4_ZoneGates.V4_Gate_<a>_<b>_Screen` is a see-through part 0.08 studs in front of each dark wall (112 × 41 studs, measured with rays). It has an unlit SurfaceGui (`LightInfluence` 0) in the wall's colour, a bit blacker as asked (gradient 86, 68, 70 → 42, 35, 50, calibrated against the colour correction). That covers the tree / vine shadows. A 2 % see-through wall and Neon did not stop them (tested).
      - **Since the user's 2nd round: everything is ONE SurfaceGui on the screen** (9 px / stud; a canvas wider than ~1024 px may not draw), exactly the size of the wall face (113.4 × 41, measured with rays in 0.1 stud steps). That removes the overlap edge and the seam line between stacked GUIs; never stack see-through SurfaceGuis.
      - On the screen, centred and bigger:
        - a gold padlock (`rbxassetid://96631962086033`, `make_extras_icons.py lock()`);
        - "ZONE n" (Heavy, white, dark outline, 5 studs);
        - the zone name (the sign's attribute `ZoneName`);
        - a coin + the price (`PriceText`, written from the button's `ZoneCost` by `ZoneGateClient`);
        - the green UNLOCK button (`Button`: UIScale `Pop`, UIGradient `Btn.Face`, 14 × 4.2 studs).
      - The sign part `V4_Sign_Zone<nn>` only keeps `ZoneName` (empty GUI).
      - **Button** `V4_ZoneUnlock_<nn>` is now only the see-through click target over the drawn button (attribute `FlatButton`; ZoneServer still adds the ClickDetector).
        - `ZoneGateClient` makes the drawing grow on hover, squash on a click and flash red when you can't pay. The new map's button wins over the old map's zone-2 button.
      - **Price: 1 coin for every new zone for now** (the user). `ZoneCost` = 1 on `V4_ZoneUnlock_02`–`10`; the old prices are in the attribute `ZoneCostPlanned`.
      - Old signs / buttons in `ServerStorage.ZoneScreen_Backup_20261009`.
    - Placeholder unlock buttons and signs are in `Workspace.V4_ZoneGates`.
  - `_NoCollide` parts have CanCollide / CanTouch / CanQuery off (they came in collidable; fixed 2026-10-08). **Exception (2026-10-09):** `*_Path_NoCollide`, `*_PathStones_NoCollide` and the pavers are CanQuery + PreciseConvexDecomposition + attribute `GoblinGround`, so the goblins and their rings stand on them (the renderer's ground ray). The solid floors are the `V4_<Zone>_Terrain` parts at Y 150.1.
  - Night, life and water: see "V4 world FX" below.
- **Layout:** 10 zones in a row along +Y, each about 43.6 x 69.4 units, with a gate between each pair (gates at y 34.7 … 589.9).
  - The zones are Plains, DarkForest, Cave, Desert, Beach, Ice, Temple, Graveyard, Swamp and Heaven.
  - The invisible walls are at x ±16.4.
- **The original** stays untouched at x = 0, in its own collections (Zones, Boulders, Cliffs, Gates, InvisibleWalls).
- **Three new versions** stand next to it, each a complete map with gates and invisible walls:
  - `MR_Versie_1_Kliffen` at x +140;
  - `MR_Versie_2_Themawanden` at x +280;
  - `MR_Versie_3_Avontuur` at x +420.
  - Every zone has its own collection, e.g. `V2_Zone_04_Desert`.
  - Labels and the preview camera / sun are in `MR_Preview`.
  - To use a version, move its zone collection by −140 / −280 / −420 in x.
- **Version 4 "Mix" (2026-10-08, the user's favourite direction):** `MR_Versie_4_Mix` (zones `V4_Zone_xx_*`, links `V4_Link_xx_yy`).
  - What it combines: the V2 walls; the V3 "ceiling" landmark (arch, fallen tree, rock ribs …) on cliff pillars; and the detail of the user's three references (forest canyon, crystal mine, temple ruins).
  - Reference details:
    - stepped grey stone cliffs with grass caps, vines and trees on top (green zones);
    - a full rim of trees or themed items (`rim_cover`);
    - waterfalls from a cliff into a pool;
    - stone terraces with stairs (Cave, Temple, Heaven);
    - paved paths, path-edge stones, fences;
    - flower / mushroom clusters;
    - a themed set per zone (mine scaffolds, ruin arches, market stalls, parasols …).
    - **No lantern posts** (the user removed them).
  - Layout: the zones are not in a straight line (`mr_v4_layout.py`). Each zone has its own heading (≤ 26° change per step), and all are at the same height (the user dropped the height differences).
  - Links: winding corridors between zones, starting at (1000, 0) for zone 1. Each link is:
    - clipped exactly to both gate lines, so nothing overlaps a zone;
    - matched to the zones' own floor heights at the gates;
    - built half in zone i's style and half in zone i+1's.
  - Floor and path change texture along an organic, noisy border (the user's reference). Each texture is mapped in its own zone's frame, so it runs on seamlessly through the gate.
    - The floor triangles are cut exactly on the border, so there are no saw teeth.
    - `build_link(P, i, only_floor=True)` rebuilds just the two floor meshes of a link.
  - Seal rocks close the corners where zone walls meet link walls. Rock arches stand on links 2→3 and 5→6.
  - Ray test: 0 leaks to the outside, around all gates and along all zones and links.
  - Generator steps:
    - `mr_v4.build_zone(i)` builds zone i straight at x = 700;
    - `mr_v4_layout.run_all()` moves the zones once (collection flag `mr_layout`) and builds the links;
    - `reheight()` applies a changed `HEIGHT` (flag `mr_hz`).
    - To rebuild: rebuild all zones first, then run `run_all()`.
  - Size: 1621 meshes, about 1.73 M tris (V2: 658 k).
  - Export: `Blend/Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` (66 MB) and `…_B_zones6-10.fbx` (61 MB).
- **Textures:** one 1024 atlas per zone (`MR_<Zone>_Atlas`), plus a ground texture (`MR_<Zone>_Ground`) and a path texture (`MR_<Zone>_Path`).
  - The atlas has 8 gradient strips for the top-to-bottom wall transition and 128 swatch slots for props.
    - `mr_core` fills slots 0–79.
    - In `MR_Desert_Atlas`, slots 80–101 hold the skull colours from `mr_skulls.EXTRA_SW`.
  - The V4 desert landmark skulls (mammoth with crossing tusks, T. rex with open jaws) come from `mr_skulls.py`, not from `mr_props.skull`.
  - The V4 temple landmark is the Mayan sun gate from `mr_sungate.py` (`V4_Temple_SunGate_01` + `_Glow_NoCollide` fire + `_Vines_NoCollide`).
    - It replaces the V3 stone face gate.
    - `mr_v3.landmark` builds it when `ctx.ver == 4`; V3 keeps the face gate.
  - Extra set pieces placed after the zone build come from `mr_fun.py` (`run()`; a V4 rebuild needs it run again):
    - Ice: igloo camp, penguins, ice-fishing hole;
    - Swamp: alligator, toxic barrels, sunken boat, toxic bubbles, danger sign.
  - Link paths get stone slabs on the side of a paved zone (`link_pavers` in `build_link`).
  - The style follows the game's own `RAG_Forest_ColorAtlas`. Plains uses the game's `RAG_Path_PackedEarth_512` path.
  - Glow parts are `*_Glow_NoCollide`, water is `*_Water_NoCollide`.
- **Generator (re-runnable):** `Blend/Map_Restyle/`:
  - `mr_core` (mesh builder, texture painter), `mr_themes` (colours of the 10 zones), `mr_props` (props), `mr_walls` (cliffs);
  - `mr_decor` (placement, paths, ponds, waterfalls, copying the originals), `mr_terrain`, `mr_elements`, `mr_landmarks`;
  - `mr_v1` / `mr_v2` / `mr_v3` (one `build_zone(i)` per version);
  - `mr_qa` (`check` / `fix`: floating, buried and in-rock pieces), `mr_render`, `mr_view`.
- **Roblox rules followed:**
  - merged meshes are split at 18k tris;
  - decoration is `_NoCollide`;
  - trees are `_Trunk` + `_Canopy_NoCollide`;
  - one atlas per zone.
- **Animation pack (`mr_animpack.py`, 2026-10-08):** the V4 grass / flowers / reeds / lilies are replaced in Studio by clones of 23 small template meshes so every tuft can sway.
  - `run()` builds collection `V4_AnimPack` (excluded from the view layer afterwards; the map collections are not changed), writes `Map_Export/V4_foliage.json` (4011 placements in Studio coords) and exports `Map_Export/V4_AnimPack.fbx`.
  - The pack also holds the cluster objects without their grass / flowers (`AP_R_*`), the 69 bushes and 6 Heaven floor clouds as single pieces (`AP_S_*`), the windmill body + a separate rotor (`AP_W_*`) and 3 marker cubes to solve the import transform.
  - A re-export of the full map would bring the merged grass back; run the pack again and redo the Studio swap (see the log entry).
- **Pack 4 (`Map_Restyle/mr_pack4.py`, 2026-10-09; replaces pack 3, which was never imported):**
  - `build()` makes collection `V4_AnimPack4`:
    - `AP4_T_*` hill trees, `AP4_B_*` walkable bushes from EdgeGreen / LinkGreen, `AP4_V_*` cliff-vine pieces;
    - `AP4_R_<k>__<group>` rebuilt merged groups, `AP4_C__<obj>` cliffs without vines, `AP4_X__<obj>` single objects;
    - `<src>` after `__` = the Studio part it belongs to.
  - `export_ab()` writes `Map_Export/V4_AnimPack4_A.fbx` / `_B.fbx`.
  - `fix_ice_path_gap()` repaired the Ice path after the lake was moved.
  - Studio side: `ServerStorage.V4FX_Pack4Swap`.
  - `SKIP_SINGLES` keeps out objects that Studio already has as single pieces (the Plains arch / DarkForest giant hangs).
- **Link 9 (Swamp → Heaven) is Heaven from right after the gate** (2026-10-09): `mr_v4_layout.SPLIT = {9: 0.06}` puts the A/B style split of that link at 6 % instead of halfway (only link 9). Rebuilt with `build_link(plan(), 9)` + `mr_animpack.fix_link_paths([9])` / `straight_link_paths([9])`.
- **Angels (`Map_Restyle/mr_angels.py`, 2026-10-09):** `V4_Heaven_Angel_01_NoCollide` (trumpet angel, ~5 m) and `_02_NoCollide` (harp cherub), standing on their origin facing −Y. They live in collection `V4_Heaven_Angels` (under `V4_Zone_10_Heaven`) and stand on two floating clouds: 01 on `V4_Heaven_HoverCloud_5074`, 02 on `V4_Link09B_HoverCloud_5005` (`mr_pack5.ANGELS`).
- **Pack 5 (`Map_Restyle/mr_pack5.py`, 2026-10-09):**
  - `build()` makes collection `V4_AnimPack5`: all of link 9 (`AP5_X__` singles, `AP5_R_` group rests, `AP5_T_` / `AP5_B_` trees / bushes, `AP5_H_` clouds), the Heaven groups WallElements / EdgeGreen / RimCover / ElementTop / WallTop split into clouds `AP5_H_`, pillars `AP5_P_` (stone, slender, > 4.5 m), trees `AP5_T_` and rests `AP5_R_`, and the angels `AP5_N__`. Plus `Data/pack5_foliage.json` (link 9 grass / flowers) and `Data/pack5_lights.json`.
  - `export()` → `Map_Export/V4_AnimPack5.fbx` (markers `AP_Marker_*_P5`). Studio side: `ServerStorage.V4FX_Pack5Swap`.
  - `hover_data()` → `Data/pack5_hover.json`: per floating piece how far it may bob (U up / D down, in studs), whether it really floats (hover FX), plus `_links` (which pieces touch: "h" = move as one, "s" = two clouds side by side). Measured on the exact meshes against everything static around them; trees that stand on a cloud or float free are included. Studio side: `ServerStorage.V4FX_HoverGroups`.
  - `studio_name()` gives the Studio name of a pack-5 piece (`V4_<Zone>_HoverCloud_<5000+n>_NoCollide`, `…_HoverPillar_…`, trees `…_HillTree_<5000+n>_NoCollide`).
- **Animation pack 3 (`mr_animpack.split_trees` / `export_pack3`, 2026-10-09, superseded by pack 4):** the trees inside the merged hill / cliff-top groups (`_RimCover`, `_ElementTop`, `_WallTop`, `_ArchTop`, `_CliffTop`) as single pieces `AP3_T_<Zone>_<nnn>` (862), plus those groups without their trees `AP3_R_<group>` (34).
  - Collection `V4_AnimPack3`; export `Map_Export/V4_AnimPack3.fbx` (34.7 MB, markers `AP_Marker_*_P3`).
  - In Studio they become `V4_<Zone>_HillTree_<nnn>_NoCollide` (kind HillTree in V4WorldFX).
  - The cliff vines (vine slot 71 inside 202 cliff / wall meshes, ~2070 strands) and the bushes inside merged groups still have to be split the same way (kinds CliffVine / Bush are ready in the script).
- **Sync Blender → Studio (`Map_Restyle/mr_sync.py`, 2026-10-09):**
  - `snapshot()` writes a fingerprint of every object to `Map_Export/blender_snapshot.json` (transform, visibility, md5 of vertices / faces / UVs, materials, modifiers).
  - `diff()` lists what was changed / added / removed since the snapshot.
  - `studio_bbox(o)` gives an object's Studio centre and size, to compare with the Studio part.
  - `export_update(names)` exports only those objects, plus 3 marker cubes `AP_Marker_*_UP`, to `Map_Export/V4_Update.fbx`.
  - The snapshot was taken 2026-10-09 00:40 (12,010 objects), before the user's own Blender edits. Backup: `Downloads/GoblinRNG_MAP_backup_20261009_voor_jouw_edits.blend`.
  - `Map_Export/Data/snapshot_base_20261008_2044.json` is the fingerprint of `GoblinRNG_MAP_backup_before_animpack_20261008.blend`, which is the state the Studio map was imported from. `diff(that file)` shows everything changed since the import. Make it with a background Blender: `blender -b <file> --python-expr "...mr_sync.snapshot(path)"`.
  - **Current base: `Map_Export/Data/snapshot_base_20261009_pack5.json`** (13,908 objects, after pack 5 + the angels on their clouds). Studio = Blender at that point; diff against it for the user's next edits.
  - Compare bboxes from real vertices, not from `bound_box` corners: the corners of a rotated object give a too-big box. That mistake made the paths look "older" in Studio; they were not.

## V4 world FX: sunrise, life, water (client, 2026-10-08 / 09)
- **Look (since 2026-10-09): sunrise.** `Lighting.Sky` = our own pastel sunrise skybox (`fx_textures.skybox_sunrise`: warm glow at the horizon, pink / gold lit clouds, no painted sun); Roblox draws the sun (`CelestialBodiesShown`, SunAngularSize 14).
  - ClockTime 9.682 + GeographicLatitude −36.85 put the sun 24° up, toward −Z turned 18° to +X, right above the painted glow (a lower sun left the canyon floors in shadow).
  - Per-zone light / atmosphere / bloom / colour correction / sun rays: `ReplicatedStorage.V4WorldData.LOOK` (sunrise), blended along `ZONES` (zone centres) while you walk through a link.
  - `V4WorldData.NIGHT = false`. The night version is kept: `LOOK_NIGHT`, `SKY_NIGHT` (night skybox, moon in −Z 30°, ClockTime 12 / latitude −36.5). To go back: set `NIGHT = true`, swap `LOOK` / `LOOK_NIGHT`, put the `SKY_NIGHT` textures on `Lighting.Sky` (CelestialBodiesShown off) and the clock / latitude.
  - In the sunrise: no shooting stars / aurora, fewer fireflies, butterflies are not neon, the character light is weaker, lantern lights are dimmed (0.45), morning birds instead of night crickets (`SOUNDS.birds` / `forest`).
  - The Studio Lighting is set to the Plains look; the old day lighting is in `ServerStorage.V4FX_DayLightingBackup`.
- **Script:** `StarterPlayerScripts.V4WorldFX` (LocalScript; local copy `Blend/Map_Restyle/studio/V4WorldFX.client.luau`, Studio stays the source of truth). It does:
  - air particles per zone around the player (fireflies, pollen, spores, crystal dust, sand, snow, wisps, bats, feathers, leaves, ground mist), wind gusts with leaves / sand / snow / sparkles, glowing butterflies that fly to the flowers;
  - sway: `Workspace.V4_Foliage` (Grass / Flower / Reed / Lily), bushes `V4_<Zone>_Bush##_NoCollide`, trees (trunk + crown sway together around the foot of the trunk, the crown rustles on top), palms, hanging vines / the shipwreck sail; leaves (Ice: snow, Heaven: petals) fall from crowns within 65 studs;
  - the windmill rotor spins (part attributes `SpinAxis` / `SpinSpeed`); floating islands, floor clouds and toxic bubbles float; lilies bob;
  - the floor of every zone and link (`V4_*_Terrain`): wind bands rolling over grass / sand (`fx_windsheen`), drifting cloud shadows (`fx_cloudshadow`), twinkling frost / gold / crystal glitter on Ice / Heaven / Cave (`fx_glints`) — settings per zone in `GROUND` in the script;
  - water: scrolling toon line textures on ponds / lakes / the stream, ripple rings and glints (bubbles in the Swamp); waterfalls get two falling streak layers and an impact (white water, splashes, spray, foam rings and patches on the pool, a soft light, 3D sound); frost glints on the frozen fall; fountain jets in Heaven;
  - fire: campfires get flames, embers, smoke, a flickering light and crackle; the orange glow points (sun gate bowls) get flames;
  - lights: `ReplicatedStorage.V4WorldLights` (538 glow points from Blender, generated) — the nearest 36 are lit (pool, fades in / out);
  - sky: shooting stars, an aurora over Ice (partly visible from Beach / Temple), light shafts in Heaven, the turning lighthouse beam + spotlight;
  - sound: night ambience per zone (`V4WorldData.SOUNDS` / `AMBIENCE`, licensed Pro Sound Effects / DistroKid sounds), 3D waterfall + fire;
  - the lighting is left alone while the camera is Scriptable (cutscenes) and below Y 120 (old map); particle rates / radii go down on low graphics settings.
  - (v10, 2026-10-09) **big wind gusts** (`BIG`):
    - every 4–7 s a band of 5–7 wide wind ribbons (`fx_windribbon` trails) with ~40 leaves/s (petals in Heaven, snow in Ice, sand in Desert) rolls 240 studs along the wind, through a tree crown 40–200 studs in front of you (high ground and hill trees preferred);
    - everything it passes bends along (`KIND[...].gust`, `BIG.at`) and you hear it rush past (3D wind sound);
    - at most 2 at a time.
  - (v10) **camera floor:** `BindToRenderStep("V4FX_CameraFloor")` keeps the camera at Y ≥ 151.4 on the V4 map, so nobody can look under the world (not in cutscenes).
  - (v10) **frozen lake** (`ICE`, `V4_Ice_FrozenLake_NoCollide`): two layers of fish (`fx_fish`) swim under the ice, plus frost glitter, a sliding shine and powder snow / drifts blowing across. A footstep on the ice leaves a crack star (`fx_icecrack`) and a frost puff.
    - Note: on a Top face a growing `OffsetStudsU` moves the image towards +X, while the right side of the image lies at −X.
  - (v10) **waterfall springs:** `V4_Props.WaterfallSources`. On the cliff top behind each fall (Plains, DarkForest, 2× Temple, Heaven) there are spring rocks, a stream channel (flat water, flows 3× faster), and two overhanging rocks the fall now comes out from under.
    - The `_Spring_NoCollide` part (attributes `Lip`, `LipWidth`) gets welling foam, flecks riding to the lip, glints, a white crest + mist where the water leaves the rock, and a soft splash sound.
  - (v10) kinds **HillTree** (`V4_<Zone>_HillTree_<nnnn>_NoCollide`: amp 0.026, r 330; 862 in the map) and **CliffVine** (`V4_<Zone>_CliffVine_<nnnn>_NoCollide`, 505 in the map) since pack 4 (2026-10-09). Pack 4 also added 174 bushes `V4_<Zone>_Bush501`–`674` (kind Bush: sway + push). Every zone and link now has walkable bushes.
    - A CliffVine swings along its wall and lifts off it. The wall normal comes from 8 level rays; when no solid wall is hit (decor walls are not queryable), it uses the direction to the zone centre (v11).
    - Canopy / Palm radius is now 220.
  - (v11) the temple sun gate vines are no longer kind Hang (one block: the far vines left the stone).
  - (v12, 2026-10-09) **Heaven floats:** kinds `HoverCloud` (`V4_<Zone>_HoverCloud_<n>_NoCollide`, 89), `HoverPillar` (`…_HoverPillar_<n>_NoCollide` 38 + the old `V4_Heaven_Column_01`–`08`) and `Angel` (`V4_Heaven_Angel_<n>_NoCollide`), plus the HillTrees that carry Hover attributes (7 Heaven trees on a cloud / floating free; they bob and still sway).
    - Every piece bobs y = `HoverOffset` + `HoverAmp` · sin(2π t / `HoverPeriod` + `HoverPhase`); `HoverSink` = no wobble (the group stands in the ground: it sinks a bit and comes back, never out of the ground); free groups wobble a tiny bit around `HoverPivot`. Pieces that touch (a tree / angel on its cloud, a pillar in a cloud) share one beat and pivot and move as one; clouds side by side get one period and a 0.55 rad phase step (a wave). Not all at once, nothing runs into each other.
    - The attributes come from `ServerStorage.V4FX_HoverGroups` (run `require(fresh clone)(hoverData)` in Edit mode with `Data/pack5_hover.json` decoded from its data PNG). State 2026-10-09: 144 pieces in 120 groups; 3 stay still (clouds that carry a glow ornament with no room to move), 118 bob ≥ 1.2 studs.
    - Hover FX only on pieces that really float (attribute `Floating`, 20): gold / white sparkles dripping, soft wisps, and at the lowest point of every bob a push off the air (2 rings + a sparkle puff). Angels always have a soft warm light + rising motes (`V4FX_AngelGlow`).
    - All of them have CanCollide / CanQuery / CanTouch off (the user: "niet colliden"); the 8 columns have attribute `HoverNoCollide`.
    - Cost (Heaven entrance, 80 hover emitters): render CPU +0.08 ms, GPU +0.14 ms.
  - (v12) **The Heaven look starts right after the Swamp → Heaven gate:** `BLEND["Swamp|Heaven"] = {0.29, 0.1}` (the zone weight runs from 0.29 to 0.39 of that zone segment instead of 0.36 → 0.64; the gate sits at 0.293).
  - **Flat meshes are double sided** (2026-10-09): path / pond / lake meshes had down-facing end-cap triangles that Roblox culled, which showed as V-shaped gaps in the paths. All `_Path_NoCollide`, pond water / shores, lakes and the stream have `DoubleSided = true` (attribute `DoubleSidedFix`).
- **Props:** `Workspace.V4_Props.Lanterns` — 63 night lights next to the paths (spots in `ServerStorage.V4FX_LanternSpots`): lanterns (Plains, plaza, Link01A), torches (Desert), tiki torches (Beach), frost lanterns (Ice), gold lamps (Heaven) + the links next to those zones. `V4_Props.WindmillLights`: the lit windmill windows. `V4_Props.WaterfallSources`: the springs and overhang rocks on top of the waterfalls (2026-10-09).
- **Ground clutter:** Pebbles / PathStones / FloorClusters / LinkClusters are 0.14 studs lower and cast no shadow (attributes `ClutterSunk`, `ClutterOldShadow`).
- **ForestAmbience** (old map) now stops its pollen / gusts / butterflies on the new map.
- **Textures:** `Map_Restyle/fx_textures.py` → `Map_Export/FX_Textures/*.png`; asset ids in `FX_Textures/asset_ids.json` and `V4WorldData.TEX`.
- **Data into Studio without pasting:** `Map_Restyle/studio/pack_data_png.py` packs a file into a PNG; upload it (Studio MCP `upload_image` from a local `python -m http.server`) and decode it with `require(ServerStorage.V4FX_DataPNG)(assetId, magic)` (EditableImage). Used for the foliage placements, the light points and the script itself.
- **Performance (Studio, 2026-10-08):** frame time the same with and without the FX (24.24 ms both; Studio is capped there). The sway costs about 0.5–1 ms per 30 Hz update at the busiest spot.

## Enemies and combat (main mechanic, replaces mining)
- **Zone 1 enemies** (Blender: `Enemy Models copy 2.blend`, builder `Fernwood_Mobs/build_enemy_models_2.py`, export `Fernwood_Mobs/export_enemies_fbx.py`):

  | EnemyId | Name | Tier | MaxHP | Spawn chance (planned) |
  |---|---|---|---|---|
  | MossBeetle | Moss Beetle | Common | 100 | common |
  | CrystalbackBoar | Crystalback Boar | Rare | 600 | 3 % |
  | ElderRootGolem | Elder Root Golem | Boss | 10,000 | 0.1 % |

  - Templates: `ReplicatedStorage.EnemyModels.Enemy_*`. Test copies that are live in the map: `Workspace.EnemyPreview` (spawning is not built yet).
    - **Since 2026-10-09 they stand in the new V4 map, zone 2 Fernwood Forest** (between gate 1→2 at z 230 and gate 2→3 at z 576):
      - Moss Beetle (72.7, 365.6), right after the entrance;
      - Crystalback Boar (44.7, 413.6);
      - Elder Root Golem (62.7, 499.6), in the clearing by the stream, reachable over the bridge.
    - Their old spots on the old map (under Spawn, y −59) are kept in the attribute `OldMapPivot`.
    - Moving one: keep the pivot's own rotation (these pivots are tilted, LookVector = up) and only turn around Y. `CFrame.lookAt` tips the model over. The facing that counts is the Hitbox's LookVector (CombatServer reads it).
  - Each model: skinned MeshPart (PrimaryPart, bones for almost everything) + invisible `Hitbox` (physics root) + `Hitbox.HPBar`. `ModelStreamingMode = Atomic`.
  - Sizes now (mesh X × Y × Z studs): beetle 5.03 × 3.20 × 6.10, boar 3.94 × 6.62 × 8.91, golem 11.42 × 14.34 × 4.96. HP bar `StudsOffsetWorldSpace.Y = mesh.Size.Y / 2 + barWidth × 0.1 + 0.4` (bar widths 8 / 9.5 / 13.5). Resize with `Model:ScaleTo` + put the feet back on the ground + recompute that offset.
  - Attributes: `EnemyId`, `EnemyName`, `Tier`, `MaxHP`, `HP`, and from the combat server `State` (Idle/Walk/Fight/Attack/Dead), `AttackSeq`, `HitSeq`, `SpawnSeq`, `TargetUser` + `TargetSlot` (the goblin its current attack goes for).
- **HP bars:** `ServerScriptService.EnemyHPBars` (fill + "33 / 100 HP" text) and `StarterPlayerScripts.EnemyNametagFX` (tier animations).
  - **Look since 2026-10-09 (the user's reference):** the name in white Montserrat Heavy with a dark outline (Rare / Boss keep their colour flow), under it a thin white line that fades out to the left and right, a pill bar (white rim, dark edge `BarEdge` that follows the HP colour, track, fill, HP text) and a white pointer (`rbxassetid://126786995804752`). No plate, icon, stars or aura any more.
  - Built by `ServerStorage.EnemyTagBuilder` (run `require(fresh clone)(model)` in Edit; it keeps the names the scripts use, the bar width and where the bottom sits). Done on the 3 templates and the 3 preview enemies; old bars in `ServerStorage.EnemyTag_Backup_20261009`.
  - **Tier stars (2026-10-09, the user's second reference):** gold stars above the name show how hard the mob is: Common 1, Rare 2, Boss 3 (`STARS` in the builder).
    - Each star is drawn by `Goblin_UI/make_extras_icons.py` `tier_star()` (`rbxassetid://90618753203553`) and sits in `Frame.Header.Stars.Star1..n`. `EnemyNametagFX` pulses them (Rare) or twinkles them one by one (Boss).
    - Size: a star is 0.1 tag widths. The tag grew upwards by `TOP` = 0.112 (height 0.452 × width, `TagStyle` "ref20261009c"), and its bottom stays where it was.
    - The white line under the name stays as it was (the user prefers it); the Elder Root Golem keeps its gold → emerald name flow.
- **How a fight goes:**
  - Enemies **ignore players**; without a fight they stroll around their home spot.
  - When a player with equipped goblins comes within `ENGAGE_RANGE` (26) of an enemy, the goblins run to it.
    - They stand around it just outside its hitbox, on both flanks first; the player keeps the front.
    - The ring of spots is turned once, when the fight starts, so walking round does not make anyone circle.
    - On the way they go round the enemy, never through or over it.
  - **Click to target:** clicking (or tapping) an enemy within `CLICK_RANGE` (55) sends all goblins to it (`CombatRemotes.Target`).
    - Hovering an enemy outlines it and shows Roblox's own button hand cursor (`UserInputService.MouseIcon`).
    - Nothing blocks the right-mouse camera drag.
    - A clicked fight is kept up to `CLICK_RANGE + 10` (65) studs from the player, an automatic one up to `ENGAGE_RANGE × 1.35`; then the goblins run back.
    - Goblins still running over join the strikes once within 6 studs of their spot (`GoblinCombatClient` `arrived`). The enemy's strike only hits a goblin that has arrived (within 8 studs, `EnemyAnimator` `targetPet`); otherwise it lands in front of the enemy.
  - All goblins strike together every `1 / ATTACK_SPEED` s.
  - The enemy stands its ground and fights back **against the goblins**: it turns to one goblin and attacks it on its cooldown.
    - The goblin hit flinches, flashes white and is knocked back in a short hop, with an impact, dust and dizzy stars. It then walks back.
    - Enemies do no damage yet.
- **Enemy looks (`EnemyAnimator`):**
  - **Red circle under every enemy**, like the goblin aura: a dashed red ring that keeps turning plus a small red glow in the middle.
    - Centred between the feet (the boar's centre is moved forward with `forward` 0.3); diameter per model scale is beetle 2.74, boar 2.69, golem 3.26 (`THREAT`, 2026-10-08: ×0.85).
    - Its own thinner dashed ring `THREAT_RING` = `rbxassetid://93690048613676` (`HUD_Mockup/make_threat_ring.py`, ~75 % of the old line). `DASH_RING` stays the icons' ring.
  - Measured joint pivots (PIVOTS, `Fernwood_Mobs/rig_pivots.py`) keep legs, tail and antennae attached.
  - The boar's crystals and the golem's core gem glow. Heads look around lightly.
  - HP bar colour: green, orange below 50 %, red below 15 %.
- **Combat scripts** (local sources in `Combat/`, hash-identical to Studio):
  - `ReplicatedStorage.CombatConfig`: all tuning.
    - `GOBLIN_DAMAGE = 1`, `ATTACK_SPEED = 1` (strikes per second), `ENGAGE_RANGE = 26`, `RING_GAP` / `RING_FRONT` / `PET_GAP` (goblin ring);
    - `ENEMY_DAMAGE = 0`, `ENEMY_FIRST_ATTACK`, `WANDER_*`, and per enemy speed / cooldown / respawn / attackTime;
    - `CombatConfig.RingSpots(...)` = where each goblin stands. It is shared by the server (to aim enemy attacks) and the renderer (to place the goblins); spot i = the player's i-th equipped goblin.
  - `ServerScriptService.CombatServer`:
    - drives every model with `EnemyId` in the Workspace (AlignPosition/AlignOrientation on the Hitbox, server-owned);
    - enemy AI:
      - fighting: face the target goblin's spot and attack on cooldown; it picks mostly goblins in front of it, never the same one twice in a row;
      - not fighting: stroll around home with pauses;
      - always: separate from other enemies, die at 0 HP, respawn at home;
      - **walking on the floor like a character (2026-10-09):**
        - `groundY` casts down from `GROUND_STEP` (2.5) above the feet and only hits CanCollide floor (`RespectCanCollide`). So enemies never float on mushrooms, path stones, branches or invisible decor, and never climb onto anything above a step.
        - Each enemy keeps the foot height it had when it was placed (`e.footLift`, at most 0.6; 0.26 for the current models). Before, the server sank them by that much.
    - goblins: each player with equipped goblins auto-engages the nearest enemy; ALL goblins strike on the same beat; damage = `GOBLIN_DAMAGE × goblins`, applied 0.3 s after the strike;
    - remote `ReplicatedStorage.CombatRemotes.Combat`: `engage` / `disengage` / `attack` to all clients.
  - `ReplicatedStorage.CombatFX`:
    - an effect toolkit: flash, burst, ring, projectile, beam, bolt, slash, spike, cracks, shake and light;
    - `damageNumber`: Montserrat Heavy, gold gradient, brown outline; it pops, rises and fades;
    - lights are dimmed (`LIGHT_SCALE` 0.4, at most 4 at once) and flashes are see-through, so 6 goblins hitting together no longer wash the enemy out;
    - `CFX.glow` / `CFX.withGlow` set a glow level per attacking goblin's rarity (see `GoblinAttacks` `GLOW_BY_RARITY`). It darkens neon a little and scales lights and light emission; the effects themselves stay the same;
    - effect errors are reported once with `[CombatFX]`.
  - `ReplicatedStorage.GoblinAttacks`:
    - one attack per goblin (all 42), each with its **own** motion (42 different motions, e.g. jab, combo, hook, lob, coin flick, lunge, X-slash, pirouette, bowling, finger snap, javelin, pocket watch) plus its own effect;
    - `A.Hurt(pet, fromPos, strength)` is the flinch + knockback when an enemy hits a goblin;
    - to add a goblin, add an entry to `A.ATTACKS` with a new motion in `M`.
  - `StarterPlayerScripts.GoblinCombatClient`: turns the events into the goblin ring + attacks + damage numbers.
  - `StarterPlayerScripts.EnemyAnimator`:
    - procedural bone animation per enemy: idle life, walk cycle, attack, hit flinch + soft white flash, death + fade + coin/leaf burst, spawn rise;
    - attack effects land on the targeted goblin (`TargetUser`/`TargetSlot` → `Renderer.GetPets`), and the goblins hit flinch;
    - the boss alternates Root Slam (hits goblins within 6.5 studs) and Core Nova (hits goblins within 11 studs).
  - Patches:
    - `GoblinPetRenderer`: the combat ring via `CombatConfig.RingSpots`; goblins keep facing the enemy, also while knocked back; `Renderer.SetCombat`, `Renderer.GetPets`.
    - `GoblinPetFX`: combat pose layers `fx.attack` and `fx.hurt`; no gestures or waves while fighting.
- **Goblin stats, goblin health, enemy damage, coin drops (2026-10-09):**
  - **Stats:** `GoblinPets.StatsOf(goblinId, mutations)` = `{ damage, health }`. These come from `CombatConfig.RARITY_STATS` and are multiplied by `MUTATION_STATS` (Huge ×2, Shiny ×1.5, Rainbow ×2.5). The inventory hover card shows them.

    | Rarity | Damage | Health |
    |---|---|---|
    | Common | 1 | 20 |
    | Uncommon | 2 | 35 |
    | Rare | 4 | 60 |
    | Epic | 8 | 100 |
    | Legendary | 16 | 170 |
    | Mythic | 32 | 280 |
    | Divine | 64 | 450 |
    | Celestial | 128 | 700 |

    - Every goblin strikes with its own damage. The "attack" event sends the damage per slot (0 = knocked out).
  - **Enemy damage:** `CombatConfig.ENEMIES[id].hits` per attack variant: `at` (when the animation's blow lands), `damage`, and `radius` (others nearby, 0 = only the target; `around = "self"` = round the enemy).

    | Enemy / attack | Damage | Area |
    |---|---|---|
    | Beetle | 6 | one goblin |
    | Boar | 22 | 2.6 studs |
    | Golem Root Slam | 90 | 6.5 studs |
    | Golem Core Nova | 55 | 11 studs round the golem |

    - The server lands it with `enemyHit` on the goblins' `RingSpots` spots.
  - **Goblin health (CombatServer):**
    - Kept per goblin instance for the session.
    - At 0 HP a goblin is knocked out for `KO_TIME` (8 s): it doesn't fight, isn't targeted and walks back to the player. It then stands up with full HP (`REVIVE_HEALTH` = 1, the user).
    - Out of a fight goblins heal 15 %/s after 3 s. When all goblins are knocked out, the fight stops.
    - Player attribute `GoblinHealth` = JSON `{ ids, hp, max, ko = seconds left }`; event `goblinHit` (player, slot, damage, ko).
  - **Client `StarterPlayerScripts.GoblinHealthClient`** (local `Combat/GoblinHealthClient.client.luau`):
    - Knocked-out goblins (all players): `pet.down` (the renderer's `combatRing` skips them), half see-through, dizzy stars, a grey puff; standing up gives a green sparkle ring.
    - Own goblins only: a red damage number per hit and **the HP bar under the name plate, only during a fight (+1.2 s)** (the user: never 6 bars on screen).
  - **The HP bar (`GoblinTagFX.SetHealth`)**, after the user's reference:
    - a dark rounded bar under the plate, with a fill that is green, yellow at ≤ 50 % and red at ≤ 20 %;
    - "140/200" in white;
    - a hit leaves a pale piece that shrinks a moment later;
    - knocked out: "KO" with a grey bar filling up to the revive.
    - While shown the tag hangs `HP_LIFT` higher (`tag.hpLift`; `Rescale` keeps it).
  - **Coins:**
    - **Since 2026-10-09 (the user): no coins on the ground.** A kill pays `coins` at once, and `CoinDropClient` shows a "+n" popup over the enemy (coin face + pickup sound), like the Forest Token popup. The ground-drop system below is switched off with `COINS_ON_GROUND = false` in CombatServer; the code is kept.
    - (old, off) A kill drops `coins` as `coinPieces` coins (beetle 25 / 5, boar 250 / 9, golem 5,000 / 16) for every player whose goblins hurt the enemy (`e.contrib`). They land on spots round the enemy, chosen by the server.
    - **Client `StarterPlayerScripts.CoinDropClient`** (local `Combat/CoinDropClient.client.luau`):
      - 3D coins standing, turning and bobbing, with a soft glint (no lights). **A dark outline** comes from one Highlight on the model `CoinDrops_Local` that holds every coin (Roblox draws at most 31 Highlights).
      - **5 of each to choose from** (player attributes; defaults in `CombatConfig`):
        - **Coin model** (`CoinModel` / `COIN_MODEL`). Faces drawn by `Goblin_UI/make_coin_icons.py` → `extras/coinface_*.png`.

          | # | Model | Build |
          |---|---|---|
          | 1 | Classic | the game's coin image |
          | 2 | Star Coin | stepped rim |
          | 3 | Goblin Token | bronze, goblin head |
          | 4 | Gem Coin | a 3D emerald in the middle |
          | 5 | Crown Coin | big, double rim |

        - **Pickup sound** (`CoinSound` / `COIN_SOUND`):

          | # | Sound | Asset |
          |---|---|---|
          | 1 | Classic | 4612374807 |
          | 2 | Bright | 113730061669739 |
          | 3 | Retro | 93529351909119 |
          | 4 | Pop | 75734523213145 |
          | 5 | Sparkle | Pro Sound Effects, 9126073001 |

          Each plays with a rising pitch in a row.
        - **Drop animation** (`CoinDropStyle` / `COIN_DROP_STYLE`), each with its own effect:
          1. Fountain: arcs up out of the enemy, two bounces.
          2. Rain: a twinkle above, then they fall one by one.
          3. Skip: a ring at the feet, 3 hops outwards.
          4. Burst: a golden shockwave, all at once fast and low.
          5. Vortex: a golden spiral up round the enemy, then out to their spots.
        - Drop sound "Coin Throws" (Pro Sound Effects).
      - Within `MAGNET_RANGE` (**32**, ×2 on 2026-10-09) they hop up and fly to the player. There they become a coin on the screen that flies into the coin counter.
      - Then `CombatRemotes.CollectCoin(dropId, index)`. The server checks that the drop is the player's, not taken yet, and that the player is near, then pays in batches of 0.2 s (`Inventory.AddCoins`).
      - Uncollected coins shrink away after `COIN_LIFETIME` (60 s).
    - The old coin particles in `EnemyAnimator.deathFX` are gone.
  - **Ranges ×2 (the user, 2026-10-09):** `ENGAGE_RANGE` 52 (goblins run to enemies from twice as far), `CLICK_RANGE` 110.
  - **Test (Studio only):**
    - The admin window's "Coins" tab (`CoinDropClient`, was the "Coin drop (test)" panel): three rows of 5 buttons (Coin / Sound / Drop).
    - A coin or drop button sets the choice and drops the nearest living enemy's coins for you (`CombatRemotes.StudioCombatTest` "drop"). A sound button plays the sound.
    - "Hurt goblins (−40% HP)" ("hurt").
- **Not built yet:** spawn chances and spawn locations, Zone 3+ enemies, upgrades for damage / health.

## Admin
- **One admin window (2026-10-10):** `ReplicatedStorage.AdminPanelUI`, ScreenGui `AdminPanelGui`.
  - It sits at the right edge and fits the screen height; the header "–" folds it into an ADMIN pill.
  - Visible when the player attribute `AdminPanel` is true (`/admin` in chat). In Studio it is always on unless `AdminPanel` is false.
  - Tabs (texts: GameText `Admin`):
    - **Roll**: speed, columns, Clover Coin, spawning (RollClient).
    - **Reveal**: Huge / Rainbow reveal styles + test rolls (RollClient).
    - **Cutscene**: New Best Roll cutscene + test (RollClient).
    - **Best Roll** (`StarterPlayerScripts.AdminBestRoll`): shows the current record and sets a new one.
      - Goblin: search + list, rarest first. Mutations as toggles.
      - Odds: auto from the goblin, or typed (`203Sx`, `2.03e23`, `203,000,000`).
      - Player: username or UserId; empty = you.
      - "Zet als best roll" / "Reset best roll" each need a second click.
      - Remote `GoblinInventoryRemotes.AdminSetBestRoll` (admins live, everyone in Studio) → `GlobalBest.AdminSet`.
    - **Coins** (Studio only): coin model / sound / drop, hurt goblins (CoinDropClient).
    - **Data** (`StarterPlayerScripts.AdminDataTools`, 2026-10-10): "Reset progress" (second click confirms) → AdminAction "resetProgress" → `GoblinInventory.ResetProgress`: your own profile back to new (goblins, coins, luck, zones, index, rolls, best odds, roll upgrades, powers, potions; settings stay). "Alle potions +5" → AdminAction "potions".
    - **Pets** (Studio only, `StarterPlayerScripts.AdminPetTest`, local `Goblin_UI/AdminPetTest.client.luau`): v4 animation tests.
      - Zweefstijl: Standaard / 1 Drift / 2 Glide / 3 Swim → workspace `V4HoverStyle`. "Zwevers 1–5" equip 6 floating goblins each (hover > 0 in GoblinV4MotionData).
      - Aanval / klik test: "Aanvallen herhalen" / "Klikken herhalen" toggle `V4AttackTest` / `V4ClickTest` = "*"; "Bevriezen op" Uit / 0.2 s / Klap (0.4) / 0.6 s → `V4ChoreoT`.
      - Goblins equippen: type names (max 6) or a group 1–14 of all v4 goblins (alphabetical; hover shows the names). Server: `AdminAction` "v4equip:Id,Id" (Studio only, in GoblinInventoryServer): gives them and replaces the equipped goblins.
  - New tools add a tab with `Admin.Section(tab, title)`, `Admin.Grid`, `Admin.Button`, `Admin.SetOn`, `Admin.TextBox`, `Admin.List`. These replace the old separate panels RollUpgradeTest, MutationRevealTest, CutsceneTest and CoinDropTestGui.
- **`ServerScriptService.GoblinAdminCommands` + `StarterPlayerScripts.GoblinAdminChat`:**
  - `/give all [n]` gives one (or n) of every goblin.
  - `/give <goblin> [n]` gives a specific goblin.
  - Allowed for the place owner, the listed admins, and everyone in a Studio test.
- **Admin lists:**
  - `GoblinInventoryServer.ADMINS` (`/admin` panel + admin remotes): 591297703 (the user), 2741325690 (friend), 2970683260, 1124895504.
  - `GoblinAdminCommands.ADMINS` (`/give`): 591297703, 2970683260, 1124895504.

## Studio test hooks (ServerStorage attributes, Studio only)
- `StudioGrantGoblins` = "Basic,Leaf,Zeus" gives those goblins.
- `StudioSetLuck` = 2 sets the luck multiplier, with the clover animation.
- `StudioGivePotions` = "lucky*5,damage*2" or "all*3" gives potions (PotionServer).
- `StudioGiveCoins` = 6000 gives coins; a negative value takes them away.
- ReplicatedStorage attribute `GoblinV4Test` = true: pets use the v4 / new models from `ReplicatedStorage.GoblinModelsV4` (Studio only).
  - The 36 new goblins and Sprout / Pip also get a temporary test design in `GoblinConfig.GetGoblin` (shown as Celestial, never rolled), so they can be equipped by setting `EquippedGoblins`.
- workspace attributes for GoblinV4Motion (Studio only):
  - `V4HoverStyle` = 1 / 2 / 3 switches the hover style.
  - `V4ClickTest` = the pet model's name (or "*" for all, repeating) plays the click animation.
  - `V4MotionOff` = true turns the layer off for an A/B comparison.
  - `V4MotionMs` (read-only) = the layer's Lua ms per frame.
- DataStore saving only works with "Game Settings > Security > Enable Studio Access to API Services".
- Client, Studio only: the `GoblinMenuGui` attribute `StudioOpenPanel` = "Inventory" / "Index" / "Settings" / "" opens or closes a panel from a script (for performance checks).
- Client, Studio only: the `GoblinExtrasGui` attribute `StudioOpen` = true / false opens or closes the EXTRAS list.
- Client, Studio only:
  - The `RollGui` attribute `StudioTestReveal` = "Huge,Rainbow" plays a display-only test roll; "Record" plays a New Best Roll test (the rarest design with a Rainbow combination, using the player's `CutsceneStyle` 1–14).
  - When the monitor is in standby (RenderStepped stops), the cutscenes can still be tested through a client-only harness: clone `RecordCutscenes` (and `RecordCutscenesExtra`) in the Play client with `RunService.RenderStepped` replaced by `Heartbeat`, require it with a stub env and call `MutReveal.cutscene(nil, face)`. Tweens do not run in standby, so this checks errors, timings and camera framing, not the looks.
  - `StudioSetHidden` = true / false switches Hide Rolls.
- Combat test (server, during Play): set the player attribute `EquippedGoblins` to `GoblinPets.EncodeEquipped({{i="t1", g="Pyro"}, …})` to equip any goblins without the inventory, move the character within ~25 studs of an enemy in `Workspace.EnemyPreview`, and set an enemy's `HP` attribute low to test deaths.

## Backups in ServerStorage
- `ClaudeBackups_20261003_inventory`
- `ClaudeBackups_20261003_menuUI`
- `ClaudeBackups_20261003_rollUI`
- `ClaudeBackups_20261003_personalities`
- `ClaudeBackups_20261004_nametags`
- `GoblinImportBackups`
- `GoblinModels_Before_NoPickaxes`
- `UI_Backup` (incl. `ForestAmbience_before_v4fx_20261008`, `V4WorldFX_v9_20261009`, `V4WorldFX_v11_20261009`, `GoblinMutationFX_before_shiny_less_light_20261009`, `RollClient_before_shownOdds_20261009`, `GoblinPetClient_before_coinfade_20261009`, `GoblinPetClient_before_coinclip_20261009`)
- `V4_Backup_Pack5_20261009` (116 parts that pack 5 replaced: all of the old link 9 incl. its pack-4 trees / bushes / vines, the merged Heaven groups EdgeGreen / ElementTop / RimCover / WallTop / WallElements and their 31 pack-4 hill trees; attribute `OrigParent`)
- `V4_Backup_AnimPack_20261008` (the merged V4 grass / flowers / reeds / lilies / clusters / bushes / clouds / windmill that the animation pack replaced; attribute `OrigParent`)
- `V4_Backup_Pack4_20261009` (208 parts that pack 4 replaced: cliffs with their vines, merged hill / edge / link groups, the sun gate vines, the Ice path / pavers / terrain, the deleted EdgeTree trunk; attribute `OrigParent`)
- `V4FX_DayLightingBackup` (the day lighting + sky from before the night look)
- the `RAG_*` map backups

Script backups are StringValues, because the MCP cannot clone scripts.

## Built elsewhere, but NOT in the connected Studio place (2026-10-05)
- **Mining and progression loop (`Mining_Assets/HANDOFF.md`, `PROGRESS_CHECKLIST.md`):**
  - Scripts: MiningConfig/Shared/Server, VillageServer, MiningClient/Menus, GoblinRevealFX (42 reveals), Economy and others.
  - Gameplay: ore, selling, upgrades, crafting, rebirth, and Zone 2 "Crystal Caverns v3".
  - None of these scripts or models are in the place that was open on 2026-10-05.
  - Possibly another copy of the place, or the work was undone. **Check before building on it.**
- **OreSpawner / OreStoneFX** (one big ore rock per Fernwood platform) are mentioned in memory, but were not found in Studio.
- **Zones 3–9** exist only in Blender: `Roll_A_Goblin_Zone1/Roll_A_Goblin_Zone1.blend`, described in `ZONES_1_TO_9_README_NL.md`.
  - The nine zones are: Fernwood, Crystal Cave, Magma Depths, Sunken Ruins, Toxic Swamp, Underground Jungle, Ice Cavern, Abandoned Mine and Shadow Void.
  - Every gate in the Blender file has a movable panel.
- **Full game plan:** `CLAUDE_COMPLETE_GAME_LOOP_EN.md`.

## Important folders
- `Goblin_Pets/`: local copies of the pet/inventory/roll/UI scripts (older than Studio), plus `PERSONALITIES_NL.md`.
- `Goblin_Variants/`: all variants with READMEs, plus `Nametag/` and `nameplate_concepts/`.
- `Goblin_UI/`: icon generators (`make_icons*.py`).
- `HUD_Mockup/`: the user's HUD mock-up, the icons cut out of it, the dock atlas and the cut/measure scripts.
- `Roll_A_Goblin_Zone1/`: the map (Blender + FBX).
- `Mining_Assets/`: the mining work (see above).
- `AI_HANDOFF_CURRENT/OVERDRACHT_NL.md`: a handoff from 2026-10-03, now historical.
- `CLAUDE_HANDOFF.md`: oldest and historical.

## Potions (2026-10-10)
- **Config:** `ReplicatedStorage.PotionConfig` (local `Goblin_UI/PotionConfig.luau`). All numbers are placeholders.
  - 8 potions: Lucky Brew, Greater Lucky Brew, Roll Speed Elixir, Damage Elixir, Gold Elixir, Mastery Elixir, Mutation Elixir, Overcharged Elixir.
  - Each has `id`, `name`, `icon` (the user's art), `duration` (s), `effects = { {kind, value} }`, the bar colours `bg`, `stripe`, `mark`, `markColor`.
  - Effect kinds and where they count:
    - `luck`: `GoblinInventory.publish` Luck × potions;
    - `rollSpeed`: RollServer cooldown + RollClient speed;
    - `damage` and `coins`: CombatServer;
    - `mutation`: `GoblinPets.RollMutations(mult)`;
    - `columns`: RollServer, capped at MAX_ROLL_COLUMNS;
    - `mastery`: attribute only (no Mastery XP yet).
  - Different potions of one kind add up. Using an active potion again adds its duration.
- **Server:** `ServerScriptService.PotionServer`.
  - Profile `data.potions = { owned = {id = n}, active = {id = end os.time} }`.
  - Remote `PotionRemotes.UsePotion(id)`.
  - Attributes: `PotionsOwned` / `PotionsActive` (JSON), plus `PotionLuck`, `PotionRollSpeed`, `PotionDamage`, `PotionCoins`, `PotionMastery`, `PotionMutation`, `PotionColumns`.
  - Expired potions are cleared once a second.
  - Give potions from other server scripts: `ServerStorage.GivePotion:Invoke(player, id | "all", n)`.
- **Page:** `StarterPlayerScripts.GoblinPotionsInvClient` (local copy in `Goblin_UI/`). An Inventory ext tab (GoblinPetClient tab `{ key = "Potions", ext = true }`), after the user's "Clean List" reference.
  - One bar per potion, 4 visible, scroll for more. Built in reference px (975 wide) in a UIScale; the menu's own Stage scale is measured with a probe.
  - Bar contents:
    - colour stripe;
    - the potion icon with a glow;
    - name, effect lines and time line (it shows "9:32 left" in green while active);
    - floating theme figures in the background;
    - the + button: opens the Shop (`OpenPanel` = "Shop");
    - the owned count;
    - Use Potion: green, or grey when you have 0.
  - Images: `Goblin_UI/make_potion_icons.py` → `Goblin_UI/potions/` (potion icons with their own alpha, stat icons clock / DNA / column / star, white mark silhouettes), ids in `asset_ids.json`.
  - Effect icons: the game's clover, dice (tinted blue), the goblins' damage icon, coin.
  - Texts: `GameText.Potions`.

## Titles (2026-10-10)
- **Config (the user edits names + requirements here):** `ReplicatedStorage.TitleConfig` (local `Goblin_UI/TitleConfig.luau`).
  - Each title has `id`, `name`, `stat`, `goal`, `text` (`%s` = the goal with commas), `special`, `colors` (name gradient) and `stroke`.
  - `stat` is one of: "rolls" (StatRolls), "kills" (StatKills), "ascensions" (StatAscensions).
  - All titles are placeholders: Goblin Keeper, Trailblazer, Lucky Streak, Riftwalker, Forest Guardian, First Ascend, Beyond Bound.
  - A title counts as unlocked as soon as the stat reaches the goal.
- **Server:** `ServerScriptService.TitleServer`.
  - Profile: `data.title` (the equipped id) and `data.kills` (counted from `ServerStorage.EnemyKilled`).
  - Remote: `TitleRemotes.EquipTitle(id | nil)`; only unlocked titles can be equipped.
  - Attributes: `EquippedTitle`, `StatKills`.
- **Page:** `StarterPlayerScripts.GoblinTitlesInvClient` (local copy in `Goblin_UI/`). An Inventory ext tab, after the user's reference (images/249), built in reference px (1180 wide) in a UIScale.
  - Banner:
    - their map shot, blurred: `Goblin_UI/make_title_images.py` → `titles/banner_blur.png`;
    - the player's AvatarBust thumbnail where the reference player stands;
    - the equipped title in its colours with wings (`wing_fill` / `wing_line`, tinted per title, the right wing mirrored) or "No Title";
    - "@username".
  - All / Special pills and a search box. Special shows `GameText.Titles.NoSpecial`.
  - Rows:
    - the name in its colours, with a star (unlocked) or a lock;
    - the requirement and "n / goal";
    - an equal-length bar (546 ref px);
    - Equip (green), Equipped (blue, click to remove) or Locked (grey + lock).
  - Texts: `GameText.Titles`.
  - Not shown anywhere else yet: no overhead tag or player-list title.

## Powers / Goblin Shrines (LIVE since 2026-10-09; see "How it works now" first)
- **How it works now:**
  - **Config:** `ReplicatedStorage.PowerConfig`. All numbers are placeholders; the user balances at the end.
    - `SHRINES[<internal zone>]` holds `name`, `rollTitle`, `token`, `tokenIcon`, `icon`, `cost` (10) and `powers`.
    - Each power has `id`, `name`, `rarity`, `chance` %, `luck` and `icon`.
    - Shrine 2 = Forest Shrine with 6 powers: Leaf Whisper, Rootbound, Forest's Blessing, Ancient Canopy, Heartwood, World Tree.
    - `TOKEN_DROPS[enemyId]` holds `shrine`, `chance`, `min` and `max`.
    - `FAVOR_MAX` 50, `FAVOR_MIN_RARITY` "Epic".
    - **Since 2026-10-09 (v2), powers count only when equipped:** `SLOTS` 2 unlocked, `SLOTS_MAX` 5. `LuckOf(data)` = the product of the equipped powers' luck at their stars. Also `EquippedIds`, `BestIds`. Profile `data.powers.equipped`, attribute `PowerEquipped`, remotes `EquipPower(id, on)` / `EquipBestPowers()`.
    - The Inventory > Powers page (`GoblinPowersInvClient` v2): Equipped Powers slots, shrine filters, card grid, detail with Equip / Unequip, Sort, Equip Best.
    - **New shrine = a new `SHRINES` entry + a part with attribute `ShrineZone`.**
  - **Server:** `ServerScriptService.PowerServer`.
    - Profile data: `data.powers = { tokens = {["2"]=n}, owned = {[id]=count}, favor = {["2"]=n} }`.
    - Player attributes (JSON): `PowerTokens`, `PowerFavor`, `PowersOwned`; plus `PowerLuck`.
    - `GoblinInventory.publish` sets `Luck` = `data.luck × PowerConfig.LuckOf`, so `RollServer`, the clover and the Luck Cap use it automatically.
    - Kills: `CombatServer` fires `ServerStorage.EnemyKilled(player, enemyId, pos)` for every helper. A token drop is followed by `PowerRemotes.PowerEvent("token", shrine, n, pos)`.
    - Rolls: `PowerRemotes.RollPower:InvokeServer(shrine)`.
      - It checks that the player is within 60 studs of that shrine's `ShrineZone` part, and that they have enough tokens.
      - It costs tokens and adds +1 Favor.
      - When Favor is full, that roll is guaranteed Epic+ and the bar starts again at 0.
    - Studio test: set the ServerStorage attribute `StudioGiveTokens` = n.
  - **Stars / Upgrade (v2):**
    - Profile `data.powers.stars[id]` (1..5), attribute `PowerStars`.
    - `PowerRemotes.UpgradePower:InvokeServer(id)` uses `UPGRADE_COST[stars]` spare copies.
    - Luck = `PowerConfig.LuckFor(p, stars)`.
  - **Shrine panel (v3, 2026-10-09):** 1:1 after `images/160.webp`, measured with pixel scans.
    - It is built in reference pixels (1242 × 995, origin (116, 32) of that image) inside a UIScale of 0.64.
    - Borders are nested rounded frames (outline → rim → line → fill). The colour tables `DARK` (carousel / Upgrade tile), `LIGHT` (Possible cards) and `RTEXT` (rarity words) are at the top of the script.
    - Favor bar on top; "Roll a <..> Power" with a smooth continuous carousel (cards morph with their distance to the middle); "Upgrade Power" (the middle power, gold / grey stars, Upgrade); "Possible Powers" as one row.
    - Icons come from `PowerConfig` (`icon` + `iconSize` = shown size in reference px). Since 2026-10-09 they are **the user's original art** (`Goblin_UI/extras/originals`, prepared by `Goblin_UI/prep_icons.py`); the same goes for the stars, arrows and the X. New powers: send the art and run `prep_icons.py`.
    - The v1 notes below describe the old layout.
  - **Shrine panel:** `StarterPlayerScripts.GoblinShrineClient` (local `Goblin_UI/GoblinShrineClient.client.luau`).
    - An external panel, key "Shrine", 880 × 560.
    - It opens when the player walks into the `ShrineTrigger` circle (through `GoblinMenuGui` attribute `OpenPanel`) and closes when they walk out.
    - Header: the shrine icon, the name, the token pill and X.
    - Left:
      - carousel (gold arrows browse it; a roll spins it to the result);
      - ROLL <cost> with the token;
      - Auto Roll switch (every `AUTO_ROLL_DELAY` s while the panel is open; stops when tokens run out);
      - Shrine Favor bar.
    - Right: "Possible Powers" as 3×2 square cards (chance, name, rarity, xN owned; a click puts the card in the carousel), plus "Your bonus".
    - It also draws the "+n token" popup where a mob died.
  - **Powers tab:** `StarterPlayerScripts.GoblinPowersInvClient`.
    - GoblinPetClient's tab list has `{ key = "Powers", ext = true }`. `selectTab` sets `GoblinMenuGui.InventoryTab`, and ext tabs skip the SoonPage.
    - The page lies next to SoonPage. Per shrine it shows a header (icon, name, tokens) and 6 cards: found ones in colour with Luck and Owned xN, the best one marked ACTIVE, the unfound ones as "???" silhouettes. On top: the total Power Luck.
  - **Sign:** `ShrineFX` builds the BillboardGui `ShrineSign` (‹ token "Forest Shrine" ›) from the model attributes `ShrineTitle` / `ShrineToken`.
    - Size 25 × 5.17 studs.
    - It fades out between 70 and 90 studs from the camera and is hidden beyond (`SIGN_FADE_NEAR` / `SIGN_FADE_FAR`).
  - **Icons:**
    - The 6 power icons are crops from the user's mock-up; the originals are still to come.
    - Forest Token: the user's own image, cut out.
    - Powers tab icon + shrine icon: cut from the user's images.
- **Open:** what goes in place of "Next guarantee"; stars for duplicates; more bonus types (coins, roll speed, damage); balance.
- **The original plan (2026-10-09):**
- **The idea** (the user, after a reference video of another game):
  - Mobs drop **tokens, one kind per zone**.
  - At the zone's **shrine** you spend tokens on rolls that give a random **power** in our own rarities (Common … Celestial).
  - Powers give a **luck multiplier** now; later also coins, roll speed, damage etc.
- **Not a copy:** our own names, numbers, UI in the house style and our own twists (Claude's proposal):
  - a shrine per zone with its own power pool;
  - tokens drop through our coin-drop system;
  - rolls use our own slot animation;
  - duplicates give ★ stars (★1–★5, a higher multiplier);
  - "Shrine Favor" (every roll fills it; full = the next roll is at least Epic) instead of pity bars;
  - power slots;
  - the active power floats as a rune by the player.
- **Balance** (roll price, chances, multipliers) is decided at the very end (the user).
- **The user made the token + its UI themselves.** Claude may only look at them when the user shows them.
- **Shrine model:** `Blend/Shrines.blend` (the user's file), built by `Blend/Shrines/build_shrine.py`.
  - Run it in Blender: `exec(open(...).read()); build()`. It (re)builds in the active scene.
  - Collection `Shrine_Fernwood`: 18 objects, one per material, named `Shrine_<Material>`, ~3.2k faces.
  - Collection `Shrine_Preview` (not part of the shrine): ground, player-size marker of 1.4 units = 5 studs, sun, world, camera.
  - The look:
    - a three-tier octagonal stone platform (purple-grey like the Fernwood cliffs) with tiles and a broken ring of glowing runes;
    - a stone arch with a gold trim and keystone, and a hanging gold ring with an emerald;
    - four broken pillars with moss and glowing runes, vines with leaves;
    - an altar with a rune strip, a stone bowl and four gold claws holding a big floating emerald crystal (glowing core);
    - three shards and three rune tablets round it, small glowing orbs;
    - roots, glowing pastel mushrooms (like the zone), flowers, grass and rocks.
  - Size: about 9 × 9 units (~32 studs); the arch top is at 5.3 units (~19 studs). Glow materials end in `_Glow`. No lights.

## Still open (big lines)
- Odds for Divine and Celestial.
- Enemy spawn chances + spawn locations (the user wants to decide these later).
- Coins now come from defeated enemies (2026-10-09). Still to decide: the real coin amounts per enemy, and the zone prices (all 1 coin for now).
- Building Zone 2 and opening the gate after unlock.
- Shop contents.
- Leaderboard scripts.
- Ascend contents (the button and an empty panel exist).
- Mobile testing.
- Testing DataStore saving.
- **A full performance audit when the game is finished** (the user's plan, 2026-10-06):
  - client: render CPU/GPU, draw calls and memory per system (pets + FX, nametags, enemies, UI, map) in a busy scene;
  - server: Heartbeat time and the cost of the enemy AI and combat;
  - network traffic;
  - low-end and mobile settings;
  - the result is a list of the biggest costs with fixes.
  - Until then every new feature is still measured on its own (see the rule from 2026-10-06).

---

# 2. Log (newest at the top)

> **Working rule (the user, 2026-10-10):** if the user is in Play while a chat needs Edit mode, the chat may ALWAYS stop Play and go to Edit (no waiting).

## 2026-10-10 — Inventory > Titles tab (banner with avatar + title, All / Special, progress rows) (Claude)
- What: the Titles page after the user's reference.
  - **Banner:**
    - their windmill map shot, blurred, as the background;
    - the player's own avatar (bust) where the reference player stands;
    - next to it the equipped title in its colours with wings in the title colour, and "@username" under it.
  - **All / Special + search.** Special shows "There are no special titles available at this time!".
  - **Rows:**
    - title name in its own gradient, with a star when unlocked or a lock when locked;
    - requirement and "n / goal";
    - progress bars all the same length;
    - Equip / Equipped (click again to remove) / Locked.
  - Names, requirements, goals and colours are placeholders in `TitleConfig` (easy to edit).
  - Kills now count (`data.kills`) for "Defeat N mobs".
- Where:
  - New: `ReplicatedStorage.TitleConfig`, `ServerScriptService.TitleServer`, `StarterPlayerScripts.GoblinTitlesInvClient`, `TitleRemotes`.
  - Changed:
    - `GameText`: `Titles`, pushed via UpdateSourceAsync because it is open in a script tab;
    - `GoblinPetClient`: the Titles tab is an ext tab.
  - Backups: `ServerStorage.UI_Backup.GameText_before_titles`, `GoblinPetClient_before_titles`.
  - Local: `Goblin_UI/TitleConfig.luau`, `TitleServer.server.luau`, `GoblinTitlesInvClient.client.luau`, `make_title_images.py`, `titles/` (banner_blur 102787413438335, wing_fill 74167114122678, wing_line 139272135166096).
- Test (Play):
  - StatRolls 4,500 / StatKills 120 / StatAscensions 2: Goblin Keeper and Trailblazer unlocked (star, Equip); the others Locked with partial bars.
  - Equip Trailblazer: the banner shows it with gold wings and the row turns into Equipped (blue edge).
  - Special shows the message.
  - No errors.
- Still open:
  - Show the equipped title in the world / player list?
  - Real requirements.
  - Ascensions do not exist yet (stays 0).

## 2026-10-10 — Inventory > Potions tab (8 potions, working effects) + admin "Reset progress" (Claude)
- What:
  - **Potions tab:** after the user's "Clean List" reference, with their own 8 potion icons.
    - 4 bars per page, scroll for the rest.
    - The + opens the Shop. The number is how many you own.
    - Use Potion is green, or grey at 0.
    - The background figures float and sway.
    - The + was redone on request: chunky round arms with a soft shadow.
    - While a potion runs, its time line shows "m:ss left".
  - **Potions work:**
    - Luck (also the clover and the Luck Cap): tested 1 → 2.5 with Greater Lucky Brew.
    - Roll speed: server cooldown + client animation.
    - Damage and coins from enemies.
    - Mutation chance.
    - +Roll Column (capped at max 5).
    - Mastery is only stored, because there is no Mastery XP yet.
    - The numbers are placeholders.
  - **Admin window "Data" tab:**
    - "Reset progress" (second click confirms) resets your own profile and keeps your settings.
    - "Alle potions +5".
- Where:
  - New: `ReplicatedStorage.PotionConfig`, `ServerScriptService.PotionServer`, `StarterPlayerScripts.GoblinPotionsInvClient`, `StarterPlayerScripts.AdminDataTools`, `PotionRemotes`, `ServerStorage.GivePotion`.
  - Changed:
    - `GoblinInventory`: PotionConfig luck in publish, `ResetProgress` and the `ProgressReset` event.
    - `GoblinInventoryServer`: AdminAction resetProgress / potions.
    - `PowerServer`: republishes on reset.
    - `RollServer`, `RollClient`, `CombatServer`, `GoblinPets` (`RollMutations(mult)`).
    - `GoblinPetClient`: the Potions tab is an ext tab.
    - `AdminPanelUI`: TAB_ORDER adds "Data".
    - `GameText`: `Potions` + Admin Data texts. It was open in a script tab, so it was pushed with ScriptEditorService:UpdateSourceAsync.
  - Backups: `ServerStorage.UI_Backup.*_before_potions` / `*_before_reset` / `AdminPanelUI_before_data`.
  - Local: `Goblin_UI/PotionConfig.luau`, `PotionServer.server.luau`, `GoblinPotionsInvClient.client.luau`, `make_potion_icons.py`, `potions/` (images + asset_ids.json).
  - The local `Goblin_UI/GameText.luau` is behind Studio.
- Test (Play):
  - `StudioGivePotions` = "all*3": the counts show and the 4-bar layout matches the reference (captures).
  - UsePotion: Luck 1 → 2.5 and the time line counts down. Using one you don't have shows a "none left" message.
  - Reset progress: coins 1M → 0, goblins 2 → 0, potions → {}, Luck back to 1.
  - No errors.
- Still open:
  - The Shop has no potions yet (the + opens the Coming soon Shop).
  - Balance the numbers.
  - There is no HUD display of active potions.
  - Mastery XP doesn't exist yet.

## 2026-10-10 — Record cutscenes: goblin readable in the dark scenes again (Claude)
- What:
  - After the goblin's own glow was removed, the goblin was almost a silhouette in Supernova and other night cutscenes.
  - Local lights barely reach it there: a camera fill SpotLight and a test PointLight of 5 changed nothing. Ambient light does work.
  - So while the puppet is on stage, `spawnPuppet` keeps `Lighting.Ambient` / `OutdoorAmbient` at a floor of at least 0.56 (blue channel ×1.08). Values are only raised, never lowered.
  - Lighting is restored after the cutscene, as before. No glow is added to the goblin.
- Where: Studio `ReplicatedStorage.RecordCutscenes` (`spawnPuppet`, `FLOOR`). Backup: `ServerStorage.UI_Backup.RecordCutscenes_before_fill_light`.
- Test (Play, `StudioTestReveal` = "Record"): Supernova (12) and Kristalgeode (2) — the goblin is clearly visible with its colours. Steencirkel (14) is unchanged. No errors.
- Still open: if a scene should be darker / lighter, change `FLOOR`.

## 2026-10-10 — All goblins upright (icons + game), cutscene rarity pill, white cutscene name, less glow in the cutscene (Claude)
- What:
  - **Upright goblins (the user: "alle goblins niet meer schuin, het is lelijk"):** every v4 pose tilt is now {0,0,0}.
    - Changed in `GoblinV4MotionData` (Studio + `Goblin_Pets/GoblinV4MotionData.luau`) and in `Goblin_Concepts/physics_parts.json`.
    - The 14 that were still tilted: Deepwake, Flowbound, Ivory, Kilnborn, Liminal, Phoenix, Resinborn, Resonant, Ringmaster, Runescribe, ShadowStalker, Skyweaver, Unwritten, Veilkeeper. Manyfold and Puppeteer were already done.
    - The glide lean while moving and the click / attack leans stay.
  - **Icons:** `render_v4_icons.py` now always renders upright: the display parent is reset to (0, 5.4°, 0), because the tilt only lives there.
    - Re-rendered the 16 tilted goblins, ran `make_v4_icons.py`, uploaded 48 images and replaced their `GoblinConfig.V4_ICONS` entries.
    - Ids are saved in `Goblin_Icons_V4/asset_ids.json`.
  - **Blender file:** `Goblin_Library/09_new_Omniversal_Absolute.blend` now has Manyfold and Puppeteer straight (`v4_pose` tilt 0, display rotation reset, Manyfold Head held pose cleared, `v4_legs` null).
    - Script: `Goblin_Concepts/straighten_manyfold_puppeteer.py`. Backup: `09_..._before_straight.blend`.
    - The other library files still hold their old display tilt and `v4_pose`; the icon render ignores it.
  - **Record cutscene title card:**
    - The rarity pill is now the same as the hover card / index pill: gradient fill for Divine+, turning border.
    - That pill moved to `GoblinUIKit.StyleRarityPill` / `RarityEdge` / `SpinGradient` / `RarityColor`. GoblinPetClient now uses the kit versions.
    - The goblin's name is plain white with a dark outline. The mutation words keep their colours.
  - **Less light in the cutscene:**
    - The puppet's own lights are off.
    - Its particle LightEmission is capped at 0.15, re-applied every 0.5 s because the FX modules switch things on later.
    - Cutscene bloom: Intensity × 0.45, Threshold ≥ 0.96.
- Where:
  - Studio:
    - `ReplicatedStorage.GoblinUIKit`
    - `ReplicatedStorage.RecordCutscenes` (`spawnPuppet`, `applyLook`, `titleCard` footer + name)
    - `StarterPlayerScripts.GoblinPetClient` (top: Kit aliases)
    - `ReplicatedStorage.GoblinConfig`
    - `ReplicatedStorage.GoblinV4MotionData`
  - Backups `ServerStorage.UI_Backup.*_before_cutscene_pill`, `GoblinConfig_before_straight_icons`, `GoblinV4MotionData_before_all_upright`.
- Test (Play):
  - The inventory shows the new upright icons (Manyfold x2, Puppeteer, Unwritten, Skyweaver, Flowbound, Deepwake, Liminal).
  - Record cutscenes (`RollGui` attribute `StudioTestReveal` = "Record", styles 14 and 3): white name, gradient pill, no glare, puppet lights 0.
  - 6 formerly tilted goblins equipped: no errors.
- Still open: if one of them should keep a small lean, set its tilt in `GoblinV4MotionData` again.

## 2026-10-10 — Equipping moves a goblin out of the inventory grid into the equipped row (Claude)
- What:
  - In the inventory grid, each card's count is now the free copies (n − equipped). Equipping one of 34 shows "x33" in the grid and puts 1 in the equipped row.
  - A stack with every copy equipped is no longer shown in the grid. Unequipping brings it back.
  - The green tick and outline on grid cards are gone.
  - The hover tooltip still shows the total owned. The server is unchanged: it already equipped copies per stack.
- Where: Studio `StarterPlayer.StarterPlayerScripts.GoblinPetClient`:
  - `makeCell`: `anyEquipped = false`, count = `stack.free`;
  - grid build: only `s.free > 0`; Sig = `free > 1`; the count update uses `s.free`.
  - Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_equipmove`.
- Test (Play): `StudioGrantGoblins` = "Arbiter:Huge+Shiny+Rainbow*34,Leaf".
  - Equipping Arbiter: grid x34 → x33, and 1 in the equipped row.
  - Equipping Leaf (x1): Leaf left the grid.
  - Unequip All: back to x34 + Leaf.
  - Capture checked, no errors.
- Still open: nothing.

## 2026-10-10 — Goblins hardly glow any more (aura + Rainbow light minimal) (Claude)
- What: the user was blinded by the glow of a Rainbow Huge Shiny Caliber, so goblins now give off at most a very small amount of light.
  - **High rarities:** the aura's body glow cloud (`auraBody`) was almost opaque at high ranks: transparency 0.86 − 0.018·rank, LightEmission 1.
    - Now: transparency 0.93 − 0.002·rank, LightEmission 0.15.
  - **Aura light:** `auraLight` PointLight brightness min(cfg.light, 0.2), range 5. It was up to 1.2, range 7.
  - **Ground aura:** SurfaceGui Brightness 2.2 → 1.2.
  - **Motes:** LightEmission 1 → 0.3.
  - **Rainbow:**
    - RainbowLight 0.42 / range 5 → 0.12 / range 4;
    - RainbowSparkles LightEmission 1 → 0.35;
    - RainbowGlintSparkles LightEmission 1 → 0.3.
  - Shiny was already minimal and is unchanged.
- Where:
  - Studio `ReplicatedStorage.GoblinPetFX` (aura section).
  - Studio `ReplicatedStorage.GoblinMutationFX` (Rainbow setup).
  - Backups: `ServerStorage.UI_Backup.GoblinPetFX_before_lesslight` / `GoblinMutationFX_before_lesslight`.
- Test: in Play I equipped Caliber Huge, Shiny, Rainbow via `EquippedGoblins`. There was no white glow cloud any more, and no errors.
- Still open: if it should be zero light, set the `auraLight` / `RainbowLight` brightness to 0.

## 2026-10-10 — Puppeteer strings stay on the control cross; Puppeteer + Manyfold stand straight (Claude)
- What:
  - **Puppeteer strings:** the 6 string meshes (`<Bone>_puppetString`) were skinned only to their arm / cuff / shoulder / bow / bell bone, so their tops came loose from the moving ControlCross and glitched.
    - They are now hidden (Transparency 1, kept in the model).
    - In their place are 6 Beams `<Bone>_String` (width 0.035, the string colour, FaceCamera).
    - Each beam runs from a Bone `<Bone>_StringTop` under the `ControlCross` bone to a Bone `<Bone>_StringEnd` under its own bone, so both ends follow the rig exactly.
    - The cross keeps its hinge, click and attack swings.
  - **Tilt:** Puppeteer tilt {0,6,0} → {0,0,0}; Manyfold tilt {0,7,0} → {0,0,0}, and its held Head tilt was removed.
    - This was changed in `GoblinV4MotionData` (Studio + `Goblin_Pets/GoblinV4MotionData.luau`) and in `Goblin_Concepts/physics_parts.json`.
    - The glide lean into motion and Manyfold's click hop dance are unchanged.
- Where:
  - Studio: `ReplicatedStorage.GoblinModelsV4.Puppeteer` (beams + string bones are a Studio addition, so keep them when re-syncing from Blender). Backup: `ServerStorage.Puppeteer_before_beamstrings`.
  - Studio: `ReplicatedStorage.GoblinV4MotionData`.
- Test: in Play I equipped Puppeteer + Manyfold via `EquippedGoblins` = `GoblinPets.EncodeEquipped(...)`. The 6 beams ran from the cross to the arms, and both pets stood upright. No errors.
- Still open:
  - A re-export with `export_v4_motion_data.py` would bring back Manyfold's held Head tilt from `held_poses.json` (Blender rig pose). Remove it there too, or set the Head pose to rest in the Blender library file.

## 2026-10-10 — Rarity pills for every rarity, turning edge on the hover card, short odds (K / M / …, max 2 decimals) (Claude)
- What:
  - `styleRarityPill` now styles every rarity: Divine and rarer filled with their gradient, Common → Mythic with their rarity colour; all get the turning border (light → colour → dark, 3.5 s per round) and white text outlined in the dark colour. Helpers `rarityEdge(rarity)` and `spinGradient(g)`.
  - The hover card's edge (`cardEdge`) is a turning gradient in the hovered goblin's rarity colours; the tail's edge takes the dark colour.
  - **Odds everywhere short:** new `GoblinPets.ShortNumber(n)` = K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc with at most 2 digits after the point, cut off (700000 → 700K, 7290 → 7.29K, 2.0111e23 → 201.11Sx). `RatioText` / `OddsText` use it (the `decimals` argument is ignored now); `PlayerListClient` `short()` follows the same rule. The podium / record cutscene keep their full numbers (the user's earlier wish).
- Where: `StarterPlayerScripts.GoblinPetClient` (backup `GoblinPetClient_before_pill_all`), `ReplicatedStorage.GoblinPets`, `StarterPlayerScripts.PlayerListClient`.
- Test (Play): cards show 1/700K, 1/100K, 1/25.6K, 1/3.2K, 1/4, 1/2; the Scout hover card shows a green UNCOMMON pill with turning border and a green turning card edge. No errors.

## 2026-10-10 — Rarity gradient on the inventory cards + gradient rarity pill (Claude)
- What: the inventory / index card odds still used the old light → colour gradient (the earlier replace had not applied); they now use `GoblinConfig.RarityGradient` / `RarityStroke` like the equipped row. New `styleRarityPill(pill, label, rarity)` in `GoblinPetClient`: for Divine and rarer the rarity pill (hover card, index detail) gets the rarity gradient as fill, a 2.5 px border with a gradient (light → end colour → outline colour) that turns round (TweenService, 3.5 s per turn, endless) and white text outlined in the outline colour. Other rarities keep the navy pill.
- Where: `StarterPlayerScripts.GoblinPetClient` (backup `ServerStorage.UI_Backup.GoblinPetClient_before_rarity_pill`).
- Test (Play): hover cards of Nebula (CELESTIAL) and Veilkeeper (BEYOND) show the gradient pill with the turning border; card odds match the equipped row. No errors.

## 2026-10-10 — New rarities Astral → Beyond with the user's gradients; odds for Divine / Celestial; 36 new goblins in the catalog (Claude)
- What:
  - **Rarities** (the user's Figma design: vertical gradient + outline, Montserrat Bold): Divine 1/100K, Celestial 1/200K, Astral 1/300K, Cosmic 1/400K, Transcendent 1/500K, Primordial 1/600K, Eternal 1/700K, Infinite 1/800K, Omniversal 1/900K, Absolute 1/1M, Beyond 1/1.1M. Each has `Gradient` (ColorSequence from their stops) and `Stroke` in `GoblinConfig.Rarities`; `Color` = the gradient's middle. Helpers `GoblinConfig.RarityGradient(name)` / `RarityStroke(name)` (nil for Common–Mythic: those keep their light → colour gradient).
  - Used by: inventory + index card odds and the equipped-row odds (`GoblinPetClient`), the roll odds (`RollClient`), the world nametag odds (`GoblinTagFX` palette `oddsSeq`), the player list's Rarest Roll (`PlayerListClient`).
  - **The 36 new goblins** are real catalog designs now (`addNew`, 4 per rarity as built, own descriptions + theme colours, `V4Only = true`, icons from `V4_ICONS`). A rarity whose designs are all `V4Only` gets Weight 0 unless `V4Active()` (the live game has no v4 models yet). Sprout / Pip stay test designs.
  - Combat stats (`CombatConfig.RARITY_STATS`, doubling up to Beyond 65,536 dmg / 7,600 HP) and auras (`GoblinPetFX` rarity table, ranks 9–17, light kept at 1.2) for the new rarities.
  - **Empty-slot silhouette** = the original Basic icon again (`GoblinPetClient` `SILHOUETTE` = 92019507826785, no v4 bindle stick).
- Where: Studio `GoblinConfig`, `CombatConfig`, `GoblinPetFX`, `GoblinTagFX`, `GoblinPetClient`, `RollClient`, `PlayerListClient`. Backups `ServerStorage.UI_Backup.*_before_new_rarities`.
- Test (Play): granted one goblin of every rarity Divine → Beyond: the cards show 1/100,000 … 1/1.1M in each rarity's gradient + outline, the hover card shows e.g. INFINITE / 8,192 dmg / 4,000 HP, the empty equipped slots show the plain silhouette. No errors.
- Still open: the new goblins have no own equip effect yet (`EquipEffects` in the renderer); live: Divine (1/100K) and Celestial (1/200K) are now rollable after the next publish.

## 2026-10-10 — v4 icons for all 80 goblins + mutations on the v4 models (Claude)
- What: the user asked to update the icons, put them in the inventory and everywhere icons are used, and check the mutations on the new models.
  - **Icons:** all 80 v4 / new / admin goblins rendered in Blender with the old icon camera and turned into the 3 icon types (inventory 512, roll reveal 1024, Shiny sweep 512), uploaded (240) and set in `GoblinConfig.V4_ICONS` (applied in the v4 Studio test). Inventory grid, equipped row, index (old 42) and the roll use them through the existing `Image` / `RevealImage` fields.
  - **Mutations:** Shiny and Rainbow did not fit the v4 models (new texture ids, accessories on a palette texture, the skin spread over several meshes). New per-texture layers in `GoblinMutationAssetsV4` (157 Shiny + 78 Rainbow textures uploaded); `GoblinMutationFX` and `RollClient` use them on v4 models. Huge was already fine.
- Where: Studio `ReplicatedStorage.GoblinConfig` (`V4Active`, `V4_ICONS`), `GoblinMutationAssetsV4` (new), `GoblinMutationFX`, `StarterGui.RollGui.RollClient`. Backups `ServerStorage.UI_Backup.GoblinMutationFX_before_v4`, `RollClient_before_v4icons`, `GoblinConfig_before_v4icons`. Local: `Goblin_Concepts/render_v4_icons.py`, `make_v4_icons.py`, `make_v4_mutation_textures.py`, `v4_skin_colours.py`; `Goblin_Icons_V4/` (renders, icons, textures, `asset_ids.json`, `save_ids.ps1` upload helper).
- Test (Play, GoblinV4Test on): granted King Shiny, Wayfinder Rainbow, Tollkeeper Huge, Ivory, Seraph, Bramble, Mushroom, Sprout, Ringmaster, Basic, Leaf, Pyro → the inventory shows the new icons; a test roll reveals the v4 Basic icon. Equipped King Shiny / Ivory Shiny / Leaf + Wayfinder Rainbow / Sleepy Huge Shiny Rainbow / Sprout Shiny: every textured mesh got its layer (e.g. Sleepy 23/23 Shiny, 4 skin meshes Rainbow, scale 1.6×), no errors.
- Still open:
  - the 36 new goblins + admin pets are still test designs ("1/?" odds, no index page) until they get real rarities in `GoblinConfig`;
  - the per-goblin icon effects (`GoblinIconFX`) were tuned for the old framing; goblins drawn smaller may need small offset tweaks;
  - when v4 goes live: apply `V4_ICONS` and `GoblinMutationAssetsV4` always (drop the `V4Active` checks).

## 2026-10-10 — Admin window "Pets" tab: hover style + attack test (Claude)
- What: new Studio-only admin tab to pick the hover style, loop attacks / clicks, freeze poses and equip test goblins (see section 1, Admin → Pets).
- Where: `StarterPlayerScripts.AdminPetTest` (new), `AdminPanelUI` (TAB_ORDER + Pets), `GameText.Admin` (Pets texts), `GoblinInventoryServer` (AdminAction "v4equip:"). Backups `ServerStorage.UI_Backup.*_before_adminpets`.
- Test: Play: the tab shows all 3 sections; "Zwevers" equip worked (6 floaters equipped). The other buttons were not confirmed by click (MCP clicks after the first didn't land); they only set workspace attributes.
- Still open: the user picks the hover style, then set `Motion.HOVER_STYLE`.

## 2026-10-10 — v4 goblins: own themed click + accessory attacks for all 80, no spinning, smoother follow (Claude)
- What: the user's feedback on the first v4 animation round:
  - every goblin needs its own click animation that fits its theme, and no spinning round while floating;
  - pets must follow more smoothly and react faster;
  - attacks must match each goblin's style, cooler with rarity, and really use its accessories (the user's example: Ivory attacks with the tusks on its back).
- Done:
  - `GoblinV4Motion`: the tier clicks, idle flourishes and Helix's self-spin are gone. A choreography DSL (`newChor`, `K` table) gives all 80 goblins a themed click, and the new `attack(...)` section gives all 80 an accessory-based attack (≤ 0.95 s). Item overrides (`spin` / `swing` / `move`) work on both moving and rigid accessories.
  - `GoblinAttacks`: `A.Play` uses the v4 attack's hit time; `ANCHORS` + `liveAnchors` make the effects start at the accessory's live bone. Effects reworked for Sleepy (pillow feathers), Crystal (lamp beam), Engineer (claw zap instead of jetpack), Stormbringer (bolts from its cloud + sky bolt), Volcanic (volcano eruptions), Deepwake (bubbles from both hoses).
  - `GoblinPetRenderer` follow constants tuned (see section 1, "Follow tuning").
  - Studio hooks `V4AttackTest` and `V4ChoreoT` added.
- Where: Studio `ReplicatedStorage.GoblinV4Motion` (= local `Goblin_Pets/GoblinV4Motion.luau`, hash-identical), `ReplicatedStorage.GoblinAttacks` (backup `ServerStorage.UI_Backup.GoblinAttacks_before_accessory_fx`), `ReplicatedStorage.GoblinPetRenderer` (Studio only; the local copy is behind).
- Test: in Play, poses checked frozen at the hit (`V4AttackTest = "*"`, `V4ChoreoT = 0.42`) for Ivory, Knight, Sleepy, Moorbound, Silkenveil, DragonRider. Fixes after that check: Ivory's tusks swung too far down (−80 → −55), and the spine twist on throws / sweeps was reversed in 15 places. A real fight on the Elder Root Golem with Pyro, Cyborg, DragonRider, Volcanic, Stormbringer, Deepwake ran with no errors. The attack choreography really runs there (Volcanic's volcanoes pump 0.35 studs per hit). Layer cost 0.29 ms per frame, ~200 FPS.
- Still open:
  - the user should look at the attacks in a real fight; only 6 poses were checked visually;
  - the user still has to pick a hover style (1 / 2 / 3);
  - effects on FX_* bones / glow parts, Shiny on v4, going live (`GoblinV4Test`).

## 2026-10-10 — v4 animations: held poses, secondary motion, rarity flair, click animations, 3 hover styles (Claude)
- What: the user asked for well-crafted animations for the v4 / new goblins: the rarer, the cooler, without lag and without hurting the looks. During the work they also asked for:
  - new click animations that use all the new bones;
  - no more waving etc.;
  - legs that move while floating;
  - arms and legs doing something while floating;
  - 3 floating variants to choose from.
- Built: the new module `ReplicatedStorage.GoblinV4Motion` and its data module `GoblinV4MotionData` (section 1, "Pets in the world").
  - **Held poses:** Basic's bindle arm, King's scepter, Vigil's folded hands, Resonant's sitting pose, etc.
  - **Hover glide** with legs and arms in motion.
  - **Secondary motion** on ~1,400 extra bones: capes, robes, chains, tails, wings, orbiters and mechanisms.
  - **Rarity flair** with idle flourishes from Legendary up.
  - **Click animations** by tier, during which every part goes wild.
- Changed:
  - `GoblinPetRenderer`: the hooks; `noWave` on v4 models.
  - `GoblinPetFX`: v4 models get no old cape / wing / hat rigs, no `def.hold` / `def.idle`, no random gestures; a click goes to V4Motion.
  - `GoblinConfig.GetGoblin`: test designs for the new goblins in Studio test mode.
- Backups: `ServerStorage.UI_Backup.GoblinPetRenderer_before_v4motion`, `GoblinPetFX_before_v4motion`, `GoblinPets_before_v4test`.
- Where: Studio, plus the local copies `Goblin_Pets/GoblinV4Motion.luau` and `Goblin_Pets/GoblinV4MotionData.luau`. The generator is `Goblin_Concepts/export_v4_motion_data.py`.
- Test (Play on a test platform, GoblinV4Test on):
  - All 80 viewed in groups of 6: nothing broken or floating loose.
  - Tested in motion: King's cape (lift reduced afterwards: it went horizontal), Wayfinder's glide, Tollkeeper's bell and infinity flourish (logged), Basic's held arm, the clicks of six tiers.
  - Cost: ~0.3 ms Lua per frame for the 6 heaviest; render CPU +0.4 ms; 120 FPS before and after. No console errors.
- Still open:
  - **The user picks a hover style** (`V4HoverStyle` 1 / 2 / 3), then set `Motion.HOVER_STYLE`.
  - Fine-tuning per goblin after the user's review (the SIG table).
  - Shiny's SurfaceAppearance still targets the old meshes.
  - Effects on the `FX_*` bones and the glow parts (next step).
  - Going live (GoblinModels → v4, the new goblins in GoblinConfig).

## 2026-10-10 — 80 goblin FBXs in Studio as templates + v4 test switch (Claude)
- What:
  - The user imported all 80 FBXs into the Workspace. They came in at import scale (about 120 studs wide), floating above the map, with 3× Leaf_v4 and 2× Sleepy/Scout.
  - **Templates:** I built `ReplicatedStorage.GoblinModelsV4` (80 models) from the best import of each goblin. For each one:
    - scaled exactly like the current templates (`GetScale` 0.03), pivot at the origin;
    - `InitialPoses` removed;
    - model attributes `Rarity` / `Kind` / `SourceFbx`, plus `GlowColor` on the glow parts.
  - **Part names fixed:** the user had imported the first export, and Studio cuts names over 50 characters to 23 + "..." + 24.
    - The 16 v4 goblins with glow parts were rebuilt from the raw import. Each part was matched to its current export name by size and position: 115 parts, no mismatch (largest error 0.004).
    - The new goblins' `<Name>_Body.001` became `<Name>_Body`.
    - The Alchemist flask (untextured in the first export) got its colour plus Transparency 0.82.
  - The raw imports went to `ServerStorage.GoblinImport_Raw` (+ `Duplicates`).
  - **Test switch:** `GoblinPets.GetTemplate` reads `GoblinModelsV4` first when the ReplicatedStorage attribute `GoblinV4Test` is set (Studio only; it is on now). New constant `GoblinPets.V4_FOLDER`. Backup: `ServerStorage.UI_Backup.GoblinPets_before_v4test`.
- Where: `ReplicatedStorage.GoblinModelsV4`, `ReplicatedStorage.GoblinPets`, `ServerStorage.GoblinImport_Raw`. The name map is made by the scratch script `name_map.py` (old → new names from the library).
- Test:
  - All 80 pass `GoblinPets.ValidateTemplate`; no cut-off names are left.
  - In Play I equipped 6 goblins (Seraph, Volcanic, Leaf, King, Phoenix, Basic) via `EquippedGoblins`. They spawned as v4 (Seraph has 86 parts), at the right size, followed the player with the shared motion, and got the normal FX (aura / tag). No errors, and no double capes or wings.
- Still open:
  - **Animations:** the extra bones (capes, tails, chains, wings) don't move yet. The held poses (`arm_hold` / `leg_pose` in `physics_parts.json`) aren't applied yet.
  - **Effects:** the glow parts (`GlowColor`), the `FX_*` bones, and an equip effect for each new goblin.
  - **Going live:** swap `GoblinModels` for v4; add the 36 new goblins and their rarities (Astral → Beyond) to `GoblinConfig` (odds, colours, icons); add Sprout / Pip as admin pets.

## 2026-10-10 — FBX export fix: Leaf body untextured, flask, part names (Claude)
- What:
  - In Studio, the Leaf_v4 body arrived without a texture. Its body has 13 flat-colour materials and no atlas, but the exporter only kept material slot 0.
    - Fix: flat body colours now go into the goblin's palette.
    - The body now always gets exactly one material per exported mesh. A body that mixes an atlas with flat colours would be split into `<Name>_Body` and `<Name>_Body_Palette`. None do that right now.
  - The Alchemist flask material had an image node that wasn't wired to anything, so it would have arrived untextured too.
    - Fix: `tex_image` now only counts images wired into Base Color.
  - Clean part names: `<Bone>_<colour>` without the "Goblin |" prefix and without `.001`. Export objects take their exact name; library objects with that name are renamed in memory only, never saved.
  - The manifest now has `transparent` (part → Roblox Transparency) and `glow` (part → emission colour) per goblin. The palette stays opaque, so Studio has to apply these values itself. 16 v4 goblins have glow parts; the Alchemist flask has transparency 0.82.
- Where: `Goblin_Concepts/export_goblins_fbx.py`. All 80 files in `Goblin_Export/` were re-exported, along with `export_manifest.json`.
- Test: every FBX was re-imported on its own. All 80 files passed: every mesh skinned, every mesh textured, one material per mesh, no `.001` names. A render showed the Leaf body green, with its eyes and face.
- Still open: re-import Leaf_v4 into Studio. Glow and transparency still have to be applied in Studio from the manifest.

## 2026-10-10 — All 78 goblins + 2 admin pets exported as FBX for Roblox (Claude)
- What: the user asked for Roblox export files for all 78 goblins (to import in Studio and then work on animations and effects). They also asked for 2 concepts from the archive, which become **admin pets**: Concept 6 "Sprout" and Concept 7 "Pip".
  - 80 FBX files: 42 v4 redesigns, 36 new goblins, Sprout and Pip.
  - The script is `Goblin_Concepts/export_goblins_fbx.py`. It runs in a background Blender, only reads the library files and never saves them:
    ```
    blender -b --factory-startup --python Goblin_Concepts/export_goblins_fbx.py [-- <file.blend> ...]
    ```
    With no file arguments it exports every `Goblin_Library/NN_*.blend`. It takes about 50 s.
  - It follows the original goblin FBX convention:
    - the rig sits at the origin in its bind pose, with no display parent;
    - MESH + ARMATURE, −Z forward, Y up, bake space transform;
    - no leaf bones; textures embedded.
  - **Merge per bone + colour:** every flat colour becomes its own skinned mesh `<Bone>_<colour>`, 100 % on that bone. The body stays one skinned mesh, `<Name>_Body`.
  - Flat colours also go into a 64×64 palette PNG (`<Name>_palette`, one 4 px cell per colour). Textured pieces keep their atlas.
  - **All bones are kept**, including the `FX_*` effect bones and the accessory bones.
- Where:
  - `Goblin_Export/v4/<Name>_v4.fbx` (42), `Goblin_Export/new/<Name>.fbx` (36), `Goblin_Export/admin/Sprout.fbx` and `Pip.fbx`.
  - `Goblin_Export/textures/` holds the loose PNGs, which are also embedded in the FBX files.
  - `Goblin_Export/export_manifest.json` lists per goblin: rarity, file, meshes, triangles, bones, FX bones and colours.
  - Each goblin has 10–106 meshes and 0.5k–24k triangles. Seraph_v4 has the most triangles (24.4k); Vigil has the most meshes (106).
- Test:
  - Re-imported Sprout, Pip, Basic_v4, Seraph_v4, Unwritten and Vigil in Blender. Every mesh was skinned, the bones and FX bones were present, the textures loaded, and an arm rotation moved everything attached to that arm.
- Still open:
  - Import into Studio with the 3D Importer (you can select several files at once), then set up the animations and effects per `physics_parts.json`.
  - Sprout and Pip don't exist in `GoblinConfig` yet (admin-only pets).
  - The files are 1–6 MB each, because Blender writes a skin cluster for every bone in every mesh. Studio imports them fine.

## 2026-10-10 — New goblins: own skin colours, head pieces, detailed faces (Claude)
- What: the user asked why the new goblins (Astral and up) were all green with little on their heads. Their faces should also get much more detail.
  - **Skin:** each new goblin now has its own skin colour. It is a recoloured copy of the Basic atlas: the 3 skin greens become the goblin's shades, and Basic's brown under-clothes become its cloth colour. The atlas is packed as `<Name>_skin_atlas` on an object-linked material `<Name> skin`; the shared body mesh is untouched.
  - **Head:** 30 themed head pieces on the Head bone. For example:
    - Runescribe: beret with quill; Polaris: mini horseshoe magnet; Ringmaster: top hat; Moorbound: captain's cap;
    - Arbiter: judge cap; Bulwark: tower crown; Kilnborn: clay bowl hat; Flintshade: obsidian crown; Helix: spiral horn.
    - Already had headwear and were left as they were: Wayfinder, Skyweaver, Nacre, Farwatch, Deepwake, Puppeteer.
    - Zephyr's hair fan was made bigger.
  - **Face:** separate pieces on the Head and Ear bones, so the anatomy stays intact:
    - coloured irises and pupils, themed brows;
    - eye liner and frames, tear marks;
    - cheek and forehead pixel marks;
    - earrings (hoop, stud or cuff), small tusks;
    - and per goblin: scar, freckles, monocle, moustache or mouth stitches.
  - Higher rarity gets more face elements.
- Where:
  - `Goblin_Concepts/goblin_new_looks.py` (`SKIN`, `HEADS`, `FACES`, `apply`), called from `goblin_new_kit.finish()`.
  - Files `Goblin_Library/06…10_new_*.blend` rebuilt. The previous versions are in `Goblin_Library/backup_before_looks/`.
  - `export_v4_specs.py` skips the looks module; the hand-off was re-run for 78 goblins.
- Test: background renders of all 36 (face close-ups, front and side). The build checks are clean (0 loose, rig OK, no head clipping). The fixes found:
  - Arbiter's shoulder plate moved out by one vertex's worth.
  - Helix's forehead spiral mark reshaped (it read as a "5").
- Still open: the user's review of the colours and faces.
- **Update, same day:**
  - The user found the eyes too much ("die ogen hoeven niet zo overdreven"). The big iris block with pupil and the liner wing are gone. The dark eye with its shine is the main read again, with only a small colour accent low in the eye and a thinner lower liner.
  - The user's still-open old Blender windows had saved 06, 08 and 09 back to the old version (Blender moved my version to `.blend1`). A comparison showed no manual edits by the user, so all five files were rebuilt and installed again. The hand-off was re-run.
  - **Lesson:** before writing a library file from the background, the user's Blender windows of that file must be closed.

## 2026-10-10 — Goblin library split into files of about 9 goblins (Claude)
- What: the user found Blender lagging hard with everything in one file (12,424 objects) and asked for about 9 goblins per file.
  - `Goblin designs.blend` is now only the **base library**: the original designs, Basic, Leaf, Preview lights and the Actions. About 2,400 objects.
  - The 11 row files are in `Goblin_Library/`: see the table in section 1 → Blender. Each has 350–1,850 objects.
- Where:
  - `Goblin_Concepts/goblin_files.py`: the file table plus `check_file` and `ensure_collection`.
  - `Goblin_Concepts/split_library.py`: makes the split.
  - Hooks in `redesign_v4.py` `Design()`.
  - The merge mode in `export_v4_specs.py`.
  - `AGENTS.md` (library rule) and section 1 are updated.
- How (safe):
  - The first attempt ran `bpy.data.libraries.write` with a Scene inside the user's open Blender, and **Blender crashed**. Nothing was lost: the file had been saved and copied to `Goblin designs_backup_before_split.blend` just before.
  - The split was then done in a background `blender.exe -b --factory-startup` process: open the full backup, delete the other rows, purge, save as.
- Test:
  - Every split file was compared with the full backup: identical object lists per goblin, bones, armature modifiers, `v4_*` props and Actions (7, fake user). **0 differences.**
  - In a background test:
    - Outrider rebuilt inside `06_new_Astral_Cosmic` with identical counts.
    - The same script was refused in `10_new_Beyond` (wrong file).
    - Knight v4 rebuilt inside `03_v4_batch3_Rare_Epic`, with `Knight_Goblin` appended automatically from the base library.
  - (These test builds were not saved.)
- Still open: nothing for the split. Work in the file that holds the goblin; run `export_v4_specs.py` in each file you change.

## 2026-10-10 — 36 new goblins finished: final checks (Claude)
- What: the autonomous task (v4 for the existing goblins, then the 36 new goblins) is done.
  - Started 04:35, finished around 07:10, well inside the 8-hour budget.
  - Section 1 now has a "36 NEW goblins" overview.
- Final checks on all 36 new goblins:
  - **Bone hierarchy:** no orphan bones; every accessory bone chains to an anatomy bone; no unused bones.
  - **Shared animation:** all rigs were put on `Goblin_Walk` (frame 6). Skirts, capes, back constructions and props follow the right bones.
  - **Rebuild:** all 9 rows were rebuilt with the final kit. This also brought Astral up to date with the boot fix and back panels. The rebuild restored the held poses (held goblins: no Action; the others: `Goblin_Idle`).
  - **Gameplay distance:** looked at from behind and from the front.
    - Nullborn's frames were made thicker, with a black square against the white mantle.
    - Unwritten's book was raised so it shows above the head.
- Where: `Goblin designs.blend` (saved), `Goblin_Concepts/goblin_new_*.py`, `physics_parts.json` / `V4_GOBLIN_SPECS.md` (78 goblins).
- Still open:
  - The user's review of the 36 new goblins.
  - Roblox export (merge per bone + colour); holding the arm, leg and head poses in the pet animator; secondary motion and physics; the effects at the `FX_*` bones.
  - Puppeteer's strings should become Beams between `FX_StringTop_k` and `FX_StringEnd_k`.
  - Odds and GameText names for the new rarities.

## 2026-10-10 — NEW goblins, Beyond: Veilkeeper, Nullborn, Liminal, Unwritten (Claude)
- What: the last four new goblins in `Goblin designs.blend`, row `Goblin_New_Beyond` (y = −53.4), script `Goblin_Concepts/goblin_new_beyond.py`. **All 36 new goblins are now built.**
- **Veilkeeper** (114 pieces):
  - **a big connected curtain construction on the back** (`CurtainFrame`):
    - a gold rod on two posts above the head, with 10 hanging rings and finials, and a pearl valance swag with gold trim;
    - two heavy pearl curtain halves of pleated folds in 3 bone segments each (`Curtain_L1..3`, `Curtain_R1..3`), with gold hems;
    - gold tasselled tie-backs (`TieBack_L/R`);
    - a dark inner panel with specks between the curtains (the "impossible space").
  - The clothing repeats the curtain shapes: pleated pearl drapes over a dark robe, a gold chest swag, a waist swag, tassels (`RobeTassel_0/1`), scalloped shoulder valances, a 3-segment drape skirt.
  - Floats **half-sitting** (legs held), hover 0.16. FX: `FX_Beyond`.
- **Nullborn** (72), about negative space (no eye beam):
  - **3 hollow back frames** as thick open outlines, each on its own bone: a large tilted black square (`HollowSquare`), a white ring above the shoulder (`HollowCircle`), a grey inverted triangle (`HollowTriangle`).
  - **A wide white outer mantle** of 5×3 bone chains (`Mantle_00..42`) over a dark suit with dark gloves, a dark belt and boots.
  - **Cut-out shoulder panels** with square holes, a hollow square clasp, a hollow ring buckle.
  - **Stands almost still with the left arm out to the side** (held). FX: `FX_Void0..2` (background colours vanish inside the frames).
- **Liminal** (64):
  - **a tall asymmetric threshold on the back** (`Threshold`): a slim portal frame tilted 6° off to the right, an overshooting lintel, a dark inner panel, gold marks.
  - **5 shifted step panels** climbing diagonally across the back (`Step_0..4`), with gold edges and gold links.
  - Thin overlapping pale clothing layers with hem lines and uneven 2-layer skirts (`LayerA_*`, `LayerB_*`), a thin gold sash.
  - **Bare feet** (no shoes, gold anklets only). **Floats with the left foot higher** (legs held). FX: `FX_Steps`.
- **Unwritten** (81):
  - **a big open blank book on the back** (`Book`):
    - a spine with gold bands, two covers with black corner caps on hinge bones (`Cover_L/R`), thick blank page blocks;
    - 3 loose pages lifting off (`Page_0..2`);
    - 2 ribbon bookmarks (`Ribbon_0/1`).
  - White clothing panels with loose black sketch-stroke edges.
  - **Long blank bands** hanging from the belt (3 bands × 3 segments, `Band_00..22`).
  - Drifts slowly sideways (tilt y −6°, hover 0.12). FX: `FX_Outlines`.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved. Exporter re-run (78 goblins).

## 2026-10-10 — NEW goblins, Absolute: Bulwark, Arbiter, Caliber, Cipher (Claude)
- What: four new Absolute goblins in `Goblin designs.blend`, row `Goblin_New_Absolute` (y = −50.4), script `Goblin_Concepts/goblin_new_absolute.py`.
- **Bulwark** (66 pieces):
  - **a fortress back plate** (`Fortress`), not knight armour:
    - white stone courses, a stepped plinth, a crenellated wall top;
    - two tall crenellated turrets beside and above the head, arrow slits, gold bands;
    - a black arched gate with a gold **portcullis** (`Portcullis`) and **2 sliding wall panels** (`SlideWall_L/R`).
  - Heavy white stepped clothing plates, tower-shaped shoulder blocks with merlons and slits, stone arm, thigh and shin courses.
  - Small black gate details on the chest and belt.
  - **Stands wide**, almost still (legs held). FX: `FX_LightWalls`.
- **Arbiter** (93):
  - **4 stacked judgement plates fanned on the back** (`JudgePlate_0..3` chain): black with gold borders and engraved lines; the top plate carries a scales sign.
  - **A heavy open collar** standing behind the head, below the ears, with gold rims and front edges.
  - Wide black-gold ceremonial clothing: broad layered shoulder plates with gold edges, an ivory stole, a long 3-segment robe, a gold sash.
  - **A big seal-like hand plate held in front of the body** on a hinge bone (`HandPlate`; held arm R (−0.05, −0.35, −0.94) / (0.5, −0.82, 0.28)).
  - Floats at 0.12. FX: `FX_Judgement`.
- **Caliber** (88):
  - **a big symmetric measuring instrument on the back** (`Instrument`): giant dividers with a knurled hinge drum and an adjustment knob above the head, two measuring arms with scales and cyan knobs (`MeasureArm_L/R`), and a graduated protractor arc.
  - **Slide rails with sliders on the shoulders** (`Slider_L/R`).
  - A sleek white technical robe with scale ticks, small adjustment knobs, cyan accents and a belt dial.
  - **Stands symmetric with both arms slightly out** (both held). FX: `FX_MeasureLines`.
- **Cipher** (158):
  - **a mechanical code disc on the back** (`CodeDisc`):
    - **4 concentric rings of different widths**, each on its own bone (`Ring_0..3`), with raised gold/cyan glyphs, dividers and brass edges;
    - **each ring has its own brass clamp lock** with a keyhole (`Lock_0..3`);
    - a hub with a key slot, a brass rim.
  - Black-metal (gunmetal) clothing with inlaid glyphs, lames, bracers with glyphs, shin plates.
  - **Hovers low (0.06) with the head bowed 10°** (held). FX: `FX_Combination`.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved. Exporter re-run (74 goblins).
- Still open: the 4 Beyond goblins (Veilkeeper, Nullborn, Liminal, Unwritten).

## 2026-10-10 — NEW goblins, Omniversal: Riftwarden, Puppeteer, Manyfold, Ringmaster (Claude)
- What: four new Omniversal goblins in `Goblin designs.blend`, row `Goblin_New_Omniversal` (y = −47.4), script `Goblin_Concepts/goblin_new_omniversal.py`.
- **Riftwarden** (89 pieces):
  - **a big arched window on the back** (`Window`), not a door and not a star ring:
    - a dark metal frame with gold trims, a keystone, a sill, mullions and a transom;
    - 9 coloured inner panes on their own bones `Pane_0..8`, ready for the changing landscapes.
  - **Louvred shutters hinged on both shoulders** (`Shutter_L/R`).
  - A long midnight coat with arch-shaped gold clasps, arched pockets, arch cuffs and an arch buckle.
  - Floats at 0.12. FX: `FX_Rift`.
- **Puppeteer** (94):
  - **a cross-shaped marionette control frame above the back** on a pole (`ControlCross`): wooden bars, a front arm, brass caps, string eyes, a grip.
  - Strings hang down to loose costume accessories, each on its own bone:
    - shoulder pompoms (`ShoulderPom_L/R`);
    - cuff ruffles (`CuffRuffle_L/R`);
    - a bell on top of the head (`HeadBell`);
    - a back bow (`BackBow`).
  - The strings are modelled on the accessory bones, with **FX attachments at both ends** (`FX_StringTop_k` / `FX_StringEnd_k`). In Roblox, swap them for Beams so they stay attached while the arms move.
  - A harlequin doublet with diamonds, a ruff, slashed puffed sleeves, torso sides, a heart medal, theatre masks on the belt (`Mask_0/1`), striped stockings, pointed shoes with bells.
  - Hangs tilted 6° at 0.12 with the toes down (legs held).
- **Manyfold** (83):
  - **a big quilted patchwork mantle**: 4×3 patches on bone chains `Mantle_00..32` in one calm palette (mustard, teal, rust, cream, plum) with stripes, dots and checks and thick seams.
  - Front patch panels with big wooden buttons and hanging toggle closures (`Toggle_L/R`), patchwork sleeves.
  - **An asymmetric collar**: a high folded left side and a low right side.
  - Walks with a playful sideways lean (tilt y 7°, head held −8°). FX: `FX_PatchImages`.
- **Ringmaster** (93):
  - **a carousel on the back** (`Carousel`) on a braced arm:
    - a red/white platform with a gold rim, a centre pole;
    - 6 twisted gold poles with carved horses on bob bones (`Horse_0..5`);
    - a striped scalloped canopy with bulbs and a waving flag (`Flag`).
  - A red show tailcoat with wide gold-edged lapels, a bow tie, gold epaulettes with fringe, lace cuffs, long tails.
  - White breeches, black riding boots. Stands proudly (tilt −4°). FX: `FX_Miniatures`.
- Checks: 0 loose, rig OK, no head clipping. Saved. Exporter re-run.
- Still open: 8 new goblins (Absolute, Beyond).

## 2026-10-10 — NEW goblins, Infinite: Helix, Labyrinth, Flowbound, Recursor (Claude)
- What: four new Infinite goblins in `Goblin designs.blend`, row `Goblin_New_Infinite` (y = −44.4), script `Goblin_Concepts/goblin_new_infinite.py`.
- **Helix** (84 pieces):
  - **a big metal spiral with real depth on the back**: a chunky conical spiral band (2.25 turns, radius 0.08→0.68, moving 0.24 backward);
  - it has a gold front edge, a dark back edge and cyan light studs, split into 4 segment bones `Spiral_0..3`;
  - it sits on a compact hub mount (`SpiralHub`) with bolts and 4 spokes.
  - An indigo coat with twisted two-tone cord edges, curved steel and gold shoulder frames, gold spiral clasps and a belt spiral.
  - Floats at 0.12 and turns slowly. FX: `FX_SpiralCenter`, `FX_SpiralEdge`.
- **Labyrinth** (66):
  - **a thick square maze panel on the back, turned to a diamond** (`Maze`):
    - a generated 6×6 perfect maze with readable corridors;
    - floor tiles on 2 levels, dark walls of 2 heights, a bronze frame, a goal stud;
    - 3 sliding wall panels on bones `SlidePanel_0..2`.
  - A teal tunic with a raised corridor pattern, angular belt compartments with lids, stone shoulder blocks, bronze bracers.
  - **Head held tilted 9°** (`rig["v4_legs"]` = held bone rotations, now also used for the head). FX: `FX_MazeLight`.
- **Flowbound** (77):
  - **one continuous ribbon from a back frame** (`RibbonFrame`, ring guides):
    - an infinity loop behind the shoulders in 8 colour sections with contrasting hems (`Ribbon_0..7` chain);
    - two tails over the shoulders (through shoulder clasps) down the front with silver tips (`Tail_L1/2`, `Tail_R1/2`).
  - A white suit with a violet wrap edge.
  - **Hangs relaxed, leaning back 12° with soft knees** (legs held); hover 0.2. FX: `FX_Flow`.
- **Recursor** (90):
  - **three ever-smaller rectangular frames on the back**:
    - each frame is turned a bit further and sits deeper (`Frame_0..2` chain);
    - contrasting navy / orange inner faces, gold corner struts;
    - the innermost frame holds a dark window with nested squares.
  - A navy coat with repeated frame details: frame buttons, a nested-frame emblem, frame cuffs, a nested buckle.
  - **The right hand is stretched out** (held R (−0.14, −0.55, −0.83) / (−0.06, −1, 0.05)) with a tiny floating frame in front of it (`MiniFrame`, MAGIC).
  - FX: `FX_Recursion`.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved. Exporter re-run.
- Still open: 12 new goblins (Omniversal → Beyond).

## 2026-10-10 — NEW goblins, Eternal rarity: Moorbound, Tollkeeper, Signet, Vigil (Claude)
- What: four new goblins of the Eternal **rarity** in `Goblin designs.blend`, row `Goblin_New_Eternal` (y = −41.4), script `Goblin_Concepts/goblin_new_eternal.py`.
  - These are separate from the existing Divine goblin "Eternal", which stays.
- **Moorbound** (107 pieces):
  - **a big weathered anchor on the back** (`Anchor`):
    - a ring above the head and a wooden stock with iron bands and ball ends beside the head;
    - curved arms with flukes;
    - rust patches, barnacles and a kelp strand (`Kelp`).
  - **Two heavy chains** from the anchor over the shoulders (outside the head) down to riveted iron clasps on the belt (`Chain_L1/L2`, `Chain_R1/R2`).
  - A long navy double-breasted coat:
    - brass buttons, wide riveted iron clasps, a turned-down collar, epaulettes, cuffs;
    - 6-part, 3-segment coat ends with brass hem lines.
  - Sea boots. **Stands wide** (legs held apart). FX: `FX_WaterRing`.
- **Tollkeeper** (124):
  - **a big bronze bell hanging in a wooden yoke** on two banded posts from a back plate:
    - the bell has its own hinge bone (`Bell`) and a clapper (`Clapper`);
    - crown loops, a boss, raised bands, an inscription ring.
  - Not a clock: no hands, no time mechanism.
  - A dark-red ceremonial mantle: a 3×2 shoulder cape (`Cape_00..21`), a tabard with a pixel-art bell, a stole with bronze edges.
  - **Small bells** on the stole and at the hem (`SmallBell_0..3`, `HemBell`).
  - An ivory under-robe with a 3-segment skirt. Floats upright at 0.14. FX: `FX_BellStroke`.
- **Signet** (95):
  - **a big sealing-wax seal on the back** (`Seal`):
    - a dripped irregular rim with drips, a pressed rim ring, a ring of letter marks;
    - an embossed crown emblem;
    - two forked ribbon tails with gold aglets (`SealRibbon_0/1`).
  - Layered official clothing: an ivory robe with gold buttons, a dark-red surcoat with gold borders, a 2-layer collar, shoulder ribbons with aglets (`Ribbon_L/R`), a chest seal.
  - Heavy seal stamps on the belt (`BeltStamp_0/1`).
  - **A big seal stamp held in the right fist** (held arm R (−0.06, −0.14, −1) / (0, −1, −0.1), bone `HandStamp`).
  - Stands straight. FX: `FX_GroundSeal`.
- **Vigil** (127):
  - **a branched candelabra on the upper back** (`Candelabra`):
    - a dark metal stem with knots;
    - 5 S-curved arms with curls and cups (`Cup_0..4`);
    - ivory candles with wax drips;
    - small cool blue-white flames on flicker bones (`Flame_0..4`);
    - teardrop wax pendants (`Pendant_0/1`).
  - A long charcoal robe with ivory trims and torso side panels, a draped cowl, a cord belt with wax-drop pendants (`BeltDrop_0/1`).
  - **Walks slowly with the hands folded in front** (both arms held). FX: `FX_Flame0..4`.
- Kit: `torso_sides()` adds side panels for poses where the arms reveal the torso flanks.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved. Exporter re-run.
- Still open: 16 new goblins (Infinite → Beyond).

## 2026-10-10 — NEW goblins, Primordial: Resinborn, Ivory, Kilnborn, Flintshade (Claude)
- What: four new Primordial goblins in `Goblin designs.blend`, row `Goblin_New_Primordial` (y = −38.4), script `Goblin_Concepts/goblin_new_primordial.py`.
- **Resinborn** (102 pieces):
  - **a big tilted honey-amber plate on the back** (bone `Amber`):
    - an irregular polished slab with a light rim and a deep core, bubbles;
    - dark insects inside: a beetle, a wasp with wings, gnats;
    - held by 2 wide stitched leather straps that wrap round the edges, with bronze buckles and loose strap ends (`StrapEnd_0/1`);
    - 3 resin drops dripping from the lower edge (`ResinDrop_0..2`, bob).
  - A stitched leather vest with fur trims, an amber bead necklace with a drop, forearm wraps, a 6-part strip skirt, hide boots.
  - Stands firm, leaning 5° forward with soft knees (legs held). FX: `FX_AmberGlow`.
- **Ivory** (100):
  - **two big curved fossil tusks on a back harness** (`TuskHarness`, `Tusk_L/R`): growth-ring bands, cracks, leather and bone bindings, sockets;
  - the tusks curve out, up and forward, well clear of the head and ears.
  - A weathered patched leather tunic, a frayed 6-part hem with fringe, a tooth necklace, bone buckles and toggles.
  - **Uneven shoulders**: a big layered left pauldron with a cracked bone plate and fur, a small right pad.
  - Hanging bone charms (`BoneCharm_0/1`) and hide flaps (`HideFlap_0/1`).
  - Proud chest (tilt −4°). FX: `FX_DustStep_L/R`.
- **Kilnborn** (90):
  - **a big fired clay pot on the back**:
    - slightly tilted, resting on a padded back pad;
    - pressed dot and stripe patterns, cream slip bands, handles;
    - a rope cradle and sling ropes;
    - a lid tied with rope (`PotLid`), rope ends (`RopeEnd_0/1`), pot bone `ClayPot`.
  - Terracotta chest, shoulder and shin plates with pressed patterns, tied with rope; a rope belt.
  - **3 small sealed pots** with cloth caps, ties and wax seals hanging from the belt (`HangPot_0..2`).
  - Slightly bent knees (legs held). FX: `FX_ClayHand_L/R`, `FX_PotSteam`.
- **Flintshade** (98):
  - **black obsidian knife plates fanning from both shoulders** (broad faces to the front, pale knapped edges);
  - **a crest of 7 tall obsidian blades on the upper back** (`BladeCrest`);
  - a coarse woven tunic with weave bands, an obsidian pendant;
  - **a woven mantle in 3×3 bone chains** (`Mantle_00..22`) with uneven obsidian edge points;
  - 4 magical obsidian shards at the shoulders (`Shard_0..3`, MAGIC);
  - no lava, no fire. Hovers 0.06. FX: `FX_EdgeGlint`.
- Checks: 0 loose, rig OK, no head clipping. Saved. Exporter re-run.
- Still open: 20 new goblins (Eternal → Beyond).

## 2026-10-10 — NEW goblins, Transcendent: Resonant, Silkenveil, Ambrosia, Zephyr (Claude)
- What: four new Transcendent goblins in `Goblin designs.blend`, row `Goblin_New_Transcendent` (y = −35.4), script `Goblin_Concepts/goblin_new_transcendent.py`.
- **Resonant** (111 pieces):
  - **a big golden concert harp on a sturdy back frame**, turned 25° so it reads from the front and the side. It has:
    - a pillar with rings, a crown capital and beads;
    - a curved neck with a scroll and tuning pins;
    - a tapering soundbox with a light soundboard and 3 sound holes;
    - 17 strings (red C and blue F strings) in 3 string groups `Strings_A/B/C`;
    - a base, 3 struts with collars, and a frame plate with straps.
  - Ivory wrap robe with a gold edge, a sash with a sky line, a lyre chest ornament with strings, tuning pegs on gold shoulder plates.
  - Bracelets with tuning-fork hand accents (`Accent_L/R`).
  - The robe has back and side panels plus lap and shin panels that follow the legs.
  - **Floats sitting**: thighs level, shins forward; legs held in `rig["v4_legs"]`; hover 0.16, tilt −3.
  - FX: `FX_Strings`, `FX_SoundWave`.
- **Silkenveil** (194):
  - **a wide, partly furled silk parasol**:
    - the shaft comes from a back holster over the left shoulder;
    - 15 open rose/cream panels with a plum embroidered border, flowers and dots;
    - ribs, stretchers and a runner;
    - a furled bundle of 3 ribs with pleats and a gold ribbon (`ParasolFold`);
    - 3 big tassels (`Tassel_0..2`), main bone `Parasol`.
  - Layered robe: a plum outer robe, a cream inner collar with gold edges, a wide rose obi with gold and teal bands and embroidery, a knot, 2 sash ends with big tassels (`SashEnd_0/1`).
  - Hanging kimono sleeve ends (`SleeveEnd_L/R`).
  - A long 6-part, 3-segment skirt with teal and gold hems, a cream under layer, a flower hairpin.
  - Walks stately. FX: `FX_Sheen`.
- **Ambrosia** (121):
  - **an ornate faceted perfume flask on the back**:
    - half-full pink perfume under lilac glass, a gold neck collar, a tall crystal stopper with gems that rises above the head;
    - a gold filigree cage, a cradle, a medallion with a flower;
    - **a gold atomizer pump** (`Atomizer`) with a tube to a hanging squeeze bulb with a gold net and tassel (`AtomizerBulb`).
  - Glass reservoirs (amber / pink) in gold bands on the shoulder sides (`Reservoir_L/R`).
  - Draped double gold chains to a chest brooch (`Chain_L/R`).
  - A rich purple coat: embroidered vine bands, gold buttons, a lace jabot with a gem pin, lace cuffs, side panels, long tails.
  - **Left arm slightly raised** (held pose L (0.42, −0.3, −0.86) / (0.3, −0.85, 0.42)).
  - FX: `FX_ScentCloud`, `FX_ScentHand`, `FX_Stopper`.
- **Zephyr** (114):
  - **a huge open brisé folding fan**:
    - 15 pointed red lacquer slats, each on its own bone `Fan_00..14` under `FanPivot`;
    - the slats have gold edge lines and gold disc motifs;
    - black guard sticks with gold inlay;
    - a gold pivot boss connected to a black lacquer back mount.
  - Compact red fitted jacket: a black gold-edged wrap, frog clasps, black shoulder pieces.
  - A wide layered waistband (black, red, gold) with a cord knot and cord ends (`CordEnds`).
  - Black bracers with gold rims, fitted trousers, a short skirt, a fan-shaped hair ornament.
  - **Floats with the feet together, toes down** (legs held); hover 0.16. FX: `FX_FanEdge`, `FX_AirWave`.
- Kit: `coat_skirt(..., trim2=)` adds a second hem line.
- Lesson: the blocky thighs are about as deep as they are long, so a sitting pose only reads with the thighs level AND the shins forward.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved. Exporter re-run.
- Still open: 24 new goblins (Primordial → Beyond).

## 2026-10-10 — NEW goblins, Cosmic: Polaris, Farwatch, Deepwake, Outrider (Claude)
- What: four new Cosmic goblins in `Goblin designs.blend`, row `Goblin_New_Cosmic` (y = −32.4), script `Goblin_Concepts/goblin_new_cosmic.py`.
- **Polaris** (182 pieces):
  - **a big red (N) / blue (S) horseshoe magnet on the back** whose poles rise above the head;
  - on the magnet: silver pole caps with N/S letters, two copper coil windings per pole with iron collars, iron arm clamps, a mount, a silver split band;
  - **6 magical iron fragments** (nuts, gear, bolt, shards) float between the poles (`Fragment_0..5`, MAGIC bones);
  - red/blue pauldron lames with shoulder caps and N/S letters, copper forearm coils between iron bracers;
  - a dark work suit with red/blue trims and a zip;
  - two field gauges whose needles have their own bones (`Needle_N`, `Needle_S`), a polarity toggle;
  - an iron belt with a horseshoe-magnet buckle, a washer ring (`Washers`) and a nut chain (`NutChain`);
  - iron-toe work boots;
  - **hovers 0.07 with bent knees**: a leg pose stored in `rig["v4_legs"]`, which Roblox must hold.
  - FX: `FX_Fragments`, `FX_PoleN`, `FX_PoleS`.
- **Farwatch** (136):
  - **a slanted radar dish above the right shoulder** on a mast from a back plate (hazard stripes, collars, cable);
  - the dish: a turntable (`DishBase`, yaw), the dish itself (`Dish`, tilt), 8 ribs, an orange rim, a feed horn on 3 struts;
  - **a wide raised visor band** on a headband with side hinges and pods (`Visor`, tilt);
  - a long grey observation coat:
    - lapels, orange piping, reflective strips, shoulder tabs, cuffs;
    - a 6-part, 3-bone skirt **with side panels** (new `coat_skirt` parts `SL`/`SR`), pockets, a back vent;
  - measuring panels with screens and LEDs, a wrist read-out, a data tablet on the belt (`Tablet`);
  - stands firmly. FX: `FX_Visor`, `FX_ScanLine`.
- **Deepwake** (123):
  - a white padded space suit with petrol stripes, copper shoulder/elbow/wrist rings and bellows, knee rings, big gloves with copper cuffs, space boots, a chest unit with lights and dials, hose ports;
  - **an open helmet frame round the free head**:
    - a copper collar ring;
    - a white back shell with petrol and copper bands, ribs, bolts and a vented hatch;
    - side struts behind the ears, a top ring with two lamps, a ridge, an antenna (`HelmetFrame`);
  - **a back module with a large observation lens** (`Lens`) and hinged instrument flaps (`Flap_L/R`);
  - 2 ribbed pressure hoses over the shoulders into the chest ports (`Hose_L1/2`, `Hose_R1/2`);
  - drifts at hover 0.13, tilt [−4, 3, 0]. FX: `FX_LensBeam`, `FX_HelmetLamps`.
- **Outrider** (102):
  - **a satellite bus on the back**: gold foil with crinkles, a frame, a star tracker, a small dish (`SatDish`), 2 whip antennas (`Antenna_0/1`);
  - **two folding solar-panel arms**: a boom (`Boom_L/R`) and 3 panels each (`Panel_L1..3`, `Panel_R1..3`, one hinge bone per panel), with cells on both faces and signal lights at the tips;
  - a white quilted MLI suit with gold-foil patches and bands, white gloves and boots, belt instrument pods;
  - hovers 0.1. FX: `FX_Signal_L/R`, `FX_Beacon`.
- Fixes found while building:
  - `panel(..., side="back")` was missing on every back panel, so the back panel sat on the FRONT (hiding front details) and the back was bare. This also affected Astral (Wayfinder, Skyweaver, Nacre, Runescribe); fixed and rebuilt.
  - New kit helper `face_y()`: details on garments are placed on the panel's outer face, not on the bare body.
  - Boots now have a smaller bevel (the leg poked through the boot top).
  - `bend_knees()` now has the right signs and stores the leg pose.
- `export_v4_specs.py` now also writes `leg_pose`. It was re-run: `physics_parts.json` and `V4_GOBLIN_SPECS.md` cover 50 goblins.
- Checks: 0 loose, rig OK, no head clipping. Saved.
- Still open: 28 new goblins (Transcendent → Beyond), continuing in the same way (`goblin_new_<rarity>.py`).

## 2026-10-10 — NEW goblins, Astral: Wayfinder, Skyweaver, Nacre, Runescribe (Claude)
- How (applies to all 36 new goblins):
  - Each one is a copy of the approved Basic/Common goblin (`Basic_Goblin` = `Common_Goblin/common_goblin.blend`): the same body mesh, the 19-bone rig and the shared Actions.
  - Each sits in its own collection `<Name>_Goblin` with `<Name>_Rig`, `<Name>_Body` and an upright display parent `<Name>_Goblin_DisplayOffset`.
  - All clothing and accessories are separate pieces, skinned to one bone each. No pickaxe mesh.
  - Kit: `Goblin_Concepts/goblin_new_kit.py` (`new_design`, garment `panel`, `sleeve`, `coat_skirt` with bone chains, `boots`, `finish`).
  - Rows: `Goblin_New_Astral` (y = −29.4), and later `Goblin_New_<Rarity>`. Backup before the new goblins: `Goblin designs_backup_before_new36.blend`.
- **Wayfinder** (101 pieces):
  - a dark-blue layered traveller's coat (open fronts, a lighter capelet over the shoulders, lapels, brass piping, buttons, pocket flaps, a 4-panel 2-bone skirt with brass trim, brass cuffs);
  - brass direction arrows; a navigator's cap with a compass-rose badge and a spinning brass pointer (CapPointer);
  - belt instruments, each with its own swing bone: a telescoping spyglass, a sextant, dividers, a pocket compass on a chain;
  - **a big mechanical octagonal compass on the upper back** in a gimbal yoke on a back plate: an octagon frame with screws, a dial with a compass rose, a rotating inner ring with ticks and N/E/S/W (**CompassRing**), and a red/white needle with a pivot cap and gem (**CompassNeedle**);
  - hovers 0.08.
  - FX: `FX_Needle`, `FX_CourseLines` (glowing course lines under him).
- **Skyweaver** (72):
  - **a big asymmetric stitched kite on a back frame**: 4 colour panels, binding, seam stitches, a spine and bowed cross spar, a bridle, a pivot (bones KiteFrame → Kite, tiltable);
  - a 5-bone bow-tied ribbon tail and 2 side streamers;
  - a quilted sky-blue jacket with wind-arrow sleeves, a crossed harness with a ring and D-rings, an aviator cap with goggles;
  - **hangs tilted 8° at 0.18**. FX `FX_Wind`.
- **Nacre** (59):
  - **an open oyster on the back**: a ribbed base shell and a hinged lid with strong mother-of-pearl colour bands inside, plus the pearl (bones ShellBase → ShellLid, Pearl);
  - an ivory coat with gold scallop clasps, a scalloped collar with pearls, scalloped edges, a long skirt with a pearl hem, pearl cuffs;
  - 2 pearl strands (bones) with a shell pendant, a pearl circlet, a shell comb;
  - **the right hand rests back near the pearl** (held arm). FX `FX_Pearl` (bubbles + light pulses).
- **Runescribe** (99):
  - **a wide fan of 9 quills in a leather rack on the back** (feather quills with brass nibs);
  - a layered writer's coat: burgundy waistcoat with buttons, cravat ruffles, lapels, turned white cuffs with ink stains, a 4-panel 3-bone long skirt;
  - 3 ink pots (black / blue / red) in a holder, a blotter roll, reading spectacles;
  - **a big quill held in the right fist** (held arm, bone HandQuill);
  - leans forward 5°. FX `FX_RuneWrite` (runes written in the air), `FX_QuillFan`.
- Checks: 0 loose, rig OK, no (posed) head clipping. Saved.

## 2026-10-10 — Celestial v4: Nebula, GalaxyWalker, Supernova, Eclipse, Starlight, VoidWatcher (Claude)
- Each was built and checked one by one in `Goblin_Concepts/redesign_v4_batch7.py`, row `Goblin_Redesign_v4_B7` (y = −25.8). Each has a different hero element:
  - **Nebula** (79 new pieces):
    - a puffy cosmic cloud mantle round the shoulders (purple / pink / blue, star specks), below the head and outside its footprint;
    - **a big nebula cloud collar rising behind the head**, the silhouette from behind;
    - the cape grows 2 flowing segments per column to the ground (Cape_C0..C4) with a pixel nebula swirl and stars;
    - a silver star circlet (replaces the crown), robe stitches;
    - **6 orbiting star motes** (magical). Hovers 0.1.
  - **GalaxyWalker** (42):
    - **4 detailed planets on a tilted dotted orbit round the head** (magical; Orbit → Planet_0..3): a ringed Saturn, an Earth with a moon, a Mars with a moon, a banded gas giant;
    - galaxy boots with glowing cyan soles, star cuffs, toe caps and star wings;
    - a starry scarf with 2 tails (Scarf bone chains), a pixel spiral galaxy on the chest, planet charms on the belt;
    - walks; star footprints.
  - **Supernova** (53):
    - **an exploding star behind the back** (magical): a white-yellow core, 13 asymmetric spiky rays (white → yellow → orange / magenta tips), 4 broken shock-ring pieces;
    - flare shards on the shoulders, a spiked radiant diadem, a blazing chest star, a gold belt;
    - the old corona and rays are removed. Hovers 0.12.
  - **Eclipse** (57):
    - **a black moon disc with a corona ring + glow, 5 solar prominences, 10 streamers and a diamond-ring flash behind the head** (magical);
    - **a sun / moon split outfit**:
      - a gold sun pauldron with rays and a gem (left), a silver crescent pauldron (right);
      - a split tabard with an eclipse disc;
      - a gold / indigo split cape with a sun and a moon (EclipseCape_C0/1);
      - robe suns and moons, gold / silver bracers, a circlet with an eclipse. Hovers 0.1.
  - **Starlight** (47):
    - **a big star wand in the right fist** (held arm): silver shaft, gold bands, a two-layer 0.17 star with a gem, 2 dangling star charms on ribbons (bones);
    - a star tiara (replaces the crown), star pauldrons, a sky-blue star sash;
    - a big two-layer star with gem studs on the back, a medallion, 2 ribbon chains. Hovers 0.1.
  - **VoidWatcher** (41):
    - **a floating void eye above the head** (magical): a star-speckled sclera, a violet iris and ring, a pupil, mechanical lids with violet rims, 4 orbit shards;
    - a starfield hood with ear openings, violet rims, an eye emblem, a drooping 3-bone tip;
    - void crystal pauldrons, a pixel space rift on the cape, robe starfield, cape bone chains;
    - the highest hover (0.15).
- FX: one `FX_*` bone per effect (motes, planets, steps, nova, eclipse, wand star, void eye, …). Checks: 0 loose (magical pieces excepted), rig OK, no head clipping (the Nebula mantle and Starlight back star were moved for it).
- `physics_parts.json` / `V4_GOBLIN_SPECS.md` regenerated (all 42 existing goblins). Saved.
- **All 42 existing goblins now have a v4 version.** Next: the 36 new goblins (Astral → Beyond).

## 2026-10-10 — Phoenix v4 (Divine) (Claude)
- What (107 new pieces):
  - **flame-feather wings swept back from the upper back.** Every feather is a 3-layer flame shape (red / orange / yellow core), in 3 rows per wing. There is a bone per wing joint, and a red mount with gold studs. The wings stay below the head top and clear of the head and ears; the old voxel flame wings are removed.
  - **5 long flame tail plumes** from the lower back, each a 3-bone chain (Tail_K1..3). Each ends in a flame eye-feather with a yellow and blue eye spot.
  - **A fire crest** of 5 layered flame feathers on a gold band with a ruby. It replaces the crown and has its own bone, Crest.
  - The flame markings on the face are kept.
  - A gold pixel phoenix on the chest, gold talon anklets with rubies, ember eye glints.
- Pose: hovers 0.12, leaning forward 4°. Roblox: wing beats + tail sway.
- FX: `FX_WingFlame_L/R`, `FX_TailTrail`, `FX_Crest`, `FX_Rebirth` (ring of fire + ash at the feet on equip / level-up).
- Divine row done: Seraph, Zeus, Eternal, Phoenix. `physics_parts.json` / `V4_GOBLIN_SPECS.md` regenerated. Saved.
- Next: Celestial (Nebula, GalaxyWalker, Supernova, Eclipse, Starlight, VoidWatcher), then the 36 new goblins.

## 2026-10-10 — Eternal v4 (Divine) (Claude)
- What (58 new pieces):
  - **a floating clockwork ring behind him** (magical; it replaces the old clock ring, face and hands):
    - a gold outer and inner ring, and an open midnight-blue numeral band with pixel Roman numerals XII / III / VI / IX and hour marks;
    - 36 gear teeth, 3 gears on the rim;
    - 4 spokes to a hub with a sapphire;
    - **separate hour and minute hand bones** (HourHand / MinuteHand → ClockRing) that Roblox rotates;
    - the ring is open, so his back and cape stay visible through it;
  - **an hourglass hanging from the left fist by a ring handle** (pendulum bone Hourglass):
    - gold caps, 3 twisted pillars, a glass double bulb;
    - sand above, a falling stream, a pile below;
  - a star-chart cape (constellation pixels, a crescent), cape bone chains;
  - a small clock badge on the crown, Roman-numeral belt plates, starry eye shine.
- Pose: floats 0.1 in front of the clock.
- FX:
  - `FX_ClockRing`: the hands tick, and a golden time ripple every 12 s;
  - `FX_Hourglass`: glowing sand and rising motes.
- Note: the Tollkeeper (new, Eternal rarity) must not use clock hands or time mechanisms; this goblin keeps the clock theme.
- Where: `redesign_v4_batch6.py` (`eternal`), `Goblin_Redesign_v4_B6`. 0 loose (except the clock on purpose), rig OK; saved.

## 2026-10-10 — Zeus v4 (Divine) (Claude)
- What (about 75 new pieces):
  - **a big cloud beard:** 4 rows of white/grey puffy curls from the mouth down over the chest, gold lightning streaks, 2 braids with gold beads. Bones Beard → BeardTip. The old lightning beard is removed;
  - **a golden thunderbolt through the right fist** (held arm, forearm forward): a gold zigzag with a white-hot core, 3 branch bolts, a grip band. It rises up and out beside the head, not across the face. The old held lightning is removed. A fully raised throwing arm was tested but pushes the bolt or arm into the ear, so it was not used;
  - a toga cape (4 × 2 bones Toga_C0/1) with a gold Greek-key hem and a gold pixel eagle; round gold clasps with bolts;
  - **cloud anklets** (6 puffs per foot; he walks on air);
  - bolt gems on the crown, a chest bolt, gold bracers with rims and bolts, a gold belt with a bolt buckle, white storm eye glints.
- Pose: hovers 0.1. FX: `FX_Bolt` (arcs, lightning strike on attack), `FX_CloudFoot_L/R`.
- Kit: the head-clipping checks ignore points on a face plane's extension (a false hit at the head corner).
- Where: `redesign_v4_batch6.py` (`zeus`), `Goblin_Redesign_v4_B6`. 0 loose, rig OK, no (posed) head clipping; saved.

## 2026-10-10 — Seraph v4 (Divine) (Claude)
- What (155 new pieces). The hero element is **two pairs of real feathered angel wings** (they replace the old voxel wings):
  - Main wings on the upper back spread out and up. Their tips stay below the head top, behind and clear of the head and ears (AGENTS.md).
  - Feather rows per wing: 9 primaries with pale-gold tips, 8 secondaries, 9 coverts, 7 lesser coverts. These are the approved feather shape, placed by matrix.
  - Each joint has a bone: Wing_L/R1..3. A smaller lower pair has LowWing_L/R1..2.
  - A wing mount on the back with gold straps over the shoulders.
  - **A floating halo** (magical, Halo bone on Head): a gold ring, an inner glow ring, 8 star points.
  - A gold circlet with a star sapphire and little feather wings at the temples (replaces the crown).
  - A white stole with gold trims, stars and fringe hanging down the front (2-bone chains Stole_L/R2..3).
  - A radiant heart (gold sunburst + glowing gem), light-blue eye shine.
- Pose: hovers 0.12, wings spread. Roblox: slow bob + gentle wing beats.
- FX: `FX_WingTip_L/R` (drifting glowing feathers), `FX_Halo`, `FX_Heart`.
- Kit: `feather_m` (feather placed by matrix, rows collected per colour), `plates` caps are ear-clipped (concave outlines). The `wing()` builder in batch 6.
- Where: new script `Goblin_Concepts/redesign_v4_batch6.py` (Divine), row `Goblin_Redesign_v4_B6` (y = −23.0). 0 loose (except the halo on purpose), rig OK, no head clipping; saved.

## 2026-10-10 — Stormbringer v4 (Mythic) + posed clipping check (Claude)
- What, Stormbringer (52 new pieces). The hero element is a hand prop:
  - **a rune storm hammer through the right fist, resting on the shoulder** (Basic's held arm pose), with the head behind the ear:
    - the head: dark iron with silver caps and faces, a centre band, storm sapphires, blue rune glyphs and bolt marks, 2 crackle bolts;
    - the handle: wood, a wrapped grip, silver collars, a pommel with a loop;
  - **a storm cloud on the left shoulder** (7 two-tone puffs, kept outside the head and under the ear) with 2 lightning bolts hanging down the arm (bone Cloud);
  - a short storm cape: 4 columns × 2 segments (StormCape_C0/1), a zigzag hem, shoulder clasps, a lightning emblem on the back;
  - a bolt badge and storm gems on the crown, a lightning bolt and rune studs on the chest, electric eye glints.
- FX: `FX_HammerArc`, `FX_CloudFlash`, `FX_StaticSteps`.
- Kit:
  - `posed_head_clearance` checks clipping in the DISPLAYED pose (held arms, evaluated meshes);
  - result: Stormbringer, Knight, Pyro, King, Cyborg, Basic are clear;
  - the Chef's cleaver touched the chin with 1 vertex → his arm pose is now a bit more forward and down (rebuilt, clear).
- Mythic row done: Demigod, ShadowStalker, Volcanic, CrystalLord, Stormbringer. `physics_parts.json` / `V4_GOBLIN_SPECS.md` regenerated (32 goblins). Saved.
- Next: Divine (Seraph, Zeus, Eternal, Phoenix), then Celestial (6), then the 36 new goblins.

## 2026-10-10 — CrystalLord v4 (Mythic) (Claude)
- What (58 new pieces). The hero element is headwear:
  - **a crystal spire crown** (replaces the old crown):
    - a silver band with a rim and amethyst gems;
    - 8 hexagonal crystal spires tilting outward, with an ice-white centre spire (0.38 tall);
    - 6 small crystals between them;
  - crystal pauldron clusters on silver collars, angled out under the ears;
  - a royal violet cloak from the shoulders: 4 columns × 2 segments (bones Cloak_C0/C1), a silver hem trim with hanging crystals, shoulder clasps, a silver/amethyst crystal emblem on the back;
  - **3 orbiting crystals** above the crown (magical, Orbit_0..2);
  - crystal shards on the gauntlets, silver rays round the chest crystal, belt amethysts, amethyst eye glints.
- FX: `FX_CrownGlint`, `FX_Orbit_0..2`, `FX_CrystalStep`.
- Where: `redesign_v4_batch5.py` (`crystal_lord`), `Goblin_Redesign_v4_B5`. 0 loose (except the orbit crystals on purpose), rig OK; saved.

## 2026-10-10 — Volcanic v4 (Mythic) (Claude)
- What (about 66 new pieces). The hero element is the shoulders:
  - **two mini-volcano pauldrons** on the back half of each shoulder, behind the ears, so they can be tall (0.24):
    - basalt chunks round the base (none toward the head), a two-tone cone;
    - a crater with a glowing rim, a lava pool and a yellow core;
    - 5 lava drips down the sides, glowing cracks;
    - the old smoking shoulder rocks are removed;
    - bones Volcano_L/R (children of UpperArm_L/R);
  - **a magma heart on the chest**: a basalt socket, a faceted magma gem, lava cracks spreading over the armour;
  - obsidian gauntlets with lava veins and basalt knuckle spikes;
  - a basalt-plate belt with glowing seams;
  - a lava-crack network and 4 basalt spine spikes with glowing bases on the back;
  - fire glints in the eyes.
- FX: `FX_Crater_L/R` (smoke, embers, small lava bursts), `FX_MagmaHeart`, `FX_Footstep_L/R` (lava footprints).
- Where: `redesign_v4_batch5.py` (`volcanic`), `Goblin_Redesign_v4_B5`. 0 loose, rig OK, no head clipping; saved.

## 2026-10-10 — ShadowStalker v4 (Mythic) (Claude)
- What (66 new pieces). The hero element is a wraith cloak silhouette:
  - **a deep hood**:
    - ear holes like Shy's hood, an overhanging brim with ragged tatters, a dark forehead shadow over the upper face;
    - a 3-bone drooping tip;
    - violet eye glow in the eye slits;
  - ragged collar flaps over the chest and a ragged mantle on the upper back;
  - **the cape grows 2 wispy violet tendril segments per column down to the ground** (bone chains Cape_C0..C4; FX smoke at every tip);
  - **shadow claws:** an obsidian gauntlet with 3 curved, violet-edged claws on each hand;
  - **5 floating obsidian shards with violet cores** (magical, Shard_0..4, orbiting);
  - violet rune stitching on the robe, a gem on the belt buckle, 3 hanging talismans with violet runes (bones).
- Pose: **hovers 0.07 and leans forward 6°** (display parent).
- FX: `FX_Wisp_0..4`, `FX_Claw_L/R`, `FX_Shard_0..4`.
- Where: `redesign_v4_batch5.py` (`shadow_stalker`), `Goblin_Redesign_v4_B5`. 0 loose (except the shards on purpose), rig OK; saved.

## 2026-10-10 — Demigod v4 (Mythic) (Claude)
- What (79 new pieces). The hero element is a radiant **sun aureole** behind the head:
  - the aureole: a gold outer and inner ring, 16 long/short rays and sun studs. It is magical and floats on purpose (bone Aureole, a child of Head, listed in `MAGIC`);
  - a gold laurel wreath (leaf pairs round the head, a sun gem in a bezel at the front);
  - a toga drape over the left shoulder across the chest to the right hip, with gold trims, a sun brooch with rays, and a 3-bone drape hanging down the back with a gold hem;
  - **a sun scepter in the left fist** (replaces the old staff / sun scepter / halo): a banded gold shaft, foot, rings, and a sun head (ring, 12 rays, a glowing heart);
  - a sun emblem on the chest; gold arm bands with gems;
  - sun discs and rays on the shoulder plates, greave trims, little gold wings at the ankles;
  - a gold rope cincture with a knot and 2 tasselled cords (bones), golden eye shine.
- Pose: **hovers 0.1 above the ground** (the display parent is raised; `POSES` → `physics_parts.json` "pose"). Roblox adds the slow bob.
- FX: `FX_Aureole`, `FX_SunHeart`, `FX_HoverGlow` (on Root, under the feet).
- Where: new script `Goblin_Concepts/redesign_v4_batch5.py` (Mythic), row `Goblin_Redesign_v4_B5` (y = −20.4). Its `build()` applies hover/tilt poses to the display parent. 0 loose (except the aureole on purpose), rig OK; saved.

## 2026-10-10 — DragonRider v4 (Legendary) + Bandit cape bones (Claude)
- What, DragonRider (129 new pieces). The hero elements are a creature on the shoulder and a helm:
  - **a baby red dragon perched on the left shoulder**, placed so its body sits under the ear, its head in front of it and its wings behind it:
    - body, belly plates, back spikes, 4 legs with claws;
    - an S-neck, a head with snout, jaw, yellow slit eyes, horns, nostrils, fangs;
    - bat wings (arm + finger + membranes), a 2-bone tail with a gold spade;
    - bones: Dragon → DragonNeck → DragonHead, DragonWing_L/R, DragonTail1 → 2 (animation preparation: head turns, wing flaps, tail sway);
  - **a horned dragon helm** (replaces the crown and the 6 small horns):
    - a red cap with scale plates, side plates above the ears;
    - a snout visor with a row of ivory teeth and nostrils, gem dragon eyes in gold bezels;
    - two big swept ivory horns in gold sockets, a 5-spike gold frill with an orange membrane;
  - rider gear: scale rows on the shoulder plates, a riding strap with a gold ring, an ivory war horn with gold bands on the right hip, a rope rein coil on the left hip;
  - a red pixel dragon on the cape, a scalloped hem, ember eye glints;
  - cape bone chains.
- FX: `FX_DragonBreath` (fire puffs and embers from the baby dragon's mouth).
- **Bandit:** the original cape now also has bone chains (Cape_C0..C2); its tatters and patches follow the cape bones. Batch 3 also calls `fix_parents`.
- Where: `redesign_v4_batch4.py` (`dragon_rider`), `redesign_v4_batch3.py` (Bandit). `physics_parts.json` + `V4_GOBLIN_SPECS.md` regenerated (27 goblins). Saved.
- Legendary row done: King, Cyborg, Necromancer, Assassin, DragonRider. Next: Mythic (Demigod, ShadowStalker, Volcanic, CrystalLord, Stormbringer).

## 2026-10-10 — Assassin v4 (Legendary) (Claude)
- What (102 new pieces). The hero element is clothing: a face mask whose scarf tails flow behind it.
  - **A face mask** below the eye slit: a face band, a snout cover, sides and back, folds, and a blood-red top edge.
  - **Two long blood-red tails** flow back from a knot on the back of the head. Each is a 4-bone chain (MaskTail_L1..4 / R1..4) with frayed tips.
  - The hood gets a dark rim and seams.
  - **Hidden wrist blades:** leather bracers with buckled straps and a steel plate, plus a blade mechanism. A bevelled blade runs down along the outside of each fist.
  - A shuriken strap across the chest (3 four-point shuriken), two poison vials in belt loops, and a crescent + dagger emblem on the cape.
  - Red eye glints and cape bone chains.
- FX: `FX_Blade_L/R`, `FX_SmokeBomb`.
- Where: `redesign_v4_batch4.py` (`assassin`), `Goblin_Redesign_v4_B4`. 0 loose, rig OK, no head clipping; saved.

## 2026-10-10 — Necromancer v4 (Legendary) (Claude)
- What (119 new pieces). The hero element is the collar silhouette, plus a hand prop:
  - **a high bone collar:** 9 fanned ribs with knuckles round the back of the head, on a rounded path that stays clear of the head block and the ears. Dark membranes between the ribs, an iron base band with studs;
  - **a crook staff in the left fist** (replaces the old staff): dark wood with iron rings, a wrapped grip, an iron crook with bone spikes, a skull with green glowing sockets at the bend;
  - **a soul lantern hanging from the hook** (pendulum bone): hex cage, glass, a green soul flame;
  - **3 ghost-skull souls with wisp tails** (Soul_0..2). They are magical and float on purpose (AGENTS.md); Roblox orbits them. The old soul orbs are removed.
  - a grimoire with a skull and clasp on the right hip;
  - a belt chain with a tiny skull;
  - glowing green runes on the robe hem and a pixel rune circle on the cape;
  - a torn cape hem, green eye shine;
  - cape bone chains Cape_C0..C2.
- Kit:
  - `MAGIC[key]` + `floating_report_m` skip deliberately floating magical bones; they are stored on the rig (`v4_magic`);
  - `physics_parts.json` marks those bones as `orbit (magical, floats on purpose)`.
- FX: `FX_SoulLantern`, `FX_Soul_0..2`.
- Where: `redesign_v4_batch4.py` (`necromancer`), `Goblin_Redesign_v4_B4`. 0 loose (except the souls on purpose), rig OK, no head clipping; saved.

## 2026-10-10 — Cyborg v4 (Legendary) (Claude)
- What (118 new pieces). The hero elements are an arm cannon and a cyber half mask (not a back object):
  - **a cybernetic half mask on the right of the face:**
    - forehead, cheek, under-eye and side plates, with a socket frame round the ear root;
    - a scanner ring round the red robot eye, glowing cyan circuit traces, bolts, and an antenna with a red LED (bone);
    - the old robot head panels are removed; the red eye and the metal jaw are kept;
  - **an arm cannon on the right forearm** (held arm pose, forearm forward):
    - a gunmetal sleeve with bands, vents and 3 status LEDs;
    - a caged cyan energy cell;
    - a barrel above the fist with cooling fins, a muzzle with a cyan core and glow ring, a sight with a red lens;
    - a trigger grip in the fist and a striped hose loop;
  - the chest power core gets a glowing core, a housing and bolts;
  - a power pack on the back: a hex cyan core window, heat-sink fins, 2 antennas with LEDs, side vents, an LED row and a hazard band, on a harness.
- Bones:
  - Cannon (child of Hand_R), Antenna, PowerPack;
  - FX: `FX_ScanEye`, `FX_CannonMuzzle`, `FX_Core`, `FX_PackVent`.
- Where: `redesign_v4_batch4.py` (`cyborg`), `Goblin_Redesign_v4_B4`. Kit: `Hold` now also gives the posed elbow and wrist. 0 loose, rig OK, no head clipping; saved.

## 2026-10-10 — King v4 (Legendary) (Claude)
- What (110 new pieces, 45 bones). The hero element is the crown and mantle:
  - **a grand jewelled crown:**
    - an ermine roll with black spots;
    - a gold band with rims, 6 set gems in bezels and pearls;
    - 8 fleur-de-lis points with pearl tips, two pearl-studded arches over a red velvet cap, a gold monde and a cross with a ruby;
    - the old crown pieces are removed;
  - a big ermine mantle collar over the shoulders (it replaces the old fur collar), with ruby clasps and a hanging gold chain;
  - **a sapphire scepter held upright in the right fist** (held arm pose): banded rod, knob, collars, a cup with claws round a big sapphire, a crownlet and cross, a ferrule;
  - gem rings on both hands, two medals with ribbons on the left chest;
  - the compact back shields (AGENTS.md) get gold rims and rivets;
  - a pixel-art crown and two fleurs embroidered on the cape;
  - **the mantle continues below the cape to the calves** (a 4th segment per column) with an ermine hem.
- Bones:
  - Crown, Scepter (child of Hand_R), MantleChain, Medals;
  - **cape chains Cape_C0..C3 for all 5 columns** (the original cape segments + trims are re-skinned to them: physics in Roblox);
  - FX: `FX_CrownGleam`, `FX_ScepterGem`.
- Where: `Goblin_Concepts/redesign_v4_batch4.py` (Legendary; `ONLY = ("King",)`), new row `Goblin_Redesign_v4_B4` (y = −17.8). Kit: `cape_bones()`, `fix_parents()`. 0 loose, rig OK, no head clipping; saved.
- Still open: Cyborg, Necromancer, Assassin, DragonRider (next, one by one).

## 2026-10-10 — Goblin v4 batch 3, round 2: far more detail (the user's review) + physics/FX hand-off (Claude)
- What: the user was disappointed by the detail of batch 3, so all 9 were rebuilt with a new detail kit. Pets are mostly seen from behind, so every back got real work. Held props now go THROUGH the fist on a held arm pose (like Basic).
  - **Hoarder** (202 pieces):
    - an open overflowing sack: a gold mound with embedded coins, coin stacks, coins over the rim, a jewelled goblet, a small crown, a ruby scepter and a pearl string draped over the rim;
    - patches sewn into the cloth with X stitches, neck folds, a twisted rope cinch with tasselled ends;
    - a luxury gold watch on the left wrist (diamond bezel, blue dial, markers, hands, crown, lugs);
    - a Cuban-link bracelet with 3 set gems and a coin charm on the right wrist;
    - a key ring with 3 keys on the right of the belt.
  - **Brawler** (126):
    - gloves with a raised knuckle cushion, side piping, a thumb with a tie, a padded cuff with trims, eyelets, criss-cross laces with a bow, and a star logo;
    - armour: rims, rivets, a red-star medallion, panel lines, scratches and dents, a back plate with a red "KO", and layered pauldron lames;
    - a champion belt: a studded strap, an octagonal plate with medallion, laurel, crown and gems, and a "1" back plate;
    - a towel tucked in the back (2 bones).
  - **Mystic** (101):
    - the turban: crossing wraps, gold trim, a coin fringe (3 bones), a sunburst brooch with an emerald, a pearl drop, two plumes, and a long tail down the back (3 bones);
    - an all-seeing-eye sigil and a moon on the vest back;
    - a teal sash with a back knot and two tails;
    - hoop earrings, bangles, a tarot pouch, vest stars;
    - **the crystal ball now hangs from the sash** (chain, 4-prong brass cage, cup, finial, a violet→pale gradient ball with a swirl and highlight).
  - **Knight** (145):
    - **the sword in the right fist** (held arm pose): leather-wrapped grip, gem pommel, curved quillons, ricasso, bevelled blade with fuller;
    - **the heraldic shield on the back**: rim, gold bordure, silver chevron, crown, star, rivets, a guige strap across the chest;
    - an empty scabbard behind the left hip, tabard flaps, mail, gauntlets, pauldron lames, helmet brow band and rivets, a chest crown, knee bosses, a 3-strand plume.
  - **Pyro** (283):
    - a TNT crate on the back: planks, posts, iron brackets with nails, a "TNT" stencil, a hazard band, rope handles;
    - a 7-stick dynamite bundle with labels, caps, rope and curling fuses (one lit, with a spark);
    - a fuse coil, and a detonator with plunger and wires;
    - a stitched bandolier with detailed sticks;
    - **the torch held up in the left fist**: wrapped grip, iron basket, layered flame with 4 flame-tongue bones;
    - detailed goggles, a welding gauntlet, scorch marks, a matchbox pouch.
  - **Bandit** (151):
    - **a drawn dagger in the right fist** and a detailed sheathed one: wire grip, S-quillons, gem pommel, bevelled curved blade;
    - an empty right sheath, 3 throwing knives with ring pommels on a thigh strap;
    - a pointed hood tip (3 bones), hood trims and folds, a dotted neck bandana with back tails, cape clasps with a chain, a torn hem and patches;
    - the loot sack: a sewn "$" patch, a tear with coins;
    - a coin purse, arm wraps, an earring, vest stitching and pockets.
  - **Shaman** (184):
    - **a gnarled staff held in the left fist**: leather grip, bone ring, a ram skull with spiral horns, glowing sockets and a spirit gem, charm strings with beads and feathers, a red ribbon;
    - a frame drum on the back: painted sun, lacing, pegs, hanging feathers, a beater;
    - the headdress: fur band, carved forehead disc with gem, antler tines and cord wraps, a back feather fan;
    - face paint, bead bracelets, a medicine pouch, a gourd rattle.
  - **Chef** (100):
    - **the cleaver held up in the left fist**: bevelled edge forward, spine, bolster, riveted handle, hang hole, shine;
    - **sausages removed**;
    - a curled moustache, a neckerchief, double-breasted buttons;
    - an apron pocket with a wooden spoon and a whisk, an apron bow at the back (2 bones), a checkered towel (2 bones), a pepper mill;
    - a pan on the back with a fried egg, the handle out past the shoulder.
  - **Engineer** (177):
    - the claw arm: turret on the tank, bolted hubs, housed segments with hazard panels, hydraulic pistons, cable bundles and clips, a "07" number, a beacon, a camera, and 3 two-segment fingers with pads **gripping a wrench**;
    - tank gauges, a valve wheel, a copper pipe;
    - a walkie-talkie (antenna bone), a wrist computer, a chest pocket with pens and a ruler, a multimeter with probes, hazard tape, a gear cap badge.
- New tooling:
  - `Goblin_Concepts/redesign_v4_kit.py` (used by batch 3 and all later batches). It makes:
    - swept tubes, rings, link chains, gems, bevelled blades, chamfered boxes;
    - patches sewn into round surfaces with X stitches;
    - pixel text / pixel art / hazard stripe panels;
    - `Hold` (posed holds), atlas colour matching.
  - It also adds:
    - `FX_*` effect bones;
    - `rig_check` (one vertex group per piece on a deforming bone, weight 1, Armature modifier → follows the shared Walk / Idle / Wave);
    - `head_clearance` (pieces not on the head chain that cut into the real head mesh).
  - Batch 3 run: 0 loose, 0 rig issues, 0 head clipping (the pan, a fuse, the walkie antenna and the arm turret were moved for it).
- Hand-off for Roblox:
  - `Goblin_Concepts/export_v4_specs.py` scans all `<Key>_v4` collections and writes:
    - `Goblin_Concepts/physics_parts.json`: per goblin the accessory bones (parent, pivot, kind chain/pendulum/swing/bob/flicker/rigid, physics candidate, motion reference, piece count), held arm poses, pose (hover/tilt), and FX bones with the intended effect;
    - `Goblin_Concepts/V4_GOBLIN_SPECS.md`: the same, readable.
  - Currently 22 v4 goblins.
  - Held arm poses (the animator must hold them, like Basic): Knight R, Pyro L, Chef L. Bandit and Shaman hold with the normal hanging arm.
- Where: `Goblin_Concepts/redesign_v4_batch3.py` (single goblins: `ONLY = ("Knight",)` before exec), collection `Goblin_Redesign_v4_B3`. Backup before this round: `Goblin designs_backup_before_B3r2.blend`. Saved.
- Still open:
  - the user's verdict on round 2;
  - Legendary → Celestial (in progress, one by one), then the 36 new goblins (Astral → Beyond).

## 2026-10-10 — Goblin v4 batch 3: Hoarder, Brawler, Mystic (Rare), Knight, Pyro, Bandit, Shaman, Chef, Engineer (Epic) (Claude)
- What: following the batch-2 rules, with new fitting items:
  - **Hoarder:** an overstuffed treasure sack on the back (gold coins, a goblet with a ruby, a spoon spilling out), sewn patches, straps, a golden key on the belt. The cape is removed.
  - **Brawler:** big red boxing gloves (laced white cuffs, thumbs, gold logo) and a gold champion belt plate with a gem.
  - **Mystic:** a grand tiered purple turban with wraps, a brooch, an emerald and a white plume; a crystal ball in a brass cradle in the left hand. Bare feet kept.
  - **Knight:** a red 4-bone plume on the helmet, a blue kite shield with a cross on the left forearm, a sword in a scabbard on the right hip.
  - **Pyro:** a dynamite bandolier across the chest, round welding goggles on the forehead, soot smudges. The torch now has its own bone.
  - **Bandit:** a loot sack with a pixel "$" patch and coins on the back, a knife strap round the right thigh.
  - **Shaman:** an antler headdress (bone/bead band, branching antlers, 4 hanging feathers with their own bones).
  - **Chef:** a tall puffy chef hat with pleats, a frying pan strapped on the back, a 5-bone sausage string on the belt.
  - **Engineer:** a hazard-striped robotic claw arm mounted on the back, reaching over the right shoulder (Arm1-3 + Claw, with hoses).
- Where: `Goblin_Concepts/redesign_v4_batch3.py` (it uses batch 2 as a library), collection `Goblin_Redesign_v4_B3` (y = −15.2). 0 loose pieces; saved.
- Still open: the last batch (Legendary/Mythic/Divine/Celestial: King, Cyborg, Necromancer, Assassin, DragonRider, Demigod, ShadowStalker, Volcanic, CrystalLord, Stormbringer, Seraph, Zeus, Eternal, Phoenix, Nebula, GalaxyWalker, Supernova, Eclipse, Starlight, VoidWatcher).

## 2026-10-10 — Alchemist v4: tank details cleaned up (Claude)
- What: the flat square "bubbles", the ticks, the shine strip and the label stuck out across the octagon faces, and a clamp ring cut through the label.
  - The bubbles are now round 6-sided beads half sunk into the glass, on several faces.
  - The ticks and shines sit inside the back face, between the rings.
  - The clamps moved to z 0.765 / 0.915.
  - The warning label is a yellow diamond with a "!" on the empty glass top.
- Where: `redesign_v4_batch2.py` (alchemist); rebuilt + saved, no re-appearing user deletions.

## 2026-10-10 — Pebble sack + Muddy shovel fixes (Claude)
- Pebble:
  - The pebbles are now rounded 8-sided lathe stones with a light highlight, bigger.
  - The sack patches are sewn into the cloth: curved pieces that follow the sack surface (`sack_r` profile interpolation), with stitches along their edges.
- Muddy:
  - The shovel lay against the back of the head and floated off the back. It now runs flat across the back of the torso, beside the head (0.32,0.4 → −0.45,0.95), with the blade beside the head and ear.
  - The back rest is ray-cast on the torso only (z < 0.87).
- Batch rebuilt + saved; no re-appearing user deletions.

## 2026-10-10 — Basic v4: sack dots removed (the user) (Claude)
- What: the user deleted the white polka dots on Basic's bindle sack in Blender. Saved; `redesign_v4_batch2.py` no longer builds them, so a rebuild keeps the plain red sack.

## 2026-10-10 — Basic v4: bent carry arm, the stick on the shoulder (Claude)
- What: following the user's sketch, Basic's right arm is bent (upper arm down, forearm forward). The bindle stick is gripped in the hand, rests on the outer top corner of the shoulder and runs up behind; the sack hangs behind the head.
  - `carry_pose()` (`redesign_v4_batch2.py`) sets the pose on the copy's rig (its idle action is removed) and returns the hand bone's rest→posed delta.
  - The stick and sack are authored in the posed position and converted to the hand's rest space (`L["TO_RIG"]` is swapped temporarily), so they follow the arm. Bones: Bindle (child of Hand_R) → Bundle.
- Roblox (still to do at export): the pet animator must hold this arm pose for Basic instead of the walk arm-swing (a small bob only). The sack's Bundle bone swings with physics.

## 2026-10-10 — Goblin v4 batch 2, round 2 (the user's review) (Claude)
- What (all in `Goblin_Redesign_v4_B2`):
  - **Basic:** the bindle stick goes through the right hand and leans on the back of the shoulder. A cinched round red polka-dot sack (lathe) hangs from its end on a string. Bones: Bindle (child of Hand_R), Bundle.
  - **Leaf:** the giant leaf hat is replaced by a leaf necklace: a twisted vine cord on the chest round to the back, 7 shaped leaves (greens + orange/yellow, fine veins), wood and berry beads, an acorn. Each leaf has its own bone LeafCharm_0-6 for Studio physics.
  - **Angry:** the bandage bar is removed. The mohawk now sits exactly on the head top (ray cast): 5 layered spikes with their own bones, stubble only on the top.
  - **Sleepy:** the nightcap is a band + crown + a tube flopping to the side, with bone chain Cap1-6 + Pompom so in Studio it can swing back while walking and hang at rest. The pillow is strapped flat on the back.
  - **Pebble:** the boulder pack is replaced by a round cinched stone sack on the back (tie, tag, patches, pebbles peeking out), with shoulder straps, buckles and a belt tie. Bone: Sack.
  - **Shy:** a snug knitted hood (ear holes, ribbed rolled rim round the face, a collar into the sweater, eyelets with the drawstrings coming out, a 2-bone droopy tip, a heart patch).
  - **Muddy:** a detailed shovel lying flat on the back (D-grip, leather wrap, wood grain, ferrule, socket, spade blade with step, rivets, mud), with two buckled loops and a diagonal body strap over the shoulder.
  - **Crystal:** the sparkles are removed. A rocky geode is strapped on with a harness; 7 natural hexagonal crystals (two-tone facets, pointed) grow out of it, each with its own bone, plus small crystals at their feet.
  - **Alchemist:** the coat tails are removed. The tank is smaller and more detailed: on a brass backplate with clamp brackets + rings, liquid + empty glass top, bubbles, ticks, a warning label, a gauge, a side tap with drip, a valve wheel, and a copper tube over the shoulder.
- Tooling:
  - `Surface` ray casts snap pieces onto the body.
  - `lathe` / `tube` make round shapes.
  - `EXPLICIT` bone chains.
  - `floating_report()` checks every new piece touches the body (or a chain that does), with an inside test. The final run reports 0 loose pieces.
- Memory: `goblin-design-rules` (the user's rules for every next batch).
- Where: `Goblin_Concepts/redesign_v4_batch2.py`; saved in `Goblin designs.blend`.
- Still open: next batch (Hoarder, Brawler, Mystic, Knight, Pyro, Bandit, Shaman, Chef, Engineer); batch 1 (`Goblin_Redesign_v4`) has not been checked with `floating_report` yet.

## 2026-10-10 — Goblin v4: eye / face pieces sat in front of the face (Claude)
- What: the eye accents (shines, glints, monocle, squint lid, cheek mud) and the Tinkerer goggles floated ~0.12 in front of the face.
  - Cause: `FRONT = -0.336` is the nose-tip depth, not the face.
  - Measured on the Basic mesh (upright frame):
    - face plane y −0.2;
    - eye slabs x ±0.094..0.167, z 1.144..1.257, front y −0.216;
    - brows y −0.222;
    - snout to y −0.336.
  - Now `FRONT = −0.2`, `FRONT_NOSE = −0.336` (for things in front of the nose, e.g. Crystal's helmet lamp), eye accents at y −0.222 on x ±0.1305 / z 1.2, cheek pieces at y −0.205. The goggle strap now fits the head depth (y −0.215..0.215).
- Where: `Goblin_Concepts/redesign_v4.py` (constants, Tinkerer strap), `redesign_v4_batch2.py` (EX/EZ/EY, CHEEK_Y, lamp). Both batches rebuilt + saved; close-ups checked (Shy, Alchemist, Tinkerer).

## 2026-10-10 — Goblin redesign v4 batch 2: 9 goblins (Common ×7, Crystal, Alchemist) (Claude)
- What: the user asked for 9 goblins at a time; the higher the rarity, the more detail and the cooler; small eye accents are allowed. Each goblin has one big silhouette piece plus details, its own bones for every moving piece and a tiny eye accent.
  - **Basic** (17 pieces, 21 bones): a bindle stick over the right shoulder with a red polka-dot bundle peeking beside the head, a patched tunic.
  - **Leaf** (15, 22): one giant serrated leaf as an umbrella hat with a midrib, veins, stem and dewdrop; the old crown twigs are removed.
  - **Angry** (15, 25): a red 5-spike mohawk (one bone per spike), a forehead bandage with a tail, a red glint in the eyes.
  - **Sleepy** (21, 26): a long floppy striped nightcap flopping over to the side (bone chain Cap1–5 + Pompom: the "hood bouncing" idea), a big pillow on the back with moon and Z.
  - **Pebble** (15, 25): a big roped boulder pack (one bone per rock) showing above and beside the head, moss on top, one squinting eye.
  - **Shy** (23, 23): a big soft lilac hoodie hood with the ears poking out, a hood tip, drawstrings (own bones), a heart on the back, sparkly eyes.
  - **Muddy** (17, 22): a big shovel strapped diagonally on the back with its blade above the shoulder, mud splats, two dripping mud drops (bones), mud on the cheek.
  - **Crystal** (Uncommon, 33, 28): a tall miner's helmet with a lamp, a big crystal cluster on the back (6 shards with facet strips, their own bones), sparkles, crystal glints in the eyes.
  - **Alchemist** (Rare, 52, 34):
    - a big bubbling octagonal tank on the back, rising above the head, with brass rings, struts, glass shines, gauge, valve wheel and 5 bubble bones;
    - a copper tube over the right shoulder to a dripping nozzle;
    - two long lab-coat tails (bones), a monocle with chain, a formula scroll, a mortar + pestle;
    - the AGENTS.md belt flask is kept (now its own Flask bone).
- Where:
  - `Goblin designs.blend`, collection `Goblin_Redesign_v4_B2` (row y = −12.6).
  - Script `Goblin_Concepts/redesign_v4_batch2.py`, which uses `redesign_v4.py` as a library (now `if __name__ == "__main__"`, with `Design(..., y=)`).
  - New pieces may name their bone directly; `direct_bones` creates those bones with parents from `EXTRA_PARENTS`.
- Still open: the next batch (Hoarder, Brawler, Mystic, Knight, Pyro, Bandit, Shaman, Chef, Engineer); later Legendary → Celestial with the most detail.

## 2026-10-10 — Goblin v4: extra accessory bones, a real feather, the pencil fixed (Claude)
- What:
  - Each v4 rig has its own armature copy; the originals keep 19 bones. New bones:
    - Mushroom +9 (28): Hat, HatTop, BabyShroom, HatSprout, Shroom_L/R, SporePouch, PouchFlap, Basket.
    - Scout +14 (33): Hat, Feather, FeatherTip, FeatherBlue, Backpack, Bedroll, Lantern, Pan, Cup, PackMap, RopeCoil, ScarfEnd, HipPouch, HipMap.
    - Tinkerer +13 (32): Goggles, Tuft_0-2, Pencil, Wrench, Screwdriver, Hammer, Pliers, Tape, SpareGear, Pouch_L/R.
    - Bramble +12 (31): Branch_0-10 along the branches, with side branches parented to the main ones and the leaves, knots, thorns and flowers on the nearest branch; plus BerryPouch.
  - The pieces are re-skinned to their bone, with pivots at sensible spots (hat bottom, the top of hanging things, the goggle hinge).
  - Scout's feathers are now real feather shapes (`Design.feather`): a curved quill, a soft vane with two splits, a darker tip. Accessories may be less blocky (the user).
  - The Tinkerer's pencil lies straight front-to-back on the left ear root against the side of the head, with a wood tip, lead, ferrule and eraser.
- Decision (the user's question "animations better in Roblox?"): yes. Blender provides the bones and pivots; the secondary motion (hat hop, feather and lantern swing, branch sway) is done in Roblox at runtime on top of the walk.
  - The Blender walk preview is off (`PREVIEW_WALK = False`).
  - The `MOTION` table in the script holds the intended amplitudes and phases for the Roblox side.
- Where: `Goblin_Concepts/redesign_v4.py` (`BONES`, `rig_extend`, `MOTION`, `BRANCHES`), saved in `Goblin designs.blend`.
- Still open: the Roblox side (export with the extra bones + runtime secondary motion in the pet animator); the other 38 goblins.

## 2026-10-10 — Goblin redesign v4: detail pass 2, Tinkerer shoulder piece removed (Claude)
- What: on the user's request ("even more little squares") each design got a second detail pass (`mushroom2` / `scout2` / `tinkerer2` / `bramble2` in the script).
  - **Mushroom** (101 pieces): side spots on all tiers, dark rim shading pixels, tiny spots, a moss patch + sprout on the hat, shirt buttons, vest trim, mini mushrooms on both shoulders.
  - **Scout** (99): a second blue feather, hat band stitches, brim notches, stitching round the backpack, side-pocket flaps + buckles, a hanging lantern, a frying pan, strap holes.
  - **Tinkerer** (68): lens glints, strap studs, cuff bolts, a measuring tape, a spare gear on the hip, screws in a pouch, apron stitching. **The whole shoulder piece is removed** (the user: it floated and was not finished nicely): pauldron, gear, gauge, pipe, steam vent. The `DROP` list in the script does this.
  - **Bramble** (144): bark grain strips, leaf clusters with veins, pink flowers, vines round both arms, a berry pouch.
- Decision (the user): merge goblin pieces per bone + colour on the Roblox export. In Blender everything stays separate; memory `separate-parts-export` is updated.
- Where: `Goblin_Concepts/redesign_v4.py`, rebuilt + saved in `Goblin designs.blend` (`Goblin_Redesign_v4`).

## 2026-10-10 — Tinkerer v4: shoulder piece no longer inside the body (Claude)
- What: the brass pauldron (0.36 wide, centred at x 0.34) and its 0.3 gear (z 1.17) cut into the head and body. Both were moved and made smaller.
  - The pauldron is now 0.26 × 0.34, on the outer shoulder at x 0.45, sloping 25° down.
  - The gear stands on the pauldron's outer side, facing out (YZ plane, x 0.6).
  - Rivets, gauge and pipe follow the new position.
- Where: `Goblin_Concepts/redesign_v4.py` (tinkerer), rebuilt + saved in `Goblin designs.blend`.

## 2026-10-10 — Goblin redesign v4: more cube detail (Claude)
- What: on the user's request ("more detail, more cubes") the four v4 designs got detail built from small cubes; the big silhouettes stay.
  - **Mushroom** (+43 pieces): rim lips on two hat tiers, collar, 9 gill ridges underneath, 7 extra rim spots plus pixel steps, a baby mushroom on top, a spore pouch with a mushroom emblem.
  - **Scout** (+40): brim trim, crown crease, band buckle, feather barbs, strap buckles + badge, bedroll stripes, flap clasp, a patch with a star, a rolled map, a rope coil.
  - **Tinkerer** (+67): goggle rivets and side screws, strap buckle, a pencil behind the ear, pauldron rivets, a pressure gauge, an arm pipe, gear bolts, a belt buckle, pouch flaps + buttons, a hammer, pliers.
  - **Bramble** (+82): bark knots, thorns, side twigs with leaves, extra leaves, moss patches, shelf fungus, berry clusters.
- Where: `Goblin_Concepts/redesign_v4.py` (rerun rebuilds `Goblin_Redesign_v4`); saved in `Goblin designs.blend`.
- Still open:
  - The user's verdict, then the other 38 goblins.
  - Piece counts are now 60–110 per goblin. In Roblox each one becomes a MeshPart; merging them per bone (same material) at export would keep the pet cost down, if the user agrees, because their rule is "separate parts".

## 2026-10-10 — Goblin redesign v4: one big silhouette theme per goblin, same anatomy + rig (Claude)
- What: following a review the user pasted, the goblins keep the Basic anatomy and rig (in line with `AGENTS.md`). Each gets ONE big recognisable shape instead of small decorations; small or flat head pieces and tiny decorations are removed.
  - **Mushroom:** a wide 3-tier red mushroom hat (1.12 m) with big white spots on top, front and sides, cream gills underneath. The pins and badge are removed.
  - **Scout:** a slanted green hat (−13°) with a big red-yellow feather, and a big backpack: bedroll on top, side pockets, front and shoulder straps, a cup. The cap, small bedroll and compass are removed.
  - **Tinkerer:** big brass goggles with lenses over the eyes plus a strap, singed hair tufts, one brass pauldron with a gear on the left shoulder (UpperArm_L), and a tool belt with pouches, a big wrench and a screwdriver. The cap, chest gear and small tools are removed.
  - **Bramble:** thick angular branches round the shoulders, upper back and above the head (visible from the front), a bark mantle and back plate, leaves and berries. The twig headband and small bark pieces are removed.
- How:
  - Each design is copied: the same Body mesh, rig and accessories (data shared, the originals untouched).
  - New pieces are separate objects skinned 100 % to one bone, with flat sRGB `RV4_*` materials.
  - Parts are authored upright and transformed by the inverse display tilt (rig rotation y 0.095).
- Where: `Goblin designs.blend`, collection `Goblin_Redesign_v4` (row y = −10). Script: `Goblin_Concepts/redesign_v4.py`.
- Still open:
  - The user's verdict.
  - Then the other 38 goblins the same way.
  - Per-goblin poses (Sleepy slumps, King proud, Shy arms in) would need a per-goblin idle offset on the shared FollowMotion in Roblox.
  - The Roblox templates have to be re-exported.

## 2026-10-10 — Goblin remodel round 2: 3 concepts close to the original (Claude)
- What: the user found the v2 concepts much too far from the original. Three new concepts start from Basic's measured sizes and keep its look: one big head block, dark rectangular eyes, brow strips, a block nose, wedge ears, a brown tunic + belt + buckle, shorts, blocky limbs, the atlas greens. Each turns one knob:
  - **A Droopy** (head): long hanging ears, a hooked nose, heavy lids, an underbite with two teeth.
  - **B Stocky** (build): wider and shorter, big fists, short legs, small upright ears, brow bar, tusks, leather vest.
  - **C Lanky** (silhouette): longer legs and arms, a narrow tall head, long swept-back ears, big eyes with a shine, a pointy nose, blue tunic, scarf.
- Rig: 35 bones each (the original 19 names + the v2 extras); every piece is a separate object.
- Where: `Goblin designs.blend`, collection `Goblin_Concepts_v3` (y = −7). Script: `Goblin_Concepts/build_concepts_v3.py` (it uses `build_concepts.py` as a library). v2 (`Goblin_Concepts_v2`) is still there.
- Still open: the user picks one. Next steps: update `AGENTS.md`, apply the direction to the 42 designs, animations, Roblox export.

## 2026-10-10 — Goblin remodel: 7 new body concepts with extended rigs (Claude)
- What: the user finds all goblins too alike and wants them remodelled. Seven new concepts were built in the existing blocky style: sharp boxes and tapered boxes, flat sRGB colours, no bevels, spheres or cones.
  - 1 Brute: hulking, hunched, huge forearms, tusks.
  - 2 Sneak: tall and lanky, horizontal ears, long nose, scarf, dagger.
  - 3 Chonk: round belly, overalls, beanie, cookie.
  - 4 Imp: orange, huge head and bat ears, horns, digitigrade legs, spade tail.
  - 5 Elder: hunched under a robe, hooked nose, beard, mushroom staff.
  - 6 Sprout (cute): chibi, leaf sprout.
  - 7 Pip (cute): chibi in a hoodie, floppy hood tip, backpack.
- Rig: 36–39 bones each.
  - The original 19 bone names are kept, so the shared Walk / Idle / Wave actions still drive them.
  - Added: Chest, Neck, Jaw, Nose, Eye_L/R, Brow_L/R, EarTip_L/R, Fingers_L/R, Thumb_L/R, Toes_L/R.
  - Extras: Loincloth, Scarf1-2, Belly, Hat, Tail1-3, Beard1-2, Staff, RobeSkirt, Sprout1-2, HoodTip, Backpack.
  - Each piece is a separate object, skinned 100 % to one bone. Deformation was tested on 4 bones.
- Where:
  - `Goblin designs.blend`, collection `Goblin_Concepts_v2` (row at y = −4; the 42 old designs are untouched).
  - Rebuild script: `Goblin_Concepts/build_concepts.py` (it rebuilds the whole collection).
  - Backup: `Goblin designs_backup_before_concepts_20261010.blend`.
- Still open:
  - The user picks a direction.
  - Then `AGENTS.md` ("every goblin = the Basic anatomy, no remodel") must be updated, because it conflicts with this remodel.
  - Then the 42 designs get re-dressed on the chosen bodies, followed by animations for the new bones and the Roblox export / pet templates (`GoblinPetRenderer` expects the old rig).

## 2026-10-10 — Record cutscene footer: only the rarity pill (Claude)
- What: the "PREVIOUS BEST 1/…" / "YOUR FIRST BIG RECORD" line under the odds is removed. The rarity pill (e.g. MYTHIC) stays alone and is centred, because the footer is a centred horizontal UIListLayout.
- Where: `ReplicatedStorage.RecordCutscenes` (the title block's footer, ~line 630). Studio only; the local copy, if any, is behind.
- Test: compiles; not replayed (the RollGui attribute `StudioTestReveal` = "Record" shows it).

## 2026-10-10 — No roll spoilers in the player list / podium (Claude)
- What: the server decides a roll at once, so your new stats (and a world record) showed before the reveal ended.
  - `RollGui.RollClient` now sets the RollGui attribute `Rolling` true while `playRoll` runs (pcall'd, then false).
  - `PlayerListClient`: your own row holds updates while `Rolling` and catches up when it ends.
  - `GlobalBestPodium`: your own new world record waits until `Rolling` ends (max 12 s).
- Also: the player-list toggle now has its own podium icon (`Goblin_UI/make_leaderboard_icon.py`, rbxassetid 115159503313974) instead of the crown, and the rows are TextButtons (Activated) so the player menu opens reliably.
- Test (Studio): with `Rolling` = true, a StatRolls change left the row on 5; after `Rolling` = false it showed the new value.
- Still open: other players can still see your new stats early in the list (only your own view is held back).

## 2026-10-10 — Player list like Roblox's: corner toggle, fade, player menu (Claude)
- What: `PlayerListClient` (v5, data PNG 76834430892790 / PLS5).
  - It now sits in the top-right corner (ScreenGui IgnoreGuiInset; the list at y 60).
  - A round toggle button with the gold crown icon sits in the top bar's right corner, like Roblox's leaderboard button. The X in the list and Tab also close it.
  - The list fades and slides in/out (CanvasGroup GroupTransparency, 0.22 s).
  - There is no coloured bar behind your own row; a row turns light grey on hover.
  - Clicking a row opens Roblox-style menu, with a fade, left of the list (on the ScreenGui, not clipped):
    - headshot, display name, @username;
    - Friend request (`SetCore PromptSendFriendRequest`) or Unfriend (`PromptUnfriend`);
    - Examine avatar (`GuiService:InspectPlayerFromUserId`);
    - Block (`PromptBlockPlayer`).
    - It closes on a click outside.
  - The header reads "People". Texts: GameText `PlayerList.Open/FriendRequest/Unfriend/Examine/Block`.
- Test: the capture shows the crown button in the corner and the list under it. The row menu can't be clicked through the MCP (simulated clicks don't reach GUI input), so check it by hand.

## 2026-10-10 — Own player list: Rolls, Ascensions, Rarest Roll with rarity gradient (Claude)
- What: `StarterPlayerScripts.PlayerListClient` replaces Roblox's player list (CoreGui PlayerList off). It sits top right (y 64) and Tab folds it.
  - Columns, in the user's order: Player | Rolls | Ascensions | Rarest Roll.
  - Rarest Roll shows "1/250K" in the colour of the rarity those odds reach, as a light → colour UIGradient with an outline. Odds beyond 10× the rarest rarity get a cycling rainbow.
  - Rows are sorted by rolls, and your own row is highlighted.
- Data: `GoblinInventory` keeps `data.rolls` (+1 per roll in `RecordRoll`) and `data.ascensions` (always 0 for now: Ascend is not built yet).
  - Player attributes StatRolls / StatAscensions / StatBestOdds are set in `publish`; RecordRoll updates StatRolls and StatBestOdds.
  - Backup: `UI_Backup.GoblinInventory_before_playerlist_20261010`.
- Texts: GameText `T.PlayerList`. Local: `Goblin_UI/PlayerListClient.client.luau`.
- Test (Studio): after 2 rolls the list showed superQcrazy | 2 | 0 | 1/8 in the rarity colour.
- Still open: the admin window (right side) overlaps the list for admins; fold one of them. When Ascend is built, it must increase `data.ascensions`.

## 2026-10-10 — Giant avatar: always cleaned + admin "hide avatar" button (Claude)
- What: Roblox has no API to detect inappropriate avatars, so two safeguards were added.
  - **Always cleaned (server, `GlobalBestRoll.cleanDescription`):**
    - removed: layered clothing; the bundle body parts (Torso / arms / legs reset to default);
    - clamped: Height 0.95–1.05, Width/Depth 0.8–1, Head 0.95–1; BodyType and Proportion set to 0;
    - kept: skin colours, head + face, shirt/pants, normal accessories.
  - **Emergency button** in the Best Roll tab: "Verberg avatar (standaard skin)" / "Toon avatar weer".
    - It sets `record.hidden` with rev + 1, and the giant becomes a neutral default avatar. The record, name and odds stay.
    - Server: `AdminSet({ hide = bool })`.
    - The podium skips the storm/notification when only `hidden` changes.
- Where: `GlobalBestRoll` (GR06), `AdminBestRoll` (ABR2), `GlobalBestPodium` v26 (GB26), GameText `Admin.HideAvatar/ShowAvatar/HiddenDone/ShownDone`.
- Test (Studio): zwieber21's Rthro avatar is now built without layered clothing and with a default body (shirt, head and accessories kept). Hide → default avatar, 0 accessories, `"hidden":true`; show → back.
- Still open: publish for live.

## 2026-10-10 — Giant: the same arm for every avatar + two connected points (palm ↔ statue head) (Claude)
- What:
  - With bigger avatars the arm went wrong. Rthro arms are up to 1.55× longer, so the arm aimed at the goblin reached far past it and crossed in front of the chest; the user's own avatar also had the hand and circle in front of the chest.
  - Now every avatar is scaled so its arm is 105 studs. The body is moved so the right shoulder sits on one fixed spot, and the arm reaches forward and out over the cliffs (24° out, 24° down), clear of the body.
  - Two points (the user's idea):
    - in the middle of the palm, 5 studs off it;
    - in the statue's head (its Head bone, so it follows the idle animation).
  - The rune circle lies flat against the palm, facing the head; every stream effect runs between the two points.
- Where: `StarterPlayerScripts.GlobalBestPodium` v25 (data PNG 116303851234541 / GB25), local `Goblin_UI/GlobalBestPodium.client.luau`.
  - The Giant block: `ARM`, `SHOULDER`, `ARM_DIR`, `HAND_GAP`, the Build pose.
  - Step: the two points.
  - The heartbeat: `goblinHead`.
- Test (Studio, isolated): the admin Best Roll tab set different holders. These avatars were tested:
  - 591297703, 2970683260 (zwieber21, Rthro), 1124895504, 1 (Roblox), 156.
  - Results:
    - the hand lands at the same spot (±10 studs);
    - the waists stay behind the wall;
    - the circle is exactly 5 studs off the palm for every hand shape and faces the goblin (dot 0.99–1.00);
    - captures from the spawn and from beside the podium show the hand and circle left of the body, with the stream diagonal to the goblin's head.
- Still open: nothing new. The live game needs a publish (see the entry below).

## 2026-10-10 — One admin window + Best Roll editor; giant: bigger eyes + sound; small avatar removed (Claude)
- What:
  - **Admin window:** every admin / test tool in ONE folding window on the right, with tabs Roll / Reveal / Cutscene / Best Roll / Coins. The four separate panels that covered the screen are gone.
  - **Best Roll tab:** admins can set the worldwide best roll to any goblin, mutations, odds and player (username or UserId), or reset it. It also works with a lower record (`rev` override); see section 1 "Best roll ever podium".
  - **Giant:** much bigger, cooler eyes and 3D sound that swells near the podium; see section 1.
  - The small dancing avatar beside the stairs (+ `Podium.AvatarTag`) is removed.
- **Incident (2026-10-10):** MessagingService works from Studio.
  - My Studio admin tests were published on topic `GlobalBestRoll`: "@Roblox Stormbringer 1/203Sx" (rev 1), "@builderman Pyro 1/16" (rev 2), reset (rev 3), plus a separate test instance's "@superQcrazyXD Stormbringer 1/203Sx" (rev 0).
  - zwieber21's server (UserId 2970683260, running the new code, probably a Team Create Studio Play) then posted "Assassin 1/32" with rev 3.
  - Live servers on the published (older) code would accept a 2.03e23 record in memory, so the fake "@Roblox 1/203Sx" may show on live podiums until those servers restart. It is not verified whether Studio messages reach live servers.
  - The DataStore was not written (Studio has no API access; the old live code only writes on real rolls, and nothing beats 2.03e23 there).
  - Fixed: Studio is now isolated (no publish/subscribe, no DataStore writes).
- Where:
  - `ReplicatedStorage.AdminPanelUI` (new, local `Goblin_UI/AdminPanelUI.luau`).
  - `StarterPlayerScripts.AdminBestRoll` (new, `Goblin_UI/AdminBestRoll.client.luau`).
  - `RollGui.RollClient`: admin block (`Goblin_UI/RollClient_adminblock.luau`).
  - `CoinDropClient`: Coins tab (`Goblin_UI/CoinDropClient_adminblock.luau`).
  - `GoblinInventoryServer`: `AdminSetBestRoll` (`Goblin_UI/GoblinInventoryServer_bestroll.luau`).
  - `GlobalBestRoll` (rev, AdminSet, LIVE gate; `Goblin_UI/GlobalBestRoll.luau`).
  - `GlobalBestPodium` v22 (`Goblin_UI/GlobalBestPodium.client.luau`).
  - GameText `T.Admin`.
  - Backups: `ServerStorage.UI_Backup.*_before_adminpanel_20261010`.
- Test (Studio Play):
  - All 5 tabs show; the old panels are gone.
  - Set "@Roblox 203Sx" → the giant became the Roblox avatar.
  - A lower record (builderman 1/16 by UserId 156) replaced it.
  - An unknown player or goblin is refused. Reset clears the record and the giant.
  - A real roll can't beat a high admin record; after a reset the next real roll (Crystal 1/4) became the record.
  - No GlobalBestRoll DataStore writes from Studio. All 10 sounds load; the loops play under the SFX volume.
  - Cost: render CPU about +1.0 ms from the spawn (was +0.9), GPU within noise.
- Still open:
  - **The live game only gets all of this after a publish.** To clear a possibly stuck fake "@Roblox 1/203Sx" on live servers, restart them (Creator Dashboard → Shut down all servers / migrate to the latest update).
  - The Best Roll tab is easiest to use in a live server: there the override reaches every server and the DataStore. In Studio it now stays local.

## 2026-10-10 — Podium: the colossal record holder behind the back cliffs (Claude)
- What: after the user's two references (a cosmic giant behind the forest projecting the goblin):
  - The best-roll holder's real avatar (outfit, accessories, dynamic head) now stands colossal behind the back wall: scale 72, the lower body hidden by the cliffs.
  - It leans forward, looks down at the goblin, and reaches its right arm over the wall towards the goblin.
  - A gold rune circle hangs past the hand. From it a gold → purple → cyan stream with a spiral of comet trails, rune rings, streaks, sparks and floating runes runs down into the goblin's aura and onto the floor circle.
  - Glowing cyan eyes with short trails, little lightning flashes along the face, a halo, stars and an outline. Only the wrist moves.
  - It pulses just before every surge and rises from behind the cliffs for a new holder. The map itself is untouched.
- Where:
  - `StarterPlayerScripts.GlobalBestPodium`: the `Giant` block before "-- the goblin". v21, data PNG 134890841189339 / GB21. Local `Goblin_UI/GlobalBestPodium.client.luau`.
  - Hooks: avatar `refresh()` builds and clears it, `setActive`, `Giant.Cull(dist)`, `Giant.Step(...)`, `setRecord` (rise + big pulse).
  - Textures: `Goblin_UI/make_giant_fx.py`.
- Test:
  - Set the ServerStorage attribute `StudioGlobalBest` to "Stormbringer|Huge,Shiny|203000000000000000000000". Look up from the spawn.
  - Rise test: post a better record, then "reset", then a record again. The head goes from y 122 to 440 in ~2.6 s.
  - Cost, podium + giant on vs off: render CPU about +0.9 ms, GPU +0.9–1.5 ms near (the podium alone was +0.7 / +0.7).
- Gotchas:
  - These avatars are AnimationConstraint rigs, not Motor6D. Anchoring and placing the parts by FK is safe; physical joints would sag at 72×.
  - Black captures came from a minimised Studio window (see memory).
  - The Edit-mode prototype was removed again.
- Still open:
  - From the spawn you must look up ~35–55° to see the giant. The wall + trees hide everything below that, so the default low camera only shows the hand, circle and stream.
  - Pose tuning per avatar (very tall or short Rthro bodies) is untested.

## 2026-10-09 — Podium: the record holder's avatar beside the stairs (Claude)
- What: the avatar of whoever rolled the best roll ever now stands left of the podium stairs (the user's red dot).
  - It cycles through emotes: wave, dance 2, cheer, dance 3, dance.
  - "@username" sits above its head, white→light-cyan with a dark outline; it pops in and bobs.
  - A small rotating cyan rune ring and glow pad lies under its feet.
  - It is rebuilt only when the record's userId changes.
- Where:
  - Server: `ServerScriptService.GlobalBestRoll` (`buildAvatar`, the R15 model into `ReplicatedStorage.GlobalBestAvatar` with attributes UserId, Name, Count). Local copy `Goblin_UI/GlobalBestRoll.luau`.
  - Client: `StarterPlayerScripts.GlobalBestPodium` (the avatar do-block, `avatarStep`), v18 (data PNG 108691539246579 / GB18).
  - Text: GameText `Podium.AvatarTag = "@%s"`.
- Test: ServerStorage `StudioGlobalBest` = "Stormbringer|Huge,Shiny|203000000000000000000000" shows your own avatar. Cost is within noise (render CPU ±0, GPU ±0.4 ms).
- Gotchas found:
  - The 258-part model replicates in pieces. A clone made on ChildAdded had no clothes or hair, so the client now waits for `Count` descendants.
  - The visible sand there is about 0.35 studs above its collision (150.1). The pad and feet use 150.45.
- Still open: in a live server, players with an R6 avatar are also shown as R15 (forced, because the emotes are R15).

## 2026-10-09 — Podium: effects visible when zoomed out + animated gold odds (Claude)
- What:
  - Zoomed out, the effects shrank to a few pixels. Bolt, cage, streak, pillar and comet widths and all particle sizes now scale with camera distance: `zoomK` = dist / 55, clamped 1..3.2. Particle sizes are rebuilt only when zoomK changes by more than 4 %.
  - Odds text is animated:
    - the gold gradient's bright band breathes;
    - a white/cyan shine sweeps across every 2.6 s (a separate `OddsShine` label);
    - the odds punch on every surge.
  - Huge odds stay exact and on one line. `full()` used to print float junk past 1e15 (203 Sx became 202,999,...,805,696). The odds are no longer wrapped, and the sign widens with text length (44 → max 77 studs). Sparkles use ScaleType Fit so they don't stretch.
- Where: `StarterPlayerScripts.GlobalBestPodium` (v14, data PNG 80209952054323 / GB14), local `Goblin_UI/GlobalBestPodium.client.luau`.
- Test:
  - Set the ServerStorage attribute `StudioGlobalBest` to "Stormbringer|Huge,Shiny|203000000000000000000000". It shows 1 / 203,000,000,000,000,000,000,000 on one line.
  - Cost, podium on vs off: near render CPU +0.7 ms and GPU +0.7 ms; at 128 studs CPU +0.36 ms and GPU +0.4 ms.
- Still open: the Studio test record (203 Sx) only lives in this Play session; the real DataStore path is still untested.

## 2026-10-09 — Podium: the goblin's walking ring really gone (Claude)
- What:
  - The big blocky ground ring came back on the podium goblin. The cause: `GoblinPetFX` (line ~510) sets `self.auraGui.Enabled = near` every frame, so switching the SurfaceGui off did not last.
  - The fix: GlobalBestPodium now sets `fx.auraGui = nil` and takes the whole `FXAura` part out (`Parent = nil`). That removes the ring, its glow and its white square motes (the white square seen on the ground). GoblinPetFX still moves the parentless part, which is harmless.
- Where: `StarterPlayerScripts.GlobalBestPodium` (+ local copy).
- Test (Play): 5 s after the record shows, 0 `FXAura` in the Workspace; capture without the ring; no errors.
- Gotcha: to hide a part of a goblin's FX, check whether GoblinPetFX re-sets it every frame (`auraGui.Enabled`, `wanted[e]` for emitters).

## 2026-10-09 — Spawn podium: the BEST ROLL EVER, worldwide (giant goblin + electric FX + sign) (Claude)
- What (the user's design: the best roll ever over all servers; when someone rolls better, the new one appears; the sign like their reference `images/185.png`; effects like their reference: electric cyan, lightning everywhere, outline):
  - **Server `ServerScriptService.GlobalBestRoll` (ModuleScript):**
    - DataStore `GlobalBestRoll_v1` key `record` = { odds, goblinId, mutations, userId, name, display, t }.
    - Written with UpdateAsync that only keeps the better record (two servers can't overwrite a better one). Writes are coalesced (one per 6 s with the best pending).
    - Live sync over MessagingService topic `GlobalBestRoll`, plus a DataStore re-read every 120 s.
    - For clients: `ReplicatedStorage.GlobalBestRoll` (StringValue JSON).
    - RollServer calls `GlobalBest.Submit(player, goblinId, mutations, odds)` with the rarest result of every roll, in a pcall.
    - Studio (no API access): it works in memory for the session.
  - **Client `StarterPlayerScripts.GlobalBestPodium`** on the podium top (75.18, 155.21, 7.65):
    - **The goblin:** `GoblinPetRenderer.Puppet` (the real model with its mutation looks), 7.5× (Huge 4.5× on top of its 2×), floating, idle only, turning a little towards the spawn.
      - Cyan outline (Highlight, no fill).
      - Its own walking ground ring (`FXAura`) is switched off, and its own sparkles are scaled back (×0.2).
    - **Sign** (BillboardGui, after the reference):
      - the user's crown style (`Goblin_UI/make_crown.py` → 91542675775753);
      - "1 / 100,000,000": the **full number, never shortened** (the user), gold gradient, dark outline + shadow;
      - "Rolled by @username";
      - twinkling sparkles.
      - It sits 8 studs above the goblin's real top (measured from its visible body parts once a second).
    - **FX, all electric cyan / white** (the user: no red rarity glow; only Rainbow cycles):
      - floor: a rotating hexagon / circuit circle + rune ring (`Goblin_UI/make_podium_fx.py`) and lightning cracks that flash over it;
      - a permanent crackling lightning cage (6 bolts) and a pool of 32 bolts (arcs round the goblin, into the floor, from the sky, ~60 / s) with spark impacts;
      - 4 comet trails spiralling up, 10 energy streaks round the edge, a light pillar;
      - rising sparks, hexagon glyphs, falling glitter, rotating rays behind the goblin.
      - Every 5 s a surge: triple shockwave, crack flashes, a burst of 90 sparks, bolts from the sky, an energy ring.
    - **New record:** a lightning storm, the old goblin is pulled up into the light, the new one drops in with a shockwave, the sign pops, and a toast "NEW BEST ROLL EVER by @x".
    - No lights (light minimal); everything stops beyond 320 studs. PerformanceMode / LessVFX: fewer bolts and particles.
  - GameText: new section `Podium` (Odds "1 / %s", RolledBy, Empty, NewRecord).
- Changed on the user's feedback during the build:
  - the crown redrawn like their reference;
  - no red glow (fixed cyan palette, no aura slab);
  - the goblin bigger twice;
  - the goblin's own walking ring removed (I first removed the hexagon circle by mistake; it is back);
  - the sign lower;
  - many more and bolder effects;
  - the odds written out in full.
- Where:
  - Studio: `ServerScriptService.GlobalBestRoll`, `ServerScriptService.RollServer` (require + Submit after RecordRoll), `StarterPlayerScripts.GlobalBestPodium`, `ReplicatedStorage.GameText`.
  - Local: `Goblin_UI/GlobalBestRoll.luau`, `Goblin_UI/GlobalBestPodium.client.luau`, `Goblin_UI/GameText.luau`, `Goblin_UI/make_crown.py`, `Goblin_UI/make_podium_fx.py`.
  - Textures: magic_circle 139936856008517, magic_inner 114413534523841, lightning 116310075477732, beam_soft 103908910362405, shock_ring 117210184921967, floor_cracks 135687695372934, hex_particle 90845075940390.
- Test (Play):
  - ServerStorage attribute `StudioGlobalBest` = "Stormbringer|Huge,Shiny|100000000" shows a test record (`"reset"` clears it). Captures after every change. No errors.
  - Cost from the spawn view, 3× on / off: render CPU +0.4 ms, GPU +0.6 ms, heartbeat +0.08 ms per frame.
- Still open:
  - The real DataStore / MessagingService path can only be tested in a published server (Studio has no API access).
  - The look at night, and on a phone.

## 2026-10-09 — Shrine Auto Roll switch: knob centred in the track (Claude)
- What:
  - The cause: the white knob (49 × 49) was not concentric with the track's round ends (track 118 × 50). With ON it sat 3.5 px too far left and 1.5 px too high, so it looked off at the right end.
  - The fix: the knob is now 44 × 44, 3 ref px inside the track on every side: `KNOB_OFF` (135, 22), `KNOB_ON` (203, 22). It slides with Quad instead of Back, so it no longer overshoots the end.
- Where: `StarterPlayerScripts.GoblinShrineClient` (+ local copy).
- Test (Play): measured gaps OFF left / top / bottom 1.81 px, ON right / top / bottom 1.81 px (equal); captures; no errors.

## 2026-10-09 — Powers detail: "Maxed Out!" at 5 stars (Claude)
- What: at 5 stars the "Next star" box no longer shows its label, bar and number. It only says "Maxed Out!" centred (green Montserrat Heavy). GameText `Powers.MaxStars` = "Maxed Out!".
- Where: `StarterPlayerScripts.GoblinPowersInvClient` (`dMaxed`, `nextLabel`), `ReplicatedStorage.GameText` (+ local copies).
- Test (Play): World Tree 5★ selected → "Maxed Out!" centred (capture); no errors.

## 2026-10-09 — Shared UI pieces: ReplicatedStorage.GoblinUIKit (SortPill); Powers grid outline (Claude)
- What:
  - **New rule (the user):** a UI element that appears on more than one page is ONE shared piece; a page only passes its texts.
    - New ModuleScript `ReplicatedStorage.GoblinUIKit` (local `Goblin_UI/GoblinUIKit.luau`).
    - First piece: `Kit.SortPill(opts)` = the Inventory's sort pill + the menu that opens upwards (moved out of GoblinPetClient unchanged).
    - opts: `bar`, `page`, `options` ({key, label}), `format` ("Sort: %s"), `default`, `onChange(key)`, and optionally `x` / `width` / `mid` / `barH`.
    - Returns `{ pill, value, menu, set, get, close }`.
  - The Goblins page (GoblinPetClient) now calls `Kit.SortPill` (Rarity / Best Goblin / Newest). Its own `pill()` and sort-menu code are gone (198,820 → 195,305 chars). Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_uikit_sort`.
  - The Powers page calls the same `Kit.SortPill` (Rarity / Stars / Luck). The earlier clone + rewire is removed.
  - The Powers grid box now has the same outline as the Equipped Powers box and the detail panel (#3b8fe0, 4 px; detail changed from #2e8fe8 to match).
- Where: `ReplicatedStorage.GoblinUIKit`, `StarterPlayerScripts.GoblinPetClient`, `StarterPlayerScripts.GoblinPowersInvClient` (+ local copy).
- Test (Play): Goblins tab sort menu captured (Best Goblin green, same as before); Powers tab sort menu captured (same piece, Rarity green); the grid outline matches; no errors.
- How to add a piece: move the builder into GoblinUIKit with an opts table (texts from GameText), then call it from every page that needs it. Never clone instances and rewire them.

## 2026-10-09 — Powers page polish: fits the Inventory, rarest first, sort menu, owned only, empty state, pattern (Claude)
- What (the user's list):
  - **Fits inside the Inventory.** The content sits in a frame that copies the Inventory's `Sheet` (position and size). `fitRef()` sets the UIScale so the 1201 × 808 reference layout fits it in both directions; it runs on show and on resize. Before, the grid stuck out below the sheet.
  - **Sort menu** like the Goblins page: a clone of `GoblinsPage.SortMenu` with Rarity (default), Stars and Luck. It opens upwards, shows the current one in green and closes when the page hides.
  - **The rarest power top left.** Sort Rarity = rarity high → low, then stars. Stars / Luck sort by that first, then by rarity. The default selection is the first equipped power, else the rarest owned one.
  - **Only owned powers** in the grid; there are no ??? / black cards any more.
    - No powers: an empty state like the Goblins page (3 soft blue power silhouettes bobbing, "No powers yet", "Roll powers at a shrine with its tokens!").
    - The detail panel then shows the shrine icon + "Get powers at the Forest Shrine with tokens from mobs."
    - A shrine filter tab only shows when you own a power from that shrine (no powers: only "All").
  - **"2 slots unlocked • Max 5" removed.**
  - **Background pattern** on the grid and the detail panel, after the idea of the goblin hover card's square pattern but with a powers theme: sparkles + diamonds (`Goblin_UI/make_powers_pattern.py` → 81739447297831). It is faint on the left and clear on the right, with a shine sweep every 6 s while the page is open.
  - The Equipped badge is no longer cut off at the top of the grid (`ROW_Y` 22).
  - GameText Powers: + `NoPowers`, `NoPowersSub`, `DetailEmpty`; − `SlotsInfo`.
- Where: `StarterPlayerScripts.GoblinPowersInvClient`, `ReplicatedStorage.GameText` (pushed via `ScriptEditorService:UpdateSourceAsync`); local copies in `Goblin_UI/`.
- Test (Play):
  - No powers → empty state, only "All", everything inside the Inventory (capture).
  - 6 powers → World Tree top left, Rarity / Stars / Luck menu, Forest tab, badges complete, pattern visible (capture).
  - No errors.

## 2026-10-09 — Inventory > Powers page v2 (after reference 177) + power slots (Claude)
- What:
  - **New mechanic: power slots.**
    - Powers count only when equipped. `PowerConfig.SLOTS` = 2 are unlocked and `SLOTS_MAX` = 5 are shown; the locked ones say "Ascend".
    - All equipped powers multiply (`PowerConfig.LuckOf` → `EquippedIds`).
    - A newly rolled power goes into a free slot by itself. Old profiles without an `equipped` list get their best powers (`BestIds`).
    - Before this, the best power per shrine counted automatically.
  - **The Powers page** (`GoblinPowersInvClient` v2), after the user's reference `images/177.webp`. Their changes: the Powers tab button stays as it was, and where the reference had a description it says "You own xN of this power." Icons are the user's original art.
    - Header: "Search powers..." (a clone of the Goblins page's search box). It searches owned powers by name.
    - "Equipped Powers": 2 slots (equipped power or "+ Empty"), 3 locked slots (grey padlock `Goblin_UI/make_grey_lock.py` → 80012040084904, "Ascend"), the "1/2" pill and "2 slots unlocked • Max 5".
    - Filter tabs: "All" + one per shrine with its token ("Forest").
    - Grid: 3 cards per row (dark icon panel per rarity, name, rarity pill, 5 stars, "✓ Equipped" badge). A click selects the card; the grid scrolls when needed. Unowned cards show a silhouette and "???".
    - Detail: name, icon tile, rarity pill, shrine pill (token + "Forest"), stars, "You own xN of this power.", Luck xN (clover), "Duplicates upgrade this power.", a Next star bar (spare / `UPGRADE_COST`), and Equip / ✓ Equipped + Unequip (grey "Locked" when not owned).
    - Bottom: a clone of the Goblins page's bar (no trash): Sort (Rarity / Stars / Luck; owned first), "Powers: N", "Equip Best".
    - Built in reference px (sheet 1201 × 808) inside a UIScale of 0.614.
  - Server (PowerServer):
    - `data.powers.equipped`, attribute `PowerEquipped` (JSON);
    - RemoteFunctions `PowerRemotes.EquipPower(id, on)` and `EquipBestPowers()`;
    - message `Server.SlotsFull`;
    - `StudioPowerState` also takes `equipped`.
  - GameText: the `Powers` section was rewritten for the new page.
- Where:
  - Studio:
    - `StarterPlayerScripts.GoblinPowersInvClient` (v1 backup `ServerStorage.UI_Backup.GoblinPowersInvClient_v1`);
    - `ReplicatedStorage.PowerConfig`, `ServerScriptService.PowerServer`, `ReplicatedStorage.GameText`.
  - Local: `Goblin_UI/GoblinPowersInvClient.client.luau`, `Goblin_UI/GameText.luau`.
- Test (Play), with `StudioPowerState` set to 6 owned powers and Forest's Blessing equipped:
  - The page opened on the Powers tab, captured.
  - Clicking Heartwood → Equip: 2/2 equipped.
  - A 3rd equip gave "All power slots are full…".
  - Equip Best → World Tree + Heartwood.
  - Luck followed the equipped powers. No errors.
- **Gotcha:** writing `.Source` of a script that is open in an editor tab (e.g. after a multi_edit) can be reverted when Play starts. Write through `ScriptEditorService:UpdateSourceAsync`.
- Still open:
  - Balance: two high-star powers multiply quickly (World Tree 5★ × Heartwood 4★ = x24.5).
  - Ascend unlocking more slots.

## 2026-10-09 — Shrine: luck with the clover on the carousel card too, more room round the clover (Claude)
- What:
  - The middle carousel card shows its luck like the global luck: clover + "1.25x", green gradient, black outline.
    - The clover sways together with the Upgrade box's clover: one Heartbeat while the panel is open, ±8°.
    - The side cards fade it out and hide it.
    - To make room, the middle card's icon, name and rarity sit a little higher (icon −44, ×1.62; name +50; rarity +77).
  - Upgrade Power box: more room above and below the clover row.
    - The tile, name, rarity and stars moved up 6–7.
    - The button and hint moved down 6.
    - The clover is 34 and the text 30.
  - GameText: `Shrine.CardLuck` is removed. Both places use `Shrine.LuckValue` ("%sx").
- Where: `StarterPlayerScripts.GoblinShrineClient` (+ local copy), `ReplicatedStorage.GameText` (+ local copy).
- Test (Play): capture: Leaf Whisper in the middle shows clover + "1.05x", and the Upgrade box has room round the clover; no errors.

## 2026-10-09 — All UI texts in one place: ReplicatedStorage.GameText (Claude)
- What:
  - New ModuleScript `ReplicatedStorage.GameText` with every text players see in the menus, per section:
    - `Shrine`, `Powers`, `Teleport` (incl. the zone names + preview texts), `Coins`, `Settings`, `Extras`, `Gates`;
    - `Inv` (Inventory, Index, Shop, Ascend, toasts);
    - `Roll` (ROLL, Auto Roll, Hide Rolls, LUCK UP!, COIN RAIN, BONUS!, toasts);
    - `Server` (PowerServer / zone-unlock messages).
  - The user edits a text there and presses Play. The header explains the rules: keep the quotes, keep `%d` / `%s`, keep the comma.
  - Every script reads it through GoblinUI: `UI.Text.<Section>.<Key>` (`UI.T` is the same table). GoblinUI now does `UI.Text = require(GameText)`. Scripts that don't use GoblinUI require GameText themselves (PowerServer, GoblinInventory).
  - Converted:
    - GoblinShrineClient, GoblinPowersInvClient, GoblinTeleportClient (its `ZONES` table now only holds the icons), GoblinSettingsClient (also the tab labels), GoblinExtrasClient, ZoneGateClient;
    - GoblinPetClient: panel titles, tab labels, search, sort, equip, empty states, delete bar, Shop / Ascend, Index, stats, odds, toasts. Its size went from 198,899 to 198,820 chars;
    - RollGui.RollClient (player-visible texts only);
    - PowerServer, GoblinInventory (zone messages).
  - **Not in GameText, on purpose:**
    - content names stay with their content: goblin names / rarities in `GoblinConfig`; shrine / token / power names and chances in `PowerConfig`;
    - the dock labels are part of the dock icon image;
    - the gate screens' "ZONE n" / "UNLOCK" are built once by `ServerStorage.ZoneScreenBuilder`;
    - the Dutch Studio-only test panels and the unused old Settings / Teleport code in GoblinPetClient.
- Where:
  - Studio: `ReplicatedStorage.GameText` (local copy `Goblin_UI/GameText.luau`), `ReplicatedStorage.GoblinUI`, and the scripts above.
  - Local: `Goblin_UI/GoblinShrineClient.client.luau` is updated. The local copies of Teleport / Settings / Extras / PetClient / RollClient are now **behind Studio**: read from Studio before editing.
  - Tools: `Goblin_UI/replace_exact.py` (exact replacements with count checks); the same in Lua for Studio scripts.
- Test (Play):
  - Opened Inventory, Index, Settings, Teleport and Shrine via `OpenPanel`. Each shows its GameText texts (checked by value): Shrine "Upgrade Power" / hint, Teleport title + zone names, Settings rows + tabs, Inventory search / equip / Powers tab, Index "Discovered", EXTRAS, ROLL / Auto Roll.
  - A server message ("Too fast") came from GameText.
  - No errors.
- Rule from now on: new UI texts go into GameText, never hard-coded.

## 2026-10-09 — Shrine roll glitch fixed (spring vs tween) (Claude)
- What:
  - Symptom (the user's video, auto roll): from the 2nd roll on, the carousel jumped back and forth during the spin. Left of the middle stayed empty, and at the end it snapped to the result.
  - Measured per frame in Play: the position flipped between two states about 2 slots apart.
  - Cause: with deferred events, the previous roll tween's `Completed` (fired again by `:Cancel()`) arrived during the next roll. It set `spinning = false`, so the arrow spring and the roll tween fought over the position.
  - Fix: only the current tween may end the spin (`if slideTween ~= tw then return end`). A finished tween is cleared.
- Where: `StarterPlayerScripts.GoblinShrineClient` (`slideTo`) and the local copy.
- Test (Play): auto roll with 4 rolls in a row and a per-frame probe: 0 direction reversals, 0 cards hidden near the middle, no errors.
- Gotcha for later: with `SignalBehavior` Deferred, a tween's `Completed` can arrive late. Guard handlers that change shared state.

## 2026-10-09 — Upgrade Power: luck shown like the global luck (clover + "2x") (Claude)
- What:
  - In the Upgrade Power box, the plain "Luck x2" text is replaced by the style of the global luck above ROLL (RollClient):
    - the clover icon (`UI.Images.clover`), swaying ±8° just like the global one;
    - "2x" / "1.375x" in Montserrat Heavy with the same green gradient (190,255,150 → 70,200,80) and a black outline.
  - The row is centred and pops (scale 1.15 → 1) when the value changes (another power, a star more).
- Where: `StarterPlayerScripts.GoblinShrineClient` (`luckRow`, `luckWord`) and the local copy.
- Test (Play): Heartwood selected → clover + "2x" under the stars (capture); no errors.

## 2026-10-09 — Shrine carousel stutter really fixed: fixed cards + spring (Claude)
- What:
  - **The real cause**, seen frame by frame in the user's 60 fps video: the 5 pooled cards swapped their content whenever the carousel passed a card. The text changed at once, but Roblox showed the new icon one frame later, so for one frame names and icons didn't match (e.g. "World Tree" with the leaf icon). Preloading did not help.
  - **Fix 1:** one card per power with fixed content. Only position, size, fade and glow change, and a card is hidden beyond 2.2 slots. Shrines with fewer than 5 powers get copies, so the wrap-around always happens out of view. The cards are rebuilt (`buildCards`) when the shrine changes.
  - **Fix 2:** the arrows and card clicks no longer restart a tween. A critically damped spring (ω 15) chases `targetPos`, and fast clicks just move the target further. Rolls still use the 2.4 s Quint tween (`spinning`).
  - `fitText` remembers measured sizes, so the Upgrade name doesn't pop.
  - The arrows are named `LeftArrow` / `RightArrow`.
- Where: `StarterPlayerScripts.GoblinShrineClient` (and the local copy).
- Test (Play): a probe sampled every frame while the right arrow was clicked 4 times.
  - 0 name / icon mismatches.
  - The movement per frame ramps up and down smoothly (10, 15, 24 … 2, 1 px) without jumps.
  - No errors.

## 2026-10-09 — Shrine carousel: no icon blink when sliding far (Claude)
- What:
  - The cause: the 5 pooled carousel cards swap their `Image` while they slide. An icon that wasn't loaded yet blinked for a frame, and you saw that as stutter when clicking a far card (the user's video).
  - The fix: all power icons are preloaded when the script starts (`ContentProvider:PreloadAsync`).
- Where: `StarterPlayerScripts.GoblinShrineClient` (and the local copy).
- Test: compile only. The user should check in Play.
- Still open: if it still stutters, the next suspect is the Upgrade tile refreshing on every focus change while sliding.

## 2026-10-09 — Enemies always outlined (Claude)
- What: the black outline that enemies only had on hover is now always on (the user).
- Where:
  - `StarterPlayerScripts.EnemyAnimator`: the `HitFlash` Highlight now has `OutlineTransparency` 0 and `OutlineColor` black.
  - `StarterPlayerScripts.GoblinCombatClient`: `outline()` no longer switches the outline off on hover-leave.
- Test (Play): all 3 enemies had `HitFlash.OutlineTransparency` 0; no errors. Not checked visually: the capture camera snapped back to the player.
- Still open:
  - Check in-game how the outline looks while a dead enemy fades out and respawns.
  - The Highlight budget is 31 per client; it is fine for 3 enemies.

## 2026-10-09 — Shrine panel: the user's original art (sharp icons, stars, arrows, X) (Claude)
- What: the user sent the original art: 12 images, 1254 × 1254, already transparent.
  - What they are: the 6 power icons (Leaf Whisper, Rootbound, Forest's Blessing, Ancient Canopy, Heartwood, World Tree), the shrine icon, the gold and grey star, the left and right arrow, and the red X button.
  - Each is trimmed to its visible pixels and scaled to at most 512 px with a premultiplied area filter, so there are no fringes (`Goblin_UI/prep_icons.py`). They are uploaded and used everywhere instead of the cut-outs and my drawn stars / arrows.
  - **Same visual sizes as the reference:** `PowerConfig` `iconSize` was set so each icon covers the same area as before (e.g. Leaf Whisper 79 × 97, World Tree 125 × 114).
    - Tall icons on the Possible cards move up so they end above the name.
    - Stars are 31 × 29 every 33.6 px, like the reference.
    - The arrows are 52 × 64 (their own aspect) at the reference's centres.
    - The X is 82 × 81 at the reference's centre.
  - The Powers inventory tab picks up the new icons automatically (from `PowerConfig`).
- Where:
  - Originals: `Goblin_UI/extras/originals/src_*.webp`.
  - Prepared: `Goblin_UI/extras/orig_*.png`.
  - Asset ids:
    - Leaf 139406196839416, Rootbound 86245783099105, Blessing 112438678288394, Canopy 117698258720072, Heartwood 88633765795522, World Tree 140567900988383;
    - shrine 107831628409401;
    - stars 116015407274597 / 97670933684306;
    - arrows 92601240020507 / 76722949568799;
    - X 116636458058164.
  - Studio: `ReplicatedStorage.PowerConfig`, `StarterPlayerScripts.GoblinShrineClient`. Local copy updated.
- Test (Play): captured with the reference's numbers and compared side by side. The art is sharp and the layout unchanged. A roll worked; no errors.
- Still open: nothing for the art. Balance is still placeholder.

## 2026-10-09 — Forest Shrine panel v3: really 1:1 after reference 160 (measured), new icon cut-outs, real stars; shrine sign smaller + fades out (Claude)
- What:
  - **Shrine panel rebuilt 1:1 after `images/160.webp`, measured instead of eyeballed.**
    - A pixel-scan script (`scratchpad measure.py`: colour runs along rows / columns, text extents / colours) gave every edge, size and colour.
    - Panel = 1242 × 995 reference px (origin = the panel's top-left at (116, 32) in the image) inside a UIScale of 0.64.
    - Borders are nested rounded frames like in the image: outline → rim → inner line → fill. UIStroke would draw outside the measured edges.
    - Montserrat sizes are chosen so each text's width / cap height matches the reference (`TextService` measurements).
    - Checked side by side with the reference at the same scale (ffmpeg hstack) and corrected until they lined up.
  - **The user's deviations kept:**
    - "Upgrade Power" / "Upgrade";
    - ROLL reads [token] ROLL 10, no 2nd token;
    - Montserrat;
    - the scroll arrow on the right of Possible Powers only shows with more than 6 powers (it pages then);
    - Luck shows the starred value (2★ Forest's Blessing = x1.375).
  - **Carousel:**
    - Cards morph with their distance to the middle: 284 × 276 in the middle → 197 × 231 at the sides, text size, icon size, glow, and a fade overlay (#cfdbe6, 58 %).
    - The luck line only shows in the middle.
  - **Icons:**
    - The 6 power icons and the shrine icon are cut out of reference 160 with transparency (`Goblin_UI/cut_power_icons.py`: smooth-background fit + flood fill + soft alpha). No more square tiles from the low-res first mock-up.
    - They are shown at their natural reference size (`PowerConfig` `iconSize`): ×1 on the Possible cards, ×1.72 in the middle of the carousel, ×1.2 at the sides, up to ×1.45 in the Upgrade tile.
  - **Stars:** drawn like the reference (`Goblin_UI/make_shrine_parts.py`): rounded gold stars with a dark outline, flat grey-blue empty ones. New carousel arrows in the reference proportions.
  - **Buttons:**
    - ROLL and Upgrade stay coloured like the reference; when you can't afford / upgrade, clicking explains why.
    - The hint stays "Duplicates upgrade stars".
  - **Shrine sign** (ShrineFX):
    - 1.2× smaller (25 × 5.17 studs);
    - fades out between 70 and 90 studs from the camera, hidden beyond (`MaxDistance` 90).
  - **New Studio test hook:** ServerStorage attribute `StudioPowerState` = JSON of a whole `data.powers` table (sets tokens / owned / favor / stars for every player).
- Where:
  - Studio:
    - `StarterPlayerScripts.GoblinShrineClient` (v2 backup `ServerStorage.UI_Backup.GoblinShrineClient_v2`);
    - `ReplicatedStorage.PowerConfig` (icon ids + `iconSize`, shrine icon 92625940045719);
    - `ServerScriptService.PowerServer` (`StudioPowerState`);
    - `StarterPlayerScripts.ShrineFX` (sign).
  - Local:
    - `Goblin_UI/GoblinShrineClient.client.luau`, `Goblin_UI/cut_power_icons.py`, `Goblin_UI/make_shrine_parts.py`;
    - `Goblin_UI/extras/power_v2_1..6.png`, `shrine_icon_v2.png`, `star_gold.png`, `star_grey.png`, `arrow_v3_l/r.png`.
  - Asset ids:
    - icons 133443869063105 / 113719039378060 / 112004932195479 / 104855120607468 / 122255539182150 / 97501802333923;
    - stars 75157906184952 / 116872512099434;
    - arrows 88448978762625 / 122969191454613.
- Test (Play):
  - With `StudioPowerState` = the reference's numbers (240 tokens, favor 35/50, Forest's Blessing 2★): side-by-side with the reference matched in layout, sizes and colours.
  - A roll worked (230 tokens, favor 36).
  - The sign fade was measured: 0 at 50 studs, 0.21 at 75, 0.7 at 85, 1 at 95.
  - No errors.
- Still open:
  - The icons are as sharp as the reference allows (about 100–150 px). Original PNGs from the user would be sharper.

## 2026-10-09 — Forest Shrine panel v2: 1:1 after reference 159, smooth carousel, Upgrade Power + stars (Claude)
- What:
  - **Panel rebuilt 1:1 after the user's 2nd reference** (`images/159.webp`), with the user's changes: "Active Power" → **"Upgrade Power"** and "Equip" → **"Upgrade"**.
    - The frame is built in the reference's own pixels (1237 × 995, origin = the panel's top-left in the image) and scaled by a UIScale (0.64) to about Inventory size.
    - Layout: header (shrine icon, title, token pill, X); full-width Shrine Favor bar + "Full favor: next roll Epic or higher"; "Roll a Forest Power" with the carousel, blue triangle arrows, ROLL [token] 10 and Auto Roll; "Upgrade Power"; "Possible Powers" as one row of 6 cards + "Roll for a random forest power!".
    - Left out because it made no sense with 6 powers: the scroll arrow on the right of the Possible Powers row.
  - **Smooth carousel:**
    - One continuous position (`posValue` NumberValue) and 5 pooled cards. Each card's x, scale (0.76 at the sides), fade and glow follow its distance to the middle.
    - The arrows and clicking a Possible card slide there (Quart Out). A roll spins 2 laps + the way to the result in 2.4 s (Quint Out), then the card pops with a glow and sparkles.
    - While a roll spins, the Upgrade box stays still.
  - **Stars (Upgrade):**
    - A power starts at 1★; duplicates upgrade it up to `STAR_MAX` 5, using `UPGRADE_COST` {2,3,5,8} spare copies (owned − 1).
    - Each extra star adds `STAR_LUCK` 0.5 × the power's own bonus (`PowerConfig.LuckFor(p, stars)`; `LuckOf` uses it).
    - The box shows the middle power: tile, name, rarity, 5 stars, luck at its stars, the Upgrade button (grey when it can't) and "Duplicates upgrade stars (have/need)".
  - The Powers tab shows the starred luck and "xN • n stars".
- Where:
  - Studio: `StarterPlayerScripts.GoblinShrineClient` (v1 backup `ServerStorage.UI_Backup.GoblinShrineClient_v1`, Disabled), `ReplicatedStorage.PowerConfig`, `ServerScriptService.PowerServer` (RemoteFunction `PowerRemotes.UpgradePower`, `data.powers.stars`, attribute `PowerStars`), `GoblinPowersInvClient`.
  - Local: `Goblin_UI/GoblinShrineClient.client.luau`, `Goblin_UI/GoblinPowersInvClient.client.luau`, `Goblin_UI/make_shrine_sign.py` (`triangle()` → `extras/arrow_blue_l/r.png`: 88727433205324 / 83907425791464).
- Test (Play):
  - The panel opened in the circle and was captured next to the reference.
  - Captured mid-spin: the cards slide smoothly.
  - It landed on the server's result; tokens / Favor / luck updated.
  - Upgrade on Leaf Whisper worked (stars filled, duplicates used).
  - No errors.
- Still open:
  - Upgrade costs and star luck are placeholders (balance at the end).
  - The power icons are still crops from the mock-up.

## 2026-10-09 — Coins no longer drop on the ground: "+n" popup like the Forest Tokens (Claude)
- What:
  - A kill now pays the enemy's coins at once (`Inventory.AddCoins`) to every player who helped.
  - The client shows a "+n" popup over the enemy's head: the chosen coin face (`CoinModel`), gold Montserrat text and the chosen pickup sound (`CoinSound`). It pops in, rises and fades, just like the token popup.
  - Popups are no longer AlwaysOnTop: AlwaysOnTop billboards did not appear in captures. The token popup sits 3 studs above the coin popup.
  - The old ground drops (magnet, flight to the counter, `CollectCoin`) are kept in the code behind `COINS_ON_GROUND = false` in CombatServer.
- Where:
  - `ServerScriptService.CombatServer`: `dropCoins` + `COINS_ON_GROUND`, event `"coinpop"`; `EnemyKilled` pos is now over the head.
  - `StarterPlayerScripts.CoinDropClient`: `coinPopup`.
  - `GoblinShrineClient`: token popup lifted, not on top (local copy updated).
- Test (Play): the Studio "drop" at the Moss Beetle showed "+25" over its head, coins +25, nothing on the ground (capture). The token popup also works. No errors.
- Still open: the Studio coin test panel's Drop-style row now only matters with `COINS_ON_GROUND = true`.

## 2026-10-09 — Powers live: Forest Shrine panel, tokens, Powers tab, shrine sign, zone renumbering, goblin rings on dirt paths (Claude)
- What:
  - **Goblin rings on the dirt paths:** the 28 `V4_*_Path_NoCollide` meshes are now goblin ground: CanQuery, PreciseConvexDecomposition, attribute `GoblinGround`; CanCollide stays off. Before this, the ring sat on the terrain about 0.2 studs under the path surface (e.g. DarkForest path 150.315 vs terrain 150.1).
  - **Zone renumbering (the user):**
    - Fernwood (internal zone 2) is now "Zone 1" and free: `GoblinInventory.publish` always sets `data.zones["2"]`, so gate 1→2 opens for everyone.
    - Internal zone 1 is the Spawn: no mobs, and Teleport shows "Spawn • No mobs".
    - Shown number = internal − 1: gate screen titles (`V4_Gate_*_Screen.Screen.Title`, and `ZoneScreenBuilder`), Teleport subtexts, the "Zone n unlocked!" message.
    - Internal numbers (attributes, `Zones`, `ZoneUnlock`) are unchanged.
  - **Shrine sign** (ShrineFX): a BillboardGui above the shrine that always faces the camera: gold ‹, the Forest Token, "Forest Shrine" (Montserrat Heavy, cream gradient, dark-green outline + shadow), gold ›. The outline thickness follows the billboard size.
  - **Power system:** tokens drop from mobs, shrine rolls, Shrine Favor, owned powers saved, luck bonus. Details in section 1 "Powers / Goblin Shrines".
  - **Forest Shrine panel** after the user's layout, with the user's changes: square Possible Powers cards, no scrollbar, no "Next guarantee" text.
  - **Powers tab** in the Inventory, with the user's icon.
- Where:
  - Studio:
    - `ReplicatedStorage.PowerConfig`;
    - `ServerScriptService.PowerServer`;
    - `StarterPlayerScripts.GoblinShrineClient`, `GoblinPowersInvClient`;
    - edits in `GoblinInventory` (PowerConfig luck, `Inventory.Publish`, zone "2" free), `CombatServer` (`ServerStorage.EnemyKilled` BindableEvent), `GoblinPetClient` (Powers tab entry + `OpenPanel` attribute hook; now ~198.9k chars), `GoblinTeleportClient`, `ShrineFX`, `ZoneScreenBuilder`;
    - `Shrine_Fernwood` attributes `ShrineTitle` / `ShrineToken` / `ShrineZone`.
  - Local:
    - `Goblin_UI/GoblinShrineClient.client.luau`, `Goblin_UI/GoblinPowersInvClient.client.luau`;
    - `Goblin_UI/make_shrine_sign.py` (token cut-out + chevrons), `Goblin_UI/cut_powers_tab.py`;
    - `Goblin_UI/extras/forest_token.png`, `chevron_l/r.png`, `power_1..6.png`, `powers_tab.png`, `shrine_icon.png` (ids in `asset_ids.json`).
- Test (Play):
  - Walking into the shrine circle opened the panel.
  - ROLL 10 spun the carousel and landed on the result: tokens went down by 10, Favor +1, Luck went up (Rootbound → 1.1, Forest's Blessing → 1.25).
  - The Powers tab showed found powers, ACTIVE on the best one, and the rest as "???".
  - `EnemyKilled` for the golem gave +11 tokens and showed the popup.
  - The sign was captured over the shrine.
  - No errors.
  - The goblin rings on the path were checked with raycasts only (the test profile had no goblins).
- Still open:
  - What the user wants in place of "Next guarantee: Epic+" (their message was cut off). The Favor bar now uses that space.
  - The original PNGs of the 6 power icons: the current ones are crops from the mock-up and a bit soft.
  - Balance: cost 10, chances, luck values, token drops (all placeholders).
  - Stars for duplicates.

## 2026-10-09 — Studio fly mode + shrine floating pieces (Claude)
- What:
  - New `StarterPlayerScripts.StudioFly` (Studio only, does nothing in the real game). F toggles flying. WASD moves where the camera looks, Space goes up, Ctrl down, Shift fast (LinearVelocity + AlignOrientation, PlatformStand).
  - The shrine is re-exported as `Shrines/Shrine_Fernwood.fbx` (static) + `Shrines/Shrine_Fernwood_Floating.fbx` (11 separate pieces).
  - `ShrineFX` animates the floating pieces:
    - tablets orbit round the crystal and bob;
    - shards orbit the other way and spin;
    - the charm swings.
  - `ShrineFX` also sets Neon on all glow parts itself.
- Still open: the user re-imports both FBX files (model name `Shrine_Fernwood`); then a Play check.

## 2026-10-09 — Fernwood Shrine in Studio: Neon, floating animation, step-in circle (Claude)
- What:
  - The user imported `Blend/Shrines/Shrine_Fernwood.fbx`. Claude exported it with one palette texture (`Shrines/Shrine_Palette.png`) after the user's own edits in Blender.
  - In Studio the `*_Glow` parts are Neon: crystal green, core white-green, runes, orbs. They don't collide and cast no shadows.
  - New `StarterPlayerScripts.ShrineFX`:
    - the crystal bobs;
    - its core turns and pulses;
    - the orbs drift.
  - A plain glowing ring on the platform that turns slowly.
    - `ShrineCircle` (Transparency 0.995: a SurfaceGui is not drawn on a part at 1) has an unlit SurfaceGui with our own ring image `rbxassetid://109399029198645` (`Goblin_UI/extras/shrine_ring.png`), just round the green rune ring.
    - An invisible `ShrineTrigger` (attribute `ShrineZone` = 2) sits under it. Walking in will open the shrine UI later.
- Where: `Workspace.Shrine_Fernwood`, `StarterPlayerScripts.ShrineFX`.
- Still open:
  - The parts came in merged per material, so the big crystal, the shards and the emeralds on the arch move as one piece, and the rune tablets (inside the dark stone part) can't move. For spinning / orbiting pieces, a re-export with separate floating parts is needed.
  - The UI behind `ShrineTrigger` (the power system).

## 2026-10-09 — Power system planned (tokens per zone, shrines), Fernwood Shrine designed in Blender (Claude)
- What:
  - Talked through the user's power system idea, with a proposal for how to make it our own (section 1, "Powers / Goblin Shrines").
  - The user's choices: one token kind per zone; powers give luck now and other bonuses later; balance at the end.
  - Designed the Fernwood Shrine in the user's new Blender file: low-poly, no goblin head, stone + moss + gold + a floating emerald.
- Where:
  - `Blend/Shrines.blend` (saved; collections `Shrine_Fernwood` + `Shrine_Preview`).
  - Generator `Blend/Shrines/build_shrine.py`.
- Test: looked at it from 4 sides and through the preview camera (EEVEE renders). Arch moss, normals (all outward) and the scale against a player marker are checked.
- Still open:
  - The user's feedback on the look.
  - Then the export to Roblox: an FBX import plus Neon / glow for `*_Glow`, and a spot in zone 2.
  - The power system itself (tokens, rolls, powers) still has to be built. The user's token and UI come later.

## 2026-10-09 — 5 coin models, 5 pickup sounds, 5 drop animations, outlines, magnet + attack range ×2, KO → full health (Claude)
- What:
  - After a knock-out a goblin stands up with full HP.
  - Coins:
    - 5 coin models: Classic, Star, Goblin Token, Gem with a 3D emerald, Crown (new faces drawn in the house style).
    - 5 pickup sounds.
    - 5 drop animations, each with its own effect: Fountain, Rain, Skip, and new Burst and Vortex.
    - A dark outline on every coin (one Highlight for all).
    - Pickup range ×2 (32).
  - Goblins run to enemies from ×2 as far (`ENGAGE_RANGE` 52, `CLICK_RANGE` 110).
  - The Studio test panel now has three rows (Coin / Sound / Drop × 5) to choose from.
- Where:
  - Studio: `ReplicatedStorage.CombatConfig`, `StarterPlayerScripts.CoinDropClient` (backup `UI_Backup.CoinDropClient_v1`).
  - Local: `Combat/CoinDropClient.client.luau`, `Goblin_UI/make_coin_icons.py`, `Goblin_UI/extras/coinface_*.png`.
  - Coin face asset ids: star 115077615109126, goblin 109344164176722, gem 109846453483051, crown 124182894574122.
- Test (Play):
  - 5 drops (one per model, Burst) at the beetle: all models stood round it with outlines (capture).
  - Vortex ran without errors.
  - Walking up to 30 studs pulled the coins in (+100 coins).
  - The 5 sounds were tested to load. No errors in the output.
- Still open: the user picks a coin model, sound and drop; those become the defaults in `CombatConfig` and the test panel can go.

## 2026-10-09 — Goblins take damage, enemies hit back, coin drops with 3 animations + magnet, HP bar under the name tag (Claude)
- What:
  - Goblin stats (`GoblinPets.StatsOf`, per rarity × mutations).
  - Goblin health with knock-outs and healing.
  - Enemy blows do damage, timed like their animations.
  - Kills drop coins round the enemy (3 animations to choose from in the Studio test panel). A magnet pulls them to the player, they fly into the coin counter, and then the server pays.
  - An HP bar under each own goblin's name plate, only during a fight, after the user's reference: green → yellow ≤ 50 % → red ≤ 20 %, "140/200".
  - Details in section 1, Enemies and combat.
- Where:
  - Studio:
    - `ReplicatedStorage`: `CombatConfig` (stats, hits, coins, KO / regen), `GoblinPets` (`StatsOf`), `GoblinTagFX` (`SetHealth`, Rescale keeps the lift), `GoblinPetRenderer` (`combatRing` skips `pet.down`);
    - `ServerScriptService.CombatServer` (health, `enemyHit`, `dropCoins`, `CollectCoin`, StudioCombatTest);
    - `StarterPlayerScripts`: `GoblinCombatClient` (damage per slot), `EnemyAnimator` (no coin particles), new `GoblinHealthClient`, new `CoinDropClient`.
  - Backups in `UI_Backup`: `CombatConfig_before_damage_coins`, `GoblinPets_before_stats`, `CombatServer_before_damage_coins`, `GoblinTagFX_before_hpbar`, `GoblinPetRenderer_before_ko`, `GoblinCombatClient_before_damage`, `EnemyAnimator_before_coins`.
  - Local: `Combat/GoblinHealthClient.client.luau`, `Combat/CoinDropClient.client.luau` (the other `Combat/` copies are behind Studio now).
- Test (Play, 6 goblins of Rare / Uncommon):
  - 21 damage/s; the beetle dies in ~5 s; its bites take 6 HP; the goblins heal after the fight.
  - Coins came in per kill (25 each). The user played along and fought the boar: goblins were knocked out (dizzy stars, see-through, walking back) and $1,600 came in.
  - A test drop out of range stayed lying (19 coins). Walking up collected 10 coins = +50, the rest stayed at the boar.
  - HP bars showed under the plates in the fight ("60/60"); made bigger after the first capture. No errors in the output.
- Still open:
  - **The user picks a drop animation (1 / 2 / 3)**; that becomes `COIN_DROP_STYLE` and the test panel can go.
  - The coin amounts and the stats are first values to tune.
  - Measure render cost with many coins + 6 HP bars when the look is final.

## 2026-10-09 — Shiny light really minimal + a light budget for the goblins (Claude)
- What:
  - The user: Shiny goblins still give far too much light.
  - Measured on 6 Rainbow Huge Shiny Celestials (`Eclipse` / `GalaxyWalker`): they carried 24 PointLights.
    - Celestial aura: 6 × 1.2, range 11 (Huge makes it bigger).
    - Back / core personality lights: 0.35–0.7, range 8–10.
    - Rainbow: 6 × 0.42, range 8.
    - Shiny: 6 × 0.12.
    - Switching groups off one by one showed the white flood came from that light stack; the glint at its peak only lightens them a little.
  - **Shiny (`GoblinMutationFX`):**
    - no PointLight;
    - glitter without glow (LightEmission 0, 3/s, smaller);
    - glint every ~3.6 s with peak 0.93;
    - ting star smaller with LightEmission 0.1; ting / footstep sparkles LightEmission 0.15 and fewer;
    - equip flash without Neon and without light, with fewer and dimmer sparkles.
  - **Light budget:** new `StarterPlayerScripts.GoblinPetLightBudget`. Per player only 2 goblins keep their PointLights.
- Where:
  - Studio: `ReplicatedStorage.GoblinMutationFX`, `StarterPlayerScripts.GoblinPetLightBudget` (new).
  - Backup: `ServerStorage.UI_Backup.GoblinMutationFX_before_minimal_shiny`.
  - Local: `Goblin_Pets/GoblinPetLightBudget.client.luau`.
- Test: in Play, the same 6 goblins with the same camera.
  - Before: a white-yellow blob on the ground and the bodies.
  - After: 6 lights on, 12 off, no ShinyLight; the ground under them is normal and the goblins keep their colours (captures).
- Still open: if it should be even calmer, `LIGHT_PETS` can go to 1, or the rarity aura light (GoblinPetFX, Celestial `light = 1.2`) can go down.

## 2026-10-09 — Settings / Teleport as real menu panels, zone icons, enemy tier stars, enemies moved to Fernwood Forest, enemies walk on the floor (Claude)
- What:
  - **Settings + Teleport panels follow the house panel rules** (the user: too big next to Inventory / Index, both could be open at once, no particles behind them):
    - They are now panels of GoblinPetClient's system ("external panels", see section 1 Settings). That gives the same scale as the Inventory, one panel at a time, the dim, the halo of stars, the dock highlight and X / dock / Esc.
    - Smaller: Settings 600 × 540 and Teleport 780 × 540 Stage px. On a 1811 × 668 screen they are 478 × 432 and 618 × 432 px, against Inventory 671 × 489 and Index 748 × 461.
    - Slim switches and sliders, shorter sub lines.
  - **Teleport zone icons:** 10 painted icons instead of map screenshots.
  - **Enemy tags:** gold tier stars above the name (1 / 2 / 3), the rest as before.
  - **Enemies moved** from the old map (under Spawn) to zone 2 Fernwood Forest in the V4 map, upright, facing the way players come in.
  - **Enemies walk on the floor like the player:** CombatServer only uses CanCollide floor, from one step above the feet, and keeps the placed foot height.
- Where:
  - Studio:
    - `StarterPlayerScripts.GoblinPetClient` (external panel block; the Settings / Teleport intercepts in `open()` removed);
    - `GoblinSettingsClient`, `GoblinTeleportClient`;
    - `ServerStorage.EnemyTagBuilder`;
    - `ReplicatedStorage.EnemyModels.*` and `Workspace.EnemyPreview.*` (tags; positions);
    - `ServerScriptService.CombatServer` (`groundY`, `footLift`).
  - Backups in `ServerStorage.UI_Backup`: `GoblinPetClient_before_ext_panels`, `GoblinSettingsClient_before_v3`, `GoblinTeleportClient_before_v2`, `EnemyTagBuilder_before_stars`, `CombatServer_before_floor_walk`.
  - Local: `Goblin_UI/GoblinSettingsClient.client.luau`, `Goblin_UI/GoblinTeleportClient.client.luau`, `Goblin_UI/make_zone_icons.py`, `Goblin_UI/make_extras_icons.py` (`tier_star`). The local `Combat/CombatServer` copy is now behind Studio.
- Test (Play):
  - Settings → Teleport → Inventory → Index: only one is visible each time, `PanelOpen` follows, and the halo shows (captures).
  - The 3 enemies wander at their new homes. Over 15 s of walking their feet stayed exactly 0.26 above the floor (the placed height). The beetle's tag shows 1 star in-game.
- Still open:
  - The fight test needs a profile with goblins (the Studio memory profile owns none; use the test panel "Alle pets").
  - Enemy spawning / more enemies per zone is not built (only the 3 preview enemies).

## 2026-10-09 — Goblins + ring stand on path tiles everywhere, Settings v2, Teleport menu, zone names (Claude)
- What:
  - **Goblin ground fix (all zones and links, not only Heaven).**
    - Cause: all V4 path tiles (`*_Pavers_NoCollide`) and stepping stones (`*_PathStones_NoCollide`) were CanQuery = false. "Stone" was also in the renderer's skip names. So goblins, and the ring under them, stood on the terrain 0.6–1 stud under the tiles: inside them.
    - Fix: the 42 parts are now CanQuery + PreciseConvexDecomposition + attribute `GoblinGround`. Pavers are also CanCollide, so the player walks on the tiles too; the stones are not. `GoblinPetRenderer.skipPart` never skips `GoblinGround` parts, and parts that stream in later are added to the skip list.
    - Enemies (`CombatServer.groundY`) now also stand on the tiles.
  - **Settings v2 applied:**
    - smaller panel;
    - aligned tabs;
    - sliders no longer cross the text;
    - a second dock click closes it;
    - UI Scale keeps the scroll position.
  - **Teleport menu installed.** Long names shrink to fit beside the GO/LOCKED button; the sub line shows "Zone n • 1 Coin" for the next zone.
  - **Gate screens renamed** to the user's zone names and rebuilt for zones 2–10.
- Where:
  - Studio: `ReplicatedStorage.GoblinPetRenderer` (`skipPart`, streamed skip list); the 42 map parts in `MR_Versie_4_Mix_A_zones1-5` / `_B_zones6-10`; `StarterPlayerScripts.GoblinSettingsClient`, `.GoblinTeleportClient` (new), `.GoblinPetClient` (`open()`: `OpenTeleport`); `V4_Sign_Zone02..10` attribute `ZoneName`.
  - Backups in `ServerStorage.UI_Backup`: `GoblinPetRenderer_before_ground_tiles`, `GoblinSettingsClient_before_v2`, `GoblinPetClient_before_teleport`.
  - Local: `Goblin_UI/GoblinSettingsClient.client.luau`, `Goblin_UI/GoblinTeleportClient.client.luau`.
- Test:
  - Play-tested on `V4_Link09B_Pavers` (the way into Heaven): paver top 150.71, player feet 150.71, goblins on the tiles 150.76; goblins beside the tiles stand on the ground (150.10).
  - The Teleport and Settings panels open without errors (captures checked).
- Still open:
  - The user should check other zones by eye.
  - Stepping stones make goblins step up and down (0.9 studs); if that looks jumpy, drop `GoblinGround` from the `PathStones`.
  - "Abandoned Graveyard" shrinks to 13 px in its card.

## 2026-10-09 — Enemy name / HP tag in the reference style, EXTRAS button + list, Bingo UI removed, zone screens without shadows (Claude)
- What:
  - **Enemy tag** like the user's reference:
    - name in white with a dark outline;
    - under it a thin white line that fades out to the left and right (solid white in the middle, as asked);
    - a pill HP bar (white rim, dark edge that follows the HP colour, green fill, "600 / 600 HP");
    - a white pointer below.
    - The plate, icon, stars and boss aura are gone.
    - Built by `ServerStorage.EnemyTagBuilder` on the 3 templates + 3 preview enemies; `EnemyNametagFX` tints the edge.
  - **EXTRAS** left of ROLL (section 1, EXTRAS):
    - orange button + pop-up list with Profile Stats, Bingo, Achievements, Codes, Mastery, Playtime Rewards;
    - 4 rows show, the rest scroll with an orange scrollbar;
    - the user's 3 icons + 3 drawn in the same style.
    - After the user's feedback:
      - the button is smaller (68 px) and lower;
      - the list sits right on the button (it hung 58 px too high: the top bar inset);
      - a cleaner tail image;
      - the button hides during a full-screen roll (not with Hide Rolls on).
    - All rows say "coming soon" for now.
  - **Bingo UI removed** (the dock icon + the card): `GoblinBingoClient` deleted, backup in `UI_Backup`. The server side still counts.
  - **Zone gate screens** (section 1, gate screens):
    - no shadows: an unlit screen part in front of each of the 9 dark walls (`V4_Gate_<a>_<b>_Screen`), a bit blacker as asked;
    - restyled after the user's reference in Montserrat: padlock, "ZONE n", the name, coin + price, and a green UNLOCK button;
    - every new zone costs 1 coin for now;
    - `ZoneGateClient` hides the screens with the gate, animates the drawn button and writes the price.
- Where:
  - Studio:
    - scripts: `StarterPlayerScripts.GoblinExtrasClient` (new), `ZoneGateClient`, `EnemyNametagFX`;
    - `ServerStorage.EnemyTagBuilder`;
    - `ReplicatedStorage.EnemyModels.*.Hitbox.HPBar`, `Workspace.EnemyPreview`;
    - `Workspace.V4_ZoneGates` (screens, signs, buttons), `ServerStorage.ZoneScreenBuilder`.
  - Backups:
    - `ServerStorage.EnemyTag_Backup_20261009`, `ServerStorage.ZoneScreen_Backup_20261009`;
    - in `UI_Backup`: `GoblinBingoClient_removed_20261009`, `ZoneGateClient_before_noshadow_20261009`.
  - Local:
    - `Goblin_UI/make_extras_icons.py`, `Goblin_UI/extras/*.png` + `asset_ids.json`;
    - `Goblin_UI/GoblinExtrasClient.client.luau`.
- Test (Play):
  - Enemy tags look like the reference (captures).
  - EXTRAS:
    - the button sits left of ROLL;
    - the list opens (Studio hook `StudioOpen`) 3 px above the button;
    - scrolled to the end, the thumb reaches the bottom;
    - with `Spotlight` the button hides and the list closes, and it comes back after;
    - the Bingo icon is gone from the dock and `GoblinBingoGui` no longer exists.
  - Zone screens:
    - 9 placed (112 × 41 studs) and 9 signs rebuilt;
    - a Play capture showed the shadows gone;
    - the colour was calibrated in 2 rounds against a capture of the lit wall (within a few shades), then made blacker as asked.
    - The final look of the new sign + button could not be captured (the Studio viewport rendered black), so it is still to check by eye.
- Still open:
  - Check the new gate screen by eye: padlock / ZONE n / name / 1 Coin / UNLOCK, the text in front of the screen from every angle, hover + click on UNLOCK.
  - Build Profile Stats / Achievements / Codes / Mastery / Playtime Rewards (and decide whether Bingo comes back).
  - The bingo server still counts progress without a UI.

## 2026-10-09 — Floating Heaven (clouds, pillars, trees, angels on clouds), link 9 Heaven after the gate, pack 5, sprint, Shiny Rainbow odds, roll outline, coin shadow (Claude)
- What:
  - **Roll outline** round the cycling goblin icons: 2.2 px (was 3).
  - **Odds with Shiny Rainbow / Huge Shiny Rainbow:** the Shiny reveal no longer jumps to the final odds. It counts to the odds of what has been shown (Shiny, or Huge + Shiny); the Rainbow reveal then counts on to the final odds.
  - **Coin shadow:** starts in a lighter black (40, 40, 47) and ends in the same place. Its bottom is now clipped straight, so the black "$" outline no longer bulges out under the dollar sign.
  - **Sprint:** walk 2 s → 1.2× speed (`SprintMultiplier` for later upgrades), FOV +7, dust + streak at the feet (see section 1, Sprint).
  - **Blender:** link 9 is Heaven style from 6 % after the gate (`SPLIT`); 2 angels built (`mr_angels`); the user's edits (moved pillars, the temple sun gate vines) taken over. In Studio the 12 sun gate vine pieces got the Blender delta (attribute `Pack5Move`).
  - **Pack 5** (`mr_pack5`, the user imported `V4_AnimPack5.fbx`): link 9 rebuilt, the Heaven groups split into 89 clouds + 38 pillars + trees, the angels. Swap: 116 old parts to `ServerStorage.V4_Backup_Pack5_20261009`, link 9 grass / flowers 39 old → 46 new clones, glow points 16 out / 8 in (530 total). The import came in at 5.1× scale; the swap's marker alignment fixed it (error 0.000).
  - **Floating Heaven** (V4WorldFX v12 + `ServerStorage.V4FX_HoverGroups` + `mr_pack5.hover_data`):
    - clouds, pillars, the 8 old columns, the angels, and the 7 Heaven trees that do not stand on the ground bob, each group on its own beat;
    - pieces that touch move as one; pieces in the ground only sink a bit and come back;
    - hover FX only on pieces that really float;
    - none of them collide.
    - Details in section 1 (V4 world FX, v12).
  - **Angels on random clouds** (the user, mid-prompt): 01 (trumpet) on `V4_Heaven_HoverCloud_5074` in the arena, 02 (harp cherub) on `V4_Link09B_HoverCloud_5005` near the Heaven entrance; both face the nearest path point and bob with their cloud.
  - **Heaven look right after the gate** (`BLEND["Swamp|Heaven"]` in V4WorldFX).
- Where:
  - Studio: `StarterPlayerScripts.V4WorldFX` v12 (data PNG `rbxassetid://107856125633404`, magic WFXC; local copy `Map_Restyle/studio/V4WorldFX.client.luau`), `StarterPlayerScripts.SprintClient` (new), `StarterGui.RollGui.RollClient`, `StarterPlayerScripts.GoblinPetClient`, `ServerStorage.V4FX_Pack5Swap`, `ServerStorage.V4FX_HoverGroups`, `ReplicatedStorage.V4WorldLights`.
  - Local: `Map_Restyle/mr_pack5.py`, `mr_angels.py`, `mr_v4_layout.py` (`SPLIT`); `Map_Export/V4_AnimPack5.fbx`; `Map_Export/Data/pack5_*.json` (foliage, lights, floating, hover).
  - Blender: `GoblinRNG_MAP.blend` (saved; backup `Downloads/GoblinRNG_MAP_backup_20261009_voor_link9.blend`).
  - Blender base: `Map_Export/Data/snapshot_base_20261009_pack5.json`.
- Test (Play):
  - Sprint: WalkSpeed 16 → 19.2 after 2 s, FOV 70 → 77, dust + streak on; stop → 16 / 70, effects off.
  - Test roll "Shiny,Rainbow": the Shiny reveal counts to 1/12,796; the Rainbow reveal then counts to 1/63M+.
  - Heaven:
    - 130 of 137 pieces moved (7 still, as planned at that point);
    - after the tree / angel update: the angels and the trees keep their height on their cloud (relative spread 0.000–0.003 studs while the cloud moved 2–3.6 studs);
    - the free tree floats with FX.
  - Coin `Shadow` frame in place; capture looks straight.
  - Hover FX cost: +0.08 ms render CPU, +0.14 ms GPU. No script errors.
- Still open:
  - 3 clouds stay still: they carry glow ornaments that sit inside the merged `V4_Heaven_RimCover` / `ElementTop` parts. Splitting those ornaments out would take one more small import.
  - Angel 01 is ~5 m (3.5× a player); scale it in Blender if it should be smaller.
  - After new Blender edits in Heaven, run `mr_pack5.hover_data()` again and re-run `V4FX_HoverGroups` with the new data.

## 2026-10-09 — V4: the user's Blender edits into Studio, path gaps fixed, Ice path, pack 4 (hill trees, bushes, cliff vines) in (Claude)
- What:
  - **The user's Blender edits** (found with `mr_sync.diff`, plus a diff against `GoblinRNG_MAP_backup_before_animpack_20261008.blend`, the state that was imported into Studio).
    - Moves of single objects were applied in Studio directly (Blender delta → Studio affine; the parts carry attribute `Pack4Move`):
      - DarkForest bridge (0.44 lower), pond (water, shore, foam + its 3 lily clones in `V4_Foliage`) and waterfall (+ its spring / overhang rocks in `V4_Props.WaterfallSources`);
      - the frozen lake;
      - the Plains bench (turned 10°), CliffTree_32;
      - the windmill (turned 38°, moved): body, rotor (attribute `SpinAxis` turned along) and the `V4_Props.WindmillLights` windows.
    - `V4_Plains_EdgeTree_01_Trunk` (deleted in Blender) went to the backup.
    - Moved members of merged groups (DarkForest PondRocks, DarkForest WallElements, Link02A RimCover / WallElements) come with pack 4.
  - **Gaps where paths meet:** the cause was path / pond meshes whose end-cap triangles face down; Roblox culls back faces, so the grass showed through as V-notches.
    - Fix: `DoubleSided = true` on 65 flat parts (all `_Path_NoCollide`, pond water / shores, lakes, the stream; attribute `DoubleSidedFix`).
    - Note: my earlier "Studio paths are older" finding was wrong (it came from bbox math on rotated objects). Blender and Studio paths match.
  - **Link paths in Blender:** the straight A/B cut of pack 2 was missing in the .blend. It was re-run (`fix_link_paths` + `straight_link_paths`), and all 18 now match Studio exactly.
  - **Ice path** (the user moved the lake), in Blender:
    - `mr_pack4.fix_ice_path_gap` bridged the hole in `V4_Ice_Path_NoCollide` (+8 faces, the 12 down-facing faces flipped) and laid 9 slabs in `V4_Ice_Pavers_NoCollide`;
    - the uncovered rim of the old lake basin in `V4_Ice_Terrain` (80 verts) was raised to the ground; sunken slabs were lifted.
  - **"Flying green things" at the temple sun gate:** `V4_Temple_SunGate_01_Vines_NoCollide` was swung as one block (kind Hang), so its far vines left the stone. It is no longer swung (V4WorldFX v11); pack 4 brings it as single vines.
  - **Pack 4** (`Map_Restyle/mr_pack4.py` `build` / `export_ab`):
    - 862 hill trees, 174 walkable bushes (EdgeGreen / LinkGreen; the zones without separate bushes), 505 cliff-vine pieces (2–10 strands + their leaves; 127 cliffs / walls without their vines);
    - 47 rebuilt group pieces, plus Ice path / pavers / terrain;
    - files `Map_Export/V4_AnimPack4_A.fbx` (zones 1–5, 25 MB) and `_B.fbx` (zones 6–10, 29 MB), no textures, markers `AP_Marker_*_P4A/B`.
  - **Swap module:** `ServerStorage.V4FX_Pack4Swap`, run as `require(...)(workspace.V4_AnimPack4_A)`.
    - It aligns with the markers and renames the pieces to `V4_<Zone>_HillTree_<n>` / `Bush<500+n>` / `CliffVine_<n>` `_NoCollide`.
    - It replaces the old group / cliff parts (texture, collision flags, attributes and children copied over) and puts the old parts in `ServerStorage.V4_Backup_Pack4_20261009`.
    - All 1718 names were checked against Studio: every source was found.
  - Studio crashed once during a test of that module with empty MeshParts. Nothing was lost (Team Create). Lesson: do not test with empty MeshParts.
- Where: `Map_Restyle/mr_pack4.py`, `mr_sync.py`; `Map_Export/Data/` (snapshots, name list); Blender `GoblinRNG_MAP.blend` (saved; backups `…_voor_jouw_edits` and `…_na_jouw_edits`); Studio `V4WorldFX` v11 (data PNG `rbxassetid://94511261134415`), `ServerStorage.V4FX_Pack4Swap`.
- Test:
  - Top-down Studio capture of the J3 junction before / after DoubleSided: the notch is gone.
  - Bbox comparison Blender ↔ Studio: only the expected differences are left.
  - Blender renders of the Ice path.
  - The name check of pack 4.
- **Import + swap (done the same day):**
  - The user imported both files. In Studio there are now:
    - 862 `V4_<Zone>_HillTree_<n>_NoCollide`, 174 new bushes (`Bush501`–`Bush674`), 505 `CliffVine` pieces;
    - 127 cliffs / pillars / arches without vines, 47 rebuilt group pieces, plus the Ice path / pavers / terrain.
    - The 208 old parts are in `ServerStorage.V4_Backup_Pack4_20261009`.
  - The first run (A) used a cached old module (`require` caches; my parse fix was not loaded).
    - The C pieces were not placed, so their cliffs went to the backup. Trees and two group pieces were parented into the backup, because the template parent was read after the old parts had moved.
    - All of it was repaired by hand.
  - `V4FX_Pack4Swap` is now the fixed version: it plans first and moves the old parts to the backup last. Run it through a fresh clone of the module. (Its header still says `…Swap2`.)
  - Play test:
    - no errors;
    - hill trees and cliff vines move;
    - the new Swamp bush bends when walked through (0.059 rad);
    - the 12 sun-gate vine pieces move and stay on the stone.
  - Performance at the Plains spawn: 623 active sway parts.
    - The replica sway update takes 1.0 ms per 30 Hz tick.
    - Render CPU is 3.31 ms with the sway and 2.89 ms without (0.4 ms for moving the parts).
    - Frame time is 6.06 ms either way.
  - New Blender base: `Map_Export/Data/snapshot_base_20261009_pack4.json`. Studio = Blender from here on.

## 2026-10-09 — V4: waterfalls come out of the rock, big wind gusts, frozen lake, camera floor, softer Shiny, Blender sync (Claude)
- What:
  - **Waterfalls** ("komt niet mooi uit de berg"): every fall used to start as a flat blue cap sticking out of the cliff top. Now on the cliff top there are:
    - spring rocks and a stream channel;
    - two big overhanging rocks the water comes out from under;
    - in Play: welling foam, flecks riding to the lip, a white crest + mist and a soft splash.
  - **Big wind gusts** through the tree crowns on the hills (wide wind ribbons, many leaves / petals / snow, rushing sound); trees, bushes and grass bend as a gust passes.
  - **Camera** can no longer go under the floor (Y ≥ 151.4 on the V4 map).
  - **Frozen lake:** fish swim under the ice, frost glitter, a sliding shine, powder snow blowing across, crack stars under your feet.
  - **Shiny pets give less light:**
    - the glint flash is every 2.6 s instead of 2.2 s, and its fill peak is 0.8 instead of 0.62;
    - glitter is 5/s instead of 8.4/s, smaller and with less glow;
    - the ting star is smaller;
    - the light is 0.12 / range 2;
    - the equip flash is softer.
  - **Path gaps** ("bij elke connectie van paden"): found, but not fixed yet.
    - In Studio, 9 of the 10 zone paths and link paths 03 / 05 / 06 / 07 are older, smaller versions than in Blender.
    - Blender's paths join without gaps (raster check: no hole over 0.1 sq unit).
    - The fix is a re-export + swap of those paths with the next import.
  - **Bush physics** works: tested by walking through `V4_Plains_Bush09`, which bent 0.185 rad. But only 69 bushes are separate parts; most bushes are inside merged groups and do not react. They need the same split as the hill trees.
  - **Blender sync** (`mr_sync.py`): a snapshot of the whole Blender scene, so the user's own Blender edits can be found (`diff()`) and exported alone (`export_update`). The user is now editing Blender.
- Where:
  - Studio: `StarterPlayerScripts.V4WorldFX` v10 (local copy `Map_Restyle/studio/V4WorldFX.client.luau`; pushed as data PNG `rbxassetid://140105850756193`, magic V4SC); `ReplicatedStorage.V4WorldData.TEX` (fish, windribbon, icecrack); `Workspace.V4_Props.WaterfallSources`; `ReplicatedStorage.GoblinMutationFX`.
  - Local: `Map_Restyle/fx_textures.py` (`fish_school`, `windribbon`, `icecrack`) and `Map_Export/FX_Textures/asset_ids.json`; `Map_Restyle/mr_sync.py`; `Map_Export/blender_snapshot.json`.
  - Backups: `ServerStorage.UI_Backup.V4WorldFX_v9_20261009`, `…GoblinMutationFX_before_shiny_less_light_20261009`; the .blend backup `Downloads/GoblinRNG_MAP_backup_20261009_voor_jouw_edits.blend`.
- Test:
  - Play: no errors.
  - Camera forced to Y 148.5 ends at 151.4.
  - Gusts spawn at crown height (2 at a time, 5–7 ribbons each).
  - Fish visible and swimming head first (offset direction measured on a test part).
  - Spring FX present.
  - Captures of the Plains / DarkForest waterfall tops.
- Still open:
  - When the user says "klaar":
    1. Run `mr_sync.diff()` for the user's edits.
    2. Split the cliff vines (`V4_<Zone>_CliffVine_<nnn>`, cluster the strands per cliff) and the bushes inside merged groups.
    3. Re-run `split_trees` if needed.
    4. Export one FBX: user edits + all paths + hill trees + vines + bushes + markers.
    5. After the import: align with the markers, swap in Studio (back up the old parts), set CanQuery off on vines, measure performance.
  - Watch the cost of the hill trees / vines.

## 2026-10-09 — V4: sunrise instead of night; link path fix + single vines in; the floating bridge grounded (Claude)
- What:
  - **Sunrise:** the user wanted "a bit of sunrise lighting, not night". New stylized sunrise skybox (pastel peach / lilac / blue, pink and gold lit clouds) made by `fx_textures.skybox_sunrise()`, Roblox's own sun 24° up; ten new sunrise looks in `V4WorldData.LOOK` (golden Plains, misty golden forest with sun rays, purple dawn Cave, orange Desert, pink Beach, pastel Ice, golden Temple, foggy Graveyard, misty neutral Swamp, radiant Heaven). Night FX are switched off by `V4WorldData.NIGHT = false`; the night looks / sky are kept (`LOOK_NIGHT`, `SKY_NIGHT`).
  - Morning birds replace the night crickets (`SOUNDS.birds` = Pro Sound Effects "Morning Birds 2", `SOUNDS.forest` = "Morning Forest Birds 4").
  - The user imported `Map_Export/V4_AnimPack2.fbx`: the 18 link paths now change texture in one straight cut (no jagged plank seam) and the Plains arch / DarkForest giant hang as 21 single vines that swing on their own (with falling leaves and glow bugs). The replaced parts are in `ServerStorage.V4_Backup_AnimPack_20261008` (`…__old2`).
  - `V4_DarkForest_Bridge_01` was floating (ends 0.6 studs above the ground): 0.68 studs lower in Studio (attribute `GroundFix`) and 0.19 m lower in Blender.
- Where: `ReplicatedStorage.V4WorldData` (NIGHT, SKY, LOOK, LOOK_NIGHT, SKY_NIGHT, SOUNDS, AMBIENCE), `Lighting.Sky`, `StarterPlayerScripts.V4WorldFX` (local copy `Map_Restyle/studio/V4WorldFX.client.luau`), `Workspace.V4_Props.Lanterns` (dimmed), `Map_Restyle/fx_textures.py` (`skybox_sunrise`, `petal_fb`), `Map_Restyle/mr_animpack.py` (`fix_link_paths`, `straight_link_paths`, `split_hangs`, `export_pack2`).
- Test: Play: spawn, dark forest and Swamp checked at sunrise (sun rays through the trees, readable shade, no errors in the output).
- Still open: check every zone's sunrise look with the user; the rest as in the entry below.

## 2026-10-08 — V4 map alive: night world, animated grass / trees / water, lights, sky FX, sounds (Claude)
- What:
  - **Animated foliage:** the user wanted the Blender grass itself to move (no terrain grass; my terrain grass test and the terrain water were removed again, the lilies put back).
    - `Map_Restyle/mr_animpack.py` split the merged grass / flowers / reeds / lilies into 4011 placements of 23 template meshes and exported `Map_Export/V4_AnimPack.fbx`; the user imported it and I aligned it with 3 marker cubes.
    - In Studio the clones are `Workspace.V4_Foliage/<zone>/Grass|Flower|Reed|Lily` (pivot at the foot, the zone's atlas texture); the cluster parts were swapped for versions without grass / flowers; bushes and Heaven floor clouds are single parts now; the windmill has a separate rotor.
  - **Night:** our own stylized night skybox (big moon, nebulas, stars), ClockTime 12 / latitude −36.5 so the light comes from the painted moon; a light look per zone, blended through the links (`V4WorldData.LOOK`), tuned after screenshots so every zone reads as night (Heaven was blown out, Desert looked like day).
  - **`StarterPlayerScripts.V4WorldFX`:** particles per zone, gusts, glowing butterflies, sway of grass / flowers / bushes / whole trees (trunk + crown, the user asked for more tree movement), falling leaves / snow / petals from crowns, spinning windmill, floating islands / clouds / bubbles, toon water lines + ripples + glints, waterfalls with falling streaks and a full impact (white water, splashes, spray, foam rings / patches, light, sound — the user asked for this), fire FX, 36 nearest glow lights, shooting stars, aurora, Heaven light shafts, lighthouse beam, night ambience + 3D waterfall / fire sounds, and animated floors (wind bands, cloud shadows, glitter).
  - **Props:** 63 lanterns / torches / tiki torches / frost lanterns / gold lamps along the paths (`Workspace.V4_Props.Lanterns`), lit windmill windows.
  - **Clean-up:** all `_NoCollide` parts are really non-collidable now; small ground clutter (pebbles, path stones, clusters) sits 0.14 studs lower without shadows so it stands out less (the user's request); ForestAmbience no longer runs its pollen / gusts / butterflies on the new map.
  - 25 stylized textures made by `Map_Restyle/fx_textures.py` and uploaded (ids in `Map_Export/FX_Textures/asset_ids.json`).
- Where:
  - Studio: `StarterPlayerScripts.V4WorldFX`, `ReplicatedStorage.V4WorldData` (textures, zones, looks, sounds), `ReplicatedStorage.V4WorldLights` (generated), `Workspace.V4_Foliage`, `Workspace.V4_Props`, `Lighting` (night sky + Plains look), `ServerStorage.V4FX_Templates` / `V4FX_LanternSpots` / `V4FX_DataPNG`.
  - Backups: `ServerStorage.V4_Backup_AnimPack_20261008` (replaced map parts), `ServerStorage.V4FX_DayLightingBackup` (day lighting + sky), `UI_Backup.ForestAmbience_before_v4fx_20261008`.
  - Local: `Map_Restyle/mr_animpack.py`, `Map_Restyle/fx_textures.py`, `Map_Restyle/studio/V4WorldFX.client.luau` (copy; Studio is the source), `Map_Restyle/studio/pack_data_png.py`, `Map_Export/V4_AnimPack.fbx`, `V4_foliage.json`, `FX_Textures/`.
  - Blender: collection `V4_AnimPack` (excluded from the view layer); backup `GoblinRNG_MAP_backup_before_animpack_20261008.blend`.
- Test: Play and walk from the spawn through all zones (the look changes in every link). No errors in the output. Performance: the frame time was the same with and without the FX in Studio; the sway costs about 0.5–1 ms per 30 Hz update in the busiest spot.
- Still open: the giant rarest-roll goblin + its VFX on the podium (to design with the user), leaderboard scripts, real zone prices / names, mobs; check the night look and the sounds on a phone / low graphics; a full map re-export from Blender would bring back the merged grass (run `mr_animpack.run()` and redo the swap).
- **Second round (after the user's play test), now in Studio** (script packed as `rbxassetid://106448942431886`; V4WorldData got `TEX.windsheen/cloudshadow/glints/petal`, `SOUNDS.angel`, new `LOOK`s; Heaven ambience = APM "Angel Voices A" with much less wind). Extra in this round: floating islands bob / turn / tilt with the things on top + hover FX under them (sparkles, wisps, hover rings, glow); cherry blossom petals (`fx_petal_fb`) + glints + pink glow on the Heaven trees and island tops, petals in the Heaven gusts; single hanging vines (see below) swing and drop leaves with glow bugs around them. Tested in Play: no errors, the dark forest gets light streams through the trees. Items:
  - animated floors (wind bands, cloud shadows, glitter), warm light shafts through the trees (also the cliff-top trees and the dark forest), touch physics (plants bend away and spring back, bushes shed leaves, flowers puff pollen, lilies dip), footsteps (water rings / sand / snow puffs), a soft light on the characters (the user could hardly see the goblins in the dark zones), glittering + breathing crystals, cacti / parasols sway, cloud puffs float, smoke from the witch hut chimney, toxic fumes from the barrels, a falling-spray veil + lip foam over every waterfall (hides the thin edges / the join at the cliff);
  - new looks: light zones lighter with warm moonlight, the dark forest lighter and warm, the Swamp much less green; lanterns / torches dimmer (Plains first);
  - link paths: the texture now changes in one straight cut in the middle of the link instead of the jagged border (Blender `mr_animpack.fix_link_paths()` + `straight_link_paths()`);
  - the hanging vines of the Plains sky arch (14) and the DarkForest fallen giant (7) are split into single vines (`mr_animpack.split_hangs()`, collection `V4_AnimPack2`, excluded from the view layer);
  - both are in `Map_Export/V4_AnimPack2.fbx` (18 link paths + 21 vines + markers `AP_Marker_*_P2`; replaces the older `V4_LinkPaths_Fix.fbx`). **Waiting for the user's import**; then: align with the markers, swap the 18 `V4_LinkXX{A,B}_Path_NoCollide` parts and replace `V4_Plains_ArchHang_NoCollide` / `V4_DarkForest_FallenGiantHang_NoCollide` by `V4_Plains_ArchVine_NN_NoCollide` / `V4_DarkForest_GiantVine_NN_NoCollide` (the script already knows them as kind "Vine").

## 2026-10-08 — V4 spawn plaza: the 4 v2 leaderboards in place (Claude)
- What:
  - The user's `Leaderboard_RarestRoll_v2`, `_Rolls_v2`, `_Playtime_v2` and `_Ascensions_v2` (props, no scripts yet) now stand in the curved row at the back west of the new spawn plaza, at the spots of `mr_plaza.board_spots()`.
    - From the podium side: RarestRoll, Rolls, Playtime, Ascensions.
    - Each is turned towards (702, -16) in zone 1's frame, so the text faces the entrance / spawn.
  - Changes to the models:
    - They were tilted ~18° backwards. They now stand upright, with the side that has the text in front.
    - They were 53–58 × 70–76 studs, too big for the plaza. They are scaled to 14 studs wide (18.4 high), which fits the ~18-stud spacing of the arc.
    - They stand on their own stone foot, 0.15 studs into the dirt.
  - The stone bases (`LeaderboardBase_01..04`) are no longer in the .blend (the user removed them; the boards have their own foot), so they are not in the Studio import either.
  - Each model got the attributes `OldPivot` / `OldScale` (its state before, to undo).
- Where: `Workspace.Leaderboard_*_v2`. The old `Leaderboard_Rolls / Playtime / Coins` in the old map are untouched.
- Test: every board is upright (UpVector.Y 1.000) with its text towards the SpawnLocation; Studio screenshot from above the spawn: the row of four at the back right, the podium on the left.
- Still open:
  - Leaderboard scripts (live data on the boards).
  - A bigger size if the user wants: up to ~16 studs wide fits the spacing.

## 2026-10-08 — V4 zone gates: invisible top piece, open once the zone is bought, placeholder unlock buttons; 2 new admins (Claude)
- What:
  - **Gates** (`V4_Gate_<a>_<b>_Barrier / _Wall`, 9 gates in the imported V4 models):
    - The light top piece (`_Barrier`) is now Transparency 1 for everyone, but it still blocks.
    - The dark `_Wall` stays visible while the zone is locked.
    - As soon as you own zone `<b>` (player attribute `Zones`), both stop colliding for you only (client side) and the wall fades out (0.8 s). Other players still see the gate closed until they own the zone.
  - **Placeholder unlock buttons + signs** (`Workspace.V4_ZoneGates`, 18 parts):
    - Per gate, on the zone side: a gold button `V4_ZoneUnlock_<b>` and a sign `V4_Sign_Zone<b>` on the dark wall.
      - The button is 16 × 3.4 × 0.7 with "UNLOCK" in Montserrat. It is a normal `ZoneUnlock` / `ZoneCost` part, so `ZoneServer` adds the ClickDetector and pays / saves, and `ZoneGateClient` does hover / press / shake.
      - The sign is a copy of the old zone-2 sign's look with "ZONE n", the name and the price.
    - Both have the attribute `Placeholder` and hide once the zone is owned.
    - Placeholder names and prices:
      - 2 Dark Forest 5,000; 3 Crystal Cave 25,000; 4 Desert 100,000;
      - 5 Beach 400,000; 6 Ice Peaks 1.5M; 7 Jungle Temple 5M;
      - 8 Graveyard 20M; 9 Toxic Swamp 75M; 10 Heaven 250M.
  - **Admins:**
    - UserIds 2970683260 and 1124895504 were added to `GoblinInventoryServer.ADMINS` (the `/admin` panel + admin remotes) and to `GoblinAdminCommands.ADMINS` (`/give`).
    - The friend 2741325690 stays only in the first list, as before.
    - Live servers need a publish.
- Where:
  - `StarterPlayerScripts.ZoneGateClient`: block "New map (V4) gates": `v4parts`, `applyV4`, `setupV4` (DescendantAdded/Removing, streaming-proof).
  - `Workspace.V4_ZoneGates`.
  - `ServerScriptService.GoblinInventoryServer` and `GoblinAdminCommands` (`ADMINS`).
  - Backups: `UI_Backup.ZoneGateClient_before_v4gates_20261008`, `GoblinInventoryServer_before_admins2_20261008`, `GoblinAdminCommands_before_admins2_20261008`.
- Test in Play:
  - Locked: Wall collide + visible, Barrier collide + invisible, the button has its `UnlockClick`.
  - After setting `Zones` = "1,2" (what an unlock publishes): gate 1→2 Wall / Barrier no collide + invisible, button + sign hidden; gate 2→3 still locked.
  - No errors.
  - (A real purchase could not be clicked from the test tools; it goes through the same `Zones` attribute.)
- Still open:
  - Real prices and names for zones 2–10, nicer buttons.
  - The old zone-2 button in the old map still works too, and unlocks the same zone 2.

## 2026-10-08 — V4 in Studio: spawn on the new plaza + invisible barriers at the outer edge of every area (Claude)
- What:
  - **Spawn:** `Workspace.SpawnLocation` moved onto the new spawn plaza, at Blender world (1000, 22): Studio (108.4, 149.7, 46.6). It faces -Z, towards the goblin podium and the leaderboard bases.
    - The old CFrame is kept in the attribute `OldCFrame`.
    - The Teleport menu entry "Fernwood Forest • Zone 1 Spawn" uses `workspace.SpawnLocation`, so it now goes to the new plaza.
    - The pad is still visible (14.6 × 14.6); it can be made invisible if wanted.
  - **Barriers** (`Workspace.V4_InvisibleBarriers`, 290 parts in 36 lines):
    - the user's sketch: a box around every zone, about 8 Blender units into the cliffs beyond the widest point of the zone's wall line;
    - closed at zone 1's south end and zone 10's north end (4 beyond the U wall);
    - open at every gate for the link (±19 units);
    - both sides of every link 4 units beyond the link walls.
    - The zone boxes and the link sides meet exactly at the gate lines.
    - The parts are 2 studs thick and 60 units (~214 studs) high from 2 units under the floor: Transparency 1, CanCollide on, CanTouch / CanQuery / CastShadow off, Locked.
    - The data was computed in Blender from the generator outlines (`mr_terrain.outline_xin`, `mr_v4_layout.plan` / `bezier`) with the Studio mapping 3.571 × (-x, z, y) + (3679.80, 150.10, -31.98).
- Where: Workspace (`SpawnLocation`, `V4_InvisibleBarriers`). The barrier lines in Studio XZ coordinates are kept in `Blend/Map_Export/V4_barriers_studio.json`.
- Test: in Play the character spawns at Y 153 on the SpawnLocation on the new plaza, looking at -Z, and stays standing. No errors.
- Still open:
  - NoCollide setup (734 parts still collide), Neon / lights, water, the old invisible walls inside the model;
  - the 4 `Leaderboard_*_v2` onto the bases; the podium goblin;
  - whether the spawn pad should be invisible.

## 2026-10-08 — GoblinRNG_MAP V4 in Studio: both halves aligned, zone 1 above the old zone 1 (Claude)
- What:
  - The user imported both V4 FBX files (Import 3D).
    - The first try of A failed: 2 small meshes timed out during upload, while A and B uploaded at the same time.
    - The user then re-imported A.
  - The importer had given each half its own scale and height:
    - A: 15.85 studs per Blender unit, floor at Y 809;
    - B: 15.15 studs per unit, floor at Y -49.
  - Both now use one mapping: Studio = S × (-x, z, y)_Blender + T.
    - S = 1/0.28 = 3.571 studs per Blender unit, the same as the V1–V3 walk test (so a bench seat is ~1.8 studs and the path ~16 studs wide).
    - T = (3679.80, 150.10, -31.98).
    - It was solved from 5 reference meshes per half (residual 0.00), then `ScaleTo` + `PivotTo` on both models.
    - Check after the move: 0.007 studs off.
  - Placement:
    - the new zone 1's centre (Blender world (1000, 36.9)) stands straight above the centre of the old zone 1 (`RAG_TEXTURED_EXPORT`, X 108.4 / Z 99.8);
    - the new floor is at Y 150.1, 40 studs above the old map's top;
    - same direction as the old zone 1: spawn at low Z, the gate to zone 2 towards +Z.
  - Both models have the attributes `ImportPivot`, `ImportScale` and `ImportStudsPerUnit` (the state straight after the import, to undo exactly) and `StudsPerBlenderUnit` = 3.571.
- Where: `Workspace.MR_Versie_4_Mix_A_zones1-5` (1,028 parts) and `Workspace.MR_Versie_4_Mix_B_zones6-10` (818 parts). All anchored.
- Test: the reference meshes of both halves sit exactly on the common mapping, so the seam (link 5 in A, zone 6 in B) fits.
- Still open:
  - The setup: 734 `*_NoCollide` parts still collide; Neon / lights on `*_Glow_NoCollide`; water; invisible walls; SpawnLocation on the new plaza; the 4 `Leaderboard_*_v2` models onto the bases; the goblin podium; lighting / effects.
  - The V1–V3 test imports are no longer in Workspace (removed by the user or a collaborator).
  - Formula for placing things from Blender: Studio = 3.571 × (-x, z, y) + (3679.80, 150.10, -31.98).

## 2026-10-08 — GoblinRNG_MAP V4: final FBX export for the Studio import (Claude)
- What: the user is done in Blender and wants the map in Studio.
  - V4 was exported again with `mr_v4_layout.export_fbx`. It includes everything of today: the new skulls, the sun gate, the smooth link floors and link pavers, the ice / swamp pieces and the spawn plaza with the goblin podium and leaderboard bases.
  - The .blend was saved before the export and was not changed by it.
- Where:
  - `Blend/Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx`: 65.6 MB, 899 single meshes + 129 joined groups.
  - `Blend/Map_Export/MR_Versie_4_Mix_B_zones6-10.fbx`: 58.7 MB, 704 single meshes + 114 joined groups.
  - No mesh is over 15k tris.
- Test: the export ran in 11.9 s with no temporary collection left.
- Still open: the user imports both files (Import 3D, scene positions on, same settings as the V1–V3 walk test); then Claude does the Studio setup.

## 2026-10-08 — GoblinRNG_MAP V4 zone 1: grass poking through the podium steps fixed (Claude)
- What:
  - The user's screenshot showed a green lens on the second step of the goblin podium. It was the mossy rim of the podium's base tier, which reaches into the stair area at r 4.05–4.42 and stood higher than step 2.
  - The steps moved 0.4 forward (centres -5.4 / -4.8 / -4.2 / -3.6, tops 0.26 / 0.52 / 0.79 / 1.08). The third step now covers the whole rim, so no rim sticks out above a lower step.
  - The cheek walls follow: 2.2 long, 0.95 high.
- Where: `Blend/Map_Restyle/mr_plaza.py` (`podium`); `V4_Plains_SpawnPodium` rebuilt (same seed, the rest of the plaza unchanged); Blender saved.
- Test: renders straight onto the steps and from the side: clean stone steps, no green.
- Still open: nothing new.

## 2026-10-08 — GoblinRNG_MAP V4 zone 1: spawn plaza bigger again + a smooth join with the path (Claude)
- What:
  - **Bigger** (the user's red circle): the plaza ellipse is now r (14.5, 11.5) around (700, -23), about 29 × 23. The podium and the four leaderboard bases now stand fully on the dirt.
    - The dirt keeps 0.6 away from the waterfall pool on the east (`POOL`).
    - Trees are only removed when they stand well inside (sdf < -1), so the edge trees by the pool stay.
  - **Join with the path** (the user's screenshot showed steps / notches in the edge):
    - Cause: the widened path ended abruptly at y -2, wider than the old ribbon, where the ribbon had been cut off.
    - Fix: the wide path now narrows smoothly (7.0 at the plaza → 4.9 at y -3 → 3.2 at y 4.5), so it ends inside the old ribbon (4.2–4.6 wide). The visible edge runs from one into the other without a step.
    - The path edge has no noise (only the plaza ellipse has the organic edge).
    - North of y -3 the ribbon, its path stones and the grass beside it are left alone; the plaza's own edge stones only line the stretch south of -2.5 that lies outside the plaza.
- Where: `Blend/Map_Restyle/mr_plaza.py` (`PLAZA_ELL`, `POOL`, `PATH_CENTRES`, `PATH_END`, `path_w`, `sdf`, `clear`); Blender `GoblinRNG_MAP.blend`, saved.
- Test: top renders of the plaza and of the join (smooth edges, no notches).
- Still open: the FBX re-export once the user is done, then the Studio work.

## 2026-10-08 — GoblinRNG_MAP V4 zone 1: spawn plaza made bigger (Claude)
- What: the user wanted the spawn area a bit bigger.
  - The dirt plaza ellipse grew from r (11, 8.2) to (13, 10), centred 2 further north: about 26 × 20 instead of 22 × 16.
  - The pieces moved outward with it:
    - the goblin podium to (709.3, -28.0);
    - the leaderboard bases onto an arc of r 11.5 around (699.5, -23) at 20/45/70/95°, turned towards (702, -16);
    - the benches to (712.2, -18.0) and (687.6, -16.4).
  - Also removed for the bigger area: EdgeRock_05 / 08 and EdgeGreen_0004 / 0006. The walls, cliffs and waterfall pool are untouched.
- Where: `Blend/Map_Restyle/mr_plaza.py` (constants at the top, then `run()`); Blender `GoblinRNG_MAP.blend`, saved.
- Test: render from the entrance and from the top. The bases stay clear of the south and west walls.
- Still open: as in the entry below (FBX re-export once the user is done, then the Studio work).

## 2026-10-08 — GoblinRNG_MAP V4 zone 1: spawn plaza with the goblin podium and 4 leaderboard bases (Claude)
- What: the start of the map (the dead end of the zone-1 path), after the user's concept 1.
  - A first build after concept 2 (pond with an island and a bridge) was removed again on the user's request.
  - The user's choice: the goblin, its VFX, the odds and the leaderboards are built later in Roblox Studio, because they are live data.
  - The path:
    - it widens from 4.4 to 6.4 towards the plaza (`SpawnPlaza_Floor_NoCollide`, same path texture and mapping, so it joins seamlessly) and opens into a large dirt plaza with an organic edge;
    - the floor is cut exactly along the edge, so there are no saw teeth;
    - grey stones line the wide path (`SpawnStones_NoCollide`); grass, flowers and pebbles soften the plaza edge (`SpawnPlazaEdge_NoCollide`).
  - The goblin podium in the back-east corner (`SpawnPodium`):
    - a rough stone base tier with a grassy rim, then a flagstone top tier with a gold inlay ring at r 2.5–2.72; the top is at local z 1.4;
    - 4 steps with cheek walls point at the plaza;
    - bushes, rocks and flowers around it (`SpawnPodiumGreen_NoCollide`).
  - `LeaderboardBase_01..04`: 4 bases in a curved row along the back-west, each 4 × 1.7 with its top at 0.66, for upright boards about 3.2 wide. They are turned towards (702, -18) so they read from the entrance.
  - 2 `Bench`es.
  - The meadow around (cleared by the first version) has grass, flowers, mushrooms and a few bushes and rocks again (`SpawnMeadow_NoCollide`).
  - No lantern posts (the user removed them from the map before; lights can come in Studio).
  - Removed from the plaza:
    - 36 path stones, 2 fences, 4 rocks, 4 bushes, 1 mushroom group, 1 EdgeGreen and Tree_02 (trunk + crown, it stood where the boards are);
    - the pieces of merged grass, flowers, pebbles and floor clusters there, and the end of the path ribbon.
    - Walls, cliffs, the waterfall pool and the edge trees stay.
  - Scale: the earlier Studio import used ~3.57 studs per Blender unit. Bench seat ~1.8 studs, board bases ~14 studs wide, podium top ~5 studs.
- Where:
  - Code: `Blend/Map_Restyle/mr_plaza.py`: `run()` = `clear()` + `build()`.
    - The positions live in zone 1's straight frame: `PODIUM` (708.3, -28.3), `BOARD_ARC`, `BENCHES`, the plaza shape `sdf()`.
    - Zone 1's frame is only shifted: world = local + (300, 39.1).
  - Blender: `GoblinRNG_MAP.blend`, saved. Backup: `GoblinRNG_MAP_backup_before_spawnplaza_20261008.blend`.
- Test: renders from the entrance, at an angle and from the top. The bases stay clear of the walls and the waterfall pool.
- Still open:
  - The user still wants to change something before the FBX re-export and the Studio import.
  - In Studio:
    - SpawnLocation on the plaza facing the podium / boards;
    - the 4 leaderboards (the old map has `Leaderboard_Rolls / Playtime / Coins` props without a script) with live data;
    - the giant goblin of the rarest global roll with VFX and odds (DONE 2026-10-09: see "Best roll ever podium" in section 1).

## 2026-10-08 — GoblinRNG_MAP V4: FBX re-exported with all of today's changes (Claude)
- What: the user wanted to upload the map to Roblox, so V4 was exported again with `mr_v4_layout.export_fbx`. The export includes:
  - the new skulls and the sun gate;
  - the smooth link floors and the link pavers;
  - the igloo camp and penguins, and the swamp pieces.
- Where:
  - `Blend/Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx`: 65.5 MB, 900 single meshes + 129 joined groups.
  - `Blend/Map_Export/MR_Versie_4_Mix_B_zones6-10.fbx`: 58.7 MB, 704 single meshes + 114 joined groups.
  - Both have embedded textures (incl. the desert atlas with the skull swatches) and scene positions; no mesh is over 18k tris.
- Test: export ran in 11.5 s; the .blend was not changed by it (not dirty, no temporary collection left).
- Still open:
  - The user imports both FBX files into Studio (Import 3D; the MCP cannot import files).
  - Then the setup: anchor, CanCollide off on `*_NoCollide`, Neon on `*_Glow_NoCollide`, water, invisible walls, scale to the goblins, fire / bubble particles.
  - `Workspace` still holds the V1–V3 walk-test imports (2,673 MeshParts).

## 2026-10-08 — GoblinRNG_MAP V4 swamp: the frog on the sunken boat removed (Claude)
- What: the user did not want the frog on the rowboat in swamp pond 3.
  - `V4_Swamp_SunkenBoat_01_NoCollide` was rebuilt without the frog, at the same spot (its world matrix was kept).
  - The `frog()` builder is removed from `mr_fun.py`.
- Where: `Blend/Map_Restyle/mr_fun.py` (`_sunken_boat`); Blender `GoblinRNG_MAP.blend`, saved.
- Test: render of the boat — the boat, seats and oar are there, the frog is gone.
- Still open: nothing.

## 2026-10-08 — Rainbow reveal default = 2 Supernova; new Shiny reveal sounds (Claude)
- What:
  - The user: "for the rainbow animation I'd rather have the 2nd one after all, and change the sfx when you get a shiny".
  - `MutReveal.DEFAULT_STYLE.RainbowReveal` is now 2 (Supernova, was 3 Regenboogvortex). It is used when the player attribute `RainbowReveal` is not set (the test panel can still set another style).
  - The Shiny roll reveal has new sounds (Pro Sound Effects, licensed in the Roblox library). All preloaded in SoundService:
    - while the light beam sweeps over the goblin: `ShinyShimmer1` = 9116394545 "Magic Glows Soft Clusters Of Chiming Hits 1", vol 0.55;
    - the moment the gold glow appears (and the end-picture ting after a cutscene): `ShinyTing1` = 9126073001 "Synth Sparkle Tone High Pitch Bell Tone Ding", vol 0.75, plus `ShinyTing2` = 9116395089 "Magic Glows … 4", vol 0.5.
    - Old ids for going back: 72277486589033 (0.8) + 79130903404606 (0.4, pitch 1.1). The Shiny equip sounds (GoblinMutationFX) are unchanged; the equip burst stays silent.
- Where: `StarterGui.RollGui.RollClient`: the `shinyReveal` block (`TING`, `TING.shimmer`, `ting(pitch, vol, set)`) and `DEFAULT_STYLE`. Backup: `ServerStorage.UI_Backup.RollClient_before_shinySfx_20261008`.
- Test:
  - In Play, all three sounds were loaded (`IsLoaded`).
  - A "Shiny,Rainbow" test roll (`StudioTestReveal`): SHINY at 2.9 s, RAINBOW at 4.7 s, no errors or warnings.
  - The user judges the sound by ear.
- Still open: if the sound is not liked, other candidates: 9116394876 / 9116394756 (other "Magic Glows" clusters), 9119447936 (bell tree).

## 2026-10-08 — GoblinRNG_MAP V4: igloo camp + penguins (Ice), fun in the toxic ponds (Swamp), white slabs on the link paths (Claude)
- What:
  - **Ice (zone 6).** The user had removed the snowmen and found the zone bare.
    - Igloo camp by the south gate: `V4_Ice_Igloo_01` (snow blocks, tunnel with a dark doorway, ice window, little red flag), `Campfire_01` (glow) with 3 `LogSeat`s, `Sled_01` with presents, `Firewood_01` and `Snowballs_01`.
    - Across the path: `V4_Ice_Igloo_02`, smaller and without a flag.
    - On the frozen lake and its east shore:
      - 7 `V4_Ice_Penguin_01..07`: one with a beanie, one with a scarf, one waving, one sliding on its belly and a baby;
      - `FishingHole_01_NoCollide`: a hole with dark water and broken ice, a rod with its line in the hole, a bucket with fish and a fish on the ice.
  - **Swamp (zone 9).** The user had removed the lily pads.
    - Pond 1: `V4_Swamp_Alligator_01_NoCollide` peeks out of the water (yellow slit eyes, teeth, back scutes, tail tip).
    - Pond 2: `ToxicBarrels_01`, one barrel tipped over and leaking glowing goo, plus one upright.
    - Pond 3: `SunkenBoat_01`, a half-sunken rowboat with an oar and a frog on the seat.
    - Glowing toxic bubbles (`ToxicBubbles_01..04_Glow_NoCollide`) in all three ponds and in the big glowing lake.
    - `DangerSign_01`: a skull-and-crossbones sign with dripping goo, by the path at the lake.
  - **Link paths.** The user said the white stones were missing at the start of the Heaven path.
    - The link paths now get the same stone slabs as the paths inside paved zones (Cave, Ice, Temple, Graveyard, Heaven), only on that zone's side of the texture border.
    - New objects: `V4_LinkxxA/B_Pavers_NoCollide` in links 2, 3, 5, 6, 7, 8 and 9.
  - Every new item is its own object (selectable and movable). Glow parts are `*_Glow_NoCollide`. Swamp items in the water are `_NoCollide`.
- Where:
  - Code: `Blend/Map_Restyle/mr_fun.py`. `run()` places or replaces everything; `items()` holds the zone-local positions and sizes; `free_map(zi)` prints an ASCII map of the free floor.
  - `mr_v4_layout.build_link`: `link_pavers()`, plus `only_pavers=True` to rebuild just the slabs.
  - Blender: `GoblinRNG_MAP.blend`, saved. Backup: `GoblinRNG_MAP_backup_before_icefun_20261008.blend`.
- Test: renders of the camp, the penguins, every pond, the sign and the Heaven link path.
- Still open:
  - In Studio, the bubbles and campfire could get ParticleEmitters (rising bubbles, smoke) and lights.
  - The V4 FBX in `Map_Export` is older than today's map changes. Re-export before the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4 temple: the stone face gate became a Mayan sun gate (the user's reference) (Claude)
- What:
  - The user did not like the stone face (brow, cyan eyes, triangle nose) on top of the temple gate. They showed a reference: a Mayan-style gate without a face.
  - New `V4_Temple_SunGate_01`, same spot and rotation as the old gate (about 33 x 20 x 32 m, 4.3k tris, collides):
    - two tapered pillars in stone courses, on three-step plinths, each with a stepped buttress on the outer side;
    - on the front and back of each pillar: a gold meander panel, a small gold square, and a fire bowl on a stone bracket (bronze bowl, gold rim, coals);
    - the lintel: end blocks with inverted gold triangles, a beam with 7 gold square inlays, and a cornice;
    - a 4-step crown with a gold sun medallion (frame, disc, 8 rays, bright centre) between two gold key spirals;
    - moss on every top, moss drips over the edges, broken stones on top, and some fallen stones at the base.
  - The flames are glow: `V4_Temple_SunGate_01_Glow_NoCollide` (lava / ember).
  - The vines are their own object: `V4_Temple_SunGate_01_Vines_NoCollide` (5.5k tris).
  - Removed: `V4_Temple_FaceGate_01`, its `_Glow_NoCollide` eyes, and `V4_Temple_GateVines_NoCollide` (those vines hung from the old lintel).
  - The plinths still touch some neighbours: the edge of the left pond (water, shore, 2 pond rocks), a ruin block, a bush, grass and a tree canopy. The old pillar already stood in that pond. The tree, the rock and the pyramid that overlapped the first try are clear now.
- Where:
  - Code: `Blend/Map_Restyle/mr_sungate.py`. `swap()` replaces the gate in place and can be run again.
  - `mr_v3.landmark` (`face_gate`): when `ctx.ver == 4` it builds the sun gate, so a V4 rebuild keeps it. The zone's random stream is unchanged.
  - `mr_qa.GROUND_WORDS` now includes `SunGate`.
  - Blender: `GoblinRNG_MAP.blend`, saved. Backup: `GoblinRNG_MAP_backup_before_sungate_20261008.blend`.
- Test: renders from the front and at an angle, in isolation and in place; BVH overlap test against all neighbours (see above).
- Still open:
  - In Studio, the fire bowls could get a real Fire / ParticleEmitter and light (attach to `…_Glow_NoCollide`).
  - The V4 FBX in `Map_Export` still has the old gate (and the old skulls and the saw-tooth link floors). Re-export before the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4: no more saw teeth where two link floor textures meet (Claude)
- What:
  - The user saw a zigzag (saw teeth) between snow and grass in the Ice → Temple link.
  - Cause: the link floor gave each whole 0.9 m grid triangle to one texture, so a slanted border became a staircase of triangles.
  - Fix:
    - The border field is now continuous (`t_along` + noise, `a_field`).
    - Triangles that cross the border are cut exactly on it, so the border is a smooth line.
  - Only the floor meshes `V4_LinkxxA/B_Terrain` of all 9 links were replaced. Walls, paths, stones and plants are untouched.
- Where:
  - Code: `Blend/Map_Restyle/mr_v4_layout.py`: `build_link(P, i, only_floor=False)`. `only_floor=True` rebuilds only the two floor meshes; it also adds `t_along`, `a_field`, `put` and `cut`.
  - Blender: `GoblinRNG_MAP.blend`, saved. Backup: `GoblinRNG_MAP_backup_before_linkfloors_20261008.blend`.
- Test: ray grid every 0.5 m over every link floor: 0 holes and 0 places where both floors overlap, in all 9 links. Top render of link 6 shows a smooth border.
- Still open: the V4 FBX in `Map_Export` is from before this fix and before the new skulls. Re-export before the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4 desert: the two skulls remodelled (mammoth + T. rex, after the user's reference photos) (Claude)
- What:
  - The user did not like the round cartoon skulls. Only those two objects were replaced; the user's own edits stayed untouched.
  - `V4_Desert_GiantSkull_01` (at the oasis) became `V4_Desert_MammothSkull_01`, about 6 x 8.7 x 5.6 m, 13.5k tris.
    - It has a high dome, a big nasal opening, eye sockets, fused tusk sheaths and a lower jaw.
    - Two long curved tusks cross each other in front.
    - Olive-yellow bone with ivory tusks.
  - `V4_Desert_GiantSkull_02` (next to the ribcage) became `V4_Desert_TrexSkull_01`, about 4.5 x 10 x 7 m, 14.2k tris.
    - The jaws are open, with 32 upper and 26 lower curved teeth.
    - The skull is hollow, and its fenestrae are see-through: nostril, maxillary, antorbital, keyhole eye socket, lateral and supratemporal, and the mandibular one.
    - Grey-brown fossil colour with cream teeth.
  - Both skulls keep the old spot and rotation, stand on the floor at z 0, use `MR_Desert_Atlas`, and still collide (no `_NoCollide`). The only thing they overlap is the grass tufts.
  - No 3D generator was available (all off), so the shapes are signed distance fields (numpy), meshed with OpenVDB, decimated, and coloured per face.
  - The reference colours did not exist in the palette. They were painted into the empty swatch slots 80–101 of `MR_Desert_Atlas`; slots 0–79 and the strips were checked and are unchanged.
    - The PNG was saved and repacked in the .blend.
    - `mr_skulls.ensure_swatches()` repaints the slots if the atlas is ever regenerated.
- Where:
  - Code: `Blend/Map_Restyle/mr_skulls.py`. In Blender, `run()` rebuilds both skulls in place; `MAMMOTH_S` and `TREX_L` set the size.
  - Blender: `GoblinRNG_MAP.blend`, saved. Backups: `GoblinRNG_MAP_backup_before_skulls_20261008.blend` and `Map_Restyle/textures/MR_Desert_Atlas_backup_before_skulls_20261008.png`.
- Test: checked with renders from several sides, in isolation and in place, and with a BVH overlap test against all neighbours.
- Still open: the V4 FBX in `Map_Export` still has the old skulls. Re-export with `mr_v4_layout.export_fbx` before the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4 rebuilt after the Blender restart; stones / bushes as separate objects; fast export (Claude)
- What:
  - Blender had restarted (the MCP disconnected) and V4 had never been saved, so it was rebuilt from the generator (flat, zigzag, seamless links) and saved by Claude this time.
  - Separate objects (the user: "a stone or a bush I must be able to select and move"): the categories in `mr_decor.SPLIT_V4` are one object per item, with the origin on the item and the custom property `mr_merge` = category.
    - Separate: path stones, bushes, rocks, rim trees, cliff-top plants, mushrooms, crystals, wall elements, seal rocks …
    - Still merged: tiny things (grass, flowers, pebbles, clusters, pavers, reeds, lilies).
  - V4 now has 7,279 mesh objects in Blender.
  - `mr_v4_layout.export_fbx(folder)` joins the `mr_merge` pieces again per category (≤ 18k tris) with `join_fast` (numpy, no `bpy.ops.join`). It now takes 13 s instead of 18+ minutes, and exports about 1,840 meshes as before.
- Where:
  - Code: `Blend/Map_Restyle/mr_decor.py` (`SPLIT`, `SPLIT_V4`, `Ctx._flush_split`), `mr_v4.py` (per-item merges), `mr_v2.element_rows` (now `ctx.merge`), `mr_v4_layout.py` (`join_fast`, `export_fbx`).
  - Blender: `C:\Users\qmarc\Downloads\GoblinRNG_MAP.blend`, saved (159 MB). Backup of the file before this: `GoblinRNG_MAP_backup_before_v4_20261008.blend`.
  - FBX: `Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` (65 MB) and `…_B_zones6-10.fbx` (58 MB).
- Test: QA 0 issues in all zones and links; 0 side leaks and 0 floor holes at the gates; floor at the gates within 0.06.
- Still open: the Studio import. To rebuild V4, set `mr_decor.SPLIT = mr_decor.SPLIT_V4`, then run `build_zone` 1–10, clear the `mr_layout` / `mr_hz` flags, then `plan` / `apply` / `build_link`.

## 2026-10-08 — GoblinRNG_MAP V4: completely flat (Claude)
- What (the user: "no height differences anywhere, just a flat map"):
  - `mr_v4.FLAT = True`:
    - the zone floor is height 0 everywhere: no noise, no rise towards the walls, no terraces with stairs;
    - only the water (ponds, lakes, the stream) still lies lower.
  - The link floors are flat too.
  - All 10 zones were rebuilt and QA-fixed, the zigzag was applied again, and the links rebuilt.
- Where: `Blend/Map_Restyle/mr_v4.py` (`FLAT`), `mr_v4_layout.py`; FBX re-exported (`Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` 69 MB, `…_B_zones6-10.fbx` 62 MB).
- Test:
  - 7,200 floor samples over all zones and links: highest 0.00; only 0.7% lower than −0.05 (the edges of the water dips).
  - 0 floor holes; 0 side leaks at the gates; 0 holes from above near the gates.
- Still open: the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4: link floors at the right height, ground under the walls (Claude)
- What (the user's screenshots: steps and empty strips at the gates, a path standing up like a wall, empty spots under the walls):
  - The links had been built against the zones' old heights: `matrix_world` is stale after moving objects until the depsgraph updates. `build_link` now calls `view_layer.update()` first.
  - Each link floor now runs 1.8 units on under the zone floor, slightly lower (zone texture on top), and out to 26 units beside the corridor, under the walls.
- Where: `Blend/Map_Restyle/mr_v4_layout.py` (`build_link`); FBX re-exported (`Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` 71 MB, `…_B_zones6-10.fbx` 64 MB).
- Test:
  - At every gate (2,610 points): 0 floor holes, the largest zone / link step is 0.09.
  - Beside and under the walls up to 40 out: 0 holes seen from above.
  - `mr_qa.fix` on all links.
- Still open: the Studio import.

## 2026-10-08 — GoblinRNG_MAP V4: no height differences, seamless links with blending textures, no gaps or overlaps (Claude)
- What (the user: "connect everything perfectly, fade the colours / textures into each other, remove all height differences, keep the zigzag, fix the gaps", plus screenshots of a desert floor overlapping the cave and a slit in the walls):
  - All zones are at height 0 (`reheight()`).
  - The links were rebuilt from scratch:
    - floor, walls and greenery are clipped to the gate lines (this fixes the overlap);
    - the floor height meets the zones at the gates;
    - floor and path go from zone i's texture to zone i+1's along an organic border in the middle, each mapped in its zone's frame (no seam at the gate);
    - the walls are half zone i's style and half zone i+1's;
    - seal rocks sit at every junction.
- Where:
  - `Blend/Map_Restyle/mr_v4_layout.py`: `reheight`, `build_link`, `link_path`, `link_green`.
  - `mr_v4.rim_cover(..., clip=)`.
  - FBX re-exported: `Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` (70 MB) and `…_B_zones6-10.fbx` (62 MB).
- Test:
  - `mr_qa.fix` on all links: 0 issues.
  - Ray sweep from the gates, zones and links (5,220 rays): 0 leaks.
  - Renders of the joins from above.
- Still open: the Studio import; the user's crop of wooden planks is probably the Beach path, which is a boardwalk (`MR_Beach_Path`).

## 2026-10-08 — GoblinRNG_MAP version 4 "Mix": V2 walls + V3 ceilings + reference detail, winding layout with height (Claude)
- What:
  - The user asked for a mix of map 2 (walls) and map 3 (the ceiling), with about the detail of their three reference pictures, a full rim, and no straight line but height differences.
  - New: stepped stone cliff rows with grass caps, vines and trees; a tree / themed rim everywhere; waterfalls into pools; stone terraces with stairs; paved paths; edge stones; fences; floor clusters; themed sets per zone.
  - The V3 landmarks span the canyon on cliff pillars.
  - The zones wind and differ in height, joined by link corridors (stairs on the steep ones, arches on 2 of them).
  - The lantern posts were removed on request (158 objects, and the generators no longer make them).
- Where:
  - Blender `GoblinRNG_MAP.blend`, collection `MR_Versie_4_Mix` (unsaved: the user must press Ctrl+S).
  - Code: `Blend/Map_Restyle/mr_v4.py` and `mr_v4_layout.py`.
  - Renders: `Map_Restyle/renders/v4_*.png`.
  - FBX: `Map_Export/MR_Versie_4_Mix_A_zones1-5.fbx` and `…_B_zones6-10.fbx`.
- Test:
  - `mr_qa.fix` on every zone and link: 0 floating / buried issues left.
  - Renders of all 10 zones, the layout and 3 links at player height.
- Still open:
  - Import of the two FBX files into Studio (manual).
  - Then in Studio: CanCollide off on `*_NoCollide`, positions / scale, and effects (waterfall splash, crystal / lamp light, fireflies) from object names.
  - 1.73 M tris is heavy; thin out `rim_cover` if Studio struggles.
  - The user picks whether V4 replaces the others.

## 2026-10-08 — New Best Roll: cutscene per "x rarer" tier, Filter Cutscenes slider in Settings (Claude)
- What:
  - The user's tiers are locked in: the record's ratio to the previous best picks the cutscene. 20x Toverportaal, 50x Jackpot, 100x Steencirkel, 200x Supernova, 300x+ Kristalgeode. A first record counts as 300x+. Below 20x the style setting plays (Mix).
  - The Mix is now 3 or 4; the geode left it because it is the 300x tier.
  - Settings: "Hide Cutscenes" became the slider "Filter Cutscenes" (Low … Max, 7 snapped steps, no numbers, as the user asked). Each step hides the next smaller tier; Max turns all cutscenes off, and the normal reveals play.
  - The test panel's "Test: New Best Roll" now fakes a previous best that fits the picked style's tier, so picking Jackpot really plays the 50x path.
- Where:
  - `ReplicatedStorage.RecordCutscenes`: `TIERS`, `MutReveal.TIERS`, `MutReveal.ratioOf`, `FILTER_MIN`, `MIX = { 3, 4 }` (in front of / inside `MutReveal.cutscene`).
  - `StarterPlayerScripts.GoblinPetClient`: the Filter Cutscenes row (do-block after Show Other Pets), `settings.cutsceneFilter`, and `applySnapshot` sets `CutsceneFilter`.
  - `ServerScriptService.GoblinInventory`: `CleanSettings` keeps `cutsceneFilter` 1–7; old `hideCutscenes = true` becomes 7.
  - `StarterGui.RollGui.RollClient`: `preview` (prevBest per tier).
  - Backups: `UI_Backup.*_before_cutsceneTiers_20261008` (RecordCutscenes, RollClient, GoblinPetClient, GoblinInventory).
- Test (Play, monitor on):
  - The style 13 test played the Jackpot (reels present).
  - The style 9 test (20–27x) with the filter on step 3 played no cutscene.
  - Setting `CutsceneFilter` to 7 sets `HideCutscenes` = true.
  - The tier table was checked for ratios 5 … 1000 and a first record.
  - Screenshots stayed black (Studio was in the background).
- Still open: which cutscene the user wants below 20x (now the Mix 3/4); the user has not seen the slider yet.

## 2026-10-08 — GoblinRNG_MAP: FBX export of the 3 versions for a walk test in Studio (Claude)
- What: each restyled version exported as its own FBX, with embedded textures, for a manual Studio import (the MCP cannot import files).
- Where: `Blend/Map_Export/MR_Versie_1_Kliffen.fbx` (1069 meshes, 573k tris, 58 MB), `MR_Versie_2_Themawanden.fbx` (822 meshes, 658k tris, 63 MB), `MR_Versie_3_Avontuur.fbx` (782 meshes, 375k tris, 47 MB). Exported in Blender units with scene positions (x +140 / +280 / +420), no meshes over 18k tris.
- Test: import in Studio (Home > Import 3D), one version at a time; afterwards, set CanCollide off on `*_NoCollide`, hide the invisible walls, and check scale and position against the spawn.
- Still open: the user's import; scale check (Blender gate = 31.8 × 6 units); a Blender walk-through is possible with View > Navigation > Walk Navigation.

## 2026-10-08 — New Best Roll: 8 extra cutscenes, styles 7–14 (Claude)
- What (the user: "8 extra, extremely cool cutscenes, like the geode one"):
  - 7 Meteoorinslag: a star grows into a burning meteor, the last bit in slow motion, an impact with three shockwaves and a crater; the goblin rises on a beam of light.
  - 8 Drakenei: a lava nest at dusk; the egg rocks three times, light shoots out of cracks (2, 4, 8 beams), it hatches in fire and blows the front nest stones away; the goblin gets wings of fire.
  - 9 Toverportaal: rune circles draw themselves and swing upright into a spinning portal with lightning; the goblin walks out and the portal collapses.
  - 10 Raketlancering: a 3-2-1 countdown on screen, lift-off, a firework in the goblin's colours, a capsule on a parachute; the goblin jumps out of the hatch.
  - 11 Parelschelp: underwater (sunbeams, seaweed, a school of fish, coral); the shell opens, the pearl rises, three heartbeats, it bursts into bubbles; the goblin floats down.
  - 12 Supernova: a spiral galaxy winds into one star, three heartbeats, a supernova; the goblin on an asteroid.
  - 13 Jackpot: a giant slot machine; the reels stop one by one on the rolled goblin's icon (the third one crawls), JACKPOT!, coins; the goblin jumps out of the tray.
  - 14 Steencirkel: a stone circle at dusk with the goblin as a grey statue (a Highlight, the puppet frozen); runes light up and send beams, a column of light, three heartbeats, it breaks and the goblin comes alive.
  - All 8 use the same director, puppet, poses, title card and the user's picked sounds (build / reveal / title) as 1–5, and no applause.
  - Set dressing stays out of the camera corridor, so nothing stands between the camera and the goblin in the title shot.
  - Style 6 (Mix) is unchanged: 2, 3 or 4.
  - The test panel "New Best Roll (test)" now lists the 14 styles in two columns.
- Where:
  - `ReplicatedStorage.RecordCutscenesExtra` (new ModuleScript, ~80k characters; local copy `Goblin_Pets/RecordCutscenesExtra.luau`, same hash as Studio).
  - `ReplicatedStorage.RecordCutscenes`: the `EXTRA_NAMES` block before `MutReveal.CUTSCENES` requires the extra module with a helper table, in a pcall; the names are appended to `MutReveal.CUTSCENES`; the style clamp is now `1..#MutReveal.CUTSCENES`.
  - `StarterGui.RollGui.RollClient`: the `CutsceneTest` panel (two columns, 372 px wide).
  - Backups: `UI_Backup.RecordCutscenes_before_extraCuts_20261008`, `UI_Backup.RollClient_before_extraCuts_20261008`.
- Test:
  - Set the player attribute `CutsceneStyle` to 7–14 (or click it in the test panel), then set the `RollGui` attribute `StudioTestReveal` = "Record".
  - All 8 were played in Play with plain, Rainbow, Huge, Shiny and Huge+Shiny+Rainbow goblins. The monitor was in standby, so this went through the Heartbeat harness (see the test hooks in section 1).
  - Results: no errors; 12–15 s each (the geode takes 13.4 s); stage, light and camera restored afterwards; no blockers in the title shot; the Jackpot reels land exactly on the rolled goblin.
  - Cost: peak 100–330 parts (geode 163), with the same ~40 ms start hitch as the geode.
- Still open:
  - The user has not seen them yet. Screenshots were impossible with the monitor off, and tweens don't run in standby, so the looks are unchecked.
  - Whether the new ones should join the Mix.

## 2026-10-08 — GoblinRNG_MAP: 3 restyled versions of all 10 zones, game textures, high walls, QA (Claude)
- What (the user: "really textured in our game style, many more objects, a new shape, really high walls with a nice top-to-bottom transition", then "3 versions per map" and "check for floating / misplaced objects"):
  - **Versie 1 "Hoge Kliffen":**
    - keeps each zone's original layout, restyled onto the zone atlas;
    - replaces the low boulder walls with tiered cliffs 30–46 units high, with grass, snow or sand ledges and tops, strata, vegetation on ledges and tops, and hanging vines or icicles;
    - one feature per zone: waterfall + pond, waterfall into the stream, crystal veins, oasis, lagoon waterfall, frozen fall, temple stairs, crypts, glowing fall + lilies and reeds, golden light-fall;
    - many more props, plus paths with lanterns where none existed;
    - the Cave tunnel box became a timbered mine entrance.
  - **Versie 2 "Thema-wanden":**
    - new organic outline with alcoves and gate pinches, and a rolling terrain;
    - walls built from rows of big themed elements: hills, giant trees with a canopy roof, crystal spires, mesas and hoodoos, sea stacks, glacier blocks, temple blocks, castle walls, mangroves, cloud banks;
    - landmarks: windmill, stream + bridge, crystal lake, skull + oasis, lighthouse, frozen lake + snowmen, pyramid temple, mausoleum, witch hut, fountain.
  - **Versie 3 "Avontuur":**
    - crag walls about 50 units high, with overhangs and spires;
    - floor on several levels: plateaus with ramps, a river or lake with a bridge;
    - one landmark over the path per zone: natural arch, fallen giant tree, rock ribs, ribcage + giant skull, sea arch, ice arch, stone face gate, broken bridge, root arch, golden gate + floating islands (the islands float on purpose, flagged `mr_float_ok`).
- Where:
  - `C:\Users\qmarc\Downloads\GoblinRNG_MAP.blend`: the user saves it with Ctrl+S. A safety copy is `GoblinRNG_MAP_3_versies.blend`, and the state from before is `GoblinRNG_MAP_backup_before_restyle_20261008.blend`.
  - Scripts and textures: `Blend/Map_Restyle/`.
  - Renders: `Map_Restyle/renders/` (`Zone_XX_<Key>_V1_V2_V3.png`, `Overzicht_alle_versies.png`).
  - See section 1, "Map in Blender".
- Test:
  - `mr_qa.check` on all 30 zone collections (2,661 objects, ≈1.6 M tris): 0 floating and 0 buried pieces.
  - A path / gate-opening check: nothing solid on a path or in a gate opening. The only exceptions are walk-through arches.
  - Every zone was checked visually from inside and from the air, and rendered in EEVEE.
  - The render engine and world were set back to Cycles / World afterwards.
- Still open:
  - the user picks a version per zone;
  - FBX export to Studio (and the scale) is not done;
  - Roblox collision on the big wall meshes;
  - next request: 8 new New Best Roll cutscenes.

## 2026-10-08 — Hide Rolls comes back after a Rainbow, calmer gear, smaller enemy rings, thinner roll outline, less Rainbow light, fewer Shiny particles (Claude)
- What:
  - **Hide Rolls:** a Rainbow still switches Hide Rolls off for that roll. Now `playRoll` sets `MutReveal.restoreHidden`, and the next roll starts by switching Hide Rolls back on. Pressing Hide Rolls yourself (`setHidden(true)`) clears the flag; it is only set when Hide Rolls was on.
  - **Settings dock icon click:** no full 360° turn any more. The gear kicks to −28° and springs back (Elastic, 0.55 s), sparkles unchanged. The hover wiggle is unchanged.
  - **Enemy red circle:** diameter ×0.85 (beetle 2.74, boar 2.69, golem 3.26). It has its own thinner dashed ring `rbxassetid://93690048613676` (~75 % of the old line width), drawn by `HUD_Mockup/make_threat_ring.py`.
  - **Roll outline:** the black outline round the cycling goblin icons is 3 px instead of 4 (`UI.outline(icon, 3, …)` in the slot builder).
  - **Rainbow goblins give off less light:**
    - PointLight Range 7 → 5, Brightness 0.86 → 0.42;
    - the white glint sweep over the skin lerps to white at 0.4 (was 0.61);
    - glint sparkles 0.6 → 0.4 chance per frame;
    - the prism footprints' SurfaceGui Brightness 2.5 → 1.5.
  - **Shiny particles ~30 % fewer:**
    - glitter Rate 12 → 8.4;
    - the small ting sparkles 4 → 3;
    - footstep twinkles 3 → ~2.1 and sparks 2 → ~1.4 per step;
    - equip burst ring 28 → 20, star rain 34 → 24.
- Where: Studio `StarterGui.RollGui.RollClient`, `StarterPlayerScripts.GoblinPetClient` (dock `Settings` click), `StarterPlayerScripts.EnemyAnimator` (`THREAT`, `THREAT_RING`), `ReplicatedStorage.GoblinMutationFX`. Backups in `ServerStorage.UI_Backup`: `RollClient_before_restore_hidden`, `GoblinPetClient_before_settings_click`, `EnemyAnimator_before_thin_ring`, `GoblinMutationFX_before_less_light`.
- Test (Play, no errors):
  - `StudioSetHidden` true, then `StudioTestReveal` "Rainbow": RollsHidden false during the roll and after it. The next test roll ("Huge"): RollsHidden true again.
  - The three ThreatRings use the new texture.
  - Live values: RainbowLight 0.42 / 5, ShinyGlitter Rate 8.4.
  - All changes only lower effect counts and brightness, so render cost can only go down (not measured separately).
- Still open: the user judges the look. More or less change is one number each (see above).

## 2026-10-08 — Hover card background: square pattern with a soft shine (Claude)
- What: the goblin hover card got the background from the user's reference. The card's look now sits in layers under its content (bottom to top):
  - `Bg`: white, with the navy edge.
  - `Pattern`: a tiled grid of small rounded light-blue squares in a few sizes and tints, with some cells empty. A UIGradient makes it faint on the left and clear on the right.
  - `PatternShine`: the same tile in a deeper blue. A soft diagonal band of it sweeps over the card in 1.6 s, then rests 1.8 s (a 3.4 s cycle). This is the light animation the user asked for.
  - `Card`: the content (transparent background), then `TailJoin`.
  - The layers are siblings of `Card`, because the card's UIListLayout would lay them out. `place()` sizes them to the card every frame and drives the shine offset.
- Where:
  - `StarterPlayerScripts.GoblinPetClient`, tooltip section.
  - Tile `rbxassetid://98696322203887`, drawn by `HUD_Mockup/make_tooltip_pattern.py` (6 × 6 cells of 16 px, drawn at 2× and shown as a 96 px tile).
  - Backup: `UI_Backup.GoblinPetClient_before_tooltip_pattern`.
- Test: in Play the pattern shows on the card as in the reference. Render CPU with the card shown: 3.57 ms vs 3.51 ms hidden, so the shine costs nothing measurable. No errors.
- Still open: nothing.

## 2026-10-08 — Hover card: Damage/Health rows with icons, scroll fix; inventory sorted on Best Goblin (Claude)
- What:
  - **Stats in the hover card.** Under "1 in N" there is now a thin divider that fades out to both ends, then two rows that are always there: crossed swords "Damage:" and a heart "Health:", with "—" as the value until the stats model exists. Extra stat rows still go under them, and the card grows with them.
    - Values come from `GoblinPets.StatsOf(goblinId, mutations)` or `design.Stats`: `{ damage =, health = }` plus an optional list of extra rows.
  - **New icons** in the user's reference style (thick near-black outline):
    - swords `rbxassetid://139254435392729`: broad blue blades, short hilts, round pommels;
    - heart `rbxassetid://122611404171170`: the classic parametric heart (two round lobes, a dip, a sharp point), coral red with a soft highlight and a thicker black border, as the user asked.
    - Both are drawn by `HUD_Mockup/make_stat_icons.py`, which runs with Blender's bundled Python (numpy only), and uploaded through a local `http.server` + `upload_image`.
  - **Scroll fix (the user's video).** Scrolling moved the cards under a still mouse, and Roblox fires no MouseEnter/Leave then, so the card kept following its old goblin out of view.
    - Now every frame checks whether the mouse is still on the card's goblin. If not, it looks up the card under the mouse (`PlayerGui:GetGuiObjectsAtPosition`, buttons registered by `withTip`) and switches to it, or fades out.
    - After each mouse-wheel turn (while a menu panel is open) it keeps looking for 0.8 s, so it also finds the card once the scroll stops between two cards.
  - **The inventory opens sorted on Best Goblin** (`invUI.sortMode = "best"`).
- Where:
  - `StarterPlayerScripts.GoblinPetClient`: the tooltip section and the sort default.
  - Backup: `UI_Backup.GoblinPetClient_before_tooltip_stats`.
- Test:
  - In Play the inventory opened as "Sort: Best Goblin" with the Rainbow Huge Shiny Void Watcher first.
  - Hovering showed both stat rows with icons.
  - A hover followed by two wheel turns (canvas 0 → 200) switched the card to Crystal Lord, the goblin now under the mouse, placed beside it.
  - No errors.
- Still open: the real Damage/Health values (the stats model).

## 2026-10-07 — Goblin hover tooltip in the user's design, with a stats section (Claude)
- What: the tooltip when hovering a goblin (inventory cards and equipped slots) now follows the user's mock-up.
  - **Shape:** a white speech-bubble card with a navy edge beside the hovered icon. Its tail points at the icon; the card goes to the left when there is no room on the right.
  - **The name.** Mutation words are coloured like the world nametags: Huge purple, Shiny pale gold, Rainbow in moving rainbow colours; each coloured word has a thin navy outline.
  - **The pills.** A navy rarity pill (e.g. CELESTIAL), then a mutation pill: "Normal" when there is none, or Rainbow / Huge / Shiny pills.
  - **The odds.** A dice icon with "1 in N" ("1 in ?" without odds).
  - **Stats section.** It replaces the lore text, which is removed at the user's request.
    - Each stat is a row with an icon, a label and a value, and the card grows with every row; with no stats there is no section.
    - Source: `GoblinPets.StatsOf(goblinId, mutations)` when that exists, else `design.Stats` (a list `{ {label, value, icon, iconColor, color} }` or a plain `{ Name = value }` table).
    - This is where the coming Damage/HP model plugs in.
  - **The "Owned: N" bar.**
  - The card is placed once per frame in a render step before `First` (no layout reads between writes).
- Where:
  - `StarterPlayerScripts.GoblinPetClient`, section "Tooltip (hover over a goblin)". It sits in a do-block; only `tipOwner`, `showTip`, `hideTip`, `withTip` (plus `GuiService` / `RunService`) are top-level.
  - Backup: `UI_Backup.GoblinPetClient_before_tooltip_v2`.
- Test: in Play, hovering showed the right content for Void Watcher (CELESTIAL / Normal / 1 in ? / Owned: 1) and Rainbow Huge Shiny King (coloured words, 4 pills, 1 in 25.6B). A temporary `StatsOf` test (Damage 65, HP 390, since removed) showed the stat rows with icons and the card growing. No errors.
- Still open: the real stats arrive with the damage/HP model.

## 2026-10-07 — HP icon designs for the goblins (Claude)
- What: four HP icon concepts on a design canvas, nothing built in the game yet.
  - Each one is shown big, at small sizes, and in an example goblin HP bar ("281 / 390").
  - The four concepts:
    1. Goblinhart: a red heart with green goblin ears.
    2. HP-schild: a green shield with a heart and "HP".
    3. HP-badge: a round heart with an "HP" pill.
    4. Kristalhart: a low-poly heart with a leaf, Fernwood style.
  - This is part of the goblin damage/HP model; that combat balance proposal is still waiting for the user's choices.
- Where: the private design canvas https://claude.ai/artifact/2MYyWRtwZqp3RaZM6Hg9Uo (only the user can open it).
- Test: none; these are designs only.
- Still open:
  - The user picks a concept.
  - It then gets made as an image/icon asset for the HP bars, inventory cards and goblin nametags.

## 2026-10-07 — Cutscene sound test panel removed (Claude)
- What: at the user's request, the "Cutscene geluid (test)" panel (`CutsceneSfxTest`) is gone. The chosen sounds stay as fixed defaults in `CUT_SFX`: build 7, reveal 8, title 8, count 2. The `CutSfx_*` attributes still override them if they are set, but no UI sets them anymore.
- Where:
  - `StarterGui.RollGui.RollClient`: the panel block was removed.
  - `ReplicatedStorage.RecordCutscenes`: only a comment changed.
  - Backup: `UI_Backup.RollClient_before_sfxpanel_removed`.
- Test: in Play the panel is gone and "New Best Roll (test)" is still there. A test cutscene (14.1 s) played all four default sounds. No errors.
- Still open: nothing.

## 2026-10-07 — New Best Roll: the Mix as default, the user's sounds, the end picture ready after the cutscene (Claude)
- What:
  - **Mix (style 6, the new default).** Each New Best Roll randomly plays Kristalgeode, Rode loper or Krachtexplosie, each equally likely. The test panel has a 6th button "Mix (2, 3, 4)"; picking 1–5 still forces one cutscene.
  - **The user's sounds are now the defaults** (`default` field in `CUT_SFX`):
    - Opbouw: 7, Magie: licht-spreuk;
    - Onthulling: 8, Magie: explosie 3;
    - Titel: 8, Magie: zilveren droom;
    - Odds tellen: 2, Harp.
    - The picker panel shows these as selected until another is clicked.
  - **Smoother transition.** The roll's end picture is built silently under the cover of the cutscene's last 0.9 s (`MutReveal.cutscene(slot, face, prepare)` → `MutReveal.finale(..., instant)`).
    - The words stand at full size straight away (word mode "instant", Shiny mode "instant"). The rainbow star is full size and the odds and name are final.
    - No dim, sounds or confetti during that build.
    - When the white flash (now 0.9 s) fades, everything already stands there; the hold continues from there. The extra flash, ring and sparks in the roll after the cutscene are gone.
- Where:
  - `ReplicatedStorage.RecordCutscenes`: `CUT_SFX` defaults, `MIX`, `MutReveal.cutscene(..., prepare)`.
  - `StarterGui.RollGui.RollClient`: playRoll, `MutReveal.finale(..., instant)`, `shinyReveal` "instant", `word` "instant", test panels.
  - Backups: `UI_Backup.RecordCutscenes_before_mix`, `UI_Backup.RollClient_before_mix`.
- Test: with no attributes set, the mix gave Rode loper and Kristalgeode ×2, and style 4 was tested directly. Each run is 13–14.5 s and all four default sounds play at their moments. Right when the cutscene ends, the HUGE / SHINY / RAINBOW words already stand at scale 1. A screenshot of the moment the flash fades shows the full end picture (words, star, odds, name). No errors.
- Still open: nothing known.

## 2026-10-07 — Cutscene sound picker: magic options added (Claude)
- What: 15 magic sound effects were added to the "Cutscene geluid (test)" picker, so the user has more to choose from. Their names start with "Magie:". All were checked to load.
  - **Opbouw:** options 6–9 — opladen, licht-spreuk, whoosh-spreuk, kristal-glinster.
  - **Onthulling:** options 6–9 — grote explosie, explosie 2, explosie 3, element-inslag.
  - **Titel:** options 7–10 — herrijzen, zilveren droom, kristal-glinster, twinkel.
  - **Odds tellen:** options 5–7 — glitter, diamant-bling, tik.
  - The panel now wraps at 5 buttons per line (300 × 480).
- Where: `ReplicatedStorage.RecordCutscenes` (`CUT_SFX`) and `StarterGui.RollGui.RollClient` (panel `CutsceneSfxTest`).
- Test: in Play the panel shows 35 buttons. A Kristalgeode run with magic picks 6/6/7/5 played all four sounds. No errors.
- Still open: the user names the numbers they like; those become the defaults (index 1).

## 2026-10-07 — New Best Roll: choose the sounds yourself, odds in full digits, no shade (Claude)
- What:
  - **The user picks the cutscene sounds.** There are four roles, each with a few options:
    - **Opbouw:** the tension, timed to end at the big moment;
    - **Onthulling:** the big moment;
    - **Titel:** under the title card;
    - **Odds tellen:** while the odds count up.
  - The new test panel "Cutscene geluid (test)" (next to the New Best Roll panel) has numbered buttons. A click plays the option and picks it (player attributes `CutSfx_build` / `CutSfx_reveal` / `CutSfx_title` / `CutSfx_count`, default 1). Options marked "(oud)" are the earlier choir / harp sounds.
  - The other cutscene sounds are neutral again (whoosh, slam, clobber, the UFO's own sounds); the divine choirs and bells are gone.
  - **Title card:** the dark shade behind the text is gone. The odds and the previous best show in full digits ("1/25,600,000,000"), with no K / M / B / T.
- Where:
  - `ReplicatedStorage.RecordCutscenes`: `CUT_SFX`, `playRole`, `buildUp` / `revealSnd` / `startTitle`, `MutReveal.previewCutSfx`.
  - `StarterGui.RollGui.RollClient`: panel `CutsceneSfxTest`.
  - Backup: `UI_Backup.RecordCutscenes_before_sfxpicker`.
- Test: in Play, with picks 2/2/3/1, all four role sounds played at the right moments (riser from 1.1 s so it ends at the reveal, drum roll from 3.4 s so it ends at the odds slam). No errors.
- Still open: the user listens and names the numbers they want; those become the defaults (the default index 1 in `CUT_SFX`). They can also send their own Creator Store audio ids to add as options.

## 2026-10-07 — New Best Roll cutscenes: 5 new ones, an animated goblin, a title card, divine sound, Hide Cutscenes (Claude)
- What: this replaces the earlier Rainbow cutscenes and the first 5 record cutscenes; the user liked none of them.
  - **When:** a roll that beats the player's rarest roll ever and is at least 1/100k.
    - Server: `GoblinInventory.OddsOf` / `RecordRoll` with `bestOdds` in the profile.
    - `RollServer` sets `result.cutscene = true` and `result.prevBest`.
  - **Order (the user's call):**
    1. The roll lands.
    2. The cutscene plays straight away.
    3. Then only the roll's end picture shows (`MutReveal.finale`): the HUGE / SHINY / RAINBOW words, the rainbow star, the odds and the full name, with no build-up animations.
  - **Settings > Hide Cutscenes** (new toggle, saved in the profile as `settings.hideCutscenes`): no cutscene, and the normal reveals play instead (Huge 4 Titaan-zoom, Rainbow 3 Regenboogvortex, Shiny).
  - **Stage:** each cutscene builds its own set high above the map (y 1800), on the client only.
    - It sets its own Lighting, Atmosphere, Bloom and DepthOfField, all restored afterwards.
    - Black bars and a white flash in and out.
    - The controls and the other GUIs are off while it plays.
  - **The goblin is a real animated pet:** new `GoblinPetRenderer.Puppet(goblinId, mutations, scale, parent)`.
    - It is levelled with its feet, so it no longer stands crooked.
    - It walks, runs, idles and waves, plays its own personality gesture and FX, and has its mutation looks and Huge size.
    - Extra poses come through `pet.poseHook`, a new hook in `GoblinPetFX.PoseUpdate`: cheer, jump, tuck, power, roar, flail, land, dizzy, star.
    - Its head follows the camera.
  - **The 5 cutscenes** (player attribute `CutsceneStyle`):
    1. **Schatkist:** a gold chest falls from the night sky, rattles 3 times (light leaks out, closer shots), bursts open (a light column and a coin fountain), and the goblin jumps out with a spin.
    2. **Kristalgeode:** crystals sprout in a spiral; a crystal bud with the goblin curled up inside rises, gives three heartbeats, cracks and shatters, and the shards keep floating around it.
    3. **Rode loper:** a red carpet rolls out, the post lights switch on, and the backdrop shows the goblin's name in neon. The goblin walks down the carpet through flashes and strikes a pose.
    4. **Krachtexplosie:** close-ups of the feet, fist and face. Rocks float, an aura and lightning build up, then a pillar of light, shockwaves and floor cracks; the goblin hovers.
    5. **UFO-bezorging:** a saucer zig-zags in; the goblin floats down in the tractor beam, spinning and wiggling, lands dizzy, and the saucer shoots off into the stars.
  - **Title card:**
    - NEW BEST ROLL! in the top bar.
    - The name word by word: Rainbow (live rainbow), Huge (purple), Shiny (gold), then the name in its rarity colour.
    - The odds count up digit by digit and slam down (rainbow for Rainbow).
    - A rarity pill and "PREVIOUS BEST 1/…"; the "x… RARER" text was removed at the user's request.
    - Rays, sparkles and glints.
  - **Sound:** divine only (the user: no applause, crowd or shutter clicks).
    - A choral bed (`lightDark`) under the build-up, and a celestial choir (`adAstra`) under the title.
    - Angel choir, harp glissandi, bells, a bell tree, glockenspiel and airy wind whooshes.
    - The new ids are in `SFX_IDS` in RollClient; all were checked to load.
- Where:
  - `ReplicatedStorage.RecordCutscenes` (new module; RollClient calls it once with its helpers, in a pcall).
  - `ReplicatedStorage.GoblinPetRenderer`: `Puppet`, `YawOf`, shared `poseBones`, `pet.air`.
  - `ReplicatedStorage.GoblinPetFX`: `poseHook`.
  - `StarterGui.RollGui.RollClient`: playRoll order, `MutReveal.finale`, `shinyReveal(..., "end")`, SFX ids, panel "New Best Roll (test)".
  - `ServerScriptService.RollServer`: `prevBest`.
  - `ServerScriptService.GoblinInventory`: `hideCutscenes` setting.
  - `StarterPlayerScripts.GoblinPetClient`: Settings row 4; the panel is 478 high.
  - Backups in `UI_Backup`: `RollClient_before_cutscenes_v2`, `RollClient_before_finale`, `GoblinPetRenderer_before_puppet`, `GoblinPetFX_before_puppet`, `RollServer_before_prevbest`, `GoblinInventory_before_hidecutscenes`, `GoblinPetClient_before_hidecutscenes`, `RecordCutscenes_v2_first`.
- Test:
  - The "New Best Roll (test)" panel (left) picks the style and has "Test: New Best Roll".
  - Hook: `RollGui` attribute `StudioTestReveal = "Record"`.
  - All 5 ran without errors, 12–13.7 s each, and restore the camera, lighting, walk speed and GUIs.
  - Render CPU averaged 2.3–2.8 ms per cutscene (peaks 4–15 ms when the set or puppet spawns); 125–165 fps when Studio is in the foreground.
  - With Hide Cutscenes on: no cutscene, and the normal reveals play (HUGE 2.8 s, SHINY 4.2 s, RAINBOW 5.9 s after the roll).
  - A real ≥1/100k record was flagged by the server (forced mutation test).
- Still open:
  - The user picks a default style (now 1).
  - The cutscenes were judged from stills only; the user should look at them in Play.
  - Ideas: a skip button, and one cutscene per rarity tier.

## 2026-10-07 — Rainbow cutscenes (3, in the 3D world) + cutscene test panel (Claude)
- What: three Rainbow reveal styles that are real cutscenes, RainbowReveal 9-11 (the user: "30000000000x cooler, with a cutscene"). The camera takes over (Scriptable) and the moment plays in the world in front of the player:
  - **Setup:**
    - black letterbox bars;
    - a client-side `ColorCorrectionEffect` in Lighting grades the world;
    - the other ScreenGuis and the roll UI hide (only the flash layer stays);
    - the player stands still (WalkSpeed / JumpPower / JumpHeight 0, restored after);
    - the rolled goblin is cloned from its template at 2-2.2x its pet size, with GoblinMutationFX's Rainbow skin and sparkles (Attach / Step / Detach).
  - **Afterwards:** the camera glides back, everything is restored, then the roll shows the RAINBOW finale.
  - **The cutscenes:**
    1. Hemelinslag: dark grey world, light gathers in the sky (heartbeat, riser); a colour-cycling rainbow beam strikes (explosion + bass drop, three ground shockwaves, a 170-particle rainbow blast, camera shake); the goblin rises out of the light, the beam narrows away, the camera circles it with a choir.
    2. Regenboogbrug: a 7-band rainbow (36 x 7 neon segments) builds across the sky; the goblin slides down it with a star trail while the camera tracks it from the side; a heavy landing, the bridge fades band by band, an orbit.
    3. Dimensiescheur: the world goes colourless, a vertigo dolly-zoom; a portal (28 turning colour stones, a flickering core with particles, crackling lightning) tears open; the goblin steps out; colour floods back as the portal implodes and explodes, then an orbit.
  - **Rules:**
    - Skipped (style 3 plays) while a menu panel is open or without a character.
    - With several columns it plays once (the other slots wait, `Cut.playing`).
    - Never sped up (~7 s; the whole roll ~11.6 s).
    - Setup and teardown run in pcall; on an error the camera goes back to Custom.
  - **Panel** "Rainbow cutscene (test)" (left, next to the dock): Uit (stijl 3) / 1 Hemelinslag / 2 Regenboogbrug / 3 Dimensiescheur, plus "Test: Rainbow roll". It shows with the admin panels. The Rainbow row in the Mutation reveal panel shows the cutscene name when one is picked.
- Where: `StarterGui.RollGui.RollClient` (`MutReveal`: block "RAINBOW CUTSCENES", `RAINBOW[9..11]`, `style()` up to 11; the panel inside the Mutation reveal panel block). Backup: `UI_Backup.RollClient_before_cutscenes`.
- Test:
  - All three play and restore: camera Custom, FOV 70, the scene folder gone, the menus on again, WalkSpeed 16.
  - Performance during the cutscenes: 165 fps, worst frame 9-17 ms.
  - Found and fixed: this game's PlayerModule is not in PlayerScripts (WaitForChild hung forever), so the Humanoid is frozen instead.
  - Note for testing: the Assistant's execute_luau resets the camera type at the end of a call when a cutscene changed it during that call; keep a whole cutscene inside one call.
- Still open: the user picks a cutscene (or keeps style 3) as the default.

## 2026-10-07 — Shiny reveal readable at high roll speeds (Claude)
- What: at roll speed 4x the Shiny reveal played ~6x faster and the roll did not wait for it. Now:
  - The Shiny reveal never plays faster than 0.45 of its length (`kShiny = max(k, 0.45)`), alone and in combinations.
  - `playRoll` waits for every mutation reveal (`slot.revealing = true` for any mutation). Shiny alone gets 0.5 x kShiny extra to look at.
  - A mutation without its own reveal (Glitched later) does not block the roll.
  - Rolls without a mutation are unchanged.
- Where: `StarterGui.RollGui.RollClient` (`MutReveal.play`, `playRoll`).
- Test at RollSpeed 4x:
  - Shiny: the roll takes 2.19 s; SHINY appears at 1.04 s and stays 1.15 s.
  - Huge+Shiny: 2.87 s; SHINY stays 0.90 s.
  - Output clean; RollSpeed set back to 1x.
- Still open: if it should stay longer, raise 0.45.

## 2026-10-07 — A Rainbow switches Hide Rolls off; Rainbow chance 1/5,000 (Claude)
- What:
  - **Hide Rolls:** when a roll result is a Rainbow while Hide Rolls is on, Hide Rolls switches off at the start of the roll (the full-screen roll comes back for the whole animation). This does not happen while a menu panel is open (GoblinMenuGui `PanelOpen` not empty) or while the player types in a TextBox (chat). It stays off afterwards.
  - **Chance:** Rainbow is now 1/5,000 (`GoblinPets.MUTATIONS`, was 12,800). All odds texts, EquipBest and the roll derive from it.
  - **Test hook (Studio only):** the RollGui attribute `StudioSetHidden` = true / false switches Hide Rolls (simulated mouse clicks don't reach Studio's game buttons).
- Where:
  - `StarterGui.RollGui.RollClient`: the start of `playRoll`; `MutReveal.setHidden` is set after `setHidden`; the hook.
  - `ReplicatedStorage.GoblinPets`.
- Test:
  - Hide Rolls on: Huge → stays hidden; Rainbow with the inventory open → stays hidden; Rainbow with no menu → Hide Rolls off.
  - `MutationOdds("Rainbow")` = 5000.
- Still open: none.

## 2026-10-07 — Mutation words stay on screen: the roll moves down for a moment (Claude)
- What: when a roll has mutation words (HUGE / SHINY / RAINBOW, also Shiny alone) that would reach above the screen, the roll stage moves down for a moment, exactly as far as needed (`makeRoomForWords` at the start of `MutReveal.play`; 0.3 s Back tween). This mostly happens with Hide Rolls (the roll sits high) or with three stacked words.
  - The space needed: (-CY + SLOT_SIZE x wordBase + (words - 1) x 48 + 64) x stage scale above the stage centre; the centre lies at screen height x HIDDEN_Y / SHOWN_Y (RollGui ignores the GUI inset).
  - Several columns: the largest shift wins.
  - After the goblins left, `playRoll` puts the stage back (`placeStage(false)`, `MutReveal.shift`).
- Where: `StarterGui.RollGui.RollClient` (`MutReveal`, the end of `playRoll`).
- Test:
  - Normal roll, Huge+Shiny+Rainbow: it moved 21 px and was back in place afterwards.
  - Hide Rolls, Huge+Shiny+Rainbow: it moved 84 px; the top of RAINBOW was 10 px below the screen top with the old 44 margin (now 64).
- Still open: none.

## 2026-10-07 — Reveals fit the screen with Hide Rolls (Claude)
- What: with Hide Rolls on (the roll plays small at the top of the screen) the big rainbow star of Rainbow 3 stuck out above the screen. Now `hs(n)` halves the big backdrops when `hidden`:
  - Rainbow 3: the vortex rings and the giant star;
  - Rainbow 5: the beam fan and its star;
  - the giant word slam: 2x → 1.2x (was 3.4x → 1.6x);
  - the Huge 4 camera zoom: only 35 % of the zoom.
  - Normal (full-screen) rolls are unchanged.
- Where: `StarterGui.RollGui.RollClient`, `MutReveal` (`hs`, `word`, `zoom`, `RAINBOW[3]`, `RAINBOW[5]`).
- Test: Rainbow 5 and Huge+Rainbow (H4 R3) test rolls run without errors (5.6 / 7.6 s). The Hide Rolls view itself was not tested by script (the user checks it).
- Still open: none.

## 2026-10-07 — Rainbow reveal: the big star (and other backdrops) behind the goblin (Claude)
- What: a new layer `MutBack` behind the whole roll slot:
  - a Frame in the stage at ZIndex 0, under the slot's CanvasGroup (ZIndex 1);
  - destroyed with the slot, and it fades out with the effects when the goblins leave.
  - It holds the backdrops:
    - the giant spinning rainbow star (the user: "behind the goblin and behind everything") and the vortex rings of Rainbow 3;
    - the beam fan and star of Rainbow 5;
    - the light column of Rainbow 7;
    - the aurora curtains of Rainbow 8.
  - Words, sparks, rings and confetti stay in front.
- Where: `StarterGui.RollGui.RollClient`, `MutReveal` (`backLayer`); the exit loop in `playRoll` also fades `slot.back`.
- Test: Rainbow test roll; the backdrop sits in MutBack (ZIndex 0 vs slot 1); the stage is empty after the roll; Output clean.
- Still open: the user's message ended with "Ook bij" (cut off), ask what else they meant.

## 2026-10-07 — Reveals: names appear with their animation, Huge glow, picks as defaults, Rainbow 3 new ending, base roll 1.3x slower (Claude)
- What:
  - **Names:** the mutation words are added to the roll name only when their animation reveals them (`revealName`, `slot.shownMuts`).
    - Huge at its HUGE word, Shiny when the Shiny reveal writes its word, Rainbow at its RAINBOW word. Before that the plain name shows.
    - Measured: "Hoarder" → "Huge Hoarder" (2.5 s) → "Huge Shiny Hoarder" (3.9 s) → "Rainbow Huge Shiny Hoarder" (5.5 s).
  - **Rainbow end:** the final picture stays 1.6 x max(k, 0.6) + 0.4 s longer, so the odds can be read.
  - **Huge:** no purple outline copies any more (they showed as purple patches in the ears). Instead a soft purple glow behind the goblin (`HugeGlow`, puff 1.35x the goblin, breathing to 1.55x).
  - **Defaults** (the user's picks): Huge style 4 Titaan-zoom, Rainbow style 3 Regenboogvortex (`MutReveal.DEFAULT_STYLE`, used when the player attribute is not set). The other styles stay in the test panel for now.
  - **Rainbow 3:** the six colour beams at the end are replaced by a giant spinning rainbow star behind the goblin, three rainbow shockwaves and 30 rainbow stars raining down round it.
  - **Base roll speed** 1.3x slower: `BASE_SPEED` = 2 / 1.3 (was 2). Every roll timing (cycle, hold, reveals) is 1.3x longer at speed 1x; the RollSpeed upgrade still multiplies on top.
- Where: `StarterGui.RollGui.RollClient` (`MutReveal` block, test panel, `BASE_SPEED`).
- Test: test rolls Huge / Rainbow / Huge+Shiny+Rainbow: 5.0 / 6.5 / 8.8 s; names as above; Output clean.
- Still open: once the user is happy, the unused styles can be removed.

## 2026-10-07 — Huge + Rainbow roll reveals v2: 8 new styles each with SFX; Rainbow has top priority (Claude)
- What (the user: Rainbow 3 and Huge 2 were the best but "not cool enough"; Rainbow must stun people):
  - **Order:** Huge → Shiny → Rainbow LAST.
    - The name puts Rainbow first everywhere: `GoblinPets.DisplayName` moves "Rainbow" to the front ("Rainbow Huge Shiny Zeus"); the roll name is coloured the same way.
    - The words stack HUGE / SHINY / RAINBOW (the Shiny word now takes its own place).
    - A Rainbow result gets the inventory's moving rainbow odds: white text, a gradient `fromHSV((i/6 - t*0.35) % 1, 0.6, 1)`, stroke (30,20,50).
    - A rainbow light runs along the rays already while a Rainbow result cycles.
  - **Huge 1-8** (`HugeReveal`, all built on "inflate", ending with a purple outline + HUGE):
    1. Opblazen XL: wobbly bulging pulses with poofs, trembling at bursting point with a riser, bass-drop POP, shockwave.
    2. Kolos: dim, heartbeat, 3 heavy footsteps (each bigger, shake + dust), cinematic boom.
    3. Meteoriet: launched out of view, a purple meteor with a trail crashes in: explosion, crater ring, debris, big shake.
    4. Titaan-zoom: the camera zooms in (heartbeat), then slams out while it bursts to giant: bass drop, giant HUGE.
    5. Onweer: dark sky, 3 purple lightning strikes, each grows it a step, the last a double strike with thunder.
    6. Breekt los: a purple box closes round it, it presses the walls out, the box shatters into shards.
    7. Aardbeving: a rumble builds (shake, rising dust, cracks from its feet) while it swells; a boom.
    8. Energie-overload: purple orbs fly in from all sides (rising tones, a step each), overload flash + bass drop.
  - **Rainbow 1-8** (`RainbowReveal`; the whole screen joins in; ends with a rainbow outline + rainbow rays + rainbow odds + 26 sparks + confetti over the screen + a giant RAINBOW that slams down into its place):
    1. Kleurgolven XL: dark, heartbeat, 12 colour waves over the whole screen, faster with rising chimes; a white flash and a choir.
    2. Supernova: a black silhouette with rainbow light leaking out, rays spinning faster (riser), implosion, explosion with 3 rainbow shockwaves.
    3. Regenboogvortex: a 3-ring rainbow vortex sucks in stars, bursts, 6 colour beams shoot up.
    4. Kleurstorm: pitch dark, 6 coloured lightning bolts in turn (screen flashes in each colour), then all at once.
    5. Prisma-explosie: a white beam, then 12 rainbow beams over the whole screen keep turning with a giant spinning rainbow star; letters one by one.
    6. Vuurwerkfeest: confetti cannons from the bottom corners, 6 fireworks round the goblin, letters bounce.
    7. Hemelpoort: almost black, a light column comes down with a choir, stars float up, it opens into a rainbow.
    8. Aurora: rainbow aurora curtains sweep over the screen with chimes, then a crisp flash.
  - **SFX:** 22 free Creator Store sounds (all checked to load), folder `SoundService.MutRevealSfx` (created at runtime): booms, a heavy hitter, explosion, bass drops, sub drop, a reverse riser (`riser(at)` ends exactly on the impact), charge, chimes, correct/prize stings, magic, curaga, angelic choirs, poof, hiss, heartbeat. One play per 0.12 s per sound (columns don't stack).
  - **Speed / columns:**
    - Rainbow never plays faster than 0.6, Huge never faster than 0.45 of its length.
    - Dim, screen flash, confetti, zoom and shake play once for all columns.
    - Huge grows 1.28x with several columns (1.45x alone).
    - `playRoll` waits for the reveals.
  - **Test panel:** 8 style buttons per mutation (2 rows) with names. The Studio hook (RollGui attribute `StudioTestReveal`) stays.
- Where:
  - `StarterGui.RollGui.RollClient`: the `MutReveal` block before `playRoll`; the Shiny finish word; the test panel.
  - `ReplicatedStorage.GoblinPets` (`DisplayName`).
  - Backup: `UI_Backup.RollClient_before_reveals_v2`.
- Test:
  - All 16 styles and Huge+Shiny+Rainbow / Shiny+Rainbow played without errors (Huge 3.8 s per test roll at speed 1, Rainbow 3.8–4.6 s, all three 6.9 s).
  - Performance with 5 columns and all three mutations: 165 fps on average, worst frame 12 ms, render CPU 2.7–3.2 ms.
  - Screenshots can't catch the animations (the capture arrives seconds late).
- Still open: the user picks one style per mutation; that becomes the default (the others can go).

## 2026-10-07 — Huge + Rainbow roll reveals, 4 styles each to choose from (Claude)
- What: after the landing a mutated roll plays its reveal (`MutReveal` in `RollClient`).
  - **Order:** Rainbow first, then Huge, then the existing Shiny reveal. Each phase stamps its word (RAINBOW / HUGE / SHINY stack upwards).
  - **Name and odds:** the name gets every mutation in colour (Huge lilac, Shiny pale gold, Rainbow letter by letter). The odds count up once to the total (the Shiny reveal does it when it plays).
  - **Huge styles** (player attribute `HugeReveal`). The goblin ends 1.45x, or 1.28x with several columns; the black outline becomes a purple one.
    1. Groeispurt (the user's idea): a deep boom, two hop-grows, the view pulls back, the purple rim thickens, landing with a screen shake + dust, HUGE stamped.
    2. Opblazen: three purple pulse rings while it swells and wobbles, then POP (flash, ring, sparks), HUGE drops in.
    3. Schaduw: a giant purple shadow rises behind it, purple sparks drift up, it grows smoothly into the shadow, thud, HUGE rises.
    4. Uit de lucht: it jumps out of view and lands back huge with a heavy impact (big shake, dust ring), HUGE stamped tilted.
  - **Rainbow styles** (`RainbowReveal`). The goblin keeps its own colours and gets a rainbow outline (12 copies, every one its own hue, slowly turning) plus rainbow rays.
    1. Kleur bouwt op (the user's idea): a rainbow light runs along the rays already during the cycling; a coloured band turns round the goblin, red → purple flash in turn, the band closes into the rim, a bright ting, RAINBOW with a sheen.
    2. Prisma: a white beam falls onto the goblin and fans out into 6 coloured rays; RAINBOW pops letter by letter in colour.
    3. Kleurgolven: one ring per colour ripples outward (each with a rising tone).
    4. Regenboogboog: a rainbow arc draws itself over the goblin from left to right with sparkles at the ends; RAINBOW slides in.
  - **Roll speed / columns:**
    - Every duration scales with the roll speed (k = 1 / speed).
    - The screen shake plays once when several columns land together.
    - `playRoll` waits until the reveals are done (`slot.revealing`) before the goblins fly off; normal and Shiny-only rolls keep their timing.
  - **Sounds:** the existing ting, pitched: very low = the boom, high = the bright tone.
- **Test panel** "Mutation reveal (test)", left of the roll test panel (Studio, or admins with /admin):
  - Huge stijl 1-4 and Rainbow stijl 1-4, with the style's name.
  - Test buttons Huge / Rainbow / Rainbow Huge / Shiny Huge / Rainbow Shiny / Alle 3: a display-only test roll (nothing awarded) with the set RollColumns and RollSpeed.
  - Studio hook: the RollGui attribute `StudioTestReveal` = "Huge,Rainbow".
- Where: `StarterGui.RollGui.RollClient` (block "Huge + Rainbow reveals" before `playRoll`; the panel before the luck clover section). The Shiny finish uses `slot.mutName` / `slot.wordY`. Backup: `UI_Backup.RollClient_before_mutation_reveals`.
- Test: all 8 styles and Huge+Shiny+Rainbow ran without errors (3.8–4.7 s per test roll at speed 1). Screenshots could not catch the animation (the capture arrives seconds late), so the user judges the looks.
- Still open: the user picks a style per mutation; the chosen one then becomes the default (and the others can be removed). Glitched later.

## 2026-10-07 — Index Normal filter: star with a thinner outline (Claude)
- What: the Normal filter button in the Index uses its own yellow shard with a thinner outline, `rbxassetid://88446795752223`.
  - The yellow shard's dark outline was measured at 7 px against the white (Shiny) shard's 5 px; the outer 2 px of alpha were peeled off, so both outlines are now the same width.
  - All other stars (cards, equipped bar, other filters) keep `CARD.SHARDS`.
- Where: `GoblinPetClient`, the Index `FILTERS` table. Image: `HUD_Mockup/shards/shard_yellow_thin.png`, made by `HUD_Mockup/thin_yellow_shard.py` (Blender python).
- Test: open the Index; screenshot checked.
- Still open: none.

## 2026-10-07 — Index filter outline 2 px (Claude)
- What: the picked mutation filter's near-black outline is 2 px (was 3.5); the shadow on the other buttons now shows which one is picked.
- Where: `GoblinPetClient`, `renderIndex` (`fb.edge.Thickness = 2`).
- Test: open the Index and click a filter.
- Still open: none.

## 2026-10-07 — Index filters: the others fade under a shadow (Claude)
- What: the mutation filter buttons that are not picked fade (0.25 s) under a dark shadow: a `Dim` frame over the face, color (20, 8, 40), transparency 0.5. The picked one stays bright with its 3.5 px dark outline.
- Where: `GoblinPetClient`, the Index filter buttons (`IDX.filterButtons[key].dim`) and `renderIndex`.
- Test: open the Index and click Shiny / Huge / Rainbow / Normal; screenshot checked.
- Still open: none.

## 2026-10-07 — Index filter outline thinner (Claude)
- What: the picked mutation filter's near-black outline is 3.5 px (was 5); the others stay 2 px.
- Where: `GoblinPetClient`, `renderIndex` (`fb.edge.Thickness`).
- Test: open the Index and click a filter.
- Still open: none.

## 2026-10-07 — Index: single filter, black outline, quick book (Claude)
- What:
  - **Filters:** the Index shows one mutation at a time: Normal, Shiny, Huge or Rainbow; combinations are no longer possible.
  - **Picked filter:** a thick near-black outline (18, 6, 34; 5 px) and 1.05x; the others a thin, half see-through outline (2 px, 0.45). No white glow.
  - **Book animation (`IDX.playBook`), ~0.65 s:**
    1. The book grows from the dock to the middle (0.22 s).
    2. The cover opens once (0.2 s), with a soft glow.
    3. The Index comes up out of the open book: from 0.8 scale, Quad, no bounce. The book grows to 1.5x and fades.
    - No vignette, page flips or rays any more.
- Where: `StarterPlayer.StarterPlayerScripts.GoblinPetClient` (`IDX` block, `open()`).
- Test: Studio was reopened, so its studio id changed; the earlier changes of today were all still there. Index opens; the book shows 0–0.65 s; outlines Normal 5 / others 2; screenshot checked.
- Still open: none.

## 2026-10-07 — New Index with mutation filter + book opening animation; bigger odds on icons (Claude)
- What:
  - **Index** (after the user's reference, not 1:1; `GoblinPetClient`, block "Index (2026-10-07…)", table `IDX`). The panel is 980 x 604, purple; makePanel's sheet is hidden. Parts:
    - top: the dock's book before the title "Index", a purple search bar ("Search goblins...") and X;
    - left: "Filter by Mutation" with Normal / Shiny / Huge / Rainbow (shard icons from `CARD.SHARDS`). Normal = no mutation; Shiny, Huge and Rainbow toggle and combine, e.g. Huge + Shiny + Rainbow. The picked button grows 1.05x with a white glow and outline;
    - middle: 42 cards (`IDX.card`) in the inventory card style. The rim belongs to the mutation (thickRim / shinyBand / hugeBand / rainbowBand, the same effects), the name sits above the hill and the odds on the hill. Not discovered: a black silhouette and "???". A click picks the card (cyan ring). The first 16 cards are built right away, the rest within frames;
    - right: the picked goblin as a big card, its name, a rarity pill, Discovered ✓ / Not discovered ✗, "1 in 4.1B" and the description;
    - bottom: the book, "Discovered n/42" for the current filter with a progress bar, and a Collection Rewards box (🎁, no rewards yet).
  - **Discovery per mutation:** the server index now also stores `"<id>|<mutations>"` (Award, and on load for every owned stack). Normal = the design id, as before.
  - **Card effects:** they run while the Inventory **or** the Index is open (`CARD.inventoryOpen`).
  - **Book animation** (`IDX.playBook`, every time the Index opens), ~0.75 s:
    1. The dock's book (two halves of its atlas image) flies from the dock button to the middle of the screen (half the screen height), growing and closed.
    2. It flaps open with a purple glow, turning light rays and 14 sparkles.
    3. The Index pops in while the book zooms past and fades.
    - `open()` waits for it via `show()`.
  - **Odds on icons:** bigger and rounded to 1 decimal.
    - Inventory cards: max text 23 px in a 0.94 x 0.25 box (was 18 / 0.86 x 0.2).
    - Equipped bar: 21 px (was 17).
    - `GoblinPets.RatioText(rarity, mu, decimals)` (default 2, so the nametags are unchanged; the icons pass 1): "1/163.84M" → "1/163.8M".
- Where:
  - `StarterPlayerScripts.GoblinPetClient` (the old small Index panel was removed);
  - `ReplicatedStorage.GoblinPets` (`RatioText`);
  - `ServerScriptService.GoblinInventory` (index keys).
- Test: open the Index from the dock (or the GoblinMenuGui attribute `StudioOpenPanel` = "Index"); toggle the filters; give mutations with the admin panel. Screenshots checked (Index + the open book).
- Still open: Collection Rewards have no content yet. Combinations discovered before today only count from now on (the old index only knew design ids).
- Follow-up (same day, the user's feedback):
  - **Not discovered:** the card shows "1/?" and the detail box "1 in ?".
  - **Books:** the title book is centred on the title line inside the frame (74 x 53, -6°); the bottom book is centred in its box.
  - **Progress bar:** 44 px tall with a light top band and the count in the same chunky 30 px outlined font as "Discovered n/42".
  - **Book animation, improved** (~1.3 s):
    1. A dark vignette fades in. The closed book (only the cover half, in the middle) flies from the dock on an arc, turning and growing, and lands with a squash.
    2. The cover swings open while the spread slides to the middle.
    3. Two pages turn over from right to left.
    4. Flash, rays and 16 sparkles.
    5. The Index pops in while the book zooms past and fades.

## 2026-10-07 — "Infinite" inventory: 20 million goblins, one counter per stack (Claude)
- What: the inventory no longer stores one entry per goblin. Each stack (design + mutations, key `"Zeus"` / `"King|Huge,Shiny"`, the same key as the cards) is one counter. So data size, snapshot, network and UI never grow with the number of goblins: at most 336 stacks, a snapshot of ~0.5 KB even with 19 million goblins. `GoblinPets.MAX_INVENTORY` = 20,000,000 (total).
  - **Data v2** (`ServerScriptService.GoblinInventory`):
    - `stacks = { [key] = { g, mu, n, t = newest, f = first } }`.
    - `equipped` = a list of stack keys (one key can appear up to n times).
    - v1 profiles (items + instance ids) are converted in `sanitize` on load and nothing is dropped: equipped ids become keys, duplicates are removed, unknown designs are kept.
    - **Don't go back to the old inventory script once real data is converted** (it would not read `stacks`). The backup `UI_Backup.GoblinInventory_before_stacks` is for reading only.
  - **Server API:**
    - `Award` returns the stack key (RollServer's `instanceId`).
    - New: `AwardCount(player, id, mutations, count)` and `Inventory.StackKey`.
    - `AwardMany` and `FreeSlots` count totals.
    - `Delete(player, { {k = key, n = count}, ... })` deletes free copies only.
    - `SetEquipped(id, equip)`: id = a stack key (equip one more copy) or a slot id `"<key>#<k>"` (take that copy off). The message is "All of them are equipped already" when there is no free copy.
    - `EquipBest` takes several copies of the best stack.
    - The `EquippedGoblins` attribute keeps `{ i = "<key>#<k>", g, m }`, so the renderer and combat are unchanged.
  - **Snapshot:** `{ stacks = { {k, g, mu, n, t, f} }, equipped = { {i, g, m} }, owned = total, capacity }`.
  - **Client (`GoblinPetClient`):**
    - Cards are built from `snapshot.stacks` (`invUI.stackOf[key]` = the latest stack).
    - Clicks and tooltips read the latest counts.
    - The card signature no longer contains the count: a new copy only updates the "x12" label (`CARD.countLabels`, `CARD.countText`: up to 9,999 in full, then 12.3K / 1.2M / 20M), so Auto Roll doesn't rebuild cards.
    - The tooltip shows "Owned: 19,000,000".
    - The delete slider and Delete (All) work with counts.
- Where: `ServerScriptService.GoblinInventory`, `GoblinInventoryServer` (Studio grant), `ReplicatedStorage.GoblinPets` (`MAX_INVENTORY`), `StarterPlayerScripts.GoblinPetClient`. Backups: `ServerStorage.UI_Backup.*_before_stacks`.
- Test:
  - Studio grant: the ServerStorage attribute `StudioGrantGoblins` = `"Zeus*19000000,King:Huge+Shiny*5000"` (new `*count`).
  - Checked:
    - owned 19,005,004 in 4 stacks, snapshot 563 bytes, cards x19M / x5000 / x3;
    - equipping 3 copies of one stack (a 4th is refused) and taking off copy #2;
    - EquipBest with 6 copies of Zeus;
    - deleting 1,000,000 at once (equipped copies stay);
    - a real roll (+1 on its stack);
    - v1 → v2 conversion of a sample profile (also run twice, JSON OK);
    - 336 stacks with the inventory open while rolling: 334 cards kept (only 1 → 2 rebuilt), worst frame 13 ms.
- Still open: test DataStore saving with "Enable Studio Access to API Services" (in Studio it only runs memory-only).

## 2026-10-07 — Huge: ground circle 1.3x smaller (Claude)
- What: in `FX.Rescale` the aura part (`FXAura`, the ground circle) now grows by `k / 1.3` instead of `k`.
  - Note: the model's `ScaleTo` already grows the aura, and Rescale grows it again, so a Huge circle was 2.56x a normal one.
  - It is now 1.97x (Zeus: 3.80 → 7.48 studs, was ~9.7).
- Where: `ReplicatedStorage.GoblinPetFX` (`FX.Rescale`).
- Test: equip a normal and a Huge Zeus; compare the size of `FXAura`.
- Still open: none.

## 2026-10-07 — Rainbow goblins: white glow 1.4x softer (Claude)
- What: in `stepRainbow` the white glint sweep over the skin now lerps to white at `0.85 / 1.4` (was 0.85). The Rainbow PointLight Brightness is now `1.2 / 1.4` (was 1.2; Range 7 unchanged). The white glint sparkles, outline and colours are unchanged.
- Where: `ReplicatedStorage.GoblinMutationFX` (`stepRainbow`, `attachRainbow`).
- Test: equip a Rainbow goblin; the glint and the light should be visibly softer.
- Still open: if it should be even softer, or only one of the two, change those two numbers.

## 2026-10-07 — Inventory and pet-spawn lag, round 2 (Claude)
- What: the user still had lag when opening the inventory (many Rainbows) and frame drops when pets spawned. Measured live, with rendering on.
  - **Inventory, off-screen cards (`GoblinPetClient`, "Off-screen cards" block after the card visibility pass):**
    - Cause: the open animation (the panel's UIScale pop) re-computed the layout of every GUI object each frame. With 336 stacks that is ~15,000 objects and 35–50 ms frames. Hiding them with `Visible` does not help.
    - A card far outside the visible part of the list now keeps only its empty button, so the grid layout and the scroll bar stay the same. Its children are detached (`CARD.detach` / `CARD.attach`).
    - The content comes back before the card scrolls into view: a margin of one list height above and below, in BindToRenderStep "GoblinCardVirtual" (First − 11). It runs when CanvasPosition, the viewport, a render or the search changes (`CARD.virtDirty`).
    - Cards built later in a big build are created detached (`makeCell(..., true)`).
    - `CARD.owner` maps detached content to its card, so `cellOf` / `CARD.vis` keep working. A destroyed card destroys its detached content.
    - The rainbow odds are now applied right after the odds label is made (the old `task.defer` lookup missed detached content).
    - A first build makes 30 cards right away (48 in grid view; was 56).
  - **Pets (`GoblinPetRenderer`):**
    - Cause: every frame in which a MeshPart near the camera (< ~60 studs) changes size costs ~10 ms of render CPU, however many parts change. Plain Parts and moving are free; CastShadow, CollisionFidelity, RenderFidelity, SurfaceAppearance and lights make no difference. The pop-in (`ScaleTo` every frame) therefore made ~1.2 s of 20 ms frames.
    - Now all growing goblins are resized in the same frames, 30 times a second (`REVEAL_SCALE_HZ`, `revealScaleNow`), and only when the size changes by more than 1 %. The lift, spin and movement still update every frame. **The only visible difference: the growth itself steps at 30 Hz.**
    - Hidden (pop-in) goblins are built one per frame (`group.buildMore` → `syncGroup` again next frame). The reveals keep their 0.15 s rhythm (`group.nextReveal`).
    - The formation search (`permute`) uses branch and bound: the same result as all 720 orders, a fraction of the work. It ran once per built pet.
  - **Templates:** the Animation Editor's `InitialPoses` folders were moved out of `ReplicatedStorage.GoblinModels` / `EnemyModels`: 45 folders, 9,576 instances, 62 % of a goblin template. Cloning is 28 % faster and there is less to download. No script uses them.
- Where: `StarterPlayerScripts.GoblinPetClient`, `ReplicatedStorage.GoblinPetRenderer`. The InitialPoses are kept in `ServerStorage.UI_Backup.InitialPoses_backup`, one folder per model (`GoblinModels.<Id>.<parent>`).
- Test (336 stacks, 6 Huge+Shiny+Rainbow pets, Studio at 165 fps):

  | Situation | Before | After |
  |---|---|---|
  | Open animation | 35–50 ms frames | ~11 ms |
  | First open | 62 ms | 39 ms |
  | Inventory open, steady | 67 fps, render CPU 8.6 ms | 121 fps, 5.1 ms |
  | GUI objects | ~15,000 | 2,857 |
  | Equip Best (2.2 s window) | ~1.2 s of 18–24 ms frames, worst 48 ms | 124–127 fps on average, worst 27–28 ms |

  - Scrolling (jumps and a fast 2.5 lists/s scroll): never a visible card without content.
  - Search filtering and rainbow rims / odds / icon effects on re-attached cards were checked, and the Output had no errors.
- Still open:
  - The remaining spawn cost is one ~20 ms frame per resize tick during pop-ins, the first equip loading the personality modules (~45 ms once) and ~7 ms per pet for parenting on reveal.
  - If the 30 Hz growth is visible, `REVEAL_SCALE_HZ` can go up (each tick costs ~10 ms of render CPU).
  - Steady inventory cost: the panel is a CanvasGroup (~1 ms GPU). Swapping it for a plain Frame after the fade was not done, because it would re-parent the whole panel.

## 2026-10-07 — Performance audit overnight: inventory, pet spawning, running pets (Claude)
- What: performance-only changes; nothing should look different (the user: "VERDER MAG JE LETTERLIJK NIETS KWA VISUELE DINGEN VERANDEREN").
  - **Inventory cards (`StarterPlayerScripts.GoblinPetClient`):**
    - All card loops (shards, twinkles, rainbow rims / rim sparks / odds / bits, Shiny rims) only run while the Inventory panel is open, and only for cards that are on screen.
    - Visibility is computed by one read-only pass every 0.15 s at the start of the frame (BindToRenderStep "GoblinCardVisibility", priority First − 9).
    - Every loop entry remembers its card (`CARD.vis(e, obj)` returns "dead" / true / false), so hidden cards cost no property reads. Removed cards are cleaned up by the pass (`deadCells`).
    - Rainbow odds and Shiny rims are entry tables now (`{ g = gradient }`).
    - Cards are built off-tree and parented once they are complete.
    - `renderInventory` builds the new cards up to the visible part of the list (24 past the last visible one, or the first 56) right away. The rest are built after that within 4 ms per frame (`invUI.buildGen` cancels an older build).
  - **`ReplicatedStorage.GoblinIconFX`:**
    - `IconFX.Attach` no longer reads `AbsoluteSize`. That read forced a full GUI layout per card, so building 336 cards froze the game for ~0.4 s.
    - New icons are measured in the next frame's read pass (`fresh`). Visibility of all icons is batched in BindToRenderStep "GoblinIconFXVisibility" (First − 10, every 0.15 s; reads first, then writes). The render loop skips hidden icons before any property read.
    - Effects may start ≤ 1 frame later; off-screen / scrolled-out icons pause.
  - **`ReplicatedStorage.GoblinUI` `UI.aura`:** the layer size is cached (updated by its AbsoluteSize signal) instead of read every frame.
  - **`ReplicatedStorage.GoblinPetRenderer`:**
    - Newly equipped goblins of the local player (the pop-in ones) are built and dressed (FX, tag, mutation, Huge scale) outside the world. They are parented once, on reveal (`newPet(..., hidden)`); before, each was built in the world, pulled out and put back.
    - The model is moved with one `workspace:BulkMoveTo` per pet (`moveModel`, the same result as PivotTo; the part list is rebuilt on DescendantAdded/Removing; PivotTo is still used outside the world or without a PrimaryPart).
    - `FollowAnimation` is written only when it changes.
    - The pet hover runs its GUI hit test (`GetGuiObjectsAtPosition`) only when a goblin is under the mouse.
  - **`StarterPlayerScripts.GoblinCombatClient`:** the enemy hover runs `overInterface` (the GUI hit test) only when an enemy is under the mouse. The result is the same.
- Where: the Studio scripts above.
  - Backups in `ServerStorage.UI_Backup`: `GoblinIconFX_before_perf`, `GoblinPetClient_before_perf`, `GoblinPetRenderer_before_perf`, `GoblinCombatClient_before_perf`. There is no backup for the small `UI.aura` change.
  - The temp module `PerfProbe_TEMP` was removed.
- Test (measured):

  | Situation | Before | After |
  |---|---|---|
  | Inventory open, 336 stacks | 36 fps, render CPU 6.94 ms, scripts 487 ms/s | 95 fps, 5.88 ms, 260 ms/s (with the gating + batched visibility) |
  | Freeze on opening the inventory | 425 ms | 42 ms |
  | Equipping 6 Huge+Shiny+Rainbow pets (spawn script time in total) | ~141 ms (freezes 63/30/25/26/26 ms) | ~71 ms (one 46 ms freeze: the first load of the personality modules) |
  | 6 detailed pets running (Heartbeat harness, 165 Hz) | 189 ms/s | 179 ms/s (moving the models: 108 → 79 ms/s) |

  - The rest of the running cost is the engine moving ~300 parts per frame, plus raycasts (18 ms/s) and FX.After (35–44 ms/s); it can't go down without changing the look.
  - Idle (no pets): EnemyAnimator ~16 ms/s, ForestAmbience ~13.5 ms/s and the camera ~16 ms/s, all small; they were left as they were.
- New test hook (Studio only): set the `GoblinMenuGui` attribute `StudioOpenPanel` = "Inventory" / "Index" / "" to open or close a panel from a script.
- Still open:
  - The monitor went into standby during the night. Studio's Play viewport was then 1×1 and RenderStepped didn't run, so the card-loop and icon-loop changes made after that (vis-first entries, deferred building, IconFX `fresh`) were checked for errors by code review and a clean Output, **not watched on screen**.
  - Next session: open the inventory with 300+ pets and scroll through it (effects should look as before), equip / pop-in, run with 6 heavy pets, and re-measure render CPU / GPU with the monitor on.
  - GPU cost of the Highlights (Shiny / Rainbow outline) and lights was not measured.

## 2026-10-07 — Admin panel for a friend (Claude)
- What: UserId 2741325690 (Roblox name imjustafk2, the user's friend) was added to `ADMINS` in `GoblinInventoryServer`. They can type `/admin` in live servers after the next publish.
- Where: `ServerScriptService.GoblinInventoryServer`. Backup: `GoblinInventoryServer_before_admin_friend`.
- Still open: publish the game.

## 2026-10-07 — Shiny nametag twinkles; admin panel spawns every mutation combination (Claude)
- What:
  - **Shiny nametag:** 3 small white / pale-gold twinkles appear one after the other at random spots around the word "Shiny" (they grow, turn and fade, then jump).
    - They sit in a `ShinySparkZone` frame inside the name label, over the word. Its position comes from `textWidth` of the text before "Shiny" and of "Shiny" itself, relative to the full name.
    - They are animated in the existing per-tag step next to the Shiny glint.
  - **Admin panel:** every mutation and every combination is a button: Alle Huge / Shiny / Rainbow, Huge Rainbow, Huge Shiny, Shiny Rainbow, Huge Shiny Rainbow (built from `GoblinPets.MUTATIONS`, so new mutations add their combos automatically). The labels are TextScaled (max 14).
    - The server `AdminAction` accepts "mut:A+B+C" and cleans it with `CleanMutations`.
- Where: `GoblinTagFX` (backup `GoblinTagFX_before_shiny_sparks`), `GoblinInventoryServer` and `RollClient` (backups `*_before_admin_combos`).
- Test: Play — 6 ShinySparkZones (one per Shiny tag), twinkles visible on "Shiny King Goblin" (screenshot); the panel shows all 7 mutation buttons; "Huge Shiny Rainbow" gives every goblin with all three; no errors.
- Still open: nothing new.

## 2026-10-07 — Rainbow nametag built in (design 8 without the pill); rainbow odds on cards and the equipped bar (Claude)
- What:
  - **Nametag** (`GoblinTagFX`, `rainbowTag`, called from Attach for goblins with Rainbow):
    - "Rainbow" in moving rainbow colours (a copy of the name with only that word visible);
    - a rainbow plate (fill, border, glow, gems, chevrons: white images + UIGradient);
    - rainbow odds with a dark violet outline, and a rainbow halo / orbit rings / waves;
    - two twinkling rainbow stars beside the plate;
    - a white glint over the name every 2.6 s;
    - no "RAINBOW" pill.
    - Works together with the Huge / Shiny words.
  - **Tag loop:** one loop for all Rainbow tags (only visible tags). Colours update at 12 Hz (every colour change re-renders the BillboardGui: 217 gradients at 30 Hz cost ~0.6 ms with 6 tags); the glint and the stars move every frame. One shared ColorSequence per kind per tick.
  - **Inventory:** cards (`Odds`) and the equipped bar odds of Rainbow goblins use the same moving rainbow (sat 0.6, white text, dark violet outline): `CARD.rainbowOdds`, updated in the card loop (only while the inventory is open). The card odds are hooked with task.defer, because the label is created after the rim code.
- Where: `GoblinTagFX` (backup `GoblinTagFX_before_rainbow_tag`), `GoblinPetClient` (backup `GoblinPetClient_before_rainbow_odds`).
- Test: Play — 6 Rainbow tags created with RainbowWord, no errors; equipped bar odds in rainbow (screenshot).
  - Render CPU, 6 Rainbow vs 6 normal: 4.97 vs 4.05 ms in a throttled Studio (~42 fps, the user was playing). The earlier unthrottled run before the 12 Hz change was 4.33–4.47 vs 3.32–3.54 ms.
- Still open: a clean performance re-measure with Studio in focus; Rainbow in the roll animation; Glitched.

## 2026-10-07 — 8 more Rainbow nametag designs (9–16) (Claude)
- What: a second set, all animated rainbow (demo `Workspace.RainbowTags2_TEMP` + a 2D sheet `RainbowTagPreview2`, client only; sheet 1 hidden; screenshot `HUD_Mockup/rainbow_tags/rainbow_nametags_9-16.jpg`):
  9. white name with a flowing rainbow outline;
  10. the whole name in a turning diagonal rainbow (prism);
  11. only the plate border in rainbow;
  12. rainbow odds with a white shine sweeping through;
  13. pastel rainbow on name, odds and plate glow;
  14. a smooth 7-band rainbow arch over the odds whose colours run through it (7 round frames with strokes, top half clipped);
  15. rainbow sparkles orbiting the tag;
  16. neon: name + odds in bright rainbow with a coloured glow behind the name.
- Still open: the user picks one of 1–16.

## 2026-10-07 — Rainbow: only the skin, in sync, rainbow ground circle, lag fix; 8 Rainbow nametag designs (Claude)
- What:
  - **Only the skin:** new textures `HUD_Mockup/make_rainbow_skin_masks.py` → `HUD_Mockup/rainbow/skin/`, 42 uploaded into `GoblinMutationAssets.Rainbow.<Id>` with `AlphaMode = TintMask`.
    - The skin colours are read per goblin from its own .blend: the dominant colour of the faces on the Head bone plus its shades of the same hue. The eyes, teeth and mouth are left out.
    - The texture alpha is the mask: the skin takes the rainbow, everything else keeps its own colour.
    - Only the `*_Body` mesh gets the Rainbow SA; clothes, hats and accessories (separate meshes) keep their texture (Shiny's layer stays on them).
  - **All goblins in sync:** one shared clock (`st.rbT0 = 0`), and the body is one part, so every equipped Rainbow goblin has the same colour at the same time.
  - **Ground circle:** the goblin's aura ring and glow (`pet.fx.aura`) get a rainbow UIGradient that turns.
  - **Lag fix:**
    - per goblin there is now 1 coloured SA instead of ~16–56 per-part colour sets;
    - the inventory card loops (rainbow rims, rim sparkles, particles in `GoblinPetClient`) only run while the Inventory panel is open.
    - Measured, 6 Rainbow vs 6 normal: render CPU 3.57/3.70 vs 3.37/3.47 ms (~+0.2 ms), 164 fps; no errors.
  - **Clean-up:** `ReplicatedStorage.RainbowPreview_TEMP` deleted.
  - **8 nametag designs** (all animated rainbow; demo `Workspace.RainbowTags_TEMP` + a 2D sheet `RainbowTagPreview`, client only; screenshot `HUD_Mockup/rainbow_tags/rainbow_nametags_8.jpg`):
    1. "Rainbow" in moving rainbow colours;
    2. + a white glint over the name;
    3. the whole name in moving rainbow;
    4. rainbow plate;
    5. rainbow odds + halo / rings;
    6. a rainbow "RAINBOW" pill;
    7. rainbow glow round the plate + rainbow stars;
    8. deluxe (plate + odds + pill + stars + glint).
- Where: `GoblinMutationFX` (backup `GoblinMutationFX_before_rainbow_skin`), `GoblinMutationAssets.Rainbow`, `GoblinPetClient` (backup `GoblinPetClient_before_card_loop_gate`).
- Still open: the user picks a nametag; Rainbow in the roll animation; Glitched.

## 2026-10-07 — Rainbow mutation in the world for every goblin (3+ spiral + glint, D light, T2 steps, E equip) + lag check (Claude)
- What:
  - **Textures:** a pale holographic texture for each of the 42 goblins (`HUD_Mockup/make_rainbow_holo_all.py` → `HUD_Mockup/rainbow/holo/`). Uploaded and set up as `ReplicatedStorage.GoblinMutationAssets.Rainbow.<Id>` SurfaceAppearances (holo ColorMap + Shiny's metal / roughness maps).
  - **`ReplicatedStorage.GoblinMutationFX`, new Rainbow part (`attachRainbow`, `stepRainbow`, `prismPrint`, `rainbowEquip`):**
    - **Look ("3+"):** every mesh with the body texture gets the Rainbow SA (it replaces Shiny's layer when both; Shiny keeps its glitter / rim / light). Each frame its `SurfaceAppearance.Color` = a rainbow spiral `hsv(angle + height·0.6 − t·0.4)`.
    - **Glint:** every 2.6 s a white band runs top-left → feet in 0.9 s; parts lerp to white and white sparkles ride along (an attachment on the body moved along the band).
    - **Extras:** rainbow sparkles from the body; D = a colour-changing PointLight (Range 7, Brightness 1.2); a rainbow outline (reuses Shiny's rim when present, else its own, inside the shared Highlight budget of 20).
    - **T2 steps:** at every step (pet.phase halves, like Shiny's footsteps) a prism star print blooms at the foot: glow + star + ripple ring + rising sparkles. It uses a pool of 4 prints per goblin (SurfaceGui parts reused, no new parts per step).
    - **E equip:** in `M.EquipBurst` a white-rainbow star with a short rainbow trail falls from the sky (0.45 s) and lands with a big star print, a shockwave ring and a rainbow star burst.
    - **Huge / distance:** everything scales with `pet.size` (Huge), and colours / prints stop updating beyond 120 studs.
  - **Clean-up:** all Rainbow demo folders in the workspace (`*_TEMP`) were removed.
- Where: `GoblinMutationFX` (backup `ServerStorage.UI_Backup.GoblinMutationFX_before_rainbow`), `GoblinMutationAssets.Rainbow`.
- **Lag check** (Play, the user's 5 equipped Rainbow goblins, 192 coloured meshes):
  - colour updates cost 0.11 ms Lua per frame;
  - render CPU 3.50 ms with all Rainbow effects vs 3.73 ms with its particles / light / outline switched off → no measurable extra cost;
  - worst frame 17 ms (Studio in the background);
  - no errors.
- Still open:
  - `ReplicatedStorage.RainbowPreview_TEMP` (Edit) still has to be deleted (Play was running);
  - the "Rainbow" word colour in the nametag;
  - Rainbow in the roll animation;
  - Glitched.

## 2026-10-07 — Rainbow: 6 equip animations as a looping showcase (A–F) (Claude)
- What: 6 stations on the outer ring of the pool square (radius 22, round the 3+ side; `Workspace.RainbowEquip_TEMP`, loop `_G.equipConn` + one task per station, client demo), each with a letter on a post.
  - **Goblin at every station:** the full Rainbow look so far: 3+ spiral + glint + D light.
  - **Loop:** about every 4.5 s the goblin vanishes for 0.6 s and comes back with that station's equip animation.
  - **The animations:**
    - A Prism birth: a white light beam slams down, bursts into white sparks, 7 rainbow rays fan out and spin round the spot while the goblin grows out of it.
    - B Rainbow bridge: a rainbow arch grows from the sky, the goblin slides down it in an arc, and lands with a T2 star print, a ripple and a sparkle burst.
    - C Colour explosion: the goblin appears as a white silhouette, 7 coloured light rings run up its body (red → violet) while its glow takes each colour, then it bursts into colour with a star shower and a ground ring.
    - D Crystal bloom: 8 rainbow crystals sprout in a ring, close inwards like a bud, open again with a burst while the goblin grows inside, then sink away.
    - E Star-fall landing: a white-rainbow star with a short rainbow tail and star sparks falls from the sky, impacts with a star print, a shockwave ring and a big star burst, and the goblin jumps up out of the impact.
    - F Spectrum spin: the goblin grows while spinning twice; 4 coloured ribbons swirl round it like a rainbow whirlwind, ending in a sparkle burst.
- Test: render CPU 3.45 ms with all 6 loops (plus 3+ and the combo demo); no errors / warnings.
- Still open: the user picks an equip animation; then build the complete Rainbow (3+ + D + T2 + chosen equip) into `GoblinMutationFX` for every goblin.

## 2026-10-07 — Rainbow: combination 3+ + D (light) + T2 (prism steps) (Claude)
- What:
  - **Choice so far:** the user picked T2 as the trail and D (the colour-changing light) as an extra.
  - **Demo:** one walking demo "3+ D T2" (`Workspace.RainbowCombo_TEMP`, loop `_G.comboConn`, client demo) walks in a circle across the pool from the static 3+.
    - It has the spiral rainbow + glint (3+), a colour-changing light (Range 7, Brightness 1.2) and prism star prints at every step (T2).
  - **Clean-up:** the A–F and T1–T4 demos were removed.
- Still open: the user looks at it and may want more particles / animations; after that, build Rainbow into `GoblinMutationFX` for all goblins and clean up `*_TEMP` + `ReplicatedStorage.RainbowPreview_TEMP`.

## 2026-10-07 — Rainbow: 4 walking trail designs on top of 3+ (T1–T4) (Claude)
- What: the user asked for a rainbow trail that is not a basic trail ("wonderbaarlijk").
  - **Setup:** 4 copies of 3+ (the same spiral + glint) walk in small circles (radius 3.6, ~7 studs/s, a 1.5 s pause every 7.5 s) on the open sand across the pool from 3+ (`Workspace.RainbowTrails_TEMP`, loop `_G.trailConn`, client demo). The models slide; the bones are not animated in the demo.
  - **T1 Silk ribbon:** a short ribbon (Trail, 0.45 s) at hip height that waves; the rainbow colours run along it; twinkling stardust falls from it.
  - **T2 Prism steps:** every step a glowing rainbow star print blooms on the ground with a ripple ring and rising sparkles (fades in ~1 s).
  - **T3 Crystal bloom:** at every step 3 small neon rainbow crystals sprout from the ground, sparkle and sink away.
  - **T4 Aurora wake:** 3 silky weaving aurora ribbons behind it (each its own colour band) plus falling stardust.
- Note: the memory rule "no long trails" was kept: all trails are short (≤ 0.6 s).
- Test: render CPU 3.36 ms with 3+, A–F and T1–T4 running; no errors.
- Still open: the user's pick (trail + extras); then build Rainbow into `GoblinMutationFX` (steps from `pet.phase` like the Shiny footsteps) and remove all `*_TEMP` demos.

## 2026-10-07 — Rainbow: other demos removed, 6 extra effects on top of 3+ (A–F) (Claude)
- What:
  - **Clean-up:** the demos 1–6 and 7–12 were removed (`RainbowWave_TEMP`, `RainbowGlint_TEMP`); "3+" stays as the base.
  - **Cost of 3+:** render CPU 2.82 ms with it vs 2.84 ms without → free.
  - **A–F:** 6 copies of 3+ (exactly the same spiral + glint) in an arc on both sides of it (`Workspace.RainbowExtras_TEMP`, loop `_G.extraConn`, client demo). Each adds one extra effect:
    - A: a turning rainbow ring + soft rainbow glow under the feet;
    - B: three small rainbow orbs orbiting the goblin, with sparkle tails;
    - C: rainbow motes rising from the ground round it;
    - D: a small colour-changing rainbow light;
    - E: a rainbow arch (Beam) over the head;
    - F: now and then a big rainbow star flare flashes on the body.
- Test: render CPU with 3+ and A–F running: 2.96 ms; screenshot taken.
- Still open: the user picks which extras go with 3+; then build the Rainbow world look into `GoblinMutationFX` for all goblins and remove all `*_TEMP` demos and `ReplicatedStorage.RainbowPreview_TEMP`.

## 2026-10-07 — Rainbow body: variant 3 (spiral) + glint = "3+" (Claude)
- What: the user picked variant 3 (spiral rainbow) and asked for a glint, with the rest exactly the same.
  - A copy "3+" stands 6 studs in front of #3 (`Workspace.RainbowSpiralGlint_TEMP`, loop `_G.spiralGlintConn`). It has the same holo texture, the same per-part spiral colours `hsv(a + y·0.6 − t·0.4)`, the same rainbow sparkles and colour outline.
  - Plus a cheap glint every 2.6 s: in 0.9 s a white light band runs from the top-left corner (as you look at the goblin) down to the feet. Every part it passes lerps towards white (band width 0.16), and white sparkles ride along the band in front of the body (ZOffset 1).
  - No textures are animated; it only uses the same per-part `SurfaceAppearance.Color` sets plus one particle emitter.
- Test: screenshot mid-sweep shows the white diagonal glint over the spiral rainbow. (The render CPU of 5.24 ms in that test still included the expensive demos 7/8 running.)
- Still open: the user's verdict. Then build it into `GoblinMutationFX` for all goblins (holo texture per goblin, the spiral + glint loop, a performance check with 6 pets) and remove the temp demos and `ReplicatedStorage.RainbowPreview_TEMP`.

## 2026-10-07 — Rainbow body: 6 extra variants with a diagonal rainbow glint (7–12) (Claude)
- What: 6 more King copies next to variants 1–6 (continuing the arc round the spawn pool, `Workspace.RainbowGlint_TEMP`, loop `_G.glintConn`, client demo). Every one has the "rising rainbow" per-part wave (variant 1) plus a rainbow glint from the top-left corner down to the feet: one sweep of 0.9 s every 2.6 s.
  7. Sliding rainbow-band texture (`rainbow_band.png` 108370455990538) on the main body only (4 faces).
  8. The same texture on every part, aligned so the band runs through body + clothes + crown.
  9. A rainbow streak particle (`rainbow_streak.png` 72820287517041) flies diagonally over the goblin, leaving rainbow sparkles.
  10. A rainbow beam sweeps diagonally in front of the body.
  11. The parts flash white one after the other along the diagonal (a light wave through the clothes).
  12. A rainbow ForceField shell over the body that brightens with the sweep.
  - Images from `HUD_Mockup/make_rainbow_glint.py` (`HUD_Mockup/rainbow/`).
- **Cost** (render CPU, average over a full glint period with all 12 demos running):
  - 5.24 ms with the textures, 3.06 ms without.
  - Variant 7 alone: +2.1 ms for ONE goblin; variant 8 is about the same again.
  - → 7 and 8 (sliding textures, the only ones that look masked to the body) are too heavy for 6 equipped pets, as the earlier Shiny glint showed. Variants 9–12 cost ~nothing.
- Still open: the user picks one; the temp demos and `ReplicatedStorage.RainbowPreview_TEMP` get removed after the choice.

## 2026-10-07 — Rainbow body: 6 variants of design 1 (the per-part colour wave) (Claude)
- What: the earlier 5-design demo was replaced by 6 King copies in an arc on the far side of the spawn pool (`Workspace.RainbowWave_TEMP`, client demo, loop `_G.wave2Conn`). Every copy has the pale holo texture on all 93 MeshParts, and the `SurfaceAppearance.Color` of each part is set per frame from its normalised height y / side x / angle a. Each also has rainbow sparkles and a colour outline.
  1. **Rising rainbow:** the full rainbow from feet to head, flowing up.
  2. **Diagonal sweep:** diagonal bands, faster.
  3. **Spiral:** the rainbow turns round the body (barber pole).
  4. **Pastel dream:** soft pastel, slow.
  5. **Neon:** full saturation, a colour light and an outline in the opposite colour.
  6. **Stripes:** clear moving rainbow bands.
- Limitation: the goblin's main body (head + torso + limbs) is one MeshPart, so it shows one colour at a time and changes through the rainbow. The bands run over the separate parts (crown, clothes, accessories). A gradient over the body itself would need texture animation, which is ruled out for performance.
- Cost: render CPU 2.71 ms with all 6 running (the same as without).
- Still open: the user picks a variant; then it gets built into `GoblinMutationFX` for all goblins (holo/pearl textures per goblin, like Shiny) and the temp preview is cleaned up.

## 2026-10-07 — Rainbow body: 5 preview designs (Claude)
- What: 5 Rainbow looks on 5 King copies, standing in an arc on the far side of the spawn pool (centre (109, −5), radius 19), with a number above each. They are a demo in the client of the running Play session (`Workspace.RainbowDemo_TEMP`, loops `_G.rbDemoConn` / `_G.waveConn`). They are gone when Play stops.
  1. **Rainbow wave:** a pale holo texture on all 93 MeshParts. Each part's `SurfaceAppearance.Color` cycles through the hue with a phase by height, so a rainbow wave runs over the body. Plus rainbow sparkles and a faint colour outline.
  2. **RGB pulse:** the goblin's own texture; a Highlight fill cycles through every colour (gamer RGB) with a bright colour outline.
  3. **Holo foil:** a pearl-white texture with pastel rainbow shimmer (static), a soft pastel colour-cycling glow, white star glints and a tiny pastel light.
  4. **Aurora:** the goblin's own texture, a strong colour-changing light, a turning rainbow ring under the feet, rainbow motes rising, and a colour outline.
  5. **Spectrum:** the goblin's own colours hue-shifted into rainbow colours (static), a rainbow arch (Beam) over the head and a soft white shine pulse.
- **Assets:** textures from `HUD_Mockup/make_rainbow_textures.py` (only for King; prisma 126392688789187, holo 124498922424534, shift 101405498094442), as SurfaceAppearances in `ReplicatedStorage.RainbowPreview_TEMP` (Edit, temporary). The goblin texture is an atlas of flat colour cells, so a baked rainbow becomes a palette swap per face. A real flowing rainbow needs the per-part `SurfaceAppearance.Color` wave (design 1).
- **Cost:** the design 1 wave (93 colour sets per frame): render CPU 2.67 ms with it vs 2.71 ms without → free.
- Still open: the user picks one (or a mix). Then build it into `GoblinMutationFX` for all goblins and delete `ReplicatedStorage.RainbowPreview_TEMP`.

## 2026-10-07 — Rainbow mutation: inventory cards (rainbow rim, particles, shards) (Claude)
- What:
  - **Mutation:** `GoblinPets.MUTATIONS` has `{ Name = "Rainbow", Chance = 12800 }` (after Shiny, so names read "Huge Shiny Rainbow King Goblin"). It can be rolled, saved and granted ("Alle Rainbow" in the admin panel), and the odds multiply (e.g. Huge Shiny Rainbow King = 1/65.54B). **The world look (RGB over the body) and the purple/coloured name word are not built yet.**
  - **Shards:** Rainbow alone uses 93389232075141 (the user's purple shard recoloured with the rainbow star's colours, `HUD_Mockup/make_rainbow_shard.py`). The combos use the user's icons (`CARD.SHARDS`: Rainbow+Shiny, Huge+Rainbow, Huge+Rainbow+Shiny).
  - **Card rim (`CARD.rainbowBand`):**
    - the same thick rim as every card, filled with a full rainbow gradient (`CARD.RAINBOW`) that turns round the card (80°/s) and drifts (Offset wobble), so the colours flow along the rim;
    - a thin white gloss line on the outer side and a dark violet outer line;
    - Rainbow + Shiny (and + Huge): 3 white sparkles run round the rim, twinkling and spinning (`CARD.rimSparks`), plus the Shiny white twinkles over the goblin.
  - **Particles (`CARD.rainbowParticles`):** 5 small star / sparkle bits per Rainbow card drift up inside the card behind and over the goblin, each cycling through the rainbow colours.
  - **Icon size:** with Huge the goblin is drawn big (as all Huge cards), otherwise normal size.
  - **Rim priority:** one RenderStepped loop drives rims, rim sparkles and particles. Rainbow's rim takes priority over the Huge / Shiny rims.
- Where: `GoblinPets` (MUTATIONS), `GoblinPetClient` (CARD.SHARDS, CARD.rainbowBand, CARD.rainbowParticles, makeCell). Backups in `ServerStorage.UI_Backup`: `GoblinPets_before_rainbow`, `GoblinPetClient_before_rainbow_cards`.
- Test: Play with King/Mystic in every Rainbow combination + Huge+Shiny + normal. The zoomed screenshot shows the right shards, a rainbow rim, Huge = big icon, and rim sparkles on the Shiny ones. Inventory open with 7 Rainbow cards: render CPU 3.16 ms, 165 fps; no errors.
- Still open:
  - Rainbow in the world (RGB effect over the whole body), the nametag word and the roll animation;
  - the equipped bar has no rims;
  - Glitched.

## 2026-10-07 — King icon: calmer crown twinkles (Claude)
- What: the King icon's crown twinkles are calmer: 0.7 per second instead of 1.6, size 0.055 instead of 0.07. The jewel twinkle comes every 4.5–7 s instead of 2.5–4 s, at size 0.1 instead of 0.14. This is used on every King icon (inventory, equipped bar, roll).
- Where: `ReplicatedStorage.GoblinIconFX` (DEFS.King). Backup: `ServerStorage.UI_Backup.GoblinIconFX_before_king_calm`.
- Test: the source compiles.
- Still open: nothing.

## 2026-10-07 — Shard icons for every mutation combination (Claude)
- What:
  - **Combination shards:** the user's sheet with combination shards (`HUD_Mockup/shards/ref_shard_combos.png`) was cut with `HUD_Mockup/cut_combo_shards.py` into 128 px icons and uploaded:
    - Rainbow + Shiny + Huge: 99339973620159;
    - Rainbow + Shiny: 83070047414459;
    - Rainbow + Huge: 87181169321532;
    - Huge + Shiny (new design, now in use instead of 101114067243003): 85703211884213.
  - **Lookup:** `GoblinPetClient` has `CARD.SHARDS` (key = the mutation names sorted A–Z) and `CARD.shardFor(mutations)`. `CARD.gem` uses it, and any combination shard is drawn 1.18×. The Rainbow ones are ready for when Rainbow exists.
  - **Memory:** the claude memory `mutation-shard-icons` lists every shard for future goblin icons.
- Where: `GoblinPetClient` (CARD.SHARDS, CARD.shardFor, CARD.gem). Backup: `GoblinPetClient_before_combo_shards`.
- Test: Play — King|Huge,Shiny shows 85703211884213, King|Huge shows the purple shard, King|Shiny shows the white one, King shows the yellow one; no errors.
- Still open: shards for Rainbow alone and for Glitched (+ combinations).

## 2026-10-07 — Huge: effects and animations scale along; bigger Huge inventory icons (Claude)
- What:
  - **World effects for Huge** (`GoblinPetFX`):
    - All particle emitters, lights and beams on the goblin are scaled ×size (incl. Shiny's). This uses `FX.scaleEmitter`: Size, Speed and Acceleration. It applies to those that exist at Rescale and to every one added later (model.DescendantAdded, attribute `FXScaled`).
    - World puffs from personality ticks and gestures use `FX.curScale` (set around def.tick / gesture.run).
    - `Fx:wp` scales the model-space points the personalities pass; `Fx:ball`, `Fx:arc` and `Fx:footprint` scale too.
    - Rig motions (`r.T` translations: a tossed cleaver, a lifted flask, a torch grip …) scale too.
    - Zone parts and the aura ring grow ×size.
  - **Animations** (`GoblinPetRenderer`): the bone Transform translations (clip bobs, gesture moves `P:move`) and the gesture lift (`fx.lift`) are ×size, so a Huge goblin moves like a big goblin instead of with normal-sized motions.
  - **Inventory icon:** a Huge card draws the goblin bigger, filling the card like the user's reference (holder 1.5 instead of 1.12, centre (0.45, 0.68)). The head sits near the top and the body is cut off by the card and the hill.
- Where: `GoblinPetFX`, `GoblinPetRenderer`, `GoblinPetClient` (makeCell). Backups in `ServerStorage.UI_Backup`: `*_before_huge_fx`, `GoblinPetClient_before_huge_icon`.
- Test:
  - Play: the Huge cards show the big goblin (zoomed screenshot); no errors.
  - The world gesture test with Huge Phoenix / Pyro / Engineer was cut short because the user was playing in the same session. The effect scaling is not checked visually yet.
- Still open:
  - the user checks the Huge effects in the world;
  - effects whose code sets new absolute sizes later (e.g. `e.Size = …` inside a tick) stay normal size;
  - the equipped bar icon for Huge is not enlarged.

## 2026-10-07 — Huge 1.6x; Huge / Huge + Shiny shards and card rims (Claude)
- What:
  - **Huge size:** 1.8 → 1.6 (`GoblinPets.MUTATIONS` Huge `Size = 1.6`). Everything follows `SizeOf`; the inventory icons are not scaled (the icon art stays the same).
  - **Shards (top right, same spot as the others):**
    - Huge: the user's purple shard (81695202540613).
    - Huge + Shiny: the user's white + purple pair (101114067243003), drawn 1.18× so each star is about single-star size.
    - Both were cut from the dark background with `HUD_Mockup/cut_dark_shards.py` (`HUD_Mockup/shards/`).
    - `CARD.gem` picks yellow / white / purple / pair from the mutations.
  - **Card rims (same thickness as all cards, `CARD.thickRim`):**
    - Huge only: a solid violet rim (light violet top → deep violet bottom), no light spots, no animation, and a dark violet outer line.
    - Huge + Shiny: a light lilac rim with two white light spots gliding round, the same loop as Shiny (`CARD.shinyRims`).
    - `CARD.hugeBand(cell, edge, equipped, shiny)`. Shiny goblins (also Huge + Shiny) keep the white twinkles over the goblin.
  - **Studio grant hook:** "King:Huge+Shiny" now gives several mutations ("+" → ",").
- Where: `GoblinPets`, `GoblinPetClient` (CARD.gem, CARD.hugeBand, makeCell), `GoblinInventoryServer` (grant hook). Backup: `GoblinPetClient_before_huge_cards`.
- Test: Play with King/Mystic as Huge+Shiny, Huge, Shiny and normal. Zoomed screenshot: each kind has its own shard; Huge = deep violet rim, Huge+Shiny = lilac rim with white spots, Shiny = silver; odds totals (1/5.12M etc.). No errors.
- Still open:
  - the Huge nametag choice (1–8);
  - Huge in the roll animation;
  - the equipped bar has no rims (only shards);
  - Rainbow and Glitched.

## 2026-10-07 — Huge mutation (1/800, 1.8x size) + 8 Huge nametag designs to choose from (Claude)
- What:
  - **Mutation:** `GoblinPets.MUTATIONS` now starts with `{ Name = "Huge", Chance = 800, Size = 1.8 }` (before Shiny, so names read "Huge Shiny …"). The chance is fixed and multiplies like Shiny. New `GoblinPets.SizeOf(mutations)` gives the size factor. Rolling, saving, stacking (`g|mu`), the tooltips, the total odds, Equip Best and the admin panel ("Alle Huge" appears automatically) all work through the generic mutation code.
  - **World model** (`GoblinPetRenderer.newPet`):
    - The model is scaled ×1.8 with `ScaleTo` after the effects are attached, and `footLift` grows with it, so the feet stay on the ground.
    - New `GoblinPetFX.Rescale(fx, k)` scales every stored model-space offset (bone rests, rigs, attachments, zones, cape segments and hinges, wing feathers and hinges), so capes, wings, hats and effects stay attached. The nametag hangs higher (`TagFX.Rescale`).
    - Step hop ×size, hover/click box ×size, and a Huge goblin keeps more room from its player (+1.2 studs).
    - With a Huge goblin in the group, the whole formation spreads out ×1.35.
  - **Nametag:** "Huge" in purple (#C58BFF, `TagFX.HUGE_WORD`) before the name. It combines with Shiny's pale gold, and the odds are the total (e.g. Huge King 1/25,600).
  - **8 nametag ideas** (screenshots on non-Epic goblins, because Epic plates are purple already): `HUD_Mockup/huge_tags/huge_nametags_8.png`.
    1. only the purple word;
    2. plus a white glint over "Huge";
    3. a purple plate;
    4. a purple "HUGE" pill above the odds;
    5. purple odds with a purple halo and orbit rings;
    6. two twinkling purple stars beside the plate;
    7. "Huge" with a moving purple → pink gradient;
    8. deluxe: purple plate + odds + pill + glint + stars.
    The demo (`Workspace.HugeTags_TEMP`, client only) was removed after the screenshots.
- Where: `GoblinPets`, `GoblinPetRenderer`, `GoblinPetFX` (Rescale), `GoblinTagFX` (HUGE_WORD, Rescale, the coloured-name block). Backups in `ServerStorage.UI_Backup`: `*_before_huge`.
- Test:
  - Play: King/Mystic/Leaf Huge at scale ×1.8 (bodies 6.6 × 7.8 vs 3.7 × 4.4), on the ground, tags higher, name "Huge King Goblin" in purple. No renderer / FX errors.
  - Performance, 3 normal vs 3 Huge (same designs, twice each): render CPU 3.28/3.76 vs 3.43/3.30 ms, GPU 2.97/2.68 vs 2.73/3.12 ms → no difference. Studio was throttled to ~40 fps in the background.
  - The GoblinTagFX "Parent property locked" warnings came from the destroyed demo tags (separate module instance), not from the game.
- Still open:
  - the user picks a nametag design (then build it into GoblinTagFX);
  - Huge in the roll animation (the name and odds in the roll still ignore the mutation) and on the inventory icons;
  - Rainbow (1/12,800) and Glitched (1/3,200).

## 2026-10-07 — Shiny equip burst without sound (Claude)
- What: when a Shiny goblin spawns (equip burst), no sound plays any more. The flash, sparkle ring and star rain stay. The preloaded sound templates are still created but unused.
- Where: `ReplicatedStorage.GoblinMutationFX` (EquipBurst). Backup: `ServerStorage.UI_Backup.GoblinMutationFX_before_mute_equip`.
- Test: the source compiles; the sound loop is removed.
- Still open: nothing.

## 2026-10-07 — Normal goblin cards get exactly the Shiny rim thickness (Claude)
- What:
  - **Rim:** the thick rim geometry now lives in `CARD.thickRim(cell, colorSeq, rotation)`: frame `Rim`, inset 7 × grid scale, a UIStroke of thickness inset − 1.5, round inner corners.
    - Shiny cards use it with the rotating silver gradient (`CARD.shinyBand`).
    - Normal cards use it with their own blue band colours (`CARD.band`, rotation 90, static).
  - **Outer line:** 2.2 on both (green 2.6 stays for equipped).
- Where: `GoblinPetClient` (CARD.thickRim, CARD.shinyBand, makeCell). Backup: `GoblinPetClient_before_thick_rims`.
- Test: Play — zoomed screenshot: Shiny and normal rims are equally thick; no errors.
- Still open: nothing.

## 2026-10-07 — Shiny rim no longer cuts the hill off (Claude)
- What: moving the card's inner part in (for the thick Shiny rim) left the hill at the bottom ending in an ugly line.
  - **Inner part:** it keeps its normal size again.
  - **Rim:** the thick rim is now a `ShinyRim` frame over the card (inset 7, ZIndex 6) whose UIStroke (thickness inset − 1.5, × grid scale) covers the band and the outer 2 px of the inner part, with rounded inner corners. The hill disappears neatly under the rim.
  - **Animation:** the rotating light gradient now sits on that stroke (`CARD.shinyRims`); the band underneath is a static light silver.
- Where: `GoblinPetClient` (CARD.shinyBand). Backup: `GoblinPetClient_before_shiny_rim3`.
- Test: Play — zoomed screenshot shows the hill tucked under the rim on Shiny cards; no errors.
- Still open: nothing.

## 2026-10-07 — Shiny card rim like the user's reference: thicker, brighter, light spots gliding round (Claude)
- What: `CARD.shinyBand` was rewritten.
  - **Thicker rim:** the inner part moves in (inset 8 instead of 5, × grid scale), so the silver rim is ~6.5 px.
  - **Brighter:** a bluish-silver gradient (`CARD.SHINY_RIM`) with two bright white spots, a dark navy outer edge (2.2) and a soft blue inner line.
  - **Animation:** one RenderStepped loop turns every Shiny rim gradient (70°/s, about 5 s per turn), so the light spots glide smoothly round the border in one direction. This replaces the old diagonal sweep tween.
- Where: `StarterPlayerScripts.GoblinPetClient` (CARD.shinyBand, CARD.shinyRims). Backup: `GoblinPetClient_before_shiny_rim2`.
- Test: Play with 3 Shiny cards: thicker silver rim (zoomed screenshot), gradient rotation 234° → 269° in 0.5 s, render CPU 2.89 ms, 166 fps, no errors.
- Still open: nothing.

## 2026-10-07 — Inventory Sort button; Shiny cards get a silver rim with a moving light (Claude)
- What:
  - **Sort:** the old "owned / capacity" pill at the bottom left of the inventory is now a Sort button ("Sort: Rarity", three-bar icon). Clicking it opens a small menu upwards with three options (the user dropped Name, Amount and Oldest):
    - **Rarity** (default): rarity, then mutated first, then id.
    - **Best Goblin:** total odds incl. mutations, the same formula as Equip Best.
    - **Newest:** the newest copy in the stack, using item `t`.
    - Ties fall back to Rarity. The menu closes when the inventory closes, and reused cards just get a new LayoutOrder (`invUI.sortMode`). `invUI.setCounts` is now a no-op.
  - **Shiny cards:** the card rim (the Band gradient) turns silver-white, and a white light slides over it diagonally in one direction (looping Offset tween: 1.3 s sweep + 1.1 s pause). The outer edge is grey (green stays when equipped). `CARD.shinyBand`; the tween is cancelled when the card is destroyed.
- Where: `StarterPlayerScripts.GoblinPetClient` (bottom bar, renderInventory sort, makeCell). Backup: `GoblinPetClient_before_sort`.
- Test: Play — the Sort menu opens with Rarity / Best Goblin / Newest (screenshot); Shiny cards show the silver rim; no errors. Picking an option was not verified live (the test click missed), but the sort code compiles.
- Still open: nothing.

## 2026-10-07 — Sparkles only on Shiny icons (Claude)
- What:
  - **Shard sparkle:** the sparkle flash on the shard is gone for normal goblins; Shiny goblins keep it.
  - **Shiny twinkles:** Shiny goblins also get small white sparkles that twinkle at random spots over the goblin: 3 on inventory cards, 2 in the equipped bar. `CARD.shinySparkles(parent, z, count)`, with one RenderStepped loop over `CARD.twinkles`.
  - **Unchanged:** the goblins' own themed IconFX effects (e.g. Mystic's stars).
- Where: `StarterPlayerScripts.GoblinPetClient` (CARD.gem, CARD.shinySparkles, makeCell, makeEquippedIcon). Backup: `GoblinPetClient_before_icon_sparkles`.
- Test: Play — King|Shiny and Mystic|Shiny had 3 twinkles + shard spark; King, Mystic and Basic had none; no errors.
- Still open: whether the themed IconFX sparkles (Mystic etc.) should go too.

## 2026-10-07 — Shiny card stripes removed (Claude)
- What: on the user's request the light-stripes background on Shiny inventory cards (and its lilac tint) was removed. Shiny cards look like normal cards again, apart from the white shard.
- Where: `GoblinPetClient` (makeCell). `CARD.shinyStripes` removed; the asset 76204659159687 is unused.
- Test: the source compiles; no "ShinyStripes" left.
- Still open: the Shiny inventory icon design.

## 2026-10-07 — Shards on goblin cards; Shiny cards get light stripes (Claude)
- What:
  - **Shards:** the gem at the top right of every goblin card (inventory cells and the equipped bar) is now the user's shard. Yellow (76564998425227) for normal goblins, white (127587484117839) for Shiny ones.
    - `CARD.gem(parent, _, pos, size, z, mutations)` picks the image.
    - Sizes are a bit larger for the star shape: 28 on cards, 27 in the bar. It still sways and sparkles.
  - **Shiny cards:** a soft lilac card (UIGradient 226,214,250 → 196,184,240) with three diagonal white light stripes rising to the right behind the goblin (`ShinyStripes`, 76204659159687), like the user's reference card.
  - **Assets:** `HUD_Mockup/make_shards.py`, outputs in `HUD_Mockup/shards/`. The shard files already had transparency; it is kept and cropped to 128 px. Stripes: 256 px, alpha 0.55–0.6.
- Where: `StarterPlayerScripts.GoblinPetClient` (CARD.gem, makeCell, makeEquippedIcon). Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_shards`.
- Test: Play with Shiny and normal goblins granted. Shiny cards show the white shard and the stripes; normal cards show the yellow shard. No errors.
- Still open: nothing.

## 2026-10-06 — Admin panel: spawn buttons (Claude)
- What: a "Spawnen" section in the roll test / admin panel (`/admin`) with these buttons:
  - "Alle pets": one of every goblin.
  - One button per mutation in `GoblinPets.MUTATIONS` (now only "Alle Shiny"; new mutations get their button automatically).
  - "+1M $": `Inventory.AddCoins` +1,000,000.
  - "Coin Rain": a 15 s rain right away.
  - The buttons fire the new remote `AdminAction` ("allPets", "mut:<Name>", "money", "coinRain"). The server checks Studio or `ADMINS`, and checks that a mutation name exists. The panel height follows the number of buttons.
- Where: `GoblinInventoryServer` (AdminAction), `RollClient` (panel). Backups in `ServerStorage.UI_Backup`: `*_before_admin_actions`.
- Test: Play — allPets gave 42 goblins, Alle Shiny another 42 (84), money 0 → 1,000,000; the buttons show in the panel; no errors.
- Still open: nothing.

## 2026-10-06 — Coin Rain starts when the last coin enters the bar; bigger "COIN RAIN!" splash (Claude)
- What:
  - **Start moment:** the 3rd coin no longer erupts at the reveal; it only charges up (glow, shake, flash). On exit it flies into the 3rd segment like the others, and when it lands the rain pours out of the bar (`Coin.exit` → `startRain(segment)`).
    - `pendingRainCoin` stops the afterRoll fallback from starting the rain early.
    - Server: `CLOVER_COIN.StartDelay` 2.2 → 3.8 s (hold + flight at roll speed x1), so the 15 s count from the moment the rain starts.
  - **Splash:** "COIN RAIN! / 30x LUCK" slams in at 1.9x, settles at 1.5x (was 1x), throbs 3 times, has bigger gold and clover bursts, and stays 2.0 s before flying into the bar title.
  - **Fix:** a new rain started while the previous one was still fading out showed no rain and no splash. `CoinRainFX` handle `restart(origin)` now brings spawning, glow and rays back, and `startRain` uses it plus the splash and sound.
- Where: `RollClient`, `CoinRainFX`, `GoblinPets`. Backup: `ServerStorage.UI_Backup.RollClient_before_rain_from_bar`.
- Test: Play, 3 coins: segment 3 filled at 4.09 s, rain started at 3.98 s after the click, server end matched (15 s). Big splash screenshot. No errors.
- Still open: nothing.

## 2026-10-06 — /admin chat command: the roll test panel in live games (Claude)
- What:
  - **Command:** admins type `/admin` (or `!admin`) in chat to show or hide the roll test panel (speed, columns, "Volgende roll: Clover Coin"), also in live servers.
    - `GoblinInventoryServer` has an `ADMINS` list (UserId 591297703 = the user), a TextChatCommand `AdminPanelCommand` (plus a Player.Chatted fallback for legacy chat), and toggles the player attribute `AdminPanel`.
    - In Studio the panel shows by default, as before.
  - **Remotes:** in live servers `StudioRollUpgrades` and `StudioForceCoin` now also exist, but only work for ADMINS.
  - **Client:** RollClient always builds the panel; it is visible when `AdminPanel` is true (in Studio: unless it is false).
  - **RollServer:** the forced coin now reads the attribute in live games too. Only the admin remote can set it.
- Where: `GoblinInventoryServer`, `RollServer`, `RollClient`. Backups in `ServerStorage.UI_Backup`: `*_before_admin_cmd`.
- Test: Play — the command is registered (TextChatService), the UserId matches, and the panel hides with `AdminPanel = false`. No errors.
- Still open: add more admins to `ADMINS` when needed. The panel needs a publish to work live.

## 2026-10-06 — Luck clover above ROLL animates again (Claude)
- What: the user missed the clover's small animation after the layout change.
  - The clover icon now sits in its own `CloverBox` frame inside the UIListLayout, so the layout never touches it.
  - It keeps its sway (±8°, 1.6 s) and also gets a small breathing scale (1.0 ↔ 1.1, 1.1 s).
  - Nothing else changed. A short try that let the clover show 30x during the Coin Rain was reverted on the user's request (backup `RollClient_before_rain_clover`).
- Where: `StarterGui.RollGui.RollClient` (Coin Rain section, clover layout).
- Test: Play — rotation −7.3° … +6.7°, scale 1.00–1.10; no errors.
- Still open: nothing.

## 2026-10-06 — Clover Coin a bit lower and smaller in the roll (Claude)
- What: the coin overlay overlapped the "1/300" odds. It now sits 8 % of the icon size lower (centre at icon centre + 3 % instead of −5 %) and is a touch smaller (0.66 of the icon box instead of 0.7, including the toss, squash and bonus spin sizes). It stays clear of the odds and still sits above "Clover Coin".
- Where: `StarterGui.RollGui.RollClient` (coinOverlay, Coin.reveal, bonusReveal).
- Test: Play with a forced coin — the coin sits free of "1/300", above the name.
- Still open: nothing.

## 2026-10-06 — No walk/jump sound, Coin Rain layout and fixes, inventory no longer rebuilds on every roll (Claude)
- What:
  - **Walk and jump sounds off:** new LocalScript `StarterPlayerScripts.MuteCharacterSounds` keeps the default "Running" and "Jumping" sounds (made by Roblox's RbxCharacterSounds on every character) at volume 0. The other character sounds stay.
  - **Coin next to its name:** the Clover Coin overlay used to take the icon's size while the landing punch had it at 1.25x, so it sat too big and too high. It now follows the slot icon every frame (until it flies to the bar), sitting just above "Clover Coin".
  - **Round coins:** the user's coin screenshots were cut off at the bottom and right. `HUD_Mockup/cut_coins.py` now fits a circle to the real outline and rebuilds the missing rim from the nearest intact angle.
    - New assets: Clover Coin 124799728984513, coin-pile coin 104831425111147 → 111711170619552.
    - The medallion and the rain use them.
  - **Layout (the user's final pick):**
    - The luck clover (`1x`) is centred above ROLL.
    - The COIN RAIN bar sits right of ROLL, in the middle of its height (`group` at x +150).
    - In the full-screen roll it slides past the Hide Rolls button (x +240).
  - **BONUS resets the time:** a BONUS coin also sets `CoinRainEnd` again (StartDelay/RollSpeed + 15 s).
    - The client bar keeps counting the old time (`holdEnd`) until the bonus coin erupts on screen, then refills with a shine and the countdown pops.
    - The rain keeps going on the server time.
  - **Inventory during rolls:** every new roll used to rebuild every card (`renderInventory` destroyed all cells), which cut off the hover animation and tooltip and caused a frame spike per roll. Now:
    - each card stores a signature (ids, equipped, free, grid view, delete mode) and is reused when unchanged;
    - only changed stacks get a new card;
    - the tooltip only closes when its own card is replaced;
    - delete mode still rebuilds.
- Where: `MuteCharacterSounds`, `RollClient`, `RollServer`, `GoblinPetClient` (`renderInventory`).
  - Backups in `ServerStorage.UI_Backup`: `*_before_coin_fixes`, `GoblinPetClient_before_cell_reuse`.
- Test:
  - Play, character sounds: Running = 0, Jumping = 0, the rest unchanged.
  - Coin reveal: the coin sits right above its name; layout screenshots taken.
  - Inventory open with 42 goblin stacks plus 3 rolls: only 3 new cards (one per roll), worst frame 8 ms.
  - Render CPU with the inventory open is 4.4 ms vs 2.7 ms closed. That comes from the ~18 visible cards (off-screen cards are already culled); fine at 143 fps.
  - No errors.
- Still open: nothing.

## 2026-10-06 — Coin Rain: BONUS coin (100x), 15 s, 1/300, Studio "next roll: Clover Coin" button (Claude)
- What:
  - **Numbers:** `GoblinPets.CLOVER_COIN` now has `Chance = 300`, `RainTime = 15` and a new `BonusLuck = 100`.
  - **BONUS coin:** a coin rolled during an active rain (same fixed chance) is a BONUS.
    - It does not count for the bar.
    - The server sets the player attribute `CoinRainLuck = 100`, and every roll for the rest of the rain uses it (a new rain resets it to 30).
    - The result is `{ coin = true, bonus = true }`.
  - **BONUS effect** (`RollClient`, `bonusReveal` / `bonusSplash` / `badgeCountUp`):
    - After the toss the coin spins up green-gold, flashes, sends out three green/gold shockwaves and a storm of clovers and sparkles.
    - "BONUS!" (Montserrat Heavy, green → gold, glint) and "100x LUCK" burst out of the coin, throb, then fly into the badge, which counts 30x → 100x.
    - The rain gets `CoinRainFX` handle `boost(origin, color)`: a new 90-coin fountain plus a stream out of the coin, 1.8x more rain, and the edge glow and god rays turn green-gold. The bar glow turns green.
    - The badge, the splash and "x LUCK" read `CoinRainLuck`.
  - **Studio test panel** (`RollUpgradeTest`, right of the screen): a new "Clover Coin" row with a "Volgende roll: Clover Coin" button.
    - It fires the Studio-only remote `StudioForceCoin`, which toggles the player attribute `StudioForceCoin`. The button turns gold ("Coin klaar: druk op ROLL").
    - `RollServer` turns the first result of the next roll into a coin and clears the flag. During a rain that coin is a BONUS.
- Where: `GoblinPets`, `RollServer`, `GoblinInventoryServer`, `CoinRainFX`, `RollClient`.
  - Backups in `ServerStorage.UI_Backup`: `*_before_coin_bonus`.
- Test: Play — the forced coin via the panel button worked, the rain ran 15 s, and the BONUS showed (green-gold rain, "BONUS! 100x LUCK"; screenshot). No errors.
- Still open: nothing new (see the luck note).

## 2026-10-06 — Clover Coin and Coin Rain (Claude)
- What (the user's spec):
  - **Rolling a coin:** every rolled result (per column) has a fixed 1/400 chance to be a **Clover Coin** instead of a goblin; luck does not change it. The coin never shows in the cycling, only as a result.
  - **Rain:** 3 coins start a **Coin Rain**: for 10 s every roll gets 30x luck (`Luck × 30` in `pickRarity`).
  - **Server:**
    - `GoblinPets.CLOVER_COIN = { Name, Chance 400, Needed 3, RainTime 10, RainLuck 30, StartDelay 2.2 }`.
    - `GoblinInventory`: saved `data.rainCoins` (0–2), player attribute `RainCoins`, `Inventory.AddRainCoin(player) -> count, started`.
    - `RollServer`: a coin result is `{ coin = true, rarity = "Coin", rainStart = true|nil }`.
    - When the rain starts, the player attribute `CoinRainEnd` = server time + StartDelay/RollSpeed + 10. StartDelay is the time until the coin lands and erupts on the client.
  - **Client — the roll** (`RollClient`, "Clover Coin + Coin Rain" section, plus `Coin` hooks in setFace, playRoll and tryRoll):
    - **Coin reveal:** the result is the user's Clover Coin at goblin size, in an overlay outside the slot's CanvasGroup. It is tossed up while it flips (its back is the user's coin-pile coin). It lands with a squash, ting + coin sound, a gold shockwave ring, sparkles and clover leaves. The name is "Clover Coin" in gold, the odds read 1/400, and a star twinkles on the coin.
    - **Into the bar:** on exit the coin flies into the next bar segment. The segment fills left to right with a shine sweep and sparkles, the count pops and the medallion bounces.
    - **Starting the rain:** the 3rd coin charges up (glow swell, shake), flashes with a double shockwave, and the rain pours out of it.
  - **Client — the bar** (`RollHud.CoinRain`): a gold tube with 3 segments and an "n/3" count, the Clover Coin as the medallion on the right (rocking gently), and a "COIN RAIN" title in Montserrat Heavy gold with a white glint every 3.2 s. The luck clover sits to the left; the bar and clover are centred over ROLL and hide with the clover when a menu panel opens.
  - **Client — the rain** (`ReplicatedStorage.CoinRainFX`, new ModuleScript):
    - The splash "COIN RAIN! / 30x LUCK" pops up with a glint and flies into the bar title.
    - The bar fills, glows and gets a green "30x" badge (pulsing) on the medallion, then drains as a countdown ("9s").
    - The rain is a pooled UI particle system in `RollGui.CoinRainLayer` (ZIndex 0, behind the roll and the HUD, over the roll tint), driven by one RenderStepped loop:
      - small coins of both designs, each showing the other design while it flips;
      - a fountain out of the coin for 2.6 s, then rain over the whole screen (55/s, depth = size, speed and paleness);
      - near coins bounce once on the bottom edge;
      - twinkling stars, drifting clover leaves, slow gold god rays and a gold edge glow;
      - the "Free Fall Coins" loop sound.
  - **Assets:**
    - The user's coins were cut out with `HUD_Mockup/cut_coins.py` and upscaled to 256 px: Clover Coin 75231984220498, coin-pile coin 104831425111147.
    - The references and outputs are in `HUD_Mockup/coin_rain/`.
    - Sounds: Coin Pickup 113730061669739, Free Fall Coins 1836041075, and the glockenspiel/crystal tings.
- Where: `GoblinPets`, `GoblinInventory`, `RollServer`, `GoblinInventoryServer` (Studio hook), `RollClient`, `CoinRainFX`.
  - Backups in `ServerStorage.UI_Backup`: `*_before_coin_rain`.
- Test:
  - **Studio hooks:** ServerStorage `StudioCoinChance = 1` makes every result a coin; `StudioCoinRain = true` starts a rain right away.
  - **Played in Studio:** coin toss and landing, the segment filling (1/3, 2/3), the 3rd coin erupting into the rain, the splash, and the countdown with the 30x badge. Afterwards the bar was back to n/3, and there were no errors.
  - **Performance during a full rain** (144 coins at once): 165 fps, render CPU 3.02 ms vs 2.70 without, GPU 2.28 ms.
- Still open:
  - 30x on the current luck system mostly removes Commons (see the Totem note).
  - The coins rolled during a rain only show on the bar after the rain.

## 2026-10-06 — Totem feature reverted (Claude)
- What: on the user's request the whole totem feature (the entry below) was removed. The five scripts were restored from their `*_before_totem` backups: no totem pieces, no Totem Roll, no slots, and the clover is back in its old place.
- Kept on purpose:
  - the earlier changes: hide flow and TextureKeeper;
  - the backups in `ServerStorage.UI_Backup` (`*_before_totem`);
  - the uploaded totem images and the local `HUD_Mockup/make_totem.py` + `HUD_Mockup/totem/`. Nothing uses them now.
- Where: `RollClient`, `GoblinPets`, `GoblinInventory`, `RollServer`, `GoblinInventoryServer`.
- Test: Play — no "Totem" in RollHud, clover at its old position, the roll returns no totem fields, no errors. No "totem" text is left in the five scripts.
- Still open: the luck note below (x25 on the current luck system mostly removes Commons) is still true for luck upgrades in general.

## 2026-10-06 — Totem pieces and the Totem Roll (Claude) — REVERTED, see above
- What (the user's idea; their picks: x25 luck, 1/40, random piece):
  - **Drops:** every roll (per click, not per column) has a 1/40 chance to drop a random totem piece (Foot / Body / Head). A piece you already have does not count.
  - **Totem Roll:** with all three, the next roll is a Totem Roll. Every column of that roll uses `Luck × 25`, and the pieces are used up. No piece can drop on a Totem Roll.
  - **Server:**
    - `GoblinPets.TOTEM_PIECES`, `TOTEM_CHANCE = 40`, `TOTEM_LUCK = 25`.
    - `GoblinInventory`: saved `data.totem`, player attribute `TotemPieces` ("Foot,Head"), `Inventory.TotemDrop` / `ConsumeTotem` / `SetTotemPieces`.
    - `RollServer`: answers with `totemRoll` (bool) and `totem = { piece, new }`.
  - **Client** (`RollClient`, "Totem" section at the bottom, plus the `Totem` hooks in playRoll and tryRoll):
    - **Bar:** `RollHud.Totem` has three dark slots with an empty grey silhouette; a filled slot gets the colour icon, a gold ring and a glow. The clover moved left; both hide while a menu panel is open.
    - **Drop:** when the roll lands, the piece pops out of the roll (rays, "TOTEM PIECE!", thud + ting) and flies into its slot. A duplicate shakes and crumbles ("ALREADY HAVE").
    - **Build:** after the roll the totem builds itself above the bar. Foot, body and head drop onto each other (wood thud, dust), then rays, a glow, stars and "TOTEM ROLL READY!". It then shrinks to a small breathing totem where the slots were, and ROLL turns gold with the text "TOTEM".
    - **Totem Roll:** the totem flies into the roll and bursts, a "TOTEM ROLL / x25 LUCK" banner shows over the roll (not in Hide Rolls mode), the clover shows 25x during the roll, and the slots come back empty.
    - **Auto Roll** waits for a drop or build animation before the next roll.
  - **Art:** `HUD_Mockup/make_totem.py` (background Blender, not the user's open file) renders a carved-wood goblin totem in the goblins' block style into `HUD_Mockup/totem/`. The three layers share one 512×768 frame so they stack exactly; there are also 256 px slot icons.
    - Icons: Foot 108270553598731, Body 121678160395393, Head 124669857890703.
    - Layers: Foot 108476858579811, Body 106467014384976, Head 118187489096471.
    - Thud SFX: 9126267420 (ProSoundEffects).
- Where: `ReplicatedStorage.GoblinPets`, `ServerScriptService.GoblinInventory`, `RollServer`, `GoblinInventoryServer` (Studio hook), `StarterGui.RollGui.RollClient`.
  - Backups in `ServerStorage.UI_Backup`: `*_before_totem`.
- Test:
  - **Studio hooks:** ServerStorage attribute `StudioTotemChance = 1` makes every roll drop a piece. `StudioTotemPieces = "Foot,Body"` / `"All"` / `"None"` sets the pieces.
  - **Played in Studio:** Foot then Head dropped into their slots; with the third piece the totem built and ROLL turned gold ("TOTEM"). The Totem Roll showed the banner and 25x on the clover, and the slots came back empty.
  - No errors; render CPU 2.60 ms, 166 fps.
- Still open:
  - **The luck system is weak for "x25":** luck multiplies every non-Common rarity by the same amount, so x25 mostly removes Commons (Common 50.8 → 4 %, Legendary 3.2 → 6.2 %, Mythic 1.6 → 3.1 %). Making luck raise every rarity's own chance (e.g. a 1/1000 becomes 1/40 at x25) would need a different `pickRarity`. Waiting for the user's decision.
  - The empty slot silhouettes could be a bit lighter.

## 2026-10-06 — Auto Roll / Hide Rolls only in the full-screen roll; ROLL brings it back (Claude)
- What (from the user's reference video):
  - Auto Roll / Hide Rolls only show while the full-screen roll (spotlight) is on. `syncSideButtons` listens to the `Spotlight` attribute.
  - **Hide Rolls is now a one-way "leave" button** (`setHidden(true)`): the tint fades, the HUD comes back, both buttons pop away, and Auto keeps rolling small at the top. The "Show Rolls" state is gone.
  - **ROLL** calls `setHidden(false)` first: during a hidden roll or Auto it brings the full-screen roll and the buttons back without an extra roll. Otherwise it rolls as usual.
  - The ROLL busy shade only shows in the full-screen roll; with Hide on, ROLL stays light green.
- Where: `StarterGui.RollGui.RollClient` (setBusy, syncSideButtons, setHidden, the ROLL / Hide handlers). Backup: `ServerStorage.UI_Backup.RollClient_before_hide_flow`.
- Test: in Play, clicked ROLL → Auto on → Hide → ROLL → Auto off.
  - After Hide: spotlight off, buttons hidden, shade 1.0, Auto kept rolling (2 rolls in 8 s).
  - After ROLL: spotlight on, buttons visible.
  - After Auto off: everything closed.
  - A single ROLL: buttons visible during the roll and gone after it.
  - No errors; render CPU 2.93 ms.
- Still open: nothing.

## 2026-10-06 — Roll icons no longer flash empty while cycling (texture keeper) (Claude)
- What:
  - **Cause:** Roblox drops textures that are not on screen. While the roll cycled, almost every icon swap showed a few empty frames while the texture came back. Probe: 42/42 icons were blank for 1–55 frames, 110 blank icon frames in one roll. In the live client this is less visible because of its cache.
  - **Second bug:** `ContentProvider:GetAssetFetchStatus` / `PreloadAsync` reported valid roll images (all FinalImages, the shared GoldImage) as **Failure**. That made the old `isLoaded` check useless.
  - **Fix:** RollClient has a `TextureKeeper` Frame (1 px, bottom left of RollGui). It holds one 1×1 ImageLabel per roll image at ImageTransparency 0.99, so the textures stay resident.
    - `preload(list)` adds keeper labels.
    - `isLoaded(id)` uses the keeper label's `IsLoaded`.
    - At start only the 42 cycle images (Reveal/Image) plus the defaults are kept. A result's final / gold / Shiny `IconBright` image is added when that roll comes in.
- Where: `StarterGui.RollGui.RollClient` (top, "Texture keeper"; playRoll's preload line). Backup: `ServerStorage.UI_Backup.RollClient_before_texture_keeper`.
- Test: Play with 3 columns: 3 blank icon frames per roll (only the very first frame of each new slot, while it is still scale 0) instead of 110; roll 3.76 s; render CPU 2.92 ms. Texture memory ~+40–60 MB (256 → ~300–320 MB).
- Still open: for the performance audit, re-upload the cycle icons at 512 px instead of 1024 px (a quarter of the texture memory, matters on mobile).

## 2026-10-06 — Shiny equip sound, columns in sync, faster base roll, side buttons only during a roll (Claude)
- What:
  - **Shiny equip SFX:** the sounds were played before they had loaded, so nothing was heard. `GoblinMutationFX` now makes preloaded templates in SoundService (`ShinyEquipSound1/2`, via ContentProvider). Your own pets play them with `SoundService:PlayLocalSound`; other players' pets play a clone on the pet. The TING sounds of the Shiny roll reveal are preloaded the same way (`ShinyTing*`).
  - **Columns in sync:** `SLOT_STAGGER = 0` and every slot pops in at the same moment, so all columns spin and land together (there was a left-to-right wave).
  - **Base roll speed:** `BASE_SPEED = 2` in RollClient, multiplied with `RollSpeed`. A new player now rolls at the old 2x tempo; the test panel's "1x" is this new base.
  - **Auto Roll / Hide Rolls:** only visible during a roll or while Auto Roll is on; they hide 0.6 s after the roll ends.
- Where: `ReplicatedStorage.GoblinMutationFX`, `StarterGui.RollGui.RollClient`.
- Test: in Play with 3 columns, all three landed together; the roll took 3.73 s (was 7.4 s). The buttons were hidden at the start, visible during the roll and hidden afterwards. The sound templates report IsLoaded = true. 165 fps, render CPU 2.58 ms.
- Still open:
  - With Hide Rolls on, the button can only be switched off again during a roll.
  - Server cooldown is still `3 / RollSpeed` (shorter than the 3.7 s animation, so no conflict).

## 2026-10-06 — Roll speed up to 20x (Claude)
- What:
  - `GoblinPets.MAX_ROLL_SPEED` is now 20, so speed can be anything from 1 to 20. The test panel offers 1x–5x and 20x.
  - At 20× the cooldown is 0.15 s.
  - The final wait in playRoll also divides by the speed now.
- Where: `GoblinPets`, `RollClient` (playRoll, `RollUpgradeTest` panel).
- Test: Auto Roll at 20× with 5 columns: 18 rolls in 8 s (0.48 s each, 5 goblins each), 164 fps, render CPU 3.39 ms, no errors.
- Still open: as below (how the upgrades are obtained).

## 2026-10-06 — Roll upgrades: roll speed x1–x5 and 1–5 roll columns (Claude)
- What:
  - The roll speed and the number of roll columns are real per-player upgrades, ready to be sold later.
    - They are saved in the profile (`rollSpeed`, `rollColumns`, clamped to `GoblinPets.MAX_ROLL_SPEED` / `MAX_ROLL_COLUMNS` = 5).
    - They are published as the player attributes `RollSpeed` / `RollColumns`.
    - They are set with `Inventory.SetRollUpgrades(player, speed, columns)`.
  - **Server:** RollServer gives `RollColumns` goblins per roll, and the cooldown is 3 s / RollSpeed.
  - **Client:** RollClient divides every roll timing by RollSpeed: the cycling, the stagger, the steps, the hold (incl. SHINY_TIME), the Shiny reveal (k = 1 / speed) and the exit.
    - The stage shrinks to fit the columns (`stageColumns`, 76 % of the screen width).
    - The cycle loop now counts real time (`t += task.wait(step)`), because steps get shorter than a frame at 5×.
  - **Testing:** a Studio-only panel "Roll upgrades (test)" (right side): speed 1x–5x and columns 1–5 through the Studio-only remote `GoblinInventoryRemotes.StudioRollUpgrades`, then roll as usual.
- Where: `GoblinPets` (MAX_*), `GoblinInventory` (data, publish, SetRollUpgrades), `RollServer`, `GoblinInventoryServer` (Studio remote), `RollClient` (speed, placeStage fit, the `RollUpgradeTest` panel). Backups: `UI_Backup.*_before_roll_upgrades`.
- Test (playtest, real rolls from the ROLL button):
  - 5 columns at 1×: 9.18 s (5 goblins side by side);
  - 5 columns at 5×: 2.29 s;
  - 164 fps, render CPU 2.95 ms, no errors.
- Still open: how the upgrades are obtained (shop / gamepass / rebirth); remove or keep the test panel once that exists.

## 2026-10-06 — Roll timing: every roll as long as a Shiny one (Claude)
- What:
  - The user picked timing option 2, "alles even lang". Every roll's result now stays `SHINY_TIME` (2.2 s) longer, and a Shiny reveal plays inside that time (`task.spawn`).
  - Normal and Shiny rolls now take the same time, and all columns leave together.
  - The `RollTiming` attribute, timing option 3 (reveal at 0.55×) and the `TimingTest` panel are removed. `shinyReveal(slot, face, k)` keeps the speed factor k for an adjustable roll speed later.
- Where: `StarterGui.RollGui.RollClient` (`SHINY_TIME` next to `HOLD_TIME`, playRoll).
- Test: real rolls from the ROLL button, timed from RollGui's `Rolling` attribute: normal 7.38 s, Shiny (forced with `StudioForceMutation`) 7.40 s. No errors. The test attribute is cleared again.
- Still open: Shiny inventory icons; more roll columns and an adjustable roll speed (the user's plans).

## 2026-10-06 — Shiny roll: concept 1 chosen; roll timing test panel with 1–2 columns (Claude)
- What:
  - **Concept 1 ("Light beam") is the user's pick.** Concepts 2 and 3 and the concept test panel are removed.
  - **Roll timing:** the user asked whether a Shiny roll (longer, because of the reveal) and a normal one can take the same time, and wants to test the options. RollGui attribute `RollTiming`:
    1. Shiny may take longer: normal 5.2 s, Shiny 7.4 s (the reveal blocks, then the normal hold). This is the default and my advice.
    2. Everything the same: every roll waits `SHINY_TIME` 2.2 s extra and the reveal plays inside it (7.4 / 7.4 s).
    3. Shiny faster: the reveal at 0.55× speed inside the normal hold (5.2 / 5.2 s).
  - With several slots everything always leaves together: playRoll waits for every slot, including its reveal.
  - **Studio test panel `TimingTest`** (right side, Studio only):
    - the 3 timing options;
    - "Kolommen: 1/2" (columns);
    - Normal roll / Shiny in column 1 / Shiny in column 2 / All Shiny.
    - Demo rolls don't touch the inventory. The duration of each roll is printed in the Output.
  - **Measured:** 2 Shiny columns at once run at 165 fps (render CPU 2.61 ms).
- Where: `StarterGui.RollGui.RollClient` (`shinyReveal(slot, face, k)` with speed factor k, `finish(..., k)`, playRoll timing, the `TimingTest` panel).
- Still open:
  - The user picks a timing option; then remove the panel and fix it in code.
  - **Plans of the user (info, not built yet):** more roll columns later (several goblins per roll, `GoblinPets.ROLLS_PER_SPIN`) and an adjustable roll speed. The slot layout already supports n slots, and CYCLE_TIME / steps / HOLD_TIME are the speed knobs.

## 2026-10-06 — Roll: normal results keep their outline, 3 Shiny reveal concepts, slower roll (Claude)
- What:
  - **Normal roll results** keep the black outline as during cycling. There is no gold glow any more (the user).
  - **Shiny results** get the gold glow behind the goblin, the FinalImage that every roll used to show (`showGlow`), after one of three reveal concepts:
    1. **Light beam:** a diagonal beam sweeps over the goblin and the glow appears behind it.
    2. **Star rain:** stars rain onto the goblin, then a flash and S-H-I-N-Y drops in letter by letter.
    3. **Charge & boom:** the goblin trembles while a ring closes in and sparkles get sucked in, then a flash, a shockwave, a shake and a big SHINY slam.
  - **Shared ending of all three:** "Shiny <name>" (Shiny in pale gold), white-gold odds counting up to the total, a "ting", and white-gold rays.
  - **Choosing a concept:** RollGui attribute `ShinyStyle` (1–3, default 1) until the user picks one.
  - **Studio test panel** (right side, Studio only): Normaal / Concept 1 / 2 / 3 play a demo roll without the server.
  - **Timing (the user, for every roll):** rolling is 1.2× slower (CYCLE_TIME 2.4, SLOT_STAGGER 0.42, steps 0.072 → 0.36); the result stays 1.5× longer (HOLD_TIME 1.875).
  - **Performance:**
    - the reveal effects live in their own plain Frame `ShinyFx` over the slot, not inside the slot's CanvasGroup: changes inside a CanvasGroup re-render it, which cost 165 → 99 fps;
    - tremble and shake move the whole slot;
    - measured: concept 3 runs at 165 fps (render CPU 3.17 ms), the same as a normal roll.
- Where:
  - `StarterGui.RollGui.RollClient`: faceFor (mutations / shiny), `showGlow`, `land`, the `shinyReveal` block (CONCEPTS), playRoll (`shinyStyle`), the test panel `ShinyTest`.
  - Backup: `ServerStorage.UI_Backup.RollClient_before_shiny_reveal`.
  - Icons: `HUD_Mockup/make_shiny_icons.py` → `HUD_Mockup/shiny/icons/` (512 px, 42 tinted + 42 bright; uploads of 1024 px timed out). The IDs are attributes `IconTint` / `IconBright` on `GoblinMutationAssets.Shiny.<Id>`; the bright one is the light beam in concept 1. The tinted one is not used yet (possibly for the inventory icons).
  - `HUD_Mockup/shiny/shiny_icons_compare.png`.
- Test: the panel buttons in a playtest; no errors; render CPU measured for concept 3 and a normal roll.
- Still open: the user picks a concept (then remove the panel and set the default); Shiny inventory icons.

## 2026-10-06 — Rule for every chat: measure effects before handing them over (Claude)
- What: the user approved the lag-free Shiny with "learn from your mistakes".
  - Rule from now on: every new visual effect is measured in a playtest before it is called done. Measure `Stats.RenderCPUFrameTime` / `RenderGPUFrameTime` with 6 pets carrying the effect against 6 without, and report the numbers.
  - Never animate Texture properties on the moving pets.
- Where: this log, plus memory (`performance-check-effects`, `roblox-studio-gotchas`).
- Test: none needed (no game change).
- Still open: Shiny roll animation and Shiny inventory icons, with the user.

## 2026-10-06 — Shiny made lag-free (Claude)
- What: the user reported that the whole game lagged with Shiny goblins.
  - **Measured** (`Stats.RenderCPUFrameTime`, 6 equipped pets): normal 3.2 ms; Shiny 9.6 ms (game 165 → 68 fps).
  - **Testing one part at a time:**
    - the SurfaceAppearances, rim Highlights, particles and lights cost nothing measurable, and static Textures cost nothing either;
    - **moving a Texture's offset on a walking, animated goblin costs ~5 ms per goblin**, even at 30 Hz, on the body or on the clothes alike.
  - **Fix:** the sliding-band glint is gone.
    - The glint is now a soft white shine over the whole goblin every ~2.2 s (the rim Highlight's fill, eased in and out) plus a big four-point star and small sparkles flashing on the body.
    - The rest of Shiny is unchanged: white-gold layer, metal shine, glitter, small light, faint rim, equip moment, footstep sparkles, nametag.
  - **Result:** 6 Shiny goblins 3.19 ms, the same as 6 normal (3.21 ms), 165 fps.
- Where: `ReplicatedStorage.GoblinMutationFX` (SHINE_PERIOD / SHINE_TIME / SHINE_PEAK, ShinyTing emitters; the Texture code is removed). Leftover assets: glint.png / glint_ff.png (not used).
- Test: playtest, 6 normal vs 6 Shiny twice, no errors.
- Still open: the user checks whether the new shine looks good. If a moving band is still wanted, it would have to be done without moving Textures (e.g. a separate unanimated shell part, not tested yet).

## 2026-10-06 — Shiny mutation built into the game (Claude)
- What: the Shiny mutation (1/200, fixed chance) now works for real: rolling, saving, the look on every goblin in the world, and the nametag. Full description: section 1, "Mutations".
  - **Design choices (the user):**
    - Model: the "B" test goblin with the light white-gold filter.
    - Glint: diagonal, top to bottom, 1.2× slower.
    - Extras: glitter, the equip moment (flash softened) and footstep sparkles.
    - Light: "only a very small bit".
    - Rim: "less clear".
    - Nametag: design 1 ("Shiny" in pale gold before the name) plus a glint over "Shiny".
  - **Not done on purpose** (the user wants to design these together next): the Shiny roll animation and the Shiny icons in the inventory. The inventory only got what is needed to work: its own stack per mutation, total odds, and the tooltip name.
- Where:
  - `ReplicatedStorage.GoblinPets` (mutation rules), `ServerScriptService.GoblinInventory`, `GoblinInventoryServer` (hook), `RollServer`.
  - New `ReplicatedStorage.GoblinMutationFX` and `ReplicatedStorage.GoblinMutationAssets.Shiny` (42 SurfaceAppearances).
  - `GoblinPetRenderer`, `GoblinTagFX`, `StarterPlayerScripts.GoblinPetClient`.
  - Backups: `ServerStorage.UI_Backup.*_before_mutations` (GoblinPets, GoblinInventory, GoblinInventoryServer, RollServer, GoblinPetRenderer, GoblinTagFX, GoblinPetClient).
  - Textures: `HUD_Mockup/make_shiny_textures.py` → `HUD_Mockup/shiny/textures/` (42 files + `index.json`), plus `make_shiny_maps.py`, `make_glint.py` and `glint.png`.
  - The temporary preview folders (`MutationPreview_TEMP`, `ShinyTags_TEMP`) are removed.
- Test (playtest):
  - Granted "Mystic:Shiny, Basic:Shiny, King:Shiny, ShadowStalker:Shiny, Nebula:Shiny" plus a normal Mystic. The snapshot items carry `mu`; EquipBest picked the 6 Shiny ones; the attribute carries `m`.
  - Every Shiny pet got its SurfaceAppearances, glint textures, light and rim.
  - The tags read "Shiny Shadow Stalker Goblin  1/12,800" and "Shiny King Goblin  1/6,400".
  - No errors.
  - The camera could not be moved for a close-up (execute_luau resets it), so the close look in the game is still to be checked by the user.
- Still open:
  - The Shiny roll animation and the Shiny inventory icons (next, together with the user).
  - Shiny versions in the Index.
  - Huge, Glitched and Rainbow.
  - The 42 tinted textures come from the local `Goblin_Variants` textures; if a goblin was re-textured in Studio later, its Shiny texture must be made again.
  - Divine and Celestial still show "1/?" (their odds are not decided).## 2026-10-06 — Shiny nametag: 8 designs to choose from (Claude)
- What: 8 Shiny versions of the real goblin nametag, from simple to rich, on 8 copies of the Shiny test goblin in a ring round the spawn pool (centre (109, −5), radius 15–16, facing inwards, a big number in front of each).
  - **Shared by all:** the total odds (Rare 1/8 × Shiny 1/200 = **1/1,600**).
  - **The designs:**
    1. "Shiny" in white gold before the name, normal rarity plate.
    2. A small white-gold "SHINY" under the plate, between two twinkling stars.
    3. A white-gold "SHINY" pill above the odds.
    4. A white-gold plate (the whole tag in white gold), "Shiny" before the name.
    5. Rarity plate, white-gold odds, a twinkling star left and right of the plate.
    6. A pearl-white plate with small stars twinkling round it.
    7. A white-gold plate with a glint running over the name.
    8. Deluxe: white-gold plate and odds, the SHINY pill with glint, stars beside the plate and the pill, and the name glint.
- How:
  - In the client of the running Play session, `GoblinTagFX.Attach` builds the tags with a fake `fx` (design = Mystic with a fake Rarity).
  - The `GetRarity`, `RatioText` and `RarityRank` of that context's own module instances were patched for the fake rarities ShinyRare / ShinyGold / ShinyPearl. That does not touch the game's tags (execute_luau has separate module instances).
  - The extras (pill, stars, glints) are added into the tag's Root (40 px per unit, laid out like `placeIn`).
  - Folder `Workspace.ShinyTags_TEMP` and `_G.TagDemoConn` (client, Play only).
- Still open: the user picks a design; then it is built into GoblinTagFX properly (a mutation-aware palette, prefix, label and total odds).
- Round 2 (the user: always "Shiny" before the name like their picture, and a plate wide enough for it):
  - The name is now "Shiny Mystic Goblin", with "Shiny" in pale gold (#F5E3A1) through RichText. The plate is built for the whole name.
  - In GoblinTagFX the plate width is clamped by `T_MAX` 8.8 units, so longer names shrank instead. The test uses a client-side clone, `ReplicatedStorage.GoblinTagFX_WideTest`, with `T_MAX` 12 (`loadstring` is not available in execute_luau, but setting the Source of a clone and requiring it works). Name size 36 at REF_PPU, plate 10.3 units.
  - **When building it in for real:** raise `T_MAX` in GoblinTagFX (or add the prefix width).
  - The 8 new designs, all with the "Shiny" prefix:
    1. only the prefix;
    2. a glint over the name;
    3. white-gold odds;
    4. white-gold plate edge, gems and triangles (dark rarity fill kept);
    5. edge + odds + twinkling stars beside the plate;
    6. pearl plate + twinkles round it;
    7. white-gold plate + glint over the name;
    8. deluxe (edge, odds, glint, side stars, twinkles).
  - Also at the user's request, the body light that lit up the surroundings was made much smaller (PointLight Range 9 → 3.5, Brightness → 0.7), on the test goblin and the 8 copies.

## 2026-10-06 — Shiny mutation: two design previews (Claude)
- What: the user's mutations plan, with fixed chances that luck does not change: Shiny 1/200, Huge 1/800 (1.8× size), Glitched 1/3,200, Rainbow 1/12,800. They stack and their odds multiply.
  - The first step was two Shiny designs to choose from, on Mystic:
    - **A, "Sparkle":** the goblin's own colours, a bit glossy, a white glowing rim, dense white glitter and big four-point stars on the body, a cool light.
    - **B, "Golden":** a real metallic gold statue. A gold version of the goblin's own texture (luminance → gold ramp, eyes stay dark) is put in a SurfaceAppearance with full metalness and low roughness, plus a gold rim and rising gold glitter.
- Where:
  - `Workspace.MutationPreview_TEMP` (**temporary**: A left, normal middle, B right, by the spawn at z −21). Delete it once a design is chosen.
  - `HUD_Mockup/make_shiny_maps.py` → `HUD_Mockup/shiny/`: mystic_gold 82306400297961, metal_full 82917117205864, rough_low 75294398209894, rough_gloss 92699093006002.
  - For B every goblin needs its own gold texture. All textures are local in `Goblin_Variants/<name>/<name>_texture.png`.
- Test: Edit captures from (108.7, −55.2, −11) looking at (108.7, −57.2, −21).
- Still open: the user picks A or B. A SurfaceAppearance that reuses a goblin's original TextureID as its ColorMap renders grey, so A keeps the original texture.
- Follow-up (same day):
  - The user liked B's effect, but yellow gold would clash with Rainbow.
  - A became **C, "Chrome"**: the goblin's own colours (re-uploaded texture 76355213819079) with metalness 150 (`metal_mid` 71567097059396), low roughness, white-silver glitter and a white rim.
  - B was made lighter: **white gold** (`mystic_white` 111319206996393, made by the same script, near white with a touch of warmth), with a white-cream rim, glitter and light.
  - Both changes were made **only in the running Play session** (the user was looking at them). The Edit copy of `MutationPreview_TEMP` still has the first versions.
  - The user found white gold too colourless and asked for a "layer filter" over the goblin.
    - B is now the goblin's own texture with a light white-gold layer: every colour moved 30 % towards a warm white (255, 244, 214) and made 6 % brighter, eyes unchanged.
    - Plus metalness 150, low roughness, the white rim and glitter.
    - `mystic_tint` 120080332512543, made by `make_shiny_maps.py`. The strength is the 0.30 in that script.
    - A freshly uploaded texture renders grey for a few seconds until it has loaded.
  - **Glint** (the user: animated, diagonal over the whole body, top to bottom with velocity):
    - Every face (Front / Back / Left / Right / Top) of every MeshPart gets a `Texture` "Glint" (`glint.png` 81960862416839, `HUD_Mockup/make_glint.py`: a tileable 45° white band plus a thin second line), with StudsPerTile 16.
    - The offsets are set per part so the band runs on as one straight line across all parts:
      - per-face U/V directions in part space: Front U = −X, Back U = +X, Right U = −Z, Left U = +Z, V = −Y;
      - `OffsetStudsU = wU·pos − halfU + k`, `OffsetStudsV = wV·pos − halfV`;
      - the sign was checked with two test cubes.
    - The sweep runs from above the head (s = (wU + wV)·p minimum over the bbox − 2.2) to under the feet (max + 2.2), in 0.85 s with ease in-out cubic, then a 1.7 s pause in which the textures are hidden. Each texture has its own range, so all faces sweep together.
    - Made in the client of the running Play session (`_G.GlintConn`).
    - Then made 1.2× slower and more often (the user): sweep 1.02 s, pause 0.9 s.
  - **Equip moment + footstep sparkles** (the user picked these two extras), as a looping demo on B (client, `_G.ShinyDemoId`):
    - Equip: the goblin pops in from 0.05 of its size with a Back ease (0.55 s). The base scale of the templates is about 0.03 (`Model:GetScale()`): `ScaleTo(1)` makes it ~33× too big, so always multiply by the base scale.
    - At the same time: a white Highlight flash (0 → 1 in 0.5 s), a white neon sphere 1 → 8 studs that fades, a light burst, a ring of sparkles shooting out along the ground, and 34 white-gold stars raining down from above.
    - Sound: Glockenspiel C7 (72277486589033, vol 0.9) plus "crystal fx" (79130903404606, vol 0.45, pitch 1.1), both free Creator Store audio.
    - Footsteps: while walking, every 0.26 s, alternating Foot_L / Foot_R (bone WorldPosition): 3 star twinkles plus 2 small sparkles stay on the ground and fade in 0.6–1 s. No trail.
    - The flash first washed out the screen when the camera was close. At the user's request it was made softer:
      - Highlight fill starts at 0.4 instead of 0;
      - the sphere starts at 0.6 transparency, grows to 6 studs instead of 8, and is cream white;
      - the light is Brightness 2.5, Range 9 (was 8 / 16).

## 2026-10-06 — Grid-view button with the user's four diamonds; equipped tooltips that sometimes did not show (Claude)
- What:
  - **Grid-view button:** the icon is now the user's four cyan diamonds (images/69.png). One diamond is cut 1:1 and shown four times, so each can move on its own. This replaces the triangles / pinwheel.
    - Click: a small ripple, where one after the other (clockwise) the diamonds pop (1.35 → 1) and slide to their place; the button pops a little.
    - Grid view on: the diamonds pack closer together (× 0.86), and the dark-blue plate with the breathing cyan rim stays behind the button.
  - **Tooltip on the equipped goblins** "sometimes not showing":
    - Cause: neighbouring goblin boxes overlap a few px, so the old box's MouseLeave came after the new box's MouseEnter and hid the new tooltip.
    - Fix: `withTip` remembers the owner (`tipOwner`), and only the owner may hide the tip.
    - Also: the bar's glow, shadow, pedestal and badge layers have `Interactable = false`, so they never block the goblins' hover or clicks.
- Where: `StarterPlayerScripts.GoblinPetClient`: `GRID_GEM`, `GEM_POS`, `showGrid`; `withTip` / `tipOwner`; the `eqBar` layers. Asset `HUD_Mockup/equip_bar/grid_diamond.png` 84985184296228 (`HUD_Mockup/cut_grid_gem.py`).
- Test (playtest):
  - Moving the mouse from Eclipse to Mystic and back over the overlap: the tooltip stays visible with the right name.
  - Grid view on: the diamonds packed on the plate.
  - The user then played in the same session, so I stopped clicking; the off state was not captured.
- Still open: the diamonds could be a touch bigger in the button if wanted (now 19 × 17 px).

## 2026-10-06 — Equipped bar: goblins in their pedestals up to the hips, one gem for all, pinwheel grid button (Claude)
- What (the user's feedback with a screenshot next to the reference, plus a sketch for the grid button):
  - **Goblins bigger and lower, like the reference** ("bij hun buik in de blauwe bar"):
    - the image went from 98 to 118 px, with the head top where the reference has it;
    - the pedestal crosses at the hips;
    - everything below the pedestal's band is clipped away, so the legs never show.
    - Empty-slot silhouettes are clipped the same way.
  - **Gentle hover:** first the pedestal grew along with the hover. The user then found the bounce too big (the goblin went over the small pedestal), so the pedestal now stays put and the goblin only grows to 1.04 with a 1.5° tilt.
  - **Gem:** no colour per rarity any more. It is one image, cut 1:1 out of the user's card reference (gold star, blue stone), on the cards and in the bar.
  - **Grid-view button:** in the on state every shard turns to its opposite direction, a pinwheel with all four tips at the middle (the user's sketch). The animation is a half spin while shrinking, a swap to the new shape, then the rest of the spin while it springs back.
- Where:
  - `StarterPlayerScripts.GoblinPetClient`: `eqBar` (GOB, TOP, CUT, `eqBar.clip`, `clipInset`); `makeEquippedIcon` (clip, gentle hover, glow `S * 1.6` at (0.52, 0.42)); `CARD.gem` (`CARD.gemImage`); `showGrid` spin; `GRID_ATLAS`.
  - Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_hip_goblins`.
  - Assets: `gem_ref.png` 136161945361156 (`HUD_Mockup/cut_gem_ref.py`), `grid_atlas.png` 108254907018184 (right half = the pinwheel; `HUD_Mockup/make_equip_bar.py`). `eq_preview.png` now shows the hip-height layout next to the reference.
- Test (playtest): ShadowStalker / Eclipse / Mystic / Seraph / Eternal / Nebula equipped. Same size as the reference, legs hidden, gems all blue. Hovering Mystic: a small growth, it stays in its pedestal. Grid view on: pinwheel. No errors.
- Still open: nothing for this change.

## 2026-10-06 — Equipped bar 1:1 from the user's reference, gems, grid view (Claude)
- What:
  - **Equipped bar rebuilt 1:1 from the user's reference** (images/63.png), with its colours. It was measured in the reference: the frame, inner edge shading, pill, line, slot positions, badge circles and pedestal edges.
  - **In the bar:**
    - "Equipped" + the dark "x/6" pill top left, with the cyan line;
    - numbered badges 1–6 and cyan pedestals;
    - the goblins with a thin black outline and their own icon effect;
    - a rarity glow behind each goblin, without the old star sparkles;
    - the odds tilted over the head, between the left ear and the head;
    - empty slots as silhouettes.
    - The old "x/6 Equipped" pill bottom left (goblin icon) is gone.
  - **Gem:** a small gold star with a stone in the rarity colour, at the top right of every goblin, on the inventory cards and in the equipped bar. It sways, and a sparkle flashes on it now and then. On the cards, "x2" moved to the top left (after the equipped tick), as in the user's card reference.
  - **Grid view:** a button with 4 blue triangles bottom left (from the user's icon reference).
    - It makes the cards smaller: 8 per row instead of 6, about 1.8× as many goblins in view. The equipped bar does not change.
    - On: the triangles flip one by one to point inwards, the button pops, and a dark-blue plate with a breathing cyan rim stays behind it. Off: they flip back.
- Where:
  - `StarterPlayerScripts.GoblinPetClient` (Studio only):
    - `iconGlow(..., noSparkles)`;
    - the grid-view button in the bottom bar (`invUI.gridView`, `invUI.rerender`);
    - `eqBar` + the `EquipBar` image;
    - `CARD.gem` before `makeCell`, and `makeCell` with `gk`;
    - `shinyText` `place.rot`, `oddsTag`, `makeEquippedIcon(item, slot)`;
    - `renderInventory` (count, slot index, grid sizes).
  - Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_equip_bar`.
  - Assets: `HUD_Mockup/make_equip_bar.py` → `HUD_Mockup/equip_bar/`:
    - bg 97221895569819, pedestal 139770799777132, badge 95165509693778;
    - gem frame 116804317816272, gem stone 82935310930286, grid atlas 75306416302597;
    - `eq_preview.png` = reference vs rebuilt, `gems_preview.png`.
- Test (playtest):
  - 6 Celestials, then a mix of Rare / Legendary / Epic / Uncommon / Mythic with one empty slot: odds, gems and glows show in each rarity colour, and the empty slot shows a silhouette.
  - Grid view on/off: 8 columns, the equipped bar is unchanged, the plate is visible.
  - No errors.
- Still open: our goblin renders have longer legs than the reference goblins, so they stand slightly smaller (about 85 %) with their soles behind the pedestal. Grid view is not saved between sessions.

## 2026-10-06 — Goblins no longer teleport to / from enemies and walk through each other (Claude)
- What (the user's video: some goblins snapped to a clicked enemy and back when leaving its range, and goblins bumped into each other on the way back):
  - **Cause of the snapping:** the renderer snapped every goblin more than `TELEPORT_DIST` = 60 studs from its target. A click reaches 55 studs (plus the enemy's size) and a clicked fight is only let go at 65 studs, so the goblins furthest away snapped.
  - **Fix:**
    - `TELEPORT_DIST` 250 is now only a safety net.
    - Far from its spot a goblin sprints, up to 46 studs/s.
    - A real teleport of the player (≥ 40 studs in one frame) still brings the goblins along at once.
  - **Bumping:** the pet-to-pet push (`separate`) is removed, so goblins pass through each other. They still keep off their own player.
  - **After a fight:** each goblin takes the nearest formation slot (`reslot`), so the paths back don't cross.
  - **Arriving later:** goblins that are still running over (far click) only join the strikes once they arrive, and enemy strikes only hit goblins that have arrived.
- Where:
  - `ReplicatedStorage.GoblinPetRenderer`: constants `CATCHUP_*`, `TELEPORT_DIST`, `PLAYER_JUMP`; `updatePet` (maxSpeed); `keepOffPlayer`, `reslot`; `step` (teleport snap, `group.fought`).
  - `StarterPlayerScripts.GoblinCombatClient` (`arrived`) and `StarterPlayerScripts.EnemyAnimator` (`targetPet`). Local copies in `Combat/` updated, hash-identical: 7112 / 719824004 and 38731 / 30449050.
  - Backups: `ServerStorage.UI_Backup.GoblinPetRenderer_before_no_teleport`, `GoblinCombatClient_before_arrive_check`, `EnemyAnimator_before_arrive_check`.
- Test (playtest, 6 goblins, positions measured every frame):
  - Clicking the Elder Root Golem from 58 studs: all 6 ran over in 1.0–1.2 s, largest step 0.36 studs per frame (no snap).
  - Walking out of the 65 stud range: all 6 ran back into formation, largest step 0.39 studs (no snap).
  - A 99 stud teleport: the goblins stood in formation next to the player 3 frames later.
  - No errors in the output.
- Still open: nothing for this fix. The video's exact moments could not be matched frame by frame, but the cause and fix are measured in Studio.

## 2026-10-06 — Inventory: tabs, search, count pills, one Equip All / Unequip All button (Claude)
- What (after the user's "My Pets" reference, not 1:1):
  - **Tabs left of the panel:**
    - Goblins on top (where "Titles" is in the reference), with the user's goblin-in-a-box icon cut out of its navy background.
    - Potions, Titles and Items as empty placeholders: they show a "Coming soon" page with a floating icon and a line per tab.
    - The open tab has the panel colour and merges into the panel; the others are darker and tucked behind the edge.
  - **Title:** the dock backpack now stands before "Inventory" (wiggle + glint now and then).
  - **Sorting:** the SORT button is gone; the grid is always sorted by rarity (rarest first).
  - **Pills bottom left:** the old "x/6 Equipped • N Goblins" line moved here as "x/6" (basic goblin icon) and "N / 10,000" (backpack).
  - **Search top right:** a dark-blue pill with a magnifier; filters by goblin name or rarity. Shows "No goblins found" when nothing matches; × clears.
  - **Bottom right:** one button for both: green "Equip All" (EquipBest) while nothing is equipped, red "Unequip All" otherwise. The header EQUIP BEST button is gone.
  - **Layout:** the panel is 780 × 630 now (was 580): the header is lower without the info line and the sheet stops above the bottom bar. The equip row is unchanged.
- Where:
  - `StarterPlayerScripts.GoblinPetClient` (Studio only): `makePanel` options `left` / `sheetBottom`; the Inventory section (`invUI`, `invPage`, tabs, search, pills, toggle); `invUI.applySearch` / `invUI.leaving` before `renderInventory`; the toggle handler replaces the old sort / best handlers; `open()` calls `invUI.reset()`; `updateScale` counts the tab column.
  - Backup before this change: `ServerStorage.UI_Backup.GoblinPetClient_before_inventory_tabs`.
  - Icons: `HUD_Mockup/make_inventory_icons.py` → `HUD_Mockup/inventory/` (tab_goblins 87311180912818, tab_potions 78471968777362, tab_titles 84542011393714, tab_items 74613359933132, search 102008035420924; `icons_preview.png`).
  - No server change: Unequip All uses the existing `SetEquipped("*", false)`.
- Test: playtest with all 42 designs granted (`StudioGrantGoblins`):
  - tab switching (Potions / Titles / Items show "Coming soon", Goblins comes back);
  - search "mythic" → 5 cells, "zzz" → "No goblins found";
  - Equip All → 6/6 + red "Unequip All"; Unequip All → 0/6 + green "Equip All";
  - no errors in the output.
- Still open:
  - Content and final icons for Potions / Titles / Items.
  - The soft glow behind the "Coming soon" icon is very faint on the light sheet.
  - On very narrow windows the dock can overlap the tabs (the panel's minimum scale is 0.42).

## 2026-10-06 — Coin bar starts in the shape of the $ (Claude)
- What:
  - The black bar behind the coins no longer starts with a straight vertical edge in the dollar sign. The black now runs along the left side of the "$" in its shape: a black "$" with a thick round outline (UIStroke 7, LineJoinMode Round) behind the amount. The fade starts right there.
  - The bar is black at its start (gradient 0.03 → 0.18 → 0.45 → 0.7 → 1).
  - The halo pops along with the amount (`haloPop`).
- Where: `StarterPlayerScripts.GoblinPetClient` (Studio only), the coins section: `coinHalo`; `coinText` ZIndex 3.
- Test: playtest capture on a flat blue backdrop: the black hugs the left contour of the $ and runs into the fade, with no straight start edge.
- Still open: spawn chances + locations; Ascend contents.

## 2026-10-06 — Inventory cells in the reference card style (Claude)
- What: every goblin cell in the inventory grid now uses the card format the user approved (equipped row unchanged).
  - **Card:**
    - dark outline, dark blue ring, light blue band, navy inner line;
    - lavender → mint background;
    - the goblin big from the head to the belt, with a thin black outline (8 tinted copies behind it, 1.6 px), its legs hidden behind a hill in the rarity colour.
  - **Odds:** on the hill as **"1/4"** (user: "/" instead of "in"; `GoblinPets.RatioText`), in the rarity colour with a dark outline, auto-shrinking for long numbers.
  - **Corners:** "x2" white top right; a green outline + green tick when equipped. Delete mode still marks red.
  - **Effects:** the goblin's IconFX effect stays on it.
  - **Hover:** the whole goblin (with outline and effect) grows (holder UIScale). The card outline no longer fades on hover.
  - **Grid:** cells are 108 × 104 (were 104 × 118).
- Where:
  - `StarterPlayerScripts.GoblinPetClient` (Studio only): `CARD`, `makeCell`; `cellHover` (the edge is optional); grid `CellSize`.
  - Backup: `ServerStorage.UI_Backup.GoblinPetClient_before_card_cells`.
  - Hill texture: `rbxassetid://89890540329845` (`HUD_Mockup/make_arch.py`).
- Test: playtest with 45 goblins. The grid shows cards for all rarities (Celestial cyan, Divine yellow, Mythic pink, …), equipped cards are green with a tick, effects are visible, and the console is clean.
- Still open: spawn chances + locations; Ascend contents.

## 2026-10-06 — Roll streaks stand still; inventory cell redesign shown as a preview (Claude)
- What:
  - **Roll streaks** (user request): the two white streaks no longer whoosh during the roll.
    - While the slot cycles they stand still beside the goblin at its middle height (they fade in with the slot).
    - When the result lands they slide in a little (about 50 px) towards the goblin with a short flash and a soft 0.7 s fade (`addStreaks` / `landStreaks` in RollClient).
  - **Inventory cell redesign** (user reference: Mushroom "1 in 4" card): **preview only, nothing changed in the game yet** (the user asked to see an example first).
    - Card:
      - dark outline, dark blue ring, light blue band, navy inner line;
      - lavender → mint background;
      - the goblin big, from the head to the belt, with its legs hidden behind a hill in the rarity colour;
      - "1 in X" on the hill in the rarity colour with a dark outline;
      - "x2" white top right; the green tick for equipped.
    - Framing: all renders share one silhouette (ears 0.176–0.867 of the image, feet at 0.98; measured with `scratchpad/icon_extents.py`). The goblin image is 1.12 × the inner area at (−0.127, 0.02), so the belt meets the hill top.
    - New texture: hill `rbxassetid://89890540329845` (white, tinted `lighten(rarity colour, 0.62)`, made by `HUD_Mockup/make_arch.py`).
    - Preview picture: `HUD_Mockup/previews/inventory_cells_preview.png`.
- Where: `StarterGui.RollGui.RollClient` (streaks). The cell redesign is waiting for the user's OK (it will go into `GoblinPetClient` `makeCell`; the equipped row stays as it is).
- Test: a roll capture shows the still streaks at both sides; the preview was built in a play session (8 rarities).
- Still open: the cell redesign itself (after approval), spawn chances + locations, Ascend contents.

## 2026-10-06 — Roll: light streaks at the sides; coin fade starts at the $ stripe (Claude)
- What:
  - **Roll animation:** two white light streaks (a thin tail with a bright soft head pointing at the goblin) rush in from the left and right and vanish against the goblin.
    - While a slot cycles: one pair every 0.3 s.
    - On landing: one longer, brighter pair, then one more pair 0.35 s later.
    - Positions are random around the goblin's middle. The streaks scale with the roll display, so they are small in Hide Rolls mode.
    - Texture: `rbxassetid://78863255721677` (512 × 64, head on the right), made by `HUD_Mockup/make_streak.py`.
  - **Coin fade:** now starts at the left edge of the stripe through the $ (`FADE_X` 27.9 → 24.0). The right end stays at the Shop icon (`FADE_W` 191.5).
- Where:
  - `StarterGui.RollGui.RollClient` (Studio only): `streakPair`, `STREAK_IMAGE`, `STREAK_EVERY`, called in `playRoll`.
  - `StarterPlayerScripts.GoblinPetClient` (Studio only): `FADE_X` / `FADE_W`.
- Test (playtest):
  - captures during cycling show the pair at both sides at mid height;
  - sampling counted 2–4 streak images live during a roll.
  - Side note: in a fresh test session the first roll can show empty frames while the goblin images are still loading (existing preload behaviour, not changed).
- Still open: spawn chances + locations; Ascend contents.

## 2026-10-06 — Roll display: name closer to the feet, odds a touch closer (Claude)
- What:
  - The name under a rolled goblin moved 34 design px up, so it now sits just under the feet (`SLOT_SIZE * 0.36 - 28`).
  - The tilted odds at the top left moved a little towards the goblin: `Position (0, 3, 0.125, 0)`, was `(0, -6, 0.1, 0)`.
- Where: `StarterGui.RollGui.RollClient` (Studio only), `buildSlot`.
- Test: two rolls in a playtest (Cyborg 1/32, Leaf 1/2); the name sits about 15 px under the feet.
- Still open: spawn chances + locations; Ascend contents.

## 2026-10-06 — Polish: gear icon, smaller nametags, softer high-rarity attack glow, inventory keeps its scroll, longer coin fade, odds on roll icons, boar circle (Claude)
- What:
  - **Settings icon:** the gear's centre hole showed jagged cut-out pixels. It is now a solid dark hole (`fillHoles` in `HUD_Mockup/ui_cut.py`). New atlas: `rbxassetid://128455311894694` (`DOCK_ATLAS`).
  - **Goblin nametags are 1.3× smaller** (`GoblinTagFX`: `TagFX.SIZE = 1 / 1.3`). The world unit, the min/max pixel sizes, the detail-level thresholds and the animation throttles all follow it.
  - **Goblin attacks are a bit less bright** (`CombatFX`): a new glow level `CFX.glow`, set per attacking goblin's rarity in `GoblinAttacks.Play` with `CFX.withGlow`.
    - Common–Epic 0.9, Legendary 0.82, Mythic 0.75, Divine 0.68, Celestial 0.62.
    - What it changes: neon parts get a slightly darker colour, and lights and particle / trail / beam light emission are scaled. The effects themselves (shapes, timings) are unchanged.
    - Delayed effects keep the level they were scheduled with (`CFX.after`, projectile hits). Enemy effects stay at 1.
  - **"Equipping makes me shoot up":** reproduced as the inventory grid jumping back to the top after every equip / unequip (CanvasPosition 400 → 0).
    - `renderInventory(toTop)` now keeps the scroll position. Opening the inventory and changing the sort still start at the top.
    - The character itself was tested and never moved: all 42 goblins equipped, no collidable parts, no script touches the character's physics.
  - **Coin fade** reaches the right edge of the Shop icon (`FADE_W` 187.6 design px, still starting in the middle of the $).
  - **Roll display:** the odds ("1/32", the same text as the world tags) sit at the top left of the goblin, tilted −12°, in the rarity colour with a dark outline. The name sits right under the icon.
  - **Boar circle:** moved towards the head (`THREAT.CrystalbackBoar.forward = 0.3` per unit of model scale, about 1.1 studs), so it is centred under the body.
- Where:
  - Studio only: `ReplicatedStorage.GoblinTagFX`, `StarterPlayerScripts.GoblinPetClient`, `StarterGui.RollGui.RollClient`.
  - Local = Studio (hash-checked): `Combat/CombatFX.luau` (18539), `Combat/GoblinAttacks.luau` (44950), `Combat/EnemyAnimator.client.luau` (38495).
  - `HUD_Mockup/`: `ui_cut.py`, `ui_cut_params.json`, `dock_atlas.png`, `cut_settings.png`.
- Test (playtest):
  - gear hole, coin fade and boar circle checked on screen (crops);
  - nametags visibly smaller;
  - a roll shows "1/32" at the top left with the name under it;
  - grid scroll was 400 → 0 before the fix.
  - The fix itself was not clicked through, because the user was playing in the same test session; please check once by scrolling down, equipping and seeing the list stay put.
- Still open: spawn chances + locations; Ascend contents.

## 2026-10-06 — HUD 1:1 from the user's mock-up, Roblox hand cursor, enemy circles, walk round enemies, knockback, bigger icon effects (Claude)
- What:
  - **New HUD, a literal 1:1 copy of the user's mock-up** (the user explicitly asked for 1:1; it is their own design):
    - Left dock, 2 × 3: Inventory (backpack), Shop (treasure chest), Index (book), Settings (gear + toggle), Teleport (map + pin), **Ascend** (gold bust with laurels + arrow, new).
    - The six painted icons were cut out of the mock-up (dark outline = barrier, flood fill from outside, label text and see-through holes removed), upscaled 2× and packed into one atlas, `rbxassetid://94268795310200`. The top half holds the icons; the bottom half holds their white silhouettes, which drive the glint sweep.
    - Positions and sizes were measured on the mock-up. The mock-up is the Studio viewport scaled ×1.077, so design px = mock-up px / 1.077 with a 58 px top bar, laid out for a 772 px high screen.
    - Names use Montserrat Heavy 18 with a black 3 px stroke. The "x/6" count sits over the backpack.
    - Coins: "$0" in flat gold with a black outline. Behind it is a black fade that starts in the middle of the $ and runs out at x ≈ 160; longer amounts get a longer fade.
    - Comparison screenshots (mock-up vs game) match within 1–3 px.
  - **Dock animations:**
    - Hover: grows to 1.12, tilts, a soft coloured glow lights up behind it, a light glint sweeps over the icon, sparkles appear, and the name grows.
    - Click: squash, then bounce, plus a burst of its own:
      - Inventory: stars + ripple + wiggle
      - Shop: coins jump out of the chest
      - Index: the book flaps + sparkles
      - Settings: the gear spins a full turn
      - Teleport: double ripple from the pin + hop
      - Ascend: rising gold + hop
    - Idle: every 2.8–5.6 s one icon does a little something (wiggle, coin pops out, glint, ripple, hop). A glint also runs over the coins every 5–8 s.
    - Rolled goblins landing in the backpack give a sparkle burst.
  - **Ascend panel:** a "Coming soon" container, like the Shop.
  - **Bottom bar, styled like the mock-up:**
    - ROLL is 272 wide in the mock-up's deeper greens (88,188,62) → (2,143,42), with a (0,66,12) edge, a see-through white dice and off-white text.
    - Auto / Hide-Show Rolls are now dark round buttons with a coloured ring and a glowing icon, at ±189.
    - The mock-up's bar sat 40 px right of centre; the game keeps it centred.
  - **Cursor:**
    - The invisible button that followed the mouse over enemies is gone. It swallowed the right-mouse camera drag, so the cursor "stuck" on enemies.
    - Hovering enemies and goblins now sets `UserInputService.MouseIcon = "rbxasset://textures/Cursors/KeyboardMouse/ArrowCursor.png"`. That is Roblox's own button-hover hand, so it looks exactly the same as over UI.
    - There is no hover while the right mouse button is held.
    - The old custom pixel hand (`rbxassetid://110909264872156`) is no longer used.
  - **Enemy circle:**
    - It is centred between the feet. The mesh box also holds the antennae / snout, so its centre sat too far forward.
    - It is now one dashed red ring that keeps turning (beetle 26°/s, boar 22°/s, golem 16°/s) with a small red glow in the middle, like the goblin aura.
    - Diameter per unit of model scale: beetle 3.22 (≈ 10.4 studs), boar 3.16 (≈ 11.6), golem 3.83 (≈ 14.5).
  - **Goblins no longer walk over enemies:**
    - The ground raycast ignores every model with `EnemyId`. They used to climb onto an enemy in their way.
    - While fighting, a straight path through the enemy's hitbox (+1.6 studs margin) becomes a detour round an ellipse just outside it.
    - There is no player-speed lead while fighting.
  - **Knockback when an enemy hits a goblin** (`GoblinAttacks.Hurt`): the goblin hops away from the hit and then walks back.
    - Hop: 1.6 + 0.9 × strength studs over 0.34 s, up to 0.55 + 0.35 × strength high. It is moved by the renderer through `pet.knock`.
    - Effects:
      - a 0.28 s white Highlight flash over the goblin
      - an impact flash and sparks on its body
      - dust at its feet
      - dizzy stars over its head
  - **Icon effects much more visible** (`GoblinIconFX`):
    - Everything is ×1.6, particles a further ×1.3, opacity ×1.45 and emission ×1.6.
    - Particles are at least 9 px on screen, and persistent pieces (planets, clocks, orbiting stars) at least 11 px.
    - Light particles get a soft dark shadow so they read on the white cells.
    - Sleepy's cloud stays smaller (×0.8).
  - The equipped row's odds are now compact "1/64" / "1/?", the same as the world tags (`GoblinPets.RatioText`). They auto-fit right of the feet, so nothing overlaps.
- Where:
  - Studio only:
    - `StarterPlayerScripts.GoblinPetClient` (left HUD, Ascend panel, rarity tags), backup `ServerStorage.UI_Backup.GoblinPetClient_before_mockup_hud`.
    - `StarterGui.RollGui.RollClient` (bottom bar), backup `RollClient_before_mockup_hud`.
    - `ReplicatedStorage.GoblinPetRenderer` (cursor, enemies excluded from ground rays, detour, knock hop), backup `GoblinPetRenderer_before_detour`.
  - Local = Studio (hash-checked): `Combat/GoblinCombatClient.client.luau` (6728), `Combat/EnemyAnimator.client.luau` (38190), `Combat/GoblinAttacks.luau` (44439), `Goblin_Pets/GoblinIconFX.luau` (43114).
  - `HUD_Mockup/`:
    - `mockup_reference.png`, the cut icons, `dock_atlas.png`
    - scripts: `ui_ref_crops.py` → `ui_cut.py` (+ `ui_cut_params.json`) → `ui_atlas.py`, and `ui_compare.py` for side-by-side checks
    - run `ui_ref_crops.py` on the PNG first; it rebuilds `uiref/ref.npy`.
  - `Fernwood_Mobs/rig_pivots.py`: the pivot measurement used by EnemyAnimator's PIVOTS.
- Test (playtest, 18 test goblins, the best 6 equipped):
  - side-by-side screenshots against the mock-up (left HUD and bottom bar)
  - hover on Shop and click on Ascend (panel opens)
  - hovering the beetle gives MouseIcon = ArrowCursor + outline
  - clicking the boar sent all goblins to it (server got the Target event)
  - re-targeting the golem: 0 goblins higher than standing height (no climbing), 2 of 546 samples just touched the hitbox edge (0.16 studs), so the margin was raised from 1.0 to 1.6
  - golem attacks: all 6 goblins flashed white and hopped (1.0–2.1 studs up)
  - new beetle circle checked from the side; inventory effects checked at normal size
  - console clean.
- Still open:
  - The right-mouse camera drag over an enemy cannot be simulated by the test tools. The cause (the invisible button) is gone; please check it once.
  - Ascend contents.
  - The open-eye "Hide Rolls" icon is the game's own; the mock-up only showed "Show Rolls".
  - Spawn chances + locations.

## 2026-10-06 — Enemy polish round 2: threat circles, no circling, rig pivots, click-to-target, glows, HP colours, inventory rarity tags, icon effects (Claude)
- What:
  - Red threat circle under each enemy (replaced on the same day by the turning ring above).
  - **No circling:** while the player walks round a fight, the goblins and the enemy stay put. The ring of spots is turned once, when the fight starts (`f.base` on the server, `group.combatBase` in the renderer).
  - **Floating parts fixed with measured joint pivots** (`EnemyAnimator` PIVOTS: a rotation turns round the real joint):
    - the boar's back legs no longer open a gap;
    - the boar's tail only tilts up and down at the rump;
    - the beetle's antennae stay in its head with a light left/right sway.
  - **Click to target:** clicking an enemy sends all goblins to it.
    - `CombatRemotes.Target`, `CLICK_RANGE` 55 (distance minus body size).
    - Hovering outlines the enemy (the cursor was reworked the same day, see above).
    - Goblin hover/click is off while they fight.
  - **Glows:**
    - the boar's back crystals glow and glint, with their own light;
    - the golem's yellow core gem glows, with a glint and a light (Core Nova starts there).
  - Enemy heads now and then look somewhere else, lightly.
  - **HP bar colours:** green, orange below 50 %, red below 15 % (smooth blend in `EnemyNametagFX`).
  - **Elder Root HP bar:** the pointer now hangs centred under the middle gem (plate shifted, pointer and aura X adjusted).
  - **Equipped goblins in the inventory:**
    - the rarity name in its colour sits under the goblin;
    - the odds sit bottom right;
    - both have a moving light spot and move with the hover.
  - **New `ReplicatedStorage.GoblinIconFX`:** small 2D versions of each goblin's world effect on its icons (inventory grid, equipped row, index, roll reveal).
    - All 42 goblins have one, e.g. Sleepy's dream cloud with Z's, Pyro's flames, Hoarder's coins, Galaxy Walker's planets, Eternal's clocks.
    - Assets: flame `rbxassetid://106924903263772`, dashed ring `rbxassetid://98455048052707`.
- Where:
  - `Combat/` = Studio: CombatConfig, CombatServer, EnemyAnimator, GoblinCombatClient.
  - `Fernwood_Mobs/ui/EnemyNametagFX.client.luau`.
  - `Goblin_Pets/GoblinIconFX.luau`.
  - Studio only: GoblinPetRenderer, GoblinPetFX, GoblinPetClient, RollClient.
- Test: playtests of every point; enemy models are `ModelStreamingMode = Atomic`, and setup retries cover streaming.
- Still open: see the entry above.

## 2026-10-05 — Combat polish: enemies fight goblins, less glow, 42 motions, enemies 1.1× smaller (Claude)
- What (user feedback):
  - **Too much glow** when attacking enemies:
    - CombatFX lights are dimmed (`LIGHT_SCALE` 0.4, range ×0.75) with at most 4 at the same time;
    - flash cores and halos are see-through and smaller;
    - sparkles are less bright;
    - the enemy hit flash is softer (Highlight fill 0.72 instead of 0.35).
  - **Enemies attack the goblins, not the player:**
    - enemies ignore players and stroll around home;
    - when goblins engage, the enemy stands its ground, turns to one goblin and attacks it;
    - the attack effect lands on that goblin, which flinches and is knocked back;
    - boss attacks hit every goblin near the impact.
  - **Not every goblin the same "two hands forward" move:**
    - the shared `cast` motion is gone, and each of the 42 goblins now has its own motion (42 different ones);
    - examples: Leaf left-right combo, Brawler hook, Alchemist underhand lob, Hoarder coin flick, Knight lunge, Bandit X-slash, Pyro palm push, Cyborg visor, Necromancer staff swirl, Dragon Rider command point, Demigod levitate, Crystal Lord ground punch, Stormbringer finger snap, Zeus javelin, Eternal pocket watch, Nebula pirouette, Galaxy Walker bowling (the planet rolls along the ground), Supernova arms burst, Eclipse sun + moon, Starlight star sweep, Void Watcher chest push.
  - **All enemies 1.1× smaller** (templates + previews; feet on the ground; HP bar offset recomputed).
- Where:
  - `Combat/` = Studio (hash-checked): CombatConfig (new `RingSpots`, `ENEMY_FIRST_ATTACK`, `WANDER_*`; aggro/leash/reach removed), CombatServer, CombatFX, GoblinAttacks, EnemyAnimator.
  - Studio only: GoblinPetRenderer and GoblinPetFX.
- Test (playtest):
  - without goblins, the beetle strolls and never attacks the player;
  - with goblins, enemies aim at a goblin (0° facing error), the hit goblin is knocked about 0.4 studs off its spot at the impact, and the hit flash stays at 0.75 or higher;
  - all 42 goblins fought the beetle, the boar and the boss (both boss attacks) with 0 errors;
  - an isolated harness confirmed 42 motions with no duplicates.
  - Not checked visually: screen captures are black while Studio is in the background.
- Still open: spawn chances + locations, coins for kills, real stats, enemy damage / goblin HP.

## 2026-10-05 — Combat: goblins fight the Zone 1 enemies (Claude)
- **What the user asked:**
  - equipped goblins attack nearby enemies automatically, each with its own attack animation and effect (Basic/Leaf: a normal punch);
  - all goblins strike at the same time, based on attack speed (placeholder: speed 1, damage 1);
  - enemies fight back with attacks, a walk animation and more animations, but deal no damage; the boss is the coolest;
  - effects must be very pretty and easy to see; damage numbers in the game theme with Montserrat (not too pixel);
  - spawn rate and spawn locations are explicitly NOT part of this.
- **Resize first:** beetle 1.5× smaller, boar and golem 1.2× smaller (templates + previews, feet kept on the ground; bar widths 8 / 9.5 / 13.5).
- **New scripts (sources in `Combat/`, installed and hash-checked):** CombatConfig, CombatServer, CombatFX, GoblinAttacks, GoblinCombatClient, EnemyAnimator. Section 1 ("Enemies and combat") describes what each one does.
- **Goblin attacks (all 42):**
  - Angry kick + steam puff, Sleepy eye-rub tap with Z's, Pebble stone throw, Shy hesitant tap with hearts, Muddy shuffle + mud splash, Scout knife stab;
  - Mushroom hat-tap spore, Tinkerer screwdriver spark, Bramble twig swing, Crystal "pling" tap, Alchemist droplet, Hoarder coin throw, Brawler right hook, Mystic light orb;
  - Knight sword thrust, Pyro flame, Bandit double dagger slash, Shaman green ground circle, Chef ladle swing, Engineer jetpack dash, King sceptre gold flash, Cyborg red eye beam;
  - Necromancer purple orb, Assassin sidestep stab, Dragon Rider fire puffs, Demigod gold light pillar, Shadow Stalker shadow lunge, Volcanic stomp + lava cracks, Crystal Lord crystal spikes under the enemy;
  - Stormbringer finger sparks + bolt, Seraph wing light pulse, Zeus lightning strike, Eternal clock rings, Phoenix flutter + fire burst, Nebula purple star, Galaxy Walker mini planet, Supernova big flash;
  - Eclipse sun + moon shadow slash, Starlight handful of stars, Void Watcher chest-eye dark beam.
- **Enemy animations:**
  - beetle: tripod leg walk, antennae, mandibles, blink, a lunging bite with green double slash;
  - boar: trot, tail wag, ear flicks, sniffing, a hoof scrape → crystal flare → charge with crystal spikes;
  - golem (boss): heavy steps with dust rings and camera shake, a breathing core light; it alternates **Root Slam** (fists charge, ground slam, root spikes, cracks, leaf storm) and **Core Nova** (energy gathers into the core, then a light pillar, rings and a leaf storm);
  - all: hit flinch + white flash, death animation + fade + coin/leaf burst, respawn rise.
- **Problems found in the playtest and fixed:**
  - StreamingEnabled: an enemy model can arrive before its parts → EnemyAnimator and EnemyNametagFX now retry. Enemy models are set to `ModelStreamingMode = Atomic`.
  - An enemy could stay stuck in "Walk" exactly at reach (floating point) → it now has a tolerance, plus 1.5 studs of hysteresis while fighting.
  - Enemies overlapped each other → they now separate.
  - Effect errors were swallowed silently → they are now reported once with `[CombatFX]`.
  - Not combat, but found in the test: the nametag signatures of Pyro and Assassin (variant 3) failed. In `GoblinTagFX`, a line's `w` (thickness) hid the `:w()` wait method; it is now stored as `width`. All 195 signature builds now pass.
- **Verified in Studio playtests:**
  - enemies walk, attack, flash, die, fade and respawn;
  - goblins stand around the enemy facing it, and HP drops 6 per second with 6 goblins;
  - all 42 goblin attacks run in game and in an isolated harness with 0 errors;
  - both boss attacks and the boss death were triggered;
  - no warnings from the combat scripts.
- **Not verified visually:** screen captures stayed black while Studio was in the background, so the user should look at the effects in a playtest.
- `Goblin_Pets/` local copies of GoblinPetRenderer/GoblinPetFX do NOT have the combat patches; Studio is the source.

## 2026-10-05 — HP bars: nicer HP text, stone pointers, tier animations (Claude)
- HP text is now white with a mint gradient and a dark green outline (no longer red).
- **Pointer** under each bar, in the bar's stone style (`Frame.Pointer`; made by `ui/make_arrows.py`):
  - common: plain stone, rbxassetid://89101492814755
  - rare: + moss, rbxassetid://71784548368754
  - boss: + emerald, rbxassetid://131632039048927
- New `StarterPlayerScripts.EnemyNametagFX` (client, cosmetic, skips bars > 160 studs away). It animates per `Tier`:
  - **Common:** pointer bobs gently; a soft shine sweeps the bar every 6 s.
  - **Rare:** stronger bob; shine every 3.5 s; stars pulse one after another; mint shimmer on the name.
  - **Boss:**
    - the whole bar floats;
    - a green `Aura` glow breathes behind it;
    - the stars twinkle and rock;
    - the name colour flows gold → emerald → gold (`EnemyName.UIGradient.Offset`);
    - shine every 2.2 s.
- New pieces on the bars: `BarSlot.Sweep` (shine strip; BarSlot clips), `EnemyName.UIGradient` and `HPBar.Aura` (boss only).
- Verified in a playtest: the animations run and the console shows no errors.

## 2026-10-05 — HP bar layout v2: name + stars above, HP text in the bar (Claude)
- What: new layout after the user's sketch.
  - The name sits on the stone above the bar (white, Montserrat Bold, left).
  - Gold level stars sit to the right in that header: Common 1, Rare 2, Boss 3. The star is rbxassetid://134470179468851, made by `ui/make_star.py`.
  - The green bar is lower in the frame.
  - Inside the bar: "33 / 100 HP" in red with a thin black outline (Montserrat ExtraBold).
- New frame assets (bar slot now at x 143..595, y 97..133 of 640×200):
  - common rbxassetid://90310020205105
  - rare rbxassetid://131889871104957
  - boss rbxassetid://135835596217777
- Model attributes `MaxHP` / `HP` (placeholders): Beetle 100, Boar 600, Golem 10,000.
- New `ServerScriptService.EnemyHPBars`: hooks every enemy model in the Workspace (attribute `EnemyId`). Change `HP` and the fill tweens and the text updates.
- The preview beetle is set to 33 HP as a demo.

## 2026-10-05 — Enemies: hitbox + HP bars (Claude)
- What: each enemy template now has two additions. HP itself is not built yet; the bars are visual and full.
  - **Hitbox:** an invisible `Hitbox` Part (85 % of the mesh width/length, 95 % of the height). Anchored, CanCollide off, CanQuery/CanTouch on.
  - **HP bar:** a BillboardGui `Hitbox.HPBar` in one of three styles, after the user's reference bars. Each has an enemy icon on the left and the name inside the bar (Montserrat Bold).
- Bar styles:

  | Style | Enemy | Look | Frame asset |
  |---|---|---|---|
  | Common | Moss Beetle | plain stone | rbxassetid://114730189800420 |
  | Rare | Crystalback Boar | stone + carved corners + moss | rbxassetid://92112935230719 |
  | Boss | Elder Root Golem | stone crowned with emeralds | rbxassetid://140197987898879 |

- Bar structure: `Frame` (image) → `BarSlot` → `Fill` (green gradient + `Shine`; set `Fill.Size` X to HP/MaxHP) + `EnemyName`, plus `Icon`.
- Icons (rendered from the models):
  - beetle rbxassetid://130598308297600
  - boar rbxassetid://133294913102136
  - golem rbxassetid://101113341979605
- Attributes on the models: `EnemyName`, `Tier` (Common/Rare/Boss), `EnemyId`.
- Sources:
  - `Fernwood_Mobs/ui/make_hp_frames.py` (frames, 640×200; slot layout is in the header).
  - `Fernwood_Mobs/ui/render_icons.py` (icons).
  - PNGs in `Fernwood_Mobs/ui/`.
- Where: `ReplicatedStorage.EnemyModels.*` and the preview copies in `Workspace.EnemyPreview`.
- Still open: HP values, damage, combat system, spawning, animations.

## 2026-10-05 — Enemies in Roblox Studio (Claude)
- What: the user imported the 3 FBX files with the 3D Importer (MCP cannot import FBX). Claude then set them up:
  - Skinned MeshPart + bones + AnimationController + texture all came through (Moss Beetle 35 bones, Boar and Golem 23 each).
  - Scaled to the goblin ratio: 1 Blender unit = 3.03 studs (Basic goblin is 4.4 studs tall).
    - Moss Beetle: 8.3 × 5.3 × 10.1 studs.
    - Crystalback Boar: 5.2 × 8.7 × 11.8 studs.
    - Elder Root Golem: 15.1 × 18.9 × 6.6 studs.
  - Templates are upright (mesh Y up, -Z = front), anchored, CanCollide off, with attribute `EnemyId`.
  - Glow: PointLight `Glow` in the eye bones, plus on `Crystals` (Boar) and `Core` (Golem).
- Where:
  - Templates: `ReplicatedStorage.EnemyModels.Enemy_MossBeetle / Enemy_CrystalbackBoar / Enemy_ElderRootGolem`.
  - Test copies near the spawn: `Workspace.EnemyPreview`. Delete these later.
  - Note: the model pivot is rotated relative to the mesh. Place enemies via the PrimaryPart (`part CFrame * part:ToObjectSpace(pivot)`), not via a plain PivotTo with lookAt.
- Still open: combat system (spawning with 3 % / 0.1 % chances, HP, AI, damage, coin drops) and animations (Roblox AnimationController + KeyframeSequences).

## 2026-10-05 — Enemies exported to FBX with palette texture (Claude)
- What: `Fernwood_Mobs/export_enemies_fbx.py` exports each enemy as a Roblox-ready FBX. It does not change the source .blend.
  - Every material colour becomes an 8 px cell in a 64×64 palette PNG, and each face gets its UV in its colour's cell.
  - The mesh gets ONE material (texture set to Closest).
  - Mesh + armature are exported with skinning (FBX_SCALE_ALL, -Z forward / Y up, no leaf bones, no animation, texture embedded).
  - Glowing parts are baked a little brighter, because there is no emission in the texture.
- Result:
  - Moss Beetle: 6,020 triangles, 35 bones, 15 colours.
  - Crystalback Boar: 3,058 triangles, 24 bones, 22 colours.
  - Elder Root Golem: 6,948 triangles, 23 bones, 16 colours.
  - Checked by re-importing the FBX into an empty Blender: texture, UVs and bones all come back, and a test pose deforms correctly (`exports/_reimport_check.png`).
- Where: `Fernwood_Mobs/exports/Enemy_*.fbx` + `Enemy_*_Palette.png`.
  - Run again after model changes: `blender -b "C:\Users\qmarc\Downloads\Enemy Models copy 2.blend" --python Fernwood_Mobs/export_enemies_fbx.py`
- Still open: import into Roblox (3D Importer) and scale it the same as the goblins; glow (Neon/PointLight) is to be added in Roblox; animations.

## 2026-10-05 — Enemies: full animation rigs (Claude)
- User request: almost everything must be animatable. When the user later says "animate X", Claude must be able to do it.
- Bones per enemy (every part of the mesh is bound to exactly one bone; `Root` is the only bone without geometry; 0 vertices without a bone):
  - **Moss Beetle (35):**
    - Root, Body, Head, Jaw, Sprout
    - Eye_L/R, Brow_L/R
    - Antenna_L/R_1 and _2
    - Mandible_L/R
    - Shell_L/R (the shell halves, so it can open or shake)
    - per leg (6 legs): Upper, Lower, Foot
  - **Crystalback Boar (24):**
    - Root, Body, Head, Jaw (lower jaw + small tusks), Snout (nose + nostrils)
    - Eye_L/R, Brow_L/R, Ear_L/R, Tusk_L/R
    - Crystals (the whole crystal cluster), Tail, Tail_Tip
    - per leg: Upper, Lower
  - **Elder Root Golem (23):**
    - Root, Hips, Chest, Head, Brow, Jaw (mouth + moss beard)
    - Eye_L/R, Antler_L/R, Core (glowing crystal + shards; the core light hangs on this bone)
    - per arm: UpperArm, Forearm, Fist
    - per leg: Thigh, Shin, Foot (root toes)
- Where: `C:\Users\qmarc\Downloads\Enemy Models copy 2.blend`, built by `Fernwood_Mobs/build_enemy_models_2.py`.
  - Backups before this change: `Fernwood_Mobs/backups/Enemy Models copy 2_before_fullrig.blend` and `..._before_tailbone.blend`.
  - `Fernwood_Mobs/add_boar_tail_bone.py` was a one-off edit; the builder now creates Tail_Tip itself.
- Still open: the animations themselves (idle, walk, attack, hit, death; eye blinks, ear flicks and so on).

## 2026-10-05 — Moss Beetle legs: segmented and curved (Claude)
- What: the beetle legs now follow the reference close-up. Each leg is made of separate chunky blocks along a curved path (hip → knee → ankle → foot):
  - a brown femur with a big moss cap on top, sticking out from under the shell;
  - a dark knee joint, then a brown shin;
  - a dark ankle joint, then a dark foot with two split toes.
  - New helper: `chunk()`, an oriented block between two points.
  - Bones per beetle leg: `Upper` (femur + moss), `Lower` (knee + shin) and `Foot` (ankle + foot + toes). The beetle now has 23 bones in total.
- Where: `C:\Users\qmarc\Downloads\Enemy Models copy 2.blend`; builder `Fernwood_Mobs/build_enemy_models_2.py`. 0 floating parts.

## 2026-10-05 — Enemies: working file moved, legs fixed, nothing floats (Claude)
- **Working file is now `C:\Users\qmarc\Downloads\Enemy Models copy 2.blend`** (user's choice; it is outside the Blend folder).
  - `Blend/Enemy Models 2.blend` holds an older version.
  - Backup of copy 2 before this change: `Fernwood_Mobs/backups/`.
  - Build: `blender -b "C:\Users\qmarc\Downloads\Enemy Models copy 2.blend" --python Fernwood_Mobs/build_enemy_models_2.py`
- What:
  - **Moss Beetle**, narrower:
    - Shell width ~1.75; total ~2.3 wide including the legs (was ~3.0).
    - Legs follow the reference: a thick femur from under the shell to a high knee with a moss cap, then a tapering shin and a dark pointed foot.
    - Front legs point forward and back legs point backward (`BEETLE_LEGS`, `beetle_leg_points`).
  - **Crystalback Boar** legs now bend:
    - The front legs have a small forward knee.
    - The hind legs angle back to a hock, then forward to the hoof, like the reference.
    - 2 bones per leg (`Leg_FL_Upper/_Lower` …).
  - Less detail: the boar's scars are gone, and there are fewer bristles, tufts and shards; the beetle has fewer panels, rim stones and drips; the golem has fewer moss drips.
  - **Floating-parts check:** the builder tests every piece against all the others (BVH overlap). Any loose piece is pushed onto the nearest surface and checked again. The result is 0 loose pieces for all three enemies.
  - Armatures are hidden and drawn as STICK. The "grey sticks" around the boar in the viewport were the bones. Unhide `Enemy_*_Rig` to see the rig.
- Renders: `Fernwood_Mobs/renders/`, including `07_boar_side` and `08_beetle_front`.
- Still open: user feedback; animations; Roblox export.

## 2026-10-05 — Zone 1 enemies: less blocky (Claude)
- What: shapes made closer to the concept art, while keeping the same low-poly style.
  - Builder `box()` now softens big blocks: a deep 2-segment chamfer, a sphere cast (0.45) and jitter, so they read as faceted rock or wood. Small details stay crisp (`soft=False` forces that).
  - New helpers: `limb()` (tapered round limb between two points) and `arc()` (curved root band).
  - Beetle: rounded head, round tapered legs with knee balls and rounded feet.
  - Boar: wedge-shaped head with a forehead, round pig snout with a rim and nostrils, a two-part curved tusk, round legs, round wooden guards with a rope ring, rounded hooves.
  - Golem: round foot, tapered root toes, curved roots around the core (ribcage-like), draped moss on the shoulders, log forearm, faceted fist.
- Where: `Enemy Models 2.blend`, built by `Fernwood_Mobs/build_enemy_models_2.py`; renders in `Fernwood_Mobs/renders/`.
- Still open: user feedback; animations; export to Roblox.

## 2026-10-05 — Zone 1 enemies: detail pass (Claude)
- What: much more detail on all three enemies in `Enemy Models 2.blend`.
  - Eyes, all three: layered (socket, glowing iris, coloured core, slit pupil, small highlight), plus heavier brows.
  - Moss Beetle: shell panels, rim stones, moss drips, sprouts, connected antennae, cheek/forehead plates, teeth, serrated mandibles, knee joints, leg spikes and claws.
  - Crystalback Boar: bristle mane, flank tufts, scars, lighter belly, brow ridge, lower jaw, snout detail, double tusks, inner ears, small crystal shards, guard slats, rope band, framed gem, split hooves, tail tuft.
  - Elder Root Golem: radial root toes, bark grooves and knots, glowing cracks around the core, extra core shards, vines with leaves, moss beard and drips, shoulder stones and mushrooms, knuckles, leaves on the antlers.
  - Polygons: Beetle ~3.2k faces, Boar ~2.4k, Golem ~3.9k. Bones are unchanged.
- Where: builder `Fernwood_Mobs/build_enemy_models_2.py`; renders, including face close-ups 04–06, in `Fernwood_Mobs/renders/`.
- Still open: animations, export to Roblox, combat system.

## 2026-10-05 — Zone 1 enemy models in "Enemy Models 2.blend" (Claude)
- What: the three Fernwood enemies, built after the user's concept art. They replace the earlier Mossling/Glowcap/Elder Stump concepts.
  - **Moss Beetle**, common, variant A: moss carapace in two halves, rock body, 6 chunky legs, mandibles, orange eyes.
  - **Crystalback Boar**, rare (3 % spawn), variant B: grey-brown hide, a cluster of chunky green crystals on the back, wooden shin guards with a crystal gem, tusks, green eyes.
  - **Elder Root Golem**, boss (0.1 % spawn), variant A: log/root body, mossy shoulders, an organic chest cavity with a glowing amber crystal core, antler branches, root feet.
- Model setup: each enemy is ONE joined low-poly mesh (pieces overlap, nothing floats) plus its own armature.
  - Rigid vertex groups per body part: Beetle 17 bones, Boar 8, Golem 12.
  - Front = -Y; same unit scale as the goblins (Basic is ~1.45 tall).
  - Sizes: Beetle ~1.5 tall, Boar ~2.9 tall including crystals, Golem ~6.2 tall.
  - Flat colour materials (`Enemy_*`); glowing parts use low emission, which becomes Neon in Roblox.
- Where:
  - `Enemy Models 2.blend`, one collection per enemy.
  - `Scale_Reference_BasicGoblin` and `Preview_Stage (not for export)` are helper collections and are not to be exported.
  - Builder: `Fernwood_Mobs/build_enemy_models_2.py`, which rebuilds everything; run it on the file in background.
  - Renders: `Fernwood_Mobs/renders/`.
  - Backup of the empty start file: `Fernwood_Mobs/backups/`.
- Still open: user feedback on the shapes; animations (idle/walk/attack/hit/death); FBX export and Roblox import; the enemy and combat system in Studio.

## 2026-10-05 — Combat replaces mining as the main mechanic + Zone 1 enemy concepts (Claude)
- Decision (user):
  - Rolled goblins fight enemies instead of mining. Each area has its own enemies, and defeating them gives coins, the main currency.
  - Rarer currencies come later, including a rebirth currency that raises the luck multiplier.
  - Attacks: a shared punch/headbutt for every goblin, plus a signature move per design (proposed, not built yet).
- What: blocky concept models for Zone 1 Fernwood, in the goblin style:
  - Mossling: common enemy.
  - Glowcap: rare, 3 % spawn chance.
  - Elder Stump: boss, 0.1 % spawn chance.
- Where:
  - Builder: `Fernwood_Mobs/build_concepts.py`.
  - Renders: `Fernwood_Mobs/concepts/`.
  - Blend file: `Fernwood_Mobs/fernwood_mobs_concepts.blend` (not rigged).
- Still open: user feedback on the designs; rig and animations; Roblox enemy system (spawning, HP, AI, combat, drops). Nothing is in Studio yet.

## 2026-10-05 — Full game overview written (Claude)
- What: wrote section 1 above, checked against the open Studio place. Found that the mining and progression scripts from `Mining_Assets/HANDOFF.md` are missing from that place.
- Still open: find out where the mining work lives (another place copy?).

## 2026-10-05 — Inventory, HUD, Zone 2 unlock, forest effects (Claude)
- What:
  - Inventory: duplicates stack into one cell ("x5"); rarity backdrop (tint + pattern) in Inventory and Index cells; hover tooltip (name, rarity, odds, stats, owned).
  - Delete mode: click a goblin, then a slider sets how many copies to delete. Equipped goblins are never deleted.
  - Panels sit above the ROLL button; the dim also covers the Roblox top bar; clicking next to a panel does not close it (only X or the dock icon).
  - No panels open while rolling; toasts live in their own `GoblinToastGui` above the roll animation.
  - Luck clover ("1x") above ROLL (player attribute `Luck`), with an animation when luck increases; hidden while a panel is open.
  - Dock: 5 buttons (Inventory, Shop, Index, Settings, Teleport) with drifting icon backgrounds. The Teleport panel goes to the Fernwood spawn; Zone 2 shows LOCKED/SOON.
  - Zone 2 gate: the UNLOCK button (part with attributes `ZoneUnlock=2`, `ZoneCost=5000`) is clickable. The server spends the coins and saves `data.zones`; player attribute `Zones` = "2". The sign then shows UNLOCKED / OPENING SOON. The gate stays closed because Zone 2 is not built yet.
  - Forest effects: falling leaves from the canopies, swaying trees and shrubs, wind gusts, pollen, fireflies at the lanterns, spores at the toadstools, butterflies.
  - Rarities: during this work only Common was active; someone has since put all rarities back (see section 1).
- Where (Studio):
  - Scripts: `StarterPlayerScripts.GoblinPetClient` (menu/inventory/dock/teleport/tooltip), `StarterGui.RollGui.RollClient` (roll + clover), `ServerScriptService.GoblinInventory` (data: coins, luck, zones, `UnlockZone`, `AddCoins`, `SetLuck`), `ServerScriptService.ZoneServer` (gate click), `StarterPlayerScripts.ZoneGateClient` (gate feedback), `StarterPlayerScripts.ForestAmbience` (map effects), `ReplicatedStorage.GoblinUI` (theme, `UI.tiles`, image IDs).
  - Signs: `Workspace.SignTexts` (Montserrat SurfaceGuis). The old invisible collision box `BarrierBox_GateTemporary` was replaced by `Zone2Barrier`, so the UNLOCK button can be clicked.
  - Local copies: `Goblin_Pets/*.luau`. GoblinPetClient, RollClient and GoblinInventory were later changed in Studio by someone else, so the local copies are behind.
- Test (Studio, ServerStorage attributes):
  - `StudioGrantGoblins` = "Basic,Leaf" gives goblins.
  - `StudioSetLuck` = 2 sets luck.
  - `StudioGiveCoins` = 6000 gives coins.
  - DataStore saving only works after enabling "Enable Studio Access to API Services".
- Still open:
  - Build Zone 2 and open the gate after unlock.
  - Add a coin source (coins are only earned via the test hook).
  - The coin counter uses FredokaOne, which conflicts with the "Montserrat everywhere" rule; it was changed by someone else, so it was left as is.
  - Mobile is untested.
