# Data source: Synthea

## What is Synthea

[Synthea](https://synthetichealth.github.io/synthea/) is an open-source patient population simulator developed by The MITRE Corporation. It generates **fully synthetic** (meaning: completely made-up, not de-identified real patient data) patient records that look and feel like real EHR data. It is the standard free dataset for healthcare AI and interoperability research.

- GitHub: https://github.com/synthetichealth/synthea
- Sample downloads: https://synthetichealth.github.io/synthea-sample-data/downloads/download.html
- License: Apache 2.0 (free for any use)

## Why Synthea for this project

- **Zero PHI risk** — every patient is fabricated. Safe to put on a public GitHub repo.
- **FHIR-native** — Synthea can export in FHIR JSON or flat CSV. Both map to the same schema.
- **Realistic distributions** — diagnoses, lab values, and treatment patterns are clinically plausible.
- **Free** — no licensing hurdles for a portfolio project.

## Getting the data

For this project, download the **100-sample CSV export**:

1. Visit https://synthetichealth.github.io/synthea-sample-data/downloads/download.html
2. Scroll to "100 Sample Synthetic Patient Records, CSV: 7 MB"
3. Download and unzip. You get ~20 CSV files.
4. We use only 5: `patients.csv`, `conditions.csv`, `observations.csv`, `encounters.csv`, `medications.csv`.

That's it — no API keys, no installation, no licenses.

## Using the data in the evaluation

For scoring v1, you don't need to actually run the SQL. You can visually inspect each generated query and check:

1. **Parses?** — is it syntactically valid SQL?
2. **Correct tables?** — does it join the right tables from the schema?
3. **Correct filter?** — does the `WHERE` clause match the question's intent?

For more rigorous scoring (Week 4+), load the 5 CSVs into SQLite or DuckDB and actually execute each query. DuckDB is simplest:

```sql
-- Load in DuckDB (free, no install beyond the CLI)
CREATE TABLE patients     AS SELECT * FROM read_csv_auto('patients.csv');
CREATE TABLE conditions   AS SELECT * FROM read_csv_auto('conditions.csv');
CREATE TABLE observations AS SELECT * FROM read_csv_auto('observations.csv');
CREATE TABLE encounters   AS SELECT * FROM read_csv_auto('encounters.csv');
CREATE TABLE medications  AS SELECT * FROM read_csv_auto('medications.csv');
```

But you can absolutely ship v1 without running SQL at all — pure visual scoring is how most prompt-engineering evaluations start.
