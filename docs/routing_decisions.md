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
> SLA xử lý ticket P1 là bao lâu?

**Worker được chọn:** `retrieval_worker`  
**Route reason (từ trace):** `priority retrieval: task contains P1/escalation/ticket keywords | No MCP`  
**MCP tools được gọi:** None  
**Workers called sequence:** `retrieval_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): SLA P1: phản hồi 15 phút, xử lý 4 giờ.
- confidence: 0.13
- Correct routing? Yes

**Nhận xét:** Routing chính xác vì câu hỏi chứa keyword P1, hệ thống ưu tiên retrieval ngay lập tức để lấy thông tin SLA khẩn cấp.

---

## Routing Decision #2

**Task đầu vào:**
> Sản phẩm kỹ thuật số (license key) có được hoàn tiền không?

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `policy tool: task contains policy/access keywords | MCP selected`  
**MCP tools được gọi:** `search_kb`  
**Workers called sequence:** `policy_tool_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): Không được hoàn tiền theo Điều 3 chính sách v4.
- confidence: 0.22
- Correct routing? Yes

**Nhận xét:** Supervisor nhận diện đúng keyword "hoàn tiền" (policy) và route sang policy_tool. Worker này sử dụng MCP search_kb để tìm tài liệu trước khi phân tích.

---

## Routing Decision #3

**Task đầu vào:**
> ERR-403-AUTH là lỗi gì và cách xử lý?

**Worker được chọn:** `human_review` (HITL)  
**Route reason (từ trace):** `human review: task contains unknown error or risk keywords`  
**MCP tools được gọi:** None  
**Workers called sequence:** `human_review` -> `retrieval_worker` -> `synthesis_worker`

**Kết quả thực tế:**
- final_answer (ngắn): Không đủ thông tin trong tài liệu nội bộ.
- confidence: 0.30
- Correct routing? Yes

**Nhận xét:** Câu hỏi chứa mã lỗi lạ (ERR-) nên Trigger HITL là quyết định an toàn và chính xác. Sau khi auto-approve, hệ thống vẫn cố gắng retrieve nhưng không thấy kết quả, dẫn đến câu trả lời phủ định an toàn.

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

- Câu route đúng: 15 / 15 (Test questions)
- Câu route sai (đã sửa bằng cách nào?): 0
- Câu trigger HITL: 8 / 38 (Toàn bộ trace)

### Lesson Learned về Routing

> Quyết định kỹ thuật quan trọng nhất nhóm đưa ra về routing logic là gì?  
> (VD: dùng keyword matching vs LLM classifier, threshold confidence cho HITL, v.v.)

1. Sử dụng Keyword-based routing kết hợp với trọng số (Priority) giúp hệ thống xử lý cực nhanh các câu hỏi khẩn cấp (P1) mà không cần tốn chi phí gọi LLM ở bước định tuyến.
2. Thiết lập cơ chế HITL cho các mã lỗi lạ (`ERR-`) là một chốt chặn quan trọng để tránh hallucination khi gặp dữ liệu nằm ngoài Knowledge Base.

### Route Reason Quality

> Nhìn lại các `route_reason` trong trace — chúng có đủ thông tin để debug không?  
> Nếu chưa, nhóm sẽ cải tiến format route_reason thế nào?

Các `route_reason` hiện tại đã khá đủ thông tin cho việc debug (ghi rõ keyword nào kích hoạt và có chọn MCP hay không). Cải tiến tiếp theo là ghi thêm "confidence score" của supervisor nếu chuyển sang dùng LLM Classifier để định lượng độ tin cậy của việc định tuyến.
