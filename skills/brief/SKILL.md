---
name: brief
description: The two scheduled Bikun runs for a UI student, the morning brief (deadlines, new EMAS materials, priorities) and the evening run (H-1 reminder, Preview and Review diktats from the schedule). Also answers on demand "brief hari ini", "deadline minggu ini", "besok ada apa".
---

# Bikun brief

Before anything, read the config `bikun-config.json` from Google Drive (**bikun:setup**, "Config file"). If there is no config, tell the student to run setup: "Bikun belum di-setup. Ketik: setup bikun". Then stop.

Then:
1. Open EMAS and check the login (**bikun:emas**). If logged out, put "EMAS logout. Login dulu di browser Claude desktop." at the top of the final message and continue with the parts that don't need EMAS.
2. Read the schedule sheet (`drive.schedule_sheet`) with `read_file_content`.
3. Use today's date in Asia/Jakarta.

Write the final message in the student's language (config `language`), short and scannable. It must make sense on a phone lock screen: most urgent first.

## Morning brief

1. **Deadlines, next 7 days:** EMAS action events + schedule rows where `jenis` is `deadline`, `kuis`, `ujian`, or `praktikum`. For each: day, time (WIB), course, what it is. Mark EMAS assignments ✓ submitted or ✗ not yet (`action.name` "Add submission" = ✗).
2. **New on EMAS:** compare each course's item list (`core_courseformat_get_state` → `cm` ids and names) with the state file (`drive.state_file`, a JSON map of course → item ids). List new items by name. Then save the new state. On the first run, save the state and say "baseline disimpan".
3. **Fokus hari ini:** at most 5 objectives in order, one line each with the reason (deadline soonest first; quick almost-done tasks early; dated kuis before undated work).
4. **Ujian within 14 days:** one line asking "Mau dibuatkan diktat Ultimate untuk <ujian>?". Don't build it.

Do not build diktats in the morning run.

## Evening run

1. **H-1:** everything due tomorrow and not yet submitted (EMAS + schedule rows), with exact time and how to submit (EMAS upload, hardcopy, to asisten, etc.).
2. **Preview diktat:** for each of today's rows with `aksi_agent = preview-diktat` whose course has `preview: true`, build a Preview diktat (**bikun:diktat**, Preview mode) for tomorrow's lecture.
3. **Review diktat:** if `runs.review_diktat` is true, then for each of today's rows with `aksi_agent = review-diktat`, build a Review diktat (**bikun:diktat**, Review mode) for that kuis, with archives from **bikun:arsip**.
4. Upload each PDF to the course's `drive_folder` (`disableConversionToGoogleType: true`), named `<Mode>_<course>_<topic>_<YYYY-MM-DD>.pdf`.
5. If nothing applies for steps 2–3, skip them. Build nothing else.

Final message: the H-1 list first, then one line per diktat with its Drive link, then anything skipped and why (computer offline, EMAS logged out, no slides yet).

## Failure handling

- **Computer unreachable** (browser tools error): send the H-1 list and the deadlines from the schedule sheet only, and say EMAS could not be checked.
- **A diktat build fails:** send the rest of the message and say which diktat failed and why, in one line. Don't retry in a loop.
- **The schedule sheet disagrees with EMAS** (e.g., a different deadline time): trust EMAS for EMAS assignments, and mention the difference in one line.
