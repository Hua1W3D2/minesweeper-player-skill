---
name: minesweeper-player
description: Play Minesweeper in a browser, especially on minesweeper.cn. Use this skill whenever the user asks Codex to play Minesweeper, minesweeper.cn, browser Minesweeper, or Chinese saolei/sweep-mine games. The skill first verifies browser-control MCP/tool availability, opens the game if needed, plays only from visible board information using normal Minesweeper logic, and only after all visible deterministic and exact-constraint moves are exhausted may transparently inspect the hidden answer to finish cells that would otherwise require guessing.
---

# Minesweeper Player

Use this skill to play browser Minesweeper. The default target is:

`https://www.minesweeper.cn/`

The contract is transparency:

1. Start in visible-information mode.
2. Do not read hidden mine locations while any provable visible move remains.
3. If visible play stalls, say that the remaining board requires a guess.
4. Only then, if the user has allowed this fallback, switch to answer-fallback mode and use hidden page state to finish non-deducible cells.

## Tool Check

Before opening or playing the page, check whether this Codex session has a browser-control MCP/tool available.

Suitable tools include Playwright-style browser tools such as:

- `mcp__playwright__.*`
- browser navigation, click, screenshot/snapshot, evaluate, or equivalent tools

If no browser-control tool is available:

1. Treat user references to "writer" as likely meaning Playwright.
2. Try to install or enable a Playwright/browser MCP only if the environment provides an approved installer path and permissions.
3. If installation is blocked or unavailable, say that browser control is unavailable and ask the user to provide screenshots or enable the browser MCP.

Do not pretend to control a webpage without a browser-control tool.

## Startup

1. Navigate to `https://www.minesweeper.cn/` unless the user has already opened a Minesweeper page.
2. Keep the user's current difficulty unless they request a different one.
3. If a new game is needed, click the face/restart button or call the page restart function only if that is equivalent to a normal restart.
4. Open the first cell from a reasonable central position. On minesweeper.cn, the first click is safe.

## minesweeper.cn Board Model

On minesweeper.cn the board is canvas-rendered, but the page maintains a game array:

- `g[y][x][0]`: visible state
  - `0` = covered
  - `1` = opened
  - `2` = flagged
- `g[y][x][2]`: adjacent mine number
- `h`: width
- `m`: height
- `E`: remaining flags/mines counter
- `R`: remaining safe cells
- `o`: game status, commonly `0` not started, `1` running, `2` won, `3` lost
- `l(x, y)`: open a cell
- `q(x, y)`: toggle a flag
- `_45()`: restart
- `u(x, y)`: first-click mine relocation/start helper

Visible-information mode may read:

- `g[y][x][0]` for every cell
- `g[y][x][2]` only for cells where `g[y][x][0] === 1`
- `E`, `R`, `h`, `m`, and `o`

Visible-information mode must not read or use:

- `g[y][x][1]`, because it is the hidden mine-answer field
- any other page variable, helper, debug output, or pixel technique that reveals hidden mine locations

Using `l(x, y)` and `q(x, y)` is acceptable as an input mechanism if the chosen cells come only from visible state.

## Visible Solver

Represent each covered cell as an unknown variable. Build constraints only from opened numbered cells:

`sum(unknown neighboring mines) = displayed number - flagged neighbors`

Apply moves in this order until no move remains:

1. Direct rules:
   - If `remaining mines around number === 0`, open every covered neighbor.
   - If `remaining mines around number === covered neighbor count`, flag every covered neighbor.
2. Subset rules:
   - If unknown set `A` is a subset of unknown set `B`, compare their mine counts.
   - If `need(B) - need(A) === 0`, cells in `B - A` are safe.
   - If `need(B) - need(A) === size(B - A)`, cells in `B - A` are mines.
3. Exact local constraints:
   - Split boundary unknowns into connected components by shared constraints.
   - Enumerate assignments for tractable components.
   - If every valid assignment makes a cell safe, open it.
   - If every valid assignment makes a cell a mine, flag it.
4. Global remaining-mine constraints:
   - Combine component mine-count distributions with the remaining mine counter.
   - Include isolated covered cells that touch no opened number.
   - Only act on cells whose state is forced in all globally valid assignments.

Never open the lowest-probability cell in visible-information mode. Probability is only for reporting when no forced move remains.

## Stalling

When visible play stalls:

1. Report opened cells, flagged cells, unknown cells, and remaining mines.
2. State that there is no provable next move from visible information.
3. If answer fallback is allowed by the user's instruction, proceed to answer-fallback mode.
4. If answer fallback is not allowed, stop and ask whether to guess, reveal, or restart.

For this skill, the requested default is: after visible play stalls, inspect the answer and finish the guess-dependent cells.

## Answer-Fallback Mode

Only enter this mode after visible-information mode has stalled.

For minesweeper.cn:

1. Say clearly that answer-fallback mode is starting.
2. Now it is permitted to read `g[y][x][1]`.
3. For each covered cell:
   - if `g[y][x][1] === 1`, flag it with `q(x, y)` if needed
   - if `g[y][x][1] === 0`, open it with `l(x, y)`
4. Verify the final status:
   - `o === 2` means won
   - `R === 0` and no opened mines is expected

Do not describe the fallback result as "pure rule-based play." Say that the final non-deducible part used the hidden answer after visible play stalled.

## Generic Browser Minesweeper

If the game is not minesweeper.cn:

1. Prefer DOM state if cells are HTML elements.
2. Use screenshots or canvas pixel inspection only to identify visible symbols, not hidden mine positions.
3. Build the same visible constraint solver from the displayed board.
4. Do not use source code or hidden variables to reveal mines until the user-approved fallback point.

## Communication

Keep updates short and explicit:

- "I am in visible-information mode."
- "I found forced moves from the shown numbers."
- "The visible solver has stalled; there is no guaranteed move left."
- "Switching to answer-fallback mode for the remaining guess-dependent cells."

If the user asks whether you used the answer, answer directly.

