<div align="center">

# VisualSearch — AI Image Similarity Search Engine

> Find visually similar images instantly using CLIP embeddings and vector search.

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?logo=fastapi)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react)](https://react.dev)
[![CLIP](https://img.shields.io/badge/CLIP-ViT--B/32-412991?logo=openai)](https://openai.com/research/clip)
[![ChromaDB](https://img.shields.io/badge/ChromaDB-0.5-orange)](https://trychroma.com)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)](https://docker.com)

</div>
<img src="assets/image_search.svg" width="100%">

---

## Features

| Feature                  | Description                                             |
| ------------------------ | ------------------------------------------------------- |
| **Image Search**         | Upload any image → get top-N visually similar results   |
| **Text Search**          | Search by natural language description (CLIP zero-shot) |
| **Multi-modal Search**   | Combine image + text for refined results                |
| **Reverse Image Search** | Find near-duplicates of any indexed image               |
| **Bulk Upload**          | Index up to 20 images at once                           |
| **Collection Browser**   | Paginated gallery of all indexed images                 |
| **REST API**             | Full OpenAPI documentation                              |
| **Docker**               | One-command deployment                                  |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        User Browser                          │
│                    React Frontend (port 3000)                │
│         Search │ Upload │ Collection Management UI           │
└─────────────────────────┬────────────────────────────────────┘
                          │ HTTP / REST
                          ▼
┌──────────────────────────────────────────────────────────────┐
│                  FastAPI Backend (port 8000)                 │
│                                                              │
│  ┌────────────┐  ┌────────────┐  ┌───────────────────────┐   │
│  │  /api/     │  │  /api/     │  │  /api/collection/     │   │
│  │  search/   │  │  upload    │  │  images / stats       │   │
│  │  image     │  │  bulk      │  │  clear                │   │
│  │  text      │  │            │  │                       │   │
│  │  multimodal│  │            │  │                       │   │
│  │  reverse   │  │            │  │                       │   │
│  └─────┬──────┘  └─────┬──────┘  └───────────┬───────────┘   │
│        │               │                     │               │
│        ▼               ▼                     │               │
│  ┌─────────────────────────────┐             │               │
│  │    EmbeddingService         │             │               │
│  │  CLIP ViT-B/32 (512-dim)    │             │               │
│  │  • embed_image()            │             │               │
│  │  • embed_text()             │             │               │
│  │  • embed_image_text()       │             │               │
│  └─────────────┬───────────────┘             │               │
│                │                             │               │
│                ▼                             ▼               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │            VectorStoreService (ChromaDB)            │     │
│  │   index_image() │ search_similar() │ get_image()    │     │
│  │   list_images() │ delete_image()   │ clear()        │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  ┌──────────────────────┐   ┌──────────────────────────┐     │
│  │   ChromaDB           │   │   Local File Storage     │     │
│  │   chroma_db/         │   │   uploads/               │     │
│  │   (Cosine similarity)│   │   (Static files)         │     │
│  └──────────────────────┘   └──────────────────────────┘     │
└──────────────────────────────────────────────────────────────┘
```

### Data Flow

```
Query Image ──► CLIP Vision Encoder ──► 512-dim embedding
                                               │
                                               ▼
                                    ChromaDB cosine search
                                               │
                                               ▼
                              Top-K results (sorted by similarity)
                                               │
                                               ▼
                                      JSON response to UI
```

---

## Quick Start

### Option A: Docker Compose (Recommended)

```bash
# Clone / unzip the project
cd image-search-engine

# Start everything
docker compose up --build

# App: http://localhost:3000
# API: http://localhost:8000
# Docs: http://localhost:8000/api/docs
```

> **Note:** First run downloads the CLIP model (~600 MB). Subsequent starts use the cache.

### Option B: Local Development

**Backend**

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Seed sample images (optional)
python scripts/seed_images.py

# Start server
uvicorn app.main:app --reload --port 8000
```

**Frontend**

```bash
cd frontend

npm install
npm start      # http://localhost:3000
```

---

## Project Structure

```
image-search-engine/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI app + lifespan
│   │   ├── api/
│   │   │   ├── search.py            # Search endpoints
│   │   │   ├── upload.py            # Upload + delete endpoints
│   │   │   ├── collection.py        # Collection management
│   │   │   └── health.py            # Health check
│   │   ├── core/
│   │   │   ├── config.py            # Pydantic settings
│   │   │   └── database.py          # ChromaDB init
│   │   ├── models/
│   │   │   └── schemas.py           # Pydantic response models
│   │   ├── services/
│   │   │   ├── embedding_service.py # CLIP embeddings (singleton)
│   │   │   └── vector_store_service.py # ChromaDB operations
│   │   └── utils/
│   │       └── image_utils.py       # PIL helpers, file ops
│   ├── scripts/
│   │   └── seed_images.py           # Demo data seeder
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── App.js                   # Router + Nav
│   │   ├── App.css                  # All styles
│   │   ├── pages/
│   │   │   ├── SearchPage.js        # Search UI (image/text/multimodal)
│   │   │   ├── UploadPage.js        # Single + bulk upload
│   │   │   └── CollectionPage.js    # Gallery + management
│   │   ├── components/
│   │   │   ├── ImageDropzone.js     # Drag-and-drop file input
│   │   │   ├── ResultCard.js        # Search result card
│   │   │   └── Toast.js             # Notification system
│   │   └── utils/
│   │       └── api.js               # Axios API calls
│   ├── public/index.html
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml
├── docker-compose.dev.yml
└── README.md
```

---

## API Reference

### Base URL: `http://localhost:8000`

### Health

| Method | Endpoint      | Description            |
| ------ | ------------- | ---------------------- |
| GET    | `/api/health` | Service health + stats |

```bash
curl http://localhost:8000/api/health
# {"status":"healthy","model_loaded":true,"collection_count":20,"version":"1.0.0"}
```

---

### Upload

#### `POST /api/upload` — Upload & index a single image

```bash
curl -X POST http://localhost:8000/api/upload \
  -F "file=@car.jpg" \
  -F "tags=car,red,sports" \
  -F "description=A red Ferrari sports car"
```

**Response:**

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "filename": "3fa85f64.jpg",
  "url": "http://localhost:8000/uploads/3fa85f64.jpg",
  "width": 1920,
  "height": 1080,
  "format": "JPEG",
  "file_size": 245120,
  "embedding_dim": 512,
  "indexed": true,
  "message": "Image indexed successfully in 180ms"
}
```

#### `POST /api/upload/bulk` — Upload multiple images

```bash
curl -X POST http://localhost:8000/api/upload/bulk \
  -F "files=@img1.jpg" \
  -F "files=@img2.jpg" \
  -F "files=@img3.png"
```

#### `DELETE /api/images/{id}` — Delete an indexed image

```bash
curl -X DELETE http://localhost:8000/api/images/3fa85f64-5717-4562-b3fc-2c963f66afa6
```

---

### Search

#### `POST /api/search/image` — Search by image

```bash
curl -X POST http://localhost:8000/api/search/image \
  -F "file=@query.jpg" \
  -F "top_k=5" \
  -F "min_similarity=0.5"
```

**Response:**

```json
{
  "query_image_id": null,
  "query_text": null,
  "total_results": 5,
  "search_time_ms": 42.3,
  "results": [
    {
      "id": "abc123",
      "filename": "ferrari.jpg",
      "url": "http://localhost:8000/uploads/ferrari.jpg",
      "similarity_score": 0.9234,
      "distance": 0.1532,
      "width": 1920,
      "height": 1080,
      "description": "Red Ferrari sports car",
      "tags": ["car", "red", "sports"]
    }
  ]
}
```

#### `GET /api/search/text` — Search by text

```bash
curl "http://localhost:8000/api/search/text?q=red+sports+car&top_k=5"
```

#### `POST /api/search/multimodal` — Image + text combined search

```bash
curl -X POST http://localhost:8000/api/search/multimodal \
  -F "file=@car.jpg" \
  -F "text=racing track" \
  -F "alpha=0.7" \
  -F "top_k=5"
```

> `alpha`: 0.0 = text only, 1.0 = image only, 0.7 = 70% image + 30% text

#### `GET /api/search/reverse/{id}` — Reverse image search

```bash
curl "http://localhost:8000/api/search/reverse/3fa85f64-...?top_k=5"
```

---

### Collection

| Method | Endpoint                      | Description             |
| ------ | ----------------------------- | ----------------------- |
| GET    | `/api/collection/stats`       | Total count, model info |
| GET    | `/api/collection/images`      | List images (paginated) |
| GET    | `/api/collection/images/{id}` | Get single image        |
| DELETE | `/api/collection/clear`       | Clear all images        |

```bash
# List images page 2
curl "http://localhost:8000/api/collection/images?limit=20&offset=20"

# Stats
curl "http://localhost:8000/api/collection/stats"
```

**Interactive docs:** http://localhost:8000/api/docs

---

## Configuration

| Variable           | Default                        | Description            |
| ------------------ | ------------------------------ | ---------------------- |
| `CLIP_MODEL`       | `openai/clip-vit-base-patch32` | HuggingFace model ID   |
| `EMBEDDING_DIM`    | `512`                          | Vector dimensions      |
| `UPLOAD_DIR`       | `uploads`                      | Image storage path     |
| `CHROMA_DIR`       | `chroma_db`                    | Vector DB path         |
| `DEFAULT_TOP_K`    | `5`                            | Default search results |
| `MAX_FILE_SIZE_MB` | `10`                           | Max upload size        |
| `DEBUG`            | `false`                        | Enable debug logging   |

---

## How It Works

1. **Embedding Generation** — Images are processed through `openai/clip-vit-base-patch32`, a vision-language model that produces 512-dimensional embeddings capturing both visual and semantic features.

2. **Vector Storage** — Embeddings are stored in ChromaDB with `hnsw:space=cosine`. HNSW (Hierarchical Navigable Small World) enables sub-linear approximate nearest-neighbor search.

3. **Similarity Search** — Query embedding is compared against all stored embeddings using cosine similarity. ChromaDB returns distances in [0, 2] range, converted to similarity scores in [0, 1].

4. **Multi-modal Fusion** — For image+text queries, both embeddings are computed and linearly blended: `embedding = α * image_emb + (1-α) * text_emb`, then L2-normalized.

---

## Seeding Demo Data

```bash
cd backend
python scripts/seed_images.py
```

Seeds 20 sample images (cars, nature, animals, cityscapes) downloaded from Unsplash.

---

## 🐳 Docker Commands

```bash
# Start
docker compose up --build -d

# Logs
docker compose logs -f backend

# Seed data in container
docker compose exec backend python scripts/seed_images.py

# Stop
docker compose down

# Stop and remove volumes
docker compose down -v
```

---

## Performance

| Metric                       | Value                      |
| ---------------------------- | -------------------------- |
| Embedding generation         | ~150–300ms per image (CPU) |
| Search latency (1K images)   | < 10ms                     |
| Search latency (100K images) | < 50ms                     |
| CLIP model size              | ~600 MB                    |
| GPU acceleration             | Auto-detected (CUDA)       |

---

## Bonus Features Implemented

- ✅ **Text search** — CLIP zero-shot text embeddings
- ✅ **Multi-modal search** — Image + text fusion with configurable alpha
- ✅ **Reverse image search** — Find near-duplicates by ID
- ✅ **Docker deployment** — Full docker-compose setup
- ✅ **Bulk upload** — Up to 20 images per request
- ✅ **Similarity score** — Cosine similarity [0–1] displayed per result
- ✅ **Collection management** — Browse, delete, clear
- ✅ **Persistent storage** — ChromaDB + file system volumes

---

## 👤 Author

- **Rohith Gowda R** (<rohithgowdax@gmail.com>)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
