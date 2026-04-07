# Skill: Agent Browser

**Purpose:** Teach agents how to use `agent-browser` — a native Rust CLI for full browser automation — when web search is insufficient. Use this for dynamic/JavaScript-rendered content, paywalled sources, authenticated sessions, and multi-step extraction workflows.

**Tool:** [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) — installed via `npm install -g agent-browser`

---

## 1. When to Use agent-browser vs Web Search

Use the **escalation ladder** below. Start at Level 1, escalate only when needed.

| Level | Tool | When to use |
|-------|------|-------------|
| **1** | Web search (default) | Public, static, indexable content — arXiv, GitHub README, news headlines |
| **2** | `agent-browser` snapshot | JavaScript-rendered pages, content not in search index, full article text |
| **3** | `agent-browser` interact | Paywall navigation, form-based search, paginated results requiring clicks |
| **4** | `agent-browser` + auth | Login-required content, session-persisted scraping |

**Escalate to agent-browser when:**
- Web search returns a snippet but not the full content you need
- The page is JavaScript-rendered (GitHub Trending, Reddit, HN, Substack)
- The source is paywalled and you need to check what's visible to a free user
- You need to click "Load more" or paginate through results
- You need to extract data from a table or structured UI element
- Content requires a login (use saved auth state if available)

**Do NOT use agent-browser when:**
- A direct web search returns the full content you need — don't over-engineer
- The URL is a direct PDF or API endpoint — fetch directly
- The site is known to block automation (some news sites with strict bot detection)

---

## 2. Setup and Installation

### Install
```bash
npm install -g agent-browser
agent-browser install        # downloads Chrome for Testing (~170 MB, one-time)
```

### Verify
```bash
agent-browser --version      # should print v0.25.x or later
```

### First run test
```bash
agent-browser open https://example.com
agent-browser snapshot -i    # should return element refs like @e1, @e2
```

---

## 3. Core Workflow — The Snapshot+Refs Pattern

Every agent-browser session follows this pattern:

```bash
# Step 1: Navigate
agent-browser open https://target-site.com

# Step 2: Snapshot — get interactive elements as lightweight refs
agent-browser snapshot -i          # interactive elements only (less tokens)
# OR
agent-browser snapshot             # full accessibility tree (more detail)
# OR
agent-browser snapshot -c          # compact mode (fewest tokens)

# Step 3: Interact using refs from snapshot output (@e1, @e2, etc.)
agent-browser click @e3
agent-browser fill @e5 "search query"
agent-browser press Enter

# Step 4: Re-snapshot after page changes (refs are invalidated)
agent-browser snapshot -i

# Step 5: Extract what you need
agent-browser get text @e8
agent-browser screenshot           # visual confirmation
```

**Critical rule:** Element refs (`@e1`, `@e2`) are **invalidated after any navigation or DOM change**. Always re-snapshot after clicking a link, submitting a form, or waiting for dynamic content to load.

---

## 4. Recipes for Each Source Agent

### 4.1 GitHub Trending (github-agent)

GitHub Trending is JavaScript-rendered — web search often returns stale or empty results. Use agent-browser:

```bash
agent-browser open "https://github.com/trending/python?since=weekly"
agent-browser snapshot -c          # compact snapshot of the page

# Extract repo names and descriptions
agent-browser get text @e1         # repo name
agent-browser get attr @e1 href    # repo URL

# For star counts (usually in a specific element)
agent-browser snapshot --urls      # get all links on page
```

**What to extract per repo:**
- Repository name (owner/repo)
- Description text
- Star count (look for `★` or star icon adjacent element)
- Language
- Stars gained this week (usually shown as "+ N this week")

### 4.2 HuggingFace Papers & Models (papers-agent)

HuggingFace Papers feed is partially JS-rendered:

```bash
agent-browser open "https://huggingface.co/papers"
agent-browser wait 2000            # wait for JS to load
agent-browser snapshot -i

# For model pages
agent-browser open "https://huggingface.co/models?sort=downloads&direction=-1&limit=20"
agent-browser wait 2000
agent-browser snapshot -c
```

### 4.3 Paywalled News Articles (news-agent)

For sites like MIT Technology Review, Bloomberg, Fortune Tech:

```bash
# Step 1: Open the article
agent-browser open "https://www.technologyreview.com/article-url"
agent-browser wait 2000
agent-browser snapshot -i

# Step 2: Check what's visible to an unauthenticated user
agent-browser get text @e1         # Try to get article body

# Step 3: If paywalled, get whatever is visible (usually first 2-3 paragraphs)
# Note "paywalled: true" in your output and apply -1 to relevance score
agent-browser screenshot           # screenshot shows actual visible content
```

**Paywall detection signals:**
- Snapshot contains elements with text "Subscribe", "Sign in to read", "Member only"
- Article body element contains < 200 characters
- A modal/overlay is present in the snapshot

### 4.4 Reddit (podcast-community-agent)

Reddit's new interface is fully JS-rendered:

```bash
# Get top posts from r/MachineLearning this week
agent-browser open "https://www.reddit.com/r/MachineLearning/top/?t=week"
agent-browser wait 3000            # Reddit needs time to load
agent-browser snapshot -c

# For each top post, open and extract comments count, upvotes, title
agent-browser get text @e2         # post title
agent-browser get text @e4         # upvote count
agent-browser get text @e5         # comment count
```

**Alternative for Reddit (lower friction):**
Use old Reddit which is static HTML and works better with web search:
```bash
agent-browser open "https://old.reddit.com/r/MachineLearning/top/?t=week"
agent-browser snapshot -c          # cleaner output, no JS required
```

### 4.5 Hacker News (podcast-community-agent)

HN is static HTML — web search usually works. Use agent-browser only for comment extraction:

```bash
agent-browser open "https://news.ycombinator.com"
agent-browser snapshot -c          # clean output — HN is simple HTML

# For a specific thread
agent-browser open "https://news.ycombinator.com/item?id=12345"
agent-browser snapshot -c
agent-browser get text @e1         # extract top comments
```

### 4.6 Substack Newsletters (blogs-agent)

Substack posts are JS-rendered and may require email-based auth:

```bash
# Free posts
agent-browser open "https://authorname.substack.com"
agent-browser wait 2000
agent-browser snapshot -c

# Find latest post link
agent-browser snapshot --urls      # extracts all links — find latest post URL
agent-browser open "<post-url>"
agent-browser wait 2000
agent-browser get text @e1         # article body
```

### 4.7 YouTube Video Pages (youtube-agent)

For getting video descriptions and metadata when search results are incomplete:

```bash
agent-browser open "https://www.youtube.com/watch?v=VIDEO_ID"
agent-browser wait 3000            # YouTube is heavy JS

# Get video description (often collapsed — need to click "more")
agent-browser snapshot -i
agent-browser click @e_more        # click "Show more" description button
agent-browser snapshot -c
agent-browser get text @e_desc     # get expanded description
```

---

## 5. Multi-Step Extraction Workflows

### Batch mode for efficiency

For sequential commands with no branching, use batch mode (single daemon roundtrip):

```bash
agent-browser batch << 'EOF'
open https://github.com/trending/python?since=weekly
wait 2000
snapshot -c
EOF
```

### Extracting paginated results

```bash
# Page 1
agent-browser open "https://site.com/search?q=llm&page=1"
agent-browser snapshot -c
agent-browser get text @results

# Page 2 — find and click "Next" or construct URL directly
agent-browser open "https://site.com/search?q=llm&page=2"
agent-browser snapshot -c
agent-browser get text @results
```

### Session persistence for multi-site runs

When scraping multiple sites in one pipeline run, use named sessions to avoid re-launching Chrome:

```bash
# All commands in the same session reuse the same browser instance
agent-browser --session weekly-run open https://site1.com
agent-browser --session weekly-run snapshot -c
agent-browser --session weekly-run open https://site2.com
agent-browser --session weekly-run snapshot -c
```

---

## 6. Authentication Workflows

### Save auth state for sources that require login
```bash
# One-time setup (run manually, not in automated pipeline)
agent-browser --headed open https://site-requiring-login.com
# (browser opens visibly — log in manually in the browser window)
agent-browser state save skills/auth-states/sitename.json
```

### Use saved auth state in automated runs
```bash
agent-browser state load skills/auth-states/sitename.json
agent-browser open https://site-requiring-login.com
# You are now logged in
agent-browser snapshot -c
```

**Auth state files to consider saving (one-time manual setup):**
- `auth-states/reddit.json` — Reddit (for higher rate limits and better content access)
- `auth-states/substack.json` — Substack (for paid newsletter access if you subscribe)

**Never commit auth state files to git.** Add to `.gitignore`:
```
skills/auth-states/
agent-browser.json
```

---

## 7. Output Handling and Token Efficiency

### Snapshot output format

`agent-browser snapshot -i` returns an accessibility tree fragment like:
```
[1] button "Sign in" @e1
[2] link "Getting Started" @e2 href="/docs"
[3] heading "Latest Papers" @e3
[4] link "FlashRAG: Modular Toolkit" @e4 href="/papers/flashrag"
[5] text "Trending this week" @e5
```

Each `@eN` ref can be used in subsequent commands.

### Token efficiency comparison

| Snapshot mode | Approx tokens | When to use |
|---------------|--------------|-------------|
| `snapshot -i` | 200–400 | When you need to interact (click/fill) |
| `snapshot -c` | 100–200 | When you only need to read/extract |
| `snapshot` | 500–1500 | When you need full accessibility tree |
| `snapshot --urls` | 50–100 | When you only need links |

Use the lightest mode that gives you what you need.

---

## 8. Error Handling and Fallbacks

### Common errors and fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `timeout after 25s` | Page too slow to load | Use `agent-browser wait 5000` before snapshot; or increase timeout with `AGENT_BROWSER_DEFAULT_TIMEOUT=45000` |
| `@e1 not found` | Ref invalidated after DOM change | Re-snapshot before using ref |
| `Chrome not found` | Browser not installed | Run `agent-browser install` |
| Empty snapshot | JS not finished loading | Add `agent-browser wait 3000` or `agent-browser wait --load networkidle` (avoid on ad-heavy sites) |
| `Connection refused` | Daemon not running | Daemon auto-starts on first command — retry once |

### When agent-browser itself fails

If agent-browser cannot access a source:
1. Log `"agent_browser_used": true, "agent_browser_status": "failed"` in item metadata
2. Fall back to web search for that source
3. Mark the result with `"content_confidence": "low"` if web search also gives partial data
4. Continue — never abort the whole pipeline run for a single source failure

---

## 9. Integration Notes for AI Intel Agent Pipeline

### Where agent-browser adds the most value

| Source agent | Default tool | Use agent-browser when... |
|-------------|-------------|--------------------------|
| papers-agent | Web search | HuggingFace Papers JS feed; OpenReview submissions |
| github-agent | Web search | GitHub Trending (JS-rendered); HF Models feed |
| news-agent | Web search | Paywall navigation; JS-rendered article pages |
| youtube-agent | Web search | Getting full description text; chapter markers |
| blogs-agent | Web search | Substack posts; Medium behind soft-login |
| podcast-community-agent | Web search | Reddit new interface; HN thread extraction |

### Pipeline position

agent-browser is called **within individual source agents** as a fallback or enhancement — not as a separate pipeline stage. Each source agent decides when to escalate from web search to agent-browser based on the escalation ladder in Section 1.

### Performance impact

Adding agent-browser calls adds ~3–10 seconds per page (Chrome startup + navigation + snapshot). Since source agents run in parallel, this only adds time within a single agent's execution, not to the overall pipeline wall-clock time.

Use agent-browser **selectively** — not for every URL. The web search fallback is fast enough for most public content.
