# Phân tích chain học tập — Vector Search với PostgreSQL (pgvector)

> Mục tiêu file này KHÔNG phải tóm tắt nội dung, mà tách rõ:
> - **Nhóm A — Nhân quả**: chỗ suy ra được, chỉ cần hiểu "vì sao", KHÔNG học thuộc.
> - **Nhóm B — Ghi nhớ**: định nghĩa / tên gọi / con số / cú pháp — phần DUY NHẤT đáng học thuộc.
> - **Nhóm C — Một thứ hai tên**: hai mắt xích trông như nhân quả nhưng là cùng một thứ.
> - **⚠ Mắt xích bị nhảy**: chỗ logic hụt, thực chất là dữ kiện chèn vào — đừng cố hiểu, hãy ghi nhớ riêng.

Tài liệu có **27 chain**. Cách học đúng là **tái tạo** (che bản gốc, tự dựng lại chuỗi theo logic), không phải chép lại.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — Từ vấn đề tới embedding

**A — Nhân quả**
- `LIKE` chỉ khớp ký tự => "đồ giữ ấm mùa đông" trả 0 kết quả dù DB có "áo khoác lông vũ" — *vì* so ký tự không có khái niệm nghĩa nên không thấy từ đồng nghĩa.
- 0 kết quả cho synonym => cần máy tính hiểu ý nghĩa — *vì* keyword search miss những thứ cùng nghĩa khác chữ.

**B — Ghi nhớ**
- "vector search" — *tên gọi* của giải pháp cho nhu cầu hiểu nghĩa.
- "bản đồ ý nghĩa" — *mental model* (quy ước): mọi object là một điểm, thứ giống nghĩa nằm gần nhau.
- **embedding** — *định nghĩa*: dãy số biểu diễn ý nghĩa của một object (chữ/ảnh/âm thanh); chính là tọa độ điểm trên bản đồ.
- **vector** — *tên gọi*: chính là dãy số đó.

> Chain này **chủ yếu nhóm B**: sau 2 mắt xích nhân quả đầu, phần còn lại toàn là đặt tên / định nghĩa cho cùng một ý tưởng.

**Xương sống**: keyword search miss nghĩa → cần hiểu nghĩa → (đặt tên) vector search / embedding.

---

## Chain 2 — Vector, dimension, distance, k-NN

**A — Nhân quả**
- vector nhiều chiều => "bản đồ ý nghĩa" là hàng trăm/nghìn chiều chứ không phải 2D — *vì* dimension của model lớn.

**B — Ghi nhớ**
- **dimension** — *định nghĩa*: độ dài cố định của vector (số chiều).
- `text-embedding-3-small` = **1536 dimensions** — *con số* (ví dụ thực tế).
- **distance** — *định nghĩa*: cách đo "gần nhau" trong không gian nhiều chiều; distance nhỏ = giống nghĩa (quy ước diễn giải).
- **Nearest Neighbor Search** / **k-NN** — *tên gọi + viết tắt*: bài toán tìm K điểm gần nhất với query.

> Chain này gần như toàn nhóm B — chuỗi thuật ngữ nền tảng.

**Xương sống**: (gần như không có nhân quả) — vector → dimension → distance → k-NN, thuần định nghĩa.

---

## Chain 3 — Ai sinh embedding vs ai lưu

**A — Nhân quả**
- embedding model là thứ sinh vector => pgvector KHÔNG sinh embedding — *vì* việc "áo phao → [2,9,0]" là của model, không phải của kho lưu trữ.

**B — Ghi nhớ**
- **embedding model** — *định nghĩa*: mô hình AI đã học sẵn (OpenAI API, `sentence-transformers`, Cohere…) biến object thành vector.
- **pgvector** — *định nghĩa*: chỉ *lưu và tìm* vector; là **extension** mã nguồn mở của **PostgreSQL** (một RDBMS), thêm vào Postgres kiểu dữ liệu mới tên `vector`.
- **"kiến trúc 2 phần"** — *tên gọi*: model sinh vs pgvector lưu&tìm; junior hay nhầm gộp làm một.

> Chain này chủ yếu nhóm B (phân định vai trò + định nghĩa). Điểm cần khắc: pgvector **không** sinh embedding.

**Xương sống**: model sinh vector → pgvector chỉ lưu & tìm (kiến trúc 2 phần).

---

## Chain 4 — Tính tay L2 tới toán tử `<->`

**A — Nhân quả**
- distance nhỏ nhất => áo phao & khăn len là hàng xóm gần nhất, xa kem chống nắng — *vì* "gần = distance nhỏ", chọn min.

**B — Ghi nhớ**
- **Euclidean distance (L2)** = `sqrt(Σ(xᵢ - yᵢ)²)` — *công thức*; mental model "đường chim bay".
- Ví dụ tính tay query "giữ ấm"(2,8): áo phao=1.00, khăn len=1.00, kem chống nắng≈9.90 — *con số*.
- Toán tử `<->` = "L2 distance"; `ORDER BY cot_vector <-> query_vector LIMIT k` = "cho tôi k bản ghi giống nhất" — *cú pháp*.

**C — Một thứ hai tên**
- "tính tay L2 rồi chọn min" ⟺ "gõ `ORDER BY embedding <-> query`" — cùng MỘT phép tính, hai cách thể hiện (bằng tay vs bằng SQL). Đừng tìm chữ "vì" ở mắt xích này.

**Xương sống**: L2 = đo khoảng cách → distance nhỏ nhất là hàng xóm gần nhất → đó chính là `ORDER BY <->`.

---

## Chain 5 — Hello world SQL tới lý do cần index

**A — Nhân quả**
- bảng nhỏ => seq scan vẫn nhanh và chính xác tuyệt đối — *vì* ít hàng nên quét hết không tốn kém.
- quét hết = exact => 100% recall, KHÔNG cần index — *vì* đã so với mọi vector nên không bỏ sót.
- dữ liệu lớn => cần index, và khi đó đánh đổi độ chính xác lấy tốc độ — *vì* seq scan chậm dần theo n, mà index nhanh là index xấp xỉ (ANN).

**B — Ghi nhớ**
- Cú pháp vòng đời tối thiểu: `CREATE EXTENSION IF NOT EXISTS vector` (1 lần/database) → cột `vector(3)` → `INSERT '[a,b,c]'` (nháy đơn) → `ORDER BY embedding <-> '[...]' LIMIT k` — *cú pháp*.
- Ngưỡng bảng nhỏ: **< ~10K–50K hàng** dùng seq scan; **100% recall** khi exact — *con số*.

**Xương sống**: bảng nhỏ → seq scan exact 100% recall (không index) → bảng lớn → cần index → đánh đổi accuracy lấy speed.

---

## Chain 6 — Ba distance metric (cơ chế)

**A — Nhân quả**
- L2 nhạy độ dài => với text L2 không lý tưởng — *vì* text chỉ quan tâm hướng ngữ nghĩa, không quan tâm độ dài vector.
- text chỉ cần hướng => phải đo **góc** giữa 2 vector — *vì* bỏ độ dài đi thì chỉ còn góc.
- vector đã normalize length 1 => inner product ≡ cosine nhưng nhanh hơn — *vì* mẫu số `|a|·|b|` = 1, phép chia chuẩn hóa biến mất.
- ORDER BY tăng dần mà muốn "gần = nhỏ" => pgvector trả **negative** inner product (`<#>`) — *vì* inner product lớn = gần, phải đảo dấu cho nhất quán.

**B — Ghi nhớ**
- **cosine similarity** = `(a·b)/(|a|·|b|)`, nằm `[-1, 1]`, giá trị 1 = cùng hướng — *công thức + con số*.
- **cosine distance** = `1 - cosine_similarity`, nằm `[0, 2]` — *công thức*.
- **inner product** = `Σ aᵢbᵢ`; toán tử negative = `<#>` — *công thức + cú pháp*.
- cosine (`<=>`) là **mặc định cho text embeddings** — *quy ước*.

**Xương sống**: L2 nhạy độ dài → text cần đo góc → cosine → (nếu normalize) inner product nhanh hơn → negate để nhất quán ORDER BY.

---

## Chain 7 — Data modeling schema thật

**A — Nhân quả**
- lưu vector cạnh business data => join/filter được bằng SQL — *vì* chúng ở cùng một bảng/DB.
- nhét vector 768 vào cột `vector(1536)` => Postgres báo lỗi — *vì* dimension cột buộc khớp output của model.
- buộc khớp dim => đổi embedding model = phải re-embed toàn bộ corpus — *vì* model mới có dim / không gian khác, vector cũ vô nghĩa với model mới.

**B — Ghi nhớ**
- Schema thật: `tenant_id, title, content, category, created_at` + cột `embedding vector(1536)` — *ví dụ/định nghĩa*.
- Cột metadata đánh **B-tree index** để filter nhanh — *cú pháp/thực hành*.
- Con số `1536` trong `vector(1536)` **phải khớp** output dimension của model — *quy tắc*.

**Xương sống**: vector sống cạnh business data (join SQL) → dim cột phải khớp model → đổi model = re-embed cả corpus.

---

## Chain 8 — HNSW: build & tune

**A — Nhân quả**
- bảng lớn seq scan chậm => thêm ANN index — *vì* cần tránh quét tuần tự khi n lớn.
- KHÔNG dùng `SET` toàn cục trên production => vì rò rỉ qua connection pooler (PgBouncer) sang query người khác — *vì* pooler tái dùng connection, session sau kế thừa `SET` của session trước.

**B — Ghi nhớ**
- HNSW là default 2026; tạo bằng `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops)` — *cú pháp*.
- **ops class** (`vector_cosine_ops`) **phải khớp** toán tử query (`<=>`) — *quy tắc*.
- `m` = số kết nối tối đa mỗi node, **default 16** — *con số* (cao hơn → recall↑, index to, build chậm).
- `ef_construction` = độ cẩn thận khi build, **default 64** — *con số* (cao hơn → recall↑, build chậm).
- `ef_search` = độ cẩn thận khi query, **default 40** — *con số* (cao hơn → recall↑, latency↑).
- Chỉnh `ef_search` bằng `SET LOCAL` trong `BEGIN...COMMIT` — *cú pháp*.

> Ba tham số `m` / `ef_construction` / `ef_search` cùng một logic: đẩy recall lên → trả giá bằng dung lượng / thời gian build / latency. Hiểu một lần, dùng cho cả ba.

**Xương sống**: bảng lớn → thêm HNSW → ops class khớp toán tử → tune m/ef_* (recall ↔ chi phí) → dùng SET LOCAL, không SET global.

---

## Chain 9 — Pipeline Python thật

**A — Nhân quả**
- model `all-MiniLM-L6-v2` cho 384 chiều => cột phải là `vector(384)` — *vì* dim cột buộc khớp output model (áp lại quy tắc chain 7).
- đánh HNSW index **SAU** khi nạp data ban đầu => build nhanh hơn — *vì* build một lần trên data đã có rẻ hơn cập nhật graph từng dòng.
- query không chứa "áo khoác"/"khăn len" nhưng model hiểu nghĩa => trả đúng đồ mùa đông — *vì* embedding capture ngữ nghĩa chứ không phải từ khóa.

**B — Ghi nhớ**
- Pipeline: `sentence-transformers` (local, free) + `psycopg` v3 — *dữ kiện*.
- `register_vector(conn)` dạy psycopg hiểu `vector` ↔ numpy array — *cú pháp* (không có nó psycopg không hiểu kiểu vector).
- `all-MiniLM-L6-v2` = **384 chiều**; `model.encode(content)` → numpy 384 chiều — *con số + cú pháp*.
- Kết quả gọi là **semantic search** — *tên gọi*.

**Xương sống**: model output 384 → cột vector(384) → nạp data rồi mới build index → embed query cùng model → match theo nghĩa = semantic search.

---

## Chain 10 — Filtered search (siêu năng lực pgvector)

**A — Nhân quả**
- gộp `WHERE` + `ORDER BY <=> LIMIT` trong MỘT câu SQL => một round-trip, một transaction — *vì* Postgres xử lý filter và search cùng lúc.
- Pinecone tách vector store khỏi DB nghiệp vụ => phải 2 service, 2 network hop — *vì* IDs trả về từ vector store còn phải query lại DB để lấy chi tiết + filter.
- nhiều round-trip => nhiều glue code và failure mode — *vì* mỗi hop là một chỗ có thể hỏng và cần code nối.

**B — Ghi nhớ**
- **filtered search** — *định nghĩa*: filter business logic + similarity search trong một câu SQL.

**Xương sống**: filter + search trong 1 câu SQL → 1 round-trip/transaction → ít glue code, ít failure mode → đó là siêu năng lực pgvector.

---

## Chain 11 — Overfiltering & iterative scan

**A — Nhân quả**
- ANN lấy `ef_search` hàng xóm TRƯỚC rồi mới áp `WHERE` => filter hiếm (category 0.5%) chỉ còn 1–2 kết quả thay vì 10 — *vì* WHERE loại gần hết đám ứng viên đã bốc ra trước.

**B — Ghi nhớ**
- **overfiltering** — *tên gọi* của hiện tượng trên.
- **iterative index scan** — *tên gọi* giải pháp (tính năng chủ lực từ **pgvector 0.8.0**).
- `SET hnsw.iterative_scan = relaxed_order` (hoặc `strict_order`) kèm `hnsw.max_scan_tuples` — *cú pháp*.
- `relaxed_order` giữ **~95–99% chất lượng**, thường tốt nhất cho production — *con số*.

**Xương sống**: ANN bốc hàng xóm trước, filter sau → filter hiếm gây overfiltering → bật iterative scan để quét thêm.

---

## Chain 12 — Hai bẫy index còn lại

**A — Nhân quả**
- index `vector_l2_ops` nhưng query `<=>` (cosine) => Postgres **âm thầm bỏ qua index**, rơi về seq scan — *vì* planner chỉ dùng index khi ops class khớp toán tử; không khớp thì bỏ, không báo lỗi.
- ANN vẫn là approximate => recall có thể < 100% — *vì* bản chất xấp xỉ, đổi độ chính xác lấy tốc độ.
- junior test bảng nhỏ (exact) thấy đúng, lên production (index) thấy "sai" => hoảng — *vì* bảng nhỏ dùng exact 100% recall, production dùng ANN <100%, hai kết quả khác nhau.

**B — Ghi nhớ**
- Kiểm bằng `EXPLAIN ANALYZE` xem có dòng "Index Scan using ...hnsw..." — *cú pháp/công cụ*.
- Giải "sai" ở production: tăng `ef_search`/`m` hoặc đo recall — *khuyến nghị*.

> Đây là ví dụ điển hình của **lỗi im lặng**: hệ không báo, chỉ cho kết quả kém → phải chủ động kiểm.

**Xương sống**: ops class không khớp → Postgres âm thầm bỏ index → dùng EXPLAIN kiểm; ANN vốn <100% recall là by design, không phải bug.

---

## Chain 13 — Curse of dimensionality

**A — Nhân quả**
- brute-force `O(n·d)` với 100M × 1536 chiều => ~1.5×10¹¹ phép nhân/query, vô phương real-time — *vì* số phép tính tỉ lệ thẳng với cả n và d.
- không cứu được bằng kd-tree => vì **curse of dimensionality** — *vì* ở chiều cao mọi điểm gần như cách đều nhau, cây phân hoạch không gian mất tác dụng.
- exact quá đắt + cây vô dụng => chấp nhận **approximate NN (ANN)** — *vì* hai con đường chính xác đều tắc.

**B — Ghi nhớ**
- Big-O brute-force: `O(n·d)` (n = số vector, d = số chiều) — *con số*.
- **curse of dimensionality** — *định nghĩa*: d lớn → mọi điểm cách đều nhau → cây phân hoạch vô dụng.
- **ANN** — *tên gọi*: bỏ sót vài hàng xóm đổi lấy tốc độ gấp hàng nghìn lần.

**Xương sống**: exact O(n·d) quá đắt → kd-tree chết vì curse of dimensionality → chấp nhận ANN.

---

## Chain 14 — IVFFlat (chia quận)

**A — Nhân quả**
- k-means tạo `lists` centroid => mỗi vector gán vào quận có centroid gần nhất — *vì* đó là cách k-means phân cụm.
- IVFFlat chạy k-means => cần data sẵn để train (khác HNSW) — *vì* không có data thì không tính được centroid.
- query so với centroid, chọn `probes` quận gần nhất, brute-force TRONG các quận đó => bỏ qua phần lớn data → nhanh — *vì* chỉ quét vài quận thay vì toàn bộ.
- phân hoạch cứng => hàng xóm sát ranh giới quận bị bỏ sót — *vì* điểm gần nhưng rơi sang quận không được chọn.
- centroid cố định lúc build => thêm data mới → phân hoạch lệch, recall tụt, phải `REINDEX` — *vì* IVFFlat tĩnh, không tự cập nhật quận.

**B — Ghi nhớ**
- **IVFFlat** / `lists` (số quận) / `probes` (số quận query) — *tên gọi/tham số*.
- `probes` là núm xoay recall–speed: `probes=1` cực nhanh dễ trượt; `probes=lists` = exact nhưng mất hết lợi ích — *quy tắc*.

**Xương sống**: k-means chia quận (cần train) → query chỉ vào `probes` quận gần → nhanh nhưng miss ranh giới + tĩnh phải REINDEX.

---

## Chain 15 — HNSW (graph nhiều tầng)

**A — Nhân quả**
- graph nhiều tầng (tầng cao "cao tốc", tầng thấp "hẻm") => query bắt đầu tầng cao nhảy xa nhanh rồi tụt xuống tầng thấp tinh chỉnh — *vì* tầng cao ít node nối vùng xa, tầng thấp dày để dò chính xác.

**B — Ghi nhớ**
- **HNSW = Hierarchical Navigable Small World** — *tên gọi*.
- Build: chèn từng node, nối `m` hàng xóm gần nhất mỗi tầng; tầng chọn ngẫu nhiên theo phân phối mũ — *cơ chế/định nghĩa*.
- Search complexity **~O(log n)** — *con số vàng*.
- Build complexity **~O(n·log n·m·d)** — chậm và tốn RAM hơn IVFFlat — *con số*.

> Chain này chủ yếu nhóm B (cơ chế + Big-O). Điểm cần thuộc: `O(log n)` khi search.

**Xương sống**: graph nhiều tầng → navigate cao xuống thấp → search O(log n), build tốn hơn IVFFlat.

---

## Chain 16 — Edge case giới hạn 2000 chiều

**A — Nhân quả**
- `text-embedding-3-large` (3072 dims) => dính giới hạn index — *vì* 3072 > 2000.
- Matryoshka cắt vector ngắn lại (3072 → 1024) => vẫn dùng được — *vì* model dồn thông tin quan trọng về đầu vector.

**B — Ghi nhớ**
- Kiểu `vector` **lưu** tới **16000 chiều**, nhưng HNSW/IVFFlat chỉ **index** tối đa **2000 chiều** — *con số* (tương phản lưu vs index).
- `text-embedding-3-large` = **3072 dims** — *con số*.
- **halfvec** (fp16): index tới ~**4000 chiều**, giảm ~50% dung lượng, mất recall không đáng kể — *con số/kỹ thuật*.
- **Matryoshka**: cắt chiều — *tên gọi/kỹ thuật*.
- **DiskANN** (`pgvectorscale`): native tới **16000 chiều** — *con số/kỹ thuật*.

> Chain này chủ yếu nhóm B (dữ kiện giới hạn + 3 giải pháp). Con số cần thuộc: lưu 16000 / index 2000.

**Xương sống**: vector >2000 chiều không index được → chọn 1 trong 3 cách (halfvec / Matryoshka / DiskANN).

---

## Chain 17 — DiskANN / pgvectorscale

**A — Nhân quả**
- giữ phần lớn graph trên SSD + binary quantization => index nhỏ hơn HNSW ~9x — *vì* không phải nhét cả graph vào RAM và vector bị nén.

**B — Ghi nhớ**
- **DiskANN** qua extension **`pgvectorscale`** của Timescale — *tên gọi* ("player thứ 3" tầm staff).
- Index nhỏ hơn HNSW ~**9x** (21MB vs 193MB benchmark); support tới **16000 chiều native** — *con số*.

> Chain ngắn, chủ yếu nhóm B.

**Xương sống**: DiskANN để graph trên SSD + nén → index nhỏ hơn HNSW nhiều → chịu được dim rất lớn.

---

## Chain 18 — Normalize & NULL embedding

**A — Nhân quả**
- dùng `<#>` sau khi normalize insert => phải normalize CẢ vector query, nếu không kết quả sai lệch — *vì* `<#>` ≡ cosine chỉ đúng khi cả hai vector đều length 1.
- NULL thường do pipeline embed lỗi vài dòng => nên có job kiểm `WHERE embedding IS NULL` rồi re-embed — *vì* dòng lỗi im lặng không tự sửa.

**B — Ghi nhớ**
- Hàng có `embedding IS NULL` **không xuất hiện** trong ANN result — *dữ kiện/quy tắc*.

> ⚠ Mắt xích "normalize" → "NULL embedding" hơi **nhảy chủ đề**: nối chúng chỉ vì cùng là "bẫy vận hành" chứ không phải nhân quả. Ghi nhớ hai bẫy riêng biệt, đừng cố tìm chữ "vì" giữa chúng.

**Xương sống**: normalize thì phải normalize cả query → NULL embedding vô hình với ANN → job kiểm NULL định kỳ.

---

## Chain 19 — Scale & bottleneck RAM

**A — Nhân quả**
- HNSW graph phải nằm gọn trong RAM để nhanh => RAM là bottleneck số một — *vì* graph spill khỏi RAM thì tốc độ sụp.
- hết RAM => tính tới quantization và DiskANN — *vì* hai thứ này giảm dung lượng cần nạp.

**B — Ghi nhớ**
- < 10 triệu vectors: một node Postgres + HNSW là đủ và tốt nhất (query single-digit ms, 90% use case) — *con số/ngưỡng*.
- `1536 dims × 4 bytes ≈ 6KB/vector`; 100M ≈ **600GB** chỉ raw vectors — *con số*.

**Xương sống**: <10M thì 1 node là đủ → graph phải in-RAM → RAM là bottleneck → hết RAM thì quantization/DiskANN.

---

## Chain 20 — Build time & scale out

**A — Nhân quả**
- HNSW build `O(n log n)` tốn hàng giờ => tăng tốc bằng nạp data xong mới build + tăng `maintenance_work_mem` + tăng workers — *vì* build một lượt và nhiều RAM/luồng nhanh hơn.
- scale vertical làm trước => vì đơn giản và đủ xa — *vì* thêm RAM/CPU/SSD cho 1 instance không cần đổi kiến trúc.
- search là read-heavy => **read replicas** scale rất tốt — *vì* replica gánh được tải đọc.
- partition theo access pattern => planner **prune** partition không liên quan — *vì* query chỉ chạm partition khớp filter.
- 1 node không chứa nổi => **sharding** (Citus/PgDog/pgvectorscale) — *vì* vượt giới hạn một máy.

**B — Ghi nhớ**
- `maintenance_work_mem` nên **≤ 50–60% RAM** — *con số*.
- Bảng non-partitioned giới hạn **32TB**, partitioned gần như vô hạn — *con số*.
- Công cụ shard: Citus / PgDog / pgvectorscale — *tên gọi*.

**Xương sống**: build chậm → tối ưu build → scale vertical trước → hết thì horizontal (replica đọc → partition prune → shard).

---

## Chain 21 — Quantization & two-stage retrieval

**A — Nhân quả**
- N ứng viên nhỏ => re-rank rẻ mà chất lượng cuối cao — *vì* chỉ tính exact trên vài ứng viên, không phải cả corpus.

**B — Ghi nhớ**
- **quantization** — *tên gọi*: đòn bẩy cắt RAM/storage lớn nhất tầm staff.
- `halfvec` (fp16) cắt ~50%, ảnh hưởng recall rất nhỏ; **binary quantization** cắt tới ~32x nhưng mất recall — *con số*.
- **two-stage retrieval** (recall → precision) — *tên gọi/pattern*: stage 1 index nén lấy top-N nhanh thô; stage 2 re-rank exact + business signals (độ mới, popularity, quyền).

> Chain này chủ yếu nhóm B (định nghĩa pattern + con số). Điểm cần hiểu: binary quant mất recall nên **bắt buộc** đi kèm two-stage re-rank.

**Xương sống**: quantization cắt RAM nhưng mất recall → bù bằng two-stage (thô nhanh → tinh exact trên N nhỏ).

---

## Chain 22 — Cost & operational simplicity

**A — Nhân quả**
- không thêm hệ thống mới => bớt một failure domain, một on-call surface — *vì* mỗi hệ thống riêng là một chỗ hỏng và một ca trực.

**B — Ghi nhớ**
- Managed Postgres có pgvector sẵn: Supabase / Neon / RDS / Aurora, từ vài chục USD/tháng — *dữ kiện/tên gọi*.

> Chain ngắn. Ý chốt: giá trị lớn nhất của pgvector là **operational simplicity**, không phải hiệu năng đỉnh.

**Xương sống**: pgvector rẻ hơn vector DB riêng → không thêm hệ thống → ít failure domain.

---

## Chain 23 — Latency & monitoring recall

**A — Nhân quả**
- ANN không có "đúng/sai" nhị phân => phải ĐO recall định kỳ — *vì* chất lượng là liên tục, không thể assert đúng/sai.
- recall tụt => tín hiệu cần re-tune / re-index — *vì* graph/phân hoạch đã lệch khỏi trạng thái tốt.

**B — Ghi nhớ**
- Đo latency theo **p50/p95/p99**, không đo trung bình — *quy tắc* (trung bình giấu đuôi latency).
- Đo recall: ép exact bằng `SET LOCAL enable_indexscan = off` làm ground truth, so overlap → **recall@k** — *cú pháp/định nghĩa*.

**Xương sống**: đo latency percentile → ANN không nhị phân nên phải đo recall@k (exact làm ground truth) → recall tụt = tín hiệu re-tune.

---

## Chain 24 — Failure modes cần chuẩn bị

**A — Nhân quả**
- embedding model đổi phiên bản => vector cũ "lệch không gian" với vector mới => kết quả rác — *vì* hai model dùng không gian khác nhau, trộn lẫn thì so sánh vô nghĩa.

**B — Ghi nhớ (4 failure mode)**
- (1) đổi model → lệch không gian → rác.
- (2) `VACUUM` trên HNSW rất chậm → reindex trước rồi mới vacuum.
- (3) pipeline embed lỗi âm thầm → để lại `NULL`/vector rác.
- (4) `SET` toàn cục rò qua connection pooler sang query người khác.

> ⚠ Đây thực chất là **danh sách 4 mục**, không phải chuỗi nhân quả — các dấu `=>` chỉ ngăn cách các mode. Đừng cố nối "vì… nên…" giữa mode 1 và mode 2. Học như một checklist.

**Xương sống**: (không phải nhân quả liền mạch) — checklist 4 failure mode: đổi model / VACUUM HNSW / NULL im lặng / SET global rò pooler.

---

## Chain 25 — Khi nào NÊN / KHÔNG NÊN dùng pgvector

**A — Nhân quả**
- vector search **LÀ** workload chính => hệ chuyên dụng (Pinecone/Weaviate/Qdrant/Milvus) tối ưu hơn — *vì* khi vector là cả sản phẩm, cần peak performance mà pgvector không nhắm tới.

**B — Ghi nhớ (tiêu chí quyết định)**
- **NÊN**: vector search là *một tính năng* trong app đã chạy Postgres; cần filter/join với business data; < 10–50M vectors; muốn ít hệ thống + ACID.
- **KHÔNG NÊN**: > 50M–hàng tỷ vectors cần globally distributed multi-region; cần hybrid search (vector + BM25) built-in mạnh; managed scaling vượt provider; hoặc vector search là workload chính.
- Câu chốt cần thuộc: **pgvector thắng ở operational simplicity, không phải peak performance; chọn khi vector là feature, không phải cả sản phẩm.**

> ⚠ Phần lớn chain này là **danh sách tiêu chí**, không phải nhân quả — các `=>` chỉ liệt kê điều kiện. Chỉ mắt xích cuối là suy luận thật.

**Xương sống**: vector là feature + <50M + cần join/ACID → chọn pgvector; vector là cả sản phẩm / >50M / multi-region → chọn hệ chuyên dụng.

---

## Chain 26 — Tổ chức & quyết định chọn model

**A — Nhân quả**
- đổi model = re-embed toàn bộ corpus (tốn API + downtime) => nên chốt model sớm + version hóa embedding + kế hoạch backfill — *vì* làm lại cả corpus rất đắt, phải chuẩn bị trước.
- giữ vector trong Postgres => team backend hiện tại tự lo, không cần tuyển riêng vector DB stack — *vì* cùng một công nghệ đội đã biết vận hành.

**B — Ghi nhớ**
- Giải thích cho PM/sếp bằng **risk & cost**, không bằng jargon — *khuyến nghị*.
- "Quyết định nặng ký nhất không phải chọn index mà là chọn embedding model" — *nhận định cần thuộc*.
- Version hóa embedding: cột `embedding_v2`, blue-green khi đổi — *thực hành*.

**Xương sống**: chọn model quan trọng hơn chọn index → đổi model = re-embed cả corpus → chốt sớm, version hóa, backfill.

---

## Chain 27 — RAG, ANN, Recall (thuật ngữ chốt)

**A — Nhân quả**
- (không có) — chain thuần định nghĩa.

**B — Ghi nhớ**
- vector search = bước **"retrieval"** trong RAG — *định nghĩa*.
- **RAG** = Retrieval-Augmented Generation — *viết tắt*.
- **ANN** = Approximate Nearest Neighbor: đánh đổi recall lấy tốc độ, ngược với exact — *viết tắt/định nghĩa*.
- **recall** = % hàng xóm thật mà search tìm được; là thước đo chất lượng của ANN — *định nghĩa*.

> Chain này **hoàn toàn nhóm B** — chỉ là bảng chú giải viết tắt. Học thuộc như từ vựng.

**Xương sống**: (không có nhân quả) — retrieval → RAG → ANN → recall, thuần thuật ngữ.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC

Nối các xương sống thành **một mạch nhân quả duy nhất** — đây là bộ khung suy ra được, phần còn lại chỉ là từ vựng/số liệu treo lên khung:

> keyword search miss nghĩa → cần hiểu nghĩa → **vector search / embedding** (điểm trên "bản đồ ý nghĩa") → đo gần bằng **distance** → tìm k gần nhất = **k-NN** → **embedding model sinh, pgvector chỉ lưu & tìm** → bảng nhỏ **seq scan exact 100% recall** (không index) → bảng lớn seq scan chậm → **cần index = ANN = đánh đổi accuracy lấy speed** → exact quá đắt `O(n·d)` + kd-tree chết vì **curse of dimensionality** → chấp nhận **ANN** → chọn thuật toán: **IVFFlat** (chia quận, tĩnh, miss ranh giới) / **HNSW** (graph nhiều tầng, `O(log n)`, động, default) / **DiskANN** (SSD, dim lớn) → **tune** (m / ef_construction / ef_search: recall ↔ speed ↔ RAM) → **RAM là bottleneck** → scale **vertical → horizontal** (replica đọc, partition prune, shard) → hết RAM: **quantization** → binary quant mất recall → bù bằng **two-stage re-rank** → **đo recall@k** định kỳ → chọn pgvector **khi vector là feature, không phải cả sản phẩm**.

Nắm được mạch này là dựng lại được ~80% tài liệu. Mọi con số (1536, m=16…), cú pháp (`<=>`, `CREATE INDEX`…) và tên riêng chỉ là chi tiết treo lên khung — tra khi cần, không cần "hiểu".

---

## 8. MẪU HÌNH LẶP LẠI

Gom các chain nói **cùng một nguyên lý** dù tên khác nhau — hiểu một lần, dùng cho nhiều chỗ:

**Mẫu hình 1 — Núm vặn recall ↔ tốc độ/tài nguyên.**
Xuất hiện ở: exact vs ANN (chain 5, 13), `ef_search`/`m`/`ef_construction` của HNSW (chain 8), `probes` của IVFFlat (chain 14), quantization halfvec/binary (chain 16, 21), `relaxed_order` của iterative scan (chain 11).
*Vì sao là một*: mọi tham số này chỉ đang trượt trên **cùng một trục** — đẩy recall lên thì trả giá bằng latency / RAM / dung lượng, và ngược lại. Không có bữa trưa miễn phí.

**Mẫu hình 2 — Lỗi im lặng (silent failure).**
Xuất hiện ở: ops class không khớp → âm thầm bỏ index (chain 12), NULL embedding vô hình với ANN (chain 18, 24), `SET` global rò qua PgBouncer (chain 8, 24), pipeline embed lỗi để lại NULL/rác (chain 18, 24), ANN recall<100% khiến junior tưởng "sai" (chain 12).
*Vì sao là một*: hệ **không báo lỗi**, chỉ trả kết quả thiếu/kém → phải **chủ động kiểm** (`EXPLAIN ANALYZE`, đo recall@k, job kiểm NULL). Bài học chung: im lặng ≠ ổn.

**Mẫu hình 3 — Vector gắn chặt với model → đổi model = làm lại corpus.**
Xuất hiện ở: dim cột phải khớp model (chain 7, 9), failure mode 1 "lệch không gian" (chain 24), quyết định chọn model nặng hơn chọn index (chain 26).
*Vì sao là một*: embedding chỉ có nghĩa trong không gian của model sinh ra nó; đổi model = không gian khác = **re-embed toàn bộ**. Hệ quả: version hóa embedding, chốt model sớm, blue-green.

**Mẫu hình 4 — Ops class / metric phải khớp toán tử query.**
Xuất hiện ở: build index đúng ops class (chain 8), sai ops class → bỏ index (chain 12).
*Vì sao là một*: index chỉ dùng được nếu ops class lúc build (`vector_cosine_ops`) khớp toán tử lúc query (`<=>`).

**Mẫu hình 5 — Lọc/thô trước, tinh/đắt sau.**
Xuất hiện ở: two-stage retrieval recall→precision (chain 21), partition prune trước rồi search (chain 20), filter rồi ANN (chain 11).
*Vì sao là một*: đều là "thu hẹp tập ứng viên bằng bước rẻ trước, rồi mới bỏ công tính chính xác trên tập nhỏ".

**Mẫu hình 6 — Operational simplicity là lý do tồn tại của pgvector.**
Xuất hiện ở: kiến trúc 2 phần trong một DB (chain 3), filtered search một câu SQL (chain 10), rẻ + ít failure domain (chain 22), chọn khi vector là feature (chain 25).
*Vì sao là một*: lợi thế xuyên suốt không phải tốc độ mà là **một hệ thống, một câu SQL, một transaction**.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

Đây là tài liệu ôn tập cô đọng (gộp cả các BẢNG/ĐÃ PHỦ trong file gốc).

### Con số
- `text-embedding-3-small` = **1536** dims · `text-embedding-3-large` = **3072** dims · `all-MiniLM-L6-v2` = **384** dims.
- HNSW default: **m = 16**, **ef_construction = 64**, **ef_search = 40**.
- IVFFlat: `lists` = rows/1000 (≤1M) hoặc sqrt(rows) (>1M); `probes` khởi đầu ≈ sqrt(lists).
- Bảng nhỏ dùng exact: **< 10K–50K** hàng; exact cho **100% recall**.
- Kiểu `vector` **lưu** tới **16000** chiều; HNSW/IVFFlat **index** tối đa **2000** chiều.
- `halfvec` index tới ~**4000** chiều, cắt ~**50%** dung lượng · binary quantization cắt tới ~**32x** · Matryoshka cắt tùy tỉ lệ.
- `relaxed_order` giữ **~95–99%** chất lượng.
- Kích thước: `1536 × 4 bytes ≈ 6KB/vector`; 100M ≈ **600GB**; 50M ≈ **300GB** raw.
- `maintenance_work_mem` **≤ 50–60% RAM**; non-partitioned giới hạn **32TB**.
- DiskANN index nhỏ hơn HNSW ~**9x** (21MB vs 193MB).
- Ngưỡng scale: **<10M** (1 node đủ, 90% use case) · **<50M** · **>50M** (cân hệ khác).
- L2 ví dụ tính tay: áo phao **1.00**, khăn len **1.00**, kem chống nắng **≈9.90**.
- cosine similarity ∈ **[-1, 1]**; cosine distance ∈ **[0, 2]**.
- Big-O: brute-force **O(n·d)** · HNSW query **O(log n)**, build **O(n·log n·m·d)** · IVFFlat query **O(lists·d)+O((n/lists)·probes·d)**.

### Định nghĩa & tên gọi
- **embedding** (dãy số biểu diễn nghĩa) · **vector** · **dimension** · **distance** · **Nearest Neighbor / k-NN**.
- **vector search / similarity search**; mental model **"bản đồ ý nghĩa"**.
- **Kiến trúc 2 phần**: embedding model *sinh* vs pgvector *lưu & tìm*.
- **pgvector** = extension của **PostgreSQL** (RDBMS), thêm kiểu `vector`.
- **L2 / Euclidean** · **cosine similarity** = `(a·b)/(|a||b|)` · **cosine distance** = `1 - sim` · **inner product** = `Σaᵢbᵢ`.
- Toán tử: **`<->`** L2 · **`<=>`** cosine (mặc định text) · **`<#>`** negative inner product (khi đã normalize).
- ops class: `vector_l2_ops` · `vector_cosine_ops` · `vector_ip_ops`.
- **HNSW** = Hierarchical Navigable Small World · **IVFFlat** · **DiskANN** (`pgvectorscale`).
- **overfiltering** · **iterative scan** (`relaxed_order`/`strict_order`) · **curse of dimensionality**.
- **two-stage retrieval** (recall → precision) · **re-rank** · **semantic search** · **quantization** (halfvec / binary / Matryoshka).
- **RAG** = Retrieval-Augmented Generation · **ANN** = Approximate Nearest Neighbor · **recall** = % hàng xóm thật tìm được · **recall@k**.
- Tham số: `m`, `ef_construction`, `ef_search`, `lists`, `probes`, `max_scan_tuples`.

### Cú pháp
- `CREATE EXTENSION IF NOT EXISTS vector;` (1 lần/database)
- `CREATE TABLE ... (embedding vector(1536));`
- `INSERT` vector dạng `'[a,b,c]'` (nháy đơn)
- `SELECT ... ORDER BY embedding <=> '[...]' LIMIT k;`
- `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops);` (ops class khớp toán tử)
- `SET LOCAL hnsw.ef_search = ...;` trong `BEGIN...COMMIT` — **không** dùng `SET` global.
- `SET hnsw.iterative_scan = relaxed_order;` + `hnsw.max_scan_tuples`.
- `register_vector(conn)` (psycopg v3) · `model.encode(content)` → numpy.
- `SET LOCAL enable_indexscan = off;` để ép exact làm ground truth đo recall.
- `EXPLAIN ANALYZE` để xác nhận có "Index Scan using ...hnsw...".
- Filtered search: `WHERE ... ORDER BY embedding <=> :q LIMIT k;` (một câu).

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Cách dùng: che toàn bộ tài liệu, trả lời bằng lời của mình. Chỗ nào **ấp úng = lỗ hổng cần đào**, không phải "chưa thuộc" — hãy quay lại đúng chain đó và tự dựng lại chuỗi theo logic.

1. Vì sao keyword search (`LIKE`) trả 0 kết quả cho "đồ giữ ấm mùa đông" dù DB có "áo khoác lông vũ"?
2. Vì sao bảng nhỏ (<10K–50K) cho kết quả chính xác tuyệt đối mà **không cần** index?
3. Vì sao khi dữ liệu lớn, thêm index lại đồng nghĩa với "đánh đổi độ chính xác lấy tốc độ"?
4. Vì sao L2 không lý tưởng cho text embeddings, còn cosine thì hợp?
5. Vì sao pgvector trả về **negative** inner product (`<#>`) thay vì inner product thường?
6. Vì sao đổi embedding model buộc phải re-embed toàn bộ corpus?
7. Vì sao ops class sai (`vector_l2_ops` mà query `<=>`) khiến Postgres bỏ qua index — và vì sao đây là lỗi **im lặng**?
8. Vì sao exact search không cứu được bằng kd-tree ở chiều cao?
9. Vì sao IVFFlat cần data sẵn để build còn HNSW thì không?
10. Vì sao IVFFlat bỏ sót hàng xóm nằm **sát ranh giới quận**?
11. Vì sao HNSW search chỉ ~O(log n) nhờ cấu trúc nhiều tầng?
12. Vì sao RAM là bottleneck số một của HNSW, và khi nào phải chuyển sang quantization/DiskANN?
13. Vì sao binary quantization **bắt buộc** đi kèm two-stage re-rank?
14. Vì sao phải đo latency theo p95/p99 thay vì trung bình, và vì sao phải đo recall định kỳ?
15. Vì sao filtered search "một câu SQL" là siêu năng lực của pgvector so với Pinecone?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- **Nhóm A**: tự hỏi "vì sao mũi tên này tồn tại", hiểu là xong — **đừng chép lại**.
- **Nhóm B**: học thuộc riêng như từ vựng (con số / định nghĩa / cú pháp ở mục 9). Số lượng ít, gom một chỗ.
- **Cách học đúng là tái tạo**: che bản gốc, tự dựng lại xương sống toàn cục (mục 7) theo logic, không nhớ từng chữ. Chép lại nhiều lần là lãng phí.
- Ưu tiên nắm **6 mẫu hình lặp lại** (mục 8) — hiểu một lần, áp cho nhiều chain, giảm hẳn số thứ phải nhớ.
