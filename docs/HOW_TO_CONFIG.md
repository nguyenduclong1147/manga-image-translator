# Hướng Dẫn Cấu Hình Chi Tiết

## 🤔 Tại Sao Có 2 File Config?

### File .env - API Credentials
```bash
# Chứa: API Keys, Secrets
GEMINI_API_KEY='sk-...'
GEMINI_MODEL='gemini-1.5-flash-002'
```

**Tại sao?**
- Để authentication với dịch vụ bên ngoài
- Không thể hardcode vào code (security)
- Không commit lên Git

### File config.json - Processing Parameters
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  }
}
```

**Tại sao?**
- Để control behavior của translation
- Có thể share, commit vào Git
- Không chứa sensitive data

---

## 📝 Cách Tạo Config.json

### Bước 1: Copy Template

```bash
# Windows
copy examples\config-example.json my-config.json

# Mac/Linux
cp examples/config-example.json my-config.json
```

### Bước 2: Chỉnh Sửa

Mở file `my-config.json` và sửa:

```json
{
  "translator": {
    "translator": "gemini",     ← Chọn translator
    "target_lang": "VIN"        ← Ngôn ngữ đích
  },
  "detector": {
    "detector": "default",
    "detection_size": 2048
  },
  "ocr": {
    "ocr": "48px"               ← Tốt cho Chinese
  },
  "inpainter": {
    "inpainter": "lama_large",
    "inpainting_size": 2048
  },
  "render": {
    "renderer": "default",
    "font_size_offset": 5       ← Tăng font 5px
  }
}
```

### Bước 3: Sử dụng

```bash
python -m manga_translator local \
    -i ./images \
    --config-file my-config.json
```

---

## 🎯 Complete Workflow Example

### 1. File .env (ở root folder)
```bash
# .env
GEMINI_API_KEY='AIzaSy...your_real_key'
GEMINI_MODEL='gemini-1.5-flash-002'
DEEPSEEK_API_KEY=''
GROQ_API_KEY=''
```

### 2. File config.json (ở bất kỳ đâu)
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "ctd",
    "detection_size": 2048
  },
  "ocr": {
    "ocr": "48px"
  },
  "render": {
    "renderer": "default",
    "font_size_offset": 10
  }
}
```

### 3. Chạy lệnh
```bash
python -m manga_translator local \
    -i ./chinese_images \
    --config-file config.json \
    --use-gpu \
    -v
```

---

## 🔄 Flow Load Config

```
┌─────────────────────────────────────────┐
│  Start Application                      │
└──────────────┬──────────────────────────┘
               │
               ▼
    ┌──────────────────────┐
    │  Load .env           │
    │  from root folder    │
    └──────────┬───────────┘
               │
               ▼ Environment variables set
    ┌──────────────────────┐
    │  GEMINI_API_KEY      │
    │  GEMINI_MODEL        │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │  Parse config.json   │
    │  (nếu có --config)   │
    └──────────┬───────────┘
               │
               ▼ Processing settings
    ┌──────────────────────┐
    │  translator, detector│
    │  ocr, render...     │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │  Apply CLI args      │
    │  (override config)   │
    └──────────┬───────────┘
               │
               ▼ Final config
    ┌──────────────────────┐
    │  Start Translation   │
    └──────────────────────┘
```

---

## ✅ Ví Dụ Thực Tế: Chinese → Vietnamese

### Setup hoàn chỉnh:

**File 1: `.env`**
```bash
GEMINI_API_KEY='AIzaSyA8bp0L7p21ExKdRua3G43oPLLvBrlAorg'
GEMINI_MODEL='gemini-1.5-flash-002'
```

**File 2: `config.json`**
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "default"
  },
  "ocr": {
    "ocr": "48px"
  },
  "inpainter": {
    "inpainter": "lama_large"
  },
  "render": {
    "renderer": "default",
    "font_size_offset": 5
  }
}
```

**Command:**
```bash
python -m manga_translator local \
    -i C:\path\to\chinese_images \
    --config-file config.json
```

---

## 📊 So Sánh Ngắn Gọn

| Đặc điểm | .env | config.json |
|----------|------|-------------|
| **Nội dung** | API keys | Settings |
| **Git** | ❌ Ignore | ✅ Commit |
| **Security** | ⚠️ Sensitive | ✅ Safe |
| **Dùng cho** | Authentication | Behavior |
| **Format** | KEY='value' | {"key": "value"} |

---

*Complete guide for configuration*


