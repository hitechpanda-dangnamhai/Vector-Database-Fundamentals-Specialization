# Vector Search với PostgreSQL (pgvector) — Giáo trình từ Basic đến Staff Level

> **Nguồn gốc:** Bài giảng gốc là video giới thiệu của khóa *IBM Vector Database Fundamentals — Course 3*. Video gốc chỉ nêu 3 mục tiêu học tập (modeling vector data, indexing techniques, dùng PostgreSQL để store/retrieve/query vectors) chứ chưa dạy nội dung kỹ thuật. Vì vậy **~95% giáo trình này là phần giảng mở rộng** của tôi (Staff Engineer), bám đúng 3 mục tiêu đó. Những chỗ mở rộng sâu ngoài phạm vi thông thường sẽ được đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Cập nhật tới 2026:** pgvector bản mới nhất là **v0.8.x** (nhánh 0.8.4). HNSW là default index được khuyên dùng. Các tính năng mới đáng chú ý: iterative index scans (0.8.0), `halfvec`/`sparsevec`, binary quantization, và hệ sinh thái DiskANN qua `pgvectorscale`.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Cách biến một database quan hệ bình thường (PostgreSQL) thành một hệ thống có khả năng **similarity search** — tìm những bản ghi "giống nhau về mặt ngữ nghĩa" thay vì khớp chính xác từ khóa. Ta làm điều đó bằng extension **pgvector**: lưu embedding (vector số) ngay trong bảng SQL, đánh index để tìm nhanh, rồi query bằng chính SQL quen thuộc.

**Vấn đề nó giải quyết:** Database truyền thống chỉ giỏi tìm khớp chính xác (`WHERE name = 'áo thun'`). Nó không hiểu "áo phông", "T-shirt", "đồ mặc mùa hè" là *gần nghĩa*. Vector search lấp đúng lỗ hổng này — nền tảng của semantic search, recommendation, và RAG (Retrieval-Augmented Generation).

**Học xong bạn sẽ làm được:**

- Giải thích embedding là gì và tại sao "khoảng cách vector" = "độ giống nhau".
- Thiết kế schema lưu vector trong PostgreSQL (data modeling).
- Store, query, filter vector bằng SQL với 3 distance metric chính.
- Chọn và tune đúng index (HNSW vs IVFFlat vs DiskANN), hiểu Big-O của chúng.
- Phân tích trade-off ở quy mô lớn (triệu → tỷ vectors), chi phí, và trả lời được câu hỏi system design về vector search.

**Mạch kiến thức basic → staff:**

- 🟢 **Basic:** Embedding là gì → distance = similarity → cài pgvector, tạo cột `vector`, query bằng toán tử `<->`.
- 🟡 **Intermediate:** Ba distance metric (L2 / cosine / inner product), tạo HNSW index, pipeline Python thật với embeddings, filtered search, lỗi kinh điển.
- 🔴 **Advanced:** Cơ chế bên trong HNSW & IVFFlat, Big-O, curse of dimensionality, tự implement k-NN + IVF, edge cases (giới hạn 2000 dims, chuẩn hóa vector).
- 🟣 **Staff:** Scale lên hàng tỷ vectors, sharding/partitioning, quantization để cắt chi phí, monitoring recall, khi nào KHÔNG dùng pgvector, câu hỏi system design mẫu.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Vấn đề thực tế trước đã

Bạn có một shop online. Khách gõ ô tìm kiếm: **"đồ giữ ấm mùa đông"**. Trong DB bạn có sản phẩm tên *"áo khoác lông vũ"*, *"khăn len"*, *"găng tay"*. Không sản phẩm nào chứa đúng cụm từ khách gõ.

- `WHERE name LIKE '%đồ giữ ấm mùa đông%'` → trả về **0 kết quả**. Khách bỏ đi.

Vấn đề: SQL truyền thống so khớp **ký tự/từ khóa**, nó không hiểu *ý nghĩa*. Con người thì hiểu ngay "áo khoác lông vũ" liên quan mật thiết tới "giữ ấm mùa đông". Ta cần máy tính "hiểu nghĩa" giống vậy. Đó là lý do **vector search** ra đời.

### 1.2. Analogy đời thường: bản đồ ý nghĩa

Tưởng tượng một **tấm bản đồ khổng lồ**, nhưng thay vì đặt thành phố, ta đặt **từ ngữ/câu/sản phẩm** lên đó. Quy tắc đặt: **thứ nào nghĩa giống nhau thì nằm gần nhau**.

- "áo khoác lông vũ", "áo phao", "khăn len" → nằm sát nhau một góc bản đồ.
- "kem chống nắng", "quần short" → nằm sát nhau ở góc đối diện.

Giờ khách gõ "đồ giữ ấm mùa đông" → ta cũng đặt cụm này lên bản đồ, rồi hỏi: **"những điểm nào gần điểm này nhất?"** → trả về áo phao, khăn len. Xong. Đó chính là **similarity search**: tìm hàng xóm gần nhất (nearest neighbors) trên bản đồ ý nghĩa.

Tấm bản đồ này thực tế không phải 2 chiều mà là hàng trăm/nghìn chiều. Tọa độ của mỗi điểm chính là **embedding**.

### 1.3. Định nghĩa các thuật ngữ (lần đầu xuất hiện)

- **Embedding** (nhúng): một dãy số (vector) biểu diễn ý nghĩa của một object (chữ, ảnh, âm thanh). Ví dụ `[0.12, -0.98, 0.33, ...]`. Được sinh ra bởi một **embedding model** (mô hình AI đã học sẵn). Cùng ý nghĩa → vector gần nhau.
- **Vector**: chính là embedding — một mảng số thực có độ dài cố định. Độ dài đó gọi là **dimension** (số chiều). Ví dụ model `text-embedding-3-small` của OpenAI cho vector **1536 dimensions**.
- **Distance / similarity**: thước đo "gần nhau" giữa 2 vector. Khoảng cách nhỏ = giống nhau. (Chi tiết ở Phần 2.)
- **Nearest Neighbor Search (NN search)**: bài toán tìm K điểm gần nhất với một điểm truy vấn. "k-NN" = k nearest neighbors.
- **pgvector**: extension mã nguồn mở giúp PostgreSQL lưu và search vector. Nó thêm kiểu dữ liệu mới tên `vector`.
- **PostgreSQL / Postgres**: một hệ quản trị relational database (RDBMS) phổ biến, mã nguồn mở.
- **Extension**: một gói mở rộng cài thêm vào Postgres để có tính năng mới (pgvector là một extension).

### 1.4. Ví dụ chạy tay — "thấy" cơ chế bằng tay

Ta bỏ qua model AI, tự bịa embedding 2 chiều để tính bằng tay. Giả sử ta đặt 3 sản phẩm và 1 câu truy vấn lên "bản đồ 2D":

| Object | Vector (x, y) |
|---|---|
| Áo phao (A) | (2, 9) |
| Khăn len (B) | (3, 8) |
| Kem chống nắng (C) | (9, 1) |
| **Query: "giữ ấm mùa đông" (Q)** | **(2, 8)** |

**Euclidean distance (L2)** giữa 2 điểm = `sqrt((x1-x2)² + (y1-y2)²)`. Ta tính khoảng cách từ Q tới từng điểm:

- `distance(Q, A) = sqrt((2-2)² + (8-9)²) = sqrt(0 + 1) = 1.00`
- `distance(Q, B) = sqrt((2-3)² + (8-8)²) = sqrt(1 + 0) = 1.00`
- `distance(Q, C) = sqrt((2-9)² + (8-1)²) = sqrt(49 + 49) = sqrt(98) ≈ 9.90`

**Sắp xếp tăng dần:** A (1.00) ≈ B (1.00) ≪ C (9.90).

→ Query "giữ ấm mùa đông" gần **áo phao và khăn len**, xa **kem chống nắng**. Đúng như trực giác! Đây **chính xác** là những gì pgvector làm bên trong khi bạn gõ `ORDER BY embedding <-> query`, chỉ khác là với 1536 chiều thay vì 2. Toán không đổi, chỉ nhiều số hạng hơn.

### 1.5. Code "hello world" đơn giản nhất

Toàn bộ chạy trong `psql` hoặc bất kỳ SQL client nào, sau khi đã cài pgvector.

```sql
-- Bước 1: Bật extension (chạy 1 lần cho mỗi database)
CREATE EXTENSION IF NOT EXISTS vector;

-- Bước 2: Tạo bảng với 1 cột kiểu vector 3 chiều (dùng 3D cho dễ nhìn)
CREATE TABLE items (
    id     bigserial PRIMARY KEY,   -- khóa chính tự tăng
    name   text,                    -- cột dữ liệu bình thường
    embedding vector(3)             -- cột vector: mỗi giá trị là 1 mảng 3 số
);

-- Bước 3: Chèn dữ liệu. Vector viết trong dấu nháy đơn, dạng '[a,b,c]'
INSERT INTO items (name, embedding) VALUES
    ('áo phao',        '[2, 9, 0]'),
    ('khăn len',       '[3, 8, 0]'),
    ('kem chống nắng', '[9, 1, 0]');

-- Bước 4: Similarity search!
-- Toán tử <-> nghĩa là "L2 distance". ORDER BY ... LIMIT k = tìm k hàng xóm gần nhất.
SELECT name, embedding <-> '[2, 8, 0]' AS distance
FROM items
ORDER BY embedding <-> '[2, 8, 0]'   -- gần nhất lên đầu
LIMIT 2;
```

**Kết quả:**

```
   name    | distance
-----------+----------
 áo phao   |        1
 khăn len  |        1
```

Giống hệt phép tính tay ở 1.4. **Điểm mấu chốt cần khắc cốt:** toán tử `<->` là trái tim của mọi thứ. `ORDER BY cot_vector <-> query_vector LIMIT k` = "cho tôi k bản ghi giống nhất". Bạn đã biết viết vector search rồi đấy.

### 1.6. Ba toán tử distance cần biết ngay (sẽ đào sâu ở Phần 2)

| Toán tử | Ý nghĩa | Dùng khi |
|---|---|---|
| `<->` | L2 (Euclidean) distance | mặc định, an toàn |
| `<=>` | Cosine distance | phổ biến nhất với text embeddings |
| `<#>` | Negative inner product | khi vector đã normalize, cần tốc độ |

### 1.7. [MỞ RỘNG NGOÀI BÀI GỐC] — 2 điều một Staff Engineer nhắc ngay từ đầu

1. **pgvector KHÔNG sinh ra embedding.** Nó chỉ *lưu và tìm* vector. Việc biến "áo phao" → `[2,9,0]` là do một **embedding model** riêng (OpenAI API, `sentence-transformers`, Cohere...). Đây là kiến trúc 2 phần mà junior hay nhầm gộp làm một.
2. **Mặc định pgvector làm exact search (100% recall) và KHÔNG cần index.** Với bảng nhỏ (< ~10K–50K hàng), Postgres quét tuần tự (sequential scan) vẫn nhanh và cho kết quả *chính xác tuyệt đối*. Index (Phần 2) chỉ cần khi dữ liệu lớn — và khi đó ta **đánh đổi độ chính xác lấy tốc độ**. Nhiều người vội đánh index quá sớm rồi than "sao kết quả sai".

### ✅ Self-check Phần 1

1. Tại sao `WHERE name LIKE '%...%'` không giải quyết được bài toán "đồ giữ ấm mùa đông"? (Gợi ý: khớp ký tự vs khớp ý nghĩa.)
2. Trong `ORDER BY embedding <-> '[2,8,0]' LIMIT 3`, con số `<->` càng nhỏ nghĩa là gì?
3. pgvector có tự tạo embedding từ chữ "áo phao" không? Ai làm việc đó?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

Giả định bạn đã nắm: embedding, distance, cột `vector`, toán tử `<->`.

### 2.1. Ba distance metric — hoạt động bên trong thế nào

Đây là chỗ nhiều người dùng sai và làm hỏng chất lượng search. Cho 2 vector **a** và **b**:

**1) L2 / Euclidean distance (`<->`)** — khoảng cách đường chim bay.
```
L2(a, b) = sqrt( Σ (aᵢ - bᵢ)² )
```
Nhạy cả với *hướng* lẫn *độ dài* của vector.

**2) Cosine distance (`<=>`)** — đo **góc** giữa 2 vector, bỏ qua độ dài.
```
cosine_similarity(a, b) = (a · b) / (|a| · |b|)      # trong [-1, 1], 1 = cùng hướng
cosine_distance     = 1 - cosine_similarity          # pgvector trả cái này, [0, 2]
```
**Đây là lựa chọn mặc định cho text embeddings.** Vì với text, cái ta quan tâm là *hướng ngữ nghĩa*, không phải "vector dài hay ngắn".

**3) Inner product / dot product (`<#>`)** — tích vô hướng.
```
a · b = Σ aᵢ bᵢ
```
pgvector trả về **negative** inner product (`<#>` = `-(a·b)`) để "nhỏ hơn = gần hơn" cho nhất quán với `ORDER BY`. **Nếu vector đã được normalize (độ dài = 1) thì inner product ≡ cosine, nhưng tính nhanh hơn** (bỏ được phép chia chuẩn hóa). Đây là mẹo tối ưu hay dùng ở production.

> **Quy tắc chọn nhanh:** Text embeddings từ OpenAI/most models → dùng **cosine** (`<=>`). Nếu bạn tự normalize vector về length 1 → dùng **inner product** (`<#>`) để nhanh hơn. Chỉ dùng L2 khi độ lớn vector thực sự mang nghĩa (ít gặp với text).

### 2.2. Data modeling: thiết kế schema thật (mục tiêu #1 của bài gốc)

Trong thực tế bạn không lưu mỗi vector. Bạn lưu vector **cạnh** dữ liệu nghiệp vụ — đây chính là siêu năng lực của pgvector (vector sống chung nhà với business data, join được bằng SQL).

```sql
CREATE TABLE documents (
    id          bigserial PRIMARY KEY,
    tenant_id   bigint      NOT NULL,          -- multi-tenant: lọc theo khách hàng
    title       text        NOT NULL,
    content     text        NOT NULL,
    category    text,                          -- metadata để filter
    created_at  timestamptz NOT NULL DEFAULT now(),
    embedding   vector(1536)                   -- khớp số chiều của model bạn dùng!
);

-- Index thường trên metadata để filter nhanh (RẤT quan trọng, xem 2.6)
CREATE INDEX ON documents (tenant_id, category);
CREATE INDEX ON documents (created_at);
```

**Điểm staff cần lưu ý ngay:** con số `1536` **phải khớp** đúng output dimension của embedding model. Nhét vector 768 chiều vào cột `vector(1536)` → Postgres báo lỗi. Đổi model = phải re-embed lại toàn bộ (một quyết định migration tốn kém, xem Phần 4).

### 2.3. Đánh HNSW index — cách làm chuẩn

Khi bảng lớn, sequential scan chậm. Ta thêm **approximate nearest neighbor (ANN) index**. HNSW là mặc định 2026.

```sql
-- Cú pháp: chọn ops class KHỚP với distance metric bạn sẽ query
--   vector_cosine_ops  -> dùng với <=>
--   vector_l2_ops      -> dùng với <->
--   vector_ip_ops      -> dùng với <#>
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

- `m` = số kết nối tối đa mỗi node trong graph (mặc định 16). Cao hơn → recall tốt hơn, index to hơn, build chậm hơn.
- `ef_construction` = độ "cẩn thận" khi xây graph (mặc định 64). Cao hơn → recall tốt hơn, build chậm hơn.

**Query-time tuning** (quan trọng ngang lúc build):

```sql
BEGIN;
SET LOCAL hnsw.ef_search = 100;   -- mặc định 40. Cao hơn = recall tốt hơn, chậm hơn.
SELECT id, title
FROM documents
ORDER BY embedding <=> '[...1536 số...]'
LIMIT 10;
COMMIT;
```

> **Mẹo staff:** Dùng `SET LOCAL` bên trong `BEGIN...COMMIT`, **KHÔNG** dùng `SET` toàn cục trên production. `SET` thường sẽ dính vào connection và — với connection pooler (PgBouncer) — rò rỉ sang query của người khác. Đây là bug production rất khó truy.

### 2.4. Pipeline thực tế bằng Python (mục tiêu #3 của bài gốc)

Đây là code gần công việc thật: dùng `sentence-transformers` để tạo embedding (chạy local, miễn phí, không cần API key) + `psycopg` (v3) để nói chuyện với Postgres.

```python
# pip install sentence-transformers psycopg[binary] pgvector
from sentence_transformers import SentenceTransformer
import psycopg
from pgvector.psycopg import register_vector

# Model này cho vector 384 chiều -> cột phải là vector(384)
model = SentenceTransformer("all-MiniLM-L6-v2")

DOCS = [
    ("Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét", "outerwear"),
    ("Kem chống nắng SPF50 bảo vệ da khỏi tia UV mùa hè",     "skincare"),
    ("Khăn len dệt tay ấm áp cho những ngày lạnh",            "accessories"),
    ("Quần short thoáng mát đi biển ngày nắng",               "clothing"),
]

conn = psycopg.connect("postgresql://user:pass@localhost/shop")
register_vector(conn)  # dạy psycopg hiểu kiểu vector <-> numpy array

with conn.cursor() as cur:
    cur.execute("CREATE EXTENSION IF NOT EXISTS vector")
    cur.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id serial PRIMARY KEY,
            content text,
            category text,
            embedding vector(384)
        )
    """)

    # 1) Sinh embedding cho từng document rồi INSERT
    for content, category in DOCS:
        vec = model.encode(content)  # -> numpy array 384 chiều
        cur.execute(
            "INSERT INTO products (content, category, embedding) VALUES (%s, %s, %s)",
            (content, category, vec),
        )
    conn.commit()

    # 2) Đánh HNSW index (nên làm SAU khi đã nạp dữ liệu ban đầu -> nhanh hơn)
    cur.execute("""
        CREATE INDEX IF NOT EXISTS products_emb_idx
        ON products USING hnsw (embedding vector_cosine_ops)
    """)
    conn.commit()

    # 3) Query: embed câu hỏi rồi tìm 2 sản phẩm gần nhất
    query_vec = model.encode("đồ mặc cho trời lạnh mùa đông")
    cur.execute("""
        SELECT content, embedding <=> %s AS distance
        FROM products
        ORDER BY embedding <=> %s
        LIMIT 2
    """, (query_vec, query_vec))

    for content, distance in cur.fetchall():
        print(f"{distance:.4f}  |  {content}")

conn.close()
```

**Kết quả kỳ vọng** (khoảng cách nhỏ = gần nghĩa):
```
0.31xx  |  Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét
0.42xx  |  Khăn len dệt tay ấm áp cho những ngày lạnh
```
Câu query không chứa từ "áo khoác" hay "khăn len", nhưng model hiểu *ý nghĩa* → trả về đúng đồ mùa đông. Đó là semantic search thật sự.

### 2.5. Filtered search — join vector với business logic

Đây là chỗ pgvector đánh bại vector DB chuyên dụng (Pinecone...): filter bằng SQL trong **một** câu, một round-trip.

```sql
-- "Tìm 10 sản phẩm giống nhất, NHƯNG chỉ trong category 'outerwear',
--  của tenant 42, còn hàng, tạo trong 30 ngày gần đây"
SELECT id, content
FROM products
WHERE category  = 'outerwear'
  AND tenant_id = 42
  AND in_stock  = true
  AND created_at > now() - interval '30 days'
ORDER BY embedding <=> '[...]'
LIMIT 10;
```

Với Pinecone bạn phải: query vector → nhận IDs → query lại DB nghiệp vụ để lấy chi tiết + filter → 2 service, 2 network hop, nhiều glue code, nhiều failure mode. Với pgvector: 1 query. **Đây là "one-liner" đắt giá để nói trong phỏng vấn.**

### 2.6. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp (và cách tránh)

**Lỗi 1 — Overfiltering / "thiếu kết quả khi filter chặt".** ANN index (HNSW) tìm `ef_search` hàng xóm *trước*, rồi Postgres mới áp `WHERE`. Nếu filter của bạn hiếm (vd category chỉ chiếm 0.5% dữ liệu), có thể sau khi lọc chỉ còn 1–2 kết quả thay vì 10 bạn yêu cầu.
→ **Cách tránh:** bật **iterative index scan** (tính năng chủ lực từ pgvector 0.8.0):
```sql
SET hnsw.iterative_scan = relaxed_order;  -- hoặc strict_order
SET hnsw.max_scan_tuples = 20000;         -- trần số tuple quét thêm
```
`relaxed_order` cho tốc độ tốt, giữ ~95–99% chất lượng — thường là lựa chọn production tốt nhất.

**Lỗi 2 — Dùng sai ops class so với toán tử query.** Đánh index `vector_l2_ops` nhưng query bằng `<=>` (cosine) → **Postgres bỏ qua index**, âm thầm rơi về sequential scan (chậm) hoặc cho kết quả bạn không ngờ. Ops class trong index PHẢI khớp toán tử query. Kiểm bằng `EXPLAIN ANALYZE` xem có "Index Scan using ...hnsw..." không.

**Lỗi 3 — Quên rằng ANN là *approximate*.** Sau khi đánh index, kết quả có thể *thiếu* vài hàng xóm thật (recall < 100%). Junior test trên bảng nhỏ (exact) thấy đúng, lên production (index) thấy "sai", hoảng loạn. Đây là *by design*. Giải: tăng `ef_search`/`m`, hoặc đo recall (Phần 4) để biết mình đang ở đâu.

### ✅ Self-check Phần 2

1. Vector embeddings của bạn từ OpenAI. Nên chọn `<->`, `<=>` hay `<#>`? Vì sao?
2. Bạn đánh index `USING hnsw (embedding vector_cosine_ops)` nhưng query bằng `<->`. Chuyện gì xảy ra?
3. Filter `WHERE category='rare_cat'` (chiếm 0.3% data) trả về chỉ 2 kết quả thay vì 10. Tên hiện tượng này là gì, khắc phục ra sao?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Vì sao NN search khó? — Curse of dimensionality

Ở Phần 1 ta tính distance tới cả 3 điểm — đó là **brute-force exact search**, độ phức tạp `O(n · d)` với `n` = số vector, `d` = số chiều. Với 100 triệu vector × 1536 chiều, mỗi query là ~1.5×10¹¹ phép nhân → **vô phương** cho real-time.

Tại sao không dùng cây kd-tree như trong không gian 2D/3D? Vì **curse of dimensionality**: khi `d` lớn (hàng trăm–nghìn), mọi điểm gần như *cách đều nhau*, cây phân hoạch không gian mất tác dụng (phải duyệt gần hết cây). Đây là lý do ta cần **approximate** NN (ANN): chấp nhận bỏ sót vài hàng xóm để đổi lấy tốc độ gấp hàng nghìn lần.

### 3.2. IVFFlat — cơ chế "chia ô rồi chỉ tìm trong vài ô"

**Ý tưởng (analogy):** thay vì hỏi cả thành phố "ai gần tôi nhất", ta chia thành phố thành các **quận** (dùng k-means clustering trên tập vector), mỗi vector thuộc quận có tâm (centroid) gần nó nhất. Khi query, ta chỉ vào **vài quận gần nhất** với điểm query để tìm — bỏ qua phần lớn dữ liệu.

- Xây index: chạy k-means để tìm `lists` centroid → gán mỗi vector vào 1 list. **IVFFlat cần dữ liệu sẵn để train** (khác HNSW).
- Query: so query với các centroid → chọn `probes` list gần nhất → brute-force *trong* các list đó.

```sql
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 1000);       -- gợi ý: rows/1000 (≤1M rows), sqrt(rows) (>1M rows)

SET ivfflat.probes = 32;   -- gợi ý khởi đầu: sqrt(lists). Cao hơn = recall↑, chậm↓
```

**Trade-off `probes`:** đây là núm xoay recall–speed. `probes=1` cực nhanh nhưng dễ trượt (nếu hàng xóm thật nằm ở list kế bên). `probes=lists` = quét hết = exact nhưng mất hết lợi ích.

- Query complexity xấp xỉ: `O(lists·d)` (so centroid) `+ O((n/lists)·probes·d)` (quét trong list).
- Điểm yếu chí mạng: **hàng xóm nằm sát ranh giới quận bị bỏ sót**. Và IVFFlat *tĩnh* — thêm nhiều dữ liệu mới sau khi build → phân hoạch lệch, recall tụt, phải `REINDEX`.

### 3.3. HNSW — graph nhiều tầng "đi tắt"

**Ý tưởng (analogy):** giống hệ thống giao thông có nhiều tầng. Tầng trên cùng ít node nhưng là "đường cao tốc" nối các vùng xa; tầng dưới cùng chứa mọi node với đường "hẻm" ngắn. Query bắt đầu ở tầng cao (nhảy xa nhanh về đúng vùng), rồi tụt dần xuống tầng thấp để tinh chỉnh tới hàng xóm chính xác. HNSW = **Hierarchical Navigable Small World**.

- Build: chèn từng node, nối nó với `m` hàng xóm gần nhất ở mỗi tầng; tầng của node chọn ngẫu nhiên theo phân phối mũ (ít node lên tầng cao).
- Search complexity: **~`O(log n)`** cho mỗi query (thay vì `O(n)`) — đây là con số vàng cần nhớ.
- Build complexity: **~`O(n · log n · m · d)`** — chậm và tốn RAM hơn IVFFlat nhiều.

### 3.4. Bảng so sánh index (câu hỏi phỏng vấn kinh điển)

| Tiêu chí | IVFFlat | HNSW | DiskANN (pgvectorscale) |
|---|---|---|---|
| Query speed/recall | Tốt | **Rất tốt** | Rất tốt |
| Build time | **Nhanh** | Chậm | Trung bình |
| Bộ nhớ | **Ít** | Nhiều (graph in-RAM) | Ít (SSD-based + quantize) |
| Cần data để build? | **Có** (train k-means) | Không | Có |
| Query complexity | ~O((n/lists)·probes·d) | **~O(log n)** | ~O(log n) |
| Thêm data mới (dynamic) | Kém (lệch phân hoạch) | **Tốt** | Tốt |
| Dim > 2000 (native) | Không (cần halfvec) | Không (cần halfvec) | **Có (tới 16000)** |
| Khi nào chọn | build lại thường xuyên, RAM hạn chế, dữ liệu tĩnh | **default 2026**, cần latency thấp + recall cao | dataset khổng lồ, tiết kiệm RAM/chi phí |

> **[MỞ RỘNG NGOÀI BÀI GỐC]** DiskANN (qua extension `pgvectorscale` của Timescale) là "player thứ 3" đáng biết ở tầm staff: giữ phần lớn graph trên SSD thay vì RAM + binary quantization → chỉ số cùng dataset có thể nhỏ hơn HNSW ~9x (vd 21MB vs 193MB trong benchmark công khai) và support vector tới 16000 chiều native. Nhắc được cái này trong phỏng vấn là điểm cộng lớn.

### 3.5. Edge case: giới hạn 2000 chiều & cách xử lý

Đây là "cú vấp" kinh điển với model hiện đại. Kiểu `vector` **lưu** được tới 16000 chiều, nhưng HNSW/IVFFlat index chỉ **index** được tối đa **2000 chiều**:

```sql
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
-- ERROR: column cannot have more than 2000 dimensions for hnsw index
```

Model `text-embedding-3-large` (3072 dims) dính ngay lỗi này. **Cách xử lý (staff phải biết cả 3):**

1. **`halfvec`** — half-precision (16-bit thay 32-bit): index được tới ~4000 chiều, giảm nửa dung lượng, mất recall không đáng kể:
   ```sql
   CREATE INDEX ON docs USING hnsw ((embedding::halfvec(3072)) halfvec_cosine_ops);
   ```
2. **Matryoshka / dimension reduction** — nhiều model (OpenAI v3) cho phép cắt vector ngắn lại (vd 3072 → 1024) với mất mát nhỏ, vì chúng train kiểu "thông tin quan trọng dồn về đầu vector".
3. **DiskANN (`pgvectorscale`)** — support native tới 16000 chiều, không cần workaround.

### 3.6. Edge case: cosine với vector chưa normalize; và giá trị NULL

- **Normalize để dùng inner product:** nếu bạn normalize mọi vector về length 1 lúc insert, có thể dùng `<#>` (nhanh hơn cosine). Nhưng phải normalize **cả** vector query lúc search, nếu không kết quả sai lệch.
- **NULL embedding:** hàng có `embedding IS NULL` sẽ không xuất hiện trong ANN result. Thường gặp khi pipeline embed lỗi vài dòng. Nên có job kiểm `WHERE embedding IS NULL` và re-embed.

### 3.7. Tự implement để hiểu tận gốc

**(a) Exact k-NN cosine bằng NumPy** — chính là cái pgvector làm khi không có index:

```python
import numpy as np

def cosine_knn(query: np.ndarray, corpus: np.ndarray, k: int = 3):
    """Trả về (indices, distances) của k vector gần nhất theo cosine distance.
    query:  shape (d,)
    corpus: shape (n, d)
    """
    # Chuẩn hóa để tích vô hướng == cosine similarity
    q = query / np.linalg.norm(query)
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)
    sims = c @ q                      # (n,) cosine similarity từng vector với query
    dists = 1.0 - sims                # cosine distance (khớp <=> của pgvector)
    idx = np.argsort(dists)[:k]       # k khoảng cách nhỏ nhất
    return idx, dists[idx]

corpus = np.array([[2,9,0],[3,8,0],[9,1,0]], dtype=float)
idx, d = cosine_knn(np.array([2,8,0], dtype=float), corpus, k=2)
print(idx, d)   # -> [0 1] ... (áo phao, khăn len)
```

**(b) IVF mini — hiểu "chia ô rồi tìm vài ô":**

```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def __init__(self, n_lists=8):
        self.n_lists = n_lists

    def build(self, corpus):
        self.corpus = corpus
        km = KMeans(n_clusters=self.n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_          # tâm mỗi "quận"
        self.assign = km.labels_                       # mỗi vector thuộc quận nào
        return self

    def search(self, query, k=3, probes=2):
        # 1) tìm `probes` quận gần query nhất (thay vì quét toàn bộ corpus)
        cdist = np.linalg.norm(self.centroids - query, axis=1)
        near_lists = np.argsort(cdist)[:probes]
        # 2) chỉ brute-force trong các quận đã chọn
        mask = np.isin(self.assign, near_lists)
        cand_idx = np.where(mask)[0]
        d = np.linalg.norm(self.corpus[cand_idx] - query, axis=1)
        order = np.argsort(d)[:k]
        return cand_idx[order], d[order]
```
Chạy thử với `probes` nhỏ vs lớn để **tự thấy** trade-off recall–speed: `probes` nhỏ nhanh nhưng đôi khi trượt hàng xóm nằm ở quận không được chọn.

### ✅ Self-check Phần 3

1. Query complexity của HNSW là bao nhiêu? Của brute-force? Nêu bằng Big-O.
2. Bạn có model 3072 chiều và cần HNSW index. Nêu 2 cách xử lý.
3. Trong IVFFlat, tăng `probes` ảnh hưởng recall và latency thế nào? Vì sao hàng xóm sát ranh giới list dễ bị bỏ sót?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Scale: từ 1 triệu tới hàng tỷ vectors

**Câu hỏi cốt lõi ở tầm staff không phải "dùng index nào" mà là "hệ thống này gãy ở đâu khi lớn lên".**

- **< 10 triệu vectors:** một node PostgreSQL + HNSW là *đủ và tốt nhất* về vận hành. Query single-digit ms. Đây là 90% use case B2B RAG/search. Đừng over-engineer.
- **Bottleneck xuất hiện:** HNSW graph phải nằm gọn trong RAM để nhanh. Vector 1536 dims × 4 bytes ≈ 6KB/vector; 100M vector ≈ **600GB chỉ riêng raw vectors**, chưa kể overhead graph. RAM là ràng buộc số một → đây là lúc tính tới **quantization** và **DiskANN**.
- **Build time bottleneck:** HNSW build là `O(n log n)`, tốn hàng giờ ở quy mô lớn. Tăng tốc bằng: nạp data xong mới build; tăng `maintenance_work_mem` (khuyến nghị ~working set, nhưng ≤ 50–60% RAM); tăng `max_parallel_maintenance_workers`.

**Scale vertical (dễ, làm trước):** thêm RAM/CPU/SSD cho 1 instance. Đơn giản, đủ xa.

**Scale horizontal (khi hết đường vertical):**
- **Read replicas** cho query load (search là read-heavy → replica scale rất tốt).
- **Partitioning** theo access pattern (KHÔNG chỉ theo size): range/list-partition theo `tenant_id`, `locale`, `product_line` — cột hay xuất hiện trong filter. Build vector index *per-partition* → planner prune (loại bỏ) cả partition không liên quan trước khi search → giảm latency mạnh dưới tải multi-tenant. Bảng non-partitioned giới hạn 32TB; partitioned thì gần như vô hạn.
- **Sharding** qua **Citus**, **PgDog**, hoặc **pgvectorscale** khi 1 node không chứa nổi.

### 4.2. Quantization — cắt chi phí RAM/storage ở tầm kiến trúc [MỞ RỘNG]

Đây là đòn bẩy chi phí lớn nhất mà staff phải nắm:

| Kỹ thuật | Cắt dung lượng | Ảnh hưởng recall |
|---|---|---|
| `halfvec` (fp16) | ~50% | rất nhỏ |
| Binary quantization | tới ~32x | có (bù bằng re-rank exact) |
| Matryoshka (cắt chiều) | tùy tỉ lệ cắt | nhỏ nếu model hỗ trợ |

**Pattern kinh điển: two-stage retrieval (recall → precision).** Giai đoạn 1 dùng index nén (binary/HNSW) lấy top-N ứng viên *nhanh, thô*; giai đoạn 2 **re-rank** N ứng viên đó bằng exact distance (+ business signals: độ mới, popularity, quyền truy cập). Vì N nhỏ nên re-rank rẻ, mà chất lượng cuối cao. Làm re-rank ngay trong SQL để giữ tính atomic.

### 4.3. Chi phí, latency, reliability, monitoring

- **Cost:** managed Postgres có pgvector sẵn (Supabase, Neon, RDS/Aurora) từ vài chục USD/tháng — **rẻ hơn nhiều** so với thêm một vector DB chuyên dụng ($$$/tháng + team vận hành riêng). Giá trị lớn nhất: **không thêm hệ thống mới** (bớt một failure domain, một on-call surface).
- **Latency:** đo p50/p95/p99, không đo trung bình. HNSW tail latency phình khi `ef_search` cao hoặc graph spill khỏi RAM.
- **Monitoring recall (điểm phân biệt staff):** ANN không có "đúng/sai" nhị phân, phải *đo* recall định kỳ bằng cách so kết quả approximate với exact:
  ```sql
  BEGIN;
  SET LOCAL enable_indexscan = off;   -- ép exact search làm ground truth
  SELECT id FROM docs ORDER BY embedding <=> :q LIMIT 10;
  COMMIT;
  -- so overlap với kết quả có index -> recall@10
  ```
  Dựng dashboard theo dõi recall@k theo thời gian; recall tụt = tín hiệu cần re-tune/re-index.
- **Failure modes cần chuẩn bị:** (1) embedding model đổi phiên bản → toàn bộ vector cũ "lệch không gian" với vector mới, kết quả rác; (2) VACUUM trên HNSW rất chậm → reindex trước rồi vacuum; (3) pipeline embed lỗi âm thầm để lại `NULL`/vector rác; (4) `SET` toàn cục rò qua connection pooler.
- **Ops:** `pg_stat_statements`, PgHero để soi query nóng.

### 4.4. Khi nào NÊN và KHÔNG NÊN dùng pgvector

**NÊN:** vector search là *một tính năng trong* app đã chạy Postgres; cần filter/join vector với business data; < 10–50M vectors; muốn ít hệ thống, ít chi phí, ACID transaction.

**KHÔNG NÊN (dùng Pinecone/Weaviate/Qdrant/Milvus):** > 50M–hàng tỷ vectors cần globally distributed multi-region; cần hybrid search (vector + keyword BM25) built-in mạnh; cần managed scaling vượt provider Postgres; **vector search LÀ workload chính** (không phải phụ) → hệ chuyên dụng tối ưu hơn; hoặc tổ chức vốn không dùng Postgres.

> Câu chốt tư duy: **"pgvector thắng ở operational simplicity, không phải ở peak performance. Chọn nó khi vector là một feature, không phải cả sản phẩm."**

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Chúng ta không cần mua và vận hành thêm một database mới. Tận dụng Postgres sẵn có, đội ngũ đã biết vận hành, giảm rủi ro và chi phí. Đổi lại, nếu sau này vượt ~50 triệu bản ghi và cần đa vùng, ta sẽ phải cân nhắc di trú — nhưng đó là bài toán của thành công, giải sau." → framing bằng **risk & cost**, không bằng jargon.
- **Ảnh hưởng roadmap:** quyết định *chọn embedding model* là quyết định kiến trúc nặng ký hơn cả chọn index — đổi model = re-embed toàn bộ corpus (tốn tiền API + downtime/blue-green). Staff phải chốt model sớm, version hóa embedding (`embedding_v2` column), và có kế hoạch backfill.
- **Team topology:** giữ vector trong Postgres nghĩa là team backend hiện tại tự lo được, không cần tuyển/đào tạo riêng cho một vector DB stack — một lập luận tổ chức mạnh.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff

> **"Thiết kế semantic search cho một e-commerce có 50 triệu sản phẩm, hỗ trợ filter theo category/giá/tồn kho, p95 < 100ms, multi-tenant."**

**Khung trả lời staff (nói to các trade-off):**

1. **Clarify trước:** QPS bao nhiêu? Tần suất update catalog? Recall target? Ngân sách? (Staff luôn hỏi trước khi vẽ.)
2. **Embedding layer:** chọn model (vd 1024–1536 dims để cân recall/chi phí), pipeline sinh embedding bất đồng bộ qua queue khi sản phẩm thêm/sửa; version hóa embedding.
3. **Storage/model:** một bảng `products` chứa cả metadata + `vector`, **partition theo tenant_id** (prune sớm), index B-tree trên (category, price, in_stock).
4. **Index:** HNSW `vector_cosine_ops`, `m=16..32`, tune `ef_search` theo SLA; bật **iterative_scan = relaxed_order** để tránh overfiltering khi filter chặt.
5. **Query:** một câu SQL: filter metadata (prune partition) → ORDER BY `<=>` → LIMIT; two-stage re-rank nếu cần precision cao.
6. **Scale/latency:** read replicas cho search traffic; 50M × ~6KB ≈ 300GB → cân RAM, cân **halfvec/binary quantization** hoặc **DiskANN** để vừa RAM và cắt chi phí; đo p95 thật.
7. **Ops:** monitor recall@10 (so exact định kỳ), p99 latency, tỉ lệ NULL embedding; kế hoạch reindex; blue-green khi đổi model.
8. **Khi nào bỏ pgvector:** nếu tăng tới hàng trăm triệu + đa vùng + hybrid search nặng → nêu thẳng là sẽ cân nhắc Milvus/Vespa. **Biết giới hạn của lựa chọn của mình chính là dấu hiệu staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **Embedding** — vector số biểu diễn ý nghĩa của object; gần nhau = giống nghĩa.
- **Vector / dimension** — mảng số thực độ dài cố định; số phần tử = số chiều.
- **Similarity search / k-NN** — tìm k vector gần nhất với query vector.
- **ANN (Approximate Nearest Neighbor)** — đánh đổi recall lấy tốc độ; ngược với exact.
- **Recall** — % hàng xóm thật mà search tìm được; thước đo chất lượng của ANN.
- **pgvector** — extension cho Postgres, thêm kiểu `vector`.
- **HNSW** — Hierarchical Navigable Small World; graph nhiều tầng; query ~O(log n); default 2026.
- **IVFFlat** — Inverted File Flat; chia list bằng k-means; build nhanh, cần data để train.
- **DiskANN / pgvectorscale** — index SSD-based + quantization; tiết kiệm RAM, dim tới 16000.
- **Cosine / L2 / inner product** — 3 distance metric; toán tử `<=>` / `<->` / `<#>`.
- **ef_search / ef_construction / m** — tham số tune HNSW (recall vs speed vs build cost).
- **lists / probes** — tham số tune IVFFlat.
- **Iterative index scan** — chống overfiltering khi combine ANN + WHERE (pgvector 0.8+).
- **halfvec / binary quantization / Matryoshka** — kỹ thuật nén vector cắt RAM/chi phí.
- **RAG** — Retrieval-Augmented Generation; vector search là bước "retrieval".

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Vector search = tìm hàng xóm gần nhất trên "bản đồ ý nghĩa"; distance nhỏ = giống nghĩa.
2. pgvector chỉ *lưu & tìm* vector; embedding do model AI riêng sinh ra.
3. Mặc định là **exact search (100% recall)**; index (HNSW/IVFFlat) đổi recall lấy tốc độ.
4. **HNSW = default**: query ~O(log n), recall cao, đổi lại build chậm + tốn RAM.
5. Ops class trong index PHẢI khớp toán tử query (`vector_cosine_ops` ↔ `<=>`).
6. Siêu năng lực của pgvector: **filter/join vector với business data trong 1 câu SQL** — thứ vector DB chuyên dụng làm tốn 2 round-trip.
7. Text embeddings → dùng **cosine** (hoặc inner product nếu đã normalize).
8. Overfiltering là bẫy kinh điển khi filter chặt → bật iterative scan.
9. Chọn pgvector khi vector là *một feature*; đổi sang vector DB chuyên dụng khi vector là *cả workload* hoặc > ~50M + đa vùng.
10. Ở tầm staff: bottleneck là RAM & recall monitoring; đòn bẩy là quantization + partitioning + two-stage re-rank.

### 5.3. Ideas / mental models để nói trôi chảy

- **"Bản đồ ý nghĩa":** giải thích embedding cho người mới trong 1 câu.
- **"Đường cao tốc nhiều tầng":** giải thích HNSW navigate từ tầng cao xuống thấp.
- **"Chia quận, chỉ vào vài quận gần":** giải thích IVFFlat (và điểm yếu ranh giới quận).
- **"Recall → precision":** two-stage retrieval — thô rồi tinh.
- **"Vector là feature hay là product?":** kim chỉ nam chọn pgvector vs Pinecone.

### 5.4. Code cần thuộc lòng

**(a) Vòng đời tối thiểu — interviewer hay bảo viết tại chỗ:**
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

**(b) Filtered search 1 câu (câu "flex" siêu năng lực pgvector):**
```sql
SELECT id FROM items
WHERE tenant_id = 42 AND in_stock
ORDER BY embedding <=> :query_vec
LIMIT 10;
```

**(c) Exact k-NN cosine bằng NumPy (chứng tỏ hiểu bản chất):**
```python
def cosine_knn(q, corpus, k=10):
    q = q / (q @ q) ** 0.5
    c = corpus / (corpus ** 2).sum(1, keepdims=True) ** 0.5
    dist = 1 - c @ q
    idx = dist.argsort()[:k]
    return idx, dist[idx]
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Khác nhau HNSW vs IVFFlat?"** → HNSW: graph, query ~O(log n), recall cao, build chậm + tốn RAM, thêm data mới tốt, không cần train. IVFFlat: k-means lists, build nhanh, ít RAM, cần data để train, kém với data động. Default nay là HNSW.

2. **[BẪY] "Vì sao kết quả search 'sai' sau khi thêm index?"** → Không sai — ANN đổi recall lấy tốc độ, kết quả *approximate* by design. Tăng `ef_search`/`m` hoặc `probes`, hoặc đo recall để định lượng.

3. **[TRADE-OFF] "Tăng `ef_search` được gì mất gì?"** → recall↑ nhưng latency↑ và RAM/CPU↑. Là núm xoay recall–speed ở query time; tune theo SLA p95.

4. **"Nên dùng cosine hay L2?"** → Với text embeddings, cosine (đo hướng ngữ nghĩa, bỏ độ dài). L2 khi độ lớn vector có nghĩa. Nếu đã normalize length=1 thì inner product ≡ cosine nhưng nhanh hơn.

5. **[BẪY] "Filter chặt trả về ít kết quả hơn LIMIT, vì sao?"** → Overfiltering: index lấy `ef_search` hàng xóm trước, WHERE lọc sau. Fix: `hnsw.iterative_scan = relaxed_order` (pgvector 0.8+).

6. **[SCALE] "1 tỷ vectors, làm sao?"** → Nêu bottleneck RAM (raw vectors + graph), rồi: quantization (halfvec/binary) + two-stage re-rank; partition theo access pattern; DiskANN/pgvectorscale; read replicas; và thành thật cân nhắc vector DB chuyên dụng nếu đa vùng.

7. **[TRADE-OFF] "Khi nào KHÔNG dùng pgvector, chọn Pinecone?"** → Khi vector là workload chính, > ~50M + đa vùng, cần hybrid search built-in, hoặc không chạy Postgres. pgvector thắng ở operational simplicity + join SQL, không phải peak performance.

8. **"Đổi embedding model thì sao?"** → Vector cũ và mới không cùng không gian → phải re-embed toàn bộ; version hóa cột, backfill, blue-green. Đây là quyết định kiến trúc nặng hơn cả chọn index.

### 5.6. One-liner đắt giá (thả đúng lúc để ghi điểm)

- *"pgvector's superpower isn't speed — it's that your vectors live next to your business data, so filtered search is one SQL query, one round-trip, one transaction."*
- *"ANN is a recall-for-speed trade; the real skill is monitoring recall in production, because there's no compile error when relevance quietly degrades."*
- *"Choose pgvector when vector search is a feature, not the product."*
- *"The heaviest architectural decision isn't the index — it's the embedding model, because changing it means re-embedding the entire corpus."*
- *"At a billion vectors the bottleneck is RAM, not CPU — that's when quantization and two-stage re-ranking stop being optional."*

---

### 📌 Ghi chú cuối

- **Kiểm chứng phiên bản khi ôn:** pgvector đang phát triển nhanh. Trước phỏng vấn, xem CHANGELOG mới nhất trên GitHub `pgvector/pgvector` (tính năng như iterative scan, quantization, giới hạn dimension có thể đổi).
- **Thực hành:** dựng Postgres + pgvector bằng Docker (`docker run pgvector/pgvector:pg17`), chạy lại toàn bộ code trong Phần 1–3. Đọc lý thuyết không thay được việc tự `EXPLAIN ANALYZE` một query và thấy "Index Scan using ...hnsw..." bằng mắt.
- **Chủ đề nối tiếp nên học thêm:** RAG end-to-end, hybrid search (kết hợp BM25 full-text của Postgres với vector), reranking models, và evaluation của retrieval (recall@k, MRR, nDCG).
