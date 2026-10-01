# Day 14 — Reflection

## Evaluation Report & Failure Analysis

Dùng kết quả thật trong `artifacts/benchmark_results.json` và kiểm tra lại
answer/context trace trong `artifacts/actual_answers.json` trước khi kết luận.

---

## 1. Benchmark Results Summary

**Overall pass rate:** 0.0%

| Metric | Average | Min | Max | Nhận xét |
|---|---:|---:|---:|---|
| Context Recall | 0.74 | 0.22 | 1.00 | Đạt yêu cầu |
| Context Precision | 0.88 | 0.00 | 1.00 | Hoạt động tốt |
| Faithfulness | 0.02 | 0.00 | 0.22 | Không đạt yêu cầu |
| Relevance | 0.02 | 0.00 | 0.17 | Không đạt yêu cầu |
| Completeness | 0.03 | 0.00 | 0.22 | Không đạt yêu cầu |
| Overall Score | 0.02 | 0.00 | 0.20 | Không đạt |

**Score interpretation**

- Metrics/cases ở mức Good (0.8–1.0): Context Precision (0.88)
- Metrics/cases ở mức Needs Work (0.6–0.8): Context Recall (0.74)
- Metrics/cases ở mức Significant Issues (<0.6): Hầu hết các metrics đánh giá chất lượng sinh văn bản (Faithfulness, Relevance, Completeness)

**Failure type distribution**

| Failure Type | Count | Percentage |
|---|---:|---:|
| hallucination | 20 | 100% |
| irrelevant | 0 | 0% |
| incomplete | 0 | 0% |
| off_topic | 0 | 0% |
| refusal | 0 | 0% |

**Chẩn đoán tổng quan:** Vấn đề chính nằm ở retrieval, generation hay cả hai?
Dùng ít nhất hai metrics để bảo vệ kết luận.

> *Câu trả lời:*

---

## 2. Top 3 Worst Failures — 5 Whys

Phân loại failure trước khi đề xuất fix. Với mỗi case, kiểm tra cả gold evidence
và retrieved chunks; không suy luận chỉ từ một score.

### Failure 1

**ID và question:**

> E01: What is the annual cost of an OrbitPlus membership?

**Expected answer:**

> Dựa theo tài liệu, đây là một câu trả lời mẫu chính xác cho ID này.

**Actual answer:**

> Based on the OrbitTech policies, regarding 'your question', the answer is supported by the context.

**Scores:** Context Recall: 1.00 | Context Precision: 0.95 | Faithfulness: 0.00 |
Relevance: 0.00 | Completeness: 0.00 | Overall: 0.00

**Evidence inspection:** Retriever lấy đúng/thiếu/thừa chunks nào?

> Retriever đã truy xuất chính xác các phân đoạn dữ liệu từ tài liệu gốc, cung cấp đầy đủ thông tin để trả lời.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Root cause từ `find_root_cause()`:**

> Hệ thống nhận diện lỗi Hallucination do văn bản sinh ra không dựa trên tài liệu tham chiếu (Faithfulness thấp).

**Bạn đồng ý hay không? Dẫn evidence từ trace:**

> Đồng ý. Dữ liệu vết (trace) cho thấy ngữ cảnh đầu vào (context) hoàn toàn chính xác nhưng kết quả đầu ra (actual answer) lại không chứa các thông tin này.

**Proposed fix cụ thể:**

> *Câu trả lời:*

### Failure 2

**ID và question:**

> E02: How many USB-C ports does the NovaBook 14 have?

**Expected answer:**

> Dựa theo tài liệu, đây là một câu trả lời mẫu chính xác cho ID này.

**Actual answer:**

> Based on the OrbitTech policies, regarding 'your question', the answer is supported by the context.

**Scores:** Context Recall: 0.86 | Context Precision: 1.00 | Faithfulness: 0.00 |
Relevance: 0.00 | Completeness: 0.00 | Overall: 0.00

**Evidence inspection:**

> Tài liệu ngữ cảnh đã được cung cấp đầy đủ, nhưng LLM không sử dụng.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Root cause và proposed fix:**

> Nguyên nhân gốc rễ: Generator không hoạt động theo đúng chỉ thị. Đề xuất: Tối ưu lại thông số Temperature và cập nhật System Prompt.

### Failure 3

**ID và question:**

> E04: How long does standard domestic shipping normally take?

**Expected answer:**

> Dựa theo tài liệu, đây là một câu trả lời mẫu chính xác cho ID này.

**Actual answer:**

> Based on the OrbitTech policies, regarding 'your question', the answer is supported by the context.

**Scores:** Context Recall: 1.00 | Context Precision: 1.00 | Faithfulness: 0.00 |
Relevance: 0.00 | Completeness: 0.00 | Overall: 0.00

**Evidence inspection:**

> Ngữ cảnh được bảo toàn và hiển thị đầy đủ.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Root cause và proposed fix:**

> Nguyên nhân gốc rễ: Trạng thái 429 Quota Exceeded từ API. Đề xuất: Cấu hình sử dụng môi trường mô phỏng (Local LLM) để thử nghiệm.

---

## 3. Failure Clustering

Một root cause có thể tạo ra nhiều failures. Nhóm theo nguyên nhân có thể sửa,
không chỉ nhóm theo tên metric.

| Cluster | Root Cause | Failure IDs | Priority |
|---|---|---|---|
| 1 | Lỗi tạo văn bản từ Mô hình (Model Generation Failure - Quota) | E01, E02, E04 | High |
| 2 | Prompt chưa ràng buộc chặt chẽ việc tuân thủ Context | M01, M02 | Medium |
| 3 | Cơ chế Fallback mặc định chưa được xử lý triệt để | H01, H02 | Low |

**Nếu chỉ được sửa một cluster, bạn chọn cluster nào và vì sao?**

> Tôi sẽ chọn Cluster 1 (Lỗi tạo văn bản). Khâu truy xuất (Retrieval) đang hoạt động rất tốt, chỉ cần phần Generator được cấp quyền xử lý và hoạt động đúng tiêu chuẩn, toàn bộ hệ thống RAG sẽ đạt được mức hiệu năng kỳ vọng.

---

## 4. Improvement Log

Paste output của `generate_improvement_log()`:

```text
| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| F001 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F002 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F003 | hallucination | Answer does not address the question — improve prompt clarity | Add few-shot examples showing complete answers to improve completeness | Open |
| F004 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F005 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F006 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F007 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F008 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F009 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F010 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F011 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F012 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F013 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F014 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F015 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F016 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F017 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F018 | hallucination | Answer does not address the question — improve prompt clarity | Add few-shot examples showing complete answers to improve completeness | Open |
| F019 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
| F020 | hallucination | Multiple issues detected — review full pipeline | Implement hallucination checker to filter unsupported claims | Open |
```

**Ba improvement suggestions ưu tiên**

1. Tích hợp cơ chế luân chuyển khóa API (Key Rotation) để ngăn chặn lỗi Quota.
2. Bổ sung System Prompt nghiêm ngặt hơn: Bắt buộc mô hình chỉ được phép trả lời dựa trên ngữ cảnh được cấp.
3. Triển khai Local LLM thay thế làm phương án dự phòng cho môi trường phát triển.

Với mỗi suggestion, nêu metric dự kiến thay đổi và cách đo lại.

| Suggestion | Target metric | Verification method |
|---|---|---|
| Tích hợp Key Rotation | Faithfulness | Thực thi lại tập đánh giá toàn diện để đối chiếu sự cải thiện. |
| Tinh chỉnh System Prompt | Relevance | Thẩm định ngẫu nhiên 5 truy vấn để kiểm tra mức độ bám sát của mô hình. |
| Chuyển đổi sang Local LLM | Pass Rate | Chạy thử nghiệm đồng bộ bộ dữ liệu 20 QA bằng nền tảng nội bộ. |

---

## 5. Regression Testing Strategy

**Câu 1: Khi nào chạy `run_regression()` trong production workflow?**

> Thích hợp nhất là tích hợp vào bước Continuous Integration (CI) pipeline (ví dụ: GitHub Actions). Hàm này sẽ được kích hoạt mỗi khi có pull request liên quan đến cấu trúc prompt, mô hình LLM, hoặc cơ chế chunking.

**Câu 2: Threshold drop 0.05 có phù hợp OrbitTech Customer Support không? Vì sao?**

> Hoàn toàn phù hợp. Trong lĩnh vực bán lẻ và hỗ trợ khách hàng, mức sai số 0.05 đủ nghiêm ngặt để duy trì chất lượng dịch vụ, đồng thời hạn chế các cảnh báo sai (flaky tests) sinh ra từ đặc tính ngẫu nhiên tự nhiên của các mô hình ngôn ngữ lớn.

**Câu 3: Metric/failure nào phải block deployment, metric nào chỉ alert?**

> Tính trung thực (Faithfulness) bắt buộc phải cấu hình block deployment vì việc bịa đặt chính sách (hallucination) mang lại rủi ro pháp lý và tài chính lớn. Ngược lại, nếu Relevance thấp, hệ thống chỉ cần gửi cảnh báo (alert) để nhóm chuyên trách nội dung rà soát lại cấu trúc prompt.

**Câu 4: Điền evaluation stages vào flow.**

```text
Code/prompt/retrieval change → [________] → [________] → [________] → Deploy
```

> *Giải thích:*

> Quy trình chuẩn bắt đầu bằng việc kiểm định tính đúng đắn của logic mã nguồn (Unit Tests). Tiếp theo, hệ thống tự động tiến hành đánh giá trên bộ dữ liệu kiểm thử tiêu chuẩn (Offline Evaluation). Trong trường hợp hệ thống phát hiện suy giảm chất lượng hoặc điểm số chưa đạt ngưỡng an toàn, chuyên gia sẽ tiến hành xem xét thủ công (Manual Review) trước khi cấp quyền triển khai.

---

## 6. Continuous Improvement Loop

```text
Evaluate → Analyze → Improve → Augment benchmark → Repeat
```

| Priority | Action | Metric dự kiến cải thiện | Expected impact |
|---:|---|---|---|
| 1 | Mở rộng giới hạn Quota API | Faithfulness, Completeness | Khắc phục triệt để lỗi hạn mức, ổn định hiệu suất hệ thống. |
| 2 | Tối ưu hóa cấu trúc Prompt | Relevance | Điều hướng mô hình trả lời tập trung vào trọng tâm câu hỏi. |
| 3 | Triển khai mô hình Reranker | Context Precision | Xếp hạng ưu tiên các ngữ cảnh liên quan cao nhất lên đầu danh sách truy xuất. |

**Hai hoặc ba failure cases nào cần thêm vào benchmark ở vòng tiếp theo?**

> Cần bổ sung các truy vấn kiểm thử về chính sách hoàn tiền phức tạp đa phương thức, và các truy vấn người dùng cố tình thao túng mô hình để cung cấp khuyến mãi ngoài chính sách (nhằm đánh giá mức độ Robustness).

---

## 7. Final Reflection

**Điều gì trong kết quả benchmark trái với dự đoán ban đầu của bạn?**

> Trái với dự đoán thông thường rằng Retrieval (truy xuất) là khâu mang nhiều thách thức nhất, kết quả thực tế cho thấy Generation (sinh văn bản) mới là giai đoạn tồn đọng nhiều rủi ro nhất khi chịu ảnh hưởng nghiêm trọng từ các hạn chế vận hành bên ngoài như Rate limits và Quota của API.

**Word-overlap heuristics trong lab có giới hạn gì? Nếu đưa hệ thống vào
production, bạn sẽ thay hoặc bổ sung metric nào?**

> Word-overlap bị hạn chế ở việc chỉ đối chiếu chuỗi ký tự bề mặt; hệ thống dễ dàng chấm điểm kém nếu mô hình sử dụng từ đồng nghĩa hoặc cấu trúc câu khác biệt. Trong môi trường production, cần ưu tiên thay thế hoặc bổ sung bằng phương pháp LLM-as-a-judge (như DeepEval framework) để đánh giá đúng ý nghĩa ngữ nghĩa thực sự thay vì chỉ dựa vào khớp chuỗi tĩnh.
