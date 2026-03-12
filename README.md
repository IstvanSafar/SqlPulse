# SqlPulse - Productivity Extension for SSMS

[![SSMS 18-22](https://img.shields.io/badge/SSMS-18--22-blue.svg)](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)
[![Windows](https://img.shields.io/badge/Windows-10%2F11-0078D6.svg)]()
[![.NET Framework](https://img.shields.io/badge/.NET_Framework-4.8-purple.svg)]()

**SqlPulse** is a productivity extension for SQL Server Management Studio that adds query history, result grid filtering, SQL code analysis, execution plan comparison, and workspace management to SSMS.

## Features

### Free

| Feature | Description |
|---------|-------------|
| **Query History** | Automatically captures every executed query. Search, filter by date, and replay with one click. |
| **Result Grid Filter** | Live row filtering directly in the SSMS result grid. Type to search across all columns instantly. |
| **Data Export** | Export result grids to XLSX, CSV or JSON. |
| **SQL Formatter** | Format T-SQL with configurable rules (keyword casing, indentation, newlines). |

#### Visuals

*Result Grid Filter*
![Result Grid Filter](docspict/Animation.webp)

*Data Export*
![Data Export](docspict/ResultGrid_ExportMenu.png)

### Pro

| Feature | Description |
|---------|-------------|
| **SQL Code Inspector** | 50+ static analysis rules across 4 categories: modernization, performance, best practices, and security. AST-based detection using ScriptDom. |
| **Execution Plan Analyzer** | Visual plan tree, side-by-side plan comparison, table usage summary, and operator search within complex plans. |
| **Advanced Grid Filter** | Multi-condition filters with AND/OR logic and 9 operators (Contains, Regex, Equals, Starts With, etc.). |
| **Session Management** | Save and restore entire tab groups with connection contexts. Resume complex tasks exactly where you left off. |
| **Object Search** | Fast full-text search across all database objects (tables, views, procedures, functions) with cached metadata. |
| **Tab Management** | Auto-rename and color-code query tabs based on environment. Red for Production, Green for Dev - prevent accidental executions. |
| **DB & Job Grouping** | Organize databases and SQL Agent jobs into colored groups in Object Explorer. |
| **Query Playbooks** | Define multi-step query workflows with connection overrides. Execute, track results, and export. |
| **Excel Export** | Export result grids to XLSX (OpenXML). |
| **Encrypted History** | AES-256 encrypted history storage with passphrase protection. |
| **SQL Server History Backend** | Store query history in a shared SQL Server table for team-wide access. |

#### Visuals
*Grid Filter*
![Grid Filter](docspict/Animation.webp)

*Session Management*
![Session Management](docspict/HistoryWindow_Sessions.png)

*Object Explorer Database and Job Grouping*
![DB Grouping](docspict/ObjectExplorer_DBGrouping.png)
![Job Grouping](docspict/ObjectExplorer_JobGrouping.png)

*Object Search*
![Object Search Results](docspict/ObjectSearch_Results.png)
![Object Search Definition](docspict/ObjectSearch_Definition.png)

*SQL Code Inspector*
![SQL Code Inspector](docspict/SQLCodeInspector_Window.png)

## Integration in SSMS

SqlPulse integrates seamlessly into SSMS via the Tools menu and context menus.

*Tools Menu*
![SSMS Tools Menu](docspict/SSMS_ToolsMenu.png)
![SSMS Tools Menu Expanded](docspict/SSMS_ToolsMenu_Expanded.png)

*Query Editor Context Menu*
![Query Editor Context Menu](docspict/QueryEditor_ContextMenu.png)
![Query Editor Context Menu Expanded](docspict/QueryEditor_ContextMenu_Expanded.png)

## Settings

SqlPulse is highly configurable to suit your workflow.

![General Settings](docspict/Settings_General.png)
![Storage Settings](docspict/Settings_Storage.png)
![History Settings](docspict/Settings_History.png)
![Session Settings](docspict/Settings_Sessions.png)
![DB Groups Settings](docspict/Settings_DBGroups.png)
![Job Groups Settings](docspict/Settings_JobGroups.png)
![Shared Settings](docspict/Settings_Shared.png)
![Plan Analyzer Settings](docspict/Settings_PlanAnalyzer.png)
![SQL Formatter Settings](docspict/Settings_SQLFormatter.png)
![Advanced Settings](docspict/Settings_Advanced.png)

## Installation

### Setup Installer (Recommended)

1. Download the latest **SqlPulseInstaller.exe** from [Releases](https://github.com/IstvanSafar/SqlPulse/releases)
2. Run the installer
3. Restart SSMS
4. The **SqlPulse** menu appears under **Tools**

![SqlPulse Installer Start](docspict/Installer_Start.png)
![SqlPulse Installer Running](docspict/Installer_Running.png)
![SqlPulse Installer Finished](docspict/Installer_Finished.png)

> The installer automatically detects all SSMS versions on your machine and installs the extension for each.

## Quick Start

![SqlPulse Menu](docspict/Screenshot%202026-03-02%20165309.png)

| Action | How |
|--------|-----|
| Open Query History | **Tools** > **SqlPulse** > **Query History** |
| Filter Result Grid | Right-click result grid > **Filter Grid...** |
| Format SQL | **Tools** > **SqlPulse** > **Format SQL** |
| Inspect SQL Code | **Tools** > **SqlPulse** > **SQL Code Inspector** |
| Analyze Execution Plan | **Tools** > **SqlPulse** > **Plan Analyzer** |
| Search Objects | **Tools** > **SqlPulse** > **Object Search** |
| Open Settings | **Tools** > **SqlPulse** > **Settings** |

## SQL Code Inspector Rules

50+ rules across 4 categories, powered by Microsoft ScriptDom AST analysis.

<details>
<summary><strong>Modernization (20 rules)</strong> - Legacy pattern detection with modern T-SQL suggestions</summary>

| Rule | Detection | Suggestion |
|------|-----------|------------|
| SQL001 | `LTRIM(RTRIM(x))` nesting | `TRIM()` (2017+) |
| SQL002 | `FOR XML PATH('')` aggregation | `STRING_AGG()` (2017+) |
| SQL003 | `IF OBJECT_ID... DROP` pattern | `DROP IF EXISTS` (2016+) |
| SQL004 | `RAISERROR` usage | `THROW` (2012+) |
| SQL005 | `ISNULL` concatenation | `CONCAT()` (2012+) |
| SQL006 | Stored proc without `SET NOCOUNT ON` | Add `SET NOCOUNT ON` |
| SQL007 | Simple `CASE` with one `WHEN`+`ELSE` | `IIF()` (2012+) |
| SQL008 | `@@IDENTITY` | `SCOPE_IDENTITY()` |
| SQL009 | `sp_` prefix on user procs | Rename without `sp_` prefix |
| SQL010 | `SELECT TOP` without `ORDER BY` | Add `ORDER BY` |
| SQL011 | `EXEC()` with string concatenation | `sp_executesql` |
| SQL012 | Old-style comma `JOIN` | ANSI `JOIN` syntax |
| SQL013 | `GETDATE()` | `SYSDATETIME()` for higher precision |
| SQL014 | `TEXT`/`NTEXT`/`IMAGE` types | `VARCHAR(MAX)`/`NVARCHAR(MAX)`/`VARBINARY(MAX)` |
| SQL015 | Missing semicolon terminator | Add `;` |
| SQL016 | Other deprecated data types | Modern replacements |
| SQL017 | `GOTO` statement | Structured control flow |
| SQL021 | `TOP (100) PERCENT` | Remove (ignored by optimizer) |
| SQL024 | `DATETIME` type | `DATETIME2` for precision |
| SQL027 | `MERGE` without `HOLDLOCK` | Add `WITH (HOLDLOCK)` |

</details>

<details>
<summary><strong>Performance (21 rules)</strong> - Query performance anti-patterns</summary>

| Rule | Detection | Impact |
|------|-----------|--------|
| PERF001 | `SELECT *` | Unnecessary I/O, blocks covering indexes |
| PERF002 | `NOLOCK` / `READUNCOMMITTED` | Dirty reads, data inconsistency |
| PERF003 | Cursor declarations | Row-by-row processing |
| PERF004 | Scalar UDF in queries | Hidden RBAR, blocks parallelism |
| PERF005 | `NOT IN (subquery)` | NULL handling issues, poor plans |
| PERF006 | `LIKE '%pattern'` | Cannot use index seek |
| PERF007 | `N''` literal vs `varchar` column | Implicit conversion, index scan |
| PERF008 | Function on column in `WHERE`/`JOIN` | Non-SARGable, forces scan |
| PERF009 | `INSERT` without column list | Fragile, breaks on schema changes |
| PERF010 | `UNION` (implicit `DISTINCT`) | Unnecessary sort, use `UNION ALL` |
| PERF011 | Table variables for large sets | No statistics, cardinality = 1 |
| PERF012 | `WHILE` loops | Set-based alternatives preferred |
| PERF013 | `EXISTS(SELECT *)` | Use `EXISTS(SELECT 1)` |
| PERF014 | Scalar subqueries | May execute per row |
| PERF015 | `ORDER BY` ordinal position | Fragile, hard to read |
| PERF016 | Excessive `DISTINCT` | May mask join issues |
| PERF018 | Large `IN (...)` literal list | Use temp table or TVP |
| PERF019 | `ORDER BY NEWID()` | Expensive random sort |
| PERF021 | Random sampling patterns | Consider `TABLESAMPLE` |
| PERF023 | `OR` across multiple columns | Consider `UNION ALL` |
| PERF032 | `LEFT JOIN` with `WHERE` filter on right | Effectively `INNER JOIN` |

</details>

<details>
<summary><strong>Best Practice (9 rules)</strong> - Code quality and safety</summary>

| Rule | Detection |
|------|-----------|
| BP001 | `EXEC proc` without schema prefix |
| BP002 | `IF`/`ELSE` without `BEGIN...END` |
| BP005 | Procedure with `TRANSACTION` missing `SET XACT_ABORT ON` |
| BP008 | Statement missing semicolon terminator |
| BP011 | `DELETE`/`UPDATE` without `WHERE` |
| BP012 | `ROLLBACK` without `@@TRANCOUNT` check |
| BP013 | `WAITFOR DELAY` usage |
| BP018 | `FLOAT`/`REAL` for exact values |

</details>

<details>
<summary><strong>Security (1 rule)</strong></summary>

| Rule | Detection |
|------|-----------|
| SEC001 | `UPDATE`/`DELETE` without `WHERE` - affects all rows |

</details>

## Query History Backends

| Backend | Use Case |
|---------|----------|
| **JSON** (default) | Local file, optional AES-256 encryption, portable |
| **SQLite** | Local database, fast queries, good for large history |
| **SQL Server** (Pro) | Shared team history via central table |

## Compatibility

| SSMS Version | Status |
|--------------|--------|
| SSMS 18.x | Supported |
| SSMS 19.x | Supported |
| SSMS 20.x | Supported |
| SSMS 22.x | Supported |

**Requirements:** Windows 10 or later, .NET Framework 4.8 (included with SSMS).

## Licensing

SqlPulse has a **Free** tier with core features and a **Pro** tier with the full suite. Pro activation requires a one-time internet connection; after that, it works entirely offline.

Activate via: **Tools** > **SqlPulse** > **Activate License**

## Troubleshooting

**Extension doesn't load after install**
- Restart SSMS completely (close all instances)
- Check `%LOCALAPPDATA%\SqlPulseTool\extension.log` for errors

**Filter toolbar doesn't appear**
- Execute a query first so a result grid is visible
- Right-click the result grid and select **Filter Grid...**

**Code Inspector shows no results**
- Ensure the active document contains valid T-SQL
- Click **Analyze** or switch to the SQL Inspector tool window

**Releases**

[GitHub Releases](https://github.com/IstvanSafar/SqlPulse/releases)


## Security & Privacy

- [Security policy & activation source code](SECURITY.md) — what data leaves your machine, full source of the license system
- [Privacy policy](PRIVACY.md) — what is stored locally and what (if anything) is transmitted

## Support

- **Bug reports & feature requests:** [GitHub Issues](https://github.com/IstvanSafar/SqlPulse/issues)
- **Discussions:** [GitHub Discussions](https://github.com/IstvanSafar/SqlPulse/discussions)
- **Log file:** `%LOCALAPPDATA%\SqlPulseTool\extension.log` (attach when reporting issues)

![Downloads](https://img.shields.io/github/downloads/IstvanSafar/SqlPulse/total)
