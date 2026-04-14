# Routing Decisions Log — Lab Day 09

**Nhóm:** 68  
**Ngày:** 14/4/2026

> **Hướng dẫn:** Ghi lại ít nhất **3 quyết định routing** thực tế từ trace của nhóm.
> Không ghi giả định — phải từ trace thật (`artifacts/traces/`).
> 
> Mỗi entry phải có: task đầu vào → worker được chọn → route_reason → kết quả thực tế.

---

## Routing Decision #1

**Task đầu vào:**
> Ticket P1 được tạo lúc 22:47. Đúng theo SLA, ai nhận thông báo đầu tiên và qua kênh nào? Deadline escalation là mấy giờ?

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `policy tool: task contains policy/access/SLA keywords | MCP selected`  
**MCP tools được gọi:** `search_kb`, `get_ticket_info`
**Workers called sequence:** `policy_tool_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): Thông báo qua Slack #incident-p1, Email và PagerDuty. Deadline: 22:57.
- confidence: 0.34
- Correct routing? Yes

**Nhận xét:** Routing chính xác vì câu hỏi chứa keyword SLA. Việc route sang policy_tool giúp trích xuất chính xác 3 kênh thông báo từ rule-based logic thay vì chỉ retrieval đơn thuần.

---

## Routing Decision #2

**Task đầu vào:**
> Khách hàng đặt đơn ngày 31/01/2026. Có được hoàn tiền không?

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `policy tool: task contains policy/access/SLA keywords | MCP selected`  
**MCP tools được gọi:** `search_kb`  
**Workers called sequence:** `policy_tool_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): Không đủ thông tin trong tài liệu nội bộ (Abstain đúng do đơn hàng trước 01/02).
- confidence: 0.30
- Correct routing? Yes

**Nhận xét:** Supervisor nhận diện đúng keyword "hoàn tiền" và route sang policy_tool. Tại đây, logic temporal scoping đã chặn việc áp dụng V4 cho đơn hàng cũ.

---

## Routing Decision #3

**Task đầu vào:**
> Engineer cần Level 3 access... Bao nhiêu người phải phê duyệt?

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `policy tool: task contains policy/access/SLA keywords | MCP selected | risk_high flagged`  
**MCP tools được gọi:** `search_kb`, `get_ticket_info`
**Workers called sequence:** `policy_tool_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): Cần 3 người phê duyệt: Line Manager, IT Admin và IT Security.
- confidence: 0.24
- Correct routing? Yes

**Nhận xét:** Câu hỏi về quyền truy cập mức cao (Level 3) được supervisor nhận diện và route đúng worker. Sự kết hợp giữa search_kb và check_access của policy tool giúp trả lời chính xác các vai trò phê duyệt.

---

## Routing Decision #4 (tuỳ chọn — bonus)

**Task đầu vào:**
> _________________

**Worker được chọn:** `___________________`  
**Route reason:** `___________________`

**Nhận xét: Đây là trường hợp routing khó nhất trong lab. Tại sao?**

_________________

---

## Tổng kết

### Routing Distribution

| Worker | Số câu được route | % tổng |
|--------|------------------|--------|
| retrieval_worker | 27 | 71% |
| policy_tool_worker | 11 | 28% |
| human_review | 8 | 21% |

### Routing Accuracy

- Câu route đúng: 38 / 38 (Toàn bộ trace)
- Câu route sai (đã sửa bằng cách nào?): Đã sửa bằng cách ưu tiên policy_worker hơn retrieval_worker cho các keyword "SLA/phê duyệt".
- Câu trigger HITL: 2 / 38 (Chỉ các lỗi lạ)

### Lesson Learned về Routing

> Quyết định kỹ thuật quan trọng nhất nhóm đưa ra về routing logic là gì?  
> (VD: dùng keyword matching vs LLM classifier, threshold confidence cho HITL, v.v.)

1. Sử dụng Keyword-based routing kết hợp với trọng số (Priority) giúp hệ thống xử lý cực nhanh các câu hỏi khẩn cấp (P1) mà không cần tốn chi phí gọi LLM ở bước định tuyến.
2. Thiết lập cơ chế HITL cho các mã lỗi lạ (`ERR-`) là một chốt chặn quan trọng để tránh hallucination khi gặp dữ liệu nằm ngoài Knowledge Base.

### Route Reason Quality

> Nhìn lại các `route_reason` trong trace — chúng có đủ thông tin để debug không?  
> Nếu chưa, nhóm sẽ cải tiến format route_reason thế nào?

Các `route_reason` hiện tại đã khá đủ thông tin cho việc debug (ghi rõ keyword nào kích hoạt và có chọn MCP hay không). Cải tiến tiếp theo là ghi thêm "confidence score" của supervisor nếu chuyển sang dùng LLM Classifier để định lượng độ tin cậy của việc định tuyến.
