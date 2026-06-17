# Part G — Protocols & Deliverables

🌐 English · [Türkçe](../tr/part-g-protocols-deliverables.md)

*[← Part F](part-f-agentic-os-layer.md) · [INDEX](INDEX.md)*

> **This Part is the peak:** the standard protocols from Discovery to the final implementation
> prompt + common failure modes + ready archetype templates + checklists. Using this Part, an LLM
> guides the user with the right questions and produces a **low-ambiguity, traceable** prompt for
> Claude Code (or another agent).

---

## G1. LLM Interview Protocol

The information-gathering protocol an LLM follows when helping a user with this guide. **Don't
assume; ask for what's missing.**

**Step 1 — Position:** What is the archetype (G4) and maturity level (Part A §A3)?

**Step 2 — Scope questions (Discovery, Part B):**
```
- Problem/goal; how is success measured (metric)?
- User segments; the most critical journey?
- What's IN the MVP, what's explicitly OUT?
- Stack, existing codebase, constraints?
- NFRs: performance/scale/cost/compliance targets?
- Data & state: which entities, which transitions, which contract?
- Security class: PII/secret? Is auth/authz changing? (if so, extra gate)
- Existing test/CI/observability infrastructure?
- Time/risk tolerance: fast MVP or audit-critical?
```

**Step 3 — Flag the gaps:** Which questions are unanswered? These must be resolved before going
to Claude (open questions → `PRODUCT_BRIEF`).

---

## G2. Final Prompt Generation Protocol (7-step)

The standard flow that turns discovery output into an executable, low-ambiguity implementation
prompt:

```
1. GATHER      → Collect gaps from Part B artifacts (BRIEF/DOMAIN/UX/RISKS); ask via G1.
2. ASSUMPTIONS → Surface implicit assumptions; assign each a validation method + status.
3. RISKS       → Risk register (probability×impact); riskiest item → a spike task.
4. DECOMPOSE   → Task graph (dependencies + critical path); did each task pass DoR (Part B §B5).
5. ASSIGN      → Assign to Part C roles (session/isolated/headless); clarify file ownership.
6. CRITERIA    → For each task, acceptance criteria + test strategy (Part D) + DoD.
7. EMIT        → Produce the final prompt (template below). Traceability: each task tied to a Req.
```

**Final Implementation Prompt template (output):**
```markdown
# Implementation Prompt: <T-xxx / feature>
## Role: <backend/frontend/...>
## Context (read): PRODUCT_BRIEF §, DOMAIN_MODEL §, ARCHITECTURE §, contract file, relevant ADR
## Task (measurable, no vague verbs):
## Scope DO / DON'T:
## Acceptance criteria: [ ] ... [ ] test/lint/build green [ ] observability added
## Test strategy: <which test types — Part D §D1>
## Constraints: <dependency, secret, fail-closed, performance NFR>
## Workflow: plan first (wait for approval) → small loop → handoff. Isolated review+QA after.
## Traceability: this task satisfies <FR-xx/NFR-xx>.
```

> **Prompt engineering is not a separate topic:** this protocol is the natural last step of the
> Discovery→Planning→Execution→Review→Handoff chain.

---

## G3. Failure Modes (common failures + early detection)

Where agentic systems most often break. For each: an **early detection signal** and a **defense.**

| Failure mode | Symptom / early signal | Defense (Artifact+Procedure+Gate) |
|---|---|---|
| **Wrong requirement** | "I didn't ask for this" at the demo | DoR + product validation (Part B); success metric |
| **Wrong decomposition** | Tasks keep blocking each other, re-split | Task graph + critical path (Part C); spike |
| **Context poisoning** | Agent relies on a stale decision/wrong info | `/clear` + reload from file; KNOWLEDGE.md stale flag |
| **Stale documentation** | Doc contradicts the code | Knowledge decay flag (Part F); doc-update gate; single source of truth |
| **Contract drift** | Consumer breaks, "but it worked for me" | Contract freeze + contract test (Part D); single contract file |
| **Orphan task** | `[>]` in TASKS.md ownerless for ages | TASKS review; every `[>]` has owner+date; WIP limit |
| **Hidden dependency** | "Changing this blew up that" (invisible coupling) | Fitness functions (Part E); dependency map; integration test |
| **Fake completion** | "Done, tests pass" — but no evidence | Verify first; isolated QA + command/output evidence; Stop/CI gate |
| **Review blindness** | The code's author reviewed their own code | Isolated review (Part C §C2); adversarial verify |
| **Scope creep** | Diff exceeds the brief's DON'Ts | Explicit OUT-OF-SCOPE in brief; PR-task match |
| **Token blowup / low ROI** | Cost rising, progress slow | Context economics (Part F §F2); lazy load; `/clear` |

---

## G4. Project Archetypes (ready starter templates)

Each archetype: recommended agent setup · quality gates · workflow emphasis · file structure.

### SaaS Platform
- **Agents:** PM, Architect, Backend, Frontend, Reviewer(security+code), QA, Docs.
- **Gates:** unit+integration+e2e+contract; tenant isolation; canary+rollback; Core Web Vitals.
- **Workflow:** full Discovery→...→Release; feature flags + progressive delivery essential.
- **Files:** all `docs/agent/*` + UX_SPEC + RUNBOOK + TRACEABILITY.

### Internal Tooling
- **Agents:** Backend(+light frontend), QA(light). One developer often carries most roles.
- **Gates:** unit+smoke; least-privilege; audit log. Fast iterate, minimal gate.
- **Workflow:** L2–L3 is enough; no heavy ceremony.
- **Files:** CLAUDE.md + TASKS + DECISIONS + minimal TEST_PLAN.

### Security Tooling
- **Agents:** + **isolated** Security Reviewer (mandatory), Threat-model agent.
- **Gates:** + SAST/secret-scan/semgrep; fuzz/property test; extra gate on auth change; evidence mandatory.
- **Workflow:** Security Deep-Dive (Part E §E5.2); reviewer never the same session.
- **Files:** + THREAT_MODEL.md (mandatory).

### AI Application
- **Agents:** + ML/Prompt engineer, Eval agent (isolated).
- **Gates:** eval set + threshold; prompt-injection threat model; $/req NFR; regression guard.
- **Workflow:** **eval** instead of test; model migration procedure (Part F §F3).
- **Files:** + EVAL_SET, prompt registry, model card.

### Data Platform
- **Agents:** Data engineer, Pipeline reviewer, QA(data-quality).
- **Gates:** data-quality + pipeline test; schema-compat; idempotency; freshness SLA; lineage.
- **Workflow:** migration/backfill safety central; access control + retention (Part E).
- **Files:** + schema registry, lineage doc.

### Mobile Application
- **Agents:** Mobile, Backend(API), QA(device matrix), UX.
- **Gates:** unit+UI+device matrix; cold start/memory/battery; secure storage; staged rollout.
- **Workflow:** UX_SPEC mandatory; store review + staged rollout release gate.
- **Files:** + UX_SPEC, device matrix.

### OSS Project
- **Agents:** Maintainer, Reviewer, Docs. External contribution flow.
- **Gates:** CI matrix; semver+CHANGELOG; SECURITY.md; supply-chain + signed release.
- **Workflow:** contribution guide; public API deprecation discipline (Part D §D4).
- **Files:** + CONTRIBUTING, SECURITY.md, CHANGELOG, LICENSE.

---

## G5. Anti-Patterns

| Anti-pattern | The right way |
|---|---|
| Bloating the rules file | < ~150 lines; procedure → procedure layer |
| Putting everything in output style | A single short discipline tone |
| Big single prompt | Split into small tasks (task graph) |
| Everything in one session | Roles + isolated review/QA |
| Not asking for tests / shallow tests | Test pyramid + verify first |
| Heavy tests on every edit (hook) | Edit hook = single-file format; tests = Stop/CI |
| Not producing a handoff | A handoff file at the end of every task |
| Not writing headless output to a file | `> docs/agent/handoffs/...` |
| Having the author's session review it | Isolated context + adversarial verify |
| Not recording decisions/status | DECISIONS.md (ADR) + live TASKS.md |
| Trusting "done" without evidence | Command + raw output required |
| Skipping discovery | No plan/implement without DoR |
| Thinking of NFRs late | NFRs first-class (Part B §B4) |
| Thinking "deploy = done" | Observability + rollback + ORR (Part D) |

---

## G6. Deliverables & Checklists

### Recommended folder structure
```
repo/
├── CLAUDE.md / AGENTS.md            # rules (short)
├── docs/
│   ├── agentic-os/                  # THIS GUIDE (reference)
│   └── agent/                       # runtime control plane (live)
│       ├── PRODUCT_BRIEF.md DOMAIN_MODEL.md UX_SPEC.md RISKS.md
│       ├── ARCHITECTURE.md TASKS.md DECISIONS.md DEBT.md
│       ├── TEST_PLAN.md RUNBOOK.md TRACEABILITY.md THREAT_MODEL.md KNOWLEDGE.md
│       └── handoffs/
├── (Claude Code example) .claude/{settings.json,skills/,hooks/,output-styles/}
└── (extra files per mobile/data/ai archetype)
```

### Project startup checklist
```
[ ] Archetype + maturity level chosen (Part A §A3, G4)
[ ] git init; rules file < 150 lines
[ ] Enforcement set up: CI + gates (+ hooks/permissions if Claude Code)
[ ] Discovery: PRODUCT_BRIEF + DOMAIN_MODEL (+ UX_SPEC) + RISKS filled → DoR ✅
[ ] KNOWLEDGE.md: repo map + glossary
[ ] TASKS.md: epic + acceptance criteria; task graph + critical path
[ ] TRACEABILITY skeleton
```

### Feature implementation checklist
```
[ ] DoR passed; task graph clear
[ ] Plan approved (no implement without approval); branch created
[ ] Small loop: each step tested, small commits, task-id'd
[ ] DoD passed (Part D §D5): test/lint/build/observability/migration
[ ] Isolated review + QA PASS (with evidence); Critical/High = 0
[ ] Handoff written; TASKS/DECISIONS/TRACEABILITY current; /clear
```

### Release gate checklist
```
[ ] All DoD; CI green (+ project-type gates)
[ ] Security scan clean; performance NFR met
[ ] Rollback playbook + flag/canary plan; ORR complete
[ ] CHANGELOG + docs current; version tagged
```

### Final prompt quality checklist (for G2 output)
```
[ ] Role + context file references clear
[ ] Task measurable (no vague verbs); DO/DON'T explicit
[ ] Acceptance criteria + test strategy present
[ ] Assumptions explicit; no open questions left
[ ] Traceability: each task tied to a requirement
```

---

*[Back to INDEX](INDEX.md) — end of the guide.*
