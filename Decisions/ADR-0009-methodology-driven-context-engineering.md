# ADR-0009: Methodology-Driven Context Engineering

- **Status**: Proposed
- **Date**: 2026-07-01
- **Deciders**: Wayne

## Context

Audit of the AI pipeline (2026-07-01) found the data model encodes the methodology well, but the pipeline flattens it into strings:

- The "5-layer truncation" has `TOTAL_BUDGET = 1_000_000` — effectively "dump everything that fits," no summarization tiers, no cost control
- Narrative memory when writing chapter N is **the last 500 characters of chapter N-1** (no chapter summaries, no retrieval of earlier prose)
- `searchRelevantContext()` is a stub returning `[]` while a working FTS5 SearchService sits unconnected
- Taxonomy semantics are discarded at injection: foreshadowing lifecycle (`plannedPayoff`, `term`, overdue-ness) never drives injection decisions; **three of five Wayne R1 fields never enter any prompt**; the timeline is never injected; no pacing instructions
- **The methodology itself never enters generation prompts**: Wayne Principles exist only as spec documents; `GeminiProvider` sends one flat `contents` string — no systemInstruction, no caching

ADR-0002 settled *who* assembles context (builder ownership — implemented, and inherited here); Issue #13 settles *how a single prompt is arranged*. Neither answers: what stays resident, what gets retrieved, and how the taxonomy drives it.

## Decision

Three principles, one architecture:

### 1. Prose is never injected wholesale — three context tiers

```
Tier 0  Resident (stable prefix, cacheable):  methodology pack (systemInstruction)
        + BibleDigest (all characters/relationships incl. full R1 fields/world/
        timeline/foreshadowing ledger + 1–2-line rolling summary per chapter)
Tier 1  Retrieved (per request):  taxonomy-rule retrieval (below) + last 1–3
        chapters full text + FTS5 recall of relevant historical scenes
Tier 2  Working (never cached):  current chapter + outline + user instruction
```

- Assembly order is fixed and serialization deterministic → free implicit-caching wins; explicit cache (bible-version key, EventBus invalidation) for long sessions
- Budgets: Tier 0 ≤ 250k tokens (digest degradation beyond), Tier 1 ≤ 50k
- Builders emit a provider-neutral `ContextPlan` (blocks with role/cacheable/stability), mapped per provider (Gemini cachedContent; future Anthropic `cache_control`)
- Cost validated: mature serial ≈ $2/chapter of AI spend under caching vs $4+ naive resend; full-prose injection (50万字 ≈ 750k tokens) rejected at $22+/session

### 2. Taxonomy is the index — retrieval priority: rules > FTS5 > vectors

Author-annotated structure outperforms embeddings because it expresses *schedules and obligations*, not similarity. A `RetrievalPlanner` (the real implementation behind the `searchRelevantContext` stub) applies SQL-implementable rules; abbreviated table:

| Rule | Trigger (writing chapter N) | Injection |
|---|---|---|
| F1 due | `plannedPayoff ∈ [N, N+5]` | instruction: "set up/trigger this payoff" |
| F2 overdue | active beyond term horizon (short>15ch …) | warning: hint or resolve |
| H1 hook | previous chapter's hook | "opening must answer this {style} hook" |
| R2 conflict | outline matches `disagreeScenarios` text | R1 card as *generation constraint* |
| T1 timeline | events sharing this chapter's characters | fact list (anti-continuity-error) |
| C1 arc phase | arc phase within ±10 chapters | "character is in transition X→Y" |
| P1 pacing | 3 consecutive high-tension chapters | "ease this chapter" |

FTS5 recalls named history (S1); sqlite-vec (M8) only backstops *unnamed* conceptual similarity. This inverts standard RAG priority — the differentiation. These same rules surface as ADR-0007's opinionated MCP queries (`bible_whats_due` = F1+F2+C1).

### 3. Methodology: one definition, three runtime forms

A single rule registry renders as: (a) **systemInstruction** at generation time (per-mode methodology pack: P1–P4, taxonomy semantics, conditional cards), (b) **QualityService checks** post-hoc (M7 Track A's 26 rules — same registry, so "what the prompt forbids" and "what the checker detects" can never drift), (c) **few-shot exemplars** (voiceSamples; violation/fix pairs per red-line rule).

### Rollout

- **Phase 1** (with M7 Track A; superset of Issue #13): semantic prompt sections, systemInstruction support in provider, methodology pack v1, fixed assembly order, rolling chapter summaries on `status→done`
- **Phase 2**: RetrievalPlanner + rule table; wire FTS5; inject full R1 fields/timeline/foreshadowing lifecycle
- **Phase 3** (with M8 Track A): BibleDigest builder + explicit cache lifecycle; sqlite-vec backstop; cost meter in UI

**ADR-0002 should be marked Accepted** — its builder-ownership decision is implemented in code and is inherited unchanged by this ADR.

## Alternatives Considered

### Alternative 1: Full-prose injection (1M window era, "just put it all in")
- **Pros**: Zero retrieval engineering.
- **Cons**: 200万字 ≈ 3M tokens exceeds the window; 50万字 fits but costs $1.1+/generation uncached; taxonomy semantics still lost.
- **Why not**: Economically unviable for BYOK users; bigger windows don't create *instructions* out of raw prose.

### Alternative 2: Embedding-first RAG (standard pattern)
- **Pros**: Uniform pipeline; well-trodden.
- **Cons**: Similarity cannot express "this foreshadow is due in 3 chapters" or "this companion hasn't disagreed in 30 chapters" — the exact signals our schema already holds.
- **Why not**: Throws away the product's core asset; vectors demoted to backstop.

### Alternative 3: Prompt-only fix (Issue #13 alone)
- **Pros**: Small.
- **Cons**: Rearranges a single prompt but leaves 500-char memory, dead R1 fields, absent methodology, and no caching untouched.
- **Why not**: Subsumed as Phase 1 of this ADR.

## Consequences

### Positive
- Methodology becomes runtime behavior ("长出牙齿"): same registry drives generation constraints, quality checks, and MCP opinionated queries
- Long-serial memory problem solved structurally (summaries + rules + recall), ~50–90% token-cost reduction via caching
- Provider-neutral `ContextPlan` unblocks multi-provider (incl. Ollama) cleanly

### Negative
- Rolling summaries add one small AI call per completed chapter (~$0.001)
- Rule registry becomes a new shared dependency between AI and Quality modules — needs owned tests
- Explicit cache lifecycle adds state to manage (invalidation via EventBus)

### Risks
- Rule thresholds (e.g., overdue horizons) are guesses until used: store as data, tune without releases
- Digest quality bounds everything resident: degradation rules (sampling voiceSamples, one-lining inactive characters) must be tested against large seeds

## Related

- [ADR-0002](ADR-0002-ai-context-injection-strategy.md) — builder ownership (inherited; recommend → Accepted)
- [ADR-0007](ADR-0007-product-repositioning-okb-dual-host.md) — opinionated queries share this rule registry
- GitHub Issue #13 — Phase 1 superset; M7.md Track A (QualityService), M8.md Track A (sqlite-vec)
- Modules/03_ai_service.md, Architecture/BusinessLogic/ContextInjection.md — to be revised on acceptance
