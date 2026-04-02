# Skill: PDF Extractor (Research Papers)

**Purpose:** Teach agents how to efficiently extract key information from research paper PDFs — without reading every word.

---

## 1. Locating Paper PDFs

### Finding PDFs from search results
Most AI papers are freely available. Priority access paths:

| Source | PDF URL pattern |
|--------|----------------|
| arXiv | `arxiv.org/pdf/{id}.pdf` (from `arxiv.org/abs/{id}`) |
| HuggingFace Papers | Link from the paper card → arXiv PDF |
| Papers With Code | Direct arXiv link in sidebar |
| OpenReview | `openreview.net/pdf?id={id}` |
| ACL Anthology | `aclanthology.org/{id}.pdf` |
| Semantic Scholar | PDF link in paper metadata |

### When PDF is not directly accessible
1. Try `{arxiv_id}` on `ar5iv.labs.arxiv.org` (HTML version of arXiv papers — easier to parse)
2. Search for the paper title + "pdf" to find mirrored versions
3. Use the abstract page to extract metadata even without full PDF access

---

## 2. Navigating Paper Structure Quickly

### Standard academic paper sections (in order)
1. **Abstract** — always read this first (100–300 words, contains everything essential)
2. **Introduction** — context, problem statement, key contributions (bullet list in last paragraph)
3. **Related Work** — what came before; skip unless cross-referencing
4. **Method / Approach** — the core technical contribution
5. **Experiments / Results** — benchmark numbers, comparison tables
6. **Conclusion** — summary of contributions and future work
7. **Appendix** — additional details; usually skippable for the brief

### Fast-reading strategy (5-minute paper read)
1. Read abstract in full (2 min)
2. Scan introduction — find the contributions bullet list (usually ends with "In summary, we contribute...") (1 min)
3. Jump to results tables — extract the headline numbers (1 min)
4. Read conclusion paragraph (1 min)

This is sufficient for generating a high-quality summary for the weekly brief.

---

## 3. Extracting Key Fields

### What to extract from every paper

```json
{
  "title": "exact title from paper",
  "authors": ["First Last", "First Last"],
  "institution": "primary institution or lab",
  "abstract_summary": "2-3 sentence synthesis of the abstract",
  "key_contribution": "1 sentence: what is new/novel about this paper",
  "method_summary": "1-2 sentences on how they do it",
  "main_results": "headline benchmark result or performance claim",
  "code_available": true,
  "code_url": "https://github.com/...",
  "dataset_released": false,
  "paper_url": "https://arxiv.org/abs/...",
  "pdf_url": "https://arxiv.org/pdf/....pdf",
  "submission_date": "YYYY-MM-DD",
  "relevance_tags": ["RAG", "retrieval", "benchmark"]
}
```

### Extracting the key contribution (most important field)
The key contribution is usually found in one of these locations:
- Last paragraph of introduction: "In this work, we..." or "Our contributions are..."
- Abstract: second or third sentence
- First sentence of conclusion

Write it as: **[verb] [what] [that achieves what]**
Example: "Introduces a sparse retrieval method that reduces latency by 40% while matching dense retrieval accuracy."

---

## 4. Figure Captions and Tables

### When to include figure information
Include figure/table data when:
- A figure shows a performance comparison (benchmark table)
- A figure shows the architecture diagram (useful for understanding novelty)
- A table shows ablation results explaining which component matters most

### Extracting table data
For benchmark tables, extract:
1. The metric name (e.g. MMLU, HellaSwag, ROUGE-L)
2. The proposed method's score
3. The best competing method's score
4. The delta (how much better)

Format: `[Method]: [score] vs [best baseline]: [score] (+[delta]%)`

---

## 5. Handling Different Paper Types

| Paper type | Focus on |
|-----------|---------|
| New model/architecture | Abstract + key contribution + main benchmark table |
| New dataset/benchmark | Dataset stats + example tasks + baseline results |
| Survey/position paper | Key claims + taxonomy + recommendations section |
| Workshop paper | Abstract only (lower signal, less rigorous review) |
| Technical report | Usually from a major lab — treat as high priority, read fully |

---

## 6. Quality Standards

Before including a paper:
- [ ] Paper is actually from the past 7 days (check submission date, not publication date)
- [ ] Key contribution is specific and factual (not "proposes a new method")
- [ ] Any benchmark numbers are quoted accurately from the paper
- [ ] Code URL is verified to exist (search for repo if not in paper)
- [ ] Do not hallucinate results — if you cannot access the paper, note `"access": "abstract_only"`
