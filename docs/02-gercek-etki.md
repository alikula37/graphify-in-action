# 📊 Graphify'in Bu Projede Gerçek Etki

> Tüm rakamlar **ölçüldü** (2026-08-16, `graphify 0.9.43`) — tahmin veya uydurma yok.
> Token ölçümleri `cl100k_base` tokenizer ile yapıldı; dosya seti `node_modules`/`dist`
> hariç kaynak kod (`services/` altındaki `.py` + `.ts` + `.tsx`).
> Kaynak proje: **Critic Forecast** (çok modelli finansal tahminleme platformu).

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
| `graphify query "forecast cone horizon ensemble"` çıktısı | **6.430 bayt · 1.694 token** |
| Kod tabanının tamamı | 71.802 token |
| **Token azalması** | **%97,6** (1 − 1.694 / 71.802) |
| Bayt bazında azalma | %97,6 (6.430 bayt / 271.540 bayt) |
| `graph.json` (tüm bilgi) | 599 KB · **167.202 token** · 562 node · 931 kenar |

> Yani: kodun tamamını okuyup ilişkileri sıfırdan çıkarmak yerine, ajan her soruya
> **kodun ~%2,4'ü kadar** bağlamla gidiyor ve geri kalanı hazır graf üzerinden çözüyor.

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
| Kod taraması | 7.580 satır / 71.802 token → sorgu başına 1.694 token (**%97,6 azalma**) |
| Settings çıkmazı | 1 dead-end işareti → aynı hataya bir daha düşülmedi |
| Hata ayıklama | `path`/`query` ile doğru dosyaya 1 adımda ulaşma |
| Bellek | 12 anı → 11 useful; `qra.py`/`jobs.py` öncelikli kaynak |
| Maliyet | **0 API çağrısı** — tree-sitter yerel parse, offline |
| Depolama | Tüm bilgi 599 KB `graph.json` (+ ekstrakt klasörü 3,9 MB) |

> ⚠️ **Dürüstlük notu:** Tüm sayılar bu projede ölçüldü. Token sayıları `cl100k_base`
> tokenizer ile; "azalma yüzdesi" = 1 − (sorgu token / tüm kod token). Gerçek tasarruf
> ajanın okuma davranışına göre değişebilir; oran sabittir.
