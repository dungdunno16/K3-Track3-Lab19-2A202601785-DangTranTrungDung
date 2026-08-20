# Phân tích ca lỗi — Lab 19

## Các nhóm lỗi cần theo dõi

### Lỗi trích xuất JSON

Groq có thể trả nhiều JSON liên tiếp, markdown hoặc dấu câu sai. Wrapper trước tiên thử JSON mode, sau đó retry không bật JSON mode và dùng `JSONDecoder.raw_decode()` để lấy object hợp lệ đầu tiên. Các batch lỗi vẫn được lưu trong `extraction_errors_df`, không biến thành fact ngầm.

### Phân giải đồng tham chiếu sai

Tham chiếu như “the company” có thể trỏ nhầm về thực thể trước đó. Chính sách an toàn là giữ nguyên mention gốc và ghi vào `unresolved_mentions`; extractor không được tự bịa thực thể thay thế.

### Merge thực thể sai

Chỉ dùng embedding có thể gộp nhầm công ty với sản phẩm hoặc hai người có tên gần giống. Lexical guard, phân biệt type, alias thủ công và bảng audit giúp giảm rủi ro. Các cặp có similarity cao nhưng bị từ chối cần được kiểm tra trong `entity_resolution_audit_df`.

### Flat RAG và GraphRAG

Flat RAG có thể bỏ sót câu trả lời multi-hop khi bằng chứng nằm ở nhiều chunk khác nhau. GraphRAG có thể thất bại nếu seed extraction bỏ sót entity, extraction thiếu edge hoặc super-node cap loại mất cạnh lịch sử. File evaluation ghi lại cả hai câu trả lời, lý do của judge và số lần kích hoạt super-node để phân tích nhóm kết quả thấp nhất.

## Bằng chứng cần bổ sung sau khi chạy

- Dùng `extraction_errors_df` để thống kê các batch extraction lỗi.
- Dùng `entity_resolution_audit_df` để trích dẫn ít nhất một cặp `REJECT_GUARD`.
- Dùng `top_degree_df` và `graph_debug.diagnostics` để phân tích super-node.
- Dùng `outputs/graphrag_eval_results.csv` để chọn một ca Flat RAG thất bại và một ca GraphRAG thất bại.
