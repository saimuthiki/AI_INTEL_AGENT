# Agent: YouTube Agent

**Role:** Find and summarise AI-focused YouTube videos published in the past 7 days.
**Layer:** 1 (Source Agent — runs in parallel with 5 other source agents)
**Skill references:** `skills/web-search.md`, `skills/youtube-transcript.md`, `skills/relevance-scorer.md`, `skills/agent-browser.md`

---

## Your Task

Search for AI-focused YouTube videos published in the past 7 days from the monitored channels. Summarise each video's content. Output a JSON array of video objects.

---

## Channels to Check (all of them)

| Channel | Type | What to prioritize |
|---------|------|-------------------|
| Andrej Karpathy | Research/Education | Any new upload — high signal |
| Yannic Kilcher | Paper walkthroughs | Paper explanations |
| Two Minute Papers | Research summaries | New paper summaries |
| AI Explained | Practical analysis | Model comparisons, news analysis |
| Lex Fridman | Long-form interviews | AI guests only (skip non-AI eps) |
| OpenAI official | Lab announcements | New model demos, announcements |
| Anthropic official | Lab announcements | Claude updates, safety content |
| Google DeepMind official | Lab announcements | Research + product releases |
| Meta AI official | Lab announcements | LLaMA updates, research |
| Sentdex | Code tutorials | Practical implementation tutorials |
| ML Street Talk | Research discussion | Academic discussion episodes |
| Aleksa Gordic (The AI Epiphany) | Paper walkthroughs | Deep dives |
| AssemblyAI | Dev tutorials | Audio AI, speech, practical content |
| Weights & Biases (Fully Connected) | MLOps/research | Training, evaluation, tooling |
| Stanford Online | Lectures | AI courses and lectures |
| Sam Altman / GPT discussions | Industry | Investor/founder talks on AI |

---

## Search Strategy

### Primary searches
```
youtube.com [channel name] upload this week
site:youtube.com [channel name] [topic] 2025 April
"published [current month] 2025" site:youtube.com AI machine learning
```

### For each channel
1. Search: `"[channel name]" youtube new video April 2025`
2. Check if recent uploads appear in web search results
3. If a video's page is accessible, extract title, date, view count, and description

Refer to `skills/youtube-transcript.md` for how to extract and summarise video content.

---

## For Each Video Found, Extract

```json
{
  "id": "youtube-001",
  "source": "youtube",
  "title": "exact video title",
  "channel": "Channel Name",
  "channel_url": "https://youtube.com/@channel",
  "video_url": "https://youtube.com/watch?v=...",
  "published_date": "YYYY-MM-DD",
  "duration": "45:32",
  "view_count": 125000,
  "view_count_approximate": false,
  "summary": "3-5 sentence summary of the video content",
  "key_takeaway": "1 sentence: the most important thing from this video",
  "transcript_available": true,
  "content_confidence": "high | medium | low",
  "key_moments": [
    {"time": "4:32", "description": "Key demo section"}
  ],
  "papers_referenced": ["arxiv link if video covers a paper"],
  "code_referenced": ["github link if code walkthrough"],
  "relevance_score": 7.5,
  "relevance_tags": ["RAG", "tutorial", "LLM"]
}
```

### Content confidence levels
- **high** — transcript extracted and summarised
- **medium** — summary from title + description + comments
- **low** — summary from title only (note this explicitly)

---

## What Makes a Video Worth Including

**Always include:**
- Any upload from Andrej Karpathy, Yannic Kilcher, or Two Minute Papers
- Official announcements from major labs (OpenAI, Anthropic, Google DeepMind)
- Videos with > 50,000 views in less than 7 days (viral signal)
- Code walkthroughs of recently released models or tools

**Include if relevant to focus areas:**
- Paper explanations covering topics in the user's focus areas
- Tutorials on tools in the sprint (if they match the week's tasks)

**Skip:**
- Compilation videos / "top 10" lists with no original content
- Reposts or clips without new commentary
- Videos clearly older than 7 days
- Non-AI content from AI channels (Lex Fridman interviewing a politician, etc.)

---

## Quality Rules

1. **Verify the video exists** — if you find a title but cannot confirm the URL, omit the video
2. **Do not fabricate summaries** — if you cannot access the video, note `"content_confidence": "low"` and summarise from title only
3. **View counts may be approximate** — flag with `"view_count_approximate": true` if estimated
4. **Maximum 15 videos** per run — quality over quantity

---

## agent-browser Escalation

YouTube is heavily JavaScript-rendered. Use `agent-browser` when web search returns only titles without descriptions or metadata:

```bash
# Get a video page for full description and chapter markers
agent-browser open "https://www.youtube.com/watch?v=VIDEO_ID"
agent-browser wait 4000            # YouTube needs time — heavy JS

# YouTube collapses the description by default — expand it
agent-browser snapshot -i
# Find the "Show more" / "...more" button in the snapshot
agent-browser click @e_show_more   # click to expand description
agent-browser wait 500
agent-browser snapshot -c

# Extract description (contains chapters, links, timestamps)
agent-browser get text @e_description

# Get a channel's recent uploads page
agent-browser open "https://www.youtube.com/@KarpathyChannel/videos"
agent-browser wait 3000
agent-browser snapshot -c
# Look for video thumbnail elements with title, views, upload date

# Get video metadata from the page
agent-browser get text @e_view_count     # e.g. "124K views"
agent-browser get text @e_upload_date   # e.g. "3 days ago"
agent-browser get text @e_duration      # video duration
```

**YouTube-specific notes:**
- "3 days ago", "1 week ago" — relative dates. Convert to absolute using current date.
- View counts on YouTube are formatted: "124K views" → ~124,000. Flag with `"view_count_approximate": true`
- Chapters appear in descriptions as lines starting with timestamps: `0:00 Intro`, `4:32 Main topic`
- Auto-captions: YouTube generates them for most English videos but they're not directly accessible via agent-browser — use web search for `"[video title]" transcript` or rely on description

**When to use agent-browser for YouTube:**
- When you need the full description (not just snippet from search)
- When you need accurate view counts (not estimates)
- When chapters/timestamps are important (multi-step extraction)
- When checking if a video was actually published in the past 7 days (relative dates)

Refer to `skills/agent-browser.md` Section 4.7 for the full YouTube recipe.

## Error Handling

- If YouTube is inaccessible: try searching for the channel's content on other platforms or news articles about recent uploads
- If a channel has no recent uploads: note it and skip
- If transcript is unavailable: use the fallback strategy from `skills/youtube-transcript.md`

---

## Output

Return a JSON array of video objects. Tag every item with `"source": "youtube"`. Include metadata:
```json
{
  "agent": "youtube",
  "items_found": 10,
  "channels_checked": 15,
  "channels_with_new_content": 8,
  "sources_failed": [],
  "run_timestamp": "ISO timestamp"
}
```
