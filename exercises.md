# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Mô hình bổ sung các kiến thức chung chung (vô hại), thêm các từ ngữ để làm tăng tính diễn đạt dù không có sẵn trong context nhưng không làm sai lệch thông tin | Mô hình tự ý bịa đặt, xuyên tạc thông tin (số liệu, dữ liệu...) sai lệch, không có căn cứ - đặc biệt trong các lĩnh vực nhạy cảm như: giáo dục, y học, tài chính, pháp luật... | siết chặt system prompt: "Chỉ trả lời thông tin được lưu trong Context cung cấp, không được bịa thông tin nếu không chắc".<br>Tăng cường few-shot example: cung cấp ví dụ minh họa cho output mong muốn<br>Giảm (temperature) của mô hình để giảm tính sáng tạo |
| Answer Relevance | Khi câu hỏi của người dùng rất mở và câu trả lời bắt đầu bằng việc giải thích bối cảnh rộng trước khi đi vào trọng tâm, khiến hệ thống đo lường tự động (RAGAS) đánh giá thấp điểm liên quan trực tiếp tạm thời. | Câu trả lời không đi vào trọng tâm câu hỏi mà lạc đề sang chủ đề khác | Cần tinh chỉnh prompt để mô hình trả lời thẳng vào vấn đề hơn, tránh trả lời lan man hoặc diễn giải dài dòng không cần thiết. |
| Context Recall | Khi thông tin người dùng tìm kiếm nằm rải rác ở nhiều đoạn context khác nhau và hệ thống chỉ retrieve được một phần trong số đó (Ưu tiên chính xác - chất lượng hơn là số lượng trả về) | Hệ thống không retrieve được bất kỳ đoạn context nào chứa thông tin cần thiết để trả lời câu hỏi. | Cải thiện chiến lược chunking, tăng kích thước chunk hoặc sử dụng kỹ thuật re-ranking để ưu tiên các đoạn context chứa từ khóa quan trọng từ câu hỏi |
| Context Precision | Khi context chứa nhiều thông tin gây nhiễu hoặc thông tin không liên quan đến câu hỏi, làm giảm độ chính xác khi đánh giá mức độ liên quan của retrieved chunks. | Context chứa thông tin sai lệch, mâu thuẫn hoặc không đáng tin cậy, dẫn đến việc mô hình dựa vào sai sót để trả lời. | Cần xử lý dữ liệu trước khi indexing (ví dụ: metadata filtering, semantic filtering, hybrid search) để giảm thiểu việc retrieve các chunks không liên quan |
| Completeness | Khi câu trả lời bao gồm đầy đủ thông tin cần thiết nhưng được trình bày ngắn gọn, súc tích. | Câu trả lời bị thiếu sót, bỏ qua các phần quan trọng của thông tin cần thiết. | Rà soát lại yêu cầu đầu ra của LLM (ví dụ: bullet points, tóm tắt theo ý chính...) để đảm bảo tính đầy đủ thông tin cần truyền đạt |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Để phát hiện position bias ta cung cấp cho Judge cùng 1 câu hỏi với 2 câu trả lời có thứ tự đảo ngược cho nhau.
> Condition 1: Đưa 2 answer vào prompt theo thứ tự gốc (Answer_A, Answer_B)
> Condition 2: Đưa 2 answer vào prompt theo thứ tự đảo ngược (Answer_B, Answer_A) - giữ nguyên nội dung
> Sau đó: Nếu judge liên tục chọn đáp án ở vị trí đầu (Answer_A) thì hệ thống đánh giá có position bias. Nếu không có sự thay đổi đáng kể khi thay đổi vị trí, có thể kết luận hệ thống ít bị ảnh hưởng bởi position bias
**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Thêm tiêu chí "trình bày súc tích, ngắn gọn" vào rubric desgin và điểm phạt nếu trả lời quá dài dòng không cần thiết. 
> Yêu cầu Judge chấm theo mật độ thông tin (số ý đúng) thay vì số lượng từ.
> System prompt: "Không ưu tiên trả lời câu dài: Ưu tiên trả lời ngắn gọn, súc tích"

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Đo lường độ chính xác: Kiểm tra mức độ đồng thuậngiữa LLM và con người.
> Khắc phục thiên kiến: Tránh các lỗi như self-preference hoặc đánh giá sai lệch ngữ cảnh thực tế.
> Cải tiến liên tục: Cung cấp phản hồi cho việc cải tiến hệ thống.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness |>= 0.85 | Rất quan trọng, không chấp nhận bịa đặt thông tin |
| Answer Relevance |>= 0.8 | Quan trọng, đảm bảo trả lời đúng trọng tâm |
| Completeness |>= 0.8 | Quan trọng, đảm bảo trả lời đầy đủ thông tin |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Offline evaluation: Dùng trong giai đoạn phát triển và CI/CD trên golden dataset để kiểm thử trước khi deploy product
> Online evaluation: Dùng trên production với dữ liệu người dùng thực tế để giám sát chất lượng phản hồi
> Human review: Dùng định kỳ để audit chất lượng, xây dựng golden dataset ban đầu, hoặc đánh giá các trường hợp phức tạp mà mô hình tự động chưa đủ tin cậy

## Part 2 — Core Coding (14:45–15:40)

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

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

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

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
