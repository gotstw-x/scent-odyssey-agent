# Skill: scent-evidence-retriever

## Purpose
Given a structured profile and a candidate perfume pool, retrieve and normalize “evidence” signals:
- note pyramid + accords
- public sentiment keywords (reddit-like)
- official description keywords
This skill can operate in 2 modes:
- MOCK MODE (offline, reproducible): use seeded DB fields only
- LIVE MODE (optional later): fetch from web + summarize

For HW2 benchmark rigor, default to MOCK MODE.

## Inputs
- profile: JSON from scent-intake-profiler
- candidates: array of perfume objects (from DB)
- mode: "mock" | "live"
- retrieval_budget: integer (max candidates to process), default 60

## Outputs
- evidence_pack: array where each item:
  - perfume_id, name, brand
  - notes_top/mid/base
  - accords (normalized 0–100)
  - price_min, price_max, concentrations
  - evidence_keywords:
      - reddit_like: array of {term, weight}
      - official_like: array of {term, weight}
      - merged_top_terms: array (top 20)
  - risk_tags: array (sweet, powdery, smoky, animalic, strong_projection, short_lasts, etc.)
  - availability_links: array
  - evidence_quality: 0–1 (mock=1.0 unless missing fields)

## Deterministic Normalization Rules (MOCK MODE)
1) If perfume has keywords[] in DB, split into:
   - reddit_like: terms tagged source="reddit" if available; else assume 50/50 random is NOT allowed.
   - In mock mode, store separately in DB as keyword_source_map or pre-annotated lists.
2) merged_top_terms = top terms by (weight) with duplicates removed, cap 20.
3) risk_tags derived from accords + keywords:
   - gourmand_sweet >= 60 -> "sweet"
   - powdery >= 55 or keyword includes "powdery" -> "powdery"
   - smoky >= 45 or keyword includes "smoky" -> "smoky"
   - musky_clean >= 70 AND keyword includes "laundry/soapy" -> "clean_musk"
   - projection_hint (if provided) >= 70 -> "strong_projection"

## Prompt (if LLM used in LIVE MODE)
You are an evidence extractor. You must:
- Extract only what is supported by the provided text snippets.
- Return a strict JSON evidence_pack schema.
- Never fabricate notes or prices.
- Mark evidence_quality lower (0.3–0.7) if snippets are sparse or contradictory.

## Example (MOCK MODE)
Input: profile + candidates(60 perfumes from DB)
Output: evidence_pack(60 items) with merged_top_terms and risk_tags.
