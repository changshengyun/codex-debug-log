# Codex CLI Windows Device-Auth Login Failure: TLS CA Initialization Issue

## 1. Summary

Codex CLI failed during ChatGPT device-auth login on Windows. The visible error was a generic request failure when calling the OpenAI device-auth endpoint:

```text
Error logging in with device code: error sending request for url
https://auth.openai.com/api/accounts/deviceauth/usercode
```

The final verified fix was to set `CODEX_CA_CERTIFICATE` to Git for Windows' PEM CA bundle:

```text
C:\Program Files\Git\mingw64\etc\ssl\cert.pem
```

After that, `codex login --device-auth` succeeded and `codex login status` returned:

```text
Logged in using ChatGPT
```

The strongest evidence indicates that the failure occurred after the HTTP proxy tunnel was established, but before Codex emitted the TLS ClientHello. The problem was not directly caused by MFA, OpenAI rejecting the account, the proxy endpoint being unreachable, or the network path being generally broken.

## 2. Environment

```text
Operating system: Windows
User directory: C:\Users\Legion
Original Codex CLI version: codex-cli 0.142.5
Current Codex CLI version: codex-cli 0.143.0-alpha.36
Primary proxy: 127.0.0.1:7897
Logger proxy: 127.0.0.1:18181
Fake proxy: 127.0.0.1:18182
Effective CA bundle: C:\Program Files\Git\mingw64\etc\ssl\cert.pem
Final state: Successfully logged in / Logged in using ChatGPT
```

## 3. User-Level Changes Made

The fix wrote user-level Windows environment variables:

```powershell
[Environment]::SetEnvironmentVariable(
  "CODEX_CA_CERTIFICATE",
  "C:\Program Files\Git\mingw64\etc\ssl\cert.pem",
  "User"
)

[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://127.0.0.1:7897", "User")
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://127.0.0.1:7897", "User")
[Environment]::SetEnvironmentVariable("NO_PROXY", "localhost,127.0.0.1,::1", "User")
```

These changes affect newly started user processes.

No evidence indicates that the following were modified:

- Windows system root certificate store
- Windows firewall
- Schannel TLS registry settings
- Clash configuration
- OpenAI account security settings
- Codex `auth.json`
- Git's `cert.pem` file

## 4. Expected Device-Auth Flow

The expected successful flow is:

```text
1. User runs codex login --device-auth.
2. Codex loads config, environment variables, and auth state.
3. Codex reads HTTP_PROXY / HTTPS_PROXY.
4. Codex connects to local proxy 127.0.0.1:7897.
5. Codex sends HTTP CONNECT for auth.openai.com:443.
6. Proxy returns HTTP/1.1 200 Connection established.
7. Codex switches the proxy tunnel into a TLS stream.
8. Codex sends TLS ClientHello.
9. OpenAI returns ServerHello and certificate chain.
10. Codex verifies the server certificate through a CA bundle.
11. Codex sends POST /api/accounts/deviceauth/usercode over TLS.
12. OpenAI returns device-auth metadata.
13. User completes browser login, MFA, and authorization.
14. Codex receives and stores the login token.
15. codex login status reports Logged in using ChatGPT.
```

## 5. Actual Failure Point

The observed failing flow was:

```text
Codex CLI
  -> connects to proxy
  -> sends CONNECT auth.openai.com:443
  -> receives HTTP/1.1 200 Connection established
  -> sends no TLS ClientHello
  -> closes the connection with FIN/EOF
```

Precise breakpoint:

```text
after HTTP proxy CONNECT 200
before TLS ClientHello
```

Narrowed technical location:

```text
during Codex TLS client setup
related to default system roots / CA loading path
before the first TLS record is written
```

## 6. Evidence Timeline

| Step | Test | Result | Conclusion |
| ---: | --- | --- | --- |
| 1 | Ran `codex login --device-auth` | Generic request error | Surface symptom was request failure |
| 2 | Used `curl` through the same proxy against device-auth endpoint | Returned HTTP 200 and device code | OpenAI endpoint and proxy path were reachable |
| 3 | Used OpenSSL TLS 1.3 through the proxy | TLS handshake succeeded, certificate verified | TLS 1.3 path was usable |
| 4 | Used OpenSSL TLS 1.2 through the proxy | TLS handshake succeeded | TLS 1.2 path was usable |
| 5 | Sent full POST through OpenSSL TLS | Device code returned | OpenAI API could respond normally |
| 6 | Used PowerShell `Invoke-RestMethod` through the same proxy | Device code returned | .NET/PowerShell network stack worked |
| 7 | Used a clean temporary `CODEX_HOME` | Codex still failed | Not caused by local `.codex` config pollution |
| 8 | Ran vendor `codex.exe` directly | Codex still failed | Not caused by npm wrapper |
| 9 | Inserted logger proxy between Codex and primary proxy | CONNECT 200 followed by EOF | Failure was after CONNECT, before TLS |
| 10 | Checked loopback traffic with Wireshark | FIN/ACK, not RST | Codex closed the socket normally at app level |
| 11 | Used fake proxy that only returns CONNECT 200 | EOF with no TLS bytes | Failure did not require Clash or OpenAI |
| 12 | Located Git CA bundle | Found valid `cert.pem` | Candidate for `CODEX_CA_CERTIFICATE` |
| 13 | Set `CODEX_CA_CERTIFICATE` to Git CA bundle | Fake proxy saw bytes beginning `16 03 01` | Codex emitted TLS ClientHello |
| 14 | Retried real device-auth login | `Successfully logged in` | Fix worked |
| 15 | Checked login state | `Logged in using ChatGPT` | Login state was valid |

## 7. Key Evidence

Logger captured Codex successfully sending a proxy tunnel request:

```text
CONNECT auth.openai.com:443 HTTP/1.1
Host: auth.openai.com:443
```

The proxy returned:

```text
HTTP/1.1 200 Connection established
```

Before the CA override, the connection then ended without any TLS bytes:

```text
client -> upstream EOF
```

Wireshark confirmed this was a normal FIN/ACK close, not a TCP reset.

The fake proxy test was important because it did not connect to Clash or OpenAI. It only returned a standard CONNECT 200 response. Codex still closed the socket without TLS bytes, which isolated the failure to Codex's local transition from proxy tunnel to TLS stream.

After setting:

```powershell
$env:CODEX_CA_CERTIFICATE="C:\Program Files\Git\mingw64\etc\ssl\cert.pem"
```

the fake proxy saw TLS-like bytes:

```text
RESULT: got 1484 bytes after 200
first bytes hex: 16 03 01 05 c7 01 00 05 c3 03 03 ...
```

`16 03 01` is consistent with the start of a TLS record, which indicates Codex reached the ClientHello stage.

## 8. Root-Cause Analysis

### Direct Failure Point

Codex CLI closed the connection after the HTTP proxy CONNECT succeeded and before sending TLS ClientHello.

### Likely Technical Mechanism

Codex CLI failed while preparing the TLS client configuration using its default system roots / Windows root certificate loading path. Because TLS setup did not complete, Codex closed the socket before writing the first TLS record.

### Verified Workaround

Setting `CODEX_CA_CERTIFICATE` to a valid PEM CA bundle bypassed the failing default root-loading path:

```text
CODEX_CA_CERTIFICATE=C:\Program Files\Git\mingw64\etc\ssl\cert.pem
```

After this, Codex successfully emitted TLS ClientHello and completed device-auth login.

### Incorrect or Weaker Explanations

The evidence does not support these as direct causes:

- "The OpenAI endpoint was down."
- "The account or MFA directly caused the CLI failure."
- "Clash or the proxy was generally broken."
- "The Windows network stack was broken."

### Still Not Fully Determined

The exact low-level internal cause remains unconfirmed. Plausible possibilities include:

- Codex or its Rust TLS backend failed to read the Windows root store.
- The current Windows user/system certificate store had a state incompatible with Codex's root-loading path.
- Codex CLI `0.142.5` / `0.143.0-alpha.36` had a Windows system-roots compatibility issue.
- Security software, certificate store history, or system policy affected Codex's certificate-loading library.
- The version change and CA override both contributed to the final observed behavior.

The verified operational conclusion is enough for maintenance:

```text
Reproducible symptom: no CA override -> EOF after CONNECT 200
Verified workaround: CODEX_CA_CERTIFICATE -> TLS ClientHello + successful login
```

## 9. Rollback

To remove proxy variables:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", $null, "User")
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", $null, "User")
[Environment]::SetEnvironmentVariable("NO_PROXY", $null, "User")
```

To remove the Codex CA override:

```powershell
[Environment]::SetEnvironmentVariable("CODEX_CA_CERTIFICATE", $null, "User")
```

Recommended default:

```text
Keep CODEX_CA_CERTIFICATE unless a later Codex version or Windows root-store fix makes it unnecessary.
Only change HTTP_PROXY / HTTPS_PROXY when the VPN or Clash proxy port changes.
```

## 10. Proxy Port Maintenance

If the proxy remains:

```text
127.0.0.1:7897
```

no change is needed.

If a different VPN or proxy changes the HTTP/Mixed port, update only the proxy variables:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://127.0.0.1:<new-port>", "User")
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://127.0.0.1:<new-port>", "User")
```

Then open a new PowerShell session and verify:

```powershell
codex login status
codex "reply OK to confirm Codex is currently usable"
```

## 11. Maintenance Notes

For future Debug records:

1. Record the surface error first.
2. Draw the expected normal request chain.
3. Identify the exact layer where the actual flow diverges.
4. Build comparison tests.
5. Change only one variable at a time.
6. Use logs, packet capture, or fake services to prove where the request stops.
7. Separate verified conclusions from inference.
8. Include rollback commands.
9. Store the result as a Markdown file in this repository.

## 12. Source Reference

OpenAI's Codex authentication documentation describes ChatGPT login, device-code login, local credential caching, and the `CODEX_CA_CERTIFICATE` environment variable for specifying a PEM CA bundle:

```text
https://developers.openai.com/codex/auth
```

