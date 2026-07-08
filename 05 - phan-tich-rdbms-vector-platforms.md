# Phân tích chain học tập — RDBMS Nào Hỗ Trợ Vector? PostgreSQL vs MySQL vs MariaDB

> Mục tiêu: tách *suy ra được* (nhóm A) khỏi *phải ghi nhớ* (nhóm B), đánh dấu **⚠** chỗ danh sách/logic nhảy, rút **xương sống**.

Đây là **mảnh 5/5** khép series — bài **chiến lược**: đặt vector ở đâu (RDBMS nào, hay vector DB riêng). Khác các bài trước (cơ chế), bài này **nghiêng mạnh nhóm B**: đặc điểm nền tảng, bảng so sánh, tiêu chí quyết định. Phần A cốt lõi chỉ nằm ở vài mắt xích (cùng bảng → hybrid; in-DB embedding → lock-in; privacy → ép self-host). Nhiều chain là **danh sách tiêu chí** — học như checklist quyết định, đừng ép "vì… nên…".

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Vấn đề & analogy

**A — Nhân quả**
- tách vector sang hệ riêng => phải đồng bộ dữ liệu, trả hai tiền điện, trông hai nhà — *vì* hai kho tách biệt buộc phải nối và giữ đồng bộ.

**B — Ghi nhớ**
- Hai hướng: A = dedicated vector DB (Pinecone/Weaviate/Qdrant/Milvus) = "mua nhà thứ hai"; B = thêm vector vào RDBMS đang chạy = "cơi nới một phòng" — *mental model nền*.

> Chain này chủ yếu nhóm B (analogy + framing). Nắm "cơi nới vs nhà thứ hai" là nắm cả bài.

**Xương sống**: app đã chạy RDBMS cần vector → cơi nới RDBMS (một nhà, một câu SQL) vs mua nhà thứ hai (đồng bộ, hai chi phí).

---

## Chain 2 — Ba mô hình hỗ trợ vector

**A — Nhân quả**
- ba mô hình khác nhau *cách* hỗ trợ => ảnh hưởng lớn tới chi phí, vận hành, vendor lock-in — *vì* cài-thêm / xây-sẵn / thuê-trọn kéo theo hệ quả vận hành khác nhau.

**B — Ghi nhớ**
- **extension**: cài thêm gói (PostgreSQL + pgvector, cần `CREATE EXTENSION`).
- **native**: kiểu `VECTOR` xây thẳng vào engine, khỏi cài gì (MariaDB Vector).
- **managed service**: dịch vụ đám mây làm sẵn cả vector *và* sinh embedding (MySQL HeatWave).

> Chủ yếu nhóm B (định nghĩa 3 mô hình).

**Xương sống**: RDBMS cơi nới vector theo 3 mô hình (extension / native / managed) → khác nhau về chi phí/vận hành/lock-in.

---

## Chain 3 — Mental model chung (4 bước)

**A — Nhân quả**
- luồng luôn giống nhau => học một nền tảng kỹ thì ba nền tảng chỉ khác *cú pháp*, không khác *ý tưởng* — *vì* 4 bước là bất biến, chỉ tên hàm đổi.

**B — Ghi nhớ (4 bước)**
- (1) tạo cột kiểu vector (độ dài = số chiều model): `vector(1536)` / `VECTOR(1536)`.
- (2) sinh embedding rồi `INSERT`.
- (3) tạo index (thường HNSW).
- (4) query `ORDER BY <distance>(cột, query_vector) LIMIT k`.

**Xương sống**: cột vector → embed & insert → index HNSW → ORDER BY distance LIMIT; ba nền tảng chỉ khác cú pháp.

---

## Chain 4 — BYO vs in-database embedding

**A — Nhân quả**
- in-DB embedding dùng model của vendor => khóa vào model đó => đổi nền tảng phải re-embed toàn bộ bằng model khác — *vì* vector cũ/mới không cùng không gian.
- BYO cho bạn kiểm soát model => linh hoạt hơn — *vì* không phụ thuộc model độc quyền.

**B — Ghi nhớ**
- **BYO (bring-your-own)**: tự sinh vector bằng model ngoài rồi nhét vào (pgvector, MariaDB).
- **in-database embedding**: DB tự sinh từ text bằng LLM tích hợp (HeatWave GenAI).

**Xương sống**: BYO (kiểm soát model, linh hoạt) vs in-DB embedding (tiện nhưng khóa vào model vendor → đổi = re-embed).

---

## Chain 5 — pgvector (extension)

**A — Nhân quả**
- (không nhiều) — chuỗi đặc điểm.

**B — Ghi nhớ**
- ⚠ **`tsvector` là FTS (lexeme), KHÁC hẳn kiểu `vector` (embedding)** — đừng nhầm hai thứ. *(đính chính quan trọng)*
- pgvector là **BYO-embedding** (không tự sinh vector).
- Điểm mạnh: **hybrid query** (join/filter 1 câu SQL); hệ sinh thái managed rộng (Supabase, Neon, RDS/Aurora); **nhiều "món" index nhất** (HNSW, IVFFlat, DiskANN).

> Chain này **hoàn toàn nhóm B** — đặc điểm pgvector. Học như thẻ tra cứu.

**Xương sống**: pgvector = extension, BYO, hybrid 1 câu, nhiều index nhất; tsvector ≠ vector.

---

## Chain 6 — MySQL HeatWave (managed)

**A — Nhân quả**
- (không) — chuỗi đặc điểm.

**B — Ghi nhớ**
- **HeatWave** = dịch vụ *fully managed* của Oracle, gộp OLTP + analytics + ML; differentiator = **in-database embedding (HeatWave GenAI)** tự sinh real-time; chạy OCI + AWS, **cloud-only**.
- MySQL Community 9.0+ có kiểu `VECTOR` + distance function (`STRING_TO_VECTOR`, `DISTANCE`) nhưng **ANN index chủ yếu qua HeatWave** (proprietary).
- Hệ sinh thái MySQL phân mảnh: PlanetScale/Vitess (**SPANN**, 3/2025), Cloud SQL (ScaNN), TiDB (beta); RDS for MySQL chưa có vector index.

> Chain này **hoàn toàn nhóm B** — đặc điểm HeatWave + hệ sinh thái MySQL.

**Xương sống**: HeatWave = managed, in-DB embedding, cloud-only; MySQL Community có VECTOR nhưng ANN index qua HeatWave.

---

## Chain 7 — MariaDB Vector (native) & cùng bảng

**A — Nhân quả**
- lưu embedding trong cột `VECTOR(n)` NGAY trong bảng, cạnh business data => chạy **hybrid query một câu** — *vì* cùng bảng nên filter/join bằng SQL luôn.
- BẮT BUỘC có `ORDER BY VEC_DISTANCE_*() + LIMIT` thì index mới được dùng => thiếu `LIMIT` → full scan — *vì* planner chỉ kích hoạt vector index khi có ORDER BY distance + LIMIT.

**B — Ghi nhớ**
- MariaDB = fork của MySQL, đưa vector search **thẳng vào lõi** (không extension/plugin).
- ⚠ **Đính chính:** MariaDB KHÔNG lưu vector "riêng biệt ở database đặc biệt" — nằm cùng bảng.
- Cú pháp: `VECTOR INDEX ... DISTANCE=cosine`, `VEC_FromText` (parse chuỗi), `VEC_DISTANCE_COSINE` (query) — *cú pháp*.

**Xương sống**: MariaDB xây vector vào lõi → cột VECTOR cùng bảng → hybrid một câu; cần ORDER BY VEC_DISTANCE + LIMIT thì index mới chạy.

---

## Chain 8 — MariaDB đặc điểm 2026

**A — Nhân quả**
- (không) — chuỗi đặc điểm/con số.

**B — Ghi nhớ**
- GA từ **11.8 LTS (6/2025)**, nằm trong **Community Server (miễn phí)** — khác MySQL (index qua HeatWave proprietary).
- Dùng **HNSW**; hỗ trợ **cosine & Euclidean**; chiều tới **16.383** (so pgvector index tối đa **2000**).
- Tối ưu **SIMD** (AVX2/AVX512 Intel, NEON ARM) tính distance nhanh, khỏi cấu hình.
- **BYO** (không tự sinh embedding); có trên RDS for MariaDB 11.8; tích hợp LangChain/LlamaIndex/Spring AI.

> Chain này **hoàn toàn nhóm B** — con số + đặc điểm MariaDB. Điểm cần thuộc: **16.383 chiều** (gấp ~8 lần giới hạn index 2000 của pgvector) và **SIMD**.

**Xương sống**: MariaDB Vector GA 11.8, Community miễn phí, HNSW, 16.383 chiều, SIMD, BYO.

---

## Chain 9 — Đánh đổi 3 mô hình kiến trúc

**A — Nhân quả**
- (không nối) — bảng ưu-nhược.

**B — Ghi nhớ (ưu / nhược)**
- **extension** (pgvector): ưu linh hoạt tối đa (mã mở, cập nhật độc lập core, nhiều index); nhược phải cài & version, vài managed khóa version cũ.
- **native** (MariaDB): ưu không cài gì, SIMD ở tầng engine, tính năng đi cùng release; nhược buộc vào nhịp release DB, ít "món" (chỉ HNSW, cosine/L2).
- **managed** (HeatWave): ưu không vận hành, in-DB embedding, gộp OLTP+analytics+ML; nhược vendor lock-in, chi phí theo dịch vụ, không self-host.

> ⚠ Đây là **danh sách ưu-nhược 3 mô hình**, không phải chuỗi nhân quả. Học như bảng đối chiếu.

**Xương sống**: (bảng) extension = linh hoạt nhất / native = ít ma sát vận hành / managed = không vận hành nhưng lock-in.

---

## Chain 10 — Thuật toán index không đồng nhất

**A — Nhân quả**
- thuật toán index khác nhau (HNSW in-RAM vs SPANN/DiskANN disk-friendly) => quyết định trade-off memory/disk/recall/tốc độ — *vì* mỗi thuật toán đặt dữ liệu ở RAM hay SSD khác nhau.

**B — Ghi nhớ**
- pgvector/MariaDB/phần lớn MySQL forks → **HNSW**; PlanetScale (Vitess) → **SPANN** (Space-Partitioned ANN, lai cụm+graph, hợp disk); Cloud SQL → **ScaNN** (quantization+phân vùng); pgvectorscale → **DiskANN** (SSD) — *tên/dữ kiện*.

**Xương sống**: "vector search" không đồng nhất → HNSW / SPANN / ScaNN / DiskANN → thuật toán quyết định trade-off RAM ↔ disk ↔ recall.

---

## Chain 11 — Cùng bảng = hybrid; tách hệ = bug đồng bộ ↩(≈ file 1, 2)

**A — Nhân quả**
- tách vector sang hệ riêng => query vector → nhận IDs → query lại DB nghiệp vụ để lọc => hai hệ, hai round-trip, đồng bộ phức tạp — *vì* dữ liệu chi tiết nằm ở DB khác.
- mỗi CRUD phải sync sang hệ vector => nguồn bug kinh điển (dữ liệu lệch nhau) — *vì* hai kho phải đồng bộ tay.
- vector cùng bảng => hybrid query một câu, một transaction => lý do RDBMS-native hấp dẫn — *vì* không cần sync, không lệch.

**Xương sống**: tách hệ → 2 round-trip + sync bug; cùng bảng → hybrid một câu, một transaction → RDBMS-native hấp dẫn.

---

## Chain 12 — Edge cases nền tảng

**A — Nhân quả (mỗi edge có "vì" riêng)**
- **distance vs similarity**: nơi trả *distance* (nhỏ=gần), nơi trả *similarity* (lớn=gần) => `ORDER BY` có thể ngược — *vì* hai quy ước trái dấu, đọc kỹ tài liệu.
- **chiều vượt giới hạn index**: insert được (lưu) nhưng không index được => âm thầm seq scan chậm — *vì* vượt trần index thì planner bỏ index.
- **trộn hai model khác chiều**: không cùng không gian => distance vô nghĩa (lệch chiều còn lỗi insert) — *vì* toàn cột phải cùng model + version.

**B — Ghi nhớ**
- **managed lock-in**: dữ liệu/embedding ở HeatWave khó bê đi → tính chi phí rời đi trước khi cam kết — *khuyến nghị*.

> ⚠ Đây là **danh sách 4 edge case**, không phải chuỗi nhân quả. Học như checklist thủ sẵn.

**Xương sống**: (checklist) distance vs similarity (ORDER BY ngược) / vượt giới hạn → seq scan im lặng / trộn 2 model → vô nghĩa / managed lock-in.

---

## Chain 13 — Khung RDBMS-native vs Dedicated ↩(≈ file 1 chain 25, file 2 chain 21)

**A — Nhân quả**
- (phần lớn là tiêu chí — xem ⚠)

**B — Ghi nhớ (tiêu chí quyết định)**
- Chọn **RDBMS-native** khi: vector là *một tính năng* trong app đã chạy RDBMS; cần hybrid query (vector + filter/join) trong transaction ACID; **< ~10–50M vector**; muốn ít hệ thống.
- Chọn **dedicated** khi: vector search *là sản phẩm chính*; **> ~50M–hàng tỷ** cần multi-region; cần hybrid dense+sparse built-in / multi-tenancy nâng cao / filtering phức tạp quy mô lớn.
- Câu chốt cần thuộc: **RDBMS-native thắng operational simplicity, dedicated thắng peak scale; chọn theo "vector là feature hay là product".**

> ⚠ **Danh sách tiêu chí**, không phải nhân quả. Đây là bản thứ ba của cùng một kim chỉ nam ("feature hay product") đã gặp ở file 1 (NÊN pgvector) và file 2 (vượt Elasticsearch).

**Xương sống**: feature + <50M + cần hybrid/ACID → RDBMS-native; product + >50M + multi-region → dedicated.

---

## Chain 14 — Managed vs self-host & privacy ↩(≈ file 3 chain 13)

**A — Nhân quả**
- privacy/compliance (dữ liệu khách hàng không được rời hệ thống) => đôi khi **ép self-host** bất kể tiện lợi của managed — *vì* ràng buộc pháp lý thắng tiện lợi.

**B — Ghi nhớ (trade-off)**
- **managed** (HeatWave/Supabase/Neon/RDS): nhà cung cấp lo vận hành + tính năng ăn liền, nhưng lock-in cao, dữ liệu ở cloud bên thứ ba.
- **self-host** (pgvector/MariaDB Community): toàn quyền, dữ liệu ở nhà, lock-in thấp (mã mở), nhưng tự lo patch/backup/scale.

**Xương sống**: managed (tiện, lock-in cao) vs self-host (toàn quyền, tự lo) → privacy có thể ép self-host bất kể tiện.

---

## Chain 15 — Lock-in & migration (chi phí ẩn lớn nhất)

**A — Nhân quả**
- in-DB embedding sinh bởi model Oracle => đổi nền tảng phải re-embed toàn bộ (vector không tương thích không gian) — *vì* model khác không gian khác.
- đổi RDBMS = migration nặng: không chỉ data mà cả **cú pháp** (`<=>` vs `VEC_DISTANCE_COSINE` vs `DISTANCE()`), index, re-embed — *vì* mỗi nền tảng có phương ngữ riêng.
- lock-in đắt => ưu tiên chuẩn mở & BYO-embedding để giữ **đường lui** — *vì* BYO/chuẩn mở cho phép mang đi.

**B — Ghi nhớ**
- Ba phương ngữ distance: pgvector `<=>` · MariaDB `VEC_DISTANCE_COSINE` · MySQL `DISTANCE()` — *cú pháp*.

**Xương sống**: in-DB embedding & phương ngữ riêng → đổi nền tảng = data + cú pháp + re-embed → chọn BYO/chuẩn mở giữ đường lui.

---

## Chain 16 — Reliability, cost & tổ chức

**A — Nhân quả**
- RDBMS-native thừa hưởng HA/backup/replication trưởng thành của database mẹ => lợi thế so với vector DB non trẻ ("một hệ để trông" thay vì hai) — *vì* dùng lại hạ tầng đã chín.
- tách vector DB riêng => kéo theo kỹ năng/on-call mới — *vì* thêm một hệ là thêm một mặt vận hành.
- nhiều team cần vector => chuẩn hóa *một* mô hình (pgvector cho mọi service Postgres) giảm phân mảnh — *vì* nhiều mô hình rải rác khó bảo trì.

**B — Ghi nhớ**
- Cost drivers: RAM cho HNSW in-memory, phí managed theo dùng, phí sinh embedding, re-embed khi đổi model — *dữ kiện*.
- Framing cho PM/sếp bằng **rủi ro / chi phí / lock-in / privacy** — *khuyến nghị*.

**Xương sống**: RDBMS-native thừa hưởng HA ("một hệ để trông") → tách hệ thêm on-call → chuẩn hóa một mô hình toàn công ty.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (file 5)

> app đã chạy RDBMS cần vector → **cơi nới RDBMS** (một nhà) vs **mua nhà thứ hai** (dedicated) → RDBMS cơi nới theo **3 mô hình** (extension / native / managed) → luồng **4 bước giống nhau, chỉ khác cú pháp** → nguồn embedding: **BYO** (kiểm soát model) vs **in-DB** (tiện nhưng lock-in) → ba nền tảng: pgvector (extension, BYO, nhiều index) / HeatWave (managed, in-DB embedding, cloud-only) / MariaDB (native, cùng bảng, 16.383 chiều, SIMD) → **thuật toán index không đồng nhất** (HNSW/SPANN/ScaNN/DiskANN) quyết định trade-off RAM↔disk → **cùng bảng = hybrid một câu; tách hệ = 2 round-trip + bug đồng bộ** → **quyết định 1**: RDBMS-native (feature, <50M) vs dedicated (product, >50M) → **quyết định 2**: managed vs self-host, **privacy ép self-host** → **lock-in & migration** (đổi nền tảng = data + cú pháp + re-embed) → chọn **BYO/chuẩn mở giữ đường lui** → RDBMS-native thừa hưởng HA "một hệ để trông" → **chuẩn hóa một mô hình** toàn công ty.

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (5 file)

> **text → [ENCODE] embed (file 3)** → **[INDEX] B-tree gãy → ANN, IVFFlat/HNSW (file 4)** → **[STORE & SEARCH] pgvector (file 1)** → **[KEYWORD] full-text search (file 2)** → **[HYBRID] FTS + vector RRF** → **[CHỌN NỀN TẢNG] RDBMS-native vs dedicated, managed vs self-host (file 5)** → **RAG**.

File 5 là bài **lùi một bước** hỏi câu chiến lược trùm lên cả bốn bài kỹ thuật: bốn mảnh kia dạy *làm thế nào*, bài này hỏi *đặt ở đâu và vì sao*. Năm mảnh, (lý tưởng là) một Postgres.

---

## 8. MẪU HÌNH LẶP LẠI

**Mẫu hình 1 — "Feature hay product?" quyết định RDBMS-native vs dedicated.** ⭐ kim chỉ nam trùm series.
Chain 13 ↔ file 1 chain 25 (NÊN pgvector) ↔ file 2 chain 21 (vượt Elasticsearch). *Vì sao là một*: cả ba bài chốt cùng một câu — RDBMS-native thắng operational simplicity, hệ chuyên dụng thắng peak scale; vector là *feature* thì ở RDBMS, là *product* thì tách ra.

**Mẫu hình 2 — Cùng bảng = hybrid một câu; tách hệ = round-trip + bug đồng bộ (operational simplicity).**
Chain 11 ↔ file 1 chain 10 (siêu năng lực pgvector) ↔ file 2 (một Postgres cho hybrid). *Vì sao là một*: lợi thế xuyên series là **một hệ thống, một câu SQL, một transaction** — không sync tay giữa hai kho.

**Mẫu hình 3 — Privacy có thể ép self-host bất kể benchmark/tiện lợi.**
Chain 14 ↔ file 3 chain 13 (API vs self-host). *Vì sao là một*: khi dữ liệu không được rời hệ thống, compliance thắng cả hiệu năng lẫn tiện lợi.

**Mẫu hình 4 — Đổi nền tảng/model = re-embed + migration nặng (lock-in).** ⭐ mẫu hình mạnh nhất xuyên cả 5 file.
Chain 4 & 15 ↔ file 1 & 3 (đổi embedding model = re-embed) ↔ file 2 (đổi config = reindex) ↔ file 4 (index có vòng đời). *Vì sao là một*: mọi thứ phái sinh (vector/tsvector/index) gắn với một "gốc" (model/config/cú pháp nền tảng); đổi gốc = làm lại toàn bộ → luôn ưu tiên BYO/chuẩn mở + version hóa.

**Mẫu hình 5 — Lỗi im lặng.**
Chain 12: chiều vượt giới hạn → âm thầm seq scan; distance vs similarity → ORDER BY ngược; trộn 2 model → distance vô nghĩa ↔ file 1/2/4 (ops class sai bỏ index, NULL, recall tụt). *Vì sao là một*: hệ không báo lỗi, chỉ trả kết quả sai/chậm → phải đọc kỹ + kiểm.

**Mẫu hình 6 — Học một cái, ba cái chỉ khác cú pháp.**
Chain 3. *Vì sao đáng nhớ*: 4 bước (cột → embed → index → ORDER BY distance LIMIT) là bất biến qua pgvector/MariaDB/MySQL; nắm ý tưởng thì tra cú pháp là xong.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

Bài này rất nặng nhóm B — đây là phần chính cần học.

### Con số
- MariaDB Vector GA **11.8 LTS (6/2025)**, Community miễn phí; chiều tới **16.383**.
- pgvector index tối đa **2000 chiều** (halfvec cao hơn).
- MySQL Community **9.0+** có `VECTOR`; PlanetScale SPANN từ **3/2025**.
- Ngưỡng chọn: **< ~10–50M** → RDBMS-native; **> ~50M–hàng tỷ** + multi-region → dedicated.

### Định nghĩa & tên gọi
- **3 mô hình**: extension (pgvector) / native (MariaDB) / managed (HeatWave).
- **BYO-embedding** vs **in-database embedding** (HeatWave GenAI).
- **Mental model 4 bước**: cột vector → embed+insert → index HNSW → ORDER BY distance LIMIT.
- ⚠ **`tsvector` (FTS/lexeme) ≠ kiểu `vector` (embedding)** — đừng nhầm.
- **Thuật toán ANN theo nền tảng**: HNSW (pgvector/MariaDB/MySQL forks) · **SPANN** = Space-Partitioned ANN (PlanetScale/Vitess) · **ScaNN** (Cloud SQL) · **DiskANN** (pgvectorscale).
- **Distance metrics**: cosine, Euclidean (L2), Manhattan, Jaccard.
- **RDBMS-native** vs **dedicated** (Pinecone/Weaviate/Qdrant/Milvus).
- **managed** vs **self-host**; **vendor lock-in**; **operational simplicity**.
- Mental models: "cơi nới vs nhà thứ hai", "feature hay product", "một hệ để trông", "tiện đi kèm xích".
- SIMD (AVX2/AVX512/NEON) — MariaDB tối ưu phần cứng tính distance.

### ⚠ Năm đính chính bài gốc (phải ghi đè)
1. MariaDB Vector đã **GA** trong **11.8 LTS (6/2025)** — không phải "tương lai".
2. MariaDB lưu embedding **cùng bảng** với business data — không "riêng biệt ở database đặc biệt".
3. PlanetScale (Vitess) có vector index từ **3/2025** (thuật toán **SPANN**).
4. `tsvector` là **FTS (lexeme)**, khác hẳn kiểu `vector` (embedding) của pgvector.
5. MySQL Community 9.0+ có `VECTOR` + distance function nhưng **ANN index chủ yếu qua HeatWave** (managed/proprietary).

### Cú pháp (3 phương ngữ)
- **pgvector:** `CREATE EXTENSION vector;` · `embedding vector(1536)` · `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops);` · `ORDER BY embedding <=> '[...]' LIMIT 5;`
- **MariaDB:** `embedding VECTOR(1536) NOT NULL, VECTOR INDEX (embedding) DISTANCE=cosine` · `ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]')) LIMIT 5;` (**LIMIT bắt buộc** để dùng index)
- **MySQL (9.x/HeatWave):** `val VECTOR(5)` · `STRING_TO_VECTOR('[1,2,3,4,5]')` · `ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE') LIMIT 5;`

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào.

1. Vì sao "mua nhà thứ hai" (dedicated vector DB) sinh **bug đồng bộ** mà "cơi nới" (RDBMS-native) thì không?
2. Vì sao in-database embedding (HeatWave) tiện nhưng **khóa chặt** hơn BYO?
3. Vì sao ba mô hình (extension/native/managed) chỉ khác **cú pháp** chứ không khác ý tưởng?
4. Vì sao `tsvector` KHÁC hẳn kiểu `vector`, và nhầm hai thứ dẫn tới hậu quả gì?
5. Vì sao MariaDB **bắt buộc** `ORDER BY VEC_DISTANCE + LIMIT` thì index mới được dùng?
6. Vì sao thuật toán index (HNSW vs SPANN/DiskANN) **quyết định** trade-off RAM/disk?
7. Vì sao "vector cùng bảng với business data" là lý do RDBMS-native hấp dẫn?
8. Vì sao **privacy/compliance** đôi khi ép self-host bất kể managed tiện hơn?
9. Vì sao đổi RDBMS là **migration nặng** — nêu ba thứ phải làm lại?
10. Vì sao BYO-embedding + chuẩn mở giữ được **"đường lui"**?
11. Vì sao chiều vượt giới hạn index gây **lỗi im lặng** (seq scan)?
12. Vì sao RDBMS-native thừa hưởng **reliability** tốt hơn vector DB non trẻ?
13. Vì sao **"feature hay product"** là kim chỉ nam chọn RDBMS-native vs dedicated?
14. Vì sao **chuẩn hóa một mô hình** vector toàn công ty giảm phân mảnh?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- Bài này **nặng nhóm B** hơn hẳn các bài cơ chế — chấp nhận điều đó: đây là bài phải *học thuộc* nhiều (đặc điểm nền tảng, con số, 3 phương ngữ cú pháp, 5 đính chính). Đừng cố "hiểu" chỗ vốn là dữ kiện.
- **Nhóm A ít nhưng quan trọng**: cùng bảng → hybrid (chain 11); in-DB embedding → lock-in → re-embed (chain 4, 15); privacy → ép self-host (chain 14). Nắm ba mắt xích này là nắm phần "vì sao" của cả bài.
- **Tái tạo**: dựng lại hai trục quyết định (RDBMS-native vs dedicated; managed vs self-host) + bảng 3 nền tảng, rồi ghép vào **meta-xương sống 5 file**.
- **Kim chỉ nam trùm series**: "vector là *feature* hay *product*?" — xuất hiện ở file 1, 2, và 5. Một câu hỏi, quyết cả kiến trúc.
