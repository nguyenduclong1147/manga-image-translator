# GPT Config Flow - Toàn Bộ Luồng Hoạt Động

## 🎯 Trả Lời Câu Hỏi: Config có chỉ hoạt động với API không?

**KHÔNG!** Config hoạt động ở **CẢ LOCAL mode và SERVER mode**. 

Chỉ khác nhau ở cách truyền config:

### 📊 So Sánh

| Mode | Cách Truyền Config | File Liên Quan |
|------|-------------------|----------------|
| **Local Mode** | Từ `--config-file` hoặc default | `config.json` → `manga_translator/mode/local.py` |
| **Server Mode** | Từ API request (JSON) | Request body → `server/main.py` → `server/request_extraction.py` |

---

## 🔄 Luồng Hoạt Động CHUNG (Cả 2 mode)

### Step 1: Đường dẫn YAML được set

**Local Mode:**
```python
# manga_translator/mode/local.py
config = Config.parse_file(args.config_file)
# → TranslatorConfig.gpt_config = "/path/to/config.yaml"
```

**Server Mode:**
```python
# server/request_extraction.py
conf = Config.parse_raw(config_json)  # Parse từ API
# → TranslatorConfig.gpt_config = "/app/config/gemini-vietnamese-fix.yaml"
```

### Step 2: Load YAML qua `chatgpt_config` property

```python
# manga_translator/config.py:263-267
@property
def chatgpt_config(self):
    if self.gpt_config is not None and self._gpt_config is None:
        #todo: load from already loaded file
        self._gpt_config = OmegaConf.load(self.gpt_config)  # ← LOAD YAML
    return self._gpt_config
```

### Step 3: Translator parse_args nhận config

```python
# manga_translator/translators/common_gpt.py:57-58
def parse_args(self, args: CommonTranslator):
    self.config = args.chatgpt_config  # ← SET CONFIG
```

### Step 4: ConfigGPT sử dụng config khi dịch

```python
# manga_translator/translators/config_gpt.py:178-194
def _config_get(self, key: str, default=None):
    if not self.config:  # ← KIỂM TRA config có tồn tại không
        return default
    
    # Traverse from deepest to root level
    parts = self._CONFIG_KEY.split('.') if self._CONFIG_KEY else []
    for i in range(len(parts), -1, -1):
        lookup_key = f"{prefix}.{key}" if prefix else key
        value = OmegaConf.select(self.config, lookup_key)  # ← TÌM GIÁ TRỊ
        if value is not None:
            break
    return value if value is not None else default

# Dùng config khi dịch
@property
def chat_system_template(self) -> str:
    return self._config_get('chat_system_template', self._CHAT_SYSTEM_TEMPLATE)
    # ↑ Ưu tiên config, nếu không có thì dùng default
```

### Step 5: Gemini sử dụng prompt từ config

```python
# manga_translator/translators/gemini.py (trong _request_translation)
if self.useCache:
    # ... cached mode
else:
    config_kwargs['system_instruction'] = self.chat_system_template.format(to_lang=to_lang)
    # ↑ Dùng prompt từ config (hoặc default)
    
response = await self.client.aio.models.generate_content(...)
```

---

## 📝 Ví Dụ Cụ Thể

### Local Mode

```bash
# Chạy local
python manga_translator.py -i input.jpg --translator gemini --target-lang VIN --config-file config/config-vietnamese.json

# Flow:
# 1. Load config.json → TranslatorConfig.gpt_config = "/app/config/gemini-vietnamese-fix.yaml"
# 2. ChatgptConfig property load YAML → OmegaConf.load("/app/config/gemini-vietnamese-fix.yaml")
# 3. Translator.parse_args(args) → self.config = loaded_yaml
# 4. Khi dịch: _config_get('chat_system_template') → Trả về prompt từ YAML
```

### Server Mode (API)

```bash
# Gọi API
curl -X POST "http://localhost:5003/translate/with-form/image/stream" \
  -F "image=@input.jpg" \
  -F 'config={
    "translator": {
      "translator": "gemini",
      "target_lang": "VIN",
      "gpt_config": "/app/config/gemini-vietnamese-fix.yaml"
    }
  }'

# Flow:
# 1. API nhận request → Parse JSON → TranslatorConfig.gpt_config = "/app/config/gemini-vietnamese-fix.yaml"
# 2. ChatgptConfig property load YAML → OmegaConf.load("/app/config/gemini-vietnamese-fix.yaml")
# 3. Translator.parse_args(args) → self.config = loaded_yaml
# 4. Khi dịch: _config_get('chat_system_template') → Trả về prompt từ YAML
```

---

## 🎯 Kết Luận

**Config hoạt động Ở CẢ 2 MODE!**

Điều quan trọng:
- ✅ YAML file phải tồn tại và được mount (trong Docker)
- ✅ Đường dẫn trong config phải đúng
- ✅ `parse_args()` phải được gọi để set `self.config`

**Khác biệt duy nhất:** Cách truyền `gpt_config` vào `TranslatorConfig`:
- Local: Qua `--config-file` 
- Server: Qua JSON trong API request

---

## 📚 Files Tham Khảo

- Load config: `manga_translator/config.py:266` - `OmegaConf.load()`
- Set config: `manga_translator/translators/common_gpt.py:58` - `self.config = args.chatgpt_config`
- Dùng config: `manga_translator/translators/config_gpt.py:178-194` - `_config_get()`
- Gemini sử dụng: `manga_translator/translators/gemini.py` - `system_instruction = self.chat_system_template`




