# PostgreSQL: Relational DB & Full-Text Search (tsvector / tsquery) — Giáo trình Basic → Staff

> **Nguồn gốc:** Video *"Refresh your knowledge of relational databases with a focus on PostgreSQL"* — Course 3, IBM Vector Database Fundamentals. Bài gốc có nội dung thật: ôn RDBMS/DBMS, primary/foreign key, BLOB, các data types của PostgreSQL, và **full-text search với tsvector/tsquery/lexemes**. Tôi giảng bám sát rồi đào sâu. Chỗ vượt ra ngoài bài gốc đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.
>
> **Cập nhật tới 2026:** PostgreSQL bản hiện hành là **18** (17/16/15/14 vẫn được hỗ trợ, **19 beta** ra tháng 6/2026). Với FTS: **GIN** vẫn là index chuẩn; best practice 2026 là dùng **generated tsvector column (STORED)** hoặc **functional GIN index** thay vì tự sync bằng trigger; và điểm quan trọng nhất cho khóa này — FTS (keyword) **kết hợp** pgvector (semantic) tạo thành **hybrid search**.
>
> **Liên hệ bài trước:** Giáo trình này là "người anh em" của giáo trình pgvector. FTS ở đây là *keyword/lexeme search*; vector search bên kia là *semantic search*. Phần Staff sẽ ghép cả hai lại.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

**Bài này dạy gì:** Hai tầng kiến thức. Tầng 1 (refresher) — cách relational database tổ chức dữ liệu: DBMS, tables, primary/foreign key, các kiểu dữ liệu, và vì sao PostgreSQL đặc biệt (nó là **ORDBMS** — mở rộng được). Tầng 2 (trọng tâm) — **full-text search (FTS)**: cách Postgres biến văn bản thô thành `tsvector` (danh sách lexeme đã chuẩn hóa) và match với `tsquery` để tìm tài liệu theo *nghĩa của từ* thay vì khớp ký tự thô như `LIKE`.

**Vấn đề nó giải quyết:** `WHERE col LIKE '%run%'` vừa **chậm** (quét tuần tự, không index nổi tiền tố kiểu `%...%`), vừa **ngớ ngẩn** (không hiểu "running", "ran", "runs" là cùng một từ; lại match nhầm "brunch"). FTS giải cả hai: chuẩn hóa từ (stemming) + bỏ stop-word + index kiểu inverted → tìm nhanh và đúng ngữ nghĩa từ vựng.

**Học xong bạn sẽ làm được:**

- Giải thích DBMS, table, primary key vs foreign key, BLOB, và vì sao PostgreSQL là ORDBMS mở rộng được.
- Liệt kê và chọn đúng data type của PostgreSQL cho từng nhu cầu.
- Hiểu lexeme, `tsvector`, `tsquery`, toán tử `@@`; viết FTS chạy được.
- Đánh GIN index đúng cách, rank kết quả bằng `ts_rank`/`setweight`.
- Phân tích khi nào FTS đủ (thay Elasticsearch), khi nào cần vector/hybrid search, và trả lời câu hỏi system design về search.

**Mạch basic → staff:**

- 🟢 **Basic:** RDBMS/DBMS/PK-FK/BLOB refresher → vì sao `LIKE` gãy → khái niệm lexeme → `to_tsvector`/`to_tsquery`/`@@` hello world.
- 🟡 **Intermediate:** Pipeline nội bộ (parser → dictionary → lexeme), họ hàm query (`to_tsquery` vs `plainto_` vs `phraseto_` vs `websearch_`), GIN index, ranking + weights, lỗi thường gặp.
- 🔴 **Advanced:** Inverted index internals, GIN vs GiST + Big-O, generated column vs functional index vs trigger, `ts_rank` vs `ts_rank_cd`, custom configuration/dictionary, `pg_trgm` chống typo, edge cases.
- 🟣 **Staff:** Scale hàng chục triệu docs, FTS vs Elasticsearch, **hybrid search FTS + pgvector**, cost/ops/monitoring, org impact, câu hỏi system design.
- 🎯 **Cheatsheet:** Keywords, core concepts, code thuộc lòng, câu hỏi phỏng vấn + one-liner.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

### 1.1. Refresher: relational database tổ chức dữ liệu thế nào

- **Database:** một tập hợp dữ liệu được tổ chức, lưu trữ số hóa để dùng lại.
- **DBMS (Database Management System):** phần mềm quản lý database — cho phép truy cập, sửa, cập nhật nhanh. Ví dụ: PostgreSQL, MySQL, Oracle. DBMS thường lưu dữ liệu dưới dạng **file** trên đĩa, nhưng trình bày cho bạn dưới dạng **table**.
- **Table (bảng):** dữ liệu xếp thành hàng (row/record) và cột (column). Mỗi hàng là một **entity** (thực thể) — ví dụ một khách hàng.
- **Primary key (PK):** cột (hoặc nhóm cột) **định danh duy nhất** mỗi hàng. Ví dụ `cust_id` trong bảng `customers`.
- **Foreign key (FK):** cột trong bảng này *trỏ tới* primary key của bảng khác, tạo **quan hệ (relationship)** giữa các bảng. Ví dụ `cust_id` xuất hiện trong bảng `orders` (ở đó nó là FK) để biết đơn hàng thuộc khách nào.
- **BLOB (Binary Large Object):** cách lưu dữ liệu phức tạp (ảnh, video, file) dưới dạng nhị phân (0 và 1) ngay trong database.

**Ví dụ (từ bài gốc):** một retailer có 4 bảng `customers`, `orders`, `orderline_items`, `items`. `cust_id` là PK ở `customers` và là FK ở `orders` → nối "khách nào đặt đơn nào". Đây là bản chất "relational".

### 1.2. Các RDBMS phổ biến & PostgreSQL đặc biệt ở đâu

Các RDBMS quen thuộc: **IBM Db2, Oracle Database, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, MariaDB, SAP HANA, Amazon RDS** (có cả proprietary lẫn open-source, cả on-prem lẫn cloud).

PostgreSQL là **ORDBMS (Object-Relational DBMS)** — mở rộng mô hình quan hệ bằng tính năng hướng đối tượng. Điểm mấu chốt: **PostgreSQL mở rộng được** — thêm data type mới, function mới, **index method** mới, extension mới. Chính nhờ đặc tính này mà:

- `pgvector` (extension) thêm được kiểu `vector` cho vector search (giáo trình trước).
- FTS thêm được kiểu `tsvector`/`tsquery` và index GIN/GiST cho keyword search (bài này).

> **[MỞ RỘNG NGOÀI BÀI GỐC]** "Extensibility" không phải chi tiết vặt — nó là *lý do chiến lược* để chọn Postgres. Bạn không phải mua thêm hệ thống riêng (vector DB, search engine); bạn *cắm thêm khả năng* vào database sẵn có. Đây là luận điểm sẽ quay lại ở Phần 4.

### 1.3. Data types của PostgreSQL (nhớ nhóm, không học vẹt)

- **Numeric:** `integer`, `bigint`, `real`, `double precision`, `numeric` (chính xác cao cho tiền).
- **Character:** `text`, `varchar(n)`, `char(n)`.
- **Boolean:** `boolean`.
- **Date/Time:** `date`, `time`, `timestamp`, `timestamptz`, `interval`.
- **Array:** cột chứa mảng, vd `text[]`.
- **Enumerated (`enum`):** tập giá trị cố định, vd `mood AS ENUM ('sad','ok','happy')`.
- **Đặc thù:** `money` (tiền), geometric (`point`, `polygon`...), network (`inet`, `cidr`), `uuid`, `json`/`jsonb`.
- **Mở rộng qua extension:** `vector` (pgvector), và **`tsvector`/`tsquery`** (full-text search) — tâm điểm bài này.

> **Lưu ý staff:** với tiền, dùng `numeric`/`money`, **không** dùng `float`/`double` — số thực dấu phẩy động làm tròn sai, kế toán sẽ lệch xu. Đây là lỗi kinh điển junior hay mắc.

### 1.4. Vấn đề thực tế: vì sao `LIKE` không đủ cho tìm kiếm văn bản

Bạn có bảng `articles(body text)`. Người dùng gõ tìm "running".

```sql
SELECT title FROM articles WHERE body LIKE '%running%';
```

Ba vấn đề:

1. **Chậm & không scale:** `%...%` (wildcard hai đầu) không dùng được B-tree index → **sequential scan**. Bảng càng to càng chậm; thời gian tỉ lệ với *tổng số hàng*.
2. **Không hiểu biến thể từ:** "running", "ran", "runs" là cùng ý nhưng `LIKE '%running%'` chỉ khớp đúng chuỗi "running". Bỏ sót "ran".
3. **Match nhầm:** `LIKE '%run%'` khớp cả "b**run**ch", "p**run**e". Kết quả rác.

FTS sinh ra để giải cả 3. Cốt lõi: nó không so *ký tự*, nó so **lexeme**.

### 1.5. Analogy: mục lục cuối sách (inverted index)

Cuối một cuốn sách kỹ thuật có phần **Index** (mục lục tra cứu): "database → trang 12, 45, 88". Bạn muốn biết chỗ nào nói về "database" → tra thẳng mục lục, nhảy tới đúng trang, **không** đọc cả cuốn.

FTS làm y hệt: nó xây một **inverted index** — bảng tra "từ (lexeme) → những tài liệu chứa từ đó". Tìm "database" = tra bảng, nhảy thẳng tới các hàng khớp. Tốc độ phụ thuộc *số hàng khớp*, không phụ thuộc *tổng số hàng*. Đó là lý do FTS nhanh còn `LIKE '%...%'` chậm.

### 1.6. Định nghĩa thuật ngữ (lần đầu xuất hiện)

- **Full-text search (FTS):** tìm trong tập tài liệu ngôn ngữ tự nhiên những tài liệu khớp nhất với một truy vấn.
- **Lexeme:** một từ đã được **chuẩn hóa** để gộp mọi biến thể về cùng một dạng gốc (stemming). "running", "ran", "runs" → lexeme `run`. Đây là "đơn vị tìm kiếm" của FTS.
- **Stop-word:** từ quá phổ biến, không mang nghĩa phân biệt ("the", "is", "a") → bị **loại bỏ** khỏi tsvector.
- **`tsvector`:** kiểu dữ liệu lưu **danh sách lexeme phân biệt, đã sắp xếp**, kèm vị trí. Là "tài liệu đã tiền xử lý sẵn để tìm".
- **`tsquery`:** kiểu dữ liệu biểu diễn **truy vấn** — các lexeme nối bằng toán tử logic `&` (AND), `|` (OR), `!` (NOT), `<->` (FOLLOWED BY / kề nhau).
- **`@@`:** toán tử **match** — "tsvector này có thỏa tsquery kia không?" Trả về `true`/`false`.
- **RegEx vs FTS:** RegEx khớp theo *pattern ký tự*; FTS khớp theo *lexeme* đã chuẩn hóa (khác biệt bài gốc nhấn mạnh).

### 1.7. Ví dụ chạy tay — "thấy" tsvector được sinh ra

Cho câu: **"The quick brown foxes are running"**. Postgres xử lý qua 4 bước để ra `tsvector`:

1. **Tokenize (tách token):** `The`, `quick`, `brown`, `foxes`, `are`, `running`.
2. **Bỏ stop-word:** `the`, `are` bị loại → còn `quick`, `brown`, `foxes`, `running`.
3. **Stemming (chuẩn hóa về lexeme):** `foxes → fox`, `running → run`, `quick → quick`, `brown → brown`.
4. **Sắp xếp + gắn vị trí, loại trùng:** kết quả tsvector:

```
'brown':3 'fox':4 'quick':2 'run':6
```

(Số là *vị trí token gốc* trong câu — dùng cho phrase/ranking.) Giờ nếu người dùng tìm lexeme `run`, tsvector này match — dù câu gốc viết "running", không hề có chữ "run" nguyên văn. **Đó chính là điều `LIKE` không làm được.**

### 1.8. Code "hello world" đơn giản nhất

```sql
-- 1) Biến văn bản thành tsvector (2 tham số: config ngôn ngữ + text)
SELECT to_tsvector('english', 'The quick brown foxes are running');
-- => 'brown':3 'fox':4 'quick':2 'run':6   (đúng như chạy tay)

-- 2) Biến truy vấn thành tsquery
SELECT to_tsquery('english', 'running');
-- => 'run'                                  (query cũng bị stem về lexeme!)

-- 3) Match bằng @@  -> true/false
SELECT to_tsvector('english', 'The quick brown foxes are running')
    @@ to_tsquery('english', 'running');
-- => t   (true: tài liệu chứa lexeme 'run')

-- 4) Áp lên bảng thật: tìm bài báo chứa "run" (mọi biến thể)
SELECT title
FROM articles
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'running');
```

**Điểm khắc cốt:** cả *tài liệu* lẫn *truy vấn* đều đi qua cùng bộ chuẩn hóa (stem + bỏ stop-word), nên chúng "gặp nhau" ở cấp lexeme. Toán tử `@@` là trái tim của FTS — giống như `<->` là trái tim của vector search vậy.

### 1.9. Ánh xạ tsvector/tsquery về ví dụ trong bài gốc

Bài gốc nêu: query cụm "hands-on data scientist", kiểm tra "hands-on" và "data" theo sau bởi "scientist" có trong tài liệu không → trả `t` (true). Diễn giải: tài liệu được lưu dạng `tsvector`, còn cụm tìm kiếm được viết thành `tsquery` (có thể dùng toán tử `<->` để yêu cầu "data" đứng ngay trước "scientist"), rồi `@@` cho biết khớp hay không.

### ✅ Self-check Phần 1

1. Nêu 3 lý do `WHERE body LIKE '%running%'` kém cho tìm kiếm văn bản.
2. Lexeme là gì? "cats", "cat", "catty" (nếu cùng gốc) sẽ về lexeme nào?
3. `to_tsvector('english','The cats are sleeping')` — dự đoán các lexeme (không cần vị trí). Vì sao "the"/"are" biến mất?

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

Giả định đã nắm: lexeme, tsvector, tsquery, `@@`.

### 2.1. Pipeline nội bộ: text → tsvector đi qua những gì

Một **text search configuration** (vd `'english'`) gồm 2 bộ phận:

1. **Parser:** tách văn bản thô thành **token** và gán *loại token* (từ thường, số, email, URL, host...). Postgres parse "khôn": email, URL được nhận diện thành token riêng chứ không vỡ vụn.
2. **Dictionaries:** xử lý từng token → ra **lexeme** hoặc *loại bỏ*. Gồm:
   - **Stop-word dictionary:** loại "the", "is", "a"...
   - **Stemmer (Snowball):** chuẩn hóa "running" → "run".
   - **Synonym / thesaurus** (tùy chọn): "supernovae stars" → "sn".

Chọn config sai = kết quả sai. Config `'simple'` **không** stem, không bỏ stop-word (giữ nguyên token, chỉ lowercase) — dùng cho mã sản phẩm, username. Config `'english'` stem theo tiếng Anh. Với tiếng Việt, stemming hạn chế → thường dùng `'simple'` + unaccent, hoặc chuyển hẳn sang vector search (Phần 4).

### 2.2. Họ hàm tạo tsquery — chọn đúng cái cho đúng ngữ cảnh

| Hàm | Cú pháp input | Dùng khi |
|---|---|---|
| `to_tsquery` | phải có toán tử: `'run & fast'` | bạn kiểm soát query, cần chính xác; **không** đưa input thô của user vào (dễ lỗi cú pháp) |
| `plainto_tsquery` | text thô, tự AND các từ: `'run fast'` → `'run' & 'fast'` | input đơn giản của user, muốn "có tất cả các từ" |
| `phraseto_tsquery` | text thô → cụm kề nhau: `'full text search'` → `'full' <-> 'text' <-> 'search'` | tìm **cụm từ** đúng thứ tự |
| `websearch_to_tsquery` | cú pháp kiểu Google: `'postgres -mysql "full text"'` | **input trực tiếp từ ô search của user** — an toàn, quen thuộc (PG 11+) |

> **Mẹo staff:** đưa raw input của user vào `to_tsquery` là mời gọi lỗi (user gõ khoảng trắng, dấu chấm → syntax error). Với ô tìm kiếm hướng người dùng, dùng **`websearch_to_tsquery`** — nó nuốt được cú pháp lộn xộn, hỗ trợ dấu ngoặc kép (cụm), dấu trừ (loại trừ), OR.

### 2.3. Đánh index cho FTS — GIN là chuẩn

Không index, FTS vẫn chạy nhưng rơi về sequential scan (chậm). Với cột được tìm thường xuyên, đánh **GIN index**.

**Cách 1 — Functional GIN index (khuyến nghị 2026, không tốn thêm cột):**

```sql
CREATE INDEX articles_fts_idx
ON articles
USING GIN (to_tsvector('english', body));

-- Query PHẢI khớp đúng biểu thức đã index (cùng config, cùng 2-arg form)
SELECT title FROM articles
WHERE to_tsvector('english', body) @@ websearch_to_tsquery('english', 'postgres index');
```

**Cách 2 — Generated `tsvector` column + GIN (khi tìm rất thường xuyên, muốn tách nguồn từ nhiều cột):**

```sql
ALTER TABLE articles
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    to_tsvector('english', coalesce(title,'') || ' ' || coalesce(body,''))
) STORED;                          -- tự cập nhật mỗi khi title/body đổi

CREATE INDEX articles_sv_idx ON articles USING GIN (search_vector);

SELECT title FROM articles
WHERE search_vector @@ websearch_to_tsquery('english', 'postgres index');
```

> **[MỞ RỘNG]** Ngày xưa người ta duy trì cột tsvector bằng **trigger** thủ công — dễ quên, dễ lệch. Từ khi có **generated column STORED** (PG 12+), đây là cách gọn và an toàn nhất; Postgres tự đồng bộ. Functional index thì khỏi cần cột luôn. Đừng viết trigger sync tay nữa trừ khi có lý do đặc biệt.

Lưu ý `coalesce(...,'')`: nếu một cột NULL mà không bọc coalesce, **toàn bộ** tsvector thành NULL → hàng đó biến mất khỏi search. Lỗi rất hay gặp.

### 2.4. Ranking — sắp kết quả theo độ liên quan

Match `@@` chỉ cho true/false. Muốn "kết quả liên quan nhất lên đầu" cần **ranking**:

```sql
SELECT title,
       ts_rank(search_vector, query) AS rank
FROM articles,
     websearch_to_tsquery('english', 'postgres performance') AS query
WHERE search_vector @@ query
ORDER BY rank DESC
LIMIT 10;
```

- `ts_rank`: điểm dựa trên **tần suất** lexeme khớp.
- `ts_rank_cd` (cover density): thưởng thêm khi các từ khớp **nằm gần nhau** → hợp cho tìm cụm.

**`setweight` — điều khiển "trường nào quan trọng hơn":** gán trọng số A/B/C/D (A cao nhất) cho từng nguồn văn bản:

```sql
GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title,'')),       'A') ||   -- title quan trọng nhất
    setweight(to_tsvector('english', coalesce(body,'')),        'B') ||
    setweight(to_tsvector('english', coalesce(array_to_string(tags,' '),'')), 'C')
) STORED
```

→ Khớp ở **title** rank cao hơn khớp ở body. Đây là cách chỉnh relevance **không cần viết code ứng dụng**.

### 2.5. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp

**Lỗi 1 — Index không được dùng vì config/biểu thức không khớp.** Index tạo bằng `to_tsvector('english', body)` nhưng query viết `to_tsvector(body)` (thiếu config) → **không dùng được index**, rơi về seq scan. Quy tắc: biểu thức trong `WHERE` phải **trùng khít** biểu thức lúc `CREATE INDEX` (cùng config name, cùng 2-arg form). Kiểm bằng `EXPLAIN ANALYZE` xem có `Bitmap Index Scan` không.

**Lỗi 2 — Quên coalesce → NULL nuốt cả tsvector.** Như 2.3: một cột NULL làm hàng biến mất khỏi kết quả. Luôn `coalesce(col,'')` khi ghép nhiều cột.

**Lỗi 3 — Nhét raw user input vào `to_tsquery`.** User gõ `"c# tips"` hay `machine learning` (có khoảng trắng) → `to_tsquery` báo syntax error, request 500. Dùng `websearch_to_tsquery`/`plainto_tsquery` cho input người dùng.

### ✅ Self-check Phần 2

1. Ô search của user, bạn nên dùng hàm tạo tsquery nào và vì sao KHÔNG dùng `to_tsquery`?
2. Bạn muốn khớp ở `title` được ưu tiên hơn ở `body`. Dùng cơ chế gì?
3. Index `GIN(to_tsvector('english', body))` nhưng query dùng `to_tsvector(body)`. Điều gì xảy ra và cách phát hiện?

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

### 3.1. Inverted index & vì sao GIN nhanh

**GIN (Generalized Inverted Index)** lưu ánh xạ **lexeme → danh sách document chứa nó** (giống mục lục cuối sách). Query "database" → tra thẳng posting list của lexeme `databas`, lấy về các row id → chỉ đọc đúng những hàng khớp.

- **Độ phức tạp query:** phụ thuộc **số document khớp** (kích thước posting list), *không* phụ thuộc tổng số hàng `N`. Đây là lý do FTS giữ tốc độ khi bảng phình to, còn `LIKE '%...%'` là `O(N)`.
- **Build/update:** GIN build tốn hơn B-tree; **nhạy với `maintenance_work_mem`** (tăng RAM này → build nhanh hơn nhiều). GIN update chậm hơn khi ghi nhiều (có `fastupdate` buffer để hoãn).

### 3.2. GIN vs GiST — bảng so sánh (câu hỏi phỏng vấn kinh điển)

| Tiêu chí | **GIN** | **GiST** |
|---|---|---|
| Bản chất | inverted index (chính xác) | signature tree (lossy, có false positive → recheck) |
| Query speed | **Nhanh hơn** cho FTS | Chậm hơn (phải recheck heap) |
| Build/insert | Chậm hơn, tốn RAM | **Nhanh hơn**, nhẹ hơn |
| Update thường xuyên | Kém hơn | **Tốt hơn** |
| Kích thước index | To hơn | **Nhỏ hơn** (tùy `siglen`) |
| Nhạy `maintenance_work_mem` | Có | Không |
| Chọn khi | **read-heavy, text ít đổi** (đa số case) | write-heavy, index cần nhỏ, cần proximity/phrase với dữ liệu động |

**Chốt:** mặc định chọn **GIN**. Chỉ nghiêng GiST khi ghi rất nhiều hoặc bị giới hạn dung lượng index.

### 3.3. Generated column vs functional index vs trigger — chọn thế nào

| Cách | Ưu | Nhược |
|---|---|---|
| **Functional GIN index** `GIN(to_tsvector('english', body))` | không nhân đôi dữ liệu, không cần cột, không sync | query phải khớp đúng biểu thức; ghép nhiều cột hơi rườm |
| **Generated column STORED** + GIN | query gọn (`search_vector @@ q`), tự sync, ghép nhiều cột dễ, gắn weight tiện | tốn thêm dung lượng (lưu tsvector) |
| **Trigger tự viết** | linh hoạt tối đa | dễ quên/lệch, nhiều code, lỗi thầm lặng — **né trừ khi bắt buộc** |

Quy tắc thực chiến: cần **weight nhiều trường** → generated column; chỉ tìm 1 trường, muốn gọn → functional index; hầu như không bao giờ cần trigger tay nữa.

### 3.4. ts_rank vs ts_rank_cd & tinh chỉnh relevance

- `ts_rank(weights, vector, query, normalization)` — theo tần suất + trọng số A/B/C/D. `weights` default `{0.1, 0.2, 0.4, 1.0}` cho D/C/B/A.
- `ts_rank_cd` — **cover density**: thưởng khi các query term nằm gần nhau → tốt cho tìm cụm.
- **Normalization flag** (bitmask) để chia điểm theo độ dài tài liệu (tránh doc dài luôn thắng chỉ vì chứa nhiều từ). Vd cờ `32` = `rank/(rank+1)` đưa điểm về khoảng dễ so sánh.
- **Cảnh báo hiệu năng:** ranking phải mở tsvector của **từng document khớp** → tốn I/O, có thể là bottleneck. Tối ưu: filter cứng trước (giảm số hàng khớp) rồi mới rank; hoặc chỉ rank top-N.

### 3.5. Prefix, phrase, và fuzzy (typo)

**Prefix matching** (`:*`) — gõ nửa từ:
```sql
SELECT to_tsquery('english', 'postgr:*');   -- khớp postgres, postgresql...
```

**Phrase / proximity** (`<->` FOLLOWED BY, `<N>` cách N từ):
```sql
SELECT to_tsvector('english','data science is fun')
    @@ phraseto_tsquery('english','data science');   -- 'data' <-> 'science'
```

**Fuzzy / chống lỗi chính tả — `pg_trgm` (trigram):** FTS *không* sửa typo ("postgrsql" ≠ lexeme `postgresql`). Kết hợp extension `pg_trgm` để bắt gần đúng:
```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX articles_title_trgm ON articles USING GIN (title gin_trgm_ops);

SELECT title, similarity(title, 'postgrsql') AS sim
FROM articles
WHERE title % 'postgrsql'          -- % : đủ giống theo trigram
ORDER BY sim DESC;
```

> **[MỞ RỘNG]** Pattern production thường gặp: **FTS (lexeme, chính xác) + trigram (typo) + vector (ngữ nghĩa)** — ba tầng bổ trợ. FTS bắt đúng từ; trigram cứu lỗi gõ; vector hiểu nghĩa gần. Phần 4 sẽ ghép FTS + vector thành hybrid.

### 3.6. Edge cases cần thủ sẵn

- **NULL nuốt tsvector** (đã nêu) — luôn `coalesce`.
- **Ngôn ngữ:** stemmer `'english'` sai bét với tiếng Việt/Nhật/Trung. Tiếng Việt: cân nhắc `'simple'` + `unaccent` để bỏ dấu, hoặc chuyển sang vector search. Đa ngôn ngữ trong 1 bảng → lưu config theo cột và index `GIN(to_tsvector(config_col, body))`.
- **Stop-word làm mất cụm:** phrase "the who" (tên ban nhạc) → "the" bị bỏ → khó tìm. Đôi khi phải config `'simple'` cho trường đặc biệt.
- **Build index blocking:** `CREATE INDEX` khóa ghi. Trên bảng lớn production, dùng **`CREATE INDEX CONCURRENTLY`** để không chặn write.
- **`@@` không dùng index nếu vế trái không phải biểu thức đã index** — luôn `EXPLAIN ANALYZE` để xác nhận `Bitmap Index Scan`.

### 3.7. So sánh cốt lõi: FTS vs RegEx/LIKE vs Vector search

| | LIKE / RegEx | FTS (tsvector) | Vector search (pgvector) |
|---|---|---|---|
| Khớp theo | pattern ký tự | **lexeme** (từ chuẩn hóa) | **ngữ nghĩa** (embedding) |
| "run" khớp "running"? | không (trừ pattern tay) | **có** (stemming) | có (nếu nghĩa gần) |
| "áo phao" khớp "đồ giữ ấm"? | không | **không** (khác lexeme) | **có** (hiểu nghĩa) |
| Index | kém (`%...%` = seq scan) | **GIN** (inverted) | **HNSW/IVFFlat** (ANN) |
| Chính xác | tuyệt đối | tuyệt đối theo lexeme | approximate |
| Cần model AI? | không | không | **có** (embedding model) |

**Bài học:** FTS mạnh khi user gõ *đúng từ khóa* (dù khác biến thể). Nó **không** hiểu đồng nghĩa ngữ nghĩa ("xe hơi" vs "ô tô" trừ khi bạn cấu hình thesaurus). Chỗ đó là đất của vector search → dẫn tới hybrid.

### ✅ Self-check Phần 3

1. Vì sao GIN giữ tốc độ khi bảng lớn còn `LIKE '%x%'` thì không? Query complexity mỗi cái ~ bao nhiêu?
2. Ghi rất nhiều, index cần nhỏ, cần proximity → GIN hay GiST?
3. User hay gõ sai chính tả tên sản phẩm. FTS thuần giải được không? Bổ sung gì?

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

### 4.1. Scale FTS: từ nghìn tới hàng chục triệu documents

**Câu hỏi staff không phải "dùng GIN hay GiST" mà là "chỗ nào gãy khi lớn".**

- **Điểm mạnh cấu trúc:** vì GIN là inverted index, query time ~ số hàng khớp, không phải tổng N → FTS chạy sub-second trên hàng triệu doc *nếu* filter/query đủ chọn lọc. Postgres FTS thực sự thay được Elasticsearch cho **small–medium** apps.
- **Bottleneck 1 — ranking I/O:** `ts_rank` phải mở tsvector từng doc khớp. Query phổ biến ("the", "data") khớp cả triệu hàng → rank triệu tsvector = I/O bound. **Giải:** filter cứng (metadata, thời gian) trước để thu hẹp; persist tsvector (generated column) để khỏi tính lại; rank chỉ top-N.
- **Bottleneck 2 — write amplification:** GIN update chậm; bảng ghi nặng + FTS → cân nhắc `fastupdate`, hoặc GiST, hoặc tách bảng search riêng.
- **Bottleneck 3 — index build:** GIN build khóa write. Dùng `CREATE INDEX CONCURRENTLY`; tăng `maintenance_work_mem`.

**Scale-out:** read replicas cho search (read-heavy); **partition** bảng lớn (theo tenant/thời gian) + GIN per-partition → planner prune sớm. Ranking dùng thông tin *local* nên còn phân tán được qua Foreign Data Wrapper.

### 4.2. Quyết định kiến trúc: FTS (Postgres) vs Elasticsearch/OpenSearch

| Tiêu chí | PostgreSQL FTS | Elasticsearch/OpenSearch |
|---|---|---|
| Hạ tầng | **không thêm gì** (dùng DB sẵn có) | cụm riêng, JVM, vận hành, đồng bộ dữ liệu |
| Transaction/consistency | **ACID, real-time** | eventually consistent, phải ETL/sync |
| Join với business data | **1 câu SQL** | 2 hệ, glue code |
| Relevance nâng cao (BM25, analyzer phong phú) | cơ bản (`ts_rank`) | **mạnh hơn** |
| Quy mô cực lớn / analytics text | khá | **vượt trội** |
| Chi phí + độ phức tạp vận hành | **thấp** | cao |

> **One-liner staff:** *"Với small–medium apps, Postgres FTS xóa nhu cầu chạy thêm Elasticsearch — cùng một database, ACID, join thẳng SQL, không đồng bộ, không cụm riêng để on-call."* Chọn ES khi search là *sản phẩm chính*, cần relevance/analyzer cao cấp, hoặc quy mô rất lớn.

### 4.3. [MỞ RỘNG — ĐIỂM VÀNG] Hybrid Search: FTS + pgvector

Đây là chỗ khóa học này (vector database) hội tụ. **FTS và vector bổ khuyết nhau:**

- **FTS (keyword/lexeme):** chính xác với thuật ngữ, mã, tên riêng; giải thích được ("khớp vì chứa đúng từ"); nhanh; nhưng **mù ngữ nghĩa** ("xe hơi" ≠ "ô tô").
- **Vector (semantic):** hiểu đồng nghĩa/diễn giải; nhưng có thể trượt thuật ngữ hiếm/mã sản phẩm và khó giải thích.

**Hybrid** chạy cả hai rồi hợp nhất điểm — thường bằng **Reciprocal Rank Fusion (RRF)** hoặc weighted sum:

```sql
-- Ý tưởng: lấy top-K từ FTS và top-K từ vector, rồi fuse bằng RRF
WITH kw AS (   -- nhánh keyword (FTS)
  SELECT id, row_number() OVER (ORDER BY ts_rank(search_vector, q) DESC) AS r
  FROM docs, websearch_to_tsquery('english', 'wireless headphones') q
  WHERE search_vector @@ q
  LIMIT 50
),
sem AS (       -- nhánh semantic (pgvector)
  SELECT id, row_number() OVER (ORDER BY embedding <=> :query_vec) AS r
  FROM docs
  ORDER BY embedding <=> :query_vec
  LIMIT 50
)
SELECT id, SUM(1.0 / (60 + r)) AS rrf_score   -- RRF: 60 là hằng số làm mượt
FROM (SELECT id, r FROM kw UNION ALL SELECT id, r FROM sem) t
GROUP BY id
ORDER BY rrf_score DESC
LIMIT 10;
```

**Vì sao staff phải biết:** hybrid là kiến trúc retrieval mặc định cho search/RAG hiện đại 2026, và **pgvector + FTS làm được ngay trong một Postgres** — không cần Pinecone + Elasticsearch song song. Đây là luận điểm "extensibility của Postgres" (Phần 1) trả cổ tức.

### 4.4. Cost, ops, monitoring, failure modes

- **Cost:** FTS gần như **miễn phí về hạ tầng** (tận dụng Postgres). So với vận hành cụm ES riêng → tiết kiệm lớn cả tiền lẫn on-call surface.
- **Monitoring:** theo dõi p95/p99 của search query; tỉ lệ query rơi về seq scan (index không dùng); độ trễ ranking; kích thước GIN index; write latency khi GIN update. Dùng `pg_stat_statements`, `EXPLAIN ANALYZE`.
- **Failure modes:** (1) config/biểu thức lệch → index bị bỏ, latency tăng âm thầm; (2) NULL nuốt tsvector → doc "mất tích" khỏi search; (3) truy vấn stop-word/từ siêu phổ biến → rank cả triệu hàng, treo; (4) đổi text search config/ngôn ngữ → phải reindex; (5) GIN `fastupdate` buffer phình khi ghi nhiều mà chưa vacuum.
- **Reindex an toàn:** `REINDEX INDEX CONCURRENTLY` (PG 12+) để không khóa.

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder

- **Nói với PM/sếp:** "Ta thêm tìm kiếm mạnh mà *không dựng thêm hệ thống* — dùng chính database hiện có. Đội ngũ đã biết vận hành Postgres, nên rủi ro và chi phí thấp. Nếu sau này cần relevance cực cao hoặc quy mô rất lớn, ta cân nhắc Elasticsearch — nhưng đó là bài toán khi đã thành công." → framing bằng **rủi ro & chi phí**, không bằng jargon.
- **Roadmap:** quyết định *text search config* (ngôn ngữ, thesaurus, weight) ảnh hưởng relevance toàn sản phẩm; đổi config = reindex toàn bảng (tốn thời gian/khóa) → phải version hóa và lên kế hoạch như một migration.
- **Team topology:** FTS trong Postgres nghĩa là backend team hiện tại tự lo được, khỏi tuyển/đào tạo riêng cho một search stack — luận điểm tổ chức mạnh, giống lập luận pgvector ở giáo trình trước.

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff

> **"Thiết kế search cho một trang tin/e-commerce 20 triệu bài viết/sản phẩm: hỗ trợ keyword + gõ sai chính tả + hiểu đồng nghĩa, filter theo category/thời gian, p95 < 150ms, đa ngôn ngữ."**

**Khung trả lời staff (nói to trade-off):**

1. **Clarify:** QPS? tần suất update nội dung? recall/precision ưu tiên cái nào? ngân sách? cần giải thích được kết quả không?
2. **Data model:** một bảng `docs` chứa metadata + `search_vector` (generated, `setweight` title>body>tags) + `embedding vector(...)`. Partition theo `lang`/thời gian.
3. **Index:** GIN trên `search_vector`; `pg_trgm` GIN trên title (typo); HNSW trên `embedding` (semantic). Tất cả trong cùng Postgres.
4. **Query = hybrid:** nhánh FTS (`websearch_to_tsquery`, an toàn với input user) + nhánh trigram (typo) + nhánh vector (đồng nghĩa) → fuse bằng RRF; filter cứng category/thời gian **trước** để thu hẹp trước khi rank.
5. **Latency:** filter chọn lọc trước ranking; persist tsvector; rank top-N; read replicas cho search traffic; đo p95 thật.
6. **Đa ngôn ngữ:** lưu config theo cột, index `GIN(to_tsvector(lang_col, body))`; embedding đa ngữ cho nhánh vector.
7. **Ops:** monitor tỉ lệ seq-scan fallback, p99, index size, write latency; reindex concurrently khi đổi config; blue-green khi đổi embedding model.
8. **Khi nào vượt Postgres:** nếu search trở thành workload chính + cần BM25/analyzer cao cấp + quy mô rất lớn → cân nhắc Elasticsearch/OpenSearch cho nhánh keyword, giữ pgvector cho semantic. **Biết giới hạn lựa chọn của mình = dấu hiệu staff.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

### 5.1. Keywords bắt buộc nhớ

- **RDBMS / DBMS** — hệ quản trị database quan hệ / phần mềm quản lý database.
- **Primary key / Foreign key** — định danh duy nhất một hàng / trỏ tới PK bảng khác để tạo quan hệ.
- **BLOB** — Binary Large Object, lưu ảnh/video/file dạng nhị phân.
- **ORDBMS** — object-relational; PostgreSQL mở rộng được (data type, function, index, extension).
- **Full-text search (FTS)** — tìm tài liệu ngôn ngữ tự nhiên theo lexeme, không theo ký tự.
- **Lexeme** — từ đã chuẩn hóa (stem), gộp mọi biến thể ("running"→"run").
- **Stop-word** — từ phổ biến vô nghĩa phân biệt, bị loại ("the","is","a").
- **`tsvector`** — danh sách lexeme phân biệt, đã sắp xếp, kèm vị trí (tài liệu tiền xử lý).
- **`tsquery`** — truy vấn dạng lexeme + toán tử `& | ! <->`.
- **`@@`** — toán tử match tsvector với tsquery.
- **`to_tsvector` / `to_tsquery` / `plainto_` / `phraseto_` / `websearch_to_tsquery`** — họ hàm tạo vector/query.
- **`setweight` / `ts_rank` / `ts_rank_cd`** — gán trọng số A-D / xếp hạng theo tần suất / theo cover density.
- **GIN / GiST** — inverted index (chuẩn cho FTS) / signature tree (build nhanh, index nhỏ).
- **`pg_trgm`** — extension trigram cho fuzzy/typo search.
- **Hybrid search / RRF** — kết hợp FTS + vector; hợp nhất thứ hạng bằng Reciprocal Rank Fusion.

### 5.2. Core concepts — nếu chỉ nhớ vài điều

1. FTS khớp theo **lexeme** (từ đã stem + bỏ stop-word), nên "run" tìm được "running" — điều `LIKE` không làm được.
2. `tsvector @@ tsquery` là trái tim FTS; cả tài liệu lẫn query đều đi qua cùng bộ chuẩn hóa.
3. **GIN inverted index** làm query time ~ số hàng khớp (không phải tổng N) → nhanh khi bảng lớn.
4. Best practice 2026: **generated tsvector column STORED** hoặc **functional GIN index**; đừng viết trigger sync tay.
5. Input user → **`websearch_to_tsquery`** (an toàn), đừng nhét thẳng vào `to_tsquery`.
6. `setweight` (A/B/C/D) + `ts_rank` điều khiển relevance mà không cần code ứng dụng.
7. Luôn `coalesce(col,'')` khi ghép cột — NULL nuốt cả tsvector.
8. FTS mù ngữ nghĩa ("ô tô" ≠ "xe hơi"); vector search bù chỗ đó → **hybrid = FTS + pgvector**.
9. Postgres FTS thay được Elasticsearch cho small–medium; chọn ES khi search là sản phẩm chính / quy mô rất lớn.
10. Postgres **extensibility** là luận điểm chiến lược: FTS + vector + fuzzy đều trong một DB, không thêm hệ thống.

### 5.3. Ideas / mental models

- **"Mục lục cuối sách":** giải thích inverted index/GIN trong 1 câu.
- **"Run = running = ran":** giải thích lexeme/stemming.
- **"Keyword vs meaning":** FTS bắt đúng từ, vector hiểu nghĩa → hybrid.
- **"Một database, ba tầng search":** lexeme (FTS) + typo (trigram) + semantic (vector).
- **"Search là feature hay là product?":** kim chỉ nam Postgres FTS vs Elasticsearch.

### 5.4. Code cần thuộc lòng

**(a) FTS tối thiểu + index (interviewer hay bảo viết):**
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

**(c) Chạy tay tsvector (chứng tỏ hiểu bản chất):**
```
'The quick brown foxes are running'
 → bỏ stop-word (the, are) → stem (foxes→fox, running→run)
 → tsvector: 'brown':3 'fox':4 'quick':2 'run':6
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

1. **"Khác nhau FTS và `LIKE`?"** → `LIKE '%x%'` khớp ký tự, seq scan `O(N)`, không hiểu biến thể từ, match nhầm ("run" trong "brunch"). FTS khớp lexeme, dùng GIN inverted (nhanh, không phụ thuộc N), stem + bỏ stop-word.

2. **"tsvector và tsquery là gì, `@@` làm gì?"** → tsvector = tài liệu đã tiền xử lý thành danh sách lexeme sắp xếp + vị trí; tsquery = truy vấn dạng lexeme + toán tử; `@@` kiểm tra tsvector có thỏa tsquery không.

3. **[TRADE-OFF] "GIN hay GiST?"** → GIN: chính xác, query nhanh, build chậm + tốn RAM, index to → default cho read-heavy. GiST: lossy (recheck), build nhanh, index nhỏ, update tốt → chọn khi write-heavy/giới hạn dung lượng.

4. **[BẪY] "Tại sao index FTS không được dùng?"** → Biểu thức/config trong query không khớp lúc CREATE INDEX (thiếu config, sai 2-arg form), hoặc query không phải dạng đã index. Kiểm bằng `EXPLAIN ANALYZE` tìm `Bitmap Index Scan`.

5. **[BẪY] "Một hàng biến mất khỏi kết quả search dù có chứa từ khóa?"** → Thường do NULL ở một cột khi ghép mà quên `coalesce` → cả tsvector thành NULL. Bọc `coalesce(col,'')`.

6. **"Xử lý input user thô thế nào?"** → `websearch_to_tsquery` (kiểu Google, an toàn) hoặc `plainto_tsquery`; KHÔNG dùng `to_tsquery` với raw input (syntax error).

7. **"FTS có hiểu đồng nghĩa không? Nếu cần thì sao?"** → Không (trừ khi cấu hình thesaurus). Cần semantic → thêm vector search (pgvector), kết hợp **hybrid** bằng RRF.

8. **[SCALE] "20M docs, khi nào bỏ Postgres FTS sang Elasticsearch?"** → Khi search là workload chính, cần BM25/analyzer cao cấp, hoặc quy mô rất lớn. Trước đó Postgres FTS thắng ở operational simplicity + join SQL + ACID; có thể giữ pgvector cho semantic ngay trong Postgres.

### 5.6. One-liner đắt giá

- *"FTS matches lexemes, not characters — that's why 'run' finds 'running' and `LIKE` never will."*
- *"A GIN inverted index makes search time scale with the number of matches, not the size of the table."*
- *"For small-to-medium apps, Postgres FTS deletes your Elasticsearch cluster — same database, ACID, one SQL join, nothing to sync."*
- *"FTS knows words, vectors know meaning — hybrid search is just both, fused with RRF, inside one Postgres."*
- *"The strategic feature isn't tsvector — it's Postgres being extensible enough to do keyword, fuzzy, and semantic search without adding a single new system."*

---

### 📌 Ghi chú cuối

- **Kiểm chứng phiên bản khi ôn:** PostgreSQL hiện hành là 18 (19 beta). `websearch_to_tsquery` cần PG 11+, generated column STORED cần PG 12+. Xem docs `postgresql.org/docs/current/textsearch.html`.
- **Thực hành:** dựng Postgres bằng Docker, chạy lại `to_tsvector`/`to_tsquery`/`@@`, tạo generated column + GIN, rồi `EXPLAIN ANALYZE` để *thấy* `Bitmap Index Scan` bằng mắt. Sau đó cắm pgvector và tự viết một query hybrid RRF.
- **Nối mạch với giáo trình pgvector:** bài này (keyword/FTS) + bài trước (semantic/vector) = **hybrid search** — kiến trúc retrieval nền tảng cho search hiện đại và RAG.
- **Học tiếp:** RRF vs weighted fusion, reranking models, BM25 (và extension như ParadeDB/`pg_search` mang BM25 vào Postgres), text search configuration/dictionary tùy biến cho tiếng Việt.
