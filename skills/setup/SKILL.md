---
name: setup
description: First-time setup of Bikun, a personal EMAS3 agent for Universitas Indonesia students. Reads the student's courses and SAPs from EMAS, builds a semester schedule in their Google Drive, registers archive drives, and creates the morning brief and evening run scheduled tasks. Trigger on "/bikun:setup", "setup bikun", "pasang bikun", or when another bikun skill finds no config file. Also handles changes later: "ubah jam brief", "tambah matkul", "jadwal kuis berubah", "matikan bikun".
---

# Bikun setup

Bikun is an unofficial student tool, not affiliated with Universitas Indonesia. Speak to the student in their language (default Bahasa Indonesia, casual and clear). Keep each message short; one question at a time.

Use a task list for setup, one item per step below.

## Step 0: Check prerequisites

Bikun needs all of these. Check silently, then name only what's missing and how to fix it:

| Need | Check | If missing |
|---|---|---|
| Built-in browser on their computer | `Claude_Browser` tools present (or `enable__mcp__remote-devices__Claude_Browser`) | Open this chat in the Claude desktop app on their computer |
| Google Drive connector | Drive tools present | Connect Google Drive in claude.ai connector settings |
| Scheduled tasks | `create_trigger` (or this session's scheduling tool) present | Scheduled runs unavailable; setup can still build the schedule and Drive folders |

Look for an existing config first (see "Config file"). If one exists, this is a change, not a fresh setup. Jump to "Changes after setup".

## Step 1: EMAS login

Follow **bikun:emas** "Opening EMAS". Ask the student to log in to EMAS in the browser pane themselves (SSO UI). Never type their credentials. Wait for "udah login".

## Step 2: Courses

List in-progress courses (**bikun:emas**). Show them as a numbered list with a short name you propose (e.g., "Persamaan Diferensial Biasa - PDB (B)" → `PDB`). Ask which to include (default: all) and whether any short names should change.

## Step 3: Read SAPs and build the schedule

For each course, find and read its SAP / RPS / Kontrak Kuliah (**bikun:emas** "Finding a course's SAP"). From each, extract only rows from today until the end of the semester:

| Column | Content |
|---|---|
| `tanggal` | YYYY-MM-DD (resolve "Senin, 2 Nov" with the semester year; ranges like "9–13 Nov" use the first day) |
| `hari` | Indonesian weekday, checked against the date |
| `matkul` | short name |
| `jenis` | `kuliah`, `kuis`, `ujian`, `deadline`, `praktikum`, `preview` |
| `materi` | topic / chapter / subbab |
| `aksi_agent` | `preview-diktat`, `review-diktat`, or empty |
| `catatan` | time, room, format (e.g., "3 PG + 1 esai, 50 menit"), collection rules |
| `sumber` | which document the row came from |

Rules:
- **`review-diktat`** goes on the date of the **last lecture that covers a kuis's scope** (the evening after the chapter is finished). Only for kuis or post-tests that have a date in the SAP.
- **`preview-diktat`** rows go on the **day before** each lecture of the courses the student picks in Step 6. Add them after Step 6.
- Unknown kuis dates ("tak terjadwal", "random", no date given): no Review row; list them in the setup report.
- Check every weekday against its date in code (`datetime`). Fix mismatches before saving.

Then show the student a short summary per course (next kuis, next ujian, tugas deadlines) and ask them to check the dates. SAP dates can be wrong.

## Step 4: Archive drives

Ask once: "Kamu punya Google Drive arsip soal (misal bank soal himpunan) yang bisa kamu akses? Kirim link foldernya, atau bilang 'skip'." For each link, follow **bikun:arsip** "Adding an archive drive". Tell them they can add more anytime with: `Bikun, tambahkan arsip: <link folder>`.

## Step 5: Drive folders

Create in the student's Drive (root): `Bikun/`, one subfolder per course, the schedule sheet `Jadwal <semester> (Bikun)` (upload CSV with `contentMimeType: text/csv` so it becomes a Google Sheet), and the config file (below).

## Step 6: Preferences

Ask in one message, with defaults:
1. Morning brief time (default 05:40 WIB).
2. Evening run time (default 18:42 WIB).
3. Which courses get a **Preview diktat** the evening before each lecture (default: none; suggest the 1–2 hardest).
4. Automatic **Review diktat** before dated kuis (default: yes).

Avoid exact :00 or :30 times; many scheduled tasks start then, and those runs can be delayed. Move a requested 06:00 to 05:4x.

## Step 7: Scheduled tasks

Create two recurring scheduled tasks that need the student's computer (`requires_local_device: true`), cron in `CRON_TZ=Asia/Jakarta`, with push notifications on. Use exactly these prompts:

**Bikun morning brief**
```
Run the Bikun morning brief for today (Asia/Jakarta). Load the bikun:brief skill and follow its "Morning brief" section exactly. Read the config file bikun-config.json from Google Drive first. Read-only on EMAS; never type the student's password. Finish with one short message to the student via SendUserMessage.
```

**Bikun evening run**
```
Run the Bikun evening run for today (Asia/Jakarta). Load the bikun:brief skill and follow its "Evening run" section exactly. Read the config file bikun-config.json from Google Drive first. Read-only on EMAS; never type the student's password. Build only the diktats today's schedule rows ask for. Finish with one short message to the student via SendUserMessage.
```

Save both task ids in the config. Tell the student in one sentence whether the tasks run automatically or ask for approval (from the tool result), and that their computer must be on with the Claude desktop app open at those times.

## Step 8: Report

One short message:
- what was set up (courses, schedule rows, archives, run times)
- kuis with no date (no automatic Review)
- the link to the `Bikun` folder
- the first run time

## Config file

`Bikun/bikun-config.json` in the student's Drive. Find it with `search_files` using `title = 'bikun-config.json'`; read it with `read_file_content`.

```json
{
  "version": 1,
  "semester": "Gasal 2026/2027",
  "language": "id",
  "drive": {"root": "<id>", "schedule_sheet": "<id>", "state_file": "<id or null>"},
  "courses": [
    {"short": "PDB", "name": "Persamaan Diferensial Biasa - PDB (B)", "emas_id": 0,
     "sap_resource_ids": [0], "drive_folder": "<id>", "preview": false}
  ],
  "archives": [
    {"name": "Bank soal himpunan", "root_folder_id": "<id>", "courses": {"PDB": "<folder id>"}, "added": "YYYY-MM-DD"}
  ],
  "runs": {"brief": "05:40", "evening": "18:42", "review_diktat": true},
  "scheduled_tasks": {"brief": "<id>", "evening": "<id>"}
}
```

The Drive connector can't edit a file's content in place. To update the config or state file, create the new version with `create_file` (`disableConversionToGoogleType: true`, `contentMimeType: application/json`) in the `Bikun` folder, then move the old one to trash with `trash_file`. If two copies exist, use the newest `modifiedTime`.

## Changes after setup

| Student says | Do |
|---|---|
| "jadwal kuis X berubah ke …" | Update the schedule sheet rows (move `review-diktat` to the new last-lecture date) and confirm |
| "tambah matkul …" / "hapus matkul …" | Update config, folders, and schedule |
| "tambahkan arsip: <link>" | **bikun:arsip** |
| "ubah jam brief ke …" | Update the scheduled task's cron and the config |
| "preview untuk <matkul>" / "stop preview" | Update `courses[].preview` and the schedule's preview rows |
| "matikan bikun" | Disable both scheduled tasks (don't delete); keep the Drive folder |
| New semester | Run full setup again; keep `archives` |
