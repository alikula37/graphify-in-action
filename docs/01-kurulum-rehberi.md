# 🔧 Graphify Kurulum ve Kullanım Rehberi

Bu kılavuz, bir projeye graphify'ı nasıl kurup etkin kullanacağını adım adım anlatır.
Amacı tek cümlede: **kod tabanını her oturumda baştan taramadan**, ajanın (LLM asistanın)
**token/context kullanımını azaltarak** hızlı ve doğru çalışmasını sağlamak.

---

## 🎯 Karar Kriteri

Graphify kurulmadan önce şu kriterle değerlendirilir:

> Eğer graphify işe yaramazsa, okuma/token/context kullanımını olumsuz etkiliyorsa veya
> kötüye götürüyorsa **kurulmaz**.

Bu projede kriter sağlandı ve kurulum yapıldı — ölçülmüş etkisi için
[`02-gercek-etki.md`](02-gercek-etki.md) dosyasına bak.

## 📦 Kurulum

```bash
# Proje kökünde, strict modda kur
graphify install --project --strict

# Kaynak: https://github.com/Graphify-Labs/graphify
```

### 🧊 Strict Mode Ne Yapar?

- Oturumun **ilk ham kaynak okumasını bloklar** ve ajanı doğrudan grafa yönlendirir.
- Sorgudan sonra tekrar nudge (yönlendirme) moduna döner — **takılı kalmaz**, sürekli engel
  olmaz.

## 🔍 Nasıl Çalışır?

| Özellik | Açıklama |
|---|---|
| 🌳 **Tree-sitter ile yerel parse** | Kod, **LLM'e uğramadan** yerel olarak AST'ye çevrilir |
| 🛰️ **Tamamen offline** | `--code-only` ile API key **gerekmez**, internete bağımlılık yok |
| 💾 **Kalıcı bilgi** | Ekstrakt edilen graf, her oturumda yeniden taramayı gereksiz kılar |
| ⚡ **Hızlı güncelleme** | Kod değişince `graphify update .` — AST üzerinden saniyeler içinde tazelenir |

## 🧠 Öğrenme Döngüsü: save-result + reflect

```bash
# 1. Sorgu sonucunu kaydet — işe yaradı mı, çıkmaz mıydı?
graphify save-result --question "..." --answer "..." --type query --outcome useful|dead_end

# 2. Reflect: birikmiş dersleri derle, node'ları işaretle
graphify reflect
```

- **save-result:** hangi sorgunun işe yaradığını, hangisinin **çıkmaz (dead end)** olduğunu
  kaydeder.
- **reflect:** kayıtları derleyip node'ları **güvenilir (preferred)** ya da **şüpheli
  (tentative)** diye işaretler; çıkmazları "tekrar keşfetme" listesine alır.
- **Kod değişince:** ilgili derse **"yeniden doğrula" (revalidate)** bayrağı basılır —
  bayat bilgiye güvenilmez.

## ⚠️ Kritik Uyarı: .gitignore

> `graphify-out/` klasörünü `.gitignore`'a eklemezsen, her `extract` sonrası bu klasör
> (ve içindeki büyük JSON'lar) prompt cache'ini düşürebilir / şişirebilir. **Mutlaka
> ignore edilmelidir.**

## 🧭 Günlük Kullanım

| Komut | Ne yapar? |
|---|---|
| `graphify query "<soru>"` | Odaklı alt graf döndürür — kodla ilgili soruların ilk adımı |
| `graphify path "<A>" "<B>"` | İki kavram arasındaki ilişkiyi gösterir |
| `graphify explain "<kavram>"` | Tek bir kavramın açıklamasını getirir |
| `graphify update .` | Kod değişikliklerinden sonra grafi tazeler |
| `graphify reflect` | Birikmiş dersleri derler, güvenilir/şüpheli node işaretler |

> 💡 İpucu: Sorgu sonuçları genellikle ham kaynak taramasından çok daha küçük ve odaklıdır —
> bu projede tek bir sorgu **6,4 KB** alt graf döndürürken kod tabanı **164.816 satırdır**.

---

## 🤖 Tek Adımda Kurulum (AI Agent Promptu)

Aşağıdaki prompt'u AI asistanına (ajanına) vererek graphify'ın kurulumunu, karar kontrolünü
ve temel iş akışını tek adımda yaptırabilirsin. Prompt, kullanıcının **son haliyle**
kopyalayıp yapıştırmasına hazırdır:

```text
Bu projeye graphify'ı kur.

Kurulum:
- Kaynak: https://github.com/Graphify-Labs/graphify
- Komut: graphify install --project --strict
- Ayrıca graphify-out/ klasörünü .gitignore'a ekle (eklenmemişse) —
  aksi halde her extract prompt cache'ini şişirebilir.

Karar kriteri:
- Kurulumdan önce şunu değerlendir: graphify işe yaramazsa, benim okuma/token/context
  kullanımımı olumsuz etkiliyorsa veya kötüye götürüyorsa KURMA ve bana nedenini anlat.
- Kod tarafı için API key gerekmez (tree-sitter ile yerel parse, LLM'e uğramaz);
  --code-only ile tamamen offline çalışır. Eğer durum buysa kur.

Kurulumdan sonra yapılacaklar:
1. Ekstraktı çalıştır ve grafi oluştur.
2. Strict mode'un nasıl çalıştığını doğrula: oturumun ilk ham kaynak okumasını
   bloklayıp grafa yönlendirmeli, sonra tekrar nudge moduna dönmeli (takılmamalı).
3. Bana bu iş akışını özetle: kodla ilgili sorularımda önce graphify query/path/explain
   kullanacaksın; her önemli sorgunun sonucunu "işe yaradı (useful)" ya da
   "çıkmaz (dead_end)" olarak save-result ile kaydedeceksin; reflect ile birikmiş
   dersleri derleyip node'ları güvenilir/şüpheli diye işaretleyeceksin.
4. Kod değişikliklerinden sonra graphify update . çalıştırarak grafi taze tutacaksın
   (bayat bilgiye güvenmeyeceksin).
```

> 💡 Bu prompt, bu rehberdeki tüm maddeleri tek mesaja sığdırır: karar kriteri,
> strict mode, offline kurulum, save-result/reflect döngüsü ve .gitignore uyarısı.
