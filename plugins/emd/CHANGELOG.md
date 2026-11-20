# Changelog

Major version 0 started on 2026-11-19.

## 0.1.4 (2026-11-20)

- The sync now proposes deleting every project `CLAUDE.md`, including a
  symlink to `AGENTS.md` and one that only imports it, in favor of
  `AGENTS.md`. If the user keeps a `CLAUDE.md`, it recommends making
  `@AGENTS.md` its only content.

## 0.1.3 (2026-11-20)

- `sync-directives` no longer keeps a copy of `AGENTS.md` in each project's
  Claude memory folder. Claude Code now reads `AGENTS.md` directly and in
  full, while a memory copy loaded the directives a second time, cut off
  after 200 lines. The sync proposes removing every file in the memory
  folders, after merging any directives that aren't in `AGENTS.md`.
- The sync now proposes `"autoMemoryEnabled": false` in
  `~/.claude/settings.json` when it isn't set.
- The sync now warns that a `CLAUDE.local.md` stops Claude Code from
  reading `AGENTS.md`.

## 0.1.2 (2026-11-20)

- Pinned interrogate's reviewers to exact models at high effort: Opus 5.5
  and Fable 5.2 through two new agents, `emd:opus-5-5-high` and
  `emd:fable-5-2-high`, and GPT 6.1 Sol through the Codex command.

## 0.1.1 (2026-11-20)

- Renamed the plugin from `emd-directives` to `emd`, which will hold all of
  these skills. The sync skill is now `sync-directives`, run as
  `/emd:sync-directives`.
- Added `interrogate`, adapted from pstack's skill of the same name.
  Reviewers on different models try to break a change, and only findings
  proven by a run survive. A small change gets GPT and Opus; a large one
  gets Fable, Opus, and GPT. Reviewers read the checkout but write only to
  their own temporary folders, with no git worktrees. It runs only when the
  user asks for it.

## 0.0.6 (2026-11-19)

Fixes from the first real sync:

- When a project's own directive contradicts a master directive, the sync
  now asks about the conflict instead of quietly leaving the master
  directive out of that project.
- The check for past removals now reads uncommitted edits too, so a
  directive deleted in an uncommitted edit is not added back.
- The sync now ends with a ready-to-paste commit message for each changed
  project and for the master file.

## 0.0.5 (2026-11-19)

- The sync no longer assumes one machine's folder layout. It treats the
  session's folder as the folder that holds all projects, and warns if the
  session started inside a project or a folder with fewer than two
  projects.
- The sync now consolidates other agent instruction files (`CLAUDE.md`,
  `GEMINI.md`, Cursor, Windsurf, Cline, and Copilot rules, and extra Claude
  memory files) into `AGENTS.md`, and proposes removing them. It leaves
  personal files such as `CLAUDE.local.md` alone.

## 0.0.4 (2026-11-19)

- Renamed the plugin from `directives` to `emd-directives`, so that it does
  not collide with another plugin of the same name. The command is now
  `/emd-directives:sync`.

## 0.0.3 (2026-11-19)

- The sync now includes ai.tools as a project, and proposes an `AGENTS.md`
  for a project that has none.
- The sync now keeps each project's Claude memory copy
  (`~/.claude/projects/<folder>/memory/MEMORY.md`) identical to its
  `AGENTS.md`. If the two have drifted, it asks which is current. This
  replaces the earlier mention of `CLAUDE.md`.

## 0.0.2 (2026-11-19)

- Replaced `baseline.md` and `projects.md` with one master file,
  `directives/AGENTS.md`. The sync no longer stores anything about
  individual projects; it reads each project's git history on every run.
- Removed project kinds and tags. The sync decides fresh on each run
  whether a directive fits a project.
- The sync now asks whether a deleted directive should be dropped
  everywhere or only in that project, and respects past removals.
- The first run writes the master file only after the user approves the
  change set.

## 0.0.1 (2026-11-19)

- First version of the `sync` skill: syncs the shared directives across every
  project's `AGENTS.md`, with a bootstrap mode for the first run.
