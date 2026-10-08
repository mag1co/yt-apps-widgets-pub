# Adv.DocExporter for YouTrack

Export Knowledge Base article tables and content to **XLSX** and **DOCX** files directly from YouTrack.

Works with YouTrack Server and Cloud.

> The docs for the current generation (**YouTrack 2026.2+**) live in [`2026`](../2026/). This folder documents the **last release line for YouTrack 2025.3–2026.1** (v2.0.x).

## Features

- **XLSX Export** — Export article tables to Excel with bold headers, hyperlinks, and rich text formatting.
- **DOCX Export** — Export full article content to Word with headings, lists, tables, inline formatting, and hyperlinks.
- **Raw Export Mode** — Plain text export without formatting for clean data extraction.
- **Markdown Parsing** — Automatic table detection from Markdown and HTML content with YouTrack escape handling.
- **Multiple Tables** — Export all tables from a single article into one file, each on a separate sheet (XLSX) or section (DOCX).

## Getting Started

### 1. Install the app

Install from YouTrack Marketplace or upload manually to your YouTrack instance.

### 2. Attach to a project

Go to YouTrack Admin → Apps → Adv.DocExporter and attach it to the desired project. No additional configuration required.

### 3. Export

Open any Knowledge Base article — a new menu item will appear in the article options menu (⋯):

- **Export article...** — Opens an interactive dialog to select format (XLSX/DOCX), content type, and text formatting options.

The file downloads automatically.

## Export Settings

### Formats
- **Excel (XLSX)** — Exports tables on separate sheets.
- **Word (DOCX)** — Exports full article content (headings, text, tables) or tables only.

### Text Formatting Options
- **Keep formatting (Checked by default)** — Preserves inline styling from the article:
  - **Bold**, *italic*, ~~strikethrough~~ text
  - Hyperlinks (clickable in Excel/Word; unsafe schemes like `javascript:` and `file://` are filtered)
  - Table headers with bold formatting
- **Raw Export (Unchecked)** — Strips all formatting, outputs clean plain text. Useful for data pipelines and further processing.

## Architecture

The app consists of:

- **1 Widget Module** — Export Dialog (article options menu item)
- **Shared Table Parser** — Markdown/HTML table extraction with rich text support
- **File Generator** — XLSX/DOCX generation (ZIP + XML) runs entirely in the browser
- **Shared Utils** — ZIP generation, CRC32, UTF-8 encoding

## License

Proprietary. See [LICENSE](LICENSE) for terms.

## Vendor

- **Author**: mag1cc
- **Source**: [GitHub](https://github.com/mag1co/yt-apps-widgets-pub/tree/main/adv-docexporter/2025)
