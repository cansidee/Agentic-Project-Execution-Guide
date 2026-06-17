# Part G — Protocols & Deliverables

🌐 [English](../en/part-g-protocols-deliverables.md) · Türkçe

*[← Part F](part-f-agentic-os-layer.md) · [INDEX](INDEX.md)*

> **Bu Part doruk noktasıdır:** Discovery'den final implementation prompt'una giden standart
> protokoller + sık başarısızlık modları + hazır archetype şablonları + checklist'ler. Bir LLM
> bu Part'ı kullanarak kullanıcıyı doğru sorularla yönlendirir ve Claude Code'a (veya başka
> ajana) **düşük belirsizlikli, traceable** bir prompt üretir.

---

## G1. LLM Interview Protocol

Bu rehberle bir kullanıcıya yardım eden LLM'in izleyeceği soru-toplama protokolü. **Varsayım
yapma; eksik bilgiyi sor.**

**Adım 1 — Konumlandır:** Archetype (G4) ve olgunluk seviyesi (Part A §A3) nedir?

**Adım 2 — Kapsam soruları (Discovery, Part B):**
```
- Problem/hedef; başarı nasıl ölçülür (metrik)?
- Kullanıcı segmentleri; en kritik journey?
- MVP'de ne VAR, açıkça ne YOK?
- Stack, mevcut kod tabanı, kısıtlar?
- NFR'ler: performans/ölçek/maliyet/uyumluluk hedefleri?
- Veri & state: hangi entity, hangi geçişler, hangi contract?
- Güvenlik sınıfı: PII/secret? Auth/authz değişiyor mu? (varsa ekstra gate)
- Mevcut test/CI/observability altyapısı?
- Zaman/risk toleransı: hızlı MVP mi, audit-critical mi?
```

**Adım 3 — Eksikleri işaretle:** Hangi sorular cevapsız? Bunlar Claude'a gitmeden çözülmeli
(open questions → `PRODUCT_BRIEF`).

---

## G2. Final Prompt Generation Protocol (7-step)

Discovery çıktısını çalıştırılabilir, düşük-belirsizlikli implementation prompt'una çeviren
standart akış:

```
1. GATHER      → Part B artefaktlarından (BRIEF/DOMAIN/UX/RISKS) eksikleri topla; G1 ile sor.
2. ASSUMPTIONS → Örtük varsayımları açığa çıkar; her birine doğrulama yöntemi + durum ata.
3. RISKS       → Risk register (olasılık×etki); en riskli madde → spike görevi.
4. DECOMPOSE   → Task graph (bağımlılık + critical path); her task DoR'ı geçti mi (Part B §B5).
5. ASSIGN      → Part C rollerine ata (session/izole/headless); dosya sahipliği netleştir.
6. CRITERIA    → Her task için acceptance criteria + test stratejisi (Part D) + DoD.
7. EMIT        → Final prompt'u üret (aşağıdaki şablon). Traceability: her task bir Req'e bağlı.
```

**Final Implementation Prompt şablonu (çıktı):**
```markdown
# Implementation Prompt: <T-xxx / feature>
## Rol: <backend/frontend/...>
## Bağlam (oku): PRODUCT_BRIEF §, DOMAIN_MODEL §, ARCHITECTURE §, contract dosyası, ilgili ADR
## Görev (ölçülebilir, muğlak fiil yok):
## Kapsam DO / DON'T:
## Acceptance criteria: [ ] ... [ ] test/lint/build yeşil [ ] observability eklendi
## Test stratejisi: <hangi test türleri — Part D §D1>
## Kısıtlar: <dependency, secret, fail-closed, performans NFR>
## Workflow: önce plan (onay bekle) → küçük loop → handoff. İzole review+QA sonra.
## Traceability: bu görev <FR-xx/NFR-xx>'i karşılar.
```

> **Prompt engineering ayrı konu değil:** bu protokol Discovery→Planning→Execution→Review→
> Handoff zincirinin doğal son adımıdır.

---

## G3. Failure Modes (sık başarısızlıklar + erken tespit)

Agentic sistemlerin en sık çöktüğü yerler. Her biri için **erken tespit sinyali** ve **savunma.**

| Failure mode | Belirti / erken sinyal | Savunma (Artifact+Procedure+Gate) |
|---|---|---|
| **Yanlış requirement** | Demo'da "bunu istememiştim" | DoR + product validation (Part B); başarı metriği |
| **Yanlış decomposition** | Görevler sürekli birbirini blokluyor, yeniden bölünüyor | Task graph + critical path (Part C); spike |
| **Context poisoning** | Ajan geçersiz eski karara/yanlış bilgiye dayanıyor | `/clear` + dosyadan yeniden yükle; KNOWLEDGE.md stale işareti |
| **Stale documentation** | Doküman kodla çelişiyor | Knowledge decay işareti (Part F); doc-update gate; tek doğruluk kaynağı |
| **Contract drift** | Consumer kırılıyor, "ama bende çalışıyordu" | Contract freeze + contract test (Part D); tek contract dosyası |
| **Orphan task** | TASKS.md'de `[>]` aylardır sahipsiz | TASKS review; her `[>]` sahip+tarih; WIP limiti |
| **Hidden dependency** | "Bunu değiştirince şu patladı" (görünmeyen bağ) | Fitness functions (Part E); dependency map; integration test |
| **Fake completion** | "Done, testler geçti" — ama kanıt yok | Verify first; QA izole + komut/çıktı kanıtı; Stop/CI gate |
| **Review blindness** | Kodu yazan kendi review'unu yapmış | İzole review (Part C §C2); adversarial verify |
| **Scope creep** | Diff brief'in DON'T'larını aşıyor | Brief'te explicit OUT-OF-SCOPE; PR-task eşleşmesi |
| **Token blowup / düşük ROI** | Maliyet artıyor, ilerleme yavaş | Context ekonomisi (Part F §F2); lazy load; `/clear` |

---

## G4. Project Archetypes (hazır başlangıç şablonları)

Her archetype: önerilen agent yapısı · quality gates · workflow vurgusu · dosya yapısı.

### SaaS Platform
- **Agentler:** PM, Architect, Backend, Frontend, Reviewer(security+code), QA, Docs.
- **Gates:** unit+integration+e2e+contract; tenant izolasyonu; canary+rollback; Core Web Vitals.
- **Workflow:** tam Discovery→...→Release; feature flags + progressive delivery şart.
- **Dosyalar:** tüm `docs/agent/*` + UX_SPEC + RUNBOOK + TRACEABILITY.

### Internal Tooling
- **Agentler:** Backend(+light frontend), QA(hafif). Tek geliştirici çoğu rolü taşır.
- **Gates:** unit+smoke; least-privilege; audit log. Hızlı iterate, minimal gate.
- **Workflow:** L2–L3 yeterli; ağır ceremony yok.
- **Dosyalar:** CLAUDE.md + TASKS + DECISIONS + minimal TEST_PLAN.

### Security Tooling
- **Agentler:** + **izole** Security Reviewer (zorunlu), Threat-model agent.
- **Gates:** + SAST/secret-scan/semgrep; fuzz/property test; auth değişikliğinde ekstra gate;
  evidence zorunlu.
- **Workflow:** Security Deep-Dive (Part E §E5.2); reviewer asla aynı session.
- **Dosyalar:** + THREAT_MODEL.md (zorunlu).

### AI Application
- **Agentler:** + ML/Prompt engineer, Eval agent (izole).
- **Gates:** eval set + threshold; prompt-injection threat model; $/req NFR; regression guard.
- **Workflow:** test yerine **eval**; model migration prosedürü (Part F §F3).
- **Dosyalar:** + EVAL_SET, prompt registry, model card.

### Data Platform
- **Agentler:** Data engineer, Pipeline reviewer, QA(data-quality).
- **Gates:** data-quality + pipeline test; schema-compat; idempotency; freshness SLA; lineage.
- **Workflow:** migration/backfill güvenliği merkezde; access control + retention (Part E).
- **Dosyalar:** + schema registry, lineage doc.

### Mobile Application
- **Agentler:** Mobile, Backend(API), QA(device matrix), UX.
- **Gates:** unit+UI+device matrix; cold start/memory/battery; secure storage; staged rollout.
- **Workflow:** UX_SPEC zorunlu; store review + staged rollout release gate.
- **Dosyalar:** + UX_SPEC, device matrix.

### OSS Project
- **Agentler:** Maintainer, Reviewer, Docs. Dış katkı akışı.
- **Gates:** CI matrix; semver+CHANGELOG; SECURITY.md; supply-chain + signed release.
- **Workflow:** contribution guide; public API deprecation disiplini (Part D §D4).
- **Dosyalar:** + CONTRIBUTING, SECURITY.md, CHANGELOG, LICENSE.

---

## G5. Anti-Patterns

| Anti-pattern | Doğrusu |
|---|---|
| Kurallar dosyasını şişirmek | < ~150 satır; prosedür→procedure katmanı |
| Her şeyi output style'a koymak | Tek kısa disiplin tonu |
| Büyük tek-prompt | Küçük görevlere böl (task graph) |
| Tek session'da her şey | Roller + izole review/QA |
| Test istememek / yüzeysel test | Test pyramid + verify first |
| Her edit sonrası ağır test (hook) | Edit hook=tek-dosya format; test=Stop/CI |
| Handoff yazdırmamak | Her görev sonu handoff dosyası |
| Headless çıktıyı dosyaya yazmamak | `> docs/agent/handoffs/...` |
| Review'u yazan session'a yaptırmak | İzole context + adversarial verify |
| Kararları/durumu yazmamak | DECISIONS.md (ADR) + TASKS.md canlı |
| Kanıtsız "done"a güvenmek | Komut + ham çıktı zorunlu |
| Discovery atlamak | DoR olmadan plan/implement yok |
| NFR'i sonradan düşünmek | NFR first-class (Part B §B4) |
| "Deploy = done" sanmak | Observability + rollback + ORR (Part D) |

---

## G6. Deliverables & Checklists

### Önerilen klasör yapısı
```
repo/
├── CLAUDE.md / AGENTS.md            # kurallar (kısa)
├── docs/
│   ├── agentic-os/                  # BU REHBER (referans)
│   └── agent/                       # çalışma zamanı kontrol düzlemi (canlı)
│       ├── PRODUCT_BRIEF.md DOMAIN_MODEL.md UX_SPEC.md RISKS.md
│       ├── ARCHITECTURE.md TASKS.md DECISIONS.md DEBT.md
│       ├── TEST_PLAN.md RUNBOOK.md TRACEABILITY.md THREAT_MODEL.md KNOWLEDGE.md
│       └── handoffs/
├── (Claude Code örneği) .claude/{settings.json,skills/,hooks/,output-styles/}
└── (mobile/data/ai archetype'a göre ek dosyalar)
```

### Proje başlatma checklist'i
```
[ ] Archetype + olgunluk seviyesi seçildi (Part A §A3, G4)
[ ] git init; kurallar dosyası < 150 satır
[ ] Enforcement kuruldu: CI + gate'ler (+ Claude Code ise hooks/permissions)
[ ] Discovery: PRODUCT_BRIEF + DOMAIN_MODEL (+ UX_SPEC) + RISKS dolu → DoR ✅
[ ] KNOWLEDGE.md: repo map + glossary
[ ] TASKS.md: epic + acceptance criteria; task graph + critical path
[ ] TRACEABILITY iskeleti
```

### Feature implementation checklist'i
```
[ ] DoR geçti; task graph net
[ ] Plan onaylandı (onaysız implement yok); branch açıldı
[ ] Küçük loop: her adım test'li, küçük commit, task-id'li
[ ] DoD geçti (Part D §D5): test/lint/build/observability/migration
[ ] İzole review + QA PASS (kanıtlı); Critical/High = 0
[ ] Handoff yazıldı; TASKS/DECISIONS/TRACEABILITY güncel; /clear
```

### Release gate checklist'i
```
[ ] Tüm DoD; CI yeşil (+ proje-tipi gate'leri)
[ ] Güvenlik taraması temiz; performans NFR karşılandı
[ ] Rollback playbook + flag/canary planı; ORR tamam
[ ] CHANGELOG + docs güncel; versiyon etiketlendi
```

### Final prompt kalite checklist'i (G2 çıktısı için)
```
[ ] Rol + bağlam dosya referansları net
[ ] Görev ölçülebilir (muğlak fiil yok); DO/DON'T açık
[ ] Acceptance criteria + test stratejisi var
[ ] Varsayımlar açık; open question kalmadı
[ ] Traceability: her task bir requirement'a bağlı
```

---

*[INDEX'e dön](INDEX.md) — rehberin sonu.*
