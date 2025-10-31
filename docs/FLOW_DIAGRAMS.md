# Manga Image Translator - Flow Diagrams

## 📊 Diagram 1: Tổng Quan Pipeline

```mermaid
graph TD
    A[Input Image] --> B{Colorization?}
    B -->|Yes| C[Apply Colorization]
    B -->|No| D[Upscaling?]
    C --> D
    D -->|Yes| E[Upscale Image]
    D -->|No| F[Detection]
    E --> F
    F --> G[OCR]
    G --> H[Textline Merge]
    H --> I[Translation]
    I --> J[Mask Refinement]
    J --> K[Inpainting]
    K --> L[Rendering]
    L --> M[Output Image]
    
    style A fill:#90EE90
    style M fill:#90EE90
    style F fill:#FFD700
    style G fill:#FFD700
    style I fill:#FFD700
    style K fill:#FFD700
    style L fill:#FFD700
```

---

## 📊 Diagram 2: Context Object Flow

```mermaid
stateDiagram-v2
    [*] --> Created: Create Context
    Created --> InputLoaded: Load Image
    InputLoaded --> DetectionDone: Detection
    DetectionDone --> OCRDone: OCR
    OCRDone --> Merged: Merge Textlines
    Merged --> Translated: Translation
    Translated --> MaskRefined: Mask Refinement
    MaskRefined --> Inpainted: Inpainting
    Inpainted --> Rendered: Rendering
    Rendered --> Complete: Save Result
    Complete --> [*]
    
    note right of InputLoaded
        ctx.input = PIL.Image
    end note
    
    note right of DetectionDone
        ctx.text_regions = []
        ctx.panels = []
    end note
    
    note right of OCRDone
        ctx.textlines = [TextLine]
    end note
    
    note right of Translated
        ctx.textlines[].translation
    end note
    
    note right of Complete
        ctx.result = PIL.Image
    end note
```

---

## 📊 Diagram 3: Web Server Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        C1[Web Browser]
        C2[Mobile App]
        C3[API Client]
    end
    
    subgraph "Web Server (FastAPI)"
        WS[Main Server<br/>server/main.py]
        
        subgraph "API Endpoints"
            E1[/translate/image]
            E2[/translate/image/stream]
            E3[/translate/batch/json]
            E4[/queue-size]
        end
        
        subgraph "Request Processing"
            RE[Request Extraction<br/>server/request_extraction.py]
            TQ[Task Queue<br/>server/myqueue.py]
        end
        
        subgraph "Streaming"
            ST[Streaming Handler<br/>server/streaming.py]
            SD[Sent Data Internal<br/>server/sent_data_internal.py]
        end
    end
    
    subgraph "Translation Instances"
        subgraph "Instance Manager"
            IM[Executor Instances<br/>server/instance.py]
        end
        
        subgraph "Translation Workers"
            TW1[Translator Instance 1]
            TW2[Translator Instance 2]
            TW3[Translator Instance 3]
        end
    end
    
    subgraph "Core Processing"
        MT[MangaTranslator<br/>manga_translator/manga_translator.py]
        
        subgraph "Processing Modules"
            DET[Detection]
            OCR[OCR]
            TRANS[Translation]
            INP[Inpainting]
            REND[Rendering]
        end
    end
    
    C1 --> E1
    C2 --> E2
    C3 --> E3
    
    E1 --> RE
    E2 --> RE
    E3 --> RE
    
    RE --> TQ
    TQ --> IM
    IM --> TW1
    IM --> TW2
    IM --> TW3
    
    TW1 --> MT
    TW2 --> MT
    TW3 --> MT
    
    MT --> DET
    DET --> OCR
    OCR --> TRANS
    TRANS --> INP
    INP --> REND
    
    REND --> SD
    SD --> ST
    ST --> C1
    
    style WS fill:#4169E1,color:#FFF
    style MT fill:#32CD32,color:#FFF
    style TQ fill:#FF6347,color:#FFF
    style IM fill:#FFD700,color:#000
```

---

## 📊 Diagram 4: Task Queue Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Endpoint
    participant RQ as Request Extraction
    participant Q as Task Queue
    participant E as Executor Manager
    participant W as Translation Worker
    participant S as Streaming Handler
    
    C->>API: POST /translate/image/stream
    API->>RQ: Parse request
    
    Note over RQ: Create QueueElement
    Note over RQ: image, config, req
    
    RQ->>Q: Add to queue
    Q-->>RQ: Return task
    
    RQ->>Q: Wait in queue
    loop Check Position
        Q->>RQ: Position = 5 (example)
        Note over RQ: position >= free_executors()
        RQ->>E: Waiting...
    end
    
    Note over E: Free executor available
    E->>W: Assign task
    Q->>Q: Remove from queue
    
    W->>S: Process translation
    loop Streaming Updates
        W->>S: Progress update (code 1)
        S->>C: Stream chunk
    end
    
    W->>S: Result ready (code 0)
    S->>C: Final image
    W->>E: Release executor
    E->>E: Mark free
```

---

## 📊 Diagram 5: Translation Pipeline Sequence

```mermaid
sequenceDiagram
    participant MT as MangaTranslator
    participant C as Context
    participant DET as Detector
    participant OCR as OCR Module
    participant TX as Translator
    participant MSK as Mask Refinement
    participant INP as Inpainter
    participant RND as Renderer
    
    MT->>C: Create Context(input=image)
    
    Note over MT: Optional: Colorization
    MT->>DET: Detect text regions
    
    DET-->>C: text_regions, panels
    MT->>OCR: Extract text
    
    loop For each region
        OCR-->>C: textlines.append(TextLine)
    end
    
    Note over MT: Merge textlines
    MT->>TX: Translate texts
    
    TX-->>C: textlines[].translation
    MT->>MSK: Refine masks
    MSK-->>C: refined_masks
    
    MT->>INP: Inpaint (remove text)
    INP-->>C: inpaint_result
    
    MT->>RND: Render translated text
    RND-->>C: result (final image)
    
    C->>C: ctx.result saved
```

---

## 📊 Diagram 6: Batch Translation Flow

```mermaid
graph TD
    A[Start Batch] --> B{Read Config}
    B --> C[batch_size = N]
    C --> D[Load Images]
    
    D --> E{Split into batches}
    E -->|Batch 1| F1[Process Image 1-N]
    E -->|Batch 2| F2[Process Image N+1-2N]
    E -->|Batch N| FN[Process Image N*M+1-end]
    
    F1 --> G1[Context 1]
    F2 --> G2[Context 2]
    FN --> GN[Context N]
    
    G1 --> H{Concurrent Mode?}
    G2 --> H
    GN --> H
    
    H -->|Yes| I[Share context<br/>between pages]
    H -->|No| J[Independent processing]
    
    I --> K[Collect Results]
    J --> K
    
    K --> L[Save Outputs]
    L --> M[End]
    
    style A fill:#90EE90
    style M fill:#90EE90
    style K fill:#FFD700
```

---

## 📊 Diagram 7: Stream Format Structure

```mermaid
graph LR
    subgraph "Stream Chunk"
        A[Status: 1 byte] --> B[Size: 4 bytes big-endian]
        B --> C[Data: N bytes]
    end
    
    subgraph "Status Codes"
        D[0 = Result Data]
        E[1 = Progress Report]
        F[2 = Error]
        G[3 = Queue Position]
        H[4 = Translation Started]
    end
    
    A -.-> D
    A -.-> E
    A -.-> F
    A -.-> G
    A -.-> H
    
    style A fill:#FFD700
    style B fill:#FFD700
    style C fill:#90EE90
```

---

## 📊 Diagram 8: Model Lifecycle

```mermaid
stateDiagram-v2
    [*] --> NotLoaded
    
    NotLoaded --> Downloading: First Use
    Downloading --> Downloaded
    Downloaded --> Loading: Model Needed
    Loading --> InMemory
    InMemory --> Processing: Translation Request
    Processing --> InMemory: Continue Processing
    InMemory --> Unloading: TTL Expired
    Unloading --> NotLoaded
    
    InMemory --> NotLoaded: Force Unload
    
    note right of Downloading
        Auto-download models
        to ./models/
    end note
    
    note right of InMemory
        Track last usage time
        Background cleanup task
    end note
    
    note right of Unloading
        models_ttl seconds
        elapsed since last use
    end note
```

---

## 📊 ASCII Art: Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    MANGA IMAGE TRANSLATOR                       │
└─────────────────────────────────────────────────────────────────┘

                        ┌─────────────┐
                        │ INPUT IMAGE │
                        │  (PIL.Image)│
                        └──────┬──────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ 1. COLORIZATION      │ (Optional)
                    │    Apply colorizer   │
                    └──────┬───────────────┘
                           │
                           ▼
                    ┌─────────────────────┐
                    │ 2. UPSCALING         │ (Optional)
                    │    Enhance resolution │
                    └──────┬───────────────┘
                           │
                           ▼
        ┌───────────────────────────────────────┐
        │ 3. DETECTION - Find Text Regions      │
        │    - Detector models (DB, CTD, etc)   │
        │    Output: text_regions, panels        │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 4. OCR - Extract Text                  │
        │    - 48px, 32px, Manga OCR             │
        │    Output: TextLine objects            │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 5. TEXTLINE MERGE                      │
        │    - Combine related lines             │
        │    - Group by bubbles/panels          │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 6. TRANSLATION                         │
        │    - Pre-dict replacement              │
        │    - Online/Offline translators       │
        │    - Post-dict replacement            │
        │    Output: translated texts           │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 7. MASK REFINEMENT                     │
        │    - Expand text masks                 │
        │    - Smooth edges                      │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 8. INPAINTING - Remove Old Text        │
        │    - LaMa, Stable Diffusion            │
        │    Output: image without text          │
        └──────┬────────────────────────────────┘
               │
               ▼
        ┌───────────────────────────────────────┐
        │ 9. RENDERING - Add New Text           │
        │    - Font selection & sizing          │
        │    - Color detection                   │
        │    - Alignment                         │
        │    Output: FINAL IMAGE                 │
        └──────┬────────────────────────────────┘
               │
               ▼
                    ┌─────────────┐
                    │OUTPUT IMAGE │
                    └─────────────┘
```

---

## 📊 ASCII Art: Web Server Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                          WEB SERVER ARCHITECTURE                     │
└──────────────────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │   Client     │
    │ (Browser/App)│
    └──────┬───────┘
           │ HTTP POST
           ▼
    ┌────────────────────────────────────┐
    │        FastAPI Server               │
    │     server/main.py                  │
    │                                     │
    │  ┌──────────────────────┐          │
    │  │  API Endpoints:      │          │
    │  │  • /translate/image  │          │
    │  │  • /translate/json   │          │
    │  │  • /batch/json       │          │
    │  └──────────┬───────────┘          │
    │             │                       │
    │             ▼                       │
    │  ┌──────────────────────┐          │
    │  │ Request Extraction   │          │
    │  │ request_extraction.py│          │
    │  └──────────┬───────────┘          │
    └──────────────┼──────────────────────┘
                   │
                   ▼
    ┌──────────────────────────────────────┐
    │         Task Queue Manager            │
    │         myqueue.py                    │
    │                                       │
    │  QueueElement:                       │
    │  - image: PIL.Image                  │
    │  - config: Config                    │
    │  - req: Request                      │
    └───────────┬──────────────────────────┘
                │
                │ Wait for executor
                ▼
    ┌──────────────────────────────────────┐
    │     Executor Instance Manager         │
    │         instance.py                   │
    │                                       │
    │  ┌─────────┐  ┌─────────┐  ┌───────┐│
    │  │ Exec 1  │  │ Exec 2  │  │Exec 3 ││
    │  └─────────┘  └─────────┘  └───────┘│
    └───────────┬──────────────────────────┘
                │
                │ Assign task
                ▼
    ┌──────────────────────────────────────┐
    │      Translation Worker               │
    │    MangaTranslator                   │
    │                                       │
    │  ┌─────────────────────┐             │
    │  │  Full Pipeline:    │             │
    │  │  1. Detection      │             │
    │  │  2. OCR            │             │
    │  │  3. Translation    │             │
    │  │  4. Inpainting     │             │
    │  │  5. Rendering      │             │
    │  └─────────────────────┘             │
    └───────────┬──────────────────────────┘
                │
                │ Result
                ▼
    ┌──────────────────────────────────────┐
    │       Streaming Handler               │
    │       streaming.py                   │
    │                                       │
    │  Format chunks:                      │
    │  [status(1)][size(4)][data(N)]      │
    └───────────┬──────────────────────────┘
                │
                │ Stream chunks
                ▼
    ┌──────────────┐
    │   Client     │
    │  Receives    │
    │  Result      │
    └──────────────┘
```

---

## 📊 ASCII Art: Context Data Flow

```
┌────────────────────────────────────────────────────────────────────┐
│                      CONTEXT OBJECT - DATA FLOW                    │
└────────────────────────────────────────────────────────────────────┘

Create Context:
    ctx = Context(input=image)
    └─> ctx.input: PIL.Image
        └─> Original input image
    
    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
    │ CONTEXT OBJECT                      │
    │                                     │
    ├─ Step 1: Detection                 │
    │  ├─> ctx.text_regions: []          │
    │  │   └─> List of detected polygons │
    │  └─> ctx.panels: []                 │
    │      └─> List of panel regions      │
    │                                     │
    ├─ Step 2: OCR                        │
    │  └─> ctx.textlines: [TextLine]     │
    │      ├─ TextLine.text: "原文"       │
    │      ├─ TextLine.bbox: (x,y,w,h)   │
    │      └─ TextLine.translation: ""   │
    │                                     │
    ├─ Step 3: Translation               │
    │  └─> ctx.textlines[].translation = "Translation"
    │                                     │
    ├─ Step 4: Inpainting                │
    │  └─> ctx.inpaint_result: PIL.Image │
    │                                     │
    ├─ Step 5: Rendering                  │
    │  └─> ctx.result: PIL.Image         │
    │      └─> FINAL OUTPUT              │
    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
```

---

## 📊 Mermaid: Error Handling Flow

```mermaid
flowchart TD
    Start[Start Processing] --> TryCatch[Enter Try Block]
    
    TryCatch --> Detection{Detection Error?}
    Detection -->|No| OCR{OCR Error?}
    Detection -->|Yes| DET_ERR[Log Error]
    DET_ERR --> DET_HANDLE{Ignore Errors?}
    DET_HANDLE -->|Yes| Skip[Skip Image]
    DET_HANDLE -->|No| RetryDET{Retries < max?}
    RetryDET -->|Yes| Detection
    RetryDET -->|No| Skip
    
    OCR -->|No| Translation{Translation Error?}
    OCR -->|Yes| OCR_ERR[Log Error]
    OCR_ERR --> OCR_HANDLE{Ignore Errors?}
    OCR_HANDLE -->|Yes| Skip
    OCR_HANDLE -->|No| RetryOCR{Retries < max?}
    RetryOCR -->|Yes| OCR
    RetryOCR -->|No| Skip
    
    Translation -->|No| Inpainting{Inpainting Error?}
    Translation -->|Yes| TRANS_ERR[Log Error]
    TRANS_ERR --> TRANS_HANDLE{Ignore Errors?}
    TRANS_HANDLE -->|Yes| Skip
    TRANS_HANDLE -->|No| RetryTRANS{Retries < max?}
    RetryTRANS -->|Yes| Translation
    RetryTRANS -->|No| Skip
    
    Inpainting -->|No| Rendering{Rendering Error?}
    Inpainting -->|Yes| INP_ERR[Log Error]
    INP_ERR --> INP_HANDLE{Ignore Errors?}
    INP_HANDLE -->|Yes| Skip
    INP_HANDLE -->|No| RetryINP{Retries < max?}
    RetryINP -->|Yes| Inpainting
    RetryINP -->|No| Skip
    
    Rendering -->|No| Success[Save Result]
    Rendering -->|Yes| REND_ERR[Log Error]
    REND_ERR --> REND_HANDLE{Ignore Errors?}
    REND_HANDLE -->|Yes| Skip
    REND_HANDLE -->|No| RetryREND{Retries < max?}
    RetryREND -->|Yes| Rendering
    RetryREND -->|No| Skip
    
    Success --> End[Complete]
    Skip --> End
    
    style Success fill:#90EE90
    style Skip fill:#FF6347
    style End fill:#90EE90
```

---

## 📊 Mermaid: Model TTL System

```mermaid
graph TD
    A[Start Processing] --> B[Check models_ttl]
    B -->|models_ttl == 0| C[Keep models forever]
    B -->|models_ttl > 0| D[Setup TTL tracking]
    
    C --> E[Load models when needed]
    E --> F[Process image]
    F --> G[Keep in memory]
    
    D --> H[Load models]
    H --> I[Record usage time]
    I --> J[Process image]
    J --> K[Update usage time]
    
    K --> L{Background cleanup task}
    L -->|Every 30s| M[Check TTL]
    M --> N{Time since last use > TTL?}
    
    N -->|Yes| O[Unload model]
    N -->|No| P[Keep in memory]
    
    O --> Q[Free GPU/CPU memory]
    P --> L
    
    Q --> R[Next image needs model?]
    R -->|Yes| H
    R -->|No| S[Idle]
    S --> L
    
    style E fill:#90EE90
    style H fill:#90EE90
    style O fill:#FF6347
    style P fill:#FFD700
```

---

## 📊 Summary Table: Module Responsibilities

| Module | Location | Responsibility | Dependencies |
|--------|----------|----------------|-------------|
| **MangaTranslator** | `manga_translator.py` | Main orchestrator | All modules |
| **Detection** | `detection/*.py` | Find text regions | DB, CTD models |
| **OCR** | `ocr/*.py` | Extract text | OCR models (48px, etc) |
| **Translation** | `translators/*.py` | Translate text | API services, models |
| **Inpainting** | `inpainting/*.py` | Remove old text | LaMa, SD models |
| **Rendering** | `rendering/*.py` | Add new text | PIL, Pillow |
| **Config** | `config.py` | Configuration | Omegaconf |
| **Queue** | `server/myqueue.py` | Task management | asyncio |
| **Server** | `server/main.py` | HTTP interface | FastAPI |

---

## 📊 Usage Examples

### Example 1: Local Batch Mode
```bash
python -m manga_translator local \
    -i /path/to/images \
    -o /path/to/output \
    --use-gpu \
    -v
```

### Example 2: Web Server
```bash
# Terminal 1: Start web server
python server/main.py --start-instance --use-gpu

# Terminal 2: Upload via curl
curl -X POST http://localhost:8000/translate/with-form/image/stream \
    -F "image=@image.png" \
    -F "config={\"translator\":{\"translator\":\"sugoi\"}}"
```

### Example 3: Python API
```python
from manga_translator import MangaTranslator
from manga_translator import Config

translator = MangaTranslator({
    'use_gpu': True,
    'verbose': True
})

config = Config()
config.translator.translator = 'sugoi'
config.translator.target_lang = 'ENG'

result = await translator.translate_path(
    'input.png', 
    'output.png', 
    vars(config)
)
```

---

## 🔗 Related Files

- **Main Flow**: `manga_translator/manga_translator.py`
- **Context**: `manga_translator/utils/generic.py:28-72`
- **Detection**: `manga_translator/detection/`
- **OCR**: `manga_translator/ocr/`
- **Translation**: `manga_translator/translators/`
- **Inpainting**: `manga_translator/inpainting/`
- **Rendering**: `manga_translator/rendering/`
- **Server**: `server/main.py`
- **Queue**: `server/myqueue.py`

---

*Generated diagrams for Manga Image Translator project*


