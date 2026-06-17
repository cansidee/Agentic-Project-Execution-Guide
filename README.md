# Agentic Software Engineering Operating System (Agentic-OS)

> Büyük ölçekli yazılım projelerini LLM ajanlarıyla (Claude, GPT, Gemini, Qwen, DeepSeek, OSS
> agent framework'leri) **düşük belirsizlik** ve **yüksek doğrulanabilirlikle** yürütmek için
> **model-agnostik** bir operasyon çerçevesi.

Bu, bir "Claude Code kullanım kılavuzu" değildir. Claude Code, burada anlatılan kavramların
**bir örnek implementasyonudur.** Omurga modelden bağımsızdır.

## Felsefe (4 ilke)
1. **Kalıcı hafıza = dosya, context değil.**
2. **Three-Part Transform:** her süreç = Artifact (contract) + Procedure (prompt/skill) + Gate (hook/CI).
3. **Guidance ≠ Enforcement.** Kritik olan her şey enforce edilir.
4. **Verify first.** Kanıt olmadan "done" yok.

## Yapı
| Part | Başlık | Çözdüğü problem |
|---|---|---|
| [INDEX](docs/agentic-os/INDEX.md) | Giriş + mechanism map | Rehberin nasıl kullanılacağı (insan + LLM) |
| [A](docs/agentic-os/part-a-foundations.md) | Foundations | Ajanı nasıl düşünmeli; kontrol düzlemi; maturity model (L0–L5) |
| [B](docs/agentic-os/part-b-discovery-definition.md) | Discovery & Definition | Yanlış şeyi inşa etmeyi önler; *ne* sorusu |
| [C](docs/agentic-os/part-c-execution.md) | Execution | Decomposition, orchestration, handoff, workflow |
| [D](docs/agentic-os/part-d-quality-operability.md) | Quality & Operability | Test stratejisi, gates, observability, release |
| [E](docs/agentic-os/part-e-governance-evolution.md) | Governance & Evolution | Dependency, refactor, traceability, security |
| [F](docs/agentic-os/part-f-agentic-os-layer.md) | Agentic OS Layer | Knowledge, cost/context, AI-features, human-in-the-loop |
| [G](docs/agentic-os/part-g-protocols-deliverables.md) | Protocols & Deliverables | Interview, final prompt, failure modes, archetypes, checklists |

## Nasıl kullanılır
- **İnsanlar:** `INDEX → A → B → G` ile başla; diğer Part'ları ihtiyaç oldukça aç.
- **LLM'ler:** [INDEX §2](docs/agentic-os/INDEX.md) operasyon protokolünü ve
  [Part G](docs/agentic-os/part-g-protocols-deliverables.md) interview + 7-step final prompt
  protokolünü izle.

## Başla
→ **[docs/agentic-os/INDEX.md](docs/agentic-os/INDEX.md)**

## Lisans
[MIT](LICENSE).
