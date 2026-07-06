# Case: Codex CLI Device-Auth TLS CA Failure

## Metadata

```text
Timestamp: 2026-07-06 10:59
Debug Name: Codex CLI device-auth TLS CA failure
Bug Cause: TLS CA initialization path
Cause Summary: Codex CLI could establish the HTTP proxy tunnel but closed the connection before sending TLS ClientHello. The most actionable cause was Codex failing on the default Windows/system roots TLS setup path; setting CODEX_CA_CERTIFICATE to Git for Windows' PEM CA bundle allowed TLS setup and login to complete.
Tool: Codex CLI
Original failure area: device-auth login / HTTPS proxy / TLS CA initialization
Direct breakpoint: after HTTP CONNECT 200, before TLS ClientHello
Primary fix: CODEX_CA_CERTIFICATE -> Git for Windows PEM CA bundle
```

## Files

- [Professional debug report](professional-debug-report/report.md)
- [Personal retrospective](personal-retrospective/retrospective.md)

## One-Line Conclusion

The visible login failure was not directly caused by MFA, account state, OpenAI rejection, or proxy reachability. The decisive breakpoint was that Codex closed the connection after the proxy tunnel succeeded but before TLS ClientHello; setting `CODEX_CA_CERTIFICATE` let Codex build TLS client config and complete login.
