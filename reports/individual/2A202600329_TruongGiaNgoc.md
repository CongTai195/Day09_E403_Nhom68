# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Truong Gia Ngoc
**Vai trò trong nhóm:** Docs Specialist
**Ngày nộp:** 2026-04-14

---

## 1. Tôi phụ trách phần nào?

Tôi chịu trách nhiệm toàn bộ về khâu tài liệu kiến trúc và phân tích quyết định của nhóm. Đây là phần quan trọng để thuyết minh cho thiết kế kỹ thuật của dự án.

**Module/file tôi chịu trách nhiệm:**
- Thư mục chính: `docs/`
- Files cụ thể: `system_architecture.md`, `routing_decisions.md`, `single_vs_multi_comparison.md`.

Tôi làm việc trực tiếp với Trace Owner để lấy số liệu thực tế từ các file JSON trong `artifacts/traces/`, sau đó trực quan hóa chúng thành sơ đồ và bảng so sánh.

---

## 2. Tôi đã ra một quyết định kỹ thuật gì?

**Quyết định:** Sử dụng **Mermaid Diagram** để thể hiện luồng xử lý của hệ thống thay vì dùng hình ảnh tĩnh.

**Lý do:**
Việc dùng Mermaid giúp tài liệu kiến trúc luôn đồng bộ với code. Khi supervisor thay đổi logic routing, tôi chỉ cần cập nhật lại text trong file markdown là sơ đồ sẽ tự động thay đổi theo. Điều này cực kỳ hữu hiệu trong môi trường làm việc Agile của lab.

**Lựa chọn thay thế:**
- Dùng hình ảnh (JPG/PNG): Đẹp nhưng khó cập nhật và không thể "diff" được trên Git.
- ASCII Art: Đơn giản nhưng khó nhìn với các hệ thống graph phức tạp.

**Kết quả:**
Sơ đồ trong `docs/system_architecture.md` mô tả cực kỳ rõ ràng ranh giới giữa Supervisor và các Worker chuyên biệt, giúp giảng viên dễ dàng nắm bắt kiến trúc nhóm chỉ trong 30 giây.

---

## 3. Tôi đã sửa một lỗi gì?

**Lỗi:** Sai lệch số liệu giữa Trace thực tế và Báo cáo so sánh.

**Symptom:**
Trong file `single_vs_multi_comparison.md`, tôi ghi nhận latency của Multi-Agent là 5s, nhưng thực tế các file trace lại ghi nhận trung bình lên tới 7.6s.

**Root cause:**
Do tôi lấy số liệu từ một phiên chạy thử (test run) cũ thay vì lấy từ phiên chạy evaluation chính thức.

**Cách sửa:**
Tôi đã xây dựng một file bảng tính nhỏ để tổng hợp (aggregate) dữ liệu từ toàn bộ 15 file trace trong `artifacts/traces/`, sau đó cập nhật lại số liệu chính xác vào tài liệu so sánh, đảm bảo tính trung thực theo yêu cầu của SCORING.md.

---

## 4. Tôi tự đánh giá đóng góp của mình

**Tôi làm tốt nhất ở điểm nào?**
Khả năng trực quan hóa các khái niệm trừu tượng (như multi-agent graph) thành các tài liệu dễ hiểu và chuyên nghiệp.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Tôi chưa dành nhiều thời gian để viết sâu về phần phân tích "Trade-off" giữa chi phí và hiệu năng.

**Nhóm phụ thuộc vào tôi ở đâu?**
Tôi là người "kết nối" các thành phần code rời rạc thành một bức tranh tổng thể cho bài nộp. Không có tài liệu của tôi, nhóm sẽ mất 10 điểm Group Documentation.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì?

Tôi sẽ bổ sung thêm một trang **Dashboard Preview** bằng Markdown để hiển thị top 3 quyết định routing thú vị nhất kèm theo link dẫn thẳng tới file trace tương ứng.
