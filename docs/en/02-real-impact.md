# 📊 graphify's Real Impact on This Project

> All numbers are **measured** (2026-08-16, `graphify 0.9.43`) — no estimates or invention.
> Token counts use the `cl100k_base` tokenizer; the file set is source code only
> (`.py` + `.ts` + `.tsx` under `services/`, excluding `node_modules`/`dist`).
> Source project: **a multi-model financial forecasting platform** (private).

🇹🇷 [Türkçe sürüm](../tr/02-gercek-etki.md)

---

## 📐 Measured Data

### Codebase (the world mapped into the graph)

| Metric | Value |
|---|---|
| Source files | **67 files** |
| Total lines of code | **7,580 lines** |
| Raw text size | **265 KB · 71,802 tokens** |
| First extract (right after install) | 354 nodes · 680 edges · 13 communities |
| Final extract (after all development) | **562 nodes · 931 edges · 44 communities** |

> The graph grew with the code: as the project expanded (features + backfill + UI), the
> graph grew from 354→562 nodes. Each `graphify update .` refreshes it via AST in seconds.

### Cost of a single query (measured)

| Metric | Value |
|---|---|
| `graphify query "forecast cone horizon ensemble"` output | **6,430 bytes · 1,694 tokens** (81 nodes) |
| Files an agent reads *without* graphify — 6 core files | **9,636 tokens** |
| Files an agent reads *without* graphify — all 19 relevant files | **16,842 tokens** |
| **Context reduction** | **82–90%** (1 − 1,694 / 9,636 … 1 − 1,694 / 16,842) |
| Whole codebase (for scale only) | 67 files · 71,802 tokens |

> The comparison is against what the agent would **actually read** to answer the same
> question — not the whole codebase. graphify returns the relationship map in
> **1,694 tokens**; without it, the agent greps and reads the relevant files
> (**9,636–16,842 tokens**, i.e. 5.7×–10× more).

### Extraction quality (real, from GRAPH_REPORT.md)

- **100% EXTRACTED · 0% AMBIGUOUS** — deterministic tree-sitter parse, no LLM
- Only **4 inferred edges** (0.4% of 931) — average confidence **0.8**
- No API calls, fully offline, **$0 cost**

### Memory system (save-result + reflect — real records)

| Metric | Value |
|---|---|
| Saved query memories | **12** |
| Useful | **11** |
| Dead-end | **1** |
| Reflect output: "Preferred sources" | the ensemble module (3×), the job module (2×) |
| "Tentative" | the API client, the historical-data module, the dataset module, the ensembler module, the walk-forward module |

> Reflect's output was actually used: in later sessions, questions about the ensemble and the job
> infrastructure went **to the ensemble and job modules first** — verified sources are prioritized.

---

## 🧪 Real Flows (happened on this project)

### Flow 1 — The "Settings page" dead-end 🚧→💡

1. `graphify query "make the settings page items wider"` → the graph **couldn't find a
   settings page in this project** (no node) → marked as a dead-end.
2. `graphify query "does this project have a settings page"` → answered in one query:
   **that page belongs to an older, unrelated project — not this one**. Saved as `useful`.
3. `graphify reflect` → "settings" is now on the **known dead-end** list → nobody re-scans
   that path.

> ⏱️ 2 queries ≈ 2 minutes. Without the graph: manually searching the UI components and page
> definitions, diving into an unrelated old project and wasting time in the wrong place…

### Flow 2 — UI audit: empty fan chart 🖼️

- The agent audited the simulation and chart components; found that the **fan chart was
  never actually built** (conditionally rendered container). The lesson was saved via
  `save-result`.
- The API client, the historical-data module, etc. were added to "Preferred sources" in LESSONS.md.
- Result: **the same class of bugs** (silent `catch`, missing unmount cleanup) became a
  priority checklist in later audits.

### Flow 3 — Backfill queue stuck 🐛

- A multi-layer error chain (a wrong worker image name, queue timeouts, a lost offset, a
  typo, a storage hiccup, a slow data fetch) — at each step `graphify path`/`query`
  navigated to the right files, and each lesson was saved.
- After reflect, the job module rose to "Preferred" with **2× useful**.

### Flow 4 — Ensemble corner solution 🎯

- `graphify query "why does the ensemble return a corner solution"` → the optimizer collapses
  onto a single model in a small sample → the **30% equal-weight shrink** decision. Lesson
  saved; the ensemble module was marked as a reliable source.

### Flow 5 — Slow data fetch 🐢→⚡

- `graphify query` reached the provider cache logic → understood why one asset's 2,197 bars were
  re-fetched every time (a cache-size condition) → fixed.
- 31-second loads dropped to **~200 ms**. Lesson: "don't re-fetch when the cache is big enough".

---

## 🔢 Summary of the numbers

| Where it helped | What it delivered |
|---|---|
| Code navigation | 9,636–16,842 tokens (files read without graphify) → 1,694 tokens per query (**82–90% reduction**) |
| Settings dead-end | 1 dead-end flag → the same mistake was never repeated |
| Debugging | `path`/`query` → the right file in one step |
| Memory | 12 memories → 11 useful; the ensemble and job modules prioritized |
| Cost | **0 API calls** — tree-sitter local parse, offline |
| Storage | All knowledge in a 599 KB `graph.json` (+ 3.9 MB extraction folder) |

> ⚠️ **Honesty note:** All numbers were measured on this project. Token counts use the
> `cl100k_base` tokenizer. "Reduction %" compares the graphify query output against the
> files the agent would actually read (grep + read) for the same question — not against the
> whole codebase. The 6-core-file figure (9,636 tokens) is the conservative end of the range;
> the 19-file figure (16,842 tokens) is every file the query points to.
