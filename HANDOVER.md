# Handover — Recruitment ATS training sandbox

Written to carry this project into a new conversation. Read this first, then the README, then
`app.js`.

---

## 1. What this is

A Bullhorn-shaped ATS/CRM training simulator for recruiters. Single-player, browser-only, no
install, no server, no dependencies. Hosted on GitHub Pages.

Built for an AVP running Learning & Development in a corporate recruitment environment. Trainees
will complete classroom call training separately, then use this to practise the system workflow.

**Deliberately generic.** No vendor name, logo or brand asset. Terminology and structure follow
Bullhorn; the branding does not.

---

## 2. Files

Three files, same folder, no build step:

| File | Size | Contents |
|---|---|---|
| `index.html` | ~3 KB | Shell markup, a visible fallback if `app.js` fails to load, and **no third-party requests** |
| `app.css` | ~47 KB | Base styles, then skin / spacing / navigation / chart override passes |
| `app.js` | ~401 KB | Everything else, inside one IIFE |

GitHub Pages: put all three in the repo root, Settings → Pages → Deploy from a branch → root.
Serving over `https://` matters — some browsers block storage on `file://`.

**No external requests.** Google Fonts was removed; the font stacks name Archivo and IBM Plex Mono
first, so dropping `.woff2` files into a `fonts/` folder picks them up, and otherwise the operating
system supplies a close grotesque and monospace. This matters on a filtered corporate network, and
it matters to the argument in §13 that nothing leaves the machine.

**`app.css` is base rules plus four appended override passes**, so `:root` tokens are redefined
mid-file. It works but is annoying to edit. Consolidating it is an outstanding ~20 minute job.

---

## 3. Architecture

- One IIFE. No modules, no framework, no libraries. Rendering is string templates into `#main`.
- `DB` is the single in-memory state object. `render()` redraws the current view wholesale.
- Routing is `route = {view, id, tab}`; `VIEWS` maps view keys to `v*()` functions.
- Events are delegated from `document` on `data-act` (actions) and `data-go` (navigation).
- Persistence is IndexedDB (`recruitment_ats_sandbox`), two stores: `state` and `files`.
- **Resume text is stored per candidate in the `files` store, not inside the state record.** With
  3,000 CVs the state blob was 9.6 MB and every autosave stringified it. Only changed resumes are
  rewritten.
- Autosave is debounced ~900 ms and fires from `render()`; UI preferences save immediately.

### Shared render helpers (this is the closest thing to a component library)

`recHead` green record header · `rtabs` record tab strip (config-driven) · `chevBar` workflow
chevrons · `detailRows` label/value table · `withDetails` content + right-hand Details card ·
`listBar` list header · `sortTh` sortable column · `selBox` row selection · `selBar` bulk toolbar ·
`panelIcons` card chrome · `noteList` / `activityList` timelines · `taskTable` · `subPill` /
`cdPill` / `joPill` / `plPill` status badges · `covBar` · `met` metric tile · `funnelBar` ·
`svgPie` / `svgLine` charts · `openForm` validated form modal · `openInfo` · `toast` · `crumb` ·
`A.actions` contextual action menu · `filesPanel`.

Data helpers are separate from rendering: `jobSubs`, `candSubs`, `funnel`, `timeToFill`,
`staleSubs`, `onboardGaps`, `sendouts`, `marginBand`, `markup`.

### Generated data

3,012 candidates, all with full CV text, produced at boot by `generatePool()` from a **fixed seed**
(`rng(20260907)`). Every trainee gets a byte-identical dataset with zero storage cost. Domains and
counts live in `VERTICALS`: IT 620, Light Industrial 545, Healthcare 504, Admin & Clerical 401,
Engineering 380, Finance & Accounting 220, Customer Support 180, Skilled Trades 100, Retail
Operations 62. Name pools are 622 first × 687 last names.

The 12 hand-written candidates carry the seeded desk activity (submissions, placements,
timesheets). The 3,000 are an unattached sourcing pool.

---

## 4. Data model

`Lead → Opportunity → Company → Contact → Job Order → Candidate → Submission → Appointment →
Placement → Time Entry`, plus Notes, Tasks, Tearsheets, Saved Searches, Files, Audit,
Notifications.

The submission carries the candidate-to-job relationship. A candidate has a global status *and*
a per-submission status, which matches Bullhorn.

---

## 5. Configuration layer (important)

Bullhorn holds picklists, required fields and tabs **per tenant**, not fixed. So they live in
`DB.config` as data, editable at **Menu → Configuration**, and `syncConfig()` mutates the live
constant arrays in place so existing references stay valid.

Configurable: all status picklists and their order, note actions, candidate sources, job and
employment types, categories, required fields per entity, candidate and job tab order, navigation
order, report card set.

Guards: a picklist needs ≥2 values and no duplicates; the submission pipeline is fixed at 6 stages
because the board is built around them; anchor fields (candidate name, job title and company,
contact company) cannot be made optional.

Renaming a picklist value deliberately does **not** rewrite existing records — that mirrors a real
reconfiguration and is a useful exercise.

---

## 6. Rules

Hybrid: strict about integrity, flexible about business decisions. Errors matching `/HARD STOP/`
or a denylist of patterns always block; everything else warns and can be overridden in permissive
mode, with the override logged.

**Hard:** no CV on a client submission; candidate Do Not Call or Archive; job Closed or Cancelled;
bill ≤ pay on a contract role; no salary on a direct hire; openings below placements made; Filled
while coverage short; retrospective pay-rate change on approved weeks; >24 overtime hours;
hand-converting a lead or opportunity; end date before start.

**Soft:** missing screening notes or client summary; pay below confirmed expectation; bill above
the job order rate; duplicate candidate; unapproved overtime; missing rate, phone, email, source.

**Margin bands on placement approval** — contract: ≥20% normal, 10–20% needs a manager, <10%
refused. Permanent: judged on fee as % of salary (≥15% / 10–15% / <10%).

**Approval separation:** the creator of a placement cannot approve it.

**Compensation split** — Contract: pay rate, bill rate, mark-up % = (bill − pay) / pay.
Direct Hire: salary, flat fee, no timesheets. Employment type is filtered by job type.

**Resumes:** attaching a file changes nothing. Files → the resume → **Parse as Existing** previews
current vs proposed field by field with per-field checkboxes. `parserOverwritePrevention` protects
populated fields and appends to list fields; default off.

---

## 7. Modes (all persisted)

| Mode | Default | Effect |
|---|---|---|
| Interface | Redesigned | Redesigned = permanent left sidebar with all navigation, Fast Find and Preferences in it, no record tabs in the sidebar. Legacy = Menu flyout + pinned sections + open records in the left column |
| Rules | Permissive | Strict blocks on soft rules; permissive warns with an override |
| Training | On | Off hides the practice panel and strips coaching prose from every form |
| Desk type | 360° | Scopes where scenarios end — see below |

**Desk types** (`DESKS`, each step carries `r: 1|2|3`):

- **180°** — delivery and sourcing, finishes at Client Submission. Scenario 1 = 5 steps.
- **360°** — full desk, finishes at the candidate starting. Scenario 1 = 16 steps.
- **VMS** — full lifecycle, finishes at approved hours. Scenario 1 = 18 steps.

Scoping applies to every scenario, and progress counts against the scoped list.

---

## 8. Built

Navigation and shell, Fast Find with grouped results and Ctrl+K, Add New menu, record pinning via
tabs (legacy), notification centre.

Lists with filters, sorting, paging, multi-select, select-all, bulk toolbar (Mass Update, Add to
Job, Add to Tearsheet, Add Note), quick-view slide-out preview.

Records with green header, Actions dropdowns, tab strips, clickable workflow chevrons, Details card
on the right, Overview / Activity / Notes / Submissions / Placements / Files / Resume / Edit.

Boolean search with AND / OR / NOT / `-` / brackets / quoted phrases / `*` wildcards / field
scoping, query interpretation displayed, matched-terms per row, saved searches, mass actions from
results, find-candidates-from-job-order.

Job order pipeline board with drag-and-drop (gates enforced on drop), submission funnel with
counts, submission drawer with status history.

Email composer with 7 templates, CV attach, logged as a linked activity. Appointments. Tasks with
assignment. Placement approval, onboarding checklist, time entry with overtime rules.

Reports: 6 chart cards (inline SVG, add/remove persisted), conversion by job order, source
performance, data-quality flags, weekly desk review. Dashboard: metrics, needs attention, funnel,
practice progress, tasks, notes.

Training: 4 scenarios with automatic step verification, optional first-run guided tour (11 steps,
skippable, replayable), 12-question knowledge check (pass 8), session summary for the trainer.

Database screen: export, import, save, clear, record counts, storage status.

---

## 9. Not built

- First/Middle/Last name split; `phone2`/`phone3`, `email2`/`email3`
- Draggable Overview cards with per-user persistence (the handle was **removed** rather than faked)
- PlacementChangeRequest, and permission-based approval (currently creator-cannot-approve)
- Edit as a form tab rather than a modal
- Column chooser with saved list views
- Calendar view
- Work History / Education / References / Credentials tabs
- Duplicate-review screen on creation
- Custom fields (`customText1` etc.) and a metadata-driven field layer
- Roughly half the 57 screens in the client's §89 inventory

---

## 10. Testing

**The suite is in `tests/` and must be committed to the repo.** It was not, in an earlier handover,
and that made every subsequent change unverifiable. See `tests/TESTING.md`.

20 suites plus 2 audits, all passing. Node + jsdom + fake-indexeddb, no browser needed.

```
cd tests && npm install && npm test     # everything, one command
node test19.js                          # one suite
```

`harness.js` discovers `index.html`, `app.js` and `app.css` in `.`, `..`, `../app` or `../src`, so
it runs from a `tests/` folder beside the app or from the app folder itself. It also accepts
`dist_*` names for a working copy.

| Suite | Covers |
|---|---|
| test2 | Full desk cycle end to end (runs in **legacy** interface and **strict** mode) |
| test3 | CV pool, boolean semantics, saved searches, mass actions, edit guards |
| test4 | Scenario 4 sourcing completion |
| test5 | Persistence across reloads, clear, migration |
| test6 | Guided tour |
| test7 | Collapsible practice panel, job-order sourcing |
| test8 | Email, notifications, training mode |
| test9 | Navigation, **both** interfaces |
| test10 | Permissive mode, bulk, duplicates, drag-and-drop, preview |
| test11 | Agreed picklists, rule bands, layout |
| test12 | Files, Parse as Existing, Direct Hire pricing |
| test13 | Tenant configuration, workflow icons |
| test14 | Details card, funnel, Actions menus |
| test15 | Report chart cards against real data totals |
| test16 | **State-transition matrix** and **boundary values** |
| test17 | **Injection / escaping** (7 payloads), **fuzzing** (43 queries), extreme values |
| test18 | **400-step randomised walk** + invariants, export/import round trip |
| test19 | **Business flows** — Direct Hire end to end, decline paths, cross-mode |
| test20 | **Accessibility sweep** — roles, names, labels, keyboard, every screen and dialog |
| test21 | Desk types scoping scenarios, progress and the session summary |

### Testing lessons that cost real bugs

1. **Feature coverage is not business-flow coverage.** Fifteen suites passed while a **direct hire
   could never be client-submitted** — the sendout applied the contract "bill > pay" rule to
   permanent roles with no hourly rates. 30% of the intended mix was dead. Found only by writing a
   perm flow end to end (test19).
2. **Fresh-install testing hides upgrade bugs.** Moving resume text to its own store lost every CV
   for anyone with previously saved data — restored, displayed once, then stripped by the first
   save. Found by the client's screenshot, not by tests. There is now a migration path plus a
   stale-dataset notice.
3. **There is no browser or screenshot tool in these sessions.** jsdom has no layout engine, so
   duplicated headings, clipped elements, overlapping text and repeated group headers all pass
   automated checks. **Client screenshots found four visual bugs that 20 suites did not.** Ask for
   screenshots.
4. **Patch scripts that abort mid-way leave a broken state.** Several `python` patch runs asserted
   and exited before writing, once leaving quick-view controls rendering with no handler. The
   dead-control audit caught it. Always report applied/failed per item and re-run the audits.

---

## 11. Decisions and rationale

- **Generic branding, Bullhorn structure.** Client asked for close fidelity without trademarks.
- **Picklists and required fields as data, not code.** Bullhorn documentation is explicit these are
  per-tenant. Hard-coding them would teach a configuration that isn't the client's.
- **Permissive as the default.** Real systems mostly let you save the wrong thing and surface it in
  a report. Guardrails that don't exist in production teach a dependence on a safety net.
- **Two interface modes.** The client's screenshots were the legacy top-bar UI; documentation says
  the redesign is a phased mandatory migration. The switch is labelled in-app as a sandbox feature,
  not a Bullhorn one.
- **No affordance without behaviour.** A card drag handle and a Layout button that did nothing were
  removed / made real. Worth holding to.
- **Charts hand-rolled in SVG.** Keeps the zero-dependency, no-build constraint.

---

## 12. Open questions

**Blocking accurate configuration:**

1. The client's actual status picklists, read off their own screens. Everything currently in the
   app is a reasonable Bullhorn-shaped configuration, **not their tenant's**.
2. Which fields their team marks required, at which stage.
3. Their approval routing and margin threshold (10% / 20% bands are invented).
4. A screenshot of a **job order record** — that layout was inferred entirely.

**Blocking the next phase:** see §13.

---

## 13. Next phase — multi-user trainer/trainee portal

The client's proposal, and it is a good one:

- 5–6 **trainer** logins. Trainer creates or uploads a job description.
- Trainees log in with an employee ID (no password, accepted by the client) and see jobs assigned
  to them or in a pool.
- Trainee sources a candidate internally or externally, records it, adds notes, makes an **internal
  submission**.
- Trainer acts as **Account Manager**: approves, rejects, or raises queries for more detail.
- Trainee answers the query, contacts the candidate, adds notes, and so on.
- After training, the trainer downloads a report of trainee activity and archives the batch.
  Multiple batches run concurrently.

In Bullhorn terms this is the **Internal Submission → Client Submission** gate with a human on the
other side, and the queries are **Notes against the submission**. It teaches the thing automated
validation cannot: submission *quality*.

### Advice already given to the client

- **This breaks "no installation".** Shared state needs a server. Trainees still just open a URL;
  someone has to stand up a backend once. It does, however, **solve their IT objection** to browser
  storage, which is a better answer than the memory-only mode also proposed.
- **Trainers need real passwords** (M365 sign-in is free and gives identity in the audit trail).
  Employee-ID-only for trainees is acceptable for fabricated data, but add a per-batch join code.
- **Archive, don't wipe** — keeps batch-over-batch comparison.
- **Make the query a first-class state** (`Awaiting trainee response`) so a trainer queue shows
  what is blocked on whom.
- **Do not sync the 3,000-CV pool.** It is seed-generated and identical everywhere. Only sync the
  *work*: batches, users, job descriptions, submissions, notes/queries, sourced candidates, activity
  log. Hundreds of small rows per batch, not megabytes.
- **Backend:** SharePoint Lists via **Microsoft Graph** (not SharePoint REST — CORS) is the right
  call for a Microsoft shop; needs an Azure AD app registration with admin consent, PKCE only, no
  client secret in the file. Fallback: Supabase free tier (new vendor; free projects pause after
  inactivity).
- **Build order:** (1) data model and trainer/trainee screens against local storage — fully
  testable; (2) a storage adapter with a clean seam; (3) the Graph implementation, tested in the
  client's tenant. The existing single-player sandbox keeps working throughout as a second mode.

### Questions the client still needs to answer

1. Do trainees have M365 accounts? If yes, drop employee-ID auth entirely.
2. Can they get an Azure AD app registration with admin consent? This decides SharePoint vs Supabase.
3. Batch size and concurrency — 10 trainees or 50, one batch or four?
4. Do trainers approve **live** while a trainee waits, or work a **queue**? A queue is much simpler
   (no polling).

### Also outstanding: the IT objection

Their IT team said browser cache storage is dangerous. The reply given: all data here is
fabricated, so sensitivity does not apply, and moving to a database *enlarges* the attack surface
(auth tokens, network path, real employee identities). The question put back to them was whether
the concern is (a) data sensitivity, (b) browser storage being insecure, or (c) a policy that no
data may persist on the endpoint. **If (c), a memory-only mode was offered** — roughly an hour of
work, nothing written to the endpoint, results out via the session summary. Awaiting their answer.

---

## 14. Practical notes for whoever picks this up

- Work on `dist_index.html`, `dist_app.css`, `dist_app.js`, then copy to outputs. `harness.js` and
  the suites read the `dist_*` files.
- After **any** change: run all 20 suites and both audits. They are fast and they have caught real
  defects repeatedly.
- Ask the client for a screenshot after any visual change. It is the only layout verification
  available.
- If a client reports a blank page: `index.html` now shows a diagnostic placeholder if `app.js`
  fails to load, and a visible error with a stack if it throws. Ask which they see.
- Tell the client to **Reset all data** after any change to the seed or generator, or they will keep
  loading the previous dataset from browser storage.
