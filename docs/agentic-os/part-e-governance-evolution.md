# Part E — Governance & Evolution

*[← Part D](part-d-quality-operability.md) · Sonraki: [Part F — Agentic OS Layer](part-f-agentic-os-layer.md)*

> **Bu Part sistemin zamanla çürümesini önler.** Dependency hijyeni, refactor disiplini,
> izlenebilirlik ve güvenlik — büyük projede teknik borcun ve riskin yönetildiği katman.

---

## E1. Dependency & Architecture Decision Policy

Ajan "X kütüphanesini kur" der; lisans/supply-chain/lock-in riski denetimsiz kalmamalı.

**Yeni dependency checklist (her ekleme):**
```
[ ] Gerçekten gerekli mi? (std lib / mevcut dep yeterli mi)
[ ] Lisans uyumlu mu? (copyleft/ticari risk)
[ ] Maintenance sağlığı: son commit, maintainer sayısı, açık CVE
[ ] Supply-chain: lockfile + pin, SBOM, imza/provenance
[ ] Build/runtime etkisi: bundle size, transitive deps, cold start
[ ] Alternatifler değerlendirildi; vendor lock-in riski not edildi
→ Karar DECISIONS.md'ye ADR olarak. Install komutu permission'da "ask".
```

**Architecture Decision Record (ADR):**
```markdown
## ADR-<n>: <başlık>
- Tarih · Bağlam (neden gündeme geldi)
- Karar · Alternatifler (reddedilenler + neden)
- Sonuç/etki (hangi görev, hangi bileşen, geri-dönüş maliyeti)
```

**Fitness functions:** Mimari kuralları otomatik test et (katman ihlali, dependency yön kuralı,
bundle bütçesi, public API yüzeyi). Hook/CI ile enforce → mimari erozyon sessizce büyümez.

---

## E2. Refactor Policy & Technical Debt

**Refactor ≠ feature.** Karışırsa review imkânsızlaşır, regresyon gizlenir.

```
- Refactor PR davranış DEĞİŞTİRMEZ → ayrı commit/branch; testler aynı kalır (diff yok).
- Büyük rename/move: önce codemod/script + test yeşil, sonra manuel dokunuş.
- Public API değişikliği: deprecation window + versioning + migration guide (Part D §D4).
- Refactor acceptance: davranış aynı + ölçüt iyileşti (complexity/coverage/coupling).
```

**Technical debt → `DEBT.md`:**
```markdown
- DEBT-01 | açıklama | etki (ne yavaşlatıyor) | faiz (büyüyor mu) | önerilen çözüm | öncelik
```
Borç görünür olmadan ödenmez. Backlog'a (TASKS.md) periyodik çekilir.

---

## E3. Traceability, Provenance & Audit → `TRACEABILITY.md`

Büyük org'da "bu kod hangi requirement'tan, hangi karardan, hangi ajandan geldi" izlenebilmeli.
AI-üretimi değişikliklerde bu **denetim gerekliliğidir.**

**Traceability matrix:**
```markdown
| Req   | Task   | Test(ler)        | Code / PR        | Durum |
|-------|--------|------------------|------------------|-------|
| FR-01 | T-101  | TestAuth*, e2e-1 | src/auth, #123   | done  |
| NFR-2 | T-110  | load-suite       | —                | open  |
```

**Provenance (kaynak izi):**
- Commit/PR mesajında **task-id zorunlu** (hook ile enforce).
- AI-üretimi değişiklik işaretlenir (hangi ajan, hangi prompt/handoff).
- Geri-dönülmez/yüksek-etki kararlar **human approval gate**'inden geçer (Part F §F4).

**Audit faydası:** Bir requirement değişince etkilenen test/kod anında bulunur; bir bug'ın
hangi karardan doğduğu izlenir; compliance denetimi kanıtlanır.

---

## E4. Governance, Compliance & Risk

Enterprise/regüle bağlamda zorunlu; diğerlerinde seçici uygulanır.

| Konu | Pratik |
|---|---|
| Veri sınıflandırma | PII/secret/internal/public; her veri tipine işlem kuralı |
| Retention & residency | Saklama süresi, silme, veri ikametgâhı (GDPR/data residency) |
| Compliance | SOC2 / HIPAA / GDPR checklist'leri archetype'a göre (Part G) |
| Erişim & audit log | Least-privilege; kim ne yaptı izlenir |
| RAID | Risk register canlı tutulur (Part B §B4) |
| Escalation | Hangi karar/insidan kime yükselir; eşikler tanımlı |

---

## E5. Security — Cross-Cutting Layer + Deep-Dive Module

**Güvenlik artık merkez değil, her projeye uygulanan jenerik bir katman + derinleşilebilir modül.**

### E5.1 Jenerik güvenlik (her proje)
- **Threat modeling (STRIDE):** her feature için varlık + trust boundary + tehdit sınıfı.
  Artifact: `THREAT_MODEL.md`.
- **Fail-closed:** belirsizlikte en güvenli yorum (örn. auth doğrulanamıyorsa reddet).
- **Untrusted input:** her parser'a fuzz/property test (Part D §D1).
- **Secret/PII:** loglanmaz/commit'lenmez; permission `deny` (secret dosyaları) + CI secret-scan.
- **Evidence zorunlu:** her güvenlik bulgusu yer+etki+PoC durumu+fail-closed öneri içerir;
  emin olunmayan "doğrulanamadı (manuel kontrol)" diye işaretlenir.

### E5.2 Security Deep-Dive (security tooling / regüle / yüksek-risk)
- **Reviewer izolasyonu:** güvenlik review'u kodu yazan session'da ASLA; izole context (Part C §C2).
- **Auth/authz değişikliği = ekstra gate:** bu dosyalara dokunan PR'da güvenlik review + ikinci
  insan onayı zorunlu.
- **SAST / secret-scan / dependency-scan / semgrep / gosec:** yeri **CI (pre-merge)** + opsiyonel
  hızlı lokal tarama. Ağır taramayı her-edit hook'una koyma (Part G failure modes).
- **Adversarial verify:** kritik güvenlik bulguları bağımsız doğrulayıcı panelinden geçer (Part C §C2).

> Bu modül eski "AppSec rehberi"nin yerini tutar; artık genel framework'ün bir parçası.

---

*Sonraki: [Part F — Agentic OS Layer](part-f-agentic-os-layer.md)*
