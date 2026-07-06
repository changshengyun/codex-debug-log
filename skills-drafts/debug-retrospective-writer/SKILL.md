---
name: debug-retrospective-writer
description: Convert spoken or messy debug notes into structured Codex debug case folders, professional debug reports, personal retrospectives, and reusable lessons. Use when the user asks to summarize a debug session, maintain a debug log, turn口语化 notes into重点, create a复盘, or generate a professional fault-analysis Markdown record.
---

# Debug Retrospective Writer

## Workflow

1. Extract facts before writing conclusions.
2. Separate verified evidence from inference.
3. Build the debug case around the normal path versus actual path.
4. Ask the user before writing to the central debug log unless they already explicitly requested logging in the current task.
5. Write two outputs for each approved case:
   - `professional-debug-report/report.md`
   - `personal-retrospective/retrospective.md`
6. If the user speaks informally, remove filler words and preserve intent. Do not preserve hesitation, repetition, or self-correction unless it changes meaning.
7. Clean up temporary backups and diagnostic artifacts before finishing unless the user explicitly asks to keep them.

## Case Folder Format

Create one folder per debug case:

```text
debug-cases/
  <debug-type>/
    <project-name>/
      YYYY-MM-DD_HHMM_short-error-cause/
        README.md
        professional-debug-report/
          report.md
        personal-retrospective/
          retrospective.md
```

Choose one debug type:

- `frontend`: UI, browser, CSS, React/Vue/etc., layout, client-side state
- `backend`: APIs, services, server errors, queues, workers
- `ai-agent`: LLM prompts, tool calls, RAG, agents, model integration
- `system-environment`: OS, shell, TLS, certs, proxy, credentials, local machine state
- `tooling-devops`: Git, CI, build tools, package managers, deployment
- `database-data`: SQL, migrations, data integrity, analytics
- `mobile-desktop`: native mobile or desktop app issues
- `security-auth`: auth flows, permissions, OAuth, tokens, secrets
- `other`: cases that do not fit the above

Use a case folder name that includes:

- timestamp
- affected system or tool
- original failure area or likely cause

Example:

```text
debug-cases/system-environment/codex-cli/2026-07-06_1059_codex-cli-device-auth-tls-ca-failure
```

Each case `README.md` must include quick-recall fields:

```text
Debug Type: <debug-type>
Project: <project-name>
Debug Name: <short human-readable name>
Bug Cause: <short cause label>
Cause Summary: <one to three sentences explaining what happened and why>
```

## Professional Report Structure

Use this structure unless the user asks otherwise:

```markdown
# Debug Report: <title>

## 1. Problem
- Trigger:
- Original error:
- User impact:

## 2. Environment
- OS:
- Tool versions:
- Network/proxy:
- Relevant config:

## 3. Expected Flow
1.
2.
3.

## 4. Actual Flow
1.
2.
3.
- Breakpoint:

## 5. Evidence Matrix
| Hypothesis | Test | Result | Status |
|---|---|---|---|

## 6. Root Cause Analysis
- Surface symptom:
- Triggering event:
- Direct failure point:
- Root-cause mechanism:
- Excluded causes:
- Unconfirmed points:

## 7. Fix
- Change:
- Why it works:
- Validation:

## 8. Rollback
- Commands:
- Risk:

## 9. References
```

## Personal Retrospective Structure

Use this structure for the user's learning record:

```markdown
# Retrospective: <title>

## 1. Initial Judgment
- What I first suspected:
- Why it looked plausible:
- What was weak in that reasoning:

## 2. Turning Point
- The key question that improved the debug:
- The evidence that narrowed the scope:

## 3. Correct Debug Model
- Normal path:
- Actual path:
- Breakpoint:

## 4. Lessons
| Lesson | Why it matters | How to apply next time |
|---|---|---|

## 5. Reusable Checklist
- 
```

## Oral Notes Cleanup Rules

When converting口语化 text:

- Delete fillers such as "嗯", "呃", "就是", repeated words, and abandoned sentence starts.
- Preserve the user's main claim, requested action, constraints, and naming preferences.
- Convert vague phrases into explicit nouns when context supports it.
- Mark uncertain parts as assumptions instead of inventing missing facts.
- If the user says "第八个" but context means "Debug", normalize it to "Debug".
- Keep the final text direct and technically precise.

## Debug Reasoning Rules

- Do not start from likely causes. Start from the expected flow.
- Identify which step completed and which step did not occur.
- Build comparison tests when possible.
- Change one variable at a time.
- Keep a status table with `verified`, `excluded`, and `unconfirmed`.
- Do not call an inducing event the root cause unless evidence proves causality.
- If changing environment variables, registry values, credentials, or tool config, create a backup when appropriate, use it only as a rollback safety net, and delete the backup after validation unless it must be retained.
- Before final response, check for local artifacts such as `*.reg`, `*.log`, trace files, packet captures, temporary scripts, and generated diagnostics. Delete unneeded artifacts and report any retained files.
- Do not write every intermediate debug attempt to the central log. If the user is still asking for modifications or is not satisfied with the result, wait. Ask for approval after the current debug/fix is complete.

## References

Read `references/debug-template.md` when the task requires generating a reusable template or explaining the debug method.
