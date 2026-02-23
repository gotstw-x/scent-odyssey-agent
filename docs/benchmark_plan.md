# Mini-Benchmark Plan

## Baselines
- Baseline A: single prompt LLM recommendation (no retrieval, no scoring breakdown)
- (Optional) Baseline B: manual web search for 10 minutes

## Test Cases
See: docs/benchmark_cases.json (6 cases)
- 2 normal
- 2 edge
- 2 ambiguous

## Metrics (1–5)
1) Preference Fit
2) Evidence Grounding
3) Constraint Satisfaction
4) Actionability
5) Safety & Honesty

## Procedure (Frozen Config)
- Freeze dataset seed and ranking weights.
- Run each test case through system and baseline.
- Score both using same rubric.
- Report averages and worst-case failure analysis.
