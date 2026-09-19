<p align="center">
  <img src="assets\dmtools.png" alt="Dungeon Master Tools" width="560" />
</p>


site > [DM Tools](https://sethy.github.io/dmtoolsweb/)

Browser **all-in-one editor** for **Dungeon Master** and **Chaos Strikes Back** data files:

- `dungeon.dat` — maps, objects, Hall of Champions, wall/scroll texts
- `GRAPHICS.DAT` — creatures, items, spells, attacks, sprites, palettes
- `CSBGAME*.DAT` **/** `DMGAME.DAT` — save games (party, inventory, stats, and the dungeon frozen in the save)

Nothing is written back to the original file. Edits live in memory; **Download** rebuilds bytes.

Credits: programming Luciano Palladino; reference tools ADGE by kentaro.k-21, CSBwin/CSBuild by Paul Stevens, GraphReader by Benjamin Prieu.


| Workspace        | Open              | Typical files                                                |
| ---------------- | ----------------- | ------------------------------------------------------------ |
| **Dungeon.dat**  | Open dungeon.dat  | Uncompressed DM `dungeon.dat`; compressed CSB (`81 04`…)     |
| **Graphics.dat** | Open graphics.dat | Atari ST DM/CSB, plus PC / Amiga / Apple (auto-detected)     |
| **Saves**        | Open CSBGAME*.DAT | Scrambled saves (`CSBGAME`, `DMGAME`, CSB Extended Features) |


**Download** encodes the active workspace. Dungeon Download always **compacts** unreachable object records first. Click **cmp/raw** on the dungeon status line to wrap/unwrap CSB compression on Download.

Version (`v…` from `package.json`) and **About** sit on the **right of the status bar** (the grey line under the logo).

---



## App
<p align="center">
  <img src="screenshots\img1.png" width="560" />
</p>

Top bar: workspace buttons (Dungeon.dat / Graphics.dat / Saves) · Open · Download.

Second bar: file summary (endian, map count, party start, …) · version · **?** About.

Third bar: sections of the current workspace.

**Undo / redo** is keyboard-only: **Ctrl+Z** / **Ctrl+Y** (or **Ctrl+Shift+Z**). One stack **per section** (Maps undo does not pop Champions; Creatures undo does not pop Objects). Paint strokes on the map coalesce into one step; form fields usually do not.

---



## Dungeon.dat
<p align="center">
  <img src="screenshots\img2.png" width="560" />
</p>

Open a `dungeon.dat` (or CSB compressed dungeon) here.

### Maps (map editor)

Three columns: **levels / tools** (left) · **canvas** (center) · **Properties** (right).

The wood frame around the map turns **red** while **Draw** is on.

#### Layers

Toggle Map, Items, Sensors, Floor/Wall Decorations, Grid, Onion skin. Onion skin outlines the previous map (cyan = Up) and next (orange = Down) and marks neighbor stairs.

#### Create / Edit brushes

**Tiles · Creatures · Sensors · Items · Text.** Pick a graphic in the flyout (Text has no flyout — it places the last unused wall/scroll string from **Texts**). Placement also appends that monster type or decoration graphic to the map’s read-only TOC lists in Properties (max 15). The number in parentheses is a **live count** on this map, not the type id. Deleting the last use drops the row.

#### Draw · Single · Area


| Control                 | Role                                                                           |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Draw** (or **Space**) | Paint/place vs select/pan. Space also picks **Tiles** if no brush is selected. |
| **Single** (cross)      | One cell at a time                                                             |
| **Area** (square)       | With Draw + Tiles = rectangle fill. Without Draw = drag a region select        |


**Erase by type** (current brush) / **Erase all** ask for confirmation. Object records stay in the databases until **Compact** or Download.

**Copy / Paste** (Ctrl+C / Ctrl+V) stamp a tile or region (terrain + object chain) onto **existing** cells — they do not grow the map.

**New level** appends a 16×16 wall map (max 64). **Duplicate** clones tiles and objects. **Delete level** is blocked if only one map remains. **Move ▲/▼ / Move to** reorder maps and remap teleporter destinations; **stairs are not retargeted**. Party start always lives on map 0.

**Validate** lists unpaired stairs, out-of-range teleporters, orphan texts (click a row to jump). **Compact** drops unreachable DB records (not text slots).

#### Mouse on the canvas


| Input                             | Draw **off**                                                             | Draw **on**                                                                                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Left click**                    | Select tile or object → Properties. Click empty void → level properties. | **Tiles:** paint terrain type only (keeps flags + objects). **Creatures / sensors / items / text:** place. Click position on the cell sets wall face or floor corner.   |
| **Left drag**                     | Pan. With **Area**: rubber-band region select.                           | Paint/place along the stroke. **Tiles + Area**: rectangle fill. Painting into the void **grows** the map (max 32, new cells Wall). Objects still need an existing cell. |
| **Right click / right drag**      | (ignored)                                                                | **Tiles:** erase to Wall (Area = rectangle erase). **Creatures / sensors / items / text:** remove the matching object under the cursor.                                 |
| **Middle button**                 | Pan                                                                      | Pan                                                                                                                                                                     |
| **Wheel**                         | Zoom toward cursor                                                       | same                                                                                                                                                                    |
| **Ctrl + wheel** / trackpad pinch | Zoom                                                                     | same                                                                                                                                                                    |
| **Trackpad two-finger scroll**    | Pan                                                                      | same                                                                                                                                                                    |


There is **no double-click** on the map (double-click is used in **Saves → Champions** inventory).

**Esc** clears selection back to level properties.

Hover shows coordinates, stairs destination (same abs x/y on the adjacent depth), and object names. Level 0 paints a purple facing arrow on the party-start tile.

#### Properties (right)

- **Level:** NESW **N+/N− … W+/W−** pad (grow/shrink, new cells Wall, size 1–32), **Crop** to non-empty tiles, origin offsets, door types, random wall/floor decoration **counts** (first N of the TOC lists), experience, TOC lists (read-only).
- **Tile:** type (Wall / Floor / Pit / Stairs / Door / Teleporter / Trick wall), flags (random décor faces, pit, stairs dir, door, …). Floor or teleporter on **level 0**: **Set party start** + facing N/E/S/W.
- **Object:** monster type/count/facing, sensor type/action/target/graphic, item flags, teleporter dest, door ornate, wall text / scroll DB2 picker, trash (unlink) and **Clear all objects** on the tile.

Stairs have no dest fields: dest = same absolute `(x+offsetX, y+offsetY)` one depth up or down. Teleporters store dest map (array index) + x/y.

Portrait sensors (type 127) edit Hall of Champions fields from the facing floor mirror text.

### Champions

Hall of Champions **mirror texts** (name, title, stats as stored in dungeon text). Same fields as portrait sensors on the map. Edits stay in memory until Download.

### Texts

Pool of **wall / scroll** strings (not champion portraits). Create, edit (`/` = line break), delete. Delete unlinks wall placements and retargets scrolls; indices are **not** compacted. A wall DB2 can sit on only one tile chain.

---



## Graphics.dat
<p align="center">
  <img src="screenshots\img3.png" width="560" />
</p>

Open a `GRAPHICS.DAT`. Platform is fingerprinted on load.


| Section                               | What you can do                                                                               |
| ------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Creatures**                         | Item 559 stats, flags, names (556). Undo is this section only.                                |
| **Objects**                           | Item 559 item stats + names. Every field change is an undo step.                              |
| **Assets**                            | Browse images / sounds / text; TXT1 names are editable. UI / Decor / Palettes stay read-only. |
| **Spells / Combos / Attacks / Runes** | Item 560. Attack names share a **300-byte** pool. Own undo stack per tab.                     |
| **UI**                                | Item 561 hitboxes (read).                                                                     |
| **Decor**                             | Item 558 wall/floor/door décor tables (read).                                                 |
| **Palettes**                          | Item 562 palettes / brightness (read).                                                        |


Download rebuilds the item table (LZW where the original used it).

---



## Saves
<p align="center">
  <img src="screenshots\img4.png" width="560" />
</p>

Open a scrambled save. **Download platform** on Champions can differ from the detected source (Atari / Amiga / CSB / PC layout).

### Maps

Same map editor as Dungeon.dat, bound to the **dungeon embedded in the save**. Leaving the tab writes that dungeon back into the save in memory.

### Champions

Party (count is read-only) plus one character at a time: paper-doll slots, backpack, chest, HP/food/water, attributes, skills.

#### Inventory mouse


| Input                 | Slot / backpack / chest                                                                                   |
| --------------------- | --------------------------------------------------------------------------------------------------------- |
| **Left click**        | Select the slot. If you are already holding an item, try to **place / swap** it here (engine carry mask). |
| **Double-click**      | **Pick up** the item (hold it).                                                                           |
| **Right click**       | **Empty** the slot (drop out of inventory into the dungeon DBs, unplaced).                                |
| **Drag from catalog** | Drop a new item from the item list (if the slot allows it).                                               |


Click a chest in the doll to open its 8 contents. Item type id is not editable (the catalog item is the type).

---

