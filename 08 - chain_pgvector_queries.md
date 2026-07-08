# Query Vector trong pgvector: Toán Tử Distance & Threshold — Chain gối đầu

> Vị trí series (mảnh 8 "lấy dữ liệu ra ĐÚNG", hoàn tất bộ hands-on cài #6 → nạp #7 → **query bài này**). Toán tử distance đã xuất hiện ở bài pgvector khái niệm/FTS; ở đây tập trung *cách viết query thực tế* + đào sâu **threshold**.
>
> **Đính chính bài gốc (chưa chuẩn):** (1) `<=>` là cosine **distance**, không phải similarity (`= 1 − similarity`, miền [0,2]); muốn similarity thì lấy `1 - (embedding <=> q)`. (2) Threshold là **metric-dependent** và **dataset-dependent** — số "5" trong ví dụ L2 không phải hằng số vạn năng (cosine [0,2]; L2 không giới hạn). (3) Bẫy index: threshold *thuần* (`WHERE distance < x` không ORDER BY/LIMIT) thường **không dùng ANN index** → seq scan.

---

## CHAIN — Hai yêu cầu bắt buộc khi query

để distance có nghĩa cần hai yêu cầu bắt buộc => yêu cầu 1: **cùng model + version** — embedding trong DB và embedding câu tìm phải do cùng model sinh, nếu không nằm ở *hai không gian khác nhau* → distance vô nghĩa, kết quả loạn => yêu cầu 2: **cùng số chiều (length)** — để tính distance, hai vector phải cùng độ dài => model quyết định số chiều, nên đổi model = đổi chiều = phải re-embed toàn bộ

## CHAIN — Đừng SELECT raw vector

query vector viết gần giống SQL thường, nhưng đừng SELECT thẳng cột vector => `SELECT embedding` trả về mảng float khổng lồ `[0.0123, -0.87, ...]` — con người không đọc hiểu gì => lấy nguyên trạng chẳng cho thông tin, làm hỏng mục đích => cái ta muốn không phải *xem* vector mà *so sánh* nó để tìm bản ghi giống => nên query luôn xoay quanh **distance/similarity**, không phải bản thân cột vector

## CHAIN — tsquery vs vector (đừng nhầm `<->`)

có hai loại "search văn bản" dễ lẫn: `tsquery` trên `tsvector` (lexeme, keyword chính xác) vs pgvector (embedding, ngữ nghĩa) => "ô tô" khớp "xe hơi": tsquery **không**, pgvector **có** => trong tsquery, toán tử `<->` nghĩa là **FOLLOWED BY** (kề nhau): `red <-> mat` = red ngay trước mat => nhưng trong *vector*, `<->` là **L2 distance** => trùng ký hiệu, khác ngữ cảnh — đừng nhầm

## CHAIN — Query similarity đơn giản nhất

query semantic đơn giản nhất: `ORDER BY embedding <=> '[...]' LIMIT 5` => `<=>` là **cosine distance** (đính chính: KHÔNG phải cosine similarity) => `cosine_distance = 1 − cosine_similarity`, miền [0,2] => mặc định `ORDER BY` là **ASC** → distance nhỏ nhất (giống nhất) lên đầu => `ORDER BY <toán tử distance> LIMIT k` là khung xương của MỌI vector query

## CHAIN — Ba toán tử trong ngữ cảnh query

ba toán tử: `<->` (L2, không giới hạn trên), `<#>` (negative inner product), `<=>` (cosine distance, miền [0,2]) => `<#>` trả *âm* IP nên `ORDER BY ... ASC` vẫn cho gần nhất (đừng ngạc nhiên khi thấy số âm) => mỗi toán tử khớp một **ops class** khi index: `vector_l2_ops`/`vector_ip_ops`/`vector_cosine_ops` => chọn metric: text mặc định cosine, normalize length=1 dùng inner product (nhanh hơn), L2 khi độ lớn vector có nghĩa

## CHAIN — Pattern tìm giống bản ghi ĐÃ CÓ

tìm giống một bản ghi đã có (vd 5 review giống review id=1) => vector truy vấn chính là embedding của dòng có sẵn → dùng subquery `(SELECT embedding FROM reviews WHERE id=1)` => cắm vector dòng nguồn vào phép so: `ORDER BY embedding <=> (SELECT ...)` => phải `WHERE id != 1` để loại CHÍNH NÓ => vì distance tới chính nó = 0 nên nếu không loại, review 1 luôn đứng đầu, che mất một kết quả thật

## CHAIN — Lấy distance/similarity ra & hiển thị %

thường muốn *thấy* điểm số chứ không chỉ thứ tự => SELECT thêm `embedding <=> q AS cosine_distance` (0=giống hệt, 2=ngược) => và `1 - (embedding <=> q) AS cosine_similarity` (1=giống hệt, -1=ngược) => hiển thị "độ giống %" cho user: dùng `1-(<=>)` rồi ×100 => đừng show thẳng cosine *distance* rồi bảo "độ giống" — ngược nghĩa (distance nhỏ mới là giống)

## CHAIN — Ba lỗi query

lỗi 1 — quên loại chính bản ghi nguồn (`WHERE id != 1`) → kết quả đầu bảng là chính nó, che một kết quả thật => lỗi 2 — ops class index không khớp toán tử: index `vector_cosine_ops` nhưng query `<->` → Postgres bỏ index, seq scan âm thầm; kiểm bằng `EXPLAIN ANALYZE` => lỗi 3 — nhầm distance với similarity: `<=>` nhỏ = giống, `<#>` là *âm*; nếu sắp `ORDER BY DESC` vì tưởng "lớn = giống" → đảo ngược kết quả => mặc định ASC là đúng cho cả ba (pgvector thiết kế vậy)

## CHAIN — Threshold: gần nhất ≠ đủ tốt

`ORDER BY ... LIMIT 5` LUÔN trả 5 dòng — kể cả khi 5 dòng chẳng liên quan gì (chỉ là "ít xa nhất trong đám xa") => threshold khắc phục: chỉ tính match nếu distance trong ngưỡng (`WHERE embedding <=> q < 0.3`) => tức LIMIT giới hạn *số lượng*, threshold đảm bảo *chất lượng* — hai việc khác nhau => "5 gần nhất" không có nghĩa "5 cái *tốt*" => threshold là bộ lọc chất lượng: chỉ trả kết quả đủ giống, nếu không thì thà không trả

## CHAIN — Threshold phụ thuộc metric & dataset

KHÔNG có "ngưỡng vạn năng" — mỗi metric có miền giá trị khác nhau => cosine distance [0,2] (thường <0.2–0.4 là rất giống); L2 [0,∞) không giới hạn (phụ thuộc thang & chuẩn hóa); negative IP phụ thuộc magnitude => nên số "5" trong ví dụ L2 của bài gốc chỉ đúng với dataset & model cụ thể — đổi model/chuẩn hóa là số đó vô nghĩa => **cosine DỄ đặt ngưỡng hơn** vì miền cố định [0,2] → ngưỡng tái dùng ổn định hơn L2 => cách hiệu chỉnh: chạy query trên vài cặp *biết trước là giống* và *biết trước là khác*, xem distance rơi vào khoảng nào, rồi chọn ngưỡng tách hai nhóm

## CHAIN — Bẫy index: threshold thuần

k-NN query (`ORDER BY distance LIMIT k`) dùng được ANN index => threshold kèm `ORDER BY` + `LIMIT` vẫn dùng index => nhưng threshold THUẦN (`WHERE embedding <=> q < 0.3` không ORDER BY/LIMIT) → CÓ THỂ seq scan, tính distance MỌI dòng => vì HNSW/IVFFlat được thiết kế cho k-nearest-neighbor, không phải range scan thuần => muốn vừa nhanh vừa có ngưỡng → LUÔN kèm `ORDER BY distance LIMIT k` rồi mới lọc threshold => xác nhận bằng `EXPLAIN ANALYZE` (thấy `Index Scan` không)

## CHAIN — Threshold + filter & edge cases

kết hợp threshold với filter nghiệp vụ (`WHERE category=...`) có thể gặp **overfiltering** (ANN lấy `ef_search` hàng xóm trước, lọc sau → thiếu kết quả) => khắc phục bằng `SET hnsw.iterative_scan = relaxed_order` (pgvector 0.8+) => edge: threshold quá chặt → 0 kết quả (hợp lệ, xử lý ở app: "không tìm thấy" thay vì lỗi) => edge: đổi model → ngưỡng cũ sai, phải hiệu chỉnh lại (ngưỡng gắn với model+metric) => edge: `WHERE 1-(embedding<=>q) > 0.7` ≡ `WHERE embedding<=>q < 0.3` nhưng viết theo *distance* giúp planner tối ưu tốt hơn

## CHAIN — Query pattern ở quy mô lớn

**pre-filter** trước khi rank: đặt filter chọn lọc (tenant_id/category/thời gian) thu hẹp tập ứng viên TRƯỚC, giảm chi phí distance (kết hợp partition prune) => **hybrid query**: kết hợp vector (`<=>`) với full-text (`@@`) fuse bằng RRF → tốt hơn keyword-only hoặc semantic-only (default retrieval 2026) => **pagination**: `ORDER BY distance LIMIT k OFFSET n` chạy được nhưng OFFSET lớn tốn kém với ANN → thực tế thường lấy top-k rồi rerank, ít phân trang sâu => **cache query embedding**: embedding của cùng câu truy vấn là xác định → cache theo hash để khỏi gọi model lại trong đường nóng

## CHAIN — Hiệu chỉnh threshold bằng golden set

threshold không phải số phép màu đoán bừa, mà là **đường biên quyết định** calibrate trên dữ liệu => bước 1: dựng **golden set** — các cặp query–kết quả *đã gán nhãn* liên quan/không liên quan => bước 2: chạy query, ghi distance của cặp liên quan vs không liên quan → vẽ phân phối => bước 3: chọn threshold *tách hai phân phối* tốt nhất (đánh đổi precision vs recall: chặt → precision cao bỏ sót nhiều, lỏng → recall cao lẫn rác) => bước 4: theo dõi & tinh chỉnh khi dữ liệu/model đổi => golden set còn dùng lại để đo recall và so sánh model (tài sản nên đầu tư)

## CHAIN — Correctness: ANN + threshold

ANN là **approximate** → threshold áp lên kết quả *xấp xỉ* => có thể bỏ sót bản ghi thật sự trong ngưỡng nhưng ANN không tìm ra => nếu cần đảm bảo (vd compliance) → **two-stage**: ANN lấy top-N rộng → tính EXACT distance + áp threshold trên N đó => và phải **monitor recall**: threshold không cứu được recall kém của index => giám sát thêm: phân phối distance (drift → threshold cần chỉnh), tỉ lệ query 0 kết quả (threshold quá chặt?), tỉ lệ seq-scan fallback (query viết sai làm mất index)

## CHAIN — Tổ chức & khép series

nói với PM/sếp: kết quả search luôn *xếp hạng* theo độ giống; LIMIT bảo "cho tôi 5 gần nhất" nhưng "5 gần nhất" ≠ "5 cái *tốt*" => threshold là bộ lọc chất lượng: chỉ trả kết quả đủ giống, nếu không thì thà không trả; con số ngưỡng hiệu chỉnh trên dữ liệu thật, chỉnh lại nếu đổi model => framing bằng **chất lượng kết quả & đánh đổi precision/recall** => UX: quyết định "trả 0 kết quả khi không đủ gần" là quyết định sản phẩm (thà im lặng còn hơn gợi ý rác) => khép series 8 mảnh: embed → index → cài → nạp → **query (bài này)** → store/query khái niệm → FTS → chọn nền tảng

---

## BẢNG — tsquery vs vector

| | `tsquery` trên `tsvector` | pgvector (`<=>`...) |
|---|---|---|
| Tìm theo | **lexeme** (từ khóa chuẩn hóa) | **ngữ nghĩa** (embedding) |
| Hợp cho | keyword search chính xác | semantic search (nghĩa gần) |
| "red" khớp "reddish"? | có (cùng lexeme) | có nếu nghĩa gần |
| "ô tô" khớp "xe hơi"? | không | **có** |
| `<->` nghĩa là | FOLLOWED BY (kề nhau) | L2 distance |

## BẢNG — Ba toán tử distance

| Toán tử | Metric | Ghi chú | Ops class |
|---|---|---|---|
| `<->` | L2 / Euclidean | khoảng cách thẳng; không giới hạn trên | `vector_l2_ops` |
| `<#>` | Negative inner product | pgvector trả *âm* IP (để ASC = gần) | `vector_ip_ops` |
| `<=>` | Cosine distance | phổ biến nhất với text; miền [0,2] | `vector_cosine_ops` |

## BẢNG — Threshold theo metric

| Metric | Miền giá trị | "Gần" nghĩa là |
|---|---|---|
| Cosine distance (`<=>`) | **[0, 2]** | thường < 0.2–0.4 là rất giống |
| L2 (`<->`) | **[0, ∞)** không giới hạn | phụ thuộc thang & chuẩn hóa vector |
| Negative inner product (`<#>`) | phụ thuộc magnitude | khó đặt ngưỡng nếu chưa normalize |

Cosine dễ đặt ngưỡng ổn định nhất (miền cố định). Threshold gắn với model+metric → đổi model phải hiệu chỉnh lại.

## BẢNG — Code thuộc lòng

**(a) k-NN cơ bản:**
```sql
SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;   -- ASC = gần nhất trước
```

**(b) Giống một bản ghi có sẵn (loại chính nó):**
```sql
SELECT * FROM reviews
WHERE id != 1
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;
```

**(c) Threshold + LIMIT (giữ index) + similarity %:**
```sql
SELECT id, content, 1 - (embedding <=> :q) AS similarity
FROM reviews
WHERE embedding <=> :q < 0.3          -- threshold: chỉ nhận đủ gần
ORDER BY embedding <=> :q             -- + LIMIT để DÙNG ANN index
LIMIT 5;
```

**(d) Chống overfiltering khi filter chặt:**
```sql
SET hnsw.iterative_scan = relaxed_order;   -- pgvector 0.8+
```

## BẢNG — Khung system design ("tìm sản phẩm tương tự", max 8, không rác, filter còn-hàng/category, p95<100ms)

1. **Clarify:** metric (cosine)? có golden set để calibrate không? tần suất đổi model?
2. **Query shape:** `WHERE in_stock AND category=:c AND id!=:self AND embedding <=> :q < :threshold ORDER BY embedding <=> :q LIMIT 8` — threshold "không rác", LIMIT cap, filter + iterative_scan chống overfiltering.
3. **Threshold:** hiệu chỉnh trên golden set (cặp giống/khác), chọn ngưỡng cân precision/recall; version hóa theo model.
4. **Index & latency:** HNSW `vector_cosine_ops`; tune `ef_search` theo p95; đảm bảo `ORDER BY <=> LIMIT` để dùng index (không threshold thuần).
5. **Correctness:** two-stage nếu cần chắc (ANN rộng → exact + threshold).
6. **UX:** sau threshold còn 0 sản phẩm → ẩn khối "tương tự" thay vì show rác.
7. **Monitor:** phân phối distance, tỉ lệ 0-kết-quả, p99, recall trên golden set.
8. **Tự đánh giá:** LIMIT giới hạn *số lượng*, threshold đảm bảo *chất lượng*; phân biệt "gần nhất" vs "đủ tốt" = tư duy staff.

## BẢNG — Mental models

- **"Gần nhất ≠ đủ tốt"** → LIMIT vs threshold.
- **"Distance nhỏ = giống, đừng đảo"** → `<=>` nhỏ là gần; `<#>` âm.
- **"Ngưỡng là đường biên quyết định, không phải số phép màu"** → calibrate trên dữ liệu.
- **"k-NN thì index, range thuần thì seq scan"** → luôn ORDER BY + LIMIT.
- **"Cùng model hay vô nghĩa"** → vector hai model không so được.

---

## TỪ KHÓA MỒI

- Hai yêu cầu query → **"cùng model, cùng length"**
- Đừng SELECT raw vector → **"so sánh không phải xem"**
- tsquery vs vector → **"<-> hai nghĩa"**
- Query đơn giản nhất → **"ORDER BY <=> LIMIT k"**
- Ba toán tử → **"<#> trả số âm"**
- Tìm giống bản ghi có sẵn → **"WHERE id != self"**
- Lấy distance/similarity → **"1 - (<=>) ×100"**
- Ba lỗi query → **"ops class không khớp"**
- Threshold gần nhất ≠ đủ tốt → **"LIMIT số lượng, threshold chất lượng"**
- Threshold phụ thuộc metric → **"không có ngưỡng vạn năng"**
- Bẫy index threshold thuần → **"WHERE distance < x → seq scan"**
- Threshold + filter → **"iterative_scan"**
- Query pattern lớn → **"pre-filter + hybrid RRF"**
- Golden set → **"đường biên quyết định"**
- Correctness ANN + threshold → **"two-stage exact"**
- Tổ chức & series → **"thà im lặng còn hơn rác"**

---

## ĐÃ PHỦ

**Yêu cầu & cơ bản:** cùng model+version (hai không gian → vô nghĩa), cùng length (tính distance cần cùng độ dài, đổi model → re-embed); đừng SELECT raw vector (mảng float vô dụng, query xoay quanh distance/similarity); tsquery vs vector (lexeme vs ngữ nghĩa, "ô tô"/"xe hơi", `<->` FOLLOWED BY vs L2); query đơn giản `ORDER BY <distance> LIMIT k` ASC khung xương.

**Ba toán tử:** `<->` L2 [0,∞) / `<#>` negative IP (âm, ASC vẫn gần) / `<=>` cosine [0,2]; ops class khớp; chọn metric text-cosine/normalize-IP/L2-magnitude; đính chính `<=>` = distance.

**Pattern query:** tìm giống bản ghi có sẵn (subquery embedding + `WHERE id != self` vì distance tới chính nó = 0); lấy distance/similarity ra SELECT; similarity = `1-(<=>)` ×100 để hiển thị %; 3 lỗi (quên id!=self, ops class không khớp → seq scan, nhầm distance/similarity → ORDER BY DESC đảo).

**Threshold (trọng tâm):** ORDER BY LIMIT luôn trả k dòng kể cả rác ("ít xa nhất trong đám xa"); LIMIT=số lượng vs threshold=chất lượng; metric-dependent (cosine [0,2] <0.2-0.4 rất giống, L2 [0,∞), IP magnitude) + dataset-dependent (số "5" không vạn năng); cosine dễ đặt nhất; hiệu chỉnh bằng cặp giống/khác.

**Bẫy index:** k-NN (ORDER BY LIMIT) dùng index; threshold + ORDER BY + LIMIT vẫn dùng; threshold THUẦN (WHERE distance < x) → seq scan tính mọi dòng vì HNSW/IVFFlat cho k-NN không range scan → luôn kèm ORDER BY distance LIMIT; EXPLAIN ANALYZE.

**Edge cases:** threshold + filter overfiltering → iterative_scan; threshold quá chặt → 0 kết quả (xử lý app); đổi model → ngưỡng cũ sai; `<#>` chưa normalize; NULL vector; similarity vs distance trong WHERE (distance giúp planner).

**Staff:** query pattern lớn (pre-filter + partition prune, hybrid vector+FTS RRF, pagination OFFSET tốn kém → top-k rerank, cache query embedding hash); golden set calibrate threshold 4 bước (gán nhãn → phân phối → tách → theo dõi) + precision/recall trade-off; correctness ANN approximate → two-stage exact + monitor recall; cost/latency (ef_search, cache, top-k nhỏ); monitoring (phân phối distance drift, tỉ lệ 0-kết-quả, seq-scan fallback); tổ chức (framing chất lượng/precision-recall, golden set là tài sản, UX 0-kết-quả là quyết định sản phẩm); khung system design 8 bước.

**Đính chính bài gốc:** `<=>` = cosine distance (không similarity); threshold metric/dataset-dependent; threshold thuần không dùng index; nhớ `WHERE id != self`.

**Học tiếp (bài nêu):** reranking sau retrieval (cross-encoder), query rewriting/expansion, đánh giá retrieval (precision@k/recall@k/nDCG) trên golden set.
