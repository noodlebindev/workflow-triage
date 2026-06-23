# Security Policy

## Supported Versions

| Version | Supported |
| ------- | --------- |
| Latest release on `main` | ✅ |
| Older releases | ❌ |

## Reporting a Vulnerability

**Please do not file a public issue for security vulnerabilities.**

If you find a security issue in this skill — for example a prompt injection
that bypasses a documented safeguard, or wording that causes the skill to
recommend launching or executing something it should not — please report it
privately.

Two options:

1. **GitHub Private Vulnerability Reporting** (preferred) — open a private
   advisory through the repository's Security tab.
2. **Email** — noodlebin.dev@gmail.com

Please include:

- A clear description of the issue
- Steps to reproduce (a minimal example if possible)
- What you'd expect to happen vs. what actually happens
- Any suggested fix

## Response

I aim to acknowledge reports within 5 business days and to publish a fix or
mitigation within 30 days for confirmed issues. For critical issues affecting
many users, I will prioritise faster turnaround.

After a fix is released, I will coordinate disclosure with you and credit you
in the release notes unless you prefer to remain anonymous.

## Scope

This policy covers:

- The skill's SKILL.md instructions
- REFERENCE.md and any bundled guidance the skill writes or relies on

Out of scope:

- The underlying Claude Code or Anthropic API platforms — please report those
  directly to Anthropic.
- Third-party tools the skill recommends but does not bundle.
