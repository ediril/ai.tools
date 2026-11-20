---
name: sync-directives
description: Sync the shared, project-agnostic directives across every project's AGENTS.md. Reads what each project added, changed, or deleted since the last sync, merges it into one master file, then decides which master directives apply to each project and updates them, without touching project-specific directives. Use for "sync directives", "sync AGENTS.md", or "consolidate directives".
---

# Sync directives

Each project's `AGENTS.md` mixes two kinds of directives:

- **Shared directives** are project-agnostic.
- **Project-specific directives** name the project's files, domain, data,
  commands, dates, or decisions.

Shared directives evolve in whichever project the user happens to be working
in. A sync runs in two directions:

1. **Inbound:** collect each project's changes to shared directives into one
   master file.
2. **Outbound:** decide which master directives apply to each project, and
   update the project.

The sync never moves, rewords, or removes a project-specific directive.

## Locations

The sync assumes that all projects sit side by side in one folder, the
**projects folder**, and that the session started there. It sees only the
projects directly inside the session's folder.

- **Projects:** every top-level folder of the projects folder that has an
  `AGENTS.md` or one of the [other directive files](#other-directive-files),
  plus the ai.tools checkout, which is a project like any other. If the user
  names projects, read and write only those. If a project has no
  `AGENTS.md`, outbound proposes one.
- **Master file:** `directives/AGENTS.md` in the user's local checkout of the
  ai.tools repo, normally `ai.tools/` in the projects folder. Use the
  checkout, not the installed plugin copy. The master file is separate from
  the repo's own `AGENTS.md`. If there is no checkout in the projects folder,
  ask the user where it is.
- **Memory copies:** a project's directives may say that its Claude memory
  file is an exact copy of `AGENTS.md`. That file is `MEMORY.md` in
  `~/.claude/projects/<folder>/memory/`, where `<folder>` is the project's
  absolute path with every character other than a letter or digit replaced
  by `-`. For example, `/Users/me/code/ai.tools` maps to
  `-Users-me-code-ai-tools`.

Before you start, check the session's folder. If it is itself a project (it
has a `.git` folder or an `AGENTS.md`), or it holds fewer than two projects,
tell the user: the sync can see only the projects inside this folder, so it
can't gather directives from, or update, projects elsewhere. Suggest
restarting the session from the folder that holds all the projects. Continue
only if the user says so.

## No cached state

The master file holds only the shared directives and the time of the last
sync. Do not record anything about individual projects. Each project's git
history is the record of what it chose: what it added, reworded, or removed,
and when. Read it on every run.

Master file format:

```markdown
# Shared directives

Last sync: 2026-11-19 21:30

## Working Style

- Investigate the root cause before fixing.
- Before reporting a change done, drive the real interactions in the app.
```

- Write directives in neutral terms, so that each one reads correctly in any
  project it applies to: "the project's test suites", not "pytest".
- Section names in the master are canonical. Projects may use different
  headings, so match sections by meaning.

If the master file does not exist, follow [First run](#first-run).

## Other directive files

Other tools read their own instruction files. All of a project's directives
belong in its `AGENTS.md`, so the sync consolidates these files into it:

- In the project: `CLAUDE.md`, `GEMINI.md`, `.cursorrules`, `.cursor/rules/`,
  `.windsurfrules`, `.clinerules`, and `.github/copilot-instructions.md`.
- In the project's Claude memory folder: every file other than `MEMORY.md`.

For each file you find:

- If it is a symlink to `AGENTS.md`, it is already consolidated. Leave it.
- Otherwise, compare each directive in it with `AGENTS.md`. Skip duplicates.
  Treat every other directive as an addition to the project's `AGENTS.md`:
  inbound sorts it into shared or project-specific like any other addition.
  If a directive contradicts `AGENTS.md`, quote both and ask which holds.
- Propose removing the file once its directives are in `AGENTS.md`. Before
  you remove a `CLAUDE.md`, make sure Claude Code still reads the
  directives: the project needs a memory copy of `AGENTS.md` or a symlink to
  it.

Leave personal, untracked files such as `CLAUDE.local.md` alone, and mention
them in the report. Their content may not belong in a committed file.

## 1. Inbound: projects to master

For each project:

1. Read `AGENTS.md` and consolidate any
   [other directive files](#other-directive-files).
2. Read the project's changes to it since the last sync:
   - Committed: `git -C <project> log --since="<last sync>" -p -- AGENTS.md`
   - Uncommitted: `git -C <project> diff HEAD -- AGENTS.md`. Mention
     uncommitted edits in the report.
3. If the project keeps a memory copy, compare it with `AGENTS.md`. An agent
   may have edited either one. If they differ, show the difference and ask
   which version is current. Treat the answer as the project's current
   `AGENTS.md`. If the copy is missing, propose creating it. If it is a
   symlink to `AGENTS.md`, skip this check.
4. Skip changes to project-specific directives. If you can't tell whether a
   directive is shared or project-specific, ask.
5. Sort each change to a shared directive:
   - **Added:** a new directive. Add it to the master, reworded in neutral
     terms if needed.
   - **Edited:** a master directive that the project reworded, tightened, or
     extended. If the new wording reads correctly in any project, adopt it.
     If it is tailored to this project, ask: adopt it everywhere, or keep it
     local to this project?
   - **Deleted:** a master directive that the project removed. The intent
     is ambiguous, so always ask: drop it everywhere, or just here?

Also read the changes to the master file since the last sync, committed and
uncommitted, in the ai.tools checkout. The user may have edited it directly.

Ask questions as they come up while you read, a few at a time. Do not
hold them all to the end.

Changes from different projects can conflict:

- Two projects edited the same directive differently.
- One project edited a directive and another deleted it.
- An addition duplicates or contradicts a master directive.

If the versions differ only in wording, propose the newest one by commit
date, and say so. If they differ in substance, quote each version with its
project and date, and ask.

## 2. Outbound: master to projects

Keep a copy of the master as it was before this run. Then, for each project,
compare every master directive with the project's file:

- **Present with the same meaning:** leave it alone.
- **Present with different wording:** check the project's history.
  - If the project's wording matches an earlier version of the master
    directive (the pre-run copy, or the master file's git history),
    the project only has the old version. Update it.
  - Otherwise, the project chose its own wording. That wording is a local
    variant. Leave it alone, and list it in the change set.
- **Missing:** search the project's full history for the directive in any
  wording, committed and uncommitted: `git -C <project> log -p -- AGENTS.md`
  and `git -C <project> diff HEAD -- AGENTS.md`.
  - If the project had it and removed it, respect the removal. Do not
    re-add it.
  - If the project never had it, decide whether it fits this project, using
    the project's `AGENTS.md`, `README.md`, and build files. If it fits,
    propose adding it. If not, say why in the change set.
- **Contradicted by one of the project's own directives:** this is a
  conflict, not a question of fit. Do not settle it by leaving the master
  directive out. Quote both, and ask: does the master directive hold here
  too, should the project's directive replace it in the master, or is this
  a local exception?

Decide fit fresh on every run. Do not rely on an earlier run's decision.

A project with no `AGENTS.md` has never had any directive. Propose a new
file with the directives that fit it, in the master's sections and order.
If the master tells projects to keep a memory copy, propose that copy too.

## 3. Confirm

Before you write anything, show the change set:

- **Master:** each directive added, edited, or removed, with its source
  project and date.
- **Per project:** each directive to add, update, or remove, and its target
  section, and each other directive file to consolidate and remove.
- **Left alone:** local variants, respected removals, and directives
  judged not to fit, each with a one-line reason.

Wait for the user's approval. The user may drop items or move a directive
in or out of a project.

## 4. Apply

1. Update the master file, including the `Last sync` time (local time, to
   the minute).
2. Edit each project's `AGENTS.md`:
   - Put each addition in the section that matches it by meaning. Follow the
     file's heading style, bullet style, and line wrapping. Create a section
     only if no section matches.
   - Replace updated directives in place. Remove directives that were
     dropped everywhere.
   - Do not reorder, reword, or reformat anything else.
3. If the project keeps a memory copy, make it identical to the new
   `AGENTS.md`. If it is a symlink, do nothing.
4. Remove the other directive files that were approved for removal.
5. Do not commit unless the user asks. Remind the user that each project's
   history is the record that the next sync reads. A sync's edits and
   decisions only count once they are committed. This includes the master
   file in ai.tools.

## 5. Verify

1. Run `git -C <project> diff -- AGENTS.md` for each project you changed.
   Every hunk must match an approved change, and no project-specific line may
   change. Fix any hunk that does not match.
2. Check that each memory copy is identical to its `AGENTS.md`, and that
   every directive from a removed file is in `AGENTS.md`.
3. Report each project: the directives added, updated, and removed, with a
   short name for each, and the projects left unchanged.
4. End with a ready-to-paste commit message for each changed project and
   one for the master file in ai.tools, with no attribution trailer. Use
   each project's subject style from its `git log`. If a project has other
   uncommitted changes, say so, and say which files the message covers.

## First run

When the master file does not exist, build it before the outbound step:

1. Split every project's `AGENTS.md` into individual directives. Set aside
   the project-specific ones.
2. Group the remaining directives by meaning across projects. For each group,
   start from the newest wording, using `git blame --date=short` for the
   date of each line. Merge in points from older versions only if the newest
   version lacks them and they still hold.
3. Ask the user about:
   - Groups whose versions disagree in substance.
   - Directives that appear in only one or two projects: share them, or keep
     them local?
4. Hold the result as the proposed master, with no earlier version. Run
   [Outbound](#2-outbound-master-to-projects). Treat a project's differing
   wording as the old version unless it is tailored to the project. If it is
   tailored, ask.
5. Continue with [Confirm](#3-confirm). Write the master file only after
   approval.
