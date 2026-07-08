# Vector Embeddings: Tạo & Đo Similarity — Chain gối đầu

> Vị trí series: đây là mảnh "TẠO ra vector" — đứng trước pgvector (lưu & tìm) và cạnh FTS (keyword). Nhớ câu ở bài pgvector: "pgvector không sinh embedding, một model mới làm việc đó" → bài này là chỗ model đó xuất hiện.
>
> **Đính chính bài gốc (đã cũ):** (1) `@xenova/transformers` → đổi tên **`@huggingface/transformers`** (Transformers.js v3+, WebGPU). (2) "Vector giới hạn 3 chiều trong toán" là SAI — toán có không gian *n* chiều; chỉ thị giác người giới hạn 3D; embedding thường 384–3072 chiều. (3) Cosine KHÔNG luôn [0,1] — miền toán là **[-1,1]**; chỉ [0,1] khi mọi thành phần không âm. (4) Word2Vec/GloVe/Paragram/USE là nền tảng lịch sử, không còn SOTA; 2026 dẫn đầu OpenAI text-embedding-3, Cohere embed-v4, Voyage, Gemini Embedding 2; mở nguồn BGE-M3, Qwen3-Embedding, Jina v5.

---

## CHAIN — Vấn đề tới embedding

máy tính không "hiểu" chữ: "con mèo đang ngủ" và "chú miu đang thiu thiu" cùng nghĩa nhưng không chung từ nào => khớp ký tự (`LIKE`) hay lexeme (full-text search) đều KHÔNG bắt được sự tương đồng ngữ nghĩa này => cần biến *ý nghĩa* thành thứ máy tính tính toán được => thứ đó là **embedding** => embedding là mảng số thực nhiều chiều biểu diễn ý nghĩa của một object (text/ảnh/video/audio/sensor) => object giống nghĩa được đặt gần nhau bởi một **model tạo embedding** => model tạo embedding là ML đã **pretrained** trên hàng triệu mẫu

## CHAIN — Model, dimension, bản đồ ý nghĩa

model pretrained học cách đặt object giống nghĩa vào vị trí gần nhau => vị trí đó là tọa độ trên một "bản đồ ý nghĩa" nhiều chiều (không phải 2D) => độ dài của mảng tọa độ = **dimension** (số chiều) => dimension tùy model: MiniLM 384, USE 512, OpenAI 3-small 1536 => "vector giới hạn 3 chiều trong toán" là SAI: toán có không gian *n* chiều bất kỳ, chỉ *thị giác con người* giới hạn 3D => trong không gian *n* chiều đó máy vẫn tính khoảng cách bình thường (mở rộng Pythagoras thêm số hạng) => tìm kiếm theo nghĩa = tìm điểm gần nhất trên bản đồ

## CHAIN — Cosine intuition (đo hướng)

đo hai embedding có giống nhau không, cách phổ biến nhất là **cosine similarity** => cosine similarity đo *góc* giữa hai vector => cùng hướng (góc≈0°) → cosine≈1 → rất giống => vuông góc (90°) → cosine=0 → không liên quan => ngược hướng (180°) → cosine=−1 → đối nghĩa/trái ngược => vì đo góc nên cosine chỉ quan tâm **HƯỚNG**, không quan tâm **ĐỘ DÀI** => với text "hướng" chính là ngữ nghĩa nên cosine là lựa chọn mặc định => và miền cosine là **[-1,1]** (không phải [0,1]); chỉ rơi [0,1] khi mọi thành phần vector không âm

## CHAIN — Chạy tay cosine

công thức cosine = `(A·B)/(|A|·|B|)`, với `A·B` = dot product và `|A|` = độ dài (magnitude) => cho A("mèo")=(2,1), B("miu")=(4,2), C("ô tô")=(-1,3) => A·B = 2·4+1·2 = 10; |A|=√5≈2.236, |B|=√20≈4.472 => cosine(A,B) = 10/(2.236·4.472) = **1.00** → "mèo"≈"miu", cùng hướng hoàn toàn (B=2×A) => A·C = 2·(-1)+1·3 = 1; cosine(A,C) = 1/(2.236·3.162) ≈ **0.14** → "mèo" ít liên quan "ô tô" => đây *chính xác* là phép pgvector chạy khi `ORDER BY embedding <=> query`, chỉ khác 384–1536 chiều thay vì 2

## CHAIN — Hello world code & không human-readable

`pipeline('feature-extraction', model)` là "nhà máy" biến text → vector => encode câu cần `pooling:'mean'` + `normalize:true` để ra 1 vector câu (không thì nhận vector từng token) => output là `Float32Array` — mảng số dài, KHÔNG human-readable => không human-readable vì mỗi chiều không có nghĩa riêng dễ diễn giải => chỉ *tổng thể* vector mới mã hóa ngữ nghĩa (dựa trên cách model được train) => nên bạn không *đọc* embedding, bạn *so sánh* nó (bằng `cos_sim`)

## CHAIN — Static vs contextual

**static embeddings** (Word2Vec/GloVe/Paragram, ~2013–2018): mỗi *từ* có đúng MỘT vector cố định bất kể ngữ cảnh => vì cố định nên "bank" trong "river bank" (bờ sông) và "bank account" (ngân hàng) → CÙNG một vector → sai nghĩa => static tạo ra bằng học "từ nào hay đứng cạnh từ nào" (co-occurrence) => **contextual embeddings** (BERT/GPT/USE/transformer) khác ở chỗ vector phụ thuộc CẢ câu chứa nó => nhờ vậy "bank" trong hai câu trên có hai vector khác nhau => contextual thắng vì tính embedding *có điều kiện theo ngữ cảnh* nhờ **self-attention** => nêu được static vs contextual + attention chính là câu trả lời tầm staff cho "embedding là gì"

## CHAIN — Cơ chế static vs contextual (sâu)

Word2Vec/GloVe học một ma trận `|vocab|×d`, mỗi từ tra 1 hàng cố định (nhanh, nhẹ, "một từ một nghĩa") => transformer thì mỗi token đi qua nhiều lớp **self-attention** => self-attention "trộn" thông tin từ các token xung quanh theo trọng số attention => nhờ trộn ngữ cảnh, cùng một từ ở ngữ cảnh khác → vector khác => embedding câu = pool các token contextual này => hệ quả: contextual bắt được "không tệ"≈"khá ổn" (phủ định/sắc thái) mà static bó tay => đổi lại contextual chậm hơn, nặng hơn, phải chạy inference cho *từng* input

## CHAIN — Pooling & normalize

model transformer xuất ra một vector cho MỖI token, không phải một vector cho cả câu => để có vector câu, **mean pooling** lấy *trung bình* các token vector (một số model dùng token `[CLS]`) => sau pooling, **normalize (L2)** chia vector cho độ dài của nó → length = 1 => khi đã normalize, **dot product ≡ cosine similarity** (nhanh hơn) và khoảng cách nhất quán => bỏ hai bước pooling+normalize = nguồn bug số 1 khi tự tạo embedding => cụ thể quên pooling → feature-extraction trả `[1, num_tokens, 384]` (vector từng token) chứ không phải `[1, 384]` → so sánh sai shape

## CHAIN — Ba lỗi tạo embedding

lỗi 1 — quên pooling → so sánh sai shape (đã nêu) => lỗi 2 — trộn embedding từ hai model khác nhau: MiniLM(384) và OpenAI(1536) KHÔNG cùng không gian, khác cả chiều lẫn ý nghĩa tọa độ => vì không cùng không gian, toàn bộ corpus phải dùng CÙNG một model + version => đổi model = re-embed lại tất cả (nối thẳng bài pgvector) => lỗi 3 — không normalize rồi dùng dot product làm "similarity": dot chưa normalize bị *độ dài vector* làm nhiễu (câu dài magnitude lớn hơn) → xếp hạng lệch => fix: normalize trước, hoặc dùng cosine

## CHAIN — Toán cosine (chứng minh đo hướng)

định nghĩa hình học của dot product: `A·B = |A||B|cos(θ)`, θ là góc giữa hai vector => đảo lại: `cos(θ) = (A·B)/(|A||B|)` => chia cho tích độ dài chính là **khử ảnh hưởng của magnitude** => khử magnitude thì chỉ còn lại `cos(θ)` — thuần về hướng => đó là toàn bộ lý do cosine "bỏ qua độ dài" => miền giá trị **[-1,1]** vì cos của mọi góc nằm trong đó => **cosine distance** (pgvector trả cho `<=>`) = `1 − cosine_similarity ∈ [0,2]`, nhỏ = gần

## CHAIN — Cosine ↔ L2 khi đã normalize

trên vector đã normalize (`|A|=|B|=1`): `|A−B|² = |A|²+|B|²−2(A·B) = 2 − 2·cos(θ)` => vì bình phương L2 chỉ phụ thuộc `cos(θ)`, L2 và cosine **đơn điệu tương đương** trên vector normalize => đơn điệu tương đương nghĩa là xếp hạng theo cái này = theo cái kia => nên nhiều hệ chỉ cần normalize rồi dùng dot/L2 tùy tiện mà top-k không đổi

## CHAIN — Dimensionality & Matryoshka

nhiều chiều hơn (3072 vs 384) thường **recall tốt hơn** => nhưng chiều cao tốn RAM/băng thông/tính toán khi search (nối bottleneck RAM ở bài pgvector) => giảm chi phí đó bằng **Matryoshka Representation Learning (MRL)** => MRL (OpenAI v3, Gemini Embedding 2) train sao cho *thông tin quan trọng dồn về ĐẦU vector* => nhờ dồn về đầu, có thể CẮT vector 3072 → 1024/512 với mất mát nhỏ => cắt được thì tự chọn điểm cân bằng chất lượng–chi phí (công cụ tối ưu staff)

## CHAIN — Tự implement & lý do cần pgvector

tự tính cosine: `dot / (norm_a × norm_b)`, guard `denom == 0` để tránh chia 0 => batch nhanh hơn: normalize query + corpus rồi `c @ q` (một phép nhân ma trận) → `sims`, `argsort` lấy top-k => code này tốt để HỌC, nhưng chạy exact trên hàng triệu vector thì memory-intensive & chậm => lấy hết vector ra tính cosine từng cái trong SQL thuần chính là thứ chậm đó => đó là lý do tồn tại pgvector + ANN index (không dùng để exact triệu vector ở production)

## CHAIN — Edge cases embedding

edge case 1 — vector 0/NaN: input rỗng hoặc lỗi encode → chia 0; luôn guard `denom==0` => edge case 2 — **truncation**: model có giới hạn token (256/512), văn bản dài bị CẮT mất đuôi → phải **chunk** tài liệu dài rồi embed từng đoạn => edge case 3 — ngôn ngữ: model English-only cho tiếng Việt kém → dùng model đa ngôn ngữ (BGE-M3/Cohere embed-v4/gtr-multilingual) => edge case 4 — không normalize giữa các batch: trộn vector normalize và chưa normalize → xếp hạng loạn; chuẩn hóa nhất quán toàn hệ => edge case 5 — cosine ra âm: hợp lệ về toán; nếu logic giả định [0,1] thì clip hoặc dùng cosine distance

## CHAIN — Bottleneck pipeline: batching, cost, latency, cache

ở bài này bottleneck là *sản xuất embedding*, không phải công thức cosine => tối ưu số 1: **batching** — gom 32–256 câu mỗi lần gọi model/API → throughput tăng nhiều lần => chi phí API là ràng buộc thật: OpenAI 3-small ~$0.02/1M token => 100M doc × ~200 token = 20 tỉ token ≈ **$400 chỉ để embed MỘT lần** (chưa kể re-embed khi đổi model) => latency: API ~50–200ms/request, self-host GPU ~5–15ms => search real-time cần embed *query* trong đường nóng → self-host hoặc cache query phổ biến => cache được vì embedding của cùng input là *xác định* (cùng model+version) → cache theo `hash(input)`

## CHAIN — Local vs API & privacy

local vs API là đánh đổi lớn => **API** = không vận hành, luôn cập nhật, nhưng tốn tiền theo token + phụ thuộc bên thứ ba + dữ liệu RỜI khỏi hệ thống => dữ liệu rời hệ thống là vấn đề privacy/compliance (gửi văn bản khách hàng tới API có thể vướng GDPR/nội bộ) => vì vậy privacy đôi khi *bắt buộc* self-host, bất kể benchmark => **self-host** (BGE-M3/MiniLM) = miễn phí biên, dữ liệu ở nhà, latency thấp, nhưng phải nuôi GPU + tự cập nhật => nêu rõ privacy là yếu tố quyết định = dấu hiệu staff

## CHAIN — Chọn model (quy trình staff)

đừng chọn theo top MTEB một cách mù quáng => **MTEB** (Massive Text Embedding Benchmark) tốt để *lập shortlist* nhưng điểm benchmark KHÔNG đảm bảo hợp *dữ liệu của bạn* => bước 1: xác định ràng buộc theo thứ tự — modality, ngôn ngữ, self-host được không (privacy/cost), latency, ngân sách => bước 2: shortlist 2–3 model từ MTEB/MMTEB khớp ràng buộc => bước 3: **benchmark trên dữ liệu & truy vấn THẬT của bạn** (recall@k / nDCG trên tập query có nhãn) — bước quyết định => bước 4: cân nhắc **fine-tune** cho domain hẹp (legal/medical/code), thường +10–30% chất lượng => chốt lại: MTEB là prior, không phải verdict

## CHAIN — Migration model

đổi embedding model = **re-embed toàn bộ corpus** vì vector cũ và mới không cùng không gian => chiến lược 1: **version hóa** embedding — cột `embedding_v2` song song `embedding_v1`, backfill nền => chiến lược 2: **blue-green / dual-write** — ghi cả hai version một thời gian, so chất lượng trên traffic thật, rồi cắt sang => trước khi hứa với sếp phải tính chi phí & downtime (dataset lớn = dự án hàng tuần, không phải config một dòng) => nên **chốt model sớm** ở giai đoạn thiết kế để giảm số lần migrate

## CHAIN — Reliability, monitoring, failure modes

API embedding có rate limit & downtime → cần retry/backoff, hàng đợi bất đồng bộ, fallback self-host cho đường nóng => monitoring: tỉ lệ input bị truncate, tỉ lệ vector NULL/lỗi encode, drift recall@k trên tập vàng, chi phí token/ngày => failure mode 1: nhà cung cấp API đổi model version *âm thầm* → vector mới lệch vector cũ → kết quả rác → phải **pin version** rõ ràng => failure mode 2: văn bản dài bị cắt mất thông tin; mode 3: sai ngôn ngữ model; mode 4: trộn normalize/không normalize => cost drivers cần canh: token count (đừng embed rác), số lần re-embed, dimension (Matryoshka cắt)

## CHAIN — Tổ chức & khép series

nói với PM/sếp: chọn model là đánh đổi *chất lượng ↔ chi phí ↔ privacy* => framing bằng chi phí, privacy, và **chi phí thay đổi** (đổi model về sau tốn kém nên chốt sớm) => pipeline embedding (queue/batching/backfill/versioning) là hạ tầng dùng chung nhiều team → xây như một **service**, không nhét rải rác từng feature => và bài này ("tạo vector") khép vào series: text → **embed** → **store & ANN search (pgvector)** → **hybrid với FTS** → RAG => ba mảnh, một Postgres ("model là đầu bếp nấu ra vector, DB pgvector là tủ lạnh cất & lấy")

---

## BẢNG — Landscape model 2026

| Model / Provider | Kiểu | Dimension | Ghi chú |
|---|---|---|---|
| OpenAI `text-embedding-3-small` | API | 1536 | rẻ (~$0.02/1M tok), khởi đầu tốt cho English |
| OpenAI `text-embedding-3-large` | API | 3072 | mạnh, phổ biến, MTEB ~64.6 |
| Cohere `embed-v4` | API | tùy | 100+ ngôn ngữ, dense + sparse, rerank tốt |
| Voyage `voyage-3/4-large` | API | tùy | tối ưu retrieval, top MTEB, domain-specific |
| Google Gemini Embedding 2 | API | 3072 | **multimodal** (text/ảnh/video/audio/PDF), MRL |
| BGE-M3 (BAAI) | Open-source | 1024 | chuẩn mở nguồn, 100+ ngôn ngữ, dense+sparse |
| Qwen3-Embedding / Jina v5 | Open-source | tùy | mở nguồn mạnh, cạnh tranh API |
| `all-MiniLM-L6-v2` | Open-source | 384 | nhẹ, chạy local/browser, default học tập |
| TensorFlow USE | Open-source | 512 | (bài gốc) — cũ, ít dùng cho hệ mới |

**Quy tắc chọn nhanh:** English + đơn giản → OpenAI 3-small; đa ngôn ngữ (kể cả tiếng Việt) → BGE-M3 / Cohere embed-v4; cần local/miễn phí/nhẹ → all-MiniLM / BGE; multimodal → Gemini Embedding 2.

## BẢNG — Metric nào cho việc gì

| Metric | Công thức lõi | Dùng khi | pgvector |
|---|---|---|---|
| **Cosine similarity** | `(A·B)/(|A||B|)` | mặc định cho text; đo hướng ngữ nghĩa | `<=>` |
| **Euclidean (L2)** | `sqrt(Σ(aᵢ-bᵢ)²)` | khi độ lớn vector có nghĩa (ít gặp text) | `<->` |
| **Dot / inner product** | `Σ aᵢbᵢ` | khi đã normalize (≡ cosine, nhanh hơn) | `<#>` |

## BẢNG — Code thuộc lòng

**(a) Tạo embedding (JS, bản mới):**
```javascript
import { pipeline } from '@huggingface/transformers';
const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
const out = await extractor(text, { pooling: 'mean', normalize: true });
const vec = Array.from(out.data);   // 384 số
```

**(b) Tạo embedding (Python):**
```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-MiniLM-L6-v2")
vec = model.encode(text, normalize_embeddings=True)   # numpy (384,)
# hoặc API:
# resp = client.embeddings.create(model="text-embedding-3-small", input=text)
# vec = resp.data[0].embedding   # list 1536 float
```

**(c) Cosine từ số 0 (interviewer hay bắt viết):**
```python
def cosine(a, b):
    dot = sum(x*y for x, y in zip(a, b))
    na  = sum(x*x for x in a) ** 0.5
    nb  = sum(y*y for y in b) ** 0.5
    return dot / (na * nb) if na and nb else 0.0
```

**(d) TensorFlow.js USE (bài gốc):**
```javascript
import * as use from '@tensorflow-models/universal-sentence-encoder';
const model = await use.load();
const embeddings = await model.embed(['hello']);   // tensor [1, 512]
const arr = (await embeddings.array())[0];          // mảng 512 số
```

## BẢNG — Khung system design (50M doc, real-time, Anh+Việt, dữ liệu nhạy cảm)

1. **Clarify:** QPS query? tần suất thêm doc? độ dài doc? recall target? được gọi API ngoài không (privacy)?
2. **Chọn model:** nhạy cảm + đa ngôn ngữ (có tiếng Việt) → nghiêng **self-host BGE-M3/multilingual** thay vì API; benchmark trên query thật trước khi chốt.
3. **Ingestion:** doc mới → **chunk** (tránh truncation) → **batch** embed trên GPU worker → normalize → ghi Postgres+pgvector; bất đồng bộ qua queue.
4. **Query path:** embed query (self-host, latency thấp) → cache query phổ biến → search HNSW trong pgvector → (tùy) hybrid với FTS.
5. **Scale/cost:** 50M × chiều → cân RAM (Matryoshka cắt chiều nếu cần); cache theo hash input; đo chi phí GPU.
6. **Versioning:** pin model version; cột versioned; backfill blue-green khi nâng model.
7. **Monitoring:** truncate rate, encode-error rate, recall@k trên tập vàng, cost/ngày, drift.
8. **API vs self-host:** không vướng privacy + team nhỏ không muốn nuôi GPU → OpenAI/Cohere. Nêu rõ yếu tố quyết định (privacy) = dấu hiệu staff.

## BẢNG — Mental models

- **"Bản đồ ý nghĩa"** → embedding là tọa độ trên bản đồ nhiều chiều.
- **"Cosine = so hướng, không so độ dài"** → câu chốt gọn.
- **"bank hai nghĩa"** → static vs contextual tức thì.
- **"Model đầu bếp, DB cái tủ lạnh"** → model *nấu* ra vector, pgvector *cất & lấy*.
- **"MTEB là prior, data của bạn là verdict"** → chọn model chín chắn.

---

## TỪ KHÓA MỒI

- Vấn đề tới embedding → **"ý nghĩa thành số"**
- Model, dimension, bản đồ → **"n chiều bất kỳ"**
- Cosine intuition → **"cùng hướng → 1"**
- Chạy tay cosine → **"1.00 vs 0.14"**
- Hello world code → **"không human-readable"**
- Static vs contextual → **"bank hai nghĩa"**
- Cơ chế sâu → **"self-attention trộn ngữ cảnh"**
- Pooling & normalize → **"1 vector mỗi token"**
- Ba lỗi tạo embedding → **"không cùng không gian"**
- Toán cosine → **"khử magnitude"**
- Cosine ↔ L2 normalize → **"đơn điệu tương đương"**
- Dimensionality & Matryoshka → **"dồn về đầu vector"**
- Tự implement & pgvector → **"exact triệu vector chậm"**
- Edge cases → **"truncation → chunk"**
- Bottleneck pipeline → **"batching 32-256"**
- Local vs API & privacy → **"dữ liệu rời hệ thống"**
- Chọn model → **"MTEB là prior"**
- Migration → **"re-embed toàn bộ"**
- Reliability/monitoring → **"pin version"**
- Tổ chức & series → **"ba mảnh một Postgres"**

---

## ĐÃ PHỦ

**Nền tảng:** máy tính không đọc chữ (ví dụ mèo/miu), LIKE & lexeme không bắt ngữ nghĩa; embedding (mảng số nhiều chiều, đa modality); model pretrained; dimension (MiniLM 384 / USE 512 / OpenAI 1536,3072); bản đồ ý nghĩa; tìm nghĩa = điểm gần nhất.

**Cosine:** intuition góc (0°→1, 90°→0, 180°→−1); đo hướng bỏ độ dài; miền [-1,1] không phải [0,1]; chạy tay A=(2,1)/B=(4,2)/C=(-1,3) → 1.00 & 0.14; là phép pgvector chạy với `<=>`; cosine distance = 1−sim ∈ [0,2].

**Con số chốt:** dimension từng model; $0.02/1M token; 100M×200 token = 20 tỉ ≈ $400/lần embed; API latency 50–200ms vs GPU 5–15ms; batch 32–256; giới hạn token 256/512; fine-tune +10–30%.

**Static vs contextual:** static (Word2Vec/GloVe/Paragram, 1 vector/từ, co-occurrence, "bank" 1 vector); contextual (BERT/GPT/USE/transformer, phụ thuộc câu, "bank" 2 vector); cơ chế (ma trận |vocab|×d vs self-attention trộn token); contextual bắt phủ định/sắc thái, chậm+nặng hơn; câu trả lời staff.

**Code & pipeline tạo vector:** `pipeline('feature-extraction')`, `pooling:'mean'`+`normalize:true`, Float32Array không human-readable; 3 cách (Transformers.js `@huggingface/transformers`, TF.js USE 512, Python sentence-transformers + OpenAI API cần key); mean pooling ([CLS] alt) + L2 normalize (dot≡cosine); 3 lỗi (quên pooling shape [1,T,384], trộn 2 model không cùng không gian, không normalize dùng dot).

**Toán advanced:** dot = |A||B|cos(θ) → cos(θ)=(A·B)/(|A||B|), khử magnitude; miền [-1,1]; |A−B|²=2−2cos(θ) → L2 & cosine đơn điệu tương đương khi normalize; tự implement (guard denom==0, batch `c@q` argsort); SQL thuần exact triệu vector chậm → lý do có pgvector+ANN.

**Dimensionality:** nhiều chiều recall↑ nhưng RAM/băng thông↑; Matryoshka (MRL, OpenAI v3/Gemini) dồn info về đầu → cắt 3072→1024/512 mất mát nhỏ.

**Edge cases:** vector 0/NaN chia 0; truncation → chunk; ngôn ngữ English-only kém VN → đa ngôn ngữ; trộn normalize/không → loạn; cosine âm hợp lệ, clip/dùng distance nếu giả định [0,1].

**Staff:** bottleneck = sản xuất embedding; batching (tối ưu #1); chi phí API + con số $400; latency API vs GPU + embed query đường nóng; caching hash(input) do embedding xác định; local vs API trade-off + privacy/GDPR ép self-host; chọn model (MTEB shortlist, quy trình 4 bước ràng buộc→shortlist→benchmark data thật recall@k/nDCG→fine-tune, "prior không verdict"); migration (re-embed toàn bộ, version hóa v2 song song, blue-green/dual-write, chốt sớm); reliability (rate limit/retry/backoff/queue/fallback); monitoring (truncate/NULL/drift/cost); 4 failure modes + pin version; cost drivers; tổ chức (framing chất lượng/chi phí/privacy, pipeline là service); series text→embed→pgvector→hybrid FTS→RAG.

**Đính chính bài gốc:** package đổi tên; vector n chiều (không giới hạn 3D); cosine [-1,1] không [0,1]; Word2Vec/GloVe/USE là lịch sử không SOTA + landscape 2026.

**Học tiếp (bài nêu, không đào sâu):** chunking strategy cho RAG, reranking (cross-encoder) sau retrieval, multimodal embeddings (ảnh+text), đánh giá retrieval (recall@k/nDCG/MRR).
