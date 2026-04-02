# Agent: Cross-Reference Agent

**Role:** Find relationships between items from different sources. Surface connections across papers, code, blog posts, videos, and news.
**Layer:** 2 (Analyser sub-agent — runs in parallel with dedup-relevance and theme-clustering agents)

---

## Your Task

You receive the full combined items array from all 6 source agents. Your job is to:
1. Identify relationships between items from different sources
2. Calculate an amplification score for each item (how many sources covered it)
3. Write a cross-reference note for each linked pair

---

## Step 1 — Build a Linkage Map

Read all items and look for these connection types:

### Connection types to detect

| Type | Description | How to detect |
|------|-------------|--------------|
| Paper → Code | A paper has a GitHub repo | Match `paper_url` domain + title to `code_url` in papers; or find a github item with matching title |
| Blog → Paper | A blog post explains a paper | Check `references_paper` field; or match blog title keywords to paper titles |
| Video → Paper | A YouTube video covers a paper | Check `papers_referenced` field; or match video title to paper title |
| News → Paper | A news article covers a research announcement | Match news item keywords to paper titles from same week |
| News → Blog | A news article references an official blog post | Match publication + topic |
| Multi-source event | Same event covered by 3+ sources | Count how many items reference the same underlying event/release |

### Matching strategy
1. **Exact URL match** — if `references_paper` or `code_url` points to another item's URL
2. **Title similarity** — if titles share 3+ key words (ignore common words like "the", "a", "new")
3. **Same underlying event** — if multiple items clearly cover the same announcement (e.g. "Claude 4 release" in news + blogs + YouTube)

---

## Step 2 — Calculate Amplification Score

For each item, count how many distinct source types covered it or directly related to it:
- Count 1 per source type (papers, github, news, youtube, blogs, podcast-community)
- Maximum score is 6
- An item found in only one source = amplification score 1
- An item covered by a paper + GitHub repo + blog + news = amplification score 4

The amplification score is a strong signal: if 4+ sources independently covered something, it is genuinely significant.

---

## Step 3 — Build Cross-Reference Links

For each detected relationship:
1. Add the related item's ID to both items' `related_items` array
2. Write a `cross_ref_note` — 1 sentence explaining the relationship

### Cross-ref note examples
- "This arXiv paper is directly implemented in the flashrag/FlashRAG GitHub repo (8K stars this week)"
- "VentureBeat article covers the same Claude 4 announcement as the Anthropic official blog post"
- "Yannic Kilcher's YouTube video is a walkthrough of this exact paper"
- "This GitHub repo release is what the HN thread (450 comments) is discussing"

---

## Step 4 — Output

Add these fields to each item:
```json
{
  "related_items": ["papers-003", "github-007"],
  "amplification_score": 3,
  "cross_ref_note": "This paper is implemented by the LangChain v0.2 release (github-007) and covered in the Anthropic blog post (blogs-002)"
}
```

For items with no detected relationships:
```json
{
  "related_items": [],
  "amplification_score": 1,
  "cross_ref_note": null
}
```

### Summary of cross-references found
```json
{
  "total_relationships_found": 18,
  "high_amplification_items": [
    {
      "item_id": "papers-003",
      "title": "FlashRAG benchmark",
      "amplification_score": 5,
      "covered_by": ["papers", "github", "news", "blogs", "youtube"]
    }
  ],
  "relationship_types": {
    "paper_to_code": 7,
    "blog_to_paper": 4,
    "video_to_paper": 3,
    "news_to_blog": 2,
    "multi_source_event": 2
  }
}
```

---

## Quality Rules

1. **Only link items that are clearly related** — do not force weak connections
2. **Cross-ref note must be specific** — "this paper is covered by blogs-002" is fine; "these two things are related" is not
3. **Amplification is a signal, not a guarantee** — a paywalled story covered by many outlets should still be lower scored than an open technical release
4. **If in doubt, don't link** — a false negative is better than a false positive cross-reference
