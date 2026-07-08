# CAPSTONE — Vector Databases với PostgreSQL: Tổng Hợp Toàn Khóa & Đi Phỏng Vấn

> **Nguồn gốc:** Video *"Congratulations / Course wrap-up"* — Course 3, IBM Vector Database Fundamentals. Video này **không có nội dung kỹ thuật mới** — nó tổng kết lại toàn khóa (lexeme/FTS, model tạo embedding, cosine similarity, pgvector cho non-text, guidelines INSERT/`<=>`/COPY/pg-promise, và housekeeping cuối khóa).
>
> Vì thế, thay vì soạn một giáo trình song song lặp lại, tôi làm **giáo trình CAPSTONE**: khâu cả 8 giáo trình trước thành *một hệ thống* — cho bạn tấm bản đồ tổng, phần "mọi mảnh ghép với nhau ra sao", một **capstone project** để build, và một **master cheatsheet** ôn nhanh cả khóa trước phỏng vấn. Toàn bộ **[LÀ PHẦN TỔNG HỢP/MỞ RỘNG NGOÀI BÀI GỐC]** vì bài gốc chỉ là recap.
>
> **Bộ 8 giáo trình của khóa (thứ tự học đề xuất):**
> 1. `giao-trinh-vector-embeddings.md` — tạo vector
> 2. `giao-trinh-vector-indexing.md` — làm search nhanh (binary search → IVFFlat/HNSW)
> 3. `giao-trinh-rdbms-vector-platforms.md` — chọn nền tảng (Postgres/MySQL/MariaDB)
> 4. `giao-trinh-pgvector.md` — store & query (khái niệm)
> 5. `giao-trinh-pgvector-hands-on.md` — cài & dùng pgvector
> 6. `giao-trinh-bulk-insert-pgvector.md` — nạp dữ liệu quy mô lớn
> 7. `giao-trinh-query-vector-pgvector.md` — query & threshold
> 8. `giao-trinh-postgres-fts.md` — full-text search & hybrid

---

## Phần 0 — 🗺️ Bản đồ toàn khóa: 8 mảnh là MỘT hệ thống

Cả khóa trả lời đúng một câu hỏi lớn: **"Làm sao biến PostgreSQL thành một hệ tìm-kiếm-theo-nghĩa (semantic search / RAG) chạy được ở production?"** Tám chủ đề không rời rạc — chúng là các trạm trên *một* đường ống dữ liệu:

```
        [1] EMBED                [2] INDEX            [3] PLATFORM
   text/ảnh → vector      làm k-NN search nhanh    Postgres? MySQL? MariaDB?
   (model AI)             (IVFFlat / HNSW)          → chọn pgvector
        │                        │                        │
        └──────────┬─────────────┴────────────┬───────────┘
                   ▼                           ▼
            [5] CÀI pgvector            [4] STORE model
         (Docker/apt/managed)      (schema: cột vector + metadata)
                   │
                   ▼
            [6] BULK INSERT ──────► [7] QUERY ──────► [8] HYBRID (＋FTS)
         (COPY, nạp trước           (ORDER BY <=>       keyword ＋ semantic
          index sau)                 LIMIT ＋ threshold)  fuse bằng RRF
                                          │
                                          ▼
                                  SEMANTIC SEARCH / RAG
```

**Đọc bản đồ:** bạn *tạo* vector (1) bằng một model AI, chọn cách *index* để tìm nhanh (2), chọn *nền tảng* để chứa (3 → pgvector), *thiết kế schema* (4), *cài* extension (5), *nạp* dữ liệu hiệu quả (6), *query* đúng và có kiểm soát chất lượng (7), và (tùy chọn) ghép với *full-text search* thành hybrid (8). Kết quả: một hệ retrieval hoàn chỉnh — nền của mọi ứng dụng RAG.

**Ba câu chốt xuyên suốt cả khóa (từ chính video tổng kết):**
- Embedding biến dữ liệu phức tạp (kể cả ảnh/audio/video, không chỉ text) thành vector so sánh được.
- **Cosine gần 1 = giống nhau** (lưu ý: toán tử `<=>` trả *distance* = 1 − similarity).
- pgvector cho **non-text** (khác `tsvector`/FTS chỉ cho text/keyword).

---

## Phần 1 — 🟢 BASIC: Toàn hệ thống trong một đoạn + ví dụ end-to-end nhỏ nhất

### 1.1. Giải thích cả khóa cho người mới trong 5 câu

Máy tính không hiểu chữ/ảnh. Ta dùng một **model AI** biến chúng thành **embedding** (mảng số nắm bắt ý nghĩa) — thứ giống nhau về nghĩa thì vector nằm gần nhau. Ta lưu embedding vào **PostgreSQL** nhờ extension **pgvector**, và dùng **index** (HNSW) để tìm "hàng xóm gần nhất" thật nhanh dù có hàng triệu vector. Khi người dùng hỏi, ta embed câu hỏi rồi tìm các bản ghi có vector gần nhất — đó là **semantic search**. Ghép thêm **full-text search** (tìm theo từ khóa) thành **hybrid** cho kết quả tốt nhất.

### 1.2. Ví dụ end-to-end nhỏ nhất (cả 8 mảnh trong ~15 dòng)

```sql
-- [5] CÀI + [4] SCHEMA
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE docs (id bigserial PRIMARY KEY, content text, embedding vector(384));

-- [6] NẠP (ở đây insert tay; production dùng COPY, nạp trước index sau)
INSERT INTO docs (content, embedding) VALUES ('...', '[...]'), ('...', '[...]');

-- [2] INDEX (sau khi nạp)
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);

-- [7] QUERY: 5 kết quả gần nhất, chỉ nhận "đủ giống" (threshold)
SELECT content, 1 - (embedding <=> :q) AS similarity
FROM docs
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q
LIMIT 5;
```
`:q` là embedding của câu truy vấn — do **[1] model** sinh ra bằng **cùng model** đã dùng để embed `docs`. Đó là cả pipeline thu nhỏ.

### ✅ Self-check Phần 1

1. Kể tên 4 bước từ "một câu chữ" tới "kết quả semantic search".
2. Vì sao câu truy vấn phải dùng *cùng model* với dữ liệu trong bảng?
3. Trong ví dụ trên, dòng nào là "index", dòng nào là "threshold"?

---

## Phần 2 — 🟡 INTERMEDIATE: Đi dọc pipeline, mỗi trạm một quyết định

Mỗi trạm có *một quyết định cốt lõi* — nắm được chuỗi quyết định này là nắm cả khóa.

| # | Trạm | Quyết định cốt lõi | Chốt nhanh |
|---|---|---|---|
| 1 | **Embed** | chọn model nào? | English→OpenAI 3-small; đa ngữ/tiếng Việt→BGE-M3; nhẹ/local→MiniLM. **Đổi model = re-embed tất cả.** |
| 2 | **Index** | index nào? | Nhỏ (<50K)→không index (exact). Vừa→**HNSW** (default). RAM ít/tĩnh→IVFFlat/DiskANN. |
| 3 | **Platform** | RDBMS-native hay dedicated? | Vector là *feature* + <50M→**pgvector**. Vector là *product*/tỷ vector→Pinecone/Milvus. |
| 4 | **Store** | schema thế nào? | Cột `vector(n)` (n khớp model) **cạnh** business data → hybrid query 1 câu. |
| 5 | **Cài** | cài kiểu gì? | **Docker/apt/managed**, không build source trên prod; pin version. Cài file ≠ `CREATE EXTENSION`. |
| 6 | **Nạp** | insert kiểu gì? | **COPY** nhanh nhất; **nạp trước, index sau**; idempotent qua temp+upsert. |
| 7 | **Query** | lấy ra thế nào? | `ORDER BY <=> LIMIT k` (số lượng) + **threshold** (chất lượng); loại chính nó. |
| 8 | **Hybrid** | keyword + semantic? | FTS (`tsvector`/`@@`) bắt từ khóa; vector bắt nghĩa; fuse **RRF**. |

**Mạch nối các trạm (ví dụ một sự thật đi xuyên nhiều trạm):**
- *"Cùng model"* xuất hiện ở trạm 1 (tạo), trạm 4 (schema chiều khớp), trạm 7 (query cùng model). Một sợi chỉ đỏ.
- *"Nạp trước index sau"* (trạm 6) chỉ có nghĩa vì hiểu index maintenance đắt (trạm 2).
- *"Threshold"* (trạm 7) phụ thuộc metric chọn ở trạm 1/4 (cosine [0,2] vs L2 ∞).

### ✅ Self-check Phần 2

1. Với dữ liệu tiếng Việt nhạy cảm, quyết định ở trạm 1 (model) và trạm 3 (platform) nên là gì?
2. Vì sao "nạp trước index sau" (trạm 6) liên quan tới hiểu biết ở trạm 2?
3. Một câu hybrid (trạm 8) gồm hai nhánh nào, fuse bằng gì?

---

## Phần 3 — 🔴 ADVANCED: Bốn chủ đề xuyên suốt (connective insights)

Đây là những ý *không thuộc riêng bài nào* mà xuyên qua nhiều bài — chính chúng phân biệt người học vẹt từng bài với người hiểu hệ thống.

### 3.1. Approximate ↔ Recall: cái giá của tốc độ

Từ trạm 2 (index) tới trạm 7 (query threshold): **ANN đổi recall lấy tốc độ**. Không index = exact = recall 100% nhưng `O(n)`. Có HNSW/IVFFlat = nhanh nhưng recall < 100% (bỏ sót vài hàng xóm thật). Threshold áp lên kết quả *xấp xỉ* → có thể sót. Ai cũng nói "vector search"; người hiểu nói được **"tôi đang ở đâu trên đường cong recall–speed, và tôi đo nó thế nào"** (so ANN với exact định kỳ).

### 3.2. Luận điểm "một Postgres cho tất cả"

Từ trạm 3 (platform) + trạm 8 (FTS): siêu năng lực của pgvector không phải tốc độ đỉnh mà là **vector sống cạnh business data** → filter/join/hybrid trong *một* câu SQL, *một* transaction ACID. Dedicated vector DB (Pinecone) buộc 2 hệ, 2 round-trip, đồng bộ. Kim chỉ nam: **"vector là feature hay là product?"** Feature → Postgres; product/tỷ-vector/đa-vùng → dedicated.

### 3.3. Chi phí ẩn lớn nhất: re-embedding & lock-in

Từ trạm 1 + trạm 3 + trạm 6: **đổi embedding model = re-embed toàn bộ corpus** (vector hai model khác không gian). Đây là quyết định kiến trúc *nặng hơn cả chọn index*. Hệ quả: chốt model sớm, version hóa cột (`embedding_v2`), backfill blue-green, ưu tiên **BYO-embedding + chuẩn mở** để tránh lock-in (in-DB embedding kiểu HeatWave tiện nhưng khóa chặt).

### 3.4. Hybrid: keyword và nghĩa bù nhau

Từ trạm 8 + trạm 1: FTS (lexeme) *chính xác với thuật ngữ/mã/tên riêng* nhưng **mù ngữ nghĩa** ("ô tô" ≠ "xe hơi"); vector *hiểu nghĩa* nhưng có thể trượt thuật ngữ hiếm. **Hybrid = cả hai, fuse bằng RRF** — kiến trúc retrieval mặc định 2026, và pgvector + FTS làm được ngay trong *một* Postgres. Đây là nơi hai nửa của khóa (keyword ở bài FTS, semantic ở phần còn lại) hội tụ.

### ✅ Self-check Phần 3

1. "ANN đổi recall lấy tốc độ" thể hiện ở những trạm nào? Đo recall bằng cách nào?
2. Vì sao re-embedding là quyết định nặng hơn chọn index?
3. Hybrid search giải quyết điểm yếu nào của mỗi phương pháp (FTS và vector)?

---

## Phần 4 — 🟣 STAFF LEVEL: Capstone — Thiết kế & Build hệ hoàn chỉnh

### 4.1. System design tổng: "Semantic search + RAG cho một sản phẩm thật"

> **"Thiết kế hệ semantic search/RAG cho 20 triệu tài liệu, đa ngôn ngữ (gồm tiếng Việt), dữ liệu khách hàng nhạy cảm, p95 < 150ms, cập nhật hàng ngày, không downtime, team backend nhỏ dùng PostgreSQL."**

Câu này gộp *mọi* quyết định của khóa. Hướng trả lời staff, đi theo pipeline:

1. **Clarify** (luôn hỏi trước): QPS, recall target, ràng buộc compliance, ngân sách, tần suất update.
2. **[1 Embed]** Nhạy cảm + đa ngữ → **self-host BGE-M3** (dữ liệu ở nhà, tiếng Việt tốt, BYO tránh lock-in); chunk tài liệu dài; batch trên GPU; cache theo hash.
3. **[3 Platform]** 20M < ngưỡng dedicated + vector là feature + đã có Postgres → **pgvector self-host** (không gửi API ngoài vì privacy).
4. **[4 Store]** Bảng `docs(tenant_id, content, lang, embedding vector(1024), ...)`; **partition theo tenant/lang**; B-tree trên metadata filter.
5. **[5 Cài]** Docker tag pinned hoặc gói OS; `CREATE EXTENSION` qua migration; đồng bộ version trên replica.
6. **[6 Nạp]** Nạp ban đầu bằng **COPY (binary)** vào bảng chưa index; **idempotent** (temp+upsert); cập nhật hàng ngày qua queue bất đồng bộ.
7. **[2 Index]** **HNSW** `vector_cosine_ops`, build **CONCURRENTLY** sau nạp với `maintenance_work_mem` cao + parallel; cân **quantization/DiskANN** nếu RAM căng.
8. **[7 Query]** `WHERE <filter> AND embedding <=> :q < :thr ORDER BY embedding <=> :q LIMIT k`; tune `ef_search` theo p95; **iterative_scan** chống overfiltering; threshold hiệu chỉnh trên **golden set**.
9. **[8 Hybrid]** FTS (`tsvector` GIN) + vector, fuse **RRF** cho recall tốt hơn; tiếng Việt cân nhắc `simple`+unaccent hoặc dựa nhiều vào nhánh vector.
10. **Vận hành:** monitor recall@k (so exact), p99, WAL/replica lag, drift phân phối distance; blue-green khi đổi model; kế hoạch re-embed versioned.
11. **Tự đánh giá (chốt staff):** "Điểm nghẽn là *embedding generation* không phải insert; privacy *ép* self-host bất kể benchmark; 20M chọn pgvector nhưng tôi biết *ngưỡng* (hàng trăm triệu + đa vùng) sẽ khiến tôi cân nhắc dedicated." → **biết lý do & giới hạn của mọi lựa chọn.**

### 4.2. Capstone Project — build để đưa vào portfolio (như video gợi ý)

Video khuyên "chia sẻ lab trong repo công khai để showcase với nhà tuyển dụng". Đây là spec một project trọn vẹn đáng để build:

**"Semantic + Hybrid Search Engine trên PostgreSQL"**

- **Mục tiêu:** tìm kiếm theo nghĩa trên một corpus thật (vd Wikipedia dump, hoặc tập FAQ/sản phẩm), có cả keyword lẫn semantic.
- **Stack:** PostgreSQL + pgvector (Docker), model embedding self-host (sentence-transformers/BGE-M3), API bằng FastAPI/Express.
- **Các mảnh phải có (đúng 8 trạm của khóa):**
  1. Pipeline chunk + embed (batch).
  2. Schema `vector(n)` + metadata + GIN cho FTS.
  3. Nạp bằng **COPY**, nạp-trước-index-sau, idempotent.
  4. HNSW index + tune `ef_search`.
  5. Endpoint query: k-NN + **threshold** + filter.
  6. **Hybrid** endpoint: vector + FTS fuse **RRF**.
  7. `EXPLAIN ANALYZE` chứng minh dùng index; đo recall@k trên golden set nhỏ.
  8. README giải thích trade-off (vì sao HNSW, vì sao pgvector, ngưỡng chọn thế nào).
- **Điểm gây ấn tượng nhà tuyển dụng:** phần README nói về *trade-off và giới hạn* (recall vs speed, khi nào không dùng pgvector) — thể hiện tư duy staff, không chỉ "chạy được".

### 4.3. Ảnh hưởng tổ chức (tổng hợp)

Một câu framing cho stakeholder gói cả khóa: *"Chúng ta thêm tìm-kiếm-theo-nghĩa vào chính database đang chạy — không dựng hệ thống mới, đội hiện tại vận hành được, dữ liệu ở nguyên một chỗ. Chi phí lớn nhất không phải lưu trữ mà là (a) để AI sinh embedding và (b) nếu sau này đổi mô hình thì phải xử lý lại toàn bộ dữ liệu — nên ta chốt lựa chọn sớm và làm pipeline chạy-lại-được."* → **rủi ro thấp, chi phí rõ, ít lock-in.**

---

## Phần 5 — 🎯 MASTER CHEATSHEET: Ôn nhanh CẢ KHÓA trước phỏng vấn

> Nếu chỉ kịp đọc một trang trước buổi phỏng vấn, đọc phần này. Chi tiết từng chủ đề: mở file tương ứng trong bộ 8.

### 5.1. 20 keyword sống còn (cả khóa)

**Embedding** (vector mã hóa nghĩa) · **dimension** · **static vs contextual** (Word2Vec vs transformer) · **cosine similarity/distance** (`<=>` trả distance = 1−sim) · **L2/inner product** (`<->`/`<#>`) · **k-NN / ANN** · **recall** · **HNSW** (graph, ~O(log n)) · **IVFFlat** (centroid/list/probe) · **DiskANN** · **curse of dimensionality** · **pgvector** (extension) · **ops class** (khớp toán tử) · **tsvector/tsquery** (FTS lexeme, khác vector) · **GIN** (inverted index) · **hybrid search / RRF** · **COPY** (bulk nhanh nhất) · **threshold** (metric/dataset-dependent) · **re-embedding** (đổi model = làm lại hết) · **BYO vs in-DB embedding**.

### 5.2. 12 core concept (nếu chỉ nhớ ngần này)

1. Model tạo vector; **pgvector chỉ lưu & tìm**, không sinh embedding.
2. **Static vs contextual** — transformer thắng vì hiểu ngữ cảnh (attention).
3. Cosine đo **hướng** (nghĩa), bỏ độ dài; `<=>` là **distance**, sim = `1 - <=>`.
4. Không index = exact (recall 100%, O(n)); **ANN đổi recall lấy tốc độ**.
5. **HNSW** default (recall cao, tốn RAM); **IVFFlat** nhẹ (cần data, ngại data động).
6. B-tree/binary search **không dùng cho vector** (không sắp thứ tự 1 trục; curse of dimensionality) → cần ANN.
7. Siêu năng lực pgvector: **vector cạnh business data → hybrid query 1 câu SQL**.
8. **Vector là feature → Postgres; là product/tỷ-vector → dedicated.**
9. **COPY nhanh nhất**; **nạp trước, index sau**; idempotent qua temp+upsert.
10. Query: `ORDER BY <=> LIMIT k` (số lượng) + **threshold** (chất lượng); loại chính nó; threshold thuần bỏ index.
11. **Hybrid = FTS (keyword) + vector (nghĩa), fuse RRF** — default 2026.
12. **Đổi model = re-embed tất cả** — quyết định nặng nhất; **privacy có thể ép self-host** bất kể benchmark.

### 5.3. Code thuộc lòng (cả vòng đời)

```sql
-- Cài + schema + index
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE docs (id bigserial PRIMARY KEY, content text, embedding vector(1536));
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);

-- Nạp nhanh
\copy docs (content, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)

-- Query: k-NN + threshold + similarity%
SELECT content, 1 - (embedding <=> :q) AS sim
FROM docs
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q LIMIT 5;

-- Hybrid (RRF, rút gọn)
WITH kw AS (SELECT id, row_number() OVER (ORDER BY ts_rank(sv,q) DESC) r
            FROM docs, websearch_to_tsquery('english',:txt) q WHERE sv @@ q LIMIT 50),
     sem AS (SELECT id, row_number() OVER (ORDER BY embedding <=> :qv) r
            FROM docs ORDER BY embedding <=> :qv LIMIT 50)
SELECT id, SUM(1.0/(60+r)) rrf FROM (SELECT * FROM kw UNION ALL SELECT * FROM sem) t
GROUP BY id ORDER BY rrf DESC LIMIT 10;
```
```python
# Tạo embedding (cùng model cho DB và query!)
from sentence_transformers import SentenceTransformer
m = SentenceTransformer("all-MiniLM-L6-v2")
vec = m.encode(text, normalize_embeddings=True)
```

### 5.4. 12 câu phỏng vấn "trải cả khóa" + chốt

1. **Embedding là gì?** → vector mã hóa nghĩa; ghi điểm bằng **static vs contextual**.
2. **Cosine vs L2?** → cosine đo hướng (text); normalize thì L2≡cosine về xếp hạng.
3. **[Bẫy] `<=>` là similarity?** → không, **distance**; sim = `1 - <=>`.
4. **HNSW vs IVFFlat?** → graph ~O(log n), recall cao, tốn RAM vs centroid, nhẹ, cần data.
5. **[Bẫy] Sao không dùng B-tree cho vector?** → không sắp thứ tự 1 trục; curse of dimensionality → ANN.
6. **pgvector vs Pinecone?** → feature vs product; Postgres thắng operational simplicity + hybrid SQL.
7. **[Bẫy] `tsvector` = vector search?** → không, đó là FTS (lexeme); semantic cần pgvector `vector`.
8. **Bulk load nhanh nhất?** → **COPY** vào bảng chưa index → build index sau; idempotent.
9. **[Bẫy] Threshold `WHERE <=> < x` có nhanh?** → thuần thì seq scan; phải kèm `ORDER BY <=> LIMIT k`.
10. **Đổi model thì sao?** → **re-embed toàn bộ**; version hóa, blue-green.
11. **[Scale] Tỷ vector, RAM ít?** → quantization/DiskANN + two-stage re-rank + partition; cân dedicated nếu đa vùng.
12. **Hybrid search là gì?** → FTS + vector fuse **RRF**; keyword chính xác + nghĩa gần.

### 5.5. One-liner "đắt giá" tổng hợp

- *"The model makes the vector, the database stores it — pgvector never embeds anything itself."*
- *"You can't binary-search vectors — there's no single line to sort along, so we approximate instead."*
- *"pgvector wins on operational simplicity, not peak speed: choose it when vector search is a feature, not the product."*
- *"COPY into an unindexed table, then build the index — never make Postgres maintain an HNSW graph mid-load."*
- *"LIMIT caps how many; threshold guarantees how good — and a bare threshold quietly drops the index."*
- *"Changing the embedding model means re-embedding everything — it's the heaviest decision in the whole stack."*
- *"FTS knows words, vectors know meaning — hybrid is both, fused with RRF, inside one Postgres."*

---

### 📌 Ghi chú cuối & Next steps

- **Bạn giờ có bộ 9 file** (8 chủ đề + capstone này) phủ trọn: tạo vector → index → nền tảng → store → cài → nạp → query → hybrid → tổng hợp. Ôn theo thứ tự Phần 0, dùng master cheatsheet (Phần 5) để nước rút trước phỏng vấn.
- **Kiểm chứng khi ôn:** mảng này đổi nhanh — trước phỏng vấn xem lại pgvector README, MTEB leaderboard (model), và docs Postgres FTS cho con số/cú pháp mới nhất.
- **Build capstone project** (Phần 4.2) và đưa lên GitHub công khai — đúng như video khuyên, một repo "semantic + hybrid search" với README bàn *trade-off* là minh chứng kỹ năng thuyết phục hơn mọi chứng chỉ.
- **Học tiếp (nâng cao khóa này):** RAG end-to-end (retrieval → rerank → LLM synthesis), reranking bằng cross-encoder, chunking strategy, evaluation retrieval (recall@k/nDCG/MRR trên golden set), multimodal embeddings, và sharding vector (Citus/pgvectorscale) khi vượt một node.
- **Chúc mừng hoàn thành khóa.** Bạn đã đi từ "vector là gì" tới "thiết kế & vận hành một hệ semantic search production trên PostgreSQL" — đủ chiều sâu để đóng góp thật và tự tin đi phỏng vấn big tech.

---

> *Muốn tôi gộp toàn bộ 9 file thành một cuốn giáo trình duy nhất (có mục lục, đánh số chương, đường học tuyến tính) để in/ôn liền mạch? Chỉ cần nói một tiếng.*
