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
- The Synthea CSV schema (see [schema.md](./schema.md))
- A handful of few-shot examples (see [v1-system-prompt.md](./v1-system-prompt.md))
- A repair loop added in v3 that sends parse errors back to the model

## What "good" looks like

A 25-question evaluation set ([questions.csv](./questions.csv)) covering 8 easy, 12 medium, and 5 hard questions. The headline metric is **query validity rate**: of the 25 questions, what percentage produces SQL that (a) parses, (b) runs without error against the Synthea schema, and (c) returns the correct answer.

| Version | Target | What changed |
|---|---|---|
| v1 | Baseline ~60-70% | Schema injection + 5 few-shot examples |
| v2 | 75-80% | Structured JSON output, tighter constraints |
| v3 | 85%+ | Self-repair loop on parse failure |

## Tech stack

- **LLM:** Claude (via claude.ai Projects) — primary. Prompt is portable to ChatGPT.
- **Data:** Synthea 100-patient sample (CSV export) — download in [data-source.md](./data-source.md)
- **Eval:** spreadsheet-based, no code required — rubric in [SCORING_RUBRIC.md](./SCORING_RUBRIC.md)

## Files in this folder

| File | Purpose |
|---|---|
| [README.md](./README.md) | This file |
| [HOW_TO_RUN_WEEK_3.md](./HOW_TO_RUN_WEEK_3.md) | Playbook for the 90-minute v1 evaluation session |
| [schema.md](./schema.md) | The 5 Synthea tables with FHIR resource mappings |
| [data-source.md](./data-source.md) | How to download Synthea and why it's safe |
| [v1-system-prompt.md](./v1-system-prompt.md) | The v1 system prompt (baseline) |
| [questions.csv](./questions.csv) | 25-question evaluation set with reference SQL |
| [SCORING_RUBRIC.md](./SCORING_RUBRIC.md) | How to score the model's outputs |

## Headline result *(updated when Project ships in Week 5)*

> Lifted query validity from **XX%** in v1 to **XX%** in v3 across a 25-question evaluation set, by introducing structured outputs and a self-repair loop.

## Data & safety

This project uses **only Synthea synthetic patient data**, generated specifically for software testing and research. No real Protected Health Information (PHI) is used, stored, or transmitted. The Synthea generator is open-source and freely redistributable.

## What V2 would look like

- Run against a real FHIR server (HAPI FHIR sandbox) to generate actual FHIR REST queries
- Add chart generation for trend questions
- Add a "show your work" mode where the model explains its decomposition
- Wire into a Streamlit or Gradio interface for non-technical end users
