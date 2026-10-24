---
name: diktat
description: Builds a diktat (a LaTeX-compiled PDF study guide) for a UI course in three modes. Preview is the evening before a lecture (short, mostly intuition). Review is for an upcoming kuis (compact summary, past kuis from archives, 20+ practice problems). Ultimate is for UTS/UAS. Pulls its own sources from EMAS and archive drives. Trigger on "buat diktat", "diktat kuis/UTS/UAS", "preview materi besok", "rangkuman + latihan", and inside the Bikun evening run.
---

# Diktat

A diktat is the student's study guide: a LaTeX-compiled PDF. Correctness matters more than polish. A wrong answer key teaches a wrong method the night before a kuis.

Write in the student's language (config `language`, default Bahasa Indonesia), using the notation and terms of the course's own slides.

## Pick the mode

| Mode | When | Size |
|---|---|---|
| **Preview** | Evening before a lecture (`preview-diktat` row, or "preview materi besok") | 3–6 pages, mostly intuition |
| **Review** | A kuis / post-test is coming and its scope has been taught (`review-diktat` row, or "diktat kuis") | ~10–25 pages, mostly practice |
| **Ultimate** | UTS, UAS, Ujian I/II/III, only when the student asks | 40–100 pages |

In a scheduled run, the mode comes from the schedule row; don't ask. In a chat, ask once if unclear.

## 1. Scope

Course, topics, assessment format and date: take them from the schedule sheet and the SAP (e.g., "Kuis 2: Bab 5–6, 5 PG + 1 esai, 30 menit"). Scope = exactly what the SAP lists for that assessment.

## 2. Gather sources first

Read everything before writing LaTeX:
1. **EMAS:** this course's slides for the scope, the SAP, any posted exercises or past kuis (**bikun:emas**).
2. **Archive drives:** past exams for this assessment (**bikun:arsip**). Review and Ultimate only.
3. **Earlier diktats** in the course's Drive folder: match their format and don't repeat their problems.
4. **The textbook** named in the SAP, for anything the slides only sketch.

If slides for the topic aren't on EMAS yet (common for Preview), use the SAP's textbook section. If neither the SAP nor EMAS names the topic, infer it from the course sequence and say on the cover that it is a guess.

If fewer than 2 years of archive exist, say so in the delivery message and build anyway.

## 3. Check every answer

- Quantitative courses (math, stats, physics, chemistry, economics, CS): compute every numeric answer and every derivation in code before writing it (`sympy`, exact `fractions`, `scipy`, Python traces for algorithms). Never put an unchecked number in the PDF.
- Conceptual courses (law, social science, humanities): tie every claim to a slide, the SAP's textbook, or a cited source. Mark your own examples as examples.
- When a source contains an error (typo, wrong value, two correct PG options), flag it in an `awasbox` titled "Koreksi" with the fix.

## 4. Structure

### Preview (before a lecture)

The goal: the student walks into class already holding the big idea. No practice set, no archive.

1. **Pertanyaan besar:** the one question this lecture answers.
2. **Intuisi:** the idea in plain words, an analogy or picture, and a TikZ sketch where it helps.
3. **Hubungan:** how it connects to earlier material.
4. **Satu contoh kecil:** one short example, fully worked and checked.
5. **Istilah kunci:** a table of new terms and notation.
6. **Pertanyaan untuk dibawa ke kelas:** 2–3 questions to listen for answers to.

It should take 15–20 minutes to read.

### Review (before a kuis)

1. **Ringkasan** per topic (1–3 pages): definitions and key results in boxes, the standard procedure in a `langkahbox`, a one-line "kenapa" for each formula or rule, and typical traps in an `awasbox`.
2. **Arsip kuis:** every past kuis found, fully solved, by year.
3. **Latihan bertingkat** per topic: Dasar (4–6), Gabungan (4–6), Variasi sulit (2–4).
4. **Campuran:** 5–8 mixed problems with no topic labels, so the student has to choose the method.
5. **Simulasi kuis:** one set in the real format and time limit.
6. **Kunci jawaban:** a separate section at the end, with short answers first and then full solutions.
7. **Kartu rumus:** half a page.

**Example rule:** before each new problem type, include one fully worked example of that type (`contohbox`). No practice problem appears without a matching example earlier in its section.

### Ultimate (UTS / UAS)

Add an archive analysis first: a table of topics × years, question formats, recurring traps, and which archive is closest to this semester's style. Then:
1. Cover and topic map, with a scope note.
2. Theory chapters.
3. Solved exercises from slides, responsi, and tugas.
4. Solved archives by year.
5. A prediction package in the real format, with its answer key separate.
6. Exam strategy: a time plan and a common-mistakes checklist.
7. A formula sheet.

## 5. Build

- Source: `main.tex` plus one file per chapter, via `\input`. Write long documents in stages.
- `pdflatex` three passes. Fix overfull hboxes (`\quad` instead of `\qquad`, `align*` for long displays, `tabularx` for wide tables).
- Render a few pages to PNG and look at them before delivering.

Preamble:

```latex
\documentclass[11pt,a4paper]{article}
\usepackage[a4paper,margin=2.2cm]{geometry}
\usepackage{amsmath,amssymb,mathtools}
\usepackage[most]{tcolorbox}
\usepackage{tikz,pgfplots}\pgfplotsset{compat=1.18}
\usetikzlibrary{arrows.meta,calc,positioning}
\usepackage{tabularx,booktabs,enumitem,listings,xcolor,fancyhdr,hyperref}
\tcbset{diktat/.style={enhanced,breakable,fonttitle=\bfseries,arc=2pt,boxrule=0.6pt,left=6pt,right=6pt}}
\newtcolorbox{defbox}[1][]{diktat,colback=blue!4,colframe=blue!60!black,title=Definisi,#1}
\newtcolorbox{thmbox}[1][]{diktat,colback=violet!4,colframe=violet!60!black,title=Teorema,#1}
\newtcolorbox{langkahbox}[1][]{diktat,colback=teal!5,colframe=teal!60!black,title=Langkah baku,#1}
\newtcolorbox{contohbox}[1][]{diktat,colback=green!4,colframe=green!45!black,title=Contoh,#1}
\newtcolorbox{tipbox}[1][]{diktat,colback=orange!6,colframe=orange!70!black,title=Tips,#1}
\newtcolorbox{awasbox}[1][]{diktat,colback=red!5,colframe=red!65!black,title=Awas,#1}
\lstset{basicstyle=\ttfamily\small,breaklines=true,frame=single,columns=fullflexible}
\pagestyle{fancy}\fancyhf{}\rhead{\thepage}\lhead{\small Dibuat dengan Bikun}
```

## 6. Deliver

- **In a chat:** send the PDF and a zip of the source, plus one line: mode, page count, number of problems, anything flagged.
- **In a scheduled run:** upload the PDF to the course's Drive folder (see **bikun:brief**) and put the link in the run's message.

The diktat contains lecturers', assistants', and himpunan material. It is for the student's own study. Don't publish it.
