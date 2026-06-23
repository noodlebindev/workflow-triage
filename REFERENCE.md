# workflow-triage — Reference

Deep material for the triage skill. Load when `SKILL.md` alone is not enough to resolve a
case. Naming here matches `SKILL.md` exactly: the six verdicts (`inline`, `few-subagents`,
`dynamic-workflow`, `loop`, `hybrid`, `do-not-automate`), the five dimensions, and the
eight anti-signal gates.

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

**External-state dependency** — *Does the task need to wake up later?*
- 0 = all inputs available now; finishes in one pass.
- 5 = blocked on state that changes over time (CI, deploy, queue, webhook, prices, inbox).

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
  production-state change*.
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
review of *one* artifact from a handful of angles. Escalate to `dynamic-workflow` only when
there are 5+ independent *units* and/or each unit runs a repeatable multi-stage pipeline.
The line: angles-on-one-thing → few-subagents; same-pipeline-over-many-things → workflow.

**`dynamic-workflow` vs `loop`** — Use `dynamic-workflow` when all units exist *now* and you
fan out across them in one shot. Use `loop` when the work is blocked on external state that
changes *over time* and must wake up later. Breadth points to workflow; external-state
dependency points to loop. If both are present, see hybrid.

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

---

## 5. Generic-lesson mapping

The core skill is generic and portable. Personal cases are illustrations only — each maps
to a generic lesson, and only the generic lesson belongs in the procedural sections.

| Personal case | Generic lesson |
| --- | --- |
| Bulk SEO sweep with unresolved strategy | Bulk optimisation work where strategic scope is unresolved → resolve scope first (`hybrid`) or `do-not-automate`. |
| "Go all-in" investing decision | Financial decision-making where research can inform but not decide → research may be `dynamic-workflow`; the decision is `do-not-automate`. |
| Production DB mutation | Irreversible production-state change → blocked from autonomous fan-out; read-only + human apply or `do-not-automate`. |
| Multi-campaign PPC analysis | Multi-entity analytics workflow with human approval before budget/action changes → analysis is `dynamic-workflow`; budget/action change needs a human checkpoint. |
