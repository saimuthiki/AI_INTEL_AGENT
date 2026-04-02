# AI Intel Agent — Complete Build Blueprint for Claude Code

> **How to use this file:** Paste this entire document into Claude Code and say:
> *"Build this project exactly as described. Create every file, every agent, every skill. Follow the project structure precisely."*

---

## 1. What This System Does

**AI Intel Agent** is a fully automated weekly research intelligence system for developer teams. It works like a personal research army — every week it scans the entire AI landscape across papers, GitHub, news, YouTube, blogs, podcasts, and communities, analyses everything, clusters it by theme, scores each item by impact, synthesises actionable insights, and produces a ready-to-execute sprint plan assigned to however many people are available that week.

The end user triggers it with a single natural language message:

```
"Hey research agent — 4 people are free this week, focus on RAG and agent frameworks"
```

The system returns:
1. A **weekly brief** — a full digest of everything that happened in AI that week, with TL;DRs and source links
2. A **sprint board** — N tasks assigned to N people, each tied to a specific finding, with rationale and effort estimate
3. A **tracker update** — the weekly log updated with new items, previous week carry-overs surfaced

There is no config file, no team profiles JSON, no setup needed. Team size and focus areas are parsed entirely from the user's natural language trigger each week.

---

## 2. Project Structure — Build This Exactly

```
ai-intel-agent/
│
├── CLAUDE.md                          ← Master instructions (Claude Code reads this first)
│
├── .claude/
│   └── commands/
│       ├── run-weekly.md              ← /run-weekly slash command
│       └── check-tracker.md          ← /check-tracker slash command
│
├── agents/
│   │
│   ├── orchestrator.md               ← Root agent — entry point for all runs
│   │
│   ├── sources/                      ← Layer 1: 6 parallel scraper agents
│   │   ├── papers-agent.md
│   │   ├── github-agent.md
│   │   ├── news-agent.md
│   │   ├── youtube-agent.md
│   │   ├── blogs-agent.md
│   │   └── podcast-community-agent.md
│   │
│   ├── analyser/                     ← Layer 2: analyser shell + 3 parallel sub-agents
│   │   ├── analyser-orchestrator.md
│   │   ├── dedup-relevance-agent.md
│   │   ├── theme-clustering-agent.md
│   │   └── cross-ref-agent.md
│   │
│   ├── insight-agent.md              ← Layer 3: maps findings to team actions
│   ├── sprint-planner-agent.md       ← Layer 4: generates N tasks for N people
│   └── tracker-agent.md             ← Layer 5: persists and diffs weekly log
│
├── skills/
│   ├── web-search.md                 ← How to search the web effectively
│   ├── youtube-transcript.md         ← How to extract YouTube content
│   ├── pdf-extractor.md              ← How to read research paper PDFs
│   ├── relevance-scorer.md           ← How to score items 1–10 by impact
│   ├── theme-clusterer.md            ← How to group items into topic clusters
│   ├── task-formatter.md             ← How to format sprint tasks
│   └── tracker-schema.md            ← The JSON schema for weekly-log.json
│
├── tracker/
│   └── weekly-log.json               ← Persisted week-by-week sprint history (starts empty)
│
├── output/                           ← One markdown file generated per run
│   └── .gitkeep
│
└── samples/
    └── sample-brief-week01.md        ← Example output for agents to follow as reference
```

---

## 3. Agent Architecture — Full Pipeline

The system has **5 layers**. Layers 1 and 2 run agents in parallel. Layers 3–5 run sequentially.

```
USER TRIGGER (natural language)
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                   ORCHESTRATOR AGENT                     │
│  Parses: headcount, names (if given), focus topics       │
│  Fans out Layer 1 agents all at once in parallel         │
└─────────────────────────────────────────────────────────┘
        │ (parallel fan-out)
        ├──────────┬──────────┬──────────┬──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼          ▼
  [Papers]   [GitHub]    [News]    [YouTube]   [Blogs]  [Podcast+
   agent      agent      agent      agent      agent   Community]
                                                         agent
        │          │          │          │          │          │
        └──────────┴──────────┴──────────┴──────────┴──────────┘
                              │ (all results collected)
                              ▼
┌─────────────────────────────────────────────────────────┐
│              ANALYSER ORCHESTRATOR                       │
│  Receives raw items from all 6 source agents             │
│  Fans out 3 sub-agents in parallel                       │
│                                                          │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │
│   │Dedup+Relevance│ │   Theming    │ │  Cross-ref   │   │
│   │    agent      │ │   agent      │ │    agent     │   │
│   │               │ │              │ │              │   │
│   │ Remove dupes  │ │ Group into   │ │ Link items   │   │
│   │ Score 1–10    │ │ 4–6 clusters │ │ across srcs  │   │
│   └──────────────┘ └──────────────┘ └──────────────┘   │
│              │              │              │             │
│              └──────────────┴──────────────┘            │
│                    Merged scored+clustered+linked        │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│                   INSIGHT AGENT                          │
│  Reads team headcount + focus from original trigger      │
│  Translates top-scored items into "what we should do"   │
│  Estimates hours of effort per action item               │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│                SPRINT PLANNER AGENT                      │
│  Generates exactly N tasks for N people                  │
│  Assigns tasks by inferred skill or topic diversity      │
│  Writes rationale + source link for each task            │
└─────────────────────────────────────────────────────────┘
                    │           │           │
                    ▼           ▼           ▼
             [Weekly Brief] [Sprint Board] [Tracker Delta]
                    │           │           │
                    └─────┬─────┴─────┬─────┘
                          ▼           ▼
┌─────────────────────────────────────────────────────────┐
│                   TRACKER AGENT                          │
│  Appends new week to weekly-log.json                     │
│  Diffs vs previous week — flags carry-overs              │
│  Updates item statuses (todo/in-progress/done/carry-over)│
│  Writes output/YYYY-WW-brief.md                          │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
                    DELIVERED TO USER
           (markdown file + console summary)
```

---

## 4. Every Agent — Detailed Spec

### 4.1 Orchestrator Agent (`agents/orchestrator.md`)

**Role:** Root entry point. Receives the user's natural language trigger. Parses it. Fans out all 6 source agents simultaneously. Waits for all to complete. Passes combined results to the analyser.

**Responsibilities:**
- Parse team headcount from trigger (e.g. "4 people", "3 devs", "just me and a colleague")
- Parse any named team members and their roles/skills if mentioned
- Parse any focus areas mentioned (e.g. "focus on RAG", "multimodal stuff", "agent frameworks")
- If no focus is mentioned, default to broad AI coverage
- Spawn all 6 source agents in parallel — do NOT wait for one before starting the next
- Collect all 6 result sets and pass them as a single combined payload to the analyser orchestrator
- Pass the original parsed team info alongside so downstream agents have it

**Input:** Raw user trigger string
**Output:** Combined raw items array (tagged by source) + parsed team context object

---

### 4.2 Papers Agent (`agents/sources/papers-agent.md`)

**Role:** Scrape and summarise AI research papers published in the past 7 days.

**Sources to check (all of them, every run):**
- arXiv (cs.AI, cs.LG, cs.CL, cs.CV, cs.NE categories)
- HuggingFace Papers (huggingface.co/papers)
- Semantic Scholar (recent high-citation papers)
- Papers With Code (trending papers with code)
- ACL Anthology (NLP papers)
- NeurIPS / ICML / ICLR proceedings (new accepted papers)
- OpenReview (new submissions and reviews)
- bioRxiv (AI + biology intersection papers)

**For each paper found, extract:**
- Title
- Authors
- Abstract (2–3 sentence summary)
- Key contribution (1 sentence: what is new about this)
- Link to paper
- Link to code repo if available
- Number of citations or engagement signal if available
- Relevance tags (e.g. RAG, LLM, multimodal, agents, fine-tuning, inference, etc.)

**Output format:** JSON array of paper objects with the above fields + `source: "papers"`

---

### 4.3 GitHub Agent (`agents/sources/github-agent.md`)

**Role:** Find new and trending AI repositories, model releases, and library updates from the past 7 days.

**Sources to check:**
- GitHub Trending (today, this week — filter AI/ML repos)
- GitHub Releases (for major AI libraries: LangChain, LlamaIndex, AutoGen, CrewAI, Transformers, vLLM, Ollama, LiteLLM, DSPy, Instructor, Pydantic-AI, Haystack)
- HuggingFace Models (newly uploaded models with high downloads this week)
- HuggingFace Spaces (trending new demo spaces)
- GitLab trending (AI projects)
- PyPI new releases (AI/ML packages with significant download spikes)
- npm new releases (AI-related JS/TS packages)
- LangChain changelog / release notes

**For each repo/release found, extract:**
- Repo name and owner
- Description (1–2 sentences)
- What changed or why it is notable
- GitHub stars and star velocity (stars gained this week)
- Link
- Relevance tags

**Output format:** JSON array with above fields + `source: "github"`

---

### 4.4 News Agent (`agents/sources/news-agent.md`)

**Role:** Gather AI industry news, announcements, and events from the past 7 days.

**Sources to check:**
- TechCrunch AI section
- The Verge AI section
- VentureBeat AI
- Wired (AI articles)
- MIT Technology Review
- Bloomberg Technology
- Reuters Technology
- The Information (AI articles, if accessible)
- X / Twitter (curated AI lists — top posts from key AI researchers and lab accounts)
- Protocol (if still active)
- Ars Technica (AI articles)
- Fortune Tech

**For each news item found, extract:**
- Headline
- Publication and date
- 2–3 sentence summary
- Why it matters (1 sentence)
- Link
- Relevance tags
- Sentiment: positive / neutral / negative / controversial

**Output format:** JSON array with above fields + `source: "news"`

---

### 4.5 YouTube Agent (`agents/sources/youtube-agent.md`)

**Role:** Find and summarise AI-focused YouTube videos published in the past 7 days.

**Channels to check:**
- Andrej Karpathy
- Yannic Kilcher
- Two Minute Papers
- AI Explained
- Lex Fridman (AI episodes only)
- OpenAI official channel
- Anthropic official channel
- Google DeepMind official channel
- Meta AI official channel
- Sentdex
- ML Street Talk
- Aleksa Gordic (The AI Epiphany)
- AssemblyAI
- Weights & Biases (Fully Connected)
- Stanford Online (AI lectures)

**For each video found, extract:**
- Title
- Channel
- Published date
- View count
- Duration
- Summary of content (3–5 sentences using transcript if available, title+description if not)
- Key takeaway (1 sentence)
- Link
- Relevance tags

**Output format:** JSON array with above fields + `source: "youtube"`

---

### 4.6 Blogs Agent (`agents/sources/blogs-agent.md`)

**Role:** Gather posts from AI lab blogs, researcher blogs, and technical newsletters published in the past 7 days.

**Sources to check:**
- Anthropic blog (anthropic.com/news)
- OpenAI blog (openai.com/blog)
- Google DeepMind blog
- Meta AI blog
- Microsoft Research blog (AI posts)
- fast.ai blog
- The Batch by deeplearning.ai
- Lilian Weng's blog (lilianweng.github.io)
- Jay Alammar's blog (jalammar.github.io)
- Sebastian Ruder's blog
- Eugene Yan's blog
- Chip Huyen's blog
- Simon Willison's blog (simonwillison.net)
- Towards Data Science (top AI posts)
- Towards AI
- The Gradient
- Import AI newsletter (Jack Clark)
- Last Week in AI newsletter

**For each post found, extract:**
- Title
- Author and publication
- Date
- 3–4 sentence summary
- Key insight or finding (1 sentence)
- Link
- Relevance tags

**Output format:** JSON array with above fields + `source: "blogs"`

---

### 4.7 Podcast and Community Agent (`agents/sources/podcast-community-agent.md`)

**Role:** Surface notable discussions, episodes, and community signals from AI podcasts and online communities in the past 7 days.

**Sources to check:**
- TWIML AI Podcast (This Week in Machine Learning)
- Practical AI Podcast
- Latent Space podcast
- Lex Fridman podcast (audio episodes — if not caught by YouTube agent)
- Gradient Dissent (Weights & Biases podcast)
- Reddit r/MachineLearning (top posts this week)
- Reddit r/LocalLLaMA (top posts this week)
- Reddit r/artificial (top posts this week)
- Hacker News (AI-related threads with 50+ comments)
- Discord AI servers (notable announcements — EleutherAI, Midjourney, LangChain)
- Substack AI newsletters (new editions)
- LinkedIn (posts from notable AI figures with high engagement)
- Product Hunt (new AI products launched this week)

**For each item found, extract:**
- Title or topic
- Source platform
- Date
- 2–3 sentence summary of what was discussed or announced
- Community sentiment / reaction (1 sentence)
- Link
- Relevance tags

**Output format:** JSON array with above fields + `source: "podcast-community"`

---

### 4.8 Analyser Orchestrator (`agents/analyser/analyser-orchestrator.md`)

**Role:** Receives the combined raw items from all 6 source agents. Acts as a mini-orchestrator — spawns 3 sub-agents simultaneously, waits for all to complete, merges their outputs into a single enriched dataset.

**Responsibilities:**
- Receive combined raw items array (tagged by source)
- Spawn all 3 analyser sub-agents in parallel, passing the full items array to each
- Wait for all 3 to complete
- Merge outputs: each item now has a relevance score (from dedup agent), a cluster assignment (from theming agent), and cross-reference links (from cross-ref agent)
- Sort merged items by relevance score descending
- Pass the top 30 items (or all if fewer) to the insight agent

**Input:** Combined raw items array from all 6 sources
**Output:** Enriched, deduplicated, scored, clustered, cross-linked items array

---

### 4.9 Dedup and Relevance Agent (`agents/analyser/dedup-relevance-agent.md`)

**Role:** Remove duplicate items and score each unique item by its potential impact for an AI development team.

**Deduplication rules:**
- If the same paper, repo, or article appears in multiple sources, keep it once and note all source mentions
- Merge items that clearly refer to the same thing (e.g. a blog post about a paper + the paper itself — keep both but link them)
- Remove exact duplicate URLs

**Relevance scoring (score each item 1–10):**

| Signal | Score boost |
|--------|-------------|
| Released by major lab (OpenAI, Anthropic, Google, Meta, Mistral) | +3 |
| Has working code / GitHub repo | +2 |
| High engagement (citations, stars, views, upvotes) | +1 to +2 |
| Directly applicable to building RAG, agents, or LLM apps | +2 |
| New model or benchmark result | +2 |
| Tutorial or how-to (immediately actionable) | +1 |
| Opinion piece or discussion (low actionability) | -1 |
| Older than 5 days | -0.5 |
| Paywalled | -1 |

**Output:** Same items array with `relevance_score` field (1–10) and `sources_found_in` array added to each item

---

### 4.10 Theme Clustering Agent (`agents/analyser/theme-clustering-agent.md`)

**Role:** Group all items into 4–6 named topic clusters based on their content and tags.

**How to cluster:**
- Read all items and their relevance tags
- Identify the 4–6 dominant themes present in this week's items (themes change week to week — do not use a fixed list)
- Examples of clusters that might emerge: "Long-context and memory", "Multimodal models", "RAG advances", "Agent frameworks and tooling", "Inference optimization and quantization", "Fine-tuning techniques", "AI safety and alignment", "Vision and image generation", "Code generation", "Voice and audio AI"
- Assign each item to its best-fit cluster
- Name each cluster with a short, descriptive label (3–5 words)
- Identify the 1–2 most important items per cluster (the "flagship" items)

**Output:** Same items array with `cluster` field added. Also output a `clusters_summary` object: a named list of clusters with item count and flagship items per cluster.

---

### 4.11 Cross-Reference Agent (`agents/analyser/cross-ref-agent.md`)

**Role:** Find relationships between items from different sources. Surface items that connect — a paper and its GitHub implementation, a YouTube explanation and the blog post it covers, a news announcement and the technical paper behind it.

**What to cross-reference:**
- Paper ↔ GitHub repo (paper has code, or repo implements a paper)
- Blog post ↔ paper (blog explains or discusses a paper)
- YouTube video ↔ paper or blog (video explains or reacts to a paper/blog)
- News article ↔ technical paper (news covers a research announcement)
- Multiple sources covering the same event (amplification signal — if 4 sources covered it, it's significant)

**For each link found, add:**
- `related_items` array on each item (IDs of related items)
- `amplification_score` — number of distinct sources that covered this item/topic (1–6)
- `cross_ref_note` — 1 sentence explaining the relationship

**Output:** Same items array with `related_items`, `amplification_score`, `cross_ref_note` fields added

---

### 4.12 Insight Agent (`agents/insight-agent.md`)

**Role:** The bridge between "what happened" and "what we should do about it." Translates the top scored, clustered, cross-linked items into concrete actionable insights tailored to the team's headcount and focus areas.

**Input received:**
- Enriched items array from analyser (scored + clustered + cross-linked)
- Team context: headcount, member descriptions if provided, focus areas if provided, from original trigger

**Responsibilities:**
- Select the top 15–20 most important items (by relevance score + amplification score)
- For each selected item, write an insight:
  - **What happened** (1–2 sentences — the fact)
  - **Why it matters** (1–2 sentences — the implication for builders)
  - **What we could do with it** (1–2 sentences — specific, concrete action)
  - **Effort estimate** (rough: 1–2 hours / half day / full day / multi-day spike)
- Group insights by cluster
- Flag the top 3 "must not miss" items for the week
- Consider the team's stated focus areas and weight those clusters more heavily in selection

**Output:** Structured insights object with clusters, items, and effort estimates. + team context passed through.

---

### 4.13 Sprint Planner Agent (`agents/sprint-planner-agent.md`)

**Role:** Takes the insights and team context, produces a ready-to-execute sprint board.

**Critical rule — no config file:**
The team size and member details come ONLY from what the user typed in the trigger. Handle all these cases:

| Trigger style | What to do |
|--------------|------------|
| "4 people free" | Generate 4 tasks. Vary topics across clusters. Name tasks Person 1–4. |
| "3 devs: Arjun ML, Priya backend, Dev frontend" | Generate 3 tasks. Assign Arjun ML/model tasks, Priya API/infra tasks, Dev UI/demo tasks. |
| "just me this week" | Generate 1 focused task — the single highest-value item. |
| "2 people, both beginners" | Generate 2 tasks scoped to reading/learning, not complex implementation. |
| "5 people, all senior" | Generate 5 tasks with higher complexity and implementation focus. |
| No headcount mentioned | Default to 3 tasks. |

**For each task, output:**
```
Task ID: T{week}-{N}
Assignee: [name or "Person N"]
Title: [Action verb + specific thing to do]
Cluster: [which theme cluster this comes from]
Why this week: [1 sentence linking to the specific finding]
What to do: [3–5 bullet points of concrete steps]
Effort: [estimated hours or days]
Source links: [1–3 direct links to the source materials]
Priority: [High / Medium / Low]
```

**Also output:**
- A 3–5 sentence weekly intro paragraph (what was the biggest story this week in AI)
- A "worth watching but not actioning" list — 3–5 items that were interesting but didn't make the sprint

**Output:** Sprint board markdown + structured task array for tracker

---

### 4.14 Tracker Agent (`agents/tracker-agent.md`)

**Role:** Persist everything, maintain history, surface carry-overs.

**Responsibilities:**
1. Read current `tracker/weekly-log.json`
2. Get current week identifier (YYYY-WW format)
3. Check previous week's items — find any with status `todo` or `in-progress` → mark as `carry-over` and surface them at the top of the new week's brief
4. Append new week entry to `weekly-log.json` with all new tasks (status: `todo`)
5. Write the full weekly brief to `output/YYYY-WW-brief.md`
6. Print a console summary to the user

**weekly-log.json schema:**
```json
{
  "last_updated": "ISO timestamp",
  "weeks": [
    {
      "week": "2025-W15",
      "trigger": "original user message",
      "team_size": 3,
      "items_found": 47,
      "clusters": ["RAG advances", "Agent frameworks", "Multimodal"],
      "tasks": [
        {
          "id": "T15-1",
          "title": "Reproduce FlashRAG benchmark pipeline",
          "assignee": "Arjun",
          "cluster": "RAG advances",
          "source_links": ["https://..."],
          "effort": "1 day",
          "priority": "High",
          "status": "todo",
          "created": "ISO timestamp",
          "completed": null
        }
      ],
      "carry_overs_from_previous": ["T14-2", "T14-3"]
    }
  ]
}
```

**Output:** Updated `tracker/weekly-log.json` + written `output/YYYY-WW-brief.md` + console summary

---

## 5. Skills — What Each Skill File Should Contain

Each skill file in `skills/` is a markdown document that teaches an agent how to do a specific technique. Build them with this content:

### `skills/web-search.md`
- How to formulate effective search queries for recent content
- How to filter results to the past 7 days
- How to extract clean content from search results
- How to handle paywalled content (summarise from preview, note paywall)
- Rate limiting and retry strategy

### `skills/youtube-transcript.md`
- How to retrieve YouTube video transcripts
- How to summarise transcripts efficiently
- How to handle videos with no transcript (use title + description + comments)
- How to identify the key timestamp moments in a long video

### `skills/pdf-extractor.md`
- How to extract text from research paper PDFs
- How to identify the abstract, contributions, and results sections quickly
- How to summarise a paper without reading every word
- How to extract figure captions and tables

### `skills/relevance-scorer.md`
- The full scoring rubric (same as section 4.9 above)
- How to normalise scores across different content types
- How to handle ties
- Example scored items for calibration

### `skills/theme-clusterer.md`
- How to identify emergent themes (do not use fixed categories)
- How to name clusters clearly
- How to handle items that fit multiple clusters (assign to best fit, note secondary)
- How to identify flagship items per cluster

### `skills/task-formatter.md`
- The exact task output format (same as section 4.13 above)
- How to write action verbs for task titles
- How to scope tasks to the stated effort estimate
- How to write concrete "what to do" bullets
- Examples of well-formed tasks vs poorly-formed tasks

### `skills/tracker-schema.md`
- The full JSON schema for `weekly-log.json` (same as section 4.14 above)
- How to handle first-run (empty log)
- How to calculate the week identifier (YYYY-WW)
- Carry-over logic rules
- Status transition rules: todo → in-progress → done / carry-over

---

## 6. Slash Commands — Build These

### `.claude/commands/run-weekly.md`

```
Command: /run-weekly

Description: Run the full weekly AI research intelligence pipeline

Usage:
  /run-weekly "your message here"

Examples:
  /run-weekly "4 people free this week, focus on RAG and agents"
  /run-weekly "just me this week, interested in everything"
  /run-weekly "3 devs: Arjun does ML, Priya does backend, Dev does frontend UI"
  /run-weekly "5 people available, we are building a multimodal RAG system"

What happens:
  1. Orchestrator parses your message for team size and focus areas
  2. All 6 source agents run in parallel (papers, github, news, youtube, blogs, podcasts)
  3. Analyser processes everything (dedup, scoring, clustering, cross-referencing) in parallel
  4. Insight agent maps findings to your team context
  5. Sprint planner generates your tasks
  6. Tracker persists everything and writes the brief
  7. Output file written to output/YYYY-WW-brief.md
```

### `.claude/commands/check-tracker.md`

```
Command: /check-tracker

Description: View the current state of the weekly tracker

Options:
  /check-tracker             → Show current week summary
  /check-tracker --week 14   → Show specific week
  /check-tracker --all       → Show all weeks with task statuses
  /check-tracker --update "T15-1 done"  → Mark a task as done
```

---

## 7. Sample Output Format (`samples/sample-brief-week01.md`)

Build the sample file with this structure — agents use it as a reference for formatting their output:

```markdown
# AI Weekly Brief — Week 15, 2025
*Generated: Monday April 14, 2025*
*Team: 3 people | Focus: RAG, Agent frameworks*
*Sources scanned: 48 items across 6 source types*

---

## This week's biggest story
[3–5 sentence paragraph about the most significant development of the week]

---

## Carry-overs from last week
- [ ] T14-2: [task title] — [assignee] — still in progress
- [ ] T14-3: [task title] — [assignee] — not started, bumped to this week

---

## Sprint board — Week 15

### T15-1 · HIGH PRIORITY · Arjun
**Reproduce the FlashRAG benchmark pipeline**
*Cluster: RAG advances | Effort: 1 day*
*Why this week: A new RAG benchmarking suite was released with 12 datasets and 5 retrieval methods. Directly applicable to our RAG Matrix validation layer.*

What to do:
- Clone the FlashRAG repo and read the README
- Run the default benchmark on one of the 12 datasets
- Compare results against our current RAG Matrix outputs
- Note which retrieval methods outperform our current approach
- Write a 1-page summary of findings

Sources: [https://arxiv.org/...] [https://github.com/...]

---

### T15-2 · MEDIUM PRIORITY · Priya
**Set up vLLM 0.4 with the new prefix caching feature**
...

---

### T15-3 · MEDIUM PRIORITY · Dev
**Build a quick demo using the new GPT-4o image input API**
...

---

## Full weekly digest

### Cluster: RAG advances (8 items this week)
*Flagship: FlashRAG benchmark suite | Amplification: 4 sources*

**[Paper] FlashRAG: A Modular Toolkit for Efficient RAG Research**
- Source: arXiv + Papers With Code + GitHub trending
- TL;DR: [2–3 sentences]
- Key contribution: [1 sentence]
- Links: [paper] [code]
- Relevance score: 9/10

...

### Cluster: Agent frameworks (6 items this week)
...

### Cluster: Inference optimisation (5 items this week)
...

---

## Worth watching (not actioning this week)
1. [Item] — [1 sentence why interesting but not urgent]
2. [Item] — ...
3. [Item] — ...

---

*Next run: /run-weekly "[your message]"*
*Tracker: tracker/weekly-log.json*
```

---

## 8. CLAUDE.md — The Master Instructions File

Build this as the root `CLAUDE.md` file. Claude Code reads this automatically at the start of every session.

```markdown
# AI Intel Agent

You are the orchestrator of an automated weekly AI research intelligence system.

## Primary command
When the user runs /run-weekly or asks you to run the weekly research pipeline:

1. Read the user's message carefully. Extract:
   - Team headcount (number of people available)
   - Member names and skills if mentioned (optional — work without them if not given)
   - Focus areas or topics of interest (optional — default to broad AI coverage)

2. Spawn ALL 6 source agents IN PARALLEL — do not run them sequentially:
   - agents/sources/papers-agent.md
   - agents/sources/github-agent.md
   - agents/sources/news-agent.md
   - agents/sources/youtube-agent.md
   - agents/sources/blogs-agent.md
   - agents/sources/podcast-community-agent.md

3. Collect results from all 6. Pass to agents/analyser/analyser-orchestrator.md
   which will spawn its 3 sub-agents in parallel internally.

4. Pass enriched results to agents/insight-agent.md
   (include original team context — headcount, focus areas)

5. Pass insights + team context to agents/sprint-planner-agent.md

6. Pass all outputs to agents/tracker-agent.md
   which persists everything and writes the output file.

7. Tell the user where their brief is and print a short summary.

## Critical rules
- NEVER ask for a team config file. Parse team info from the user's words only.
- NEVER run source agents sequentially — always parallel.
- NEVER skip the tracker step — always persist results.
- If a source is unavailable or rate-limited, skip it and note it in the output.
- If the user only says "run the weekly" with no other info, default to: 3 tasks, broad AI coverage.
- Always write output to output/YYYY-WW-brief.md using ISO week number.
- Use skills/ files for technique guidance. Use samples/ for output format reference.

## Tracker commands
- /check-tracker → read and display tracker/weekly-log.json summary
- /check-tracker --update "T15-1 done" → update a task status

## File locations
- Agent files: agents/
- Skill files: skills/
- Tracker: tracker/weekly-log.json
- Output: output/
- Sample reference: samples/sample-brief-week01.md
```

---

## 9. Key Technical Decisions to Implement

### Parallelism
Claude Code supports spawning sub-agents in parallel using the Task tool. The orchestrator must use this — do not loop through agents sequentially. The 6 source agents should all start at the same moment. The 3 analyser sub-agents should also all start at the same moment.

### No external APIs needed
All data collection is done via web search. The agents use Claude Code's built-in web search capability to hit each source. No API keys are required for the core pipeline.

### State management
The only persistent state is `tracker/weekly-log.json`. Everything else is computed fresh each run. The tracker agent is the only one that reads and writes this file.

### Error handling
If any source agent fails or a source is unavailable, the orchestrator should log the failure and continue with the results from the other agents. A run with 5/6 sources is better than a crashed run.

### Output file naming
Use Python's `datetime.isocalendar()` or JavaScript's equivalent to get the ISO week number for the filename: `output/2025-W15-brief.md`

### First run
On the first run, `tracker/weekly-log.json` will not exist or will be empty. The tracker agent should handle this gracefully — create the file with an empty weeks array, then append the first week.

---

## 10. Build Order for Claude Code

Build the files in this order to avoid dependency issues:

1. `CLAUDE.md`
2. `tracker/weekly-log.json` (empty: `{"last_updated": null, "weeks": []}`)
3. `output/.gitkeep`
4. All 7 skill files in `skills/`
5. All 6 source agent files in `agents/sources/`
6. All 4 analyser files in `agents/analyser/`
7. `agents/insight-agent.md`
8. `agents/sprint-planner-agent.md`
9. `agents/tracker-agent.md`
10. `agents/orchestrator.md`
11. Both slash command files in `.claude/commands/`
12. `samples/sample-brief-week01.md`

Then do a test run:
```
/run-weekly "2 people free this week, one does ML one does backend, broadly interested in everything AI"
```

Verify the output file is written to `output/` and `tracker/weekly-log.json` is updated before declaring the build complete.

---

*End of blueprint. Build everything above.*
