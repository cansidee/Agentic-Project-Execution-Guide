# Part A — Foundations

🌐 [English](../en/part-a-foundations.md) · Türkçe

*[← INDEX](INDEX.md) · Sonraki: [Part B — Discovery & Definition](part-b-discovery-definition.md)*

---

## A1. Mental Model

LLM ajanını şöyle düşün: **dar çalışma belleği olan, çok yetenekli ama hafızasız bir senior
engineer.** Her session bir "vardiya". Vardiya bitince hafızası silinir; geriye yalnızca diske
yazılan kalır. Bu, Claude/GPT/Gemini/OSS fark etmeksizin geçerlidir.

| Kavram | Anlamı | Operasyonel sonuç |
|---|---|---|
| **Context** | Anlık çalışma belleği (prompt + okunan dosya + araç çıktısı) | Sınırlı kaynak; gürültüyle doldurma |
| **Memory** | Otomatik yüklenen kalıcı kurallar (CLAUDE.md/AGENTS.md) | Kısa tut; her token her turda yüklenir |
| **Session** | Bir vardiya; context uçucudur | Görev bitince temizle (`/clear` vb.) |
| **File persistence** | Diske yazılan = gerçek kalıcı hafıza | Kritik her şey diske |
| **Agentic loop** | Oku→Plan→Değiştir→Test→Gör→Düzelt | Her tur küçük olmalı |

**Hangi bilgi diske, hangisi context'te:**

| Diske (kalıcı) | Context'te (uçucu) |
|---|---|
| Kararlar, görev durumu, mimari | Ara muhakeme |
| API/contract imzaları | Denenmiş çıkmaz yollar |
| Handoff/güvenlik/test raporları | Geçici debug log'u, tek seferlik grep |

---

## A2. Core Principles

1. **Kalıcı hafıza = dosya.** Diske yazılmayan karar yok sayılır.
2. **Küçük doğrulanabilir loop.** Büyük tek-prompt = doğrulanmamış dev diff = teknik borç.
3. **Sabit döngü:** `Discovery → Plan → Implement → Test → Verify → Handoff`.
4. **Verify first.** Kanıt (komut + ham çıktı) olmadan "done" yok.
5. **Küçük görevler.** Her görev tek sorumluluk, tek branch, küçük commit.
6. **Three-Part Transform.** Her süreç = Artifact + Procedure + Gate.

---

## A3. Agent Maturity Model

Nerede olduğunu bil; bir sonraki seviyeye geç. Atlama yapma — her seviye altındakini varsayar.

| Level | Ad | Ne var | Sınır / risk | Geçiş sinyali |
|---|---|---|---|---|
| **L0** | Tek prompt | "Şunu yap" | Hafıza yok, tekrar iş, doğrulama yok | Aynı bağlamı tekrar yazıyorsun |
| **L1** | Kalıcı kurallar | CLAUDE.md/AGENTS.md + temel dosya hafızası | Kurallar atlanabilir; prosedür yok | Aynı çok-adımlı promptu yapıştırıyorsun |
| **L2** | Prosedürler | Skills / saved prompts; sabit-format çıktı | Hâlâ guidance; enforce yok | "Test çalıştırmayı unuttu" oluyor |
| **L3** | Enforcement | Hooks + CI + permissions; gate'ler | Tek-ajan; orkestrasyon yok | Paralel/izole iş gerekiyor |
| **L4** | Orchestration | Multi-agent: pipeline, fan-out, izole review/QA | Bilgi/hafıza ad-hoc; ekonomisiz | Context şişiyor, maliyet/drift artıyor |
| **L5** | Agentic OS | Bilgi katmanı + context ekonomisi + traceability + HITL + archetypes | — (sürekli iyileştirme) | — |

**Kullanım:** Küçük internal tool L2–L3'te kalabilir. Büyük SaaS/platform L4–L5 hedefler.
Olgunluk seviyeni INDEX'teki archetype ile birlikte seç (Part G).

---

## A4. Project Control Plane (model-agnostik)

İki katman: **kurallar (memory)** ve **canlı artefaktlar (docs/agent/)**.

### Kurallar dosyası (CLAUDE.md / AGENTS.md / system prompt)
- **Amaç:** Her session otomatik yüklenen kalıcı kural & gerçekler.
- **Yaz:** Working agreement, build/test komutları, sınırlar, kullanılacak prosedür listesi.
- **YAZMA:** Uzun prosedür, geçici not, mimari roman. **< ~150 satır.**
- **Best practice:** Prosedürü procedure katmanına (skill/saved-prompt) taşı.

### Canlı artefaktlar — `docs/agent/`
Her dosya: amaç / ne zaman / ne yazılır / ne YAZILMAZ.

| Dosya | Amaç | YAZMA |
|---|---|---|
| `ARCHITECTURE.md` | Bileşen, sorumluluk, veri akışı, trust boundary | Geçici keşif notu |
| `TASKS.md` | Canlı görev durumu (`[x]/[>]/[ ]` + sahip + commit) | Bitmiş işi silme; tarihçe kalsın |
| `DECISIONS.md` | ADR; geri dönülmesi pahalı kararlar | Önemsiz tercih |
| `TEST_PLAN.md` | Test stratejisi + acceptance + komutlar | — |
| `handoffs/*` | Ajanlar arası devir contract'ı | Uçucu muhakeme |

> **Claude Code örneği:** `.claude/settings.json` (permissions+hooks, commit'lenir),
> `.claude/settings.local.json` (kişisel, gitignore), `.claude/skills/*`, `.claude/hooks/*`,
> `.claude/output-styles/*`. **Bunlar L3 implementasyonudur; omurga değildir.** Başka araçta
> CI + pre-commit + saved prompts ile aynı sonucu kurarsın.

---

## A5. Tooling Separation (guidance vs enforcement)

| Mekanizma sınıfı | Sınıf | Ne zaman yüklenir | Atlanabilir? | Doğru kullanım | Anti-pattern |
|---|---|---|---|---|---|
| Kurallar dosyası | Guidance | Her session | Evet | Kısa kurallar | Şişirmek, prosedür gömmek |
| Prosedür (skill/prompt) | Guidance | Çağrılınca | Çağrılırsa uygulanır | Tekrarlanan workflow | Kurallar dosyasına gömmek |
| Gate (hook/CI) | **Enforcement** | Olay/PR'da | **Hayır** | test/lint/build, komut engeli | Her edit'te ağır test |
| Erişim sınırı (permission/sandbox) | **Enforcement** | Her session | Hayır | push/secret/dış-çağrı engeli | Prompt'la "yapma" demek |
| Rol/ton (output style) | Guidance | System prompt | Eğilim | Tek disiplin tonu | Her şeyi buraya koymak |

**Karar kuralı:** Tekrarlanan prosedür→procedure; her zaman doğru gerçek→kurallar dosyası;
atlanamaz kapı→gate; erişim sınırı→permission. Her kritik süreçte bu dağıtımı yeniden uygula.

---

*Sonraki: [Part B — Discovery & Definition](part-b-discovery-definition.md)*
