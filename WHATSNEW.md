# What's New in SqlPulse

**v0.1.230 - 2026-03-29**

---

### Added
- **Keyboard Shortcuts** settings page — configurable shortcuts for all major features (Profiler, Query History, Plan Analyzer, Object Search, SQL Inspector, Quick Connect, Format SQL, Script Object)
- Foreground-only shortcut firing — hooks do not steal keystrokes from other applications
- Multi-shortcut support in `KeyboardHookService` with per-entry debounce
- **SQL Profiler** — live query capture via DMVs; blocking tree, top queries, wait stats panels; compressed session file export; one-click plan analysis for slow queries
- **Quick Connect** — floating dropdown with preferred + recent connections; toolbar button on Standard toolbar
- **Connection Strip** — thin colored bar at the bottom of every query editor showing server/database; color configurable per connection
- **Grid Conditional Formatting (Pro)** — color rows/cells by column value with 13 operators; zebra striping; condition rules override zebra layer
- **Important DB Alert** — colored floating banner on production database switch; server/database pattern matching with wildcards
- **Transaction Guard** — floating reminder for open transactions; auto-dismisses on commit/rollback
- **Object Search (Pro)** — full-text search across tables, views, procedures, functions with cached metadata
- **DB & Job Grouping (Pro)** — colored group folders in Object Explorer for databases and SQL Agent jobs
- **Plan Analyzer (Pro)** — visual execution plan tree, side-by-side plan comparison, table usage summary, operator search
- **Query Playbooks (Pro)** — multi-step query workflows with connection overrides
- **Advanced Grid Filter (Pro)** — multi-condition filters with AND/OR logic and 9 operators
- **IntelliSense schema cache warmup** — auto-warms on view attach for faster IntelliSense
- Dual build: internal VSIX (SSMS 21/22) + shell15 variant (SSMS 18/19/20)
- Auto version increment on every `deploy.ps1` run
- SMO schema cache builder for Object Search metadata

### Changed
- `KeyboardHookService` refactored: multi-shortcut list replaces single volatile fields
- Script Object shortcut moved from General settings to the new Keyboard Shortcuts page
- `ShortcutSettings.Normalize()` no longer forces "Ctrl+Shift+S" as default — allows empty key
- ThemeService refactored — consistent VS theme propagation across all settings pages
- Settings dialog (`UnifiedSettingsDialog`) unified into single tree-navigation window
- Connection detection: `DispatcherTimer` + `DTE.CommandEvents` 3-layer approach (no polling lag)
- Settings architecture: `SettingsService` singleton with clone pattern; `Normalize()` on every load
- Feature flags refactored: `FeatureFlags.IsPro` replaces scattered license checks

### Fixed
- `System.Windows.Forms.Timer` replaced by `DispatcherTimer` everywhere (WinForms timer silently fails in SSMS/WPF host)
- Installer: VSIXInstaller path fixed for SSMS 22+ (uses VS Installer service, not in-product installer)
- SQL Profiler: duplicate/malformed multi-line SQL normalization

---

---

[Back to README](README.md)
