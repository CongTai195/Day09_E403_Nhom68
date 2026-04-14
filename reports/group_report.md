# Báo Cáo Nhóm — Lab Day 09: Multi-Agent Orchestration

**Tên nhóm:** Group 68  
**Thành viên:**
| Tên | Vai trò | Trách nhiệm chính |
|-----|---------|-------|
| Đinh Công Tài | Supervisor & MCP lead | graph.py, mcp_server.py, contracts (Sprint 1&3) |
| Trương Gia Ngọc | Docs Specialist | docs/ architecture, routing, comparison (Docs Owner) |
| Đào Văn Sơn | Reports Specialist | reports/ group & individual reports (Reports Owner) |
| Phạm Minh Quang | Workers Specialist | workers/ retrieval, policy, synthesis (Worker Owner) |
| Nguyễn Trọng Tín | Eval Specialist | eval_trace.py, trace generation, metric analysis (Trace Owner) |

**Ngày nộp:** 14/04/2026  

---

> **Hướng dẫn nộp group report:**
> 
> - File này nộp tại: `reports/group_report.md`
> - Deadline: Được phép commit **sau 18:00** (xem SCORING.md)
> - Tập trung vào **quyết định kỹ thuật cấp nhóm** — không trùng lặp với individual reports
> - Phải có **bằng chứng từ code/trace** — không mô tả chung chung
> - Mỗi mục phải có ít nhất 1 ví dụ cụ thể từ code hoặc trace thực tế của nhóm

---

## 1. Kiến trúc nhóm đã xây dựng (150–200 từ)

> Mô tả ngắn gọn hệ thống nhóm: bao nhiêu workers, routing logic hoạt động thế nào,
> MCP tools nào được tích hợp. Dùng kết quả từ `docs/system_architecture.md`.

**Hệ thống tổng quan:**
Hệ thống sử dụng mô hình Supervisor-Worker với 3 worker chính: Retrieval (truy xuất), Policy Tool (kiểm tra chính sách qua MCP), và Synthesis (tổng hợp). Supervisor điều phối luồng xử lý thông qua keyword-based classification và quản lý state tập trung.

**Routing logic cốt lõi:**
Supervisor sử dụng một hệ thống phân loại dựa trên trọng số từ khóa (keyword weights). Các từ khóa khẩn cấp như "P1", "escalation" được ưu tiên định tuyến thẳng đến Retrieval, trong khi các từ khóa như "refund", "policy", "access" được định tuyến đến Policy Tool. Nếu gặp mã lỗi lạ (`ERR-`), hệ thống tự động trigger HITL.

**MCP tools đã tích hợp:**
- `search_kb`: Công cụ tìm kiếm Knowledge Base tích hợp ChromaDB.
- `get_ticket_info`: Tra cứu thông tin chi tiết ticket (Priority, SLA).
- `check_access_permission`: Kiểm tra quyền hạn và người phê duyệt cho các yêu cầu access level.

Ví dụ trace gọi MCP tool: `run_20260414_154415.json` cho thấy `policy_tool_worker` gọi `search_kb` để kiểm tra ngoại lệ sản phẩm kỹ thuật số.

---

## 2. Quyết định kỹ thuật quan trọng nhất (200–250 từ)

> Chọn **1 quyết định thiết kế** mà nhóm thảo luận và đánh đổi nhiều nhất.
> Phải có: (a) vấn đề gặp phải, (b) các phương án cân nhắc, (c) lý do chọn phương án đã chọn.

**Quyết định:** Sử dụng Shared State (`AgentState`) thay vì truyền message trực tiếp giữa các nodes.

**Bối cảnh vấn đề:**
Trong các phiên bản đầu, việc truyền context giữa retrieval và policy worker rất khó kiểm soát, dẫn đến việc synthesis worker thiếu thông tin về nguyên nhân routing hoặc kết quả từ policy tool.

**Các phương án đã cân nhắc:**
| Phương án | Ưu điểm | Nhược điểm |
|-----------|---------|-----------|
| Message Passing | Stateless, đơn giản | Khó track lịch sử worker gọi |
| Shared State | Dễ audit, persistence | State object có thể trở nên quá lớn |

**Phương án đã chọn và lý do:**
Nhóm chọn Shared State vì yêu cầu của bài lab là phải "traceable". Shared state cho phép mỗi worker ghi lại log của mình vào một `worker_io_logs` duy nhất trong state, giúp việc debug cực kỳ hiệu quả.

**Bằng chứng từ trace/code:**
Trong `graph.py`, hàm `make_initial_state` khởi tạo object chứa mọi field cần thiết, và tất cả worker đều nhận/trả về chính object này (lines 60-80).

---

## 3. Kết quả grading questions (150–200 từ)

> Sau khi chạy pipeline với grading_questions.json (public lúc 17:00):
> - Nhóm đạt bao nhiêu điểm raw?
> - Câu nào pipeline xử lý tốt nhất?
> - Câu nào pipeline fail hoặc gặp khó khăn?

**Tổng điểm raw ước tính:** ___ / 96

**Câu pipeline xử lý tốt nhất:**
- ID: q07 (Sản phẩm kỹ thuật số) — Lý do tốt: Policy tool nhận diện chính xác exception ngoại lệ thông qua MCP call và synthesis worker trích dẫn đúng số Điều trong chính sách.

**Câu gq09 (multi-hop khó nhất):** Trace ghi được 2 workers không? Kết quả thế nào?
Có. Trace `run_20260414_154446.json` (tương ứng q15/gq09) ghi nhận cả `retrieval_worker` và `policy_tool_worker` được gọi theo chuỗi để giải quyết cả yếu tố SLA và Access Control.

---

## 4. So sánh Day 08 vs Day 09 — Điều nhóm quan sát được (150–200 từ)

> Dựa vào `docs/single_vs_multi_comparison.md` — trích kết quả thực tế.

**Metric thay đổi rõ nhất (có số liệu):**
Latency trung bình tăng từ 3.2s (Day 08) lên 7.6s (Day 09). Đây là cái giá phải trả cho việc định tuyến và các worker chạy tuần tự, nhưng đổi lại tính minh bạch của câu trả lời tăng lên đáng kể.

**Điều nhóm bất ngờ nhất khi chuyển từ single sang multi-agent:**
Khả năng "phản biện" của hệ thống. Khi policy tool phát hiện rủi ro, nó có thể chặn pipeline trước khi synthesis worker kịp tạo ra câu trả lời sai (hallucination).

**Trường hợp multi-agent KHÔNG giúp ích hoặc làm chậm hệ thống:**
Với các câu hỏi retrieval cực kỳ đơn giản (như q01), việc đi qua Supervisor và Retrieval Node làm tăng độ trễ (~13s do overhead của graph) mà không đem lại giá trị Accuracy cộng thêm so với vanilla RAG.

---

## 5. Phân công và đánh giá nhóm (100–150 từ)

> Đánh giá trung thực về quá trình làm việc nhóm.

**Phân công thực tế:**

| Kant | Supervisor & MCP | 1 & 3 |
| Nguyen Van A | Docs Folder | 4 |
| Tran Thi B | Reports Folder | 4 |
| Le Van C | Workers Folder | 2 |
| Pham Van D | eval_trace.py | 4 |

**Điều nhóm làm tốt:**
- Tích hợp thành công MCP và quản lý state cực kỳ ổn định.
- Chạy evaluation 15 câu với tỷ lệ thành công 100%.

**Điều nhóm làm chưa tốt hoặc gặp vấn đề về phối hợp:**
- Latency còn cao do chưa tối ưu hóa việc gọi worker song song.

**Nếu làm lại, nhóm sẽ thay đổi gì trong cách tổ chức?**
Nhóm sẽ dành nhiều thời gian hơn cho việc prompt tuning cho supervisor để giảm thiểu việc routing nhầm sang retrieval worker cho các câu hỏi policy.

---

## 6. Nếu có thêm 1 ngày, nhóm sẽ làm gì? (50–100 từ)
Nhóm sẽ nâng cấp Supervisor sang LLM-based Classifier để xử lý context nhạy bén hơn. Ngoài ra, sẽ implement một bảng dashboard preview cho HITL thay vì chỉ print ra console để cải thiện UX của người đánh giá.

---

*File này lưu tại: `reports/group_report.md`*  
*Commit sau 18:00 được phép theo SCORING.md*
