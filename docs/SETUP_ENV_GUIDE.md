# Hướng dẫn Setup .env File

## 📍 Vị trí file .env

File `.env` phải đặt ở **root folder** của project (cùng thư mục với `manga_translator/`, `server/`, etc.)

```
manga-image-translator/
├── .env                    ← TẠO FILE Ở ĐÂY!
├── manga_translator/
├── server/
├── examples/
└── README.md
```

---

## 🚀 Cách 1: Tạo file .env từ template

### Bước 1: Copy template
```bash
# Windows
copy examples\Example.env .env

# Mac/Linux
cp examples/Example.env .env
```

### Bước 2: Mở file .env và điền API key
```bash
# Edit file .env
notepad .env      # Windows
nano .env         # Linux/Mac
# hoặc mở bằng VS Code/Cursor
```

### Bước 3: Điền API key của bạn

**Ví dụ cho Gemini:**
```bash
GEMINI_API_KEY='AIzaSy...your_key_here'
GEMINI_MODEL='gemini-1.5-flash-002'
```

**Ví dụ cho DeepSeek:**
```bash
DEEPSEEK_API_KEY='sk-...your_key_here'
DEEPSEEK_API_BASE='https://api.deepseek.com'
DEEPSEEK_MODEL='deepseek-chat'
```

---

## 🚀 Cách 2: Tạo file .env thủ công

### 1. Tạo file `.env` ở root folder

### 2. Copy nội dung này vào:
```bash
# Chinese -> Vietnamese Translation
# Choose ONE translator below and fill its API key

# Option 1: GEMINI (Recommended)
GEMINI_API_KEY='your_api_key_here'
GEMINI_MODEL='gemini-1.5-flash-002'

# Option 2: DEEPSEEK (Good for Chinese)
# DEEPSEEK_API_KEY='your_api_key_here'
# DEEPSEEK_API_BASE='https://api.deepseek.com'
# DEEPSEEK_MODEL='deepseek-chat'

# Option 3: GROQ (Free tier)
# GROQ_API_KEY='your_api_key_here'
# GROQ_MODEL='mixtral-8x7b-32768'

# Option 4: OPENAI
# OPENAI_API_KEY='your_api_key_here'
# OPENAI_API_BASE='https://api.openai.com/v1'
# OPENAI_MODEL='gpt-4o-mini'
```

### 3. Lưu file và chạy:
```bash
python -m manga_translator local -i ./images --translator gemini --target-lang VIN
```

---

## 🎯 Quick Setup cho Chinese → Vietnamese

### Chọn 1 trong 4 options:

#### Option 1: GEMINI (Khuyên dùng)
```bash
# Get key: https://aistudio.google.com/app/apikey
GEMINI_API_KEY='AIzaSy...your_key'
GEMINI_MODEL='gemini-1.5-flash-002'
```

#### Option 2: DEEPSEEK (Tốt cho Chinese)
```bash
# Get key: https://platform.deepseek.com/api_keys
DEEPSEEK_API_KEY='sk-...your_key'
DEEPSEEK_API_BASE='https://api.deepseek.com'
DEEPSEEK_MODEL='deepseek-chat'
```

#### Option 3: GROQ (Free tier)
```bash
# Get key: https://console.groq.com/keys
GROQ_API_KEY='gsk_...your_key'
GROQ_MODEL='mixtral-8x7b-32768'
```

#### Option 4: Offline (Không cần API key)
```bash
# Không cần điền gì, chỉ dùng translator offline
# Ví dụ: --translator nllb hoặc --translator qwen2
```

---

## 📝 Example .env File

Tạo file `.env` ở root:

```bash
# Gemini Configuration
GEMINI_API_KEY='AIzaSyAbc123def456ghi789jkl012mno345pqr678stu901vwx234yz'
GEMINI_MODEL='gemini-1.5-flash-002'

# DeepSeek Configuration (uncomment to use)
# DEEPSEEK_API_KEY='sk-your-deepseek-key'
# DEEPSEEK_API_BASE='https://api.deepseek.com'
# DEEPSEEK_MODEL='deepseek-chat'

# Groq Configuration (uncomment to use)
# GROQ_API_KEY='gsk_your_groq_key'
# GROQ_MODEL='mixtral-8x7b-32768'

# OpenAI Configuration (uncomment to use)
# OPENAI_API_KEY='sk-your-openai-key'
# OPENAI_API_BASE='https://api.openai.com/v1'
# OPENAI_MODEL='gpt-4o-mini'

# Leave empty to use offline translators
# BAIDU_APP_ID=''
# BAIDU_SECRET_KEY=''
# YOUDAO_APP_KEY=''
# YOUDAO_SECRET_KEY=''
# DEEPL_AUTH_KEY=''
# CAIYUN_TOKEN=''
```

---

## ✅ Kiểm tra file .env

### 1. Xác nhận vị trí:
```bash
# Windows
dir .env

# Mac/Linux
ls -la .env
```

### 2. Xác nhận format đúng:
```bash
# ĐÚNG:
GEMINI_API_KEY='AIzaSy...'
GEMINI_MODEL='gemini-1.5-flash-002'

# SAI:
GEMINI_API_KEY = 'AIzaSy...'  # Không có spaces xung quanh =
GEMINI_API_KEY=AIzaSy...      # Thiếu quotes
```

### 3. Test đọc env:
```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('GEMINI_KEY:', os.getenv('GEMINI_API_KEY')[:10]+'...' if os.getenv('GEMINI_API_KEY') else 'Not found')"
```

---

## 🎮 Command Examples

### With Gemini:
```bash
python -m manga_translator local -i ./chinese_images \
    --translator gemini \
    --target-lang VIN \
    --use-gpu
```

### With DeepSeek:
```bash
python -m manga_translator local -i ./chinese_images \
    --translator deepseek \
    --target-lang VIN
```

### With Offline (No .env needed):
```bash
python -m manga_translator local -i ./chinese_images \
    --translator nllb \
    --target-lang VIN
```

---

## 🔒 Security Notes

### ✅ DO:
- Tạo file `.env` ở root folder
- Thêm `.env` vào `.gitignore` (đã tự động ignore)
- Giữ bí mật API keys
- Dùng quotes: `GEMINI_API_KEY='key'`

### ❌ DON'T:
- Commit file `.env` lên Git
- Chia sẻ API keys
- Để API key trong code
- Hardcode keys trong scripts

---

## 📚 Links để lấy API Keys

1. **Gemini**: https://aistudio.google.com/app/apikey
2. **DeepSeek**: https://platform.deepseek.com/api_keys
3. **Groq**: https://console.groq.com/keys (Free tier)
4. **OpenAI**: https://platform.openai.com/api-keys

---

## ⚡ Quick Start

### 1. Tạo file .env:
```bash
copy examples\Example.env .env
```

### 2. Mở và điền key:
```bash
# Chỉ cần điền key của translator bạn dùng
GEMINI_API_KEY='your_key_here'
```

### 3. Chạy:
```bash
python -m manga_translator local -i ./images --translator gemini --target-lang VIN
```

---

## 🆘 Troubleshooting

### "GEMINI_API_KEY not found"
**Giải pháp**: 
- Kiểm tra file `.env` ở đúng vị trí (root folder)
- Kiểm tra không có spaces: `KEY='value'` không phải `KEY = 'value'`
- Restart terminal/IDE

### "Invalid API Key"
**Giải pháp**:
- Kiểm tra key có quotes: `'your_key'`
- Kiểm tra không copy thêm spaces
- Lấy key mới từ web

### Environment variables not loading
**Giải pháp**:
```bash
# Verify load_dotenv() is called
python -c "from dotenv import load_dotenv; load_dotenv(); import os; print(os.getenv('GEMINI_API_KEY'))"
```

---

*Setup guide for .env configuration*


