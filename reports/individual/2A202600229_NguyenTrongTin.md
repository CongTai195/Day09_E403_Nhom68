# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Trọng Tín
**Vai trò trong nhóm:** Eval Specialist
**Ngày nộp:** 2026-04-14

---

## 1. Tôi phụ trách phần nào?

Tôi chịu trách nhiệm về khâu đánh giá hiệu năng và tự động hóa việc thu thập dữ liệu (Evaluation & Tracing). Công việc của tôi là làm "thước đo" cho toàn bộ hệ thống.

**Module/file tôi chịu trách nhiệm:**
- File chính: `eval_trace.py`
- Functions tôi implement: `analyze_traces`, `run_test_questions`, `run_grading_questions`.

Tôi xây dựng các script để chạy hàng loạt câu hỏi test, tính toán các chỉ số như Latency, Confidence trung bình, và tỷ lệ kích hoạt HITL, từ đó cung cấp dữ liệu thô cho các thành viên khác viết báo cáo.

---

**Quyết định:** Xây dựng script **Automated Criteria Verification** bên trong `eval_trace.py` để tự động kiểm tra tính chính xác của câu trả lời trước khi nộp bài.

**Lý do:**
Việc đọc thủ công 10-15 file JSONL rất dễ bỏ sót các lỗi nhỏ (như thiếu 1 trong 3 kênh thông báo). Tôi đã viết thêm một module nhỏ để parse câu trả lời và so khớp với các keyword bắt buộc cho từng câu hỏi Grading. Điều này giúp nhóm tự tin 100% về điểm số trước khi nhấn nút nộp bài.

**Lựa chọn thay thế:**
- Kiểm tra thủ công: Chậm và dễ sai sót khi áp lực thời gian lớn (sát 18:00).
- Dùng LLM-as-a-judge: Có thể bị hallucination chính việc chấm điểm.

**Kết quả:**
Nhóm đã phát hiện ra câu `gq01` bị thiếu kênh thông báo "PagerDuty" ở phiên bản chạy thử đầu tiên, từ đó kịp thời điều chỉnh Supervisor logic để lấy đầy đủ thông tin. Bằng chứng là bản log cuối cùng `grading_run.jsonl` đã đạt điểm tuyệt đối.

---

**Lỗi:** Logic Abstain quá nhạy cảm làm mất điểm các câu Policy hợp lệ.

**Symptom:**
Agent trả lời "Không đủ thông tin" cho cả những câu nằm trong chính sách V4 (ví dụ câu `gq10` về Flash Sale), mặc dù tài liệu đã có sẵn trong database.

**Root cause:**
Do tôi thiết lập threshold confidence trong `synthesis.py` quá cao (0.5), dẫn đến việc các câu hỏi có exception (khiến conf giảm xuống ~0.3) bị đánh dấu là không tin cậy và tự động chuyển sang câu trả lời phủ định.

**Cách sửa:**
Tôi đã phối hợp với Worker Owner để điều chỉnh lại hàm tính `confidence`, giảm penalty cho các "ngoại lệ chính đáng" và hạ threshold abstain xuống `0.2` cho các câu hỏi policy có context mạnh. Kết quả là câu `gq10` đã trả về đúng nội dung về quy định Flash Sale.

---

## 4. Tôi tự đánh giá đóng góp của mình

**Tôi làm tốt nhất ở điểm nào?**
Tính cẩn thận và kỷ luật trong việc xử lý dữ liệu. Các file trace do tôi tạo ra luôn tuân thủ 100% schema bắt buộc của lab, giúp việc chấm bài tự động trở nên thuận lợi.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Tôi chưa xây dựng được các bảng dashboard trực quan hóa dữ liệu ngay trong script, hiện tại vẫn phải export ra file text rồi nhờ thành viên khác xử lý.

**Nhóm phụ thuộc vào tôi ở đâu?**
Nếu script evaluation của tôi không chạy được vào lúc 17:00, nhóm sẽ không có file `grading_run.jsonl` để nộp — đồng nghĩa với việc mất 30 điểm Grading Questions.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì?

Tôi sẽ cài đặt thêm **Auto-Grading mini-bot** để nó tự đánh giá câu trả lời của pipeline dựa trên `expected_answer`, giúp nhóm biết chính xác mình đang đạt được bao nhiêu % điểm trước khi nộp bài thật.
