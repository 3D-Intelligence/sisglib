# Adapter Architecture Guide

## Overview

sisglib's adapter system provides backend-agnostic interfaces for storage, metadata, and vector operations. This design lets you write code once and swap backends via configuration - from local development to cloud production without refactoring.

```
┌────────────────────────────────────────────────────────┐
│                    sisglib.adapters                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Metadata    │  │   Vectors    │  │   Storage    │  │
│  │  Adapter     │  │   Adapter    │  │   Adapter    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                 │                 │          │
│         │                 │                 │          │
│  ┌──────▼─────────────────▼─────────────────▼───────┐  │
│  │         Unified Retrieval Pipeline               │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
└──────────────────────────┬─────────────────────────────┘
                           │
                ┌──────────▼──────────┐
                │  HuggingFace Hub    │
                │  - Lazy Loading     │
                │  - HTTP Serving     │
                │  - Version Control  │
                └─────────────────────┘
                           │
                ┌──────────▼──────────┐
                │ modular/composable  |
                │ scene building with │
                │ sisglib.strategies  │
                │ & sisglib.pipelines │
                └─────────────────────┘
```


## The Three Adapter Types

### Storage Adapter

Manages file operations (3D assets, scenes, etc.) across different storage backends.

**Supported Backends:**
- Local filesystem
- Amazon S3
- Azure Blob Storage
- Google Cloud Storage
- HTTP/HTTPS (including HuggingFace Datasets)
- FTP/SFTP
- In-memory storage

**Common Operations:**

```python
from sisglib.adapters.storage import StorageAdapter

# Read files
content = await adapter.read("path/to/file.glb")

# Write files
await adapter.write("path/to/file.glb", content)

# List files
files = await adapter.list("path/to/directory/")

# Check existence
exists = await adapter.exists("path/to/file.glb")

# Delete files
await adapter.delete("path/to/file.glb")
```

### Metadata Adapter

Manages JSON-like document storage for asset metadata, scene descriptions, and annotations.

**Supported Backends:**
- JSON (in-memory and file-based)
- MongoDB
- Amazon DynamoDB
- Azure CosmosDB

**Common Operations:**

```python
from sisglib.adapters.metadata import MetadataAdapter

# Insert metadata
await metadata.insert({"asset_id": "123", "category": "furniture", "tags": ["modern"]})

# Query with filters
results = await metadata.query(
    filter={"category": "furniture"},
    limit=10
)

# Get by ID
item = await metadata.get("asset_id_123")

# Update
await metadata.update("asset_id_123", {"tags": ["modern", "wooden"]})

# Delete
await metadata.delete("asset_id_123")

# Count
total = await metadata.count(filter={"category": "furniture"})
```

### Vector Adapter

Manages high-dimensional vector embeddings for semantic similarity search.

**Supported Backends:**
- Numpy (in-memory)
- ChromaDB
- Pinecone
- Weaviate
- Faiss
- Qdrant
- Milvus
- PostgreSQL + pgvector

**Common Operations:**

```python
from sisglib.adapters.vectors import VectorsAdapter

# Insert vectors
await vectors.insert(
    id="asset_123",
    vector=embedding_array,
    metadata={"category": "furniture"}
)

# Semantic search
results = await vectors.search(
    query_vector=query_embedding,
    k=10,
    metadata_filter={"category": "furniture"},
    min_certainty=0.7
)

# Batch operations
await vectors.insert_batch([
    {"id": "1", "vector": vec1, "metadata": meta1},
    {"id": "2", "vector": vec2, "metadata": meta2},
])
```

## Configuration Patterns

### URL-Based Configuration (Storage)

The simplest way to configure storage adapters:

```python
# Local filesystem
adapter = await StorageAdapter.from_url("file:///local/assets")

# Amazon S3
adapter = await StorageAdapter.from_url("s3://bucket-name/prefix")

# Azure Blob Storage
adapter = await StorageAdapter.from_url("az://container-name/prefix")

# Google Cloud Storage
adapter = await StorageAdapter.from_url("gs://bucket-name/prefix")

# HuggingFace Datasets
adapter = await StorageAdapter.from_url(
    "https://huggingface.co/datasets/org/dataset/tree/main"
)

# Environment variable
import os
adapter = await StorageAdapter.from_url(os.getenv("STORAGE_URL"))
```

### Dictionary-Based Configuration

For more control and non-storage adapters:

```python
# Storage with detailed config
adapter = await StorageAdapter.from_config({
    "backend": "s3",
    "config": {
        "bucket": "my-bucket",
        "prefix": "scenes/",
        "region": "us-west-2",
    }
})

# Metadata adapter
metadata = await MetadataAdapter.from_config({
    "backend": "mongodb",
    "config": {
        "connection_string": "mongodb://localhost:27017",
        "database": "sisglib",
        "collection": "assets"
    }
})

# Vector adapter
vectors = await VectorsAdapter.from_config({
    "backend": "pinecone",
    "config": {
        "api_key": os.getenv("PINECONE_API_KEY"),
        "environment": "us-west1-gcp",
        "index_name": "asset-embeddings",
        "dimension": 768
    }
})
```

## Resource Management

Adapters support both manual and automatic resource management:

### Automatic (Recommended)

```python
# Async context manager handles cleanup automatically
async with await StorageAdapter.from_url("s3://bucket/prefix") as adapter:
    content = await adapter.read("file.txt")
    # Resources cleaned up automatically when exiting context
```

### Manual

```python
# Manual resource management
adapter = await StorageAdapter.from_url("s3://bucket/prefix")
try:
    content = await adapter.read("file.txt")
finally:
    await adapter.close()  # Must close explicitly
```

## Backend Combinations

One of sisglib's key features is backend independence. You can mix and match:

**Development:**
```python
storage = await StorageAdapter.from_url("file:///local/assets")
vectors = NumpyAdapter(dimension=768)
metadata = JSONLocalAdapter(file_path="metadata.json")
```

**Staging:**
```python
storage = await StorageAdapter.from_url("https://huggingface.co/datasets/...")
vectors = ChromaDBAdapter(path="./chromadb")
metadata = JSONLocalAdapter(file_path="metadata.json")
```

**Production:**
```python
storage = await StorageAdapter.from_url("s3://prod-bucket/assets")
vectors = PineconeAdapter(api_key=os.getenv("PINECONE_KEY"))
metadata = MongoDBAdapter(connection_string=os.getenv("MONGODB_URI"))
```

This gives you **320 possible backend combinations!** (8 storage types × 8 vector types × 5 metadata type, etc.) without changing your application code.
