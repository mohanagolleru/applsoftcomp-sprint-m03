---
name: sanity-check
version: 1.0
---

**Authors:** Mohana Siva Sai Yasodhar Golleru, Rahul Balamurugan

# PRD: sanity-check

## Goal

A personalized AI assistant that checks ML/data analysis setups before modeling and flags issues.

## Input

- Dataset description (features, size, source)
- Target variable (name, type)
- Method (algorithm, framework, evaluation approach)

## Output (short, direct)

- **Leakage risks** — features that could cause data leakage
- **Class imbalance issues** — target distribution problems
- **Metric issues** — wrong or misleading metrics for the task
- **Validation issues** — flawed CV/train-test split strategy
- **Fixes** — concrete, actionable steps to address each issue

## Architecture

### Steps

1. **Extract** — parse dataset description, target variable, method from user input
2. **Check** — run risk checks across all four issue categories
3. **Output** — emit short flagged report with fixes; persist run to history

### Memory System

| File | Purpose |
|---|---|
| `preferences.md` | User feedback on past responses; shapes future tone/depth |
| `history.jsonl` | One JSON line per run: input summary + flags raised |
| `memory.md` | Repeated patterns across runs (e.g. user always does time-series, prefers F1 over accuracy) |

Only load history entries relevant to current input (same domain/method keywords) — not full history.

## Requirements

- Responses are short. No fluff.
- Load only relevant past cases from history (keyword match on dataset type or method).
- Apply preferences from `preferences.md` before generating output.
- Append each run to `history.jsonl` after output.
- Update `memory.md` if a pattern appears 2+ times.

## Test Cases

### Test Case 1 — Leakage in tabular classification

**Input:**
```
Dataset: customer churn table with features including "days_since_last_contact", "churn_date", "support_tickets_after_churn"
Target: churn (binary)
Method: XGBoost, AUC, stratified 5-fold CV
```

**Evaluation Criteria:**
- Flags `churn_date` and `support_tickets_after_churn` as leakage
- Confirms AUC is appropriate for binary classification
- No false positives on `days_since_last_contact`
- Output is under 200 words

### Test Case 2 — Imbalanced medical classification

**Input:**
```
Dataset: 10,000 patient records, 2% positive (disease present)
Target: disease_present (binary)
Method: Logistic Regression, accuracy, random 80/20 split
```

**Evaluation Criteria:**
- Flags class imbalance (2% positive rate)
- Flags accuracy as misleading metric for imbalanced data
- Recommends AUROC, F1, or precision-recall curve
- Suggests stratified split or oversampling fix

### Test Case 3 — Time-series leakage

**Input:**
```
Dataset: daily stock prices 2010–2023, features include rolling 7-day mean
Target: next-day price direction (up/down)
Method: Random Forest, accuracy, random k-fold CV
```

**Evaluation Criteria:**
- Flags random k-fold as invalid for time-series (future leakage)
- Recommends walk-forward or time-based split
- Notes accuracy may be misleading if classes unbalanced

## How to Use

Run the skill in Claude Code by providing:
- dataset description
- target variable
- method

Example:
Dataset: 25 samples, binary classification  
Target: disease  
Method: logistic regression with accuracy  

The assistant returns risks and fixes. It updates preferences and history across runs.
