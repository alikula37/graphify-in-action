# 🔧 Graphify Installation & Usage Guide

This guide walks through installing graphify in a project and using it effectively. Its
goal in one sentence: to let your agent (LLM assistant) work **fast and accurately** by
**reducing token/context usage** — instead of re-scanning the codebase every session.

🇹🇷 [Türkçe sürüm](../tr/01-kurulum-rehberi.md)

---

## 🎯 Decision Criterion

Before installing graphify, evaluate it against this criterion:

> If graphify doesn't help, or negatively affects read/token/context usage, **don't install it**.

On this project the criterion was met and it was installed — for the measured impact, see
[`02-real-impact.md`](02-real-impact.md).

## 📦 Installation

```bash
# In the project root, strict mode
graphify install --project --strict

# Source: https://github.com/Graphify-Labs/graphify
```

### 🧊 What Strict Mode Does

- Blocks the session's **first raw source read** and redirects the agent to the graph.
- After the query it reverts to nudge mode — it **doesn't get stuck**, and isn't a constant
  blocker.

## 🔍 How It Works

| Feature | Description |
|---|---|
| 🌳 **Local tree-sitter parse** | Code is parsed to an AST **locally, without touching an LLM** |
| 🛰️ **Fully offline** | `--code-only` needs **no API key**, no internet dependency |
| 💾 **Persistent knowledge** | The extracted graph makes re-scanning every session unnecessary |
| ⚡ **Fast updates** | `graphify update .` refreshes via AST in seconds after code changes |

## 🧠 The Learning Loop: save-result + reflect

```bash
# 1. Save a query outcome — was it useful or a dead-end?
graphify save-result --question "..." --answer "..." --type query --outcome useful|dead_end

# 2. Reflect: aggregate the accumulated lessons, tag the nodes
graphify reflect
```

- **save-result:** records which queries worked and which were **dead-ends**.
- **reflect:** aggregates the records and tags nodes as **preferred** or **tentative**;
  adds dead-ends to the "don't re-explore" list.
- **When code changes:** the relevant lesson gets a **"re-verify"** flag — stale knowledge
  is not trusted.

## ⚠️ Critical Warning: .gitignore

> If you don't add `graphify-out/` to `.gitignore`, every `extract` can bloat / invalidate
> the prompt cache with large JSONs. **It must be ignored** (or committed deliberately as
> a team map — see graphify's own docs).

## 🧭 Daily Usage

| Command | What it does |
|---|---|
| `graphify query "<question>"` | Returns a focused subgraph — the first step for code questions |
| `graphify path "<A>" "<B>"` | Shows the relationship between two concepts |
| `graphify explain "<concept>"` | Fetches an explanation of a single concept |
| `graphify update .` | Refreshes the graph after code changes |
| `graphify reflect` | Aggregates lessons, tags preferred/tentative nodes |

> 💡 Tip: Query results are typically much smaller and more focused than raw source scans —
> on this project a single query returned a **1,694-token** subgraph while the whole
> codebase is **71,802 tokens** (97.6% reduction — measured).

---

## 🤖 One-Step Install (AI Agent Prompt)

Give the prompt below to your AI assistant to install graphify, run the decision check, and
set up the basic workflow in a single step. It is ready to copy-paste as-is:

```text
Install graphify in this project.

Installation:
- Source: https://github.com/Graphify-Labs/graphify
- Command: graphify install --project --strict
- Also add graphify-out/ to .gitignore (if not already present) —
  otherwise every extract can bloat the prompt cache.

Decision criterion:
- Before installing, evaluate: if graphify doesn't help, or negatively affects my
  read/token/context usage, or makes things worse, DON'T install it and tell me why.
- Code requires no API key (local tree-sitter parse, no LLM); --code-only works fully
  offline. If that's the case, install it.

After installing:
1. Run the extraction and build the graph.
2. Verify strict mode: it should block the session's first raw source read and redirect to
   the graph, then revert to nudge mode (not get stuck).
3. Summarize this workflow for me: for code questions you'll use graphify
   query/path/explain first; you'll save every important query outcome as "useful" or
   "dead_end" via save-result; you'll run reflect to aggregate lessons and tag nodes as
   preferred/tentative.
4. After code changes, run graphify update . to keep the graph fresh (don't trust stale
   knowledge).
```

> 💡 This prompt packs every item from this guide into one message: the decision criterion,
> strict mode, offline install, the save-result/reflect loop, and the .gitignore warning.
