# AI Intel Agent

> **Automated weekly AI research intelligence for developer teams.** One message in → full sprint board out.

---

## What This Does

Every week, you send one natural language message. The system:

1. **Scans** the entire AI landscape — papers, GitHub, news, YouTube, blogs, podcasts, communities — in parallel
2. **Analyses** everything — deduplicates, scores by impact, clusters by theme, cross-references across sources
3. **Synthesises** actionable insights tailored to your team size and focus areas
4. **Generates** a ready-to-execute sprint board with exactly N tasks for N people, with rationale, steps, and source links
5. **Persists** everything to a tracker with carry-over logic for unfinished tasks

No config files. No API keys. No team profiles JSON. Everything is parsed from your natural language trigger.

---

## Quick Start

```
/run-weekly "4 people free this week, focus on RAG and agents"
```

That's it. The system returns:
- `output/YYYY-WW-brief.md` — full weekly digest + sprint board
- `tracker/weekly-log.json` — updated with new tasks
- Console summary — stats and task overview

---

## Usage Guide

### Trigger Format

The trigger is a single natural language message. You can include:

| What | Examples | Default if omitted |
|------|----------|-------------------|
| Team size | "4 people", "3 devs", "just me", "me and a colleague" | 3 tasks |
| Member names | "Arjun, Priya, Dev" | Person 1, Person 2, etc. |
| Member roles | "Arjun does ML, Priya does backend" | Topics varied automatically |
| Focus areas | "focus on RAG", "interested in agents and multimodal" | Broad AI coverage |
| Experience level | "all senior", "we're beginners", "mixed team" | Intermediate |

### Trigger Examples

```
# Basic — just a headcount
/run-weekly "4 people free this week"

# With focus areas
/run-weekly "3 people, focus on RAG and agent frameworks"

# With named members and roles
/run-weekly "3 devs: Arjun does ML, Priya does backend, Dev does frontend UI"

# Solo
/run-weekly "just me this week, interested in everything"

# Experienced team with a specific product focus
/run-weekly "5 senior engineers building a multimodal RAG system for legal documents"

# Small beginner team
/run-weekly "2 people, both beginners learning about LLMs and prompt engineering"

# Named team without roles (topics varied automatically)
/run-weekly "4 people: Alice, Bob, Carol, Dave — focus on inference optimization"
```

---

## Sample Inputs and Expected Outputs

### Input 1: Basic 4-person team
```
/run-weekly "4 people free this week, focus on RAG and agents"
```

**What happens:**
- 4 tasks generated, spread across RAG and agent clusters
- Tasks named Person 1–4 (or with inferred roles if context hints exist)
- Source links from actual papers/repos found this week

**Output structure:**
```
output/2025-W15-brief.md
├── This week's biggest story (3–5 sentences)
├── Carry-overs from last week (if any)
├── Sprint board — 4 tasks
│   ├── T2025-W15-1 · HIGH · Person 1 — RAG task
│   ├── T2025-W15-2 · HIGH · Person 2 — Agent task
│   ├── T2025-W15-3 · MEDIUM · Person 3 — Supporting task
│   └── T2025-W15-4 · MEDIUM · Person 4 — Adjacent topic
├── Full weekly digest (all clusters, all items with TL;DRs)
└── Worth watching (3–5 items that didn't make the sprint)
```

---

### Input 2: Named team with roles
```
/run-weekly "3 devs: Arjun does ML research, Priya does backend/infra, Dev does frontend"
```

**Task assignment logic:**
- **Arjun (ML)** → Paper reproduction, model evaluation, benchmarking
- **Priya (backend/infra)** → Library integration, API setup, inference server
- **Dev (frontend)** → Demo building, UI prototype, new API usage

**Sample task for Arjun:**
```
Task T2025-W15-1 · HIGH · Arjun
Reproduce the FlashRAG benchmark pipeline
Why this week: FlashRAG released a modular RAG evaluation toolkit with 12 datasets...
What to do:
- Clone the repo and run the quickstart
- Compare results against our current DPR baseline
- Write a 1-page summary of which retrieval method wins
Effort: 1 day
Sources: [arxiv link] [github link]
```

---

### Input 3: Solo run
```
/run-weekly "just me this week, deep interest in inference optimization"
```

**What happens:**
- 1 task generated — the single highest-value, most actionable item
- Focused specifically on inference optimization
- Effort scoped appropriately for solo work

---

### Input 4: No message
```
/run-weekly
```

**Defaults to:**
- 3 tasks
- Broad AI coverage (no focus filter)
- Intermediate team level
- Team members named Person 1, Person 2, Person 3

---

## Tracker Commands

```bash
# View current week's sprint board and task statuses
/check-tracker

# View a specific week
/check-tracker --week 15

# View all weeks
/check-tracker --all

# Mark a task as done
/check-tracker --update "T2025-W15-1 done"

# Mark a task as in-progress
/check-tracker --update "T2025-W15-2 in-progress"

# Add a note to a task
/check-tracker --update "T2025-W15-3 notes: Blocked on GPU access, trying Colab"

# View statistics
/check-tracker --stats
```

---

## System Architecture

```
USER TRIGGER (natural language)
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                          │
│  Parses: headcount, names, roles, focus areas            │
└─────────────────────────────────────────────────────────┘
        │ (parallel fan-out — all 6 at once)
   ┌────┴────┬────────┬────────┬────────┬──────┐
   ▼         ▼        ▼        ▼        ▼      ▼
[Papers] [GitHub] [News] [YouTube] [Blogs] [Community]
   └────┬────┴────────┴────────┴────────┴──────┘
        │ (combined: ~50-70 raw items)
        ▼
┌─────────────────────────────────────────────────────────┐
│               ANALYSER ORCHESTRATOR                      │
│  [Dedup+Score] [Theme Cluster] [Cross-Ref] — in parallel │
│  Enriched top-30 items                                   │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                   INSIGHT AGENT                          │
│  Top 15-20 items → What happened / Why it matters /     │
│  What we could do / Effort estimate                      │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│               SPRINT PLANNER AGENT                       │
│  N tasks for N people, assigned by role, with steps     │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                  TRACKER AGENT                           │
│  Writes output/YYYY-WW-brief.md                          │
│  Updates tracker/weekly-log.json                         │
│  Surfaces carry-overs from last week                     │
└─────────────────────────────────────────────────────────┘
        │
        ▼
   OUTPUT TO USER
```

---

## File Structure

```
ai-intel-agent/
├── CLAUDE.md                          ← System instructions (auto-loaded)
├── README.md                          ← This file
│
├── .claude/commands/
│   ├── run-weekly.md                  ← /run-weekly slash command
│   └── check-tracker.md              ← /check-tracker slash command
│
├── agents/
│   ├── orchestrator.md               ← Entry point
│   ├── sources/
│   │   ├── papers-agent.md           ← arXiv, HuggingFace, PapersWithCode
│   │   ├── github-agent.md           ← GitHub trending, releases, HF models
│   │   ├── news-agent.md             ← TechCrunch, Verge, VentureBeat, etc.
│   │   ├── youtube-agent.md          ← Karpathy, Kilcher, labs, etc.
│   │   ├── blogs-agent.md            ← Lab blogs, researcher blogs, newsletters
│   │   └── podcast-community-agent.md ← Reddit, HN, ProductHunt, podcasts
│   ├── analyser/
│   │   ├── analyser-orchestrator.md  ← Coordinates 3 sub-agents in parallel
│   │   ├── dedup-relevance-agent.md  ← Dedup + 1-10 scoring
│   │   ├── theme-clustering-agent.md ← Groups into 4-6 clusters
│   │   └── cross-ref-agent.md        ← Links items across sources
│   ├── insight-agent.md              ← What happened → what to do
│   ├── sprint-planner-agent.md       ← N tasks for N people
│   └── tracker-agent.md             ← Persist + write output
│
├── skills/
│   ├── web-search.md                 ← Query formulation, date filtering, paywall handling
│   ├── youtube-transcript.md         ← Transcript extraction + fallbacks
│   ├── pdf-extractor.md              ← Fast paper reading strategy
│   ├── relevance-scorer.md           ← 1-10 scoring rubric
│   ├── theme-clusterer.md            ← Emergent clustering guide
│   ├── task-formatter.md             ← Sprint task format + examples
│   └── tracker-schema.md            ← JSON schema + carry-over logic
│
├── tracker/
│   └── weekly-log.json               ← Persistent history (all weeks)
│
├── output/                           ← Generated briefs (YYYY-WW-brief.md)
│   └── .gitkeep
│
└── samples/
    └── sample-brief-week01.md        ← Reference output format
```

---

## Output Format Reference

Each weekly run generates `output/YYYY-WW-brief.md`:

```markdown
# AI Weekly Brief — Week {WW}, {YYYY}
*Generated: {date}*
*Team: {N} people | Focus: {topics}*
*Sources scanned: {N} items across 6 source types*

## This week's biggest story
[3-5 sentence narrative of the week's most significant development]

## Carry-overs from last week
[Tasks from last week that weren't completed]

## Sprint board — Week {WW}
[N formatted tasks with: title, assignee, priority, cluster, effort, steps, links]

## Full weekly digest
[All items grouped by cluster, with TL;DRs and relevance scores]

## Worth watching (not actioning this week)
[3-5 items that are interesting but not urgent]
```

See `samples/sample-brief-week01.md` for a complete example.

---

## Tracker Schema

`tracker/weekly-log.json` persists all runs:

```json
{
  "last_updated": "2025-04-14T09:00:00Z",
  "weeks": [
    {
      "week": "2025-W15",
      "trigger": "4 people free this week, focus on RAG and agents",
      "team_size": 4,
      "items_found": 65,
      "clusters": ["RAG pipeline improvements", "Agent frameworks", "..."],
      "carry_overs_from_previous": [],
      "tasks": [
        {
          "id": "T2025-W15-1",
          "title": "Reproduce the FlashRAG benchmark pipeline",
          "assignee": "Person 1",
          "cluster": "RAG pipeline improvements",
          "effort": "1 day",
          "priority": "High",
          "status": "todo",
          "created": "2025-04-14T09:00:00Z",
          "completed": null
        }
      ]
    }
  ]
}
```

---

## How the Scoring Works

Every item is scored 1–10:

| Signal | Score |
|--------|-------|
| **Start** | 5/10 |
| Major lab release (OpenAI/Anthropic/Google/Meta/Mistral) | +3 |
| Has GitHub repo with working code | +2 |
| New model or benchmark | +2 |
| Applicable to RAG/agents/LLM building | +2 |
| High engagement (see full rubric) | +1 to +2 |
| Tutorial / how-to | +1 |
| Multi-source coverage | +1 |
| Opinion piece, no technical content | -1 |
| Paywalled | -1 |
| Older than 5 days | -0.5 |

Cap: 10. Floor: 1.

---

## No Setup Required

- No API keys needed — data collection uses web search
- No team config file — team info is parsed from your trigger each week
- No database — state lives in `tracker/weekly-log.json`
- First run: tracker auto-creates with empty state

---

## Troubleshooting

**"No items found for my focus area"**
The system falls back to broad AI coverage. Try a broader trigger next run.

**"Some sources failed"**
Normal — the system continues with available sources. A run with 5/6 sources beats a crashed run.

**"Tracker not updating"**
Check that `tracker/weekly-log.json` is writable. The output brief is still generated even if tracker write fails.

**"I want to re-run this week"**
Run `/run-weekly` again. The tracker creates a new entry for the same week number.
