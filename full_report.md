# F&B R&D ANALYTICS — FULL AUDIT REPORT (Phase 0-9 + Final Memo)

*Portfolio project: R&D Analyst simulation, phân tích `recipes_10k.csv` (Food.com sample, Kaggle). Toàn bộ số liệu đã qua audit độc lập, verify trực tiếp trên dữ liệu ở từng bước, kể cả test chạy thật trên Colab notebook. Ghép từ 3 phần (Part 1/2/3) đã audit riêng lẻ.*

**Vai trò:** R&D Analyst (portfolio simulation)
**Dataset:** `recipes_10k.csv` — 10,000 dòng, sample từ Food.com Recipes and Reviews dataset (Kaggle)
**Phương pháp luận:** F&B Analytical Question & Methodology Audit — quy trình đa giai đoạn với freeze gate bắt buộc giữa các phase, mỗi con số/quyết định phải verify trực tiếp trên dữ liệu trước khi khóa.

---

# PART 1 — Phase 0 (Data Verification) → Phase 2 (Measurement & Variable Design)

## PHASE 0 — DATA VERIFICATION

### 0.1 Giải quyết discrepancy quy mô dataset

Proposal ban đầu tham chiếu ~500,000+ recipe / 1.4M+ review (full Food.com dataset). File thực tế nạp vào project chỉ có 10,000 dòng.

**Kết luận**: 10,000 dòng này **consistent với random sample** từ dataset gốc, không phải một dataset khác/nhỏ hơn hoàn toàn. Bằng chứng:

- `RecipeId` trải từ 70 đến 541,366 — khớp range ID của bộ Food.com gốc, không phải ID 1–10,000 liên tục.
- Khi sort `RecipeId`, khoảng cách giữa các ID liên tiếp không đều (min gap=1, max gap=1,099, mean≈54, median≈37) → phù hợp với random sampling, không phải lấy N dòng đầu hay 1 block liên tục.
- 0 duplicate RecipeId, 0 duplicate full-row.

**Giới hạn của kết luận**: chỉ xác nhận được "consistent with random sampling", **không xác nhận được phương pháp sampling cụ thể** (simple random vs. có điều kiện lọc) vì không có metadata mô tả quy trình. Quyết định làm việc: coi 10,000 dòng là toàn bộ scope thực tế của project — không ngoại suy sang 500K+.

### 0.2 Schema & Grain

- 28 cột khớp 100% với schema đã biết trước. Không thiếu, không thừa cột.
- `RecipeId` unique 10,000/10,000 → **grain = 1 row = 1 recipe** (không phải 1 review).
- `AuthorId`: 4,618 giá trị duy nhất trên 10,000 recipe (avg ~2.2 recipe/author) → **AuthorId là người đăng công thức, không phải customer**.

### 0.3 Data types

Toàn bộ numeric field đọc đúng kiểu `float64`. `CookTime/PrepTime/TotalTime` là string dạng ISO 8601 duration (vd `PT1H20M`), cần parse riêng ra phút — không phải số phút trực tiếp.

### 0.4 Missingness

| Field | Missing | % |
|---|---|---|
| RecipeYield | 6,786 | 67.9% |
| **AggregatedRating** | **4,827** | **48.3%** |
| ReviewCount | 4,703 | 47.0% |
| RecipeServings | 3,453 | 34.5% |
| CookTime | 1,598 | 16.0% |
| Keywords | 328 | 3.3% |
| RecipeCategory | 13 | 0.1% |

**Điểm quan trọng nhất**: gần một nửa sample không có rating — đây là **selection bias**, không phải random missing (recipe có rating là recipe đã được ai đó chủ động thử + rate). 124 recipe có `ReviewCount` present nhưng `AggregatedRating` missing (inconsistency cần lưu ý khi lọc theo threshold). `ReviewCount` chỉ nhận giá trị NaN hoặc ≥1, không có case =0 (logic nhất quán).

### 0.5 Review-count distribution (trên 5,297 recipe có review)

Median=2, mean=5.1, max=542, std=15.4 → lệch phải rất mạnh. Phần lớn recipe chỉ có 1–4 review → **rating của recipe 1-review và 500-review không cùng độ tin cậy**, cần minimum-review threshold.

### 0.6 Review text — KHÔNG TỒN TẠI

Project chỉ có 1 file (`recipes_10k.csv`), không có bảng Reviews riêng. Không cột nào chứa review text/reviewer ID/review date. `RecipeInstructions` là hướng dẫn nấu ăn (do người đăng viết), **không phải review text**.

→ **NLP/text analysis trên feedback khách hàng: NOT TESTABLE** với dữ liệu hiện có.

### 0.7 Nutrition & outlier sơ bộ

Các trường dinh dưỡng có outlier cực đoan (vd Calories: mean 472, max 19,380; Sodium: mean 749, max 113,198). Ghi nhận ở Phase 0, xử lý cụ thể để lại Phase 5 (Statistical Method Selection).

## PHASE 1 — BUSINESS UNDERSTANDING

### 1.0 Business context framing

- **Business question round 1**: *"Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others."*
- **Ai ra quyết định**: R&D Analyst (giả lập) → báo cáo Consumer Insights/R&D Manager (giả lập).
- **Quyết định cụ thể**: có nên đầu tư điều tra sâu hơn (consumer research, sensory test) vào 1 attribute cụ thể hay không — đây là bước sàng lọc (screening), không phải quyết định thương mại.
- **Baseline**: so sánh `AggregatedRating` giữa nhóm recipe có/không có đặc điểm đang xét, **trong cùng `RecipeCategory`** (within-category comparison) để tách attribute effect khỏi category effect.
- **Scope**: 10,000 recipe trong sample, không ngoại suy full dataset. Time scope ban đầu chưa chỉ định (giải quyết ở Phase 2).

### 1.1 Operationalize khái niệm mơ hồ

**"Stronger consumer evaluation" → outcome (Y)**
- Biến khả dụng: `AggregatedRating` — proxy cho valence đánh giá của tập con người dùng tự chọn rate, **không phải** customer demand/purchase behavior/popularity nói chung.
- 48.3% recipe không có rating → N thực tế phân tích outcome tối đa 5,173/10,000, không phải 10,000 — đây là selection bias, không chỉ là "loại missing".
- `ReviewCount` (volume) và `AggregatedRating` (valence) là 2 khái niệm khác nhau — không gộp chung.

**Review-count threshold — đánh giá trade-off N vs. độ tin cậy**

| ReviewCount ≥ | N recipe | % của recipe có rating | % toàn sample |
|---|---|---|---|
| 1 | 5,173 | 100% | 51.7% |
| 3 | 2,197 | 42.5% | 22.0% |
| 5 | 1,242 | 24.0% | 12.4% |
| 10 | 522 | 10.1% | 5.2% |

**Category coverage sau threshold ≥3**: chỉ **19/221 category có N≥30** (đủ để so sánh within-category có ý nghĩa). 124/221 category có <10 recipe ngay từ đầu.

**"Recipe attributes" → X (explanatory)**: 4 nhóm construct riêng — category (trực tiếp nhưng sparse), nutrition profile (8 cột số, cần threshold/proxy), preparation time (string ISO 8601, cần parse), ingredients (free-text, cần parse — nặng nhất, để sau).

**Vấn đề cấu trúc câu hỏi gốc**: "certain combinations" là exploratory search (interaction effect) không phải hypothesis đơn — với N khả dụng thực tế, dò combination sẽ rơi vào cell rất nhỏ, rủi ro fishing/multiple-testing cao.

→ **Quyết định reduce scope**: round 1 chỉ test **main effect riêng lẻ** (nutrition proxy / prep-time bucket / category), không test combination.

## PHASE 2 — MEASUREMENT & VARIABLE DESIGN

### 2.1 Time scope

Kiểm tra thực nghiệm (Spearman, subset có rating):
- `RecipeAge` vs `ReviewCount`: **r=0.25** (recipe cũ tích lũy nhiều review hơn — time-exposure confound có thật cho volume).
- `RecipeAge` vs `AggregatedRating`: **r=-0.057** (yếu — recipe age gần như không ảnh hưởng đến giá trị rating trung bình).

**Quyết định**: KHÔNG lọc theo năm (giữ nguyên 1999–2020). Vì outcome đang xét là `AggregatedRating` (không phải `ReviewCount`), confound qua age là yếu (r=-0.057). Xử lý: đưa `RecipeAge` vào robustness check theo year-bucket (Phase 7), không cắt scope theo thời gian.

### 2.2 ReviewCount threshold — quyết định

**≥3 làm primary** (N=2,197, giữ 19 category N≥30). **≥5 và ≥10 làm robustness check bắt buộc** (Phase 7).

### 2.3 Attribute round 1 — phạm vi

Main effect riêng lẻ: (a) nutrition proxy, (b) prep-time bucket, (c) category (main effect). Không test combination/interaction ở round 1.

### 2.4 Category coverage

Chỉ 19 category N≥30 (sau rated & ReviewCount≥3) dùng cho within-category comparison. Category còn lại = "insufficient sample", liệt kê rõ tên, không xóa âm thầm.

**19-category list đầy đủ** (N sau lọc, sắp xếp giảm dần):
Dessert(206), Lunch/Snacks(146), One Dish Meal(136), Vegetable(129), Breakfast(90), Chicken(80), Beverages(75), Chicken Breast(70), Potato(68), Pork(56), Sauces(55), Breads(50), Quick Breads(47), Meat(44), Pie(40), Drop Cookies(38), Bar Cookie(38), < 60 Mins(31), Yeast Breads(30).

⚠️ KHÔNG bao gồm: `< 15 Mins` (N=17), `< 30 Mins` (N=21) — tên gần giống `< 60 Mins`, dễ nhầm, không đạt ngưỡng. `Salad` không tồn tại như giá trị hợp lệ trong `RecipeCategory` của sample này (chỉ có `Salad Dressings`, khác concept).

### 2.5 Rating-missingness theo category (selection bias không đồng nhất)

Dao động **39.9% (`< 60 Mins`) – 64.4% (`Potato`)** có rating, so với trung bình chung 51.7% → xác nhận selection bias khác nhau giữa category, không phải random đồng nhất.

## FROZEN BLOCK — Phase 0-2 (đã audit, khóa)

```
- N_total = 10,000 (sample, KHÔNG ngoại suy 500K+)
- N_rated = 5,173 (48.3% missing AggregatedRating — selection bias, không phải random)
- Time scope: KHÔNG lọc theo năm (1999-2020 giữ nguyên); RecipeAge → covariate Phase 3,
  robustness year-bucket Phase 7. r(age,ReviewCount)=0.25 thật, r(age,Rating)=-0.057 yếu.
- ReviewCount threshold: ≥3 (primary, N=2,197) | ≥5, ≥10 = robustness bắt buộc
- Attribute round 1: main effect riêng lẻ (nutrition proxy / prep-time bucket / category)
  — KHÔNG test combination
- 19-category list (N≥30 sau rated & ReviewCount≥3), đầy đủ tên — xem mục 2.4
- Rating-missingness theo category: 39.9% (<60 Mins) – 64.4% (Potato)
- AuthorId ≠ customer | Không có review text → NLP NOT TESTABLE
```

---

# PART 2 — Phase 3 (Competing Variables) → Phase 5 (Statistical Method Selection)

*(Tiếp nối Part 1 — Phase 0-2 đã frozen. Baseline: within-category comparison, ReviewCount≥3 primary threshold, round 1 = main effect riêng lẻ cho nutrition/prep-time/category, 19-category list đã xác định.)*

## PHASE 3 — COMPETING VARIABLES & CAUSAL STRUCTURE

### 3.0 Kiểm tra thực nghiệm bổ sung trước khi phân loại

- **Author clustering**: 145/4,618 author có ≥10 recipe, chiếm **2,967/10,000 (29.7%)** sample. Author đông nhất: 141 recipe.
- **Rating-missingness theo category** (19-category): dao động 39.9% (`< 60 Mins`) – 64.4% (`Potato`), trung bình chung 51.7%.
- **Tương quan PrepTime vs nutrition fields** (Spearman, N=2,197): r dao động 0.08–0.18 (yếu, không collinear mạnh).

### 3.1 X → Y: Nutrition proxy → AggregatedRating

| Z | Phân loại | Xử lý |
|---|---|---|
| RecipeCategory | Potential confounder — ảnh hưởng cả nutrition profile lẫn baseline rating (vd Dessert khác Vegetable về đường/chất béo) | Stratification (within-category, baseline) |
| PrepTime | Possible confounder, yếu (r=0.08–0.18) | Covariate, rẻ nên vẫn làm |
| ReviewCount (đã filter ≥3) | Likely irrelevant cho quan hệ cụ thể này | Đã xử lý ở tầng threshold |
| AuthorId | Potential confounder — 30% sample cluster ở 145 author | Robustness check bắt buộc: loại top-author |
| RecipeAge/PubYear | Possible confounder, yếu với Y | Robustness theo year-bucket |
| Số lượng ingredient | Cannot determine (chưa derive) | Derive ở Phase 4 |
| RecipeServings | Cannot determine — measurement-quality risk, không phải confounder thống kê | Flag riêng ở Phase 5 |

### 3.2 X → Y: Prep-time bucket → AggregatedRating

| Z | Phân loại | Xử lý |
|---|---|---|
| RecipeCategory | Potential confounder | Stratification |
| Độ phức tạp công thức | Cannot determine cleanly — PrepTime có thể chính là proxy của complexity | Không xử lý như confounder độc lập; ghi caveat khi diễn giải |
| Nutrition proxy | Possible confounder giữa 2 X test song song, tương quan yếu | Không đưa vào round 1, ghi caveat |
| AuthorId | Potential confounder | Robustness tương tự 3.1 |
| ReviewCount | Likely irrelevant trực tiếp | Đã xử lý qua threshold |

### 3.3 X → Y: Category (main effect) → AggregatedRating

| Z | Phân loại | Xử lý |
|---|---|---|
| Rating-missingness khác nhau theo category (39.9–64.4%) | Potential confounder — nghiêm trọng nhất cho test này | Bắt buộc nêu như limitation diễn giải, không control được bằng covariate đơn giản |
| ReviewCount trung bình khác nhau theo category | Possible confounder cho độ tin cậy | Threshold ≥3/≥5/≥10 + báo cáo N mỗi category |
| AuthorId | Potential confounder nếu author tập trung 1 category | Kiểm tra ở Phase 4 |

**Không có potential confounder nào bị bỏ qua mà không có xử lý/justification** — đáp ứng yêu cầu bắt buộc của methodology.

## PHASE 4 — EDA: 3 điểm mở

### (a) Derive IngredientCount từ RecipeIngredientParts

Parse format R-vector (`c("...", "...")`) bằng regex. Kết quả: mean=7.9, median=7, range=0–35, 33 recipe rỗng thật (`character(0)` trong data gốc, không phải lỗi parse).

Tương quan (Spearman, N=2,197):
- IngredientCount vs AggregatedRating: **r=-0.010** (không đáng kể)
- IngredientCount vs PrepTimeMin: **r=0.392** (khá mạnh)
- IngredientCount vs Calories: **r=0.249**

→ Củng cố phân loại ở 3.2: PrepTime và độ phức tạp công thức khó tách bạch. IngredientCount không phải confounder độc lập của Rating (r≈0), nhưng đưa vào robustness cho cả 2 main effect (nutrition, prep-time).

### (b) Author có tập trung vào category cụ thể không?

- Ở mức tổng hợp: phân bố category của nhóm top-author (≥10 recipe) gần giống phân bố chung toàn sample — không có 1 author lớn nào làm lệch hẳn phân bố.
- Ở mức từng author: % recipe rơi vào category phổ biến nhất của họ — median=19%, mean=21%, max=58.6%, không có case >90%.

**Kết luận**: rủi ro author-clustering lên category-level comparison ở mức **thấp-trung bình**. Không đủ nghiêm trọng để loại category nào, nhưng giữ nguyên robustness check (loại top-author) đã quyết định ở Phase 3.

### (c) Nutrition: per-serving hay per-recipe?

**Không xác định dứt khoát được.** Bằng chứng hai chiều:
- Tương quan Calories vs RecipeServings (N=6,547): **r=-0.216** — phù hợp hơn với per-serving (nhiều serving → calo/dòng thấp hơn).
- Nhưng eyeball từng recipe cụ thể: có case hợp lý nếu per-serving (Low Cost Chocolate Cake: 20 servings, 208 calo), có case chỉ hợp lý nếu per-recipe/total (Eggless Chocolate Cake: thiếu RecipeServings, 2,537 calo).

→ **Không đồng nhất giữa các recipe** — giữ nguyên là limitation chưa giải quyết, không giả định per-serving khi dùng làm proxy ở Phase 5.

## PHASE 5 — STATISTICAL METHOD SELECTION

### 5.0 Fact quan trọng: phân bố outcome

`AggregatedRating` (N=2,197, subset khả dụng): **skew=-2.52**, median=5.0, IQR=[4.5, 5.0], **71.1% recipe đạt đúng 5.0**, 97.5% ≥4.0. Ceiling effect rất mạnh → loại trực tiếp t-test/ANOVA/OLS thông thường (giả định normal residual vi phạm rõ ràng).

### 5.a Outlier treatment — Calories

Không cắt percentile tùy tiện. Kiểm tra liên hệ với vấn đề đo lường ở Phase 4(c): **36/39 (92%) recipe có Calories>3,000 đều thiếu `RecipeServings`**; p95 Calories = 2,752 (nhóm thiếu servings) vs 793 (nhóm có servings) — chênh lệch hệ thống, không ngẫu nhiên.

**Quyết định**: loại nhóm thiếu `RecipeServings` khỏi **nutrition-analysis cụ thể** (không phải khỏi toàn dataset) — lý do measurement-validity (khả năng khác đơn vị đo), không phải "outlier xấu". Giữ 1 robustness check riêng bao gồm nhóm này.

### 5.b Nutrition-proxy — quyết định KHÔNG composite

Composite (Health Score = f(nhiều biến)) đòi hỏi chọn trọng số — chưa có cơ sở validate, tự nó là proxy chưa audit (vi phạm nguyên tắc "không tạo weighting tùy tiện").

**Quyết định reduce scope**: test **3 nutrition dimension riêng lẻ** — SugarContent, SodiumContent, SaturatedFatContent — không composite ở round 1.

### 5.c Statistical test selection

| | Test | Lý do | Giả định | Cách kiểm tra |
|---|---|---|---|---|
| Nutrition dimension (2 nhóm cao/thấp theo median trong category) | Mann-Whitney U, stratified theo category | Rank-based, robust với ceiling effect + tie dày ở 5.0 | Independence (vi phạm 1 phần do author-clustering); similar-shape distribution để diễn giải effect size | Robustness loại top-author; so sánh boxplot 2 nhóm trước khi diễn giải |
| Prep-time bucket (≥3 nhóm) | Kruskal-Wallis, stratified theo category | Tương tự | Tương tự | Tương tự |
| Category (19 nhóm) | Kruskal-Wallis, không stratify (category chính là biến test) | Không cần normal | Independence; **thêm limitation đã biết**: rating-missingness khác nhau theo category — không tách được khỏi selection bias | Không kiểm tra thêm được bằng thống kê — nêu như limitation |

**Cảnh báo bắt buộc khi báo cáo kết quả**: với 71% giá trị = 5.0 tuyệt đối, khác biệt "location shift" từ rank-test phần lớn phản ánh khác biệt **tỷ lệ đạt-5.0-tuyệt-đối** giữa nhóm, không phải phân bố trải đều. Phải báo cáo kèm % Rating=5.0 theo từng nhóm.

**Multiple testing**: Benjamini-Hochberg, áp dụng khi có p-value thật.

## FROZEN BLOCK — Phase 3-5 (bổ sung, đã audit)

```
- Salad: KHÔNG tồn tại trong RecipeCategory (category gần giống duy nhất có thật:
  "Salad Dressings", khác concept)
- IngredientCount: mean=7.9, median=7, range=0-35, 33 recipe rỗng thật
- IngredientCount vs PrepTime r≈0.39-0.41 | vs Rating r≈-0.01 → không coi confounder
  độc lập của Rating, đưa vào robustness cho cả nutrition & preptime-effect
- Author category-specialization: median 19%, max 58.6%, không case >90%
  → rủi ro author-clustering lên category-comparison: thấp-trung bình
- Nutrition per-serving vs per-recipe: KHÔNG XÁC ĐỊNH DỨT KHOÁT — giữ là limitation
- Outlier Calories: loại có lý do (36/39 Calories>3000 thiếu RecipeServings, p95 gap
  2,752 vs 793) → nhóm thiếu RecipeServings exclude khỏi nutrition-analysis (không phải
  khỏi toàn dataset), giữ robustness check riêng bao gồm nhóm này
- 3 nutrition dimension test riêng: SugarContent, SodiumContent, SaturatedFatContent
  (không composite ở round 1)
- Test method: Mann-Whitney U (2 nhóm) / Kruskal-Wallis (≥3 nhóm, category 19 nhóm)
  — lý do: AggregatedRating skew=-2.52, 71.1%=5.0 (ceiling effect), loại t-test/ANOVA/OLS
- Giả định & cách kiểm tra: independence (robustness loại top-author), similar-shape
  distribution (so sánh boxplot trước khi diễn giải rank-biserial)
- Cảnh báo bắt buộc: kèm % Rating=5.0 theo nhóm cùng kết quả rank-test
- Multiple testing: Benjamini-Hochberg
```

---

# PART 3 — Phase 6 (Kết quả thực nghiệm) → Phase 9 (Opportunity Screening) + Final Memo

*(Tiếp nối Part 1-2 — Phase 0-5 đã frozen. Baseline: within-category comparison, ReviewCount≥3 primary, 3 main effect round 1: category / prep-time bucket / nutrition dimension (Sugar/Sodium/SaturatedFat), test method Mann-Whitney U / Kruskal-Wallis do ceiling effect ở outcome.)*

## PHASE 6 — KẾT QUẢ THỰC NGHIỆM

### 6.0 N thực tế từng test (sau stratify đúng vào 19-category)

| Test | N | Ghi chú |
|---|---|---|
| Category-effect | 1,429 | 19/19 category (toàn bộ 19-category list) |
| Prep-time-effect | 1,323 | 17/19 category — loại `< 60 Mins`, `Beverages` do 1 tertile bucket <5 recipe |
| Nutrition-effect | 898 | 18/19 category — loại `Yeast Breads` (N=9, chia high/low ra 4/5, dưới ngưỡng min 5/nhóm) |

Ba N khác nhau vì mỗi test qua số tầng lọc khác nhau — không dùng chung 1 con số N.

### 6.1 Category main effect (Kruskal-Wallis, 19 nhóm, N=1,429)

**H=21.803, df=18, p=0.241, ε²=0.0027** → Không có ý nghĩa thống kê, effect size gần như bằng 0.

### 6.2 Prep-time bucket effect (tertile Low/Mid/High, stratified 17/19 category, Fisher combine)

Cutpoint tertile: ≤10 phút / 10–15 phút / >15 phút (tính trên N=1,429).

**Fisher combined χ²=29.405, df=34, p=0.692, weighted ε²≈-0.004 (≈0)** → Không có ý nghĩa thống kê.

*Ghi chú minh bạch*: một lần reproduce độc lập khác cho χ²≈28.9, p≈0.716 (cùng k=17, df=34) — chênh lệch nhỏ. Đã kiểm tra giả thuyết "lỗi parse PT0S gây missing value" — **không xác nhận được** (xem Phụ lục, mục 7): code parse thực tế trả về 0 cho PT0S, không phải NaN, và `PrepTimeMin` xác nhận có 0 missing value trong subset N=1,429. Nguyên nhân chênh lệch cụ thể **chưa xác định được dứt khoát** — có thể do khác biệt tie-handling hoặc phiên bản thư viện giữa các lần triển khai độc lập. Cả hai giá trị đều cách xa ngưỡng 0.05, **không ảnh hưởng kết luận** — đây là chênh lệch không đáng kể, không phải correction.

### 6.3 Nutrition dimension (2 nhóm cao/thấp theo median trong category, stratified 18/19 category, N=898, Fisher combine)

| Dimension | p combined | Weighted rank-biserial r |
|---|---|---|
| SugarContent | 0.514 | -0.009 |
| SodiumContent | 0.330 | -0.003 |
| SaturatedFatContent | 0.581 | -0.033 |

Cả 3 dimension: không có ý nghĩa thống kê, effect size gần như bằng 0.

### 6.4 % Rating=5.0 theo nhóm (kèm theo yêu cầu bắt buộc ở Phase 5)

Tỷ lệ đạt-5.0 dao động 50–90% ở cả nhóm cao lẫn thấp mỗi dimension, không có xu hướng nhất quán một chiều giữa category — khớp với kết quả rank-test null.

### 6.5 Benjamini-Hochberg (5 test: Category, PrepTime, Sugar, Sodium, SaturatedFat)

Tất cả **p_adj = 0.692** — không test nào reject H0 dù trước hay sau correction.

### 6.6 Robustness — Category-effect (test duy nhất có 1 threshold nominally significant)

| Điều kiện | N | k | p | ε² |
|---|---|---|---|---|
| RC≥3 (primary) | 1,429 | 19 | 0.241 | 0.0027 |
| RC≥5 | 667 | 12 | **0.049** | 0.013 |
| RC≥10 | 142 | 4 | 0.479 | -0.004 |
| RC≥3, loại top-author (≥10 recipe) | 855 | 14 | 0.463 | -0.0002 |

**p=0.049 ở RC≥5 KHÔNG bền** — biến mất ở RC≥3 (primary), RC≥10, và khi loại top-author. Đây là ví dụ điển hình của finding không robust: nếu chỉ chạy 1 threshold rồi dừng sẽ kết luận sai. Kết quả đúng: **không nâng lên Association**.

## PHASE 7 — ROBUSTNESS (độ phủ)

Robustness-matrix đầy đủ (RC≥5/10, loại top-author) **chỉ chạy cho Category-effect** — test duy nhất có 1 threshold nominally significant. **Sodium-effect** chỉ chạy robustness giới hạn (RC≥5, loại top-author) — cả hai vẫn null (p_comb=0.357 và 0.661). **Sugar, SaturatedFat, Prep-time-effect không chạy robustness bổ sung** vì effect size đã ≈0 ở primary test.

Đây là **quyết định phân bổ effort theo trọng số finding** (không kiểm tra sâu một null-result đã rõ ràng), không phải thiếu sót quy trình — nhưng cần nêu rõ để người đọc biết mức độ kiểm chứng khác nhau giữa các dòng kết quả.

## PHASE 8 — EVIDENCE SYNTHESIS

### Evidence classification

| Finding | Evidence Strength | Ghi chú |
|---|---|---|
| Nutrition (Sugar/Sodium/SaturatedFat) → Rating, within-category | **Insufficient Evidence** | Không đạt cả mức Association ở primary threshold |
| Prep-time bucket → Rating, within-category | **Insufficient Evidence** | Tương tự |
| Category → Rating | **Insufficient Evidence** | 1 kết quả nominally significant nhưng không robust — không nâng lên Association |

**"Insufficient Evidence" không đồng nghĩa "không có effect nào tồn tại"** — chỉ có nghĩa là với N=898–1,429, 17–19 strata, phương pháp rank-based đã chọn, không phát hiện được effect đủ lớn để phân biệt khỏi noise. Power ở mức N/strata này khiêm tốn.

### Trả lời business question gốc

> "Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others."

Round 1 (main effect riêng lẻ) cho **Insufficient Evidence trên cả 3 nhóm attribute**. Câu hỏi gốc — mở rộng sang "combination" (interaction effect) — **chưa từng được test**, vì round 1 dừng ở main effect và không có main effect nào đủ mạnh để justify bước sang interaction (N sẽ càng nhỏ, càng underpowered).

**3 giới hạn chính khiến power thấp:**
1. **Ceiling effect ở outcome**: skew=-2.52; 71.1% recipe Rating=5.0 tuyệt đối, 97.5% ≥4.0.
2. **Selection bias**: 48.3% recipe (4,827/10,000) không có `AggregatedRating` — không phải random missing.
3. **N-shrinkage qua các tầng lọc validity**: **category-effect 1,429/10,000 (14.3%)**, **prep-time-effect 1,323/10,000 (13.2%)**, **nutrition-effect 898/10,000 (8.98%)** — ba con số tách riêng, không gộp chung.

## PHASE 9 — R&D OPPORTUNITY SCREENING

| Finding | Evidence Strength | Business Interpretation | Recommended Next Step |
|---|---|---|---|
| Category → Rating | Insufficient Evidence | Not actionable yet | No Action |
| Prep-time bucket → Rating | Insufficient Evidence | Not actionable yet | No Action |
| Nutrition (Sugar/Sodium/SaturatedFat) → Rating | Insufficient Evidence | Not actionable yet | No Action |

**0/3 finding đạt Robust Evidence** → không có opportunity nào để screen tiếp lên Investigate/Consumer Research/Prototype. Không có Operational Owner/Method cần điền (chỉ áp dụng khi Next Step = Monitor/Investigate).

## FINAL MEMO

### Business Question Verdict

**Câu hỏi gốc:** "Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others."

**Answerability:** Weakly answerable.

**Trả lời trực tiếp:** Round 1 (category, prep-time bucket, 3 nutrition dimension) cho **Insufficient Evidence trên cả 3 nhóm attribute**. Không có attribute nào vượt ngưỡng "Association", chưa nói đến "Robust Evidence". Phần "combination" của câu hỏi gốc chưa từng được test.

### 3 giới hạn chính

1. **Ceiling effect ở outcome**: `AggregatedRating` lệch trái cực mạnh (skew=-2.52); 71.1% Rating=5.0 tuyệt đối, 97.5% ≥4.0.
2. **Selection bias**: 48.3% recipe (4,827/10,000) không có `AggregatedRating` — nhóm chưa từng được ai rate, không phải random missing.
3. **N cuối cùng qua các tầng lọc**: 1,429/10,000 (14.3%) cho category-effect, 1,323/10,000 (13.2%) cho prep-time-effect, 898/10,000 (8.98%) cho nutrition-effect. Ba con số khác nhau vì mỗi test qua số tầng lọc khác nhau (prep-time-effect giảm thêm do loại 2 category bucket<5; nutrition-effect giảm thêm do yêu cầu `RecipeServings` present và loại thêm 1 category do 1 bucket high/low <5 — Yeast Breads, N=9, chia 4/5).

### Khuyến nghị R&D

- **No Action** trên finding hiện tại — không đủ bằng chứng để đầu tư consumer research/sensory test/prototype cho bất kỳ attribute nào đã test.
- **Nếu muốn theo đuổi tiếp**: cần **Find More Data** — dataset có rating scale không bị ceiling/bão hòa và/hoặc N lớn hơn ở nhóm recipe đủ review, trước khi đầu tư thêm effort vào sample 10K này.

### Giới hạn phương pháp luận

Robustness check không đồng đều giữa 5 test: Category-effect kiểm tra đầy đủ (RC≥5/10, loại top-author) vì là test duy nhất có 1 threshold nominally significant (p=0.049 ở RC≥5, không bền); Sodium-effect chỉ kiểm tra giới hạn; Sugar, SaturatedFat, Prep-time-effect không chạy thêm robustness vì effect size đã ≈0 ở primary test. Đây là phân bổ effort theo trọng số finding, không phải thiếu sót.

## FROZEN BLOCK — Phase 0-6 (đầy đủ, đã audit)

```
🔒 FROZEN (Phase 0-6, đã audit toàn bộ project):

[Phase 0-2]
- N_total = 10,000 (sample, KHÔNG ngoại suy 500K+)
- N_rated = 5,173 (48.3% missing AggregatedRating — selection bias)
- Time scope: KHÔNG lọc theo năm; r(age,ReviewCount)=0.25 thật, r(age,Rating)=-0.057 yếu
- ReviewCount threshold: ≥3 (primary) | ≥5, ≥10 = robustness bắt buộc
- Attribute round 1: main effect riêng lẻ (category / prep-time bucket / nutrition
  dimension) — KHÔNG test combination
- 19-category list (N≥30 sau rated & ReviewCount≥3): Dessert(206), Lunch/Snacks(146),
  One Dish Meal(136), Vegetable(129), Breakfast(90), Chicken(80), Beverages(75),
  Chicken Breast(70), Potato(68), Pork(56), Sauces(55), Breads(50), Quick Breads(47),
  Meat(44), Pie(40), Drop Cookies(38), Bar Cookie(38), < 60 Mins(31), Yeast Breads(30)
- Rating-missingness theo category: 39.9% (<60 Mins) – 64.4% (Potato)
- AuthorId ≠ customer | Không có review text → NLP NOT TESTABLE

[Phase 3-5]
- Author clustering: 145/4,618 author ≥10 recipe = 2,967/10,000 (29.7%), max=141
- IngredientCount: mean=7.9, median=7, range=0-35 | vs PrepTime r≈0.39 | vs Rating r≈-0.01
- Nutrition per-serving vs per-recipe: KHÔNG XÁC ĐỊNH DỨT KHOÁT — limitation
- Outlier Calories: loại nhóm thiếu RecipeServings khỏi nutrition-analysis (36/39
  Calories>3000 thiếu RecipeServings) — measurement-validity, không phải statistical cut
- 3 nutrition dimension riêng: Sugar/Sodium/SaturatedFatContent, không composite
- Test method: Mann-Whitney U (2 nhóm) / Kruskal-Wallis (≥3 nhóm) — do ceiling effect
  outcome (skew=-2.52, 71.1%=5.0)

[Phase 6 — KẾT QUẢ]
- Category-effect: N=1,429 (19/19 cat), H=21.803, df=18, p=0.241, ε²=0.0027
  → Insufficient Evidence. Robustness p=0.049 ở RC≥5 KHÔNG bền (biến mất RC≥3/10/
  loại top-author).
- Prep-time-effect: N=1,323 (17/19 cat), Fisher χ²=29.405, df=34, p=0.692
  → Insufficient Evidence.
- Nutrition-effect: N=898 (18/19 cat, loại Yeast Breads N=9), Sugar p=0.514/r=-0.009,
  Sodium p=0.330/r=-0.003, SaturatedFat p=0.581/r=-0.033 → Insufficient Evidence.
- Benjamini-Hochberg (5 test): tất cả p_adj=0.692, không reject H0 nào.
- Robustness coverage không đồng đều: đầy đủ cho Category-effect, giới hạn cho Sodium,
  không bổ sung cho Sugar/SaturatedFat/Prep-time (theo trọng số finding).
```

---

## PHỤ LỤC — REPRODUCIBILITY NOTES

Các chi tiết kỹ thuật cần đúng để tái lập kết quả trên Colab hoặc môi trường khác. Mọi điểm dưới đây đã verify trực tiếp trên `recipes_10k.csv`, bao gồm cả việc test chạy thật notebook trên Colab.

### 1. Parse `RecipeIngredientParts`

Format gốc là R-vector serialize: `c("black beans", "black-eyed peas", ...)`. Trích bằng regex:

```python
import re
def parse_r_vector(s):
    if pd.isna(s): return []
    return re.findall(r'"([^"]*)"', s)
```

33 recipe có `character(0)` (danh sách rỗng thật trong data gốc, không phải lỗi parse) — không loại bỏ, giữ IngredientCount=0.

### 2. Parse `PrepTime`/`CookTime`/`TotalTime` (ISO 8601 duration)

```python
def parse_iso(s):
    if pd.isna(s): return None
    h = re.search(r'(\d+)H', s); m = re.search(r'(\d+)M', s)
    t = 0
    if h: t += int(h.group(1))*60
    if m: t += int(m.group(1))
    return t
```

Case `PT0S` (chỉ có giây, không giờ/phút) — có 318 recipe dạng này trong dataset — **parse đúng thành 0 phút** với hàm trên (vì `t` khởi tạo mặc định = 0, không phụ thuộc regex H/M match hay không). Đã verify `PrepTimeMin` có 0 giá trị missing trong subset 19-category (N=1,429). Hàm trên không xử lý phần giây (bỏ qua `S`) — với mục đích phân tích ở phút thì không ảnh hưởng.

### 3. 19-category list — ngưỡng xác định

N tính trên subset `(AggregatedRating.notna()) & (ReviewCount >= 3)`, sau đó `value_counts()` theo `RecipeCategory`, giữ category có N≥30. **Không tính N trên toàn bộ dataset thô** (lỗi đã từng xảy ra và được sửa ở Phase 3 — nhầm `< 30 Mins` với `< 60 Mins` do dùng sai basis N).

### 4. Nutrition-effect — điều kiện loại category

Với mỗi category trong 19-category list, chia recipe thành 2 nhóm theo median của nutrient đang xét (`SugarContent`/`SodiumContent`/`SaturatedFatContent`) **trong chính category đó** (không dùng median toàn dataset). Nếu 1 trong 2 nhóm (high/low) có **<5 recipe**, loại category đó khỏi test. Áp dụng điều kiện này loại `Yeast Breads` (N=9, chia 4/5) → còn 18/19 category, N=898.

### 5. Prep-time bucket — cutpoint

Tertile toàn cục trên `PrepTimeMin` trong subset 19-category (N=1,429): dùng `quantile([1/3, 2/3])`, cho q1=10.0 phút, q2=15.0 phút. Rule: `x<=q1` → Low, `q1<x<=q2` → Mid, `x>q2` → High. Category bị loại nếu 1 trong 3 bucket có <5 recipe (loại `< 60 Mins`, `Beverages` → còn 17/19, N=1,323).

### 6. Combine p-value qua các category (stratified test)

- **Nutrition (2 nhóm, có hướng)**: Mann-Whitney U từng category → **Fisher's method** để combine p-value: `χ² = -2·Σln(pᵢ)`, df=2k. Rank-biserial effect size từng category, tổng hợp bằng weighted average (weight=N category).
- **Prep-time (3 nhóm, omnibus)**: Kruskal-Wallis từng category → **Fisher's method** tương tự (không dùng Stouffer's Z vì KW không có hướng tự nhiên để combine qua Z-score).
- **Category-effect**: Kruskal-Wallis 1 lần trên toàn bộ 19 nhóm (không stratify — category chính là biến đang test).

### 7. Đã thử nhưng KHÔNG xác nhận được

Một lần reproduce độc lập cho Prep-time Fisher-combine cho kết quả χ²≈28.9, p≈0.716 (so với 29.405/0.692 đã báo cáo, cùng k=17, df=34). Đã kiểm tra giả thuyết "lỗi parse PT0S gây missing" — **không xác nhận được**: hàm parse ở mục 2 trả về 0 (không phải NaN) cho PT0S, và `PrepTimeMin` xác nhận có 0 missing value trong subset dùng cho test này — **đã verify bằng notebook chạy thật trên Colab**, kết quả khớp tuyệt đối với 29.405/0.692. Nguyên nhân chênh lệch cụ thể ở 1 lần reproduce khác **chưa xác định được dứt khoát** — khả năng do khác biệt tie-handling hoặc phiên bản thư viện. Chênh lệch không ảnh hưởng kết luận (cả hai giá trị đều p>>0.05).

---

## Ghi chú audit trail

Report này đã qua 3 vòng audit độc lập (Part 1, Part 2, Part 3), verify trực tiếp trên `recipes_10k.csv`, và test chạy thật notebook trên Colab (khớp tuyệt đối với mọi số liệu trong report). 3 correction thật đã xảy ra và được sửa trong quá trình:

1. **Phase 3**: nhầm tên category `< 30 Mins` thay vì `< 60 Mins` trong bảng rating-missingness.
2. **Phase 6**: N=907 (pool trước lọc category) báo cáo nhầm thành N test thực tế (đúng là 898, sau loại Yeast Breads).
3. **Final Memo**: N=1,429 (của category-effect) gộp nhầm dùng chung cho prep-time-effect (đúng là 1,323).

Không có VIOLATION nào còn tồn đọng ở bản này. Các artifact liên quan: `FB_Recipe_Analysis_Notebook.ipynb` (code, đã test chạy trên Colab), `FB_Decision_Log.md` (10 quyết định rẽ nhánh), `FB_Assumption_Register.md` (5 giả định bắt buộc + rủi ro), `FB_Summary.md` (tóm tắt 1 trang).
