# Skill: Web Search

**Purpose:** Teach agents how to search the web effectively for recent AI content, handle paywalls, and extract clean structured data from results.

> **Escalation path:** Web search is the default tool. When it returns incomplete, stale, or paywalled results, escalate to `agent-browser` (see `skills/agent-browser.md`). The decision rule: if web search gives you what you need in one query, use it. If you'd need to click, scroll, login, or wait for JavaScript — use agent-browser.

---

## 0. Tool Selection Decision Tree

```
Do you need content from this URL?
├── Is it a static, public, indexable page? (arXiv, GitHub README, news article)
│   └── YES → Use web search first
│       └── Did you get the full content you need?
│           ├── YES → Done ✓
│           └── NO → Escalate to agent-browser (snapshot + extract)
│
├── Is it JavaScript-rendered? (GitHub Trending, Reddit, HN, HuggingFace feed)
│   └── YES → Use agent-browser directly (skip web search for these)
│
├── Is it behind a paywall?
│   └── YES → Web search for the headline/excerpt, then agent-browser to check
│             what's visible to unauthenticated users
│
└── Does it require interaction? (clicking "Load more", filling forms, login)
    └── YES → agent-browser only
```

**Sources where agent-browser is PREFERRED over web search:**
- GitHub Trending (JS-rendered)
- HuggingFace Papers/Models feed (JS-rendered)
- Reddit new interface → use `old.reddit.com` via agent-browser
- Substack newsletters (JS + modal)
- YouTube video pages (description, chapters, view counts)
- OpenReview submissions (JS-rendered pagination)
- Product Hunt (JS-rendered cards)

---

## 1. Query Formulation for Recent Content

### Date-bounding your searches
Always bound searches to the past 7 days. Use these patterns:
```
site:arxiv.org after:2025-04-01 "transformer" OR "LLM" OR "RAG"
site:github.com trending AI machine-learning created:>2025-04-01
"last week" OR "this week" OR "released" site:huggingface.co
```

### High-signal query templates by source type

**Papers (arXiv / HuggingFace):**
```
arxiv.org cs.AI cs.LG cs.CL [topic] 2025
huggingface.co/papers [topic] week
"submitted to arXiv" [topic] April 2025
```

**GitHub repositories:**
```
site:github.com [topic] stars:>100 pushed:>2025-04-01
github trending [AI/ML topic] this week
[library name] release changelog 2025
```

**News:**
```
[topic] AI announcement April 2025 site:techcrunch.com OR site:theverge.com
[lab name] release blog 2025
"new model" OR "new release" [topic] this week
```

**Blogs and newsletters:**
```
site:lilianweng.github.io OR site:jalammar.github.io 2025
"last week in AI" OR "import AI" OR "the batch" 2025
[author name] blog post [topic] April 2025
```

---

## 2. Filtering Results to Past 7 Days

### Priority signals for recency
- Explicit date in URL or headline (e.g. `/2025/04/`, `April 2025`)
- "Released", "Launched", "Announced", "New", "Introducing" in title
- "This week", "Last week", "Today" in content
- ISO date format in JSON-LD metadata

### When to include borderline-dated items
- If the item is highly relevant (score ≥ 8) and < 14 days old, include it with a note
- If the item is a major release (e.g. GPT-5), include regardless of exact date
- If date is ambiguous, note it as "date uncertain" and include if relevant

---

## 3. Extracting Clean Content from Results

### What to extract per result
1. **Title** — exact, unmodified
2. **URL** — canonical link, not redirect
3. **Date** — ISO format (`YYYY-MM-DD`), or "unknown" if not found
4. **Summary** — 2–4 sentences synthesized from the content, not copy-pasted
5. **Key contribution / why it matters** — 1 sentence
6. **Relevance tags** — 3–5 tags from the standard tag list below

### Standard relevance tags
Use these consistently across all agents:
`RAG`, `LLM`, `multimodal`, `agents`, `fine-tuning`, `inference`, `quantization`, `vision`, `audio`, `code-gen`, `reasoning`, `memory`, `safety`, `alignment`, `benchmark`, `tooling`, `deployment`, `evaluation`, `embedding`, `retrieval`, `training`, `architecture`, `open-source`, `API`, `product-launch`

---

## 4. Handling Paywalled Content

### Detection signals
- "Subscribe to read", "Sign in", "Member only", "Paywall" in response
- HTTP 402 or redirect to login page
- Partial content with "Continue reading" cut-off

### What to do with paywalled content
1. Extract everything visible in the preview/excerpt
2. Check if the same story is covered by a non-paywalled source (cross-reference)
3. Note in output: `"paywalled": true`
4. Apply `-1` to relevance score
5. Still include if the item is highly significant — summarise from the headline and any visible excerpt

### Sources that are frequently paywalled
- The Information (always paywalled — use headline only)
- Bloomberg Technology (often paywalled — look for Reuters/AP coverage)
- Fortune Tech (often paywalled)
- MIT Technology Review (partial paywall)

---

## 5. Rate Limiting and Retry Strategy

### Request spacing
- Space web searches at least 2 seconds apart when hitting the same domain
- For high-volume runs (6 source agents in parallel), stagger domain hits
- Maximum 3 requests per domain per run to avoid being blocked

### Retry logic
On failure (timeout, 429, 503):
1. Wait 5 seconds
2. Retry once with a simplified query
3. If still failing, skip the source and log: `"source_status": "unavailable"` in output
4. Do NOT crash the entire run — partial results are better than no results

### Signs of rate limiting
- Empty results for normally-active sources
- Identical results returned for different queries
- Unusually slow response times

---

## 6. Quality Checks Before Outputting

Before returning results, verify each item:
- [ ] Has a valid URL (not a redirect, not a 404)
- [ ] Date is within the target window (or flagged as outside)
- [ ] Summary is factual — do not hallucinate content not found in the source
- [ ] Tags are from the standard list
- [ ] Relevance score is justified by the scoring rubric in `skills/relevance-scorer.md`

If you cannot verify a URL is real, omit the item rather than include a potentially hallucinated link.
