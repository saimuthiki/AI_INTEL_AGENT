# Skill: Theme Clusterer

**Purpose:** Group a collection of AI items into 4–6 named topic clusters that reflect the actual themes present that week — not a fixed taxonomy.

---

## 1. Core Principle: Emergent Themes Only

**Do NOT use a fixed list of categories.** The themes that emerge each week depend on what actually happened. Some weeks will be dominated by inference optimization. Others by safety announcements. Others by a wave of new agent frameworks.

The clusters must reflect the actual signal in the data, not a predetermined structure.

---

## 2. How to Identify Emergent Themes

### Step 1 — Read all items and their relevance tags
Scan all items collected from all 6 source agents. Note which tags appear most frequently. High-frequency tags are strong candidates for cluster labels.

### Step 2 — Look for narrative threads
Beyond tags, look for items that tell a connected story:
- Multiple items all responding to a single announcement → one cluster
- A series of papers on the same technique → one cluster
- Lab vs lab competition (e.g. OpenAI vs Anthropic both releasing agents) → potentially one "agent race" cluster

### Step 3 — Draft 4–8 candidate clusters
Write candidate cluster names and count how many items fit each. If a cluster has only 1–2 items, merge it with the nearest related cluster.

### Step 4 — Converge to 4–6 final clusters
- **Minimum 3 items per cluster** (merge smaller clusters)
- **Maximum 6 clusters** (split only if a cluster has 12+ items with clearly different themes)
- Every item must belong to exactly one cluster

---

## 3. Naming Clusters

### Good cluster name criteria
- **3–5 words** — specific enough to be meaningful
- **Descriptive, not generic** — "Long-context memory advances" beats "LLMs"
- **Action or development oriented** — captures what changed this week

### Examples of well-named clusters (from past weeks)
- "RAG pipeline improvements"
- "Multimodal reasoning models"
- "Open-source inference engines"
- "Agent tool use and planning"
- "AI safety and evaluation methods"
- "Code generation and developer tools"
- "Voice and real-time AI systems"
- "Fine-tuning on synthetic data"
- "Benchmark wars and model comparisons"
- "Enterprise AI deployment patterns"

### Examples of poorly-named clusters (avoid)
- "AI papers" — too generic
- "Other" — not allowed
- "Miscellaneous" — not allowed
- "LLMs" — too broad
- "Research" — describes format, not content

---

## 4. Assigning Items to Clusters

### Primary assignment rule
Assign each item to its **best-fit** cluster based on:
1. Its relevance tags (highest weight)
2. Its title and summary content
3. The cluster it would benefit most from being read alongside

### Handling multi-topic items
Some items fit multiple clusters (e.g. a paper on "multimodal RAG" fits both "RAG advances" and "Multimodal models"):
1. Assign to the **primary** cluster (the dominant topic)
2. Add a `secondary_cluster` field noting the secondary fit
3. Do NOT duplicate the item in two clusters

### Edge cases
- **If an item fits no cluster well**: Add it to the nearest cluster and note `"cluster_fit": "weak"`
- **If a high-score item doesn't fit any cluster**: Consider creating a new cluster for it if ≥ 3 other items share its theme
- **If a cluster would have only 1 item after assignment**: Merge that item into the nearest cluster

---

## 5. Identifying Flagship Items Per Cluster

For each cluster, identify 1–2 **flagship items** — the most important, highest-signal items in that cluster.

### Flagship selection criteria (in order of priority)
1. Highest relevance score in the cluster
2. Highest amplification score (covered by most sources)
3. Most actionable (has code, has tutorial, directly applicable)
4. Released by most authoritative source

### Flagship items in the output
Flagship items appear:
- In the cluster summary header
- Listed first within their cluster
- Referenced in the weekly intro paragraph
- Prioritized for sprint task assignment

---

## 6. Output Format

### Cluster summary object
```json
{
  "clusters_summary": {
    "RAG pipeline improvements": {
      "item_count": 8,
      "flagship_items": ["FlashRAG benchmark suite", "ColBERT v3 release"],
      "amplification": 4,
      "avg_relevance_score": 7.2
    },
    "Agent tool use and planning": {
      "item_count": 6,
      "flagship_items": ["AutoGen v0.4 release"],
      "amplification": 3,
      "avg_relevance_score": 6.8
    }
  }
}
```

### Per-item cluster field
Add to each item:
```json
{
  "cluster": "RAG pipeline improvements",
  "secondary_cluster": null,
  "cluster_fit": "strong",
  "is_flagship": true
}
```

---

## 7. Quality Checks

Before finalising clusters:
- [ ] 4–6 clusters total
- [ ] No cluster has fewer than 3 items
- [ ] Every item is assigned to exactly one cluster
- [ ] All cluster names are 3–5 words and descriptive
- [ ] Each cluster has 1–2 flagship items identified
- [ ] No "Other" or "Miscellaneous" cluster exists
