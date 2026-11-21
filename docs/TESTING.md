# CharacterHomes in-game tests

Note anything unexpected along with the exact chat output. A Lua error popup at any point is a
failure. Copy its first few lines.

Every test still to do is under **To do**, grouped by when you can run it. Everything after that
is history: what changed, what passed, and notes worth keeping. Passed tests are removed from
To do.

## To do

### Whenever it happens

| # | Do | Expect |
|---|---|---|
| SV2 | Next time you play on EU: log in, `/homes` (homes as before), log out, then open `SavedVariables\CharacterHomes.lua` | Under `["EU Megaserver"]`, `["migrated"] = true` and `["ownerFilter"] = 0` are now there, and the EU homes are unchanged |
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

Passed:
- LD1: listed and enabled at the character screen, with no "out of date" warning.
- LD2: no Lua error and no Character Homes chat line at login.
- SC1–SC4: `/setglobalhome`, `/delhome` and `/delmyhome` with no name show their usage lines, and `/setmyhome` outside a house says `You must be inside a house.`
- SC5: `/homes` opens the window with every existing home.
- LS1–LS3: columns line up on every row, the primary residence has the home icon and the rest the wayshrine, and the colours are right.
- LS4: teleport icon tooltips are right for your own homes and friends' homes.
- LS5–LS6: hovering a house name highlights it with the zone tooltip, and clicking it opens the map.
- LS7: searching `zzzz` shows both "no match" notes.
- LS8: the last row is fully visible at the bottom of the list, with no blank area under it.
- PL1: the list updates on every keystroke while typing and deleting a search, with no leftover, doubled or blank rows.
- PL2: each Owner filter option shows the right homes.
- PL4: closing and reopening five times shows the same list, with nothing doubled.
- PL3 (after the fix below): the window reopens on the saved owner filter.
- PL5: the row count stays the same after more searching, so rows are reused.
- PL6: deleting a row under the mouse leaves no stuck highlight.
- PL7: no house name stuck white and no leftover tooltip after a redraw.
- ED1: the label editor opens with the row's own house already selected in the dropdown.
- ED2: friends' homes get the label editor only.
- ED4: clicking ✓ saves the label.
- ED3 (after the fix below): Enter saves the label.
- ED5: Escape cancels the edit and leaves the window open, and a second Escape closes the window.
- ED6: clicking X in edit mode cancels without deleting. But the X's tooltip still said `Remove from list`, which makes it look like it deletes (fixed, see below).
- ED7: changing a row's house saves, and the new zone shows on hover.
- ED8: clicking the search box and typing takes the row out of edit mode. (Enter not opening chat while the window is open is older behaviour, see Results worth keeping.)
- ED9: the ✓ and X tooltips read `Save` and `Cancel` in edit mode and switch straight away when clicked.
- OB1: the Owned bar teleport button's tooltip mentions right-click for outside.
- TP1–TP2: your own homes teleport inside on left-click, and right-click offers Inside/Outside, with outside working.
- TP3–TP4: friends' homes teleport inside on left-click, and right-click does nothing.
- OB2–OB4: the Owned bar teleport button goes inside on left-click, and its right-click Inside and Outside both work.
- SC6–SC8: `/gohome`, `/gohome outside` and `/gohome <part of a label>` all arrive where expected.
- SV1: the NA Megaserver save has `["migrated"] = true` and `["ownerFilter"] = 0`, no leftover `["Default"]` section, and all homes. EU doesn't have the new keys yet because it hasn't been played since (SV2).

Failed, then fixed:
- ED6 tooltip: in edit mode the ✓ and X buttons kept their normal tooltips (`Edit label or change house`, `Remove from list`). They now say `Save` and `Cancel` while editing, and switch straight away if the mouse is already on the button (checked in ED9).
- ED3: Enter in the label editor did nothing. The addon set `OnEnterPressed` and `OnEscapePressed` handlers, but ESO's edit boxes fire `OnEnter` and `OnEscape` (as in ESO's own `ZO_DefaultEdit` template), so neither ever ran. This was there before today's changes. Both now use the right names (Escape is checked in ED5).
- PL3: with Friends' homes saved, the window reopened on My homes. The dropdown was sorting its options alphabetically (All, Friends', My), so restoring the saved choice by its position picked the wrong one. That dropdown now keeps its own order. The house dropdown in each row's editor had the same risk, since it also picks by position, so it keeps its own order too (checked in ED1).

## Results worth keeping
- **Enter doesn't open chat while the homes window is open.** The window has been `keyboardEnabled` since 1.4 so it can close on Escape, which means it takes every key while open, and `CH.OnKeyDown` only acts on Escape. Clicking into chat still works. Not changed so far; if wanted, `CH.OnKeyDown` could open chat on Enter.
- **Outside teleport only works for your own homes.** `JumpToSpecificHouse(owner, houseId)` has no outside option; only `RequestJumpToHouse(houseId, true)` does, and that's for houses you own (same as BeamMeUp). Friends' homes have no Inside/Outside menu on purpose.
