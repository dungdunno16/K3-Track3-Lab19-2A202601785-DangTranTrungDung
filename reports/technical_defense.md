# Technical Defense — Lab 19

## 1. Coreference

Coreference is conservative: only a pronoun or generic reference whose antecedent is explicit in the same chunk is resolved. Ambiguous mentions remain unchanged and are logged in `unresolved_mentions`. This prevents a false coreference from becoming a false graph edge.

## 2. Near dedup

The pipeline uses word 5-shingles with MinHash (64 permutations) and LSH (16 bands × 4 rows). Exact Jaccard `>= 0.82` is required before merging. LSH generates candidates; it does not itself merge records. `near_dedup_pairs` stores both IDs, Jaccard, merge decision, audit label and notes. The longest article is retained as canonical.

## 3. Entity resolution

Manual aliases are applied first. Remaining mentions use FAISS inner-product ANN with cosine threshold `0.90`, followed by a lexical guard: suffixes are removed and normalized names must match or have `SequenceMatcher >= 0.72`. Union-Find produces canonical clusters. The audit records `MERGE_MANUAL`, `MERGE_VECTOR` and `REJECT_GUARD`.

## 4. Neo4j ingestion

Nodes and edges are written with `UNWIND $rows AS row` in batches of 1000. Every edge carries `source_chunk_id`, `published_date`, `evidence` and `confidence`. The required provenance query must return zero invalid edges.

## 5. Retrieval

Flat RAG uses normalized MiniLM embeddings and FAISS `IndexFlatIP`, retrieving top-k chunks. Hybrid GraphRAG extracts seed entities with Groq, resolves exact aliases first and uses embedding fallback at `0.66`. BFS uses two hops, caps super-nodes at 50 newest edges, caps total edges at 250, and caps graph context at 14,000 characters.

## 6. Evaluation

The local detailed golden dataset is loaded from `data/graphrag_golden_50_first5000_detailed.csv`. The lab run evaluates five rows to control Groq usage; the runner checkpoints each row and exports both required CSV files.

## 7. AI-agent control

The agent was not allowed to introduce a global pairwise cosine `O(N²)` deduplication pass. API calls are serialized through the Groq RPM limiter, with bounded retry/backoff and JSON parsing fallback.

## 8. Scale trade-off

GraphRAG adds extraction, canonicalization and graph-ingestion cost, but improves multi-hop and provenance-aware retrieval. Flat RAG is cheaper and often faster for single-hop factoids. At larger scale, batching, ANN indexes, queue-based extraction, checkpointing and incremental graph updates are required.
