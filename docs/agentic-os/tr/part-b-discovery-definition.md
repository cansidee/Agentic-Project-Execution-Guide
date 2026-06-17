# Part B — Discovery & Definition

🌐 [English](../en/part-b-discovery-definition.md) · Türkçe

*[← Part A](part-a-foundations.md) · Sonraki: [Part C — Execution](part-c-execution.md)*

> **Bu Part'ın amacı:** Koda başlamadan *ne* inşa edileceğini sabitlemek. En pahalı hata yanlış
> ürünü doğru inşa etmektir; LLM'ler memnuniyetle muğlak istekleri kodlar. Burada üretilen
> artefaktlar, Part G'deki final prompt'un **belirsizliğini baştan keser.**
>
> **Çıkış kapısı — Definition of Ready (DoR):** Bu Part'ın artefaktları minimum doldurulmadan
> planlama/implementation başlamaz.

---

## B1. Product Discovery & Requirement Hardening → `PRODUCT_BRIEF.md`

**Prompt kalıbı (Discovery fazı):** "Aşağıdaki brief'i birlikte dolduralım. Eksik/muğlak her
alan için bana net soru sor; varsayım yapma. Doldurana kadar implementasyon önerme."

```markdown
# Product Brief: <ürün/feature>
## Problem statement
- Kim, hangi acı, hangi kanıt (data/quote)?
## Kullanıcı segmentleri & JTBD (jobs-to-be-done)
## MVP tanımı
- IN-SCOPE (madde):
- OUT-OF-SCOPE (açıkça yazılır — scope creep'i keser):
## Functional requirements (FR-01, FR-02 …)
## Non-functional requirements (NFR — ayrı first-class, bkz B4)
## Başarı metrikleri
- North Star metric + guardrail metrics (bozulmaması gerekenler)
## Riskli varsayımlar
- A-01: <varsayım> → doğrulama yöntemi → durum (open/validated/invalid)
## Open questions (Claude'a/ajana gitmeden çözülmeli)
```

**Product validation:** En riskli varsayımı en ucuz yolla test et (prototip, fake door,
kullanıcı görüşmesi) — kod yazmadan önce. Validasyon sonucu brief'e işlenir.

---

## B2. Domain Model, Data Contract & State Design → `DOMAIN_MODEL.md`

**Neden first-class:** Veri ve durum tasarımı geri dönülmesi en pahalı katmandır; ajanlar
entity'leri tutarsız üretir. Contract "freeze" edilmeden bağımlı ajanlar başlamamalı.

```markdown
# Domain Model
## Ubiquitous language / glossary
- <terim> = <tanım>   (tüm ajanlar aynı terimi kullanır)
## Entities & invariants
- <Entity>: alanlar, invariant'lar (her zaman doğru olması gerekenler)
## Bounded contexts / aggregate sınırları
## State machines
- <Entity>: durum → izinli geçişler → guard (koşul) → yan etki
## API contract (single source of truth)
- OpenAPI / proto / GraphQL şeması referansı; sürüm
## Validation rules
- Alan-bazlı + iş kuralı; nerede uygulanır (client/server/db)
## Error model
- Hata taksonomisi: kod, şekil (shape), retry'lanabilir mi, kullanıcıya mesaj
## Permission model
- Rol → kaynak → aksiyon matrisi
## Migration strategy
- expand → migrate → contract; backfill planı; geri-alınabilirlik; downtime
```

**Gate:** Contract dosyası "frozen" işaretlenmeden frontend/consumer ajanlar başlamaz (bkz
Part E — contract drift).

---

## B3. UX / User Flow Specification → `UX_SPEC.md`

*(UI içeren projeler için: SaaS, web, mobil. CLI/data/kütüphane için skip — bkz Part D §16.)*

```markdown
# UX Spec
## User journeys
- <journey>: adım adım (happy path + alternate + error path)
## Screen inventory + navigation model
## Her ekran için durumlar
- loading / error / empty / partial / success
## Form davranışı
- inline validation, optimistic UI, retry, autosave, dirty-state uyarısı
## Accessibility
- WCAG hedefi (örn. AA), klavye navigasyonu, screen-reader, kontrast
## Responsive & platform
- breakpoint'ler; mobilde dokunma hedefleri; i18n/l10n, RTL
## UX acceptance criteria (ölçülebilir)
- örn. "boş durumda CTA görünür", "hata 1 retry sunar"
```

---

## B4. Non-Functional Requirements & Risk Register → `RISKS.md`

**NFR'ler brief'in parçası ama gate'lerde ölçülebilir olmalı.** Sık unutulan NFR'ler:

| NFR sınıfı | Örnek ölçüt |
|---|---|
| Performans | p95/p99 latency, throughput, Core Web Vitals, cold start |
| Ölçeklenebilirlik | eşzamanlı kullanıcı, RPS, veri hacmi büyüme |
| Erişilebilirlik | WCAG AA |
| Güvenilirlik | SLO/error budget, RTO/RPO |
| Güvenlik & gizlilik | authz, PII sınıflandırma, retention |
| Maliyet | $/istek, altyapı bütçesi, AI token maliyeti (bkz Part F) |
| Operability | log/metric/trace kapsamı, alerting |
| Uyumluluk | GDPR/SOC2/HIPAA, veri ikametgâhı |

**Risk Register (RAID):**
```markdown
# RISKS.md (Risks / Assumptions / Issues / Dependencies)
- R-01 | risk | olasılık×etki | sahip | azaltma | durum
- A-01 | assumption | doğrulama yöntemi | durum
- I-01 | issue (gerçekleşmiş) | etki | aksiyon
- D-01 | dependency (dış) | sahip | hazır mı | tarih
```
**En riskli madde → spike (timeboxed araştırma görevi)** olarak Part C'ye taşınır.

---

## B5. Definition of Ready (DoR) — bu Part'ın çıkış gate'i

```
[ ] PRODUCT_BRIEF: problem, MVP scope/out-of-scope, FR/NFR, başarı metriği dolu
[ ] DOMAIN_MODEL: entity + state + API contract + error/permission modeli dolu
[ ] UX_SPEC dolu (UI'lı projede) veya "UI yok" işaretli
[ ] RISKS: en riskli 3 varsayım + azaltma; open question kalmadı
[ ] Acceptance criteria taslağı her FR için var
→ Hepsi ✅ değilse Part C (planning/implementation) BAŞLAMAZ.
```

---

*Sonraki: [Part C — Execution](part-c-execution.md)*
