skills/scent-intake-profiler.md
# Skill: scent-intake-profiler

## Purpose
Turn a beginner's vague perfume intent into a structured, machine-actionable preference profile.
Outputs a normalized JSON schema that downstream skills can use consistently.

## Inputs
- user_free_text: string (EN/ZH allowed)
- selected_keywords: array of strings (0–12; may be empty)
- ui_prefs: object from Page 3 sliders (budget, accords, scenario, mood, projection, concentration)

## Outputs
- profile: JSON object with:
  - language: "en" | "zh" | "mixed"
  - intent_summary: 1–2 sentence summary
  - constraints:
      - budget_min, budget_max (USD)
      - concentration_allowed: ["EDT","EDP","Parfum","Extrait"]
      - scenarios: array (office/date/daily/cozy/summer/winter)
      - projection_pref: 0–100 (skin ↔ strong)
      - mood_axis: 0–100 (intense ↔ calm)
      - hard_avoids: array (e.g., sweet, smoky, powdery, animalic, heavy amber)
  - accord_targets: object (0–100 each)
      - woody, floral, fresh_citrus, musky_clean, ambery_warm, gourmand_sweet, green, spicy, powdery, smoky, tea
  - keywords:
      - selected: array
      - inferred: array (synonyms and expansions)
  - clarification_questions: up to 3 questions (optional)
  - fallback_defaults_applied: boolean
  - confidence: 0–1

## Deterministic Rules (for reproducibility)
1) If selected_keywords is empty AND user_free_text is short (<12 tokens), set fallback_defaults_applied=true and use safe defaults:
   - scenarios=["daily","office"]
   - projection_pref=30 (skin-leaning)
   - mood_axis=60 (calm-leaning)
   - concentration_allowed=["EDT","EDP"]
   - accord_targets: fresh_citrus=60, musky_clean=55, woody=30, floral=25, gourmand_sweet=10, ambery_warm=20
2) Map common phrases to avoids:
   - "不甜" / "not sweet" -> hard_avoids includes "gourmand_sweet"
   - "不粉" / "no powdery" -> hard_avoids includes "powdery"
   - "不熏" / "no smoke" -> hard_avoids includes "smoky"
   - "晕香" / "headache" -> projection_pref <= 25 and add avoid "strong_projection"
3) Synonym expansion (inferred keywords) using a fixed dictionary (no internet):
   - clean -> soapy, laundry, aldehydic, musky
   - tea -> bergamot, mate, iris (if powdery not avoided), fig
   - woody -> cedar, sandalwood, vetiver
   - fresh -> citrus, aquatic, airy
4) Clarification questions only when critical fields are missing:
   - budget range missing
   - scenario missing
   - any strong avoid conflict (e.g., user selects "gourmand" but says "不甜")

## Prompt (LLM Instruction Template)
You are a scent preference profiler. Your job is to convert user intent into a structured profile JSON following the schema strictly.
- Always respect explicit avoids. If conflict exists, mark it and ask 1 clarification question.
- Do NOT invent personal facts. Only infer synonyms using the fixed dictionary.
- Keep clarification_questions <= 3. If user cannot answer, proceed with defaults.
- Output JSON ONLY.

## Example
### Input
user_free_text: "我想要很干净、像刚洗完澡，不要甜，通勤用，别太冲。预算$150以内。"
selected_keywords: ["clean","soapy","fresh"]
ui_prefs: {budget_min:50,budget_max:150, concentration:["EDP"], scenarios:["office","daily"], mood_axis:70, projection_pref:25, accords:{fresh_citrus:65,musky_clean:70,woody:25,floral:20,gourmand_sweet:5,ambery_warm:15,tea:10}}

### Output (JSON)
{
  "language":"zh",
  "intent_summary":"通勤日常的干净皂感清新香，低扩散、不甜，预算不超过$150。",
  "constraints":{
    "budget_min":50,"budget_max":150,
    "concentration_allowed":["EDP"],
    "scenarios":["office","daily"],
    "projection_pref":25,
    "mood_axis":70,
    "hard_avoids":["gourmand_sweet","strong_projection"]
  },
  "accord_targets":{
    "woody":25,"floral":20,"fresh_citrus":65,"musky_clean":70,"ambery_warm":15,
    "gourmand_sweet":5,"green":20,"spicy":10,"powdery":10,"smoky":0,"tea":10
  },
  "keywords":{"selected":["clean","soapy","fresh"],"inferred":["laundry","aldehydic","airy","citrus","musky"]},
  "clarification_questions":[],
  "fallback_defaults_applied":false,
  "confidence":0.86
}
