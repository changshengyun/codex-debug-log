# Debug Template Reference

## Core Model

Use this model for technical debug records:

```text
Surface symptom
-> Expected flow
-> Actual flow
-> Breakpoint
-> Hypotheses
-> Tests
-> Evidence status
-> Root-cause mechanism
-> Fix
-> Rollback
-> Retrospective
```

## Four Tables

### 1. Possible Failure Area Table

```markdown
| Layer | Possible issue | Initial evidence | Priority |
|---|---|---|---|
```

### 2. Suspect Flow Table

```markdown
| Step | Expected behavior | Actual behavior | Evidence |
|---|---|---|---|
```

### 3. Investigation Matrix

```markdown
| Hypothesis | Test method | Expected result | Actual result | Conclusion |
|---|---|---|---|---|
```

### 4. Evidence Status Table

```markdown
| Claim | Status | Evidence | Notes |
|---|---|---|---|
```

## Retrospective Questions

Ask these questions in the generated retrospective:

1. What did the user initially suspect?
2. Was that suspicion a root cause, trigger, or coincidence?
3. Which evidence narrowed the failure layer?
4. Which comparison test was decisive?
5. Which assumptions were eliminated?
6. What should be done earlier next time?

