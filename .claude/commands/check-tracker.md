View and update the weekly tracker state.

Read the arguments to determine what to display or update.

## Argument handling

**No arguments** → Show current week summary:
- Read tracker/weekly-log.json
- Find the most recent week entry
- Display: week ID, trigger, team size, clusters, task statuses, any carry-overs

**--week {N}** → Show specific week by number:
- Find the entry where week contains "W{N}" (e.g. --week 15 finds "2025-W15")
- Display full week details including all tasks and their statuses

**--all** → Show all weeks with task statuses:
- Display a summary table: week | team size | tasks | completed | carry-overs
- Then list all tasks across all weeks grouped by week

**--update "{task_id} {new_status}"** → Update a task's status:
- Parse task ID (e.g. "T2025-W15-1") and new status ("done", "in-progress")
- Find the task in tracker/weekly-log.json (search all weeks)
- Update the status field
- If status is "done", set completed timestamp to now
- If format is "T15-1 notes: {text}", append to the task's notes field
- Save the updated tracker
- Confirm: "Task {id} updated to {status} ✓"

**--stats** → Show aggregate statistics:
- Total tasks created across all weeks
- Completion rate (done / total)
- Average tasks per week
- Most common clusters
- Most active assignees

## Status values
Valid status transitions:
- todo → in-progress → done
- todo → carry-over
- in-progress → carry-over  
- carry-over → done

## Examples
- /check-tracker
- /check-tracker --week 15
- /check-tracker --all
- /check-tracker --update "T2025-W15-1 done"
- /check-tracker --update "T2025-W15-2 in-progress"
- /check-tracker --update "T2025-W15-3 notes: Blocked on GPU access, trying Colab instead"
- /check-tracker --stats
