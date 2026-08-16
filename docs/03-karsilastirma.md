# ⚖️ Graphify vs Muadilleri — Tarafsız Karşılaştırma

> Amaç: hangi araç ne zaman mantıklı? Graphify'in artıları/eksileri + alternatiflerin
> artıları/eksileri. Sıralama/övgü yok, karşılaştırma var. 🔍

---

## 🆚 Özet Tablo

| Araç | Tür | LLM Ajanına Bağlam | Grafik/Bağlantı | Offline | Bellek (ders çıkarma) | Kurulum |
|---|---|---|---|---|---|---|
| **Graphify** | Kod grafi + ajan memnory | ✅ `query/path/explain` | ✅ god nodes + topluluklar | ✅ (code-only) | ✅ save-result/reflect | Hafif (uv ile tek CLI) |
| grep / ripgrep | Metin arama | ➖ ham satırlar | ❌ | ✅ | ❌ | ✅ anında |
| IDE + LSP (go-to-def, outline) | İnteraktif gezinme | ➖ (insan için) | kısmen | ✅ | ❌ | ✅ (IDE'de) |
| ctags / universal-ctags | Sembol indeksi | ➖ dosya:satır listesi | ❌ | ✅ | ❌ | ✅ hafif |
| Sourcegraph | Kurumsal kod arama + grafik | kısmen (API ile) | ✅ | ➖ (sunucu) | ❌ | ❌ ağır |
| CodeQL | Statik analiz / güvenlik | ❌ | kısmen (dataflow) | ✅ | ❌ | ❌ ağır + sorgu dili |
| Semgrep | Desen eşleştirme | ❌ | ❌ | ✅ | ❌ | ✅ |
| LLM + tüm kod (copy-paste) | Bağlam doldurma | ✅ ama limitli | ❌ | ✅ | ❌ | ✅ ama pratik değil |
| code2prompt / ctx | Bağlam toplayıcı | ✅ (dosya seç) | ❌ | ✅ | ❌ | ✅ |

---

## 🔬 Detaylı Karşılaştırma

### 🆚 grep / ripgrep
- **Graphify +:** Bağlamı **ilişki düzeyinde** verir — "X'i kim çağırıyor, hangi servis
  nereye bağlı" sorusuna grep'le 5-6 ayrı arama yapılıp sonuç birleştirmek gerekir;
  graphify `path A B` ile tek komutta verir. Ayrıca arama sonuçlarını "işe yaradı/çıkmaz"
  olarak hafızaya alır.
- **grep +:** Her yerde var, sıfır kurulum, binary/JS dahil her dosyada çalışır, öğrenme
  yok. Graphify yalnızca desteklenen dillerin AST'sini görür.
- **Sonuç:** grep anlık keşif, graphify yapısal keşif — çoğu projede ikisi birlikte.

### 🆚 IDE + LSP
- **Graphify +:** Ajan (LLM) için **programatik bağlam** üretir; IDE navigasyonu insan
  gözü içindir. God node'lar (örn. `predict()` 24 kenar) projenin gerçek çekirdeğini
  önceden gösterir.
- **IDE +:** Anlık, etkileşimli, hata ayıklama entegreli; graphify bunun yerine geçmez.
- **Sonuç:** İkisi farklı tüketicilere hizmet eder; graphify ajan devri için tasarlandı.

### 🆚 ctags
- **Graphify +:** ctags dosya:satır listesi verir, ilişki yok; graphify **kenarları** (çağrı,
  import, kullanım) ve topluluk yapısını verir. Ayrıca bellek sistemi var.
- **ctags +:** Daha hafif, daha geniş dil desteği, basit.
- **Sonuç:** ctags "nerede tanımlı?" sorusunun hızlı cevabı; graphify "nasıl bir arada
  çalışıyor?" sorusunun cevabı.

### 🆚 Sourcegraph
- **Graphify +:** Kurulum/sunucu yok, tek CLI; ajan entegrasyonu ve offline.
- **Sourcegraph +:** Tarayıcı arayüzü, kurumsal izinler, devasa repo'lar, daha güçlü grafik
  (kod nerede kullanılıyor) — graphify'ın ölçek/ağırlık kategorisi değil.
- **Sonuç:** Küçük/orta yerel repo + ajan iş akışı → graphify; kurumsal çok kişilik kod
  arama → Sourcegraph.

### 🆚 CodeQL / Semgrep
- **Graphify +:** Bu araçlar **kural yazmayı** gerektirir (sorgu dili / desen); graphify
  sıfır kural ile gezinme bağlamı verir. Graphify güvenlik analizi yapmaz.
- **CodeQL/Semgrep +:** Gerçek statik analiz: güvenlik açığı, dataflow, lint — graphify'ın
  yeteneği değil.
- **Sonuç:** Farklı işler. Güvenlik taraması istiyorsan CodeQL/Semgrep; "kodu anlamak"
  istiyorsan graphify.

### 🆚 LLM'e tüm kodu yapıştırmak
- **Graphify +:** 164.816 satırlık bir projeyi hiçbir bağlam penceresi taşımaz; graphify
  ajanı **sorgu başına 6,4 KB** ile doğru yere yönlendirir (bkz. `docs/02-gercek-etki.md`).
- **LLM full-context +:** Tek seferlik küçük projelerde basit; kurulum yok.
- **Sonuç:** Repo büyüdükçe graphify'nin ölçek avantajı belirginleşir.

### 🆚 code2prompt / ctx (bağlam toplayıcılar)
- **Graphify +:** Dosya seçimini ajan kendi yapar (query ile); toplayıcılar elle dosya
  listesi ister ve ilişki/öncelik bilgisi vermez. Ayrıca bellek (lessons) yok.
- **Toplayıcı +:** Basit, dil bağımsız, tek komutla tüm repo'yu tek dosyaya döker.
- **Sonuç:** "Hepsini dök, ben seçerim" ile "akıllıca yönlendir" arasındaki fark.

---

## ✅ Graphify'nin Artıları

1. **Gerçekten offline:** `--code-only` ile API key yok, LLM yok — tree-sitter yerel parse.
   (Ölçüldü: ekstraksiyon %100 extracted, yalnızca 4/931 kenar çıkarımsal.)
2. **Ajan-first tasarım:** `query / path / explain` çıktıları doğrudan LLM bağlamı olacak
   şekilde kısa ve odaklı (ölçülen tek sorgu: 6,4 KB).
3. **Bellek sistemi:** `save-result` + `reflect` → "hangi yol işe yaradı" öğrenilir;
   çıkmazlar bir daha taranmaz. (Ölçüldü: 12 anı, 11 useful, 1 dead end.)
4. **God node'lar / topluluklar:** Projenin gerçek çekirdeği (`predict()` 24 kenar,
   `get_conn()` 21 kenar) otomatik keşfedilir — yeni gelen ajan 0. saniyede "nereden
   başlamalı" bilir.
5. **Kod değişiminde yeniden doğrulama bayrağı:** Bayat bilgiye güvenilmez.
6. **Gitignore bilinci:** `graphify-out/` prompt cache riski konusunda net uyarı var.

## ❌ Graphify'nin Eksileri

1. **AST sınırı:** Yalnızca desteklenen dillerin sembolleri/kenarları; makrolar, dinamik
   çağrılar, string'deki kod, template'ler görünmez. (Bu yüzden %0.4 "inferred" kenar var.)
2. **Semantik zenginlik:** Yorum/doküman/README anlamsal ekstraksiyonu için **API key
   ister** (Gemini) — `--code-only` bunu atlar; yani "kodu anlama" offline, "doküman
   anlama" değil.
3. **Dosya ağırlığı:** `graphify-out/` 3,9 MB + her ekstrakt sonrası `graph.json` 599 KB
   yeniden yazılır; disk ve git ignore disiplini şart.
4. **Graf eskimesi:** Kod değişince `graphify update .` çalıştırılmazsa graf bayatlar
   (yeniden doğrulama bayrağı bunu azaltır ama ortadan kaldırmaz).
5. **Bağımlılık:** CLI kurulumu için `uv` gerekir (pip de olur); node tabanlı plugin
   entegrasyonu (opencode) node_modules getirir.
6. **Öğrenme eğrisi:** `save-result`/`reflect` disiplini (hangi sorgu useful/dead_end)
   sürdürülmezse bellek sistemi çöp toplar.

---

## 🧭 Ne Zaman Hangisi? (tek paragraf)

**Küçük tek-dosya işler** için grep yeter; **insan gözüyle etkileşimli gezinme** için IDE;
**güvenlik/doğrulama** için CodeQL/Semgrep; **kurumsal çok kişilik arama** için Sourcegraph.
Ama **LLM ajanının bir repo'da hızlı ve doğru çalışmasını** istiyorsan — özellikle
tekrarlayan oturumlarda (her seferinde sıfırdan taramak yerine) ve öğrenen bir hafızayla —
graphify bu kategoriye oturuyor. Hiçbiri diğerinin yerine geçmiyor; **soru türü aracı
belirler.**
