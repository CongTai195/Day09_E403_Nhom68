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

**Quyết định:** Triển khai **Hybrid Policy Engine** (kết hợp Rule-based Python và LLM context) bên trong `policy_tool.py`.

**Lý do:**
Chỉ dựa vào LLM để phân tích chính sách hoàn tiền thường dẫn đến sai sót khi gặp các mốc thời gian nhạy cảm (như đơn hàng trước ngày 01/02). Tôi quyết định viết thêm một lớp logic bằng Python để "block" cứng các đơn hàng không thuộc phạm vi chính sách V4 trước khi gửi dữ liệu lên LLM. Điều này đảm bảo tính tuân thủ tuyệt đối cho các câu hỏi về thời gian.

**Lựa chọn thay thế:**
- Chỉ dùng Prompt Engineering: Không an toàn, LLM vẫn có thể "ảo giác" và áp dụng luật mới cho đơn hàng cũ.
- Dùng hoàn toàn Hard-code: Quá cứng nhắc, không xử lý được các trường hợp tinh tế như lỗi từ phía nhà sản xuất.

**Kết quả:**
Hệ thống đạt độ chính xác 100% trong việc phát hiện các đơn hàng "lỗi thời" (ví dụ câu `gq02`). Bằng chứng là trong trace `run_20260414_171446.json`, Policy Worker đã trả về đúng `policy_version_note` cảnh báo về việc thiếu tài liệu V3.

---

**Lỗi:** Hallucination về chính sách hoàn tiền cho đơn hàng cũ (`gq02`).

**Symptom:**
Agent trả lời khách hàng *được hoàn tiền* cho đơn hàng ngày 31/01/2026 dựa trên chính sách V4 (trong khi chính sách này chỉ áp dụng từ tháng 2).

**Root cause:**
LLM trong Policy Tool bị nhầm lẫn giữa ngày đặt hàng và ngày yêu cầu hoàn tiền, tự động "áp đặt" các điều kiện của phiên bản tài liệu mới nhất có trong database.

**Cách sửa:**
Tôi đã thêm kiểm tra logic thời gian trong hàm `analyze_policy` của `policy_tool.py`. Nếu đơn hàng trước ngày `2026-02-01`, Agent sẽ trả ra kết quả "Không đủ thông tin" và ngăn chặn synthesis worker bịa đặt câu trả lời. Kết quả là câu `gq02` trong `grading_run.jsonl` hiện đã trả ra "Không đủ thông tin" chính xác 100%.

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
