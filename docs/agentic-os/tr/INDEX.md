# Agentic Software Engineering Operating System (Agentic-OS)

🌐 [English](../en/INDEX.md) · Türkçe

> **Bir cümlede:** Büyük ölçekli yazılım projelerini LLM ajanlarıyla (Claude, GPT, Gemini,
> Qwen, DeepSeek, OSS agent framework'leri) düşük belirsizlik ve yüksek doğrulanabilirlikle
> yürütmek için **model-agnostik** bir operasyon çerçevesi.

Bu rehber bir "Claude Code kullanım kılavuzu" değildir. Claude Code, burada anlatılan
kavramların **bir örnek implementasyonudur.** Omurga modelden bağımsızdır; mekanizmalar
(skills, hooks, permissions) yerine başka araçlar koyabilirsin.

---

## 0. Temel Felsefe (her şeyin altındaki 4 ilke)

1. **Kalıcı hafıza = dosya, context değil.** Ajan hafızasızdır; gerçek hafıza repodaki
   dosyalardır (`docs/agent/*`). Bu, *hangi modeli kullanırsan kullan* geçerlidir.
2. **Three-Part Transform.** Olgun mühendislikteki her süreç üç parçaya çevrilir:
   **Artifact** (doldurulmuş dosya = contract) + **Procedure** (prompt/skill = nasıl yapılır)
   + **Gate** (hook/CI = atlanamaz kontrol). Bir süreç bu üçlemeye oturmuyorsa eksiktir.
3. **Guidance ≠ Enforcement.** Yönlendirme esnektir, model atlayabilir (prompt/rules).
   Zorlama deterministiktir, model atlayamaz (CI/hook/sandbox). Kritik olan her şey enforce edilir.
4. **Verify first.** "Çalışıyor / geçti" iddiası kanıtsız (komut + ham çıktı) kabul edilmez.

---

## 1. Model-Agnostic Mechanism Map

Omurga kavramlar ve bunların farklı araçlardaki karşılığı. **Rehber soldaki sütuna dayanır;
sağdaki sütunlar değiştirilebilir.**

| Omurga kavram (model-agnostik) | Claude Code | GPT / OpenAI | Gemini | OSS agent framework |
|---|---|---|---|---|
| Kalıcı kurallar | `CLAUDE.md` | `AGENTS.md` / system prompt | system instruction | rules / system prompt |
| Tekrarlanan prosedür | Skill / slash command | saved prompt / custom GPT / function | gem / prompt template | tool / agent template |
| Deterministik kapı | Hook (`exit 2`) | CI / pre-commit | CI | callback / middleware / CI |
| Erişim sınırı | Permissions | tool allowlist / sandbox | function allowlist | guardrail / policy |
| İzole denetim | `context: fork` / `claude -p` | ayrı thread / ayrı agent | ayrı session | sub-agent / fresh context |
| Dosya-tabanlı hafıza | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` | `docs/agent/*` (her yerde aynı) |

> **Çıkarım:** Dosya-tabanlı hafıza ve contract'lar her modelde **aynı** çalışır. Sadece
> "procedure" ve "gate" katmanının aracı değişir. Bu yüzden omurgayı oraya değil dosyalara bağlıyoruz.

---

## 2. Bu Rehberi Nasıl Kullanmalı

### İnsanlar için — okuma sırası
1. **Part A — Foundations**: zihinsel model, kontrol düzlemi, olgunluk seviyeni belirle.
2. **Part B — Discovery & Definition**: koda başlamadan *ne* inşa edeceğini sabitle.
3. **Part C — Execution**: ajan decomposition, orchestration, handoff, workflow.
4. **Part D — Quality & Operability**: test stratejisi, proje-tipi gate'leri, observability, release.
5. **Part E — Governance & Evolution**: dependency/refactor politikası, traceability, güvenlik.
6. **Part F — Agentic OS Layer**: bilgi/hafıza katmanı, context ekonomisi, AI-feature, HITL.
7. **Part G — Protocols & Deliverables**: interview protokolü, final prompt üretimi, failure modes,
   archetype şablonları, checklist'ler.

İlk kez mi başlıyorsun? **Part A → Part B → Part G** yeterli. Diğerlerini ihtiyaç oldukça aç.

### LLM'ler için — operasyon protokolü
Bu rehberi sana veren bir kullanıcıyla çalışıyorsan:
1. **Önce olgunluk ve archetype'ı belirle** (Part A §Maturity Model, Part G §Archetypes).
2. **Eksik bilgiyi topla** — varsayım yapma. Bkz **Part G — LLM Interview Protocol**.
3. **Discovery artefaktlarını** birlikte doldur (Part B). Definition of Ready geçmeden planlama yok.
4. **Task decomposition + agent assignment** üret (Part C).
5. **Acceptance criteria + test stratejisi** ekle (Part D).
6. **Final implementation prompt** üret — **Part G — Final Prompt Generation Protocol (7-step)**.
7. Çıktın *düşük belirsizlikli, traceable* olmalı; muğlak fiil yasak, ölçülebilir hedef şart.

> **Prompt engineering ayrı bir konu değildir.** Discovery → Planning → Execution → Review →
> Handoff sürecinin doğal parçasıdır. Her Part içinde ilgili faza ait prompt kalıpları gömülüdür;
> ayrı bir "prompt bölümü" arama.

---

## 3. Parts Haritası (ne işe yarar)

| Part | Başlık | Çözdüğü problem |
|---|---|---|
| **A** | Foundations | Ajanı nasıl düşünmeli; kontrol düzlemi; olgunluk seviyesi |
| **B** | Discovery & Definition | Yanlış şeyi inşa etmeyi önler; *ne* sorusunu sabitler |
| **C** | Execution | Büyük işi ajanlara böler, orkestre eder, devreder |
| **D** | Quality & Operability | "Done"u tanımlar; test/perf/observability/release |
| **E** | Governance & Evolution | Dependency/refactor/traceability/güvenlik disiplini |
| **F** | Agentic OS Layer | Bilgi/hafıza, context ekonomisi, AI-feature, insan onayı |
| **G** | Protocols & Deliverables | LLM interview, final prompt üretimi, failure modes, archetypes |

---

## 4. İki Ayrı Klasör — Karıştırma

- **`docs/agentic-os/`** = **bu rehber** (referans, nadir değişir, genel).
- **`docs/agent/`** = **çalışma zamanı kontrol düzlemi** (projeye özel canlı artefaktlar:
  TASKS.md, DECISIONS.md, ARCHITECTURE.md, handoffs/ …). Her projede ayrı doldurulur.

Rehber (`agentic-os`) sana *nasıl* doldurulacağını söyler; kontrol düzlemi (`agent`) doldurulmuş halidir.

---

## 5. Artifact Registry (rehber boyunca üretilen contract'lar)

| Artifact | Part | Amaç |
|---|---|---|
| `PRODUCT_BRIEF.md` | B | Problem, segment, MVP, FR/NFR, başarı metriği |
| `DOMAIN_MODEL.md` | B | Entity, state machine, contract, migration |
| `UX_SPEC.md` | B | Journey, ekran, state, a11y (UI'lı projeler) |
| `RISKS.md` (RAID) | B | Risk/varsayım/issue/dependency takibi |
| `ARCHITECTURE.md` | C | Bileşen, sınır, veri akışı |
| `TASKS.md` | C | Canlı görev durumu + sahiplik (kilit mekanizması) |
| `DECISIONS.md` (ADR) | C/E | Geri dönülmesi pahalı kararlar |
| `handoffs/*` | C | Ajanlar arası devir contract'ları |
| `TEST_PLAN.md` | D | Test stratejisi + acceptance |
| `RUNBOOK.md` | D | Operasyon/incident hazırlığı |
| `TRACEABILITY.md` | E | Requirement→Task→Test→Code matrisi |
| `KNOWLEDGE.md` / glossary | F | Repo map, ubiquitous language |

---

*Sonraki: [Part A — Foundations](part-a-foundations.md)*
