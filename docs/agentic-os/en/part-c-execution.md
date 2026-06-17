# Part C — Execution

🌐 English · [Türkçe](../tr/part-c-execution.md)

*[← Part B](part-b-discovery-definition.md) · Next: [Part D — Quality & Operability](part-d-quality-operability.md)*

> **Prerequisite:** Definition of Ready (Part B §B5) must have passed.

---

## C1. Agent / Task Decomposition Model

**Choosing the working mode (model-agnostic):**
- **Same session/context:** tightly coupled, touching the same files, small-to-medium work.
- **Isolated context (fork / new thread / sub-agent):** work needing a perspective independent of
  the implementer's blind spots — review, QA.
- **Separate headless process (`claude -p` / batch API):** independent, parallel work with a clear
  input/output contract.

> **Golden rule:** An agent that wrote the code **cannot** do its own review/QA. Always isolate.

| Role | Purpose | Input contract | Output contract | Working mode |
|---|---|---|---|---|
| Product/PM | Scope & criteria | PRODUCT_BRIEF | TASKS epic + acceptance | Single session (upfront) |
| Architect | Design & decisions | Brief, domain, code | ARCHITECTURE + ADR | Single session / plan mode |
| Backend | API/business logic | Implementation brief | Code + tests + handoff | Single session, branch |
| Frontend/Mobile | UI | Brief + API contract + UX_SPEC | Code + tests + handoff | Separate session, branch |
| Reviewer (security/code) | Vuln & quality review | Diff + standards | Severity-tagged report | **ISOLATED** |
| QA/verification | Evidence-based verification | Diff + TEST_PLAN | Pass/fail report | **ISOLATED** |
| Documentation | Documentation | Final diff + handoff | Docs/README diff | Separate headless |
| Refactor | Cleanup (behavior unchanged) | Diff | Findings | Isolated |

**Spike role:** If uncertainty is high, run a *timeboxed research task* before implementation.
Its output is not code but a decision/recommendation (a draft ADR). It closes risky assumptions cheaply.

**Task graph & critical path:** Lay out tasks with their dependencies. Start critical-path items
first and/or in parallel. Mark dependencies in TASKS.md (`T-103 (blocked: T-102)`).

**Preventing collision/duplication/context loss:**
- **TASKS.md = the lock:** whoever starts marks status `[>]` + owner. Two agents don't enter the same `[ ]`.
- **File ownership:** each agent writes to its own directory; don't have parallel agents write the same file.
- **Shared contract = interface file:** one writes it, the other reads it.
- **Every agent ends with a handoff;** the next agent feeds from that file, not from context.

---

## C2. Agent Orchestration Patterns

Beyond the single agent. In large work, orchestration determines quality.

| Pattern | When | How |
|---|---|---|
| **Pipeline** | Multi-stage, independent per-item chain | item → stage1 → stage2 …; no barrier between stages |
| **Fan-out / parallel** | Independent jobs at once | N agents in parallel, collect at the end (barrier) |
| **Judge panel / adversarial verify** | A finding/decision to verify | N independent verifiers, each tries to *refute*; majority decides |
| **Loop-until-dry** | Unknown-size discovery (bugs, edge cases) | Repeat until K rounds yield nothing new |
| **Completeness critic** | "What's missing?" | Final agent: unrun modality, unverified claim, unread source |

**Adversarial verify (the most critical quality pattern):** For every important finding/claim, an
independent verifier — *not the author* — reviews it with the instruction "refute this — if
unsure, reject." It eliminates plausible-but-wrong outputs. Mandatory in security and AI-feature
work (Part E, F).

> **Claude Code example:** Workflow/Task tools for orchestration; `context: fork` or parallel
> `claude -p` processes for isolated review. In another framework: a graph executor / sub-agent
> fan-out. The backbone is the same: **independent verification + file-based collection.**

---

## C3. Handoff & Contract System

Agent outputs are written as contracts into `docs/agent/handoffs/`. Fillable templates:

### Implementation Brief
```markdown
# Brief: <T-xxx>
## Goal · Scope DO · Scope DON'T
## Inputs/references: <ADR, DOMAIN_MODEL §, contract file, UX_SPEC §>
## Acceptance criteria: [ ] … [ ] test/lint green
## Constraints: <no new dependency / etc.>
```

### Handoff Summary (the most important contract)
```markdown
# Handoff: <T-xxx> (<source> → <target>)
## Status: DONE|PARTIAL|BLOCKED — branch, commit <hash>
## What was done · Public API/contract (signatures, error types)
## Decisions (ADR ref) · Test status (command + result)
## Open risks & assumptions
## For the next agent: <one clear instruction / review focus>
```

### Other templates
- **Task Plan** (DO/DON'T, file order, test strategy, risk, acceptance; no implement without approval)
- **Security Review Report** (summary severity, finding = location+impact+PoC status+fail-closed fix) → Part E
- **QA Verification Report** (PASS/FAIL, gates command+output, acceptance item+evidence) → Part D
- **ADR** (context, decision, alternatives, outcome) → Part E
- **Next-agent prompt** (role+input file+precise task+out-of-scope+output format+constraint)

**Minimum info to carry (the essence of a handoff):** branch+hash · public contract · decisions
(ADR) · test status+repro command · open risk+assumption · one clear instruction.

---

## C4. Recommended Workflow (end-to-end, model-agnostic)

| Stage | Procedure | Reads | Writes | Gate / output |
|---|---|---|---|---|
| Discovery | (Part B patterns) | — | PRODUCT_BRIEF, DOMAIN_MODEL, UX_SPEC, RISKS | **DoR** |
| Architecture | architect (plan mode) | brief, domain | ARCHITECTURE, ADR | Design approved |
| Task planning | task-plan | ARCH, TASKS | plan (approval) | No progress without approval |
| Implementation | implement-small-loop | brief, plan, contract | src, tests | Each step tested |
| Testing | (within loop) | — | — | A step doesn't close until green |
| Review (code/security) | review (ISOLATED) | diff, standards | handoffs/ | Critical/High = 0 |
| QA verification | qa-verify (ISOLATED) | TEST_PLAN, diff | handoffs/ | PASS, with evidence |
| Documentation | doc agent (headless) | handoff, code | docs, README | Diff |
| Release gate | CI + ORR | — | — | see Part D |
| Session cleanup | handoff + commit + clear | git | handoffs/, TASKS | Handoff written, status current |

**Prompt engineering lives inside this flow:** each stage's prompt has role+context(file ref)+
precise task+DO/DON'T+acceptance+output format+constraint. No vague verbs.

---

## C5. Headless / `claude -p` Usage Model

**When headless:** Isolated/parallel work with a clear input-output contract that can be automated
(independent review, QA, documentation). **When interactive:** exploration, plan discussion,
high-uncertainty work.

**Rules:** Always redirect output to a file (`> docs/agent/handoffs/...`). Produce fixed-format
output (HEADLINE + HANDOFF blocks). The next step feeds from the file, not from context.

```bash
claude -p "/analyze-readonly src/auth" --permission-mode plan > docs/agent/handoffs/auth-analysis.md
claude -p "/security-review feat/x"  > docs/agent/handoffs/x-sec.md &
claude -p "/qa-verify T-102"          > docs/agent/handoffs/x-qa.md  & wait
```
> Equivalent in other models: OpenAI/Gemini batch or CLI + script; OSS framework headless runner.

---

## C6. Automation Strategy

- **Don't embed prompts in the rules file** (every token loads every turn → bloat). Make a
  repeated prompt a **procedure** (skill/saved-prompt) (lazy load + fixed format).
- **Hook/CI** = un-skippable gate (test/lint/build, dangerous command). **Permission** = access
  boundary (push/secret/outbound). **Output style/role** = minimal, a single discipline tone.
- General rule: distribute every mature process into **Artifact + Procedure + Gate**.

---

*Next: [Part D — Quality & Operability](part-d-quality-operability.md)*
