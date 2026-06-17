# Part D — Quality & Operability

🌐 English · [Türkçe](../tr/part-d-quality-operability.md)

*[← Part C](part-c-execution.md) · Next: [Part E — Governance & Evolution](part-e-governance-evolution.md)*

> **This Part defines "done."** Done = not "code written" but "tested + observable + safely
> releasable." Exit gate: **Definition of Done** + **Release Gate**.

---

## D1. Test Strategy & Verification Taxonomy → `TEST_PLAN.md`

"make test" is not a single line. Test pyramid: many unit, fewer integration, few e2e.

| Test type | What it verifies | When mandatory |
|---|---|---|
| Unit | Pure logic, edge cases | Always |
| Integration | Component + DB/queue | Any work with data/IO |
| Contract | Schema between services/consumers | Multiple services/consumers |
| E2E | A user journey | Critical user flows |
| Property-based / fuzz | Wide input space, untrusted input | Parsers, serializers, security boundaries |
| Load / performance | NFR thresholds (p95/p99, RPS) | Anything with a performance NFR |
| Chaos / resilience | Fault injection, durability | High-reliability systems |
| **Eval set (AI/ML)** | Model output quality (golden set + threshold) | Anything with an LLM/ML feature (see Part F) |

**Flaky test policy:** A flaky test is a bug; quarantine + issue. **Coverage** is a signal, not a
target; if the critical path isn't covered, 90% lies.

**Verify first:** The (ISOLATED) QA agent collects **command + raw output** for each gate; it
never writes "passing" without evidence.

---

## D2. Project-Type-Specific Quality Gates

A single uniform gate is wrong. Choose gates by project archetype (archetype detail: Part G).

| Type | Test requirement | Release criteria | Performance | Security |
|---|---|---|---|---|
| **SaaS/Web** | unit+integration+e2e+contract | canary + rollback ready | p95 latency, Core Web Vitals | authz, tenant isolation, OWASP |
| **Backend API** | unit+contract+load | versioned, backward-compat | RPS, p99, error budget | input validation, rate limit, authz |
| **CLI tool** | unit+golden-output+cross-platform | semver, changelog, binary signing | startup time | arg/path injection, secret handling |
| **Mobile** | unit+UI+device matrix | store review, staged rollout | cold start, memory, battery | secure storage, cert pinning |
| **AI/ML** | unit+**eval set**+regression | model card, eval threshold passed | inference latency, $/req | prompt injection, data/PII leak |
| **Data platform** | unit+data-quality+pipeline | schema-compat, idempotency | throughput, freshness SLA | access control, lineage, retention |
| **Internal tooling** | unit+smoke | fast iterate, minimal gate | "fast enough" | least-privilege, audit log |
| **OSS** | unit+CI matrix+docs | semver, CHANGELOG, release notes | — | SECURITY.md, supply-chain, signed release |

---

## D3. Performance, Observability & Operability → `RUNBOOK.md`

"Done" doesn't end at deploy; a system you can't observe you can't operate.

```
Observability (3 pillars):
- Logs: structured (JSON), correlation-id, PII-safe, levels
- Metrics: RED (Rate/Errors/Duration) or USE (Utilization/Saturation/Errors)
- Traces: spans, distributed tracing

Operability:
- Health checks: liveness / readiness; graceful shutdown
- SLI/SLO + error budget; alerting actionable & low-noise
- Capacity planning, performance budgets, load test thresholds
- Error reporting (Sentry, etc.); RUNBOOK.md (incident steps, rollback)
- Operational Readiness Review (ORR): part of the release gate
```

**Observability is a feature, not an afterthought:** every new service/endpoint ships with
log+metric+trace.

---

## D4. Release, Rollout & Versioning

| Topic | Practice |
|---|---|
| Branch strategy | Trunk-based or short-lived feature branch; stacked PRs |
| Versioning | Semver; deprecation window + migration guide on public APIs |
| Changelog | Every release; "keep a changelog" format |
| Feature flags | Risky change behind a flag; dark launch |
| Progressive delivery | Canary / staged rollout / blue-green |
| **Rollback playbook** | Each deploy's undo steps written **in advance** (RUNBOOK) |
| Data migration safety | expand → migrate → contract; backfill; reversible; downtime plan |
| Backward compatibility | Don't break consumers; guaranteed by contract tests |

---

## D5. Definition of Done (DoD) & Release Gate

**DoD (each task):**
```
[ ] Acceptance criteria met (item by item from the brief)
[ ] Test pyramid applied: unit + (required types); raw output seen
[ ] Lint + build green
[ ] Observability added (log/metric/trace; where needed)
[ ] Migration up+down tested (if any)
[ ] Isolated review + QA PASS; Critical/High = 0
[ ] Docs/changelog current; handoff written; TASKS/DECISIONS current
```

**Release Gate (before merge/deploy, enforced by CI):**
```
[ ] All DoD passed
[ ] CI: test + lint + build + (project-type gates) green
[ ] Security scan (SAST/secret-scan/dep-scan) clean
[ ] Performance NFR thresholds met (load test if required)
[ ] Rollback playbook ready; flag/canary plan clear
[ ] ORR complete (operational readiness)
```

> **Enforcement:** DoD/Release Gate is *not guided* by a prompt; it's *enforced* by CI/hook. An
> agent saying "done" cannot pass without showing evidence.

---

*Next: [Part E — Governance & Evolution](part-e-governance-evolution.md)*
