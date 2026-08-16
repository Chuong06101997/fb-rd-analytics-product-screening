# F&B R&D ANALYTICS — DECISION LOG

*Extract từ Full Report (Phase 0-9 + Final Memo), tổng hợp toàn bộ quyết định phân tích có tính rẽ nhánh xuyên suốt Phase 0-6. Không có phân tích mới, không số liệu mới — chỉ đóng gói lại nội dung đã audit.*

| Decision | Rationale | Alternative rejected | Phase |
|---|---|---|---|
| Coi 10,000 dòng là toàn bộ scope, không ngoại suy 500K+ | RecipeId range 70–541,366 khớp full dataset nhưng gap giữa các ID không đều → "consistent với random sampling", song "không xác nhận được phương pháp sampling cụ thể" | Ngoại suy kết luận sang toàn bộ ~500K+ recipe của Food.com dataset gốc | 0 |
| Không lọc theo thời gian (giữ nguyên 1999–2020); RecipeAge → covariate | r(age,ReviewCount)=0.25 thật (ảnh hưởng volume), r(age,Rating)=-0.057 yếu → "outcome đang xét là AggregatedRating... confound qua age là yếu" | Lọc/giới hạn phạm vi thời gian phân tích để loại time-exposure confound | 2 |
| ReviewCount≥3 làm primary threshold | "giữ N=2,197 (42.5% tập có rating) và vẫn giữ được 19 category có N≥30" | Threshold ≥5 (N=1,242) hoặc ≥10 (N=522) làm primary | 2 |
| Round 1 chỉ test main effect riêng lẻ, không test combination | "'certain combinations' là exploratory search... với N khả dụng thực tế, dò combination sẽ rơi vào cell rất nhỏ, rủi ro fishing/multiple-testing cao" | Test combination/interaction attribute ngay từ round 1 (theo đúng nghĩa đen câu hỏi gốc) | 1 |
| Giới hạn phân tích vào 19-category N≥30, category còn lại = "insufficient sample" | "chỉ 19/221 category có N≥30 (đủ để so sánh within-category có ý nghĩa)" | Đưa toàn bộ 221 (hoặc 152) category vào phân tích, kể cả category N<30 | 1–2 |
| Loại nhóm thiếu RecipeServings khỏi nutrition-analysis | "36/39 (92%) recipe Calories>3,000 đều thiếu RecipeServings; p95 gap 2,752 vs 793 — chênh lệch hệ thống, không ngẫu nhiên" | Cắt outlier theo percentile tùy tiện (arbitrary percentile cutoff) | 5 |
| Không dùng nutrition composite; test 3 dimension riêng (Sugar/Sodium/SaturatedFat) | "Composite đòi hỏi chọn trọng số — chưa có cơ sở validate, tự nó là proxy chưa audit" | Composite Health Score = f(Calories, Fat, Sodium, Sugar, Protein, Fiber) | 5 |
| Chọn Mann-Whitney U / Kruskal-Wallis thay vì t-test/ANOVA/OLS | "AggregatedRating skew=-2.52, 71.1%=5.0 (ceiling effect) → loại t-test/ANOVA/OLS thường (giả định normal residual vi phạm rõ ràng)" | t-test/ANOVA/OLS thông thường (giả định normal distribution) | 5 |
| Robustness-matrix đầy đủ (RC≥5/10, loại top-author) chỉ chạy cho Category-effect | "test duy nhất có 1 threshold nominally significant (p=0.049 ở RC≥5)... quyết định phân bổ effort theo trọng số finding" | Chạy robustness-matrix đầy đủ như nhau cho cả 5 test (Category, PrepTime, Sugar, Sodium, SaturatedFat) | 6–7 |
| Combine p-value qua category bằng Fisher's method (không dùng Stouffer's Z) cho Prep-time test | "không dùng Stouffer's Z vì KW không có hướng tự nhiên để combine qua Z-score" | Stouffer's Z-combination | 6 (Phụ lục Reproducibility) |
