# Shared Model-Selection Reference Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extract subagent model-selection rules into one canonical reference file that every dispatching skill points to, so subagents run on the least powerful model tier that can handle the role.

**Architecture:** A new reference file at `skills/using-superpowers/references/model-selection.md` (the repo's existing home for cross-skill reference files) holds tier definitions, a per-harness mapping (Claude Code: Haiku → Sonnet → Opus → Fable), and role defaults. The four dispatching skills each carry a short pointer instead of duplicated rules.

**Tech Stack:** Markdown only. No code, hooks, or manifest changes. Verification is via `grep` structural checks.

**Spec:** `docs/superpowers/specs/2026-07-23-model-selection-design.md`

## Global Constraints

- Documentation-only change: no hooks, scripts, or plugin-manifest edits.
- Preserve the repo's tuned voice ("your human partner", existing phrasing) — reuse existing sentences verbatim where they move; do not rewrite unrelated content.
- All cross-skill links use relative paths of the form `../using-superpowers/references/model-selection.md` (matching the existing per-platform reference links).
- Work lands on the `model-selection-reference` branch of the jonyen/superpowers fork. No upstream PR.
- Commit after every task.

---

### Task 1: Create the canonical reference file

**Files:**
- Create: `skills/using-superpowers/references/model-selection.md`

**Interfaces:**
- Produces: the file path `skills/using-superpowers/references/model-selection.md`, referenced by Tasks 2–5 as `../using-superpowers/references/model-selection.md`.

- [ ] **Step 1: Write the file with exactly this content**

````markdown
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
````

- [ ] **Step 2: Verify the file exists and links resolve**

Run: `ls skills/using-superpowers/references/model-selection.md && grep -c "Fable" skills/using-superpowers/references/model-selection.md`
Expected: the path printed, then `1`

- [ ] **Step 3: Commit**

```bash
git add skills/using-superpowers/references/model-selection.md
git commit -m "Add shared model-selection reference for subagent dispatch"
```

---

### Task 2: Shrink subagent-driven-development's Model Selection section to a pointer

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md:157-192` (the `## Model Selection` section, everything between the `## Model Selection` heading and the `## The Task Loop` heading)

**Interfaces:**
- Consumes: `../using-superpowers/references/model-selection.md` from Task 1.
- Produces: nothing used by later tasks.

- [ ] **Step 1: Replace the section body**

Replace the entire `## Model Selection` section (heading through the last
task-complexity bullet, currently lines 157–192) with exactly:

````markdown
## Model Selection

Choose each subagent's model per
[model-selection.md](../using-superpowers/references/model-selection.md).
**Always specify the model explicitly when dispatching** — an omitted model
inherits your session's model, often the most capable and most expensive.

**Task complexity signals (implementation tasks):**
- Touches 1-2 files with a complete spec → cheap model
- Touches multiple files with integration concerns → standard model
- Requires design judgment or broad codebase understanding → most capable model

The final whole-branch review takes the most capable available model, not
the session default. Fix-loop escalation (rounds 4-5) takes a model at
least one tier above the stuck implementer.
````

Do not touch the `[MODEL]` placeholders in `implementer-prompt.md`,
`task-reviewer-prompt.md`, or `re-review-prompt.md` — they already require
an explicit model.

- [ ] **Step 2: Verify the shrink**

Run: `awk '/^## Model Selection/,/^## The Task Loop/' skills/subagent-driven-development/SKILL.md | wc -l && grep -c "model-selection.md" skills/subagent-driven-development/SKILL.md`
Expected: first number ≤ 20 (was ~37), second number `1`

- [ ] **Step 3: Verify no orphaned rule text remains**

Run: `grep -c "Turn count beats token price" skills/subagent-driven-development/SKILL.md`
Expected: `0` (exit code 1 from grep is the pass signal)

- [ ] **Step 4: Commit**

```bash
git add skills/subagent-driven-development/SKILL.md
git commit -m "Point subagent-driven-development at shared model-selection reference"
```

---

### Task 3: Add model-selection note to requesting-code-review

**Files:**
- Modify: `skills/requesting-code-review/SKILL.md:32-40` (the "**2. Dispatch code reviewer subagent:**" step)

**Interfaces:**
- Consumes: `../using-superpowers/references/model-selection.md` from Task 1.

- [ ] **Step 1: Insert the note**

After the line:

```markdown
Dispatch a `general-purpose` subagent, filling the template at [code-reviewer.md](code-reviewer.md)
```

insert a blank line and then exactly:

```markdown
Choose the reviewer's model per
[model-selection.md](../using-superpowers/references/model-selection.md),
scaled to the diff's size, complexity, and risk — and specify it explicitly
in the dispatch.
```

- [ ] **Step 2: Verify**

Run: `grep -c "model-selection.md" skills/requesting-code-review/SKILL.md`
Expected: `1`

- [ ] **Step 3: Commit**

```bash
git add skills/requesting-code-review/SKILL.md
git commit -m "Add model-selection guidance to requesting-code-review"
```

---

### Task 4: Add model-selection note to dispatching-parallel-agents

**Files:**
- Modify: `skills/dispatching-parallel-agents/SKILL.md:66-77` (the "### 3. Dispatch in Parallel" section)

**Interfaces:**
- Consumes: `../using-superpowers/references/model-selection.md` from Task 1.

- [ ] **Step 1: Insert the note**

After the line:

```markdown
Multiple dispatch calls in one response = parallel execution. One per response = sequential.
```

insert a blank line and then exactly:

```markdown
Parallel investigators are usually scoped, independent tasks: default to a
cheap-to-standard tier per
[model-selection.md](../using-superpowers/references/model-selection.md),
and specify the model explicitly on every dispatch in the batch.
```

- [ ] **Step 2: Verify**

Run: `grep -c "model-selection.md" skills/dispatching-parallel-agents/SKILL.md`
Expected: `1`

- [ ] **Step 3: Commit**

```bash
git add skills/dispatching-parallel-agents/SKILL.md
git commit -m "Add model-selection guidance to dispatching-parallel-agents"
```

---

### Task 5: Add one-line mention to executing-plans

**Files:**
- Modify: `skills/executing-plans/SKILL.md:14` (the "**Note:**" paragraph in the Overview)

**Interfaces:**
- Consumes: `../using-superpowers/references/model-selection.md` from Task 1.

- [ ] **Step 1: Append the sentence**

At the end of the existing paragraph:

```markdown
**Note:** Tell your human partner that Superpowers works much better with access to subagents (Claude Code, Codex CLI, Codex App, Copilot CLI, and Gemini CLI all qualify; see the per-platform tool refs in `../using-superpowers/references/`). If subagents are available, use superpowers:subagent-driven-development instead of this skill.
```

append (same paragraph, after the final period):

```markdown
 If you do dispatch any subagent, choose its model per `../using-superpowers/references/model-selection.md` and specify it explicitly.
```

- [ ] **Step 2: Verify**

Run: `grep -c "model-selection.md" skills/executing-plans/SKILL.md`
Expected: `1`

- [ ] **Step 3: Commit**

```bash
git add skills/executing-plans/SKILL.md
git commit -m "Mention model-selection reference in executing-plans"
```

---

### Task 6: Whole-branch structural verification

**Files:**
- Test: none created — command-line checks only.

**Interfaces:**
- Consumes: all files from Tasks 1–5.

- [ ] **Step 1: Every dispatching skill references the file exactly once**

Run: `grep -rc "model-selection.md" skills/subagent-driven-development/SKILL.md skills/requesting-code-review/SKILL.md skills/dispatching-parallel-agents/SKILL.md skills/executing-plans/SKILL.md`
Expected: each of the four lines ends in `:1`

- [ ] **Step 2: No skill retains a full inline copy of the tier rules**

Run: `grep -rl "Turn count beats token price" skills/ | sort`
Expected: only `skills/using-superpowers/references/model-selection.md`

- [ ] **Step 3: Relative links resolve**

Run: `for f in skills/subagent-driven-development skills/requesting-code-review skills/dispatching-parallel-agents skills/executing-plans; do (cd "$f" && ls ../using-superpowers/references/model-selection.md >/dev/null) && echo "$f OK"; done`
Expected: four `... OK` lines

- [ ] **Step 4: Commit the plan checkboxes and finish**

```bash
git add docs/superpowers/plans/2026-07-24-model-selection-reference.md
git commit -m "Complete model-selection reference rollout"
```
