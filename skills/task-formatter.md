# Skill: Task Formatter

**Purpose:** Define the exact format for sprint tasks, with guidance on writing action verbs, scoping effort, and crafting concrete "what to do" bullets.

---

## 1. Exact Task Output Format

Every task MUST follow this format exactly:

```
---
**Task ID:** T{YYYY-WW}-{N}
**Assignee:** [Full name or "Person N"]
**Priority:** HIGH / MEDIUM / LOW
**Cluster:** [Theme cluster name]
**Effort:** [X hours / X days / multi-day spike]

### [Action verb] [specific thing to do]

**Why this week:**
[1 sentence linking this task directly to a specific finding from this week's research. Must reference the source item.]

**What to do:**
- [Concrete step 1]
- [Concrete step 2]
- [Concrete step 3]
- [Concrete step 4 — optional]
- [Concrete step 5 — optional]

**Source links:**
- [URL 1 — primary source]
- [URL 2 — secondary source, optional]
- [URL 3 — optional]
---
```

### Task ID format
- `T` + ISO year-week + `-` + task number within that week
- Example: `T2025-W15-1`, `T2025-W15-2`, `T2025-W15-3`
- Numbers are sequential per week, starting at 1

---

## 2. Writing Strong Action Verbs for Task Titles

The task title should start with a strong, specific action verb. The verb sets expectations for what the assignee will actually do.

### Verb categories and when to use them

| Verb | When to use | Example |
|------|------------|---------|
| **Reproduce** | Replicate a paper's results or benchmark | "Reproduce the FlashRAG benchmark pipeline" |
| **Implement** | Build a feature or technique from scratch | "Implement prefix caching with vLLM 0.4" |
| **Evaluate** | Test something against your current system | "Evaluate ColBERT v3 on our retrieval pipeline" |
| **Integrate** | Add a new library or capability to existing code | "Integrate the new LangChain v0.2 streaming API" |
| **Build** | Create a new demo, prototype, or tool | "Build a demo using the GPT-4o image input API" |
| **Read and summarise** | For learning tasks or long-form content | "Read and summarise the Attention Sinks paper" |
| **Benchmark** | Compare performance of options | "Benchmark Llama 3 vs Mistral 7B on our eval set" |
| **Prototype** | Quick exploratory implementation | "Prototype a multi-agent loop using CrewAI" |
| **Review and report** | Audit or assess something | "Review all AutoGen v0.4 breaking changes" |
| **Set up** | Configure tooling or infrastructure | "Set up a local vLLM inference server" |

### What makes a bad title
- Vague: "Look into RAG stuff" → Too vague
- Non-actionable: "Read about agents" → No concrete deliverable
- Too broad: "Research everything on LLMs this week" → Not scoped
- Past tense: "Reviewed the paper" → Should be imperative

---

## 3. Scoping Tasks to Effort Estimates

Tasks must be right-sized. Use these guidelines:

| Effort level | What it means | Appropriate task scope |
|-------------|--------------|----------------------|
| **2–3 hours** | Afternoon task | Read a paper + write notes; run a benchmark; read a repo README + test one feature |
| **Half day (4 hrs)** | Morning or afternoon session | Reproduce a benchmark pipeline; build a quick prototype; write a 1-page technical summary |
| **1 day** | Full day | Implement a new technique end-to-end; set up and configure a new tool with evaluation; write a detailed comparison |
| **2–3 days** | Multi-day project | Full integration into production system; comprehensive benchmarking across multiple approaches; tutorial creation |
| **Multi-day spike** | Open-ended exploration | Experimental implementation; deep dive into new research direction; architecture evaluation |

### Calibrating to team experience level
- **Beginners**: Bias toward "read and summarise" and "set up" tasks; cap at 1 day effort
- **Intermediate**: Mix of learning and implementation tasks; all effort levels appropriate
- **Senior**: Bias toward "implement", "evaluate", "benchmark"; can handle multi-day spikes

---

## 4. Writing Concrete "What to Do" Bullets

Each bullet should be:
1. **Specific** — tells exactly what to do, not what to think about
2. **Verifiable** — produces a visible artifact or result
3. **Ordered** — steps build on each other logically

### Good vs bad bullets

❌ **Bad:** "Look into the paper and understand it"
✅ **Good:** "Read sections 1, 2, and 4 of the FlashRAG paper (skip related work)"

❌ **Bad:** "Try to get the code working"
✅ **Good:** "Clone the repo, run `pip install -e .`, execute `python run_benchmark.py --dataset nq` from the README"

❌ **Bad:** "Compare with what we have"
✅ **Good:** "Run both FlashRAG and our current pipeline on the NQ dataset. Record F1, latency, and memory usage in a comparison table"

❌ **Bad:** "Share findings"
✅ **Good:** "Write a 200-word Slack summary with your top 3 findings and whether we should adopt this"

### Standard closing bullets
Every task should end with one of:
- "Write a 200–300 word summary of your findings and share in [Slack/Notion/wherever]"
- "Note any blockers or follow-up questions in the tracker"
- "Demo to the team in the next sync if you get a working prototype"

---

## 5. Priority Assignment Rules

| Priority | When to assign |
|----------|---------------|
| **HIGH** | Item scored 8–10 + directly applicable + time-sensitive (competitor just released it; major version just dropped) |
| **MEDIUM** | Item scored 6–7 + applicable to current work or near-term goals |
| **LOW** | Item scored 4–5 + interesting but not urgent; background reading; "worth knowing" |

### Constraints
- At most 1 HIGH priority task per person per week
- The default for unspecified focus areas is MEDIUM
- Never assign LOW priority to the only task a person has

---

## 6. Well-Formed vs Poorly-Formed Task Examples

### Well-formed task
```
Task ID: T2025-W15-1
Assignee: Arjun
Priority: HIGH
Cluster: RAG pipeline improvements
Effort: 1 day

### Reproduce the FlashRAG benchmark pipeline

Why this week:
FlashRAG was released this week with 12 benchmark datasets and 5 retrieval methods — directly
applicable to validating our RAG Matrix retrieval layer.

What to do:
- Clone https://github.com/RUC-NLPIR/FlashRAG and read the README (30 min)
- Install dependencies: `pip install flashrag`
- Run the default dense retrieval benchmark on the NQ dataset
- Compare F1 scores against our current DPR baseline
- Write a 1-page summary: what outperforms our current setup and why

Source links:
- https://arxiv.org/abs/2405.13576
- https://github.com/RUC-NLPIR/FlashRAG
```

### Poorly-formed task (what NOT to do)
```
Task: Look at RAG stuff
Person: Someone
Do: Research RAG and report back
Priority: Important
```
