# RDBMS Nào Hỗ Trợ Vector? PostgreSQL vs MySQL vs MariaDB — Chain gối đầu

> Vị trí series (mảnh 5/5 "chọn nhà cho vector"): embed → index → store/query (pgvector) → keyword/FTS → **chọn nền tảng (bài này)**. Bài lùi một bước hỏi câu chiến lược: đặt vector ở RDBMS nào — hay ở vector DB riêng?
>
> **Đính chính bài gốc (đã lỗi thời):** (1) MariaDB Vector đã **GA** trong **11.8 LTS (6/2025)** — bài gốc nói thì tương lai. (2) MariaDB KHÔNG "lưu vector riêng biệt ở database đặc biệt" — embedding nằm trong cột `VECTOR` **cùng bảng** với business data. (3) PlanetScale (Vitess) đã có vector index từ 3/2025 (thuật toán **SPANN**). (4) "PostgreSQL offers tsvector as a vector type" gây nhầm — `tsvector` là **FTS (lexeme)**, khác hẳn kiểu `vector` (embedding) của pgvector. (5) MySQL Community (9.0+) có kiểu `VECTOR` + distance function nhưng **ANN vector index chủ yếu qua HeatWave** (managed/proprietary).

---

## CHAIN — Vấn đề & analogy

đã có app chạy PostgreSQL/MySQL nhiều năm, giờ cần vector search => hai hướng: A dựng vector DB chuyên dụng (Pinecone/Weaviate/Qdrant/Milvus), B thêm vector vào chính RDBMS đang chạy => hướng A = mua nhà thứ hai chỉ để chứa vector (đi lại giữa hai nhà = đồng bộ dữ liệu, trả hai tiền điện, trông hai nhà) => hướng B = cơi nới thêm một phòng trong nhà sẵn có (mọi thứ dưới một mái, đi lại bằng một câu SQL) => RDBMS lớn giờ đều hỗ trợ vector, chỉ khác *cách* hỗ trợ

## CHAIN — Ba mô hình hỗ trợ vector

RDBMS "cơi nới" vector theo ba cách khác nhau => mô hình 1 — **extension**: cài thêm gói vào server sẵn có (PostgreSQL + pgvector, cần `CREATE EXTENSION`) => mô hình 2 — **native**: kiểu `VECTOR` xây thẳng vào engine, khỏi cài gì (MariaDB Vector) => mô hình 3 — **managed service**: dịch vụ đám mây làm sẵn cả vector *và* sinh embedding (MySQL HeatWave) => ba mô hình khác nhau ảnh hưởng lớn tới chi phí, vận hành, và vendor lock-in

## CHAIN — Mental model chung (4 bước)

dù extension/native/managed, luồng luôn giống nhau => bước 1: tạo cột kiểu vector (độ dài = số chiều model) — `vector(1536)` / `VECTOR(1536)` => bước 2: sinh embedding bằng model rồi `INSERT` vào cột đó => bước 3: tạo index (thường HNSW) để tìm nhanh => bước 4: query `ORDER BY <distance>(cột, query_vector) LIMIT k` => nên học một nền tảng kỹ thì ba nền tảng chỉ khác *cú pháp*, không khác *ý tưởng*

## CHAIN — BYO vs in-database embedding

mọi nền tảng đều cần embedding, nhưng *nguồn sinh* khác nhau => **bring-your-own (BYO)**: bạn tự sinh vector bằng model bên ngoài rồi nhét vào (pgvector, MariaDB) => **in-database embedding**: database tự sinh embedding từ text bằng LLM tích hợp (HeatWave GenAI) => in-DB tiện (khỏi model ngoài) nhưng khóa vào model của vendor => vì khóa model, đổi nền tảng phải re-embed toàn bộ bằng model khác => nên BYO linh hoạt hơn: bạn kiểm soát model

## CHAIN — pgvector (extension)

PostgreSQL hỗ trợ vector qua **extension pgvector** — cài lên server là có kiểu `vector` => CẨN THẬN: `tsvector` là full-text search (lexeme), KHÁC hẳn kiểu `vector` (embedding) — đừng nhầm hai thứ => pgvector là **BYO-embedding** (không tự sinh vector) => điểm mạnh: **hybrid query** (join/filter với business data trong 1 câu SQL) => và hệ sinh thái managed rộng (Supabase, Neon, RDS/Aurora) => nhiều "món" index nhất: HNSW, IVFFlat, DiskANN (qua pgvectorscale)

## CHAIN — MySQL HeatWave (managed)

MySQL HeatWave là dịch vụ *fully managed* của Oracle, gộp OLTP + analytics + ML trong một service => differentiator: **in-database embedding (HeatWave GenAI)** — tự sinh embedding real-time từ text, khỏi gọi model ngoài => chạy trên OCI (Oracle Cloud) và AWS, cloud-only => còn MySQL Community Server (9.0+) có kiểu `VECTOR` + distance function (`STRING_TO_VECTOR`, `DISTANCE`) nhưng ANN vector *index* chủ yếu qua HeatWave (managed, proprietary) => hệ sinh thái MySQL phân mảnh: PlanetScale/Vitess (SPANN, 3/2025), Cloud SQL (ScaNN), TiDB (beta); RDS for MySQL chưa có vector index

## CHAIN — MariaDB Vector (native) & cùng bảng

MariaDB Server là fork của MySQL, đưa vector similarity search THẲNG vào lõi (không extension/plugin) => đính chính bài gốc: MariaDB KHÔNG lưu vector "riêng biệt ở database đặc biệt" => bản GA lưu embedding trong cột `VECTOR(n)` NGAY trong bảng bình thường, cạnh business data => nhờ cùng bảng, chạy được **hybrid query một câu** (vd "giống nhất NHƯNG còn hàng và dưới 50$") => cú pháp: `VECTOR INDEX ... DISTANCE=cosine`, `VEC_FromText` parse chuỗi, `VEC_DISTANCE_COSINE` khi query => BẮT BUỘC có `ORDER BY VEC_DISTANCE_*()` + `LIMIT` thì index mới được dùng (thiếu `LIMIT` → full scan)

## CHAIN — MariaDB đặc điểm 2026

MariaDB Vector GA từ **11.8 LTS (6/2025)** => nằm trong **Community Server (miễn phí)**, không chỉ enterprise — khác MySQL (index qua HeatWave proprietary) => dùng **HNSW**, hỗ trợ cosine & Euclidean => chiều tới **16.383** (so với pgvector index tối đa 2000 chiều) => tối ưu **SIMD** phần cứng (AVX2/AVX512 trên Intel, NEON trên ARM) tính distance nhanh, không cần cấu hình => không tự sinh embedding (BYO như pgvector); có trên Amazon RDS for MariaDB 11.8; tích hợp LangChain/LlamaIndex/Spring AI

## CHAIN — Đánh đổi 3 mô hình kiến trúc

**extension** (pgvector): ưu linh hoạt tối đa (mã nguồn mở, cập nhật độc lập với core, nhiều index); nhược phải cài & version, một số managed khóa version cũ => **native** (MariaDB): ưu không cài gì (giảm ma sát vận hành), tối ưu SIMD ở tầng engine, tính năng đi cùng release; nhược bị buộc vào nhịp release database, ít "món" hơn (chỉ HNSW, cosine/L2) => **managed** (HeatWave): ưu không vận hành hạ tầng, in-DB embedding, gộp OLTP+analytics+ML; nhược vendor lock-in (OCI/AWS proprietary), chi phí theo dịch vụ, không self-host nên khó rời đi

## CHAIN — Thuật toán index không đồng nhất

"vector search" không phải một thứ đồng nhất — thuật toán index khác nhau giữa các nền tảng => pgvector/MariaDB/phần lớn MySQL forks dùng **HNSW** (graph nhiều tầng) => PlanetScale (Vitess) dùng **SPANN** (Space-Partitioned ANN — lai cụm + graph, hợp disk) => Google Cloud SQL dùng **ScaNN** (quantization + phân vùng); pgvectorscale dùng **DiskANN** (SSD-based) => ý nghĩa: HNSW in-RAM recall cao vs SPANN/DiskANN disk-friendly tiết kiệm RAM ở quy mô lớn => thuật toán index quyết định trade-off memory/disk/recall/tốc độ

## CHAIN — Cùng bảng = ưu điểm hybrid; tách hệ = bug đồng bộ

tách vector sang hệ riêng (dedicated vector DB) thì phải: query vector → nhận IDs → query lại DB nghiệp vụ để lọc giá/tồn kho => hai hệ, hai round-trip, đồng bộ phức tạp => mỗi create/update/delete phải sync sang hệ vector → nguồn bug kinh điển (dữ liệu lệch nhau) => ngược lại, vector cùng bảng với business data cho hybrid query một câu, một transaction => đó chính là lý do RDBMS-native hấp dẫn

## CHAIN — Edge cases nền tảng

edge case 1 — **distance vs similarity**: cùng "cosine" nhưng có nền tảng trả *distance* (nhỏ=gần), nơi khác trả *similarity* (lớn=gần) → đọc kỹ tài liệu kẻo `ORDER BY` ngược => edge case 2 — **chiều vượt giới hạn index**: insert được (lưu) nhưng không index được → âm thầm seq scan chậm => edge case 3 — **trộn hai model khác chiều** trong cùng cột: không cùng không gian → distance vô nghĩa, lệch chiều còn lỗi insert; toàn cột phải cùng model + version => edge case 4 — **managed lock-in**: dữ liệu/embedding ở HeatWave khó bê nguyên sang nơi khác → tính chi phí rời đi trước khi cam kết

## CHAIN — Khung RDBMS-native vs Dedicated

quyết định kiến trúc thật: RDBMS-native (pgvector/MariaDB/HeatWave) hay dedicated vector DB (Pinecone/Weaviate/Qdrant/Milvus) => chọn **RDBMS-native** khi vector là *một tính năng* trong app đã chạy RDBMS => cần hybrid query (vector + filter/join business data) trong một transaction ACID, < ~10–50M vector, muốn ít hệ thống => chọn **dedicated** khi vector search *là sản phẩm chính*, > ~50M–hàng tỷ vector cần multi-region => hoặc cần hybrid dense+sparse built-in / multi-tenancy nâng cao / filtering phức tạp ở quy mô lớn => chốt: RDBMS-native thắng operational simplicity, dedicated thắng peak scale; chọn theo "vector là feature hay là product"

## CHAIN — Managed vs self-host & privacy

trục quyết định thứ hai: **managed** (HeatWave/Supabase/Neon/RDS) vs **self-host** (pgvector/MariaDB Community) => managed: nhà cung cấp lo vận hành + tính năng ăn liền (in-DB embedding, auto-scale) nhưng lock-in cao, dữ liệu ở cloud bên thứ ba => self-host: toàn quyền, dữ liệu ở nhà, lock-in thấp (mã nguồn mở), nhưng tự lo patch/backup/scale => ràng buộc privacy/compliance đôi khi *ép* self-host (dữ liệu khách hàng không được rời hệ thống) => bất kể tiện lợi của managed, staff phải nêu ràng buộc này sớm

## CHAIN — Lock-in & migration (chi phí ẩn lớn nhất)

in-database embedding (HeatWave GenAI) tiện nhưng khóa chặt: embedding sinh bởi model của Oracle => đổi sang nền tảng khác phải re-embed toàn bộ bằng model khác (vector không tương thích không gian) => hơn nữa đổi RDBMS = migration nặng: không chỉ data mà cả **cú pháp** (`<=>` pgvector vs `VEC_DISTANCE_COSINE` MariaDB vs `DISTANCE()` MySQL), index, có thể re-embed => chiến lược: ưu tiên chuẩn mở & BYO-embedding để giữ đường lui => version hóa embedding; tránh phụ thuộc tính năng độc quyền của một vendor nếu tính di động quan trọng

## CHAIN — Reliability, cost & tổ chức

cost drivers: RAM cho HNSW in-memory, phí managed theo dùng, phí sinh embedding, re-embed khi đổi model => reliability: RDBMS-native thừa hưởng HA/backup/replication trưởng thành của database mẹ — lợi thế lớn so với vector DB non trẻ ("một hệ để trông" thay vì hai) => nói với PM/sếp bằng **rủi ro/chi phí/lock-in/privacy**: thêm search theo nghĩa mà không dựng hệ thống mới => team topology: RDBMS-native = *một* đội database lo tất; tách vector DB riêng kéo theo kỹ năng/on-call mới => chuẩn hóa toàn công ty: nhiều team cần vector thì chọn *một* mô hình (vd pgvector cho mọi service Postgres) giảm phân mảnh

---

## BẢNG — Ba mô hình "cơi nới"

| Mô hình | Ví dụ | Nghĩa là |
|---|---|---|
| **Extension** | PostgreSQL + pgvector | cài thêm gói vào server sẵn có → có kiểu `vector` |
| **Native** | MariaDB Vector | kiểu `VECTOR` xây thẳng vào engine, khỏi cài gì |
| **Managed service** | MySQL HeatWave | dịch vụ đám mây làm sẵn cả vector *và* sinh embedding |

## BẢNG — So sánh 3 nền tảng

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
| Giới hạn chiều (index) | 2000 (halfvec cao hơn) | — | **16.383** |

## BẢNG — Thuật toán ANN theo nền tảng

| Nền tảng | Thuật toán |
|---|---|
| pgvector, MariaDB, phần lớn MySQL forks | **HNSW** (graph nhiều tầng, in-RAM, recall cao) |
| PlanetScale (Vitess) | **SPANN** (Space-Partitioned ANN — lai cụm + graph, hợp disk) |
| Google Cloud SQL for MySQL | **ScaNN** (Google — quantization + phân vùng) |
| Timescale pgvectorscale | **DiskANN** (SSD-based, tiết kiệm RAM) |

## BẢNG — Managed vs Self-host

| | Self-host (pgvector, MariaDB Community) | Managed (HeatWave, Supabase, Neon, RDS) |
|---|---|---|
| Vận hành | tự lo (patch, backup, scale) | nhà cung cấp lo |
| Chi phí | rẻ về license, tốn nhân lực | trả theo dịch vụ |
| Kiểm soát/dữ liệu | **toàn quyền, dữ liệu ở nhà** | tiện nhưng dữ liệu ở cloud bên thứ ba |
| Lock-in | thấp (mã nguồn mở) | **cao** (đặc biệt HeatWave) |
| Tính năng ăn liền | tự lắp | in-DB embedding, auto-scale sẵn |

Ràng buộc **privacy/compliance** đôi khi ép self-host (dữ liệu khách hàng không được rời hệ thống), bất kể tiện lợi managed.

## BẢNG — Distance metrics

Vector similarity search tìm vector gần nhất theo distance: **cosine**, **Euclidean (L2)**, **Manhattan**, **Jaccard**.

## BẢNG — Code thuộc lòng

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
SELECT id FROM t ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE') LIMIT 5;
```

## BẢNG — Khung system design (Postgres core, 20M docs, dữ liệu nhạy cảm không ra API ngoài, team nhỏ)

1. **Clarify:** QPS? recall target? tăng trưởng dữ liệu? ràng buộc compliance? ngân sách?
2. **Loại theo ràng buộc:** nhạy cảm + team nhỏ → **loại managed proprietary (HeatWave)** và **loại API embedding ngoài** → nghiêng **self-host pgvector** + model embedding self-host.
3. **Vì sao không dedicated:** 20M < ngưỡng cần Pinecone/Milvus; vector là feature không phải product; tránh thêm hệ thống + đồng bộ; hybrid query filter tenant/quyền ngay trong SQL.
4. **Kiến trúc:** cột `vector` + HNSW; embedding self-host (BGE-M3/MiniLM); hybrid với FTS nếu cần keyword; partition theo tenant.
5. **Migration/lock-in:** BYO-embedding + chuẩn mở → giữ đường lui; version hóa embedding.
6. **Khi nào đổi ý:** tăng tới hàng trăm triệu + đa vùng → cân nhắc tách dedicated; nêu rõ *ngưỡng* kích hoạt.
7. **Chốt:** chọn RDBMS-native self-host vì ràng buộc *privacy* và *quy mô hiện tại*, và biết ngưỡng nào sẽ khiến đổi = dấu hiệu staff.

## BẢNG — Mental models

- **"Cơi nới phòng vs mua nhà thứ hai"** → RDBMS-native vs dedicated vector DB.
- **"Feature hay product?"** → kim chỉ nam chọn RDBMS vs Pinecone.
- **"Extension = cài thêm, native = xây sẵn, managed = thuê trọn"** → ba mô hình.
- **"Tiện đi kèm xích"** → in-DB embedding (HeatWave) đánh đổi lock-in.
- **"Một hệ để trông, không phải hai"** → lợi thế vận hành của native.

---

## TỪ KHÓA MỒI

- Vấn đề & analogy → **"cơi nới vs nhà thứ hai"**
- Ba mô hình → **"extension / native / managed"**
- Mental model 4 bước → **"khác cú pháp không khác ý tưởng"**
- BYO vs in-DB embedding → **"HeatWave GenAI"**
- pgvector → **"tsvector ≠ vector"**
- MySQL HeatWave → **"in-database embedding"**
- MariaDB native → **"cùng bảng, GA 11.8"**
- MariaDB đặc điểm → **"16.383 chiều + SIMD"**
- Đánh đổi 3 mô hình → **"lock-in không self-host"**
- Thuật toán index → **"không phải ai cũng HNSW"**
- Cùng bảng = ưu điểm → **"một transaction"**
- Edge cases → **"distance vs similarity"**
- RDBMS-native vs dedicated → **"feature hay product"**
- Managed vs self-host → **"privacy ép self-host"**
- Lock-in & migration → **"đổi cú pháp + re-embed"**
- Reliability & tổ chức → **"một hệ để trông"**

---

## ĐÃ PHỦ

**Vấn đề & mô hình:** dựng vector DB riêng vs cơi nới RDBMS; analogy nhà thứ hai vs cơi phòng; ba mô hình extension (pgvector) / native (MariaDB) / managed (HeatWave); mental model 4 bước (cột vector → embed+insert → index HNSW → ORDER BY distance LIMIT).

**Thuật ngữ:** vector/embedding, vector similarity search (cosine/L2/Manhattan/Jaccard), extension, native support, managed database service, OLTP, in-database embedding vs BYO-embedding.

**pgvector (extension):** CREATE EXTENSION; PostgreSQL dùng bởi Uber/Netflix/Instagram/Spotify; BYO; hybrid query 1 câu; hệ sinh thái Supabase/Neon/RDS/Aurora; nhiều index HNSW/IVFFlat/DiskANN; đính chính tsvector ≠ vector.

**MySQL/HeatWave (managed):** Oracle fully managed gộp OLTP+analytics+ML; in-DB embedding HeatWave GenAI; OCI+AWS cloud-only; STRING_TO_VECTOR/DISTANCE; Community 9.0+ có VECTOR nhưng ANN index chủ yếu qua HeatWave; hệ sinh thái phân mảnh PlanetScale SPANN / Cloud SQL ScaNN / TiDB beta; RDS for MySQL chưa có index.

**MariaDB Vector (native):** fork MySQL, vào lõi không extension/plugin; đính chính lưu cùng bảng (không riêng biệt); cú pháp VECTOR INDEX DISTANCE=cosine / VEC_FromText / VEC_DISTANCE_COSINE; BẮT BUỘC ORDER BY VEC_DISTANCE + LIMIT; GA 11.8 LTS (6/2025); Community miễn phí; HNSW; cosine & Euclidean; chiều tới 16.383; RDS for MariaDB 11.8; SIMD AVX2/AVX512/NEON; BYO; tích hợp LangChain/LlamaIndex/Spring AI.

**So sánh & kiến trúc:** bảng 3 nền tảng đầy đủ; đánh đổi extension/native/managed (ưu-nhược từng cái); thuật toán index không đồng nhất (HNSW/SPANN/ScaNN/DiskANN); giới hạn chiều pgvector 2000 (halfvec) vs MariaDB 16.383; SIMD; cùng bảng = hybrid 1 câu vs tách hệ 2 round-trip+sync bug.

**Edge cases:** distance vs similarity (ORDER BY ngược); chiều vượt giới hạn insert được không index được → seq scan; trộn 2 model khác chiều; managed lock-in khó bê đi.

**Staff:** khung RDBMS-native vs dedicated (feature<50M / product>50M multi-region); managed vs self-host (bảng) + privacy/compliance ép self-host; lock-in & migration (in-DB embedding khóa, đổi RDBMS = data+cú pháp+index+re-embed, ưu tiên chuẩn mở/BYO); cost drivers; reliability thừa hưởng HA/backup/replication ("một hệ để trông"); monitoring recall@k/p95-p99/index size/độ lệch đồng bộ; tổ chức (framing rủi ro/chi phí/lock-in/privacy, team topology một đội, chuẩn hóa một mô hình toàn công ty); khung system design 7 bước.

**Đính chính bài gốc:** MariaDB GA 11.8 + lưu cùng bảng; PlanetScale SPANN; tsvector ≠ vector; MySQL ANN index qua HeatWave.

**Học tiếp (bài nêu):** benchmark có hệ thống (ann-benchmarks, Big Vector Search Benchmark), migration giữa vector stores, hybrid dense+sparse (BGE-M3, Cohere embed-v4).
