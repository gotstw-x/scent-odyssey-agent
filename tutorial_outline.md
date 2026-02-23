# Odyssey of Scent | 选香之旅 — Tutorial Outline (HW2 PDF)

## 1. Problem & Motivation
- Beginner perfume selection is hard because preferences are vague and reviews are noisy.
- Goal: translate vague intent into structured profile → retrieve evidence → rank & explain → collect feedback.

## 2. System Overview
- Frontend: 4-page experience (Home → Keywords → Sliders → Results + Community).
- Backend: seeded dataset (perfumes + keywords) and deterministic recommender.
- Agentic Skills (Path A):
  1) scent-intake-profiler
  2) scent-evidence-retriever
  3) scent-rank-explainer
  4) benchmark-judge (optional)

## 3. Data
- data/keywords.seed.json: bilingual keyword pool with category, weight, source.
- data/perfumes.seed.json: ~100 perfumes with accords, notes, price ranges, official-like + reddit-like keywords.

## 4. Build Steps (Reproducible)
1) Import seed JSON to DB (Supabase or local JSON).
2) Implement API hooks:
   - /api/intake-profile
   - /api/retrieve-evidence
   - /api/rank-recommendations
3) Connect UI states from Page2 & Page3 into the API calls.
4) Render Page4 results and enable reviews to feed back into DB.

## 5. Real Usage Runs (2 runs minimum)
- Run #1: your personal use case
- Iteration: adjust synonyms, avoid penalties, explanation template
- Run #2: different scenario or different user

## 6. Mini-Benchmark
- Baseline: single-prompt recommendation.
- 6 test cases: 2 normal + 2 edge + 2 ambiguous.
- Rubric: Preference Fit, Evidence Grounding, Constraint Satisfaction, Actionability, Safety & Honesty.
- Report: per-case scores, averages, worst-case analysis, fixes.

## 7. Limitations & Ethics
- Seed dataset bias; community bias; “sample first” disclaimer.
