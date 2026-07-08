# Cài & Dùng pgvector Thực Chiến: Install → Insert → Query → Node.js — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"pgvector extension for PostgreSQL"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy hands-on: yêu cầu cài đặt, cài trên Linux (git clone + make), verify, tạo bảng có cột vector, insert, query bằng `<=>`, và dùng pgvector với Node.js. Tôi giảng bám sát rồi đào sâu thành hướng dẫn vận hành thật. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là **bản thực hành** của giáo trình pgvector khái niệm (bài #3). Bài #3 dạy *tại sao & cơ chế* (HNSW/IVFFlat, trade-off); bài này dạy *tay chân*: cài ra sao, gõ lệnh nào, kết nối app thế nào. Cơ chế index tôi chỉ nhắc lại ngắn và trỏ về #3.
>
> **⚠️ Đính chính & cập nhật tới 2026 (bài gốc có chỗ đã cũ/thiếu chính xác):**
> 1. **PostgreSQL 12 KHÔNG còn đủ.** pgvector mới đã bỏ hỗ trợ PG cũ; hiện cần **PostgreSQL 13+** (bản prebuilt khuyến nghị **15+**). Đừng theo con số "12" trong bài gốc.
> 2. **Build-from-source (git clone + make) là cách KHÓ NHẤT** — bài gốc chỉ dạy cách này. 2026 có cách dễ hơn nhiều: **Docker** (`pgvector/pgvector:pg17`), **apt/yum** (`apt install postgresql-17-pgvector`), hoặc **managed** (Supabase/Neon/RDS đã cài sẵn, chỉ cần `CREATE EXTENSION`). Nên ưu tiên các cách này.
> 3. **`<=>` là cosine *distance*, không phải cosine *similarity*.** Bài gốc gọi nó "cosine similarity" là chưa chuẩn. `cosine_distance = 1 − cosine_similarity`; nhỏ hơn = giống hơn, nên `ORDER BY ... <=> ... ASC` cho kết quả giống nhất lên đầu.
> 4. **Node.js:** sau `require('pgvector/pg')` phải **đăng ký kiểu** với client thì mới parse `vector` ↔ mảng JS đúng.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Toàn bộ vòng đời thực hành của pgvector — từ *cài extension vào PostgreSQL*, *verify nó chạy*, *tạo bảng có cột `vector`*, *insert embedding*, *query k gần nhất bằng toán tử distance*, tới *kết nối từ app Node.js/Python*. Kết thúc là bạn có một database vector chạy được thật, không chỉ hiểu lý thuyết.

**Vấn đề nó giải quyết:** Hiểu HNSW/cosine là một chuyện; *cài được pgvector và query từ code* là chuyện khác — và đây là chỗ người mới hay tắc (build lỗi, `type "vector" does not exist`, index không được dùng, app không parse được vector). Bài này gỡ từng nút.

**Học xong bạn sẽ làm được:**

- Nêu đúng yêu cầu cài đặt (PostgreSQL 13+) và chọn cách cài phù hợp (Docker/apt/source/managed).
- Cài, verify, và bật extension `vector` trong một database.
- Tạo bảng có cột `vector(n)` khớp số chiều model, insert & query bằng `<=>`/`<->`/`<#>`.
- Kết nối pgvector từ Node.js và Python, đăng ký kiểu đúng.
- Xử lý gotcha cài đặt phổ biến; vận hành ở production (Docker, CREATE INDEX CONCURRENTLY, version management).

**Mạch basic → staff:**

- 🟢 **Basic:** Yêu cầu (PG 13+) → cách cài DỄ NHẤT (Docker/managed) → `CREATE EXTENSION` → verify → bảng + insert + query đầu tiên.
- 🟡 **Intermediate:** Mọi cách cài so sánh; verify sâu; toán tử distance; client Node.js + Python; gotcha cài đặt.
- 🔴 **Advanced:** Build-from-source dưới nắp capo (PGXS, `.so`, `-march=native`); schema/search_path; generated column; `EXPLAIN ANALYZE` để chắc index được dùng; upgrade extension.
- 🟣 **Staff:** Cài đặt & vận hành production (Docker trong prod, migration, CI, security, reproducibility); vì sao *không* build source trên prod; câu hỏi ops/system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Yêu cầu cài đặt

- **PostgreSQL 13 trở lên** (bài gốc nói 12 — nay đã bỏ; bản prebuilt khuyến nghị **15+**). Kiểm tra:
  ```sql
  SELECT version();
  ```
- **pgvector** lấy từ repo chính thức: `https://github.com/pgvector/pgvector`.
- Nếu build từ source: cần trình biên dịch C (`build-essential`) và header của Postgres (`postgresql-server-dev-XX`). Nhưng **bạn thường không cần build tay** — xem 1.2.

### 1.2. Cách cài DỄ NHẤT (khác bài gốc — nên dùng cái này trước)

Bài gốc dạy build-from-source (khó, dễ lỗi). Với người mới, ba cách dưới đây gọn hơn nhiều:

**(a) Docker — nhanh nhất để học/thử (khuyến nghị):**
```bash
# Image chính thức đã cài sẵn pgvector, chọn phiên bản Postgres qua tag
docker run -d --name pgv \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  pgvector/pgvector:pg17
# Vào psql:
docker exec -it pgv psql -U postgres
```

**(b) Gói hệ điều hành (production trên máy thật):**
```bash
# Debian/Ubuntu (thay 17 bằng phiên bản Postgres của bạn)
sudo apt install postgresql-17-pgvector
# RHEL/Rocky/Fedora
sudo dnf install pgvector_17
```

**(c) Managed — không cài gì cả:** Supabase, Neon, AWS RDS/Aurora đã có sẵn pgvector; bạn chỉ cần bước `CREATE EXTENSION` ở 1.3.

### 1.3. Bật extension trong database + verify

Sau khi cài (bằng bất kỳ cách nào), *bật* extension trong **mỗi database** muốn dùng:

```sql
-- Bật (idempotent: chạy lại không lỗi). Cần quyền CREATE hoặc superuser.
CREATE EXTENSION IF NOT EXISTS vector;

-- Verify: extension đã có chưa & version bao nhiêu
\dx vector
-- hoặc:
SELECT extname, extversion FROM pg_extension WHERE extname = 'vector';
```

Thấy `vector | 0.8.x` là thành công.

### 1.4. Tạo bảng có cột vector — số chiều PHẢI khớp model

```sql
CREATE TABLE items (
    id        serial PRIMARY KEY,
    content   text,
    embedding vector(512)     -- 512 = số chiều model sinh embedding của bạn
);
```

> Bài gốc nói đúng điểm cốt: **độ dài vector (512) phải khớp output của model**. Model đổi → số chiều đổi → phải sửa cột (và re-embed). Nhét vector 384 chiều vào `vector(512)` → lỗi ngay.

### 1.5. Insert & query đầu tiên

```sql
-- Insert: vector viết dạng chuỗi '[a,b,c,...]'
INSERT INTO items (content, embedding) VALUES
  ('the capital of the UK is London',        '[...512 số...]'),
  ('the German railways are Deutsche Bahn',  '[...512 số...]');

-- Query: tìm 1 câu GIỐNG NHẤT với embedding của câu tìm kiếm
SELECT content
FROM items
ORDER BY embedding <=> '[...embedding câu tìm kiếm...]'   -- <=> = cosine distance
LIMIT 1;
```

> **[Đính chính bài gốc]** Bài gốc gọi `<=>` là "cosine similarity". Chính xác: `<=>` trả về **cosine distance** (`= 1 − cosine_similarity`). Vì *nhỏ hơn = gần hơn*, ta `ORDER BY ... ASC` (mặc định) để câu giống nhất lên đầu. Kết quả sẽ là "the capital of the UK is London" nếu bạn tìm câu về thủ đô nước Anh.

Trong thực tế embedding do model sinh (xem giáo trình embeddings), không gõ tay. Ở đây `'[...]'` là chỗ bạn cắm mảng float model trả về.

### ✅ Self-check Phần 1

1. Phiên bản PostgreSQL tối thiểu cho pgvector hiện nay? (Không phải con số trong bài gốc.)
2. Nêu 2 cách cài pgvector dễ hơn build-from-source.
3. `<=>` trả về similarity hay distance? Vậy `ORDER BY` để giống-nhất-lên-đầu dùng ASC hay DESC?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. Mọi cách cài — chọn cái nào khi nào

| Cách | Lệnh cốt lõi | Dùng khi |
|---|---|---|
| **Docker** | `docker run pgvector/pgvector:pg17` | học, dev, CI — nhanh, sạch, reproducible |
| **apt/yum** | `apt install postgresql-17-pgvector` | production trên VM/bare-metal có Postgres sẵn |
| **Managed** | (có sẵn) → `CREATE EXTENSION vector` | Supabase/Neon/RDS — khỏi vận hành |
| **Homebrew/conda/PGXN/pkg/APK** | tùy nền tảng | môi trường dev đặc thù |
| **Build-from-source** | `git clone && make && make install` | phiên bản chưa có gói, hoặc cần custom |

### 2.2. Build-from-source — cách bài gốc dạy (khi bắt buộc)

Chỉ dùng khi không có gói dựng sẵn cho môi trường của bạn:

```bash
# Cần header Postgres + trình biên dịch trước
sudo apt install build-essential git postgresql-server-dev-17

git clone https://github.com/pgvector/pgvector.git   # tải mã nguồn
cd pgvector                                           # vào thư mục
make                                                  # biên dịch
sudo make install                                     # cài file .so + .sql vào Postgres
# Rồi vào psql chạy: CREATE EXTENSION IF NOT EXISTS vector;
```

`make && make install` biên dịch extension rồi chép file vào thư mục Postgres. **Sau đó vẫn phải `CREATE EXTENSION`** trong database — cài file ≠ bật extension (hai bước khác nhau, người mới hay tưởng là một).

### 2.3. Ba toán tử distance — dùng đúng cái

pgvector cung cấp ba toán tử, khớp với ba metric (và ba ops class khi đánh index):

| Toán tử | Metric | Ghi chú | Ops class index |
|---|---|---|---|
| `<->` | Euclidean (L2) | khoảng cách thẳng | `vector_l2_ops` |
| `<=>` | Cosine distance | phổ biến nhất với text | `vector_cosine_ops` |
| `<#>` | Negative inner product | pgvector trả *âm* IP (để ASC = gần) | `vector_ip_ops` |

```sql
-- Ba kiểu query tương ứng:
SELECT content FROM items ORDER BY embedding <-> '[...]' LIMIT 5;  -- L2
SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 5;  -- cosine
SELECT content FROM items ORDER BY embedding <#> '[...]' LIMIT 5;  -- inner product
```

Với text embeddings, mặc định dùng **cosine** (`<=>`). (Chi tiết metric: giáo trình embeddings.)

### 2.4. Dùng pgvector với Node.js (bám bài gốc + sửa cho đúng)

```bash
npm install pg pgvector
```

```javascript
const { Client } = require('pg');
const pgvector = require('pgvector/pg');   // helper cho kiểu vector

const client = new Client({ connectionString: 'postgresql://user:pass@localhost/db' });
await client.connect();

// QUAN TRỌNG: đăng ký kiểu vector với client thì mảng JS <-> vector mới parse đúng
await pgvector.registerType(client);       // (tên hàm theo README của package)

// Insert: dùng pgvector.toSql() để chuyển mảng JS -> literal vector
const emb = [/* ...512 số... */];
await client.query(
  'INSERT INTO items (content, embedding) VALUES ($1, $2)',
  ['xin chào', pgvector.toSql(emb)]
);

// Query k gần nhất
const q = pgvector.toSql([/* ...embedding câu tìm... */]);
const res = await client.query(
  'SELECT content FROM items ORDER BY embedding <=> $1 LIMIT 1',
  [q]
);
console.log(res.rows[0].content);
await client.end();
```

> Bài gốc dừng ở `npm install pgvector` + `require('pgvector/pg')`. Bước bị thiếu là **đăng ký kiểu** (`registerType`) — không có nó, driver `pg` trả về vector dưới dạng chuỗi thô, bạn phải tự parse. Đăng ký xong thì làm việc với mảng số tự nhiên.

### 2.5. Dùng pgvector với Python (thêm — chuẩn công nghiệp)

```bash
pip install "psycopg[binary]" pgvector
```

```python
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("postgresql://user:pass@localhost/db")
register_vector(conn)                      # tương đương registerType bên Node

conn.execute("INSERT INTO items (content, embedding) VALUES (%s, %s)",
             ("xin chào", emb))            # emb là numpy array / list, tự chuyển
row = conn.execute(
    "SELECT content FROM items ORDER BY embedding <=> %s LIMIT 1", (query_vec,)
).fetchone()
print(row[0])
```

### 2.6. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi cài đặt kinh điển

**Lỗi 1 — `type "vector" does not exist` sau khi `CREATE EXTENSION`.** Extension nằm ở schema khác `search_path`. Kiểm & sửa:
```sql
\dx vector                                   -- xem extension ở schema nào
SET search_path TO public, that_schema;      -- hoặc cài vào public từ đầu:
-- DROP EXTENSION vector; CREATE EXTENSION vector SCHEMA public;
```

**Lỗi 2 — Cài file nhưng quên `CREATE EXTENSION`.** `make install` chỉ chép file; phải chạy `CREATE EXTENSION` trong *mỗi* database. Hai bước riêng biệt.

**Lỗi 3 — Sai lib path khi có nhiều bản Postgres.** Build source với header của PG 15 nhưng server chạy PG 17 → `.so` sai chỗ, `CREATE EXTENSION` lỗi. Kiểm:
```bash
ls -l /usr/lib/postgresql/17/lib/vector.so   # Ubuntu
```
Đây là lý do nữa để dùng Docker/gói thay vì build tay.

### ✅ Self-check Phần 2

1. Sau `make install`, đã dùng được vector chưa? Còn thiếu bước gì?
2. Ba toán tử `<->`, `<=>`, `<#>` ứng với metric nào?
3. Trong Node.js, vì sao phải `registerType(client)` trước khi query vector?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Build-from-source dưới nắp capo

- **PGXS:** hệ thống build extension của Postgres. `make` gọi `pg_config` để tìm header/thư viện của bản Postgres đang cài, biên dịch C thành `vector.so` (shared library) + các file `.sql`/`.control`.
- **`make install`** chép: `vector.so` → thư mục `lib/`; `vector.control` + `vector--*.sql` → thư mục `extension/`. Từ đó `CREATE EXTENSION` mới thấy.
- **`-march=native` vs portable:** build tay mặc định tối ưu cho CPU máy build (`-march=native`) → nhanh hơn nhưng **không chạy được trên CPU khác** (lỗi illegal instruction khi bê sang máy khác). Image Docker chính thức build với `OPTFLAGS=""` (tắt `-march=native`) để chạy mọi CPU — đánh đổi một chút tốc độ lấy tính di động. Biết điều này giải thích vì sao "binary tôi build ở máy A crash ở máy B".

### 3.2. Verify index THỰC SỰ được dùng — `EXPLAIN ANALYZE`

Cài xong, tạo bảng + index (cơ chế HNSW/IVFFlat: xem giáo trình #3 và giáo trình indexing). Nhưng *có index* không có nghĩa *index được dùng*:

```sql
EXPLAIN ANALYZE
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

- Thấy **`Index Scan using ..._hnsw...`** → tốt, index đang chạy.
- Thấy **`Seq Scan`** → planner bỏ index. Nguyên nhân: bảng nhỏ (seq scan rẻ hơn), ops class không khớp toán tử, hoặc `enable_seqscan` khác. Đọc kỹ output trước khi kết luận sai cấu hình.

### 3.3. Generated column — tự động hóa embedding-adjacent (mẫu hay dùng)

Nếu vector sinh từ nguồn cố định, cột thường + index đủ. Nhưng khi ghép metadata + đảm bảo tính nhất quán, người ta hay dùng bảng gọn kèm ràng buộc chiều:

```sql
CREATE TABLE docs (
    id        bigserial PRIMARY KEY,
    tenant_id bigint NOT NULL,
    content   text,
    embedding vector(1536)
);
CREATE INDEX ON docs (tenant_id);                              -- filter nhanh
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops); -- vector search
```

(pgvector không tự sinh embedding — bạn insert vector từ model. Nếu muốn DB tự sinh, đó là HeatWave; xem giáo trình so sánh nền tảng.)

### 3.4. Nâng cấp pgvector đúng cách

```sql
-- Sau khi cài bản mới (cùng cách cài ban đầu), trong mỗi database:
ALTER EXTENSION vector UPDATE;
-- Kiểm version mới:
SELECT extversion FROM pg_extension WHERE extname='vector';
```

Lưu ý: một số bản có ghi chú upgrade (vd re-create IVFFlat index sau khi lên từ bản rất cũ). Đọc CHANGELOG trước khi nâng trên production.

### 3.5. Edge cases

- **Dimension mismatch khi insert:** vector khác `vector(n)` của cột → lỗi. Chuẩn hóa chiều ở tầng app trước khi insert.
- **`<#>` là *negative* inner product:** đừng ngạc nhiên khi thấy giá trị âm — pgvector dùng âm để ASC vẫn = "gần nhất" (Postgres chỉ index ASC).
- **`NULL` embedding:** hàng có `embedding IS NULL` không xuất hiện trong ANN result → job kiểm & backfill.
- **Bảng nhỏ dùng seq scan:** không phải lỗi — exact scan rẻ và cho recall 100% khi n nhỏ.
- **Nhiều bản Postgres trên một host:** `.so` phải khớp bản đang chạy (mục 2.6 lỗi 3).

### ✅ Self-check Phần 3

1. Vì sao binary pgvector build tay ở máy này có thể crash ở máy khác? Docker giải quyết ra sao?
2. Bạn thấy `Seq Scan` trong `EXPLAIN ANALYZE` dù đã tạo HNSW index — nêu 2 nguyên nhân khả dĩ.
3. Lệnh nâng cấp extension trong một database là gì?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Đừng build-from-source trên production — reproducibility

Bài gốc dạy git clone + make. Ở tầm staff, **build tay trên server production là anti-pattern**: kết quả phụ thuộc CPU/compiler máy đó (`-march=native`), khó tái lập, khó rollback, dễ lệch giữa các node. Thay vào đó:

- **Docker image cố định tag** (`pgvector/pgvector:pg17`) → mọi môi trường (dev/staging/prod) giống hệt.
- **Gói OS pin version** (`postgresql-17-pgvector=0.8.x`) qua công cụ quản lý cấu hình (Ansible/Terraform).
- **Managed provider** → nhà cung cấp lo binary.

Nguyên tắc: *pin phiên bản pgvector + Postgres rõ ràng*, coi nó như dependency có version, không phải "build lúc deploy".

### 4.2. Cài đặt & migration ở production

- **`CREATE EXTENSION` cần quyền cao** (CREATE trên database hoặc superuser) → đưa vào migration script chạy một lần, không rải rác trong code app.
- **Đánh index trên bảng lớn:** dùng **`CREATE INDEX CONCURRENTLY`** để không khóa write suốt quá trình build (build HNSW có thể hàng giờ).
- **Nâng cấp:** `ALTER EXTENSION vector UPDATE` trong cửa sổ bảo trì; test trên staging trước; đọc CHANGELOG cho ghi chú re-index.
- **Version skew:** giữ pgvector đồng bộ trên primary và mọi replica (khác version có thể gây lỗi khi replica áp WAL liên quan kiểu vector).

### 4.3. Security & multi-tenancy

- **Privilege tối thiểu:** app user chỉ cần INSERT/SELECT trên bảng, không cần superuser. Bật extension bằng migration chạy bởi admin, không bằng app user.
- **Schema/search_path** rõ ràng (mục 2.6 lỗi 1) để tránh "type vector does not exist" trong môi trường nhiều schema/tenant.
- **Connection pooler (PgBouncer):** cẩn thận `SET` toàn cục (như `hnsw.ef_search`) rò qua connection tái dùng → dùng `SET LOCAL` trong transaction (nối lại giáo trình pgvector #3).

### 4.4. CI/CD & test

- Dịch vụ Postgres trong CI dùng chính image `pgvector/pgvector:pgXX` → test vector chạy đúng môi trường như prod.
- Migration idempotent: `CREATE EXTENSION IF NOT EXISTS vector;` an toàn chạy lại.
- Test cả đường "index được dùng" bằng cách assert `EXPLAIN` có `Index Scan` (tránh regression âm thầm về seq scan).

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Thêm vector search *không cần dựng hệ thống mới* — chỉ bật một extension trên database sẵn có. Rủi ro thấp: nếu dùng Docker/managed thì cài đặt tái lập được, nâng cấp/rollback rõ ràng như mọi dependency. Cái cần đầu tư là quy trình (migration, đánh index không khóa, pin version), không phải mua phần mềm mới." → framing bằng **rủi ro thấp + quy trình chuẩn**.
- **Chuẩn hóa:** nếu nhiều team dùng Postgres, thống nhất một cách cài (Docker tag / gói pinned) giảm "hoạt động trên máy tôi" và lệch môi trường.
- **Vận hành:** coi pgvector là dependency có vòng đời — theo dõi CHANGELOG, lên lịch nâng cấp, test trên staging.

### 4.6. Câu hỏi ops/system-design mẫu + hướng trả lời staff

> **"Team muốn thêm pgvector vào một PostgreSQL production đang phục vụ khách hàng, không được downtime. Bạn triển khai thế nào?"**

**Khung trả lời staff:**

1. **Clarify:** self-host hay managed? phiên bản Postgres? có replica không? kích thước bảng sẽ đánh index?
2. **Cài binary không gián đoạn:** managed → bật qua console/`CREATE EXTENSION`. Self-host → cài gói `postgresql-XX-pgvector` (không cần restart để có file); **không** build tay trên prod.
3. **Bật extension:** `CREATE EXTENSION IF NOT EXISTS vector;` qua migration chạy bởi admin (idempotent, nhanh).
4. **Thêm cột & backfill:** `ALTER TABLE ADD COLUMN embedding vector(n)` (nullable, nhanh); backfill embedding theo lô ở nền (queue), không khóa.
5. **Đánh index không khóa:** `CREATE INDEX CONCURRENTLY ... USING hnsw ...` sau khi backfill xong; canh `maintenance_work_mem`.
6. **Verify:** `\dx vector`, `EXPLAIN ANALYZE` để chắc `Index Scan`.
7. **Replica & version:** đảm bảo mọi replica cùng version pgvector.
8. **Rollback plan:** vì mọi thứ là migration + dependency pinned, rollback rõ ràng (drop index/column, gỡ extension nếu cần). **Có kế hoạch lui = tư duy staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **pgvector** — extension thêm kiểu `vector` + vector search cho PostgreSQL.
- **`CREATE EXTENSION vector`** — bật extension trong một database (sau khi cài binary).
- **`vector(n)`** — kiểu cột lưu embedding n chiều; n khớp model.
- **`<->` / `<=>` / `<#>`** — L2 / cosine distance / negative inner product.
- **ops class** — `vector_l2_ops` / `vector_cosine_ops` / `vector_ip_ops` (khớp toán tử khi index).
- **PGXS** — hệ build extension của Postgres (dùng khi build source).
- **`make && make install`** — biên dịch + cài file extension (chưa bật).
- **`\dx` / `pg_extension`** — verify extension & version.
- **`ALTER EXTENSION vector UPDATE`** — nâng cấp extension.
- **`CREATE INDEX CONCURRENTLY`** — đánh index không khóa write.
- **`EXPLAIN ANALYZE`** — kiểm index có thực sự được dùng (`Index Scan` vs `Seq Scan`).
- **registerType / register_vector** — đăng ký kiểu vector ở client Node.js/Python.
- **`pgvector/pgvector:pgXX`** — Docker image chính thức, cài sẵn.

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Yêu cầu: **PostgreSQL 13+** (không phải 12); prebuilt khuyến nghị 15+.
2. Cài binary **rồi** mới `CREATE EXTENSION` — hai bước khác nhau, làm ở mỗi database.
3. Cách cài dễ: **Docker / apt-yum / managed**; build-from-source là phương án cuối.
4. Cột `vector(n)` phải **khớp số chiều model**; đổi model → sửa cột + re-embed.
5. `<=>` là **cosine distance** (nhỏ = gần) → `ORDER BY ... <=> ... LIMIT k` ASC.
6. Ops class trong index **phải khớp** toán tử query, nếu không index bị bỏ.
7. Client (Node/Python) phải **đăng ký kiểu** thì vector ↔ mảng mới parse đúng.
8. Verify bằng `\dx` (extension) và `EXPLAIN ANALYZE` (index được dùng).
9. Production: **không build source**; pin version (Docker/gói), migration idempotent, `CREATE INDEX CONCURRENTLY`.
10. Gotcha hàng đầu: `type "vector" does not exist` (schema/search_path), dimension mismatch, seq scan trên bảng nhỏ.

### 5.3. Ideas / mental models

- **"Cài file ≠ bật extension":** hai bước, đừng gộp.
- **"Có index ≠ dùng index":** luôn `EXPLAIN ANALYZE`.
- **"Build tay = crash lạ ở máy khác":** dùng Docker/gói cho di động.
- **"Pin version như mọi dependency":** pgvector không phải thứ build lúc deploy.
- **"registerType là cây cầu":** không có nó, app nhận vector dạng chuỗi.

### 5.4. Code cần thuộc lòng

**(a) Cài + bật + tạo bảng (chuỗi lệnh cơ bản):**
```bash
docker run -d -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg17
```
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id serial PRIMARY KEY, content text, embedding vector(512));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
```

**(b) Insert + query:**
```sql
INSERT INTO items (content, embedding) VALUES ('hello', '[...]');
SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 1;
```

**(c) Node.js (đăng ký kiểu — bước hay bị quên):**
```javascript
const pgvector = require('pgvector/pg');
await pgvector.registerType(client);
await client.query('INSERT INTO items (embedding) VALUES ($1)', [pgvector.toSql(vec)]);
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Cài pgvector cần gì và làm sao?"** → PostgreSQL 13+; cài binary (Docker/apt/yum/source) rồi `CREATE EXTENSION IF NOT EXISTS vector` trong mỗi database; verify `\dx vector`.

2. **[BẪY] "`make install` xong là dùng được vector chưa?"** → Chưa. Mới chép file; phải `CREATE EXTENSION` trong database. Hai bước riêng.

3. **[BẪY] "`<=>` trả về cosine similarity đúng không?"** → Không — trả **cosine distance** (`1 − similarity`), nhỏ = gần, dùng `ORDER BY ASC`.

4. **"Vì sao có index HNSW mà query vẫn `Seq Scan`?"** → Bảng nhỏ (seq rẻ hơn), ops class không khớp toán tử, hoặc cost estimation. Xác nhận bằng `EXPLAIN ANALYZE`.

5. **[BẪY] "`type vector does not exist` sau CREATE EXTENSION?"** → Extension ở schema khác `search_path`; sửa search_path hoặc tạo extension ở `public`.

6. **[OPS] "Thêm pgvector vào Postgres production không downtime?"** → Cài gói/managed (không build tay), `CREATE EXTENSION` qua migration, `ADD COLUMN` nullable + backfill nền, `CREATE INDEX CONCURRENTLY`, verify, đồng bộ version trên replica.

7. **"Kết nối pgvector từ app thế nào?"** → Cài client (`pgvector` cho Node, `pgvector[psycopg]` cho Python), **đăng ký kiểu** (`registerType`/`register_vector`), rồi query như bình thường với `$1`/`%s`.

8. **[STAFF] "Nên build-from-source trên production không?"** → Không — phụ thuộc CPU (`-march=native`), khó tái lập/rollback. Pin version qua Docker tag hoặc gói OS.

### 5.6. One-liner đắt giá

- *"Installing the binary and enabling the extension are two different steps — `make install` then `CREATE EXTENSION`."*
- *"`<=>` is cosine distance, not similarity — smaller is closer, so you ORDER BY ascending."*
- *"Having an index doesn't mean it's used — always confirm with EXPLAIN ANALYZE."*
- *"Don't build pgvector from source in production; pin a Docker tag or OS package like any other dependency."*
- *"Register the vector type in your client, or your app gets back a string instead of an array."*
- *"On a live database: extension via migration, column nullable then backfill, index CONCURRENTLY — zero downtime by construction."*

---

### 📌 Ghi chú cuối

- **Đính chính để nhớ đúng:** PostgreSQL **13+** (không phải 12); `<=>` = cosine **distance**; build-from-source là cách khó nhất, ưu tiên **Docker/apt/managed**; nhớ **đăng ký kiểu** ở client.
- **Kiểm chứng khi ôn:** xem README chính thức `github.com/pgvector/pgvector` cho phiên bản Postgres tối thiểu, các cách cài, và API client mới nhất (`registerType`/`toSql` có thể đổi theo bản).
- **Thực hành:** chạy `docker run pgvector/pgvector:pg17`, làm trọn vòng: `CREATE EXTENSION` → tạo bảng `vector(384)` → insert vài embedding thật (từ giáo trình embeddings) → `ORDER BY <=>` → `EXPLAIN ANALYZE` để thấy `Index Scan`. Rồi nối từ một script Node.js/Python.
- **Nối mạch series (đủ 6 mảnh):** embed (tạo vector) → index (làm nhanh) → **cài & dùng pgvector (bài này, thực hành)** → store/query khái niệm (#3) → keyword/FTS → chọn nền tảng. Bạn giờ có cả *hiểu* lẫn *làm được* semantic search / RAG trên PostgreSQL.
- **Học tiếp:** kết nối pgvector với framework (LangChain/LlamaIndex), pipeline embedding bất đồng bộ, và triển khai một RAG app end-to-end.
