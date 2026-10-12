# Changelog

## v1.9.25

### Versioning

- **Minimum YouTrack version raised to 2025.3** — the manifest now requires YouTrack 2025.3 or later.
- **Last release line for YouTrack 2025** — the next major version, 2.0.0, will require YouTrack 2026. The 1.9.x line stays available for YouTrack 2025 on the **Versions** tab.

### Security

- **Per-attachment visibility enforced in all native endpoints** — the widget's list, download and delete endpoints read attachments through the server-side entity API, which does not apply per-attachment visibility (permitted users/groups). A user who could not see a restricted attachment in the YouTrack UI could list it through the widget and read its content; the only check was the app's own lock map, and it was active only in Sensitive mode. Every native endpoint now compares the current user against the attachment's permitted users/groups. Restricted files disappear from the list and read back as "not found" instead of leaking their metadata or content. Found in a JetBrains YouTrack app audit.
- **Attachment deletion requires the Delete Attachment permission** — the delete endpoint only required Update Issue, so a user could delete attachments they had no permission to delete natively, including attachments hidden from them. The endpoint now requires the same project permission as YouTrack's native attachment deletion (`DELETE_ATTACHMENT_ISSUE`), re-checks it in the handler, and enforces per-attachment visibility. Found in a JetBrains YouTrack app audit.
- **`meta/set` no longer accepts lock state** — the per-issue metadata endpoint wrote the client payload into the issue extension property that also held the lock maps, so any user with Update Issue could clear the locks and then read the locked files. With the Sensitive Files feature switched off (see below) the property holds client file metadata only; legacy lock keys from older versions are dropped on read and write.

### Removed

- **Sensitive Files feature switched off in the 1.9.x line** — per-file lock/unlock, Sensitive mode, access fields and groups, permanent locks and access-log comments no longer activate: the feature mode is forced to "disabled, clear" in code, and the related settings are gone from app settings. The feature is not deleted: it stays in the 2.0.0 line for YouTrack 2026 and will be reworked there. Reason: its lock state lives in an issue extension property that any user with Update Issue access can read and overwrite, so it cannot be patched as a security fix in place.
- **Existing locks are lifted automatically** — the sync endpoint runs the built-in disable migration: the first time the widget is opened on an issue by a user with Update Issue permission, file visibility restrictions are cleared, the lock maps and the `!_LOCK_!` tag are removed, and `hasLockedFiles` is reset. Issues that are never opened keep their old restrictions; the widget reads them correctly either way. The tag can also be removed manually at any time — the workflow that protected it ships no longer.
- The related settings are gone from the app settings: Sensitive Files Mode, Access Field, Access Groups, Deny Groups, State Field, Reveal States, Permanent Lock Trigger, access-log comments and the sync poll interval. The Performance Profile setting is unchanged and now drives only the native list refresh interval and list TTL.

### Fixes

- **RAR5 archives could not be downloaded or previewed from the widget** — clicking download did nothing (no error was shown) and preview failed with a browser error. Only this archive format was affected; downloading from the standard YouTrack attachments panel always worked. Both download and preview now work for all file types.
- **File-operation comments silently failed on YouTrack 2026.x** — upload/delete audit comments were dropped with an internal NPE when no Comment Author (bot user) was configured. Comments now post as the current user in that setup.

### Improvements

- **Internal cleanup** — download, preview and thumbnail paths now share a single module for fetching native attachment content, replacing three divergent copies of the same logic.
- **Settings cleanup** — removed dead/unreachable settings: `presignNoCache` (cache-busting is now hardcoded on — no legitimate off scenario), `s3EncryptedConnectionString` (deprecated legacy fallback key), `debugSigV4Logging` / `debugFrontLogging` (were referenced in code but absent from `settings.json`, so could never be enabled; SigV4 debug logging remains available via `logLevel = debug`)
- **No log spam in native-only installations** — missing S3 connection string is logged at `debug` instead of `warn`.

## v1.7.37

### Security

- **XSS fix in markdown preview** — HTML escaped before transformations; links and images sanitized with URL allowlist (`http(s):`, `data:image/*`), blocking `javascript:` and other dangerous schemes
- **`generateUUID` / `generateIV` weak randomness** — both now use `crypto.getRandomValues` with `Math.random` fallback; UUID updated to RFC 4122 v4

### Sensitive Files

- **Setting: Sensitive Files: Mode** — new per-project setting: `enabled` — locking active; `disabled_clear` *(default)* — feature off, existing locks removed automatically; `disabled_leave` — feature off, locked files stay locked until manually unlocked
- **Fix: files stayed locked after feature was disabled** — disabling now removes all attachment restrictions on next sync (`disabled_clear` mode)
- **Fix: lock button and file visibility did not update after Assignee change** — permissions re-evaluated on every sync poll and after file list load; safeguard against runaway re-sync chains added
- **Fix: performance issue with large user groups** — group membership check uses direct server-side lookup instead of loading all members into memory
- **Fix: locked files not shown after page load** — logic error prevented lock sync from running after successful file list load
- **Fix: admin bypass missing** — system admins can now always toggle locks regardless of group membership
- **Fix: sensitive feature not activating** — YouTrack returns array settings as managed collections; feature now detected correctly
- **Fix: stale lock entries** — deleted attachments left orphan lock records; sync now removes them automatically

### Fixes

- **Alert fallback** — RingUI alert fallback now correctly uses `warning` and `error` methods; removed non-existent `successMessage` call
- **`maxFileSizeMB` schema cap** — raised from 50 MB to 2048 MB to match "Set to 0 for unlimited" description

### Improvements

- **Delete confirmation** — Cancel is now the default button in delete dialogs to prevent accidental deletion
- **Timing diagnostics log level** — presign, upload, metadata and finalize timing logs downgraded from `warn` to `debug`
- **Dependencies** — updated to current versions

## v1.6.0

- **Internal refactoring** — improved code modularity and reduced complexity for faster future development
- **Fix: delete handler could act on stale selection** — resolved a rare edge case where deleting a file could clear the wrong selection
- **New issue draft support** — the panel no longer shows errors when creating a new issue; a friendly message is displayed until the issue is saved
- **Stability improvements** — minor internal fixes to file path handling

## v1.5.2

- **Zero configuration** — with `clockSkewAdjustSec = 0` (default) the widget automatically detects the time difference between the server and S3 and compensates all presigned URLs. Frontend now probes S3 directly and sends the time to the backend for compensation
- **Instant calibration** — S3 time is fetched before the first file listing, skew is applied immediately
- **Self-healing on 403** — if a presigned URL expires, frontend re-probes S3 time and regenerates the URL
- **Skeleton loading** — file cards show placeholder skeletons while S3 metadata loads, eliminating layout shift
- **Refactoring** — deduplicated shared logic across presign handlers and admin UI; fixed async handler reliability; extracted inline constants to module level

## v1.4.71

- **Fix: timer cascade in background tabs** — hidden tabs no longer accumulate retries every 16 seconds; refresh happens once instantly when user returns to the tab (seamless, no spinner)

## v1.4.69

- **Fix: 403 "Request has expired" on S3 operations** — automatic clock skew detection between YouTrack server and S3 was not working; now all presigned URLs are automatically compensated for server time drift
- **Reliability** — clock skew probe has a 5-second timeout to prevent slowdowns if S3 endpoint is temporarily unreachable
- **Setting: Clock Skew Adjustment** — `0` (default) = automatic detection, non-zero = manual override

## v1.4.67

- **Fix: precise native attachment deletion** — backend now matches by `name` + `created` timestamp instead of name-only; correctly deletes the exact file even when duplicate names exist. If exact match not found, returns 404 instead of silently deleting a wrong file
- **Native refresh interval** — extracted into `NATIVE_REFRESH_INTERVAL_MS` constant (60s) for easier tuning
- **UX: sequential loading flow** — settings load first (with spinner), then native files shown immediately, S3 files loaded in background with inline loader
- **Fix: infinite spinner when S3 not configured** — spinner now reliably clears after native files load; API calls wrapped with timeouts to prevent hanging
- **Fix: S3 config error no longer shown as user error** — missing S3 credentials silently skipped, EmptyState or native files displayed correctly
- **Fix: no file list flicker on silent refresh** — S3 files merged without replacing already-displayed native files
- **Default sort changed to oldest-first** — prevents visible reordering when S3 files load after native
- **Preview: download progress for archives** — ZIP/TAR/GZ previews now show real-time download progress
- **Fix: no more security warning on native attachment delete** — delete now goes through backend proxy instead of direct REST API call, eliminating the YouTrack "DELETE request to issues endpoint" trust warning
- **Fix: overlay icons not appearing on wrapped grid items** — hover icons (delete/download) now always mounted in DOM
- **CSS optimization** — removed ~40 redundant fallback values for Ring UI variables guaranteed by YouTrack iframe predefined styles; replaced hardcoded colors with Ring UI variables in admin widget
- **Refactoring** — deduplicated progress-reporting fetch logic across preview renderers
- **Docs: S3/MinIO CORS fix** — updated allowed origins in all CORS setup examples

## v1.4.52

- **Fix: 404 noise for issues without S3 files** — repeated `404 Not Found` errors in the browser console eliminated for issues that have no uploaded files
- **Fix: no more retry spam when S3 is not configured** — no more exponential backoff retries when S3 credentials are missing; one log entry, no retry loop
- **Optimization: skip S3 when not configured** — if S3 has no setup, presign/list is skipped entirely, only native attachments are shown (with 2-min auto-refresh)

## v1.4.37

- **Date formatting** — all dates now respect the user's YouTrack profile settings: timezone, date format pattern, 12/24h preference, and UI language; fetched once via REST API and cached for the session
- **Cleanup** — removed dead code: unused handlers, stale imports, deprecated helpers, orphaned utility module
- **Settings** — renamed connection string key with backward-compatible fallback; clarified that stored credentials are obfuscated, not encrypted, and readable by instance admins (per JetBrains Marketplace review feedback, see JT-88186); updated UI labels and documentation

## v1.4.34

- **Security** — removed insecure endpoint, hardened delete handler (server-side clock skew only)
- **Security** — added empty prefix guard in presigned URL generation
- **Performance** — S3 thumbnails no longer re-fetched on a timer; once loaded, they stay in browser cache permanently. Expired presigned URLs are retried on demand (only on `<img>` error), eliminating periodic thumbnail traffic entirely
- **Performance** — backend settings cached (30s TTL + inflight dedup) instead of per-thumbnail HTTP request
- **Performance** — concurrent presign requests for the same file key deduplicated via inflight guard
- **Performance** — React state updates skipped when nothing actually changes in thumb invalidation
- **Performance** — metadata write skipped on auto-refresh when counters unchanged (saves 3–5 HTTP requests/min in steady state); `storageId` fetch cached from meta
- **Performance** — deduplicated presign setup and backend settings loading across all call sites
- **Preview** — XLS and XLSB support via full SheetJS library (bundled); mini build used for XLSX
- **Preview** — text-based file thumbnails rendered as canvas previews (code, configs, logs, CSV, XML, etc.)
- **Reliability** — in-browser mutex serializes all meta read-modify-write cycles, preventing lost updates when upload and auto-refresh overlap
- **Native attachments** — upload/delete now logged to issue comments (same as S3), unified comment format
- **Cleanup** — removed all debug infrastructure and unused backend handlers from production bundle

## v1.3.476

- **Encryption hardening** — per-installation dynamic key (`a3x:` format) with backward-compatible fallback
- **Meta storage in YouTrack** — per-issue metadata moved to extension properties; bidirectional S3 ↔ YT lazy migration; `metaStorage` setting (`youtrack` / `s3`)
- **Document previews** — bundled local libs (no CDN); XLSX multi-sheet tabs; odf-kit replaces webodf (AGPL → Apache-2.0)
- **UI/UX improvements** — dark theme for thumbnails and previews; list view column alignment; typed thumbnail backgrounds
- **Build & backend cleanup** — debug files excluded from production; deduplicated code, dead code removal

## v1.3.400

- **Unified File View** — browse native YouTrack attachments and S3 files in one widget
- **Preview Without Download** — preview native attachments directly
- **Full native attachment management** — list, preview, download and delete
- **Progressive loading** — non-blocking UI, inline loaders, parallel fetch
- **JSON preview** — syntax-highlighted, auto-formatted
- **Markdown preview** — rendered with proper formatting
- **ZIP preview** — browse archive contents without downloading
- **Colored badges** — file-type badges with distinct colors
- **Renamed** — widget renamed to Adv.Attachments
- **Widget permissions** — added CREATE_ATTACHMENT_ISSUE, DELETE_ATTACHMENT_ISSUE

## v1.2.300

- **Configurable file size limits** — min/max upload sizes in MB with user-friendly messages
- **File operation comments** — automatic issue comments on upload/delete with grouping
- **Comment visibility control** — optional restriction to specific user groups
- **Backend minification** — all backend JS minified with Terser for smaller package

## v1.2.x

- **No credentials leak** — all sensitive data redacted from logs regardless of level
- **Auto clock skew detection** — transparent time sync between YouTrack and S3
- **Zero-flicker auto-refresh** — silent background updates every 60 seconds
- **Persistent thumbnails** — image previews preserved between refreshes
- **Lazy loading** — images load on-demand as you scroll
- **Office files preview** — XLSX, DOCX, ODT preview directly in YouTrack
- **Reorganized settings** — logical groups for S3, TTL, Widget UI
- **Modular backend** — refactored into organized modules

## v1.0.x

- **Initial Release**: S3-compatible storage integration for YouTrack Server with support for Amazon S3, MinIO, Wasabi, DigitalOcean Spaces, Backblaze B2, and any S3-compatible service.
- **Grid and List Views**: Browse attachments in modern grid view with thumbnails or compact list view. Automatic switching based on file count (default threshold: 20 files).
- **Direct File Operations**: Upload, download, and delete files directly via AWS Signature V4 presigned URLs without proxy overhead.
- **Improved Empty State**: When no files attached, header and actions are hidden, Attach button is prominently displayed with proper spacing.
- **24-Hour Time Format**: File timestamps always displayed in 24-hour format for consistency.
- **Enhanced Logging**: Detailed DEBUG logging for presigned URL operations helps troubleshoot connection issues with S3 providers.
- **Unified Request Handling**: Consistent request body parsing across all operations for reliability.
- **Security**: Implemented least-privilege permission model — read operations require READ_ISSUE, write operations require UPDATE_ISSUE, admin operations require ADMIN_READ_APP or ADMIN_UPDATE_APP.
- **Path Normalization**: Unified key building with base directory support to prevent double-prefixing and ensure correct file paths.
- **Centralized Presign Logic**: All presign URL requests and anti-cache handling centralized for maintainability and consistency.
