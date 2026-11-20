# ai.tools

My agent skills, packaged as a plugin marketplace for Claude Code and Codex.

## Plugins

| Plugin | Skill | What it does |
| --- | --- | --- |
| `directives` | `sync` | Syncs the shared, project-agnostic directives across every project's `AGENTS.md`, without touching project-specific directives. |

## Install

Claude Code:

```sh
claude plugin marketplace add ediril/ai.tools
claude plugin install directives@ai-tools
```

Codex:

```sh
codex plugin marketplace add ediril/ai.tools
codex plugin add directives@ai-tools
```

## Update

Each plugin's version lives in its two `plugin.json` files. The tools update
an installed plugin only when this version changes.

1. Change the plugin.
2. Run `scripts/bump-version <plugin> patch` (or `minor`, `major`, or an
   exact `X.Y.Z`). It sets the version in both manifests and adds an entry to
   the plugin's `CHANGELOG.md`.
3. Describe the change in that entry, then commit and push.
3. Refresh the marketplace:
   - Claude Code: `claude plugin marketplace update ai-tools`
   - Codex: `codex plugin marketplace upgrade`

## Layout

- `plugins/<plugin>/`: each plugin, with a manifest for each tool in
  `.claude-plugin/` and `.codex-plugin/`, and its skills in `skills/`.
- `directives/`: the sync state that the `directives:sync` skill reads and
  writes. The first sync creates it.
