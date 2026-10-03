# Changelog

## v3.0.0

### Compatibility

- **Requires YouTrack 2026.2 or later** — the minimum YouTrack version is raised to 2026.2. On YouTrack 2025, keep using the 2.0.x line from the **Versions** tab.

### Features

- **Remember export settings (optional)** — a new **Remember these settings in this browser** checkbox in the export dialog. When it is ticked, the format, content and formatting option you export with are preselected the next time you open the dialog. Unticking it and exporting forgets them. Off by default; the settings are kept in your browser only.
- **Dialog language follows YouTrack** — the dialog (English or Russian) now follows the language of your YouTrack interface instead of the browser language. Other interface languages get English.

### Under the hood

- Updated to React 19 and the current YouTrack Apps tooling (Enhanced DX). No changes to export output.

## v2.0.4

### Versioning

- **Minimum YouTrack version declared** — the app manifest now explicitly requires YouTrack 2025.1 or later.
- **Last release line for YouTrack 2025** — the next major version, 3.0.0, will require YouTrack 2026. The 2.0.x line stays available for YouTrack 2025 on the **Versions** tab.
- No functional changes since v2.0.3.

## v2.0.3

- **Feature / UX:** Unified export dialog. 4 separate export menu items are merged into a single "Export article..." menu item with format and content settings.
- **BREAKING:** Removed legacy Admin and Project settings panels from YouTrack menus, as the app no longer requires backend configuration.
- **Under the hood:** Cleaned up unused legacy backend code, removed obsolete permission checks, and optimized the app's build footprint for a smaller, cleaner release.

## v1.2.2

- Performance: export now runs entirely in the browser with no server requests
- Fix: hyperlinks in XLSX files are now clickable in Excel
- **BREAKING (admins migrating from v1.0.x):** configurable settings removed — file prefix (`export`), date suffix, and notification mode are no longer configurable per-project

## v1.0.13

- Security: added safe url guard for XLSX hyperlinks (both single-cell and multi-run)
- Security: added safe url guard for DOCX table hyperlinks
- Refactoring: dry refactoring
- Fix: corrected table heading detection — use line index instead of `indexOf` to avoid wrong heading for duplicate table rows

## v1.0.12

- Security: sanitize URL schemes in hyperlinks during Office export (block javascript:, file://, etc.)
- DOCX: blockquote styling with grey left border

## v1.0.11

- Renamed app display title to Adv.DocExporter
- Updated logging to be more centralized and context-aware
- Normalized app to comply with updated Marketplace guidelines

## v1.0.1

- Security: added READ_ARTICLE permission check on global /export endpoint

## v1.0.0

- First stable release
- Published to JetBrains Marketplace (Stable channel)
- Project-level settings: file name prefix, date suffix, notification mode
- EULA, PRIVACY, GETTING_STARTED for Marketplace listing
- Publish and sync scripts for Marketplace, GitLab, public GitHub
