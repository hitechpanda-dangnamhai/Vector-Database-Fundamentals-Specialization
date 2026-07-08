# Phân tích chain học tập — Vector Embeddings: Tạo & Đo Similarity

> **Lưu ý định dạng:** File gốc KHÔNG phải tài liệu chain (`=>`) như hai bài trước — nó là **giáo trình văn xuôi** Basic→Staff kèm cheatsheet. Để giữ đúng mục tiêu (tách *suy ra được* khỏi *phải ghi nhớ*), tôi **tái dựng các chain logic ẩn** trong mạch giảng, rồi phân loại A/B/C và rút xương sống. Phần Cheatsheet (5.x) và các bảng landscape/metric là nhóm B trình bày sẵn — dùng ở mục 9, không phân tích như chain.

Đây là **mảnh ghép thứ 3** của series: **tạo vector** (bài này) → **lưu & tìm** (pgvector, file 1) → **keyword/hybrid** (FTS, file 2) → RAG. Cách học đúng vẫn là **tái tạo**.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN (tái dựng từ văn xuôi)

## Chain 1 — Vấn đề: máy không đọc được chữ → embedding

**A — Nhân quả**
- máy tính chỉ thấy chuỗi ký tự => "con mèo đang ngủ" và "chú miu đang thiu thiu" là hai chuỗi khác nhau — *vì* không chung từ nào sau khi bỏ "đang".
- keyword (`LIKE`) và lexeme (FTS) so ký tự/từ => không bắt được tương đồng ngữ nghĩa => cần biến *ý nghĩa* thành số — *vì* phải có thứ máy tính tính toán được để so.

**B — Ghi nhớ**
- **embedding** — *định nghĩa*: mảng số thực nhiều chiều biểu diễn ý nghĩa của object (text/ảnh/video/âm thanh/sensor).
- **model tạo embedding** — *định nghĩa*: mô hình ML **pretrained**, học đặt object giống nghĩa vào vị trí gần nhau.
- **dimension** — *con số*: MiniLM **384**, TF USE **512**, OpenAI 3-small **1536**.
- "bản đồ ý nghĩa" — *mental model*: nghĩa giống nhau → nằm gần nhau.

> ⚠ **Đính chính bài gốc (nhóm B phải nhớ):** "vector giới hạn 3 chiều trong toán" là SAI — toán có không gian *n* chiều; chỉ *thị giác người* giới hạn 3D. Đây là chỗ bài gốc dạy sai, phải ghi đè.

**Xương sống**: keyword/lexeme miss nghĩa → cần số hóa ý nghĩa → embedding (model pretrained sinh ra).

> Nối file 2: chính là chỗ "so ký tự / so lexeme" bó tay mà bài FTS đã chỉ ra.

---

## Chain 2 — Cosine similarity (trực giác đo góc)

**A — Nhân quả**
- cosine chỉ quan tâm **hướng**, không quan tâm **độ dài** => hợp với text — *vì* với text "hướng" chính là ngữ nghĩa, độ dài vector không mang nghĩa.

**B — Ghi nhớ**
- **cosine similarity** = đo góc: cùng hướng (0°) → **1** (rất giống); vuông góc (90°) → **0** (không liên quan); ngược hướng (180°) → **−1** (đối nghĩa) — *định nghĩa/con số*.

> ⚠ **Đính chính bài gốc:** cosine nằm **[-1, 1]**, KHÔNG phải [0,1]. Chỉ rơi [0,1] khi mọi thành phần vector không âm. Embedding hiện đại có thành phần âm → có thể ra âm.

**Xương sống**: đo giống nhau = đo góc → cosine bỏ độ dài, giữ hướng → hợp text.

---

## Chain 3 — Chạy tay cosine

**A — Nhân quả**
- (không có) — worked example thuần.

**B — Ghi nhớ**
- Công thức: `cosine(A,B) = (A·B)/(|A|·|B|)`, với `A·B = a₁b₁+a₂b₂`, `|A| = sqrt(a₁²+a₂²)` — *công thức*.
- Ví dụ: A(2,1) "mèo", B(4,2) "miu", C(-1,3) "ô tô"; A·B=10, |A|≈2.236, cosine A–B = **1.00** (B=2×A, cùng hướng); cosine A–C ≈ **0.14** (gần vuông góc) — *con số*.

**C — Một thứ hai tên**
- "tính tay cosine 2 chiều" ⟺ "pgvector chạy `ORDER BY embedding <=> query`" — **cùng một phép tính**, chỉ khác 2 chiều vs 384–1536 chiều. Đừng tìm chữ "vì" ở đây. (Song song hệt file 1 chain 4: tính tay L2 ⟺ `<->`.)

**Xương sống**: công thức cosine → ví dụ ra 1.00 & 0.14 → đúng phép pgvector chạy, chỉ khác số chiều.

---

## Chain 4 — Static vs contextual embedding

**A — Nhân quả**
- static gán **một** vector cố định cho mỗi từ => "bank" trong "river bank" và "bank account" cùng một vector => sai nghĩa — *vì* không xét ngữ cảnh.
- transformer dùng **self-attention** trộn thông tin token xung quanh => vector một từ phụ thuộc cả câu => "bank" hai câu ra hai vector => transformer thắng — *vì* nó hiểu ngữ cảnh.

**B — Ghi nhớ**
- **static embedding**: Word2Vec, GloVe, Paragram (~2013–2018); học co-occurrence ("từ nào hay đứng cạnh từ nào") — *tên gọi/định nghĩa*.
- **contextual embedding**: BERT, GPT, USE (transformer); embed cả câu/đoạn — *tên gọi/định nghĩa*.
- **self-attention** — *tên gọi*: cơ chế cho embedding phụ thuộc ngữ cảnh.

> ⚠ **Đính chính bài gốc:** Word2Vec/GloVe/USE là **nền tảng lịch sử**, không còn SOTA. 2026: OpenAI text-embedding-3, Cohere embed-v4, Voyage, Gemini Embedding 2; mở nguồn BGE-M3, Qwen3-Embedding, Jina v5.

**Xương sống**: static = 1 vector/từ → mù ngữ cảnh ("bank") → contextual (self-attention) phụ thuộc câu → transformer thắng.

---

## Chain 5 — Pooling & normalize

**A — Nhân quả**
- transformer xuất **một vector cho MỖI token** => phải **pooling** (mean) để có một vector đại diện câu — *vì* không pool thì bạn có `[1, T, d]` (vector từng token), không so được.
- normalize L2 về length 1 => **dot product ≡ cosine similarity** (nhanh hơn) — *vì* khi |A|=|B|=1 thì mẫu số biến mất.

**B — Ghi nhớ**
- **pooling (mean pooling)** — *định nghĩa*: trung bình các token vector → vector câu (một số model dùng token `[CLS]`).
- **normalize (L2)** — *định nghĩa*: chia vector cho độ dài → length = 1.

**Xương sống**: transformer ra vector/token → pooling để có vector câu → normalize L2 → dot ≡ cosine.

---

## Chain 6 — Ba lỗi thường gặp khi tự tạo embedding

**A — Nhân quả (mỗi lỗi có "vì" riêng)**
- quên pooling => so sánh sai shape — *vì* feature-extraction thô trả `[1, T, 384]` chứ không phải `[1, 384]`.
- trộn embedding hai model khác nhau => không so sánh được — *vì* MiniLM (384) và OpenAI (1536) **khác cả chiều lẫn ý nghĩa tọa độ**, không cùng không gian → đổi model = **re-embed toàn bộ**.
- không normalize rồi dùng dot => xếp hạng lệch — *vì* độ dài vector (câu dài magnitude lớn) làm nhiễu.

> ⚠ Đây là **danh sách 3 lỗi**, không phải chuỗi nhân quả nối nhau. Mỗi lỗi tự đứng; học như checklist. (Lỗi 2 nối thẳng file 1: "đổi model = re-embed".)

**Xương sống**: (checklist) quên pooling → sai shape / trộn 2 model → khác không gian / không normalize → dot nhiễu.

---

## Chain 7 — Toán cosine: vì sao đo hướng (chứng minh)

**A — Nhân quả**
- định nghĩa hình học `A·B = |A||B|cos(θ)` => đảo lại `cos(θ) = (A·B)/(|A||B|)` => chia cho tích độ dài = **khử magnitude**, chỉ còn hướng — *vì* phép chia loại bỏ đúng thành phần độ dài.
- normalize (|A|=|B|=1) => `|A-B|² = 2 - 2cos(θ)` => L2 và cosine **đơn điệu tương đương** về xếp hạng — *vì* L2 là hàm giảm của cosine khi đã chuẩn hóa.

**B — Ghi nhớ**
- Miền cosine = **[-1, 1]** (vì `cos` mọi góc nằm đó) — *con số*.
- **cosine distance** (pgvector `<=>`) = `1 - cosine_similarity ∈ [0, 2]`, nhỏ = gần — *công thức/con số*.

**Xương sống**: dot = |A||B|cosθ → chia tích độ dài khử magnitude → thuần hướng; normalize thì L2 ≡ cosine về xếp hạng.

---

## Chain 8 — Dimensionality & Matryoshka

**A — Nhân quả**
- nhiều chiều hơn (3072 vs 384) => recall tốt hơn nhưng tốn RAM/băng thông/tính toán khi search — *vì* mỗi vector to hơn (nối bottleneck RAM ở file 1).
- MRL train sao cho **thông tin quan trọng dồn về đầu vector** => cắt 3072 → 1024/512 với mất mát nhỏ — *vì* phần đầu đã giữ hầu hết tín hiệu.

**B — Ghi nhớ**
- **Matryoshka Representation Learning (MRL)** — *tên gọi*: OpenAI v3, Gemini Embedding 2; công cụ tối ưu chi phí staff cần biết.

**Xương sống**: chiều cao → recall tốt nhưng đắt → Matryoshka cắt chiều (thông tin dồn về đầu) → tự chọn điểm cân bằng.

---

## Chain 9 — Edge cases cần thủ sẵn

**A — Nhân quả (mỗi cái có "vì" riêng)**
- vector 0/NaN => chia 0 → guard `denom == 0` — *vì* input rỗng/lỗi encode cho vector 0.
- văn bản dài => bị **truncate** mất đuôi → phải **chunk** — *vì* model có giới hạn token (vd 256/512).

**B — Ghi nhớ (checklist)**
- Ngôn ngữ: model English-only cho tiếng Việt kém → dùng model đa ngôn ngữ (BGE-M3, Cohere embed-v4).
- Không normalize giữa các batch → xếp hạng loạn → chuẩn hóa nhất quán toàn hệ.
- Cosine ra âm là hợp lệ; nếu code giả định [0,1] phải clip hoặc dùng cosine distance.

> ⚠ Đây là **danh sách 5 edge case**, không phải nhân quả. Học như checklist thủ sẵn.

**Xương sống**: (checklist) guard chia 0 / chunk chống truncation / model đúng ngôn ngữ / normalize nhất quán / cosine âm hợp lệ.

---

## Chain 10 — Vì sao pgvector/ANN tồn tại

**A — Nhân quả**
- tự lấy hết vector ra tính cosine từng cái trong SQL thuần => memory-intensive & chậm => đó **chính là lý do** pgvector + ANN index tồn tại — *vì* exact search trên hàng triệu vector không khả thi real-time.

**B — Ghi nhớ**
- Code cosine tự viết tốt để *học*, không phải để chạy exact trên hàng triệu vector production — *nhận định*.

**Xương sống**: exact cosine trên hàng triệu vector chậm/tốn RAM → cần pgvector + ANN index (đúng nội dung file 1).

---

## Chain 11 — Embedding ở quy mô lớn (bottleneck = tạo vector)

**A — Nhân quả**
- ở bài này bottleneck là **sản xuất embedding** (khác pgvector: lưu & search) => embed từng câu là lãng phí → **batch** (32–256) tăng throughput — *vì* gọi model/API theo lô rẻ hơn nhiều lần.
- search real-time cần embed **query** trong đường nóng => phải self-host (latency thấp) hoặc cache query phổ biến — *vì* API 50–200ms quá chậm cho đường nóng.
- embedding của cùng input là **xác định** (cùng model+version) => cache theo `hash(input)` — *vì* kết quả không đổi nên tính lại là phí.

**B — Ghi nhớ**
- Batch **32–256** câu/lần; API OpenAI 3-small ~**$0.02/1M token**; 100M doc × ~200 token ≈ 20 tỉ token ≈ **$400** embed một lần; latency API **50–200ms** vs self-host GPU **5–15ms** — *con số*.

**Xương sống**: bottleneck = tạo vector → batch để tăng throughput → query đường nóng cần self-host/cache → embedding xác định nên cache theo hash.

---

## Chain 12 — Chọn model & migration

**A — Nhân quả**
- MTEB benchmark KHÔNG đảm bảo hợp *dữ liệu của bạn* => phải benchmark trên **data & query thật** mới quyết — *vì* điểm chung chung không nói lên hiệu quả trên domain cụ thể.
- vector cũ và mới **không cùng không gian** => đổi model = **re-embed toàn bộ corpus** => version hóa cột + blue-green + tính chi phí + chốt sớm — *vì* không thể so vector giữa hai không gian.

**B — Ghi nhớ**
- **MTEB / MMTEB** — *tên gọi*: benchmark chuẩn để *lập shortlist* (prior, không phải phán quyết).
- Quy trình: ràng buộc (modality/ngôn ngữ/self-host/latency/budget) → shortlist 2–3 từ MTEB → benchmark data thật → cân nhắc fine-tune (+10–30% domain hẹp) — *quy trình*.
- Migration: cột `embedding_v2` song song, dual-write/blue-green, coi như dự án hàng tuần — *thực hành*.

> Đây là bản thứ ba của mẫu hình "đổi nền tảng = làm lại toàn bộ" (file 1: re-embed; file 2: reindex config).

**Xương sống**: MTEB shortlist → benchmark data thật mới quyết → đổi model = re-embed (khác không gian) → version hóa, blue-green, chốt sớm.

---

## Chain 13 — API vs self-host & privacy

**A — Nhân quả**
- gửi văn bản khách hàng tới API bên thứ ba => vướng GDPR/nội bộ => đôi khi **bắt buộc self-host, bất kể benchmark** — *vì* dữ liệu không được rời hệ thống.

**B — Ghi nhớ**
- **API**: không vận hành, luôn mới, nhưng tốn tiền/token + phụ thuộc bên thứ ba + dữ liệu rời hệ thống.
- **Self-host** (BGE-M3, MiniLM): rẻ biên, dữ liệu ở nhà, latency thấp, nhưng phải nuôi GPU + tự cập nhật.

> ⚠ **Failure mode im lặng (nhớ):** nhà cung cấp API đổi model version *âm thầm* → vector mới lệch vector cũ → kết quả rác → **pin version** rõ ràng.

**Xương sống**: privacy (data rời hệ thống) → có thể ép self-host bất kể benchmark; framing cho sếp bằng chi phí ↔ privacy ↔ chi phí thay đổi.

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC (của bài này)

> keyword/lexeme miss nghĩa → cần số hóa ý nghĩa → **embedding** (model pretrained sinh ra, "bản đồ ý nghĩa") → đo giống nhau = **cosine** (đo góc, bỏ độ dài, miền **[-1,1]**) → toán: `cos = dot/(|A||B|)` khử magnitude → **static** (1 vector/từ, mù ngữ cảnh) thua **contextual** (self-attention, phụ thuộc câu) → transformer ra vector/token nên phải **pooling + normalize** → normalize thì dot ≡ cosine, L2 ≡ cosine → chiều cao recall tốt nhưng đắt → **Matryoshka** cắt chiều → ở quy mô lớn bottleneck là **tạo vector** → **batch + cache + chunk** → chọn model bằng **MTEB (prior) + benchmark data thật (verdict)** → đổi model = **re-embed toàn bộ** (khác không gian) → **privacy** có thể ép self-host → tự tính cosine hàng triệu vector là sai → dùng **pgvector + ANN** (file 1).

### 🗺️ META-XƯƠNG SỐNG CẢ SERIES (3 file)

Tài liệu tự khai đây là mảnh ghép thứ ba. Ghép cả ba xương sống thành **một mạch retrieval hoàn chỉnh**:

> **text → [ENCODE] embed (file 3: model sinh vector, cosine)** → **[STORE & SEARCH] pgvector (file 1: lưu, HNSW/IVFFlat, ANN, đánh đổi recall↔speed)** → **[KEYWORD] full-text search (file 2: tsvector/GIN, so lexeme)** → **[HYBRID] FTS + vector fuse bằng RRF trong một Postgres** → **RAG**.

Ba mảnh, một Postgres. Nắm meta-mạch này là thấy cả ba tài liệu chỉ là một hệ thống.

---

## 8. MẪU HÌNH LẶP LẠI (nhấn mạnh cross-file — vì đây là mảnh thứ 3)

**Mẫu hình 1 — Cosine = so hướng, bỏ độ dài.**
Bài này 2 (trực giác) + 7 (chứng minh) ↔ file 1 chain 6 (L2 nhạy độ dài → text cần góc). *Vì sao là một*: text quan tâm ngữ nghĩa = hướng, nên mọi bài đều chọn cosine và bỏ magnitude.

**Mẫu hình 2 — "Tính tay = phép DB chạy" (một thứ hai cách nhìn).**
Bài này 3 (cosine tay ⟺ `<=>`) ↔ file 1 chain 4 (L2 tay ⟺ `<->`). *Vì sao là một*: worked example và toán tử SQL là **cùng một phép tính**, chỉ khác số chiều — đừng tìm nhân quả giữa chúng.

**Mẫu hình 3 — Đổi quyết định nền tảng = làm lại toàn bộ dữ liệu phái sinh.** ⭐ mẫu hình mạnh nhất xuyên 3 file.
Bài này 6 & 12 (đổi model = re-embed) ↔ file 1 (re-embed corpus) ↔ file 2 (đổi config = reindex bảng). *Vì sao là một*: vector/tsvector đều sinh từ một "cấu hình gốc" (model / config); đổi gốc thì mọi thứ phái sinh không cùng không gian nữa → phải build lại → luôn version hóa + coi như migration.

**Mẫu hình 4 — Hai thứ chỉ so được khi qua CÙNG một phép biến đổi / cùng không gian.**
Bài này 5 & 6 (pooling+normalize nhất quán, không trộn 2 model) ↔ file 2 (doc & query cùng bộ chuẩn hóa) ↔ file 1 (ops class khớp toán tử). *Vì sao là một*: so sánh chỉ có nghĩa khi hai vế đồng nhất về cách xử lý.

**Mẫu hình 5 — Lỗi im lặng.**
Bài này: đổi model version âm thầm, vector 0/NaN chia 0, truncation cắt mất đuôi, trộn normalize/không (6, 9, 13) ↔ file 1 (ops class sai bỏ index, NULL embedding) ↔ file 2 (NULL nuốt tsvector, config lệch). *Vì sao là một*: hệ không báo lỗi, chỉ trả kết quả kém → phải pin version, guard, monitor.

**Mẫu hình 6 — Núm vặn recall ↔ chi phí.**
Bài này 8 (dimension cao recall tốt nhưng tốn RAM; Matryoshka cắt) ↔ file 1 (quantization, ef_search, exact vs ANN). *Vì sao là một*: cùng trục đánh đổi — đẩy chất lượng lên thì trả giá bằng RAM/tiền/tốc độ.

**Mẫu hình 7 — Làm rẻ cái đắt bằng gom lô & tránh lặp.**
Bài này 11 (batch, cache theo hash, chunk) ↔ file 1 (two-stage re-rank trên N nhỏ) ↔ file 2 (filter cứng trước rồi rank top-N). *Vì sao là một*: giảm khối lượng của bước đắt trước khi bỏ công tính.

**Mẫu hình 8 — Operational simplicity: ba mảnh, một Postgres.**
Meta-pattern của cả series: encode + store&search + keyword/hybrid gói trong một Postgres, không dựng Pinecone + Elasticsearch riêng.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

(Gộp Cheatsheet 5.x + số liệu rải rác + 4 đính chính.)

### Con số
- Dimension: MiniLM **384** · TF USE **512** · OpenAI 3-small **1536** · 3-large **3072** · BGE-M3 **1024** · Gemini Embedding 2 **3072**.
- Cosine similarity miền **[-1, 1]** (chỉ [0,1] khi vector không âm); cosine distance = `1 - sim ∈ [0, 2]`.
- Ví dụ chạy tay: A(2,1), B(4,2), C(-1,3); A·B=10, |A|≈2.236; cosine A–B = **1.00**; A–C ≈ **0.14**.
- Batch **32–256** câu/lần; OpenAI 3-small ~**$0.02/1M token**; 100M doc ≈ **$400** embed một lần; latency API **50–200ms** vs self-host **5–15ms**; fine-tune domain hẹp **+10–30%**; giới hạn token model ~**256/512**; OpenAI 3-large MTEB ~**64.6**.

### Định nghĩa & tên gọi
- **embedding**, **dimension**, **model pretrained**, "bản đồ ý nghĩa".
- **static embedding** (Word2Vec/GloVe/Paragram, co-occurrence, "một từ một nghĩa") vs **contextual embedding** (BERT/GPT/USE, transformer, **self-attention**).
- **pooling** (mean pooling / `[CLS]`), **normalize (L2)** (→ dot ≡ cosine).
- **cosine similarity/distance**, **dot / inner product**, **Euclidean (L2)**; liên hệ pgvector: `<=>` cosine, `<->` L2, `<#>` inner product.
- **feature-extraction pipeline**; **Transformers.js** = `@huggingface/transformers` (tên cũ `@xenova/transformers`, v3+ có WebGPU); **TensorFlow.js USE** (512).
- **MTEB / MMTEB**; **Matryoshka (MRL)**; **chunking / truncation**; **re-embedding**.
- Landscape 2026: OpenAI text-embedding-3, Cohere embed-v4, Voyage, Gemini Embedding 2 (multimodal), BGE-M3, Qwen3-Embedding, Jina v5.

### ⚠ Bốn đính chính bài gốc (phải ghi đè)
1. Vector **không** giới hạn 3 chiều — chỉ thị giác người giới hạn 3D; embedding thường 384–3072 chiều.
2. Cosine similarity miền **[-1, 1]**, không phải [0,1].
3. Word2Vec/GloVe/USE là **lịch sử**, không còn SOTA.
4. Package JS: **`@huggingface/transformers`** (không phải `@xenova/transformers` cũ).

### Cú pháp
- **JS (mới):** `pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2')`; `extractor(text, { pooling:'mean', normalize:true })`; `cos_sim(a, b)`; `Array.from(out.data)`.
- **Python:** `SentenceTransformer("all-MiniLM-L6-v2")`; `model.encode(text, normalize_embeddings=True)`.
- **OpenAI:** `client.embeddings.create(model="text-embedding-3-small", input=text)` → `resp.data[0].embedding`.
- **TF.js USE:** `use.load()`; `model.embed(['hello'])` → `[1, 512]`.
- **cosine từ số 0** (interviewer hay bắt): `dot/(na*nb)`, guard `denom == 0 → 0`.
- **cosine_topk NumPy:** normalize q & corpus → `sims = c @ q` → `argsort(-sims)[:k]`.

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Che tài liệu, trả lời bằng lời của mình. Ấp úng = lỗ hổng cần đào, không phải "chưa thuộc".

1. Vì sao keyword/lexeme search không bắt được "con mèo đang ngủ" ≈ "chú miu đang thiu thiu"?
2. Vì sao cosine đo **hướng** chứ không phải **độ dài** — chứng minh ngắn từ `A·B = |A||B|cos(θ)`?
3. Vì sao cosine similarity nằm **[-1, 1]** chứ không phải [0,1]?
4. Vì sao "bank" trong "river bank" và "bank account" cần hai vector khác nhau, và static embedding sai ở đâu?
5. Vì sao transformer (self-attention) tạo được contextual embedding còn Word2Vec thì không?
6. Vì sao phải **pooling** khi lấy embedding câu từ transformer?
7. Vì sao **normalize** khiến dot product ≡ cosine, và L2 ≡ cosine về xếp hạng?
8. Vì sao vector từ hai model/version khác nhau **không so sánh được**?
9. Vì sao đổi embedding model buộc **re-embed toàn bộ** corpus? (nối cả 3 file)
10. Vì sao **Matryoshka** cho phép cắt chiều với mất mát nhỏ?
11. Vì sao ở quy mô lớn, bottleneck là **sản xuất vector** chứ không phải công thức cosine?
12. Vì sao query trong **đường nóng** cần self-host/cache, còn ingestion thì batch API cũng được?
13. Vì sao **MTEB** chỉ là "prior" chứ không phải "phán quyết" khi chọn model?
14. Vì sao **privacy** đôi khi ép self-host bất kể benchmark?
15. Vì sao tự tính cosine trên hàng triệu vector trong SQL thuần là sai ở production — và pgvector/ANN giải quyết thế nào?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- **Nhóm A**: tự hỏi "vì sao", hiểu là xong. Bài này nhiều A ngon (cosine đo hướng, static vs contextual, pooling, re-embed) — đây là phần dễ nhớ nhất vì có logic.
- **Nhóm B**: học thuộc mục 9. Chú ý **4 đính chính** — đó là chỗ bài gốc dạy sai, phải ghi đè chứ không chỉ ghi nhớ.
- **Tái tạo**: dựng lại xương sống bài này (mục 7), rồi dựng **meta-xương sống 3 file** (encode → store&search → keyword/hybrid → RAG). Đó là bức tranh lớn nhất của cả series.
- Ưu tiên **8 mẫu hình** (mục 8): phần lớn trùng file 1 & 2, nên hiểu một lần dùng cho cả ba tài liệu — đặc biệt "đổi nền tảng = làm lại toàn bộ" và "lỗi im lặng".
