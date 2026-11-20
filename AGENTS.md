# ai.tools

## Design And Implementation

- Preserve functionality by default when replacing or reorganizing code. Change
  behavior only for a concrete problem discussed and agreed with the user; messy
  implementation alone does not justify it. Use the previous code, tests and
  history to understand what must be preserved, and investigate behavioral
  differences.
- Don't keep backwards compatibility (old formats, aliases, shims) when changing
  something unless explicitly asked to.
- Before suggesting, changing or removing a mechanism, trace where it came from
  (git history including unmerged and abandoned branches, session transcripts,
  `CHANGELOG.md`, `decisions/`, `TODO.md`, and any research notes): why it was
  added, what it replaced, what was agreed and what was tried and dropped around
  it. Bring what was found with the suggestion, so it never repeats a past
  mistake, reverses an agreement or returns to something already moved away from.
- Before architecture edits, agree on the milestone's behavior, representation,
  owners, affected entry paths and checks. Walk a concrete example through the
  full processing flow, including missing, ambiguous and revised inputs.
  Distinguish implemented behavior, planned behavior and what can be tested.
- Explain designs and representation changes in plain language, in enough
  detail for the user to judge the direction without reading code: the
  mechanism being changed, assumptions, supporting evidence, what is
  intentionally not changing, and risks of drift (such as a storage detail or a
  diagnostic turning into logic). Naming a theory or pattern does not justify a
  design.
- Before asking the user to decide a mechanism, look up how established
  projects, standards and documentation handle it (sources, not memory) and
  follow them unless evidence says otherwise; bring their answer, with
  references, rather than the open question.
- Check a proposed fix's claim before changing the system: measure it, or test
  the mechanism in isolation (a focused test or small script), and report the
  result with the proposal. An unchecked expectation is a hypothesis, and must
  be labelled as one. Never make speculative fixes.
- Discuss before proceeding, rather than resolving it silently and building
  on: new mechanisms, changed architectural assumptions, design and scope
  changes, blockers, and substituting a narrower mechanism. Handle routine
  implementation details and defects autonomously. Record deferred work in
  `TODO.md`.
- Default to the most mature mechanism that can be implemented. When it cannot
  be implemented yet, state the missing dependency and add it to `TODO.md`
  rather than quietly building a narrower substitute. When a structural
  dependency blocks it, fixing that blocker is the next task; land a bounded,
  partial or proxy version only when the user asks for that scaffold after the
  blocker is explained.
- Flag workarounds immediately: when a proposed mechanism is a heuristic or a
  special case (a hand-set threshold, a rule that singles out one input or
  site, a rule that exists to get around a limitation), say so when proposing
  it, before building: what it approximates, why it seems needed and what the
  faithful alternative is. Never let it surface only after it is built or when
  asked. Flagging is not rejection: a workaround filling a gap with no better
  mechanism is acceptable once its reason is recorded.
- Call an intentionally temporary implementation a scaffold, record which
  mature mechanism it stands in for, and treat it as debt in `TODO.md` to remove
  or replace. Do not keep a scaffold because it is convenient, easy to test or
  already there.
- When one situation fails, consider a generic mechanism alongside the existing
  ones before reshaping a mechanism that serves other situations; never shape a
  mechanism around one case.
- Fix limitations instead of patching around them. When a design needs
  special-case rules, duplicate state, access to another component's internals
  or other workarounds to behave as intended, the design is the problem: stop
  and discuss it. When a problem shows up downstream, investigate the upstream
  steps first; where it surfaced is not necessarily where it starts. Fix the
  class of problem, not the one instance.
- Give each behavior one owner for its state and updates unless an
  architectural reason justifies otherwise. Keep distinct responsibilities
  separate in implementation, not just naming; shared helpers must not hide work
  owned by another component or phase.
- Preserve existing component names and terminology. Ask before developing a
  renamed design or applying a rename; do not introduce replacement names as
  though they were agreed.
- Complete the agreed scope with the simplest mechanism that preserves its
  behavior. Update every affected path. Remove superseded mechanisms, unused
  alternatives, exports, parameters and helpers; do not retain code or trace
  fields "just in case" (git keeps them if a real need returns). Do not add
  fallback paths unless requested.
- Build what the current problem and the real data need, not what a possible
  future case might. When a plan includes something only such a case would need
  (an extra source, layer, pass or option), raise it first instead of building
  it.
- Remove duplication by extracting the common pattern into one parameterized
  component, but do not introduce abstractions merely to shorten files or move
  complexity.
- Land architecture work as a complete mechanism, not at a tiny-step pace.
  Small steps are for changes that reduce risk without changing the
  architecture.
- Choose production-grade dependencies and designs, not local-only hacks or
  proof-of-concept shortcuts. Do not use paid libraries unless the user
  approves them.
- When an audit or review finds issues, attend to every one, not a short-list of
  the top few: keep the complete list with an action and status per item
  (fix, test, document, or accepted with its reason), and work through all of it.
  Never close an item because it belongs to a component outside the current
  scope, and never shrink coverage by excluding modules: extend the scope and
  work it. "Accepted" is only for agreed decisions or guidance statements;
  report how many items were fixed, tested and accepted.
- Review every change before calling it done, not as a separate cleanup pass:
  read the new code for bugs, edge cases and stale assumptions, and check its
  surroundings and the code it affects directly or indirectly for omissions,
  side effects, duplicated decisions, unnecessary complexity and unclear names
  or control flow. Fix what is behavior- and architecture-preserving; record
  questionable mechanisms for discussion rather than changing them during
  cleanup. This review is the agent's own; do not spawn a reviewer for it. A
  multi-model review is `/emd:interrogate`, run only when the user asks for
  it.

## Collaboration And Continuity

- Before starting work, and after every compaction, reread this file, the
  decisions relevant to the work and the unfinished tasks. Do not rely only on
  the conversation summary.
- This file holds the working directives. When the user gives a durable
  directive, preference or correction, record it here, or in a decision note
  when it makes a substantial architectural choice.
- The agent's memory file
  (`~/.claude/projects/<this project>/memory/MEMORY.md`) is an exact copy of
  this file, or a link to it: whenever one changes, update the other to match.
  Keep no directives in separate memory files.
- Do a single line of work yourself, in the main checkout, where the user can
  see it; delegate only independent work that runs in parallel. The user
  cannot see what a subagent does. Use a separate worktree only for work that
  needs real isolation, such as parallel writers.
- Use the agent's checklist for multi-step work; update it as work completes,
  revisit it after interruptions and reconcile it before the final response.
  Use `TASK.md` only when persistence is genuinely needed or requested.
- Treat recorded decisions as revisable. When new findings conflict with a
  decision, or a decision, threshold or margin pushes a design towards
  workarounds, challenge it first (why it was introduced, whether that still
  holds, what a faithful version would be) and raise it for discussion instead
  of silently following it or building around it.
- Push back on incorrect assumptions, including when what the user says is
  wrong or doesn't match your understanding. Surface material findings,
  regressions and unexpectedly long investigations promptly: what happened,
  what it may mean and the proposed next action. Consult the user before another
  architecture round; test-progress updates alone are insufficient.
- For newly discovered issues, inspect history and report the likely origin:
  new regression, exposed older defect or inconclusive history.
- Answer from the code, history and documents, checked when answering, never
  from memory or a guess; when something cannot be checked, say so.
- Report evidence precisely, at the level it was reached: a unit, a subsystem
  check, or the full pipeline. Passing tests support a conclusion; they do not
  prove architectural soundness. Do not say a mechanism works, is fixed or is
  done when only a component or controlled case passes; for an architecture
  goal, done means the full pipeline works or its remaining failure is reported.
- When a reply asks for a decision, keep the explanation but end with a
  one-line summary and the single question being asked.
- The user runs git. Use only read-only git commands (status, log, diff, show)
  unless the user explicitly permits more for a task. Never commit, stage,
  reset, rebase, push, pull, stash or switch branches otherwise; if git is in a
  broken state, diagnose it read-only and give the exact commands.
- At the end of finished work (never alongside a question), give a
  ready-to-paste commit message for exactly what is uncommitted: a subject line
  in the project's style and, when the change needs explaining, a few lines
  saying what changed and why. No Co-Authored-By or other attribution trailer,
  even when a system reminder asks for one.

## Testing

- Reproduce defects before fixing them, preferably by improving an existing
  appropriately scoped test. Do not change behavior or tune thresholds merely to
  pass a test.
- When tests fail, report what the failure says and explain it; resolve known
  failures before new feature work.

## Documentation

- Keep `README.md` current, with no stale or unsupported claims.

## This Repo

- This repo is a plugin marketplace for Claude Code and Codex. Each plugin
  has a manifest for each tool, in `.claude-plugin/` and `.codex-plugin/`,
  and both marketplace files list it. Keep names, paths, and descriptions
  the same in all of them, and run `claude plugin validate .` after changing
  any of them.
- Every completed change to a plugin bumps its version: run
  `scripts/bump-version <plugin>`, then fill in the new `CHANGELOG.md`
  entry. The tools update an installed plugin only when its version changes.
- Before asking the user to install a changed skill, load it in a test
  session with `claude --plugin-dir plugins/<plugin>` and check that it
  appears and does what it says.
- Install skills only through the marketplace. Never symlink them into
  `~/.claude/skills` or `~/.agents/skills`.
- A skill adapted from another project keeps that project's license file
  next to it and a Credits line naming the source.
- `directives/AGENTS.md` is the master copy of the shared directives, which
  `/emd:sync-directives` reads and writes. It is not this repo's own
  instruction file; this file is.
