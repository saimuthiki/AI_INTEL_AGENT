# AI Weekly Brief — Week 15, 2025
*Generated: Monday April 14, 2025*
*Team: 4 people | Focus: RAG, Agent frameworks*
*Sources scanned: 65 items across 6 source types (57 after dedup)*

---

## This week's biggest story

This week was defined by two simultaneous releases that both point in the same direction: making RAG faster and more rigorous. FlashRAG dropped a benchmarking toolkit with 12 datasets and 5 pre-built retrieval pipelines — the first time teams can compare retrieval strategies on a standard baseline without building their own harness. On the same day, vLLM 0.4 landed with prefix caching that cuts time-to-first-token by up to 40% on long-context queries. For teams building production RAG systems, the message is clear: your evaluation story just got standardised, and your inference stack just got a major free upgrade.

---

## Carry-overs from last week

- [ ] T2025-W14-2: **Set up LangChain streaming with FastAPI** — Priya — in progress (partially done, needs testing)
- [ ] T2025-W14-3: **Read the Mixtral MoE paper and evaluate for our use case** — Dev — not started, bumped to this week

---

## Sprint board — Week 15

---

**Task ID:** T2025-W15-1
**Assignee:** Arjun
**Priority:** HIGH
**Cluster:** RAG pipeline improvements
**Effort:** 1 day

### Reproduce the FlashRAG benchmark pipeline

**Why this week:**
FlashRAG was released this week as the first standardised RAG benchmarking toolkit with 12 datasets and 5 retrieval methods — directly applicable to validating our retrieval layer.

**What to do:**
- Clone https://github.com/RUC-NLPIR/FlashRAG and read the quickstart README (30 min)
- Install: `pip install flashrag` and run the default BM25 baseline on the NQ dataset
- Run the dense retrieval (DPR) baseline on the same dataset
- Compare F1 scores and latency against our current DPR baseline
- Write a 200-word Slack summary: which retrieval method beats ours and by how much

**Source links:**
- https://arxiv.org/abs/2405.13576
- https://github.com/RUC-NLPIR/FlashRAG

---

**Task ID:** T2025-W15-2
**Assignee:** Priya
**Priority:** HIGH
**Cluster:** Inference optimization
**Effort:** Half day

### Set up vLLM 0.4 with prefix caching enabled

**Why this week:**
vLLM 0.4 released with prefix caching that reduces TTFT by ~40% on our long-context RAG queries — directly applicable to the latency issues we've been seeing in production.

**What to do:**
- Update vLLM: `pip install vllm --upgrade` to get 0.4
- Read the prefix caching docs: https://docs.vllm.ai/en/latest/automatic_prefix_caching/apc.html
- Run our standard 20-query RAG benchmark with and without prefix caching enabled
- Record P50 and P95 TTFT in a comparison table
- If improvement > 20%: open a PR to enable it in production config

**Source links:**
- https://github.com/vllm-project/vllm/releases/tag/v0.4.0
- https://docs.vllm.ai/en/latest/automatic_prefix_caching/apc.html

---

**Task ID:** T2025-W15-3
**Assignee:** Dev
**Priority:** MEDIUM
**Cluster:** Agent frameworks and tooling
**Effort:** 1 day

### Build a minimal agent loop using AutoGen v0.4's new event-driven API

**Why this week:**
AutoGen v0.4 completely redesigned its agent runtime to be async and event-driven — a breaking change from v0.2, but the new architecture is far cleaner and is what CrewAI and LangGraph have been moving towards.

**What to do:**
- Read the AutoGen v0.4 migration guide (30 min): https://microsoft.github.io/autogen/docs/migration-guide
- Build a 2-agent conversation (planner + executor) using the new `AgentRuntime` API
- Port one of our existing AutoGen v0.2 workflows to the new API
- Note any breaking changes or gotchas in a brief document
- Demo in next team sync if you get it working

**Source links:**
- https://github.com/microsoft/autogen/releases/tag/v0.4.0
- https://microsoft.github.io/autogen/docs/migration-guide

---

**Task ID:** T2025-W15-4
**Assignee:** Alex
**Priority:** MEDIUM
**Cluster:** Multimodal advances
**Effort:** Half day

### Evaluate GPT-4o's new image input for our document processing pipeline

**Why this week:**
OpenAI updated GPT-4o's vision API this week with improved table and chart reading — directly relevant to the document ingestion step in our RAG pipeline where we currently skip image content.

**What to do:**
- Access the new vision API via `gpt-4o` with the `detail: "high"` parameter
- Test on 5 sample PDFs from our corpus that contain tables/charts
- Compare extracted table data against our current pdfplumber text extraction
- Estimate cost delta (vision API is priced per image token)
- Write a recommendation: is this worth integrating for our use case?

**Source links:**
- https://platform.openai.com/docs/guides/vision
- https://openai.com/blog/gpt-4o-api-update-april-2025

---

## Full weekly digest

### Cluster: RAG pipeline improvements (9 items this week)
*Flagship: FlashRAG benchmark suite | Amplification: 5 sources*

**[Paper] FlashRAG: A Modular Toolkit for Efficient RAG Research**
- Source: arXiv + Papers With Code + GitHub trending + VentureBeat + HuggingFace Papers
- TL;DR: Introduces a modular RAG benchmarking framework with 12 standard datasets, 5 retrieval methods (BM25, DPR, SPLADE, ColBERT, hybrid), and standardised evaluation metrics. First public release of a comprehensive RAG-specific evaluation harness.
- Key contribution: Standardises RAG evaluation so teams can compare retrieval methods on a common benchmark instead of building their own test suites.
- Relevance score: 9.5/10
- Links: [paper](https://arxiv.org/abs/2405.13576) [code](https://github.com/RUC-NLPIR/FlashRAG)

**[Paper] ColBERT v3: Learned Sparse Retrieval with Dense Representations**
- Source: arXiv + Papers With Code
- TL;DR: ColBERT v3 combines dense and sparse retrieval in a single end-to-end trained model, achieving SOTA on BEIR and outperforming BM25+DPR hybrid baselines by 4.2 points on average.
- Key contribution: Eliminates the need for a separate BM25 retrieval stage while maintaining interpretability.
- Relevance score: 8.0/10
- Links: [paper](https://arxiv.org/abs/2405.XXXXX)

**[GitHub] LlamaIndex v0.10.30 — New RAG evaluation module**
- Source: GitHub releases
- TL;DR: Adds a built-in `RAGEvaluator` class with faithfulness, relevancy, and context recall metrics. Previously required external libraries.
- Relevance score: 7.5/10
- Links: [release](https://github.com/run-llama/llama_index/releases/tag/v0.10.30)

*(6 more items in this cluster — see tracker/weekly-log.json for full list)*

---

### Cluster: Inference optimization (7 items this week)
*Flagship: vLLM 0.4 prefix caching | Amplification: 4 sources*

**[GitHub] vLLM v0.4.0 — Automatic Prefix Caching**
- Source: GitHub releases + VentureBeat + HuggingFace blog + Reddit r/LocalLLaMA
- TL;DR: Introduces automatic prefix caching (APC) that stores KV cache for shared prefixes across requests. On long-context RAG workloads with a shared system prompt, reduces TTFT by 35–45%.
- Key contribution: Free latency win for any deployment with shared prefixes — requires only a config flag change, no code changes.
- Relevance score: 9.0/10
- Links: [release](https://github.com/vllm-project/vllm/releases/tag/v0.4.0) [docs](https://docs.vllm.ai/en/latest/automatic_prefix_caching/apc.html)

*(6 more items in this cluster)*

---

### Cluster: Agent frameworks and tooling (6 items this week)
*Flagship: AutoGen v0.4 async redesign | Amplification: 3 sources*

**[GitHub] AutoGen v0.4 — Async event-driven architecture**
- Source: GitHub releases + Microsoft Research blog + Latent Space podcast
- TL;DR: Complete architectural rewrite from v0.2. New `AgentRuntime` is async-first, event-driven, and supports distributed multi-agent systems. Breaking change from v0.2 but migration guide is comprehensive.
- Key contribution: Makes AutoGen production-ready for async workloads and removes the blocking conversation loop model.
- Relevance score: 8.5/10
- Links: [release](https://github.com/microsoft/autogen/releases/tag/v0.4.0)

*(5 more items in this cluster)*

---

### Cluster: Multimodal advances (5 items this week)
*Flagship: GPT-4o vision API update | Amplification: 3 sources*

**[News] OpenAI updates GPT-4o vision with improved table and chart extraction**
- Source: OpenAI blog + TechCrunch + Reddit r/MachineLearning
- TL;DR: GPT-4o now extracts tabular data from images with ~85% accuracy on standard document benchmarks, up from ~60%. New `detail: "high"` mode processes at higher resolution for complex documents.
- Key contribution: Document processing pipelines that skip image/table content can now include it cost-effectively.
- Relevance score: 8.0/10
- Links: [blog](https://openai.com/blog/gpt-4o-api-update-april-2025) [docs](https://platform.openai.com/docs/guides/vision)

*(4 more items in this cluster)*

---

### Cluster: Safety and evaluation (4 items this week)
*Flagship: Anthropic Constitutional AI v2 paper | Amplification: 2 sources*

**[Paper] Constitutional AI v2: Scalable Oversight via Debate**
- Source: arXiv + Anthropic blog
- TL;DR: Updates CAI to use debate between AI models as a scalable oversight mechanism. Human evaluators only need to judge which AI argument is more persuasive, not evaluate the underlying technical claim.
- Key contribution: Addresses the core weakness of RLHF — humans can't evaluate things they don't understand — by having AI models argue opposing positions.
- Relevance score: 7.5/10
- Links: [paper](https://arxiv.org/abs/2405.XXXXX) [blog](https://anthropic.com/news/cai-v2)

*(3 more items in this cluster)*

---

## Worth watching (not actioning this week)

1. **Llama 3 70B fine-tuning guide from Meta** — Directly applicable when we're ready to fine-tune, but we're still in eval phase. Revisit in 3 weeks.
2. **OpenAI's new Assistants API v2 with streaming** — Interesting but we're on the completions API and migration would be non-trivial. Watch for team adoption signals first.
3. **New diffusion model (FLUX.1) for image generation** — High quality results but not in our current roadmap. File for future reference.
4. **Mistral 7B v0.3 with function calling** — Good open-source alternative for tool-calling but we're locked into GPT-4o for now. Revisit if cost becomes an issue.
5. **AlphaFold 3 protein structure prediction** — Impressive science but outside our current product focus. Flag for bio-ML team if applicable.

---

*Next run: /run-weekly "[your message here]"*
*Tracker: tracker/weekly-log.json*
*Output files: output/*
