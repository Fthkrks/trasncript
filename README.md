# 🎙️ Video Transkript Aracı

MP4 video dosyalarını **OpenAI Whisper** ile otomatik olarak transkript eden ve çıktıyı **zaman damgalı PDF + TXT** olarak kaydeden bir Python aracıdır.

## ✨ Özellikler

- 🎬 MP4 video dosyalarını otomatik transkript etme
- ⏱️ Zaman damgalı çıktı (`[HH:MM:SS --> HH:MM:SS]  metin`)
- 📄 PDF ve TXT formatında kaydetme
- 🌐 Otomatik dil algılama (100+ dil desteği)
- 🤖 5 farklı model boyutu seçeneği (tiny → large)

## 📋 Gereksinimler

- **Python 3.8+**
- **FFmpeg** (ses/video işleme için)

## 🚀 Kurulum

### 1. Python Paketleri

```bash
pip install openai-whisper fpdf2
```

### 2. FFmpeg

FFmpeg sisteminizde yüklü değilse:

**Windows (PowerShell ile indirme):**
```powershell
Invoke-WebRequest -Uri "https://www.gyan.dev/ffmpeg/builds/ffmpeg-release-essentials.zip" -OutFile "$env:TEMP\ffmpeg.zip"
Expand-Archive -Path "$env:TEMP\ffmpeg.zip" -DestinationPath "C:\ffmpeg" -Force
```

> **Not:** Script, `C:\ffmpeg` altındaki FFmpeg'i otomatik olarak PATH'e ekler.

## 📖 Kullanım

### Temel Kullanım

MP4 dosyanızı script ile aynı dizine koyun ve çalıştırın:

```bash
python transcribe.py
```

Dizindeki ilk MP4 dosyasını otomatik bulur ve transkript eder.

### Belirli Dosya

```bash
python transcribe.py toplanti.mp4
```

### Seçenekler

| Parametre    | Varsayılan | Açıklama                                      |
|-------------|-----------|-----------------------------------------------|
| `video`     | (otomatik) | MP4 dosya yolu                                |
| `--model`   | `small`   | Whisper modeli: `tiny`, `base`, `small`, `medium`, `large` |
| `--language` | (otomatik) | Dil kodu: `tr`, `en`, `de`, `fr` vb.         |
| `--output`  | (otomatik) | Çıktı PDF dosya adı                           |

### Örnekler

```bash
# Türkçe video, medium model ile
python transcribe.py video.mp4 --model medium --language tr

# İngilizce video, large model ile, özel çıktı adı
python transcribe.py meeting.mp4 --model large --language en --output meeting_notes.pdf

# Otomatik dil algılama ile
python transcribe.py conference.mp4 --model small
```

## 📁 Çıktılar

Script çalıştıktan sonra iki dosya oluşturulur:

- **`<video_adı>_transcript.pdf`** — Zaman damgalı transkript (PDF)
- **`<video_adı>_transcript.txt`** — Zaman damgalı transkript (TXT)

### Çıktı Formatı

```
[00:00:00 --> 00:00:05]  Merhaba, bugünkü toplantımıza hoş geldiniz.
[00:00:05 --> 00:00:12]  İlk olarak proje durumunu değerlendireceğiz.
[00:00:12 --> 00:00:18]  Ardından gelecek hafta için planlarımızı konuşacağız.
```

## 🤖 Whisper Model Karşılaştırması

| Model    | Boyut  | Hız       | Doğruluk  | VRAM   |
|---------|--------|-----------|----------|--------|
| `tiny`  | 39 MB  | ⚡ Çok hızlı | ★★☆☆☆   | ~1 GB  |
| `base`  | 74 MB  | ⚡ Hızlı    | ★★★☆☆   | ~1 GB  |
| `small` | 461 MB | 🔄 Orta     | ★★★★☆   | ~2 GB  |
| `medium`| 1.5 GB | 🐢 Yavaş    | ★★★★★   | ~5 GB  |
| `large` | 2.9 GB | 🐌 Çok yavaş | ★★★★★  | ~10 GB |

> **İpucu:** GPU yoksa `tiny` veya `base` modeli önerilir. GPU varsa en iyi sonuç için `medium` veya `large` kullanın.

## ⚠️ Bilinen Notlar

- İlk çalıştırmada Whisper modeli indirilecektir (internet bağlantısı gerekli)
- Uzun videolarda işlem süresi artabilir
- GPU (CUDA) varsa otomatik olarak kullanılır, yoksa CPU ile çalışır
