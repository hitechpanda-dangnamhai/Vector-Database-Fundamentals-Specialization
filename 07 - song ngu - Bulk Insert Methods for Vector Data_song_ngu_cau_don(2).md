# Bulk Insert — chèn dữ liệu hàng loạt cho vector trong pgvector: COPY, pg-promise, psycopg — Giáo trình Basic → Staff
*Bulk Insert for Vector Data in pgvector: COPY, pg-promise, psycopg — Basic to Staff Course*

> **Nguồn gốc**  
> *Source*
>
> Nguồn là video *"Techniques for bulk data transfer, including pg-promise"*.  
> *The source is the video titled "Techniques for bulk data transfer, including pg-promise".*
>
> Video thuộc Course 3 của IBM Vector Database Fundamentals.  
> *The video belongs to Course 3 of IBM Vector Database Fundamentals.*
>
> Bài gốc giải thích tầm quan trọng của bulk insert.  
> *The original lesson explains the importance of bulk insert.*
>
> Bài gốc trình bày lệnh **COPY**.  
> *The original lesson presents the **COPY** command.*
>
> Bài gốc trình bày bulk insert trong Node.js bằng **pg-promise**.  
> *The original lesson presents bulk insert in Node.js with **pg-promise**.*
>
> Bài gốc trình bày bulk insert trong Python bằng **psycopg2**.  
> *The original lesson presents bulk insert in Python with **psycopg2**.*
>
> Bài gốc sử dụng hàm `execute_values`.  
> *The original lesson uses the `execute_values` function.*
>
> Tài liệu này bám sát bài gốc.  
> *This course follows the original lesson closely.*
>
> Tài liệu này mở rộng thành pipeline ingestion — quy trình nạp dữ liệu thực tế.  
> *This course expands into a real ingestion pipeline.*
>
> Nhãn **[MỞ RỘNG NGOÀI BÀI GỐC]** đánh dấu phần mở rộng.  
> *The **[EXPANSION BEYOND THE ORIGINAL LESSON]** label marks expanded content.*
>
> **Vị trí trong series**  
> *Position in the series*
>
> Đây là mảnh *"nạp dữ liệu vào NHANH"*.  
> *This is the "load data FAST" lesson.*
>
> Bài này nối tiếp bài hands-on — thực hành cài đặt số 6.  
> *This lesson follows hands-on installation lesson number 6.*
>
> Bạn đã cài đặt xong.  
> *You have completed the installation.*
>
> Bạn đã tạo bảng xong.  
> *You have created the table.*
>
> Bây giờ, bạn cần nạp hàng triệu vector.  
> *Now you need to load millions of vectors.*
>
> Quá trình này không được làm sập performance — hiệu năng.  
> *This process must not destroy performance.*
>
> **⚠️ Đính chính và cập nhật tới 2026**  
> *⚠️ Corrections and updates through 2026*
>
> Bài gốc có một số nội dung đã cũ.  
> *The original lesson contains some outdated content.*
>
> Bài gốc có một số nội dung chưa được sắp xếp đúng.  
> *The original lesson contains some poorly ordered content.*
>
> 1. **COPY là cách bulk insert nhanh nhất.**  
>    ***COPY is the fastest bulk insert method.***
>
> COPY không chỉ là một lựa chọn cho CSV.  
> *COPY is not merely an option for CSV.*
>
> Thứ tự đầu tiên là `row-by-row INSERT`.  
> *The first method is `row-by-row INSERT`.*
>
> Thứ tự tiếp theo là `batch/multi-row INSERT`.  
> *The next method is `batch/multi-row INSERT`.*
>
> Phương pháp này gồm pg-promise và execute_values.  
> *This method includes pg-promise and execute_values.*
>
> **`COPY`** đứng cuối thứ tự về thời gian.  
> ***`COPY`** stands last in elapsed time.*
>
> **`COPY`** đứng đầu thứ tự về tốc độ.  
> ***`COPY`** stands first in speed.*
>
> Bài gốc trình bày COPY ngang hàng với các phương pháp khác.  
> *The original lesson presents COPY as an equal method.*
>
> Phần này cần được nói rõ lại.  
> *This point needs clarification.*
>
> 2. **`psycopg2` là một thư viện cũ.**  
>    ***`psycopg2` is an old library.***
>
> Bản kế nhiệm là `psycopg` v3.  
> *Its successor is `psycopg` v3.*
>
> Tên gọi khác của nó là psycopg3.  
> *Its other name is psycopg3.*
>
> Psycopg3 không còn `execute_values`.  
> *Psycopg3 no longer includes `execute_values`.*
>
> Psycopg3 dùng `executemany()`.  
> *Psycopg3 uses `executemany()`.*
>
> Hàm này đã tối ưu pipeline.  
> *This function has an optimized pipeline.*
>
> Psycopg3 cũng dùng `cursor.copy()`.  
> *Psycopg3 also uses `cursor.copy()`.*
>
> `cursor.copy()` là lựa chọn nhanh nhất.  
> *`cursor.copy()` is the fastest option.*
>
> Bài gốc vẫn dạy psycopg2.  
> *The original lesson still teaches psycopg2.*
>
> Code đó vẫn chạy.  
> *That code still works.*
>
> Bạn vẫn nên biết phiên bản mới.  
> *You should still know the newer version.*
>
> 3. **Bài gốc bỏ qua nguyên tắc vàng của bulk load.**  
>    ***The original lesson omits the golden rule of bulk loading.***
>
> Bạn cần nạp toàn bộ dữ liệu trước.  
> *You need to load all data first.*
>
> Bạn cần tạo index — chỉ mục sau.  
> *You need to create the index afterward.*
>
> Index có thể là HNSW hoặc IVFFlat.  
> *The index can be HNSW or IVFFlat.*
>
> Việc duy trì index trong lúc bulk insert rất chậm.  
> *Index maintenance during bulk insert is very slow.*
>
> 4. **Vector cần được đăng ký kiểu.**  
>    ***Vectors require type registration.***
>
> Bạn có thể dùng `register_vector`.  
> *You can use `register_vector`.*
>
> Bạn cũng có thể dùng format `'[1,2,3]'`.  
> *You can also use the `'[1,2,3]'` format.*
>
> Khi đó, driver có thể serialize vector.  
> *The driver can then serialize the vector.*
>
> Quy tắc này áp dụng cho bulk insert và COPY.  
> *This rule applies to bulk insert and COPY.*

---

## Phần 0: 🗺️ Bản đồ bài học
*Part 0: 🗺️ Lesson Overview*

**Bài này dạy gì**  
*What this lesson teaches*

Bài này dạy cách nạp lượng lớn embedding — biểu diễn số vào PostgreSQL/pgvector.  
*This lesson teaches large-scale embedding loading into PostgreSQL/pgvector.*

Cách nạp này có hiệu quả cao.  
*This loading method is highly efficient.*

Cách này thay thế insert từng dòng.  
*This method replaces row-by-row insertion.*

Insert từng dòng chậm khủng khiếp.  
*Row-by-row insertion is extremely slow.*

Bài này trình bày ba kỹ thuật.  
*This lesson presents three techniques.*

Kỹ thuật đầu tiên là **COPY**.  
*The first technique is **COPY**.*

COPY là kỹ thuật nhanh nhất.  
*COPY is the fastest technique.*

COPY có thể đọc từ file hoặc stream — luồng dữ liệu.  
*COPY can read from a file or a stream.*

Kỹ thuật thứ hai là **batch insert — chèn theo lô** trong Node.js.  
*The second technique is **batch insert** in Node.js.*

Node.js sử dụng **pg-promise**.  
*Node.js uses **pg-promise**.*

Kỹ thuật thứ ba là batch insert trong Python.  
*The third technique is batch insert in Python.*

Python sử dụng **psycopg2** hoặc **psycopg3**.  
*Python uses **psycopg2** or **psycopg3**.*

Bài này cũng trình bày một nguyên tắc vận hành.  
*This lesson also presents an operational principle.*

Bạn nạp dữ liệu trước.  
*You load the data first.*

Bạn tạo index sau.  
*You create the index afterward.*

**Vấn đề nó giải quyết**  
*The problem it solves*

Mỗi câu `INSERT` là một transaction — giao dịch.  
*Each `INSERT` statement is one transaction.*

Mỗi transaction có overhead — chi phí phụ cố định.  
*Each transaction has fixed overhead.*

Overhead gồm WAL, commit và khóa.  
*The overhead includes WAL, commit, and locking.*

Một triệu vector riêng lẻ tạo một triệu transaction.  
*One million separate vectors create one million transactions.*

Quá trình đó cực chậm.  
*That process is extremely slow.*

Vector nhiều chiều làm vấn đề nghiêm trọng hơn.  
*High-dimensional vectors make the problem worse.*

Mỗi dòng có thể chiếm vài KB.  
*Each row can occupy several KB.*

Bulk insert gom nhiều dòng vào ít transaction.  
*Bulk insert groups many rows into few transactions.*

Cách này có thể nhanh hơn hàng chục lần.  
*This method can be tens of times faster.*

Cách này cũng có thể nhanh hơn hàng trăm lần.  
*This method can also be hundreds of times faster.*

**Học xong bạn sẽ làm được**  
*What you can do after this lesson*

- Bạn có thể giải thích lợi ích của bulk insert.  
  *You can explain the benefit of bulk insert.*

- Bạn có thể giải thích transaction overhead.  
  *You can explain transaction overhead.*

- Bạn có thể xếp đúng thứ tự tốc độ.  
  *You can rank the methods by speed.*

- Thứ tự là row-by-row, batch và COPY.  
  *The order is row-by-row, batch, and COPY.*

- Bạn có thể chọn phương pháp phù hợp.  
  *You can choose the suitable method.*

- Bạn có thể dùng COPY để nạp vector.  
  *You can use COPY to load vectors.*

- COPY có thể đọc CSV.  
  *COPY can read CSV.*

- COPY cũng có thể đọc stream.  
  *COPY can also read a stream.*

- Bạn có thể bulk insert từ Node.js.  
  *You can perform bulk insert from Node.js.*

- Node.js có thể dùng pg-promise.  
  *Node.js can use pg-promise.*

- Bạn có thể bulk insert từ Python.  
  *You can perform bulk insert from Python.*

- Python có thể dùng psycopg2 `execute_values`.  
  *Python can use psycopg2 `execute_values`.*

- Python có thể dùng psycopg3 `copy`.  
  *Python can use psycopg3 `copy`.*

- Bạn có thể thiết kế pipeline ingestion.  
  *You can design an ingestion pipeline.*

- Pipeline gồm nạp, build index và verify — xác minh.  
  *The pipeline includes loading, index building, and verification.*

- Pipeline có thể xử lý hàng triệu vector.  
  *The pipeline can process millions of vectors.*

**Mạch Basic → Staff**  
*Basic to Staff progression*

- 🟢 **Basic:** Bạn tìm hiểu nguyên nhân one-by-one chậm.  
  *🟢 **Basic:** You learn why one-by-one loading is slow.*

- Bạn tìm hiểu ba phương pháp.  
  *You learn three methods.*

- Bạn tìm hiểu thứ tự tốc độ.  
  *You learn the speed ranking.*

- Bạn thực hành COPY cơ bản từ CSV vào table — bảng.  
  *You practice basic COPY from CSV into a table.*

- 🟡 **Intermediate:** Bạn tìm hiểu COPY sâu hơn.  
  *🟡 **Intermediate:** You study COPY in greater depth.*

- Bạn tìm hiểu pg-promise.  
  *You study pg-promise.*

- Nội dung gồm `ColumnSet`, `helpers.insert` và `db.none`.  
  *The content includes `ColumnSet`, `helpers.insert`, and `db.none`.*

- Bạn tìm hiểu psycopg2 `execute_values`.  
  *You study psycopg2 `execute_values`.*

- Bạn tìm hiểu các lỗi thường gặp.  
  *You study common errors.*

- 🔴 **Advanced:** Bạn tìm hiểu nguyên nhân COPY nhanh nhất.  
  *🔴 **Advanced:** You learn why COPY is the fastest method.*

- Nội dung gồm transaction, WAL và binary — nhị phân.  
  *The content includes transactions, WAL, and binary format.*

- Bạn tìm hiểu psycopg3 `copy()` hiện đại.  
  *You study the modern psycopg3 `copy()` method.*

- Bạn áp dụng nguyên tắc nạp trước.  
  *You apply the load-first principle.*

- Bạn áp dụng nguyên tắc tạo index sau.  
  *You apply the index-later principle.*

- Bạn tìm hiểu batch size — kích thước lô.  
  *You study batch size.*

- Bạn tìm hiểu `maintenance_work_mem`.  
  *You study `maintenance_work_mem`.*

- Bạn tìm hiểu edge cases — trường hợp biên.  
  *You study edge cases.*

- Các trường hợp gồm vector trong CSV và `register_vector`.  
  *The cases include vectors in CSV and `register_vector`.*

- 🟣 **Staff:** Bạn thiết kế ingestion pipeline cho hàng triệu vector.  
  *🟣 **Staff:** You design an ingestion pipeline for millions of vectors.*

- Bạn tìm hiểu drop và rebuild index.  
  *You study index dropping and rebuilding.*

- Bạn tìm hiểu parallel — xử lý song song.  
  *You study parallel processing.*

- Bạn tìm hiểu WAL và replication — sao chép dữ liệu.  
  *You study WAL and replication.*

- Bạn tìm hiểu upsert idempotent — nạp lại an toàn.  
  *You study idempotent upserts.*

- Bạn tìm hiểu monitoring — giám sát.  
  *You study monitoring.*

- Bạn luyện câu hỏi system design — thiết kế hệ thống.  
  *You practice system design questions.*

- 🎯 **Cheatsheet:** Bạn ghi nhớ keywords — từ khóa.  
  *🎯 **Cheatsheet:** You memorize keywords.*

- Bạn ghi nhớ core concepts — khái niệm cốt lõi.  
  *You memorize core concepts.*

- Bạn ghi nhớ code quan trọng.  
  *You memorize important code.*

- Bạn luyện câu hỏi phỏng vấn.  
  *You practice interview questions.*

- Bạn ghi nhớ các one-liner — câu chốt ngắn.  
  *You memorize useful one-liners.*

---

## Phần 1: 🟢 BASIC
*Part 1: 🟢 BASIC*

### 1.1. Nguyên nhân insert từng dòng chậm
*1.1. Why row-by-row insertion is slow*

- Mỗi `INSERT` một dòng tạo một transaction độc lập.  
  *Each single-row `INSERT` creates an independent transaction.*

- Mỗi transaction có chi phí cố định.  
  *Each transaction has fixed costs.*

- Chi phí gồm ghi WAL.  
  *The costs include WAL writing.*

- Chi phí gồm commit.  
  *The costs include commit operations.*

- Chi phí gồm fsync.  
  *The costs include fsync operations.*

- Chi phí gồm network round-trip — lượt truyền mạng khứ hồi.  
  *The costs include network round trips.*

- Một triệu dòng riêng lẻ tạo một triệu lần overhead.  
  *One million separate rows create one million overhead payments.*

- Tổng overhead trở nên khổng lồ.  
  *The total overhead becomes enormous.*

- Vector nặng làm vấn đề nghiêm trọng hơn.  
  *Heavy vectors make the problem worse.*

- Embedding có thể chứa 512–1536+ chiều.  
  *An embedding can contain 512–1536+ dimensions.*

- Mỗi dòng có thể chiếm vài KB.  
  *Each row can occupy several KB.*

- One-by-one tạo một triệu round-trip.  
  *One-by-one loading creates one million round trips.*

- Mỗi round-trip mang một khối vài KB.  
  *Each round trip carries a block of several KB.*

- Cách này cực kém hiệu quả.  
  *This method is extremely inefficient.*

Bài gốc trình bày đúng ý cốt lõi.  
*The original lesson presents the core idea correctly.*

Bulk insert giúp duy trì performance.  
*Bulk insert helps maintain performance.*

Bulk insert giúp nạp dữ liệu nhanh hơn.  
*Bulk insert enables faster data loading.*

Bulk insert gom nhiều dòng vào ít transaction.  
*Bulk insert groups many rows into few transactions.*

Bulk insert cũng giảm số round-trip.  
*Bulk insert also reduces the number of round trips.*

### 1.2. Analogy — phép so sánh: chuyển nhà bằng xe tải hoặc bằng tay
*1.2. Analogy: moving with a truck or by hand*

- **Row-by-row insert** giống việc bê từng cái ghế.  
  ***Row-by-row insert** resembles carrying one chair at a time.*

- Mỗi lượt cần đi bộ ra xe.  
  *Each trip requires a walk to the vehicle.*

- Mỗi lượt cần quay lại.  
  *Each trip requires a return walk.*

- 1000 món cần 1000 chuyến.  
  *One thousand items require one thousand trips.*

- **Batch insert** giống việc chất 50 món lên xe đẩy.  
  ***Batch insert** resembles loading 50 items onto a cart.*

- Bạn đẩy xe một lần cho mỗi lô.  
  *You push the cart once for each batch.*

- 1000 món cần 20 chuyến.  
  *One thousand items require 20 trips.*

- **COPY** giống việc thuê xe tải.  
  ***COPY** resembles hiring a truck.*

- Bạn chất toàn bộ đồ lên xe.  
  *You load all items onto the truck.*

- Xe chỉ chạy một chuyến.  
  *The truck makes only one trip.*

- COPY là phương pháp nhanh nhất.  
  *COPY is the fastest method.*

Khối lượng đồ không thay đổi.  
*The amount of cargo does not change.*

Số chuyến quyết định tốc độ.  
*The number of trips determines the speed.*

Trong database, chuyến là transaction hoặc round-trip.  
*In a database, a trip is a transaction or round trip.*

### 1.3. Ba phương pháp và thứ tự tốc độ
*1.3. Three methods and their speed ranking*

| Phương pháp / Method | Cơ chế / Mechanism | Tốc độ tương đối / Relative speed |
|---|---|---|
| Row-by-row `INSERT` / Row-by-row `INSERT` | 1 dòng / 1 câu / 1 transaction / 1 row / 1 statement / 1 transaction | 🐢 chậm nhất / 🐢 slowest |
| **Batch / multi-row `INSERT`** / **Batch / multi-row `INSERT`** | nhiều dòng / 1 câu / many rows / 1 statement | 🚗 nhanh hơn nhiều / 🚗 much faster |
| **`COPY`** / **`COPY`** | giao thức nạp khối chuyên dụng / dedicated bulk-loading protocol | 🚀 **nhanh nhất** / 🚀 **fastest** |

> **[Đính chính bài gốc]**  
> *[Correction to the original lesson]*
>
> Bài gốc trình bày COPY gần ngang hàng với pg-promise và execute_values.  
> *The original lesson presents COPY near pg-promise and execute_values.*
>
> Thực tế, COPY là phương pháp nhanh nhất.  
> *In practice, COPY is the fastest method.*
>
> Bạn nên dùng COPY trong mọi trường hợp phù hợp.  
> *You should use COPY whenever it fits.*
>
> Batch insert phù hợp với dữ liệu từ code hoặc app.  
> *Batch insert suits data from code or an app.*
>
> Batch insert cũng phù hợp với logic phức tạp.  
> *Batch insert also suits complex logic.*
>
> Dữ liệu từ code không nhất thiết nằm trong file.  
> *Data from code does not necessarily reside in a file.*

### 1.4. COPY cơ bản: nạp từ CSV
*1.4. Basic COPY: loading from CSV*

`COPY` chuyển dữ liệu giữa file và bảng.  
*`COPY` transfers data between a file and a table.*

COPY hỗ trợ cả hai chiều.  
*COPY supports both directions.*

Bài gốc dùng file `FAQdata.csv`.  
*The original lesson uses the `FAQdata.csv` file.*

File chứa text và embedding.  
*The file contains text and embeddings.*

Dữ liệu được nạp vào bảng `faqs`.  
*The data enters the `faqs` table.*

Bảng có các cột `id`, `text` và `embedding`.  
*The table has `id`, `text`, and `embedding` columns.*

```sql
-- Bảng đích
-- Target table
-- embedding là cột vector
-- embedding is the vector column
CREATE TABLE faqs (
    id        serial PRIMARY KEY,
    text      text,
    embedding vector(512)
);

-- COPY phía server
-- Server-side COPY
-- File phải nằm trên MÁY CHỦ Postgres
-- The file must reside on the PostgreSQL SERVER
-- Lệnh cần quyền cao
-- The command requires elevated privileges
COPY faqs (text, embedding)
FROM '/path/FAQdata.csv'
WITH (FORMAT csv, HEADER true);
```

Trong `psql`, bạn có thể dùng `\copy`.  
*In `psql`, you can use `\copy`.*

`\copy` chạy ở phía client — máy khách.  
*`\copy` runs on the client side.*

File nằm trên máy của bạn.  
*The file resides on your machine.*

Lệnh không cần quyền server.  
*The command does not require server privileges.*

```sql
\copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)
```

**Gotcha — điểm dễ lỗi của vector trong CSV**  
*Gotcha for vectors in CSV*

Embedding có thể có dạng `[0.1,0.2,...]`.  
*An embedding can use the `[0.1,0.2,...]` form.*

Dạng này chứa dấu phẩy.  
*This form contains commas.*

Trong CSV, bạn phải bọc giá trị bằng nháy kép.  
*In CSV, you must wrap the value in double quotes.*

```csv
text,embedding
"thủ đô Anh là London","[0.12,0.98,...]"
"đường sắt Đức là Deutsche Bahn","[0.44,0.03,...]"
```

Bạn có thể quên bọc nháy kép.  
*You can forget the double quotes.*

Khi đó, CSV parser tách từng số thành một cột.  
*The CSV parser then splits each number into a column.*

Quá trình nạp sẽ báo lỗi.  
*The loading process will report an error.*

### 1.5. Xuất bảng ra CSV
*1.5. Exporting a table to CSV*

```sql
\copy (SELECT text, embedding FROM faqs) TO 'export.csv' WITH (FORMAT csv, HEADER true)
```

### ✅ Self-check Phần 1
*✅ Part 1 Self-check*

1. Tại sao một triệu `INSERT` riêng lẻ chậm hơn một batch?  
   *Why are one million separate `INSERT` statements slower than one batch?*

2. Hãy xếp row-by-row, batch insert và COPY theo tốc độ.  
   *Rank row-by-row, batch insert, and COPY by speed.*

3. Tại sao embedding `[1,2,3]` cần nháy kép trong CSV?  
   *Why does the `[1,2,3]` embedding need double quotes in CSV?*

---

## Phần 2: 🟡 INTERMEDIATE
*Part 2: 🟡 INTERMEDIATE*

### 2.1. Bulk insert trong Node.js với pg-promise
*2.1. Bulk insert in Node.js with pg-promise*

Phần này bám sát bài gốc.  
*This section follows the original lesson.*

Dữ liệu có thể không ở dạng CSV.  
*The data may not use CSV format.*

Dữ liệu có thể đến từ code.  
*The data may come from code.*

Ví dụ, app vừa sinh embedding.  
*For example, the app just generated embeddings.*

Trong trường hợp này, hãy dùng **pg-promise**.  
*In this case, use **pg-promise**.*

Pg-promise thực hiện batch insert.  
*Pg-promise performs batch insert.*

```javascript
// npm install pg-promise
const pgp = require('pg-promise')();
const db  = pgp('postgresql://user:pass@localhost/db');   // khởi tạo với config DB
// initialize with the DB configuration

// 1) Định nghĩa bảng và cột sẽ insert
// 1) Define the target table and columns
// ColumnSet có thể tái sử dụng
// ColumnSet is reusable
// ColumnSet có hiệu năng tốt
// ColumnSet has good performance
const cs = new pgp.helpers.ColumnSet(['text', 'embedding'], { table: 'faqs' });

// 2) Chuẩn bị dữ liệu dưới dạng mảng object
// 2) Prepare the data as an object array
// Format embedding thành vector literal '[...]'
// Format each embedding as the '[...]' vector literal
const rows = [
  { text: 'thủ đô Anh là London',        embedding: '[' + emb1.join(',') + ']' },
  { text: 'đường sắt Đức Deutsche Bahn',  embedding: '[' + emb2.join(',') + ']' },
  // ... hàng nghìn dòng
  // ... thousands of rows
];

// 3) pgp.helpers.insert tạo MỘT câu INSERT nhiều dòng từ ColumnSet
// 3) pgp.helpers.insert creates ONE multi-row INSERT statement from ColumnSet
const query = pgp.helpers.insert(rows, cs);   // INSERT INTO faqs(text,embedding) VALUES (...),(...),...

// 4) db.none chạy query không trả về dữ liệu
// 4) db.none runs a query without returned data
// Câu insert không trả về row
// The insert statement returns no rows
await db.none(query);
```

Bước đầu tiên là cài pg-promise.  
*The first step is installing pg-promise.*

Bước tiếp theo là import thư viện.  
*The next step is importing the library.*

Sau đó, bạn init — khởi tạo thư viện.  
*Next, you initialize the library.*

Bạn định nghĩa `ColumnSet`.  
*You define the `ColumnSet`.*

Bạn gom các values — giá trị.  
*You collect the values.*

Bạn gọi `pgp.helpers.insert`.  
*You call `pgp.helpers.insert`.*

Cuối cùng, bạn gọi `db.none`.  
*Finally, you call `db.none`.*

Đây là luồng của bài gốc.  
*This is the original lesson flow.*

`helpers.insert` gộp nhiều dòng.  
*`helpers.insert` groups many rows.*

Các dòng nằm trong một câu INSERT.  
*The rows reside in one INSERT statement.*

Câu lệnh đó là một batch.  
*That statement is one batch.*

Cách này thay thế hàng nghìn câu riêng.  
*This method replaces thousands of separate statements.*

> **[MỞ RỘNG]**  
> *[EXPANSION]*
>
> Lô dữ liệu có thể rất lớn.  
> *A data batch can be very large.*
>
> Bạn nên chia lô thành nhiều chunk — khối nhỏ.  
> *You should split the batch into chunks.*
>
> Mỗi chunk có thể chứa 1.000–10.000 dòng.  
> *Each chunk can contain 1,000–10,000 rows.*
>
> Bạn xử lý lần lượt từng chunk.  
> *You process each chunk in sequence.*
>
> Một câu INSERT có thể quá lớn.  
> *An INSERT statement can become too large.*
>
> Câu đó có thể chứa hàng trăm nghìn dòng.  
> *That statement can contain hundreds of thousands of rows.*
>
> Câu đó có thể vượt giới hạn tham số.  
> *That statement can exceed parameter limits.*
>
> Câu đó cũng có thể vượt giới hạn bộ nhớ.  
> *That statement can also exceed memory limits.*
>
> Trong Node, COPY là phương pháp nhanh nhất.  
> *In Node, COPY is the fastest method.*
>
> Bạn có thể dùng `pg-copy-streams`.  
> *You can use `pg-copy-streams`.*

### 2.2. Bulk insert trong Python với psycopg2
*2.2. Bulk insert in Python with psycopg2*

Phần này bám sát bài gốc.  
*This section follows the original lesson.*

```python
# pip install psycopg2-binary pgvector
import psycopg2
from psycopg2.extras import execute_values
from pgvector.psycopg2 import register_vector

conn = psycopg2.connect("dbname=db user=... password=... host=localhost")
register_vector(conn)                      # để embedding (list/ndarray) serialize thành vector
# serialize an embedding from a list or ndarray into a vector

# data là list các tuple
# data is a list of tuples
# Mỗi tuple chứa sentence và embedding
# Each tuple contains a sentence and an embedding
# Mỗi tuple có độ dài 2
# Each tuple has a length of 2
data = [
    ("thủ đô Anh là London",        emb1),   # emb1 là list/ndarray float
    # emb1 is a float list or ndarray
    ("đường sắt Đức Deutsche Bahn",  emb2),
    # ... nhiều dòng
    # ... many rows
]

with conn.cursor() as cur:
    execute_values(                        # gộp nhiều dòng vào 1 câu INSERT
        # group many rows into one INSERT statement
        cur,
        "INSERT INTO faqs (text, embedding) VALUES %s",   # %s là chỗ execute_values chèn tất cả
        # %s is the position for all values from execute_values
        data,
    )
conn.commit()                              # commit MỘT lần cho cả lô
# commit ONCE for the entire batch
cur.close(); conn.close()
```

Đầu tiên, bạn cài psycopg2.  
*First, you install psycopg2.*

Bạn import thư viện.  
*You import the library.*

Bạn import `execute_values` từ `extras`.  
*You import `execute_values` from `extras`.*

Bạn truyền query template — mẫu truy vấn.  
*You pass the query template.*

Bạn cũng truyền một list tuple.  
*You also pass a tuple list.*

Bạn mở connection — kết nối.  
*You open a connection.*

Bạn tạo cursor — con trỏ truy vấn.  
*You create a cursor.*

Bạn chạy lệnh execute.  
*You run the execute operation.*

Bạn gọi **commit**.  
*You call **commit**.*

Cuối cùng, bạn đóng cursor.  
*Finally, you close the cursor.*

Bạn cũng đóng connection.  
*You also close the connection.*

Đây là luồng đúng của bài gốc.  
*This is the correct original lesson flow.*

> **[Đính chính quan trọng]**  
> *[Important correction]*
>
> `psycopg2` là một thư viện cũ.  
> *`psycopg2` is an old library.*
>
> Bản kế nhiệm là **`psycopg` v3**.  
> *Its successor is **`psycopg` v3**.*
>
> Psycopg3 không còn `execute_values`.  
> *Psycopg3 no longer includes `execute_values`.*
>
> Psycopg3 dùng `executemany()`.  
> *Psycopg3 uses `executemany()`.*
>
> Hàm này đã tối ưu pipeline.  
> *This function has an optimized pipeline.*
>
> Psycopg3 cũng dùng `cursor.copy()`.  
> *Psycopg3 also uses `cursor.copy()`.*
>
> `cursor.copy()` là phương pháp nhanh nhất.  
> *`cursor.copy()` is the fastest method.*
>
> Code psycopg2 vẫn chạy tốt.  
> *Psycopg2 code still works well.*
>
> Dự án mới nên cân nhắc psycopg3.  
> *New projects should consider psycopg3.*
>
> Mục 3.2 trình bày thêm nội dung này.  
> *Section 3.2 presents more details.*

### 2.3. [MỞ RỘNG NGOÀI BÀI GỐC]: Ba lỗi thường gặp
*2.3. [EXPANSION BEYOND THE ORIGINAL LESSON]: Three common errors*

**Lỗi 1: Quên `commit()` trong Python**  
*Error 1: Forgetting `commit()` in Python*

Psycopg2 mặc định không bật autocommit.  
*Psycopg2 disables autocommit by default.*

Thiếu `conn.commit()` làm dữ liệu không được lưu.  
*Missing `conn.commit()` leaves the data unsaved.*

Connection đóng sẽ rollback dữ liệu.  
*Connection closure will roll back the data.*

Bài gốc nhấn mạnh đúng bước commit.  
*The original lesson correctly emphasizes the commit step.*

Bạn không được bỏ bước này.  
*You must not skip this step.*

**Lỗi 2: Quên đăng ký kiểu hoặc dùng sai format vector**  
*Error 2: Forgetting type registration or using the wrong vector format*

Bạn có thể quên `register_vector` trong Python.  
*You can forget `register_vector` in Python.*

Bạn có thể dùng sai format `'[...]'` trong Node.  
*You can use the wrong `'[...]'` format in Node.*

Driver không thể chuyển list thành vector.  
*The driver cannot convert the list into a vector.*

Quá trình có thể báo lỗi.  
*The process can report an error.*

Dữ liệu cũng có thể bị lưu sai.  
*The data can also be stored incorrectly.*

Bạn luôn cần đăng ký kiểu.  
*You always need type registration.*

Bạn cũng có thể format vector literal đúng.  
*You can also format the vector literal correctly.*

**Lỗi 3: Một câu INSERT hoặc COPY quá khổng lồ**  
*Error 3: An excessively large INSERT or COPY statement*

Bạn không nên đưa cả triệu dòng vào một lệnh.  
*You should not place one million rows in one command.*

Lệnh đó có thể làm tràn bộ nhớ.  
*That command can exhaust memory.*

Lệnh đó có thể vượt giới hạn.  
*That command can exceed system limits.*

Bạn nên chia dữ liệu thành các chunk.  
*You should split the data into chunks.*

Batch size phù hợp là 1k–10k dòng.  
*A suitable batch size is 1k–10k rows.*

Bạn có thể bọc mỗi chunk trong một transaction riêng.  
*You can wrap each chunk in a separate transaction.*

Lựa chọn này phụ thuộc nhu cầu.  
*This choice depends on your needs.*

Cách này cân bằng tốc độ và bộ nhớ.  
*This method balances speed and memory.*

### ✅ Self-check Phần 2
*✅ Part 2 Self-check*

1. `ColumnSet` làm gì trong pg-promise?  
   *What does `ColumnSet` do in pg-promise?*

`pgp.helpers.insert` làm gì trong pg-promise?  
*What does `pgp.helpers.insert` do in pg-promise?*

2. Điều gì xảy ra sau việc quên `conn.commit()` trong psycopg2?  
   *What happens after forgetting `conn.commit()` in psycopg2?*

3. Psycopg3 còn `execute_values` không?  
   *Does psycopg3 still include `execute_values`?*

Phương pháp nào thay thế nó?  
*Which methods replace it?*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED*

### 3.1. Vì sao COPY nhanh nhất — dưới nắp capo
*3.1. Why COPY is the fastest under the hood*

- COPY sử dụng ít transaction hơn.  
  *COPY uses fewer transactions.*

COPY nạp cả khối trong một luồng.  
*COPY loads the entire block through one stream.*

COPY không parse lại từng câu INSERT.  
*COPY does not parse every INSERT statement again.*

COPY không lặp overhead commit.  
*COPY does not repeat commit overhead.*

- COPY sử dụng giao thức binary.  
  *COPY uses the binary protocol.*

`FORMAT BINARY` gửi dữ liệu nhị phân thô.  
*`FORMAT BINARY` sends raw binary data.*

Nó không gửi dữ liệu dưới dạng text.  
*It does not send data as text.*

Cách này tiết kiệm băng thông.  
*This method saves bandwidth.*

Server không cần parse text.  
*The server does not need to parse text.*

- COPY tối ưu buffer và cache.  
  *COPY optimizes buffers plus cache usage.*

COPY sử dụng một ring buffer riêng.  
*COPY uses a dedicated ring buffer.*

Cơ chế này giảm áp lực lên shared buffers.  
*This mechanism reduces pressure on shared buffers.*

- COPY vẫn bảo đảm an toàn.  
  *COPY still preserves safety.*

COPY vẫn ghi WAL.  
*COPY still writes WAL.*

Pgvector cũng sử dụng WAL.  
*Pgvector also uses WAL.*

Replication vẫn hoạt động.  
*Replication still works.*

Point-in-time recovery vẫn được bảo toàn.  
*Point-in-time recovery remains available.*

Các benchmark phổ biến cho thấy COPY nhanh hơn batch INSERT nhiều lần.  
*Common benchmarks show much faster COPY performance than batch INSERT.*

COPY bỏ xa row-by-row.  
*COPY greatly outperforms row-by-row insertion.*

### 3.2. psycopg3 `copy()` — cách nhanh nhất hiện đại (thay execute_values)
*3.2. psycopg3 `copy()` as the fastest modern method*

```python
# pip install "psycopg[binary]" pgvector
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("dbname=db user=... host=localhost")
register_vector(conn)

rows = [("thủ đô Anh là London", emb1), ("đường sắt Đức", emb2)]  # generator được luôn (lazy)
# A generator also works here (lazy)

with conn.cursor() as cur:
    with cur.copy("COPY faqs (text, embedding) FROM STDIN WITH (FORMAT BINARY)") as copy:
        copy.set_types(["text", "vector"])       # khai báo kiểu để serialize đúng
        # Declare the types for correct serialization
        for text, emb in rows:
            copy.write_row([text, emb])           # từng dòng được stream thẳng vào server
            # Each row streams directly to the server
conn.commit()
```

Psycopg3 `copy()` có tốc độ cao nhất.  
*Psycopg3 `copy()` provides the highest speed.*

Phương pháp này hỗ trợ stream lazy.  
*This method supports lazy streaming.*

Nó không cần nạp một triệu dòng vào RAM.  
*It does not load one million rows into RAM.*

Nó hỗ trợ format binary.  
*It supports the binary format.*

Đây là cách khuyến nghị cho ingestion lớn trong năm 2026.  
*This is the recommended method for large ingestion jobs in 2026.*

Bạn có thể không muốn sử dụng COPY.  
*You may not want to use COPY.*

Psycopg3 cung cấp `executemany()` đã tối ưu pipeline.  
*Psycopg3 provides a pipeline-optimized `executemany()` method.*

```python
cur.executemany("INSERT INTO faqs (text, embedding) VALUES (%s, %s)", rows)
```

### 3.3. [NGUYÊN TẮC VÀNG] Nạp dữ liệu TRƯỚC, tạo index SAU
*3.3. [GOLDEN RULE] Load data FIRST, create the index LATER*

Bài gốc bỏ qua nguyên tắc này.  
*The original lesson omits this rule.*

Nguyên tắc này quyết định tốc độ ingestion.  
*This rule determines ingestion speed.*

- Bảng có thể đã chứa index HNSW hoặc IVFFlat.  
  *The table may already contain an HNSW or IVFFlat index.*

Mỗi lần insert phải cập nhật index.  
*Every insert must update the index.*

Quá trình insert sẽ chậm rõ rệt.  
*The insertion process becomes noticeably slower.*

HNSW chậm hơn với `ef_construction` cao.  
*HNSW becomes slower with a high `ef_construction` value.*

- Đây là cách đúng cho bulk load.  
  *This is the correct method for bulk loading.*

  ```sql
  -- 1) Bảng KHÔNG index
  -- 1) Table WITHOUT an index
  CREATE TABLE faqs (id serial PRIMARY KEY, text text, embedding vector(512));
  -- 2) COPY / bulk insert toàn bộ dữ liệu (nhanh vì không maintain index)
  -- 2) COPY or bulk insert all data without index maintenance
  \copy faqs (text, embedding) FROM 'FAQdata.csv' WITH (FORMAT csv, HEADER true)
  -- 3) SAU KHI nạp xong mới build index (một lần, nhanh hơn nhiều)
  -- 3) Build the index only AFTER loading all data
  SET maintenance_work_mem = '2GB';                    -- tăng RAM cho build
  -- Increase RAM for the build
  CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
  ```

IVFFlat bắt buộc dùng quy trình này.  
*IVFFlat requires this process.*

IVFFlat cần dữ liệu có sẵn cho k-means.  
*IVFFlat needs existing data for k-means.*

Bạn có thể xem thêm giáo trình indexing.  
*You can review the indexing lesson.*

HNSW cũng nên dùng quy trình này.  
*HNSW should also use this process.*

Cách này giúp build nhanh hơn.  
*This method makes the build faster.*

### 3.4. Batch size & transaction — tinh chỉnh
*3.4. Batch size plus transaction tuning*

- Batch size phổ biến là 1.000–10.000 dòng mỗi lô.  
  *A common batch size is 1,000–10,000 rows per batch.*

Khoảng này thường là điểm ngọt.  
*This range is usually the sweet spot.*

Batch quá nhỏ tạo nhiều round-trip.  
*An excessively small batch creates many round-trips.*

Batch quá lớn tiêu thụ nhiều RAM.  
*An excessively large batch consumes much RAM.*

Một lỗi có thể làm hỏng toàn bộ lô lớn.  
*One error can break the entire large batch.*

- Bạn nên bọc mỗi lô trong một transaction.  
  *You should wrap every batch in one transaction.*

Ví dụ là `BEGIN...COMMIT`.  
*One example is `BEGIN...COMMIT`.*

Cách này giảm số lần commit.  
*This method reduces the number of commits.*

Cách này vẫn giới hạn thiệt hại của lỗi.  
*This method still limits the damage from an error.*

- `maintenance_work_mem` phục vụ bước build index sau khi nạp.  
  *`maintenance_work_mem` supports the index build after loading.*

Bạn nên đặt giá trị này ở mức cao.  
*You should set this value high.*

Giá trị này không nên vượt 50–60% RAM.  
*This value should not exceed 50–60% of RAM.*

HNSW có thể được build song song.  
*HNSW can use a parallel build.*

Trong Docker, `--shm-size` phải đủ lớn.  
*In Docker, `--shm-size` must be large enough.*

### 3.5. Edge cases
*3.5. Edge cases*

- Vector trong CSV phải được bọc bằng nháy kép.  
  *A vector in CSV must use double quotes.*

Bạn có thể xem lại mục 1.4.  
*You can review Section 1.4.*

- Một dòng có thể chứa vector sai chiều.  
  *One row may contain a vector with the wrong dimension.*

COPY hoặc insert sẽ làm cả lô thất bại.  
*COPY or INSERT will make the whole batch fail.*

Một lệnh COPY thường có tính all-or-nothing.  
*A COPY command usually has all-or-nothing behavior.*

Bạn phải validate số chiều trước khi nạp.  
*You must validate the dimensions before loading.*

- Cột nullable có thể chứa NULL embedding.  
  *A nullable column can contain a NULL embedding.*

Dòng đó không xuất hiện trong ANN search.  
*That row does not appear in ANN search.*

Bạn có thể backfill embedding sau.  
*You can backfill the embedding later.*

- COPY không hỗ trợ `ON CONFLICT` trực tiếp.  
  *COPY does not support `ON CONFLICT` directly.*

Bạn có thể COPY dữ liệu vào bảng tạm.  
*You can COPY the data into a temporary table.*

Sau đó, bạn chạy `INSERT ... SELECT ... ON CONFLICT DO UPDATE`.  
*Next, you run `INSERT ... SELECT ... ON CONFLICT DO UPDATE`.*

Bạn có thể xem mục 4.3.  
*You can review Section 4.3.*

- `register_vector` là yêu cầu bắt buộc.  
  *`register_vector` is a mandatory requirement.*

Yêu cầu này áp dụng cho execute_values.  
*This requirement applies to execute_values.*

Yêu cầu này cũng áp dụng cho copy.  
*This requirement also applies to copy.*

Nó giúp serialize vector chính xác.  
*It enables correct vector serialization.*

### ✅ Self-check Phần 3
*✅ Part 3 Self-check*

1. Hãy nêu hai lý do COPY nhanh hơn batch INSERT.  
   *Give two reasons for faster COPY performance than batch INSERT.*

2. Vì sao bạn nên nạp dữ liệu trước?  
   *Why should you load the data first?*

Vì sao bạn nên tạo HNSW index sau?  
*Why should you create the HNSW index later?*

Quy trình với IVFFlat có điểm gì khác?  
*What differs in the IVFFlat process?*

3. Psycopg3 `copy()` có ưu điểm gì về bộ nhớ?  
   *What memory advantage does psycopg3 `copy()` provide?*

Hãy so sánh nó với execute_values dùng cả list.  
*Compare it with execute_values using a complete list.*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL: Systems thinking plus technical leadership*

### 4.1. Ingestion pipeline ở quy mô triệu vector
*4.1. An ingestion pipeline for millions of vectors*

Bulk insert không hoạt động độc lập.  
*Bulk insert does not operate alone.*

Nó là một mắt trong pipeline.  
*It is one stage in the pipeline.*

Sơ đồ sau mô tả quá trình nạp hàng triệu tài liệu.  
*The following diagram describes the loading of millions of documents.*

```
1. Chunk tài liệu dài (tránh truncation của model)      [giáo trình embeddings]
   Chunk long documents (avoid model truncation)        [embedding lesson]
2. Sinh embedding theo BATCH (32-256/lần) trên GPU/API   [batch = tối ưu #1]
   Generate embeddings in BATCHES (32-256 each) on GPU/API [batch = optimization #1]
3. COPY (binary) vào bảng CHƯA index                     [nạp nhanh nhất]
   COPY (binary) into a table WITHOUT an index            [fastest loading]
4. Build index HNSW SAU khi nạp xong (maintenance_work_mem cao)
   Build the HNSW index AFTER loading (high maintenance_work_mem)
5. VERIFY (EXPLAIN ANALYZE thấy Index Scan; recall@k)
   VERIFY (EXPLAIN ANALYZE shows Index Scan, recall@k)
```

Bottleneck thường nằm ở bước sinh embedding.  
*The bottleneck usually lies in embedding generation.*

Bottleneck thường không nằm ở insert.  
*The bottleneck usually does not lie in insertion.*

Đây là bước 2 trên API hoặc GPU.  
*This is Step 2 on an API or GPU.*

Row-by-row insert có thể tạo thêm một nút thắt.  
*Row-by-row insertion can create another bottleneck.*

Khi đó insert trở thành bottleneck.  
*Insertion then becomes the bottleneck.*

COPY loại bỏ nút thắt này.  
*COPY removes this bottleneck.*

### 4.2. Chiến lược drop & rebuild index cho re-load lớn
*4.2. Drop plus rebuild strategies for a large reload*

Bạn có thể cần nạp lại lượng dữ liệu rất lớn.  
*You may need to reload a very large dataset.*

Bạn cũng có thể cần nạp thêm lượng dữ liệu lớn.  
*You may also need to load a large data increment.*

Bảng đích có thể đã chứa index.  
*The target table may already contain an index.*

- Với đợt nạp lớn, bạn nên cân nhắc `DROP INDEX`.  
  *For a large load, you should consider `DROP INDEX`.*

Sau đó, bạn chạy bulk load.  
*Next, you run the bulk load.*

Cuối cùng, bạn chạy lại `CREATE INDEX CONCURRENTLY`.  
*Finally, you run `CREATE INDEX CONCURRENTLY` again.*

Cách này có chi phí thấp hơn.  
*This method has a lower cost.*

Database không phải duy trì index trong mỗi insert.  
*The database does not maintain the index during every insert.*

- Service không thể mất search trong lúc re-load.  
  *The service cannot lose search during the reload.*

Bạn có thể dùng `CREATE INDEX CONCURRENTLY`.  
*You can use `CREATE INDEX CONCURRENTLY`.*

Lệnh này không khóa bảng.  
*This command does not lock the table.*

Bạn cũng có thể dùng bảng shadow.  
*You can also use a shadow table.*

Sau đó, bạn thực hiện swap.  
*Next, you perform the swap.*

- Tăng `max_parallel_maintenance_workers`.  
  *Increase `max_parallel_maintenance_workers`.*

Nhiều core sẽ build index nhanh hơn.  
*More cores will build the index faster.*

### 4.3. Idempotency & upsert — nạp lại an toàn
*4.3. Idempotency plus upsert for safe reloading*

Pipeline có thể chạy lại cho retry hoặc backfill.  
*The pipeline may run again for retry or backfill.*

Mỗi lần chạy không được tạo dòng trùng.  
*Each run must not create duplicate rows.*

COPY không hỗ trợ `ON CONFLICT`.  
*COPY does not support `ON CONFLICT`.*

```sql
-- 1) COPY vào bảng tạm không ràng buộc
-- 1) COPY into an unconstrained temporary table
CREATE TEMP TABLE faqs_stage (LIKE faqs INCLUDING DEFAULTS);
\copy faqs_stage (text, embedding) FROM 'batch.csv' WITH (FORMAT csv)

-- 2) Merge vào bảng thật, xử lý trùng
-- 2) Merge into the real table with duplicate handling
INSERT INTO faqs (text, embedding)
SELECT text, embedding FROM faqs_stage
ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;
```

Mẫu này có tên "COPY → temp → upsert".  
*This pattern is called "COPY → temp → upsert".*

Mẫu này phù hợp cho production ingestion idempotent.  
*This pattern suits idempotent production ingestion.*

### 4.4. WAL, replication, cost, monitoring
*4.4. WAL, replication, cost, monitoring*

- COPY vẫn ghi WAL.  
  *COPY still writes WAL.*

Bulk load lớn tạo nhiều WAL.  
*A large bulk load creates much WAL.*

Điều này có thể tăng replication lag.  
*This can increase replication lag.*

Điều này cũng chiếm nhiều dung lượng đĩa.  
*This also consumes much disk space.*

Bạn cần theo dõi WAL generation.  
*You need to monitor WAL generation.*

Bạn cũng cần theo dõi replica lag.  
*You also need to monitor replica lag.*

Hoạt động theo dõi cần diễn ra trong bulk load lớn.  
*The monitoring must continue during a large bulk load.*

- Bảng `UNLOGGED` bỏ qua WAL.  
  *An `UNLOGGED` table skips WAL.*

Cách này giúp nạp nhanh hơn.  
*This method enables faster loading.*

Dữ liệu có thể mất sau crash.  
*The data may disappear after a crash.*

Bảng này không được replicate.  
*This table does not replicate.*

Bạn chỉ nên dùng nó cho staging.  
*You should use it only for staging.*

Dữ liệu staging phải có thể tái tạo.  
*The staging data must be reproducible.*

- Chi phí ingestion chủ yếu nằm ở sinh embedding.  
  *The main ingestion cost lies in embedding generation.*

Chi phí này gồm API token hoặc GPU.  
*This cost includes API tokens or GPU usage.*

Insert thường không phải phần tốn kém nhất.  
*Insertion is usually not the most expensive part.*

Insert kém hiệu quả kéo dài thời gian nạp.  
*Inefficient insertion extends the loading time.*

Thời gian dài hơn làm tăng compute.  
*Longer runtime increases compute usage.*

Bạn nên cache embedding theo hash.  
*You should cache embeddings by hash.*

Cache giúp tránh re-embed trong lần nạp lại.  
*The cache avoids re-embedding during a reload.*

- Theo dõi số rows mỗi giây.  
  *Monitor the number of rows per second.*

Theo dõi thời gian build index.  
*Monitor the index build time.*

Theo dõi WAL và replica lag.  
*Monitor WAL plus replica lag.*

Theo dõi tỷ lệ dòng lỗi.  
*Monitor the failed-row rate.*

Lỗi thường gồm dimension mismatch và NULL.  
*Common errors include dimension mismatch plus NULL values.*

Theo dõi RAM trong lúc build.  
*Monitor RAM during the build.*

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder
*4.5. Organizational impact plus explanations for non-technical stakeholders*

- **Nói với PM hoặc sếp:**  
  *Explain it to the PM or manager:*

Đợt nạp dữ liệu ban đầu có thể chứa 5 triệu tài liệu.  
*The initial load may contain 5 million documents.*

Đây là một tác vụ chạy một lần.  
*This is a one-time task.*

Tác vụ này vẫn tốn nhiều thời gian.  
*This task still takes much time.*

Phần chậm nhất là quá trình AI sinh embedding.  
*The slowest part is AI embedding generation.*

Việc lưu vào database không phải phần chậm nhất.  
*Database storage is not the slowest part.*

Kỹ thuật đúng gồm COPY.  
*The correct technique includes COPY.*

Kỹ thuật đúng cũng gồm nạp trước.  
*The correct technique also includes loading first.*

Index được tạo sau.  
*The index comes later.*

Những kỹ thuật này có thể rút thời gian từ nhiều ngày xuống vài giờ.  
*These techniques can reduce the runtime from days to hours.*

Chúng ta nên lên lịch nạp ngoài giờ cao điểm.  
*We should schedule the load outside peak hours.*

Lịch này giúp tránh ảnh hưởng tới khách hàng.  
*This schedule avoids customer impact.*

Cách giải thích này tập trung vào thời gian.  
*This explanation focuses on time.*

Nó cũng tập trung vào chi phí compute.  
*It also focuses on compute cost.*

Nó còn tập trung vào lịch vận hành.  
*It also focuses on the operating schedule.*

- Ingestion pipeline gồm chunk, embed, COPY, index và verify.  
  *The ingestion pipeline includes chunking, embedding, COPY, indexing, plus verification.*

Pipeline này là hạ tầng dùng lại.  
*This pipeline is reusable infrastructure.*

Mỗi lần thêm dữ liệu đều sử dụng hạ tầng này.  
*Every data addition uses this infrastructure.*

Bạn nên xây nó thành job hoặc service.  
*You should build it as a job or service.*

Job cần hỗ trợ retry.  
*The job needs retry support.*

Job cũng cần idempotency.  
*The job also needs idempotency.*

Bạn không nên viết script dùng một lần.  
*You should not write a one-time script.*

Bạn không nên bỏ script sau đó.  
*You should not discard the script afterward.*

- Idempotency sử dụng upsert.  
  *Idempotency uses upsert.*

Nó cho phép job chạy lại sau lỗi.  
*It lets the job run again after an error.*

Dữ liệu vẫn không bị hỏng.  
*The data remains intact.*

Khả năng này rất quan trọng.  
*This capability is crucial.*

Job nạp có thể chạy nhiều giờ.  
*The loading job may run for many hours.*

Job cũng có thể bị đứt giữa chừng.  
*The job may also stop midway.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff
*4.6. A sample system design question plus a staff-level answer*

> **Bạn cần nạp 10 triệu tài liệu vào pgvector.**  
> *You need to load 10 million documents into pgvector.*
>
> **Mỗi tài liệu chứa khoảng 500 token.**  
> *Each document contains about 500 tokens.*
>
> **Dữ liệu phục vụ semantic search.**  
> *The data supports semantic search.*
>
> **Bạn cần đạt tốc độ cao nhất có thể.**  
> *You need the highest possible speed.*
>
> **Service hiện có không được gián đoạn.**  
> *The existing service must remain uninterrupted.*
>
> **Job có thể bị đứt giữa chừng.**  
> *The job may stop midway.*
>
> **Job phải chạy lại được sau đó.**  
> *The job must run again afterward.*

**Khung trả lời staff:**  
*Staff-level answer framework:*

1. **Clarify:** Xác định nguồn dữ liệu.  
   *Clarify: Identify the data source.*

Nguồn có thể là file, database hoặc stream.  
*The source may be a file, database, or stream.*

Xác định nơi sinh embedding.  
*Identify the embedding generation location.*

Embedding có thể được sinh local hoặc qua API.  
*Embeddings may come from local compute or an API.*

Xác định khả năng downtime.  
*Determine the available downtime.*

Xác định ngân sách.  
*Determine the budget.*

2. Sinh embedding theo batch.  
   *Generate embeddings in batches.*

Đây là bottleneck thật.  
*This is the real bottleneck.*

Chạy các worker song song.  
*Run parallel workers.*

Batch size phù hợp là 128–256.  
*A suitable batch size is 128–256.*

Cache embedding theo hash.  
*Cache embeddings by hash.*

Cache giúp tránh trả lại chi phí embedding.  
*The cache prevents repeated embedding costs.*

3. Nạp dữ liệu bằng COPY binary.  
   *Load the data with binary COPY.*

Bảng đích chưa có index.  
*The target table has no index.*

Bạn cũng có thể dùng bảng staging.  
*You can also use a staging table.*

COPY là phương pháp nhanh nhất.  
*COPY is the fastest method.*

Chia dữ liệu thành các chunk.  
*Split the data into chunks.*

Cách này tăng khả năng chịu lỗi.  
*This method improves fault tolerance.*

4. Pipeline phải idempotent.  
   *The pipeline must be idempotent.*

COPY dữ liệu vào temp table.  
*COPY the data into a temporary table.*

Sau đó, chạy `INSERT ... ON CONFLICT DO UPDATE`.  
*Next, run `INSERT ... ON CONFLICT DO UPDATE`.*

Cách này giúp retry an toàn.  
*This method enables safe retries.*

5. Build index sau.  
   *Build the index later.*

Dùng `CREATE INDEX CONCURRENTLY ... hnsw ...`.  
*Use `CREATE INDEX CONCURRENTLY ... hnsw ...`.*

Đặt `maintenance_work_mem` ở mức cao.  
*Set `maintenance_work_mem` to a high value.*

Sử dụng các parallel worker.  
*Use parallel workers.*

Quá trình này không khóa service.  
*This process does not lock the service.*

6. Bảo vệ service đang chạy.  
   *Protect the running service.*

Nạp dữ liệu vào bảng shadow.  
*Load the data into a shadow table.*

Sau đó, thực hiện swap.  
*Next, perform the swap.*

Một lựa chọn khác là index CONCURRENTLY.  
*Another option is a concurrent index build.*

Theo dõi WAL và replica lag.  
*Monitor WAL plus replica lag.*

Giảm tốc độ nạp khi lag tăng.  
*Reduce the loading rate when lag increases.*

7. Chạy `EXPLAIN ANALYZE`.  
   *Run `EXPLAIN ANALYZE`.*

Kiểm tra Index Scan.  
*Check the Index Scan.*

Đo recall@k.  
*Measure recall@k.*

Đếm rows.  
*Count the rows.*

8. Điểm nghẽn là embedding generation.  
   *The bottleneck is embedding generation.*

Insert không phải điểm nghẽn chính.  
*Insertion is not the main bottleneck.*

COPY giúp database không trở thành nút thắt.  
*COPY prevents the database from becoming a bottleneck.*

Idempotency cho phép pipeline chạy lại.  
*Idempotency lets the pipeline run again.*

Khả năng xác định bottleneck thật thể hiện tư duy staff.  
*Identifying the real bottleneck demonstrates staff-level thinking.*

---
## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 FINAL INTERVIEW REVIEW (Interview Cheatsheet)*

### 5.1. Keywords bắt buộc nhớ
*5.1. Required keywords*

- **Bulk / batch insert** nạp nhiều dòng bằng ít transaction.  
  ***Bulk / batch insert** loads many rows with fewer transactions.*

Phương pháp này nhanh hơn one-by-one.  
*This method is faster than one-by-one insertion.*

- **Transaction overhead** là chi phí cố định của mỗi transaction.  
  ***Transaction overhead** is the fixed cost of each transaction.*

Các chi phí gồm WAL, commit và round-trip.  
*The costs include WAL, commit, and round-trip.*

Transaction overhead là lý do one-by-one chậm.  
*Transaction overhead makes one-by-one insertion slow.*

- **`COPY` / `\copy`** là lệnh nạp khối nhanh nhất.  
  ***`COPY` / `\copy`** is the fastest bulk loading command.*

`COPY` chạy ở server-side.  
*`COPY` runs on the server side.*

`\copy` chạy ở client-side.  
*`\copy` runs on the client side.*

- **`FORMAT BINARY`** là chế độ COPY nhị phân.  
  ***`FORMAT BINARY`** is the binary COPY mode.*

Chế độ này nhanh hơn định dạng text.  
*This mode is faster than the text format.*

- **pg-promise** là một thư viện Node.js.  
  ***pg-promise** is a Node.js library.*

Thư viện dùng `ColumnSet`, `helpers.insert` và `db.none` cho batch.  
*The library uses `ColumnSet`, `helpers.insert`, and `db.none` for batches.*

- **psycopg2 / `execute_values`** là công cụ Python cũ.  
  ***psycopg2 / `execute_values`** is an older Python toolset.*

`execute_values` là một hàm batch insert.  
*`execute_values` is a batch insert function.*

Hàm này nằm trong `extras`.  
*This function belongs to `extras`.*

- **psycopg v3** là thư viện Python mới.  
  ***psycopg v3** is the newer Python library.*

Thư viện này dùng `cursor.copy()` và `executemany()`.  
*This library uses `cursor.copy()` and `executemany()`.*

`copy()` là lựa chọn nhanh nhất.  
*`copy()` is the fastest option.*

Psycopg v3 không còn `execute_values`.  
*Psycopg v3 no longer includes `execute_values`.*

- **`register_vector`** đăng ký kiểu vector.  
  ***`register_vector`** registers the vector type.*

Việc đăng ký giúp list hoặc ndarray serialize thành `vector`.  
*Registration lets a list or ndarray serialize into `vector`.*

- **`commit()`** là bước bắt buộc để lưu dữ liệu.  
  ***`commit()`** is required for data persistence.*

Psycopg2 không bật autocommit mặc định.  
*Psycopg2 does not enable autocommit by default.*

- **Load-then-index** nghĩa là nạp dữ liệu trước.  
  ***Load-then-index** means loading the data first.*

Bạn tạo index sau.  
*You create the index afterward.*

Cách này nhanh hơn nhiều.  
*This approach is much faster.*

- **`maintenance_work_mem`** là RAM dành cho quá trình build index.  
  ***`maintenance_work_mem`** is RAM for the index build process.*

Quá trình build diễn ra sau bước nạp dữ liệu.  
*The build process occurs after data loading.*

- **`ON CONFLICT DO UPDATE`** thực hiện upsert cho idempotency.  
  ***`ON CONFLICT DO UPDATE`** performs an idempotent upsert.*

Mẫu này dùng temp table với COPY.  
*This pattern uses a temp table with COPY.*

- **`CREATE INDEX CONCURRENTLY`** build index mà không khóa write.  
  ***`CREATE INDEX CONCURRENTLY`** builds an index without blocking writes.*

- **`UNLOGGED` table** bỏ qua WAL.  
  ***An `UNLOGGED` table** skips WAL.*

Bảng này nạp dữ liệu nhanh.  
*This table loads data quickly.*

Bảng này không bền sau sự cố.  
*This table is not durable after a failure.*

Bảng này không replicate.  
*This table does not replicate.*

Bạn chỉ nên dùng nó cho staging.  
*You should use it only for staging.*

### 5.2. Core concepts — nếu chỉ nhớ vài điều
*5.2. Core concepts — remember only these points*

1. Mỗi INSERT là một transaction.  
   *Each INSERT is one transaction.*

One-by-one trả overhead hàng triệu lần.  
*One-by-one insertion pays the overhead millions of times.*

Cách này rất chậm.  
*This approach is very slow.*

2. Thứ tự tốc độ là **row-by-row ≪ batch insert ≪ COPY**.  
   *The speed order is **row-by-row ≪ batch insert ≪ COPY**.*

Bạn nên dùng COPY khi có thể.  
*You should use COPY whenever possible.*

3. COPY nhanh nhất nhờ số transaction ít.  
   *COPY is fastest due to fewer transactions.*

COPY hỗ trợ định dạng binary.  
*COPY supports the binary format.*

COPY sử dụng buffer riêng.  
*COPY uses its own buffer.*

COPY vẫn ghi WAL.  
*COPY still writes WAL.*

Cơ chế này bảo vệ dữ liệu.  
*This mechanism protects the data.*

4. **Bạn nạp dữ liệu trước.**  
   ***You load the data first.***

**Bạn build index sau.**  
***You build the index afterward.***

Đây là nguyên tắc vàng của bulk load.  
*This is the golden rule of bulk loading.*

5. Node.js batch sử dụng **pg-promise**.  
   *Node.js batches use **pg-promise**.*

Pg-promise dùng `ColumnSet` và `helpers.insert`.  
*Pg-promise uses `ColumnSet` and `helpers.insert`.*

Python cũ sử dụng psycopg2 `execute_values`.  
*Older Python projects use psycopg2 `execute_values`.*

Python mới sử dụng psycopg3 `copy()`.  
*Newer Python projects use psycopg3 `copy()`.*

`copy()` là lựa chọn nhanh nhất.  
*`copy()` is the fastest option.*

6. **Psycopg3 không còn `execute_values`.**  
   ***Psycopg3 no longer includes `execute_values`.***

Bạn dùng `executemany()` hoặc `copy()`.  
*You use `executemany()` or `copy()`.*

7. Bạn luôn dùng **`register_vector`**.  
   *You always use **`register_vector`**.*

Bạn cũng có thể format vector thành `'[...]'`.  
*You can also format a vector as `'[...]'`.*

Bạn luôn gọi **`commit()`** trong psycopg2.  
*You always call **`commit()`** in psycopg2.*

8. Vector trong CSV phải được **bọc nháy kép**.  
   *A vector in CSV must use **double quotes**.*

Vector chứa dấu phẩy.  
*The vector contains commas.*

9. Idempotency sử dụng mẫu COPY vào temp table.  
   *Idempotency uses the COPY-to-temp-table pattern.*

Sau đó, pipeline chạy `ON CONFLICT DO UPDATE`.  
*The pipeline then runs `ON CONFLICT DO UPDATE`.*

Job có thể chạy lại an toàn.  
*The job can run again safely.*

10. Bottleneck ingestion thật thường là quá trình sinh embedding.  
    *The real ingestion bottleneck is usually embedding generation.*

Insert thường không phải bottleneck chính.  
*Insertion is usually not the main bottleneck.*

Insert kém tạo ra nút thắt thứ hai.  
*Poor insertion creates a second bottleneck.*

### 5.3. Ideas / mental models
*5.3. Ideas and mental models*

- **"Xe tải so với bê tay"** mô tả COPY và row-by-row.  
  ***"Truck versus hand-carrying"** describes COPY and row-by-row insertion.*

Số chuyến transaction quyết định tốc độ.  
*The number of transaction trips determines the speed.*

- **"Nạp trước, index sau"** là một nguyên tắc vận hành.  
  ***"Load first, index second"** is an operational rule.*

Bạn không nên bắt database vừa nạp vừa vá index.  
*You should not make the database load data while repairing the index.*

- **"COPY → temp → upsert"** là mẫu idempotent cho ingestion.  
  ***"COPY → temp → upsert"** is an idempotent ingestion pattern.*

- **"Bottleneck là embedding, không phải insert"** giúp bạn nhìn đúng nút thắt.  
  ***"The bottleneck is embedding, not insertion"** identifies the correct bottleneck.*

- **"Commit hoặc mất trắng"** nhắc bạn về psycopg2.  
  ***"Commit or lose everything"** reminds you about psycopg2.*

Psycopg2 không bật autocommit mặc định.  
*Psycopg2 does not enable autocommit by default.*

### 5.4. Code cần thuộc lòng
*5.4. Code to memorize*

**(a) COPY là cách nhanh nhất.**  
***(a) COPY is the fastest method.***

```sql
\copy faqs (text, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)
```

**(b) Node.js dùng pg-promise cho batch.**  
***(b) Node.js uses pg-promise for batches.***

```javascript
const cs = new pgp.helpers.ColumnSet(['text','embedding'], {table:'faqs'});
await db.none(pgp.helpers.insert(rows, cs));   // 1 câu INSERT nhiều dòng
                                               // One INSERT statement with multiple rows
```

**(c) Python có cách cũ và cách mới.**  
***(c) Python has an old method and a new method.***

```python
# psycopg2 (bài gốc):
# psycopg2 (original lesson):
from psycopg2.extras import execute_values
execute_values(cur, "INSERT INTO faqs (text,embedding) VALUES %s", data); conn.commit()

# psycopg3 (nhanh nhất, khuyến nghị):
# psycopg3 (fastest, recommended):
with cur.copy("COPY faqs (text,embedding) FROM STDIN") as cp:
    cp.set_types(["text","vector"])
    for t,e in rows: cp.write_row([t,e])
conn.commit()
```

**(d) Bạn nạp trước.**  
***(d) You load first.***

**Bạn tạo index sau.**  
***You create the index afterward.***

```sql
-- COPY hết → rồi mới:
-- Finish COPY first, then continue:
SET maintenance_work_mem='2GB';
CREATE INDEX ON faqs USING hnsw (embedding vector_cosine_ops);
```

### 5.5. Câu hỏi phỏng vấn thường gặp và gợi ý trả lời
*5.5. Common interview questions and suggested answers*

1. **"Vì sao bulk insert nhanh hơn insert từng dòng?"**  
   ***"Why is bulk insert faster than row-by-row insertion?"***

Mỗi INSERT là một transaction.  
*Each INSERT is one transaction.*

Mỗi transaction có overhead cố định.  
*Each transaction has fixed overhead.*

Overhead gồm WAL, commit và round-trip.  
*The overhead includes WAL, commit, and round-trip.*

Bulk insert gom nhiều dòng vào ít transaction.  
*Bulk insert groups many rows into fewer transactions.*

Cách này giảm overhead nhiều lần.  
*This approach reduces the overhead many times.*

2. **[XẾP HẠNG] "COPY, batch INSERT, row-by-row có thứ tự nào?"**  
   ***[RANKING] "What is the order of COPY, batch INSERT, and row-by-row?"***

COPY là phương pháp nhanh nhất.  
*COPY is the fastest method.*

COPY dùng ít transaction.  
*COPY uses fewer transactions.*

COPY hỗ trợ binary.  
*COPY supports binary data.*

COPY sử dụng buffer riêng.  
*COPY uses its own buffer.*

Batch hoặc multi-row INSERT đứng thứ hai.  
*Batch or multi-row INSERT ranks second.*

Row-by-row đứng cuối.  
*Row-by-row insertion ranks last.*

Bạn nên dùng COPY khi có thể.  
*You should use COPY whenever possible.*

3. **[BẪY] "Psycopg3 dùng `execute_values` như thế nào?"**  
   ***[TRICK] "How does psycopg3 use `execute_values`?"***

Đây là một câu hỏi bẫy.  
*This is a trick question.*

Psycopg3 không có `execute_values`.  
*Psycopg3 does not include `execute_values`.*

Bạn dùng `executemany()` cho pipeline.  
*You use `executemany()` for pipelining.*

Bạn dùng `cursor.copy()` cho tốc độ cao nhất.  
*You use `cursor.copy()` for the highest speed.*

`execute_values` thuộc psycopg2.  
*`execute_values` belongs to psycopg2.*

4. **[STAFF] "Làm sao nạp 10 triệu vector nhanh nhất?"**  
   ***[STAFF] "How do you load 10 million vectors fastest?"***

Bạn dùng COPY binary.  
*You use binary COPY.*

Bạn nạp vào bảng chưa có index.  
*You load into a table without an index.*

Sau đó, bạn chạy `CREATE INDEX CONCURRENTLY`.  
*You then run `CREATE INDEX CONCURRENTLY`.*

Bạn đặt `maintenance_work_mem` ở mức cao.  
*You set `maintenance_work_mem` to a high value.*

Bạn dùng parallel workers.  
*You use parallel workers.*

Bạn sinh embedding theo batch trước.  
*You generate embeddings in batches first.*

Bạn dùng temp table cho idempotency.  
*You use a temp table for idempotency.*

Bạn dùng upsert sau bước COPY.  
*You use an upsert after COPY.*

5. **[BẪY] "Tại sao build index trước bulk insert lại chậm?"**  
   ***[TRICK] "Why is index creation before bulk insert slow?"***

Mỗi insert phải cập nhật index.  
*Each insertion must update the index.*

HNSW chịu ảnh hưởng đặc biệt lớn.  
*HNSW suffers a particularly large impact.*

Bạn nên nạp dữ liệu trước.  
*You should load the data first.*

Bạn chỉ build index một lần sau đó.  
*You build the index only once afterward.*

Cách này nhanh hơn nhiều.  
*This approach is much faster.*

6. **[BẪY] "Tại sao truy vấn không thấy dữ liệu sau insert trong Python?"**  
   ***[TRICK] "Why is inserted data missing from a Python query?"***

Bạn có thể đã quên `conn.commit()`.  
*You may have forgotten `conn.commit()`.*

Psycopg2 không bật autocommit mặc định.  
*Psycopg2 does not enable autocommit by default.*

Connection rollback dữ liệu lúc đóng.  
*The connection rolls back the data at close.*

7. **"Bulk insert có làm mất replication hoặc recovery không?"**  
   ***"Does bulk insert remove replication or recovery?"***

Bảng thường vẫn hỗ trợ hai cơ chế này.  
*Regular tables still support both mechanisms.*

COPY vẫn ghi WAL.  
*COPY still writes WAL.*

`UNLOGGED` table bỏ qua WAL.  
*An `UNLOGGED` table skips WAL.*

Bảng này nạp nhanh hơn.  
*This table loads faster.*

Bảng này mất dữ liệu sau crash.  
*This table loses data after a crash.*

Bảng này không replicate.  
*This table does not replicate.*

Bạn chỉ dùng nó cho staging.  
*You use it only for staging.*

8. **"Làm sao để job nạp chạy lại an toàn?"**  
   ***"How can an ingestion job run again safely?"***

Bạn thiết kế idempotency.  
*You design for idempotency.*

Bạn COPY dữ liệu vào temp table.  
*You COPY the data into a temp table.*

Sau đó, bạn chạy `INSERT ... SELECT ... ON CONFLICT DO UPDATE`.  
*You then run `INSERT ... SELECT ... ON CONFLICT DO UPDATE`.*

Retry không tạo dữ liệu trùng.  
*The retry does not create duplicate data.*

### 5.6. One-liner đắt giá
*5.6. Valuable one-liners*

- *"Mỗi INSERT là một transaction."*  
  *"Each INSERT is a transaction."*

*"Bulk loading chỉ trả overhead một lần."*  
*"Bulk loading pays the overhead only once."*

*"Bulk loading không trả overhead một triệu lần."*  
*"Bulk loading does not pay the overhead a million times."*

- *"COPY là cách nhanh nhất để nạp rows vào Postgres."*  
  *"COPY is the fastest way to get rows into Postgres."*

*"COPY dùng ít transaction hơn."*  
*"COPY uses fewer transactions."*

*"COPY dùng binary protocol."*  
*"COPY uses the binary protocol."*

*"COPY sử dụng buffer riêng."*  
*"COPY uses its own buffer."*

- *"Bạn nạp dữ liệu trước."*  
  *"Load first."*

*"Bạn tạo index sau."*  
*"Index second."*

*"Đừng bắt database duy trì graph HNSW trong lúc nạp hàng triệu rows."*  
*"Never make the database maintain an HNSW graph during a million-row load."*

- *"Psycopg3 đã loại bỏ execute_values."*  
  *"Psycopg3 dropped execute_values."*

*"Executemany được tối ưu sẵn."*  
*"Optimized executemany already performs well."*

*"Copy() cũng được tối ưu sẵn."*  
*"Optimized copy() already performs well."*

*"Bạn nên dùng copy() ở quy mô lớn."*  
*"Reach for copy() at scale."*

- *"Bạn COPY dữ liệu vào temp table."*  
  *"COPY into a temp table."*

*"Sau đó, bạn thực hiện upsert."*  
*"Then perform an upsert."*

*"Mẫu này giúp job ingestion nhiều giờ chạy lại an toàn."*  
*"This pattern makes a multi-hour ingestion job safe to rerun."*

- *"Bottleneck thật của vector ingestion thường là embedding generation."*  
  *"The real bottleneck in vector ingestion is usually embedding generation."*

*"Insert thường không phải bottleneck thật."*  
*"Insertion is usually not the real bottleneck."*

*"Insert ngây thơ tạo ra bottleneck thứ hai."*  
*"A naive insertion method creates the second bottleneck."*

---

### 📌 Ghi chú cuối
*📌 Final notes*

- **Đính chính này giúp bạn nhớ đúng.**  
  ***This correction helps you remember the facts accurately.***

**COPY là phương pháp nhanh nhất.**  
***COPY is the fastest method.***

COPY không ngang hàng với batch insert.  
*COPY is not equal to batch insert.*

**Psycopg3 thay thế psycopg2.**  
***Psycopg3 replaces psycopg2.***

Psycopg3 không có `execute_values`.  
*Psycopg3 does not include `execute_values`.*

Psycopg3 dùng `copy()` hoặc `executemany()`.  
*Psycopg3 uses `copy()` or `executemany()`.*

Bạn nạp dữ liệu trước.  
*You load the data first.*

Bạn tạo index sau.  
*You create the index afterward.*

Bạn cần nhớ `register_vector`.  
*You need to remember `register_vector`.*

Bạn cũng cần nhớ `commit()`.  
*You also need to remember `commit()`.*

- **Bạn nên kiểm chứng kiến thức trong lúc ôn.**  
  ***You should verify the material during review.***

Bạn xem tài liệu psycopg.org v3 về `copy()`.  
*You review the psycopg.org v3 documentation for `copy()`.*

Bạn xem pgvector README về ví dụ bulk load hiện hành.  
*You review the pgvector README for current bulk load examples.*

Bạn xem tài liệu pg-promise về `ColumnSet` và `helpers.insert`.  
*You review the pg-promise documentation for `ColumnSet` and `helpers.insert`.*

- **Bạn nên thực hành với một bảng mẫu.**  
  ***You should practice with a sample table.***

Bạn tạo bảng `faqs(text, embedding vector(384))`.  
*You create the `faqs(text, embedding vector(384))` table.*

Bảng này chưa có index.  
*This table has no index.*

Bạn sinh khoảng 100k embedding thật.  
*You generate about 100k real embeddings.*

Bạn dùng kiến thức từ giáo trình embeddings.  
*You use knowledge from the embeddings course.*

Bạn nạp dữ liệu bằng row-by-row.  
*You load the data with row-by-row insertion.*

Bạn nạp dữ liệu bằng execute_values hoặc pg-promise.  
*You load the data with execute_values or pg-promise.*

Bạn nạp dữ liệu bằng COPY.  
*You load the data with COPY.*

Bạn đo thời gian của cả ba cách.  
*You measure the time for all three methods.*

Kết quả giúp bạn thấy sự khác biệt.  
*The results show you the difference.*

Sau đó, bạn chạy `CREATE INDEX ... hnsw`.  
*You then run `CREATE INDEX ... hnsw`.*

Cuối cùng, bạn chạy `EXPLAIN ANALYZE`.  
*Finally, you run `EXPLAIN ANALYZE`.*

- **Series có đủ 7 mảnh kiến thức.**  
  ***The series contains all seven knowledge pieces.***

Mạch bắt đầu từ embed.  
*The sequence starts with embedding.*

Bước tiếp theo là index.  
*The next step is indexing.*

Sau đó, bạn cài pgvector.  
*You then install pgvector.*

Tiếp theo, bạn học store và query trong bài số 3.  
*Next, you learn storage and querying in lesson 3.*

Bài này trình bày bulk insert.  
*This lesson presents bulk insert.*

Bước tiếp theo là FTS.  
*The next step is FTS.*

Bước cuối là chọn nền tảng.  
*The final step is platform selection.*

Bây giờ, bạn có thể nạp dữ liệu quy mô lớn.  
*You can now load data at scale.*

Dữ liệu phục vụ semantic search hoặc RAG.  
*The data supports semantic search or RAG.*

Quá trình nạp đạt hiệu quả cao.  
*The loading process is highly efficient.*

- **Bạn có thể học tiếp streaming ingestion.**  
  ***You can continue with streaming ingestion.***

Pipeline mẫu là Kafka → embed → COPY.  
*The sample pipeline is Kafka → embed → COPY.*

Bạn có thể học incremental re-embedding.  
*You can study incremental re-embedding.*

Kỹ thuật này hữu ích sau khi đổi model.  
*This technique is useful after a model change.*

Bạn cũng có thể tối ưu build HNSW song song.  
*You can also optimize parallel HNSW builds.*

Quy mô mục tiêu là hàng chục triệu vector.  
*The target scale is tens of millions of vectors.*
