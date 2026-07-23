# Vector Embeddings: Biến chữ thành số & đo độ giống nhau
## Giáo trình SIÊU DỄ HIỂU — Basic → Staff Engineer

> **Nguồn gốc:** Video *"Vector embeddings in relational databases"* — Course 3, IBM Vector Database Fundamentals.
>
> **Cách đọc giáo trình này:** Tôi sẽ đi rất chậm. Mỗi khi có một từ chuyên ngành hoặc từ tiếng Anh xuất hiện lần đầu, tôi dừng lại giải thích ngay tại chỗ rồi mới đi tiếp. Nếu bạn thấy chỗ nào dài dòng quá so với trình độ của mình, cứ nhảy qua — nhưng nếu bạn là người mới, hãy đọc tuần tự, đừng nhảy cóc.
>
> **Quy ước ký hiệu:**
> - 🧩 **[Ngoài bài gốc]** = phần tôi bổ sung thêm, video gốc không nói, nhưng một staff engineer bắt buộc phải biết.
> - ⚠️ **[Đính chính bài gốc]** = chỗ video gốc nói chưa chính xác hoặc đã cũ.
> - 💡 = mẹo/ý quan trọng cần khắc cốt ghi tâm.

---

## ⚠️ Bốn điểm cần đính chính & cập nhật (tính đến tháng 7/2026)

Trước khi vào bài, tôi liệt kê trước những chỗ video gốc đã cũ hoặc nói chưa đúng, để bạn khỏi học nhầm. Đọc lướt qua cũng được, các phần sau tôi sẽ giải thích kỹ từng điểm.

**Một là, tên thư viện JavaScript đã đổi.** Video dùng gói phần mềm tên `@xenova/transformers`. Gói này giờ đã đổi tên thành `@huggingface/transformers` và đã lên **phiên bản 4** (ra mắt tháng 2/2026, thêm khả năng chạy trên card đồ hoạ trong trình duyệt). Tên cũ vẫn cài được nhưng đã ngừng cập nhật — đi phỏng vấn mà nói tên mới là một điểm cộng nhỏ nhưng có thật.

**Hai là, câu "trong toán học vector chỉ có tối đa 3 chiều" trong video là SAI.** Toán học có không gian vector với số chiều tuỳ ý — 10 chiều, 384 chiều, 3072 chiều đều hợp lệ. Cái bị giới hạn ở 3 chiều là *thị giác của con người*, không phải toán học. Tôi sẽ giải thích kỹ ở Mục 1.4.

**Ba là, câu "cosine similarity chạy từ 0 đến 1" trong video cũng chưa chính xác.** Về mặt toán học nó nằm trong khoảng **từ −1 đến 1**. Nó chỉ rơi vào khoảng 0 đến 1 trong một trường hợp riêng. Đây là một câu hỏi bẫy rất hay gặp khi phỏng vấn — tôi sẽ giải thích ở Mục 1.6.

**Bốn là, danh sách model trong video đã cũ.** Video nhắc Word2Vec, GloVe, Paragram, Universal Sentence Encoder — đây là các model *lịch sử*, giá trị để hiểu nguyên lý nhưng gần như không ai dùng cho hệ thống mới năm 2026 nữa. Bảng cập nhật ở Mục 2.3.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### Bài này dạy gì, nói bằng một câu

**Bài này dạy bạn cách biến một câu chữ thành một dãy số, sao cho hai câu có ý nghĩa giống nhau sẽ cho ra hai dãy số "gần nhau" — rồi cách đo xem chúng gần nhau đến mức nào.**

Nếu trong câu trên có từ nào bạn thấy mơ hồ (kiểu "gần nhau là gần thế nào?", "dãy số thì liên quan gì đến nghĩa?"), thì đó là hoàn toàn bình thường và đúng như dự đoán. Phần 1 sinh ra chính là để lấp những chỗ đó. Bạn không cần biết gì trước khi bắt đầu.

### Nó giải quyết nỗi đau gì

Máy tính không "hiểu" chữ. Với máy tính, chữ chỉ là một dãy ký tự — nó biết so sánh xem hai dãy ký tự có giống hệt nhau không, nhưng không biết "áo thun" và "áo phông" là cùng một thứ.

Nhưng rất nhiều tính năng hiện đại đòi hỏi máy phải *hiểu nghĩa*: ô tìm kiếm thông minh trong website bán hàng, tính năng "sản phẩm tương tự", chatbot trả lời dựa trên tài liệu công ty. Muốn làm được, ta phải tìm cách chuyển *ý nghĩa* thành *con số*, vì con số thì máy tính tính toán và so sánh được.

**Embedding chính là cây cầu đó.** Nó là kỹ thuật biến ý nghĩa thành số, theo quy tắc: nội dung giống nghĩa thì cho ra những dãy số nằm gần nhau trong không gian. Và "gần nhau" là thứ đo được bằng một công thức toán rất đơn giản mà bạn sẽ tính bằng tay ở Mục 1.7.

### Học xong bạn sẽ làm được

- Giải thích được embedding là gì bằng lời của mình, và vì sao "khoảng cách giữa hai vector" lại phản ánh được "độ giống nghĩa".
- Phân biệt được hai thế hệ embedding: **static** (tĩnh) và **contextual** (theo ngữ cảnh) — đây là câu trả lời làm người phỏng vấn gật đầu.
- Viết code thật tạo ra embedding bằng ba con đường: JavaScript chạy ngay trong trình duyệt, JavaScript với TensorFlow, và Python (con đường chuẩn của ngành).
- Tính cosine similarity bằng tay trên giấy, và tự viết lại hàm đó từ số 0 bằng code (interviewer rất hay bắt viết).
- Chọn được model embedding phù hợp cho một dự án thật, và biết vì sao việc *đổi model sau này* lại là một quyết định kiến trúc nặng ký.
- Trả lời được một câu hỏi system design về pipeline embedding ở quy mô hàng chục triệu tài liệu.

### Mạch kiến thức: đi từ đâu đến đâu

- 🟢 **Basic** — Nỗi đau của tìm kiếm theo từ khoá → analogy "bản đồ ý nghĩa" → embedding là gì → ai tạo ra nó → đo độ giống bằng cosine → ví dụ tính tay → code "hello world".
- 🟡 **Intermediate** — Embedding được tạo ra như thế nào (static vs contextual) → bức tranh model năm 2026 → code thật ở ba ngôn ngữ → hai bước bị bỏ quên là pooling và normalize → ba lỗi kinh điển.
- 🔴 **Advanced** — Toán đằng sau cosine và chứng minh ngắn → vì sao phải chuẩn hoá vector → số chiều và kỹ thuật Matryoshka → tự viết lại từ số 0 → các trường hợp biên.
- 🟣 **Staff** — Embedding ở quy mô hàng chục triệu bản ghi → chi phí thật bằng tiền → quy trình chọn model → kế hoạch đổi model không làm sập hệ thống → giám sát, các kiểu hỏng → nói chuyện với sếp không rành kỹ thuật → một câu hỏi system design đầy đủ.
- 🎯 **Cheatsheet** — Bảng từ khoá, ý cốt lõi, code thuộc lòng, câu hỏi phỏng vấn kèm gợi ý trả lời.

### Bài này nằm ở đâu trong bức tranh lớn

Nếu bạn đã học bài **pgvector** trước đó, hãy nhớ lại một luận điểm quan trọng ở đó: *"pgvector không tự sinh ra embedding, phải có một model làm việc đó."*

Bài này chính là chỗ cái model đó xuất hiện. Toàn bộ chuỗi hoàn chỉnh trông như sau:

```
   văn bản
      ↓  ← BÀI NÀY: model biến chữ thành vector (embedding)
   vector số
      ↓  ← Bài pgvector: cất vector vào database và tìm vector gần nhất
   kết quả tìm kiếm theo nghĩa
      ↓
   Semantic search / RAG / recommendation
```

Sơ đồ trên có bốn từ chưa được giải thích. Giải thích luôn ở đây, để không có từ nào trôi qua mà bạn phải đoán:

> **pgvector:** một phần mở rộng cắm thêm vào PostgreSQL, giúp database này lưu được vector và tìm vector gần nhất thật nhanh. Nó **không** tự tạo ra vector — điểm này rất quan trọng, tôi sẽ nói kỹ ở Mục 1.5.
>
> **Semantic search (tìm kiếm ngữ nghĩa):** tìm kiếm theo **ý nghĩa** thay vì theo mặt chữ. Gõ "áo phông" mà vẫn ra "áo thun", vì máy hiểu hai từ này cùng nghĩa — thứ mà tìm kiếm theo từ khoá không làm được.
>
> **RAG (Retrieval-Augmented Generation, "sinh văn bản có tra cứu bổ trợ"):** cách làm chatbot trả lời dựa trên tài liệu riêng của bạn. Nó gồm hai bước. Bước một, khi người dùng hỏi, hệ thống *tra cứu* (retrieval) trong kho tài liệu của công ty để tìm ra vài đoạn liên quan nhất — và nó tra bằng đúng kỹ thuật embedding của bài này. Bước hai, nó đưa các đoạn tìm được cùng câu hỏi cho một model ngôn ngữ lớn để *viết ra* (generation) câu trả lời. Nhờ vậy chatbot trả lời dựa trên tài liệu thật của bạn thay vì bịa ra.
>
> **Recommendation (gợi ý):** tính năng "sản phẩm tương tự", "có thể bạn cũng thích". Cách làm bằng embedding rất trực tiếp: tìm những sản phẩm có vector gần nhất với sản phẩm người dùng đang xem.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người **chưa biết gì** về chủ đề. Đây cũng là phần dài nhất và chậm nhất của giáo trình. Nếu bạn đã có nền, cứ đọc lướt — nhưng đừng bỏ Mục 1.7 (ví dụ tính tay), vì nó là chỗ mọi thứ "khớp" lại.

### 1.1. Nỗi đau: máy tính không đọc được chữ như con người

Hãy tưởng tượng bạn đang làm ô tìm kiếm cho một website bán quần áo. Trong kho hàng có một sản phẩm tên là **"áo thun cotton nam"**.

Một khách hàng gõ vào ô tìm kiếm: **"áo phông nam"**.

Bạn và tôi đều biết ngay đây là cùng một thứ — "áo thun" và "áo phông" chỉ là hai cách gọi khác nhau ở hai miền. Nhưng máy tính thì không. Với máy tính, `"áo thun cotton nam"` và `"áo phông nam"` là hai chuỗi ký tự khác nhau.

> **Chuỗi ký tự (string):** trong lập trình, "string" nghĩa là một dãy các chữ cái, số, dấu cách xếp cạnh nhau, được máy tính lưu như một dãy. Máy tính lưu chữ "áo" không phải như khái niệm cái áo, mà như một dãy mã số đại diện cho từng ký tự. Nó hoàn toàn không biết cái áo là gì.

Nếu bạn viết câu lệnh tìm kiếm truyền thống trong database, ví dụ:

```sql
SELECT * FROM products WHERE name LIKE '%áo phông%';
```

> **SQL:** viết tắt của *Structured Query Language* — ngôn ngữ tiêu chuẩn để "hỏi chuyện" một database quan hệ. Câu lệnh trên đọc là: "lấy tất cả (`SELECT *`) từ bảng `products`, với điều kiện cột `name` có chứa cụm 'áo phông' ở đâu đó".
>
> **Database quan hệ (relational database, viết tắt RDBMS — Relational Database Management System):** loại cơ sở dữ liệu lưu dữ liệu dưới dạng các **bảng** (table) gồm **hàng** (row, mỗi hàng là một bản ghi/record, ví dụ một sản phẩm) và **cột** (column, còn gọi là **field** — mỗi cột là một thuộc tính, ví dụ tên, giá, màu). PostgreSQL, MySQL, SQL Server đều là RDBMS.
>
> **`LIKE '%...%'`:** cách so khớp chuỗi ký tự trong SQL. Dấu `%` nghĩa là "bất cứ thứ gì cũng được ở chỗ này". Nên `'%áo phông%'` nghĩa là "có chứa cụm chữ *áo phông* ở bất kỳ đâu trong tên".

...thì câu lệnh này trả về **không có gì cả**. Bởi vì trong tên sản phẩm không hề tồn tại cụm ký tự "áo phông". Khách hàng thấy trang trắng và bỏ đi. Bạn vừa mất một đơn hàng vì máy tính không biết hai từ đồng nghĩa.

Bạn có thể nghĩ: "vậy dùng full-text search thì sao?"

> **Full-text search (tìm kiếm toàn văn):** kỹ thuật tìm kiếm nâng cao hơn `LIKE`. Nó cắt câu thành các từ, đưa từng từ về dạng gốc (ví dụ tiếng Anh: "running" → "run"), rồi tìm theo từ gốc đó. Dạng gốc này gọi là **lexeme**. Nhờ vậy nó tìm được "run" khi bạn gõ "running".

Full-text search giỏi hơn `LIKE` thật, nhưng nó vẫn thua ở đây. Vì "thun" và "phông" **không cùng một từ gốc** — chúng là hai từ hoàn toàn khác nhau về mặt chữ, chỉ giống nhau về *ý nghĩa*. Full-text search làm việc ở tầng **từ**, không phải tầng **nghĩa**.

Đây chính là bức tường mà mọi kỹ thuật tìm kiếm dựa trên chữ đều đâm phải:

| Kỹ thuật | Làm việc ở tầng | "áo thun" ≈ "áo phông"? |
|---|---|---|
| `LIKE` | ký tự | ❌ Không |
| Full-text search | từ (lexeme) | ❌ Không |
| **Embedding** | **ý nghĩa** | ✅ Có |

Muốn phá bức tường đó, ta cần một cách biểu diễn *ý nghĩa* mà máy tính tính toán được. Và cách đó là **embedding**.

### 1.2. Analogy: tấm bản đồ ý nghĩa

Trước khi đưa ra định nghĩa, tôi muốn bạn có hình ảnh trong đầu đã. Định nghĩa mà không có hình ảnh thì học xong quên ngay.

Hãy tưởng tượng một **tấm bản đồ khổng lồ**. Nhưng đây không phải bản đồ địa lý — nó là **bản đồ của ý nghĩa**.

Trên tấm bản đồ này, mỗi từ, mỗi câu, mỗi đoạn văn được đặt vào một vị trí. Và có đúng một quy tắc chi phối việc đặt vị trí:

> **Cái gì giống nghĩa nhau thì được đặt gần nhau. Cái gì khác nghĩa nhau thì bị đặt xa nhau.**

Nghĩa là trên tấm bản đồ đó, "áo thun", "áo phông", "T-shirt", "áo cotton ngắn tay" sẽ nằm túm tụm lại thành một cụm nhỏ ở một góc. Trong khi "xe máy", "ô tô", "xe hơi" nằm túm tụm ở một góc hoàn toàn khác, cách rất xa cụm áo.

Còn "mèo" và "chó" thì sao? Chúng nằm cách nhau một khoảng vừa phải — không sát nhau như "áo thun/áo phông" (vì mèo không phải chó), nhưng gần nhau hơn nhiều so với khoảng cách từ "mèo" đến "ô tô" (vì cả hai đều là thú cưng bốn chân).

Bây giờ, mấu chốt: **trên một tấm bản đồ, mỗi vị trí đều có toạ độ.** Trên bản đồ địa lý, toạ độ là hai con số (kinh độ, vĩ độ). Trên bản đồ ý nghĩa này, toạ độ cũng là một bộ số — chỉ là nhiều số hơn, có thể vài trăm số.

**Bộ số toạ độ đó chính là embedding.** Đó là toàn bộ ý tưởng.

Và giờ phép ví von "khớp" hoàn toàn với kỹ thuật, ở cả bốn điểm:

| Trên bản đồ | Trong kỹ thuật |
|---|---|
| Toạ độ của một điểm | Embedding của một câu |
| Số toạ độ cần để định vị (2 với bản đồ giấy) | Số chiều (dimension) của embedding — thường 384–3072 |
| Hai điểm gần nhau trên bản đồ | Hai câu giống nghĩa |
| Đo khoảng cách giữa hai điểm | Tính cosine similarity giữa hai embedding |
| Người vẽ bản đồ, quyết định đặt cái gì ở đâu | Model AI đã được huấn luyện |

Câu cuối cùng của bảng là điều quan trọng nhất và cũng hay bị bỏ qua nhất: **phải có ai đó vẽ ra tấm bản đồ này**. Bản đồ không tự sinh ra. Người vẽ nó là một model AI, và tôi sẽ nói về "người vẽ bản đồ" đó ở Mục 1.5.

### 1.3. Định nghĩa: vector là gì

Từ "vector" nghe rất toán học và làm nhiều người sợ. Nhưng trong ngữ cảnh này nó cực kỳ đơn giản.

> **Vector:** một danh sách các con số xếp theo thứ tự. Chấm hết. Không có gì bí ẩn hơn.

Ví dụ, đây là một vector 3 chiều: `[2, 5, -1]`. Đây là một vector 5 chiều: `[0.3, -0.8, 0.1, 0.9, 0.02]`.

Trong lập trình, thứ này thường được gọi là **array** (mảng) hoặc **list** (danh sách) — cùng một khái niệm, khác cái tên tuỳ ngôn ngữ.

Hai điều duy nhất bạn cần nhớ về vector ở tầng basic:

Thứ nhất, **thứ tự có ý nghĩa**. Vector `[1, 2]` khác vector `[2, 1]`. Đây không phải một cái túi đựng số lộn xộn — vị trí thứ nhất luôn mang một loại thông tin, vị trí thứ hai mang loại khác.

Thứ hai, **vector có thể hiểu như một mũi tên chỉ hướng, hoặc như một điểm trên bản đồ**. Vector `[2, 1]` có thể hiểu là "điểm nằm ở vị trí đi sang phải 2, đi lên 1", hoặc "mũi tên xuất phát từ gốc, chỉ về hướng phải-chếch-lên". Hai cách hiểu này tương đương nhau và đều hữu ích — cách "điểm" giúp bạn hình dung bản đồ, cách "mũi tên" sẽ cần đến ở Mục 1.6 khi ta nói về góc.

### 1.4. Định nghĩa: embedding và số chiều

Giờ ta ghép hai mảnh lại.

> **Embedding (còn gọi đầy đủ là *vector embedding*):** một vector — tức một dãy số — biểu diễn **ý nghĩa** của một đối tượng nào đó. Nghĩa đen của từ "embed" trong tiếng Anh là *"nhúng vào"*: ta đang "nhúng" một câu chữ vào trong một không gian số.
>
> Đối tượng ở đây không nhất thiết là chữ. Nó có thể là một câu, một đoạn văn, một tấm ảnh, một đoạn video, một file âm thanh, thậm chí số liệu từ cảm biến. Miễn là có model biết cách xử lý loại đó.

Ví dụ cụ thể, rút gọn cho dễ nhìn:

```
"áo thun cotton"  →  [ 0.21, -0.87,  0.44,  0.03, ... ]   (384 số)
"áo phông"        →  [ 0.19, -0.83,  0.47,  0.05, ... ]   (384 số)  ← rất giống ở trên
"xe máy Honda"    →  [-0.62,  0.11, -0.35,  0.88, ... ]   (384 số)  ← khác hẳn
```

Hãy nhìn kỹ ba dòng trên. Hai dòng đầu có các con số gần giống nhau ở từng vị trí (0.21 vs 0.19, −0.87 vs −0.83...). Dòng thứ ba có các con số lệch hẳn. Đó chính xác là cái mà "gần nhau trên bản đồ ý nghĩa" trông như thế nào khi viết ra dưới dạng số.

> **Dimension (số chiều):** đơn giản là **độ dài của cái dãy số đó** — nó gồm bao nhiêu số. Mỗi model tạo ra embedding với số chiều cố định riêng của nó.

Vài con số thực tế bạn nên nhớ:

| Model | Số chiều |
|---|---|
| `all-MiniLM-L6-v2` (model nhẹ, hay dùng để học) | 384 |
| TensorFlow Universal Sentence Encoder | 512 |
| OpenAI `text-embedding-3-small` | 1536 |
| OpenAI `text-embedding-3-large` | 3072 |
| Google Gemini Embedding | 3072 |

⚠️ **[Đính chính bài gốc]** Đây là chỗ video gốc nói sai, và nó gây hiểu lầm khá nghiêm trọng nên tôi giải thích kỹ.

Video nói đại ý: *"trong toán học, vector giới hạn ở 3 chiều, nhưng trong AI thì nhiều chiều hơn."* Câu này ngược hoàn toàn với sự thật.

Toán học **chưa bao giờ** giới hạn vector ở 3 chiều. Không gian vector *n* chiều — với *n* là số nguyên dương bất kỳ — là khái niệm chuẩn mực, đã được xây dựng chặt chẽ từ thế kỷ 19, rất lâu trước khi có máy tính. Bạn hoàn toàn có thể làm toán trong không gian 384 chiều hay 1 triệu chiều, mọi công thức đều hoạt động bình thường.

Cái *thực sự* bị giới hạn ở 3 chiều là **trí tưởng tượng thị giác của con người**. Chúng ta sống trong không gian 3 chiều nên não không hình dung nổi 4 chiều trở lên. Nhưng "không hình dung được" hoàn toàn khác với "không tính được".

💡 **Cách để không bị rối:** đừng cố tưởng tượng không gian 384 chiều — bạn sẽ không làm được và cũng không cần. Hãy tưởng tượng trong 2 hoặc 3 chiều để lấy trực giác, rồi tin rằng công thức toán mở rộng lên 384 chiều một cách hoàn toàn máy móc. Chẳng hạn công thức tính khoảng cách trong không gian 2 chiều là `sqrt(x² + y²)` (định lý Pythagoras bạn học hồi cấp 2). Lên 384 chiều, nó chỉ là `sqrt(x₁² + x₂² + ... + x₃₈₄²)` — cùng một công thức, cộng thêm nhiều số hạng. Không có phép màu nào ở đây cả.

### 1.5. Ai tạo ra embedding: model

Ở Mục 1.2 tôi có nói "phải có ai đó vẽ ra tấm bản đồ ý nghĩa". Giờ ta nói về người vẽ đó.

> **Model (mô hình):** trong machine learning, "model" là một chương trình đã *học* được cách làm một việc gì đó từ dữ liệu, thay vì được lập trình thủ công từng bước.
>
> **Machine learning (học máy):** cách làm phần mềm trong đó ta không viết ra luật cụ thể, mà đưa cho máy rất nhiều ví dụ rồi để nó tự rút ra quy luật. Ví dụ: thay vì viết luật "email có chữ 'trúng thưởng' là spam", ta đưa cho máy 1 triệu email đã dán nhãn spam/không spam và để nó tự tìm ra dấu hiệu.
>
> **Pretrained (đã huấn luyện sẵn):** model đó đã được người khác — thường là Google, OpenAI, Meta, hoặc cộng đồng mã nguồn mở — huấn luyện xong trên hàng tỉ mẫu dữ liệu, tốn hàng triệu đô tiền điện và card đồ hoạ. Bạn chỉ việc **tải về và dùng**. Bạn không tự huấn luyện model embedding, trừ khi bạn làm ở một lab nghiên cứu.

Cách một model embedding học được "bản đồ ý nghĩa" nghe rất khó tin nhưng lại rất đơn giản về nguyên lý: nó đọc lượng văn bản khổng lồ (gần như toàn bộ internet) và học theo nguyên tắc **"những từ hay xuất hiện trong ngữ cảnh giống nhau thì có nghĩa giống nhau"**.

Nghĩ mà xem: trong hàng tỉ câu tiếng Việt trên mạng, từ "áo thun" và "áo phông" hầu như luôn xuất hiện cạnh những từ giống hệt nhau — "mặc", "size", "cotton", "ngắn tay", "giặt máy". Còn "xe máy" thì xuất hiện cạnh "xăng", "biển số", "đội mũ bảo hiểm". Model không cần ai dạy nó "áo thun là áo phông"; nó tự rút ra điều đó từ việc quan sát hai từ này luôn được dùng trong cùng hoàn cảnh. Đây là một ý tưởng cũ trong ngôn ngữ học, thường được tóm gọn: *"bạn hiểu một từ qua những từ đứng cạnh nó"*.

💡 **Đây là câu quan trọng nhất của cả bài, nhắc lại lần nữa vì nó sẽ quay lại ở Phần 4:**

> **MODEL tạo ra embedding. DATABASE chỉ lưu và tìm nó.**
>
> Đây là hai công việc hoàn toàn tách biệt, do hai thành phần khác nhau đảm nhiệm.

> **pgvector:** một extension (phần mở rộng) cắm thêm vào PostgreSQL, cho phép database này lưu được cột kiểu vector và tìm vector gần nhất một cách nhanh chóng. Nó **không** sinh ra embedding — bạn phải tự gọi model, lấy vector, rồi mới đưa vector đó vào cho pgvector cất giữ.

Analogy để nhớ: **model là đầu bếp, database là cái tủ lạnh.** Đầu bếp nấu ra món ăn; tủ lạnh cất món ăn và giúp bạn lấy ra nhanh khi cần. Bảo cái tủ lạnh tự nấu ăn là vô lý — nhưng đây lại là hiểu nhầm cực kỳ phổ biến của người mới, và là một câu hỏi phỏng vấn hay gặp.

### 1.6. Đo "gần nhau" như thế nào: cosine similarity (trực giác trước)

Ta đã có embedding. Giờ làm sao biết hai embedding có gần nhau không?

Cách phổ biến nhất trong ngành gọi là **cosine similarity**.

> **Cosine similarity (độ tương đồng cosin):** một con số đo mức độ giống nhau giữa hai vector, bằng cách đo **góc** giữa chúng.
>
> Từ "cosine" (cosin) là hàm lượng giác bạn học hồi phổ thông. Bạn không cần nhớ gì về lượng giác để hiểu bài này — chỉ cần nhớ một tính chất duy nhất: cosin của một góc cho ta biết hai hướng "chụm" nhau đến mức nào.

Nhớ lại ở Mục 1.3, tôi nói vector có thể hiểu như một **mũi tên chỉ hướng**. Giờ là lúc dùng cách hiểu đó. Hãy hình dung hai mũi tên cùng xuất phát từ một điểm gốc:

- Hai mũi tên **chỉ cùng một hướng** (góc giữa chúng ≈ 0°) → cosine similarity ≈ **1** → *rất giống nhau*.
- Hai mũi tên **vuông góc với nhau** (góc = 90°) → cosine similarity = **0** → *chẳng liên quan gì đến nhau*.
- Hai mũi tên **chỉ ngược hướng nhau** (góc = 180°) → cosine similarity = **−1** → *trái ngược nhau*.

Đây là bảng bạn nên nhớ nằm lòng:

| Góc giữa hai vector | Cosine similarity | Ý nghĩa với văn bản |
|---|---|---|
| 0° (cùng hướng) | 1.0 | Cùng nghĩa |
| ~30° | ~0.87 | Rất liên quan |
| ~60° | ~0.5 | Hơi liên quan |
| 90° (vuông góc) | 0.0 | Không liên quan |
| 180° (ngược hướng) | −1.0 | Đối lập |

Và đây là ý then chốt, cũng là câu chốt hay dùng khi phỏng vấn:

> 💡 **Cosine chỉ quan tâm đến HƯỚNG, hoàn toàn không quan tâm ĐỘ DÀI của mũi tên.**

Vì sao đặc tính này lại đúng là thứ ta cần cho văn bản? Hãy nghĩ về hai câu này:

- Câu A: *"Con mèo đang ngủ."*
- Câu B: *"Con mèo đang ngủ say sưa trên chiếc ghế sofa màu xám ở phòng khách."*

Câu B dài hơn câu A rất nhiều. Trong nhiều cách biểu diễn, câu dài hơn sẽ tạo ra vector có "độ dài" lớn hơn. Nhưng về mặt **chủ đề**, hai câu này nói về cùng một chuyện: con mèo đang ngủ.

Cosine bỏ qua độ dài và chỉ so hướng, nên nó nhận ra hai câu này cùng chủ đề. Nếu ta dùng một phép đo có tính đến độ dài, nó có thể kết luận sai rằng hai câu này khác nhau nhiều — chỉ vì một câu dài hơn câu kia. Đó là lý do cosine là lựa chọn mặc định cho văn bản.

⚠️ **[Đính chính bài gốc]** Video nói cosine similarity chạy "từ 0 đến 1". Chính xác hơn: **miền giá trị toán học của nó là từ −1 đến 1**, vì cosin của bất kỳ góc nào cũng nằm trong khoảng đó.

Nó chỉ rơi vào khoảng từ 0 đến 1 trong một trường hợp riêng: khi **mọi thành phần của cả hai vector đều không âm** (không có số âm nào). Khi đó hai mũi tên không bao giờ chỉ ngược hướng nhau được, nên góc luôn ≤ 90°.

Embedding hiện đại **có** chứa số âm (nhìn lại ví dụ ở Mục 1.4, có `-0.87`), nên về lý thuyết cosine hoàn toàn có thể ra giá trị âm. Trên thực tế, các câu văn bản thật hiếm khi cho ra giá trị âm mạnh — nhưng câu "cosine luôn nằm trong [0,1]" vẫn là **sai về mặt toán học**, và đây là câu hỏi bẫy rất hay gặp khi phỏng vấn. Nhớ trả lời **[−1, 1]**.

### 1.7. Ví dụ chạy tay — tính cosine similarity bằng bút và giấy

Đây là mục quan trọng nhất Phần 1. Đừng đọc lướt — hãy lấy giấy ra tính theo. Khi bạn tự tính được một lần, cosine similarity vĩnh viễn không còn đáng sợ nữa.

Để tính được bằng tay, tôi dùng vector chỉ **2 chiều** thay vì 384 chiều. Toán học hoàn toàn giống nhau, chỉ là ít số hạng hơn. Các số dưới đây là tôi bịa ra cho dễ tính, không phải embedding thật.

**Công thức:**

```
                        A · B
cosine_similarity(A, B) = ───────────
                        |A| × |B|

trong đó:
    A · B  =  a₁×b₁ + a₂×b₂          ← gọi là DOT PRODUCT (tích vô hướng)
    |A|    =  sqrt(a₁² + a₂²)         ← gọi là MAGNITUDE / NORM (độ dài của vector)
```

> **Dot product (tích vô hướng, ký hiệu bằng dấu chấm `·`):** nhân từng cặp số ở cùng vị trí của hai vector, rồi cộng tất cả lại thành một số duy nhất. Với `[2,1]` và `[4,2]`: nhân cặp thứ nhất được 2×4=8, nhân cặp thứ hai được 1×2=2, cộng lại được 10.
>
> **Magnitude / norm (độ dài của vector, ký hiệu bằng hai vạch đứng `|A|`):** độ dài của mũi tên, tính bằng định lý Pythagoras — bình phương từng số, cộng lại, rồi lấy căn bậc hai. Ký hiệu `sqrt` là *square root*, tức căn bậc hai.

**Dữ liệu ví dụ.** Giả sử ba từ có embedding 2 chiều như sau:

- **A = "mèo" = (2, 1)**
- **B = "miu" = (4, 2)** ← từ đồng nghĩa của "mèo"
- **C = "ô tô" = (−1, 3)** ← chẳng liên quan gì

---

**Bước 1 — Tính similarity giữa A ("mèo") và B ("miu"):**

Tính dot product:
```
A · B = 2×4 + 1×2
      = 8   + 2
      = 10
```

Tính độ dài của A:
```
|A| = sqrt(2² + 1²) = sqrt(4 + 1) = sqrt(5) ≈ 2.236
```

Tính độ dài của B:
```
|B| = sqrt(4² + 2²) = sqrt(16 + 4) = sqrt(20) ≈ 4.472
```

Ghép vào công thức:
```
cosine = 10 / (2.236 × 4.472)
       = 10 / 10.0
       = 1.00
```

**Kết quả: 1.00 — giống nhau hoàn toàn.** ✅

Vì sao lại đúng bằng 1? Hãy nhìn lại: B = (4, 2) chính là **A nhân đôi**, vì (2×2, 1×2) = (4, 2). Nghĩa là hai mũi tên chỉ **đúng cùng một hướng**, chỉ khác nhau ở độ dài — B dài gấp đôi A.

Và đây là chỗ bạn *nhìn thấy tận mắt* điều tôi nói ở Mục 1.6: **cosine không quan tâm độ dài.** Mũi tên B dài gấp đôi mũi tên A, nhưng kết quả vẫn là 1.00 tuyệt đối, vì phép chia cho `|A| × |B|` đã triệt tiêu hết ảnh hưởng của độ dài. Chỉ còn lại hướng.

---

**Bước 2 — Tính similarity giữa A ("mèo") và C ("ô tô"):**

Tính dot product:
```
A · C = 2×(−1) + 1×3
      = −2     + 3
      = 1
```

Tính độ dài của C:
```
|C| = sqrt((−1)² + 3²) = sqrt(1 + 9) = sqrt(10) ≈ 3.162
```

Ghép vào công thức (`|A|` ta đã tính ở trên là 2.236):
```
cosine = 1 / (2.236 × 3.162)
       = 1 / 7.07
       ≈ 0.14
```

**Kết quả: 0.14 — gần như không liên quan.** ✅

Con số 0.14 rất gần 0, và ở Mục 1.6 ta biết cosine ≈ 0 nghĩa là hai mũi tên gần vuông góc, tức là "chẳng liên quan gì". Đúng như kỳ vọng: "mèo" và "ô tô" không liên quan.

---

**Tổng kết ví dụ chạy tay:**

| Cặp | Dot product | Cosine | Diễn giải |
|---|---|---|---|
| mèo – miu | 10 | **1.00** | Đồng nghĩa ✅ |
| mèo – ô tô | 1 | **0.14** | Không liên quan ✅ |

💡 **Điều quan trọng nhất cần nhận ra:** phép tính bạn vừa làm bằng tay **chính xác là phép tính** mà pgvector chạy bên trong khi bạn viết một câu truy vấn tìm kiếm ngữ nghĩa. Khác biệt duy nhất là nó làm với 384 hoặc 1536 chiều thay vì 2 chiều, và làm hàng triệu lần mỗi giây. **Bản chất toán học không đổi một chút nào.**

Nếu bạn hiểu được ví dụ này, bạn đã hiểu cái lõi của toàn bộ ngành vector search.

### 1.8. Code "hello world": tạo embedding thật và đo similarity

Giờ ta chạy code thật. Ví dụ này dùng JavaScript, chạy được ngay trên máy bạn (hoặc thậm chí trong trình duyệt), **không cần API key, không tốn tiền**.

> **API key (khoá truy cập):** một chuỗi ký tự bí mật giống mật khẩu, dùng để chứng minh bạn là ai khi gọi dịch vụ của một công ty qua mạng, và để họ tính tiền bạn. Ví dụ dưới đây chạy model *ngay trên máy bạn* nên không cần thứ này.

**Chuẩn bị:** cài thư viện bằng lệnh sau trong terminal.

```bash
npm install @huggingface/transformers
```

> **npm:** viết tắt của *Node Package Manager* — công cụ cài đặt thư viện cho JavaScript. Gõ `npm install <tên>` là nó tải thư viện đó về dự án của bạn.
>
> **Thư viện (library) / gói (package):** code do người khác viết sẵn mà bạn dùng lại, thay vì viết lại từ đầu.
>
> **Hugging Face:** một công ty và nền tảng cộng đồng, đóng vai trò như "GitHub của giới AI" — nơi mọi người chia sẻ hàng trăm nghìn model đã huấn luyện sẵn để tải về dùng miễn phí.
>
> **Transformers.js:** thư viện của Hugging Face cho phép chạy model AI **trực tiếp bằng JavaScript** — trên máy bạn hoặc trong trình duyệt của người dùng, không cần server. Tên gói cũ là `@xenova/transformers` (video gốc dùng tên này), tên mới là `@huggingface/transformers`, hiện đã ở phiên bản 4.

**Code:**

```javascript
// Nhập hai thứ từ thư viện: hàm pipeline (tạo "dây chuyền" xử lý)
// và hàm cos_sim (tính cosine similarity có sẵn)
import { pipeline, cos_sim } from '@huggingface/transformers';

// ─── BƯỚC 1: Nạp model ───────────────────────────────────────────
// pipeline() tạo ra một "dây chuyền xử lý" hoàn chỉnh: nó lo hết
// mọi thứ từ cắt chữ thành token cho tới chạy model và trả về số.
const extractor = await pipeline(
  'feature-extraction',          // TÊN CÔNG VIỆC: "trích xuất đặc trưng"
                                 // = tên gọi chính thức của việc tạo embedding
  'Xenova/all-MiniLM-L6-v2'      // TÊN MODEL: nhẹ (~23MB), 384 chiều, chạy được cả trong browser
);
// Lần chạy đầu tiên sẽ tải model từ mạng về (mất vài giây).
// Các lần sau nó lấy từ bộ nhớ đệm trên máy, chạy ngay lập tức.

// ─── BƯỚC 2: Biến câu thành vector ───────────────────────────────
// pooling:'mean' và normalize:true là HAI THAM SỐ BẮT BUỘC.
// Thiếu chúng, bạn KHÔNG nhận được vector của cả câu.
// Lý do chi tiết ở Mục 2.5 — đây là lỗi số 1 của người mới.
const a = await extractor('con mèo đang ngủ', {
  pooling: 'mean',      // gộp vector của từng từ thành MỘT vector cho cả câu
  normalize: true       // co vector về độ dài đúng bằng 1
});

const b = await extractor('chú miu đang thiu thiu', {
  pooling: 'mean',
  normalize: true
});

const c = await extractor('giá xăng hôm nay tăng', {  // câu không liên quan, để đối chứng
  pooling: 'mean',
  normalize: true
});

// ─── BƯỚC 3: Nhìn xem ta vừa nhận được cái gì ────────────────────
console.log(a.dims);           // [1, 384]  → 1 câu, mỗi câu 384 chiều
console.log(a.data.length);    // 384       → đúng 384 con số
console.log(a.data.slice(0, 5)); // 5 số đầu, ví dụ: [0.031, -0.084, 0.012, ...]

// ─── BƯỚC 4: Đo độ giống nhau ────────────────────────────────────
console.log(cos_sim(a.data, b.data));   // ~0.65  → hai câu cùng nói về mèo ngủ
console.log(cos_sim(a.data, c.data));   // ~0.05  → chẳng liên quan gì
```

**Đọc lại kết quả để thấy phép màu:** hai câu *"con mèo đang ngủ"* và *"chú miu đang thiu thiu"* **không có một từ nào chung** (bỏ qua từ "đang"). Với `LIKE` hay full-text search ở Mục 1.1, chúng hoàn toàn không khớp. Nhưng cosine similarity cho ra ~0.65 — cao rõ rệt so với 0.05 của câu về giá xăng. Máy tính vừa nhận ra hai câu này cùng nghĩa **mà không cần từ chung nào**. Đó chính là bức tường mà ta đã phá được.

**Về `a.data`:** kiểu dữ liệu của nó là `Float32Array` — một mảng các số thực dùng 32 bit mỗi số.

> **Float (số thực dấu chấm động):** kiểu số có phần thập phân, ví dụ 0.031. Con số 32 nghĩa là mỗi số chiếm 32 bit = 4 byte trong bộ nhớ. Nhân lên: một embedding 384 chiều tốn 384 × 4 = **1536 byte ≈ 1.5 KB**. Con số này nhìn nhỏ, nhưng nhân với 100 triệu tài liệu thì thành 150 GB — và đó là lúc nó trở thành vấn đề kiến trúc, sẽ bàn ở Phần 4.

### 1.9. 🧩 [Ngoài bài gốc] Hai điều video gốc chạm tới nhưng chưa nói hết

**Điều thứ nhất: embedding không đọc hiểu được, và điều đó hoàn toàn ổn.**

Video gốc có một nhận xét rất đúng: embedding "không tồn tại ở dạng con người có thể suy luận được". Tôi muốn nói rõ hơn vì sao.

Nếu bạn nhìn vào chiều thứ 47 của một embedding và thấy giá trị 0.31, thì con số đó **không có ý nghĩa riêng** nào cả. Nó không phải "độ vui vẻ" hay "mức độ liên quan đến động vật". Model tự sắp xếp thông tin theo cách riêng của nó trong quá trình huấn luyện, và không ai — kể cả người tạo ra model — biết chính xác từng chiều mã hoá cái gì.

Chỉ có **toàn bộ 384 con số cùng nhau** mới mang ý nghĩa, và ý nghĩa đó chỉ bộc lộ khi bạn **so sánh** hai embedding với nhau.

💡 Câu chốt cần nhớ: **bạn không đọc embedding, bạn so sánh embedding.** Đừng phí thời gian cố diễn giải từng con số.

**Điều thứ hai: đây là lý do vector search tồn tại như một ngành riêng.**

Video có nhắc rằng nếu bạn tự làm mọi thứ bằng SQL thuần — lấy toàn bộ vector ra khỏi database, tính cosine với từng cái một, rồi sắp xếp — thì sẽ **rất tốn bộ nhớ và rất chậm**. Nhận xét này hoàn toàn đúng và đáng đào sâu.

Hãy làm phép tính. Bạn có 10 triệu sản phẩm, mỗi cái một embedding 384 chiều. Người dùng gõ một từ khoá tìm kiếm. Nếu làm theo cách thủ công:

- Đọc 10 triệu × 1.5 KB = **15 GB** dữ liệu từ đĩa lên bộ nhớ.
- Thực hiện 10 triệu phép tính cosine, mỗi phép gồm ~1150 phép nhân và cộng → khoảng **11.5 tỉ phép tính**.
- Sắp xếp 10 triệu kết quả để lấy ra top 10.

Và phải làm tất cả những việc đó **trong vòng dưới 100 mili giây**, cho *mỗi lượt tìm kiếm*, của *mỗi người dùng*. Điều này bất khả thi.

Đó chính là lý do tồn tại của **ANN index**.

> **ANN:** viết tắt của *Approximate Nearest Neighbor* — "láng giềng gần nhất **xấp xỉ**". Từ khoá quan trọng là **xấp xỉ**: thay vì kiểm tra hết 10 triệu vector để tìm ra đáp án chính xác tuyệt đối, ta chấp nhận bỏ qua phần lớn chúng và có thể sai một chút, đổi lại tốc độ nhanh gấp hàng nghìn lần.
>
> **Index (chỉ mục):** một cấu trúc dữ liệu phụ được xây sẵn để tìm kiếm nhanh — giống như mục lục tra cứu ở cuối một quyển sách dày. Không có mục lục, bạn phải lật từng trang. Có mục lục, bạn tra ra trang cần tìm trong vài giây.

Bài pgvector đi sâu vào chủ đề này. Ở bài này, bạn chỉ cần nhớ mối quan hệ: **bài này lo việc *sản xuất* vector; ANN index lo việc *tìm* vector nhanh.** Hai bài toán khác nhau, hai bộ kỹ thuật khác nhau.

### ✅ Self-check Phần 1

Hãy tự trả lời trước khi xem gợi ý.

**Câu 1.** Cosine similarity đo *hướng* hay *độ dài* của vector? Vì sao đặc tính đó lại phù hợp với văn bản?

<details>
<summary>Gợi ý đáp án</summary>

Đo **hướng**, hoàn toàn bỏ qua độ dài — vì công thức chia cho tích hai độ dài `|A| × |B|`, triệt tiêu ảnh hưởng của chúng. Phù hợp với văn bản vì một câu dài và một câu ngắn có thể nói cùng một chủ đề; ta muốn máy nhận ra *chủ đề giống nhau* chứ không phải phạt câu dài chỉ vì nó dài. Bằng chứng cụ thể: ở Mục 1.7, vector B dài gấp đôi vector A nhưng cosine vẫn ra đúng 1.00.
</details>

**Câu 2.** Hai câu cùng nghĩa cho cosine gần giá trị nào? Hai câu không liên quan thì gần giá trị nào? Miền giá trị đầy đủ của cosine similarity là gì?

<details>
<summary>Gợi ý đáp án</summary>

Cùng nghĩa → gần **1**. Không liên quan → gần **0**. Miền giá trị đầy đủ là **[−1, 1]**, không phải [0, 1] — nó chỉ nằm trong [0,1] khi mọi thành phần của cả hai vector đều không âm. Đây là câu bẫy phỏng vấn.
</details>

**Câu 3.** Vì sao embedding "không đọc hiểu được" mà vẫn cực kỳ hữu dụng?

<details>
<summary>Gợi ý đáp án</summary>

Vì công dụng của nó không nằm ở việc *đọc*, mà ở việc *so sánh*. Từng chiều riêng lẻ không mang ý nghĩa mà con người diễn giải được; chỉ có toàn bộ dãy số cùng nhau mới mã hoá ngữ nghĩa, và ngữ nghĩa đó bộc lộ ra khi ta đo khoảng cách giữa hai embedding. Ta không cần hiểu bản đồ được vẽ theo quy tắc nào — chỉ cần biết hai điểm gần nhau nghĩa là gần nghĩa nhau.
</details>

**Câu 4.** Đúng hay sai: "pgvector nhận vào một câu văn và tự tạo ra embedding cho nó."

<details>
<summary>Gợi ý đáp án</summary>

**Sai.** pgvector chỉ *lưu trữ* và *tìm kiếm* vector. Bạn phải tự gọi một model embedding để biến câu văn thành vector, rồi mới đưa vector đó vào cho pgvector. Model là đầu bếp, database là tủ lạnh.
</details>

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Từ đây tôi giả định bạn đã nắm Phần 1: embedding là dãy số mã hoá ý nghĩa, model tạo ra nó, cosine đo góc giữa hai vector. Phần này trả lời câu hỏi: **model tạo ra embedding bằng cách nào, và làm sao dùng nó cho đúng trong công việc thật.**

### 2.1. Hai thế hệ embedding: static và contextual

Đây là phân biệt quan trọng nhất của cả bài, và cũng là chỗ video gốc gộp chung một cách gây hiểu lầm — nó liệt kê Word2Vec nằm cạnh GPT như thể chúng cùng loại. Chúng **không** cùng loại. Chúng cách nhau cả một thế hệ công nghệ.

#### Thế hệ 1: Static embedding (embedding tĩnh)

> **Static embedding:** cách làm trong đó **mỗi từ có đúng MỘT vector cố định**, không bao giờ thay đổi, bất kể từ đó đang nằm trong câu nào.
>
> Các đại diện: **Word2Vec** (Google, 2013), **GloVe** (Stanford, 2014), **Paragram**. Video gốc nhắc cả ba.

Cách nó hoạt động rất trực tiếp: model học sẵn một bảng tra cứu khổng lồ. Bảng đó có bao nhiêu hàng thì từ điển có bấy nhiêu từ, mỗi hàng là vector của một từ. Muốn lấy embedding của từ "mèo"? Tra bảng, lấy hàng "mèo" ra. Xong.

Cách học của nó là đếm **co-occurrence** — từ nào hay đứng cạnh từ nào.

> **Co-occurrence (đồng xuất hiện):** hiện tượng hai từ hay xuất hiện gần nhau trong văn bản. Model quét hàng tỉ câu và ghi lại thống kê: từ "mèo" thường có "kêu", "meo", "chuột", "lông" ở xung quanh. Từ nào có "hàng xóm" giống nhau thì được gán vector giống nhau.

**Vấn đề chí mạng của static embedding:** nó không phân biệt được từ đồng âm khác nghĩa.

Ví dụ kinh điển trong tiếng Anh là từ **"bank"**:

- *"I sat on the river **bank**"* → bank ở đây là **bờ sông**.
- *"I opened a **bank** account"* → bank ở đây là **ngân hàng**.

Hai nghĩa hoàn toàn khác nhau. Nhưng vì Word2Vec chỉ có một hàng duy nhất cho từ "bank", nó buộc phải trả về **cùng một vector** cho cả hai câu. Kết quả là vector đó bị "trung bình hoá" giữa hai nghĩa — nó không đại diện đúng cho nghĩa nào cả.

Tiếng Việt cũng đầy trường hợp như vậy: "đường" (đường đi / đường ăn), "chín" (số 9 / đã chín), "bàn" (cái bàn / bàn bạc).

Cách nhớ ngắn gọn: **static embedding = "một từ một nghĩa"**, và thực tế ngôn ngữ không hoạt động như vậy.

#### Thế hệ 2: Contextual embedding (embedding theo ngữ cảnh)

> **Contextual embedding:** cách làm trong đó vector của một từ **phụ thuộc vào toàn bộ câu chứa nó**. Cùng một từ, đặt trong hai câu khác nhau, sẽ cho ra hai vector khác nhau.
>
> Các đại diện: **BERT** (Google, 2018), **GPT** (OpenAI), **Universal Sentence Encoder**, và toàn bộ model hiện đại năm 2026. Tất cả đều dựa trên kiến trúc **transformer**.

> **Transformer:** kiến trúc mạng nơ-ron được công bố năm 2017, đứng đằng sau gần như mọi tiến bộ AI của thập kỷ qua (ChatGPT, Claude, Gemini đều là transformer). Điểm cốt lõi của nó là cơ chế **self-attention**.
>
> **Self-attention (tự chú ý):** cơ chế cho phép mỗi từ trong câu "nhìn" tất cả các từ khác và tự quyết định nên chú ý đến từ nào để hiểu chính mình. Khi xử lý từ "bank" trong câu *"I sat on the river bank"*, cơ chế này để từ "bank" nhìn thấy từ "river", tự nhận ra "à, ngữ cảnh này là sông" và điều chỉnh vector của mình về phía nghĩa bờ sông.

Nhờ vậy, contextual embedding giải được đúng bài toán mà static bó tay: hai chữ "bank" trong hai câu trên nhận **hai vector khác nhau**, mỗi vector phản ánh đúng nghĩa trong ngữ cảnh của nó.

Ngoài ra, contextual embedding còn bắt được sắc thái mà static hoàn toàn mù:

- **Phủ định:** *"phim này không dở"* và *"phim này khá hay"* → contextual nhận ra chúng gần nghĩa; static thấy chữ "dở" nên đẩy về phía tiêu cực.
- **Cả câu:** với static, muốn có vector của một câu bạn phải lấy trung bình vector các từ, mất hết thông tin về trật tự và quan hệ. Với transformer, model xử lý cả câu như một thể thống nhất.

#### Bảng so sánh

| | Static (Word2Vec, GloVe) | Contextual (BERT, GPT, model 2026) |
|---|---|---|
| Số vector cho một từ | 1, cố định vĩnh viễn | Vô số, tuỳ ngữ cảnh |
| Từ "bank" hai nghĩa | Cùng 1 vector ❌ | 2 vector khác nhau ✅ |
| Hiểu phủ định, sắc thái | Không | Có |
| Cách lấy vector | Tra bảng (cực nhanh) | Chạy model (chậm hơn nhiều) |
| Chi phí tính toán | Rất thấp | Cao — phải chạy model cho *từng* input |
| Kích thước | Nhỏ | Lớn (hàng trăm MB đến hàng chục GB) |
| Vị thế 2026 | Giá trị lịch sử | Tiêu chuẩn thực tế |

🧩 **[Ngoài bài gốc] — Vì sao mục này đáng giá trong phỏng vấn:**

Khi được hỏi *"embedding là gì?"*, câu trả lời tầm thường là "một mảng số". Ai cũng nói được câu đó và nó không chứng minh gì cả.

Câu trả lời tầm senior/staff là: *"một mảng số mã hoá ngữ nghĩa; và quan trọng hơn, cần phân biệt static embedding — một vector cố định mỗi từ, kiểu Word2Vec — với contextual embedding từ transformer, trong đó vector phụ thuộc ngữ cảnh nhờ self-attention. Đó là lý do transformer thắng: nó tính embedding *có điều kiện theo ngữ cảnh*."*

Câu trả lời thứ hai cho thấy bạn hiểu *lịch sử phát triển* và *cơ chế*, không phải học thuộc định nghĩa.

### 2.2. Từ chữ đến vector: chuyện gì xảy ra bên trong

Ở Mục 1.8 bạn gọi `extractor(text)` và nhận về 384 con số. Bên trong cái hộp đen đó có bốn bước. Hiểu bốn bước này giúp bạn tránh được ba lỗi ở Mục 2.5.

```
   "con mèo đang ngủ"
          │
          ▼  BƯỚC 1: TOKENIZE (cắt thành token)
   ["con", "mèo", "đang", "ngủ"]
          │
          ▼  BƯỚC 2: CHẠY TRANSFORMER
   [vec_con, vec_mèo, vec_đang, vec_ngủ]     ← MỘT vector cho MỖI token!
          │
          ▼  BƯỚC 3: POOLING (gộp lại)
   [vec_cả_câu]                               ← chỉ còn MỘT vector
          │
          ▼  BƯỚC 4: NORMALIZE (chuẩn hoá)
   [vec_cả_câu, độ dài = 1]                   ← sẵn sàng để so sánh
```

**Bước 1 — Tokenize.**

> **Token:** đơn vị nhỏ nhất mà model xử lý. Token *không* hoàn toàn trùng với "từ" — nó có thể là một từ, một phần của từ, hoặc một dấu câu. Ví dụ tiếng Anh, từ "unhappiness" thường bị cắt thành ba token: `un` + `happi` + `ness`. Cách cắt này giúp model xử lý được cả những từ nó chưa từng thấy bao giờ.
>
> **Tokenizer:** bộ phận làm nhiệm vụ cắt. Mỗi model có tokenizer riêng đi kèm — dùng nhầm tokenizer của model khác sẽ cho kết quả sai.

💡 Quy tắc ước lượng thô cần nhớ để tính chi phí: **1 token ≈ 0.75 từ tiếng Anh**, hay nói cách khác, 1000 token ≈ 750 từ. Tiếng Việt tốn nhiều token hơn (thường gấp 1.5–2 lần cùng nội dung) vì các tokenizer chủ yếu được tối ưu cho tiếng Anh. Đây là chi tiết ảnh hưởng trực tiếp đến hoá đơn, sẽ dùng lại ở Phần 4.

**Bước 2 — Chạy transformer.** Đây là chỗ quan trọng nhất và cũng gây bất ngờ nhất cho người mới:

> ⚠️ **Transformer trả về MỘT vector cho MỖI token, không phải một vector cho cả câu.**

Câu 4 token ở trên cho ra **4 vector**, mỗi vector 384 chiều. Nói theo ngôn ngữ kỹ thuật, kết quả có **shape** là `[1, 4, 384]`.

> **Shape (hình dạng):** cách mô tả kích thước của một khối dữ liệu nhiều tầng. `[1, 4, 384]` đọc là: 1 câu, mỗi câu 4 token, mỗi token 384 số.
>
> **Tensor:** tên gọi chung cho khối số nhiều chiều như vậy. Vector là tensor 1 chiều, ma trận là tensor 2 chiều, và `[1, 4, 384]` là tensor 3 chiều. Từ này xuất hiện khắp nơi trong AI nên nên quen mặt.

Nhưng bạn muốn **một** vector đại diện cho cả câu, không phải 4 vector. Nên phải có bước 3.

**Bước 3 — Pooling.**

> **Pooling (gộp):** phép biến nhiều vector thành một vector duy nhất.
>
> **Mean pooling (gộp bằng trung bình):** cách phổ biến nhất — lấy **trung bình cộng** của tất cả vector token, tính riêng cho từng chiều.

Ví dụ minh hoạ với vector 3 chiều cho dễ nhìn:

```
vec_con   = [0.2, 0.8, 0.1]
vec_mèo   = [0.6, 0.4, 0.3]
vec_đang  = [0.1, 0.2, 0.5]
vec_ngủ   = [0.5, 0.6, 0.1]
──────────────────────────────  cộng theo cột rồi chia 4
mean pool = [0.35, 0.5, 0.25]     ← (0.2+0.6+0.1+0.5)/4 = 0.35, v.v.
```

Chỉ đơn giản vậy thôi. Có model dùng cách khác — lấy vector của một token đặc biệt tên `[CLS]` được gắn ở đầu câu, vì token đó được huấn luyện để "tóm tắt" cả câu. Nhưng mean pooling là mặc định phổ biến nhất, và là thứ tham số `pooling: 'mean'` ở Mục 1.8 làm.

**Bước 4 — Normalize.**

> **Normalize (chuẩn hoá, cụ thể là L2 normalization):** chia mọi thành phần của vector cho độ dài của chính nó, để vector kết quả có độ dài đúng bằng 1. Về mặt hình học: giữ nguyên hướng mũi tên, chỉ co/giãn cho nó dài đúng 1 đơn vị.

Ví dụ tính tay với vector `A = (3, 4)`:

```
Độ dài:      |A| = sqrt(3² + 4²) = sqrt(9+16) = sqrt(25) = 5
Chuẩn hoá:   A_norm = (3/5, 4/5) = (0.6, 0.8)
Kiểm tra:    sqrt(0.6² + 0.8²) = sqrt(0.36 + 0.64) = sqrt(1) = 1  ✅
```

Vì sao phải làm bước này? Có hai lý do rất thực dụng.

**Lý do một: nó làm phép so sánh rẻ hơn.** Nhìn lại công thức cosine ở Mục 1.7:

```
cosine(A, B) = (A · B) / (|A| × |B|)
```

Nếu cả A và B đã được chuẩn hoá thì `|A| = 1` và `|B| = 1`, nên mẫu số bằng 1 và công thức rút gọn thành:

```
cosine(A, B) = A · B        ← chỉ còn dot product!
```

Bạn vừa loại bỏ hai phép căn bậc hai và một phép chia khỏi mỗi lần so sánh. Nghe nhỏ, nhưng khi phải làm hàng triệu phép so sánh mỗi giây thì đây là khác biệt lớn về hiệu năng. Đây cũng là lý do các database vector khuyến nghị lưu vector đã chuẩn hoá sẵn.

**Lý do hai: nó khiến mọi phép đo trở nên nhất quán.** Nếu trong database của bạn có lẫn lộn vector đã chuẩn hoá và vector chưa chuẩn hoá, thứ tự xếp hạng kết quả tìm kiếm sẽ sai lệch mà bạn khó phát hiện ra. Chuẩn hoá tất cả là cách phòng bệnh.

### 2.3. 🧩 [Ngoài bài gốc] Bức tranh model năm 2026 — ai đang dùng gì

Video gốc liệt kê Word2Vec, GloVe, Paragram, USE, và nhắc tới OpenAI, Google. Danh sách đó đã cũ. Đây là bức tranh cập nhật tháng 7/2026 (tôi đã tra cứu lại trước khi viết).

| Model | Kiểu | Số chiều | Điểm mạnh |
|---|---|---|---|
| **Google Gemini Embedding** | API trả phí | 3072 | Dẫn đầu bảng xếp hạng tiếng Anh trong phần lớn 2026; hỗ trợ đa phương thức (text, ảnh, video, audio, PDF) trong cùng một không gian vector |
| **Cohere `embed-v4`** | API trả phí | tuỳ chỉnh | 100+ ngôn ngữ; là model đầu tiên đưa multimodal vào production ở quy mô lớn |
| **OpenAI `text-embedding-3-small`** | API trả phí | 1536 | Rẻ, đơn giản, tài liệu tốt — lựa chọn khởi đầu an toàn cho tiếng Anh |
| **OpenAI `text-embedding-3-large`** | API trả phí | 3072 | Mạnh hơn bản small, rất phổ biến trong doanh nghiệp |
| **Voyage `voyage-3.x/4`** | API trả phí | tuỳ chỉnh | Tối ưu riêng cho tìm kiếm; có bản chuyên ngành (luật, tài chính, code) vượt model chung 10–15% trong lĩnh vực của nó |
| **Qwen3-Embedding-8B** (Alibaba) | Mã nguồn mở | 4096 | Rất mạnh về đa ngôn ngữ; đã vượt OpenAI trên benchmark; là lựa chọn tự vận hành mặc định năm 2026 |
| **BGE-M3** (BAAI) | Mã nguồn mở | 1024 | Chuẩn mực mở nguồn, 100+ ngôn ngữ, xuất ra cả dạng dày và thưa cùng lúc |
| **Jina v5-text-small** | Mã nguồn mở | tuỳ chỉnh | Tỉ lệ chất lượng/kích thước tốt nhất — mạnh mà nhẹ |
| **`all-MiniLM-L6-v2`** | Mã nguồn mở | 384 | Chỉ ~23MB, chạy được trong trình duyệt — model mặc định để học và làm demo |
| **TensorFlow USE** | Mã nguồn mở | 512 | *(video gốc dùng)* — đã cũ, không nên chọn cho hệ thống mới |

> **API (Application Programming Interface):** ở đây hiểu đơn giản là "gửi dữ liệu của bạn lên server của một công ty qua internet, họ xử lý rồi trả kết quả về, và tính tiền bạn theo lượng dùng". Bạn không cần máy mạnh, nhưng dữ liệu phải rời khỏi hệ thống của bạn.
>
> **Mã nguồn mở / self-host (tự vận hành):** bạn tải model về, chạy trên máy chủ của chính mình. Không mất tiền theo lượt dùng, dữ liệu không đi đâu cả, nhưng bạn phải tự lo hạ tầng — thường là máy có **GPU**.
>
> **GPU (Graphics Processing Unit):** card đồ hoạ. Vốn sinh ra để chơi game, nhưng do giỏi làm hàng nghìn phép tính song song nên trở thành phần cứng bắt buộc để chạy model AI với tốc độ chấp nhận được.
>
> **Multimodal (đa phương thức):** model xử lý được nhiều loại dữ liệu (chữ, ảnh, âm thanh) và nhúng chúng vào **cùng một không gian vector**. Hệ quả rất mạnh: bạn có thể gõ chữ "con mèo vàng đang ngủ" và tìm ra được *tấm ảnh* con mèo vàng đang ngủ, vì chữ và ảnh cùng nằm trên một bản đồ.

**Quy tắc chọn nhanh (khi bạn cần quyết định trong 30 giây):**

| Tình huống | Chọn |
|---|---|
| Chỉ tiếng Anh, muốn đơn giản, chạy được ngay | OpenAI `text-embedding-3-small` |
| Có tiếng Việt hoặc nhiều ngôn ngữ | BGE-M3, Qwen3-Embedding, hoặc Cohere `embed-v4` |
| Dữ liệu nhạy cảm, không được ra khỏi công ty | Bắt buộc self-host: Qwen3-Embedding hoặc BGE-M3 |
| Chạy trong trình duyệt / máy yếu / học tập | `all-MiniLM-L6-v2` |
| Cần tìm ảnh bằng chữ (hoặc ngược lại) | Gemini Embedding hoặc Cohere `embed-v4` |

⚠️ **Cảnh báo về độ tươi của thông tin:** bảng xếp hạng embedding thay đổi gần như hàng tháng. Bảng trên đúng ở thời điểm tháng 7/2026. Trước khi chốt model cho một dự án thật — và đặc biệt trước khi đi phỏng vấn — hãy mở bảng xếp hạng MTEB trên Hugging Face xem tình hình mới nhất. Ở Phần 4 tôi sẽ nói vì sao **không nên** chọn model chỉ bằng cách nhìn vào vị trí trên bảng xếp hạng.

### 2.4. Code thật — ba con đường

Ba đoạn code dưới đây làm cùng một việc (biến chữ thành vector) nhưng ở ba hoàn cảnh khác nhau. Bạn nên đọc cả ba để biết lúc nào dùng cái nào.

#### (a) JavaScript + Transformers.js — chạy local, miễn phí, không cần API key

Đây là bản cập nhật của cách làm trong video gốc.

```javascript
// npm install @huggingface/transformers      (tên gói MỚI, video gốc dùng @xenova/transformers)
import { pipeline } from '@huggingface/transformers';

// Nạp model MỘT LẦN duy nhất và dùng lại cho mọi câu.
// ⚠️ LỖI HAY GẶP: gọi pipeline() bên trong hàm embed() -> nạp lại model
//    mỗi lần gọi -> chậm gấp hàng trăm lần. Luôn nạp ở ngoài, một lần.
const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');

// Hàm tiện ích: đưa vào một chuỗi, nhận về một mảng 384 số thường
async function embed(text) {
  const out = await extractor(text, {
    pooling: 'mean',     // gộp vector từng token thành một vector câu (Mục 2.2, bước 3)
    normalize: true      // đưa độ dài về 1 (Mục 2.2, bước 4)
  });
  return Array.from(out.data);   // đổi Float32Array -> mảng JS thường, dễ đem đi lưu/gửi
}

// Xử lý NHIỀU câu cùng lúc: nhanh hơn nhiều so với gọi từng câu một.
// Lý do ở Phần 4 — đây là tối ưu quan trọng nhất khi xử lý dữ liệu lớn.
const vectors = await Promise.all([
  embed('áo thun cotton nam'),
  embed('áo phông nam'),
  embed('xe máy Honda')
]);

console.log(vectors[0].length);   // 384
```

**Khi nào dùng:** làm demo, chạy trong trình duyệt, ứng dụng nhỏ, hoặc khi bạn không muốn phụ thuộc dịch vụ bên ngoài.

#### (b) JavaScript + TensorFlow.js Universal Sentence Encoder — đúng như video gốc

```javascript
// npm install @tensorflow/tfjs @tensorflow-models/universal-sentence-encoder
import * as use from '@tensorflow-models/universal-sentence-encoder';

// Tải model USE đã huấn luyện sẵn (~25MB, tải một lần rồi lưu bộ đệm)
const model = await use.load();

// USE nhận vào một MẢNG các câu và trả về một tensor.
// Khác Transformers.js: USE tự lo pooling bên trong, bạn không cần chỉ định.
const embeddings = await model.embed([
  'áo thun cotton nam',
  'áo phông nam'
]);

console.log(embeddings.shape);              // [2, 512] -> 2 câu, mỗi câu 512 chiều

// .array() chuyển tensor thành mảng JavaScript thường để dùng tiếp
const arr = await embeddings.array();
console.log(arr[0].length);                 // 512

// ⚠️ QUAN TRỌNG với TensorFlow.js: phải tự giải phóng bộ nhớ.
// TF.js cấp phát bộ nhớ ngoài tầm quản lý của trình dọn rác JavaScript,
// nên quên dòng này = rò rỉ bộ nhớ, chạy lâu sẽ sập.
embeddings.dispose();
```

> **Universal Sentence Encoder (USE):** model của Google, tạo embedding 512 chiều cho cả câu. Đây là model video gốc dùng. Nó vẫn chạy tốt, nhưng năm 2026 thì đã lạc hậu về chất lượng — chỉ nên dùng nếu bạn đang bảo trì hệ thống cũ.
>
> **Tensor `.dispose()`:** đây là chi tiết mà video gốc không nhắc và người mới rất hay bỏ sót. TensorFlow.js quản lý bộ nhớ thủ công, không tự dọn.

**Khi nào dùng:** khi bạn đã có sẵn hệ sinh thái TensorFlow, hoặc đang bảo trì hệ thống viết theo cách này.

#### (c) Python — con đường chuẩn của ngành

Trong công việc thật, phần lớn pipeline embedding được viết bằng Python. Nếu bạn đi phỏng vấn vị trí backend/ML, khả năng cao interviewer sẽ mong bạn viết bằng Python.

```python
# ── Cách 1: chạy local, miễn phí ─────────────────────────────────
# pip install sentence-transformers
from sentence_transformers import SentenceTransformer

# Nạp model một lần, dùng lại (giống lưu ý ở ví dụ JavaScript)
model = SentenceTransformer("all-MiniLM-L6-v2")

# encode() tự lo hết: tokenize + chạy model + pooling.
# normalize_embeddings=True tương đương normalize:true bên JS.
vec = model.encode("áo thun cotton nam", normalize_embeddings=True)
print(vec.shape)        # (384,) -> một vector 384 chiều

# XỬ LÝ THEO LÔ: đưa vào cả DANH SÁCH câu thay vì gọi từng câu.
# Đây là tối ưu quan trọng nhất — nhanh hơn nhiều lần vì tận dụng
# khả năng tính song song của GPU/CPU.
texts = ["áo thun cotton nam", "áo phông nam", "xe máy Honda"]
vecs = model.encode(
    texts,
    normalize_embeddings=True,
    batch_size=32,           # xử lý 32 câu mỗi lượt
    show_progress_bar=True   # hiện thanh tiến trình, hữu ích khi chạy hàng triệu dòng
)
print(vecs.shape)       # (3, 384) -> 3 câu, mỗi câu 384 chiều


# ── Cách 2: gọi API OpenAI ───────────────────────────────────────
# pip install openai
from openai import OpenAI

client = OpenAI()   # tự đọc khoá từ biến môi trường OPENAI_API_KEY

resp = client.embeddings.create(
    model="text-embedding-3-small",       # ⚠️ GHIM TÊN MODEL RÕ RÀNG — lý do ở Mục 4.5
    input=["áo thun cotton nam", "áo phông nam"]   # gửi cả lô trong MỘT lần gọi mạng
)

# Kết quả trả về là danh sách, thứ tự khớp với thứ tự input
vec1 = resp.data[0].embedding    # list gồm 1536 số thực
vec2 = resp.data[1].embedding
print(len(vec1))                 # 1536
print(resp.usage.total_tokens)   # số token đã dùng -> đây là thứ bị tính tiền
```

> **Biến môi trường (environment variable):** cách lưu thông tin bí mật (như API key) ở bên ngoài code. Bạn đặt nó ở cấu hình hệ thống, chương trình đọc ra lúc chạy. Lý do bắt buộc: **không bao giờ viết thẳng API key vào code rồi đẩy lên GitHub** — có bot chuyên quét GitHub tìm key bị lộ và sẽ dùng hết tiền của bạn trong vài phút.
>
> **Batch (lô):** gom nhiều đầu vào lại xử lý cùng một lần thay vì từng cái một. Đây là kỹ thuật tối ưu quan trọng nhất khi xử lý dữ liệu lớn, sẽ nói kỹ ở Mục 4.1.

Video gốc có nhắc rằng dùng OpenAI hay Google thì cần lấy **API key** từ website của họ rồi kết nối bằng key đó. Điều này đúng, và nó kéo theo cả một chuỗi hệ quả về chi phí, quyền riêng tư và độ tin cậy mà Phần 4 sẽ bàn kỹ.

### 2.5. 🧩 [Ngoài bài gốc] Ba lỗi kinh điển khi làm việc với embedding

Ba lỗi này chiếm phần lớn số giờ debug của người mới. Cả ba đều "chạy được, không báo lỗi, nhưng kết quả sai" — loại bug khó chịu nhất.

> **Debug (gỡ lỗi):** quá trình truy tìm nguyên nhân khiến chương trình chạy sai. Loại bug dễ nhất là loại làm chương trình sập ngay — vì nó chỉ thẳng vào chỗ hỏng. Loại khó nhất là loại vẫn chạy êm nhưng cho kết quả sai, vì bạn không biết mình cần tìm ở đâu. Cả ba lỗi dưới đây đều thuộc loại thứ hai.
>
> **Production (môi trường thật):** hệ thống đang chạy phục vụ người dùng thật, đối lập với môi trường thử nghiệm trên máy lập trình viên. "Lỗi trong production" nghĩa là lỗi mà khách hàng nhìn thấy.

#### Lỗi 1 — Quên pooling, và không hiểu vì sao shape lạ

**Triệu chứng:** bạn embed hai câu, in ra kích thước, thấy `[1, 7, 384]` và `[1, 12, 384]` thay vì `[1, 384]` như mong đợi. Rồi khi tính cosine, hoặc chương trình báo lỗi, hoặc ra một con số vô nghĩa.

**Nguyên nhân:** đúng như Mục 2.2 bước 2 — transformer trả về một vector cho **mỗi token**. Câu 7 token cho 7 vector, câu 12 token cho 12 vector. Hai câu dài khác nhau nên số vector khác nhau, không thể so sánh trực tiếp.

**Cách sửa:**

```javascript
// SAI ❌
const a = await extractor('con mèo đang ngủ');
console.log(a.dims);   // [1, 5, 384] -> 5 vector, không phải 1!

// ĐÚNG ✅
const a = await extractor('con mèo đang ngủ', { pooling: 'mean', normalize: true });
console.log(a.dims);   // [1, 384] -> đúng 1 vector cho cả câu
```

```python
# Python: sentence-transformers tự pooling giúp bạn, nhưng vẫn nên normalize
vec = model.encode(text, normalize_embeddings=True)   # ✅
```

💡 **Mẹo phòng bệnh:** ngay sau khi tạo embedding lần đầu, **luôn in ra kích thước và kiểm tra nó bằng đúng số chiều bạn kỳ vọng**. Ba giây kiểm tra này tiết kiệm cho bạn ba giờ debug.

#### Lỗi 2 — Trộn embedding từ hai model khác nhau

**Triệu chứng:** kết quả tìm kiếm trả về những thứ hoàn toàn ngẫu nhiên, như thể database bị hỏng.

**Nguyên nhân:** trong database của bạn có vector được tạo bởi model A, có vector được tạo bởi model B.

Nghĩ lại analogy bản đồ ở Mục 1.2. Mỗi model vẽ ra **một tấm bản đồ riêng của nó**, theo hệ quy chiếu riêng của nó. Toạ độ (2, 1) trên bản đồ của model A và toạ độ (2, 1) trên bản đồ của model B **chỉ vào hai chỗ hoàn toàn khác nhau**. So sánh chúng cũng vô nghĩa như so sánh "số nhà 15" ở Hà Nội với "số nhà 15" ở TP.HCM — cùng con số, chẳng liên quan gì.

Trường hợp dễ phát hiện là khi số chiều khác nhau (384 vs 1536) — code sẽ báo lỗi ngay. **Trường hợp nguy hiểm hơn nhiều là khi hai model tình cờ có cùng số chiều**: code chạy trơn tru, không báo lỗi gì, chỉ có kết quả là rác. Không có cách nào phát hiện tự động ngoài việc bạn tự kỷ luật.

**Cách phòng:**

```python
# Lưu tên model + phiên bản NGAY BÊN CẠNH vector trong database
{
    "id": 12345,
    "text": "áo thun cotton nam",
    "embedding": [0.21, -0.87, ...],
    "embedding_model": "all-MiniLM-L6-v2",    # ← dòng này cứu bạn sau này
    "embedding_dim": 384
}
```

Quy tắc sắt: **toàn bộ dữ liệu trong một chỉ mục phải dùng chung một model, chung một phiên bản.** Không có ngoại lệ.

Và hệ quả nặng ký: **đổi model = phải tạo lại embedding cho toàn bộ dữ liệu.** Với 10 triệu tài liệu, đây không phải một dòng cấu hình — đây là một dự án kéo dài nhiều ngày, tốn tiền thật. Mục 4.3 sẽ nói cách làm việc này mà không làm sập hệ thống.

#### Lỗi 3 — Dùng dot product mà quên chuẩn hoá

**Triệu chứng:** kết quả tìm kiếm luôn ưu tiên các tài liệu dài, dù nội dung không liên quan lắm.

**Nguyên nhân:** ở Mục 2.2 tôi có nói, sau khi chuẩn hoá thì `cosine = dot product`. Nhiều người nhớ vế sau ("dùng dot cho nhanh") mà quên vế trước ("phải chuẩn hoá trước đã").

Nếu chưa chuẩn hoá, dot product bị ảnh hưởng bởi **độ dài** của vector. Mà văn bản dài thường tạo ra vector có độ dài lớn hơn. Kết quả: những tài liệu dài dòng luôn được xếp trên, dù chúng kém liên quan hơn — đúng thứ mà cosine sinh ra để tránh.

**Cách sửa:** chọn một trong hai, và giữ nhất quán toàn hệ thống:
- Chuẩn hoá **tất cả** vector khi lưu, rồi dùng dot product (nhanh nhất), **hoặc**
- Không chuẩn hoá, nhưng luôn dùng công thức cosine đầy đủ có mẫu số.

Điều tuyệt đối không được làm là **trộn lẫn** hai cách.

### 2.6. Chọn phép đo nào cho việc gì

Cosine không phải phép đo duy nhất. Đây là ba phép đo bạn sẽ gặp:

| Phép đo | Công thức | Đo cái gì | Dùng khi | Toán tử pgvector |
|---|---|---|---|---|
| **Cosine similarity** | `(A·B)/(\|A\|\|B\|)` | Góc giữa hai vector | **Mặc định cho văn bản** | `<=>` |
| **Dot / inner product** | `Σ aᵢbᵢ` | Góc *và* độ dài | Khi đã chuẩn hoá — tương đương cosine nhưng nhanh hơn | `<#>` |
| **Euclidean (L2)** | `sqrt(Σ(aᵢ−bᵢ)²)` | Khoảng cách đường chim bay | Khi độ lớn vector *có* ý nghĩa (hiếm gặp với văn bản) | `<->` |

> **Euclidean distance (khoảng cách Euclid, hay khoảng cách L2):** khoảng cách "đường chim bay" giữa hai điểm — chính là định lý Pythagoras mở rộng. Nếu hai điểm ở `(0,0)` và `(3,4)` thì khoảng cách là `sqrt(3² + 4²) = 5`.

Một điểm dễ gây nhầm cần nhớ: **similarity và distance ngược chiều nhau.**

- **Similarity (độ tương đồng):** càng **CAO** càng giống. Cosine similarity = 1 là giống nhất.
- **Distance (khoảng cách):** càng **THẤP** càng giống. Khoảng cách = 0 là trùng nhau.

pgvector làm việc với **distance**, nên toán tử `<=>` của nó trả về **cosine distance**, tính bằng:

```
cosine_distance = 1 − cosine_similarity        (miền giá trị: từ 0 đến 2)
```

Nên khi bạn viết `ORDER BY embedding <=> query_vector` (sắp xếp tăng dần), bạn đang lấy ra những vector **giống nhất** trước — vì giống nhất nghĩa là khoảng cách nhỏ nhất. Nhầm chiều ở đây là một lỗi phổ biến khiến kết quả tìm kiếm ra ngược hoàn toàn.

### ✅ Self-check Phần 2

**Câu 1.** Từ "bank" trong "river bank" và "bank account": static embedding cho ra mấy vector? Contextual cho ra mấy? Vì sao?

<details>
<summary>Gợi ý đáp án</summary>

Static: **một** vector duy nhất, vì nó chỉ có một hàng trong bảng tra cứu cho mỗi từ — nghĩa bị trộn lẫn, không đúng nghĩa nào. Contextual: **hai** vector khác nhau, vì cơ chế self-attention cho phép từ "bank" nhìn thấy các từ xung quanh ("river" hoặc "account") và điều chỉnh vector của mình theo ngữ cảnh đó.
</details>

**Câu 2.** Vì sao bắt buộc phải pooling khi lấy embedding của một câu từ transformer?

<details>
<summary>Gợi ý đáp án</summary>

Vì transformer trả về một vector cho **mỗi token**, không phải một vector cho cả câu — shape là `[1, số_token, 384]`. Hai câu dài ngắn khác nhau sẽ có số vector khác nhau, không so sánh được. Pooling (thường là lấy trung bình) gộp chúng thành một vector duy nhất đại diện cả câu.
</details>

**Câu 3.** Sau khi chuẩn hoá vector, vì sao cosine similarity rút gọn thành dot product? Điều đó có lợi gì?

<details>
<summary>Gợi ý đáp án</summary>

Vì chuẩn hoá làm `|A| = |B| = 1`, nên mẫu số `|A| × |B|` của công thức cosine bằng 1, chỉ còn lại tử số là dot product. Lợi ích: loại bỏ hai phép căn bậc hai và một phép chia khỏi mỗi lần so sánh — không đáng kể với một phép tính, nhưng rất đáng kể khi làm hàng triệu phép tính mỗi giây.
</details>

**Câu 4.** Bạn đã embed 1 triệu tài liệu bằng model A. Giờ muốn chuyển sang model B tốt hơn. Phải làm gì với dữ liệu cũ?

<details>
<summary>Gợi ý đáp án</summary>

Phải **tạo lại embedding cho toàn bộ 1 triệu tài liệu** bằng model B. Vector cũ và mới nằm trên hai "bản đồ" khác nhau, không so sánh được với nhau, kể cả khi tình cờ cùng số chiều. Đây là một dự án tốn thời gian và tiền bạc, không phải một dòng cấu hình — chi tiết cách làm an toàn ở Mục 4.3.
</details>

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Phần này nhìn xuống **bên dưới bề mặt**: toán học đằng sau cosine, độ phức tạp tính toán, đánh đổi giữa số chiều và chi phí, và các trường hợp biên sẽ làm hỏng hệ thống của bạn lúc 3 giờ sáng.
>
> Trước hết, giải thích hai từ sẽ dùng liên tục từ đây:
>
> **Trade-off (sự đánh đổi):** tình huống trong đó cải thiện mặt này bắt buộc phải hy sinh mặt khác — không có lựa chọn nào tốt hơn ở *mọi* mặt. Ví dụ: tăng số chiều embedding thì chất lượng tìm kiếm tốt hơn, nhưng tốn RAM hơn và tìm chậm hơn. Bạn không thể có cả hai. Kỹ năng cốt lõi của một staff engineer chính là *nhận ra* các trade-off và *nói rõ* mình chọn đánh đổi cái gì lấy cái gì, thay vì giả vờ có giải pháp hoàn hảo.
>
> **Edge case (trường hợp biên):** tình huống hiếm gặp, nằm ở rìa của những gì bình thường xảy ra, thường bị bỏ quên khi viết code — và thường là thứ làm sập hệ thống trong production. Ví dụ: chuyện gì xảy ra nếu có người gửi vào một chuỗi rỗng?

### 3.1. Toán học của cosine — vì sao nó đo được "hướng"

Ở Phần 1 tôi đưa công thức cosine và bảo bạn tin. Giờ tôi chứng minh vì sao nó đúng.

Điểm khởi đầu là **định nghĩa hình học của dot product**:

```
A · B = |A| × |B| × cos(θ)
```

trong đó θ (đọc là "theta", chữ Hy Lạp, quy ước dùng để ký hiệu góc) là góc giữa hai vector.

Công thức này nói: dot product của hai vector bằng tích của hai độ dài, nhân với cosin của góc giữa chúng. Đây là một định lý cơ bản của hình học vector, và nó đúng ở *mọi* số chiều — kể cả 384 chiều, dù ta không hình dung nổi.

Giờ ta chỉ cần biến đổi đại số. Chia cả hai vế cho `|A| × |B|`:

```
   A · B
──────────── = cos(θ)
 |A| × |B|
```

Vế trái chính là công thức cosine similarity mà bạn đã dùng ở Mục 1.7. Vế phải là cosin của góc, thuần tuý.

💡 **Đây là lời giải thích sâu nhất cho câu "cosine bỏ qua độ dài":** phép chia cho `|A| × |B|` **triệt tiêu chính xác** hai thừa số độ dài có trong định nghĩa dot product. Sau khi triệt tiêu, thứ duy nhất còn sót lại là `cos(θ)` — một đại lượng chỉ phụ thuộc vào góc. Không phải người ta "thiết kế" cho nó bỏ qua độ dài; độ dài bị khử đi một cách tự nhiên bởi phép chia.

Từ đó suy ra ngay miền giá trị: vì `cos(θ)` của bất kỳ góc nào cũng nằm trong **[−1, 1]**, cosine similarity cũng nằm trong **[−1, 1]**. Đây là chứng minh một dòng cho điều tôi đính chính ở Mục 1.6.

### 3.2. Vì sao chuẩn hoá làm cosine và Euclidean trở nên tương đương

Đây là một kết quả rất hữu ích và hay được hỏi. Nó giải thích vì sao nhiều hệ thống dùng L2 hay dot product tuỳ tiện mà kết quả top-k vẫn không đổi.

Giả sử A và B đã được chuẩn hoá, tức `|A| = |B| = 1`. Ta tính bình phương khoảng cách Euclid giữa chúng:

```
|A − B|²  =  |A|² + |B|² − 2(A · B)          ← khai triển đại số cơ bản
          =    1  +   1  − 2·cos(θ)           ← thay |A|=|B|=1 và A·B = cos(θ)
          =    2 − 2·cos(θ)
```

Hãy nhìn kỹ kết quả `2 − 2·cos(θ)`. Nó nói rằng khoảng cách Euclid là một hàm **giảm nghiêm ngặt** theo cosine similarity: cosine càng lớn thì khoảng cách càng nhỏ, không có ngoại lệ.

> **Đơn điệu tương đương (monotonically equivalent):** hai phép đo mà khi cái này tăng thì cái kia luôn giảm (hoặc luôn tăng). Hệ quả thực dụng: **thứ tự xếp hạng theo hai phép đo là như nhau.**

Nghĩa là: trên vector đã chuẩn hoá, tìm 10 vector có **cosine similarity cao nhất** và tìm 10 vector có **khoảng cách Euclid nhỏ nhất** cho ra **đúng cùng một danh sách, đúng cùng thứ tự**. Chỉ có con số điểm là khác.

💡 **Ứng dụng thực tế:** đây là lý do lời khuyên "cứ chuẩn hoá hết vector rồi dùng dot product" lại an toàn — bạn được tốc độ của dot product mà không mất gì về chất lượng xếp hạng. Nếu interviewer hỏi *"cosine hay L2?"*, câu trả lời ghi điểm là: *"nếu vector đã chuẩn hoá thì hai cái tương đương về xếp hạng, nên tôi chọn theo tiêu chí hiệu năng chứ không phải chất lượng."*

### 3.3. Độ phức tạp tính toán — vì sao tìm kiếm chính xác không khả thi

> **Big-O:** ký hiệu mô tả *tốc độ tăng* của chi phí tính toán khi dữ liệu lớn dần, bỏ qua các hằng số. `O(n)` nghĩa là gấp đôi dữ liệu thì gấp đôi thời gian. `O(n²)` nghĩa là gấp đôi dữ liệu thì gấp bốn thời gian. `O(1)` nghĩa là thời gian không đổi bất kể dữ liệu lớn cỡ nào.

Chi phí của các thao tác trong bài này:

| Thao tác | Big-O | Nghĩa là |
|---|---|---|
| Tính cosine giữa **hai** vector d chiều | `O(d)` | Phải chạm vào tất cả d con số một lần |
| Tìm vector gần nhất trong **n** vector, cách vét cạn | `O(n × d)` | Tính cosine với từng cái trong n cái |
| Tìm vector gần nhất bằng **ANN index** (ví dụ HNSW) | `≈ O(log n × d)` | Chỉ chạm vào một phần rất nhỏ của n |

Con số cụ thể cho thấy khác biệt lớn đến mức nào. Với n = 10 triệu vector, d = 384 chiều:

```
Vét cạn:      10.000.000 × 384  ≈  3.8 tỉ phép tính mỗi lượt tìm kiếm
ANN (HNSW):   log₂(10.000.000) ≈ 23, và 23 × 384 × (hệ số ~100)
                                ≈  vài trăm nghìn phép tính
```

Khác biệt khoảng **10.000 lần**. Đó không phải tối ưu vặt; đó là ranh giới giữa "làm được" và "không làm được".

> **HNSW (Hierarchical Navigable Small World):** thuật toán ANN phổ biến nhất hiện nay, được pgvector và hầu hết vector database dùng. Ý tưởng: xây một mạng lưới nhiều tầng nối các vector với nhau, tầng trên thưa để nhảy xa, tầng dưới dày để tìm chính xác. Tìm kiếm giống như đi từ sân bay quốc tế → sân bay nội địa → xe buýt → đi bộ: mỗi tầng đưa bạn gần hơn.

**Cái giá của tốc độ đó là gì?** Chính là chữ "approximate" — *xấp xỉ*. ANN có thể bỏ sót một vài kết quả đúng.

> **Recall (độ bao phủ):** tỉ lệ kết quả đúng mà hệ thống tìm ra được. Recall = 0.95 nghĩa là trong 100 kết quả *lẽ ra* phải tìm thấy, hệ thống tìm được 95, bỏ sót 5.
>
> **Recall@k:** recall khi chỉ xét k kết quả đầu tiên. Recall@10 là chỉ số hay dùng nhất cho tìm kiếm, vì người dùng hiếm khi xem quá 10 kết quả đầu.

Và đây là một **trade-off** kinh điển, đúng nghĩa của từ này: bạn chỉnh tham số của HNSW để đổi giữa tốc độ và recall. Muốn recall 0.99 thì chậm hơn; chấp nhận recall 0.90 thì nhanh hơn nhiều. Không có cấu hình nào tốt ở cả hai mặt. Việc chọn điểm cân bằng phụ thuộc vào bài toán: tìm kiếm sản phẩm bỏ sót một kết quả không sao, nhưng tra cứu hồ sơ y tế thì có sao.

### 3.4. Số chiều: nhiều hơn có phải luôn tốt hơn?

Trực giác đầu tiên: nhiều chiều hơn = chứa nhiều thông tin hơn = tìm kiếm chính xác hơn. Điều này **đúng, nhưng chỉ đúng đến một mức**.

Cái giá phải trả tăng theo tuyến tính ở mọi mặt:

| Số chiều | Dung lượng mỗi vector | 10 triệu vector | Chi phí tính một cosine |
|---|---|---|---|
| 384 | 1.5 KB | ~15 GB | 1× |
| 1024 | 4 KB | ~40 GB | 2.7× |
| 1536 | 6 KB | ~60 GB | 4× |
| 3072 | 12 KB | ~120 GB | 8× |

Con số cần chú ý là cột thứ ba. Để tìm kiếm nhanh, ANN index cần nằm trong **RAM**.

> **RAM (Random Access Memory):** bộ nhớ trong của máy chủ — nhanh gấp hàng trăm lần ổ cứng nhưng đắt hơn nhiều và dung lượng nhỏ hơn nhiều. Nếu chỉ số không vừa RAM, hệ thống phải đọc từ đĩa và tốc độ sụp đổ.

Nhìn bảng: 15 GB thì một máy chủ tầm trung xử lý được. 120 GB thì bạn cần máy chủ RAM lớn, đắt gấp nhiều lần — hoặc phải chia dữ liệu ra nhiều máy, kéo theo cả một tầng phức tạp mới. **Số chiều embedding là một quyết định kiến trúc, không phải một tham số kỹ thuật vặt.**

Nhưng có một kỹ thuật thoát khỏi thế lưỡng nan này.

#### Matryoshka — cắt bớt chiều mà gần như không mất chất lượng

> **Matryoshka Representation Learning (MRL):** kỹ thuật huấn luyện model sao cho **thông tin quan trọng nhất được dồn về những chiều đầu tiên** của vector. Tên đặt theo búp bê Matryoshka của Nga — búp bê lớn chứa búp bê nhỏ, mỗi con nhỏ vẫn là một con búp bê hoàn chỉnh.

Hệ quả cực kỳ hữu dụng: bạn có thể **cắt cụt** vector 3072 chiều xuống còn 1024 hoặc 512 chiều — chỉ đơn giản là vứt bỏ phần đuôi — mà chất lượng tìm kiếm chỉ giảm rất ít.

Với model thông thường (không train theo MRL), làm vậy sẽ phá hỏng vector, vì thông tin trải đều khắp các chiều nên vứt bỏ phần nào cũng mất mát nặng.

Con số thực tế được báo cáo: cắt xuống 256 chiều làm giảm độ chính xác chỉ khoảng 2–3%, đổi lại **giảm chi phí lưu trữ khoảng 4 lần**. Với hệ thống hàng trăm triệu vector, đó là khoản tiết kiệm rất lớn cho một mất mát rất nhỏ.

Code minh hoạ:

```python
# Với model hỗ trợ MRL (OpenAI text-embedding-3, Gemini Embedding):
resp = client.embeddings.create(
    model="text-embedding-3-large",
    input="áo thun cotton nam",
    dimensions=512          # ← yêu cầu 512 chiều thay vì 3072 mặc định
)
# API trả về vector 512 chiều, đã được cắt và chuẩn hoá lại đúng cách.

# ⚠️ CẢNH BÁO: chỉ làm được với model được HUẤN LUYỆN theo MRL.
#    Tự tay cắt vector[:512] của một model thường sẽ phá hỏng nó.
```

Các model hỗ trợ MRL tính đến 2026: OpenAI `text-embedding-3-*`, Google Gemini Embedding, và một số model mở nguồn mới.

💡 **Vì sao đây là kiến thức tầm staff:** nó cho bạn một **núm xoay** để điều chỉnh trade-off chi phí–chất lượng, thay vì phải chọn cứng một trong hai. Khi sếp hỏi "làm sao giảm 60% chi phí hạ tầng search?", câu trả lời "giảm số chiều từ 3072 xuống 1024 bằng MRL, mất khoảng 1–2% recall, tôi sẽ đo trên tập dữ liệu thật trước khi triển khai" là câu trả lời của một staff engineer.

### 3.5. Tự viết lại cosine similarity từ số 0

Interviewer rất hay bắt viết hàm này tại chỗ. Nó ngắn, nhưng lộ ra ngay bạn có hiểu bản chất và có nghĩ đến trường hợp biên hay không.

#### Bản Python thuần — không thư viện

```python
def cosine_similarity(a, b):
    """
    Tính cosine similarity giữa hai vector (list số).
    Đây là bản viết tay để thể hiện hiểu bản chất — production dùng NumPy.
    """
    # Kiểm tra đầu vào trước: hai vector khác chiều là lỗi logic,
    # phải báo lỗi rõ ràng chứ không được im lặng cho ra số sai.
    if len(a) != len(b):
        raise ValueError(f"Số chiều không khớp: {len(a)} vs {len(b)}")

    # Tính cả ba đại lượng trong MỘT vòng lặp duy nhất, thay vì ba vòng.
    # Chi tiết nhỏ này là điểm cộng: nó giảm số lần duyệt mảng từ 3 xuống 1.
    dot = 0.0      # tích vô hướng: Σ aᵢ × bᵢ
    norm_a = 0.0   # bình phương độ dài của a: Σ aᵢ²
    norm_b = 0.0   # bình phương độ dài của b: Σ bᵢ²

    for x, y in zip(a, b):     # zip ghép từng cặp phần tử cùng vị trí
        dot += x * y
        norm_a += x * x
        norm_b += y * y

    # EDGE CASE quan trọng nhất: vector 0 (mọi phần tử bằng 0).
    # Nó xảy ra khi input rỗng hoặc bước encode lỗi.
    # Vector 0 KHÔNG có hướng, nên cosine không xác định về mặt toán học.
    # Không guard dòng này = chương trình sập vì chia cho 0.
    denominator = (norm_a ** 0.5) * (norm_b ** 0.5)
    if denominator == 0:
        return 0.0             # quy ước: coi như không liên quan

    return dot / denominator
```

#### Bản NumPy — dùng trong thực tế

```python
import numpy as np

def cosine_similarity_np(a, b) -> float:
    a = np.asarray(a, dtype=np.float32)
    b = np.asarray(b, dtype=np.float32)

    denom = np.linalg.norm(a) * np.linalg.norm(b)   # norm() = độ dài vector
    if denom == 0:
        return 0.0
    return float(np.dot(a, b) / denom)


def cosine_top_k(query, corpus, k=5):
    """
    Tìm k vector giống query nhất trong corpus.

    query:  vector 1 chiều, shape (d,) — vector của câu người dùng tìm
    corpus: ma trận, shape (n, d) — n vector, mỗi vector d chiều.
            "Corpus" (kho ngữ liệu) là từ chuyên ngành chỉ TOÀN BỘ tập
            tài liệu mà hệ thống của bạn có, đã được embed sẵn.
    """
    # Chuẩn hoá query về độ dài 1
    q = query / np.linalg.norm(query)

    # Chuẩn hoá TỪNG HÀNG của corpus.
    # axis=1 nghĩa là tính norm theo chiều ngang (mỗi hàng một norm).
    # keepdims=True giữ shape (n,1) để phép chia broadcast đúng theo hàng.
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)

    # ⭐ MẤU CHỐT HIỆU NĂNG: một phép nhân ma trận thay cho vòng lặp n lần.
    # Vì cả hai đã chuẩn hoá, dot product CHÍNH LÀ cosine (chứng minh ở Mục 2.2).
    # NumPy đẩy phép này xuống thư viện đại số tuyến tính viết bằng C,
    # chạy nhanh hơn vòng lặp Python khoảng 100 lần.
    sims = c @ q                      # kết quả shape (n,)

    # argsort trả về CHỈ SỐ sắp xếp tăng dần.
    # Dấu trừ trước sims biến nó thành sắp xếp giảm dần -> lấy điểm CAO nhất.
    idx = np.argsort(-sims)[:k]
    return idx, sims[idx]
```

#### Bản JavaScript thuần

```javascript
function cosineSimilarity(a, b) {
  if (a.length !== b.length) {
    throw new Error(`Số chiều không khớp: ${a.length} vs ${b.length}`);
  }

  let dot = 0, normA = 0, normB = 0;

  // Vòng lặp chỉ số (không phải for...of) vì nhanh hơn đáng kể
  // với Float32Array — kiểu dữ liệu mà Transformers.js trả về.
  for (let i = 0; i < a.length; i++) {
    dot   += a[i] * b[i];
    normA += a[i] * a[i];
    normB += b[i] * b[i];
  }

  const denom = Math.sqrt(normA) * Math.sqrt(normB);
  return denom === 0 ? 0 : dot / denom;   // guard chia 0
}
```

> ⚠️ **Đọc kỹ đoạn này:** ba hàm trên rất tốt để **học** và để **viết trong phỏng vấn**, nhưng **không** dùng để tìm kiếm trên hàng triệu vector trong production. Lý do đã tính ở Mục 3.3: vét cạn là `O(n×d)`, quá chậm. Production dùng ANN index (pgvector, FAISS, Qdrant...). Nếu interviewer hỏi "vậy dùng hàm này để search trong 10 triệu doc nhé?" thì đó là câu bẫy — trả lời rằng cần ANN index, và nói được con số ước lượng ở Mục 3.3 là bạn ghi điểm.

### 3.6. 🧩 [Ngoài bài gốc] Các trường hợp biên phải thủ sẵn

Đây là danh sách những thứ sẽ làm hỏng hệ thống của bạn trong production. Mỗi cái tôi đều gặp hoặc đã thấy người khác gặp.

**1. Vector 0 hoặc NaN.**

> **NaN:** viết tắt của *Not a Number* — giá trị đặc biệt biểu thị "kết quả tính toán không hợp lệ", ví dụ 0 chia 0. Tính chất nguy hiểm của NaN là nó **lây lan**: bất cứ phép tính nào có NaN tham gia đều cho ra NaN. Một vector NaN lọt vào database có thể làm hỏng toàn bộ kết quả xếp hạng.

Nguyên nhân thường gặp: chuỗi rỗng, chuỗi chỉ có dấu cách, hoặc lỗi mạng lúc gọi API trả về dữ liệu không đầy đủ.

Cách phòng: kiểm tra **trước khi ghi vào database**, không phải lúc đọc ra.

```python
import numpy as np

def is_valid_embedding(vec, expected_dim):
    if vec is None:                       return False
    if len(vec) != expected_dim:          return False   # sai số chiều
    if np.isnan(vec).any():               return False   # chứa NaN
    if np.isinf(vec).any():               return False   # chứa vô cực
    if np.linalg.norm(vec) < 1e-8:        return False   # vector gần như bằng 0
    return True
```

**2. Truncation — văn bản dài bị cắt cụt âm thầm.**

> **Truncation (cắt cụt):** mọi model embedding có giới hạn số token đầu vào (thường 256, 512, hoặc 8192 tuỳ model). Văn bản dài hơn giới hạn sẽ **bị cắt bỏ phần đuôi** trước khi đưa vào model.

Đây là edge case nguy hiểm nhất vì nó **hoàn toàn im lặng**. Không có lỗi, không có cảnh báo. Bạn đưa vào một tài liệu 50 trang, model đọc 2 trang đầu rồi vứt 48 trang còn lại, trả về một vector trông rất bình thường. Bạn lưu vào database và tin rằng mình đã đánh chỉ mục toàn bộ tài liệu. Sáu tháng sau có người hỏi vì sao tìm kiếm không bao giờ ra được nội dung ở phần cuối tài liệu.

Cách xử lý là **chunking**.

> **Chunking (chia đoạn):** cắt tài liệu dài thành nhiều đoạn nhỏ, embed *từng đoạn* riêng, lưu tất cả vào database. Khi tìm kiếm, bạn tìm ra đoạn liên quan nhất chứ không phải cả tài liệu.

```python
def chunk_text(text, max_tokens=400, overlap=50):
    """
    Chia văn bản thành các đoạn có CHỒNG LẤN.

    Vì sao cần chồng lấn (overlap)? Nếu cắt sát rạt, một ý nghĩa
    nằm vắt ngang ranh giới hai đoạn sẽ bị xẻ đôi và mất ngữ cảnh.
    Cho hai đoạn liền kề chia sẻ ~50 token giúp tránh chuyện đó.
    """
    words = text.split()
    step = max_tokens - overlap        # bước nhảy nhỏ hơn kích thước đoạn
    chunks = []
    for i in range(0, len(words), step):
        chunk = " ".join(words[i : i + max_tokens])
        if chunk.strip():              # bỏ qua đoạn rỗng ở cuối
            chunks.append(chunk)
    return chunks
```

Cách chia đoạn tốt hơn nữa là cắt theo **ranh giới ngữ nghĩa** (hết đoạn văn, hết mục) thay vì đếm từ máy móc — vì cắt giữa câu làm hỏng ngữ nghĩa của cả hai nửa. Chiến lược chunking là cả một chủ đề riêng, ảnh hưởng rất lớn đến chất lượng hệ thống RAG.

**3. Sai ngôn ngữ.**

Rất nhiều model phổ biến được huấn luyện chủ yếu trên tiếng Anh. Đưa tiếng Việt vào, chúng vẫn trả về vector trông bình thường — nhưng chất lượng ngữ nghĩa kém hẳn, và một lần nữa **không có lỗi nào được báo**.

Cách kiểm tra: tự làm một bộ 20–30 cặp câu tiếng Việt mà bạn *biết chắc* là đồng nghĩa, đo cosine giữa từng cặp. Nếu model tốt cho tiếng Việt, các cặp này phải cho điểm cao rõ rệt so với các cặp ngẫu nhiên. Nếu điểm lẫn lộn, đổi sang model đa ngôn ngữ (BGE-M3, Qwen3-Embedding, Cohere embed-v4).

**4. Cosine ra giá trị âm.**

Về toán học hoàn toàn hợp lệ (Mục 3.1). Nhưng nếu code của bạn ngầm giả định giá trị nằm trong [0,1] — ví dụ dùng nó làm phần trăm để hiển thị cho người dùng — bạn sẽ hiển thị "độ khớp: −12%". Hoặc xử lý bằng `max(0, sim)`, hoặc chuyển sang dùng cosine distance.

**5. Trộn vector đã chuẩn hoá và chưa chuẩn hoá.**

Đã nói ở Lỗi 3 Mục 2.5, nhắc lại vì nó thuộc nhóm "âm thầm gây sai". Đặc biệt hay xảy ra khi hệ thống có nhiều luồng ghi dữ liệu do các team khác nhau viết. Cách phòng duy nhất hiệu quả: **chuẩn hoá tập trung ở một chỗ duy nhất trong code**, ngay trước khi ghi database, chứ không để mỗi nơi tự lo.

**6. Nhà cung cấp âm thầm đổi phiên bản model.**

Đây là edge case nguy hiểm nhất và bất ngờ nhất, tôi sẽ nói kỹ ở Mục 4.5 vì nó thuộc tầm kiến trúc.

### ✅ Self-check Phần 3

**Câu 1.** Chứng minh ngắn gọn vì sao chia cho `|A| × |B|` khiến cosine "bỏ qua độ dài".

<details>
<summary>Gợi ý đáp án</summary>

Từ định nghĩa hình học `A · B = |A| × |B| × cos(θ)`, chia cả hai vế cho `|A| × |B|` thì hai thừa số độ dài bị triệt tiêu, chỉ còn `cos(θ)` — đại lượng chỉ phụ thuộc góc. Độ dài không bị "bỏ qua" một cách nhân tạo; nó bị khử đi một cách tự nhiên bởi phép chia.
</details>

**Câu 2.** Trên vector đã chuẩn hoá, xếp hạng theo cosine và theo khoảng cách Euclid có khác nhau không? Vì sao?

<details>
<summary>Gợi ý đáp án</summary>

**Không khác.** Vì `|A−B|² = 2 − 2·cos(θ)` khi cả hai đã chuẩn hoá — đây là hàm giảm nghiêm ngặt theo cosine, nên hai phép đo đơn điệu tương đương và cho ra cùng thứ tự xếp hạng. Chỉ giá trị điểm khác nhau, thứ tự thì giống hệt.
</details>

**Câu 3.** Matryoshka embedding cho phép làm gì, và vì sao không áp dụng được với model thường?

<details>
<summary>Gợi ý đáp án</summary>

Cho phép **cắt cụt** vector (ví dụ 3072 → 512 chiều) để tiết kiệm lưu trữ và tăng tốc tìm kiếm, chỉ mất vài phần trăm độ chính xác. Không áp dụng được với model thường vì ở đó thông tin trải đều trên mọi chiều, vứt bỏ phần đuôi là mất mát nặng. Model MRL được *huấn luyện có chủ đích* để dồn thông tin quan trọng về các chiều đầu.
</details>

**Câu 4.** Vì sao truncation là edge case nguy hiểm hơn cả vector NaN?

<details>
<summary>Gợi ý đáp án</summary>

Vì NaN thường gây lỗi rõ ràng hoặc dễ phát hiện bằng kiểm tra, còn truncation **hoàn toàn im lặng** — model vẫn trả về một vector hợp lệ, chỉ là nó chỉ mã hoá phần đầu tài liệu. Hệ thống chạy bình thường trong nhiều tháng trước khi có người phát hiện tìm kiếm không bao giờ ra được nội dung cuối tài liệu. Cách xử lý là chunking, và nên giám sát tỉ lệ input bị cắt.
</details>

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Đây là phần phân biệt một senior engineer với một staff engineer. Senior giải quyết được bài toán kỹ thuật. Staff nhìn ra được **bài toán nào đáng giải**, **cái giá phải trả là gì**, và **giải thích được điều đó cho những người không viết code**.
>
> Vài từ sẽ dùng liên tục từ đây:
>
> **Scale (quy mô):** khi dữ liệu hoặc lượng truy cập tăng lên nhiều bậc. **Scale dọc (vertical)** = mua máy to hơn. **Scale ngang (horizontal)** = thêm nhiều máy. Scale dọc đơn giản hơn nhưng có trần cứng; scale ngang không có trần nhưng phức tạp hơn nhiều.
>
> **Bottleneck (điểm nghẽn):** khâu chậm nhất trong toàn hệ thống, quyết định tốc độ chung. Tối ưu bất cứ khâu nào khác mà không phải bottleneck là lãng phí công sức.
>
> **Latency (độ trễ):** thời gian từ lúc gửi yêu cầu tới lúc nhận kết quả, tính bằng mili giây.
>
> **Throughput (thông lượng):** số lượng công việc xử lý được trong một đơn vị thời gian. Latency và throughput là hai thứ khác nhau và thường đánh đổi nhau.

### 4.1. Bottleneck ở đâu: không phải công thức cosine

Người mới hay nghĩ chỗ chậm nằm ở phép tính cosine. Không phải.

Ở bài **pgvector**, bottleneck là *lưu trữ và tìm kiếm* — RAM để chứa chỉ mục, thời gian duyệt chỉ mục.

Ở bài **này**, bottleneck là *sản xuất ra vector*. Cụ thể là ba thứ: thời gian chạy model, tiền trả cho API, và độ trễ khi phải embed câu truy vấn của người dùng theo thời gian thực.

Đây là bốn kỹ thuật giải quyết, xếp theo mức độ quan trọng.

#### Kỹ thuật 1 — Xử lý theo lô (batching)

Embed từng câu một là cách lãng phí nhất có thể. Lý do: GPU được thiết kế để làm hàng nghìn phép tính song song. Đưa vào một câu, bạn dùng khoảng 1% năng lực của nó; phần lớn thời gian là chi phí cố định để khởi động phép tính.

Gom 32–256 câu vào một lô và đưa vào cùng lúc, tổng thời gian gần như không tăng — nhưng bạn xử lý được gấp hàng chục lần số câu.

```python
# ❌ CHẬM: 1 triệu lần gọi model, mỗi lần lãng phí chi phí khởi động
for doc in documents:
    vec = model.encode(doc)
    save_to_db(vec)

# ✅ NHANH: chia thành các lô, số lần gọi giảm hàng chục lần
BATCH = 128
for i in range(0, len(documents), BATCH):
    batch = documents[i : i + BATCH]
    vecs = model.encode(batch, normalize_embeddings=True)   # 1 lần gọi cho 128 câu
    save_batch_to_db(vecs)                                   # 1 lần ghi cho 128 vector
```

💡 Với API trả phí, batching còn giúp ở một mặt khác: giảm số lần đi qua mạng. Mỗi lần gọi API tốn 50–200ms chỉ riêng cho việc đi và về. Gộp 128 câu vào một lần gọi tiết kiệm 127 lần đi lại đó.

#### Kỹ thuật 2 — Nhớ đệm (caching)

Điểm mấu chốt: **embedding là hàm xác định** — cùng một đoạn văn, cùng một model, cùng một phiên bản, thì luôn cho ra cùng một vector. Nghĩa là tính lại lần thứ hai là lãng phí hoàn toàn.

```python
import hashlib, json

def get_embedding_cached(text, model_name, cache):
    # Tạo "vân tay" duy nhất từ nội dung + tên model.
    # Phải có model_name trong khoá! Nếu không, đổi model sẽ lấy nhầm
    # vector cũ trong cache — một bug cực kỳ khó tìm.
    key = hashlib.sha256(f"{model_name}::{text}".encode()).hexdigest()

    if key in cache:
        return cache[key]              # tiết kiệm 1 lần gọi model + 1 lần trả tiền

    vec = model.encode(text, normalize_embeddings=True)
    cache[key] = vec
    return vec
```

> **Hash (băm):** hàm biến một dữ liệu bất kỳ (dài hay ngắn) thành một chuỗi ký tự có độ dài cố định, đóng vai trò "vân tay". Cùng đầu vào luôn cho cùng vân tay; khác đầu vào thì gần như chắc chắn khác vân tay. Dùng nó làm khoá cache vì nó ngắn gọn và so sánh nhanh.

Tiết kiệm thực tế lớn hơn bạn tưởng, ở hai chỗ. Một là **câu truy vấn lặp lại**: trong một sàn thương mại điện tử, vài trăm từ khoá phổ biến chiếm phần lớn lưu lượng tìm kiếm — cache chúng là gần như miễn phí. Hai là **nội dung không đổi**: khi bạn cập nhật giá của một sản phẩm, mô tả không đổi, không cần embed lại.

#### Kỹ thuật 3 — Tách đường nóng và đường nguội

Hệ thống của bạn có hai luồng embedding với yêu cầu hoàn toàn khác nhau:

| | Đường nguội (indexing) | Đường nóng (query) |
|---|---|---|
| Việc gì | Embed tài liệu để nạp vào kho | Embed câu truy vấn người dùng vừa gõ |
| Ai chờ | Không ai | Người dùng đang nhìn màn hình |
| Ưu tiên | **Throughput** — làm được nhiều | **Latency** — làm thật nhanh |
| Chấp nhận được | Chạy nền vài giờ | Tối đa ~50ms |
| Cách làm | Lô lớn, chạy bất đồng bộ qua hàng đợi | Lô nhỏ, self-host, cache mạnh |

> **Hàng đợi / queue (bất đồng bộ):** cơ chế trong đó công việc được xếp vào một hàng chờ, rồi các tiến trình xử lý lấy ra làm dần. Người gửi không phải đứng đợi kết quả. Đây là cách chuẩn để hệ thống chịu được tải đột biến — khi có 10.000 tài liệu mới đổ vào cùng lúc, chúng xếp hàng chờ thay vì làm sập hệ thống.

💡 **Điểm hay ghi điểm trong phỏng vấn:** rất nhiều ứng viên chỉ nghĩ đến việc embed *tài liệu* mà quên rằng **câu truy vấn cũng phải được embed**, và nó nằm trên đường nóng. Nếu bạn dùng API bên ngoài cho câu truy vấn, mỗi lượt tìm kiếm phải cộng thêm 50–200ms độ trễ mạng — người dùng cảm nhận rõ. Đây là một trong những lý do mạnh nhất để self-host model cho đường query, dù dùng API cho đường indexing.

#### Kỹ thuật 4 — Tính tiền trước khi hứa

Đây là kỹ năng đặc trưng của staff: **quy mọi quyết định kỹ thuật về con số tiền**.

Ví dụ tính toán cho 100 triệu tài liệu, mỗi tài liệu trung bình 200 token, dùng model API rẻ (khoảng 0.02 USD cho mỗi triệu token — hãy tra lại bảng giá hiện hành vì con số này thay đổi):

```
Tổng token = 100.000.000 × 200 = 20.000.000.000 token = 20.000 triệu token
Chi phí     = 20.000 × 0.02 USD ≈ 400 USD    ← cho MỘT lần embed toàn bộ
```

400 USD nghe không nhiều. Nhưng đây mới là những con số cần nói ra:

- Đó là chi phí cho **một lần**. Đổi model sau này = trả lại 400 USD nữa.
- Nếu chọn model đắt hơn 10 lần (điều rất dễ xảy ra), con số thành 4.000 USD mỗi lần.
- **Tiếng Việt tốn khoảng gấp rưỡi đến gấp đôi số token** so với cùng nội dung tiếng Anh, vì tokenizer được tối ưu cho tiếng Anh. Nhân đôi ước tính trên.
- Chưa tính chi phí embed *câu truy vấn* hằng ngày, chi phí lưu trữ, và chi phí RAM cho chỉ mục.

💡 **Câu nói của staff:** *"Trước khi chốt model, tôi cần biết chi phí embed toàn bộ corpus lần đầu, chi phí embed lại khi đổi model, và chi phí truy vấn hằng tháng. Ba con số đó thường quyết định lựa chọn nhiều hơn là điểm benchmark."*

### 4.2. Chọn model — quy trình đúng

**Sai lầm phổ biến nhất: mở bảng xếp hạng MTEB, lấy model đứng đầu, xong.**

> **MTEB (Massive Text Embedding Benchmark):** bộ benchmark tiêu chuẩn để so sánh model embedding, do Hugging Face duy trì. Bảng tiếng Anh gồm 56 tập dữ liệu trải trên 8 loại nhiệm vụ (tìm kiếm, phân loại, gom cụm, xếp hạng lại...). Bảng đa ngôn ngữ **MMTEB** gồm 131 nhiệm vụ trên hơn 250 ngôn ngữ.
>
> **Benchmark:** một bộ bài kiểm tra chuẩn hoá, để so sánh các model một cách công bằng trên cùng tiêu chí.

MTEB rất hữu ích, nhưng có ba giới hạn mà bạn phải biết:

**Một, điểm số là do nhà cung cấp tự nộp.** Mã đánh giá là mở, nhưng không có bên thứ ba độc lập kiểm chứng. Hãy đọc thông cáo báo chí của các hãng với tinh thần đó.

**Hai, benchmark đo trên dữ liệu chung, không phải dữ liệu của bạn.** Một model đứng đầu về tìm kiếm tin tức tiếng Anh chưa chắc tốt cho mô tả sản phẩm tiếng Việt hoặc văn bản pháp lý. Chênh lệch có thể rất lớn.

**Ba, các phiên bản bảng xếp hạng không so sánh trực tiếp được với nhau.** MTEB v1 và v2 dùng bộ dữ liệu khác nhau; bảng tiếng Anh và bảng đa ngôn ngữ dùng cách tính điểm khác nhau. So sánh chéo hai bảng là sai.

**Quy trình đúng gồm bốn bước, theo thứ tự:**

**Bước 1 — Liệt kê ràng buộc trước khi nhìn bất kỳ điểm số nào.** Theo thứ tự ưu tiên:

| Ràng buộc | Câu hỏi cần trả lời |
|---|---|
| Loại dữ liệu | Chỉ chữ? Hay có cả ảnh, PDF? |
| Ngôn ngữ | Có tiếng Việt không? Bao nhiêu ngôn ngữ? |
| Quyền riêng tư | Dữ liệu có được phép rời khỏi hệ thống không? |
| Độ trễ | Đường nóng cần bao nhiêu ms? |
| Ngân sách | Bao nhiêu tiền một tháng, cả embed lẫn hạ tầng? |

Nếu ràng buộc quyền riêng tư nói "không được gửi ra ngoài", thì toàn bộ nhóm model API bị loại ngay từ đầu, **bất kể chúng đứng đầu bảng xếp hạng**. Nhận ra điều này sớm tiết kiệm cho bạn hàng tuần công sức.

**Bước 2 — Lập danh sách ngắn 2–3 model** từ MTEB/MMTEB thoả mãn các ràng buộc trên.

**Bước 3 — Tự đo trên dữ liệu thật của bạn.** Đây là bước quyết định, và cũng là bước hay bị bỏ qua nhất.

Cách làm: xây một **tập vàng** gồm 50–200 câu truy vấn thật, kèm nhãn cho biết tài liệu nào *đúng ra* phải xuất hiện trong kết quả. Chạy từng model ứng viên trên tập này rồi đo:

> **nDCG (normalized Discounted Cumulative Gain):** chỉ số đo chất lượng xếp hạng, có tính đến **vị trí** — kết quả đúng nằm ở hạng 1 có giá trị cao hơn cùng kết quả đó nằm ở hạng 9. Đây là chỉ số sát nhất với trải nghiệm người dùng thật.
>
> **MRR (Mean Reciprocal Rank):** trung bình của nghịch đảo thứ hạng của kết quả đúng đầu tiên. Nếu kết quả đúng luôn nằm ở hạng 1 thì MRR = 1. Hợp với bài toán "chỉ cần một câu trả lời đúng", ví dụ hỏi đáp.
>
> (Recall@k đã giải thích ở Mục 3.3.)

**Bước 4 — Cân nhắc fine-tune nếu lĩnh vực rất hẹp.**

> **Fine-tune (tinh chỉnh):** lấy một model đã huấn luyện sẵn rồi huấn luyện thêm một chút trên dữ liệu chuyên ngành của bạn. Rẻ hơn huấn luyện từ đầu rất nhiều.

Với các lĩnh vực có ngôn ngữ đặc thù — pháp lý, y tế, mã nguồn — fine-tune thường cải thiện 10–30%. Nhưng nó thêm một tầng vận hành: bạn phải giữ dữ liệu huấn luyện, lặp lại quá trình khi đổi model nền, và tự chịu trách nhiệm về chất lượng. Đừng bắt đầu bằng fine-tune; chỉ làm khi model có sẵn thực sự không đủ.

💡 **Câu chốt đắt giá:** *"MTEB narrows the shortlist; your own labeled queries make the decision. Benchmark rank is a prior, not a verdict."* — (MTEB thu hẹp danh sách; dữ liệu có nhãn của chính bạn mới ra quyết định. Thứ hạng benchmark là một giả định ban đầu, không phải một phán quyết.)

### 4.3. Đổi model — quyết định kiến trúc nặng ký nhất

Đây là chỗ nhiều đội ngũ bị đau, và là chủ đề system design rất hay được hỏi.

Nhắc lại điều đã biết từ Mục 2.5: **đổi model embedding = phải tạo lại vector cho toàn bộ dữ liệu**, vì vector cũ và mới nằm trên hai bản đồ khác nhau.

Với 100 triệu tài liệu, việc này có thể mất nhiều ngày chạy máy và tốn hàng nghìn đô. Và trong suốt thời gian đó, hệ thống tìm kiếm vẫn phải phục vụ người dùng bình thường.

**Chiến lược an toàn gồm bốn phần:**

**Một — Đánh phiên bản cho cột dữ liệu.** Đừng ghi đè lên cột cũ. Thêm cột mới song song:

```sql
ALTER TABLE products ADD COLUMN embedding_v2 vector(1024);
-- Cột embedding_v1 vẫn nguyên vẹn và vẫn đang phục vụ người dùng.
-- Nếu model mới hoá ra tệ hơn, bạn chỉ việc bỏ cột v2 đi. Không mất gì.
```

**Hai — Backfill chạy nền.**

> **Backfill (nạp bù):** chạy một tiến trình nền để lấp đầy dữ liệu cho phần đã tồn tại từ trước. Nó phải chạy chậm rãi, có giới hạn tốc độ, để không cạnh tranh tài nguyên với hệ thống đang phục vụ người dùng thật.

**Ba — Ghi kép (dual-write) trong giai đoạn chuyển tiếp.** Trong thời gian backfill, mọi tài liệu mới phải được ghi vào **cả hai cột**. Nếu không, dữ liệu mới sẽ chỉ có ở một cột và bạn sẽ có một vùng dữ liệu bị thiếu, rất khó phát hiện.

**Bốn — Chuyển đổi kiểu blue-green.**

> **Blue-green deployment:** kỹ thuật triển khai trong đó bạn duy trì hai môi trường song song — "xanh dương" đang chạy thật và "xanh lá" là bản mới. Bạn kiểm tra kỹ bản mới, rồi chuyển lưu lượng sang bằng một thao tác. Nếu có vấn đề, chuyển ngược lại tức thì.

Với embedding, cách làm cụ thể: cho một phần nhỏ lưu lượng thật (5–10%) chạy trên cột v2, so sánh chất lượng bằng số liệu thật (tỉ lệ người dùng bấm vào kết quả, tỉ lệ tìm kiếm không ra gì), rồi mới tăng dần lên 100%.

💡 **Bài học lớn nhất, và cũng là câu trả lời phỏng vấn hay:** *"Vì việc đổi model tốn kém đến vậy, tôi dành nhiều thời gian hơn cho khâu chọn model **ở giai đoạn thiết kế**. Đầu tư một tuần đánh giá kỹ lúc đầu rẻ hơn nhiều so với một dự án di trú kéo dài một tháng sau này."*

### 4.4. Chi phí, độ tin cậy, giám sát

#### Những thứ đẩy chi phí lên

| Yếu tố | Cách kiểm soát |
|---|---|
| Số token đã embed | Chunking hợp lý; đừng embed rác (menu, footer, boilerplate) |
| Số lần embed lại | Chốt model sớm; đánh phiên bản cẩn thận |
| Số chiều vector | Dùng Matryoshka để cắt nếu model hỗ trợ (Mục 3.4) |
| Câu truy vấn lặp | Cache theo hash (Mục 4.1) |
| Tiếng Việt tốn token | Chọn model có tokenizer tốt cho tiếng Việt |

#### Độ tin cậy

> **Reliability (độ tin cậy):** khả năng hệ thống vẫn hoạt động đúng khi có sự cố.
>
> **Rate limit (giới hạn tốc độ):** giới hạn số lần gọi API trong một khoảng thời gian mà nhà cung cấp áp cho bạn. Vượt quá thì bị từ chối.
>
> **Retry with backoff (thử lại có giãn cách):** khi gặp lỗi tạm thời, thử lại — nhưng chờ lâu dần sau mỗi lần thất bại (1 giây, 2 giây, 4 giây, 8 giây...). Nếu thử lại ngay lập tức và liên tục, bạn góp phần làm sập hệ thống đang quá tải.

```python
import time, random

def embed_with_retry(text, max_retries=5):
    for attempt in range(max_retries):
        try:
            return client.embeddings.create(model=MODEL, input=text)
        except RateLimitError:
            # Chờ tăng theo cấp số nhân: 1s, 2s, 4s, 8s, 16s
            # Cộng thêm một chút ngẫu nhiên ("jitter") để tránh việc
            # hàng nghìn tiến trình cùng thử lại đúng một thời điểm.
            wait = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait)
    raise Exception("Đã hết số lần thử lại")
```

Ngoài ra, nên có **phương án dự phòng** cho đường nóng: nếu API embedding chết, hệ thống tìm kiếm của bạn không nên chết theo. Hai lựa chọn: giữ sẵn một model nhỏ self-host để dùng tạm, hoặc tạm thời rơi về tìm kiếm bằng từ khoá (full-text search) — kết quả kém hơn nhưng vẫn còn dùng được, hơn là trang lỗi.

#### Giám sát — bốn chỉ số phải theo dõi

> **Monitoring (giám sát):** thu thập số liệu về hệ thống đang chạy để phát hiện vấn đề *trước khi* người dùng phát hiện.

| Chỉ số | Vì sao quan trọng |
|---|---|
| **Tỉ lệ input bị cắt cụt** | Phát hiện truncation âm thầm (Mục 3.6) — chỉ số bị bỏ quên nhiều nhất |
| **Tỉ lệ lỗi encode / vector rỗng** | Phát hiện dữ liệu hỏng lọt vào kho |
| **Recall@k trên tập vàng, đo định kỳ** | Phát hiện chất lượng tìm kiếm suy giảm theo thời gian |
| **Chi phí token mỗi ngày** | Phát hiện rò rỉ chi phí trước khi thành hoá đơn gây sốc |

> **Drift (trôi dạt):** hiện tượng chất lượng model giảm dần theo thời gian, dù model không đổi — vì *dữ liệu và cách người dùng gõ* thay đổi. Ví dụ: một sàn thương mại thêm ngành hàng mới, xuất hiện thuật ngữ mà model chưa từng gặp lúc huấn luyện. Cách phát hiện duy nhất là đo recall trên tập vàng **định kỳ**, không phải chỉ đo một lần lúc ra mắt.

#### Các kiểu hỏng — xếp theo mức độ nguy hiểm

**1. Nhà cung cấp âm thầm đổi phiên bản model.** ⚠️ Nguy hiểm nhất.

Bạn gọi API với tên `"text-embedding-3-small"`. Nhà cung cấp cập nhật model đứng sau cái tên đó. Từ thời điểm đó, vector mới sinh ra **không còn cùng không gian** với vector cũ trong database của bạn. Không có thông báo. Không có lỗi. Chất lượng tìm kiếm chỉ đơn giản là kém dần đi, và bạn không hiểu vì sao.

Cách phòng: **ghim phiên bản** rõ ràng nếu nhà cung cấp hỗ trợ, ghi lại tên và phiên bản model bên cạnh mỗi vector, và giám sát recall trên tập vàng để bắt được sự cố này.

**2. Tài liệu dài bị cắt cụt.** Đã nói ở Mục 3.6. Cách phòng: chunking + giám sát tỉ lệ cắt.

**3. Dùng model sai ngôn ngữ.** Cách phòng: tự kiểm tra bằng bộ câu đồng nghĩa tiếng Việt.

**4. Trộn vector chuẩn hoá và chưa chuẩn hoá.** Cách phòng: chuẩn hoá tập trung một chỗ.

### 4.5. Nói chuyện với người không rành kỹ thuật

Đây là kỹ năng khiến một staff engineer khác với một senior giỏi. Nếu bạn không giải thích được cho quản lý sản phẩm hoặc giám đốc vì sao nên chọn phương án A, bạn sẽ không được quyết định.

**Nguyên tắc: bỏ hết thuật ngữ, quy về ba thứ họ quan tâm — tiền, rủi ro, và tốc độ ra sản phẩm.**

Ví dụ một đoạn nói thật, có thể dùng nguyên văn:

> *"Embedding là phần giúp ô tìm kiếm hiểu được **ý** khách hàng muốn gì, chứ không chỉ so khớp chữ. Nhờ nó, khách gõ 'áo phông' vẫn tìm ra 'áo thun'.*
>
> *Chọn công cụ nào cho phần này là đánh đổi giữa ba thứ. Một, **chi phí**: thuê dịch vụ ngoài thì trả tiền theo lượng dùng, tự vận hành thì tốn đầu tư máy chủ ban đầu nhưng rẻ về sau. Hai, **quyền riêng tư**: thuê ngoài nghĩa là dữ liệu khách hàng của mình được gửi sang máy chủ công ty khác — với dữ liệu nhạy cảm, đây có thể là điều không được phép. Ba, **tốc độ ra mắt**: thuê ngoài thì chạy được trong một tuần, tự vận hành thì mất khoảng một tháng.*
>
> *Một điểm quan trọng nữa cần biết trước: **đổi lựa chọn về sau rất tốn kém**. Nếu sau này ta muốn đổi sang công cụ khác, phải xử lý lại toàn bộ dữ liệu — với quy mô hiện tại là khoảng hai tuần công và vài nghìn đô. Nên tôi đề nghị dành thêm một tuần đánh giá kỹ ngay bây giờ, thay vì phải làm lại sau."*

Hãy để ý đoạn trên **không có một thuật ngữ nào** không được giải thích: không có "vector", không có "cosine", không có "MTEB". Nhưng nó truyền tải đủ ba trade-off và một cảnh báo rủi ro quan trọng.

**Về mặt tổ chức**, có hai điều một staff engineer nên chủ động nêu:

**Một, quyền riêng tư đôi khi lấn át benchmark.** Nếu công ty làm việc với dữ liệu y tế, tài chính, hoặc dữ liệu cá nhân thuộc phạm vi GDPR, việc gửi dữ liệu đó tới API bên thứ ba có thể vi phạm quy định. Khi đó bạn **buộc phải** self-host, dù model tự vận hành hơi yếu hơn model API. Đây là ràng buộc cần nêu ngay từ cuộc họp đầu tiên, không phải sau khi đã xây xong.

> **GDPR:** bộ luật bảo vệ dữ liệu cá nhân của châu Âu, quy định chặt chẽ việc xử lý và chuyển giao dữ liệu cá nhân. Nhiều nước có luật tương tự; Việt Nam có Nghị định về bảo vệ dữ liệu cá nhân.

**Hai, pipeline embedding nên là hạ tầng dùng chung.** Nếu ba đội trong công ty đều cần embedding, đừng để mỗi đội tự viết một bản riêng. Rất nhanh bạn sẽ có ba model khác nhau, ba cách chunking khác nhau, ba hoá đơn — và không ai so sánh được kết quả với ai. Xây nó thành một dịch vụ nội bộ có phiên bản rõ ràng, cache chung, và giám sát chung.

### 4.6. Câu hỏi system design mẫu + hướng trả lời

> **Đề bài:** *"Thiết kế hệ thống sinh và phục vụ embedding cho tìm kiếm ngữ nghĩa trên 50 triệu tài liệu. Tài liệu mới được thêm vào theo thời gian thực. Nội dung gồm cả tiếng Anh và tiếng Việt. Dữ liệu là hồ sơ khách hàng, có tính nhạy cảm."*

Đây là cách một staff engineer trả lời. Chú ý khung: **làm rõ đề → chọn → thiết kế → quy mô → rủi ro**, và đặc biệt là **nói rõ mình đang đánh đổi cái gì lấy cái gì** ở mỗi bước.

**Bước 1 — Làm rõ đề trước khi vẽ gì cả.** (Đây là bước ứng viên hay bỏ qua nhất, và bỏ qua nó là mất điểm nặng.)

- Lượng truy vấn mỗi giây ở giờ cao điểm là bao nhiêu?
- Tài liệu mới thêm vào với tốc độ nào — trăm cái một ngày hay triệu cái một ngày?
- Độ dài trung bình một tài liệu?
- Mục tiêu chất lượng là gì — đo bằng recall@10 hay bằng tỉ lệ người dùng bấm vào kết quả?
- **Ràng buộc pháp lý về dữ liệu khách hàng cụ thể là gì?** Có được gửi ra ngoài lãnh thổ không?

**Bước 2 — Chọn model, và nói rõ vì sao.**

Ràng buộc *dữ liệu nhạy cảm* cộng với *cần tiếng Việt* dẫn thẳng tới **self-host một model đa ngôn ngữ** — ví dụ Qwen3-Embedding hoặc BGE-M3 — thay vì dùng API.

Điểm mấu chốt cần nói ra: **quyền riêng tư là yếu tố quyết định ở đây, không phải điểm benchmark.** Tôi chấp nhận một model có thể yếu hơn vài điểm so với model API tốt nhất, để đổi lấy việc dữ liệu khách hàng không rời khỏi hạ tầng của công ty. Tôi vẫn sẽ đo hai đến ba ứng viên trên tập truy vấn thật trước khi chốt.

**Bước 3 — Đường nạp dữ liệu (đường nguội).**

```
Tài liệu mới → hàng đợi → worker chunking → worker embedding (GPU, xử lý lô)
             → chuẩn hoá → ghi vào PostgreSQL + pgvector (vector + siêu dữ liệu)
```

> **Worker (tiến trình thợ):** một chương trình chạy nền, liên tục lấy việc từ hàng đợi ra xử lý. Muốn xử lý nhanh gấp đôi, bạn chỉ cần chạy gấp đôi số worker — đây chính là scale ngang trong thực tế.

Vì sao đi qua hàng đợi: để chịu được tải đột biến. Khi khách hàng nhập một lúc 100.000 tài liệu, chúng xếp hàng chờ thay vì làm sập cụm GPU.

Vì sao chunking đứng trước embedding: để tránh truncation âm thầm — vấn đề đã phân tích ở Mục 3.6.

**Bước 4 — Đường truy vấn (đường nóng).**

```
Người dùng gõ → kiểm tra cache (hash của truy vấn)
              → nếu trượt: embed bằng model self-host (~10ms)
              → tìm bằng chỉ mục HNSW trong pgvector
              → (tuỳ chọn) kết hợp thêm full-text search
              → trả kết quả
```

Vì sao self-host cho đường này: độ trễ. Gọi API mất 50–200ms chỉ riêng phần mạng, người dùng cảm nhận được. Self-host trên GPU nội bộ khoảng 5–15ms.

Vì sao cache: một phần rất lớn lưu lượng tìm kiếm tập trung vào vài trăm từ khoá phổ biến.

Vì sao kết hợp full-text: embedding tìm theo nghĩa nhưng yếu ở mã sản phẩm, tên riêng, số hiệu. Full-text search bắt chính xác những thứ đó. Kết hợp cả hai — gọi là **hybrid search** — cho kết quả tốt hơn hẳn từng cái riêng lẻ.

**Bước 5 — Quy mô và chi phí.**

- 50 triệu tài liệu × ~1.5 chunk mỗi tài liệu ≈ 75 triệu vector.
- Với 1024 chiều: 75 triệu × 4 KB ≈ **300 GB**. Con số này **không vừa RAM** của một máy chủ thông thường.
- Hai hướng xử lý, cần nêu cả hai: (a) dùng Matryoshka cắt xuống 512 hoặc 256 chiều, giảm còn 75–150 GB, mất 2–3% độ chính xác; (b) chia dữ liệu ra nhiều máy (scale ngang), giữ nguyên chất lượng nhưng thêm một tầng phức tạp về vận hành.
- Tôi sẽ **đo trước rồi mới chọn**: nếu mất 2–3% recall là chấp nhận được với bài toán này, phương án (a) đơn giản hơn nhiều và tôi chọn nó.

**Bước 6 — Phiên bản và kế hoạch di trú.**

Ngay từ ngày đầu: cột vector có đánh phiên bản, lưu tên model bên cạnh mỗi bản ghi, và có sẵn quy trình backfill. Không phải vì tôi định đổi model ngay, mà vì chắc chắn sẽ có ngày phải đổi, và chuẩn bị trước rẻ hơn rất nhiều so với chữa cháy sau.

**Bước 7 — Giám sát.**

Tỉ lệ tài liệu bị cắt cụt, tỉ lệ lỗi encode, recall@10 trên tập vàng đo hằng tuần, độ trễ đường nóng ở phân vị 95, và chi phí GPU mỗi ngày.

**Bước 8 — Khi nào tôi sẽ chọn ngược lại.**

Nếu ràng buộc quyền riêng tư *không* tồn tại và đội ngũ nhỏ không muốn vận hành GPU, thì dùng API đơn giản hơn hẳn và tôi sẽ chọn API. **Yếu tố quyết định trong bài này là quyền riêng tư — nếu nó thay đổi, quyết định của tôi cũng thay đổi.**

💡 **Vì sao bước 8 là dấu hiệu của staff:** nêu rõ *điều kiện nào sẽ khiến mình đổi ý* cho thấy bạn hiểu quyết định của mình dựa trên cái gì, thay vì học thuộc một kiến trúc "đúng". Người phỏng vấn tìm chính xác điều này.

### ✅ Self-check Phần 4

**Câu 1.** Bottleneck của một hệ thống embedding nằm ở đâu? Nêu hai kỹ thuật quan trọng nhất để giải quyết.

<details>
<summary>Gợi ý đáp án</summary>

Không nằm ở phép tính cosine mà ở **khâu sản xuất vector**: thời gian chạy model, tiền API, và độ trễ khi embed câu truy vấn theo thời gian thực. Hai kỹ thuật quan trọng nhất: **batching** (gom nhiều câu vào một lần gọi, tận dụng khả năng song song của GPU) và **caching** (embedding là hàm xác định nên cùng input luôn cho cùng output — tính lại là lãng phí).
</details>

**Câu 2.** Vì sao không nên chọn model chỉ bằng cách lấy model đứng đầu MTEB?

<details>
<summary>Gợi ý đáp án</summary>

Ba lý do. Một, điểm số do nhà cung cấp tự nộp, không có kiểm chứng độc lập. Hai, benchmark đo trên dữ liệu chung chứ không phải dữ liệu của bạn — model giỏi tìm kiếm tin tức tiếng Anh chưa chắc giỏi mô tả sản phẩm tiếng Việt. Ba, ràng buộc cứng (quyền riêng tư, độ trễ, ngân sách) có thể loại thẳng model đứng đầu bất kể điểm số. Quy trình đúng: liệt kê ràng buộc → lập danh sách ngắn từ MTEB → đo trên tập truy vấn thật có nhãn của mình.
</details>

**Câu 3.** Kể tên kiểu hỏng nguy hiểm nhất trong hệ thống embedding, và cách phòng.

<details>
<summary>Gợi ý đáp án</summary>

**Nhà cung cấp âm thầm đổi phiên bản model đứng sau một cái tên API.** Nguy hiểm vì hoàn toàn im lặng: vector mới không còn cùng không gian với vector cũ, chất lượng tìm kiếm giảm dần mà không có lỗi nào được báo. Phòng bằng cách ghim phiên bản model rõ ràng, lưu tên/phiên bản bên cạnh mỗi vector, và giám sát recall@k trên tập vàng định kỳ để phát hiện suy giảm.
</details>

**Câu 4.** Giải thích trong 3 câu cho một giám đốc không rành kỹ thuật vì sao "đổi model embedding" lại đắt.

<details>
<summary>Gợi ý đáp án</summary>

*"Mỗi công cụ hiểu-nghĩa mã hoá dữ liệu theo một cách riêng, giống hai người ghi chú bằng hai hệ tốc ký khác nhau — ghi chú của người này người kia đọc không được. Nên đổi công cụ nghĩa là phải xử lý lại toàn bộ dữ liệu từ đầu, với quy mô hiện tại là khoảng X ngày công và Y đô. Vì vậy tôi đề nghị đầu tư thêm một tuần đánh giá ngay bây giờ, thay vì phải làm lại toàn bộ sau sáu tháng."*
</details>

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn. Nếu bạn đã đọc bốn phần trên, mọi thứ ở đây chỉ là gợi nhớ.

### 5.1. Bảng thuật ngữ bắt buộc nhớ

Giữ nguyên tiếng Anh vì phỏng vấn kỹ thuật dùng tiếng Anh.

| Thuật ngữ | Định nghĩa một dòng |
|---|---|
| **Embedding** | Dãy số nhiều chiều mã hoá ý nghĩa của một đối tượng; giống nghĩa → vector gần nhau |
| **Dimension** | Số phần tử trong vector (MiniLM 384, USE 512, OpenAI 1536/3072) |
| **Static embedding** | Một vector cố định cho mỗi từ (Word2Vec, GloVe) — "một từ một nghĩa" |
| **Contextual embedding** | Vector phụ thuộc ngữ cảnh (BERT/GPT/transformer) — "bank" hai nghĩa hai vector |
| **Transformer / self-attention** | Kiến trúc cho phép mỗi từ "nhìn" các từ xung quanh để điều chỉnh vector của mình |
| **Token / tokenizer** | Đơn vị nhỏ nhất model xử lý; bộ phận cắt chữ thành token (1 token ≈ 0.75 từ tiếng Anh) |
| **Pooling (mean pooling)** | Gộp nhiều vector token thành một vector câu, thường bằng trung bình cộng |
| **Normalize (L2)** | Đưa vector về độ dài 1; sau đó dot product ≡ cosine similarity |
| **Dot product** | `Σ aᵢbᵢ` — nhân từng cặp rồi cộng lại |
| **Magnitude / norm** | `sqrt(Σ aᵢ²)` — độ dài của vector |
| **Cosine similarity** | `(A·B)/(\|A\|\|B\|)`, đo góc, miền **[−1, 1]** |
| **Cosine distance** | `1 − cosine_similarity`, miền [0, 2]; nhỏ = gần (đây là thứ pgvector `<=>` trả về) |
| **Euclidean (L2) distance** | `sqrt(Σ(aᵢ−bᵢ)²)` — khoảng cách đường chim bay |
| **Feature-extraction pipeline** | Tên công việc "tạo embedding" trong Transformers.js |
| **Transformers.js** | Chạy model Hugging Face bằng JavaScript; gói `@huggingface/transformers` (tên cũ `@xenova/transformers`), hiện ở v4 |
| **Universal Sentence Encoder** | Model của Google, 512 chiều — model dùng trong video gốc, nay đã cũ |
| **MTEB / MMTEB** | Benchmark chuẩn so sánh model embedding (56 tập tiếng Anh / 131 nhiệm vụ đa ngôn ngữ) |
| **Recall@k / nDCG / MRR** | Ba chỉ số đo chất lượng tìm kiếm; nDCG có tính đến vị trí kết quả |
| **Matryoshka (MRL)** | Kỹ thuật cho phép cắt cụt vector với mất mát rất nhỏ |
| **Chunking / truncation** | Chia tài liệu dài thành đoạn / hiện tượng model cắt bỏ phần vượt giới hạn token |
| **ANN / HNSW** | Tìm láng giềng gần nhất xấp xỉ; thuật toán ANN phổ biến nhất |
| **Re-embedding** | Tạo lại toàn bộ vector khi đổi model |
| **Hybrid search** | Kết hợp tìm kiếm vector (theo nghĩa) với full-text (theo từ khoá) |

### 5.2. Mười ý cốt lõi — nếu chỉ được nhớ mấy điều

1. Embedding biến ý nghĩa thành số; giống nghĩa → vector gần nhau → máy tính so sánh được bằng toán.
2. **Model tạo ra embedding, database chỉ lưu và tìm.** pgvector không tự sinh vector. *(Model là đầu bếp, database là tủ lạnh.)*
3. **Static vs contextual** là phân biệt cốt lõi. Transformer thắng vì self-attention cho phép vector phụ thuộc ngữ cảnh.
4. Cosine đo **hướng**, bỏ qua độ dài — đúng thứ ta cần cho văn bản. Miền giá trị **[−1, 1]**, không phải [0, 1].
5. Transformer xuất một vector cho **mỗi token** → phải **pooling + normalize** mới có vector câu.
6. Sau khi normalize, `cosine = dot product`, và cosine với L2 cho **cùng thứ tự xếp hạng**.
7. Vector từ hai model (hoặc hai phiên bản) khác nhau **không so sánh được**. Đổi model = **embed lại toàn bộ**.
8. Chọn model: ràng buộc trước → MTEB để lập danh sách ngắn → **đo trên dữ liệu thật** để quyết.
9. Ở quy mô lớn: **batch, cache, chunk**. Chi phí token và độ trễ là ràng buộc thật, phải tính bằng tiền.
10. Chuỗi hoàn chỉnh: text → **embed (bài này)** → **lưu & tìm bằng ANN (pgvector)** → **hybrid với full-text** → RAG.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Bản đồ ý nghĩa"** — embedding là toạ độ trên một tấm bản đồ nhiều chiều; giống nghĩa thì gần nhau.
- **"Model là đầu bếp, database là tủ lạnh"** — trả lời tức thì cho nhầm lẫn "pgvector tự tạo embedding".
- **"Cosine so hướng, không so độ dài"** — một câu là xong, và giải thích được vì sao hợp với văn bản.
- **"Bank hai nghĩa"** — minh hoạ static vs contextual chỉ trong 10 giây.
- **"Hai bản đồ khác nhau, cùng toạ độ, hai địa điểm khác nhau"** — vì sao không trộn model được.
- **"MTEB là giả định ban đầu, dữ liệu của bạn là phán quyết"** — thể hiện sự chín chắn khi chọn công cụ.
- **"Đường nóng và đường nguội"** — nhớ rằng câu truy vấn cũng phải được embed, và nó cần nhanh.

### 5.4. Code cần thuộc lòng

**(a) Tạo embedding — JavaScript:**

```javascript
import { pipeline } from '@huggingface/transformers';
const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
const out = await extractor(text, { pooling: 'mean', normalize: true });
const vec = Array.from(out.data);   // 384 số
```

**(b) Tạo embedding — Python:**

```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-MiniLM-L6-v2")
vec = model.encode(text, normalize_embeddings=True)          # một câu
vecs = model.encode(texts, normalize_embeddings=True, batch_size=32)   # cả lô
```

**(c) Cosine similarity từ số 0 — interviewer hay bắt viết tại chỗ:**

```python
def cosine(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    na  = sum(x * x for x in a) ** 0.5
    nb  = sum(y * y for y in b) ** 0.5
    return dot / (na * nb) if na and nb else 0.0
    #                          ↑ guard vector 0 — nhớ nói ra edge case này,
    #                            đây là chi tiết phân biệt ứng viên cẩn thận
```

**(d) Tìm top-k bằng NumPy — khi được hỏi "làm sao cho nhanh":**

```python
import numpy as np
def top_k(query, corpus, k=5):
    q = query / np.linalg.norm(query)
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)
    sims = c @ q                      # một phép nhân ma trận thay cho vòng lặp
    idx = np.argsort(-sims)[:k]
    return idx, sims[idx]
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Embedding là gì?"**

Một dãy số nhiều chiều mã hoá ý nghĩa của một đối tượng, sao cho những thứ giống nghĩa nằm gần nhau trong không gian đó. Điểm ghi thêm: nêu phân biệt **static** (Word2Vec — một vector cố định mỗi từ) và **contextual** (transformer — vector phụ thuộc ngữ cảnh nhờ self-attention), và giải thích vì sao điều đó khiến transformer thắng.

**2. "Vì sao dùng cosine chứ không dùng Euclidean cho văn bản?"**

Cosine đo hướng và bỏ qua độ dài; với văn bản, hướng chính là chủ đề/ngữ nghĩa, còn độ dài thường chỉ phản ánh độ dài câu. Điểm ghi thêm: nói rằng **nếu vector đã được chuẩn hoá thì cosine và L2 cho cùng thứ tự xếp hạng**, nên lúc đó lựa chọn thuần tuý là chuyện hiệu năng chứ không phải chất lượng.

**3. [BẪY] "Cosine similarity chạy từ 0 đến 1, đúng không?"**

Không hẳn. Miền giá trị toán học là **[−1, 1]**, vì cosin của mọi góc nằm trong đó. Nó chỉ rơi vào [0, 1] khi mọi thành phần của cả hai vector đều không âm. Embedding hiện đại có thành phần âm nên về lý thuyết cosine có thể ra âm — dù trên thực tế các cặp văn bản thật sự giống nghĩa hầu như luôn cho giá trị dương và cao.

**4. [BẪY] "Tôi embed hai câu bằng transformer rồi so sánh, nhưng shape lệch nhau. Vì sao?"**

Vì thiếu bước **pooling**. Pipeline feature-extraction trả về một vector cho **mỗi token**, shape `[1, số_token, d]`. Hai câu dài ngắn khác nhau thì số token khác nhau. Cần `pooling: 'mean', normalize: true` (JavaScript) hoặc dùng `sentence-transformers` với `normalize_embeddings=True` (Python).

**5. "pgvector tự tạo embedding cho tôi, đúng không?"**

Không. pgvector chỉ lưu trữ vector và tìm vector gần nhất bằng chỉ mục ANN. Việc tạo ra vector là của một model embedding hoàn toàn tách biệt — bạn gọi model, lấy vector, rồi mới đưa vào pgvector. Model là đầu bếp, database là tủ lạnh.

**6. "Bạn tìm ra một model embedding tốt hơn. Làm gì với dữ liệu cũ?"**

Phải **embed lại toàn bộ**, vì vector cũ và mới nằm trong hai không gian khác nhau, không so sánh được — kể cả khi tình cờ cùng số chiều. Cách làm an toàn: thêm cột có đánh phiên bản chạy song song, backfill chạy nền có giới hạn tốc độ, ghi kép cho dữ liệu mới trong giai đoạn chuyển tiếp, rồi chuyển lưu lượng dần theo kiểu blue-green. Đây là dự án nhiều ngày chứ không phải một dòng cấu hình — nên tôi dành nhiều công sức hơn cho khâu chọn model ngay từ đầu.

**7. [TRADE-OFF] "Self-host model hay dùng API?"**

API: không phải vận hành hạ tầng, luôn được cập nhật, ra mắt nhanh — nhưng trả tiền theo lượng dùng, phụ thuộc bên thứ ba (rate limit, downtime, đổi phiên bản âm thầm), độ trễ mạng cao hơn, và **dữ liệu rời khỏi hệ thống**.

Self-host: rẻ về biên, dữ liệu ở nhà, độ trễ thấp hơn nhiều cho đường truy vấn — nhưng phải nuôi GPU, tự cập nhật, và cần người có kinh nghiệm vận hành.

Câu chốt: **quyền riêng tư thường là yếu tố quyết định** — nếu dữ liệu không được phép rời công ty thì mọi so sánh benchmark trở nên vô nghĩa.

**8. [SCALE] "Bạn cần embed 100 triệu tài liệu. Tối ưu những gì?"**

Theo thứ tự tác động: (1) **chunking** hợp lý để tránh truncation và tránh embed rác; (2) **batching** 32–256 câu mỗi lượt để tận dụng GPU; (3) **cache** theo hash của nội dung để không tính lại; (4) cân nhắc **giảm số chiều bằng Matryoshka** nếu RAM cho chỉ mục là ràng buộc; (5) chạy bất đồng bộ qua hàng đợi để chịu tải đột biến. Và trước tất cả: **tính chi phí bằng tiền trước khi cam kết** — 100 triệu tài liệu × 200 token là 20 tỉ token, nhân với đơn giá của model để ra con số cụ thể.

**9. "Bạn chọn embedding model thế nào?"**

Liệt kê ràng buộc trước tiên (loại dữ liệu, ngôn ngữ, quyền riêng tư, độ trễ, ngân sách) — ràng buộc cứng có thể loại thẳng model đứng đầu bảng. Rồi lập danh sách ngắn 2–3 model từ MTEB/MMTEB. Rồi **đo trên tập truy vấn thật có nhãn của chính mình** bằng recall@k và nDCG — đây là bước quyết định. Cuối cùng cân nhắc fine-tune nếu lĩnh vực rất hẹp. MTEB là giả định ban đầu, không phải phán quyết.

**10. [BẪY] "Hệ thống của bạn tìm kiếm ngày càng kém đi mà không ai đổi gì. Chuyện gì xảy ra?"**

Vài khả năng, xếp theo xác suất: (a) **nhà cung cấp âm thầm đổi phiên bản model** đứng sau tên API, khiến vector mới lệch khỏi vector cũ trong kho — phòng bằng cách ghim phiên bản và lưu tên model bên cạnh mỗi vector; (b) **drift** — dữ liệu và cách người dùng gõ đã thay đổi so với lúc model được huấn luyện; (c) tài liệu mới dài hơn và đang bị **truncation** âm thầm. Cả ba đều chỉ phát hiện được nếu bạn **đo recall@k trên tập vàng định kỳ**, không phải chỉ đo một lần lúc ra mắt.

### 5.6. Câu chốt "one-liner" đắt giá

Vài câu ngắn, sắc, thể hiện độ sâu. Dùng đúng lúc, mỗi câu đáng giá bằng cả đoạn giải thích.

- *"The model makes the vector; the database stores it — pgvector never embeds anything itself."*
- *"Cosine measures direction, not magnitude — which is exactly what you want for meaning."*
- *"Static embeddings give one vector per word; contextual embeddings give one vector per word **in context** — that's why transformers won."*
- *"Vectors from two different models don't live in the same space, so changing your embedding model means re-embedding everything."*
- *"MTEB narrows the shortlist; your own labeled queries make the decision."*
- *"Privacy can override the benchmark — sometimes you self-host a slightly weaker model because the data can't leave the building."*
- *"Truncation is the quietest bug in the stack: no error, no warning, just half your document silently missing from the index."*
- *"Don't forget the query is an embedding too — and it's on the hot path."*

---

## 📌 Ghi chú cuối

### Ba việc nên làm ngay sau khi đọc xong

**Một — Chạy thật một lần.** Cài `sentence-transformers` (Python) hoặc `@huggingface/transformers` (JavaScript), embed ba câu: hai câu đồng nghĩa và một câu không liên quan. Tự tính cosine bằng hàm ở Mục 5.4. Kiểm chứng bằng mắt rằng hai câu đồng nghĩa cho điểm cao hơn hẳn. Việc này mất 15 phút và biến kiến thức trên giấy thành kiến thức thật.

**Hai — Tự kiểm tra chất lượng tiếng Việt.** Viết ra 20 cặp câu tiếng Việt bạn *biết chắc* là đồng nghĩa, cộng 20 cặp ngẫu nhiên. Đo cosine cho cả 40 cặp. Nếu hai nhóm không tách bạch rõ ràng, model bạn chọn không phù hợp với tiếng Việt — đây chính là bước 3 của quy trình chọn model ở Mục 4.2, làm ở quy mô nhỏ.

**Ba — Đóng vòng với pgvector.** Lấy vector vừa tạo, cắm vào PostgreSQL có pgvector, chạy một câu truy vấn tìm kiếm ngữ nghĩa. Khi bạn thấy toàn bộ chuỗi **embed → lưu → tìm** chạy từ đầu đến cuối, mọi mảnh ghép mới thật sự khớp vào nhau trong đầu.

### Nối mạch với các bài khác trong series

```
   ┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐
   │   BÀI NÀY       │   │  BÀI pgvector    │   │   BÀI FTS       │
   │  Tạo vector     │ → │  Lưu & tìm vector│ + │  Tìm theo từ khoá│
   │  (embedding)    │   │  (ANN, HNSW)     │   │  (lexeme)       │
   └─────────────────┘   └──────────────────┘   └─────────────────┘
              ↓                    ↓                     ↓
         ══════════════════════════════════════════════════
                    HYBRID SEARCH → SEMANTIC SEARCH → RAG
         ══════════════════════════════════════════════════
                        Tất cả chạy trên một PostgreSQL
```

Ba mảnh ghép, một database. Embedding hiểu **nghĩa**, full-text bắt chính xác **từ khoá, mã sản phẩm, tên riêng** — kết hợp cả hai luôn cho kết quả tốt hơn từng cái riêng lẻ.

### Nhớ kiểm chứng lại khi ôn

- Tên gói JavaScript hiện là **`@huggingface/transformers`** (v4, ra tháng 2/2026, hỗ trợ WebGPU). Tên cũ `@xenova/transformers` vẫn cài được nhưng đã ngừng phát triển.
- Bức tranh model **thay đổi hàng tháng**. Bảng ở Mục 2.3 đúng tại tháng 7/2026. Trước khi phỏng vấn, mở lại **bảng xếp hạng MTEB trên Hugging Face** để cập nhật.
- Giá API cũng thay đổi liên tục. Các con số trong Mục 4.1 dùng để *minh hoạ cách tính*, không phải để trích dẫn — hãy tra bảng giá hiện hành khi cần con số thật.

### Học tiếp gì sau bài này

- **Chiến lược chunking cho RAG** — cách chia tài liệu ảnh hưởng đến chất lượng nhiều hơn bạn tưởng, thường hơn cả việc chọn model.
- **Reranking bằng cross-encoder** — bước tinh chỉnh sau khi ANN đã lọc ra 100 ứng viên, giúp cải thiện đáng kể độ chính xác của top 10.
- **Multimodal embedding** — nhúng chữ và ảnh vào cùng một không gian, cho phép tìm ảnh bằng chữ.
- **Đánh giá hệ thống retrieval** — recall@k, nDCG, MRR, và cách xây một tập vàng đủ tốt để tin được.

---

*Giáo trình soạn theo INSTRUCTION "giáo trình siêu dễ hiểu". Thông tin cập nhật tới tháng 7/2026 — các phần liên quan đến bảng xếp hạng model, tên gói phần mềm và giá API nên được kiểm chứng lại trước khi dùng cho quyết định thật.*
