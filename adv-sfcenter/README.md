# Adv.ServiceCenter — Service Center / Forms Center for YouTrack

Unified portal for requests, surveys, and forms inside YouTrack — implemented as a native YouTrack App.

Works with YouTrack Server and Cloud.

## Why

Standard YouTrack helpdesk online forms are limited in UI flexibility and logic. Adv.ServiceCenter provides a full-featured service catalog with a schema-driven form engine, multi-step wizards, and field mapping — all running inside YouTrack as a native app.

## Features

- **Service Catalog** — portal with categories, form cards, search, and deep-link URLs
- **Schema-Driven Forms** — 15 field types configured via admin UI, not code
- **File Attachments** — users can upload multiple files using a native RingUI dropzone; uploaded files are reliably attached to created YouTrack issues bypassing sandbox restrictions via Data URI parsing
- **Multi-step Wizard** — single-page and wizard layouts with arrow stepper
- **Field Mapping** — map form fields to YouTrack custom fields with type compatibility checks
- **Static Mapping** — set fixed values on created issues (e.g., Type=Bug, Priority=Critical, Assignee=user); multi-value fields (`enum[*]`, `version[*]`, etc.) use a multi-select picker; user-type fields use a searchable picker with avatars
- **Initial Comment** — optional static comment added to each created issue
- **Hint Text** — any input field can have an optional hint shown between its label and the input
- **Paragraph Formatting** — paragraph fields support bold, italic, and underline; multiline content with Enter
- **Dynamic Options** — select/multiselect/tags fields source options from YouTrack enum fields at runtime; single-value enum fields can be used as a value source for multiselect and tags
- **Option Order** — a field sourcing its options from YouTrack can keep the instance order (default, meaningful for workflow values) or show them A→Z / Z→A; sorting happens in the browser and follows the reader's language
- **Attachments** — file fields with a size limit taken from App Settings (default 10 MB per file, 50 MB per submission); the form shows the limits and rejects an oversized file before uploading; uploaded file names appear in the issue description as links to the attachments
- **Templates** — summary and description templates with `{Field Label}` and `{field_id}` placeholders; the form editor shows both options as inline hints next to each template field, and each field block displays its internal ID as a small badge
- **Auto-generated Description** — leaving the description template empty is a valid setup: the issue description is composed from every answer that is not mapped to a custom field, with attached files listed as links at the end
- **Per-project Settings** — enable/disable service center, manage forms per project; admin endpoints use `UPDATE_PROJECT` permission (Project Admins have full access)
- **Project Isolation** — forms are strictly scoped to their project; Project Admins can only view, edit, and delete forms belonging to their own project
- **Storage Usage Indicator** — the Forms tab toolbar shows how much storage the project's forms occupy, so a project can be noticed before it outgrows the undocumented limit
- **Backend Issue Creation** — issues created server-side, no permanent tokens needed
- **Access Levels** — four visibility levels per form: Project Members, Link Only, All YouTrack Users, Guests (external)
- **Access Level Tags** — admin form list shows a colored Ring UI Tag per access level: purple (project), yellow (yt-users), red (public/guests), grey (link-only); status icon (eye/eye-crossed) shown inline
- **ACL Group Guard** — restrict who can create `All YT Users` / `Guests` forms via configurable YouTrack groups in App Settings; server-side validation on save
- **Form Authorship** — `updatedBy` / `updatedAt` tracked per form and displayed as subtext in the admin forms list
- **Guest Access** — guest-accessible forms with email collection and issue tracking
- **Guest Tracking Page** — stateless tracking links for guests to monitor issue status and updates
- **Reporter Mode** — choose who creates the issue: current user (default) or a fixed login (service bot); guest forms enforce fixed reporter
- **Guest Preview** — admin toggle to preview the portal as a guest user, with a warning banner
- **Guest Field Options Filter** — per-field include/exclude options for guest users on select/radio/tags fields
- **Conditional Static Mappings** — static mappings with a "Guest only" flag, applied only for guest submissions
- **Backend Form Validation** — on every save the backend checks guest forms for security violations (e.g. user-type and group-type source fields); sets a `validated` flag stored alongside the form; unvalidated forms are hidden from guests in catalog and blocked at direct URL — even if the frontend check is bypassed
- **Auto-unpublish on security violation** — a guest form that fails backend validation is automatically unpublished; admin sees a warning icon in the forms list and an inline error banner in the editor listing the offending fields
- **Publish/Unpublish UX** — clicking publish or unpublish shows an inline spinner in the status column; the forms list updates in place without a full reload
- **Template Placeholder Hints** — summary/description/comment template fields show a live list of available `{Field Label}` placeholders derived from the current form fields; guest forms additionally show `{__guest_email}`
- **Demo Data Prefill** — one-click demo data with sample categories, forms, and a multi-step showcase
- **Form Validation** — required field checks with red borders and inline error messages
- **My Requests** — authenticated users can view all issues they submitted via Service Center. Shows summary (clickable link), form name (with fallback for deleted forms), and creation date. Loaded page by page with a page size selector, ordered by creation date (newest first). Accessible via the "⋯" menu in the portal toolbar.
- **Catalog View Modes** — categories, grouped, and list views with dedicated icons
- **Ring UI** — native YouTrack look and feel using JetBrains Ring UI components

## Guest Mode — Setup & Limitations

### How to enable

1. In **Project Settings → Service Center**, open the form editor
2. Set **Access → Guests (external)**
3. Configure **Reporter**:
   - **Current user (default)** — issue is created as the Guest system user. Requires the YouTrack project to grant Guest permission to create issues.
   - **Fixed login** — issue is created on behalf of a specified user (e.g. a service bot). Use this when the project does not allow Guest to create issues directly. The specified user must exist in YouTrack and have permission to create issues in the target project.

### How it works

- An automatic **email** field is added to the top of the form
- User-type fields (Assignee, Verified by, etc.) are hidden from mapping to protect sensitive data; group-type fields are not supported by the app at all
- If the guest's email matches a registered YouTrack user, that user is subscribed to the issue (receives notifications)
- If no match is found, the email is stored in the issue description for manual follow-up
- Guests receive a **tracking link** (HMAC-SHA256 token) to monitor issue status without YouTrack access
- On every save the **backend validates** the form and sets a `validated` flag; forms that fail validation are automatically unpublished and hidden from guests at both catalog and direct URL level

### Limitations

- **No direct access** — after submission, a guest cannot view the issue in YouTrack (unless their email corresponds to a registered user via LDAP/SSO/Hub)
- **No user creation** — the app cannot create new YouTrack users (httpHandler sandbox blocks outgoing HTTP)
- **No user-field mapping** — Assignee, Verified by, etc. cannot be set when the submitter is a guest
- **Project permissions** — if Reporter is set to "Current user" but the project forbids Guest from creating issues, submission will fail with 403. Solution: switch to "Fixed login"

### Security

- **Backend access control** — `/catalog`, `/forms/get`, and `/submit` enforce `accessLevel` checks via a unified `checkFormAccess()` method. The `project_members` check relies on `project.team.users.has()` rather than simple read-access to the project. Guests cannot retrieve the form structure or create issues for `project_members` / `youtrack_users` forms, even if they know the exact slug.
- **Tracking token** — Uses HMAC-SHA256 (256-bit) with a 32-character installation salt generated via `java.util.UUID.randomUUID()` (SecureRandom CSPRNG). Fallbacks to `Math.random()` only if Java classes are unavailable.
- **Tracking response** — Returns only the issue summary, state, priority, dates, and the values the guest submitted. The description and project name are withheld, and the guest's own email address is never served back: the link is valid for whoever holds it, so it must not disclose anything the guest did not already provide.
- **User-type fields** — Hidden from dynamic mapping and source field selection in guest mode (guests cannot see the list of users). They remain available in static mappings, which are applied server-side and invisible to guests.
- **Group-type fields** — Not supported on any access level: group names describe who has access rather than what a request is about, and YouTrack gives apps no way to offer them as a list to pick from. Sources, dynamic mappings and static values targeting a group field are dropped when the form is saved, and the submit path refuses to write such a field. Use a YouTrack workflow rule instead.
- **Guest field filter** — Per-field include/exclude value configurations to restrict the visibility of dropdown options for guests.

## Access Levels

Each form has an **Access** setting that controls who can see and submit it:

| Level | Catalog | Submit | Badge |
| --- | --- | --- | --- |
| **Project Members** (default) | Members only | Members only | — |
| **Link Only** | Hidden | Any logged-in user | `link only` |
| **All YouTrack Users** | All logged-in users | Any logged-in user | `all users` |
| **Guests (external)** | All users incl. guests | Anyone | `public` |

### Project Members (default)

Form is visible and submittable only by members of the linked YouTrack project team (`project.team.users`). Enforced server-side — non-members receive 403 even if the project is publicly readable.

### Link Only

Form does not appear in the service catalog. Any authenticated YouTrack user can open and submit it via a direct link (deep-link URL). Use this for internal forms that should not be publicly browsable.

### All YouTrack Users

Form appears in the catalog for all logged-in YouTrack users regardless of project membership. Guests (unauthenticated) do not see it. No project membership check on submit.

### Guests (external)

Form is publicly accessible — visible in catalog and submittable without authentication. An automatic email field is added to the form. See **Guest Mode** section for setup details and limitations.

> **Note on project permissions:** For `Link Only`, `All YouTrack Users`, and `Guests` access levels, the target YouTrack project must be readable by the submitter's effective identity. If it is not, issue creation will fail. Use `reporterMode: Fixed login` with a service account that has write access to the project.

## Extension Points

| Extension Point | Purpose |
| --- | --- |
| `MAIN_MENU_ITEM` | Service Center portal page in sidebar |
| `PROJECT_SETTINGS` | Per-project form management and configuration |

## Requirements

- **YouTrack 2025.1+** — minimum version
- **YouTrack 2026.1.1+** — recommended for full functionality.
  Deep-link navigation (direct URLs to forms, browser back/forward support) uses the `host.navigation` API introduced in YouTrack 2026.1.1.
  On older versions the app still works, but deep-linking and back/forward navigation are unavailable.

## Tech Stack

- React 18 + TypeScript
- Ring UI (JetBrains component library)
- Host API (`host.fetchApp`, `host.fetchYouTrack`, `host.navigation`)
- Vite build

## Settings

Defined in `settings.json`:

| Setting | Type | Default | Description |
| --- | --- | --- | --- |
| Log Level | `enum` | `no_log` | Backend logging: `no_log`, `minimal`, `debug` |
| ACL: Who can create 'All YT-Users' forms | `UserGroup` | _(none)_ | Members of this group may set 'All YouTrack Users' access on forms. Leave empty to disable the option entirely. |
| ACL: Who can create 'Guests' forms | `UserGroup` | _(none)_ | Members of this group may set 'Guests (external)' access on forms. Leave empty to disable the option entirely. |
| Attachments: max size per file, MB | `integer` | `10` | Limit for files submitted through forms — YouTrack does not apply its own `Max upload file size` to files attached by apps, so set this to the same value. Values above 50 are capped; a single submission is additionally bounded by 50 MB in total. |

## License

Proprietary. See [LICENSE](LICENSE) for terms.

## Vendor

- **Author**: mag1cc
- **Source**: [GitHub](https://github.com/mag1co/yt-apps-widgets-pub/tree/main/adv-sfcenter)
