# Vector Search với PostgreSQL (pgvector) — Giáo trình siêu dễ hiểu, từ Basic đến Staff Level

> **Bài giảng gốc:** video mở đầu khoá *IBM Vector Database Fundamentals — Course 3*. Video gốc chỉ nêu 3 mục tiêu học tập (modeling vector data, indexing techniques, dùng PostgreSQL để store/retrieve/query vectors) chứ chưa dạy nội dung kỹ thuật. Vì vậy phần lớn giáo trình này là phần **giảng mở rộng** của tôi (một Staff Engineer), bám sát đúng 3 mục tiêu đó. Những chỗ đi xa hơn bài gốc được đánh dấu 🧩 **[Ngoài bài gốc]**.
>
> **Cập nhật tới tháng 7/2026:** pgvector đang ở nhánh **0.8.x**, bản mới nhất trên GitHub là **v0.8.5**. **HNSW** là loại index được khuyên dùng mặc định. Một lưu ý vận hành quan trọng: bản **0.8.2 (2026-02)** vá một lỗ hổng bảo mật (CVE-2026-3172) gây tràn bộ nhớ khi build index HNSW song song — nếu bạn đang chạy 0.8.0/0.8.1 thì nên nâng cấp. Trước khi đi phỏng vấn, nhớ liếc lại CHANGELOG trên GitHub `pgvector/pgvector` vì dự án này phát triển rất nhanh.
>
> **Cách đọc file này:** đọc tuần tự từ Phần 0 → Phần 5. Mỗi phần đều dựa trên phần trước. Nếu gặp một từ lạ, đừng lo — quy tắc của giáo trình này là **mọi thuật ngữ đều được giải thích ngay lần đầu nó xuất hiện**. Nếu bạn thấy một từ chưa được giải thích, đó là lỗi của tôi chứ không phải của bạn.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### Bài này dạy gì — nói bằng một câu đơn giản nhất

**Bài này dạy bạn cách làm cho một cơ sở dữ liệu bình thường có khả năng "tìm theo ý nghĩa" thay vì chỉ "tìm theo đúng chữ".**

Ví dụ: khách hàng gõ "đồ giữ ấm mùa đông", và hệ thống trả về "áo khoác lông vũ" — dù trong tên sản phẩm đó **không có một chữ nào** trùng với câu khách gõ. Máy tính hiểu được rằng hai thứ này *liên quan về mặt ý nghĩa*.

Công cụ để làm điều đó tên là **pgvector**. Nếu ngay bây giờ bạn chưa biết pgvector là gì, PostgreSQL là gì, hay "vector" nghĩa là gì — **hoàn toàn bình thường**. Phần 1 sẽ giải thích từng chữ một, từ con số 0, không giả định bạn biết trước bất cứ điều gì.

### Vấn đề thực tế mà nó giải quyết

Cơ sở dữ liệu truyền thống rất giỏi việc **tìm khớp chính xác**: "cho tôi bản ghi nào có tên đúng bằng chữ *áo thun*". Nhưng nó **không hiểu** rằng "áo phông", "T-shirt", "đồ mặc mùa hè" là những thứ gần nghĩa với "áo thun".

Con người thì hiểu ngay. Máy thì không. Khoảng trống đó chính là chỗ mà **vector search** (tìm kiếm bằng vector) lấp vào. Đây là nền tảng của ba thứ đang cực kỳ hot hiện nay:

- **Semantic search** (tìm kiếm ngữ nghĩa) — tìm theo ý, không theo từ khoá.
- **Recommendation** (gợi ý sản phẩm) — "người mua món này cũng thích món kia".
- **RAG** (viết tắt của *Retrieval-Augmented Generation*, tạm dịch "sinh văn bản có tra cứu bổ trợ") — kỹ thuật cho chatbot AI tra cứu tài liệu riêng của công ty trước khi trả lời, để nó không bịa. Bước "tra cứu" trong RAG chính là vector search.

### Học xong bạn sẽ làm được

- Giải thích được **embedding là gì** và vì sao "khoảng cách giữa hai vector" lại chính là "độ giống nhau về ý nghĩa".
- Thiết kế được **cấu trúc bảng** để lưu vector trong PostgreSQL (đây là mục tiêu *data modeling* của bài gốc).
- **Lưu, truy vấn và lọc** vector bằng SQL với 3 thước đo khoảng cách chính.
- Chọn và tinh chỉnh đúng loại **index** (HNSW, IVFFlat, DiskANN) và hiểu độ phức tạp tính toán của chúng (đây là mục tiêu *indexing techniques*).
- Phân tích **đánh đổi ở quy mô lớn** (từ triệu đến tỷ vector), tính được chi phí, và trả lời trôi chảy một câu hỏi phỏng vấn kiểu *system design* về vector search.

### Mạch kiến thức: basic → staff

- 🟢 **Basic** — Embedding là gì → vì sao khoảng cách = độ giống nhau → cài pgvector, tạo cột kiểu `vector`, viết câu tìm kiếm đầu tiên bằng toán tử `<->`.
- 🟡 **Intermediate** — Ba thước đo khoảng cách (L2 / cosine / inner product), thiết kế bảng thật, tạo index HNSW, viết một pipeline Python hoàn chỉnh, tìm kiếm có lọc điều kiện, và 3 lỗi kinh điển.
- 🔴 **Advanced** — Bên trong HNSW và IVFFlat hoạt động ra sao, độ phức tạp Big-O, "lời nguyền số chiều", tự viết lại thuật toán bằng Python, và các trường hợp biên (giới hạn 2000 chiều, chuẩn hoá vector).
- 🟣 **Staff** — Mở rộng lên hàng tỷ vector, chia nhỏ dữ liệu, nén vector để cắt chi phí, giám sát chất lượng tìm kiếm, khi nào **không** nên dùng pgvector, và một câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — Bảng từ khoá, ý cốt lõi, code cần thuộc lòng, câu hỏi phỏng vấn và các câu chốt "đắt giá".

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người **chưa biết gì**. Tôi sẽ đi rất chậm. Nếu bạn đã là dev backend quen Postgres, bạn có thể đọc lướt mục 1.1–1.3, nhưng đừng bỏ mục 1.5 (ví dụ chạy tay) — nó là chìa khoá để hiểu mọi thứ phía sau.

### 1.1. Bắt đầu từ nỗi đau: vì sao tìm kiếm truyền thống thất bại

Trước khi nói về giải pháp, ta phải *cảm* được vấn đề. Nếu không thấy nỗi đau, bạn sẽ không nhớ nổi liều thuốc.

Hãy tưởng tượng bạn đang làm một shop bán hàng online. Bạn có một **database** (cơ sở dữ liệu — nơi lưu trữ dữ liệu có tổ chức của ứng dụng, thay vì để rải rác trong file Excel).

Trong database đó có một **table** (bảng — giống một sheet Excel: có các cột và các dòng). Bảng `products` của bạn có 3 dòng như sau, mỗi dòng gọi là một **row** hoặc **record** (bản ghi — thông tin về một sản phẩm cụ thể):

| id | name |
|---|---|
| 1 | áo khoác lông vũ |
| 2 | khăn len |
| 3 | găng tay |

Bây giờ khách hàng gõ vào ô tìm kiếm: **"đồ giữ ấm mùa đông"**.

Ứng dụng của bạn sẽ chạy một **query** (câu truy vấn — một câu lệnh hỏi database "cho tôi dữ liệu thoả điều kiện này"). Query được viết bằng **SQL** (viết tắt của *Structured Query Language*, ngôn ngữ tiêu chuẩn để nói chuyện với database quan hệ). Câu SQL kinh điển cho ô tìm kiếm là:

```sql
SELECT name FROM products
WHERE name LIKE '%đồ giữ ấm mùa đông%';
```

Giải thích từng chữ trong câu trên, vì đây là câu SQL đầu tiên trong giáo trình:

- `SELECT name` — "lấy cho tôi cột `name`".
- `FROM products` — "từ bảng tên là `products`".
- `WHERE ...` — "chỉ lấy những dòng thoả điều kiện sau".
- `LIKE '%...%'` — **LIKE** là phép so khớp chuỗi ký tự; dấu `%` nghĩa là "bất cứ thứ gì cũng được ở vị trí này". Nên `'%đồ giữ ấm mùa đông%'` nghĩa là "tên sản phẩm có chứa đâu đó cụm chữ *đồ giữ ấm mùa đông*".

Kết quả trả về là: **0 dòng**. Khách hàng thấy trang trắng, thoát ra, và bạn mất một đơn hàng.

Vì sao? Vì không có sản phẩm nào **chứa đúng cụm ký tự** đó. `LIKE` so khớp từng **ký tự** (character — từng chữ cái một), nó tuyệt đối không hiểu *ý nghĩa*. Với `LIKE`, chữ "áo khoác lông vũ" và chữ "đồ giữ ấm mùa đông" là hai chuỗi ký tự khác nhau hoàn toàn, xa lạ như tiếng Việt với tiếng Nhật.

Nhưng bộ não bạn thì hiểu ngay: áo khoác lông vũ **chính là** đồ giữ ấm mùa đông. Vấn đề của chúng ta gói gọn trong một câu: **làm sao dạy máy tính cái hiểu biết đó?**

Đó chính xác là lý do **vector search** ra đời.

### 1.2. Analogy: tấm bản đồ ý nghĩa

Đây là hình ảnh quan trọng nhất trong toàn bộ giáo trình. Nếu bạn chỉ nhớ được một thứ, hãy nhớ cái này. Tôi sẽ dựng nó thật đầy đủ chứ không nói lướt.

Hãy tưởng tượng một **tấm bản đồ khổng lồ**. Nhưng thay vì đặt các thành phố lên đó, ta đặt **từ ngữ, câu chữ, sản phẩm** lên đó. Quy tắc đặt duy nhất và bất di bất dịch là:

> **Thứ nào giống nhau về ý nghĩa thì được đặt gần nhau trên bản đồ.**

Nếu ta tuân theo quy tắc đó, bản đồ sẽ tự động phân vùng:

- "áo khoác lông vũ", "áo phao", "khăn len", "găng tay" → tụ lại thành một cụm ở góc trên bên trái. Ta có thể gọi đó là "vùng đồ mùa đông", dù chẳng ai dán nhãn cho nó cả — chúng tự tụ lại vì nghĩa giống nhau.
- "kem chống nắng", "quần short", "áo ba lỗ" → tụ thành một cụm khác ở góc dưới bên phải, "vùng đồ mùa hè".
- "bàn phím cơ", "chuột máy tính" → tụ ở một vùng thứ ba, cách xa cả hai vùng trên, vì nó chẳng liên quan gì tới quần áo.

Bây giờ khách gõ **"đồ giữ ấm mùa đông"**. Ta làm hai bước:

**Bước 1:** Ta cũng đặt chính câu truy vấn đó lên bản đồ, theo đúng quy tắc trên. Nó sẽ rơi vào đâu? Rơi vào giữa vùng đồ mùa đông, dĩ nhiên — vì nghĩa của nó gần với "áo phao", "khăn len".

**Bước 2:** Ta hỏi bản đồ một câu duy nhất: **"những điểm nào nằm gần điểm này nhất?"** Trả lời: áo phao, khăn len, áo khoác lông vũ.

Xong. Đó chính là kết quả tìm kiếm ta muốn.

Bây giờ hãy để ý chỗ **khớp** giữa phép ví von và kỹ thuật, vì đây là điểm mấu chốt:

- Câu hỏi "tìm sản phẩm giống nghĩa nhất" — một câu hỏi về **ngôn ngữ**, rất khó cho máy tính —
- đã biến thành câu hỏi "tìm điểm gần nhất trên bản đồ" — một câu hỏi về **hình học**, mà máy tính giải được bằng phép trừ và phép căn bậc hai, cực kỳ dễ.

**Toàn bộ ngành vector search chỉ là việc thực hiện phép biến đổi đó.** Biến bài toán ngữ nghĩa thành bài toán khoảng cách.

Có hai chỗ mà phép ví von này hơi khác thực tế, tôi nói thẳng ra để bạn không bị bất ngờ sau:

1. Bản đồ thật **không phải 2 chiều** (dài × rộng) mà là hàng trăm đến hàng nghìn chiều. Bạn *không thể* hình dung nổi 1536 chiều — và không ai hình dung được cả, kể cả các nhà nghiên cứu. Nhưng may mắn thay, công thức tính khoảng cách vẫn y hệt, chỉ là nhiều số hạng hơn. Ta sẽ thấy tận mắt ở mục 1.5.
2. Ai vẽ bản đồ? Không phải pgvector. Đó là việc của một **mô hình AI** riêng biệt, tôi sẽ nói ngay dưới đây.

### 1.3. Từ bản đồ đến con số: embedding là gì

Trên một tấm bản đồ, mỗi điểm được xác định bằng **toạ độ** — ví dụ điểm A ở toạ độ (2, 9), nghĩa là đi ngang 2 và đi lên 9.

Toạ độ của một vật trên "bản đồ ý nghĩa" chính là thứ ta gọi là **embedding**.

Giải thích cho tận gốc:

- **Embedding** (từ động từ *embed* trong tiếng Anh, nghĩa đen là "nhúng vào", "gắn vào"). Trong ngữ cảnh này, embedding là **một dãy số biểu diễn ý nghĩa của một vật** — vật đó có thể là một đoạn chữ, một tấm ảnh, hay một đoạn âm thanh. Ta "nhúng" ý nghĩa của vật vào một dãy số.
  > Ví dụ cụ thể: câu "áo khoác lông vũ" có thể được biến thành dãy `[0.12, -0.98, 0.33, 0.07, ...]` gồm 1536 con số.
  > Analogy: cứ hình dung nó như một **tấm nhãn toạ độ** dán lên món hàng. Nhìn vào nhãn là biết món hàng nằm ở đâu trên bản đồ ý nghĩa.
- **Vector** — trong ngữ cảnh này, "vector" và "embedding" gần như là hai từ chỉ cùng một thứ. Nói chính xác thì *vector* là kiểu dữ liệu (một mảng số thực có độ dài cố định), còn *embedding* là ý nghĩa mà vector đó mang. Trong đời sống hàng ngày dân trong nghề dùng lẫn lộn hai từ, và điều đó không sao cả.
- **Dimension** (số chiều) — là **độ dài của dãy số**, tức là dãy đó có bao nhiêu con số. Dãy `[2, 9, 0]` có 3 dimensions. Model `text-embedding-3-small` của OpenAI sinh ra vector **1536 dimensions**, nghĩa là mỗi đoạn chữ biến thành một dãy 1536 con số.
  > Vì sao cần nhiều chiều đến thế? Vì ý nghĩa của ngôn ngữ rất phức tạp. Với 2 chiều bạn chỉ diễn tả được vài khía cạnh; với 1536 chiều, model có 1536 "trục" khác nhau để ghi lại các sắc thái (trang trọng hay suồng sã, vật thể hay khái niệm, tích cực hay tiêu cực, và hơn 1500 sắc thái khác mà không con người nào đặt tên nổi).
- **Embedding model** (mô hình sinh embedding) — là **một mô hình AI đã được huấn luyện sẵn**, có nhiệm vụ nhận vào một đoạn chữ và trả ra dãy số. Chính nó là "người vẽ bản đồ". Nó học được cách vẽ bằng cách đọc hàng tỷ câu chữ trên internet, và tự rút ra rằng "áo khoác" hay đi cùng ngữ cảnh với "mùa đông", nên nên đặt chúng gần nhau. Ví dụ các model phổ biến: `text-embedding-3-small` (OpenAI, gọi qua mạng, tính phí), `all-MiniLM-L6-v2` (chạy ngay trên máy bạn, miễn phí).

**Điểm cực kỳ quan trọng cần khắc cốt ngay từ bây giờ:** pgvector **không** sinh ra embedding. Nó chỉ *lưu* và *tìm* các dãy số đó. Việc biến chữ "áo phao" thành `[2, 9, 0]` là việc của embedding model, một hệ thống hoàn toàn riêng biệt. Đây là kiến trúc gồm hai mảnh mà người mới rất hay gộp nhầm làm một.

### 1.4. Bảng thuật ngữ còn lại của tầng Basic

Trước khi vào code, tôi trả nốt các món nợ thuật ngữ, để đến lúc gõ code bạn không phải đoán chữ nào cả.

- **PostgreSQL** (thường gọi tắt là **Postgres**) — một phần mềm quản trị cơ sở dữ liệu, mã nguồn mở và miễn phí, cực kỳ phổ biến trong ngành. Nó thuộc loại **RDBMS** (viết tắt của *Relational Database Management System*, hệ quản trị cơ sở dữ liệu quan hệ) — "quan hệ" ở đây nghĩa là dữ liệu được tổ chức thành các bảng có cột/dòng và các bảng có thể liên kết với nhau.
- **Extension** (phần mở rộng) — một gói phần mềm cài thêm vào Postgres để nó có thêm tính năng mới, giống như cài extension cho trình duyệt Chrome. Postgres được thiết kế để cho phép làm việc này, đó là một trong những lý do nó mạnh.
- **pgvector** — chính là một extension như vậy, mã nguồn mở, giúp Postgres biết lưu và tìm kiếm vector. Cụ thể nó thêm cho Postgres một **kiểu dữ liệu** mới tên là `vector` (bên cạnh các kiểu có sẵn như `text` cho chữ, `int` cho số nguyên).
- **Distance** (khoảng cách) / **similarity** (độ tương đồng) — thước đo mức độ "gần nhau" giữa hai vector. Quy ước cần nhớ: **khoảng cách nhỏ = giống nhau nhiều**. Hai khái niệm này ngược chiều nhau: khoảng cách càng nhỏ thì độ tương đồng càng lớn. Chi tiết ba loại khoảng cách nằm ở Phần 2.
- **Nearest Neighbor Search** (tìm kiếm hàng xóm gần nhất), viết tắt **NN search** — bài toán: cho một điểm truy vấn, hãy tìm các điểm gần nó nhất trong tập dữ liệu. Khi ta muốn lấy đúng `k` điểm gần nhất (ví dụ 10 sản phẩm giống nhất) thì gọi là **k-NN** (*k nearest neighbors*).
- **Similarity search** (tìm kiếm theo độ tương đồng) — tên gọi khác của cùng bài toán trên, nhìn từ góc độ ứng dụng thay vì góc độ toán học.
- **psql** — chương trình dòng lệnh chính thức để gõ câu SQL trực tiếp vào Postgres. Khi tôi nói "chạy câu SQL này", bạn có thể chạy trong psql hoặc bất kỳ công cụ nào khác (DBeaver, TablePlus, pgAdmin...).

### 1.5. Ví dụ chạy tay — tự tay tính để "thấy" cơ chế

Đây là mục quan trọng nhất của Phần 1. Ta sẽ **tạm bỏ qua model AI** và tự bịa ra embedding chỉ có **2 chiều**, để có thể tính bằng tay và nhìn thấy con số biến đổi. Toán học ở đây y hệt với 1536 chiều, chỉ khác là ít số hạng hơn nên ta viết ra giấy được.

Giả sử ta đặt 3 sản phẩm và 1 câu truy vấn lên bản đồ 2 chiều như sau:

| Object | Vector (x, y) | Ý nghĩa của toạ độ |
|---|---|---|
| Áo phao (A) | (2, 9) | nằm góc trên bên trái |
| Khăn len (B) | (3, 8) | ngay cạnh áo phao |
| Kem chống nắng (C) | (9, 1) | tít góc dưới bên phải |
| **Query: "giữ ấm mùa đông" (Q)** | **(2, 8)** | rơi vào giữa vùng đồ mùa đông |

Bây giờ ta dùng thước đo khoảng cách quen thuộc nhất: **Euclidean distance**, còn gọi là **L2 distance** — chính là "khoảng cách đường chim bay", cái mà bạn học ở phổ thông. Công thức cho 2 chiều:

```
distance = sqrt( (x1 - x2)² + (y1 - y2)² )
```

(`sqrt` là viết tắt của *square root*, tức căn bậc hai. Dấu `²` là bình phương, tức nhân số đó với chính nó.)

Ta tính khoảng cách từ điểm truy vấn Q tới từng sản phẩm, từng bước một:

**Q tới A (áo phao):**
```
hiệu theo trục x:  2 - 2 = 0     →  0² = 0
hiệu theo trục y:  8 - 9 = -1    →  (-1)² = 1
tổng:              0 + 1 = 1
căn bậc hai:       sqrt(1) = 1.00
```

**Q tới B (khăn len):**
```
hiệu theo trục x:  2 - 3 = -1    →  (-1)² = 1
hiệu theo trục y:  8 - 8 = 0     →  0² = 0
tổng:              1 + 0 = 1
căn bậc hai:       sqrt(1) = 1.00
```

**Q tới C (kem chống nắng):**
```
hiệu theo trục x:  2 - 9 = -7    →  (-7)² = 49
hiệu theo trục y:  8 - 1 = 7     →  7² = 49
tổng:              49 + 49 = 98
căn bậc hai:       sqrt(98) ≈ 9.90
```

Sắp xếp kết quả theo thứ tự tăng dần (gần nhất lên trước):

| Hạng | Sản phẩm | Khoảng cách |
|---|---|---|
| 1 | Áo phao | 1.00 |
| 2 | Khăn len | 1.00 |
| 3 | Kem chống nắng | 9.90 |

Hãy dừng lại một giây và nhìn kết quả này cho kỹ. Câu truy vấn "giữ ấm mùa đông" cho ra khoảng cách **1.00** tới áo phao và khăn len, nhưng **9.90** tới kem chống nắng — xa gần **10 lần**. Máy tính vừa "hiểu" đúng như trực giác con người, mà nó chỉ làm mấy phép trừ và phép căn.

Để ý luôn một chi tiết tinh tế: chú ý là chữ "phao" trong "áo phao" **không hề xuất hiện** trong câu truy vấn. Không có một ký tự nào trùng. Nếu dùng `LIKE` như mục 1.1 thì kết quả vẫn là 0 dòng. Nhưng ở đây ta tìm được — vì ta so **toạ độ ý nghĩa**, không so ký tự.

**Và đây là điều tôi muốn bạn nhớ nhất:** những gì bạn vừa làm bằng tay ở trên **chính xác** là những gì pgvector làm bên trong khi bạn gõ `ORDER BY embedding <-> query_vector`. Không có phép màu nào cả. Chỉ khác là nó làm với 1536 số hạng thay vì 2, và làm cho hàng triệu dòng thay vì 3 dòng. Toán không đổi.

### 1.6. Code "hello world" — câu vector search đầu tiên của bạn

Bây giờ ta viết lại đúng ví dụ trên bằng SQL thật. Đoạn này chạy được trong psql sau khi đã cài pgvector (cách cài dễ nhất: dùng Docker, xem ghi chú cuối file).

Tôi dùng vector 3 chiều thay vì 2 (thêm số 0 vào cuối) chỉ để bạn quen mắt với việc vector có thể dài tuỳ ý — kết quả không đổi.

```sql
-- BƯỚC 1: Bật extension pgvector.
-- Chỉ cần chạy MỘT LẦN cho mỗi database. Sau lệnh này Postgres mới "biết"
-- kiểu dữ liệu vector và các toán tử như <-> tồn tại.
-- "IF NOT EXISTS" nghĩa là: nếu đã bật rồi thì bỏ qua, đừng báo lỗi.
CREATE EXTENSION IF NOT EXISTS vector;

-- BƯỚC 2: Tạo bảng.
CREATE TABLE items (
    id        bigserial PRIMARY KEY,  -- khoá chính, tự động tăng 1,2,3... mỗi khi thêm dòng
                                      -- (PRIMARY KEY = cột định danh duy nhất cho mỗi dòng)
    name      text,                   -- cột chữ bình thường, lưu tên sản phẩm
    embedding vector(3)               -- ĐÂY LÀ CỘT MỚI: kiểu vector, mỗi ô chứa 3 con số.
                                      -- Con số 3 phải khớp đúng số chiều của embedding bạn dùng.
);

-- BƯỚC 3: Chèn dữ liệu vào bảng.
-- Vector được viết như một chuỗi, đặt trong nháy đơn, dạng '[a,b,c]'.
INSERT INTO items (name, embedding) VALUES
    ('áo phao',        '[2, 9, 0]'),
    ('khăn len',       '[3, 8, 0]'),
    ('kem chống nắng', '[9, 1, 0]');

-- BƯỚC 4: Similarity search — phần chính!
SELECT name,
       embedding <-> '[2, 8, 0]' AS distance   -- <-> là toán tử tính L2 distance.
                                               -- AS distance = đặt tên cho cột kết quả.
FROM items
ORDER BY embedding <-> '[2, 8, 0]'             -- sắp xếp tăng dần: gần nhất lên đầu
LIMIT 2;                                       -- chỉ lấy 2 dòng đầu = "2 hàng xóm gần nhất"
```

Kết quả chạy ra:

```
   name    | distance
-----------+----------
 áo phao   |        1
 khăn len  |        1
```

Giống hệt con số bạn vừa tính tay ở mục 1.5. Không sai một chữ số.

**Điểm mấu chốt cần khắc cốt:** ba thành phần `ORDER BY cot_vector <-> query_vector` cộng với `LIMIT k` gộp lại có nghĩa là **"cho tôi k bản ghi giống nhất"**. Đó là toàn bộ vector search ở dạng đơn giản nhất. Bạn vừa viết được câu vector search đầu tiên của mình rồi đấy.

### 1.7. Ba toán tử khoảng cách — biết mặt trước, đào sâu sau

pgvector cho bạn ba toán tử. Ở tầng Basic bạn chỉ cần **nhận mặt** chúng; Phần 2 sẽ giải thích cặn kẽ khi nào dùng cái nào.

| Toán tử | Tên đầy đủ | Dùng khi nào |
|---|---|---|
| `<->` | L2 / Euclidean distance | mặc định, an toàn, dễ hình dung |
| `<=>` | Cosine distance | **phổ biến nhất với embedding của văn bản** |
| `<#>` | Negative inner product | khi vector đã được chuẩn hoá, cần tốc độ tối đa |

Nếu bây giờ bạn thấy "cosine" hay "inner product" còn xa lạ thì không sao — mục 2.1 sẽ dựng lại từ đầu.

### 1.8. 🧩 [Ngoài bài gốc] — Hai điều một Staff Engineer nhắc bạn ngay từ ngày đầu

**Điều 1: pgvector KHÔNG sinh ra embedding.** Tôi đã nói ở mục 1.3 và tôi cố ý nhắc lại, vì đây là hiểu nhầm phổ biến nhất của người mới. Kiến trúc thực tế luôn có **hai mảnh**:

```
[Đoạn chữ]  →  Embedding model (OpenAI / sentence-transformers / Cohere)  →  [Dãy số]
                                                                                 ↓
                                                     pgvector chỉ nhận dãy số ở đây,
                                                     lưu nó, rồi tìm hàng xóm gần nhất.
```

Hệ quả thực tế: nếu kết quả tìm kiếm của bạn *dở về mặt ngữ nghĩa* (tìm "áo mùa đông" ra kem chống nắng), thủ phạm thường là **embedding model**, không phải pgvector. Đổi index sẽ không cứu được gì. Biết chỉ đúng thủ phạm là một kỹ năng debug quan trọng.

**Điều 2: Mặc định pgvector làm exact search, và bạn KHÔNG cần index ngay.** Hai từ mới ở đây:

- **Exact search** (tìm kiếm chính xác) — quét qua *toàn bộ* các dòng, tính khoảng cách với từng dòng, rồi lấy k dòng nhỏ nhất. Kết quả **luôn đúng tuyệt đối**. Cách này còn gọi là **brute force** (vét cạn).
- **Sequential scan** (quét tuần tự) — thuật ngữ của Postgres cho việc đọc lần lượt từ dòng đầu đến dòng cuối. Đây chính là cách nó thực hiện exact search khi chưa có index.

Với bảng nhỏ (khoảng dưới 10.000–50.000 dòng), quét tuần tự vẫn nhanh trong tầm mili-giây, và cho kết quả *chính xác 100%*. Index (Phần 2) chỉ cần khi dữ liệu lớn — và khi đó bạn sẽ phải **đánh đổi độ chính xác để lấy tốc độ**.

Rất nhiều người vội đánh index từ ngày đầu, rồi hoảng lên vì "sao kết quả sai so với lúc test". Nó không sai; nó *xấp xỉ*, và đó là do bạn tự chọn. Chi tiết ở mục 2.6.

### ✅ Self-check Phần 1

Trả lời trong đầu trước, rồi mở gợi ý ra so.

**Câu 1.** Vì sao `WHERE name LIKE '%đồ giữ ấm mùa đông%'` không giải quyết được bài toán của ta?
> *Gợi ý đáp án:* Vì `LIKE` so khớp **ký tự**, còn bài toán đòi hỏi so khớp **ý nghĩa**. Không sản phẩm nào chứa đúng cụm ký tự đó, nên trả về 0 dòng, dù về nghĩa thì "áo khoác lông vũ" rất liên quan.

**Câu 2.** Trong câu `ORDER BY embedding <-> '[2,8,0]' LIMIT 3`, con số khoảng cách càng nhỏ thì nghĩa là gì?
> *Gợi ý đáp án:* Càng **giống nhau về ý nghĩa**. Khoảng cách nhỏ = hai điểm nằm gần nhau trên bản đồ ý nghĩa. Vì thế ta luôn sắp xếp tăng dần và lấy các dòng đầu.

**Câu 3.** pgvector có tự biến chữ "áo phao" thành `[2,9,0]` không? Nếu không thì ai làm?
> *Gợi ý đáp án:* Không. Việc đó do **embedding model** (một mô hình AI riêng như OpenAI API hay sentence-transformers) làm. pgvector chỉ lưu dãy số và tìm hàng xóm gần nhất.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Phần này giả định bạn đã nắm: embedding là gì, khoảng cách nghĩa là gì, cột kiểu `vector`, và toán tử `<->`. Nếu còn lấn cấn chỗ nào, quay lại mục 1.5 đọc lại ví dụ chạy tay — nó là nền của mọi thứ dưới đây.

### 2.1. Ba thước đo khoảng cách — bên trong chúng thực sự làm gì

Đây là chỗ nhiều người chọn bừa rồi làm hỏng chất lượng tìm kiếm mà không hiểu vì sao. Ta đi từng cái một, có ví dụ số cụ thể.

Quy ước ký hiệu: ta có hai vector **a** và **b**. Ký hiệu `aᵢ` nghĩa là "phần tử thứ i của vector a". Ký hiệu `Σ` (đọc là *sigma*) nghĩa là "cộng dồn tất cả lại".

#### (1) L2 / Euclidean distance — toán tử `<->`

```
L2(a, b) = sqrt( Σ (aᵢ - bᵢ)² )
```

Chính là công thức bạn đã tính tay ở mục 1.5, chỉ viết ở dạng tổng quát cho d chiều. Nó đo **khoảng cách đường chim bay** giữa hai điểm.

Đặc điểm cần nhớ: L2 **nhạy với cả hướng lẫn độ dài** của vector. Hai vector chỉ về cùng một hướng nhưng một cái dài gấp đôi cái kia thì L2 vẫn coi chúng khá xa nhau.

#### (2) Cosine distance — toán tử `<=>`

Trước hết phải giải thích hai thứ mà công thức cosine dùng tới.

**Dot product** (tích vô hướng, còn gọi là *inner product*) — ký hiệu `a · b`. Cách tính cực đơn giản: nhân từng cặp phần tử tương ứng rồi cộng tất cả lại.

```
a · b = Σ aᵢ · bᵢ
```
> Ví dụ chạy tay: a = [2, 9], b = [3, 8] → `a · b = 2×3 + 9×8 = 6 + 72 = 78`.

**Norm / magnitude** (độ dài của vector) — ký hiệu `|a|`. Chính là khoảng cách từ gốc toạ độ tới điểm đó, tính bằng căn bậc hai của tổng bình phương.

```
|a| = sqrt( Σ aᵢ² )
```
> Ví dụ chạy tay: a = [2, 9] → `|a| = sqrt(4 + 81) = sqrt(85) ≈ 9.22`.

Giờ mới tới cosine. Ý tưởng của nó là: **chỉ quan tâm hai vector chỉ về cùng hướng hay không, kệ chúng dài ngắn thế nào.**

```
cosine_similarity(a, b) = (a · b) / (|a| · |b|)     # nằm trong khoảng [-1, 1]; 1 = cùng hướng hoàn toàn
cosine_distance(a, b)   = 1 - cosine_similarity     # nằm trong [0, 2]; 0 = giống hệt
```

pgvector trả về **cosine distance** (dòng thứ hai), để giữ đúng quy ước "nhỏ hơn = gần hơn", nhờ đó `ORDER BY` tăng dần vẫn cho ra kết quả giống nhất trước.

> Ví dụ chạy tay đầy đủ với a = [2, 9], b = [3, 8]:
> ```
> a · b            = 2×3 + 9×8 = 78
> |a|              = sqrt(2² + 9²) = sqrt(85)  ≈ 9.220
> |b|              = sqrt(3² + 8²) = sqrt(73)  ≈ 8.544
> cosine_similarity= 78 / (9.220 × 8.544) = 78 / 78.78 ≈ 0.990
> cosine_distance  = 1 - 0.990 = 0.010     → rất gần nhau
> ```

**Vì sao cosine là lựa chọn mặc định cho văn bản?** Vì với text, cái ta quan tâm là *hướng ngữ nghĩa*, chứ không phải "vector dài hay ngắn". Một đoạn văn 500 chữ và một câu 5 chữ nói về cùng một chủ đề thường cho ra vector cùng hướng nhưng khác độ dài; cosine coi chúng là giống nhau (đúng ý ta muốn), còn L2 thì đẩy chúng ra xa (không đúng ý ta muốn).

#### (3) Negative inner product — toán tử `<#>`

Chỉ là dot product ở trên, nhưng pgvector trả về **giá trị âm** của nó:

```
<#>  =  -(a · b)
```

Vì sao đảo dấu? Vì dot product càng **lớn** thì càng giống nhau — ngược với quy ước "nhỏ = gần" của `ORDER BY`. Đảo dấu xong thì mọi thứ nhất quán trở lại.

**Mẹo tối ưu quan trọng:** nếu bạn đã **normalize** (chuẩn hoá) mọi vector — tức chia mỗi vector cho độ dài của chính nó để độ dài mới bằng đúng 1 — thì mẫu số `|a| · |b|` trong công thức cosine trở thành `1 × 1 = 1`. Lúc đó **inner product tương đương hoàn toàn với cosine**, nhưng tính nhanh hơn vì bỏ được phép chia và hai phép căn. Đây là mẹo hay dùng ở production (môi trường chạy thật, phục vụ người dùng thật).

#### Quy tắc chọn nhanh

| Tình huống | Chọn |
|---|---|
| Embedding văn bản từ OpenAI / sentence-transformers / Cohere | `<=>` cosine |
| Bạn đã tự normalize vector về độ dài 1 | `<#>` inner product (nhanh hơn, kết quả như cosine) |
| Độ lớn của vector thực sự mang ý nghĩa (hiếm với text; hay gặp hơn ở dữ liệu số học/ảnh đặc thù) | `<->` L2 |

Nếu phân vân, chọn **cosine**. Đó là câu trả lời đúng trong khoảng 90% trường hợp thực tế với văn bản.

### 2.2. Data modeling: thiết kế bảng thật (mục tiêu #1 của bài gốc)

**Data modeling** (mô hình hoá dữ liệu) nghĩa là: quyết định xem dữ liệu của bạn nên được tổ chức thành bảng nào, cột nào, kiểu gì.

Trong thực tế bạn **không bao giờ** lưu mỗi cột vector trơ trọi. Bạn lưu vector **nằm ngay cạnh dữ liệu nghiệp vụ** — và đây chính là siêu năng lực của pgvector, ta sẽ thấy rõ ở mục 2.5.

```sql
CREATE TABLE documents (
    id          bigserial   PRIMARY KEY,
    tenant_id   bigint      NOT NULL,       -- xem giải thích "multi-tenant" bên dưới.
                                            -- NOT NULL = cột này bắt buộc có giá trị.
    title       text        NOT NULL,
    content     text        NOT NULL,       -- nội dung gốc, để hiển thị lại cho người dùng
    category    text,                       -- metadata: dùng để lọc kết quả
    created_at  timestamptz NOT NULL DEFAULT now(),
                                            -- timestamptz = thời điểm có kèm múi giờ.
                                            -- DEFAULT now() = tự điền giờ hiện tại khi thêm dòng.
    embedding   vector(1536)                -- SỐ 1536 PHẢI KHỚP số chiều của model bạn dùng!
);

-- Đánh index thông thường trên các cột metadata hay dùng để lọc.
-- Việc này RẤT quan trọng ở quy mô lớn, lý do xem mục 2.6 và 4.1.
CREATE INDEX ON documents (tenant_id, category);
CREATE INDEX ON documents (created_at);
```

Ba thuật ngữ mới trong đoạn trên:

- **Metadata** (siêu dữ liệu) — dữ liệu *mô tả về* dữ liệu chính. Ở đây `category`, `created_at`, `tenant_id` là metadata mô tả tài liệu; còn `content` mới là dữ liệu chính. Ta dùng metadata để lọc.
- **Multi-tenant** (đa khách thuê) — kiến trúc mà **một** hệ thống phục vụ **nhiều** khách hàng/tổ chức khác nhau, dữ liệu của họ nằm chung bảng nhưng phải được cách ly tuyệt đối. Cột `tenant_id` là "biển số" xác định dòng này thuộc về khách nào. Mọi câu query bắt buộc phải lọc theo nó, nếu không bạn sẽ để lộ dữ liệu khách A cho khách B — sự cố nghiêm trọng nhất trong loại kiến trúc này.
- **Index** (chỉ mục) — ta sẽ giải thích đầy đủ ở mục 2.3 ngay dưới. Tạm hiểu: một cấu trúc phụ giúp Postgres tìm nhanh mà không phải đọc hết bảng.

**Điểm mà một staff engineer sẽ nhấn mạnh ngay ở bước này:** con số `1536` **phải khớp chính xác** số chiều mà embedding model của bạn sinh ra. Nhét vector 768 chiều vào cột `vector(1536)` → Postgres báo lỗi ngay và từ chối. Nghe thì có vẻ là chi tiết vặt, nhưng nó dẫn tới một hệ quả kiến trúc rất nặng: **đổi embedding model đồng nghĩa với phải sinh lại toàn bộ embedding cho toàn bộ dữ liệu**. Ta sẽ quay lại chuyện này ở Phần 4, vì nó là một trong những quyết định đắt đỏ nhất của cả hệ thống.

### 2.3. Đánh index HNSW — làm cho nó nhanh

#### Trước hết: index là cái gì?

**Index** (chỉ mục) là một cấu trúc dữ liệu phụ mà database dựng thêm bên cạnh bảng, để tìm nhanh hơn.

> Analogy: hãy nghĩ tới **mục lục tra cứu ở cuối một cuốn sách dày**. Không có mục lục, muốn tìm chữ "photosynthesis" bạn phải lật từng trang từ đầu đến cuối (đó là *sequential scan*). Có mục lục, bạn tra chữ P, thấy ghi "trang 412", mở thẳng tới đó.
> Chỗ khớp với kỹ thuật: mục lục **tốn thêm giấy** (index tốn thêm dung lượng đĩa/RAM), và mỗi khi sách sửa nội dung thì **mục lục cũng phải sửa theo** (mỗi lần INSERT/UPDATE, Postgres phải cập nhật index → ghi dữ liệu chậm đi một chút). Đó là cái giá bạn trả để đọc nhanh.

Loại index quen thuộc nhất trong Postgres là **B-tree** (cây cân bằng), dùng cho số và chữ — đó chính là loại được tạo ra ở hai dòng `CREATE INDEX ON documents (...)` phía trên. Nhưng B-tree **không dùng được cho vector**, vì nó cần một thứ tự sắp xếp rõ ràng ("nhỏ hơn / lớn hơn"), mà với điểm trong không gian 1536 chiều thì khái niệm "nhỏ hơn" không tồn tại. Đó là lý do ta cần một họ index hoàn toàn khác.

#### ANN và recall — hai từ phải hiểu trước khi đánh index

- **ANN** (viết tắt của *Approximate Nearest Neighbor*, hàng xóm gần nhất **xấp xỉ**) — nhóm thuật toán chấp nhận **có thể bỏ sót** vài hàng xóm thật, để đổi lấy tốc độ nhanh hơn hàng nghìn lần. Ngược với ANN là *exact* (chính xác tuyệt đối, nhưng chậm).
- **Recall** (độ bao phủ / tỷ lệ tìm được) — thước đo chất lượng của ANN. Định nghĩa: trong `k` hàng xóm thật sự gần nhất, thuật toán tìm được bao nhiêu phần trăm.
  > Ví dụ cụ thể: bạn hỏi 10 kết quả gần nhất. Nếu so với đáp án chính xác thì thuật toán tìm đúng 9/10, còn 1 cái nó bỏ sót và thay bằng một kết quả xa hơn → **recall@10 = 90%**.

Nói cách khác: **index ANN không làm search "nhanh hơn miễn phí" — nó bán tốc độ cho bạn bằng cách lấy đi một chút độ chính xác.** Hiểu điều này là ranh giới giữa người dùng thư viện và người thiết kế hệ thống.

#### Tạo index HNSW

**HNSW** (viết tắt của *Hierarchical Navigable Small World*, tạm dịch "mạng lưới thế giới nhỏ có phân tầng, dẫn đường được") là loại index ANN được khuyên dùng mặc định năm 2026. Cơ chế bên trong của nó nằm ở mục 3.3; ở đây ta học cách dùng trước.

```sql
-- Cú pháp: chọn "ops class" KHỚP với toán tử khoảng cách mà bạn sẽ query.
--   vector_cosine_ops  ->  dùng với <=>
--   vector_l2_ops      ->  dùng với <->
--   vector_ip_ops      ->  dùng với <#>
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**Ops class** (viết đầy đủ là *operator class*, lớp toán tử) là cách bạn nói cho Postgres biết index này được xây dựng theo thước đo khoảng cách nào. Nó phải khớp với toán tử bạn dùng lúc query, nếu không index sẽ **bị bỏ qua hoàn toàn** — một lỗi im lặng rất khó phát hiện (xem mục 2.6).

Hai tham số lúc build:

- **`m`** — số kết nối tối đa mà mỗi điểm được nối tới các điểm khác trong mạng lưới (mặc định 16). Cao hơn → recall tốt hơn, nhưng index to hơn và build lâu hơn.
- **`ef_construction`** — độ "cẩn thận" khi xây mạng lưới; cụ thể là số ứng viên mà thuật toán xem xét khi quyết định nối điểm mới vào đâu (mặc định 64). Cao hơn → chất lượng mạng lưới tốt hơn → recall tốt hơn, nhưng build chậm hơn.

#### Tinh chỉnh lúc query — quan trọng ngang lúc build

```sql
BEGIN;                            -- mở một transaction (một nhóm lệnh chạy như một khối)
SET LOCAL hnsw.ef_search = 100;   -- mặc định là 40. Cao hơn = recall tốt hơn, nhưng chậm hơn.
SELECT id, title
FROM documents
ORDER BY embedding <=> '[... 1536 con số ...]'
LIMIT 10;
COMMIT;                           -- kết thúc transaction
```

**`ef_search`** là số ứng viên mà thuật toán giữ lại trong lúc dò đường tìm kiếm. Đây là **núm xoay recall–tốc độ ở thời điểm query** — thứ bạn có thể chỉnh mà không cần build lại index. Xoay lên: chính xác hơn, chậm hơn. Xoay xuống: nhanh hơn, dễ sót hơn.

> **🧩 Mẹo staff — một cái bẫy production thật:** hãy dùng `SET LOCAL` bên trong `BEGIN ... COMMIT`, **đừng** dùng `SET` toàn cục trên production.
>
> Lý do: `SET` thường sẽ gắn giá trị đó vào **connection** (kết nối tới database). Ứng dụng thật hầu như luôn dùng **connection pool** (bể kết nối — một tập kết nối được tạo sẵn và **dùng lại luân phiên** cho nhiều request khác nhau, để khỏi phải tạo kết nối mới mỗi lần, vì tạo kết nối rất tốn). Phổ biến nhất là **PgBouncer**.
>
> Hậu quả: bạn `SET hnsw.ef_search = 500` cho một truy vấn nặng của mình; kết nối đó được trả về pool; request tiếp theo của một người dùng khác vớ đúng kết nối ấy và bỗng dưng chạy chậm gấp 10 lần mà không ai hiểu vì sao. Đây là loại bug cực kỳ khó truy vì nó *ngẫu nhiên* và không tái hiện được trên máy dev. `SET LOCAL` chỉ có hiệu lực trong transaction hiện tại nên miễn nhiễm với vấn đề này.

### 2.4. Pipeline thực tế bằng Python (mục tiêu #3 của bài gốc)

Đây là đoạn code gần với công việc thật nhất trong giáo trình: sinh embedding rồi lưu vào Postgres rồi tìm kiếm, đủ vòng đời.

Vài thuật ngữ trước khi đọc code:

- **Library** (thư viện) — gói code do người khác viết sẵn mà bạn cài về dùng lại. **pip** là công cụ cài thư viện của Python.
- **sentence-transformers** — thư viện chạy embedding model **ngay trên máy bạn**, miễn phí, không cần khoá API. Lần chạy đầu nó sẽ tải model về (vài trăm MB).
- **psycopg** — thư viện Python để nói chuyện với PostgreSQL. Bản mới nhất là phiên bản 3, viết là `psycopg[binary]`.
- **NumPy array** — kiểu mảng số của thư viện NumPy, chuẩn de-facto cho tính toán số học trong Python. Embedding model trả về đúng kiểu này.
- **Cursor** (con trỏ) — đối tượng dùng để gửi lệnh SQL và nhận kết quả về.
- **Commit** — xác nhận "ghi thật" các thay đổi xuống database. Chưa commit thì thay đổi chưa chính thức.

```python
# Cài đặt: pip install sentence-transformers "psycopg[binary]" pgvector

from sentence_transformers import SentenceTransformer   # để sinh embedding
import psycopg                                          # để nói chuyện với Postgres
from pgvector.psycopg import register_vector            # cầu nối kiểu vector <-> numpy

# Model này sinh ra vector 384 chiều -> cột trong DB phải là vector(384). Phải khớp!
model = SentenceTransformer("all-MiniLM-L6-v2")

DOCS = [
    ("Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét", "outerwear"),
    ("Kem chống nắng SPF50 bảo vệ da khỏi tia UV mùa hè",    "skincare"),
    ("Khăn len dệt tay ấm áp cho những ngày lạnh",           "accessories"),
    ("Quần short thoáng mát đi biển ngày nắng",              "clothing"),
]

# Chuỗi kết nối có dạng: postgresql://user:mật_khẩu@địa_chỉ/tên_database
conn = psycopg.connect("postgresql://user:pass@localhost/shop")
register_vector(conn)   # dạy psycopg cách chuyển numpy array <-> kiểu vector của Postgres.
                        # Thiếu dòng này bạn sẽ phải tự nối chuỗi '[...]' bằng tay, rất dễ sai.

with conn.cursor() as cur:                 # "with" đảm bảo cursor tự đóng khi xong
    cur.execute("CREATE EXTENSION IF NOT EXISTS vector")
    cur.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id serial PRIMARY KEY,
            content text,
            category text,
            embedding vector(384)          -- khớp 384 chiều của all-MiniLM-L6-v2
        )
    """)

    # --- BƯỚC 1: sinh embedding cho từng tài liệu rồi chèn vào bảng ---
    for content, category in DOCS:
        vec = model.encode(content)        # biến chữ -> numpy array 384 số. ĐÂY là bước AI.
        cur.execute(
            "INSERT INTO products (content, category, embedding) VALUES (%s, %s, %s)",
            (content, category, vec),      # %s là "chỗ trống" được điền an toàn bởi thư viện.
                                           # Luôn làm thế này, ĐỪNG nối chuỗi bằng dấu +,
                                           # vì nối chuỗi mở đường cho lỗi bảo mật SQL injection.
        )
    conn.commit()                          # ghi thật xuống database

    # --- BƯỚC 2: đánh index HNSW ---
    # Lưu ý thứ tự: nên nạp dữ liệu xong RỒI mới build index. Build một lần trên
    # dữ liệu có sẵn nhanh hơn nhiều so với việc cập nhật index sau từng dòng INSERT.
    cur.execute("""
        CREATE INDEX IF NOT EXISTS products_emb_idx
        ON products USING hnsw (embedding vector_cosine_ops)
    """)
    conn.commit()

    # --- BƯỚC 3: tìm kiếm ---
    query_vec = model.encode("đồ mặc cho trời lạnh mùa đông")   # câu hỏi cũng phải
                                                                # đi qua ĐÚNG model đó
    cur.execute("""
        SELECT content, embedding <=> %s AS distance
        FROM products
        ORDER BY embedding <=> %s
        LIMIT 2
    """, (query_vec, query_vec))                                # truyền vector 2 lần: 1 cho
                                                                # cột distance, 1 cho ORDER BY

    for content, distance in cur.fetchall():                    # fetchall = lấy hết kết quả
        print(f"{distance:.4f}  |  {content}")                  # in khoảng cách làm tròn 4 số lẻ

conn.close()
```

**Kết quả kỳ vọng** (nhớ: khoảng cách nhỏ = gần nghĩa):

```
0.31xx  |  Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét
0.42xx  |  Khăn len dệt tay ấm áp cho những ngày lạnh
```

Hãy nhìn kỹ điều vừa xảy ra: câu truy vấn **"đồ mặc cho trời lạnh mùa đông"** không chứa chữ "áo khoác", không chứa chữ "khăn len", không chứa chữ "lông vũ". Với `LIKE` thì kết quả là con số 0 tròn trĩnh. Nhưng model *hiểu ý nghĩa* và trả về đúng hai món đồ mùa đông, đúng thứ tự mức độ liên quan. **Đó là semantic search thật sự**, và bạn vừa tự dựng nó bằng khoảng 30 dòng code.

Một chi tiết bắt buộc phải nhớ: **câu truy vấn phải đi qua đúng cùng một embedding model với dữ liệu.** Embedding từ hai model khác nhau nằm trên hai "bản đồ" hoàn toàn khác nhau; đem so khoảng cách giữa chúng thì con số vẫn ra (máy vẫn tính được), nhưng ý nghĩa là **rác**. Đây là lỗi âm thầm và nguy hiểm, vì không có thông báo lỗi nào cả — chỉ có chất lượng tìm kiếm tự nhiên tệ đi.

### 2.5. Filtered search — nơi pgvector toả sáng

**Filtered search** (tìm kiếm có lọc) nghĩa là: vừa tìm theo độ giống nhau, vừa áp thêm điều kiện nghiệp vụ. Trong đời thật thì **gần như mọi** tìm kiếm đều có lọc.

```sql
-- "Tìm 10 sản phẩm giống nhất với câu truy vấn, NHƯNG chỉ trong nhóm 'outerwear',
--  của khách hàng số 42, còn hàng, và được tạo trong 30 ngày gần đây."
SELECT id, content
FROM products
WHERE category   = 'outerwear'
  AND tenant_id  = 42
  AND in_stock   = true
  AND created_at > now() - interval '30 days'
ORDER BY embedding <=> '[...]'
LIMIT 10;
```

Một câu SQL. Một lần gọi. Xong.

Bây giờ hãy so với cách làm khi bạn dùng một **vector database chuyên dụng** (như Pinecone, Weaviate, Qdrant — các hệ thống chỉ chuyên lưu vector, tách rời khỏi database chính của bạn):

1. Gọi sang vector DB với vector truy vấn → nhận về một danh sách ID.
2. Gọi tiếp sang database nghiệp vụ với danh sách ID đó để lấy chi tiết và áp các điều kiện lọc.
3. Nếu sau khi lọc không đủ 10 kết quả → phải quay lại bước 1 xin thêm ứng viên. Vòng lặp thủ công.

So sánh cho rõ hai thứ đánh đổi:

- **Round-trip** (lượt đi–về qua mạng) — mỗi lần gọi sang một service khác đều tốn thời gian đi–về qua mạng, thường vài đến vài chục mili-giây. pgvector: 1 round-trip. Cách kia: ít nhất 2.
- **Failure mode** (kiểu hỏng) — mỗi hệ thống thêm vào là thêm một chỗ có thể sập, thêm một thứ phải giám sát, thêm một trạng thái có thể lệch (vector DB có dòng mà DB chính đã xoá). pgvector: dữ liệu vector và dữ liệu nghiệp vụ **luôn nhất quán tuyệt đối** vì chúng nằm trong cùng một transaction.

> **Đây là câu "flex" đắt giá nhất về pgvector, nên thuộc lòng để nói trong phỏng vấn:** *siêu năng lực của pgvector không phải là tốc độ, mà là vector sống chung nhà với dữ liệu nghiệp vụ — nên tìm kiếm có lọc chỉ là một câu SQL, một lượt đi về, một transaction.*

### 2.6. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh

#### Lỗi 1 — Overfiltering: "filter chặt xong bị thiếu kết quả"

Hiện tượng: bạn yêu cầu `LIMIT 10` nhưng chỉ nhận về 2 dòng, dù trong bảng rõ ràng có nhiều dòng thoả điều kiện.

Nguyên nhân — và đây là chỗ khó thật sự, người rất giỏi cũng vấp: thứ tự thực hiện bên trong không như bạn tưởng. Index HNSW đi tìm khoảng `ef_search` hàng xóm gần nhất **trước**, rồi Postgres mới áp mệnh đề `WHERE` lên nhóm ứng viên đó **sau**. Nếu điều kiện lọc của bạn hiếm (ví dụ `category` đó chỉ chiếm 0,5% dữ liệu), thì trong 40 ứng viên ban đầu có thể chỉ 1–2 cái sống sót qua bộ lọc.

Từ chuyên môn cho hiện tượng này là **overfiltering** (lọc quá tay).

Cách chữa — dùng **iterative index scan** (quét index lặp lại), tính năng chủ lực từ pgvector 0.8.0:

```sql
SET hnsw.iterative_scan = relaxed_order;  -- cho phép index tự động quét thêm
                                          -- cho tới khi đủ số kết quả yêu cầu
SET hnsw.max_scan_tuples = 20000;         -- trần an toàn: quét thêm tối đa 20k dòng rồi dừng,
                                          -- tránh một query xấu quét cả bảng
```

Hai lựa chọn giá trị: `strict_order` giữ đúng thứ tự khoảng cách tuyệt đối; `relaxed_order` cho phép thứ tự xê dịch chút ít nhưng nhanh hơn đáng kể, thường giữ được khoảng 95–99% chất lượng. Với đa số hệ thống thật, `relaxed_order` là lựa chọn đúng.

#### Lỗi 2 — Ops class không khớp toán tử query

Bạn đánh index bằng `vector_l2_ops` (dành cho `<->`) nhưng lại query bằng `<=>` (cosine). Chuyện gì xảy ra?

**Postgres im lặng bỏ qua index của bạn** và rơi về sequential scan. Không có lỗi. Không có cảnh báo. Chỉ có một query đáng lẽ chạy 5ms giờ chạy 5 giây — và nó chỉ lộ ra khi dữ liệu đã lớn, thường là vào lúc đông người dùng nhất.

Cách kiểm tra: dùng **`EXPLAIN ANALYZE`** — lệnh của Postgres cho biết nó *thực sự* thực thi query bằng cách nào.

```sql
EXPLAIN ANALYZE
SELECT id FROM documents ORDER BY embedding <=> '[...]' LIMIT 10;
```

Trong kết quả, hãy tìm dòng có chữ **`Index Scan using ..._hnsw...`** → index đang được dùng, tốt. Nếu bạn thấy **`Seq Scan`** → index đang bị bỏ qua, phải sửa. Thói quen `EXPLAIN ANALYZE` sau mỗi lần đánh index là thói quen phân biệt người cẩn thận với người đoán mò.

#### Lỗi 3 — Quên rằng ANN là *approximate*

Kịch bản kinh điển: bạn test trên máy dev với 500 dòng (chưa có index → exact → đúng 100%). Lên production 5 triệu dòng, có index HNSW. Người dùng báo "kết quả sai so với trước". Bạn hoảng.

Nhưng **không có gì sai cả** — đây là *by design* (đúng theo thiết kế). Bạn đã đồng ý đánh đổi recall lấy tốc độ ngay từ lúc gõ `CREATE INDEX`. Cách xử lý đúng gồm ba bước, theo thứ tự:

1. **Đo** recall hiện tại (cách đo cụ thể ở mục 4.3) để biết mình đang ở 85% hay 99%.
2. Nếu chưa đạt yêu cầu, **tăng `ef_search`** (rẻ, không cần build lại index) và đo lại.
3. Nếu vẫn chưa đủ, **tăng `m` và `ef_construction`** rồi build lại index (đắt hơn, tốn thời gian).

Điều tệ nhất bạn có thể làm là mò mẫm chỉnh tham số mà không đo gì cả.

### ✅ Self-check Phần 2

**Câu 1.** Embedding của bạn lấy từ OpenAI. Nên chọn `<->`, `<=>` hay `<#>`? Vì sao?
> *Gợi ý đáp án:* Chọn `<=>` (cosine), vì với văn bản ta quan tâm **hướng ngữ nghĩa** chứ không phải độ dài vector. Nếu bạn tự normalize mọi vector về độ dài 1 thì có thể dùng `<#>` để nhanh hơn mà kết quả tương đương.

**Câu 2.** Bạn đánh index `USING hnsw (embedding vector_cosine_ops)` nhưng query bằng `<->`. Chuyện gì xảy ra và làm sao phát hiện?
> *Gợi ý đáp án:* Ops class không khớp toán tử → Postgres **âm thầm bỏ qua index**, rơi về sequential scan, query chậm đi rất nhiều mà không báo lỗi. Phát hiện bằng `EXPLAIN ANALYZE`: nếu thấy `Seq Scan` thay vì `Index Scan using ..._hnsw...` là dính lỗi này.

**Câu 3.** Câu `WHERE category='rare_cat'` (nhóm này chỉ chiếm 0,3% dữ liệu) trả về 2 kết quả thay vì 10. Tên hiện tượng là gì và khắc phục thế nào?
> *Gợi ý đáp án:* **Overfiltering**. Index lấy ứng viên trước, `WHERE` lọc sau, nên gần hết ứng viên bị loại. Khắc phục: bật `hnsw.iterative_scan = relaxed_order` (pgvector 0.8+), kèm `hnsw.max_scan_tuples` làm trần an toàn.

**Câu 4.** Vì sao tuyệt đối không nên dùng `SET hnsw.ef_search = ...` toàn cục trên production?
> *Gợi ý đáp án:* Vì giá trị đó bám vào connection, mà connection được dùng lại luân phiên qua connection pool (PgBouncer) → nó "rò rỉ" sang query của người dùng khác, gây chậm/lỗi ngẫu nhiên rất khó truy. Dùng `SET LOCAL` trong `BEGIN...COMMIT` thay thế.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Ở đây ta chui xuống **bên dưới bề mặt**: các thuật toán này thực sự hoạt động thế nào, tốn bao nhiêu, và hỏng ở đâu. Tôi nói trước: mục 3.1 hơi nặng toán. Nhưng tôi sẽ giải thích cả những ký hiệu mà sách thường bỏ qua, nên bạn cứ đi chậm là được.

### 3.1. Vì sao bài toán này khó? — Big-O và lời nguyền số chiều

#### Trước hết: Big-O là gì?

**Big-O** (ký hiệu O lớn, ví dụ `O(n)`) là cách mô tả **thời gian chạy của một thuật toán tăng lên thế nào khi dữ liệu to ra**. Nó không cho biết chương trình chạy bao nhiêu giây — nó cho biết *xu hướng*, và đó mới là thứ quyết định hệ thống sống hay chết ở quy mô lớn.

- `O(n)` — thời gian tăng **tỷ lệ thuận** với số dòng. Dữ liệu gấp 10 lần → chậm gấp 10 lần.
- `O(log n)` — thời gian tăng **cực chậm**. Dữ liệu gấp 1000 lần → chỉ chậm thêm khoảng 10 lần. Đây là con số vàng mà mọi hệ thống lớn theo đuổi.
  > Analogy: tra từ điển giấy. Từ điển dày gấp đôi không làm bạn tra lâu gấp đôi — bạn chỉ cần lật thêm đúng một nhát nữa để chia đôi. Đó là `O(log n)`.
- `O(n · d)` — tỷ lệ thuận với cả số dòng `n` **và** số chiều `d`.

#### Chi phí thật của exact search

Ở Phần 1 ta tính khoảng cách tới **tất cả** các điểm. Đó là **brute-force exact search**, và chi phí của nó là `O(n · d)`.

Hãy quy ra con số cho thật cụ thể. Giả sử `n` = 100 triệu vector, `d` = 1536 chiều:

```
100.000.000 vector  ×  1536 chiều  ≈  153.600.000.000 phép nhân cho MỖI câu truy vấn
```

Hơn 150 tỷ phép tính cho một lần bấm nút tìm kiếm. Ngay cả với CPU hiện đại và tính toán song song, con số này là **bất khả thi** cho một hệ thống thời gian thực cần trả lời trong 100 mili-giây. Đó là lý do ta buộc phải tìm cách khác.

#### Vì sao không dùng cây phân hoạch như trong không gian 2D?

Câu hỏi rất hợp lý — trong không gian 2 hay 3 chiều, dân đồ hoạ và bản đồ có sẵn những cấu trúc như **kd-tree** (cây chia không gian thành các nửa) giải bài toán hàng xóm gần nhất rất gọn. Sao không dùng luôn?

Vì một hiện tượng có tên nghe rất kêu: **curse of dimensionality** (lời nguyền số chiều).

Nội dung của lời nguyền: khi số chiều `d` lớn (hàng trăm đến hàng nghìn), **mọi điểm trở nên gần như cách đều nhau**. Khoảng cách tới hàng xóm gần nhất và khoảng cách tới điểm xa nhất xích lại gần bằng nhau. Khái niệm "gần" mất dần ý nghĩa phân biệt.

> Analogy để cảm nhận: trên một đường thẳng (1 chiều), bạn chỉ có 2 hướng để trốn — trái hoặc phải, nên hàng xóm rất dễ xác định. Trên mặt phẳng (2 chiều) bạn có vô số hướng nhưng vẫn quản được. Trong không gian 1536 chiều, có quá nhiều "hướng" để một điểm có thể lệch đi, đến mức hai điểm ngẫu nhiên bất kỳ gần như luôn lệch nhau ở đâu đó — và kết quả là chúng đều cách nhau xấp xỉ như nhau.

Hệ quả kỹ thuật: kd-tree phải duyệt gần hết cây mới dám khẳng định đã tìm đúng, nên nó **không nhanh hơn brute force là bao**. Cây phân hoạch không gian mất tác dụng.

Và đó chính là lý do ngành này chuyển sang **ANN** — chấp nhận sai một chút để nhanh hơn hàng nghìn lần. Không phải vì lười, mà vì exact search ở quy mô lớn là **bất khả thi về mặt vật lý**.

Từ đây trở đi ta xem hai họ thuật toán ANN mà pgvector cung cấp.

### 3.2. IVFFlat — "chia quận, chỉ tìm trong vài quận gần"

**IVFFlat** = *Inverted File with Flat compression* (tệp đảo, lưu vector nguyên bản không nén).

#### Ý tưởng, kể bằng analogy

Bạn đứng ở Quận 1 và muốn tìm quán phở gần nhất trong thành phố. Cách ngu ngốc là đo khoảng cách tới từng quán phở trong cả thành phố (brute force). Cách khôn ngoan:

1. Chia thành phố thành các **quận**, mỗi quận có một điểm trung tâm.
2. So vị trí của bạn với **trung tâm các quận** — chỉ vài chục phép so.
3. Chọn ra 2–3 quận có trung tâm gần bạn nhất.
4. Chỉ đo khoảng cách tới các quán phở **trong những quận đó**. Bỏ qua toàn bộ phần còn lại của thành phố.

IVFFlat làm đúng như vậy trên không gian vector. Chỗ khớp với kỹ thuật:

- "Quận" trong thuật toán gọi là **list**, và số quận là tham số `lists`.
- "Trung tâm quận" gọi là **centroid** (tâm cụm) — một vector đại diện cho cả cụm.
- Việc chia quận được làm bằng thuật toán **k-means clustering** (phân cụm k-means): một thuật toán tự động nhóm các điểm gần nhau thành `k` cụm và tính tâm của mỗi cụm. Nó lặp đi lặp lại hai bước — gán mỗi điểm vào cụm có tâm gần nhất, rồi tính lại tâm — cho tới khi ổn định.
- Số quận được xem lúc tìm kiếm gọi là `probes` (số lần thăm dò).

#### Điểm cực kỳ quan trọng: IVFFlat cần dữ liệu để "học" trước

Vì phải chạy k-means để tìm centroid, IVFFlat **bắt buộc phải có sẵn dữ liệu trong bảng** lúc build index. Build index trên bảng rỗng rồi mới đổ dữ liệu vào → các centroid được tính trên hư không → phân hoạch vô nghĩa → recall thảm hại. Đây là điểm khác biệt lớn so với HNSW (không cần train).

#### Cú pháp

```sql
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 1000);       -- công thức gợi ý: số_dòng/1000 nếu ≤ 1 triệu dòng;
                           -- sqrt(số_dòng) nếu > 1 triệu dòng.

SET ivfflat.probes = 32;   -- gợi ý khởi đầu: sqrt(lists). Cao hơn = recall↑ nhưng chậm hơn.
```

#### Trade-off — và định nghĩa của chính từ này

**Trade-off** (sự đánh đổi) nghĩa là: **được cái này thì phải mất cái kia**; không có lựa chọn nào tốt hơn mọi mặt. Nhận diện và nói rõ trade-off là kỹ năng cốt lõi ở tầm senior/staff — người mới thường tìm "cách tốt nhất", người có kinh nghiệm hỏi "tốt nhất *cho ràng buộc nào*".

Với IVFFlat, `probes` chính là núm xoay trade-off:

- `probes = 1` → cực nhanh, nhưng dễ trượt: nếu hàng xóm thật sự nằm ở quận kế bên thì bạn không bao giờ thấy nó.
- `probes = lists` → quét hết mọi quận = đúng bằng exact search, nhưng mất sạch lợi ích tốc độ.

Chi phí truy vấn xấp xỉ bằng: `O(lists · d)` cho việc so với các centroid, **cộng** `O((n/lists) · probes · d)` cho việc quét bên trong các quận đã chọn.

#### Điểm yếu chí mạng của IVFFlat

1. **Hàng xóm nằm sát ranh giới quận dễ bị bỏ sót.** Một điểm ở rìa quận A có thể gần bạn hơn mọi điểm trong quận B, nhưng nếu bạn chỉ probe quận B thì nó vô hình.
2. **IVFFlat mang tính tĩnh.** Sau khi build, bạn thêm hàng triệu dòng mới → chúng vẫn bị gán vào các centroid cũ, phân hoạch dần lệch đi, recall tụt xuống một cách âm thầm. Cách chữa duy nhất là `REINDEX` (dựng lại index từ đầu) định kỳ. Với dữ liệu thay đổi liên tục, đây là gánh nặng vận hành thật sự.

### 3.3. HNSW — mạng lưới nhiều tầng có "đường cao tốc"

**HNSW** = *Hierarchical Navigable Small World* — phân tầng (hierarchical), đi lại được (navigable), trên một mạng lưới "thế giới nhỏ" (small world — thuật ngữ từ lý thuyết mạng, chỉ loại mạng mà hai điểm bất kỳ chỉ cách nhau vài bước nhảy, giống ý tưởng "sáu độ phân cách" giữa hai người bất kỳ trên Trái Đất).

#### Ý tưởng, kể bằng analogy giao thông

Hãy tưởng tượng một hệ thống giao thông nhiều tầng:

- **Tầng trên cùng:** rất ít nút, nhưng là **đường cao tốc liên tỉnh** — mỗi lần đi là nhảy được rất xa.
- **Tầng giữa:** nhiều nút hơn, là đường quốc lộ nối các thành phố.
- **Tầng dưới cùng:** chứa **mọi** nút, toàn ngõ nhỏ, mỗi bước đi rất ngắn.

Muốn đi từ Hà Nội tới một con hẻm cụ thể ở Sài Gòn, bạn không bò theo ngõ nhỏ suốt 1700 km. Bạn đi cao tốc để tới đúng vùng trước (vài bước nhảy dài), rồi tụt xuống quốc lộ, rồi cuối cùng mới rẽ vào từng ngõ nhỏ để tới đúng số nhà.

HNSW tìm kiếm y hệt như vậy: bắt đầu ở tầng cao nhất, nhảy những bước dài để về đúng vùng của không gian vector, rồi tụt dần xuống tầng thấp hơn để tinh chỉnh, cho tới tầng đáy thì tìm ra hàng xóm chính xác.

Chỗ khớp với kỹ thuật, nói bằng thuật ngữ chuẩn:

- **Graph** (đồ thị) — cấu trúc gồm các **node** (nút, ở đây mỗi node là một vector) nối với nhau bằng **edge** (cạnh, ở đây là "hàng xóm của nhau").
- Mỗi node được nối tối đa `m` hàng xóm gần nhất ở mỗi tầng — đó chính là tham số `m` bạn đã gặp ở mục 2.3.
- Tầng của mỗi node được chọn **ngẫu nhiên theo phân phối mũ**: đa số node chỉ nằm ở tầng đáy, càng lên cao càng ít node. Đây chính là thứ tạo ra hiệu ứng "cao tốc thưa, ngõ nhỏ dày".

#### Con số cần thuộc lòng

- **Độ phức tạp truy vấn: khoảng `O(log n)`.** Đây là con số vàng, gần như chắc chắn sẽ được hỏi trong phỏng vấn. Nó có nghĩa: từ 1 triệu lên 1 tỷ vector, thời gian tìm kiếm chỉ tăng vài lần chứ không tăng 1000 lần.
- **Độ phức tạp lúc build: khoảng `O(n · log n · m · d)`.** Chậm hơn IVFFlat rất nhiều và tốn RAM hơn hẳn, vì phải chèn từng node một và với mỗi node lại phải đi tìm hàng xóm.

Đó chính là trade-off cốt lõi của HNSW: **query nhanh và recall cao, đổi lại build chậm và ngốn RAM.**

### 3.4. Bảng so sánh index — câu hỏi phỏng vấn kinh điển

Trước bảng, một thuật ngữ mới cần giải thích: **DiskANN** — một họ thuật toán ANN được thiết kế để giữ phần lớn dữ liệu trên **SSD** (ổ cứng thể rắn) thay vì trong **RAM** (bộ nhớ trong, nhanh hơn nhiều nhưng đắt hơn nhiều). Trong hệ sinh thái Postgres, DiskANN đến qua extension **`pgvectorscale`** của Timescale.

| Tiêu chí | IVFFlat | HNSW | DiskANN (pgvectorscale) |
|---|---|---|---|
| Tốc độ / recall khi query | Tốt | **Rất tốt** | Rất tốt |
| Thời gian build index | **Nhanh** | Chậm | Trung bình |
| Tiêu thụ bộ nhớ | **Ít** | Nhiều (graph nằm trong RAM) | Ít (dựa vào SSD + nén) |
| Cần dữ liệu sẵn để build? | **Có** (train k-means) | Không | Có |
| Độ phức tạp query | ~O((n/lists)·probes·d) | **~O(log n)** | ~O(log n) |
| Thêm dữ liệu mới liên tục | Kém (phân hoạch lệch dần) | **Tốt** | Tốt |
| Hỗ trợ > 2000 chiều? | Không (phải dùng halfvec) | Không (phải dùng halfvec) | **Có (tới 16000)** |
| Khi nào chọn | RAM hạn chế, dữ liệu tĩnh, cần build nhanh | **mặc định 2026**: cần latency thấp + recall cao | dataset khổng lồ, muốn cắt chi phí RAM |

> **🧩 [Ngoài bài gốc]** DiskANN là "tay chơi thứ ba" mà rất ít ứng viên nhắc tới trong phỏng vấn, nên nhắc được là điểm cộng lớn. Điểm bán hàng của nó: giữ phần lớn đồ thị trên SSD kết hợp **binary quantization** (nén mỗi con số xuống còn 1 bit) → cùng một tập dữ liệu, index có thể nhỏ hơn HNSW gần một bậc độ lớn, và hỗ trợ vector tới 16000 chiều mà không cần thủ thuật gì. Câu chốt để nói: *"HNSW đánh cược rằng graph vừa RAM; khi cược đó không còn đúng, DiskANN là câu trả lời."*

### 3.5. Edge case: giới hạn 2000 chiều

**Edge case** (trường hợp biên) nghĩa là: tình huống nằm ở rìa của điều kiện bình thường, ít gặp nhưng khi gặp thì làm hệ thống hỏng theo cách bất ngờ. Người viết code tốt là người nghĩ tới edge case *trước khi* nó xảy ra.

Đây là cú vấp kinh điển nhất với các model hiện đại. Kiểu dữ liệu `vector` của pgvector **lưu** được tới 16000 chiều — nhưng index HNSW và IVFFlat chỉ **index** được tối đa **2000 chiều**:

```sql
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
-- ERROR: column cannot have more than 2000 dimensions for hnsw index
```

Model `text-embedding-3-large` của OpenAI sinh vector 3072 chiều → dính lỗi này ngay lập tức. Một staff engineer cần biết cả ba cách xử lý:

**Cách 1 — `halfvec` (half precision, độ chính xác một nửa).** Mỗi con số bình thường được lưu bằng 32 bit; `halfvec` lưu bằng 16 bit. Đổi lại, index được tới khoảng 4000 chiều, dung lượng giảm một nửa, và mất mát recall nhỏ tới mức thường không đo được trong thực tế:

```sql
CREATE INDEX ON docs USING hnsw ((embedding::halfvec(3072)) halfvec_cosine_ops);
--                                          ^^ dấu :: là cú pháp ép kiểu của Postgres
```

**Cách 2 — Matryoshka / giảm chiều.** **Matryoshka embeddings** (đặt theo tên búp bê gỗ Nga lồng nhau) là kỹ thuật huấn luyện model sao cho **thông tin quan trọng nhất dồn về đầu vector**. Nhờ đó bạn có thể cắt bỏ phần đuôi — ví dụ dùng 1024 số đầu trong 3072 số — mà chỉ mất rất ít chất lượng. Các model dòng `text-embedding-3` của OpenAI hỗ trợ sẵn điều này.
> Analogy: giống ảnh JPEG chất lượng 80% thay vì 100% — nhẹ hơn nhiều, mắt thường gần như không phân biệt được.

**Cách 3 — DiskANN qua `pgvectorscale`.** Hỗ trợ native tới 16000 chiều, không cần thủ thuật nào.

### 3.6. Edge case: chuẩn hoá vector và giá trị NULL

**Chuẩn hoá (normalize).** Nếu bạn normalize mọi vector về độ dài 1 lúc chèn vào để được dùng `<#>` cho nhanh, thì bạn **bắt buộc phải normalize cả vector truy vấn** lúc tìm kiếm. Quên bước này thì không có lỗi nào được báo — chỉ có kết quả sai lệch một cách âm thầm. Cách phòng: gói việc normalize vào **một hàm duy nhất** dùng chung cho cả đường ghi lẫn đường đọc, đừng viết hai chỗ.

**Giá trị NULL.** `NULL` trong SQL nghĩa là "không có giá trị". Những dòng có `embedding IS NULL` sẽ **không bao giờ** xuất hiện trong kết quả tìm kiếm — chúng vô hình. Tình huống này rất hay xảy ra khi pipeline sinh embedding bị lỗi ở vài dòng (gọi API thất bại, nội dung rỗng, quá dài...) và không ai để ý.

Cách phòng, nên coi là bắt buộc ở production:

```sql
-- Chạy định kỳ như một "cảm biến sức khoẻ" của hệ thống tìm kiếm
SELECT count(*) FROM documents WHERE embedding IS NULL;
```

Nếu con số này lớn hơn 0 và đang tăng, bạn có một mảng dữ liệu vô hình với người dùng. Nên có job tự động tìm và sinh lại embedding cho các dòng đó.

### 3.7. Tự implement để hiểu tận gốc

Cách chắc chắn nhất để thực sự hiểu một thuật toán là tự viết lại phiên bản đơn giản của nó. Hai đoạn dưới đây ngắn thôi nhưng đủ để bạn tự tin trả lời "bên trong nó làm gì".

#### (a) Exact k-NN bằng cosine với NumPy

Đây chính là thứ pgvector làm khi **chưa có index**:

```python
import numpy as np

def cosine_knn(query: np.ndarray, corpus: np.ndarray, k: int = 3):
    """Trả về (chỉ số, khoảng cách) của k vector gần nhất theo cosine distance.
    query:  mảng 1 chiều, kích thước (d,)      -> vector câu truy vấn
    corpus: mảng 2 chiều, kích thước (n, d)    -> n vector trong "kho"
    """
    # Chuẩn hoá query về độ dài 1. np.linalg.norm chính là công thức |a| ở mục 2.1.
    q = query / np.linalg.norm(query)

    # Chuẩn hoá từng dòng của corpus. axis=1 nghĩa là "tính theo từng dòng";
    # keepdims=True giữ nguyên hình dạng để phép chia broadcast đúng theo dòng.
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)

    # Sau khi cả hai đã có độ dài 1, dot product CHÍNH LÀ cosine similarity.
    # Toán tử @ trong NumPy là phép nhân ma trận; kết quả là mảng (n,).
    sims = c @ q

    # Đổi từ similarity (lớn = giống) sang distance (nhỏ = giống), khớp với <=> của pgvector.
    dists = 1.0 - sims

    # argsort trả về CHỈ SỐ sắp xếp tăng dần; lấy k cái đầu = k khoảng cách nhỏ nhất.
    idx = np.argsort(dists)[:k]
    return idx, dists[idx]


# Chạy thử với đúng dữ liệu ở mục 1.5
corpus = np.array([[2, 9, 0], [3, 8, 0], [9, 1, 0]], dtype=float)
idx, d = cosine_knn(np.array([2, 8, 0], dtype=float), corpus, k=2)
print(idx, d)   # -> [0 1] ... tức áo phao và khăn len, đúng như tính tay
```

#### (b) MiniIVF — tự dựng lại ý tưởng "chia quận"

```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def __init__(self, n_lists=8):
        self.n_lists = n_lists          # số "quận" sẽ chia

    def build(self, corpus):
        """Giai đoạn build index: chạy k-means để chia quận và tính tâm."""
        self.corpus = corpus
        km = KMeans(n_clusters=self.n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_   # toạ độ tâm của mỗi quận
        self.assign    = km.labels_            # mỗi vector thuộc quận số mấy
        return self

    def search(self, query, k=3, probes=2):
        """Giai đoạn truy vấn: chỉ tìm trong `probes` quận gần nhất."""
        # Bước 1: so query với các TÂM quận (rẻ: chỉ n_lists phép so, không phải n)
        cdist = np.linalg.norm(self.centroids - query, axis=1)
        near_lists = np.argsort(cdist)[:probes]      # chọn ra `probes` quận gần nhất

        # Bước 2: chỉ brute-force TRONG các quận đã chọn, bỏ qua toàn bộ phần còn lại
        mask = np.isin(self.assign, near_lists)      # đánh dấu các vector thuộc quận đó
        cand_idx = np.where(mask)[0]                 # lấy chỉ số của chúng
        d = np.linalg.norm(self.corpus[cand_idx] - query, axis=1)
        order = np.argsort(d)[:k]
        return cand_idx[order], d[order]
```

**Bài tập tôi thực sự khuyên bạn làm:** chạy `search()` với `probes=1` rồi với `probes=n_lists`, so kết quả với hàm `cosine_knn` exact ở trên. Bạn sẽ **tự mắt thấy** những hàng xóm bị bỏ sót khi `probes` nhỏ, và thấy chúng quay lại khi `probes` lớn. Đọc mười lần về trade-off recall–tốc độ không bằng nhìn thấy nó xảy ra một lần.

### 🧩 [Ngoài bài gốc] — Hai điều nữa ở tầng Advanced

**1. Build index song song và bộ nhớ.** HNSW build rất chậm, nhưng Postgres có thể build song song nhiều tiến trình. Hai tham số quyết định: `maintenance_work_mem` (lượng RAM dành cho thao tác bảo trì như build index — nếu graph không vừa lượng RAM này, Postgres phải đổ ra đĩa và chậm thảm hại) và `max_parallel_maintenance_workers` (số tiến trình song song). Đây là kiến thức vận hành mà rất ít người học lý thuyết biết tới.

**2. Bảo mật cũng là chuyện của index.** Như đã nói ở đầu file, pgvector 0.8.2 vá CVE-2026-3172 — lỗi tràn bộ đệm khi build HNSW song song, có thể làm rò dữ liệu từ bảng khác hoặc làm sập server. Bài học tổng quát ở tầm staff: **extension của database cũng là bề mặt tấn công**, và việc theo dõi CVE của các extension bạn dùng là một phần của công việc, không phải chuyện của riêng đội bảo mật.

### ✅ Self-check Phần 3

**Câu 1.** Nêu độ phức tạp truy vấn của HNSW và của brute-force bằng ký hiệu Big-O. Ý nghĩa thực tế của sự khác biệt đó là gì?
> *Gợi ý đáp án:* HNSW ≈ `O(log n)`; brute-force = `O(n · d)`. Nghĩa là khi dữ liệu tăng gấp 1000 lần, brute-force chậm đi 1000 lần còn HNSW chỉ chậm thêm khoảng chục lần — đó là khác biệt giữa hệ thống dùng được và không dùng được ở quy mô lớn.

**Câu 2.** Bạn có model 3072 chiều và cần đánh index HNSW. Nêu ít nhất hai cách xử lý.
> *Gợi ý đáp án:* (1) Ép sang `halfvec` rồi index (`(embedding::halfvec(3072)) halfvec_cosine_ops`). (2) Dùng Matryoshka để cắt vector xuống ví dụ 1024 chiều. (3) Dùng DiskANN qua `pgvectorscale`, hỗ trợ tới 16000 chiều.

**Câu 3.** Trong IVFFlat, tăng `probes` ảnh hưởng recall và latency thế nào? Vì sao hàng xóm nằm sát ranh giới list hay bị bỏ sót?
> *Gợi ý đáp án:* `probes` tăng → xét nhiều quận hơn → recall tăng nhưng latency tăng. Hàng xóm sát ranh giới bị sót vì nó thuộc về một quận không nằm trong nhóm `probes` quận được chọn, dù về khoảng cách thực nó rất gần điểm truy vấn — thuật toán không bao giờ nhìn tới nó.

**Câu 4.** Vì sao IVFFlat cần có dữ liệu sẵn lúc build còn HNSW thì không?
> *Gợi ý đáp án:* IVFFlat phải chạy k-means trên dữ liệu để tìm centroid — không có dữ liệu thì không có gì để phân cụm. HNSW xây đồ thị bằng cách chèn dần từng node và nối với hàng xóm hiện có, nên nó lớn lên tự nhiên cùng dữ liệu.

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Đây là phần phân biệt một senior engineer với một staff engineer. Senior trả lời câu hỏi *"dùng cái gì và dùng thế nào"*. Staff trả lời câu hỏi *"hệ thống này sẽ gãy ở đâu khi lớn lên, tốn bao nhiêu tiền, ai phải trực đêm vì nó, và khi nào ta nên bỏ lựa chọn hiện tại"*.
>
> Vài từ vựng nghề nghiệp cần giải thích trước:
> - **Staff Engineer** — cấp bậc kỹ sư cao hơn senior ở các công ty công nghệ lớn. Đặc trưng không phải là code giỏi hơn, mà là **phạm vi ảnh hưởng**: quyết định kiến trúc, đánh đổi dài hạn, và ảnh hưởng tới nhiều team.
> - **Production** — môi trường chạy thật, phục vụ người dùng thật. Đối lập với môi trường dev/staging (để thử nghiệm).
> - **SLA** (viết tắt của *Service Level Agreement*, cam kết mức dịch vụ) — lời hứa đo được về chất lượng, ví dụ "95% truy vấn phải trả lời dưới 100 mili-giây".
> - **On-call** — chế độ trực: khi hệ thống có sự cố ngoài giờ, ai đó bị điện thoại đánh thức lúc 3 giờ sáng. Mỗi hệ thống bạn thêm vào là thêm gánh nặng on-call cho một con người thật.

### 4.1. Scale: từ 1 triệu tới hàng tỷ vector

**Scale** (mở rộng quy mô) là câu hỏi: hệ thống hoạt động ra sao khi dữ liệu và lưu lượng tăng lên nhiều lần?

Câu hỏi cốt lõi ở tầm staff **không phải** "dùng index nào", mà là **"hệ thống này gãy ở đâu trước tiên khi lớn lên?"**

#### Dưới 10 triệu vector — đừng phức tạp hoá

Một node PostgreSQL duy nhất với index HNSW là **đủ, và thường là lựa chọn tốt nhất** xét về tổng thể vận hành. Truy vấn trả về trong khoảng vài mili-giây. Đây là khoảng 90% các use case RAG và tìm kiếm nội bộ doanh nghiệp trong thực tế.

Lời khuyên staff thẳng thắn: **đừng over-engineer** (đừng thiết kế thừa cho quy mô mà bạn chưa có). Chi phí lớn nhất của việc dựng sẵn kiến trúc cho một tỷ vector khi bạn mới có một triệu không phải là tiền server, mà là **thời gian và độ phức tạp** — thứ khiến team đi chậm lại suốt nhiều năm.

#### Bottleneck xuất hiện ở đâu — hãy tính bằng con số

**Bottleneck** (điểm nghẽn) là bộ phận chạm giới hạn trước tiên và kéo cả hệ thống chậm lại. Tìm đúng bottleneck quan trọng hơn tối ưu mọi thứ.

Với vector search, bottleneck số một hầu như luôn là **RAM**, vì đồ thị HNSW phải nằm gọn trong RAM thì mới nhanh. Hãy tính:

```
Mỗi con số trong vector      = 4 byte (kiểu float 32 bit)
Một vector 1536 chiều        = 1536 × 4 = 6.144 byte ≈ 6 KB
1 triệu vector               = 6 GB
100 triệu vector             = 600 GB   ← chỉ riêng dữ liệu thô, CHƯA tính đồ thị HNSW!
```

Đồ thị HNSW còn ngốn thêm đáng kể nữa cho các liên kết. Khi con số này vượt quá RAM của máy, hệ điều hành phải đọc từ đĩa, và **tail latency** (độ trễ ở nhóm chậm nhất) phình lên khủng khiếp. Đó chính là thời điểm bạn phải nghĩ tới **nén vector** (mục 4.2) hoặc **DiskANN**.

#### Bottleneck thứ hai: thời gian build index

HNSW build là `O(n · log n)`, có thể tốn hàng giờ ở quy mô lớn. Ba cách tăng tốc:

1. **Nạp toàn bộ dữ liệu xong rồi mới build index** — luôn nhanh hơn build trước rồi chèn dần.
2. **Tăng `maintenance_work_mem`** — lượng RAM mà Postgres được phép dùng cho các thao tác bảo trì như build index. Khuyến nghị: đặt đủ lớn để chứa **working set** (phần dữ liệu đang thực sự được đụng tới trong lúc build), nhưng đừng vượt quá 50–60% RAM của máy, kẻo hệ điều hành hết chỗ thở và mọi thứ chậm lại.
3. **Tăng `max_parallel_maintenance_workers`** — số tiến trình build song song.

#### Vertical scaling — làm trước, vì nó dễ

**Vertical scaling** (mở rộng theo chiều dọc) = làm cho **một máy** mạnh hơn: thêm RAM, thêm CPU, đổi SSD nhanh hơn.

Đây là thứ nên làm trước tiên, và nó đi xa hơn nhiều người tưởng — máy cloud hiện nay có thể có vài TB RAM. Đơn giản, không thay đổi kiến trúc, không sinh bug mới.

#### Horizontal scaling — khi hết đường vertical

**Horizontal scaling** (mở rộng theo chiều ngang) = **thêm nhiều máy**. Phức tạp hơn hẳn, nên chỉ làm khi thật sự cần. Ba công cụ chính:

**a) Read replicas** (bản sao chỉ đọc) — các máy giữ bản sao dữ liệu, chỉ phục vụ truy vấn đọc. Tìm kiếm là công việc **read-heavy** (chủ yếu là đọc, ít ghi), nên cách này hiệu quả tuyệt vời: cần chịu tải gấp 5 lần thì thêm 5 replica.

**b) Partitioning** (phân mảnh bảng) — chia một bảng lớn thành nhiều bảng con bên trong *cùng một* máy chủ, theo một quy tắc.

Điểm mà staff engineer nói khác người mới: **hãy phân mảnh theo *cách dữ liệu được truy cập*, không phải theo kích thước.** Nếu mọi truy vấn của bạn đều có `WHERE tenant_id = ?`, thì hãy phân mảnh theo `tenant_id`. Khi đó Postgres có thể **prune** (cắt bỏ) toàn bộ các mảnh không liên quan *trước khi* tìm kiếm, và bạn xây index vector riêng cho từng mảnh — nhỏ hơn, nằm gọn trong RAM, nhanh hơn nhiều.
> Ghi chú kỹ thuật: một bảng không phân mảnh trong Postgres có giới hạn mặc định 32 TB; bảng đã phân mảnh thì gần như không giới hạn.

**c) Sharding** (phân mảnh qua nhiều máy) — chia dữ liệu ra **nhiều máy chủ khác nhau**. Đây là bước phức tạp nhất. Công cụ: **Citus** (extension biến Postgres thành cụm phân tán), **PgDog**, hoặc **pgvectorscale**. Chỉ làm khi một máy không còn chứa nổi dữ liệu.

### 4.2. 🧩 [Ngoài bài gốc] Quantization — đòn bẩy chi phí lớn nhất

**Quantization** (lượng tử hoá) nghĩa là: **giảm số bit dùng để lưu mỗi con số**, chấp nhận mất một chút độ chính xác để đổi lấy dung lượng nhỏ hơn nhiều.

> Analogy: giống như lưu ảnh dưới dạng JPEG thay vì RAW. File nhỏ đi hàng chục lần; mắt thường gần như không thấy khác biệt; nhưng nếu phóng to soi kỹ thì đúng là có mất chi tiết.

| Kỹ thuật | Cắt được bao nhiêu dung lượng | Ảnh hưởng tới recall |
|---|---|---|
| `halfvec` (16 bit thay vì 32 bit) | ~50% | rất nhỏ, thường không đo thấy |
| Binary quantization (1 bit mỗi số) | tới ~32 lần | đáng kể — phải bù bằng bước re-rank |
| Matryoshka (cắt bớt số chiều) | tuỳ tỷ lệ cắt | nhỏ, nếu model có hỗ trợ |

#### Pattern kinh điển: two-stage retrieval (thô trước, tinh sau)

Đây là mô hình thiết kế mà bạn nên biết vì nó xuất hiện ở khắp nơi trong ngành tìm kiếm:

**Giai đoạn 1 — recall (quét rộng):** dùng index đã nén (ví dụ binary quantization) để lấy nhanh top 200 ứng viên. Rẻ, nhanh, hơi thô. Mục tiêu duy nhất: *đừng bỏ sót* thứ tốt.

**Giai đoạn 2 — precision (lọc tinh):** với 200 ứng viên đó, tính lại khoảng cách bằng vector gốc đầy đủ, và trộn thêm các tín hiệu nghiệp vụ (độ mới, độ phổ biến, quyền truy cập của người dùng, tồn kho). Chọn ra top 10 cuối cùng. Vì chỉ có 200 ứng viên nên bước này rất rẻ dù tính toán chính xác.

Kết quả: chi phí gần bằng giai đoạn 1, chất lượng gần bằng exact search. Đó là lý do mô hình này thắng thế. Và một mẹo triển khai: **làm cả hai bước ngay trong một câu SQL** để giữ tính nhất quán và tránh thêm round-trip.

### 4.3. Chi phí, độ trễ, độ tin cậy, giám sát

#### Cost (chi phí)

Postgres có sẵn pgvector trên hầu hết dịch vụ quản lý (Supabase, Neon, AWS RDS/Aurora, Google Cloud SQL) với giá từ vài chục USD/tháng. So với việc thêm một vector database chuyên dụng, bạn tiết kiệm không chỉ hoá đơn hàng tháng.

Giá trị lớn nhất — và đây là lập luận mà staff engineer đưa ra chứ ít ai khác nghĩ tới — là **bạn không thêm một hệ thống mới vào bức tranh**. Mỗi hệ thống mới nghĩa là: một **failure domain** mới (một chỗ nữa có thể sập độc lập), một bề mặt on-call mới, một thứ nữa phải backup, phải vá bảo mật, phải đào tạo người mới. Những chi phí này không nằm trên hoá đơn nhưng chúng có thật và chúng lớn.

#### Latency (độ trễ) — và vì sao đừng bao giờ đo trung bình

**Latency** là thời gian từ lúc gửi truy vấn tới lúc nhận được kết quả.

Hãy đo bằng **percentile** (bách phân vị) chứ đừng đo trung bình:

- **p50** = giá trị mà 50% truy vấn nhanh hơn nó (tức là trung vị).
- **p95** = 95% truy vấn nhanh hơn nó; 5% chậm hơn.
- **p99** = 99% nhanh hơn; 1% chậm hơn. Đây là nhóm "tail" (đuôi).

Vì sao trung bình lừa dối bạn? Vì nếu 99 truy vấn chạy 10ms và 1 truy vấn chạy 5 giây, trung bình là ~60ms — nghe rất ổn. Nhưng có một người dùng thật đã chờ 5 giây và có thể đã bỏ đi. Với hệ thống có hàng triệu truy vấn/ngày, "1% chậm" nghĩa là hàng chục nghìn người mỗi ngày. **Trung bình che giấu nỗi đau; percentile phơi bày nó.**

Với HNSW, tail latency phình lên khi `ef_search` quá cao, hoặc khi đồ thị không còn vừa RAM.

#### Monitoring recall — điểm phân biệt staff rõ nhất

Đây là ý quan trọng nhất của cả Phần 4, nên tôi nói thật kỹ.

Với ANN, **không có lỗi biên dịch, không có exception, không có alert nào** khi chất lượng tìm kiếm suy giảm. Hệ thống vẫn chạy, vẫn trả kết quả, vẫn nhanh — chỉ là kết quả ngày càng kém liên quan. Người dùng không báo lỗi, họ chỉ lặng lẽ dùng ít đi.

Cho nên bạn **phải chủ động đo recall**. Cách đo: so kết quả có index với kết quả exact (dùng làm **ground truth**, tức "sự thật nền" để đối chiếu).

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép Postgres KHÔNG dùng index -> buộc phải exact search.
                                    -- Kết quả này là ground truth (đáp án đúng tuyệt đối).
SELECT id FROM documents ORDER BY embedding <=> :q LIMIT 10;
COMMIT;

-- Sau đó chạy lại đúng query đó với index bật bình thường,
-- rồi tính tỷ lệ trùng nhau giữa hai danh sách ID: đó là recall@10.
```

Cách làm ở production: lấy mẫu vài trăm truy vấn thật mỗi ngày, chạy đối chiếu như trên trên một replica (để không ảnh hưởng người dùng), rồi vẽ **recall@10 theo thời gian** lên dashboard. Khi đường đó đi xuống, bạn biết mình cần tune lại hoặc reindex — **trước khi** kinh doanh cảm nhận được.

Đây là thứ tôi khuyên bạn nói ra trong phỏng vấn. Rất ít ứng viên nhắc tới, và nó cho thấy bạn từng vận hành thật chứ không chỉ đọc tài liệu.

#### Failure modes — các kiểu hỏng cần chuẩn bị trước

**Failure mode** nghĩa là "cách mà hệ thống hỏng". Liệt kê trước các kiểu hỏng là công việc thiết kế, không phải bi quan.

1. **Đổi phiên bản embedding model.** Vector cũ và vector mới nằm trên hai không gian khác nhau. Đem so với nhau vẫn ra số, nhưng kết quả là rác. Không có lỗi nào được báo. Đây là failure mode nguy hiểm nhất trong danh sách này.
2. **`VACUUM` trên bảng có index HNSW rất chậm.** (`VACUUM` là tiến trình dọn dẹp của Postgres, thu hồi không gian của các dòng đã xoá.) Với HNSW nó có thể kéo dài rất lâu. Cách xử lý thường dùng: reindex trước rồi mới vacuum.
3. **Pipeline sinh embedding lỗi âm thầm**, để lại các dòng `NULL` hoặc vector rác (xem mục 3.6).
4. **`SET` toàn cục rò rỉ qua connection pooler** (xem mục 2.3).
5. **Lỗ hổng bảo mật của chính extension** — như CVE-2026-3172 trong pgvector 0.8.0/0.8.1.

#### Ops (công cụ vận hành)

- **`pg_stat_statements`** — extension thống kê những câu query nào chạy nhiều nhất và tốn nhất. Bật nó lên là việc đầu tiên trên mọi Postgres production.
- **PgHero** — giao diện web xem nhanh tình trạng sức khoẻ database.

### 4.4. Khi nào NÊN và khi nào KHÔNG NÊN dùng pgvector

Biết giới hạn của lựa chọn của chính mình là dấu hiệu rõ nhất của tư duy staff. Người mới bảo vệ công cụ mình thích; người có kinh nghiệm nói rõ khi nào công cụ đó là lựa chọn sai.

**NÊN dùng pgvector khi:**

- Vector search là **một tính năng bên trong** một ứng dụng vốn đã chạy Postgres.
- Bạn cần **lọc và join** vector với dữ liệu nghiệp vụ (đây là lý do mạnh nhất).
- Quy mô dưới khoảng 10–50 triệu vector.
- Bạn muốn ít hệ thống, ít chi phí vận hành, và cần đảm bảo **ACID transaction** (nhóm tính chất đảm bảo dữ liệu luôn nhất quán kể cả khi có sự cố — vector và dữ liệu nghiệp vụ được ghi cùng lúc hoặc cùng không được ghi).

**KHÔNG NÊN (hãy cân nhắc Pinecone / Weaviate / Qdrant / Milvus / Vespa) khi:**

- Quy mô vượt 50 triệu tới hàng tỷ vector **và** cần phân tán trên nhiều vùng địa lý.
- Bạn cần **hybrid search** mạnh sẵn có (hybrid search = kết hợp tìm theo vector với tìm theo từ khoá truyền thống như BM25, rồi trộn điểm số lại).
- Tổ chức của bạn vốn không dùng Postgres.
- **Vector search chính là workload chính** của sản phẩm, không phải một tính năng phụ. Khi đó một hệ chuyên dụng sẽ tối ưu tốt hơn ở mọi khía cạnh.

> **Câu chốt tư duy nên thuộc lòng:** *"pgvector thắng ở sự đơn giản trong vận hành, không phải ở hiệu năng đỉnh. Hãy chọn nó khi vector search là một tính năng, chứ không phải là cả sản phẩm."*

### 4.5. Ảnh hưởng tổ chức & cách nói với người không kỹ thuật

Một staff engineer phải giải thích được quyết định kỹ thuật cho **stakeholder** (các bên liên quan — quản lý sản phẩm, sếp, phòng tài chính) bằng ngôn ngữ của *họ*, tức là bằng **rủi ro và chi phí**, không phải bằng thuật ngữ.

**Ví dụ cách nói với PM hoặc sếp:**

> *"Chúng ta không cần mua thêm và vận hành thêm một cơ sở dữ liệu mới. Tận dụng hệ thống Postgres sẵn có nghĩa là đội ngũ hiện tại đã biết cách vận hành nó, không phải tuyển thêm người, không thêm một chỗ nữa có thể sập lúc nửa đêm. Chi phí thấp hơn và rủi ro thấp hơn. Đổi lại, nếu sau này vượt khoảng 50 triệu bản ghi và cần phục vụ nhiều châu lục, chúng ta sẽ phải tính chuyện chuyển đổi — nhưng đó là bài toán của thành công, và ta giải nó khi tới đó."*

Chú ý cách nói trên hoàn toàn không có chữ HNSW, recall, hay embedding. Nó nói bằng ba thứ mà người nghe quan tâm: **tiền, rủi ro, và tốc độ ra sản phẩm**. Kèm theo là một lời thừa nhận trung thực về giới hạn — điều này xây dựng lòng tin tốt hơn mọi lời hứa hẹn.

**Ảnh hưởng tới roadmap (lộ trình sản phẩm):** quyết định *chọn embedding model nào* là quyết định kiến trúc **nặng hơn** cả quyết định chọn index. Lý do: đổi model đồng nghĩa với việc phải sinh lại embedding cho **toàn bộ** kho dữ liệu — tốn tiền gọi API, tốn thời gian, và có thể phải ngừng dịch vụ. Việc staff cần làm sớm:

1. Chốt model càng sớm càng tốt, sau khi đã thử nghiệm đàng hoàng.
2. **Version hoá** cột embedding (ví dụ thêm cột `embedding_v2` bên cạnh `embedding`) để có thể chuyển đổi dần.
3. Có kế hoạch **backfill** (sinh lại dữ liệu cũ theo cách mới) và **blue-green deployment** (chạy song song hai phiên bản rồi chuyển lưu lượng dần từ cũ sang mới, để có thể quay đầu nếu hỏng).

**Team topology (cấu trúc đội ngũ):** giữ vector trong Postgres nghĩa là đội backend hiện tại tự lo được toàn bộ. Không cần tuyển người biết một stack vector database riêng, không cần đào tạo lại, không tạo ra một "ốc đảo tri thức" mà chỉ một người trong công ty hiểu. Đây là một lập luận về mặt tổ chức mạnh không kém lập luận kỹ thuật — và là kiểu lập luận rất được đánh giá cao trong phỏng vấn cấp staff.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của một staff engineer

> **Đề bài:** *"Thiết kế hệ thống semantic search cho một sàn thương mại điện tử có 50 triệu sản phẩm, hỗ trợ lọc theo danh mục / giá / tồn kho, yêu cầu p95 dưới 100ms, kiến trúc multi-tenant."*

**Khung trả lời — 8 bước, và nhớ nói to các trade-off:**

**1. Làm rõ đề bài trước khi vẽ gì cả.** Đây là bước mà ứng viên hay bỏ qua và mất điểm ngay lập tức. Hãy hỏi: QPS (số truy vấn mỗi giây) là bao nhiêu? Catalog cập nhật thường xuyên thế nào? Mục tiêu recall là bao nhiêu? Ngân sách? Có yêu cầu đa vùng địa lý không?
> *Vì sao quan trọng:* staff engineer không thiết kế cho đề bài tưởng tượng. Việc hỏi trước cho thấy bạn biết rằng câu trả lời đúng phụ thuộc vào ràng buộc.

**2. Tầng embedding.** Chọn model khoảng 1024–1536 chiều để cân bằng recall và chi phí. Sinh embedding **bất đồng bộ** qua một hàng đợi (queue) mỗi khi sản phẩm được thêm hoặc sửa — tuyệt đối không gọi API embedding ngay trong luồng ghi dữ liệu, vì API bên ngoài chậm và có thể lỗi. Version hoá embedding ngay từ ngày đầu.

**3. Data model.** Một bảng `products` chứa cả metadata lẫn cột `vector`. **Partition theo `tenant_id`** để prune sớm. Đánh index B-tree thông thường trên `(category, price, in_stock)`.

**4. Index vector.** HNSW với `vector_cosine_ops`, `m` trong khoảng 16–32, tune `ef_search` theo SLA p95. Bật `hnsw.iterative_scan = relaxed_order` để chống overfiltering khi bộ lọc chặt.

**5. Truy vấn.** Một câu SQL duy nhất: lọc metadata (giúp prune partition) → `ORDER BY <=>` → `LIMIT`. Nếu cần độ chính xác cao hơn thì thêm bước re-rank hai giai đoạn.

**6. Scale và latency.** Read replica cho lưu lượng tìm kiếm. Tính nhẩm dung lượng ngay tại chỗ: 50 triệu × ~6KB ≈ **300GB** chỉ riêng vector thô → cân nhắc `halfvec` hoặc binary quantization, hoặc DiskANN, để vừa RAM và cắt chi phí. Rồi đo p95 thật chứ không đoán.
> *Mẹo phỏng vấn:* việc bạn **tự nhẩm ra con số 300GB** ngay tại chỗ gây ấn tượng mạnh hơn mọi thuật ngữ bạn liệt kê. Nó chứng tỏ bạn suy nghĩ bằng ràng buộc vật lý, không phải bằng buzzword.

**7. Vận hành.** Giám sát recall@10 (so định kỳ với exact), p99 latency, tỷ lệ dòng có embedding NULL. Có kế hoạch reindex. Dùng blue-green khi đổi model.

**8. Nói rõ khi nào ta nên bỏ pgvector.** Nếu tăng lên hàng trăm triệu sản phẩm, phục vụ nhiều châu lục, và cần hybrid search nặng → nói thẳng rằng lúc đó sẽ cân nhắc Milvus hoặc Vespa.
> **Đây là bước ghi điểm cao nhất.** Biết và nói ra giới hạn của lựa chọn của chính mình chính là dấu hiệu staff. Người phỏng vấn không tìm người bảo vệ công cụ đến cùng; họ tìm người biết công cụ nào phù hợp với ràng buộc nào.

### ✅ Self-check Phần 4

**Câu 1.** Vì sao đo latency trung bình là sai lầm, và nên đo cái gì thay thế?
> *Gợi ý đáp án:* Trung bình bị các truy vấn nhanh kéo xuống, che giấu nhóm chậm. Nên đo p50/p95/p99. Với hàng triệu truy vấn/ngày, "1% chậm" là hàng chục nghìn người thật bị ảnh hưởng mỗi ngày.

**Câu 2.** Vì sao phải chủ động giám sát recall, trong khi các chỉ số khác thì có alert tự động?
> *Gợi ý đáp án:* Vì recall suy giảm **không gây lỗi**. Hệ thống vẫn chạy, vẫn nhanh, chỉ là kết quả kém liên quan dần. Không có exception nào để bắt, nên phải chủ động đo bằng cách so với exact search làm ground truth.

**Câu 3.** Bạn có 50 triệu vector 1536 chiều. Ước lượng dung lượng thô và nêu hai cách giảm.
> *Gợi ý đáp án:* 50.000.000 × 1536 × 4 byte ≈ **300 GB** (chưa tính đồ thị HNSW). Giảm bằng: `halfvec` (cắt ~50%), binary quantization kèm re-rank (cắt tới ~32 lần), Matryoshka (cắt số chiều), hoặc chuyển sang DiskANN để dùng SSD thay RAM.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn. Nó cố tình viết ngắn và cô đặc — ngược hẳn với bốn phần trên. Nếu một dòng nào ở đây bạn đọc mà thấy mơ hồ, hãy quay lại đúng phần tương ứng phía trên đọc lại cho chậm.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ tiếng Anh | Định nghĩa một dòng |
|---|---|
| **Embedding** | Dãy số biểu diễn ý nghĩa của một object; gần nhau = giống nghĩa. |
| **Vector / dimension** | Mảng số thực độ dài cố định; số phần tử chính là số chiều. |
| **Embedding model** | Mô hình AI biến chữ/ảnh thành vector; **không phải** pgvector làm việc này. |
| **Similarity search / k-NN** | Tìm k vector gần nhất với vector truy vấn. |
| **Exact search / brute force** | Quét hết, tính khoảng cách với mọi dòng; recall 100% nhưng `O(n·d)`. |
| **ANN** | *Approximate Nearest Neighbor* — đổi recall lấy tốc độ. |
| **Recall** | % hàng xóm thật mà thuật toán tìm được; thước đo chất lượng của ANN. |
| **pgvector** | Extension cho Postgres, thêm kiểu dữ liệu `vector`. |
| **HNSW** | *Hierarchical Navigable Small World*; đồ thị nhiều tầng; query ~`O(log n)`; mặc định 2026. |
| **IVFFlat** | *Inverted File Flat*; chia list bằng k-means; build nhanh, cần dữ liệu để train. |
| **DiskANN / pgvectorscale** | Index dựa trên SSD + nén; tiết kiệm RAM, hỗ trợ tới 16000 chiều. |
| **Cosine / L2 / inner product** | Ba thước đo khoảng cách; toán tử `<=>` / `<->` / `<#>`. |
| **Ops class** | `vector_cosine_ops` v.v.; **phải khớp** toán tử query, nếu không index bị bỏ qua. |
| **m / ef_construction** | Tham số build HNSW: recall vs thời gian build và dung lượng. |
| **ef_search** | Tham số query HNSW: núm xoay recall vs tốc độ, chỉnh được lúc chạy. |
| **lists / probes** | Tham số tương ứng của IVFFlat (số quận / số quận được thăm dò). |
| **Overfiltering** | Filter chặt + ANN → trả về thiếu kết quả so với `LIMIT`. |
| **Iterative index scan** | Cơ chế chống overfiltering (pgvector 0.8+): `hnsw.iterative_scan`. |
| **halfvec / binary quantization / Matryoshka** | Ba kỹ thuật nén vector để cắt RAM và chi phí. |
| **Two-stage retrieval** | Lấy thô top-N bằng index nén, rồi re-rank chính xác. |
| **Curse of dimensionality** | Ở số chiều lớn mọi điểm gần như cách đều nhau → cây phân hoạch vô dụng. |
| **RAG** | *Retrieval-Augmented Generation*; vector search chính là bước "retrieval". |
| **Hybrid search** | Kết hợp vector search với tìm từ khoá (BM25) rồi trộn điểm. |

### 5.2. Core concepts — nếu chỉ được nhớ mười điều

1. Vector search = tìm hàng xóm gần nhất trên "bản đồ ý nghĩa"; **khoảng cách nhỏ = giống nghĩa**.
2. pgvector chỉ **lưu và tìm** vector; embedding do một model AI riêng sinh ra. Kết quả dở về ngữ nghĩa → thủ phạm thường là model, không phải index.
3. Mặc định pgvector làm **exact search với recall 100%** và không cần index. Index chỉ cần khi dữ liệu lớn.
4. Index ANN **bán tốc độ bằng cách lấy đi độ chính xác**. Đó là lựa chọn của bạn, không phải lỗi của nó.
5. **HNSW là mặc định**: query ~`O(log n)`, recall cao — đổi lại build chậm và tốn RAM.
6. **Ops class trong index phải khớp toán tử query**, nếu không Postgres âm thầm bỏ qua index.
7. Với embedding văn bản, dùng **cosine** (`<=>`); nếu đã normalize thì `<#>` nhanh hơn mà tương đương.
8. **Siêu năng lực của pgvector**: lọc và join vector với dữ liệu nghiệp vụ trong **một câu SQL, một round-trip, một transaction**.
9. **Overfiltering** là bẫy kinh điển khi filter chặt → bật `iterative_scan`.
10. Ở tầm staff, bottleneck là **RAM** và việc **giám sát recall**; đòn bẩy là quantization + partitioning + two-stage re-rank.

### 5.3. Mental model / analogy để nói cho trôi chảy

- **"Bản đồ ý nghĩa"** → giải thích embedding cho người mới trong đúng một câu. Dùng được cả với sếp không kỹ thuật.
- **"Đường cao tốc nhiều tầng"** → giải thích cách HNSW đi từ tầng cao xuống tầng thấp.
- **"Chia quận, chỉ vào vài quận gần"** → giải thích IVFFlat, kèm điểm yếu ở ranh giới quận.
- **"Thô trước, tinh sau"** → giải thích two-stage retrieval.
- **"Vector là một tính năng hay là cả sản phẩm?"** → kim chỉ nam chọn giữa pgvector và vector DB chuyên dụng.
- **"Mục lục cuối sách"** → giải thích index nói chung, kèm cái giá của nó (tốn giấy, phải cập nhật).

### 5.4. Code cần thuộc lòng

**(a) Vòng đời tối thiểu — interviewer rất hay bắt viết tại chỗ:**

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

**(b) Filtered search — câu "flex" thể hiện siêu năng lực của pgvector:**

```sql
SELECT id FROM items
WHERE tenant_id = 42 AND in_stock
ORDER BY embedding <=> :query_vec
LIMIT 10;
```
*(Nói kèm: "với vector DB tách rời thì đây là hai round-trip và một vòng lặp thủ công.")*

**(c) Exact k-NN cosine bằng NumPy — chứng tỏ hiểu bản chất chứ không chỉ biết gọi thư viện:**

```python
def cosine_knn(q, corpus, k=10):
    q = q / (q @ q) ** 0.5                              # chuẩn hoá query
    c = corpus / (corpus ** 2).sum(1, keepdims=True) ** 0.5   # chuẩn hoá từng dòng
    dist = 1 - c @ q                                    # cosine distance
    idx = dist.argsort()[:k]                            # k khoảng cách nhỏ nhất
    return idx, dist[idx]
```

**(d) Đo recall — đoạn ít ai nhớ, nhưng gây ấn tượng mạnh:**

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact search làm ground truth
SELECT id FROM docs ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so độ trùng với kết quả có index -> recall@10
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Khác nhau giữa HNSW và IVFFlat là gì?"**
> HNSW: đồ thị nhiều tầng, query ~`O(log n)`, recall cao, không cần train, xử lý dữ liệu thêm mới tốt — đổi lại build chậm và tốn RAM. IVFFlat: chia list bằng k-means, build nhanh, ít RAM — nhưng cần dữ liệu sẵn để train và kém với dữ liệu thay đổi liên tục. Mặc định hiện nay là HNSW.

**2. [BẪY] "Vì sao kết quả tìm kiếm 'sai' sau khi tôi thêm index?"**
> Không sai — ANN là *approximate* theo đúng thiết kế, bạn đã đổi recall lấy tốc độ. Cách xử lý đúng: **đo** recall trước, rồi tăng `ef_search` (rẻ), rồi mới tăng `m`/`ef_construction` và build lại (đắt). Đừng chỉnh mò mà không đo.

**3. [TRADE-OFF] "Tăng `ef_search` thì được gì mất gì?"**
> Recall tăng, nhưng latency tăng và CPU/RAM tốn hơn. Nó là núm xoay recall–tốc độ **ở thời điểm query**, chỉnh được mà không cần build lại index. Tune theo SLA p95.

**4. "Nên dùng cosine hay L2?"**
> Với embedding văn bản: cosine, vì ta quan tâm hướng ngữ nghĩa chứ không quan tâm độ dài vector. L2 khi độ lớn của vector thực sự mang ý nghĩa. Nếu đã normalize về độ dài 1 thì inner product tương đương cosine nhưng tính nhanh hơn.

**5. [BẪY] "Filter chặt mà trả về ít kết quả hơn `LIMIT`, vì sao?"**
> Overfiltering. Index lấy khoảng `ef_search` ứng viên **trước**, `WHERE` lọc **sau**, nên với bộ lọc hiếm thì gần hết ứng viên bị loại. Fix: `hnsw.iterative_scan = relaxed_order` kèm `max_scan_tuples` làm trần an toàn (pgvector 0.8+).

**6. [SCALE] "Một tỷ vector thì làm thế nào?"**
> Bắt đầu bằng việc nêu bottleneck: RAM (vector thô + đồ thị). Rồi: quantization (`halfvec`/binary) kèm two-stage re-rank; partition theo access pattern; DiskANN qua `pgvectorscale`; read replica cho tải đọc. Và thành thật: nếu cần đa vùng địa lý thì cân nhắc vector DB chuyên dụng.

**7. [TRADE-OFF] "Khi nào KHÔNG dùng pgvector mà chọn Pinecone?"**
> Khi vector search là workload chính chứ không phải một tính năng; khi vượt ~50 triệu vector và cần đa vùng; khi cần hybrid search mạnh sẵn có; hoặc khi tổ chức vốn không chạy Postgres. pgvector thắng ở sự đơn giản trong vận hành, không phải ở hiệu năng đỉnh.

**8. "Đổi embedding model thì chuyện gì xảy ra?"**
> Vector cũ và mới không cùng một không gian → phải sinh lại embedding cho toàn bộ kho dữ liệu. Cách làm: version hoá cột embedding, backfill dần, blue-green khi chuyển. Đây là quyết định kiến trúc **nặng hơn** cả việc chọn index.

**9. "Làm sao bạn biết chất lượng tìm kiếm đang xấu đi trên production?"**
> Câu này ít được hỏi trực tiếp nhưng cực kỳ ghi điểm khi bạn chủ động nói ra: recall suy giảm **không gây lỗi**, nên phải chủ động đo bằng cách so kết quả có index với exact search làm ground truth, lấy mẫu định kỳ trên replica, và vẽ recall@10 theo thời gian lên dashboard.

**10. [DATA MODELING] "Bạn thiết kế bảng thế nào cho multi-tenant?"**
> Vector nằm cùng bảng với metadata; cột `tenant_id NOT NULL` và mọi query đều lọc theo nó; partition theo `tenant_id` để prune sớm và giữ index vector của mỗi partition đủ nhỏ để vừa RAM; index B-tree trên các cột lọc thường dùng.

### 5.6. One-liner đắt giá — thả đúng lúc để ghi điểm

- *"pgvector's superpower isn't speed — it's that your vectors live next to your business data, so filtered search is one SQL query, one round-trip, one transaction."*
- *"ANN is a recall-for-speed trade; the real skill is monitoring recall in production, because there's no compile error when relevance quietly degrades."*
- *"Choose pgvector when vector search is a feature, not the product."*
- *"The heaviest architectural decision isn't the index — it's the embedding model, because changing it means re-embedding the entire corpus."*
- *"At a billion vectors the bottleneck is RAM, not CPU — that's when quantization and two-stage re-ranking stop being optional."*
- *"Partition by access pattern, not by size — so the planner can prune before the index even runs."*

---

## 📌 Ghi chú cuối — làm gì tiếp theo

**1. Thực hành ngay, đừng chỉ đọc.** Cách nhanh nhất để có một Postgres kèm pgvector là dùng **Docker** (công cụ chạy phần mềm trong "hộp" đóng gói sẵn, không cần cài đặt lằng nhằng lên máy):

```bash
docker run --name pgvec -e POSTGRES_PASSWORD=pass -p 5432:5432 -d pgvector/pgvector:pg17
```

Rồi chạy lại **toàn bộ** code trong Phần 1 đến Phần 3. Đọc lý thuyết mười lần không thay được một lần tự gõ `EXPLAIN ANALYZE` và nhìn thấy dòng `Index Scan using ..._hnsw...` hiện ra bằng chính mắt mình.

**2. Kiểm chứng phiên bản trước khi phỏng vấn.** pgvector phát triển rất nhanh (0.8.5 tính tới tháng 7/2026). Liếc CHANGELOG trên GitHub `pgvector/pgvector` — các con số như giới hạn số chiều, tên tham số, hay tính năng mới có thể đã đổi. Nhắc được một tính năng mới ra là dấu hiệu bạn theo dõi công nghệ chứ không học vẹt.

**3. Bài tập tự kiểm tra tổng hợp.** Hãy tự dựng lại toàn bộ ví dụ shop quần áo: sinh embedding cho 1000 sản phẩm, đo latency khi chưa có index, đánh HNSW rồi đo lại, sau đó **tự tính recall@10** so với exact search. Khi bạn nhìn thấy con số recall của chính mình bằng mắt, mọi thứ trong Phần 3 và Phần 4 sẽ ăn sâu theo cách mà đọc không bao giờ làm được.

**4. Chủ đề nên học tiếp, theo thứ tự:**

- **RAG end-to-end** — ghép vector search với một LLM để làm chatbot tra cứu tài liệu nội bộ.
- **Chunking strategies** — cách cắt tài liệu dài thành đoạn trước khi embed; ảnh hưởng tới chất lượng còn nhiều hơn cả việc chọn index.
- **Hybrid search** — kết hợp full-text search sẵn có của Postgres (BM25) với vector search, rồi trộn điểm bằng Reciprocal Rank Fusion.
- **Reranking models** — model chuyên xếp hạng lại top-N kết quả, chính là giai đoạn 2 của two-stage retrieval.
- **Evaluation của retrieval** — recall@k, MRR, nDCG: cách đo chất lượng tìm kiếm một cách khoa học. Đây là kỹ năng mà rất ít kỹ sư có, và nó là thứ tách biệt người đoán mò với người biết mình đang làm gì.
