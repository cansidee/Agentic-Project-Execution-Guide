# Part C — Execution

🌐 [English](../en/part-c-execution.md) · Türkçe

*[← Part B](part-b-discovery-definition.md) · Sonraki: [Part D — Quality & Operability](part-d-quality-operability.md)*

> **Önkoşul:** Definition of Ready (Part B §B5) geçilmiş olmalı.

---

## C1. Agent / Task Decomposition Model

**Çalışma şekli seçimi (model-agnostik):**
- **Aynı session/context:** sıkı bağlı, aynı dosyalara dokunan küçük-orta iş.
- **İzole context (fork / yeni thread / sub-agent):** kodu yazanın körlüğünden bağımsız bakış
  gereken iş — review, QA.
- **Ayrı headless proses (`claude -p` / batch API):** bağımsız, paralel, net girdi/çıktı kontratlı iş.

> **Altın kural:** Kodu yazan ajan kendi review/QA'sını **yapamaz.** Daima izole context.

| Rol | Amaç | Input contract | Output contract | Çalışma şekli |
|---|---|---|---|---|
| Product/PM | Kapsam & kriter | PRODUCT_BRIEF | TASKS epic + acceptance | Tek session (başta) |
| Architect | Tasarım & karar | Brief, domain, kod | ARCHITECTURE + ADR | Tek session / plan mode |
| Backend | API/iş mantığı | Implementation brief | Kod + test + handoff | Tek session, branch |
| Frontend/Mobile | UI | Brief + API contract + UX_SPEC | Kod + test + handoff | Ayrı session, branch |
| Reviewer (security/code) | Açık & kalite denetimi | Diff + standartlar | Severity'li rapor | **İZOLE** |
| QA/verification | Kanıtlı doğrulama | Diff + TEST_PLAN | Pass/fail rapor | **İZOLE** |
| Documentation | Dokümantasyon | Final diff + handoff | Docs/README diff | Ayrı headless |
| Refactor | Temizlik (davranış değişmez) | Diff | Bulgular | İzole |

**Spike rolü:** Belirsizlik yüksekse, implementation'dan önce *timeboxed araştırma görevi*.
Çıktısı kod değil, bir karar/öneri (ADR taslağı). Riskli varsayımları ucuza kapatır.

**Task graph & critical path:** Görevleri bağımlılıklarıyla diz. Kritik yoldakileri önce ve/veya
paralel başlat. TASKS.md'de bağımlılık işaretle (`T-103 (blocked: T-102)`).

**Çakışma/duplicate/context kaybı önleme:**
- **TASKS.md = kilit:** başlayan durumu `[>]` + sahip yazar. İki ajan aynı `[ ]`'a girmez.
- **Dosya sahipliği:** her ajan kendi dizinine yazar; paralelde aynı dosyaya yazdırma.
- **Paylaşımlı contract = interface dosyası:** biri yazar, diğeri okur.
- **Her ajan handoff ile biter;** sonraki ajan context'ten değil o dosyadan beslenir.

---

## C2. Agent Orchestration Patterns

Tek-ajanın ötesi. Büyük işte orkestrasyon kaliteyi belirler.

| Pattern | Ne zaman | Nasıl |
|---|---|---|
| **Pipeline** | Çok-aşamalı, item başına bağımsız zincir | item → stage1 → stage2 …; aşamalar arası bariyer yok |
| **Fan-out / parallel** | Bağımsız işler aynı anda | N ajan paralel, sonunda topla (bariyer) |
| **Judge panel / adversarial verify** | Bir bulgu/karar doğrulanacak | N bağımsız doğrulayıcı, her biri *çürütmeye* çalışır; çoğunluk kararı |
| **Loop-until-dry** | Bilinmeyen-boyut keşif (bug, edge-case) | K tur yeni bulgu gelmeyene dek tekrarla |
| **Completeness critic** | "Ne eksik kaldı?" | Final ajan: çalışmayan modalite, doğrulanmamış iddia, okunmamış kaynak |

**Adversarial verify (en kritik kalite pattern'i):** Her önemli bulgu/iddia için, onu *yazan*
değil, bağımsız bir doğrulayıcı "bunu çürüt — emin değilsen reddet" talimatıyla denetler.
Plausible-ama-yanlış çıktıları eler. Security ve AI-feature işinde zorunlu (Part E, F).

> **Claude Code örneği:** orchestration için Workflow/Task araçları; izole denetim için
> `context: fork` veya paralel `claude -p` prosesleri. Başka framework'te: graph executor /
> sub-agent fan-out. Omurga aynı: **bağımsız doğrulama + dosya-tabanlı toplama.**

---

## C3. Handoff & Contract System

Agent çıktıları contract olarak `docs/agent/handoffs/`'a yazılır. Doldurulabilir şablonlar:

### Implementation Brief
```markdown
# Brief: <T-xxx>
## Hedef · Kapsam DO · Kapsam DON'T
## Girdiler/referans: <ADR, DOMAIN_MODEL §, contract dosyası, UX_SPEC §>
## Acceptance criteria: [ ] … [ ] test/lint yeşil
## Kısıtlar: <yeni dependency yok / vb.>
```

### Handoff Summary (en önemli contract)
```markdown
# Handoff: <T-xxx> (<kaynak> → <hedef>)
## Durum: DONE|PARTIAL|BLOCKED — branch, commit <hash>
## Ne yapıldı · Public API/contract (imzalar, hata tipleri)
## Kararlar (ADR ref) · Test durumu (komut + sonuç)
## Açık riskler & varsayımlar
## Sonraki ajan için: <net tek talimat / review odağı>
```

### Diğer şablonlar
- **Task Plan** (DO/DON'T, dosya sırası, test stratejisi, risk, acceptance; onaysız implement yok)
- **Security Review Report** (özet severity, bulgu = yer+etki+PoC durumu+fail-closed öneri) → Part E
- **QA Verification Report** (PASS/FAIL, gate'ler komut+çıktı, acceptance madde+kanıt) → Part D
- **ADR** (bağlam, karar, alternatifler, sonuç) → Part E
- **Next-agent prompt** (rol+girdi dosyası+net görev+kapsam dışı+çıktı formatı+kısıt)

**Minimum taşınacak bilgi (handoff'un özü):** branch+hash · public contract · kararlar(ADR) ·
test durumu+tekrar komutu · açık risk+varsayım · tek net talimat.

---

## C4. Recommended Workflow (uçtan uca, model-agnostik)

| Aşama | Procedure | Okur | Yazar | Gate / çıktı |
|---|---|---|---|---|
| Discovery | (Part B kalıpları) | — | PRODUCT_BRIEF, DOMAIN_MODEL, UX_SPEC, RISKS | **DoR** |
| Architecture | architect (plan mode) | brief, domain | ARCHITECTURE, ADR | Tasarım onaylı |
| Task planning | task-plan | ARCH, TASKS | plan (onay) | Onaysız ilerleme yok |
| Implementation | implement-small-loop | brief, plan, contract | src, test | Her adım test'li |
| Testing | (loop içinde) | — | — | Yeşil olmadan adım kapanmaz |
| Review (code/security) | review (İZOLE) | diff, standartlar | handoffs/ | Critical/High = 0 |
| QA verification | qa-verify (İZOLE) | TEST_PLAN, diff | handoffs/ | PASS, kanıtlı |
| Documentation | doc agent (headless) | handoff, kod | docs, README | Diff |
| Release gate | CI + ORR | — | — | bkz Part D |
| Session cleanup | handoff + commit + clear | git | handoffs/, TASKS | Handoff yazıldı, durum güncel |

**Prompt engineering bu akışın içindedir:** her aşamanın prompt'u rol+bağlam(dosya ref)+net
görev+DO/DON'T+acceptance+çıktı formatı+kısıt içerir. Muğlak fiil yok.

---

## C5. Headless / `claude -p` Usage Model

**Ne zaman headless:** İzole/paralel, net girdi-çıktı kontratlı, otomatikleştirilebilir iş
(bağımsız review, QA, dokümantasyon). **Ne zaman interaktif:** keşif, plan tartışması, belirsizliğin
yüksek olduğu iş.

**Kurallar:** Çıktıyı daima dosyaya yönlendir (`> docs/agent/handoffs/...`). Sabit-format çıktı
üret (HEADLINE + HANDOFF blokları). Sonraki adım context'ten değil dosyadan beslenir.

```bash
claude -p "/analyze-readonly src/auth" --permission-mode plan > docs/agent/handoffs/auth-analysis.md
claude -p "/security-review feat/x"  > docs/agent/handoffs/x-sec.md &
claude -p "/qa-verify T-102"          > docs/agent/handoffs/x-qa.md  & wait
```
> Diğer modellerde karşılığı: OpenAI/Gemini batch veya CLI + script; OSS framework'te headless runner.

---

## C6. Automation Strategy

- **Promptu kurallar dosyasına gömme** (her token her turda yüklenir → şişme). Tekrarlanan
  promptu **procedure** (skill/saved-prompt) yap (lazy load + sabit format).
- **Hook/CI** = atlanamaz kapı (test/lint/build, tehlikeli komut). **Permission** = erişim sınırı
  (push/secret/dış-çağrı). **Output style/role** = en az, tek disiplin tonu.
- Genel kural: her olgun süreç → **Artifact + Procedure + Gate** üçlemesine dağıt.

---

*Sonraki: [Part D — Quality & Operability](part-d-quality-operability.md)*
