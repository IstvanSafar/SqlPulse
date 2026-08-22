# Security & Transparency

This document describes SqlPulse's security model and what data leaves your machine. The source code of the security-relevant paths is included below so you can audit it directly.

---

## What data leaves your machine

**During normal use: nothing.**

Query history, filter state, settings, and all other data is stored locally in `%LOCALAPPDATA%\SqlPulseTool\`. Nothing is uploaded anywhere.

---

## Credential storage policy

**SqlPulse does not store SQL Server passwords.**

This is a deliberate design decision, not a limitation:

- SQL Server passwords are not written to `settings.json` or any other file
- The **Quick Connect** feature stores server name, database, and username (for display only) — never the password
- Authentication is handled by SSMS itself when you open a new query window — SqlPulse does not intercept or cache credentials
- **Windows Authentication (Integrated Security)** is recommended and fully supported with no credentials to manage

If you use SQL Server Authentication, your login name is shown in the Quick Connect list as a reminder, but no password is stored or transmitted.

> **Why not store passwords with DPAPI?** DPAPI (Windows Data Protection API) encrypts data so only the current Windows user can decrypt it on the same machine. While this is a common pattern, it still results in credential material on disk — accessible if the Windows account is compromised, accidentally committed to version control, or included in a backup. SSMS itself uses Windows Credential Manager rather than file-based credential storage. SqlPulse follows the same principle: **no passwords on disk**.

---

**During licence activation: there is no licence activation.**

SqlPulse is free - every feature, no licence key, no account, no activation step. The extension makes
**no network calls at all** and works fully offline. There is no server to talk to, so there is nothing
to send: no fingerprint, no hostname, no username, no query content, no connection strings, no telemetry.

> Builds up to 0.1.298 contained an unused RSA licence-verification path for a planned Pro tier. The
> activation server was never deployed and no key was ever issued, so it never contacted anything - and
> as of **0.1.299** the tier and its code are gone entirely. The only outbound request the extension can
> make is the optional update check against the public GitHub releases API.

---

## Verification steps

If you want to independently verify the above:

1. **Wireshark / Fiddler** — monitor outbound traffic while using SSMS; apart from the optional update check against `api.github.com`, you will see nothing leave the machine
2. **Process Monitor** — filter on the SSMS process to verify that no unexpected file or network activity occurs
3. **ILSpy / dnSpy** — decompile `SqlPulseTool.SsmsExtension.dll` and confirm there is no activation or telemetry code path

---

## Reporting security issues

Please report security vulnerabilities via [GitHub Issues](https://github.com/IstvanSafar/SqlPulse/issues) or by emailing directly. Do not post exploit details publicly before a fix is available.
