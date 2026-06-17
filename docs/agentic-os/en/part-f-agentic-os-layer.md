# Part F — Agentic OS Layer

🌐 English · [Türkçe](../tr/part-f-agentic-os-layer.md)

*[← Part E](part-e-governance-evolution.md) · Next: [Part G — Protocols & Deliverables](part-g-protocols-deliverables.md)*

> **This Part is the L4→L5 difference.** The layer that turns individual agents into a real
> operating system: persistent knowledge, context economics, AI-feature engineering, and a
> human-approval model.

---

## F1. Agentic Knowledge & Memory Layer → `KNOWLEDGE.md` + glossary

**Problem:** The rules file must be short (Part A), but a large project has a lot of knowledge. If
the agent rediscovers everything from scratch each session, it burns time/tokens and produces
inconsistency.

**Solution — layered knowledge:**
```
1. Rules (CLAUDE.md/AGENTS.md)  → always loaded, short, "always true"
2. Live state (docs/agent/*)     → tasks/decisions/architecture, read on demand
3. Indexed knowledge (KNOWLEDGE.md) → repo map + glossary + "where to start" pointers
```

**`KNOWLEDGE.md` contents:**
- **Repo map:** what each directory does, entry points, "for X look here".
- **Ubiquitous-language glossary:** term → definition (all agents/models speak the same language).
- **Pointers:** where frequently needed info lives (DOMAIN_MODEL, ADRs, contract files).
- **Knowledge decay management:** aging/suspect info flagged `⚠️ stale?` → prevents drift (Part G).

> Model-agnostic: in frameworks using RAG/embeddings these files are indexed; in others the agent
> reads them directly. In both cases the source is the same files.

---

## F2. Cost & Context Economics

In a large project, token cost and context quality directly determine ROI. **Context is a budget.**

### Context budget principles
| Principle | Practice |
|---|---|
| Information density | Tables/checklists > long paragraphs; same info, fewer tokens |
| Lazy loading | Don't load procedures/knowledge until invoked (procedure layer) |
| Right granularity | Delegate to an isolated sub-task; keep the main context clean |
| Noise cleanup | Don't dump long logs/diffs into context; write to file, summarize |
| Frequent `/clear` | Start clean when a task is done; don't carry old context |

### Token cost & ROI
- **Prompt ROI = (verified value produced) / (tokens spent).** Low-ROI signals: re-discovery,
  bloated rules file, summary-less headless output (re-reading), big single prompt.
- **Cache awareness:** most APIs offer a prompt cache; a stable prefix (rules+system) is cached.
  Put frequently changing content at the end. Long waits cool the cache.
- **Model selection cost:** cheap model for routine work (formatting, simple transforms), strong
  model for reasoning. In orchestration, match the role to the model.
- **AI-feature cost (LLM in the product):** $/request is an **NFR** (Part B §B4); token budget
  enters acceptance criteria.

### File organization = context economics
A good `KNOWLEDGE.md` + short rules + fixed-format handoffs = fewer tokens per session, higher
hit rate. Bad organization = a re-discovery tax every session.

---

## F3. AI/ML & LLM-Feature Engineering

If the product *itself* contains an LLM/ML, normal tests are not enough.

| Topic | Practice |
|---|---|
| **Eval harness** | Golden set + threshold; every change passes the eval (Part D §D1 test type) |
| Regression guard | Does a new prompt/model break old goldens |
| Prompt/version registry | Prompts versioned, traceable (provenance) |
| Hallucination/grounding | Source-bound (RAG) verification; fabrication detection |
| **Prompt-injection threat model** | If untrusted input reaches the model, add to STRIDE (Part E §E5) |
| Data governance | PII in training/inference data; retention; opt-out |
| Cost/token budget | $/request NFR; budget-overrun alert (Part F §F2) |
| Model migration | On model change: eval + cost + behavior diff; rollback |

> Rule: **Verify an AI feature with an "eval," not a "test."** Non-deterministic output needs a
> deterministic gate: golden set + threshold + adversarial verify.

---

## F4. Human-in-the-Loop & Escalation Policy

Autonomy is not over-trusted; some decisions require human approval.

**Cases requiring an approval gate (example thresholds):**
```
- Irreversible action: prod migration, data deletion, publishing to an external service
- High impact: breaking a public API, auth/authz change, pricing/billing
- Security: new trust boundary, secret access, new dependency (E1)
- Uncertainty: proceeding before a risky assumption is validated
→ These are not "guided" by a prompt but "ask/deny" + human approval via gate/permission.
```

**Escalation model:** The agent stops at a blocker, makes no assumption; writes it to
`RISKS.md`/handoff and asks the human a precise question. "Make reasonable assumptions" is used
only for low-risk, reversible work and **with every assumption logged.**

**Provenance link (Part E §E3):** every change that required approval is traced with who approved
+ which handoff.

---

*Next: [Part G — Protocols & Deliverables](part-g-protocols-deliverables.md)*
