# Flow Dịch Thuật: Model Nhận Gì và Trả Về Gì?

## 🎯 Trả Lời: Model NHẬN TEXT, KHÔNG nhận ảnh trực tiếp

### 📊 Flow Hoạt Động:

```
Ảnh Input → OCR → Text → Model → Translation → Render → Ảnh Output
```

---

## 🔄 Chi Tiết Từng Bước:

### 1. **Detection** (Phát hiện vùng text)
```python
# manga_translator/manga_translator.py:470
ctx.textlines, ctx.mask_raw, ctx.mask = await self._run_detection(config, ctx)
```
- **Input:** Ảnh gốc
- **Output:** Vùng text (bounding boxes)

### 2. **OCR** (Nhận dạng text)
```python
# manga_translator/manga_translator.py:497
ctx.textlines = await self._run_ocr(config, ctx)
```
- **Input:** Ảnh + vùng text
- **Output:** Text đã nhận dạng

### 3. **Text Merge** (Gộp text)
```python
# manga_translator/manga_translator.py:513
ctx.text_regions = await self._run_textline_merge(config, ctx)
```
- **Input:** Text từ OCR
- **Output:** Text regions đã gộp

### 4. **Translation** (Dịch thuật)
```python
# manga_translator/manga_translator.py:1094-1096
texts = [region.text for region in ctx.text_regions]
translated_sentences = await self._dispatch_with_context(config, texts, ctx)
```
- **Input:** **TEXT** (không phải ảnh)
- **Output:** Text đã dịch

### 5. **Render** (Vẽ text lên ảnh)
```python
# manga_translator/manga_translator.py:562+
ctx.result = await self._run_render(config, ctx)
```
- **Input:** Ảnh gốc + text đã dịch
- **Output:** Ảnh hoàn chỉnh

---

## 🤖 Model Nhận Gì?

### Gemini Translator:
```python
# manga_translator/translators/gemini.py:458
messages.append(types.Content(role='user', parts=[types.Part.from_text(text=prompt)]))
```

**Input cho Gemini:**
- ✅ **System prompt** (từ `gpt_config.yaml`)
- ✅ **Text cần dịch** (từ OCR)
- ❌ **KHÔNG có ảnh**

### ChatGPT 2-Stage (Đặc biệt):
```python
# manga_translator/translators/chatgpt_2stage.py:175-177
{"role": "user", "content": [
    {"type": "text", "text": refine_prompt},
    {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{base64_img}"}}
]}
```

**Input cho ChatGPT 2-Stage:**
- ✅ **System prompt**
- ✅ **Text cần dịch**
- ✅ **Ảnh** (chỉ để OCR correction)

---

## 📝 Ví Dụ Cụ Thể:

### Input ảnh manga:
```
[Ảnh manga có text tiếng Trung]
```

### Sau OCR:
```
Text regions: [
  "你好世界",
  "这是测试",
  "漫画翻译"
]
```

### Gửi cho Gemini:
```json
{
  "system_instruction": "Bạn là chuyên gia dịch tiếng Trung sang Tiếng Việt...",
  "user_content": "<|1|>你好世界<br><|2|>这是测试<br><|3|>漫画翻译"
}
```

### Gemini trả về:
```
"<|1|>Xin chào thế giới<br><|2|>Đây là thử nghiệm<br><|3|>Dịch manga"
```

### Sau render:
```
[Ảnh manga với text tiếng Việt đã vẽ lên]
```

---

## 🎯 Kết Luận:

| Translator | Nhận Ảnh? | Nhận Text? | Mục đích |
|------------|-----------|------------|----------|
| **Gemini** | ❌ Không | ✅ Có | Dịch text |
| **ChatGPT** | ❌ Không | ✅ Có | Dịch text |
| **ChatGPT 2-Stage** | ✅ Có | ✅ Có | OCR correction + dịch text |
| **Offline (NLLB, etc)** | ❌ Không | ✅ Có | Dịch text |

**Đa số model chỉ nhận TEXT, không nhận ảnh trực tiếp.**

---

## 📚 Nguồn Tham Khảo:

- **Flow chính:** `manga_translator/manga_translator.py:432-562`
- **Translation:** `manga_translator/manga_translator.py:1057-1096`
- **Gemini:** `manga_translator/translators/gemini.py:426-484`
- **ChatGPT 2-Stage:** `manga_translator/translators/chatgpt_2stage.py:354-391`
- **OCR:** `manga_translator/ocr/model_manga_ocr.py:132-166`



