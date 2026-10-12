# Changelog

## v2.8.10

### Versioning

- **Minimum YouTrack version raised to 2025.3** — the manifest now requires YouTrack 2025.3 or later. 
- **Last release line for YouTrack 2025** — the next major version, 3.0.0, will require YouTrack 2026. The 2.8.x line stays available for YouTrack 2025 on the **Versions** tab.

### Security

- **Project settings can no longer address another project** — `POST project/settings` used the client-supplied `internalId` as the address of the write. YouTrack checks the *Update project* permission against the project in the URL only, and the internal id is short, sequential (`0-0`, `0-1`…) and public through the catalog, so an admin of any project could disable another project's portal or overwrite its entry with their own key. The id is now resolved server-side (from the enabled-projects map or the legacy per-project record), and a client-supplied id is accepted only for a first enable, where the entry is still free — anything occupied by another project is refused with 403, the same rule the sibling `updateKey` endpoint has always applied.

### Features

- *Smart option order* — the **Option order** setting gains a **Smart** mode: natural A→Z where numbers compare by value ("Item 2" before "Item 10", "v1.9" before "v1.10"), with catch-all answers such as "Other", "N/A", "Другое" or "Затрудняюсь ответить" kept at the end. A→Z and Z→A now use the same natural comparison instead of character-by-character order, and ignore case.
- *Option order for manual options* — the order setting is now available for options typed in by the form author, not only for options taken from a YouTrack field. The default keeps the order as entered.
- *Translated values are shown as translated* — options taken from a YouTrack dropdown field now display the value's localized name when the YouTrack admin has set one ("Задача" instead of "Task"), and fall back to the name otherwise. Sorting follows the displayed text, and the issue summary, description, comment templates and the tracking page show the same text the submitter picked; the issue field itself still receives the original value.
- *Static values: tags* — *Static Values* can add tags to every issue created from the form, one row per tag, in two ways: **Tag — pick existing** chooses from the tags you can see, **Tag — by name** takes a typed name and, if no such tag exists, creates it when the form is saved — owned by you and shared with the group you pick (the project team by default). The tag is fixed when the form is saved, so it reaches every issue whoever submits the form — including guests and people outside the team who cannot see the tag themselves. A renamed tag keeps working; a deleted one is skipped with a note in the submission result and the form is flagged in the admin list; a tag recreated under the same name is picked up again automatically. Only tags you may apply yourself can be added to a form.
- *Forms out of sync with the project are flagged* — the admin form list shows a warning triangle when a form refers to something the project no longer has: a detached field, a deleted or archived value, a user removed from a field or banned, a deleted tag, an option source with no values left, a fixed reporter who was deleted or banned. Problems come in two levels. A form that cannot do its job — its fixed reporter is gone, or a required field lost its option source or its target field, so it cannot be filled in or its answer would be lost — is closed on the portal: people who open it see that it is misconfigured and should contact the project administrator, and submissions are refused, including direct API calls. Its row is tinted red. Other problems leave the form working, only the affected values are skipped; such rows are tinted orange. The tooltip lists both groups. Opening such a form in the editor shows the same lists at the top — a red one for what keeps the form closed, an orange one for skipped values — grouped by section, with links that expand the section to fix; inside the sections the affected static rows and option sources are highlighted, and mappings dropped because their field is gone are named instead of disappearing silently.

### Fixes

- *Conditional fields no longer make submissions fail* — a YouTrack field configured to appear only when another field has a certain value (for example, *Components* shown only for *Priority = Major*) rejected any value while hidden, and the whole submission failed with "Cannot set value to custom field". The app now writes such a field once the mappings that open it have been applied, regardless of their order in the form; if it stays hidden, the value is skipped with a note in the submission result and the issue is created.
- *The admin form list no longer breaks on legacy data* — YouTrack forbids changes while serving a read request, but loading the form list could write in three cases: migrating storage from an older version, and recording the security check of guest forms saved before that check existed. Any of them made the whole list fail to load. The list now only reads; the pending changes are applied by a separate request the admin page sends automatically, and the "Migrate now" button performs the storage migration again. The guest tracking page had the same flaw on an instance where no request had been submitted yet.
- *A deleted or archived static value is reported instead of silently dropped* — when a fixed value of a dropdown field disappeared from YouTrack, the form kept submitting without it and nobody noticed. The value is now skipped with a note in the submission result; the rest of the issue is filled in as before.

### UI

- *My Requests is a button in the portal toolbar* — it used to be hidden in the "More options" menu next to the catalog search; it is now a visible button, shown even when no forms are published. The menu keeps the guest preview.
- *Dropdowns close when clicking outside the app* — an open dropdown (for example a value picker in *Static Values*) stayed open after a click on the surrounding YouTrack page, and Esc stopped working too: the app runs in its own frame and never saw that click. Dropdowns now close as soon as the app loses focus, in both the admin page and the portal.
- *User search in static values* — the user picker lists people as "Full Name [login]" and finds them by login, by full name or by any part of it, in any word order ("ivan pet", "petrov", "ipetrov"); "ё" and "е" are treated as the same letter. Multi-user fields get a multi-select picker with the same search.
- *One label format in static value pickers* — what people read comes first and the stored key follows in square brackets: "Ivan Petrov [ipetrov]", and for translated values "Важный [Major]" (previously "Major (Важный)"). Square brackets keep the key apart from names that contain parentheses.
- *Access level is color-coded in the portal catalog* — cards, grouped-list rows and table rows carry a colored stripe matching the access tags in the admin list (project members, all users, guests), with the access level in the tooltip. Guests do not see it: every form they can open has the same level.

## v2.7.4

### Fixes

- *Attached files are readable in the issue description* — a file upload field reached the description as `[object Object]`, because the auto-generated text stringified the file objects instead of reading their names. Every uploaded file is now listed as a download link. The same applied to a `{field}` placeholder pointing at an upload field in a description, summary or comment template, which resolved to `[object Object]` as well; it now yields the file names.
- *Empty upload fields show a dash* — an upload field left blank produced an empty value in the auto-generated description instead of the `—` used by every other field.

### UI

- *Textarea fields show several lines* — multiline text fields in the portal were rendered as a single-line input that the user had to drag to enlarge. They now default to a height of several rows and remain resizable vertically.
- *Attachments are listed at the end of the description* — the auto-generated description placed upload fields wherever they appeared in the form, mixing file lists into the answers. They are now grouped after the text fields.
- *Loading indicator in the form editor* — the *Field Mapping* and *Static Values* sections showed a bare "Loading custom fields…" line while the project's fields were being fetched, which read like a stuck editor on a large project. They now show a spinner.

## v2.7.3

### Fixes

- *Empty description template falls back to auto-generation* — a description template that is empty or contains only whitespace used to be treated as a real template, so the issue description came out as whitespace instead of the list of unmapped fields. It is now treated as unset and the auto-generated description is used again.

## v2.7.2

### Fixes

- *Catalog loads faster on large instances* — the catalog listing filtered every form through a project-membership test, and each form of the same project asked the identical question: thirty forms across five projects ran thirty checks instead of five. The membership test involves a project lookup and a team scan, which on a corporate instance means thousands of entries, so the portal's first screen paid for it on every load. The answer is now computed once per project per request.
- *My Requests resolves projects without rescanning* — each request in the list translated its project key by scanning the whole map of enabled projects; with a long history of requests the scan repeated hundreds of times. The map is now inverted once per request, turning every lookup into a constant-time one.

### Security

- *Attachment size limits* — YouTrack's own `Max upload file size` is not applied to files attached by apps, so form submissions were effectively unlimited: a 27 MB file went through on an instance capped at 3 MB, and a 157 MB one crashed the server with `Java heap space`. Submissions are now bounded by a new global setting *Attachments: max size per file, MB* (default 10, mirrors YouTrack's own default) plus a 50 MB total per submission, and oversized requests are refused before the body is parsed.
- *Guest option filters are enforced on the server* — the per-field "Guest filter", which hides selected options from external users, was only applied while drawing the form: a guest posting directly to the API could still submit a hidden value. Such submissions are now rejected.
- *Guest email is no longer served by the tracking link* — the address a guest types in a form used to be stored on the issue and returned by the tracking page to anyone holding the link. It now stays in the issue description only, where support reads it; requests created earlier are filtered when the tracking page is opened.
- *A project admin can no longer delete another project's form* — form bodies live in per-project storage, so a delete request from a foreign project's scope found no body, the ownership guard never fired, and the form's entry was silently stripped from the shared global index — making the form vanish from that project's admin list and the portal catalog. Ownership is now verified against the index entry and the request is rejected with 403.
- *Mappings to user and group fields are enforced when a request is submitted* — a mapping is checked while the form is saved, but a target that does not exist in the project yet is kept on purpose. Attaching a user or group field with that name later made the mapping live: the form itself was refused for guests, while a submission still filled the field in. Both paths now apply the same rule.

### Features

- *Group fields are not filled in by forms* — a form cannot set a user-group field of an issue, neither from an answer nor as a fixed value: group names describe who has access rather than what a request is about, and YouTrack gives apps no way to offer them as a proper list to choose from. A form saved earlier with such a value keeps working — the value is simply dropped the next time the form is saved. Use a YouTrack workflow rule to set such a field.
- *Choose the order of options taken from a YouTrack field* — a field that sources its options from the instance now has an **Option order** setting: as in YouTrack (default), A→Z or Z→A. The default keeps the order configured in YouTrack, which is the meaningful one for workflow values such as To do → In Progress → Done; alphabetical is what long people lists usually want. Sorting happens in the browser, so it follows the reader's language. Existing forms are unaffected — no setting means the previous behaviour.
- *My Requests: pagination* — the list is loaded page by page instead of returning every request at once, with a page size selector and ordering by creation date. Only the visible rows are assembled, which on a test instance with 274 requests removes roughly 150 ms from every call. Sorting by form or summary was removed: those columns could only be ordered within one page, which was misleading.
- *Admin: forms storage usage indicator* — every form body of a project lives in a single storage value whose size limit YouTrack does not document, and a project that grows past it stops accepting saves. The Forms tab toolbar now shows the currently occupied size next to the New form button, updated as forms are saved and deleted, so a growing project can be noticed before it becomes a problem.
- *Mappings: a radio field can now target a Build field* — the compatibility matrix allowed this for dropdowns but not for radio buttons, with no reason behind the difference.

### UI

- *Streamlined form creation* — the quick-create bar now asks for the title only, Enter creates and Escape closes, and the full editor opens immediately. The form is kept in memory alone: it reaches the server only on the editor's Save, so abandoning the editor no longer leaves a title-only draft hanging in the list — previously the stub was persisted up front, while the built form itself was lost anyway unless saved, so the early write bought nothing.
- *Leaving the form editor with unsaved changes now asks first* — Cancel shows an inline confirmation when the form has been modified, and closes silently when it has not. This matters doubly for a freshly created form, where Cancel discards the whole draft rather than just edits.
- *Escape cancels a delete confirmation* — the inline "Delete?" prompts in the form editor and the admin list now close on Escape. Focus is intentionally not moved into any of these prompts, so a stray Enter can never confirm a deletion or a discard.
- *Attachment limits in the form* — the upload field shows the effective limits and rejects a file immediately after it is picked, before anything is uploaded.
- *Opening a form gives immediate feedback* — the catalog card that was clicked shows an inline spinner and the other cards ignore clicks while the form loads, where previously nothing reacted until the form appeared. A full-screen loader is kept for opens with no card to mark — a direct link to a link-only form or a form filtered out by the current search.

### Fixes

- *My Requests order is deterministic for every user* — the list is now sorted inside the app's backend handler. The server-side search directive used for ordering proved unreliable when combined with an extension-properties filter: for some users the results came back in insertion order while the code believed the server had sorted them, so with more than a page of requests the newest ones silently fell onto page two and the list looked unsorted. Search itself is still used — it runs as the requesting user and therefore respects issue visibility — but its ordering is no longer trusted.
- *Attachment errors reach the user* — a file that YouTrack refused to attach used to be dropped silently while the submission reported success; such failures now come back with the response, carrying the server's own message instead of a generic "Total size of attached files is too large", so the offending file and the limit it exceeded are named.
- *Malformed file types no longer lose files* — a client-supplied MIME type is validated before it goes into the data URI, instead of breaking the parser and discarding the file.
- *Form opens faster on large instances* — resolving a form's dynamic options no longer walks every option bundle of the project: only the fields the form actually reads, as sources or mapping targets, are resolved.
- *Navigating while a form is loading* — opening another form by a deep link or the browser Back button while one was still loading let the two requests race: whichever answered last won, and a straggler could reopen a form the user had already left and push its hash back into browser history. The latest action now wins — a superseded request cannot change what is shown, and its own failure cannot cover the loading form with a full-screen error.

## v2.4.3

- *Feature: user and group fields as dynamic source* — user-type (`user[1]`, `user[*]`) and user-group (`group[1]`, `group[*]`) YouTrack custom fields can now be used as a data source for select, multiselect, radio, and tags form fields on non-guest forms. Previously these field types were hidden from the source picker entirely.
- *Change: directory sources submit keys, not display strings* — dynamic options for user/group fields travel as `{key, label}`; the dropdown shows full names / group names while the form submits the login or group name directly. Logins no longer appear in issue summaries, descriptions, or the tracking page — full names are shown there instead.
- *Fix: multi-value directory sources* — `user[*]` and `group[*]` sources now add each selected value to the issue field instead of assigning a joined string; every value is resolved against the field's own value set on the server.
- *UI: privacy warning for wide-access forms* — when a form with `All YouTrack Users` or `Link only` access level uses a user-type field as its source, the field block is highlighted with a warning border and a banner explains that everyone who can open the form will see the full list of users in the dropdown.
- *Security: guest forms unchanged* — guest-accessible forms still block user/group source fields server-side and auto-unpublish on violation.
- *Fix: hidden values in dropdowns* — archived enum values and banned users are filtered out of dynamic options on both the server and the fallback path; rejections of out-of-bundle users return a single generalized message (no login-existence oracle), with the precise reason logged server-side.
- *Fix: guest forms no longer accept dynamic mappings to user/group fields* — such mappings are stripped on save and detected on open, closing a frontend-only gap.
- *UI: form list ordering and status badges* — within each category, published forms are listed first and drafts are moved to the end. The status column now shows a worded "Live"/"Draft" badge with an eye icon merged in, replacing the bare status icon.

## v2.3.10

- *Feature: File Attachments Support* — added a new `attachment` field type to the form builder. Users can now upload multiple files directly within Service Center forms.
- *UI: RingUI Dropzone Integration* — file uploads utilize the native JetBrains RingUI `<Upload />` component for a seamless and native visual experience.
- *Feature: Native Issue Attachments* — uploaded files are reliably attached to the newly created YouTrack issues using YouTrack's native Data URI parser via the workflow API, ensuring zero data loss or binary corruption regardless of sandbox restrictions.

## v2.2.8

- *Fix: Form editor stability* — fixed an internal issue in the form builder that could cause unexpected behavior when using step separators.
- *Security: Admin Widget Permissions* — Project Settings widget now strictly requests `UPDATE_PROJECT` permission in its manifest. 
- *Fix: Page scroll jumps on dropdown/date open* — opening a select, multiselect, or datepicker field on the form no longer scrolls the viewport down to the Submit button.
- *Fix: Manual option selection in multiselect* — selecting a single option no longer triggers selection of all other manual options at once.
- *Fix: Archived custom field values* — values marked as archived in YouTrack custom fields are now hidden from form dropdowns.
- *UI: Editor panels collapsed by default* — all settings sections (Fields, Field Mapping, and Static Values) now start collapsed to keep the editor clean.
- *UI: Hint input styling* — hint settings and their labels in the editor are styled in italics and slightly dimmed for better visual separation.
- *UI: Step separator alignment* — fixed unequal top and bottom padding inside step-break rows.
- *Perf: Lag-free collapsible panels* — disabled height transition animations on collapsible headers to make expanding/collapsing instant.
- *Fix: Detailed YouTrack workflow errors* — when a workflow blocks issue creation, the error message now includes the specific rule name (e.g., "Workflow runtime error (rule: wf-rule-xxx)") instead of a generic message.
- *UI: Drag-and-drop field reordering* — added support for drag-and-drop sorting of fields and step separators using a tactile grab handle, accompanied by a horizontal glowing blue drop line indicator.
- *UI: Form editor alignment & styling* — aligned all secondary settings rows (hint, alignment, text content, options) with the main input start, styled format checkboxes in their respective styles (bold, italic, underline), and balanced top/bottom margins for display-only cards.
- *UI: Manual options tree structure* — manual options and the "+ option" button are organized into a nested tree diagram using dashed branch lines.
- *UI: Form editor layout improvements* — labels and placeholders now share space equally (50/50), and decorative display-only fields (headings/paragraphs) stretch to full width.
- *My Requests: deep-link & URL sync* — navigating to "My Requests" updates the URL hash (`#MY_REQUESTS`); loading a page with this hash automatically opens the requests list.
- *Refactoring: flat manual & dynamic options* — manual and dynamic option lists are now stored and transmitted as flat string arrays. Existing forms are migrated transparently on read; no re-save required.
- *UI: Block deletion overlay confirmation* — clicking the delete button on form fields or step breaks now displays a semi-transparent glassmorphism overlay directly on top of the block with confirmation buttons. Initiating any drag-and-drop action automatically dismisses the confirmation window.

## v2.1.1

Major release — the most significant update to date. Each project now stores its own forms independently, eliminating data limits and conflicts when multiple projects use Service Center at the same time. Security was overhauled: guest-accessible forms are strictly validated and user data is protected from leaking to unauthorized users. New in this release: My Requests for logged-in users, multi-value field support in static mappings, initial comments on issues, a full set of UI improvements in the form editor and portal, and numerous bug fixes.

### Architecture & Performance

- *Faster portal load* — Catalog assembly was redesigned end-to-end. The server now returns a flat list; the portal groups it locally in a single pass. For large portals with many forms this is noticeably faster.
- *Fewer API calls on startup* — Removed a redundant project-lookup step that ran on every portal open. Access checks are now done entirely on the server before the response is sent.

### New features

- *My Requests* — authenticated users can view all issues they submitted through Service Center forms. Accessible via the "⋯ → My Requests" menu in the portal. Shows summary (clickable link to the issue), form name, and creation date. Sortable by any column; newest first by default.
- *Multi-value Static Values* — static mappings now support multi-value fields (e.g. multi-select, multi-version)
- *Single-value field as multi source* — single-value select fields can now be used as the option source for multiselect and tags fields
- *Initial Comment* — optional comment added to the issue after creation; configured per form

### UI / UX & Performance

- *Fixed reporter save bug* — fixed a bug where quickly pasting a reporter login and immediately clicking Save would cancel the save due to a timing issue. Save now correctly waits for the login to be validated before proceeding.
- *Field mapping editor fix* — fixed an issue where the field mapping editor could get stuck in an infinite update loop under certain conditions.
- *Form name in My Requests* — the Form column shows the human-readable form title; displays "(unavailable)" if the form has been removed or unpublished
- *Dates use YouTrack user locale* — creation dates in My Requests are formatted according to the user's YouTrack locale and timezone
- *Loading spinner* — My Requests shows a spinner while fetching data
- *Empty state in portal* — catalog shows an illustration and message when no forms are published or no search results are found
- *Support Chat link* — internal builds show a support link in the admin banner
- *Search reset on guest preview* — switching to guest view clears the search field
- *Template placeholder hints in field editor* — each field now shows the exact placeholder syntax you can use in summary/description templates, e.g. `{Field Label}` or `{field_id}`; the internal field ID is also shown as a badge
- *Hint text on fields* — optional hint between label and input, supports multiline
- *Paragraph formatting* — bold, italic, underline; multiline with Enter
- *User picker in Static Mappings* — user-type fields use a searchable picker with avatars
- *Live placeholder list* — summary, description, and comment template inputs show a live list of all available placeholders while you type
- *Static mapping validation* — saving with incomplete rows is blocked; incomplete rows are highlighted

### Storage, Reliability & Security

- *Per-project storage* — each project stores its own forms independently; no more shared storage limits or write conflicts between projects
- *Isolated form data* — form data is no longer packed into a single large blob; each record is stored separately, improving reliability at scale
- *Automatic data migration* — existing forms are migrated automatically on first use; no manual steps required
- *Group fields protected* — YouTrack group fields are now treated as sensitive data, same as user fields. They cannot be used as option sources in guest-accessible forms.
- *Live form safety check* — when a guest opens a form, the backend re-validates it against the current project schema. If an admin changed a field to a sensitive type after the form was published, the form is blocked immediately — no stale cached state can leak data.
- *Category delete safety* — deleting a category now automatically removes the category reference from all affected forms in the same operation, preventing forms from pointing to a non-existent category.
- *Orphaned form handling* — if a form loses its category due to an unexpected data issue, the portal now shows it under "Uncategorized" instead of hiding it silently.
- Guest-accessible forms are re-validated on every save; if a form contains user-type fields as option sources, it is automatically unpublished for guests
- User and group fields are blocked as dynamic option sources for all access levels, not just guest forms
- Several security issues fixed; no action required from administrators

### Bug fixes

- Invalid date input now shows an error instead of silently using current date
- Stale field mappings cleared on source field change
- Type switch clears inapplicable field properties
- Select/multiselect options load correctly with mapped enum fields
- User fields applied correctly on submit

## v1.3.2

- **Admin endpoints → project scope** — `admin/*` handlers moved from global to project scope with `UPDATE_PROJECT` permission; Project Admins can manage forms without system-level access
- **Strict project isolation** — form endpoints enforce `projectId` checks; admins can only view, edit, and delete forms belonging to their own project
- **Uninitialized project guard** — if Service Center is not enabled for a project, form list returns empty and write operations return 403
- **projectId enforcement on save** — `projectId` is always set to the current project, preventing client-side reassignment
- **Fix: unpublish order on disable** — forms are unpublished before removing the project from enabled list, fixing a potential race condition
- **Cleanup** — removed custom role checks, replaced by native `UPDATE_PROJECT` permission
- **UI: success page buttons** — swapped button order; "Back to Portal" (default) on the left, "Track status" / "View issue" (primary) on the right
- **Docs** — expanded HOW_TO_USE with wizard, dynamic options, reporter mode, form width, guest preview, guest field filter, conditional static mappings, catalog view modes; updated README features list

## v1.2.47

- **Security: project team membership check** — `project_members` forms now enforce access via project.team.users; non-team-members are denied even if the project is publicly readable
- **Refactor: unified Form Access check** — single access control method handles all four access levels (`guests`, `link_only`, `youtrack_users`, `project_members`) for public endpoints
- **Strict access level validation** — forms with unknown or missing `accessLevel` are denied with 403 instead of silently defaulting
- **Catalog cleanup** — empty categories (no visible forms after access filtering) are no longer returned in the `/catalog` response

## v1.2.31

- **UI: Access level tags** — access level badges in the admin forms list replaced with Ring UI `Tag` components with distinct colors per level (`project` → purple, `yt-users` → warning/yellow, `public` → error/red, `link-only` → default/grey)
- **UI: Status icon** — publish/draft status icon moved to the status column, shown left of the access level tag
- **UI: Admin forms table** — refactored from multiple per-group tables to a single table with group header rows; column widths are now consistent across all groups; status column has a fixed minimum width; actions column shrinks to content
- **UI: Uncategorized group** — always shown first in the admin forms list with a subtle warning background to draw attention
- **UI: Group dividers** — replaced `<hr>` between system and user groups with CSS `border-top` on the first user group header row
- **Bug fix: Toggle disabled on error** — `handleToggleEnabled` now uses `try/finally` so the Service Center toggle is always re-enabled even if unpublishing forms throws
- **Bug fix: Delete form error** — `handleDeleteForm` now shows an error banner if the backend returns an error (previously silently failed)
- **CSS: Duplicate rule removed** — removed duplicate `.sfc-nowrap` declaration
- **Security: Stable tracking token seed** — tracking links now use a persistent installation identifier generated on first use; improves token stability and security across requests
- **Bug fix: required multiselect/tags validation** — submitting a required multiselect or tags field with no values selected now correctly returns a validation error
- **Bug fix: form submission** — fixed internal variable handling in the submit handler that could cause errors in edge cases
- **Code quality** — internal refactoring of access control checks for form create and delete operations; no behavior change
- **UI fix: catalog chevron icons** — expand/collapse icons in grouped catalog view now render correctly
- **Docs: Link Only access level** — updated documentation with a clearer description of how the Link Only access level works
- **ACL group guard** — new global settings `aclForAllUsers` / `aclForGuests` (YouTrack group pickers in App Settings); if a group is not configured, the corresponding access level option is disabled in the form editor; if configured, only members of the group can select it
- **Server-side ACL validation** — reject unauthorized access level assignments with 403 and a descriptive error message; displayed as an error banner in the form editor
- **Access level policy endpoint** — per-level allow/member flags for the current user; used by the form editor to disable options without a round-trip on save
- **Form authorship tracking** — `createdBy`, `createdAt`, `updatedBy`, `updatedAt` fields are now saved with every form; `updatedBy`/`updatedAt` also stored in the forms index
- **Updated-by subtext** — forms table in Project Settings → Portal shows `<login> · <date>`, formatted according to the user's YouTrack locale and timezone

## v1.1.21

- **Link Only access level** — new Access option "Link only": form is hidden from catalog but accessible and submittable by any authenticated YT user via direct link
- **All Users access level** — new Access option "All YouTrack users": form appears in catalog and is accessible to any authenticated YT user (no project membership required)
- **Access level badges** — project widget form list now shows a badge next to the copy link button indicating the form's visibility (`link only`, `all users`, `public`)
- **Security fix: catalog disclosure** — `youtrack_users` and `project_members` forms are now filtered server-side in `/catalog`; guests no longer receive metadata for non-guest forms in API response
- **Security fix: catalog unified filter** — single unified access filter covering all four access levels in one pass
- **Security fix: submit bypass** — `/submit` now enforces project membership check for `project_members` forms; non-members receive 403
- **Security fix: submit project check** — `/submit` now verifies the target project is enabled in Service Center before creating an issue; prevents submission to disabled projects

## v1.1.17

- **Compact Tracking Layout** — status, priority, and updated date displayed in a single row on the tracking page; updated/resolved shown in secondary style on the right
- **Template Label Resolution** — Template resolver now supports field labels as placeholders (`{Summary}`, `{System / Service}`, etc.) in addition to field IDs
- **Field Label Uniqueness** — duplicate field labels are detected in the form editor with Ring UI ErrorBubble notification and red border; empty labels also flagged for non-display fields
- **Field Label Validation** — labels containing reserved characters (`{}`, `<>`, `` ` ``, `\`, `|`) show an error; all label errors displayed via Ring UI ErrorBubble
- **Demo Data Prefill** — "Prefill with demo data" button in admin UI (empty state) creates sample categories (IT Support, HR Requests) and forms (Bug Report, Access Request, Leave Request, Multi-Step Demo) for quick onboarding; reuses existing categories by title match; Multi-Step Demo showcases all 13 field types in a 2-step wizard
- **Orphan Forms Visibility** — forms with missing/deleted category now displayed in "Category not found" group instead of silently disappearing from admin UI
- **Submit Success UX** — after form submission, "Back to Portal" button navigates to catalog; browser back no longer shows blank page
- **Required Field Validation** — all field types (select, radio, checkbox, date, tags) now show ErrorBubble on submit when required but empty
- **Form Validation Styling** — inline error messages styled with Ring UI error color
- **Island Form Layout** — form view uses native Ring UI card styling with header border
- **Ring UI Panel** — form navigation buttons use native Ring UI panel; Back aligned left, Submit/Next aligned right
- **Catalog View Modes** — removed compact grid view; updated view mode icons
- **Tracking Fix** — fixed error when loading tracking page
- **Dead Code Cleanup** — removed some unused variables from track handler single-pass optimization
- **ValuesMap Collision Guard** — label-based keys in valuesMap are only added if not already occupied, preventing overwrite from duplicate labels
- **Tags Deduplication** — selected tags are now hidden from the dropdown; duplicate selection is prevented
- **Empty Fields in Description** — auto-generated and template descriptions now include all fields; empty values shown as "—"
- **State Reset Consolidation** — unified portal state reset into a single function, fixing stale state on browser back navigation
- **Summary Fallback** — if summary template resolves to empty string, falls back to form title
- **Boolean Fields Display** — checkbox values shown as "Yes"/"No" in descriptions instead of raw "true"/"false"
- **Field Mapping Type Safety** — Simple custom fields (string, integer, date, etc.) now filtered by subtype; incompatible mappings (e.g. string → date) no longer offered in editor
- **Required Label** — form editor checkbox label changed from abbreviated "req" to full "required"
- **Guest Mode: Enforced Fixed Reporter** — selecting Access=Guests automatically switches Reporter to Fixed login and disables the toggle; existing guest forms also enforce this on edit
- **Guest Banned Detection** — form editor checks if Guest user is globally banned in YouTrack; if so, the Guests option is disabled with a label and an error banner is shown. Check README for detailed Guest Mode setup guide, behavior description, and troubleshooting

## v1.0.117

- **Issue Description on Tracking** — tracking page now displays the issue description with lightweight markdown rendering (bold, horizontal rules, newlines)
- **Dynamic State Detection** — state field identified dynamically via `isResolved` property instead of hardcoded field name; works across all locales and custom field names
- **Dynamic Priority Detection** — priority field identified dynamically via `ordinal` property instead of hardcoded "Priority" field name
- **Description Generation Fix** — fields mapped to `__description` pseudo-target are no longer incorrectly excluded from auto-generated description
- **Tracking Resilience** — `loadTracking` wrapped in try/finally to prevent infinite loading state on network errors
- **Salt Consistency** — track handler now passes `installationId` to `getOrCreateTrackingSalt`, matching submit handler behavior
- **Form Migration** — added `formWidth` to `FORM_DEFAULTS` for consistent migration of older stored forms
- **Security: Access Control** — `/forms/get` and `/submit` endpoints now enforce `accessLevel` check on the backend; guests are denied access to `project_members` and `youtrack_users` forms even if they know the slug
- **Security: CSPRNG Salt** — tracking salt generation upgraded to `java.util.UUID.randomUUID()` (backed by `SecureRandom`) with `Math.random()` fallback for restricted sandboxes

## v1.0.59

- **Guest Tracking Page** — guests can track issue status via stateless deep-link token (`TR_<token>` in URL hash); tracking page shows summary, status, priority, created/updated/resolved dates; token generated with HMAC-SHA256 + installation salt
- **Submitted Fields on Tracking** — tracking page displays all visible form fields filled by the guest (label + value table); data stored in `issue.extensionProperties.sfcenter`; guest email excluded from display
- **HMAC-SHA256 tracking tokens** — replaced djb2 (31-bit) with HMAC-SHA256 (256-bit) for cryptographically secure tracking links; pure JS implementation ported from adv.attachments; 32-char random salt
- **DRY: parseStore optimization** — `getOrCreateTrackingSalt` now accepts pre-parsed store to avoid redundant JSON parse/stringify cycles during submit
- **Track status button** — success page shows "Track status" button for guests (deep-link navigation) and "View issue" link for members
- **Security fixes** — removed `description` and `projectName` from track response (information leak prevention); guest email excluded from `submittedFields`

## v1.0.39

- **Guest view (preview)** — toggle in dropdown menu on the portal to preview catalog and forms as a guest user; Ring UI warning Banner with icon
- **Guest email field** — automatic required email field for guest users on guest-accessible forms; appended to issue description on submit
- **Internal Project ID** — project internal ID (`internalProjectId`) is resolved and saved when editing forms in the constructor; portal uses it directly without calling project API (fixes guest 404 errors)
- **Navigation sync** — polling fallback for browser back/forward via `getAppLocation()` with `useRef` to prevent repeated form opens; `lastHashRef` synced on openForm/closeForm/deep-link
- **Form state protection** — `handleLocationChange` skips re-opening if the same form is already active, preventing form values/errors reset during polling
- **Ring UI search input** — portal search replaced with Ring UI `<Input>` with search icon and clear button
- **Auto-growing textarea** — Ring UI `<Input multiline>` without fixed rows for auto-expanding behavior
- **Breadcrumbs cleanup** — removed redundant "Portal Index" link, leaving only back arrow + active form title
- **Project ID resolution** — `resolveProjectId` uses admin endpoint with fallback to public search; skips API calls entirely when `internalProjectId` is available
- **Backend public form DTO** — `internalProjectId` included in public form response so portal can create issues without project API access

## v1.0.2

- **Wizard Step Editor** — new `step-break` field type to split wizard forms into steps; dedicated "Add Step Break" button in field list; step-break renders as a distinct darker block with badge
- **Wizard ↔ Single layout** — switching to single automatically removes all step-breaks; switching to wizard shows an info banner if no step-breaks exist
- **Steps conversion** — on save, flat field list with step-breaks converts to `steps[]` array; on load, `steps[]` converts back to flat list with step-break separators
- **Arrow stepper on portal** — step progress indicator with numbered badges, arrow separators, and 3 visual states (done/active/pending)
- **Portal navigation buttons** — Back button aligned left, Next/Submit aligned right
- **Form Editor layout** — Category, Layout, Width controls moved above Title for better workflow
- **Success page width** — submission confirmation now respects the form's width setting (compact/normal/wide)

## v0.9.235

- **Service Catalog** — public portal page with categories, form cards, search, and direct deep-link URLs
- **Form Engine** — schema-driven forms with 14 field types (string, textarea, select, multiselect, radio, checkbox, date, number, email, tags, hidden, heading, paragraph, banner)
- **Multi-step Wizard** — forms support single-page and wizard (multi-step) layouts with step progress indicator
- **Field Mapping** — map form fields to YouTrack custom fields (enum, date, text, etc.) with automatic type compatibility checks
- **Static Mapping** — set fixed values on created issues (e.g., Type=Bug, Priority=Critical)
- **Dynamic Options** — select/multiselect/radio fields can source options from YouTrack enum custom fields at runtime
- **Summary & Description Templates** — configurable templates with `{field_name}` placeholders for issue creation
- **Category Management** — create, edit, delete categories with duplicate-title protection and in-use checks
- **Form Editor** — inline editor with field reordering, type-specific settings, and live slug/deep-link preview
- **Project Settings Widget** — enable/disable service center per project, manage forms, publish/unpublish
- **Deep Linking** — direct URLs to specific forms via `host.navigation` API (YouTrack 2026.1.1+)
- **Form Width** — compact, normal, and wide layout options for forms

## v0.1.0

- **Initial scaffold** — project structure, build scripts, docs, and licenses adapted for Adv.ServiceCenter
