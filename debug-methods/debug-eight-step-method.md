# Debug Eight-Step Method

This is the reusable template for future Codex debug cases.

## Sources Used For This Template

This template borrows the incident-review shape from public postmortem practices:

- Atlassian's incident postmortem guidance emphasizes documenting impact, root cause, timeline, and follow-up actions.
- Google SRE's postmortem culture emphasizes learning-oriented, blameless analysis and actionable prevention.

This repository adapts those ideas for individual technical debugging: the center is not service outage management, but precise flow定位, evidence status, and personal skill improvement.

## 1. State The Problem

Capture the exact trigger and original error.

```markdown
- Command / operation:
- Original error:
- First observed:
- User impact:
```

## 2. Freeze The Environment

Record the facts that can affect reproducibility.

```markdown
- OS:
- Tool version:
- Network/proxy:
- Account/auth state:
- Relevant config:
- Environment variables:
```

## 3. Draw The Expected Flow

Describe what should happen if everything works.

```text
A -> B -> C -> D -> success
```

## 4. Draw The Actual Flow

Describe what actually happened.

```text
A -> B -> C -> stop
```

The key question:

```text
What is the first expected step that did not happen?
```

## 5. Locate The Breakpoint

Write the breakpoint as a narrow boundary:

```text
after <last confirmed step>
before <first missing step>
```

Example:

```text
after HTTP CONNECT 200
before TLS ClientHello
```

## 6. Build Four Tables

### Possible Failure Area Table

| Layer | Possible issue | Initial evidence | Priority |
|---|---|---|---|

### Suspect Flow Table

| Step | Expected behavior | Actual behavior | Evidence |
|---|---|---|---|

### Investigation Matrix

| Hypothesis | Test method | Expected result | Actual result | Conclusion |
|---|---|---|---|---|

### Evidence Status Table

| Claim | Status | Evidence | Notes |
|---|---|---|---|

Use statuses:

- `verified`
- `excluded`
- `supported`
- `unconfirmed`

## 7. Verify With Single-Variable Tests

Do not change several things at once.

Bad:

```text
upgrade version + change proxy + change CA + restart tool
```

Good:

```text
same version + same proxy + only change CA
```

## 8. Write The Final Record

Each final case must include:

- professional debug report
- personal retrospective
- fix validation
- rollback
- open questions
- temporary artifact cleanup status

Before finishing, delete temporary backups, diagnostic logs, registry exports, packet captures, and local proof files unless they are intentionally retained. If a temporary artifact is retained, document:

- path
- reason
- when it can be deleted

## Oral Notes Cleanup

When converting spoken notes:

1. Remove fillers.
2. Preserve constraints and decisions.
3. Normalize repeated terms.
4. Turn vague pronouns into explicit nouns when context is clear.
5. Mark assumptions.
6. Output direct Markdown.
