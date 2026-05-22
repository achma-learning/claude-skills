# qe-analysis

A Claude skill that turns a corpus of past medical exam questions (QEs — *questions d'examen*) into a high-yield, ranked, deduplicated study document — cross-matched against the year's official lesson program and exported in your chosen format (`.txt`, `.docx`, or `.pdf`).

> **Module-agnostic.** Built for any medical specialty : anatomie pathologique, médecine interne, traumatologie, gynécologie, cardiologie, pharmacologie, histologie, anatomie, etc. Works with any school's concours format (FMPM, FMPC, FMPR, CEM, ECNi).

---

## What it does

Given a question bank file (a text dump of many past exams, each question tagged with its topic), this skill produces a single document containing four mandatory sections plus a final cheatsheet :

| Section | Content |
|---|---|
| **PARTIE 1** | Lessons ranked by frequency (descending) — table view |
| **PARTIE 2** | Statistics : average questions per topic per exam, % of an exam per lesson |
| **PARTIE 3** | Deduplicated high-yield Q&A per topic, in strict rank order (1→N) |
| **PARTIE 4** | Topics **HORS programme** (cours annulé) — with a mandatory disclaimer |
| **Aide-mémoire** | Cross-cutting cheatsheet adapted to the subject |

The skill prefers a **deduplication-based strategy** (recommended) : if the same question appears across 4 sessions, it's listed *once* with all distinct correct answers merged and all the wrong-but-tempting traps (`⚠`) surfaced — because pitfalls are the highest-value content for exam prep.

---

## Quick example

> *— I have my Anapath QE file with 8 years of past papers, can you build me a high-yield revision document ?*

Claude triggers the skill and asks for five inputs :

1. 📄 **The question bank** — the `.txt` file you uploaded
2. 📚 **The official course PDF** *(optional — e.g. AIO.pdf, polycopié)*
3. 📋 **The list of this year's official lessons** (with professors)
4. 🔗 **A direct link per lesson** (Google Slides, Drive, etc.)
5. 📎 **Supplementary resources** *(optional — fiches résumées, additional courses)*

Then it :
1. Parses the question file (auto-detects format — FMPM-style, markdown headers, CSV)
2. Cross-matches each topic to an official lesson (with a French medical synonym dictionary : sein ↔ mammaire, foie ↔ hépatique, etc.)
3. Flags ambiguous matches for you to confirm
4. Deduplicates questions across sessions
5. Surfaces `⚠ FAUX` traps explicitly
6. Asks which format you want
7. Generates the file

---

## The strict ranking block format

Every lesson in **PARTIE 3** uses this exact header :

```
═══════════════════════════════════════════════════════════════════
[RANG 1] CANCER DU SEIN
Pr. Rais • ~6.79 Q/examen
📖 Classification OMS 2019 des tumeurs du sein
📎 Cours officiel : Ouvrir le cours officiel ↗
═══════════════════════════════════════════════════════════════════
```

- In `.txt`, the `📎` line shows the actual URL.
- In `.docx` and `.pdf`, "Ouvrir le cours officiel ↗" is a clickable hyperlink.

Below the header, content follows this pattern :

```
* Le grade SBR — 3 critères UNIQUEMENT :
-Différenciation glandulaire
-Atypies cytonucléaires
-Index mitotique
-⚠ NE FONT PAS PARTIE du SBR : stroma-réaction, emboles, localisation
📋 Vu : N2017 Q5, N2019 Q12, R2023 Q3 (3×)
```

---

## The HORS programme disclaimer

For any topic in the question bank that doesn't match an official lesson this year, the skill places it in **PARTIE 4** with a mandatory disclaimer :

> ⚠ **DISCLAIMER** — These lessons are NOT in the 2026 exam program.
> *Cours annulé — NOT officially scheduled.*
> **USE AT YOUR OWN RISK — BUT GOOD TO KNOW.**
>
> *Ces topics représentaient historiquement ~X% des questions des examens passés. S'ils ne sont PAS au programme cette année, ils peuvent malgré tout apparaître en :*
> - question résiduelle ou rattrapage
> - recyclage de dernière minute
> - culture générale médicale

Rendered as a red-bordered danger box in `.docx`/`.pdf`.

---

## Architecture

```
qe-analysis/
├── SKILL.md                         # Procedural workflow (entry point)
├── references/
│   ├── output_format_template.md   # The exact output document structure
│   ├── content_schema.md           # JSON schema for content.json (the renderer contract)
│   └── parsing_patterns.md         # Common question-bank file formats (A/B/C/D)
└── scripts/
    ├── analyze_questions.py        # Parse the question bank → analysis.json
    ├── match_lessons.py            # Cross-match topics → matched.json (with synonym dict)
    ├── generate_txt.py             # Render .txt from content.json
    ├── generate_docx.js            # Render .docx from content.json (Node.js + docx pkg)
    └── generate_pdf.py             # Convert .docx → .pdf via LibreOffice headless
```

### The 5-step pipeline

```
Question bank .txt
       │
       ▼
   analyze_questions.py ──► analysis.json
       │ (stats, sessions, topics, raw question text per topic)
       │
       ├─◄ Claude asks for : official lessons + links + cours PDF
       │   Builds lessons.json
       │
       ▼
   match_lessons.py ──► matched.json
       │ (in_program / hors_program split, ambiguous flags)
       │
       ├─◄ Claude reads each topic's raw questions, synthesizes facts + FAUX traps
       │   (Option B : dedup repeated questions, merge corrections, surface pitfalls)
       │   Writes content.json
       │
       ▼
   generate_{txt,docx,pdf} ──► final document(s) in /mnt/user-data/outputs/
```

---

## Scripts — usage details

### `analyze_questions.py`

```bash
python3 scripts/analyze_questions.py <question_bank.txt> --output analysis.json
```

Auto-detects the file format (Format A : FMPM headers `# Normal 2017 Q1 - topic`, Format B : markdown headers, Format C : CSV, Format D : unstructured → unsupported). Emits :
- `total_questions`, `total_sessions`, `avg_q_per_exam`, `year_start`, `year_end`
- `topics[]` with count, avg_per_exam, pct_of_exam
- `questions_by_topic{topic: [raw question texts]}`

### `match_lessons.py`

```bash
python3 scripts/match_lessons.py analysis.json lessons.json --output matched.json
```

Cross-matches each topic from the question bank against the official lessons using :
- A built-in French medical synonym dictionary (sein↔mammaire, foie↔hépatique, rein↔rénal, poumon↔pulmonaire/bronchique, etc. — easily extensible)
- Stemming + canonicalization (strip noise words like *tumeur, cancer, pathologie*)
- Jaccard scoring with containment bonus

Matches above 0.75 are auto-accepted. Below 0.90 they're flagged as **ambiguous** — the skill then asks you to confirm.

### `generate_txt.py` / `generate_docx.js` / `generate_pdf.py`

All three consume the same `content.json` (Claude-written) and produce the final document. See `references/content_schema.md` for the schema.

---

## Why deduplication (Option B) over a flat question list

The skill recommends Option B because :

- **Repeated questions = profs' priority signals.** If "SBR 3 critères" appears in 4 sessions, it's the most-likely-to-appear concept next year — not 4 separate items to memorize.
- **FAUX traps are the real teaching value.** What students fail on isn't the correct answer — it's the *wrong-but-tempting* distractor. Surfacing these explicitly with `⚠` gives concrete elimination signals during the exam.
- **Compact without losing distinct wordings.** Different sessions phrase the same concept differently. Merging into one entry preserves all distinct true facts and all distinct traps, without listing the question 4 times.
- **Citations preserve auditability.** Optional `📋 Vu : N2017 Q5, R2024 Q3 (2×)` lines tell you exactly where each fact came from.

---

## Adapting to other medical modules

The aide-mémoire section (final cheatsheet) is **subject-tailored**. Examples :

| Module | Aide-mémoire sections |
|---|---|
| Anatomie pathologique | OMS classifications by organ/year • IHC markers • Pitfalls • Carcinogenic sequences |
| Médecine interne | Critères diagnostiques (Glasgow, qSOFA…) • Scores pronostiques • Schémas thérapeutiques de référence |
| Cardiologie | Critères ECG • Biomarqueurs (kinetics) • Classifications NYHA/CCS • Scores CHA₂DS₂-VASc |
| Traumatologie | Classifications fractures (AO/OTA, Garden, Schatzker) • Scores de gravité (ISS) |
| Gynécologie | Classifications FIGO • ACR/BI-RADS • Scores de Bishop |
| Pharmacologie | Familles • Mécanismes d'action • Interactions • Contre-indications |
| Histologie | Caractéristiques tissulaires • Marqueurs cellulaires • Embryologie d'origine |

When you invoke the skill, Claude will ask which module so it can tailor the cheatsheet appropriately.

---

## Installation

Drop `qe-analysis.skill` into Claude's skills folder. Next time you upload a question bank and ask Claude to analyze it, the skill triggers automatically.

---

## Dependencies

- **Python 3** (for `analyze_questions.py`, `match_lessons.py`, `generate_txt.py`, `generate_pdf.py`)
- **Node.js + `docx` npm package** (for `generate_docx.js`) — already installed in Claude's environment
- **LibreOffice headless** (for `generate_pdf.py`) — already available via `/mnt/skills/public/docx/scripts/office/soffice.py`

No external dependencies beyond what Claude's runtime already provides.

---

## Limitations and what NOT to expect

- **Won't invent medical content.** If a topic has too few QEs and no lesson PDF reference, Claude will ask for more material rather than fabricate facts.
- **Synonym dictionary covers oncology/pathology well, less so for niche specialties.** Easy to extend in `scripts/match_lessons.py` (the `SYNONYMS` list at the top).
- **Format D (unstructured question dumps with no topic tags)** can't be analyzed — you need at least topic tagging.
- **The skill is designed for corpus-level analysis.** It's not for grading single MCQs or short Q&A pairs.

---

## Credits

Built for FMPM 2026 concours preparation, generalized to any medical module. The skill encodes a teaching strategy refined across iterative passes on anatomie pathologique question banks (695 QEs / 14 sessions / 2017→2025).
