---
name: sanity-check
description: Check ML/data analysis setups before modeling and flag issues (leakage, imbalance, metrics, validation).
---

Memory files (relative to skill dir): `memory/preferences.md`, `memory/history.jsonl`, `memory/memory.md`.

## Step 1 — Extract

Parse user input for:
- `dataset`: description of features, size, source
- `target`: variable name + type (binary, multiclass, regression)
- `method`: algorithm + metric + validation strategy

If any are missing, ask once. Do not proceed with gaps.

## Step 2 — Load Context

1. Read `memory/preferences.md` — apply any user preferences to output style/depth.
2. Read `memory/memory.md` — check for known patterns relevant to this input.
3. Read `memory/history.jsonl` — load only lines where dataset type or method keyword matches current input. Ignore unrelated runs.

## Step 3 — Check Risks

Run all four checks. Flag only real issues; do not invent warnings.

**Leakage**
- Features derived from or correlated with future outcomes (e.g., post-event data, target-encoded at full dataset level)
- Features that wouldn't exist at prediction time
- Target encoding or normalization done before split

**Class Imbalance**
- Flag if target is binary/multiclass and minority class < ~10% (or clearly skewed)
- Note impact on chosen metric

**Metric Issues**
- Accuracy on imbalanced data: misleading
- AUC without calibration for probability outputs: warn if calibration matters
- RMSE vs MAE: flag if outliers present and method doesn't handle them
- Regression metric on classification target or vice versa

**Validation Issues**
- Random k-fold on time-series: invalid (future leakage)
- No stratification on imbalanced classification
- Test set too small (<10% or <100 samples)
- Hyperparameter tuning on test set

## Step 4 — Output

Format exactly as below. Omit any section with no issues.

```
LEAKAGE
- <flag>: <one-line reason>

CLASS IMBALANCE
- <flag>: <one-line reason>

METRICS
- <flag>: <one-line reason>

VALIDATION
- <flag>: <one-line reason>

FIXES
1. <actionable fix>
2. <actionable fix>
...
```

If no issues found in a category, omit that section entirely.
Keep total output under 250 words.

## Step 5 — Persist

Append to `memory/history.jsonl`:
```json
{"date": "<ISO date>", "dataset_keywords": ["..."], "method": "...", "flags": ["..."]}
```

If the same flag appears for the same method/domain in 2+ history entries, append a pattern note to `memory/memory.md`.

If user provides feedback (corrections, ratings, style preferences), append to `memory/preferences.md`.

## Stop Conditions

- Output delivered and history updated → done
- User provides feedback → update `memory/preferences.md`, re-run if requested
- Input too vague after one clarification attempt → tell user what's missing and stop
