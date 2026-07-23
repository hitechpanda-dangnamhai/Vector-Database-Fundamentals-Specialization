# Bulk Insert dữ liệu Vector vào pgvector: COPY, pg-promise, psycopg — Giáo trình siêu dễ hiểu, Basic → Staff

> **Bài giảng gốc:** video *"Techniques for bulk data transfer, including pg-promise"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy bốn thứ: vì sao bulk insert quan trọng, lệnh **COPY**, bulk insert trong Node.js bằng **pg-promise**, và trong Python bằng **psycopg2** (`execute_values`). Tôi giảng bám sát rồi đào sâu thành một pipeline nạp dữ liệu thật. Chỗ đi xa hơn bài gốc được đánh dấu 🧩 **[Ngoài bài gốc]**.
>
> **Vị trí trong series:** đây là mảnh **"nạp dữ liệu vào cho NHANH"**. Cài pgvector xong, tạo bảng xong — giờ là lúc đổ hàng triệu vector vào mà không làm sập hệ thống.
>
> **⚠️ Bốn đính chính và cập nhật tới tháng 7/2026 — đọc kỹ để đi phỏng vấn không nói sai:**
> 1. **COPY là cách bulk insert NHANH NHẤT**, không phải "một lựa chọn dành cho file CSV". Thứ tự tốc độ đúng là: `INSERT từng dòng` ≪ `batch INSERT` (pg-promise, execute_values) ≪ **`COPY`**. Bài gốc trình bày COPY gần như ngang hàng với batch insert, nên cần nói rõ lại.
> 2. **`psycopg2` là thư viện cũ; bản kế nhiệm là `psycopg` phiên bản 3** (dân trong nghề hay gọi là "psycopg3", hiện ở nhánh 3.3.x). Trong psycopg3 **không còn hàm `execute_values`** — thay bằng `executemany()` (đã được tối ưu bằng pipeline mode) hoặc `cursor.copy()` (nhanh nhất). Code psycopg2 của bài gốc vẫn chạy được, nhưng dự án mới nên biết bản mới.
> 3. **Bài gốc bỏ qua nguyên tắc vàng của bulk load:** *nạp toàn bộ dữ liệu TRƯỚC, tạo index (HNSW/IVFFlat) SAU.* Bắt database vừa nhận dữ liệu vừa vá index là cách chắc chắn nhất để biến một job 2 tiếng thành một job 2 ngày.
> 4. **Vector phải được đăng ký kiểu** (`register_vector`) hoặc viết đúng định dạng chuỗi `'[1,2,3]'` thì driver mới chuyển đổi được khi bulk insert hoặc COPY.
>
> **Giáo trình này tự đứng độc lập được** — mọi thuật ngữ đều được giải thích lại từ đầu, bạn không cần đọc các bài trước trong series.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### Bài này dạy gì — nói bằng một câu đơn giản nhất

**Bài này dạy bạn cách đổ một lượng dữ liệu khổng lồ vào cơ sở dữ liệu cho thật nhanh, thay vì đưa vào từng món một.**

Nghe thì tưởng là chuyện vặt. Nhưng khác biệt giữa làm đúng và làm sai ở đây là khác biệt giữa **hai tiếng** và **hai ngày** cho cùng một công việc. Và với dữ liệu vector — nơi mỗi dòng nặng vài kilobyte thay vì vài chục byte — khoảng cách đó còn giãn ra thêm nữa.

Nếu ngay lúc này bạn chưa biết "transaction" là gì, "COPY" là gì, hay "vector" là gì — hoàn toàn bình thường. Phần 1 dựng lại từ con số 0.

### Vấn đề nó giải quyết

Mỗi câu lệnh thêm dữ liệu mà bạn gửi cho database đều phải trả một khoản **chi phí cố định** — bất kể bạn thêm một dòng hay một nghìn dòng. Chi phí đó gồm việc gửi qua mạng, ghi nhật ký an toàn, và xác nhận đã lưu.

Nếu bạn thêm 1 triệu dòng bằng 1 triệu câu lệnh riêng lẻ, bạn trả khoản phí đó **một triệu lần**.

Bulk insert (nạp hàng loạt) là nghệ thuật trả khoản phí đó **một lần thay vì một triệu lần**. Chỉ vậy thôi. Nhưng nó cho ra kết quả nhanh hơn hàng chục tới hàng trăm lần.

### Học xong bạn sẽ làm được

- Giải thích được **chính xác vì sao** bulk insert nhanh hơn insert từng dòng (chứ không chỉ nói "vì nó gom lại").
- Xếp đúng thứ tự tốc độ của ba phương pháp, và biết chọn cái nào trong tình huống nào.
- Dùng **COPY** để nạp vector từ file CSV và từ luồng dữ liệu.
- Viết bulk insert từ **Node.js** (pg-promise) và từ **Python** (cả psycopg2 cũ lẫn psycopg3 mới).
- Thiết kế một **pipeline nạp dữ liệu** hoàn chỉnh ở quy mô hàng triệu vector: nạp → build index → kiểm chứng, có khả năng chạy lại an toàn khi đứt giữa chừng.

### Mạch kiến thức: basic → staff

- 🟢 **Basic** — Vì sao insert từng dòng lại chậm → analogy chuyển nhà → ba phương pháp và thứ tự tốc độ → COPY cơ bản (CSV vào bảng) → cái bẫy vector trong file CSV.
- 🟡 **Intermediate** — pg-promise trong Node.js (ColumnSet, helpers.insert, db.none); psycopg2 trong Python (execute_values, register_vector, commit); ba lỗi kinh điển.
- 🔴 **Advanced** — Vì sao COPY nhanh nhất (mổ xẻ transaction, WAL, giao thức binary); psycopg3 `copy()` hiện đại; **nguyên tắc vàng nạp-trước-index-sau**; chọn batch size; các trường hợp biên.
- 🟣 **Staff** — Pipeline nạp ở quy mô triệu vector; drop rồi rebuild index; idempotency và upsert; áp lực WAL lên replication; giám sát; câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — Từ khoá, ý cốt lõi, code cần thuộc, câu hỏi phỏng vấn và các câu chốt "đắt giá".

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người chưa biết gì, và tôi giải thích cả những thứ mà dân backend coi là hiển nhiên. Nếu bạn đã quen với SQL, đừng bỏ mục 1.2 — nó là chỗ giải thích *vì sao* mọi thứ chậm, và mọi kỹ thuật ở các phần sau đều mọc lên từ đó.

### 1.1. Bắt đầu từ nỗi đau: một triệu vector và một buổi tối dài

Vài từ cần giải thích trước đã, để đoạn sau bạn không phải đoán chữ nào:

- **Database** (cơ sở dữ liệu) — nơi lưu trữ dữ liệu có tổ chức của một ứng dụng.
- **Table** (bảng) — dữ liệu được tổ chức thành cột và dòng, giống một sheet Excel.
- **Row** / **record** (dòng / bản ghi) — một dòng trong bảng, chứa thông tin về một đối tượng.
- **SQL** (viết tắt của *Structured Query Language*) — ngôn ngữ tiêu chuẩn để ra lệnh cho database.
- **`INSERT`** — câu lệnh SQL để **thêm dữ liệu mới** vào bảng.
- **Vector** (trong ngữ cảnh này còn gọi là **embedding**) — một dãy số biểu diễn ý nghĩa của một đoạn chữ, do một mô hình AI sinh ra. Ví dụ câu "thủ đô Anh là London" biến thành `[0.12, -0.98, 0.33, ...]` gồm 512 con số. Số lượng con số trong dãy gọi là **dimension** (số chiều).
- **pgvector** — phần mở rộng cho PostgreSQL, giúp nó lưu và tìm kiếm được vector. Nó thêm cho Postgres một kiểu dữ liệu mới tên là `vector`.

Bây giờ tới tình huống. Bạn vừa sinh xong embedding cho **1 triệu tài liệu** và cần đưa chúng vào database. Cách ngây thơ nhất là viết một vòng lặp:

```python
for tai_lieu in danh_sach:                       # lặp qua 1 triệu tài liệu
    cur.execute("INSERT INTO faqs (text, embedding) VALUES (%s, %s)",
                (tai_lieu.text, tai_lieu.embedding))
    conn.commit()                                # ← lưu ngay sau mỗi dòng
```

Đoạn code này **đúng**. Nó chạy. Nó sẽ nạp đủ 1 triệu dòng.

Vấn đề là nó có thể mất **nhiều giờ**, trong khi cách đúng chỉ mất **vài phút**. Và điều tệ hơn cả sự chậm là: bạn thường không biết mình đang chậm, vì lúc test với 100 dòng thì nó nhanh như chớp.

### 1.2. ⭐ Vì sao insert từng dòng lại chậm đến vậy — mổ xẻ chi phí

Đây là mục quan trọng nhất của Phần 1. Nếu bạn hiểu mục này, mọi kỹ thuật ở các phần sau sẽ trở nên hiển nhiên.

Câu trả lời ngắn: **mỗi câu `INSERT` riêng lẻ là một transaction độc lập, và mỗi transaction có một khoản chi phí cố định phải trả — bất kể bạn thêm 1 dòng hay 10.000 dòng.**

Trước hết, **transaction** là gì?

> **Transaction** (giao dịch) là **một nhóm thao tác được database đảm bảo là "hoặc tất cả cùng thành công, hoặc tất cả cùng không xảy ra"**. Không bao giờ có chuyện làm được một nửa.
>
> Analogy: chuyển tiền ngân hàng. Việc "trừ tiền tài khoản A" và "cộng tiền tài khoản B" phải nằm trong cùng một transaction. Nếu điện mất giữa chừng, hệ thống phải quay về trạng thái ban đầu chứ tuyệt đối không được để tiền bốc hơi khỏi A mà chưa tới B.
>
> Chính vì phải đảm bảo điều đó, transaction cần một số công đoạn "sổ sách" — và đó là chỗ phát sinh chi phí.

Bây giờ ta bóc từng khoản chi phí mà **mỗi** transaction phải trả:

**Khoản 1 — Network round-trip (một lượt đi và về qua mạng).**
Ứng dụng của bạn gửi câu lệnh tới server database, rồi **đứng chờ** phản hồi. Một lượt đi–về như vậy tốn từ dưới một mili-giây (cùng máy) tới vài chục mili-giây (khác vùng địa lý). Nghe nhỏ, nhưng nhân với 1 triệu:
```
1.000.000 dòng × 1 mili-giây/lượt = 1.000.000 mili-giây ≈ 17 phút
```
**Chỉ riêng thời gian chờ mạng.** Chưa làm gì cả.

**Khoản 2 — Parse và plan (phân tích và lập kế hoạch câu lệnh).**
Mỗi lần nhận một câu SQL, Postgres phải đọc nó, hiểu cú pháp, kiểm tra bảng và cột có tồn tại không, rồi lập kế hoạch thực thi. Một triệu câu lệnh giống hệt nhau nghĩa là làm công việc đó một triệu lần.

**Khoản 3 — WAL (ghi nhật ký an toàn).**
**WAL** viết tắt của *Write-Ahead Log* (nhật ký ghi trước). Đây là cơ chế an toàn cốt lõi của Postgres: **trước khi** thay đổi dữ liệu thật, nó ghi lại ý định thay đổi vào một cuốn nhật ký.
> Analogy: giống như một y tá ghi vào sổ trực "sẽ tiêm thuốc X cho bệnh nhân Y" *trước khi* tiêm. Nếu mất điện giữa chừng, người ta đọc sổ và biết chính xác chuyện gì đang dang dở để xử lý tiếp.
>
> Nhờ WAL mà Postgres phục hồi được sau khi máy sập, và cũng nhờ WAL mà các bản sao dữ liệu ở máy khác biết được có gì thay đổi.

**Khoản 4 — fsync (ép ghi xuống đĩa thật).**
Đây là khoản đắt nhất. Khi bạn `COMMIT` (xác nhận kết thúc transaction), Postgres phải **đảm bảo dữ liệu đã thực sự nằm trên đĩa vật lý**, chứ không chỉ nằm trong bộ nhớ đệm — vì bộ nhớ đệm bốc hơi khi mất điện. Thao tác ép ghi này gọi là **fsync**, và nó chậm vì phải chờ phần cứng xác nhận.

Cộng bốn khoản trên lại, và nhân với một triệu. Đó là lý do vòng lặp ở mục 1.1 chạy hàng giờ.

**Và với vector thì tệ hơn nữa.** Một dòng dữ liệu thông thường (tên, email, ngày tháng) nặng vài chục byte. Một dòng chứa vector 1536 chiều nặng khoảng:
```
1536 con số × 4 byte mỗi số = 6.144 byte ≈ 6 KB
```
Tức là mỗi round-trip phải mang theo một khối 6 KB. Một triệu lượt như vậy là 6 GB dữ liệu, chia thành một triệu gói tin nhỏ — cách truyền dữ liệu kém hiệu quả nhất có thể tưởng tượng ra.

**Vậy giải pháp là gì?** Rất đơn giản, và nó là ý duy nhất của cả bài giảng này:

> **Gom nhiều dòng vào ít transaction và ít round-trip. Trả khoản phí cố định một lần thay vì một triệu lần.**

Đó chính xác là điều mà bài gốc muốn nói khi bảo *"bulk insert giúp duy trì hiệu năng và nạp nhanh hơn"*.

### 1.3. Analogy: chuyển nhà

Hãy dựng phép ví von này cho đầy đủ, vì nó là thứ bạn sẽ dùng để giải thích cho cả sếp lẫn người phỏng vấn.

Bạn cần chuyển 1000 món đồ từ nhà cũ sang nhà mới, cách nhau 2 km.

**Cách 1 — Bê từng món.** Cầm một cái ghế, đi bộ 2 km, đặt xuống, đi bộ 2 km về, cầm món tiếp theo. **1000 chuyến.** Bạn sẽ chuyển nhà trong ba tuần.

**Cách 2 — Dùng xe đẩy.** Chất 50 món lên xe đẩy, đẩy một lần. **20 chuyến.** Nhanh gấp 50 lần.

**Cách 3 — Thuê xe tải.** Chất toàn bộ lên, chạy một chuyến. **1 chuyến.**

Bây giờ là phần quan trọng — **chỉ rõ chỗ khớp** giữa phép ví von và kỹ thuật:

| Trong analogy | Trong kỹ thuật |
|---|---|
| Một món đồ | Một dòng dữ liệu |
| Một chuyến đi | Một transaction / một round-trip |
| Quãng đường 2 km (cố định mỗi chuyến) | **Chi phí cố định** của transaction: mạng, parse, WAL, fsync |
| Bê từng món | `INSERT` từng dòng |
| Xe đẩy 50 món | **Batch insert** (pg-promise, execute_values) |
| Xe tải | **`COPY`** |
| Trọng lượng món đồ | Kích thước dòng (vector nặng hơn dữ liệu thường nhiều) |

Điểm mấu chốt mà analogy này làm sáng tỏ: **tổng khối lượng đồ không đổi trong cả ba cách.** Cái quyết định thời gian là **số chuyến đi**. Đó là lý do tối ưu bulk insert không phải là chuyện "nén dữ liệu cho nhỏ lại" mà là chuyện "giảm số lần đi về".

### 1.4. Ba phương pháp và thứ tự tốc độ — nhớ kỹ thứ tự này

| Phương pháp | Cơ chế | Tốc độ |
|---|---|---|
| `INSERT` từng dòng | 1 dòng / 1 câu lệnh / 1 transaction | 🐢 chậm nhất |
| **Batch / multi-row `INSERT`** | nhiều dòng gộp vào **1 câu lệnh** (pg-promise, execute_values) | 🚗 nhanh hơn nhiều |
| **`COPY`** | giao thức nạp khối chuyên dụng của Postgres | 🚀 **nhanh nhất** |

Giải thích nhanh sự khác nhau giữa hai cách sau, vì người mới hay lẫn:

**Batch insert** vẫn là câu lệnh `INSERT` bình thường, chỉ là nó chứa nhiều bộ giá trị:
```sql
INSERT INTO faqs (text, embedding) VALUES
  ('câu 1', '[...]'),
  ('câu 2', '[...]'),
  ('câu 3', '[...]');     -- 3 dòng, 1 câu lệnh, 1 transaction
```

**COPY** thì **không phải câu lệnh SQL thông thường** — nó là một *giao thức truyền dữ liệu riêng*. Postgres chuyển sang chế độ "đổ dữ liệu", nhận một luồng byte liên tục, và bỏ qua gần hết các bước xử lý dành cho câu lệnh SQL. Chi tiết cơ chế nằm ở mục 3.1.

> **⚠️ [Đính chính bài gốc]** Bài gốc trình bày COPY và pg-promise/execute_values gần như ngang hàng nhau. Thực tế **COPY nhanh nhất, cách biệt rõ rệt**.
>
> Vậy khi nào dùng batch insert thay vì COPY? Khi dữ liệu **đến từ code của bạn** chứ không phải từ file, khi bạn cần logic phức tạp trong lúc chèn (ví dụ `ON CONFLICT` để xử lý trùng lặp — xem mục 4.3), hoặc khi lượng dữ liệu vừa phải và bạn ưu tiên code đơn giản. Đây là một trade-off (sự đánh đổi — được cái này thì mất cái kia) hoàn toàn hợp lệ, miễn là bạn *biết* mình đang đánh đổi.

### 1.5. COPY cơ bản — nạp từ file CSV

**CSV** viết tắt của *Comma-Separated Values* (các giá trị ngăn cách bằng dấu phẩy) — một định dạng file văn bản đơn giản, mỗi dòng là một bản ghi, các cột ngăn nhau bằng dấu phẩy. Excel mở được nó.

Lệnh **`COPY`** chuyển dữ liệu **giữa file và bảng**, theo cả hai chiều. Bài gốc dùng ví dụ file `FAQdata.csv` chứa câu hỏi kèm embedding, nạp vào bảng `faqs`.

```sql
-- Tạo bảng đích trước.
CREATE TABLE faqs (
    id        serial PRIMARY KEY,   -- serial = số tự tăng 1,2,3... mỗi khi thêm dòng.
                                    -- PRIMARY KEY = cột định danh duy nhất cho mỗi dòng.
    text      text,                 -- cột chữ thông thường
    embedding vector(512)           -- cột vector: mỗi ô chứa dãy 512 con số.
                                    -- Số 512 PHẢI khớp số chiều của model bạn dùng.
);

-- Nạp dữ liệu từ CSV vào bảng.
COPY faqs (text, embedding)         -- liệt kê các cột trong file, theo đúng thứ tự
FROM '/path/FAQdata.csv'            -- đường dẫn tới file
WITH (FORMAT csv, HEADER true);     -- FORMAT csv = file kiểu CSV.
                                    -- HEADER true = dòng đầu là tên cột, hãy bỏ qua nó.
```

#### `COPY` và `\copy` — khác biệt nhỏ nhưng hay làm người mới mắc kẹt

Đây là chỗ gây bối rối kinh điển, nên tôi tách riêng ra giải thích.

**`COPY` chạy ở phía server.** Postgres tự mở file bằng chính tay nó. Nghĩa là:
- File phải nằm **trên máy chủ chạy Postgres**, không phải trên laptop của bạn.
- Bạn cần quyền cao (thường là quyền quản trị), vì cho phép database đọc file bất kỳ trên máy chủ là một rủi ro bảo mật.
- Nếu database của bạn nằm trên cloud (RDS, Supabase, Neon), bạn **không có** máy chủ đó để đặt file lên → cách này thường không dùng được.

**`\copy` chạy ở phía client.** Đây là lệnh của chương trình **`psql`** (công cụ dòng lệnh chính thức để gõ SQL vào Postgres), không phải lệnh SQL. `psql` tự đọc file trên máy bạn rồi truyền nội dung qua mạng cho server.
- File nằm **trên máy bạn**. Tiện hơn nhiều.
- **Không cần quyền đặc biệt** nào.
- Dùng được với database trên cloud.

```sql
-- Trong psql. Lưu ý: KHÔNG có dấu chấm phẩy ở cuối, vì đây là lệnh của psql chứ không phải SQL.
\copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)
```

**Quy tắc nhớ gọn:** trong đời thực bạn sẽ dùng `\copy` khoảng 90% số lần. Nhớ dấu gạch chéo ngược.

### 1.6. ⚠️ Cái bẫy vector trong file CSV — hầu như ai cũng vấp một lần

Đây là lỗi mà bạn sẽ gặp trong 15 phút đầu tiên nếu không biết trước, nên tôi nói kỹ.

Vector được viết dưới dạng `[0.12,0.98,0.33]`. Hãy nhìn kỹ: **bên trong nó có dấu phẩy**.

Mà CSV thì dùng dấu phẩy để **ngăn cách các cột**.

Nên nếu bạn viết file thế này:

```csv
text,embedding
thủ đô Anh là London,[0.12,0.98,0.33]
```

thì bộ đọc CSV sẽ hiểu dòng đó có **5 cột**: `thủ đô Anh là London`, `[0.12`, `0.98`, `0.33]`. Trong khi bảng chỉ có 2 cột. Kết quả: lỗi, hoặc tệ hơn là dữ liệu bị nhét sai chỗ.

**Cách sửa: bọc giá trị vector trong dấu nháy kép.** Đó là quy ước chuẩn của CSV để nói "mọi thứ bên trong đây là một giá trị duy nhất, đừng tách".

```csv
text,embedding
"thủ đô Anh là London","[0.12,0.98,0.33]"
"đường sắt Đức là Deutsche Bahn","[0.44,0.03,0.71]"
```

Tôi cũng bọc luôn cột `text` — không bắt buộc nếu văn bản không chứa dấu phẩy, nhưng bọc hết là thói quen an toàn, vì câu chữ tiếng Việt rất hay có dấu phẩy.

**Mẹo thực hành:** đừng tự nối chuỗi CSV bằng tay. Hãy dùng thư viện ghi CSV có sẵn (`csv` trong Python, `csv-stringify` trong Node) — chúng tự xử lý việc bọc nháy và thoát ký tự đặc biệt. Tự nối chuỗi bằng tay là nguồn gốc của phần lớn lỗi loại này.

### 1.7. Chiều ngược lại: xuất bảng ra CSV

`COPY` chạy được cả hai chiều, nên xuất dữ liệu ra cũng dùng đúng lệnh đó, chỉ đổi `FROM` thành `TO`:

```sql
\copy (SELECT text, embedding FROM faqs) TO 'export.csv' WITH (FORMAT csv, HEADER true)
--     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ có thể xuất kết quả của một câu truy vấn bất kỳ,
--                                       không nhất thiết phải là cả bảng
```

Việc này hữu ích hơn bạn tưởng: sao lưu nhanh một tập con dữ liệu, chuyển dữ liệu giữa hai môi trường, hoặc trích một mẫu để phân tích.

### 1.8. 🧩 [Ngoài bài gốc] — Hai điều một Staff Engineer nói ngay từ đầu

**1. Đừng tin con số của tôi. Hãy tự đo.**

Mọi con số về hiệu năng đều phụ thuộc vào phần cứng, mạng, kích thước dòng và cấu hình của bạn. Cách duy nhất để *thực sự* biết là tự đo. Và bài tập này mất 15 phút:

```python
import time

t0 = time.time()
# ... nạp 10.000 dòng bằng cách A ...
print(f"Cách A: {time.time() - t0:.2f} giây")
```

Chạy cùng 10.000 dòng qua ba cách (từng dòng, batch, COPY) và ghi lại ba con số. Khi bạn nhìn thấy chênh lệch bằng số liệu của **chính mình**, kiến thức này sẽ ăn sâu theo cách mà đọc không bao giờ làm được. Nó cũng cho bạn một câu chuyện cụ thể để kể trong phỏng vấn, thứ có sức nặng hơn mọi lý thuyết.

**2. Khoảng cách mạng quan trọng hơn bạn tưởng.**

Nhớ khoản chi phí "round-trip" ở mục 1.2. Nếu ứng dụng của bạn chạy ở Singapore còn database ở Virginia, mỗi lượt đi–về mất khoảng 200 mili-giây. Với 1 triệu round-trip, đó là **hơn hai ngày** chỉ để chờ ánh sáng đi lại trong cáp quang.

Bài học vận hành: **hãy chạy job nạp dữ liệu ở gần database nhất có thể** — cùng vùng, lý tưởng là cùng mạng nội bộ. Đây là loại tối ưu không cần viết dòng code nào mà đôi khi cho hiệu quả lớn hơn mọi thứ khác cộng lại.

### ✅ Self-check Phần 1

Trả lời trong đầu trước, rồi mở gợi ý ra so.

**Câu 1.** Vì sao insert 1 triệu dòng riêng lẻ lại chậm hơn nhiều so với gom thành batch? Kể tên ít nhất ba khoản chi phí.
> *Gợi ý đáp án:* Vì mỗi `INSERT` riêng lẻ là một transaction, và mỗi transaction phải trả một khoản **chi phí cố định** bất kể chứa bao nhiêu dòng: (1) network round-trip — gửi đi rồi đứng chờ phản hồi; (2) parse và plan câu lệnh; (3) ghi WAL; (4) fsync ép dữ liệu xuống đĩa lúc commit. Gom nhiều dòng vào một transaction nghĩa là trả khoản đó một lần thay vì một triệu lần.

**Câu 2.** Xếp thứ tự tốc độ ba phương pháp và nói rõ vì sao COPY thắng.
> *Gợi ý đáp án:* `INSERT` từng dòng ≪ batch INSERT ≪ **COPY**. COPY thắng vì nó không phải là câu lệnh SQL thông thường mà là một *giao thức truyền khối riêng* — Postgres chuyển sang chế độ nhận luồng byte liên tục và bỏ qua gần hết các bước xử lý câu lệnh. (Chi tiết đầy đủ ở mục 3.1.)

**Câu 3.** Trong file CSV, vì sao cột embedding `[0.1,0.2,0.3]` bắt buộc phải bọc nháy kép?
> *Gợi ý đáp án:* Vì bản thân vector chứa dấu phẩy, mà CSV dùng dấu phẩy để ngăn cách cột. Không bọc nháy thì bộ đọc CSV tách mỗi con số thành một cột riêng → sai số cột → lỗi hoặc dữ liệu vào nhầm chỗ.

**Câu 4.** Database của bạn nằm trên Supabase (cloud). Bạn dùng `COPY` hay `\copy`?
> *Gợi ý đáp án:* `\copy`. Vì `COPY` đọc file **trên máy chủ Postgres** — mà máy chủ đó bạn không truy cập được, lại còn cần quyền cao. `\copy` chạy phía client: `psql` đọc file trên máy bạn rồi truyền qua mạng, không cần quyền đặc biệt.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Phần này giả định bạn đã nắm: transaction có chi phí cố định, gom nhiều dòng thì rẻ hơn, và thứ tự tốc độ là từng-dòng ≪ batch ≪ COPY. Nếu còn lấn cấn, hãy quay lại mục 1.2 — mọi thứ dưới đây mọc lên từ đó.
>
> Ở Phần 1 ta nạp từ **file**. Nhưng trong đời thật, dữ liệu thường **đến thẳng từ code** — bạn vừa gọi mô hình AI sinh embedding xong và có sẵn chúng trong bộ nhớ, chẳng có file CSV nào cả. Phần này giải quyết đúng tình huống đó, trong hai ngôn ngữ phổ biến nhất.

### 2.1. Bulk insert trong Node.js với pg-promise

Vài từ cần giải thích trước, dành cho người không làm JavaScript:

- **Node.js** — môi trường chạy JavaScript ở phía máy chủ (thay vì trong trình duyệt). Rất phổ biến để viết backend.
- **npm** — công cụ cài thư viện của Node.js, tương đương `pip` của Python.
- **pg-promise** — một thư viện Node.js để làm việc với PostgreSQL. Cái tên gồm hai phần: `pg` là tên driver Postgres của Node, còn `promise` là kiểu lập trình bất đồng bộ của JavaScript.
- **Promise** / **`async` / `await`** — cách JavaScript xử lý các việc mất thời gian (như gọi database). `await` nghĩa là "chờ việc này xong rồi mới đi tiếp", giúp code bất đồng bộ đọc như code tuần tự bình thường.

Bây giờ tới code, chú thích từng dòng:

```javascript
// Cài đặt: npm install pg-promise

// BƯỚC 1: khởi tạo thư viện.
// Chú ý cặp ngoặc () thứ hai: pg-promise là một "factory" — gọi nó một lần
// để lấy ra đối tượng pgp thực sự dùng được.
const pgp = require('pg-promise')();

// BƯỚC 2: kết nối tới database.
// Chuỗi kết nối có dạng: postgresql://user:mật_khẩu@địa_chỉ/tên_database
const db = pgp('postgresql://user:pass@localhost/db');

// BƯỚC 3: định nghĩa ColumnSet — "khuôn" mô tả bảng và các cột sẽ chèn.
// Vì sao cần khuôn này? Vì pg-promise dùng nó để dựng sẵn cấu trúc câu lệnh MỘT LẦN
// rồi tái sử dụng, thay vì phân tích lại mỗi lần chèn. Với hàng nghìn dòng thì
// việc dựng sẵn này tiết kiệm đáng kể thời gian ở phía Node.
const cs = new pgp.helpers.ColumnSet(
    ['text', 'embedding'],       // danh sách các cột sẽ chèn
    { table: 'faqs' }            // tên bảng đích
);

// BƯỚC 4: chuẩn bị dữ liệu — một mảng các object.
// Mỗi object có key trùng tên với cột trong ColumnSet.
// ⚠️ QUAN TRỌNG: embedding phải được biến thành chuỗi dạng '[0.1,0.2,...]'
//    thì Postgres mới hiểu là kiểu vector. emb1 là một mảng số thường,
//    .join(',') nối chúng lại bằng dấu phẩy, rồi ta thêm hai dấu ngoặc vuông.
const rows = [
  { text: 'thủ đô Anh là London',          embedding: '[' + emb1.join(',') + ']' },
  { text: 'đường sắt Đức là Deutsche Bahn', embedding: '[' + emb2.join(',') + ']' },
  // ... hàng nghìn dòng khác
];

// BƯỚC 5: dựng câu lệnh. helpers.insert gộp TẤT CẢ các dòng vào MỘT câu INSERT:
//   INSERT INTO faqs(text,embedding) VALUES (...),(...),(...),...
const query = pgp.helpers.insert(rows, cs);

// BƯỚC 6: chạy. db.none nghĩa là "chạy câu lệnh này, tôi KHÔNG mong nhận về dòng nào".
// Dùng đúng hàm này (thay vì db.any) giúp pg-promise báo lỗi nếu query lỡ trả về dữ liệu
// — một cách bắt lỗi logic sớm.
await db.none(query);
```

Sáu bước trên đúng theo luồng của bài gốc: cài → khởi tạo → định nghĩa `ColumnSet` → gom dữ liệu → `pgp.helpers.insert` → `db.none`.

**Điểm mấu chốt cần khắc cốt:** `helpers.insert` gộp *nhiều* dòng vào *một* câu lệnh. Đó chính là "xe đẩy 50 món" trong analogy ở mục 1.3 — thay vì nghìn chuyến đi, bạn còn vài chuyến.

> **🧩 [Ngoài bài gốc] Hai điều cần biết thêm ngay:**
>
> **1. Chia lô (chunk) khi dữ liệu rất lớn.** Đừng nhét cả triệu dòng vào một câu `INSERT`. Câu lệnh sẽ khổng lồ, ngốn RAM cả ở Node lẫn ở Postgres, và có thể vượt giới hạn kỹ thuật (chi tiết ở mục 2.4, Lỗi 3). Hãy chia thành các lô 1.000–10.000 dòng rồi lặp qua từng lô.
>
> **2. Muốn nhanh nhất trong Node thì vẫn là COPY.** Thư viện `pg-copy-streams` cho phép dùng giao thức COPY từ Node.js. pg-promise tiện và đủ nhanh cho phần lớn trường hợp, nhưng nếu bạn đang nạp hàng chục triệu dòng thì COPY vẫn là câu trả lời.

### 2.2. Bulk insert trong Python với psycopg2

Đây là cách bài gốc dạy. Vài từ cần giải thích trước:

- **pip** — công cụ cài thư viện của Python.
- **psycopg2** — thư viện Python để nói chuyện với PostgreSQL. Đây là thư viện **thế hệ cũ** (xem đính chính ở mục 2.3).
- **Cursor** (con trỏ) — đối tượng dùng để gửi lệnh SQL và nhận kết quả về. Bạn mở nó từ connection.
- **`register_vector`** — hàm của thư viện `pgvector` dạy cho driver Python biết cách chuyển đổi giữa danh sách số của Python và kiểu `vector` của Postgres.
- **`execute_values`** — hàm tiện ích của psycopg2 để gộp nhiều dòng vào một câu `INSERT`.

```python
# Cài đặt: pip install psycopg2-binary pgvector

import psycopg2
from psycopg2.extras import execute_values      # nằm trong module "extras" (tiện ích mở rộng)
from pgvector.psycopg2 import register_vector

# BƯỚC 1: kết nối
conn = psycopg2.connect("dbname=db user=... password=... host=localhost")

# BƯỚC 2: đăng ký kiểu vector.
# Thiếu dòng này, driver không biết phải chuyển list/numpy array thành cái gì
# -> hoặc báo lỗi, hoặc tệ hơn là lưu sai định dạng.
register_vector(conn)

# BƯỚC 3: chuẩn bị dữ liệu — một list các tuple.
# Mỗi tuple có đúng số phần tử bằng số cột, theo đúng thứ tự trong câu INSERT.
data = [
    ("thủ đô Anh là London",          emb1),   # emb1 là list hoặc numpy array các số thực
    ("đường sắt Đức là Deutsche Bahn", emb2),
    # ... nhiều dòng khác
]

# BƯỚC 4: chèn hàng loạt
with conn.cursor() as cur:          # "with" đảm bảo cursor tự đóng khi xong khối lệnh
    execute_values(
        cur,
        "INSERT INTO faqs (text, embedding) VALUES %s",
        #                                          ^^ CHÚ Ý: chỉ MỘT %s duy nhất.
        #  Đây không phải chỗ điền một giá trị — đây là chỗ execute_values chèn
        #  TOÀN BỘ danh sách dòng vào, thành (...),(...),(...)
        data,
    )

# BƯỚC 5: commit — BẮT BUỘC.
# psycopg2 mặc định KHÔNG tự lưu. Không gọi dòng này thì khi đóng kết nối,
# mọi thứ bị huỷ (rollback) và bạn mất trắng. Xem mục 2.4, Lỗi 1.
conn.commit()          # commit MỘT lần cho cả lô -> đó chính là chỗ tiết kiệm thời gian

conn.close()
```

Luồng này đúng theo bài gốc: cài psycopg2 → import → lấy `execute_values` từ `extras` → truyền khuôn câu lệnh cùng danh sách tuple → connect → cursor → execute → **commit** → close.

Hãy để ý một chi tiết quan trọng: `conn.commit()` được gọi **một lần duy nhất, sau cả lô**. So với vòng lặp ở mục 1.1 gọi commit sau mỗi dòng, đây chính là chỗ bạn tiết kiệm được hàng giờ.

### 2.3. ⚠️ Đính chính quan trọng: psycopg2 đã cũ

`psycopg2` là thư viện **thế hệ cũ**. Bản kế nhiệm tên là **`psycopg`** (phiên bản 3, dân trong nghề gọi là "psycopg3", hiện đang ở nhánh 3.3.x).

Ba điều cần nhớ về sự khác biệt:

1. **psycopg3 KHÔNG có `execute_values`.** Nếu bạn viết `from psycopg.extras import execute_values` thì code sẽ lỗi ngay. Đây là một câu hỏi bẫy rất hay gặp trong phỏng vấn.
2. Thay thế: **`executemany()`** (đã được tối ưu bằng pipeline mode — giải thích ở mục 3.2) hoặc **`cursor.copy()`** (nhanh nhất, xem mục 3.2).
3. Tên gói cài đặt cũng khác: `pip install "psycopg[binary]"` thay vì `pip install psycopg2-binary`. Và module import là `pgvector.psycopg` thay vì `pgvector.psycopg2`.

Code psycopg2 của bài gốc **vẫn chạy tốt** — thư viện này vẫn được bảo trì và vô số hệ thống production đang dùng. Nhưng dự án mới thì nên dùng psycopg3, và khi đi phỏng vấn thì nhất định phải biết sự khác biệt này.

### 2.4. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh

#### Lỗi 1 — Quên `commit()`

Triệu chứng: code chạy không báo lỗi gì, in ra "đã insert 10.000 dòng", nhưng khi bạn `SELECT` thì bảng **rỗng**.

Nguyên nhân: psycopg2 (và psycopg3) mặc định **không autocommit**. Khi bạn thực thi câu lệnh, một transaction được mở ngầm. Nếu bạn đóng kết nối mà chưa `commit()`, Postgres coi transaction đó là chưa hoàn tất và **huỷ bỏ toàn bộ** (rollback).

Đây là hành vi **cố ý và đúng đắn** — nhớ lại định nghĩa transaction ở mục 1.2: "hoặc tất cả, hoặc không gì cả". Nếu chương trình của bạn chết giữa chừng, việc huỷ bỏ phần dở dang chính là điều bạn muốn.

Cách phòng tốt nhất là dùng `with` cho cả connection, để Python tự commit khi khối lệnh kết thúc bình thường và tự rollback khi có lỗi:

```python
with psycopg.connect(dsn) as conn:      # tự commit khi thoát khối này không lỗi
    with conn.cursor() as cur:
        ...
# tới đây đã commit rồi, không cần gọi tay
```

#### Lỗi 2 — Quên đăng ký kiểu, hoặc format vector sai

Triệu chứng: lỗi kiểu dữ liệu khó hiểu, hoặc tệ hơn — dữ liệu vào được nhưng sai, và bạn chỉ phát hiện ra nhiều tuần sau khi kết quả tìm kiếm vô nghĩa.

Nguyên nhân: driver không tự biết một `list` của Python hay một mảng của JavaScript phải được chuyển thành kiểu `vector` của Postgres. Bạn phải nói cho nó biết:

- **Python:** gọi `register_vector(conn)` ngay sau khi kết nối.
- **Node.js:** tự format thành chuỗi `'[' + arr.join(',') + ']'`.

Quy tắc: **đăng ký kiểu hoặc format literal — chọn một, đừng bỏ cả hai.**

#### Lỗi 3 — Một câu lệnh quá khổng lồ

Triệu chứng: chương trình ngốn hết RAM rồi chết, hoặc Postgres trả về lỗi lạ khi bạn nhét quá nhiều dòng vào một câu `INSERT`.

Có một giới hạn kỹ thuật cụ thể mà rất ít người biết, và nói ra nó trong phỏng vấn thì rất ghi điểm: **giao thức của PostgreSQL chỉ cho phép tối đa 65.535 tham số trong một câu lệnh.**

Hãy tính xem điều đó nghĩa là gì với bảng của ta. Bảng `faqs` có 2 cột được chèn (`text`, `embedding`), tức mỗi dòng dùng 2 tham số:

```
65.535 tham số ÷ 2 tham số/dòng ≈ 32.767 dòng tối đa cho một câu lệnh
```

Nếu bảng của bạn có 5 cột thì trần chỉ còn khoảng 13.000 dòng. Vượt qua là lỗi.

**Cách tránh: chia lô.** Kích thước lô 1.000–10.000 dòng là vùng "điểm ngọt" phổ biến — vừa đủ lớn để tiết kiệm round-trip, vừa đủ nhỏ để không ngốn RAM và để một lỗi không phá hỏng cả khối lượng lớn công việc.

```python
BATCH = 5000

for i in range(0, len(data), BATCH):        # cắt data thành từng lô 5000 dòng
    lo = data[i : i + BATCH]
    with conn.cursor() as cur:
        execute_values(cur, "INSERT INTO faqs (text, embedding) VALUES %s", lo)
    conn.commit()                            # commit sau mỗi lô, không phải sau mỗi dòng
    print(f"Đã nạp {i + len(lo)}/{len(data)} dòng")   # in tiến độ: rất đáng giá
                                                      # khi job chạy hàng giờ
```

Dòng in tiến độ cuối cùng nghe có vẻ thừa, nhưng khi bạn ngồi nhìn một job chạy 3 tiếng mà không biết nó đang ở đâu, bạn sẽ hiểu vì sao nó đáng giá.

### ✅ Self-check Phần 2

**Câu 1.** Trong pg-promise, `ColumnSet` và `pgp.helpers.insert` mỗi cái làm gì?
> *Gợi ý đáp án:* `ColumnSet` là "khuôn" mô tả bảng đích và các cột sẽ chèn — pg-promise dựng sẵn cấu trúc câu lệnh từ khuôn này một lần rồi tái dùng, thay vì phân tích lại mỗi lần. `helpers.insert` nhận mảng dữ liệu cùng khuôn đó và dựng ra **một** câu `INSERT` chứa nhiều bộ giá trị.

**Câu 2.** Nếu quên `conn.commit()` trong psycopg2 thì chuyện gì xảy ra với dữ liệu?
> *Gợi ý đáp án:* Mất trắng. psycopg2 không autocommit; transaction đang mở sẽ bị **rollback** khi đóng kết nối. Code không báo lỗi gì cả, nên đây là lỗi rất khó phát hiện. Cách phòng: dùng `with psycopg.connect(...) as conn:` để commit/rollback tự động.

**Câu 3.** `execute_values` có còn trong psycopg3 không? Nếu không thì thay bằng gì?
> *Gợi ý đáp án:* **Không.** `execute_values` là hàm của psycopg2 (`psycopg2.extras`). Trong psycopg3 hãy dùng `executemany()` (đã tối ưu bằng pipeline mode) hoặc `cursor.copy()` (nhanh nhất, khuyến nghị cho lượng lớn).

**Câu 4.** Bảng của bạn có 4 cột được chèn. Tối đa bao nhiêu dòng cho một câu `INSERT`?
> *Gợi ý đáp án:* Giao thức Postgres giới hạn 65.535 tham số cho một câu lệnh → `65.535 ÷ 4 ≈ 16.383` dòng. Vượt qua là lỗi. Trong thực tế nên chia lô 1.000–10.000 dòng, thấp hơn trần này khá xa, để tiết kiệm RAM và giới hạn thiệt hại khi có lỗi.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Ở đây ta chui xuống bên dưới bề mặt: COPY thực sự làm gì mà nhanh đến thế, và cái nguyên tắc vận hành mà bài gốc bỏ sót nhưng lại quyết định phần lớn tốc độ của cả job nạp dữ liệu.

### 3.1. Vì sao COPY nhanh nhất — mổ xẻ dưới nắp capo

Ở mục 1.4 tôi mới nói COPY "là một giao thức riêng". Giờ ta xem cụ thể nó bỏ qua được những gì.

Hãy nhớ lại bốn khoản chi phí ở mục 1.2, và xem COPY xử lý từng khoản ra sao.

#### Lý do 1 — Bỏ qua vòng đời xử lý câu lệnh

Với mỗi câu SQL bình thường, Postgres phải chạy một quy trình bốn bước:

```
Parse (phân tích cú pháp)  →  Analyze (kiểm tra bảng/cột có thật không)
                           →  Plan (lập kế hoạch thực thi)
                           →  Execute (chạy)
```

Ba bước đầu là công việc "hành chính". Với một câu lệnh chèn 5.000 dòng thì chi phí đó chia đều cho 5.000 dòng nên không đáng kể. Nhưng với 5.000 câu lệnh riêng lẻ thì bạn trả nó 5.000 lần.

COPY chỉ trả **một lần duy nhất** cho toàn bộ luồng dữ liệu. Sau khi khởi động, Postgres chuyển sang chế độ "đổ dữ liệu" và chỉ còn việc đọc byte rồi ghi vào bảng.

#### Lý do 2 — Giao thức binary thay vì text

Mặc định, dữ liệu được truyền dưới dạng **text** (văn bản). Nghĩa là số thực `0.123456` được gửi đi thành **8 ký tự**, rồi phía server phải *phân tích chuỗi đó ngược lại* thành số.

Với vector, hãy tính xem điều đó tốn thế nào. Một vector 1536 chiều:

| Cách truyền | Kích thước | Việc server phải làm |
|---|---|---|
| **Text** | ~1536 × 9 ký tự ≈ **14 KB** | phân tích 1536 chuỗi thành 1536 số thực |
| **Binary** (`FORMAT BINARY`) | 1536 × 4 byte = **6 KB** | không phải phân tích gì cả, copy thẳng byte |

Vừa nhẹ hơn **hơn hai lần** trên đường truyền, vừa bỏ được toàn bộ công phân tích chuỗi. Với dữ liệu vector — vốn gần như chỉ toàn số — lợi ích này lớn hơn hẳn so với dữ liệu thông thường.

Đây là lý do vì sao ở mục 3.2 tôi dùng `WITH (FORMAT BINARY)`.

#### Lý do 3 — Dùng vùng đệm riêng, không làm ô nhiễm bộ nhớ đệm chung

Postgres có một vùng bộ nhớ đệm chung tên **shared buffers**, giữ những trang dữ liệu đang được dùng nhiều để khỏi phải đọc đĩa. Đây là tài nguyên quý.

Nếu một job nạp hàng triệu dòng chiếm hết vùng đệm này, nó sẽ **đẩy văng** dữ liệu nóng mà những người dùng khác đang cần — và mọi truy vấn khác trên hệ thống bỗng chậm đi. Hiện tượng này gọi là *cache pollution* (ô nhiễm bộ nhớ đệm).

COPY dùng một **ring buffer** (vùng đệm vòng) nhỏ riêng cho việc nạp, nên nó không phá bộ nhớ đệm chung. Đây là chi tiết ít người biết, và nó có ý nghĩa thực tế lớn: **COPY không chỉ nhanh hơn, nó còn "lịch sự" hơn với phần còn lại của hệ thống.**

#### Lý do 4 — Vẫn an toàn tuyệt đối

Một hiểu nhầm phổ biến: "nhanh thế chắc nó bỏ qua khâu an toàn nào đó".

**Không.** COPY vẫn ghi WAL đầy đủ. Nghĩa là:
- Dữ liệu vẫn phục hồi được nếu máy sập.
- Các bản sao (replica) ở máy khác vẫn nhận được thay đổi.
- Point-in-time recovery (khôi phục về một thời điểm trong quá khứ) vẫn hoạt động.

COPY nhanh vì nó **bỏ qua các bước thừa**, không phải vì nó bỏ qua các bước an toàn. Đây là một điểm rất đáng nói trong phỏng vấn, vì nó cho thấy bạn phân biệt được "tối ưu" với "đánh đổi độ bền".

### 3.2. psycopg3 `copy()` — cách nhanh nhất hiện đại

```python
# Cài đặt: pip install "psycopg[binary]" pgvector

import psycopg
from pgvector.psycopg import register_vector      # chú ý: pgvector.psycopg, KHÔNG phải .psycopg2

conn = psycopg.connect("dbname=db user=... host=localhost")
register_vector(conn)

# rows có thể là list, NHƯNG cũng có thể là một generator (hàm sinh dữ liệu dần dần).
# Đây là điểm mạnh lớn nhất, giải thích ngay bên dưới.
rows = [("thủ đô Anh là London", emb1), ("đường sắt Đức", emb2)]

with conn.cursor() as cur:
    # Mở một phiên COPY. Câu lệnh dùng "FROM STDIN" nghĩa là
    # "dữ liệu sẽ được đẩy vào từ phía client qua kết nối này".
    with cur.copy("COPY faqs (text, embedding) FROM STDIN WITH (FORMAT BINARY)") as copy:

        # Khai báo kiểu của từng cột. Bắt buộc khi dùng FORMAT BINARY, vì ở chế độ nhị phân
        # không có cách nào đoán kiểu từ nội dung — server cần được nói trước.
        copy.set_types(["text", "vector"])

        for text, emb in rows:
            copy.write_row([text, emb])    # mỗi dòng được stream THẲNG vào server,
                                           # không tích trong bộ nhớ chờ gom lô

conn.commit()
```

**Ba ưu điểm của cách này, theo thứ tự quan trọng:**

**1. Nhanh nhất.** Vì nó chính là COPY, với đủ bốn lý do ở mục 3.1, cộng thêm binary.

**2. Streaming lười (lazy streaming) — ưu điểm bị đánh giá thấp nhất.** Hãy so hai cách:

- Với `execute_values`, bạn phải **gom toàn bộ dữ liệu vào một list trong RAM trước**. Một triệu vector 1536 chiều là khoảng 6 GB trong bộ nhớ Python. Nhiều máy sẽ chết ngay tại đây.
- Với `copy.write_row()`, mỗi dòng được đẩy đi ngay khi sinh ra. Bạn có thể truyền vào một **generator** đọc dữ liệu dần từ file hoặc sinh embedding theo lô, và **bộ nhớ tiêu thụ gần như không đổi dù có 1 triệu hay 100 triệu dòng**.

Đây là khác biệt giữa một script chạy được trên laptop và một script chỉ chạy được trên máy chủ 64 GB RAM.

**3. Format binary** — như đã phân tích ở mục 3.1, đặc biệt có lợi với dữ liệu toàn số như vector.

Đây là **cách được khuyến nghị cho mọi job nạp dữ liệu lớn ở thời điểm 2026.**

#### 🧩 [Ngoài bài gốc] `executemany()` của psycopg3 thực chất làm gì?

Nếu bạn không muốn dùng COPY, psycopg3 có `executemany()`:

```python
cur.executemany("INSERT INTO faqs (text, embedding) VALUES (%s, %s)", rows)
```

Nhiều người tưởng nó gộp các dòng thành một câu `INSERT` nhiều giá trị giống `execute_values`. **Không phải.** Nó vẫn gửi từng câu `INSERT` riêng — nhưng bằng **pipeline mode**.

**Pipeline mode** (chế độ đường ống) nghĩa là: client gửi liên tiếp nhiều câu lệnh đi mà **không đứng chờ phản hồi của từng cái**, rồi mới nhận tất cả kết quả về sau.

> Analogy: gọi mười món ở quán ăn. Cách cũ là gọi món 1, chờ bưng ra, ăn xong mới gọi món 2. Pipeline là đọc một lượt cả mười món cho phục vụ rồi ngồi chờ.
>
> **Chỗ khớp với kỹ thuật:** thứ được tiết kiệm ở đây là **khoản chi phí round-trip** (khoản 1 ở mục 1.2) — thứ thường chiếm phần lớn thời gian. Các khoản khác (parse, WAL) vẫn phải trả từng lần.

Nên thứ tự tốc độ trong psycopg3 là: `executemany()` **nhanh hơn nhiều** so với vòng lặp `execute()` từng dòng, nhưng **vẫn chậm hơn** `copy()`. Đội phát triển psycopg đã bỏ `execute_values` chính vì lý do này: `executemany()` đã đủ nhanh cho trường hợp thường, còn khi cần nhanh thật thì `copy()` thắng tuyệt đối.

### 3.3. ⭐ [NGUYÊN TẮC VÀNG] Nạp dữ liệu TRƯỚC, tạo index SAU

Đây là điều bài gốc bỏ qua hoàn toàn, nhưng nó thường quyết định tốc độ nhiều hơn cả việc bạn chọn COPY hay batch insert.

#### Index là gì và vì sao nó làm chậm việc ghi

**Index** (chỉ mục) là một cấu trúc dữ liệu phụ, dựng thêm bên cạnh bảng, giúp tìm kiếm nhanh hơn.

> Analogy: mục lục tra cứu ở cuối một cuốn sách. Muốn tìm chủ đề "transaction", bạn tra mục lục thấy "trang 301" rồi nhảy thẳng tới, thay vì đọc từ trang 1.
>
> **Chỗ khớp quan trọng cho bài này:** mỗi khi bạn **sửa nội dung sách**, mục lục cũng **phải được cập nhật theo**. Và đó chính là cái giá — mỗi dòng bạn chèn vào bảng, database phải cập nhật mọi index liên quan.

Với index vector, cái giá đó **cao hơn hẳn** so với index thông thường:

- **HNSW** (*Hierarchical Navigable Small World*) xây dựng một mạng lưới nhiều tầng nối các vector gần nhau. Mỗi lần chèn một vector mới, nó phải **đi tìm hàng xóm gần nhất trong mạng lưới** rồi nối vào — bản thân việc đó đã là một cuộc tìm kiếm. Tham số `ef_construction` càng cao (chất lượng index càng tốt) thì mỗi lần chèn càng chậm.
- **IVFFlat** chia dữ liệu thành các cụm bằng thuật toán k-means. Nó **bắt buộc phải có dữ liệu sẵn** để tính được tâm các cụm — nên với IVFFlat, chuyện "nạp trước index sau" không phải lời khuyên mà là **yêu cầu bắt buộc**. Build index trên bảng rỗng thì các cụm được tính trên hư không, và chất lượng tìm kiếm sau đó sẽ rất tệ mà không có lỗi nào báo.

#### Cách làm đúng

```sql
-- BƯỚC 1: Tạo bảng KHÔNG có index vector.
CREATE TABLE faqs (
    id        serial PRIMARY KEY,
    text      text,
    embedding vector(512)
);

-- BƯỚC 2: Nạp toàn bộ dữ liệu. Nhanh, vì không phải vá index sau mỗi dòng.
\copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)

-- BƯỚC 3: SAU KHI nạp xong mới build index — một lần duy nhất.
SET maintenance_work_mem = '2GB';    -- cho phép Postgres dùng nhiều RAM hơn khi build index.
                                     -- Mặc định rất thấp (thường 64MB), và với HNSW
                                     -- thì thiếu RAM ở bước này làm build chậm thảm hại.
                                     -- Quy tắc: đặt cao nhưng ≤ 50-60% RAM của máy.

CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
--                                          ^^^^^^^^^^^^^^^^ "ops class" — phải khớp với
--                    toán tử khoảng cách bạn dùng lúc truy vấn (vector_cosine_ops ↔ <=>).
--                    Sai chỗ này thì Postgres âm thầm bỏ qua index.
```

**Vì sao build một lần lại nhanh hơn nhiều so với vá dần?** Vì khi có sẵn toàn bộ dữ liệu, Postgres tổ chức được công việc hiệu quả hơn hẳn: đọc dữ liệu tuần tự, sắp xếp theo lô, và chạy song song trên nhiều nhân CPU. Vá từng dòng thì mỗi lần là một thao tác ngẫu nhiên nhỏ lẻ, không tận dụng được gì.

> **Câu chốt đáng nhớ:** *"Đừng bắt database vừa nhận hàng vừa sắp kho. Đổ hết hàng vào đã, rồi sắp một lượt."*

### 3.4. Chọn batch size và cách bọc transaction

**Batch size (kích thước lô)** — vùng "điểm ngọt" phổ biến là **1.000 đến 10.000 dòng** một lô. Lý do:

| Lô quá nhỏ (ví dụ 10) | Lô quá lớn (ví dụ 500.000) |
|---|---|
| Quá nhiều round-trip → chậm | Ngốn RAM cả hai phía |
| Chi phí cố định vẫn bị trả nhiều lần | Có thể vượt giới hạn 65.535 tham số |
| | Một lỗi làm hỏng cả khối công việc lớn |
| | Không thấy được tiến độ, khó chẩn đoán |

Với vector (dòng nặng ~6 KB), hãy nghiêng về phía **thấp** của khoảng đó — 1.000 đến 2.000 dòng một lô là hợp lý, vì một lô 5.000 dòng đã là 30 MB.

**Bọc transaction:** bọc mỗi lô trong một transaction (`BEGIN ... COMMIT`). Cách này cho bạn điều tốt nhất của cả hai phía — ít commit hơn hẳn so với từng dòng, nhưng nếu lô thứ 700 lỗi thì 699 lô trước vẫn an toàn, bạn chỉ cần chạy lại từ đó.

### 3.5. Edge cases — các trường hợp biên phải biết

**Edge case** (trường hợp biên) là tình huống nằm ở rìa điều kiện bình thường: ít gặp, nhưng khi gặp thì làm hệ thống hỏng theo cách bất ngờ.

**1. Vector trong CSV phải bọc nháy kép** (đã nói ở mục 1.6, nhắc lại vì nó là lỗi phổ biến nhất).

**2. Sai số chiều (dimension mismatch).** Một dòng có vector 384 chiều lọt vào lô nạp cho cột `vector(512)` → **cả lệnh COPY thất bại**, không phải chỉ dòng đó. COPY mang tính "được ăn cả, ngã về không" trong phạm vi một lệnh.

Cách phòng — kiểm tra trước khi nạp, chứ đừng để database phát hiện hộ bạn:
```python
DIM = 512
xau = [i for i, (_, emb) in enumerate(data) if len(emb) != DIM]
if xau:
    raise ValueError(f"{len(xau)} dòng sai số chiều, ví dụ ở vị trí {xau[:5]}")
```
Ba dòng code này tiết kiệm cho bạn hàng giờ mò mẫm khi một job nạp 3 tiếng chết ở phút thứ 170.

**3. Embedding bị `NULL`.** Nếu cột cho phép rỗng, dòng thiếu embedding vẫn vào được bảng — nhưng nó sẽ **không bao giờ xuất hiện** trong kết quả tìm kiếm vector. Nó vô hình. Nên có kiểm tra định kỳ:
```sql
SELECT count(*) FROM faqs WHERE embedding IS NULL;
```
Con số này lớn hơn 0 nghĩa là bạn có một mảng dữ liệu mà người dùng không bao giờ tìm thấy.

**4. COPY không hỗ trợ xử lý trùng lặp.** Không có `ON CONFLICT` cho COPY. Nếu chạy lại job, bạn sẽ nhân đôi dữ liệu. Cách giải quyết chuẩn là mẫu "COPY vào bảng tạm rồi merge" — trình bày đầy đủ ở mục 4.3.

**5. Thứ tự cột phải khớp.** Trong `COPY faqs (text, embedding)`, các cột trong file phải theo **đúng thứ tự đó**. Đảo hai cột thì hoặc lỗi kiểu, hoặc — nếu hai cột cùng kiểu — dữ liệu vào nhầm chỗ mà không báo gì.

### ✅ Self-check Phần 3

**Câu 1.** Nêu ít nhất hai lý do COPY nhanh hơn batch INSERT.
> *Gợi ý đáp án:* (1) Chỉ trả chi phí parse/analyze/plan **một lần** cho cả luồng, thay vì mỗi câu lệnh một lần. (2) Truyền được ở **định dạng binary**, vừa nhẹ hơn ~2 lần với vector vừa bỏ được công phân tích chuỗi thành số ở phía server. (3) Dùng **ring buffer riêng**, không đẩy văng dữ liệu nóng khỏi shared buffers. Và quan trọng: nó **vẫn ghi WAL**, nên không hy sinh độ an toàn.

**Câu 2.** Vì sao nên nạp dữ liệu trước rồi mới tạo index HNSW? Với IVFFlat thì khác gì?
> *Gợi ý đáp án:* Với HNSW, mỗi lần chèn phải đi tìm hàng xóm trong mạng lưới rồi nối vào — rất tốn; build một lần trên dữ liệu có sẵn thì Postgres đọc tuần tự, sắp lô và chạy song song được, nhanh hơn nhiều. Với IVFFlat thì đây **không phải lời khuyên mà là bắt buộc**: nó cần dữ liệu sẵn để k-means tính tâm cụm; build trên bảng rỗng cho phân cụm vô nghĩa và recall tệ mà không báo lỗi.

**Câu 3.** `copy()` của psycopg3 có ưu điểm gì về bộ nhớ so với gom cả list rồi gọi `execute_values`?
> *Gợi ý đáp án:* `copy.write_row()` **stream từng dòng thẳng vào server**, nên bộ nhớ tiêu thụ gần như không đổi dù có 1 triệu hay 100 triệu dòng — và nhận được cả generator. `execute_values` bắt bạn gom toàn bộ vào RAM trước; một triệu vector 1536 chiều là khoảng 6 GB, đủ để giết tiến trình Python trên phần lớn máy.

**Câu 4.** `executemany()` của psycopg3 gộp các dòng thành một câu `INSERT` phải không?
> *Gợi ý đáp án:* **Không.** Nó vẫn gửi từng câu `INSERT` riêng, nhưng qua **pipeline mode** — gửi liên tiếp mà không đứng chờ phản hồi từng cái. Nó tiết kiệm chi phí round-trip nhưng vẫn phải trả chi phí parse cho từng câu, nên chậm hơn `copy()`.

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Đây là phần phân biệt senior với staff. Senior trả lời *"dùng COPY hay batch insert"*. Staff hỏi *"trong cả pipeline này thì nút thắt thật nằm ở đâu, job chạy ba tiếng mà đứt ở phút 170 thì sao, và việc nạp này có làm sập hệ thống đang phục vụ khách hàng không"*.
>
> Vài từ vựng nghề nghiệp cần giải thích trước:
> - **Staff Engineer** — cấp bậc kỹ sư cao hơn senior. Đặc trưng không phải code giỏi hơn mà là **phạm vi ảnh hưởng**: quyết định kiến trúc, đánh đổi dài hạn, ảnh hưởng nhiều team.
> - **Production** — môi trường chạy thật, phục vụ người dùng thật.
> - **Pipeline** — một chuỗi các bước xử lý nối tiếp nhau, đầu ra của bước này là đầu vào của bước sau.
> - **Job** — một tác vụ chạy nền, thường kéo dài, không phải phản hồi trực tiếp cho người dùng.

### 4.1. Toàn cảnh pipeline nạp dữ liệu — và nút thắt thật nằm ở đâu

Bulk insert không đứng một mình. Nó là **một mắt xích** trong chuỗi dưới đây. Khi nạp hàng triệu tài liệu vào một hệ thống tìm kiếm ngữ nghĩa, toàn cảnh là:

```
1. CHUNK    Cắt tài liệu dài thành đoạn ngắn vừa với giới hạn của model
                ↓
2. EMBED    Gọi mô hình AI sinh embedding, theo LÔ 32–256 đoạn mỗi lần
                ↓
3. LOAD     COPY (binary) vào bảng CHƯA có index vector
                ↓
4. INDEX    Build HNSW SAU khi nạp xong, với maintenance_work_mem cao
                ↓
5. VERIFY   Kiểm chứng: đếm dòng, EXPLAIN ANALYZE, đo recall
```

Giải thích hai bước lạ:

- **Chunk** (cắt đoạn) — mô hình embedding có giới hạn độ dài đầu vào. Tài liệu 50 trang phải được cắt thành các đoạn nhỏ, nếu không model sẽ cắt cụt và bạn mất phần lớn nội dung mà không hề biết.
- **Verify** (kiểm chứng) — bước mà ai cũng bỏ qua và ai cũng hối hận. Chi tiết ở cuối mục 4.6. Trong đó **recall** là thước đo chất lượng tìm kiếm: trong `k` kết quả gần nhất *thật sự*, hệ thống tìm được bao nhiêu phần trăm. Index vector đánh đổi một phần recall để lấy tốc độ, nên sau khi build index xong bạn cần đo xem mình đang ở 95% hay 70%.

Bây giờ tới câu hỏi quan trọng nhất của cả Phần 4:

> **Nút thắt thật của pipeline này nằm ở bước nào?**

Câu trả lời làm nhiều người bất ngờ: **bước 2 — sinh embedding**, không phải bước 3 — insert.

Hãy tính. Sinh embedding cho 1 triệu đoạn văn bằng API bên ngoài, mỗi lô 256 đoạn mất khoảng 1 giây:
```
1.000.000 ÷ 256 ≈ 3.900 lô × 1 giây ≈ 65 phút
```
Trong khi COPY 1 triệu dòng chỉ mất **vài phút**.

**Vậy vì sao vẫn phải học bulk insert?** Vì nếu bạn insert từng dòng, chính bước 3 sẽ trở thành nút thắt **thứ hai** — và một nút thắt hàng giờ chồng lên một nút thắt hàng giờ thì bạn có một job chạy qua đêm. COPY không làm pipeline nhanh hơn nút thắt chính, nhưng nó **đảm bảo database không trở thành nút thắt**.

> **Đây là câu trả lời "đắt" nhất của bài này trong phỏng vấn:** *"Nút thắt thật trong ingestion vector thường là khâu sinh embedding chứ không phải khâu insert — nhưng một cách insert ngây thơ sẽ tạo ra nút thắt thứ hai."* Nói được câu này chứng tỏ bạn nhìn cả hệ thống chứ không chỉ nhìn đoạn code trước mắt.

Và hệ quả thực tế: **tối ưu bước 2 trước.** Cụ thể là gọi API theo lô lớn, chạy song song nhiều **worker** (tiến trình chạy nền, mỗi cái xử lý một phần dữ liệu cùng lúc), và **cache embedding theo hash của nội dung** — để khi phải chạy lại job, bạn không trả tiền API lần thứ hai cho những đoạn văn không đổi.

### 4.2. Nạp lại vào bảng đã có index: drop rồi rebuild

Mục 3.3 nói về bảng mới. Nhưng còn tình huống thường gặp hơn: bảng **đã có** index và đang phục vụ người dùng, giờ bạn cần nạp thêm vài triệu dòng.

Có ba chiến lược, chọn theo ràng buộc:

**Chiến lược 1 — Drop rồi rebuild (nhanh nhất, nhưng mất tìm kiếm tạm thời).**
```sql
DROP INDEX faqs_embedding_idx;
-- ... bulk load bằng COPY ...
CREATE INDEX faqs_embedding_idx ON faqs USING hnsw (embedding vector_cosine_ops);
```
Khi lượng nạp thêm lớn so với dữ liệu hiện có, cách này rẻ hơn nhiều so với việc bắt index tự vá suốt quá trình. Cái giá: trong lúc đó tìm kiếm vector rơi về quét tuần tự (chậm) hoặc bị lỗi tuỳ ứng dụng.

**Chiến lược 2 — Build lại mà không khoá (giữ được dịch vụ).**
```sql
CREATE INDEX CONCURRENTLY faqs_embedding_idx
ON faqs USING hnsw (embedding vector_cosine_ops);
```
**`CONCURRENTLY`** cho phép build index mà **không chặn thao tác ghi** lên bảng. Đánh đổi: chậm hơn (Postgres phải quét bảng hai lượt) và có thể thất bại giữa chừng, để lại một index hỏng cần dọn tay (`DROP INDEX` rồi làm lại). Nhưng trên production, cách thông thường sẽ **chặn mọi `INSERT`/`UPDATE` suốt hàng giờ** — nên `CONCURRENTLY` gần như luôn là lựa chọn đúng.

**Chiến lược 3 — Bảng bóng rồi hoán đổi (an toàn nhất).**
Nạp vào một bảng mới hoàn toàn (`faqs_new`), build index xong xuôi, kiểm chứng đầy đủ, rồi đổi tên trong một transaction để hoán đổi hai bảng. Người dùng không cảm nhận được gì ngoài một khoảnh khắc rất ngắn. Cái giá: cần gấp đôi dung lượng đĩa tạm thời.

**Tăng tốc build:** tăng `max_parallel_maintenance_workers` để Postgres build index trên nhiều nhân CPU cùng lúc. Kết hợp với `maintenance_work_mem` cao, đây là hai núm vặn có tác dụng lớn nhất.

### 4.3. Idempotency và upsert — làm cho job chạy lại được an toàn

**Idempotency** (tính lũy đẳng) nghĩa là: **chạy một thao tác nhiều lần cho ra kết quả giống hệt như chạy một lần.**

> Analogy: bấm nút "khoá cửa" trên chìa khoá ô tô. Bấm một lần hay năm lần thì kết quả đều là cửa đã khoá. Ngược lại, "thêm 100 nghìn vào tài khoản" **không** lũy đẳng — bấm năm lần là thành 500 nghìn.

Vì sao điều này tối quan trọng ở đây? Vì một job nạp 3 tiếng **sẽ** đứt giữa chừng vào một ngày nào đó — mất mạng, hết bộ nhớ, deploy nhầm, ai đó rút nhầm dây. Nếu job không lũy đẳng, chạy lại nghĩa là dữ liệu bị nhân đôi, và bạn phải dọn dẹp bằng tay trong hoảng loạn.

Vấn đề: **COPY không hỗ trợ `ON CONFLICT`** (mệnh đề xử lý trùng lặp của Postgres). Vậy làm sao?

**Mẫu chuẩn: COPY vào bảng tạm, rồi merge sang bảng thật.**

```sql
-- BƯỚC 1: tạo bảng tạm có cấu trúc giống bảng thật.
-- TEMP = bảng tạm, tự động biến mất khi kết thúc phiên làm việc.
-- LIKE faqs INCLUDING DEFAULTS = sao chép cấu trúc cột và giá trị mặc định.
CREATE TEMP TABLE faqs_stage (LIKE faqs INCLUDING DEFAULTS);

-- BƯỚC 2: COPY vào bảng tạm. Nhanh, và không có ràng buộc nào cản trở.
\copy faqs_stage (doc_id, text, embedding) FROM 'batch.csv' WITH (FORMAT csv)

-- BƯỚC 3: merge sang bảng thật, xử lý trùng lặp.
INSERT INTO faqs (doc_id, text, embedding)
SELECT doc_id, text, embedding FROM faqs_stage
ON CONFLICT (doc_id) DO UPDATE                      -- nếu doc_id đã tồn tại thì...
    SET embedding = EXCLUDED.embedding,             -- ...cập nhật thay vì tạo dòng mới.
        text      = EXCLUDED.text;
    -- EXCLUDED là bảng ảo chứa dòng ĐANG ĐỊNH CHÈN — dùng để lấy giá trị mới.
```

Ba điểm cần chú ý để mẫu này thực sự hoạt động:

1. **Cần một khoá nghiệp vụ ổn định** để nhận diện dòng trùng — ở đây là `doc_id`, một mã định danh có ý nghĩa từ hệ thống nguồn. **Đừng dùng cột `id` tự tăng**, vì nó là số mới mỗi lần chèn, không nhận diện được gì cả. Cột đó cũng cần có ràng buộc `UNIQUE` thì `ON CONFLICT` mới dùng được.
2. Bảng tạm **không cần index**, nên COPY vào đó rất nhanh.
3. Bước merge chạy hoàn toàn **bên trong database**, không có dữ liệu nào đi qua mạng lần nữa.

> **Câu chốt đáng nhớ:** *"COPY vào bảng tạm rồi upsert — đó là cách bạn biến một job nạp dữ liệu nhiều giờ thành một job chạy lại được an toàn."*

### 4.4. WAL, replication, chi phí và giám sát

#### WAL và áp lực lên bản sao

Nhớ lại mục 1.2: WAL là nhật ký ghi trước, và **COPY vẫn ghi WAL đầy đủ**. Điều đó tốt cho độ an toàn, nhưng sinh ra một hệ quả vận hành mà người mới không lường được:

**Nạp khối lượng lớn tạo ra khối lượng WAL khổng lồ.** Với 100 GB dữ liệu nạp vào, bạn có thể sinh ra hơn 100 GB WAL trong thời gian ngắn. Hai rủi ro:

1. **Đầy đĩa.** WAL tích tụ nhanh hơn tốc độ dọn dẹp.
2. **Replica lag (bản sao bị trễ).** **Replica** là một máy chủ giữ bản sao dữ liệu, thường dùng để chia tải đọc hoặc dự phòng. Nó cập nhật bằng cách đọc lại WAL từ máy chính. Nếu bạn đổ WAL vào nhanh hơn tốc độ replica tiêu hoá, nó sẽ **tụt lại phía sau** — có thể hàng phút hoặc hàng giờ. Nếu ứng dụng của bạn đọc từ replica, người dùng sẽ thấy dữ liệu cũ. Nếu máy chính sập lúc đó, bạn mất phần chênh lệch.

**Cách xử lý:** giám sát độ trễ replica **trong lúc** nạp, và **giãn tốc độ** (thêm khoảng nghỉ giữa các lô) nếu độ trễ tăng. Việc nạp nhanh hơn 20% không đáng để đánh đổi lấy một replica tụt lại 2 tiếng.

#### Bảng `UNLOGGED` — công cụ sắc nhưng nguy hiểm

```sql
CREATE UNLOGGED TABLE faqs_stage (...);
```

Bảng **`UNLOGGED`** **không ghi WAL**. Nạp nhanh hơn đáng kể. Nhưng cái giá rất nặng:

- **Mất sạch dữ liệu khi máy chủ sập** (Postgres tự động xoá sạch bảng đó khi khởi động lại sau sự cố).
- **Không được replicate** sang các bản sao.

**Quy tắc dùng:** chỉ dùng cho bảng **staging** (bảng trung chuyển) mà bạn có thể tạo lại từ nguồn. Tuyệt đối không dùng cho bảng chứa dữ liệu thật. Đây là một trade-off hoàn toàn hợp lệ **miễn là bạn nói rõ nó ra** — và trong phỏng vấn, nói kèm giới hạn của nó là điều ghi điểm.

#### Chi phí

Chi phí thật của một job ingestion vector nằm ở **sinh embedding** (tiền API hoặc thời gian GPU), không phải ở việc insert. Nhưng insert kém hiệu quả kéo dài thời gian chạy → tốn tiền thuê máy → và tốn thứ đắt nhất là **thời gian của kỹ sư ngồi chờ**.

Đòn bẩy chi phí lớn nhất: **cache embedding theo hash nội dung**. Khi chạy lại job, những đoạn văn không thay đổi sẽ lấy embedding từ cache thay vì gọi API lần nữa. Với một job nạp lại toàn bộ, việc này có thể cắt gần hết hoá đơn.

#### Giám sát — đo gì trong lúc nạp

| Chỉ số | Vì sao quan trọng |
|---|---|
| **Số dòng mỗi giây** | Ước tính được còn bao lâu nữa xong, và phát hiện khi tốc độ tụt bất thường |
| **Thời gian build index** | Thường là phần dài thứ hai; cần biết để lên lịch |
| **Độ trễ replica** | Cảnh báo sớm rằng bạn đang đổ WAL quá nhanh |
| **Tỷ lệ dòng lỗi** | Sai số chiều, embedding NULL — phát hiện sớm thay vì phút thứ 170 |
| **RAM khi build index** | Thiếu là build chậm thảm hại |

### 4.5. Ảnh hưởng tổ chức & cách nói với người không kỹ thuật

Một staff engineer phải giải thích được quyết định kỹ thuật cho **stakeholder** (các bên liên quan — quản lý sản phẩm, sếp, tài chính) bằng ngôn ngữ của *họ*: **thời gian, chi phí, và lịch vận hành**.

**Ví dụ cách nói với PM hoặc sếp:**

> *"Việc nạp dữ liệu ban đầu — khoảng 5 triệu tài liệu — là một tác vụ chạy một lần nhưng tốn thời gian. Phần lâu nhất không phải là lưu vào cơ sở dữ liệu, mà là để AI đọc và sinh ra biểu diễn cho từng tài liệu. Dùng đúng kỹ thuật thì thời gian rút từ khoảng hai ngày xuống còn vài giờ. Tôi sẽ xếp lịch chạy ngoài giờ cao điểm để khách hàng không bị ảnh hưởng, và thiết kế sao cho nếu job đứt giữa chừng thì chạy lại được mà không hỏng dữ liệu."*

Chú ý: đoạn trên không có chữ COPY, không có WAL, không có HNSW. Nó nói bằng ba thứ người nghe quan tâm — **thời gian, ảnh hưởng tới khách hàng, và rủi ro**. Và nó chủ động nêu kế hoạch cho tình huống xấu, điều xây dựng lòng tin tốt hơn mọi lời hứa.

**Ảnh hưởng tới roadmap:** pipeline nạp dữ liệu (chunk → embed → COPY → index → verify) **không phải script dùng một lần rồi bỏ**. Bạn sẽ cần nó lại mỗi khi: thêm nguồn dữ liệu mới, đổi mô hình embedding (phải sinh lại toàn bộ), hoặc khôi phục sau sự cố.

Hãy xây nó thành một **job hoặc service có retry và idempotency ngay từ đầu**, đừng viết script tạm rồi hứa sẽ dọn sau. Cái giá của việc làm đúng ngay lần đầu nhỏ hơn nhiều so với cái giá của việc chạy lại một script không lũy đẳng lúc 2 giờ sáng.

**Ảnh hưởng tới team:** vì toàn bộ việc này nằm trong Postgres — công cụ mà đội backend đã biết — nên không cần thêm hệ thống mới, không cần tuyển kỹ năng mới, không tạo ra "ốc đảo tri thức" mà chỉ một người trong công ty hiểu.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff

> **Đề bài:** *"Bạn cần nạp 10 triệu tài liệu (mỗi cái khoảng 500 **token** — đơn vị đo độ dài văn bản mà mô hình AI dùng, một token xấp xỉ ba phần tư một từ tiếng Anh) vào pgvector cho tìm kiếm ngữ nghĩa. Yêu cầu: nhanh nhất có thể, không làm gián đoạn dịch vụ đang chạy, và job phải chạy lại được nếu đứt giữa chừng."*

**Khung trả lời — 8 bước, và nhớ nói to các trade-off:**

**1. Làm rõ đề bài trước khi vẽ gì cả.** Hỏi: dữ liệu nguồn ở đâu (file, database khác, hay luồng streaming)? Sinh embedding bằng API bên ngoài hay mô hình chạy tại chỗ? Có được phép có downtime không, hay tuyệt đối không? Ngân sách bao nhiêu? Số chiều vector?
> *Vì sao quan trọng:* staff engineer không thiết kế cho đề bài tưởng tượng. Đây là bước ứng viên hay bỏ qua rồi mất điểm ngay lập tức.

**2. Chỉ ra nút thắt thật ngay từ đầu.** Nói thẳng: *"Nút thắt sẽ là khâu sinh embedding, không phải khâu insert."* Rồi tối ưu nó: gọi API theo lô 128–256, chạy song song nhiều worker, và cache theo hash nội dung để lần chạy lại không phải trả tiền lần nữa.
> *Đây là bước ghi điểm sớm nhất.* Nó cho thấy bạn nhìn cả hệ thống thay vì lao vào tối ưu phần mình vừa học.

**3. Nạp bằng COPY binary vào bảng chưa có index vector.** Chia thành các chunk để một lỗi không phá hỏng toàn bộ. Dùng `copy.write_row()` của psycopg3 với generator để bộ nhớ không phình theo dữ liệu.

**4. Đảm bảo idempotency.** COPY vào bảng staging → `INSERT ... SELECT ... ON CONFLICT (doc_id) DO UPDATE`. Cần một khoá nghiệp vụ ổn định (`doc_id`), không phải id tự tăng.

**5. Build index SAU khi nạp xong.** `CREATE INDEX CONCURRENTLY ... USING hnsw ...` với `maintenance_work_mem` cao và `max_parallel_maintenance_workers` tăng lên. `CONCURRENTLY` là thứ đáp ứng yêu cầu "không gián đoạn dịch vụ" trong đề bài.

**6. Bảo vệ dịch vụ đang chạy.** Hai lựa chọn: dùng `CONCURRENTLY`, hoặc nạp vào bảng bóng rồi hoán đổi. Song song đó, **giám sát độ trễ replica và giãn tốc nếu nó tăng** — nêu được ý này là dấu hiệu bạn từng vận hành thật.

**7. Kiểm chứng — đừng bỏ bước này.** Ba việc:
```sql
SELECT count(*) FROM faqs;                          -- đúng số dòng mong đợi?
SELECT count(*) FROM faqs WHERE embedding IS NULL;  -- có dòng nào vô hình không?
EXPLAIN ANALYZE SELECT ... ORDER BY embedding <=> '[...]' LIMIT 10;
                                    -- có thấy "Index Scan using ..._hnsw..." không?
```
Nếu `EXPLAIN ANALYZE` cho thấy `Seq Scan` thay vì `Index Scan`, index của bạn đang bị bỏ qua — thường là do ops class không khớp toán tử truy vấn.

**8. Tự đánh giá lại to thành tiếng.** Kết bằng: *"Nút thắt là khâu sinh embedding chứ không phải insert; COPY đảm bảo database không trở thành nút thắt thứ hai; idempotency cho phép chạy lại an toàn; và `CONCURRENTLY` giữ cho dịch vụ không gián đoạn."*
> **Biết đâu là nút thắt thật, và nói được vì sao mỗi lựa chọn giải quyết đúng một ràng buộc trong đề bài — đó là tư duy staff.**

### ✅ Self-check Phần 4

**Câu 1.** Trong pipeline nạp vector, nút thắt thật thường nằm ở đâu? Vậy vì sao vẫn phải học bulk insert?
> *Gợi ý đáp án:* Nút thắt thật thường là **khâu sinh embedding** (gọi API hoặc chạy GPU), không phải khâu insert. Nhưng nếu insert từng dòng, chính nó sẽ thành nút thắt **thứ hai** — hai nút thắt hàng giờ chồng lên nhau thành một job qua đêm. COPY không làm pipeline nhanh hơn nút thắt chính, nhưng đảm bảo database không tạo thêm nút thắt.

**Câu 2.** Vì sao COPY không dùng được `ON CONFLICT`, và mẫu thay thế là gì?
> *Gợi ý đáp án:* COPY là giao thức nạp khối, không phải câu lệnh `INSERT`, nên không có mệnh đề xử lý trùng lặp. Mẫu chuẩn: **COPY vào bảng tạm → `INSERT ... SELECT ... ON CONFLICT (khoá) DO UPDATE`** sang bảng thật. Cần một khoá nghiệp vụ ổn định có ràng buộc `UNIQUE`, không phải id tự tăng.

**Câu 3.** Bulk load có ảnh hưởng gì tới replication? Xử lý thế nào?
> *Gợi ý đáp án:* COPY vẫn ghi WAL đầy đủ, nên nạp lớn sinh WAL rất nhiều → nguy cơ đầy đĩa và **replica tụt lại phía sau** (người dùng đọc từ replica thấy dữ liệu cũ; nếu máy chính sập thì mất phần chênh). Xử lý: giám sát độ trễ replica trong lúc nạp và giãn tốc độ khi nó tăng.

**Câu 4.** Khi nào dùng bảng `UNLOGGED`, và cái giá là gì?
> *Gợi ý đáp án:* Chỉ dùng cho bảng **staging** mà bạn tạo lại được từ nguồn. Nó bỏ ghi WAL nên nạp nhanh hơn, nhưng **mất sạch dữ liệu khi máy chủ sập** và **không được replicate**. Tuyệt đối không dùng cho dữ liệu thật.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn. Nó cố tình viết ngắn và cô đặc — ngược hẳn với bốn phần trên. Nếu một dòng nào đọc mà thấy mơ hồ, hãy quay lại đúng mục tương ứng đọc lại cho chậm.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ tiếng Anh | Định nghĩa một dòng |
|---|---|
| **Bulk / batch insert** | Nạp nhiều dòng trong ít transaction; nhanh hơn hẳn từng dòng. |
| **Transaction** | Nhóm thao tác "hoặc tất cả cùng thành công, hoặc không gì cả". |
| **Transaction overhead** | Chi phí cố định mỗi transaction: round-trip, parse, WAL, fsync. |
| **Round-trip** | Một lượt gửi đi và chờ phản hồi qua mạng; nhân với số dòng là ra thời gian chờ. |
| **WAL** | *Write-Ahead Log* — nhật ký ghi trước, nền tảng của phục hồi và replication. |
| **fsync** | Ép dữ liệu xuống đĩa vật lý lúc commit; khoản chi phí đắt nhất. |
| **`COPY` / `\copy`** | Giao thức nạp khối nhanh nhất; `COPY` phía server, `\copy` phía client. |
| **`FORMAT BINARY`** | COPY ở dạng nhị phân: nhẹ hơn ~2 lần với vector và bỏ được công parse. |
| **Shared buffers / ring buffer** | Bộ nhớ đệm chung; COPY dùng vùng đệm riêng nên không làm ô nhiễm nó. |
| **pg-promise** | Thư viện Node.js; `ColumnSet` + `helpers.insert` + `db.none` để batch. |
| **psycopg2 / `execute_values`** | Thư viện Python **cũ** + hàm gộp nhiều dòng vào một `INSERT`. |
| **psycopg (v3) / `copy()` / `executemany()`** | Thư viện Python **mới**; `copy()` nhanh nhất; **không còn `execute_values`**. |
| **Pipeline mode** | Gửi liên tiếp nhiều lệnh mà không chờ phản hồi từng cái; cơ chế của `executemany()`. |
| **`register_vector`** | Đăng ký kiểu để list/ndarray chuyển được thành `vector`. |
| **`commit()`** | Bắt buộc để lưu; psycopg mặc định không autocommit. |
| **Load-then-index** | Nạp dữ liệu trước, tạo index sau — nguyên tắc vàng của bulk load. |
| **`maintenance_work_mem`** | RAM cho việc build index; mặc định quá thấp cho HNSW. |
| **`max_parallel_maintenance_workers`** | Số tiến trình build index song song. |
| **`ON CONFLICT DO UPDATE`** | Upsert — nền tảng của idempotency. |
| **Idempotency** | Chạy nhiều lần cho kết quả như chạy một lần; điều kiện để job retry được. |
| **`CREATE INDEX CONCURRENTLY`** | Build index không khoá thao tác ghi; bắt buộc trên production. |
| **`UNLOGGED` table** | Bỏ WAL, nạp nhanh — nhưng mất khi sập và không replicate; chỉ dùng staging. |
| **Replica lag** | Bản sao tụt lại vì WAL đổ về quá nhanh; rủi ro chính khi bulk load. |
| **65.535 parameters** | Giới hạn tham số cho một câu lệnh Postgres → trần số dòng mỗi lô. |

### 5.2. Core concepts — nếu chỉ được nhớ mười điều

1. Mỗi `INSERT` riêng lẻ là một transaction có **chi phí cố định** (round-trip, parse, WAL, fsync); làm một triệu lần thì trả một triệu lần.
2. Thứ tự tốc độ: **từng dòng ≪ batch insert ≪ COPY**. Dùng COPY khi có thể.
3. COPY nhanh vì: parse một lần, giao thức binary, ring buffer riêng — **nhưng vẫn ghi WAL**, không hy sinh độ an toàn.
4. **Nạp dữ liệu TRƯỚC, build index SAU** — nguyên tắc vàng. Với IVFFlat thì đây là bắt buộc, không phải lời khuyên.
5. Node.js → **pg-promise** (`ColumnSet` + `helpers.insert`); Python → psycopg2 `execute_values` (cũ) hoặc **psycopg3 `copy()`** (mới, nhanh nhất).
6. **psycopg3 không còn `execute_values`**; dùng `executemany()` (pipeline mode) hoặc `copy()`.
7. Luôn **`register_vector`** (hoặc format `'[...]'`) và luôn nhớ **`commit()`**.
8. Vector trong CSV **phải bọc nháy kép**, vì bản thân nó chứa dấu phẩy.
9. Idempotency: **COPY → bảng tạm → `ON CONFLICT DO UPDATE`**, dùng khoá nghiệp vụ chứ không phải id tự tăng.
10. **Nút thắt thật thường là sinh embedding, không phải insert** — nhưng insert ngây thơ tạo ra nút thắt thứ hai.

### 5.3. Mental model / analogy để nói cho trôi chảy

- **"Xe tải, xe đẩy, hay bê tay"** → ba phương pháp; thứ quyết định là **số chuyến**, không phải khối lượng.
- **"Nạp trước, index sau"** → đừng bắt database vừa nhận hàng vừa sắp kho.
- **"COPY vào bảng tạm rồi upsert"** → mẫu chuẩn cho ingestion chạy lại được.
- **"Nút thắt là embedding, không phải insert"** → câu thể hiện tầm nhìn hệ thống.
- **"Không commit là mất trắng"** → psycopg không autocommit.
- **"Mục lục phải sửa theo mỗi lần sửa sách"** → vì sao index làm chậm việc ghi.

### 5.4. Code cần thuộc lòng

**(a) COPY — nhanh nhất:**
```sql
\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)
```

**(b) Node.js với pg-promise:**
```javascript
const cs = new pgp.helpers.ColumnSet(['text','embedding'], { table: 'faqs' });
await db.none(pgp.helpers.insert(rows, cs));   // 1 câu INSERT chứa nhiều dòng
```

**(c) Python — cũ và mới:**
```python
# psycopg2 (bài gốc):
from psycopg2.extras import execute_values
execute_values(cur, "INSERT INTO faqs (text,embedding) VALUES %s", data)
conn.commit()

# psycopg3 (nhanh nhất, khuyến nghị):
with cur.copy("COPY faqs (text,embedding) FROM STDIN WITH (FORMAT BINARY)") as cp:
    cp.set_types(["text", "vector"])
    for t, e in rows:
        cp.write_row([t, e])
conn.commit()
```

**(d) Nạp trước, index sau:**
```sql
-- COPY toàn bộ dữ liệu xong rồi mới:
SET maintenance_work_mem = '2GB';
CREATE INDEX CONCURRENTLY ON faqs USING hnsw (embedding vector_cosine_ops);
```

**(e) Idempotent upsert:**
```sql
CREATE TEMP TABLE faqs_stage (LIKE faqs INCLUDING DEFAULTS);
\copy faqs_stage (doc_id, text, embedding) FROM 'batch.csv' WITH (FORMAT csv)

INSERT INTO faqs (doc_id, text, embedding)
SELECT doc_id, text, embedding FROM faqs_stage
ON CONFLICT (doc_id) DO UPDATE SET embedding = EXCLUDED.embedding;
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Vì sao bulk insert nhanh hơn insert từng dòng?"**
> Mỗi `INSERT` riêng lẻ là một transaction với chi phí cố định: network round-trip, parse/plan câu lệnh, ghi WAL, và fsync lúc commit. Gom nhiều dòng vào ít transaction nghĩa là trả khoản đó một lần thay vì một triệu lần. Với vector còn tệ hơn vì mỗi dòng nặng ~6 KB.

**2. [XẾP HẠNG] "COPY, batch INSERT, từng dòng — cái nào nhanh nhất và vì sao?"**
> COPY > batch INSERT > từng dòng. COPY thắng vì nó là giao thức nạp khối riêng: chỉ parse một lần cho cả luồng, truyền được ở dạng binary (nhẹ hơn ~2 lần với vector và không phải parse chuỗi thành số), và dùng ring buffer riêng nên không đẩy văng dữ liệu nóng khỏi shared buffers. Quan trọng: nó vẫn ghi WAL nên không hy sinh độ bền.

**3. [BẪY] "Trong psycopg3 bạn dùng `execute_values` thế nào?"**
> Đây là câu bẫy: psycopg3 **không có** `execute_values` — đó là hàm của psycopg2 (`psycopg2.extras`). Trong psycopg3 dùng `executemany()` (chạy qua pipeline mode) hoặc `cursor.copy()` (nhanh nhất).

**4. [BẪY] "Vì sao build index trước rồi mới bulk insert lại chậm?"**
> Vì mỗi dòng chèn vào đều buộc database cập nhật index. Với HNSW, mỗi lần chèn là một cuộc tìm kiếm hàng xóm trong đồ thị rồi nối cạnh — rất tốn, và càng tốn khi `ef_construction` cao. Build một lần sau khi có đủ dữ liệu thì Postgres đọc tuần tự, sắp lô và chạy song song được.

**5. [BẪY] "Insert xong mà truy vấn không thấy dữ liệu, lỗi ở đâu?"**
> Thiếu `conn.commit()`. psycopg mặc định không autocommit, nên transaction đang mở bị rollback khi đóng kết nối — và không có thông báo lỗi nào cả. Cách phòng: dùng `with psycopg.connect(...) as conn:`.

**6. [STAFF] "Nạp 10 triệu vector nhanh nhất bằng cách nào?"**
> Nói nút thắt trước: khâu sinh embedding, không phải insert — nên batch embedding và cache theo hash. Rồi: COPY binary vào bảng **chưa index**, chia chunk; `CREATE INDEX CONCURRENTLY` sau với `maintenance_work_mem` cao và parallel workers; idempotent qua bảng tạm + upsert; giám sát replica lag.

**7. "Bulk insert có làm hỏng replication hoặc khả năng phục hồi không?"**
> Không, nếu dùng bảng thường — COPY vẫn ghi WAL đầy đủ. Nhưng nạp lớn sinh WAL rất nhiều nên cần theo dõi **replica lag** và dung lượng đĩa, giãn tốc nếu cần. Bảng `UNLOGGED` thì bỏ WAL (nhanh hơn) nhưng mất dữ liệu khi sập và không replicate — chỉ dùng cho staging.

**8. "Làm sao để job nạp chạy lại được an toàn sau khi đứt?"**
> Idempotency. Vì COPY không có `ON CONFLICT`, dùng mẫu: COPY vào bảng tạm → `INSERT ... SELECT ... ON CONFLICT (doc_id) DO UPDATE`. Cần một khoá nghiệp vụ ổn định có ràng buộc `UNIQUE` — không dùng id tự tăng vì nó không nhận diện được dòng cũ.

**9. "Batch size bao nhiêu là hợp lý?"**
> Thường 1.000–10.000 dòng; với vector (dòng ~6 KB) nên nghiêng về phía thấp, 1.000–2.000. Quá nhỏ thì vẫn tốn nhiều round-trip; quá lớn thì ngốn RAM, có thể vượt giới hạn **65.535 tham số** cho một câu lệnh, và một lỗi phá hỏng cả khối công việc lớn.

**10. "Bạn kiểm chứng thế nào sau khi nạp xong?"**
> Ba việc: đếm số dòng khớp kỳ vọng; đếm `WHERE embedding IS NULL` (những dòng đó vô hình với tìm kiếm vector); và `EXPLAIN ANALYZE` một truy vấn thật để xác nhận thấy `Index Scan using ..._hnsw...` chứ không phải `Seq Scan`.

### 5.6. One-liner đắt giá — thả đúng lúc để ghi điểm

- *"Each INSERT is a transaction; bulk loading is about paying that overhead once instead of a million times."*
- *"COPY is the fastest way to get rows into Postgres — fewer parses, a binary protocol, and its own buffer — and it still writes WAL, so you give up nothing in durability."*
- *"Load first, index second — never make the database maintain an HNSW graph while you're pouring millions of rows into it."*
- *"psycopg3 dropped execute_values because optimized executemany and copy() already win — reach for copy() at scale."*
- *"COPY into a temp table, then upsert — that's how you make a multi-hour ingest job safe to re-run."*
- *"The real bottleneck in vector ingestion is usually embedding generation, not the insert — but a naive insert makes it the second one."*
- *"Bulk loading is a WAL firehose; if you're not watching replica lag while you pour, you'll find out the hard way."*

---

## 📌 Ghi chú cuối — làm gì tiếp theo

**1. Bốn điểm cần nhớ đúng khi đi phỏng vấn:**
- **COPY nhanh nhất**, không ngang hàng với batch insert.
- **psycopg3 thay psycopg2** và **không có `execute_values`** — dùng `copy()` hoặc `executemany()`.
- **Nạp trước, index sau** — bắt buộc với IVFFlat, rất nên với HNSW.
- Nhớ **`register_vector`** và nhớ **`commit()`**.

**2. Thực hành — bài tập đo đạc 30 phút, đáng giá hơn cả buổi đọc.**

Dựng Postgres kèm pgvector bằng **Docker** (công cụ chạy phần mềm trong "hộp" đóng gói sẵn):
```bash
docker run --name pgvec -e POSTGRES_PASSWORD=pass -p 5432:5432 -d pgvector/pgvector:pg17
```

Rồi làm đúng chuỗi việc này:
1. Tạo bảng `faqs(text, embedding vector(384))`, **chưa có index**.
2. Sinh khoảng 100.000 embedding thật (dùng `sentence-transformers` với model `all-MiniLM-L6-v2`, chạy miễn phí trên máy).
3. Nạp cùng lượng dữ liệu đó bằng **cả ba cách** — từng dòng, `execute_values`/pg-promise, và COPY — và **bấm giờ từng cách**.
4. So ba con số. Đây là lúc kiến thức trở thành trực giác.
5. Rồi mới `CREATE INDEX ... USING hnsw ...` và chạy `EXPLAIN ANALYZE` để thấy `Index Scan`.
6. Bonus: thử build index **trước** rồi nạp, và đo lại — để tự thấy nguyên tắc vàng ở mục 3.3 đắt tới mức nào.

Bước 3 và bước 6 là hai bước quan trọng nhất. Con số của chính bạn có sức thuyết phục hơn mọi bảng benchmark trên mạng, và nó cho bạn một câu chuyện cụ thể để kể trong phỏng vấn.

**3. Kiểm chứng tài liệu khi ôn:** đọc `psycopg.org` (phần v3, mục `copy()`), README của `pgvector` cho ví dụ bulk load hiện hành, và docs của pg-promise cho `ColumnSet`/`helpers.insert`. Các thư viện này thay đổi nhanh.

**4. Nối lại mạch của cả series** (tất cả nằm trong một Postgres duy nhất):
> embed (tạo vector) → index (làm search nhanh) → cài đặt pgvector → store & query → **bulk insert (bài này)** → full-text search → hybrid.
>
> Tới đây bạn đã nạp được dữ liệu quy mô lớn vào một hệ thống tìm kiếm ngữ nghĩa hoặc RAG một cách hiệu quả và an toàn.

**5. Chủ đề nên học tiếp, theo thứ tự:**
- **Streaming ingestion** — nạp liên tục theo thời gian thực (Kafka → embed → COPY) thay vì nạp một lần.
- **Incremental re-embedding** — khi đổi mô hình embedding, làm sao sinh lại toàn bộ mà không ngừng dịch vụ (version hoá cột, backfill dần, blue-green).
- **Chunking strategies** — cách cắt tài liệu dài trước khi embed; ảnh hưởng tới chất lượng tìm kiếm còn nhiều hơn cả việc chọn index.
- **Tối ưu build HNSW song song** ở quy mô hàng chục triệu vector.
- **Quantization** — nén vector (`halfvec`, binary) để cắt RAM và chi phí lưu trữ.
