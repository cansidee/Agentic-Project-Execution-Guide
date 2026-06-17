# Agentic Software Engineering Operating System (Agentic-OS)

🌐 **Language:** English · [Türkçe](../tr/INDEX.md)

> **In one sentence:** A **model-agnostic** operating framework for running large-scale software
> projects with LLM agents (Claude, GPT, Gemini, Qwen, DeepSeek, OSS agent frameworks) with low
> ambiguity and high verifiability.

This is not a "Claude Code usage manual." Claude Code is **one example implementation** of the
concepts here. The backbone is model-independent; you can replace the mechanisms (skills, hooks,
permissions) with other tools.

---

## 0. Core Philosophy (the 4 principles under everything)

1. **Persistent memory = files, not context.** The agent is stateless; real memory lives in
   repository files (`docs/agent/*`). This holds *whatever model you use*.
2. **The Three-Part Transform.** Every process from mature engineering is expressed as three
   parts: **Artifact** (a filled-in file = contract) + **Procedure** (prompt/skill = how it's
   done) + **Gate** (hook/CI = a check that can't be skipped). If a process doesn't map to all
   three, it's incomplete.
3. **Guidance ≠ Enforcement.** Guidance is flexible; the model can ignore it (prompts/rules).
   Enforcement is deterministic; the model cannot bypass it (CI/hook/sandbox). Everything
   critical is enforced.
4. **Verify first.** A "works / passing" claim is not accepted without evidence (command + raw
   output).

---

## 1. Model-Agnostic Mechanism Map

The backbone concepts and their equivalents in different tools. **The guide relies on the left
column; the right columns are interchangeable.**

| Backbone concept (model-agnostic) | Claude Code | GPT / OpenAI | Gemini | OSS agent framework |
|---|---|---|---|---|
| Persistent rules | `CLAUDE.md` | `AGENTS.md` / system prompt | system instruction | rules / system prompt |
| Reusable procedure | Skill / slash command | saved prompt / custom GPT / function | gem / prompt template | tool / agent template |
| Deterministic gate | Hook (`exit 2`) | CI / pre-commit | CI | callback / middleware / CI |
| Access control | Permissions | tool allowlist / sandbox | function allowlist | guardrail / policy |
| Isolated review | `context: fork` / `claude -p` | separate thread / separate agent | separate session | sub-agent / fresh context |
| File-based memory | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` (identical everywhere) |

> **Implication:** File-based memory and contracts work the **same** across every model. Only the
> "procedure" and "gate" layer's tooling changes. That's why we anchor the backbone to files, not
> to them.

---

## 2. How to Use This Guide

### For humans — reading order
1. **Part A — Foundations**: mental model, control plane, determine your maturity level.
2. **Part B — Discovery & Definition**: lock down *what* you'll build before any code.
3. **Part C — Execution**: agent decomposition, orchestration, handoff, workflow.
4. **Part D — Quality & Operability**: test strategy, type-specific gates, observability, release.
5. **Part E — Governance & Evolution**: dependency/refactor policy, traceability, security.
6. **Part F — Agentic OS Layer**: knowledge/memory layer, context economics, AI features, HITL.
7. **Part G — Protocols & Deliverables**: interview protocol, final-prompt generation, failure
   modes, archetype templates, checklists.

First time? **Part A → Part B → Part G** is enough. Open the rest as needed.

### For LLMs — operating protocol
If you're working with a user who handed you this guide:
1. **Locate** maturity and archetype first (Part A §Maturity Model, Part G §Archetypes).
2. **Gather** missing information — don't assume. See **Part G — LLM Interview Protocol**.
3. **Fill the discovery artifacts** together (Part B). No planning until Definition of Ready passes.
4. **Produce task decomposition + agent assignment** (Part C).
5. **Add acceptance criteria + test strategy** (Part D).
6. **Emit the final implementation prompt** — **Part G — Final Prompt Generation Protocol (7-step)**.
7. Your output must be *low-ambiguity and traceable*; no vague verbs, measurable goals required.

> **Prompt engineering is not a separate topic.** It is a natural part of the
> Discovery → Planning → Execution → Review → Handoff chain. Prompt patterns for each phase are
> embedded inside the relevant Part; don't look for a standalone "prompt section."

---

## 3. Map of the Parts (what each is for)

| Part | Title | Problem it solves |
|---|---|---|
| **A** | Foundations | How to think about the agent; the control plane; maturity level |
| **B** | Discovery & Definition | Prevents building the wrong thing; pins down the *what* |
| **C** | Execution | Splits big work into agents, orchestrates, hands off |
| **D** | Quality & Operability | Defines "done"; test/perf/observability/release |
| **E** | Governance & Evolution | Dependency/refactor/traceability/security discipline |
| **F** | Agentic OS Layer | Knowledge/memory, context economics, AI features, human approval |
| **G** | Protocols & Deliverables | LLM interview, final-prompt generation, failure modes, archetypes |

---

## 4. Two Distinct Folders — Don't Mix Them

- **`docs/agentic-os/`** = **this guide** (reference, rarely changes, generic).
- **`docs/agent/`** = the **runtime control plane** (project-specific live artifacts:
  TASKS.md, DECISIONS.md, ARCHITECTURE.md, handoffs/ …). Filled in per project.

The guide (`agentic-os`) tells you *how* to fill it; the control plane (`agent`) is the filled-in form.

---

## 5. Artifact Registry (the contracts produced throughout the guide)

| Artifact | Part | Purpose |
|---|---|---|
| `PRODUCT_BRIEF.md` | B | Problem, segment, MVP, FR/NFR, success metric |
| `DOMAIN_MODEL.md` | B | Entity, state machine, contract, migration |
| `UX_SPEC.md` | B | Journey, screen, state, a11y (UI projects) |
| `RISKS.md` (RAID) | B | Risk/assumption/issue/dependency tracking |
| `ARCHITECTURE.md` | C | Component, boundary, data flow |
| `TASKS.md` | C | Live task status + ownership (locking mechanism) |
| `DECISIONS.md` (ADR) | C/E | Decisions that are expensive to reverse |
| `handoffs/*` | C | Inter-agent handoff contracts |
| `TEST_PLAN.md` | D | Test strategy + acceptance |
| `RUNBOOK.md` | D | Operations/incident readiness |
| `TRACEABILITY.md` | E | Requirement→Task→Test→Code matrix |
| `KNOWLEDGE.md` / glossary | F | Repo map, ubiquitous language |

---

*Next: [Part A — Foundations](part-a-foundations.md)*
