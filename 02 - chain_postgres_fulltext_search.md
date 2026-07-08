# PostgreSQL: Relational DB & Full-Text Search (tsvector / tsquery) — Chain gối đầu

> Phiên bản mốc: PostgreSQL hiện hành là **18** (17/16/15/14 vẫn hỗ trợ, **19 beta** ra 6/2026).
> FTS: **GIN** vẫn là index chuẩn; best practice 2026 là **generated tsvector column (STORED)** hoặc **functional GIN index** thay vì trigger; điểm hội tụ của khóa: FTS (keyword) + pgvector (semantic) = **hybrid search**.
> `websearch_to_tsquery` cần PG 11+; generated column STORED cần PG 12+.

---

## CHAIN — RDBMS refresher tới relationship

Database là một tập hợp dữ liệu được tổ chức, lưu số hóa để dùng lại => quản lý database là phần mềm **DBMS** (PostgreSQL/MySQL/Oracle) => DBMS lưu dữ liệu dưới dạng **file** trên đĩa nhưng trình bày cho bạn dưới dạng **table** => table xếp thành row (record) và column, mỗi hàng là một **entity** => trong mỗi hàng cần một cột định danh duy nhất = **primary key (PK)** (vd `cust_id`) => cột ở bảng khác trỏ tới PK đó gọi là **foreign key (FK)** => FK tạo ra **relationship** giữa các bảng (`cust_id` là PK ở `customers`, là FK ở `orders` → biết đơn thuộc khách nào)

## CHAIN — BLOB tới ORDBMS & extensibility

dữ liệu phức tạp (ảnh/video/file) lưu nhị phân ngay trong DB gọi là **BLOB** (Binary Large Object) => ngoài kiểu nhị phân, một số DB còn *mở rộng* được kiểu và tính năng => PostgreSQL là **ORDBMS** (Object-Relational DBMS) — mở rộng được => mở rộng được nghĩa là thêm được data type, function, **index method**, extension mới => nhờ extensibility, `pgvector` cắm thêm kiểu `vector` cho semantic search => và FTS cắm thêm kiểu `tsvector`/`tsquery` + index GIN/GiST cho keyword search => vì *cắm thêm vào DB sẵn có* nên không phải mua thêm hệ thống riêng — đó là luận điểm chiến lược để chọn Postgres

## CHAIN — Tiền dùng kiểu nào

với tiền phải dùng `numeric` hoặc `money` => KHÔNG dùng `float`/`double` vì số thực dấu phẩy động làm tròn sai => làm tròn sai thì kế toán lệch xu (lỗi kinh điển junior hay mắc)

## CHAIN — Vì sao LIKE gãy, dẫn tới lexeme

tìm văn bản bằng `WHERE body LIKE '%running%'` gặp 3 vấn đề => vấn đề 1: `%...%` (wildcard 2 đầu) không dùng được B-tree index → **sequential scan**, chậm và tỉ lệ với *tổng số hàng* => vấn đề 2: không hiểu biến thể từ — "running"/"ran"/"runs" cùng ý nhưng `LIKE` chỉ khớp đúng chuỗi "running", bỏ sót "ran" => vấn đề 3: **match nhầm** — `'%run%'` khớp cả "b**run**ch", "p**run**e" (kết quả rác) => cả 3 vì `LIKE` so *ký tự* => FTS sinh ra để so **lexeme** thay vì so ký tự

## CHAIN — Inverted index (mục lục cuối sách)

FTS so lexeme bằng cách xây một **inverted index** => inverted index giống **mục lục cuối sách**: "từ → những trang chứa từ đó" => tra mục lục thì nhảy thẳng tới đúng trang, không đọc cả cuốn => tương tự FTS tra "lexeme → document chứa nó" rồi nhảy thẳng tới hàng khớp => vì vậy tốc độ phụ thuộc *số hàng khớp*, không phụ thuộc *tổng số hàng* => đó là lý do FTS nhanh còn `LIKE '%...%'` là `O(N)` (chậm khi bảng to)

## CHAIN — Lexeme, stop-word, tsvector, tsquery, @@

**lexeme** là một từ đã được chuẩn hóa (stemming) gộp mọi biến thể về gốc: "running"/"ran"/"runs" → `run` => trước khi ra lexeme, từ quá phổ biến (the/is/a) bị loại vì không mang nghĩa phân biệt = **stop-word** => sau khi bỏ stop-word + stem, danh sách lexeme phân biệt đã sắp xếp kèm vị trí được lưu trong kiểu **`tsvector`** (tài liệu đã tiền xử lý) => phía truy vấn, các lexeme nối bằng toán tử logic `&`(AND) `|`(OR) `!`(NOT) `<->`(FOLLOWED BY) tạo thành **`tsquery`** => hỏi "tsvector này có thỏa tsquery kia không" dùng toán tử **`@@`** (trả về true/false)

## CHAIN — Chạy tay tsvector (4 bước)

cho câu "The quick brown foxes are running", Postgres xử lý qua 4 bước => bước 1 **tokenize**: The/quick/brown/foxes/are/running => bước 2 **bỏ stop-word**: the, are bị loại → còn quick/brown/foxes/running => bước 3 **stemming** về lexeme: foxes→fox, running→run => bước 4 **sắp xếp + gắn vị trí token gốc + loại trùng** → tsvector `'brown':3 'fox':4 'quick':2 'run':6` => số là vị trí token gốc trong câu, dùng cho phrase/ranking => nhờ vậy tìm lexeme `run` vẫn match dù câu gốc viết "running", không hề có chữ "run" nguyên văn (điều `LIKE` không làm được)

## CHAIN — Hello world @@ & đối xứng chuẩn hóa

`to_tsvector('english', text)` (2 tham số: config ngôn ngữ + text) biến văn bản thành tsvector => `to_tsquery('english','running')` biến truy vấn thành tsquery và **cũng bị stem** về `'run'` => `@@` ghép tsvector với tsquery, trả true/false => điểm khắc cốt: cả *tài liệu* lẫn *truy vấn* đều đi qua CÙNG bộ chuẩn hóa (stem + bỏ stop-word) => nhờ cùng bộ chuẩn hóa nên chúng "gặp nhau" ở cấp lexeme => `@@` là trái tim của FTS (như `<->` là trái tim của vector search)

## CHAIN — Pipeline nội bộ: text → tsvector

một **text search configuration** (vd `'english'`) gồm 2 bộ phận => bộ phận 1 **parser**: tách văn bản thô thành **token** và gán *loại token* (từ/số/email/URL/host) => parser parse "khôn": email và URL được nhận diện thành token riêng chứ không vỡ vụn => bộ phận 2 **dictionaries**: xử lý từng token → ra lexeme hoặc *loại bỏ* => dictionaries gồm stop-word dict (loại the/is/a), stemmer Snowball (running→run), synonym/thesaurus (tùy chọn) => chọn config sai = kết quả sai => config `'simple'` KHÔNG stem, không bỏ stop-word (chỉ lowercase) → dùng cho mã sản phẩm/username => config `'english'` stem theo tiếng Anh; còn tiếng Việt stemming hạn chế → dùng `'simple'` + `unaccent` hoặc chuyển sang vector search

## CHAIN — GIN index & coalesce

cột được tìm thường xuyên nên đánh **GIN index** (không index thì FTS rơi về seq scan) => cách khuyến nghị 1 — **functional GIN**: `CREATE INDEX ... USING GIN (to_tsvector('english', body))`, không tốn thêm cột => nhưng query PHẢI khớp đúng biểu thức đã index (cùng config, cùng 2-arg form) => cách 2 — **generated `tsvector` column STORED** + GIN: `ALTER TABLE ... ADD COLUMN ... GENERATED ALWAYS AS (...) STORED`, Postgres tự đồng bộ mỗi khi cột nguồn đổi => generated column (PG 12+) thay cho **trigger** sync tay ngày xưa (dễ quên/lệch) => khi ghép nhiều cột phải bọc `coalesce(col,'')` => vì nếu một cột NULL mà không coalesce thì TOÀN BỘ tsvector thành NULL → hàng đó biến mất khỏi search

## CHAIN — Ranking & setweight

toán tử `@@` chỉ cho true/false nên muốn "liên quan nhất lên đầu" cần **ranking** => `ts_rank` chấm điểm dựa trên **tần suất** lexeme khớp => biến thể `ts_rank_cd` (**cover density**) thưởng thêm khi các từ khớp nằm gần nhau → hợp tìm cụm => muốn "trường nào quan trọng hơn" thì dùng **`setweight`** gán trọng số A/B/C/D (A cao nhất) cho từng nguồn text => vd `setweight` title→'A', body→'B', tags→'C' → khớp ở title rank cao hơn khớp ở body => đây là cách chỉnh relevance mà KHÔNG cần viết code ứng dụng

## CHAIN — ts_rank chi tiết & I/O bottleneck

`ts_rank(weights, vector, query, normalization)` chấm theo tần suất + trọng số => tham số `weights` default `{0.1, 0.2, 0.4, 1.0}` tương ứng D/C/B/A => tham số `normalization` là bitmask chia điểm theo độ dài tài liệu (tránh doc dài luôn thắng chỉ vì chứa nhiều từ) => vd cờ `32` = `rank/(rank+1)` đưa điểm về khoảng dễ so sánh => nhưng ranking phải MỞ tsvector của từng document khớp → tốn I/O, dễ thành bottleneck => giải: filter cứng trước để giảm số hàng khớp rồi mới rank; hoặc chỉ rank top-N

## CHAIN — Xử lý input user an toàn

đưa raw input của user vào `to_tsquery` là mời gọi lỗi (khoảng trắng/dấu chấm → syntax error → request 500) => vì vậy với ô search hướng người dùng dùng **`websearch_to_tsquery`** => `websearch_to_tsquery` nuốt được cú pháp lộn xộn kiểu Google: dấu ngoặc kép (cụm), dấu trừ (loại trừ), OR (PG 11+) => nếu chỉ cần "có tất cả các từ" đơn giản thì `plainto_tsquery` (tự AND các từ) cũng được => còn `to_tsquery` chỉ dùng khi BẠN kiểm soát query và cần toán tử chính xác

## CHAIN — GIN internals & vì sao nhanh

**GIN** = Generalized Inverted Index, lưu ánh xạ lexeme → danh sách document chứa nó (**posting list**) => query "database" → tra thẳng posting list của lexeme `databas` → lấy về row id → chỉ đọc đúng những hàng khớp => query complexity phụ thuộc **số document khớp** (kích thước posting list), không phụ thuộc tổng `N` => đó là lý do FTS giữ tốc độ khi bảng phình to, còn `LIKE '%...%'` là `O(N)` => đổi lại GIN build tốn hơn B-tree và **nhạy `maintenance_work_mem`** (tăng RAM này → build nhanh hơn nhiều) => và GIN update chậm khi ghi nhiều, có **`fastupdate`** buffer để hoãn cập nhật

## CHAIN — Prefix, phrase, fuzzy (3 tầng search)

**prefix matching** `:*` cho gõ nửa từ: `'postgr:*'` khớp postgres/postgresql => **phrase/proximity** dùng `<->` (FOLLOWED BY) hoặc `<N>` (cách N từ) để yêu cầu thứ tự => nhưng FTS KHÔNG sửa lỗi chính tả: "postgrsql" không thành lexeme `postgresql` => chống typo dùng extension **`pg_trgm`** (trigram): `CREATE INDEX ... USING GIN (title gin_trgm_ops)`, toán tử `%` và hàm `similarity()` => gộp lại thành pattern production **3 tầng: FTS (lexeme, chính xác) + trigram (typo) + vector (ngữ nghĩa)**

## CHAIN — Edge cases cần thủ sẵn

edge case 1 — NULL nuốt tsvector: luôn `coalesce` khi ghép cột => edge case 2 — ngôn ngữ: stemmer `'english'` sai bét với tiếng Việt/Nhật/Trung; đa ngôn ngữ 1 bảng → lưu config theo cột và index `GIN(to_tsvector(config_col, body))` => edge case 3 — **stop-word làm mất cụm**: "the who" (tên ban nhạc) bị bỏ "the" → khó tìm; trường đặc biệt dùng config `'simple'` => edge case 4 — `CREATE INDEX` khóa ghi: trên bảng lớn production dùng **`CREATE INDEX CONCURRENTLY`** để không chặn write => edge case 5 — `@@` không dùng index nếu vế trái không phải biểu thức đã index → luôn `EXPLAIN ANALYZE` xác nhận có "Bitmap Index Scan"

## CHAIN — Scale FTS & 3 bottleneck

GIN inverted → query time ~ số hàng khớp nên FTS chạy sub-second trên hàng triệu doc NẾU filter/query đủ chọn lọc => nhờ vậy Postgres FTS thực sự thay được Elasticsearch cho app **small–medium** => **bottleneck 1 — ranking I/O**: `ts_rank` phải mở tsvector từng doc khớp; query phổ biến ("the","data") khớp cả triệu hàng → I/O bound => giải bottleneck 1: filter cứng (metadata/thời gian) trước, persist tsvector (generated column), rank chỉ top-N => **bottleneck 2 — write amplification**: GIN update chậm; bảng ghi nặng → cân nhắc `fastupdate`/GiST/tách bảng search riêng => **bottleneck 3 — index build khóa write**: dùng `CREATE INDEX CONCURRENTLY` và tăng `maintenance_work_mem` => scale-out: read replicas cho search + partition (tenant/thời gian) + GIN per-partition để planner prune sớm

## CHAIN — Hybrid search: FTS + pgvector

FTS và vector **bổ khuyết** nhau => FTS (keyword/lexeme) chính xác với thuật ngữ/mã/tên riêng, giải thích được, nhanh, nhưng MÙ ngữ nghĩa ("xe hơi" ≠ "ô tô") => vector (semantic) hiểu đồng nghĩa/diễn giải nhưng có thể trượt thuật ngữ hiếm/mã sản phẩm và khó giải thích => vì bổ khuyết nên chạy CẢ HAI rồi hợp nhất điểm = **hybrid search** => cách hợp nhất phổ biến là **Reciprocal Rank Fusion (RRF)**: lấy top-K mỗi nhánh, cộng `SUM(1/(60 + r))` => hằng số `60` trong RRF để làm mượt (giảm ảnh hưởng của thứ hạng rất cao) => điểm vàng: pgvector + FTS làm hybrid ngay TRONG một Postgres, không cần Pinecone + Elasticsearch song song => hybrid là kiến trúc retrieval mặc định cho search/RAG hiện đại 2026

## CHAIN — Cost, monitoring, failure, reindex

FTS gần như miễn phí về hạ tầng (tận dụng Postgres) → tiết kiệm cả tiền lẫn on-call surface so với cụm ES riêng => vận hành thì monitor p95/p99 search, tỉ lệ query rơi về seq scan, độ trễ ranking, kích thước GIN, write latency (`pg_stat_statements`, `EXPLAIN ANALYZE`) => failure mode hay gặp: (1) config/biểu thức lệch → index bị bỏ, latency tăng âm thầm => (2) NULL nuốt tsvector → doc "mất tích"; (3) truy vấn stop-word/từ siêu phổ biến → rank cả triệu hàng, treo => (4) đổi text search config/ngôn ngữ → phải reindex; (5) GIN `fastupdate` buffer phình khi ghi nhiều mà chưa vacuum => reindex an toàn dùng **`REINDEX INDEX CONCURRENTLY`** (PG 12+) để không khóa

## CHAIN — Tổ chức & config roadmap

nói với PM/sếp bằng **risk & cost** không jargon: thêm search mạnh mà không dựng thêm hệ thống, đội đã biết vận hành Postgres => quyết định **text search config** (ngôn ngữ/thesaurus/weight) ảnh hưởng relevance TOÀN sản phẩm => đổi config = reindex toàn bảng (tốn thời gian/khóa) → phải version hóa và lên kế hoạch như một migration => và giữ FTS trong Postgres nghĩa là backend team hiện tại tự lo được, khỏi tuyển/đào tạo riêng cho một search stack

## CHAIN — Khi nào vượt Postgres sang Elasticsearch

Postgres FTS thắng ở operational simplicity + join SQL + ACID cho small–medium => nhưng khi search trở thành **workload/sản phẩm chính** thì cân nhắc Elasticsearch/OpenSearch => hoặc khi cần **BM25/analyzer cao cấp** (relevance mạnh hơn `ts_rank`) => hoặc khi quy mô rất lớn / analytics text (ES vượt trội) => khi đó có thể giữ pgvector cho nhánh semantic, chỉ chuyển nhánh keyword sang ES => biết giới hạn lựa chọn của mình = dấu hiệu staff

---

## BẢNG — Data types PostgreSQL (nhớ theo nhóm)

- **Numeric:** `integer`, `bigint`, `real`, `double precision`, `numeric` (chính xác cao cho tiền).
- **Character:** `text`, `varchar(n)`, `char(n)`.
- **Boolean:** `boolean`.
- **Date/Time:** `date`, `time`, `timestamp`, `timestamptz`, `interval`.
- **Array:** cột chứa mảng, vd `text[]`.
- **Enum:** tập giá trị cố định, vd `mood AS ENUM ('sad','ok','happy')`.
- **Đặc thù:** `money`, geometric (`point`/`polygon`), network (`inet`/`cidr`), `uuid`, `json`/`jsonb`.
- **Mở rộng qua extension:** `vector` (pgvector), `tsvector`/`tsquery` (FTS).

## BẢNG — RDBMS phổ biến

IBM Db2, Oracle Database, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, MariaDB, SAP HANA, Amazon RDS (có cả proprietary lẫn open-source, on-prem lẫn cloud). Ví dụ "relational" từ bài gốc: retailer 4 bảng `customers`, `orders`, `orderline_items`, `items`.

## BẢNG — Họ hàm tạo tsquery

| Hàm | Input | Dùng khi |
|---|---|---|
| `to_tsquery` | phải có toán tử: `'run & fast'` | bạn kiểm soát query, cần chính xác; **không** đưa input thô của user |
| `plainto_tsquery` | text thô, tự AND: `'run fast'` → `'run' & 'fast'` | input đơn giản, muốn "có tất cả các từ" |
| `phraseto_tsquery` | text thô → cụm kề: `'full text search'` → `'full'<->'text'<->'search'` | tìm **cụm từ** đúng thứ tự |
| `websearch_to_tsquery` | cú pháp Google: `'postgres -mysql "full text"'` | **input trực tiếp từ ô search user** — an toàn (PG 11+) |

## BẢNG — GIN vs GiST

| Tiêu chí | GIN | GiST |
|---|---|---|
| Bản chất | inverted index (chính xác) | signature tree (lossy → recheck heap) |
| Query speed | **Nhanh hơn** cho FTS | Chậm hơn (recheck) |
| Build/insert | Chậm hơn, tốn RAM | **Nhanh hơn**, nhẹ hơn |
| Update thường xuyên | Kém hơn | **Tốt hơn** |
| Kích thước index | To hơn | **Nhỏ hơn** (tùy `siglen`) |
| Nhạy `maintenance_work_mem` | Có | Không |
| Chọn khi | **read-heavy, text ít đổi** (đa số case) | write-heavy, index cần nhỏ, proximity với data động |

Chốt: mặc định chọn **GIN**; chỉ nghiêng GiST khi ghi rất nhiều hoặc giới hạn dung lượng.

## BẢNG — Generated column vs functional index vs trigger

| Cách | Ưu | Nhược |
|---|---|---|
| **Functional GIN** `GIN(to_tsvector('english', body))` | không nhân đôi data, không cần cột, không sync | query phải khớp đúng biểu thức; ghép nhiều cột rườm |
| **Generated column STORED** + GIN | query gọn, tự sync, ghép nhiều cột dễ, gắn weight tiện | tốn thêm dung lượng (lưu tsvector) |
| **Trigger tự viết** | linh hoạt tối đa | dễ quên/lệch, nhiều code, lỗi thầm lặng — **né trừ khi bắt buộc** |

Quy tắc: weight nhiều trường → generated column; 1 trường muốn gọn → functional index; hầu như không cần trigger tay nữa.

## BẢNG — FTS vs RegEx/LIKE vs Vector search

| | LIKE / RegEx | FTS (tsvector) | Vector (pgvector) |
|---|---|---|---|
| Khớp theo | pattern ký tự | **lexeme** (từ chuẩn hóa) | **ngữ nghĩa** (embedding) |
| "run" khớp "running"? | không | **có** (stemming) | có (nếu nghĩa gần) |
| "áo phao" khớp "đồ giữ ấm"? | không | **không** (khác lexeme) | **có** (hiểu nghĩa) |
| Index | kém (`%...%` = seq scan) | **GIN** (inverted) | **HNSW/IVFFlat** (ANN) |
| Chính xác | tuyệt đối | tuyệt đối theo lexeme | approximate |
| Cần model AI? | không | không | **có** (embedding model) |

## BẢNG — PostgreSQL FTS vs Elasticsearch/OpenSearch

| Tiêu chí | PostgreSQL FTS | Elasticsearch/OpenSearch |
|---|---|---|
| Hạ tầng | **không thêm gì** (DB sẵn có) | cụm riêng, JVM, vận hành, đồng bộ |
| Transaction/consistency | **ACID, real-time** | eventually consistent, phải ETL/sync |
| Join business data | **1 câu SQL** | 2 hệ, glue code |
| Relevance nâng cao (BM25, analyzer) | cơ bản (`ts_rank`) | **mạnh hơn** |
| Quy mô cực lớn / analytics text | khá | **vượt trội** |
| Chi phí + độ phức tạp vận hành | **thấp** | cao |

## BẢNG — Code thuộc lòng

**(a) FTS tối thiểu + index:**
```sql
CREATE INDEX docs_fts ON docs USING GIN (to_tsvector('english', body));
SELECT title FROM docs
WHERE to_tsvector('english', body) @@ websearch_to_tsquery('english', 'postgres index');
```

**(b) Generated column có weight + rank:**
```sql
ALTER TABLE docs ADD COLUMN sv tsvector GENERATED ALWAYS AS (
  setweight(to_tsvector('english', coalesce(title,'')), 'A') ||
  setweight(to_tsvector('english', coalesce(body,'')),  'B')
) STORED;
CREATE INDEX ON docs USING GIN (sv);

SELECT title, ts_rank(sv, q) r
FROM docs, websearch_to_tsquery('english', :input) q
WHERE sv @@ q ORDER BY r DESC LIMIT 10;
```

**(c) Hybrid RRF (FTS + vector fuse):**
```sql
WITH kw AS (
  SELECT id, row_number() OVER (ORDER BY ts_rank(sv, q) DESC) AS r
  FROM docs, websearch_to_tsquery('english', 'wireless headphones') q
  WHERE sv @@ q LIMIT 50
),
sem AS (
  SELECT id, row_number() OVER (ORDER BY embedding <=> :query_vec) AS r
  FROM docs ORDER BY embedding <=> :query_vec LIMIT 50
)
SELECT id, SUM(1.0 / (60 + r)) AS rrf_score
FROM (SELECT id, r FROM kw UNION ALL SELECT id, r FROM sem) t
GROUP BY id ORDER BY rrf_score DESC LIMIT 10;
```

## BẢNG — Khung system design (20M docs, keyword + typo + đồng nghĩa, filter, p95<150ms, đa ngôn ngữ)

1. **Clarify:** QPS? tần suất update? recall hay precision ưu tiên? ngân sách? cần giải thích kết quả không?
2. **Data model:** bảng `docs` = metadata + `search_vector` (generated, `setweight` title>body>tags) + `embedding vector(...)`. Partition theo `lang`/thời gian.
3. **Index:** GIN trên `search_vector`; `pg_trgm` GIN trên title (typo); HNSW trên `embedding` (semantic). Tất cả trong cùng Postgres.
4. **Query = hybrid:** nhánh FTS (`websearch_to_tsquery`) + nhánh trigram (typo) + nhánh vector (đồng nghĩa) → fuse RRF; filter cứng category/thời gian **trước** khi rank.
5. **Latency:** filter chọn lọc trước ranking; persist tsvector; rank top-N; read replicas; đo p95 thật.
6. **Đa ngôn ngữ:** config theo cột `GIN(to_tsvector(lang_col, body))`; embedding đa ngữ cho nhánh vector.
7. **Ops:** monitor tỉ lệ seq-scan fallback, p99, index size, write latency; reindex concurrently khi đổi config; blue-green khi đổi embedding model.
8. **Khi nào vượt Postgres:** search là workload chính + BM25/analyzer cao cấp + quy mô rất lớn → ES/OpenSearch cho nhánh keyword, giữ pgvector cho semantic.

## BẢNG — Mental models

- **"Mục lục cuối sách"** → inverted index / GIN trong 1 câu.
- **"Run = running = ran"** → lexeme / stemming.
- **"Keyword vs meaning"** → FTS bắt đúng từ, vector hiểu nghĩa → hybrid.
- **"Một database, ba tầng search"** → lexeme (FTS) + typo (trigram) + semantic (vector).
- **"Search là feature hay là product?"** → kim chỉ nam Postgres FTS vs Elasticsearch.

---

## TỪ KHÓA MỒI

- RDBMS refresher → **"định danh duy nhất"**
- BLOB & ORDBMS → **"mở rộng được"**
- Tiền dùng kiểu nào → **"lệch xu"**
- Vì sao LIKE gãy → **"so ký tự vs so lexeme"**
- Inverted index → **"mục lục cuối sách"**
- Lexeme/stop-word/tsvector/tsquery/@@ → **"từ đã chuẩn hóa"**
- Chạy tay tsvector → **"'brown':3 'fox':4"**
- Hello world @@ → **"cùng bộ chuẩn hóa"**
- Pipeline nội bộ → **"parser + dictionaries"**
- GIN index & coalesce → **"NULL nuốt tsvector"**
- Ranking & setweight → **"A/B/C/D"**
- ts_rank chi tiết → **"chia điểm theo độ dài"**
- Input user an toàn → **"websearch_to_tsquery"**
- GIN internals → **"posting list"**
- Prefix/phrase/fuzzy → **"pg_trgm typo"**
- Edge cases → **"CREATE INDEX CONCURRENTLY"**
- Scale & 3 bottleneck → **"ranking I/O"**
- Hybrid search → **"Reciprocal Rank Fusion"**
- Cost/monitoring/failure → **"REINDEX CONCURRENTLY"**
- Tổ chức & config → **"đổi config = reindex"**
- Khi nào vượt Postgres → **"search là product"**

---

## ĐÃ PHỦ

**Refresher RDBMS:** database, DBMS (lưu file, trình bày table), table/row/entity/column, primary key, foreign key, relationship, BLOB, ví dụ retailer 4 bảng, danh sách RDBMS phổ biến, PostgreSQL = ORDBMS + extensibility (data type/function/index method/extension), luận điểm "không mua thêm hệ thống".

**Data types:** đủ 8 nhóm (numeric/character/boolean/date-time/array/enum/đặc thù/extension); quy tắc tiền dùng numeric/money không dùng float/double.

**Vì sao LIKE gãy:** 3 vấn đề (seq scan không index nổi `%...%` = O(N); không hiểu biến thể từ; match nhầm brunch/prune); FTS so lexeme không so ký tự.

**Khái niệm FTS:** FTS định nghĩa, lexeme (stemming), stop-word, tsvector (danh sách lexeme sắp xếp + vị trí), tsquery (toán tử `& | ! <->`), `@@`, RegEx vs FTS, inverted index (analogy mục lục), tốc độ ~ số hàng khớp.

**Chạy tay & hello world:** 4 bước tokenize→bỏ stop-word→stem→sắp xếp+vị trí; kết quả `'brown':3 'fox':4 'quick':2 'run':6`; `to_tsvector`/`to_tsquery`/`@@` 2-arg; query cũng bị stem; đối xứng chuẩn hóa; ví dụ "hands-on data scientist" với `<->`.

**Pipeline nội bộ:** text search config = parser (token + loại token, email/URL không vỡ) + dictionaries (stop-word/Snowball stemmer/thesaurus); config `simple` vs `english`; tiếng Việt `simple`+unaccent hoặc vector.

**Họ hàm tsquery:** `to_tsquery` / `plainto_tsquery` / `phraseto_tsquery` / `websearch_to_tsquery` (bảng); quy tắc input user an toàn.

**Index:** functional GIN vs generated column STORED (PG 12+) vs trigger (bảng so sánh); coalesce chống NULL; GIN internals (posting list, complexity ~ số hàng khớp, maintenance_work_mem, fastupdate); GIN vs GiST (bảng, lossy/recheck/siglen); `CREATE INDEX CONCURRENTLY`.

**Ranking:** `@@` chỉ true/false; `ts_rank` (tần suất) vs `ts_rank_cd` (cover density); `setweight` A/B/C/D; `ts_rank` 4 tham số, weights default `{0.1,0.2,0.4,1.0}` D/C/B/A, normalization bitmask (cờ 32 = rank/(rank+1)); ranking I/O bottleneck.

**Nâng cao:** prefix `:*`, phrase `<->`/`<N>`, fuzzy `pg_trgm` (gin_trgm_ops, `%`, `similarity()`); pattern 3 tầng FTS+trigram+vector; edge cases (NULL, ngôn ngữ + config theo cột, stop-word mất cụm "the who", CREATE INDEX blocking, `@@` cần biểu thức đã index + EXPLAIN ANALYZE Bitmap Index Scan); bảng FTS vs RegEx vs Vector.

**Staff/scale:** GIN sub-second nếu filter chọn lọc; thay ES cho small-medium; 3 bottleneck (ranking I/O, write amplification, index build) + cách giải; scale-out (read replicas, partition + GIN per-partition prune, FDW); bảng FTS vs Elasticsearch; hybrid search (FTS+vector bổ khuyết, RRF `SUM(1/(60+r))`, hằng số 60, trong 1 Postgres, mặc định RAG 2026); cost/monitoring/5 failure modes/REINDEX CONCURRENTLY; tổ chức (risk & cost, config = reindex migration, team topology); khung system design 8 bước; khi nào vượt sang ES + BM25.

**Version:** PostgreSQL 18 (19 beta 6/2026); GIN chuẩn; generated column STORED / functional GIN thay trigger; `websearch_to_tsquery` PG 11+, generated column PG 12+.

**Học tiếp (bài nêu, không đào sâu):** RRF vs weighted fusion, reranking models, BM25 + extension ParadeDB/`pg_search` mang BM25 vào Postgres, text search configuration/dictionary tùy biến tiếng Việt.
