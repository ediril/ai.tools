---
name: sync
description: Sync the shared, project-agnostic directives across every project's AGENTS.md. Finds what each project added, changed, or deleted since the last sync, resolves conflicts with the user, and applies the result to every project without touching project-specific directives. Use for "sync directives", "sync AGENTS.md", or "consolidate directives".
---

# Sync directives

Each project's `AGENTS.md` mixes two kinds of directives:

- **Shared directives** are project-agnostic. They hold in any project of the
  kinds they apply to.
- **Project-specific directives** name the project's files, domain, data,
  commands, dates, or decisions.

Shared directives evolve in whichever project the user happens to be working
in. This skill carries those changes to every other project, and it never
moves, rewords, or removes a project-specific directive.

## Locations

- **Projects:** `~/Documents/^_PROJ/*/AGENTS.md`. Skip `attic/` and
  `ai.tools/`.
- **Sync state:** `~/Documents/^_PROJ/ai.tools/directives/`. This is the
  user's local checkout of the ai.tools repo, not the installed plugin copy.
  If the folder is missing, ask the user where the checkout is.
  - `baseline.md`: the shared directives as of the last sync. This is the
    common ancestor for the comparison.
  - `projects.md`: each project's kinds and local exceptions.

If the user names projects, read and write only those projects. If a named
project has no `AGENTS.md`, offer to create one from the baseline directives
that match its kinds.

## State formats

`baseline.md`:

```markdown
# Shared directives

Last sync: 2026-11-19

## Working Style

- Investigate the root cause before fixing.
- [web] Before reporting a UI change done, drive the real interactions in a
  browser.
```

- An untagged directive applies to every project.
- A tagged directive, such as `[web]` or `[cli]`, applies only to projects of
  that kind.
- Section names are canonical. Projects may use different headings, so match
  sections by meaning.
- Write directives in neutral terms: "the project's test suites", not
  "pytest".

`projects.md`:

```markdown
| Project  | Kinds |
| -------- | ----- |
| doshmodo | web   |
| hlsync   | cli   |

## Local exceptions

- doshmodo, Versioning: keeps its own version scheme. Do not apply the shared
  versioning directive.
```

A local exception records that a project deliberately diverges from a shared
directive, either with its own wording or by opting out. Respect every
exception on every run.

## Procedure

If `baseline.md` does not exist, follow [Bootstrap](#bootstrap) instead of
steps 2 and 3.

### 1. Gather

For each project:

1. Read `AGENTS.md`.
2. Run `git -C <project> log --since=<last sync> -p -- AGENTS.md` to see what
   the project changed since the last sync.
3. Run `git -C <project> status --porcelain -- AGENTS.md`. Uncommitted edits
   are part of the project's current state. Mention them in the report.

### 2. Classify each project's differences

Compare each project's file with the baseline, filtered by the project's kinds
and exceptions. Each difference is one of these:

- **Added:** a directive that is not in the baseline. Decide whether it is
  shared or project-specific. It is shared only if it reads correctly in other
  projects as written, or after a neutral rewording. If you are not sure, ask.
- **Changed:** a baseline directive that the project reworded, tightened, or
  extended.
- **Deleted:** a baseline directive that applies to the project but is no
  longer in its file.
- **Not yet applied:** a baseline directive that the project never received.
  This is not a change by the project.

Use the git history to tell these apart:

- A difference that the project made since the last sync is the project's
  change.
- A directive is deleted only if the log shows the project removing it since
  the last sync. Otherwise, it is not yet applied.
- A difference that is older than the last sync is a local variant. Ask
  whether to record it as a local exception or overwrite it.

### 3. Resolve conflicts

Merge the changes from all projects into one change set. These are conflicts:

- Two projects changed the same directive differently.
- One project changed a directive and another project deleted it.
- An addition duplicates or contradicts an existing directive.

For a conflict in wording only, propose the newest version by commit date and
say so. For a conflict in substance, quote each version with its project and
date, and ask the user. Do not resolve a conflict in substance silently. Batch
the questions, a few at a time.

### 4. Confirm

Before you write anything, show the change set:

- For each directive: the action, the source project and date, the target
  section, and the projects that receive it.
- The projects that receive no change, and why (kind or exception).

Wait for the user's approval.

### 5. Apply

1. Update `baseline.md`, including the `Last sync` date. Update `projects.md`
   with any new kinds or exceptions.
2. Edit each project's `AGENTS.md`:
   - Put each addition in the section that matches it by meaning. Follow the
     file's heading style, bullet style, and line wrapping. Create a section
     only if no section matches.
   - Replace changed directives in place. Remove deleted directives.
   - Do not reorder, reword, or reformat anything else.
3. If the project's directives say that a memory file is an exact copy of
   `AGENTS.md`, apply the same edits to that file. If it is a symlink, do
   nothing.
4. Do not commit in the projects or in ai.tools unless the user asks.

### 6. Verify

1. Run `git -C <project> diff -- AGENTS.md` for each project you changed.
   Every hunk must match an approved change, and no project-specific line may
   be changed or removed. Fix any hunk that does not match.
2. Report each project: the directives added, changed, and removed, with a
   short name for each, and the projects left unchanged.

## Bootstrap

Use this on the first run, when no baseline exists.

1. Propose each project's kinds (for example, web app or CLI) from its
   `README.md` and build files. Ask the user to confirm them, then write
   `projects.md`.
2. Split every `AGENTS.md` into individual directives. Set aside the
   project-specific ones.
3. Group the remaining directives by meaning across projects. For each group,
   start from the newest wording, using `git blame --date=short` for the date
   of each line. Merge in points from older versions only if the newest
   version lacks them and they still hold.
4. Tag the directives that fit only some kinds of project.
5. Ask the user about:
   - Groups whose versions disagree in substance.
   - Directives that appear in only one or two projects: spread them, or keep
     them local?
6. Write `baseline.md`. Then continue with step 4 (Confirm), and treat every
   baseline directive that is missing from a project as an addition.
