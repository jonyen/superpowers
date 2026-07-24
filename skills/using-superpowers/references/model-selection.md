# Model Selection for Subagent Dispatch

Use the least powerful model that can handle each role to conserve cost and
increase speed.

**Always specify the model explicitly when dispatching a subagent.** An
omitted model inherits your session's model — often the most capable and
most expensive — which silently defeats this guidance.

## Tiers

Tiers are defined generically so they apply on any harness:

- **Cheap/fast** — mechanical transcription, single-file fixes, tasks whose
  plan text contains the complete code to write.
- **Standard** — implementers working from prose descriptions, integration
  across multiple files, typical reviews, debugging.
- **Most capable** — architecture and design tasks, subtle or risky diffs,
  the final whole-branch review.
- **Frontier** (when available) — fix-loop escalation above a stuck
  most-capable implementer; the hardest design and review judgment calls.

## Harness mapping

| Tier | Claude Code | Other harnesses |
|---|---|---|
| Cheap/fast | Haiku | smallest current model |
| Standard | Sonnet | mid-tier model |
| Most capable | Opus | flagship model |
| Frontier | Fable | frontier tier, if the platform offers one |

Model lineups change. Prefer the current generation of whichever model fills
each tier, not a pinned version.

## Unmapped models

When the available models don't appear in the mapping above:

- **Map by relative position, not name.** Rank what the harness offers by
  capability (vendor tier naming, price, context size). The cheapest fills
  the cheap/fast tier, the strongest fills most-capable/frontier, and
  whatever sits between them is standard.
- **Fewer models than tiers: round up.** With only two models, cheap/fast
  takes the smaller and every other tier takes the larger. Turn count beats
  token price — a too-weak model costs more than a too-strong one.
- **One model only: still specify it.** Pinning it keeps dispatches
  deterministic when the lineup later grows; the tier rules then govern how
  much you hand each subagent, not which model runs it.
- **Never invent a model ID.** Use only names the harness's dispatch tool
  documents as valid. If a chosen name is rejected, fall back to explicitly
  specifying the session's model — an explicit fallback is visible and
  fixable; a silently omitted field is neither.

## Turn count beats token price

Wall-clock and context cost scale with how many turns a subagent takes, and
the cheapest models routinely take 2-3× the turns on multi-step work —
costing more overall. Use a mid-tier model as the floor for reviewers and
for implementers working from prose descriptions. When the task's plan text
contains the complete code to write, the implementation is transcription
plus testing: use the cheapest tier for that implementer. Single-file
mechanical fixes also take the cheapest tier.

## Review tasks

Choose the reviewer's model with the same judgment, scaled to the diff's
size, complexity, and risk. A small mechanical diff does not need the most
capable model; a subtle concurrency change does. Scoped re-reviews of small
fix diffs take a cheap-to-mid tier.

## Role defaults

| Role | Default tier |
|---|---|
| Implementer (complete code in plan) | Cheap/fast |
| Implementer (prose spec) | Standard |
| Reviewer (small mechanical diff) | Cheap–standard |
| Reviewer (typical diff) | Standard |
| Reviewer (subtle or risky diff) | Most capable |
| Scoped re-review of a small fix diff | Cheap–standard |
| Parallel investigator | Cheap–standard |
| Final whole-branch review | Most capable |
| Architecture / design | Most capable |
| Fix-loop escalation (rounds 4-5) | One tier above the stuck implementer |
