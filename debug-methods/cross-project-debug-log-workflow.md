# Cross-Project Debug Log Workflow

This repository is the central debug log for all projects.

## Global Setup

The global Codex instruction file is:

```text
C:\Users\Legion\.codex\AGENTS.md
```

It tells Codex to write meaningful debug cases back to:

```text
D:\DevEnv\Work\codex-debug-log
```

This means a Debug task from another project should still produce or update a case folder in this central repository.

## How To Ask In Any Project

Use one of these prompts:

```text
这次 Debug 完成后，把专业报告和我的复盘写入中央 debug log，并同步 GitHub。
```

```text
用 debug-retrospective-writer，把这次排障记录到 D:\DevEnv\Work\codex-debug-log。
```

```text
这次修复也要进入 codex-debug-log，按时间戳案例目录保存。
```

## Case Directory Rule

Each debug case gets one timestamped directory:

```text
debug-cases/
  YYYY-MM-DD_HHMM_short-error-cause/
    README.md
    professional-debug-report/
      report.md
    personal-retrospective/
      retrospective.md
```

The folder name must include:

- date
- time
- affected tool/project
- short failure cause or original error area

Example:

```text
2026-07-06_1059_codex-cli-device-auth-tls-ca-failure
```

## What Gets Written

Every meaningful debug case should include two views.

### Professional Debug Report

Use this for technical facts:

- original error
- environment
- expected flow
- actual flow
- breakpoint
- evidence matrix
- root-cause analysis
- fix
- validation
- rollback

### Personal Retrospective

Use this for learning:

- initial judgment
- why the early judgment was plausible
- what was weak in the reasoning
- turning point
- better debug model
- reusable lessons
- next-time checklist

## Cleanup Rule

Temporary files are allowed while investigating, but they should not remain after the task.

Delete these before finishing unless explicitly retained:

- `*.reg` registry exports
- `*.log` diagnostic logs
- trace files
- packet captures
- one-off test scripts
- temporary backups

If a temporary file must be kept, write down:

- exact path
- reason
- when it can be deleted

## Git Sync Rule

After updating this repository:

```powershell
cd D:\DevEnv\Work\codex-debug-log
git status --short
git add <changed files>
git commit -m "<clear message>"
git push
```

The repository currently uses GitHub SSH over port 443:

```text
ssh://git@ssh.github.com:443/changshengyun/codex-debug-log.git
```

## Project-Level AGENTS.md

For most projects, no project-level file is required because the global file already covers the central debug log.

Add a project-level `AGENTS.md` only when the project has special rules, for example:

```markdown
# Project Debug Rules

When debugging this project, also include:

- service name
- local dev command
- failing endpoint
- relevant environment file
- test command

Still write final debug reports to:

D:\DevEnv\Work\codex-debug-log
```

