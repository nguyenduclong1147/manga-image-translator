# API Endpoints Documentation

## 🌐 Base URL

```
http://localhost:8000      # Web UI
http://localhost:5003      # API Server
```

---

## 📡 Endpoints List

### 1. **POST /translate/json**
Trả về JSON với kết quả translation

**Request:**
```json
{
  "image": "base64_image_string",
  "config": {
    "translator": {
      "translator": "gemini",
      "target_lang": "VIN"
    }
  }
}
```

**Response:**
```json
{
  "textlines": [
    {
      "id": 1,
      "text": "原文",
      "translation": "Bản dịch",
      "bbox": [x, y, w, h]
    }
  ]
}
```

---

### 2. **POST /translate/image**
Trả về ảnh đã dịch

**Request:**
```json
{
  "image": "base64_string",
  "config": {...}
}
```

**Response:** Binary PNG image

---

### 3. **POST /translate/image/stream**
Trả về ảnh qua streaming (có progress)

**Request:** Tương tự trên

**Response Format:**
```
[status(1byte)][size(4bytes)][data(Nbytes)]
```

Status codes:
- `0`: Final result
- `1`: Progress update
- `2`: Error
- `3`: Queue position
- `4`: Translation started

---

### 4. **POST /translate/batch/json**
Dịch nhiều ảnh cùng lúc

**Request:**
```json
{
  "images": ["img1", "img2", "img3"],
  "config": {...},
  "batch_size": 4
}
```

**Response:**
```json
[
  { "textlines": [...] },
  { "textlines": [...] },
  { "textlines": [...] }
]
```

---

### 5. **POST /translate/with-form/image**
Upload ảnh qua form-data

**Request:**
- Content-Type: `multipart/form-data`
- `image`: File upload
- `config`: JSON string

**Response:** Binary PNG

---

### 6. **GET /queue-size**
Kiểm tra số task đang đợi

**Response:** 
```json
5  // Có 5 task đang chờ
```

---

### 7. **GET /results/list**
List tất cả results folders

**Response:**
```json
{
  "directories": ["folder1", "folder2"]
}
```

---

### 8. **DELETE /results/clear**
Xóa tất cả results

**Response:**
```json
{
  "message": "Deleted 10 result directories"
}
```

---

### 9. **GET /result/{folder_name}/final.png**
Lấy ảnh result theo folder name

---

### 10. **GET /docs**
Swagger UI documentation

---
---

## 💻 Code Examples

### Python

```python
import requests
import base64

# 1. Read image
with open("chinese.png", "rb") as f:
    img_data = base64.b64encode(f.read()).decode()

# 2. Prepare request
payload = {
    "image": img_data,
    "config": {
        "translator": {
            "translator": "gemini",
            "target_lang": "VIN"
        }
    }
}

# 3. Call API
response = requests.post(
    "http://localhost:5003/translate/json",
    json=payload
)

# 4. Get result
result = response.json()
print(result["textlines"])
```

---

### JavaScript/TypeScript

```typescript
async function translateImage(imageFile: File) {
  // Convert to base64
  const reader = new FileReader();
  const base64 = await new Promise<string>((resolve) => {
    reader.onload = () => resolve(reader.result as string);
    reader.readAsDataURL(imageFile);
  });

  // Call API
  const response = await fetch('http://localhost:5003/translate/json', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      image: base64,
      config: {
        translator: {
          translator: 'gemini',
          target_lang: 'VIN'
        }
      }
    })
  });

  const result = await response.json();
  return result.textlines;
}
```

---

### cURL

```bash
# Get translated image
curl -X POST http://localhost:5003/translate/image \
  -H "Content-Type: application/json" \
  -d '{
    "image": "base64_string_here",
    "config": {
      "translator": {"translator": "gemini", "target_lang": "VIN"}
    }
  }' \
  --output translated.png
```

---

### Multi-part Form

```bash
# Upload via form-data
curl -X POST http://localhost:5003/translate/with-form/image \
  -F "image=@chinese.png" \
  -F 'config={"translator":{"translator":"gemini","target_lang":"VIN"}}' \
  --output vietnamese.png
```

---

## 🔄 Streaming Example

### Python Streaming Client

```python
import requests
import io

# Call streaming endpoint
response = requests.post(
    "http://localhost:5003/translate/image/stream",
    json=payload,
    stream=True
)

# Parse streaming response
final_image = None
for chunk in response.iter_content(chunk_size=1024):
    # Format: [status][size][data]
    status = chunk[0]
    size = int.from_bytes(chunk[1:5], 'big')
    data = chunk[5:5+size]
    
    if status == 0:
        # Final result
        final_image = io.BytesIO(data)
        break
    elif status == 1:
        print(f"Progress: {data.decode()}")
    elif status == 3:
        print(f"Queue position: {int.from_bytes(data, 'big')}")
```

---

## 📊 API Architecture

```
┌────────────────────────────────────────┐
│           API Client                   │
│  (Python/JavaScript/cURL/Postman)      │
└────────────┬───────────────────────────┘
             │ HTTP Request
             ▼
┌────────────────────────────────────────┐
│        FastAPI Server                  │
│     server/main.py                     │
│     Port: 5003                         │
└────────────┬───────────────────────────┘
             │ Queue Task
             ▼
┌────────────────────────────────────────┐
│         Task Queue                      │
│     server/myqueue.py                   │
└────────────┬───────────────────────────┘
             │ Assign to Executor
             ▼
┌────────────────────────────────────────┐
│      Translation Worker                 │
│     manga_translator.py                 │
│     Full Pipeline Processing            │
└────────────┬───────────────────────────┘
             │ Result
             ▼
┌────────────────────────────────────────┐
│      Streaming Response                │
│     server/streaming.py                │
└────────────┬───────────────────────────┘
             │ Stream Chunks
             ▼
       Client receives result
```

---

## 🚀 Quick Start: Test API

### 1. Start Server
```bash
python server/main.py --use-gpu --start-instance
```

### 2. Check API Docs
```
http://localhost:5003/docs
```

### 3. Test with Python
```python
import requests
import base64

# Read image
with open("test.png", "rb") as f:
    img = base64.b64encode(f.read()).decode()

# Call API
response = requests.post(
    "http://localhost:5003/translate/image",
    json={
        "image": img,
        "config": {
            "translator": {"translator": "gemini", "target_lang": "VIN"}
        }
    }
)

# Save result
with open("translated.png", "wb") as f:
    f.write(response.content)
```

---

## 📝 Request/Response Schemas

### TranslateRequest
```python
{
  "image": str,        # Base64 hoặc URL hoặc bytes
  "config": Config     # Translation config object
}
```

### Config
```python
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN"
  },
  "detector": {...},
  "ocr": {...},
  "inpainter": {...},
  "render": {...},
  "upscale": {...}
}
```

### TranslationResponse
```python
{
  "textlines": [
    {
      "id": int,
      "text": str,           # Original text
      "translation": str,     # Translated text
      "bbox": [x, y, w, h],  # Bounding box
      "source_lang": str,
      "target_lang": str
    }
  ]
}
```

---

## 🎮 Interactive API Testing

### Swagger UI
```
http://localhost:5003/docs
```
- Try out all endpoints
- Auto-generated documentation
- Interactive request builder

### Manual HTML
```
http://localhost:5003
```
- Simple web interface
- Upload & translate
- Download result

---

*Complete API documentation*


