# Agent: Analyser Orchestrator

**Role:** Receives all raw items from the 6 source agents. Runs 3 analysis sub-agents in parallel. Merges their outputs into a single enriched dataset.
**Layer:** 2 (Analyser Orchestrator)

---

## Your Task

You receive a combined array of raw items from all 6 source agents (papers, github, news, youtube, blogs, podcast-community), tagged by source. Your job is to coordinate the 3 analyser sub-agents in parallel, then merge and sort their outputs.

---

## Step 1 — Validate Input

Before spawning sub-agents:
1. Count total items received — log this count
2. Check that items are tagged with `"source"` field
3. If any item is missing a required field, note it but continue
4. Log which sources are represented: e.g. "Received: 12 papers, 8 github, 15 news, 9 youtube, 11 blogs, 10 podcast-community = 65 total"

---

## Step 2 — Spawn 3 Sub-Agents IN PARALLEL

Immediately spawn all 3 sub-agents simultaneously, passing the full items array to each:

### Sub-agent 1: Dedup and Relevance
**File:** `agents/analyser/dedup-relevance-agent.md`
**Input:** Full items array
**What it does:** Removes duplicates, merges near-duplicates, scores each item 1–10
**Output:** Deduplicated items array with `relevance_score` and `sources_found_in` fields

### Sub-agent 2: Theme Clustering
**File:** `agents/analyser/theme-clustering-agent.md`
**Input:** Full items array
**What it does:** Groups items into 4–6 named topic clusters, identifies flagship items
**Output:** Items with `cluster`, `is_flagship` fields + `clusters_summary` object

### Sub-agent 3: Cross-Reference
**File:** `agents/analyser/cross-ref-agent.md`
**Input:** Full items array
**What it does:** Links related items across sources, calculates amplification scores
**Output:** Items with `related_items`, `amplification_score`, `cross_ref_note` fields

**CRITICAL: Do NOT run these sequentially. Spawn all 3 at the same time and wait for all to complete.**

---

## Step 3 — Merge Outputs

Once all 3 sub-agents return:

### Merge algorithm
1. Use the **dedup agent's output** as the canonical item list (it has the definitive deduplicated set)
2. For each item in the canonical list, add fields from the other two agents by matching on item ID:
   - From clustering agent: `cluster`, `secondary_cluster`, `cluster_fit`, `is_flagship`
   - From cross-ref agent: `related_items`, `amplification_score`, `cross_ref_note`
3. Also add the `clusters_summary` object from the clustering agent to the merged output

### Handling merge conflicts
- If an item appears in dedup output but not in clustering output (shouldn't happen but may): assign `"cluster": "uncategorised"` and log it
- If item IDs don't match: fall back to title-based matching

### Composite score calculation
After merging, calculate a composite score for each item:
```
composite_score = (relevance_score * 0.6) + (amplification_score * 0.4)
```
Add `composite_score` field to each item.

---

## Step 4 — Sort and Select Top Items

1. Sort all merged items by `composite_score` descending
2. Apply secondary sort: items with `is_flagship: true` move up within same score tier
3. Take the **top 30 items** (or all items if fewer than 30)
4. Ensure representation: if top 30 would exclude an entire cluster, promote the cluster's flagship item

---

## Step 5 — Output

Pass to `agents/insight-agent.md`:
1. **Enriched items array** — top 30 items, sorted, with all merged fields
2. **Clusters summary** — the `clusters_summary` object
3. **Run stats** — total items found, deduplicated count, sources that ran, any failures
4. **Team context** — pass through the original team context received from orchestrator (headcount, focus areas, member details)

### Output structure
```json
{
  "enriched_items": [...],
  "clusters_summary": {...},
  "run_stats": {
    "total_items_raw": 65,
    "total_items_after_dedup": 48,
    "top_items_selected": 30,
    "sources_succeeded": ["papers", "github", "news", "youtube", "blogs"],
    "sources_failed": ["podcast-community"]
  },
  "team_context": {
    "headcount": 4,
    "focus_areas": ["RAG", "agents"],
    "members": []
  }
}
```
