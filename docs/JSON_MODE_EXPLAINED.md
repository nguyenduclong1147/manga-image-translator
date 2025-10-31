# JSON Mode: True vs False - Giải Thích Chi Tiết

## 🎯 `json_mode: false` là gì?

**`json_mode: false`** = **Standard Mode** (Chế độ chuẩn)
**`json_mode: true`** = **JSON Mode** (Chế độ JSON)

---

## 📊 So Sánh 2 Chế Độ:

### 🔄 Standard Mode (`json_mode: false`)

**Cách hoạt động:**
```python
# manga_translator/translators/gemini.py:262-264
def _init_standard_mode(self):
    """Use default method implementations"""
    self._assemble_prompts = super()._assemble_prompts
```

**Input cho model:**
```
System: "Bạn là chuyên gia dịch tiếng Trung sang Tiếng Việt..."

User: "<|1|>你好世界<br><|2|>这是测试"
```

**Output từ model:**
```
"<|1|>Xin chào thế giới<br><|2|>Đây là thử nghiệm"
```

**Đặc điểm:**
- ✅ **Đơn giản** - Text thuần túy
- ✅ **Dễ hiểu** - Format `<|ID|>Text`
- ✅ **Ổn định** - Ít lỗi parsing
- ❌ **Không có cấu trúc** - Chỉ là text

---

### 🔄 JSON Mode (`json_mode: true`)

**Cách hoạt động:**
```python
# manga_translator/translators/gemini.py:253-260
def _init_json_mode(self):
    """Activate JSON-specific behavior"""
    self._json_funcs = _GeminiTranslator_json(self)
    self._createContext = self._json_funcs._createContext
    self._request_translation = self._json_funcs._request_translation
    self._assemble_prompts = self._json_funcs._assemble_prompts
    self._parse_response = self._json_funcs._parse_response
```

**Input cho model:**
```json
System: "Bạn là chuyên gia dịch tiếng Trung sang Tiếng Việt..."

User: {
  "TextList": [
    {"ID": "1", "text": "你好世界"},
    {"ID": "2", "text": "这是测试"}
  ]
}
```

**Output từ model:**
```json
{
  "TextList": [
    {"ID": "1", "text": "Xin chào thế giới"},
    {"ID": "2", "text": "Đây là thử nghiệm"}
  ]
}
```

**Đặc điểm:**
- ✅ **Có cấu trúc** - JSON format
- ✅ **Metadata** - Có thể thêm thông tin khác
- ✅ **Validation** - Schema checking
- ❌ **Phức tạp** - Cần parse JSON
- ❌ **Dễ lỗi** - JSON parsing có thể fail

---

## 🔧 Cách Cấu Hình:

### Trong `gpt_config.yaml`:

```yaml
gemini:
  json_mode: false  # ← Standard mode
  temperature: 0.3
  chat_system_template: >
    Bạn là chuyên gia dịch...
```

### Hoặc:

```yaml
gemini:
  json_mode: true   # ← JSON mode
  temperature: 0.3
  chat_system_template: >
    Bạn là chuyên gia dịch...
  json_sample:      # ← Cần có sample cho JSON mode
    VIN:
      - TextList:
          - ID: "1"
            text: "你好"
      - TextList:
          - ID: "1" 
            text: "Xin chào"
```

---

## 🎯 Khi Nào Dùng Mode Nào?

### Dùng **Standard Mode** (`json_mode: false`) khi:
- ✅ **Đơn giản** - Chỉ cần dịch text
- ✅ **Ổn định** - Không muốn lỗi parsing
- ✅ **Manga thông thường** - Text đơn giản
- ✅ **Không cần metadata** - Chỉ cần bản dịch

### Dùng **JSON Mode** (`json_mode: true`) khi:
- ✅ **Cần cấu trúc** - Metadata phức tạp
- ✅ **Batch processing** - Xử lý nhiều text cùng lúc
- ✅ **API integration** - Cần format chuẩn
- ✅ **Advanced features** - Cần thông tin bổ sung

---

## 📝 Ví Dụ Thực Tế:

### Standard Mode:
```
Input:  "你好世界"
Output: "Xin chào thế giới"
```

### JSON Mode:
```json
Input:  {"TextList": [{"ID": "1", "text": "你好世界"}]}
Output: {"TextList": [{"ID": "1", "text": "Xin chào thế giới"}]}
```

---

## ⚠️ Lưu Ý Quan Trọng:

1. **JSON Mode cần `json_sample`** - Không có sẽ fallback về `chat_sample`
2. **Standard Mode đơn giản hơn** - Ít lỗi, dễ debug
3. **JSON Mode mạnh mẽ hơn** - Nhưng phức tạp hơn
4. **Mặc định là `false`** - Standard mode

---

## 📚 Nguồn Tham Khảo:

- **Config:** `manga_translator/translators/config_gpt.py:291-293`
- **Standard:** `manga_translator/translators/gemini.py:262-264`
- **JSON:** `manga_translator/translators/gemini.py:253-260`
- **JSON Logic:** `manga_translator/translators/common_gpt.py:280-405`



