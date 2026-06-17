# Part F — Agentic OS Layer

🌐 [English](../en/part-f-agentic-os-layer.md) · Türkçe

*[← Part E](part-e-governance-evolution.md) · Sonraki: [Part G — Protocols & Deliverables](part-g-protocols-deliverables.md)*

> **Bu Part, L4→L5 farkıdır.** Tek-tek ajanları gerçek bir işletim sistemine çeviren katman:
> kalıcı bilgi, context ekonomisi, AI-feature mühendisliği ve insan-onay modeli.

---

## F1. Agentic Knowledge & Memory Layer → `KNOWLEDGE.md` + glossary

**Problem:** Kurallar dosyası kısa olmalı (Part A) ama büyük projede çok bilgi var. Ajan her
session sıfırdan keşfederse zaman/token yakar ve tutarsızlık üretir.

**Çözüm — katmanlı bilgi:**
```
1. Kurallar (CLAUDE.md/AGENTS.md)  → daima yüklü, kısa, "her zaman doğru"
2. Canlı durum (docs/agent/*)       → görev/karar/mimari, talep üzerine okunur
3. Indexed knowledge (KNOWLEDGE.md) → repo map + glossary + "nereden başla" pointer'ları
```

**`KNOWLEDGE.md` içeriği:**
- **Repo map:** hangi dizin ne yapar, giriş noktaları, "X için şuraya bak".
- **Ubiquitous-language glossary:** terim → tanım (tüm ajanlar/modeller aynı dili konuşur).
- **Pointer'lar:** sık gereken bilgi nerede (DOMAIN_MODEL, ADR'ler, contract dosyaları).
- **Knowledge decay yönetimi:** eskiyen/şüpheli bilgi `⚠️ stale?` işaretlenir → drift'i önler (Part G).

> Model-agnostik: RAG/embedding kullanan framework'lerde bu dosyalar index'lenir; kullanmayanda
> ajan doğrudan okur. Her iki durumda kaynak aynı dosyalardır.

---

## F2. Cost & Context Economics

Büyük projede token maliyeti ve context kalitesi doğrudan ROI'yi belirler. **Context bir bütçedir.**

### Context budget ilkeleri
| İlke | Pratik |
|---|---|
| Bilgi yoğunluğu | Tablo/checklist > uzun paragraf; aynı bilgi az token |
| Lazy loading | Prosedür/bilgi çağrılana kadar yüklenmesin (procedure katmanı) |
| Doğru granülerlik | İzole alt-göreve delege et; ana context'i temiz tut |
| Gürültü temizliği | Uzun log/diff'i context'e dökme; dosyaya yaz, özetle |
| Sık `/clear` | Görev bitince temiz başla; eski context taşıma |

### Token maliyeti & ROI
- **Prompt ROI = (üretilen doğrulanmış değer) / (harcanan token).** Düşük ROI sinyalleri:
  tekrar keşif, şişmiş kurallar dosyası, summary'siz headless çıktı (yeniden okuma), büyük tek-prompt.
- **Cache farkındalığı:** çoğu API prompt-cache sunar; kararlı prefix (kurallar+sistem) cache'lenir.
  Sık değişen içeriği sona koy. Uzun beklemeler cache'i soğutur.
- **Model seçimi maliyeti:** ucuz model rutin işe (format, basit dönüşüm), güçlü model akıl
  yürütmeye. Orchestration'da rolü modele eşle.
- **AI-feature maliyeti (üründe LLM):** $/istek bir **NFR**'dir (Part B §B4); token bütçesi
  acceptance criteria'ya girer.

### Dosya organizasyonu = context ekonomisi
İyi `KNOWLEDGE.md` + kısa kurallar + sabit-format handoff = her session daha az token, daha
yüksek isabet. Kötü organizasyon = her session yeniden keşif vergisi.

---

## F3. AI/ML & LLM-Feature Engineering

Ürünün *kendisi* LLM/ML içeriyorsa normal testler yetmez.

| Konu | Pratik |
|---|---|
| **Eval harness** | Golden set + threshold; her değişiklik eval'den geçer (Part D §D1 test türü) |
| Regression guard | Yeni prompt/model eski golden'ları bozmuyor mu |
| Prompt/version registry | Prompt'lar versiyonlu, izlenebilir (provenance) |
| Hallucination/grounding | Kaynak-bağlı (RAG) doğrulama; uydurma tespiti |
| **Prompt-injection threat model** | Untrusted input modele gidiyorsa STRIDE'a ekle (Part E §E5) |
| Data governance | Training/inference verisinde PII; retention; opt-out |
| Cost/token budget | $/istek NFR; bütçe aşımı alarmı (Part F §F2) |
| Model migration | Model değişiminde eval + maliyet + davranış diff'i; geri-alma |

> Kural: **AI feature'ı "test" ile değil "eval" ile doğrula.** Deterministik olmayan çıktı,
> deterministik gate ister: golden set + eşik + adversarial verify.

---

## F4. Human-in-the-Loop & Escalation Policy

Otonomi abartılmaz; bazı kararlar insan onayı ister.

**Approval gate gereken durumlar (örnek eşikler):**
```
- Geri-dönülmez işlem: prod migration, veri silme, dış servise yayın
- Yüksek-etki: public API kırılması, auth/authz değişikliği, fiyat/faturalama
- Güvenlik: yeni trust boundary, secret erişimi, yeni dependency (E1)
- Belirsizlik: riskli varsayım doğrulanmadan ilerleme
→ Bu durumlar prompt'la değil, gate/permission ile "ask/deny" + insan onayı.
```

**Escalation modeli:** Ajan blocker'da durur, varsayım yapmaz; `RISKS.md`/handoff'a yazar,
insana net soru sorar. "Make reasonable assumptions" yalnız düşük-risk, geri-alınabilir işlerde
ve **her varsayım loglanarak** kullanılır.

**Provenance bağı (Part E §E3):** onay gerektiren her değişiklik, kim onayladı + hangi handoff
ile izlenir.

---

*Sonraki: [Part G — Protocols & Deliverables](part-g-protocols-deliverables.md)*
