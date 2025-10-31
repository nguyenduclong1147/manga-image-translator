# Hướng dẫn: Chinese → Vietnamese Translation

## 📋 Mã ngôn ngữ

```
CHS = Chinese (Simplified) = Tiếng Trung Giản Thể
VIN = Vietnamese = Tiếng Việt
```

## 🎯 Các Translator Hỗ Trợ CHS → VIN

### ✅ Online (Cần Internet + API Key)

| Translator | Chất lượng | Tốc độ | API Key | Ghi chú |
|------------|-----------|--------|---------|---------|
| **gemini** | ⭐⭐⭐⭐⭐ | Nhanh | Google API | Khuyên dùng |
| **deepseek** | ⭐⭐⭐⭐⭐ | Nhanh | DeepSeek API | Tốt cho Chinese |
| **groq** | ⭐⭐⭐⭐ | Rất nhanh | Groq API | Miễn phí hạn mức |
| **openai** | ⭐⭐⭐⭐⭐ | Nhanh | OpenAI API | Chat-GPT |
| **deepl** | ⭐⭐⭐⭐⭐ | Nhanh | DeepL API | Chất lượng tốt |
| **baidu** | ⭐⭐⭐⭐ | Nhanh | Baidu API | Chinese native |
| **youdao** | ⭐⭐⭐⭐ | Nhanh | Youdao API | Chinese native |
| **caiyun** | ⭐⭐⭐⭐ | Nhanh | Caiyun API | Chinese native |

### ✅ Offline (Không cần Internet)

| Translator | Chất lượng | Tốc độ | Setup | Ghi chú |
|------------|-----------|--------|-------|---------|
| **m2m100** | ⭐⭐⭐ | Chậm | Auto download | Hỗ trợ đa ngôn ngữ |
| **nllb** | ⭐⭐⭐⭐ | Trung bình | Auto download | Facebook NLLB |
| **qwen2** | ⭐⭐⭐⭐⭐ | Trung bình | Auto download | Alibaba Qwen |

---

## 📝 Cách 1: Dùng Online Translator (Khuyên dùng)

### Option A: Gemini (Google)

```bash
# 1. Tạo file .env
echo "GEMINI_API_KEY=your_api_key_here" > .env
echo "GEMINI_MODEL=gemini-1.5-flash-002" >> .env

# 2. Chạy translation
python -m manga_translator local \
    -i /path/to/chinese/images \
    --config-file examples/config-chinese-to-vietnam.json \
    --translator gemini \
    --target-lang VIN
```

**Config file** (tạo file mới hoặc chỉnh sửa):
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  }
}
```

### Option B: DeepSeek (Tốt cho Chinese)

```bash
# 1. Tạo file .env
echo "DEEPSEEK_API_KEY=your_api_key_here" > .env

# 2. Chạy translation
python -m manga_translator local \
    -i /path/to/chinese/images \
    -o /path/to/output \
    --translator deepseek \
    --target-lang VIN
```

### Option C: OpenAI ChatGPT

```bash
# 1. Tạo file .env
echo "OPENAI_API_KEY=your_api_key_here" > .env

# 2. Chạy translation
python -m manga_translator local \
    -i /path/to/chinese/images \
    --translator chatgpt \
    --target-lang VIN
```

---

## 📝 Cách 2: Dùng Offline Translator (Không cần API)

### Option A: NLLB (Facebook - Hỗ trợ tốt VIN)

```bash
python -m manga_translator local \
    -i /path/to/chinese/images \
    --translator nllb \
    --target-lang VIN
```

### Option B: M2M100 (Multilingual)

```bash
python -m manga_translator local \
    -i /path/to/chinese/images \
    --translator m2m100 \
    --target-lang VIN
```

### Option C: Qwen2 (Alibaba - Tốt cho Chinese)

```bash
python -m manga_translator local \
    -i /path/to/chinese/images \
    --translator qwen2 \
    --target-lang VIN
```

---

## 🎮 Command Line Full Example

### Ví dụ 1: Gemini + GPU

```bash
python -m manga_translator local \
    -i ./chinese_manga \
    -o ./vietnamese_manga \
    --translator gemini \
    --target-lang VIN \
    --use-gpu \
    -v
```

### Ví dụ 2: NLLB Offline

```bash
python -m manga_translator local \
    -i ./chinese_manga \
    --translator nllb \
    --target-lang VIN \
    --use-gpu
```

### Ví dụ 3: Config File

Tạo file `my-config.json`:
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "default",
    "detection_size": 2048
  },
  "ocr": {
    "ocr": "48px"
  },
  "inpainter": {
    "inpainter": "lama_large"
  }
}
```

Chạy:
```bash
python -m manga_translator local \
    -i ./chinese_images \
    --config-file my-config.json
```

---

## ⚙️ Config Options Chi Tiết

### 1. Detector (Phát hiện text)
```json
"detector": {
  "detector": "default",  // Hoặc "ctd" cho tốt hơn
  "detection_size": 2048,
  "box_threshold": 0.75
}
```

### 2. OCR (Nhận dạng text)
```json
"ocr": {
  "ocr": "48px"  // Tốt cho Chinese
}
```

### 3. Translator
```json
"translator": {
  "translator": "gemini",  // Chọn translator
  "target_lang": "VIN"
}
```

### 4. Inpainting (Xóa text cũ)
```json
"inpainter": {
  "inpainter": "lama_large",
  "inpainting_size": 2048
}
```

### 5. Rendering (Chèn text mới)
```json
"render": {
  "font_size_offset": 0,
  "alignment": "auto",
  "direction": "auto"  // auto phát hiện ngang/dọc
}
```

---

## 🎨 Web Server Mode

### Start Web Server:

```bash
cd server
python main.py --use-gpu --start-instance
```

### Upload qua Web UI:
1. Mở http://localhost:8000
2. Chọn translator: `gemini` hoặc `deepseek`
3. Chọn Target Language: `VIN` (Tiếng Việt)
4. Upload ảnh và chờ kết quả

---

## 🚀 Docker Mode

```bash
# Với Gemini API
docker run --rm \
  -v $(pwd)/chinese_images:/app/input \
  -v $(pwd)/vietnamese_images:/app/output \
  -e GEMINI_API_KEY="your_key" \
  -e GEMINI_MODEL="gemini-1.5-flash-002" \
  zyddnys/manga-image-translator:main \
  local -i /app/input -o /app/output \
  --translator gemini --target-lang VIN
```

---

## ⚡ Tips để Tăng Chất lượng

### 1. Cho Chinese Text:
```json
{
  "ocr": {
    "ocr": "48px"  // Tốt cho Chinese
  },
  "detector": {
    "detector": "default",
    "detection_size": 2048
  }
}
```

### 2. Tăng độ phân giải nếu text nhỏ:
```json
{
  "upscale": {
    "upscale_ratio": 2  // Phóng to 2x trước khi detect
  }
}
```

### 3. Tăng font size cho text tiếng Việt:
```json
{
  "render": {
    "font_size_offset": 10,  // Tăng font size
    "alignment": "auto",
    "direction": "auto"
  }
}
```

### 4. Dùng glossary (từ điển):
Tạo file `dict.txt`:
```
他 = anh ấy
她 = cô ấy
你 = bạn
```

```bash
python -m manga_translator local \
    -i ./images \
    --translator gemini \
    --target-lang VIN \
    --post-dict dict.txt
```

---

## 🎓 Recommended Setup

### Best Quality (Online - Cần API):
```bash
# .env
GEMINI_API_KEY=sk-...
GEMINI_MODEL=gemini-1.5-flash-002

# Command
python -m manga_translator local \
    -i ./chinese \
    --translator gemini \
    --target-lang VIN \
    --use-gpu
```

### Best Speed (Offline - Không cần API):
```bash
python -m manga_translator local \
    -i ./chinese \
    --translator nllb \
    --target-lang VIN \
    --use-gpu
```

### Budget Option (Offline miễn phí):
```bash
python -m manga_translator local \
    -i ./chinese \
    --translator qwen2 \
    --target-lang VIN
```

---

## ✅ Quick Start Checklist

- [ ] Cài đặt dependencies: `pip install -r requirements.txt`
- [ ] Chọn translator (gemini/deepseek/nllb/qwen2)
- [ ] Lấy API key (nếu dùng online)
- [ ] Setup .env file với API key
- [ ] Chuẩn bị ảnh Chinese trong folder
- [ ] Chạy lệnh với `--target-lang VIN`
- [ ] Kiểm tra kết quả trong `output` folder

---

## 📚 Tham khảo

- **Language codes**: `manga_translator/translators/common.py:14-41`
- **Config schema**: `examples/config-example.json`
- **Translators list**: README.md
- **API keys setup**: `examples/Example.env`

---

*Generated guide for Chinese to Vietnamese translation*


