# Indexing cho Vector Search: Từ Binary Search tới IVFFlat & HNSW — Giáo trình Basic → Staff

> **Nguồn gốc:** Reading *"Creating Indexes for Vector Search Optimization"* (tác giả Lavanya T S) — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy: định nghĩa database index, index tiết kiệm thời gian thế nào (linear vs binary search), và hai index của pgvector (IVFFlat, HNSW) kèm tham số. Tôi giảng bám sát rồi đào sâu. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là mảnh *"làm cho search NHANH"*. Bộ ba trước: embed (tạo vector) → store & search (pgvector) → keyword (FTS). Bài này zoom vào một câu hỏi: *khi có hàng triệu vector, làm sao tìm nhanh mà không quét hết?* Có một phần trùng với giáo trình pgvector đầu tiên (HNSW/IVFFlat) — ở đây tôi tiếp cận từ **lý thuyết indexing** và bổ sung mắt xích mà reading bỏ qua, thay vì lặp lại.
>
> **⚠️ Đính chính nhỏ trong reading (để bạn đi phỏng vấn không nói sai):**
> 1. **HNSW = Hierarchical Navigable Small *World*** (thế giới nhỏ), reading viết "Small Words" là sai chính tả thuật ngữ.
> 2. Reading nói HNSW "links are the Euclidean distances" — thực ra HNSW dùng **bất kỳ distance metric nào bạn cấu hình** (chính ví dụ của reading dùng `vector_cosine_ops` → cosine, không phải Euclidean). Cạnh graph nối theo *độ gần* theo metric đã chọn.
> 3. Hình HNSW trong reading đánh nhãn tầng lỗi ("Layer 3, 2, 3, 0") — đúng phải là **Layer 3 → 2 → 1 → 0** (trên xuống dưới).

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** *Index* là gì và vì sao nó biến việc tìm kiếm từ "quét từng cái" (chậm khủng khiếp khi dữ liệu lớn) thành "nhảy thẳng tới nơi cần" (nhanh). Ta đi từ index kinh điển (binary search, B-tree cho dữ liệu 1 chiều) → **vì sao chúng gãy với vector nhiều chiều** → hai index chuyên cho vector: **IVFFlat** và **HNSW**, cùng cách tune tham số.

**Vấn đề nó giải quyết:** Tìm 1 vector giống nhất trong 100 triệu vector bằng cách so từng cái (brute-force) là `O(n)` — bất khả thi real-time. Index approximate nearest neighbor (ANN) đưa việc đó về gần `O(log n)`, đổi một chút độ chính xác (recall) lấy tốc độ gấp hàng nghìn lần.

**Học xong bạn sẽ làm được:**

- Định nghĩa database index và tính được chi phí linear search `(n+1)/2` vs binary search `O(log n)`.
- Giải thích **vì sao B-tree/binary search không dùng được cho vector nhiều chiều** (mắt xích cốt lõi).
- Mô tả cơ chế IVFFlat (centroid/list/probe) và HNSW (graph nhiều tầng), với Big-O.
- Viết câu lệnh tạo index và tune `lists`/`probes`, `m`/`ef_construction`/`ef_search`.
- Chọn index đúng theo quy mô, phân tích trade-off recall–speed–memory, và trả lời câu hỏi system design.

**Mạch basic → staff:**

- 🟢 **Basic:** Index là gì (analogy sách) → linear search `(n+1)/2` → sorted + binary search → ví dụ tìm "Jack" chạy tay → B-tree.
- 🟡 **Intermediate:** **Vì sao index kinh điển gãy với vector** → ANN → IVFFlat (centroid/list/probe) + code + exact vs approximate + lỗi thường gặp.
- 🔴 **Advanced:** HNSW internals + ví dụ "tìm 12", Big-O IVFFlat vs HNSW vs brute-force, curse of dimensionality, DiskANN, tune sâu, tự implement, edge cases.
- 🟣 **Staff:** Chọn index ở quy mô lớn, build cost, recall monitoring, memory, khi nào exact tốt hơn, filtered search, câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Index là gì (định nghĩa + analogy)

- **Database index:** một *cấu trúc dữ liệu* tăng tốc tìm kiếm và truy xuất. Nó chứa một bản sao (đã tổ chức) của một phần dữ liệu từ bảng, giúp định vị nhanh dòng cần tìm.
- **Analogy (bài gốc dùng):** giống *mục lục tra cứu (index) cuối một cuốn sách*. Muốn tìm chủ đề "database", bạn không đọc cả cuốn — bạn tra mục lục, thấy "database → trang 12, 45", nhảy thẳng tới đó. Database index là "mục lục" của bảng.

Ý tưởng cốt lõi: **trả một chút chi phí lưu trữ + cập nhật để đổi lấy tốc độ tìm kiếm khổng lồ.**

### 1.2. Vì sao index tiết kiệm thời gian — linear search tốn thế nào

**Linear search (quét tuyến tính):** so từng dòng một cho tới khi tìm thấy.

- Trung bình phải đọc `(n + 1) / 2` dòng (may thì gặp sớm, xui thì gặp cuối).
- Tệ nhất: `n` dòng (nằm ở cuối, hoặc không tồn tại).
- Với `n = 1.000.000`, trung bình đọc **~500.000 dòng** cho *một* lần tìm. Nhân với hàng nghìn query/giây → sập.

Đây là lý do "không index" là bất khả thi khi dữ liệu lớn — đúng như summary bài gốc nhấn mạnh: *hiệu quả của index chỉ thực sự thấy rõ trên tập dữ liệu lớn.*

### 1.3. Ví dụ chạy tay — tìm "Jack" (bám ví dụ bài gốc)

**Không index (unsorted), tìm "Jack" ở vị trí 20/20:**
> Emily → James → Olivia → ... phải đọc **cả 20 dòng** mới tới "Jack" ở cuối. Với danh sách chưa sắp xếp, không có cách nào nhanh hơn.

**Có sorted index (sắp xếp theo bảng chữ cái) + binary search:**

Danh sách đã sort: Alexander, Amelia, Ava, Benjamin, Charlotte, Daniel, Emily, Emma, Grace, **Isabella**, Jack, James, Lucas, **Michael**, Mia... William (20 tên).

Binary search chia đôi liên tục:

1. **Bước 1:** chia đôi 20 tên. Xem tên đầu nửa sau = **Isabella** (I). "Jack" (J) đứng *sau* I → chắc chắn nằm ở **nửa sau** (10 tên: Isabella...William). Loại nửa đầu. *Vừa loại 10 tên trong 1 bước.*
2. **Bước 2:** chia đôi 10 tên còn lại. Điểm giữa = **Michael** (M). "Jack" (J) đứng *trước* M → nằm ở nửa đầu (Isabella, Jack, James, Lucas, Mia). Loại nửa sau.
3. **Bước 3:** chia đôi tiếp (Isabella | Jack, James...). "Jack" đứng sau Isabella → còn (Jack, James, Lucas, Mia).
4. **Bước 4:** điểm giữa → chạm **Jack**. ✅

**4 bước** thay vì 20 lần đọc. Với `n` phần tử, binary search chỉ cần **~log₂(n)** bước: 20 tên → ~5 bước; **1 triệu** tên → chỉ **~20 bước** (2²⁰ ≈ 1.048.576). Đây là sức mạnh của `O(log n)`.

### 1.4. Điều kiện tiên quyết của binary search — phải SẮP XẾP ĐƯỢC

Binary search chỉ chạy được vì tên **sắp thứ tự trên một trục** (bảng chữ cái). Ở mỗi bước ta trả lời được: *"mục tiêu nằm trước hay sau điểm giữa?"* — nhờ có **thứ tự tuyến tính (total order)**.

> **[Ghi nhớ — mắt xích quan trọng nhất cả bài]** Điều kiện "sắp xếp được trên một trục" chính là chỗ **sẽ vỡ** khi ta chuyển sang vector nhiều chiều. Giữ ý này trong đầu; Phần 2 sẽ giải thích vì sao và ta phải làm khác.

### 1.5. Index kinh điển trong DB = B-tree

Trong RDBMS, index mặc định là **B-tree** (một cây cân bằng), cho phép tìm/`range scan` ở `O(log n)`. Nó là "binary search có cấu trúc cây" trên dữ liệu *sắp thứ tự được*: số, chuỗi, ngày tháng. `CREATE INDEX ON users(email)` → tra email cực nhanh.

```sql
-- Index kinh điển: B-tree trên cột sắp thứ tự được
CREATE INDEX ON users (email);           -- tìm theo email: O(log n)
CREATE INDEX ON orders (created_at);     -- range: "đơn trong 7 ngày qua": nhanh
```

B-tree tuyệt vời cho dữ liệu **1 chiều, sắp thứ tự được**. Vector thì không như vậy — và đó là lý do pgvector cần index khác.

### ✅ Self-check Phần 1

1. Linear search trên 1 triệu dòng trung bình đọc bao nhiêu dòng? Binary search cần bao nhiêu bước?
2. Vì sao binary search *bắt buộc* dữ liệu phải sắp xếp?
3. B-tree hợp với kiểu dữ liệu nào? Cho một ví dụ cột nên đánh B-tree.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. [MẮT XÍCH CỐT LÕI] Vì sao B-tree/binary search GÃY với vector

Bài gốc nhảy thẳng từ binary search sang IVFFlat/HNSW mà không giải thích *vì sao cần index mới*. Đây là chỗ phân biệt người hiểu và người học vẹt.

**Vấn đề 1 — vector không có "thứ tự tuyến tính".** Bạn sort tên theo bảng chữ cái được. Nhưng vector `[0.2, -0.7, 0.4, ...]` (384 chiều) sort theo *chiều nào*? Sort theo chiều 1 thì hai vector "gần nhau về nghĩa" có thể cách xa nhau ở chiều 1. **Không tồn tại một trục duy nhất** để sắp mọi vector sao cho "gần trên trục = gần thật sự". Binary search mất điều kiện tiên quyết (mục 1.4).

**Vấn đề 2 — câu hỏi khác hẳn.** B-tree trả lời *"có bằng đúng giá trị X không / trong khoảng [a,b] không"*. Vector search hỏi *"vector nào GẦN NHẤT về khoảng cách"* — một bài toán **nearest neighbor**, không phải khớp/khoảng.

**Vấn đề 3 — curse of dimensionality.** Người ta từng thử cây phân hoạch không gian (kd-tree) cho NN. Nhưng khi số chiều lớn (hàng trăm–nghìn), mọi điểm gần như *cách đều nhau*, cây phải duyệt gần hết → không còn nhanh hơn brute-force. (Đào sâu ở Phần 3.)

**Kết luận:** với vector nhiều chiều, ta **từ bỏ tìm chính xác tuyệt đối** và chuyển sang **Approximate Nearest Neighbor (ANN)** — chấp nhận thỉnh thoảng bỏ sót hàng xóm thật để đổi lấy tốc độ. IVFFlat và HNSW là hai cách làm ANN.

### 2.2. IVFFlat — "chia cụm rồi chỉ tìm trong vài cụm"

**IVFFlat = Inverted File with Flat compression.** Cơ chế (bám bài gốc + hình 3 centroid):

1. **Build:** chạy thuật toán clustering (k-means) chia toàn bộ vector thành `n` **list**, mỗi list là một cụm quanh một **centroid** (tâm cụm).
2. **Search:** so query với các *centroid* → chọn centroid gần nhất → chỉ tìm brute-force *bên trong* cụm của centroid đó (thay vì cả triệu vector).

Trong hình bài gốc: Search Vector (màu tím) được so với Centroid 1, 2, 3; nó gần **Centroid 3** nhất → chỉ lục trong cụm xanh lá.

**Hai tham số phải hiểu:**

- **`lists`** (số cụm): bài gốc/pgvector gợi ý `rows/1000` cho ≤ 1 triệu rows. 1 triệu vector → 1000 list, mỗi list ~1000 phần tử. (Trên 1 triệu: `sqrt(rows)`.)
- **`probes`** (số cụm dò khi search): mặc định 1. Cao hơn → xét thêm cụm lân cận → **recall tốt hơn** nhưng **chậm hơn**. Gợi ý khởi đầu `sqrt(lists)`.

**Code (bám bài gốc):**

```sql
-- Tạo IVFFlat index. LƯU Ý: phải tạo SAU khi bảng đã có dữ liệu (cần data để k-means).
CREATE INDEX ON sentence USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Khi search, đặt số cụm dò. probes cao = recall cao, tốc độ giảm.
SET ivfflat.probes = 32;

SELECT * FROM sentence ORDER BY embedding <=> '[...]' LIMIT 10;
```

**Điểm yếu IVFFlat:** hàng xóm thật nằm **sát ranh giới cụm** dễ bị bỏ sót (nó ở cụm bên cạnh mà `probes` không dò tới). Và IVFFlat *tĩnh*: thêm nhiều dữ liệu mới sau khi build → phân hoạch lệch, recall tụt, phải `REINDEX`.

### 2.3. Exact vs Approximate — điều junior hay hiểu nhầm

- **Không index → exact search:** pgvector quét hết, tính distance từng cái → **recall 100%** (luôn đúng), nhưng `O(n)` (chậm khi lớn).
- **Có ANN index → approximate:** nhanh (~log n hoặc theo cụm), nhưng **recall < 100%** — có thể thiếu vài hàng xóm thật. Đây là *by design*, không phải bug.

> Bài gốc nói đúng ở summary: mỗi phương pháp có strength & trade-off; **chọn index và tune tham số là cân bằng accuracy ↔ speed ↔ resource**. Câu này là linh hồn của cả bài.

### 2.4. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — Đánh index quá sớm trên dữ liệu nhỏ.** Với vài nghìn–vài chục nghìn vector, brute-force exact *đã đủ nhanh* và cho recall 100%. Đánh ANN index lúc này chỉ tổ giảm độ chính xác mà chẳng lợi tốc độ đáng kể. Index chỉ đáng khi dữ liệu đủ lớn (đúng như summary bài gốc: "chỉ thấy hiệu quả trên tập lớn").

**Lỗi 2 — Tạo IVFFlat khi bảng còn rỗng.** IVFFlat cần dữ liệu để k-means tìm centroid. Tạo lúc bảng trống → phân cụm vô nghĩa, recall tệ. (Khác HNSW — HNSW tạo được cả khi bảng rỗng.)

**Lỗi 3 — Sai ops class so với toán tử query.** Index `vector_cosine_ops` nhưng query bằng `<->` (L2) → Postgres **bỏ qua index**, rơi về seq scan. Ops class phải khớp toán tử: `vector_cosine_ops`↔`<=>`, `vector_l2_ops`↔`<->`, `vector_ip_ops`↔`<#>`. Kiểm bằng `EXPLAIN ANALYZE`.

### ✅ Self-check Phần 2

1. Nêu 2 lý do binary search/B-tree không dùng được để tìm nearest neighbor của vector.
2. IVFFlat: `lists` và `probes` mỗi cái điều khiển gì? Tăng `probes` được/mất gì?
3. Vì sao không nên tạo IVFFlat index khi bảng còn rỗng?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. HNSW — graph nhiều tầng "đường cao tốc"

**HNSW = Hierarchical Navigable Small World.** Cơ chế:

- Xây một **graph nhiều tầng**: node = vector, cạnh nối một node với các hàng xóm gần nó (theo distance metric đã chọn — cosine/L2/ip, **không chỉ Euclidean** như reading viết nhầm).
- **Tầng trên cùng:** ít node, cạnh "đường dài" — nhảy xa nhanh về đúng vùng.
- **Tầng dưới cùng:** chứa mọi node, cạnh "đường ngắn" — tinh chỉnh tới hàng xóm chính xác.
- **Search:** bắt đầu ở tầng trên, đi tham lam (greedy) tới node gần query nhất, rồi tụt xuống tầng dưới, lặp lại tới tầng đáy.

**Ví dụ "tìm 12" của reading** (diễn giải + sửa nhãn tầng): danh sách đáy 2,4,8,12,14,16,19.
- **Layer 3 (đỉnh):** chỉ có 4 → từ entry nhảy tới 4.
- **Layer 2:** có 4, 16 → 12 nằm giữa, đi về phía phù hợp.
- **Layer 1:** có 4, 12, 16 → thấy 12.
- **Layer 0 (đáy):** dừng ở **12**. ✅

Analogy: giống bản đồ giao thông — bay tuyến quốc tế (tầng cao) về đúng thành phố, rồi taxi (tầng thấp) tới đúng địa chỉ. Nhờ vậy search ~**O(log n)**.

**Code (bám bài gốc):**

```sql
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

- **`m`** = số cạnh tối đa mỗi node ở mỗi tầng. Cao → recall tốt hơn, graph to hơn, build/RAM tốn hơn.
- **`ef_construction`** = số ứng viên hàng xóm xét *khi xây* graph (reading: 64 ứng viên mỗi lần chèn). Cao → graph chất lượng hơn, recall tốt hơn, **build chậm hơn**.
- **`ef_search`** (tham số *query-time*, reading không nhắc): số ứng viên xét *khi tìm*. Mặc định 40; cao → recall tốt hơn, chậm hơn. Đây là núm xoay recall–speed quan trọng nhất lúc chạy:

```sql
SET hnsw.ef_search = 100;   -- tăng recall khi cần
```

### 3.2. Big-O: brute-force vs IVFFlat vs HNSW

| Phương pháp | Query complexity | Build | Memory | Cần data để build? | Data động |
|---|---|---|---|---|---|
| Brute-force (no index) | `O(n·d)` | 0 | thấp | — | tốt (recall 100%) |
| **IVFFlat** | ~`O(lists·d + (n/lists)·probes·d)` | **nhanh** | **thấp** | **có** (k-means) | kém (lệch cụm) |
| **HNSW** | ~**`O(log n · d)`** | chậm | cao (graph in-RAM) | không | **tốt** |

(`n` = số vector, `d` = số chiều.) **Chốt:** HNSW nhanh & recall cao nhất nhưng tốn RAM + build lâu; IVFFlat nhẹ & build nhanh nhưng recall kém hơn và ngại data động. Đây là bảng phỏng vấn hay hỏi.

### 3.3. Curse of dimensionality — vì sao NN "khó" ở chiều cao

Ở 2D/3D, chia không gian thành ô (kd-tree) rất hiệu quả. Nhưng khi `d` lên hàng trăm–nghìn:

- Thể tích không gian tăng theo hàm mũ → dữ liệu trở nên *thưa*.
- Khoảng cách giữa điểm gần nhất và xa nhất *co lại gần nhau* → "gần nhất" mất ý nghĩa phân biệt.
- Cây phân hoạch phải duyệt gần hết nhánh → sập về `O(n)`.

Đó là lý do ta cần **approximate** + cấu trúc kiểu graph (HNSW) hoặc cụm (IVFFlat) thay vì cây chính xác. Nêu được "curse of dimensionality" khi phỏng vấn là điểm cộng lớn.

### 3.4. [MỞ RỘNG] DiskANN — "người thứ ba" ngoài bài gốc

Bài gốc chỉ nêu IVFFlat & HNSW. Ở tầm 2026, có thêm **DiskANN** (qua extension `pgvectorscale` của Timescale): giữ phần lớn graph trên **SSD** thay vì RAM + binary quantization → index nhỏ hơn HNSW nhiều lần, hỗ trợ vector tới 16000 chiều native (HNSW/IVFFlat của pgvector giới hạn **2000 chiều** khi index — một cú vấp kinh điển với model 3072 chiều như OpenAI 3-large; workaround: `halfvec`). Biết DiskANN + giới hạn 2000 chiều là dấu hiệu bạn cập nhật.

### 3.5. Tự implement để hiểu tận gốc

**(a) Binary search (nền tảng bài gốc):**
```python
def binary_search(sorted_names, target):
    lo, hi = 0, len(sorted_names) - 1
    steps = 0
    while lo <= hi:
        steps += 1
        mid = (lo + hi) // 2
        if sorted_names[mid] == target:
            return mid, steps                 # tìm thấy + số bước
        elif sorted_names[mid] < target:
            lo = mid + 1                       # loại nửa trái
        else:
            hi = mid - 1                       # loại nửa phải
    return -1, steps

names = sorted(["Emily","James","Olivia","Jack","Isabella","Michael","Mia","William"])
print(binary_search(names, "Jack"))            # ~3 bước thay vì quét tuyến tính
```

**(b) IVF mini — thấy trade-off `probes`:**
```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def build(self, corpus, n_lists=16):
        self.corpus = corpus
        km = KMeans(n_clusters=n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_     # tâm mỗi cụm
        self.assign = km.labels_                 # vector -> cụm nào
        return self

    def search(self, q, k=5, probes=2):
        # 1) chọn `probes` cụm gần query (thay vì quét cả corpus)
        near = np.argsort(np.linalg.norm(self.centroids - q, axis=1))[:probes]
        # 2) brute-force chỉ trong các cụm đã chọn
        cand = np.where(np.isin(self.assign, near))[0]
        d = np.linalg.norm(self.corpus[cand] - q, axis=1)
        top = np.argsort(d)[:k]
        return cand[top], d[top]
# Chạy với probes=1 vs probes=8 để tự thấy: probes nhỏ nhanh nhưng đôi khi trượt.
```

### 3.6. Edge cases

- **Giới hạn 2000 chiều** cho HNSW/IVFFlat index → dùng `halfvec` (tới ~4000) hoặc DiskANN cho model chiều cao.
- **IVFFlat sau khi nạp thêm nhiều data** → centroid lệch, recall tụt → `REINDEX`.
- **VACUUM trên HNSW rất chậm** → reindex trước rồi vacuum.
- **Recall âm thầm tụt** khi tham số quá thấp → phải *đo* recall (Phần 4), không đoán.
- **Sai ops class** → index bị bỏ, seq scan âm thầm.

### ✅ Self-check Phần 3

1. HNSW search ~`O(?)`. Vì sao "graph nhiều tầng" cho tốc độ đó (analogy)?
2. `ef_construction` (build-time) và `ef_search` (query-time) khác nhau chỗ nào?
3. "Curse of dimensionality" khiến kd-tree/exact tree hỏng thế nào ở chiều cao?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Chọn index theo quy mô — quyết định thật ở tầm staff

**Câu hỏi không phải "IVFFlat hay HNSW" trừu tượng, mà là "ở quy mô & ràng buộc CỦA TÔI thì cái nào".**

- **Dữ liệu nhỏ (< ~10K–50K vector):** **đừng đánh index.** Brute-force exact đủ nhanh, recall 100%, không phải lo tune. Đánh index sớm chỉ tự chuốc recall thấp. (Đúng như summary bài gốc: hiệu quả index chỉ thấy trên tập lớn.)
- **Trung bình (tới ~vài triệu):** **HNSW** là mặc định — recall cao, latency thấp, chịu data động tốt. Chấp nhận build lâu + RAM cao.
- **RAM là ràng buộc / build lại thường xuyên / dữ liệu khá tĩnh:** **IVFFlat** (nhẹ, build nhanh) hoặc **DiskANN** (SSD-based) khi dataset khổng lồ.
- **Chiều > 2000:** halfvec + HNSW, hoặc DiskANN.

### 4.2. Build cost, memory, và bottleneck vận hành

- **Build time:** HNSW build `O(n log n)`, hàng giờ ở quy mô lớn. Tăng tốc: nạp data xong mới build; tăng `maintenance_work_mem` (≤ 50–60% RAM); tăng `max_parallel_maintenance_workers`; build **`CREATE INDEX CONCURRENTLY`** để không khóa write.
- **Memory:** HNSW graph phải nằm gọn trong RAM để nhanh. 100M vector × 1536 chiều × 4 byte ≈ 600GB *chỉ riêng raw vectors* + overhead graph → RAM là ràng buộc số một → cân **quantization** (halfvec/binary) hoặc DiskANN.
- **Data động:** IVFFlat lệch cụm khi ghi nhiều → lịch reindex; HNSW chịu insert tốt hơn nhưng VACUUM chậm.

### 4.3. Monitoring recall — thứ phân biệt staff

ANN không có "đúng/sai" nhị phân. Phải **đo recall** định kỳ bằng cách so kết quả có-index với exact (ground truth):

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact search làm chuẩn vàng
SELECT id FROM items ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so overlap với kết quả có index -> recall@10; dashboard theo thời gian
```

Recall tụt = tín hiệu tăng `ef_search`/`probes`/`m` hoặc reindex. **Không đo recall = bay mù**: chất lượng search có thể xuống dốc mà không có lỗi nào báo.

### 4.4. Filtered search & overfiltering [MỞ RỘNG]

Khi kết hợp ANN với `WHERE` (vd `category='rare'`), index lấy `ef_search` hàng xóm *trước*, lọc *sau* → nếu filter hiếm, trả về ít hơn `LIMIT`. Đây là **overfiltering**. Khắc phục bằng **iterative index scan** (pgvector 0.8+):

```sql
SET hnsw.iterative_scan = relaxed_order;
SET hnsw.max_scan_tuples = 20000;
```

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Index cho search giống mục lục cuốn sách: trả một chút chi phí (bộ nhớ, thời gian xây dựng) để đổi lấy tìm kiếm nhanh gấp hàng nghìn lần. Với vector, ta *đánh đổi thêm một chút độ chính xác lấy tốc độ* — điều chỉnh được bằng vài núm vặn tùy nhu cầu. Trên dữ liệu nhỏ thì chưa cần; nó chỉ đáng khi lớn." → framing bằng **chi phí, tốc độ, và đánh đổi độ chính xác**.
- **Roadmap:** chọn/tune index không phải one-off — cần đo recall & latency liên tục, reindex khi data đổi. Nên coi đây là hạ tầng có vòng đời, không phải "set-and-forget".
- **Team:** vì mọi index này nằm trong Postgres, backend team hiện tại tự vận hành được, khỏi thêm hệ thống search riêng — nối lại luận điểm "một Postgres cho tất cả" của cả series.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Thiết kế vector search cho 100 triệu embedding, p95 < 100ms, có filter theo tenant/category, dữ liệu cập nhật hàng ngày, ngân sách RAM hạn chế."**

**Khung trả lời staff (nói to trade-off):**

1. **Clarify:** QPS? recall target? tần suất update? RAM/ngân sách cụ thể? chiều vector?
2. **Chọn index theo ràng buộc:** RAM hạn chế + 100M → HNSW thuần có thể không vừa RAM → cân **DiskANN (pgvectorscale)** hoặc **HNSW + quantization** (halfvec/binary) + two-stage re-rank (thô bằng index nén → tinh bằng exact trên top-N).
3. **Filter:** partition theo tenant/category (planner prune sớm) + bật **iterative_scan** chống overfiltering.
4. **Build/update:** nạp rồi build; `CREATE INDEX CONCURRENTLY`; lịch reindex cho phần data động; HNSW chịu insert hàng ngày tốt hơn IVFFlat.
5. **Latency:** tune `ef_search` theo SLA p95; read replicas cho search traffic.
6. **Monitoring:** recall@10 (so exact định kỳ), p99 latency, RAM/index size, tỉ lệ overfiltering.
7. **Khi nào không cần ANN:** nếu một tenant chỉ có vài nghìn vector, exact per-tenant còn nhanh & chính xác hơn — **không phải mọi thứ đều cần index.** Nêu được điều này = tư duy staff.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **Index** — cấu trúc dữ liệu tăng tốc tìm kiếm; như mục lục cuối sách.
- **Linear search** — quét từng dòng; trung bình `(n+1)/2`, tệ nhất `n`.
- **Binary search** — chia đôi liên tục trên dữ liệu đã sort; `O(log n)`.
- **B-tree** — index kinh điển của RDBMS cho dữ liệu 1 chiều sắp thứ tự được.
- **Nearest Neighbor (NN) / k-NN** — tìm k vector gần nhất theo distance.
- **ANN (Approximate NN)** — đổi recall lấy tốc độ; cần khi vector nhiều chiều.
- **Recall** — % hàng xóm thật mà search tìm được; thước đo chất lượng ANN.
- **Curse of dimensionality** — ở chiều cao, khoảng cách co lại, cây/exact index hỏng.
- **IVFFlat** — Inverted File Flat; chia `lists` cụm bằng k-means; `probes` = số cụm dò.
- **Centroid** — tâm cụm; query so với centroid để chọn cụm cần tìm.
- **HNSW** — Hierarchical Navigable Small **World**; graph nhiều tầng; ~`O(log n)`.
- **`m` / `ef_construction` / `ef_search`** — cạnh mỗi tầng / ứng viên lúc build / ứng viên lúc query.
- **`lists` / `probes`** — tham số IVFFlat.
- **DiskANN / pgvectorscale** — index SSD-based, tiết kiệm RAM, chiều tới 16000.
- **ops class** — `vector_cosine_ops`↔`<=>`, `vector_l2_ops`↔`<->`, `vector_ip_ops`↔`<#>`.
- **Iterative scan** — chống overfiltering khi ANN + WHERE (pgvector 0.8+).

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Index đổi chút chi phí lưu/build lấy tốc độ tìm kiếm khổng lồ; chỉ đáng khi dữ liệu lớn.
2. Linear `O(n)` → binary/B-tree `O(log n)`, nhưng **binary search cần dữ liệu sắp thứ tự được**.
3. **Vector nhiều chiều không sắp thứ tự trên một trục** → B-tree/binary search vô dụng → cần ANN.
4. ANN đổi **recall lấy tốc độ**; exact (no index) = recall 100% nhưng `O(n)`.
5. **IVFFlat:** chia cụm (centroid/list), dò `probes` cụm; build nhanh, nhẹ, cần data, ngại data động.
6. **HNSW:** graph nhiều tầng, ~`O(log n)`; recall cao, chịu data động; tốn RAM + build lâu.
7. `probes`/`ef_search` là núm xoay recall–speed lúc query; `lists`/`m`/`ef_construction` lúc build.
8. Dữ liệu nhỏ → **đừng index** (exact đủ nhanh, chính xác hơn).
9. Ops class phải khớp toán tử, nếu không index bị bỏ (seq scan âm thầm).
10. Ở quy mô lớn: RAM là bottleneck → quantization/DiskANN; và phải **đo recall** vì ANN xuống cấp không báo lỗi.

### 5.3. Ideas / mental models

- **"Mục lục cuối sách":** index là gì trong 1 câu.
- **"Sắp được thì binary search được":** vì sao vector cần index khác.
- **"Chia quận, chỉ vào vài quận":** IVFFlat (và điểm yếu ranh giới cụm).
- **"Đường cao tốc nhiều tầng":** HNSW navigate từ tầng cao xuống thấp.
- **"ANN xuống cấp trong im lặng":** vì sao phải monitor recall.

### 5.4. Code cần thuộc lòng

**(a) Hai câu tạo index (interviewer hay bắt viết):**
```sql
CREATE INDEX ON items USING hnsw    (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
SET hnsw.ef_search = 100;      -- query-time recall knob
SET ivfflat.probes  = 32;
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
-- exact (ground truth): SET LOCAL enable_indexscan = off; trong BEGIN...COMMIT
-- rồi so overlap top-k với kết quả có index -> recall@k
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Index tiết kiệm thời gian thế nào?"** → Linear `O(n)` (trung bình `(n+1)/2`) vs binary/B-tree `O(log n)`. Với 1M dòng: ~500K lần đọc vs ~20 bước.

2. **[MẤU CHỐT] "Vì sao không dùng B-tree cho vector search?"** → Vector nhiều chiều không có thứ tự tuyến tính trên một trục để sort; câu hỏi là "gần nhất" (NN) chứ không phải khớp/khoảng; curse of dimensionality làm cây phân hoạch sập về `O(n)`. → cần ANN.

3. **"IVFFlat vs HNSW?"** → IVFFlat: cụm centroid, `lists`/`probes`, build nhanh + nhẹ, cần data, kém với data động, recall thấp hơn. HNSW: graph nhiều tầng, ~`O(log n)`, recall cao, chịu data động, build lâu + tốn RAM. Default nay là HNSW.

4. **[TRADE-OFF] "Tăng `probes`/`ef_search` được gì mất gì?"** → recall↑ nhưng latency↑. Là núm xoay recall–speed lúc query; tune theo SLA.

5. **[BẪY] "Kết quả 'sai' sau khi đánh index?"** → Không sai — ANN là approximate, đổi recall lấy tốc độ. Tăng tham số hoặc đo recall để định lượng.

6. **[BẪY] "Nên đánh index cho bảng 5000 vector không?"** → Không. Exact brute-force đủ nhanh + recall 100%. Index chỉ đáng khi dữ liệu lớn.

7. **[SCALE] "100M vector, RAM hạn chế?"** → HNSW thuần có thể không vừa RAM → quantization (halfvec/binary) + two-stage re-rank, hoặc DiskANN; partition + iterative_scan cho filter; đo recall & p95.

8. **"Vì sao IVFFlat không nên tạo khi bảng rỗng?"** → Cần data để k-means tìm centroid; bảng rỗng → phân cụm vô nghĩa. HNSW thì tạo được cả khi rỗng.

### 5.6. One-liner đắt giá

- *"An index trades a little storage and build time for a massive drop in search time — but only pays off at scale."*
- *"Binary search needs a line to sort along; vectors don't have one, so we approximate instead of sorting."*
- *"ANN trades recall for speed — the real skill is measuring recall, because it degrades silently with no error to catch it."*
- *"IVFFlat clusters and probes; HNSW climbs a multi-layer highway — same goal, different bet on memory vs build time."*
- *"Not everything needs an index — on small data, exact search is both faster to reason about and 100% correct."*

---

### 📌 Ghi chú cuối

- **Đính chính để nhớ đúng:** HNSW = *Small World* (không phải Small Words); cạnh HNSW theo **metric bạn cấu hình** (không riêng Euclidean); giới hạn **2000 chiều** cho HNSW/IVFFlat index (dùng `halfvec`/DiskANN nếu cao hơn).
- **Kiểm chứng khi ôn:** pgvector hiện là **v0.8.x** (HNSW default; iterative scan; DiskANN qua `pgvectorscale`). Xem CHANGELOG trên GitHub `pgvector/pgvector`.
- **Thực hành:** dựng Postgres + pgvector, nạp ~100K vector, tạo cả IVFFlat và HNSW, chạy `EXPLAIN ANALYZE` để thấy `Index Scan`, rồi đo recall bằng cách tắt index (`enable_indexscan=off`) so top-k. Vặn `probes`/`ef_search` để *thấy* trade-off recall–latency.
- **Nối mạch series (đủ 4 mảnh):** embed (tạo vector) → **index (bài này: làm search nhanh)** → store & query (pgvector) → keyword/FTS → hybrid. Toàn bộ trong một Postgres.
- **Học tiếp:** quantization (scalar/product/binary), two-stage retrieval & reranking, sharding vector (Citus/pgvectorscale), và benchmark recall/latency có hệ thống (ann-benchmarks).
