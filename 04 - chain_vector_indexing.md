# Indexing cho Vector Search: Binary Search → IVFFlat & HNSW — Chain gối đầu

> Vị trí series (mảnh 4/4 "làm search NHANH"): embed (tạo vector) → **index (bài này)** → store & query (pgvector) → keyword FTS → hybrid. Bài này tiếp cận từ *lý thuyết indexing* và bổ sung mắt xích reading bỏ qua (vì sao B-tree gãy với vector).
>
> **Đính chính reading (để phỏng vấn không nói sai):** (1) HNSW = Hierarchical Navigable Small **World** (reading viết "Small Words" sai). (2) Cạnh HNSW nối theo **bất kỳ metric nào bạn cấu hình** (ví dụ reading dùng `vector_cosine_ops` → cosine), không riêng Euclidean như reading viết. (3) Tầng HNSW đúng là **Layer 3 → 2 → 1 → 0** (trên xuống dưới).
> Mốc: pgvector **v0.8.x** (HNSW default; iterative scan; DiskANN qua `pgvectorscale`); giới hạn **2000 chiều** cho HNSW/IVFFlat index.

---

## CHAIN — Index là gì & vì sao đáng

**database index** là một cấu trúc dữ liệu tăng tốc tìm kiếm/truy xuất => nó chứa bản sao đã tổ chức của một phần dữ liệu, giúp định vị nhanh dòng cần tìm => giống **mục lục tra cứu cuối sách**: "database → trang 12, 45" → nhảy thẳng, không đọc cả cuốn => ý tưởng cốt lõi: trả một chút chi phí lưu trữ + cập nhật để đổi lấy tốc độ tìm kiếm khổng lồ => nhưng đổi chác đó chỉ đáng khi dữ liệu LỚN (không index thì linear search phải quét từng dòng)

## CHAIN — Linear search tốn thế nào

**linear search** so từng dòng một cho tới khi tìm thấy => trung bình phải đọc `(n+1)/2` dòng, tệ nhất `n` dòng => với `n = 1.000.000`, trung bình đọc ~500.000 dòng cho MỘT lần tìm => nhân với hàng nghìn query/giây → sập => đó là lý do "không index" bất khả thi khi dữ liệu lớn => cứu bằng cách sắp xếp dữ liệu rồi dùng **binary search**

## CHAIN — Binary search chạy tay ("Jack")

binary search cần dữ liệu đã SORT, rồi chia đôi liên tục => tìm "Jack" trong 20 tên đã sort: bước 1 xem đầu nửa sau = Isabella(I), Jack(J) đứng sau I → loại nửa đầu (loại 10 tên trong 1 bước) => bước 2 điểm giữa nửa sau = Michael(M), Jack trước M → loại nửa sau => bước 3–4 thu hẹp tiếp tới chạm Jack => **4 bước** thay vì 20 lần đọc => tổng quát binary search chỉ cần ~`log₂(n)` bước: 1 triệu tên → chỉ ~20 bước (2²⁰ ≈ 1.048.576) => đó là sức mạnh `O(log n)`

## CHAIN — Điều kiện tiên quyết binary search → B-tree

binary search chỉ chạy được vì tên sắp thứ tự trên MỘT trục (bảng chữ cái) => nhờ có **thứ tự tuyến tính (total order)**, mỗi bước trả lời được "mục tiêu trước hay sau điểm giữa?" => trong RDBMS, index kinh điển hiện thực ý này là **B-tree** (cây cân bằng) => B-tree = "binary search có cấu trúc cây", tìm/range scan ở `O(log n)` => B-tree tuyệt vời cho dữ liệu 1 chiều, sắp thứ tự được (số/chuỗi/ngày) => nhưng chính điều kiện "sắp xếp được trên một trục" là chỗ sẽ VỠ khi chuyển sang vector nhiều chiều

## CHAIN — Vì sao B-tree GÃY với vector (mắt xích cốt lõi)

vector nhiều chiều VỠ điều kiện "sắp xếp được trên một trục" vì 3 lý do => lý do 1: vector `[0.2,-0.7,...]` (384 chiều) sort theo *chiều nào*? không tồn tại một trục duy nhất để "gần trên trục = gần thật sự" => lý do 2: câu hỏi khác hẳn — B-tree hỏi "bằng đúng X / trong khoảng [a,b]", vector search hỏi "vector nào GẦN NHẤT về khoảng cách" (**nearest neighbor**) => lý do 3: **curse of dimensionality** — cây phân hoạch (kd-tree) ở chiều cao phải duyệt gần hết → không nhanh hơn brute-force => kết luận: với vector nhiều chiều ta TỪ BỎ tìm chính xác tuyệt đối, chuyển sang **Approximate Nearest Neighbor (ANN)** => ANN chấp nhận thỉnh thoảng bỏ sót hàng xóm thật để đổi lấy tốc độ; **IVFFlat** và **HNSW** là hai cách làm ANN

## CHAIN — IVFFlat cơ chế

**IVFFlat** = Inverted File with Flat compression => build: chạy **k-means** chia toàn bộ vector thành `lists` cụm, mỗi cụm quanh một **centroid** (tâm cụm) => search: so query với các centroid → chọn centroid gần nhất → chỉ brute-force TRONG cụm đó (thay vì cả triệu vector) => tham số `lists` (số cụm): gợi ý `rows/1000` cho ≤1M rows, `sqrt(rows)` cho >1M => tham số `probes` (số cụm dò khi search): mặc định 1, cao hơn → xét thêm cụm lân cận → recall tốt hơn nhưng chậm hơn (khởi đầu `sqrt(lists)`) => điểm yếu: hàng xóm thật nằm SÁT ranh giới cụm dễ bị bỏ sót (ở cụm bên cạnh mà `probes` không dò tới) => và IVFFlat *tĩnh*: thêm nhiều data sau build → phân hoạch lệch, recall tụt, phải `REINDEX`

## CHAIN — Exact vs Approximate

không index → exact search: pgvector quét hết, tính distance từng cái → **recall 100%** luôn đúng nhưng `O(n)` chậm khi lớn => có ANN index → approximate: nhanh (~log n hoặc theo cụm) nhưng **recall < 100%**, có thể thiếu vài hàng xóm thật => recall < 100% là *by design* không phải bug => vì vậy chọn index và tune tham số chính là cân bằng **accuracy ↔ speed ↔ resource** (linh hồn cả bài)

## CHAIN — Ba lỗi index

lỗi 1 — đánh index quá sớm trên dữ liệu nhỏ: vài nghìn–vài chục nghìn vector thì brute-force exact đã đủ nhanh + recall 100%, đánh ANN chỉ giảm chính xác mà không lợi tốc độ đáng kể => lỗi 2 — tạo IVFFlat khi bảng còn RỖNG: IVFFlat cần data để k-means tìm centroid, bảng trống → phân cụm vô nghĩa, recall tệ => khác HNSW: HNSW tạo được cả khi bảng rỗng (không cần data để build) => lỗi 3 — sai ops class so với toán tử: index `vector_cosine_ops` nhưng query bằng `<->` (L2) → Postgres bỏ qua index, rơi về seq scan => ops class phải khớp toán tử (`cosine_ops`↔`<=>`, `l2_ops`↔`<->`, `ip_ops`↔`<#>`), kiểm bằng `EXPLAIN ANALYZE`

## CHAIN — HNSW graph nhiều tầng

**HNSW** = Hierarchical Navigable Small **World** (không phải "Small Words") => là graph nhiều tầng: node = vector, cạnh nối node với hàng xóm gần theo *distance metric đã cấu hình* (cosine/L2/ip, không riêng Euclidean) => tầng trên cùng ít node, cạnh "đường dài" → nhảy xa nhanh về đúng vùng => tầng dưới cùng chứa mọi node, cạnh "đường ngắn" → tinh chỉnh tới hàng xóm chính xác => search: bắt đầu tầng trên, đi greedy tới node gần query nhất, rồi tụt xuống tầng dưới, lặp tới đáy (Layer 3→2→1→0) => nhờ vậy search ~`O(log n)` (như bay tuyến quốc tế về đúng thành phố rồi taxi tới đúng địa chỉ)

## CHAIN — HNSW tham số tune

tạo HNSW: `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops) WITH (m=16, ef_construction=64)` => `m` = số cạnh tối đa mỗi node ở mỗi tầng; cao → recall tốt, graph to, build/RAM tốn hơn => `ef_construction` = số ứng viên hàng xóm xét *khi xây* graph (64 mỗi lần chèn); cao → graph chất lượng hơn, recall tốt, build chậm => `ef_search` = số ứng viên xét *khi tìm* (query-time), mặc định 40; cao → recall tốt, chậm => `ef_search` là núm xoay recall–speed quan trọng nhất lúc chạy (`SET hnsw.ef_search = 100`)

## CHAIN — Curse of dimensionality (cơ chế)

ở 2D/3D chia không gian thành ô (kd-tree) rất hiệu quả => nhưng khi `d` lên hàng trăm–nghìn, thể tích không gian tăng theo hàm mũ → dữ liệu trở nên *thưa* => đồng thời khoảng cách giữa điểm gần nhất và xa nhất *co lại gần nhau* → "gần nhất" mất ý nghĩa phân biệt => cây phân hoạch phải duyệt gần hết nhánh → sập về `O(n)` => đó là lý do cần approximate + cấu trúc graph (HNSW) hoặc cụm (IVFFlat) thay vì cây chính xác

## CHAIN — DiskANN & giới hạn 2000 chiều

reading chỉ nêu IVFFlat & HNSW, nhưng 2026 có thêm **DiskANN** (qua `pgvectorscale` của Timescale) => DiskANN giữ phần lớn graph trên SSD thay vì RAM + binary quantization → index nhỏ hơn HNSW nhiều lần => và support vector tới 16000 chiều native => vì HNSW/IVFFlat của pgvector chỉ INDEX được tối đa **2000 chiều** => model 3072 chiều (OpenAI 3-large) dính lỗi này → workaround `halfvec` (index tới ~4000) hoặc DiskANN

## CHAIN — Chọn index theo quy mô

câu hỏi staff không phải "IVFFlat hay HNSW" trừu tượng mà "ở quy mô & ràng buộc CỦA TÔI thì cái nào" => dữ liệu nhỏ (<~10K–50K vector): ĐỪNG đánh index — brute-force exact đủ nhanh, recall 100%, không phải lo tune => trung bình (tới vài triệu): HNSW là mặc định — recall cao, latency thấp, chịu data động tốt (chấp nhận build lâu + RAM cao) => RAM hạn chế / build lại thường xuyên / data khá tĩnh: IVFFlat (nhẹ, build nhanh) hoặc DiskANN (SSD) khi dataset khổng lồ => chiều > 2000: halfvec + HNSW hoặc DiskANN

## CHAIN — Build cost & memory

HNSW build là `O(n log n)`, tốn hàng giờ ở quy mô lớn => tăng tốc: nạp data xong mới build + tăng `maintenance_work_mem` (≤50–60% RAM) + tăng `max_parallel_maintenance_workers` + `CREATE INDEX CONCURRENTLY` (không khóa write) => memory: HNSW graph phải nằm gọn trong RAM để nhanh => 100M vector × 1536 chiều × 4 byte ≈ 600GB chỉ riêng raw vectors + overhead graph => RAM là ràng buộc số một → cân quantization (halfvec/binary) hoặc DiskANN => data động: IVFFlat lệch cụm khi ghi nhiều → lịch reindex; HNSW chịu insert tốt hơn nhưng VACUUM chậm (reindex trước rồi vacuum)

## CHAIN — Monitoring recall

ANN không có "đúng/sai" nhị phân nên phải ĐO recall định kỳ => đo bằng cách ép exact search (`SET LOCAL enable_indexscan = off`) làm chuẩn vàng (ground truth) => so overlap top-k của kết quả có-index với exact → **recall@k**, dựng dashboard theo thời gian => recall tụt = tín hiệu tăng `ef_search`/`probes`/`m` hoặc reindex => không đo recall = **bay mù**: chất lượng search xuống dốc mà không lỗi nào báo

## CHAIN — Filtered search & overfiltering

kết hợp ANN với `WHERE` (vd `category='rare'`): index lấy `ef_search` hàng xóm *trước*, lọc *sau* => nếu filter hiếm thì trả về ít hơn `LIMIT` = **overfiltering** => khắc phục bằng **iterative index scan** (pgvector 0.8+): `SET hnsw.iterative_scan = relaxed_order` + `max_scan_tuples` => còn để prune sớm thì partition theo tenant/category (planner loại partition không liên quan trước khi search)

## CHAIN — Tổ chức & khép series

nói với PM/sếp: index như mục lục sách — trả chút chi phí (bộ nhớ, thời gian xây) đổi lấy tìm kiếm nhanh gấp nghìn lần => với vector còn đánh đổi thêm chút độ chính xác lấy tốc độ, điều chỉnh được bằng vài núm vặn; data nhỏ chưa cần => framing bằng **chi phí, tốc độ, đánh đổi độ chính xác** => roadmap: chọn/tune index không one-off — đo recall & latency liên tục, reindex khi data đổi (hạ tầng có vòng đời, không "set-and-forget") => mọi index này nằm trong Postgres nên backend team tự vận hành, khỏi thêm hệ thống search riêng => khép series 4 mảnh: embed → **index (bài này)** → store & query (pgvector) → keyword FTS → hybrid, tất cả trong một Postgres

---

## BẢNG — Big-O: brute-force vs IVFFlat vs HNSW

| Phương pháp | Query complexity | Build | Memory | Cần data để build? | Data động |
|---|---|---|---|---|---|
| Brute-force (no index) | `O(n·d)` | 0 | thấp | — | tốt (recall 100%) |
| **IVFFlat** | ~`O(lists·d + (n/lists)·probes·d)` | **nhanh** | **thấp** | **có** (k-means) | kém (lệch cụm) |
| **HNSW** | ~**`O(log n · d)`** | chậm | cao (graph in-RAM) | không | **tốt** |

`n` = số vector, `d` = số chiều. Chốt: HNSW nhanh & recall cao nhất nhưng tốn RAM + build lâu; IVFFlat nhẹ & build nhanh nhưng recall kém hơn và ngại data động.

## BẢNG — HNSW "tìm 12" (ví dụ reading, sửa nhãn tầng)

Danh sách đáy: 2, 4, 8, 12, 14, 16, 19.
- **Layer 3 (đỉnh):** chỉ có 4 → từ entry nhảy tới 4.
- **Layer 2:** có 4, 16 → 12 nằm giữa, đi về phía phù hợp.
- **Layer 1:** có 4, 12, 16 → thấy 12.
- **Layer 0 (đáy):** dừng ở **12** ✅.

## BẢNG — Code thuộc lòng

**(a) Hai câu tạo index:**
```sql
CREATE INDEX ON items USING hnsw    (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
SET hnsw.ef_search = 100;      -- recall knob (query-time)
SET ivfflat.probes  = 32;
-- IVFFlat: tạo SAU khi bảng có dữ liệu (cần data cho k-means)
```

**(b) Binary search từ số 0:**
```python
def binary_search(a, x):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == x: return mid
        if a[mid] < x:  lo = mid + 1
        else:           hi = mid - 1
    return -1
```

**(c) Đo recall (so ANN với exact):**
```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact = ground truth
SELECT id FROM items ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so overlap top-k với kết quả có index -> recall@10
```

## BẢNG — Khung system design (100M embedding, p95<100ms, filter tenant/category, update hàng ngày, RAM hạn chế)

1. **Clarify:** QPS? recall target? tần suất update? RAM/ngân sách? chiều vector?
2. **Chọn index:** RAM hạn chế + 100M → HNSW thuần có thể không vừa RAM → **DiskANN (pgvectorscale)** hoặc **HNSW + quantization** (halfvec/binary) + two-stage re-rank (thô bằng index nén → tinh bằng exact trên top-N).
3. **Filter:** partition theo tenant/category (planner prune) + bật **iterative_scan** chống overfiltering.
4. **Build/update:** nạp rồi build; `CREATE INDEX CONCURRENTLY`; lịch reindex cho data động; HNSW chịu insert hàng ngày tốt hơn IVFFlat.
5. **Latency:** tune `ef_search` theo SLA p95; read replicas cho search traffic.
6. **Monitoring:** recall@10 (so exact định kỳ), p99 latency, RAM/index size, tỉ lệ overfiltering.
7. **Khi nào không cần ANN:** một tenant chỉ vài nghìn vector → exact per-tenant còn nhanh & chính xác hơn. Không phải mọi thứ đều cần index = tư duy staff.

## BẢNG — Mental models

- **"Mục lục cuối sách"** → index là gì trong 1 câu.
- **"Sắp được thì binary search được"** → vì sao vector cần index khác.
- **"Chia quận, chỉ vào vài quận"** → IVFFlat (và điểm yếu ranh giới cụm).
- **"Đường cao tốc nhiều tầng"** → HNSW navigate từ tầng cao xuống thấp.
- **"ANN xuống cấp trong im lặng"** → vì sao phải monitor recall.

---

## TỪ KHÓA MỒI

- Index là gì → **"mục lục cuối sách"**
- Linear search → **"(n+1)/2"**
- Binary search chạy tay → **"4 bước tìm Jack"**
- Điều kiện tiên quyết → B-tree → **"total order"**
- Vì sao B-tree gãy → **"sort theo chiều nào"**
- IVFFlat cơ chế → **"centroid / probes"**
- Exact vs approximate → **"recall 100% by design"**
- Ba lỗi index → **"IVFFlat khi bảng rỗng"**
- HNSW graph nhiều tầng → **"Small World"**
- HNSW tham số → **"m / ef_construction / ef_search"**
- Curse of dimensionality → **"khoảng cách co lại"**
- DiskANN & 2000 chiều → **"SSD-based"**
- Chọn index theo quy mô → **"của TÔI thì cái nào"**
- Build cost & memory → **"600GB"**
- Monitoring recall → **"bay mù"**
- Filtered search → **"overfiltering"**
- Tổ chức & series → **"một Postgres cho tất cả"**

---

## ĐÃ PHỦ

**Lý thuyết index:** định nghĩa index + analogy mục lục sách; đánh đổi chi phí lưu/build lấy tốc độ, chỉ đáng khi lớn.

**Linear vs binary:** linear `(n+1)/2` trung bình, `n` tệ nhất, 1M → ~500K reads; binary search chạy tay "Jack" 4 bước, `log₂(n)`, 1M → ~20 bước; điều kiện total order; B-tree = binary search cấu trúc cây, `O(log n)`, 1 chiều sắp thứ tự được, ví dụ `CREATE INDEX users(email)`/`orders(created_at)`.

**Mắt xích cốt lõi — B-tree gãy với vector:** 3 lý do (không total order/sort chiều nào; câu hỏi NN chứ không khớp-khoảng; curse of dimensionality) → chuyển sang ANN.

**IVFFlat:** Inverted File Flat; k-means → lists cụm quanh centroid; probes dò cụm; `lists` = rows/1000 (≤1M) hay sqrt(rows) (>1M); `probes` mặc định 1, khởi đầu sqrt(lists); ví dụ hình 3 centroid; điểm yếu ranh giới cụm + tĩnh REINDEX.

**Exact vs approximate:** no index = exact recall 100% `O(n)`; ANN = approximate recall<100% by design; cân bằng accuracy↔speed↔resource; 3 lỗi (index quá sớm data nhỏ, IVFFlat bảng rỗng, sai ops class → seq scan).

**HNSW:** Small **World** (đính chính), graph nhiều tầng, cạnh theo metric cấu hình (đính chính), tầng cao đường dài / đáy đường ngắn, greedy từ trên xuống Layer 3→2→1→0, ví dụ "tìm 12" (2,4,8,12,14,16,19), ~`O(log n)`; tham số `m` (cạnh/tầng), `ef_construction` (build 64), `ef_search` (query 40).

**Big-O:** brute-force `O(n·d)`; IVFFlat `O(lists·d + (n/lists)·probes·d)`; HNSW `O(log n·d)` (bảng đầy đủ build/memory/cần data/data động).

**Curse of dimensionality:** thể tích mũ → thưa; khoảng cách gần-xa co lại → mất phân biệt; cây duyệt gần hết → `O(n)`.

**DiskANN & edge:** pgvectorscale SSD + binary quant, index nhỏ hơn, 16000 chiều native; giới hạn index 2000 chiều, model 3072 (OpenAI 3-large) dính → halfvec (~4000)/DiskANN; VACUUM HNSW chậm reindex trước; recall âm thầm tụt phải đo; sai ops class.

**Staff/scale:** chọn index theo quy mô (nhỏ đừng index / trung bình HNSW / RAM hạn chế IVFFlat hoặc DiskANN / >2000 halfvec); build cost `O(n log n)` + maintenance_work_mem ≤50-60% RAM + max_parallel_maintenance_workers + CREATE INDEX CONCURRENTLY; memory 600GB cho 100M×1536×4byte → quantization/DiskANN; monitoring recall (enable_indexscan=off, recall@k, "bay mù"); filtered search/overfiltering + iterative_scan + partition prune; tổ chức (framing chi phí/tốc độ/độ chính xác, index có vòng đời, một Postgres); khung system design 7 bước + "không phải mọi thứ đều cần index".

**Đính chính reading:** Small World (không Words); metric cấu hình (không riêng Euclidean); Layer 3→2→1→0.

**Học tiếp (bài nêu):** quantization (scalar/product/binary), two-stage retrieval & reranking, sharding vector (Citus/pgvectorscale), benchmark recall/latency (ann-benchmarks).
