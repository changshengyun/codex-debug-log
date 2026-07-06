# Codex Debug Log

This repository stores Codex-related debug cases and retrospectives.

## Repository Structure

Each debug case uses one timestamped folder:

```text
debug-cases/
  YYYY-MM-DD_HHMM_short-error-cause/
    README.md
    professional-debug-report/
      report.md
    personal-retrospective/
      retrospective.md
```

The professional report records the technical investigation, evidence, root cause, fix, and rollback. The personal retrospective records the user's debug thinking, mistakes, turning points, and reusable lessons.

## Temporary Artifact Rule

When a task creates temporary backups, diagnostic logs, exported registry files, traces, or local proof files, remove them before finishing unless the user explicitly asks to keep them. If a backup must be kept for safety, mention the exact path and reason in the final response.

## Method Library

Reusable debug templates and thinking models live in:

```text
debug-methods/
```

## Skill Drafts

Draft Codex skills for future automation live in:

```text
skills-drafts/
```

These are repo-managed drafts. To make a skill automatically discoverable by Codex, copy or install the skill folder into the user's Codex skills directory.

## Cases

| Date | Case | Professional Report | Retrospective |
| --- | --- | --- | --- |
| 2026-07-06 | [Codex CLI device-auth TLS CA failure](debug-cases/2026-07-06_1059_codex-cli-device-auth-tls-ca-failure/README.md) | [report.md](debug-cases/2026-07-06_1059_codex-cli-device-auth-tls-ca-failure/professional-debug-report/report.md) | [retrospective.md](debug-cases/2026-07-06_1059_codex-cli-device-auth-tls-ca-failure/personal-retrospective/retrospective.md) |
