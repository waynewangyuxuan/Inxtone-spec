# ADR-0007: Product Repositioning — Opinionated Knowledge Base + Dual-Host Architecture

- **Status**: Proposed
- **Date**: 2026-07-01
- **Deciders**: Wayne

## Context

After a 4-month pause (2026-02 → 2026-07), a full audit + competitive research session surfaced three shifts:

1. **Agent ecosystem matured.** Claude Code plugin marketplace has 100+ official plugins with real distribution; MCP Apps became an official standard (2026-01). The fiction category on these marketplaces contains only trivial markdown-based projects (best: 8 stars) — no one has shipped a structured, opinionated story database as a plugin. The slot is empty.
2. **Generic agent memory is a crowded commodity.** mem0/Letta/built-in memories all converge on "flat facts + embedding retrieval" — value collapses into retrieval quality. Meanwhile our AI orchestration layer (~1.8k lines, the hardest code to maintain) competes head-on with what agent runtimes improve weekly.
3. **Our differentiators were static.** Wayne Principles, foreshadowing lifecycle, and the five-layer context existed as fill-in-forms and docs, not as active runtime behavior. Competitors' first interaction (conversational brainstorm, chat-with-your-book) was where we were absent.

## Decision

**Inxtone is an opinionated knowledge base (OKB) for fiction writing.** The taxonomy — foreshadowing with a plant→hint→payoff lifecycle, relationships with Wayne R1 fields, hooks with style/strength — *is* the product's opinion, and the opinion encodes craft expertise. Delivery moves to a **dual-host architecture**:

```
                ┌─ Host 1: Claude Code / agent CLIs (plugin: skills + MCP)   ← Phase 1
core (SQLite    │
 story bible ───┼─ Host 2: inxtone conversational CLI (Agent SDK / own loop) ← Phase 2
 + MCP tools)   │
                └─ Always: inxtone view — local webview codex ("书房/图鉴")
```

- **Phase 1 (4–6 weeks)**: `@inxtone/mcp` wraps the existing repositories/services as MCP tools; prompt templates + context layering become a skill pack; the web UI is trimmed to a read-mostly codex (`inxtone view`). The user's own agent is the brainstorm runtime; model billing is natively BYOK; data stays in local SQLite.
- **Phase 2 (after Phase 1 validates)**: `npx inxtone` ships its own conversational CLI over the *same* MCP tool layer (Agent SDK primary, provider-neutral loop w/ Ollama/Gemini as fallback), reaching writers without Claude Code.
- **Rejected: cloud runtime.** It voids local-first/own-your-data, converts BYOK into credit billing (i.e., becomes an unfunded Sudowrite), and open source + hosted inference is economically upside-down. Cloud remains admissible later only as *sync/share links* (sync ≠ runtime).

### MCP tool design principle: two tiers

1. **CRUD tier** — `bible_get/upsert_character`, `bible_add_foreshadowing`, `chapter_save`, … (anyone can wrap a DB; this is table stakes).
2. **Opinionated query tier** — the differentiation: `bible_whats_due` (foreshadowing/arc phases near chapter N), `bible_check_wayne_r1` (which companions are degrading into tools), `bible_find_tensions` (unexploited relationship conflicts).

### Why opinionated beats generic (the "negative space" argument)

A generic KB can only answer "what did you store." An opinionated schema knows *what is missing*: an empty `independentGoal`, a short-term foreshadow 30 chapters stale, three suspense hooks in a row. Each gap is a brainstorm prompt. **Inspiration is not retrieved from similarity; it grows out of structural holes and tensions.** Retrieval likewise becomes structural query ("what's due in chapter 40") — a schedule, not a similarity — which embeddings cannot express (see ADR-0009).

### Why plugin + prompting resolves the classic OKB flaw

Opinionated tools historically charged a learning tax (users must internalize the taxonomy first — Novelcrafter's "configuration hell"). In the plugin form, **the LLM is the impedance-matching layer**: the user chats freely; skills translate chatter into schema writes. The user never faces a form, yet the data lands structured. This is newly viable in 2026 and is the reason to move now.

### Positioning statements

- To writers: **"你的 Story Bible，现在活了。"**
- To the agent/developer audience: **"Your agent has memory. It doesn't have craft. Inxtone is an opinionated knowledge base for fiction — the taxonomy is the expertise."**

## Alternatives Considered

### Alternative 1: Strengthen status quo (full local app, multi-provider AI)
- **Pros**: 100% code reuse; zero architectural risk; local-first fully honored.
- **Cons**: End state is "CLI Novelcrafter" — same BYOK, same bible, same setup friction, minus browser reach; we keep maintaining 106 endpoints, a bespoke editor, and an AI orchestration layer that agent runtimes out-iterate weekly. No distribution story.
- **Why not**: Dies of obscurity, not infeasibility. "Small and special" fails: it is a general shape made small.

### Alternative 2: Cloud runtime + thin local client
- **Pros**: Reaches non-CLI users (e.g., 番茄 authors).
- **Cons**: Breaks local-first/own-your-data/BYOK; ~40% reuse (effectively a rewrite plus auth/billing/ops); competes with funded SaaS on their turf.
- **Why not**: The exact opposite of "small and special". Rejected outright.

### Alternative 3: Own conversational CLI only (skip plugin phase)
- **Pros**: Full experience control; no ecosystem dependency.
- **Cons**: 5–10k lines of new TUI/agent-loop work *before* any market validation; rebuilding an agent harness is the 2025–26 graveyard genre.
- **Why not**: Its tool layer is identical to the plugin's MCP layer — so it is Phase 2 of this ADR, not a competitor to it.

## Consequences

### Positive
- Smallest possible new code (~2k lines MCP + skill text) to a shippable, category-empty wedge.
- Deletes/freezes the hardest-to-maintain code (AI orchestration, editor arms race); core 34k lines fully retained.
- BYOK becomes purer (user's own agent subscription is the fuel); local SQLite untouched.
- Brainstorm-first onboarding ("丢给我一个脑洞") provided by the world's best conversational runtimes for free.
- M7 Track A (QualityService) and Track B (visualizations) both *gain* strategic value (rules = the KB's teeth; codex = the brand surface).

### Negative
- Phase 1 funnel excludes Chinese webnovel authors (no CLI, no API keys) — an explicit, accepted trade-off; that market is a separate later decision.
- Platform dependence on the Claude Code ecosystem during Phase 1; the webview codex is the only independent brand carrier and must be excellent.
- Web UI's editor/CRUD investment is partially sunk; further editor work frozen pending validation.

### Risks
- **Phase 1 validation fails** (nobody installs/returns). Mitigation: success metric defined up front — organic installs with second-session return; core survives intact for Phase 2 or fallback to Alternative 1.
- **Ecosystem policy shifts.** Mitigation: MCP is a multi-vendor standard; tool layer is host-agnostic by construction.

## Related

- [ADR-0008](ADR-0008-design-language-paper-ink-cinnabar.md) — Design language for the codex view
- [ADR-0009](ADR-0009-methodology-driven-context-engineering.md) — Context engineering; taxonomy-as-index
- [ADR-0003](ADR-0003-milestone-reorder-smart-intake.md) — "Schema stays opinionated" (this ADR extends that thesis to positioning)
- M7.md / M8.md — track priorities shift per Consequences
