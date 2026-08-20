# Reflection & Action Plan

## Lecture-to-code mapping

| Concept | Implementation |
|---|---|
| Conservative coreference | `run_coref`, `unresolved_mentions` |
| Near dedup | `minhash_lsh_near_dedup`, `near_dedup_pairs` |
| Schema and allowlists | `ALLOWED_NODE_TYPES`, `ALLOWED_RELATIONS` |
| Entity resolution | `build_resolution_map`, FAISS ANN, `merge_guard`, `UF` |
| Bulk ingestion | `bulk_insert_nodes`, `bulk_insert_edges`, `UNWIND` |
| Super-node mitigation | `retrieve_graph_context`, `SUPER_NODE_EDGE_CAP` |
| Evaluation | `run_evaluation`, `comparison_table` |

## Debugging lesson

The main debugging issue was that the HackerNoon export used `description` rather than `text`. A second issue was malformed/multiple JSON returned by the LLM. Both are handled explicitly in the loader and parser, and failed extraction batches are retained for audit.

## Project action plan

For a document-heavy project, begin with Flat RAG as the baseline. Add GraphRAG when questions require entity relationships, temporal transitions or multi-hop evidence. Use typed nodes, provenance-bearing relations, ANN candidate generation and a conservative lexical guard. Apply degree caps and temporal ranking to super-nodes, while retaining an uncapped offline audit path.

## Self-assessment

The implementation prioritizes precision, provenance and recoverability over maximum recall. Before submission, replace qualitative failure descriptions with measured values from the two exported evaluation CSV files and the Neo4j sanity check.
