# CAPSTONE — Vector Databases với PostgreSQL: Tổng Hợp Toàn Khóa & Đi Phỏng Vấn
*CAPSTONE — Vector Databases with PostgreSQL: Full Course Summary and Interview Preparation*

> **Nguồn gốc**  
> *Source*
>
> Video gốc có tên *"Congratulations / Course wrap-up"*.  
> *The original video is titled "Congratulations / Course wrap-up".*
>
> Video thuộc Course 3 của IBM Vector Database Fundamentals.  
> *The video belongs to Course 3 of IBM Vector Database Fundamentals.*
>
> Video không có nội dung kỹ thuật mới.  
> *The video contains no new technical content.*
>
> Video tổng kết toàn khóa.  
> *The video summarizes the entire course.*
>
> Nội dung recap gồm lexeme và FTS.  
> *The recap covers lexemes and FTS.*
>
> Nội dung recap gồm model tạo embedding — vector biểu diễn ý nghĩa.  
> *The recap covers embedding models.*
>
> Nội dung recap gồm cosine similarity — độ tương đồng cosin.  
> *The recap covers cosine similarity.*
>
> Nội dung recap gồm pgvector cho dữ liệu non-text.  
> *The recap covers pgvector for non-text data.*
>
> Nội dung recap gồm hướng dẫn về `INSERT`, `<=>`, `COPY` và pg-promise.  
> *The recap covers guidance for `INSERT`, `<=>`, `COPY`, and pg-promise.*
>
> Nội dung recap gồm housekeeping cuối khóa.  
> *The recap covers end-of-course housekeeping.*
>
> Tôi không soạn một giáo trình song song lặp lại.  
> *I do not create another repetitive course guide.*
>
> Thay vào đó, tôi tạo một giáo trình CAPSTONE.  
> *Instead, I create a CAPSTONE course guide.*
>
> Giáo trình kết nối cả 8 giáo trình trước thành một hệ thống.  
> *The guide connects all eight earlier guides into one system.*
>
> Giáo trình cung cấp một bản đồ tổng.  
> *The guide provides an overall map.*
>
> Giáo trình giải thích quan hệ giữa mọi mảnh ghép.  
> *The guide explains the relationship among all components.*
>
> Giáo trình cung cấp một capstone project — dự án tổng kết.  
> *The guide provides a capstone project.*
>
> Bạn có thể build project này.  
> *You can build this project.*
>
> Giáo trình cung cấp một master cheatsheet — bảng ghi nhớ tổng hợp.  
> *The guide provides a master cheatsheet.*
>
> Bạn có thể ôn nhanh cả khóa trước phỏng vấn.  
> *You can review the whole course before an interview.*
>
> Bài gốc chỉ là recap.  
> *The original lesson is only a recap.*
>
> Toàn bộ tài liệu này là phần tổng hợp ngoài bài gốc.  
> *This entire guide is a synthesis beyond the original lesson.*
>
> Toàn bộ tài liệu này là phần mở rộng ngoài bài gốc.  
> *This entire guide is an extension beyond the original lesson.*
>
> **Bộ 8 giáo trình của khóa**  
> *The course set of eight guides*
>
> Thứ tự sau là thứ tự học đề xuất.  
> *The following order is the recommended learning order.*
>
> 1. `giao-trinh-vector-embeddings.md` tạo vector.  
> *1. `giao-trinh-vector-embeddings.md` creates vectors.*
>
> 2. `giao-trinh-vector-indexing.md` làm search nhanh.  
> *2. `giao-trinh-vector-indexing.md` makes search fast.*
>
> Tài liệu đi từ binary search đến IVFFlat và HNSW.  
> *The guide moves from binary search to IVFFlat and HNSW.*
>
> 3. `giao-trinh-rdbms-vector-platforms.md` hướng dẫn chọn nền tảng.  
> *3. `giao-trinh-rdbms-vector-platforms.md` explains platform selection.*
>
> Các nền tảng gồm Postgres, MySQL và MariaDB.  
> *The platforms include Postgres, MySQL, and MariaDB.*
>
> 4. `giao-trinh-pgvector.md` trình bày khái niệm store và query.  
> *4. `giao-trinh-pgvector.md` presents storage and query concepts.*
>
> 5. `giao-trinh-pgvector-hands-on.md` hướng dẫn cài pgvector.  
> *5. `giao-trinh-pgvector-hands-on.md` explains pgvector installation.*
>
> Tài liệu cũng hướng dẫn sử dụng pgvector.  
> *The guide also explains pgvector usage.*
>
> 6. `giao-trinh-bulk-insert-pgvector.md` hướng dẫn nạp dữ liệu quy mô lớn.  
> *6. `giao-trinh-bulk-insert-pgvector.md` explains large-scale data loading.*
>
> 7. `giao-trinh-query-vector-pgvector.md` trình bày query và threshold.  
> *7. `giao-trinh-query-vector-pgvector.md` presents queries and thresholds.*
>
> 8. `giao-trinh-postgres-fts.md` trình bày full-text search và hybrid.  
> *8. `giao-trinh-postgres-fts.md` presents full-text search and hybrid search.*

---

## Phần 0 — 🗺️ Bản đồ toàn khóa: 8 mảnh là MỘT hệ thống
*Part 0 — 🗺️ Full course map: eight components form ONE system*

Cả khóa trả lời một câu hỏi lớn.  
*The entire course answers one major question.*

**Làm sao biến PostgreSQL thành một hệ semantic search — tìm kiếm theo nghĩa?**  
***How can PostgreSQL become a semantic search system?***

**Làm sao hệ thống hỗ trợ RAG — tạo sinh tăng cường bằng truy xuất?**  
***How can the system support retrieval-augmented generation?***

**Làm sao hệ thống chạy được trong môi trường production?**  
***How can the system run in production?***

Tám chủ đề không rời rạc.  
*The eight topics are not isolated.*

Tám chủ đề là các trạm trên một pipeline — đường ống xử lý.  
*The eight topics are stations in one pipeline.*

```
        [1] EMBED                [2] INDEX            [3] PLATFORM
   text/ảnh → vector      làm k-NN search nhanh    Postgres? MySQL? MariaDB?
   text/images → vector   fast k-NN search         Postgres? MySQL? MariaDB?
   (model AI)             (IVFFlat / HNSW)          → chọn pgvector
   (AI model)             (IVFFlat / HNSW)          → choose pgvector
        │                        │                        │
        └──────────┬─────────────┴────────────┬───────────┘
                   ▼                           ▼
            [5] CÀI pgvector            [4] STORE model
            [5] INSTALL pgvector        [4] STORE model
         (Docker/apt/managed)      (schema: cột vector + metadata)
         (Docker/apt/managed)      (schema: vector column + metadata)
                   │
                   ▼
            [6] BULK INSERT ──────► [7] QUERY ──────► [8] HYBRID (＋FTS)
         (COPY, nạp trước           (ORDER BY <=>       keyword ＋ semantic
         (COPY, load first          (ORDER BY <=>       keyword ＋ semantic
          index sau)                 LIMIT ＋ threshold)  fuse bằng RRF
          index later)               LIMIT ＋ threshold)  fuse with RRF
                                          │
                                          ▼
                                  SEMANTIC SEARCH / RAG
```

### Đọc bản đồ
*Reading the map*

Bạn tạo vector ở trạm 1.  
*You create vectors at station 1.*

Một model AI tạo các vector này.  
*An AI model creates these vectors.*

Bạn chọn index — cấu trúc tìm kiếm nhanh ở trạm 2.  
*You choose an index at station 2.*

Index giúp hệ thống tìm nhanh.  
*The index helps the system search quickly.*

Bạn chọn nền tảng lưu trữ ở trạm 3.  
*You choose the storage platform at station 3.*

Lựa chọn trong bản đồ là pgvector.  
*The choice in the map is pgvector.*

Bạn thiết kế schema — cấu trúc dữ liệu ở trạm 4.  
*You design the schema at station 4.*

Bạn cài extension — tiện ích mở rộng ở trạm 5.  
*You install the extension at station 5.*

Bạn nạp dữ liệu hiệu quả ở trạm 6.  
*You load data efficiently at station 6.*

Bạn thực hiện query đúng ở trạm 7.  
*You run correct queries at station 7.*

Bạn kiểm soát chất lượng query ở trạm 7.  
*You control query quality at station 7.*

Bạn có thể ghép full-text search với vector search ở trạm 8.  
*You can combine full-text search with vector search at station 8.*

Cách ghép này tạo hybrid search — tìm kiếm lai.  
*This combination creates hybrid search.*

Kết quả là một hệ retrieval — truy xuất hoàn chỉnh.  
*The result is a complete retrieval system.*

Hệ retrieval là nền tảng của mọi ứng dụng RAG.  
*The retrieval system is the foundation of every RAG application.*

### Ba câu chốt xuyên suốt cả khóa
*Three key statements across the course*

- Embedding biến dữ liệu phức tạp thành vector có thể so sánh.  
  *Embeddings turn complex data into comparable vectors.*

- Dữ liệu phức tạp gồm ảnh, audio, video và text.  
  *Complex data includes images, audio, video, and text.*

- Cosine gần 1 biểu thị mức giống nhau cao.  
  *A cosine value near 1 indicates high similarity.*

- Toán tử `<=>` trả về distance — khoảng cách.  
  *The `<=>` operator returns distance.*

- Distance bằng 1 trừ similarity.  
  *Distance equals 1 minus similarity.*

- pgvector hỗ trợ dữ liệu non-text.  
  *pgvector supports non-text data.*

- `tsvector` và FTS chỉ hỗ trợ text hoặc keyword.  
  *`tsvector` and FTS support only text or keywords.*

---

## Phần 1 — 🟢 BASIC: Toàn hệ thống trong một đoạn + ví dụ end-to-end nhỏ nhất
*Part 1 — 🟢 BASIC: the entire system in one passage plus the smallest end-to-end example*

### 1.1. Giải thích cả khóa cho người mới trong 5 câu
*1.1. The entire course for beginners in five sentences*

Máy tính không hiểu chữ.  
*Computers do not understand text.*

Máy tính không hiểu ảnh.  
*Computers do not understand images.*

Ta dùng một model AI cho dữ liệu đầu vào.  
*We use an AI model for the input data.*

Model biến dữ liệu thành embedding.  
*The model turns the data into embeddings.*

Embedding là một mảng số biểu diễn ý nghĩa.  
*An embedding is a meaning-bearing numeric array.*

Các nội dung giống nghĩa có vector nằm gần nhau.  
*Semantically similar content has nearby vectors.*

Ta lưu embedding trong PostgreSQL.  
*We store embeddings in PostgreSQL.*

Extension pgvector hỗ trợ việc lưu trữ này.  
*The pgvector extension supports this storage.*

Ta dùng index HNSW.  
*We use an HNSW index.*

HNSW tìm các hàng xóm gần nhất rất nhanh.  
*HNSW finds nearest neighbors very quickly.*

HNSW vẫn hoạt động với hàng triệu vector.  
*HNSW still works with millions of vectors.*

Người dùng gửi một câu hỏi.  
*The user submits a question.*

Hệ thống embed câu hỏi.  
*The system embeds the question.*

Hệ thống tìm các bản ghi có vector gần nhất.  
*The system finds records with the nearest vectors.*

Quy trình này là semantic search.  
*This process is semantic search.*

Ta có thể thêm full-text search.  
*We can add full-text search.*

Full-text search tìm theo keyword — từ khóa.  
*Full-text search uses keywords.*

Sự kết hợp tạo thành hybrid search.  
*The combination creates hybrid search.*

Hybrid search cho kết quả tốt nhất.  
*Hybrid search provides the best results.*

### 1.2. Ví dụ end-to-end nhỏ nhất
*1.2. The smallest end-to-end example*

Ví dụ chứa cả 8 mảnh trong khoảng 15 dòng.  
*The example contains all eight components in about 15 lines.*

```sql
-- [5] CÀI + [4] SCHEMA
-- [5] INSTALL + [4] SCHEMA
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE docs (id bigserial PRIMARY KEY, content text, embedding vector(384));

-- [6] NẠP (ở đây insert tay; production dùng COPY, nạp trước index sau)
-- [6] LOAD (this example uses manual inserts; production uses COPY and builds the index later)
INSERT INTO docs (content, embedding) VALUES ('...', '[...]'), ('...', '[...]');

-- [2] INDEX (sau khi nạp)
-- [2] INDEX (after loading)
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);

-- [7] QUERY: 5 kết quả gần nhất, chỉ nhận "đủ giống" (threshold)
-- [7] QUERY: 5 nearest results with a similarity threshold
SELECT content, 1 - (embedding <=> :q) AS similarity
FROM docs
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q
LIMIT 5;
```

`:q` là embedding của câu truy vấn.  
*`:q` is the embedding of the query.*

Model ở trạm 1 tạo embedding này.  
*The model at station 1 creates this embedding.*

Model đó cũng đã embed bảng `docs`.  
*The same model has also embedded the `docs` table.*

Đây là toàn bộ pipeline thu nhỏ.  
*This is the entire pipeline in miniature.*

### ✅ Self-check Phần 1
*✅ Part 1 self-check*

1. Hãy kể 4 bước từ một câu chữ đến kết quả semantic search.  
*1. Name four steps from a text sentence to semantic search results.*

2. Tại sao câu truy vấn phải dùng cùng model với dữ liệu bảng?  
*2. Why must the query use the same model as the table data?*

3. Dòng nào tạo index trong ví dụ trên?  
*3. Which line creates the index in the example?*

4. Dòng nào đặt threshold trong ví dụ trên?  
*4. Which line sets the threshold in the example?*

---

## Phần 2 — 🟡 INTERMEDIATE: Đi dọc pipeline, mỗi trạm một quyết định
*Part 2 — 🟡 INTERMEDIATE: follow the pipeline, one decision at each station*

Mỗi trạm có một quyết định cốt lõi.  
*Each station has one core decision.*

Bạn nắm chuỗi quyết định này.  
*You understand this decision sequence.*

Khi đó bạn nắm toàn bộ khóa học.  
*At that point, you understand the entire course.*

| # / No. | Trạm / Station | Quyết định cốt lõi / Core decision | Chốt nhanh / Quick summary |
|---|---|---|---|
| 1 | **Embed / Embed** | Chọn model nào? / Which model? | Tiếng Anh: OpenAI 3-small, đa ngữ hoặc tiếng Việt: BGE-M3, nhẹ hoặc local: MiniLM, đổi model: re-embed tất cả / English: OpenAI 3-small, multilingual or Vietnamese: BGE-M3, lightweight or local: MiniLM, model change: re-embed everything |
| 2 | **Index / Index** | Chọn index nào? / Which index? | Nhỏ dưới 50K: không index và exact, quy mô vừa: HNSW mặc định, ít RAM hoặc dữ liệu tĩnh: IVFFlat hay DiskANN / Small below 50K: no index and exact search, medium scale: default HNSW, low RAM or static data: IVFFlat or DiskANN |
| 3 | **Platform / Platform** | RDBMS-native hay dedicated? / RDBMS-native or dedicated? | Vector là feature dưới 50M: pgvector, vector là product hoặc đạt tỷ vector: Pinecone hay Milvus / Vector as a feature below 50M: pgvector, vector as a product or at billion scale: Pinecone or Milvus |
| 4 | **Store / Store** | Schema thế nào? / What schema? | Cột `vector(n)` với `n` khớp model cạnh business data, hybrid query trong một câu / A `vector(n)` column with model-matching `n` beside business data, one-query hybrid search |
| 5 | **Cài / Install** | Cài kiểu gì? / How to install? | Docker, apt hoặc managed, không build source trên production, pin version, cài file khác `CREATE EXTENSION` / Docker, apt, or managed service, no source build in production, pinned version, file installation differs from `CREATE EXTENSION` |
| 6 | **Nạp / Load** | Insert kiểu gì? / How to insert? | COPY nhanh nhất, nạp trước, tạo index sau, idempotent qua temp và upsert / COPY is fastest, load first, build the index later, idempotency through temp tables and upsert |
| 7 | **Query / Query** | Lấy ra thế nào? / How to retrieve? | `ORDER BY <=> LIMIT k` kiểm soát số lượng, threshold kiểm soát chất lượng, loại chính bản ghi đó / `ORDER BY <=> LIMIT k` controls quantity, threshold controls quality, exclude the record itself |
| 8 | **Hybrid / Hybrid** | Keyword cộng semantic? / Keyword plus semantic? | FTS với `tsvector` và `@@` bắt từ khóa, vector bắt nghĩa, RRF fuse kết quả / FTS with `tsvector` and `@@` captures keywords, vectors capture meaning, RRF fuses results |

### Mạch nối các trạm
*Connections among the stations*

Một sự thật có thể đi xuyên nhiều trạm.  
*One fact can span several stations.*

- Cùng model xuất hiện ở trạm 1.  
  *The same-model rule appears at station 1.*

- Trạm 1 tạo vector.  
  *Station 1 creates vectors.*

- Cùng model xuất hiện ở trạm 4.  
  *The same-model rule appears at station 4.*

- Trạm 4 yêu cầu chiều schema khớp model.  
  *Station 4 requires the schema dimension to match the model.*

- Cùng model xuất hiện ở trạm 7.  
  *The same-model rule appears at station 7.*

- Trạm 7 yêu cầu query dùng cùng model.  
  *Station 7 requires the query to use the same model.*

- Quy tắc cùng model là một sợi chỉ đỏ.  
  *The same-model rule is a unifying thread.*

- Quy tắc nạp trước index sau xuất hiện ở trạm 6.  
  *The load-first rule appears at station 6.*

- Index maintenance — bảo trì index có chi phí cao.  
  *Index maintenance has a high cost.*

- Kiến thức này thuộc trạm 2.  
  *This knowledge belongs to station 2.*

- Threshold xuất hiện ở trạm 7.  
  *The threshold appears at station 7.*

- Threshold phụ thuộc metric — thước đo đã chọn.  
  *The threshold depends on the selected metric.*

- Metric được chọn ở trạm 1 hoặc trạm 4.  
  *The metric is selected at station 1 or station 4.*

- Cosine có miền distance từ 0 đến 2.  
  *Cosine distance ranges from 0 to 2.*

- L2 có miền distance đến vô cực.  
  *L2 distance extends to infinity.*

### ✅ Self-check Phần 2
*✅ Part 2 self-check*

1. Dữ liệu tiếng Việt có tính nhạy cảm.  
*1. The Vietnamese data is sensitive.*

Bạn nên chọn model nào ở trạm 1?  
*Which model should you choose at station 1?*

Bạn nên chọn platform nào ở trạm 3?  
*Which platform should you choose at station 3?*

2. Tại sao quy tắc nạp trước index sau liên quan đến trạm 2?  
*2. Why does the load-first rule relate to station 2?*

3. Một query hybrid gồm hai nhánh nào?  
*3. Which two branches form a hybrid query?*

Cơ chế nào fuse hai nhánh?  
*Which mechanism fuses the two branches?*

---

## Phần 3 — 🔴 ADVANCED: Bốn chủ đề xuyên suốt
*Part 3 — 🔴 ADVANCED: four cross-cutting topics*

Đây là các connective insights — hiểu biết kết nối nhiều phần.  
*These are connective insights across multiple sections.*

Các ý này không thuộc riêng một bài.  
*These ideas do not belong to only one lesson.*

Các ý này xuyên qua nhiều bài.  
*These ideas span many lessons.*

Chúng phân biệt người học vẹt với người hiểu hệ thống.  
*They distinguish rote learners from system thinkers.*

### 3.1. Approximate ↔ Recall: cái giá của tốc độ
*3.1. Approximate ↔ Recall: the price of speed*

Chủ đề đi từ trạm 2 đến trạm 7.  
*The topic spans station 2 through station 7.*

Trạm 2 liên quan đến index.  
*Station 2 concerns indexes.*

Trạm 7 liên quan đến query threshold.  
*Station 7 concerns query thresholds.*

ANN đổi recall lấy tốc độ.  
*ANN trades recall for speed.*

Không có index tạo exact search.  
*No index produces exact search.*

Exact search đạt recall 100%.  
*Exact search achieves 100% recall.*

Exact search có độ phức tạp `O(n)`.  
*Exact search has `O(n)` complexity.*

HNSW hoặc IVFFlat tạo search nhanh.  
*HNSW or IVFFlat creates fast search.*

Recall có thể thấp hơn 100%.  
*Recall can fall below 100%.*

Index có thể bỏ sót vài hàng xóm thật.  
*The index can miss some true neighbors.*

Hệ thống áp threshold lên kết quả xấp xỉ.  
*The system applies the threshold to approximate results.*

Kết quả cuối có thể bị sót.  
*The final results can contain omissions.*

Ai cũng có thể nói về vector search.  
*Anyone can talk about vector search.*

Người hiểu xác định vị trí trên đường cong recall–speed.  
*An expert identifies a position on the recall–speed curve.*

Người hiểu biết cách đo vị trí đó.  
*An expert knows how to measure that position.*

Hãy so sánh ANN với exact định kỳ.  
*Compare ANN with exact search periodically.*

### 3.2. Luận điểm "một Postgres cho tất cả"
*3.2. The "one Postgres for everything" argument*

Chủ đề kết nối trạm 3 với trạm 8.  
*The topic connects station 3 with station 8.*

Trạm 3 liên quan đến platform.  
*Station 3 concerns the platform.*

Trạm 8 liên quan đến FTS.  
*Station 8 concerns FTS.*

Siêu năng lực của pgvector không phải tốc độ đỉnh.  
*The superpower of pgvector is not peak speed.*

Siêu năng lực nằm ở vector cạnh business data.  
*The superpower comes from vectors beside business data.*

Thiết kế này hỗ trợ filter trong một câu SQL.  
*This design supports filtering in one SQL statement.*

Thiết kế này hỗ trợ join trong một câu SQL.  
*This design supports joins in one SQL statement.*

Thiết kế này hỗ trợ hybrid trong một câu SQL.  
*This design supports hybrid search in one SQL statement.*

Mọi thao tác nằm trong một transaction ACID.  
*Every operation stays in one ACID transaction.*

Dedicated vector database như Pinecone buộc dùng hai hệ thống.  
*A dedicated vector database like Pinecone requires two systems.*

Kiến trúc này cần hai round-trip.  
*This architecture needs two round trips.*

Kiến trúc này cần cơ chế đồng bộ.  
*This architecture needs synchronization.*

Kim chỉ nam là một câu hỏi.  
*The guiding principle is one question.*

**Vector là feature hay product?**  
***Is vector search a feature or a product?***

Feature phù hợp với Postgres.  
*A feature fits Postgres.*

Product phù hợp với dedicated vector database.  
*A product fits a dedicated vector database.*

Quy mô tỷ vector phù hợp với dedicated vector database.  
*Billion-vector scale fits a dedicated vector database.*

Triển khai đa vùng phù hợp với dedicated vector database.  
*Multi-region deployment fits a dedicated vector database.*

### 3.3. Chi phí ẩn lớn nhất: re-embedding và lock-in
*3.3. The largest hidden cost: re-embedding and lock-in*

Lock-in là sự phụ thuộc vào một nhà cung cấp.  
*Lock-in is dependence on one provider.*

Chủ đề kết nối trạm 1, trạm 3 và trạm 6.  
*The topic connects stations 1, 3, and 6.*

Bạn đổi embedding model.  
*You change the embedding model.*

Khi đó bạn phải re-embed toàn bộ corpus — tập dữ liệu.  
*At that point, you must re-embed the entire corpus.*

Vector từ hai model thuộc hai không gian khác nhau.  
*Vectors from two models belong to different spaces.*

Đổi model là một quyết định kiến trúc rất nặng.  
*Changing the model is a major architectural decision.*

Quyết định này nặng hơn lựa chọn index.  
*This decision is heavier than index selection.*

Hệ quả đầu tiên là chốt model sớm.  
*The first consequence is early model selection.*

Hệ quả tiếp theo là version hóa cột.  
*The next consequence is column versioning.*

Ví dụ cột mới là `embedding_v2`.  
*An example new column is `embedding_v2`.*

Bạn có thể backfill theo chiến lược blue-green.  
*You can backfill with a blue-green strategy.*

Bạn nên ưu tiên BYO-embedding — embedding của chính bạn.  
*You should prefer bring-your-own embeddings.*

Bạn nên ưu tiên các chuẩn mở.  
*You should prefer open standards.*

Các lựa chọn này giúp tránh lock-in.  
*These choices help avoid lock-in.*

In-DB embedding kiểu HeatWave rất tiện.  
*HeatWave-style in-database embedding is convenient.*

Cách này tạo lock-in chặt hơn.  
*This approach creates tighter lock-in.*

### 3.4. Hybrid: keyword và nghĩa bù nhau
*3.4. Hybrid: keywords and meaning complement each other*

Chủ đề kết nối trạm 8 với trạm 1.  
*The topic connects station 8 with station 1.*

FTS làm việc với lexeme — đơn vị từ đã chuẩn hóa.  
*FTS works with lexemes.*

FTS chính xác với thuật ngữ.  
*FTS is precise with terminology.*

FTS chính xác với mã.  
*FTS is precise with codes.*

FTS chính xác với tên riêng.  
*FTS is precise with proper names.*

FTS không hiểu ngữ nghĩa.  
*FTS does not understand semantics.*

"Ô tô" khác "xe hơi" về mặt chữ.  
*"Ô tô" differs from "xe hơi" on the surface.*

Vector hiểu nghĩa.  
*Vectors understand meaning.*

Vector có thể trượt thuật ngữ hiếm.  
*Vectors can miss rare terminology.*

Hybrid chứa cả hai phương pháp.  
*Hybrid search contains both methods.*

RRF fuse kết quả từ hai phương pháp.  
*RRF fuses results from both methods.*

Đây là kiến trúc retrieval mặc định năm 2026.  
*This is the default retrieval architecture in 2026.*

pgvector và FTS hỗ trợ kiến trúc này.  
*pgvector and FTS support this architecture.*

Mọi thành phần nằm trong một Postgres.  
*Every component stays in one Postgres instance.*

Đây là điểm hội tụ của hai nửa khóa học.  
*This is the convergence point of both course halves.*

Nửa đầu về keyword nằm trong bài FTS.  
*The keyword half appears in the FTS lesson.*

Nửa còn lại về semantic nằm trong các phần khác.  
*The semantic half appears in the other sections.*

### ✅ Self-check Phần 3
*✅ Part 3 self-check*

1. ANN đổi recall lấy tốc độ tại những trạm nào?  
*1. At which stations does ANN trade recall for speed?*

Bạn đo recall bằng cách nào?  
*How do you measure recall?*

2. Tại sao re-embedding nặng hơn chọn index?  
*2. Why is re-embedding heavier than index selection?*

3. Hybrid search giải quyết điểm yếu nào của FTS?  
*3. Which FTS weakness does hybrid search address?*

Hybrid search giải quyết điểm yếu nào của vector search?  
*Which vector-search weakness does hybrid search address?*

---

## Phần 4 — 🟣 STAFF LEVEL: Capstone — Thiết kế và build hệ hoàn chỉnh
*Part 4 — 🟣 STAFF LEVEL: Capstone — design and build a complete system*

### 4.1. System design tổng: "Semantic search + RAG cho một sản phẩm thật"
*4.1. Overall system design: "Semantic search plus RAG for a real product"*

> **Bài toán thiết kế**  
> *Design problem*
>
> Hệ thống phục vụ 20 triệu tài liệu.  
> *The system serves 20 million documents.*
>
> Hệ thống hỗ trợ nhiều ngôn ngữ.  
> *The system supports multiple languages.*
>
> Tiếng Việt nằm trong tập ngôn ngữ.  
> *Vietnamese belongs to the language set.*
>
> Hệ thống xử lý dữ liệu khách hàng nhạy cảm.  
> *The system handles sensitive customer data.*
>
> Mục tiêu p95 thấp hơn 150ms.  
> *The p95 target is below 150ms.*
>
> Dữ liệu được cập nhật hàng ngày.  
> *The data receives daily updates.*
>
> Hệ thống không có downtime — thời gian ngừng hoạt động.  
> *The system has no downtime.*
>
> Một team backend nhỏ vận hành hệ thống.  
> *A small backend team operates the system.*
>
> Team sử dụng PostgreSQL.  
> *The team uses PostgreSQL.*

Bài toán này gộp mọi quyết định của khóa.  
*This problem combines every course decision.*

Câu trả lời staff đi theo pipeline.  
*A staff-level answer follows the pipeline.*

1. **Clarify**  
   *Clarify*

   Bạn luôn làm rõ yêu cầu trước.  
   *You always clarify requirements first.*

   Hãy hỏi về QPS.  
   *Ask about QPS.*

   Hãy hỏi về recall target.  
   *Ask about the recall target.*

   Hãy hỏi về ràng buộc compliance — tuân thủ.  
   *Ask about compliance constraints.*

   Hãy hỏi về ngân sách.  
   *Ask about the budget.*

   Hãy hỏi về tần suất update.  
   *Ask about the update frequency.*

2. **[1 Embed]**  
   *[1 Embed]*

   Dữ liệu có tính nhạy cảm.  
   *The data is sensitive.*

   Dữ liệu có tính đa ngữ.  
   *The data is multilingual.*

   Lựa chọn đề xuất là self-host BGE-M3.  
   *The recommended choice is self-hosted BGE-M3.*

   Dữ liệu được giữ trong hệ thống nội bộ.  
   *The data stays inside the internal system.*

   BGE-M3 hỗ trợ tiếng Việt tốt.  
   *BGE-M3 supports Vietnamese well.*

   BYO giúp tránh lock-in.  
   *Bring-your-own embeddings help avoid lock-in.*

   Hãy chunk — chia nhỏ tài liệu dài.  
   *Chunk long documents.*

   Hãy xử lý batch trên GPU.  
   *Run batches on a GPU.*

   Hãy cache theo hash.  
   *Cache by hash.*

3. **[3 Platform]**  
   *[3 Platform]*

   Quy mô là 20 triệu tài liệu.  
   *The scale is 20 million documents.*

   Quy mô này thấp hơn ngưỡng dedicated.  
   *This scale is below the dedicated threshold.*

   Vector search là một feature.  
   *Vector search is a feature.*

   Hệ thống đã có Postgres.  
   *The system already has Postgres.*

   Lựa chọn đề xuất là pgvector self-host.  
   *The recommended choice is self-hosted pgvector.*

   Hệ thống không gửi dữ liệu đến API ngoài.  
   *The system sends no data to external APIs.*

   Privacy — quyền riêng tư yêu cầu lựa chọn này.  
   *Privacy requires this choice.*

4. **[4 Store]**  
   *[4 Store]*

   Hãy tạo bảng `docs`.  
   *Create the `docs` table.*

   Bảng có cột `tenant_id`.  
   *The table has a `tenant_id` column.*

   Bảng có cột `content`.  
   *The table has a `content` column.*

   Bảng có cột `lang`.  
   *The table has a `lang` column.*

   Bảng có cột `embedding vector(1024)`.  
   *The table has an `embedding vector(1024)` column.*

   Hãy partition — phân vùng theo tenant hoặc lang.  
   *Partition by tenant or language.*

   Hãy tạo B-tree cho metadata filter.  
   *Create a B-tree for metadata filters.*

5. **[5 Cài]**  
   *[5 Install]*

   Bạn có thể dùng Docker tag đã pin.  
   *You can use a pinned Docker tag.*

   Bạn có thể dùng gói hệ điều hành.  
   *You can use an operating-system package.*

   Hãy chạy `CREATE EXTENSION` qua migration.  
   *Run `CREATE EXTENSION` through a migration.*

   Hãy đồng bộ version trên replica.  
   *Synchronize the version on replicas.*

6. **[6 Nạp]**  
   *[6 Load]*

   Hãy nạp dữ liệu ban đầu bằng `COPY (binary)`.  
   *Load the initial data with `COPY (binary)`.*

   Bảng chưa có index tại thời điểm nạp.  
   *The table has no index during loading.*

   Pipeline phải idempotent — chạy lặp không gây sai lệch.  
   *The pipeline must be idempotent.*

   Temp table và upsert hỗ trợ idempotency.  
   *Temporary tables and upserts support idempotency.*

   Hãy cập nhật hàng ngày qua queue bất đồng bộ.  
   *Process daily updates through an asynchronous queue.*

7. **[2 Index]**  
   *[2 Index]*

   Hãy dùng HNSW với `vector_cosine_ops`.  
   *Use HNSW with `vector_cosine_ops`.*

   Hãy build index `CONCURRENTLY` sau khi nạp.  
   *Build the index `CONCURRENTLY` after loading.*

   Hãy đặt `maintenance_work_mem` ở mức cao.  
   *Set `maintenance_work_mem` to a high value.*

   Hãy dùng parallel build.  
   *Use a parallel build.*

   RAM có thể bị căng.  
   *RAM can become constrained.*

   Khi đó hãy cân nhắc quantization — lượng tử hóa.  
   *At that point, consider quantization.*

   Bạn cũng có thể cân nhắc DiskANN.  
   *You can also consider DiskANN.*

8. **[7 Query]**  
   *[7 Query]*

   Query dùng mẫu sau.  
   *The query uses the following pattern.*

   `WHERE <filter> AND embedding <=> :q < :thr ORDER BY embedding <=> :q LIMIT k`  
   *`WHERE <filter> AND embedding <=> :q < :thr ORDER BY embedding <=> :q LIMIT k`*

   Hãy tune `ef_search` theo p95.  
   *Tune `ef_search` against p95.*

   Hãy dùng `iterative_scan` chống overfiltering.  
   *Use `iterative_scan` against overfiltering.*

   Hãy hiệu chỉnh threshold trên golden set — tập chuẩn.  
   *Calibrate the threshold on a golden set.*

9. **[8 Hybrid]**  
   *[8 Hybrid]*

   Nhánh FTS dùng `tsvector` và GIN.  
   *The FTS branch uses `tsvector` and GIN.*

   Nhánh còn lại dùng vector.  
   *The other branch uses vectors.*

   RRF fuse hai nhánh.  
   *RRF fuses both branches.*

   Cách này cải thiện recall.  
   *This approach improves recall.*

   Với tiếng Việt, hãy cân nhắc cấu hình `simple` và unaccent.  
   *For Vietnamese, consider the `simple` configuration and unaccent.*

   Bạn cũng có thể dựa nhiều hơn vào nhánh vector.  
   *You can also rely more heavily on the vector branch.*

10. **Vận hành**  
    *Operations*

    Hãy monitor — giám sát recall@k.  
    *Monitor recall@k.*

    Hãy so sánh recall@k với exact search.  
    *Compare recall@k with exact search.*

    Hãy monitor p99.  
    *Monitor p99.*

    Hãy monitor WAL và replica lag.  
    *Monitor WAL and replica lag.*

    Hãy monitor drift — độ trôi của phân phối distance.  
    *Monitor distance-distribution drift.*

    Hãy dùng blue-green trong lần đổi model.  
    *Use blue-green deployment during model changes.*

    Hãy chuẩn bị kế hoạch re-embed có version.  
    *Prepare a versioned re-embedding plan.*

11. **Tự đánh giá**  
    *Self-assessment*

    Điểm nghẽn là embedding generation — quá trình tạo embedding.  
    *The bottleneck is embedding generation.*

    Insert không phải điểm nghẽn.  
    *Insertion is not the bottleneck.*

    Privacy ép hệ thống self-host.  
    *Privacy forces the system to self-host.*

    Benchmark không thay đổi ràng buộc này.  
    *Benchmarks do not change this constraint.*

    Quy mô 20 triệu phù hợp với pgvector.  
    *A scale of 20 million fits pgvector.*

    Tôi vẫn biết ngưỡng chuyển đổi.  
    *I still understand the transition threshold.*

    Hàng trăm triệu vector có thể vượt ngưỡng.  
    *Hundreds of millions of vectors can cross the threshold.*

    Triển khai đa vùng có thể vượt ngưỡng.  
    *Multi-region deployment can cross the threshold.*

    Khi đó tôi sẽ cân nhắc dedicated vector database.  
    *At that point, I will consider a dedicated vector database.*

    Câu trả lời thể hiện lý do của mọi lựa chọn.  
    *The answer shows the reason for every choice.*

    Câu trả lời thể hiện giới hạn của mọi lựa chọn.  
    *The answer shows the limits of every choice.*

### 4.2. Capstone Project — build để đưa vào portfolio
*4.2. Capstone Project — build it for your portfolio*

Video gợi ý chia sẻ lab trong repo công khai.  
*The video suggests sharing the lab in a public repository.*

Repo giúp showcase năng lực với nhà tuyển dụng.  
*The repository showcases your skills to employers.*

Đây là spec — đặc tả của một project trọn vẹn.  
*This is the specification for a complete project.*

Project này đáng để build.  
*This project is worth building.*

**Semantic + Hybrid Search Engine trên PostgreSQL**  
***Semantic + Hybrid Search Engine on PostgreSQL***

- **Mục tiêu**  
  ***Goal***

  Hệ thống tìm kiếm theo nghĩa trên một corpus thật.  
  *The system performs semantic search on a real corpus.*

  Corpus có thể là Wikipedia dump.  
  *The corpus can be a Wikipedia dump.*

  Corpus cũng có thể là tập FAQ hoặc sản phẩm.  
  *The corpus can also be an FAQ or product dataset.*

  Hệ thống có nhánh keyword.  
  *The system has a keyword branch.*

  Hệ thống có nhánh semantic.  
  *The system has a semantic branch.*

- **Stack**  
  ***Stack***

  Stack dùng PostgreSQL và pgvector.  
  *The stack uses PostgreSQL and pgvector.*

  PostgreSQL chạy trong Docker.  
  *PostgreSQL runs in Docker.*

  Embedding model được self-host.  
  *The embedding model is self-hosted.*

  Model có thể dùng sentence-transformers hoặc BGE-M3.  
  *The model can use sentence-transformers or BGE-M3.*

  API dùng FastAPI hoặc Express.  
  *The API uses FastAPI or Express.*

- **Các mảnh bắt buộc**  
  ***Required components***

  Các mảnh tương ứng đúng 8 trạm của khóa.  
  *The components match the eight course stations.*

  1. Pipeline thực hiện chunk tài liệu.  
  *1. The pipeline chunks documents.*

  Pipeline thực hiện embed theo batch.  
  *The pipeline embeds data in batches.*

  2. Schema có cột `vector(n)`.  
  *2. The schema has a `vector(n)` column.*

  Schema có metadata.  
  *The schema has metadata.*

  Schema có GIN cho FTS.  
  *The schema has a GIN index for FTS.*

  3. Hệ thống nạp dữ liệu bằng `COPY`.  
  *3. The system loads data with `COPY`.*

  Hệ thống nạp dữ liệu trước.  
  *The system loads the data first.*

  Hệ thống tạo index sau.  
  *The system builds the index later.*

  Pipeline có tính idempotent.  
  *The pipeline is idempotent.*

  4. Hệ thống có HNSW index.  
  *4. The system has an HNSW index.*

  Hệ thống tune `ef_search`.  
  *The system tunes `ef_search`.*

  5. Endpoint query hỗ trợ k-NN.  
  *5. The query endpoint supports k-NN.*

  Endpoint query hỗ trợ threshold.  
  *The query endpoint supports a threshold.*

  Endpoint query hỗ trợ filter.  
  *The query endpoint supports filters.*

  6. Endpoint hybrid dùng vector.  
  *6. The hybrid endpoint uses vectors.*

  Endpoint hybrid dùng FTS.  
  *The hybrid endpoint uses FTS.*

  RRF fuse hai kết quả.  
  *RRF fuses both results.*

  7. `EXPLAIN ANALYZE` chứng minh query dùng index.  
  *7. `EXPLAIN ANALYZE` confirms index usage.*

  Hệ thống đo recall@k trên một golden set nhỏ.  
  *The system measures recall@k on a small golden set.*

  8. README giải thích trade-off — sự đánh đổi.  
  *8. The README explains trade-offs.*

  README giải thích lý do chọn HNSW.  
  *The README explains the choice of HNSW.*

  README giải thích lý do chọn pgvector.  
  *The README explains the choice of pgvector.*

  README giải thích cách chọn threshold.  
  *The README explains threshold selection.*

- **Điểm gây ấn tượng nhà tuyển dụng**  
  ***Employer-impressing element***

  README trình bày trade-off và giới hạn.  
  *The README presents trade-offs and limitations.*

  README trình bày recall so với speed.  
  *The README presents recall versus speed.*

  README giải thích thời điểm không dùng pgvector.  
  *The README explains when pgvector is unsuitable.*

  Nội dung này thể hiện tư duy staff.  
  *This content demonstrates staff-level thinking.*

  Project không chỉ dừng ở mức chạy được.  
  *The project goes beyond merely working.*
