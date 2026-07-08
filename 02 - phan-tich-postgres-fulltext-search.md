# Phân tích chain học tập — PostgreSQL: Relational DB & Full-Text Search (tsvector / tsquery)

> Mục tiêu: tách rõ chỗ **suy ra được** (nhóm A, chỉ cần hiểu "vì sao") với chỗ **phải ghi nhớ** (nhóm B: định nghĩa / tên / con số / cú pháp), đánh dấu **⚠** chỗ logic nhảy, và rút ra **xương sống** để tái tạo cả tài liệu từ một bộ khung nhỏ.

Tài liệu có **21 chain**. File này nối trực tiếp với file 1 (pgvector): điểm hội tụ là **hybrid search = FTS (keyword) + vector (semantic)**. Cách học đúng là **tái tạo**, không phải chép lại.

---

# PHẦN 1 — PHÂN TÍCH TỪNG CHAIN

## Chain 1 — RDBMS refresher tới relationship

**A — Nhân quả**
- (gần như không có) — chuỗi định nghĩa nối tiếp.

**B — Ghi nhớ**
- **database** (dữ liệu tổ chức, số hóa), **DBMS** (phần mềm quản lý: Postgres/MySQL/Oracle) — *định nghĩa*.
- DBMS lưu dạng **file** trên đĩa nhưng trình bày dạng **table** (row/record + column, mỗi hàng là **entity**) — *định nghĩa*.
- **primary key (PK)** — *định nghĩa*: cột định danh duy nhất mỗi hàng (vd `cust_id`).
- **foreign key (FK)** — *định nghĩa*: cột ở bảng khác trỏ tới PK.
- **relationship** — *định nghĩa*: FK tạo liên kết giữa bảng (`cust_id` PK ở `customers`, FK ở `orders`).

> Chain này **chủ yếu nhóm B** — thuật ngữ nền tảng RDBMS, học như từ vựng.

**Xương sống**: (không nhân quả) database → DBMS → table → PK → FK → relationship.

---

## Chain 2 — BLOB tới ORDBMS & extensibility

**A — Nhân quả**
- Postgres mở rộng được => `pgvector` cắm thêm kiểu `vector`, FTS cắm thêm `tsvector`/`tsquery` + GIN/GiST — *vì* extensibility cho phép thêm data type / index method mới.
- cắm thêm vào DB sẵn có => không phải mua hệ thống riêng (luận điểm chiến lược chọn Postgres) — *vì* mở rộng tại chỗ nên không cần dựng stack mới.

**B — Ghi nhớ**
- **BLOB** (Binary Large Object) — *định nghĩa*: dữ liệu nhị phân (ảnh/video/file) lưu trong DB.
- **ORDBMS** (Object-Relational DBMS) — *tên gọi*: Postgres mở rộng được.
- **extensibility** = thêm được data type, function, **index method**, extension — *định nghĩa*.

**Xương sống**: Postgres là ORDBMS (mở rộng được) → pgvector & FTS cắm thêm được → không phải mua hệ thống riêng.

---

## Chain 3 — Tiền dùng kiểu nào

**A — Nhân quả**
- `float`/`double` là số thực dấu phẩy động => làm tròn sai => kế toán lệch xu — *vì* biểu diễn nhị phân không lưu chính xác số thập phân, sai số tích lũy.

**B — Ghi nhớ**
- Với tiền dùng **`numeric`** hoặc **`money`**, KHÔNG dùng `float`/`double` — *quy tắc* (lỗi kinh điển junior).

**Xương sống**: float làm tròn sai → lệch xu → tiền phải dùng numeric/money.

---

## Chain 4 — Vì sao LIKE gãy, dẫn tới lexeme

**A — Nhân quả**
- `%...%` (wildcard 2 đầu) => không dùng được B-tree index → **seq scan** chậm, tỉ lệ *tổng số hàng* — *vì* index B-tree cần tiền tố cố định, wildcard đầu phá điều đó.
- LIKE so đúng chuỗi => bỏ sót biến thể "ran"/"runs" — *vì* nó không biết các từ này cùng gốc.
- `'%run%'` => khớp nhầm "b**run**ch", "p**run**e" — *vì* so *ký tự con* nên trúng cả từ không liên quan.
- cả 3 vấn đề => đều **vì LIKE so ký tự** — *nguyên nhân gốc chung*.
- so ký tự hỏng => FTS sinh ra để **so lexeme** — *vì* cần đơn vị so sánh là "từ đã chuẩn hóa" chứ không phải chuỗi ký tự.

**B — Ghi nhớ**
- `LIKE '%...%'` = **O(N)** (chậm khi bảng to) — *con số/tính chất*.

**Xương sống**: LIKE so ký tự → 3 lỗi (seq scan O(N) / miss biến thể / match nhầm) → cần so lexeme → FTS.

---

## Chain 5 — Inverted index (mục lục cuối sách)

**A — Nhân quả**
- tra mục lục nhảy thẳng tới đúng trang => không đọc cả cuốn — *vì* mục lục ánh xạ "từ → trang".
- FTS tra "lexeme → document" => nhảy thẳng tới hàng khớp — *vì* cùng cấu trúc mục lục.
- chỉ nhảy tới hàng khớp => tốc độ phụ thuộc *số hàng khớp*, không phụ thuộc *tổng số hàng* — *vì* không quét toàn bảng.
- không quét toàn bảng => FTS nhanh còn `LIKE '%...%'` là O(N) — *vì* LIKE vẫn phải duyệt hết.

**B — Ghi nhớ**
- **inverted index** — *định nghĩa/analogy*: "từ → những trang/document chứa từ đó" (mục lục cuối sách).

**Xương sống**: FTS xây inverted index (mục lục) → nhảy thẳng tới hàng khớp → tốc độ ~ số hàng khớp, không phụ thuộc N.

---

## Chain 6 — Lexeme, stop-word, tsvector, tsquery, @@

**A — Nhân quả**
- (không có) — chuỗi định nghĩa.

**B — Ghi nhớ**
- **lexeme** — *định nghĩa*: từ đã chuẩn hóa (stemming) gộp biến thể về gốc: running/ran/runs → `run`.
- **stop-word** — *định nghĩa*: từ quá phổ biến (the/is/a) bị loại vì không mang nghĩa phân biệt.
- **`tsvector`** — *định nghĩa*: danh sách lexeme (đã bỏ stop-word + stem) sắp xếp kèm vị trí — tài liệu đã tiền xử lý.
- **`tsquery`** — *định nghĩa*: các lexeme nối bằng `&`(AND) `|`(OR) `!`(NOT) `<->`(FOLLOWED BY).
- **`@@`** — *cú pháp*: hỏi "tsvector này có thỏa tsquery kia không" → true/false.

> Chain này **hoàn toàn nhóm B** — chuỗi định nghĩa lõi FTS. Học thuộc như từ vựng.

**Xương sống**: (không nhân quả) lexeme → stop-word → tsvector → tsquery → @@.

---

## Chain 7 — Chạy tay tsvector (4 bước)

**A — Nhân quả**
- doc và query cùng stem về `run` => tìm lexeme `run` vẫn match dù câu gốc viết "running" — *vì* stemming đưa cả hai về cùng gốc (điều LIKE không làm được).

**B — Ghi nhớ (4 bước)**
- Bước 1 **tokenize**: The/quick/brown/foxes/are/running.
- Bước 2 **bỏ stop-word**: the, are bị loại.
- Bước 3 **stemming**: foxes→fox, running→run.
- Bước 4 **sắp xếp + gắn vị trí + loại trùng** → `'brown':3 'fox':4 'quick':2 'run':6` — *ví dụ cần thuộc*; số là vị trí token gốc, dùng cho phrase/ranking.

> Chain này chủ yếu nhóm B (worked example). Nhớ được output `'brown':3 'fox':4 'quick':2 'run':6` là nắm được cả cơ chế.

**Xương sống**: tokenize → bỏ stop-word → stem → sắp xếp+vị trí → tsvector; nhờ stem nên `run` match "running".

---

## Chain 8 — Hello world @@ & đối xứng chuẩn hóa

**A — Nhân quả**
- cả tài liệu lẫn truy vấn đi qua CÙNG bộ chuẩn hóa => chúng "gặp nhau" ở cấp lexeme — *vì* nếu chỉ một bên stem thì "running" (query) và `run` (doc) không khớp; cùng stem thì cùng thành `run`.

**B — Ghi nhớ**
- `to_tsvector('english', text)` (config + text) → tsvector; `to_tsquery('english','running')` → tsquery **cũng bị stem** về `'run'`; `@@` ghép hai vế trả true/false — *cú pháp*.
- `@@` là "trái tim" của FTS (tương tự `<->` là trái tim của vector search trong file 1) — *mental model*.

**Xương sống**: query cũng bị stem như doc → cùng bộ chuẩn hóa → gặp nhau ở lexeme → @@ match.

---

## Chain 9 — Pipeline nội bộ: text → tsvector

**A — Nhân quả**
- chọn config sai => kết quả sai — *vì* config quyết định stem/bỏ stop-word; sai config thì lexeme sinh ra sai.

**B — Ghi nhớ**
- **text search configuration** (vd `'english'`) = **parser** + **dictionaries** — *định nghĩa*.
- **parser**: tách token + gán loại token (từ/số/email/URL/host); email và URL nhận diện riêng, không vỡ vụn — *định nghĩa*.
- **dictionaries**: stop-word dict, Snowball stemmer, synonym/thesaurus (tùy chọn) — *định nghĩa*.
- `'simple'` KHÔNG stem, không bỏ stop-word (chỉ lowercase) → mã sản phẩm/username; `'english'` stem tiếng Anh; tiếng Việt stemming hạn chế → `'simple'` + `unaccent` hoặc chuyển vector search — *quy tắc*.

> Chain này chủ yếu nhóm B (định nghĩa pipeline). Điểm suy luận duy nhất: config sai → kết quả sai.

**Xương sống**: config = parser + dictionaries → chọn config sai thì kết quả sai.

---

## Chain 10 — GIN index & coalesce

**A — Nhân quả**
- không index => FTS rơi về seq scan => nên đánh **GIN index** — *vì* cần tra inverted index thay vì quét.
- functional GIN index một biểu thức cụ thể => query PHẢI khớp đúng biểu thức (cùng config, cùng 2-arg form) — *vì* planner chỉ dùng index khi biểu thức query trùng biểu thức index.
- một cột NULL mà không `coalesce` => TOÀN BỘ tsvector thành NULL → hàng biến mất khỏi search — *vì* `NULL || x = NULL`, phép nối lan NULL ra cả tsvector.

**B — Ghi nhớ**
- **functional GIN**: `CREATE INDEX ... USING GIN (to_tsvector('english', body))` — không tốn thêm cột — *cú pháp*.
- **generated `tsvector` column STORED** (PG 12+): `ALTER TABLE ... ADD COLUMN ... GENERATED ALWAYS AS (...) STORED`, Postgres tự đồng bộ; thay cho **trigger** sync tay ngày xưa (dễ quên/lệch) — *cú pháp*.
- Ghép nhiều cột phải bọc `coalesce(col,'')` — *cú pháp/quy tắc*.

**Xương sống**: không index → seq scan → đánh GIN → query khớp biểu thức index → coalesce chống NULL nuốt tsvector.

---

## Chain 11 — Ranking & setweight

**A — Nhân quả**
- `@@` chỉ cho true/false => muốn "liên quan nhất lên đầu" cần **ranking** — *vì* boolean không xếp thứ tự được.

**B — Ghi nhớ**
- **`ts_rank`** — *định nghĩa*: chấm điểm theo **tần suất** lexeme khớp.
- **`ts_rank_cd`** (cover density) — *định nghĩa*: thưởng khi từ khớp nằm gần nhau → hợp tìm cụm.
- **`setweight`** — *định nghĩa/cú pháp*: gán trọng số **A/B/C/D** (A cao nhất) cho từng nguồn text (title→A, body→B, tags→C).

> Chain này chủ yếu nhóm B (định nghĩa hàm ranking). Điểm suy luận: @@ boolean nên phải có ranking riêng.

**Xương sống**: @@ chỉ true/false → cần ranking → ts_rank (tần suất) / ts_rank_cd (cụm) / setweight (trọng số trường).

---

## Chain 12 — ts_rank chi tiết & I/O bottleneck

**A — Nhân quả**
- ranking phải MỞ tsvector của từng document khớp => tốn I/O, dễ thành bottleneck — *vì* mỗi doc khớp phải đọc lên để chấm điểm.
- I/O tỉ lệ số hàng cần rank => giải bằng filter cứng trước để giảm số hàng, hoặc chỉ rank top-N — *vì* càng ít hàng rank càng ít I/O.

**B — Ghi nhớ**
- `ts_rank(weights, vector, query, normalization)` — *cú pháp*.
- `weights` default `{0.1, 0.2, 0.4, 1.0}` tương ứng **D/C/B/A** — *con số*.
- `normalization` là bitmask chia điểm theo độ dài tài liệu (tránh doc dài luôn thắng); cờ **32** = `rank/(rank+1)` — *con số/cú pháp*.

**Xương sống**: ts_rank mở tsvector từng doc khớp → I/O bottleneck → filter cứng trước + rank top-N.

---

## Chain 13 — Xử lý input user an toàn

**A — Nhân quả**
- raw input user vào `to_tsquery` => khoảng trắng/dấu chấm → syntax error → request 500 — *vì* `to_tsquery` yêu cầu cú pháp toán tử chặt.
- cần chịu input lộn xộn => dùng **`websearch_to_tsquery`** — *vì* nó nuốt được cú pháp tự do mà không gãy.

**B — Ghi nhớ**
- `websearch_to_tsquery` (PG 11+): nuốt cú pháp Google — ngoặc kép (cụm), dấu trừ (loại trừ), OR — *định nghĩa*.
- `plainto_tsquery`: text thô, tự AND các từ ("có tất cả các từ") — *định nghĩa*.
- `to_tsquery`: chỉ khi BẠN kiểm soát query và cần toán tử chính xác — *quy tắc*.

**Xương sống**: input user thô làm to_tsquery gãy (500) → ô search user dùng websearch_to_tsquery.

---

## Chain 14 — GIN internals & vì sao nhanh

**A — Nhân quả**
- GIN tra thẳng **posting list** của lexeme → row id => chỉ đọc đúng hàng khớp => complexity phụ thuộc số document khớp, không phụ thuộc N — *vì* posting list liệt kê sẵn doc chứa lexeme.
- không phụ thuộc N => FTS giữ tốc độ khi bảng phình, còn `LIKE '%...%'` là O(N) — *vì* LIKE vẫn duyệt hết.

**B — Ghi nhớ**
- **GIN** = Generalized Inverted Index, lưu lexeme → **posting list** — *tên gọi/định nghĩa*.
- GIN build tốn hơn B-tree, **nhạy `maintenance_work_mem`** (tăng RAM → build nhanh hơn nhiều) — *dữ kiện*.
- GIN update chậm khi ghi nhiều, có **`fastupdate`** buffer để hoãn cập nhật — *tên gọi/cơ chế*.

**Xương sống**: posting list → chỉ đọc hàng khớp → complexity ~ số hàng khớp (không phụ thuộc N); đổi lại build/update tốn hơn.

---

## Chain 15 — Prefix, phrase, fuzzy (3 tầng search)

**A — Nhân quả**
- FTS stem không sửa lỗi chính tả => "postgrsql" không thành lexeme `postgresql` → miss => cần **`pg_trgm`** (trigram) chống typo — *vì* trigram so đoạn 3 ký tự nên chịu được sai vài chữ.

**B — Ghi nhớ**
- **prefix matching** `:*` — gõ nửa từ: `'postgr:*'` khớp postgres/postgresql — *cú pháp*.
- **phrase/proximity** `<->` (FOLLOWED BY) / `<N>` (cách N từ) — yêu cầu thứ tự — *cú pháp*.
- **`pg_trgm`**: `CREATE INDEX ... USING GIN (title gin_trgm_ops)`, toán tử `%`, hàm `similarity()` — *cú pháp/tên*.
- Pattern production **3 tầng**: FTS (lexeme, chính xác) + trigram (typo) + vector (ngữ nghĩa) — *pattern* (tầng vector nối với file 1).

> Chain này chủ yếu nhóm B (cú pháp các kỹ thuật). Điểm suy luận: FTS không sửa typo → cần trigram.

**Xương sống**: prefix/phrase mở rộng FTS → nhưng FTS không sửa typo → thêm trigram → gộp 3 tầng.

---

## Chain 16 — Edge cases cần thủ sẵn

**A — Nhân quả**
- (giữa các edge case KHÔNG có nhân quả — xem ⚠)

**B — Ghi nhớ (5 edge case)**
- (1) NULL nuốt tsvector → luôn `coalesce` khi ghép cột.
- (2) Ngôn ngữ: stemmer `'english'` sai với tiếng Việt/Nhật/Trung; đa ngôn ngữ 1 bảng → lưu config theo cột: `GIN(to_tsvector(config_col, body))`.
- (3) Stop-word làm mất cụm: "the who" (tên ban nhạc) bị bỏ "the" → dùng config `'simple'`.
- (4) `CREATE INDEX` khóa ghi → dùng **`CREATE INDEX CONCURRENTLY`** trên bảng lớn production.
- (5) `@@` không dùng index nếu vế trái không phải biểu thức đã index → luôn `EXPLAIN ANALYZE` xác nhận "Bitmap Index Scan".

> ⚠ Đây là **danh sách 5 edge case**, không phải chuỗi nhân quả — các `=>` chỉ ngăn cách mục. Đừng cố nối "vì… nên…" giữa case 1 và case 2. Học như một checklist thủ sẵn.

**Xương sống**: (không phải nhân quả liền mạch) — checklist: coalesce / config theo cột / 'simple' cho stop-word / CONCURRENTLY / EXPLAIN.

---

## Chain 17 — Scale FTS & 3 bottleneck

**A — Nhân quả**
- GIN query ~ số hàng khớp => FTS sub-second trên triệu doc NẾU filter đủ chọn lọc => thay được Elasticsearch cho app small–medium — *vì* độ nhanh đủ dùng ở quy mô nhỏ-vừa.
- **bottleneck 1 — ranking I/O**: `ts_rank` mở tsvector từng doc; query phổ biến ("the","data") khớp triệu hàng => I/O bound → giải: filter cứng trước, persist tsvector, rank top-N — *vì* rank nhiều hàng = nhiều I/O.
- **bottleneck 2 — write amplification**: GIN update chậm → ghi nặng thì cân `fastupdate`/GiST/tách bảng — *vì* mỗi ghi phải cập nhật nhiều posting list.
- **bottleneck 3 — index build khóa write** → `CREATE INDEX CONCURRENTLY` + tăng `maintenance_work_mem` — *vì* build thường khóa, CONCURRENTLY tránh chặn.

**B — Ghi nhớ**
- Scale-out: read replicas cho search + partition (tenant/thời gian) + GIN per-partition để planner prune sớm — *dữ kiện*.

> ⚠ Phần "3 bottleneck" nửa là danh sách — mỗi bottleneck có nhân quả riêng nhưng không suy ra nhau. Nắm theo cụm "vấn đề → cách giải".

**Xương sống**: FTS đủ nhanh nếu filter chọn lọc → 3 bottleneck (ranking I/O / write amplification / build khóa) mỗi cái có cách giải → scale-out replica + partition prune.

---

## Chain 18 — Hybrid search: FTS + pgvector

**A — Nhân quả**
- FTS và vector **bổ khuyết** nhau (FTS chính xác thuật ngữ/mã nhưng mù nghĩa; vector hiểu nghĩa nhưng trượt thuật ngữ hiếm) => chạy CẢ HAI rồi hợp nhất = **hybrid search** — *vì* điểm mù của bên này được bên kia bù.

**B — Ghi nhớ**
- Đặc điểm hai bên: FTS = keyword/lexeme, chính xác, giải thích được, nhanh, MÙ ngữ nghĩa ("xe hơi" ≠ "ô tô"); vector = semantic, hiểu đồng nghĩa, khó giải thích, trượt mã/thuật ngữ hiếm — *dữ kiện*.
- **Reciprocal Rank Fusion (RRF)** — *định nghĩa/công thức*: lấy top-K mỗi nhánh, cộng `SUM(1/(60 + r))`.
- Hằng số **60** trong RRF để làm mượt (giảm ảnh hưởng thứ hạng rất cao) — *con số*.
- Hybrid ngay TRONG một Postgres (không cần Pinecone + Elasticsearch song song); là kiến trúc retrieval mặc định cho search/RAG 2026 — *dữ kiện*.

**Xương sống**: FTS và vector bổ khuyết → chạy cả hai → fuse bằng RRF → hybrid search trong một Postgres.

---

## Chain 19 — Cost, monitoring, failure, reindex

**A — Nhân quả**
- FTS tận dụng Postgres (gần như miễn phí hạ tầng) => tiết kiệm tiền + on-call surface so với cụm ES riêng — *vì* không dựng thêm hệ thống.

**B — Ghi nhớ**
- Monitor: p95/p99 search, tỉ lệ query rơi seq scan, độ trễ ranking, kích thước GIN, write latency (`pg_stat_statements`, `EXPLAIN ANALYZE`) — *dữ kiện/công cụ*.
- 5 failure mode: (1) config/biểu thức lệch → index bị bỏ, latency tăng âm thầm; (2) NULL nuốt tsvector → doc mất tích; (3) query stop-word/siêu phổ biến → rank triệu hàng, treo; (4) đổi config → phải reindex; (5) `fastupdate` buffer phình khi ghi nhiều chưa vacuum — *checklist*.
- Reindex an toàn: **`REINDEX INDEX CONCURRENTLY`** (PG 12+) để không khóa — *cú pháp*.

> ⚠ Phần 5 failure mode là **danh sách**, không phải nhân quả. Học như checklist.

**Xương sống**: FTS tận dụng Postgres → rẻ + ít on-call → monitor + thủ 5 failure mode → reindex bằng CONCURRENTLY.

---

## Chain 20 — Tổ chức & config roadmap

**A — Nhân quả**
- đổi **text search config** => tsvector sinh ra khác => phải reindex toàn bảng (tốn/khóa) → version hóa + lên kế hoạch như một migration — *vì* mọi tsvector cũ được tạo bằng config cũ, đổi config làm chúng lệch.
- giữ FTS trong Postgres => backend team hiện tại tự lo, khỏi tuyển search stack riêng — *vì* cùng công nghệ đội đã biết.

**B — Ghi nhớ**
- Nói với PM/sếp bằng **risk & cost**, không jargon — *khuyến nghị*.
- Text search config (ngôn ngữ/thesaurus/weight) ảnh hưởng relevance **toàn sản phẩm** — *nhận định cần thuộc*.

> Đây là bản song song của "đổi embedding model = re-embed" ở file 1: **đổi quyết định nền tảng = làm lại toàn bộ dữ liệu phái sinh**.

**Xương sống**: config quyết định relevance toàn sản phẩm → đổi config = reindex toàn bảng (migration) → version hóa, lên kế hoạch.

---

## Chain 21 — Khi nào vượt Postgres sang Elasticsearch

**A — Nhân quả**
- search trở thành **workload/sản phẩm chính** => cân nhắc Elasticsearch/OpenSearch — *vì* khi đó cần relevance/scale chuyên dụng mà `ts_rank` không đạt.

**B — Ghi nhớ (tiêu chí)**
- Postgres FTS thắng ở operational simplicity + join SQL + ACID cho **small–medium** — *nhận định*.
- Vượt sang ES khi: search là sản phẩm chính; cần **BM25/analyzer cao cấp**; quy mô rất lớn / analytics text — *tiêu chí*.
- Khi đó có thể giữ pgvector cho nhánh semantic, chỉ chuyển nhánh keyword sang ES — *khuyến nghị*.

> ⚠ Phần lớn chain này là **danh sách tiêu chí**, không phải nhân quả (giống chain "khi nào NÊN/KHÔNG NÊN pgvector" ở file 1).

**Xương sống**: FTS đủ cho small-medium → search là sản phẩm chính / cần BM25 / quy mô rất lớn → vượt sang Elasticsearch (giữ pgvector cho semantic).

---

# PHẦN 2 — CÁC MỤC TOÀN CỤC

## 7. XƯƠNG SỐNG TOÀN CỤC

Nối các xương sống thành **một mạch nhân quả duy nhất**:

> RDBMS (table / PK / FK / relationship) → Postgres là **ORDBMS mở rộng được** → cắm thêm FTS & pgvector mà không mua hệ thống riêng → `LIKE` **so ký tự nên gãy** (seq scan O(N) / miss biến thể / match nhầm) → FTS **so lexeme** → xây **inverted index** (mục lục cuối sách) → tốc độ ~ **số hàng khớp**, không phụ thuộc N → lexeme = từ đã stem + bỏ stop-word, lưu trong **tsvector**; query → **tsquery**; **`@@`** so khớp → doc và query **cùng bộ chuẩn hóa** nên gặp nhau ở lexeme → đánh **GIN index** (posting list) để khỏi seq scan → `@@` chỉ true/false nên cần **ranking** (ts_rank / setweight) → ranking mở tsvector từng doc → **I/O bottleneck** → filter cứng trước + rank top-N → input user thô làm `to_tsquery` gãy → **`websearch_to_tsquery`** → FTS không sửa typo → thêm **trigram (pg_trgm)**; FTS mù nghĩa → thêm **vector** → **3 tầng / hybrid search** → fuse bằng **RRF** trong một Postgres → chọn Postgres khi search là *feature*; vượt **Elasticsearch** khi search là *sản phẩm chính*.

Nắm mạch này là dựng lại được phần lớn tài liệu. Mọi con số (`{0.1,0.2,0.4,1.0}`, hằng số 60…), cú pháp (`@@`, `websearch_to_tsquery`…) và tên riêng chỉ là chi tiết treo lên khung.

---

## 8. MẪU HÌNH LẶP LẠI

Gom các chain nói **cùng một nguyên lý** — hiểu một lần, dùng nhiều chỗ:

**Mẫu hình 1 — Đối xứng: hai phía phải qua cùng một phép biến đổi.**
Xuất hiện ở: doc và query cùng bộ chuẩn hóa mới gặp nhau ở lexeme (chain 8); functional GIN đòi query khớp đúng biểu thức đã index (chain 10). *Vì sao là một*: FTS chỉ hoạt động khi hai vế được xử lý bằng **cùng một hàm** — lệch một chút là không match / không dùng được index. (Song song với "ops class phải khớp toán tử" ở file 1.)

**Mẫu hình 2 — Lỗi im lặng (silent failure).**
Xuất hiện ở: NULL nuốt tsvector → doc mất tích (chain 10, 16, 19); config/biểu thức lệch → index bị bỏ, latency tăng âm thầm (chain 16, 19); trigger sync tay dễ lệch (chain 10). *Vì sao là một*: hệ **không báo lỗi**, chỉ mất kết quả hoặc chậm đi → phải chủ động `coalesce`, `EXPLAIN ANALYZE`, kiểm định kỳ. (Trùng nguyên lý với "lỗi im lặng" ở file 1.)

**Mẫu hình 3 — Inverted index: tốc độ ~ số hàng khớp, không phụ thuộc N.**
Xuất hiện ở: analogy mục lục (chain 5); posting list GIN (chain 14); FTS sub-second nếu filter chọn lọc (chain 17). *Vì sao là một*: đều cùng một lý do FTS nhanh — nhảy thẳng tới hàng khớp thay vì quét toàn bảng.

**Mẫu hình 4 — Đổi quyết định nền tảng = làm lại toàn bộ dữ liệu phái sinh.**
Xuất hiện ở: đổi text search config = reindex toàn bảng (chain 20); và cross-file: đổi embedding model = re-embed corpus (file 1). *Vì sao là một*: tsvector/embedding đều được sinh từ một "cấu hình gốc"; đổi gốc thì mọi thứ phái sinh phải build lại → luôn version hóa + lên kế hoạch như migration.

**Mẫu hình 5 — Lọc thô/cứng trước, tinh/đắt sau.**
Xuất hiện ở: filter cứng trước rồi rank (chain 12, 17); rank top-N; RRF fuse top-K mỗi nhánh (chain 18). *Vì sao là một*: đều thu hẹp tập ứng viên bằng bước rẻ trước, rồi mới bỏ công tính đắt trên tập nhỏ. (Trùng "two-stage retrieval" ở file 1.)

**Mẫu hình 6 — Operational simplicity: cắm vào Postgres, không mua hệ thống riêng.**
Xuất hiện ở: extensibility (chain 2); hybrid trong một Postgres (chain 18); FTS tận dụng Postgres nên rẻ (chain 19); chọn Postgres khi search là feature (chain 21). *Vì sao là một*: luận điểm chiến lược xuyên suốt — một hệ thống, một câu SQL, một transaction. (Trùng file 1.)

**Mẫu hình 7 — `CONCURRENTLY` để không khóa write trên production.**
Xuất hiện ở: `CREATE INDEX CONCURRENTLY` (chain 16, 17); `REINDEX INDEX CONCURRENTLY` (chain 19). *Vì sao là một*: mọi thao tác index nặng trên bảng đang chạy đều phải tránh khóa ghi.

---

## 9. NHÓM B GỘP LẠI — DANH SÁCH PHẢI HỌC THUỘC

(Gộp cả các BẢNG/ĐÃ PHỦ trong file gốc.)

### Con số
- `ts_rank` weights default `{0.1, 0.2, 0.4, 1.0}` = **D/C/B/A**.
- `normalization` cờ **32** = `rank/(rank+1)`.
- `setweight` nhãn **A/B/C/D** (A cao nhất).
- RRF: `SUM(1/(60 + r))`, hằng số **60** (làm mượt).
- tsvector ví dụ chạy tay: `'brown':3 'fox':4 'quick':2 'run':6`.
- `LIKE '%...%'` = **O(N)**.
- Version: PostgreSQL **18** (19 beta 6/2026; 17/16/15/14 còn hỗ trợ); `websearch_to_tsquery` **PG 11+**; generated column STORED & `REINDEX CONCURRENTLY` **PG 12+**.

### Định nghĩa & tên gọi
- **database, DBMS, table/row/entity/column, primary key, foreign key, relationship, BLOB**.
- **ORDBMS**; **extensibility** = data type + function + index method + extension.
- **lexeme** (stemming), **stop-word**, **tsvector**, **tsquery**, **`@@`**.
- Toán tử tsquery: `&`(AND) `|`(OR) `!`(NOT) `<->`(FOLLOWED BY) `<N>`(cách N từ).
- **inverted index** / mục lục cuối sách / **posting list**; **GIN** = Generalized Inverted Index; **GiST** (signature tree, lossy → recheck).
- **text search configuration** = **parser** (token + loại token) + **dictionaries** (stop-word / Snowball stemmer / thesaurus).
- config **`'simple'`** (chỉ lowercase) vs **`'english'`** (stem).
- Họ hàm tsquery: `to_tsquery` / `plainto_tsquery` / `phraseto_tsquery` / `websearch_to_tsquery`.
- Cách persist: **functional GIN** vs **generated column STORED** vs **trigger** (né).
- **`ts_rank`** (tần suất) vs **`ts_rank_cd`** (cover density); **`setweight`**; **normalization** bitmask.
- **prefix** `:*`; **phrase** `<->`/`<N>`; **`pg_trgm`** (trigram, `gin_trgm_ops`, `%`, `similarity()`).
- **`fastupdate`**, **`maintenance_work_mem`**.
- **hybrid search**, **Reciprocal Rank Fusion (RRF)**.
- Kiểu tiền: **`numeric`/`money`** (không `float`/`double`).

### Cú pháp
- `CREATE INDEX docs_fts ON docs USING GIN (to_tsvector('english', body));`
- `to_tsvector('english', text)` · `to_tsquery('english','running')` · `... @@ ...`
- `websearch_to_tsquery('english', 'postgres -mysql "full text"')`
- `ALTER TABLE docs ADD COLUMN sv tsvector GENERATED ALWAYS AS (setweight(to_tsvector('english', coalesce(title,'')),'A') || setweight(to_tsvector('english', coalesce(body,'')),'B')) STORED;`
- `SELECT title, ts_rank(sv, q) r FROM docs, websearch_to_tsquery('english', :input) q WHERE sv @@ q ORDER BY r DESC LIMIT 10;`
- `CREATE INDEX ... USING GIN (title gin_trgm_ops);` (typo) · prefix `'postgr:*'`
- `CREATE INDEX CONCURRENTLY` / `REINDEX INDEX CONCURRENTLY`
- Hybrid RRF: `WITH kw AS (...), sem AS (...) SELECT id, SUM(1.0/(60+r)) AS rrf_score ... GROUP BY id ORDER BY rrf_score DESC;`
- `EXPLAIN ANALYZE` → xác nhận "Bitmap Index Scan".

---

## 10. BỘ CÂU HỎI "VÌ SAO" (luyện tái tạo — không kèm đáp án)

> Cách dùng: che tài liệu, trả lời bằng lời của mình. Chỗ **ấp úng = lỗ hổng cần đào**, không phải "chưa thuộc" — quay lại đúng chain và tự dựng lại chuỗi theo logic.

1. Vì sao `LIKE '%running%'` bị 3 lỗi (seq scan, miss biến thể, match nhầm), và cả 3 quy về nguyên nhân gốc nào?
2. Vì sao inverted index / posting list khiến tốc độ FTS phụ thuộc **số hàng khớp** chứ không phải tổng số hàng?
3. Vì sao cả tài liệu lẫn truy vấn phải đi qua **cùng bộ chuẩn hóa** thì `@@` mới match?
4. Vì sao một cột NULL không `coalesce` lại làm **toàn bộ** tsvector thành NULL và hàng biến mất khỏi search?
5. Vì sao functional GIN đòi query phải khớp **đúng biểu thức** đã index?
6. Vì sao `@@` cần đi kèm `ts_rank`/`setweight` khi muốn "liên quan nhất lên đầu"?
7. Vì sao ranking (`ts_rank`) dễ thành **I/O bottleneck**, và cách giảm là gì?
8. Vì sao đưa raw input user vào `to_tsquery` nguy hiểm, và `websearch_to_tsquery` xử lý thế nào?
9. Vì sao FTS không bắt được typo "postgrsql", và cần thêm công cụ gì?
10. Vì sao FTS và vector search **bổ khuyết** nhau, dẫn tới hybrid search?
11. Hằng số **60** trong RRF có tác dụng gì?
12. Vì sao đổi text search config lại là một **migration nặng** (phải reindex toàn bảng)?
13. Vì sao GIN build/update tốn hơn B-tree, và `fastupdate` / `maintenance_work_mem` liên quan thế nào?
14. Vì sao dùng `CREATE INDEX CONCURRENTLY` / `REINDEX CONCURRENTLY` trên production?
15. Vì sao Postgres FTS đủ cho small–medium nhưng nên vượt sang Elasticsearch khi search là **sản phẩm chính**?

---

# CHIẾN LƯỢC HỌC (ngắn gọn)

- **Nhóm A**: tự hỏi "vì sao mũi tên này tồn tại", hiểu là xong — đừng chép lại.
- **Nhóm B**: học thuộc riêng như từ vựng (mục 9). Chú ý chain 1 và chain 6 gần như toàn B — chuỗi định nghĩa nền tảng.
- **Tái tạo**: che bản gốc, tự dựng lại xương sống toàn cục (mục 7) theo logic, không nhớ từng chữ.
- Ưu tiên nắm **7 mẫu hình lặp lại** (mục 8) — nhiều mẫu trùng với file 1 (lỗi im lặng, đổi nền tảng = làm lại, lọc thô trước tinh sau, operational simplicity), nên hiểu một lần là dùng được cho cả hai tài liệu.
