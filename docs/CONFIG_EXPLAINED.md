# Giải Thích Ý Nghĩa Các Cấu Hình

## 📋 Tổng Quan

Project có **3 loại config chính**:
1. **Config File (JSON/TOML)** - Cấu hình processing pipeline
2. **GPT Config (YAML)** - Cấu hình cho AI translators
3. **Environment Variables (.env)** - API keys và connection settings

---

## 🔧 Config File (JSON/TOML)

### 1. Translator Config

```json
{
  "translator": {
    "translator": "gemini",        // Dịch vụ sử dụng
    "target_lang": "VIN",          // Ngôn ngữ đích
    "no_text_lang_skip": false,    // Skip text đã là ngôn ngữ đích
    "skip_lang": null,             // Bỏ qua các ngôn ngữ cụ thể
    "gpt_config": null,            // Path to GPT config file
    "translator_chain": null,      // Chuỗi translator
    "selective_translation": null  // Chọn translator theo ngôn ngữ
  }
}
```

**Ý nghĩa:**
- **translator**: Chọn dịch vụ dịch (gemini, deepseek, nllb, etc.)
- **target_lang**: Ngôn ngữ muốn dịch SANG (VIN = Tiếng Việt)
- **no_text_lang_skip**: Nếu text đã là ngôn ngữ đích → có dịch lại không?
- **skip_lang**: Bỏ qua nếu detect ngôn ngữ nào đó (vd: `"JPN,ENG"`)
- **translator_chain**: Dùng nhiều translator liên tiếp (vd: `"google:JPN;sugoi:ENG"`)

---

### 2. Detector Config

```json
{
  "detector": {
    "detector": "default",         // Model phát hiện text
    "detection_size": 2048,        // Kích thước xử lý
    "text_threshold": 0.5,         // Ngưỡng phát hiện text
    "box_threshold": 0.75,         // Ngưỡng bounding box
    "unclip_ratio": 2.3,           // Tỷ lệ mở rộng bbox
    "det_rotate": false,           // Có xoay ảnh không
    "det_auto_rotate": false,      // Tự động xoay theo text
    "det_invert": false,           // Đảo màu
    "det_gamma_correct": false     // Sửa gamma
  }
}
```

**Ý nghĩa:**
- **detector**: Model nào phát hiện text
  - `default`: DBNet (tốt cho đa mục đích)
  - `ctd`: Tốt hơn cho manga
  - `paddle`: PaddleOCR detector
  
- **detection_size**: Kích thước ảnh resize trước khi detect (2048 = full size)
  - Nhỏ hơn → Nhanh nhưng bỏ sót text nhỏ
  - Lớn hơn → Chậm nhưng detect tốt hơn

- **text_threshold**: Độ "chắc chắn" phát hiện là text (0.0-1.0)
  - 0.3 = Phát hiện nhiều (có thể sai)
  - 0.7 = Chỉ phát hiện chắc chắn
  - 0.5 = Cân bằng

- **box_threshold**: Ngưỡng tạo bounding box
  - Cao hơn = Ít box hơn (chắc chắn hơn)
  - Thấp hơn = Nhiều box hơn (có thể có noise)

---

### 3. OCR Config

```json
{
  "ocr": {
    "ocr": "48px",                 // Model OCR
    "use_mocr_merge": false,       // Merge bbox trong Manga OCR
    "min_text_length": 0,         // Độ dài text tối thiểu
    "ignore_bubble": 0            // Bỏ qua text ngoài bubble
  }
}
```

**Ý nghĩa:**
- **ocr**: Model nhận dạng text
  - `48px`: Tốt cho Chinese, Japanese
  - `32px`: Basic, nhanh hơn
  - `mocr`: Manga OCR (rất tốt cho manga)
  
- **min_text_length**: Bỏ qua text ngắn hơn N ký tự
- **ignore_bubble**: Bỏ qua text không trong bubble (1-50, khuyên dùng 5-10)

---

### 4. Inpainter Config

```json
{
  "inpainter": {
    "inpainter": "lama_large",     // Model xóa text
    "inpainting_size": 2048,       // Kích thước xử lý
    "inpainting_precision": "bf16" // Độ chính xác
  }
}
```

**Ý nghĩa:**
- **inpainter**: Model nào xóa text cũ
  - `lama_large`: Chất lượng cao, chậm
  - `lama_mpe`: Nhanh hơn
  - `sd`: Stable Diffusion (rất chậm)
  
- **inpainting_size**: Kích thước ảnh khi xóa text
  - 2048 = Chuẩn
  - 1024 = Nhanh hơn, chất lượng kém
  - 4096 = Chất lượng cao hơn, OOM risk

- **inpainting_precision**: Độ chính xác tính toán
  - `bf16`: Cân bằng tốc độ/chất lượng
  - `fp32`: Chất lượng cao nhất, chậm
  - `fp16`: Nhanh nhưng kém chính xác

---

### 5. Render Config

```json
{
  "render": {
    "renderer": "default",         // Cách chèn text
    "alignment": "auto",           // Căn chỉnh
    "font_size_offset": 0,         // Điều chỉnh size font
    "font_size_minimum": -1,       // Font size tối thiểu
    "direction": "auto",           // Hướng text
    "uppercase": false,            // Viết hoa tất cả
    "lowercase": false,            // Viết thường tất cả
    "disable_font_border": false,  // Bỏ viền chữ
    "font_color": null,            // Màu chữ (hex)
    "line_spacing": null,          // Khoảng cách dòng
    "rtl": false                   // Phải sang trái
  }
}
```

**Ý nghĩa:**
- **renderer**: Cách render text
  - `default`: Cơ bản
  - `manga2eng`: Thông minh, tự fit vào bubble
  - `none`: Không render (chỉ inpaint)
  
- **font_size_offset**: Tăng/giảm font size
  - `+10`: Lớn hơn 10px
  - `-5`: Nhỏ hơn 5px

- **alignment**: Căn chỉnh text trong box
  - `auto`: Tự động phát hiện
  - `left`: Trái
  - `center`: Giữa
  - `right`: Phải

- **direction**: Hướng text
  - `auto`: Tự động
  - `horizontal`: Ngang
  - `vertical`: Dọc (cho Chinese/Japanese)

- **font_color**: Màu chữ
  - Format: `"FFFFFF:000000"` (foreground:background)
  - VD: `"FFFFFF"` = trắng, `":000000"` = nền đen

---

### 6. Upscale Config

```json
{
  "upscale": {
    "upscaler": "esrgan",          // Model phóng to
    "upscale_ratio": null,         // Tỷ lệ phóng to
    "revert_upscaling": false      // Có thu nhỏ lại sau khi dịch
  }
}
```

**Ý nghĩa:**
- **upscaler**: Model phóng to ảnh
  - `esrgan`: Chất lượng tốt
  - `waifu2x`: Chuyên cho anime/manga
  - `4xultrasharp`: Siêu nét
  
- **upscale_ratio**: Tỷ lệ phóng to (2 = x2, 4 = x4)
  - Giúp detect text nhỏ tốt hơn
  - VD: Text nhỏ → `upscale_ratio: 2` trước detect

- **revert_upscaling**: Có thu nhỏ lại về kích thước gốc không?

---

### 7. Colorizer Config

```json
{
  "colorizer": {
    "colorizer": "none",            // Có tô màu không
    "colorization_size": 576,      // Kích thước tô màu
    "denoise_sigma": 30            // Độ nhiễu
  }
}
```

**Ý nghĩa:**
- **colorizer**: Model tô màu
  - `none`: Không tô màu (giữ nguyên)
  - `mc2`: Tô màu cho manga đen trắng
  
- **colorization_size**: Kích thước khi tô màu
- **denoise_sigma**: Độ làm mịn (0-255, 30 = mặc định)

---

## 🤖 GPT Config (YAML) - Cho AI Translators

File: `examples/gpt_config-example.yaml`

### 1. Temperature & Top_p

```yaml
temperature: 0.5     # Độ "sáng tạo" (0.0-2.0)
top_p: 0.95          # Nucleus sampling (0.0-1.0)
```

**Ý nghĩa:**
- **temperature**: Kiểm soát độ ngẫu nhiên
  - `0.0-0.3`: Rất tập trung, nhất quán
  - `0.5-0.7`: Cân bằng
  - `0.8-2.0`: Sáng tạo, đa dạng hơn
  
- **top_p**: Chọn top N% tokens có xác suất cao nhất
  - `0.1`: Chỉ lấy top 10% tokens
  - `0.95`: Lấy 95% tokens

### 2. Chat System Template

```yaml
chat_system_template: >
  You are a professional translation engine.
  Translate text to {to_lang}.
  Maintain tone and meaning.
```

**Ý nghĩa:**
- Prompt hướng dẫn AI cách dịch
- `{to_lang}`: Placeholder cho ngôn ngữ đích
- Càng chi tiết → Translation càng tốt
- Nên include: Role, Method, Rules

### 3. Chat Samples

```yaml
chat_sample:
  English: 
    - <|1|>Original text 1
      <|2|>Original text 2
    - <|1|>Translated text 1
      <|2|>Translated text 2
```

**Ý nghĩa:**
- Ví dụ input-output mẫu để AI học style
- Format: `<|ID|>text` với cùng ID
- Giúp AI hiểu cách dịch phong cách này

### 4. JSON Mode

```yaml
json_mode: false              # Bật JSON mode
json_sample:                  # Ví dụ cho JSON mode
  Simplified Chinese:
    - TextList:
        - ID: 1
          text: "Original"
    - TextList:
        - ID: 1
          text: "Translated"
```

**Ý nghĩa:**
- **json_mode**: Dùng JSON format thay vì text
  - Chính xác hơn
  - Chỉ Gemini hỗ trợ tốt
  
- **json_sample**: Ví dụ JSON để AI học
  - ID phải khớp nhau
  - Format chuẩn để AI parse

### 5. Model-specific Config

```yaml
chatgpt:
  gpt-4o-mini:
    temperature: 0.4
  
gemini:
  temperature: 0.5
  top_p: 0.95
```

**Ý nghĩa:**
- Cấu hình riêng cho từng model
- Override settings mặc định
- Cho phép tối ưu cho từng AI

---

## 🌐 Environment Variables (.env)

### API Keys

```bash
# GEMINI
GEMINI_API_KEY='AIzaSy...'
GEMINI_MODEL='gemini-1.5-flash-002'

# DEEPSEEK
DEEPSEEK_API_KEY='sk-...'
DEEPSEEK_API_BASE='https://api.deepseek.com'
DEEPSEEK_MODEL='deepseek-chat'

# GROQ
GROQ_API_KEY='gsk_...'
GROQ_MODEL='mixtral-8x7b-32768'

# OPENAI
OPENAI_API_KEY='sk-...'
OPENAI_API_BASE='https://api.openai.com/v1'
OPENAI_MODEL='gpt-4o-mini'
```

**Ý nghĩa:**
- **API_KEY**: Key để authenticate với service
- **MODEL**: Model AI nào sử dụng
- **API_BASE**: URL endpoint (cho custom servers)

### Glossary & Dictionary

```bash
OPENAI_GLOSSARY_PATH='./dict/mit_glossary.txt'
SAKURA_DICT_PATH='./dict/sakura_dict.txt'
```

**Ý nghĩa:**
- File từ điển/thuật ngữ
- Giúp dịch nhất quán (tên riêng, thuật ngữ)
- Format: `original = translation`

---

## 📖 Complete Example: Chinese → Vietnamese

### Config File: `config.json`
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
    "inpainter": "lama_large",
    "inpainting_size": 2048
  },
  "render": {
    "renderer": "default",
    "font_size_offset": 5,
    "alignment": "auto",
    "direction": "auto"
  },
  "upscale": {
    "upscaler": "esrgan",
    "upscale_ratio": null
  },
  "kernel_size": 3,
  "mask_dilation_offset": 30
}
```

### Environment: `.env`
```bash
GEMINI_API_KEY='AIzaSy...your_key'
GEMINI_MODEL='gemini-1.5-flash-002'
```

### Command
```bash
python -m manga_translator local \
    -i ./chinese_images \
    --config-file config.json \
    --use-gpu
```

---

## 🎯 Giải Thích Các Tham Số Quan Trọng

### `kernel_size` (Line 973-976 in config.py)
```json
"kernel_size": 3
```
**Ý nghĩa**: Kích thước kernel để xóa phần còn thừa của text
- Số lẻ: 3, 5, 7
- Càng lớn → Xóa mạnh hơn (có thể làm mất detail)
- 3 = Mặc định (cân bằng)

### `mask_dilation_offset` (Line 978-981)
```json
"mask_dilation_offset": 30
```
**Ý nghĩa**: Mở rộng mask để chắc chắn che hết text cũ
- Tăng lên nếu text bị "lộ" sau inpainting
- 10-30 = Phạm vi an toàn
- Lớn hơn → Mask lớn hơn → Vùng xóa lớn hơn

### `no_text_lang_skip` (Line 222)
```json
"no_text_lang_skip": false
```
**Ý nghĩa**: Có dịch text đã là ngôn ngữ đích không?
- `false`: Bỏ qua text đã là VIN
- `true`: Dịch lại tất cả (kể cả đã là VIN)

### `skip_lang` (Line 224)
```json
"skip_lang": "JPN,ENG"
```
**Ý nghĩa**: Bỏ qua nếu detect là ngôn ngữ này
- VD: `"JPN,ENG"` → Không dịch nếu là Japanese/English
- Dùng khi muốn chỉ dịch từ Chinese

---

## 📊 Config Priority

```
Command line > Config file > Default values
```

Ví dụ:
```bash
# Command line override
--translator gemini --target-lang VIN --font-size-offset 10

# Config file
{
  "translator": { "translator": "nllb", "target_lang": "ENG" },
  "render": { "font_size_offset": 5 }
}

# Result: gemini, VIN, +10 (CLI wins)
```

---

## 🔗 Reference

- **Config schema**: `manga_translator/config.py`
- **GPT config**: `examples/gpt_config-example.yaml`
- **Examples**: `examples/config-example.json`
- **API keys**: `examples/Example.env`

---

*Complete config explanation guide*


