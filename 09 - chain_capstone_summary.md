# CAPSTONE — Vector Databases với PostgreSQL: Tổng Hợp Toàn Khóa — Chain gối đầu

> Bài tổng kết (không có kiến thức kỹ thuật MỚI): khâu 8 bài trước thành MỘT hệ thống. Chi tiết từng chủ đề nằm ở 8 file chain tương ứng; ở đây là *bản đồ tổng*, *sợi chỉ đỏ xuyên khóa*, và *master cheatsheet*.
>
> **Bộ 8 mảnh (thứ tự học):** 1 embeddings (tạo vector) · 2 indexing (làm nhanh) · 3 chọn nền tảng · 4 pgvector khái niệm (store & query) · 5 cài pgvector · 6 bulk insert · 7 query & threshold · 8 FTS & hybrid.

---

## CHAIN — Câu hỏi lớn & đường ống 8 trạm

cả khóa trả lời MỘT câu hỏi: làm sao biến PostgreSQL thành hệ semantic search / RAG chạy được ở production => tám chủ đề không rời rạc mà là các trạm trên MỘT đường ống dữ liệu => trạm 1 **embed** sinh ra vector từ text/ảnh bằng model AI => vector cần **index** để k-NN search nhanh (trạm 2, IVFFlat/HNSW) => index đó phải nằm trên một nền tảng, chọn **platform** (trạm 3 → pgvector) => trên pgvector thiết kế **schema** cột vector cạnh metadata (trạm 4 store) => schema cần pgvector được **cài** trước (trạm 5, Docker/apt/managed) => cài xong thì **nạp** dữ liệu vào (trạm 6 bulk insert: COPY, nạp trước index sau) => nạp xong thì **query** lấy ra (trạm 7: ORDER BY `<=>` LIMIT + threshold) => query semantic ghép thêm keyword thành **hybrid** (trạm 8: FTS + vector fuse RRF) => tám trạm hợp thành hệ semantic search / RAG hoàn chỉnh

## CHAIN — Ba câu chốt xuyên khóa

câu chốt 1: **embedding** biến dữ liệu phức tạp — kể cả ảnh/audio/video, không chỉ text — thành vector so sánh được => câu chốt 2: cosine gần 1 = giống nhau, NHƯNG toán tử `<=>` trả *distance* = 1 − similarity (nhỏ = gần) => câu chốt 3: pgvector cho **non-text** (embedding), khác `tsvector`/FTS vốn chỉ cho text/keyword => ba câu này là sợi tư tưởng chạy suốt 8 trạm

## CHAIN — Sợi chỉ đỏ "cùng model"

"cùng model" không thuộc riêng trạm nào mà xuyên nhiều trạm => trạm 1: embedding trong DB do một model sinh => trạm 4: schema `vector(n)` với n khớp đúng số chiều model đó => trạm 7: câu truy vấn phải embed bằng CÙNG model mới so được distance => vì vector hai model nằm ở hai không gian khác nhau => nên đổi model = re-embed toàn bộ (sợi chỉ đỏ nối trạm 1–4–7)

## CHAIN — Insight 1: Approximate ↔ Recall

ANN đổi **recall lấy tốc độ** — ý này xuyên từ trạm 2 (index) tới trạm 7 (threshold) => không index = exact = recall 100% nhưng `O(n)` chậm => có HNSW/IVFFlat = nhanh nhưng recall < 100% (bỏ sót vài hàng xóm thật) => threshold ở trạm 7 áp lên kết quả *xấp xỉ* → có thể sót thêm => người hiểu hệ thống nói được "tôi đang ở đâu trên đường cong recall–speed và đo nó thế nào" => đo bằng cách so ANN với exact định kỳ (recall@k)

## CHAIN — Insight 2: Một Postgres cho tất cả

siêu năng lực pgvector không phải tốc độ đỉnh mà là vector sống CẠNH business data (trạm 3+8) => vector cạnh business data → filter/join/hybrid trong MỘT câu SQL, MỘT transaction ACID => dedicated vector DB (Pinecone) buộc 2 hệ, 2 round-trip, đồng bộ phức tạp => kim chỉ nam: **"vector là feature hay là product?"** => feature → Postgres; product / tỷ-vector / đa-vùng → dedicated

## CHAIN — Insight 3: Re-embedding & lock-in

đổi embedding model = re-embed toàn bộ corpus vì vector hai model khác không gian (trạm 1+3+6) => đây là quyết định kiến trúc NẶNG hơn cả chọn index => hệ quả: chốt model sớm ở giai đoạn thiết kế => version hóa cột (`embedding_v2`), backfill blue-green => và ưu tiên **BYO-embedding + chuẩn mở** để tránh lock-in (in-DB embedding kiểu HeatWave tiện nhưng khóa chặt)

## CHAIN — Insight 4: Hybrid — keyword & nghĩa bù nhau

FTS (lexeme) chính xác với thuật ngữ/mã/tên riêng nhưng MÙ ngữ nghĩa ("ô tô" ≠ "xe hơi") (trạm 8+1) => vector hiểu nghĩa nhưng có thể trượt thuật ngữ hiếm/mã sản phẩm => hai bên bù nhau nên **hybrid** = chạy CẢ hai rồi fuse bằng **RRF** => hybrid là kiến trúc retrieval mặc định 2026, làm được ngay trong MỘT Postgres => đây là nơi hai nửa của khóa (keyword ở bài FTS, semantic ở phần còn lại) hội tụ

## CHAIN — Các trạm ràng buộc lẫn nhau

các trạm không độc lập — quyết định trạm này ràng buộc trạm khác => "nạp trước index sau" ở trạm 6 chỉ có nghĩa vì hiểu index maintenance đắt ở trạm 2 => nếu bảng đã có index HNSW, mỗi insert phải cập nhật graph → chậm => nên nạp toàn bộ vào bảng chưa index rồi build một lần sau => và "threshold" ở trạm 7 phụ thuộc metric chọn ở trạm 1/4 (cosine [0,2] dễ đặt vs L2 ∞)

## CHAIN — Tổ chức: một câu framing gói cả khóa

một câu framing cho stakeholder gói cả khóa: thêm tìm-kiếm-theo-nghĩa vào chính database đang chạy => không dựng hệ thống mới, đội hiện tại vận hành được, dữ liệu ở nguyên một chỗ => chi phí lớn nhất KHÔNG phải lưu trữ mà là (a) để AI sinh embedding và (b) nếu đổi model phải xử lý lại toàn bộ dữ liệu => nên chốt lựa chọn sớm và làm pipeline chạy-lại-được => tóm lại: rủi ro thấp, chi phí rõ, ít lock-in

## CHAIN — Cả khóa trong 5 câu

máy tính không hiểu chữ/ảnh => dùng model AI biến chúng thành embedding (giống nghĩa → vector gần nhau) => lưu embedding vào PostgreSQL nhờ pgvector, dùng index HNSW tìm "hàng xóm gần nhất" nhanh dù triệu vector => khi user hỏi, embed câu hỏi rồi tìm bản ghi có vector gần nhất = semantic search => ghép thêm full-text search (từ khóa) thành hybrid cho kết quả tốt nhất

---

## BẢNG — 8 trạm & quyết định cốt lõi

| # | Trạm | Quyết định | Chốt nhanh |
|---|---|---|---|
| 1 | **Embed** | model nào? | English→OpenAI 3-small; đa ngữ/VN→BGE-M3; nhẹ/local→MiniLM. **Đổi model = re-embed tất cả.** |
| 2 | **Index** | index nào? | Nhỏ (<50K)→không index (exact). Vừa→**HNSW**. RAM ít/tĩnh→IVFFlat/DiskANN. |
| 3 | **Platform** | native hay dedicated? | Feature + <50M→**pgvector**. Product/tỷ vector→Pinecone/Milvus. |
| 4 | **Store** | schema? | Cột `vector(n)` (n khớp model) **cạnh** business data → hybrid 1 câu. |
| 5 | **Cài** | cài kiểu gì? | **Docker/apt/managed**, không build source trên prod; pin version. Cài file ≠ `CREATE EXTENSION`. |
| 6 | **Nạp** | insert kiểu gì? | **COPY** nhanh nhất; **nạp trước, index sau**; idempotent temp+upsert. |
| 7 | **Query** | lấy ra thế nào? | `ORDER BY <=> LIMIT k` (số lượng) + **threshold** (chất lượng); loại chính nó. |
| 8 | **Hybrid** | keyword + semantic? | FTS (`tsvector`/`@@`) bắt từ khóa; vector bắt nghĩa; fuse **RRF**. |

## BẢNG — End-to-end nhỏ nhất (cả 8 mảnh)

```sql
CREATE EXTENSION IF NOT EXISTS vector;                                    -- [5] cài
CREATE TABLE docs (id bigserial PRIMARY KEY, content text, embedding vector(384));  -- [4] schema
INSERT INTO docs (content, embedding) VALUES ('...', '[...]');            -- [6] nạp (prod: COPY)
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);           -- [2] index (sau nạp)
SELECT content, 1 - (embedding <=> :q) AS similarity                      -- [7] query
FROM docs WHERE embedding <=> :q < 0.3                                    --     + threshold
ORDER BY embedding <=> :q LIMIT 5;
-- :q do [1] model sinh, CÙNG model đã embed docs
```

## BẢNG — System design tổng (20M docs, đa ngữ+VN, nhạy cảm, p95<150ms, hàng ngày, không downtime, team nhỏ)

1. **Clarify:** QPS, recall target, compliance, ngân sách, tần suất update.
2. **[1 Embed]** nhạy cảm + đa ngữ → **self-host BGE-M3** (dữ liệu ở nhà, VN tốt, BYO); chunk + batch GPU + cache hash.
3. **[3 Platform]** 20M < ngưỡng dedicated + feature + đã có Postgres → **pgvector self-host** (privacy, không API ngoài).
4. **[4 Store]** `docs(tenant_id, content, lang, embedding vector(1024), ...)`; **partition theo tenant/lang**; B-tree metadata.
5. **[5 Cài]** Docker tag pinned/gói OS; `CREATE EXTENSION` qua migration; đồng bộ version replica.
6. **[6 Nạp]** COPY (binary) vào bảng chưa index; **idempotent** temp+upsert; update hàng ngày qua queue.
7. **[2 Index]** HNSW `vector_cosine_ops`, build **CONCURRENTLY** sau nạp + `maintenance_work_mem` cao + parallel; quantization/DiskANN nếu RAM căng.
8. **[7 Query]** `WHERE <filter> AND <=> < :thr ORDER BY <=> LIMIT k`; tune `ef_search` theo p95; **iterative_scan**; threshold hiệu chỉnh trên golden set.
9. **[8 Hybrid]** FTS (`tsvector` GIN) + vector fuse **RRF**; VN cân nhắc `simple`+unaccent hoặc dựa nhiều vào vector.
10. **Vận hành:** monitor recall@k, p99, WAL/replica lag, drift distance; blue-green đổi model; re-embed versioned.
11. **Tự đánh giá:** bottleneck là *embedding generation* không phải insert; privacy *ép* self-host; 20M chọn pgvector nhưng biết *ngưỡng* (hàng trăm triệu + đa vùng) sẽ khiến cân nhắc dedicated = biết lý do & giới hạn.

## BẢNG — Capstone Project (đưa lên GitHub)

**"Semantic + Hybrid Search Engine trên PostgreSQL"** — corpus thật (Wikipedia/FAQ/sản phẩm); stack PostgreSQL + pgvector (Docker) + model self-host (sentence-transformers/BGE-M3) + FastAPI/Express.

8 mảnh phải có: (1) pipeline chunk+embed batch; (2) schema `vector(n)` + metadata + GIN cho FTS; (3) nạp COPY, nạp-trước-index-sau, idempotent; (4) HNSW + tune `ef_search`; (5) endpoint k-NN + threshold + filter; (6) hybrid endpoint fuse RRF; (7) `EXPLAIN ANALYZE` chứng minh dùng index + đo recall@k trên golden set; (8) README bàn *trade-off* (vì sao HNSW, vì sao pgvector, ngưỡng chọn thế nào). Điểm gây ấn tượng: README nói *trade-off & giới hạn* = tư duy staff.

## BẢNG — Master cheatsheet: 20 keyword

**Embedding** · **dimension** · **static vs contextual** (Word2Vec vs transformer) · **cosine similarity/distance** (`<=>` = distance = 1−sim) · **L2/inner product** (`<->`/`<#>`) · **k-NN / ANN** · **recall** · **HNSW** (graph, ~O(log n)) · **IVFFlat** (centroid/list/probe) · **DiskANN** · **curse of dimensionality** · **pgvector** (extension) · **ops class** (khớp toán tử) · **tsvector/tsquery** (FTS lexeme, khác vector) · **GIN** (inverted index) · **hybrid search / RRF** · **COPY** (bulk nhanh nhất) · **threshold** (metric/dataset-dependent) · **re-embedding** (đổi model = làm lại hết) · **BYO vs in-DB embedding**.

## BẢNG — 12 core concept

1. Model tạo vector; **pgvector chỉ lưu & tìm**, không sinh embedding.
2. **Static vs contextual** — transformer thắng nhờ attention hiểu ngữ cảnh.
3. Cosine đo **hướng** (nghĩa), bỏ độ dài; `<=>` là **distance**, sim = `1 - <=>`.
4. Không index = exact (recall 100%, O(n)); **ANN đổi recall lấy tốc độ**.
5. **HNSW** default (recall cao, tốn RAM); **IVFFlat** nhẹ (cần data, ngại data động).
6. B-tree/binary search **không dùng cho vector** (không sắp thứ tự 1 trục; curse of dimensionality) → cần ANN.
7. Siêu năng lực pgvector: **vector cạnh business data → hybrid query 1 câu SQL**.
8. **Vector là feature → Postgres; là product/tỷ-vector → dedicated.**
9. **COPY nhanh nhất**; **nạp trước, index sau**; idempotent temp+upsert.
10. Query: `ORDER BY <=> LIMIT k` (số lượng) + **threshold** (chất lượng); loại chính nó; threshold thuần bỏ index.
11. **Hybrid = FTS (keyword) + vector (nghĩa), fuse RRF** — default 2026.
12. **Đổi model = re-embed tất cả** — quyết định nặng nhất; **privacy có thể ép self-host**.

## BẢNG — Code cả vòng đời

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE docs (id bigserial PRIMARY KEY, content text, embedding vector(1536));
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
\copy docs (content, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)

-- Query: k-NN + threshold + similarity%
SELECT content, 1 - (embedding <=> :q) AS sim
FROM docs WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q LIMIT 5;

-- Hybrid (RRF)
WITH kw AS (SELECT id, row_number() OVER (ORDER BY ts_rank(sv,q) DESC) r
            FROM docs, websearch_to_tsquery('english',:txt) q WHERE sv @@ q LIMIT 50),
     sem AS (SELECT id, row_number() OVER (ORDER BY embedding <=> :qv) r
            FROM docs ORDER BY embedding <=> :qv LIMIT 50)
SELECT id, SUM(1.0/(60+r)) rrf FROM (SELECT * FROM kw UNION ALL SELECT * FROM sem) t
GROUP BY id ORDER BY rrf DESC LIMIT 10;
```
```python
from sentence_transformers import SentenceTransformer   # cùng model cho DB và query!
m = SentenceTransformer("all-MiniLM-L6-v2")
vec = m.encode(text, normalize_embeddings=True)
```

## BẢNG — 12 câu phỏng vấn "trải cả khóa"

1. **Embedding là gì?** → vector mã hóa nghĩa; ghi điểm bằng static vs contextual.
2. **Cosine vs L2?** → cosine đo hướng (text); normalize thì L2≡cosine về xếp hạng.
3. **[Bẫy] `<=>` là similarity?** → không, **distance**; sim = `1 - <=>`.
4. **HNSW vs IVFFlat?** → graph ~O(log n) recall cao tốn RAM vs centroid nhẹ cần data.
5. **[Bẫy] Sao không B-tree cho vector?** → không sắp thứ tự 1 trục; curse of dimensionality → ANN.
6. **pgvector vs Pinecone?** → feature vs product; Postgres thắng operational simplicity + hybrid SQL.
7. **[Bẫy] `tsvector` = vector search?** → không, đó là FTS (lexeme); semantic cần `vector`.
8. **Bulk load nhanh nhất?** → COPY vào bảng chưa index → build index sau; idempotent.
9. **[Bẫy] Threshold `WHERE <=> < x` nhanh?** → thuần thì seq scan; phải kèm `ORDER BY <=> LIMIT k`.
10. **Đổi model?** → re-embed toàn bộ; version hóa, blue-green.
11. **[Scale] Tỷ vector, RAM ít?** → quantization/DiskANN + two-stage re-rank + partition; cân dedicated nếu đa vùng.
12. **Hybrid search?** → FTS + vector fuse RRF; keyword chính xác + nghĩa gần.

---

## TỪ KHÓA MỒI

- Câu hỏi lớn & 8 trạm → **"một đường ống dữ liệu"**
- Ba câu chốt xuyên khóa → **"non-text, distance = 1−sim"**
- Sợi chỉ đỏ cùng model → **"trạm 1–4–7"**
- Insight 1 Approximate↔Recall → **"đâu trên đường cong recall–speed"**
- Insight 2 một Postgres → **"feature hay product"**
- Insight 3 re-embedding → **"nặng hơn chọn index"**
- Insight 4 hybrid → **"keyword & nghĩa bù nhau"**
- Các trạm ràng buộc → **"nạp trước index sau nối 6↔2"**
- Tổ chức framing → **"rủi ro thấp, chi phí rõ"**
- Cả khóa 5 câu → **"embed → lưu → tìm → hybrid"**

---

## ĐÃ PHỦ

**Bản đồ hệ thống:** câu hỏi lớn (PostgreSQL → semantic search/RAG production); đường ống 8 trạm (embed → index → platform → store → cài → nạp → query → hybrid → RAG); 3 câu chốt xuyên khóa (embedding non-text đa modality; cosine gần 1 = giống nhưng `<=>` = distance; pgvector non-text khác tsvector); cả khóa trong 5 câu; end-to-end nhỏ nhất (8 mảnh ~15 dòng).

**Bảng 8 trạm & quyết định cốt lõi + chốt nhanh** (mỗi trạm một quyết định).

**4 connective insights (xuyên nhiều bài):** (1) Approximate↔Recall — ANN đổi recall lấy tốc độ từ trạm 2→7, đo bằng so exact định kỳ; (2) một Postgres cho tất cả — vector cạnh business data → hybrid 1 câu ACID, feature vs product; (3) re-embedding & lock-in — đổi model = re-embed toàn bộ, nặng hơn chọn index, chốt sớm/version hóa/blue-green/BYO+chuẩn mở; (4) hybrid — FTS + vector bù nhau, fuse RRF, một Postgres.

**Sợi chỉ đỏ xuyên trạm:** "cùng model" ở trạm 1/4/7; "nạp trước index sau" (6) nối index maintenance đắt (2); "threshold" (7) phụ thuộc metric chọn ở 1/4.

**Staff:** system design tổng 11 bước (20M đa ngữ nhạy cảm không downtime, đi theo pipeline); capstone project spec (8 mảnh + README trade-off); tổ chức framing gói cả khóa (rủi ro thấp, chi phí rõ, ít lock-in).

**Master cheatsheet:** 20 keyword sống còn; 12 core concept; code cả vòng đời (cài+schema+index+COPY+query threshold+hybrid RRF+Python embed); 12 câu phỏng vấn trải khóa + chốt; one-liner tổng hợp.

**Nối bộ 9 file:** capstone này + 8 chủ đề = phủ trọn tạo vector → index → nền tảng → store → cài → nạp → query → hybrid → tổng hợp.

**Học tiếp (bài nêu):** RAG end-to-end (retrieval → rerank → LLM synthesis), reranking cross-encoder, chunking strategy, evaluation (recall@k/nDCG/MRR golden set), multimodal embeddings, sharding vector (Citus/pgvectorscale).
