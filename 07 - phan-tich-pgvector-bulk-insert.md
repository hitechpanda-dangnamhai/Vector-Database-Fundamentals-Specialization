# Phân tích chain học tập — Bulk Insert Vector vào pgvector (COPY, pg-promise, psycopg)

> Mục tiêu: tách *suy ra được* (nhóm A) khỏi *phải ghi nhớ* (nhóm B), đánh dấu **⚠** chỗ danh sách/logic nhảy, rút **xương sống**.

Đây là **mảnh 7** — bài ops thứ hai (cặp với file 6), về **nạp hàng triệu vector vào NHANH**. Khác file 6 (nặng cú pháp thuần), bài này **giàu nhóm A**: phần "vì sao" là trọng tâm — vì sao insert từng dòng chậm, vì sao COPY nhanh nhất, vì sao nạp-trước-index-sau, vì sao nút thắt là *sinh embedding* chứ không phải insert. Đây là bài đáng "hiểu" chứ không chỉ "thuộc". Có 4 đính chính bài gốc.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Vì sao insert từng dòng chậm

**A — Nhân quả**
- mỗi `INSERT` một dòng = một **transaction độc lập** => phải trả chi phí cố định: ghi WAL, commit, fsync, network round-trip — *vì* mỗi transaction có overhead không đổi.
- 1 triệu dòng riêng lẻ => trả phí đó 1 triệu lần → overhead khổng lồ — *vì* overhead nhân theo số transaction.
- vector nặng (512–1536 chiều, vài KB/dòng) => 1 triệu round-trip mang từng khối — *vì* mỗi dòng là một chuyến mang payload lớn.
- => cứu bằng **bulk insert**: gom nhiều dòng vào ÍT transaction/round-trip.

> Chain này **thuần nhóm A** — hiểu là xong, đừng chép. B chỉ là các tên WAL/commit/fsync.

**Xương sống**: mỗi INSERT = một transaction có overhead cố định → nhân triệu lần = khổng lồ → gom vào ít transaction (bulk).

---

## Chain 2 — Analogy & thứ tự tốc độ

**A — Nhân quả**
- số **chuyến** (transaction/round-trip) quyết định tốc độ, không phải số món — *vì* overhead nằm ở mỗi chuyến.

**B — Ghi nhớ**
- **row-by-row** = bê từng ghế (chậm nhất) · **batch/multi-row** = xe đẩy 50 món (nhanh hơn) · **COPY** = xe tải một chuyến (nhanh nhất) — *mental model/thứ tự*.
- ⚠ **Đính chính:** COPY **KHÔNG** ngang hàng batch — thứ tự **row-by-row ≪ batch ≪ COPY**. Batch (pg-promise/execute_values) dùng khi dữ liệu đến từ *code*, không phải file.

**Xương sống**: số chuyến quyết định tốc độ → row-by-row ≪ batch ≪ COPY.

---

## Chain 3 — COPY từ CSV

**A — Nhân quả**
- embedding `[0.1,0.2,...]` chứa dấu phẩy => trong CSV phải **BỌC NHÁY KÉP**; quên bọc → parser tách mỗi số thành một cột → lỗi — *vì* CSV phân tách theo dấu phẩy.

**B — Ghi nhớ**
- COPY chuyển dữ liệu **file ↔ bảng** (hai chiều) — *định nghĩa*.
- Server-side `COPY faqs FROM '/path/file.csv' WITH (FORMAT csv, HEADER true)` — file trên **máy chủ**, cần quyền cao; `\copy` trong `psql` — file trên **máy client**, không cần quyền server — *cú pháp*.

**Xương sống**: COPY nối file↔bảng (\copy client / COPY server) → vector trong CSV phải bọc nháy kép.

---

## Chain 4 — pg-promise (Node batch)

**A — Nhân quả**
- (không) — chuỗi cú pháp.

**B — Ghi nhớ**
- Dữ liệu đến từ **code** (vừa sinh embedding) → dùng pg-promise batch; 4 bước:
  - `ColumnSet(['text','embedding'], {table:'faqs'})` (tái dùng, hiệu năng tốt);
  - format embedding thành literal `'[' + emb.join(',') + ']'`;
  - `pgp.helpers.insert(rows, cs)` dựng MỘT câu INSERT nhiều dòng;
  - `db.none(query)` chạy không mong trả row.
- Lô lớn → chia chunk **1k–10k**/lần; nhanh nhất trong Node dùng COPY qua `pg-copy-streams` — *cú pháp/con số*.

> Chain này **chủ yếu nhóm B** — cú pháp pg-promise. Học như thẻ code.

**Xương sống**: dữ liệu từ code → pg-promise: ColumnSet → format '[...]' → helpers.insert (1 câu) → db.none.

---

## Chain 5 — psycopg2 & commit

**A — Nhân quả**
- psycopg2 mặc định **KHÔNG autocommit** => quên `conn.commit()` thì dữ liệu không được lưu (rollback khi đóng) — *vì* không có commit thì transaction chưa được ghi bền.

**B — Ghi nhớ**
- `register_vector(conn)` để serialize vector; `execute_values(cur, "INSERT ... VALUES %s", data)` (từ `psycopg2.extras`) gộp nhiều dòng; `conn.commit()` một lần **BẮT BUỘC** — *cú pháp*.
- ⚠ **Đính chính:** psycopg2 là thư viện **cũ**; psycopg (v3) **không còn `execute_values`**.

> ⭐ Mental model: **"commit hay mất trắng"** — psycopg2 không autocommit.

**Xương sống**: psycopg2 register_vector → execute_values → commit BẮT BUỘC (không autocommit, quên = mất).

---

## Chain 6 — psycopg3 copy() (nhanh nhất hiện đại)

**A — Nhân quả**
- `copy.write_row` **stream LAZY** => không cần nạp cả triệu dòng vào RAM một lúc — *vì* dữ liệu chảy thẳng vào server từng dòng.

**B — Ghi nhớ**
- psycopg3 thay `execute_values` bằng `executemany()` (pipeline) hoặc `cursor.copy()` (nhanh nhất); `with cur.copy("COPY faqs FROM STDIN WITH (FORMAT BINARY)") as copy:` → `copy.set_types(["text","vector"])` → `copy.write_row([text, emb])` — *cú pháp*; cách khuyến nghị cho ingestion lớn 2026.

**Xương sống**: psycopg3 copy() FORMAT BINARY → set_types → write_row (stream lazy, khỏi giữ hết trong RAM).

---

## Chain 7 — Vì sao COPY nhanh nhất

**A — Nhân quả**
- COPY nạp cả khối trong một luồng => **ít transaction/parse**: không parse lại từng INSERT, không lặp overhead commit — *vì* một lệnh xử lý cả khối.
- **giao thức binary** (`FORMAT BINARY`) => gửi nhị phân thô, tiết kiệm băng thông + khỏi parse text phía server — *vì* không phải chuyển đổi text↔binary.
- **ring buffer riêng** => giảm áp lực lên shared buffers — *vì* COPY có vùng đệm chuyên dụng.
- COPY vẫn ghi WAL => replication & point-in-time recovery **không mất** — *vì* nhanh nhưng vẫn durable (đây là điểm khắc: nhanh ≠ mất an toàn).

> Chain này **thuần nhóm A** — hiểu 3–4 lý do là nắm. B chỉ là tên FORMAT BINARY / ring buffer / WAL.

**Xương sống**: COPY = ít parse/commit + binary + ring buffer, vẫn ghi WAL → nhanh nhất mà vẫn an toàn.

---

## Chain 8 — Nguyên tắc vàng: nạp trước, index sau ↩(≈ file 1, 4, 6)

**A — Nhân quả**
- bảng ĐÃ có index HNSW/IVFFlat => mỗi insert phải **cập nhật index** → chậm hẳn (HNSW `ef_construction` cao càng chậm) — *vì* phải maintain graph/cụm từng dòng.
- => nguyên tắc vàng: nạp toàn bộ TRƯỚC, tạo index SAU — *vì* build một lần rẻ hơn maintain từng dòng.
- IVFFlat **BẮT BUỘC** kiểu này => vì cần dữ liệu sẵn để k-means; HNSW cũng nên (build nhanh hơn).

**B — Ghi nhớ**
- Cách đúng: bảng KHÔNG index → COPY/bulk toàn bộ → `SET maintenance_work_mem` cao → `CREATE INDEX` một lần — *quy trình*.

**Xương sống**: bảng có index → mỗi insert vá index chậm → nạp trước, index sau (IVFFlat bắt buộc vì cần data k-means).

---

## Chain 9 — Batch size & transaction

**A — Nhân quả**
- batch quá nhỏ => nhiều round-trip; quá lớn => tốn RAM + một lỗi hỏng cả lô lớn — *vì* hai cực đều có cái giá.
- bọc mỗi lô trong transaction (`BEGIN...COMMIT`) => ít commit hơn row-by-row nhưng vẫn giới hạn thiệt hại nếu lỗi — *vì* lỗi chỉ ảnh hưởng một lô.

**B — Ghi nhớ**
- Batch size **1.000–10.000 dòng/lô** là điểm ngọt; `maintenance_work_mem` cho build index ≤ **50–60% RAM**; build HNSW song song cần `--shm-size` (Docker) đủ lớn — *con số*.

**Xương sống**: batch 1k–10k (quá nhỏ nhiều round-trip / quá lớn tốn RAM + rủi ro) → mỗi lô một transaction.

---

## Chain 10 — Edge cases

**A — Nhân quả (mỗi edge có "vì" riêng)**
- **dimension mismatch**: một dòng sai chiều => COPY/insert fail **cả lô** (all-or-nothing) → validate chiều trước khi nạp.
- **NULL embedding**: cho phép nếu cột nullable nhưng dòng đó **không xuất hiện** trong ANN search → backfill sau.
- **upsert khi trùng**: COPY không hỗ trợ `ON CONFLICT` trực tiếp → mẹo COPY vào bảng tạm rồi `INSERT ... SELECT ... ON CONFLICT DO UPDATE`.

**B — Ghi nhớ**
- Vector trong CSV bọc nháy kép; `register_vector` bắt buộc cho execute_values/copy — *quy tắc*.

> ⚠ **Danh sách 4 edge case**, không phải chuỗi nhân quả. Học như checklist thủ sẵn.

**Xương sống**: (checklist) nháy kép / dimension mismatch fail cả lô / NULL vô hình ANN / upsert qua temp.

---

## Chain 11 — Ingestion pipeline & bottleneck thật

**A — Nhân quả**
- bottleneck thường **KHÔNG phải insert** mà là bước **sinh embedding** (API/GPU) — *vì* encode chậm hơn nhiều so với lưu.
- nhưng nếu insert **row-by-row** thì chính insert thành nút thắt => COPY gỡ nút đó — *vì* insert kém sẽ vượt cả embedding.

**B — Ghi nhớ (5 bước pipeline)**
- (1) chunk tài liệu dài (tránh truncation) → (2) sinh embedding theo **BATCH (32–256)** GPU/API → (3) **COPY (binary)** vào bảng **CHƯA index** → (4) build HNSW **SAU** (`maintenance_work_mem` cao) → (5) verify (`EXPLAIN ANALYZE` Index Scan, recall@k) — *quy trình*.

> ⭐ Mental model: **"bottleneck là embedding, không phải insert"** — nhìn đúng nút thắt.

**Xương sống**: pipeline chunk→embed batch→COPY chưa index→build index sau→verify; nút thắt thật là embedding (nhưng insert kém cũng thành nút thắt).

---

## Chain 12 — Drop & rebuild index cho re-load lớn

**A — Nhân quả**
- nạp thêm rất nhiều vào bảng ĐÃ có index => cân nhắc `DROP INDEX` → bulk load → `CREATE INDEX CONCURRENTLY` lại — *vì* rẻ hơn maintain index suốt quá trình insert khối lượng lớn.
- không được mất search khi re-load => `CREATE INDEX CONCURRENTLY` (không khóa) và/hoặc **shadow + swap** — *vì* cần giữ service chạy.

**B — Ghi nhớ**
- Build nhanh hơn: tăng `max_parallel_maintenance_workers` (build trên nhiều core) — *cú pháp*.

**Xương sống**: re-load lớn → DROP index → bulk load → CREATE INDEX CONCURRENTLY (shadow+swap để không mất search).

---

## Chain 13 — Idempotency & upsert (nạp lại an toàn)

**A — Nhân quả**
- pipeline chạy lại (retry, backfill) không được tạo dòng trùng, nhưng COPY không có `ON CONFLICT` => mẹo **COPY → temp → upsert** — *vì* cần đích đến có `ON CONFLICT` mà COPY không cung cấp.

**B — Ghi nhớ**
- `CREATE TEMP TABLE faqs_stage (LIKE faqs)` → COPY vào temp → `INSERT INTO faqs SELECT ... ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding` — *cú pháp/pattern*.
- **idempotency** = job nạp *chạy lại được* sau lỗi mà không hỏng dữ liệu (tối quan trọng khi nạp nhiều giờ có thể đứt giữa chừng) — *định nghĩa*.

**Xương sống**: cần chạy-lại-được nhưng COPY thiếu ON CONFLICT → mẫu "COPY → temp → upsert".

---

## Chain 14 — WAL, UNLOGGED, cost, monitoring

**A — Nhân quả**
- COPY vẫn ghi WAL => nạp lớn tạo NHIỀU WAL => áp lực replication lag + dung lượng đĩa → theo dõi WAL generation & replica lag — *vì* mỗi thay đổi phải vào WAL và replica phải áp lại.
- bảng **`UNLOGGED`** bỏ WAL => nạp nhanh hơn nhưng MẤT dữ liệu khi crash + không replicate → chỉ staging/tái tạo được — *vì* không có WAL thì không phục hồi/nhân bản được.
- embedding xác định => cache theo hash để khỏi re-embed khi nạp lại — *vì* cùng input cho cùng vector.

**B — Ghi nhớ**
- Cost ingestion chủ yếu ở *sinh embedding* (API token/GPU), không phải insert; monitoring: rows/giây, thời gian build index, WAL/replica lag, tỉ lệ dòng lỗi, RAM khi build — *dữ kiện*.

**Xương sống**: COPY ghi nhiều WAL → theo dõi replica lag; UNLOGGED nhanh hơn nhưng chỉ staging; cost & bottleneck ở embedding → cache hash.

---

## Chain 15 — Tổ chức & khép series

**A — Nhân quả**
- ingestion pipeline là hạ tầng dùng lại => xây thành job/service có retry & idempotency, đừng script một lần rồi bỏ — *vì* sẽ chạy lại nhiều lần.

**B — Ghi nhớ**
- Framing cho PM/sếp: nạp ban đầu là tác vụ *chạy một lần* nhưng tốn — phần lâu nhất là **AI sinh embedding**, không phải lưu DB; dùng đúng kỹ thuật (COPY, nạp trước index sau) rút từ **nhiều ngày xuống vài giờ** — *khuyến nghị*.
- Khép series 7 mảnh: embed → index → cài pgvector → store/query → **bulk insert** → FTS → chọn nền tảng — *tổng hợp*.

**Xương sống**: nạp ban đầu tốn (chủ yếu vì embedding) → dùng COPY + nạp-trước-index-sau rút ngày xuống giờ → pipeline là job/service có retry.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (file 7)

> insert từng dòng = một transaction có **overhead cố định** (WAL/commit/fsync/round-trip) → nhân triệu lần = khổng lồ → **bulk insert** gom vào ít chuyến → thứ tự **row-by-row ≪ batch ≪ COPY** → **COPY nhanh nhất** (ít parse/commit + binary + ring buffer, **vẫn ghi WAL** nên an toàn) → COPY từ file (\copy client / COPY server), vector-CSV **bọc nháy kép** → dữ liệu từ code thì **batch** (pg-promise / psycopg2 execute_values → psycopg3 `copy()` stream lazy) → psycopg2 phải **commit** (không autocommit) → **nguyên tắc vàng: nạp trước, index sau** (index vá mỗi insert thì chậm; IVFFlat bắt buộc) → batch size **1k–10k** điểm ngọt → build index sau với `maintenance_work_mem` cao + parallel → re-load lớn: **DROP → load → CREATE INDEX CONCURRENTLY** (shadow+swap) → **idempotency: COPY → temp → upsert** → COPY tạo nhiều **WAL** → theo dõi replica lag; UNLOGGED chỉ staging → **bottleneck thật là sinh embedding**, không phải insert → cache theo hash → pipeline là job/service có retry.

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (7 file)

> **text → [ENCODE] embed (f3)** → **[INDEX] B-tree gãy → ANN (f4)** → **[STORE & SEARCH] pgvector: khái niệm (f1) + cài đặt/ops (f6) + nạp dữ liệu/bulk (f7, bài này)** → **[KEYWORD] FTS (f2)** → **[HYBRID] RRF** → **[CHỌN NỀN TẢNG] (f5)** → **RAG**.

File 6 và 7 là **tầng ops của pgvector** (file 1 là tầng concept): f6 = *cài & vận hành*, f7 = *nạp dữ liệu vào nhanh*.

---

## 8. MẪU HÌNH LẶP LẠI

**Mẫu hình 1 — Nạp trước, index sau (đừng bắt DB vừa nạp vừa vá index).** ⭐ nói kỹ nhất ở bài này.
Chain 8, 11, 12 ↔ file 1 chain 20, file 4 chain 14, file 6 chain 12. *Vì sao là một*: build index một lượt rẻ hơn maintain incremental từng dòng; IVFFlat còn *bắt buộc* vì k-means cần data sẵn.

**Mẫu hình 2 — Bottleneck thật là *sinh embedding*, không phải insert/lưu.** ⭐ nối thẳng file 3.
Chain 11, 14, 15 ↔ file 3 chain 11 (bottleneck = tạo vector). *Vì sao là một*: encode chậm hơn store nhiều lần → tối ưu đúng chỗ (batch embedding + cache hash), đừng tối ưu nhầm insert.

**Mẫu hình 3 — Gom lô để trả phí cố định ít lần.**
Chain 1, 2, 9 (batch/COPY) ↔ file 3 (batch embed 32–256) ↔ file 1 (two-stage trên N nhỏ). *Vì sao là một*: mọi thao tác có overhead cố định thì gom lô để chia đều phí — cùng gốc với "batch embedding".

**Mẫu hình 4 — Đánh đổi durability/an toàn lấy tốc độ (chỉ dùng khi chấp nhận được).**
Chain 14 (UNLOGGED bỏ WAL) ↔ file 6 (`-march=native` bỏ portability) ↔ file 1 (quantization bỏ recall). *Vì sao là một*: bỏ một thuộc tính an toàn để nhanh hơn — phải biết mình đang đánh đổi gì và chỉ dùng ở nơi cho phép.

**Mẫu hình 5 — Bẫy "quên bước ẩn" (lỗi im lặng người mới).**
Quên `commit()` = mất trắng (chain 5) · quên `register_vector` · quên bọc nháy kép (chain 3, 10) ↔ file 6 (cài file ≠ bật extension, registerType là cây cầu). *Vì sao là một*: có một bước bắt buộc dễ bỏ qua, bỏ là hỏng mà không rõ tại sao.

**Mẫu hình 6 — Idempotency / chạy-lại-được cho job dài.**
Chain 13 (COPY → temp → upsert) ↔ file 6 (migration + rollback rõ). *Vì sao là một*: job nạp nhiều giờ có thể đứt → phải thiết kế để retry an toàn.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

### Con số
- Batch size **1.000–10.000 dòng/lô** (điểm ngọt); embedding batch **32–256**/lần.
- `maintenance_work_mem` ≤ **50–60% RAM**; pg-promise chunk **1k–10k**.

### Định nghĩa & tên gọi
- 3 phương pháp: **row-by-row** / **batch (multi-row)** / **COPY**; thứ tự tốc độ **row-by-row ≪ batch ≪ COPY**.
- Transaction overhead: WAL, commit, fsync, round-trip.
- **COPY** (server, quyền cao) vs **`\copy`** (client); **FORMAT BINARY**; **ring buffer** vs shared buffers.
- **pg-promise**: `ColumnSet`, `helpers.insert`, `db.none`, `pg-copy-streams`.
- **psycopg2**: `execute_values` (`psycopg2.extras`), `register_vector`, `commit` (không autocommit).
- **psycopg3**: `executemany()`, `cursor.copy()`, `set_types`, `write_row` (stream lazy).
- **Nguyên tắc vàng**: nạp trước, index sau.
- **UNLOGGED** table; **WAL**; **replication lag**; PITR.
- **idempotency**; mẫu **"COPY → temp → upsert"**; `ON CONFLICT DO UPDATE`.
- **DROP INDEX → load → CREATE INDEX CONCURRENTLY**; **shadow + swap**; `max_parallel_maintenance_workers`; `--shm-size`.
- Bottleneck = **sinh embedding**; **cache theo hash**.
- Mental models: "xe tải vs bê tay", "nạp trước index sau", "COPY → temp → upsert", "bottleneck là embedding", "commit hay mất trắng".

### ⚠ Bốn đính chính bài gốc (phải ghi đè)
1. **COPY là NHANH NHẤT** — không ngang hàng batch. Thứ tự: row-by-row ≪ batch ≪ COPY.
2. `psycopg2` cũ; **`psycopg` (v3)** không còn `execute_values` → `executemany()` hoặc `cursor.copy()`.
3. Nguyên tắc vàng: **nạp toàn bộ TRƯỚC, tạo index SAU**.
4. Vector cần **`register_vector`** (hoặc format đúng `'[1,2,3]'`) mới serialize.

### Cú pháp (thuộc lòng)
- `\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)` — cột embedding **bọc nháy kép**.
- **pg-promise:** `new pgp.helpers.ColumnSet([...], {table})` → `db.none(pgp.helpers.insert(rows, cs))`.
- **psycopg2:** `register_vector(conn)` → `execute_values(cur, "INSERT ... VALUES %s", data)` → `conn.commit()`.
- **psycopg3:** `with cur.copy("COPY faqs (text,embedding) FROM STDIN WITH (FORMAT BINARY)") as cp: cp.set_types(["text","vector"]); cp.write_row([t,e])`.
- **Nạp-trước-index-sau:** `CREATE TABLE ... (KHÔNG index)` → `\copy ...` → `SET maintenance_work_mem='2GB'` → `CREATE INDEX ... USING hnsw ...`.
- **Idempotent:** `CREATE TEMP TABLE faqs_stage (LIKE faqs)` → `\copy faqs_stage ...` → `INSERT INTO faqs SELECT ... ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding`.

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào.

1. Vì sao insert **từng dòng** chậm ở quy mô triệu dòng?
2. Vì sao **COPY nhanh nhất** — nêu ít nhất 3 lý do?
3. Vì sao thứ tự là **row-by-row ≪ batch ≪ COPY** chứ không phải batch ≈ COPY?
4. Vì sao vector trong CSV phải **bọc nháy kép**?
5. Vì sao psycopg2 quên `commit()` = **mất trắng** dữ liệu?
6. Vì sao **"nạp trước, index sau"** là nguyên tắc vàng — và vì sao IVFFlat *bắt buộc*?
7. Vì sao batch size **quá nhỏ VÀ quá lớn** đều tệ?
8. Vì sao bottleneck ingestion thường là **sinh embedding** chứ không phải insert?
9. Vì sao COPY vẫn **"an toàn"** dù nhanh (WAL)?
10. Vì sao **UNLOGGED** nhanh hơn nhưng chỉ dùng cho staging?
11. Vì sao cần **idempotency**, và mẫu "COPY → temp → upsert" giải quyết thế nào?
12. Vì sao re-load lớn nên **DROP INDEX** rồi rebuild thay vì giữ index?
13. Vì sao COPY tạo **nhiều WAL**, và điều đó ảnh hưởng replica thế nào?
14. Vì sao **cache embedding theo hash** tiết kiệm khi nạp lại?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- Bài này **giàu nhóm A** hơn file 6 — nên **hiểu**, đừng chỉ thuộc: chain 1, 7 (vì sao chậm / vì sao COPY nhanh) và chain 8, 11 (nạp trước index sau / bottleneck là embedding) là phần cốt lõi, nắm logic là nhớ được.
- **Nhóm B** (mục 9) là các phương ngữ cú pháp (pg-promise / psycopg2 / psycopg3) + con số batch — học như thẻ code.
- **Tái tạo**: dựng lại xương sống (chậm → gom lô → COPY → nạp trước index sau → idempotency → WAL → bottleneck embedding), rồi ghép vào **meta-xương sống 7 file**. File 6 + 7 = tầng ops của pgvector.
- **4 đính chính** phải ghi đè (COPY nhanh nhất / psycopg3 không execute_values / nạp trước index sau / register_vector).
