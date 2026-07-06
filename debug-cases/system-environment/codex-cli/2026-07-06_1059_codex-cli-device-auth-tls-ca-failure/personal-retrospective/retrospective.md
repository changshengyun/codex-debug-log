# Retrospective: Codex CLI Device-Auth TLS CA Failure

## 1. Initial Judgment

The initial reasoning focused on recent visible events:

- MFA had been enabled.
- Old login state might have become invalid.
- Multiple mobile approval attempts might have triggered account risk controls.
- Codex login failed after those account-related changes.

That reasoning was understandable because the timing looked related. The weak point was that it treated temporal correlation as technical causality. At that stage, there was not yet evidence that the request had reached OpenAI's account or MFA layer.

## 2. Turning Point

The investigation improved when the core question changed from:

```text
Which likely reason caused this?
```

to:

```text
Where does the normal request flow stop?
```

The decisive narrowing questions were:

- Can Codex send HTTP CONNECT?
- Does the proxy return `HTTP/1.1 200 Connection established`?
- After CONNECT 200, does Codex send TLS ClientHello?
- Is the close a FIN or RST?
- Does the same EOF happen with a fake proxy that does not connect to Clash or OpenAI?
- Does setting `CODEX_CA_CERTIFICATE` change the behavior?

These questions turned the debug from broad guessing into layer-by-layer fault isolation.

## 3. Correct Debug Model

### Normal Path

```text
codex login --device-auth
-> read proxy variables
-> connect to local proxy
-> send CONNECT auth.openai.com:443
-> receive CONNECT 200
-> create TLS client config
-> send TLS ClientHello
-> complete TLS handshake
-> POST /api/accounts/deviceauth/usercode
-> receive device code
-> complete browser authorization
-> cache login token
```

### Actual Path Before Fix

```text
codex login --device-auth
-> read proxy variables
-> connect to local proxy
-> send CONNECT auth.openai.com:443
-> receive CONNECT 200
-> no TLS ClientHello
-> Codex closes socket with FIN/EOF
```

### Breakpoint

```text
after HTTP proxy CONNECT 200
before TLS ClientHello
```

This matters because the failure occurred before OpenAI account logic, MFA, device-code handling, or token exchange could participate.

## 4. Four Tables That Made the Debug Work

### Possible Failure Area Table

| Layer | Initial suspicion | Final status |
|---|---|---|
| Account / MFA | Login state changed after MFA | Excluded as direct cause |
| OpenAI endpoint | Device-auth endpoint might reject request | Excluded by curl/OpenSSL/PowerShell |
| Proxy | Clash or proxy might be broken | Excluded by comparison tests |
| Codex config | `.codex` or wrapper might be polluted | Excluded by clean `CODEX_HOME` and vendor exe |
| TLS / CA | TLS client setup might fail | Strongly supported |
| Windows roots | Default root loading path might fail | Strongly supported, exact low-level cause unconfirmed |

### Suspect Flow Table

| Step | Expected behavior | Actual behavior | Evidence |
|---|---|---|---|
| Proxy tunnel | CONNECT sent and 200 returned | Completed | Logger |
| TLS start | Codex sends ClientHello | Did not happen before fix | Logger / fake proxy |
| Socket close | Continue TLS or fail with TLS error | FIN/EOF | Wireshark |
| CA override | Should not matter if CA path is healthy | Changed behavior | ClientHello appeared after setting CA |

### Investigation Matrix

| Hypothesis | Test | Result | Conclusion |
|---|---|---|---|
| OpenAI endpoint unavailable | curl/OpenSSL/PowerShell through same proxy | Returned device code | Excluded |
| Proxy path broken | Same proxy with other clients | Worked | Excluded |
| Clash caused EOF | fake proxy only returning CONNECT 200 | EOF still reproduced | Excluded |
| TLS/CA initialization failed in Codex | Set `CODEX_CA_CERTIFICATE` | TLS ClientHello appeared and login succeeded | Supported |

### Evidence Status Table

| Claim | Status | Evidence |
|---|---|---|
| The account/MFA layer directly caused the CLI error | Excluded as direct cause | Codex did not reach TLS/HTTP POST before fix |
| Codex could reach the local proxy | Verified | Logger saw CONNECT |
| Proxy returned success | Verified | CONNECT 200 |
| Codex closed before TLS | Verified | No TLS bytes, FIN/EOF |
| CA override was the decisive variable | Strongly supported | Behavior changed immediately after setting CA |
| Exact Windows root-store internal failure | Unconfirmed | Workaround proves path, not exact internal exception |

## 5. Main Lessons

| Lesson | Why It Matters | How To Apply Next Time |
|---|---|---|
| Do not start from likely causes | Recent events can be triggers, not root causes | First draw the normal path |
| Find the breakpoint before explaining it | Root cause analysis without a breakpoint becomes guessing | Ask what completed and what did not happen |
| Separate trigger, symptom, breakpoint, and root cause | MFA was a trigger for re-login, not the direct TLS failure | Label each role explicitly |
| Use comparison clients | They show whether the network/service is generally broken | Compare curl, OpenSSL, PowerShell, app client |
| Use fake services when possible | They remove upstream variables | A fake proxy proved the issue was local to Codex |
| Change one variable at a time | Otherwise the fix cannot be attributed | CA override was tested as a decisive variable |

## 6. Reusable Debug Checklist

1. Write the exact command and original error.
2. Capture the environment and recent changes.
3. Draw the expected normal path.
4. Draw the actual path.
5. Mark the first missing step.
6. List hypotheses only for the narrow breakpoint.
7. Build comparison tests.
8. Change one variable per test.
9. Record evidence status: verified, excluded, unconfirmed.
10. Write fix, validation, rollback, and personal lesson.

## 7. Standard Retrospective Summary

The early phase was experience-driven: MFA changed, login failed, so account/MFA looked suspicious. That was plausible but not proven.

The later phase became process-driven: the investigation checked whether the request reached each layer of the normal flow. Logger, Wireshark, fake proxy, and CA override narrowed the issue to Codex's local TLS setup before ClientHello.

The core improvement is not just learning about CA bundles. The important skill is converting a vague failure into a state machine with observable transitions.

