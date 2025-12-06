# Semantic Asset Intelligence

sisglib's vector adapters enable semantic search over 3D asset libraries using natural language or visual queries. Each adapter implements similarity search functionality for **cross-modal retrieval** - allowing you to find 3D assets using text descriptions, images, or other modalities without exact keyword matching.

## Overview

Traditional asset retrieval relies on manual tagging and keyword search. Semantic asset intelligence uses learned embeddings to:

- **Find assets by description**: Query "modern sofa" to retrieve relevant furniture without exact tags
- **Cross-modal search**: Use text queries to find 3D models, or image queries to find similar assets
- **Fuzzy matching**: Discover visually or semantically similar assets even without identical metadata

All vector adapters share a unified interface, so you can swap backends (numpy, Pinecone, Weaviate, etc.) without changing your research code.

## Quick Example

```python
from sisglib.adapters.vectors import WeaviateAdapter

# Initialize vector store (or use numpy, Pinecone, ChromaDB, Faiss...)
vectors = WeaviateAdapter(
    http_url="http://localhost:8080",
    class_name="AssetEmbeddings",
    dimension=768
)

# Text-guided 3D asset retrieval
results = await vectors.search(
    query_vector=clip_text_embed("modern sofa"),
    k=10,
    metadata_filter={"category": "furniture"}
)
```

## Supported Embeddings

Use any embedding model that produces vector representations:

- **Multi-modal**: CLIP, BLIP, BLIP-2, OpenShape
- **3D-specific**: ULIP, PointCLIP

## Supported Vector Backends

- **Development**: Numpy (in-memory), Faiss
- **Research**: ChromaDB, Qdrant
- **Production**: Pinecone, Weaviate, Milvus, PostgreSQL+pgvector

Swap backends by changing configuration - your research code stays the same.

## Learn More

- [Adapter Architecture Guide](../adapter-architecture.md) - Understanding the adapter system
- [Vector Adapters API Reference](../../../../sisglib/adapters/vectors/base.py) - Base interface and implementations
