# Backend Configuration Guide

## Overview

This guide provides detailed configuration instructions for all supported backends in sisglib. Use this as a reference when setting up adapters for different environments.

## Storage Backends

### Local Filesystem

**Use case:** Development, testing, small datasets

```python
from sisglib.adapters.storage import StorageAdapter

# URL-based
adapter = await StorageAdapter.from_url("file:///absolute/path/to/assets")

# Config-based
adapter = await StorageAdapter.from_config({
    "backend": "file",
    "config": {
        "base_path": "/absolute/path/to/assets"
    }
})
```

**Notes:**
- Paths must be absolute
- Fast for local development
- No network latency

---

### Amazon S3

**Use case:** Production, cloud deployments, large datasets

```python
# URL-based (simplest)
adapter = await StorageAdapter.from_url("s3://bucket-name/optional-prefix")

# Config-based (more control)
adapter = await StorageAdapter.from_config({
    "backend": "s3",
    "config": {
        "bucket": "my-bucket",
        "prefix": "assets/",
        "region": "us-west-2",
        "endpoint_url": None,  # Custom endpoint for S3-compatible services
        "aws_access_key_id": None,  # Uses AWS credentials chain if None
        "aws_secret_access_key": None,
    }
})
```

**Environment variables:**
```bash
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-west-2
```

**Notes:**
- Credentials from environment, AWS config files, or IAM roles
- Enable versioning on bucket for safety
- Consider S3 lifecycle policies for cost optimization

---

### Azure Blob Storage

**Use case:** Production on Azure, cloud deployments

```python
# URL-based
adapter = await StorageAdapter.from_url("az://container-name/optional-prefix")

# Config-based
adapter = await StorageAdapter.from_config({
    "backend": "azure",
    "config": {
        "container": "my-container",
        "prefix": "assets/",
        "connection_string": None,  # From environment if None
        "account_name": None,
        "account_key": None,
    }
})
```

**Environment variables:**
```bash
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=..."
# OR
export AZURE_STORAGE_ACCOUNT_NAME=your_account
export AZURE_STORAGE_ACCOUNT_KEY=your_key
```

---

### Google Cloud Storage

**Use case:** Production on GCP, cloud deployments

```python
# URL-based
adapter = await StorageAdapter.from_url("gs://bucket-name/optional-prefix")

# Config-based
adapter = await StorageAdapter.from_config({
    "backend": "gcs",
    "config": {
        "bucket": "my-bucket",
        "prefix": "assets/",
        "project": None,  # From environment if None
        "credentials_path": None,  # Path to service account JSON
    }
})
```

**Environment variables:**
```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
export GCP_PROJECT=your-project-id
```

---

### HTTP/HTTPS (HuggingFace)

**Use case:** Streaming datasets, public assets, no-download workflows

```python
# HuggingFace Datasets
adapter = await StorageAdapter.from_url(
    "https://huggingface.co/datasets/org/dataset/resolve/main/",
    asynchronous=True
)

# Generic HTTP
adapter = await StorageAdapter.from_url("https://example.com/assets/")
```

**Notes:**
- Read-only
- No authentication for public URLs
- Ideal for lazy-loading datasets
- See [HuggingFace Integration Guide](adapters/storage/huggingface-integration.md) for details

---

### FTP/SFTP

**Use case:** Legacy systems, university servers

```python
# FTP
adapter = await StorageAdapter.from_url("ftp://user:password@host:21/path")

# SFTP
adapter = await StorageAdapter.from_url("sftp://user:password@host:22/path")
```

---

## Vector Backends

### Numpy (In-Memory)

**Use case:** Development, small datasets, testing

```python
from sisglib.adapters.vectors import NumpyAdapter

adapter = NumpyAdapter(dimension=768)

# Manually populate
adapter.vectors = {
    "id1": embedding1,
    "id2": embedding2,
}
adapter.metadata = {
    "id1": {"category": "furniture"},
    "id2": {"category": "lighting"},
}
```

**Notes:**
- Fast for small datasets (<10k vectors)
- No persistence (data lost when process ends)
- Great for unit tests

---

### Faiss

**Use case:** Large datasets, research, local development

```python
from sisglib.adapters.vectors import FaissAdapter

# Config-based
adapter = await VectorsAdapter.from_config({
    "backend": "faiss",
    "config": {
        "dimension": 768,
        "index_type": "Flat",  # or "IVF", "HNSW"
        "metric": "cosine",
        "persist_path": "./faiss_index"  # Optional persistence
    }
})
```

**Notes:**
- Excellent performance for millions of vectors
- Multiple index types (Flat, IVF, HNSW)
- CPU and GPU support
- Local or distributed

---

### Pinecone

**Use case:** Production, managed service, minimal ops

```python
from sisglib.adapters.vectors import PineconeAdapter

adapter = PineconeAdapter(
    api_key=os.getenv("PINECONE_API_KEY"),
    environment="us-west1-gcp",
    index_name="asset-embeddings",
    dimension=768,
)
```

**Environment variables:**
```bash
export PINECONE_API_KEY=your_api_key
```

**Notes:**
- Fully managed
- Auto-scaling
- Requires API key
- Pay per usage

---

### Weaviate

**Use case:** Production, self-hosted or cloud, rich metadata

```python
from sisglib.adapters.vectors import WeaviateAdapter

adapter = WeaviateAdapter(
    http_url="http://localhost:8080",  # Or Weaviate Cloud URL
    class_name="AssetEmbeddings",
    dimension=768,
    create_if_not_exists=True,
)
```

**Docker setup:**
```bash
docker run -d \
  -p 8080:8080 \
  -e AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=true \
  semitechnologies/weaviate:latest
```

**Notes:**
- Open source
- Rich filtering capabilities
- Can self-host or use Weaviate Cloud
- GraphQL API

---

### ChromaDB

**Use case:** Development, research, local deployments

```python
from sisglib.adapters.vectors import ChromaDBAdapter

adapter = ChromaDBAdapter(
    path="./chromadb",  # Persistent storage
    collection_name="assets",
    dimension=768,
)
```

**Notes:**
- Embedded database (no server needed)
- Persistent local storage
- Easy setup
- Good for research workflows

---

### Qdrant

**Use case:** Production, self-hosted or cloud

```python
from sisglib.adapters.vectors import QdrantAdapter

adapter = QdrantAdapter(
    host="localhost",
    port=6333,
    collection_name="assets",
    dimension=768,
    api_key=None,  # For Qdrant Cloud
)
```

**Docker setup:**
```bash
docker run -p 6333:6333 qdrant/qdrant
```

---

### Milvus

**Use case:** Large-scale production, distributed systems

```python
from sisglib.adapters.vectors import MilvusAdapter

adapter = MilvusAdapter(
    host="localhost",
    port="19530",
    collection_name="assets",
    dimension=768,
)
```

**Notes:**
- Highly scalable
- Distributed architecture
- Good for very large datasets (>100M vectors)

---

### PostgreSQL + pgvector

**Use case:** When you already have PostgreSQL, unified storage

```python
from sisglib.adapters.vectors import PostgreSQLAdapter

adapter = PostgreSQLAdapter(
    connection_string="postgresql://user:password@localhost:5432/dbname",
    table_name="asset_embeddings",
    dimension=768,
)
```

**Setup:**
```sql
CREATE EXTENSION vector;

CREATE TABLE asset_embeddings (
    id TEXT PRIMARY KEY,
    embedding vector(768),
    metadata JSONB
);
```

**Notes:**
- Leverages existing PostgreSQL infrastructure
- Good for moderate scale
- Native JSON support for metadata

---

## Metadata Backends

### JSON (In-Memory)

**Use case:** Development, testing, small datasets

```python
from sisglib.adapters.metadata import JSONLocalAdapter

# In-memory
adapter = JSONLocalAdapter()

# File-based (persistent)
adapter = JSONLocalAdapter(file_path="metadata.json")
```

**Notes:**
- Simple and fast
- File-based persistence optional
- No query optimization

---

### MongoDB

**Use case:** Production, flexible schemas, rich queries

```python
from sisglib.adapters.metadata import MongoDBAdapter

adapter = MongoDBAdapter(
    connection_string="mongodb://localhost:27017",
    database="sisglib",
    collection="assets",
)
```

**Docker setup:**
```bash
docker run -d -p 27017:27017 mongo:latest
```

**Environment variables:**
```bash
export MONGODB_URI="mongodb://localhost:27017"
```

**Notes:**
- Flexible schema
- Powerful query language
- Good indexing support
- Self-hosted or MongoDB Atlas

---

### Amazon DynamoDB

**Use case:** Production on AWS, serverless, high scale

```python
from sisglib.adapters.metadata import DynamoDBAdapter

adapter = DynamoDBAdapter(
    table_name="sisglib-assets",
    region="us-west-2",
    aws_access_key_id=None,  # Uses AWS credentials chain
    aws_secret_access_key=None,
)
```

**Notes:**
- Fully managed
- Auto-scaling
- Pay per request
- Requires AWS credentials

---

### Azure CosmosDB

**Use case:** Production on Azure, global distribution

```python
from sisglib.adapters.metadata import CosmosDBAdapter

adapter = CosmosDBAdapter(
    endpoint="https://your-account.documents.azure.com:443/",
    key=os.getenv("COSMOS_KEY"),
    database="sisglib",
    container="assets",
)
```

**Notes:**
- Globally distributed
- Multiple API compatibility (MongoDB, SQL, etc.)
- Auto-scaling

---

## Environment-Based Configuration

A recommended pattern for managing configurations across environments:

```python
# config.py
import os
from enum import Enum

class Environment(Enum):
    DEVELOPMENT = "dev"
    STAGING = "staging"
    PRODUCTION = "prod"

def get_storage_config(env: Environment) -> dict:
    configs = {
        Environment.DEVELOPMENT: {
            "url": "file:///local/assets"
        },
        Environment.STAGING: {
            "url": "https://huggingface.co/datasets/org/staging-assets"
        },
        Environment.PRODUCTION: {
            "url": os.getenv("STORAGE_URL", "s3://prod-bucket/assets")
        }
    }
    return configs[env]

def get_vector_config(env: Environment) -> dict:
    configs = {
        Environment.DEVELOPMENT: {
            "backend": "numpy",
            "config": {"dimension": 768}
        },
        Environment.STAGING: {
            "backend": "chromadb",
            "config": {"path": "./chromadb", "dimension": 768}
        },
        Environment.PRODUCTION: {
            "backend": "pinecone",
            "config": {
                "api_key": os.getenv("PINECONE_API_KEY"),
                "environment": "us-west1-gcp",
                "dimension": 768
            }
        }
    }
    return configs[env]

# Usage
env = Environment(os.getenv("ENV", "dev"))
storage = await StorageAdapter.from_url(**get_storage_config(env))
vectors = await VectorsAdapter.from_config(get_vector_config(env))
```

## Cost Optimization Tips

### Storage
- Use S3 Intelligent-Tiering for automatic cost optimization
- Implement lifecycle policies to move old data to cheaper storage classes
- Use CloudFront or similar CDN for frequently accessed assets

### Vector Databases
- Use Faiss locally during development to avoid cloud costs
- In production, choose based on scale:
  - Small (<100k vectors): ChromaDB, Qdrant
  - Medium (100k-10M): Pinecone, Weaviate
  - Large (>10M): Milvus, custom Faiss cluster

### Metadata
- MongoDB Atlas free tier is sufficient for small projects
- DynamoDB's on-demand pricing is good for unpredictable workloads
- Use PostgreSQL if you already have it running

## Security Best Practices

1. **Never hardcode credentials**
   ```python
   # Bad
   api_key = "pk_abc123..."

   # Good
   api_key = os.getenv("API_KEY")
   ```

2. **Use IAM roles when possible** (AWS, Azure, GCP)

3. **Enable encryption at rest** for cloud storage

4. **Use connection strings from environment variables**
   ```bash
   export MONGODB_URI="mongodb://..."
   export PINECONE_API_KEY="..."
   export AWS_ACCESS_KEY_ID="..."
   ```

5. **Rotate credentials regularly**

6. **Use least-privilege access**
   - Storage: read-only for most operations
   - Vector DB: limit to specific indexes/collections
   - Metadata: scoped to specific databases

## Troubleshooting

### Connection Issues

```python
from sisglib.adapters.exceptions import ConnectionError

try:
    adapter = await StorageAdapter.from_url("s3://bucket/prefix")
except ConnectionError as e:
    print(f"Failed to connect: {e}")
    # Check credentials, network, endpoint URL
```

### Performance Issues

- Enable connection pooling for database adapters
- Use batch operations when available
- Consider caching for frequently accessed data
- Profile queries to identify bottlenecks

### Authentication Issues

- Verify environment variables are set
- Check credential file permissions
- Ensure IAM roles/policies are correct
- Test with minimal examples first

## Next Steps

- [Adapter Architecture Guide](adapters/adapter-architecture.md) - Understanding the adapter system
- [Custom Strategies Guide](pipelines/custom-strategies.md) - Using adapters in scene generation
- [HuggingFace Integration](adapters/storage/huggingface-integration.md) - Streaming datasets
