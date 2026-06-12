Evaluation Strategies

Link bài báo: https://aclanthology.org/2024.emnlp-main.911.pdf

Nghiên cứu tập trung đánh giá dựa trên 3 phương pháp:
Single-answer scoring: chấm điểm độc lập một câu trả lời duy nhất dựa trên tri thức sẵn có của mô hình, thay vì so sánh hay đối chiếu với đáp án mẫu
Pairwise comparison: Thay vì chấm điểm riêng lẻ, mô hình đánh giá (Evaluator LLM) sẽ được cung cấp hai câu trả lời cùng lúc và nhiệm vụ của nó là so sánh rồi chọn ra câu trả lời tốt hơn (hoặc xác định chúng ngang nhau).
Referenceguided evaluation

4.1. Single-answer scoring
Vanilla*: Người đánh giá chỉ nhận được câu hỏi mẫu và câu trả lời của mô hình, sau đó trực tiếp đưa ra điểm số.
Vanilla: Tương tự như Vanilla*, nhưng người đánh giá được yêu cầu phải đưa ra lời giải thích/lý giải trước khi đưa ra điểm số cuối cùng.
Rubric: Ngoài câu hỏi và câu trả lời, người đánh giá được cung cấp thêm một bảng tiêu chí chấm điểm (rubric) cụ thể để hướng dẫn cách chấm lỗi và phân bổ điểm.
Axis: Người đánh giá được yêu cầu tập trung kiểm tra câu trả lời theo một khía cạnh cụ thể phù hợp với loại câu hỏi
Axis+Rubric: Kết hợp cả việc chỉ định khía cạnh cần đánh giá (Axis) lẫn cung cấp các quy tắc chấm điểm chi tiết (Rubric) cho khía cạnh đó.

Ví dụ minh họa (Single-answer scoring):
- Câu hỏi: "Tóm tắt cuộc đời và sự nghiệp của Isaac Newton."
- Câu trả lời của mô hình: "Isaac Newton là nhà toán học, vật lý học và thiên văn học người Anh. Ông đã phát biểu các định luật về chuyển động và vạn vật hấp dẫn, phát minh ra phép tính vi tích phân, và có những đóng góp đột phá trong lĩnh vực quang học..."
- Vanilla* → Điểm: 9/10
- Vanilla → Giải thích: "Câu trả lời đã bao quát chính xác và đầy đủ các thành tựu chính của Newton. Có đề cập đến những đóng góp quan trọng trong vật lý, toán học và thiên văn học." → Điểm: 9/10
- Rubric → (tiêu chí: độ chính xác thực tế 40%, độ đầy đủ 30%, độ rõ ràng 20%, độ súc tích 10%) → "Độ chính xác: 9, Độ đầy đủ: 9, Độ rõ ràng: 9, Độ súc tích: 8" → Điểm: 8.8/10
- Axis → (Khía cạnh: độ chính xác thực tế) → "Câu trả lời nêu đúng các định luật chuyển động, lực hấp dẫn, phép tính vi tích phân và công trình quang học của Newton. Không phát hiện lỗi sai thực tế nào." → Điểm: 9/10
- Axis+Rubric → (Khía cạnh: độ chính xác thực tế + tiêu chí chi tiết) → "Theo tiêu chí độ chính xác: tất cả thông tin được xác minh, không có ảo giác, nguồn tham khảo hợp lý." → Điểm: 9/10

4.2. Pairwise comparison
Pairwise*: Người đánh giá chỉ nhận được câu hỏi và hai câu trả lời ($A_1, A_2$), sau đó trực tiếp đưa ra phán quyết (verdict) xem bên nào thắng mà không cần giải thích.
Pairwise: Tương tự như Pairwise*, nhưng người đánh giá bắt buộc phải đưa ra lý do/lời giải thích trước khi chốt phán quyết cuối cùng.
Rules: Ngoài câu hỏi và hai câu trả lời, người đánh giá được cung cấp thêm một bộ quy tắc ứng xử và tiêu chí so sánh chi tiết để hướng dẫn cách phân xử.
Axis: Yêu cầu người đánh giá so sánh hai câu trả lời dựa trên một khía cạnh cụ thể phù hợp với thể loại câu hỏi.
Axis+Rules (Kết hợp Trục & Quy tắc): Kết hợp việc chỉ định khía cạnh cần soi xét (Axis) cùng các quy tắc chấm điểm chi tiết (Rules) dành riêng cho khía cạnh đó.

Ví dụ minh họa (Pairwise comparison):
- Câu hỏi: "Tôi có thể làm gì trong 6 ngày ở Đà Lạt?"
- Câu trả lời A₁: "Đà Lạt là một điểm đến tuyệt vời. Bạn có thể tham quan Hồ Xuân Hương, Thung Lũng Tình Yêu và chợ đêm Đà Lạt. Thưởng thức ẩm thực địa phương."
- Câu trả lời A₂: "Đà Lạt có lịch trình 6 ngày phong phú: Ngày 1 — Khám phá Hồ Xuân Hương và dạo phố trung tâm. Ngày 2 — Tham quan Thung Lũng Tình Yêu và đồi Mộng Mơ. Ngày 3 — Ghé thăm Thiền Viện Trúc Lâm và hồ Tuyền Lâm. Ngày 4 — Khám phá làng hoa Vạn Thành và vườn dâu tây. Ngày 5 — Chinh phục đỉnh Langbiang. Ngày 6 — Thư giãn tại quán cà phê view đồi và tham quan chợ đêm."
- Pairwise* → Phán quyết: [[A₂]]
- Pairwise → Giải thích: "Câu trả lời A₂ đưa ra lịch trình chi tiết từng ngày với các địa điểm cụ thể, giúp người dùng dễ dàng lên kế hoạch hơn hẳn A₁." → Phán quyết: [[A₂]]
- Rules → (Quy tắc: độ liên quan 30%, chi tiết 30%, tính thực tiễn 20%, cấu trúc 20%) → "A₂ vượt trội về chi tiết và cấu trúc, A₁ ngắn gọn nhưng thiếu chiều sâu." → Phán quyết: [[A₂]]
- Axis → (Khía cạnh: tuân thủ yêu cầu) → "A₂ tuân thủ yêu cầu đầy đủ hơn khi cung cấp kế hoạch trọn vẹn 6 ngày. A₁ chỉ đưa ra các gợi ý chung chung." → Phán quyết: [[A₂]]
- Axis+Rules → (Khía cạnh: tuân thủ yêu cầu + quy tắc chi tiết) → "Theo quy tắc IF: A₂ đáp ứng đầy đủ yêu cầu 6 ngày với kế hoạch cụ thể từng ngày. A₁ không cấu trúc theo ngày." → Phán quyết: [[A₂]]

4.3. Reference-guided Single Answer Scoring
Cách thức hoạt động: Người đánh giá nhận được câu hỏi, đáp án mẫu chuẩn xác và câu trả lời cần chấm. Nhiệm vụ của nó là so sánh độ khớp, tính chính xác của câu trả lời dựa trên đáp án mẫu rồi đưa ra lời giải thích kèm điểm số.
Hạn chế trong thực tế: Nhóm tác giả lưu ý rằng phương pháp này không khả thi cho nhiều câu hỏi dạng mở (open-ended questions) vì câu hỏi mở không có một đáp án cố định hay duy nhất.

Ví dụ minh họa (Reference-guided):
- Câu hỏi: "Chức năng chính của tụ điện trong mạch điện là gì?"
- Đáp án mẫu: "Tụ điện lưu trữ năng lượng điện trong một điện trường. Nó gồm hai bản dẫn đặt cách nhau bởi một lớp điện môi. Trong mạch điện, tụ điện được dùng để lưu trữ năng lượng, lọc nhiễu và làm mịn dao động điện áp."
- Câu trả lời của mô hình: "Tụ điện dùng để lưu trữ năng lượng trong các mạch điện tử."
- Giải thích: "Câu trả lời đã nêu được chức năng cốt lõi (lưu trữ năng lượng) nhưng thiếu cơ chế hoạt động (điện trường), cấu tạo vật lý (hai bản dẫn + lớp điện môi) và các ứng dụng cụ thể (lọc nhiễu, làm mịn điện áp)." → Điểm: 5/10
- Lưu ý: Phương pháp này chỉ hiệu quả với câu hỏi có đáp án xác định (factual, reasoning). Với câu hỏi mở như "Tóm tắt cuộc đời và sự nghiệp của Isaac Newton" hay "Tôi có thể làm gì trong 6 ngày ở Đà Lạt?", không tồn tại một đáp án mẫu duy nhất nên không thể áp dụng reference-guided evaluation.
