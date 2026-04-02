Run the full weekly AI research intelligence pipeline.

Read the arguments provided after the command as the user's trigger message.

Then follow CLAUDE.md instructions exactly:

1. Parse the trigger for team headcount, member names/roles, focus areas, and experience level.
2. Spawn all 6 source agents IN PARALLEL (papers, github, news, youtube, blogs, podcast-community).
3. Pass combined results to the analyser orchestrator (which runs 3 sub-agents in parallel internally).
4. Pass enriched results to the insight agent.
5. Pass insights to the sprint planner.
6. Pass everything to the tracker agent to persist and write the brief.
7. Report output file location and a short summary to the user.

If no arguments are provided, default to: 3 tasks, broad AI coverage, intermediate team.

Usage examples:
- /run-weekly "4 people free this week, focus on RAG and agents"
- /run-weekly "3 devs: Arjun does ML, Priya does backend, Dev does frontend"
- /run-weekly "just me this week, interested in everything"
- /run-weekly "5 people available, we are building a multimodal RAG system, all senior engineers"
- /run-weekly "2 people, both are ML beginners learning about LLMs"
