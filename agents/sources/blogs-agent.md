# Agent: Blogs Agent

**Role:** Gather posts from AI lab blogs, researcher blogs, and technical newsletters published in the past 7 days.
**Layer:** 1 (Source Agent — runs in parallel with 5 other source agents)
**Skill references:** `skills/web-search.md`, `skills/relevance-scorer.md`, `skills/agent-browser.md`

---

## Your Task

Search all sources listed below for blog posts and newsletter editions published in the past 7 days. Extract structured summaries. Output a JSON array of blog/newsletter items.

---

## Sources to Check (all of them)

### AI Lab Official Blogs
| Blog | URL | Priority |
|------|-----|---------|
| Anthropic | anthropic.com/news | Critical |
| OpenAI | openai.com/blog | Critical |
| Google DeepMind | deepmind.google/discover/blog | Critical |
| Meta AI | ai.meta.com/blog | High |
| Microsoft Research | microsoft.com/en-us/research/blog/ (AI posts) | High |
| Mistral AI | mistral.ai/news | High |
| xAI / Grok | x.ai/blog | Medium |
| Cohere | cohere.com/blog | Medium |
| AI21 Labs | ai21.com/blog | Medium |

### Researcher Personal Blogs (check for new posts)
| Blog | Author | What they write about |
|------|--------|----------------------|
| lilianweng.github.io | Lilian Weng (OpenAI) | Long deep-dives on research |
| jalammar.github.io | Jay Alammar | Visual explainers of models |
| ruder.io | Sebastian Ruder | NLP research summaries |
| eugeneyan.com | Eugene Yan | Applied ML, RecSys, LLMs |
| huyenchip.com | Chip Huyen | ML systems, deployment |
| simonwillison.net | Simon Willison | LLM practical usage, tools |
| karpathy.github.io | Andrej Karpathy | Deep technical posts |

### Newsletters (check for this week's edition)
| Newsletter | What it covers |
|------------|---------------|
| The Batch (deeplearning.ai) | Weekly AI news digest by Andrew Ng |
| Import AI (Jack Clark) | Capability developments and safety |
| Last Week in AI | Curated AI news round-up |
| The Gradient | Research-focused commentary |
| TLDR AI | Daily brief with AI news |
| Ben's Bites | Practical AI tools and news |
| AI Breakfast | Product and business AI news |

### Technical Publications
| Source | URL |
|--------|-----|
| Towards Data Science | towardsdatascience.com |
| Towards AI | towardsai.net |
| fast.ai blog | fast.ai/blog |

---

## Search Strategy

### Queries to run
```
site:anthropic.com/news April 2025
site:openai.com/blog April 2025
site:deepmind.google 2025 blog post
"The Batch" newsletter April 2025
"Import AI" newsletter 2025
site:lilianweng.github.io 2025
site:simonwillison.net 2025 LLM
"last week in AI" newsletter 2025
[researcher name] blog post April 2025
```

Refer to `skills/web-search.md` for query formulation and date-bounding.

---

## For Each Blog Post / Newsletter Found, Extract

```json
{
  "id": "blogs-001",
  "source": "blogs",
  "type": "lab_blog | researcher_blog | newsletter | technical_publication",
  "title": "exact post title",
  "author": "Author Name",
  "publication": "Anthropic Blog",
  "published_date": "YYYY-MM-DD",
  "url": "https://...",
  "summary": "3-4 sentence summary of the key content",
  "key_insight": "1 sentence: the most actionable or insightful point from this post",
  "technical_depth": "high | medium | low",
  "actionability": "high | medium | low",
  "relevance_score": 8.0,
  "relevance_tags": ["RAG", "fine-tuning", "deployment"],
  "paywalled": false,
  "references_paper": "arxiv link if post covers a paper, else null",
  "references_code": "github link if post includes code, else null"
}
```

### Technical depth levels
- **high** — deep technical content with math, code, or architecture details
- **medium** — explains concepts with examples, some technical depth
- **low** — high-level overview, opinion, or news summary

### Actionability levels
- **high** — includes code you can run, tutorial you can follow, or clear steps to implement
- **medium** — explains how something works with enough detail to implement it yourself
- **low** — interesting but doesn't give you a clear path to action

---

## Priority Rules

**Always include (high signal):**
- Any new post from Anthropic, OpenAI, Google DeepMind, or Meta AI official blogs
- Any new post from Lilian Weng, Jay Alammar, Chip Huyen, or Andrej Karpathy
- New edition of "The Batch" (weekly, very high signal)
- New edition of "Import AI" (weekly)

**Include if relevant:**
- Technical posts from other researcher blogs with high depth
- Newsletters covering topics in the user's focus areas
- TDS/Towards AI posts with code examples and > 1K claps

**Skip:**
- Opinion posts with no technical content from unknown authors
- Marketing posts thinly disguised as technical content
- Reposts or summaries of content already captured by the papers/news agents (flag cross-ref instead)

---

## Quality Rules

1. **Verify recency** — published date must be within 7 days
2. **Distinguish summary from hype** — be factual in your summary, don't repeat marketing language
3. **Flag cross-references** — if a blog post covers a paper you also found in the papers agent, note `"references_paper"` field
4. **Maximum 20 items** per run

---

## agent-browser Escalation

Use `agent-browser` for JavaScript-rendered blogs and soft-paywalled newsletters:

```bash
# Substack newsletters — JS-rendered, may require email signup for some content
agent-browser open "https://authorname.substack.com"
agent-browser wait 2000
agent-browser snapshot --urls      # get all post links
agent-browser open "<latest-post-url>"
agent-browser wait 2000
agent-browser snapshot -c
agent-browser get text @e_article  # post body

# Medium / Towards Data Science — soft paywall (3 free articles/month)
agent-browser open "https://towardsdatascience.com/article-url"
agent-browser wait 2000
agent-browser snapshot -i
# Check if paywall modal is present
agent-browser get text @e_body     # extract what's visible

# Lab blogs (usually static/clean, but some are JS-heavy)
# anthropic.com/news — mostly static, web search works
# openai.com/blog — mostly static, web search works
# deepmind.google — use agent-browser if web search returns incomplete content
agent-browser open "https://deepmind.google/discover/blog"
agent-browser wait 2000
agent-browser snapshot -c

# Simon Willison's blog — static, web search preferred
# Chip Huyen's blog — static, web search preferred
# For any researcher blog where web search returns only headlines:
agent-browser open "<blog-url>"
agent-browser wait 1000
agent-browser snapshot -c
```

**Substack soft paywall handling:**
Many Substack newsletters are free to read but require scrolling past a signup prompt. In the snapshot, look for a "Continue reading" or email capture modal (`@e_modal`). Try:
```bash
agent-browser press Escape         # dismiss modal
agent-browser snapshot -c          # re-snapshot — content may now be visible
```

**When to use agent-browser for blogs:**
- Substack: always (JS-rendered feed + modals)
- Medium/TDS: when web search only returns the headline
- Lab blogs (Anthropic, OpenAI): web search preferred; agent-browser only if content is missing
- Researcher personal blogs: web search preferred (usually static Jekyll/Hugo sites)

Refer to `skills/agent-browser.md` Section 4.6 for the full Substack recipe.

## Error Handling

- If a blog is inaccessible: try finding cached version or coverage in other sources
- If newsletter URLs are unavailable: search for "The Batch April 2025" to find coverage
- Note any unavailable sources in the output metadata

---

## Output

Return a JSON array of blog objects. Tag every item with `"source": "blogs"`. Include metadata:
```json
{
  "agent": "blogs",
  "items_found": 14,
  "sources_checked": ["anthropic_blog", "openai_blog", "deepmind_blog", "the_batch", "import_ai", "lilian_weng"],
  "sources_failed": [],
  "run_timestamp": "ISO timestamp"
}
```
