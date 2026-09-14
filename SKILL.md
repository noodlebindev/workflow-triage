---
name: workflow-triage
description: Manual-invoke orchestration gate. Use ONLY when the user explicitly asks how a task should be executed — e.g. "triage this", "should this be a workflow?", "inline or subagents?", "how should I run this?". Returns a structured verdict (inline / few-subagents / dynamic-workflow / loop / hybrid / do-not-automate) plus execution shape — never a script, never a launch. Does NOT scan proactively or run on a loop.
---

# Workflow Triage

## Purpose

This is an **assessment gate, not an execution tool**. You hand it a chunk of work; it
returns a structured verdict on *how* that work should be executed — and stops there.

Its value is choosing the **lightest credible execution mode**, especially knowing when
*not* to orchestrate. A triage that recommends heavy machinery most of the time is worse
than no triage: it burns tokens and removes human judgment from work that needed it. Bias
toward the lighter mode; make the heavier mode earn its place.

## When to invoke

**Invoke** only when the user explicitly asks to triage a specific, named chunk of work
("triage this", "should this be a workflow?", "inline or subagents?").

**Do NOT:**
- Volunteer this skill or suggest it unprompted.
- Scan projects hunting for workflow opportunities. If asked "what could be a workflow?",
  decline to scan and ask the user to name the specific task to triage.
- Run on a schedule or loop.
- Generate a runnable script or launch anything (see **Hard boundaries**).

## Decision procedure

Reach the verdict in two ordered steps. **Gates first.**

1. **Apply the hard gates** (below). If any gate trips, return the gate-appropriate
   verdict — usually `do-not-automate`, `inline`, or `hybrid`. Gates are resolved *before*
   any execution-mode comparison.
2. **If no gate trips, score the five dimensions and choose the lightest credible mode**
   from the ladder: `inline → few-subagents → loop / dynamic-workflow → hybrid`.

## The six verdicts

- **`inline`** — small, sequential, low-risk work; one context handles it.
- **`few-subagents`** — small multi-perspective pass (2–4 angles), merged once.
- **`dynamic-workflow`** — breadth (5+ independent units), repeatable per-unit stages,
  deterministic orchestration, or enforced verification — or work that must run unattended
  and survive interruption regardless of count. Maps to Claude Code's actual background,
  resumable Dynamic Workflows feature (not the lead manually spawning subagents inline);
  see REFERENCE.md §5 for the mechanism, triggers, and prerequisites.
- **`loop`** — task is blocked **waiting on something else to change**, with nothing
  productive to do meanwhile; it must wake up later (CI, deploy, queue, webhook, prices,
  inbox). Not the same as work that can keep actively continuing toward a checkable
  end-state — see REFERENCE.md §3's loop-vs-continue-until-done tie-break.
- **`hybrid`** — a **sequenced** mode: one mode must safely unlock another. *Not* inherently
  the heaviest — often the *safer* choice because it inserts a human gate before fan-out.
  (e.g. scope-lock → workflow; manual reality-check → fan-out; read-only analysis → human
  approval → controlled execution; loop until state changes → then workflow or summary.)
- **`do-not-automate`** — a **protection verdict**, not an execution mode. Decision-heavy,
  risky, production-sensitive, financial, or taste-led work where the human judgment *is*
  the work. "Stop and decide."

## Five-dimension rubric (scores inform, gates decide)

Score each 0–5.

| # | Dimension | Question |
| --- | --- | --- |
| 1 | Breadth | Are there 5+ genuinely independent units? |
| 2 | Stage depth | Does each unit pass through repeatable stages? |
| 3 | Verification need | Would adversarial review materially improve quality? |
| 4 | External-state dependency | Is Claude blocked *waiting on someone/something else* to act (not just still working)? |
| 5 | Human judgment sensitivity | Are there decisions that should stay with the human? |

Note: "keep working across turns until a checkable condition holds" (e.g. "until tests
pass") scores low on dimension 4 even if it runs many turns — that's active work with a
stop condition, not waiting on an external actor. See REFERENCE.md §3 for the
`loop`-vs-continue-until-done tie-break.

## Anti-signal gates

**Gates override scores.** A high breadth score does not earn a workflow if a gate trips.
Each gate: **name → what it caps/forces → why.**

- **Bundled unrelated deliverables** → forces a split before triaging → the procedure
  assumes one chunk of work; scoring several independent asks as a single unit produces a
  verdict that fits none of the pieces.
- **Single-file / small scope** → caps at `inline` (or `few-subagents` for review) → fan-out buys nothing.
- **Mostly sequential** → blocks `dynamic-workflow` → no independent units to parallelise.
- **Unclear goal / undefined "done"** → forces `do-not-automate` → scope must be resolved first (real next step is often brainstorm/scope).
- **Production mutation risk** → blocks autonomous writing fan-out → downgrade to read-only workflow + human apply, or `do-not-automate`.
- **Financial decision-making** → forces `do-not-automate` for the decision → research may be a workflow, the decision stays human.
- **Strategy / taste decision** → forces `do-not-automate` → human judgment is the work.
- **Exploratory / unbounded scope** → blocks `dynamic-workflow` → no possible stop rule.
- **Orchestration-to-dodge-a-decision** → forces `do-not-automate` → make the hard call first.

When a gate trips, name it in "Why" and describe the lighter path (or the human decision
that must come first) in "Recommended Execution Shape".

## Gates override scores + restraint tie-break

- **Gates decide when tripped; scores only inform.** Resolve gates before comparing modes.
- **Burden of proof is on the heavier mode.** A workflow must be justified by a concrete
  breadth/stage/verification need a lighter mode cannot meet — not merely permitted by the
  scores.
- **Under Low confidence between a lighter and heavier mode, default to the lighter** and
  state in "Next Step" what evidence would justify escalating.
- **`do-not-automate` is not on the execution ladder.** It is a stop-and-decide outcome,
  reached via a gate — never chosen as "the lightest way to execute".
- **`hybrid` sits last in the ladder only because it composes other modes**, not because it
  costs most. Reach for it when a human gate makes fan-out safe that otherwise would not be.

## Output template

Always respond in exactly this structure:

```markdown
# Workflow Triage Verdict

## Verdict
[inline / few-subagents / dynamic-workflow / loop / hybrid / do-not-automate]

## Confidence
[High / Medium / Low]

## Why
* Reason 1
* Reason 2
* Reason 3

## Fit Scores
| Dimension                  | Score / 5 | Notes |
| -------------------------- | --------: | ----- |
| Breadth                    |           |       |
| Stage depth                |           |       |
| Verification need          |           |       |
| External-state dependency  |           |       |
| Human judgment sensitivity |           |       |

## Recommended Execution Shape
Plain-English structure only. No script yet. If units write files in the same repo, name
per-unit worktree isolation as the mechanism that keeps concurrent edits from colliding
(REFERENCE.md §5) — this is a shape detail, separate from the Production mutation risk
gate, which concerns writes to a live external system, not concurrent edits in a repo.

## Human Checkpoints
* What the user needs to decide
* What needs approval
* What risk needs review

## Token / Complexity Estimate
* Expected agent count
* Expected stages
* Cost level: Low / Medium / High
* Why the cost is or is not justified

## Stop Rule
When the system should stop instead of continuing.

## Next Step
The single next action.
```

## Hard boundaries

**This skill never:**
- Generates a runnable workflow script. Script generation is a *separate, later,
  user-triggered* step that happens only after a verdict is approved.
- Launches, spawns, or starts anything — no agents, no workflow, no loop.
- Scans projects proactively or hunts for opportunities.
- Runs on a schedule or loop.

If asked to "just build it too", return the verdict and execution shape only, and state
that script generation is the next, separate step the user can trigger.

## Reference

For scoring anchors (what 0 vs 5 looks like), gate reasoning, tie-break logic
(inline-vs-few-subagents, few-subagents-vs-workflow, workflow-vs-loop,
loop-vs-continue-until-done, hybrid detection, do-not-automate as protection), the
dynamic-workflow mechanism and prerequisites (§5), and worked eval examples with filled
scorecards, see `REFERENCE.md`.
