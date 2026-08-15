# Summary — F&B R&D Analytics: Recipe Attributes & Consumer Evaluation

## 1. Business question & câu trả lời

**Câu hỏi gốc:** Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others.

**Answerability:** Weakly answerable.

Round 1 (test main effect riêng lẻ — category, prep-time bucket, 3 nutrition dimension) cho **Insufficient Evidence trên cả 3 nhóm attribute**. Không có attribute nào vượt ngưỡng "Association", chưa nói đến "Robust Evidence". Phần "combination" của câu hỏi gốc chưa từng được test. Đây là kết quả null đạt được qua quy trình kiểm định có phương pháp (stratified Mann-Whitney U/Kruskal-Wallis, phần lớn có robustness check, Benjamini-Hochberg correction) — không phải phân tích thất bại hay bỏ dở giữa chừng.

**Lưu ý diễn giải quan trọng**: "Insufficient Evidence" nghĩa là chưa đủ dữ liệu để phát hiện effect, KHÔNG đồng nghĩa với việc đã chứng minh các attribute này không ảnh hưởng đến rating.

## 2. 3 giới hạn chính

- **Ceiling effect ở outcome**: `AggregatedRating` lệch trái cực mạnh (skew=-2.52); 71.1% recipe có Rating=5.0 tuyệt đối, 97.5% ≥4.0.
- **Selection bias**: 48.3% recipe (4,827/10,000) không có `AggregatedRating` — nhóm chưa từng được ai rate, không phải random missing.
- **N-shrinkage qua các tầng lọc**: 1,429/10,000 (14.3%) cho category-effect, 1,323/10,000 (13.2%) cho prep-time-effect, 898/10,000 (8.98%) cho nutrition-effect.

## 3. Khuyến nghị

**No Action** trên finding hiện tại — không đủ bằng chứng để đầu tư consumer research/sensory test/prototype cho bất kỳ attribute nào đã test. Nếu muốn theo đuổi tiếp câu hỏi gốc: **Find More Data** — dataset có rating scale không bị ceiling và/hoặc N lớn hơn ở nhóm recipe đủ review.

## 4. Hướng dẫn đọc 4 file kèm theo

| File | Nội dung | Đọc khi nào |
|---|---|---|
| Full Report | Phase 0-9 đầy đủ, mọi số liệu/lý do | Muốn hiểu chi tiết 1 phase cụ thể |
| Notebook | Code thực thi, đã verify chạy khớp trên Colab | Muốn tự chạy lại/verify số liệu |
| Decision Log | 10 quyết định rẽ nhánh + lý do + alternative bị loại | Muốn hiểu tại sao chọn phương pháp X thay vì Y |
| Assumption Register | 5 giả định bắt buộc + rủi ro nếu sai | Muốn đánh giá độ tin cậy/rủi ro của kết luận |
