# MoM Uygulaması — Devir Planı (Claude Desktop oturumu için)

Bu belge, Claude Code (web) oturumunda hazırlandı ve masaüstündeki Claude oturumuna
**tek başına yeterli bir brif** olarak devredilmek üzere yazıldı. Web oturumu ağ
politikası yüzünden Notability bağlantısını açamadı, ses dosyasını yazıya dökemedi.
Masaüstü oturumu bu kısıtlara tabi değil; plan bu varsayımla kuruldu.

Belgedeki güven etiketleri: **[Kesin]** sağlam kanıt, **[Muhtemel]** güçlü çıkarım,
**[Tahmin]** doğrulanması gereken boşluk doldurma. Masaüstü oturumu **[Muhtemel]**
ve **[Tahmin]** etiketli her teknik iddiayı uygulamadan önce güncel belgelerden
doğrulamalı.

---

## 0. Masaüstü oturumuna ilk komut (kopyala-yapıştır)

```
mom/PLAN.md dosyasını baştan sona oku. Bölüm 3'teki girdileri benimle birlikte topla,
Faz 0'ı çalıştır (şablon çıkarımı + yerel transkripsiyon kıyası), Bölüm 8'deki açık
soruları bana sor, cevaplarımla Faz 1'e geç. Kararlarını gerekçelendir; katılmadığın
yerde itiraz et. Türkçe çalış. Örnek not ve kayıt dosyalarını mom/ornek/ altına koy;
bu klasör .gitignore'da, depoya girmeyecek.
```

---

## 1. Amaç

Toplantı ses kaydından veya hazır transkriptten, kullanıcının kendi not biçimine uyan
**yapılandırılmış toplantı tutanağı (MoM, Minutes of Meeting)** üreten; düzenlenip
paylaşılabilen; geçmişi ve **toplantılar arası açık aksiyonları** cihazda tutan uygulama.

Asıl katma değer transkripsiyon değil (o bir emtia ve en riskli parça); değer şurada:
1. Tutarlı, kullanıcının alıştığı şablonda tutanak.
2. Aksiyonların sorumlu ve terminle çıkarılması, toplantılar arası takibi.
3. Tek dokunuşla paylaşım (Mail, WhatsApp, Markdown).

---

## 2. Depo ve konvansiyonlar (masaüstü oturumu bunları bilmeli)

Depo: `bektaron/interval-sayac`. Çalışma dalı: `claude/mom-app-plan-4dpw2h`
(bu plan o dalda; taslak PR açık). Uygulama `mom/` klasöründe yaşayacak.

Depodaki iki mevcut uygulama aynı deseni izler; MoM da izlesin:

| Konu | Mevcut desen | Nerede |
|---|---|---|
| Yapı | Tek dosya `index.html` (CSS+JS gömülü), derleme yok, bağımlılık yok, vanilla JS | `index.html`, `yemek/index.html` |
| Dil / tema | Türkçe arayüz, koyu tema, CSS değişkenleri `--bg --panel --panel2 --line --txt --muted --accent` | `yemek/index.html` `:root` |
| PWA | `manifest.json` (`start_url ./index.html`, `scope ./`, standalone, portrait), `sw.js`: HTML için önce ağ, varlıklar için önce önbellek, **çapraz kökene dokunma** | `yemek/sw.js`, `yemek/manifest.json` |
| İkonlar | `icon-192.png`, `icon-512.png` (maskable), `apple-touch-icon.png` | `yemek/` |
| iOS meta | `apple-mobile-web-app-capable`, `status-bar-style black-translucent`, `viewport-fit=cover`, `env(safe-area-inset-*)` | `yemek/index.html` head |
| Depolama | `load(k,d)` / `save(k,v)` localStorage yardımcıları; kota hatasında Türkçe uyarı | `yemek/index.html` ~716 |
| Claude çağrısı | `claudeRequest()`: `https://api.anthropic.com/v1/messages`, model `claude-opus-5`, başlıklar `x-api-key`, `anthropic-version: 2023-06-01`, **`anthropic-dangerous-direct-browser-access: true`**; önce `anthropic-beta: server-side-fallback-2026-07-01` + `fallbacks:"default"`, 400 gelirse sade istek; `pause_turn` döngüsü; `stop_reason === "refusal"` kontrolü; `output_config:{effort}` | `yemek/index.html` ~1004-1050 |
| Anahtar | Kullanıcı Ayarlar'da girer, yalnızca cihazda (localStorage), ücretlendirme uyarısı metni | `yemek/index.html` ~365-372 |

Barındırma: [Tahmin] GitHub Pages, `main` dalından. Masaüstü oturumu depo ayarlarından doğrulasın.

Commit mesajları Türkçe, kısa, ne değiştiğini söyleyen tarzda (bkz. `git log`).

---

## 3. Girdiler ve Faz 0 (masaüstünde ilk iş, ~yarım gün)

### 3.1 Toplanacak girdiler

Hepsi `mom/ornek/` altına (**.gitignore'da**; toplantı içeriği gizli olabilir):

| Girdi | Nasıl | Hedef dosya |
|---|---|---|
| Kullanıcının örnek MoM notu | Notability paylaşım bağlantısı: `https://notability.com/app/note/ac250250-ebaf-4d19-be4a-c8b9a0db9a59` — tarayıcıda aç, metni al. Açılmazsa kullanıcıdan Notability'den metin/PDF dışa aktarmasını iste. | `mom/ornek/notlar.md` |
| Toplantı ses kaydı | iPhone **Sesli Notlar** → Paylaş → AirDrop / iCloud Drive / Dosyalara Kaydet. Notability kaydı da olur ama Sesli Notlar tercih (iOS 17+ "Kaydı İyileştir" gürültüyü azaltır). | `mom/ornek/kayit.m4a` |
| iOS'un kendi transkripti (varsa) | Sesli Notlar'da kaydı aç, tırnak simgesine dokun. **Türkçe çıkıyor mu?** [Tahmin: iOS 18+ transkript Türkçe desteği belirsiz] Çıkıyorsa kopyala. | `mom/ornek/transkript-ios.txt` |
| Kayıt bilgisi | Süre, dosya boyutu, konuşmacı sayısı, ortam (oda/telefon/online) | plan notuna |

### 3.2 Şablon çıkarımı

`notlar.md`'yi incele ve şunları yaz:
- `mom/sablon.md`: tutanağın bölümleri, sırası, başlık dili, madde uzunluğu, ton
  (resmi/özet), tarih/isim biçimleri, aksiyon gösterimi (sorumlu, termin nasıl yazılmış).
- Bölüm 5.4'teki JSON şemasını bu şablona göre **düzelt**. Şema taslaktır, notlara uymalı.
- Kullanıcıyla teyit: "Şablon bu mu, eksik/fazla bölüm var mı?"

### 3.3 Yerel transkripsiyon kıyası (Mac'te)

Amaç: hangi STT yolunun Türkçe toplantı kaydında yeterli olduğunu **kullanıcının kendi
kaydıyla** görmek. API anahtarı gerektirmeyen yol:

```bash
# Apple Silicon (önerilen) [Muhtemel]
pip install mlx-whisper
mlx_whisper mom/ornek/kayit.m4a --model mlx-community/whisper-large-v3-turbo \
  --language tr --output-dir mom/ornek --output-format txt

# Alternatifler: brew install whisper-cpp (ggml large-v3-turbo modeli), veya pip install faster-whisper
```

Kıyasla: (a) whisper çıktısı, (b) varsa iOS transkripti, (c) kullanıcının notları.
Rapor: isim/terim hataları, konuşmacı karışması, bariz atlamalar. Bu rapor STT kararının
dayanağı (Bölüm 4).

### 3.4 Ürün biçimi kararı

Varsayım: **PWA** (telefondan kullanım, mevcut iki uygulamayla aynı model). Eğer kullanıcı
"sadece Mac'te çalışsın, ses cihazdan çıkmasın" derse Bölüm 4'teki C yolu (yerel whisper +
küçük yerel sunucu) ürün olur; arayüz yine aynı `index.html` ama STT adaptörü yerel
sunucuya bakar. Bu kararı Faz 0 sonunda kullanıcıya sor.

---

## 4. Transkripsiyon (STT) stratejisi

Claude Messages API ses girdisi almaz; STT ayrı bir bileşen olmak zorunda. **[Kesin]**

| Yol | Ne | Artı | Eksi |
|---|---|---|---|
| **A. iOS transkripti yapıştır** | Sesli Notlar'ın cihaz içi transkripti → uygulamaya metin | Sıfır maliyet, ses cihazdan çıkmaz, en basit MVP | Türkçe desteği belirsiz [Tahmin]; konuşmacı ayrımı yok |
| **B. Tarayıcıdan API** | Uygulama ses dosyasını doğrudan STT servisine gönderir | Telefonda uçtan uca çalışır | Anahtar yönetimi, ses üçüncü tarafa gider, maliyet |
| **C. Yerel whisper (Mac)** | `mlx-whisper`/`whisper.cpp` + yerel küçük sunucu | Ücretsiz, gizli, Türkçe kalitesi iyi (large-v3) [Muhtemel] | Telefondan kullanılamaz, Mac açık olmalı |

B içinde sağlayıcı seçimi:

| Sağlayıcı | Not |
|---|---|
| **Deepgram** (öneri) | `POST https://api.deepgram.com/v1/listen`, `Authorization: Token <key>`, gövde ham ses; parametreler `model=nova-3` (Türkçe yoksa `nova-2`), `language=tr`, `diarize=true`, `smart_format=true`, `utterances=true`, `paragraphs=true`. Tarayıcıdan CORS çalışır, dosya sınırı büyük (parçalama gerekmez), konuşmacı ayrımı var. [Muhtemel] Türkçe kalitesi Faz 0'da ölçülmeli. [Tahmin] |
| **OpenAI** | `POST https://api.openai.com/v1/audio/transcriptions` multipart; `model=gpt-4o-transcribe` veya `whisper-1`; `language=tr`. Dosya başına ~25 MB sınırı → 1 saatlik m4a sınırda, tarayıcıda parçalama gerekir (m4a bayt bölünemez; Web Audio ile çözüp yeniden kodlamak lazım). Konuşmacı ayrımı için `gpt-4o-transcribe-diarize` var mı, kontrol et. [Tahmin] |

**Karar önerisi:** A çalışıyorsa MVP A ile başlar (STT adaptörü "metin yapıştır"). A yoksa
B/Deepgram. Her durumda STT bir **adaptör arayüzü** arkasında olsun ki sağlayıcı
değişebilsin:

```js
// mom/index.html içinde
// transcribe(file, {language, diarize, onProgress}) -> {provider, language, text, segments:[{start,end,speaker,text}]}
const STT = { deepgram: {...}, openai: {...}, paste: {...} /* metin yapıştır */ };
```

---

## 5. Faz 1 — MVP (1-2 gün iş)

### 5.1 Dosyalar
`mom/index.html`, `mom/sw.js` (önbellek adı `mom-v1`), `mom/manifest.json`
(`name: "Toplantı Tutanağı"`, `short_name: "MoM"`), `mom/icon-192.png`, `mom/icon-512.png`,
`mom/apple-touch-icon.png` (yemek ikonlarının stilinde, başka renk/simge).

### 5.2 Ekranlar
1. **Toplantılar** (ana): liste (tarih, başlık, durum rozeti: transkript / tutanak / paylaşıldı), arama, "Yeni toplantı".
2. **Yeni toplantı**: başlık, tarih (varsayılan bugün), katılımcılar (virgülle), girdi seçimi:
   - "Ses dosyası seç" (`<input type=file accept="audio/*">`; iPhone'da Dosyalar'dan m4a),
   - "Transkript / not yapıştır" (textarea),
   - "Kendi notlarım" (isteğe bağlı ek textarea; tutanağa bağlam olarak girer).
3. **İşleniyor**: aşama göstergesi (Yükleniyor → Yazıya dökülüyor → Tutanak yazılıyor), iptal.
4. **Transkript**: konuşmacı etiketli paragraflar; etikete dokununca isim ata (tüm transkripte uygulanır); metin düzenlenebilir; "Tutanağı yeniden üret".
5. **Tutanak**: şablon bölümleri (Bölüm 5.4), her bölüm düzenlenebilir; aksiyonlar tablo (sorumlu, termin, öncelik, tamamlandı kutusu).
6. **Paylaş**: Markdown kopyala, `navigator.share` (metin; destek varsa `.md` dosya), `.md` indir, `mailto:` (konu+gövde; uzunluk sınırına dikkat).
7. **Aksiyonlar** sekmesi: tüm toplantılardan açık aksiyonlar; sorumluya/termine göre sırala; kapatınca ilgili tutanağa işlenir.
8. **Ayarlar**: Anthropic anahtarı; STT yolu (yapıştır / Deepgram / OpenAI) ve anahtarı; tutanak dili (Türkçe öntanımlı, "toplantı dilini izle"); model ve `effort`; "Sesi cihazda sakla" (öntanımlı açık); depolama kullanımı ve "tümünü sil".

### 5.3 Veri modeli ve depolama
- **IndexedDB** `mom-db`: `meetings` (aşağıdaki nesne), `audio` (meetingId → Blob). localStorage ses için yetmez. **[Kesin]**
- **localStorage**: ayarlar ve anahtarlar (`mom.settings`).

```json
{
  "id": "m_...", "title": "", "date": "2026-09-26", "participants": ["..."],
  "source": {"type": "audio|text", "fileName": "", "durationSec": 0, "sizeBytes": 0},
  "userNotes": "",
  "transcript": {"provider": "deepgram|openai|paste|ios", "language": "tr",
                 "speakerNames": {"0": "Ad"}, "segments": [{"start": 0, "end": 0, "speaker": "0", "text": ""}], "text": ""},
  "mom": { "...": "Bölüm 5.4 şeması" },
  "status": "new|transcribed|drafted|shared",
  "createdAt": 0, "updatedAt": 0
}
```

Safari, kullanılmayan sitelerin depolamasını 7 gün sonra silebilir; ana ekrana eklenen PWA'da
bu kural gevşer. [Muhtemel] Ayarlar'da "Tutanakları .json olarak dışa/içe aktar" ekle.

### 5.4 Tutanak üretimi (Claude)

İstek biçimi: `yemek/index.html` `claudeRequest()` deseninin aynısı (başlıklar, fallback
denemesi, refusal kontrolü) + **yapılandırılmış çıktı** + uzun girdide **akış**:

```json
{
  "model": "claude-opus-5",
  "max_tokens": 16000,
  "stream": true,
  "output_config": {
    "effort": "high",
    "format": { "type": "json_schema", "schema": { "...": "aşağıdaki şema" } }
  },
  "system": "…Bölüm 5.5…",
  "messages": [{ "role": "user", "content": "…meta + transkript + kullanıcı notları…" }]
}
```

- Akışta `content_block_delta` / `text_delta` parçalarını birleştir; ilk `text` bloğu
  geçerli JSON'dur (`output_config.format` bunu garanti eder). [Kesin, skill belgesinden]
- `thinking` parametresini gönderme (Opus 5'te adaptif düşünme öntanımlı). [Kesin]
- 1 saatlik Türkçe transkript ≈ 25-35K token; tek istekte sığar, parçalama gerekmez. [Muhtemel]
- Hata mesajları Türkçe; 120 sn zaman aşımı yerine akışta "son parça zamanı"na göre 60 sn sessizlik zaman aşımı.

**Taslak şema** (Faz 0'da `notlar.md`'ye göre düzeltilecek; `additionalProperties:false`
ve tüm alanlar `required` olmalı):

```json
{
  "type": "object",
  "properties": {
    "baslik": {"type": "string"},
    "tarih": {"type": "string"},
    "katilimcilar": {"type": "array", "items": {"type": "object",
      "properties": {"ad": {"type": "string"}, "rol": {"type": "string"}},
      "required": ["ad", "rol"], "additionalProperties": false}},
    "gundem": {"type": "array", "items": {"type": "string"}},
    "konular": {"type": "array", "items": {"type": "object",
      "properties": {"baslik": {"type": "string"}, "ozet": {"type": "string"},
                     "tartisma": {"type": "array", "items": {"type": "string"}}},
      "required": ["baslik", "ozet", "tartisma"], "additionalProperties": false}},
    "kararlar": {"type": "array", "items": {"type": "object",
      "properties": {"karar": {"type": "string"}, "gerekce": {"type": "string"}, "konu": {"type": "string"}},
      "required": ["karar", "gerekce", "konu"], "additionalProperties": false}},
    "aksiyonlar": {"type": "array", "items": {"type": "object",
      "properties": {"aksiyon": {"type": "string"}, "sorumlu": {"type": "string"},
                     "termin": {"type": "string"}, "oncelik": {"type": "string", "enum": ["yuksek", "orta", "dusuk"]},
                     "konu": {"type": "string"}},
      "required": ["aksiyon", "sorumlu", "termin", "oncelik", "konu"], "additionalProperties": false}},
    "acik_sorular": {"type": "array", "items": {"type": "string"}},
    "sonraki_toplanti": {"type": "string"},
    "belirsizlikler": {"type": "array", "items": {"type": "string"}}
  },
  "required": ["baslik", "tarih", "katilimcilar", "gundem", "konular", "kararlar",
               "aksiyonlar", "acik_sorular", "sonraki_toplanti", "belirsizlikler"],
  "additionalProperties": false
}
```

### 5.5 Sistem istemi (taslak, şablona göre uyarlanacak)

```
Sen deneyimli bir toplantı sekreterisin. Girdi: toplantı transkripti (konuşmacı etiketli
olabilir, yazıya dökme hataları içerebilir), toplantı bilgileri ve katılımcının kendi notları.
Çıktı: {tutanak dili} dilinde, verilen şemaya uyan tutanak.

Kurallar:
- Transkriptte olmayan hiçbir şeyi ekleme. Emin olmadığın isim, rakam ve tarihleri
  "belirsizlikler" listesine yaz; metinde tahmin yürütme.
- Aksiyonun sorumlusu söylenmediyse "atanmadı", termini yoksa "belirtilmedi" yaz.
- Katılımcının notları transkriptle çelişirse notları esas al ve bunu "belirsizlikler"e ekle.
- Yazıya dökme hatalarını bağlamdan düzelt (özel isimler, teknik terimler), ama anlamı değiştirme.
- Kısa ve somut yaz: her madde tek fikir, gereksiz nezaket ve dolgu yok.
- Şablon: {mom/sablon.md içeriği}
```

### 5.6 Tutanak → metin
`mom` nesnesini Markdown'a çeviren tek bir `momToMarkdown()`; ekranda aynı Markdown'ın
basit HTML render'ı; paylaşımda düz metin. Şablon `sablon.md`'deki sıra ve başlıklarla.

### 5.7 Kabul kriterleri (MVP)
- iPhone Safari'de Sesli Notlar'dan kaydedilmiş 60 dakikalık m4a seçilip işlenebiliyor.
- Yapıştırılan transkriptten tutanak üretiliyor; JSON parse hatası yok.
- Aksiyonlar sorumlu/terminle çıkıyor; Aksiyonlar sekmesinde kapatılabiliyor.
- Markdown kopyalama ve Web Share çalışıyor (iOS ve masaüstü Chrome).
- Çevrimdışıyken uygulama açılıyor, geçmiş görülüyor, API isteği anlaşılır hata veriyor.
- Anahtarlar yalnızca localStorage'da; Ağ sekmesinde yalnız api.anthropic.com ve seçilen STT alanına istek gidiyor.
- Depolama kullanımı Ayarlar'da görünüyor; ses silinebiliyor.

---

## 6. Faz 2 ve Faz 3

**Faz 2 (1-2 gün):**
- Uygulama içi kayıt (`MediaRecorder`; iOS'ta `audio/mp4`). iOS'ta ekran kilitlenince
  kaydın kesilme riski var [Muhtemel]; bu yüzden MVP'de Sesli Notlar esas.
- Konuşmacı → isim eşlemesini katılımcı listesinden öner.
- Transkript ile ses oynatıcı senkronu (segment'e dokun → o saniyeden çal).
- Tutanak bölümünü tek başına yeniden üret ("kararları yeniden yaz").
- `.docx` dışa aktarma (tarayıcıda küçük bir docx üreteci; bağımlılıksız kalınacaksa minimal OOXML).
- JSON yedek dışa/içe aktarma (Faz 1'de yoksa).

**Faz 3 (isteğe bağlı):** takvimden toplantı bilgisi çekme, e-posta gönderimi, çok dilli
tutanak, birden fazla şablon.

---

## 7. Güvenlik ve gizlilik

- Başkalarının sesi üçüncü taraf servislere gidiyor. Kurumsal toplantılarda katılımcı
  bilgilendirmesi/onayı (KVKK) kullanıcının sorumluluğu; Ayarlar'da tek satır uyarı olsun.
- Anahtarlar cihazda; paylaşılan cihazda risk. Sunucu yok, dolayısıyla anahtar döndürme de yok.
- `mom/ornek/` depoya girmez. Örnek transkript veya tutanak commit'lenmez.
- Service worker çapraz kökeni önbelleğe almaz (yemek deseni). API yanıtları önbelleğe girmez.

---

## 8. Kullanıcıya sorulacak açık sorular (Faz 0 sonunda)

| Soru | Öntanımlı (cevap gelmezse) |
|---|---|
| Sesli Notlar Türkçe transkript veriyor mu? | Hayır varsay → B/Deepgram |
| STT sağlayıcı tercihi ve mevcut hesaplar? | Deepgram, OpenAI adaptörü yedek |
| Ürün PWA mı, sadece Mac mi? | PWA |
| Ses cihazda saklansın mı? | Evet, silinebilir |
| Tutanak dili? | Türkçe öntanımlı, ayarlanabilir |
| Şablon: örnek nottaki bölümler yeterli mi, eksik var mı? | Örnek not + Bölüm 5.4 |
| Aksiyon takibi toplantılar arası olsun mu? | Evet |

---

## 9. Maliyet tahmini (1 saatlik toplantı)

| Kalem | Tahmin |
|---|---|
| STT (Deepgram/OpenAI) | 0,25 - 0,40 USD [Muhtemel] |
| Claude Opus 5 tutanak | 0,20 - 0,35 USD [Tahmin] |
| A veya C yolu | yalnız Claude kalemi |

---

## 10. Web oturumunda yapılanlar ve yapılamayanlar

Yapıldı: depo konvansiyonları çıkarıldı, Claude API istek biçimleri skill belgesinden
doğrulandı (yapılandırılmış çıktı, akış, fallback), plan yazıldı, `mom/ornek/` gitignore'a eklendi.

Yapılamadı (ağ politikası): Notability notu okunamadı; ses yazıya dökülemedi
(Deepgram, OpenAI, Hugging Face ve Whisper model sunucuları engelli). Drive'daki Notability
yedeklerine bakıldı, MoM notu orada yok.
