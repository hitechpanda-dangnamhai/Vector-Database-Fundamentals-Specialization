# Phân tích chain học tập — Indexing cho Vector Search: Binary Search → IVFFlat & HNSW

> Mục tiêu: tách *suy ra được* (nhóm A) khỏi *phải ghi nhớ* (nhóm B), đánh dấu **⚠** chỗ logic nhảy, rút **xương sống** để tái tạo.

Đây là **mảnh 4/4** khép series ("làm search nhanh"). **Lưu ý về trùng lặp:** phần lõi (IVFFlat / HNSW / curse of dimensionality / DiskANN / build & memory / monitoring / overfiltering) gần như **trùng hoàn toàn với file 1 (pgvector)** — nếu đã nắm file 1 thì đây chỉ là ôn lại. **Phần MỚI, đáng học nhất của file 4** là chain 1–5: *lý thuyết index → binary search → total order → B-tree → vì sao B-tree gãy với vector*. File 1 bỏ qua mắt xích này; nó là câu trả lời cho "vì sao vector cần index khác hẳn".

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Index là gì & vì sao đáng ⭐(mới so với file 1)

**A — Nhân quả**
- index trả chút chi phí lưu trữ + cập nhật => đổi lấy tốc độ tìm kiếm khổng lồ — *vì* có bản sao đã tổ chức thì định vị nhanh, khỏi quét cả bảng.
- đổi chác đó chỉ đáng khi dữ liệu LỚN — *vì* data nhỏ thì linear search quét cũng đã nhanh.

**B — Ghi nhớ**
- **database index** — *định nghĩa*: cấu trúc dữ liệu tăng tốc tìm kiếm; chứa bản sao đã tổ chức của một phần dữ liệu.
- Analogy **"mục lục cuối sách"**: "database → trang 12, 45" → nhảy thẳng — *mental model*.

**Xương sống**: index = mục lục → trả chi phí lưu/build đổi lấy tốc độ → chỉ đáng khi data lớn.

---

## Chain 2 — Linear search tốn thế nào ⭐(mới)

**A — Nhân quả**
- mỗi query đọc ~`n/2` dòng × hàng nghìn query/giây => sập — *vì* khối lượng đọc nhân lên khổng lồ.
- "không index" bất khả thi khi lớn => cứu bằng sort + **binary search** — *vì* cần rẻ hơn `O(n)`.

**B — Ghi nhớ**
- **linear search**: so từng dòng; trung bình `(n+1)/2`, tệ nhất `n`; `n=1M` → ~**500.000** dòng cho một lần tìm — *con số*.

**Xương sống**: linear search O(n) → 500K reads/query khi n=1M → sập dưới tải → sort + binary search.

---

## Chain 3 — Binary search chạy tay ("Jack") ⭐(mới)

**A — Nhân quả**
- chia đôi mỗi bước => chỉ cần ~`log₂(n)` bước — *vì* mỗi bước loại nửa số ứng viên.

**B — Ghi nhớ**
- Cần data đã **SORT**, rồi chia đôi liên tục; tìm "Jack" trong 20 tên = **4 bước** (loại 10 tên/bước) — *ví dụ*.
- `log₂(n)`: 1 triệu tên → chỉ ~**20 bước** (`2²⁰ ≈ 1.048.576`) — *con số*; đó là sức mạnh `O(log n)`.

> Chain này chủ yếu nhóm B (worked example). Nhớ "4 bước tìm Jack / 20 bước cho 1 triệu" là nắm được ý.

**Xương sống**: sort rồi chia đôi → mỗi bước loại nửa → ~log₂(n) bước → O(log n).

---

## Chain 4 — Điều kiện tiên quyết → B-tree ⭐(mới, mắt xích bắc cầu)

**A — Nhân quả**
- tên sắp thứ tự trên MỘT trục (**total order**) => mỗi bước trả lời được "mục tiêu trước hay sau điểm giữa?" => binary search chạy được — *vì* không có thứ tự thì không biết vứt nửa nào.
- điều kiện "sắp xếp được trên một trục" chính là chỗ sẽ **VỠ** với vector nhiều chiều — *bắc cầu sang chain 5*.

**B — Ghi nhớ**
- **B-tree** — *định nghĩa*: cây cân bằng hiện thực binary search, tìm/range scan `O(log n)`; tuyệt cho dữ liệu 1 chiều sắp thứ tự được (số/chuỗi/ngày).

**Xương sống**: total order (sắp trên một trục) → binary search / B-tree chạy được → điều kiện này là chỗ vector sẽ phá.

---

## Chain 5 — Vì sao B-tree GÃY với vector ⭐⭐(mắt xích CỐT LÕI của cả file)

**A — Nhân quả**
- vector nhiều chiều VỠ total order **vì 3 lý do**:
  - lý do 1: sort theo *chiều nào*? — *vì* không tồn tại một trục duy nhất mà "gần trên trục = gần thật sự".
  - lý do 2: câu hỏi khác hẳn — B-tree hỏi "bằng đúng X / trong khoảng [a,b]", vector hỏi "vector nào **gần nhất**" (**nearest neighbor**) — *vì* bài toán là khoảng cách, không phải khớp giá trị.
  - lý do 3: **curse of dimensionality** — kd-tree ở chiều cao phải duyệt gần hết → không nhanh hơn brute-force.
- ba lý do đó => từ bỏ tìm chính xác tuyệt đối, chuyển **Approximate Nearest Neighbor (ANN)** — *vì* cả B-tree lẫn cây phân hoạch đều tắc.

**B — Ghi nhớ**
- **ANN** — *tên gọi*: chấp nhận bỏ sót vài hàng xóm thật đổi lấy tốc độ; **IVFFlat** và **HNSW** là hai cách làm.

**Xương sống**: vector phá total order (sort chiều nào / bài toán NN / curse) → bỏ exact → ANN (IVFFlat, HNSW).

---

## Chain 6 — IVFFlat cơ chế ↩(≈ file 1)

**A — Nhân quả**
- **k-means** chia `lists` cụm quanh **centroid** => search chỉ brute-force TRONG cụm gần nhất — *vì* bỏ qua phần lớn vector.
- phân hoạch cứng => hàng xóm sát **ranh giới cụm** bị bỏ sót — *vì* rơi sang cụm `probes` không dò tới.
- centroid cố định lúc build => thêm data → lệch, recall tụt, phải `REINDEX` — *vì* IVFFlat tĩnh.

**B — Ghi nhớ**
- **IVFFlat** = Inverted File with Flat compression — *tên*.
- `lists` = `rows/1000` (≤1M) / `sqrt(rows)` (>1M); `probes` mặc định **1**, khởi đầu `sqrt(lists)` — *con số*.

**Xương sống**: k-means chia cụm (cần data) → query vào `probes` cụm gần → nhanh nhưng miss ranh giới + tĩnh phải REINDEX.

---

## Chain 7 — Exact vs Approximate ↩(≈ file 1)

**A — Nhân quả**
- no index → exact: quét hết, recall **100%** đúng nhưng `O(n)` chậm — *vì* so với mọi vector.
- ANN index → approximate: nhanh nhưng **recall < 100%** — *vì* bỏ qua phần dữ liệu để đi tắt.
- chọn + tune index => cân bằng **accuracy ↔ speed ↔ resource** (linh hồn cả bài) — *vì* mọi tham số đều trượt trên trục này.

**B — Ghi nhớ**
- recall < 100% là **by design**, không phải bug — *nhận định cần thuộc*.

**Xương sống**: exact recall 100% nhưng O(n) → ANN nhanh nhưng recall<100% (by design) → tune = cân bằng accuracy↔speed↔resource.

---

## Chain 8 — Ba lỗi index ↩(≈ file 1, có bổ sung)

**A — Nhân quả (mỗi lỗi có "vì" riêng)**
- lỗi 1 — index quá sớm trên data nhỏ => chỉ giảm chính xác mà không lợi tốc độ — *vì* brute-force exact đã đủ nhanh + recall 100%.
- lỗi 2 — tạo IVFFlat khi bảng RỖNG => phân cụm vô nghĩa, recall tệ — *vì* k-means cần data để tìm centroid (HNSW thì tạo được cả khi bảng rỗng).
- lỗi 3 — sai ops class (`vector_cosine_ops` mà query `<->`) => Postgres **bỏ qua index**, rơi seq scan — *vì* planner chỉ dùng index khi ops class khớp toán tử.

**B — Ghi nhớ**
- ops class phải khớp: `cosine_ops`↔`<=>`, `l2_ops`↔`<->`, `ip_ops`↔`<#>`; kiểm bằng `EXPLAIN ANALYZE` — *quy tắc/cú pháp*.

> ⚠ Đây là **danh sách 3 lỗi**, không phải chuỗi nhân quả nối nhau. Học như checklist.

**Xương sống**: (checklist) đừng index data nhỏ / đừng IVFFlat bảng rỗng / ops class phải khớp toán tử.

---

## Chain 9 — HNSW graph nhiều tầng ↩(≈ file 1)

**A — Nhân quả**
- tầng trên ít node, cạnh "đường dài" => nhảy xa nhanh về đúng vùng; tầng dưới mọi node, cạnh ngắn => tinh chỉnh — *vì* cấu trúc phân tầng cho phép greedy từ thô đến tinh.
- greedy từ tầng trên tụt xuống đáy => search ~`O(log n)` — *vì* mỗi tầng thu hẹp nhanh (như bay quốc tế về đúng thành phố rồi taxi tới địa chỉ).

**B — Ghi nhớ**
- **HNSW** = Hierarchical Navigable Small **World** — *tên*.
- Cạnh nối theo **distance metric đã cấu hình** (cosine/L2/ip) — *đính chính*.
- Search đi **Layer 3 → 2 → 1 → 0** (trên xuống) — *đính chính*.

> ⚠ **3 đính chính reading (phải ghi đè):** (1) "Small **World**" không phải "Small Words"; (2) cạnh theo metric cấu hình, không riêng Euclidean; (3) tầng 3→2→1→0.

**Xương sống**: graph nhiều tầng → tầng cao nhảy xa, tầng thấp tinh chỉnh → greedy 3→2→1→0 → O(log n).

---

## Chain 10 — HNSW tham số tune ↩(≈ file 1)

**A — Nhân quả**
- (trade-off, không nhân quả tuyến tính)

**B — Ghi nhớ**
- `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops) WITH (m=16, ef_construction=64)` — *cú pháp*.
- `m` = cạnh tối đa/node/tầng (16); cao → recall tốt, graph to, RAM/build tốn — *con số/trade-off*.
- `ef_construction` = ứng viên xét *khi xây* (64); cao → graph tốt hơn, build chậm — *con số*.
- `ef_search` = ứng viên xét *khi tìm* (40); cao → recall tốt, chậm; là **núm xoay recall–speed quan trọng nhất lúc chạy** (`SET hnsw.ef_search = 100`) — *con số/cú pháp*.

> Chain này chủ yếu nhóm B (con số + cú pháp). Ba tham số cùng một trục: recall lên ↔ chi phí lên.

**Xương sống**: m (build) / ef_construction (build) / ef_search (query) → đều recall ↔ chi phí; ef_search là núm chỉnh lúc chạy.

---

## Chain 11 — Curse of dimensionality (cơ chế) ↩(≈ file 1)

**A — Nhân quả**
- `d` lên hàng trăm–nghìn => thể tích tăng theo hàm mũ → data trở nên *thưa* — *vì* không gian phình quá nhanh.
- đồng thời khoảng cách gần nhất và xa nhất **co lại gần nhau** => "gần nhất" mất ý nghĩa phân biệt — *đặc trưng của curse*.
- cây phân hoạch phải duyệt gần hết nhánh => sập về `O(n)` => cần approximate + graph/cụm thay cây — *vì* cây không còn cắt bớt được gì.

**Xương sống**: d cao → data thưa + khoảng cách co lại → cây duyệt gần hết → O(n) → cần ANN (graph/cụm).

---

## Chain 12 — DiskANN & giới hạn 2000 chiều ↩(≈ file 1)

**A — Nhân quả**
- DiskANN giữ graph trên **SSD** + binary quantization => index nhỏ hơn HNSW nhiều lần — *vì* không nhét cả graph vào RAM và vector bị nén.
- model 3072 chiều (OpenAI 3-large) => dính giới hạn — *vì* 3072 > 2000 → workaround `halfvec` (~4000) hoặc DiskANN.

**B — Ghi nhớ**
- **DiskANN** qua `pgvectorscale` (Timescale); native tới **16000 chiều** — *tên/con số*.
- HNSW/IVFFlat pgvector chỉ **index tối đa 2000 chiều** — *con số*.

**Xương sống**: DiskANN (SSD + nén) → index nhỏ, chịu dim lớn; vector >2000 chiều → halfvec hoặc DiskANN.

---

## Chain 13 — Chọn index theo quy mô ↩(≈ file 1)

**A — Nhân quả**
- nhỏ (<10K–50K vector) => ĐỪNG index — *vì* brute-force exact đủ nhanh, recall 100%, khỏi tune.

**B — Ghi nhớ (tiêu chí)**
- Trung bình (tới vài triệu): **HNSW** mặc định (recall cao, latency thấp, chịu data động).
- RAM hạn chế / build lại thường / data tĩnh: **IVFFlat** (nhẹ, build nhanh) hoặc **DiskANN** khi khổng lồ.
- Chiều > 2000: **halfvec + HNSW** hoặc **DiskANN**.

> ⚠ Phần lớn là **danh sách tiêu chí theo quy mô**. Câu chốt: "câu hỏi staff không phải IVFFlat-hay-HNSW trừu tượng mà 'ở quy mô CỦA TÔI thì cái nào'."

**Xương sống**: nhỏ đừng index → HNSW default → IVFFlat/DiskANN khi RAM hẹp → halfvec/DiskANN khi >2000 chiều.

---

## Chain 14 — Build cost & memory ↩(≈ file 1)

**A — Nhân quả**
- HNSW build `O(n log n)` tốn hàng giờ => nạp data xong mới build + tăng `maintenance_work_mem` + `max_parallel_maintenance_workers` + `CREATE INDEX CONCURRENTLY` — *vì* build một lượt + nhiều RAM/luồng + không khóa write.
- graph phải nằm gọn RAM để nhanh => RAM là ràng buộc số một => quantization/DiskANN — *vì* graph spill khỏi RAM thì sập tốc độ.

**B — Ghi nhớ**
- `100M × 1536 × 4 byte ≈ 600GB` raw + overhead graph — *con số*.
- `maintenance_work_mem` ≤ **50–60% RAM** — *con số*.
- Data động: IVFFlat lệch cụm → reindex; HNSW chịu insert tốt hơn nhưng VACUUM chậm (reindex trước) — *dữ kiện*.

**Xương sống**: build O(n log n) tốn → tối ưu build → graph phải in-RAM → RAM là ràng buộc số một → quantization/DiskANN.

---

## Chain 15 — Monitoring recall ↩(≈ file 1)

**A — Nhân quả**
- ANN không có "đúng/sai" nhị phân => phải ĐO recall định kỳ — *vì* chất lượng liên tục, không assert được.
- không đo recall => **bay mù**: chất lượng xuống dốc mà không lỗi nào báo — *vì* ANN không phát tín hiệu lỗi.
- recall tụt => tăng `ef_search`/`probes`/`m` hoặc reindex — *vì* đó là các núm nâng recall.

**B — Ghi nhớ**
- Đo bằng ép exact `SET LOCAL enable_indexscan = off` làm ground truth → so overlap top-k → **recall@k**, dashboard — *cú pháp/định nghĩa*.

**Xương sống**: ANN không nhị phân → phải đo recall@k (exact làm ground truth) → tụt thì tune/reindex; không đo = bay mù.

---

## Chain 16 — Filtered search & overfiltering ↩(≈ file 1)

**A — Nhân quả**
- index lấy `ef_search` hàng xóm *trước*, lọc *sau* => filter hiếm → trả ít hơn `LIMIT` = **overfiltering** — *vì* WHERE loại gần hết đám ứng viên đã bốc ra.

**B — Ghi nhớ**
- Khắc phục: **iterative index scan** (pgvector 0.8+): `SET hnsw.iterative_scan = relaxed_order` + `max_scan_tuples`; prune sớm bằng **partition** theo tenant/category — *cú pháp*.

**Xương sống**: ANN bốc hàng xóm trước, lọc sau → filter hiếm gây overfiltering → iterative scan + partition prune.

---

## Chain 17 — Tổ chức & khép series

**A — Nhân quả**
- data đổi liên tục => index có **vòng đời**, không "set-and-forget" — *vì* phải đo recall & latency liên tục, reindex khi data đổi.
- mọi index nằm trong Postgres => backend team tự vận hành, khỏi thêm hệ thống search riêng — *vì* cùng một stack.

**B — Ghi nhớ**
- Framing cho PM/sếp bằng **chi phí, tốc độ, đánh đổi độ chính xác** — *khuyến nghị*.
- Khép series 4 mảnh: embed → **index** → store & query → keyword FTS → hybrid, tất cả trong một Postgres — *tổng hợp*.

**Xương sống**: index như mục lục (chi phí đổi tốc độ + chút độ chính xác) → có vòng đời phải đo/reindex → một Postgres cho cả series.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (file 4)

> index = mục lục (trả chi phí đổi tốc độ, chỉ đáng khi lớn) → **linear search O(n)** quá đắt (500K reads/query ở 1M) → sort + **binary search O(log n)** → binary search cần **total order** (sắp trên một trục) → **B-tree** hiện thực điều đó cho dữ liệu 1 chiều → **vector nhiều chiều PHÁ total order** (sort chiều nào? / bài toán nearest neighbor / curse of dimensionality) → bỏ exact, chuyển **ANN** → **IVFFlat** (k-means chia cụm, probes) hoặc **HNSW** (graph nhiều tầng, O(log n)) → exact recall 100% vs ANN <100% *by design* → tune = **accuracy ↔ speed ↔ resource** → chọn index theo quy mô (nhỏ đừng index / HNSW default / IVFFlat–DiskANN khi RAM hẹp / >2000 chiều halfvec) → build O(n log n), **RAM là ràng buộc số một** → **đo recall** (không đo = bay mù) → filtered search gây **overfiltering** → iterative scan + partition prune → mọi thứ trong **một Postgres**, khép series.

**Phần thực sự mới của file 4** là nửa đầu (đến "phá total order"). Nửa sau treo lên đúng bộ khung file 1.

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (4 file)

> **text → [ENCODE] embed (file 3)** → **[INDEX] binary/B-tree gãy → ANN, IVFFlat/HNSW (file 4)** → **[STORE & SEARCH] pgvector (file 1)** → **[KEYWORD] full-text search (file 2)** → **[HYBRID] FTS + vector fuse RRF** → **RAG**. Bốn mảnh, một Postgres.

---

## 8. MẪU HÌNH LẶP LẠI

**Mẫu hình 1 — "Sắp xếp được (total order) thì binary search/B-tree được; vector phá total order nên phải ANN."** ⭐ đóng góp RIÊNG của file 4.
Chain 3, 4, 5. *Vì sao quan trọng*: đây là câu trả lời cho "vì sao vector cần index khác" mà file 1 chỉ nói kết quả (curse of dimensionality) chứ không dẫn từ gốc (total order). Nếu chỉ đọc file 1, bạn thiếu mắt xích này.

**Mẫu hình 2 — Núm vặn recall ↔ speed ↔ resource ("linh hồn cả bài").**
Chain 7, 10, 13, 14 ↔ file 1 (khắp nơi), file 3 (dimension/Matryoshka). *Vì sao là một*: exact-vs-ANN, m/ef_*, probes, quantization đều trượt trên cùng một trục đánh đổi.

**Mẫu hình 3 — Lỗi im lặng.**
Sai ops class → âm thầm bỏ index (chain 8); recall tụt "bay mù" (chain 15) ↔ file 1, 2, 3. *Vì sao là một*: hệ không báo lỗi, chỉ trả kết quả kém → phải `EXPLAIN ANALYZE` + đo recall.

**Mẫu hình 4 — Ops class phải khớp toán tử (cùng một phép biến đổi hai phía).**
Chain 8 ↔ file 1 (chain 8/12) ↔ file 2 (doc & query cùng bộ chuẩn hóa) ↔ file 3 (không trộn 2 model). *Vì sao là một*: index/so sánh chỉ dùng được khi hai vế đồng nhất về metric/không gian.

**Mẫu hình 5 — Data nhỏ đừng over-engineer.**
Chain 1, 7, 8, 13 ↔ file 1 (chain 5/19). *Vì sao là một*: dưới ngưỡng ~10K–50K, exact 100% recall còn nhanh hơn và đúng hơn ANN. "Không phải mọi thứ đều cần index = tư duy staff."

**Mẫu hình 6 — Index/graph có vòng đời, không set-and-forget.**
Chain 6 (IVFFlat REINDEX), 14 (VACUUM/reindex), 17 (đo & reindex khi data đổi) ↔ file 2 (đổi config = reindex), file 1 & 3 (đổi model = re-embed). *Vì sao là một*: cấu trúc phái sinh (index/tsvector/embedding) đều xuống cấp khi dữ liệu/cấu hình gốc đổi → phải bảo trì có kế hoạch.

**Mẫu hình 7 — Operational simplicity: một Postgres cho tất cả.**
Chain 17 ↔ mọi file. Meta-pattern khép series.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

### Con số
- linear: trung bình `(n+1)/2`, tệ nhất `n`; 1M → ~**500.000** reads.
- binary: `log₂(n)`; 1M → ~**20 bước** (`2²⁰ ≈ 1.048.576`); "Jack" **4 bước**/20 tên.
- IVFFlat: `lists` = `rows/1000` (≤1M) / `sqrt(rows)` (>1M); `probes` default **1**, khởi đầu `sqrt(lists)`.
- HNSW default: `m=16`, `ef_construction=64`, `ef_search=40`.
- Giới hạn index **2000 chiều**; `vector` lưu **16000**; OpenAI 3-large **3072**; `halfvec` ~**4000**.
- `100M × 1536 × 4 byte ≈ 600GB`; `maintenance_work_mem` ≤ **50–60% RAM**.
- Ngưỡng: **<10K–50K** vector thì đừng index.
- Big-O: brute-force `O(n·d)`; IVFFlat `O(lists·d + (n/lists)·probes·d)`; HNSW `O(log n·d)`.

### Định nghĩa & tên gọi
- **database index**, mục lục cuối sách; **linear search**, **binary search**, **total order**, **B-tree**.
- **nearest neighbor**, **ANN** (Approximate Nearest Neighbor).
- **IVFFlat** = Inverted File with Flat compression; **k-means**, **centroid**, `lists`, `probes`.
- **HNSW** = Hierarchical Navigable Small **World**; graph nhiều tầng; `m` / `ef_construction` / `ef_search`.
- **curse of dimensionality**; **DiskANN** / `pgvectorscale`; **halfvec**, **binary quantization**.
- ops class: `vector_cosine_ops`↔`<=>`, `vector_l2_ops`↔`<->`, `vector_ip_ops`↔`<#>`.
- **recall@k**, ground truth, **overfiltering**, **iterative_scan**.

### ⚠ Ba đính chính reading (phải ghi đè)
1. HNSW = Hierarchical Navigable Small **World** (không "Small Words").
2. Cạnh HNSW nối theo **metric đã cấu hình** (cosine/L2/ip), không riêng Euclidean.
3. Tầng đi **Layer 3 → 2 → 1 → 0** (trên xuống dưới).

### Cú pháp
- `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops) WITH (m=16, ef_construction=64);`
- `CREATE INDEX ... USING ivfflat (embedding vector_cosine_ops) WITH (lists=100);` — **tạo SAU khi bảng có dữ liệu**.
- `SET hnsw.ef_search = 100;` · `SET ivfflat.probes = 32;`
- Đo recall: `BEGIN; SET LOCAL enable_indexscan = off; SELECT ... ORDER BY embedding <=> :q LIMIT 10; COMMIT;`
- `SET hnsw.iterative_scan = relaxed_order;` + `max_scan_tuples`.
- `CREATE INDEX CONCURRENTLY`; `maintenance_work_mem`; `max_parallel_maintenance_workers`.
- `binary_search(a, x)` Python: `lo/hi/mid`, `a[mid] < x → lo = mid+1`.

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào.

1. Vì sao linear search bất khả thi khi `n` lớn (nêu con số cụ thể)?
2. Vì sao binary search chỉ cần ~`log₂(n)` bước?
3. Vì sao binary search / B-tree **đòi total order** (sắp trên một trục)? ⭐
4. Vì sao vector nhiều chiều **phá vỡ** điều kiện total order — nêu 3 lý do? ⭐⭐
5. Vì sao câu hỏi "nearest neighbor" khác câu hỏi mà B-tree trả lời được?
6. Vì sao curse of dimensionality khiến kd-tree sập về `O(n)` ở chiều cao?
7. Vì sao IVFFlat cần data để build còn HNSW thì không?
8. Vì sao IVFFlat bỏ sót hàng xóm **sát ranh giới cụm**?
9. Vì sao HNSW search ~`O(log n)` nhờ cấu trúc nhiều tầng?
10. Vì sao recall < 100% của ANN là **by design** chứ không phải bug?
11. Vì sao sai ops class khiến Postgres **âm thầm** bỏ index?
12. Vì sao không đo recall là **"bay mù"**?
13. Vì sao overfiltering xảy ra, và iterative scan sửa thế nào?
14. Vì sao đánh index **quá sớm** trên data nhỏ là sai lầm?
15. Vì sao **RAM** là ràng buộc số một của HNSW ở quy mô lớn?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- **Nếu đã học file 1**: tập trung vào **chain 1–5** (lý thuyết index → total order → B-tree gãy). Đó là phần MỚI. Chain 6–16 gần như ôn lại — lướt nhanh, chỉ kiểm chỗ nào chưa chắc.
- **Nhóm A**: tự hỏi "vì sao". Mắt xích cốt lõi là chain 5 (3 lý do B-tree gãy) — nắm chắc là trả lời được câu phỏng vấn "vì sao vector cần index khác".
- **Nhóm B**: học mục 9. Chú ý **3 đính chính reading** — ghi đè, đừng nhớ bản sai.
- **Tái tạo**: dựng lại xương sống file 4 (mục 7), rồi **meta-xương sống 4 file** (encode → index → store&search → keyword/hybrid → RAG). Bốn mảnh, một Postgres.
