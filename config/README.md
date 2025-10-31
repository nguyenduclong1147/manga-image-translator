# Hướng Dẫn Fix Vấn Đề "LITERAL TRANSLATION:" Xuất Hiện Khi Dịch

## 🔴 Vấn Đề

Khi sử dụng Gemini với config mặc định, kết quả dịch thường xuất hiện text "LITERAL TRANSLATION:" hay các label khác trong output.

**Nguyên nhân:** Prompt mặc định quá dài, Gemini hiểu nhầm là phải output cả quy trình 3 bước.

## ✅ Giải Pháp

Đã tạo config tùy chỉnh với prompt ngắn gọn, chỉ yêu cầu output bản dịch.

### Files Đã Tạo:

1. **`config/gemini-vietnamese-fix.yaml`** - Config fix cho Gemini
2. **`config/config-vietnamese.json`** - Config chính  
3. **`docker-compose-fix.yml`** - Docker compose file

### Cách Sử Dụng:

#### Bước 1: Cấu trúc thư mục
```bash
your-project/
├── config/
│   ├── gemini-vietnamese-fix.yaml  # ← Config fix
│   └── config-vietnamese.json      # ← Config chính
├── docker-compose-fix.yml          # ← Docker compose
└── result/                         # ← Output folder
```

#### Bước 2: Chạy Docker

```bash
# Nếu dùng file mới
docker-compose -f docker-compose-fix.yml up

# Hoặc nếu dùng file cũ
docker-compose up
```

#### Bước 3: Kiểm tra kết quả

File `/app/config/gemini-vietnamese-fix.yaml` sẽ override prompt mặc định.

## 📝 Nội Dung Config Fix

File `gemini-vietnamese-fix.yaml` chứa:

```yaml
gemini:
  chat_system_template: >
    Bạn là chuyên gia dịch manga...
    
    YÊU CẦU NGHIÊM NGẶT:
    1. CHỈ output bản dịch
    2. KHÔNG output "LITERAL TRANSLATION:"
    3. Format: <|1|>Bản dịch
```

**Điểm khác biệt:** Prompt ngắn gọn, chỉ yêu cầu output bản dịch, không có phần mô tả 3 bước process.

## 🎯 Kết Quả Mong Đợi

### ❌ TRƯỚC (Có lỗi):
```
<|1|>LITERAL TRANSLATION: Xin chào
<|2|>REFINEMENT: Bạn có khỏe không?
```

### ✅ SAU (Đã fix):
```
<|1|>Xin chào
<|2|>Bạn có khỏe không?
```

## ⚠️ LƯU Ý: Server Mode

**Server mode (main.py) KHÔNG hỗ trợ `--config-file`**

Config được truyền qua API request. Có 2 cách:

### Cách 1: Qua Web UI
Config sẽ tự động load file YAML tại `/app/config/gemini-vietnamese-fix.yaml` khi bạn gọi API.

### Cách 2: Gọi API trực tiếp
Thêm `gpt_config` vào JSON config:

```json
{
  "translator": {
    "translator": "gemini",
    "target_lang": "VIN",
    "gpt_config": "/app/config/gemini-vietnamese-fix.yaml"
  }
}
```

## 🔧 Tùy Chỉnh Thêm

Nếu muốn thay đổi instruction khác, sửa file `config/gemini-vietnamese-fix.yaml`:

```yaml
gemini:
  chat_system_template: >
    YOUR_CUSTOM_INSTRUCTION_HERE
```

Không cần rebuild image, chỉ restart container:

```bash
docker-compose restart
```

## 📚 Tham Khảo

- File gốc: `manga_translator/translators/config_gpt.py` - Chứa prompt mặc định
- Code parse: `manga_translator/translators/gemini.py:357` - Parse response
- Config load: `manga_translator/config.py:266` - Load YAML config

