# Changelog

Major version 0 started on 2026-11-19.

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
