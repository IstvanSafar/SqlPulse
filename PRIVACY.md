# Privacy Policy

**Last updated: March 2026**

SqlPulse is a local SSMS extension. It has no accounts, no cloud sync, and no telemetry by default.

---

## Data stored locally

All data SqlPulse creates stays on your machine under `%LOCALAPPDATA%\SqlPulseTool\`:

| File | Contents | Sensitive? |
|------|----------|------------|
| `history.jsonl` | Query text, timestamp, connection info | Yes — contains your SQL queries |
| `settings.json` | Extension configuration | Low — no credentials |
| `extension.log` | Diagnostic log with `[Tag]` prefixes | Low — no query content by default |
| `db-groups.json` | Database group definitions | Low |
| `job-groups.json` | Job group definitions | Low |

**SqlPulse never reads or transmits the contents of your query history, result grids, or connection strings.**

---

## Data sent over the network

### License activation

**None.** SqlPulse is free - there is no licence key, no account and no activation step, so no machine
fingerprint or licence data is ever created or sent. Builds up to 0.1.298 carried an unused activation
code path for a planned Pro tier; the server was never deployed, and as of 0.1.299 the code is gone.

### Update check

The only outbound request the extension can make: an optional check against the public GitHub releases
API (`api.github.com`) to see whether a newer version exists. It sends no identifying information beyond
what any HTTPS request necessarily reveals.

### Telemetry / usage analytics

**Disabled by default.** If you enable "Track feature usage" in Settings > Advanced, only anonymous feature counts are stored locally (e.g. `filter: 12`, `export_csv: 5`). This data is never uploaded anywhere.

---

## SQL Server History Backend

If you configure the SQL Server History Backend, query history is written to a SQL Server table **you specify**. SqlPulse connects using the connection string you provide — it does not control or have access to that server independently.

---

## Data retention

SqlPulse does not impose retention limits. History grows until you clear it manually (Settings > History > Clear History) or configure a max-entry limit.

---

## Children's privacy

SqlPulse is a developer tool not intended for use by anyone under 16.

---

## Changes to this policy

Updates will be reflected in this file in the [SqlPulse documentation repository](https://github.com/IstvanSafar/SqlPulse). The "Last updated" date at the top will change.

---

## Contact

For privacy questions or data requests: open an issue at [GitHub Issues](https://github.com/IstvanSafar/SqlPulse/issues).
