### The challenge: **High-Volume Financial Document Ingestion**
SEC 10-K filings are **large, unstructured, and repetitive** (~10–100 MB each), leading to:
- **Storage bloat** (100s of GB for 50+ tickers × 5 years)
- **Vector DB rate limits** (Pinecone, Chroma, Weaviate, OpenAI embeddings)
- **Colab disk exhaustion** (70 GB free tier)

### Solution: **Multi-Layer Optimization Strategy**
| Layer | Technique | Impact |
|------|---------|--------|
| **1. Selective Parsing** | Extract only **Item 1A (Risk Factors)**, **Item 7 (MD&A)** using regex + `sec-edgar-downloader` | **70% reduction** in chunk volume |
| **2. Semantic Chunking** | `RecursiveCharacterTextSplitter` with **800-token chunks + 200 overlap** | Improved retrieval relevance |
| **3. Batch Processing** | **Streaming ingestion** with `ratelimit` + `time.sleep` | Respects **OpenAI RPM/TPM**, avoids 429 errors |
| **4. Embedding Compression** | `text-embedding-3-small` + **int8 quantization** via FAISS PQ | **4x smaller index**, faster inference |
| **5. Transient Storage** | **Parse → Index → Delete raw/JSON** | **95% disk savings** |
| **6. Index Sharding** | Per-year collections (`10k_2023`, `10k_2024`) | Bypasses free-tier vector caps |
| **7. Hybrid Retrieval** | **BM25 + Dense** (rank fusion) | 25% recall boost on long-tail queries |
---

### Architecture
```mermaid
graph TD
    A[SEC EDGAR] --> B[sec-edgar-downloader]
    B --> C[Raw HTML/PDF]
    C --> D[Selective Parser<br/>(Item 1A, 7)]
    D --> E[Semantic Chunking]
    E --> F[Batch Embedding<br/>(text-embedding-3-small)]
    F --> G[FAISS Index<br/>(IVF-PQ, int8)]
    G --> H[Hybrid Search<br/>(BM25 + Dense)]
    H --> I[RAG Query Engine]
    style C fill:#ffcccc
    style G fill:#ccffcc



(1) https://arxiv.org/pdf/2402.05131
(2) Langchain documentation
(3) https://arxiv.org/pdf/2103.00020
