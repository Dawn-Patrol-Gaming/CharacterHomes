# CharacterHomes

A standalone Elder Scrolls Online addon for managing and teleporting to your homes, per character and account-wide, with a built-in friend housing bar.

No addon dependencies. Uses only native ESO APIs and textures.

Players: see [`Instructions.txt`](Instructions.txt) for the in-game guide. It ships with the addon; this README doesn't.

---

## Overview

CharacterHomes gives you a single, searchable window to manage every home you want quick access to:

- Save homes **per character** (one primary plus unlimited named) or **account-wide**
- Save a **friend's house** (or a guild master's or officer's) to your list for one-click access later
- Set each of your own homes to teleport **inside or outside** by default
- **Search** across labels, official ESO house names, zones and owner @accounts
- **Teleport** to any saved home from the window or with a slash command

---

## Installation

1. Download and extract the `CharacterHomes` folder
2. Place it in `\Elder Scrolls Online\live\AddOns\CharacterHomes\`
3. Enable the addon in the ESO Addon Manager

**No library or addon dependencies required.**

---

## Slash Commands

`/home` and `/gohome` do the same thing, and so do `/sethome` and `/setmyhome`.

| Command | Description |
|---|---|
| `/homes` | Open the Character Homes window |
| `/home` (or `/gohome`) | Teleport to your character's primary home, inside or outside as set for it |
| `/home <query>` | Teleport to the saved home whose label or official house name matches (partial match); falls back to the names of houses you own |
| `/home inside [query]`, `/home outside [query]` | Same, but overrides the home's default for this trip (outside works for your own houses only) |
| `/sethome` (or `/setmyhome`) | Set the current house as your character's primary home |
| `/sethome <label>` | Save the current house as a named character home |
| `/setglobalhome <label>` | Save the current house as an account-wide home |
| `/resethome` | Clear your character's primary home |
| `/delhome <name>` | Remove an account-wide home by label (partial match; lists ambiguous matches) |
| `/delmyhome <name>` | Remove a named character home; use `primary` to remove the primary home |

Saving a house that's already in that list just changes its label, so you don't get duplicates. The primary home is a separate slot.

> **Command conflicts:** Essential Housing Tools also uses `/home` and `/sethome`. CharacterHomes claims those two on `EVENT_PLAYER_ACTIVATED`, and only if nothing else has them by then, so with EHT enabled they stay EHT's and you use `/gohome` and `/setmyhome`. Every other command is registered on load only if it's free; any that are taken are listed in one chat line after login.

---

## The window

Open with `/homes`. Close with Escape, the Close Dialog keybind, or the X.

### Top

- **Search:** filters the list as you type (labels, house names, zones, @owners)
- **Owner:** filters the account homes section: all, only yours, or only friends'. Remembered between sessions
- **Port to Friend's House** link, if that addon is loaded

### List

Two sections: MY HOMES (this character's primary and named homes) and ACCOUNT HOMES.

| Element | Description |
|---|---|
| **Teleport icon** | House icon for the primary home; wayshrine for the rest. Left-click teleports to the home's default and closes the window. Right-click (your own houses only) chooses Inside or Outside for that trip |
| **Owner** | @account for a friend's house; blank for your own |
| **Label** | Your name for the entry (white) |
| **Inside/Outside** | The home's default teleport destination (grey). Friends' houses are always Inside |
| **House name** | The official ESO house name (grey). Hover for the zone; click to open the map at the house |
| **Edit pencil** | Inline edit: label, plus house and Inside/Outside for your own houses. Save with ✓ or Enter, cancel with X or Escape |
| **X** | Removes the entry from your list (Cancel while editing) |

### Friend bar (upper)

| Control | Description |
|---|---|
| **Friend** | Your friends, plus guild masters and officers grouped by guild |
| **Type** | Where to save: Primary, Named or Account wide |
| **House** | Every ESO house |
| **Name** | Optional label |
| **✓** | Saves the house to your list |
| **Teleport** | Teleports to it now (inside) without saving |

### Owned bar (lower)

| Control | Description |
|---|---|
| **Zone** | Narrows the house list to one zone |
| **Type** | Where to save: Primary, Named or Account wide |
| **House** | Houses you own |
| **Name** | Optional label (defaults to the house name, or "Primary Residence" for a primary home) |
| **✓** | Saves to your list and clears the Name box |
| **Teleport** | Teleports there now; right-click for Inside or Outside |

---

## SavedVariables Structure

One account-wide table, split by megaserver (an ESOUI requirement). The server name is passed as the `ZO_SavedVars` profile:

```lua
ZO_SavedVars:NewAccountWide("CharacterHomesSavedVars", 3, nil, defaults, GetWorldName())
```

So the file looks like this (`nil` fields are simply absent):

```lua
CharacterHomesSavedVars = {
    ["NA Megaserver"] = {
        ["@YourAccount"] = {
            ["$AccountWide"] = {
                version     = 3,
                migrated    = true,       -- one-time re-key/zone migration has run
                ownerFilter = 0,          -- Owner dropdown: 0 all, 1 mine, 2 friends'
                account = {
                    -- your own house: key = official house name
                    ["Elinhir Private Arena"] = {
                        houseId     = 66,
                        displayName = "A Test Arena",
                        zone        = "Craglorn",
                        exterior    = true,      -- default teleport goes outside
                    },
                    -- friend's house: key = "@Owner_" .. official house name
                    ["@FriendName_Seaveil Spire"] = {
                        houseId     = 94,
                        owner       = "@FriendName",
                        displayName = "Guild Hall",
                        zone        = "Betnikh",
                        exterior    = false,     -- always false for friends' houses
                    },
                },
                characters = {
                    ["Character Name"] = {
                        primary = { houseId = 89, displayName = "Primary Residence", zone = "Blackwood", exterior = false },
                        named   = {
                            ["Elinhir Private Arena"] = {
                                houseId = 66, displayName = "My Arena", zone = "Craglorn", exterior = false,
                            },
                        },
                    },
                },
            },
        },
    },
    ["EU Megaserver"] = { ... },  -- same layout, separate data
}
```

**Key points:**
- Keys come from `makeEntryKey(houseId, owner)`: the official house name (`GetCollectibleName(GetCollectibleIdForHouse(houseId))`) for your own houses, `"@Owner_" .. houseName` for friends'. Renaming an entry only changes `displayName`, and saving the same house again overwrites the same key, so there's one entry per house (per owner) in each list.
- `owner` is the friend's `@account`; absent for your own houses.
- `zone` is the **geographic parent zone** (for example `"Artaeum"`, not `"Grand Psijic Villa"`).
- `exterior` is the default teleport destination. Entries saved before it existed are set to `false` (inside) on load.
- Character data is keyed by character name (`GetUnitName("player")`).
- Saves from before the server split (`["Default"]` profile) are moved to the current server's profile on first load.

---

## Compatibility

| Addon | Status |
|---|---|
| Essential Housing Tools | Compatible: EHT keeps `/home` and `/sethome` when enabled; use `/gohome` and `/setmyhome` |
| Port to Friend's House | Optional: a quick-open link appears in the window if the addon is loaded |

---

## For Developers / Maintenance Guide

### Files

| File | Ships | Responsibility |
|---|---|---|
| `CharacterHomes.txt` | yes | Manifest |
| `CharacterHomes.xml` | yes | Window, bars, and the virtual templates `CharacterHomesRow`, `CharacterHomesHeader`, `CharacterHomesNote` |
| `CharacterHomes.lua` | yes | Logic, combo boxes populated from game data, row pools, slash commands |
| `Instructions.txt` | yes | Player guide (keep it current when features change) |
| `README.md`, `docs/`, screenshots, `*.zip` | no | Repo only |

Release zips are built by hand from the installed addon folder (only the files that ship) and kept in the repo root as `CharacterHomes_vX.Y.Z.zip`.

**Rule:** Never position or size controls in Lua if they can be defined in XML. Lua only creates controls that depend on runtime data (the bars' combo boxes).

---

### Key ESO APIs used

```lua
GetCurrentZoneHouseId()                          -- are we inside a house?
GetCurrentHouseOwner()                           -- @account of house owner
GetCollectibleIdForHouse(houseId)                -- collectible linked to house
GetCollectibleName(collectibleId)                -- official ESO house name
GetCollectibleUnlockStateById(collectibleId)     -- COLLECTIBLE_UNLOCK_STATE_UNLOCKED_OWNED
GetHouseZoneId(houseId)                          -- house's own sub-zone id
GetParentZoneId(zoneId)                          -- geographic parent zone id
GetZoneNameById(zoneId)                          -- zone name string
JumpToSpecificHouse(ownerAccount, houseId)       -- teleport to a friend's house (inside only)
RequestJumpToHouse(houseId, exterior)            -- teleport to YOUR OWN house; exterior = true goes outside
CanJumpToHouseFromCurrentLocation()              -- guard before teleporting
GetNumFriends() / GetFriendInfo(i)               -- enumerate friends list
GetNumGuildMembers(guildId) / GetGuildMemberInfo -- guild masters and officers for the friend bar
ZO_ComboBox_ObjectFromContainer(ctrl)            -- get combo object from a control reference
```

> **Critical:** `RequestJumpToHouse` only works for houses you own. For any friend's house use `JumpToSpecificHouse(owner, houseId)`, which has no outside option. Passing an owner to `RequestJumpToHouse` throws a type-check error.

---

### Getting the geographic zone for a house

`GetHouseZoneId` returns the house's own sub-zone (for example `"Grand Psijic Villa"`), not the geographic region. Always walk up to the parent:

```lua
local function getHouseZone(houseId)
    local zoneId = GetHouseZoneId(houseId)
    if zoneId and zoneId > 0 then
        local parentId = GetParentZoneId(zoneId)
        return GetZoneNameById((parentId and parentId > 0) and parentId or zoneId)
    end
    return nil
end
```

---

### XML anchor patterns

**Label left-of-editbox** (RIGHT→LEFT trick, vertically centers the label automatically):
```xml
<Label name="$(parent)MyLabel">
    <Anchor point="RIGHT" relativePoint="LEFT" relativeTo="$(parent)MyEditBg" offsetX="-4" offsetY="0"/>
</Label>
```

**Invisible reference control** (used to Y-center Lua-created combo boxes in a bar):
```xml
<Backdrop name="$(parent)FHRefBg" centerColor="00000000" edgeColor="00000000">
    <Anchor point="BOTTOMLEFT" relativePoint="BOTTOMLEFT" relativeTo="$(parent)" offsetX="0" offsetY="-76"/>
    <Dimensions x="1" y="28"/>
</Backdrop>
```
Then in Lua:
```lua
comboCtrl:SetAnchor(LEFT, CharacterHomesDialogFHRefBg, LEFT, 78, 0)
```

---

### List rows (object pools)

Rows, section headers and notes are created from the XML templates through `ZO_ObjectPool` (`newPool` / `acquire`) and reused on every redraw. Released controls are hidden and unanchored, never destroyed (`Destroy` is unreliable in ESO). Per-row handlers that depend on the entry are reassigned each time a row is acquired, so a reused row never shows a previous entry's state.

`zo_callLater(populateDialog, 0)` defers rebuilds to the next frame so the UI is consistent.

---

### ZO_ComboBox and EditBox gotchas

- `ZO_ComboBox` sorts its items by default. Any combo that's selected by index (owner filter, house, Inside/Outside) calls `SetSortsItems(false)` first.
- ESO edit boxes fire `OnEnter` and `OnEscape` (not `OnEnterPressed`/`OnEscapePressed`).
- A plain `EditBox` only takes focus on click with `mouseEnabled="true"` and `<OnMouseDown>self:TakeFocus()</OnMouseDown>`, as in `ZO_DefaultEdit`.

---

### Forward declarations

A function called by another function defined earlier in the file must be forward-declared at the top:

```lua
local populateDialog   -- forward declare
local refreshIfOpen
local getOwnedHouses

-- ... later in the file ...
populateDialog = function()  -- NOT: local function populateDialog()
    ...
end
```

`getHouseZone` must be defined **before** `currentHouseInfo`: it is used as an upvalue and is not forward-declared.

---

### Tooltips

Interactive controls use the `addTooltip` helper:

```lua
local function addTooltip(ctrl, text)
    ctrl:SetHandler("OnMouseEnter", function(self)
        InitializeTooltip(InformationTooltip, self, BOTTOM, 0, -4)
        InformationTooltip:AddLine(text, "ZoFontGame", 1, 1, 1)
    end)
    ctrl:SetHandler("OnMouseExit", function()
        ClearTooltip(InformationTooltip)
    end)
end

addTooltip(myButton, "Description of what this does")
-- color codes work: "First line\n|c888888Muted hint|r"
```

When a tooltip's text changes while the mouse is already on the control (edit mode's ✓/X), `refreshHoverTooltip` re-runs its `OnMouseEnter`.

---

### SavedVars versioning

Current version: **3**. Bumping the version in `ZO_SavedVars:NewAccountWide` **wipes** the stored data for anyone on a lower version, so don't bump it for structure changes. Migrate in place in `onAddonLoaded` instead, guarded by a flag (like `migrated`) or by checking for a missing field (like `exterior`). New top-level keys can go in the defaults table; `ZO_SavedVars` fills in missing defaults on every load.

---

### House enumeration and caches

`ownedHouseCache` and `allHouseCache` are module-level locals populated once per session by scanning house IDs 1–1000. There is no ESO API to enumerate all houses directly. If a player buys a new house mid-session, the cache won't show it until `/reloadui`. This is intentional.

---

### Slash command conflicts

Commands are only registered if `SLASH_COMMANDS` doesn't already have them. When adding new ones, check against EHT, BeamMeUp and Port to Friend's House first.

---

### Debugging

Set `local CH_DEBUG = true` near the top of `CharacterHomes.lua` for verbose output from slash commands and list building. Output goes to chat via `d()`.

---

### Testing

In-game test steps and results are tracked in [`docs/TESTING.md`](docs/TESTING.md).

---

## Feedback and Contributions

Bug reports and suggestions welcome via GitHub Issues.
