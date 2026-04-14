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

## 2. Tôi đã ra một quyết định kỹ thuật gì?

**Quyết định:** Sử dụng định dạng **JSONL** cho file grading log (`grading_run.jsonl`) thay vì JSON array.

**Lý do:**
Khi chạy hàng chục câu hỏi lớn cho grading, nếu một câu làm pipeline bị crash và tôi đang dùng JSON array, toàn bộ file JSON đó sẽ bị hỏng cấu trúc (invalid JSON) và không thể đọc được. Với JSONL (mỗi dòng là một JSON object), nếu một câu lỗi, tôi vẫn giữ được kết quả của tất cả các câu trước đó.

**Lựa chọn thay thế:**
- Standard JSON: Dễ đọc hơn cho con người nhưng kém ổn định khi streaming dữ liệu lớn.
- CSV: Gọn nhưng không thể hiện được các cấu trúc dữ liệu lồng nhau (như list các sources).

**Kết quả:**
Quá trình chạy evaluation diễn ra cực kỳ ổn định. Bằng chứng là khi gặp một câu hỏi gây lỗi timeout từ LLM, script `eval_trace.py` của tôi vẫn tiếp tục chạy các câu sau và lưu lại log đầy đủ cho 14/15 câu còn lại mà không làm mất dữ liệu.

---

## 3. Tôi đã sửa một lỗi gì?

**Lỗi:** Sai lệch thời gian đo Latency.

**Symptom:**
Latency được ghi nhận trong trace cực kỳ thấp (vài ms), trong khi thực tế người dùng phải chờ 5-7 giây mới thấy câu trả lời.

**Root cause:**
Do tôi đặt hàm đo thời gian `time.time()` bên trong từng node thay vì đo bao quát toàn bộ lượt chạy `graph.invoke()`, dẫn đến việc bỏ sót overhead của framework và thời gian truyền dữ liệu giữa các node.

**Cách sửa:**
Tôi đã refactor lại wrapper trong `eval_trace.py` để bao bọc toàn bộ lời gọi graph, đảm bảo `latency_ms` trong trace phản ánh chính xác trải nghiệm thực tế của người dùng.

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
