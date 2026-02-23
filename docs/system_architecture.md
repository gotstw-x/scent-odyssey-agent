# System Architecture (Path A + Lovable)

## Frontend Pages
1) Home: history + interactive map + CTA
2) Keywords: drag/fishing UI → selected_keywords
3) Sliders: constraints & style controls → ui_prefs
4) Results: top5 + next10 + buy queries + review drawer

## Backend Modules
- Seed DB tables:
  - perfumes
  - keywords
  - reviews
  - feedback_training_rows

## Agentic Skills Interfaces
### 1) scent-intake-profiler
Input: free text + selected_keywords + ui_prefs  
Output: profile JSON (constraints, accord_targets, inferred keywords)

### 2) scent-evidence-retriever
Input: profile + candidate perfumes  
Output: evidence_pack (merged_top_terms, risk_tags)

### 3) scent-rank-explainer
Input: profile + evidence_pack  
Output: best_matches (5) + inspiration_picks (10) + debug breakdown

### 4) benchmark-judge (optional)
Input: benchmark_cases + system vs baseline outputs  
Output: benchmark_report (scores + worst-case + next steps)

## Data Flow
UI → intake-profile → retrieve-evidence → rank-recommendations → results → reviews → feedback rows
