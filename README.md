# RAG Pipeline for Financial Document Q&A

A retrieval-augmented generation system built from scratch for querying Apple's 10-K SEC filings. Built without LangChain or LlamaIndex to understand the retrieval stack at a fundamental level.

## Architecture

```
SEC 10-K Filing
      ↓
Sentence-aware chunking (~1000 chars)
      ↓
OpenAI Embeddings (text-embedding-3-small, 1536 dims)
      ↓
FAISS Vector Index (L2 distance)
      ↓
Top-k Retrieval (k=10 candidates)
      ↓
Cohere Reranker (cross-encoder, top 3)
      ↓
GPT-4o-mini Generation
      ↓
Answer
```

## Stack

- **Chunking** — sentence-aware splitting that respects sentence boundaries rather than cutting on fixed character counts
- **Embeddings** — OpenAI `text-embedding-3-small` (1536 dimensions). Vectors are unit-normalized, making L2 distance mathematically equivalent to cosine similarity
- **Vector Index** — FAISS `IndexFlatL2` for nearest neighbor search. In-memory and fast for single-document retrieval
- **Reranker** — Cohere `rerank-english-v3.0`, a cross-encoder that reads query and chunk together rather than comparing them as separate vectors. Improves precision on multi-part and causally specific queries
- **Generation** — GPT-4o-mini with retrieved context injected into the prompt

## Why Reranking

FAISS retrieves by geometric proximity in embedding space — it finds chunks that are topically related to the query. The Cohere reranker applies reading comprehension on top of that: it scores whether each candidate chunk actually answers the specific question asked.

For a single structured document like a 10-K, the difference between the two pipelines is small — SEC filings have low semantic overlap between sections and formal consistent language that embeds cleanly. The reranker shows meaningful improvement in multi-document settings where competing chunks from different contexts create retrieval ambiguity that vector similarity alone cannot resolve.

Both pipelines run side by side in the notebook so the tradeoff is visible directly.

## Key Design Decisions

| Decision | Rationale |
|---|---|
| Sentence-aware chunking over fixed-char splitting | Avoids cutting mid-sentence which destroys meaning and degrades embedding quality |
| L2 over cosine | OpenAI embeddings are unit-normalized so L2 and cosine are mathematically equivalent — L2 is faster |
| k=10 retrieval before reranking | Wide candidate pool gives the reranker enough to work with; FAISS at k=10 is still fast |
| GPT-4o-mini over GPT-4 | Retrieval quality matters more than model size — clean context with a smaller model outperforms noisy context with a larger one |
| No LangChain or LlamaIndex | Building from primitives gives full visibility into where retrieval succeeds and fails |

## What I Would Improve

- **Semantic chunking** — split on topic boundaries detected by embedding similarity between adjacent sentences rather than character count
- **Chunk overlap** — 1-2 sentence overlap between chunks to avoid missing answers that span a boundary
- **Metadata filtering** — tag chunks by SEC section (Item 7, Item 8) to enable targeted retrieval before vector search
- **Multi-document corpus** — extend to multiple company 10-Ks across years where reranking shows meaningful improvement on comparative and temporal queries
- **Eval framework** — systematic measurement of retrieval quality using ground truth Q&A pairs rather than qualitative inspection

## Credentials

API keys are loaded from Google Colab secrets (never hardcoded). To run locally, set `OPENAI_API_KEY` and `COHERE_API_KEY` as environment variables.
