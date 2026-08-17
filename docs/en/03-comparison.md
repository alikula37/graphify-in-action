# ⚖️ Graphify vs Alternatives — A Neutral Comparison

> Goal: when does each tool make sense? graphify's pros/cons + the alternatives' pros/cons.
> No ranking or cheerleading — just comparison. 🔍

🇹🇷 [Türkçe sürüm](../tr/03-karsilastirma.md)

---

## 🆚 Summary Table

| Tool | Type | Agent context | Graph/links | Offline | Memory (lessons) | Setup |
|---|---|---|---|---|---|---|
| **Graphify** | Code graph + agent memory | ✅ `query/path/explain` | ✅ god nodes + communities | ✅ (code-only) | ✅ save-result/reflect | Light (one CLI via uv) |
| grep / ripgrep | Text search | ➖ raw lines | ❌ | ✅ | ❌ | ✅ instant |
| IDE + LSP (go-to-def, outline) | Interactive navigation | ➖ (for humans) | partial | ✅ | ❌ | ✅ (in IDE) |
| ctags / universal-ctags | Symbol index | ➖ file:line list | ❌ | ✅ | ❌ | ✅ light |
| Sourcegraph | Enterprise code search + graph | partial (via API) | ✅ | ➖ (server) | ❌ | ❌ heavy |
| CodeQL | Static analysis / security | ❌ | partial (dataflow) | ✅ | ❌ | ❌ heavy + query language |
| Semgrep | Pattern matching | ❌ | ❌ | ✅ | ❌ | ✅ |
| LLM + whole code (copy-paste) | Context stuffing | ✅ but limited | ❌ | ✅ | ❌ | ✅ but impractical |
| code2prompt / ctx | Context collectors | ✅ (pick files) | ❌ | ✅ | ❌ | ✅ |

---

## 🔬 Detailed Comparison

### 🆚 grep / ripgrep
- **graphify +:** Gives context at the **relationship level** — "who calls X, which service
  connects to what" requires 5-6 separate greps and merging; graphify answers with
  `path A B` in one command. It also saves results as "useful/dead-end" in memory.
- **grep +:** Ubiquitous, zero setup, works on every file including binary/JS, no learning
  curve. graphify only sees the AST of supported languages.
- **Verdict:** grep for instant discovery, graphify for structural discovery — in most
  projects, both together.

### 🆚 IDE + LSP
- **graphify +:** Produces **programmatic context** for an agent (LLM); IDE navigation is
  for human eyes. God nodes (e.g. a core function with 24 edges) reveal the project's true core
  up front.
- **IDE +:** Instant, interactive, debugger-integrated; graphify doesn't replace it.
- **Verdict:** They serve different consumers; graphify is built for the agent era.

### 🆚 ctags
- **graphify +:** ctags gives a file:line list with no relationships; graphify gives
  **edges** (call, import, usage) and community structure. Plus a memory system.
- **ctags +:** Lighter, broader language support, simpler.
- **Verdict:** ctags is the quick answer to "where is it defined?"; graphify to "how does it
  fit together?"

### 🆚 Sourcegraph
- **graphify +:** No setup/server, a single CLI; agent integration and offline.
- **Sourcegraph +:** Browser UI, enterprise permissions, massive repos, stronger graph
  ("where is code used") — not graphify's scale/weight category.
- **Verdict:** Small/medium local repo + agent workflow → graphify; enterprise multi-person
  code search → Sourcegraph.

### 🆚 CodeQL / Semgrep
- **graphify +:** These tools require **writing rules** (query language / patterns);
  graphify gives navigation context with zero rules. graphify doesn't do security analysis.
- **CodeQL/Semgrep +:** Real static analysis: vulnerabilities, dataflow, lint — not
  graphify's capability.
- **Verdict:** Different jobs. Security scanning → CodeQL/Semgrep; "understanding the code"
  → graphify.

### 🆚 Pasting the whole code into an LLM
- **graphify +:** Pasting the whole code (71,802 tokens) fills the context and forces the
  agent to derive relationships from scratch; graphify points the agent to the right place
  with **1,694 tokens per query** (see `02-real-impact.md`).
- **LLM full-context +:** Simple for one-off small projects; no setup.
- **Verdict:** As the repo grows, graphify's scaling advantage becomes clear.

### 🆚 code2prompt / ctx (context collectors)
- **graphify +:** The agent picks the files itself (via query); collectors need a manual
  file list and give no relationship/priority info. Also no memory (lessons).
- **Collector +:** Simple, language-agnostic, dumps the whole repo into one file with one
  command.
- **Verdict:** "Dump everything, I'll pick" vs "route intelligently".

---

## ✅ graphify's Pros

1. **Truly offline:** `--code-only` needs no API key, no LLM — local tree-sitter parse.
   (Measured: 100% extracted, only 4/931 edges inferred.)
2. **Agent-first design:** `query / path / explain` outputs are short and focused, meant to
   become LLM context directly (measured single query: 1,694 tokens).
3. **Memory system:** `save-result` + `reflect` → "which path worked" is learned;
   dead-ends aren't re-scanned. (Measured: 12 memories, 11 useful, 1 dead-end.)
4. **God nodes / communities:** The project's true core (a core function with 24 edges,
   a connection helper with 21 edges) is auto-discovered — a new agent knows "where to start" at
   second zero.
5. **Re-verify flag on code change:** Stale knowledge isn't trusted.
6. **Gitignore awareness:** Clear warning about the `graphify-out/` prompt-cache risk.

## ❌ graphify's Cons

1. **AST limit:** Only symbols/edges of supported languages; macros, dynamic calls, code in
   strings, and templates are invisible. (Hence the 0.4% "inferred" edges.)
2. **Semantic richness:** Semantic extraction of comments/docs/README requires an **API key**
   (Gemini) — `--code-only` skips it; so "understanding code" is offline, "understanding
   docs" isn't.
3. **File weight:** `graphify-out/` is 3.9 MB + `graph.json` (599 KB) is rewritten after each
   extract; disk and gitignore discipline required.
4. **Graph staleness:** If `graphify update .` isn't run after code changes, the graph goes
   stale (the re-verify flag reduces but doesn't eliminate this).
5. **Dependency:** The CLI install requires `uv` (pip works too); the node-based plugin
   integration (opencode) pulls in node_modules.
6. **Learning curve:** Without sustained `save-result`/`reflect` discipline, the memory
   system accumulates garbage.

---

## 🧭 When to Use Which? (one paragraph)

For **small single-file tasks**, grep is enough; for **interactive human navigation**, the
IDE; for **security/verification**, CodeQL/Semgrep; for **enterprise multi-person search**,
Sourcegraph. But if you want **an LLM agent to work fast and correctly in a repo** —
especially across repeated sessions (instead of re-scanning from scratch each time) and with
a memory that learns — graphify is the fit. None replaces the other; **the question type
decides the tool.**
