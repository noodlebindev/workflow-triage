# workflow-triage

A manual-invoke **orchestration gate** for AI coding agents. Before you spin up a
multi-agent workflow, it answers a prior question: *should this even be a workflow?*

`workflow-triage` returns a structured verdict on the **lightest credible execution mode**
for a task — and stops there. No scripts, no launches. Its whole value is restraint:
knowing when **not** to orchestrate.

## Why

Agentic harnesses make it cheap to fan out dozens of subagents. That's powerful for
genuinely broad work and wasteful — sometimes harmful — for everything else.
**Over-orchestration** burns tokens and removes human judgment from decisions that needed
it. **Under-orchestration** leaves big work cramped in a single context. This skill is the
gate that picks the right altitude, biased toward the lighter option.

## The six verdicts

| Verdict | When |
| --- | --- |
| `inline` | Small, sequential, low-risk; one context handles it. |
| `few-subagents` | Small multi-perspective pass (2–4 angles), merged once. |
| `dynamic-workflow` | Breadth (5+ independent units), repeatable per-unit stages, or enforced verification. One-shot fan-out *now*. |
| `loop` | Depends on **external state changing over time** (CI, deploy, queue, prices, inbox). |
| `hybrid` | A **sequenced** mode — one mode safely unlocks another (e.g. scope-lock → workflow). Often *safer*, not heavier. |
| `do-not-automate` | A **protection verdict** — decision-heavy, risky, financial, production-sensitive, or taste-led work where the human judgment *is* the work. |

## How it decides

**Gates first, then the lightest credible mode.**

1. **Apply hard gates.** If one trips, return the gate-appropriate verdict (usually
   `do-not-automate`, `inline`, or `hybrid`).
2. **If none trips,** score five dimensions and pick the lightest mode that fits:
   `inline → few-subagents → loop / dynamic-workflow → hybrid`.

**Gates override scores.** A high breadth score does *not* earn a workflow if the task
mutates production, is a financial decision, has an undefined "done", or is being
orchestrated to dodge a hard call. Scores inform; gates decide.

## Cheat sheet

```
DECISION ORDER
  1. gates  → if tripped, stop at the gate verdict
  2. else   → lightest credible mode on the ladder

LADDER (lightest → composed)
  inline → few-subagents → loop / dynamic-workflow → hybrid
  (do-not-automate is OFF the ladder — it's a protection verdict)

DIMENSIONS (0–5, inform only)
  breadth · stage-depth · verification-need · external-state · human-sensitivity

GATES (override scores)
  single-file · sequential · undefined-"done" · production-mutation
  financial-decision · strategy/taste · unbounded-scope · dodging-a-decision
```

## Output

The skill always responds in a fixed structure: **Verdict · Confidence · Why · Fit Scores
(table) · Recommended Execution Shape · Human Checkpoints · Token/Complexity Estimate ·
Stop Rule · Next Step.** Verdict and shape only — never a runnable script.

## Install

Copy this folder into your agent's skills directory:

```
cp -r workflow-triage ~/.claude/skills/
```

Then invoke it manually when scoping work — e.g. *"triage this"*, *"should this be a
workflow?"*, *"inline or subagents?"*. It does **not** trigger proactively, scan for
opportunities, or run on a schedule.

## Files

- **`SKILL.md`** — always-loaded core: procedure, verdicts, rubric, gates, output template.
- **`REFERENCE.md`** — on-demand depth: scoring anchors, gate reasoning, tie-break logic,
  and worked examples with filled scorecards.

## Status

Early release. Eval-validated against its designed case set; real-world dogfooding is
ongoing and improvements land gradually.

## License

[MIT](LICENSE).
