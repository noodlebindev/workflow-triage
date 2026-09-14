# workflow-triage — Reference

Deep material for the triage skill. Load when `SKILL.md` alone is not enough to resolve a
case. Naming here matches `SKILL.md` exactly: the six verdicts (`inline`, `few-subagents`,
`dynamic-workflow`, `loop`, `hybrid`, `do-not-automate`), the five dimensions, and the
nine anti-signal gates.

---

## 1. Scoring detail (0-vs-5 anchors)

Score each dimension 0–5. Anchors:

**Breadth** — *Are there 5+ genuinely independent units?*
- 0 = one indivisible unit.
- 5 = 5+ units with no shared state; any one could be processed alone.

**Stage depth** — *Does each unit pass through repeatable stages?*
- 0 = single action per unit, no pipeline.
- 5 = each unit runs the same multi-stage pipeline (e.g. gather → extract → verify → score).

**Verification need** — *Would adversarial review materially improve quality?*
- 0 = output is trivially checkable or low-stakes.
- 5 = plausible-but-wrong output is likely and costly; independent skeptics would catch it.

**External-state dependency** — *Is Claude blocked waiting on someone/something else to
act (not just still working)?*
- 0 = all inputs available now; finishes in one pass, or Claude can keep working itself.
- 5 = blocked on state that changes over time via someone/something else (CI, deploy,
  queue, webhook, prices, inbox) — nothing to do meanwhile but check back.

Note: work that can keep actively continuing toward a checkable end-state (e.g. "until
tests pass") scores low here even across many turns — see §3's loop-vs-continue-until-done
tie-break.

**Human judgment sensitivity** — *Are there decisions that should stay with the human?*
- 0 = mechanical, reversible, no taste or risk.
- 5 = the decision *is* the deliverable — financial, strategic, irreversible, or taste-led.

The scores **inform**. They never override a tripped gate (§2).

---

## 2. Gate reasoning

Why each gate exists, and the generic category it protects. A gate fires regardless of how
high the rubric scores.

- **Single-file / small scope** — protects against fan-out where there is nothing to fan
  out to. Coordination overhead exceeds any parallelism gain. Generic category: *trivial
  scope*.
- **Mostly sequential** — protects against parallelising work whose steps depend on each
  other's output. A workflow's breadth is wasted if item N needs item N-1. Generic
  category: *true dependency chain*.
- **Unclear goal / undefined "done"** — protects against orchestrating toward a target that
  is not defined. With no "done", there is no stop rule and no way to judge output. The real
  next step is scoping, not execution. Generic category: *undefined scope*.
- **Production mutation risk** — protects live state. Autonomous fan-out that *writes* to
  production can cause irreversible damage at parallel scale. Generic category: *irreversible
  production-state change*. Distinct from ordinary concurrent-edit collision in a repo (two
  workers touching the same file) — that risk is handled by per-unit worktree isolation as
  a shape detail (§5), not by this gate; this gate is about writing to a live external
  system, which isolation doesn't make safe.
- **Bundled unrelated deliverables** — protects against forcing several independent asks
  into one verdict. The decision procedure assumes a single chunk of work; scoring a bundle
  as if it were one unit produces a verdict that fits none of the pieces well. Generic
  category: *multiple unrelated deliverables presented as one request*.
- **Financial decision-making** — protects money decisions. Research can be parallelised and
  verified; the decision to commit capital must stay human. Generic category: *financial
  decision-making where research can inform but not decide*.
- **Strategy / taste decision** — protects judgment calls with no verifiable right answer
  (positioning, naming, brand, direction). Generic category: *strategy/taste decision*.
- **Exploratory / unbounded scope** — protects against open-ended exploration with no
  bounded unit set; cannot define a stop rule, so a workflow runs without a finish line.
  Generic category: *unbounded exploration*.
- **Orchestration-to-dodge-a-decision** — the tell: heavy machinery proposed where the real
  blocker is a single hard human call. Building a workflow to avoid deciding is the failure
  this skill exists to catch. Generic category: *avoidance via tooling*.

---

## 3. Tie-break logic

When two verdicts both look plausible, apply these rules. Default to the lighter mode; the
heavier mode carries the burden of proof.

**`inline` vs `few-subagents`** — Use `inline` when one perspective and one pass suffice.
Use `few-subagents` only when 2–4 *distinct* angles each benefit from isolation (a copy
reviewer should not be biased by the SEO reviewer's notes), and the results are merged once.
If it is one angle on one artifact, it is `inline`.

**`few-subagents` vs `dynamic-workflow`** — Use `few-subagents` for a bounded, single-merge
review of *one* artifact from a handful of angles, run inline in the current turn. Escalate
to `dynamic-workflow` when there are 5+ independent *units*, each unit runs a repeatable
multi-stage pipeline, **or** the work needs to run unattended in the background and survive
an interruption — even under 5 units, if it's long enough that babysitting it inline would
tie up the session. The line: angles-on-one-thing, done in one turn → few-subagents;
same-pipeline-over-many-things, or anything that should run in the background and resume if
interrupted → workflow (§5).

**`dynamic-workflow` vs `loop`** — Use `dynamic-workflow` when all units exist *now* and you
fan out across them in one shot. Use `loop` when the work is blocked on external state that
changes *over time* and must wake up later. Breadth points to workflow; external-state
dependency points to loop. If both are present, see hybrid. See also
loop-vs-continue-until-done below for the case that looks like waiting but is actually still
workable.

**`loop` vs. continue-until-done** — `loop` is for work genuinely blocked on
someone/something *else* acting, with nothing useful to do between checks (CI finishing, a
deploy landing, a price crossing a threshold). If Claude can keep actively working every
turn and the finish line is a checkable condition ("tests pass," "every call site
migrated"), that is not `loop` — it's ordinary work (inline, or a dynamic-workflow
repeat-until-condition pattern for fanned-out units) with a completion condition attached,
not a time-based wait. Recommending a time-interval poll for work that could just keep
grinding wastes cycles and delays completion; recommending "keep working" for something
genuinely blocked on an external actor burns turns for nothing. Ask: *is there anything
Claude can do right now, or is it purely waiting on someone/something else?*

**`hybrid` detection** — Reach for `hybrid` when one mode must *safely unlock* another.
Common shapes:
- scope-lock (human/brainstorm) → then `dynamic-workflow`
- manual reality-check (e.g. confirm an API actually behaves as assumed) → then fan-out
- read-only analysis `dynamic-workflow` → human approval → controlled execution
- `loop` until external state changes → then `dynamic-workflow` or a summary

`hybrid` is frequently the *more restrained* choice: it inserts a human gate before any
fan-out, converting a case that would otherwise trip a gate (undefined "done", production
mutation) into a safe sequence. It is not "the heaviest mode" — it is the *sequenced* mode.

**`do-not-automate` as protection verdict** — This is reached via a gate, never chosen off
the execution ladder. When the *decision itself* is the deliverable, return
`do-not-automate`. Crucially, separate the two halves of such tasks:
- the *research that informs* the decision — may well be a `dynamic-workflow`
- the *decision* — stays human, `do-not-automate`, with a mandatory human checkpoint

---

## 4. Worked eval examples

Each example: input → filled scorecard → gate (if any) → verdict → execution shape.

### Should return `inline`

**"Fix this one TypeScript error."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | One error, one file/context. |
| Stage depth | 0 | Single edit. |
| Verification need | 1 | Compiler verifies it. |
| External-state dependency | 0 | All available now. |
| Human judgment sensitivity | 1 | Mechanical. |

Gate: **Single-file / small scope** → caps at `inline`. Verdict: **`inline`**. Shape: read
the error, fix in place, recompile.

**"Review this one file."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | One artifact. |
| Stage depth | 0 | One pass. |
| Verification need | 1 | Low stakes. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 1 | — |

Gate: **Single-file / small scope** → `inline`. Verdict: **`inline`**. Shape: one focused
read-through with notes. (Only escalate to `few-subagents` if the user explicitly wants
multiple *distinct* angles on it.)

**"Improve this single CTA."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | One element. |
| Stage depth | 0 | One edit. |
| Verification need | 1 | — |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 2 | Minor copy taste, not strategic. |

Gate: **Single-file / small scope** → `inline`. Verdict: **`inline`**. Boundary note: a
*single* CTA tweak is small-scope, not a strategy/taste decision — so it stays `inline`.
Deciding the brand's overall positioning *would* trip the strategy/taste gate (see §4
do-not-automate).

### Should return `few-subagents`

**"Review this landing page from copy, UX, and SEO angles."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 2 | One page, three angles — not independent units. |
| Stage depth | 1 | Each angle is a single review pass. |
| Verification need | 3 | Distinct lenses catch what one misses. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 2 | Findings inform; user still decides changes. |

No gate. Verdict: **`few-subagents`**. Shape: three parallel reviewers (copy / UX / SEO),
each isolated to avoid bias, merged into one findings list once.

**"Critique this product idea from market, technical, and positioning angles."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 2 | One idea, three angles. |
| Stage depth | 1 | Single critique pass per angle. |
| Verification need | 3 | Independent angles surface blind spots. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 3 | Critique *informs*; it does not *decide*. |

No gate. Verdict: **`few-subagents`**. Boundary note: critiquing positioning to *inform* is
fine to fan out; *deciding* the final positioning would trip the strategy/taste gate and
become `do-not-automate`. The deliverable here is analysis, not the decision.

### Should return `dynamic-workflow`

**"Audit every skill in a skills repo against the same checklist."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 5 | Many independent skills. |
| Stage depth | 4 | Each: read → check vs checklist → report. |
| Verification need | 2 | Checklist is objective. |
| External-state dependency | 0 | All present now. |
| Human judgment sensitivity | 1 | Mechanical scoring. |

No gate. Verdict: **`dynamic-workflow`**. Shape: one agent per skill running the same
checklist pipeline → collected pass/fail matrix.

**"Review 40 landing pages for SEO, conversion copy, accessibility, and broken links."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 5 | 40 independent pages. |
| Stage depth | 4 | Per page: 4 review dimensions. |
| Verification need | 3 | Confirm findings before reporting. |
| External-state dependency | 0 | Pages available now. |
| Human judgment sensitivity | 1 | Read-only review, no mutation. |

No gate (read-only — no production mutation). Verdict: **`dynamic-workflow`**. Shape:
pipeline over 40 pages, each through the four dimensions, findings verified per page.

**"Research 20 stocks and adversarially verify every bullish claim before adding anything to a watchlist."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 5 | 20 independent tickers. |
| Stage depth | 4 | Per stock: gather → extract claims → adversarially verify → score. |
| Verification need | 5 | Adversarial refutation is the whole point. |
| External-state dependency | 1 | One-shot research; not waiting on state. |
| Human judgment sensitivity | 2 | Output is a *watchlist*, not a trade. |

No gate trips **for the research as scoped** — the deliverable is a verified watchlist, not
a capital commitment. Verdict: **`dynamic-workflow`**. Shape: pipeline over 20 tickers,
each running gather → extract-claims → 3-skeptic adversarial verify (kill claim if a
majority refute) → score; survivors compiled into the watchlist.

> **Mandatory carve-out — the decision is NOT part of this verdict.**
> - The **research sweep** (gather + adversarially verify + build the watchlist) is the
>   `dynamic-workflow`.
> - The **investment decision itself** — whether to buy, and how much — is **`do-not-automate`**.
>   It trips the **Financial decision-making** gate. Research can *inform* it; it cannot
>   *decide* it.
> - A **human checkpoint is mandatory before any action**. The workflow stops at "here is the
>   verified watchlist with surviving claims." It must never advance to sizing, buying, or any
>   capital action. If the request bundles "…and then invest," split it: workflow for the
>   research, `do-not-automate` for the decision.

### Should return `loop`

**"Keep checking CI until the deploy passes, then summarize failures."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | One pipeline to watch. |
| Stage depth | 1 | Check, then summarize. |
| Verification need | 0 | Status is authoritative. |
| External-state dependency | 5 | Blocked on CI finishing over time. |
| Human judgment sensitivity | 0 | Mechanical. |

No gate. Verdict: **`loop`**. Shape: poll CI at a cadence matched to run length; on
terminal state, summarize failures once. (Not a workflow — there is nothing to fan out;
the work is *waiting*.)

**"Monitor a webhook/deploy/queue status until it changes."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | Single status. |
| Stage depth | 0 | Observe. |
| Verification need | 0 | — |
| External-state dependency | 5 | Entire task is waiting on change. |
| Human judgment sensitivity | 0 | — |

No gate. Verdict: **`loop`**. Shape: wake on an interval matched to how fast the state
changes; act once it flips.

### Should return `hybrid`

**"First confirm the API reality manually, then fan out implementation tasks."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 4 | Several independent implementation tasks. |
| Stage depth | 3 | Each task builds + verifies. |
| Verification need | 2 | Standard review. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 2 | The manual API check is a human gate. |

No gate (the manual check *is* the safeguard). Verdict: **`hybrid`**. Shape: human/manual
API reality-check first → on confirmation, `dynamic-workflow` across implementation tasks.
Restraint note: jumping straight to fan-out on *assumed* API behaviour risks 4 agents
building on a wrong premise; the manual gate makes the fan-out safe.

**"First lock product scope, then run a workflow across implementation workstreams."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 4 | Multiple workstreams. |
| Stage depth | 3 | Each workstream builds + verifies. |
| Verification need | 2 | — |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 3 | Scope is a human decision. |

No gate *because* scope-lock comes first. Verdict: **`hybrid`**. Shape: scope-lock
(brainstorm/human) → `dynamic-workflow` across workstreams. Restraint note: skip the
scope-lock and the **Unclear goal / undefined "done"** gate would trip — `hybrid` is the
disciplined path that resolves scope before fanning out.

### Should return `do-not-automate`

**"Decide whether to invest heavily in a stock."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 0 | One decision. |
| Stage depth | 0 | — |
| Verification need | 2 | Research can inform. |
| External-state dependency | 1 | — |
| Human judgment sensitivity | 5 | The decision *is* the deliverable. |

Gate: **Financial decision-making** → **`do-not-automate`**. Shape: research may be a
workflow, but the capital decision stays human. Mandatory human checkpoint before any
action.

**"Decide the final positioning of a SaaS."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 1 | One strategic call. |
| Stage depth | 0 | — |
| Verification need | 2 | Analysis can inform. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 5 | Taste/strategy, no verifiable right answer. |

Gate: **Strategy / taste decision** → **`do-not-automate`**. Shape: a `few-subagents`
critique can inform it (see above), but the final positioning is a human call.

**"Make production database changes without review."**

| Dimension | Score / 5 | Notes |
| --- | --: | --- |
| Breadth | 1 | Irrelevant — gate dominates. |
| Stage depth | 0 | — |
| Verification need | 4 | High — but the issue is the missing human gate. |
| External-state dependency | 0 | — |
| Human judgment sensitivity | 5 | Irreversible production-state change. |

Gate: **Production mutation risk** → **`do-not-automate`** (or, if the change is genuinely
needed, downgrade to a read-only analysis workflow + human apply — a `hybrid`). The phrase
"without review" is the tell: an irreversible production change with no human gate must not
be automated.

### Should split before triaging (not a single verdict)

**"Fix the login bug, write the Q3 report, and refactor the payments module."**

These are three independent deliverables with no shared scope, not one chunk of work with
5+ *units* inside it. Gate: **Bundled unrelated deliverables** → split first. Triage each
piece on its own (the bug fix is likely `inline`; the report may trip a judgment gate; the
refactor may be its own `dynamic-workflow` candidate), or note that unrelated independent
tasks are simpler to just dispatch as separate sessions than to force through one verdict.

---

## 5. Dynamic-workflow mechanism and prerequisites

What `dynamic-workflow` actually maps to, so the shape and cost estimate given are accurate
to what will really happen — and what a session-scoped `loop` requires to keep firing.

**Mechanism**: Claude writes a script (`agent()`, `pipeline()`, `parallel()`) that a
separate runtime executes — not the lead spawning subagents turn-by-turn inline. The run
happens in the background, so the session stays responsive. It is resumable: a stopped or
interrupted run picks up from completed work rather than starting over (an edit or a
failure upstream still reruns everything after it).

**How to invoke it**: say "use a workflow," or include the `ultracode` keyword. Requires a
prompt a human actually typed (or an equivalent human-originated route) — it does not fire
from `-p`, an Agent SDK call, a scheduled-task prompt, or a webhook/PR-comment relay. Name
this trigger in "Recommended Execution Shape" and "Next Step" rather than leaving the
mechanism implicit.

**Native verification pattern**: use this instead of hand-rolling "N subagents report back,
lead adjudicates" — a workflow phase can have independent agents adversarially review each
other's findings, or draft a plan from several angles and weigh them against each other,
before anything is reported. This is the concrete mechanism behind the "3-skeptic
adversarial verify" language in the stock-research worked example (§4).

**Hard caps and cost signals** — cite these instead of a bare Low/Medium/High:
- Up to 16 agents run concurrently; up to 4,096 items in a single `parallel()`/`pipeline()`
  call; 1,000 agents total per run.
- A size guideline steers how many agents Claude aims for: `small` (<5), `medium` (<15,
  default), `large` (<50), `unrestricted`. State which guideline the task calls for.
- A run is flagged "Large workflow" past 25 scheduled agents or a projected 1.5M tokens —
  cite this threshold in "Token / Complexity Estimate" rather than just an adjective.

**Prerequisites / off-switches** — state these when relevant instead of assuming
availability:
- Can be turned off session-wide or org-wide by settings; if it's off, or the prompt won't
  be human-typed (e.g. this triage's output feeds an automated pipeline), fall back to
  `few-subagents` at a smaller scope, or say so explicitly in "Human Checkpoints."

**`loop` prerequisite**: a session-scoped wait only fires while that session is running and
idle — closing the terminal stops it, though backgrounding the session carries it over. If
the task must survive the machine or session being off, say so in "Human Checkpoints"
rather than assuming the wait will simply keep going.

**Worktree isolation** (referenced from the Production mutation risk gate note in §2 and
from the Output template's "Recommended Execution Shape"): when a `dynamic-workflow` or
`few-subagents` fan-out has units that write files in the same repository, give each unit
its own isolated worktree so concurrent edits can't collide. This makes autonomous fan-out
writes *safe within a repo* without a human gate — it does not by itself satisfy the
Production mutation risk gate, which is about writes to a live external system, not
concurrent edits in a checkout.

---

## 6. Generic-lesson mapping

The core skill is generic and portable. Personal cases are illustrations only — each maps
to a generic lesson, and only the generic lesson belongs in the procedural sections.

| Personal case | Generic lesson |
| --- | --- |
| Bulk SEO sweep with unresolved strategy | Bulk optimisation work where strategic scope is unresolved → resolve scope first (`hybrid`) or `do-not-automate`. |
| "Go all-in" investing decision | Financial decision-making where research can inform but not decide → research may be `dynamic-workflow`; the decision is `do-not-automate`. |
| Production DB mutation | Irreversible production-state change → blocked from autonomous fan-out; read-only + human apply or `do-not-automate`. |
| Multi-campaign PPC analysis | Multi-entity analytics workflow with human approval before budget/action changes → analysis is `dynamic-workflow`; budget/action change needs a human checkpoint. |
