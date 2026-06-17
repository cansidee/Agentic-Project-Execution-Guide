# Part B — Discovery & Definition

🌐 English · [Türkçe](../tr/part-b-discovery-definition.md)

*[← Part A](part-a-foundations.md) · Next: [Part C — Execution](part-c-execution.md)*

> **Purpose of this Part:** Lock down *what* will be built before any code. The most expensive
> mistake is building the wrong product correctly; LLMs happily code vague requests. The
> artifacts produced here **cut the ambiguity** of Part G's final prompt up front.
>
> **Exit gate — Definition of Ready (DoR):** Planning/implementation does not start until this
> Part's artifacts are minimally filled.

---

## B1. Product Discovery & Requirement Hardening → `PRODUCT_BRIEF.md`

**Prompt pattern (Discovery phase):** "Let's fill in the brief below together. For every
missing/vague field ask me a precise question; don't assume. Don't propose implementation until
it's filled."

```markdown
# Product Brief: <product/feature>
## Problem statement
- Who, what pain, what evidence (data/quote)?
## User segments & JTBD (jobs-to-be-done)
## MVP definition
- IN-SCOPE (bullets):
- OUT-OF-SCOPE (written explicitly — kills scope creep):
## Functional requirements (FR-01, FR-02 …)
## Non-functional requirements (NFR — separate, first-class, see B4)
## Success metrics
- North Star metric + guardrail metrics (must-not-break)
## Risky assumptions
- A-01: <assumption> → validation method → status (open/validated/invalid)
## Open questions (must be resolved before going to the agent)
```

**Product validation:** Test the riskiest assumption the cheapest way (prototype, fake door,
user interview) — before writing code. The validation result is folded back into the brief.

---

## B2. Domain Model, Data Contract & State Design → `DOMAIN_MODEL.md`

**Why first-class:** Data and state design is the most expensive layer to reverse; agents
produce inconsistent entities. Dependent agents must not start until the contract is "frozen."

```markdown
# Domain Model
## Ubiquitous language / glossary
- <term> = <definition>   (all agents use the same term)
## Entities & invariants
- <Entity>: fields, invariants (things that must always be true)
## Bounded contexts / aggregate boundaries
## State machines
- <Entity>: state → allowed transitions → guard (condition) → side effect
## API contract (single source of truth)
- OpenAPI / proto / GraphQL schema reference; version
## Validation rules
- Field-level + business rule; where enforced (client/server/db)
## Error model
- Error taxonomy: code, shape, retryable?, user message
## Permission model
- Role → resource → action matrix
## Migration strategy
- expand → migrate → contract; backfill plan; reversibility; downtime
```

**Gate:** Frontend/consumer agents do not start until the contract file is marked "frozen" (see
Part E — contract drift).

---

## B3. UX / User Flow Specification → `UX_SPEC.md`

*(For UI projects: SaaS, web, mobile. Skip for CLI/data/library — see Part D §16.)*

```markdown
# UX Spec
## User journeys
- <journey>: step by step (happy path + alternate + error path)
## Screen inventory + navigation model
## States per screen
- loading / error / empty / partial / success
## Form behavior
- inline validation, optimistic UI, retry, autosave, dirty-state warning
## Accessibility
- WCAG target (e.g. AA), keyboard navigation, screen-reader, contrast
## Responsive & platform
- breakpoints; mobile touch targets; i18n/l10n, RTL
## UX acceptance criteria (measurable)
- e.g. "CTA visible in empty state", "error offers one retry"
```

---

## B4. Non-Functional Requirements & Risk Register → `RISKS.md`

**NFRs are part of the brief but must be measurable in the gates.** Frequently forgotten NFRs:

| NFR class | Example metric |
|---|---|
| Performance | p95/p99 latency, throughput, Core Web Vitals, cold start |
| Scalability | concurrent users, RPS, data volume growth |
| Accessibility | WCAG AA |
| Reliability | SLO/error budget, RTO/RPO |
| Security & privacy | authz, PII classification, retention |
| Cost | $/request, infra budget, AI token cost (see Part F) |
| Operability | log/metric/trace coverage, alerting |
| Compliance | GDPR/SOC2/HIPAA, data residency |

**Risk Register (RAID):**
```markdown
# RISKS.md (Risks / Assumptions / Issues / Dependencies)
- R-01 | risk | probability×impact | owner | mitigation | status
- A-01 | assumption | validation method | status
- I-01 | issue (materialized) | impact | action
- D-01 | dependency (external) | owner | ready? | date
```
**The riskiest item → a spike (timeboxed research task)** carried into Part C.

---

## B5. Definition of Ready (DoR) — this Part's exit gate

```
[ ] PRODUCT_BRIEF: problem, MVP scope/out-of-scope, FR/NFR, success metric filled
[ ] DOMAIN_MODEL: entity + state + API contract + error/permission model filled
[ ] UX_SPEC filled (for UI projects) or marked "no UI"
[ ] RISKS: top 3 risky assumptions + mitigation; no open questions left
[ ] Acceptance criteria draft exists for each FR
→ If not all ✅, Part C (planning/implementation) DOES NOT START.
```

---

*Next: [Part C — Execution](part-c-execution.md)*
