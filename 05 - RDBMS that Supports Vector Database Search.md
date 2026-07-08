# RDBMS Nào Hỗ Trợ Vector Search? PostgreSQL vs MySQL vs MariaDB — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"RDBMS that support vector database searches"* — Course 3, IBM Vector Database Fundamentals. Bài gốc so sánh hỗ trợ vector giữa **PostgreSQL (pgvector)**, **MySQL HeatWave**, và **MariaDB Vector**. Tôi giảng bám sát rồi đào sâu thành *khung quyết định chọn nền tảng*. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là mảnh *"chọn nhà cho vector"*. Bốn mảnh trước: embed (tạo vector) → index (làm nhanh) → store/query (pgvector) → keyword/FTS. Bài này lùi một bước hỏi câu chiến lược: *nên đặt vector ở RDBMS nào — hay ở một vector DB riêng?*
>
> **⚠️ Đính chính & cập nhật tới 2026 (bài gốc nhiều chỗ đã lỗi thời):**
> 1. **MariaDB Vector đã GA** (general availability) trong **11.8 LTS (6/2025)** — bài gốc nói ở thì tương lai ("will store...", "will provide..."), khi đó còn preview.
> 2. **Bài gốc nói MariaDB "lưu primary data và vector data riêng biệt ở một database đặc biệt" — SAI với bản GA.** Thực tế embedding nằm trong **cột `VECTOR` ngay trong bảng**, cùng chỗ với dữ liệu nghiệp vụ, search bằng SQL. Đây là *ưu điểm* (hybrid query 1 câu), không phải tách rời.
> 3. **PlanetScale (Vitess) đã có vector index** từ 3/2025 (dùng thuật toán **SPANN**) — bài gốc nói "sắp ra".
> 4. **Bài gốc gọi "PostgreSQL offers tsvector as a vector data type" gây nhầm.** `tsvector` là cho **full-text search** (lexeme), *khác hẳn* kiểu `vector` (embedding) của pgvector. Đừng lẫn hai thứ này.
> 5. **MySQL:** Community Server (9.0+) có kiểu `VECTOR` + distance function, nhưng **ANN vector index** chủ yếu qua **HeatWave** (managed/proprietary). Đây là điểm khác biệt then chốt so với MariaDB (native, mã nguồn mở).

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Các relational database (PostgreSQL, MySQL, MariaDB) đang thêm khả năng lưu & tìm vector, để bạn **không cần dựng một vector database riêng**. Nhưng chúng làm điều đó theo **ba cách kiến trúc khác nhau** — extension, native type, hay managed service — và lựa chọn đó ảnh hưởng lớn tới chi phí, vận hành, và khóa nhà cung cấp (vendor lock-in).

**Vấn đề nó giải quyết:** Bạn đã có một RDBMS chạy production. Giờ cần semantic search/RAG. Câu hỏi thực tế: *cắm vector vào chính database sẵn có, hay mua/dựng Pinecone/Weaviate riêng?* Bài này cho bạn khung để trả lời, và biết mỗi RDBMS mạnh/yếu chỗ nào.

**Học xong bạn sẽ làm được:**

- Giải thích ba mô hình hỗ trợ vector: **extension** (pgvector), **native** (MariaDB Vector), **managed service** (MySQL HeatWave).
- Viết code lưu & query vector trên cả PostgreSQL, MariaDB, MySQL.
- Phân biệt "bring-your-own-embedding" vs "in-database embedding generation".
- Xây khung quyết định: RDBMS-native vs dedicated vector DB; managed vs self-host.
- Trả lời câu hỏi system design "chọn nền tảng vector cho use case X".

**Mạch basic → staff:**

- 🟢 **Basic:** Vì sao RDBMS thêm vector → ba mô hình → analogy → mental model chung.
- 🟡 **Intermediate:** Từng nền tảng (PostgreSQL/pgvector, MySQL HeatWave + hệ sinh thái, MariaDB Vector) với code hiện hành; BYO-embedding vs in-DB embedding; lỗi thường gặp.
- 🔴 **Advanced:** Kiến trúc dưới nắp capo — extension vs native engine vs managed; thuật toán index mỗi bên (HNSW, SPANN, ScaNN); giới hạn chiều; SIMD; đính chính "lưu riêng biệt".
- 🟣 **Staff:** Khung quyết định RDBMS vs dedicated vector DB; managed vs self-host; lock-in & migration; cost; org impact; câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Vấn đề thực tế: có nên dựng thêm một database chỉ để chứa vector?

Bạn có app chạy trên PostgreSQL/MySQL đã nhiều năm: user, order, product... Giờ sếp muốn thêm "tìm sản phẩm tương tự theo hình ảnh" (cần vector search). Hai hướng:

- **Hướng A:** dựng thêm một **vector database chuyên dụng** (Pinecone, Weaviate, Qdrant, Milvus) — một hệ thống mới phải học, vận hành, đồng bộ dữ liệu qua lại.
- **Hướng B:** thêm khả năng vector vào **chính RDBMS đang chạy** — vector nằm cạnh business data, dùng SQL quen thuộc, không thêm hệ thống.

Bài này về hướng B: các RDBMS lớn giờ đều hỗ trợ vector, chỉ khác *cách* hỗ trợ.

### 1.2. Analogy: nâng cấp căn nhà vs mua nhà thứ hai

- **Dedicated vector DB** = mua thêm một căn nhà thứ hai chỉ để chứa vector. Rộng rãi tối ưu cho vector, nhưng bạn phải đi lại giữa hai nhà (đồng bộ dữ liệu), trả hai tiền điện, trông hai nhà.
- **RDBMS + vector** = cơi nới thêm một phòng trong căn nhà sẵn có. Mọi thứ dưới một mái, đi lại trong nhà (một câu SQL). Đủ tốt cho phần lớn nhu cầu.

Ba cách "cơi nới":

| Mô hình | Ví dụ | Nghĩa là |
|---|---|---|
| **Extension** | PostgreSQL + **pgvector** | cài thêm gói vào server sẵn có → có kiểu `vector` |
| **Native** | **MariaDB Vector** | kiểu `VECTOR` được xây thẳng vào engine, khỏi cài gì |
| **Managed service** | **MySQL HeatWave** | dịch vụ đám mây làm sẵn cả vector *và* sinh embedding |

### 1.3. Định nghĩa thuật ngữ (lần đầu xuất hiện)

- **Vector / embedding:** mảng số biểu diễn dữ liệu phức tạp (ảnh, text, audio) để so sánh theo *độ giống nhau*. (Đã học ở giáo trình embeddings.)
- **Vector similarity search:** tìm vector gần nhất theo distance (cosine, Euclidean/L2, Manhattan, Jaccard...).
- **Extension:** gói cài thêm vào một database để có tính năng mới (pgvector). Cần `CREATE EXTENSION`.
- **Native support:** tính năng được xây thẳng vào lõi database, dùng ngay không cài (MariaDB `VECTOR`).
- **Managed database service:** database chạy & vận hành bởi nhà cung cấp đám mây (MySQL HeatWave của Oracle) — bạn không quản hạ tầng.
- **OLTP (Online Transaction Processing):** xử lý giao dịch thời gian thực (đặt hàng, thanh toán) — thế mạnh truyền thống của RDBMS.
- **In-database embedding:** database *tự* sinh embedding từ text bằng LLM tích hợp (HeatWave GenAI), khác với "bring-your-own" (bạn tự sinh vector bên ngoài rồi nhét vào).

### 1.4. Mental model chung: mọi nền tảng đều làm cùng một việc

Dù là extension, native hay managed, luồng luôn giống nhau:

```
1. Tạo cột kiểu vector (độ dài = số chiều của model)   -> vector(1536) / VECTOR(1536)
2. Sinh embedding (bằng model) rồi INSERT vào cột đó
3. Tạo index (thường HNSW) để tìm nhanh
4. Query: ORDER BY <distance>(cột, query_vector) LIMIT k
```

Học một nền tảng kỹ thì ba nền tảng chỉ khác *cú pháp*, không khác *ý tưởng*. Bài gốc nói đúng: hầu hết RDBMS giờ cung cấp add-on/wrapper để có kiểu vector và tìm theo similarity thay vì tự tính distance bằng tay.

### ✅ Self-check Phần 1

1. Ba mô hình hỗ trợ vector của RDBMS là gì? Cho một ví dụ mỗi loại.
2. Ưu điểm lớn nhất của "cơi nới RDBMS" so với "mua vector DB riêng" là gì?
3. "In-database embedding" khác "bring-your-own embedding" chỗ nào?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. PostgreSQL + pgvector (mô hình extension)

PostgreSQL là RDBMS mã nguồn mở mạnh, dùng bởi Uber, Netflix, Instagram, Spotify (bài gốc nêu). Nó hỗ trợ vector qua **extension pgvector** — cài lên server là có kiểu `vector`.

> **[Đính chính bài gốc]** Bài gốc nói "PostgreSQL offers tsvector as a vector data type". Cẩn thận: `tsvector` là cho **full-text search** (danh sách lexeme — xem giáo trình FTS), **không phải** kiểu để lưu embedding. Kiểu lưu embedding là `vector`, do **pgvector** cung cấp. Hai thứ khác nhau hoàn toàn; bài gốc phân biệt đúng ở câu sau ("pgvector cho phép xử lý vector phi-text") nhưng câu trước dễ gây nhầm.

```sql
-- 1) Thêm extension (chỉ cần một lần mỗi database)
CREATE EXTENSION IF NOT EXISTS vector;

-- 2) Bảng với cột vector; độ dài (512) = số chiều model tạo embedding
CREATE TABLE items (
    id        bigserial PRIMARY KEY,
    content   text,
    embedding vector(512)
);

-- 3) Index HNSW (khớp ops class với toán tử query)
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);

-- 4) Query k gần nhất
SELECT id, content
FROM items
ORDER BY embedding <=> '[...512 số...]'
LIMIT 5;
```

**Đặc điểm:** BYO-embedding (pgvector không sinh vector); hybrid query mạnh (join/filter với business data trong 1 câu SQL); hệ sinh thái managed rộng (Supabase, Neon, RDS/Aurora). (Chi tiết ở giáo trình pgvector.)

### 2.2. MySQL & MySQL HeatWave (mô hình managed service)

Bài gốc tập trung vào **MySQL HeatWave** — dịch vụ *fully managed* của Oracle, gộp transaction (OLTP), analytics, và machine learning trong một service. Điểm khác biệt lớn:

- **In-database embedding (HeatWave GenAI):** HeatWave có thể **tự sinh embedding real-time** từ text bằng LLM tích hợp — bạn không cần gọi model ngoài. Đây là differentiator so với pgvector/MariaDB (vốn BYO-embedding).
- Chạy trên **OCI (Oracle Cloud) và AWS**; có bản dùng thử giới hạn.
- Vì cần tài nguyên lớn cho scalability & throughput → hợp môi trường cloud.

```sql
-- Ví dụ (bám bài gốc): bảng T với cột val lưu vector 5 phần tử
CREATE TABLE t (
    id  INT PRIMARY KEY,
    val VECTOR(5)
);

-- Chèn vector (MySQL 9.x dùng STRING_TO_VECTOR để parse text -> vector)
INSERT INTO t (id, val) VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));

-- Đo distance (ANN index tối ưu có ở HeatWave)
SELECT id FROM t
ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE')
LIMIT 5;
```

> **[MỞ RỘNG — cập nhật]** Cảnh báo về MySQL: **Community Server (9.0+) có kiểu `VECTOR` + distance function, nhưng ANN vector *index* chủ yếu qua HeatWave** (managed, proprietary). Ngoài ra hệ sinh thái MySQL phân mảnh: **PlanetScale/Vitess** đã có vector index (3/2025, thuật toán **SPANN**), **Google Cloud SQL for MySQL** (2024, thuật toán **ScaNN**), **TiDB** (beta). Amazon RDS for MySQL chưa có vector index (nhưng RDS for MariaDB 11.8 thì có). Bài gốc nói PlanetScale "sắp giới thiệu vector" — nay đã có.

### 2.3. MariaDB Vector (mô hình native)

MariaDB Server là fork của MySQL, do chính những người tạo MySQL lập ra. **MariaDB Vector** đưa vector similarity search **thẳng vào lõi** — không extension, không plugin.

> **[Đính chính quan trọng]** Bài gốc nói MariaDB "lưu primary data và vectorized data *riêng biệt* ở một database *đặc biệt*". **Bản GA KHÔNG như vậy.** Embedding được lưu trong **cột `VECTOR(n)` ngay trong bảng bình thường**, cạnh dữ liệu nghiệp vụ, search bằng SQL — đúng triết lý "một database cho tất cả". Đây là *ưu điểm*, không phải tách rời. (Có thể mô tả cũ ứng với bản preview/thiết kế sớm, nhưng nay đã khác.)

```sql
-- Native: KHÔNG cần CREATE EXTENSION / INSTALL PLUGIN
CREATE TABLE chunks (
    id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    content   TEXT NOT NULL,
    embedding VECTOR(1536) NOT NULL,
    VECTOR INDEX (embedding) DISTANCE=cosine       -- index HNSW, chọn metric ngay
) ENGINE=InnoDB;

-- Chèn: VEC_FromText parse chuỗi -> vector
INSERT INTO chunks (content, embedding)
VALUES ('MariaDB lưu embedding native', VEC_FromText('[...1536 số...]'));

-- Query: BẮT BUỘC có ORDER BY VEC_DISTANCE_*() + LIMIT thì index mới được dùng
SELECT id, content
FROM chunks
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 3;
```

**Đặc điểm (cập nhật 2026):** GA từ **11.8 LTS**; nằm trong **Community Server (miễn phí)**, không chỉ enterprise — khác MySQL (index qua HeatWave proprietary); dùng **HNSW**; hỗ trợ **cosine & Euclidean**; chiều tới **16.383**; có trên **Amazon RDS for MariaDB 11.8**; tối ưu **SIMD** (AVX2/AVX512/NEON); **không** tự sinh embedding (BYO như pgvector); tích hợp LangChain/LlamaIndex/Spring AI.

### 2.4. Bảng so sánh nhanh ba nền tảng

| | PostgreSQL + pgvector | MySQL HeatWave | MariaDB Vector |
|---|---|---|---|
| Mô hình | Extension | Managed service | **Native** (trong lõi) |
| Cần cài thêm? | `CREATE EXTENSION vector` | Không (managed) | **Không** (built-in) |
| Mã nguồn mở & miễn phí? | Có | Không (proprietary/cloud) | **Có** (Community) |
| Sinh embedding trong DB? | Không (BYO) | **Có (HeatWave GenAI)** | Không (BYO) |
| Index | HNSW, IVFFlat, (DiskANN) | HNSW (HeatWave) | HNSW |
| Distance | cosine/L2/ip | cosine/L2... | cosine/L2 |
| Self-host được? | **Có** | Không (cloud-only) | **Có** |
| Hybrid query (vector + SQL) | **Có, 1 câu** | Có | **Có, 1 câu** |

### 2.5. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — MariaDB: quên `LIMIT` → index không được dùng.** Vector index của MariaDB chỉ kích hoạt khi có **cả** `ORDER BY VEC_DISTANCE_*()` **và** `LIMIT`. Thiếu `LIMIT` → full scan. (pgvector cũng ưa có `LIMIT` để tối ưu.)

**Lỗi 2 — Trộn embedding từ hai model khác nhau trong cùng cột.** Vector từ model A (512 chiều) và model B (1536 chiều) không cùng không gian → distance vô nghĩa, mà lệch chiều còn gây lỗi insert. Toàn cột phải cùng model + version. (Đúng cho cả ba nền tảng — nối lại bài embeddings.)

**Lỗi 3 — Nhầm `tsvector` (Postgres FTS) với `vector` (embedding).** Muốn semantic search phải dùng pgvector `vector`, không phải `tsvector`. `tsvector` là keyword/lexeme (FTS). Nhiều người đọc lướt bài gốc rồi tưởng Postgres đã sẵn vector search qua tsvector — không phải.

### ✅ Self-check Phần 2

1. Khác biệt then chốt giữa MySQL HeatWave và MariaDB Vector về "sinh embedding" và "mã nguồn mở" là gì?
2. Trên MariaDB, điều kiện để vector index được dùng khi query?
3. `tsvector` và `vector` trong PostgreSQL — cái nào cho semantic search?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Ba mô hình kiến trúc — đánh đổi dưới nắp capo

**Extension (pgvector):**
- *Ưu:* linh hoạt tối đa — pgvector là mã nguồn mở, cập nhật độc lập với Postgres core; nhiều index (HNSW, IVFFlat, DiskANN qua pgvectorscale); hệ sinh thái managed rộng.
- *Nhược:* phải cài & version pgvector; tính năng phụ thuộc phiên bản extension; một số managed provider khóa version cũ.

**Native (MariaDB):**
- *Ưu:* không cài gì (giảm ma sát vận hành); tối ưu sâu ở tầng engine (SIMD AVX/NEON); tính năng đi cùng release database.
- *Nhược:* bị buộc vào nhịp release của database; ít "món" hơn extension (vd chỉ HNSW, cosine/L2).

**Managed (HeatWave):**
- *Ưu:* không vận hành hạ tầng; **in-database embedding** (khỏi gọi model ngoài); gộp OLTP + analytics + ML.
- *Nhược:* **vendor lock-in** (chạy trên OCI/AWS, proprietary); chi phí theo dịch vụ; không self-host, khó rời đi.

### 3.2. Thuật toán index — không phải ai cũng HNSW

Bài gốc chỉ nhắc IVFFlat/HNSW (của pgvector). Ở tầm landscape 2026, các nền tảng dùng thuật toán khác nhau:

| Nền tảng | Thuật toán ANN |
|---|---|
| pgvector, MariaDB, phần lớn MySQL forks | **HNSW** (graph nhiều tầng) |
| PlanetScale (Vitess) | **SPANN** (Space-Partitioned ANN — lai cụm + graph, hợp disk) |
| Google Cloud SQL for MySQL | **ScaNN** (Google — quantization + phân vùng) |
| Timescale pgvectorscale | **DiskANN** (SSD-based) |

Ý nghĩa staff: "vector search" không phải một thứ đồng nhất — thuật toán index quyết định trade-off memory/disk/recall/tốc độ. HNSW (in-RAM, recall cao) vs SPANN/DiskANN (disk-friendly, tiết kiệm RAM ở quy mô lớn). (Cơ chế HNSW/IVFFlat: xem giáo trình indexing.)

### 3.3. Giới hạn chiều & SIMD — chi tiết kỹ thuật hay bị bỏ qua

- **Giới hạn chiều:** pgvector HNSW/IVFFlat index tối đa **2000 chiều** (dùng `halfvec` cho cao hơn); MariaDB `VECTOR` tới **16.383 chiều**. Với model 3072 chiều (OpenAI 3-large), khác biệt này quan trọng.
- **SIMD:** MariaDB Vector tự dùng lệnh vector hóa phần cứng (AVX2/AVX512 trên Intel, NEON trên ARM) để tính distance nhanh — không cần cấu hình. Đây là loại tối ưu low-level mà một số extension chưa có mặc định.
- **Hệ quả:** cùng "hỗ trợ vector" nhưng chi tiết engine (giới hạn chiều, SIMD, thuật toán) tạo ra khác biệt hiệu năng thật.

### 3.4. Đính chính "lưu riêng biệt" & vì sao "cùng bảng" là ưu điểm

Bài gốc mô tả MariaDB "lưu vector ở database riêng". Thực tế bản GA lưu **cùng bảng** — và đó là điểm mạnh: bạn chạy được **hybrid query** một câu:

```sql
-- Sản phẩm giống nhất, NHƯNG còn hàng và dưới 50$ — một câu, một transaction
SELECT id, name FROM products
WHERE in_stock = 1 AND price < 50
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 10;
```

Nếu tách vector sang hệ riêng (dedicated vector DB), bạn phải: query vector → nhận IDs → query lại DB nghiệp vụ để lọc giá/tồn kho → 2 hệ, 2 round-trip, đồng bộ phức tạp. "Vector cùng bảng với business data" chính là lý do RDBMS-native hấp dẫn.

### 3.5. Edge cases

- **Đồng bộ khi tách hệ:** nếu buộc phải dùng vector DB riêng, mỗi create/update/delete phải sync sang hệ vector → nguồn bug kinh điển (dữ liệu lệch nhau).
- **Distance metric khác nhau giữa nền tảng:** cùng "cosine" nhưng có nền tảng trả *distance* (nhỏ = gần), có nơi trả *similarity* (lớn = gần) → đọc kỹ tài liệu kẻo `ORDER BY` ngược.
- **Chiều vượt giới hạn index:** insert được (lưu) nhưng không index được → âm thầm seq scan chậm.
- **Managed lock-in:** dữ liệu/embedding ở HeatWave khó bê nguyên sang nơi khác → tính chi phí rời đi trước khi cam kết.

### ✅ Self-check Phần 3

1. Nêu một ưu và một nhược của mỗi mô hình: extension, native, managed.
2. Không phải nền tảng nào cũng HNSW — kể 2 thuật toán ANN khác và nền tảng dùng chúng.
3. Vì sao "vector cùng bảng với business data" tốt hơn "tách hệ riêng" cho hybrid query?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Khung quyết định: RDBMS-native vs Dedicated Vector DB

Đây là quyết định kiến trúc thật mà bài gốc không chạm tới. Bài gốc chỉ liệt kê RDBMS; staff phải biết *cả* phương án dedicated (Pinecone, Weaviate, Qdrant, Milvus) và khi nào chọn cái nào.

**Chọn RDBMS-native (pgvector/MariaDB/HeatWave) khi:**
- Vector là *một tính năng trong* app đã chạy RDBMS.
- Cần hybrid query (vector + filter/join business data) trong một transaction ACID.
- < ~10–50 triệu vector (bao trùm đa số B2B RAG/search).
- Muốn ít hệ thống, ít chi phí, đội hiện tại tự vận hành được.

**Chọn Dedicated Vector DB khi:**
- Vector search **là sản phẩm chính**, không phải phụ.
- > ~50 triệu–hàng tỷ vector, cần globally distributed multi-region.
- Cần tính năng chuyên sâu: hybrid dense+sparse built-in, multi-tenancy nâng cao, filtering phức tạp ở quy mô lớn.
- Tổ chức không dùng RDBMS phù hợp, hoặc muốn tách hoàn toàn workload vector.

> **One-liner staff:** *"Đặt vector cạnh business data (RDBMS-native) thắng ở operational simplicity; tách ra vector DB riêng thắng ở peak scale. Chọn theo 'vector là feature hay là product'."*

### 4.2. Managed vs Self-host — trục quyết định thứ hai

| | Self-host (pgvector, MariaDB Community) | Managed (HeatWave, Supabase, Neon, RDS) |
|---|---|---|
| Vận hành | tự lo (patch, backup, scale) | nhà cung cấp lo |
| Chi phí | rẻ về license, tốn nhân lực | trả theo dịch vụ |
| Kiểm soát/dữ liệu | **toàn quyền, dữ liệu ở nhà** | tiện nhưng dữ liệu ở cloud bên thứ ba |
| Lock-in | thấp (mã nguồn mở) | **cao** (đặc biệt HeatWave proprietary) |
| Tính năng ăn liền | tự lắp | in-DB embedding, auto-scale sẵn |

**Ràng buộc privacy/compliance** đôi khi *ép* self-host (dữ liệu khách hàng không được rời hệ thống) — bất kể tiện lợi của managed. Staff phải nêu ràng buộc này sớm (nối lại luận điểm ở giáo trình embeddings).

### 4.3. Lock-in & migration — chi phí ẩn lớn nhất

- **In-database embedding (HeatWave GenAI) tiện nhưng khóa chặt:** embedding sinh bởi model của Oracle → đổi sang nền tảng khác phải re-embed toàn bộ bằng model khác (vector không tương thích không gian). BYO-embedding (pgvector/MariaDB) linh hoạt hơn: bạn kiểm soát model.
- **Đổi RDBMS = migration nặng:** không chỉ chuyển data mà cả cú pháp (`<=>` của pgvector vs `VEC_DISTANCE_COSINE` của MariaDB vs `DISTANCE()` của MySQL), index, và có thể phải re-embed.
- **Chiến lược:** ưu tiên chuẩn mở & BYO-embedding để giữ đường lui; version hóa embedding; tránh phụ thuộc tính năng độc quyền của một vendor nếu tính di động quan trọng.

### 4.4. Cost, reliability, monitoring

- **Cost drivers:** hạ tầng (RAM cho HNSW in-memory), phí managed service theo dùng, phí sinh embedding (nếu in-DB thì gộp vào service), re-embed khi đổi model.
- **Reliability:** RDBMS-native thừa hưởng HA/backup/replication trưởng thành của database mẹ — lợi thế lớn so với vector DB non trẻ. "Một hệ để trông" thay vì hai.
- **Monitoring:** recall@k (so exact vs index), p95/p99 latency, index size/RAM, và — nếu tách hệ — độ trễ/độ lệch đồng bộ giữa RDBMS và vector DB.

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Chúng ta có thể thêm tìm-kiếm-theo-nghĩa mà *không dựng hệ thống mới*: cắm vào database sẵn có (Postgres/MariaDB). Đội hiện tại vận hành được, rủi ro và chi phí thấp, dữ liệu ở nguyên một chỗ. Nếu chọn dịch vụ managed như HeatWave thì nhanh và có sẵn AI, nhưng bị khóa vào một nhà cung cấp và dữ liệu ra cloud của họ. Nếu sau này vector search thành workload khổng lồ, ta cân nhắc hệ chuyên dụng — nhưng đó là bài toán của thành công." → framing bằng **rủi ro, chi phí, lock-in, privacy**.
- **Team topology:** RDBMS-native nghĩa là *một* đội database lo tất; tách vector DB riêng thường kéo theo nhu cầu kỹ năng/on-call mới. Đây là lập luận tổ chức, không chỉ kỹ thuật.
- **Chuẩn hóa toàn công ty:** nếu nhiều team đều cần vector, chọn *một* mô hình (vd pgvector cho mọi service Postgres) giảm phân mảnh hơn là mỗi team một vector DB.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Công ty bạn chạy PostgreSQL cho core app. Cần thêm semantic search cho 20 triệu tài liệu, dữ liệu khách hàng nhạy cảm (không được gửi ra API ngoài), team backend nhỏ. Chọn nền tảng vector thế nào?"**

**Khung trả lời staff (nói to trade-off):**

1. **Clarify:** QPS? recall target? tăng trưởng dữ liệu? ràng buộc compliance cụ thể? ngân sách?
2. **Loại phương án theo ràng buộc:** dữ liệu nhạy cảm + team nhỏ → **loại managed proprietary (HeatWave)** và **loại API embedding ngoài** → nghiêng **self-host pgvector** + model embedding self-host (giữ dữ liệu ở nhà, tận dụng Postgres sẵn có, team đã biết).
3. **Vì sao không dedicated vector DB:** 20M < ngưỡng cần Pinecone/Milvus; vector là feature không phải product; tránh thêm hệ thống + đồng bộ; hybrid query (filter theo tenant/quyền) làm ngay trong SQL.
4. **Kiến trúc:** cột `vector` + HNSW; embedding sinh bởi model self-host (BGE-M3/MiniLM); hybrid với FTS nếu cần keyword; partition theo tenant.
5. **Migration/lock-in:** BYO-embedding + chuẩn mở → giữ đường lui; version hóa embedding.
6. **Khi nào đổi ý:** nếu tăng tới hàng trăm triệu + đa vùng → lúc đó cân nhắc tách dedicated; nêu rõ *ngưỡng* kích hoạt quyết định đó.
7. **Chốt:** "Tôi chọn RDBMS-native self-host vì ràng buộc *privacy* và *quy mô hiện tại*, không phải vì nó luôn tốt nhất — và tôi biết ngưỡng nào sẽ khiến tôi đổi." → **biết lý do & giới hạn của lựa chọn = dấu hiệu staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **pgvector** — extension thêm kiểu `vector` cho PostgreSQL.
- **MariaDB Vector** — hỗ trợ vector **native** (kiểu `VECTOR`), GA từ 11.8 LTS, mã nguồn mở.
- **MySQL HeatWave** — dịch vụ managed của Oracle; có **in-database embedding** (HeatWave GenAI).
- **Extension / Native / Managed** — ba mô hình hỗ trợ vector của RDBMS.
- **BYO-embedding** — tự sinh vector ngoài rồi nhét vào (pgvector, MariaDB).
- **In-database embedding** — DB tự sinh vector bằng LLM tích hợp (HeatWave).
- **Hybrid query** — vector search + filter/join SQL trong một câu, một transaction.
- **HNSW / SPANN / ScaNN / DiskANN** — các thuật toán ANN; nền tảng khác nhau dùng khác nhau.
- **Dedicated vector DB** — Pinecone/Weaviate/Qdrant/Milvus; hệ chuyên cho vector.
- **Vendor lock-in** — bị khóa vào một nhà cung cấp (đặc biệt managed proprietary).
- **Distance metrics** — cosine, Euclidean (L2), Manhattan, Jaccard.
- **`VEC_FromText` / `VEC_DISTANCE_COSINE`** (MariaDB); **`STRING_TO_VECTOR` / `DISTANCE`** (MySQL); **`<=>`** (pgvector).
- **tsvector ≠ vector** — FTS lexeme vs embedding; đừng nhầm.

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. RDBMS thêm vector để bạn **khỏi dựng vector DB riêng**; ba cách: extension / native / managed.
2. **pgvector = extension, MariaDB = native, HeatWave = managed** (và HeatWave tự sinh embedding).
3. Siêu năng lực chung của RDBMS-native: **hybrid query** vector + business data trong 1 câu SQL.
4. MariaDB Vector **GA, mã nguồn mở, cùng bảng** (không lưu riêng như mô tả cũ); MySQL ANN index chủ yếu qua HeatWave (proprietary).
5. Không phải ai cũng HNSW: PlanetScale dùng SPANN, Cloud SQL dùng ScaNN.
6. **BYO-embedding linh hoạt, in-DB embedding tiện nhưng khóa chặt** hơn.
7. RDBMS-native thắng operational simplicity; dedicated vector DB thắng peak scale.
8. Trục quyết định: (a) feature hay product, (b) quy mô, (c) managed vs self-host, (d) privacy/lock-in.
9. Đổi nền tảng/model = re-embed + đổi cú pháp → chi phí ẩn lớn; ưu tiên chuẩn mở để giữ đường lui.
10. `tsvector` (FTS) khác `vector` (embedding) — semantic search cần cái sau.

### 5.3. Ideas / mental models

- **"Cơi nới phòng vs mua nhà thứ hai":** RDBMS-native vs dedicated vector DB.
- **"Feature hay product?":** kim chỉ nam chọn RDBMS vs Pinecone.
- **"Extension = cài thêm, native = xây sẵn, managed = thuê trọn":** ba mô hình.
- **"Tiện đi kèm xích":** in-DB embedding (HeatWave) đánh đổi lock-in.
- **"Một hệ để trông, không phải hai":** lợi thế vận hành của native.

### 5.4. Code cần thuộc lòng

**(a) pgvector (extension):**
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE t (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON t USING hnsw (embedding vector_cosine_ops);
SELECT id FROM t ORDER BY embedding <=> '[...]' LIMIT 5;
```

**(b) MariaDB (native):**
```sql
CREATE TABLE t (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  embedding VECTOR(1536) NOT NULL,
  VECTOR INDEX (embedding) DISTANCE=cosine
) ENGINE=InnoDB;
SELECT id FROM t
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]')) LIMIT 5;  -- LIMIT bắt buộc
```

**(c) MySQL (9.x / HeatWave):**
```sql
CREATE TABLE t (id INT PRIMARY KEY, val VECTOR(5));
INSERT INTO t VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"PostgreSQL, MySQL, MariaDB hỗ trợ vector khác nhau thế nào?"** → Postgres: extension (pgvector). MariaDB: native (`VECTOR`, GA 11.8, mã nguồn mở). MySQL: Community có kiểu VECTOR nhưng ANN index chủ yếu qua HeatWave (managed, có in-DB embedding).

2. **[BẪY] "PostgreSQL đã có vector search sẵn qua tsvector đúng không?"** → Không. `tsvector` là full-text search (lexeme). Semantic vector search cần extension **pgvector** (kiểu `vector`).

3. **[TRADE-OFF] "RDBMS-native hay dedicated vector DB (Pinecone)?"** → Native khi vector là feature, < ~50M, cần hybrid SQL, ít hệ thống. Dedicated khi vector là product, quy mô rất lớn/đa vùng, cần tính năng chuyên sâu.

4. **"HeatWave khác pgvector/MariaDB chỗ nào?"** → HeatWave managed + **tự sinh embedding** trong DB; nhưng proprietary, cloud-only, lock-in cao. pgvector/MariaDB self-host được, mã nguồn mở, BYO-embedding.

5. **[TRADE-OFF] "In-database embedding có gì hay/dở?"** → Hay: tiện, khỏi model ngoài. Dở: khóa vào model của vendor, đổi nền tảng phải re-embed; kém linh hoạt hơn BYO.

6. **[BẪY] "MariaDB lưu vector ở database riêng phải không?"** → Không (với bản GA). Embedding nằm trong cột `VECTOR` **cùng bảng** với business data → hybrid query 1 câu. (Mô tả "lưu riêng" là cũ.)

7. **[SCALE] "Công ty dùng Postgres, cần vector cho 20M docs, dữ liệu nhạy cảm?"** → Self-host pgvector + embedding self-host: tận dụng DB sẵn có, dữ liệu ở nhà, tránh lock-in, đủ cho 20M. Nêu ngưỡng sẽ khiến đổi sang dedicated.

8. **"Không phải nền tảng nào cũng HNSW — đúng không?"** → Đúng. pgvector/MariaDB/nhiều MySQL fork dùng HNSW; PlanetScale dùng SPANN; Cloud SQL dùng ScaNN; pgvectorscale dùng DiskANN.

### 5.6. One-liner đắt giá

- *"Every major RDBMS now speaks vector — the question isn't 'can it', it's 'extension, native, or managed' and what each costs you in lock-in."*
- *"RDBMS-native wins on operational simplicity; dedicated vector DBs win on peak scale. Pick by whether vector search is a feature or the product."*
- *"HeatWave generates embeddings for you — convenient, but that convenience is a leash: switch platforms and you re-embed everything."*
- *"MariaDB Vector keeps embeddings in a column next to your data, so 'find similar products under $50 in stock' is one SQL statement, one transaction."*
- *"tsvector is keywords, vector is meaning — confusing them is the fastest way to fail a vector-search interview."*
- *"Sometimes privacy picks the platform: if the data can't leave the building, managed cloud services are off the table before performance even matters."*

---

### 📌 Ghi chú cuối

- **Đính chính để nhớ đúng:** MariaDB Vector **GA (11.8 LTS), lưu cùng bảng** (không riêng biệt); PlanetScale đã có vector (SPANN); `tsvector` (FTS) ≠ `vector` (embedding); MySQL ANN index chủ yếu qua HeatWave.
- **Kiểm chứng khi ôn:** mảng này đổi nhanh — trước phỏng vấn xem tài liệu chính thức của pgvector, MariaDB (11.8+), MySQL/HeatWave để cập nhật cú pháp & tính năng.
- **Thực hành:** dựng thử cả pgvector (Docker `pgvector/pgvector`) và MariaDB 11.8 (Docker), tạo cùng một bảng vector trên mỗi bên, chạy cùng một query, so cú pháp `<=>` vs `VEC_DISTANCE_COSINE`. Cảm nhận "cùng ý tưởng, khác cú pháp".
- **Nối mạch series (đủ 5 mảnh):** embed (tạo vector) → index (làm nhanh) → store/query (pgvector) → keyword/FTS → **chọn nền tảng (bài này)**. Toàn bộ để bạn build & vận hành semantic search / RAG có hiểu biết.
- **Học tiếp:** benchmark có hệ thống giữa các nền tảng (ann-benchmarks, Big Vector Search Benchmark), chiến lược migration giữa vector stores, và hybrid dense+sparse (BGE-M3, Cohere embed-v4).
