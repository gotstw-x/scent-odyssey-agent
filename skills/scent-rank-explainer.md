# Skill: scent-rank-explainer

## Purpose
Rank perfumes for a given profile using an explainable scoring function.
Return:
- Top 5 "Best Matches"
- Next 10 "Inspiration Picks" (diverse but relevant)
Along with transparent “why it matches” explanations and constraint checks.

## Inputs
- profile: JSON
- evidence_pack: array from scent-evidence-retriever
- ranking_config (optional): weights and penalties (must be fixed for benchmark)

## Outputs
- results: JSON with:
  - best_matches: array(5) of recommendation cards
  - inspiration_picks: array(10)
  - debug:
      - weights_used
      - scoring_breakdown_per_item (top 20)
      - filters_applied
      - failure_reasons (if empty results)

## Deterministic Scoring (Reproducible)
Define:
- K = weighted keyword overlap score
- A = accord alignment score
- C = constraint satisfaction bonus
- P = penalties

Total Score S = 0.45*K + 0.35*A + 0.20*C - P

### K: Keyword overlap (0–100)
- Let user_terms = selected + inferred
- Let perfume_terms = merged_top_terms
- K = 100 * sum_{t in intersection}(w_user(t) * w_perf(t)) / normalization
- In MVP, set w_user(t)=1.0 for selected, 0.6 for inferred.

### A: Accord alignment (0–100)
- Compute distance between user accord_targets and perfume accords
- A = 100 - mean_absolute_error(user_targets, perfume_accords) scaled to 0–100

### C: Constraint bonus (0–100)
- budget fit:
  - if perfume.price_max <= profile.budget_max: +35
  - else if overlaps budget range slightly: +15
  - else: +0
- scenario fit:
  - if profile scenarios includes "office" and perfume has tag "office_safe": +25
  - if "date" and perfume has tag "date_night": +25
- concentration preference:
  - if any overlap: +20 else +0
- projection preference:
  - if profile wants calm/skin and perfume has strong_projection tag: +0 else +20

C is capped at 100.

### Penalties P (0–60)
- if hard_avoids includes "gourmand_sweet" and perfume sweet tag -> +25
- if avoid powdery and perfume powdery -> +20
- if avoid smoky and perfume smoky -> +20
- if avoid strong_projection and perfume strong_projection -> +15
- if evidence_quality < 0.6 -> +10

## Explainability Requirements
For every recommended item:
- Provide 3–5 matched keyword badges
- Provide 2 accord alignment highlights (e.g., "musky/clean high", "low gourmand")
- Provide 1–2 risk notes (e.g., “may feel powdery to some”)
- Provide constraint check lines:
  - Budget: PASS/WARN/FAIL
  - Projection: PASS/WARN
  - Scenario: PASS/WARN

## Diversity for Inspiration Picks
After selecting best_matches, pick next 10 using:
- relevance threshold (S >= best_match_5_score - 12)
- diversity penalty so you don't pick 10 near-duplicates (max 3 per brand; spread across accord families)

## Output Card Schema
{
  "perfume_id": "...",
  "name": "...",
  "brand": "...",
  "price_range": {"min":..,"max":..},
  "concentration": ["EDT","EDP"],
  "notes": {"top":[...],"mid":[...],"base":[...]},
  "matched_keywords": [...],
  "why_it_matches": [
     "Keyword overlap: ...",
     "Accord alignment: ...",
     "Scenario fit: ..."
  ],
  "risk_notes": [...],
  "constraint_checks": {"budget":"PASS","projection":"WARN","scenario":"PASS"},
  "buy_links": [{"label":"Official","url":"..."},{"label":"Retailer","url":"..."}],
  "score": 87.3
}

## Failure Modes / Fallback
If no results after filtering:
- Relax filters in order:
  1) keep avoids, relax budget to +20%
  2) broaden scenario tags
  3) reduce keyword strictness (use inferred only)
Return a message in debug.failure_reasons describing what was relaxed.

## Example
Input: profile + evidence_pack
Output: JSON with 5 best + 10 inspiration, plus debug breakdown.
