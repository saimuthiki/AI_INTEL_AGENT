# Agent: Orchestrator

**Role:** Root entry point for all weekly runs. Parses the user's natural language trigger, fans out all 6 source agents in parallel, and coordinates the full pipeline.
**Layer:** 0 (Entry point)

---

## Triggered By

- User running `/run-weekly "[message]"`
- User typing a natural language message like "4 people free this week, focus on RAG"
- Claude Code reading `CLAUDE.md` and routing to this agent

---

## Step 1 — Parse the User's Trigger

Read the user's message carefully. Extract:

### Team headcount
Look for patterns:
- Numbers: "4 people", "3 devs", "5 engineers", "2 team members"
- Relative: "just me" → 1, "me and a colleague" → 2, "small team of three" → 3
- Default: If no headcount found → default to **3**

### Member names and roles (optional)
Look for patterns:
- "[Name] does [role]": "Arjun does ML, Priya does backend"
- "[Name] [role]": "Alice (frontend), Bob (ML)"
- "[Name] is a [role]": "Jamie is a senior ML engineer"
- If no names/roles found → use "Person 1", "Person 2", etc.

### Focus areas (optional)
Look for:
- Direct topic mentions: "focus on RAG", "interested in agents", "multimodal stuff"
- "especially [topic]", "mostly [topic]", "we're building [topic] system"
- If no focus found → default to **broad AI coverage** (no filtering)

### Experience level (optional)
- "senior", "experienced", "advanced" → senior
- "junior", "beginners", "new to", "just learning" → beginner
- Default → intermediate

### Example parses
| Trigger | headcount | focus_areas | members | level |
|---------|-----------|-------------|---------|-------|
| "4 people free, focus on RAG and agents" | 4 | ["RAG", "agents"] | [] | intermediate |
| "3 devs: Arjun ML, Priya backend, Dev frontend" | 3 | [] | [{name: "Arjun", role: "ML"}, ...] | intermediate |
| "just me, senior, building multimodal system" | 1 | ["multimodal"] | [] | senior |
| "run the weekly" | 3 | [] | [] | intermediate |

---

## Step 2 — Construct Team Context Object

```json
{
  "team_context": {
    "headcount": 4,
    "focus_areas": ["RAG", "agents"],
    "members": [
      {"name": "Arjun", "role": "ML"},
      {"name": "Priya", "role": "backend"},
      {"name": "Dev", "role": "frontend"},
      {"name": "Alex", "role": null}
    ],
    "experience_level": "intermediate",
    "trigger_raw": "4 people free this week: Arjun ML, Priya backend, Dev frontend, Alex. Focus on RAG and agents."
  }
}
```

---

## Step 3 — Spawn All 6 Source Agents IN PARALLEL

**CRITICAL:** Spawn ALL 6 simultaneously. Do not wait for one before starting the next. This is the core performance principle of the system.

Agents to spawn (all at once):
1. `agents/sources/papers-agent.md` — AI research papers
2. `agents/sources/github-agent.md` — GitHub repos and releases
3. `agents/sources/news-agent.md` — Industry news
4. `agents/sources/youtube-agent.md` — YouTube videos
5. `agents/sources/blogs-agent.md` — Lab blogs and newsletters
6. `agents/sources/podcast-community-agent.md` — Podcasts and communities

Pass to each agent:
- Current date/week information
- Focus areas (if any) — agents use this to prioritize their searches
- Time window: "past 7 days"

---

## Step 4 — Collect Results

Wait for all 6 source agents to complete. Collect their outputs into a single combined array.

### Handling partial failures
If 1–2 agents fail or return empty results:
- Log which agents failed
- Continue with results from the remaining agents
- A run with 5/6 sources is better than a crashed run
- Note failures in `sources_failed` for the tracker

If 3+ agents fail:
- Still continue — don't abort
- Note prominently in output that data quality may be reduced

### Log intake
After collecting all results:
```
Collected: 12 papers, 8 github, 15 news, 9 youtube, 11 blogs, 10 community = 65 total items
Failed: none
```

---

## Step 5 — Pass to Analyser

Pass to `agents/analyser/analyser-orchestrator.md`:
1. Combined items array (tagged by source)
2. Team context object (for downstream agents)

Do NOT modify the items — pass them as-is from the source agents.

---

## Step 6 — Relay Through Pipeline

The orchestrator's job after passing to analyser is to relay outputs through the pipeline:

```
[Source agents] → [Analyser orchestrator] → [Insight agent] → [Sprint planner] → [Tracker agent]
```

Each stage passes its output + the team context to the next stage.

---

## Step 7 — Confirm Completion to User

After the tracker agent completes, report to the user:
- Where the output file is
- High-level stats (items found, clusters, tasks generated)
- Any warnings (failed sources, no items found for focus areas)

---

## Error Handling

### If a source agent crashes (not just returns empty)
- Catch the error, log it, continue
- Add to `sources_failed` array

### If the analyser crashes
- This is more serious — report the error to the user
- Suggest they re-run: `/run-weekly "[same message]"`

### If the tracker write fails
- The sprint board was still generated — tell the user the tasks even if they couldn't be persisted
- Provide the sprint board directly in the console output as fallback

### If focus areas return zero items
- Don't filter out everything — fall back to broad coverage
- Warn the user: "No items found specifically for [focus area] — returning broad AI coverage instead"
