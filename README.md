# 🧠 Graphify in Action — Bu Projede Gerçek Etki

> **7.580 satırlık bir projeyi, sorgu başına 1.694 token ile yönetmek.**
> Hepsi ölçülmüş gerçek rakamlar — uydurma yok. 🧪

![Akış](assets/flow.svg)

## 🚀 Çarpıcı Rakamlar (ölçüldü: 2026-08-16)

| 🏆 | Rakam | Ne anlama geliyor? |
|---|---|---|
| 📉 | **%97,6 daha az token** | Kod 71.802 token; tek sorgu yanıtı **1.694 token** (cl100k_base ile ölçüldü) |
| 🚫 | **0 API çağrısı · 0 ₺ maliyet** | Tree-sitter yerel parse, tamamen offline |
| ✅ | **%100 doğru ekstraksiyon** | %0 belirsiz; 931 kenardan yalnızca 4'ü çıkarımsal (%0,4) |
| ⏱️ | **~2 dakikada cevap** | Ajan, doğru dosyalara sorguyla yönlenir |
| 🗺️ | **562 node · 931 kenar · 599 KB** | Tüm kod haritası tek dosyada |
| 🧠 | **12 hafızadan 11'i işe yaradı** | Sistem öğrenir, çıkmazları tekrar aramaz |

## 🎯 "Kur, ama sadece işe yararsa" — Cevabı (ölçümle)

Kurulumdan önce sorulan kriter şuydu: *"graphify işe yaramazsa, okuma/token/context
kullanımımı olumsuz etkiliyorsa veya kötüye götürüyorsa kurma."* İşte ölçülmüş cevap:

| Kriter | Ölçüm | Sonuç |
|---|---|---|
| 💨 **Verimlilik** | Kod tabanı **71.802 token** → sorgu başına **1.694 token** | **%97,6 daha az token** |
| 🎯 **Hatasız çalışma** | Ekstraksiyon **%100 extracted · %0 ambiguous**; çıkarımsal kenar %0,4 (güven 0,8) | Deterministik — LLM yok, halüsinasyon kaynağı yok |
| 💸 **Maliyet** | **0 API çağrısı**, API key gerekmez | Tamamen offline |
| 🧠 **Öğrenme** | 12 sorgu anısı: **11'i işe yaradı**, 1'i çıkmaz işaretlendi | Çıkmaz bir daha taranmaz |

### 🔬 vs "Kodu olduğu gibi vermek" (freetext)

| | Kodu olduğu gibi yapıştırmak | Graphify sorgusu |
|---|---|---|
| Token | **71.802 token** (tüm kod) — 128K pencereye sığar ama soru/analiz payı kalmaz; `graph.json` **167.202 token** ile hiç sığmaz | **1.694 token** — aynı pencereye **~75 kez** sığar |
| İlişki bilgisi | Yok — ajan "kim kimi çağırıyor?"yu her seferinde **sıfırdan** çıkarmak zorunda | **931 kenar** önceden hesaplı, sorguyla hazır gelir |
| Yanıt süresi | Büyük bağlamda yavaşlar, yanlış dosyaya kayabilir | Odaklı alt graf → doğru dosya, saniyeler |

## 📂 İçindekiler

| Dosya | İçerik |
|---|---|
| `docs/01-kurulum-rehberi.md` | 🔧 Graphify kurulumu ve günlük kullanım kılavuzu |
| `docs/02-gercek-etki.md` | 📊 Tüm ölçümler + örnek akışlar (teknik detay) |
| `docs/03-karsilastirma.md` | ⚖️ Graphify vs muadilleri — tarafsız artı/eksi tablosu |
| `index.html` | 🎨 Vakti olmayanlar için eğlenceli özet sayfa (tarayıcıda aç) |

## 📖 Daha Fazlası

- Teknik detay ve tüm ölçümler: [`docs/02-gercek-etki.md`](docs/02-gercek-etki.md)
- Tarafsız karşılaştırma: [`docs/03-karsilastirma.md`](docs/03-karsilastirma.md)
- Kurulum ve kullanım rehberi: [`docs/01-kurulum-rehberi.md`](docs/01-kurulum-rehberi.md)
- Eğlenceli özet: `index.html` 📺

---

*Kaynak proje: [Critic Forecast](https://github.com/alikula37/critic-forecast) — çok modelli finansal tahminleme platformu (private).*
