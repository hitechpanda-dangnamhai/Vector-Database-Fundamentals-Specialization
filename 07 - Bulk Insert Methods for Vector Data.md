# Bulk Insert Dữ Liệu Vector vào pgvector: COPY, pg-promise, psycopg — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"Techniques for bulk data transfer, including pg-promise"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy: vì sao bulk insert quan trọng, lệnh **COPY**, bulk insert trong Node.js bằng **pg-promise**, và trong Python bằng **psycopg2** (`execute_values`). Tôi giảng bám sát rồi đào sâu thành pipeline ingestion thật. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là mảnh *"nạp dữ liệu vào NHANH"* — nối tiếp bài hands-on cài đặt (#6). Cài xong, tạo bảng xong, giờ là lúc *đổ hàng triệu vector vào* mà không làm sập performance.
>
> **⚠️ Đính chính & cập nhật tới 2026 (bài gốc có chỗ đã cũ/sắp xếp chưa đúng):**
> 1. **COPY là cách bulk insert NHANH NHẤT** — không chỉ "một lựa chọn cho CSV". Thứ tự tốc độ: `row-by-row INSERT` ≪ `batch/multi-row INSERT` (pg-promise, execute_values) ≪ **`COPY`**. Bài gốc trình bày COPY ngang hàng, nên nói rõ lại.
> 2. **`psycopg2` là thư viện cũ; bản kế nhiệm là `psycopg` (v3, "psycopg3").** Trong psycopg3 **không còn `execute_values`** — thay bằng `executemany()` (đã tối ưu pipeline) hoặc `cursor.copy()` (nhanh nhất). Bài gốc dạy psycopg2 vẫn chạy, nhưng nên biết bản mới.
> 3. **Bài gốc bỏ qua nguyên tắc vàng của bulk load:** *nạp toàn bộ dữ liệu TRƯỚC, tạo index (HNSW/IVFFlat) SAU.* Duy trì index trong lúc insert hàng loạt chậm hơn nhiều.
> 4. **Vector cần được đăng ký kiểu** (`register_vector`) hoặc format đúng `'[1,2,3]'` thì mới serialize được khi bulk insert/COPY.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Cách đưa một *lượng lớn* embedding vào PostgreSQL/pgvector một cách hiệu quả — thay vì insert từng dòng (chậm khủng khiếp). Ba kỹ thuật: **COPY** (nhanh nhất, từ file/stream), **batch insert** trong Node.js (**pg-promise**) và Python (**psycopg2/psycopg3**). Kèm nguyên tắc vận hành: nạp trước, index sau.

**Vấn đề nó giải quyết:** Mỗi câu `INSERT` là *một transaction* → có overhead cố định (WAL, commit, khóa). Insert 1 triệu vector một-dòng-một = 1 triệu transaction = cực chậm. Với vector nhiều chiều (mỗi dòng vài KB), càng tệ. Bulk insert gom nhiều dòng vào ít transaction → nhanh gấp hàng chục–trăm lần.

**Học xong bạn sẽ làm được:**

- Giải thích vì sao bulk insert nhanh hơn one-by-one (transaction overhead).
- Xếp đúng thứ tự tốc độ: row-by-row < batch < COPY, và biết chọn cái nào khi nào.
- Dùng COPY (CSV → table, và stream) để nạp vector.
- Bulk insert từ Node.js (pg-promise) và Python (psycopg2 execute_values / psycopg3 copy).
- Thiết kế pipeline ingestion: nạp → build index → verify, ở quy mô hàng triệu vector.

**Mạch basic → staff:**

- 🟢 **Basic:** Vì sao one-by-one chậm → ba phương pháp & thứ tự tốc độ → COPY cơ bản (CSV → table).
- 🟡 **Intermediate:** COPY sâu hơn; pg-promise (ColumnSet, helpers.insert, db.none); psycopg2 execute_values; lỗi thường gặp.
- 🔴 **Advanced:** Vì sao COPY nhanh nhất (transaction/WAL/binary); psycopg3 `copy()` hiện đại; **nạp-trước-index-sau**; batch size; `maintenance_work_mem`; edge cases (vector trong CSV, register_vector).
- 🟣 **Staff:** Ingestion pipeline quy mô triệu vector; drop/rebuild index; parallel; WAL/replication; upsert idempotent; monitoring; câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Vì sao insert từng dòng lại chậm

- Mỗi `INSERT` một dòng = **một transaction độc lập**. Transaction có chi phí cố định: ghi WAL (write-ahead log), commit, fsync, network round-trip.
- Insert 1 triệu dòng riêng lẻ = 1 triệu lần trả cái phí đó → tổng overhead khổng lồ.
- **Với vector càng nặng:** embedding nhiều chiều (512–1536+) mỗi dòng vài KB; one-by-one nghĩa là 1 triệu round-trip mang từng khối vài KB → cực kém hiệu quả.

Bài gốc nói đúng ý cốt: *bulk insert giúp duy trì performance, nạp nhanh hơn* — bằng cách **gom nhiều dòng vào ít transaction/round-trip**.

### 1.2. Analogy: chuyển nhà bằng xe tải vs bằng tay

- **Row-by-row insert** = bê từng cái ghế một, mỗi lần đi bộ ra xe rồi quay lại. 1000 chuyến cho 1000 món.
- **Batch insert** = chất 50 món lên xe đẩy rồi đẩy một lần. 20 chuyến.
- **COPY** = thuê xe tải, chất hết lên, chạy một chuyến. Nhanh nhất.

Cùng chuyển ngần ấy đồ, số "chuyến" (transaction/round-trip) mới quyết định tốc độ.

### 1.3. Ba phương pháp & thứ tự tốc độ (nhớ kỹ thứ tự này)

| Phương pháp | Cơ chế | Tốc độ tương đối |
|---|---|---|
| Row-by-row `INSERT` | 1 dòng / 1 câu / 1 transaction | 🐢 chậm nhất |
| **Batch / multi-row `INSERT`** | nhiều dòng / 1 câu (pg-promise, execute_values) | 🚗 nhanh hơn nhiều |
| **`COPY`** | giao thức nạp khối chuyên dụng | 🚀 **nhanh nhất** |

> **[Đính chính bài gốc]** Bài gốc trình bày COPY và pg-promise/execute_values gần như ngang hàng. Thực tế **COPY nhanh nhất** — dùng COPY khi có thể; batch insert (pg-promise/execute_values) là lựa chọn khi dữ liệu đến từ code/app chứ không phải file, hoặc cần logic phức tạp.

### 1.4. COPY cơ bản — nạp từ CSV

`COPY` chuyển dữ liệu giữa **file và bảng** (cả hai chiều). Bài gốc dùng ví dụ `FAQdata.csv` chứa text + embedding, nạp vào bảng `faqs(id, text, embedding)`.

```sql
-- Bảng đích (embedding là cột vector)
CREATE TABLE faqs (
    id        serial PRIMARY KEY,
    text      text,
    embedding vector(512)
);

-- COPY phía server (file phải nằm trên MÁY CHỦ Postgres, cần quyền cao)
COPY faqs (text, embedding)
FROM '/path/FAQdata.csv'
WITH (FORMAT csv, HEADER true);
```

Trong `psql`, dùng `\copy` (phía client — file nằm trên máy bạn, không cần quyền server):

```sql
\copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)
```

**Gotcha vector-trong-CSV (quan trọng):** embedding viết dạng `[0.1,0.2,...]` chứa dấu phẩy → trong CSV phải **bọc nháy kép**:
```csv
text,embedding
"thủ đô Anh là London","[0.12,0.98,...]"
"đường sắt Đức là Deutsche Bahn","[0.44,0.03,...]"
```
Quên bọc nháy → CSV parser tách nhầm mỗi số thành một cột → lỗi.

### 1.5. Ngược lại: xuất bảng ra CSV

```sql
\copy (SELECT text, embedding FROM faqs) TO 'export.csv' WITH (FORMAT csv, HEADER true)
```

### ✅ Self-check Phần 1

1. Vì sao insert 1 triệu dòng riêng lẻ chậm hơn nhiều so với gom thành batch?
2. Xếp thứ tự tốc độ: row-by-row, batch insert, COPY.
3. Trong CSV, vì sao cột embedding `[1,2,3]` phải bọc nháy kép?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. Bulk insert trong Node.js với pg-promise (bám bài gốc)

Khi dữ liệu **không** ở dạng CSV mà đến từ code (vd bạn vừa sinh embedding trong app), dùng **pg-promise** để batch insert:

```javascript
// npm install pg-promise
const pgp = require('pg-promise')();
const db  = pgp('postgresql://user:pass@localhost/db');   // khởi tạo với config DB

// 1) Định nghĩa bảng & cột sẽ insert (ColumnSet — tái dùng được, hiệu năng tốt)
const cs = new pgp.helpers.ColumnSet(['text', 'embedding'], { table: 'faqs' });

// 2) Chuẩn bị dữ liệu: mảng các object. Embedding format thành literal vector '[...]'
const rows = [
  { text: 'thủ đô Anh là London',        embedding: '[' + emb1.join(',') + ']' },
  { text: 'đường sắt Đức Deutsche Bahn',  embedding: '[' + emb2.join(',') + ']' },
  // ... hàng nghìn dòng
];

// 3) pgp.helpers.insert dựng MỘT câu INSERT nhiều dòng từ ColumnSet
const query = pgp.helpers.insert(rows, cs);   // INSERT INTO faqs(text,embedding) VALUES (...),(...),...

// 4) db.none: chạy query không mong trả về dữ liệu (insert không trả row)
await db.none(query);
```

Bốn bước đúng như bài gốc: cài → import & init → định nghĩa `ColumnSet` → gom values → `pgp.helpers.insert` → `db.none`. Điểm mấu chốt: `helpers.insert` gộp *nhiều* dòng vào *một* câu INSERT (batch), thay vì nghìn câu riêng.

> **[MỞ RỘNG]** Với lô rất lớn, chia thành các chunk (vd 1.000–10.000 dòng/lần) rồi lặp — một câu INSERT quá to (hàng trăm nghìn dòng) có thể vượt giới hạn tham số/bộ nhớ. Muốn nhanh nhất trong Node, dùng **COPY** qua `pg-copy-streams`.

### 2.2. Bulk insert trong Python với psycopg2 (bám bài gốc)

```python
# pip install psycopg2-binary pgvector
import psycopg2
from psycopg2.extras import execute_values
from pgvector.psycopg2 import register_vector

conn = psycopg2.connect("dbname=db user=... password=... host=localhost")
register_vector(conn)                      # để embedding (list/ndarray) serialize thành vector

# data: list các tuple (sentence, embedding) — mỗi tuple độ dài 2
data = [
    ("thủ đô Anh là London",        emb1),   # emb1 là list/ndarray float
    ("đường sắt Đức Deutsche Bahn",  emb2),
    # ... nhiều dòng
]

with conn.cursor() as cur:
    execute_values(                        # gộp nhiều dòng vào 1 câu INSERT
        cur,
        "INSERT INTO faqs (text, embedding) VALUES %s",   # %s là chỗ execute_values chèn tất cả
        data,
    )
conn.commit()                              # commit MỘT lần cho cả lô
cur.close(); conn.close()
```

Đúng luồng bài gốc: cài psycopg2 → import → `execute_values` từ `extras` → truyền query template + list tuple → connect → cursor → execute → **commit** → close.

> **[Đính chính — quan trọng]** `psycopg2` là thư viện *cũ*. Bản kế nhiệm **`psycopg` (v3)** không còn `execute_values`; thay bằng `executemany()` (đã tối ưu pipeline) hoặc `cursor.copy()` (nhanh nhất). Code psycopg2 vẫn chạy tốt, nhưng dự án mới nên cân nhắc psycopg3 (xem 3.2).

### 2.3. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — Quên `commit()` (Python).** psycopg2 mặc định không autocommit → không `conn.commit()` thì dữ liệu *không* được lưu (rollback khi đóng). Bài gốc nhấn đúng bước commit — đừng bỏ.

**Lỗi 2 — Quên đăng ký kiểu / format vector sai.** Không `register_vector` (Python) hoặc không format `'[...]'` (Node) → driver không biết chuyển list → vector → lỗi hoặc lưu sai. Luôn đăng ký kiểu hoặc format literal.

**Lỗi 3 — Một câu INSERT/COPY quá khổng lồ.** Nhét cả triệu dòng vào một lệnh → tràn bộ nhớ / vượt giới hạn. Chia **chunk** (batch size 1k–10k) và (tùy) bọc mỗi chunk trong transaction riêng để cân bằng tốc độ vs bộ nhớ.

### ✅ Self-check Phần 2

1. Trong pg-promise, `ColumnSet` và `pgp.helpers.insert` mỗi cái làm gì?
2. Nếu quên `conn.commit()` trong psycopg2, chuyện gì xảy ra với dữ liệu?
3. `execute_values` còn trong psycopg3 không? Thay bằng gì?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Vì sao COPY nhanh nhất — dưới nắp capo

- **Ít transaction/parse hơn:** COPY nạp cả khối trong một luồng, không parse lại từng câu INSERT, không lặp overhead commit.
- **Giao thức binary:** COPY (đặc biệt `FORMAT BINARY`) gửi dữ liệu nhị phân thô thay vì biểu diễn text → tiết kiệm băng thông + khỏi parse text phía server.
- **Tối ưu buffer/cache:** COPY dùng ring buffer riêng, giảm áp lực lên shared buffers.
- **Vẫn an toàn:** COPY vẫn ghi WAL (pgvector dùng WAL) → replication & point-in-time recovery không mất.

Kết quả benchmark phổ biến: COPY nhanh hơn batch INSERT nhiều lần, và bỏ xa row-by-row.

### 3.2. psycopg3 `copy()` — cách nhanh nhất hiện đại (thay execute_values)

```python
# pip install "psycopg[binary]" pgvector
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("dbname=db user=... host=localhost")
register_vector(conn)

rows = [("thủ đô Anh là London", emb1), ("đường sắt Đức", emb2)]  # generator được luôn (lazy)

with conn.cursor() as cur:
    with cur.copy("COPY faqs (text, embedding) FROM STDIN WITH (FORMAT BINARY)") as copy:
        copy.set_types(["text", "vector"])       # khai báo kiểu để serialize đúng
        for text, emb in rows:
            copy.write_row([text, emb])           # từng dòng được stream thẳng vào server
conn.commit()
```

Ưu điểm psycopg3 `copy()`: nhanh nhất, **stream lazy** (không cần nạp cả triệu dòng vào RAM một lúc), format binary. Đây là cách khuyến nghị cho ingestion lớn 2026.

Nếu không muốn COPY, psycopg3 có `executemany()` đã tối ưu pipeline:
```python
cur.executemany("INSERT INTO faqs (text, embedding) VALUES (%s, %s)", rows)
```

### 3.3. [NGUYÊN TẮC VÀNG] Nạp dữ liệu TRƯỚC, tạo index SAU

Đây là điều bài gốc bỏ qua nhưng quyết định tốc độ ingestion:

- Nếu bảng **đã có** index HNSW/IVFFlat, mỗi lần insert phải *cập nhật index* → chậm hẳn (đặc biệt HNSW: `ef_construction` cao càng chậm insert).
- **Cách đúng cho bulk load:**
  ```sql
  -- 1) Bảng KHÔNG index
  CREATE TABLE faqs (id serial PRIMARY KEY, text text, embedding vector(512));
  -- 2) COPY / bulk insert toàn bộ dữ liệu (nhanh vì không maintain index)
  \copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)
  -- 3) SAU KHI nạp xong mới build index (một lần, nhanh hơn nhiều)
  SET maintenance_work_mem = '2GB';                    -- tăng RAM cho build
  CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
  ```
- Với **IVFFlat** thì bắt buộc kiểu này: nó cần dữ liệu sẵn để k-means (xem giáo trình indexing). Với **HNSW** cũng nên vậy để build nhanh.

### 3.4. Batch size & transaction — tinh chỉnh

- **Batch size:** 1.000–10.000 dòng/lô là điểm ngọt phổ biến. Quá nhỏ → nhiều round-trip; quá lớn → tốn RAM, một lỗi làm hỏng cả lô lớn.
- **Transaction:** bọc mỗi lô trong một transaction (`BEGIN...COMMIT`) → ít commit hơn row-by-row nhưng vẫn giới hạn thiệt hại nếu lỗi.
- **`maintenance_work_mem`** cho bước build index sau nạp: đặt cao (nhưng ≤ 50–60% RAM); nếu build HNSW song song, đảm bảo `--shm-size` (Docker) đủ lớn.

### 3.5. Edge cases

- **Vector trong CSV:** bọc nháy kép (mục 1.4).
- **Dimension mismatch:** một dòng có vector sai chiều → COPY/insert cả lô fail (COPY thường all-or-nothing trong một lệnh). Validate chiều trước khi nạp.
- **NULL embedding:** cho phép nếu cột nullable, nhưng dòng đó không xuất hiện trong ANN search → backfill sau.
- **Upsert khi trùng:** COPY không hỗ trợ `ON CONFLICT` trực tiếp → mẹo: COPY vào **bảng tạm** rồi `INSERT ... SELECT ... ON CONFLICT DO UPDATE` (xem 4.3).
- **`register_vector` bắt buộc** cho cả execute_values/copy để serialize vector.

### ✅ Self-check Phần 3

1. Nêu 2 lý do COPY nhanh hơn batch INSERT.
2. Vì sao nên nạp dữ liệu trước rồi mới tạo HNSW index? Với IVFFlat thì sao?
3. psycopg3 `copy()` có ưu điểm gì về bộ nhớ so với gom cả list rồi execute_values?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Ingestion pipeline ở quy mô triệu vector

Bulk insert không đứng một mình — nó là một mắt trong pipeline. Toàn cảnh khi nạp hàng triệu tài liệu:

```
1. Chunk tài liệu dài (tránh truncation của model)      [giáo trình embeddings]
2. Sinh embedding theo BATCH (32-256/lần) trên GPU/API   [batch = tối ưu #1]
3. COPY (binary) vào bảng CHƯA index                     [nạp nhanh nhất]
4. Build index HNSW SAU khi nạp xong (maintenance_work_mem cao)
5. VERIFY (EXPLAIN ANALYZE thấy Index Scan; recall@k)
```

**Bottleneck thường không phải insert mà là bước sinh embedding** (mục 2 — API/GPU). Nhưng nếu insert row-by-row, chính insert thành nút thắt → COPY gỡ nút đó.

### 4.2. Chiến lược drop & rebuild index cho re-load lớn

Khi phải nạp lại/nạp thêm rất nhiều vào bảng *đã có* index:

- **Nạp thêm lớn:** cân nhắc `DROP INDEX` → bulk load → `CREATE INDEX CONCURRENTLY` lại. Rẻ hơn maintain index suốt quá trình insert khối lượng lớn.
- **Không thể mất search khi re-load:** dùng `CREATE INDEX CONCURRENTLY` (không khóa) và/hoặc bảng shadow + swap.
- **Parallel build:** tăng `max_parallel_maintenance_workers` để build index nhanh trên nhiều core.

### 4.3. Idempotency & upsert — nạp lại an toàn

Pipeline chạy lại (retry, backfill) không được tạo dòng trùng. Vì COPY không có `ON CONFLICT`:

```sql
-- 1) COPY vào bảng tạm không ràng buộc
CREATE TEMP TABLE faqs_stage (LIKE faqs INCLUDING DEFAULTS);
\copy faqs_stage (text, embedding) FROM 'batch.csv' WITH (FORMAT csv)

-- 2) Merge vào bảng thật, xử lý trùng
INSERT INTO faqs (text, embedding)
SELECT text, embedding FROM faqs_stage
ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;
```

Đây là mẫu "COPY → temp → upsert" chuẩn cho ingestion production idempotent.

### 4.4. WAL, replication, cost, monitoring

- **WAL:** COPY vẫn ghi WAL → nạp khối lượng lớn tạo *nhiều* WAL → có thể gây áp lực replication lag và dung lượng đĩa. Theo dõi WAL generation & replica lag khi bulk load lớn.
- **`UNLOGGED` table (thận trọng):** bảng `UNLOGGED` bỏ WAL → nạp nhanh hơn nhưng *mất dữ liệu khi crash* và không replicate → chỉ dùng cho staging/tái tạo được.
- **Cost:** chi phí ingestion chủ yếu ở *sinh embedding* (API token / GPU), không phải insert. Nhưng insert kém hiệu quả kéo dài thời gian nạp → tốn compute. Cache embedding theo hash để khỏi re-embed khi nạp lại.
- **Monitoring:** rows/giây khi nạp; thời gian build index; WAL/replica lag; tỉ lệ dòng lỗi (dimension mismatch/NULL); RAM khi build.

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Nạp dữ liệu ban đầu (vd 5 triệu tài liệu) là một tác vụ *chạy một lần* nhưng tốn thời gian — phần lâu nhất là để AI sinh embedding, không phải lưu vào DB. Dùng đúng kỹ thuật (COPY, nạp trước index sau) rút thời gian từ nhiều ngày xuống vài giờ. Ta lên lịch nạp ngoài giờ cao điểm để không ảnh hưởng khách." → framing bằng **thời gian, chi phí compute, và lịch vận hành**.
- **Roadmap:** ingestion pipeline (chunk → embed → COPY → index → verify) là hạ tầng dùng lại cho mọi lần thêm dữ liệu → xây thành job/service có retry & idempotency, đừng viết script một lần rồi bỏ.
- **Reliability:** idempotency (upsert) nghĩa là job nạp *chạy lại được* sau lỗi mà không hỏng dữ liệu — điều tối quan trọng khi nạp nhiều giờ và có thể đứt giữa chừng.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Bạn cần nạp 10 triệu tài liệu (mỗi cái ~500 token) vào pgvector cho semantic search, nhanh nhất có thể, không làm gián đoạn service hiện có, và job phải chạy lại được nếu đứt."**

**Khung trả lời staff:**

1. **Clarify:** nguồn dữ liệu (file/DB/stream)? sinh embedding local hay API? có được downtime không? ngân sách?
2. **Sinh embedding theo batch** (bottleneck thật): worker song song, batch 128–256, cache theo hash để nạp lại khỏi trả tiền lại.
3. **Nạp bằng COPY (binary)** vào bảng **chưa index** (hoặc bảng staging) — nhanh nhất; chia chunk để chịu lỗi.
4. **Idempotent:** COPY → temp table → `INSERT ... ON CONFLICT DO UPDATE` để retry an toàn.
5. **Build index SAU:** `CREATE INDEX CONCURRENTLY ... hnsw ...` với `maintenance_work_mem` cao + parallel workers → không khóa service.
6. **Bảo vệ service đang chạy:** nạp vào bảng shadow rồi swap, hoặc index CONCURRENTLY; theo dõi WAL/replica lag, giãn tốc nếu lag tăng.
7. **Verify:** `EXPLAIN ANALYZE` (Index Scan), đo recall@k, đếm rows.
8. **Tự đánh giá:** "Điểm nghẽn là *embedding generation* chứ không phải insert; COPY đảm bảo DB không thành nút thắt; idempotency cho phép chạy lại." → **biết đâu là bottleneck thật = tư duy staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **Bulk / batch insert** — nạp nhiều dòng trong ít transaction; nhanh hơn one-by-one.
- **Transaction overhead** — chi phí cố định (WAL, commit, round-trip) mỗi transaction; lý do one-by-one chậm.
- **`COPY` / `\copy`** — lệnh nạp khối nhanh nhất; server-side vs client-side.
- **`FORMAT BINARY`** — COPY nhị phân, nhanh hơn text.
- **pg-promise** — thư viện Node.js; `ColumnSet` + `helpers.insert` + `db.none` cho batch.
- **psycopg2 / `execute_values`** — thư viện Python cũ + hàm batch insert (từ `extras`).
- **psycopg (v3) / `cursor.copy()` / `executemany()`** — thư viện Python mới; `copy()` nhanh nhất, không còn `execute_values`.
- **`register_vector`** — đăng ký kiểu để list/ndarray serialize thành `vector`.
- **`commit()`** — bắt buộc để lưu (psycopg2 không autocommit).
- **Load-then-index** — nạp dữ liệu trước, tạo index sau (nhanh hơn nhiều).
- **`maintenance_work_mem`** — RAM cho build index sau nạp.
- **`ON CONFLICT DO UPDATE`** — upsert cho idempotency (qua temp table với COPY).
- **`CREATE INDEX CONCURRENTLY`** — build index không khóa write.
- **`UNLOGGED` table** — bỏ WAL, nạp nhanh nhưng không bền/replicate (chỉ staging).

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Mỗi INSERT là một transaction; one-by-one = trả overhead triệu lần → chậm.
2. Thứ tự tốc độ: **row-by-row ≪ batch insert ≪ COPY**. Dùng COPY khi có thể.
3. COPY nhanh nhất nhờ ít transaction + binary + buffer riêng; vẫn ghi WAL (an toàn).
4. **Nạp dữ liệu TRƯỚC, build index SAU** — nguyên tắc vàng của bulk load.
5. Node.js batch → **pg-promise** (`ColumnSet` + `helpers.insert`); Python → psycopg2 `execute_values` (cũ) hoặc psycopg3 `copy()` (mới, nhanh nhất).
6. **psycopg3 không còn `execute_values`**; dùng `executemany()`/`copy()`.
7. Luôn **`register_vector`** (hoặc format `'[...]'`) và **`commit()`** (psycopg2).
8. Vector trong CSV phải **bọc nháy kép** (vì chứa dấu phẩy).
9. Idempotency: COPY → temp table → `ON CONFLICT DO UPDATE` để job chạy lại an toàn.
10. Bottleneck ingestion thật thường là **sinh embedding**, không phải insert — nhưng insert kém làm nút thắt thứ hai.

### 5.3. Ideas / mental models

- **"Xe tải vs bê tay":** COPY vs row-by-row; số chuyến (transaction) quyết định.
- **"Nạp trước, index sau":** đừng bắt DB vừa nạp vừa vá index.
- **"COPY → temp → upsert":** mẫu idempotent cho ingestion.
- **"Bottleneck là embedding, không phải insert":** nhìn đúng nút thắt.
- **"Commit hay mất trắng":** psycopg2 không autocommit.

### 5.4. Code cần thuộc lòng

**(a) COPY (nhanh nhất):**
```sql
\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)
```

**(b) Node.js pg-promise (batch):**
```javascript
const cs = new pgp.helpers.ColumnSet(['text','embedding'], {table:'faqs'});
await db.none(pgp.helpers.insert(rows, cs));   // 1 câu INSERT nhiều dòng
```

**(c) Python — cũ vs mới:**
```python
# psycopg2 (bài gốc):
from psycopg2.extras import execute_values
execute_values(cur, "INSERT INTO faqs (text,embedding) VALUES %s", data); conn.commit()

# psycopg3 (nhanh nhất, khuyến nghị):
with cur.copy("COPY faqs (text,embedding) FROM STDIN") as cp:
    cp.set_types(["text","vector"])
    for t,e in rows: cp.write_row([t,e])
conn.commit()
```

**(d) Nạp-trước-index-sau:**
```sql
-- COPY hết → rồi mới:
SET maintenance_work_mem='2GB';
CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Vì sao bulk insert nhanh hơn insert từng dòng?"** → Mỗi INSERT là một transaction có overhead cố định (WAL, commit, round-trip); gom nhiều dòng vào ít transaction giảm overhead đó gấp nhiều lần.

2. **[XẾP HẠNG] "COPY, batch INSERT, row-by-row — cái nào nhanh nhất?"** → COPY nhanh nhất (ít transaction, binary, buffer riêng) > batch/multi-row INSERT > row-by-row. Dùng COPY khi có thể.

3. **[BẪY] "psycopg3 dùng `execute_values` thế nào?"** → Bẫy: psycopg3 **không có** `execute_values`. Dùng `executemany()` (pipeline) hoặc `cursor.copy()` (nhanh nhất). `execute_values` là của psycopg2.

4. **[STAFF] "Nạp 10 triệu vector nhanh nhất?"** → COPY (binary) vào bảng **chưa index**, rồi `CREATE INDEX CONCURRENTLY` sau với `maintenance_work_mem` cao + parallel; batch embedding trước; idempotent qua temp+upsert.

5. **[BẪY] "Tại sao build index trước rồi mới bulk insert lại chậm?"** → Vì mỗi insert phải cập nhật index (đặc biệt HNSW). Nạp trước, build index một lần sau nhanh hơn nhiều.

6. **[BẪY] "Insert xong mà truy vấn không thấy dữ liệu (Python)?"** → Quên `conn.commit()`; psycopg2 không autocommit → rollback khi đóng.

7. **"Bulk insert có làm mất replication/recovery không?"** → Không nếu dùng bảng thường (COPY vẫn ghi WAL). `UNLOGGED` thì bỏ WAL (nhanh hơn) nhưng mất khi crash và không replicate — chỉ dùng staging.

8. **"Làm sao để job nạp chạy lại an toàn?"** → Idempotency: COPY vào temp table → `INSERT ... SELECT ... ON CONFLICT DO UPDATE`. Retry không tạo trùng.

### 5.6. One-liner đắt giá

- *"Each INSERT is a transaction; bulk loading is about paying that overhead once instead of a million times."*
- *"COPY is the fastest way to get rows into Postgres — fewer transactions, binary protocol, its own buffer."*
- *"Load first, index second — never make the database maintain an HNSW graph while you're pouring millions of rows in."*
- *"psycopg3 dropped execute_values because optimized executemany and copy() already win — reach for copy() at scale."*
- *"COPY into a temp table, then upsert — that's how you make a multi-hour ingest job safe to re-run."*
- *"The real bottleneck in vector ingestion is usually embedding generation, not the insert — but a naive insert makes it the second one."*

---

### 📌 Ghi chú cuối

- **Đính chính để nhớ đúng:** **COPY nhanh nhất** (không ngang hàng batch); **psycopg3** thay psycopg2, **không có `execute_values`** (dùng `copy()`/`executemany()`); **nạp trước, index sau**; nhớ `register_vector` + `commit()`.
- **Kiểm chứng khi ôn:** xem docs `psycopg.org` (v3, `copy()`) và `pgvector` README cho ví dụ bulk load hiện hành; pg-promise docs cho `ColumnSet`/`helpers.insert`.
- **Thực hành:** tạo bảng `faqs(text, embedding vector(384))` *không index*, sinh ~100k embedding thật (giáo trình embeddings), nạp bằng cả ba cách (row-by-row, execute_values/pg-promise, COPY) và **đo thời gian** để tự thấy khác biệt; rồi mới `CREATE INDEX ... hnsw` và `EXPLAIN ANALYZE`.
- **Nối mạch series (đủ 7 mảnh):** embed → index → cài pgvector → store/query (#3) → **bulk insert (bài này)** → FTS → chọn nền tảng. Bạn giờ nạp được dữ liệu quy mô lớn vào semantic search/RAG một cách hiệu quả.
- **Học tiếp:** streaming ingestion (Kafka → embed → COPY), incremental re-embedding khi đổi model, và tối ưu build HNSW song song ở quy mô hàng chục triệu vector.
