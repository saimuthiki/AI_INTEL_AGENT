# Skill: Relevance Scorer

**Purpose:** Standardize how every item collected by source agents is scored 1–10 for relevance to an AI development team.

---

## 1. The Scoring Rubric

All items start at a baseline of **5/10**. Apply the following boosts and penalties:

### Positive signals (add to score)

| Signal | Score boost | Notes |
|--------|------------|-------|
| Released by major AI lab (OpenAI, Anthropic, Google DeepMind, Meta AI, Mistral, xAI) | +3 | Official release, not just coverage |
| Has working code with a GitHub repo | +2 | Repo must exist and be non-empty |
| New model release or significant benchmark result | +2 | Not just an incremental update |
| Directly applicable to RAG, agent systems, or LLM application building | +2 | Must be hands-on usable |
| High engagement signal | +1 to +2 | See engagement table below |
| Tutorial, walkthrough, or how-to (immediately actionable) | +1 | Step-by-step or code walkthrough |
| Multi-source amplification (same topic covered by 3+ sources) | +1 | Cross-reference signal |
| Open-source release (model weights, dataset, or framework) | +1 | Freely usable |

### Engagement signals for the +1/+2 boost

| Source type | Threshold for +1 | Threshold for +2 |
|------------|-----------------|-----------------|
| arXiv paper | 50+ citations or 100+ social shares | 200+ citations or 500+ shares |
| GitHub repo | 500+ stars | 2000+ stars |
| YouTube video | 10,000+ views | 100,000+ views |
| Reddit post | 200+ upvotes | 1000+ upvotes |
| HN post | 50+ comments | 200+ comments |
| Blog post | High-traffic publication | Top researcher's personal blog |

### Negative signals (subtract from score)

| Signal | Score penalty | Notes |
|--------|--------------|-------|
| Opinion piece, editorial, or discussion with no new technical content | -1 | Low actionability |
| Older than 5 days (within the 7-day window) | -0.5 | Recency penalty |
| Paywalled with limited preview | -1 | Low accessibility |
| Incremental update (minor version bump, typo fix) | -1 | Low impact |
| Theoretical paper with no implementation | -0.5 | Harder to act on |
| Requires expensive compute (e.g. 1000+ GPU hours to replicate) | -0.5 | Low team feasibility |

---

## 2. Score Ceilings and Floors

- **Maximum score: 10** (cap at 10 even if boosts exceed it)
- **Minimum score: 1** (floor at 1 even if penalties exceed it)
- **Round to nearest 0.5** (e.g. 7.5 is valid)

---

## 3. Normalising Scores Across Content Types

Different content types have different natural score distributions. Apply these calibration guidelines:

| Content type | Typical raw score range | Notes |
|-------------|------------------------|-------|
| Major lab model release | 8–10 | Almost always high signal |
| arXiv paper with code | 6–9 | Depends on applicability |
| arXiv paper without code | 4–7 | Less immediately useful |
| GitHub trending repo | 5–8 | Depends on relevance tags |
| YouTube tutorial (practical) | 6–8 | High actionability |
| YouTube lecture (theoretical) | 4–7 | Lower immediate actionability |
| News article (announcement) | 5–8 | Depends on what was announced |
| Opinion/analysis blog | 3–6 | Information density varies |
| Reddit/HN discussion | 3–6 | Community signal value |
| Podcast episode | 4–7 | Depends on guest and topic |

---

## 4. Handling Ties

When two or more items have the same score, rank them by:
1. **Recency** — more recent item first
2. **Amplification** — more sources covered it → higher rank
3. **Actionability** — has code or tutorial → higher rank
4. **Source authority** — major lab > independent researcher > community

---

## 5. Example Scored Items (Calibration Reference)

### Score: 10/10
> "Anthropic releases Claude 4 with 200K context, available via API today. Blog post + API docs + GitHub examples. Covered by all 6 sources."
- Major lab release (+3), has code (+2), new model (+2), applicable (+2), multi-source (+1) = 10 ✓

### Score: 8/10
> "New arXiv paper: FlashRAG — modular RAG benchmarking toolkit with 12 datasets. GitHub repo with 800 stars this week."
- Has code (+2), applicable to RAG (+2), benchmark (+2), high engagement (+1) = 8 ✓

### Score: 6/10
> "VentureBeat article: 'The future of AI agents — 5 predictions for 2025'. Opinion piece from analyst."
- No code, opinion (-1), moderate relevance tags = 6 ✓

### Score: 4/10
> "YouTube video: 2-hour theoretical lecture on transformer attention mechanisms. 500 views. No code."
- Theoretical (-0.5), low engagement (+0), no code = 4 ✓

### Score: 2/10
> "Bloomberg article: 'AI startup raises $10M seed round'. Paywalled. Minor news."
- Paywalled (-1), low actionability (-1), no technical content (-1) = 2 ✓

---

## 6. Adjusting for Focus Areas

When the user's trigger specifies focus areas (e.g. "focus on RAG and agents"):
- Items tagged with those focus areas get +1 score boost
- This is applied AFTER the base score calculation
- Maximum is still 10

Document this adjustment: `"focus_boost": 1, "focus_reason": "matches user focus area: RAG"`
