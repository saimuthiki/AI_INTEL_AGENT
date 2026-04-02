# Skill: Tracker Schema

**Purpose:** Define the complete JSON schema for `tracker/weekly-log.json`, including carry-over logic, status transitions, and first-run handling.

---

## 1. Full JSON Schema

```json
{
  "last_updated": "2025-04-14T09:00:00Z",
  "weeks": [
    {
      "week": "2025-W15",
      "trigger": "4 people free this week, focus on RAG and agents",
      "generated_at": "2025-04-14T09:00:00Z",
      "team_size": 4,
      "focus_areas": ["RAG", "agents"],
      "items_found": 47,
      "items_after_dedup": 38,
      "sources_scanned": ["papers", "github", "news", "youtube", "blogs", "podcast-community"],
      "sources_failed": [],
      "clusters": ["RAG pipeline improvements", "Agent frameworks", "Multimodal models", "Inference optimization"],
      "carry_overs_from_previous": ["T2025-W14-2", "T2025-W14-3"],
      "output_file": "output/2025-W15-brief.md",
      "tasks": [
        {
          "id": "T2025-W15-1",
          "title": "Reproduce the FlashRAG benchmark pipeline",
          "assignee": "Arjun",
          "cluster": "RAG pipeline improvements",
          "source_links": [
            "https://arxiv.org/abs/2405.13576",
            "https://github.com/RUC-NLPIR/FlashRAG"
          ],
          "effort": "1 day",
          "priority": "High",
          "status": "todo",
          "created": "2025-04-14T09:00:00Z",
          "completed": null,
          "notes": null
        }
      ]
    }
  ]
}
```

---

## 2. Field Definitions

### Root level
| Field | Type | Description |
|-------|------|-------------|
| `last_updated` | ISO timestamp or null | Timestamp of the most recent write |
| `weeks` | Array | Ordered list of weekly entries, oldest first |

### Week entry
| Field | Type | Description |
|-------|------|-------------|
| `week` | String | ISO week identifier: `YYYY-Www` (e.g. `2025-W15`) |
| `trigger` | String | The exact user message that triggered this run |
| `generated_at` | ISO timestamp | When the run completed |
| `team_size` | Integer | Number of people for whom tasks were generated |
| `focus_areas` | Array[String] | Focus areas parsed from trigger, empty array if none specified |
| `items_found` | Integer | Total raw items before deduplication |
| `items_after_dedup` | Integer | Unique items after deduplication |
| `sources_scanned` | Array[String] | Source agents that ran successfully |
| `sources_failed` | Array[String] | Source agents that failed or returned no results |
| `clusters` | Array[String] | Names of the clusters identified this week |
| `carry_overs_from_previous` | Array[String] | Task IDs carried over from the previous week |
| `output_file` | String | Path to the generated markdown brief |
| `tasks` | Array[Task] | Sprint tasks for this week |

### Task entry
| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Format: `T{YYYY}-W{WW}-{N}` (e.g. `T2025-W15-1`) |
| `title` | String | Task title starting with action verb |
| `assignee` | String | Name or "Person N" |
| `cluster` | String | Which theme cluster this task addresses |
| `source_links` | Array[String] | 1–3 direct URLs to source materials |
| `effort` | String | Human-readable estimate (e.g. "1 day", "3 hours") |
| `priority` | String | "High", "Medium", or "Low" |
| `status` | String | Status code (see transitions below) |
| `created` | ISO timestamp | When the task was created |
| `completed` | ISO timestamp or null | When status was set to "done" |
| `notes` | String or null | Optional free-text notes from the team |

---

## 3. Calculating the Week Identifier

### Formula
The ISO week number follows ISO 8601. Week 1 is the week containing the first Thursday of the year.

```python
from datetime import datetime
now = datetime.utcnow()
year, week, _ = now.isocalendar()
week_id = f"{year}-W{week:02d}"
# Example: "2025-W15"
```

```javascript
const now = new Date();
const startOfYear = new Date(now.getFullYear(), 0, 1);
const week = Math.ceil(((now - startOfYear) / 86400000 + startOfYear.getDay() + 1) / 7);
const weekId = `${now.getFullYear()}-W${String(week).padStart(2, '0')}`;
```

### Output filename from week identifier
```
output/2025-W15-brief.md
```
Always zero-pad the week number to 2 digits.

---

## 4. Carry-Over Logic

### Definition
A carry-over is a task from the previous week that was NOT completed (status is `todo` or `in-progress`) when the new week's run begins.

### Carry-over algorithm
```
1. Find the entry for the most recent previous week in weeks[]
2. Collect all tasks where status == "todo" OR status == "in-progress"
3. These task IDs become carry_overs_from_previous[] for the new week
4. Change their status to "carry-over" in the previous week's entry
5. At the top of the new week's brief, list carry-overs with their original assignee
```

### Carry-over display in the brief
```markdown
## Carry-overs from last week
- [ ] T2025-W14-2: [task title] — Priya — still in progress
- [ ] T2025-W14-3: [task title] — Dev — not started, bumped to this week
```

### First-run edge case
If `weeks` is empty, there are no carry-overs. Skip the carry-over section entirely in the brief.

---

## 5. Status Transition Rules

```
todo → in-progress → done
todo → carry-over
in-progress → carry-over
carry-over → done
```

| From | To | Trigger |
|------|-----|---------|
| `todo` | `in-progress` | Team member starts the task (via `/check-tracker --update`) |
| `in-progress` | `done` | Team member marks complete (via `/check-tracker --update "T15-1 done"`) |
| `todo` or `in-progress` | `carry-over` | Automatic — new week runs before task was completed |
| `carry-over` | `done` | Team member marks complete in a subsequent week |

### Valid status values
Only these values are allowed:
- `todo` — assigned but not started
- `in-progress` — actively being worked on
- `done` — completed
- `carry-over` — not completed when next week ran

---

## 6. First-Run Handling

When `tracker/weekly-log.json` does not exist or contains `{"last_updated": null, "weeks": []}`:

1. Do not error — treat it as a valid empty state
2. Set `carry_overs_from_previous` to `[]` (empty array)
3. Skip the carry-overs section in the brief output
4. Create the file if it doesn't exist before writing
5. Write the first week entry normally

### File creation check (pseudocode)
```python
import json, os
tracker_path = "tracker/weekly-log.json"
if not os.path.exists(tracker_path):
    with open(tracker_path, 'w') as f:
        json.dump({"last_updated": None, "weeks": []}, f, indent=2)
```

---

## 7. Updating the Tracker via `/check-tracker --update`

### Command format
```
/check-tracker --update "T2025-W15-1 done"
/check-tracker --update "T2025-W15-2 in-progress"
/check-tracker --update "T2025-W15-3 notes: Blocked on compute access"
```

### Update algorithm
1. Parse the task ID from the command
2. Find the task in `weekly-log.json` (search all weeks if needed)
3. Update the `status` field
4. If status is "done", set `completed` to current ISO timestamp
5. If notes are provided, append to the `notes` field
6. Update `last_updated` at root level
7. Write the file
8. Confirm to user: "Task T2025-W15-1 marked as done ✓"
