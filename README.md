# Agentic Software Engineering Operating System (Agentic-OS)

> A **model-agnostic** operating framework for running large-scale software projects with LLM
> agents — Claude, GPT, Gemini, Qwen, DeepSeek, or any open-source agent framework — with **low
> ambiguity** and **high verifiability**.

🌐 **Languages / Diller:** [English](docs/agentic-os/en/INDEX.md) · [Türkçe](docs/agentic-os/tr/INDEX.md)

This is **not** a "Claude Code usage manual." Claude Code is treated as *one example
implementation* of the concepts here. The backbone is independent of any model or tool: you can
swap the mechanisms (skills, hooks, permissions) for equivalents in your own stack and the
framework still holds.

---

## Why this exists

LLM agents are extraordinarily capable but **stateless** — every session is a fresh "shift" with
no memory of the last one. Left unmanaged, this produces predictable failures at scale: building
the wrong thing, unverifiable "done" claims, context poisoning, contract drift, duplicated work,
and runaway token cost.

Agentic-OS addresses these by treating agent execution as an **engineering discipline**, not a
chat. It externalizes memory to files, separates *guidance* from *enforcement*, and wraps every
mature engineering process (discovery, domain modeling, testing, release, observability,
security, traceability) into a repeatable structure that any LLM can follow.

---

## Core philosophy (4 principles)

1. **Persistent memory = files, not context.** The agent is stateless; real memory lives in
   repository files (`docs/agent/*`). This is true regardless of which model you use.
2. **The Three-Part Transform.** Every mature engineering process is expressed as three parts:
   **Artifact** (a filled-in file = a contract) + **Procedure** (a prompt/skill = how to do it) +
   **Gate** (a hook/CI check = cannot be skipped). If a process doesn't map to all three, it's
   incomplete.
3. **Guidance ≠ Enforcement.** Guidance is flexible and the model *can* ignore it (rules,
   prompts). Enforcement is deterministic and the model *cannot* bypass it (CI, hooks, sandbox).
   Everything critical is enforced, not merely requested.
4. **Verify first.** A "works / passing" claim is never accepted without evidence (the command
   plus its raw output).

---

## Model-agnostic mechanism map

The framework depends on the **left column** (the concepts). The remaining columns are
interchangeable implementations.

| Backbone concept | Claude Code | GPT / OpenAI | Gemini | OSS agent framework |
|---|---|---|---|---|
| Persistent rules | `CLAUDE.md` | `AGENTS.md` / system prompt | system instruction | rules / system prompt |
| Reusable procedure | Skill / slash command | saved prompt / custom GPT / function | gem / template | tool / agent template |
| Deterministic gate | Hook (`exit 2`) | CI / pre-commit | CI | callback / middleware / CI |
| Access control | Permissions | tool allowlist / sandbox | function allowlist | guardrail / policy |
| Isolated review | `context: fork` / `claude -p` | separate thread / agent | separate session | sub-agent / fresh context |
| File-based memory | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` (identical everywhere) |

**Takeaway:** file-based memory and contracts behave *identically* across every model — only the
procedure and gate tooling changes. That's why the backbone is anchored to files, not to any
vendor's feature set.

---

## Repository structure

The guide is split into seven Parts, available in **English** (`docs/agentic-os/en/`) and
**Türkçe** (`docs/agentic-os/tr/`). English links below:

| Part | Title | Problem it solves |
|---|---|---|
| [INDEX](docs/agentic-os/en/INDEX.md) | Entry point + mechanism map | How to use the guide (humans **and** LLMs) |
| [A](docs/agentic-os/en/part-a-foundations.md) | **Foundations** | How to think about agents; control plane; the maturity model |
| [B](docs/agentic-os/en/part-b-discovery-definition.md) | **Discovery & Definition** | Prevents building the wrong thing; pins down the *what* |
| [C](docs/agentic-os/en/part-c-execution.md) | **Execution** | Task decomposition, orchestration patterns, handoffs, workflow |
| [D](docs/agentic-os/en/part-d-quality-operability.md) | **Quality & Operability** | Test strategy, project-type gates, observability, release |
| [E](docs/agentic-os/en/part-e-governance-evolution.md) | **Governance & Evolution** | Dependency/refactor policy, traceability, security |
| [F](docs/agentic-os/en/part-f-agentic-os-layer.md) | **Agentic OS Layer** | Knowledge layer, cost/context economics, AI features, human-in-the-loop |
| [G](docs/agentic-os/en/part-g-protocols-deliverables.md) | **Protocols & Deliverables** | LLM interview, final-prompt protocol, failure modes, archetypes, checklists |

> 🇹🇷 Türkçe sürüm birebir aynı yapıdadır: [`docs/agentic-os/tr/INDEX.md`](docs/agentic-os/tr/INDEX.md)

### Two folders — don't confuse them
- **`docs/agentic-os/`** = *this guide* (reference material; changes rarely; generic; bilingual).
- **`docs/agent/`** = the *runtime control plane* (project-specific live artifacts: `TASKS.md`,
  `DECISIONS.md`, `ARCHITECTURE.md`, `handoffs/`, …). Filled in per project. The guide tells you
  *how* to fill it; the control plane is the filled-in result.

---

## The Agent Maturity Model

Know where you are; advance one level at a time (each level assumes the one below it).

| Level | Name | What you have | Typical limit |
|---|---|---|---|
| **L0** | Single prompt | "Do this" | No memory, repeated work, no verification |
| **L1** | Persistent rules | `CLAUDE.md`/`AGENTS.md` + basic file memory | Rules can be ignored; no procedures |
| **L2** | Procedures | Skills / saved prompts; fixed-format output | Still guidance; nothing enforced |
| **L3** | Enforcement | Hooks + CI + permissions; gates | Single-agent; no orchestration |
| **L4** | Orchestration | Multi-agent: pipeline, fan-out, isolated review/QA | Knowledge/memory still ad-hoc |
| **L5** | Agentic OS | Knowledge layer + context economics + traceability + human-in-the-loop + archetypes | — (continuous improvement) |

Small internal tools can comfortably stop at L2–L3. Large SaaS/platform work targets L4–L5.

---

## How to use this guide

### For humans — reading order
`INDEX → A → B → … → G`. First time? **Part A → Part B → Part G** is enough; open the rest as needed.

### For LLMs — operating protocol
If a user hands you this guide while discussing a project:
1. **Locate** the project archetype and maturity level (Part G §Archetypes, Part A §Maturity).
2. **Gather** missing information — never assume. Follow the **LLM Interview Protocol** (Part G).
3. **Fill the discovery artifacts** together (Part B). No planning until Definition of Ready passes.
4. **Decompose** into a task graph with assignments (Part C).
5. **Attach** acceptance criteria and a test strategy (Part D).
6. **Emit** a low-ambiguity, traceable implementation prompt — the **Final Prompt Generation
   Protocol (7-step)** in Part G.

> Prompt engineering is **not** a separate topic here. It's a natural part of the
> Discovery → Planning → Execution → Review → Handoff chain, embedded in each Part.

---

## Quick start

```bash
# Read the entry point (English or Turkish)
open docs/agentic-os/en/INDEX.md     # or docs/agentic-os/tr/INDEX.md — or browse on GitHub

# Adopt it in your own project: create the runtime control plane
mkdir -p docs/agent/handoffs
# then fill in PRODUCT_BRIEF.md, DOMAIN_MODEL.md, TASKS.md, DECISIONS.md … (see Part B & INDEX §5)
```

→ **Start here: [English INDEX](docs/agentic-os/en/INDEX.md) · [Türkçe INDEX](docs/agentic-os/tr/INDEX.md)**

---

## Status & contributing

This guide is intended to be a **living, reusable reference** that grows over time. Issues and
pull requests that sharpen the operational guidance, add archetypes, or improve the
model-agnostic mapping are welcome. Please keep the English and Turkish versions in sync.

## License

[MIT](LICENSE) — free to use, copy, modify, and distribute with attribution.
