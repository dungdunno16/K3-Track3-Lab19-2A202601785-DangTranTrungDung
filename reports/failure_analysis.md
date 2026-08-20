# Failure Analysis — Lab 19

## Observed failure classes

### JSON extraction failure

Groq may return multiple JSON objects, markdown or malformed punctuation. The wrapper now tries JSON mode, retries without JSON mode, and uses `JSONDecoder.raw_decode()` to recover the first valid object. Failed batches remain in `extraction_errors_df` instead of silently becoming facts.

### False coreference

An ambiguous “the company” can resolve to the wrong antecedent. The safe policy is to keep the original mention and record it in `unresolved_mentions`; the downstream extractor must not invent a replacement entity.

### False entity merge

Embedding similarity alone can merge names such as a company and its product, or two people with similar names. The lexical guard, type separation, manual alias map and audit table reduce this risk. Rejected high-similarity pairs should be reviewed from `entity_resolution_audit_df`.

### Flat vs GraphRAG

Flat RAG can miss a multi-hop answer when the supporting facts are in separate chunks. GraphRAG can fail when seed extraction misses an entity, extraction omits an edge, or the super-node cap removes a historically relevant edge. The evaluation CSV records both answers, judge rationales and super-node events for bottom-N analysis.

## Required evidence after execution

- Use `extraction_errors_df` to report failed extraction batches.
- Use `entity_resolution_audit_df` to report at least one `REJECT_GUARD` pair.
- Use `top_degree_df` and `graph_debug.diagnostics` to report super-node behavior.
- Use `outputs/graphrag_eval_results.csv` to select one Flat-RAG failure and one GraphRAG failure.
