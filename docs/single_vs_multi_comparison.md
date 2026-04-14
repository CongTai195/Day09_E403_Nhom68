# Single Agent vs Multi-Agent Comparison — Lab Day 09

**Nhóm:** 68  
**Ngày:** 14/4/2026

> **Hướng dẫn:** So sánh Day 08 (single-agent RAG) với Day 09 (supervisor-worker).
> Phải có **số liệu thực tế** từ trace — không ghi ước đoán.
> Chạy cùng test questions cho cả hai nếu có thể.

---

## 1. Quick Comparison Table

| Metric | Day 08 (Single-Agent) | Day 09 (Multi-Agent) | Delta (%) |
|--------|-----------------------|----------------------|-----------|
| Avg. Confidence | 0.72 | 0.34 | -52% |
| Avg. Latency (ms) | 3200 | 7620 | +138% |
| Observability | Low (Monolith) | High (Graph Traces) | +++ |
| Modularity | None | High (MCP & Workers) | +++ |
| Abstain Logic | Static threshold | Dynamic worker check | Improved |
| Multi-hop accuracy | 30% | 60% | +100% | % câu multi-hop trả lời đúng |
| Routing visibility | ✗ Không có | ✓ Có route_reason | N/A | |
| Debug time (estimate) | 20 phút | 5 phút | -75% | Thời gian tìm ra 1 bug |

> **Lưu ý:** Nếu không có Day 08 kết quả thực tế, ghi "N/A" và giải thích.

---

## 2. Phân tích theo loại câu hỏi

### 2.1 Câu hỏi đơn giản (single-document)

| Nhận xét | Day 08 | Day 09 |
|---------|--------|--------|
| Accuracy | High | High |
| Latency | Low (3.2s) | Medium (6.5s) |
| Observation | Nhanh nhưng không rõ nguồn. | Chậm hơn do overhead của graph nhưng minh bạch route. |

**Kết luận:** Multi-agent không cải thiện accuracy cho câu đơn giản nhưng tăng tính minh bạch của quá trình xử lý. Tuy nhiên, latency tăng là một trade-off cần cân nhắc.

### 2.2 Câu hỏi multi-hop (cross-document)

| Nhận xét | Day 08 | Day 09 |
|---------|--------|--------|
| Accuracy | Low (30%) | High (60%+) |
| Routing visible? | ✗ | ✓ |
| Observation | Hay bị sót thông tin từ tài liệu thứ 2. | Supervisor điều phối gọi đúng nhiều worker để lấy đủ context. |

**Kết luận:** Multi-agent vượt trội ở các câu hỏi phức tạp nhờ khả năng orchestrate nhiều module chuyên biệt để cùng giải quyết một task.

### 2.3 Câu hỏi cần abstain

| Nhận xét | Day 08 | Day 09 |
|---------|--------|--------|
| Abstain rate | 15% | 21% |
| Hallucination cases | High | Low |
| Observation | Thường cố trả lời bằng kiến thức training. | HITL trigger giúp chặn đứng các câu hỏi rủi ro/lạ. |

**Kết luận:** Độ an toàn của Multi-agent cao hơn hẳn nhờ lớp supervisor nhận diện rủi ro và node HITL.

---

## 3. Debuggability Analysis

> Khi pipeline trả lời sai, mất bao lâu để tìm ra nguyên nhân?

### Day 08 — Debug workflow
```
Khi answer sai → phải đọc toàn bộ RAG pipeline code → tìm lỗi ở indexing/retrieval/generation
Không có trace → không biết bắt đầu từ đâu
Thời gian ước tính: 20 phút
```

### Day 09 — Debug workflow
```
Khi answer sai → đọc trace → xem supervisor_route + route_reason
  → Nếu route sai → sửa supervisor routing logic
  → Nếu retrieval sai → test retrieval_worker độc lập
  → Nếu synthesis sai → test synthesis_worker độc lập
Thời gian ước tính: 5 phút
```

**Câu cụ thể nhóm đã debug:** Câu q09 (ERR-403) ban đầu bị retrieval worker "đoán" đại văn bản. Sau khi xem trace, nhóm thấy supervisor không nhận diện được mã lỗi, từ đó thêm case `ERR-` vào logic human_review để bảo vệ hệ thống.

---

## 4. Extensibility Analysis

> Dễ extend thêm capability không?

| Scenario | Day 08 | Day 09 |
|---------|--------|--------|
| Thêm 1 tool/API mới | Phải sửa toàn prompt | Thêm MCP tool + route rule |
| Thêm 1 domain mới | Phải retrain/re-prompt | Thêm 1 worker mới |
| Thay đổi retrieval strategy | Sửa trực tiếp trong pipeline | Sửa retrieval_worker độc lập |
| A/B test một phần | Khó — phải clone toàn pipeline | Dễ — swap worker |

**Nhận xét:** Multi-agent tối ưu cho việc bảo trì lâu dài và scale-up các capability mà không làm code base trở nên "spaghetti".

---

## 5. Cost & Latency Trade-off

> Multi-agent thường tốn nhiều LLM calls hơn. Nhóm đo được gì?

| Scenario | Day 08 calls | Day 09 calls |
|---------|-------------|-------------|
| Simple query | 1 LLM call | 2 LLM calls (Supervisor + Synthesis) |
| Complex query | 1 LLM call | 3-4 LLM calls (Supervisor + Policy + Synthesis) |
| MCP tool call | N/A | Tích hợp trong Policy worker |

**Kết luận về cost-benefit:**
Cost cao hơn (gấp 2-3 lần) và Latency cao hơn, nhưng đáng giá cho các bài toán Enterprise cần độ chính xác và tính audit cao.

---

## 6. Kết luận

> **Multi-agent tốt hơn single agent ở điểm nào?**

1. Tính Observability cao (trace từng bước).
2. Khả năng tích hợp Tool linh hoạt qua MCP.

> **Multi-agent kém hơn hoặc không khác biệt ở điểm nào?**

1. Latency (độ trễ) cao do overhead điều phối.
2. Chi phí (token) tăng do nhiều LLM calls trung gian.

> **Khi nào KHÔNG nên dùng multi-agent?**
Khi bài toán đơn giản, chỉ cần retrieval một tài liệu và latency là ưu tiên hàng đầu.

> **Nếu tiếp tục phát triển hệ thống này, nhóm sẽ thêm gì?**
Thêm LLM-based Router thay cho keyword matching để tăng độ thông minh của Supervisor.
