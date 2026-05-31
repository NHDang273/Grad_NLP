# **P1 — Introduction & Motivation**

---

### **Slide 1: LLM-as-a-Judge là gì?**

* **Khái niệm cốt lõi (Concept):** LLM-as-a-Judge là phương pháp sử dụng một mô hình ngôn ngữ lớn (chẳng hạn như GPT-4, Gemini) đóng vai trò làm "trọng tài" hoặc "giám khảo" (evaluator/judge) để tự động đánh giá, chấm điểm hoặc so sánh chất lượng các câu trả lời do các LLM khác tạo ra. Khái niệm này ra đời nhằm thay thế hoặc bổ trợ cho việc đánh giá thủ công bởi con người (human evaluation) vốn vô cùng đắt đỏ và tốn thời gian.
* **Các ứng dụng thực tế phổ biến rộng rãi:**
  * **Chấm điểm Chatbot tự động:** So sánh trực tiếp chất lượng phản hồi giữa các chatbot khác nhau trên các tập benchmark hội thoại như MT-Bench (Zheng et al., 2023).
  * **Thiết lập và duy trì Leaderboard:** Xây dựng các bảng xếp hạng năng lực mô hình tự động và liên tục (như AlpacaEval hoặc Chatbot Arena tự động) phục vụ cộng đồng nghiên cứu (Watts et al., 2024; Zheng et al., 2023).
  * **Mô hình thưởng trong RLHF (Reward Modeling):** Đóng vai trò bộ lọc phản hồi hoặc mô hình chấm điểm chất lượng để tối ưu hóa mô hình bằng học tăng cường từ phản hồi con người.
  * **Đánh giá Benchmark quy mô lớn:** Thực hiện chấm điểm hàng ngàn instances trên các tập dữ liệu đa dạng mà không thể thuê con người dán nhãn xuể (Dubois et al., 2023).
* **So sánh toàn diện: Con người (Human Evaluators) vs. LLM-as-a-Judge:**

  | Tiêu chí so sánh | Con người (Human Evaluators) | Giám khảo LLM (LLM-as-a-Judge) | Diễn giải & Insight |
  | :--- | :--- | :--- | :--- |
  | **Tốc độ (Speed)** | Rất chậm (mất nhiều ngày/tuần) | Gần như tức thời (vài phút/giờ) | LLM tối ưu hóa chu kỳ phát triển mô hình cực nhanh. |
  | **Chi phí (Cost)** | Rất đắt đỏ (nhân công dán nhãn) | Cực kỳ rẻ (phí API hoặc tài nguyên GPU) | Giảm chi phí đánh giá xuống hàng chục, hàng trăm lần. |
  | **Khả năng mở rộng (Scalability)** | Kém (giới hạn bởi nhân lực) | Vô hạn (dễ dàng chạy song song) | Khả năng mở rộng quy mô đánh giá lên hàng triệu samples. |
  | **Tính nhất quán (Consistency)** | Thấp (giữa các annotator khác nhau) | Rất cao (deterministic nếu Temp = 0) | LLM loại bỏ sự chủ quan không nhất quán giữa các cá nhân. |
  | **Độ tin cậy (Reliability)** | **Độ chuẩn xác cao (Gold Standard)** | **Chưa được kiểm chứng chi tiết** | **Mặc dù LLM có ưu thế vượt trội về hiệu năng vận hành, nhưng tính đúng đắn và độ tin cậy của nó vẫn là một dấu hỏi lớn.** |

* **Cơ sở khoa học & Sự bùng nổ của phương pháp:**
  * Các nghiên cứu nền tảng (Kim et al., 2023, 2024a; Chiang and Lee, 2023; Chen et al., 2023) đã chứng minh rằng việc sử dụng LLMs làm evaluator giúp **tiết kiệm từ 90% đến 99% chi phí và thời gian** so với việc thuê chuyên gia con người.
  * Các nghiên cứu ban đầu về độ tương quan (Dubois et al., 2023; Zheng et al., 2023) công bố rằng LLM judge đạt được **độ tương quan rất cao (strong correlation) với đánh giá của con người** (đạt trên 80% độ đồng thuận trên các task thông thường). Điều này tạo ra một làn sóng tin tưởng tuyệt đối vào LLM-as-a-Judge trong cộng đồng NLP.

---

### **Slide 2: Tại sao đây là vấn đề?**

* **Hệ quả nghiêm trọng của một Giám khảo không đáng tin:**
  * Nếu mô hình evaluator LLM có những "điểm mù" ngầm mà không bị phát hiện, toàn bộ thứ hạng trên các bảng xếp hạng (leaderboards) sẽ bị bóp méo. Các nhà phát triển sẽ dựa vào các kết luận sai lệch này để đưa ra quyết định tối ưu hóa sai hướng, dẫn đến việc **triển khai (deployment) các mô hình kém chất lượng hơn ra thực tế** nhưng lại có điểm số evaluator ảo cao hơn.
* **Hạn chế cốt lõi của phương pháp đánh giá hiện tại (The Correlation Trap):**
  * Hầu hết các nghiên cứu hiện nay chỉ đánh giá độ tin cậy của LLM judge bằng cách **đo hệ số tương quan tổng thể (overall correlation)** (như Pearson hoặc Spearman correlation) với con người trên các tập dữ liệu thô (Zheng et al., 2023).
  * **Hạn chế:** Chỉ số tương quan tổng thể này **không đủ độ chi tiết (not granular)**. Nó che giấu việc LLM evaluator thực sự hiểu lỗi hay chỉ đơn thuần là chấm điểm trùng hợp ngẫu nhiên. Nó không thể trả lời câu hỏi: *LLM có thực sự nhạy bén để phát hiện ra từng loại lỗi cụ thể hay không?* (Zeng et al., 2023).
* **Các định kiến (Biases) đã biết của LLM Evaluators:**

  | Loại định kiến (Bias) | Mô tả cơ chế ảnh hưởng | Tài liệu dẫn nguồn (Citations) |
  | :--- | :--- | :--- |
  | **Định kiến Vị trí (Position Bias)** | LLM judge có xu hướng chọn câu trả lời xuất hiện ở vị trí đầu tiên (hoặc thứ hai) khi so sánh pairwise, bất kể chất lượng thực tế. | Zheng et al. (2023); Wang et al. (2023c) |
  | **Định kiến Độ dài (Verbosity Bias)** | LLM judge bị thu hút và chấm điểm cao hơn cho các câu trả lời dài dòng, chi tiết, đầy đủ chữ, ngay cả khi nó chứa thông tin sai lệch hoặc không chính xác. | Wu and Aji (2023); Zeng et al. (2023) |
  | **Tự thiên vị (Self-preference Bias)** | LLM judge có xu hướng đánh giá cao và ưu ái các câu trả lời do chính nó (hoặc các mô hình cùng họ/cùng nhà phát triển) sinh ra. | Panickssery et al. (2024); Liu et al. (2023) |

* **Khoảng trống nghiên cứu khổng lồ (The Research Gap):**
  * **Chưa có một nghiên cứu nào kiểm tra khả năng đánh giá ở mức độ chi tiết (fine-grained capability)** của LLMs đối với từng nhóm lỗi cụ thể (như lỗi ngữ pháp, lỗi thực tế, lỗi tính toán toán học). Tất cả chỉ dừng lại ở việc chấm điểm chung chung. Cộng đồng NLP đang thiếu một **"interpretable checklist"** để chẩn đoán chính xác các điểm mù (blind spots) của các LLM giám khảo trước khi chính thức giao phó quyền đánh giá cho chúng. **FBI framework ra đời chính là để lấp đầy khoảng trống nghiên cứu sống còn này.**

---

### **Slide 3: Câu hỏi nghiên cứu (Research Questions)**

* **Câu hỏi nghiên cứu trọng tâm (Core RQ):**
  * *"Liệu các LLM Evaluators hiện tại có thực sự là một judge đáng tin cậy không? Chúng có thể đánh giá và phát hiện một cách chi tiết (fine-grained) các loại lỗi khác nhau ảnh hưởng trực tiếp đến 4 năng lực cốt lõi: factual accuracy, reasoning, instruction following, và long-form writing trong các văn bản sinh ra không?"*
* **Breakdown thành 4 câu hỏi nghiên cứu thành phần (Sub-questions) kèm ví dụ minh họa:**
  * **RQ1 — Khả năng đánh giá Văn bản dài (Long-form Writing - LF):** Evaluator có phát hiện ra sự thiếu mạch lạc (coherence), lỗi ngữ pháp (grammar), lỗi chính tả (spelling), hay trình tự thời gian bị xáo trộn (chronology) trong các văn bản viết dài không?
    * *Ví dụ cụ thể từ paper:* Trong bài viết dài về quản lý thời gian, nếu ta cố tình sửa lỗi ngữ pháp từ *"These strategies are..."* thành *"These strategies is..."* (lỗi Grammar), LLM judge có nhận ra để hạ điểm không?
  * **RQ2 — Khả năng đánh giá Độ chính xác Thực tế (Factual Accuracy - F):** Evaluator có thể phát hiện các lỗi thực tế cụ thể như sai lệch thực thể (entity error), sai lệch số liệu (number error), hoặc đưa ra sự thật mâu thuẫn hoàn toàn (opposite fact) không?
    * *Ví dụ cụ thể từ paper:* Trong câu hỏi lịch sử, nếu ta thay đổi thông tin thực tế từ *"Ba Lan nằm ở châu Âu"* thành *"Ba Lan nằm ở châu Á"* (lỗi Opposite Fact), liệu evaluator có phát hiện và trừ điểm thích đáng?
  * **RQ3 — Khả năng đánh giá Tuân thủ Chỉ dẫn (Instruction Following - IF):** Evaluator có nhận diện được việc câu trả lời vi phạm hoặc bỏ qua các chỉ dẫn định dạng phức tạp (như giới hạn số dòng, cấu trúc câu, hoặc giả định bổ sung) không?
    * *Ví dụ cụ thể từ paper:* Khi yêu cầu *"viết bài thơ đúng 4 dòng chứa các từ: peace, sky, race, ground"*, nếu mô hình sinh ra bài thơ 5 dòng (lỗi Do More), liệu evaluator có bắt lỗi vi phạm định dạng này hay vẫn cho điểm tối đa?
  * **RQ4 — Khả năng đánh giá Lập luận Toán học/Logic (Reasoning - R):** Evaluator có phát hiện được các lỗi logic, sai sót tính toán (calculations), hoặc áp dụng sai công thức (wrong formula) trong các bài giải toán không?
    * *Ví dụ cụ thể từ paper:* Trong bài giải toán logic, nếu câu trả lời thực hiện phép cộng sai *"2 + 3 = 6"* (lỗi Calculation) nhưng vẫn đưa ra kết luận rất mạch lạc, liệu evaluator có bị lừa bởi văn phong trôi chảy mà bỏ qua lỗi tính toán cơ bản?
* **Tại sao phương pháp Perturbation-based lại khác biệt và mạnh mẽ hơn Correlation-based?**
  * **Correlation-based (Chấm điểm tương quan truyền thống):** Đo lường thụ động trên dữ liệu sinh ra tự nhiên. Điều này rất dễ bị nhiễu bởi các yếu tố gây nhiễu (như độ dài câu trả lời). Nếu cả người và LLM cùng chấm điểm cao cho một câu trả lời dài, hệ số tương quan sẽ cao, nhưng không chứng minh được LLM thực sự hiểu lỗi.
  * **Perturbation-based (FBI Framework):** Can thiệp chủ động bằng cách **cố tình chèn lỗi có kiểm soát** vào câu trả lời gold standard. Phương pháp này giúp **cô lập hoàn toàn các biến số**. Chúng ta biết chính xác lỗi nào được đưa vào ở vị trí nào. Từ đó, chúng ta đo lường trực tiếp sự nhạy cảm của mô hình evaluator (nếu điểm số giảm -> evaluator phát hiện lỗi; nếu điểm số giữ nguyên -> đó là điểm mù thực sự).

---

### **Slide 4: Giới thiệu FBI Framework**

* **Giải thích ý nghĩa tên gọi Framework:**
  * **FBI** là viết tắt của **Find Blind spots in evaluator LLMs using Interpretable checklists** (Tìm kiếm Điểm mù của Giám khảo LLM bằng Checklist Diễn giải được).
  * Đây là một framework meta-evaluation có hệ thống, được thiết kế để "đánh giá chất lượng của chính các mô hình đánh giá".
* **Ý tưởng cốt lõi & Logic hoạt động của FBI:**
  * FBI bắt đầu bằng các cặp câu hỏi (prompts) và câu trả lời chuẩn hoàn hảo (gold answers) được sinh bởi GPT-4-Turbo.
  * Sau đó, framework chèn các biến đổi lỗi chủ đích (**targeted perturbations**) để làm suy giảm chất lượng câu trả lời theo đúng một khía cạnh lỗi duy nhất.
  * **Logic chấm điểm của Judge:**
    $$\text{Evaluator tốt} \implies \text{Score}(A_{\text{perturb}}) < \text{Score}(A_{\text{gold}}) \quad (\text{Hạ điểm do phát hiện lỗi})$$
    $$\text{Evaluator có điểm mù (Blind Spot)} \implies \text{Score}(A_{\text{perturb}}) \approx \text{Score}(A_{\text{gold}}) \quad (\text{Không phát hiện, giữ nguyên điểm cao})$$
* **Chi tiết cấu trúc và Quy mô thực nghiệm của FBI:**
  * **4 Task Abilities cốt lõi được đánh giá:** Factual (Factual accuracy), Reasoning (Reasoning proficiency), Instruction Following (Adherence to constraints), và Long-form Writing (Fluent & Coherent writing).
  * **22 loại lỗi chi tiết (Perturbation Categories):** Bao phủ toàn bộ các dạng lỗi phổ biến nhất trong thực tế của LLM (Table 2), từ lỗi chính tả nhỏ cho đến lỗi sai công thức hay giả định logic phức tạp.
  * **Quy mô 2400 Perturbations nghiêm ngặt:** Toàn bộ dữ liệu được sinh tự động bằng GPT-4-Turbo ban đầu, sau đó được **17 sinh viên sau đại học (graduate students) có chuyên môn sâu về NLP kiểm duyệt thủ công từng dòng một (Human-in-the-loop)** nhằm đảm bảo tính chuẩn xác tuyệt đối của từng loại lỗi.
  * **3 Paradigms đánh giá thực tế:**
    1. *Single-answer scoring:* Giám khảo tự chấm điểm 1 câu trả lời dựa trên tri thức có sẵn.
    2. *Pairwise comparison:* Giám khảo so sánh trực tiếp và chọn ra câu trả lời tốt hơn giữa 2 phương án.
    3. *Reference-guided scoring:* Giám khảo chấm điểm câu trả lời khi có sẵn đáp án chuẩn để đối chiếu.
  * **Đánh giá đa mô hình:** Thử nghiệm toàn diện trên 5 mô hình evaluator phổ biến nhất hiện nay: GPT-4-Turbo, Gemini-1.5-Pro, Claude-3-Opus, LLaMA-3-70B-Instruct, và Prometheus 2 (mô hình chuyên biệt cho evaluation).

---

### **Tóm tắt P1 — 3 Takeaways chính**

| # | Takeaway quan trọng | Minh chứng khoa học & Cơ sở thực tiễn |
|---|---|---|
| **1** | **LLM-as-a-Judge bùng nổ nhưng thiếu sự kiểm chứng chi tiết** | Giúp giảm 99% chi phí và đạt độ tương quan tổng thể cao với con người, nhưng các kiểm tra chi tiết (fine-grained) về khả năng bắt lỗi thực tế vẫn chưa từng được thực hiện trước đây. |
| **2** | **Độ tương quan tổng thể (Overall Correlation) là một cạm bẫy** | Hệ số tương quan cao che giấu các định kiến nặng nề (position, verbosity, self-preference biases), khiến evaluator dễ dàng bỏ qua các lỗi nghiêm trọng nếu văn phong trôi chảy. |
| **3** | **FBI Framework là giải pháp chẩn đoán điểm mù toàn diện** | Bằng phương pháp perturbation-based với 2400 lỗi được con người xác thực trên 22 danh mục lỗi cụ thể, FBI cung cấp một checklist rõ ràng để lật tẩy chính xác các giới hạn năng lực của LLM evaluators. |
