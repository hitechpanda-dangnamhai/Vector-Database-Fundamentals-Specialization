# PostgreSQL: Cơ sở dữ liệu quan hệ & Full-Text Search (tsvector / tsquery)
### Giáo trình SIÊU DỄ HIỂU — giải thích tận gốc mọi thuật ngữ | Basic → Staff

> **Bài giảng gốc:** *"Refresh your knowledge of relational databases with a focus on PostgreSQL"* — Course 3, IBM Vector Database Fundamentals.
>
> **Cách đọc giáo trình này:** Đọc tuần tự từ trên xuống. Mọi từ chuyên ngành đều được giải thích **ngay lần đầu nó xuất hiện**, nên bạn không cần biết gì trước. Nếu gặp một từ lạ mà chưa thấy giải thích — đó là lỗi của người viết, không phải lỗi của bạn.
>
> **Ký hiệu trong bài:**
> - 🧩 **[Ngoài bài gốc]** — phần bài giảng gốc không nói, nhưng một staff engineer bắt buộc phải biết.
> - 💡 **Mẹo thực chiến** — kinh nghiệm khi làm thật ở production (production = hệ thống đang chạy thật, phục vụ người dùng thật, hỏng là có người bị ảnh hưởng).
> - ⚠️ **Chỗ khó** — chỗ nhiều người vấp. Vấp ở đây là bình thường.
>
> **Cập nhật thông tin tới tháng 7/2026:** Bản PostgreSQL ổn định mới nhất là **18.4**; các nhánh 17 / 16 / 15 / 14 vẫn được hỗ trợ. **PostgreSQL 19 Beta 2** ra ngày 16/07/2026, bản chính thức dự kiến tháng 9/2026. Với full-text search, **GIN vẫn là loại index chuẩn**, và cách làm được khuyến nghị hiện nay là dùng **generated column (STORED)** hoặc **functional index** thay vì tự viết trigger đồng bộ bằng tay. (Các từ in đậm này sẽ được giải thích đầy đủ ở dưới — bạn chưa cần hiểu ngay lúc này.)

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### 0.1. Một câu tóm tắt siêu đơn giản

**Bài này dạy bạn cách bắt cơ sở dữ liệu tìm chữ trong văn bản một cách thông minh** — nghĩa là khi người dùng gõ "running", hệ thống vẫn tìm ra được những bài viết chỉ chứa chữ "ran" hoặc "runs", và tìm ra **rất nhanh** kể cả khi bạn có hàng chục triệu bài viết.

Nếu vài từ ở trên bạn chưa hiểu (cơ sở dữ liệu là gì? tại sao tìm chữ lại chậm?), điều đó hoàn toàn bình thường và đúng như dự kiến. Phần 1 sẽ dựng lại từng viên gạch một, bắt đầu từ con số không.

### 0.2. Bài này gồm hai tầng nội dung

**Tầng thứ nhất — ôn lại nền móng.** Cơ sở dữ liệu quan hệ tổ chức dữ liệu như thế nào: bảng là gì, khóa chính và khóa ngoại là gì, PostgreSQL có những kiểu dữ liệu nào, và vì sao PostgreSQL đặc biệt so với các cơ sở dữ liệu khác.

**Tầng thứ hai — trọng tâm thật sự của bài.** Kỹ thuật **full-text search** (tìm kiếm toàn văn): cách PostgreSQL biến một đoạn văn bản thô thành một cấu trúc gọi là `tsvector`, rồi so khớp nó với một truy vấn gọi là `tsquery`, để tìm được tài liệu theo **nghĩa của từ** thay vì so từng ký tự một cách máy móc.

### 0.3. Vấn đề thực tế mà bài này giải quyết

Bạn có một ô tìm kiếm trên website. Người dùng gõ vào chữ "running". Cách làm ngây thơ nhất là bảo cơ sở dữ liệu "tìm những dòng nào có chứa chuỗi ký tự *running*". Cách này hỏng theo hai kiểu cùng lúc: nó **chậm khủng khiếp** khi dữ liệu lớn, và nó **ngu ngốc** — nó không biết "ran", "runs", "running" là cùng một từ, trong khi lại tưởng "b**run**ch" có liên quan tới "run".

Full-text search sinh ra để chữa cả hai bệnh đó cùng lúc. Toàn bộ Phần 1 sẽ mổ xẻ chính xác vì sao cách ngây thơ kia hỏng, trước khi giới thiệu liều thuốc.

### 0.4. Học xong bạn sẽ làm được gì

- Giải thích rành mạch cơ sở dữ liệu, DBMS, bảng, khóa chính, khóa ngoại, BLOB — và vì sao PostgreSQL được gọi là "mở rộng được".
- Chọn đúng kiểu dữ liệu của PostgreSQL cho từng nhu cầu (và tránh cái bẫy lưu tiền bằng số thực).
- Hiểu bản chất lexeme, `tsvector`, `tsquery`, toán tử `@@`, và tự viết được câu lệnh tìm kiếm chạy được thật.
- Đánh index đúng cách để tìm kiếm nhanh, và xếp hạng kết quả theo độ liên quan.
- Phân tích được khi nào PostgreSQL là đủ và khi nào cần tới hệ thống tìm kiếm chuyên dụng — và trả lời được câu hỏi thiết kế hệ thống về search trong phỏng vấn.

### 0.5. Mạch kiến thức từ dễ tới khó

- 🟢 **Basic** — nền móng cơ sở dữ liệu → vì sao cách tìm bằng `LIKE` gãy → khái niệm lexeme → ví dụ "hello world" đầu tiên.
- 🟡 **Intermediate** — bên trong PostgreSQL, văn bản đi qua những chặng nào → chọn đúng hàm tạo truy vấn → đánh index GIN → xếp hạng kết quả → ba lỗi kinh điển.
- 🔴 **Advanced** — ruột gan của inverted index → GIN so với GiST → độ phức tạp thuật toán → chống lỗi chính tả → các trường hợp biên.
- 🟣 **Staff** — chạy với hàng chục triệu tài liệu thì gãy ở đâu → PostgreSQL hay Elasticsearch → hybrid search (kết hợp tìm theo từ khóa và tìm theo ngữ nghĩa) → chi phí, vận hành, ảnh hưởng tổ chức → một câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — bảng thuật ngữ, ý cốt lõi, code cần thuộc, câu hỏi phỏng vấn kèm gợi ý trả lời.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người **chưa biết gì**. Nó là phần dài nhất và chậm nhất trong giáo trình, và điều đó là cố ý. Nếu bạn đã biết cơ sở dữ liệu quan hệ, bạn vẫn nên lướt qua mục 1.4 trở đi.

### 1.1. Bắt đầu từ vấn đề: vì sao con người cần cơ sở dữ liệu

Hãy tưởng tượng bạn mở một cửa hàng bán quần áo online. Ngày đầu tiên có 3 khách đặt hàng. Bạn ghi vào một cuốn sổ tay: tên khách, số điện thoại, món hàng, ngày giao. Không vấn đề gì.

Sáu tháng sau bạn có 50.000 khách. Giờ hãy thử trả lời câu hỏi: *"Khách hàng nào đã mua áo khoác trong tháng trước và chưa được giao hàng?"*. Với cuốn sổ tay, bạn phải lật từng trang, đọc từng dòng — mất vài ngày, và chắc chắn sót.

Đó chính là **vấn đề** mà cơ sở dữ liệu sinh ra để giải: lưu trữ lượng lớn dữ liệu sao cho việc **tìm, lọc, cập nhật** vẫn nhanh khi dữ liệu phình to.

### 1.2. Các thuật ngữ nền móng — giải thích từng từ một

**Database** (đọc là "đây-ta-bây", dịch: *cơ sở dữ liệu*) là một tập hợp dữ liệu được tổ chức có quy củ và lưu trữ dưới dạng số hóa để dùng đi dùng lại nhiều lần. Từ "có tổ chức" là mấu chốt: một đống file lộn xộn trong máy tính không phải database; một tập dữ liệu được sắp xếp theo cấu trúc rõ ràng thì mới là.

**DBMS** (viết tắt của *Database Management System*, dịch: *hệ quản trị cơ sở dữ liệu*) là **phần mềm** đứng ra quản lý cái database đó. Nó cho phép bạn truy cập, sửa, thêm, xóa dữ liệu một cách nhanh chóng và an toàn. Ví dụ: PostgreSQL, MySQL, Oracle Database.

Hãy phân biệt cho rõ hai từ trên, vì rất nhiều người dùng lẫn lộn: **database là dữ liệu**, còn **DBMS là phần mềm trông coi dữ liệu đó**. Giống như "sách" và "thủ thư" là hai thứ khác nhau.

Một chi tiết đáng chú ý: bên dưới, DBMS thật ra lưu dữ liệu thành các **file** trên ổ đĩa (giống file trong máy tính bạn), nhưng nó **trình bày** cho bạn thấy dữ liệu dưới dạng **bảng** cho dễ hình dung và dễ truy vấn. Bạn làm việc với bảng, còn chuyện file nằm ở đâu trên đĩa là việc của nó.

**RDBMS** (viết tắt của *Relational Database Management System*, dịch: *hệ quản trị cơ sở dữ liệu quan hệ*) là loại DBMS tổ chức dữ liệu thành các bảng **có quan hệ với nhau**. Chữ "relational" (quan hệ) chính là điểm mấu chốt, và mục 1.3 sẽ cho bạn thấy "quan hệ" nghĩa là gì bằng ví dụ cụ thể.

**Table** (dịch: *bảng*) là dữ liệu được xếp thành hàng và cột, y hệt một trang Excel. Mỗi **column** (*cột*) là một loại thông tin, ví dụ cột "tên khách". Mỗi **row** (*hàng*, còn gọi là **record** — *bản ghi*) là một cá thể cụ thể, ví dụ một khách hàng tên Lan.

**Entity** (dịch: *thực thể*) là cách gọi mang tính khái niệm cho "một thứ có thật mà một hàng đang mô tả". Một hàng trong bảng `customers` đại diện cho một thực thể là một con người có thật. Từ này hay xuất hiện trong tài liệu thiết kế, nên bạn nên quen mặt.

**Schema** (đọc "ski-ma", dịch: *lược đồ*) là bản thiết kế của database: có những bảng nào, mỗi bảng có cột gì, kiểu dữ liệu ra sao. Nếu bảng là căn nhà thì schema là bản vẽ kiến trúc.

**Query** (dịch: *truy vấn*) là một câu lệnh bạn gửi cho DBMS để hỏi hoặc thay đổi dữ liệu. **SQL** (viết tắt của *Structured Query Language*, đọc là "ét-quờ-eo" hoặc "sí-cồ") là ngôn ngữ chuẩn để viết những câu truy vấn đó. Ví dụ `SELECT name FROM customers;` nghĩa là "lấy cột name từ bảng customers".

### 1.3. Khóa chính và khóa ngoại — trái tim của chữ "quan hệ"

Đây là chỗ nhiều người mới bị rối, nên ta đi thật chậm bằng một ví dụ cụ thể, chính là ví dụ mà bài giảng gốc dùng: một nhà bán lẻ (retailer) với bốn bảng `customers` (khách hàng), `orders` (đơn hàng), `orderline_items` (các dòng trong đơn hàng), `items` (mặt hàng).

**Primary key** (viết tắt **PK**, dịch: *khóa chính*) là cột — hoặc nhóm cột — dùng để **định danh duy nhất** mỗi hàng trong bảng. "Định danh duy nhất" nghĩa là: không có hai hàng nào được phép trùng giá trị ở cột đó, và nhìn vào giá trị đó là biết chính xác đang nói tới hàng nào.

Ví dụ trong bảng `customers`, cột `cust_id` là khóa chính. Khách Lan có `cust_id = 101`, khách Minh có `cust_id = 102`. Không bao giờ có hai khách cùng mang số 101.

Vì sao không dùng luôn tên khách làm khóa chính? Vì tên bị trùng — có hàng nghìn người tên "Nguyễn Văn An". Và tên còn có thể đổi. Khóa chính cần một giá trị **không trùng và không đổi**, nên người ta thường tạo hẳn một cột số riêng cho việc đó.

**Foreign key** (viết tắt **FK**, dịch: *khóa ngoại*) là một cột trong bảng này chứa giá trị **trỏ tới khóa chính của một bảng khác**. Chính cái mũi tên trỏ qua trỏ lại này tạo ra **relationship** (*mối quan hệ*) giữa các bảng — và đó là lý do người ta gọi loại cơ sở dữ liệu này là "quan hệ".

Cụ thể: bảng `orders` cũng có một cột tên `cust_id`. Nhưng ở đây nó **không phải** khóa chính — nó là khóa ngoại, trỏ ngược về bảng `customers`. Nếu một đơn hàng có `cust_id = 101`, ta biết ngay đơn đó là của chị Lan.

Cứ hình dung thế này để cho dễ nhớ: **khóa chính giống số căn cước công dân** — mỗi người một số, không ai trùng ai. **Khóa ngoại giống chỗ ghi "số căn cước của người nhận" trên một tờ biên nhận** — bản thân tờ biên nhận không phải là con người, nhưng nó *tham chiếu* tới một con người cụ thể qua con số ấy.

Phép ví von này khớp ở đúng chỗ quan trọng nhất: bạn không cần chép lại toàn bộ thông tin cá nhân (tên, địa chỉ, số điện thoại) lên từng tờ biên nhận. Bạn chỉ ghi một con số, và khi cần thì tra ngược lại. Cơ sở dữ liệu quan hệ hoạt động y hệt: **không lặp lại dữ liệu, chỉ tham chiếu tới nó**.

Lợi ích cụ thể của việc không lặp lại: khi chị Lan đổi số điện thoại, bạn chỉ sửa **một chỗ duy nhất** trong bảng `customers`. Nếu trước đó bạn đã chép số điện thoại vào 500 đơn hàng, bạn sẽ phải sửa 500 chỗ và chắc chắn sót vài chỗ.

**JOIN** (dịch thô: *nối*) là phép toán trong SQL dùng để ghép hai bảng lại với nhau theo đúng cặp khóa chính — khóa ngoại đó, để trả lời những câu hỏi kiểu "khách nào đã đặt đơn nào". Bạn chưa cần viết được JOIN ngay, chỉ cần biết đó là công cụ khai thác quan hệ giữa các bảng.

### 1.4. BLOB — khi dữ liệu không phải là chữ và số

**BLOB** (viết tắt của *Binary Large Object*, dịch: *đối tượng nhị phân lớn*) là cách lưu những dữ liệu phức tạp — ảnh, video, file PDF, file âm thanh — ngay bên trong cơ sở dữ liệu, dưới dạng **nhị phân**.

**Nhị phân (binary)** ở đây nghĩa là dữ liệu được lưu nguyên xi thành một chuỗi các bit 0 và 1, chứ không được diễn giải thành chữ hay số. Cơ sở dữ liệu không hiểu nội dung bức ảnh là gì, nó chỉ giữ hộ bạn nguyên khối byte đó và trả lại y nguyên khi bạn hỏi.

🧩 **[Ngoài bài gốc]** Bài gốc chỉ giới thiệu BLOB tồn tại. Nhưng ở thực chiến, câu hỏi thú vị hơn là *có nên* nhét ảnh vào database hay không. Câu trả lời của phần lớn hệ thống hiện đại là **không**: người ta lưu file lên một dịch vụ lưu trữ riêng (như Amazon S3 — một kho chứa file trên đám mây), rồi trong database chỉ lưu **đường dẫn** tới file đó. Lý do: BLOB làm database phình to rất nhanh, khiến việc sao lưu (backup) chậm và tốn kém, trong khi dịch vụ lưu trữ file chuyên dụng thì rẻ hơn nhiều. Đây là một câu hỏi phỏng vấn hay gặp, và biết nói ra trade-off này là điểm cộng.

### 1.5. Các RDBMS phổ biến, và PostgreSQL đặc biệt ở chỗ nào

Những cái tên bạn sẽ gặp trong nghề: **IBM Db2, Oracle Database, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, MariaDB, SAP HANA, Amazon RDS**. Trong đó có loại thương mại (phải trả tiền bản quyền) và loại **open-source** (*mã nguồn mở* — mã nguồn công khai, dùng miễn phí), có loại chạy trên máy chủ của chính công ty bạn và có loại chạy trên đám mây.

PostgreSQL (thường gọi tắt là **Postgres**) được xếp vào nhóm **ORDBMS** (viết tắt của *Object-Relational Database Management System*, dịch: *hệ quản trị cơ sở dữ liệu quan hệ - đối tượng*). Nghĩa là nó vẫn là cơ sở dữ liệu quan hệ bình thường, nhưng được bổ sung thêm các tính năng theo hướng "đối tượng".

Trong tất cả những tính năng đó, thứ quan trọng nhất với chúng ta là: **PostgreSQL mở rộng được**. Tiếng Anh gọi là **extensibility** (*tính mở rộng*). Cụ thể, bạn có thể cắm thêm vào Postgres những thứ mà nó vốn không có sẵn: kiểu dữ liệu mới, hàm mới, thậm chí cả **phương pháp đánh index mới**.

**Extension** (dịch: *phần mở rộng*) là gói tính năng do bên thứ ba viết, cài thêm vào Postgres bằng một câu lệnh. Cứ hình dung Postgres như một chiếc điện thoại: phần cứng là cố định, nhưng bạn cài thêm ứng dụng để nó làm được những việc mà nhà sản xuất chưa nghĩ tới.

Hai ví dụ trực tiếp liên quan tới khóa học này:

- **`pgvector`** — một extension thêm kiểu dữ liệu `vector` vào Postgres, để làm **vector search** (tìm kiếm theo ngữ nghĩa — nội dung của giáo trình trước).
- **Full-text search** — thêm hai kiểu dữ liệu `tsvector` và `tsquery`, cùng các phương pháp index tên là GIN và GiST, để tìm kiếm theo từ khóa (nội dung của giáo trình này).

🧩 **[Ngoài bài gốc]** Đừng coi "extensibility" là một chi tiết kỹ thuật vặt vãnh. Nó là **lý do chiến lược** để một công ty chọn Postgres. Khi cần tìm kiếm ngữ nghĩa, bạn không phải mua thêm và vận hành thêm một cơ sở dữ liệu vector riêng; bạn chỉ cắm thêm khả năng đó vào cái database bạn đã có. Ít hệ thống hơn nghĩa là ít thứ để hỏng hơn, ít người phải trực đêm hơn, và ít tiền hơn. Luận điểm này sẽ quay lại rất mạnh ở Phần 4.

### 1.6. Các kiểu dữ liệu của PostgreSQL — nhớ theo nhóm, đừng học vẹt

**Data type** (dịch: *kiểu dữ liệu*) là việc bạn khai báo trước cho database biết một cột sẽ chứa loại thông tin gì: số nguyên, chữ, ngày tháng... Việc khai báo này giúp database lưu trữ hiệu quả hơn và ngăn dữ liệu rác lọt vào (bạn không thể nhét chữ "abc" vào một cột khai là số).

| Nhóm | Các kiểu tiêu biểu | Dùng cho |
|---|---|---|
| **Numeric** (số) | `integer`, `bigint`, `real`, `double precision`, `numeric` | Số lượng, tuổi, giá tiền |
| **Character** (chữ) | `text`, `varchar(n)`, `char(n)` | Tên, mô tả, nội dung bài viết |
| **Boolean** (đúng/sai) | `boolean` | Cờ bật/tắt, đã thanh toán hay chưa |
| **Date/Time** (ngày giờ) | `date`, `time`, `timestamp`, `timestamptz`, `interval` | Ngày đặt hàng, thời điểm log |
| **Array** (mảng) | `text[]`, `integer[]` | Một cột chứa nhiều giá trị, ví dụ danh sách tag |
| **Enumerated** (`enum`) | `mood AS ENUM ('sad','ok','happy')` | Tập giá trị cố định, không được phép sai chính tả |
| **Chuyên biệt** | `money`, `point`/`polygon`, `inet`/`cidr`, `uuid`, `json`/`jsonb` | Tiền, tọa độ hình học, địa chỉ mạng, mã định danh, dữ liệu dạng JSON |
| **Qua extension** | `vector`, **`tsvector`/`tsquery`** | Tìm kiếm ngữ nghĩa / **tìm kiếm toàn văn** |

Giải thích vài từ trong bảng để không ai bị bỏ lại:

**`integer` và `bigint`** đều là số nguyên; `bigint` chứa được số lớn hơn rất nhiều. Chọn sai sẽ dẫn tới sự cố kinh điển: hệ thống chạy êm vài năm rồi đột nhiên gãy khi số đơn hàng vượt quá 2,1 tỷ — giới hạn của `integer`.

**`real` và `double precision`** là số thực **dấu phẩy động** (floating point). "Dấu phẩy động" nghĩa là máy tính lưu số gần đúng chứ không tuyệt đối chính xác, đổi lại tính toán rất nhanh.

**`numeric`** cũng là số thực nhưng lưu **chính xác tuyệt đối**, chậm hơn một chút.

**`varchar(n)` và `text`** đều lưu chữ; `varchar(n)` giới hạn tối đa `n` ký tự còn `text` thì không giới hạn. Trong Postgres, hai kiểu này gần như không khác nhau về hiệu năng, nên cứ dùng `text` cho tiện.

**`timestamptz`** là timestamp **có kèm múi giờ** (tz = time zone). Đây là kiểu bạn nên dùng mặc định cho mọi mốc thời gian, vì hệ thống thật luôn có người dùng ở nhiều múi giờ.

**`json`/`jsonb`** lưu dữ liệu dạng JSON — một định dạng văn bản để biểu diễn dữ liệu có cấu trúc lồng nhau. `jsonb` là bản đã được xử lý sẵn thành dạng nhị phân nên truy vấn nhanh hơn; gần như luôn chọn `jsonb`.

⚠️ **Chỗ khó — cái bẫy tiền bạc kinh điển.** Với dữ liệu là **tiền**, hãy dùng `numeric` (hoặc `money`), **tuyệt đối không** dùng `real`/`double precision`. Lý do: số dấu phẩy động lưu gần đúng, nên `0.1 + 0.2` trong máy tính ra `0.30000000000000004` chứ không phải `0.3`. Cộng dồn qua hàng triệu giao dịch, sổ sách kế toán sẽ lệch. Đây là lỗi mà rất nhiều lập trình viên mới mắc, và nó khó phát hiện vì lúc test với vài dòng dữ liệu thì trông vẫn đúng.

### 1.7. Vấn đề thực tế: vì sao cách tìm kiếm ngây thơ lại hỏng

Giờ ta bước vào trọng tâm của bài. Giả sử bạn có bảng `articles` (bài viết) với một cột `body` kiểu `text` chứa nội dung bài. Người dùng gõ vào ô tìm kiếm chữ "running".

Cách làm đầu tiên ai cũng nghĩ ra là dùng toán tử `LIKE` của SQL:

```sql
SELECT title FROM articles WHERE body LIKE '%running%';
```

**`LIKE`** là toán tử so khớp chuỗi ký tự trong SQL. Dấu **`%`** là **wildcard** (*ký tự đại diện*), nghĩa là "chỗ này có thể là bất cứ thứ gì, dài ngắn tùy ý". Vậy `'%running%'` nghĩa là: "chuỗi nào có chứa *running* ở đâu đó bên trong, trước và sau nó là gì cũng được".

Câu lệnh này chạy được. Nhưng nó hỏng theo ba cách khác nhau.

**Vấn đề thứ nhất — chậm và không thể mở rộng.**

Để hiểu vì sao chậm, cần biết **index** (*chỉ mục*) là gì. Index là một cấu trúc dữ liệu phụ mà database dựng thêm bên cạnh bảng, nhằm giúp tìm kiếm nhanh hơn — hoàn toàn giống phần mục lục tra cứu ở cuối một cuốn sách. Loại index mặc định của hầu hết database tên là **B-tree** (*cây B*), nó sắp xếp giá trị theo thứ tự nên tìm rất nhanh **nếu bạn biết đoạn đầu của giá trị cần tìm**.

Và đó chính là vấn đề: `'%running%'` có dấu `%` ở **đầu**, tức là bạn không biết chuỗi bắt đầu bằng gì. B-tree trở nên vô dụng, giống như bạn tra từ điển mà chỉ biết "từ này có chứa vần *ung* ở giữa" — bạn buộc phải lật hết từ điển.

Khi index không dùng được, database rơi vào chế độ **sequential scan** (*quét tuần tự*, viết tắt là seq scan): đọc lần lượt **từng hàng một** trong bảng và so khớp. Nếu bảng có 10 triệu hàng, nó đọc đủ 10 triệu hàng. Thời gian chạy tỉ lệ thuận với **tổng số hàng** — bảng càng lớn càng chậm, không có cách nào cứu.

**Vấn đề thứ hai — nó không hiểu biến thể của từ.**

Tiếng Anh biến đổi từ liên tục: "run", "runs", "ran", "running" đều là cùng một hành động. Nhưng `LIKE '%running%'` so khớp **từng ký tự một**, nên nó chỉ tìm ra đúng chuỗi "running". Một bài viết hay ho có câu "he ran a marathon" sẽ bị bỏ sót hoàn toàn, dù nó đúng là thứ người dùng đang cần.

**Vấn đề thứ ba — nó khớp nhầm.**

Nếu bạn nới lỏng thành `LIKE '%run%'` để bắt được nhiều biến thể hơn, bạn lập tức khớp trúng cả "b**run**ch" (bữa xế), "p**run**e" (mận khô), "**run**ny nose" (sổ mũi). Kết quả trả về đầy rác, vì `LIKE` không biết đâu là ranh giới của một từ — với nó, mọi thứ chỉ là một dãy ký tự dài.

**Tóm lại:** `LIKE` thất bại vì nó làm việc ở tầng **ký tự**. Muốn tìm kiếm văn bản cho tử tế, ta cần một công cụ làm việc ở tầng **từ** — và hiểu được rằng các biến thể của một từ thì thuộc về cùng một khái niệm.

### 1.8. Analogy: mục lục cuối cuốn sách

Trước khi vào định nghĩa kỹ thuật, hãy dựng cho xong một phép ví von, vì nó sẽ theo bạn suốt phần còn lại của giáo trình — và cả trong phòng phỏng vấn.

Bạn cầm một cuốn sách kỹ thuật dày 800 trang và muốn biết chỗ nào nói về "database". Có hai cách.

**Cách một:** mở trang 1, đọc từng dòng cho tới trang 800, ghi lại mọi chỗ thấy chữ "database". Cách này luôn cho kết quả đúng, nhưng mất cả ngày. **Đây chính xác là `LIKE '%database%'`** — quét tuần tự toàn bộ.

**Cách hai:** lật ra trang Index ở cuối sách. Ở đó có sẵn dòng: `database ....... 12, 45, 88`. Bạn nhảy thẳng tới ba trang đó. Mất 5 giây.

Bây giờ là chi tiết quan trọng nhất của phép ví von — chỗ mà nhiều người kể analogy xong rồi bỏ dở. Hãy để ý: **thời gian tra mục lục không phụ thuộc vào việc cuốn sách dày bao nhiêu**. Sách 800 trang hay 8.000 trang, bạn vẫn lật đúng một trang mục lục rồi nhảy tới đúng số trang cần đọc. Thứ quyết định công sức của bạn là **số trang được liệt kê ra** (tức là số kết quả khớp), chứ không phải độ dày cuốn sách.

Full-text search làm y hệt như vậy. Nó dựng sẵn một cấu trúc gọi là **inverted index** (*chỉ mục ngược*) — một bảng tra cứu ánh xạ **"từ" → "danh sách các tài liệu chứa từ đó"**.

Vì sao gọi là "ngược"? Vì cách lưu trữ thông thường là từ tài liệu suy ra các từ ("bài số 12 chứa những từ nào"). Inverted index lật ngược chiều đó lại: từ một từ suy ra danh sách tài liệu ("từ *database* xuất hiện ở những bài nào"). Chính chiều lật ngược này khiến việc tìm kiếm trở nên tức thì.

Nhưng mục lục sách vẫn còn thiếu một thứ mà full-text search có: mục lục sách ghi "running" và "ran" thành hai mục riêng biệt. Full-text search khôn hơn — nó gộp cả hai về cùng một mục. Cơ chế gộp đó tên là *stemming*, và ta sẽ mổ nó ngay bây giờ.

### 1.9. Bộ thuật ngữ cốt lõi của full-text search

Đây là 8 từ bạn phải nắm chắc. Hãy đọc chậm, mỗi từ một lần.

**Full-text search** (viết tắt **FTS**, dịch: *tìm kiếm toàn văn*) là kỹ thuật tìm trong một tập tài liệu viết bằng ngôn ngữ tự nhiên (tiếng người, không phải mã máy) những tài liệu khớp nhất với truy vấn của người dùng.

**Token** (dịch: *đơn vị từ*) là một mẩu văn bản được cắt ra từ câu, thường là một từ. Hành động cắt đó gọi là **tokenize** (*tách token*). Ví dụ câu "the quick fox" được tokenize thành ba token: `the`, `quick`, `fox`.

**Stemming** (nghĩa đen: *rút về gốc từ*; "stem" là thân/gốc cây) là quá trình cắt bỏ các đuôi biến đổi của một từ để đưa nó về dạng gốc chung. Ví dụ: `running` → `run`, `foxes` → `fox`, `databases` → `databas`.

Bạn có thể thấy lạ khi `databases` biến thành `databas` — thiếu mất chữ "e". Đây là điểm quan trọng: **kết quả của stemming không nhất thiết là một từ có thật trong từ điển**. Thuật toán chỉ cần đảm bảo rằng mọi biến thể của cùng một từ đều rút về **cùng một dạng**, dù dạng đó nhìn có kỳ cục. Miễn là "database" và "databases" đều ra `databas`, việc so khớp vẫn đúng.

**Lexeme** (đọc "léc-xim", dịch: *đơn vị từ vựng*) là kết quả cuối cùng sau khi một token đã được chuẩn hóa xong. Đây chính là **đơn vị tìm kiếm thật sự** của FTS. FTS không so ký tự, không so token — nó so lexeme.

**Stop-word** (dịch: *từ dừng*) là những từ xuất hiện quá nhiều nên không giúp phân biệt tài liệu nào với tài liệu nào: "the", "is", "a", "and", "of". Chúng bị **loại bỏ hoàn toàn** khi xử lý. Lý do rất thực dụng: nếu giữ lại chữ "the", thì gần như 100% tài liệu đều chứa nó, và danh sách tài liệu cho từ đó sẽ dài vô ích, vừa tốn chỗ vừa không giúp tìm ra gì.

**`tsvector`** (ts = *text search*) là một kiểu dữ liệu của PostgreSQL, dùng để lưu **danh sách các lexeme đã loại trùng, đã sắp xếp**, kèm theo vị trí của chúng trong văn bản gốc. Hãy nghĩ về nó như "phiên bản đã được tiền xử lý sẵn của tài liệu, ở dạng máy tìm kiếm dễ đọc nhất".

**`tsquery`** là kiểu dữ liệu lưu **truy vấn tìm kiếm**, cũng ở dạng lexeme, được nối với nhau bằng các toán tử logic: `&` là AND (phải có cả hai), `|` là OR (có một trong hai là được), `!` là NOT (không được chứa), và `<->` là FOLLOWED BY (từ này phải đứng ngay trước từ kia).

**`@@`** là **toán tử match** (*so khớp*). Nó đặt giữa một `tsvector` và một `tsquery` rồi trả lời đúng một câu hỏi: "tài liệu này có thỏa mãn truy vấn kia không?" Kết quả là `true` hoặc `false`.

Cuối cùng, một phân biệt mà bài giảng gốc nhấn mạnh: **RegEx so với FTS**. **RegEx** (viết tắt của *Regular Expression*, dịch: *biểu thức chính quy*) là công cụ tìm chuỗi theo một khuôn mẫu ký tự, ví dụ "tìm mọi chuỗi bắt đầu bằng chữ hoa rồi tới 3 chữ số". RegEx mạnh nhưng vẫn làm việc ở tầng **ký tự**, giống `LIKE`. FTS làm việc ở tầng **lexeme**. Đó là khác biệt bản chất, không phải khác biệt về mức độ.

### 1.10. Ví dụ chạy tay — tự mình biến một câu thành `tsvector`

Lý thuyết đủ rồi. Giờ ta làm thủ công từng bước để bạn **nhìn thấy** dữ liệu biến đổi, thay vì chỉ nghe mô tả.

Câu đầu vào: **"The quick brown foxes are running"**

**Bước 1 — Tokenize (tách thành token).** PostgreSQL cắt câu theo khoảng trắng và dấu câu:

```
The | quick | brown | foxes | are | running
 ①      ②       ③       ④      ⑤       ⑥
```

Sáu token, và ta ghi luôn số thứ tự vị trí của từng token — con số này sẽ có ích ở bước cuối.

**Bước 2 — Loại bỏ stop-word.** Trong bộ từ điển tiếng Anh của Postgres, `the` và `are` nằm trong danh sách stop-word:

```
~~The~~ | quick | brown | foxes | ~~are~~ | running
           ②       ③       ④              ⑥
```

Còn lại bốn token: `quick` (vị trí 2), `brown` (3), `foxes` (4), `running` (6). Chú ý rằng **vị trí không bị đánh số lại** — `running` vẫn giữ số 6 chứ không tụt xuống 4. Điều này quan trọng để sau này còn biết các từ có đứng cạnh nhau trong câu gốc hay không.

**Bước 3 — Stemming (chuẩn hóa về lexeme).** Bộ stemmer xử lý từng token:

| Token | → Lexeme | Ghi chú |
|---|---|---|
| `quick` | `quick` | không có đuôi biến đổi, giữ nguyên |
| `brown` | `brown` | giữ nguyên |
| `foxes` | `fox` | cắt đuôi số nhiều `-es` |
| `running` | `run` | cắt đuôi `-ning` |

**Bước 4 — Sắp xếp theo thứ tự chữ cái, loại trùng, gắn vị trí.** Kết quả cuối cùng chính là `tsvector`:

```
'brown':3 'fox':4 'quick':2 'run':6
```

Hãy đọc kỹ dòng trên: mỗi phần tử gồm một lexeme trong dấu nháy, theo sau là dấu hai chấm và vị trí của nó trong câu gốc. Danh sách được sắp theo thứ tự chữ cái (brown → fox → quick → run), không theo thứ tự trong câu.

Và đây là khoảnh khắc "à há" của cả bài học: **trong `tsvector` này có lexeme `run`, mặc dù câu gốc không hề chứa chuỗi ký tự "run" đứng riêng lẻ** — nó chỉ có "running". Nên khi người dùng tìm "run", câu này vẫn khớp. Đó chính xác là điều mà `LIKE` không bao giờ làm được.

### 1.11. Code "hello world" — chạy được thật, chú thích từng dòng

Bây giờ ta để PostgreSQL tự làm lại đúng bốn bước trên, và kiểm chứng xem kết quả có khớp với phần chạy tay không.

```sql
-- ────────────────────────────────────────────────────────────
-- BƯỚC 1: Biến một đoạn văn bản thành tsvector
-- to_tsvector nhận 2 tham số:
--   (a) tên "text search configuration" — ở đây là 'english',
--       nói cho Postgres biết dùng bộ luật của ngôn ngữ nào để
--       tách từ, bỏ stop-word và stem;
--   (b) đoạn văn bản cần xử lý.
-- ────────────────────────────────────────────────────────────
SELECT to_tsvector('english', 'The quick brown foxes are running');
-- Kết quả:  'brown':3 'fox':4 'quick':2 'run':6
-- → đúng y hệt phần chạy tay ở mục 1.10.


-- ────────────────────────────────────────────────────────────
-- BƯỚC 2: Biến truy vấn của người dùng thành tsquery
-- Điểm mấu chốt: truy vấn cũng bị stem giống hệt tài liệu.
-- ────────────────────────────────────────────────────────────
SELECT to_tsquery('english', 'running');
-- Kết quả:  'run'
-- → người dùng gõ "running", nhưng thứ đem đi so khớp là 'run'.


-- ────────────────────────────────────────────────────────────
-- BƯỚC 3: So khớp bằng toán tử @@ → trả về true / false
-- Đọc câu này là: "cái tsvector bên trái có thỏa cái tsquery
-- bên phải không?"
-- ────────────────────────────────────────────────────────────
SELECT to_tsvector('english', 'The quick brown foxes are running')
    @@ to_tsquery('english', 'running');
-- Kết quả:  t   (viết tắt của true — có khớp)


-- ────────────────────────────────────────────────────────────
-- BƯỚC 4: Áp dụng lên một bảng thật
-- Với mỗi hàng trong bảng articles, Postgres biến cột body
-- thành tsvector rồi so với tsquery. Hàng nào trả true thì lấy.
-- ────────────────────────────────────────────────────────────
SELECT title
FROM articles
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'running');
-- → trả về mọi bài viết có chứa run / runs / ran / running.
```

**Điểm cần khắc cốt ghi tâm từ đoạn code này:** cả **tài liệu** lẫn **truy vấn** đều đi qua **cùng một bộ chuẩn hóa** (cùng tách token, cùng bỏ stop-word, cùng stem). Nhờ vậy chúng "gặp nhau" ở tầng lexeme. Nếu bạn dùng config khác nhau cho hai bên — ví dụ tài liệu dùng `'english'` còn truy vấn dùng `'simple'` — chúng sẽ không gặp nhau và kết quả sẽ sai một cách khó hiểu.

### 1.12. Nối lại với ví dụ trong bài giảng gốc

Bài gốc đưa ra ví dụ: tìm cụm "hands-on data scientist", cụ thể là kiểm tra xem tài liệu có chứa "hands-on" và có chứa "data" đứng ngay trước "scientist" hay không, rồi hệ thống trả về `t`.

Diễn giải lại bằng ngôn ngữ ta vừa học: tài liệu được lưu dưới dạng `tsvector`; cụm cần tìm được viết thành `tsquery` với toán tử `<->` để yêu cầu "data" phải đứng liền trước "scientist"; và `@@` cho biết có khớp hay không. Chính nhờ Postgres đã ghi lại **vị trí** của từng lexeme ở bước 4 trong mục 1.10 mà nó mới trả lời được câu hỏi "hai từ này có đứng cạnh nhau không".

### 1.13. 🧩 [Ngoài bài gốc] — Ba điều bài gốc không nói mà bạn nên biết ngay từ Basic

**Một: FTS không phải là "AI hiểu nghĩa".** Nó vẫn chỉ là so khớp từ, chỉ là so khớp *thông minh hơn* nhờ chuẩn hóa. Nó tuyệt đối không biết "xe hơi" và "ô tô" là cùng một thứ, vì hai cụm này ra hai lexeme khác nhau. Muốn máy hiểu nghĩa, bạn cần vector search — và Phần 4 sẽ ghép hai thứ lại.

**Hai: FTS có thể phá vỡ kỳ vọng của người dùng.** Một người tìm chính xác tên sản phẩm "iPhone 15 Pro" có thể nhận về kết quả lệch, vì stemmer đôi khi cắt bậy các mã sản phẩm. Với những cột chứa mã, tên riêng, username — người ta thường dùng config `'simple'` (không stem, không bỏ stop-word, chỉ chuyển thành chữ thường) thay vì `'english'`.

**Ba: `to_tsvector` tính lại mỗi lần chạy.** Trong câu lệnh ở BƯỚC 4 phía trên, mỗi lần bạn tìm kiếm là Postgres phải xử lý lại toàn bộ nội dung của **mọi hàng** trong bảng. Với bảng lớn thì đây là thảm họa hiệu năng. Cách chữa là dựng index hoặc lưu sẵn `tsvector` vào một cột — đó là nội dung chính của Phần 2.

### ✅ Self-check Phần 1

Trả lời bằng lời của chính bạn trước khi xem gợi ý.

**1. Nêu ba lý do khiến `WHERE body LIKE '%running%'` là lựa chọn tồi cho tìm kiếm văn bản.**
> *Gợi ý đáp án:* (a) Không dùng được B-tree index vì có `%` ở đầu → rơi về sequential scan, thời gian tỉ lệ với tổng số hàng; (b) không hiểu biến thể từ nên bỏ sót "ran", "runs"; (c) khớp nhầm vào giữa từ khác như "brunch" nếu nới thành `%run%`.

**2. Lexeme là gì? Các từ "cats", "cat", "catty" sẽ về lexeme nào?**
> *Gợi ý đáp án:* Lexeme là dạng đã chuẩn hóa của một từ, dùng làm đơn vị so khớp của FTS. "cats" và "cat" đều về `cat`. "catty" là tính từ khác gốc nghĩa nên stemmer tiếng Anh thường để nó thành `catti` — một lexeme riêng. Điều này cho thấy stemming là *heuristic* (quy tắc gần đúng), không hoàn hảo.

**3. Dự đoán kết quả của `to_tsvector('english','The cats are sleeping')`. Vì sao "the" và "are" biến mất?**
> *Gợi ý đáp án:* Kết quả là `'cat':2 'sleep':4`. "the" và "are" là stop-word — quá phổ biến, không giúp phân biệt tài liệu nào với tài liệu nào — nên bị loại bỏ để index gọn và nhanh hơn.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Từ đây trở đi, giáo trình giả định bạn đã nắm chắc: lexeme, stemming, stop-word, `tsvector`, `tsquery`, `@@`. Nếu còn lăn tăn chỗ nào, quay lại mục 1.9 và 1.10 — không có gì phải ngại.

### 2.1. Bên trong PostgreSQL: văn bản đi qua những chặng nào

Ở Phần 1 ta đã chạy tay bốn bước. Bây giờ hãy xem PostgreSQL gọi tên các bước đó là gì, vì tên gọi này sẽ xuất hiện trong tài liệu chính thức và trong phỏng vấn.

**Text search configuration** (dịch: *cấu hình tìm kiếm văn bản*) là một "bộ luật xử lý ngôn ngữ" có tên. Khi bạn viết `to_tsvector('english', ...)`, chuỗi `'english'` chính là tên của một configuration. Mỗi configuration gồm hai bộ phận.

**Bộ phận thứ nhất — Parser** (dịch: *bộ phân tích cú pháp*). Nhiệm vụ của nó là cắt văn bản thô thành các token **và gán loại cho từng token**: đây là từ thường, đây là số, đây là email, đây là URL, đây là tên máy chủ.

Việc gán loại này quan trọng hơn bạn tưởng. Nếu parser ngây thơ cắt theo dấu chấm và dấu @, thì địa chỉ `lan@company.com` sẽ vỡ thành ba mảnh vô nghĩa. Parser của Postgres nhận ra đó là một email và giữ nguyên nó thành một token duy nhất. URL, số phiên bản, đường dẫn file cũng vậy.

**Bộ phận thứ hai — Dictionaries** (dịch: *các bộ từ điển*). Từng token sau khi được parser gán loại sẽ đi qua một dãy từ điển. Mỗi từ điển có quyền làm một trong ba việc: biến token thành lexeme, vứt bỏ token, hoặc chuyển nó cho từ điển kế tiếp xử lý.

Ba loại từ điển bạn cần biết:

- **Stop-word dictionary** — chứa danh sách các từ cần vứt bỏ ("the", "is", "a"...).
- **Stemmer** — bộ rút gốc từ. Postgres dùng thuật toán **Snowball**, một họ thuật toán stemming do Martin Porter phát triển, có sẵn luật cho hàng chục ngôn ngữ.
- **Synonym / thesaurus dictionary** (dịch: *từ điển đồng nghĩa*) — tùy chọn, cho phép bạn tự khai báo "những từ này coi như một". Ví dụ khai báo "supernovae" và "supernova stars" đều quy về lexeme `sn`.

**Chọn sai configuration là ra kết quả sai.** Hãy nhớ ba lựa chọn hay dùng nhất:

| Config | Nó làm gì | Dùng khi |
|---|---|---|
| `'english'` | Bỏ stop-word tiếng Anh + stem theo luật tiếng Anh | Nội dung văn xuôi tiếng Anh |
| `'simple'` | Chỉ chuyển thành chữ thường. **Không** stem, **không** bỏ stop-word | Mã sản phẩm, username, tag, tên riêng — những thứ không được phép biến dạng |
| `'vietnamese'` | Postgres **không có sẵn** config này | Xem đoạn dưới |

Về **tiếng Việt**: PostgreSQL không có bộ stemmer tiếng Việt tích hợp sẵn, và thật ra tiếng Việt cũng không cần stemming theo kiểu tiếng Anh vì từ tiếng Việt không biến đổi đuôi. Vấn đề của tiếng Việt lại nằm ở chỗ khác: **dấu thanh** (người dùng hay gõ không dấu) và **từ ghép** ("cơ sở dữ liệu" là ba token nhưng một khái niệm). Cách xử lý thực dụng là dùng config `'simple'` kết hợp extension **`unaccent`** (một extension bỏ dấu tiếng Việt, biến "tiếng" thành "tieng"), hoặc chuyển hẳn sang vector search cho phần ngữ nghĩa. Ta sẽ nói kỹ ở Phần 3 và 4.

### 2.2. Họ hàm tạo `tsquery` — chọn đúng cái cho đúng tình huống

Ở Phần 1 ta mới dùng `to_tsquery`. Thực tế Postgres có bốn hàm để tạo `tsquery`, và **chọn sai hàm là nguyên nhân số một khiến ô tìm kiếm bị lỗi 500** khi lên production.

| Hàm | Đầu vào phải trông như thế nào | Dùng khi nào |
|---|---|---|
| `to_tsquery` | Bắt buộc có toán tử: `'run & fast'` | Khi **bạn** viết truy vấn, cần kiểm soát chính xác. **Không** đưa input thô của người dùng vào |
| `plainto_tsquery` | Văn bản thô, tự động nối các từ bằng AND: `'run fast'` → `'run' & 'fast'` | Input đơn giản, muốn "phải có tất cả các từ" |
| `phraseto_tsquery` | Văn bản thô → cụm từ liền kề: `'full text search'` → `'full' <-> 'text' <-> 'search'` | Tìm đúng **cụm từ**, đúng thứ tự |
| `websearch_to_tsquery` | Cú pháp kiểu Google: `'postgres -mysql "full text"'` | **Input trực tiếp từ ô tìm kiếm của người dùng** — an toàn nhất (cần PostgreSQL 11 trở lên) |

⚠️ **Chỗ khó — và đây là chỗ rất nhiều người vấp.** `to_tsquery` yêu cầu đầu vào phải đúng cú pháp của nó. Người dùng thật thì gõ đủ thứ: khoảng trắng thừa, dấu chấm, dấu ngoặc lệch, ký tự `#` trong "c# tips". Mỗi lần như vậy `to_tsquery` sẽ **báo lỗi cú pháp**, và nếu bạn không bắt lỗi, server trả về lỗi 500 cho người dùng.

💡 **Mẹo thực chiến:** với bất kỳ ô tìm kiếm nào hướng tới người dùng cuối, hãy dùng **`websearch_to_tsquery`**. Nó nuốt được mọi thứ lộn xộn mà không báo lỗi, đồng thời hỗ trợ đúng những quy ước người dùng đã quen từ Google: đặt trong dấu ngoặc kép là tìm nguyên cụm, đặt dấu trừ phía trước là loại trừ, viết chữ OR là hoặc.

```sql
-- Người dùng gõ vào ô search:  postgres -mysql "full text"
SELECT websearch_to_tsquery('english', 'postgres -mysql "full text"');
-- Kết quả: 'postgr' & !'mysql' & 'full' <-> 'text'
--   'postgr'        → phải có (postgres đã bị stem)
--   !'mysql'        → không được có mysql
--   'full' <-> 'text' → phải có "full" đứng ngay trước "text"
```

### 2.3. Đánh index cho FTS — GIN là lựa chọn chuẩn

Nhắc lại từ mục 1.13: nếu không có index, mỗi lần tìm kiếm Postgres phải tính lại `to_tsvector` cho **từng hàng** trong bảng. Với 10 nghìn hàng thì không sao; với 10 triệu hàng thì mỗi truy vấn mất hàng chục giây.

**GIN** (viết tắt của *Generalized Inverted Index*, dịch: *chỉ mục ngược tổng quát*) chính là hiện thực cụ thể của phép ví von "mục lục cuối sách" ở mục 1.8. Nó lưu ánh xạ từ mỗi lexeme tới danh sách các hàng chứa lexeme đó. Đây là loại index tiêu chuẩn cho FTS.

Có hai cách dựng, và bạn nên biết cả hai vì mỗi cách hợp một tình huống.

**Cách 1 — Functional index (index trên biểu thức).**

**Functional index** nghĩa là bạn đánh index không phải trên giá trị gốc của cột, mà trên **kết quả của một hàm áp lên cột đó**. Ở đây ta index trên kết quả của `to_tsvector('english', body)`.

```sql
-- Dựng index GIN trên biểu thức to_tsvector('english', body)
CREATE INDEX articles_fts_idx          -- articles_fts_idx là tên ta đặt cho index
ON articles                             -- trên bảng articles
USING GIN (to_tsvector('english', body));  -- loại index là GIN, trên biểu thức này

-- Khi truy vấn, biểu thức trong WHERE PHẢI GIỐNG HỆT biểu thức lúc tạo index
SELECT title
FROM articles
WHERE to_tsvector('english', body)     -- ← giống hệt dòng CREATE INDEX ở trên
   @@ websearch_to_tsquery('english', 'postgres index');
```

Ưu điểm: không cần thêm cột nào, không tốn thêm chỗ lưu `tsvector`, không phải lo đồng bộ. Nhược điểm: câu truy vấn phải viết lặp lại đúng biểu thức đó, và nếu muốn gộp nhiều cột lại thì viết khá dài dòng.

**Cách 2 — Generated column (cột được sinh tự động) + GIN.**

**Generated column** là một cột mà bạn **không tự điền giá trị**; Postgres tự tính nó ra từ các cột khác, và **tự cập nhật lại mỗi khi các cột nguồn thay đổi**. Từ khóa `STORED` nghĩa là giá trị đó được lưu thật lên đĩa (thay vì tính lại mỗi lần đọc). Tính năng này có từ PostgreSQL 12.

```sql
ALTER TABLE articles                        -- sửa bảng articles
ADD COLUMN search_vector tsvector           -- thêm cột search_vector kiểu tsvector
GENERATED ALWAYS AS (                       -- giá trị luôn được Postgres tự sinh
    to_tsvector(
        'english',
        coalesce(title, '') || ' ' || coalesce(body, '')  -- ghép title + body
    )
) STORED;                                   -- lưu thật lên đĩa, tự cập nhật khi title/body đổi

-- Dựng index GIN trên chính cột đó
CREATE INDEX articles_sv_idx ON articles USING GIN (search_vector);

-- Truy vấn giờ ngắn gọn hẳn
SELECT title
FROM articles
WHERE search_vector @@ websearch_to_tsquery('english', 'postgres index');
```

Giải thích hai ký hiệu vừa xuất hiện. **`||`** trong PostgreSQL là toán tử **nối chuỗi** (ghép hai đoạn text lại). **`coalesce(a, b)`** là hàm trả về `a` nếu `a` không rỗng, còn nếu `a` là **NULL** thì trả về `b`.

**NULL** (dịch: *rỗng / không có giá trị*) là một giá trị đặc biệt trong SQL, nghĩa là "ô này không có dữ liệu". Nó khác chuỗi rỗng `''`: chuỗi rỗng là "có dữ liệu, và dữ liệu đó là không có ký tự nào"; NULL là "hoàn toàn không biết gì".

⚠️ **Chỗ khó — và là lỗi khiến người ta mất cả buổi chiều để debug.** Trong SQL, **NULL nuốt tất cả**: bất cứ phép toán nào có NULL tham gia đều cho kết quả NULL. Nên nếu bài viết nào đó không có `title` (title là NULL) mà bạn quên bọc `coalesce`, thì `title || ' ' || body` ra NULL, kéo theo `search_vector` của cả hàng đó thành NULL, và **hàng đó biến mất hoàn toàn khỏi mọi kết quả tìm kiếm** — dù nội dung body chứa đúng từ khóa. Không có thông báo lỗi nào cả; nó chỉ lặng lẽ mất tích. Vì vậy: **luôn bọc `coalesce(col, '')` khi ghép cột.**

🧩 **[Ngoài bài gốc]** Ngày xưa, khi Postgres chưa có generated column, người ta duy trì cột `tsvector` bằng **trigger** — một đoạn code tự động chạy mỗi khi có thao tác thêm/sửa/xóa trên bảng. Cách này chạy được nhưng dễ hỏng: quên viết trigger cho một đường cập nhật nào đó, hoặc sửa dữ liệu bằng câu lệnh thủ công, là cột `tsvector` lệch khỏi nội dung thật ngay, và không ai biết. Từ PostgreSQL 12 trở đi, **generated column STORED** đã thay thế hoàn toàn nhu cầu này. Best practice năm 2026: functional index nếu chỉ tìm trên một cột; generated column nếu cần gộp nhiều cột và gán trọng số; **đừng viết trigger sync bằng tay nữa** trừ khi có lý do rất đặc biệt.

### 2.4. Ranking — sắp xếp kết quả theo độ liên quan

Toán tử `@@` chỉ trả lời "có khớp hay không". Nhưng nếu 5.000 bài viết cùng khớp, bạn cần biết bài nào **liên quan nhất** để đưa lên đầu. Việc đó gọi là **ranking** (*xếp hạng*), và độ liên quan gọi là **relevance**.

```sql
SELECT title,
       ts_rank(search_vector, query) AS rank    -- tính điểm liên quan cho từng hàng
FROM articles,
     websearch_to_tsquery('english', 'postgres performance') AS query
       -- Dòng trên đặt tsquery vào FROM và đặt tên là query, để
       -- Postgres chỉ phải tính nó MỘT LẦN thay vì tính lại cho mỗi hàng.
WHERE search_vector @@ query                    -- lọc trước: chỉ giữ hàng khớp
ORDER BY rank DESC                              -- sắp xếp điểm cao xuống thấp
LIMIT 10;                                       -- chỉ lấy 10 kết quả đầu
```

Hai hàm tính điểm bạn cần phân biệt:

- **`ts_rank`** — chấm điểm dựa trên **tần suất** xuất hiện của các lexeme khớp. Từ khóa xuất hiện càng nhiều lần, điểm càng cao.
- **`ts_rank_cd`** — "cd" là viết tắt của **cover density** (*mật độ bao phủ*). Ngoài tần suất, nó còn **thưởng điểm khi các từ khóa nằm gần nhau** trong tài liệu. Hợp lý khi người dùng tìm một cụm từ: một bài có "machine learning" nằm liền nhau đáng lẽ phải hơn một bài có chữ "machine" ở đầu và chữ "learning" ở cuối.

**`setweight` — quyết định trường nào quan trọng hơn.**

Trong thực tế, một từ khóa xuất hiện ở **tiêu đề** rõ ràng có ý nghĩa hơn khi nó nằm lọt thỏm giữa thân bài. `setweight` cho phép bạn gán **trọng số** A, B, C hoặc D cho từng nguồn văn bản, trong đó A là quan trọng nhất và D là ít quan trọng nhất.

```sql
ALTER TABLE articles
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    -- Tiêu đề: trọng số A (cao nhất)
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    -- Thân bài: trọng số B
    setweight(to_tsvector('english', coalesce(body, '')), 'B') ||
    -- Tag: trọng số C. array_to_string biến mảng tag thành một chuỗi
    setweight(to_tsvector('english', coalesce(array_to_string(tags, ' '), '')), 'C')
) STORED;
```

Kết quả: một bài có từ khóa trong tiêu đề sẽ được `ts_rank` chấm điểm cao hơn hẳn bài chỉ có từ khóa trong thân. Và điều đáng nói là bạn đạt được điều đó **hoàn toàn bằng cấu hình trong database**, không phải viết một dòng code xử lý relevance nào ở tầng ứng dụng.

### 2.5. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh

**Lỗi 1 — Index đã dựng nhưng không được dùng, vì biểu thức không khớp.**

Bạn tạo index bằng `GIN (to_tsvector('english', body))` nhưng trong câu truy vấn lại viết `to_tsvector(body)` — thiếu mất tham số config. Với Postgres, đây là **hai biểu thức khác nhau**, nên index bị bỏ qua và truy vấn rơi về sequential scan.

Điều tệ nhất là **không có lỗi nào được báo ra**. Truy vấn vẫn trả về kết quả đúng, chỉ là chậm. Bạn sẽ chỉ phát hiện khi người dùng phàn nàn.

Cách kiểm tra: dùng **`EXPLAIN ANALYZE`** — một câu lệnh yêu cầu Postgres chạy thật rồi kể lại chi tiết nó đã làm gì.

```sql
EXPLAIN ANALYZE
SELECT title FROM articles
WHERE to_tsvector('english', body) @@ websearch_to_tsquery('english', 'postgres');
```

Trong kết quả trả về, bạn tìm dòng có chữ **`Bitmap Index Scan`** — đó là dấu hiệu index đang được dùng. Nếu thay vào đó bạn thấy **`Seq Scan`**, index đang bị bỏ qua.

**Quy tắc vàng:** biểu thức trong `WHERE` phải **trùng khít từng chữ** với biểu thức lúc `CREATE INDEX` — cùng tên config, cùng dạng hai tham số.

**Lỗi 2 — Quên `coalesce`, NULL nuốt cả `tsvector`.**

Đã mô tả kỹ ở mục 2.3. Triệu chứng đặc trưng: một bài viết rõ ràng chứa từ khóa nhưng không bao giờ xuất hiện trong kết quả. Khi gặp triệu chứng này, việc đầu tiên nên làm là chạy `SELECT id FROM articles WHERE search_vector IS NULL;` để xem có bao nhiêu hàng đang bị mất tích.

**Lỗi 3 — Nhét input thô của người dùng vào `to_tsquery`.**

Đã mô tả ở mục 2.2. Triệu chứng: log lỗi đầy `syntax error in tsquery`, và một tỉ lệ nhỏ nhưng dai dẳng các request tìm kiếm trả về lỗi 500. Cách chữa: đổi sang `websearch_to_tsquery` hoặc `plainto_tsquery`.

### ✅ Self-check Phần 2

**1. Với ô tìm kiếm của người dùng, bạn nên dùng hàm tạo `tsquery` nào, và vì sao KHÔNG dùng `to_tsquery`?**
> *Gợi ý đáp án:* Dùng `websearch_to_tsquery` (hoặc `plainto_tsquery`). Không dùng `to_tsquery` vì nó đòi hỏi cú pháp toán tử chặt chẽ; input thật của người dùng có khoảng trắng, dấu câu, ký tự lạ sẽ gây syntax error → lỗi 500.

**2. Bạn muốn từ khóa khớp ở `title` được ưu tiên hơn khớp ở `body`. Dùng cơ chế gì?**
> *Gợi ý đáp án:* `setweight` — gán trọng số 'A' cho `to_tsvector` của title và 'B' cho body, rồi nối bằng `||`, sau đó xếp hạng bằng `ts_rank`.

**3. Index là `GIN(to_tsvector('english', body))` nhưng truy vấn viết `to_tsvector(body)`. Chuyện gì xảy ra và làm sao phát hiện?**
> *Gợi ý đáp án:* Biểu thức không khớp nên index bị bỏ qua, truy vấn rơi về sequential scan — kết quả vẫn đúng nhưng chậm dần theo kích thước bảng, và không có cảnh báo. Phát hiện bằng `EXPLAIN ANALYZE`: nếu thấy `Seq Scan` thay vì `Bitmap Index Scan` thì đúng là đang dính lỗi này.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Phần này nhìn xuống **bên dưới bề mặt**: cấu trúc dữ liệu thật, độ phức tạp thuật toán, và những đánh đổi bạn buộc phải chọn. Đây là vùng kiến thức phân biệt người biết dùng công cụ với người hiểu công cụ.

### 3.1. Ruột gan của inverted index — vì sao GIN nhanh

Ta đã nói GIN là "mục lục cuối sách". Giờ hãy mở nó ra xem bên trong có gì.

GIN lưu một danh sách các lexeme (đã sắp xếp), và với mỗi lexeme, nó lưu kèm một **posting list** (*danh sách đăng*) — chính là danh sách các hàng chứa lexeme đó.

Cụ thể hơn, posting list không lưu cả hàng mà chỉ lưu **row identifier** (*mã định danh hàng*). Trong Postgres, mã này gọi là **TID** (*Tuple ID*) — một cặp số cho biết hàng đó nằm ở trang nào trên đĩa và ở vị trí thứ mấy trong trang.

Hình dung cấu trúc trong đầu:

```
lexeme 'databas'  →  [ hàng #12, hàng #45, hàng #88, hàng #301, ... ]
lexeme 'index'    →  [ hàng #7,  hàng #45, hàng #200, ... ]
lexeme 'postgr'   →  [ hàng #45, hàng #88, ... ]
```

Khi bạn tìm `'postgr' & 'index'` (cả hai từ), Postgres lấy hai posting list rồi **giao** chúng lại với nhau, ra ngay `[hàng #45]`. Nó **không hề đụng vào** những hàng còn lại của bảng.

**Bây giờ tới phần độ phức tạp.** Trước hết, giải thích ký hiệu: **Big-O** (đọc "bíc ô", viết là `O(...)`) là cách mô tả *thời gian chạy của thuật toán tăng lên như thế nào khi dữ liệu lớn dần*. `O(N)` nghĩa là thời gian tỉ lệ thuận với số lượng dữ liệu `N` — gấp đôi dữ liệu thì gấp đôi thời gian. `O(1)` nghĩa là thời gian không đổi bất kể dữ liệu lớn cỡ nào.

Với hai cách tìm kiếm ta đang so sánh:

- **`LIKE '%x%'`** phải đọc từng hàng → **`O(N)`** với `N` là **tổng số hàng trong bảng**. Bảng 10 triệu hàng thì chậm gấp 10 lần bảng 1 triệu hàng.
- **FTS với GIN** chỉ đọc những hàng nằm trong posting list → thời gian phụ thuộc **số hàng khớp**, cộng thêm chi phí tra cứu lexeme trong index (rất nhỏ, dạng logarit). Nếu chỉ 50 bài viết chứa từ khóa, thì bảng có 1 triệu hay 50 triệu hàng cũng gần như không khác nhau.

**Đây là toàn bộ bí mật vì sao FTS giữ được tốc độ khi dữ liệu phình to.** Và nó cũng ngay lập tức chỉ ra điểm yếu: nếu từ khóa người dùng tìm lại là một từ cực phổ biến khớp với 2 triệu hàng, thì posting list dài 2 triệu phần tử và FTS cũng chậm. Ta sẽ quay lại điểm này ở Phần 4.

**Mặt trái của GIN — chuyện xây và cập nhật.** Xây GIN tốn kém hơn xây B-tree, vì nó phải bóc từng tài liệu ra thành nhiều lexeme rồi gộp lại. Có hai điều đáng nhớ:

- **`maintenance_work_mem`** là một tham số cấu hình quy định lượng RAM mà Postgres được phép dùng cho các thao tác bảo trì như xây index. GIN **rất nhạy** với tham số này — tăng nó lên có thể rút ngắn thời gian xây index từ hàng giờ xuống hàng phút.
- **`fastupdate`** là một cơ chế của GIN: thay vì cập nhật index ngay mỗi lần ghi, nó gom các thay đổi vào một vùng đệm rồi gộp vào sau. Nhờ vậy ghi nhanh hơn, nhưng đổi lại truy vấn phải quét thêm vùng đệm đó, và nếu vùng đệm phình to mà chưa được dọn thì truy vấn chậm dần.

### 3.2. GIN so với GiST — câu hỏi phỏng vấn kinh điển

**GiST** (viết tắt của *Generalized Search Tree*, dịch: *cây tìm kiếm tổng quát*) là loại index thứ hai mà Postgres hỗ trợ cho FTS. Khác biệt cốt lõi: GiST là **lossy** (*có mất mát thông tin*).

"Lossy" ở đây nghĩa là GiST không lưu chính xác từng lexeme, mà lưu một **signature** — một dạng dấu vân tay nén lại của tập lexeme. Dấu vân tay này nhỏ gọn nhưng có thể trùng nhau, nên GiST có thể báo "hàng này có vẻ khớp" trong khi thật ra không khớp. Trường hợp đó gọi là **false positive** (*dương tính giả*).

Vì có false positive, sau khi tra index GiST, Postgres buộc phải mở **heap** ra kiểm tra lại từng ứng viên. (**Heap** trong Postgres là nơi lưu dữ liệu thật của bảng, phân biệt với index.) Bước kiểm tra lại này gọi là **recheck**, và nó chính là lý do GiST truy vấn chậm hơn GIN.

| Tiêu chí | **GIN** | **GiST** |
|---|---|---|
| Bản chất | Inverted index, lưu chính xác | Signature tree, lossy → phải recheck |
| Tốc độ truy vấn | **Nhanh hơn** cho FTS | Chậm hơn |
| Tốc độ xây / ghi thêm | Chậm hơn, ngốn RAM | **Nhanh hơn**, nhẹ hơn |
| Chịu cập nhật thường xuyên | Kém hơn | **Tốt hơn** |
| Kích thước index | To hơn | **Nhỏ hơn** (điều chỉnh được qua `siglen`) |
| Nhạy với `maintenance_work_mem` | Có | Không |
| Nên chọn khi | **Đọc nhiều, văn bản ít thay đổi** — tức đa số trường hợp | Ghi rất nhiều, hoặc bị giới hạn dung lượng đĩa |

**Chốt để nhớ:** mặc định chọn **GIN**. Chỉ nghiêng về GiST khi bảng của bạn ghi rất nhiều so với đọc, hoặc khi dung lượng index là ràng buộc thật sự.

### 3.3. Trade-off là gì, và ba cách duy trì `tsvector`

Trước khi so sánh, cần định nghĩa một từ sẽ xuất hiện dày đặc từ đây tới hết giáo trình.

**Trade-off** (dịch: *sự đánh đổi*) là tình huống bạn không thể có tất cả cùng lúc: muốn được điều này thì phải chấp nhận mất điều kia. Nhanh hơn thì tốn bộ nhớ hơn; đơn giản hơn thì kém linh hoạt hơn. Trong kỹ thuật, gần như **không có lựa chọn nào tốt tuyệt đối** — chỉ có lựa chọn phù hợp với ràng buộc cụ thể của bạn. Khả năng gọi tên trade-off một cách rành mạch chính là thứ người phỏng vấn tìm kiếm ở ứng viên senior trở lên.

Áp dụng vào ba cách duy trì `tsvector`:

| Cách làm | Được gì | Mất gì |
|---|---|---|
| **Functional GIN index** `GIN(to_tsvector('english', body))` | Không nhân đôi dữ liệu, không thêm cột, không lo đồng bộ | Truy vấn phải lặp lại đúng biểu thức; gộp nhiều cột thì viết rườm rà |
| **Generated column STORED** + GIN | Truy vấn gọn (`search_vector @@ q`), Postgres tự đồng bộ, gộp nhiều cột và gán `setweight` dễ dàng | Tốn thêm dung lượng đĩa để lưu `tsvector` |
| **Trigger tự viết** | Linh hoạt tối đa, làm được logic phức tạp tùy ý | Nhiều code, dễ quên đường cập nhật, lỗi xảy ra âm thầm — **nên né** |

Quy tắc thực chiến: cần gán trọng số cho nhiều trường → dùng **generated column**; chỉ tìm trên một trường và muốn tiết kiệm đĩa → dùng **functional index**; gần như không bao giờ cần trigger tay nữa.

### 3.4. Tinh chỉnh relevance sâu hơn

**`ts_rank(weights, vector, query, normalization)`** có dạng đầy đủ với bốn tham số:

- **`weights`** — mảng bốn số quyết định điểm cho từng nhãn trọng số. Mặc định là `{0.1, 0.2, 0.4, 1.0}`, tương ứng với D, C, B, A. Nghĩa là một từ khớp ở nhãn A được tính điểm gấp 10 lần so với khớp ở nhãn D.
- **`normalization`** (dịch: *chuẩn hóa*) — một **bitmask** (một số nguyên mà mỗi bit bật/tắt một tùy chọn) quyết định có chia điểm theo độ dài tài liệu hay không.

Vì sao cần chuẩn hóa theo độ dài? Vì nếu không, một bài viết 10.000 chữ gần như luôn thắng một bài 300 chữ — đơn giản vì nó dài nên chứa nhiều từ khóa hơn, chứ không phải vì nó liên quan hơn. Cờ chuẩn hóa chia điểm cho độ dài để cân bằng lại. Ví dụ cờ `32` áp dụng công thức `rank / (rank + 1)`, đưa điểm về khoảng từ 0 tới 1 cho dễ so sánh giữa các truy vấn.

⚠️ **Cảnh báo hiệu năng quan trọng.** Để tính `ts_rank`, Postgres phải **mở `tsvector` của từng tài liệu khớp** ra và đọc. Nếu truy vấn khớp một triệu hàng, nó phải đọc một triệu `tsvector` — công việc này nặng về **I/O** (*Input/Output*, tức thao tác đọc/ghi dữ liệu từ đĩa, thường là chặng chậm nhất trong máy tính).

Ba cách giảm nhẹ: (a) **lọc cứng trước khi xếp hạng** — thêm điều kiện về danh mục, khoảng thời gian để thu hẹp tập ứng viên; (b) **lưu sẵn `tsvector`** trong generated column để khỏi tính lại; (c) chỉ xếp hạng **top-N** thay vì toàn bộ tập khớp.

### 3.5. Tìm theo tiền tố, theo cụm, và chống lỗi chính tả

**Prefix matching** (*khớp theo tiền tố*) — cho phép người dùng gõ nửa từ, dùng ký hiệu `:*`:

```sql
SELECT to_tsquery('english', 'postgr:*');
-- Khớp mọi lexeme bắt đầu bằng "postgr": postgres, postgresql, postgraduate...
-- Đây là cơ chế đằng sau tính năng gợi ý khi người dùng đang gõ dở.
```

**Phrase / proximity search** (*tìm theo cụm / theo khoảng cách*) — dùng `<->` (từ này ngay trước từ kia) hoặc `<N>` (cách nhau đúng N từ):

```sql
SELECT to_tsvector('english', 'data science is fun')
    @@ phraseto_tsquery('english', 'data science');
-- phraseto_tsquery tự sinh ra 'data' <-> 'science'
-- → t (true), vì trong câu gốc "data" đứng ngay trước "science"

SELECT to_tsvector('english', 'science of data')
    @@ phraseto_tsquery('english', 'data science');
-- → f (false), vì thứ tự bị đảo. Đây chính là lúc thông tin VỊ TRÍ
--   mà tsvector lưu ở mục 1.10 phát huy tác dụng.
```

**Fuzzy search — chống lỗi chính tả bằng `pg_trgm`.**

Đây là một giới hạn quan trọng cần thuộc lòng: **FTS không sửa lỗi chính tả**. Nếu người dùng gõ "postgrsql" (thiếu chữ e), stemmer sẽ tạo ra một lexeme khác hẳn `postgresql`, và kết quả trả về là rỗng. FTS chuẩn hóa **biến thể ngữ pháp**, chứ không đoán **ý định** của người gõ sai.

Giải pháp là extension **`pg_trgm`**, dựa trên khái niệm **trigram** (*bộ ba ký tự*). Trigram nghĩa là cắt một chuỗi thành mọi đoạn ba ký tự liên tiếp. Ví dụ chuỗi "postgres" cho ra: `pos`, `ost`, `stg`, `tgr`, `gre`, `res`. Hai chuỗi càng chia sẻ nhiều trigram chung thì càng giống nhau — và một lỗi gõ nhỏ chỉ làm hỏng vài trigram, phần lớn vẫn trùng.

```sql
-- Cài extension (chỉ cần chạy một lần cho mỗi database)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Dựng index GIN theo kiểu trigram trên cột title
CREATE INDEX articles_title_trgm
ON articles
USING GIN (title gin_trgm_ops);   -- gin_trgm_ops báo cho GIN dùng luật trigram

-- Tìm gần đúng
SELECT title,
       similarity(title, 'postgrsql') AS sim   -- similarity trả điểm giống nhau từ 0 tới 1
FROM articles
WHERE title % 'postgrsql'      -- toán tử % nghĩa là "đủ giống theo ngưỡng trigram"
ORDER BY sim DESC;             -- giống nhất lên đầu
```

🧩 **[Ngoài bài gốc]** Kiến trúc tìm kiếm ở production hiện đại thường xếp **ba tầng bổ trợ nhau**: **FTS** bắt đúng từ (chính xác, nhanh, giải thích được), **trigram** cứu lỗi gõ, và **vector search** hiểu nghĩa gần. Ba tầng này không thay thế nhau — mỗi tầng bắt một loại truy vấn mà hai tầng kia bỏ lọt. Phần 4 sẽ ghép chúng lại thành một kiến trúc hoàn chỉnh.

### 3.6. Edge case — các trường hợp biên phải thủ sẵn

**Edge case** (dịch: *trường hợp biên*) là những tình huống hiếm gặp, nằm ở rìa của những gì bạn dự tính, nhưng khi xảy ra thì làm hệ thống hỏng hoặc cho kết quả sai. Người mới thường chỉ test "đường đi đẹp"; người có kinh nghiệm test các rìa. Trong phỏng vấn, chủ động nêu edge case luôn ghi điểm.

**NULL nuốt `tsvector`.** Đã nói ở mục 2.3, nhắc lại vì đây là edge case gây đau đớn nhất trong FTS. Luôn `coalesce`.

**Ngôn ngữ không phải tiếng Anh.** Stemmer `'english'` áp lên tiếng Việt, tiếng Nhật, tiếng Trung sẽ cho kết quả vô nghĩa. Với tiếng Việt: cân nhắc `'simple'` kết hợp `unaccent` để xử lý việc người dùng gõ không dấu, hoặc chuyển sang vector search cho phần ngữ nghĩa. Nếu một bảng chứa nhiều ngôn ngữ, hãy lưu tên config vào một cột riêng và index theo biểu thức `GIN(to_tsvector(lang_col, body))` — cách này cho phép mỗi hàng dùng bộ luật ngôn ngữ của riêng nó.

**Stop-word làm mất cả cụm từ.** Ban nhạc "The Who" là một ví dụ nổi tiếng: cả hai từ đều là stop-word, nên `to_tsvector('english', 'The Who')` cho ra... rỗng. Không cách nào tìm thấy ban nhạc này bằng config `'english'`. Với các trường chứa tên riêng, dùng config `'simple'`.

**Xây index làm khóa ghi.** Câu lệnh `CREATE INDEX` thông thường **khóa** bảng, chặn mọi thao tác ghi cho tới khi xây xong — có thể là hàng giờ với bảng lớn. Trên hệ thống đang chạy thật, luôn dùng **`CREATE INDEX CONCURRENTLY`**: nó xây chậm hơn nhưng không chặn ghi. Tương tự, khi cần xây lại index (ví dụ sau khi đổi config), dùng **`REINDEX INDEX CONCURRENTLY`** (có từ PostgreSQL 12).

**Index bị bỏ qua âm thầm.** Nhắc lại lần cuối vì tầm quan trọng: nếu vế trái của `@@` không phải đúng biểu thức đã index, Postgres lặng lẽ chuyển sang seq scan. Sau mỗi lần thay đổi câu truy vấn tìm kiếm, hãy chạy `EXPLAIN ANALYZE` để xác nhận vẫn thấy `Bitmap Index Scan`.

### 3.7. Bảng so sánh cốt lõi: LIKE/RegEx vs FTS vs Vector search

Đây là bảng đáng chụp màn hình lưu lại, vì nó tóm tắt toàn bộ bức tranh tìm kiếm hiện đại.

| | **LIKE / RegEx** | **FTS (`tsvector`)** | **Vector search (`pgvector`)** |
|---|---|---|---|
| Khớp theo | Mẫu ký tự | **Lexeme** (từ đã chuẩn hóa) | **Ngữ nghĩa** (embedding) |
| "run" có tìm ra "running"? | Không | **Có** (nhờ stemming) | Có (nếu nghĩa đủ gần) |
| "áo phao" có tìm ra "đồ giữ ấm"? | Không | **Không** (khác lexeme hoàn toàn) | **Có** (hiểu nghĩa) |
| Chống lỗi chính tả | Không | Không (cần `pg_trgm`) | Một phần |
| Loại index | Kém (`%...%` = seq scan) | **GIN** (inverted) | **HNSW / IVFFlat** (tìm gần đúng) |
| Độ chính xác | Tuyệt đối | Tuyệt đối theo lexeme | **Gần đúng** (approximate) |
| Cần mô hình AI? | Không | Không | **Có** (mô hình sinh embedding) |
| Giải thích được kết quả? | Có | **Có** ("khớp vì chứa đúng từ") | Khó |

Giải thích hai từ mới trong bảng: **embedding** (nghĩa đen: *sự nhúng vào*) là việc biến một đoạn văn bản thành một dãy số dài — ví dụ 768 số — sao cho hai đoạn văn có nghĩa gần nhau sẽ cho hai dãy số gần nhau trong không gian toán học. **ANN** (viết tắt của *Approximate Nearest Neighbor*, dịch: *láng giềng gần nhất gần đúng*) là họ thuật toán tìm các dãy số gần nhất một cách nhanh nhưng chấp nhận sai sót nhỏ; **HNSW** và **IVFFlat** là hai thuật toán ANN mà `pgvector` cung cấp.

**Bài học rút ra:** FTS mạnh khi người dùng gõ **đúng từ khóa**, dù ở biến thể nào. Nó **không** hiểu từ đồng nghĩa (trừ khi bạn tự cấu hình thesaurus). Chỗ trống đó chính là đất diễn của vector search — và đó là lý do hybrid search tồn tại.

### ✅ Self-check Phần 3

**1. Vì sao GIN giữ được tốc độ khi bảng lớn còn `LIKE '%x%'` thì không? Độ phức tạp mỗi bên là bao nhiêu?**
> *Gợi ý đáp án:* GIN là inverted index, tra thẳng posting list của lexeme rồi chỉ đọc các hàng khớp → thời gian phụ thuộc **số hàng khớp**, không phụ thuộc tổng số hàng. `LIKE '%x%'` không dùng được index → seq scan → `O(N)` theo tổng số hàng.

**2. Bảng ghi rất nhiều, dung lượng index cần nhỏ, cần hỗ trợ tìm cụm — chọn GIN hay GiST?**
> *Gợi ý đáp án:* GiST. Nó xây và cập nhật nhanh hơn, index nhỏ hơn; đổi lại truy vấn chậm hơn vì lossy nên phải recheck. Đây đúng là một trade-off, và nêu rõ cái mất mới là câu trả lời hoàn chỉnh.

**3. Người dùng hay gõ sai tên sản phẩm. FTS thuần giải quyết được không? Cần bổ sung gì?**
> *Gợi ý đáp án:* Không — FTS chuẩn hóa biến thể ngữ pháp chứ không sửa lỗi gõ. Bổ sung extension `pg_trgm` với index `gin_trgm_ops`, dùng toán tử `%` và hàm `similarity` để khớp gần đúng.

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Ở ba phần trước, câu hỏi luôn là "làm thế nào". Ở phần này, câu hỏi đổi thành **"khi nào nên, khi nào không nên, và cái gì sẽ gãy trước"**. Đó là khác biệt cốt lõi giữa senior và staff.

### 4.1. Scale: từ vài nghìn tới vài chục triệu tài liệu

**Scale** (dịch: *quy mô / mở rộng quy mô*) nói về việc hệ thống còn chạy tốt không khi lượng dữ liệu hoặc lượng người dùng tăng lên nhiều lần. Có hai kiểu mở rộng: **scale up / vertical scaling** (*mở rộng dọc* — mua máy mạnh hơn: nhiều RAM hơn, CPU nhanh hơn) và **scale out / horizontal scaling** (*mở rộng ngang* — thêm nhiều máy chạy song song).

**Bottleneck** (dịch: *điểm nghẽn*, nghĩa đen là *cổ chai*) là bộ phận chậm nhất trong hệ thống, thứ quyết định tốc độ của cả dây chuyền. Cứ hình dung một chai nước: dù thân chai to đến đâu, nước chảy ra nhanh hay chậm là do cái cổ chai quyết định. Tối ưu bất cứ chỗ nào **không phải** bottleneck đều là công cốc.

**Câu hỏi ở tầm staff không phải "dùng GIN hay GiST" mà là "chỗ nào sẽ gãy khi hệ thống lớn lên".** Với FTS trong Postgres, có ba bottleneck chính, theo thứ tự thường gặp.

**Điểm mạnh cấu trúc trước đã.** Vì GIN là inverted index, thời gian truy vấn phụ thuộc số hàng khớp chứ không phải tổng số hàng. Điều đó có nghĩa là Postgres FTS chạy dưới một giây trên hàng triệu tài liệu — **miễn là truy vấn đủ chọn lọc**. Đây là lý do Postgres FTS thật sự thay thế được Elasticsearch cho phần lớn ứng dụng quy mô vừa và nhỏ.

**Bottleneck 1 — I/O của khâu xếp hạng.** Như đã cảnh báo ở mục 3.4, `ts_rank` phải mở `tsvector` của **từng tài liệu khớp**. Khi người dùng tìm một từ phổ biến khớp một triệu hàng, bạn phải đọc một triệu `tsvector` từ đĩa. Đây thường là bottleneck xuất hiện **sớm nhất** và gây bất ngờ nhất, vì hệ thống chạy êm với các truy vấn hiếm rồi đột nhiên treo khi có người tìm một từ phổ thông.
> **Cách chữa:** lọc cứng bằng metadata (danh mục, khoảng thời gian, tenant) **trước** khi xếp hạng để thu hẹp tập ứng viên; lưu sẵn `tsvector` trong generated column; chỉ xếp hạng top-N.

**Bottleneck 2 — write amplification.** **Write amplification** (dịch: *khuếch đại ghi*) là hiện tượng một thao tác ghi của bạn kéo theo nhiều thao tác ghi thật xuống đĩa. Ở đây: sửa một bài viết không chỉ ghi lại hàng đó, mà còn phải cập nhật GIN index cho **mọi lexeme** trong bài — có thể là hàng trăm mục.
> **Cách chữa:** bật `fastupdate` để gom thay đổi lại; hoặc cân nhắc GiST nếu ghi áp đảo đọc; hoặc tách hẳn nội dung tìm kiếm sang một bảng riêng để bảng chính không bị ảnh hưởng.

**Bottleneck 3 — thời gian xây index.** GIN xây chậm và `CREATE INDEX` thường khóa ghi.
> **Cách chữa:** `CREATE INDEX CONCURRENTLY`, và tăng `maintenance_work_mem` trước khi xây.

**Mở rộng ngang cho FTS.** Hai kỹ thuật chính:

**Read replica** (dịch: *bản sao chỉ đọc*) là một bản sao của database, tự đồng bộ từ máy chính, chỉ phục vụ đọc. Vì tìm kiếm là thao tác đọc thuần túy, đẩy toàn bộ lưu lượng search sang replica là cách giảm tải cực kỳ hiệu quả cho máy chính vốn phải lo ghi.

**Partitioning** (dịch: *phân mảnh*) là chia một bảng khổng lồ thành nhiều bảng con theo một tiêu chí — thường là theo thời gian (mỗi tháng một mảnh) hoặc theo **tenant** (mỗi khách hàng doanh nghiệp một mảnh). Mỗi mảnh có GIN index riêng, nhỏ hơn nhiều. Khi truy vấn có kèm điều kiện lọc theo tiêu chí đó, bộ lập kế hoạch truy vấn sẽ **prune** (*cắt tỉa*) — bỏ qua hẳn những mảnh không liên quan, không thèm đụng tới.

### 4.2. Quyết định kiến trúc: PostgreSQL FTS hay Elasticsearch?

**Elasticsearch** (và bản mã nguồn mở tương đương là **OpenSearch**) là một công cụ tìm kiếm chuyên dụng, chạy như một hệ thống riêng biệt bên cạnh database của bạn. Đây là lựa chọn mặc định của rất nhiều đội — và câu hỏi ở tầm staff là: **có thật sự cần không?**

| Tiêu chí | **PostgreSQL FTS** | **Elasticsearch / OpenSearch** |
|---|---|---|
| Hạ tầng phải thêm | **Không thêm gì** — dùng chính database đang có | Một cụm máy riêng, chạy trên JVM (máy ảo Java — nền tảng mà Elasticsearch chạy trên đó, cần biết cách chỉnh bộ nhớ riêng), cần người vận hành |
| Đồng bộ dữ liệu | **Không cần** — dữ liệu vốn đã ở đó | Phải xây pipeline đồng bộ, và pipeline này sẽ hỏng |
| Tính nhất quán | **ACID, thấy ngay lập tức** | Nhất quán sau cùng — dữ liệu mới có độ trễ trước khi tìm được |
| Nối với dữ liệu nghiệp vụ | **Một câu SQL JOIN** | Hai hệ thống, phải viết code ghép ở tầng ứng dụng |
| Relevance nâng cao (BM25, analyzer phong phú) | Cơ bản (`ts_rank`) | **Mạnh hơn nhiều** |
| Quy mô rất lớn, phân tích văn bản | Khá | **Vượt trội** |
| Chi phí và độ phức tạp vận hành | **Thấp** | Cao |

Giải thích các từ trong bảng:

**ACID** là viết tắt của bốn tính chất mà một giao dịch database phải đảm bảo: *Atomicity* (nguyên tử — làm hết hoặc không làm gì), *Consistency* (nhất quán), *Isolation* (cô lập — các giao dịch không giẫm chân nhau), *Durability* (bền vững — đã ghi là không mất kể cả mất điện).

**Eventually consistent** (dịch: *nhất quán sau cùng*) nghĩa là dữ liệu mới ghi sẽ **cuối cùng** xuất hiện ở mọi nơi, nhưng có độ trễ. Với search, điều này nghĩa là người dùng vừa đăng bài xong tìm lại chưa thấy — một trải nghiệm gây khó chịu mà bạn phải thiết kế để né.

**ETL** (viết tắt của *Extract - Transform - Load*) là quy trình rút dữ liệu ra khỏi hệ này, biến đổi, rồi nạp vào hệ kia. Đây chính là "pipeline đồng bộ" mà bạn buộc phải xây và bảo trì nếu chọn Elasticsearch.

**BM25** là một công thức tính điểm liên quan tiên tiến hơn `ts_rank`, đang là tiêu chuẩn công nghiệp cho tìm kiếm theo từ khóa. Postgres lõi chưa có BM25, nhưng đã có extension mang nó vào (xem phần ghi chú cuối).

**On-call** (dịch: *trực sự cố*) là việc kỹ sư phải sẵn sàng bị gọi dậy giữa đêm khi hệ thống hỏng. Mỗi hệ thống bạn thêm vào kiến trúc là thêm một thứ có thể đánh thức ai đó lúc 3 giờ sáng. Đây là chi phí thật, và ở tầm staff bạn phải tính nó vào quyết định.

> **Câu chốt tầm staff:** *"Với ứng dụng quy mô vừa và nhỏ, Postgres FTS xóa bỏ nhu cầu chạy thêm một cụm Elasticsearch — cùng một database, có ACID, join thẳng bằng SQL, không phải đồng bộ, không thêm hệ thống nào để trực đêm."* Chọn Elasticsearch khi tìm kiếm **chính là sản phẩm** của bạn, khi bạn cần relevance/analyzer cao cấp, hoặc khi quy mô thật sự rất lớn.

### 4.3. 🧩 [Ngoài bài gốc — ĐIỂM VÀNG] Hybrid Search: FTS kết hợp pgvector

Đây là chỗ toàn bộ khóa học hội tụ, và cũng là nội dung dễ ghi điểm nhất trong phỏng vấn năm 2026.

**Điểm mạnh và điểm mù của mỗi bên:**

**FTS (tìm theo từ khóa / lexeme)** cực chính xác với thuật ngữ chuyên ngành, mã sản phẩm, tên riêng, số phiên bản. Nó nhanh, rẻ, và **giải thích được** — bạn luôn chỉ ra được "kết quả này lên đầu vì nó chứa đúng những từ này". Nhưng nó **mù ngữ nghĩa**: "xe hơi" và "ô tô" là hai lexeme khác nhau, hết chuyện.

**Vector search (tìm theo ngữ nghĩa)** hiểu được từ đồng nghĩa và cách diễn đạt khác nhau. Nhưng nó hay trượt ở đúng chỗ FTS mạnh: một mã sản phẩm hiếm như "SKU-88XZ" gần như không có ý nghĩa ngữ nghĩa nào, nên embedding của nó vô dụng. Ngoài ra kết quả khó giải thích cho người dùng và cho chính đội kỹ thuật.

Nhìn vào hai đoạn trên, kết luận là hiển nhiên: **hai kỹ thuật này không cạnh tranh, chúng bù khuyết cho nhau**. Chạy cả hai rồi hợp nhất kết quả — đó chính là **hybrid search**.

**Cách hợp nhất phổ biến nhất là RRF.** **RRF** (viết tắt của *Reciprocal Rank Fusion*, dịch: *hợp nhất theo nghịch đảo thứ hạng*) hoạt động theo một ý tưởng đơn giản đến bất ngờ: **đừng cộng điểm, hãy cộng thứ hạng**.

Vì sao không cộng điểm? Vì điểm của `ts_rank` và điểm khoảng cách vector nằm trên hai thang đo hoàn toàn khác nhau, không có cách nào so sánh trực tiếp cho công bằng. Nhưng **thứ hạng** thì luôn so sánh được: đứng thứ nhất ở nhánh nào cũng là đứng thứ nhất.

Công thức RRF: mỗi tài liệu nhận điểm `1 / (k + thứ_hạng)` từ mỗi nhánh, rồi cộng lại. Hằng số `k` (thường chọn 60) có tác dụng làm mượt, tránh việc vị trí số 1 áp đảo quá mức so với vị trí số 2. Tài liệu nào lọt vào top của **cả hai** nhánh sẽ có tổng điểm cao nhất — đúng như trực giác.

```sql
-- Ý tưởng: lấy top 50 từ nhánh keyword và top 50 từ nhánh semantic,
-- rồi hợp nhất thứ hạng bằng RRF.

WITH kw AS (                                  -- WITH ... AS: đặt tên cho một truy vấn con
  -- ===== NHÁNH 1: KEYWORD (FTS) =====
  SELECT id,
         row_number() OVER (ORDER BY ts_rank(search_vector, q) DESC) AS r
         -- row_number() gán số thứ tự 1, 2, 3... theo điểm ts_rank giảm dần
         -- → đây chính là "thứ hạng" của tài liệu ở nhánh keyword
  FROM docs, websearch_to_tsquery('english', 'wireless headphones') q
  WHERE search_vector @@ q                    -- chỉ lấy tài liệu thật sự khớp từ khóa
  LIMIT 50
),
sem AS (
  -- ===== NHÁNH 2: SEMANTIC (pgvector) =====
  SELECT id,
         row_number() OVER (ORDER BY embedding <=> :query_vec) AS r
         -- <=> là toán tử tính khoảng cách cosine giữa hai vector;
         -- càng NHỎ càng gần nghĩa, nên sắp tăng dần
         -- :query_vec là embedding của câu người dùng tìm, do ứng dụng truyền vào
  FROM docs
  ORDER BY embedding <=> :query_vec
  LIMIT 50
)
-- ===== HỢP NHẤT BẰNG RRF =====
SELECT id,
       SUM(1.0 / (60 + r)) AS rrf_score       -- 60 là hằng số làm mượt của RRF
FROM (SELECT id, r FROM kw
      UNION ALL                                -- gộp hai danh sách lại, giữ cả trùng lặp
      SELECT id, r FROM sem) t
GROUP BY id                                    -- gom theo tài liệu
ORDER BY rrf_score DESC                        -- điểm hợp nhất cao nhất lên đầu
LIMIT 10;
```

**Vì sao staff bắt buộc phải biết điều này:** hybrid search là **kiến trúc truy hồi mặc định** cho tìm kiếm hiện đại và cho **RAG** (viết tắt của *Retrieval-Augmented Generation* — kỹ thuật cho mô hình ngôn ngữ tra cứu tài liệu thật trước khi trả lời, nhằm giảm bịa đặt). Và điểm mấu chốt: **`pgvector` cộng FTS làm được toàn bộ việc này bên trong một cái Postgres duy nhất** — không cần chạy song song một cơ sở dữ liệu vector chuyên dụng và một cụm Elasticsearch.

Hãy để ý: đây chính là lúc luận điểm "PostgreSQL mở rộng được" ở mục 1.5 trả cổ tức. Một quyết định tưởng như hàn lâm ở Phần 1 hóa ra quyết định toàn bộ hình dạng kiến trúc ở Phần 4. Nói được mạch liên kết này trong phỏng vấn là dấu hiệu rõ ràng của tư duy hệ thống.

### 4.4. Chi phí, vận hành, giám sát, và các kiểu hỏng

**Chi phí.** FTS gần như **miễn phí về hạ tầng** vì nó tận dụng Postgres bạn đã trả tiền. So với việc vận hành một cụm Elasticsearch riêng, khoản tiết kiệm không chỉ là tiền máy chủ mà còn là **thời gian con người**: ít hệ thống hơn nghĩa là ít tài liệu vận hành, ít quy trình triển khai, ít bề mặt trực sự cố.

**Giám sát (monitoring).** Những chỉ số bạn phải theo dõi:

- **p95 và p99 độ trễ truy vấn tìm kiếm.** **p95** nghĩa là "95% số truy vấn nhanh hơn con số này". Người ta dùng p95/p99 thay vì trung bình, vì trung bình che giấu những trường hợp tệ nhất — mà chính những trường hợp tệ nhất mới là thứ người dùng nhớ và phàn nàn.
- **Tỉ lệ truy vấn rơi về sequential scan** — dấu hiệu index bị bỏ qua.
- **Kích thước GIN index** — nó phình theo thời gian và ăn đĩa.
- **Độ trễ ghi** — để phát hiện write amplification do GIN.

Công cụ: **`pg_stat_statements`** (một extension thống kê mọi câu lệnh đã chạy, tổng hợp thời gian và số lần gọi) và `EXPLAIN ANALYZE` cho từng truy vấn nghi ngờ.

**Failure modes** (dịch: *các kiểu hỏng*) — liệt kê trước những cách hệ thống có thể hỏng là một thói quen rất đặc trưng của staff engineer:

1. **Biểu thức hoặc config lệch** → index bị bỏ qua → độ trễ tăng dần một cách âm thầm, không có báo lỗi.
2. **NULL nuốt `tsvector`** → một số tài liệu biến mất khỏi kết quả tìm kiếm mà không ai biết. Đây là kiểu hỏng nguy hiểm nhất vì hệ thống trông vẫn hoàn toàn bình thường.
3. **Truy vấn với từ siêu phổ biến** → khớp hàng triệu hàng → khâu xếp hạng ngốn I/O → treo cả database.
4. **Đổi text search configuration hoặc ngôn ngữ** → toàn bộ index cũ trở nên vô nghĩa → phải xây lại từ đầu.
5. **Vùng đệm `fastupdate` của GIN phình to** khi ghi nhiều mà chưa được **vacuum** (thao tác dọn dẹp định kỳ của Postgres) → truy vấn chậm dần.

### 4.5. Ảnh hưởng tới tổ chức và cách nói chuyện với người không rành kỹ thuật

**Stakeholder** (dịch: *bên liên quan*) là những người có lợi ích gắn với dự án nhưng không nhất thiết hiểu kỹ thuật: quản lý sản phẩm, giám đốc, bộ phận kinh doanh. Một staff engineer phải trình bày được quyết định kỹ thuật bằng ngôn ngữ của họ — tức là bằng **rủi ro, chi phí và thời gian**, chứ không phải bằng thuật ngữ.

**Cách nói với quản lý sản phẩm hoặc sếp:**

> *"Chúng ta thêm được tính năng tìm kiếm mạnh mà **không phải dựng thêm hệ thống nào** — dùng chính cơ sở dữ liệu đang có. Đội ngũ đã quen vận hành nó, nên rủi ro thấp và không phát sinh chi phí hạ tầng. Nếu sau này lượng người dùng tăng mạnh và tìm kiếm trở thành tính năng cốt lõi, chúng ta sẽ cân nhắc một hệ thống chuyên dụng — nhưng đó là bài toán của lúc đã thành công, không phải bây giờ."*

Hãy để ý cách diễn đạt trên: không có chữ nào là "GIN", "tsvector", hay "inverted index". Thay vào đó là rủi ro thấp, không tốn thêm tiền, và một lộ trình rõ ràng cho tương lai. Đó là ngôn ngữ mà stakeholder nghe được.

**Ảnh hưởng tới roadmap.** Quyết định về text search configuration — chọn ngôn ngữ nào, có dùng thesaurus không, gán trọng số ra sao — ảnh hưởng tới chất lượng tìm kiếm của **toàn bộ sản phẩm**. Và mỗi lần đổi config là phải xây lại index cho toàn bảng, tốn thời gian và có rủi ro. Vì vậy hãy đối xử với config như một **migration** (thay đổi cấu trúc database): có phiên bản, có kế hoạch, có phương án quay lui.

**Ảnh hưởng tới cấu trúc đội ngũ.** Chọn FTS trong Postgres nghĩa là đội backend hiện tại tự lo được toàn bộ. Chọn Elasticsearch nghĩa là ai đó phải học vận hành một cụm JVM, phải trực sự cố cho nó, và về lâu dài có thể phải tuyển người chuyên trách. Đây là một luận điểm **tổ chức**, không phải kỹ thuật — và biết đưa loại luận điểm này ra bàn là điều người ta chờ đợi ở tầm staff.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff engineer

> **Đề bài:** *"Thiết kế chức năng tìm kiếm cho một trang tin tức / thương mại điện tử có 20 triệu bài viết hoặc sản phẩm. Yêu cầu: hỗ trợ tìm theo từ khóa, chịu được lỗi chính tả, hiểu được từ đồng nghĩa, lọc theo danh mục và thời gian, độ trễ p95 dưới 150ms, và hỗ trợ nhiều ngôn ngữ."*

Khung trả lời gồm bốn nhịp: **làm rõ đề → thiết kế → mở rộng quy mô → nói rõ đánh đổi**. Điều quan trọng nhất không phải là đưa ra "đáp án đúng", mà là **nói to những đánh đổi bạn đang cân nhắc**.

**Nhịp 1 — Làm rõ đề (clarify).** Đừng vẽ ngay. Hỏi trước:
- QPS (**QPS** = *Queries Per Second*, số truy vấn mỗi giây) dự kiến là bao nhiêu?
- Nội dung được cập nhật thường xuyên tới mức nào? (quyết định GIN hay GiST)
- Ưu tiên **recall** hay **precision**? (*recall* = không bỏ sót kết quả đúng; *precision* = không trả về kết quả rác. Hai thứ này đánh đổi lẫn nhau.)
- Ngân sách hạ tầng ra sao? Có bắt buộc phải giải thích được vì sao một kết quả xếp hạng cao không?

**Nhịp 2 — Mô hình dữ liệu.** Một bảng `docs` chứa: metadata (danh mục, thời gian, ngôn ngữ), cột `search_vector` là generated column có `setweight` theo thứ tự title > body > tags, và cột `embedding` kiểu `vector`. Partition bảng theo ngôn ngữ hoặc theo thời gian.

**Nhịp 3 — Index.** GIN trên `search_vector` cho nhánh từ khóa; GIN với `gin_trgm_ops` trên title cho nhánh chống lỗi chính tả; HNSW trên `embedding` cho nhánh ngữ nghĩa. **Tất cả nằm trong cùng một Postgres.**

**Nhịp 4 — Truy vấn theo kiểu hybrid.** Ba nhánh chạy song song — FTS (dùng `websearch_to_tsquery` cho an toàn với input người dùng), trigram (bắt lỗi gõ), vector (bắt đồng nghĩa) — rồi hợp nhất bằng RRF. Điểm mấu chốt về hiệu năng: **lọc cứng theo danh mục và thời gian TRƯỚC**, để mỗi nhánh chỉ phải xếp hạng một tập nhỏ.

**Nhịp 5 — Đạt mục tiêu độ trễ.** Lọc chọn lọc trước khi xếp hạng; lưu sẵn `tsvector`; chỉ xếp hạng top-N; đẩy lưu lượng tìm kiếm sang read replica; và quan trọng nhất — **đo p95 thật trên dữ liệu thật**, đừng suy đoán.

**Nhịp 6 — Đa ngôn ngữ.** Lưu tên config vào một cột, index theo `GIN(to_tsvector(lang_col, body))` để mỗi hàng dùng bộ luật ngôn ngữ riêng; dùng mô hình embedding đa ngữ cho nhánh vector.

**Nhịp 7 — Vận hành.** Giám sát tỉ lệ rơi về seq scan, p99, kích thước index, độ trễ ghi. Dùng `REINDEX CONCURRENTLY` khi đổi config. Khi đổi mô hình embedding, triển khai kiểu **blue-green** (dựng song song bộ index mới, kiểm chứng xong mới chuyển toàn bộ lưu lượng sang) — vì embedding cũ và mới **không so sánh được với nhau**, trộn lẫn sẽ cho kết quả sai hoàn toàn.

**Nhịp 8 — Biết giới hạn của lựa chọn mình đưa ra.** Kết thúc bằng câu này: *"Nếu tìm kiếm trở thành workload chính, cần BM25 và analyzer cao cấp, hoặc quy mô vượt xa mức hiện tại, tôi sẽ chuyển nhánh từ khóa sang Elasticsearch và giữ pgvector cho nhánh ngữ nghĩa."* Chủ động nêu ra khi nào giải pháp của chính mình không còn phù hợp — đó là dấu hiệu staff rõ ràng nhất trong một buổi phỏng vấn.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trước buổi phỏng vấn 30 phút. Mọi thứ ở đây đã được giải thích đầy đủ ở trên — đây chỉ là bản rút gọn để nhắc lại.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ tiếng Anh | Định nghĩa một dòng |
|---|---|
| **DBMS / RDBMS** | Phần mềm quản lý cơ sở dữ liệu / loại tổ chức dữ liệu thành các bảng có quan hệ |
| **Primary key / Foreign key** | Cột định danh duy nhất mỗi hàng / cột trỏ tới khóa chính của bảng khác |
| **BLOB** | Binary Large Object — lưu ảnh, video, file dạng nhị phân trong database |
| **ORDBMS / extensibility** | Quan hệ - đối tượng; PostgreSQL cắm thêm được kiểu dữ liệu, hàm, index, extension |
| **Full-text search (FTS)** | Tìm tài liệu ngôn ngữ tự nhiên theo lexeme, không theo ký tự |
| **Token / Tokenize** | Mẩu văn bản cắt ra từ câu / hành động cắt đó |
| **Stemming** | Rút từ về dạng gốc chung: "running" → "run" |
| **Lexeme** | Từ đã chuẩn hóa xong — đơn vị so khớp thật sự của FTS |
| **Stop-word** | Từ quá phổ biến nên bị loại bỏ: "the", "is", "a" |
| **`tsvector`** | Danh sách lexeme đã loại trùng, sắp xếp, kèm vị trí — tài liệu đã tiền xử lý |
| **`tsquery`** | Truy vấn dạng lexeme nối bằng `&` (AND), `\|` (OR), `!` (NOT), `<->` (liền kề) |
| **`@@`** | Toán tử so khớp một `tsvector` với một `tsquery`, trả true/false |
| **`to_tsvector` / `to_tsquery`** | Hàm tạo tsvector từ text / tạo tsquery từ chuỗi có toán tử |
| **`plainto_` / `phraseto_` / `websearch_to_tsquery`** | Tự AND các từ / tìm nguyên cụm / cú pháp kiểu Google, an toàn với input người dùng |
| **Text search configuration** | Bộ luật xử lý ngôn ngữ: `'english'` (stem + bỏ stop-word), `'simple'` (chỉ lowercase) |
| **Parser / Dictionary** | Bộ tách token và gán loại / bộ biến token thành lexeme hoặc loại bỏ |
| **Inverted index** | Bảng tra "từ → danh sách tài liệu chứa từ đó" — như mục lục cuối sách |
| **GIN / GiST** | Inverted index chính xác, chuẩn cho FTS / signature tree lossy, xây nhanh, index nhỏ |
| **Posting list** | Danh sách các hàng chứa một lexeme, lưu trong GIN |
| **`setweight` / `ts_rank` / `ts_rank_cd`** | Gán trọng số A-D / chấm điểm theo tần suất / chấm điểm có thưởng khi từ gần nhau |
| **Functional index / Generated column** | Index trên biểu thức / cột Postgres tự sinh và tự cập nhật (`STORED`, PG 12+) |
| **`coalesce`** | Trả về giá trị thay thế khi gặp NULL — bắt buộc dùng khi ghép cột thành tsvector |
| **`pg_trgm` / trigram** | Extension khớp gần đúng chống lỗi chính tả, dựa trên các đoạn 3 ký tự |
| **Trade-off / Edge case / Bottleneck** | Sự đánh đổi / trường hợp biên / điểm nghẽn quyết định tốc độ cả hệ thống |
| **Hybrid search / RRF** | Kết hợp FTS và vector search / hợp nhất bằng cách cộng nghịch đảo thứ hạng |
| **BM25** | Công thức chấm điểm liên quan tiên tiến, tiêu chuẩn công nghiệp cho keyword search |

### 5.2. Core concepts — nếu chỉ được nhớ 10 điều

1. **FTS khớp theo lexeme**, không theo ký tự — nên "run" tìm ra được "running", điều `LIKE` vĩnh viễn không làm được.
2. **`tsvector @@ tsquery` là trái tim của FTS**; cả tài liệu lẫn truy vấn đều đi qua **cùng một** bộ chuẩn hóa nên chúng gặp nhau ở tầng lexeme.
3. **GIN inverted index** khiến thời gian truy vấn phụ thuộc **số hàng khớp**, không phụ thuộc tổng số hàng — đó là lý do FTS vẫn nhanh khi bảng lớn.
4. Cách làm chuẩn 2026: **generated column STORED** hoặc **functional GIN index**; đừng viết trigger đồng bộ bằng tay nữa.
5. Input của người dùng luôn đi qua **`websearch_to_tsquery`**, không bao giờ nhét thẳng vào `to_tsquery`.
6. **`setweight` (A/B/C/D) + `ts_rank`** điều khiển relevance ngay trong database, không cần code ở tầng ứng dụng.
7. **Luôn `coalesce(col, '')`** khi ghép nhiều cột — một NULL là cả hàng biến mất khỏi kết quả, âm thầm không báo lỗi.
8. **FTS mù ngữ nghĩa** ("xe hơi" ≠ "ô tô") và **không sửa lỗi chính tả**. Bù bằng vector search và `pg_trgm`.
9. **Postgres FTS thay được Elasticsearch** cho ứng dụng vừa và nhỏ; chọn ES khi tìm kiếm là sản phẩm chính hoặc quy mô rất lớn.
10. **Extensibility của Postgres là luận điểm chiến lược**: keyword, fuzzy và semantic search cùng nằm trong một database, không thêm hệ thống nào.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Mục lục cuối sách"** — giải thích inverted index và GIN trong đúng một câu. Nhớ nói thêm vế quan trọng: *thời gian tra không phụ thuộc độ dày cuốn sách*.
- **"run = running = ran"** — giải thích lexeme và stemming ngay lập tức, ai cũng hiểu.
- **"Từ khóa và ý nghĩa"** — FTS bắt đúng từ, vector hiểu nghĩa, hybrid là cả hai.
- **"Một database, ba tầng tìm kiếm"** — lexeme (FTS) + lỗi gõ (trigram) + ngữ nghĩa (vector).
- **"Search là một tính năng hay là cả sản phẩm?"** — kim chỉ nam để quyết định Postgres FTS hay Elasticsearch.
- **"Mỗi hệ thống thêm vào là một người bị đánh thức lúc 3 giờ sáng"** — cách nói chi phí vận hành mà ai cũng thấm.

### 5.4. Code cần thuộc lòng

**(a) FTS tối thiểu kèm index — interviewer hay bảo viết tại chỗ:**
```sql
CREATE INDEX docs_fts ON docs USING GIN (to_tsvector('english', body));

SELECT title FROM docs
WHERE to_tsvector('english', body)
   @@ websearch_to_tsquery('english', 'postgres index');
```

**(b) Generated column có trọng số kèm xếp hạng — bản "production":**
```sql
ALTER TABLE docs ADD COLUMN sv tsvector GENERATED ALWAYS AS (
  setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
  setweight(to_tsvector('english', coalesce(body,  '')), 'B')
) STORED;

CREATE INDEX ON docs USING GIN (sv);

SELECT title, ts_rank(sv, q) AS r
FROM docs, websearch_to_tsquery('english', :input) q
WHERE sv @@ q
ORDER BY r DESC
LIMIT 10;
```

**(c) Chạy tay `tsvector` — chứng tỏ bạn hiểu bản chất chứ không học vẹt:**
```
'The quick brown foxes are running'
  → tokenize:      The | quick | brown | foxes | are | running
  → bỏ stop-word:  (bỏ the, are)
  → stemming:      foxes → fox,  running → run
  → tsvector:      'brown':3 'fox':4 'quick':2 'run':6
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Khác nhau giữa FTS và `LIKE` là gì?"**
> `LIKE '%x%'` so khớp ký tự, không dùng được B-tree index nên rơi về seq scan `O(N)`, không hiểu biến thể từ, và khớp nhầm vào giữa từ khác. FTS so khớp lexeme, dùng GIN inverted index nên thời gian phụ thuộc số kết quả chứ không phải tổng số hàng, đồng thời stem và bỏ stop-word.

**2. "`tsvector` và `tsquery` là gì, `@@` làm gì?"**
> `tsvector` là tài liệu đã tiền xử lý thành danh sách lexeme loại trùng, sắp xếp, kèm vị trí. `tsquery` là truy vấn cũng ở dạng lexeme, nối bằng toán tử logic. `@@` kiểm tra `tsvector` có thỏa `tsquery` hay không.

**3. [TRADE-OFF] "Chọn GIN hay GiST?"**
> GIN: chính xác, truy vấn nhanh, nhưng xây chậm, ngốn RAM, index to → mặc định cho hệ đọc nhiều. GiST: lossy nên phải recheck (truy vấn chậm hơn), bù lại xây nhanh, index nhỏ, chịu cập nhật tốt → chọn khi ghi nhiều hoặc bị giới hạn dung lượng. *Luôn nói ra cả cái mất, đừng chỉ nói cái được.*

**4. [BẪY] "Index FTS đã tạo rồi mà truy vấn vẫn chậm, vì sao?"**
> Khả năng cao là biểu thức trong `WHERE` không trùng khít biểu thức lúc `CREATE INDEX` — thiếu tham số config, hoặc dùng dạng một tham số thay vì hai. Index bị bỏ qua âm thầm, không báo lỗi. Kiểm tra bằng `EXPLAIN ANALYZE`, tìm `Bitmap Index Scan`; nếu thấy `Seq Scan` là dính lỗi này.

**5. [BẪY] "Một bài viết rõ ràng chứa từ khóa nhưng không bao giờ xuất hiện trong kết quả. Vì sao?"**
> Gần như chắc chắn là một cột nguồn bị NULL khi ghép mà quên `coalesce` → cả `tsvector` của hàng đó thành NULL → hàng biến mất. Kiểm tra bằng `WHERE search_vector IS NULL`, chữa bằng `coalesce(col, '')`.

**6. "Xử lý input thô của người dùng thế nào?"**
> Dùng `websearch_to_tsquery` (cú pháp kiểu Google, an toàn với ký tự lạ) hoặc `plainto_tsquery`. Tuyệt đối không đưa input thô vào `to_tsquery` vì sẽ gây syntax error và lỗi 500.

**7. "FTS có hiểu từ đồng nghĩa không? Nếu cần thì làm sao?"**
> Không, trừ khi bạn tự cấu hình thesaurus dictionary. Muốn hiểu nghĩa thật sự thì phải thêm vector search bằng `pgvector`, rồi kết hợp hai nhánh thành hybrid search và hợp nhất bằng RRF.

**8. "FTS có sửa được lỗi chính tả không?"**
> Không. FTS chuẩn hóa biến thể ngữ pháp chứ không đoán ý định người gõ sai. Cần extension `pg_trgm` với index `gin_trgm_ops`, dùng toán tử `%` và hàm `similarity`.

**9. [SCALE] "Có 20 triệu tài liệu, khi nào nên bỏ Postgres FTS để sang Elasticsearch?"**
> Khi tìm kiếm trở thành **workload** chính (*workload* = khối lượng công việc chính mà hệ thống phải gánh), khi cần BM25 và analyzer cao cấp, hoặc khi quy mô vượt xa mức Postgres phục vụ thoải mái. Trước ngưỡng đó, Postgres FTS thắng ở sự đơn giản trong vận hành, khả năng JOIN thẳng với dữ liệu nghiệp vụ, và tính nhất quán ACID — đồng thời vẫn giữ được `pgvector` cho nhánh ngữ nghĩa trong cùng một hệ.

**10. [SCALE] "Bottleneck đầu tiên của FTS khi lớn lên là gì?"**
> Thường là I/O của khâu xếp hạng: `ts_rank` phải mở `tsvector` của từng tài liệu khớp, nên một truy vấn với từ phổ biến khớp hàng triệu hàng sẽ ngốn I/O khủng khiếp. Chữa bằng cách lọc cứng theo metadata trước khi xếp hạng, lưu sẵn `tsvector`, và chỉ xếp hạng top-N.

### 5.6. Câu trả lời "one-liner" đắt giá

- *"FTS matches lexemes, not characters — that's why 'run' finds 'running' and `LIKE` never will."*
- *"A GIN inverted index makes search time scale with the number of matches, not the size of the table."*
- *"For small-to-medium apps, Postgres FTS deletes your Elasticsearch cluster — same database, ACID, one SQL join, nothing to sync, nobody paged at 3 a.m."*
- *"FTS knows words, vectors know meaning — hybrid search is just both, fused with RRF, inside one Postgres."*
- *"The strategic feature isn't `tsvector` — it's Postgres being extensible enough to do keyword, fuzzy, and semantic search without adding a single new system."*
- *"NULL doesn't throw an error in FTS; it just quietly deletes a row from every search result. That's why `coalesce` isn't optional."*

---

## 📌 Ghi chú cuối

**Kiểm chứng phiên bản khi ôn.** Tính tới tháng 7/2026, bản PostgreSQL ổn định mới nhất là **18.4**; PostgreSQL 19 đang ở giai đoạn beta (Beta 2 ra ngày 16/07/2026), bản chính thức dự kiến tháng 9/2026. Hai mốc phiên bản cần nhớ: `websearch_to_tsquery` cần **PostgreSQL 11+**, generated column `STORED` cần **PostgreSQL 12+**. Tài liệu chính thức về FTS nằm ở `postgresql.org/docs/current/textsearch.html`.

**Thực hành — đây là phần quan trọng nhất.** Đọc xong giáo trình này bạn *biết*, nhưng phải gõ tay thì mới *thuộc*. Lộ trình gợi ý:

1. Dựng một Postgres bằng Docker (một công cụ chạy phần mềm trong môi trường đóng gói sẵn, không cần cài đặt lằng nhằng).
2. Chạy lại đúng ba câu lệnh `to_tsvector` / `to_tsquery` / `@@` ở mục 1.11 và đối chiếu kết quả với phần chạy tay ở mục 1.10.
3. Tạo một bảng vài nghìn hàng, thêm generated column có `setweight`, dựng GIN index.
4. Chạy `EXPLAIN ANALYZE` để **tận mắt nhìn thấy** dòng `Bitmap Index Scan` — rồi cố tình viết sai biểu thức để nhìn thấy nó chuyển thành `Seq Scan`. Khoảnh khắc nhìn thấy sự khác biệt này có giá trị hơn đọc mười trang lý thuyết.
5. Cài `pgvector`, tự viết một truy vấn hybrid RRF theo mẫu ở mục 4.3.

**Nối mạch với giáo trình trước.** Bài này dạy tìm kiếm theo **từ khóa**; giáo trình `pgvector` dạy tìm kiếm theo **ngữ nghĩa**. Ghép hai bài lại chính là **hybrid search** — kiến trúc truy hồi nền tảng cho search hiện đại và cho RAG.

**Học tiếp gì sau bài này:**
- So sánh RRF với weighted fusion (hợp nhất theo trọng số) — khi nào dùng cái nào.
- **Reranking model** — mô hình chấm điểm lại top-N kết quả để tăng độ chính xác ở đỉnh danh sách.
- **BM25** và các extension mang nó vào Postgres (ví dụ ParadeDB / `pg_search`).
- Xây text search configuration và dictionary tùy biến cho **tiếng Việt**, kết hợp `unaccent`.
