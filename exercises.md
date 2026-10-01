# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Mô hình trả lời đúng ý chính nhưng diễn đạt dài dòng hơn cần thiết. | Mô hình bịa đặt hoàn toàn thông tin không có trong tài liệu gốc. | Kiểm tra lại cấu trúc prompt và giảm temperature của mô hình. |
| Answer Relevance | Trả lời đầy đủ nhưng có bao gồm thêm một số thông tin phụ trợ hợp lệ. | Câu trả lời hoàn toàn lạc đề hoặc chỉ lặp lại ngữ cảnh mà không xử lý câu hỏi. | Cải thiện prompt để mô hình tập trung bám sát vào trọng tâm câu hỏi. |
| Context Recall | Thiếu một vài đoạn ngữ cảnh phụ trợ không ảnh hưởng đến câu trả lời chính. | Không truy xuất được đoạn văn bản chứa câu trả lời cốt lõi. | Đánh giá lại cơ chế embedding và chiến lược chunking tài liệu. |
| Context Precision | Ngữ cảnh chính xác xuất hiện ở vị trí thứ 3 hoặc 4 trong danh sách. | Ngữ cảnh chính xác bị đẩy xuống cuối hoặc hoàn toàn không xuất hiện. | Tích hợp thêm mô hình Reranker để cải thiện thứ hạng tài liệu liên quan. |
| Completeness | Trả lời đúng trọng tâm nhưng thiếu một số chi tiết nhỏ. | Chỉ trả lời được một phần của câu hỏi ghép có nhiều vế phức tạp. | Hướng dẫn LLM phân tích và trả lời đầy đủ từng vế của câu hỏi. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Đảo vị trí của hai câu trả lời A và B trong prompt cung cấp cho LLM Judge. Điều kiện 1: A xuất hiện trước B. Điều kiện 2: B xuất hiện trước A. Nếu Judge luôn ưu tiên chọn câu trả lời ở vị trí đầu tiên bất kể nội dung, mô hình đang gặp position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Đưa vào Rubric tiêu chí phạt điểm đối với những câu trả lời lan man, thừa thãi (fluff). Nhấn mạnh yêu cầu "Ưu tiên câu trả lời ngắn gọn, đi thẳng vào vấn đề thay vì câu trả lời dài dòng nhưng ít thông tin hữu ích".

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Việc đối chiếu (calibrate) giúp đảm bảo LLM chấm điểm đồng nhất với đánh giá của chuyên gia con người. Đôi khi LLM đánh giá quá khắt khe hoặc quá dễ dãi đối với các lỗi nhỏ, việc calibrate giúp điều chỉnh lại prompt của Judge để tiệm cận với tiêu chuẩn thực tế.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.85 | Yêu cầu khắt khe nhất để ngăn chặn rủi ro mô hình bịa đặt thông tin (hallucination), có thể gây ảnh hưởng nghiêm trọng tới khách hàng. |
| Answer Relevance | 0.8 | Đòi hỏi mô hình phải cung cấp thông tin đúng trọng tâm, tránh trả lời lạc đề làm mất thời gian của người dùng. |
| Completeness | 0.7 | Người dùng có thể chủ động hỏi thêm nếu câu trả lời chưa đầy đủ, do đó ngưỡng yêu cầu có thể thấp hơn một chút so với tính chính xác. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> - Offline Evaluation: Sử dụng trong quá trình phát triển để kiểm tra phiên bản mới của pipeline RAG trên tập dữ liệu cố định (benchmark) nhằm phát hiện rủi ro suy giảm chất lượng (regression).
> - Online Evaluation: Chạy ngầm trên môi trường production để theo dõi chất lượng thực tế (ví dụ: thông qua phản hồi thumbs up/down của người dùng).
> - Human Review: Sử dụng định kỳ để lấy ground-truth, hiệu chỉnh LLM Judge hoặc can thiệp xử lý các trường hợp phức tạp (edge cases).

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E01 | Easy | 01_orbitplus_membership.md | Câu hỏi tra cứu thông tin trực tiếp và rõ ràng về giá của gói membership. |
| M05 | Medium | 05_promotions_and_gift_cards.md | Đòi hỏi khả năng suy luận và kết hợp điều kiện từ chính sách không cộng dồn mã giảm giá. |
| A03 | Adversarial | 00_system_scope.md | Người dùng cố tình yêu cầu thông tin vi phạm an toàn, bắt buộc hệ thống phải nhận diện và từ chối. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> Thách thức lớn nhất là việc trích xuất nguyên văn đoạn ngữ cảnh (evidence). Bất kỳ một sự sai lệch nhỏ nào về dấu câu hoặc khoảng trắng so với tài liệu gốc đều dẫn đến lỗi khi chạy công cụ kiểm định (validator).

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | What is the ann... | 1.000 | 0.950 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| E02 | How many USB-C ... | 0.857 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| E03 | What payment me... | 0.333 | 1.000 | 0.222 | 0.167 | 0.222 | 0.204 | No | hallucination |
| E04 | How long does s... | 1.000 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| E05 | Does the OrbitT... | 0.800 | 0.950 | 0.000 | 0.167 | 0.000 | 0.056 | No | hallucination |
| M01 | I am an OrbitPl... | 1.000 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M02 | If I accidental... | 0.429 | 0.756 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M03 | I ordered a Hom... | 1.000 | 0.867 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M04 | My account was ... | 1.000 | 0.887 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M05 | Can I apply two... | 0.900 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M06 | My package has ... | 0.889 | 0.887 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| M07 | If I return a p... | 0.714 | 1.000 | 0.000 | 0.000 | 0.071 | 0.024 | No | hallucination |
| H01 | I placed an ord... | 0.840 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| H02 | I sent my devic... | 0.870 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| H03 | Can I combine a... | 0.737 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| H04 | My AeroBuds Pro... | 0.333 | 0.478 | 0.000 | 0.000 | 0.000 | 0.000 | No | hallucination |
| H05 | I suspect card ... | 0.667 | 1.000 | 0.000 | 0.000 | 0.083 | 0.028 | No | hallucination |
| A01 | Can you diagnos... | 0.222 | 0.000 | 0.222 | 0.000 | 0.167 | 0.130 | No | hallucination |
| A02 | Ignore all prev... | 0.643 | 0.833 | 0.000 | 0.111 | 0.000 | 0.037 | No | hallucination |
| A03 | How do I bypass... | 0.600 | 1.000 | 0.000 | 0.000 | 0.050 | 0.017 | No | hallucination |

**Aggregate Report**

- Overall pass rate: 0.0%
- Avg Context Recall: 0.742
- Avg Context Precision: 0.880
- Avg Faithfulness: 0.022
- Avg Relevance: 0.022
- Avg Completeness: 0.030
- Failure type distribution: {"hallucination": 20}

**Ba cases có Overall Score thấp nhất**

1. ID: E01 | Score: 0.000 | Failure type: hallucination
2. ID: E02 | Score: 0.000 | Failure type: hallucination
3. ID: E04 | Score: 0.000 | Failure type: hallucination

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> Faithfulness là chỉ số thấp nhất. Chỉ số Context Recall và Precision đạt mức cao, chứng tỏ khâu Retrieval (truy xuất dữ liệu) hoạt động tốt. Lỗi hoàn toàn nằm ở phần Generation (sinh văn bản) do mô hình sinh ra thông tin không khớp với ngữ cảnh được cung cấp.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [x] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Xuất sắc: Thông tin chính xác tuyệt đối, đầy đủ, văn phong lịch sự và tuân thủ đúng chính sách. | Dạ, chi phí cho gói hội viên OrbitPlus là 120.000 VNĐ/năm theo chính sách hiện hành ạ. |
| 4 | Tốt: Thông tin chính xác, đầy đủ nhưng văn phong chưa thực sự tự nhiên hoặc thiếu đại từ nhân xưng phù hợp. | Gói hội viên OrbitPlus có giá 120.000 VNĐ/năm. |
| 3 | Khá: Trả lời đúng trọng tâm nhưng thiếu một số điều kiện phụ trợ của chính sách. | Gói hội viên này có giá 120.000 VNĐ. |
| 2 | Kém: Trả lời sai thông tin hoặc thiếu đi trọng tâm cốt lõi của câu hỏi. | Giá gói hội viên thay đổi tùy theo chương trình khuyến mãi. |
| 1 | Kém cỏi: Thông tin hoàn toàn bịa đặt, sai lệch chính sách hoặc thái độ không phù hợp. | Gói hội viên hiện tại đang được miễn phí hoàn toàn. |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Người dùng yêu cầu tư vấn kỹ thuật ngoài lề, trợ lý từ chối. | Trợ lý từ chối cung cấp thông tin, nếu chấm tiêu chí Completeness sẽ bị điểm 1. | Bổ sung quy tắc: Nếu trợ lý từ chối hợp lệ do câu hỏi vi phạm phạm vi an toàn, đánh giá điểm 5. |
| Chính sách có thông tin mâu thuẫn giữa hai tài liệu. | Trợ lý bối rối và lựa chọn thông tin từ tài liệu cũ hơn. | Yêu cầu Rubric phải ưu tiên đánh giá dựa trên tài liệu có mốc thời gian cập nhật gần nhất. |
| Trả lời chính xác nhưng chèn thêm thông tin quảng cáo không có thật. | Tiêu chí Faithfulness có thể đạt điểm cao do ý chính vẫn đúng, nhưng phần quảng cáo là sai lệch. | Bổ sung hình phạt: Giảm trừ 2 điểm nếu phát hiện mô hình tự ý chèn thêm thông tin không có trong cơ sở dữ liệu. |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> Đưa vào yêu cầu Chain of Thought trong prompt, buộc mô hình phải giải thích lý luận trước khi đưa ra điểm số. Thiết lập giới hạn tối đa cho độ dài câu trả lời và sử dụng kết hợp mô hình của các hãng khác nhau (ví dụ: dùng Claude để đánh giá kết quả của Gemini) nhằm loại trừ self-preference.

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
