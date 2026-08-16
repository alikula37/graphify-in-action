# 🧠 Graphify in Action — Bu Projede Gerçek Etki

> **Graphify, 164.816 satırlık bir projeyi sorgu başına 6,4 KB ile yönetmeyi nasıl başardı?**
> Hepsi **ölçülmüş gerçek rakamlar** — uydurma yok. 🧪

![Akış](assets/flow.svg)

## 📂 İçindekiler

| Dosya | İçerik |
|---|---|
| `docs/01-kurulum-rehberi.md` | 🔧 Graphify kurulumu ve günlük kullanım kılavuzu |
| `docs/02-gercek-etki.md` | 📊 Ölçülmüş rakamlar + bu projede yaşanmış örnek akışlar |
| `docs/03-karsilastirma.md` | ⚖️ Graphify vs muadilleri — tarafsız artı/eksi tablosu |
| `index.html` | 🎨 Vakti olmayanlar için eğlenceli özet sayfa (tarayıcıda aç) |

## 🚀 Hızlı Bakış (hepsi ölçüldü, 2026-08-16)

- 📏 Kod tabanı: **324 dosya · 164.816 satır**
- 🗺️ Kod grafi: **562 node · 931 kenar · 44 topluluk** (599 KB `graph.json`)
- 🎯 Tek sorgu: **6.430 bayt** odaklı alt graf — toplam bilginin ~%0.1'i
- 🔬 Ekstraksiyon: **%100 extracted · 0 API çağrısı** (tree-sitter, offline)
- 🧠 Hafıza: **12 anı · 11 useful · 1 dead end** → reflect: `qra.py` 3×, `jobs.py` 2× öncelikli
- 🏆 God node'lar: `predict()` 24 kenar, `get_conn()` 21 kenar, `_post()` 19 kenar

## 🧪 Örnek akış: "settings sayfası" 🚧→💡

1. Sorgu: "settings sayfasındaki ögeleri geniş yap" → graf bulamadı → **dead end**
2. Sorgu: "settings sayfası hangi portta" → **8080 = eski proje, bizim UI 8081**
3. `reflect` → çıkmaz bir daha taranmıyor. 2 sorgu, ~2 dakika.

## 📖 Daha Fazlası

- Teknik detay ve tüm ölçümler: [`docs/02-gercek-etki.md`](docs/02-gercek-etki.md)
- Tarafsız karşılaştırma: [`docs/03-karsilastirma.md`](docs/03-karsilastirma.md)
- Kurulum ve kullanım rehberi: [`docs/01-kurulum-rehberi.md`](docs/01-kurulum-rehberi.md)
- Eğlenceli özet: `index.html` 📺

---

*Kaynak proje: [Critic Forecast](https://github.com/alikula37/critic-forecast) — çok modelli finansal tahminleme platformu (private).*
