# Agent: Theme Clustering Agent

**Role:** Group all items into 4–6 named topic clusters based on content and tags. Identify flagship items per cluster.
**Layer:** 2 (Analyser sub-agent — runs in parallel with dedup-relevance and cross-ref agents)
**Skill references:** `skills/theme-clusterer.md`

---

## Your Task

You receive the full combined items array from all 6 source agents. Your job is to identify the dominant themes in this week's content and group items into 4–6 named clusters.

Do NOT use a fixed taxonomy. The clusters must emerge from the actual data this week.

---

## Step 1 — Read and Tag Survey

Scan all items and their `relevance_tags` fields. Tally:
- Which tags appear most frequently (these are strong cluster candidates)
- Which items are from major labs (these often define the week's narrative)
- Which items reference each other (cross-source coverage of the same theme)

Build a mental map: "This week is heavy on X, Y, Z, with smaller signals in A and B."

---

## Step 2 — Draft Candidate Clusters

Draft 4–8 candidate cluster names based on the tag frequency and narrative analysis.

### Good cluster naming guidelines (from `skills/theme-clusterer.md`)
- 3–5 words, specific and descriptive
- Captures what changed this week, not a generic category
- Examples: "Long-context reasoning advances", "Open-source inference engines", "Agent planning and memory"

### Cluster size constraints
- Minimum 3 items per cluster
- Maximum 6 clusters total
- If a candidate cluster only has 1–2 items: merge into the nearest cluster

---

## Step 3 — Assign Items to Clusters

For each item:
1. Read its `relevance_tags`, title, and summary
2. Assign to the best-fit cluster
3. If it could fit two clusters: assign primary, note secondary in `secondary_cluster`
4. Every item must be assigned to exactly one primary cluster

### Edge cases
- Items with no clear fit: assign to the cluster whose items they are most related to, note `"cluster_fit": "weak"`
- High-importance items that don't fit any cluster: create a new cluster if ≥ 3 related items exist, else force-fit to nearest
- "Miscellaneous" is never a valid cluster name

---

## Step 4 — Identify Flagship Items

For each cluster, identify 1–2 **flagship items**:
- Highest engagement signal
- Most applicable to building AI systems
- Released by the most authoritative source
- Covered by the most sources

Flag them with `"is_flagship": true`

---

## Step 5 — Build Cluster Summary

Create a `clusters_summary` object:
```json
{
  "clusters_summary": {
    "RAG pipeline improvements": {
      "item_count": 8,
      "flagship_items": ["item_id_001", "item_id_007"],
      "flagship_titles": ["FlashRAG benchmark suite", "ColBERT v3 with learned sparse retrieval"],
      "top_sources": ["papers", "github"],
      "dominant_tags": ["RAG", "retrieval", "benchmark"],
      "avg_preliminary_score": 7.1,
      "cluster_narrative": "1 sentence describing the theme this week"
    },
    "Agent frameworks and tooling": {
      "item_count": 6,
      "flagship_items": ["item_id_015"],
      "flagship_titles": ["AutoGen v0.4 — new async event-driven architecture"],
      "top_sources": ["github", "blogs"],
      "dominant_tags": ["agents", "framework", "tooling"],
      "avg_preliminary_score": 6.8,
      "cluster_narrative": "AutoGen's major redesign dominated agent discussions this week"
    }
  }
}
```

---

## Output

Return the full items array with these fields added to each item:
```json
{
  "cluster": "RAG pipeline improvements",
  "secondary_cluster": null,
  "cluster_fit": "strong | medium | weak",
  "is_flagship": false
}
```

Also return the `clusters_summary` object.

Include summary stats:
```json
{
  "total_clusters": 5,
  "cluster_names": ["RAG pipeline improvements", "Agent frameworks", "Multimodal advances", "Inference optimization", "Safety and evaluation"],
  "items_assigned": 57,
  "weak_fits": 3
}
```
