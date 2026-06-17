# Part E — Governance & Evolution

🌐 English · [Türkçe](../tr/part-e-governance-evolution.md)

*[← Part D](part-d-quality-operability.md) · Next: [Part F — Agentic OS Layer](part-f-agentic-os-layer.md)*

> **This Part prevents the system from rotting over time.** Dependency hygiene, refactor
> discipline, traceability and security — the layer where technical debt and risk are managed in
> a large project.

---

## E1. Dependency & Architecture Decision Policy

An agent says "install library X"; the license/supply-chain/lock-in risk must not go unchecked.

**New dependency checklist (every addition):**
```
[ ] Is it truly needed? (does std lib / existing dep suffice)
[ ] License compatible? (copyleft/commercial risk)
[ ] Maintenance health: last commit, maintainer count, open CVEs
[ ] Supply-chain: lockfile + pin, SBOM, signature/provenance
[ ] Build/runtime impact: bundle size, transitive deps, cold start
[ ] Alternatives evaluated; vendor lock-in risk noted
→ Decision goes to DECISIONS.md as an ADR. Install command set to "ask" in permissions.
```

**Architecture Decision Record (ADR):**
```markdown
## ADR-<n>: <title>
- Date · Context (why it came up)
- Decision · Alternatives (rejected + why)
- Outcome/impact (which task, which component, reversal cost)
```

**Fitness functions:** Automatically test architectural rules (layer violation, dependency
direction, bundle budget, public API surface). Enforce via hook/CI → architectural erosion
doesn't grow silently.

---

## E2. Refactor Policy & Technical Debt

**Refactor ≠ feature.** Mixing them makes review impossible and hides regressions.

```
- A refactor PR does NOT change behavior → separate commit/branch; tests stay the same (no diff).
- Large rename/move: first codemod/script + tests green, then manual touch-ups.
- Public API change: deprecation window + versioning + migration guide (Part D §D4).
- Refactor acceptance: behavior identical + a metric improved (complexity/coverage/coupling).
```

**Technical debt → `DEBT.md`:**
```markdown
- DEBT-01 | description | impact (what it slows) | interest (is it growing) | proposed fix | priority
```
Debt isn't paid unless it's visible. Pulled into the backlog (TASKS.md) periodically.

---

## E3. Traceability, Provenance & Audit → `TRACEABILITY.md`

In a large org, "which requirement, which decision, which agent did this code come from" must be
traceable. For AI-generated changes this is an **audit requirement.**

**Traceability matrix:**
```markdown
| Req   | Task   | Test(s)          | Code / PR        | Status |
|-------|--------|------------------|------------------|--------|
| FR-01 | T-101  | TestAuth*, e2e-1 | src/auth, #123   | done   |
| NFR-2 | T-110  | load-suite       | —                | open   |
```

**Provenance (source trail):**
- Task-id **mandatory** in commit/PR messages (enforced by hook).
- AI-generated changes flagged (which agent, which prompt/handoff).
- Irreversible/high-impact decisions pass through a **human approval gate** (Part F §F4).

**Audit benefit:** When a requirement changes, the affected tests/code are found instantly; the
decision a bug originated from is traced; compliance review is provable.

---

## E4. Governance, Compliance & Risk

Mandatory in enterprise/regulated contexts; applied selectively elsewhere.

| Topic | Practice |
|---|---|
| Data classification | PII/secret/internal/public; a handling rule per data type |
| Retention & residency | Retention period, deletion, data residency (GDPR/data residency) |
| Compliance | SOC2 / HIPAA / GDPR checklists per archetype (Part G) |
| Access & audit log | Least-privilege; who did what is traced |
| RAID | Risk register kept live (Part B §B4) |
| Escalation | Which decision/incident escalates to whom; thresholds defined |

---

## E5. Security — Cross-Cutting Layer + Deep-Dive Module

**Security is no longer the center; it's a generic layer applied to every project + a
deep-divable module.**

### E5.1 Generic security (every project)
- **Threat modeling (STRIDE):** for each feature, asset + trust boundary + threat class.
  Artifact: `THREAT_MODEL.md`.
- **Fail-closed:** the safest interpretation under uncertainty (e.g. if auth can't be verified, deny).
- **Untrusted input:** fuzz/property tests on every parser (Part D §D1).
- **Secret/PII:** never logged/committed; permission `deny` (secret files) + CI secret-scan.
- **Evidence mandatory:** every security finding includes location+impact+PoC status+fail-closed
  fix; unverified ones flagged "could not verify (needs manual check)."

### E5.2 Security Deep-Dive (security tooling / regulated / high-risk)
- **Reviewer isolation:** security review NEVER in the session that wrote the code; isolated
  context (Part C §C2).
- **Auth/authz change = extra gate:** any PR touching these files requires a security review +
  a second human approval.
- **SAST / secret-scan / dependency-scan / semgrep / gosec:** their place is **CI (pre-merge)** +
  optional fast local scan. Don't put heavy scans in an every-edit hook (Part G failure modes).
- **Adversarial verify:** critical security findings pass through an independent verifier panel
  (Part C §C2).

> This module replaces the old "AppSec guide"; it's now a part of the general framework.

---

*Next: [Part F — Agentic OS Layer](part-f-agentic-os-layer.md)*
