# Contributing

Thanks for thinking about contributing — this skill is better because of it.

## Quick links

- [Code of Conduct](CODE_OF_CONDUCT.md)
- [Security policy](SECURITY.md)
- [Issue tracker](../../issues)

## Ways to contribute

- **Try the skill and report what breaks.** The most useful contribution is a bug report
  with the exact prompt you used, the skill's verdict, and what you expected instead — file
  it as a bug-report issue. Triage disagreements (it said `dynamic-workflow`, you'd have
  done it `inline`) are especially valuable.
- **Improve the SKILL.md instructions.** Small clarifications, fixed typos, better wording
  for a gate or tie-break — open a PR.
- **Add or sharpen worked examples.** The adjudicated examples with filled scorecards in
  `REFERENCE.md` are this skill's regression tests. A real-world case that the current
  wording gets wrong is a great addition — add it with its expected verdict and reasoning.
- **Suggest new behaviour.** Open a feature-request issue and propose the change before
  writing a PR — for behavioural changes I prefer to align on the approach first.

## Development setup

This is a Claude Code skill — `SKILL.md` plus `REFERENCE.md`. To work on it locally:

```bash
git clone <repo-url>
cd workflow-triage
ln -s "$(pwd)" ~/.claude/skills/workflow-triage   # symlink into your skills dir
```

Then in Claude Code, invoke the skill on real tasks and iterate on the wording.

## Pull request process

1. Fork the repo and create a branch from `main`.
2. Make your change. Keep PRs focused — one logical change per PR.
3. If you changed triage behaviour, add or update a worked example in `REFERENCE.md` so the
   change is pinned against regression.
4. Update `CHANGELOG.md` under `[Unreleased]`.
5. Open the PR using the template. Link the issue it closes.

## Design principle to preserve

This skill exists to choose the **lightest credible execution mode** — its value is
restraint, especially knowing when *not* to orchestrate. Please don't add behaviour that
biases it toward heavier modes, proactive scanning, or generating/launching workflows; it
is an assessment gate, not an execution tool. Changes that erode that are likely to be
declined.

## Style

- `SKILL.md`: clear imperative voice, short paragraphs, explain *why* alongside *what*.
  Avoid heavy MUSTs unless the behaviour is genuinely non-negotiable.
- Commit messages: conventional commits (`feat:`, `fix:`, `docs:`, `chore:`) preferred but
  not required.

## Code of Conduct

By contributing, you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).
