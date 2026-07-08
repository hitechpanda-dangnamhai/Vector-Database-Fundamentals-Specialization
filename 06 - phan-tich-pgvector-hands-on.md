# Phân tích chain học tập — Cài & Dùng pgvector Thực Chiến (Install → Insert → Query → Node.js)

> Mục tiêu: tách *suy ra được* (nhóm A) khỏi *phải ghi nhớ* (nhóm B), đánh dấu **⚠** chỗ danh sách/logic nhảy, rút **xương sống**.

Đây là **mảnh 6** — bản **THỰC HÀNH** của pgvector, đi cặp với file 1 (khái niệm/cơ chế). File 1 dạy *tại sao & cơ chế*; file này dạy *tay chân* — cài ra sao, gõ lệnh nào, kết nối app thế nào. Vì là bài ops nên **rất nặng nhóm B** (lệnh, cú pháp, quy trình). Phần A ít nhưng đắt: đó là "vì sao" của từng quyết định vận hành (vì sao build tay crash máy khác, vì sao cần registerType, vì sao CONCURRENTLY…). Nhiều chain là **danh sách lỗi/quy trình** — học như checklist.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Yêu cầu & hai bước cài

**A — Nhân quả**
- dùng managed => bỏ luôn bước cài binary, chỉ còn bật extension — *vì* provider đã cài sẵn binary.

**B — Ghi nhớ**
- pgvector yêu cầu **PostgreSQL 13+** (prebuilt khuyến nghị 15+), kiểm `SELECT version()` — *đính chính/con số* (bài gốc nói 12, đã bỏ).
- Cài gồm **HAI bước tách biệt**: cài binary → bật extension — *mental model chốt*.
- Cách cài binary: **Docker** `pgvector/pgvector:pg17` / gói OS `apt install postgresql-17-pgvector` / **managed** / build-from-source (**khó nhất**) — *dữ kiện*.

> Chủ yếu nhóm B. Điểm khắc: cài là **hai bước**, không phải một.

**Xương sống**: PG 13+ → cài binary (Docker/gói/managed/source) → managed bỏ được bước binary.

---

## Chain 2 — Bật extension & verify

**A — Nhân quả**
- `make install` chỉ **chép file**, chưa đăng ký vào database => phải `CREATE EXTENSION` mới dùng được — *vì* "cài file" và "bật extension" là hai việc khác nhau (người mới hay tưởng là một).

**B — Ghi nhớ**
- `CREATE EXTENSION IF NOT EXISTS vector` (idempotent, cần quyền CREATE/superuser), **trong MỖI database** — *cú pháp*.
- Verify: `\dx vector` hoặc `SELECT extname, extversion FROM pg_extension WHERE extname='vector'` → thấy `vector | 0.8.x` — *cú pháp*.

> ⭐ Mental model chốt: **"cài file ≠ bật extension"** — hai bước, đừng gộp.

**Xương sống**: cài binary xong → CREATE EXTENSION mỗi database (cài file ≠ bật extension) → verify \dx.

---

## Chain 3 — Tạo bảng & dimension khớp model ↩(≈ file 1, 5)

**A — Nhân quả**
- nhét vector 384 chiều vào `vector(512)` => lỗi ngay (dimension mismatch) — *vì* độ dài cột buộc khớp output model.
- model đổi => số chiều đổi => phải sửa cột và re-embed — *vì* chiều gắn với model.

**B — Ghi nhớ**
- Số trong `vector(512)` = số chiều model; nên chuẩn hóa chiều ở tầng app trước insert — *quy tắc*.

**Xương sống**: cột vector(n) khớp chiều model → sai chiều thì lỗi → đổi model = sửa cột + re-embed.

---

## Chain 4 — Insert, query & cosine distance

**A — Nhân quả**
- `<=>` là cosine **distance** (nhỏ = gần) => `ORDER BY ... ASC` (mặc định) để câu giống nhất lên đầu — *vì* nhỏ hơn nghĩa là gần hơn.

**B — Ghi nhớ**
- Insert dạng chuỗi `'[a,b,c,...]'` (thực tế cắm mảng float model trả về); query giống nhất: `ORDER BY embedding <=> '[...]' LIMIT 1` — *cú pháp*.
- ⚠ **Đính chính:** `<=>` = cosine **distance** = `1 − cosine_similarity`, KHÔNG phải similarity.

**Xương sống**: insert '[...]' → query ORDER BY <=> LIMIT; <=> là distance (nhỏ=gần) nên ASC.

---

## Chain 5 — Ba toán tử distance & ops class ↩(≈ file 1 chain 6, file 4 chain 8)

**A — Nhân quả**
- `<#>` trả **âm** inner product => ASC vẫn = "gần nhất" — *vì* Postgres chỉ index ASC, phải đảo dấu để nhỏ=gần.
- ops class phải khớp toán tử => nếu không **index bị bỏ** — *vì* planner chỉ dùng index khi ops class trùng toán tử query.

**B — Ghi nhớ**
- `<->` L2 (`vector_l2_ops`) · `<=>` cosine distance (`vector_cosine_ops`) · `<#>` negative inner product (`vector_ip_ops`); text mặc định cosine — *cú pháp/quy tắc*.

**Xương sống**: 3 toán tử ↔ 3 ops class; <#> âm để ASC=gần; ops class phải khớp toán tử kẻo bỏ index.

---

## Chain 6 — Node.js registerType (bước hay quên)

**A — Nhân quả**
- không `registerType(client)` => driver `pg` trả vector dưới dạng **CHUỖI thô**, phải tự parse — *vì* driver mặc định không biết kiểu `vector`.

**B — Ghi nhớ**
- `npm install pg pgvector` → `require('pgvector/pg')` → `registerType(client)` → `pgvector.toSql(emb)` (mảng JS → literal vector) — *cú pháp*.
- Python tương đương: `register_vector(conn)` — *cú pháp*.

> ⭐ Mental model: **"registerType là cây cầu"** vector ↔ mảng. Không có nó, app nhận chuỗi.

**Xương sống**: require pgvector → registerType (cây cầu) → toSql; thiếu registerType thì vector về dạng chuỗi.

---

## Chain 7 — Ba lỗi cài đặt kinh điển

**A — Nhân quả (mỗi lỗi có "vì" riêng)**
- `type "vector" does not exist` sau CREATE EXTENSION => extension ở schema ngoài `search_path` → sửa search_path hoặc `CREATE EXTENSION vector SCHEMA public`.
- quên `CREATE EXTENSION` => `make install` chỉ chép file, phải bật trong *mỗi* database.
- sai lib path (build header PG 15 nhưng server PG 17) => `.so` sai chỗ, `CREATE EXTENSION` lỗi.
- cả ba => lý do nên dùng Docker/gói thay vì build tay — *vì* build tay dễ dính cả ba.

> ⚠ **Danh sách 3 lỗi**, không phải chuỗi nhân quả. Học như checklist chẩn đoán.

**Xương sống**: (checklist) schema/search_path / quên CREATE EXTENSION / sai .so path → nên dùng Docker/gói.

---

## Chain 8 — Build dưới nắp capo & -march=native

**A — Nhân quả**
- `make install` chép `.so` → `lib/`, `.control`+`.sql` → `extension/` => từ đó `CREATE EXTENSION` mới thấy — *vì* Postgres tìm extension ở đúng thư mục đó.
- build tay mặc định `-march=native` tối ưu cho CPU máy build => KHÔNG chạy được trên CPU khác (illegal instruction) => "binary build máy A crash máy B" — *vì* dùng instruction set mà CPU khác không có.
- Docker image build `OPTFLAGS=""` (tắt `-march=native`) => chạy mọi CPU — *vì* đánh đổi chút tốc độ lấy tính di động.

**B — Ghi nhớ**
- **PGXS**: `make` gọi `pg_config` tìm header bản Postgres đang cài, biên dịch C → `vector.so` + `.sql`/`.control` — *cơ chế/tên*.

**Xương sống**: PGXS build vector.so → make install đặt file đúng chỗ → -march=native không portable → Docker OPTFLAGS="" chạy mọi CPU.

---

## Chain 9 — EXPLAIN ANALYZE: có index ≠ dùng index ↩(≈ file 1 chain 12, file 4)

**A — Nhân quả**
- *có* index không nghĩa index *được dùng* => luôn `EXPLAIN ANALYZE` — *vì* planner có thể bỏ index.
- bảng nhỏ chạy Seq Scan **KHÔNG phải lỗi** — *vì* exact scan rẻ và cho recall 100% khi n nhỏ.

**B — Ghi nhớ**
- `Index Scan using ..._hnsw...` → index chạy; `Seq Scan` → planner bỏ index; nguyên nhân Seq Scan: bảng nhỏ / ops class không khớp / cost estimation — *dữ kiện chẩn đoán*.

> ⭐ Mental model: **"có index ≠ dùng index"** → luôn EXPLAIN ANALYZE.

**Xương sống**: có index ≠ dùng index → EXPLAIN ANALYZE → Seq Scan có thể do bảng nhỏ (không phải lỗi) / ops class sai / cost.

---

## Chain 10 — Upgrade & version skew

**A — Nhân quả**
- version skew: replica khác version => có thể lỗi khi áp WAL liên quan kiểu vector — *vì* WAL mang thao tác kiểu vector mà replica version khác không hiểu.

**B — Ghi nhớ**
- Nâng cấp: `ALTER EXTENSION vector UPDATE` trong mỗi database; đọc CHANGELOG trước (một số bản yêu cầu re-create IVFFlat); nâng trong cửa sổ bảo trì, test staging; giữ đồng bộ primary + mọi replica — *cú pháp/khuyến nghị*.

**Xương sống**: ALTER EXTENSION UPDATE (đọc CHANGELOG) → đồng bộ primary/replica → version skew gây lỗi WAL.

---

## Chain 11 — Đừng build source trên production

**A — Nhân quả**
- build tay trên prod là anti-pattern => vì phụ thuộc CPU/compiler (`-march=native`), khó tái lập, khó rollback, dễ lệch node.
- => thay bằng Docker image cố định tag => mọi môi trường giống hệt — *vì* image bất biến.

**B — Ghi nhớ**
- Thay thế: Docker tag cố định / gói OS pin version (Ansible/Terraform) / managed lo binary — *khuyến nghị*.
- Nguyên tắc: **pin phiên bản pgvector + Postgres**, coi như dependency có version — *nhận định chốt*.

**Xương sống**: build tay không tái lập → dùng Docker tag/gói pinned/managed → pin version như mọi dependency.

---

## Chain 12 — Migration prod & zero-downtime

**A — Nhân quả**
- `CREATE EXTENSION` cần quyền cao => đưa vào migration script chạy MỘT lần bởi admin, không rải rác trong app — *vì* app user không nên có quyền cao.
- build HNSW có thể hàng giờ + không được khóa write => `CREATE INDEX CONCURRENTLY` sau khi backfill xong — *vì* CONCURRENTLY không chặn ghi.
- mọi thứ là migration + dependency pinned => rollback rõ ràng (drop index/column, gỡ extension) — *vì* có version + bước rõ.

**B — Ghi nhớ**
- Thêm cột: `ALTER TABLE ADD COLUMN embedding vector(n)` **nullable** (nhanh); backfill theo lô ở nền (queue), không khóa; verify `\dx` + `EXPLAIN ANALYZE` — *cú pháp/quy trình*.

**Xương sống**: CREATE EXTENSION qua admin → thêm cột nullable → backfill nền → CREATE INDEX CONCURRENTLY → rollback rõ vì mọi thứ pinned.

---

## Chain 13 — Security & CI/CD

**A — Nhân quả**
- `CREATE EXTENSION` cần quyền cao => bật extension bằng migration của admin, app user chỉ INSERT/SELECT — *vì* least privilege.
- PgBouncer tái dùng connection => `SET` toàn cục (`hnsw.ef_search`) rò sang query người khác → dùng `SET LOCAL` trong transaction — *vì* pooler chia sẻ connection.
- test phải giống prod => CI dùng chính image `pgvector/pgvector:pgXX`; assert `EXPLAIN` có `Index Scan` — *vì* tránh regression âm thầm về seq scan.

**B — Ghi nhớ**
- App user chỉ cần INSERT/SELECT, KHÔNG superuser — *security*.

**Xương sống**: least privilege (admin bật extension) → SET LOCAL không SET global (PgBouncer) → CI dùng cùng image + assert Index Scan.

---

## Chain 14 — Tổ chức & khép series

**A — Nhân quả**
- cái cần đầu tư là **quy trình** (migration, index không khóa, pin version), không phải mua phần mềm — *vì* pgvector chỉ là extension trên DB sẵn có.
- nhiều team dùng Postgres => thống nhất một cách cài (Docker tag/gói pinned) giảm "chạy trên máy tôi" — *vì* đồng nhất môi trường.

**B — Ghi nhớ**
- Framing cho PM/sếp: **rủi ro thấp + quy trình chuẩn** — chỉ bật một extension, không dựng hệ thống mới — *khuyến nghị*.
- Khép series 6 mảnh: embed → index → **cài & dùng pgvector** → store/query khái niệm → keyword/FTS → chọn nền tảng — *tổng hợp*.

**Xương sống**: thêm vector = bật extension, không dựng hệ thống → đầu tư vào quy trình → chuẩn hóa một cách cài → dependency có vòng đời.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (file 6)

> pgvector cần **PG 13+** → cài **HAI bước** (binary → extension); managed bỏ bước binary → **"cài file ≠ bật extension"** (`CREATE EXTENSION` mỗi database) → tạo cột `vector(n)` **khớp chiều model** → insert `'[...]'`, query `ORDER BY <=> LIMIT` (`<=>` là cosine **distance**, nhỏ=gần → ASC) → 3 toán tử + **ops class phải khớp toán tử** → client cần **registerType/register_vector** (cây cầu vector↔mảng) → **"có index ≠ dùng index"** → luôn `EXPLAIN ANALYZE` → build source dùng `-march=native` **không portable** → dùng Docker/gói **pin version**, đừng build trên prod → migration prod: CREATE EXTENSION qua admin, cột nullable, backfill nền, **CREATE INDEX CONCURRENTLY** → security least-privilege + **SET LOCAL** + CI dùng cùng image + **assert Index Scan** → coi pgvector là **dependency có vòng đời**.

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (6 file)

> **text → [ENCODE] embed (f3)** → **[INDEX] B-tree gãy → ANN (f4)** → **[STORE & SEARCH] pgvector: khái niệm (f1) + thực chiến/ops (f6, bài này)** → **[KEYWORD] FTS (f2)** → **[HYBRID] RRF** → **[CHỌN NỀN TẢNG] RDBMS-native vs dedicated (f5)** → **RAG**.

File 6 là **bạn đồng hành thực hành của file 1**: file 1 trả lời *tại sao/cơ chế*, file 6 trả lời *cài & vận hành thế nào*. Cùng một chủ đề (pgvector), hai tầng — concept và ops.

---

## 8. MẪU HÌNH LẶP LẠI

**Mẫu hình 1 — "Hai việc trông như một, đừng gộp."** ⭐ đóng góp riêng của bài ops.
Chain 2 (cài file ≠ bật extension) · chain 9 (có index ≠ dùng index) · chain 6 (registerType là cây cầu, thiếu thì vector thành chuỗi). *Vì sao là một*: mỗi cặp có một bước ẩn mà người mới bỏ qua rồi "sao không chạy?". Bài học chung: tách rõ hai bước, verify bước thứ hai.

**Mẫu hình 2 — Lỗi im lặng.**
Seq Scan âm thầm khi ops class sai (chain 5, 9) · regression seq scan trong CI (chain 13) · version skew replica (chain 10) ↔ file 1/2/4/5. *Vì sao là một*: hệ không báo lỗi, chỉ chậm/thiếu → phải `EXPLAIN ANALYZE`, assert trong CI, đồng bộ version.

**Mẫu hình 3 — Ops class phải khớp toán tử.**
Chain 5 ↔ file 1 chain 6/8, file 4 chain 8. *Vì sao là một*: index chỉ dùng được khi ops class lúc build khớp toán tử lúc query.

**Mẫu hình 4 — Reproducibility: pin version, đừng "build lúc deploy".** ⭐ mẫu hình ops riêng.
Chain 8 (-march=native không portable) · 11 (đừng build trên prod) · 12 (migration + dependency pinned) · 14 (chuẩn hóa một cách cài). *Vì sao là một*: kết quả phải tái lập được qua mọi môi trường → coi pgvector như dependency có version, dùng image/gói cố định tag.

**Mẫu hình 5 — Đổi model = re-embed (chiều gắn model).**
Chain 3 ↔ file 1/3/5. *Vì sao là một*: cột `vector(n)` buộc khớp chiều model; đổi model = sửa cột + re-embed.

**Mẫu hình 6 — SET LOCAL không SET global (PgBouncer rò).**
Chain 13 ↔ file 1 chain 8. *Vì sao là một*: pooler tái dùng connection nên `SET` toàn cục rò sang session khác.

**Mẫu hình 7 — Operational simplicity: chỉ bật một extension, không dựng hệ thống.**
Chain 14 ↔ mọi file (đặc biệt file 5). *Vì sao là một*: luận điểm xuyên series — vector là tính năng cắm vào Postgres sẵn có.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

Bài ops → phần lớn là cú pháp & quy trình. Đây là phần chính cần thuộc.

### Con số / version
- **PostgreSQL 13+** (prebuilt khuyến nghị **15+**); pgvector **0.8.x**; Docker tag `pgvector/pgvector:pg17`.

### Định nghĩa & tên gọi / quy tắc
- Cài = **hai bước**: cài binary + bật extension.
- 5 cách cài: **Docker** / **apt-yum** / **managed** / Homebrew-conda-PGXN / **build-from-source** (khó nhất).
- `<->` L2 (`vector_l2_ops`) · `<=>` cosine distance (`vector_cosine_ops`) · `<#>` negative inner product (`vector_ip_ops`); text → cosine.
- `<=>` = cosine **distance** = `1 − cosine_similarity`; **nhỏ = gần → ASC**.
- Client: `registerType` (Node) / `register_vector` (Python) / `toSql` — cây cầu vector↔mảng.
- **PGXS**, `pg_config`, `vector.so`, `.control`/`.sql`; `-march=native` (không portable) → `OPTFLAGS=""`.
- `EXPLAIN ANALYZE` → `Index Scan` vs `Seq Scan`; "có index ≠ dùng index".
- `ALTER EXTENSION vector UPDATE`; **version skew** (WAL); `CREATE INDEX CONCURRENTLY`; cột **nullable** + backfill.
- **SET LOCAL** vs SET global (PgBouncer); least privilege (app user chỉ INSERT/SELECT); pin version = dependency có vòng đời.
- Mental models: "cài file ≠ bật extension", "có index ≠ dùng index", "build tay = crash lạ máy khác", "pin version như mọi dependency", "registerType là cây cầu".

### ⚠ Bốn đính chính bài gốc (phải ghi đè)
1. **PostgreSQL 13+** (bài gốc nói 12 — đã bỏ; prebuilt 15+).
2. Build-from-source là cách **KHÓ NHẤT** — ưu tiên Docker / gói / managed.
3. `<=>` là cosine **distance** (`= 1 − cosine_similarity`), không phải similarity; ASC = gần.
4. Node.js: sau `require('pgvector/pg')` phải **`registerType`** mới parse `vector` ↔ mảng đúng.

### Cú pháp (thuộc lòng)
- `docker run -d --name pgv -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg17`
- `CREATE EXTENSION IF NOT EXISTS vector;`
- `CREATE TABLE items (id serial PRIMARY KEY, content text, embedding vector(512));`
- `CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);` · verify `\dx vector`
- `INSERT INTO items (content, embedding) VALUES ('hello', '[...]');`
- `SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 1;`  (ASC = giống nhất)
- **Node:** `require('pgvector/pg')` → `await pgvector.registerType(client)` → `pgvector.toSql(vec)`
- **Python:** `from pgvector.psycopg import register_vector` → `register_vector(conn)`
- **Nâng cấp:** `ALTER EXTENSION vector UPDATE;`
- **Migration:** `ALTER TABLE ... ADD COLUMN embedding vector(n)` (nullable) → backfill → `CREATE INDEX CONCURRENTLY ... USING hnsw ...`
- **Build source:** `apt install build-essential git postgresql-server-dev-17` → `git clone` → `make` → `sudo make install` → vẫn phải `CREATE EXTENSION`.

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào.

1. Vì sao cài pgvector là **HAI bước**, và "cài file ≠ bật extension" nghĩa là gì?
2. Vì sao managed provider bỏ được bước cài binary?
3. Vì sao nhét vector 384 chiều vào cột `vector(512)` lỗi ngay?
4. Vì sao `<=>` cho `ORDER BY ASC` (chứ không DESC) mới lấy câu giống nhất?
5. Vì sao `<#>` trả về **ÂM** inner product?
6. Vì sao ops class phải khớp toán tử query?
7. Vì sao Node.js cần `registerType` — không có thì chuyện gì xảy ra?
8. Vì sao binary build với `-march=native` crash trên máy CPU khác?
9. Vì sao "có index" không đảm bảo "index được dùng", và cách kiểm?
10. Vì sao bảng nhỏ chạy `Seq Scan` **KHÔNG phải** lỗi?
11. Vì sao build source trên production là **anti-pattern**?
12. Vì sao dùng `CREATE INDEX CONCURRENTLY` khi thêm index trên bảng lớn?
13. Vì sao `SET ef_search` toàn cục nguy hiểm với PgBouncer, và fix thế nào?
14. Vì sao **version skew** giữa primary/replica gây lỗi?
15. Vì sao nên **assert `EXPLAIN` có `Index Scan`** trong CI?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- Bài này **nặng cú pháp/quy trình** — chấp nhận học thuộc mục 9 (lệnh cài, 3 toán tử/ops class, cú pháp client, quy trình migration). Đây là bài "gõ đúng lệnh", không phải bài "hiểu cơ chế" (cơ chế nằm ở file 1).
- **Nhóm A ít nhưng đắt**: nắm 3 mental model "hai việc trông như một" (cài file ≠ bật extension / có index ≠ dùng index / registerType là cây cầu) + "-march=native không portable" + "least privilege & SET LOCAL". Đó là phần "vì sao" phân biệt người biết gõ lệnh với người hiểu mình đang làm gì.
- **Tái tạo**: dựng lại xương sống file 6 (cài → dùng → verify → migration → ops), rồi ghép vào **meta-xương sống 6 file**. File 6 = tầng *ops* của file 1 (tầng *concept*).
- **4 đính chính** phải ghi đè (PG 13+ / build source khó nhất / `<=>` là distance / registerType bắt buộc).
