# Vector Embeddings: Tạo & Đo Similarity — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"Vector embeddings in relational databases"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy: vector trong ngữ cảnh AI, cách RDBMS truy cập vector data, các model tạo embedding (Hugging Face, TensorFlow USE, OpenAI, Google), tạo embedding bằng **Xenova Transformers** và **TensorFlow.js**, và đo **cosine similarity**. Tôi giảng bám sát rồi đào sâu. Chỗ mở rộng đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Vị trí trong series:** Đây là mảnh ghép *"tạo ra vector"* — bổ khuyết cho giáo trình **pgvector** (lưu & tìm vector) và giáo trình **full-text search** (keyword). Nhớ luận điểm ở bài pgvector: *"pgvector không sinh embedding, một model mới làm việc đó"* — bài này chính là chỗ model đó xuất hiện.
>
> **⚠️ Cập nhật & đính chính tới 2026 (bài gốc có vài chỗ đã cũ/thiếu chính xác):**
> 1. **`@xenova/transformers` đã đổi tên** thành **`@huggingface/transformers`** (Transformers.js v3+, thêm WebGPU). Package cũ vẫn chạy nhưng nên dùng tên mới.
> 2. **"Vector giới hạn 3 chiều trong toán học" là SAI.** Toán học có không gian vector *n chiều* tùy ý; chỉ *trực giác thị giác* của con người giới hạn ở 3D. Embedding AI thường 384–3072 chiều.
> 3. **Cosine similarity không phải luôn [0,1].** Về mặt toán, nó nằm trong **[-1, 1]**. Chỉ khi mọi thành phần vector không âm thì mới rơi vào [0,1]. Bài gốc nói [0,1] là trường hợp riêng.
> 4. **Word2Vec / GloVe / Paragram / USE là nền tảng lịch sử**, không còn SOTA. 2026 dẫn đầu: OpenAI text-embedding-3, Cohere embed-v4, Voyage, Google Gemini Embedding 2 (multimodal); mở nguồn: BGE-M3, Qwen3-Embedding, Jina v5.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Cách biến một object (chữ, ảnh, câu) thành **embedding** — một mảng số nhiều chiều nắm bắt *ý nghĩa* — bằng các pretrained model, rồi cách **đo độ giống nhau** giữa hai embedding bằng cosine similarity / distance metrics. Đây là bước *"encode"* đứng trước bước *"store & search"* mà pgvector lo.

**Vấn đề nó giải quyết:** Máy tính không "hiểu" chữ. Muốn tìm kiếm theo nghĩa, recommendation, RAG, ta phải chuyển ý nghĩa thành *số* để máy tính so sánh được. Embedding là cây cầu đó: **văn bản giống nghĩa → vector gần nhau**, và "gần nhau" đo được bằng toán (cosine, L2).

**Học xong bạn sẽ làm được:**

- Giải thích embedding là gì trong ngữ cảnh AI và vì sao "khoảng cách vector = độ giống nghĩa".
- Phân biệt static embeddings (Word2Vec/GloVe) vs contextual embeddings (transformer/BERT/GPT) — và vì sao điều đó quan trọng.
- Tạo embedding thật bằng Transformers.js (`@huggingface/transformers`), TensorFlow.js, và OpenAI/Python.
- Tính cosine similarity bằng tay và bằng code; chọn đúng metric.
- Chọn embedding model cho production (MTEB, chi phí, đa ngôn ngữ, migration), và trả lời câu hỏi system design về embedding pipeline.

**Mạch basic → staff:**

- 🟢 **Basic:** Embedding là gì → analogy → model biến text→vector → cosine intuition → code 1 embedding + cosine chạy tay.
- 🟡 **Intermediate:** Static vs contextual embeddings; landscape model; code thật (Transformers.js + TF.js + OpenAI/Python); pooling & normalize; metric nào cho việc gì; lỗi thường gặp.
- 🔴 **Advanced:** Toán cosine + chứng minh, vì sao normalize, dimensionality & Matryoshka, tự implement, batch, edge cases.
- 🟣 **Staff:** Embedding ở quy mô lớn (batch, cost/token, latency, cache), chọn & migrate model, MTEB đúng cách, org impact, câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Vấn đề thực tế: máy tính không đọc được chữ

Bạn có câu "con mèo đang ngủ" và "chú miu đang thiu thiu". Con người thấy hai câu này *gần như cùng nghĩa*. Nhưng với máy tính, chúng chỉ là hai chuỗi ký tự khác nhau — không chung một từ nào (sau khi bỏ "đang"). So khớp ký tự (`LIKE`) hay lexeme (full-text search) đều *không* bắt được sự tương đồng ngữ nghĩa này.

Ta cần cách biến *ý nghĩa* thành thứ máy tính tính toán được. Đó là **embedding**.

### 1.2. Embedding là gì (định nghĩa + analogy)

- **Embedding (vector embedding):** một mảng số thực nhiều chiều biểu diễn *ý nghĩa* của một object (text, ảnh, video, âm thanh, sensor reading...). Ví dụ (rút gọn): "mèo" → `[0.21, -0.87, 0.44, ...]`.
- **Model tạo embedding:** một mô hình machine learning **đã được huấn luyện sẵn** (pretrained) trên hàng triệu mẫu, học được cách đặt object giống nghĩa vào vị trí gần nhau.
- **Dimension (số chiều):** độ dài của mảng. Tùy model: `all-MiniLM-L6-v2` → **384**, TensorFlow USE → **512**, OpenAI `text-embedding-3-small` → **1536**.

**Analogy "bản đồ ý nghĩa":** hình dung một tấm bản đồ khổng lồ (không phải 2D mà hàng trăm chiều). Model đặt mọi từ/câu lên đó theo quy tắc: **nghĩa giống nhau → nằm gần nhau**. "mèo", "miu", "cat" tụm một góc; "ô tô", "xe hơi" tụm góc khác. Tọa độ mỗi điểm chính là embedding của nó. Tìm kiếm theo nghĩa = tìm điểm gần nhất trên bản đồ.

> **[Đính chính bài gốc]** Bài gốc nói "vector giới hạn 3 chiều trong toán". Không đúng: toán học có không gian *n* chiều bất kỳ. Cái giới hạn 3D chỉ là *thị giác con người* — ta không "nhìn" được không gian 384 chiều, nhưng toán vẫn tính khoảng cách trong đó bình thường (chỉ là mở rộng công thức Pythagoras thêm nhiều số hạng).

### 1.3. Đo "gần nhau": cosine similarity (trực giác trước, toán sau)

Cách phổ biến nhất để đo hai embedding có giống nhau không là **cosine similarity** — đo **góc** giữa hai vector:

- Hai vector **cùng hướng** (góc ≈ 0°) → cosine ≈ **1** → *rất giống*.
- Hai vector **vuông góc** (90°) → cosine = **0** → *không liên quan*.
- Hai vector **ngược hướng** (180°) → cosine = **−1** → *đối nghĩa/trái ngược*.

Ý tưởng chủ đạo: **cosine chỉ quan tâm HƯỚNG, không quan tâm ĐỘ DÀI** của vector. Với text, "hướng" chính là ngữ nghĩa — nên cosine là lựa chọn mặc định.

> **[Đính chính bài gốc]** Bài gốc nói cosine "từ 0 đến 1". Chính xác hơn: **[-1, 1]**. Nó chỉ nằm [0,1] khi các vector có mọi thành phần không âm. Embedding hiện đại có cả thành phần âm, nên về lý thuyết có thể ra âm — dù các câu *thật sự giống nghĩa* thường cho cosine dương và cao.

### 1.4. Ví dụ chạy tay — tính cosine bằng tay

Dùng embedding 2 chiều bịa sẵn cho dễ. Công thức:
```
cosine_similarity(A, B) = (A · B) / (|A| · |B|)
   với  A · B = a₁b₁ + a₂b₂          (dot product)
        |A|   = sqrt(a₁² + a₂²)      (độ dài / magnitude)
```

Cho: **A ("mèo") = (2, 1)**, **B ("miu") = (4, 2)**, **C ("ô tô") = (-1, 3)**.

**Similarity A–B:**
- `A · B = 2·4 + 1·2 = 8 + 2 = 10`
- `|A| = sqrt(4+1) = sqrt(5) ≈ 2.236`
- `|B| = sqrt(16+4) = sqrt(20) ≈ 4.472`
- `cosine = 10 / (2.236 · 4.472) = 10 / 10.0 = 1.00`

→ A và B **cùng hướng hoàn toàn** (B = 2×A) → cosine = 1 → "mèo" ≈ "miu". ✅

**Similarity A–C:**
- `A · C = 2·(-1) + 1·3 = -2 + 3 = 1`
- `|C| = sqrt(1+9) = sqrt(10) ≈ 3.162`
- `cosine = 1 / (2.236 · 3.162) = 1 / 7.07 ≈ 0.14`

→ A và C gần vuông góc → cosine ≈ 0.14 → "mèo" ít liên quan "ô tô". ✅

Đây **chính xác** là phép tính pgvector chạy khi bạn query `ORDER BY embedding <=> query`, chỉ khác nó làm với 384–1536 chiều thay vì 2. Toán không đổi.

### 1.5. Code "hello world": tạo 1 embedding + tính cosine

Bài gốc dùng JavaScript (Xenova/TensorFlow.js). Đây là bản **cập nhật package mới** `@huggingface/transformers`:

```javascript
// npm install @huggingface/transformers
import { pipeline, cos_sim } from '@huggingface/transformers';

// 1) Khởi tạo pipeline "feature-extraction" với model tạo embedding
const extractor = await pipeline(
  'feature-extraction',
  'Xenova/all-MiniLM-L6-v2'          // model 384 chiều, nhẹ, chạy được cả trong browser
);

// 2) Encode câu -> vector. LƯU Ý: cần pooling:'mean' + normalize:true
//    để ra 1 vector câu (xem lỗi #1 ở Phần 2). Không có, bạn nhận vector từng token.
const a = await extractor('con mèo đang ngủ',   { pooling: 'mean', normalize: true });
const b = await extractor('chú miu đang thiu thiu', { pooling: 'mean', normalize: true });

console.log(a.dims);          // [1, 384]  -> 1 câu, 384 chiều
console.log(a.data.length);   // 384 số float

// 3) Đo similarity
console.log(cos_sim(a.data, b.data));   // ~0.6-0.8: hai câu gần nghĩa
```

**Điểm khắc cốt:** `pipeline('feature-extraction', model)` là "nhà máy" biến text→vector. Output là `Float32Array` — mảng số dài, **không human-readable** (bài gốc nói đúng: embedding không có dạng con người đọc hiểu được). Bạn không đọc nó, bạn *so sánh* nó.

### 1.6. Ánh xạ về bài gốc

Bài gốc minh họa embedding của chữ "hello": một mảng float dài, độ dài tùy model (Xenova: tùy model; TensorFlow USE: **512**). Nó nhấn mạnh embedding "không tồn tại ở dạng con người suy luận được" — đúng: mỗi chiều không có nghĩa riêng dễ diễn giải, chỉ *tổng thể* mới mã hóa ngữ nghĩa dựa trên cách model được train (xét ký tự, từ trước, từ sau, ngữ cảnh...).

### ✅ Self-check Phần 1

1. Cosine similarity đo *hướng* hay *độ dài* của vector? Vì sao điều đó hợp với text?
2. Hai câu cùng nghĩa cho cosine gần giá trị nào? Hai câu không liên quan?
3. Vì sao embedding "không human-readable" nhưng vẫn hữu dụng?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

### 2.1. Embedding được TẠO RA thế nào — static vs contextual

Đây là phân biệt quan trọng nhất mà bài gốc gộp chung (liệt kê Word2Vec cạnh GPT/BERT):

**Static embeddings (Word2Vec, GloVe, Paragram — thế hệ ~2013-2018):**
- Mỗi *từ* có **đúng một** vector cố định, bất kể ngữ cảnh.
- Vấn đề: từ "bank" trong "river bank" (bờ sông) và "bank account" (ngân hàng) → *cùng một vector* → sai nghĩa.
- Tạo ra bằng cách học "từ nào hay đứng cạnh từ nào" (co-occurrence).

**Contextual embeddings (BERT, GPT, USE, và mọi model hiện đại — dựa trên transformer):**
- Vector của một từ **phụ thuộc cả câu** chứa nó → "bank" trong hai câu trên có hai vector khác nhau.
- Thường ta embed cả *câu/đoạn*, không chỉ từ.
- Đây là lý do model 2026 mạnh hơn hẳn: chúng hiểu ngữ cảnh.

> **[MỞ RỘNG]** Khi phỏng vấn hỏi "embedding là gì", câu trả lời tầm thường là "mảng số". Câu trả lời tầm senior/staff nêu được **static vs contextual** và vì sao transformer thắng: nó tính embedding *có điều kiện theo ngữ cảnh* nhờ attention. Đó là điểm gây ấn tượng.

### 2.2. Landscape model 2026 — ai dùng cái gì (cập nhật ngoài bài gốc)

| Model / Provider | Kiểu | Dimension | Ghi chú |
|---|---|---|---|
| OpenAI `text-embedding-3-small` | API | 1536 | rẻ (~$0.02/1M tok), khởi đầu tốt cho English |
| OpenAI `text-embedding-3-large` | API | 3072 | mạnh, phổ biến, MTEB ~64.6 |
| Cohere `embed-v4` | API | tùy | 100+ ngôn ngữ, xuất cả dense + sparse, rerank tốt |
| Voyage `voyage-3/4-large` | API | tùy | tối ưu retrieval, top MTEB, hợp domain-specific |
| Google Gemini Embedding 2 | API | 3072 | **multimodal** (text/ảnh/video/audio/PDF), MRL |
| BGE-M3 (BAAI) | Open-source | 1024 | chuẩn mở nguồn, 100+ ngôn ngữ, dense+sparse |
| Qwen3-Embedding / Jina v5 | Open-source | tùy | mở nguồn mạnh, cạnh tranh API |
| `all-MiniLM-L6-v2` | Open-source | 384 | nhẹ, chạy local/browser, default học tập |
| TensorFlow USE | Open-source | 512 | (bài gốc) — cũ, ít dùng cho hệ mới |

**Quy tắc chọn nhanh:** English + đơn giản → OpenAI 3-small. Đa ngôn ngữ (kể cả tiếng Việt) → BGE-M3 hoặc Cohere embed-v4. Cần chạy local/miễn phí/nhẹ → all-MiniLM/BGE. Multimodal → Gemini Embedding 2.

### 2.3. Code thật — 3 cách phổ biến

**(a) Transformers.js — chạy local/browser, không cần API key (bản mới):**

```javascript
import { pipeline } from '@huggingface/transformers';

const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');

async function embed(text) {
  const out = await extractor(text, { pooling: 'mean', normalize: true });
  return Array.from(out.data);        // -> mảng 384 số
}

const v = await embed('hello');
console.log(v.length);                 // 384
```

**(b) TensorFlow.js Universal Sentence Encoder (như bài gốc):**

```javascript
// npm install @tensorflow/tfjs @tensorflow-models/universal-sentence-encoder
import * as use from '@tensorflow-models/universal-sentence-encoder';

const model = await use.load();                    // tải pretrained USE
const embeddings = await model.embed(['hello']);   // -> tensor shape [1, 512]
const arr = (await embeddings.array())[0];         // -> mảng 512 số
console.log(arr.length);                            // 512
```

**(c) Python — chuẩn công nghiệp cho ML/RAG (thêm để phỏng vấn):**

```python
# Local, miễn phí:  pip install sentence-transformers
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-MiniLM-L6-v2")
vec = model.encode("hello", normalize_embeddings=True)   # numpy array (384,)

# Hoặc API OpenAI:  pip install openai
from openai import OpenAI
client = OpenAI()  # cần OPENAI_API_KEY
resp = client.embeddings.create(model="text-embedding-3-small", input="hello")
vec = resp.data[0].embedding                              # list 1536 float
```

> Bài gốc nhắc: OpenAI & Google Generative AI cần **API key** lấy từ website, rồi connect bằng key + credentials. Đúng — và đây là điểm chi phí/vận hành sẽ bàn ở Phần 4.

### 2.4. Pooling & normalize — hai bước bài gốc bỏ qua nhưng cực quan trọng

Model transformer xuất ra **một vector cho MỖI token**, không phải một vector cho cả câu. Để có "vector của câu":

- **Pooling (mean pooling):** lấy *trung bình* các token vector → một vector đại diện câu. (Một số model dùng token `[CLS]`.)
- **Normalize (L2):** chia vector cho độ dài của nó → length = 1. Khi đã normalize, **dot product ≡ cosine similarity** (nhanh hơn), và khoảng cách nhất quán.

Bỏ hai bước này = nguồn bug số 1 khi tự tạo embedding (xem 2.5).

### 2.5. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — Quên pooling → so sánh sai shape.** Gọi feature-extraction thô trả về `[1, num_tokens, 384]` (vector từng token), không phải `[1, 384]`. So sánh trực tiếp là sai. Luôn `pooling:'mean', normalize:true` (JS) hoặc `normalize_embeddings=True` (Python).

**Lỗi 2 — Trộn embedding từ hai model khác nhau.** Vector của `all-MiniLM` (384) và OpenAI (1536) **không cùng không gian**, không so sánh được — khác cả chiều lẫn ý nghĩa tọa độ. Toàn bộ corpus phải dùng *cùng một model + version*. Đổi model = **re-embed lại tất cả** (điểm này nối thẳng với bài pgvector).

**Lỗi 3 — Không normalize rồi dùng dot product làm "similarity".** Dot product chưa normalize bị *độ dài vector* làm nhiễu (câu dài có thể có magnitude lớn hơn) → xếp hạng lệch. Normalize trước, hoặc dùng cosine.

### 2.6. Metric nào cho việc gì

| Metric | Công thức lõi | Dùng khi |
|---|---|---|
| **Cosine similarity** | `(A·B)/(|A||B|)` | mặc định cho text; đo hướng ngữ nghĩa |
| **Euclidean (L2)** | `sqrt(Σ(aᵢ-bᵢ)²)` | khi độ lớn vector có nghĩa (ít gặp với text) |
| **Dot / inner product** | `Σ aᵢbᵢ` | khi đã normalize (≡ cosine nhưng nhanh hơn) |

Nhớ liên hệ pgvector: cosine `<=>`, L2 `<->`, inner product `<#>`.

### ✅ Self-check Phần 2

1. "bank" trong "river bank" vs "bank account": static embedding cho mấy vector? Contextual cho mấy?
2. Vì sao phải `pooling:'mean'` khi lấy embedding câu từ transformer?
3. Bạn embed 1 triệu doc bằng model A, rồi muốn đổi sang model B tốt hơn. Chuyện gì phải làm với dữ liệu cũ?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Toán cosine similarity — vì sao nó đo "hướng"

Định nghĩa hình học của dot product: `A · B = |A| |B| cos(θ)`, với θ là góc giữa hai vector. Đảo lại:
```
cos(θ) = (A · B) / (|A| |B|)
```
Chia cho tích độ dài chính là **khử ảnh hưởng của magnitude**, chỉ còn lại `cos(θ)` — thuần về hướng. Đó là toàn bộ lý do cosine "bỏ qua độ dài".

- Miền giá trị: **[-1, 1]** (vì `cos` của mọi góc nằm trong đó).
- **Cosine distance** (thứ pgvector trả cho `<=>`) `= 1 - cosine_similarity ∈ [0, 2]`. Nhỏ = gần.

**Quan hệ cosine ↔ L2 khi đã normalize:** nếu `|A| = |B| = 1` thì
```
|A - B|² = |A|² + |B|² - 2(A·B) = 2 - 2·cos(θ)
```
→ L2 và cosine **đơn điệu tương đương** trên vector đã normalize: xếp hạng theo cái này = theo cái kia. Đây là lý do nhiều hệ chỉ cần normalize rồi dùng dot/L2 tùy tiện mà kết quả top-k không đổi.

### 3.2. Static vs contextual — sâu hơn về cơ chế

- **Word2Vec/GloVe:** học một ma trận `|vocab| × d`; mỗi từ tra 1 hàng cố định. Nhanh, nhẹ, nhưng "một từ một nghĩa".
- **Transformer (BERT/GPT/USE):** mỗi token đi qua nhiều lớp **self-attention** — vector của nó được "trộn" thông tin từ các token xung quanh theo trọng số attention. Kết quả: cùng một từ, ngữ cảnh khác → vector khác. Embedding câu = pool các token contextual này.
- **Hệ quả thực chiến:** contextual bắt được "không tệ" ≈ "khá ổn" (phủ định, sắc thái) mà static bó tay. Đổi lại: chậm hơn, nặng hơn, phải chạy inference cho *từng* input.

### 3.3. Dimensionality & Matryoshka — đánh đổi chất lượng vs chi phí

- Nhiều chiều hơn (3072 vs 384) thường **recall tốt hơn** nhưng tốn RAM/băng thông/tính toán khi search hơn (nối thẳng với bottleneck RAM ở bài pgvector).
- **Matryoshka Representation Learning (MRL):** model (OpenAI v3, Gemini Embedding 2) train sao cho *thông tin quan trọng dồn về đầu vector* → bạn có thể **cắt** vector 3072 xuống 1024/512 với mất mát nhỏ, tự chọn điểm cân bằng chất lượng–chi phí. Đây là công cụ tối ưu chi phí staff cần biết.

### 3.4. Tự implement cosine similarity (chứng tỏ hiểu bản chất)

**NumPy (Python):**
```python
import numpy as np

def cosine_similarity(a: np.ndarray, b: np.ndarray) -> float:
    a = np.asarray(a, dtype=float)
    b = np.asarray(b, dtype=float)
    denom = np.linalg.norm(a) * np.linalg.norm(b)
    if denom == 0:                       # edge case: vector 0 -> tránh chia 0
        return 0.0
    return float(np.dot(a, b) / denom)

# Batch: similarity của 1 query với N vector (vectorized, nhanh)
def cosine_topk(query, corpus, k=5):
    q = query / np.linalg.norm(query)
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)
    sims = c @ q                         # (N,) — một phép nhân ma trận
    idx = np.argsort(-sims)[:k]          # k similarity CAO nhất
    return idx, sims[idx]
```

**Pure JavaScript (không thư viện):**
```javascript
function cosineSimilarity(a, b) {
  let dot = 0, na = 0, nb = 0;
  for (let i = 0; i < a.length; i++) {
    dot += a[i] * b[i];
    na  += a[i] * a[i];
    nb  += b[i] * b[i];
  }
  const denom = Math.sqrt(na) * Math.sqrt(nb);
  return denom === 0 ? 0 : dot / denom;
}
```

> Nhớ điều bài gốc cảnh báo: nếu tự làm mọi thứ trong SQL thuần (lấy hết vector ra, tính cosine từng cái, so sánh) thì **memory-intensive & chậm** — đó chính là lý do tồn tại pgvector + ANN index. Code trên tốt để *học*, không phải để chạy exact search trên hàng triệu vector ở production.

### 3.5. Edge cases cần thủ sẵn

- **Vector 0 / NaN:** input rỗng hoặc lỗi encode → vector 0 → chia 0. Luôn guard `denom == 0`.
- **Truncation:** model có giới hạn token (vd 256/512 token). Văn bản dài bị **cắt** trước khi embed → mất phần đuôi. Phải *chunk* tài liệu dài thành đoạn rồi embed từng đoạn.
- **Ngôn ngữ:** model English-only cho tiếng Việt kết quả kém. Dùng model đa ngôn ngữ (BGE-M3, Cohere embed-v4, gtr-multilingual).
- **Không normalize giữa các batch:** trộn vector normalize và chưa normalize → xếp hạng loạn. Chuẩn hóa nhất quán toàn hệ.
- **Cosine ra âm:** hợp lệ về toán; nếu logic của bạn giả định [0,1] thì phải xử lý (clip hoặc dùng cosine distance).

### ✅ Self-check Phần 3

1. Chứng minh ngắn: vì sao chia cho `|A||B|` khiến cosine "bỏ qua độ dài"?
2. Trên vector đã normalize, xếp hạng theo cosine và theo L2 có khác nhau không? Vì sao?
3. Matryoshka embedding cho phép làm gì để tiết kiệm chi phí?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Embedding ở quy mô lớn — bottleneck là *pipeline tạo vector*, không phải công thức cosine

Ở bài pgvector, bottleneck là *lưu & search*. Ở bài này, bottleneck là *sản xuất embedding*:

- **Batching:** embed từng câu một là lãng phí. Gom batch (32–256 câu) mỗi lần gọi model/API → throughput tăng nhiều lần. Đây là tối ưu số một khi index hàng triệu doc.
- **Chi phí API:** OpenAI 3-small ~$0.02/1M token. 100M doc × ~200 token = 20 tỉ token ≈ **$400 chỉ để embed một lần** — chưa kể re-embed khi đổi model. Con số này phải đưa vào quyết định kiến trúc.
- **Latency:** API embedding ~50–200ms/request; self-host GPU ~5–15ms. Search real-time cần embed *query* trong đường nóng → self-host hoặc cache query phổ biến.
- **Local vs API trade-off:** API = không vận hành, luôn cập nhật, nhưng tốn tiền theo token + phụ thuộc bên thứ ba + dữ liệu rời khỏi hệ thống (vấn đề privacy/compliance). Self-host (BGE-M3, MiniLM) = miễn phí biên, dữ liệu ở nhà, nhưng phải nuôi GPU + tự lo cập nhật.
- **Caching:** embedding của cùng một input là *xác định* (với cùng model+version) → cache theo hash(input) để khỏi trả tiền/tính lại. Tiết kiệm lớn cho nội dung lặp.

### 4.2. Chọn embedding model — quy trình staff

**Đừng chọn theo top MTEB một cách mù quáng.** MTEB (Massive Text Embedding Benchmark) là chuẩn tốt để *lập shortlist*, nhưng điểm benchmark **không** đảm bảo hợp *dữ liệu của bạn*. Quy trình:

1. **Xác định ràng buộc theo thứ tự:** (a) modality (text / multimodal / hybrid dense+sparse), (b) ngôn ngữ, (c) self-host được không (privacy/cost), (d) latency, (e) ngân sách.
2. **Shortlist 2–3 model** từ MTEB/MMTEB khớp ràng buộc.
3. **Benchmark trên dữ liệu & truy vấn thật của bạn** (đo recall@k / nDCG trên tập query có nhãn). Đây là bước quyết định.
4. Cân nhắc **fine-tune** cho domain hẹp (legal/medical/code) — thường +10–30% chất lượng.

> **One-liner staff:** *"MTEB narrows the shortlist; your own labeled queries make the decision. Benchmark rank is a prior, not a verdict."*

### 4.3. Migration model — quyết định kiến trúc nặng ký nhất

Nối thẳng cảnh báo ở bài pgvector: **đổi embedding model = re-embed toàn bộ corpus**, vì vector cũ và mới không cùng không gian. Chiến lược staff:

- **Version hóa** embedding: cột `embedding_v2` song song `embedding_v1`; backfill nền.
- **Blue-green / dual-write:** ghi cả hai version một thời gian, so chất lượng trên traffic thật, rồi cắt sang.
- **Tính chi phí & downtime** trước khi hứa với sếp. Với dataset lớn, đây là dự án hàng tuần, không phải cấu hình một dòng.
- **Chốt model sớm** ở giai đoạn thiết kế để giảm số lần phải migrate.

### 4.4. Cost / reliability / monitoring / failure modes

- **Cost drivers:** token count (chunk hợp lý, đừng embed rác), số lần re-embed, dimension (chiều cao → storage/search đắt — cân nhắc Matryoshka để cắt).
- **Reliability:** API embedding có rate limit & downtime → cần retry/backoff, hàng đợi bất đồng bộ, và fallback (self-host dự phòng cho đường nóng).
- **Monitoring:** tỉ lệ input bị truncate; tỉ lệ vector NULL/lỗi encode; drift chất lượng retrieval theo thời gian (recall@k trên tập vàng); chi phí token/ngày.
- **Failure modes:** (1) đổi model version *âm thầm* (API nhà cung cấp cập nhật) → vector mới lệch vector cũ → kết quả rác; **pin version** rõ ràng; (2) văn bản dài bị cắt mất thông tin; (3) sai ngôn ngữ model; (4) trộn normalize/không normalize.

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Embedding là bộ não hiểu-nghĩa của tính năng search/RAG. Chọn model là đánh đổi *chất lượng ↔ chi phí ↔ quyền riêng tư dữ liệu*: dùng API ngoài thì nhanh nhưng dữ liệu rời hệ thống và tính tiền theo lượng; tự host thì dữ liệu ở nhà, rẻ về biên, nhưng cần đầu tư hạ tầng. Và đổi model về sau tốn kém vì phải xử lý lại toàn bộ dữ liệu — nên ta chốt sớm." → framing bằng **chi phí, privacy, và chi phí thay đổi**.
- **Privacy/compliance:** gửi văn bản khách hàng tới API bên thứ ba có thể vướng GDPR/nội bộ → đôi khi *bắt buộc* self-host, bất kể benchmark. Staff phải nêu ràng buộc này sớm.
- **Roadmap:** pipeline embedding (queue, batching, backfill, versioning) là hạ tầng dùng chung nhiều team → nên xây như một service, không nhét rải rác trong từng feature.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Thiết kế hệ thống sinh & phục vụ embedding cho semantic search trên 50 triệu tài liệu, hỗ trợ thêm tài liệu real-time, tiếng Anh + tiếng Việt, dữ liệu khách hàng nhạy cảm."**

**Khung trả lời staff (nói to trade-off):**

1. **Clarify:** QPS query? tần suất thêm doc? độ dài doc? recall target? ràng buộc privacy (được gọi API ngoài không)?
2. **Chọn model:** dữ liệu nhạy cảm + đa ngôn ngữ (có tiếng Việt) → nghiêng **self-host BGE-M3/multilingual** (dữ liệu ở nhà, tiếng Việt tốt) thay vì API; benchmark trên query thật trước khi chốt.
3. **Ingestion pipeline:** doc mới → **chunk** (tránh truncation) → **batch** embed trên GPU worker → normalize → ghi vào Postgres+pgvector (cột `embedding` + metadata). Bất đồng bộ qua queue để chịu tải real-time.
4. **Query path:** embed query (self-host, latency thấp) → cache query phổ biến → search HNSW trong pgvector → (tùy) hybrid với full-text (nối bài FTS).
5. **Scale/cost:** 50M × chiều → cân RAM (Matryoshka cắt chiều nếu cần); cache embedding theo hash input; đo chi phí GPU.
6. **Versioning & migration:** pin model version; cột versioned; kế hoạch backfill blue-green khi nâng model.
7. **Monitoring:** truncate rate, encode-error rate, recall@k trên tập vàng, cost/ngày, drift.
8. **Khi nào chọn API thay vì self-host:** nếu *không* vướng privacy và team nhỏ không muốn nuôi GPU → OpenAI/Cohere đơn giản hơn. **Nêu rõ đâu là yếu tố quyết định (privacy) = dấu hiệu staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **Embedding** — mảng số nhiều chiều mã hóa ý nghĩa của object; giống nghĩa → gần nhau.
- **Dimension** — số chiều của vector (MiniLM 384, USE 512, OpenAI 1536/3072).
- **Static embedding** — 1 vector cố định/từ (Word2Vec, GloVe); "một từ một nghĩa".
- **Contextual embedding** — vector phụ thuộc ngữ cảnh (BERT/GPT/transformer); "bank" hai nghĩa hai vector.
- **Transformer / self-attention** — cơ chế cho phép embedding phụ thuộc ngữ cảnh.
- **Pooling (mean pooling)** — gộp vector token → vector câu.
- **Normalize (L2)** — đưa vector về length 1; khi đó dot ≡ cosine.
- **Cosine similarity** — `(A·B)/(|A||B|)`, đo góc, miền [-1,1]; distance = 1 − sim.
- **Dot / inner product, Euclidean (L2)** — các metric khác; nối pgvector `<#>`, `<->`.
- **Feature-extraction pipeline** — task tạo embedding trong Transformers.js.
- **Transformers.js (`@huggingface/transformers`)** — chạy model HF trong JS/browser (tên mới của `@xenova/transformers`).
- **TensorFlow.js USE** — Universal Sentence Encoder, 512 chiều.
- **MTEB / MMTEB** — benchmark chuẩn so sánh embedding model.
- **Matryoshka (MRL)** — cắt bớt chiều vector với mất mát nhỏ để tiết kiệm.
- **Chunking / truncation** — chia doc dài; model có giới hạn token.
- **Re-embedding** — đổi model = tính lại toàn bộ vector.

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. Embedding biến ý nghĩa thành số; giống nghĩa → vector gần nhau → so sánh được bằng toán.
2. **Model tạo embedding, không phải database.** DB (pgvector) chỉ lưu & tìm.
3. **Static vs contextual** là phân biệt cốt lõi; transformer thắng vì hiểu ngữ cảnh (attention).
4. Cosine đo **hướng** (bỏ độ dài) → hợp text; miền **[-1,1]**, không phải [0,1].
5. Transformer xuất vector/token → phải **pooling + normalize** để có vector câu.
6. Vector từ hai model/version khác nhau **không so sánh được**; đổi model = **re-embed toàn bộ**.
7. Chọn model: MTEB để shortlist, benchmark trên **dữ liệu thật** để quyết.
8. Ở quy mô lớn: **batch, cache, chunk**; chi phí token & latency là ràng buộc thật.
9. **Privacy** (dữ liệu gửi API ngoài) đôi khi ép self-host, bất kể benchmark.
10. Chuỗi hoàn chỉnh: text → **embed (bài này)** → **store & ANN search (pgvector)** → (hybrid với **FTS**) → RAG.

### 5.3. Ideas / mental models

- **"Bản đồ ý nghĩa":** embedding là tọa độ trên bản đồ nhiều chiều.
- **"Cosine = so hướng, không so độ dài":** một câu chốt gọn.
- **"bank hai nghĩa":** minh họa static vs contextual tức thì.
- **"Model đầu bếp, DB cái tủ lạnh":** model *nấu* ra vector, pgvector *cất & lấy*.
- **"MTEB là prior, data của bạn là verdict":** chọn model chín chắn.

### 5.4. Code cần thuộc lòng

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
vec = model.encode(text, normalize_embeddings=True)
```

**(c) Cosine similarity từ số 0 (interviewer hay bắt viết):**
```python
def cosine(a, b):
    dot = sum(x*y for x, y in zip(a, b))
    na  = sum(x*x for x in a) ** 0.5
    nb  = sum(y*y for y in b) ** 0.5
    return dot / (na * nb) if na and nb else 0.0
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Embedding là gì?"** → Mảng số nhiều chiều mã hóa ý nghĩa; giống nghĩa → gần nhau. Ghi điểm bằng cách nêu **contextual** (transformer) khác **static** (Word2Vec).

2. **"Vì sao dùng cosine chứ không Euclidean cho text?"** → Cosine đo hướng (ngữ nghĩa), bỏ độ dài; text quan tâm hướng. Nếu đã normalize thì L2 và cosine tương đương về xếp hạng.

3. **[BẪY] "Cosine similarity chạy từ 0 đến 1 đúng không?"** → Không hẳn: miền toán là **[-1,1]**; chỉ [0,1] khi vector không âm. Các câu thật sự giống nghĩa thường cho dương cao.

4. **[BẪY] "Embed câu bằng transformer, so hai câu thấy shape lệch, vì sao?"** → Quên **pooling**: feature-extraction trả vector/token `[1, T, d]`. Cần `pooling:'mean', normalize:true`.

5. **"Đổi embedding model tốt hơn thì làm gì với dữ liệu cũ?"** → Re-embed toàn bộ (vector cũ/mới khác không gian); version hóa cột, backfill, blue-green. Đây là quyết định kiến trúc nặng.

6. **[TRADE-OFF] "Self-host model hay dùng API?"** → API: không vận hành, luôn mới, nhưng tốn tiền/token, phụ thuộc bên thứ ba, dữ liệu rời hệ thống (privacy). Self-host: rẻ biên, dữ liệu ở nhà, latency thấp, nhưng nuôi GPU + tự cập nhật. Privacy thường là yếu tố quyết định.

7. **[SCALE] "Embed 100 triệu doc, tối ưu gì?"** → Chunk hợp lý (tránh truncation) → batch → GPU → cache theo hash → cân dimension (Matryoshka). Tính chi phí token/GPU trước.

8. **"Chọn embedding model thế nào?"** → Ràng buộc (modality/ngôn ngữ/privacy/latency/cost) → shortlist từ MTEB → benchmark trên query thật → cân nhắc fine-tune. MTEB là prior, không phải phán quyết.

### 5.6. One-liner đắt giá

- *"The model makes the vector, the database stores it — pgvector never embeds anything itself."*
- *"Cosine measures direction, not magnitude — that's exactly what you want for meaning."*
- *"Static embeddings give one vector per word; contextual embeddings give one vector per word *in context* — that's why transformers won."*
- *"Vectors from two different models don't live in the same space, so changing your embedding model means re-embedding everything."*
- *"MTEB narrows the shortlist; your own labeled queries make the decision."*
- *"Privacy can override the benchmark — sometimes you self-host a slightly weaker model because the data can't leave the building."*

---

### 📌 Ghi chú cuối

- **Kiểm chứng khi ôn:** package JS giờ là **`@huggingface/transformers`** (v3+, WebGPU); `@xenova/transformers` là tên cũ. Landscape model đổi liên tục — xem **MTEB leaderboard** trên Hugging Face trước phỏng vấn.
- **Thực hành:** cài `sentence-transformers` (Python) hoặc `@huggingface/transformers` (JS), embed vài câu, tự tính cosine bằng hàm ở 5.4, kiểm chứng câu cùng nghĩa cho cosine cao. Rồi cắm vào pgvector để đóng vòng: **embed → store → search**.
- **Nối mạch series:** bài này (**tạo vector**) + bài pgvector (**lưu & tìm vector**) + bài FTS (**keyword**) = pipeline retrieval hoàn chỉnh cho **semantic search & RAG**. Ba mảnh, một Postgres.
- **Học tiếp:** chunking strategy cho RAG, reranking (cross-encoder) sau bước embedding retrieval, multimodal embeddings (ảnh + text), và đánh giá retrieval (recall@k, nDCG, MRR).
