---
name: arsip
description: Finds past exams (kuis, UTS, UAS, ujian) and their solutions in Google Drive archive folders that a student has access to, such as a himpunan's bank soal. Use when the student says "tambahkan arsip", shares a Drive folder link of past exams, asks "ada arsip kuis X?", or when diktat-builder needs archives.
---

# Arsip

Many UI himpunan and departments keep past exams in shared Google Drive folders (e.g., a "Digitalisasi Soal" database). Bikun can use any number of them. Each one is registered in the student's config under `archives` (see **bikun:setup**, "Config file").

## Adding an archive drive

Trigger: "tambahkan arsip …", "arsip soal aku di <link>", or the archive step of setup.

1. Get the folder link or id from the student (`https://drive.google.com/drive/folders/<id>`).
2. Check access with Drive `get_file_metadata` on the id. No access → tell the student to open the link while signed in with the Google account connected to Claude, and use "Add shortcut to Drive" or ask the owner for access.
3. Map its structure, three levels deep at most: course folders → year/semester folders → assessment folders (KUIS 1, UTS, …). Use the listing method below.
4. Match course folders to the student's courses by name and code (e.g., `[PDB] PERSAMAAN DIFERENSIAL BIASA` ↔ "Persamaan Diferensial Biasa"). Show the matches and ask the student to confirm uncertain ones.
5. Save to the config: `{name, root_folder_id, courses: {<course short>: <folder id>}, added: <date>}`.
6. Report: which courses have archives, how many years per assessment, and which courses have none.

## Listing a folder reliably

The Drive connector's search is not reliable for shared archive folders. Use all three steps before saying a folder is empty or a file doesn't exist.

1. `search_files` with `parentId = '<id>'`, `excludeContentSnippets: true`. **Follow every `nextPageToken`**: the search often returns one item per page.
2. If that returns nothing, open `https://drive.google.com/drive/folders/<id>` in the built-in browser, wait about 3 seconds, and list items with `javascript_tool`:
   ```js
   [...new Map([...document.querySelectorAll('[data-id]')].filter(e => e.getAttribute('data-id').length > 20).map(e => [e.getAttribute('data-id'), (e.innerText||'').split('\n')[0]])).entries()]
   ```
3. Read each file with `read_file_content` using its id. This works even when search can't see the file.

If the browser shows a Google sign-in page, the browser is signed in to a different account (or none). Ask the student to sign in there with the account that has access. Never type their password.

## Finding archives for an assessment

Given a course and an assessment (e.g., PDB Kuis 2):
1. Start from `archives[*].courses[<course>]` in the config.
2. Walk year folders. Match assessment folders by name: `KUIS 2`, `Kuis 2`, `Q2`, `UTS`, `UAS`, `Ujian I`, and so on. The assessment number must match, and so must the scope when the folder names it.
3. Collect soal and solusi pairs by year. Typical file pattern: `<Course>_<Kuis n|UTS|UAS>_<year>_<Ganjil|Genap>_<Soal|Solusi>`.
4. Note the gaps: years with soal only, or no archive at all.

The scope of the same "Kuis 2" can differ between years and lecturers. Check each archive's topics against this semester's SAP scope, and leave out questions outside it, with a note.

## Use

Archives belong to their owners (himpunan, lecturers, assistants). Use them for the student's own study. Don't upload them anywhere public, and don't remove copyright lines from the files.
