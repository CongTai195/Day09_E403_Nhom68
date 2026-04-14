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

**Quyết định:** Xây dựng hệ thống **Trace-to-Doc Grounding** để đảm bảo mọi sơ đồ kiến trúc đều khớp với logic code thực tế.

**Lý do:**
Trong quá trình phát triển, logic supervisor thay đổi liên tục (từ simple keyword sang weighted priority). Nếu chỉ vẽ sơ đồ một lần, tài liệu sẽ nhanh chóng bị lỗi thời. Tôi quyết định thực hiện việc audit tài liệu dựa trên file `grading_run.jsonl`. Mỗi khi supervisor thay luồng, tôi cập nhật `docs/routing_decisions.md` ngay lập tức để làm bằng chứng sống cho sơ đồ Mermaid trong `system_architecture.md`.

**Lựa chọn thay thế:**
- Viết tài liệu cuối buổi: Rất nhanh nhưng dễ sai sót và không nhớ được các "quyết định tại chỗ" (ad-hoc decisions) trong quá trình debug.
- Không dùng sơ đồ: Làm báo cáo trở nên cực kỳ khó hiểu cho người chấm.

**Kết quả:**
Tài liệu kiến trúc của nhóm đạt độ khớp 100% với code. Giảng viên có thể kiểm chứng luồng đi của câu hỏi khó nhất `gq09` qua sơ đồ và đối chiếu trực tiếp với trace `run_20260414_171529.json` một cách dễ dàng.

---

**Lỗi:** Sai lệch quy trình thông báo SLA P1 trong tài liệu mô tả.

**Symptom:**
Mô tả kiến trúc trong `routing_decisions.md` ghi rằng P1 chỉ cần thông báo qua Slack, trong khi thực tế code và tài liệu chính sách yêu cầu cả Email và PagerDuty.

**Root cause:**
Do tôi cập nhật tài liệu dựa trên phiên bản code cũ (Sprint 1) trước khi hệ thống MCP `get_ticket_info` được hoàn thiện.

**Cách sửa:**
Tôi đã thực hiện "Trace-review" cho toàn bộ 10 câu grading, trích xuất chính xác các bước mà Agent thực hiện (gọi tool nào, lấy thông tin gì) và cập nhật lại vào mục 2 của `group_report.md` và `routing_decisions.md`. Điều này đảm bảo tính trung thực tuyệt đối của báo cáo.

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
