# Giải Thích: Tại Sao Có 2 Loại Config?

## 📊 So Sánh: .env vs config.json

### **File .env** (Environment Variables)
```bash
# Location: root folder/.env
GEMINI_API_KEY='AIzaSy...'
GEMINI_MODEL='gemini-1.5-flash-002'
```

**Mục đích:** API Keys & Connection Settings
- Chứa **sensitive data** (API keys)
- Không commit vào Git
- Để authentication với dịch vụ bên ngoài
- Load bằng `python-dotenv`

---

### **File config.json** (Config File)
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "default"
  }
}
```

**Mục đích:** Processing Parameters
- Chứa **processing settings**
- Không có sensitive data
- Có thể commit vào Git
- Dùng để control behavior của translation

---

## 🔑 Khi Nào Dùng File Nào?

### Scenario 1: API Keys (chỉ cần .env)
```
❌ SAI: Đặt API key vào config.json
{
  "translator": {
    "translator": "gemini",
    "api_key": "AIzaSy..."  ← ĐỪNG LÀM THẾ NÀY!
  }
}

✅ ĐÚNG: Đặt vào .env
GEMINI_API_KEY='AIzaSy...'
```

### Scenario 2: Processing Settings (chỉ cần config.json)
```
❌ SAI: Đặt settings vào .env
GEMINI_TRANSLATOR=gemini
GEMINI_TARGET_LANG=VIN  ← KHÔNG CÓ CÁCH NÀY!

✅ ĐÚNG: Đặt vào config.json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  }
}
```

---

## 📝 Hướng Dẫn Tạo Config.json

### Bước 1: Tạo file JSON

```bash
# Tạo file mới
touch my-config.json
# Hoặc copy template
copy examples\config-example.json my-config.json
```

### Bước 2: Chỉnh sửa với config cho Chinese → Vietnamese

```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN",
    "no_text_lang_skip": false
  },
  "detector": {
    "detector": "default",
    "detection_size": 2048
  },
  "ocr": {
    "ocr": "48px"
  },
  "render": {
    "renderer": "default",
    "font_size_offset": 0,
    "alignment": "auto",
    "direction": "auto"
  },
  "inpainter": {
    "inpainter": "lama_large",
    "inpainting_size": 2048
  },
  "upscale": {
    "upscaler": "esrgan"
  },
  "kernel_size": 3,
  "mask_dilation_offset": 30
}
```

### Bước 3: Sử dụng

```bash
python -m manga_translator local \
    -i ./images \
    --config-file my-config.json
```

---

## 🎯 Workflow Hoàn Chỉnh

### 1. Setup .env (API Keys)
```bash
# File: .env (ở root)
GEMINI_API_KEY='your_key'
GEMINI_MODEL='gemini-1.5-flash-002'
```

### 2. Tạo config.json (Processing)
```bash
# File: my-config.json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  }
}
```

### 3. Chạy Translation
```bash
python -m manga_translator local \
    -i ./chinese_images \
    --config-file my-config.json
    # .env sẽ tự động load
```

---

## 🔍 Chi Tiết Load Order

```
1. Load .env từ root folder
   ↓
   Set environment variables (GEMINI_API_KEY, etc)
   ↓
2. Parse config.json (nếu có)
   ↓
   Load processing settings
   ↓
3. Command line arguments
   ↓
   Override config.json settings
   ↓
4. Start translation
```

---

## 📄 Example Files

### File 1: `.env` (API credentials)
```bash
# Có ở root folder
GEMINI_API_KEY='AIzaSyA8bp0L7p21ExKdRua3G43oPLLvBrlAorg'
GEMINI_MODEL='gemini-1.5-flash-002'
DEEPSEEK_API_KEY=''
GROQ_API_KEY=''
```

### File 2: `config.json` (Settings)
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "default"
  }
}
```

---

## ⚙️ Cách Load Config

### Load từ .env
```python
# File: manga_translator/translators/keys.py
from dotenv import load_dotenv
import os

load_dotenv()  # Load .env từ root

GEMINI_API_KEY = os.getenv('GEMINI_API_KEY', '')
GEMINI_MODEL = os.getenv('GEMINI_MODEL', 'gemini-1.5-flash-002')
```

### Load từ config.json
```python
# File: manga_translator/args.py
config = Config.parse_raw(json.dumps(config_dict))
```

---

## 🎨 Quick Reference

| Aspect | .env | config.json |
|--------|------|-------------|
| **Contains** | API keys | Processing settings |
| **Security** | ⚠️ Private | ✅ Public |
| **Git** | ❌ .gitignore | ✅ Can commit |
| **When** | Authentication | Behavior control |
| **Example** | `KEY='secret'` | `{"key": "value"}` |

---

## 💡 Best Practices

### ✅ DO:

**1. Tách biệt rõ ràng:**
```bash
# .env - Sensitive data
GEMINI_API_KEY='secret'

# config.json - Settings
{
  "translator": {"translator": "gemini"}
}
```

**2. Sử dụng .env.example:**
```bash
cp .env.example .env
# Điền keys của bạn
```

**3. Commit config.json:**
```bash
git add config.json
# Không commit .env
```

### ❌ DON'T:

**1. Đừng hardcode keys:**
```json
{
  "translator": {
    "api_key": "AIzaSy..."  ❌
  }
}
```

**2. Đừng commit .env:**
```bash
git add .env  ❌
```

---

## 🚀 Complete Example

### Step 1: Setup .env
```bash
# File: .env
GEMINI_API_KEY='your_key_here'
GEMINI_MODEL='gemini-1.5-flash-002'
```

### Step 2: Create config.json
```bash
# File: chinese-to-vietnam.json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {
    "detector": "default"
  }
}
```

### Step 3: Run
```bash
python -m manga_translator local \
    -i ./chinese_images \
    --config-file chinese-to-vietnam.json
```

---

## 📚 Template Files

### Template 1: .env.example
```bash
# Copy to .env and fill
GEMINI_API_KEY=''
GEMINI_MODEL='gemini-1.5-flash-002'
```

### Template 2: config.example.json
```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  }
}
```

---

## 🔗 Files Reference

- **Config loading**: `manga_translator/args.py`
- **Keys loading**: `manga_translator/translators/keys.py`
- **Config schema**: `manga_translator/config.py`
- **Examples**: `examples/config-example.json`

---

*Explanation of config types and usage*


