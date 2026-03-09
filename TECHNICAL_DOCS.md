# 📘 Teknik Dokümantasyon — Video Transkript Aracı

## Genel Bakış

Bu araç, OpenAI Whisper modelini kullanarak MP4 video dosyalarından konuşma transkripsiyonu çıkarır ve sonuçları zaman damgalı olarak PDF ve TXT formatlarında kaydeder.

## Mimari

```
MP4 Video
    │
    ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   FFmpeg      │────▶│   Whisper    │────▶│  Segment Verisi  │
│ (Ses Çıkarma) │     │ (Transkript) │     │ (start, end, text)│
└──────────────┘     └──────────────┘     └────────┬─────────┘
                                                    │
                                          ┌─────────┴─────────┐
                                          ▼                   ▼
                                    ┌──────────┐       ┌──────────┐
                                    │ TXT Çıktı │       │ PDF Çıktı │
                                    │ (fpdf2)   │       │ (fpdf2)   │
                                    └──────────┘       └──────────┘
```

## Bağımlılıklar

| Paket           | Versiyon  | Amaç                              |
|----------------|-----------|-------------------------------------|
| `openai-whisper`| ≥20230314 | Konuşma tanıma (ASR) motoru        |
| `fpdf2`        | ≥2.7      | PDF oluşturma (Unicode destekli)    |
| `torch`        | ≥1.10     | Whisper'ın derin öğrenme altyapısı  |
| `ffmpeg`       | ≥4.0      | Ses/video codec işlemleri           |

> `torch`, `tiktoken`, `numba` gibi alt bağımlılıklar `openai-whisper` ile otomatik yüklenir.

## Dosya Yapısı

```
trasncript/
├── transcribe.py          # Ana script
├── README.md              # Kullanım kılavuzu
├── TECHNICAL_DOCS.md      # Bu dosya
└── *.mp4                  # İşlenecek video dosyaları (kullanıcı ekler)
```

## Modül Açıklamaları

### `transcribe.py`

Tek dosyalı uygulama. 6 ana fonksiyon içerir:

---

### `find_mp4_file() → str`

Mevcut çalışma dizinindeki ilk `.mp4` dosyasını bulur.

- **Parametre:** Yok
- **Dönüş:** MP4 dosya yolu
- **Hata:** Dosya bulunamazsa `sys.exit(1)`
- **Kullanım:** `glob.glob("*.mp4")` ile dosya arar

---

### `transcribe_video(video_path, model_name, language) → dict`

Whisper modelini yükler ve videoyu transkript eder.

- **Parametreler:**
  - `video_path` (str) — MP4 dosya yolu
  - `model_name` (str) — Whisper model adı: `tiny | base | small | medium | large`
  - `language` (str | None) — ISO 639-1 dil kodu, `None` ise otomatik algılar
- **Dönüş:** Whisper result dict'i:
  ```python
  {
      "text": "Tam transkript metni...",
      "segments": [
          {"start": 0.0, "end": 5.2, "text": " Merhaba."},
          {"start": 5.2, "end": 10.8, "text": " Bugün konumuz..."},
          ...
      ],
      "language": "tr"
  }
  ```
- **Not:** Model ilk çalışmada `~/.cache/whisper/` dizinine indirilir.

---

### `format_timestamp(seconds) → str`

Saniye cinsinden süreyi okunabilir formata çevirir.

- **Parametre:** `seconds` (float) — Saniye
- **Dönüş:** `"HH:MM:SS"` formatında string
- **Örnek:** `format_timestamp(3672.5)` → `"01:01:12"`

---

### `format_segments(segments) → list[str]`

Whisper segment listesini zaman damgalı metin satırlarına dönüştürür.

- **Parametre:** `segments` (list[dict]) — Whisper segment dizisi
- **Dönüş:** `["[00:00:00 --> 00:00:05]  Merhaba.", ...]`
- **Not:** Boş metinli segmentler filtrelenir

---

### `sanitize_text(text) → str`

PDF'de sorun çıkarabilecek karakterleri temizler.

- **İşlemler:**
  1. ASCII kontrol karakterlerini kaldırır (`\x00-\x08`, `\x0b`, `\x0c`, `\x0e-\x1f`, `\x7f`)
  2. 50+ karakter uzunluğundaki kesintisiz kelimeleri 40 karakterlik parçalara böler
- **Neden:** `fpdf2`, satıra sığmayan kelimeler için `FPDFException: Not enough horizontal space` hatası verir

---

### `save_as_pdf(text, output_path, video_name)`

Zaman damgalı transkript satırlarını PDF dosyasına yazar.

- **Parametreler:**
  - `text` (list[str]) — Zaman damgalı satır listesi
  - `output_path` (str) — PDF çıktı yolu
  - `video_name` (str) — Başlıkta gösterilecek dosya adı
- **Font Seçimi:** Windows sistem fontlarını sırayla dener: Arial → Calibri → Segoe UI → Tahoma
- **PDF Yapısı:**
  - Başlık (16pt, ortalı)
  - Ayırıcı çizgi
  - Segment satırları (11pt, satır aralığı 7pt + 2pt boşluk)

## FFmpeg Entegrasyonu

Script başlangıcında FFmpeg yolu otomatik olarak `os.environ["PATH"]`'e eklenir:

```python
FFMPEG_DIR = r"C:\ffmpeg\ffmpeg-8.0.1-essentials_build\bin"
```

Bu sayede Whisper'ın dahili `load_audio()` fonksiyonu FFmpeg'i `subprocess.run` ile çağırabilir. FFmpeg şu işlemi yapar:

```bash
ffmpeg -nostdin -threads 0 -i <video> -f s16le -ac 1 -acodec pcm_s16le -ar 16000 -
```

Yani videodan 16kHz mono PCM ses verisini çıkarır ve Whisper'a iletir.

## Whisper İşlem Akışı

1. **Ses Çıkarma** — FFmpeg ile videodan 16kHz mono WAV
2. **Log-Mel Spektrogram** — 80-bant mel filtre bankası, 25ms pencere, 10ms kayma
3. **Dil Algılama** — İlk 30 saniyelik ses üzerinden (language parametresi verilmemişse)
4. **Transkripsiyon** — Transformer decoder ile segment segment çözümleme
5. **Sonuç** — Her segment için `start`, `end`, `text` bilgisi

## Hata Yönetimi

| Hata Durumu | Çözüm |
|-------------|-------|
| MP4 bulunamadı | Kullanıcıya mesaj + `sys.exit(1)` |
| FFmpeg bulunamadı | `FileNotFoundError` — Script başında PATH kontrolü |
| Uzun kesintisiz kelimeler | `sanitize_text()` ile 40 karakter sınırı |
| Unicode kontrol karakterleri | Regex ile temizleme |
| Font bulunamadı | Helvetica fallback |

## Performans Notları

- **GPU Kullanımı:** `torch.cuda.is_available()` true ise otomatik CUDA kullanılır
- **Bellek:** `large` model ~10GB VRAM gerektirir, GPU belleği yetersizse CPU'ya düşer
- **İşlem Süresi (yaklaşık):**

| Model   | 1 saatlik video (GPU) | 1 saatlik video (CPU) |
|---------|-----------------------|----------------------|
| `tiny`  | ~2 dk                 | ~10 dk               |
| `small` | ~8 dk                 | ~45 dk               |
| `medium`| ~20 dk                | ~2 saat              |
| `large` | ~40 dk                | ~4+ saat             |

## CLI Argümanları

```
usage: transcribe.py [-h] [--model {tiny,base,small,medium,large}]
                     [--language LANGUAGE] [--output OUTPUT] [video]

positional arguments:
  video                 MP4 dosya yolu (varsayılan: dizindeki ilk MP4)

optional arguments:
  --model               Whisper model boyutu (varsayılan: small)
  --language            ISO 639-1 dil kodu (varsayılan: otomatik)
  --output              Çıktı PDF dosya adı
```
