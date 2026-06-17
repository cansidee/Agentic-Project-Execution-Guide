# Part D — Quality & Operability

*[← Part C](part-c-execution.md) · Sonraki: [Part E — Governance & Evolution](part-e-governance-evolution.md)*

> **Bu Part "done"u tanımlar.** Done = kod yazıldı değil; test edildi + gözlemlenebilir +
> güvenli release edilebilir. Çıkış kapısı: **Definition of Done** + **Release Gate**.

---

## D1. Test Strategy & Verification Taxonomy → `TEST_PLAN.md`

"make test" tek satır değildir. Test pyramid: çok unit, daha az integration, az e2e.

| Test türü | Ne doğrular | Ne zaman zorunlu |
|---|---|---|
| Unit | Saf mantık, edge-case | Daima |
| Integration | Bileşen + DB/queue | Veri/IO içeren her iş |
| Contract | Servisler/consumer arası şema | Birden çok servis/consumer |
| E2E | Kullanıcı journey'i | Kritik kullanıcı akışları |
| Property-based / fuzz | Geniş girdi uzayı, untrusted input | Parser, serializer, güvenlik sınırı |
| Load / performance | NFR eşikleri (p95/p99, RPS) | Performans NFR'ı olan |
| Chaos / resilience | Hata enjeksiyonu, dayanıklılık | Yüksek-güvenilirlik sistemleri |
| **Eval set (AI/ML)** | Model çıktı kalitesi (golden set + threshold) | LLM/ML feature'ı olan (bkz Part F) |

**Flaky test politikası:** Flaky test bug'dır; quarantine + issue. **Coverage** hedef değil
sinyaldir; kritik yol kapsanmıyorsa %90 yalan söyler.

**Verify first:** QA ajanı (İZOLE) her gate için **komut + ham çıktı** toplar; kanıtsız "geçti" yazmaz.

---

## D2. Project-Type-Specific Quality Gates

Tek tip gate yanlıştır. Proje archetype'ına göre gate seç (archetype detayı: Part G).

| Tip | Test gereksinimi | Release kriteri | Performans | Güvenlik |
|---|---|---|---|---|
| **SaaS/Web** | unit+integration+e2e+contract | canary + rollback hazır | p95 latency, Core Web Vitals | authz, tenant izolasyonu, OWASP |
| **Backend API** | unit+contract+load | versioned, backward-compat | RPS, p99, error budget | input validation, rate limit, authz |
| **CLI tool** | unit+golden-output+cross-platform | semver, changelog, binary signing | startup time | arg/path injection, secret handling |
| **Mobile** | unit+UI+device matrix | store review, staged rollout | cold start, memory, battery | secure storage, cert pinning |
| **AI/ML** | unit+**eval set**+regression | model card, eval threshold geçti | inference latency, $/req | prompt injection, data/PII leak |
| **Data platform** | unit+data-quality+pipeline | schema-compat, idempotency | throughput, freshness SLA | access control, lineage, retention |
| **Internal tooling** | unit+smoke | hızlı iterate, minimal gate | "yeterince hızlı" | least-privilege, audit log |
| **OSS** | unit+CI matrix+docs | semver, CHANGELOG, release notes | — | SECURITY.md, supply-chain, signed release |

---

## D3. Performance, Observability & Operability → `RUNBOOK.md`

"Done" deploy'da bitmez; gözlemlenemeyen sistem işletilemez.

```
Observability (3 pillar):
- Logs: yapısal (JSON), korelasyon-id, PII-safe, seviyeler
- Metrics: RED (Rate/Errors/Duration) veya USE (Utilization/Saturation/Errors)
- Traces: span'ler, dağıtık izleme

Operability:
- Health checks: liveness / readiness; graceful shutdown
- SLI/SLO + error budget; alerting actionable & gürültüsüz
- Capacity planning, performance budgets, load test eşikleri
- Error reporting (Sentry vb.); RUNBOOK.md (incident adımları, rollback)
- Operational Readiness Review (ORR): release gate'in parçası
```

**Observability bir feature'dır, sonradan değil:** her yeni servis/endpoint log+metric+trace ile gelir.

---

## D4. Release, Rollout & Versioning

| Konu | Pratik |
|---|---|
| Branch stratejisi | Trunk-based veya kısa-ömürlü feature branch; stacked PR |
| Versiyonlama | Semver; public API'de deprecation window + migration guide |
| Changelog | Her release; "keep a changelog" formatı |
| Feature flags | Riskli değişiklik flag arkasında; dark launch |
| Progressive delivery | Canary / staged rollout / blue-green |
| **Rollback playbook** | Her deploy'un geri-alma adımı **önceden** yazılı (RUNBOOK) |
| Data migration safety | expand → migrate → contract; backfill; geri-alınabilir; downtime planı |
| Backward compatibility | Consumer'ları kırma; contract test ile garanti |

---

## D5. Definition of Done (DoD) & Release Gate

**DoD (her görev):**
```
[ ] Acceptance criteria karşılandı (brief'teki madde madde)
[ ] Test pyramid uygulandı: unit + (gerekli türler); ham çıktı görüldü
[ ] Lint + build yeşil
[ ] Observability eklendi (log/metric/trace; gerekli yerde)
[ ] Migration up+down test edildi (varsa)
[ ] İzole review + QA PASS; Critical/High = 0
[ ] Docs/changelog güncel; handoff yazıldı; TASKS/DECISIONS güncel
```

**Release Gate (merge/deploy öncesi, CI ile enforce):**
```
[ ] Tüm DoD geçti
[ ] CI: test + lint + build + (proje-tipi gate'leri) yeşil
[ ] Güvenlik taraması (SAST/secret-scan/dep-scan) temiz
[ ] Performans NFR eşikleri karşılandı (gerekiyorsa load test)
[ ] Rollback playbook hazır; flag/canary planı net
[ ] ORR tamam (operasyonel hazırlık)
```

> **Enforcement:** DoD/Release Gate prompt'la *yönlendirilmez*, CI/hook ile *zorlanır*. "Done"
> diyen ajan kanıt göstermeden geçemez.

---

*Sonraki: [Part E — Governance & Evolution](part-e-governance-evolution.md)*
