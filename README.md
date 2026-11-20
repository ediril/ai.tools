# ai.tools

My agent skills, packaged as a plugin marketplace for Claude Code and Codex.

## Plugins

| Plugin | Skill | What it does |
| --- | --- | --- |
| `emd-directives` | `sync` | Syncs the shared, project-agnostic directives across every project's `AGENTS.md`, without touching project-specific directives. |

## Install

Claude Code:

```sh
claude plugin marketplace add ediril/ai.tools
claude plugin install emd-directives@ai-tools
```

Codex:

```sh
codex plugin marketplace add ediril/ai.tools
codex plugin add emd-directives@ai-tools
```

To try a plugin in Claude Code without installing it, load it for one
session: `claude --plugin-dir <path to ai.tools>/plugins/<plugin>`.

## Running the directives sync

`emd-directives:sync` assumes that all your projects sit side by side in one
folder, together with your clone of this repo:

```text
projects/
├── ai.tools/
├── project-a/
└── project-b/
```

Start the session from that folder, then run `/emd-directives:sync`. The sync
sees only the projects inside the folder the session started in. Started from
inside one project, or with projects spread across several folders, it can't
read or update the projects it can't see, so the shared directives drift
apart again.

The sync also folds other agent instruction files, such as `CLAUDE.md` or
`.cursorrules`, into each project's `AGENTS.md`, and removes them after you
approve.

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
  the `emd-directives:sync` skill reads and writes. The first sync creates it.
