# Scoring rubric (Project 1)

For each of the 25 questions, score the model's output against three criteria. The headline metric is **query validity rate** = the percentage of questions that pass all three.

## The three criteria

| # | Criterion | What "pass" looks like |
|---|---|---|
| 1 | **Parses** | The SQL is syntactically valid in standard SQL (PostgreSQL or DuckDB dialect). No missing parentheses, no broken keywords. |
| 2 | **Right tables & joins** | The query uses the correct tables from the schema (e.g., joins `patients` and `conditions` for Q9, not just one). |
| 3 | **Right intent** | The `WHERE`, `GROUP BY`, and aggregation match the question's intent. The answer would actually be the right answer to the question, *if* run against the data. |

A question **passes** only if all three criteria pass. This is a strict-by-design rubric — partial credit dilutes the headline number.

## How to score (no code required)

1. Open `questions.csv` in Excel or Google Sheets.
2. Add four new columns to the right: `model_sql`, `parses`, `right_tables`, `right_intent`.
3. For each question:
   - Paste the prompt + question into your LLM.
   - Copy its SQL output into `model_sql`.
   - Score `parses`, `right_tables`, `right_intent` as 1 (pass) or 0 (fail) by visual inspection. Use the `reference_sql` column as your guide — there are usually 2-3 valid ways to write the same query.
4. Add a fifth column: `passes_all = parses AND right_tables AND right_intent`.
5. Headline metric: `SUM(passes_all) / 25`.

## Common partial-credit traps (still scored as fail)

- The SQL is correct but it filters on the wrong description string (`'Diabetes'` instead of `'Diabetes mellitus type 2'` for Q9). → fail `right_intent`.
- The SQL counts duplicates (e.g., one patient with three diabetes condition rows is counted three times). → fail `right_intent`.
- The SQL hardcodes a year (`2024`) when the question says "in the past 12 months". → fail `right_intent`.
- The SQL is valid but joins the wrong table (uses `medications` when the question asks about labs). → fail `right_tables`.
- The model returns prose instead of SQL, or wraps SQL in extra commentary. → fail `parses` (until v2 fixes this with structured output).

## Acceptable variations (still scored as pass)

- Different but equivalent SQL syntax (e.g., subquery vs CTE, `INNER JOIN` vs implicit join).
- Slightly different LIKE patterns (e.g., `%Diabetes%` vs `%diabetes%`).
- Using `COUNT(DISTINCT patient)` vs adding `DISTINCT` in a subquery.
- Different but valid date functions across SQL dialects.

## Recording the headline number

After v1 scoring, fill in:

> **v1 query validity rate: __ / 25 = __%**

This is the number that goes in the README headline result. After v2 and v3, you'll have three numbers showing iteration — exactly the story recruiters want to see.
