# Truy vấn dữ liệu Vector trong pgvector: Toán tử khoảng cách & Ngưỡng lọc
### Giáo trình SIÊU DỄ HIỂU — giải thích tận gốc mọi thuật ngữ | Basic → Staff

> **Bài giảng gốc:** *"Perform vector queries, including using pgvector for data queries"* — Course 3, IBM Vector Database Fundamentals.
>
> **Vị trí trong series:** Đây là mảnh *"lấy dữ liệu ra cho ĐÚNG"*. Các bài trước đã dạy: tạo vector → đánh index cho nhanh → cài đặt → nạp dữ liệu. Bài này là bước cuối: **viết truy vấn**. Nếu bốn bài trước là xây kho và xếp hàng lên kệ, thì bài này là học cách đi tìm đúng món mình cần.
>
> **Cách đọc:** Mọi thuật ngữ được giải thích ngay lần đầu xuất hiện. Bài này có ba **ví dụ chạy tay bằng số thật** — hãy tự tính theo, vì đó là chỗ mọi thứ trở nên rõ ràng.
>
> **Ký hiệu:** 🧩 **[Ngoài bài gốc]** · ⚠️ **Chỗ khó** · 💡 **Mẹo thực chiến** · ❗ **[Đính chính]**

---

## ❗ Ba đính chính quan trọng — đọc trước khi vào bài

Bài giảng gốc có ba chỗ chưa chính xác. Cả ba đều là **câu hỏi bẫy kinh điển** trong phỏng vấn, nên ta nêu ngay từ đầu rồi giải thích kỹ ở trong bài.

**Một — `<=>` trả về cosine *distance*, không phải cosine *similarity*.** Bài gốc dùng cụm "cosine similarity" khi nói về toán tử này. Hai khái niệm ngược nhau: distance **càng nhỏ càng giống**, similarity **càng lớn càng giống**. Muốn có similarity thì phải tự đổi: `1 - (embedding <=> q)`. Mục 2.3 sẽ làm rõ.

**Hai — không có "ngưỡng vạn năng".** Bài gốc đưa ví dụ lọc `distance < 5`. Ý tưởng thì đúng, nhưng con số 5 **chỉ có nghĩa với đúng thước đo và đúng bộ dữ liệu đó**. Với cosine distance, giá trị chỉ nằm trong khoảng từ 0 tới 2, nên ngưỡng 5 sẽ **không lọc gì cả**. Mục 3.2 phân tích kỹ.

**Ba — có một cái bẫy về index mà bài gốc không nhắc.** Truy vấn vector nhanh được là nhờ dạng `ORDER BY khoảng_cách LIMIT k`. Nếu bạn chỉ viết `WHERE khoảng_cách < 0.3` mà **không** kèm `ORDER BY` và `LIMIT`, index thường **không được dùng** và database phải quét toàn bộ bảng. Mục 3.3 sẽ chỉ cách viết đúng, kèm một mẫu code lấy từ tài liệu chính thức của pgvector.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### 0.1. Một câu tóm tắt siêu đơn giản

**Bài này dạy bạn cách viết câu lệnh để hỏi cơ sở dữ liệu: "cho tôi những bản ghi giống với cái này nhất"** — và quan trọng hơn, cách bảo nó **"nhưng nếu chẳng có cái nào thật sự giống thì đừng trả về gì hết"**.

Vế thứ hai nghe có vẻ nhỏ nhặt, nhưng nó chính là ranh giới giữa một tính năng tìm kiếm dùng được và một tính năng trả về toàn kết quả rác. Phần lớn giáo trình này xoay quanh vế đó.

### 0.2. Vấn đề thực tế mà bài này giải quyết

Truy vấn vector viết **gần giống** SQL thông thường. Chính vì "gần giống" nên người ta chủ quan, và có bốn chỗ rất dễ sai mà lại **không báo lỗi gì cả**:

- Lấy thẳng cột vector ra xem → nhận về một mảng mấy trăm con số vô nghĩa với con người.
- Dùng nhầm toán tử hoặc nhầm thước đo → kết quả sai mà vẫn trông có vẻ hợp lý.
- Đặt ngưỡng lọc bằng cách đoán bừa → hoặc lọc sạch không còn gì, hoặc không lọc được gì.
- Viết câu truy vấn theo cách khiến index bị bỏ qua → hệ thống chạy đúng nhưng chậm dần theo thời gian, và không ai biết.

Bài này gỡ từng chỗ một.

### 0.3. Học xong bạn sẽ làm được gì

- Nêu được hai yêu cầu bắt buộc khi truy vấn vector, và vì sao không nên `SELECT` thẳng cột vector.
- Dùng đúng ba toán tử `<->`, `<#>`, `<=>` và biết khi nào chọn cái nào.
- Viết được truy vấn "tìm k bản ghi giống nhất với bản ghi X", nhớ loại chính nó ra, và hiển thị được điểm số độ giống cho người dùng.
- Lọc bằng **ngưỡng (threshold)** đúng cách: hiểu ngưỡng phụ thuộc vào thước đo và bộ dữ liệu, biết cách **hiệu chỉnh nó bằng dữ liệu thật** thay vì đoán, và tránh được cái bẫy mất index.
- Áp dụng các mẫu truy vấn ở quy mô lớn: lọc trước, kết hợp với tìm theo từ khóa, phân trang, giám sát.

### 0.4. Mạch kiến thức từ dễ tới khó

- 🟢 **Basic** — hai yêu cầu bắt buộc → vì sao đừng xem vector thô → phân biệt tìm theo từ khóa và tìm theo ngữ nghĩa → tự tính khoảng cách bằng tay → câu truy vấn đơn giản nhất.
- 🟡 **Intermediate** — ba toán tử và khi nào dùng cái nào → mẫu "tìm giống một bản ghi có sẵn" → hiển thị điểm số độ giống → ba lỗi kinh điển.
- 🔴 **Advanced** — đào sâu ngưỡng lọc: vì sao không có số vạn năng, cách hiệu chỉnh bằng dữ liệu, bẫy mất index và cách viết đúng theo tài liệu chính thức, các trường hợp biên.
- 🟣 **Staff** — mẫu truy vấn ở quy mô lớn, hiệu chỉnh ngưỡng một cách khoa học, đánh đổi giữa "đúng" và "nhanh", giám sát, ảnh hưởng tới trải nghiệm sản phẩm, một câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — thuật ngữ, ý cốt lõi, code cần thuộc, câu hỏi phỏng vấn kèm gợi ý trả lời.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần dài nhất và chậm nhất. Viết cho người chưa từng gõ một câu truy vấn vector nào.

### 1.1. Nhắc lại nền móng trong ba đoạn

Nếu bạn đã đọc các giáo trình trước thì đây là phần ôn nhanh; nếu chưa thì đây là tất cả những gì bạn cần để hiểu bài này.

**Vector** ở đây là một **dãy số có thứ tự**, ví dụ `[0.12, -0.85, 0.33]`. Số lượng phần tử trong dãy gọi là **số chiều (dimension)**. Các mô hình AI thực tế thường tạo ra vector 384, 768 hoặc 1536 chiều.

**Embedding** (nghĩa đen: *sự nhúng vào*) là kết quả của việc biến một nội dung — một câu văn, một bức ảnh — thành vector, sao cho **những nội dung gần nhau về ý nghĩa sẽ cho ra hai vector gần nhau về mặt toán học**. Nhờ vậy, câu hỏi "hai nội dung này có giống nhau không" biến thành phép toán "hai dãy số này có gần nhau không".

**Model** (dịch: *mô hình*) là chương trình AI chịu trách nhiệm tạo ra embedding. Nó chính là thứ quyết định "bản đồ ý nghĩa" trông như thế nào — và điều đó dẫn thẳng tới mục tiếp theo.

### 1.2. Hai yêu cầu bắt buộc khi truy vấn vector

Trước khi viết dòng code nào, phải nắm hai điều kiện này. Vi phạm là kết quả loạn.

**Yêu cầu một — cùng một mô hình.**

Embedding đang nằm trong database và embedding của câu người dùng vừa gõ **phải do cùng một mô hình, cùng một phiên bản** sinh ra.

Vì sao? Vì mỗi mô hình tự dựng "bản đồ ý nghĩa" riêng của nó. Hai mô hình khác nhau đặt cùng một khái niệm ở hai vị trí hoàn toàn khác nhau trên hai tấm bản đồ khác nhau.

Cứ hình dung thế này: bạn có tọa độ của một quán cà phê lấy từ bản đồ Hà Nội, rồi đem tra tọa độ đó trên bản đồ Paris. Bạn vẫn ra được một điểm — bản đồ Paris không báo lỗi — nhưng điểm đó chẳng liên quan gì tới quán cà phê bạn muốn tìm. Phép ví von khớp ở đúng chỗ nguy hiểm nhất: **hệ thống vẫn chạy, vẫn trả về kết quả, chỉ là kết quả vô nghĩa**.

**Yêu cầu hai — cùng số chiều.**

Để tính khoảng cách giữa hai vector, chúng phải có cùng độ dài. Bạn không thể trừ một dãy 512 số cho một dãy 1536 số.

Số chiều do mô hình quyết định. Nên **đổi mô hình nghĩa là đổi số chiều, nghĩa là phải sinh lại toàn bộ vector cho cả kho** — quá trình gọi là **re-embed**, và nó tốn kém cả về tiền lẫn thời gian.

⚠️ **Chỗ khó — hai kịch bản, và kịch bản thứ hai mới đáng sợ.** Nếu hai mô hình khác số chiều, lệnh `INSERT` báo lỗi ngay và bạn phát hiện sớm — đó là may mắn. Nhưng nếu hai mô hình **tình cờ cùng số chiều** (chuyện rất hay xảy ra, vì 1536 là con số phổ biến), database sẽ vui vẻ nhận, không báo gì, và kết quả tìm kiếm sai một cách âm thầm suốt nhiều tháng trời.

💡 **Cách phòng:** lưu **tên và phiên bản mô hình** vào một cột bên cạnh mỗi vector. Nghe thừa thãi lúc mới làm, nhưng nó là thứ cứu bạn khi cần biết hàng nào cần sinh lại.

### 1.3. Đừng `SELECT` thẳng cột vector — nó vô nghĩa với con người

Đây là phản xạ đầu tiên của mọi người khi mới học, và nó dẫn tới một khoảnh khắc bối rối:

```sql
SELECT embedding FROM reviews LIMIT 1;
-- Kết quả: [0.0123, -0.87, 0.44, 0.019, -0.33, ... còn 507 số nữa ...]
```

Bạn nhận về một mảng mấy trăm con số. Nhìn vào đó, bạn **không rút ra được bất kỳ thông tin gì**. Không biết review đó nói về cái gì, không biết nó tích cực hay tiêu cực, không biết gì hết.

Lý do rất căn bản: **từng con số riêng lẻ trong vector không mang ý nghĩa nào mà con người đọc được**. Chiều thứ 47 không phải là "độ tích cực", chiều thứ 300 không phải là "chủ đề". Ý nghĩa chỉ tồn tại trong **mối quan hệ giữa các vector với nhau** — tức là trong khoảng cách.

Đây là chỗ cần đổi tư duy, và nó chi phối toàn bộ phần còn lại của bài: **cái bạn muốn không phải là *xem* vector, mà là *so sánh* nó.** Vì vậy mọi truy vấn vector đều xoay quanh **khoảng cách**, chứ không xoay quanh bản thân cột vector.

Cột vector là **nguyên liệu để tính toán**, không phải **thông tin để hiển thị**. Cái bạn hiển thị cho người dùng luôn là các cột bình thường: tiêu đề, nội dung, giá, ảnh.

### 1.4. Hai loại "tìm kiếm văn bản" — đừng nhầm lẫn

Bài gốc nhắc tới cả `tsquery` lẫn pgvector trong cùng một mạch, rất dễ gây rối. Ta phân biệt dứt khoát.

| | **`tsquery` trên `tsvector`** | **pgvector (`<=>` và các toán tử khác)** |
|---|---|---|
| Thuộc về | PostgreSQL lõi — có sẵn | Extension pgvector — phải cài |
| Tìm theo | **Lexeme** — từ đã chuẩn hóa | **Ngữ nghĩa** — embedding |
| Hợp cho | Tìm chính xác theo từ khóa | Tìm theo ý nghĩa gần |
| "red" có tìm ra "reddish"? | Có (cùng gốc từ) | Có (nếu nghĩa gần) |
| "ô tô" có tìm ra "xe hơi"? | **Không** | **Có** |
| Cần mô hình AI? | Không | Có |

**Lexeme** (dịch: *đơn vị từ vựng*) là một từ đã được rút về dạng gốc chung, để mọi biến thể của nó gộp làm một — "running", "ran", "runs" đều về `run`. Đây là đơn vị so khớp của tìm kiếm toàn văn, và giáo trình full-text search đã nói kỹ.

Ví dụ `tsquery` mà bài gốc dùng: kiểm tra xem câu *"a red mat costs as much as a blue mat"* có chứa cả `red` lẫn `mat` không.

```sql
-- Yêu cầu "red" đứng NGAY TRƯỚC "mat"
SELECT to_tsvector('english', 'a red mat costs as much as a blue mat')
    @@ to_tsquery('english', 'red <-> mat');
-- Kết quả: t (true) — vì trong câu có cụm "red mat" liền nhau
```

⚠️ **Chỗ khó — một ký hiệu, hai nghĩa hoàn toàn khác nhau.** Trong `tsquery`, ký hiệu `<->` nghĩa là **FOLLOWED BY** (từ này đứng ngay trước từ kia). Nhưng trong pgvector, `<->` là **toán tử tính khoảng cách Euclid**. Cùng ba ký tự, hai thế giới khác nhau — cách phân biệt là nhìn xem hai bên của nó là văn bản hay là vector.

Phần còn lại của bài này chỉ nói về **pgvector**.

### 1.5. Ví dụ chạy tay #1 — tự tính khoảng cách, và thấy hai thước đo cho hai kết quả khác nhau

Đây là ví dụ quan trọng nhất trong Phần 1. Hãy tự tính theo bằng giấy bút, vì nó làm sáng tỏ một chuyện mà đọc lý thuyết mãi không hiểu.

Ta dùng vector **2 chiều** cho dễ tính tay (thực tế là hàng trăm chiều, nhưng phép toán y hệt).

Vector truy vấn: **q = [1, 0]**

Ba ứng viên trong kho:

| Ứng viên | Vector | Mô tả bằng lời |
|---|---|---|
| **A** | `[2, 0]` | Cùng hướng với q, nhưng "dài" gấp đôi |
| **B** | `[0, 3]` | Vuông góc với q |
| **C** | `[-1, 0]` | Ngược hướng hoàn toàn với q |

**Tính khoảng cách Euclid (L2) — toán tử `<->`.**

Công thức: lấy hiệu từng chiều, bình phương, cộng lại, rồi lấy căn bậc hai.

```
q → A:  hiệu = (1-2, 0-0) = (-1, 0)
        bình phương và cộng = 1 + 0 = 1
        căn bậc hai = 1.00

q → B:  hiệu = (1-0, 0-3) = (1, -3)
        bình phương và cộng = 1 + 9 = 10
        căn bậc hai ≈ 3.16

q → C:  hiệu = (1-(-1), 0-0) = (2, 0)
        bình phương và cộng = 4 + 0 = 4
        căn bậc hai = 2.00
```

Xếp hạng theo L2: **A (1.00) → C (2.00) → B (3.16)**

**Tính khoảng cách cosine — toán tử `<=>`.**

Cosine đo **góc** giữa hai vector và **bỏ qua hoàn toàn độ dài** của chúng. Công thức: `cosine distance = 1 - cos(góc)`.

```
q → A:  cùng hướng, góc = 0°,    cos(0°) = 1     → distance = 1 - 1  = 0.00
q → B:  vuông góc,   góc = 90°,  cos(90°) = 0    → distance = 1 - 0  = 1.00
q → C:  ngược hướng, góc = 180°, cos(180°) = -1  → distance = 1 - (-1) = 2.00
```

Xếp hạng theo cosine: **A (0.00) → B (1.00) → C (2.00)**

**Bây giờ hãy dừng lại và nhìn kỹ ba điều rút ra được — cả ba đều quan trọng.**

**Điều thứ nhất: hai thước đo cho hai thứ tự khác nhau.** Theo L2, C xếp thứ hai; theo cosine, C xếp cuối. Cùng một bộ dữ liệu, cùng một câu truy vấn, nhưng **chọn thước đo khác là ra kết quả khác**. Đây là lý do việc chọn thước đo không phải chi tiết vụn vặt.

**Điều thứ hai: cosine coi A giống hệt q (distance = 0), dù A dài gấp đôi q.** Vì cosine chỉ quan tâm **hướng**, không quan tâm **độ lớn**. Với văn bản, đây thường là điều ta muốn: một bài viết 2000 chữ và một bài 200 chữ cùng nói về một chủ đề thì nên được coi là giống nhau, chứ không nên bị phạt chỉ vì độ dài khác nhau. **Đây chính là lý do cosine là lựa chọn mặc định cho embedding văn bản.**

**Điều thứ ba — và đây là chỗ đính chính số hai ở đầu bài trở nên rõ ràng:** cosine distance luôn nằm trong khoảng **từ 0 tới 2**, không bao giờ vượt quá. Còn L2 thì **không có giới hạn trên** — hai vector 1536 chiều xa nhau có thể cho L2 bằng 15, 50, hay 300 tùy thang đo của dữ liệu.

Hệ quả trực tiếp: nếu bạn lấy ngưỡng `< 5` từ ví dụ L2 của bài gốc rồi áp vào cosine, ngưỡng đó **không lọc được gì cả** — vì mọi giá trị cosine distance đều nhỏ hơn 5. Bạn tưởng mình đang lọc, nhưng thực ra không.

### 1.6. Câu truy vấn vector đơn giản nhất

Giờ ta viết câu lệnh đầu tiên. Đây là **khung xương của mọi truy vấn vector** — nhớ nguyên khối này là đủ dùng cho 80% trường hợp.

```sql
-- "Cho tôi 5 review giống nhất với embedding này"
SELECT id, content                 -- lấy các cột NGƯỜI ĐỌC ĐƯỢC, không lấy cột embedding
FROM reviews
ORDER BY embedding <=> '[...các số của vector truy vấn...]'
--       ^^^^^^^^^ ^^^ ^^^^^^^^^^^^^^^^^^^^^^^
--       cột vector │   vector truy vấn (do ứng dụng sinh ra từ câu người dùng gõ)
--                  └── toán tử cosine distance
LIMIT 5;                           -- chỉ lấy 5 dòng đầu
```

Giải thích từng mảnh cho thật kỹ:

**`ORDER BY` mặc định sắp xếp tăng dần (ASC).** Với khoảng cách thì **nhỏ = gần = giống**, nên tăng dần chính là "giống nhất lên đầu". Bạn **không cần** viết `ASC`, và **không được** viết `DESC` — viết `DESC` là bạn đang yêu cầu những kết quả **ít liên quan nhất**.

**`LIMIT 5`** cắt lấy 5 dòng đầu. Nó không chỉ để đỡ tốn băng thông — như mục 3.3 sẽ chỉ ra, **`LIMIT` là điều kiện để index được sử dụng**. Đây là chi tiết mà rất nhiều người bỏ qua.

**Cả câu đọc thành lời:** *"Sắp mọi review theo độ gần với vector này, rồi đưa tôi 5 cái gần nhất."*

Bài toán này có tên riêng trong ngành: **k-NN** (viết tắt của *k-Nearest Neighbors*, dịch: *k láng giềng gần nhất*), trong đó `k` chính là con số bạn đặt sau `LIMIT`.

### 1.7. 🧩 [Ngoài bài gốc] — Hai điều nên biết ngay từ Basic

**Một: pgvector mặc định tìm chính xác tuyệt đối, chỉ khi bạn tạo index nó mới chuyển sang gần đúng.**

Đây là chi tiết ít người biết nhưng rất đáng nhớ. Nếu bảng của bạn **chưa có index vector**, pgvector so sánh câu truy vấn với **từng dòng một** trong bảng. Cách này gọi là **exact search** (*tìm chính xác*) — kết quả luôn đúng tuyệt đối, nhưng chậm dần theo kích thước bảng.

Khi bạn tạo index (HNSW hoặc IVFFlat), nó chuyển sang **ANN** (viết tắt của *Approximate Nearest Neighbor*, dịch: *láng giềng gần nhất gần đúng*) — nhanh hơn hàng nghìn lần, đổi lại **thỉnh thoảng bỏ sót** một vài kết quả đúng.

Hệ quả thực tế rất hữu ích: với vài nghìn dòng, bạn **không cần index gì cả** và vẫn có kết quả chính xác 100%. Đừng vội tối ưu khi chưa cần.

**Hai: pgvector có sáu toán tử khoảng cách, không phải ba.**

Bài gốc dạy ba toán tử. Từ phiên bản 0.7 trở đi, pgvector có thêm ba toán tử nữa: `<+>` cho khoảng cách L1 (Manhattan), `<~>` cho khoảng cách Hamming và `<%>` cho khoảng cách Jaccard — hai cái sau dùng cho vector nhị phân.

Bạn không cần thuộc ba cái sau. Nhưng biết chúng tồn tại là đủ để không lúng túng nếu người phỏng vấn hỏi "còn toán tử nào khác không", và câu trả lời "có, cho vector nhị phân và L1, nhưng thực tế 95% trường hợp dùng cosine" cho thấy bạn biết cả bức tranh lẫn thứ tự ưu tiên.

### ✅ Self-check Phần 1

**1. Vì sao embedding trong database và embedding của câu truy vấn bắt buộc phải do cùng một mô hình sinh ra?**
> *Gợi ý đáp án:* Vì mỗi mô hình dựng một "bản đồ ý nghĩa" riêng, đặt cùng một khái niệm ở vị trí khác nhau. So sánh vector từ hai bản đồ khác nhau cho khoảng cách vô nghĩa. Nguy hiểm nhất là khi hai mô hình tình cờ cùng số chiều — database không báo lỗi, kết quả cứ sai âm thầm.

**2. `SELECT embedding FROM ...` trả về cái gì, và vì sao truy vấn nên xoay quanh khoảng cách thay vì cột vector?**
> *Gợi ý đáp án:* Trả về một mảng mấy trăm số mà con người không đọc hiểu được, vì từng chiều riêng lẻ không mang ý nghĩa nào. Ý nghĩa chỉ nằm trong **quan hệ giữa các vector**, tức là trong khoảng cách. Cột vector là nguyên liệu tính toán, không phải thông tin hiển thị.

**3. `<->` trong `tsquery` và `<->` trong pgvector có nghĩa giống nhau không?**
> *Gợi ý đáp án:* Không, hoàn toàn khác. Trong `tsquery` nó nghĩa là FOLLOWED BY — từ này đứng ngay trước từ kia. Trong pgvector nó là toán tử tính khoảng cách Euclid. Chỉ trùng ký hiệu.

**4. Vì sao cosine là lựa chọn mặc định cho embedding văn bản?**
> *Gợi ý đáp án:* Vì cosine chỉ đo góc, bỏ qua độ lớn của vector — nên hai văn bản cùng chủ đề nhưng khác độ dài vẫn được coi là giống nhau. Ngoài ra giá trị của nó luôn nằm trong khoảng cố định [0, 2], nên việc đặt ngưỡng lọc dễ và ổn định hơn nhiều so với L2 (vốn không có giới hạn trên).

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Giả định bạn đã nắm: embedding, khoảng cách cosine và L2, khung `ORDER BY ... LIMIT k`.

### 2.1. Ba toán tử khoảng cách — dùng cái nào khi nào

| Toán tử | Thước đo | Miền giá trị | Ops class khi tạo index |
|---|---|---|---|
| `<->` | Khoảng cách Euclid (L2) | `[0, ∞)` — **không giới hạn trên** | `vector_l2_ops` |
| `<#>` | Tích vô hướng **âm** (negative inner product) | Phụ thuộc độ lớn vector | `vector_ip_ops` |
| `<=>` | Khoảng cách cosine | **`[0, 2]`** — cố định | `vector_cosine_ops` |

**Ops class** (viết tắt của *operator class*, dịch: *lớp toán tử*) là thứ bạn khai báo khi tạo index, để nói cho index biết **nó sẽ được dùng với thước đo nào**. Đây là chi tiết cực kỳ quan trọng, và mục 2.4 sẽ chỉ ra hậu quả khi khai báo sai.

**Về `<#>` — vì sao lại là tích vô hướng *âm*?**

Đây là chỗ khiến người mới bối rối nhất, nên ta giải thích cho rõ.

**Inner product** (dịch: *tích vô hướng*, còn gọi *dot product*) là phép nhân từng cặp phần tử rồi cộng lại. Với tích vô hướng, **giá trị càng LỚN thì hai vector càng giống nhau** — ngược với khoảng cách.

Nhưng khung truy vấn chuẩn của pgvector là `ORDER BY ... LIMIT k`, mà `ORDER BY` mặc định sắp **tăng dần**. Nếu toán tử trả về "càng lớn càng giống", sắp tăng dần sẽ đưa những kết quả **kém giống nhất** lên đầu.

Giải pháp của pgvector rất gọn: **trả về giá trị âm của tích vô hướng**. Khi đó "càng nhỏ càng giống" lại đúng, và bạn dùng chung một khung `ORDER BY ... ASC` cho cả ba toán tử mà không phải nhớ ngoại lệ nào.

Vì vậy: đừng ngạc nhiên khi thấy `<#>` trả về những con số âm như `-0.87`. Nó bình thường, và **`-0.87` giống hơn `-0.20`** (vì âm hơn tức là nhỏ hơn).

**Chọn thước đo nào?**

- **Embedding văn bản → dùng `<=>` (cosine).** Đây là mặc định, và đúng trong đại đa số trường hợp.
- **Nếu vector đã được chuẩn hóa về độ dài 1 → dùng `<#>` (inner product) sẽ nhanh hơn.** **Normalize** (dịch: *chuẩn hóa*) nghĩa là chia mọi phần tử của vector cho độ dài của nó, để mọi vector đều có độ dài đúng bằng 1. Khi mọi vector đã cùng độ dài, cosine và inner product cho ra cùng thứ tự — nhưng inner product ít phép tính hơn nên nhanh hơn một chút.
- **Dùng `<->` (L2) khi độ lớn của vector thật sự mang ý nghĩa** — thường gặp với embedding hình ảnh hoặc dữ liệu số học, ít gặp với văn bản.

💡 **Mẹo thực chiến:** nếu không chắc, cứ dùng cosine. Nó là lựa chọn an toàn nhất, dễ đặt ngưỡng nhất, và là thứ mọi tài liệu ví dụ đều dùng.

### 2.2. Mẫu kinh điển: "tìm những cái giống với một bản ghi ĐÃ CÓ"

Đây là mẫu bạn sẽ dùng nhiều nhất trong thực tế: tính năng "sản phẩm tương tự", "bài viết liên quan", "người dùng có gu giống bạn".

Điểm đặc biệt của mẫu này: vector truy vấn **không đến từ bên ngoài**, mà chính là embedding của một dòng đã có trong bảng. Vì vậy ta dùng **subquery** — một câu truy vấn con nằm lồng bên trong câu truy vấn chính, để lấy ra giá trị cần dùng.

```sql
-- "Tìm 5 review giống nhất với review có id = 1"
SELECT id, content
FROM reviews
WHERE id != 1                       -- ⚠️ LOẠI CHÍNH NÓ RA — xem giải thích bên dưới
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
--                     └─────────── subquery: lấy vector của chính review số 1 ────┘
LIMIT 5;
```

⚠️ **Chỗ khó — vì sao bắt buộc phải có `WHERE id != 1`?**

Hãy nghĩ kỹ: khoảng cách từ một vector **tới chính nó** bằng bao nhiêu? Bằng **0** — con số nhỏ nhất có thể.

Nghĩa là nếu không loại, review số 1 sẽ **luôn luôn đứng đầu bảng kết quả**, với khoảng cách 0. Bạn xin 5 sản phẩm tương tự, nhưng cái đầu tiên chính là sản phẩm người dùng đang xem. Vừa vô dụng, vừa **chiếm mất một suất** trong 5 kết quả — nên bạn chỉ còn 4 gợi ý thật.

Đây là lỗi kinh điển tới mức nó gần như chắc chắn xuất hiện trong phỏng vấn dưới dạng: *"Tôi viết truy vấn tìm sản phẩm tương tự, nhưng kết quả đầu tiên luôn là chính sản phẩm đó. Sao vậy?"*

**Ba toán tử, cùng một mẫu.** Bài gốc đưa cả ba biến thể, và điểm đáng chú ý là chúng khác nhau **đúng ba ký tự**:

```sql
-- Euclid
ORDER BY embedding <-> (SELECT embedding FROM reviews WHERE id = 1)
-- Inner product
ORDER BY embedding <#> (SELECT embedding FROM reviews WHERE id = 1)
-- Cosine
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
```

Nhưng nhớ lại ví dụ chạy tay ở mục 1.5: ba toán tử này **có thể cho ra ba thứ tự khác nhau**. Chúng không thay thế lẫn nhau được — hãy chọn một cái, dùng nó nhất quán, và tạo index với đúng ops class tương ứng.

### 2.3. Hiển thị điểm số cho người dùng — và chỗ đính chính quan trọng

Cho tới giờ, khoảng cách chỉ được dùng để **sắp thứ tự**. Nhưng thường bạn muốn **nhìn thấy** con số đó — để debug, hoặc để hiển thị "độ giống 87%" cho người dùng.

```sql
SELECT id,
       content,
       embedding <=> :q          AS cosine_distance,    -- 0 = giống hệt, 2 = ngược hẳn
       1 - (embedding <=> :q)    AS cosine_similarity   -- 1 = giống hệt, -1 = ngược hẳn
FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;
```

Về ký hiệu `:q` — đó là một **tham số truy vấn** (*query parameter*), chỗ mà ứng dụng của bạn sẽ điền vector thật vào lúc chạy. Cách viết chính xác tùy thư viện bạn dùng (`$1`, `%s`, `?`...), nhưng ý tưởng là một: **đừng nối chuỗi vector thẳng vào câu SQL**, hãy truyền nó như tham số.

❗ **[Đính chính bài gốc — và đây là câu hỏi bẫy số một của bài này]**

Bài gốc gọi truy vấn dùng `<=>` là "cosine similarity". Cần cực kỳ rõ ràng chỗ này:

| | **Cosine distance** (`<=>` trả về cái này) | **Cosine similarity** |
|---|---|---|
| Giống hệt nhau | **0** | **1** |
| Không liên quan | 1 | 0 |
| Ngược nghĩa hoàn toàn | **2** | **-1** |
| Quy tắc đọc | **Càng NHỎ càng giống** | **Càng LỚN càng giống** |
| Công thức liên hệ | `distance = 1 - similarity` | `similarity = 1 - distance` |

⚠️ **Hậu quả nếu nhầm:** giả sử bạn lấy thẳng cosine distance rồi hiển thị cho người dùng dưới nhãn "độ giống". Một kết quả **giống hệt** sẽ hiện là **0%**, còn một kết quả **ngược nghĩa hoàn toàn** hiện là **200%**. Toàn bộ giao diện nói ngược sự thật — và nó sẽ chạy êm ru cho tới khi có người dùng thắc mắc.

💡 **Quy tắc để không bao giờ nhầm:** hễ nhìn thấy chữ **distance** thì nghĩ ngay tới **"khoảng cách trên bản đồ"** — 0 mét nghĩa là đang đứng ngay tại đó. Hễ thấy **similarity** thì nghĩ tới **"điểm số"** — điểm cao là tốt.

Để hiển thị phần trăm cho người dùng: `1 - (embedding <=> :q)` rồi nhân 100.

### 2.4. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển

**Lỗi 1 — Quên loại chính bản ghi nguồn.**

Đã phân tích ở mục 2.2. Triệu chứng: kết quả đầu tiên luôn là chính nó. Cách chữa: `WHERE id != :self_id`.

**Lỗi 2 — Ops class của index không khớp với toán tử trong truy vấn.**

Đây là lỗi **âm thầm và tốn kém nhất** trong cả bài.

```sql
-- Bạn tạo index cho COSINE
CREATE INDEX ON reviews USING hnsw (embedding vector_cosine_ops);

-- ❌ Nhưng truy vấn lại dùng toán tử L2
SELECT * FROM reviews ORDER BY embedding <-> :q LIMIT 5;
--                                       ^^^ không khớp với vector_cosine_ops
```

Chuyện gì xảy ra? Postgres **lặng lẽ bỏ qua index** và quay về **sequential scan** — tức là đọc và tính khoảng cách cho **từng dòng một** trong bảng.

Điều tệ nhất: **không có lỗi nào được báo ra**. Truy vấn vẫn trả về kết quả **đúng** (thậm chí đúng hơn, vì tìm chính xác tuyệt đối). Nó chỉ **chậm**. Với bảng 10 nghìn dòng lúc phát triển thì bạn không nhận ra; với 10 triệu dòng ở production thì nó thành sự cố.

**Cách kiểm tra:** dùng **`EXPLAIN ANALYZE`** — câu lệnh yêu cầu Postgres chạy thật rồi kể lại chi tiết nó đã làm gì.

```sql
EXPLAIN ANALYZE
SELECT id FROM reviews ORDER BY embedding <=> :q LIMIT 5;
```

Trong kết quả, tìm dòng có chữ **`Index Scan`** — nghĩa là index đang được dùng. Nếu thấy **`Seq Scan`** thì index đang bị bỏ qua.

**Quy tắc vàng:** toán tử trong truy vấn phải khớp với ops class lúc tạo index. `<=>` đi với `vector_cosine_ops`, `<->` đi với `vector_l2_ops`, `<#>` đi với `vector_ip_ops`.

**Lỗi 3 — Nhầm khoảng cách với độ giống khi diễn giải kết quả.**

Đã phân tích ở mục 2.3. Biểu hiện thường gặp nhất: viết `ORDER BY ... DESC` vì nghĩ rằng "số lớn là giống hơn". Kết quả: bạn trả về đúng những bản ghi **ít liên quan nhất** trong toàn bộ kho.

Với cả ba toán tử của pgvector, **`ASC` (mặc định) luôn là đúng** — kể cả với `<#>` vốn trả về số âm, vì pgvector đã cố ý thiết kế như vậy (mục 2.1).

### ✅ Self-check Phần 2

**1. Vì sao truy vấn "tìm giống review id=1" cần `WHERE id != 1`?**
> *Gợi ý đáp án:* Khoảng cách từ một vector tới chính nó bằng 0 — nhỏ nhất có thể — nên nó luôn đứng đầu kết quả. Vừa vô dụng vừa chiếm mất một suất trong top-k.

**2. Làm sao đổi cosine distance thành "độ giống %" để hiển thị?**
> *Gợi ý đáp án:* `1 - (embedding <=> :q)` rồi nhân 100. Không được hiển thị thẳng distance dưới nhãn "độ giống", vì hai thứ ngược chiều nhau — giống hệt sẽ hiện thành 0%.

**3. `<#>` trả về giá trị gì, và vì sao vẫn dùng `ORDER BY` tăng dần?**
> *Gợi ý đáp án:* Trả về **giá trị âm** của tích vô hướng. pgvector cố ý đảo dấu để "càng nhỏ càng giống" đúng với cả ba toán tử, nhờ đó dùng chung một khung `ORDER BY ... ASC LIMIT k`.

**4. Index tạo bằng `vector_cosine_ops` nhưng truy vấn dùng `<->`. Chuyện gì xảy ra và làm sao phát hiện?**
> *Gợi ý đáp án:* Index bị bỏ qua, truy vấn rơi về sequential scan — kết quả vẫn đúng nhưng chậm dần theo kích thước bảng, và không có cảnh báo nào. Phát hiện bằng `EXPLAIN ANALYZE`: nếu thấy `Seq Scan` thay vì `Index Scan` thì đúng là lỗi này.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Phần này là trọng tâm thật sự của bài. Nó trả lời một câu hỏi mà `LIMIT` không trả lời được: **"nếu chẳng có gì thật sự giống thì sao?"**

### 3.1. Vấn đề: `LIMIT 5` luôn trả về 5 dòng, kể cả khi cả 5 đều là rác

Hãy nhìn kỹ vào khung truy vấn ta đã học:

```sql
SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;
```

Câu này nói: *"đưa tôi 5 cái gần nhất"*. Nó **không** nói: *"đưa tôi những cái thật sự giống"*.

Sự khác biệt giữa hai câu đó chính là toàn bộ nội dung Phần 3, nên ta làm rõ bằng một ví dụ chạy tay.

### 3.2. Ví dụ chạy tay #2 — thấy tận mắt vì sao cần ngưỡng lọc

Người dùng đang xem một chiếc **áo khoác mùa đông** và hệ thống muốn gợi ý "sản phẩm tương tự". Giả sử kho có 8 sản phẩm, và sau khi tính cosine distance ta được bảng sau:

| Hạng | Sản phẩm | Cosine distance | Thực tế có giống không? |
|---|---|---|---|
| 1 | Áo phao giữ ấm | **0.08** | ✅ Rất giống |
| 2 | Áo khoác gió | **0.12** | ✅ Rất giống |
| 3 | Áo len cổ lọ | **0.19** | ✅ Giống |
| 4 | Găng tay len | **0.55** | ⚠️ Liên quan xa |
| 5 | Nồi cơm điện | **0.71** | ❌ Không liên quan |
| 6 | Tai nghe bluetooth | 0.88 | ❌ Không liên quan |
| 7 | Sách nấu ăn | 0.93 | ❌ Không liên quan |
| 8 | Phân bón cây cảnh | 1.10 | ❌ Không liên quan |

**Với `LIMIT 5`, hệ thống trả về 5 dòng đầu.** Nghĩa là bên cạnh ba gợi ý hợp lý, giao diện của bạn sẽ hiển thị **một đôi găng tay và một cái nồi cơm điện** dưới tiêu đề "Sản phẩm tương tự".

Hãy để ý điều gì đã xảy ra: **thuật toán không làm sai gì cả.** Nồi cơm điện đúng là sản phẩm gần thứ năm — trong một kho chỉ có 8 món. Nó là "ít xa nhất trong đám xa". Vấn đề là câu hỏi bạn đặt ra sai: bạn hỏi "5 cái gần nhất" trong khi cái bạn cần là "những cái đủ giống".

**Đây chính là lúc threshold vào cuộc.**

**Threshold** (dịch: *ngưỡng*) là một giá trị chặn: chỉ những kết quả có khoảng cách **nhỏ hơn ngưỡng** mới được coi là khớp. Nếu đặt ngưỡng ở **0.3**:

```
0.08  ✅ nhận
0.12  ✅ nhận
0.19  ✅ nhận
──────────────── ngưỡng 0.3 ────────────────
0.55  ❌ loại
0.71  ❌ loại
...
```

Kết quả: **3 sản phẩm, tất cả đều thật sự liên quan.** Trả về 3 gợi ý tốt hơn nhiều so với 3 gợi ý tốt cộng 2 gợi ý rác — vì gợi ý rác làm người dùng mất niềm tin vào toàn bộ khối gợi ý đó.

**Câu để nhớ suốt đời:** ***`LIMIT` giới hạn SỐ LƯỢNG, threshold đảm bảo CHẤT LƯỢNG.*** Hai việc hoàn toàn khác nhau, và một hệ thống tốt cần cả hai.

Cách viết:

```sql
-- Chỉ nhận review có cosine distance nhỏ hơn 0.3, và tối đa 5 cái
SELECT id, content, embedding <=> :q AS distance
FROM reviews
WHERE embedding <=> :q < 0.3        -- ngưỡng: bộ lọc CHẤT LƯỢNG
ORDER BY embedding <=> :q
LIMIT 5;                             -- giới hạn SỐ LƯỢNG
```

### 3.3. Ngưỡng phụ thuộc thước đo và bộ dữ liệu — không có con số vạn năng

Đây là đính chính số hai ở đầu bài, giờ ta phân tích cho tới nơi.

Bài gốc dùng ví dụ `WHERE embedding <-> q < 5` với thước đo L2. Ý tưởng đúng, nhưng con số 5 **không phải hằng số của vũ trụ**. Nó phụ thuộc hai thứ.

**Phụ thuộc thứ nhất — thước đo.**

| Thước đo | Miền giá trị | "Gần" nghĩa là |
|---|---|---|
| Cosine (`<=>`) | **[0, 2]** — cố định | Thường dưới 0.2–0.4 là rất giống |
| L2 (`<->`) | **[0, ∞)** — không giới hạn | Hoàn toàn tùy thang đo của dữ liệu |
| Inner product âm (`<#>`) | Phụ thuộc độ lớn vector | Rất khó đặt ngưỡng nếu chưa chuẩn hóa |

Nhìn bảng này là thấy ngay hai hệ quả:

- Ngưỡng `< 5` áp cho **cosine** thì **vô dụng** — mọi giá trị cosine distance đều nhỏ hơn 2, nên điều kiện luôn đúng và bạn không lọc được gì. Bạn tưởng mình có bộ lọc, thực ra không.
- Ngưỡng `< 5` áp cho **L2** thì có thể chặt hoặc lỏng tùy dữ liệu — với bộ này là hợp lý, với bộ khác thì lọc sạch hoặc không lọc gì.

**Đây cũng là lý do thứ hai khiến cosine được ưa dùng** (lý do thứ nhất đã nói ở mục 1.5): vì miền giá trị cố định, một ngưỡng như 0.3 mang ý nghĩa tương đối ổn định giữa các bộ dữ liệu. Với L2 thì mỗi bộ dữ liệu một con số hoàn toàn khác.

**Phụ thuộc thứ hai — bộ dữ liệu và mô hình.**

Ngay cả trong cùng thước đo cosine, ngưỡng phù hợp vẫn khác nhau giữa các bộ dữ liệu. Kho tài liệu pháp lý — nơi mọi văn bản đều dùng cùng một hệ thuật ngữ — sẽ có khoảng cách trung bình giữa các cặp thấp hơn hẳn so với kho bình luận mạng xã hội đủ mọi chủ đề. Cùng ngưỡng 0.3, một bên lọc quá lỏng, bên kia lọc quá chặt.

**Kết luận rất quan trọng: ngưỡng không phải thứ để đoán, mà là thứ để đo.** Mục tiếp theo chỉ cách đo.

### 3.4. Ví dụ chạy tay #3 — hiệu chỉnh ngưỡng bằng dữ liệu thật

Bài gốc có gợi ý rằng nên lấy về cả những bản ghi rất ít giống để kiểm chứng, nhưng nói khá mờ. Ở đây ta làm cho rõ ràng thành một quy trình bạn làm được ngay.

**Ý tưởng cốt lõi:** thay vì đoán ngưỡng, hãy lấy vài chục cặp mà **bạn đã biết trước câu trả lời**, đo khoảng cách của chúng, rồi tìm con số nằm giữa hai nhóm.

**Bước 1 — chuẩn bị hai nhóm mẫu.** Chọn khoảng 20–30 cặp mà bạn tự tin đánh giá bằng mắt:
- Nhóm **GIỐNG**: những cặp mà con người sẽ nói "ừ, hai cái này liên quan".
- Nhóm **KHÁC**: những cặp rõ ràng không liên quan gì tới nhau.

**Bước 2 — chạy truy vấn và ghi lại khoảng cách của từng cặp.** Giả sử ta được kết quả sau:

```
Nhóm GIỐNG (khoảng cách đo được):
   0.05   0.09   0.11   0.14   0.18   0.22   0.27
   └──────────────────────────────────────────┘
                 cao nhất: 0.27

                      ⬇ khoảng trống ⬇

Nhóm KHÁC (khoảng cách đo được):
   0.48   0.55   0.62   0.70   0.77   0.91
   └────────────────────────────────────┘
              thấp nhất: 0.48
```

**Bước 3 — chọn ngưỡng nằm trong khoảng trống giữa hai nhóm.** Ở đây khoảng trống là từ 0.27 tới 0.48, nên một ngưỡng khoảng **0.35** tách hai nhóm rất sạch.

**Bước 4 — hiểu rõ mình đang đánh đổi gì khi dịch ngưỡng.**

Trước hết, hai từ cần định nghĩa:

**Precision** (dịch: *độ chính xác*) trả lời: *trong những kết quả tôi trả về, bao nhiêu phần trăm là đúng?* Precision thấp nghĩa là kết quả lẫn rác.

**Recall** (dịch: *độ bao phủ*) trả lời: *trong tất cả những kết quả đúng đang tồn tại, tôi tìm được bao nhiêu phần trăm?* Recall thấp nghĩa là bỏ sót nhiều.

Hai chỉ số này **đánh đổi lẫn nhau**, và ngưỡng chính là cái núm vặn giữa chúng:

| Ngưỡng | Hệ quả | Khi nào phù hợp |
|---|---|---|
| **Chặt** (ví dụ 0.20) | Precision cao — hầu như không lẫn rác. Nhưng bỏ sót nhiều, và **hay trả về 0 kết quả** | Khi hiển thị kết quả sai gây tổn hại: y tế, pháp lý, tài chính |
| **Lỏng** (ví dụ 0.60) | Recall cao — ít bỏ sót. Nhưng lẫn nhiều kết quả không liên quan | Khi bỏ sót tệ hơn lẫn rác: tìm kiếm khám phá, gợi ý nội dung |

💡 **Mẹo thực chiến:** hãy nhìn cả **khoảng trống** giữa hai nhóm, chứ không chỉ nhìn một con số. Nếu hai nhóm **chồng lấn** nhau — ví dụ nhóm GIỐNG có cặp ở 0.52 còn nhóm KHÁC có cặp ở 0.41 — thì **không tồn tại ngưỡng nào tách được chúng**. Khi gặp tình huống đó, vấn đề không nằm ở ngưỡng mà nằm ở **mô hình embedding không đủ tốt cho dữ liệu của bạn**. Đây là một chẩn đoán rất giá trị: nó cứu bạn khỏi việc ngồi chỉnh ngưỡng vô ích hàng tuần trong khi thứ cần đổi là mô hình.

### 3.5. ⚠️ Bẫy index: ngưỡng "thuần" khiến truy vấn mất index

Đây là đính chính số ba, và là phần kỹ thuật quan trọng nhất của Phần 3.

Hãy so sánh ba cách viết:

```sql
-- ✅ TỐT: k-NN chuẩn — index được dùng
SELECT * FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;

-- ✅ TỐT: ngưỡng kèm ORDER BY và LIMIT — index vẫn được dùng
SELECT * FROM reviews
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q
LIMIT 5;

-- ⚠️ NGUY HIỂM: ngưỡng THUẦN, không ORDER BY, không LIMIT
SELECT * FROM reviews
WHERE embedding <=> :q < 0.3;
-- → phải tính khoảng cách cho MỌI dòng trong bảng
```

**Vì sao lại thế?** Vì index ANN — cả HNSW lẫn IVFFlat — được thiết kế để trả lời đúng một loại câu hỏi: ***"cho tôi k phần tử gần nhất"***. Cấu trúc bên trong của chúng là để đi từ điểm truy vấn lan dần ra các hàng xóm gần, rồi dừng lại khi đủ k.

Chúng **không** được thiết kế để trả lời câu hỏi ***"cho tôi tất cả những phần tử trong bán kính x"*** — loại truy vấn này gọi là **range scan** (*quét theo khoảng*). Khi gặp một câu `WHERE khoảng_cách < x` đứng một mình, index không biết dừng ở đâu, nên Postgres đành tính khoảng cách cho từng dòng một.

**Quy tắc để nhớ:** ***k-NN thì dùng được index, range thuần thì không. Luôn kèm `ORDER BY khoảng_cách LIMIT k`.***

🧩 **[Ngoài bài gốc] — Cách viết chuẩn theo tài liệu chính thức của pgvector.**

Có một chi tiết tinh tế mà ngay cả người dùng pgvector lâu năm cũng hay bỏ qua. Tài liệu chính thức khuyến nghị: khi lọc theo khoảng cách, hãy **đặt điều kiện lọc BÊN NGOÀI một CTE được materialize**, thay vì viết thẳng vào `WHERE`.

**CTE** (viết tắt của *Common Table Expression*) là cách đặt tên cho một truy vấn con bằng từ khóa `WITH`, để dùng lại ở câu chính. Từ khóa **`MATERIALIZED`** buộc Postgres phải **chạy xong CTE và giữ kết quả lại** trước, thay vì trộn nó vào câu ngoài.

```sql
-- Cách viết được pgvector khuyến nghị khi lọc theo khoảng cách
WITH nearest_results AS MATERIALIZED (
    SELECT id, content, embedding <=> :q AS distance
    FROM reviews
    ORDER BY distance                     -- k-NN thuần túy → index chắc chắn được dùng
    LIMIT 20                              -- lấy rộng hơn số bạn cần một chút
)
SELECT * FROM nearest_results
WHERE distance < 0.3                      -- lọc ngưỡng ở NGOÀI, trên tập nhỏ đã lấy về
ORDER BY distance;
```

**Vì sao cách này tốt hơn?** Vì nó tách bạch hai việc: bên trong CTE là một truy vấn k-NN thuần khiết mà index xử lý tối ưu; bên ngoài chỉ là lọc trên 20 dòng đã nằm sẵn trong bộ nhớ — gần như miễn phí. Nếu viết gộp, bộ thực thi của Postgres có thể quét nhiều hơn mức cần thiết.

💡 Lưu ý mẫu này lấy `LIMIT 20` dù có thể bạn chỉ cần 5. Lý do: sau khi lọc ngưỡng, số dòng còn lại sẽ ít hơn 20. Lấy rộng ra một chút để đảm bảo sau khi lọc vẫn còn đủ kết quả — đây cũng là ý tưởng dẫn tới mục tiếp theo.

### 3.6. Overfiltering — khi lọc quá tay làm mất kết quả

**Overfiltering** (dịch: *lọc quá tay*) là hiện tượng bạn nhận về ít kết quả hơn mong đợi khi kết hợp tìm kiếm vector với điều kiện lọc nghiệp vụ.

Nguyên nhân nằm ở **thứ tự thực hiện**. Với index ANN, Postgres thường **tìm hàng xóm gần trước, rồi mới lọc sau**. Hình dung cụ thể:

```
Bước 1: index lấy 40 hàng xóm gần nhất (số này do tham số ef_search quyết định)
Bước 2: áp điều kiện WHERE category = 'áo khoác'
Bước 3: trong 40 cái đó chỉ có 3 cái thuộc danh mục áo khoác
Kết quả: bạn xin 10, nhận về 3
```

Vấn đề là **vẫn còn nhiều áo khoác phù hợp trong kho** — chúng chỉ không lọt vào 40 hàng xóm đầu tiên mà index xét tới.

**`ef_search`** là tham số quy định index xét bao nhiêu ứng viên trước khi dừng. Tăng nó lên thì recall tốt hơn nhưng truy vấn chậm hơn — một trade-off nữa.

pgvector từ phiên bản 0.8 có giải pháp riêng: **iterative scan** (*quét lặp*) — nếu sau khi lọc chưa đủ kết quả, index tự động quét tiếp thay vì bỏ cuộc.

```sql
-- Bật quét lặp cho index HNSW
SET hnsw.iterative_scan = relaxed_order;
-- hai lựa chọn:
--   strict_order  = đảm bảo kết quả đúng thứ tự khoảng cách tuyệt đối
--   relaxed_order = cho phép lệch thứ tự chút ít, đổi lại recall tốt hơn
```

Sau khi đặt tham số này, bạn **không phải sửa lại các câu truy vấn đã viết** — nó tác động ở tầng index.

### 3.7. Edge case — các trường hợp biên phải thủ sẵn

**Edge case** (dịch: *trường hợp biên*) là những tình huống hiếm gặp, nằm ở rìa của những gì bạn dự tính, nhưng khi xảy ra thì gây lỗi hoặc kết quả sai. Chủ động nêu edge case trong phỏng vấn luôn ghi điểm.

**Ngưỡng quá chặt dẫn tới 0 kết quả.** Đây là hành vi **đúng** — nghĩa là thật sự không có gì đủ giống. Nhưng ứng dụng của bạn phải xử lý được: hiển thị "không tìm thấy kết quả phù hợp" hoặc **ẩn hẳn khối gợi ý**, chứ không phải văng lỗi hay hiện một khung trống trơn.

**Đổi mô hình làm ngưỡng cũ mất hiệu lực.** Ngưỡng gắn chặt với cặp (mô hình, thước đo). Re-embed bằng mô hình mới thì **phải hiệu chỉnh lại ngưỡng từ đầu** theo quy trình ở mục 3.4. Nhiều đội quên bước này và tự hỏi vì sao chất lượng tìm kiếm tụt sau khi "nâng cấp" mô hình.

**Dùng `<#>` với vector chưa chuẩn hóa.** Tích vô hướng bị độ lớn vector làm nhiễu, nên ngưỡng đặt trên nó rất khó tin cậy. Hãy chuẩn hóa vector trước, hoặc dùng cosine.

**Vector NULL.** Những hàng chưa có embedding sẽ không bao giờ xuất hiện trong kết quả — im lặng, không báo lỗi. Hãy theo dõi số hàng có `embedding IS NULL` để biết còn bao nhiêu dữ liệu chưa được xử lý.

**Viết ngưỡng theo similarity hay theo distance?** Hai cách sau tương đương về mặt toán học:
```sql
WHERE 1 - (embedding <=> :q) > 0.7    -- ngưỡng theo similarity
WHERE embedding <=> :q < 0.3          -- ngưỡng theo distance — NÊN DÙNG CÁCH NÀY
```
Nên viết theo **distance**, vì bộ lập kế hoạch truy vấn của Postgres nhận diện được dạng này và tối ưu tốt hơn. Bọc toán tử vào trong một biểu thức tính toán khiến nó khó nhận ra hơn.

### ✅ Self-check Phần 3

**1. Vì sao không có "ngưỡng vạn năng"? Thước đo nào dễ đặt ngưỡng ổn định nhất và vì sao?**
> *Gợi ý đáp án:* Vì mỗi thước đo có miền giá trị khác nhau (cosine cố định [0,2]; L2 không giới hạn trên) và mỗi bộ dữ liệu có phân bố khoảng cách riêng. Cosine dễ nhất vì miền cố định nên một ngưỡng như 0.3 mang ý nghĩa tương đối ổn định.

**2. `WHERE embedding <=> :q < 0.3` đứng một mình có dùng được index ANN không? Nên viết thế nào?**
> *Gợi ý đáp án:* Không — index ANN chỉ tối ưu cho k-NN (`ORDER BY khoảng_cách LIMIT k`), còn range scan thuần buộc phải tính khoảng cách mọi dòng. Nên viết kèm `ORDER BY ... LIMIT k`, và tốt nhất là dùng mẫu CTE `MATERIALIZED` với điều kiện lọc đặt bên ngoài.

**3. Quy trình hiệu chỉnh ngưỡng bằng mẫu "giống" và "khác" gồm những bước nào?**
> *Gợi ý đáp án:* Chuẩn bị 20–30 cặp đã biết trước là giống và là khác → đo khoảng cách của từng nhóm → tìm khoảng trống giữa hai nhóm → chọn ngưỡng nằm trong khoảng trống đó, dịch về phía chặt hay lỏng tùy nhu cầu precision/recall. Nếu hai nhóm chồng lấn thì vấn đề nằm ở mô hình chứ không ở ngưỡng.

**4. `LIMIT` và threshold khác nhau ở chỗ nào?**
> *Gợi ý đáp án:* `LIMIT` giới hạn **số lượng** kết quả; threshold đảm bảo **chất lượng** kết quả. `LIMIT 5` luôn trả về đủ 5 dòng kể cả khi cả 5 đều không liên quan. Một hệ thống tốt cần cả hai.

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Ba phần trước dạy viết truy vấn đúng. Phần này bàn chuyện khác: **làm sao giữ cho nó vẫn đúng và vẫn nhanh khi dữ liệu lớn lên, và làm sao biết được khi nào nó bắt đầu tệ đi.**

### 4.1. Các mẫu truy vấn ở quy mô lớn

**Lọc trước khi xếp hạng (pre-filtering).**

Nguyên tắc chung: **thu hẹp tập ứng viên càng sớm càng tốt**. Nếu truy vấn của bạn luôn kèm điều kiện theo khách hàng, danh mục hoặc khoảng thời gian, hãy để những điều kiện đó phát huy tác dụng trước khi phải tính khoảng cách cho hàng triệu dòng.

Công cụ mạnh nhất cho việc này là **partitioning** (dịch: *phân mảnh*) — chia một bảng khổng lồ thành nhiều bảng con theo một tiêu chí, ví dụ mỗi khách hàng doanh nghiệp một mảnh. Khi truy vấn có điều kiện theo tiêu chí đó, bộ lập kế hoạch sẽ **prune** (*cắt tỉa*) — bỏ qua hẳn những mảnh không liên quan. pgvector hỗ trợ index vector trên từng mảnh riêng.

Nhưng nhớ mục 3.6: lọc mạnh dễ dẫn tới overfiltering. Vì vậy pre-filter phải đi kèm iterative scan hoặc `ef_search` đủ rộng.

**Kết hợp với tìm kiếm từ khóa (hybrid search).**

Tìm theo vector giỏi bắt **ý nghĩa** nhưng hay trượt ở những chỗ cần chính xác từng chữ: mã sản phẩm, tên riêng, số phiên bản. Tìm theo từ khóa thì ngược lại. Giải pháp là chạy cả hai rồi hợp nhất kết quả — thường bằng **RRF** (*Reciprocal Rank Fusion*), tức là cộng nghịch đảo thứ hạng từ hai nhánh thay vì cộng điểm số (vì hai loại điểm không cùng thang đo).

Đây là **kiến trúc truy hồi mặc định của năm 2026**, và giáo trình full-text search đã trình bày chi tiết.

**Phân trang kết quả vector.**

`ORDER BY khoảng_cách LIMIT k OFFSET n` chạy được, nhưng **`OFFSET` lớn rất tốn kém với ANN** — để bỏ qua 500 dòng đầu, index vẫn phải tìm ra đủ 500 dòng đó trước.

Thực tế, các hệ thống tìm kiếm ngữ nghĩa hiếm khi phân trang sâu. Lý do vừa kỹ thuật vừa sản phẩm: **trang thứ 20 của kết quả tìm kiếm ngữ nghĩa gần như chắc chắn là rác** — nếu 200 kết quả đầu chưa có thứ người dùng cần thì cách chữa là sửa câu truy vấn, không phải lật thêm trang. Mẫu phổ biến là lấy top-k rộng rồi **rerank** (xếp hạng lại bằng một mô hình chính xác hơn), thay vì phân trang.

**Cache embedding của câu truy vấn.**

Mỗi lần người dùng tìm kiếm, ứng dụng phải gọi mô hình AI để biến câu hỏi thành vector — việc này tốn cả thời gian lẫn tiền. Nhưng embedding của **cùng một câu chữ** luôn cho ra **cùng một vector**. Vậy nên hãy lưu đệm theo mã băm của câu truy vấn.

Với những hệ thống có một số câu hỏi phổ biến lặp đi lặp lại — mà đa số hệ thống đều vậy — kỹ thuật đơn giản này cắt được một phần đáng kể độ trễ trên đường đi nóng.

### 4.2. Hiệu chỉnh ngưỡng một cách khoa học: golden set

Mục 3.4 đã cho quy trình cơ bản. Ở tầm staff, ta biến nó thành một **tài sản của tổ chức**.

**Golden set** (dịch: *tập vàng*) là một bộ dữ liệu gồm các cặp truy vấn – kết quả **đã được con người gán nhãn** là liên quan hay không liên quan. Nó là "thước đo chuẩn" mà bạn dùng để đánh giá mọi thay đổi.

Quy trình đầy đủ:

1. **Xây golden set** — vài trăm cặp là đủ để bắt đầu. Lấy từ log tìm kiếm thật, rồi nhờ người trong đội (hoặc bộ phận nghiệp vụ) gán nhãn.
2. **Chạy truy vấn trên toàn bộ tập, ghi lại khoảng cách** của nhóm liên quan và nhóm không liên quan.
3. **Vẽ phân bố hai nhóm và chọn ngưỡng tách chúng** — theo đúng cách ở mục 3.4, dịch về phía chặt hay lỏng tùy nhu cầu sản phẩm.
4. **Chạy lại định kỳ**, và **bắt buộc chạy lại mỗi khi đổi mô hình embedding**.

Điểm quan trọng ở tầm staff: golden set **không chỉ dùng để đặt ngưỡng**. Nó còn là công cụ để đo recall của index, để so sánh hai mô hình embedding, và để trả lời câu hỏi "bản thay đổi này làm chất lượng tìm kiếm tốt lên hay tệ đi". Đó là lý do nó đáng được đầu tư xây dựng và duy trì như một phần của sản phẩm, chứ không phải một script tạm ai đó viết rồi bỏ.

> **Câu chốt tầm staff:** *"Ngưỡng không phải con số phép màu bạn đoán ra — nó là một đường biên quyết định bạn hiệu chỉnh trên dữ liệu đã gán nhãn."*

### 4.3. Đánh đổi giữa "nhanh" và "đúng": ANN cộng ngưỡng

Đây là một chi tiết tinh tế mà rất ít người nghĩ tới, và nêu ra được nó trong phỏng vấn là điểm cộng lớn.

Nhớ lại mục 1.7: khi có index, pgvector chuyển sang tìm **gần đúng**. Nghĩa là nó có thể **bỏ sót** một số kết quả thật sự gần.

Bây giờ ghép hai chuyện lại: bạn áp ngưỡng lên kết quả của một phép tìm gần đúng. Hệ quả là **có thể tồn tại những bản ghi thật sự nằm trong ngưỡng nhưng index không tìm ra**. Ngưỡng của bạn hoạt động đúng trên tập nó nhận được, nhưng tập đó đã thiếu sẵn rồi.

Với gợi ý sản phẩm thì chuyện này không sao — bỏ sót vài món không ai chết. Nhưng với những trường hợp đòi hỏi đảm bảo — ví dụ hệ thống rà soát tuân thủ pháp lý cần tìm **mọi** tài liệu tương tự — thì cần **truy hồi hai tầng (two-stage retrieval)**:

```
Tầng 1: dùng ANN lấy top-N RỘNG (ví dụ N = 500) — nhanh, chấp nhận gần đúng
Tầng 2: tính khoảng cách CHÍNH XÁC trên 500 dòng đó, rồi áp ngưỡng
```

Cách này cho bạn tốc độ của ANN ở tầng một và độ tin cậy của phép tính chính xác ở tầng hai — trên một tập nhỏ nên chi phí không đáng kể.

**Và một điều phải nhớ:** ngưỡng **không cứu được** một index có recall kém. Nếu index bỏ sót 30% kết quả đúng, thì mọi việc tinh chỉnh ngưỡng đều là chỉnh trên phần 70% còn lại. Hai vấn đề này phải chẩn đoán và chữa riêng.

### 4.4. Chi phí, độ trễ, giám sát

**Độ trễ.** Đường đi của một truy vấn tìm kiếm gồm hai chặng: (a) gọi mô hình để biến câu hỏi thành vector, (b) tìm trong database. Nhiều đội chỉ tối ưu chặng (b) rồi ngạc nhiên vì độ trễ không giảm — trong khi chặng (a) mới là chỗ tốn thời gian. Hãy đo cả hai riêng biệt trước khi tối ưu.

**`ef_search` là núm vặn chính** ở chặng (b): tăng lên thì recall tốt hơn nhưng chậm hơn. Và cần nhấn mạnh: **ngưỡng không thay thế được việc chỉnh `ef_search`** — chúng giải quyết hai vấn đề khác nhau (chất lượng lọc so với độ bao phủ của index).

**Chi phí.** Mỗi truy vấn thời gian thực đều phải trả tiền cho việc sinh embedding câu hỏi. Cách giảm: cache như mục 4.1, và giữ `k` nhỏ.

**Những chỉ số cần giám sát** — đây là danh sách đáng in ra dán lên tường:

| Chỉ số | Vì sao quan trọng | Dấu hiệu bất thường |
|---|---|---|
| **Phân bố khoảng cách theo thời gian** | Phát hiện **drift** — dữ liệu hoặc hành vi người dùng thay đổi dần | Khoảng cách trung bình dịch chuyển → ngưỡng cần hiệu chỉnh lại |
| **Tỉ lệ truy vấn trả về 0 kết quả** | Cho biết ngưỡng có quá chặt không | Tăng đột ngột → ngưỡng sai, hoặc dữ liệu thay đổi |
| **Tỉ lệ truy vấn rơi về `Seq Scan`** | Phát hiện câu truy vấn viết sai làm mất index | Bất kỳ giá trị nào lớn hơn 0 đều đáng điều tra |
| **p95 / p99 độ trễ** | **p95** = 95% truy vấn nhanh hơn con số này. Dùng thay cho trung bình vì trung bình che giấu đúng những trường hợp tệ nhất | Vượt ngưỡng cam kết với người dùng |
| **Recall trên golden set** | Chất lượng thật sự của tìm kiếm | Tụt mà không ai biết — đây là chỉ số **hay bị bỏ quên nhất** |

⚠️ **Chỗ khó — vì sao recall hay bị bỏ quên.** Vì recall tụt **không gây lỗi, không làm chậm, không có cảnh báo nào**. Hệ thống vẫn trả về đủ số kết quả, vẫn nhanh, mọi biểu đồ đều xanh. Nó chỉ lặng lẽ trả về những kết quả tệ hơn. Nếu bạn không chủ động đo bằng golden set, bạn sẽ không bao giờ biết.

### 4.5. Ảnh hưởng tới sản phẩm và cách nói với người không rành kỹ thuật

**Stakeholder** (dịch: *bên liên quan*) là những người có lợi ích gắn với dự án nhưng không hiểu kỹ thuật: quản lý sản phẩm, giám đốc, bộ phận kinh doanh.

Điều thú vị của bài này là quyết định về ngưỡng **không phải quyết định kỹ thuật thuần túy** — nó là **quyết định sản phẩm**, và staff engineer phải đưa nó ra bàn thay vì tự quyết một mình.

**Cách nói với quản lý sản phẩm:**

> *"Kết quả tìm kiếm luôn được xếp hạng theo độ giống. Khi ta bảo hệ thống 'cho tôi 5 kết quả gần nhất', nó sẽ luôn đưa đủ 5 — kể cả khi chẳng cái nào thật sự liên quan. Nó chỉ đơn giản là chọn ra 5 cái ít tệ nhất.*
>
> *Ngưỡng là bộ lọc chất lượng: 'chỉ trả về những kết quả đủ giống, còn không thì thà không trả gì'. Ta cần quyết định cùng nhau: khi không có gì đủ tốt, ta muốn **ẩn hẳn khối gợi ý** hay muốn **hiện đại một cái gì đó**?*
>
> *Con số ngưỡng không phải đoán — ta hiệu chỉnh nó trên dữ liệu thật đã gán nhãn. Và nếu sau này đổi mô hình AI, con số đó phải được hiệu chỉnh lại."*

Hãy để ý: không có chữ nào là "cosine", "HNSW", hay "recall". Thay vào đó là một câu hỏi sản phẩm rõ ràng mà quản lý sản phẩm trả lời được.

**Quyết định trải nghiệm người dùng.** "Thà im lặng còn hơn gợi ý rác" là một lựa chọn có chủ đích, và nó thường **đúng** — vì một khối "sản phẩm tương tự" chứa nồi cơm điện làm người dùng mất niềm tin vào **toàn bộ** hệ thống gợi ý, không chỉ vào một gợi ý sai đó.

**Ảnh hưởng tới roadmap.** Golden set là tài sản dùng lại được cho nhiều việc: đặt ngưỡng, đo recall, so sánh mô hình, đánh giá mọi thay đổi trong tương lai. Hãy đưa việc xây dựng nó vào kế hoạch như một hạng mục chính thức, đừng để nó thành script tạm của một người.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff engineer

> **Đề bài:** *"Xây tính năng 'sản phẩm tương tự' cho một trang thương mại điện tử. Yêu cầu: hiển thị tối đa 8 sản phẩm **thực sự** giống; nếu không có gì đủ giống thì không hiển thị rác; lọc theo còn hàng và cùng danh mục; độ trễ p95 dưới 100ms."*

Đề bài này được thiết kế khéo: cụm **"thực sự giống"** và **"không hiển thị rác"** chính là tín hiệu người phỏng vấn đang chờ bạn nói tới **threshold**. Nếu bạn chỉ trả lời bằng `ORDER BY ... LIMIT 8`, bạn đã trượt mất trọng tâm.

**Nhịp 1 — Làm rõ đề.** Dùng thước đo nào (cosine, vì là embedding sản phẩm)? Đã có golden set để hiệu chỉnh ngưỡng chưa, nếu chưa thì ai gán nhãn được? Bao lâu đổi mô hình một lần? Khi không có kết quả nào đủ giống thì sản phẩm muốn hành xử ra sao?

**Nhịp 2 — Hình dạng truy vấn.** Đây là phần lõi, và nên viết ra:

```sql
WITH nearest AS MATERIALIZED (
    SELECT id, name, embedding <=> :q AS distance
    FROM products
    WHERE in_stock = true              -- lọc nghiệp vụ
      AND category = :category
      AND id != :current_product_id    -- loại chính sản phẩm đang xem
    ORDER BY distance
    LIMIT 30                            -- lấy rộng để sau khi lọc ngưỡng vẫn còn đủ
)
SELECT id, name, 1 - distance AS similarity
FROM nearest
WHERE distance < :threshold             -- bộ lọc CHẤT LƯỢNG
ORDER BY distance
LIMIT 8;                                -- giới hạn SỐ LƯỢNG
```

Nói to lý do từng dòng: `id != :current` để không gợi ý chính nó; ngưỡng để đảm bảo chất lượng; `LIMIT 8` để giới hạn số lượng; CTE materialize để giữ index; lấy 30 rồi cắt còn 8 để chống overfiltering.

**Nhịp 3 — Ngưỡng.** Hiệu chỉnh trên golden set gồm cặp giống và cặp khác; chọn điểm tách hai phân bố; nghiêng về phía **chặt** vì đề bài nói rõ "không hiển thị rác" — tức là ưu tiên precision hơn recall. Lưu ngưỡng kèm phiên bản mô hình, không hard-code rải rác trong code.

**Nhịp 4 — Index và độ trễ.** HNSW với `vector_cosine_ops`; chỉnh `ef_search` theo mục tiêu p95; bật `hnsw.iterative_scan` vì có lọc theo danh mục; xác nhận bằng `EXPLAIN ANALYZE` rằng vẫn thấy `Index Scan`. Đo riêng thời gian sinh embedding và thời gian truy vấn — với tính năng "sản phẩm tương tự" thì vector truy vấn là embedding của sản phẩm **đã có sẵn trong bảng**, nên chặng gọi mô hình gần như biến mất. Đó là một lợi thế đáng nêu.

**Nhịp 5 — Trải nghiệm người dùng.** Nếu sau khi lọc còn 0 sản phẩm thì **ẩn hẳn khối "sản phẩm tương tự"**, không hiển thị khung trống, không hạ ngưỡng để lấp chỗ. Đây là quyết định cần thống nhất với quản lý sản phẩm chứ không tự quyết.

**Nhịp 6 — Giám sát.** Phân bố khoảng cách theo thời gian (phát hiện drift), tỉ lệ khối gợi ý bị ẩn vì không đủ kết quả, p99, tỉ lệ rơi về `Seq Scan`, recall trên golden set.

**Nhịp 7 — Câu chốt.** *"`LIMIT` giới hạn số lượng, ngưỡng đảm bảo chất lượng — chỉ khi phối hợp cả hai thì người dùng mới có trải nghiệm tốt. Và ngưỡng là con số tôi hiệu chỉnh trên dữ liệu, không phải con số tôi đoán."*

Phân biệt được **"gần nhất"** với **"đủ tốt"** chính là thứ người phỏng vấn muốn nghe ở tầm staff trong câu hỏi này.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Bản rút gọn để ôn nhanh trước buổi phỏng vấn. Mọi thứ ở đây đã được giải thích đầy đủ phía trên.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ | Định nghĩa một dòng |
|---|---|
| **Cùng model / cùng số chiều** | Hai yêu cầu bắt buộc để khoảng cách có nghĩa; khác model = kết quả vô nghĩa mà không báo lỗi |
| **`<->` / `<#>` / `<=>`** | L2 (Euclid) / tích vô hướng **âm** / cosine distance |
| **`<+>` / `<~>` / `<%>`** | L1 (Manhattan) / Hamming / Jaccard — thêm từ pgvector 0.7, ít dùng |
| **Ops class** | `vector_l2_ops` / `vector_ip_ops` / `vector_cosine_ops` — **phải khớp toán tử trong truy vấn** |
| **k-NN query** | `ORDER BY khoảng_cách LIMIT k` — khung xương của mọi truy vấn vector |
| **Cosine distance vs similarity** | `<=>` trả **distance** ∈ [0,2], nhỏ = giống; similarity = `1 - distance` |
| **Threshold (ngưỡng)** | Bộ lọc "đủ gần"; phụ thuộc **thước đo** và **bộ dữ liệu**; phải hiệu chỉnh, không đoán |
| **`WHERE id != :self`** | Loại chính bản ghi nguồn — khoảng cách tới chính nó bằng 0 nên luôn đứng đầu |
| **Exact vs ANN** | Không index = chính xác 100% nhưng chậm; có index = gần đúng, nhanh hơn nghìn lần |
| **Range scan vs k-NN** | `WHERE distance < x` thuần **không dùng được index ANN**; k-NN thì dùng được |
| **CTE `MATERIALIZED`** | Mẫu pgvector khuyến nghị: k-NN bên trong, lọc ngưỡng bên ngoài |
| **Overfiltering / iterative scan** | Lọc quá tay làm thiếu kết quả; chữa bằng `hnsw.iterative_scan` (pgvector 0.8+) |
| **`ef_search`** | Số ứng viên index xét trước khi dừng — cao thì recall tốt, chậm hơn |
| **Precision / Recall** | Kết quả trả về đúng bao nhiêu % / tìm được bao nhiêu % kết quả đúng đang có |
| **Golden set** | Tập cặp đã gán nhãn giống/khác — dùng để hiệu chỉnh ngưỡng và đo recall |
| **Two-stage retrieval** | ANN lấy top-N rộng → tính khoảng cách chính xác + áp ngưỡng trên tập nhỏ đó |
| **Drift** | Phân bố khoảng cách dịch chuyển theo thời gian → ngưỡng cần hiệu chỉnh lại |
| **`tsquery` vs pgvector** | Lexeme (từ khóa) khác embedding (ngữ nghĩa); `<->` có hai nghĩa tùy ngữ cảnh |

### 5.2. Core concepts — nếu chỉ nhớ 10 điều

1. Khung xương của mọi truy vấn vector: **`ORDER BY khoảng_cách LIMIT k`**, tăng dần → gần nhất lên đầu.
2. **Đừng `SELECT` cột vector** — từng chiều không mang ý nghĩa đọc được; mọi thứ xoay quanh **khoảng cách**.
3. Bắt buộc **cùng model, cùng số chiều** cho cả kho lẫn câu truy vấn — sai thì kết quả loạn mà không báo lỗi.
4. **`<=>` trả cosine *distance*** (nhỏ = giống), không phải similarity. Similarity = `1 - (embedding <=> q)`.
5. **`<#>` trả tích vô hướng ÂM** — pgvector cố ý đảo dấu để `ORDER BY ASC` đúng cho cả ba toán tử.
6. Tìm giống một bản ghi có sẵn: subquery lấy embedding + **`WHERE id != :self`**.
7. **`LIMIT` giới hạn SỐ LƯỢNG, threshold đảm bảo CHẤT LƯỢNG** — `LIMIT 5` luôn trả đủ 5, kể cả 5 cái rác.
8. Ngưỡng **phụ thuộc thước đo và bộ dữ liệu**; cosine [0,2] dễ đặt ổn định hơn L2 (không giới hạn trên).
9. **Ngưỡng thuần không dùng được index ANN** → luôn kèm `ORDER BY ... LIMIT k`, tốt nhất là mẫu CTE `MATERIALIZED`.
10. Hiệu chỉnh ngưỡng bằng **golden set** (cặp giống và cặp khác), không đoán; đổi model thì phải hiệu chỉnh lại.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Gần nhất không có nghĩa là đủ tốt"** — phân biệt `LIMIT` với threshold trong một câu.
- **"Distance là khoảng cách trên bản đồ, similarity là điểm số"** — 0 mét là đang đứng tại đó; điểm cao là tốt. Không bao giờ nhầm lại được.
- **"Ngưỡng là đường biên quyết định, không phải con số phép màu"** — nó được đo, không được đoán.
- **"k-NN thì có index, range thuần thì quét cả bảng"** — luôn `ORDER BY` kèm `LIMIT`.
- **"Cùng model hoặc vô nghĩa"** — vector từ hai bản đồ khác nhau không so được.
- **"Nếu hai nhóm chồng lấn thì lỗi ở model, không phải ở ngưỡng"** — chẩn đoán cứu bạn hàng tuần chỉnh vô ích.

### 5.4. Code cần thuộc lòng

**(a) k-NN cơ bản:**
```sql
SELECT id, content FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;
```

**(b) Tìm giống một bản ghi có sẵn, loại chính nó:**
```sql
SELECT id, content FROM reviews
WHERE id != 1
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;
```

**(c) Hiển thị độ giống cho người dùng:**
```sql
SELECT id, content,
       1 - (embedding <=> :q) AS similarity   -- 1 = giống hệt
FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;
```

**(d) Ngưỡng đúng cách — mẫu được pgvector khuyến nghị:**
```sql
WITH nearest AS MATERIALIZED (
    SELECT id, content, embedding <=> :q AS distance
    FROM reviews
    ORDER BY distance
    LIMIT 20                    -- k-NN thuần → index chắc chắn được dùng
)
SELECT * FROM nearest
WHERE distance < 0.3            -- lọc ngưỡng ở NGOÀI, trên tập nhỏ
ORDER BY distance;
```

**(e) Ba lệnh chẩn đoán phải nhớ:**
```sql
EXPLAIN ANALYZE SELECT ...;              -- tìm "Index Scan", nếu thấy "Seq Scan" là có vấn đề
SET hnsw.ef_search = 100;                -- tăng recall, đổi lại chậm hơn
SET hnsw.iterative_scan = relaxed_order; -- chống overfiltering khi có WHERE lọc mạnh
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Viết truy vấn tìm 5 bản ghi giống nhất."**
> `SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;` — tăng dần vì khoảng cách nhỏ là gần. Nếu tìm giống một dòng có sẵn thì dùng subquery lấy embedding của dòng đó và nhớ `WHERE id != :self`.

**2. [BẪY] "`<=>` trả về cosine similarity đúng không?"**
> Không — nó trả về cosine **distance**, miền [0, 2], **nhỏ là giống**. Similarity = `1 - (embedding <=> q)`. Nếu nhầm mà hiển thị thẳng cho người dùng thì kết quả giống hệt sẽ hiện thành 0%.

**3. [BẪY] "Tìm sản phẩm tương tự mà kết quả đầu luôn là chính sản phẩm đó. Sao vậy?"**
> Quên `WHERE id != :self`. Khoảng cách từ một vector tới chính nó bằng 0 nên nó luôn đứng đầu, đồng thời chiếm mất một suất trong top-k.

**4. "Threshold để làm gì? Đặt bao nhiêu?"**
> Để lọc **chất lượng** — `LIMIT` chỉ giới hạn **số lượng** và luôn trả đủ k dòng kể cả khi toàn rác. Con số phụ thuộc thước đo và bộ dữ liệu; hiệu chỉnh bằng golden set gồm cặp giống và cặp khác, chọn điểm tách hai phân bố. Cosine dễ đặt hơn L2 vì miền cố định [0, 2].

**5. [BẪY] "`WHERE embedding <=> q < 0.3` có nhanh không?"**
> Không, nếu đứng một mình. Index ANN chỉ tối ưu cho k-NN, còn range scan thuần buộc tính khoảng cách mọi dòng. Phải kèm `ORDER BY khoảng_cách LIMIT k`; cách chuẩn theo tài liệu pgvector là dùng CTE `MATERIALIZED` với điều kiện lọc đặt bên ngoài.

**6. [BẪY] "Index đã tạo rồi mà truy vấn vẫn chậm."**
> Kiểm tra ops class có khớp toán tử không — index `vector_cosine_ops` mà truy vấn dùng `<->` thì index bị bỏ qua âm thầm, không báo lỗi, kết quả vẫn đúng nhưng rơi về seq scan. Xác nhận bằng `EXPLAIN ANALYZE`.

**7. [TRADE-OFF] "Ngưỡng chặt hay lỏng?"**
> Chặt: precision cao, ít rác, nhưng bỏ sót nhiều và hay trả về 0 kết quả — hợp với y tế, pháp lý, tài chính. Lỏng: recall cao, ít bỏ sót, nhưng lẫn rác — hợp với tìm kiếm khám phá và gợi ý nội dung. Chọn theo nhu cầu sản phẩm, và hiệu chỉnh trên golden set chứ không đoán.

**8. [SCALE] "Khối 'sản phẩm tương tự' không được hiện rác thì làm sao?"**
> Threshold cho chất lượng, `LIMIT` cho số lượng, lọc nghiệp vụ theo danh mục và tồn kho, bật `iterative_scan` để chống overfiltering, lấy top rộng rồi mới cắt. Và nếu sau khi lọc còn 0 kết quả thì **ẩn hẳn khối đó** thay vì hạ ngưỡng để lấp chỗ — đây là quyết định sản phẩm cần thống nhất với PM.

**9. "ANN cộng threshold có đảm bảo đúng tuyệt đối không?"**
> Không. ANN là tìm gần đúng nên có thể bỏ sót bản ghi thật sự nằm trong ngưỡng. Nếu cần đảm bảo, dùng truy hồi hai tầng: ANN lấy top-N rộng, rồi tính khoảng cách chính xác và áp ngưỡng trên tập nhỏ đó.

**10. "Làm sao biết chất lượng tìm kiếm đang tệ đi?"**
> Đo recall trên golden set định kỳ. Đây là chỉ số hay bị bỏ quên nhất vì recall tụt **không gây lỗi, không làm chậm, không có cảnh báo** — hệ thống vẫn trả đủ kết quả và mọi biểu đồ vẫn xanh. Kèm theo đó: theo dõi phân bố khoảng cách để phát hiện drift, và tỉ lệ truy vấn trả về 0 kết quả.

### 5.6. Câu trả lời "one-liner" đắt giá

- *"`ORDER BY distance LIMIT k` is the backbone of every vector query — ascending, because smaller distance means closer."*
- *"`<=>` returns cosine distance, not similarity; similarity is `1 - (embedding <=> q)`."*
- *"LIMIT caps how many; the threshold guarantees how good — the nearest results are not automatically good ones."*
- *"A bare `WHERE distance < x` bypasses the ANN index entirely; always pair it with `ORDER BY distance LIMIT k`."*
- *"A threshold is a decision boundary you calibrate on labeled pairs, not a number you guess."*
- *"When finding items similar to an existing row, exclude the row itself — its distance to itself is zero."*
- *"If your similar and dissimilar pairs overlap, no threshold will save you — the problem is the embedding model."*
- *"Recall failures are silent: nothing errors, nothing slows down, the results just quietly get worse."*

---

## 📌 Ghi chú cuối

**Bốn điểm đính chính cần nhớ đúng** (đây là những chỗ hay thành câu hỏi bẫy):
1. `<=>` là cosine **distance**, không phải similarity.
2. Ngưỡng **phụ thuộc thước đo và bộ dữ liệu** — không có con số vạn năng, và ngưỡng của L2 áp sang cosine là vô dụng.
3. Ngưỡng thuần **không dùng được index** — luôn kèm `ORDER BY ... LIMIT k`.
4. Nhớ `WHERE id != :self` khi tìm giống một bản ghi có sẵn.

**Kiểm chứng khi ôn:** đọc README chính thức của pgvector để nắm bảng toán tử, ops class, và các mẫu truy vấn khuyến nghị — đặc biệt phần về CTE materialize khi lọc theo khoảng cách và về iterative scan (pgvector 0.8 trở lên).

**Thực hành — phần quan trọng nhất.** Với một bảng `reviews(embedding vector(384))` đã nạp dữ liệu:

1. Chạy cả ba toán tử `<->`, `<#>`, `<=>` để tìm giống review id=1, rồi **so ba danh sách kết quả** — bạn sẽ tận mắt thấy chúng khác nhau, đúng như ví dụ chạy tay ở mục 1.5.
2. Thêm cột `1 - (embedding <=> :q)` để thấy cùng một cặp cho hai con số ngược nhau.
3. **Cố tình quên `WHERE id != 1`** để thấy chính nó nhảy lên đầu bảng với khoảng cách 0.
4. Thử ngưỡng chặt (0.15) rồi ngưỡng lỏng (0.8) trên cùng câu truy vấn, đếm số kết quả mỗi lần — cảm nhận sự đánh đổi precision/recall bằng con số thật.
5. Chạy `EXPLAIN ANALYZE` cho cả ba dạng ở mục 3.5 để **nhìn thấy `Index Scan` biến mất** khi bạn bỏ `ORDER BY` và `LIMIT`.
6. Tự dựng một golden set mini gồm 20 cặp, đo phân bố hai nhóm, rồi chọn ngưỡng. Đây là bài tập giá trị nhất trong danh sách.

**Nối mạch series:** tạo vector → đánh index → cài đặt → nạp dữ liệu → **truy vấn (bài này)** → tìm theo từ khóa → chọn nền tảng. Tới đây bạn đã **viết được** truy vấn semantic search đúng, nhanh, và có kiểm soát chất lượng.

**Học tiếp gì sau bài này:**
- **Reranking** — dùng một mô hình chính xác hơn (cross-encoder) để xếp hạng lại top-N sau khi truy hồi, nâng chất lượng ở đỉnh danh sách.
- **Query rewriting / expansion** — biến đổi câu hỏi của người dùng trước khi embed để cải thiện kết quả.
- **Đánh giá truy hồi có hệ thống** — precision@k, recall@k, nDCG trên golden set; đây là bộ chỉ số chuẩn để so sánh mọi thay đổi.
