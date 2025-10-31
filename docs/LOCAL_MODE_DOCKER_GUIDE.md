# Hướng Dẫn Chạy Local Mode với Docker

## 🎯 Cách chỉnh Docker để chạy Local Mode

### 1. Tạo file `docker-compose-local.yml`

```yaml
services:
  manga_image_translator_local:
    image: zyddnys/manga-image-translator:main
    container_name: manga_image_translator_local_mode
    entrypoint: python
    command: manga_translator.py local --verbose --config-file /app/config/config-local.json --image "/app/input" --use-gpu
    
    volumes:
      - ./config:/app/config      # Config files
      - ./input:/app/input        # Input images
      - ./result:/app/result      # Output results
    
    ipc: host
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    
    environment:
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      GEMINI_MODEL: ${GEMINI_MODEL:-gemini-1.5-flash}
      NVIDIA_VISIBLE_DEVICES: all
      NVIDIA_DRIVER_CAPABILITIES: compute,utility
```

### 2. Tạo config file `config/config-local.json`

```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN",
    "gpt_config": "/app/config/gemini-vietnamese-fix.yaml"
  },
  "detector": {
    "detector": "default"
  }
}
```

### 3. Chuẩn bị thư mục

```bash
# Tạo thư mục input để chứa ảnh
mkdir input

# Copy ảnh vào thư mục input
cp your_image.jpg input/
```

### 4. Chạy Local Mode

```bash
# Chạy với docker-compose-local.yml
docker-compose -f docker-compose-local.yml up

# Hoặc chạy background
docker-compose -f docker-compose-local.yml up -d
```

### 5. Xem kết quả

```bash
# Kết quả sẽ được lưu trong thư mục result/
ls result/
```

---

## 🔄 So Sánh Local vs Server Mode

| Aspect | Local Mode | Server Mode |
|--------|------------|-------------|
| **Command** | `python manga_translator.py local` | `python server/main.py` |
| **Config** | `--config-file config.json` | Qua API request |
| **Input** | Folder chứa ảnh | Upload qua API |
| **Output** | File trong result/ | Response trả về |
| **Port** | Không cần | 5003 |
| **Use Case** | Batch processing | Interactive/API |

---

## ⚠️ Lưu Ý Quan Trọng

### Local Mode:
- ✅ Xử lý hàng loạt ảnh trong folder
- ✅ Không cần API, chạy trực tiếp
- ✅ Config qua `--config-file`
- ✅ Output tự động lưu vào result/

### Server Mode:
- ✅ Giao diện web tương tác
- ✅ API để tích hợp với app khác
- ✅ Config qua JSON trong request
- ✅ Streaming response

---

## 🎯 Khi Nào Dùng Mode Nào?

**Dùng Local Mode khi:**
- Cần dịch nhiều ảnh cùng lúc
- Chạy batch job tự động
- Không cần giao diện web

**Dùng Server Mode khi:**
- Cần giao diện web để test
- Tích hợp API vào app khác
- Dịch từng ảnh một cách tương tác




