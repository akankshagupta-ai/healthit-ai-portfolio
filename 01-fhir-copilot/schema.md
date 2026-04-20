# Synthea schema (5 tables) and FHIR mapping

This is the schema the FHIR Query Copilot is grounded on. It's a flattened CSV export of Synthea synthetic patient records. Each table corresponds to one or more standard FHIR resources — the mapping is shown for each.

> **Why a flattened SQL schema instead of raw FHIR?** This is what real public-health analysts actually work with: SQL over flat tables exported from a FHIR server (or directly from an EHR's SQL backend). It is more universally readable than FHIR REST queries, and easier for an evaluator to score.

---

## 1. `patients`

One row per patient. Maps to FHIR resource: **Patient**.

| Column | Type | FHIR field | Notes |
|---|---|---|---|
| `id` | string (UUID) | `Patient.id` | Primary key |
| `birthdate` | date | `Patient.birthDate` | YYYY-MM-DD |
| `deathdate` | date or null | `Patient.deceasedDateTime` | Null if patient is alive |
| `gender` | enum: `M`, `F` | `Patient.gender` | |
| `race` | string | `Patient.extension[us-core-race]` | white, black, asian, other |
| `ethnicity` | string | `Patient.extension[us-core-ethnicity]` | hispanic, nonhispanic |
| `state` | string | `Patient.address[0].state` | US state code |
| `county` | string | `Patient.address[0].district` | US county name |

---

## 2. `conditions`

One row per diagnosed condition per patient (a patient can have many). Maps to FHIR resource: **Condition**.

| Column | Type | FHIR field | Notes |
|---|---|---|---|
| `patient` | string (UUID) | `Condition.subject.reference` | Foreign key → `patients.id` |
| `start` | date | `Condition.onsetDateTime` | When the condition was diagnosed |
| `stop` | date or null | `Condition.abatementDateTime` | Null if condition is ongoing |
| `code` | string (SNOMED) | `Condition.code.coding[0].code` | SNOMED CT code |
| `description` | string | `Condition.code.coding[0].display` | Human-readable, e.g., "Diabetes mellitus type 2" |

---

## 3. `observations`

One row per measurement / lab result / vital sign per patient. Maps to FHIR resource: **Observation**.

| Column | Type | FHIR field | Notes |
|---|---|---|---|
| `patient` | string (UUID) | `Observation.subject.reference` | Foreign key → `patients.id` |
| `date` | date | `Observation.effectiveDateTime` | When the observation was taken |
| `code` | string (LOINC) | `Observation.code.coding[0].code` | LOINC code |
| `description` | string | `Observation.code.coding[0].display` | Human-readable, e.g., "Hemoglobin A1c/Hemoglobin.total in Blood" |
| `value` | numeric | `Observation.valueQuantity.value` | The measured value |
| `units` | string | `Observation.valueQuantity.unit` | e.g., "%", "mg/dL" |

---

## 4. `encounters`

One row per healthcare visit per patient. Maps to FHIR resource: **Encounter**.

| Column | Type | FHIR field | Notes |
|---|---|---|---|
| `id` | string (UUID) | `Encounter.id` | Primary key |
| `patient` | string (UUID) | `Encounter.subject.reference` | Foreign key → `patients.id` |
| `start` | datetime | `Encounter.period.start` | When the visit started |
| `stop` | datetime | `Encounter.period.end` | When the visit ended |
| `encounter_class` | enum | `Encounter.class.code` | `ambulatory`, `emergency`, `inpatient`, `wellness` |
| `description` | string | `Encounter.type[0].text` | e.g., "Encounter for check-up" |

---

## 5. `medications`

One row per medication prescribed per patient (a patient can have many). Maps to FHIR resource: **MedicationRequest**.

| Column | Type | FHIR field | Notes |
|---|---|---|---|
| `patient` | string (UUID) | `MedicationRequest.subject.reference` | Foreign key → `patients.id` |
| `start` | date | `MedicationRequest.authoredOn` | When the prescription was written |
| `stop` | date or null | `MedicationRequest.dispenseRequest.validityPeriod.end` | Null if ongoing |
| `code` | string (RxNorm) | `MedicationRequest.medicationCodeableConcept.coding[0].code` | RxNorm code |
| `description` | string | `MedicationRequest.medicationCodeableConcept.coding[0].display` | Human-readable, e.g., "Metformin 500 MG Oral Tablet" |

---

## Common joins

```
patients ← conditions     (one patient → many conditions)
patients ← observations   (one patient → many observations)
patients ← encounters     (one patient → many encounters)
patients ← medications    (one patient → many medications)
```

Most analytics questions join `patients` with one of the other four tables, then filter on a code or description, and aggregate (count, average, percent).

## Common code patterns analysts ask about

| Topic | Find via |
|---|---|
| Type 2 diabetes | `conditions.description LIKE '%Diabetes mellitus type 2%'` or SNOMED 44054006 |
| Hypertension | `conditions.description LIKE '%Hypertension%'` |
| HbA1c lab result | `observations.description LIKE '%Hemoglobin A1c%'` |
| BMI | `observations.description LIKE '%Body Mass Index%'` |
| Metformin | `medications.description LIKE '%Metformin%'` |
| Statins | `medications.description LIKE '%Atorvastatin%' OR description LIKE '%Simvastatin%' OR description LIKE '%Rosuvastatin%'` |
| Emergency visits | `encounters.encounter_class = 'emergency'` |

These shortcuts are the bedrock few-shot examples in the system prompt.
