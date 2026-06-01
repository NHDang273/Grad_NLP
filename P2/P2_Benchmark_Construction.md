# **P2 — FBI Framework: Benchmark Construction**

---

### **Slide 1: Tổng quan quy trình tạo benchmark**

* **Mục tiêu của P2:** tạo một benchmark có khả năng đo được blind spots của evaluator LLM theo cách có kiểm soát
* **Nguồn dữ liệu đầu vào:** 500 prompts chọn từ 6 benchmark phổ biến
  * WizardLM, MT-Bench, UltraChat, LIMA, LLMBar, IFEval
* **Pipeline 3 bước:**
  1. Sinh **gold answers** bằng GPT-4-Turbo
  2. Sinh **perturbed answers** theo checklist lỗi diễn giải được
  3. **Human review** để xác nhận perturbation hợp lệ
* **Đầu ra cuối:** 2400 tuples dạng *(prompt, gold answer, perturbed answer)*
* **Ý nghĩa thiết kế:** benchmark không đo "chất lượng answer" trực tiếp, mà đo "khả năng phát hiện lỗi" của evaluator

---

### **Slide 2: 4 Task Categories và logic chia bài toán**

* FBI chia benchmark thành 4 nhóm năng lực để cover các failure mode chính của LLM-as-a-Judge:
  * **LF (Long-form Writing):** coherence, chronology, grammar, completeness
  * **F (Factuality):** sai thông tin, sai thực thể, sai số liệu, thiếu fact quan trọng
  * **IF (Instruction Following):** vi phạm ràng buộc format/độ dài/trình tự/nội dung
  * **R (Reasoning):** sai tính toán, sai đơn vị, sai công thức, sai bước suy luận
* **Lý do cần 4 nhóm riêng:** mỗi nhóm cần một kiểu "kiểm tra" khác nhau
  * F cần đối chiếu sự thật
  * IF cần đối chiếu với instruction gốc
  * R cần verify logic/toán học
  * LF cần đọc toàn văn bản để bắt lỗi tinh tế
* **Thông điệp chính:** benchmark này nhắm vào độ khó của evaluators, không phải độ khó của generators

---

### **Slide 3: 22 Perturbation Categories (Table 2)**

* Tổng cộng **22 loại perturbation** được thiết kế để tạo lỗi có chủ đích, nhưng vẫn giữ context hợp lý
* **Theo từng nhóm task:**
  * LF: Grammar, Spelling, Consistency, Chronology, Coherence, Comprehensiveness
  * F: Contextual, Entity, Incorrect Fact, Number Errors, Opposite Fact, Remove Fact
  * IF: Do Less, Do More, Ignore Format, Sequence Errors, Assumptions
  * R: Calculations, Copying Numbers, Final Errors, Incorrect Units, Wrong Formula
* **Nguyên tắc xây dựng perturbation:**
  * Mỗi perturbation phải ảnh hưởng đến chất lượng theo một hướng rõ ràng
  * Có thể giải thích bằng checklist (interpretable)
  * Hạn chế sửa quá nhiều yếu tố cùng lúc để tránh nhiễu biến can thiệp
* **Ví dụ nhanh:**
  * Entity error: "Poland" -> "London"
  * Calculation error: "2 + 3 = 5" -> "2 + 3 = 6"
  * IF error: yêu cầu 4 dòng nhưng output 6 dòng

---

### **Slide 4: Perturbation Generation + Human-in-the-Loop Validation**

* **Bước tạo lỗi tự động:** GPT-4-Turbo được prompt theo từng category để tạo perturbed answer
* **3 lỗi thường gặp khi sinh tự động:**
  1. Lệch category (sinh lỗi khác với lỗi mong muốn)
  2. Mức độ thay đổi quá nhẹ hoặc quá mạnh
  3. Explanation nói có lỗi nhưng text output không thể hiện lỗi đó
* **Human validation quy mô lớn:** 17 học viên cao học review toàn bộ 2400 perturbations
* **Nhãn của human annotators:**
  * Valid
  * Invalid
  * Score-Invariant
  * Not Relevant
  * Not Sure
* **Ý nghĩa:** đây là lớp kiểm soát chất lượng quan trọng nhất để benchmark không bị "noise" ngay từ dữ liệu đầu vào

---

### **Slide 5: Score-Invariant Perturbations và kiểm tra false positive**

* **Score-Invariant perturbation:** thay đổi hình thức nhưng **không làm giảm chất lượng** (vd: paraphrase)
* Nếu evaluator tốt, nhóm này phải được chấm điểm giống gold answer
* **Nguồn tạo SI cases:**
  * Được gán nhãn trong quá trình human review
  * Sinh bổ sung bằng Gemini-1.5-Pro paraphrase, sau đó human xác thực
* **Tổng SI cases:** 516
* **Vai trò trong đánh giá:**
  * Phát hiện evaluator quá "khó tính" (hạ điểm sai)
  * Cân bằng với metric "bắt lỗi thật" ở các perturbations có lỗi
* **Thông điệp quan trọng:** một evaluator tốt cần vừa nhạy với lỗi thật, vừa ổn định với thay đổi vô hại

---

### **Slide 6: Từ benchmark đến thực nghiệm P3/P4**

* Sau khi benchmark được xây dựng, paper dùng nó để stress-test evaluator qua:
  * **P3:** 3 paradigms (single-answer, pairwise, reference-guided) + nhiều prompting strategies
  * **P4/P5:** so sánh hiệu năng detect blind spots trên GPT-4-Turbo và các model khác
* **Giá trị của P2 trong toàn bộ paper:**
  * Tạo "grounded testbed" có thể lặp lại
  * Cung cấp error taxonomy có diễn giải rõ ràng
  * Cho phép phân tích theo task, theo strategy, theo từng loại lỗi
* **Takeaway P2:** benchmark construction là nền tảng để tất cả kết luận ở các phần sau có giá trị khoa học

---

### **Tóm tắt P2 — 3 takeaway chính**

| # | Takeaway | Ý nghĩa |
|---|----------|---------|
| 1 | Dữ liệu được tạo có kiểm soát và có human validation | Giảm nhiều noise so với tạo synthetic thuần tự động |
| 2 | 22 perturbation categories bao phủ 4 năng lực cốt lõi | Đo được blind spots một cách fine-grained |
| 3 | Score-invariant là thành phần bắt buộc | Tránh đánh giá lệch theo hướng "chỉ biết phạt" |
