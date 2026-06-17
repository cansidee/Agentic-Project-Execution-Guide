# Part A — Foundations

🌐 English · [Türkçe](../tr/part-a-foundations.md)

*[← INDEX](INDEX.md) · Next: [Part B — Discovery & Definition](part-b-discovery-definition.md)*

---

## A1. Mental Model

Think of the LLM agent as a **very capable but stateless senior engineer with a narrow working
memory.** Each session is a "shift." When the shift ends, its memory is wiped; only what was
written to disk survives. This holds regardless of Claude/GPT/Gemini/OSS.

| Concept | Meaning | Operational consequence |
|---|---|---|
| **Context** | Instant working memory (prompt + files read + tool output) | Limited resource; don't fill with noise |
| **Memory** | Auto-loaded persistent rules (CLAUDE.md/AGENTS.md) | Keep short; every token loads every turn |
| **Session** | A shift; context is ephemeral | Clear when the task is done (`/clear` etc.) |
| **File persistence** | What's written to disk = real persistent memory | Everything critical goes to disk |
| **Agentic loop** | Read→Plan→Change→Test→Observe→Fix | Each turn must stay small |

**Which info goes to disk vs stays in context:**

| To disk (persistent) | In context (ephemeral) |
|---|---|
| Decisions, task status, architecture | Intermediate reasoning |
| API/contract signatures | Dead-end attempts |
| Handoff/security/test reports | Temporary debug logs, one-off greps |

---

## A2. Core Principles

1. **Persistent memory = files.** A decision not written to disk doesn't count.
2. **Small verifiable loop.** A big single prompt = an unverified huge diff = technical debt.
3. **Fixed cycle:** `Discovery → Plan → Implement → Test → Verify → Handoff`.
4. **Verify first.** No "done" without evidence (command + raw output).
5. **Small tasks.** Each task: one responsibility, one branch, small commits.
6. **Three-Part Transform.** Every process = Artifact + Procedure + Gate.

---

## A3. Agent Maturity Model

Know where you are; advance to the next level. Don't skip — each level assumes the one below.

| Level | Name | What you have | Limit / risk | Transition signal |
|---|---|---|---|---|
| **L0** | Single prompt | "Do this" | No memory, repeated work, no verification | You keep re-typing the same context |
| **L1** | Persistent rules | CLAUDE.md/AGENTS.md + basic file memory | Rules can be ignored; no procedures | You keep pasting the same multi-step prompt |
| **L2** | Procedures | Skills / saved prompts; fixed-format output | Still guidance; nothing enforced | "It forgot to run tests" happens |
| **L3** | Enforcement | Hooks + CI + permissions; gates | Single-agent; no orchestration | You need parallel/isolated work |
| **L4** | Orchestration | Multi-agent: pipeline, fan-out, isolated review/QA | Knowledge/memory ad-hoc; no economics | Context bloats, cost/drift rises |
| **L5** | Agentic OS | Knowledge layer + context economics + traceability + HITL + archetypes | — (continuous improvement) | — |

**Usage:** A small internal tool can stay at L2–L3. Large SaaS/platform work targets L4–L5.
Choose your maturity level together with the archetype from the INDEX (Part G).

---

## A4. Project Control Plane (model-agnostic)

Two layers: **rules (memory)** and **live artifacts (docs/agent/)**.

### Rules file (CLAUDE.md / AGENTS.md / system prompt)
- **Purpose:** Persistent rules & facts auto-loaded every session.
- **Write:** Working agreement, build/test commands, boundaries, list of procedures to use.
- **Don't write:** Long procedures, temporary notes, architecture essays. **< ~150 lines.**
- **Best practice:** Move procedures to the procedure layer (skill/saved-prompt).

### Live artifacts — `docs/agent/`
Each file: purpose / when / what to write / what NOT to write.

| File | Purpose | Don't write |
|---|---|---|
| `ARCHITECTURE.md` | Components, responsibilities, data flow, trust boundaries | Temporary exploration notes |
| `TASKS.md` | Live task status (`[x]/[>]/[ ]` + owner + commit) | Don't delete finished work; keep history |
| `DECISIONS.md` | ADR; decisions expensive to reverse | Trivial preferences |
| `TEST_PLAN.md` | Test strategy + acceptance + commands | — |
| `handoffs/*` | Inter-agent handoff contract | Ephemeral reasoning |

> **Claude Code example:** `.claude/settings.json` (permissions+hooks, committed),
> `.claude/settings.local.json` (personal, gitignored), `.claude/skills/*`, `.claude/hooks/*`,
> `.claude/output-styles/*`. **These are an L3 implementation, not the backbone.** In another
> tool you'd build the same outcome with CI + pre-commit + saved prompts.

---

## A5. Tooling Separation (guidance vs enforcement)

| Mechanism class | Class | When loaded | Skippable? | Right use | Anti-pattern |
|---|---|---|---|---|---|
| Rules file | Guidance | Every session | Yes | Short rules | Bloating it, embedding procedures |
| Procedure (skill/prompt) | Guidance | When invoked | Applied if invoked | Repeated workflow | Embedding in the rules file |
| Gate (hook/CI) | **Enforcement** | On event/PR | **No** | test/lint/build, command blocking | Heavy tests on every edit |
| Access control (permission/sandbox) | **Enforcement** | Every session | No | push/secret/outbound blocking | Saying "don't" in a prompt |
| Role/tone (output style) | Guidance | System prompt | Tendency | A single discipline tone | Putting everything here |

**Decision rule:** Repeated procedure→procedure; always-true fact→rules file; un-skippable
gate→gate; access boundary→permission. Re-apply this distribution for every critical process.

---

*Next: [Part B — Discovery & Definition](part-b-discovery-definition.md)*
