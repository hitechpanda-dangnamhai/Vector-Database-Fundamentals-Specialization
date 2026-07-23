# RDBMS nào hỗ trợ Vector Search? PostgreSQL vs MySQL vs MariaDB
### Giáo trình SIÊU DỄ HIỂU — giải thích tận gốc mọi thuật ngữ | Basic → Staff

> **Bài giảng gốc:** *"RDBMS that support vector database searches"* — Course 3, IBM Vector Database Fundamentals.
>
> **Vị trí trong series:** Đây là mảnh ghép cuối — *"chọn nhà cho vector"*. Bốn mảnh trước lần lượt là: tạo vector (embedding) → làm cho việc tìm nhanh (indexing) → lưu và truy vấn (pgvector) → tìm theo từ khóa (full-text search). Bài này lùi lại một bước và hỏi câu hỏi chiến lược: **nên đặt vector ở đâu?**
>
> **Cách đọc:** Mọi thuật ngữ đều được giải thích ngay lần đầu xuất hiện. Bài này có nhắc lại vắn tắt các khái niệm từ giáo trình trước (vector, embedding, HNSW), nhưng vẫn giải thích lại từ đầu để bạn không phải lật ngược.
>
> **Ký hiệu:** 🧩 **[Ngoài bài gốc]** = phần bài gốc không có. ⚠️ **Chỗ khó** = chỗ nhiều người vấp. 💡 **Mẹo thực chiến** = kinh nghiệm production. ❗ **[Đính chính]** = chỗ bài gốc nói chưa đúng hoặc đã lỗi thời.

---

## ❗ Đọc phần này TRƯỚC KHI học: bài gốc có bốn chỗ đã lỗi thời

Bài giảng gốc được quay khi công nghệ còn đang thay đổi rất nhanh. Tính tới tháng 7/2026, có bốn chỗ bạn cần nhớ lại cho đúng — và chính những chỗ này hay thành **câu hỏi bẫy** trong phỏng vấn.

**Một — MariaDB Vector đã chính thức phát hành, không còn ở thì tương lai.** Bài gốc nói "sẽ lưu...", "sẽ cung cấp..." vì lúc đó tính năng còn ở dạng thử nghiệm. Thực tế: tính năng vector xuất hiện từ MariaDB 11.7 và **chính thức ổn định trong bản 11.8 LTS (2025)**, nằm ngay trong bản Community miễn phí.

**Hai — Bài gốc nói MariaDB "lưu dữ liệu chính và dữ liệu vector riêng biệt ở một database đặc biệt". Điều này SAI với bản chính thức.** Thực tế embedding được lưu trong **một cột `VECTOR` ngay trong bảng bình thường**, nằm cạnh dữ liệu nghiệp vụ, và tìm kiếm bằng SQL như mọi cột khác. Đây là **ưu điểm lớn nhất** của mô hình này, chứ không phải sự tách rời. Mục 3.4 sẽ giải thích vì sao.

**Ba — Đừng nhầm `tsvector` với `vector` trong PostgreSQL.** Bài gốc có một câu dễ gây hiểu lầm rằng PostgreSQL đã sẵn có kiểu vector qua `tsvector`. Không phải: `tsvector` là kiểu dữ liệu của **full-text search** (tìm theo từ khóa — nội dung giáo trình trước), còn kiểu để lưu embedding là `vector`, do **extension pgvector** cung cấp. Hai thứ hoàn toàn khác nhau.

**Bốn — Hệ sinh thái MySQL phân mảnh hơn bài gốc mô tả.** MySQL Community Server có **kiểu dữ liệu** `VECTOR`, nhưng khả năng **đánh index vector** — thứ quyết định tốc độ — chủ yếu đi qua các sản phẩm độc quyền của Oracle (HeatWave). Ngoài ra một số bản phân phối khác của MySQL đã tự làm index vector theo cách riêng: PlanetScale (Vitess) dùng thuật toán SPANN, Google Cloud SQL for MySQL dùng ScaNN. Bài gốc nói PlanetScale "sắp có" — nay đã có.

💡 Một lưu ý về phương pháp học: mảng này đổi nhanh tới mức **mọi con số phiên bản trong giáo trình này đều nên được kiểm chứng lại trước buổi phỏng vấn**. Thứ không đổi là **cách tư duy** — ba mô hình kiến trúc và khung quyết định ở Phần 4. Hãy học chắc phần đó.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### 0.1. Một câu tóm tắt siêu đơn giản

**Bài này dạy bạn cách quyết định nên đặt dữ liệu vector ở đâu** — cắm thêm vào cơ sở dữ liệu công ty đang dùng, hay dựng hẳn một hệ thống mới chỉ để chứa vector — và cho bạn biết ba "ông lớn" PostgreSQL, MySQL, MariaDB đang làm chuyện đó theo ba cách khác nhau như thế nào.

Nếu bạn chưa biết "vector" ở đây nghĩa là gì, hoặc vì sao người ta cần lưu nó, thì hoàn toàn bình thường. Mục 1.2 sẽ dựng lại khái niệm đó từ con số không.

### 0.2. Vấn đề thực tế mà bài này giải quyết

Bạn đang làm ở một công ty có ứng dụng chạy nhiều năm trên PostgreSQL hoặc MySQL. Trong đó có bảng người dùng, bảng đơn hàng, bảng sản phẩm — mọi thứ vận hành trơn tru.

Rồi sếp yêu cầu thêm tính năng "tìm sản phẩm tương tự theo hình ảnh" hoặc "hỏi đáp trên tài liệu nội bộ". Cả hai đều cần **tìm kiếm theo ngữ nghĩa**, tức là cần lưu và tìm vector.

Câu hỏi đặt ra ngay lập tức, và nó là câu hỏi **kiến trúc** chứ không phải câu hỏi cú pháp: *bạn cắm khả năng vector vào chính cơ sở dữ liệu đang có, hay bạn dựng thêm một hệ thống chuyên dụng riêng?*

Bài này cho bạn khung để trả lời câu hỏi đó, và cho bạn biết từng nền tảng mạnh yếu chỗ nào.

### 0.3. Học xong bạn sẽ làm được gì

- Giải thích rành mạch ba mô hình hỗ trợ vector: **extension** (pgvector), **native** (MariaDB Vector), **managed service** (MySQL HeatWave) — và vì sao sự khác biệt này quan trọng.
- Viết được code lưu và truy vấn vector trên cả ba nền tảng.
- Phân biệt "tự sinh embedding bên ngoài rồi nạp vào" với "để database tự sinh embedding" — và biết cái tiện lợi kia phải trả giá bằng gì.
- Dựng được khung quyết định: khi nào dùng cơ sở dữ liệu quan hệ, khi nào cần hệ chuyên dụng; khi nào thuê dịch vụ, khi nào tự vận hành.
- Trả lời được câu hỏi system design dạng "chọn nền tảng vector cho tình huống này".

### 0.4. Mạch kiến thức từ dễ tới khó

- 🟢 **Basic** — vì sao các cơ sở dữ liệu truyền thống lại thêm vector → nhắc lại vector là gì → ba mô hình kiến trúc → phép ví von → mô hình tư duy chung cho cả ba nền tảng.
- 🟡 **Intermediate** — đi từng nền tảng một với code chạy được thật → tự sinh embedding hay để database sinh → ba lỗi hay gặp.
- 🔴 **Advanced** — bên dưới nắp capo: extension khác native khác managed ở tầng nào → mỗi nền tảng dùng thuật toán index nào → giới hạn số chiều, SIMD, lượng tử hóa → vì sao "vector nằm cùng bảng" lại là ưu điểm lớn.
- 🟣 **Staff** — khung quyết định thật, chi phí ẩn của việc bị khóa vào một nhà cung cấp, ảnh hưởng tới tổ chức, một câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — thuật ngữ, ý cốt lõi, code cần thuộc, câu hỏi phỏng vấn kèm gợi ý trả lời.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần dài nhất và chậm nhất. Viết cho người chưa từng nghe tới vector database.

### 1.1. Bắt đầu từ vấn đề: hai con đường trước mặt bạn

Quay lại tình huống ở mục 0.2. Bạn cần thêm khả năng tìm kiếm theo ngữ nghĩa vào một ứng dụng đang chạy. Có đúng hai con đường.

**Con đường A — dựng thêm một cơ sở dữ liệu vector chuyên dụng.** Đó là những sản phẩm sinh ra chỉ để làm một việc duy nhất là lưu và tìm vector: **Pinecone**, **Weaviate**, **Qdrant**, **Milvus**. Chúng làm việc đó rất giỏi. Nhưng bạn có thêm một hệ thống mới phải học, phải vận hành, phải sao lưu, và — điểm đau nhất — phải **đồng bộ dữ liệu qua lại** với cơ sở dữ liệu cũ.

**Con đường B — thêm khả năng vector vào chính cơ sở dữ liệu đang chạy.** Vector nằm ngay cạnh dữ liệu nghiệp vụ, truy vấn bằng SQL quen thuộc, không thêm hệ thống nào.

Bài này nói về con đường B: **cả ba cơ sở dữ liệu quan hệ lớn giờ đều hỗ trợ vector, chỉ khác nhau ở *cách* hỗ trợ**. Nhưng Phần 4 sẽ quay lại con đường A một cách nghiêm túc, vì một staff engineer phải biết **cả hai** và biết khi nào chọn cái nào.

Nhắc lại nhanh cho ai chưa đọc giáo trình trước: **RDBMS** (viết tắt của *Relational Database Management System*, dịch: *hệ quản trị cơ sở dữ liệu quan hệ*) là loại phần mềm tổ chức dữ liệu thành các bảng có quan hệ với nhau — PostgreSQL, MySQL, MariaDB, Oracle đều thuộc loại này.

### 1.2. Nhắc lại từ đầu: vector và embedding là gì

Đây là khái niệm nền của cả khóa học, nên ta dựng lại cho chắc thay vì giả định bạn còn nhớ.

**Vector** trong ngữ cảnh này đơn giản là **một dãy số có thứ tự**. Ví dụ `[0.12, -0.85, 0.33]` là một vector ba phần tử.

**Số chiều (dimension)** là số lượng phần tử trong dãy đó. Vector vừa rồi có 3 chiều. Trong thực tế, các mô hình AI thường tạo ra vector 384, 768, 1536, hoặc 3072 chiều.

**Embedding** (nghĩa đen: *sự nhúng vào*) là **kết quả của việc biến một nội dung phức tạp thành một vector** sao cho **những nội dung có ý nghĩa gần nhau sẽ cho ra hai vector nằm gần nhau về mặt toán học**.

Câu định nghĩa trên là câu quan trọng nhất trong toàn bộ bài, nên ta mổ nó ra bằng ví dụ cụ thể.

Giả sử một mô hình AI biến ba câu sau thành ba vector:
- "áo khoác mùa đông" → `[0.9, 0.1, 0.2]`
- "áo phao giữ ấm" → `[0.88, 0.12, 0.25]`
- "máy xay sinh tố" → `[0.1, 0.95, 0.7]`

Hãy để ý: hai vector đầu **rất giống nhau về số liệu**, dù hai câu chữ chẳng có từ nào chung. Vector thứ ba khác hẳn. Mô hình đã "nhúng" ý nghĩa vào những con số — và đó là lý do máy tính bỗng có thể so sánh ý nghĩa bằng phép toán.

Cứ hình dung như thế này. Trên một tấm bản đồ, mỗi địa điểm được đại diện bằng một cặp số (kinh độ, vĩ độ). Hai quán cà phê ở cùng một con phố sẽ có cặp số gần nhau; một quán ở Hà Nội và một quán ở Sài Gòn thì cặp số xa nhau. Bạn không cần biết tên quán, chỉ cần nhìn tọa độ là biết chúng có gần nhau không.

Embedding làm y hệt, chỉ khác hai điểm. Thứ nhất, thay vì 2 chiều (kinh độ, vĩ độ) thì nó dùng hàng trăm tới hàng nghìn chiều. Thứ hai, "gần nhau" ở đây không phải gần về địa lý mà là **gần về ý nghĩa**. Phép ví von này khớp ở đúng chỗ cốt lõi: **cả hai đều biến một thứ khó so sánh thành một bộ tọa độ dễ so sánh**.

**Model** (dịch: *mô hình*) là chương trình AI đã được huấn luyện, chịu trách nhiệm biến nội dung thành vector. Ví dụ các mô hình phổ biến: `text-embedding-3-small` của OpenAI (1536 chiều), `BGE-M3`, `all-MiniLM-L6-v2` (384 chiều).

⚠️ **Chỗ khó — nhớ ngay từ bây giờ:** vector do hai mô hình khác nhau tạo ra thì **không so sánh được với nhau**, kể cả khi tình cờ cùng số chiều. Mỗi mô hình tự dựng "bản đồ" riêng của nó, và hai bản đồ này không dùng chung hệ tọa độ. Trộn lẫn chúng cho ra kết quả sai hoàn toàn — mà lại **không báo lỗi gì cả**. Điều này sẽ trở lại ở mục 2.5 và 4.3 với hệ quả rất tốn kém.

### 1.3. Tìm kiếm theo độ tương đồng và các cách đo khoảng cách

**Vector similarity search** (dịch: *tìm kiếm theo độ tương đồng vector*) là việc: cho một vector truy vấn, hãy tìm trong kho những vector **gần nó nhất**.

Nhưng "gần" đo bằng gì? Có vài cách, gọi chung là **distance metric** (*thước đo khoảng cách*):

- **Cosine distance** (*khoảng cách cosin*) — đo **góc** giữa hai vector, bỏ qua độ dài của chúng. Đây là lựa chọn mặc định cho văn bản, vì với văn bản ta quan tâm "hai nội dung có cùng hướng ý nghĩa không" hơn là "nội dung nào dài hơn".
- **Euclidean distance**, còn viết là **L2** (*khoảng cách Euclid*) — khoảng cách theo đường thẳng, đúng như cách bạn đo khoảng cách giữa hai điểm trên bản đồ bằng thước kẻ.
- **Inner product / dot product** (*tích vô hướng*) — một phép nhân nhanh, hay dùng khi các vector đã được chuẩn hóa sẵn.
- **Manhattan distance** (*khoảng cách Manhattan*, còn gọi L1) — khoảng cách đi theo các đoạn vuông góc, như taxi chạy trong lưới phố ô bàn cờ ở Manhattan. Đây chính là lý do có cái tên đó.
- **Jaccard distance** — đo độ chồng lấn giữa hai tập hợp, dùng cho dữ liệu dạng nhị phân.

Với người mới, chỉ cần nhớ chắc hai cái đầu: **cosine cho văn bản, Euclid cho dữ liệu hình học/số học**. Ba cái sau chỉ cần biết mặt.

⚠️ **Chỗ khó — cái bẫy khiến kết quả bị đảo ngược.** Có nền tảng trả về **distance** (*khoảng cách*: số **càng nhỏ càng giống**), có nền tảng trả về **similarity** (*độ tương đồng*: số **càng lớn càng giống**). Nếu bạn nhầm hai thứ này khi viết `ORDER BY`, hệ thống sẽ trả về những kết quả **ít liên quan nhất** — và trớ trêu là nó vẫn chạy, vẫn trả về đủ 10 kết quả, không báo lỗi gì. Luôn đọc tài liệu để biết hàm bạn đang dùng trả về loại nào.

### 1.4. Analogy: cơi nới thêm phòng hay mua căn nhà thứ hai

Đây là phép ví von sẽ theo bạn suốt bài, kể cả trong phòng phỏng vấn. Ta dựng nó cho thật đầy đủ.

Gia đình bạn đang ở một căn nhà. Giờ có thêm nhu cầu mới — cần một phòng làm việc. Hai lựa chọn.

**Mua căn nhà thứ hai chỉ để làm phòng việc.** Căn nhà mới đó được thiết kế tối ưu cho công việc: cách âm tốt, bàn ghế chuyên dụng, ánh sáng hoàn hảo. Nhưng bạn phải **đi lại giữa hai nhà** mỗi khi cần thứ gì để quên, phải **trả hai tiền điện nước**, phải **trông coi hai nơi**, và mỗi khi mua đồ mới phải nhớ nó đang nằm ở nhà nào.

**Cơi nới thêm một phòng vào căn nhà sẵn có.** Phòng này có thể không tối ưu bằng, nhưng **mọi thứ ở dưới một mái**: bạn đi từ phòng khách sang phòng làm việc trong ba bước chân, chỉ một hóa đơn điện, chỉ một cái nhà để trông.

Ánh xạ sang kỹ thuật cho rõ ràng:

| Trong phép ví von | Trong thực tế |
|---|---|
| Căn nhà sẵn có | Cơ sở dữ liệu quan hệ đang chạy (PostgreSQL/MySQL/MariaDB) |
| Căn nhà thứ hai | Cơ sở dữ liệu vector chuyên dụng (Pinecone, Weaviate, Qdrant, Milvus) |
| Đi lại giữa hai nhà | Đồng bộ dữ liệu giữa hai hệ thống — nguồn lỗi kinh điển |
| Hai tiền điện, trông hai nhà | Hai khoản chi phí hạ tầng, hai hệ thống phải trực sự cố |
| Đi ba bước chân trong nhà | Một câu SQL duy nhất, trong một giao dịch duy nhất |

Và đây là chỗ phép ví von khớp chính xác nhất, cũng là luận điểm cốt lõi của cả bài: **cái giá của căn nhà thứ hai không nằm ở tiền mua nhà, mà nằm ở việc phải sống ở hai nơi cùng lúc.** Trong kỹ thuật cũng vậy — chi phí thật của một hệ thống mới không phải là tiền máy chủ, mà là công sức đồng bộ, vận hành, và những lỗi phát sinh từ chỗ hai hệ thống lệch nhau.

### 1.5. Ba cách "cơi nới" — ba mô hình kiến trúc

Đã quyết định cơi nới, vẫn còn ba cách làm khác nhau. Đây là **ý chính của toàn bài**, hãy đọc thật chậm.

**Mô hình 1 — Extension (phần mở rộng).** Ví dụ: **PostgreSQL + pgvector**.

**Extension** là một gói tính năng do bên thứ ba viết, cài thêm vào cơ sở dữ liệu bằng một câu lệnh. Sau khi cài, database có thêm kiểu dữ liệu mới, hàm mới, cách đánh index mới.

Cứ hình dung như cài thêm một ứng dụng vào điện thoại: phần cứng không đổi, nhưng máy làm được việc mới. Bạn phải chủ động cài, và phải để ý phiên bản của ứng dụng đó.

**Mô hình 2 — Native (hỗ trợ sẵn trong lõi).** Ví dụ: **MariaDB Vector**.

**Native support** nghĩa là tính năng được xây **thẳng vào lõi** của cơ sở dữ liệu. Không phải cài gì thêm, không có gói riêng để quản lý phiên bản. Cài database xong là kiểu `VECTOR` đã có sẵn.

Hình dung như một tính năng được nhà sản xuất tích hợp sẵn vào điện thoại — camera chẳng hạn. Bạn không cài, nó có sẵn; đổi lại bạn chỉ được nâng cấp nó khi hãng ra bản mới.

**Mô hình 3 — Managed service (dịch vụ được vận hành sẵn).** Ví dụ: **MySQL HeatWave**.

**Managed database service** là cơ sở dữ liệu do **nhà cung cấp đám mây chạy và vận hành hộ bạn**. Bạn không quản máy chủ, không vá lỗi bảo mật, không lo sao lưu — nhà cung cấp làm hết, và bạn trả tiền theo mức sử dụng.

Hình dung như thuê một căn hộ dịch vụ: có người dọn dẹp, sửa chữa, bảo trì. Rất tiện. Nhưng bạn không sở hữu nó, không được đục tường theo ý mình, và nếu muốn chuyển đi thì phải dọn hết đồ đạc sang nơi khác.

| Mô hình | Ví dụ | Nghĩa là | Bạn phải làm gì |
|---|---|---|---|
| **Extension** | PostgreSQL + **pgvector** | Cài thêm gói vào server sẵn có → có kiểu `vector` | Chạy `CREATE EXTENSION`, tự quản phiên bản |
| **Native** | **MariaDB Vector** | Kiểu `VECTOR` xây thẳng trong lõi | Không làm gì, dùng luôn |
| **Managed** | **MySQL HeatWave** | Dịch vụ đám mây làm sẵn cả vector *lẫn* việc sinh embedding | Đăng ký dịch vụ, trả tiền theo dùng |

### 1.6. Bộ thuật ngữ còn lại — giải thích trước khi dùng

Những từ này sẽ xuất hiện liên tục từ Phần 2 trở đi, nên ta lấp hố ngay bây giờ.

**Open source** (dịch: *mã nguồn mở*) nghĩa là mã nguồn của phần mềm được công khai, ai cũng xem được, sửa được, và thường là dùng miễn phí. **Proprietary** (dịch: *độc quyền / sở hữu riêng*) là ngược lại: mã nguồn đóng, thuộc về một công ty, muốn dùng phải trả tiền và phải theo điều khoản của họ.

**GA** (viết tắt của *General Availability*, dịch: *phát hành chính thức*) là mốc mà nhà sản xuất tuyên bố tính năng đã đủ ổn định để dùng trong hệ thống thật. Trước GA thường có các giai đoạn *preview* (xem trước) và *beta* (thử nghiệm) — dùng được nhưng có thể còn lỗi và cú pháp có thể thay đổi.

**LTS** (viết tắt của *Long Term Support*, dịch: *hỗ trợ dài hạn*) là loại phiên bản được cam kết vá lỗi và vá bảo mật trong nhiều năm. Với hệ thống production, gần như luôn nên chọn bản LTS — đó là lý do mốc "MariaDB 11.8 LTS" quan trọng hơn nhiều so với "MariaDB 11.7".

**Fork** (dịch: *nhánh rẽ*) là việc lấy mã nguồn của một dự án rồi tách ra phát triển thành dự án riêng. MariaDB chính là một fork của MySQL, do chính những người tạo ra MySQL lập nên sau khi MySQL bị Oracle mua lại. Biết chi tiết này giúp bạn hiểu vì sao cú pháp hai bên giống nhau tới 90% mà triết lý lại khác hẳn.

**OLTP** (viết tắt của *Online Transaction Processing*, dịch: *xử lý giao dịch trực tuyến*) là loại công việc gồm rất nhiều thao tác nhỏ, nhanh, xảy ra theo thời gian thực: đặt hàng, thanh toán, cập nhật hồ sơ. Đây là thế mạnh truyền thống của cơ sở dữ liệu quan hệ. Đối lập với nó là **OLAP** (*Online Analytical Processing*) — các truy vấn phân tích lớn, chạy lâu, quét nhiều dữ liệu để ra báo cáo.

**BYO-embedding** (viết tắt của *Bring Your Own embedding*, dịch: *tự mang embedding tới*) nghĩa là **bạn** tự chạy mô hình AI ở bên ngoài để tạo vector, rồi nạp vector đó vào database. Database chỉ lưu và tìm, không sinh. Đây là cách của pgvector và MariaDB.

**In-database embedding** (dịch: *sinh embedding ngay trong database*) là ngược lại: bạn đưa văn bản thô vào, **database tự gọi mô hình AI tích hợp sẵn** để biến nó thành vector. Đây là điểm khác biệt lớn nhất của MySQL HeatWave.

**LLM** (viết tắt của *Large Language Model*, dịch: *mô hình ngôn ngữ lớn*) là loại mô hình AI được huấn luyện trên khối lượng văn bản khổng lồ — ChatGPT, Claude, Llama đều thuộc loại này. Trong ngữ cảnh bài này, LLM là thứ tạo ra embedding hoặc sinh câu trả lời.

**Vendor lock-in** (dịch: *bị khóa vào một nhà cung cấp*) là tình trạng bạn phụ thuộc vào một nhà cung cấp tới mức việc chuyển đi trở nên cực kỳ tốn kém — về tiền, về thời gian, hoặc cả hai. Đây là **rủi ro quan trọng nhất** cần cân nhắc khi chọn mô hình managed, và mục 4.3 dành riêng để nói về nó.

**Hybrid query** (dịch: *truy vấn lai*) là một câu truy vấn duy nhất vừa tìm theo vector, vừa lọc theo điều kiện thông thường. Ví dụ: "tìm sản phẩm giống ảnh này nhất, nhưng chỉ trong số hàng còn tồn kho và giá dưới 50 đô". Đây là **siêu năng lực** của mô hình đặt vector trong cơ sở dữ liệu quan hệ, và mục 3.4 sẽ chỉ ra vì sao.

### 1.7. Mô hình tư duy chung: cả ba nền tảng làm cùng một việc

Đây là điều làm bài này dễ hơn bạn tưởng. Dù là extension, native hay managed, **luồng làm việc luôn giống hệt nhau bốn bước**:

```
Bước 1 — Tạo một cột kiểu vector, khai báo số chiều
         (số chiều PHẢI bằng đúng số chiều mà mô hình của bạn sinh ra)
              → vector(1536)   trên PostgreSQL
              → VECTOR(1536)   trên MariaDB / MySQL

Bước 2 — Sinh embedding bằng mô hình AI, rồi INSERT vào cột đó
         (trừ HeatWave, nơi database có thể tự sinh giúp bạn)

Bước 3 — Tạo index (thường là HNSW) để việc tìm kiếm nhanh lên

Bước 4 — Truy vấn: sắp xếp theo khoảng cách rồi lấy k kết quả đầu
              ORDER BY <hàm_khoảng_cách>(cột, vector_truy_vấn)
              LIMIT k
```

Học kỹ một nền tảng thì hai nền tảng còn lại chỉ khác **cú pháp**, không khác **ý tưởng**. Đây cũng chính là điều bài giảng gốc nói đúng: các RDBMS hiện nay đều cung cấp một lớp bọc để bạn có kiểu vector và tìm theo độ tương đồng, thay vì phải tự tính khoảng cách bằng tay trong code ứng dụng.

Còn một từ trong sơ đồ cần giải thích. **`k`** trong "k kết quả gần nhất" chỉ đơn giản là số lượng kết quả bạn muốn lấy về. Bài toán này có tên riêng: **k-NN** (viết tắt của *k-Nearest Neighbors*, dịch: *k láng giềng gần nhất*).

### 1.8. Ví dụ chạy tay — tự tính "gần nhau" bằng số thật

Lý thuyết đủ rồi. Hãy tự tay tính một lần để thấy chuyện gì thực sự xảy ra bên trong.

Giả sử ta có kho ba sản phẩm, mỗi sản phẩm đã được biến thành vector 3 chiều (thực tế là hàng nghìn chiều, nhưng 3 chiều thì ta tính tay được):

| Sản phẩm | Vector |
|---|---|
| A. "áo khoác mùa đông" | `[0.9, 0.1, 0.2]` |
| B. "áo phao giữ ấm" | `[0.8, 0.2, 0.3]` |
| C. "máy xay sinh tố" | `[0.1, 0.9, 0.7]` |

Người dùng tìm: **"đồ mặc cho trời lạnh"** → mô hình biến thành vector truy vấn `Q = [0.85, 0.15, 0.25]`.

Ta dùng **khoảng cách Euclid** cho dễ tính tay. Công thức: lấy hiệu từng chiều, bình phương, cộng lại, rồi lấy căn.

**Khoảng cách từ Q tới A:**
```
Hiệu từng chiều:  (0.85-0.9) = -0.05 ;  (0.15-0.1) = 0.05 ;  (0.25-0.2) = 0.05
Bình phương:       0.0025    ;   0.0025   ;   0.0025
Cộng lại:          0.0075
Căn bậc hai:       ≈ 0.087        ← rất gần
```

**Khoảng cách từ Q tới B:**
```
Hiệu:        (0.85-0.8) = 0.05 ;  (0.15-0.2) = -0.05 ;  (0.25-0.3) = -0.05
Bình phương:  0.0025   ;  0.0025  ;  0.0025
Cộng:         0.0075
Căn:          ≈ 0.087        ← cũng rất gần
```

**Khoảng cách từ Q tới C:**
```
Hiệu:        (0.85-0.1) = 0.75 ;  (0.15-0.9) = -0.75 ;  (0.25-0.7) = -0.45
Bình phương:  0.5625   ;  0.5625  ;  0.2025
Cộng:         1.3275
Căn:          ≈ 1.152        ← xa hơn khoảng mười ba lần
```

Kết quả xếp hạng: **A và B ngang nhau ở vị trí đầu (≈0.087), C xếp cuối (≈1.152)**.

Hãy dừng lại và nhìn kỹ điều vừa xảy ra, vì nó là toàn bộ phép màu của vector search: **câu truy vấn "đồ mặc cho trời lạnh" không chứa một từ nào chung với "áo phao giữ ấm"** — không có chữ "áo", không có chữ "lạnh" trong câu kia. Tìm kiếm theo từ khóa sẽ trả về con số không tròn trĩnh. Nhưng vì mô hình đã đặt hai nội dung này gần nhau trên "bản đồ ý nghĩa", phép trừ và phép cộng đơn giản ở trên tìm ra chúng ngay.

Và bây giờ là điểm quan trọng của cả bài học: **thao tác vừa rồi hoàn toàn giống nhau trên PostgreSQL, MySQL, MariaDB, hay Pinecone.** Ba nền tảng chỉ khác nhau ở chỗ *bạn gõ câu lệnh như thế nào* và *ai lo phần vận hành*. Toán học bên dưới là một.

### 1.9. Code "hello world" trên cả ba nền tảng

Giờ ta viết đúng bốn bước của mục 1.7 bằng cú pháp thật của từng bên. Hãy chú ý xem chúng giống nhau tới mức nào.

**(a) PostgreSQL + pgvector — mô hình extension:**

```sql
-- BƯỚC 0: Cài extension. Chỉ cần chạy MỘT LẦN cho mỗi database.
-- IF NOT EXISTS nghĩa là "nếu đã có rồi thì bỏ qua, đừng báo lỗi".
CREATE EXTENSION IF NOT EXISTS vector;

-- BƯỚC 1: Tạo bảng có cột vector
CREATE TABLE items (
    id        bigserial PRIMARY KEY,   -- bigserial: số tự tăng, làm khóa chính
    content   text,                    -- nội dung gốc dạng chữ, để hiển thị lại cho người dùng
    embedding vector(512)              -- cột vector 512 chiều
                                       -- số 512 PHẢI khớp số chiều của mô hình bạn dùng
);

-- BƯỚC 2: Chèn dữ liệu. Vector do ứng dụng của bạn sinh sẵn bên ngoài (BYO).
INSERT INTO items (content, embedding)
VALUES ('áo khoác mùa đông', '[0.9, 0.1, ...]');   -- pgvector nhận vector dạng chuỗi

-- BƯỚC 3: Tạo index HNSW để tìm nhanh
CREATE INDEX ON items
USING hnsw (embedding vector_cosine_ops);
--                    ^^^^^^^^^^^^^^^^^ "ops class": báo cho index biết sẽ đo bằng cosine.
--    QUY TẮC: ops class phải khớp với toán tử dùng lúc query, nếu không index bị bỏ qua.

-- BƯỚC 4: Tìm 5 sản phẩm gần nhất
SELECT id, content
FROM items
ORDER BY embedding <=> '[...512 số...]'   -- <=> là toán tử khoảng cách cosine của pgvector
LIMIT 5;                                   -- càng NHỎ càng gần, nên sắp tăng dần (mặc định)
```

**(b) MariaDB Vector — mô hình native:**

```sql
-- KHÔNG có bước 0. Không cần CREATE EXTENSION, không cần cài plugin.
-- Đây chính là ý nghĩa của chữ "native".

CREATE TABLE chunks (
    id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    content   TEXT NOT NULL,
    embedding VECTOR(1536) NOT NULL,            -- kiểu VECTOR có sẵn trong lõi
    VECTOR INDEX (embedding) DISTANCE=cosine    -- khai index NGAY trong lệnh tạo bảng,
                                                -- và chọn luôn thước đo khoảng cách
) ENGINE=InnoDB;                                -- InnoDB là storage engine mặc định

-- Chèn: VEC_FromText biến chuỗi text thành kiểu vector
INSERT INTO chunks (content, embedding)
VALUES ('MariaDB lưu embedding ngay trong bảng',
        VEC_FromText('[...1536 số...]'));

-- Truy vấn
SELECT id, content
FROM chunks
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 3;    -- ⚠️ LIMIT là BẮT BUỘC nếu muốn index được dùng (xem mục 2.5)
```

**(c) MySQL — mô hình managed (ví dụ bám theo bài gốc):**

```sql
CREATE TABLE t (
    id  INT PRIMARY KEY,
    val VECTOR(5)                    -- vector 5 phần tử, đúng ví dụ trong bài gốc
);

-- STRING_TO_VECTOR: hàm của MySQL để parse chuỗi thành vector
INSERT INTO t (id, val)
VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));

-- Đo khoảng cách. Lưu ý: khả năng đánh INDEX cho vector
-- phụ thuộc sản phẩm/phiên bản bạn dùng — xem mục 2.2.
SELECT id
FROM t
ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE')
LIMIT 5;
```

Nhìn ba đoạn code trên cạnh nhau, bạn thấy ngay: **cùng một ý tưởng, ba bộ cú pháp**. `<=>` của pgvector, `VEC_DISTANCE_COSINE()` của MariaDB, `DISTANCE()` của MySQL — ba cách viết cho cùng một phép toán mà bạn đã tự tính tay ở mục 1.8.

### 1.10. 🧩 [Ngoài bài gốc] — Hai điều nên biết ngay từ Basic

**Một: "hỗ trợ vector" là một câu nói rất mơ hồ.** Khi một nhà cung cấp tuyên bố "database của chúng tôi hỗ trợ vector", hãy luôn hỏi lại ba câu: (a) có **kiểu dữ liệu** vector không? (b) có **hàm tính khoảng cách** không? (c) có **index ANN** không? Ba thứ này khác nhau, và **thứ ba mới là thứ quyết định tốc độ**. Một database có kiểu vector nhưng không có index sẽ phải so sánh lần lượt với **từng vector một** trong bảng — chạy được với 10 nghìn dòng, chết đứng với 10 triệu dòng. Sự phân biệt này chính là chỗ MySQL và MariaDB khác nhau nhiều nhất (mục 2.2, 2.3).

**Hai: chọn nền tảng không phải quyết định thuần kỹ thuật.** Bạn sẽ thấy ở Phần 4 rằng những yếu tố quyết định thật thường nằm ngoài kỹ thuật: dữ liệu có được phép rời khỏi công ty không, đội của bạn có mấy người, công ty chấp nhận bị khóa vào một nhà cung cấp tới mức nào. Một kỹ sư giỏi chọn theo hiệu năng; một staff engineer chọn theo **ràng buộc thật của tổ chức**.

### ✅ Self-check Phần 1

**1. Ba mô hình hỗ trợ vector của RDBMS là gì? Cho một ví dụ mỗi loại và nói rõ bạn phải làm gì ở mỗi mô hình.**
> *Gợi ý đáp án:* **Extension** (PostgreSQL + pgvector — phải chạy `CREATE EXTENSION` và tự quản phiên bản extension); **Native** (MariaDB Vector — không cài gì, dùng ngay, nhưng phụ thuộc nhịp phát hành của database); **Managed** (MySQL HeatWave — đăng ký dịch vụ, nhà cung cấp vận hành hộ, trả tiền theo dùng).

**2. Ưu điểm lớn nhất của "cơi nới RDBMS" so với "mua vector DB riêng" là gì?**
> *Gợi ý đáp án:* Vector nằm cùng chỗ với dữ liệu nghiệp vụ nên chạy được hybrid query trong **một câu SQL, một giao dịch** — không phải đồng bộ hai hệ thống, không phải vận hành và trực sự cố cho hệ thứ hai. Nói ngắn gọn: thắng ở **sự đơn giản trong vận hành**.

**3. "In-database embedding" khác "bring-your-own embedding" ở chỗ nào?**
> *Gợi ý đáp án:* BYO nghĩa là bạn tự chạy mô hình bên ngoài để sinh vector rồi nạp vào (pgvector, MariaDB) — bạn kiểm soát mô hình. In-database nghĩa là database tự sinh vector bằng LLM tích hợp (HeatWave) — tiện hơn nhưng bị buộc vào mô hình của nhà cung cấp đó.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Giả định bạn đã nắm: vector/embedding, distance metric, ba mô hình extension/native/managed, và luồng bốn bước ở mục 1.7.

### 2.1. PostgreSQL + pgvector — mô hình extension

PostgreSQL là cơ sở dữ liệu quan hệ mã nguồn mở được dùng bởi Uber, Netflix, Instagram, Spotify (đúng như bài gốc nêu). Nó hỗ trợ vector thông qua **extension pgvector** — cài lên là có kiểu `vector`.

❗ **[Đính chính bài gốc — đây là câu hỏi bẫy hay gặp nhất]** Bài gốc có câu dễ gây hiểu lầm rằng "PostgreSQL cung cấp `tsvector` như một kiểu dữ liệu vector". Cần cực kỳ rõ ràng chỗ này:

| | `tsvector` | `vector` |
|---|---|---|
| Thuộc về | PostgreSQL lõi (có sẵn) | Extension **pgvector** (phải cài) |
| Lưu cái gì | Danh sách **lexeme** — các từ đã chuẩn hóa | Dãy số thực — **embedding** |
| Dùng cho | **Full-text search** — tìm theo từ khóa | **Semantic search** — tìm theo ngữ nghĩa |
| Ví dụ nội dung | `'brown':3 'fox':4 'run':6` | `[0.12, -0.85, 0.33, ...]` |
| Index | GIN | HNSW / IVFFlat |

Hai thứ này **không liên quan gì tới nhau** ngoài việc trùng ba chữ cái. Nếu trong phỏng vấn ai hỏi "Postgres đã có sẵn vector search qua `tsvector` đúng không?", câu trả lời là **không**, và giải thích như bảng trên sẽ ghi điểm rất mạnh.

**Đặc điểm của pgvector cần nhớ:**

- **BYO-embedding** — pgvector chỉ lưu và tìm vector, **không tự sinh**. Bạn phải gọi mô hình AI ở tầng ứng dụng.
- **Nhiều lựa chọn index:** HNSW (mặc định nên dùng), IVFFlat (xây nhanh hơn, tốn ít RAM hơn), và DiskANN qua extension bổ trợ `pgvectorscale`.
- **Hybrid query rất mạnh:** vì vector nằm trong Postgres, bạn `JOIN`, `WHERE`, `GROUP BY` thoải mái cùng dữ liệu nghiệp vụ.
- **Hệ sinh thái managed rộng nhất:** Supabase, Neon, AWS RDS/Aurora, Google Cloud SQL đều hỗ trợ sẵn pgvector. Đây là lợi thế thực tế rất lớn — bạn được cả tính mở của mã nguồn mở lẫn sự tiện lợi của dịch vụ vận hành sẵn.

💡 **Mẹo thực chiến về phiên bản:** pgvector hiện ở dòng **0.8.x** (giữa 2026). Nhưng có một chi tiết dễ mắc bẫy: **phiên bản pgvector bạn thực sự chạy thường do nhà cung cấp managed quyết định, không phải do bạn**. Supabase, Neon, RDS mỗi bên cập nhật theo nhịp riêng. Nên trước khi lên kế hoạch dùng một tính năng mới, hãy chạy `SELECT extversion FROM pg_extension WHERE extname = 'vector';` để biết chính xác mình đang có gì.

### 2.2. MySQL và MySQL HeatWave — mô hình managed service

Bài gốc tập trung vào **MySQL HeatWave** — dịch vụ được vận hành trọn gói của Oracle, gộp ba loại công việc vào một dịch vụ: xử lý giao dịch (OLTP), phân tích (OLAP), và học máy.

**Điểm khác biệt lớn nhất của HeatWave — nó tự sinh embedding cho bạn.**

Tính năng này có tên **HeatWave GenAI**: bạn đưa văn bản thô vào, dịch vụ tự gọi LLM tích hợp sẵn để tạo vector theo thời gian thực. Bạn **không cần** gọi API của OpenAI, không cần tự chạy mô hình, không cần viết code sinh embedding.

Hãy dừng lại một chút để thấy đây là khác biệt lớn tới mức nào. Với pgvector và MariaDB, kiến trúc của bạn luôn có hình dạng thế này:

```
Ứng dụng → gọi mô hình AI để sinh vector → nạp vector vào database
```

Với HeatWave, nó rút gọn thành:

```
Ứng dụng → đưa văn bản vào database → database tự lo phần còn lại
```

Ít mảnh ghép hơn, ít code hơn, ít thứ hỏng hơn. Rất hấp dẫn. Nhưng mục 4.3 sẽ chỉ ra cái giá phải trả — và cái giá đó không nhỏ.

**Các đặc điểm khác:** HeatWave chạy trên OCI (Oracle Cloud Infrastructure) và AWS; có bản dùng thử giới hạn. Vì cần nhiều tài nguyên để đạt khả năng mở rộng và thông lượng cao, nó được thiết kế cho môi trường đám mây — **không tự cài trên máy chủ của bạn được**.

⚠️ **Chỗ khó — và đây là điểm quan trọng nhất về MySQL.** Cần phân biệt rạch ròi ba tầng:

| Tầng | Có trong MySQL Community (bản miễn phí) không? |
|---|---|
| **Kiểu dữ liệu `VECTOR`** | Có (từ MySQL 9.x) |
| **Hàm tính khoảng cách** | Tùy phiên bản — nhiều nguồn đầu 2026 ghi nhận hàm này chưa có trong Community |
| **Index ANN cho vector** | **Không** — đi qua sản phẩm độc quyền của Oracle (HeatWave) |

Nhớ lại điều đã nói ở mục 1.10: **index mới là thứ quyết định tốc độ**. Có kiểu dữ liệu mà không có index nghĩa là mỗi truy vấn phải so sánh với từng dòng một. Vì vậy, câu nói chính xác về MySQL là: *"MySQL có kiểu vector trong bản mở, nhưng khả năng tìm kiếm vector nhanh thì chủ yếu nằm trong sản phẩm thương mại của Oracle."*

Đây chính là **điểm khác biệt then chốt so với MariaDB**, nơi mọi thứ — kể cả index — đều nằm trong bản Community miễn phí.

🧩 **[Ngoài bài gốc] Hệ sinh thái MySQL phân mảnh.** "MySQL" không còn là một thứ duy nhất. Các bản phân phối khác nhau đã tự làm index vector theo cách riêng:

- **PlanetScale (dựa trên Vitess)** — có index vector từ 2025, dùng thuật toán **SPANN**.
- **Google Cloud SQL for MySQL** — dùng thuật toán **ScaNN** của Google.
- **TiDB, AliSQL (Alibaba)** — có hỗ trợ vector ở các mức độ khác nhau.
- **Amazon RDS for MySQL** — chưa có index vector. Nhưng **RDS for MariaDB 11.8 thì có**.

Ý nghĩa thực tế: khi ai đó nói "chúng tôi dùng MySQL", câu hỏi tiếp theo bắt buộc phải là **"bản phân phối nào, chạy ở đâu?"** — vì khả năng vector khác nhau hoàn toàn giữa các bản.

### 2.3. MariaDB Vector — mô hình native

MariaDB Server là một **fork** của MySQL, do chính những người tạo ra MySQL lập nên. **MariaDB Vector** đưa khả năng tìm kiếm theo độ tương đồng **thẳng vào lõi** — không extension, không plugin, không dịch vụ riêng.

❗ **[Đính chính quan trọng]** Bài gốc mô tả MariaDB "lưu dữ liệu chính và dữ liệu vector riêng biệt ở một database đặc biệt". **Bản chính thức hoàn toàn không như vậy.** Embedding nằm trong cột `VECTOR(n)` **ngay trong bảng bình thường**, cạnh dữ liệu nghiệp vụ, tìm kiếm bằng SQL như mọi cột khác — đúng triết lý "một database cho tất cả". Mô tả cũ có lẽ ứng với bản thiết kế sơ khai; nay đã khác hẳn. Và như mục 3.4 sẽ chỉ ra, việc **nằm cùng bảng chính là ưu điểm lớn nhất**, không phải nhược điểm.

**Các đặc điểm cập nhật tới 2026:**

- **Chính thức ổn định từ bản 11.8 LTS (2025)**; kiểu `VECTOR` xuất hiện từ 11.7.
- **Nằm trong Community Server — miễn phí và mã nguồn mở.** Đây là khác biệt lớn nhất so với MySQL, nơi index vector nằm sau sản phẩm thương mại.
- Dùng thuật toán index **HNSW**.
- Hỗ trợ hai thước đo: **cosine** và **Euclid (L2)**.
- Hỗ trợ số chiều tới **16.383** — con số này cao hơn nhiều so với giới hạn index của pgvector, và mục 3.3 sẽ giải thích vì sao điều đó quan trọng.
- Có sẵn trên **Amazon RDS for MariaDB 11.8** — nghĩa là bạn dùng được dạng dịch vụ vận hành sẵn mà vẫn giữ mã nguồn mở.
- Được tối ưu bằng **SIMD** (giải thích ở mục 3.3) cho các dòng CPU AVX2, AVX512, ARM, Power10.
- **Không tự sinh embedding** — theo mô hình BYO giống pgvector.
- Có tích hợp chính thức với LangChain, LlamaIndex, Spring AI (các thư viện phổ biến để xây ứng dụng AI).

```sql
-- MariaDB: toàn bộ vòng đời trong một chỗ, không cài gì thêm

CREATE TABLE products (
    id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name      VARCHAR(255),
    price     DECIMAL(10,2),                        -- dữ liệu nghiệp vụ...
    in_stock  BOOLEAN,                              -- ...nằm ngay cạnh...
    embedding VECTOR(1536) NOT NULL,                -- ...vector. Cùng một bảng.
    VECTOR INDEX (embedding) DISTANCE=cosine
) ENGINE=InnoDB;

-- Truy vấn lai: vừa lọc nghiệp vụ, vừa tìm theo ngữ nghĩa — MỘT câu, MỘT giao dịch
SELECT id, name, price
FROM products
WHERE in_stock = 1 AND price < 50                   -- lọc theo dữ liệu nghiệp vụ
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))  -- xếp theo độ giống
LIMIT 10;
```

Hãy nhìn kỹ câu truy vấn cuối. Nó trả lời câu hỏi *"tìm cho tôi 10 sản phẩm giống ảnh này nhất, nhưng chỉ trong số còn hàng và giá dưới 50 đô"* bằng **một câu lệnh duy nhất**. Mục 3.4 sẽ cho bạn thấy nếu tách vector sang hệ thống riêng thì câu hỏi tưởng như đơn giản này trở nên phức tạp tới mức nào.

### 2.4. Bảng so sánh ba nền tảng

| | **PostgreSQL + pgvector** | **MySQL HeatWave** | **MariaDB Vector** |
|---|---|---|---|
| Mô hình | Extension | Managed service | **Native** (trong lõi) |
| Phải cài thêm gì? | `CREATE EXTENSION vector` | Không (dịch vụ lo) | **Không** (có sẵn) |
| Mã nguồn mở & miễn phí? | Có | Không (độc quyền, chỉ trên cloud) | **Có** (bản Community) |
| Database tự sinh embedding? | Không (BYO) | **Có** (HeatWave GenAI) | Không (BYO) |
| Thuật toán index | HNSW, IVFFlat, (DiskANN qua pgvectorscale) | HNSW | HNSW |
| Thước đo khoảng cách | cosine / L2 / inner product | cosine / L2... | cosine / L2 |
| Tự cài trên máy mình được? | **Có** | Không (chỉ đám mây) | **Có** |
| Giới hạn số chiều khi index | ~2.000 (cao hơn thì dùng `halfvec`) | Tùy dịch vụ | **16.383** |
| Truy vấn lai vector + SQL | **Có, một câu** | Có | **Có, một câu** |
| Rủi ro bị khóa nhà cung cấp | Thấp | **Cao** | Thấp |

💡 Nếu chỉ được nhớ một dòng trong bảng này, hãy nhớ dòng cuối. Ba dòng đầu là kỹ thuật; dòng cuối là thứ quyết định bạn có thể đổi ý sau ba năm nữa hay không.

### 2.5. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển

**Lỗi 1 — Trên MariaDB, quên `LIMIT` khiến index không được dùng.**

Vector index của MariaDB chỉ được kích hoạt khi câu truy vấn có **đồng thời cả hai** thứ: `ORDER BY VEC_DISTANCE_*()` **và** `LIMIT`. Thiếu `LIMIT` là nó lặng lẽ quét toàn bảng.

```sql
-- ❌ SAI: không có LIMIT → quét toàn bộ bảng, chậm khủng khiếp khi dữ liệu lớn
SELECT id FROM chunks
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'));

-- ✅ ĐÚNG: có LIMIT → index HNSW được dùng
SELECT id FROM chunks
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 10;
```

Vì sao lại có quy tắc kỳ lạ này? Vì bản chất của index ANN là **tìm nhanh k phần tử gần nhất**, chứ không phải sắp xếp toàn bộ bảng. Nếu bạn không nói k bằng bao nhiêu, index không biết phải làm gì và database đành quay về cách làm thủ công. Điều tương tự cũng đúng ở mức độ nhẹ hơn với pgvector — hãy luôn có `LIMIT` trong truy vấn vector, ở mọi nền tảng.

**Lỗi 2 — Trộn embedding của hai mô hình khác nhau trong cùng một cột.**

Đã cảnh báo ở mục 1.2, giờ nói rõ hậu quả. Vector do mô hình A tạo và vector do mô hình B tạo nằm trên hai "bản đồ" khác nhau. So sánh chúng ra kết quả **vô nghĩa nhưng vẫn trông có vẻ hợp lý**.

Có hai kịch bản, và kịch bản thứ hai mới đáng sợ:
- Nếu hai mô hình khác số chiều (512 và 1536) → lệnh `INSERT` báo lỗi ngay. **Đây là trường hợp may mắn**, vì bạn phát hiện sớm.
- Nếu hai mô hình **cùng số chiều** (ví dụ hai mô hình khác nhau cùng ra 1536 chiều) → database vui vẻ nhận, không báo lỗi gì, và kết quả tìm kiếm sai một cách âm thầm suốt nhiều tháng.

💡 **Cách phòng:** ghi rõ **tên và phiên bản mô hình** vào một cột metadata bên cạnh, hoặc đặt tên bảng/cột theo mô hình. Khi đổi mô hình, phải sinh lại toàn bộ vector cho cả kho — quá trình này gọi là **re-embed**, và nó tốn kém (mục 4.3).

**Lỗi 3 — Nhầm `tsvector` với `vector` trên PostgreSQL.**

Đã nói ở mục 2.1. Nhắc lại vì đây là lỗi khiến người ta đọc lướt bài gốc rồi tưởng Postgres đã sẵn có vector search mà không cần cài gì. Muốn tìm theo ngữ nghĩa, **bắt buộc phải cài pgvector**.

### ✅ Self-check Phần 2

**1. Khác biệt then chốt giữa MySQL HeatWave và MariaDB Vector về "sinh embedding" và "mã nguồn mở" là gì?**
> *Gợi ý đáp án:* HeatWave **tự sinh embedding** trong database (HeatWave GenAI) nhưng là sản phẩm **độc quyền, chỉ chạy trên đám mây**. MariaDB **không sinh embedding** (BYO) nhưng **hoàn toàn mã nguồn mở**, nằm trong bản Community miễn phí và tự cài được.

**2. Trên MariaDB, điều kiện để vector index được sử dụng khi truy vấn là gì?**
> *Gợi ý đáp án:* Phải có **cả** `ORDER BY VEC_DISTANCE_*()` **và** `LIMIT` trong cùng câu truy vấn. Thiếu `LIMIT` thì rơi về quét toàn bảng mà không có cảnh báo nào.

**3. `tsvector` và `vector` trong PostgreSQL — cái nào cho semantic search, và cái kia dùng làm gì?**
> *Gợi ý đáp án:* `vector` (từ extension pgvector) dùng cho semantic search — lưu embedding. `tsvector` là của full-text search — lưu danh sách lexeme để tìm theo từ khóa. Hai thứ khác nhau hoàn toàn.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Phần này nhìn xuống bên dưới nắp capo: ba mô hình khác nhau ở tầng kỹ thuật nào, mỗi nền tảng dùng thuật toán gì, và những chi tiết engine tạo ra khác biệt hiệu năng thật.

### 3.1. Trade-off của ba mô hình kiến trúc

Trước hết, định nghĩa một từ sẽ xuất hiện dày đặc từ đây tới hết bài.

**Trade-off** (dịch: *sự đánh đổi*) là tình huống bạn không thể có tất cả cùng lúc: được cái này thì mất cái kia. Trong kỹ thuật gần như **không có lựa chọn tốt tuyệt đối** — chỉ có lựa chọn phù hợp với ràng buộc cụ thể của bạn. Khả năng gọi tên trade-off một cách rành mạch chính là thứ người phỏng vấn tìm kiếm ở ứng viên senior trở lên.

**Mô hình Extension (pgvector):**

*Được:* Linh hoạt tối đa. pgvector là dự án mã nguồn mở phát triển **độc lập** với PostgreSQL lõi, nên nó ra tính năng mới nhanh hơn nhiều so với nhịp một năm một bản của Postgres. Nhiều lựa chọn index (HNSW, IVFFlat, và DiskANN qua `pgvectorscale`). Hệ sinh thái dịch vụ vận hành sẵn rộng nhất.

*Mất:* Phải cài và phải quản lý phiên bản của extension như một thành phần riêng. Tính năng bạn dùng được phụ thuộc phiên bản extension — mà như đã nói ở mục 2.1, phiên bản đó nhiều khi do nhà cung cấp managed quyết định chứ không phải bạn. Một số nhà cung cấp khóa ở phiên bản cũ khá lâu.

**Mô hình Native (MariaDB):**

*Được:* Không phải cài gì, giảm ma sát vận hành. Vì nằm trong lõi, nhà phát triển tối ưu được sâu tới tầng engine — ví dụ dùng lệnh SIMD của CPU (mục 3.3), thứ mà một extension khó làm triệt để. Tính năng đi kèm bản phát hành database, nên phiên bản LTS cho bạn sự ổn định nhiều năm.

*Mất:* Bị buộc vào nhịp phát hành của database — muốn tính năng mới phải chờ bản mới, và với LTS thì chờ khá lâu. Ít "món" hơn extension: chỉ HNSW, chỉ cosine và L2.

**Mô hình Managed (HeatWave):**

*Được:* Không phải vận hành hạ tầng gì cả. Có sẵn **in-database embedding** — khỏi phải dựng và duy trì một tầng gọi mô hình AI. Gộp cả OLTP, phân tích và học máy trong một dịch vụ.

*Mất:* **Vendor lock-in ở mức cao nhất trong ba mô hình.** Sản phẩm độc quyền, chỉ chạy trên đám mây của nhà cung cấp, không tự cài được. Chi phí theo mức sử dụng và có thể tăng cùng lưu lượng. Và như mục 4.3 sẽ phân tích, chính cái tiện lợi nhất (tự sinh embedding) lại là sợi xích chặt nhất.

### 3.2. Thuật toán index — không phải ai cũng dùng HNSW

Trước khi so sánh các thuật toán, cần hiểu vì sao chúng tồn tại.

Cách tìm chính xác nhất là **brute force** (*vét cạn*): so vector truy vấn với **từng vector một** trong kho, rồi lấy k cái gần nhất. Cách này luôn cho đáp án đúng tuyệt đối, nhưng với 20 triệu vector 1536 chiều thì mỗi truy vấn phải làm khoảng 30 tỷ phép nhân. Không khả thi.

Vì vậy người ta chấp nhận **ANN** (viết tắt của *Approximate Nearest Neighbor*, dịch: *láng giềng gần nhất gần đúng*): tìm nhanh hơn hàng nghìn lần, đổi lại **thỉnh thoảng bỏ sót** một vài kết quả đúng.

Mức độ "bỏ sót" đó được đo bằng **recall** (dịch: *độ bao phủ*). Ví dụ recall@10 = 0,95 nghĩa là: trong 10 kết quả đúng nhất mà cách vét cạn tìm ra, thuật toán ANN tìm được 9,5 cái (trung bình). Đây là **đánh đổi trung tâm của mọi hệ vector search**: recall cao hơn thì chậm hơn và tốn RAM hơn.

Bây giờ tới các thuật toán. Điểm mà bài gốc không đề cập: **các nền tảng khác nhau dùng thuật toán khác nhau**, và điều đó dẫn tới hiệu năng khác nhau ở cùng một quy mô.

| Nền tảng | Thuật toán ANN | Ý tưởng cốt lõi | Đánh đổi chính |
|---|---|---|---|
| pgvector, MariaDB, phần lớn bản MySQL | **HNSW** | Đồ thị nhiều tầng — tầng trên nhảy xa, tầng dưới dò kỹ, như đi từ đường cao tốc xuống ngõ nhỏ | Recall cao, truy vấn nhanh, nhưng **ngốn RAM** vì đồ thị phải nằm trong bộ nhớ |
| pgvector | **IVFFlat** | Chia không gian thành các cụm, chỉ tìm trong vài cụm gần nhất | Xây nhanh, tốn ít RAM, nhưng recall thấp hơn HNSW |
| PlanetScale (Vitess) | **SPANN** | Lai giữa phân cụm và đồ thị, thiết kế để chạy tốt trên đĩa | Tiết kiệm RAM ở quy mô rất lớn |
| Google Cloud SQL for MySQL | **ScaNN** | Của Google — kết hợp lượng tử hóa và phân vùng | Nén vector lại để tiết kiệm bộ nhớ |
| pgvectorscale (Timescale) | **DiskANN** | Thiết kế để chạy chủ yếu trên SSD thay vì RAM | Chi phí thấp hơn nhiều ở quy mô hàng trăm triệu vector |

Hai từ mới cần giải thích. **SSD** (*Solid State Drive*) là loại ổ cứng thể rắn — chậm hơn RAM nhưng nhanh hơn ổ đĩa cơ rất nhiều, và **rẻ hơn RAM khoảng một bậc**. **Quantization** (dịch: *lượng tử hóa*) là kỹ thuật giảm độ chính xác của từng số trong vector để tiết kiệm bộ nhớ — ví dụ thay vì lưu mỗi số bằng 4 byte thì lưu bằng 2 byte, hoặc thậm chí 1 bit.

**Ý nghĩa ở tầm staff:** cụm từ "hỗ trợ vector search" che giấu rất nhiều khác biệt. Thuật toán index quyết định trực tiếp bốn thứ bạn quan tâm nhất: **RAM tiêu tốn, dung lượng đĩa, độ chính xác (recall), và tốc độ**. Ở quy mô nhỏ, mọi thuật toán đều trông như nhau; ở quy mô lớn, chính lựa chọn này quyết định hóa đơn hạ tầng của bạn.

Một cách nhớ gọn: **HNSW là "để hết trong RAM cho nhanh"; DiskANN và SPANN là "để trên đĩa cho rẻ"**. Khi kho vector còn vừa RAM thì HNSW luôn thắng. Khi nó không còn vừa RAM nữa, cuộc chơi đổi hoàn toàn.

### 3.3. Giới hạn số chiều, SIMD — những chi tiết hay bị bỏ qua

**Giới hạn số chiều khi đánh index.**

Đây là chi tiết kỹ thuật ít ai nhắc nhưng có thể phá hỏng cả kiến trúc của bạn:

- **pgvector:** index HNSW và IVFFlat hỗ trợ tối đa khoảng **2.000 chiều** cho kiểu `vector` thông thường. Muốn cao hơn thì phải dùng kiểu `halfvec` — một dạng lượng tử hóa lưu mỗi số bằng 2 byte thay vì 4, cho phép index tới khoảng 4.000 chiều.
- **MariaDB:** kiểu `VECTOR` hỗ trợ tới **16.383 chiều**.

Vì sao con số này quan trọng? Vì các mô hình embedding hiện đại đang ngày càng nhiều chiều. Mô hình `text-embedding-3-large` của OpenAI cho ra **3.072 chiều** — vượt giới hạn index thông thường của pgvector. Bạn vẫn **lưu** được, nhưng **không index** được ở dạng mặc định.

⚠️ **Chỗ khó — và đây là một edge case rất khó chịu.** Khi số chiều vượt giới hạn index, database vẫn cho bạn `INSERT` bình thường. Không lỗi, không cảnh báo. Chỉ là mọi truy vấn sẽ âm thầm quét toàn bảng. Hệ thống chạy đúng, chỉ chậm — và bạn chỉ phát hiện khi dữ liệu đủ lớn để chậm thành thảm họa.

**SIMD — tối ưu ở tầng phần cứng.**

**SIMD** (viết tắt của *Single Instruction, Multiple Data*, dịch: *một lệnh, nhiều dữ liệu*) là khả năng của CPU hiện đại thực hiện **cùng một phép tính trên nhiều số cùng lúc** thay vì lần lượt từng số.

Ví dụ cụ thể: để tính khoảng cách giữa hai vector 1536 chiều, bạn cần làm 1536 phép trừ. CPU thường làm lần lượt 1536 lần. Với SIMD, nó gom lại và làm 8 hoặc 16 phép trừ trong một lệnh — nhanh gấp cả chục lần.

Các tên viết tắt bạn sẽ gặp chỉ là tên của các bộ lệnh SIMD trên từng dòng CPU: **AVX2** và **AVX512** trên chip Intel/AMD, **NEON** trên chip ARM (bao gồm Apple Silicon và phần lớn máy chủ ARM hiện đại), **Power10** trên chip của IBM.

MariaDB Vector tự động dùng những bộ lệnh này mà bạn không phải cấu hình gì. pgvector cũng có hỗ trợ SIMD ở các phiên bản gần đây. Điểm cần nhớ ở tầm staff là: **cùng một thuật toán HNSW, cùng một số chiều, hai nền tảng vẫn có thể chênh nhau đáng kể về tốc độ chỉ vì mức độ tối ưu ở tầng thấp này.**

### 3.4. Vì sao "vector nằm cùng bảng" là ưu điểm lớn nhất

Đây là mục quan trọng nhất của Phần 3, và là chỗ đính chính bài gốc có ý nghĩa thực tiễn nhất.

Hãy lấy một yêu cầu nghiệp vụ hết sức bình thường: *"Tìm 10 sản phẩm giống ảnh này nhất, nhưng chỉ trong số còn hàng và giá dưới 50 đô."*

**Nếu vector nằm cùng bảng (RDBMS-native), câu trả lời là một câu SQL:**

```sql
SELECT id, name FROM products
WHERE in_stock = 1 AND price < 50
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 10;
```

Một câu lệnh, một lần gọi mạng, một giao dịch. Kết quả luôn nhất quán vì cả điều kiện lọc lẫn vector đều đọc từ cùng một ảnh chụp dữ liệu tại một thời điểm.

**Nếu vector nằm ở hệ thống riêng, chuyện phức tạp hơn nhiều:**

```
Bước 1: Gửi vector truy vấn sang hệ vector → nhận về danh sách ID sản phẩm giống nhất
Bước 2: Gửi danh sách ID đó sang cơ sở dữ liệu chính → hỏi cái nào còn hàng, giá dưới 50
Bước 3: Ghép kết quả lại trong code ứng dụng
```

Ba vấn đề nảy sinh, và không cái nào là nhỏ:

**Vấn đề thứ nhất — hai lần gọi mạng.** Mỗi lần gọi qua mạng tốn vài mili-giây, gọi là một **round-trip** (*một vòng đi-về*). Hai round-trip tuần tự nghĩa là độ trễ cộng dồn.

**Vấn đề thứ hai — bài toán "lọc sau" rất khó chịu.** Giả sử bạn lấy top 10 từ hệ vector, rồi lọc theo điều kiện còn hàng và giá — và phát hiện chỉ 2 trong 10 cái thỏa. Giờ bạn làm gì? Quay lại hỏi top 100? Rồi nếu vẫn không đủ thì hỏi top 1000? Đây là vấn đề nổi tiếng trong ngành, và nó **không có lời giải đẹp** khi hai hệ thống tách rời. Còn khi vector nằm cùng bảng, bộ tối ưu truy vấn của database tự xử lý chuyện này giúp bạn.

**Vấn đề thứ ba — đồng bộ dữ liệu.** Mỗi lần thêm, sửa, xóa sản phẩm, bạn phải nhớ cập nhật cả hệ vector. Quên một đường cập nhật nào đó là hai hệ lệch nhau — và triệu chứng sẽ là "sản phẩm đã xóa vẫn hiện trong kết quả tìm kiếm" hoặc "sản phẩm mới thêm không tìm thấy". Đây là **nguồn lỗi kinh điển** của mọi kiến trúc hai hệ thống.

Nhìn lại phép ví von ở mục 1.4: đây chính là **cái giá của việc sống ở hai căn nhà**. Không phải tiền mua nhà, mà là việc mỗi ngày phải nhớ đồ đạc của mình đang ở nhà nào.

### 3.5. Edge case — các trường hợp biên phải thủ sẵn

**Edge case** (dịch: *trường hợp biên*) là những tình huống hiếm gặp, nằm ở rìa dự tính, nhưng khi xảy ra thì làm hệ thống hỏng hoặc cho kết quả sai. Chủ động nêu edge case trong phỏng vấn luôn ghi điểm.

**Lệch đồng bộ khi tách hệ.** Đã nói ở 3.4. Nếu buộc phải dùng hệ vector riêng, hãy thiết kế cơ chế đồng bộ ngay từ đầu — thường là qua hàng đợi sự kiện — và có công cụ đối soát định kỳ để phát hiện lệch.

**Thước đo khoảng cách ngược chiều nhau.** Đã cảnh báo ở mục 1.3: có nền tảng trả về distance (nhỏ = gần), có nơi trả similarity (lớn = gần). Nhầm là `ORDER BY` ngược, và hệ thống trả về **những kết quả ít liên quan nhất** mà không báo lỗi.

**Số chiều vượt giới hạn index.** Đã nói ở 3.3: lưu được nhưng không index được, âm thầm quét toàn bảng.

**Bị khóa vào dịch vụ managed.** Dữ liệu và embedding nằm trong HeatWave rất khó bê nguyên sang nơi khác. **Hãy tính chi phí rời đi trước khi cam kết**, không phải sau.

**Cột vector cho phép NULL.** Nếu một số hàng chưa có embedding (ví dụ chưa kịp xử lý), chúng sẽ không bao giờ xuất hiện trong kết quả tìm kiếm — giống hệt vấn đề NULL trong giáo trình full-text search. Hãy có cơ chế theo dõi "còn bao nhiêu hàng chưa được sinh embedding".

### ✅ Self-check Phần 3

**1. Nêu một ưu và một nhược của mỗi mô hình: extension, native, managed.**
> *Gợi ý đáp án:* Extension — linh hoạt, ra tính năng nhanh / nhưng phải quản phiên bản và phụ thuộc nhà cung cấp managed. Native — không cài gì, tối ưu sâu tới engine / nhưng chờ nhịp phát hành database và ít lựa chọn hơn. Managed — không phải vận hành, có sẵn sinh embedding / nhưng lock-in cao và không tự cài được.

**2. Không phải nền tảng nào cũng dùng HNSW — kể hai thuật toán ANN khác và nền tảng dùng chúng.**
> *Gợi ý đáp án:* **SPANN** — PlanetScale (Vitess); **ScaNN** — Google Cloud SQL for MySQL; **DiskANN** — pgvectorscale; **IVFFlat** — pgvector. Điểm nên nói thêm: HNSW nhanh nhưng ngốn RAM, còn DiskANN/SPANN thiết kế để chạy trên đĩa nên rẻ hơn ở quy mô rất lớn.

**3. Vì sao "vector cùng bảng với dữ liệu nghiệp vụ" tốt hơn "tách hệ riêng" cho truy vấn lai?**
> *Gợi ý đáp án:* Một câu SQL thay vì hai vòng gọi mạng; tránh hoàn toàn bài toán "lọc sau" khi top-k không đủ kết quả thỏa điều kiện; và không phải đồng bộ dữ liệu giữa hai hệ — nguồn lỗi kinh điển.

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Ba phần trước trả lời "làm thế nào". Phần này trả lời **"chọn cái gì, vì sao, và khi nào tôi sẽ đổi ý"** — đó là công việc thật của một staff engineer.

### 4.1. Khung quyết định thứ nhất: RDBMS-native hay Vector DB chuyên dụng

Đây là quyết định kiến trúc thật mà bài gốc không chạm tới. Bài gốc chỉ liệt kê các RDBMS; nhưng một staff engineer phải biết **cả** phương án chuyên dụng (Pinecone, Weaviate, Qdrant, Milvus) và biết khi nào chọn cái nào.

**Chọn RDBMS-native (pgvector / MariaDB / HeatWave) khi:**

- Vector search là **một tính năng bên trong** một ứng dụng vốn đã chạy trên cơ sở dữ liệu quan hệ.
- Bạn cần **truy vấn lai** — vector kết hợp lọc và nối với dữ liệu nghiệp vụ — trong một giao dịch ACID. (**ACID** là bộ bốn đảm bảo của giao dịch database: làm hết hoặc không làm gì, dữ liệu luôn nhất quán, các giao dịch không giẫm chân nhau, và đã ghi là không mất.)
- Quy mô dưới khoảng **10–50 triệu vector**. Con số này nghe có vẻ nhỏ nhưng thực ra bao trùm **đa số** hệ thống tìm kiếm và RAG trong doanh nghiệp.
- Bạn muốn ít hệ thống, ít chi phí, và đội hiện tại tự vận hành được.

**Chọn Vector DB chuyên dụng khi:**

- Vector search **chính là sản phẩm**, không phải một tính năng phụ.
- Quy mô từ vài chục triệu tới hàng tỷ vector, cần phân tán nhiều vùng địa lý.
- Cần tính năng chuyên sâu mà RDBMS chưa có: kết hợp sẵn tìm kiếm dày và thưa, quản lý nhiều khách hàng ở mức nâng cao, lọc phức tạp ở quy mô rất lớn.
- Tổ chức không có sẵn RDBMS phù hợp, hoặc cố ý muốn tách hoàn toàn khối lượng công việc vector ra khỏi database chính.

> **Câu chốt tầm staff:** *"Đặt vector cạnh dữ liệu nghiệp vụ thắng ở sự đơn giản trong vận hành; tách ra hệ chuyên dụng thắng ở quy mô đỉnh. Chọn theo câu hỏi: vector search là một tính năng hay là cả sản phẩm?"*

🧩 **[Ngoài bài gốc] Một xu hướng đáng nói của 2026.** Sau làn sóng ai cũng dựng vector database riêng trong giai đoạn 2023–2024, khá nhiều đội đã **quay ngược trở lại** đặt vector vào PostgreSQL. Lý do không phải vì pgvector nhanh hơn Pinecone — nó không nhanh hơn. Lý do là **vận hành thêm một database là một cái giá thật mà người ta đã đánh giá thấp**. Bài học ở đây rất đáng nhớ: khi chọn kiến trúc, hãy tính cả chi phí vận hành nhiều năm chứ đừng chỉ nhìn điểm benchmark.

### 4.2. Khung quyết định thứ hai: Managed hay Self-host

**Self-host** (dịch: *tự vận hành*) nghĩa là bạn tự cài, tự chạy, tự bảo trì phần mềm trên hạ tầng của mình.

| | **Self-host** (pgvector, MariaDB Community) | **Managed** (HeatWave, Supabase, Neon, RDS) |
|---|---|---|
| Vận hành | Bạn tự lo: vá lỗi, sao lưu, mở rộng | Nhà cung cấp lo hết |
| Chi phí | Rẻ về bản quyền, tốn về nhân lực | Trả theo mức dùng, tăng theo lưu lượng |
| Kiểm soát dữ liệu | **Toàn quyền, dữ liệu ở trong nhà** | Tiện, nhưng dữ liệu nằm trên hạ tầng bên thứ ba |
| Rủi ro lock-in | Thấp (mã nguồn mở, bê đi được) | **Cao**, đặc biệt với sản phẩm độc quyền |
| Tính năng dùng ngay | Phải tự lắp | Có sẵn: tự sinh embedding, tự mở rộng quy mô |

⚠️ **Chỗ khó — và là điều nhiều kỹ sư quên mất.** Đôi khi **ràng buộc pháp lý quyết định thay bạn**, trước cả khi bàn tới hiệu năng.

**Compliance** (dịch: *tuân thủ quy định*) là các yêu cầu pháp lý hoặc ngành nghề về cách xử lý dữ liệu. Nếu công ty bạn làm y tế, tài chính, hoặc phục vụ khách hàng châu Âu, rất có thể có điều khoản kiểu **"dữ liệu khách hàng không được rời khỏi hạ tầng của chúng tôi"**. Khi điều khoản đó tồn tại, mọi dịch vụ đám mây bị loại khỏi bàn cân **trước khi** ai kịp so sánh tốc độ.

Đây là một bài học rất đặc trưng của tầm staff: **hãy tìm ra ràng buộc cứng trước, rồi mới tối ưu trong phần không gian còn lại.** Tối ưu một phương án rồi mới phát hiện nó vi phạm quy định là cách lãng phí vài tuần của cả đội.

### 4.3. Lock-in và migration — chi phí ẩn lớn nhất

**Migration** (dịch: *di chuyển / chuyển đổi*) là việc chuyển hệ thống từ nền tảng này sang nền tảng khác. Với vector, chi phí này cao hơn người ta tưởng rất nhiều — và đây là chỗ quyết định hôm nay quyết định luôn khả năng xoay xở của bạn trong ba năm tới.

**Cái bẫy của in-database embedding.** Tính năng tự sinh embedding của HeatWave rất tiện, nhưng hãy nghĩ kỹ về hệ quả: những vector trong kho của bạn được sinh ra bởi **mô hình của nhà cung cấp đó**. Nhớ lại cảnh báo ở mục 1.2 — vector từ hai mô hình khác nhau **không so sánh được với nhau**.

Nghĩa là: nếu sau này bạn muốn chuyển sang nền tảng khác, bạn không thể chỉ **xuất dữ liệu ra rồi nạp vào**. Bạn phải **sinh lại toàn bộ vector từ đầu** bằng mô hình mới — quá trình gọi là **re-embed**.

Re-embed tốn kém tới mức nào? Với 20 triệu tài liệu, bạn phải chạy 20 triệu lượt gọi mô hình. Tùy mô hình, đó là hàng nghìn tới hàng chục nghìn đô, cộng nhiều ngày xử lý, cộng việc phải chạy song song hai bộ index trong lúc chuyển đổi.

Đây chính là ý của câu one-liner: *"tiện lợi đi kèm sợi xích"*. Với **BYO-embedding** (pgvector, MariaDB), bạn kiểm soát mô hình, nên đổi database không bắt buộc phải đổi vector.

**Đổi RDBMS cũng là migration nặng.** Không chỉ chuyển dữ liệu, mà còn phải viết lại: cú pháp truy vấn (`<=>` của pgvector khác `VEC_DISTANCE_COSINE()` của MariaDB khác `DISTANCE()` của MySQL), cách khai báo index, và toàn bộ tầng code gọi database.

**Chiến lược giữ đường lui — ba việc nên làm ngay từ đầu:**

1. **Ưu tiên BYO-embedding và chuẩn mở.** Kiểm soát mô hình embedding của chính mình.
2. **Ghi phiên bản cho embedding.** Lưu tên và phiên bản mô hình cùng mỗi vector, để sau này biết cái nào cần sinh lại.
3. **Tránh phụ thuộc vào tính năng độc quyền** của một nhà cung cấp nếu khả năng chuyển đổi là điều bạn coi trọng. Nếu vẫn quyết định dùng, hãy **ghi lại quyết định đó và cái giá của nó bằng văn bản** — để người kế nhiệm bạn ba năm sau hiểu vì sao.

### 4.4. Chi phí, độ tin cậy, giám sát

**Những thứ thật sự tốn tiền:**

- **RAM** — đây là khoản lớn nhất. Index HNSW cần nằm trong bộ nhớ để nhanh, và RAM đắt hơn đĩa khoảng một bậc. Với 20 triệu vector 1536 chiều, chỉ riêng dữ liệu thô đã khoảng 120 GB, chưa kể cấu trúc đồ thị của index.
- **Phí dịch vụ managed** theo mức sử dụng, thường tăng cùng lưu lượng.
- **Phí sinh embedding** — trả cho mỗi lượt gọi mô hình, cả lúc nạp dữ liệu ban đầu lẫn lúc có dữ liệu mới.
- **Re-embed khi đổi mô hình** — khoản chi phí bất ngờ mà không ai dự toán, như đã phân tích ở 4.3.

**Độ tin cậy — một lợi thế ít người nhắc tới.** Khi vector nằm trong RDBMS, nó **thừa hưởng toàn bộ cơ chế sao lưu, nhân bản, và khôi phục sau sự cố** đã được kiểm chứng hàng chục năm của database mẹ. Các cơ sở dữ liệu vector chuyên dụng còn khá non trẻ và chưa có bề dày đó. "Một hệ thống để trông thay vì hai" cũng đồng nghĩa với "một quy trình khôi phục thảm họa thay vì hai".

**Giám sát — những chỉ số cần theo dõi:**

- **recall@k** — định kỳ so kết quả của index với kết quả vét cạn trên một tập mẫu, để biết chất lượng tìm kiếm có tụt không. Đây là chỉ số **hay bị bỏ quên nhất**, vì recall tụt không gây lỗi, không làm chậm — nó chỉ lặng lẽ làm kết quả tệ đi.
- **Độ trễ p95 và p99.** **p95** nghĩa là "95% truy vấn nhanh hơn con số này". Dùng p95/p99 thay vì trung bình, vì trung bình che giấu đúng những trường hợp tệ nhất mà người dùng nhớ và phàn nàn.
- **Kích thước index và mức dùng RAM** — để biết trước khi nào kho vector không còn vừa bộ nhớ, thời điểm mà mọi tính toán chi phí phải làm lại.
- **Nếu tách hệ:** độ trễ đồng bộ và tỉ lệ lệch dữ liệu giữa hai hệ thống.

### 4.5. Ảnh hưởng tổ chức và cách nói chuyện với người không rành kỹ thuật

**Stakeholder** (dịch: *bên liên quan*) là những người có lợi ích gắn với dự án nhưng không nhất thiết hiểu kỹ thuật: quản lý sản phẩm, giám đốc, bộ phận kinh doanh, bộ phận pháp chế. Một staff engineer phải trình bày quyết định kỹ thuật bằng ngôn ngữ của họ — tức là bằng **rủi ro, chi phí, thời gian, và ràng buộc pháp lý**.

**Cách nói với quản lý sản phẩm hoặc sếp:**

> *"Chúng ta có thể thêm tìm kiếm theo ý nghĩa mà **không dựng hệ thống mới** — cắm thẳng vào database đang chạy. Đội hiện tại vận hành được, chi phí và rủi ro đều thấp, và dữ liệu vẫn nằm nguyên một chỗ.*
>
> *Có một lựa chọn khác là dùng dịch vụ trọn gói của nhà cung cấp: nhanh hơn khi bắt đầu và có sẵn phần AI. Nhưng đổi lại chúng ta bị khóa vào họ — nếu sau này muốn chuyển đi, chi phí sẽ rất lớn — và dữ liệu khách hàng phải ra khỏi hệ thống của mình.*
>
> *Nếu sau này lượng dữ liệu tăng gấp mười, ta sẽ cân nhắc một hệ chuyên dụng. Nhưng đó là bài toán của lúc đã thành công, không phải bây giờ."*

Hãy để ý: không có chữ nào là "HNSW", "recall", hay "cosine distance". Thay vào đó là chi phí, rủi ro, ràng buộc dữ liệu, và một lộ trình rõ ràng. Đó là ngôn ngữ mà stakeholder nghe được và ra quyết định được.

**Cấu trúc đội ngũ.** Chọn RDBMS-native nghĩa là **một** đội database lo tất. Tách hệ vector riêng thường kéo theo nhu cầu kỹ năng mới, quy trình trực sự cố mới, và về lâu dài có thể là một vị trí tuyển dụng mới. Đây là lập luận **tổ chức**, không phải kỹ thuật — và biết đưa loại lập luận này ra bàn đúng lúc là điều người ta chờ đợi ở tầm staff.

**Chuẩn hóa ở quy mô công ty.** Nếu nhiều đội cùng cần vector, hãy chọn **một** mô hình chung (ví dụ: mọi dịch vụ chạy Postgres thì dùng pgvector) thay vì để mỗi đội tự chọn một vector database khác nhau. Sự phân mảnh này ban đầu trông vô hại, nhưng sau hai năm nó biến thành năm hệ thống khác nhau mà không ai nắm hết.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff engineer

> **Đề bài:** *"Công ty bạn chạy PostgreSQL cho ứng dụng chính. Cần thêm tìm kiếm theo ngữ nghĩa cho 20 triệu tài liệu. Dữ liệu khách hàng nhạy cảm, không được gửi ra API bên ngoài. Đội backend nhỏ. Bạn chọn nền tảng vector thế nào?"*

Điều quan trọng nhất không phải "đáp án đúng", mà là **nói to quá trình loại trừ và những đánh đổi bạn đang cân nhắc**.

**Nhịp 1 — Làm rõ đề trước khi vẽ.** Hỏi: QPS dự kiến bao nhiêu? (**QPS** = *Queries Per Second*, số truy vấn mỗi giây.) Mục tiêu recall là bao nhiêu? Dữ liệu tăng trưởng thế nào? Ràng buộc tuân thủ cụ thể là gì — không được ra khỏi công ty, hay chỉ không được ra khỏi lãnh thổ? Ngân sách ra sao?

**Nhịp 2 — Dùng ràng buộc cứng để loại phương án trước.** Đây là bước thể hiện tư duy staff rõ nhất:
- Dữ liệu nhạy cảm, không được gửi ra API ngoài → **loại mọi dịch vụ managed độc quyền** (HeatWave) và **loại luôn việc gọi API embedding của bên thứ ba** như OpenAI.
- Đội backend nhỏ → **loại phương án thêm một hệ thống mới phải học và trực sự cố**.
- Còn lại: **self-host pgvector, kèm mô hình embedding tự chạy trong nhà**.

**Nhịp 3 — Giải thích vì sao không dùng vector DB chuyên dụng.** 20 triệu vector nằm dưới ngưỡng cần Pinecone/Milvus; vector là tính năng chứ không phải sản phẩm; công ty đã chạy Postgres nên tránh được cả một hệ thống lẫn bài toán đồng bộ; và lọc theo quyền truy cập của từng khách hàng làm ngay trong SQL thay vì phải xử lý thủ công ở tầng ứng dụng.

**Nhịp 4 — Kiến trúc cụ thể.** Cột `vector` với index HNSW; embedding sinh bằng mô hình mã nguồn mở tự chạy (BGE-M3, MiniLM hoặc tương đương); kết hợp thêm full-text search nếu người dùng cũng tìm bằng từ khóa chính xác — tức là **hybrid search**, nối lại đúng nội dung giáo trình trước; phân mảnh bảng theo khách hàng nếu là hệ thống nhiều khách.

**Nhịp 5 — Giữ đường lui.** BYO-embedding và chuẩn mở để không bị khóa; ghi phiên bản mô hình cùng mỗi vector; nếu sau này đổi mô hình thì triển khai kiểu **blue-green** — dựng song song bộ index mới, kiểm chứng chất lượng xong mới chuyển toàn bộ lưu lượng, vì vector cũ và mới không trộn lẫn được.

**Nhịp 6 — Nêu rõ ngưỡng sẽ khiến bạn đổi ý.** "Nếu dữ liệu lên hàng trăm triệu vector, hoặc phải phục vụ nhiều vùng địa lý, hoặc RAM cho index vượt ngân sách — lúc đó tôi sẽ cân nhắc pgvectorscale với DiskANN trước, và chỉ tách sang hệ chuyên dụng nếu vẫn không đủ."

**Nhịp 7 — Câu chốt.** *"Tôi chọn RDBMS tự vận hành vì ràng buộc về quyền riêng tư và vì quy mô hiện tại, không phải vì nó luôn là lựa chọn tốt nhất — và tôi biết chính xác ngưỡng nào sẽ khiến tôi đổi ý."*

Câu cuối cùng đó là thứ phân biệt rõ nhất giữa một câu trả lời khá và một câu trả lời ở tầm staff: **biết lý do của lựa chọn, và biết cả giới hạn của nó.**

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Bản rút gọn để ôn nhanh. Mọi thứ ở đây đã được giải thích đầy đủ phía trên.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ | Định nghĩa một dòng |
|---|---|
| **Vector / Embedding** | Dãy số biểu diễn nội dung sao cho nội dung gần nghĩa cho ra vector gần nhau |
| **Dimension (số chiều)** | Số phần tử trong vector; phải khớp giữa mô hình và cột trong database |
| **Distance metric** | Cách đo độ gần: cosine (góc — mặc định cho văn bản), Euclid/L2, inner product, Manhattan, Jaccard |
| **k-NN / ANN** | Tìm k láng giềng gần nhất / tìm gần đúng để đổi độ chính xác lấy tốc độ |
| **Recall@k** | Tỉ lệ kết quả đúng mà index ANN tìm được so với vét cạn — thước đo chất lượng tìm kiếm |
| **pgvector** | Extension thêm kiểu `vector` cho PostgreSQL; dòng 0.8.x (2026) |
| **MariaDB Vector** | Hỗ trợ **native**, kiểu `VECTOR`, ổn định từ 11.8 LTS, nằm trong Community miễn phí |
| **MySQL HeatWave** | Dịch vụ **managed** của Oracle; có **in-database embedding** (HeatWave GenAI) |
| **Extension / Native / Managed** | Ba mô hình hỗ trợ vector: cài thêm / xây sẵn trong lõi / thuê trọn gói |
| **BYO-embedding** | Tự sinh vector bên ngoài rồi nạp vào (pgvector, MariaDB) — bạn kiểm soát mô hình |
| **In-database embedding** | Database tự sinh vector bằng LLM tích hợp (HeatWave) — tiện nhưng khóa chặt |
| **Hybrid query** | Vector search + lọc/nối SQL trong một câu, một giao dịch |
| **HNSW** | Đồ thị nhiều tầng; nhanh, recall cao, **ngốn RAM** — mặc định của pgvector và MariaDB |
| **IVFFlat / SPANN / ScaNN / DiskANN** | Các thuật toán ANN khác: phân cụm / PlanetScale / Google Cloud SQL / pgvectorscale (chạy trên SSD) |
| **SIMD (AVX2, AVX512, NEON)** | Bộ lệnh CPU tính nhiều số cùng lúc — tăng tốc phép đo khoảng cách |
| **Quantization / halfvec** | Giảm độ chính xác từng số để tiết kiệm bộ nhớ và index được nhiều chiều hơn |
| **Dedicated vector DB** | Pinecone, Weaviate, Qdrant, Milvus — hệ chuyên dụng cho vector |
| **Vendor lock-in** | Bị khóa vào một nhà cung cấp; rời đi rất tốn kém |
| **Re-embed** | Sinh lại toàn bộ vector bằng mô hình mới — chi phí ẩn lớn nhất khi chuyển nền tảng |
| **GA / LTS / fork / proprietary** | Phát hành chính thức / hỗ trợ dài hạn / nhánh rẽ mã nguồn / sản phẩm độc quyền |
| **OLTP / OLAP** | Xử lý giao dịch thời gian thực / truy vấn phân tích quy mô lớn |
| **`tsvector` ≠ `vector`** | FTS theo lexeme khác hoàn toàn embedding — đừng nhầm |

### 5.2. Core concepts — nếu chỉ nhớ 10 điều

1. RDBMS thêm vector để bạn **khỏi dựng database riêng**; có ba cách: **extension / native / managed**.
2. **pgvector = extension, MariaDB = native, HeatWave = managed** — và chỉ HeatWave tự sinh embedding.
3. Siêu năng lực chung của RDBMS-native là **hybrid query**: vector cộng dữ liệu nghiệp vụ trong một câu SQL, một giao dịch.
4. **MariaDB Vector đã ổn định, mã nguồn mở, và lưu vector CÙNG BẢNG** — không tách riêng như mô tả cũ.
5. **MySQL Community có kiểu vector nhưng index ANN chủ yếu nằm sau sản phẩm độc quyền của Oracle** — đây là khác biệt then chốt so với MariaDB.
6. Không phải ai cũng dùng HNSW: PlanetScale dùng SPANN, Cloud SQL dùng ScaNN, pgvectorscale dùng DiskANN. **HNSW = nhanh nhưng ngốn RAM; DiskANN/SPANN = rẻ ở quy mô lớn.**
7. **BYO-embedding linh hoạt; in-database embedding tiện nhưng khóa chặt** — đổi nền tảng là phải re-embed toàn bộ.
8. Vector từ hai mô hình khác nhau **không so sánh được**, và nếu cùng số chiều thì database **không báo lỗi** — sai âm thầm.
9. Bốn trục quyết định: (a) vector là tính năng hay sản phẩm, (b) quy mô, (c) managed hay tự vận hành, (d) quyền riêng tư và lock-in.
10. `tsvector` (full-text search) khác `vector` (embedding) — semantic search cần cái sau.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Cơi nới thêm phòng hay mua căn nhà thứ hai"** — RDBMS-native so với vector DB chuyên dụng. Nhớ nói cả vế quan trọng: cái giá không nằm ở tiền mua nhà mà ở việc phải sống ở hai nơi.
- **"Tính năng hay sản phẩm?"** — kim chỉ nam để chọn giữa hai hướng trên.
- **"Extension là cài thêm, native là xây sẵn, managed là thuê trọn"** — ba mô hình trong một câu.
- **"Sự tiện lợi đi kèm sợi xích"** — in-database embedding đổi tiện lợi lấy lock-in.
- **"Một hệ để trông, không phải hai"** — lợi thế vận hành, nói được với cả người không rành kỹ thuật.
- **"Bản đồ tọa độ"** — giải thích embedding trong 10 giây cho bất kỳ ai.

### 5.4. Code cần thuộc lòng

**(a) PostgreSQL + pgvector (extension):**
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE t (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON t USING hnsw (embedding vector_cosine_ops);
SELECT id FROM t ORDER BY embedding <=> '[...]' LIMIT 5;
```

**(b) MariaDB (native):**
```sql
CREATE TABLE t (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  embedding VECTOR(1536) NOT NULL,
  VECTOR INDEX (embedding) DISTANCE=cosine
) ENGINE=InnoDB;

SELECT id FROM t
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 5;                        -- LIMIT là BẮT BUỘC để index được dùng
```

**(c) MySQL:**
```sql
CREATE TABLE t (id INT PRIMARY KEY, val VECTOR(5));
INSERT INTO t VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));
SELECT id FROM t ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE') LIMIT 5;
```

**(d) Hybrid query — câu đắt giá nhất để viết ra trong phỏng vấn:**
```sql
-- "Sản phẩm giống nhất, nhưng còn hàng và dưới 50 đô" — MỘT câu, MỘT giao dịch
SELECT id, name FROM products
WHERE in_stock = 1 AND price < 50
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 10;
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "PostgreSQL, MySQL và MariaDB hỗ trợ vector khác nhau thế nào?"**
> PostgreSQL: qua **extension** pgvector, phải cài, nhưng linh hoạt và có nhiều loại index. MariaDB: **native** — kiểu `VECTOR` trong lõi, ổn định từ 11.8 LTS, nằm trong bản Community mã nguồn mở. MySQL: Community có kiểu `VECTOR` nhưng khả năng đánh index vector chủ yếu qua HeatWave — sản phẩm managed, độc quyền, kèm tính năng tự sinh embedding.

**2. [BẪY] "PostgreSQL đã có sẵn vector search qua `tsvector` rồi đúng không?"**
> Không. `tsvector` là kiểu của **full-text search**, lưu danh sách lexeme để tìm theo từ khóa. Semantic search cần kiểu `vector` của extension **pgvector**. Hai thứ chỉ trùng chữ cái chứ không liên quan gì tới nhau.

**3. [BẪY] "MariaDB lưu vector ở một database riêng phải không?"**
> Không, với bản chính thức. Embedding nằm trong cột `VECTOR` **cùng bảng** với dữ liệu nghiệp vụ, nên chạy được hybrid query trong một câu SQL. Mô tả "lưu riêng biệt" là thông tin cũ từ giai đoạn thử nghiệm.

**4. [TRADE-OFF] "Chọn RDBMS-native hay vector DB chuyên dụng như Pinecone?"**
> Native khi vector là **tính năng** trong app đã chạy RDBMS, quy mô dưới vài chục triệu, cần hybrid query, và muốn ít hệ thống. Chuyên dụng khi vector search **là sản phẩm**, quy mô rất lớn hoặc đa vùng, cần tính năng chuyên sâu. Nói thêm được xu hướng 2026 — nhiều đội đã quay ngược về Postgres vì chi phí vận hành hệ thứ hai — sẽ ghi điểm.

**5. "HeatWave khác pgvector và MariaDB ở chỗ nào?"**
> HeatWave là dịch vụ managed và **tự sinh embedding trong database**; nhưng độc quyền, chỉ chạy trên đám mây, và lock-in cao. pgvector với MariaDB thì mã nguồn mở, tự cài được, theo mô hình BYO-embedding nên bạn kiểm soát mô hình.

**6. [TRADE-OFF] "In-database embedding hay ở chỗ nào, dở ở chỗ nào?"**
> Hay: bớt hẳn một tầng hạ tầng, ít code, ít thứ hỏng. Dở: vector của bạn bị buộc vào mô hình của nhà cung cấp, nên chuyển nền tảng là phải **re-embed toàn bộ kho** — với 20 triệu tài liệu thì đó là hàng nghìn đô và nhiều ngày xử lý.

**7. [SCALE] "Công ty dùng Postgres, cần vector cho 20 triệu tài liệu, dữ liệu nhạy cảm, đội nhỏ?"**
> Self-host pgvector cộng mô hình embedding tự chạy. Lý do: ràng buộc quyền riêng tư loại ngay dịch vụ đám mây và API bên ngoài; đội nhỏ loại phương án thêm hệ thống; 20 triệu vector vẫn nằm trong tầm Postgres. Nêu thêm ngưỡng sẽ khiến đổi ý — hàng trăm triệu vector hoặc đa vùng — để thể hiện biết giới hạn lựa chọn của mình.

**8. "Có phải nền tảng nào cũng dùng HNSW không?"**
> Không. pgvector, MariaDB và phần lớn bản MySQL dùng HNSW; PlanetScale dùng SPANN; Google Cloud SQL dùng ScaNN; pgvectorscale dùng DiskANN. Khác biệt cốt lõi: HNSW nằm trong RAM nên nhanh nhưng đắt; DiskANN và SPANN thiết kế để chạy trên SSD nên rẻ hơn nhiều ở quy mô rất lớn.

**9. [BẪY] "Tôi đổi mô hình embedding mới tốt hơn, cứ nạp vector mới vào cùng cột được không?"**
> Không được. Vector từ hai mô hình nằm trên hai không gian khác nhau nên so sánh vô nghĩa. Nếu khác số chiều thì `INSERT` báo lỗi — còn may; nếu **cùng số chiều thì database nhận bình thường và kết quả sai âm thầm**. Phải re-embed toàn bộ và triển khai kiểu blue-green.

**10. "Điều gì thực sự quyết định việc chọn nền tảng?"**
> Thường không phải hiệu năng. Thứ tự thực tế là: ràng buộc pháp lý về dữ liệu → quy mô hiện tại và dự kiến → năng lực vận hành của đội → mức chấp nhận lock-in. Hiệu năng chỉ là yếu tố cuối cùng, trong phần không gian còn lại sau khi các ràng buộc cứng đã loại bớt phương án.

### 5.6. Câu trả lời "one-liner" đắt giá

- *"Every major RDBMS now speaks vector — the question isn't 'can it', it's 'extension, native, or managed', and what each one costs you in lock-in."*
- *"RDBMS-native wins on operational simplicity; dedicated vector DBs win on peak scale. Pick by whether vector search is a feature or the product."*
- *"HeatWave generates embeddings for you — convenient, but that convenience is a leash: switch platforms and you re-embed everything."*
- *"MariaDB keeps embeddings in a column next to your data, so 'find similar products under $50 that are in stock' is one SQL statement, one transaction."*
- *"tsvector is keywords, vector is meaning — confusing the two is the fastest way to fail a vector-search interview."*
- *"Sometimes privacy picks the platform: if the data can't leave the building, managed cloud services are off the table before performance even enters the conversation."*
- *"The expensive part of a second database isn't the servers — it's living in two houses."*

---

## 📌 Ghi chú cuối

**Bốn điểm đính chính cần nhớ đúng** (đây là những chỗ hay thành câu hỏi bẫy):
1. MariaDB Vector đã chính thức ổn định trong 11.8 LTS và **lưu vector cùng bảng**, không tách riêng.
2. `tsvector` (full-text search) **không phải** `vector` (embedding).
3. MySQL Community có kiểu vector nhưng **index ANN chủ yếu qua sản phẩm độc quyền của Oracle**.
4. PlanetScale đã có index vector (thuật toán SPANN), không còn ở thì tương lai.

**Kiểm chứng lại trước khi phỏng vấn.** Mảng này đổi rất nhanh — mọi con số phiên bản trong giáo trình này nên được xác nhận lại từ tài liệu chính thức của pgvector, MariaDB (11.8 trở lên) và MySQL/HeatWave. Thứ **không** đổi là cách tư duy: ba mô hình kiến trúc và khung quyết định ở Phần 4. Học chắc phần đó.

**Thực hành — phần quan trọng nhất.** Đọc xong thì bạn *biết*, nhưng phải gõ tay thì mới *thuộc*:
1. Dựng song song hai container Docker: một `pgvector/pgvector`, một MariaDB 11.8.
2. Tạo **cùng một bảng vector** trên cả hai bên, nạp cùng một tập dữ liệu.
3. Chạy **cùng một truy vấn** và so cú pháp `<=>` với `VEC_DISTANCE_COSINE()`. Cảm giác "cùng ý tưởng, khác cú pháp" chỉ thật sự thấm khi bạn tự gõ.
4. Trên MariaDB, cố tình **bỏ `LIMIT`** rồi đo lại thời gian — để tận mắt thấy index không được dùng nghĩa là gì.
5. Viết một truy vấn hybrid có cả `WHERE` lọc nghiệp vụ lẫn `ORDER BY` theo khoảng cách, rồi thử hình dung xem bạn sẽ phải viết bao nhiêu code nếu vector nằm ở hệ thống khác.

**Nối mạch cả series (đủ năm mảnh):** tạo vector (embedding) → làm cho tìm nhanh (indexing) → lưu và truy vấn (pgvector) → tìm theo từ khóa (full-text search) → **chọn nền tảng (bài này)**. Năm mảnh này gộp lại cho bạn đủ hiểu biết để xây và vận hành một hệ thống semantic search hoặc RAG thật.

**Học tiếp gì sau bài này:**
- **Benchmark có hệ thống** giữa các nền tảng — tìm hiểu bộ ann-benchmarks để biết cách đo recall và độ trễ cho đúng, thay vì tin vào con số marketing.
- **Chiến lược migration** giữa các kho vector, và cách triển khai blue-green khi đổi mô hình embedding.
- **Hybrid dense + sparse** — kết hợp vector dày (ngữ nghĩa) với vector thưa (từ khóa) trong cùng một mô hình, ví dụ BGE-M3.
- **pgvectorscale và DiskANN** — con đường mở rộng Postgres lên hàng trăm triệu vector mà không cần đổi nền tảng.
