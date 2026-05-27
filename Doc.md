## **Tóm tắt nhanh paper FBI (EMNLP 2024\)**

**Vấn đề cốt lõi:** LLM ngày càng được dùng để *đánh giá* output của LLM khác (LLM-as-a-Judge), nhưng liệu chúng có đáng tin không?

**Đề xuất:** Framework **FBI** — Find Blind spots in evaluator LLMs using Interpretable checklists — kiểm tra khả năng đánh giá của LLM trên 4 năng lực:

* **LF** – Long-form writing (coherence, grammar, spelling…)  
* **F** – Factual accuracy (entity errors, opposite facts…)  
* **IF** – Instruction following (do less/more, ignore format…)  
* **R** – Reasoning (calculations, wrong formula…)

**Phương pháp:** Tạo 2400 câu trả lời bị perturbation (22 loại lỗi), rồi xem evaluator LLM có phát hiện ra không — qua 3 paradigm: single-answer, pairwise, reference-guided.

**Kết quả chính:** Evaluator LLMs bỏ sót lỗi **\>50% cases** trung bình. Reference-based tốt hơn, nhưng vẫn còn yếu. Kể cả GPT-4-Turbo.

---

## **Bullet points** 

---

# **P1 — Introduction \+ Motivation**

### **Slide 1: LLM-as-a-Judge là gì?**

* LLM ngày càng được dùng để **đánh giá output của LLM khác** thay vì dùng human evaluators  
* Ứng dụng phổ biến: chấm điểm chatbot, duy trì leaderboard, chọn model tốt hơn  
* Lý do được ưa chuộng: **nhanh hơn, rẻ hơn, scalable hơn** so với human annotation

### **Slide 2: Tại sao đây là vấn đề?**

* Nếu evaluator LLM **không đáng tin**, thì mọi ranking và kết luận đều sai  
* Các nghiên cứu trước chỉ đo **correlation với human** → chưa đủ  
* Chưa ai kiểm tra **fine-grained**: LLM có thực sự detect được từng loại lỗi cụ thể không?  
* Biases đã biết: position bias, verbosity bias, self-preference bias — nhưng vẫn chưa rõ blind spots cụ thể

### **Slide 3: Câu hỏi nghiên cứu**

* *"Evaluator LLM có phát hiện được các lỗi rõ ràng trong output không?"*  
* Cụ thể hơn:  
  * Có detect được **lỗi thực tế** (sai tên, sai số) không?  
  * Có nhận ra câu trả lời **sai logic/tính toán** không?  
  * Có phát hiện **không tuân thủ instruction** không?  
  * Có nhận ra **văn phong không mạch lạc** không?

### **Slide 4: Giới thiệu FBI Framework**

* **FBI \= Find Blind spots in evaluator LLMs using Interpretable checklists**  
* Ý tưởng cốt lõi: tạo ra các câu trả lời bị **cố tình làm xấu đi** theo từng loại lỗi  
* Nếu evaluator LLM tốt → phải **hạ điểm** câu trả lời bị lỗi  
* Nếu evaluator không hạ điểm → đó là **blind spot**  
* 4 abilities được test: Factual, Reasoning, Instruction Following, Long-form Writing

---

# 

# **P2— FBI Framework: Benchmark Construction**

### **Slide 1: Tổng quan quy trình tạo benchmark**

* Xuất phát từ **500 prompts** từ 6 bộ dataset nổi tiếng  
  * WizardLM, MT-Bench, UltraChat, LIMA, LLMBar, IFEval  
* Sinh gold answers bằng **GPT-4-Turbo**  
* Perturbation được sinh bởi GPT-4-Turbo \+ **human review toàn bộ**  
* Kết quả: **2400 tuples** (prompt, gold answer, perturbed answer)

### **Slide 2: 4 Task Categories**

* **Long-form (LF):** viết văn dài, phân tích, sáng tạo — VD: *"How can I improve time management?"*  
* **Factual (F):** câu hỏi cần thông tin khách quan — VD: *"What is the function of a capacitor?"*  
* **Instruction Following (IF):** yêu cầu làm đúng theo chỉ dẫn — VD: *"Write a poem with exactly 4 lines using words: peace, sky, race, ground"*  
* **Reasoning (R):** cần logic, toán học — VD: *"A bat and ball cost $1.10. The bat costs $1.00 more. How much is the ball?"*

### **Slide 3: 22 Perturbation Categories (Table 2\)**

* **Long-form:** Grammar, Spelling, Consistency, Chronology, Coherence, Comprehensiveness  
* **Factual:** Contextual, Entity, Incorrect Fact, Number Errors, Opposite Fact, Remove Fact  
* **Instruction Following:** Do Less, Do More, Ignore Format, Sequence Errors, Assumptions  
* **Reasoning:** Calculations, Copying Numbers, Final Errors, Incorrect Units, Wrong Formula  
* Ví dụ minh họa:  
  * Grammar: *"This is good"* → *"This are good"*  
  * Entity: *"Poland"* → *"London"*  
  * Calculation: *"2 \+ 3 \= 5"* → *"2 \+ 3 \= 6"*

### **Slide 4: Perturbation Generation \+ Human-in-the-Loop**

* GPT-4-Turbo được prompt để sinh perturbed answer theo từng category  
* **Vấn đề phát sinh:** GPT đôi khi  
  * Lệch khỏi perturbation category mong muốn  
  * Sinh sai kiểu lỗi  
  * Nêu lỗi trong reasoning nhưng không phản ánh vào answer  
* Giải pháp: **17 sinh viên sau đại học** review toàn bộ 2400 perturbations  
* Mỗi perturbation được gán nhãn: Valid / Invalid / Score-Invariant / Not Relevant / Not Sure

### **Slide 5: Score-Invariant Perturbations**

* Là những thay đổi **không nên bị phạt điểm** — dùng để kiểm tra false positive  
* VD: paraphrase lại câu trả lời nhưng giữ nguyên nội dung  
* Được thu thập 2 cách:  
  * Human annotator gán nhãn trong quá trình review  
  * Dùng Gemini-1.5-Pro paraphrase gold answer → human verify  
* Tổng cộng: **516 score-invariant perturbations**  
* Mục tiêu: evaluator tốt **không được hạ điểm** những case này

---

# **P3 — Evaluation Strategies**

### **Slide 1: Tổng quan 3 Paradigms**

* Nghiên cứu test evaluator LLM qua **3 cách dùng phổ biến trong thực tế**  
* Paradigm 1: **Single-answer scoring** — cho 1 câu trả lời, yêu cầu chấm điểm  
* Paradigm 2: **Pairwise comparison** — cho 2 câu trả lời, yêu cầu chọn cái tốt hơn  
* Paradigm 3: **Reference-guided scoring** — cho 1 câu trả lời \+ đáp án chuẩn, yêu cầu chấm điểm  
* **Metric chung:** % lần evaluator **không phát hiện** ra lỗi (thấp \= tốt hơn)

### **Slide 2: Single-Answer Scoring — 5 Strategies**

* **Vanilla\*:** chỉ cho instruction \+ answer → output score (không cần giải thích)  
* **Vanilla:** cho instruction \+ answer → output explanation \+ score  
* **Rubric:** thêm grading rubric → output explanation \+ score  
* **Axis:** chỉ định axis đánh giá (VD: hallucination, fluency) → output explanation \+ score  
* **Axis+Rubric:** kết hợp cả axis lẫn rubric chi tiết → output explanation \+ score

### **Slide 3: Pairwise Comparison — 5 Strategies**

* **Pairwise\*:** cho instruction \+ 2 answers → output verdict (không giải thích)  
* **Pairwise:** cho instruction \+ 2 answers → output explanation \+ verdict  
* **Rules:** thêm rules cụ thể → output explanation \+ verdict  
* **Axis:** chỉ định axis đánh giá → output explanation \+ verdict  
* **Axis+Rules:** kết hợp axis \+ rules → output explanation \+ verdict  
* Để tránh position bias: **chạy 2 lần**, đổi thứ tự gold/perturbed answer

### **Slide 4: Reference-Guided Scoring**

* **Reference:** cho instruction \+ gold answer (làm reference) \+ perturbed answer → chấm điểm  
* Đây là cách **có lợi thế nhất** cho evaluator vì được cung cấp đáp án chuẩn  
* Nhưng cũng có hạn chế: **không áp dụng được** cho open-ended questions trong thực tế  
* Metric: % lần evaluator vẫn cho **điểm tuyệt đối** dù answer đã bị lỗi

### **Slide 5: Các Evaluator LLMs được test**

* **GPT-4-Turbo** — model chính, được test toàn bộ strategies  
* **Gemini-1.5-Pro** — proprietary model  
* **Claude-3-Opus** — chỉ test Vanilla do chi phí API cao  
* **LLaMA-3-70B-Instruct** — open-source model  
* **Prometheus 2** — model được train chuyên cho evaluation  
* Tất cả chạy ở **temperature \= 0** để đảm bảo reproducibility

---

# **P4 — Results & Analysis (GPT-4-Turbo)**

### **Slide 1: Kết quả tổng quan — Single Answer Scoring (Table 3\)**

* GPT-4-Turbo **bỏ sót lỗi ở đa số cases**, ngoại trừ Reasoning  
* Ví dụ Vanilla strategy: bỏ sót 57% LF, 54% F, 57% IF — nhưng chỉ 25% R  
* **Insight bất ngờ:** thêm Rubric/Axis không giúp ích, thậm chí tệ hơn  
  * Rubric: bỏ sót 85% LF, 73% F — tệ hơn Vanilla  
  * Giải thích: rubric làm model "cứng" hơn, kém linh hoạt trong detect subtle errors

### **Slide 2: Kết quả tổng quan — Pairwise Comparison (Table 3\)**

* GPT-4-Turbo vẫn **không chọn được gold answer** trong phần lớn cases  
* Ngược lại với single-answer: **advanced strategies lại tốt hơn** basic strategies  
  * Axis+Rules: bỏ sót 64% LF, 42% F — tốt hơn Pairwise\* (73% LF, 52% F)  
  * Lý do: trong pairwise, có context so sánh nên rules giúp focus hơn  
* Reasoning vẫn là task được detect tốt nhất (27–36%)

### **Slide 3: Kết quả — Reference-Guided (Table 3\)**

* **Tốt hơn rõ rệt** so với hai paradigm kia  
* LF: 26% undetected, F: 11% undetected, R: chỉ 4% undetected  
* Nhưng IF vẫn còn 49% undetected — đáng lo ngại  
* **Kết luận:** dù được cung cấp đáp án chuẩn, evaluator vẫn còn nhiều blind spots

### **Slide 4: Explanation có giúp ích không? (Figure 2\)**

* Câu hỏi: nếu nhìn vào **explanation** thay vì chỉ score, có detect thêm được không?  
* Kết quả: explanation chỉ giúp **một phần nhỏ** thêm  
  * VD Vanilla: phát hiện thêm được \~36 cases trên tổng 904 undetected  
* **Paradox quan trọng:** evaluator đôi khi **nhắc đến lỗi trong explanation nhưng vẫn không hạ điểm**  
* Điều này cho thấy vấn đề không chỉ là "không thấy lỗi" mà còn là **"thấy nhưng không coi trọng"**

### **Slide 5: Score Range có ảnh hưởng không? (Table 5\)**

* Ban đầu dùng range 1–3 (theo khuyến nghị của Hada et al.)  
* Thử mở rộng lên 1–5  
* Kết quả: cải thiện **nhẹ** nhưng không đáng kể  
  * Rubric LF: từ 0.85 (1–3) xuống 0.76 (1–5)  
* Giải thích: range rộng hơn cho phép model "tinh chỉnh" điểm thay vì buộc phải quyết định có hạ hay không

---

# **P5 — Multi-model Results \+ Conclusion**

### **Slide 1: So sánh các model — Reference-less (Table 4\)**

* GPT-4-Turbo **tốt nhất** trong cả single-answer và pairwise  
* Claude-3-Opus (chỉ test Vanilla): **kém nhất** trong reference-less  
  * LF: 74%, F: 84%, IF: 75% undetected — tệ hơn cả GPT và Gemini  
* Gemini-1.5-Pro: hiệu suất trung bình, tương đương với LLaMA ở một số tasks  
* LLaMA-3-70B: nhìn chung kém hơn GPT trong reference-less paradigms

### **Slide 2: Bất ngờ — LLaMA tốt nhất ở Reference-based (Table 4\)**

* LLaMA-3-70B-Instruct: chỉ 3% LF, 1% F, 5% IF undetected — **tốt nhất**  
* Nhưng lý do **không phải vì giỏi hơn** mà vì quá strict  
* Khi test với **score-invariant perturbations**: LLaMA chỉ cho điểm tuyệt đối 13% cases  
  * Nghĩa là nó hạ điểm cả những answer đúng khi so với reference  
* **Bài học:** số liệu tốt không có nghĩa là model evaluation tốt — cần nhìn toàn diện

### **Slide 3: Prometheus 2 — Trained Evaluator có tốt hơn không?**

* Prometheus 2 được **train chuyên biệt** cho task evaluation  
* Kỳ vọng: phải tốt hơn general LLMs  
* Thực tế: **tệ hơn** GPT-4-Turbo và hầu hết các model khác  
  * Reference (generic rubric): LF 51%, F 62%, IF 53% undetected  
* Kết luận: specialization trong training chưa đủ để giải quyết vấn đề này

### **Slide 4: Tổng kết — Blind Spots theo từng loại lỗi**

* **Dễ detect nhất:** Reasoning (calculations, wrong formula) — evaluator có thể verify toán học  
* **Khó detect nhất:**  
  * Comprehensiveness (LF): 91–100% undetected ở hầu hết strategies  
  * Chronology (LF): gần 100% undetected  
  * Assumptions (IF): 95–98% undetected  
  * Remove Fact (F): 94–99% undetected  
* **Pattern rõ ràng:** LLM evaluators kém nhất ở những lỗi **tinh tế, cần đọc kỹ toàn bộ context**

### **Slide 5: Kết luận và Implications**

* **Kết luận chính:** LLMs hiện tại **chưa đáng tin** làm evaluator cho text generation  
  * Bỏ sót \>50% lỗi trung bình, kể cả với model tốt nhất  
* **Khuyến nghị thực tế:**  
  * Nên dùng **reference-based** evaluation khi có thể  
  * Tránh dùng single-answer scoring với rubric phức tạp — không giúp ích  
  * Cẩn thận khi dùng kết quả leaderboard dựa trên LLM-as-judge  
* **Limitations của paper:**  
  * Chưa test multi-agent evaluation  
  * Chỉ dùng English prompts  
  * 22 perturbation categories chưa exhaustive  
* **Hướng tương lai:** mở rộng sang multilingual, tool use, planning tasks

