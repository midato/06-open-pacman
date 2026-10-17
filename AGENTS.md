# AGENTS.md

## What this is

Vanilla JS Pac-Man (no framework, no bundler). The repo's purpose is learning
**spec-driven development**: features start as a spec in `specs/`, then get
implemented by the `spec-impl` skill.

## Run / verify

- No `package.json`, no build, no test, no lint. There is nothing to install.
- To run: open `src/index.html` directly in a browser (classic `<script>` tags,
  no ES modules, no `fetch` — `file://` works).
- Verification = play the game against the spec's acceptance criteria.

## Architecture (not obvious from filenames)

- Scripts load in a fixed order defined in `src/index.html`:
  `maze.js → game.js → render.js → main.js`. Files communicate through
  `window.*` exports (`window.MAZE`, `window.createGame`, `window.draw`, …).
  **Do not convert to ES modules or reorder scripts** without changing every
  dependency at once.
- Maze tiles (parsed in `src/js/maze.js`): `#`=1 wall, `.`=2 dot,
  `-`=3 ghost door, space=0 empty. Door (3) blocks Pac-Man but not ghosts.
- `MAZE` is the pristine template; each match copies it into `game.grid`
  (`createGame` in `src/js/game.js`). Render reads `game.grid` (eats dots),
  never `MAZE`.
- Movement uses fractional cell coords. Speeds are cells/frame
  (Pac-Man 1/8, ghost 1/10), so actors snap only when `aligned()` passes;
  tunnel wrap happens on row `TUNNEL_ROW` (14) only.

## Spec workflow

- Specs live in `specs/NN-slug.md`; the folder does not exist yet — first spec
  starts at `01-`. Template: `.agents/skills/spec/template.md`.
- `/spec <feature>` writes the spec in `Draft`; **only the user** flips it to
  `Approved`. `/spec-impl <NN>` refuses any state that does not mean "Approved".
- `/spec-impl` creates branch `spec-NN-slug` (auto unless
  `specs/.spec-config.yml` sets `AutoCreateBranch: false`), implements plan
  steps one at a time pausing for diff review, and **never commits on its own**.
- The skills in `.agents/skills/` are pinned by hash in `skills-lock.json`
  (source `klerith/fernando-skills`). Don't edit them locally or verification breaks.
- **This directory is not a git repo yet** — `/spec-impl` needs git, so
  `git init` (or the user) must happen before implementing.

## Conventions (from `../instructions.md`)

- Code, identifiers, file names: **English**. Comments and UI strings:
  **Spanish**, only where they add value.
- Be concise; prefer small functions, DRY, no superfluous comments.
