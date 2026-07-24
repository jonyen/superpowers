# Shared Model-Selection Reference — Design

**Date:** 2026-07-23
**Status:** Approved

## Problem

Guidance for choosing a cheaper/faster model when dispatching subagents exists
only in `subagent-driven-development`. The other dispatching skills
(`requesting-code-review`, `dispatching-parallel-agents`, `executing-plans`)
say nothing about model choice, so their subagents silently inherit the
session's model — usually the most capable and most expensive one.

## Goal

One canonical, harness-portable model-selection reference that every
dispatching skill points to, so any skill that spawns a subagent picks the
least powerful model tier that can handle the role — and always specifies it
explicitly.

## Design

### New file: `skills/using-superpowers/references/model-selection.md`

Single source of truth, derived from the existing Model Selection section in
`subagent-driven-development/SKILL.md`. Contents:

1. **Core principle.** Use the least powerful model tier that can handle the
   role, to conserve cost and increase speed. An omitted model inherits the
   session's model — often the most expensive — so **always specify the model
   explicitly when dispatching a subagent.**
2. **Four tiers, defined generically** (portable across Claude Code, Codex,
   Gemini, Pi):
   - **Cheap/fast** — mechanical transcription, single-file fixes, tasks
     whose plan text contains the complete code to write.
   - **Standard** — implementers working from prose descriptions,
     integration across multiple files, typical reviews, debugging.
   - **Most capable** — architecture/design tasks, subtle or risky diffs,
     the final whole-branch review.
   - **Frontier** (when available) — fix-loop escalation above a stuck
     most-capable implementer; the hardest design/review judgment calls.
3. **Per-harness mapping table.** Claude Code: Haiku → Sonnet → Opus →
   Fable (frontier). Note that lineups change: prefer the current generation
   of each tier. Other harnesses: use the platform's equivalent tiers rather
   than pinned names that would go stale.
4. **Unmapped models.** Rules for harnesses whose models the table doesn't
   cover: rank by relative position rather than name; round *up* when there
   are fewer models than tiers (a too-weak model costs more than a
   too-strong one); still pin the model when the harness offers only one;
   and never invent a model ID — if a name is rejected, fall back to
   explicitly naming the session's model, so the field is never silently
   omitted.
5. **Turn count beats token price** (kept verbatim in spirit from the
   existing section): the cheapest models routinely take 2–3× the turns on
   multi-step work and can cost more overall. Mid-tier is the floor for
   reviewers and for implementers working from prose.
6. **Role-based quick table** mapping default tiers:
   | Role | Default tier |
   |---|---|
   | Implementer (complete code in plan) | Cheap |
   | Implementer (prose spec) | Standard |
   | Reviewer (small mechanical diff) | Cheap–standard |
   | Reviewer (typical diff) | Standard |
   | Reviewer (subtle/risky diff) | Most capable |
   | Scoped re-review of a small fix diff | Cheap–standard |
   | Parallel investigator | Cheap–standard |
   | Final whole-branch review | Most capable |
   | Architecture / design | Most capable |
   | Fix-loop escalation (rounds 4–5) | One tier above the stuck implementer |

### Per-skill changes

- **`subagent-driven-development/SKILL.md`** — shrink the ~40-line Model
  Selection section to a few lines: the always-specify rule, the
  task-complexity signals (1–2 files with complete spec → cheap; multi-file
  integration → standard; design judgment → most capable), and a pointer to
  `../using-superpowers/references/model-selection.md`. The prompt templates
  (`implementer-prompt.md`, `task-reviewer-prompt.md`, `re-review-prompt.md`)
  keep their existing `[MODEL]` placeholders unchanged.
- **`requesting-code-review/SKILL.md`** — add a short Model Selection note at
  the dispatch step: pick the reviewer's model per the reference, scaled to
  the diff's size, complexity, and risk; specify it explicitly.
- **`dispatching-parallel-agents/SKILL.md`** — add a short note: parallel
  investigators are usually scoped, independent tasks, so default to
  cheap/standard tier per the reference; specify the model explicitly on
  every dispatch in the batch.
- **`executing-plans/SKILL.md`** — one sentence: if dispatching any subagent,
  consult the reference.

### Out of scope

- No hooks, scripts, or plugin-manifest changes — this is a pure
  documentation/skill change; skills are prose contracts and the harnesses'
  dispatch tools already accept an explicit model parameter.
- No changes to the main session's own model (downshifting the top-level
  agent is not part of this work).
- No upstream PR to obra/superpowers as part of this work (can follow later).

## Verification

- Run the repo's existing test suite from `package.json`.
- Grep checks: no skill retains a full inline copy of the tier rules; each
  dispatching skill references the new file; relative paths resolve.
- Work lands on the `model-selection-reference` branch of the
  jonyen/superpowers fork.
