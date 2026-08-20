# Thuyết minh kỹ thuật — Lab 19

## 1. Phân giải đồng tham chiếu

Cơ chế coreference chỉ thay thế đại từ hoặc tham chiếu chung khi tiền ngữ xuất hiện rõ ràng trong cùng một chunk. Trường hợp mơ hồ được giữ nguyên và ghi vào `unresolved_mentions`. Cách làm này ngăn việc phân giải sai tạo thành cạnh sai trong đồ thị.

## 2. Near Dedup

Pipeline dùng word 5-shingles, MinHash với 64 phép băm và LSH gồm 16 bands × 4 rows. Chỉ merge khi Jaccard chính xác `>= 0.82`. LSH chỉ sinh cặp ứng viên, không tự quyết định merge. Bảng `near_dedup_pairs` lưu hai ID, điểm Jaccard, quyết định merge, nhãn audit và ghi chú. Bài viết dài nhất trong cluster được giữ làm bản canonical.

## 3. Entity Resolution

Các alias thủ công được áp dụng trước. Những mention còn lại được tìm ứng viên bằng FAISS inner-product ANN với ngưỡng cosine `0.90`, sau đó qua lexical guard: loại bỏ hậu tố và yêu cầu tên chuẩn hóa giống nhau hoặc có `SequenceMatcher >= 0.72`. Union-Find tạo các cluster canonical. Bảng audit ghi các quyết định `MERGE_MANUAL`, `MERGE_VECTOR` và `REJECT_GUARD`.

## 4. Nạp dữ liệu vào Neo4j

Node và edge được ghi bằng `UNWIND $rows AS row`, theo batch 1000 bản ghi. Mỗi edge có `source_chunk_id`, `published_date`, `evidence` và `confidence`. Truy vấn kiểm tra provenance bắt buộc phải trả về 0 cạnh không hợp lệ.

## 5. Kiến trúc truy hồi

Flat RAG dùng embedding MiniLM đã chuẩn hóa và FAISS `IndexFlatIP` để lấy top-k chunks. Hybrid GraphRAG dùng Groq trích xuất seed entity, ưu tiên khớp chính xác/alias và dùng embedding fallback với ngưỡng `0.66`. BFS tối đa hai hop, giới hạn super-node còn 50 cạnh mới nhất, giới hạn tổng 250 cạnh và giới hạn context đồ thị 14.000 ký tự.

## 6. Đánh giá

Bộ golden local được đọc từ `data/graphrag_golden_50_first5000_detailed.csv`. Lần chạy lab đánh giá 5 câu để kiểm soát quota Groq; runner checkpoint sau từng câu và xuất đủ hai file CSV bắt buộc.

## 7. Kiểm soát AI Coding Agent

Không sử dụng phép so sánh cosine pairwise `O(N²)` trên toàn bộ dữ liệu. Các request Groq được tuần tự hóa qua RPM limiter, có retry/backoff giới hạn và parser JSON dự phòng.

## 8. Đánh đổi khi scale

GraphRAG tốn thêm chi phí trích xuất, canonicalization và nạp đồ thị nhưng hỗ trợ tốt hơn cho câu hỏi multi-hop và provenance. Flat RAG rẻ và thường nhanh hơn với factoid single-hop. Khi scale, cần batching, ANN index, hàng đợi extraction, checkpoint và cập nhật đồ thị tăng dần.
