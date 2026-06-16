# P5 — Multi-model Results + Conclusion

---

### Slide 1: So sánh các model — Ai là evaluator tốt nhất?

* Sau khi xác định các strategy tốt nhất ở P4, tác giả mở rộng đánh giá sang nhiều evaluator khác:
  * GPT-4-Turbo
  * Gemini-1.5-Pro
  * Claude-3-Opus
  * LLaMA-3-70B-Instruct
  * Prometheus 2

* Kết quả nổi bật (Table 4):

  | Model | Paradigm | Nhận xét |
  |---------|---------|---------|
  | GPT-4-Turbo | Reference-less | Tốt nhất |
  | Gemini-1.5-Pro | Reference-less | Trung bình |
  | Claude-3-Opus | Reference-less | Kém nhất |
  | LLaMA-3-70B | Reference-less | Kém hơn GPT đáng kể |

* Với Vanilla strategy:

  | Model | LF↓ | F↓ | IF↓ | R↓ |
  |---------|------|------|------|------|
  | GPT-4-Turbo | 57% | 54% | 57% | 25% |
  | Gemini-1.5-Pro | 61% | 73% | 54% | 41% |
  | LLaMA-3-70B | 86% | 95% | 90% | 71% |

* Claude-3-Opus chỉ được test ở Vanilla do chi phí API cao.
* Kết quả của Claude thậm chí còn tệ hơn GPT và Gemini:
  * LF = 74%
  * F = 84%
  * IF = 75%
  * R = 47%

* Takeaway:
  * GPT-4-Turbo vẫn là evaluator mạnh nhất trong các setup reference-less.
  * Tuy nhiên ngay cả GPT vẫn bỏ sót hơn 50% lỗi ở đa số task.

---

### Slide 2: Bất ngờ lớn — LLaMA tốt nhất ở Reference-Based

* Khi chuyển sang Reference Evaluation, bảng xếp hạng đảo ngược hoàn toàn.

| Model | LF↓ | F↓ | IF↓ | R↓ |
|---------|------|------|------|------|
| GPT-4-Turbo | 26% | 11% | 49% | 4% |
| Gemini-1.5-Pro | 25% | 7% | 17% | 3% |
| LLaMA-3-70B | **3%** | **1%** | **5%** | 5% |

* Thoạt nhìn:
  * LLaMA gần như hoàn hảo.
  * Chỉ bỏ sót 1–5% perturbations.

* Nhưng tác giả phát hiện một vấn đề khác:

  * Với Score-Invariant perturbations:
    * GPT-4-Turbo: 63%
    * Gemini: 33%
    * LLaMA: chỉ 13%

* Điều đó nghĩa là:

  * LLaMA rất hiếm khi cho điểm tuyệt đối.
  * Ngay cả những answer đúng và chỉ paraphrase lại cũng bị trừ điểm.

* Đây là hiện tượng:

  **Overly Strict Evaluator**

* Bài học quan trọng:

  > Số liệu đẹp chưa chắc evaluator tốt.
  > Một model có thể "ít bỏ sót lỗi" chỉ vì nó hạ điểm tất cả mọi thứ.

---

### Slide 3: Prometheus 2 — Evaluator chuyên dụng có thắng được GPT?

* Prometheus 2 là model được huấn luyện riêng cho nhiệm vụ evaluation.

* Kỳ vọng:
  * Evaluator chuyên biệt phải vượt các general-purpose LLMs.

* Kết quả thực tế:

| Model | LF↓ | F↓ | IF↓ | R↓ |
|---------|------|------|------|------|
| GPT-4-Turbo | 26% | 11% | 49% | 4% |
| Prometheus 2 | 51% | 62% | 53% | 12% |

* Quan sát:

  * Factual:
    * GPT = 11%
    * Prometheus = 62%

  * Long-form:
    * GPT = 26%
    * Prometheus = 51%

  * Instruction Following:
    * GPT = 49%
    * Prometheus = 53%

* Prometheus thua GPT gần như toàn bộ categories.

* Kết luận:

  * Fine-tuning cho evaluation chưa đủ.
  * Evaluator vẫn gặp cùng loại blind spots như các LLM thông thường.
  * Chuyên môn hóa training không tự động tạo ra evaluator đáng tin.

---

### Slide 4: Tổng kết Blind Spots theo từng loại lỗi

* Không phải mọi lỗi đều khó phát hiện như nhau.

* Dễ detect nhất:

  ### Reasoning

  * Calculation errors
  * Wrong formula
  * Incorrect units

* Lý do:

  * Evaluator có thể tự kiểm tra toán học.
  * Có tín hiệu đúng/sai khá khách quan.

* Khó detect nhất:

| Error Category | Tỷ lệ bỏ sót |
|----------------|-------------|
| Chronology (LF) | gần 100% |
| Comprehensiveness (LF) | 91–100% |
| Remove Fact (F) | 94–99% |
| Assumptions (IF) | 95–98% |

* Pattern rất rõ:

  * Càng cần đọc toàn bộ context,
  * càng cần hiểu ngữ nghĩa sâu,
  * evaluator càng thất bại.

* Nói cách khác:

  Evaluator giỏi bắt lỗi "cục bộ".

  Evaluator rất kém với lỗi "tinh tế và phân tán".

---

### Slide 5: Kết luận & Implications

### Kết luận chính

* LLMs hiện tại chưa đáng tin để đóng vai evaluator cho text generation.

* Trung bình:
  * bỏ sót hơn 50% perturbations.
  * kể cả với model tốt nhất.

### Khuyến nghị thực tế

1. Ưu tiên Reference-Based Evaluation khi có thể.

2. Không nên kỳ vọng:
   * rubric,
   * axis,
   * prompt engineering
   sẽ giải quyết được vấn đề.

3. Thận trọng với:
   * leaderboard,
   * benchmark,
   * model ranking
   dựa trên LLM-as-a-Judge.

### Limitations của paper

* Chưa đánh giá:
  * Multi-agent evaluation
  * Debate-based evaluation

* Chỉ sử dụng:
  * English prompts

* 22 perturbation categories chưa bao phủ toàn bộ failure modes.

### Hướng nghiên cứu tương lai

* Multilingual evaluation
* Tool-use evaluation
* Planning & agent tasks
* Multi-agent judges
* Better calibration mechanisms

### Takeaway cuối cùng

"LLMs có thể tạo ra văn bản rất tốt,
nhưng việc đánh giá chất lượng văn bản vẫn khó hơn nhiều so với chúng ta tưởng."