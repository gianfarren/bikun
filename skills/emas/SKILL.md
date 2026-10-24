---
name: emas
description: Reads EMAS3 (emas3.ui.ac.id, Universitas Indonesia's Moodle course site) through the student's own logged-in session in the Claude desktop app's built-in browser. Use for "deadline minggu ini", "tugas apa yang belum dikumpul", "materi baru", "ambil slide X", "cek EMAS", reading a SAP / RPS / Kontrak Kuliah, and inside the bikun setup and scheduled runs. Read-only: never submits, posts, or changes anything.
---

# EMAS

EMAS3 runs on Moodle with its web service API enabled. Read it through Moodle's own AJAX endpoints inside the student's logged-in browser session, not by scraping pages.

Course ids, Drive folder ids, and preferences live in the student's config file (see **bikun:setup**, "Config file"). Load it first when you need any id.

## Rules

- **Read-only.** Never submit an assignment, post in a forum, mark an activity complete, enrol, or change a setting. If a task would need that, tell the student and stop.
- **Never type the student's password or SSO credentials.** If EMAS shows the login page, stop and say: "EMAS logout. Login dulu di browser Claude desktop, lalu bilang 'udah login'." In a scheduled run, put that line at the top of the notification and continue with what doesn't need EMAS.
- **Light use.** Fetch only what the task needs. No bulk downloads in loops, no crawling other students' pages, no polling more than the scheduled runs.
- **The student's data only.** Never read or report anything about other students.

## Opening EMAS

1. Load the `built-in-browser` skill, then the `Claude_Browser` tools (`mcp__Claude_Browser__*` or `mcp__remote-devices__Claude_Browser__*`).
2. Open `https://emas3.ui.ac.id/my/` with `preview_start`, or reuse an EMAS tab from `tabs_context`.
3. Check login with `javascript_tool`:
   `!!(window.M && M.cfg && M.cfg.sesskey) && !location.pathname.startsWith('/login')`
   False means logged out (see Rules).

## Calling the API

Run in the EMAS tab with `javascript_tool`:

```js
const call = async (methodname, args) => (await fetch('/lib/ajax/service.php?sesskey=' + M.cfg.sesskey + '&info=' + methodname, {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify([{index:0, methodname, args}])}).then(r=>r.json()))[0];
```

| Need | Method | Args |
|---|---|---|
| Courses this semester | `core_course_get_enrolled_courses_by_timeline_classification` | `{classification:'inprogress', limit:0, offset:0, sort:'fullname'}` |
| Upcoming deadlines + submission status | `core_calendar_get_action_events_by_timesort` | `{timesortfrom: now-86400, timesortto: now+N*86400, limitnum: 50}` (unix seconds) |
| Sections and items of a course | `core_courseformat_get_state` | `{courseid}`; `.data` is a JSON string with `section[]` (titles) and `cm[]` (`module`, `name`, `url`, `sectionid`) |

A result with `error: true` carries `exception.message`; report it plainly.

Deadline events: `name`, `course.shortname`, `timesort` (show in WIB, Asia/Jakarta), `action.name` ("Add submission" = not submitted yet).

EMAS's calendar only shows activities set up in EMAS. Kuis held in class, hardcopy tugas, and exams usually appear only in the SAP, which is why bikun keeps a schedule sheet.

## Reading a file (slides, SAP, RPS, kontrak)

Resource URLs look like `/mod/resource/view.php?id=N`. Fetch with `&redirect=1` and extract PDF text with pdf.js inside the page:

```js
const pdfjs = await import('https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.min.mjs');
pdfjs.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.worker.min.mjs';
const r = await fetch('/mod/resource/view.php?id=N&redirect=1', {credentials:'include'});
const ct = r.headers.get('content-type') || '';
if (!ct.includes('pdf')) { ({notPdf: ct, url: r.url}) } else {
  const doc = await pdfjs.getDocument({data: await r.arrayBuffer()}).promise;
  let t = ''; for (let i = 1; i <= doc.numPages; i++) { const c = await (await doc.getPage(i)).getTextContent(); t += c.items.map(x => x.str).join(' ') + '\n---page ' + i + '---\n'; }
  window.__bikun = t; ({pages: doc.numPages, chars: t.length})
}
```

Then read `window.__bikun` in slices of about 8,000 characters, normalising whitespace with `.replace(/\s+/g,' ')`.

- Math in slide PDFs often extracts garbled. Rebuild formulas from the textbook section the slide cites; never copy a garbled formula into a diktat.
- Scanned PDFs return no text. Say so, and use the textbook or ask the student for a readable copy.
- For a `.pptx` or `.docx`, report the file name and link; ask the student to export a PDF if the content is needed.

## Finding a course's SAP

In `core_courseformat_get_state`, look for `cm` names matching `/rps|sap|silabus|kontrak|rencana|syllabus|jadwal/i`. Prefer SAP or RPS over Kontrak Kuliah; read both when the SAP lacks dates. Some courses have no SAP on EMAS. Then use dated section titles ("Pekan 3: 11 September", "Kuliah Tanggal …") and mark kuis dates as unknown.
