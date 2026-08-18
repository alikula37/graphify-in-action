# 📊 Graphify'in Bu Projede Gerçek Etki

🇺🇸 [English version](../en/02-real-impact.md)

> Tüm rakamlar **ölçüldü** (2026-08-16, `graphify 0.9.43`) — tahmin veya uydurma yok.
> Token ölçümleri `cl100k_base` tokenizer ile yapıldı; dosya seti `node_modules`/`dist`
> hariç kaynak kod (`services/` altındaki `.py` + `.ts` + `.tsx`).
> Kaynak proje: **çok modelli finansal tahminleme platformu** (private).

---

## 📐 Ölçülen Veriler

### Kod tabanı (grafa aktarılan dünya)

| Metrik | Değer |
|---|---|
| Kaynak dosya sayısı | **67 dosya** |
| Toplam kod satırı | **7.580 satır** |
| Ham metin boyutu | **265 KB · 71.802 token** |
| İlk ekstrakt (kurulum sonrası) | 354 node · 680 kenar · 13 topluluk |
| Son ekstrakt (tüm geliştirmeler sonrası) | **562 node · 931 kenar · 44 topluluk** |

> Graf, kodla birlikte büyüdü: proje özellik + backfill + UI ile genişlerken, graf da
> 354→562 node'a çıktı. Her `graphify update .` ile AST üzerinden **saniyeler içinde**
> tazeleniyor.

### Tek sorgunun maliyeti (ölçülmüş)

| Metrik | Değer |
|---|---|
| `graphify query "forecast cone horizon ensemble"` çıktısı | **6.430 bayt · 1.694 token** (81 node) |
| Ajanın graphify *olmadan* okuyacağı dosyalar — 6 çekirdek dosya | **9.636 token** |
| Ajanın graphify *olmadan* okuyacağı dosyalar — 19 ilgili dosyanın tümü | **16.842 token** |
| **Bağlam azalması** | **%82–90** (1 − 1.694 / 9.636 … 1 − 1.694 / 16.842) |
| Tüm kod tabanı (yalnızca ölçek için) | 67 dosya · 71.802 token |

> Karşılaştırma, ajanın aynı soruyu yanıtlamak için **gerçekten okuyacağı** dosyalara karşı
> yapıldı — tüm kod tabanına karşı değil. graphify ilişki haritasını **1.694 token**'da
> döndürüyor; onsuz ajan ilgili dosyaları grep'leyip okumak zorunda
> (**9.636–16.842 token**, yani 5,7×–10× daha fazla).

### Ekstraksiyon kalitesi (gerçek, GRAPH_REPORT.md'den)

- **%100 EXTRACTED · %0 AMBIGUOUS** — tree-sitter ile deterministik parse, LLM yok
- Yalnızca **4 çıkarımsal (inferred) kenar** (931 kenarın %0,4'ü) — ortalama güven **0.8**
- Hiçbir API çağrısı yok, tamamen offline, **0 ₺ maliyet**

### Bellek sistemi (save-result + reflect — gerçek kayıtlar)

| Metrik | Değer |
|---|---|
| Kaydedilmiş sorgu anısı | **12** |
| İşe yarayan (useful) | **11** |
| Çıkmaz (dead end) | **1** |
| Reflect çıktısı: "Preferred sources" | ensemble modülü (3×), job modülü (2×) |
| "Tentative" | API client, tarihsel veri modülü, veri seti modülü, ensembler modülü, walk-forward modülü |

> Reflect'in sonucu gerçekten kullanıldı: sonraki oturumlarda ensemble ve job altyapısıyla ilgili
> sorular **önce ensemble ve job modüllerine** gidiyor — doğrulanmış kaynaklar öncelikli.

---

## 🧪 Gerçek Örnek Akışlar (bu projede yaşandı)

### Akış 1 — "Settings sayfası" çıkmazı 🚧→💡

1. `graphify query "settings sayfasındaki ögeleri geniş yap"` → graf, sayfayı **bu projede
   bulamadı** (node yok) → dead end işaretlendi.
2. `graphify query "bu projede settings sayfası var mı"` → tek sorguda çözüm: **o sayfa
   alakasız, eski bir projeye ait — bu projeye değil**. Sonuç `useful` kaydedildi.
3. `graphify reflect` → "settings" artık **known dead end** listesinde → kimse aynı yolu
   bir daha taramıyor.

> ⏱️ 2 sorgu ≈ 2 dakika. Graf olmasaydı: UI bileşenlerini ve sayfa tanımlarını elle
> aramak, alakasız eski bir projeye dalıp yanlış yerde vakit harcamak…

### Akış 2 — UI denetimi: boş fan chart 🖼️

- Ajan simülasyon ve grafik bileşenlerini denetledi; **fan chart'ın hiç kurulmadığını**
  (koşullu render edilen container) buldu. Ders `save-result` ile kaydedildi.
- LESSONS.md'de "Preferred sources" arasına API client, tarihsel veri modülü vb. eklendi.
- Sonuç: **aynı sınıftan hatalar** (sessiz `catch`, unmount temizliği eksikliği) sonraki
  denetimlerde öncelikli kontrol listesinde.

### Akış 3 — Backfill kuyruğu takıldı 🐛

- Çok katmanlı hata zinciri (yanlış worker imaj adı, kuyruk timeout'ları, kayıp offset, bir
  yazım hatası, depolama aksaklığı, yavaş veri çekimi) — her adımda `graphify path`/`query`
  ile doğru dosyalara gidildi, her ders kaydedildi.
- Reflect sonrası job modülü **2× useful** olarak "Preferred" listesine çıktı.

### Akış 4 — Ensemble köşe çözümü 🎯

- `graphify query "ensemble neden köşe çözümü veriyor"` → optimizasyonun küçük örnekte tek
  modele yığılması → **%30 eşit ağırlık shrink** kararı. Ders kaydedildi, ensemble modülü
  güvenilir kaynak olarak işaretlendi.

### Akış 5 — Yavaş veri çekimi 🐢→⚡

- `graphify query` ile provider cache mantığına ulaşıldı → bir varlığın 2.197 barının neden
  her seferinde yeniden çekildiği anlaşıldı (bir cache boyutu şartı) → düzeltildi.
- 31 saniyelik yüklemeler **~200 ms**'ye indi. Ders: "cache yeterliyse boyut aramasın".

---

## 🔢 Sayıların Özeti

| Nerede işe yaradı | Ne kazandırdı |
|---|---|
| Kod taraması | 9.636–16.842 token (grafsız okunan dosyalar) → sorgu başına 1.694 token (**%82–90 azalma**) |
| Settings çıkmazı | 1 dead-end işareti → aynı hataya bir daha düşülmedi |
| Hata ayıklama | `path`/`query` ile doğru dosyaya 1 adımda ulaşma |
| Bellek | 12 anı → 11 useful; ensemble ve job modülleri öncelikli kaynak |
| Maliyet | **0 API çağrısı** — tree-sitter yerel parse, offline |
| Depolama | Tüm bilgi 599 KB `graph.json` (+ ekstrakt klasörü 3,9 MB) |

> ⚠️ **Dürüstlük notu:** Tüm sayılar bu projede ölçüldü. Token sayıları `cl100k_base`
> tokenizer ile. "Azalma yüzdesi", graphify sorgu çıktısını ajanın aynı soru için gerçekten
> okuyacağı dosyalarla (grep + oku) karşılaştırır — tüm kod tabanıyla değil. 6 çekirdek dosya
> (9.636 token) aralığın muhafazakâr ucu; 19 dosya (16.842 token) sorgunun işaret ettiği tüm
> dosyalardır.
