# 📊 graphify's Real Impact on This Project

> All numbers are **measured** (2026-08-16, `graphify 0.9.43`) — no estimates or invention.
> Token counts use the `cl100k_base` tokenizer; the file set is source code only
> (`.py` + `.ts` + `.tsx` under `services/`, excluding `node_modules`/`dist`).
> Source project: **Critic Forecast** (a multi-model financial forecasting platform).

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
| `graphify query "forecast cone horizon ensemble"` output | **6,430 bytes · 1,694 tokens** |
| Entire codebase | 71,802 tokens |
| **Token reduction** | **97.6%** (1 − 1,694 / 71,802) |
| Byte-level reduction | 97.6% (6,430 bytes / 271,540 bytes) |
| `graph.json` (all information) | 599 KB · **167,202 tokens** · 562 nodes · 931 edges |

> Instead of reading all the code and deriving relationships from scratch, the agent
> approaches each question with **~2.4% of the code** as context and resolves the rest
> through the ready-made graph.

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
| Reflect output: "Preferred sources" | `qra.py` (3×), `jobs.py` (2×) |
| "Tentative" | `client.ts`, `historical.py`, `dataset.py`, `ensembler.py`, `walkforward.py` |

> Reflect's output was actually used: in later sessions, questions about QRA and the job
> infrastructure went **to `qra.py` / `jobs.py` first** — verified sources are prioritized.

---

## 🧪 Real Flows (happened on this project)

### Flow 1 — The "Settings page" dead-end 🚧→💡

1. `graphify query "make the settings page items wider"` → the graph **couldn't find a
   settings page in this project** (no node) → marked as a dead-end.
2. `graphify query "which port is the settings page on"` → solved in one query:
   **8080 = the old project (crypto_dollar_yield), our UI is on 8081, we have no settings
   page**. Saved as `useful`.
3. `graphify reflect` → "settings" is now on the **known dead-end** list → nobody re-scans
   that path.

> ⏱️ 2 queries ≈ 2 minutes. Without the graph: manually searching App.tsx + Header + every
> page component, diving into the old project and wasting time in the wrong place…

### Flow 2 — UI audit: empty fan chart 🖼️

- The agent audited `SimulationPage.tsx` + `PriceChart.tsx`; found that the **fan chart was
  never actually built** (conditionally rendered container). The lesson was saved via
  `save-result`.
- `client.ts`, `historical.py`, etc. were added to "Preferred sources" in LESSONS.md.
- Result: **the same class of bugs** (silent `catch`, missing unmount cleanup) became a
  priority checklist in later audits.

### Flow 3 — Backfill queue stuck 🐛

- A 6-layer error chain (worker image name, RQ timeout, lost `end_offset`, NameError, redis
  blip, GLD fetch) — at each step `graphify path`/`query` navigated to the right files, and
  each lesson was saved.
- After reflect, `jobs.py` rose to "Preferred" with **2× useful**.

### Flow 4 — QRA corner solution 🎯

- `graphify query "why does QRA return a corner solution"` → the LP collapses onto a single
  model in a small sample → the **30% equal-weight shrink** decision. Lesson saved; the
  `qra.py` node was marked as a reliable source.

### Flow 5 — Slow SOL/GLD data 🐢→⚡

- `graphify query` reached the provider cache logic → understood why SOL's 2,197 bars were
  re-fetched every time (cache `len >= 3000` condition) → fixed.
- 31-second loads dropped to **~200 ms**. Lesson: "don't re-fetch when the cache is big enough".

---

## 🔢 Summary of the numbers

| Where it helped | What it delivered |
|---|---|
| Code navigation | 7,580 lines / 71,802 tokens → 1,694 tokens per query (**97.6% reduction**) |
| Settings dead-end | 1 dead-end flag → the same mistake was never repeated |
| Debugging | `path`/`query` → the right file in one step |
| Memory | 12 memories → 11 useful; `qra.py`/`jobs.py` prioritized |
| Cost | **0 API calls** — tree-sitter local parse, offline |
| Storage | All knowledge in a 599 KB `graph.json` (+ 3.9 MB extraction folder) |

> ⚠️ **Honesty note:** All numbers were measured on this project. Token counts use the
> `cl100k_base` tokenizer; "reduction %" = 1 − (query tokens / total code tokens). The
> actual savings depend on the agent's reading behavior; the ratio is fixed.
