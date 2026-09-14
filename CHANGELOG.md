# Changelog

All notable changes to this skill will be documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] — 2026-09-14

### Changed
- `dynamic-workflow` verdict reanchored to Claude Code's actual background, resumable
  Dynamic Workflows feature (the `ultracode` / "use a workflow" mechanism) instead of
  implying the lead manually spawns and babysits subagents turn-by-turn.
- `few-subagents`-vs-`dynamic-workflow` tie-break now also escalates on need for background
  execution and resumability, not only the 5-unit breadth count.
- External-state-dependency dimension and the `loop` verdict reworded to distinguish
  "blocked waiting on someone/something else" from "can keep actively working toward a
  checkable end-state" — the latter was being miscategorized as `loop`.
- Production-mutation-risk gate reasoning now distinguishes writes to a live external
  system (what the gate protects) from concurrent-edit collision in a repo (a separate,
  worktree-solvable problem, not a reason to require a human gate).

### Added
- New anti-signal gate: **Bundled unrelated deliverables** — forces a split before triaging
  when several independent asks are presented as one request. Nine gates total.
- New `loop`-vs-continue-until-done tie-break in `REFERENCE.md`.
- New `REFERENCE.md` §5, "Dynamic-workflow mechanism and prerequisites": the real
  mechanism, invocation trigger and its human-typed-prompt requirement, the native
  adversarial-verify pattern, hard caps and size-guideline numbers (in place of vague
  Low/Medium/High), off-switches, the `/loop` session-liveness prerequisite, and the
  worktree-isolation mechanic for safe concurrent-write fan-outs.
- New worked example: "Should split before triaging (not a single verdict)."

## [0.1.0] — 2026-06-23

### Added
- Initial public release.
- Six-verdict triage: `inline`, `few-subagents`, `dynamic-workflow`, `loop`, `hybrid`,
  `do-not-automate`.
- Gates-override-scores decision procedure with eight anti-signal gates.
- Five-dimension scoring rubric and fixed output template.
- `REFERENCE.md` with scoring anchors, gate reasoning, tie-break logic, and worked examples.
