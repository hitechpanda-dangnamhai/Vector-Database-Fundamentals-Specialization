# Query Dữ Liệu Vector trong pgvector: Toán Tử Distance & Threshold — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"Perform vector queries, including using pgvector for data queries"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy: yêu cầu khi query vector, viết `tsquery` cho `tsvector`, ba toán tử distance (`<->`, `<#>`, `<=>`), search theo cosine/distance, và **giới hạn bằng threshold**. Tôi giảng bám sát rồi đào sâu. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là mảnh *"lấy dữ liệu ra ĐÚNG"* — hoàn tất bộ hands-on: cài (#6) → nạp (#7) → **query (bài này)**. Toán tử distance và tsquery đã xuất hiện ở giáo trình pgvector khái niệm (#3) và FTS (#6); ở đây tôi tập trung vào *cách viết query thực tế* và đào sâu phần **threshold** mà các bài trước chưa kỹ.
>
> **⚠️ Đính chính tới 2026 (bài gốc có chỗ chưa chuẩn):**
> 1. **`<=>` là cosine *distance*, không phải cosine *similarity*.** Bài gốc nói "cosine similarity to retrieve records" nhưng toán tử trả **distance** (`= 1 − similarity`, miền [0,2]). Muốn similarity thì lấy `1 - (embedding <=> q)`.
> 2. **Threshold là *metric-dependent* và *dataset-dependent*.** Số "5" trong ví dụ L2 của bài gốc không phải hằng số vạn năng — mỗi metric và mỗi bộ dữ liệu có ngưỡng khác nhau (cosine distance ∈ [0,2]; L2 không giới hạn).
> 3. **Có một cái bẫy về index:** query k-NN nhanh nhờ `ORDER BY distance LIMIT k`. Threshold *thuần* (`WHERE distance < x` mà không ORDER BY/LIMIT) thường **không dùng được ANN index** → seq scan. Bài gốc không nhắc.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Cách *viết truy vấn* để lấy ra những bản ghi giống nhất với một vector — dùng đúng toán tử distance, đúng pattern (tìm giống một bản ghi có sẵn, loại chính nó, lấy điểm similarity), và cách **lọc bằng threshold** để chỉ nhận kết quả "đủ gần". Kèm yêu cầu bắt buộc: cùng model, cùng số chiều.

**Vấn đề nó giải quyết:** Query vector viết *gần giống* SQL thường, nhưng có những chỗ dễ sai làm kết quả vô nghĩa: SELECT thẳng cột vector (ra mảng số vô dụng), dùng sai toán tử/metric, đặt threshold bừa, hoặc viết query khiến index bị bỏ. Bài này gỡ từng chỗ.

**Học xong bạn sẽ làm được:**

- Nêu yêu cầu khi query vector (cùng model, cùng length) và vì sao không SELECT raw vector.
- Dùng đúng ba toán tử `<->` / `<#>` / `<=>` và biết khi nào chọn cái nào.
- Viết query "tìm k bản ghi giống nhất với bản ghi X" (loại chính nó, lấy distance/similarity).
- Lọc bằng **threshold** đúng cách, hiểu ngưỡng phụ thuộc metric & dataset, tránh bẫy index.
- Áp query pattern ở quy mô lớn: pre-filter, hybrid, pagination, hiệu chỉnh threshold thực nghiệm.

**Mạch basic → staff:**

- 🟢 **Basic:** Yêu cầu query (cùng model/length) → tsquery cho text vs vector cho embedding → query similarity đơn giản nhất.
- 🟡 **Intermediate:** Ba toán tử trong ngữ cảnh query; pattern "giống bản ghi có sẵn" (subquery); lấy distance/similarity ra SELECT; loại chính nó; lỗi thường gặp.
- 🔴 **Advanced:** Threshold sâu (metric-dependent, hiệu chỉnh, bẫy index); đổi distance→similarity %; threshold + LIMIT + iterative scan; edge cases.
- 🟣 **Staff:** Query pattern quy mô lớn (pre-filter, hybrid, pagination, calibrate threshold, cache, monitor); hiệu chỉnh ngưỡng bằng cả mẫu giống lẫn khác; câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Hai yêu cầu bắt buộc khi query vector

1. **Cùng model.** Embedding lưu trong DB và embedding của câu tìm kiếm **phải do cùng một model + version** sinh ra. Vector từ hai model khác nhau nằm ở *hai không gian khác nhau* → distance vô nghĩa, kết quả loạn. (Bài gốc nhấn đúng điểm này.)
2. **Cùng số chiều (length).** Để tính distance, hai vector phải cùng độ dài. Model quyết định số chiều; đổi model = đổi chiều = phải re-embed toàn bộ (xem giáo trình embeddings).

### 1.2. Đừng SELECT raw vector — nó vô nghĩa với con người

```sql
SELECT embedding FROM reviews LIMIT 1;
-- => [0.0123, -0.87, 0.44, ... 512 số ...]   <- con người không đọc hiểu gì
```

Vector chỉ là mảng float khổng lồ; lấy ra *nguyên trạng* chẳng cho thông tin gì và làm hỏng mục đích. Cái ta muốn không phải *xem* vector, mà *so sánh* nó để tìm bản ghi giống nhau. Vậy query luôn xoay quanh **distance/similarity**, không phải bản thân cột vector.

### 1.3. Hai loại "search văn bản" — đừng nhầm

Bài gốc nhắc cả `tsquery` lẫn pgvector, dễ lẫn. Phân biệt rõ:

| | `tsquery` trên `tsvector` | pgvector (`<=>`...) |
|---|---|---|
| Tìm theo | **lexeme** (từ khóa chuẩn hóa) | **ngữ nghĩa** (embedding) |
| Hợp cho | keyword search chính xác | semantic search (nghĩa gần) |
| "red" khớp "reddish"? | có (cùng lexeme) | có nếu nghĩa gần |
| "ô tô" khớp "xe hơi"? | không | **có** |
| Chi tiết | giáo trình FTS (#6) | bài này |

Ví dụ tsquery (bài gốc): chuỗi *"a red mat costs as much as a blue mat"* tìm có cả `mat` và `red` → true; muốn `red` *đứng ngay trước* `mat` thì dùng toán tử **FOLLOWED BY** (`<->` trong tsquery):
```sql
SELECT to_tsvector('english','a red mat costs as much as a blue mat')
    @@ to_tsquery('english','red <-> mat');   -- red ngay trước mat? -> ở đây false (có "red mat" thì true)
```
(Lưu ý `<->` trong *tsquery* nghĩa là FOLLOWED BY, khác `<->` của *vector* là L2 distance — trùng ký hiệu, khác ngữ cảnh.)

Phần còn lại của bài này về **pgvector** (semantic).

### 1.4. Query similarity đơn giản nhất

```sql
-- "5 review giống nhất với một embedding cho trước"
SELECT id, content
FROM reviews
ORDER BY embedding <=> '[...embedding truy vấn...]'   -- <=> cosine distance
LIMIT 5;
```

Mấu chốt: `ORDER BY <toán tử distance> LIMIT k`. Mặc định `ORDER BY` là **ASC** → distance nhỏ nhất (giống nhất) lên đầu. Đây là khung xương của mọi vector query.

### ✅ Self-check Phần 1

1. Vì sao phải dùng cùng model cho embedding trong DB và embedding câu truy vấn?
2. SELECT thẳng cột vector cho ra cái gì? Vì sao query nên xoay quanh distance thay vì cột vector?
3. `<->` trong tsquery và `<->` trong vector — nghĩa có giống nhau không?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. Ba toán tử distance trong ngữ cảnh query

| Toán tử | Metric | Ghi chú | Ops class index |
|---|---|---|---|
| `<->` | L2 / Euclidean distance | khoảng cách thẳng; không giới hạn trên | `vector_l2_ops` |
| `<#>` | Negative inner product | pgvector trả *âm* IP (để ASC = gần) | `vector_ip_ops` |
| `<=>` | Cosine distance | phổ biến nhất với text; miền [0,2] | `vector_cosine_ops` |

> **[Đính chính bài gốc]** Bài gốc gọi câu dùng `<=>` là "cosine similarity". Chính xác `<=>` trả **cosine distance**. Với `<#>`, pgvector trả **negative** inner product (nên `ORDER BY ... ASC` vẫn cho gần nhất) — đừng ngạc nhiên khi thấy số âm.

**Chọn metric:** text embeddings → cosine (`<=>`) mặc định; nếu vector đã normalize length=1 → inner product (`<#>`) nhanh hơn; L2 khi độ lớn vector có nghĩa. (Chi tiết: giáo trình embeddings.)

### 2.2. Pattern kinh điển: "tìm giống một bản ghi ĐÃ CÓ" (bám ví dụ bài gốc)

Bài gốc: lấy 5 review giống nhất với review `id=1`. Vector truy vấn chính là embedding của một dòng có sẵn → dùng subquery:

```sql
-- Euclidean
SELECT *
FROM reviews
WHERE id != 1                                              -- loại CHÍNH NÓ (distance=0)
ORDER BY embedding <-> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;

-- Inner product
SELECT *
FROM reviews
WHERE id != 1
ORDER BY embedding <#> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;

-- Cosine
SELECT *
FROM reviews
WHERE id != 1
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;
```

Hai điểm quan trọng:
- **`WHERE id != 1`**: nếu không loại, chính review 1 luôn đứng đầu (distance tới chính nó = 0) — vô dụng. Đây là bước bài gốc nêu đúng.
- **Subquery lấy embedding**: `(SELECT embedding FROM reviews WHERE id=1)` cắm vector của dòng nguồn vào phép so.

### 2.3. Lấy giá trị distance/similarity ra để đọc

Thường bạn muốn *thấy* điểm số, không chỉ thứ tự:

```sql
SELECT id, content,
       embedding <=> :q            AS cosine_distance,     -- 0 = giống hệt, 2 = ngược
       1 - (embedding <=> :q)      AS cosine_similarity     -- 1 = giống hệt, -1 = ngược
FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;
```

> **[MỞ RỘNG]** Để hiển thị cho người dùng dạng "độ giống %", dùng `1 - (embedding <=> q)` (cosine similarity) rồi ×100. Đừng show thẳng cosine *distance* rồi bảo "độ giống" — ngược nghĩa (distance nhỏ mới là giống).

### 2.4. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — Quên loại chính bản ghi nguồn** (`WHERE id != 1`) → kết quả đầu bảng là chính nó, che mất một kết quả thật.

**Lỗi 2 — Ops class index không khớp toán tử query.** Index `vector_cosine_ops` nhưng query `<->` → Postgres bỏ index, seq scan âm thầm (chậm). Toán tử phải khớp ops class. Kiểm bằng `EXPLAIN ANALYZE`.

**Lỗi 3 — Nhầm distance với similarity khi diễn giải.** `<=>` nhỏ = giống; `<#>` là *âm* inner product. Sắp `ORDER BY DESC` vì tưởng "lớn = giống" → đảo ngược kết quả. Mặc định ASC là đúng cho cả ba (pgvector thiết kế vậy).

### ✅ Self-check Phần 2

1. Vì sao query "tìm giống review id=1" cần `WHERE id != 1`?
2. Làm sao đổi cosine distance thành "độ giống %" để hiển thị?
3. `<#>` trả về gì, và vì sao vẫn `ORDER BY ASC`?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Threshold filtering — chỉ nhận kết quả "đủ gần"

`ORDER BY ... LIMIT 5` *luôn* trả 5 dòng — kể cả khi 5 dòng đó chẳng liên quan gì (chỉ là "ít xa nhất trong đám xa"). Threshold khắc phục: **chỉ tính là match nếu distance/similarity trong ngưỡng**.

```sql
-- Chỉ nhận review có cosine distance < 0.3 (khá giống), tối đa 5
SELECT id, content, embedding <=> :q AS distance
FROM reviews
WHERE embedding <=> :q < 0.3          -- threshold
ORDER BY embedding <=> :q
LIMIT 5;
```

Bài gốc dùng ví dụ L2: `WHERE embedding <-> q < 5`. Ý tưởng đúng, nhưng con số cần bàn kỹ (3.2).

### 3.2. [QUAN TRỌNG] Threshold phụ thuộc METRIC và DATASET

Không có "ngưỡng vạn năng". Mỗi metric có miền giá trị khác nhau:

| Metric | Miền giá trị | "Gần" nghĩa là |
|---|---|---|
| Cosine distance (`<=>`) | **[0, 2]** | thường < 0.2–0.4 là rất giống |
| L2 (`<->`) | **[0, ∞)** không giới hạn | phụ thuộc thang & chuẩn hóa vector |
| Negative inner product (`<#>`) | phụ thuộc magnitude | khó đặt ngưỡng nếu chưa normalize |

Hệ quả:
- Threshold `< 5` cho L2 *chỉ đúng với dataset & model cụ thể* — đổi model/chuẩn hóa là số đó vô nghĩa.
- **Cosine dễ đặt ngưỡng hơn** vì miền cố định [0,2] → ngưỡng có thể tái dùng ổn định hơn L2.
- **Cách hiệu chỉnh (bài gốc gợi ý mờ, đây làm rõ):** chạy query trên vài cặp *biết trước là giống* và *biết trước là khác*, xem distance rơi vào khoảng nào, rồi chọn ngưỡng tách hai nhóm. Đây chính là ý "retrieve cả bản ghi rất ít/không giống để validate" của bài gốc — mục đích là *calibrate ngưỡng*, không phải để trả về cho user.

### 3.3. [BẪY INDEX] Threshold thuần không dùng được ANN index

Đây là bẫy bài gốc bỏ qua:

```sql
-- ✅ Dùng ANN index (k-NN): ORDER BY + LIMIT
SELECT * FROM reviews ORDER BY embedding <=> :q LIMIT 5;

-- ✅ Vẫn dùng index: threshold + ORDER BY + LIMIT
SELECT * FROM reviews
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q
LIMIT 5;

-- ⚠️ CÓ THỂ seq scan: threshold THUẦN, không ORDER BY/LIMIT
SELECT * FROM reviews WHERE embedding <=> :q < 0.3;   -- tính distance MỌI dòng
```

Lý do: HNSW/IVFFlat được thiết kế cho **k-nearest-neighbor** (`ORDER BY distance LIMIT k`), không phải range scan thuần. Một `WHERE distance < x` đơn độc buộc Postgres tính distance cho *mọi* dòng. Muốn vừa nhanh vừa có ngưỡng → **luôn kèm `ORDER BY distance LIMIT k`** rồi mới lọc threshold. Xác nhận bằng `EXPLAIN ANALYZE` (thấy `Index Scan` không).

### 3.4. Threshold + filter + overfiltering

Kết hợp threshold với filter nghiệp vụ (`WHERE category=...`) có thể gặp **overfiltering** (ANN lấy `ef_search` hàng xóm trước, lọc sau → thiếu kết quả). Khắc phục bằng iterative scan (pgvector 0.8+, xem giáo trình indexing):
```sql
SET hnsw.iterative_scan = relaxed_order;
```

### 3.5. Edge cases

- **Threshold quá chặt → 0 kết quả:** hợp lệ (không gì đủ gần) nhưng phải xử lý ở app (thông báo "không tìm thấy" thay vì lỗi).
- **Đổi model → ngưỡng cũ sai:** ngưỡng gắn với model+metric; re-embed thì phải hiệu chỉnh lại ngưỡng.
- **`<#>` với vector chưa normalize:** inner product bị magnitude làm nhiễu → ngưỡng khó tin cậy; normalize trước.
- **Vector NULL:** không xuất hiện trong kết quả distance → lọc/backfill.
- **So similarity vs distance trong WHERE:** `WHERE 1-(embedding<=>q) > 0.7` (ngưỡng similarity) tương đương `WHERE embedding<=>q < 0.3` — nhưng viết theo distance giúp planner tối ưu tốt hơn.

### ✅ Self-check Phần 3

1. Vì sao không có "ngưỡng threshold vạn năng"? Metric nào dễ đặt ngưỡng ổn định nhất?
2. `WHERE embedding <=> q < 0.3` (không ORDER BY/LIMIT) có dùng ANN index không? Nên viết thế nào?
3. Cách hiệu chỉnh threshold bằng "mẫu giống" và "mẫu khác" là gì?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Query pattern ở quy mô lớn

- **Pre-filter trước khi rank:** đặt filter chọn lọc (`tenant_id`, `category`, thời gian) để thu hẹp tập ứng viên *trước*, giảm chi phí distance. Kết hợp partition để planner prune sớm.
- **Hybrid query:** kết hợp vector (`<=>`) với full-text (`@@`) và fuse bằng RRF cho kết quả tốt hơn keyword-only hoặc semantic-only (xem giáo trình FTS). Đây là default retrieval 2026.
- **Pagination vector kết quả:** `ORDER BY distance LIMIT k OFFSET n` hoạt động nhưng OFFSET lớn tốn kém với ANN; thực tế thường chỉ lấy top-k rồi rerank, ít phân trang sâu.
- **Cache query embedding:** embedding của cùng câu truy vấn là xác định → cache theo hash để khỏi gọi model lại trong đường nóng.

### 4.2. Hiệu chỉnh threshold một cách khoa học (nâng cấp ý bài gốc)

Bài gốc gợi ý "retrieve cả bản ghi không giống để validate" nhưng mờ. Ở tầm staff, làm có hệ thống:

1. Dựng một **tập vàng (golden set)**: các cặp query–kết quả *đã gán nhãn* liên quan / không liên quan.
2. Chạy query, ghi lại distance của cặp liên quan vs không liên quan → vẽ phân phối.
3. Chọn threshold **tách hai phân phối** tốt nhất (đánh đổi precision vs recall theo nhu cầu: ngưỡng chặt → precision cao, bỏ sót nhiều; ngưỡng lỏng → recall cao, lẫn rác).
4. Theo dõi & tinh chỉnh khi dữ liệu/model đổi.

> **One-liner staff:** *"A threshold isn't a magic number you guess — it's a decision boundary you calibrate against labeled similar and dissimilar pairs."*

### 4.3. Correctness ở quy mô: ANN + threshold

- ANN là **approximate** → threshold áp lên kết quả *xấp xỉ*: có thể bỏ sót bản ghi thật sự trong ngưỡng nhưng ANN không tìm ra. Nếu cần đảm bảo (vd compliance), cân nhắc **two-stage**: ANN lấy top-N rộng → tính **exact** distance + áp threshold trên N đó.
- Monitor **recall** như ở giáo trình indexing: threshold không cứu được recall kém của index.

### 4.4. Cost / latency / monitoring

- **Latency:** `ef_search` cao → recall tốt hơn nhưng chậm; threshold không thay việc tune ef_search. Đo p95/p99.
- **Cost:** mỗi query real-time cần embed câu truy vấn (model) + search → cache embedding + top-k nhỏ.
- **Monitoring:** phân phối distance theo thời gian (drift → threshold cần chỉnh); tỉ lệ query trả 0 kết quả (threshold quá chặt?); tỉ lệ seq-scan fallback (query viết sai làm mất index).

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Kết quả search luôn *xếp hạng* theo độ giống. `LIMIT` bảo 'cho tôi 5 cái gần nhất' — nhưng '5 gần nhất' không có nghĩa '5 cái *tốt*'. Threshold là bộ lọc chất lượng: 'chỉ trả kết quả *đủ giống*, nếu không thà không trả'. Con số ngưỡng do ta hiệu chỉnh trên dữ liệu thật, không đoán bừa, và phải chỉnh lại nếu đổi mô hình AI." → framing bằng **chất lượng kết quả & sự đánh đổi precision/recall**.
- **Roadmap:** golden set để hiệu chỉnh threshold là tài sản dùng lại (còn để đo recall, so sánh model) → nên đầu tư xây và duy trì.
- **UX:** quyết định "trả 0 kết quả khi không đủ gần" là quyết định sản phẩm (thà im lặng còn hơn gợi ý rác) — staff nên nêu rõ với PM.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Xây 'tìm sản phẩm tương tự' cho e-commerce: hiển thị tối đa 8 sản phẩm *thực sự* giống, không hiển thị rác nếu không có; filter theo còn-hàng & cùng category; p95 < 100ms."**

**Khung trả lời staff:**

1. **Clarify:** metric (cosine cho embedding sản phẩm)? có golden set để calibrate threshold không? tần suất đổi model?
2. **Query shape:** `WHERE in_stock AND category=:c AND id!=:self AND embedding <=> :q < :threshold ORDER BY embedding <=> :q LIMIT 8` — threshold để "không rác", LIMIT để cap, filter nghiệp vụ + iterative_scan chống overfiltering.
3. **Threshold:** hiệu chỉnh trên golden set (cặp giống/khác), chọn ngưỡng cân precision/recall; version hóa theo model.
4. **Index & latency:** HNSW `vector_cosine_ops`; tune `ef_search` theo p95; đảm bảo `ORDER BY <=> LIMIT` để dùng index (không threshold thuần).
5. **Correctness:** two-stage nếu cần chắc (ANN rộng → exact + threshold).
6. **UX:** nếu sau threshold còn 0 sản phẩm → ẩn khối "tương tự" thay vì show rác.
7. **Monitor:** phân phối distance, tỉ lệ 0-kết-quả, p99, recall trên golden set.
8. **Tự đánh giá:** "LIMIT giới hạn *số lượng*, threshold đảm bảo *chất lượng*; hai cái phối hợp mới ra trải nghiệm tốt." → **phân biệt được 'gần nhất' vs 'đủ tốt' = tư duy staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **Same model / same length** — yêu cầu bắt buộc để distance có nghĩa.
- **`<->` / `<#>` / `<=>`** — L2 / negative inner product / cosine distance.
- **ops class** — `vector_l2_ops` / `vector_ip_ops` / `vector_cosine_ops` (khớp toán tử).
- **k-NN query** — `ORDER BY distance LIMIT k`; khung xương mọi vector query.
- **Cosine distance vs similarity** — `<=>` trả distance [0,2]; similarity = `1 - distance`.
- **Threshold** — lọc "đủ gần"; metric-dependent & dataset-dependent.
- **`WHERE id != self`** — loại chính bản ghi nguồn khi tìm "giống nó".
- **Overfiltering / iterative scan** — thiếu kết quả khi filter chặt; `hnsw.iterative_scan`.
- **Two-stage retrieval** — ANN rộng → exact + threshold để đảm bảo correctness.
- **Golden set** — tập gán nhãn để hiệu chỉnh threshold & đo recall.
- **tsquery vs vector** — lexeme keyword vs semantic (đừng nhầm; `<->` hai nghĩa theo ngữ cảnh).

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Query vector = `ORDER BY <distance> LIMIT k`; ASC → gần nhất lên đầu.
2. Đừng SELECT raw vector (vô nghĩa); query xoay quanh **distance/similarity**.
3. Cùng **model + length** cho cả DB lẫn query, nếu không kết quả loạn.
4. `<=>` là **cosine distance** (nhỏ=gần); similarity = `1 - (<=>)`.
5. Tìm "giống bản ghi X": subquery embedding + **`WHERE id != X`**.
6. **Threshold** = bộ lọc chất lượng; LIMIT = giới hạn số lượng — hai việc khác nhau.
7. Ngưỡng **phụ thuộc metric & dataset**; cosine [0,2] dễ đặt ổn định hơn L2 (∞).
8. **Threshold thuần không dùng ANN index** → luôn kèm `ORDER BY distance LIMIT k`.
9. ANN approximate → threshold áp lên kết quả xấp xỉ; cần chắc thì two-stage exact.
10. Hiệu chỉnh threshold bằng **golden set** (cặp giống/khác), không đoán bừa.

### 5.3. Ideas / mental models

- **"Gần nhất ≠ đủ tốt":** LIMIT vs threshold.
- **"Distance nhỏ = giống, đừng đảo":** `<=>` nhỏ là gần; `<#>` âm.
- **"Ngưỡng là đường biên quyết định, không phải số phép màu":** calibrate trên dữ liệu.
- **"k-NN thì index, range thuần thì seq scan":** luôn ORDER BY + LIMIT.
- **"Cùng model hay vô nghĩa":** vector hai model không so được.

### 5.4. Code cần thuộc lòng

**(a) k-NN cơ bản:**
```sql
SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;
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
WHERE embedding <=> :q < 0.3          -- threshold
ORDER BY embedding <=> :q
LIMIT 5;
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Viết query tìm 5 bản ghi giống nhất?"** → `SELECT ... ORDER BY embedding <=> :q LIMIT 5`. ASC → gần nhất trước. Nếu tìm giống một dòng có sẵn thì subquery embedding + `WHERE id != that`.

2. **[BẪY] "`<=>` trả cosine similarity đúng không?"** → Không, trả **cosine distance** (1 − similarity). Similarity = `1 - (embedding <=> q)`.

3. **"Threshold để làm gì, đặt số bao nhiêu?"** → Lọc kết quả "đủ gần" (LIMIT chỉ giới hạn số lượng). Số phụ thuộc metric & dataset; hiệu chỉnh trên golden set; cosine [0,2] dễ đặt hơn L2.

4. **[BẪY] "`WHERE embedding <=> q < 0.3` có nhanh không?"** → Threshold thuần dễ thành seq scan (tính distance mọi dòng). Phải kèm `ORDER BY embedding <=> q LIMIT k` để dùng ANN index.

5. **[BẪY] "Tìm giống review id=1 mà kết quả đầu là chính review 1?"** → Quên `WHERE id != 1`; distance tới chính nó = 0 nên luôn đứng đầu.

6. **[TRADE-OFF] "Threshold chặt vs lỏng?"** → Chặt: precision cao, bỏ sót nhiều (recall thấp), dễ 0 kết quả. Lỏng: recall cao, lẫn rác. Chọn theo nhu cầu sản phẩm; calibrate trên golden set.

7. **[SCALE] "'Sản phẩm tương tự' không được hiện rác?"** → threshold (chất lượng) + LIMIT (số lượng) + filter nghiệp vụ + iterative_scan; nếu sau threshold 0 kết quả thì ẩn khối, đừng show rác.

8. **"ANN + threshold có đảm bảo đúng không?"** → Không tuyệt đối (ANN approximate, có thể bỏ sót). Cần chắc → two-stage: ANN lấy top-N rộng rồi tính exact + áp threshold.

### 5.6. One-liner đắt giá

- *"`ORDER BY distance LIMIT k` is the backbone of every vector query — ascending, because smaller distance means closer."*
- *"`<=>` returns cosine distance, not similarity; similarity is `1 - (embedding <=> q)`."*
- *"LIMIT caps how many; threshold guarantees how good — the nearest results aren't automatically good ones."*
- *"A bare `WHERE distance < x` bypasses the ANN index; always pair it with ORDER BY distance LIMIT k."*
- *"A threshold is a decision boundary you calibrate on labeled similar/dissimilar pairs, not a number you guess."*
- *"When finding items similar to an existing row, exclude the row itself — its distance to itself is zero."*

---

### 📌 Ghi chú cuối

- **Đính chính để nhớ đúng:** `<=>` = cosine **distance** (không phải similarity); threshold **metric/dataset-dependent**; threshold thuần **không dùng index** (luôn ORDER BY + LIMIT); nhớ `WHERE id != self`.
- **Kiểm chứng khi ôn:** xem pgvector README cho toán tử/ops class và ví dụ query; cách iterative scan tương tác filter (pgvector 0.8+).
- **Thực hành:** với bảng `reviews(embedding vector(384))` đã nạp (giáo trình bulk insert), chạy cả ba toán tử tìm giống review id=1, thêm cột `1-(<=>)` xem similarity, rồi thử threshold chặt/lỏng và `EXPLAIN ANALYZE` để thấy khi nào `Index Scan` biến mất.
- **Nối mạch series (đủ 8 mảnh):** embed → index → cài → nạp → **query (bài này)** → store/query khái niệm (#3) → FTS → chọn nền tảng. Bạn giờ *viết được* truy vấn semantic search đúng và có kiểm soát chất lượng.
- **Học tiếp:** reranking sau retrieval (cross-encoder), query rewriting/expansion, và đánh giá chất lượng retrieval có hệ thống (precision@k, recall@k, nDCG) trên golden set.
