# Recruitment ATS — training sandbox

A self-contained practice environment for training recruiters on a staffing ATS/CRM workflow.
One HTML file, no build step, no server, no dependencies to install.

All records are invented. Nothing connects to a live system.

---

## Hosting it on GitHub Pages

1. Create a repository and add `index.html` to the root (this file and the sandbox are the only two files needed).
2. In the repository, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to *Deploy from a branch*, pick your branch and the `/ (root)` folder, then **Save**.
4. Wait for the green tick, then share the URL: `https://<your-user>.github.io/<repo>/`

Notes:

- No `.nojekyll` file is needed — there are no underscore-prefixed folders.
- Serving over `https://` is **better than opening the file locally**, because some browsers restrict browser storage on `file://` URLs. On Pages the local database always works.
- A private repository needs GitHub Pages on a paid plan. For a public repo it is free. Either way the sandbox contains no real data.
- Updating is a commit. Trainees get the new version on their next refresh; their saved work is unaffected.

## Running it without hosting

Download `index.html` and open it in Chrome, Edge or Firefox. Everything works. If the storage
indicator on the **Database** screen says *memory only*, the browser has blocked storage on the
`file://` URL — host it on Pages, or use **Database → Export** to keep your work.

---

## Where the data lives

Each trainee's work is stored in **IndexedDB in their own browser**, under the key
`recruitment_ats_sandbox`. It saves about a second after every change and survives refreshes,
closing the tab and restarting the machine.

- It is **per browser and per machine**. Nothing is shared and nothing is uploaded.
- **Database → Export** produces a JSON snapshot — this is how a trainee hands work to a trainer.
- **Database → Import** replaces everything with a snapshot — this is how you put a whole cohort
  on identical starting data.
- **Reset all data** returns to the seeded starting position.

Because storage is per browser, hosting on Pages does not create a shared database. If you need
several trainees working on the same records, that requires a hosted backend (Supabase and Neon
both have free tiers) — not included here.

---

## What is in it

**Records:** Leads, Opportunities, Companies, Contacts, Job Orders, Candidates, Submissions,
Appointments, Placements, Time Entries, Notes, Tasks, Tearsheets, Saved Searches.

**Seeded data:** 4 client companies, 6 contacts, 5 job orders, 11 live submissions, 2 placements,
time entries awaiting approval, and a candidate pool of **162 records — 150 with a full CV** across
Information Technology, Light Industrial, Admin & Clerical and Healthcare. The pool is generated
from a fixed seed, so every trainee gets identical data.

**Workflow:** New Lead → Internal Submission → Client Submission (the sendout) → Interview
Scheduled → Offer Extended → Placed, then placement approval, onboarding, and time entry approval.
Statuses move one step at a time and each move asks for what that stage actually requires.

**Boolean search:** `AND`, `OR`, `NOT` / `-`, brackets, `"exact phrases"`, `*` wildcards, and field
scoping such as `skills:aws`, `status:active`, `cv:"cycle counting"`. Every search prints how it
read the query, and every result shows which terms it matched on.

**Also:** CV upload, editing on every record type with field-level change logging, sortable list
columns, mass update, mass add to tearsheet or pipeline, reporting with data-quality flags, and a
full activity log.

## Training features

- **Four scenarios** in the practice panel: full desk cycle (17 steps), clear an inherited desk (7),
  weekly desk review (4), source from the CV database (8). Steps tick only when the underlying
  record is genuinely correct — nothing can be marked complete by hand.
- **Optional guided tour** on first run, skippable at any point and replayable from the Guide.
- **Knowledge check:** 12 questions, pass mark 8.
- **Session summary:** copyable text of steps completed, scores, records created and a timestamped
  action log, for the trainee to send to a trainer.
- The practice panel collapses to a button in the bottom-right corner so the application can be
  seen at full width. The choice is remembered.

---

## Before you use this for sign-off

This mirrors common staffing-platform structure and vocabulary. It is **not** a copy of any vendor
product, and it carries no vendor names, logos or branding.

Status lists, mandatory fields, approval routing and margin thresholds are all configurable in a
real system and **will differ from your configuration**. Check this against your own system and SOP
first and correct anything that does not match — teaching a status name your platform does not use
is worse than teaching none.

## Customising

Everything is in one file. The values worth editing first are near the top of the `<script>` block:

| Constant | Controls |
|---|---|
| `PIPE`, `PIPE_OUT` | submission statuses and closure reasons |
| `JO_STATUS`, `JO_TYPE` | job order statuses and types |
| `CD_STATUS`, `CO_STATUS`, `PL_STATUS` | candidate, company and placement statuses |
| `CATEGORIES` | the vertical list used across records and the CV pool |
| `ONBOARD` | the onboarding checklist |
| `NOTE_ACTIONS` | note action types |
| `VERTICALS` | roles, skills, certifications and employers used to generate the CV pool |
| `QUESTIONS` | the knowledge check |
| `SCENARIOS` | the practice task lists and their completion checks |

The only external request is a Google Fonts stylesheet. Remove that `<link>` if you need it fully
offline; there is a system font fallback and nothing else breaks.
