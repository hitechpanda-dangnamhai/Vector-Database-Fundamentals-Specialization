# Cài & Dùng pgvector Thực Chiến: Install → Insert → Query → Node.js — Chain gối đầu

> Vị trí series (mảnh 6, bản THỰC HÀNH của pgvector khái niệm): bài khái niệm dạy *tại sao & cơ chế* (HNSW/IVFFlat, trade-off); bài này dạy *tay chân* — cài ra sao, gõ lệnh nào, kết nối app thế nào.
>
> **Đính chính bài gốc (đã cũ):** (1) **PostgreSQL 13+** (bài gốc nói 12 — nay đã bỏ; bản prebuilt khuyến nghị **15+**). (2) Build-from-source (git clone + make) là cách **KHÓ NHẤT** — ưu tiên **Docker / apt-yum / managed**. (3) `<=>` là cosine **distance** (`= 1 − cosine_similarity`), không phải similarity; nhỏ hơn = gần hơn → `ORDER BY ... ASC`. (4) Node.js: sau `require('pgvector/pg')` phải **đăng ký kiểu** (`registerType`) mới parse `vector` ↔ mảng JS đúng.

---

## CHAIN — Yêu cầu & hai bước cài

pgvector yêu cầu **PostgreSQL 13+** (bài gốc nói 12 đã bị bỏ; bản prebuilt khuyến nghị 15+), kiểm bằng `SELECT version()` => cài pgvector gồm HAI bước tách biệt: **cài binary** rồi mới **bật extension** => bước cài binary có nhiều cách; cách khó nhất là build-from-source (`git clone` + `make`) => bài gốc chỉ dạy cách khó này, nhưng 2026 có cách dễ hơn nhiều => dễ nhất: **Docker** `pgvector/pgvector:pg17`; hoặc gói OS `apt install postgresql-17-pgvector`; hoặc **managed** (Supabase/Neon/RDS đã cài sẵn) => nếu dùng managed thì bỏ luôn bước cài binary, chỉ còn bước bật extension

## CHAIN — Bật extension & verify

sau khi cài binary (bất kỳ cách nào), phải BẬT extension trong MỖI database muốn dùng => `CREATE EXTENSION IF NOT EXISTS vector` (idempotent, cần quyền CREATE hoặc superuser) => **cài file ≠ bật extension**: `make install` chỉ chép file, phải `CREATE EXTENSION` mới dùng được (người mới hay tưởng là một) => verify bằng `\dx vector` hoặc `SELECT extname, extversion FROM pg_extension WHERE extname='vector'` => thấy `vector | 0.8.x` là thành công

## CHAIN — Tạo bảng & dimension khớp model

tạo bảng có cột `vector(512)` — số 512 là số chiều model sinh embedding => độ dài vector PHẢI khớp output của model => nhét vector 384 chiều vào `vector(512)` → lỗi ngay (dimension mismatch) => model đổi → số chiều đổi → phải sửa cột và re-embed => nên chuẩn hóa chiều ở tầng app trước khi insert để tránh lỗi

## CHAIN — Insert, query & cosine distance

insert vector viết dạng chuỗi `'[a,b,c,...]'` (thực tế cắm mảng float model trả về, không gõ tay) => query 1 câu giống nhất: `ORDER BY embedding <=> '[...]' LIMIT 1` => đính chính bài gốc: `<=>` là cosine **distance**, không phải cosine **similarity** => `cosine_distance = 1 − cosine_similarity`, nên NHỎ hơn = gần hơn => vì nhỏ = gần, ta `ORDER BY ... ASC` (mặc định) để câu giống nhất lên đầu

## CHAIN — Ba toán tử distance & ops class

pgvector cung cấp ba toán tử: `<->` (L2/Euclidean), `<=>` (cosine distance), `<#>` (negative inner product) => `<#>` trả *âm* inner product để ASC vẫn = "gần nhất" (Postgres chỉ index ASC) => mỗi toán tử khớp một **ops class** khi đánh index: `<->`↔`vector_l2_ops`, `<=>`↔`vector_cosine_ops`, `<#>`↔`vector_ip_ops` => ops class phải khớp toán tử query, nếu không index bị bỏ => với text embeddings mặc định dùng cosine (`<=>`)

## CHAIN — Node.js registerType (bước hay quên)

kết nối Node.js: `npm install pg pgvector`, rồi `require('pgvector/pg')` => QUAN TRỌNG: sau require phải `registerType(client)` — bước bài gốc bỏ quên => không đăng ký kiểu thì driver `pg` trả vector dưới dạng CHUỖI thô, phải tự parse => đăng ký xong: `pgvector.toSql(emb)` chuyển mảng JS → literal vector khi insert/query => bên Python tương đương là `register_vector(conn)` (tự chuyển numpy/list)

## CHAIN — Ba lỗi cài đặt kinh điển

lỗi 1 — `type "vector" does not exist` sau `CREATE EXTENSION`: extension nằm ở schema khác `search_path` → sửa search_path hoặc `CREATE EXTENSION vector SCHEMA public` => lỗi 2 — cài file nhưng quên `CREATE EXTENSION`: `make install` chỉ chép file, phải `CREATE EXTENSION` trong *mỗi* database => lỗi 3 — sai lib path khi có nhiều bản Postgres: build source với header PG 15 nhưng server chạy PG 17 → `.so` sai chỗ, `CREATE EXTENSION` lỗi => cả ba là lý do nên dùng Docker/gói thay vì build tay

## CHAIN — Build dưới nắp capo & -march=native

build-from-source dùng **PGXS**: `make` gọi `pg_config` tìm header/thư viện bản Postgres đang cài, biên dịch C thành `vector.so` (shared library) + file `.sql`/`.control` => `make install` chép `.so` → thư mục `lib/`, `.control` + `--*.sql` → thư mục `extension/`, từ đó `CREATE EXTENSION` mới thấy => build tay mặc định tối ưu cho CPU máy build (`-march=native`) → nhanh hơn nhưng KHÔNG chạy được trên CPU khác (lỗi illegal instruction) => đó là lý do "binary tôi build ở máy A crash ở máy B" => Docker image chính thức build với `OPTFLAGS=""` (tắt `-march=native`) → chạy mọi CPU, đánh đổi chút tốc độ lấy tính di động

## CHAIN — EXPLAIN ANALYZE: có index ≠ dùng index

*có* index không có nghĩa index *được dùng* → luôn `EXPLAIN ANALYZE` => thấy `Index Scan using ..._hnsw...` → tốt, index đang chạy => thấy `Seq Scan` → planner bỏ index => nguyên nhân Seq Scan: bảng nhỏ (seq scan rẻ hơn), ops class không khớp toán tử, hoặc cost estimation => bảng nhỏ seq scan KHÔNG phải lỗi — exact scan rẻ và cho recall 100% khi n nhỏ

## CHAIN — Upgrade & version skew

nâng cấp: sau khi cài bản mới (cùng cách cài ban đầu), chạy `ALTER EXTENSION vector UPDATE` trong mỗi database => đọc CHANGELOG trước khi nâng trên production (một số bản yêu cầu re-create IVFFlat index) => nên nâng trong cửa sổ bảo trì, test trên staging trước => giữ pgvector đồng bộ trên primary và mọi replica => version skew: replica khác version có thể lỗi khi áp WAL liên quan kiểu vector

## CHAIN — Đừng build source trên production

build tay trên server production là **anti-pattern** => vì kết quả phụ thuộc CPU/compiler máy đó (`-march=native`), khó tái lập, khó rollback, dễ lệch giữa các node => thay bằng Docker image cố định tag (`pgvector/pgvector:pg17`) → mọi môi trường dev/staging/prod giống hệt => hoặc gói OS pin version (`postgresql-17-pgvector=0.8.x`) qua Ansible/Terraform => hoặc managed provider lo binary => nguyên tắc: pin phiên bản pgvector + Postgres rõ ràng, coi như dependency có version, không phải "build lúc deploy"

## CHAIN — Migration prod & zero-downtime

`CREATE EXTENSION` cần quyền cao → đưa vào migration script chạy MỘT lần bởi admin, không rải rác trong app => thêm cột: `ALTER TABLE ADD COLUMN embedding vector(n)` nullable (nhanh) => backfill embedding theo lô ở nền (queue), không khóa => đánh index không khóa: `CREATE INDEX CONCURRENTLY ... USING hnsw ...` sau khi backfill xong (build HNSW có thể hàng giờ) => verify `\dx vector` + `EXPLAIN ANALYZE` (chắc `Index Scan`) => vì mọi thứ là migration + dependency pinned nên rollback rõ ràng (drop index/column, gỡ extension)

## CHAIN — Security & CI/CD

security: app user chỉ cần INSERT/SELECT trên bảng, KHÔNG cần superuser => bật extension bằng migration chạy bởi admin, không bằng app user => connection pooler (PgBouncer): `SET` toàn cục (như `hnsw.ef_search`) rò qua connection tái dùng → dùng `SET LOCAL` trong transaction => CI/CD: dịch vụ Postgres trong CI dùng chính image `pgvector/pgvector:pgXX` để test đúng môi trường như prod => và test cả đường "index được dùng" bằng cách assert `EXPLAIN` có `Index Scan` (tránh regression âm thầm về seq scan)

## CHAIN — Tổ chức & khép series

nói với PM/sếp: thêm vector search KHÔNG cần dựng hệ thống mới — chỉ bật một extension trên database sẵn có => framing bằng **rủi ro thấp + quy trình chuẩn**: Docker/managed thì cài đặt tái lập được, nâng cấp/rollback rõ ràng như mọi dependency => cái cần đầu tư là quy trình (migration, đánh index không khóa, pin version), không phải mua phần mềm mới => chuẩn hóa: nhiều team dùng Postgres thì thống nhất một cách cài (Docker tag/gói pinned) giảm "chạy trên máy tôi" => coi pgvector là dependency có vòng đời: theo dõi CHANGELOG, lên lịch nâng cấp, test staging => khép series 6 mảnh: embed → index → **cài & dùng pgvector (bài này)** → store/query khái niệm → keyword/FTS → chọn nền tảng

---

## BẢNG — Mọi cách cài, chọn khi nào

| Cách | Lệnh cốt lõi | Dùng khi |
|---|---|---|
| **Docker** | `docker run pgvector/pgvector:pg17` | học, dev, CI — nhanh, sạch, reproducible |
| **apt/yum** | `apt install postgresql-17-pgvector` | production trên VM/bare-metal có Postgres sẵn |
| **Managed** | (có sẵn) → `CREATE EXTENSION vector` | Supabase/Neon/RDS — khỏi vận hành |
| **Homebrew/conda/PGXN/pkg/APK** | tùy nền tảng | môi trường dev đặc thù |
| **Build-from-source** | `git clone && make && make install` | phiên bản chưa có gói, hoặc cần custom |

Build source: `sudo apt install build-essential git postgresql-server-dev-17` → `git clone` → `make` → `sudo make install` → rồi vẫn phải `CREATE EXTENSION`.

## BẢNG — Ba toán tử distance

| Toán tử | Metric | Ghi chú | Ops class index |
|---|---|---|---|
| `<->` | Euclidean (L2) | khoảng cách thẳng | `vector_l2_ops` |
| `<=>` | Cosine distance | phổ biến nhất với text | `vector_cosine_ops` |
| `<#>` | Negative inner product | pgvector trả *âm* IP (để ASC = gần) | `vector_ip_ops` |

## BẢNG — Code thuộc lòng

**(a) Cài + bật + tạo bảng + index:**
```bash
docker run -d --name pgv -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg17
docker exec -it pgv psql -U postgres
```
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id serial PRIMARY KEY, content text, embedding vector(512));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
\dx vector          -- verify
```

**(b) Insert + query:**
```sql
INSERT INTO items (content, embedding) VALUES ('hello', '[...]');
SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 1;   -- ASC = giống nhất
```

**(c) Node.js (đăng ký kiểu — bước hay bị quên):**
```javascript
const { Client } = require('pg');
const pgvector = require('pgvector/pg');
const client = new Client({ connectionString: '...' });
await client.connect();
await pgvector.registerType(client);        // KHÔNG có -> vector trả về dạng chuỗi
await client.query('INSERT INTO items (embedding) VALUES ($1)', [pgvector.toSql(vec)]);
const res = await client.query('SELECT content FROM items ORDER BY embedding <=> $1 LIMIT 1',
                               [pgvector.toSql(queryVec)]);
```

**(d) Python:**
```python
import psycopg
from pgvector.psycopg import register_vector
conn = psycopg.connect("...")
register_vector(conn)                       # tương đương registerType
conn.execute("INSERT INTO items (content, embedding) VALUES (%s, %s)", ("hi", emb))
row = conn.execute("SELECT content FROM items ORDER BY embedding <=> %s LIMIT 1",
                   (query_vec,)).fetchone()
```

**(e) Nâng cấp:**
```sql
ALTER EXTENSION vector UPDATE;
SELECT extversion FROM pg_extension WHERE extname='vector';
```

## BẢNG — Khung ops/system design (thêm pgvector vào Postgres prod, không downtime)

1. **Clarify:** self-host hay managed? phiên bản Postgres? có replica? kích thước bảng sẽ đánh index?
2. **Cài binary không gián đoạn:** managed → console/`CREATE EXTENSION`; self-host → gói `postgresql-XX-pgvector` (không cần restart); **không** build tay trên prod.
3. **Bật extension:** `CREATE EXTENSION IF NOT EXISTS vector` qua migration chạy bởi admin (idempotent).
4. **Thêm cột & backfill:** `ALTER TABLE ADD COLUMN embedding vector(n)` nullable (nhanh); backfill theo lô ở nền (queue), không khóa.
5. **Đánh index không khóa:** `CREATE INDEX CONCURRENTLY ... USING hnsw ...` sau backfill; canh `maintenance_work_mem`.
6. **Verify:** `\dx vector`, `EXPLAIN ANALYZE` để chắc `Index Scan`.
7. **Replica & version:** mọi replica cùng version pgvector.
8. **Rollback plan:** mọi thứ là migration + dependency pinned → rollback rõ ràng (drop index/column, gỡ extension). Có kế hoạch lui = tư duy staff.

## BẢNG — Mental models

- **"Cài file ≠ bật extension"** → hai bước, đừng gộp.
- **"Có index ≠ dùng index"** → luôn `EXPLAIN ANALYZE`.
- **"Build tay = crash lạ ở máy khác"** → dùng Docker/gói cho di động.
- **"Pin version như mọi dependency"** → pgvector không phải thứ build lúc deploy.
- **"registerType là cây cầu"** → không có nó, app nhận vector dạng chuỗi.

---

## TỪ KHÓA MỒI

- Yêu cầu & hai bước cài → **"PostgreSQL 13+"**
- Bật extension & verify → **"cài file ≠ bật extension"**
- Tạo bảng & dimension → **"khớp số chiều model"**
- Insert, query & cosine distance → **"<=> ASC = gần nhất"**
- Ba toán tử distance → **"ops class khớp toán tử"**
- Node.js registerType → **"cây cầu vector ↔ mảng"**
- Ba lỗi cài đặt → **"type vector does not exist"**
- Build dưới nắp capo → **"-march=native"**
- EXPLAIN ANALYZE → **"có index ≠ dùng index"**
- Upgrade & version skew → **"ALTER EXTENSION UPDATE"**
- Đừng build source trên prod → **"pin version"**
- Migration prod → **"CONCURRENTLY, backfill nullable"**
- Security & CI → **"SET LOCAL, image pgvector trong CI"**
- Tổ chức & series → **"rủi ro thấp + quy trình chuẩn"**

---

## ĐÃ PHỦ

**Cài đặt:** yêu cầu PostgreSQL 13+ (prebuilt 15+), `SELECT version()`; hai bước cài binary + bật extension; các cách (Docker `pgvector/pgvector:pg17`, apt `postgresql-17-pgvector` / dnf `pgvector_17`, managed, Homebrew/conda/PGXN, build-from-source); bảng "chọn cách nào khi nào"; build source cần build-essential + postgresql-server-dev-XX.

**Bật & verify:** `CREATE EXTENSION IF NOT EXISTS vector` per-database idempotent cần quyền; cài file ≠ bật extension; `\dx vector` / `pg_extension`; thấy `vector | 0.8.x`.

**Dùng cơ bản:** cột `vector(512)` khớp model, nhét sai chiều → lỗi, đổi model → sửa cột + re-embed; insert `'[...]'`; query `ORDER BY <=> LIMIT`; đính chính `<=>` = cosine distance (1−sim, ASC = gần).

**Ba toán tử:** `<->` L2 / `<=>` cosine / `<#>` negative inner product (âm để ASC=gần, Postgres chỉ index ASC); ops class tương ứng vector_l2_ops/cosine_ops/ip_ops; text mặc định cosine.

**Client:** Node.js `npm install pg pgvector`, `require('pgvector/pg')`, `registerType(client)` (không có → vector là chuỗi), `toSql(emb)`; Python `psycopg[binary]` + `register_vector(conn)` tự chuyển numpy/list.

**3 lỗi cài đặt:** `type "vector" does not exist` (schema/search_path), quên `CREATE EXTENSION` sau make install, sai `.so` lib path (header PG15 vs server PG17).

**Advanced:** PGXS (`make`→`pg_config`→`vector.so`+`.control`/`.sql`; `make install` chép `.so`→lib/, control+sql→extension/); `-march=native` không portable → Docker `OPTFLAGS=""`; `EXPLAIN ANALYZE` có index ≠ dùng index (`Index Scan` vs `Seq Scan`, nguyên nhân bảng nhỏ/ops class/cost); generated column/schema với tenant_id + 2 index; `ALTER EXTENSION vector UPDATE` + đọc CHANGELOG (re-create IVFFlat); edge cases (dimension mismatch, `<#>` negative IP, NULL embedding không xuất hiện, bảng nhỏ seq scan recall 100%, nhiều bản Postgres `.so` khớp).

**Staff/ops:** đừng build source trên prod (anti-pattern CPU/compiler, khó tái lập/rollback) → Docker tag / gói pin version (Ansible/Terraform) / managed; migration prod (CREATE EXTENSION quyền cao qua migration 1 lần, CREATE INDEX CONCURRENTLY build HNSW hàng giờ, ALTER EXTENSION UPDATE cửa sổ bảo trì, version skew primary/replica đồng bộ WAL); security (app user chỉ INSERT/SELECT, admin bật extension, schema/search_path, PgBouncer + SET LOCAL); CI/CD (image pgvector trong CI, migration idempotent, assert EXPLAIN Index Scan); tổ chức (framing rủi ro thấp + quy trình chuẩn, chuẩn hóa một cách cài, dependency có vòng đời); khung zero-downtime 8 bước.

**Đính chính bài gốc:** PostgreSQL 13+ (không 12); `<=>` = cosine distance; build source là cách khó nhất; đăng ký kiểu ở client.

**Học tiếp (bài nêu):** kết nối framework (LangChain/LlamaIndex), pipeline embedding bất đồng bộ, triển khai RAG app end-to-end.
