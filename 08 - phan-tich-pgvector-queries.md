# Phân tích chain học tập — Query Vector trong pgvector (Toán Tử Distance & Threshold)

> Mục tiêu: tách *suy ra được* (nhóm A) khỏi *phải ghi nhớ* (nhóm B), đánh dấu **⚠** chỗ danh sách/logic nhảy, rút **xương sống**.

Đây là **mảnh 8** — hoàn tất bộ hands-on: **cài (#6) → nạp (#7) → query (bài này)**. Phần toán tử distance trùng file 1/6 (đã có `<->`/`<=>`/`<#>` và ops class). **Giá trị mới, đáng học nhất** nằm ở phần **threshold**: "gần nhất ≠ đủ tốt", hiệu chỉnh bằng golden set, bẫy index threshold-thuần, và "threshold không cứu được recall kém của index". Có 3 đính chính bài gốc.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Hai yêu cầu bắt buộc khi query ↩(≈ đổi model = re-embed)

**A — Nhân quả**
- embedding trong DB và embedding câu tìm **khác model** => nằm ở *hai không gian khác nhau* → distance vô nghĩa — *vì* mỗi model có không gian riêng.
- tính distance cần **cùng số chiều (length)** — *vì* phép tính đòi hai vector cùng độ dài.
- model quyết định số chiều => đổi model = đổi chiều = re-embed toàn bộ — *vì* chiều gắn với model.

> Chain này **thuần nhóm A** — hiểu là xong.

**Xương sống**: distance có nghĩa cần cùng model + cùng chiều → đổi model = re-embed.

---

## Chain 2 — Đừng SELECT raw vector

**A — Nhân quả**
- `SELECT embedding` trả mảng float khổng lồ, con người không đọc hiểu => lấy nguyên trạng chẳng cho thông tin — *vì* ý nghĩa nằm ở *tổng thể*, không ở từng số.
- => cái ta muốn là *so sánh* để tìm bản ghi giống, không phải *xem* vector => query luôn xoay quanh **distance/similarity** — *vì* đó mới là thông tin hữu dụng.

> ⭐ Mental model: **"so sánh, không phải xem."**

**Xương sống**: SELECT raw vector vô dụng (mảng float) → query xoay quanh distance/similarity.

---

## Chain 3 — tsquery vs vector (đừng nhầm `<->`)

**A — Nhân quả**
- (không) — phân biệt hai khái niệm.

**B — Ghi nhớ**
- **tsquery** trên `tsvector` = lexeme (keyword chính xác); **pgvector** = embedding (ngữ nghĩa). "ô tô" khớp "xe hơi": tsquery **không**, pgvector **có** — *phân biệt*.
- ⚠ **Bẫy ký hiệu:** `<->` trong tsquery = **FOLLOWED BY** (kề nhau); `<->` trong vector = **L2 distance**. Trùng ký hiệu, khác ngữ cảnh.

> Chain này **chủ yếu nhóm B**. Điểm cần khắc: `<->` có **hai nghĩa** tùy ngữ cảnh (nối file 2 FTS).

**Xương sống**: tsquery (lexeme) ≠ vector (ngữ nghĩa); `<->` = FOLLOWED BY trong tsquery vs L2 trong vector.

---

## Chain 4 — Query similarity đơn giản nhất

**A — Nhân quả**
- `<=>` là cosine distance (nhỏ = gần) => mặc định `ORDER BY` **ASC** cho distance nhỏ nhất (giống nhất) lên đầu — *vì* nhỏ hơn nghĩa là gần hơn.

**B — Ghi nhớ**
- `ORDER BY embedding <=> '[...]' LIMIT 5` — *cú pháp*; ⚠ **đính chính:** `<=>` là cosine **distance** = `1 − cosine_similarity`, miền **[0,2]**.
- `ORDER BY <toán tử distance> LIMIT k` là **khung xương của MỌI vector query** — *mental model chốt*.

**Xương sống**: khung xương query = ORDER BY <distance> LIMIT k (ASC vì nhỏ=gần).

---

## Chain 5 — Ba toán tử trong ngữ cảnh query ↩(≈ file 1, 6)

**A — Nhân quả**
- `<#>` trả **âm** inner product => `ORDER BY ASC` vẫn cho gần nhất — *vì* Postgres index ASC, đảo dấu để nhỏ=gần (đừng ngạc nhiên khi thấy số âm).

**B — Ghi nhớ**
- `<->` L2 (không giới hạn trên, `vector_l2_ops`) · `<#>` negative IP (`vector_ip_ops`) · `<=>` cosine distance [0,2] (`vector_cosine_ops`); ops class khớp toán tử; text mặc định cosine, normalize→IP, L2 khi độ lớn có nghĩa — *cú pháp/quy tắc*.

**Xương sống**: 3 toán tử + ops class khớp; <#> âm để ASC=gần; text→cosine.

---

## Chain 6 — Pattern tìm giống bản ghi ĐÃ CÓ

**A — Nhân quả**
- distance tới **chính nó = 0** => nếu không loại, bản ghi nguồn luôn đứng đầu, che mất một kết quả thật => phải `WHERE id != 1` — *vì* self-distance = 0 là nhỏ nhất tuyệt đối.

**B — Ghi nhớ**
- Vector truy vấn = embedding dòng có sẵn: `ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id=1)` — *cú pháp*.

**Xương sống**: tìm giống bản ghi có sẵn = subquery embedding + WHERE id != self (vì self-distance = 0 luôn top).

---

## Chain 7 — Lấy distance/similarity ra & hiển thị %

**A — Nhân quả**
- distance **nhỏ** mới là giống => đừng show thẳng cosine *distance* rồi gọi "độ giống" — ngược nghĩa — *vì* distance lớn = khác, similarity mới là "giống".

**B — Ghi nhớ**
- `embedding <=> q AS cosine_distance` (0=giống hệt, 2=ngược); `1 - (embedding <=> q) AS cosine_similarity` (1=giống hệt, -1=ngược); hiển thị %: `1-(<=>)` rồi ×100 — *cú pháp*.

**Xương sống**: lấy điểm ra bằng SELECT; similarity = 1-(<=>) ×100; đừng đảo distance/similarity.

---

## Chain 8 — Ba lỗi query

**A — Nhân quả (mỗi lỗi có "vì" riêng)**
- quên `WHERE id != 1` => đầu bảng là chính nó, che một kết quả thật.
- ops class không khớp toán tử (`vector_cosine_ops` mà query `<->`) => Postgres bỏ index, **seq scan âm thầm** → kiểm `EXPLAIN ANALYZE`.
- nhầm distance/similarity => `ORDER BY DESC` vì tưởng "lớn = giống" → đảo ngược kết quả (mặc định ASC là đúng).

> ⚠ **Danh sách 3 lỗi**, không phải chuỗi nhân quả. Học như checklist.

**Xương sống**: (checklist) quên id!=self / ops class không khớp → seq scan / nhầm distance-similarity → ORDER BY sai.

---

## Chain 9 — Threshold: gần nhất ≠ đủ tốt ⭐⭐(mắt xích CỐT LÕI, mới)

**A — Nhân quả**
- `ORDER BY ... LIMIT 5` LUÔN trả 5 dòng — kể cả khi cả 5 chẳng liên quan ("ít xa nhất trong đám xa") => cần **threshold** — *vì* LIMIT chỉ cắt *số lượng*, không xét *chất lượng*.
- LIMIT giới hạn **số lượng**, threshold đảm bảo **chất lượng** — hai việc khác nhau => "5 gần nhất" ≠ "5 cái *tốt*" => threshold chỉ trả kết quả đủ giống, nếu không thì **thà không trả**.

> ⭐ Mental model cốt lõi cả bài: **"gần nhất ≠ đủ tốt"** — LIMIT (số lượng) vs threshold (chất lượng).

**Xương sống**: LIMIT luôn trả k dòng kể cả rác → threshold lọc chất lượng (thà không trả còn hơn trả rác).

---

## Chain 10 — Threshold phụ thuộc metric & dataset ⭐(mới)

**A — Nhân quả**
- mỗi metric có miền giá trị khác nhau => KHÔNG có "ngưỡng vạn năng" — *vì* số ngưỡng chỉ có nghĩa trong miền của metric đó.
- số "5" trong ví dụ L2 bài gốc => chỉ đúng với dataset & model cụ thể — đổi model/chuẩn hóa là vô nghĩa — *vì* threshold gắn metric + dataset.
- cosine miền cố định [0,2] => **dễ đặt ngưỡng ổn định hơn** L2 (miền không giới hạn) — *vì* ngưỡng tái dùng được khi miền không đổi.

**B — Ghi nhớ**
- cosine distance **[0,2]** (thường <0.2–0.4 rất giống); L2 **[0,∞)** phụ thuộc thang/chuẩn hóa; negative IP phụ thuộc magnitude — *con số*.
- Hiệu chỉnh: chạy trên cặp *biết-giống* và *biết-khác*, xem distance rơi vào đâu, chọn ngưỡng tách hai nhóm — *phương pháp*.

**Xương sống**: không có ngưỡng vạn năng (metric+dataset-dependent) → cosine dễ đặt nhất (miền cố định) → hiệu chỉnh bằng cặp giống/khác.

---

## Chain 11 — Bẫy index: threshold thuần ⭐(mới)

**A — Nhân quả**
- k-NN (`ORDER BY distance LIMIT k`) dùng được ANN index; threshold + ORDER BY + LIMIT vẫn dùng => nhưng **threshold THUẦN** (`WHERE embedding <=> q < 0.3` không ORDER BY/LIMIT) → CÓ THỂ **seq scan**, tính distance MỌI dòng — *vì* HNSW/IVFFlat thiết kế cho k-nearest-neighbor, không phải range scan thuần.
- => muốn vừa nhanh vừa có ngưỡng => LUÔN kèm `ORDER BY distance LIMIT k` rồi mới lọc threshold — *vì* chỉ khi đó index mới được dùng.

**B — Ghi nhớ**
- Xác nhận bằng `EXPLAIN ANALYZE` (thấy `Index Scan` không) — *cú pháp*.

> ⭐ Mental model: **"k-NN thì index, range thuần thì seq scan"** — luôn ORDER BY + LIMIT.

**Xương sống**: k-NN dùng index; threshold THUẦN → seq scan (ANN không làm range scan) → luôn kèm ORDER BY LIMIT.

---

## Chain 12 — Threshold + filter & edge cases

**A — Nhân quả**
- threshold + filter nghiệp vụ => **overfiltering** (ANN lấy `ef_search` hàng xóm trước, lọc sau → thiếu) → `SET hnsw.iterative_scan = relaxed_order`.
- đổi model => ngưỡng cũ sai, phải hiệu chỉnh lại — *vì* ngưỡng gắn model+metric.
- `WHERE 1-(embedding<=>q) > 0.7` ≡ `WHERE embedding<=>q < 0.3` nhưng viết theo **distance** giúp planner tối ưu tốt hơn — *vì* planner nhận diện dạng distance để dùng index.

**B — Ghi nhớ**
- Threshold quá chặt → 0 kết quả (hợp lệ; xử lý ở app "không tìm thấy" thay vì lỗi) — *dữ kiện*.

> ⚠ Nửa sau là danh sách edge case. Học từng cái như lưu ý riêng.

**Xương sống**: threshold + filter → overfiltering → iterative_scan; viết WHERE theo distance (planner); đổi model phải hiệu chỉnh lại ngưỡng.

---

## Chain 13 — Query pattern ở quy mô lớn ↩(≈ file 1, 2, 7)

**A — Nhân quả**
- **pre-filter** chọn lọc (tenant/category/thời gian) TRƯỚC => giảm chi phí distance (+ partition prune) — *vì* thu hẹp tập ứng viên trước khi tính đắt.
- pagination `OFFSET n` lớn tốn kém với ANN => thực tế lấy top-k rồi rerank, ít phân trang sâu — *vì* ANN không hợp OFFSET sâu.
- embedding cùng câu truy vấn là xác định => cache theo hash để khỏi gọi model lại trong đường nóng — *vì* cùng input cho cùng vector.

**B — Ghi nhớ**
- **hybrid query**: vector (`<=>`) + full-text (`@@`) fuse bằng RRF → tốt hơn keyword-only/semantic-only (default retrieval 2026) — *dữ kiện* (nối file 2).

**Xương sống**: pre-filter/prune trước rank → hybrid RRF → tránh OFFSET sâu (top-k rerank) → cache query embedding theo hash.

---

## Chain 14 — Hiệu chỉnh threshold bằng golden set ⭐(mới, cốt lõi)

**A — Nhân quả**
- chọn threshold là **đánh đổi precision vs recall**: chặt → precision cao nhưng bỏ sót nhiều; lỏng → recall cao nhưng lẫn rác — *vì* một đường biên phải cắt giữa hai phân phối chồng lấn.

**B — Ghi nhớ (quy trình 4 bước)**
- **golden set** — *định nghĩa*: các cặp query–kết quả *đã gán nhãn* liên quan/không liên quan.
- (1) dựng golden set → (2) chạy query, ghi distance cặp liên quan vs không → vẽ phân phối → (3) chọn threshold tách hai phân phối tốt nhất → (4) theo dõi & tinh chỉnh khi dữ liệu/model đổi.
- Golden set còn dùng đo recall & so sánh model — **tài sản nên đầu tư**.

> ⭐ Mental model: **"ngưỡng là đường biên quyết định, không phải số phép màu"** — calibrate trên dữ liệu.

**Xương sống**: threshold = đường biên calibrate trên golden set (gán nhãn → phân phối → tách → theo dõi); trade-off precision/recall.

---

## Chain 15 — Correctness: ANN + threshold ⭐(mới)

**A — Nhân quả**
- ANN là **approximate** => threshold áp lên kết quả *xấp xỉ* => có thể bỏ sót bản ghi thật sự trong ngưỡng mà ANN không tìm ra — *vì* ANN vốn bỏ sót vài hàng xóm.
- cần đảm bảo (compliance) => **two-stage**: ANN lấy top-N rộng → tính EXACT distance + áp threshold trên N — *vì* exact trên N nhỏ vừa chắc vừa rẻ.
- => phải **monitor recall**: threshold **không cứu được recall kém** của index — *vì* threshold chỉ lọc chất lượng, không làm index tìm ra thêm hàng xóm.

**B — Ghi nhớ**
- Giám sát: phân phối distance (drift → chỉnh ngưỡng), tỉ lệ query 0 kết quả (ngưỡng quá chặt?), tỉ lệ seq-scan fallback (query viết sai mất index) — *dữ kiện*.

> ⭐ Insight quan trọng: **threshold không cứu recall kém** — hai vấn đề khác nhau (lọc chất lượng vs index tìm đủ hàng xóm).

**Xương sống**: ANN approximate → threshold có thể bỏ sót → two-stage exact khi cần chắc + monitor recall (threshold không cứu recall kém).

---

## Chain 16 — Tổ chức & khép series

**A — Nhân quả**
- "trả 0 kết quả khi không đủ gần" => là quyết định sản phẩm — *vì* thà im lặng còn hơn gợi ý rác.

**B — Ghi nhớ**
- Framing cho PM/sếp: LIMIT = "5 gần nhất" ≠ "5 cái tốt"; threshold là bộ lọc chất lượng, hiệu chỉnh trên data thật, chỉnh lại nếu đổi model; framing bằng **chất lượng kết quả & precision/recall** — *khuyến nghị*.
- Khép series 8 mảnh: embed → index → cài → nạp → **query** → store/query khái niệm → FTS → chọn nền tảng — *tổng hợp*.

**Xương sống**: "gần nhất ≠ đủ tốt" → threshold lọc chất lượng → 0-kết-quả là quyết định UX (thà im lặng hơn rác).

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (file 8)

> query cần **cùng model + cùng chiều** (khác → distance vô nghĩa) → đừng SELECT raw vector, query xoay quanh **distance/similarity** → đừng nhầm `<->` (FOLLOWED BY trong tsquery vs L2 trong vector) → khung xương **ORDER BY <distance> LIMIT k** (ASC vì nhỏ=gần) → 3 toán tử + **ops class khớp** → tìm giống bản ghi có sẵn (subquery + **WHERE id != self** vì self-distance=0) → similarity = `1-(<=>)` ×100, đừng đảo distance/similarity → LIMIT trả k dòng **kể cả rác** → **threshold** lọc chất lượng ("gần nhất ≠ đủ tốt") → threshold **phụ thuộc metric & dataset** (không có ngưỡng vạn năng; cosine dễ đặt nhất) → **bẫy: threshold thuần → seq scan**, luôn kèm ORDER BY LIMIT → threshold + filter gây **overfiltering** → iterative_scan → calibrate threshold bằng **golden set** (precision/recall trade-off) → ANN approximate nên threshold có thể bỏ sót → **two-stage exact** + **monitor recall** (threshold không cứu recall kém) → UX: **thà im lặng còn hơn rác**.

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (8 file)

> **text → [ENCODE] embed (f3)** → **[INDEX] B-tree gãy → ANN (f4)** → **[STORE & SEARCH] pgvector: khái niệm (f1) + cài (f6) + nạp (f7) + query (f8, bài này)** → **[KEYWORD] FTS (f2)** → **[HYBRID] RRF** → **[CHỌN NỀN TẢNG] (f5)** → **RAG**.

Bộ hands-on đã trọn: **cài (f6) → nạp (f7) → query (f8)**. Ba bài này là tầng *tay chân* của pgvector, dưới tầng *concept* (f1).

---

## 8. MẪU HÌNH LẶP LẠI

**Mẫu hình 1 — "Gần nhất ≠ đủ tốt": số lượng vs chất lượng.** ⭐ đóng góp RIÊNG của bài này.
Chain 9, 16. *Vì sao quan trọng*: LIMIT cho *số lượng*, threshold cho *chất lượng* — hai việc tách bạch. Không bài nào khác trong series nói điều này; đây là mắt xích mới cần nắm.

**Mẫu hình 2 — Ngưỡng/quyết định phải calibrate trên dữ liệu thật, không đoán bừa.** ⭐
Chain 10, 14 (golden set) ↔ file 3 ("MTEB là prior, data của bạn là verdict" khi chọn model) ↔ file 1/4 (đo recall trên tập vàng). *Vì sao là một*: threshold, model, recall — mọi con số quyết định đều phải verify trên golden set, không phải hằng số phép màu.

**Mẫu hình 3 — Cùng model/không gian mới so được.**
Chain 1 ↔ f1/f3/f5/f6/f7 (đổi model = re-embed). *Vì sao là một*: distance chỉ có nghĩa trong cùng một không gian.

**Mẫu hình 4 — Ops class khớp toán tử → nếu không, lỗi im lặng (seq scan).**
Chain 5, 8, 11 ↔ f1/f4/f6. *Vì sao là một*: index chỉ dùng khi ops class khớp; sai thì Postgres âm thầm seq scan.

**Mẫu hình 5 — k-NN dùng index; range scan thuần thì không.** ⭐ biến thể query của "có index ≠ dùng index".
Chain 11 ↔ file 6 chain 9 (có index ≠ dùng index). *Vì sao là một*: HNSW/IVFFlat thiết kế cho k-nearest-neighbor; threshold thuần (`WHERE distance < x`) là range scan → mất index. Luôn `ORDER BY ... LIMIT`.

**Mẫu hình 6 — ANN approximate → two-stage exact khi cần chắc + monitor recall.**
Chain 15 ↔ file 1 (two-stage retrieval), file 4 (monitor recall). *Vì sao là một*: kết quả xấp xỉ; khi cần đảm bảo thì lọc thô bằng ANN rồi tinh bằng exact trên N nhỏ. Và threshold **không** thay được việc đo recall.

**Mẫu hình 7 — Lọc thô/prune trước, tinh sau + cache.**
Chain 13 (pre-filter, hybrid RRF, cache query hash) ↔ file 1 (two-stage), file 2 (filter trước rank, RRF), file 7 (cache hash). *Vì sao là một*: thu hẹp bằng bước rẻ trước, và cache kết quả xác định.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

### Con số
- cosine distance **[0, 2]** (thường <0.2–0.4 rất giống); L2 **[0, ∞)**; negative IP phụ thuộc magnitude.
- Hiển thị "độ giống %": `1 - (embedding <=> q)` rồi **×100**.

### Định nghĩa & tên gọi
- 3 toán tử: `<->` L2 (`vector_l2_ops`) · `<#>` negative IP (`vector_ip_ops`) · `<=>` cosine distance [0,2] (`vector_cosine_ops`).
- `<=>` = cosine **distance** = `1 − cosine_similarity`; **ASC = gần**.
- `<->` **hai nghĩa**: FOLLOWED BY (tsquery) vs L2 (vector).
- tsquery/tsvector (lexeme) vs pgvector (ngữ nghĩa).
- **LIMIT = số lượng** vs **threshold = chất lượng**; "gần nhất ≠ đủ tốt".
- threshold **metric-dependent + dataset-dependent**; cosine dễ đặt nhất (miền cố định).
- **golden set**; "đường biên quyết định"; **precision/recall trade-off**.
- **threshold thuần → seq scan**; k-NN (ORDER BY + LIMIT) → dùng index.
- **overfiltering** → `iterative_scan` (relaxed_order); **two-stage exact**; **monitor recall**; distance **drift**.
- pre-filter/partition prune; **hybrid RRF**; pagination OFFSET (tốn với ANN); **cache query embedding theo hash**.
- Mental models: "gần nhất ≠ đủ tốt", "distance nhỏ = giống đừng đảo", "ngưỡng là đường biên không phải số phép màu", "k-NN thì index range thuần thì seq scan", "cùng model hay vô nghĩa".

### ⚠ Ba đính chính bài gốc (phải ghi đè)
1. `<=>` là cosine **distance** (`= 1 − similarity`, miền [0,2]); muốn similarity thì `1 - (embedding <=> q)`.
2. Threshold **metric-dependent & dataset-dependent** — số "5" không phải hằng số vạn năng.
3. Threshold **thuần** (`WHERE distance < x` không ORDER BY/LIMIT) thường **không dùng ANN index** → seq scan.

### Cú pháp (thuộc lòng)
- **k-NN:** `SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;` (ASC = gần nhất).
- **Giống bản ghi có sẵn:** `WHERE id != 1 ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1) LIMIT 5;`
- **Threshold + LIMIT (giữ index) + %:** `SELECT id, content, 1 - (embedding <=> :q) AS similarity FROM reviews WHERE embedding <=> :q < 0.3 ORDER BY embedding <=> :q LIMIT 5;`
- **Chống overfiltering:** `SET hnsw.iterative_scan = relaxed_order;`
- `EXPLAIN ANALYZE` để xác nhận `Index Scan`.

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào.

1. Vì sao query cần **cùng model + cùng chiều**?
2. Vì sao không nên `SELECT` thẳng cột vector?
3. Vì sao `<->` có **hai nghĩa** khác nhau (tsquery vs vector)?
4. Vì sao tìm giống một bản ghi có sẵn phải `WHERE id != self`?
5. Vì sao `<#>` trả **số âm** mà vẫn `ORDER BY ASC`?
6. Vì sao **"5 gần nhất" ≠ "5 cái tốt"** — LIMIT khác threshold thế nào?
7. Vì sao KHÔNG có ngưỡng threshold **vạn năng**?
8. Vì sao cosine **dễ đặt ngưỡng** ổn định hơn L2?
9. Vì sao threshold **THUẦN** (`WHERE distance < x`) gây seq scan?
10. Vì sao nên viết `WHERE` theo **distance** chứ không theo similarity (planner)?
11. Vì sao threshold cần calibrate trên **golden set** chứ không đoán bừa?
12. Vì sao chọn ngưỡng là đánh đổi **precision vs recall**?
13. Vì sao ANN + threshold có thể **bỏ sót**, và two-stage giải quyết thế nào?
14. Vì sao **"threshold không cứu được recall kém"** của index?
15. Vì sao **"trả 0 kết quả khi không đủ gần"** là quyết định sản phẩm tốt?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- **Phần cần nắm nhất là threshold** (chain 9–11, 14–15) — đây là giá trị mới của bài, không lặp lại ở đâu khác. Bốn ý cốt: "gần nhất ≠ đủ tốt", ngưỡng metric/dataset-dependent, threshold thuần → seq scan, threshold không cứu recall kém.
- Phần toán tử distance (chain 3–8) **trùng file 1/6** — nếu đã nắm thì lướt, chỉ chú ý bẫy `<->` hai nghĩa và `<=>` là distance.
- **Nhóm B** (mục 9): cú pháp query + miền các metric — học như thẻ code.
- **Tái tạo**: dựng lại xương sống (khung ORDER BY<distance>LIMIT → threshold → golden set → two-stage), rồi ghép vào **meta-xương sống 8 file**. f6+f7+f8 = bộ hands-on (cài/nạp/query).
- **3 đính chính** phải ghi đè (`<=>` là distance / ngưỡng không vạn năng / threshold thuần không dùng index).
