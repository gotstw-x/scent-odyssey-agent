# Skill: benchmark-judge

## Purpose
Evaluate the system output against a baseline using a consistent rubric.
This enables a rigorous mini-benchmark for HW2:
- multiple test cases
- baseline comparison
- edge + ambiguous cases
- frozen config for reproducibility

## Inputs
- test_cases: array where each case includes:
  - case_id
  - user_free_text
  - selected_keywords
  - ui_prefs
  - expected_constraints (optional)
- system_outputs: array of results JSON (from scent-rank-explainer)
- baseline_outputs: array (single-prompt baseline or manual baseline)
- rubric_config: fixed scoring rubric with anchors (1–5)

## Outputs
- benchmark_report: JSON + markdown summary:
  - per_case_scores (system vs baseline)
  - averaged_scores
  - worst_case_analysis
  - notes on hallucination/constraint violations
  - improvement_suggestions

## Rubric (1–5 each)
1) Preference Fit
2) Evidence Grounding
3) Constraint Satisfaction
4) Actionability
5) Safety & Honesty

Anchors:
- 1: major failures / frequent violations / vague
- 3: acceptable but inconsistent
- 5: strong alignment + evidence + actionable steps + honest uncertainty

## Deterministic Checks (non-LLM)
Before any subjective scoring:
- Count constraint violations:
  - budget_fail_count
  - avoided_tags_present
- Check explanation completeness:
  - each top-5 has matched_keywords>=3
  - each top-5 has risk_notes>=1
If violations exceed threshold, cap certain rubric dimensions (e.g., Constraint Satisfaction <=2).

## LLM-as-Judge (Optional)
If you use an LLM:
- Provide it the same rubric and require strict JSON output.
- The judge must cite which part of the recommendation text triggered deductions.
- The judge must not introduce new facts.

## Output Summary Template
- Table: case_id | system_avg | baseline_avg | biggest gap | notes
- Worst case: show input + top 3 items + why it failed
- Actionable next steps: 3 bullets (prompt tweaks / weight changes / keyword pool fixes)
