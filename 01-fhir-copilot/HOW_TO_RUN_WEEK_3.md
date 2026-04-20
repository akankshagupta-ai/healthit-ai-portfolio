# How to run Project 1 in Week 3 (your 90-minute session)

This is your single-page playbook for the Week 3 task: run the v1 prompt against all 25 questions and log the results.

## Before you start (5 min)

- [ ] Open https://claude.ai → Sign in → Click "Projects" in the left sidebar → "+ New Project"
- [ ] Project name: `FHIR Query Copilot v1`
- [ ] Click "Set custom instructions" → paste the entire content of `prompts/v1-system-prompt.md` (the whole code block, including the schema)
- [ ] Click "Save"

## Run the eval (60 min)

- [ ] Open `evals/questions.csv` in Excel or Google Sheets
- [ ] Add four new columns to the right: `model_sql`, `parses`, `right_tables`, `right_intent`
- [ ] For each of the 25 questions:
  - Paste the question text into your Claude Project as a new message
  - Copy the SQL the model returns into the `model_sql` column
  - Score `parses` (1 or 0), `right_tables` (1 or 0), `right_intent` (1 or 0) using the `reference_sql` column as your guide
- [ ] Add a fifth column: `passes_all = parses * right_tables * right_intent` (just multiply the three)

## Compute and record the headline number (10 min)

- [ ] Sum the `passes_all` column
- [ ] Divide by 25
- [ ] Save the file as `evals/results-v1.csv`
- [ ] Open the project README and update the "Headline result" section:

  > v1 query validity rate: __ / 25 = __%

## Reflect and seed v2 (15 min)

- [ ] Look at the questions that failed. Cluster them into 2-3 failure types (e.g., "didn't use COUNT DISTINCT", "added extra prose", "used wrong description string")
- [ ] In `prompts/v2-system-prompt.md` (create the file), write at the top:

  > **What's new in v2:** [3 bullet fixes for the failure types you saw]

That's the work for Week 4 already scoped, before you even start it.

## Done — what you'll have at the end of the session

✅ A populated `results-v1.csv` with a real numeric metric
✅ An updated README headline result
✅ A 1-page list of what to fix in v2

You're now exactly on track for the Week 5 launch post.
