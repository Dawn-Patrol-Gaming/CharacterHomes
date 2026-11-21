# CharacterHomes in-game tests

Note anything unexpected along with the exact chat output. A Lua error popup at any point is a
failure. Copy its first few lines.

Every test still to do is under **To do**, grouped by when you can run it. Everything after that
is history: what changed, what passed, and notes worth keeping. Passed tests are removed from
To do.

## To do

**Before starting: log out fully and back in** (not just `/reloadui`). The manifest changed
(`AddOnVersion`, `APIVersion`, licence lines), and the game only reads it at login.

### Right after logging in

| # | Do | Expect |
|---|---|---|
| LD1 | Add-ons list at the character screen | Character Homes listed and enabled, with no "out of date" warning |
| LD2 | Log in and watch chat | No Lua error and no Character Homes chat line. (A line only appears if another addon already uses one of our commands, see CW1) |
| SC1 | `/setglobalhome` with no name | `Usage: /setglobalhome <name>  (account-wide alias)` and a second line about `/setmyhome` |
| SC2 | `/setmyhome` while not in a house | `You must be inside a house.` |
| SC3 | `/delhome` with no name | `Usage: /delhome <name>` |
| SC4 | `/delmyhome` with no name | `Usage: /delmyhome <name>  (or 'primary')` |
| SC5 | `/homes` | The window opens with every home you had before |

### In the homes window (`/homes`)

| # | Do | Expect |
|---|---|---|
| LS1 | Look over the list | Columns line up on every row: icon, @owner (friends' homes only), label, house name, then the edit and delete buttons at the right edge |
| LS2 | Look at the icons | Your primary residence has the home icon; every other row has the wayshrine |
| LS3 | Look at the colours | Section headers gold, @owner blue, label white, house name grey |
| LS4 | Hover the teleport icon on one of your own homes, then on a friend's | Own: `Teleport to <label>` and `Right-click for outside`. Friend's: `Teleport to <label>` and `via @owner` |
| LS5 | Hover a house name | It turns white, with a tooltip of the zone and `Click to show on map`. Moving off turns it grey again |
| LS6 | Click a house name | The map opens on that house's zone, centred on the house if it has a map marker |
| LS7 | Search for `zzzz` | Grey notes: `No character homes match the filter.` and `No account homes match the filter.` |
| LS8 | Clear the search and scroll to the bottom | The last row is fully visible, with no large blank area under it |
| PL1 | Type a long search one letter at a time, then delete it letter by letter | The list updates on every keystroke, with no leftover, doubled or blank rows |
| PL2 | Owner filter: All account homes, then My homes, then Friends' homes | Account homes shows all, only yours (no @owner), then only friends' |
| PL3 | Owner filter on Friends' homes, `/reloadui`, `/homes` | Still on Friends' homes. Set it back to All account homes |
| PL4 | Close and reopen the window five times | Same list each time, nothing doubled |
| PL5 | After PL1–PL4, run `/script local n=0 while _G["CharacterHomesRow"..(n+1)] do n=n+1 end d(n)`. Type 20 more letters in the search and clear it, then run it again | Both numbers are the same, and no more than the most rows you've had on screen at once. A number that grows means rows aren't being reused |
| PL6 | Owned bar: Account wide, any house, Name `Test`, ✓. Hover the new `Test` row and click its X without moving the mouse | `Test` is removed, and the row that moves under the mouse isn't left highlighted |
| PL7 | Click the search box, then hover a house name so it turns white, and type a letter without moving the mouse | After the list redraws, no house name is stuck white and no tooltip is left showing |
| ED1 | Edit button on one of your own homes | Label editor with the current label, a house dropdown on the right, and the buttons change to ✓ and X |
| ED2 | Edit button on a friend's home | Label editor only, no house dropdown |
| ED3 | Edit a label, change it, press Enter | Saved, and the row shows the new label |
| ED4 | Edit a label, change it, click ✓ | Saved |
| ED5 | Edit a label, change it, press Escape | Label unchanged. Say whether the whole window closes too |
| ED6 | Edit a label, change it, click X | Label unchanged, and the home is **not** deleted |
| ED7 | Edit one of your own homes, pick another house in the dropdown, save | The house name changes, and hovering it shows the new house's zone. Change it back afterwards |
| ED8 | Start editing a row, click the search box and type a letter | The row goes back to normal. Then press Enter to open chat and type: the text goes into chat |
| OB1 | Hover the Owned bar's teleport button (bottom right) | `Teleport to the selected house` and `Right-click for outside` |

### Teleports (each one moves you, so do these last, from somewhere you can teleport)

| # | Do | Expect |
|---|---|---|
| TP1 | Left-click the icon on one of your own homes | Window closes, `Heading inside <label> (<house>)...`, and you arrive inside |
| TP2 | Right-click the icon on one of your own homes → Teleport Outside | Menu with Teleport Inside and Teleport Outside. `Heading outside ...`, and you arrive outside the house |
| TP3 | Left-click the icon on a friend's home | You arrive inside the friend's house |
| TP4 | Right-click the icon on a friend's home | Nothing happens, no menu |
| OB2 | Owned bar: pick a house, left-click the teleport button | `Heading inside <house>...`, and you arrive inside |
| OB3 | Owned bar: right-click the teleport button → Teleport Inside | You arrive inside |
| OB4 | Owned bar: right-click the teleport button → Teleport Outside | `Heading outside <house>...`, and you arrive outside the house |
| SC6 | `/gohome` | You arrive inside your primary residence |
| SC7 | `/gohome outside` | You arrive outside your primary residence |
| SC8 | `/gohome <part of a label>` | You arrive inside that home |

### After logging out

| # | Do | Expect |
|---|---|---|
| SV1 | Open `SavedVariables\CharacterHomes.lua` | Under `["NA Megaserver"]` (or your server), `["migrated"] = true` and `["ownerFilter"]` are there, and your homes look as before |

### Whenever it happens

| # | Do | Expect |
|---|---|---|
| OB5 | Owned bar teleport somewhere you can't teleport from (a dungeon, Cyrodiil) | `Cannot teleport from your current location.` and you stay put |
| CW1 | Optional. Another addon uses one of `/setglobalhome /setmyhome /resethome /delhome /delmyhome /gohome /homes` | After login, one chat line: `Already used by another addon, not registered: <commands>` |
| LS9 | Optional. No account homes at all | The note reads `No account homes -- use /setglobalhome <name> inside a house.` |

## History

What changed and what passed, newest first.

### Best-practice review and reused list rows (2026-11-21)

What changed:
- The list window reuses its rows, headers and notes instead of building new ones on every redraw. Before, every redraw (including each search keystroke) created new controls and left the old ones hidden forever. The row layout moved to XML templates (`CharacterHomesRow`, `CharacterHomesHeader`, `CharacterHomesNote`).
- The Owned bar's teleport button got the right-click Inside/Outside menu.
- The addon stops listening for other addons loading once it has loaded.
- The entry migration runs once (`migrated` flag) instead of on every login.
- The Owner filter setting is set explicitly for older saves.
- Slash commands are registered only if no other addon has them, with one chat line after login if any were taken.
- The teleport tooltip's house-name fallback and the `CH_DEBUG` flag were used before they were declared; both fixed.
- The manifest has `AddOnVersion: 140`, `APIVersion: 101051 101052` and the ZOS licence lines.
- The "No account homes" note now names `/setglobalhome` instead of `/sethome`.

Nothing passed yet.

## Results worth keeping
- **Outside teleport only works for your own homes.** `JumpToSpecificHouse(owner, houseId)` has no outside option; only `RequestJumpToHouse(houseId, true)` does, and that's for houses you own (same as BeamMeUp). Friends' homes have no Inside/Outside menu on purpose.
