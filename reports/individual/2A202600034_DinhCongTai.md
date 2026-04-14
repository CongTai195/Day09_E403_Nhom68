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

## 2. Tôi đã ra một quyết định kỹ thuật gì?

**Quyết định:** Tích hợp logic xử lý MCP Tool trực tiếp vào lớp Supervisor-Worker thay vì để Worker tự kết nối database.

**Lý do:**
Việc này giúp tách biệt hoàn toàn phần "Business Logic" của Worker (ví dụ: cách kiểm tra policy) và phần "Data Access" (cách lấy ticket từ Jira hay query ChromaDB). Nếu mai này công ty đổi từ ChromaDB sang Pinecone, tôi chỉ cần sửa code ở `mcp_server.py` mà không phải chạm vào bất kỳ Worker nào.

**Lựa chọn thay thế:**
- Hard-code database calls vào Retrieval worker: Nhanh nhưng khó bảo trì và không đúng tinh thần Multi-Agent.
- Dùng REST API truyền thống: Cồng kềnh cho các task xử lý văn bản nội bộ.

**Kết quả:**
Hệ thống đạt được mức độ module hóa rất cao. Bằng chứng là trong trace `run_20260414_154415.json`, field `mcp_tools_used` ghi lại chính xác tham số gọi vào tool `search_kb`, minh chứng cho việc dữ liệu chảy qua MCP server một cách minh bạch.

---

## 3. Tôi đã sửa một lỗi gì?

**Lỗi:** Định dạng dữ liệu trả về từ MCP Server không tương thích với `AgentState`.

**Symptom:**
Khi `mcp_server.py` trả về một list các kết quả, Supervisor không biết cách phân bổ chúng vào field `retrieved_chunks` hay `policy_result`, dẫn đến việc Synthesis worker không nhận được dữ liệu đầu vào.

**Root cause:**
Do thiếu một lớp "Contract" trung gian để chuẩn hóa output của Tool call trước khi ghi vào State.

**Cách sửa:**
Tôi đã xây dựng file `contracts/worker_contracts.yaml` để quy định chặt chẽ I/O cho từng node, sau đó refactor hàm `route_decision` trong `graph.py` để thực hiện việc chuyển đổi kiểu dữ liệu (parsing) ngay sau khi nhận kết quả từ MCP.

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
