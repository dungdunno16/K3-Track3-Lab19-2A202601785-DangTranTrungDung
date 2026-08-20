# Suy ngẫm và kế hoạch áp dụng

## Mapping bài giảng vào code

| Khái niệm | Thành phần triển khai |
|---|---|
| Coreference bảo thủ | `run_coref`, `unresolved_mentions` |
| Near dedup | `minhash_lsh_near_dedup`, `near_dedup_pairs` |
| Schema và allowlist | `ALLOWED_NODE_TYPES`, `ALLOWED_RELATIONS` |
| Entity Resolution | `build_resolution_map`, FAISS ANN, `merge_guard`, `UF` |
| Nạp dữ liệu theo batch | `bulk_insert_nodes`, `bulk_insert_edges`, `UNWIND` |
| Giảm thiểu super-node | `retrieve_graph_context`, `SUPER_NODE_EDGE_CAP` |
| Đánh giá | `run_evaluation`, `comparison_table` |

## Bài học debugging

Vấn đề chính là file HackerNoon dùng cột `description` thay vì `text`. Một vấn đề khác là LLM đôi khi trả JSON sai định dạng hoặc nhiều object liên tiếp. Hai trường hợp này đã được xử lý rõ trong loader và parser; các batch extraction lỗi vẫn được giữ lại để audit.

## Kế hoạch áp dụng cho đồ án

Với bài toán nhiều tài liệu, trước tiên nên xây dựng Flat RAG làm baseline. Chỉ thêm GraphRAG khi câu hỏi cần quan hệ giữa entity, diễn biến theo thời gian hoặc suy luận multi-hop. Node nên có type rõ ràng, relation phải có provenance, candidate dùng ANN và lexical guard phải bảo thủ. Super-node cần giới hạn degree và ưu tiên dữ liệu mới, đồng thời giữ một đường audit offline không giới hạn.

## Tự đánh giá

Thiết kế ưu tiên precision, provenance và khả năng khôi phục hơn là recall tối đa. Trước khi nộp, cần thay các mô tả định tính bằng số đo thực tế từ hai file CSV kết quả và truy vấn sanity check trên Neo4j.
