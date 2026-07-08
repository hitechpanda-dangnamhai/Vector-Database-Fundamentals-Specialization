# Vector Search với PostgreSQL (pgvector) — Chain gối đầu

> Phiên bản mốc: pgvector **v0.8.x** (nhánh 0.8.4). HNSW là default index khuyên dùng.
> Tính năng mới đáng chú ý: iterative index scans (0.8.0), `halfvec`/`sparsevec`, binary quantization, DiskANN qua `pgvectorscale`.

---

## CHAIN — Từ vấn đề tới embedding

SQL truyền thống chỉ khớp ký tự/từ khóa (`WHERE ... LIKE`) => khớp từ khóa không hiểu *ý nghĩa* nên "đồ giữ ấm mùa đông" trả 0 kết quả dù DB có "áo khoác lông vũ" => cần máy tính hiểu ý nghĩa giống con người => nhu cầu hiểu ý nghĩa sinh ra **vector search** => vector search đặt mọi object lên một "bản đồ ý nghĩa" => trên bản đồ ý nghĩa, thứ nào giống nghĩa thì nằm gần nhau => tọa độ của mỗi điểm trên bản đồ chính là **embedding** => embedding là một dãy số biểu diễn ý nghĩa của một object (chữ/ảnh/âm thanh) => dãy số đó chính là một **vector**

## CHAIN — Vector, dimension, distance, k-NN

vector là một mảng số thực có độ dài cố định => độ dài cố định đó gọi là **dimension** (số chiều) => ví dụ model `text-embedding-3-small` của OpenAI cho vector **1536 dimensions** => vì nhiều chiều nên "bản đồ ý nghĩa" thực chất là hàng trăm/nghìn chiều chứ không phải 2D => trên không gian nhiều chiều đó ta đo "gần nhau" bằng **distance** => distance nhỏ nghĩa là 2 vector giống nghĩa => bài toán tìm K điểm gần nhất với một điểm query gọi là **Nearest Neighbor Search** => K điểm gần nhất viết tắt là **k-NN**

## CHAIN — Ai sinh embedding vs ai lưu

embedding được sinh ra bởi một **embedding model** (mô hình AI đã học sẵn: OpenAI API, `sentence-transformers`, Cohere...) => embedding model biến "áo phao" → `[2,9,0]`, và việc này **pgvector KHÔNG làm** => pgvector chỉ *lưu và tìm* vector, không tự sinh embedding => đây là kiến trúc 2 phần mà junior hay nhầm gộp làm một => phần lưu-và-tìm do pgvector đảm nhận => pgvector là một **extension** mã nguồn mở của **PostgreSQL** (một RDBMS phổ biến) => nó thêm vào Postgres một kiểu dữ liệu mới tên `vector`

## CHAIN — Tính tay L2 tới toán tử `<->`

**Euclidean distance (L2)** = `sqrt(Σ (xᵢ - yᵢ)²)` => công thức này tính khoảng cách "đường chim bay" giữa 2 điểm => áp cho query "giữ ấm"(2,8): distance tới áo phao = 1.00, khăn len = 1.00, kem chống nắng ≈ 9.90 => distance nhỏ nhất → áo phao & khăn len là hàng xóm gần nhất, xa kem chống nắng => đây *chính xác* là việc pgvector làm khi ta gõ `ORDER BY embedding <-> query` => toán tử `<->` nghĩa là "L2 distance" => `ORDER BY cot_vector <-> query_vector LIMIT k` = "cho tôi k bản ghi giống nhất"

## CHAIN — Hello world SQL tới lý do cần index

`CREATE EXTENSION IF NOT EXISTS vector` bật extension (chạy 1 lần mỗi database) => sau khi bật, tạo cột kiểu `vector(3)` trong bảng => `INSERT` vector viết dạng `'[a,b,c]'` trong dấu nháy đơn => query bằng `ORDER BY embedding <-> '[...]' LIMIT k` => toán tử `<->` là trái tim của mọi similarity search => với bảng nhỏ (< ~10K–50K hàng) Postgres quét tuần tự (sequential scan) vẫn nhanh và cho kết quả *chính xác tuyệt đối* => exact search cho **100% recall** và **KHÔNG cần index** => index chỉ cần khi dữ liệu lớn, và khi đó ta **đánh đổi độ chính xác lấy tốc độ**

## CHAIN — Ba distance metric (cơ chế)

**L2** (`<->`) nhạy cả *hướng* lẫn *độ dài* của vector => vì nhạy độ dài nên với text (chỉ quan tâm hướng ngữ nghĩa) L2 không lý tưởng => text cần đo **góc** giữa 2 vector => đo góc bỏ độ dài chính là **cosine similarity** = `(a·b)/(|a|·|b|)` => cosine similarity nằm trong `[-1, 1]`, giá trị 1 = cùng hướng => pgvector trả **cosine distance** = `1 - cosine_similarity`, nằm `[0, 2]` => cosine (`<=>`) là lựa chọn **mặc định cho text embeddings** => nếu vector đã normalize về length 1 thì **inner product** ≡ cosine nhưng tính nhanh hơn (bỏ được phép chia chuẩn hóa) => inner product = `Σ aᵢbᵢ`; pgvector trả về bản **negative** (`<#>`) để "nhỏ hơn = gần hơn" cho nhất quán với `ORDER BY`

## CHAIN — Data modeling schema thật

thực tế lưu vector **cạnh** business data chứ không lưu mỗi vector => vector sống chung nhà với business data nên join/filter được bằng SQL => schema thật gồm `tenant_id, title, content, category, created_at` + cột `embedding vector(1536)` => các cột metadata (`tenant_id, category, created_at`) được đánh **B-tree index** để filter nhanh => còn con số `1536` trong `vector(1536)` **phải khớp** đúng output dimension của model => sai dim (nhét vector 768 vào cột 1536) → Postgres báo lỗi => vì buộc khớp dim, **đổi embedding model = phải re-embed toàn bộ corpus** (migration tốn kém)

## CHAIN — HNSW: build & tune

bảng lớn thì seq scan chậm nên thêm **ANN index**; HNSW là default 2026 => tạo bằng `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops)` => **ops class** (`vector_cosine_ops`) PHẢI khớp toán tử query (`<=>`) => tham số build `m` = số kết nối tối đa mỗi node trong graph (default 16); cao hơn → recall tốt, index to, build chậm => tham số build `ef_construction` = độ "cẩn thận" khi xây graph (default 64); cao hơn → recall tốt, build chậm => tham số query-time `ef_search` (default 40); cao hơn → recall tốt, latency chậm => chỉnh `ef_search` nên dùng `SET LOCAL` bên trong `BEGIN...COMMIT` => KHÔNG dùng `SET` toàn cục trên production vì nó rò rỉ qua connection pooler (PgBouncer) sang query của người khác

## CHAIN — Pipeline Python thật

pipeline thật: `sentence-transformers` sinh embedding (chạy local, miễn phí) + `psycopg` v3 nói chuyện với Postgres => `register_vector(conn)` dạy psycopg hiểu kiểu `vector` ↔ numpy array => model `all-MiniLM-L6-v2` cho vector 384 chiều nên cột phải là `vector(384)` => `model.encode(content)` trả về numpy array 384 chiều để `INSERT` => đánh HNSW index **SAU khi** nạp dữ liệu ban đầu → build nhanh hơn => query: embed câu hỏi rồi `ORDER BY embedding <=> query_vec` => câu query không chứa từ "áo khoác"/"khăn len" nhưng model hiểu *nghĩa* → trả đúng đồ mùa đông => đó là **semantic search** thật sự

## CHAIN — Filtered search (siêu năng lực pgvector)

filtered search = `WHERE` lọc business logic + `ORDER BY embedding <=> query LIMIT` trong **MỘT** câu SQL => một câu = một round-trip = một transaction => Pinecone thì phải: query vector → nhận IDs → query lại DB nghiệp vụ để lấy chi tiết + filter → 2 service, 2 network hop => nhiều round-trip sinh nhiều glue code và nhiều failure mode => vì vậy **filter/join vector với business data trong 1 câu SQL** chính là siêu năng lực của pgvector

## CHAIN — Overfiltering & iterative scan

ANN index (HNSW) lấy `ef_search` hàng xóm **TRƯỚC**, rồi Postgres mới áp `WHERE` => nếu filter hiếm (vd category chỉ chiếm 0.5% data) thì sau khi lọc chỉ còn 1–2 kết quả thay vì 10 => hiện tượng này gọi là **overfiltering** => fix bằng bật **iterative index scan** (tính năng chủ lực từ pgvector 0.8.0) => `SET hnsw.iterative_scan = relaxed_order` (hoặc `strict_order`) kèm `hnsw.max_scan_tuples` (trần tuple quét thêm) => `relaxed_order` cho tốc độ tốt, giữ ~95–99% chất lượng — thường là lựa chọn production tốt nhất

## CHAIN — Hai bẫy index còn lại

đánh index `vector_l2_ops` nhưng query bằng `<=>` (cosine) → ops class không khớp toán tử => không khớp thì Postgres **âm thầm BỎ QUA index**, rơi về sequential scan (chậm/kết quả bất ngờ) => kiểm bằng `EXPLAIN ANALYZE` xem có dòng "Index Scan using ...hnsw..." không => ngay cả khi index chạy đúng, ANN vẫn là **approximate** nên recall có thể < 100% => junior test trên bảng nhỏ (exact) thấy đúng, lên production (index) thấy "sai" rồi hoảng => đây là *by design*; giải: tăng `ef_search`/`m` hoặc đo recall để biết mình đang ở đâu

## CHAIN — Curse of dimensionality

brute-force exact search có độ phức tạp `O(n · d)` (n = số vector, d = số chiều) => với 100 triệu vector × 1536 chiều, mỗi query ~1.5×10¹¹ phép nhân → vô phương cho real-time => và không cứu được bằng cây kd-tree vì **curse of dimensionality** => curse of dimensionality: khi `d` lớn (hàng trăm–nghìn), mọi điểm gần như *cách đều nhau*, cây phân hoạch không gian mất tác dụng => vì exact quá đắt và cây vô dụng, ta chấp nhận **approximate NN (ANN)** => ANN bỏ sót vài hàng xóm để đổi lấy tốc độ gấp hàng nghìn lần

## CHAIN — IVFFlat (chia quận)

IVFFlat chia không gian thành các "quận" bằng **k-means** (tìm `lists` centroid) => mỗi vector được gán vào quận có centroid gần nó nhất => vì phải chạy k-means nên IVFFlat **cần data sẵn để train** (khác HNSW) => query: so query với các centroid → chọn `probes` quận gần nhất => chỉ brute-force **TRONG** các quận đã chọn, bỏ qua phần lớn dữ liệu => `probes` là núm xoay recall–speed: `probes=1` cực nhanh nhưng dễ trượt, `probes=lists` = exact nhưng mất hết lợi ích => điểm yếu chí mạng: hàng xóm nằm **sát ranh giới quận** bị bỏ sót => và IVFFlat *tĩnh*: thêm nhiều data mới sau build → phân hoạch lệch, recall tụt, phải `REINDEX`

## CHAIN — HNSW (graph nhiều tầng)

HNSW = **Hierarchical Navigable Small World**, một graph nhiều tầng => tầng trên cùng ít node = "đường cao tốc" nối các vùng xa; tầng dưới cùng chứa mọi node với đường "hẻm" ngắn => query bắt đầu ở tầng cao (nhảy xa nhanh về đúng vùng) rồi tụt dần xuống tầng thấp để tinh chỉnh => build: chèn từng node, nối nó với `m` hàng xóm gần nhất ở mỗi tầng => tầng của mỗi node chọn ngẫu nhiên theo phân phối mũ (ít node lên tầng cao) => search complexity **~O(log n)** — con số vàng cần nhớ => build complexity **~O(n · log n · m · d)** — chậm và tốn RAM hơn IVFFlat nhiều

## CHAIN — Edge case giới hạn 2000 chiều

kiểu `vector` **lưu** được tới 16000 chiều => nhưng HNSW/IVFFlat chỉ **index** được tối đa **2000 chiều** => model `text-embedding-3-large` (3072 dims) dính ngay lỗi này => cách 1 — **`halfvec`** (fp16 thay fp32): index được tới ~4000 chiều, giảm nửa dung lượng, mất recall không đáng kể => cách 2 — **Matryoshka**: cắt vector ngắn lại (3072 → 1024) vì model dồn thông tin quan trọng về đầu vector => cách 3 — **DiskANN (`pgvectorscale`)**: support native tới 16000 chiều, không cần workaround

## CHAIN — DiskANN / pgvectorscale

DiskANN qua extension `pgvectorscale` của Timescale là "player thứ 3" đáng biết ở tầm staff => nó giữ phần lớn graph trên **SSD** thay vì RAM + **binary quantization** => nhờ vậy index cùng dataset có thể nhỏ hơn HNSW ~9x (vd 21MB vs 193MB trong benchmark công khai) => và support vector tới **16000 chiều native**

## CHAIN — Normalize & NULL embedding

normalize mọi vector về length 1 lúc insert → có thể dùng `<#>` (nhanh hơn cosine) => nhưng phải normalize **CẢ** vector query lúc search, nếu không kết quả sai lệch => còn hàng có `embedding IS NULL` sẽ không xuất hiện trong ANN result => NULL thường do pipeline embed lỗi vài dòng → nên có job kiểm `WHERE embedding IS NULL` rồi re-embed

## CHAIN — Scale & bottleneck RAM

< 10 triệu vectors: một node PostgreSQL + HNSW là *đủ và tốt nhất* về vận hành (query single-digit ms, 90% use case B2B RAG/search) => đừng over-engineer; bottleneck xuất hiện vì HNSW graph phải nằm gọn trong RAM để nhanh => 1536 dims × 4 bytes ≈ 6KB/vector => 100M vector ≈ **600GB chỉ riêng raw vectors**, chưa kể overhead graph => **RAM là ràng buộc số một** => hết RAM thì tính tới **quantization** và **DiskANN**

## CHAIN — Build time & scale out

HNSW build là `O(n log n)`, tốn hàng giờ ở quy mô lớn => tăng tốc build: nạp data xong mới build + tăng `maintenance_work_mem` (~working set nhưng ≤ 50–60% RAM) + tăng `max_parallel_maintenance_workers` => scale **vertical** (thêm RAM/CPU/SSD cho 1 instance) làm trước vì đơn giản và đủ xa => hết đường vertical thì scale **horizontal** => **read replicas** cho query load (search là read-heavy → replica scale rất tốt) => **partitioning** theo access pattern (`tenant_id`/`locale`/`product_line`), build index *per-partition* → planner **prune** partition không liên quan trước khi search => bảng non-partitioned giới hạn 32TB, partitioned thì gần như vô hạn => 1 node không chứa nổi thì **sharding** qua Citus / PgDog / pgvectorscale

## CHAIN — Quantization & two-stage retrieval

quantization là đòn bẩy cắt chi phí RAM/storage lớn nhất ở tầm staff => `halfvec` (fp16) cắt ~50% dung lượng, ảnh hưởng recall rất nhỏ => **binary quantization** cắt tới ~32x nhưng có mất recall (bù bằng re-rank exact) => "bù bằng re-rank" chính là pattern **two-stage retrieval** (recall → precision) => stage 1: dùng index nén (binary/HNSW) lấy top-N ứng viên *nhanh, thô* => stage 2: **re-rank** N ứng viên đó bằng exact distance + business signals (độ mới, popularity, quyền truy cập) => vì N nhỏ nên re-rank rẻ mà chất lượng cuối cao; làm re-rank ngay trong SQL để giữ tính atomic

## CHAIN — Cost & operational simplicity

managed Postgres có pgvector sẵn (Supabase / Neon / RDS / Aurora) từ vài chục USD/tháng => rẻ hơn nhiều so với thêm một vector DB chuyên dụng ($$$/tháng + team vận hành riêng) => giá trị lớn nhất là **KHÔNG thêm hệ thống mới** => không thêm hệ thống = bớt một failure domain, một on-call surface

## CHAIN — Latency & monitoring recall

đo latency phải theo **p50/p95/p99**, không đo trung bình => HNSW tail latency phình khi `ef_search` cao hoặc graph spill khỏi RAM => còn chất lượng ANN không có "đúng/sai" nhị phân nên phải **ĐO recall** định kỳ => đo recall bằng cách ép exact search (`SET LOCAL enable_indexscan = off`) làm ground truth => so overlap kết quả approximate với exact → **recall@k** => dựng dashboard theo dõi recall@k theo thời gian; recall tụt = tín hiệu cần re-tune / re-index

## CHAIN — Failure modes cần chuẩn bị

failure mode 1: embedding model đổi phiên bản → vector cũ "lệch không gian" với vector mới → kết quả rác => failure mode 2: VACUUM trên HNSW rất chậm → reindex trước rồi mới vacuum => failure mode 3: pipeline embed lỗi âm thầm để lại `NULL`/vector rác => failure mode 4: `SET` toàn cục rò qua connection pooler sang query người khác

## CHAIN — Khi nào NÊN / KHÔNG NÊN dùng pgvector

**NÊN** pgvector khi vector search là *một tính năng* trong app đã chạy Postgres => cần filter/join vector với business data, < 10–50M vectors, muốn ít hệ thống + ACID transaction => **KHÔNG NÊN** khi > 50M–hàng tỷ vectors cần globally distributed multi-region => hoặc cần hybrid search (vector + keyword BM25) built-in mạnh, cần managed scaling vượt provider, hoặc vector search **LÀ workload chính** => khi vector là cả workload thì hệ chuyên dụng (Pinecone/Weaviate/Qdrant/Milvus) tối ưu hơn => câu chốt: **pgvector thắng ở operational simplicity, không phải peak performance; chọn khi vector là feature, không phải cả sản phẩm**

## CHAIN — Tổ chức & quyết định chọn model

giải thích cho PM/sếp bằng **risk & cost**, không bằng jargon => framing: tận dụng Postgres sẵn có, đội đã biết vận hành, giảm rủi ro và chi phí => quyết định nặng ký nhất **không phải chọn index mà là chọn embedding model** => vì đổi model = re-embed toàn bộ corpus (tốn tiền API + downtime/blue-green) => nên chốt model sớm, **version hóa embedding** (cột `embedding_v2`), có kế hoạch backfill => giữ vector trong Postgres nghĩa là team backend hiện tại tự lo được, không cần tuyển/đào tạo riêng cho một vector DB stack

## CHAIN — RAG, ANN, Recall (thuật ngữ chốt)

vector search là bước **"retrieval"** trong RAG => **RAG** = Retrieval-Augmented Generation => phần retrieval thường dùng **ANN** = Approximate Nearest Neighbor: đánh đổi recall lấy tốc độ, ngược với exact => "đánh đổi recall" nói tới **recall** = % hàng xóm thật mà search tìm được => recall chính là thước đo chất lượng của ANN

---

## BẢNG — Ba toán tử distance

| Toán tử | Ý nghĩa | Dùng khi |
|---|---|---|
| `<->` | L2 (Euclidean) distance | mặc định, an toàn |
| `<=>` | Cosine distance | phổ biến nhất với text embeddings |
| `<#>` | Negative inner product | khi vector đã normalize, cần tốc độ |

## BẢNG — Quy tắc chọn distance metric

- Text embeddings (OpenAI/most models) → **cosine** (`<=>`).
- Đã tự normalize vector về length 1 → **inner product** (`<#>`) để nhanh hơn.
- Chỉ dùng **L2** (`<->`) khi độ lớn vector thực sự mang nghĩa (ít gặp với text).

## BẢNG — So sánh index

| Tiêu chí | IVFFlat | HNSW | DiskANN (pgvectorscale) |
|---|---|---|---|
| Query speed/recall | Tốt | **Rất tốt** | Rất tốt |
| Build time | **Nhanh** | Chậm | Trung bình |
| Bộ nhớ | **Ít** | Nhiều (graph in-RAM) | Ít (SSD + quantize) |
| Cần data để build? | **Có** (train k-means) | Không | Có |
| Query complexity | ~O((n/lists)·probes·d) | **~O(log n)** | ~O(log n) |
| Thêm data mới (dynamic) | Kém (lệch phân hoạch) | **Tốt** | Tốt |
| Dim > 2000 (native) | Không (cần halfvec) | Không (cần halfvec) | **Có (tới 16000)** |
| Khi nào chọn | build lại thường xuyên, RAM hạn chế, data tĩnh | **default 2026**, latency thấp + recall cao | dataset khổng lồ, tiết kiệm RAM/chi phí |

## BẢNG — Tham số tune (chốt số)

| Index | Tham số | Mặc định / gợi ý | Tác dụng |
|---|---|---|---|
| HNSW | `m` | 16 | ↑ → recall↑, index to, build chậm |
| HNSW | `ef_construction` | 64 | ↑ → recall↑, build chậm |
| HNSW | `ef_search` | 40 | ↑ → recall↑, latency↑ (tune theo SLA p95) |
| IVFFlat | `lists` | rows/1000 (≤1M), sqrt(rows) (>1M) | số "quận" k-means |
| IVFFlat | `probes` | khởi đầu sqrt(lists) | ↑ → recall↑, chậm↓ |

- IVFFlat query complexity ≈ `O(lists·d)` (so centroid) + `O((n/lists)·probes·d)` (quét trong list).

## BẢNG — Quantization (cắt chi phí)

| Kỹ thuật | Cắt dung lượng | Ảnh hưởng recall |
|---|---|---|
| `halfvec` (fp16) | ~50% | rất nhỏ |
| Binary quantization | tới ~32x | có (bù bằng re-rank exact) |
| Matryoshka (cắt chiều) | tùy tỉ lệ cắt | nhỏ nếu model hỗ trợ |

## BẢNG — Code cần thuộc lòng

**(a) Vòng đời tối thiểu:**
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

**(b) Filtered search 1 câu (siêu năng lực):**
```sql
SELECT id FROM items
WHERE tenant_id = 42 AND in_stock
ORDER BY embedding <=> :query_vec
LIMIT 10;
```

**(c) Exact k-NN cosine bằng NumPy (hiểu bản chất):**
```python
def cosine_knn(q, corpus, k=10):
    q = q / (q @ q) ** 0.5
    c = corpus / (corpus ** 2).sum(1, keepdims=True) ** 0.5
    dist = 1 - c @ q
    idx = dist.argsort()[:k]
    return idx, dist[idx]
```

## BẢNG — Khung trả lời system design (50M sản phẩm, filter, p95<100ms, multi-tenant)

1. **Clarify trước:** QPS? tần suất update catalog? recall target? ngân sách? (staff luôn hỏi trước khi vẽ).
2. **Embedding layer:** chọn model (~1024–1536 dims), sinh embedding bất đồng bộ qua queue, version hóa embedding.
3. **Storage/model:** một bảng `products` chứa metadata + `vector`, **partition theo tenant_id**, B-tree index trên (category, price, in_stock).
4. **Index:** HNSW `vector_cosine_ops`, `m=16..32`, tune `ef_search` theo SLA, bật `iterative_scan = relaxed_order`.
5. **Query:** một câu SQL: filter metadata (prune partition) → `ORDER BY <=>` → LIMIT; two-stage re-rank nếu cần precision cao.
6. **Scale/latency:** read replicas cho search; 50M × ~6KB ≈ 300GB → cân RAM, cân halfvec/binary quantization hoặc DiskANN; đo p95 thật.
7. **Ops:** monitor recall@10 (so exact định kỳ), p99 latency, tỉ lệ NULL embedding; kế hoạch reindex; blue-green khi đổi model.
8. **Khi nào bỏ pgvector:** hàng trăm triệu + đa vùng + hybrid search nặng → cân nhắc Milvus/Vespa. Biết giới hạn lựa chọn của mình = dấu hiệu staff.

## BẢNG — Mental models để nói trôi chảy

- **"Bản đồ ý nghĩa"** → giải thích embedding cho người mới trong 1 câu.
- **"Đường cao tốc nhiều tầng"** → HNSW navigate từ tầng cao xuống thấp.
- **"Chia quận, chỉ vào vài quận gần"** → IVFFlat (và điểm yếu ranh giới quận).
- **"Recall → precision"** → two-stage retrieval (thô rồi tinh).
- **"Vector là feature hay là product?"** → kim chỉ nam chọn pgvector vs Pinecone.

---

## TỪ KHÓA MỒI

- Từ vấn đề tới embedding → **"khớp từ khóa vs khớp nghĩa"**
- Vector, dimension, distance, k-NN → **"độ dài cố định"**
- Ai sinh vs ai lưu → **"pgvector không sinh embedding"**
- Tính tay L2 tới `<->` → **"đường chim bay"**
- Hello world SQL → **"CREATE EXTENSION vector"**
- Ba distance metric → **"đo góc bỏ độ dài"**
- Data modeling schema → **"vector cạnh business data"**
- HNSW build & tune → **"ops class khớp toán tử"**
- Pipeline Python → **"register_vector"**
- Filtered search → **"một round-trip"**
- Overfiltering → **"ef_search hàng xóm trước, WHERE sau"**
- Hai bẫy index → **"Postgres âm thầm bỏ qua index"**
- Curse of dimensionality → **"mọi điểm cách đều nhau"**
- IVFFlat → **"chia quận bằng k-means"**
- HNSW mechanism → **"Hierarchical Navigable Small World"**
- Edge case 2000 chiều → **"lưu 16000, index 2000"**
- DiskANN → **"graph trên SSD"**
- Normalize & NULL → **"normalize cả query"**
- Scale & RAM → **"6KB/vector"**
- Build time & scale out → **"read replicas + partitioning"**
- Quantization & two-stage → **"recall → precision"**
- Cost & simplicity → **"không thêm hệ thống mới"**
- Latency & recall → **"p50/p95/p99"**
- Failure modes → **"lệch không gian khi đổi model"**
- Khi nào dùng/không → **"feature không phải cả sản phẩm"**
- Tổ chức & chọn model → **"quyết định nặng hơn chọn index"**
- RAG, ANN, Recall → **"retrieval"**

---

## ĐÃ PHỦ

**Khái niệm nền tảng:** vector search / similarity search, bài toán "khớp từ khóa vs khớp nghĩa", bản đồ ý nghĩa, embedding, embedding model, vector, dimension, distance = similarity, Nearest Neighbor / k-NN, pgvector, PostgreSQL/RDBMS, extension, kiểu `vector`.

**Kiến trúc 2 phần:** embedding model sinh vector vs pgvector chỉ lưu & tìm.

**Toán & metric:** công thức L2 = `sqrt(Σ(xi-yi)²)`, ví dụ tính tay (áo phao/khăn len/kem chống nắng = 1/1/9.9), cosine similarity `(a·b)/(|a||b|)` ∈ [-1,1], cosine distance = 1-sim ∈ [0,2], inner product `Σaibi` + negative, 3 toán tử `<->`/`<=>`/`<#>`, quy tắc chọn metric.

**Con số chốt:** text-embedding-3-small = 1536 dims; default `m`=16, `ef_construction`=64, `ef_search`=40; IVFFlat `lists`/`probes` gợi ý; bảng nhỏ <10K–50K exact; 100% recall khi exact; giới hạn index 2000 dims, `vector` lưu 16000 dims; text-embedding-3-large=3072; halfvec ~4000 dims; DiskANN nhỏ hơn ~9x (21MB vs 193MB); 6KB/vector, 100M≈600GB, 300GB cho 50M; maintenance_work_mem ≤50–60% RAM; non-partitioned 32TB limit; binary quant ~32x; halfvec ~50%; relaxed_order 95–99%.

**Big-O:** brute-force O(n·d); HNSW query ~O(log n), build ~O(n·log n·m·d); IVFFlat query ~O(lists·d)+O((n/lists)·probes·d).

**Code:** CREATE EXTENSION, cột vector(3), INSERT `'[...]'`, ORDER BY `<->` LIMIT, HNSW CREATE INDEX + ops class, SET LOCAL ef_search, pipeline sentence-transformers + psycopg + register_vector (all-MiniLM-L6-v2 = 384), filtered search, iterative_scan + max_scan_tuples, halfvec index cast, cosine_knn, MiniIVF, recall measurement với enable_indexscan=off.

**Cơ chế index:** IVFFlat (k-means/lists/probes, cần train, tĩnh, ranh giới quận, REINDEX); HNSW (nhiều tầng, cao tốc/hẻm, phân phối mũ, m hàng xóm); DiskANN/pgvectorscale (SSD + binary quant, 16000 dims); curse of dimensionality + vì sao kd-tree fail.

**Trade-off & đánh đổi:** exact vs ANN (accuracy↔speed), ef_search (recall↔latency↔RAM), probes (recall↔speed), quantization (dung lượng↔recall), vertical vs horizontal scale, operational simplicity vs peak performance, tam giác build-time / RAM / recall (qua bảng so sánh).

**Edge case:** overfiltering + iterative scan; sai ops class → bỏ qua index; ANN approximate/recall<100%; giới hạn 2000 dims + 3 cách xử lý (halfvec/Matryoshka/DiskANN); normalize cả query khi dùng `<#>`; NULL embedding không xuất hiện; `SET` global rò qua PgBouncer; VACUUM HNSW chậm → reindex trước.

**Staff/scale:** ngưỡng <10M / <50M / >50M; RAM là bottleneck số 1; quantization + two-stage retrieval (re-rank exact + business signals); read replicas, partitioning theo access pattern + prune, sharding (Citus/PgDog/pgvectorscale); build-time tuning; cost managed Postgres vs vector DB; monitoring p50/p95/p99 + recall@k dashboard; 4 failure modes.

**Quyết định & tổ chức:** khi NÊN/KHÔNG NÊN pgvector (Pinecone/Weaviate/Qdrant/Milvus); chọn model nặng hơn chọn index + version hóa embedding + backfill + blue-green; framing risk & cost cho stakeholder; team topology; khung system design 8 bước; mental models; RAG/ANN/recall.

**Version:** pgvector v0.8.x (0.8.4), HNSW default 2026, iterative scans (0.8.0), halfvec/sparsevec, binary quantization, pgvectorscale.

**Chưa nêu trong bài (không bịa):** giá trị `distance` cụ thể ở output pipeline Python (bài chỉ ghi ~0.31xx / ~0.42xx dạng xấp xỉ); benchmark chi tiết QPS của từng index; công thức recall@k chính xác dạng số.
