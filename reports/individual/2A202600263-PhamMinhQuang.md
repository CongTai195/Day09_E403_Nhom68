# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Phạm Minh Quang
**Vai trò trong nhóm:** Workers Specialist
**Ngày nộp:** 2026-04-14

---

## 1. Tôi phụ trách phần nào?

Tôi chịu trách nhiệm về toàn bộ "Cánh tay" của hệ thống — tức là các Worker thực thi logic nghiệp vụ.

**Module/file tôi chịu trách nhiệm:**
- Thư mục chính: `workers/`
- Files cụ thể: `retrieval.py`, `policy_tool.py`, `synthesis.py`.

Tôi đảm bảo rằng mỗi worker đều tuân thủ đúng Contract đã đề ra, nhận dữ liệu từ State và trả về đúng định dạng để các bước tiếp theo có thể xử lý mượt mà.

---

## 2. Tôi đã ra một quyết định kỹ thuật gì?

**Quyết định:** Triển khai **Grounding & Citation logic** ngay bên trong Synthesis Worker thay vì chỉ để LLM tự trả lời.

**Lý do:**
Ban đầu, synthesis worker chỉ gửi context cho GPT và yêu cầu nó trả lời. Tuy nhiên, tôi nhận thấy mô hình thường quên trích dẫn nguồn hoặc trích dẫn sai định dạng. Tôi quyết định viết thêm một lớp xử lý hậu kỳ (post-processing) để parse các `retrieved_sources` và ép LLM phải sử dụng định dạng `[1]`, `[2]` một cách nghiêm ngặt.

**Lựa chọn thay thế:**
- Dùng Few-shot prompting: Tốt nhưng tốn token và không đảm bảo 100% định dạng.
- Dùng Regex để parse answer: Phức tạp và dễ lỗi nếu câu trả lời của LLM quá bay bổng.

**Kết quả:**
Câu trả lời cuối cùng của pipeline luôn có độ tin cậy cao và minh bạch. Bằng chứng là trong mọi trace (`artifacts/traces/`), field `final_answer` luôn đi kèm với danh sách `sources` chính xác, giúp người dùng dễ dàng kiểm chứng lại thông tin.

---

## 3. Tôi đã sửa một lỗi gì?

**Lỗi:** Tràn Context (Context Overflow) trong Policy Tool.

**Symptom:**
Khi retrieval worker trả về quá nhiều chunks (ví dụ 10-15 chunks), Policy worker gửi toàn bộ chúng kèm theo logic check phức tạp lên LLM, dẫn đến lỗi "context window limit" hoặc làm tăng latency đáng kể.

**Root cause:**
Do chưa có cơ chế lọc (filter) hoặc tóm tắt (summarize) context trước khi thực hiện phân tích chính sách chuyên sâu.

**Cách sửa:**
Tôi đã thêm logic **Re-ranking đơn giản** trong `policy_tool.py` để chỉ chọn ra 5 chunks có độ tương đồng cao nhất liên quan đến từ khóa "policy" hoặc "refund" trước khi gửi lên LLM, giúp tiết kiệm token và tăng tốc độ xử lý.

---

## 4. Tôi tự đánh giá đóng góp của mình

**Tôi làm tốt nhất ở điểm nào?**
Khả năng hiện thực hóa các yêu cầu nghiệp vụ phức tạp thành các module code gọn gàng, stateless và dễ kiểm thử độc lập.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Tôi chưa tối ưu hóa được việc xử lý song song các worker, hiện tại chúng vẫn đang chạy tuần tự theo sự điều phối của Graph.

**Nhóm phụ thuộc vào tôi ở đâu?**
Tôi là người tạo ra "giá trị thực" của câu trả lời. Nếu các worker của tôi làm việc không hiệu quả, kết quả cuối cùng sẽ chỉ là những câu trả lời sáo rỗng hoặc sai lệch.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì?

Tôi sẽ thử cài đặt **Self-Correction Logic** cho Synthesis worker để nó tự kiểm tra lại câu trả lời của mình với context một lần nữa trước khi output, giúp giảm thiểu tối đa rủi ro hallucination.
