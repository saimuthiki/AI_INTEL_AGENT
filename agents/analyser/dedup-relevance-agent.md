# Agent: Dedup and Relevance Agent

**Role:** Remove duplicate items and score each unique item by its potential impact for an AI development team.
**Layer:** 2 (Analyser sub-agent — runs in parallel with theme-clustering and cross-ref agents)
**Skill references:** `skills/relevance-scorer.md`

---

## Your Task

You receive the full combined items array from all 6 source agents. Your job is to:
1. Identify and remove duplicate items
2. Merge near-duplicate items
3. Score each unique item 1–10 for relevance

---

## Step 1 — Exact Duplicate Removal

### What counts as an exact duplicate
- Same URL (ignoring query parameters like `?ref=`, `?utm_source=`)
- Same paper arXiv ID regardless of how linked
- Same GitHub repo regardless of which release URL was cited

### Algorithm
1. Normalise all URLs: strip query parameters, lowercase, remove trailing slashes
2. Build a URL lookup table
3. For any two items with the same normalised URL: keep the item with more complete data; discard the other
4. Log: "Removed X exact duplicates"

---

## Step 2 — Near-Duplicate Merging

### What counts as a near-duplicate
Items referring to the same real-world thing from different sources. Examples:
- A paper found on arXiv AND on HuggingFace Papers AND in a news article
- A library release found on GitHub AND covered by a VentureBeat article
- A YouTube video AND a blog post by the same author about the same event

### Merging rules
1. Keep BOTH items as separate entries (they contain different information)
2. BUT: link them with `"related_items"` cross-references
3. Do NOT collapse them into one — let the cross-ref agent handle the relationship
4. DO flag them: add `"appears_in_multiple_sources": true` and `"sources_found_in": ["papers", "news"]`

### How to detect near-duplicates
Match on ANY of:
- Same paper title (fuzzy match: > 85% similar)
- Same GitHub repo name from different sources
- Same news story covered by 2+ news outlets (same event, similar headline)
- Same model release mentioned across multiple source types

---

## Step 3 — Relevance Scoring

For each unique item, calculate a relevance score using the full rubric in `skills/relevance-scorer.md`.

### Summary of the rubric (refer to skills file for full detail)
Start at 5/10, then apply:

**Boosts:**
- Major lab release (OpenAI/Anthropic/Google/Meta/Mistral): +3
- Has working code/GitHub repo: +2
- New model or significant benchmark: +2
- Applicable to RAG/agents/LLM building: +2
- High engagement (see thresholds in skills file): +1 or +2
- Tutorial/how-to: +1
- Multi-source coverage: +1
- Open-source release: +1

**Penalties:**
- Opinion piece, no new technical content: -1
- Older than 5 days: -0.5
- Paywalled: -1
- Incremental update: -1
- Theoretical, no implementation: -0.5
- Requires prohibitive compute: -0.5

**Cap at 10, floor at 1.**

### Focus area boost
If focus areas were specified in the trigger:
- Items matching the focus areas get +1 bonus (applied last, after capping at 10)
- Document which focus area triggered the boost

---

## Step 4 — Output Format

For each item, add these fields:
```json
{
  "relevance_score": 8.5,
  "sources_found_in": ["papers", "blogs"],
  "appears_in_multiple_sources": true,
  "focus_boost_applied": true,
  "focus_boost_reason": "matches focus area: RAG",
  "dedup_notes": "Also found in blogs-agent (different article, cross-linked)"
}
```

Return the full items array (deduplicated) with these fields added. Preserve all original fields.

### Summary stats to include
```json
{
  "total_input_items": 65,
  "exact_duplicates_removed": 8,
  "unique_items_output": 57,
  "score_distribution": {
    "9-10": 4,
    "7-8": 12,
    "5-6": 25,
    "3-4": 14,
    "1-2": 2
  }
}
```

---

## Quality Rules

1. **Be conservative with dedup** — it is better to keep two slightly similar items than to accidentally remove something important
2. **Be accurate with scores** — justify every score with the rubric; don't score everything 8+
3. **The rubric is calibrated** — a typical week should have 2–5 items at 8+, not 20
4. **Always include score justification** in `dedup_notes` field for items scoring 8+
