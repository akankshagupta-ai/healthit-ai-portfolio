# Project 1 — FHIR Query Copilot

> Plain-English questions → valid SQL queries over Synthea synthetic patient data, with a one-line explanation of which FHIR resources are involved.

**Status:** in progress · **Target completion:** Week 5
**Author:** Akanksha Gupta
**Data:** [Synthea](https://synthetichealth.github.io/synthea/) — open-source synthetic patient records (no PHI, ever)

---

## The problem

Public-health and Health IT analysts spend hours writing SQL queries against patient databases. Most of these queries are variations of the same patterns: counts, distributions, time-trends, cross-tabulations. The team's clinical leaders, program managers, and PMs typically can't write SQL themselves, so they wait in line for an analyst — slowing every decision.

A reliable natural-language-to-SQL layer would let non-technical staff get answers in seconds, freeing analysts for higher-value work.

## The user

A **public-health program manager** at a state health department or a managed-care organization. She knows her domain (diabetes prevention, opioid surveillance, immunization coverage) but cannot write SQL. She has access to a flat-file export of patient data structured in the Synthea / OMOP / FHIR family of schemas.

## The approach

A prompt-engineered assistant that:

1. Receives a plain-English question (e.g., *"How many patients aged 50 or older have a diabetes diagnosis?"*)
2. Returns a valid SQL query against the documented schema
3. Returns a one-line explanation that names the FHIR resources behind each table

The assistant is grounded on:
- The Synthea CSV schema (5 tables documented in [`docs/schema.md`](./docs/schema.md))
- A handful of few-shot examples (see [`prompts/v1-system-prompt.md`](./prompts/v1-system-prompt.md))
- A repair loop: if the SQL fails to parse, the model gets the parse error back and tries again

## What "good" looks like

A 25-question evaluation set ([`evals/questions.csv`](./evals/questions.csv)) covering 8 easy, 12 medium, and 5 hard questions. The headline metric is **query validity rate**: of the 25 questions, what percentage produces SQL that (a) parses, (b) runs without error against the Synthea schema, and (c) returns the correct answer.

| Version | Target | What changed |
|---|---|---|
| v1 | Baseline ~60-70% | Schema injection + 5 few-shot examples |
| v2 | 75-80% | Add structured output (JSON with `sql` and `explanation` fields), tighter constraints |
| v3 | 85%+ | Add a self-repair loop: failed queries get the error message and one retry |

## Tech stack

- **LLM:** Claude (via [claude.ai Projects](https://claude.ai)) — primary. Prompt is portable to ChatGPT.
- **Data:** Synthea 100-patient sample (CSV export)
- **Eval:** spreadsheet-based, no code required (run each prompt by hand, log results)
- **Optional:** SQLite or DuckDB to actually execute the SQL — covered in Week 4

## Repo structure

```
01-fhir-copilot/
├── README.md                       (this file)
├── docs/
│   ├── schema.md                   (the 5 Synthea tables + FHIR mapping)
│   └── data-source.md              (how to download Synthea, where it came from)
├── prompts/
│   ├── v1-system-prompt.md         (baseline)
│   ├── v2-system-prompt.md         (added in Week 4)
│   └── v3-system-prompt.md         (added in Week 4)
└── evals/
    ├── questions.csv               (the 25-question eval set)
    ├── results-v1.csv              (filled in Week 3)
    ├── results-v2.csv              (filled in Week 4)
    └── results-v3.csv              (filled in Week 4)
```

## Headline result *(updated when Project ships in Week 5)*

> Lifted query validity from **XX%** in v1 to **XX%** in v3 across a 25-question evaluation set, by introducing structured outputs and a self-repair loop.

## Data & safety

This project uses **only Synthea synthetic patient data**, generated specifically for software testing and research. No real Protected Health Information (PHI) is used, stored, or transmitted. The Synthea generator is open-source and freely redistributable.

## What V2 would look like (the "future work" section recruiters love)

- Run against a real FHIR server (HAPI FHIR sandbox) to generate actual FHIR REST queries instead of SQL
- Add chart generation (return a plot for trend questions, not just a table)
- Add a "show your work" mode: the model explains its decomposition before writing SQL
- Wire into a Streamlit or Gradio interface for non-technical end users
