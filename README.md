# Minesweeper Player Skill

A Codex skill for playing browser Minesweeper, with a strict visible-information-first workflow.

The skill is designed for `https://www.minesweeper.cn/`, but the visible solver guidance also applies to other browser Minesweeper games.

## What It Does

- Checks whether a browser-control MCP/tool is available.
- Opens `https://www.minesweeper.cn/` when needed.
- Starts by playing only from visible board information:
  - opened numbers
  - flags already placed
  - covered cells
  - remaining mine counter
- Applies standard Minesweeper deduction:
  - direct number rules
  - subset rules
  - exact local constraint solving
  - global remaining-mine constraints
- Stops and reports when the visible board has no provable next move.
- If the user has allowed the fallback, then and only then reads hidden page state to finish cells that require guessing.

## Install

Copy the skill folder into your local skills directory.

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse -Force .\minesweeper-player "$env:USERPROFILE\.codex\skills"
```

macOS/Linux:

```bash
mkdir -p ~/.agents/skills
cp -R ./minesweeper-player ~/.agents/skills/minesweeper-player
```

Restart Codex after installing so the skill list refreshes.

## Usage

Example prompts:

```text
Use $minesweeper-player to play minesweeper.cn for me.
```

```text
Help me play this Minesweeper page. Use only visible information first, then reveal the answer only if the board becomes a forced guess.
```

```text
Play Minesweeper normally. Do not read hidden mines until all visible deductions are exhausted.
```

## Project Layout

```text
minesweeper-player-skill/
  README.md
  LICENSE
  minesweeper-player/
    SKILL.md
    agents/
      openai.yaml
```

## Notes

This skill deliberately separates normal play from answer fallback. If the final cells are completed using hidden page state, the agent must say that clearly and must not describe the run as purely rule-based.

