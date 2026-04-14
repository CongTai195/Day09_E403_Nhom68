# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Đinh Công Tài
**Vai trò trong nhóm:** Supervisor & MCP lead
**Ngày nộp:** 2026-04-14

---

## 1. Tôi phụ trách phần nào?

Tôi chịu trách nhiệm về "Xương sống" và "Cổng giao tiếp" của hệ thống — tức là Supervisor Orchestrator và MCP Server.

**Module/file tôi chịu trách nhiệm:**
- File chính: `graph.py`, `mcp_server.py`
- Functions tôi implement: `supervisor_node`, `route_decision`, `make_initial_state` (trong graph.py) và toàn bộ bộ công cụ `search_kb`, `get_ticket_info` (trong mcp_server.py).

Công việc của tôi đảm bảo hệ thống có khả năng định tuyến thông minh và kết nối được với các nguồn dữ liệu ngoại vi một cách tiêu chuẩn hóa qua Model Context Protocol.

---

**Quyết định:** Triển khai cơ chế **Priority-based Keyword Weighting** kết hợp cùng MCP Tool logic để xử lý các task khẩn cấp.

**Lý do:**
Ban đầu, supervisor của tôi thường xuyên định tuyến vào `retrieval_worker` cho các câu hỏi về SLA vì chúng chứa keyword "ticket". Tuy nhiên, điều này dẫn đến câu trả lời quá ngắn gọn. Tôi quyết định tăng trọng số cho các keyword như "SLA", "access" và "phê duyệt" để ưu tiên định tuyến sang `policy_tool_worker`. Điều này đảm bảo các câu hỏi phức tạp luôn được xử lý bởi logic rule-based mạnh mẽ thay vì chỉ truy xuất văn bản đơn thuần.

**Lựa chọn thay thế:**
- Dùng LLM Classifier: Thông minh hơn nhưng làm tăng latency lên gấp đôi (từ 7s lên 14s).
- Hard-code câu hỏi: Không mở rộng được cho các câu hỏi mới.

**Kết quả:**
Các câu hỏi khó như `gq01` (SLA P1) và `gq09` (Emergency Access) hiện đã được route chính xác 100% sang policy worker. Bằng chứng là trong trace mới nhất `run_20260414_171432.json`, supervisor đã chọn đúng route với lý do: "policy tool: task contains policy/access/SLA keywords".

---

**Lỗi:** Mismatch dimension khi gọi OpenAI Embedding `text-embedding-3-small`.

**Symptom:**
Khi chạy script evaluation, hệ thống liên tục crash ở bước retrieval với lỗi: `Invalid dimension, expected 384, got 1536`.

**Root cause:**
Model `text-embedding-3-small` mặc định trả về 1536 dimensions, trong khi index ChromaDB của nhóm (được tạo trước đó) chỉ hỗ trợ 384 dimensions.

**Cách sửa:**
Tôi đã cập nhật hàm `_get_embedding_fn` trong `retrieval.py` để ép tham số `dimensions=384`. Ngoài ra, tôi cấu hình lại `eval_trace.py` để chạy trong môi trường `venv` chuẩn, đảm bảo trùng khớp phiên bản và cấu hình vector store. Kết quả là hệ thống đã retrieve mượt mà cho toàn bộ 10 câu grading.

---

## 4. Tôi tự đánh giá đóng góp của mình

**Tôi làm tốt nhất ở điểm nào?**
Thiết kế được một khung Graph cực kỳ linh hoạt và hệ thống MCP chuẩn hóa, giúp các thành viên khác chỉ cần tập trung vào logic xử lý mà không cần lo lắng về việc kết nối dữ liệu.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Logic routing hiện tại vẫn đang dùng keyword-matching, nếu user đặt câu hỏi quá phức tạp hoặc "bẫy" ngữ nghĩa, supervisor có thể định tuyến sai.

**Nhóm phụ thuộc vào tôi ở đâu?**
Nếu graph của tôi không chạy hoặc MCP server bị lỗi, toàn bộ các Worker của thành viên khác sẽ không có dữ liệu đầu vào để xử lý.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì?

Tôi sẽ nâng cấp MCP Server từ mock class sang **HTTP Server thật sự** dùng thư viện `mcp` của Anthropic để đạt điểm bonus +2 theo đúng rubric của SCORING.md.
