---
name: interrogate
description: Multi-model adversarial review. Reviewers on different models try to break a change or design, and only findings proven by a run survive. Use only when the user asks for it, for example "interrogate", "adversarial review", "multi-model review", "challenge this", or "tear this apart". Do not start it on your own initiative.
---

# Interrogate

Reviewers on different models try to break a change. A finding counts only
when a run proves it. Report the findings, and do not fix anything until the
user picks what to fix.

This skill runs from Claude Code. It needs the Agent tool for the Claude
reviewers and the `codex` CLI for the GPT reviewer.

## Panel

| Change | Reviewers           |
| ------ | ------------------- |
| Small  | GPT, Opus           |
| Large  | Fable, Opus, GPT    |

A change is **small** when it has up to about 200 changed lines, stays in
one area, and changes no data format, persistence, concurrency, security
boundary, or public interface. Every other change is **large**. If the user
names a size or the reviewers, use theirs.

The reviewers, each pinned to one model at high effort:

- **Opus 5.5:** the Agent tool with `subagent_type: emd:opus-5-5-high`.
- **Fable 5.2:** the Agent tool with `subagent_type: emd:fable-5-2-high`.
- **GPT 6.1 Sol:** the Codex CLI, model `gpt-6.1-sol`, reasoning effort
  `high`. See [Launch](#2-launch) for the command.

Do not pass a `model` or `effort` to the Agent tool. The agent definitions
in the plugin's `agents/` folder set both. To change a model, edit those
files and the Codex command.

If a reviewer can't run (the CLI is missing, a login expired, the server is
overloaded), retry it once. If it still fails, name it as a dropout with the
reason, and continue with the others. Do not replace it with another model.
Agreement across different models is what makes the review worth its cost.

## 1. Frame

1. **Scope:** what the user names. Otherwise, the uncommitted changes plus
   the branch's commits since its merge base with the default branch. If
   the scope is empty, ask.
2. **Intent:** one or two sentences on what the change must do, from the
   user's message, the commit messages, and any `TODO.md` or `decisions/`
   notes that cover it. If you can't tell, ask.
3. **Size:** classify the change and pick the panel. Tell the user the size,
   the reason, and the reviewers in one line, then continue without waiting.
4. **Baseline:** record `git status --porcelain` and `git diff --stat`, so
   that you can check afterwards that no reviewer changed the checkout.
5. **Scratch folders:** create one folder per reviewer outside the repo:
   `${TMPDIR:-/tmp}/interrogate-<repo>-<YYYYMMDD-HHMM>/<reviewer>/`.

## 2. Launch

Start every reviewer at once: the Agent calls in one message, and the Codex
command with `run_in_background`. Each reviewer gets the same brief, with
its own scratch folder filled in:

> Review this change in `<repo path>`: `<scope>`. Its intent: `<intent>`.
>
> Try to break it. Prove each finding with a run against the real code: a
> failing test, a repro script, or command output that shows the wrong
> behavior. For a design or a plan, probe the current code or run a small
> prototype.
>
> You may read anything. Do not create, edit, or delete anything in the
> repo, and do not run git commands that change state. Write repro scripts
> and every other file only in `<scratch folder>`. If a check would write
> inside the repo, such as a build, run it only if it leaves the repo's
> files unchanged. Otherwise, leave it out.
>
> For each finding, report: a short descriptive title (not a code like
> "F1"), the exact command, the output it printed, and the output the
> intent requires. Leave out anything you did not run: style, naming,
> structure, and risks that need a future change to matter. No findings is
> a valid result.

Run the GPT reviewer from its scratch folder. That way, its sandbox lets it
read the repo but write only to the scratch folder:

```sh
cd <scratch>/gpt && codex exec -m gpt-6.1-sol \
  -c model_reasoning_effort='"high"' -s workspace-write \
  --skip-git-repo-check --ephemeral -o report.md - < brief.md
```

## 3. Verify

1. Compare `git status --porcelain` and `git diff --stat` with the baseline.
   If a reviewer changed the checkout, tell the user what changed. Do not
   revert it on your own.
2. Rerun each finding's command yourself. Keep a finding when the output
   shows the failure and the failure breaks the intent. Drop the rest, with
   a reason.
3. Merge findings that share a root cause, and note which reviewers found
   each one. A finding from two different models is a strong signal.

## 4. Report

- **Intent:** the restatement.
- **Panel:** the size and why, and each reviewer's model, or its dropout and
  the reason.
- **Findings,** most severe first. For each: its title, what breaks and for
  whom, the command and the key lines of its output, and the reviewers that
  found it. If none survive, say "no verified findings". If no reviewer
  completed, say that the review didn't happen.
- **Dropped:** for each reviewer, how many findings were dropped, and why.

End by asking which findings to fix. Fix them yourself, in the main session.
Keep the scratch folders until the fixes are done, since their repro scripts
can become regression tests. Then delete them.

## Credits

Adapted from the `interrogate` skill in pstack, (c) 2026 Lauren Tan, MIT
license. See [LICENSE](LICENSE).
