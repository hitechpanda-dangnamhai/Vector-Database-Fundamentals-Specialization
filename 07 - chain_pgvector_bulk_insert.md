# Bulk Insert Vector vào pgvector: COPY, pg-promise, psycopg — Chain gối đầu

> Vị trí series (mảnh 7 "nạp dữ liệu vào NHANH", nối tiếp bài hands-on #6): cài xong, tạo bảng xong, giờ đổ hàng triệu vector vào mà không làm sập performance.
>
> **Đính chính bài gốc (đã cũ):** (1) **COPY là NHANH NHẤT** — không ngang hàng batch. Thứ tự: `row-by-row` ≪ `batch/multi-row INSERT` ≪ **`COPY`**. (2) `psycopg2` cũ; bản kế nhiệm **`psycopg` (v3)** — **không còn `execute_values`**, thay bằng `executemany()` hoặc `cursor.copy()` (nhanh nhất). (3) Nguyên tắc vàng bài gốc bỏ qua: **nạp toàn bộ dữ liệu TRƯỚC, tạo index (HNSW/IVFFlat) SAU**. (4) Vector cần **đăng ký kiểu** (`register_vector`) hoặc format đúng `'[1,2,3]'` mới serialize được.

---

## CHAIN — Vì sao insert từng dòng chậm

mỗi câu `INSERT` một dòng = một **transaction độc lập** => transaction có chi phí cố định: ghi WAL (write-ahead log), commit, fsync, network round-trip => insert 1 triệu dòng riêng lẻ = 1 triệu lần trả cái phí đó → tổng overhead khổng lồ => với vector càng nặng: embedding 512–1536 chiều, mỗi dòng vài KB → 1 triệu round-trip mang từng khối vài KB => cứu bằng **bulk insert**: gom nhiều dòng vào ÍT transaction/round-trip

## CHAIN — Analogy & thứ tự tốc độ

gom nhiều dòng vào ít "chuyến" giống chuyển nhà: số **chuyến** (transaction/round-trip) mới quyết định tốc độ => **row-by-row** insert = bê từng cái ghế, 1000 chuyến cho 1000 món (🐢 chậm nhất) => **batch/multi-row** insert = chất 50 món lên xe đẩy, 20 chuyến (🚗 nhanh hơn nhiều) => **COPY** = thuê xe tải chất hết chạy một chuyến (🚀 nhanh nhất) => đính chính bài gốc: COPY KHÔNG ngang hàng batch — COPY nhanh nhất, dùng khi có thể; batch (pg-promise/execute_values) là lựa chọn khi dữ liệu đến từ *code* chứ không phải file

## CHAIN — COPY từ CSV

COPY chuyển dữ liệu giữa **file và bảng** (cả hai chiều) => phía server: `COPY faqs FROM '/path/file.csv' WITH (FORMAT csv, HEADER true)` — file phải nằm trên MÁY CHỦ Postgres, cần quyền cao => trong `psql` dùng `\copy` (phía client) — file nằm trên máy bạn, không cần quyền server => gotcha vector-trong-CSV: embedding `[0.1,0.2,...]` chứa dấu phẩy → trong CSV phải **BỌC NHÁY KÉP** => quên bọc → CSV parser tách nhầm mỗi số thành một cột → lỗi

## CHAIN — pg-promise (Node batch)

khi dữ liệu đến từ code (vừa sinh embedding trong app) chứ không phải CSV, dùng **pg-promise** để batch insert => bước 1: định nghĩa `ColumnSet(['text','embedding'], {table:'faqs'})` — tái dùng được, hiệu năng tốt => bước 2: chuẩn bị mảng object, format embedding thành literal `'[' + emb.join(',') + ']'` => bước 3: `pgp.helpers.insert(rows, cs)` dựng MỘT câu INSERT nhiều dòng (thay vì nghìn câu riêng) => bước 4: `db.none(query)` chạy query không mong trả về row => lô rất lớn thì chia chunk 1k–10k/lần (một câu INSERT quá to vượt giới hạn tham số/bộ nhớ); muốn nhanh nhất trong Node dùng COPY qua `pg-copy-streams`

## CHAIN — psycopg2 & commit

Python psycopg2: `register_vector(conn)` để embedding (list/ndarray) serialize thành vector => `execute_values(cur, "INSERT ... VALUES %s", data)` gộp nhiều dòng vào 1 câu (từ `psycopg2.extras`) => `conn.commit()` MỘT lần cho cả lô — BẮT BUỘC => psycopg2 mặc định KHÔNG autocommit → quên commit thì dữ liệu không được lưu (rollback khi đóng) => đính chính: psycopg2 là thư viện *cũ*, bản kế nhiệm psycopg (v3) không còn `execute_values`

## CHAIN — psycopg3 copy() (nhanh nhất hiện đại)

psycopg3 thay `execute_values` bằng `executemany()` (đã tối ưu pipeline) hoặc `cursor.copy()` (nhanh nhất) => `with cur.copy("COPY faqs FROM STDIN WITH (FORMAT BINARY)") as copy:` => `copy.set_types(["text","vector"])` khai báo kiểu để serialize đúng => `copy.write_row([text, emb])` từng dòng được stream thẳng vào server => ưu điểm: **stream LAZY** — không cần nạp cả triệu dòng vào RAM một lúc, format binary => đây là cách khuyến nghị cho ingestion lớn 2026

## CHAIN — Vì sao COPY nhanh nhất

COPY nhanh nhất vì **ít transaction/parse**: nạp cả khối trong một luồng, không parse lại từng câu INSERT, không lặp overhead commit => **giao thức binary** (`FORMAT BINARY`) gửi dữ liệu nhị phân thô thay text → tiết kiệm băng thông + khỏi parse text phía server => **tối ưu buffer**: COPY dùng ring buffer riêng, giảm áp lực lên shared buffers => nhưng vẫn AN TOÀN: COPY vẫn ghi WAL → replication & point-in-time recovery không mất => kết quả: COPY nhanh hơn batch INSERT nhiều lần, bỏ xa row-by-row

## CHAIN — Nguyên tắc vàng: nạp trước, index sau

nếu bảng ĐÃ có index HNSW/IVFFlat, mỗi lần insert phải *cập nhật index* → chậm hẳn (HNSW `ef_construction` cao càng chậm insert) => nên nguyên tắc vàng của bulk load: nạp toàn bộ dữ liệu TRƯỚC, tạo index SAU => cách đúng: tạo bảng KHÔNG index → COPY/bulk insert toàn bộ (nhanh vì không maintain index) → `SET maintenance_work_mem` cao → `CREATE INDEX` một lần => với **IVFFlat** thì BẮT BUỘC kiểu này (cần dữ liệu sẵn để k-means); với **HNSW** cũng nên để build nhanh => đây là điều bài gốc bỏ qua nhưng quyết định tốc độ ingestion

## CHAIN — Batch size & transaction

batch size **1.000–10.000 dòng/lô** là điểm ngọt phổ biến => quá nhỏ → nhiều round-trip; quá lớn → tốn RAM, một lỗi làm hỏng cả lô lớn => bọc mỗi lô trong một transaction (`BEGIN...COMMIT`) → ít commit hơn row-by-row nhưng vẫn giới hạn thiệt hại nếu lỗi => còn `maintenance_work_mem` cho bước build index sau nạp: đặt cao nhưng ≤50–60% RAM => nếu build HNSW song song, đảm bảo `--shm-size` (Docker) đủ lớn

## CHAIN — Edge cases

edge case 1 — vector trong CSV phải bọc nháy kép => edge case 2 — **dimension mismatch**: một dòng vector sai chiều → COPY/insert fail cả lô (COPY thường all-or-nothing trong một lệnh); validate chiều trước khi nạp => edge case 3 — **NULL embedding**: cho phép nếu cột nullable nhưng dòng đó không xuất hiện trong ANN search → backfill sau => edge case 4 — **upsert khi trùng**: COPY không hỗ trợ `ON CONFLICT` trực tiếp → mẹo COPY vào bảng tạm rồi `INSERT ... SELECT ... ON CONFLICT DO UPDATE` => và `register_vector` bắt buộc cho cả execute_values/copy để serialize vector

## CHAIN — Ingestion pipeline & bottleneck thật

bulk insert là một mắt trong pipeline nạp hàng triệu tài liệu => bước 1: chunk tài liệu dài (tránh truncation của model) => bước 2: sinh embedding theo BATCH (32–256/lần) trên GPU/API => bước 3: COPY (binary) vào bảng CHƯA index => bước 4: build index HNSW SAU khi nạp xong (`maintenance_work_mem` cao) => bước 5: verify (`EXPLAIN ANALYZE` thấy Index Scan, recall@k) => bottleneck thường KHÔNG phải insert mà là bước **sinh embedding** (API/GPU) => nhưng nếu insert row-by-row thì chính insert thành nút thắt → COPY gỡ nút đó

## CHAIN — Drop & rebuild index cho re-load lớn

khi nạp lại/nạp thêm rất nhiều vào bảng ĐÃ có index => nạp thêm lớn: cân nhắc `DROP INDEX` → bulk load → `CREATE INDEX CONCURRENTLY` lại (rẻ hơn maintain index suốt quá trình insert khối lượng lớn) => không thể mất search khi re-load: dùng `CREATE INDEX CONCURRENTLY` (không khóa) và/hoặc bảng **shadow + swap** => build nhanh hơn: tăng `max_parallel_maintenance_workers` để build trên nhiều core

## CHAIN — Idempotency & upsert (nạp lại an toàn)

pipeline chạy lại (retry, backfill) không được tạo dòng trùng => nhưng COPY không có `ON CONFLICT` => mẹo: `CREATE TEMP TABLE faqs_stage (LIKE faqs)` → COPY vào bảng tạm => rồi `INSERT INTO faqs SELECT ... FROM faqs_stage ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding` => đây là mẫu **"COPY → temp → upsert"** chuẩn cho ingestion production idempotent => idempotency nghĩa là job nạp *chạy lại được* sau lỗi mà không hỏng dữ liệu (tối quan trọng khi nạp nhiều giờ và có thể đứt giữa chừng)

## CHAIN — WAL, UNLOGGED, cost, monitoring

COPY vẫn ghi WAL → nạp khối lượng lớn tạo NHIỀU WAL => nhiều WAL có thể gây áp lực replication lag và dung lượng đĩa → theo dõi WAL generation & replica lag khi bulk load lớn => bảng **`UNLOGGED`** bỏ WAL → nạp nhanh hơn nhưng MẤT dữ liệu khi crash và không replicate → chỉ dùng cho staging/tái tạo được => cost ingestion chủ yếu ở *sinh embedding* (API token/GPU), không phải insert => cache embedding theo hash để khỏi re-embed khi nạp lại => monitoring: rows/giây, thời gian build index, WAL/replica lag, tỉ lệ dòng lỗi (dimension mismatch/NULL), RAM khi build

## CHAIN — Tổ chức & khép series

nói với PM/sếp: nạp dữ liệu ban đầu (vd 5 triệu tài liệu) là tác vụ *chạy một lần* nhưng tốn thời gian — phần lâu nhất là AI sinh embedding, không phải lưu vào DB => framing bằng **thời gian / chi phí compute / lịch vận hành**: dùng đúng kỹ thuật (COPY, nạp trước index sau) rút thời gian từ nhiều ngày xuống vài giờ, lên lịch nạp ngoài giờ cao điểm => roadmap: ingestion pipeline (chunk→embed→COPY→index→verify) là hạ tầng dùng lại → xây thành job/service có retry & idempotency, đừng viết script một lần rồi bỏ => khép series 7 mảnh: embed → index → cài pgvector → store/query → **bulk insert (bài này)** → FTS → chọn nền tảng

---

## BẢNG — Ba phương pháp & thứ tự tốc độ

| Phương pháp | Cơ chế | Tốc độ | Dùng khi |
|---|---|---|---|
| Row-by-row `INSERT` | 1 dòng / 1 câu / 1 transaction | 🐢 chậm nhất | gần như không bao giờ cho bulk |
| **Batch / multi-row `INSERT`** | nhiều dòng / 1 câu (pg-promise, execute_values) | 🚗 nhanh hơn nhiều | dữ liệu đến từ code/app, cần logic |
| **`COPY`** | giao thức nạp khối chuyên dụng (binary) | 🚀 **nhanh nhất** | có file/stream — mặc định khi có thể |

## BẢNG — Code thuộc lòng

**(a) COPY (nhanh nhất):**
```sql
\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)
-- CSV: cột embedding phải bọc nháy kép "[0.1,0.2,...]"
```

**(b) Node.js pg-promise (batch):**
```javascript
const pgp = require('pg-promise')();
const db  = pgp('postgresql://...');
const cs  = new pgp.helpers.ColumnSet(['text','embedding'], {table:'faqs'});
const rows = [{ text:'...', embedding:'[' + emb.join(',') + ']' }, /* ... */];
await db.none(pgp.helpers.insert(rows, cs));   // 1 câu INSERT nhiều dòng
```

**(c) Python — cũ vs mới:**
```python
# psycopg2 (bài gốc):
from psycopg2.extras import execute_values
from pgvector.psycopg2 import register_vector
register_vector(conn)
execute_values(cur, "INSERT INTO faqs (text,embedding) VALUES %s", data)
conn.commit()                                  # BẮT BUỘC (không autocommit)

# psycopg3 (nhanh nhất, khuyến nghị):
from pgvector.psycopg import register_vector
register_vector(conn)
with cur.copy("COPY faqs (text,embedding) FROM STDIN WITH (FORMAT BINARY)") as cp:
    cp.set_types(["text","vector"])
    for t,e in rows: cp.write_row([t,e])       # stream lazy
conn.commit()
```

**(d) Nạp-trước-index-sau:**
```sql
CREATE TABLE faqs (id serial PRIMARY KEY, text text, embedding vector(512));  -- KHÔNG index
\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)   -- nạp hết
SET maintenance_work_mem='2GB';                                              -- rồi build
CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
```

**(e) Idempotent (COPY → temp → upsert):**
```sql
CREATE TEMP TABLE faqs_stage (LIKE faqs INCLUDING DEFAULTS);
\copy faqs_stage (text, embedding) FROM 'batch.csv' WITH (FORMAT csv)
INSERT INTO faqs (text, embedding)
SELECT text, embedding FROM faqs_stage
ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;
```

## BẢNG — Khung system design (nạp 10 triệu doc, nhanh nhất, không gián đoạn, chạy lại được)

1. **Clarify:** nguồn (file/DB/stream)? embedding local hay API? có downtime không? ngân sách?
2. **Sinh embedding theo batch** (bottleneck thật): worker song song, batch 128–256, cache theo hash để nạp lại khỏi trả tiền lại.
3. **Nạp bằng COPY (binary)** vào bảng **chưa index** (hoặc staging) — nhanh nhất; chia chunk để chịu lỗi.
4. **Idempotent:** COPY → temp table → `INSERT ... ON CONFLICT DO UPDATE`.
5. **Build index SAU:** `CREATE INDEX CONCURRENTLY ... hnsw ...` với `maintenance_work_mem` cao + parallel workers → không khóa service.
6. **Bảo vệ service:** bảng shadow rồi swap, hoặc index CONCURRENTLY; theo dõi WAL/replica lag, giãn tốc nếu lag tăng.
7. **Verify:** `EXPLAIN ANALYZE` (Index Scan), đo recall@k, đếm rows.
8. **Tự đánh giá:** bottleneck là *embedding generation* chứ không phải insert; COPY đảm bảo DB không thành nút thắt; idempotency cho phép chạy lại = tư duy staff.

## BẢNG — Mental models

- **"Xe tải vs bê tay"** → COPY vs row-by-row; số chuyến (transaction) quyết định.
- **"Nạp trước, index sau"** → đừng bắt DB vừa nạp vừa vá index.
- **"COPY → temp → upsert"** → mẫu idempotent cho ingestion.
- **"Bottleneck là embedding, không phải insert"** → nhìn đúng nút thắt.
- **"Commit hay mất trắng"** → psycopg2 không autocommit.

---

## TỪ KHÓA MỒI

- Vì sao one-by-one chậm → **"mỗi INSERT một transaction"**
- Analogy & thứ tự tốc độ → **"row-by-row ≪ batch ≪ COPY"**
- COPY từ CSV → **"bọc nháy kép"**
- pg-promise → **"ColumnSet + helpers.insert"**
- psycopg2 & commit → **"không autocommit"**
- psycopg3 copy() → **"stream lazy"**
- Vì sao COPY nhanh nhất → **"binary + ring buffer"**
- Nạp trước index sau → **"nguyên tắc vàng"**
- Batch size → **"1k-10k điểm ngọt"**
- Edge cases → **"dimension mismatch fail cả lô"**
- Ingestion pipeline → **"bottleneck là embedding"**
- Drop & rebuild → **"DROP → load → CONCURRENTLY"**
- Idempotency → **"COPY → temp → upsert"**
- WAL/UNLOGGED/cost → **"UNLOGGED chỉ staging"**
- Tổ chức & series → **"nhiều ngày xuống vài giờ"**

---

## ĐÃ PHỦ

**Vì sao & thứ tự:** mỗi INSERT = 1 transaction overhead (WAL/commit/fsync/round-trip); 1M dòng = 1M overhead; vector nặng vài KB; analogy xe tải; thứ tự row-by-row ≪ batch ≪ COPY (đính chính COPY nhanh nhất).

**COPY:** chuyển file↔bảng 2 chiều; server-side `COPY FROM` (quyền cao) vs client-side `\copy`; gotcha vector-trong-CSV bọc nháy kép; xuất ngược `\copy (SELECT) TO`; vì sao nhanh nhất (ít transaction/parse, binary FORMAT BINARY, ring buffer, vẫn ghi WAL).

**Client batch:** pg-promise Node (ColumnSet, format `'[...]'`, `helpers.insert`, `db.none`, chunk 1k-10k, pg-copy-streams); psycopg2 Python (register_vector, execute_values từ extras, commit bắt buộc, không autocommit); psycopg3 (không execute_values → executemany()/copy(); set_types; write_row; stream lazy binary).

**Con số/lệnh chốt:** batch 1k-10k điểm ngọt; embedding batch 32-256; maintenance_work_mem ≤50-60% RAM; --shm-size Docker cho HNSW song song; max_parallel_maintenance_workers.

**Nguyên tắc vàng:** nạp trước index sau (bảng đã có index → mỗi insert cập nhật index chậm, HNSW ef_construction; IVFFlat bắt buộc vì cần data k-means; HNSW cũng nên); cách đúng bảng-không-index → COPY → maintenance_work_mem → CREATE INDEX.

**3 lỗi:** quên commit() (psycopg2), quên register_vector/format sai, một câu INSERT/COPY quá khổng lồ → chunk + transaction riêng.

**Edge cases:** vector CSV nháy kép; dimension mismatch → COPY all-or-nothing fail cả lô; NULL embedding không xuất hiện ANN; upsert COPY không có ON CONFLICT → temp + upsert; register_vector bắt buộc.

**Staff/pipeline:** ingestion pipeline 5 bước (chunk→embed batch→COPY binary chưa index→build HNSW sau→verify), bottleneck là embedding không phải insert; drop & rebuild (DROP INDEX → load → CREATE INDEX CONCURRENTLY; shadow+swap; parallel workers); idempotency "COPY → temp → upsert" ON CONFLICT DO UPDATE + job chạy lại được; WAL (COPY ghi WAL → replication lag/dung lượng), UNLOGGED bỏ WAL chỉ staging (mất khi crash, không replicate); cost ở sinh embedding + cache hash; monitoring rows/giây/build time/WAL-replica lag/dòng lỗi/RAM; tổ chức (framing thời gian/compute/lịch, pipeline là job/service retry+idempotency); khung system design 8 bước.

**Đính chính bài gốc:** COPY nhanh nhất; psycopg3 không execute_values; nạp trước index sau; register_vector + commit.

**Học tiếp (bài nêu):** streaming ingestion (Kafka → embed → COPY), incremental re-embedding khi đổi model, build HNSW song song ở hàng chục triệu vector.
