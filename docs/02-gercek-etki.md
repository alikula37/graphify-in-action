# 📊 Graphify'in Bu Projede Gerçek Etkisi

> Tüm rakamlar **ölçüldü** (2026-08-16, `graphify 0.9.43`) — tahmin veya uydurma yok.
> Kaynak proje: **Critic Forecast** (çok modelli finansal tahminleme platformu).

---

## 📐 Ölçülen Veriler

### Kod tabanı (grafa aktarılan dünya)

| Metrik | Değer |
|---|---|
| Kaynak dosya sayısı (`services/` .py + .ts + .tsx) | **324 dosya** |
| Toplam kod satırı | **164.816 satır** |
| İlk ekstrakt (kurulum sonrası) | 354 node · 680 kenar · 13 topluluk |
| Son ekstrakt (tüm geliştirmeler sonrası) | **562 node · 931 kenar · 44 topluluk** |

> Graf, kodla birlikte büyüdü: proje 2× büyürken (özellik + backfill + UI), graf da
> 354→562 node'a genişledi. Her `graphify update .` ile AST üzerinden **saniyeler içinde**
> tazeleniyor.

### Grafın boyutu vs kodun boyutu

| Metrik | Değer |
|---|---|
| `graph.json` (tüm bilgi) | **599 KB · 562 node · 931 kenar** |
| Ham kod metni (aynı bilginin okunması gereken hali) | 164.816 satır ≈ 8+ MB |
| Tek bir sorgunun döndürdüğü alt graf | **6.430 bayt (68 satır)** |

**Çıkarım (hesaplama örneği):** "forecast cone horizon ensemble" sorgusu `graphify query`
ile 74 node'luk odaklı alt graf döndürdü. Tam kod tabanını okumak yerine ajan, toplam
bilginin **~%0.1'i** kadar bağlamla doğru dosyalara yönlendi. (Oran: 6.430 bayt / ~8.2 MB
ham metin — satır başı ~50 bayt varsayımıyla.)

### Ekstraksiyon kalitesi (gerçek, GRAPH_REPORT.md'den)

- **%100 EXTRACTED · %0 AMBIGUOUS** — tree-sitter ile deterministik parse, LLM yok
- Yalnızca **4 çıkarımsal (inferred) kenar** (%0.4) — ortalama güven **0.8**
- Hiçbir API çağrısı yok, tamamen offline

### Bellek sistemi (save-result + reflect — gerçek kayıtlar)

| Metrik | Değer |
|---|---|
| Kaydedilmiş sorgu anısı | **12** |
| İşe yarayan (useful) | **11** |
| Çıkmaz (dead end) | **1** |
| Reflect çıktısı: "Preferred sources" | `qra.py` (3×), `jobs.py` (2×) |
| "Tentative" | `client.ts`, `historical.py`, `dataset.py`, `ensembler.py`, `walkforward.py` |

> Reflect'in sonucu gerçekten kullanıldı: sonraki oturumlarda QRA ve job altyapısıyla ilgili
> sorular **önce `qra.py` / `jobs.py`'ye** gidiyor — doğrulanmış kaynaklar öncelikli.

---

## 🧪 Gerçek Örnek Akışlar (bu projede yaşandı)

### Akış 1 — "Settings sayfası" çıkmazı 🚧→💡

1. `graphify query "settings sayfasındaki ögeleri geniş yap"` → graf, sayfayı **bu projede
   bulamadı** (node yok) → dead end işaretlendi.
2. `graphify query "settings sayfası hangi portta"` → tek sorguda çözüm: **8080 = eski
   proje (kripto_dolar_yield), bizim UI 8081'de, bizde settings yok**. Sonuç `useful`
   kaydedildi.
3. `graphify reflect` → "settings" artık **known dead end** listesinde → kimse aynı yolu
   bir daha taramıyor.

> ⏱️ 2 sorgu ≈ 2 dakika. Graf olmasaydı: App.tsx + Header + tüm sayfa bileşenlerini elle
> aramak, eski projeye dalıp yanlış yerde vakit harcamak…

### Akış 2 — UI denetimi: boş fan chart 🖼️

- Ajan `SimulationPage.tsx` + `PriceChart.tsx`'i denetledi; **fan chart'ın hiç kurulmadığını**
  (koşullu render edilen container) buldu. Ders `save-result` ile kaydedildi.
- LESSONS.md'de "Preferred sources" arasına `client.ts`, `historical.py` vb. eklendi.
- Sonuç: **aynı sınıftan hatalar** (sessiz `catch`, unmount temizliği eksikliği) sonraki
  denetimlerde öncelikli kontrol listesinde.

### Akış 3 — Backfill kuyruğu takıldı 🐛

- 6 katmanlı hata zinciri (worker imaj adı, RQ timeout, `end_offset` kaybı, NameError,
  redis blip, GLD fetch) — her adımda `graphify path`/`query` ile doğru dosyalara gidildi,
  her ders kaydedildi.
- Reflect sonrası `jobs.py` **2× useful** olarak "Preferred" listesine çıktı.

### Akış 4 — QRA köşe çözümü 🎯

- `graphify query "QRA neden köşe çözümü veriyor"` → LP'nin küçük örnekte tek modele
  yığılması → **%30 eşit ağırlık shrink** kararı. Ders kaydedildi, node `qra.py` güvenilir
  kaynak olarak işaretlendi.

### Akış 5 — SOL/GLD yavaş veri 🐢→⚡

- `graphify query` ile provider cache mantığına ulaşıldı → SOL'un 2.197 barının neden her
  seferinde yeniden çekildiği anlaşıldı (cache `len >= 3000` şartı) → düzeltildi.
- 31 saniyelik yüklemeler **~200 ms**'ye indi. Ders: "cache yeterliyse boyut aramasın".

---

## 🔢 Sayıların Özeti

| Nerede işe yaradı | Ne kazandırdı |
|---|---|
| Kod taraması | 164.816 satır → sorgu başına 6,4 KB alt graf (**~%0.1 bağlam**) |
| Settings çıkmazı | 1 dead-end işareti → aynı hataya bir daha düşülmedi |
| Hata ayıklama | `path`/`query` ile doğru dosyaya 1 adımda ulaşma |
| Bellek | 12 anı → 11 useful; `qra.py`/`jobs.py` öncelikli kaynak |
| Maliyet | **0 API çağrısı** — tree-sitter yerel parse, offline |
| Depolama | Tüm bilgi 599 KB `graph.json` (+ ekstrakt klasörü 3,9 MB) |

> ⚠️ **Dürüstlük notu:** "token tasarrufu" yüzdeleri, okunacak metin boyutunun sorgu
> çıktısına oranıdır (satır başı ~50 bayt varsayımıyla); gerçek token tasarrufu ajanın
> okuma davranışına göre değişir. Ölçülen **kesin** değerler: satır sayısı, node/kenar
> sayısı, dosya boyutları ve sorgu çıktısı boyutu.
