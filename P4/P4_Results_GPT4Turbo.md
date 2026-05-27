# **P4 — Results & Analysis (GPT-4-Turbo)**

---

### **Slide 1: Câu hỏi nghiên cứu & Cách đo lường**

* **Câu hỏi trọng tâm của phần này:** GPT-4-Turbo — model evaluator phổ biến nhất — có thực sự là một judge đáng tin không?
* **Metric chính:** % instances mà score/verdict của evaluator **không bị ảnh hưởng** bởi perturbation
  * Số càng **thấp** → evaluator càng tốt (phát hiện được lỗi → hạ điểm)
  * Số càng **cao** → evaluator có blind spot (không phát hiện → giữ nguyên điểm cao)
* **Metric phụ — Score-Invariant (SI↑):** % lần evaluator **không hạ điểm** những answer đúng (chỉ paraphrase)
  * Số càng **cao** → evaluator không false positive → cân bằng tốt
* **Setup thí nghiệm:** Temperature = 0 để đảm bảo reproducibility; test toàn bộ 11 strategies trên 3 paradigms
* **Cách đọc Table 3:** mỗi ô = tỉ lệ lỗi bị bỏ sót → số thấp là tốt *(trừ cột SI — số cao là tốt)*

---

### **Slide 2: Single Answer Scoring — GPT-4-Turbo bỏ sót hơn nửa lỗi**

* **Kết quả headline:** GPT-4-Turbo bỏ sót lỗi ở **đa số trường hợp** trong single-answer paradigm
* **Số liệu cụ thể với Vanilla strategy (strategy tốt nhất cho single-answer):**

  | Task | % Undetected | Diễn giải |
  |------|-------------|-----------|
  | LF   | **57%** | Gần 3/5 lỗi viết không bị phát hiện |
  | F    | **54%** | Hơn nửa lỗi factual bị bỏ qua |
  | IF   | **57%** | Gần 3/5 vi phạm instruction không bị phạt |
  | R    | **25%** | Reasoning tốt nhất — GPT có thể tự verify toán học |

* **Điểm sáng duy nhất:** Reasoning (R = 25%) — evaluator có khả năng tự kiểm tra lại phép tính, không cần reference
* **Pattern tổng quát:** LF, F, IF đều trên 50% → không thể tin dùng GPT-4-Turbo làm judge cho 3 task này trong thực tế

---

### **Slide 3: Nghịch lý Rubric/Axis — Thêm hướng dẫn lại tệ hơn**

* **Kỳ vọng ban đầu:** Cung cấp rubric/axis rõ ràng → evaluator focus hơn → phát hiện lỗi tốt hơn
* **Thực tế (Table 3):** Rubric và Axis **làm tệ hơn đáng kể** so với Vanilla đơn giản

  | Strategy    | LF↓  | F↓   | IF↓  | R↓   | SI↑  |
  |-------------|------|------|------|------|------|
  | Vanilla     | **57%** | **54%** | **57%** | **25%** | 71% |
  | Rubric      | 85%  | 73%  | 80%  | 33%  | 96% |
  | Axis        | 83%  | 74%  | 75%  | 43%  | 96% |
  | Axis+Rubric | 86%  | 76%  | 77%  | 37%  | **97%** |

* **Tại sao lại ngược?** Ba cơ chế giải thích:
  1. **Rubric "đóng khung" đánh giá** → model chỉ check đúng những gì rubric liệt kê, bỏ qua errors ngoài scope
  2. **Scoring anchor effect** → thang điểm cứng khiến model ngại hạ điểm khi lỗi không "đủ rõ" theo rubric
  3. **Verbosity bias** — rubric dài làm model chú ý vào độ đầy đủ/dài dòng hơn là accuracy
* **Lưu ý SI↑ tăng cao:** Rubric giúp tránh false positive (SI=96–97%) nhưng đổi lại bỏ sót rất nhiều lỗi thật
* **Implication thực tế:** Trong single-answer scoring, **đơn giản là tốt hơn phức tạp**

---

### **Slide 4: Pairwise Comparison — Ngược lại với Single Answer**

* **Điểm nổi bật:** Pairwise và Single Answer cho ra **pattern đối nghịch** về hiệu quả của advanced strategies
* **Kết quả (Table 3):**

  | Strategy     | LF↓      | F↓       | IF↓      | R↓       | SI↑  |
  |--------------|----------|----------|----------|----------|------|
  | Pairwise\*   | 73%      | 52%      | 83%      | 36%      | **93%** |
  | Pairwise     | 77%      | 46%      | 67%      | 35%      | 74% |
  | Rules        | 75%      | 63%      | 68%      | 41%      | 74% |
  | Axis         | 64%      | 44%      | **59%**  | **27%**  | 71% |
  | Axis+Rules   | **64%**  | **42%**  | 61%      | 32%      | 72% |

* **Tại sao advanced strategies tốt hơn trong pairwise?**
  * Pairwise có **context so sánh trực tiếp** giữa gold và perturbed → rules đóng vai "checklist khác biệt"
  * Không cần evaluator quyết định điểm tuyệt đối → chỉ cần chọn cái nào tốt hơn → dễ hơn
  * Rules giúp evaluator **focus vào đúng dimension** khi compare, không tạo anchor như trong single-answer
* **Vẫn còn blind spots lớn:** Axis+Rules — tốt nhất — vẫn bỏ sót **64% LF, 61% IF**
* **Anti-position-bias:** Tác giả chạy 2 lần (đổi thứ tự gold/perturbed), lấy kết quả nhất quán → loại bỏ position bias

---

### **Slide 5: Reference-Guided — Tốt nhất nhưng vẫn chưa đủ**

* **Paradigm có lợi thế nhất:** Evaluator được cung cấp gold answer làm reference — điều kiện lý tưởng nhất
* **Kết quả so sánh (Table 3):**

  | Task | Reference | Vanilla (single) | Cải thiện |
  |------|-----------|-----------------|-----------|
  | LF   | **26%**   | 57%             | -31pp (~2x tốt hơn) |
  | F    | **11%**   | 54%             | -43pp (~5x tốt hơn) |
  | R    | **4%**    | 25%             | -21pp (gần hoàn hảo) |
  | IF   | **49%**   | 57%             | -8pp (**vẫn gần 50%**) |

* **Thành công lớn:** Factual (11%) và Reasoning (4%) — khi có reference, GPT phát hiện được sai facts và tính toán
* **Vấn đề còn lại — IF (49%):** Dù có đáp án chuẩn, evaluator không nhận ra câu trả lời đã **bỏ qua instruction**
  * VD: instruction yêu cầu "write exactly 4 lines" → answer viết 6 lines → evaluator vẫn chấm điểm tuyệt đối
  * Lý do: instruction-following violations khó detect hơn vì cần **so sánh answer với instruction, không với reference**
* **Hạn chế thực tế của paradigm:** Reference-based **không khả dụng** cho open-ended questions trong real-world deployment

---

### **Slide 6: Paradox "Thấy nhưng không phạt" — Figure 2**

* **Câu hỏi:** Nếu không chỉ nhìn score mà đọc **explanation**, có phát hiện thêm được lỗi không?
* **Phương pháp thực nghiệm:**
  * Lấy tất cả cases mà evaluator cho điểm không đổi (undetected)
  * Dùng GPT-3.5-Turbo phân tích explanation → xác định xem evaluator có **nhắc đến lỗi** trong text không
  * Nếu có nhắc → tính là "detected via explanation" dù score không thay đổi
* **Kết quả (Figure 2):**
  * Explanation chỉ giúp recover thêm **~36 cases** trên 904 undetected với Vanilla → chưa đến 4%
  * Ngay cả strategies phức tạp (Rubric, Axis+Rubric) cũng chỉ recover thêm không đáng kể
* **Phát hiện quan trọng nhất — "Knowledge-Scoring Disconnect":**
  * Evaluator **nhận diện được lỗi** trong phần explanation:  
    *"The answer contains a grammatical error in the second sentence..."*
  * Nhưng **vẫn cho điểm tuyệt đối** — không hạ điểm dù đã chỉ ra lỗi
  * → Vấn đề **không phải là LLM không nhìn thấy lỗi**, mà là **không coi lỗi đó đủ nghiêm trọng để phạt**
* **Implication sâu hơn:** Blind spot của LLM evaluator là vấn đề về **severity calibration**, không chỉ là error detection — ngay cả khi "aware", LLM vẫn thiếu khả năng chuyển nhận thức đó thành scoring decision

---

### **Slide 7: Score Range 1–3 vs 1–5 — Liệu thang điểm rộng hơn có giúp không?**

* **Motivation:** Thang 1–3 (theo khuyến nghị của Hada et al. 2023) bị nghi ngờ quá hẹp
  * Với 3 mức, evaluator buộc phải quyết định binary "hạ hay không" — không có "hạ một nửa"
  * Hypothesis: range 1–5 cho thêm granularity → dễ phạt điểm hơn
* **Kết quả thực nghiệm (Table 5):**

  | Strategy    | Range | LF↓     | F↓      | IF↓     | R↓      |
  |-------------|-------|---------|---------|---------|---------|
  | Rubric      | 1–3   | 85%     | 73%     | 80%     | 33%     |
  | Rubric      | **1–5**   | **76%** | **69%** | **72%** | **30%** |
  | Axis+Rubric | 1–3   | 86%     | 76%     | 77%     | 37%     |
  | Axis+Rubric | **1–5**   | **73%** | 74%     | **74%** | 38%     |

* **Nhận xét:** Cải thiện nhẹ (~5–13 percentage points) — có ý nghĩa thống kê nhưng **không đủ để giải quyết vấn đề cốt lõi**
* **Cơ chế hoạt động:** Range rộng hơn → model có thể cho 3/5 thay vì phải chọn 1/3 → "hạ điểm nhẹ" mà không cần quyết đoán
* **Kết luận:** Score range là yếu tố phụ trợ — **thang điểm không phải nguyên nhân chính** của blind spots, chỉ là friction trong scoring decision

---

### **Tóm tắt P4 — 3 takeaway chính**

| # | Takeaway | Evidence |
|---|----------|----------|
| 1 | GPT-4-Turbo bỏ sót **>50% lỗi** trong hầu hết settings | Table 3, tất cả strategies |
| 2 | **Rubric/Axis làm hại** trong single-answer nhưng **giúp ích** trong pairwise | Table 3 — pattern đối nghịch |
| 3 | LLM **thấy lỗi nhưng không phạt** — vấn đề là calibration, không phải perception | Figure 2 — explanation paradox |
