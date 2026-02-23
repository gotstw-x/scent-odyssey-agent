What This Project Does
User Flow 

Home — perfume history + interactive world map + “Start the Scent Odyssey”

Keywords Fishing — drag floating keywords into “My Scent Basket” (supports EN/ZH + custom input)

Preference Sliders — budget, concentration, accord intensities, scenario, mood axis, projection

Results + Community — Top 5 best matches + 10 inspiration picks + buy links + review/feedback drawer

Agentic Workflow (Path A: Skills Pack)

This system is implemented as a modular, multi-step workflow:

Intake & Profiling (skills/scent-intake-profiler.md)
Converts vague intent + chosen keywords + slider prefs into a structured profile JSON.

Evidence Retrieval (skills/scent-evidence-retriever.md)
Builds an evidence pack per perfume from seeded “official-like” and “reddit-like” keywords + notes + accords.

Ranking & Explanation (skills/scent-rank-explainer.md)
Uses a deterministic, explainable scoring function to generate Top 5 + Next 10, with transparent constraint checks.

Benchmarking (Optional) (skills/benchmark-judge.md)
Scores system outputs vs baseline with a fixed rubric and highlights worst-case failures.

Data

All data in this repo is seeded for reproducibility (benchmark-friendly).

data/perfumes.seed.json
~100 perfume entries with: price ranges, concentration options, note pyramid, accord vector (0–100), tags,
and two keyword sources (official-like + reddit-like), plus search queries for purchase links.

data/keywords.seed.json
200+ bilingual keywords (EN + selected ZH), categorized (fresh/woody/floral/musky/tea/etc.) with weights and sources.

Deterministic Recommendation (Reproducible)

To keep benchmarking rigorous, the MVP uses a fixed scoring model:

Total Score

S = 0.45*K + 0.35*A + 0.20*C - P

Where:

K = weighted keyword overlap (selected + inferred vs evidence terms)

A = accord alignment (distance between user target accords and perfume accords)

C = constraint satisfaction (budget, scenario, concentration, projection)

P = penalties for explicit avoids (sweet/powdery/smoky/strong projection) and low evidence quality

Outputs:

Top 5 Best Matches

Next 10 Inspiration Picks (diversity-aware to avoid near-duplicates)

Community Feedback Loop

On the Results page, users can submit:

Match Accuracy (1–5)

Post-wear experience (free text)

Expected vs actual (optional)

Would recommend to similar keywords (yes/no)

These reviews can be stored in a DB table (e.g., Supabase) and logged as training rows for future iteration.

Mini-Benchmark

Benchmark materials live in docs/.

docs/benchmark_cases.json — 6 cases (2 normal, 2 edge, 2 ambiguous)

docs/benchmark_plan.md — baseline + rubric + frozen-config procedure

Baseline

Single-prompt LLM recommendation (no retrieval, no scoring breakdown)

Rubric (1–5)

Preference Fit

Evidence Grounding

Constraint Satisfaction

Actionability

Safety & Honesty

How to Reproduce (High Level)

Load data/*.json into your data layer (local JSON or Supabase tables).

Implement the 3 API hooks to mirror the skills interfaces:

/api/intake-profile

/api/retrieve-evidence

/api/rank-recommendations

Connect Page2 + Page3 UI state to the API calls, then render Page4 results.

Run benchmark cases and compare to baseline using the rubric in docs/benchmark_plan.md.

Docs

docs/architecture.png — system diagram

docs/system_architecture.md — data flow + components

docs/tutorial_outline.md — suggested HW2 PDF structure

Disclaimer

Recommendations are informational only. Always sample first and consider skin chemistry, allergies, and sensitivity. This project is not affiliated with any fragrance brand or retailer unless explicitly stated.
