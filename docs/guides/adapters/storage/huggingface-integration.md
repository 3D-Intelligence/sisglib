# HuggingFace Integration: Reducing Engineering Barriers in 3D Scene Generation


## Vision

**sisglib** is an open-source library for AI-driven scene generation.
Like **an OpenAI Gym for 3D scene generation**, we provide modular and composable strategy and pipeline interfaces for researchers to develop and benchmark prompt-to-3D scene generation algorithms (e.g. [Holodeck](https://yueyang1996.github.io/holodeck/)).


## Why

A key challenge in 3D scene generation is **data management**.
Typically, researchers need to:
- Download large datasets (e.g. [Objaverse](https://objaverse.allenai.org/))
- Preprocess data
- Set up vector databases for similarity search and retrieval (e.g. [Weaviate](https://weaviate.io/))
- Serve assets over HTTP (e.g. [HuggingFace Datasets](https://huggingface.co/datasets/allenai/objaverse/tree/main/glbs))

This is **time-consuming, error-prone, and requires expertise in infrastructure management**.


## Our Goal

**sisglib** aims to **simplify this process** by:
- **Unified Adapters**: Modular interfaces for metadata, vectors, and storage
- **HuggingFace Integration**: Trivially easy dataset distribution and consumption
- **Lazy Loading**: Download and serve assets on-demand over HTTP
- **Production-Ready**: From research prototype to deployment with minimal changes

This integration makes it **trivially easy to operationalize**:
- ✅ Asset serving and streaming directly from HuggingFace datasets
- ✅ Vector retrieval for semantic search
- ✅ Metadata search for filtering and querying


## Quick Start

### 1. Install sisglib

```bash
pip install sisglib
# Or for development:
pip install -e .
```

### 2. Text Similarity Search Using Holodeck Embeddings (CLIP)

To query for similar 3D objects by text, we first need to **download pre-computed embeddings** from [HuggingFace](https://huggingface.co/datasets/yunusskeete/holodeck-objaverse-embeddings/tree/main):

```python
import json
from typing import Any, Dict

from datasets import load_dataset
from huggingface_hub import hf_hub_download

from sisglib.adapters.metadata import JSONLocalAdapter
from sisglib.adapters.vectors import NumpyAdapter


# Download metadata files from HuggingFace
metadata_file = hf_hub_download(
    repo_id="yunusskeete/holodeck-objaverse-embeddings",
    filename="holodeck_metadata.json",
    repo_type="dataset"
)

# Load metadata adapter
metadata_adapter = JSONLocalAdapter(file_path=metadata_file)
print(f"✓ Metadata adapter loaded: {await metadata_adapter.count()} entries")


# Load vectors dataset (CLIP embeddings)
print("Loading vectors from HuggingFace...")
dataset = load_dataset(
    "yunusskeete/holodeck-objaverse-embeddings",
    data_files="vectors.parquet"
)
df = dataset["train"].to_pandas()  # Convert to pandas for faster bulk operations

# Initialize CLIP vector adapter
clip_adapter = NumpyAdapter()
# Vectorized assignment
clip_adapter.vectors = dict(zip(df["uuid"], df["clip_vector"]))
# Just get metadata column names
metadata_cols = [
    col
    for col in df.columns
    if col not in ["uuid", "clip_vector", "sbert_vector"]
]
# Build vector metadata for hard filtering
clip_adapter.metadata = {
    uuid: row
    for uuid, row in zip(df["uuid"], df[metadata_cols].to_dict("records"))
}
print(f"✓ Vectors adapter loaded: {len(dataset['train'])} objects with embeddings and metadata")

### 3. Semantic Search with CLIP

Now you can perform semantic searches:

```python
import torch
import torch.nn.functional as F
import open_clip

# Load CLIP model for encoding queries
clip_model, _, preprocess = open_clip.create_model_and_transforms(
    'ViT-L-14', pretrained='laion2b_s32b_b82k'
)
clip_tokenizer = open_clip.get_tokenizer('ViT-L-14')

floor_filter = {
    "on_floor": {"op": "equal", "value": True},
    "z": {"op": "less_or_equal", "value": 150},  # cm
}

# === Vector Search with Server-Side Filtering ===
query: str = "a red chair"

# Encode text query
with torch.no_grad():
    query_vec = clip_model.encode_text(clip_tokenizer([query]))
    query_vec = F.normalize(query_vec, p=2, dim=-1)

# Top-k similarity search
results = await clip_adapter.search(
    query_vector=query_vec[0].numpy(),
    k=10,                 # Number of results
    filter=floor_filter,  # Hard filtering by metadata
    min_certainty=0.31,   # Confidence threshold
)

print(f"Top results for '{query}':")
for i, result in enumerate(results, 1):
    metadata = result.metadata
    print(f"{i}. Category: {metadata['category']}, Score: {result.score:.3f}")
    print(f"   UUID: {result.id}")
```

## 3. Retrieving Full Asset Metadata

With search results, you can retrieve full metadata for each object:

```python
from huggingface_hub import hf_hub_download
from sisglib.adapters.metadata import JSONLocalAdapter

# Download metadata
metadata_file = hf_hub_download(
    repo_id="yunusskeete/holodeck-objaverse-metadata",
    filename="holodeck_metadata.json",
    repo_type="dataset"
)

# Load adapter
adapter = JSONLocalAdapter(file_path=metadata_file)

# Query by category
chairs = await adapter.query(filter={"category": "chair"}, limit=10)
print(f"Found {len(chairs)} chairs")

# Query by source
objathor_items = await adapter.query(filter={"source": "objathor"}, limit=5)
for item in objathor_items:
    print(f"  - {item.value['category']} (UUID: {item.key})")
```

## Lazy Asset Loading from HuggingFace

For exporting or rendering generated scenes, sisglib's `StorageAdapter` can read directly from HuggingFace datasets and lazy-load assets over HTTP:

```python
from sisglib.adapters import StorageAdapter

# Connect to HuggingFace dataset (no download required!)
async with await StorageAdapter.from_url(
    "https://huggingface.co/datasets/allenai/objaverse/resolve/main/",
    asynchronous=True,
) as adapter:
    # Stream file on-demand
    parquet_bytes: bytes = await adapter.read("glbs/000-000/000074a334c541878360457c672b6c2e.glb")
    print(f"✓ Lazy-loaded {len(parquet_bytes):,} bytes")
    # write to file, etc.
    ...

    # Or stream in chunks
    async for chunk in adapter.stream_read(
        "glbs/000-000/000074a334c541878360457c672b6c2e.glb",
        chunk_size=1024 * 1024  # 1MB chunks
    ):
        # Process chunk without loading entire file
        process_chunk(chunk)
```

### Benefits of Lazy Loading

- **No upfront downloads**: Assets are fetched only when needed
- **Streaming support**: Process large files without loading into memory
- **HTTP serving**: Serve assets directly from HuggingFace
- **Bandwidth efficient**: Only download what you use

## Available Datasets

### Holodeck-Objaverse Embeddings
**Dataset**: `yunusskeete/holodeck-objaverse-embeddings`

Contains:
- **51,463 3D objects** from Objaverse and AI2-THOR
- **CLIP embeddings** (ViT-L-14, 768-dim) for visual similarity
- **SBERT embeddings** (all-mpnet-base-v2, 768-dim) for text similarity
- **Metadata**: category, source, asset metadata
- **ID mapping**: holodeck_id ↔ UUID

Files:
- `vectors.parquet` - All embeddings (CLIP + SBERT)
- `holodeck_metadata.json` - Complete metadata
- `holodeck_id_to_uuid_mapping.json` - ID mapping

### Holodeck-Objaverse Metadata
**Dataset**: `yunusskeete/holodeck-objaverse-metadata`

Lightweight metadata-only version:
- `holodeck_metadata.json` - Complete metadata
- `holodeck_id_to_uuid_mapping.json` - ID mapping
