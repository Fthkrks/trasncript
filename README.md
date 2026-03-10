# 🎙️ Video Transkript Aracı

MP4 video dosyalarını **OpenAI Whisper** ile otomatik olarak transkript eden ve çıktıyı **zaman damgalı PDF + TXT** olarak kaydeden bir Python aracıdır.

## ✨ Özellikler

- 🎬 MP4 video dosyalarını otomatik transkript etme
- ⏱️ Zaman damgalı çıktı (`[HH:MM:SS --> HH:MM:SS]  metin`)
- 📄 PDF ve TXT formatında kaydetme
- 🌐 Otomatik dil algılama (100+ dil desteği)
- 🤖 5 farklı model boyutu seçeneği (tiny → large)
- ⚡ GPU (CUDA) desteği — varsa otomatik kullanır

## 📋 Gereksinimler

- **Python 3.8+**
- **FFmpeg** (ses/video işleme için)

## 🚀 Adım Adım Sıfırdan Kurulum

Eğer bilgisayarınızda daha önce hiçbir şey kurulu değilse, aşağıdaki adımları sırasıyla takip edin:

### 1. Python Kurulumu
Sisteminizde Python yüklü olmalıdır. Yüklü değilse:
1. [python.org/downloads](https://www.python.org/downloads/) adresine gidin.
2. Python 3.8 veya daha yeni bir sürümü indirin.
3. Kurulum sırasında **"Add Python 3.x to PATH"** (Python'u PATH'e ekle) kutucuğunu **kesinlikle işaretleyin**, ardından "Install Now" diyerek kurun.

### 2. Gerekli Kütüphaneleri Yükleme
Proje klasörünün içine girin (Bu `README.md` dosyasının olduğu klasör). 
Klasör içindeyken boş bir yere `Shift` tuşuna basılı tutarak sağ tıklayın ve **"PowerShell penceresini burada aç"** (veya Windows 11'de sağ tıklayıp "Terminal'de aç") seçeneğini seçin. Alternatif olarak, klasördeyken adres çubuğuna `cmd` yazıp `Enter`'a basarak Komut İstemini (Terminal) açabilirsiniz. 

Açılan siyah ekranda (terminalde) şu komutu çalıştırın:

```bash
pip install -r requirements.txt
```
> **Not:** Bu komut `openai-whisper`, `fpdf2`, `torch` gibi tüm gerekli paketleri otomatik kuracaktır.

### 3. FFmpeg Kurulumu

FFmpeg, ses ve video işleme için gereklidir. Sisteminizde yüklü değilse:

**Windows (PowerShell ile otomatik indirme):**
Başlat menüsüne sağ tıklayıp **Windows PowerShell**'i açın ve şu komutları sırasıyla kopyalayıp yapıştırıp `Enter`'a basın:
```powershell
Invoke-WebRequest -Uri "https://www.gyan.dev/ffmpeg/builds/ffmpeg-release-essentials.zip" -OutFile "$env:TEMP\ffmpeg.zip"
Expand-Archive -Path "$env:TEMP\ffmpeg.zip" -DestinationPath "C:\ffmpeg" -Force
```
> **Not:** Script, `C:\ffmpeg` altındaki FFmpeg'i otomatik olarak görecek ve kullanacaktır. Ekstra bir ayar yapmanıza gerek yoktur.

### 4. GPU (CUDA) Kurulumu — Opsiyonel ama Şiddetle Önerilir

GPU (Ekran Kartı) kullanmak transkript işlemini **5-10 kat hızlandırır**. NVIDIA ekran kartınız varsa ve `requirements.txt` ile gelen standart `torch` sürümü GPU'nuzu görmezse aşağıdaki adımları izleyin:

**Adım 1 — NVIDIA sürücüsünü ve CUDA versiyonunu kontrol et:**
Terminal veya CMD'de şu komutu çalıştırın:
```bash
nvidia-smi
```
*(Sağ üst köşede `CUDA Version: 12.x` veya `11.x` şeklinde bir versiyon yazar. Bunu not alın.)*

Eğer sistemde daha önceden kurulu PyTorch varsa ve CUDA destekli değilse, aşağıdaki komutla kaldırıp tekrar kurmanın gerekebilir:
**Adım 2 — Mevcut PyTorch'u kaldır (Eğer varsa):**
```bash
pip uninstall torch torchvision torchaudio -y
```

**Adım 3 — Ekran kartınıza (CUDA) uygun PyTorch sürümünü kurun:**
Terminalde, sisteminizdeki CUDA sürümüne göre aşağıdaki komutlardan uygun olanı çalıştırın:

| Sistemdeki CUDA Versiyonu | Çalıştırılacak Komut |
|---------------|----------------|
| **12.1** veya **12.4** | `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121` |
| **11.8** | `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118` |

**Adım 4 — GPU'nun algılandığını doğrula:**
```bash
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'Bulunamadi')"
```
`True` ve Ekran Kartı modeliniz (Örn: RTX 3060) yazıyorsa kurulum başarıyla tamamlanmıştır.

> **Not:** `nvidia-smi` komutu çalışmıyor veya hata veriyorsa [nvidia.com/drivers](https://www.nvidia.com/drivers) adresinden ekran kartı sürücünüzü güncelleyin.

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
- Çalışma esnasında konsolda `🖥️ Kullanılan cihaz: CUDA` veya `CPU` yazar