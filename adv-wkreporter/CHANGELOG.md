# Changelog

## v1.1.14

### Versioning

- **Minimum YouTrack version declared** — the app manifest now explicitly requires YouTrack 2025.1 or later.
- **Last release line for YouTrack 2025** — the next major version, 2.0.0, will require YouTrack 2026. The 1.1.x line stays available for YouTrack 2025 on the **Versions** tab.
- No functional changes since v1.1.13.

## v1.1.13
- **Performance**: Drastically reduced API load on the YouTrack server by completely removing issue count API calls on the main dashboard screen. The widget now loads instantly.
- **Feature**: Added support for displaying the native `description` field with safe Markdown rendering (bold, links) in the tooltip.
- **Feature**: Replaced raw HTML rendering with a lightweight, 100% XSS-safe custom Markdown parser.
- **Feature**: Added field aliases support in the "Columns to Display" selector (e.g. `State (S)`).
- **Fix**: Resolved an issue where custom fields (like Priority) would disappear from the dropdown due to strict API 400 Bad Request errors when fetching aliases. Added automatic safe-fallback queries.
- **UI**: Added a sensible set of default fields for newly created widgets.
- **Feature**: Added a dedicated "Project Context" selector in settings to easily load project-specific custom fields for the columns dropdown.
- **Enhancement**: The Project Context selector automatically detects and selects the project if `project: <PROJECT_ID>` is written in the main YouTrack Query.
- **UI**: Improved the widget configuration layout by moving the YouTrack Query to the top and arranging settings more compactly.
- **UI**: Cleaned up the table view and Excel exports by stripping markdown formatting (e.g. `**bold**`, links) to ensure pristine plain text display.
- **UX**: Added a native YouTrack toast notification upon successful Excel report download.

## v1.1.2
- **Security**: Fixed a potential XSS vulnerability by stripping HTML tags from comment tooltips.

## v1.1.1
- **Feature**: Added fully configurable columns selection (including standard and custom fields).
- **Feature**: Added support for displaying the latest issue `comment` directly in the table.
- **Feature**: Implemented rich HTML/Markdown Tooltips for long comments.
- **Feature**: Added Auto-Refresh capabilities (e.g., every 5, 10, 15 minutes).
- **Feature**: Advanced background API pagination allows exporting reports of any size without the standard 1000-issue limit.
- **Enhancement**: Cleaned up Excel export to strip HTML tags from comments for better readability in spreadsheets.
- **UI**: Added empty states, inline loaders, and polished the RingUI layout and components.
- **Fix**: Resolved horizontal scrollbar issues in native dropdowns and optimized layout spacing.
