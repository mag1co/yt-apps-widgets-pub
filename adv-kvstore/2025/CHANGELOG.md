# Changelog

## v1.1.15

### Versioning

- **Minimum YouTrack version raised to 2025.3** — the in-code permission check uses `User.hasPermission()`, which is available since YouTrack 2025.3. On older 2025.1–2025.2 instances, project scope would be denied for everyone.
- **Last release line for YouTrack 2025** — the next major version, 2.0.0, will require YouTrack 2026. The 1.1.x line stays available for YouTrack 2025 on the **Versions** tab.

### Security

- **Project storage now requires `UPDATE_PROJECT` permission** — project-scoped read/write/delete operations now require the user to have `UPDATE_PROJECT` permission in the target project in addition to group ACL membership. Previously, any member of the configured `aclProjectGroup` who could see the project could read, modify, and delete project storage even with only Read Project access.
- **Project ACL group is now per-project** — `aclProjectGroup` setting is scoped to the project (`x-scope: PROJECT`) instead of being a single global group inherited by every attached project. After updating, configure the allowed group for each project where the app is attached.
- **Global delete/drop/flush endpoints now require `ADMIN_UPDATE_APP`** — the public `/delete`, `/drop`, and `/flush` endpoints in the global scope now require the same `ADMIN_UPDATE_APP` permission as the `/admin/*` counterparts, closing a bypass that allowed group members to perform admin-equivalent destructive operations without admin rights.
- **Admin widget ACL tab reflects the per-project group** — since `aclProjectGroup` is now a per-project setting, the global ACL tab no longer reports it as "Disabled" and explains where to configure it.

## v1.1.13

- **New Project Settings Widget** — introduced a dedicated widget for Project Admins to manage project-scoped storage directly within project settings, resolving `403 Forbidden` backend permission constraints
- **Access Control Integration** — Native YouTrack permissions (`ADMIN_READ_APP` and `UPDATE_PROJECT`) are now enforced for the admin widgets, preventing unauthorized loading
- **UI improvements** — replaced custom confirmation dialogs with standard Ring UI components and fixed widget iframe base styling; fixed an issue where the delete confirmation dialog's layout would break if the block or key name was too long
- **Code Quality** — performed code cleanup and resolved ESLint warnings

## v1.0.7

- **Security fix** — corrected access permissions for admin and project read endpoints; read operations no longer require write-level access
- **UI polish** — "Cleanup Expired" and "Flush All" buttons are hidden when no blocks are stored
- **TTL fix** — invalid TTL values (non-numeric) are now rejected gracefully instead of silently breaking key expiration
- **Stability** — admin widget now shows an error message instead of a blank screen if a render error occurs
- **Cleanup** — removed unused Issue scope from storage schema; fixed broken links in usage documentation
- **Example workflows** — updated `config.js` to use the YouTrack internal URL; added notes on Docker port mapping

## v1.0.3

- **Date formatting from user profile** — admin panel reads date/time format, timezone, and locale from YouTrack user profile (`users/me/profiles/general`); falls back to Intl if unavailable
- **No-wrap dates** — `Updated` column uses `white-space: nowrap` to prevent date wrapping in table cells
- **Documentation cleanup** — simplified README.md (removed duplicated Limitations subsections), streamlined HOW_TO_USE.md (removed inline code duplicates, kept links to example files)
- **Dead code removal** — removed disabled issue scope ACL check from `store.js`, removed vendor placeholders from `settings.json`

## v1.0.2

- **Admin UI refinements** — right-aligned toolbar buttons, thinner table borders per Ring UI guidelines, compact section hints with info icon
- **Ring UI Loader** — native `<Loader squares />` replaces custom loading indicator in all tabs
- **Inline Settings link** — ACL tab shows "App Settings" as inline link with settings icon instead of separate toolbar
- **Scope group headers** — Cleanup Expired / Flush All buttons moved to scope-group header (consistent with Projects tab layout)
- **Refresh left-aligned** — Refresh button stays on the left, action buttons on the right
- **Footer muted** — footer colors and links dimmed for less visual noise
- **Backend JS minification** — backend files minified with terser before ZIP packaging
- **Widget icon** — PNG icon for administration menu
- **Manifest description** — removed reference to disabled issue scope
- **Key value inspector** — click any key in admin panel to view its value inline (global + project scopes), with `setAt` metadata
- **Admin GET endpoint** — `GET /admin/get?block=X&key=Y` returns value + meta, bypasses ACL (`ADMIN_UPDATE_APP` / `UPDATE_PROJECT`)
- **DRY refactor (store.js)** — extracted `_getKeyInternal`, `_deleteKeyInternal`, `_dropBlockInternal`, `_flushInternal`; public and admin handlers are thin wrappers
- **Security: isSafeKey in all admin handlers** — prototype pollution prevention now covers admin delete, drop, and get endpoints

## v0.1.58

- **Admin UI polish** — consistent table column widths, proper toolbar spacing, Ring UI unit-based layout
- **Project-scoped actions** — Drop block, Delete key, Cleanup Expired, and Flush per project in Projects tab
- **Small project buttons** — Cleanup/Flush buttons use compact Ring UI controls height
- **Flush tooltip** — hover hint explains Flush = "Drop all blocks"
- **Ring UI Banner feedback** — success, warning, error banners with modes, titles, and close buttons
- **Mock mode warnings** — simulated actions show warning banners instead of fake success
- **TTL cleanup on writes** — expired keys cleaned lazily; explicit Cleanup Expired button for manual trigger
- **ACL deny-by-default** — scope is disabled (403) when no group is configured; group must be set to enable access
- **Simplified ACL settings** — removed per-scope Enabled checkboxes; ACL is active when group is set, disabled when empty
- **Removed Issue ACL** — issue scope storage works without access control
- **Open Settings button** — ACL tab now has a button to navigate to system app settings
- **Issue scope disabled** — issue scope is blocked (403) until further notice
- **Documentation** — full English README.md with API reference + HOW_TO_USE.md practical guide
- **Security: admin endpoint permissions** — admin endpoints now require `ADMIN_UPDATE_APP` (global) / `UPDATE_PROJECT` (project) at platform level
- **Security: prototype pollution prevention** — block and key names `__proto__`, `constructor`, `prototype` are rejected (400)
- **DoS limits** — max value size 128 KB, max 1000 keys per block, max 100 blocks per scope (413/429)
- **Concurrency** — documented read-modify-write limitation (last-write-wins, eventual consistency)

## v0.1.0

- **Initial release** — key-value storage for YouTrack
- **3 scope levels** — issue (disabled since v0.1.58), project, global
- **Block-based namespaces** — TTL caching and permanent storage
- **HTTP API** — get, set, delete, drop, flush, blocks
- **ACL** — per-endpoint, per-scope access control with group restrictions
- **Admin panel** — view and manage blocks from YouTrack Administration
