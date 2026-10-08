# Adv.Weekly Report for YouTrack

A powerful, highly configurable YouTrack widget that generates grouped reports of issues and allows exporting them directly to XLSX, complete with comments and clickable links.

Works as both a Dashboard Widget and a Project Tab widget.

> The docs for the current generation (**YouTrack 2026.2+**) live in [`2026`](../2026/). This folder documents the **last release line for YouTrack 2025.1–2026.1** (v1.1.x).

## Key Features

- **Custom JQL/Query** — Build reports based on any YouTrack search query directly from the settings.
- **Project Context** — Automatically load project-specific custom fields. Auto-detects the project directly from your search query!
- **Dynamic Columns** — Select exactly which fields to display in your report, including standard fields, custom fields with aliases (e.g., `State (S)`), and descriptions.
- **Rich Comments & Descriptions** — View the latest issue comment or the full issue description directly in the table with safe HTML/Markdown rendering via tooltips.
- **Comprehensive XLSX Export** — Instantly export the current report view to an Excel spreadsheet with clickable hyperlinks and clean, markdown-stripped plain text.
- **High Performance** — The main dashboard loads instantly with zero lag thanks to heavily optimized YouTrack API requests.
- **Deep Pagination** — Handles large datasets flawlessly by streaming all matching issues via the YouTrack API without artificial limits.
- **Auto-Refresh** — Keep your dashboards up to date with configurable auto-refresh intervals.
- **Browser-Native Export** — The XLSX generation runs entirely in your browser. No data is sent to external servers.

## Getting Started

### 1. Install the app
Install from YouTrack Admin → Apps by uploading the `.zip` file from the `dist_zip` folder.

### 2. Add to Dashboard or Project
- **Dashboard**: Edit your dashboard, add a new widget, and select "Adv.Weekly Report".
- **Project**: Navigate to any project and open the new "Adv.Weekly Report" tab.

### 3. Usage
Click **Edit** on the widget to open its settings. Here you can configure your YouTrack Query, select a **Project Context** to load project-specific custom fields, choose the columns you want to display, and set the number of historical periods (weeks) to show. 
Click **Export to XLSX** on any period to download the report as an Excel file.

## License

Proprietary. See [LICENSE](LICENSE) and [EULA.md](EULA.md) for terms.

## Vendor

- **Author**: mag1cc
- **Source**: [GitHub](https://github.com/mag1co/yt-apps-widgets-pub/tree/main/adv-wkreporter/2025)
