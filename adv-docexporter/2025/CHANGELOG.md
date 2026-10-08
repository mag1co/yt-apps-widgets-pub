# Changelog

## v2.0.6

### Versioning

- **Minimum YouTrack version raised to 2025.3** — the manifest now requires YouTrack 2025.3 or later. Code unchanged: the app uses no APIs newer than 2025.1.
- **Last release line for YouTrack 2025** — the next major version, 3.0.0, will require YouTrack 2026. The 2.0.x line stays available for YouTrack 2025 on the **Versions** tab.

### Fixes

- **Malformed `.rels` XML with hyperlinks containing `&`** — URLs with query strings (e.g. `?a=1&b=2`) are now escaped in the `Target` attribute of `.rels` relationship parts. Previously the raw `&` produced invalid XML, causing Excel/Word to report the exported file as corrupt and openpyxl/Expat to reject it.

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
