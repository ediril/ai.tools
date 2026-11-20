# ai.tools

My agent skills, packaged as a plugin marketplace for Claude Code and Codex.

## Skills

All skills ship in one plugin, `emd`.

| Command | What it does |
| --- | --- |
| `/emd:sync-directives` | Syncs the shared, project-agnostic directives across every project's `AGENTS.md`, without touching project-specific directives. |
| `/emd:interrogate` | Has reviewers on different models try to break a change, and reports only the findings that a run proves. Runs only when you ask for it. |

## Install

Claude Code:

```sh
claude plugin marketplace add ediril/ai.tools
claude plugin install emd@ai-tools
```

Codex:

```sh
codex plugin marketplace add ediril/ai.tools
codex plugin add emd@ai-tools
```

To try a plugin in Claude Code without installing it, load it for one
session: `claude --plugin-dir <path to ai.tools>/plugins/emd`.

## Running the directives sync

`/emd:sync-directives` assumes that all your projects sit side by side in one
folder, together with your clone of this repo:

```text
projects/
├── ai.tools/
├── project-a/
└── project-b/
```

Start the session from that folder, then run `/emd:sync-directives`. The sync
sees only the projects inside the folder the session started in. Started from
inside one project, or with projects spread across several folders, it can't
read or update the projects it can't see, so the shared directives drift
apart again.

The sync also folds other agent instruction files, such as `CLAUDE.md` or
`.cursorrules`, into each project's `AGENTS.md`, and removes them after you
approve. Claude Code and Codex both read `AGENTS.md` directly, so the sync
also clears Claude's per-project memory folders and turns auto memory off
(`"autoMemoryEnabled": false` in `~/.claude/settings.json`), so that no
directives live outside `AGENTS.md`.

## Running interrogate

`/emd:interrogate` reviews the current change: the uncommitted changes plus
the branch's commits, or whatever you name. A small change gets two
reviewers, GPT and Opus. A large one gets three: Fable, Opus, and GPT. You
can name the size or the reviewers yourself.

The models are pinned, all at high effort: Opus 5.5 and Fable 5.2 through
the agents in `plugins/emd/agents/`, and GPT 6.1 Sol through the Codex
command in the skill. To change a model, edit those.

Reviewers read your checkout but never change it. They write their repro
scripts in a temporary folder outside the repo. The GPT reviewer runs
through the `codex` CLI, so Codex must be installed and logged in.

## Update

Each plugin's version lives in its two `plugin.json` files. The tools update
an installed plugin only when this version changes, so bump it with every
completed change.

A version is `<major>.<days since the major version started>.<increment
within that day>`. Each plugin's `CHANGELOG.md` records the start date of
each major version.

1. Change the plugin.
2. Run `scripts/bump-version <plugin>`. It sets the version in both
   manifests and adds an entry to the plugin's `CHANGELOG.md`. To start a
   new major version today, run `scripts/bump-version <plugin> major`.
3. Describe the change in that entry, then commit and push.
4. Refresh the marketplace:
   - Claude Code: `claude plugin marketplace update ai-tools`
   - Codex: `codex plugin marketplace upgrade`

## Layout

- `plugins/<plugin>/`: each plugin, with a manifest for each tool in
  `.claude-plugin/` and `.codex-plugin/`, and its skills in `skills/`.
- `directives/AGENTS.md`: the master copy of the shared directives, which
  `/emd:sync-directives` reads and writes. The first sync creates it.
