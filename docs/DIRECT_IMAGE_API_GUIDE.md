# API Trả Về Ảnh PNG Trực Tiếp (Không Cần Decode)

## 🎯 Tìm Thấy: Có 2 API trả về ảnh PNG trực tiếp!

### ✅ API 1: `/translate/with-form/image` (Form Data)

**Cách dùng:**
```bash
curl -X POST "http://localhost:5003/translate/with-form/image" \
  -F "image=@test.jpg" \
  -F 'config={
    "translator": {
      "translator": "gemini",
      "target_lang": "VIN",
      "gpt_config": "/app/config/gemini-vietnamese-fix.yaml"
    }
  }' \
  --output result.png
```

**Đặc điểm:**
- ✅ **Trả về PNG trực tiếp** - `media_type="image/png"`
- ✅ **Không cần decode** - Lưu thẳng thành file .png
- ✅ **Form data** - Upload file qua form
- ✅ **Đơn giản** - Chỉ cần `--output result.png`

### ✅ API 2: `/translate/image` (JSON Data)

**Cách dùng:**
```bash
curl -X POST "http://localhost:5003/translate/image" \
  -H "Content-Type: application/json" \
  -d '{
    "image": "data:image/jpeg;base64,/9j/4AAQ...",
    "config": {
      "translator": {
        "translator": "gemini",
        "target_lang": "VIN",
        "gpt_config": "/app/config/gemini-vietnamese-fix.yaml"
      }
    }
  }' \
  --output result.png
```

**Đặc điểm:**
- ✅ **Trả về PNG trực tiếp** - `media_type="image/png"`
- ✅ **Không cần decode** - Lưu thẳng thành file .png
- ✅ **JSON data** - Base64 encoded image
- ✅ **Đơn giản** - Chỉ cần `--output result.png`

---

## ❌ API Streaming (Cần Decode)

### `/translate/with-form/image/stream`

**Vấn đề:**
- ❌ **Binary stream** - `media_type="application/octet-stream"`
- ❌ **Cần decode** - Format: `1byte status + 4byte size + data`
- ❌ **Phức tạp** - Phải parse stream format

**Format stream:**
```
[Status: 1 byte][Size: 4 bytes][Data: N bytes]
Status codes: 0=result, 1=progress, 2=error, 3=queue, 4=waiting
```

---

## 🎯 Kết Luận

**Dùng API KHÔNG có `/stream` để nhận ảnh PNG trực tiếp:**

| API | Input | Output | Decode? |
|-----|-------|--------|---------|
| `/translate/with-form/image` | Form | PNG | ❌ Không |
| `/translate/image` | JSON | PNG | ❌ Không |
| `/translate/with-form/image/stream` | Form | Binary Stream | ✅ Có |

### 💡 Khuyến nghị:

**Cho curl/script:** Dùng `/translate/with-form/image`
```bash
curl -X POST "http://localhost:5003/translate/with-form/image" \
  -F "image=@input.jpg" \
  -F 'config={"translator":{"translator":"gemini"}}' \
  --output result.png
```

**Cho web app:** Dùng `/translate/image` với base64
```javascript
const response = await fetch('/translate/image', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({image: base64Data, config: {...}})
});
const blob = await response.blob();
```

---

## 📚 Nguồn Tham Khảo

- **Code:** `server/main.py:77-84` - `/translate/image` endpoint
- **Code:** `server/main.py:112-121` - `/translate/with-form/image` endpoint  
- **Code:** `server/main.py:139-146` - `/translate/with-form/image/stream` endpoint
- **Format:** `server/streaming.py:11-18` - Stream format structure
- **Decode:** `examples/response.cpp` - C++ decoder example



