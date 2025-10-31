# Flow & Vận hành của Manga Image Translator

## 📋 Tổng quan Project

**Manga Image Translator** là một công cụ dịch ảnh manga/comic tự động, có khả năng:
- Phát hiện và nhận dạng text trong ảnh (Detection + OCR)
- Dịch text (Translation) 
- Xóa text cũ và chèn text mới vào ảnh (Inpainting + Rendering)
- Hỗ trợ 20+ ngôn ngữ, chủ yếu là Japanese, Chinese, English
- Hỗ trợ nhiều chế độ: Local batch, Web mode, API mode, WebSocket

---

## 🏗️ Kiến trúc Tổng quan

### Thành phần chính:

```
┌─────────────────────────────────────────────────────────────┐
│                   Manga Image Translator                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Detection   │  │     OCR      │  │  Translation │      │
│  │  (Detector)  │→ │   (Model)   │→ │  (Service)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Inpainting   │  │  Rendering   │  │   Output    │      │
│  │   (Model)    │→ │   (Text)     │→ │   Image     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Flow Xử Lý Chi Tiết

### Bước 1: Input & Preprocessing
```python
# Location: manga_translator/manga_translator.py:1697-1750

Input: Image (PIL.Image)
↓
1. Load image → Context.input
2. (Optional) Colorization - Tô màu cho ảnh đen trắng
3. (Optional) Upscaling - Phóng to ảnh để cải thiện detection
```

### Bước 2: Detection & Text Region Finding
```python
# Location: manga_translator/detection/*.py

Detector models:
- default: DB-CNN (DBNet)
- ctd: Text detector tốt hơn cho manga
- dbconvnext: Enhanced version
- paddle: PaddleOCR detector

Output: List of text regions (bounding boxes + text regions)
↓
Context.text_regions = detected_regions
Context.panels = detected_panels
```

### Bước 3: OCR - Optical Character Recognition
```python
# Location: manga_translator/ocr/*.py

OCR models:
- 48px: Default Japanese OCR
- 32px: Basic OCR
- mocr: Manga OCR

Process:
- For each text region → Extract text
- Detect source language
- Store original text

Output: Context.textlines = [
    TextLine(text="原文", bbox=bbox, translation=""),
    ...
]
```

### Bước 4: Textline Merging
```python
# Location: manga_translator/textline_merge/__init__.py

Purpose: Merge related textlines together
- Combine textlines in same bubble
- Handle multi-line text
- Group by panels

Output: Merged textlines
```

### Bước 5: Translation
```python
# Location: manga_translator/translators/*.py

Translators available:
- Online: ChatGPT, DeepL, Baidu, Youdao, ...
- Offline: Sugoi, NLLB, M2M100, Qwen2, ...
- Special: ChatGTP 2-stage, Gemini JSON mode

Process:
1. Dispatch translation request
2. Apply pre-dict (before translation)
3. Translate text
4. Apply post-dict (after translation)

Output: Context.textlines with translations
```

### Bước 6: Mask Refinement
```python
# Location: manga_translator/mask_refinement/text_mask_utils.py

Purpose: Refine text masks before inpainting
- Expand mask to cover full text area
- Apply dilation offset
- Smooth mask edges
```

### Bước 7: Inpainting - Xóa Text Cũ
```python
# Location: manga_translator/inpainting/*.py

Inpainter models:
- lama_large: LaMa model (default, good quality)
- lama_mpe: Enhanced LaMa
- sd: Stable Diffusion inpainting

Process:
1. Create mask from text regions
2. Apply inpainting model to remove old text
3. Fill in background seamlessly

Output: Image with text removed
```

### Bước 8: Rendering - Chèn Text Mới
```python
# Location: manga_translator/rendering/*.py

Renderers:
- default: Basic text rendering with font
- manga2eng: Smart fitting to bubbles
- manga2eng_pillow: Enhanced Pillow-based

Features:
- Font size auto-adjustment
- Text alignment (left/center/right/auto)
- Direction handling (horizontal/vertical)
- Color detection from OCR
- Border rendering for visibility

Output: Final translated image
```

---

## 🎯 Context Object - Luồng Dữ Liệu

Context là dict-like object chứa toàn bộ thông tin xử lý:

```python
# Location: manga_translator/utils/generic.py:28-72

Context {
    input: PIL.Image              # Input image
    result: PIL.Image             # Final output
    
    # Detection results
    text_regions: List[Polygon]   # Detected text regions
    panels: List[Polygon]         # Detected panels
    
    # OCR & Translation results  
    textlines: List[TextLine]     # Text with translations
    from_lang: str               # Source language
    target_lang: str             # Target language
    
    # Processing results
    inpaint_result: PIL.Image    # After inpainting
    translate_result: PIL.Image   # After rendering
    
    # Metadata
    image_name: str              # Image filename
    result_path_callback: Callable  # Path for saving debug images
}
```

### Flow qua các bước:

```python
ctx = Context(input=image)
↓
ctx.text_regions = detection_result
ctx.panels = panel_detection_result
↓
ctx.textlines = ocr_result
↓
ctx.textlines[].translation = translation_result
↓
ctx.inpaint_result = inpainting_result
↓
ctx.result = rendering_result
```

---

## 🚀 Chế độ Hoạt động

### 1. **Local Batch Mode**
```python
# Location: manga_translator/__main__.py:dispatch()

Command: python -m manga_translator local -i <path> -o <dest>

Flow:
1. Load images from folder
2. For each image:
   - Process through full pipeline
   - Save to dest folder
3. Apply dictionaries (pre/post)
```

### 2. **Web Server Mode**
```python
# Location: server/main.py

Start: python server/main.py --start-instance

Features:
- Web UI tại http://localhost:8000
- FastAPI endpoints
- Real-time translation
- Streaming results
```

### 3. **API Mode**
```python
# Location: server/main.py, manga_translator/mode/share.py

Start: python -m manga_translator shared

Features:
- REST API endpoints
- Translation requests via HTTP
- Instance management
- Queue system
```

### 4. **WebSocket Mode**
```python
# Location: manga_translator/mode/ws.py

Start: python -m manga_translator ws --host 127.0.0.1 --port 5003

Features:
- Real-time bidirectional communication
- Progress updates
- Streaming translation
```

---

## 🔧 Queue System - Hàng đợi Xử lý

### Architecture
```python
# Location: server/myqueue.py

TaskQueue {
    queue: List[QueueElement | BatchQueueElement]
    queue_event: asyncio.Event
}

Process:
1. Client sends request → QueueElement created
2. Task added to queue
3. Wait for available executor instance
4. Process task
5. Return result
6. Remove from queue
```

### Queue Elements:
```python
class QueueElement:
    req: Request           # FastAPI request
    image: PIL.Image       # Image to process
    config: Config         # Translation config

class BatchQueueElement:
    req: Request
    images: List[PIL.Image]
    config: Config
    batch_size: int
```

### Worker Flow:
```python
# Location: server/myqueue.py:wait_in_queue()

while True:
    pos = get_position_in_queue()
    if pos < free_executors:
        instance = await find_executor()
        remove_from_queue()
        
        # Process
        if isinstance(task, BatchQueueElement):
            result = await instance.sent_batch(task.images, config)
        else:
            result = await instance.sent(task.image, config)
        
        return result
    else:
        await wait_for_event()
```

---

## 🌐 Web Server Flow

### Server Architecture
```
┌──────────────────────────────────────────────────────┐
│                    Web Server                         │
│                  (FastAPI + Uvicorn)                  │
└──────────────────────────────────────────────────────┘
                        ↓
        ┌───────────────────────────────┐
        │   Task Queue (TaskQueue)      │
        └───────────────────────────────┘
                        ↓
        ┌───────────────────────────────┐
        │  Executor Instances Manager   │
        │  (Executors.register())       │
        └───────────────────────────────┘
                        ↓
        ┌───────────────────────────────┐
        │   Translation Worker          │
        │   (MangaTranslator)           │
        └───────────────────────────────┘
```

### Request Flow:
```
1. Client → POST /translate/image/stream
2. server/request_extraction.py:
   - Parse image (base64, URL, or upload)
   - Create QueueElement
3. server/myqueue.py:
   - Add to queue
   - Wait in queue
4. When position < free_executors:
   - Get executor instance
   - Execute translation
5. Return streaming result
```

### Streaming Process:
```python
# Location: server/streaming.py, server/sent_data_internal.py

Stream format: (1byte status, 4byte size, ndata)

Status codes:
- 0: Result data (translated image)
- 1: Progress report
- 2: Error
- 3: Queue position update
- 4: Translation started

Example:
  00 00 00 00 10 [16 bytes image data]
  ↑      ↑    ↑       ↑
status  size  data
```

---

## 📦 Batch Translation Flow

### Batch Processing
```python
# Location: manga_translator/manga_translator.py:1456-1657

async def translate_batch(images_with_configs, batch_size):
    results = []
    
    # Process in batches
    for i in range(0, len(images), batch_size):
        batch = images[i:i+batch_size]
        
        # Translate each image in batch
        for image, config in batch:
            ctx = await _translate_until_translation(image, config)
            results.append(ctx)
    
    return results
```

### Concurrency Mode
```python
# When batch_concurrent = True

1. Process all images simultaneously
2. Share context between pages
3. Better translation quality with context awareness
```

---

## 🎛️ Configuration System

### Config Structure
```python
# Location: manga_translator/config.py

Config {
    filter_text: str
    render: RenderConfig
    upscale: UpscaleConfig
    translator: TranslatorConfig
    detector: DetectorConfig
    inpainter: InpainterConfig
    colorizer: ColorizerConfig
    ocr: OcrConfig
}
```

### Configuration Sources:
1. **Command line args** (highest priority)
2. **Config file** (`--config-file`)
3. **Default values** (lowest priority)

---

## 🧠 Model Loading & Management

### Model Lifecycle
```python
# Location: manga_translator/manga_translator.py

1. Parse config → Determine which models needed
2. Download models (auto-download on first use)
3. Load to memory (lazy loading)
4. Process image
5. (Optional) Unload models after TTL
```

### Model TTL System
```python
# models_ttl: Time to keep models in memory (seconds)
# 0 = forever, >0 = unload after TTL

Process:
1. Track last usage time
2. Background cleanup task runs periodically
3. Unload models not used for TTL seconds
```

---

## 🔍 Key Components Reference

### Detection
- **Purpose**: Find text regions in image
- **Models**: DBNet, CTD, PaddleOCR
- **Output**: List of bounding boxes

### OCR  
- **Purpose**: Extract text from regions
- **Models**: 48px, 32px, Manga OCR
- **Output**: TextLine objects with text + bbox

### Translation
- **Purpose**: Translate text
- **Services**: Online APIs, Offline models
- **Output**: Translated text

### Inpainting
- **Purpose**: Remove old text
- **Models**: LaMa, Stable Diffusion
- **Output**: Image with text removed

### Rendering
- **Purpose**: Add translated text
- **Methods**: Default, Manga2Eng
- **Output**: Final translated image

---

## 📝 Debugging & Logging

### Debug Output
```python
# When --verbose flag is set

Debug images saved to ./result/:
- input.png        # Original input
- colorized.png    # After colorization
- upscaled.png     # After upscaling  
- detected.png     # Detected regions visualization
- masks.png        # Text masks
- inpaint.png      # After inpainting
- render.png       # After rendering
- final.png        # Final result
```

### Logging System
```python
# Location: manga_translator/utils/log.py

1. Console output (with colors via Rich)
2. File logging (log_{timestamp}.txt)
3. Progress hooks (for streaming)
```

---

## 🚦 Error Handling

### Error Flow
```python
# Location: manga_translator/manga_translator.py

Try-catch at each stage:
1. Detection errors → Skip or retry
2. OCR errors → Continue with empty text
3. Translation errors → Use fallback
4. Inpainting errors → Retry or continue
5. Rendering errors → Skip or retry

Options:
- --ignore-errors: Skip on error
- --attempts: Retry count
```

---

## 🔗 References

- **Main flow**: `manga_translator/manga_translator.py`
- **Server**: `server/main.py`
- **Queue**: `server/myqueue.py`
- **Config**: `manga_translator/config.py`
- **Context**: `manga_translator/utils/generic.py`
- **Examples**: Check README.md for usage examples

---

*Document generated based on codebase analysis*


