# Vector Search với PostgreSQL (pgvector) — Giáo trình siêu dễ hiểu, từ Basic đến Staff Level
*Vector Search with PostgreSQL (pgvector) — A Very Easy Course, from Basic to Staff Level*

> **Bài giảng gốc — the original lecture**
>
> Giáo trình này dựa trên video mở đầu khoá *IBM Vector Database Fundamentals — Course 3*.  
> *This course is based on the opening video of IBM Vector Database Fundamentals — Course 3.*
>
> Video gốc chỉ nêu 3 mục tiêu học tập.  
> *The original video states only three learning objectives.*
>
> Mục tiêu thứ nhất là modeling vector data.  
> *The first objective is modeling vector data.*
>
> Mục tiêu thứ hai là indexing techniques.  
> *The second objective is indexing techniques.*
>
> Mục tiêu thứ ba là dùng PostgreSQL để store, retrieve và query vector.  
> *The third objective is using PostgreSQL to store, retrieve, and query vectors.*
>
> Video gốc chưa dạy nội dung kỹ thuật.  
> *The original video does not teach technical content yet.*
>
> Phần lớn giáo trình này là phần **giảng mở rộng** của tôi.  
> *Most of this course is my own **extended teaching**.*
>
> Tôi là một Staff Engineer.  
> *I am a Staff Engineer.*
>
> Tôi bám sát đúng 3 mục tiêu đó.  
> *I stay close to those three objectives.*
>
> Những chỗ đi xa hơn bài gốc được đánh dấu 🧩 **[Ngoài bài gốc]**.  
> *Parts that go beyond the original are marked 🧩 **[Ngoài bài gốc]**.*

> **Cập nhật tới tháng 7/2026 — update through July 2026**
>
> pgvector đang ở nhánh **0.8.x**.  
> *pgvector is on the **0.8.x** branch.*
>
> Bản mới nhất trên GitHub là **v0.8.5**.  
> *The latest release on GitHub is **v0.8.5**.*
>
> **HNSW** là loại index được khuyên dùng mặc định.  
> ***HNSW** is the recommended default index type.*
>
> Đây là một lưu ý vận hành quan trọng.  
> *Here is an important operational note.*
>
> Bản **0.8.2 (2026-02)** vá một lỗ hổng bảo mật.  
> *Release **0.8.2 (2026-02)** patches a security hole.*
>
> Lỗ hổng đó mang mã CVE-2026-3172.  
> *That hole carries the identifier CVE-2026-3172.*
>
> Lỗ hổng đó gây tràn bộ nhớ khi build index HNSW song song.  
> *That hole causes a memory overflow during parallel HNSW index builds.*
>
> Bạn có thể đang chạy bản 0.8.0 hoặc 0.8.1.  
> *You may be running 0.8.0 or 0.8.1.*
>
> Khi đó bạn nên nâng cấp.  
> *In that case you should upgrade.*
>
> Trước khi đi phỏng vấn, bạn hãy liếc lại CHANGELOG trên GitHub `pgvector/pgvector`.  
> *Before an interview, glance at the CHANGELOG on GitHub at `pgvector/pgvector`.*
>
> Lý do là dự án này phát triển rất nhanh.  
> *The reason is the very fast pace of this project.*

> **Cách đọc file này — how to read this file**
>
> Bạn đọc tuần tự từ Phần 0 tới Phần 5.  
> *Read straight through from Part 0 to Part 5.*
>
> Mỗi phần đều dựa trên phần trước.  
> *Each part builds on the previous one.*
>
> Bạn có thể gặp một từ lạ.  
> *You may hit an unfamiliar word.*
>
> Bạn đừng lo.  
> *Do not worry.*
>
> Quy tắc của giáo trình này rất rõ ràng.  
> *The rule of this course is very clear.*
>
> **Mọi thuật ngữ đều được giải thích ngay lần đầu nó xuất hiện.**  
> ***Every term gets an explanation at its first appearance.***
>
> Bạn có thể thấy một từ chưa được giải thích.  
> *You may find a word without an explanation.*
>
> Đó là lỗi của tôi.  
> *That is my fault.*
>
> Đó không phải lỗi của bạn.  
> *That is not your fault.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson map (Overview)*

### Bài này dạy gì — nói bằng một câu đơn giản nhất
*What this lesson teaches — in the simplest possible sentence*

**Bài này dạy bạn cách cho một database bình thường khả năng "tìm theo ý nghĩa".**  
***This lesson teaches you how to give an ordinary database the ability to "search by meaning".***

Database đó sẽ không chỉ "tìm theo đúng chữ" nữa.  
*That database will no longer only "search by exact characters".*

Đây là một ví dụ.  
*Here is an example.*

Khách hàng gõ "đồ giữ ấm mùa đông".  
*A customer types "đồ giữ ấm mùa đông".*

Hệ thống trả về "áo khoác lông vũ".  
*The system returns "áo khoác lông vũ".*

Tên sản phẩm đó **không có một chữ nào** trùng với câu khách gõ.  
*That product name shares **not a single word** with the customer's query.*

Máy tính vẫn hiểu được hai thứ này *liên quan về mặt ý nghĩa*.  
*The computer still understands that the two are *related in meaning*.*

Công cụ để làm điều đó tên là **pgvector**.  
*The tool for this job is called **pgvector**.*

Ngay bây giờ bạn có thể chưa biết pgvector là gì.  
*Right now you may not know what pgvector is.*

Bạn có thể chưa biết PostgreSQL là gì.  
*You may not know what PostgreSQL is.*

Bạn có thể chưa biết "vector" nghĩa là gì.  
*You may not know what "vector" means.*

Điều đó **hoàn toàn bình thường**.  
*That is **completely normal**.*

Phần 1 sẽ giải thích từng chữ một, từ con số 0.  
*Part 1 explains every word, starting from zero.*

Phần 1 không giả định bạn biết trước bất cứ điều gì.  
*Part 1 assumes no prior knowledge at all.*

### Vấn đề thực tế mà nó giải quyết
*The real problem it solves*

Database truyền thống rất giỏi việc **tìm khớp chính xác**.  
*A traditional database is very good at **exact matching**.*

Nó trả lời tốt câu hỏi "cho tôi bản ghi nào có tên đúng bằng chữ *áo thun*".  
*It answers "give me the record whose name is exactly *áo thun*" very well.*

Nó **không hiểu** rằng "áo phông" gần nghĩa với "áo thun".  
*It does **not understand** that "áo phông" is close in meaning to "áo thun".*

Nó cũng không hiểu "T-shirt" gần nghĩa với "áo thun".  
*It also does not understand that "T-shirt" is close in meaning to "áo thun".*

Nó cũng không hiểu "đồ mặc mùa hè" gần nghĩa với "áo thun".  
*It also does not understand that "đồ mặc mùa hè" is close in meaning to "áo thun".*

Con người thì hiểu ngay.  
*A human understands immediately.*

Máy thì không.  
*A machine does not.*

**Vector search** — tìm kiếm bằng vector — lấp vào đúng khoảng trống đó.  
***Vector search** fills exactly that gap.*

Đây là nền tảng của ba thứ đang cực kỳ hot hiện nay.  
*This is the foundation of three very hot things today.*

**Semantic search — tìm kiếm ngữ nghĩa**

Semantic search tìm theo ý.  
*Semantic search searches by intent.*

Semantic search không tìm theo từ khoá.  
*Semantic search does not search by keyword.*

**Recommendation — gợi ý sản phẩm**

Recommendation nói rằng "người mua món này cũng thích món kia".  
*Recommendation says "people who bought this also liked that".*

**RAG**

RAG viết tắt của *Retrieval-Augmented Generation*.  
*RAG stands for *Retrieval-Augmented Generation*.*

Tên này tạm dịch là "sinh văn bản có tra cứu bổ trợ".  
*The Vietnamese gloss is "sinh văn bản có tra cứu bổ trợ".*

Đây là kỹ thuật cho chatbot AI tra cứu tài liệu riêng của công ty trước khi trả lời.  
*This technique lets an AI chatbot look up private company documents before answering.*

Mục đích là để nó không bịa.  
*The purpose is to stop it from making things up.*

Bước "tra cứu" trong RAG chính là vector search.  
*The retrieval step in RAG is exactly vector search.*

### Học xong bạn sẽ làm được
*What you will be able to do*

Bạn sẽ giải thích được **embedding là gì**.  
*You will be able to explain **what an embedding is**.*

Bạn sẽ giải thích được một điều nữa.  
*You will be able to explain one more thing.*

"Khoảng cách giữa hai vector" chính là "độ giống nhau về ý nghĩa".  
*The distance between two vectors is exactly the similarity in meaning.*

Bạn sẽ thiết kế được **cấu trúc bảng** để lưu vector trong PostgreSQL.  
*You will be able to design a **table structure** for storing vectors in PostgreSQL.*

Đây là mục tiêu *data modeling* của bài gốc.  
*This is the *data modeling* objective of the original lesson.*

Bạn sẽ **lưu, truy vấn và lọc** vector bằng SQL.  
*You will **store, query, and filter** vectors with SQL.*

Bạn sẽ dùng được 3 thước đo khoảng cách chính.  
*You will use the three main distance measures.*

Bạn sẽ chọn và tinh chỉnh đúng loại **index**.  
*You will pick and tune the right **index** type.*

Các loại index gồm HNSW, IVFFlat và DiskANN.  
*The index types include HNSW, IVFFlat, and DiskANN.*

Bạn sẽ hiểu độ phức tạp tính toán của chúng.  
*You will understand their computational complexity.*

Đây là mục tiêu *indexing techniques* của bài gốc.  
*This is the *indexing techniques* objective of the original lesson.*

Bạn sẽ phân tích được **đánh đổi ở quy mô lớn**.  
*You will be able to analyze **trade-offs at large scale**.*

Quy mô đó chạy từ triệu tới tỷ vector.  
*That scale runs from millions to billions of vectors.*

Bạn sẽ tính được chi phí.  
*You will be able to compute the cost.*

Bạn sẽ trả lời trôi chảy một câu hỏi phỏng vấn kiểu *system design* về vector search.  
*You will fluently answer a *system design* interview question about vector search.*

### Mạch kiến thức: basic → staff
*The knowledge path: basic to staff*

🟢 **Basic**

Phần này giải thích embedding là gì.  
*This part explains what an embedding is.*

Phần này giải thích một điều nữa.  
*This part explains one more thing.*

Khoảng cách chính là độ giống nhau.  
*Distance is exactly similarity.*

Phần này hướng dẫn cài pgvector.  
*This part walks through installing pgvector.*

Phần này hướng dẫn tạo cột kiểu `vector`.  
*This part walks through creating a `vector` column.*

Phần này giúp bạn viết câu tìm kiếm đầu tiên bằng toán tử `<->`.  
*This part helps you write your first search using the `<->` operator.*

🟡 **Intermediate**

Phần này dạy ba thước đo khoảng cách.  
*This part teaches the three distance measures.*

Ba thước đo đó là L2, cosine và inner product.  
*Those three measures are L2, cosine, and inner product.*

Phần này dạy thiết kế bảng thật.  
*This part teaches real table design.*

Phần này dạy tạo index HNSW.  
*This part teaches how to create an HNSW index.*

Phần này dạy viết một pipeline Python hoàn chỉnh.  
*This part teaches how to write a complete Python pipeline.*

Phần này dạy tìm kiếm có lọc điều kiện.  
*This part teaches filtered search.*

Phần này nêu 3 lỗi kinh điển.  
*This part names three classic mistakes.*

🔴 **Advanced**

Phần này mở bên trong HNSW và IVFFlat.  
*This part opens up HNSW and IVFFlat.*

Phần này nói về độ phức tạp Big-O.  
*This part covers Big-O complexity.*

Phần này nói về "lời nguyền số chiều".  
*This part covers the "curse of dimensionality".*

Phần này hướng dẫn tự viết lại thuật toán bằng Python.  
*This part guides you to reimplement the algorithms in Python.*

Phần này nêu các trường hợp biên.  
*This part names the edge cases.*

Hai trường hợp biên là giới hạn 2000 chiều và chuẩn hoá vector.  
*Two edge cases are the 2000-dimension limit and vector normalization.*

🟣 **Staff**

Phần này mở rộng lên hàng tỷ vector.  
*This part scales up to billions of vectors.*

Phần này nói về chia nhỏ dữ liệu.  
*This part covers data partitioning.*

Phần này nói về nén vector để cắt chi phí.  
*This part covers vector compression for cost cutting.*

Phần này nói về giám sát chất lượng tìm kiếm.  
*This part covers monitoring search quality.*

Phần này nói khi nào bạn **không** nên dùng pgvector.  
*This part says when you should **not** use pgvector.*

Phần này đưa ra một câu hỏi system design mẫu.  
*This part presents a sample system design question.*

🎯 **Cheatsheet**

Phần này gom bảng từ khoá.  
*This part collects the keyword table.*

Phần này gom các ý cốt lõi.  
*This part collects the core ideas.*

Phần này gom code cần thuộc lòng.  
*This part collects the code you must memorize.*

Phần này gom câu hỏi phỏng vấn.  
*This part collects interview questions.*

Phần này gom các câu chốt "đắt giá".  
*This part collects the high-value closing lines.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

> Phần này viết cho người **chưa biết gì**.  
> *This part is written for someone who **knows nothing yet**.*
>
> Tôi sẽ đi rất chậm.  
> *I will go very slowly.*
>
> Bạn có thể đã là dev backend quen Postgres.  
> *You may already be a backend dev familiar with Postgres.*
>
> Khi đó bạn có thể đọc lướt mục 1.1 tới 1.3.  
> *In that case you can skim sections 1.1 through 1.3.*
>
> Bạn đừng bỏ mục 1.5 về ví dụ chạy tay.  
> *Do not skip section 1.5 on the hand-worked example.*
>
> Mục đó là chìa khoá để hiểu mọi thứ phía sau.  
> *That section is the key to everything that follows.*

### 1.1. Bắt đầu từ nỗi đau: vì sao tìm kiếm truyền thống thất bại
*1.1. Starting from the pain: why traditional search fails*

Trước khi nói về giải pháp, ta phải *cảm* được vấn đề.  
*Before discussing the solution, we must *feel* the problem.*

Bạn không thấy nỗi đau.  
*Suppose you do not feel the pain.*

Khi đó bạn sẽ không nhớ nổi liều thuốc.  
*Then you will not remember the cure.*

Hãy tưởng tượng bạn đang làm một shop bán hàng online.  
*Imagine you are building an online shop.*

Bạn có một **database** — cơ sở dữ liệu.  
*You have a **database**.*

Database là nơi lưu trữ dữ liệu có tổ chức của ứng dụng.  
*A database is where an application stores data in an organized way.*

Dữ liệu không còn để rải rác trong file Excel.  
*The data no longer lies scattered in Excel files.*

Trong database đó có một **table** — bảng.  
*Inside that database there is a **table**.*

Table giống một sheet Excel.  
*A table is like an Excel sheet.*

Table có các cột và các dòng.  
*A table has columns and rows.*

Bảng `products` của bạn có 3 dòng.  
*Your `products` table has three rows.*

Mỗi dòng gọi là một **row** hoặc **record** — bản ghi.  
*Each line is called a **row** or a **record**.*

Một record chứa thông tin về một sản phẩm cụ thể.  
*A record holds the information about one specific product.*

| id | name |
|---|---|
| 1 | áo khoác lông vũ |
| 2 | khăn len |
| 3 | găng tay |

Bây giờ khách hàng gõ vào ô tìm kiếm.  
*Now the customer types into the search box.*

Khách gõ **"đồ giữ ấm mùa đông"**.  
*The customer types **"đồ giữ ấm mùa đông"**.*

Ứng dụng của bạn sẽ chạy một **query** — câu truy vấn.  
*Your application then runs a **query**.*

Query là một câu lệnh hỏi database.  
*A query is a command that asks the database a question.*

Câu hỏi đó là "cho tôi dữ liệu thoả điều kiện này".  
*That question is "give me the data matching this condition".*

Query được viết bằng **SQL**.  
*A query is written in **SQL**.*

SQL viết tắt của *Structured Query Language*.  
*SQL stands for *Structured Query Language*.*

Đây là ngôn ngữ tiêu chuẩn để nói chuyện với database quan hệ.  
*This is the standard language for talking to a relational database.*

Đây là câu SQL kinh điển cho ô tìm kiếm.  
*Here is the classic SQL for a search box.*

```sql
SELECT name FROM products
WHERE name LIKE '%đồ giữ ấm mùa đông%';
```

Tôi giải thích từng chữ trong câu trên.  
*I will explain every word in that statement.*

Đây là câu SQL đầu tiên trong giáo trình.  
*This is the first SQL statement in this course.*

`SELECT name` nghĩa là "lấy cho tôi cột `name`".  
*`SELECT name` means "give me the `name` column".*

`FROM products` nghĩa là "từ bảng tên là `products`".  
*`FROM products` means "from the table called `products`".*

`WHERE ...` nghĩa là "chỉ lấy những dòng thoả điều kiện sau".  
*`WHERE ...` means "return only the rows matching the following condition".*

**LIKE** là phép so khớp chuỗi ký tự.  
***LIKE** is a character-string matching operation.*

Dấu `%` nghĩa là "bất cứ thứ gì cũng được ở vị trí này".  
*The `%` sign means "anything at all can sit in this position".*

Vậy `'%đồ giữ ấm mùa đông%'` nghĩa là "tên sản phẩm có chứa đâu đó cụm chữ đó".  
*So `'%đồ giữ ấm mùa đông%'` means "the product name contains that phrase somewhere".*

Kết quả trả về là **0 dòng**.  
*The result is **zero rows**.*

Khách hàng thấy trang trắng.  
*The customer sees a blank page.*

Khách hàng thoát ra.  
*The customer leaves.*

Bạn mất một đơn hàng.  
*You lose an order.*

Vì sao lại như vậy?  
*Why does this happen?*

Không có sản phẩm nào **chứa đúng cụm ký tự** đó.  
*No product **contains that exact character sequence**.*

`LIKE` so khớp từng **ký tự**.  
*`LIKE` matches **character** by character.*

Ký tự trong tiếng Anh là *character*, tức từng chữ cái một.  
*A character is a single letter.*

`LIKE` tuyệt đối không hiểu *ý nghĩa*.  
*`LIKE` understands absolutely nothing about *meaning*.*

Với `LIKE`, "áo khoác lông vũ" và "đồ giữ ấm mùa đông" là hai chuỗi ký tự khác nhau hoàn toàn.  
*For `LIKE`, "áo khoác lông vũ" and "đồ giữ ấm mùa đông" are two entirely different strings.*

Chúng xa lạ với nhau như tiếng Việt với tiếng Nhật.  
*They are as foreign to each other as Vietnamese and Japanese.*

Bộ não bạn thì hiểu ngay.  
*Your brain understands immediately.*

Áo khoác lông vũ **chính là** đồ giữ ấm mùa đông.  
*"Áo khoác lông vũ" **is exactly** "đồ giữ ấm mùa đông".*

Vấn đề của chúng ta gói gọn trong một câu.  
*Our problem fits into one sentence.*

**Làm sao dạy máy tính cái hiểu biết đó?**  
***How do we teach a computer that understanding?***

Đó chính xác là lý do **vector search** ra đời.  
*That is exactly why **vector search** exists.*

### 1.2. Analogy: tấm bản đồ ý nghĩa
*1.2. Analogy: the map of meaning*

Đây là hình ảnh quan trọng nhất trong toàn bộ giáo trình.  
*This is the most important image in the whole course.*

Bạn chỉ nhớ được một thứ duy nhất.  
*Suppose you can remember only one thing.*

Hãy nhớ cái này.  
*Remember this one.*

Tôi sẽ dựng nó thật đầy đủ.  
*I will build it out fully.*

Tôi sẽ không nói lướt.  
*I will not rush through it.*

Hãy tưởng tượng một **tấm bản đồ khổng lồ**.  
*Imagine a **giant map**.*

Ta không đặt các thành phố lên đó.  
*We do not put cities on it.*

Ta đặt **từ ngữ, câu chữ, sản phẩm** lên đó.  
*We put **words, sentences, and products** on it.*

Quy tắc đặt duy nhất và bất di bất dịch nằm ngay dưới đây.  
*The single unchanging placement rule sits right below.*

> **Thứ nào giống nhau về ý nghĩa thì được đặt gần nhau trên bản đồ.**  
> ***Things alike in meaning are placed near each other on the map.***

Ta tuân theo quy tắc đó.  
*We follow that rule.*

Khi đó bản đồ sẽ tự động phân vùng.  
*The map then partitions itself automatically.*

Cụm thứ nhất gồm "áo khoác lông vũ", "áo phao", "khăn len" và "găng tay".  
*The first cluster holds "áo khoác lông vũ", "áo phao", "khăn len", and "găng tay".*

Cụm này tụ lại ở góc trên bên trái.  
*This cluster gathers in the upper left corner.*

Ta có thể gọi đó là "vùng đồ mùa đông".  
*We can call it the "winter clothing region".*

Chẳng ai dán nhãn cho nó cả.  
*Nobody labeled it.*

Chúng tự tụ lại với nhau.  
*They gather on their own.*

Lý do là nghĩa của chúng giống nhau.  
*The reason is their shared meaning.*

Cụm thứ hai gồm "kem chống nắng", "quần short" và "áo ba lỗ".  
*The second cluster holds "kem chống nắng", "quần short", and "áo ba lỗ".*

Cụm này tụ ở góc dưới bên phải.  
*This cluster gathers in the lower right corner.*

Ta gọi đó là "vùng đồ mùa hè".  
*We call it the "summer clothing region".*

Cụm thứ ba gồm "bàn phím cơ" và "chuột máy tính".  
*The third cluster holds "bàn phím cơ" and "chuột máy tính".*

Cụm này nằm cách xa cả hai vùng trên.  
*This cluster sits far from both regions above.*

Lý do là nó chẳng liên quan gì tới quần áo.  
*The reason is its total lack of connection to clothing.*

Bây giờ khách gõ **"đồ giữ ấm mùa đông"**.  
*Now the customer types **"đồ giữ ấm mùa đông"**.*

Ta làm hai bước.  
*We take two steps.*

**Bước 1 / Step 1**

Ta cũng đặt chính câu truy vấn đó lên bản đồ.  
*We place that very query onto the map too.*

Ta đặt nó theo đúng quy tắc trên.  
*We place it by the same rule.*

Nó sẽ rơi vào đâu?  
*Where does it land?*

Nó rơi vào giữa vùng đồ mùa đông.  
*It lands in the middle of the winter clothing region.*

Lý do là nghĩa của nó gần với "áo phao" và "khăn len".  
*The reason is its closeness in meaning to "áo phao" and "khăn len".*

**Bước 2 / Step 2**

Ta hỏi bản đồ một câu duy nhất.  
*We ask the map a single question.*

**"Những điểm nào nằm gần điểm này nhất?"**  
***"Which points lie nearest to this point?"***

Trả lời là áo phao, khăn len và áo khoác lông vũ.  
*The answer is "áo phao", "khăn len", and "áo khoác lông vũ".*

Xong.  
*Done.*

Đó chính là kết quả tìm kiếm ta muốn.  
*That is exactly the search result we want.*

Bây giờ hãy để ý chỗ **khớp** giữa phép ví von và kỹ thuật.  
*Now notice the **join** between the metaphor and the technology.*

Đây là điểm mấu chốt.  
*This is the crux.*

Câu hỏi "tìm sản phẩm giống nghĩa nhất" là một câu hỏi về **ngôn ngữ**.  
*The question "find the most similar product by meaning" is a question about **language**.*

Câu hỏi về ngôn ngữ rất khó cho máy tính.  
*A question about language is very hard for a computer.*

Câu hỏi đó đã biến thành "tìm điểm gần nhất trên bản đồ".  
*That question has turned into "find the nearest point on the map".*

Đây là một câu hỏi về **hình học**.  
*This is a question about **geometry**.*

Máy tính giải nó bằng phép trừ và phép căn bậc hai.  
*A computer solves it with subtraction and square roots.*

Việc đó cực kỳ dễ.  
*That job is extremely easy.*

**Toàn bộ ngành vector search chỉ là việc thực hiện phép biến đổi đó.**  
***The entire field of vector search is just performing that transformation.***

Nó biến bài toán ngữ nghĩa thành bài toán khoảng cách.  
*It turns a semantic problem into a distance problem.*

Có hai chỗ mà phép ví von này hơi khác thực tế.  
*The metaphor differs from reality in two places.*

Tôi nói thẳng ra để bạn không bị bất ngờ sau này.  
*I state them plainly so nothing surprises you later.*

1. Bản đồ thật **không phải 2 chiều**.  
   *1. The real map is **not two-dimensional**.*

   Bản đồ 2 chiều chỉ có dài và rộng.  
   *A two-dimensional map has only length and width.*

   Bản đồ thật có hàng trăm tới hàng nghìn chiều.  
   *The real map has hundreds to thousands of dimensions.*

   Bạn *không thể* hình dung nổi 1536 chiều.  
   *You *cannot* picture 1536 dimensions.*

   Không ai hình dung được cả.  
   *Nobody can picture them.*

   Điều đó đúng với cả các nhà nghiên cứu.  
   *That holds true even for researchers.*

   May mắn thay, công thức tính khoảng cách vẫn y hệt.  
   *Fortunately the distance formula stays identical.*

   Nó chỉ có nhiều số hạng hơn.  
   *It merely has more terms.*

   Ta sẽ thấy tận mắt ở mục 1.5.  
   *We will see it with our own eyes in section 1.5.*

2. Ai vẽ bản đồ?  
   *2. Who draws the map?*

   Người vẽ không phải pgvector.  
   *The one drawing it is not pgvector.*

   Đó là việc của một **mô hình AI** riêng biệt.  
   *That job belongs to a separate **AI model**.*

   Tôi sẽ nói ngay dưới đây.  
   *I will cover it right below.*

### 1.3. Từ bản đồ đến con số: embedding là gì
*1.3. From map to numbers: what an embedding is*

Trên một tấm bản đồ, mỗi điểm được xác định bằng **toạ độ**.  
*On a map, every point is identified by its **coordinates**.*

Ví dụ điểm A ở toạ độ (2, 9).  
*For example, point A sits at coordinates (2, 9).*

Toạ độ đó nghĩa là đi ngang 2 và đi lên 9.  
*Those coordinates mean two steps across and nine steps up.*

Toạ độ của một vật trên "bản đồ ý nghĩa" chính là **embedding**.  
*The coordinates of a thing on the "map of meaning" are exactly its **embedding**.*

Tôi giải thích cho tận gốc.  
*Let me explain it down to the root.*

**Embedding**

Từ này đến từ động từ *embed* trong tiếng Anh.  
*The word comes from the English verb *embed*.*

Nghĩa đen của nó là "nhúng vào" hoặc "gắn vào".  
*Its literal sense is "to nest inside" or "to attach into".*

Trong ngữ cảnh này, embedding là **một dãy số biểu diễn ý nghĩa của một vật**.  
*In this context, an embedding is **a sequence of numbers representing the meaning of a thing**.*

Vật đó có thể là một đoạn chữ.  
*That thing can be a piece of text.*

Vật đó có thể là một tấm ảnh.  
*That thing can be an image.*

Vật đó có thể là một đoạn âm thanh.  
*That thing can be an audio clip.*

Ta "nhúng" ý nghĩa của vật vào một dãy số.  
*We "embed" the meaning of the thing into a sequence of numbers.*

> Đây là một ví dụ cụ thể.  
> *Here is a concrete example.*
>
> Câu "áo khoác lông vũ" có thể được biến thành dãy `[0.12, -0.98, 0.33, 0.07, ...]`.  
> *The phrase "áo khoác lông vũ" can turn into the sequence `[0.12, -0.98, 0.33, 0.07, ...]`.*
>
> Dãy đó gồm 1536 con số.  
> *That sequence holds 1536 numbers.*
>
> Đây là một analogy.  
> *Here is an analogy.*
>
> Cứ hình dung nó như một **tấm nhãn toạ độ** dán lên món hàng.  
> *Picture it as a **coordinate label** stuck onto the product.*
>
> Nhìn vào nhãn là biết món hàng nằm ở đâu trên bản đồ ý nghĩa.  
> *One look at the label tells you where the product sits on the map of meaning.*

**Vector**

Trong ngữ cảnh này, "vector" và "embedding" gần như chỉ cùng một thứ.  
*In this context, "vector" and "embedding" almost name the same thing.*

Nói chính xác thì *vector* là kiểu dữ liệu.  
*Strictly speaking, a *vector* is a data type.*

Kiểu dữ liệu đó là một mảng số thực có độ dài cố định.  
*That data type is a fixed-length array of real numbers.*

*Embedding* là ý nghĩa mà vector đó mang.  
*An *embedding* is the meaning that the vector carries.*

Trong đời sống hàng ngày, dân trong nghề dùng lẫn lộn hai từ.  
*In daily life, practitioners use the two words interchangeably.*

Điều đó không sao cả.  
*That is perfectly fine.*

**Dimension — số chiều**

Dimension là **độ dài của dãy số**.  
*A dimension count is **the length of the number sequence**.*

Nó cho biết dãy đó có bao nhiêu con số.  
*It tells you how many numbers the sequence holds.*

Dãy `[2, 9, 0]` có 3 dimensions.  
*The sequence `[2, 9, 0]` has three dimensions.*

Model `text-embedding-3-small` của OpenAI sinh ra vector **1536 dimensions**.  
*OpenAI's `text-embedding-3-small` model produces **1536-dimension** vectors.*

Nghĩa là mỗi đoạn chữ biến thành một dãy 1536 con số.  
*That means each piece of text becomes a sequence of 1536 numbers.*

> Vì sao cần nhiều chiều đến thế?  
> *Why do we need so many dimensions?*
>
> Ý nghĩa của ngôn ngữ rất phức tạp.  
> *The meaning of language is very complex.*
>
> Với 2 chiều, bạn chỉ diễn tả được vài khía cạnh.  
> *With two dimensions you can express only a few aspects.*
>
> Với 1536 chiều, model có 1536 "trục" khác nhau.  
> *With 1536 dimensions, the model gets 1536 different "axes".*
>
> Các trục đó ghi lại các sắc thái.  
> *Those axes record the nuances.*
>
> Một sắc thái là trang trọng hay suồng sã.  
> *One nuance is formal versus casual.*
>
> Một sắc thái khác là vật thể hay khái niệm.  
> *Another nuance is object versus concept.*
>
> Một sắc thái khác nữa là tích cực hay tiêu cực.  
> *Yet another nuance is positive versus negative.*
>
> Còn hơn 1500 sắc thái khác nữa.  
> *More than 1500 further nuances remain.*
>
> Không con người nào đặt tên nổi chúng.  
> *No human can name them all.*

**Embedding model — mô hình sinh embedding**

Embedding model là **một mô hình AI đã được huấn luyện sẵn**.  
*An embedding model is **a pre-trained AI model**.*

Nhiệm vụ của nó là nhận vào một đoạn chữ.  
*Its job is to take in a piece of text.*

Nó trả ra một dãy số.  
*It returns a sequence of numbers.*

Chính nó là "người vẽ bản đồ".  
*It is exactly the "mapmaker".*

Nó học được cách vẽ bằng cách đọc hàng tỷ câu chữ trên internet.  
*It learned to draw by reading billions of sentences on the internet.*

Nó tự rút ra rằng "áo khoác" hay đi cùng ngữ cảnh với "mùa đông".  
*It figured out on its own that "áo khoác" often shares context with "mùa đông".*

Vậy nên nó đặt chúng gần nhau.  
*It therefore places them near each other.*

Đây là các model phổ biến.  
*Here are the popular models.*

`text-embedding-3-small` là model của OpenAI.  
*`text-embedding-3-small` is a model from OpenAI.*

Bạn gọi nó qua mạng.  
*You call it over the network.*

Nó tính phí.  
*It charges a fee.*

`all-MiniLM-L6-v2` chạy ngay trên máy bạn.  
*`all-MiniLM-L6-v2` runs right on your machine.*

Nó miễn phí.  
*It is free.*

**Điểm cực kỳ quan trọng cần khắc cốt ngay từ bây giờ / A crucial point to burn in right now**

pgvector **không** sinh ra embedding.  
*pgvector does **not** generate embeddings.*

Nó chỉ *lưu* và *tìm* các dãy số đó.  
*It only *stores* and *searches* those number sequences.*

Việc biến chữ "áo phao" thành `[2, 9, 0]` là việc của embedding model.  
*Turning the words "áo phao" into `[2, 9, 0]` is the embedding model's job.*

Embedding model là một hệ thống hoàn toàn riêng biệt.  
*The embedding model is a completely separate system.*

Đây là kiến trúc gồm hai mảnh.  
*This is a two-piece architecture.*

Người mới rất hay gộp nhầm hai mảnh làm một.  
*Beginners very often merge the two pieces by mistake.*

### 1.4. Bảng thuật ngữ còn lại của tầng Basic
*1.4. The remaining glossary for the Basic tier*

Trước khi vào code, tôi trả nốt các món nợ thuật ngữ.  
*Before we reach the code, I settle the remaining terminology debts.*

Đến lúc gõ code bạn sẽ không phải đoán chữ nào cả.  
*By the time you type code, you will guess at nothing.*

> **PostgreSQL — hệ quản trị cơ sở dữ liệu PostgreSQL**
>
> PostgreSQL thường được gọi tắt là **Postgres**.  
> *PostgreSQL is often shortened to **Postgres**.*
>
> Đây là một phần mềm quản trị cơ sở dữ liệu.  
> *This is a database management program.*
>
> Nó là mã nguồn mở và miễn phí.  
> *It is open source and free.*
>
> Nó cực kỳ phổ biến trong ngành.  
> *It is extremely popular in the industry.*
>
> Nó thuộc loại **RDBMS**.  
> *It belongs to the **RDBMS** category.*
>
> RDBMS viết tắt của *Relational Database Management System*.  
> *RDBMS stands for *Relational Database Management System*.*
>
> Tên tiếng Việt là hệ quản trị cơ sở dữ liệu quan hệ.  
> *The Vietnamese name is "hệ quản trị cơ sở dữ liệu quan hệ".*
>
> "Quan hệ" ở đây nghĩa là dữ liệu được tổ chức thành các bảng có cột và dòng.  
> *"Relational" here means the data is organized into tables with columns and rows.*
>
> Các bảng có thể liên kết với nhau.  
> *The tables can link to each other.*

> **Extension — phần mở rộng**
>
> Extension là một gói phần mềm cài thêm vào Postgres.  
> *An extension is a software package installed into Postgres.*
>
> Gói đó cho Postgres thêm tính năng mới.  
> *That package gives Postgres new features.*
>
> Nó giống như cài extension cho trình duyệt Chrome.  
> *It works like installing an extension for the Chrome browser.*
>
> Postgres được thiết kế để cho phép làm việc này.  
> *Postgres was designed to allow this.*
>
> Đó là một trong những lý do nó mạnh.  
> *That is one reason for its strength.*

> **pgvector**
>
> pgvector chính là một extension như vậy.  
> *pgvector is exactly such an extension.*
>
> Nó là mã nguồn mở.  
> *It is open source.*
>
> Nó giúp Postgres biết lưu và tìm kiếm vector.  
> *It teaches Postgres to store and search vectors.*
>
> Cụ thể nó thêm cho Postgres một **kiểu dữ liệu** mới tên là `vector`.  
> *Concretely it adds a new **data type** called `vector` to Postgres.*
>
> Postgres vốn đã có các kiểu sẵn khác.  
> *Postgres already has other built-in types.*
>
> Ví dụ là `text` cho chữ và `int` cho số nguyên.  
> *Examples are `text` for text and `int` for integers.*

> **Distance — khoảng cách / similarity — độ tương đồng**
>
> Đây là thước đo mức độ "gần nhau" giữa hai vector.  
> *This measures how "close" two vectors are.*
>
> Bạn cần nhớ một quy ước.  
> *You need to remember one convention.*
>
> **Khoảng cách nhỏ nghĩa là giống nhau nhiều.**  
> ***A small distance means high similarity.***
>
> Hai khái niệm này ngược chiều nhau.  
> *The two concepts point in opposite directions.*
>
> Khoảng cách càng nhỏ thì độ tương đồng càng lớn.  
> *The smaller the distance, the larger the similarity.*
>
> Chi tiết ba loại khoảng cách nằm ở Phần 2.  
> *Details on the three distance types sit in Part 2.*

> **Nearest Neighbor Search — tìm kiếm hàng xóm gần nhất**
>
> Tên này viết tắt là **NN search**.  
> *This name is abbreviated to **NN search**.*
>
> Bài toán được phát biểu như sau.  
> *The problem is stated as follows.*
>
> Cho một điểm truy vấn, hãy tìm các điểm gần nó nhất trong tập dữ liệu.  
> *Given a query point, find the points nearest to it in the dataset.*
>
> Ta có thể muốn lấy đúng `k` điểm gần nhất.  
> *We may want exactly the `k` nearest points.*
>
> Ví dụ là 10 sản phẩm giống nhất.  
> *An example is the ten most similar products.*
>
> Trường hợp đó gọi là **k-NN**.  
> *That case is called **k-NN**.*
>
> k-NN viết đầy đủ là *k nearest neighbors*.  
> *k-NN spells out as *k nearest neighbors*.*

> **Similarity search — tìm kiếm theo độ tương đồng**
>
> Đây là tên gọi khác của cùng bài toán trên.  
> *This is another name for the same problem.*
>
> Tên này nhìn từ góc độ ứng dụng.  
> *This name comes from the application angle.*
>
> Tên kia nhìn từ góc độ toán học.  
> *The other name comes from the mathematical angle.*

> **psql**
>
> psql là chương trình dòng lệnh chính thức để gõ câu SQL trực tiếp vào Postgres.  
> *psql is the official command-line program for typing SQL straight into Postgres.*
>
> Tôi sẽ nói "chạy câu SQL này" nhiều lần.  
> *I will say "run this SQL" many times.*
>
> Bạn có thể chạy nó trong psql.  
> *You can run it in psql.*
>
> Bạn cũng có thể chạy nó trong bất kỳ công cụ nào khác.  
> *You can also run it in any other tool.*
>
> Ví dụ là DBeaver, TablePlus và pgAdmin.  
> *Examples are DBeaver, TablePlus, and pgAdmin.*

### 1.5. Ví dụ chạy tay — tự tay tính để "thấy" cơ chế
*1.5. A hand-worked example — compute it yourself to "see" the mechanism*

Đây là mục quan trọng nhất của Phần 1.  
*This is the most important section of Part 1.*

Ta sẽ **tạm bỏ qua model AI**.  
*We will **set the AI model aside for now**.*

Ta tự bịa ra embedding chỉ có **2 chiều**.  
*We make up an embedding with only **two dimensions**.*

Nhờ vậy ta tính được bằng tay.  
*This lets us compute by hand.*

Ta sẽ nhìn thấy con số biến đổi.  
*We will watch the numbers change.*

Toán học ở đây y hệt với 1536 chiều.  
*The mathematics here is identical to 1536 dimensions.*

Nó chỉ khác ở chỗ có ít số hạng hơn.  
*It differs only in having fewer terms.*

Vì thế ta viết ra giấy được.  
*We can therefore write it on paper.*

Giả sử ta đặt 3 sản phẩm và 1 câu truy vấn lên bản đồ 2 chiều.  
*Suppose we place three products and one query onto a two-dimensional map.*

| Object / Object | Vector (x, y) | Ý nghĩa của toạ độ / Meaning of the coordinates |
|---|---|---|
| Áo phao (A) | (2, 9) | nằm góc trên bên trái / in the upper left corner |
| Khăn len (B) | (3, 8) | ngay cạnh áo phao / right beside "áo phao" |
| Kem chống nắng (C) | (9, 1) | tít góc dưới bên phải / far in the lower right corner |
| **Query: "giữ ấm mùa đông" (Q)** | **(2, 8)** | rơi vào giữa vùng đồ mùa đông / lands in the middle of the winter clothing region |

Bây giờ ta dùng thước đo khoảng cách quen thuộc nhất.  
*Now we use the most familiar distance measure.*

Thước đo đó là **Euclidean distance**.  
*That measure is **Euclidean distance**.*

Nó còn gọi là **L2 distance**.  
*It is also called **L2 distance**.*

Đây chính là "khoảng cách đường chim bay".  
*This is exactly the "as the crow flies" distance.*

Bạn đã học nó ở phổ thông.  
*You learned it in high school.*

Đây là công thức cho 2 chiều.  
*Here is the formula for two dimensions.*

```
distance = sqrt( (x1 - x2)² + (y1 - y2)² )
```

`sqrt` là viết tắt của *square root*.  
*`sqrt` is short for *square root*.*

Nghĩa tiếng Việt của nó là căn bậc hai.  
*Its Vietnamese meaning is "căn bậc hai".*

Dấu `²` là bình phương.  
*The `²` sign means squaring.*

Bình phương là nhân số đó với chính nó.  
*Squaring means multiplying the number by itself.*

Ta tính khoảng cách từ điểm truy vấn Q tới từng sản phẩm.  
*We compute the distance from query point Q to each product.*

Ta làm từng bước một.  
*We go step by step.*

**Q tới A (áo phao) / Q to A ("áo phao")**

```
hiệu theo trục x:  2 - 2 = 0     →  0² = 0
difference on x:   2 - 2 = 0     →  0² = 0
hiệu theo trục y:  8 - 9 = -1    →  (-1)² = 1
difference on y:   8 - 9 = -1    →  (-1)² = 1
tổng:              0 + 1 = 1
sum:               0 + 1 = 1
căn bậc hai:       sqrt(1) = 1.00
square root:       sqrt(1) = 1.00
```

**Q tới B (khăn len) / Q to B ("khăn len")**

```
hiệu theo trục x:  2 - 3 = -1    →  (-1)² = 1
difference on x:   2 - 3 = -1    →  (-1)² = 1
hiệu theo trục y:  8 - 8 = 0     →  0² = 0
difference on y:   8 - 8 = 0     →  0² = 0
tổng:              1 + 0 = 1
sum:               1 + 0 = 1
căn bậc hai:       sqrt(1) = 1.00
square root:       sqrt(1) = 1.00
```

**Q tới C (kem chống nắng) / Q to C ("kem chống nắng")**

```
hiệu theo trục x:  2 - 9 = -7    →  (-7)² = 49
difference on x:   2 - 9 = -7    →  (-7)² = 49
hiệu theo trục y:  8 - 1 = 7     →  7² = 49
difference on y:   8 - 1 = 7     →  7² = 49
tổng:              49 + 49 = 98
sum:               49 + 49 = 98
căn bậc hai:       sqrt(98) ≈ 9.90
square root:       sqrt(98) ≈ 9.90
```

Ta sắp xếp kết quả theo thứ tự tăng dần.  
*We sort the results in ascending order.*

Điểm gần nhất lên trước.  
*The nearest point comes first.*

| Hạng / Rank | Sản phẩm / Product | Khoảng cách / Distance |
|---|---|---|
| 1 | Áo phao | 1.00 |
| 2 | Khăn len | 1.00 |
| 3 | Kem chống nắng | 9.90 |

Hãy dừng lại một giây.  
*Pause for a second.*

Hãy nhìn kết quả này cho kỹ.  
*Look closely at this result.*

Câu truy vấn "giữ ấm mùa đông" cho ra khoảng cách **1.00** tới áo phao và khăn len.  
*The query "giữ ấm mùa đông" gives a distance of **1.00** to "áo phao" and "khăn len".*

Nó cho ra khoảng cách **9.90** tới kem chống nắng.  
*It gives a distance of **9.90** to "kem chống nắng".*

Khoảng cách đó xa gần **10 lần**.  
*That distance is nearly **ten times** farther.*

Máy tính vừa "hiểu" đúng như trực giác con người.  
*The computer just "understood" it the way human intuition does.*

Nó chỉ làm mấy phép trừ và phép căn.  
*It only performed a few subtractions and a square root.*

Hãy để ý luôn một chi tiết tinh tế.  
*Notice one subtle detail as well.*

Chữ "phao" trong "áo phao" **không hề xuất hiện** trong câu truy vấn.  
*The word "phao" in "áo phao" **never appears** in the query.*

Không có một ký tự nào trùng.  
*Not a single character matches.*

Ta có thể dùng `LIKE` như mục 1.1.  
*We could use `LIKE` as in section 1.1.*

Khi đó kết quả vẫn là 0 dòng.  
*The result would still be zero rows.*

Ở đây ta lại tìm được.  
*Here we find it anyway.*

Lý do là ta so **toạ độ ý nghĩa**.  
*The reason is that we compare **coordinates of meaning**.*

Ta không so ký tự.  
*We do not compare characters.*

**Và đây là điều tôi muốn bạn nhớ nhất / And here is what I most want you to remember**

Những gì bạn vừa làm bằng tay ở trên **chính xác** là những gì pgvector làm bên trong.  
*What you just did by hand is **exactly** what pgvector does internally.*

Nó làm điều đó khi bạn gõ `ORDER BY embedding <-> query_vector`.  
*It does that when you type `ORDER BY embedding <-> query_vector`.*

Không có phép màu nào cả.  
*There is no magic at all.*

Nó chỉ làm với 1536 số hạng thay vì 2.  
*It merely works with 1536 terms instead of two.*

Nó làm cho hàng triệu dòng thay vì 3 dòng.  
*It works over millions of rows instead of three.*

Toán không đổi.  
*The mathematics does not change.*

### 1.6. Code "hello world" — câu vector search đầu tiên của bạn
*1.6. The "hello world" code — your first vector search statement*

Bây giờ ta viết lại đúng ví dụ trên bằng SQL thật.  
*Now we rewrite that same example in real SQL.*

Đoạn này chạy được trong psql sau khi đã cài pgvector.  
*This snippet runs in psql once pgvector is installed.*

Cách cài dễ nhất là dùng Docker.  
*The easiest installation route is Docker.*

Bạn xem ghi chú cuối file để biết chi tiết.  
*See the notes at the end of the file for details.*

Tôi dùng vector 3 chiều thay vì 2 chiều.  
*I use three-dimensional vectors instead of two.*

Tôi thêm số 0 vào cuối mỗi vector.  
*I append a zero to the end of each vector.*

Mục đích là để bạn quen mắt với việc vector có thể dài tuỳ ý.  
*The purpose is to get your eye used to vectors of any length.*

Kết quả không đổi.  
*The result does not change.*

```sql
-- BƯỚC 1: Bật extension pgvector.
-- STEP 1: Enable the pgvector extension.
-- Chỉ cần chạy MỘT LẦN cho mỗi database. Sau lệnh này Postgres mới "biết"
-- kiểu dữ liệu vector và các toán tử như <-> tồn tại.
-- Run it ONCE per database. After this command Postgres knows that
-- the vector data type and operators such as <-> exist.
-- "IF NOT EXISTS" nghĩa là: nếu đã bật rồi thì bỏ qua, đừng báo lỗi.
-- "IF NOT EXISTS" means: skip it when already enabled, and raise no error.
CREATE EXTENSION IF NOT EXISTS vector;

-- BƯỚC 2: Tạo bảng.
-- STEP 2: Create the table.
CREATE TABLE items (
    id        bigserial PRIMARY KEY,  -- khoá chính, tự động tăng 1,2,3... mỗi khi thêm dòng
                                      -- primary key, auto-increments 1,2,3... on every insert
                                      -- (PRIMARY KEY = cột định danh duy nhất cho mỗi dòng)
                                      -- (PRIMARY KEY = the unique identifier column per row)
    name      text,                   -- cột chữ bình thường, lưu tên sản phẩm
                                      -- an ordinary text column holding the product name
    embedding vector(3)               -- ĐÂY LÀ CỘT MỚI: kiểu vector, mỗi ô chứa 3 con số.
                                      -- THIS IS THE NEW COLUMN: vector type, 3 numbers per cell.
                                      -- Con số 3 phải khớp đúng số chiều của embedding bạn dùng.
                                      -- The number 3 must match your embedding's dimension count.
);

-- BƯỚC 3: Chèn dữ liệu vào bảng.
-- STEP 3: Insert data into the table.
-- Vector được viết như một chuỗi, đặt trong nháy đơn, dạng '[a,b,c]'.
-- A vector is written as a single-quoted string in the form '[a,b,c]'.
INSERT INTO items (name, embedding) VALUES
    ('áo phao',        '[2, 9, 0]'),
    ('khăn len',       '[3, 8, 0]'),
    ('kem chống nắng', '[9, 1, 0]');

-- BƯỚC 4: Similarity search — phần chính!
-- STEP 4: Similarity search — the main event!
SELECT name,
       embedding <-> '[2, 8, 0]' AS distance   -- <-> là toán tử tính L2 distance.
                                               -- <-> is the operator for L2 distance.
                                               -- AS distance = đặt tên cho cột kết quả.
                                               -- AS distance = names the result column.
FROM items
ORDER BY embedding <-> '[2, 8, 0]'             -- sắp xếp tăng dần: gần nhất lên đầu
                                               -- sort ascending: nearest first
LIMIT 2;                                       -- chỉ lấy 2 dòng đầu = "2 hàng xóm gần nhất"
                                               -- take only 2 rows = "the 2 nearest neighbors"
```

Đây là kết quả chạy ra.  
*Here is the output.*

```
   name    | distance
-----------+----------
 áo phao   |        1
 khăn len  |        1
```

Con số này giống hệt con số bạn vừa tính tay ở mục 1.5.  
*These numbers match exactly what you computed by hand in section 1.5.*

Không sai một chữ số.  
*Not one digit is off.*

**Điểm mấu chốt cần khắc cốt / The crucial point to burn in**

Ba thành phần `ORDER BY cot_vector <-> query_vector` cộng với `LIMIT k` gộp lại thành một ý.  
*The parts `ORDER BY cot_vector <-> query_vector` plus `LIMIT k` combine into one idea.*

Ý đó là **"cho tôi k bản ghi giống nhất"**.  
*That idea is **"give me the k most similar records"**.*

Đó là toàn bộ vector search ở dạng đơn giản nhất.  
*That is all of vector search in its simplest form.*

Bạn vừa viết được câu vector search đầu tiên của mình.  
*You just wrote your first vector search statement.*

### 1.7. Ba toán tử khoảng cách — biết mặt trước, đào sâu sau
*1.7. The three distance operators — recognize them first, dig in later*

pgvector cho bạn ba toán tử.  
*pgvector gives you three operators.*

Ở tầng Basic bạn chỉ cần **nhận mặt** chúng.  
*At the Basic tier you only need to **recognize** them.*

Phần 2 sẽ giải thích cặn kẽ khi nào dùng cái nào.  
*Part 2 explains thoroughly when to use which.*

| Toán tử / Operator | Tên đầy đủ / Full name | Dùng khi nào / When to use |
|---|---|---|
| `<->` | L2 / Euclidean distance | mặc định, an toàn, dễ hình dung / default, safe, easy to picture |
| `<=>` | Cosine distance | **phổ biến nhất với embedding của văn bản** / **most common with text embeddings** |
| `<#>` | Negative inner product | khi vector đã được chuẩn hoá, cần tốc độ tối đa / for normalized vectors when you need top speed |

Bây giờ bạn có thể thấy "cosine" hay "inner product" còn xa lạ.  
*Right now "cosine" or "inner product" may still feel foreign.*

Điều đó không sao.  
*That is fine.*

Mục 2.1 sẽ dựng lại chúng từ đầu.  
*Section 2.1 rebuilds them from scratch.*

### 1.8. 🧩 [Ngoài bài gốc] — Hai điều một Staff Engineer nhắc bạn ngay từ ngày đầu
*1.8. 🧩 [Ngoài bài gốc] — Two things a Staff Engineer tells you on day one*

**Điều 1: pgvector KHÔNG sinh ra embedding**  
*Point 1: pgvector does NOT generate embeddings*

Tôi đã nói điều này ở mục 1.3.  
*I said this in section 1.3.*

Tôi cố ý nhắc lại.  
*I repeat it on purpose.*

Đây là hiểu nhầm phổ biến nhất của người mới.  
*This is the most common misunderstanding among beginners.*

Kiến trúc thực tế luôn có **hai mảnh**.  
*The real architecture always has **two pieces**.*

```
[Đoạn chữ]  →  Embedding model (OpenAI / sentence-transformers / Cohere)  →  [Dãy số]
[Text]      →  Embedding model (OpenAI / sentence-transformers / Cohere)  →  [Numbers]
                                                                                 ↓
                                                     pgvector chỉ nhận dãy số ở đây,
                                                     lưu nó, rồi tìm hàng xóm gần nhất.
                                                     pgvector only receives the numbers here,
                                                     stores them, then finds nearest neighbors.
```

Đây là hệ quả thực tế.  
*Here is the practical consequence.*

Kết quả tìm kiếm của bạn có thể *dở về mặt ngữ nghĩa*.  
*Your search results may be *semantically poor*.*

Ví dụ là tìm "áo mùa đông" lại ra kem chống nắng.  
*An example is searching "áo mùa đông" and getting "kem chống nắng".*

Thủ phạm thường là **embedding model**.  
*The culprit is usually the **embedding model**.*

Thủ phạm không phải pgvector.  
*The culprit is not pgvector.*

Đổi index sẽ không cứu được gì.  
*Changing the index rescues nothing.*

Biết chỉ đúng thủ phạm là một kỹ năng debug quan trọng.  
*Pointing at the right culprit is an important debugging skill.*

**Điều 2: Mặc định pgvector làm exact search / Point 2: pgvector does exact search by default**

Bạn **KHÔNG** cần index ngay.  
*You do **NOT** need an index right away.*

Có hai từ mới ở đây.  
*Two new terms appear here.*

> **Exact search — tìm kiếm chính xác**
>
> Exact search quét qua *toàn bộ* các dòng.  
> *Exact search scans *all* the rows.*
>
> Nó tính khoảng cách với từng dòng.  
> *It computes the distance to every row.*
>
> Nó lấy k dòng nhỏ nhất.  
> *It takes the k smallest.*
>
> Kết quả **luôn đúng tuyệt đối**.  
> *The result is **always perfectly correct**.*
>
> Cách này còn gọi là **brute force** — vét cạn.  
> *This approach is also called **brute force**.*

> **Sequential scan — quét tuần tự**
>
> Đây là thuật ngữ của Postgres.  
> *This is Postgres terminology.*
>
> Nó chỉ việc đọc lần lượt từ dòng đầu đến dòng cuối.  
> *It names the act of reading from the first row to the last.*
>
> Đây chính là cách Postgres thực hiện exact search khi chưa có index.  
> *This is exactly how Postgres performs an exact search without an index.*

Bảng của bạn có thể còn nhỏ.  
*Your table may still be small.*

Nhỏ ở đây nghĩa là khoảng dưới 10.000 tới 50.000 dòng.  
*Small here means roughly under 10,000 to 50,000 rows.*

Với bảng đó, quét tuần tự vẫn nhanh trong tầm mili-giây.  
*For such a table, a sequential scan still runs in milliseconds.*

Nó cho kết quả *chính xác 100%*.  
*It gives *100% accurate* results.*

Index của Phần 2 chỉ cần khi dữ liệu lớn.  
*The index from Part 2 is needed only when the data grows large.*

Khi đó bạn sẽ phải **đánh đổi độ chính xác để lấy tốc độ**.  
*At that point you must **trade accuracy for speed**.*

Rất nhiều người vội đánh index từ ngày đầu.  
*Many people rush to build an index on day one.*

Rồi họ hoảng lên vì một câu hỏi.  
*Then they panic over one question.*

"Sao kết quả sai so với lúc test?"  
*"Why do the results differ from my test run?"*

Kết quả không sai.  
*The results are not wrong.*

Nó *xấp xỉ*.  
*They are *approximate*.*

Đó là do bạn tự chọn.  
*That was your own choice.*

Chi tiết nằm ở mục 2.6.  
*The details sit in section 2.6.*

### ✅ Self-check Phần 1
*✅ Self-check for Part 1*

Bạn hãy trả lời trong đầu trước.  
*Answer in your head first.*

Sau đó bạn mở gợi ý ra so.  
*Then open the hints and compare.*

**Câu 1.** Vì sao `WHERE name LIKE '%đồ giữ ấm mùa đông%'` không giải quyết được bài toán của ta?  
***Question 1.** Why does `WHERE name LIKE '%đồ giữ ấm mùa đông%'` fail to solve our problem?*

> *Gợi ý đáp án / Suggested answer*
>
> `LIKE` so khớp **ký tự**.  
> *`LIKE` matches **characters**.*
>
> Bài toán đòi hỏi so khớp **ý nghĩa**.  
> *The problem demands matching by **meaning**.*
>
> Không sản phẩm nào chứa đúng cụm ký tự đó.  
> *No product contains that exact character sequence.*
>
> Vậy nên câu lệnh trả về 0 dòng.  
> *The statement therefore returns zero rows.*
>
> Về nghĩa thì "áo khoác lông vũ" rất liên quan.  
> *In meaning, "áo khoác lông vũ" is highly relevant.*

**Câu 2.** Trong câu `ORDER BY embedding <-> '[2,8,0]' LIMIT 3`, con số khoảng cách càng nhỏ thì nghĩa là gì?  
***Question 2.** In `ORDER BY embedding <-> '[2,8,0]' LIMIT 3`, what does a smaller distance mean?*

> *Gợi ý đáp án / Suggested answer*
>
> Nó nghĩa là càng **giống nhau về ý nghĩa**.  
> *It means greater **similarity in meaning**.*
>
> Khoảng cách nhỏ nghĩa là hai điểm nằm gần nhau trên bản đồ ý nghĩa.  
> *A small distance means the two points sit close on the map of meaning.*
>
> Vì thế ta luôn sắp xếp tăng dần.  
> *We therefore always sort ascending.*
>
> Ta lấy các dòng đầu.  
> *We take the first rows.*

**Câu 3.** pgvector có tự biến chữ "áo phao" thành `[2,9,0]` không? Ai làm việc đó?  
***Question 3.** Does pgvector turn "áo phao" into `[2,9,0]` itself? Who does that job?*

> *Gợi ý đáp án / Suggested answer*
>
> pgvector không làm việc đó.  
> *pgvector does not do that job.*
>
> Việc đó do **embedding model** làm.  
> *The **embedding model** does that job.*
>
> Embedding model là một mô hình AI riêng.  
> *An embedding model is a separate AI model.*
>
> Ví dụ là OpenAI API hoặc sentence-transformers.  
> *Examples are the OpenAI API or sentence-transformers.*
>
> pgvector chỉ lưu dãy số.  
> *pgvector only stores the number sequences.*
>
> pgvector chỉ tìm hàng xóm gần nhất.  
> *pgvector only finds the nearest neighbors.*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

> Phần này giả định bạn đã nắm bốn thứ.  
> *This part assumes you already grasp four things.*
>
> Thứ nhất là embedding là gì.  
> *The first is what an embedding is.*
>
> Thứ hai là khoảng cách nghĩa là gì.  
> *The second is what distance means.*
>
> Thứ ba là cột kiểu `vector`.  
> *The third is the `vector` column type.*
>
> Thứ tư là toán tử `<->`.  
> *The fourth is the `<->` operator.*
>
> Bạn có thể còn lấn cấn chỗ nào đó.  
> *You may still feel shaky somewhere.*
>
> Hãy quay lại mục 1.5 đọc lại ví dụ chạy tay.  
> *Go back to section 1.5 and reread the hand-worked example.*
>
> Mục đó là nền của mọi thứ dưới đây.  
> *That section is the foundation of everything below.*

### 2.1. Ba thước đo khoảng cách — bên trong chúng thực sự làm gì
*2.1. The three distance measures — what they actually do inside*

Đây là chỗ nhiều người chọn bừa.  
*This is where many people choose carelessly.*

Họ làm hỏng chất lượng tìm kiếm.  
*They ruin their search quality.*

Họ không hiểu nguyên nhân.  
*They do not understand the cause.*

Ta đi từng cái một.  
*We take them one at a time.*

Ta có ví dụ số cụ thể cho từng cái.  
*We have a concrete numeric example for each.*

**Quy ước ký hiệu / Notation convention**

Ta có hai vector **a** và **b**.  
*We have two vectors, **a** and **b**.*

Ký hiệu `aᵢ` nghĩa là "phần tử thứ i của vector a".  
*The symbol `aᵢ` means "the i-th element of vector a".*

Ký hiệu `Σ` đọc là *sigma*.  
*The symbol `Σ` is read as *sigma*.*

Ký hiệu đó nghĩa là "cộng dồn tất cả lại".  
*That symbol means "sum them all up".*

#### (1) L2 / Euclidean distance — toán tử `<->`
*(1) L2 / Euclidean distance — the `<->` operator*

```
L2(a, b) = sqrt( Σ (aᵢ - bᵢ)² )
```

Đây chính là công thức bạn đã tính tay ở mục 1.5.  
*This is exactly the formula you computed by hand in section 1.5.*

Nó chỉ được viết ở dạng tổng quát cho d chiều.  
*It is merely written in general form for d dimensions.*

Nó đo **khoảng cách đường chim bay** giữa hai điểm.  
*It measures the **straight-line distance** between two points.*

Đây là đặc điểm cần nhớ.  
*Here is the characteristic to remember.*

L2 **nhạy với cả hướng lẫn độ dài** của vector.  
*L2 is **sensitive to both direction and length**.*

Hai vector có thể chỉ về cùng một hướng.  
*Two vectors may point in the same direction.*

Một cái có thể dài gấp đôi cái kia.  
*One may be twice as long as the other.*

L2 vẫn coi chúng khá xa nhau.  
*L2 still treats them as fairly far apart.*

#### (2) Cosine distance — toán tử `<=>`
*(2) Cosine distance — the `<=>` operator*

Trước hết phải giải thích hai thứ mà công thức cosine dùng tới.  
*First we must explain two things the cosine formula relies on.*

> **Dot product — tích vô hướng**
>
> Dot product còn gọi là *inner product*.  
> *A dot product is also called an *inner product*.*
>
> Ký hiệu của nó là `a · b`.  
> *Its symbol is `a · b`.*
>
> Cách tính cực kỳ đơn giản.  
> *The computation is extremely simple.*
>
> Bạn nhân từng cặp phần tử tương ứng.  
> *You multiply each corresponding pair of elements.*
>
> Sau đó bạn cộng tất cả lại.  
> *Then you add them all together.*

```
a · b = Σ aᵢ · bᵢ
```

> Đây là ví dụ chạy tay.  
> *Here is a hand-worked example.*
>
> Cho a = [2, 9] và b = [3, 8].  
> *Let a = [2, 9] and b = [3, 8].*
>
> Khi đó `a · b = 2×3 + 9×8 = 6 + 72 = 78`.  
> *Then `a · b = 2×3 + 9×8 = 6 + 72 = 78`.*

> **Norm / magnitude — độ dài của vector**
>
> Ký hiệu của nó là `|a|`.  
> *Its symbol is `|a|`.*
>
> Đây chính là khoảng cách từ gốc toạ độ tới điểm đó.  
> *This is the distance from the origin to that point.*
>
> Ta tính nó bằng căn bậc hai của tổng bình phương.  
> *We compute it as the square root of the sum of squares.*

```
|a| = sqrt( Σ aᵢ² )
```

> Đây là ví dụ chạy tay.  
> *Here is a hand-worked example.*
>
> Cho a = [2, 9].  
> *Let a = [2, 9].*
>
> Khi đó `|a| = sqrt(4 + 81) = sqrt(85) ≈ 9.22`.  
> *Then `|a| = sqrt(4 + 81) = sqrt(85) ≈ 9.22`.*

Giờ mới tới cosine.  
*Now we reach cosine.*

Ý tưởng của nó nằm trong một câu.  
*Its idea fits in one sentence.*

**Nó chỉ quan tâm hai vector có chỉ về cùng hướng hay không.**  
***It only cares whether the two vectors point the same way.***

Nó kệ chúng dài ngắn thế nào.  
*It ignores how long or short they are.*

```
cosine_similarity(a, b) = (a · b) / (|a| · |b|)     # nằm trong khoảng [-1, 1]; 1 = cùng hướng hoàn toàn
                                                    # ranges over [-1, 1]; 1 = exactly the same direction
cosine_distance(a, b)   = 1 - cosine_similarity     # nằm trong [0, 2]; 0 = giống hệt
                                                    # ranges over [0, 2]; 0 = identical
```

pgvector trả về **cosine distance**.  
*pgvector returns the **cosine distance**.*

Đó là dòng thứ hai ở trên.  
*That is the second line above.*

Cách này giữ đúng quy ước "nhỏ hơn nghĩa là gần hơn".  
*This keeps the convention that smaller means closer.*

Nhờ đó `ORDER BY` tăng dần vẫn cho ra kết quả giống nhất trước.  
*An ascending `ORDER BY` therefore still puts the most similar first.*

> Đây là ví dụ chạy tay đầy đủ với a = [2, 9] và b = [3, 8].  
> *Here is a full hand-worked example with a = [2, 9] and b = [3, 8].*
>
> ```
> a · b            = 2×3 + 9×8 = 78
> |a|              = sqrt(2² + 9²) = sqrt(85)  ≈ 9.220
> |b|              = sqrt(3² + 8²) = sqrt(73)  ≈ 8.544
> cosine_similarity= 78 / (9.220 × 8.544) = 78 / 78.78 ≈ 0.990
> cosine_distance  = 1 - 0.990 = 0.010     → rất gần nhau
> cosine_distance  = 1 - 0.990 = 0.010     → very close together
> ```

**Vì sao cosine là lựa chọn mặc định cho văn bản?**  
***Why is cosine the default choice for text?***

Với text, cái ta quan tâm là *hướng ngữ nghĩa*.  
*For text, what we care about is the *semantic direction*.*

Ta không quan tâm vector dài hay ngắn.  
*We do not care whether the vector is long or short.*

Hãy xét một đoạn văn 500 chữ và một câu 5 chữ.  
*Consider a 500-word passage and a 5-word sentence.*

Hai thứ đó nói về cùng một chủ đề.  
*The two speak about the same topic.*

Chúng thường cho ra vector cùng hướng.  
*They usually produce vectors with the same direction.*

Độ dài của hai vector lại khác nhau.  
*The lengths of the two vectors differ.*

Cosine coi chúng là giống nhau.  
*Cosine treats them as similar.*

Đó đúng ý ta muốn.  
*That matches our intent.*

L2 lại đẩy chúng ra xa.  
*L2 pushes them apart instead.*

Đó không đúng ý ta muốn.  
*That does not match our intent.*

#### (3) Negative inner product — toán tử `<#>`
*(3) Negative inner product — the `<#>` operator*

Đây chỉ là dot product ở trên.  
*This is just the dot product from above.*

pgvector trả về **giá trị âm** của nó.  
*pgvector returns its **negated value**.*

```
<#>  =  -(a · b)
```

Vì sao phải đảo dấu?  
*Why flip the sign?*

Dot product càng **lớn** thì hai vector càng giống nhau.  
*A **larger** dot product means more similar vectors.*

Điều đó ngược với quy ước "nhỏ nghĩa là gần" của `ORDER BY`.  
*That contradicts the "smaller means closer" convention of `ORDER BY`.*

Đảo dấu xong thì mọi thứ nhất quán trở lại.  
*After the flip, everything is consistent again.*

**Mẹo tối ưu quan trọng / An important optimization tip**

Bạn có thể **normalize** — chuẩn hoá — mọi vector.  
*You can **normalize** every vector.*

Normalize nghĩa là chia mỗi vector cho độ dài của chính nó.  
*Normalizing means dividing each vector by its own length.*

Độ dài mới của nó bằng đúng 1.  
*Its new length is exactly 1.*

Khi đó mẫu số `|a| · |b|` trong công thức cosine trở thành `1 × 1 = 1`.  
*The denominator `|a| · |b|` in the cosine formula then becomes `1 × 1 = 1`.*

Lúc đó **inner product tương đương hoàn toàn với cosine**.  
*At that point **inner product is completely equivalent to cosine**.*

Nó lại tính nhanh hơn.  
*It also computes faster.*

Lý do là nó bỏ được phép chia và hai phép căn.  
*The reason is the removal of one division and two square roots.*

Đây là mẹo hay dùng ở production.  
*This trick is common in production.*

Production là môi trường chạy thật, phục vụ người dùng thật.  
*Production is the live environment serving real users.*

#### Quy tắc chọn nhanh
*Quick selection rule*

| Tình huống / Situation | Chọn / Pick |
|---|---|
| Embedding văn bản từ OpenAI / sentence-transformers / Cohere — text embeddings from OpenAI, sentence-transformers, or Cohere | `<=>` cosine |
| Bạn đã tự normalize vector về độ dài 1 / You already normalized vectors to length 1 | `<#>` inner product (nhanh hơn, kết quả như cosine) / `<#>` inner product (faster, same result as cosine) |
| Độ lớn của vector thực sự mang ý nghĩa / The vector magnitude genuinely carries meaning | `<->` L2 |

Trường hợp thứ ba hiếm gặp với text.  
*The third case is rare with text.*

Nó hay gặp hơn ở dữ liệu số học hoặc ảnh đặc thù.  
*It appears more often with numeric data or specialized image data.*

Bạn có thể phân vân.  
*You may hesitate.*

Khi đó hãy chọn **cosine**.  
*In that case pick **cosine**.*

Đó là câu trả lời đúng trong khoảng 90% trường hợp thực tế với văn bản.  
*That is the right answer in about 90% of real text cases.*

### 2.2. Data modeling: thiết kế bảng thật (mục tiêu #1 của bài gốc)
*2.2. Data modeling: designing a real table (objective #1 of the original lesson)*

**Data modeling** nghĩa là mô hình hoá dữ liệu.  
***Data modeling** means shaping your data model.*

Bạn quyết định dữ liệu nên được tổ chức thành bảng nào.  
*You decide which tables your data belongs in.*

Bạn quyết định bảng đó có cột nào.  
*You decide which columns that table has.*

Bạn quyết định mỗi cột thuộc kiểu gì.  
*You decide the type of each column.*

Trong thực tế bạn **không bao giờ** lưu mỗi cột vector trơ trọi.  
*In practice you **never** store a lone vector column.*

Bạn lưu vector **nằm ngay cạnh dữ liệu nghiệp vụ**.  
*You store the vector **right beside the business data**.*

Đây chính là siêu năng lực của pgvector.  
*This is exactly pgvector's superpower.*

Ta sẽ thấy rõ điều đó ở mục 2.5.  
*We will see it clearly in section 2.5.*

```sql
CREATE TABLE documents (
    id          bigserial   PRIMARY KEY,
    tenant_id   bigint      NOT NULL,       -- xem giải thích "multi-tenant" bên dưới.
                                            -- see the "multi-tenant" explanation below.
                                            -- NOT NULL = cột này bắt buộc có giá trị.
                                            -- NOT NULL = this column requires a value.
    title       text        NOT NULL,
    content     text        NOT NULL,       -- nội dung gốc, để hiển thị lại cho người dùng
                                            -- the original content, shown back to the user
    category    text,                       -- metadata: dùng để lọc kết quả
                                            -- metadata: used to filter results
    created_at  timestamptz NOT NULL DEFAULT now(),
                                            -- timestamptz = thời điểm có kèm múi giờ.
                                            -- timestamptz = a timestamp with a time zone.
                                            -- DEFAULT now() = tự điền giờ hiện tại khi thêm dòng.
                                            -- DEFAULT now() = fills in the current time on insert.
    embedding   vector(1536)                -- SỐ 1536 PHẢI KHỚP số chiều của model bạn dùng!
                                            -- THE NUMBER 1536 MUST MATCH your model's dimensions!
);

-- Đánh index thông thường trên các cột metadata hay dùng để lọc.
-- Build ordinary indexes on the metadata columns you filter by often.
-- Việc này RẤT quan trọng ở quy mô lớn, lý do xem mục 2.6 và 4.1.
-- This matters GREATLY at scale; see sections 2.6 and 4.1 for the reason.
CREATE INDEX ON documents (tenant_id, category);
CREATE INDEX ON documents (created_at);
```

Có ba thuật ngữ mới trong đoạn trên.  
*Three new terms appear in the snippet above.*

> **Metadata — siêu dữ liệu**
>
> Metadata là dữ liệu *mô tả về* dữ liệu chính.  
> *Metadata is data that *describes* the main data.*
>
> Ở đây `category`, `created_at` và `tenant_id` là metadata mô tả tài liệu.  
> *Here `category`, `created_at`, and `tenant_id` are metadata describing the document.*
>
> Cột `content` mới là dữ liệu chính.  
> *The `content` column is the main data.*
>
> Ta dùng metadata để lọc.  
> *We use metadata for filtering.*

> **Multi-tenant — đa khách thuê**
>
> Đây là kiến trúc mà **một** hệ thống phục vụ **nhiều** khách hàng khác nhau.  
> *This is an architecture where **one** system serves **many** different customers.*
>
> Dữ liệu của họ nằm chung một bảng.  
> *Their data lives in one shared table.*
>
> Dữ liệu đó phải được cách ly tuyệt đối.  
> *That data must be perfectly isolated.*
>
> Cột `tenant_id` là "biển số" xác định dòng này thuộc về khách nào.  
> *The `tenant_id` column is the "license plate" saying which customer owns the row.*
>
> Mọi câu query bắt buộc phải lọc theo nó.  
> *Every query must filter on it.*
>
> Bạn quên lọc theo nó.  
> *Suppose you forget that filter.*
>
> Khi đó bạn sẽ để lộ dữ liệu khách A cho khách B.  
> *You then leak customer A's data to customer B.*
>
> Đây là sự cố nghiêm trọng nhất trong loại kiến trúc này.  
> *This is the gravest incident in this kind of architecture.*

> **Index — chỉ mục**
>
> Ta sẽ giải thích đầy đủ ở mục 2.3 ngay dưới.  
> *We explain it fully in section 2.3 just below.*
>
> Tạm hiểu index là một cấu trúc phụ.  
> *For now, treat an index as an auxiliary structure.*
>
> Cấu trúc đó giúp Postgres tìm nhanh mà không phải đọc hết bảng.  
> *That structure helps Postgres search fast without reading the whole table.*

**Điểm mà một staff engineer sẽ nhấn mạnh ngay ở bước này / What a staff engineer stresses at this step**

Con số `1536` **phải khớp chính xác** số chiều mà embedding model của bạn sinh ra.  
*The number `1536` **must match exactly** the dimension count your embedding model produces.*

Bạn nhét vector 768 chiều vào cột `vector(1536)`.  
*Suppose you push a 768-dimension vector into a `vector(1536)` column.*

Postgres báo lỗi ngay và từ chối.  
*Postgres raises an error immediately and refuses.*

Nghe thì có vẻ là chi tiết vặt.  
*This sounds like a trivial detail.*

Nó dẫn tới một hệ quả kiến trúc rất nặng.  
*It leads to a very heavy architectural consequence.*

**Đổi embedding model đồng nghĩa với phải sinh lại toàn bộ embedding cho toàn bộ dữ liệu.**  
***Changing the embedding model means regenerating every embedding for all your data.***

Ta sẽ quay lại chuyện này ở Phần 4.  
*We will return to this in Part 4.*

Đây là một trong những quyết định đắt đỏ nhất của cả hệ thống.  
*This is one of the most expensive decisions in the whole system.*

### 2.3. Đánh index HNSW — làm cho nó nhanh
*2.3. Building an HNSW index — making it fast*

#### Trước hết: index là cái gì?
*First: what is an index?*

**Index** là một cấu trúc dữ liệu phụ.  
*An **index** is an auxiliary data structure.*

Database dựng nó thêm bên cạnh bảng.  
*The database builds it alongside the table.*

Mục đích của nó là tìm nhanh hơn.  
*Its purpose is faster search.*

> Đây là một analogy.  
> *Here is an analogy.*
>
> Hãy nghĩ tới **mục lục tra cứu ở cuối một cuốn sách dày**.  
> *Think of the **index at the back of a thick book**.*
>
> Cuốn sách không có mục lục.  
> *Suppose the book has no index.*
>
> Muốn tìm chữ "photosynthesis" bạn phải lật từng trang từ đầu đến cuối.  
> *To find "photosynthesis" you must flip every page from front to back.*
>
> Đó chính là *sequential scan*.  
> *That is exactly a *sequential scan*.*
>
> Cuốn sách có mục lục.  
> *Now suppose the book has an index.*
>
> Bạn tra chữ P và thấy ghi "trang 412".  
> *You look up P and see "page 412".*
>
> Bạn mở thẳng tới đó.  
> *You open straight to it.*
>
> Đây là chỗ khớp với kỹ thuật.  
> *Here is where the analogy meets the technology.*
>
> Mục lục **tốn thêm giấy**.  
> *An index **costs extra paper**.*
>
> Index tốn thêm dung lượng đĩa và RAM.  
> *An index costs extra disk space and RAM.*
>
> Mỗi khi sách sửa nội dung thì **mục lục cũng phải sửa theo**.  
> *Whenever the book's content changes, **the index must change too**.*
>
> Mỗi lần INSERT hoặc UPDATE, Postgres phải cập nhật index.  
> *On every INSERT or UPDATE, Postgres must update the index.*
>
> Việc ghi dữ liệu vì thế chậm đi một chút.  
> *Writes therefore get a little slower.*
>
> Đó là cái giá bạn trả để đọc nhanh.  
> *That is the price you pay for fast reads.*

Loại index quen thuộc nhất trong Postgres là **B-tree** — cây cân bằng.  
*The most familiar index type in Postgres is the **B-tree**.*

B-tree dùng cho số và chữ.  
*A B-tree works for numbers and text.*

Đó chính là loại được tạo ra ở hai dòng `CREATE INDEX ON documents (...)` phía trên.  
*That is exactly the type created by the two `CREATE INDEX ON documents (...)` lines above.*

B-tree **không dùng được cho vector**.  
*A B-tree **cannot be used for vectors**.*

Nó cần một thứ tự sắp xếp rõ ràng.  
*It needs a clear sort order.*

Thứ tự đó là "nhỏ hơn" và "lớn hơn".  
*That order is "less than" and "greater than".*

Với điểm trong không gian 1536 chiều, khái niệm "nhỏ hơn" không tồn tại.  
*For a point in 1536-dimensional space, "less than" does not exist.*

Đó là lý do ta cần một họ index hoàn toàn khác.  
*That is why we need an entirely different index family.*

#### ANN và recall — hai từ phải hiểu trước khi đánh index
*ANN and recall — two words to understand before indexing*

> **ANN — hàng xóm gần nhất xấp xỉ**
>
> ANN viết tắt của *Approximate Nearest Neighbor*.  
> *ANN stands for *Approximate Nearest Neighbor*.*
>
> Đây là nhóm thuật toán chấp nhận **có thể bỏ sót** vài hàng xóm thật.  
> *This family of algorithms accepts **possibly missing** a few true neighbors.*
>
> Đổi lại, chúng nhanh hơn hàng nghìn lần.  
> *In exchange, they run thousands of times faster.*
>
> Ngược với ANN là *exact*.  
> *The opposite of ANN is *exact*.*
>
> Exact chính xác tuyệt đối.  
> *Exact is perfectly accurate.*
>
> Exact lại chậm.  
> *Exact is slow.*

> **Recall — độ bao phủ, tỷ lệ tìm được**
>
> Recall là thước đo chất lượng của ANN.  
> *Recall measures the quality of an ANN algorithm.*
>
> Định nghĩa nằm trong một câu.  
> *The definition fits in one sentence.*
>
> Trong `k` hàng xóm thật sự gần nhất, thuật toán tìm được bao nhiêu phần trăm?  
> *Out of the `k` truly nearest neighbors, what percentage does the algorithm find?*
>
> > Đây là ví dụ cụ thể.  
> > *Here is a concrete example.*
> >
> > Bạn hỏi 10 kết quả gần nhất.  
> > *You ask for the 10 nearest results.*
> >
> > Thuật toán tìm đúng 9 trên 10 so với đáp án chính xác.  
> > *The algorithm gets 9 out of 10 right against the exact answer.*
> >
> > Nó bỏ sót 1 cái.  
> > *It misses one.*
> >
> > Nó thay cái đó bằng một kết quả xa hơn.  
> > *It substitutes a farther result for that one.*
> >
> > Kết quả là **recall@10 = 90%**.  
> > *The result is **recall@10 = 90%**.*

Nói cách khác, index ANN không làm search "nhanh hơn miễn phí".  
*In other words, an ANN index does not make search "faster for free".*

**Nó bán tốc độ cho bạn bằng cách lấy đi một chút độ chính xác.**  
***It sells you speed by taking away a little accuracy.***

Hiểu điều này là ranh giới giữa người dùng thư viện và người thiết kế hệ thống.  
*Understanding this separates a library user from a system designer.*

#### Tạo index HNSW
*Creating an HNSW index*

**HNSW** viết tắt của *Hierarchical Navigable Small World*.  
***HNSW** stands for *Hierarchical Navigable Small World*.*

Tên này tạm dịch là "mạng lưới thế giới nhỏ có phân tầng, dẫn đường được".  
*The Vietnamese gloss is "mạng lưới thế giới nhỏ có phân tầng, dẫn đường được".*

Đây là loại index ANN được khuyên dùng mặc định năm 2026.  
*This is the recommended default ANN index type in 2026.*

Cơ chế bên trong của nó nằm ở mục 3.3.  
*Its internal mechanism sits in section 3.3.*

Ở đây ta học cách dùng trước.  
*Here we learn how to use it first.*

```sql
-- Cú pháp: chọn "ops class" KHỚP với toán tử khoảng cách mà bạn sẽ query.
-- Syntax: pick the "ops class" that MATCHES the distance operator you will query with.
--   vector_cosine_ops  ->  dùng với <=>
--   vector_cosine_ops  ->  use with <=>
--   vector_l2_ops      ->  dùng với <->
--   vector_l2_ops      ->  use with <->
--   vector_ip_ops      ->  dùng với <#>
--   vector_ip_ops      ->  use with <#>
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**Ops class** viết đầy đủ là *operator class*, tức lớp toán tử.  
***Ops class** spells out as *operator class*.*

Đây là cách bạn nói cho Postgres biết index này được xây theo thước đo khoảng cách nào.  
*This is how you tell Postgres which distance measure the index was built for.*

Nó phải khớp với toán tử bạn dùng lúc query.  
*It must match the operator you use at query time.*

Bạn để nó không khớp.  
*Suppose you leave it mismatched.*

Khi đó index sẽ **bị bỏ qua hoàn toàn**.  
*The index is then **ignored completely**.*

Đây là một lỗi im lặng rất khó phát hiện.  
*This is a silent failure and very hard to spot.*

Bạn xem mục 2.6 để biết chi tiết.  
*See section 2.6 for the details.*

Có hai tham số lúc build.  
*There are two build-time parameters.*

> **`m`**
>
> `m` là số kết nối tối đa mà mỗi điểm được nối tới các điểm khác trong mạng lưới.  
> *`m` is the maximum number of links each point gets to other points in the network.*
>
> Giá trị mặc định là 16.  
> *The default value is 16.*
>
> Giá trị cao hơn cho recall tốt hơn.  
> *A higher value gives better recall.*
>
> Giá trị cao hơn làm index to hơn.  
> *A higher value makes the index larger.*
>
> Giá trị cao hơn làm build lâu hơn.  
> *A higher value makes the build slower.*

> **`ef_construction`**
>
> Đây là độ "cẩn thận" khi xây mạng lưới.  
> *This is the "carefulness" level while building the network.*
>
> Cụ thể nó là số ứng viên mà thuật toán xem xét khi nối điểm mới vào.  
> *Concretely it is the candidate count the algorithm weighs when linking a new point.*
>
> Giá trị mặc định là 64.  
> *The default value is 64.*
>
> Giá trị cao hơn cho chất lượng mạng lưới tốt hơn.  
> *A higher value gives a better network.*
>
> Chất lượng tốt hơn dẫn tới recall tốt hơn.  
> *A better network leads to better recall.*
>
> Giá trị cao hơn làm build chậm hơn.  
> *A higher value makes the build slower.*

#### Tinh chỉnh lúc query — quan trọng ngang lúc build
*Tuning at query time — as important as build time*

```sql
BEGIN;                            -- mở một transaction (một nhóm lệnh chạy như một khối)
                                  -- open a transaction (a group of commands run as one block)
SET LOCAL hnsw.ef_search = 100;   -- mặc định là 40. Cao hơn = recall tốt hơn, nhưng chậm hơn.
                                  -- the default is 40. Higher = better recall, but slower.
SELECT id, title
FROM documents
ORDER BY embedding <=> '[... 1536 con số ...]'
LIMIT 10;
COMMIT;                           -- kết thúc transaction
                                  -- end the transaction
```

**`ef_search`** là số ứng viên mà thuật toán giữ lại trong lúc dò đường tìm kiếm.  
***`ef_search`** is the candidate count the algorithm keeps while probing during search.*

Đây là **núm xoay recall–tốc độ ở thời điểm query**.  
*This is the **recall-speed dial at query time**.*

Bạn chỉnh được nó mà không cần build lại index.  
*You can turn it without rebuilding the index.*

Xoay lên thì chính xác hơn.  
*Turn it up for more accuracy.*

Xoay lên thì chậm hơn.  
*Turn it up and it runs slower.*

Xoay xuống thì nhanh hơn.  
*Turn it down for more speed.*

Xoay xuống thì dễ sót hơn.  
*Turn it down and it misses more.*

> **🧩 Mẹo staff — một cái bẫy production thật / A real production trap**
>
> Hãy dùng `SET LOCAL` bên trong `BEGIN ... COMMIT`.  
> *Use `SET LOCAL` inside `BEGIN ... COMMIT`.*
>
> **Đừng** dùng `SET` toàn cục trên production.  
> ***Do not*** use a global `SET` in production.*
>
> Đây là lý do.  
> *Here is the reason.*
>
> `SET` thường sẽ gắn giá trị đó vào **connection** — kết nối tới database.  
> *`SET` usually attaches the value to the **connection**.*
>
> Ứng dụng thật hầu như luôn dùng **connection pool** — bể kết nối.  
> *A real application almost always uses a **connection pool**.*
>
> Connection pool là một tập kết nối được tạo sẵn.  
> *A connection pool is a set of pre-created connections.*
>
> Các kết nối đó được **dùng lại luân phiên** cho nhiều request khác nhau.  
> *Those connections are **reused in rotation** across many different requests.*
>
> Mục đích là khỏi phải tạo kết nối mới mỗi lần.  
> *The purpose is to avoid creating a new connection every time.*
>
> Lý do là việc tạo kết nối rất tốn.  
> *The reason is the high cost of creating a connection.*
>
> Phổ biến nhất là **PgBouncer**.  
> *The most popular one is **PgBouncer**.*
>
> Đây là hậu quả.  
> *Here is the consequence.*
>
> Bạn chạy `SET hnsw.ef_search = 500` cho một truy vấn nặng của mình.  
> *You run `SET hnsw.ef_search = 500` for one heavy query of your own.*
>
> Kết nối đó được trả về pool.  
> *That connection returns to the pool.*
>
> Request tiếp theo của một người dùng khác vớ đúng kết nối ấy.  
> *The next request from a different user grabs that very connection.*
>
> Request đó bỗng dưng chạy chậm gấp 10 lần.  
> *That request suddenly runs ten times slower.*
>
> Không ai hiểu nguyên nhân.  
> *Nobody understands the cause.*
>
> Đây là loại bug cực kỳ khó truy.  
> *This is an extremely hard bug to track down.*
>
> Nó xuất hiện *ngẫu nhiên*.  
> *It appears *at random*.*
>
> Nó không tái hiện được trên máy dev.  
> *It does not reproduce on a dev machine.*
>
> `SET LOCAL` chỉ có hiệu lực trong transaction hiện tại.  
> *`SET LOCAL` takes effect only inside the current transaction.*
>
> Vì thế nó miễn nhiễm với vấn đề này.  
> *It is therefore immune to this problem.*

### 2.4. Pipeline thực tế bằng Python (mục tiêu #3 của bài gốc)
*2.4. A real pipeline in Python (objective #3 of the original lesson)*

Đây là đoạn code gần với công việc thật nhất trong giáo trình.  
*This is the snippet closest to real work in the whole course.*

Nó sinh embedding.  
*It generates embeddings.*

Nó lưu embedding vào Postgres.  
*It stores the embeddings into Postgres.*

Nó tìm kiếm trên dữ liệu đó.  
*It searches over that data.*

Đó là đủ một vòng đời.  
*That is a full life cycle.*

Có vài thuật ngữ cần biết trước khi đọc code.  
*A few terms are worth knowing before you read the code.*

> **Library — thư viện**
>
> Library là gói code do người khác viết sẵn.  
> *A library is a package of code written by someone else.*
>
> Bạn cài nó về rồi dùng lại.  
> *You install it and reuse it.*
>
> **pip** là công cụ cài thư viện của Python.  
> ***pip** is the library installer for Python.*

> **sentence-transformers**
>
> Đây là thư viện chạy embedding model **ngay trên máy bạn**.  
> *This library runs an embedding model **right on your machine**.*
>
> Nó miễn phí.  
> *It is free.*
>
> Nó không cần khoá API.  
> *It needs no API key.*
>
> Lần chạy đầu nó sẽ tải model về.  
> *On the first run it downloads the model.*
>
> Model đó nặng vài trăm MB.  
> *That model weighs a few hundred MB.*

> **psycopg**
>
> Đây là thư viện Python để nói chuyện với PostgreSQL.  
> *This is the Python library for talking to PostgreSQL.*
>
> Bản mới nhất là phiên bản 3.  
> *The newest release is version 3.*
>
> Bạn viết nó là `psycopg[binary]`.  
> *You write it as `psycopg[binary]`.*

> **NumPy array — mảng NumPy**
>
> Đây là kiểu mảng số của thư viện NumPy.  
> *This is the numeric array type from the NumPy library.*
>
> Nó là chuẩn de-facto cho tính toán số học trong Python.  
> *It is the de-facto standard for numeric computing in Python.*
>
> Embedding model trả về đúng kiểu này.  
> *An embedding model returns exactly this type.*

> **Cursor — con trỏ**
>
> Cursor là đối tượng dùng để gửi lệnh SQL.  
> *A cursor is the object used to send SQL commands.*
>
> Cursor cũng nhận kết quả về.  
> *A cursor also receives the results.*

> **Commit**
>
> Commit là xác nhận "ghi thật" các thay đổi xuống database.  
> *A commit confirms that changes are truly written to the database.*
>
> Bạn chưa commit.  
> *Suppose you have not committed.*
>
> Khi đó thay đổi chưa chính thức.  
> *The changes are then not official yet.*

```python
# Cài đặt: pip install sentence-transformers "psycopg[binary]" pgvector
# Install: pip install sentence-transformers "psycopg[binary]" pgvector

from sentence_transformers import SentenceTransformer   # để sinh embedding
                                                        # to generate embeddings
import psycopg                                          # để nói chuyện với Postgres
                                                        # to talk to Postgres
from pgvector.psycopg import register_vector            # cầu nối kiểu vector <-> numpy
                                                        # bridge between the vector type and numpy

# Model này sinh ra vector 384 chiều -> cột trong DB phải là vector(384). Phải khớp!
# This model makes 384-dimension vectors -> the DB column must be vector(384). Match it!
model = SentenceTransformer("all-MiniLM-L6-v2")

DOCS = [
    ("Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét", "outerwear"),
    ("Kem chống nắng SPF50 bảo vệ da khỏi tia UV mùa hè",    "skincare"),
    ("Khăn len dệt tay ấm áp cho những ngày lạnh",           "accessories"),
    ("Quần short thoáng mát đi biển ngày nắng",              "clothing"),
]

# Chuỗi kết nối có dạng: postgresql://user:mật_khẩu@địa_chỉ/tên_database
# The connection string looks like: postgresql://user:password@host/database_name
conn = psycopg.connect("postgresql://user:pass@localhost/shop")
register_vector(conn)   # dạy psycopg cách chuyển numpy array <-> kiểu vector của Postgres.
                        # teaches psycopg to convert a numpy array to and from the vector type.
                        # Thiếu dòng này bạn sẽ phải tự nối chuỗi '[...]' bằng tay, rất dễ sai.
                        # Without it you must build the '[...]' string by hand, which is error-prone.

with conn.cursor() as cur:                 # "with" đảm bảo cursor tự đóng khi xong
                                           # "with" makes sure the cursor closes when done
    cur.execute("CREATE EXTENSION IF NOT EXISTS vector")
    cur.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id serial PRIMARY KEY,
            content text,
            category text,
            embedding vector(384)          -- khớp 384 chiều của all-MiniLM-L6-v2
                                           -- matches the 384 dimensions of all-MiniLM-L6-v2
        )
    """)

    # --- BƯỚC 1: sinh embedding cho từng tài liệu rồi chèn vào bảng ---
    # --- STEP 1: generate an embedding per document, then insert it into the table ---
    for content, category in DOCS:
        vec = model.encode(content)        # biến chữ -> numpy array 384 số. ĐÂY là bước AI.
                                           # turns text into a 384-number numpy array. THIS is the AI step.
        cur.execute(
            "INSERT INTO products (content, category, embedding) VALUES (%s, %s, %s)",
            (content, category, vec),      # %s là "chỗ trống" được điền an toàn bởi thư viện.
                                           # %s is a "blank" filled in safely by the library.
                                           # Luôn làm thế này, ĐỪNG nối chuỗi bằng dấu +,
                                           # Always do it this way, DO NOT concatenate with +,
                                           # vì nối chuỗi mở đường cho lỗi bảo mật SQL injection.
                                           # because concatenation opens the door to SQL injection.
        )
    conn.commit()                          # ghi thật xuống database
                                           # truly write it down to the database

    # --- BƯỚC 2: đánh index HNSW ---
    # --- STEP 2: build the HNSW index ---
    # Lưu ý thứ tự: nên nạp dữ liệu xong RỒI mới build index. Build một lần trên
    # dữ liệu có sẵn nhanh hơn nhiều so với việc cập nhật index sau từng dòng INSERT.
    # Note the order: load the data FIRST, then build the index. One build over existing
    # data is far faster than updating the index after every single INSERT.
    cur.execute("""
        CREATE INDEX IF NOT EXISTS products_emb_idx
        ON products USING hnsw (embedding vector_cosine_ops)
    """)
    conn.commit()

    # --- BƯỚC 3: tìm kiếm ---
    # --- STEP 3: search ---
    query_vec = model.encode("đồ mặc cho trời lạnh mùa đông")   # câu hỏi cũng phải
                                                                # the query must also
                                                                # đi qua ĐÚNG model đó
                                                                # go through the SAME model
    cur.execute("""
        SELECT content, embedding <=> %s AS distance
        FROM products
        ORDER BY embedding <=> %s
        LIMIT 2
    """, (query_vec, query_vec))                                # truyền vector 2 lần: 1 cho
                                                                # pass the vector twice: one for
                                                                # cột distance, 1 cho ORDER BY
                                                                # the distance column, one for ORDER BY

    for content, distance in cur.fetchall():                    # fetchall = lấy hết kết quả
                                                                # fetchall = take every result
        print(f"{distance:.4f}  |  {content}")                  # in khoảng cách làm tròn 4 số lẻ
                                                                # print the distance to 4 decimals

conn.close()
```

**Kết quả kỳ vọng / Expected output**

Bạn nhớ quy ước cũ.  
*Remember the old convention.*

Khoảng cách nhỏ nghĩa là gần nghĩa.  
*A small distance means close in meaning.*

```
0.31xx  |  Áo khoác lông vũ giữ ấm cực tốt cho mùa đông giá rét
0.42xx  |  Khăn len dệt tay ấm áp cho những ngày lạnh
```

Hãy nhìn kỹ điều vừa xảy ra.  
*Look closely at what just happened.*

Câu truy vấn **"đồ mặc cho trời lạnh mùa đông"** không chứa chữ "áo khoác".  
*The query **"đồ mặc cho trời lạnh mùa đông"** contains no "áo khoác".*

Nó không chứa chữ "khăn len".  
*It contains no "khăn len".*

Nó không chứa chữ "lông vũ".  
*It contains no "lông vũ".*

Với `LIKE` thì kết quả là con số 0 tròn trĩnh.  
*With `LIKE` the result is a round zero.*

Model lại *hiểu ý nghĩa*.  
*The model *understands the meaning* instead.*

Model trả về đúng hai món đồ mùa đông.  
*The model returns exactly the two winter items.*

Model trả về đúng thứ tự mức độ liên quan.  
*The model returns them in the right order of relevance.*

**Đó là semantic search thật sự.**  
***That is real semantic search.***

Bạn vừa tự dựng nó bằng khoảng 30 dòng code.  
*You just built it yourself in about 30 lines of code.*

Có một chi tiết bắt buộc phải nhớ.  
*One detail is mandatory to remember.*

**Câu truy vấn phải đi qua đúng cùng một embedding model với dữ liệu.**  
***The query must pass through the very same embedding model as the data.***

Embedding từ hai model khác nhau nằm trên hai "bản đồ" hoàn toàn khác nhau.  
*Embeddings from two different models live on two completely different "maps".*

Bạn đem so khoảng cách giữa chúng.  
*Suppose you compare distances between them.*

Con số vẫn ra.  
*A number still comes out.*

Máy vẫn tính được.  
*The machine can still compute it.*

Ý nghĩa của con số đó là **rác**.  
*The meaning of that number is **garbage**.*

Đây là lỗi âm thầm và nguy hiểm.  
*This is a silent and dangerous failure.*

Không có thông báo lỗi nào cả.  
*There is no error message at all.*

Chỉ có chất lượng tìm kiếm tự nhiên tệ đi.  
*Only the search quality quietly degrades.*

### 2.5. Filtered search — nơi pgvector toả sáng
*2.5. Filtered search — where pgvector shines*

**Filtered search** nghĩa là tìm kiếm có lọc.  
***Filtered search** means search with filters.*

Bạn vừa tìm theo độ giống nhau.  
*You search by similarity.*

Bạn vừa áp thêm điều kiện nghiệp vụ.  
*You also apply business conditions.*

Trong đời thật thì **gần như mọi** tìm kiếm đều có lọc.  
*In real life **almost every** search has filters.*

```sql
-- "Tìm 10 sản phẩm giống nhất với câu truy vấn, NHƯNG chỉ trong nhóm 'outerwear',
--  của khách hàng số 42, còn hàng, và được tạo trong 30 ngày gần đây."
-- "Find the 10 most similar products, BUT only in the 'outerwear' category,
--  for customer 42, in stock, and created within the last 30 days."
SELECT id, content
FROM products
WHERE category   = 'outerwear'
  AND tenant_id  = 42
  AND in_stock   = true
  AND created_at > now() - interval '30 days'
ORDER BY embedding <=> '[...]'
LIMIT 10;
```

Một câu SQL.  
*One SQL statement.*

Một lần gọi.  
*One call.*

Xong.  
*Done.*

Bây giờ hãy so với cách làm khi bạn dùng một **vector database chuyên dụng**.  
*Now compare this with using a **dedicated vector database**.*

Ví dụ là Pinecone, Weaviate và Qdrant.  
*Examples are Pinecone, Weaviate, and Qdrant.*

Đó là các hệ thống chỉ chuyên lưu vector.  
*Those are systems specialized purely in storing vectors.*

Chúng tách rời khỏi database chính của bạn.  
*They sit apart from your main database.*

1. Bạn gọi sang vector DB với vector truy vấn.  
   *1. You call the vector DB with the query vector.*

   Bạn nhận về một danh sách ID.  
   *You receive a list of IDs.*

2. Bạn gọi tiếp sang database nghiệp vụ với danh sách ID đó.  
   *2. You then call the business database with that list of IDs.*

   Bạn lấy chi tiết và áp các điều kiện lọc.  
   *You fetch the details and apply the filters.*

3. Sau khi lọc, bạn có thể không đủ 10 kết quả.  
   *3. After filtering, you may not have ten results.*

   Khi đó bạn phải quay lại bước 1 xin thêm ứng viên.  
   *You must then return to step 1 for more candidates.*

   Đây là một vòng lặp thủ công.  
   *This is a manual loop.*

Ta so sánh cho rõ hai thứ đánh đổi.  
*Let us compare the two trade-offs clearly.*

> **Round-trip — lượt đi và về qua mạng**
>
> Mỗi lần gọi sang một service khác đều tốn thời gian đi và về qua mạng.  
> *Every call to another service costs a network round-trip.*
>
> Thời gian đó thường là vài tới vài chục mili-giây.  
> *That time is usually a few to a few dozen milliseconds.*
>
> pgvector cần 1 round-trip.  
> *pgvector needs one round-trip.*
>
> Cách kia cần ít nhất 2 round-trip.  
> *The other approach needs at least two.*

> **Failure mode — kiểu hỏng**
>
> Mỗi hệ thống thêm vào là thêm một chỗ có thể sập.  
> *Every added system is one more place that can fail.*
>
> Đó cũng là thêm một thứ phải giám sát.  
> *It is also one more thing to monitor.*
>
> Đó cũng là thêm một trạng thái có thể lệch.  
> *It is also one more state that can drift.*
>
> Ví dụ là vector DB còn dòng mà DB chính đã xoá.  
> *An example is a row still in the vector DB after deletion from the main DB.*
>
> Với pgvector, dữ liệu vector và dữ liệu nghiệp vụ **luôn nhất quán tuyệt đối**.  
> *With pgvector, vector data and business data are **always perfectly consistent**.*
>
> Lý do là chúng nằm trong cùng một transaction.  
> *The reason is their shared transaction.*

> **Đây là câu "flex" đắt giá nhất về pgvector — the most valuable brag about pgvector**
>
> Bạn nên thuộc lòng câu này để nói trong phỏng vấn.  
> *Memorize this line for interviews.*
>
> Siêu năng lực của pgvector không phải là tốc độ.  
> *pgvector's superpower is not speed.*
>
> Siêu năng lực của nó là vector sống chung nhà với dữ liệu nghiệp vụ.  
> *Its superpower is vectors living under the same roof as business data.*
>
> Nhờ vậy tìm kiếm có lọc chỉ là một câu SQL.  
> *Filtered search is therefore just one SQL statement.*
>
> Nó chỉ là một lượt đi về.  
> *It is just one round-trip.*
>
> Nó chỉ là một transaction.  
> *It is just one transaction.*

### 2.6. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh
*2.6. 🧩 [Ngoài bài gốc] — Three classic mistakes and how to avoid them*

#### Lỗi 1 — Overfiltering: "filter chặt xong bị thiếu kết quả"
*Mistake 1 — Overfiltering: "the filter is tight and results go missing"*

Đây là hiện tượng.  
*Here is the symptom.*

Bạn yêu cầu `LIMIT 10`.  
*You ask for `LIMIT 10`.*

Bạn chỉ nhận về 2 dòng.  
*You receive only two rows.*

Trong bảng rõ ràng có nhiều dòng thoả điều kiện.  
*The table clearly holds many rows matching the condition.*

Đây là nguyên nhân.  
*Here is the cause.*

Đây là chỗ khó thật sự.  
*This is a genuinely tricky spot.*

Người rất giỏi cũng vấp.  
*Even very strong engineers trip here.*

Thứ tự thực hiện bên trong không như bạn tưởng.  
*The internal execution order is not what you assume.*

Index HNSW đi tìm khoảng `ef_search` hàng xóm gần nhất **trước**.  
*The HNSW index finds about `ef_search` nearest neighbors **first**.*

Postgres áp mệnh đề `WHERE` lên nhóm ứng viên đó **sau**.  
*Postgres applies the `WHERE` clause to that candidate set **afterwards**.*

Điều kiện lọc của bạn có thể rất hiếm.  
*Your filter condition may be very rare.*

Ví dụ là `category` đó chỉ chiếm 0,5% dữ liệu.  
*An example is a `category` covering only 0.5% of the data.*

Khi đó trong 40 ứng viên ban đầu có thể chỉ 1 tới 2 cái sống sót.  
*Then only one or two of the initial 40 candidates survive.*

Từ chuyên môn cho hiện tượng này là **overfiltering** — lọc quá tay.  
*The technical term for this is **overfiltering**.*

Cách chữa là dùng **iterative index scan** — quét index lặp lại.  
*The cure is the **iterative index scan**.*

Đây là tính năng chủ lực từ pgvector 0.8.0.  
*This is a flagship feature since pgvector 0.8.0.*

```sql
SET hnsw.iterative_scan = relaxed_order;  -- cho phép index tự động quét thêm
                                          -- lets the index automatically scan further
                                          -- cho tới khi đủ số kết quả yêu cầu
                                          -- until it has enough results
SET hnsw.max_scan_tuples = 20000;         -- trần an toàn: quét thêm tối đa 20k dòng rồi dừng,
                                          -- a safety ceiling: scan at most 20k extra rows, then stop,
                                          -- tránh một query xấu quét cả bảng
                                          -- so a bad query does not scan the whole table
```

Có hai lựa chọn giá trị.  
*There are two possible values.*

`strict_order` giữ đúng thứ tự khoảng cách tuyệt đối.  
*`strict_order` keeps the exact distance ordering.*

`relaxed_order` cho phép thứ tự xê dịch chút ít.  
*`relaxed_order` allows the ordering to shift slightly.*

Đổi lại nó nhanh hơn đáng kể.  
*In exchange it runs considerably faster.*

Nó thường giữ được khoảng 95 tới 99% chất lượng.  
*It usually preserves about 95 to 99% of the quality.*

Với đa số hệ thống thật, `relaxed_order` là lựa chọn đúng.  
*For most real systems, `relaxed_order` is the right choice.*

#### Lỗi 2 — Ops class không khớp toán tử query
*Mistake 2 — the ops class does not match the query operator*

Bạn đánh index bằng `vector_l2_ops`.  
*You build the index with `vector_l2_ops`.*

Ops class đó dành cho `<->`.  
*That ops class is meant for `<->`.*

Bạn lại query bằng `<=>`, tức cosine.  
*You then query with `<=>`, the cosine operator.*

Chuyện gì xảy ra?  
*What happens?*

**Postgres im lặng bỏ qua index của bạn.**  
***Postgres silently ignores your index.***

Nó rơi về sequential scan.  
*It falls back to a sequential scan.*

Không có lỗi.  
*There is no error.*

Không có cảnh báo.  
*There is no warning.*

Chỉ có một query đáng lẽ chạy 5ms giờ chạy 5 giây.  
*Only a query that should take 5ms now takes 5 seconds.*

Nó chỉ lộ ra khi dữ liệu đã lớn.  
*It surfaces only once the data grows large.*

Thời điểm đó thường là lúc đông người dùng nhất.  
*That moment is usually your peak traffic hour.*

Cách kiểm tra là dùng **`EXPLAIN ANALYZE`**.  
*The way to check is **`EXPLAIN ANALYZE`**.*

Đây là lệnh của Postgres cho biết nó *thực sự* thực thi query bằng cách nào.  
*This Postgres command reveals how it *actually* executes the query.*

```sql
EXPLAIN ANALYZE
SELECT id FROM documents ORDER BY embedding <=> '[...]' LIMIT 10;
```

Trong kết quả, hãy tìm dòng có chữ **`Index Scan using ..._hnsw...`**.  
*In the output, look for the line with **`Index Scan using ..._hnsw...`**.*

Dòng đó nghĩa là index đang được dùng.  
*That line means the index is in use.*

Đó là dấu hiệu tốt.  
*That is a good sign.*

Bạn có thể thấy **`Seq Scan`** thay vào đó.  
*You may see **`Seq Scan`** instead.*

Khi đó index đang bị bỏ qua.  
*The index is then being ignored.*

Bạn phải sửa ngay.  
*You must fix it right away.*

Thói quen chạy `EXPLAIN ANALYZE` sau mỗi lần đánh index rất đáng có.  
*The habit of running `EXPLAIN ANALYZE` after every index build is worth having.*

Thói quen đó phân biệt người cẩn thận với người đoán mò.  
*That habit separates the careful engineer from the guesser.*

#### Lỗi 3 — Quên rằng ANN là *approximate*
*Mistake 3 — forgetting that ANN is *approximate**

Đây là kịch bản kinh điển.  
*Here is the classic scenario.*

Bạn test trên máy dev với 500 dòng.  
*You test on a dev machine with 500 rows.*

Ở đó chưa có index.  
*There is no index there.*

Search là exact.  
*The search is exact.*

Kết quả đúng 100%.  
*The results are 100% correct.*

Bạn lên production với 5 triệu dòng.  
*You go to production with 5 million rows.*

Ở đó có index HNSW.  
*There is an HNSW index there.*

Người dùng báo "kết quả sai so với trước".  
*Users report "the results differ from before".*

Bạn hoảng.  
*You panic.*

**Không có gì sai cả.**  
***Nothing is wrong.***

Đây là *by design*, tức đúng theo thiết kế.  
*This is *by design*.*

Bạn đã đồng ý đánh đổi recall lấy tốc độ ngay từ lúc gõ `CREATE INDEX`.  
*You agreed to trade recall for speed the moment you typed `CREATE INDEX`.*

Cách xử lý đúng gồm ba bước, theo thứ tự.  
*The right response has three steps, in order.*

1. **Đo** recall hiện tại.  
   *1. **Measure** the current recall.*

   Cách đo cụ thể nằm ở mục 4.3.  
   *The measurement method sits in section 4.3.*

   Bạn cần biết mình đang ở 85% hay 99%.  
   *You need to know whether you are at 85% or 99%.*

2. Recall có thể chưa đạt yêu cầu.  
   *2. The recall may fall short of your target.*

   Khi đó bạn **tăng `ef_search`** rồi đo lại.  
   *In that case **raise `ef_search`** and measure again.*

   Cách này rẻ.  
   *This route is cheap.*

   Cách này không cần build lại index.  
   *This route needs no index rebuild.*

3. Recall vẫn có thể chưa đủ.  
   *3. The recall may still fall short.*

   Khi đó bạn **tăng `m` và `ef_construction`**.  
   *In that case **raise `m` and `ef_construction`**.*

   Sau đó bạn build lại index.  
   *Then rebuild the index.*

   Cách này đắt hơn.  
   *This route is more expensive.*

   Cách này tốn thời gian.  
   *This route takes time.*

Điều tệ nhất bạn có thể làm là mò mẫm chỉnh tham số mà không đo gì cả.  
*The worst thing you can do is tweak parameters blindly without measuring.*

### ✅ Self-check Phần 2
*✅ Self-check for Part 2*

**Câu 1.** Embedding của bạn lấy từ OpenAI. Nên chọn `<->`, `<=>` hay `<#>`? Vì sao?  
***Question 1.** Your embeddings come from OpenAI. Should you pick `<->`, `<=>`, or `<#>`? Why?*

> *Gợi ý đáp án / Suggested answer*
>
> Bạn chọn `<=>`, tức cosine.  
> *Pick `<=>`, the cosine operator.*
>
> Với văn bản ta quan tâm **hướng ngữ nghĩa**.  
> *For text we care about the **semantic direction**.*
>
> Ta không quan tâm độ dài vector.  
> *We do not care about vector length.*
>
> Bạn có thể tự normalize mọi vector về độ dài 1.  
> *You may normalize every vector to length 1.*
>
> Khi đó bạn dùng `<#>` để nhanh hơn.  
> *You can then use `<#>` for more speed.*
>
> Kết quả vẫn tương đương.  
> *The result stays equivalent.*

**Câu 2.** Bạn đánh index `USING hnsw (embedding vector_cosine_ops)` nhưng query bằng `<->`. Chuyện gì xảy ra?  
***Question 2.** You index with `USING hnsw (embedding vector_cosine_ops)` but query with `<->`. What happens?*

> *Gợi ý đáp án / Suggested answer*
>
> Ops class không khớp toán tử.  
> *The ops class does not match the operator.*
>
> Postgres **âm thầm bỏ qua index**.  
> *Postgres **silently ignores the index**.*
>
> Nó rơi về sequential scan.  
> *It falls back to a sequential scan.*
>
> Query chậm đi rất nhiều.  
> *The query becomes far slower.*
>
> Không có lỗi nào được báo.  
> *No error is reported.*
>
> Bạn phát hiện nó bằng `EXPLAIN ANALYZE`.  
> *You detect it with `EXPLAIN ANALYZE`.*
>
> Bạn thấy `Seq Scan` thay vì `Index Scan using ..._hnsw...`.  
> *You see `Seq Scan` instead of `Index Scan using ..._hnsw...`.*
>
> Đó là dấu hiệu dính lỗi này.  
> *That is the sign of this mistake.*

**Câu 3.** Câu `WHERE category='rare_cat'` trả về 2 kết quả thay vì 10. Tên hiện tượng là gì?  
***Question 3.** `WHERE category='rare_cat'` returns 2 results instead of 10. What is this called?*

> *Gợi ý đáp án / Suggested answer*
>
> Nhóm `rare_cat` chỉ chiếm 0,3% dữ liệu.  
> *The `rare_cat` group covers only 0.3% of the data.*
>
> Tên hiện tượng là **overfiltering**.  
> *The phenomenon is called **overfiltering**.*
>
> Index lấy ứng viên trước.  
> *The index picks candidates first.*
>
> `WHERE` lọc sau.  
> *`WHERE` filters afterwards.*
>
> Vậy nên gần hết ứng viên bị loại.  
> *Nearly all candidates are therefore eliminated.*
>
> Bạn khắc phục bằng `hnsw.iterative_scan = relaxed_order`.  
> *You fix it with `hnsw.iterative_scan = relaxed_order`.*
>
> Tính năng này có từ pgvector 0.8 trở lên.  
> *This feature exists from pgvector 0.8 onward.*
>
> Bạn kèm thêm `hnsw.max_scan_tuples` làm trần an toàn.  
> *Add `hnsw.max_scan_tuples` as a safety ceiling.*

**Câu 4.** Vì sao tuyệt đối không nên dùng `SET hnsw.ef_search = ...` toàn cục trên production?  
***Question 4.** Why should you never use a global `SET hnsw.ef_search = ...` in production?*

> *Gợi ý đáp án / Suggested answer*
>
> Giá trị đó bám vào connection.  
> *The value sticks to the connection.*
>
> Connection lại được dùng lại luân phiên qua connection pool.  
> *The connection gets reused in rotation through a connection pool.*
>
> Công cụ phổ biến là PgBouncer.  
> *The common tool is PgBouncer.*
>
> Giá trị đó "rò rỉ" sang query của người dùng khác.  
> *The value "leaks" into another user's query.*
>
> Nó gây chậm hoặc lỗi ngẫu nhiên.  
> *It causes random slowness or errors.*
>
> Loại lỗi này rất khó truy.  
> *This kind of bug is very hard to trace.*
>
> Bạn dùng `SET LOCAL` trong `BEGIN...COMMIT` thay thế.  
> *Use `SET LOCAL` inside `BEGIN...COMMIT` instead.*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED (In depth)*

> Ở đây ta chui xuống **bên dưới bề mặt**.  
> *Here we go **beneath the surface**.*
>
> Ta xem các thuật toán này thực sự hoạt động thế nào.  
> *We look at how these algorithms actually work.*
>
> Ta xem chúng tốn bao nhiêu.  
> *We look at how much they cost.*
>
> Ta xem chúng hỏng ở đâu.  
> *We look at where they break.*
>
> Tôi nói trước một điều.  
> *Let me say one thing up front.*
>
> Mục 3.1 hơi nặng toán.  
> *Section 3.1 leans a bit heavy on math.*
>
> Tôi sẽ giải thích cả những ký hiệu mà sách thường bỏ qua.  
> *I will explain even the symbols textbooks usually skip.*
>
> Bạn cứ đi chậm là được.  
> *Just go slowly.*

### 3.1. Vì sao bài toán này khó? — Big-O và lời nguyền số chiều
*3.1. Why is this problem hard? — Big-O and the curse of dimensionality*

#### Trước hết: Big-O là gì?
*First: what is Big-O?*

**Big-O** đọc là "O lớn".  
***Big-O** is read as "big O".*

Ví dụ ký hiệu là `O(n)`.  
*An example notation is `O(n)`.*

Đây là cách mô tả **thời gian chạy của một thuật toán tăng lên thế nào khi dữ liệu to ra**.  
*This describes **how an algorithm's running time grows as the data grows**.*

Nó không cho biết chương trình chạy bao nhiêu giây.  
*It does not tell you how many seconds the program takes.*

Nó cho biết *xu hướng*.  
*It tells you the *trend*.*

Xu hướng mới là thứ quyết định hệ thống sống hay chết ở quy mô lớn.  
*The trend is what decides life or death at scale.*

> **`O(n)`**
>
> Thời gian tăng **tỷ lệ thuận** với số dòng.  
> *The time grows **proportionally** with the row count.*
>
> Dữ liệu gấp 10 lần thì chậm gấp 10 lần.  
> *Ten times the data means ten times slower.*

> **`O(log n)`**
>
> Thời gian tăng **cực chậm**.  
> *The time grows **extremely slowly**.*
>
> Dữ liệu gấp 1000 lần thì chỉ chậm thêm khoảng 10 lần.  
> *A thousand times the data is only about ten times slower.*
>
> Đây là con số vàng mà mọi hệ thống lớn theo đuổi.  
> *This is the golden figure every large system chases.*
>
> > Đây là một analogy.  
> > *Here is an analogy.*
> >
> > Bạn tra một cuốn từ điển giấy.  
> > *You look something up in a paper dictionary.*
> >
> > Từ điển dày gấp đôi không làm bạn tra lâu gấp đôi.  
> > *A dictionary twice as thick does not take twice as long.*
> >
> > Bạn chỉ cần lật thêm đúng một nhát nữa để chia đôi.  
> > *You need exactly one more flip to halve it.*
> >
> > Đó là `O(log n)`.  
> > *That is `O(log n)`.*

> **`O(n · d)`**
>
> Thời gian tỷ lệ thuận với cả số dòng `n` và số chiều `d`.  
> *The time grows proportionally with both the row count `n` and the dimension count `d`.*

#### Chi phí thật của exact search
*The real cost of exact search*

Ở Phần 1 ta tính khoảng cách tới **tất cả** các điểm.  
*In Part 1 we computed the distance to **every** point.*

Đó là **brute-force exact search**.  
*That is **brute-force exact search**.*

Chi phí của nó là `O(n · d)`.  
*Its cost is `O(n · d)`.*

Hãy quy ra con số cho thật cụ thể.  
*Let us turn that into concrete numbers.*

Giả sử `n` bằng 100 triệu vector.  
*Suppose `n` is 100 million vectors.*

Giả sử `d` bằng 1536 chiều.  
*Suppose `d` is 1536 dimensions.*

```
100.000.000 vector  ×  1536 chiều  ≈  153.600.000.000 phép nhân cho MỖI câu truy vấn
100,000,000 vectors ×  1536 dims   ≈  153,600,000,000 multiplications for EVERY query
```

Đó là hơn 150 tỷ phép tính cho một lần bấm nút tìm kiếm.  
*That is over 150 billion operations for one press of the search button.*

CPU hiện đại có tính toán song song.  
*Modern CPUs have parallel computation.*

Con số này vẫn **bất khả thi**.  
*This number is still **infeasible**.*

Hệ thống thời gian thực cần trả lời trong 100 mili-giây.  
*A real-time system must answer within 100 milliseconds.*

Đó là lý do ta buộc phải tìm cách khác.  
*That is why we are forced to find another way.*

#### Vì sao không dùng cây phân hoạch như trong không gian 2D?
*Why not use a partitioning tree as in 2D space?*

Đây là câu hỏi rất hợp lý.  
*This is a very reasonable question.*

Trong không gian 2 hay 3 chiều, dân đồ hoạ và bản đồ có sẵn những cấu trúc tốt.  
*In two or three dimensions, graphics and mapping people already have good structures.*

Ví dụ là **kd-tree**, tức cây chia không gian thành các nửa.  
*One example is the **kd-tree**, a tree splitting space into halves.*

Nó giải bài toán hàng xóm gần nhất rất gọn.  
*It solves the nearest neighbor problem neatly.*

Sao ta không dùng luôn?  
*Why not just use it?*

Lý do là một hiện tượng có tên nghe rất kêu.  
*The reason is a phenomenon with a grand-sounding name.*

Tên đó là **curse of dimensionality** — lời nguyền số chiều.  
*That name is the **curse of dimensionality**.*

Nội dung của lời nguyền nằm trong một câu.  
*The content of the curse fits in one sentence.*

Khi số chiều `d` lớn, **mọi điểm trở nên gần như cách đều nhau**.  
*When the dimension count `d` is large, **all points become almost equidistant**.*

Lớn ở đây nghĩa là hàng trăm tới hàng nghìn chiều.  
*Large here means hundreds to thousands of dimensions.*

Khoảng cách tới hàng xóm gần nhất xích lại gần khoảng cách tới điểm xa nhất.  
*The distance to the nearest neighbor approaches the distance to the farthest point.*

Khái niệm "gần" mất dần ý nghĩa phân biệt.  
*The notion of "near" gradually loses its discriminating power.*

> Đây là analogy để cảm nhận.  
> *Here is an analogy to feel it.*
>
> Trên một đường thẳng, bạn chỉ có 2 hướng để trốn.  
> *On a straight line, you have only two directions to hide in.*
>
> Hai hướng đó là trái và phải.  
> *Those two directions are left and right.*
>
> Vậy nên hàng xóm rất dễ xác định.  
> *A neighbor is therefore very easy to identify.*
>
> Trên mặt phẳng, bạn có vô số hướng.  
> *On a plane, you have countless directions.*
>
> Bạn vẫn quản được chúng.  
> *You can still manage them.*
>
> Trong không gian 1536 chiều, có quá nhiều "hướng" để một điểm lệch đi.  
> *In 1536-dimensional space, there are far too many "directions" for a point to drift.*
>
> Hai điểm ngẫu nhiên bất kỳ gần như luôn lệch nhau ở đâu đó.  
> *Any two random points almost always differ somewhere.*
>
> Kết quả là chúng đều cách nhau xấp xỉ như nhau.  
> *The result is that they are all roughly the same distance apart.*

Đây là hệ quả kỹ thuật.  
*Here is the technical consequence.*

kd-tree phải duyệt gần hết cây mới dám khẳng định đã tìm đúng.  
*A kd-tree must walk nearly the whole tree before it can claim correctness.*

Vậy nên nó **không nhanh hơn brute force là bao**.  
*It is therefore **barely faster than brute force**.*

Cây phân hoạch không gian mất tác dụng.  
*Space-partitioning trees lose their effectiveness.*

Đó chính là lý do ngành này chuyển sang **ANN**.  
*That is exactly why the field moved to **ANN**.*

ANN chấp nhận sai một chút để nhanh hơn hàng nghìn lần.  
*ANN accepts a little error for a thousandfold speedup.*

Lý do không phải là lười.  
*The reason is not laziness.*

Exact search ở quy mô lớn là **bất khả thi về mặt vật lý**.  
*Exact search at large scale is **physically infeasible**.*

Từ đây trở đi ta xem hai họ thuật toán ANN mà pgvector cung cấp.  
*From here we look at the two ANN algorithm families pgvector offers.*

### 3.2. IVFFlat — "chia quận, chỉ tìm trong vài quận gần"
*3.2. IVFFlat — "split into districts, search only a few nearby ones"*

**IVFFlat** viết đầy đủ là *Inverted File with Flat compression*.  
***IVFFlat** spells out as *Inverted File with Flat compression*.*

Nghĩa của nó là tệp đảo, lưu vector nguyên bản không nén.  
*It means an inverted file storing the original vectors without compression.*

#### Ý tưởng, kể bằng analogy
*The idea, told through an analogy*

Bạn đứng ở Quận 1.  
*You are standing in District 1.*

Bạn muốn tìm quán phở gần nhất trong thành phố.  
*You want the nearest phở restaurant in the city.*

Cách ngu ngốc là đo khoảng cách tới từng quán phở trong cả thành phố.  
*The foolish way is measuring the distance to every phở place in the city.*

Đó chính là brute force.  
*That is exactly brute force.*

Đây là cách khôn ngoan.  
*Here is the smart way.*

1. Bạn chia thành phố thành các **quận**.  
   *1. You split the city into **districts**.*

   Mỗi quận có một điểm trung tâm.  
   *Each district has a center point.*

2. Bạn so vị trí của mình với **trung tâm các quận**.  
   *2. You compare your position against the **district centers**.*

   Việc này chỉ tốn vài chục phép so.  
   *This costs only a few dozen comparisons.*

3. Bạn chọn ra 2 tới 3 quận có trung tâm gần bạn nhất.  
   *3. You pick the two or three districts with the nearest centers.*

4. Bạn chỉ đo khoảng cách tới các quán phở **trong những quận đó**.  
   *4. You measure distances only to the phở places **inside those districts**.*

   Bạn bỏ qua toàn bộ phần còn lại của thành phố.  
   *You skip the entire rest of the city.*

IVFFlat làm đúng như vậy trên không gian vector.  
*IVFFlat does exactly this in vector space.*

Đây là chỗ khớp với kỹ thuật.  
*Here is where the analogy meets the technology.*

"Quận" trong thuật toán gọi là **list**.  
*A "district" in the algorithm is called a **list**.*

Số quận là tham số `lists`.  
*The district count is the `lists` parameter.*

"Trung tâm quận" gọi là **centroid** — tâm cụm.  
*A "district center" is called a **centroid**.*

Centroid là một vector đại diện cho cả cụm.  
*A centroid is one vector representing the whole cluster.*

Việc chia quận được làm bằng thuật toán **k-means clustering** — phân cụm k-means.  
*The district split is done by **k-means clustering**.*

Đây là một thuật toán tự động nhóm các điểm gần nhau thành `k` cụm.  
*This algorithm automatically groups nearby points into `k` clusters.*

Nó tính tâm của mỗi cụm.  
*It computes the center of each cluster.*

Nó lặp đi lặp lại hai bước.  
*It repeats two steps.*

Bước một là gán mỗi điểm vào cụm có tâm gần nhất.  
*Step one assigns each point to the cluster with the nearest center.*

Bước hai là tính lại tâm.  
*Step two recomputes the centers.*

Nó lặp cho tới khi ổn định.  
*It repeats until things stabilize.*

Số quận được xem lúc tìm kiếm gọi là `probes` — số lần thăm dò.  
*The number of districts visited at search time is called `probes`.*

#### Điểm cực kỳ quan trọng: IVFFlat cần dữ liệu để "học" trước
*A crucial point: IVFFlat needs data to "learn" from first*

IVFFlat phải chạy k-means để tìm centroid.  
*IVFFlat must run k-means to find the centroids.*

Vậy nên nó **bắt buộc phải có sẵn dữ liệu trong bảng** lúc build index.  
*It therefore **requires data already in the table** at index build time.*

Bạn build index trên bảng rỗng rồi mới đổ dữ liệu vào.  
*Suppose you build the index on an empty table, then load the data.*

Khi đó các centroid được tính trên hư không.  
*The centroids are then computed out of thin air.*

Phân hoạch trở nên vô nghĩa.  
*The partitioning becomes meaningless.*

Recall thảm hại.  
*Recall becomes disastrous.*

Đây là điểm khác biệt lớn so với HNSW.  
*This is a major difference from HNSW.*

HNSW không cần train.  
*HNSW needs no training.*

#### Cú pháp
*Syntax*

```sql
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 1000);       -- công thức gợi ý: số_dòng/1000 nếu ≤ 1 triệu dòng;
                           -- suggested formula: row_count/1000 when ≤ 1 million rows;
                           -- sqrt(số_dòng) nếu > 1 triệu dòng.
                           -- sqrt(row_count) when > 1 million rows.

SET ivfflat.probes = 32;   -- gợi ý khởi đầu: sqrt(lists). Cao hơn = recall↑ nhưng chậm hơn.
                           -- starting suggestion: sqrt(lists). Higher = recall↑ but slower.
```

#### Trade-off — và định nghĩa của chính từ này
*Trade-off — and the definition of the word itself*

**Trade-off** nghĩa là sự đánh đổi.  
*A **trade-off** is an exchange.*

Bạn được cái này thì phải mất cái kia.  
*You gain one thing and lose another.*

Không có lựa chọn nào tốt hơn mọi mặt.  
*No option is better on every axis.*

Nhận diện và nói rõ trade-off là kỹ năng cốt lõi ở tầm senior và staff.  
*Spotting and stating trade-offs is a core skill at senior and staff level.*

Người mới thường tìm "cách tốt nhất".  
*Beginners look for "the best way".*

Người có kinh nghiệm hỏi "tốt nhất *cho ràng buộc nào*".  
*Experienced people ask "best *under which constraints*".*

Với IVFFlat, `probes` chính là núm xoay trade-off.  
*For IVFFlat, `probes` is exactly the trade-off dial.*

`probes = 1` cho tốc độ cực nhanh.  
*`probes = 1` gives extreme speed.*

Nó lại dễ trượt.  
*It also misses easily.*

Hàng xóm thật sự có thể nằm ở quận kế bên.  
*The true neighbor may sit in the next district.*

Khi đó bạn không bao giờ thấy nó.  
*You then never see it.*

`probes = lists` quét hết mọi quận.  
*`probes = lists` scans every district.*

Cách đó đúng bằng exact search.  
*That is exactly equal to exact search.*

Nó mất sạch lợi ích tốc độ.  
*It loses the entire speed benefit.*

Chi phí truy vấn xấp xỉ gồm hai phần.  
*The query cost has roughly two parts.*

Phần thứ nhất là `O(lists · d)` cho việc so với các centroid.  
*The first part is `O(lists · d)` for comparing against the centroids.*

Phần thứ hai là `O((n/lists) · probes · d)` cho việc quét bên trong các quận đã chọn.  
*The second part is `O((n/lists) · probes · d)` for scanning inside the chosen districts.*

#### Điểm yếu chí mạng của IVFFlat
*The fatal weaknesses of IVFFlat*

1. **Hàng xóm nằm sát ranh giới quận dễ bị bỏ sót.**  
   *1. **Neighbors near a district boundary are easily missed.***

   Một điểm ở rìa quận A có thể gần bạn hơn mọi điểm trong quận B.  
   *A point at the edge of district A may be nearer to you than anything in district B.*

   Bạn chỉ probe quận B.  
   *Suppose you probe only district B.*

   Khi đó điểm ấy vô hình.  
   *That point is then invisible.*

2. **IVFFlat mang tính tĩnh.**  
   *2. **IVFFlat is static.***

   Sau khi build, bạn thêm hàng triệu dòng mới.  
   *After the build, you add millions of new rows.*

   Chúng vẫn bị gán vào các centroid cũ.  
   *They are still assigned to the old centroids.*

   Phân hoạch dần lệch đi.  
   *The partitioning gradually drifts.*

   Recall tụt xuống một cách âm thầm.  
   *Recall silently drops.*

   Cách chữa duy nhất là `REINDEX` định kỳ.  
   *The only cure is a periodic `REINDEX`.*

   `REINDEX` nghĩa là dựng lại index từ đầu.  
   *`REINDEX` means rebuilding the index from scratch.*

   Với dữ liệu thay đổi liên tục, đây là gánh nặng vận hành thật sự.  
   *For constantly changing data, this is a real operational burden.*

### 3.3. HNSW — mạng lưới nhiều tầng có "đường cao tốc"
*3.3. HNSW — a multi-layer network with "highways"*

**HNSW** viết đầy đủ là *Hierarchical Navigable Small World*.  
***HNSW** spells out as *Hierarchical Navigable Small World*.*

*Hierarchical* nghĩa là phân tầng.  
**Hierarchical* means layered.*

*Navigable* nghĩa là đi lại được.  
**Navigable* means traversable.*

*Small world* là thuật ngữ từ lý thuyết mạng.  
**Small world* is a term from network theory.*

Nó chỉ loại mạng mà hai điểm bất kỳ chỉ cách nhau vài bước nhảy.  
*It names a network where any two points are only a few hops apart.*

Ý tưởng đó giống "sáu độ phân cách" giữa hai người bất kỳ trên Trái Đất.  
*That idea resembles the "six degrees of separation" between any two people on Earth.*

#### Ý tưởng, kể bằng analogy giao thông
*The idea, told through a traffic analogy*

Hãy tưởng tượng một hệ thống giao thông nhiều tầng.  
*Imagine a multi-layer transport system.*

**Tầng trên cùng** có rất ít nút.  
*The **top layer** has very few nodes.*

Đó là **đường cao tốc liên tỉnh**.  
*It is the **intercity highway**.*

Mỗi lần đi là nhảy được rất xa.  
*Each move jumps a very long way.*

**Tầng giữa** có nhiều nút hơn.  
*The **middle layer** has more nodes.*

Đó là đường quốc lộ nối các thành phố.  
*It is the national road linking cities.*

**Tầng dưới cùng** chứa **mọi** nút.  
*The **bottom layer** holds **every** node.*

Đó toàn là ngõ nhỏ.  
*It is all narrow alleys.*

Mỗi bước đi rất ngắn.  
*Each step is very short.*

Bạn muốn đi từ Hà Nội tới một con hẻm cụ thể ở Sài Gòn.  
*Suppose you travel from Hanoi to a specific alley in Saigon.*

Bạn không bò theo ngõ nhỏ suốt 1700 km.  
*You do not crawl through alleys for 1700 km.*

Bạn đi cao tốc để tới đúng vùng trước.  
*You take the highway to reach the right region first.*

Đó là vài bước nhảy dài.  
*That is a few long hops.*

Rồi bạn tụt xuống quốc lộ.  
*Then you drop down to the national road.*

Cuối cùng bạn mới rẽ vào từng ngõ nhỏ để tới đúng số nhà.  
*Finally you turn into the alleys to reach the exact house number.*

HNSW tìm kiếm y hệt như vậy.  
*HNSW searches in exactly this way.*

Nó bắt đầu ở tầng cao nhất.  
*It starts at the highest layer.*

Nó nhảy những bước dài để về đúng vùng của không gian vector.  
*It takes long hops toward the right region of vector space.*

Nó tụt dần xuống tầng thấp hơn để tinh chỉnh.  
*It descends layer by layer to refine.*

Tới tầng đáy thì nó tìm ra hàng xóm chính xác.  
*At the bottom layer it finds the precise neighbors.*

Đây là chỗ khớp với kỹ thuật, nói bằng thuật ngữ chuẩn.  
*Here is the technical mapping, in standard terminology.*

> **Graph — đồ thị**
>
> Graph là cấu trúc gồm các **node** — nút.  
> *A graph is a structure made of **nodes**.*
>
> Ở đây mỗi node là một vector.  
> *Here each node is one vector.*
>
> Các node nối với nhau bằng **edge** — cạnh.  
> *The nodes connect through **edges**.*
>
> Ở đây edge nghĩa là "hàng xóm của nhau".  
> *Here an edge means "neighbors of each other".*

Mỗi node được nối tối đa `m` hàng xóm gần nhất ở mỗi tầng.  
*Each node links to at most `m` nearest neighbors on each layer.*

Đó chính là tham số `m` bạn đã gặp ở mục 2.3.  
*That is exactly the `m` parameter you met in section 2.3.*

Tầng của mỗi node được chọn **ngẫu nhiên theo phân phối mũ**.  
*Each node's layer is chosen **randomly by an exponential distribution**.*

Đa số node chỉ nằm ở tầng đáy.  
*Most nodes sit only on the bottom layer.*

Càng lên cao càng ít node.  
*The higher the layer, the fewer the nodes.*

Đây chính là thứ tạo ra hiệu ứng "cao tốc thưa, ngõ nhỏ dày".  
*This is exactly what creates the "sparse highways, dense alleys" effect.*

#### Con số cần thuộc lòng
*Numbers to memorize*

> **Độ phức tạp truy vấn: khoảng `O(log n)`**
>
> Đây là con số vàng.  
> *This is the golden figure.*
>
> Nó gần như chắc chắn sẽ được hỏi trong phỏng vấn.  
> *It is almost certain to come up in an interview.*
>
> Ý nghĩa của nó rất rõ ràng.  
> *Its meaning is very clear.*
>
> Từ 1 triệu lên 1 tỷ vector, thời gian tìm kiếm chỉ tăng vài lần.  
> *From 1 million to 1 billion vectors, search time grows only a few times.*
>
> Nó không tăng 1000 lần.  
> *It does not grow a thousandfold.*

> **Độ phức tạp lúc build: khoảng `O(n · log n · m · d)`**
>
> Cách này chậm hơn IVFFlat rất nhiều.  
> *This is far slower than IVFFlat.*
>
> Nó tốn RAM hơn hẳn.  
> *It uses considerably more RAM.*
>
> Lý do là nó phải chèn từng node một.  
> *The reason is its node-by-node insertion.*
>
> Với mỗi node nó lại phải đi tìm hàng xóm.  
> *For each node it must go find the neighbors.*

Đó chính là trade-off cốt lõi của HNSW.  
*That is exactly the core trade-off of HNSW.*

**Query nhanh và recall cao, đổi lại build chậm và ngốn RAM.**  
***Fast queries and high recall, in exchange for slow builds and heavy RAM use.***

### 3.4. Bảng so sánh index — câu hỏi phỏng vấn kinh điển
*3.4. The index comparison table — a classic interview question*

Trước bảng, có một thuật ngữ mới cần giải thích.  
*Before the table, one new term needs explaining.*

**DiskANN** là một họ thuật toán ANN.  
***DiskANN** is a family of ANN algorithms.*

Nó được thiết kế để giữ phần lớn dữ liệu trên **SSD** — ổ cứng thể rắn.  
*It is designed to keep most data on **SSD**.*

Cách kia là giữ dữ liệu trong **RAM** — bộ nhớ trong.  
*The alternative is keeping data in **RAM**.*

RAM nhanh hơn nhiều.  
*RAM is much faster.*

RAM cũng đắt hơn nhiều.  
*RAM is also much more expensive.*

Trong hệ sinh thái Postgres, DiskANN đến qua extension **`pgvectorscale`** của Timescale.  
*In the Postgres ecosystem, DiskANN arrives through Timescale's **`pgvectorscale`** extension.*

| Tiêu chí / Criterion | IVFFlat | HNSW | DiskANN (pgvectorscale) |
|---|---|---|---|
| Tốc độ và recall khi query / Query speed and recall | Tốt / Good | **Rất tốt** / **Very good** | Rất tốt / Very good |
| Thời gian build index / Index build time | **Nhanh** / **Fast** | Chậm / Slow | Trung bình / Medium |
| Tiêu thụ bộ nhớ / Memory usage | **Ít** / **Low** | Nhiều (graph nằm trong RAM) / High (the graph lives in RAM) | Ít (dựa vào SSD và nén) / Low (relies on SSD and compression) |
| Cần dữ liệu sẵn để build? / Needs existing data to build? | **Có** (train k-means) / **Yes** (k-means training) | Không / No | Có / Yes |
| Độ phức tạp query / Query complexity | ~O((n/lists)·probes·d) | **~O(log n)** | ~O(log n) |
| Thêm dữ liệu mới liên tục / Continuous new data | Kém (phân hoạch lệch dần) / Poor (the partitioning drifts) | **Tốt** / **Good** | Tốt / Good |
| Hỗ trợ > 2000 chiều? / Supports > 2000 dimensions? | Không (phải dùng halfvec) / No (halfvec required) | Không (phải dùng halfvec) / No (halfvec required) | **Có (tới 16000)** / **Yes (up to 16000)** |
| Khi nào chọn / When to pick | RAM hạn chế, dữ liệu tĩnh, cần build nhanh / limited RAM, static data, fast builds needed | **mặc định 2026**: cần latency thấp và recall cao / **the 2026 default**: low latency plus high recall | dataset khổng lồ, muốn cắt chi phí RAM / huge datasets, cutting RAM cost |

> **🧩 [Ngoài bài gốc]**
>
> DiskANN là "tay chơi thứ ba".  
> *DiskANN is the "third player".*
>
> Rất ít ứng viên nhắc tới nó trong phỏng vấn.  
> *Very few candidates mention it in interviews.*
>
> Nhắc được nó là điểm cộng lớn.  
> *Mentioning it earns a big bonus.*
>
> Đây là điểm bán hàng của nó.  
> *Here is its selling point.*
>
> Nó giữ phần lớn đồ thị trên SSD.  
> *It keeps most of the graph on SSD.*
>
> Nó kết hợp **binary quantization**.  
> *It combines that with **binary quantization**.*
>
> Binary quantization nén mỗi con số xuống còn 1 bit.  
> *Binary quantization compresses each number down to one bit.*
>
> Với cùng một tập dữ liệu, index có thể nhỏ hơn HNSW gần một bậc độ lớn.  
> *On the same dataset, the index can be nearly an order of magnitude smaller than HNSW.*
>
> Nó hỗ trợ vector tới 16000 chiều mà không cần thủ thuật gì.  
> *It supports vectors up to 16000 dimensions with no tricks.*
>
> Đây là câu chốt để nói.  
> *Here is the closing line to use.*
>
> *"HNSW đánh cược rằng graph vừa RAM. Khi cược đó không còn đúng, DiskANN là câu trả lời."*  
> *"HNSW bets the graph fits in RAM. When that bet stops holding, DiskANN is the answer."*

### 3.5. Edge case: giới hạn 2000 chiều
*3.5. Edge case: the 2000-dimension limit*

**Edge case** nghĩa là trường hợp biên.  
*An **edge case** is a boundary situation.*

Đó là tình huống nằm ở rìa của điều kiện bình thường.  
*It sits at the edge of normal conditions.*

Nó ít gặp.  
*It is rare.*

Khi gặp thì nó làm hệ thống hỏng theo cách bất ngờ.  
*When it happens, it breaks the system in surprising ways.*

Người viết code tốt nghĩ tới edge case *trước khi* nó xảy ra.  
*A good programmer thinks about edge cases *before* they happen.*

Đây là cú vấp kinh điển nhất với các model hiện đại.  
*This is the most classic stumble with modern models.*

Kiểu dữ liệu `vector` của pgvector **lưu** được tới 16000 chiều.  
*The `vector` type in pgvector can **store** up to 16000 dimensions.*

Index HNSW và IVFFlat chỉ **index** được tối đa **2000 chiều**.  
*The HNSW and IVFFlat indexes can only **index** up to **2000 dimensions**.*

```sql
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
-- ERROR: column cannot have more than 2000 dimensions for hnsw index
```

Model `text-embedding-3-large` của OpenAI sinh vector 3072 chiều.  
*OpenAI's `text-embedding-3-large` model produces 3072-dimension vectors.*

Model đó dính lỗi này ngay lập tức.  
*That model hits this error immediately.*

Một staff engineer cần biết cả ba cách xử lý.  
*A staff engineer needs all three workarounds.*

**Cách 1 — `halfvec`, tức half precision, độ chính xác một nửa**  
*Option 1 — `halfvec`, meaning half precision*

Mỗi con số bình thường được lưu bằng 32 bit.  
*Each number is normally stored in 32 bits.*

`halfvec` lưu nó bằng 16 bit.  
*`halfvec` stores it in 16 bits.*

Đổi lại, bạn index được tới khoảng 4000 chiều.  
*In exchange, you can index up to about 4000 dimensions.*

Dung lượng giảm một nửa.  
*The storage size halves.*

Mất mát recall nhỏ tới mức thường không đo được trong thực tế.  
*The recall loss is so small it is usually unmeasurable in practice.*

```sql
CREATE INDEX ON docs USING hnsw ((embedding::halfvec(3072)) halfvec_cosine_ops);
--                                          ^^ dấu :: là cú pháp ép kiểu của Postgres
--                                          ^^ the :: sign is the Postgres cast syntax
```

**Cách 2 — Matryoshka và giảm chiều**  
*Option 2 — Matryoshka and dimension reduction*

**Matryoshka embeddings** được đặt theo tên búp bê gỗ Nga lồng nhau.  
***Matryoshka embeddings** are named after the nested Russian wooden dolls.*

Đây là kỹ thuật huấn luyện model theo một cách đặc biệt.  
*This is a special way of training a model.*

**Thông tin quan trọng nhất dồn về đầu vector.**  
***The most important information is packed at the front of the vector.***

Nhờ đó bạn có thể cắt bỏ phần đuôi.  
*You can therefore chop off the tail.*

Ví dụ là dùng 1024 số đầu trong 3072 số.  
*An example is using the first 1024 numbers out of 3072.*

Bạn chỉ mất rất ít chất lượng.  
*You lose very little quality.*

Các model dòng `text-embedding-3` của OpenAI hỗ trợ sẵn điều này.  
*OpenAI's `text-embedding-3` family supports this out of the box.*

> Đây là một analogy.  
> *Here is an analogy.*
>
> Nó giống ảnh JPEG chất lượng 80% thay vì 100%.  
> *It resembles a JPEG at 80% quality instead of 100%.*
>
> Ảnh đó nhẹ hơn nhiều.  
> *That image is much lighter.*
>
> Mắt thường gần như không phân biệt được.  
> *The naked eye can barely tell the difference.*

**Cách 3 — DiskANN qua `pgvectorscale`**  
*Option 3 — DiskANN through `pgvectorscale`*

Nó hỗ trợ native tới 16000 chiều.  
*It natively supports up to 16000 dimensions.*

Bạn không cần thủ thuật nào.  
*You need no tricks.*

### 3.6. Edge case: chuẩn hoá vector và giá trị NULL
*3.6. Edge case: vector normalization and NULL values*

**Chuẩn hoá (normalize) / Normalization**

Bạn có thể normalize mọi vector về độ dài 1 lúc chèn vào.  
*You may normalize every vector to length 1 at insert time.*

Mục đích là để được dùng `<#>` cho nhanh.  
*The purpose is using `<#>` for speed.*

Khi đó bạn **bắt buộc phải normalize cả vector truy vấn** lúc tìm kiếm.  
*You then **must also normalize the query vector** at search time.*

Bạn quên bước này.  
*Suppose you forget this step.*

Không có lỗi nào được báo.  
*No error is reported.*

Chỉ có kết quả sai lệch một cách âm thầm.  
*Only silently skewed results appear.*

Đây là cách phòng.  
*Here is the prevention.*

Bạn gói việc normalize vào **một hàm duy nhất**.  
*Wrap the normalization in **a single function**.*

Hàm đó dùng chung cho cả đường ghi lẫn đường đọc.  
*That function serves both the write path and the read path.*

Bạn đừng viết nó ở hai chỗ.  
*Do not write it in two places.*

**Giá trị NULL / NULL values**

`NULL` trong SQL nghĩa là "không có giá trị".  
*`NULL` in SQL means "no value".*

Những dòng có `embedding IS NULL` sẽ **không bao giờ** xuất hiện trong kết quả tìm kiếm.  
*Rows with `embedding IS NULL` will **never** appear in search results.*

Chúng vô hình.  
*They are invisible.*

Tình huống này rất hay xảy ra.  
*This situation happens often.*

Pipeline sinh embedding có thể bị lỗi ở vài dòng.  
*The embedding pipeline may fail on a few rows.*

Nguyên nhân có thể là gọi API thất bại.  
*One cause can be a failed API call.*

Nguyên nhân có thể là nội dung rỗng.  
*Another cause can be empty content.*

Nguyên nhân có thể là nội dung quá dài.  
*Another cause can be overly long content.*

Không ai để ý tới các dòng đó.  
*Nobody notices those rows.*

Đây là cách phòng.  
*Here is the prevention.*

Bạn nên coi nó là bắt buộc ở production.  
*Treat it as mandatory in production.*

```sql
-- Chạy định kỳ như một "cảm biến sức khoẻ" của hệ thống tìm kiếm
-- Run it periodically as a "health sensor" for the search system
SELECT count(*) FROM documents WHERE embedding IS NULL;
```

Con số này có thể lớn hơn 0 và đang tăng.  
*This number may be greater than zero and rising.*

Khi đó bạn có một mảng dữ liệu vô hình với người dùng.  
*You then have a slice of data invisible to your users.*

Bạn nên có job tự động tìm và sinh lại embedding cho các dòng đó.  
*You should have a job that finds those rows and regenerates their embeddings.*

### 3.7. Tự implement để hiểu tận gốc
*3.7. Implement it yourself to understand it at the root*

Cách chắc chắn nhất để thực sự hiểu một thuật toán là tự viết lại phiên bản đơn giản của nó.  
*The surest way to truly understand an algorithm is writing a simple version yourself.*

Hai đoạn dưới đây ngắn thôi.  
*The two snippets below are short.*

Chúng đủ để bạn tự tin trả lời "bên trong nó làm gì".  
*They suffice for you to confidently answer "what does it do inside".*

#### (a) Exact k-NN bằng cosine với NumPy
*(a) Exact k-NN by cosine with NumPy*

Đây chính là thứ pgvector làm khi **chưa có index**.  
*This is exactly what pgvector does **without an index**.*

```python
import numpy as np

def cosine_knn(query: np.ndarray, corpus: np.ndarray, k: int = 3):
    """Trả về (chỉ số, khoảng cách) của k vector gần nhất theo cosine distance.
    query:  mảng 1 chiều, kích thước (d,)      -> vector câu truy vấn
    corpus: mảng 2 chiều, kích thước (n, d)    -> n vector trong "kho"
    """
    # Returns (indices, distances) of the k nearest vectors by cosine distance.
    # query:  a 1-D array of shape (d,)         -> the query vector
    # corpus: a 2-D array of shape (n, d)       -> the n vectors in the "store"

    # Chuẩn hoá query về độ dài 1. np.linalg.norm chính là công thức |a| ở mục 2.1.
    # Normalize the query to length 1. np.linalg.norm is exactly the |a| formula from 2.1.
    q = query / np.linalg.norm(query)

    # Chuẩn hoá từng dòng của corpus. axis=1 nghĩa là "tính theo từng dòng";
    # Normalize each row of the corpus. axis=1 means "compute row by row";
    # keepdims=True giữ nguyên hình dạng để phép chia broadcast đúng theo dòng.
    # keepdims=True preserves the shape so the division broadcasts correctly per row.
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)

    # Sau khi cả hai đã có độ dài 1, dot product CHÍNH LÀ cosine similarity.
    # Once both have length 1, the dot product IS the cosine similarity.
    # Toán tử @ trong NumPy là phép nhân ma trận; kết quả là mảng (n,).
    # The @ operator in NumPy is matrix multiplication; the result is an (n,) array.
    sims = c @ q

    # Đổi từ similarity (lớn = giống) sang distance (nhỏ = giống), khớp với <=> của pgvector.
    # Convert similarity (large = alike) to distance (small = alike), matching pgvector's <=>.
    dists = 1.0 - sims

    # argsort trả về CHỈ SỐ sắp xếp tăng dần; lấy k cái đầu = k khoảng cách nhỏ nhất.
    # argsort returns the INDICES in ascending order; the first k are the k smallest distances.
    idx = np.argsort(dists)[:k]
    return idx, dists[idx]


# Chạy thử với đúng dữ liệu ở mục 1.5
# Try it with exactly the data from section 1.5
corpus = np.array([[2, 9, 0], [3, 8, 0], [9, 1, 0]], dtype=float)
idx, d = cosine_knn(np.array([2, 8, 0], dtype=float), corpus, k=2)
print(idx, d)   # -> [0 1] ... tức áo phao và khăn len, đúng như tính tay
                # -> [0 1] ... that is áo phao and khăn len, matching the hand calculation
```

#### (b) MiniIVF — tự dựng lại ý tưởng "chia quận"
*(b) MiniIVF — rebuilding the "split into districts" idea yourself*

```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def __init__(self, n_lists=8):
        self.n_lists = n_lists          # số "quận" sẽ chia
                                        # the number of "districts" to create

    def build(self, corpus):
        """Giai đoạn build index: chạy k-means để chia quận và tính tâm."""
        # The index build stage: run k-means to split districts and compute centers.
        self.corpus = corpus
        km = KMeans(n_clusters=self.n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_   # toạ độ tâm của mỗi quận
                                               # the center coordinates of each district
        self.assign    = km.labels_            # mỗi vector thuộc quận số mấy
                                               # which district each vector belongs to
        return self

    def search(self, query, k=3, probes=2):
        """Giai đoạn truy vấn: chỉ tìm trong `probes` quận gần nhất."""
        # The query stage: search only inside the `probes` nearest districts.

        # Bước 1: so query với các TÂM quận (rẻ: chỉ n_lists phép so, không phải n)
        # Step 1: compare the query against the district CENTERS (cheap: n_lists comparisons, not n)
        cdist = np.linalg.norm(self.centroids - query, axis=1)
        near_lists = np.argsort(cdist)[:probes]      # chọn ra `probes` quận gần nhất
                                                     # pick the `probes` nearest districts

        # Bước 2: chỉ brute-force TRONG các quận đã chọn, bỏ qua toàn bộ phần còn lại
        # Step 2: brute-force only INSIDE the chosen districts, skipping everything else
        mask = np.isin(self.assign, near_lists)      # đánh dấu các vector thuộc quận đó
                                                     # mark the vectors belonging to those districts
        cand_idx = np.where(mask)[0]                 # lấy chỉ số của chúng
                                                     # take their indices
        d = np.linalg.norm(self.corpus[cand_idx] - query, axis=1)
        order = np.argsort(d)[:k]
        return cand_idx[order], d[order]
```

**Bài tập tôi thực sự khuyên bạn làm / An exercise I genuinely recommend**

Bạn chạy `search()` với `probes=1`.  
*Run `search()` with `probes=1`.*

Sau đó bạn chạy lại với `probes=n_lists`.  
*Then run it again with `probes=n_lists`.*

Bạn so kết quả với hàm `cosine_knn` exact ở trên.  
*Compare the results with the exact `cosine_knn` function above.*

Bạn sẽ **tự mắt thấy** những hàng xóm bị bỏ sót khi `probes` nhỏ.  
*You will **see with your own eyes** the neighbors missed when `probes` is small.*

Bạn sẽ thấy chúng quay lại khi `probes` lớn.  
*You will see them return when `probes` is large.*

Đọc mười lần về trade-off recall và tốc độ không bằng nhìn thấy nó xảy ra một lần.  
*Reading about the recall-speed trade-off ten times is worth less than seeing it once.*

### 🧩 [Ngoài bài gốc] — Hai điều nữa ở tầng Advanced
*🧩 [Ngoài bài gốc] — Two more things at the Advanced tier*

**1. Build index song song và bộ nhớ / Parallel index builds and memory**

HNSW build rất chậm.  
*An HNSW build is very slow.*

Postgres có thể build song song nhiều tiến trình.  
*Postgres can build in parallel across several processes.*

Có hai tham số quyết định.  
*Two parameters decide the outcome.*

`maintenance_work_mem` là lượng RAM dành cho thao tác bảo trì như build index.  
*`maintenance_work_mem` is the RAM allotted to maintenance work such as index builds.*

Graph có thể không vừa lượng RAM này.  
*The graph may not fit in that RAM.*

Khi đó Postgres phải đổ ra đĩa.  
*Postgres must then spill to disk.*

Việc đó chậm thảm hại.  
*That is disastrously slow.*

`max_parallel_maintenance_workers` là số tiến trình song song.  
*`max_parallel_maintenance_workers` is the number of parallel processes.*

Đây là kiến thức vận hành.  
*This is operational knowledge.*

Rất ít người học lý thuyết biết tới nó.  
*Very few theory-focused learners know it.*

**2. Bảo mật cũng là chuyện của index / Security is an index concern too**

Tôi đã nói điều này ở đầu file.  
*I mentioned this at the top of the file.*

pgvector 0.8.2 vá CVE-2026-3172.  
*pgvector 0.8.2 patches CVE-2026-3172.*

Đó là lỗi tràn bộ đệm khi build HNSW song song.  
*That is a buffer overflow during parallel HNSW builds.*

Lỗi đó có thể làm rò dữ liệu từ bảng khác.  
*The bug can leak data from other tables.*

Lỗi đó cũng có thể làm sập server.  
*The bug can also crash the server.*

Đây là bài học tổng quát ở tầm staff.  
*Here is the general lesson at staff level.*

**Extension của database cũng là bề mặt tấn công.**  
***A database extension is an attack surface too.***

Việc theo dõi CVE của các extension bạn dùng là một phần của công việc.  
*Tracking CVEs for the extensions you use is part of the job.*

Đó không phải chuyện của riêng đội bảo mật.  
*It is not the security team's business alone.*

### ✅ Self-check Phần 3
*✅ Self-check for Part 3*

**Câu 1.** Nêu độ phức tạp truy vấn của HNSW và của brute-force bằng ký hiệu Big-O.  
***Question 1.** State the query complexity of HNSW and of brute force in Big-O notation.*

Ý nghĩa thực tế của sự khác biệt đó là gì?  
*What is the practical meaning of that difference?*

> *Gợi ý đáp án / Suggested answer*
>
> HNSW xấp xỉ `O(log n)`.  
> *HNSW is approximately `O(log n)`.*
>
> Brute-force là `O(n · d)`.  
> *Brute force is `O(n · d)`.*
>
> Dữ liệu tăng gấp 1000 lần.  
> *Suppose the data grows a thousandfold.*
>
> Brute-force khi đó chậm đi 1000 lần.  
> *Brute force then becomes a thousand times slower.*
>
> HNSW chỉ chậm thêm khoảng chục lần.  
> *HNSW becomes only about ten times slower.*
>
> Đó là khác biệt giữa hệ thống dùng được và không dùng được ở quy mô lớn.  
> *That is the difference between a usable and an unusable system at scale.*

**Câu 2.** Bạn có model 3072 chiều và cần đánh index HNSW. Nêu ít nhất hai cách xử lý.  
***Question 2.** You have a 3072-dimension model and need an HNSW index. Name at least two options.*

> *Gợi ý đáp án / Suggested answer*
>
> Cách 1 là ép sang `halfvec` rồi index.  
> *Option 1 is casting to `halfvec` and then indexing.*
>
> Cú pháp là `(embedding::halfvec(3072)) halfvec_cosine_ops`.  
> *The syntax is `(embedding::halfvec(3072)) halfvec_cosine_ops`.*
>
> Cách 2 là dùng Matryoshka để cắt vector xuống ví dụ 1024 chiều.  
> *Option 2 is using Matryoshka to trim the vector down to, say, 1024 dimensions.*
>
> Cách 3 là dùng DiskANN qua `pgvectorscale`.  
> *Option 3 is using DiskANN through `pgvectorscale`.*
>
> Cách đó hỗ trợ tới 16000 chiều.  
> *That option supports up to 16000 dimensions.*

**Câu 3.** Trong IVFFlat, tăng `probes` ảnh hưởng recall và latency thế nào?  
***Question 3.** In IVFFlat, how does raising `probes` affect recall and latency?*

Vì sao hàng xóm nằm sát ranh giới list hay bị bỏ sót?  
*Why are neighbors near a list boundary often missed?*

> *Gợi ý đáp án / Suggested answer*
>
> `probes` tăng thì thuật toán xét nhiều quận hơn.  
> *Raising `probes` makes the algorithm examine more districts.*
>
> Recall vì thế tăng.  
> *Recall therefore rises.*
>
> Latency cũng tăng.  
> *Latency rises as well.*
>
> Hàng xóm sát ranh giới thuộc về một quận không nằm trong nhóm được chọn.  
> *A boundary neighbor belongs to a district outside the chosen group.*
>
> Về khoảng cách thực nó rất gần điểm truy vấn.  
> *By true distance it sits very close to the query point.*
>
> Thuật toán không bao giờ nhìn tới nó.  
> *The algorithm never looks at it.*

**Câu 4.** Vì sao IVFFlat cần có dữ liệu sẵn lúc build còn HNSW thì không?  
***Question 4.** Why does IVFFlat need existing data at build time while HNSW does not?*

> *Gợi ý đáp án / Suggested answer*
>
> IVFFlat phải chạy k-means trên dữ liệu để tìm centroid.  
> *IVFFlat must run k-means over the data to find centroids.*
>
> Không có dữ liệu thì không có gì để phân cụm.  
> *Without data there is nothing to cluster.*
>
> HNSW xây đồ thị bằng cách chèn dần từng node.  
> *HNSW builds its graph by inserting one node at a time.*
>
> Mỗi node được nối với hàng xóm hiện có.  
> *Each node links to the neighbors already present.*
>
> Vậy nên đồ thị lớn lên tự nhiên cùng dữ liệu.  
> *The graph therefore grows naturally along with the data.*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL (Systems thinking and technical leadership)*

> Đây là phần phân biệt một senior engineer với một staff engineer.  
> *This part separates a senior engineer from a staff engineer.*
>
> Senior trả lời câu hỏi *"dùng cái gì và dùng thế nào"*.  
> *A senior answers *"what to use and how to use it"*.*
>
> Staff trả lời một chuỗi câu hỏi khác.  
> *A staff engineer answers a different set of questions.*
>
> *Hệ thống này sẽ gãy ở đâu khi lớn lên?*  
> *Where will this system break as it grows?*
>
> *Nó tốn bao nhiêu tiền?*  
> *How much money does it cost?*
>
> *Ai phải trực đêm vì nó?*  
> *Who has to be on call at night for it?*
>
> *Khi nào ta nên bỏ lựa chọn hiện tại?*  
> *When should we abandon the current choice?*
>
> Có vài từ vựng nghề nghiệp cần giải thích trước.  
> *A few professional terms need explaining first.*
>
> **Staff Engineer** là cấp bậc kỹ sư cao hơn senior ở các công ty công nghệ lớn.  
> ***Staff Engineer** is a rank above senior at large tech companies.*
>
> Đặc trưng của nó không phải là code giỏi hơn.  
> *Its hallmark is not better coding.*
>
> Đặc trưng của nó là **phạm vi ảnh hưởng**.  
> *Its hallmark is **scope of influence**.*
>
> Phạm vi đó gồm quyết định kiến trúc và đánh đổi dài hạn.  
> *That scope covers architecture decisions and long-term trade-offs.*
>
> Phạm vi đó ảnh hưởng tới nhiều team.  
> *That scope reaches across many teams.*
>
> **Production** là môi trường chạy thật, phục vụ người dùng thật.  
> ***Production** is the live environment serving real users.*
>
> Nó đối lập với môi trường dev và staging.  
> *It contrasts with the dev and staging environments.*
>
> Hai môi trường đó dùng để thử nghiệm.  
> *Those two environments exist for experimentation.*
>
> **SLA** viết tắt của *Service Level Agreement*, tức cam kết mức dịch vụ.  
> ***SLA** stands for *Service Level Agreement*.*
>
> Đây là lời hứa đo được về chất lượng.  
> *This is a measurable promise about quality.*
>
> Ví dụ là "95% truy vấn phải trả lời dưới 100 mili-giây".  
> *An example is "95% of queries must answer under 100 milliseconds".*
>
> **On-call** là chế độ trực.  
> ***On-call** is the duty rotation.*
>
> Hệ thống có sự cố ngoài giờ.  
> *An incident happens outside working hours.*
>
> Khi đó ai đó bị điện thoại đánh thức lúc 3 giờ sáng.  
> *Someone then gets woken by a phone call at 3 a.m.*
>
> Mỗi hệ thống bạn thêm vào là thêm gánh nặng on-call cho một con người thật.  
> *Every system you add loads more on-call burden onto a real human being.*

### 4.1. Scale: từ 1 triệu tới hàng tỷ vector
*4.1. Scale: from 1 million to billions of vectors*

**Scale** nghĩa là mở rộng quy mô.  
***Scale** means growing in size.*

Câu hỏi của nó rất rõ ràng.  
*Its question is very clear.*

Hệ thống hoạt động ra sao khi dữ liệu và lưu lượng tăng lên nhiều lần?  
*How does the system behave when data and traffic multiply?*

Câu hỏi cốt lõi ở tầm staff **không phải** "dùng index nào".  
*The core staff-level question is **not** "which index to use".*

Câu hỏi đó là **"hệ thống này gãy ở đâu trước tiên khi lớn lên?"**  
*The question is **"where does this system break first as it grows?"***

#### Dưới 10 triệu vector — đừng phức tạp hoá
*Under 10 million vectors — do not overcomplicate*

Một node PostgreSQL duy nhất với index HNSW là **đủ**.  
*A single PostgreSQL node with an HNSW index is **enough**.*

Nó thường là lựa chọn tốt nhất xét về tổng thể vận hành.  
*It is usually the best choice on overall operations.*

Truy vấn trả về trong khoảng vài mili-giây.  
*Queries return in a few milliseconds.*

Đây là khoảng 90% các use case RAG và tìm kiếm nội bộ doanh nghiệp.  
*This covers about 90% of real RAG and internal enterprise search use cases.*

Đây là lời khuyên staff thẳng thắn.  
*Here is blunt staff-level advice.*

**Đừng over-engineer.**  
***Do not over-engineer.***

Over-engineer nghĩa là thiết kế thừa cho quy mô mà bạn chưa có.  
*Over-engineering means designing for a scale you do not have yet.*

Bạn mới có một triệu vector.  
*You have only a million vectors.*

Bạn dựng sẵn kiến trúc cho một tỷ vector.  
*You build architecture for a billion vectors.*

Chi phí lớn nhất của việc đó không phải tiền server.  
*The biggest cost of that is not server money.*

Chi phí lớn nhất là **thời gian và độ phức tạp**.  
*The biggest cost is **time and complexity**.*

Đó là thứ khiến team đi chậm lại suốt nhiều năm.  
*That is what slows a team down for years.*

#### Bottleneck xuất hiện ở đâu — hãy tính bằng con số
*Where the bottleneck appears — compute it in numbers*

**Bottleneck** nghĩa là điểm nghẽn.  
*A **bottleneck** is a choke point.*

Đó là bộ phận chạm giới hạn trước tiên.  
*It is the component that hits its limit first.*

Nó kéo cả hệ thống chậm lại.  
*It drags the whole system down.*

Tìm đúng bottleneck quan trọng hơn tối ưu mọi thứ.  
*Finding the right bottleneck matters more than optimizing everything.*

Với vector search, bottleneck số một hầu như luôn là **RAM**.  
*For vector search, the number one bottleneck is almost always **RAM**.*

Lý do là đồ thị HNSW phải nằm gọn trong RAM thì mới nhanh.  
*The reason is that the HNSW graph must fit in RAM to stay fast.*

Hãy tính thử.  
*Let us compute it.*

```
Mỗi con số trong vector      = 4 byte (kiểu float 32 bit)
Each number in a vector      = 4 bytes (32-bit float type)
Một vector 1536 chiều        = 1536 × 4 = 6.144 byte ≈ 6 KB
One 1536-dimension vector    = 1536 × 4 = 6,144 bytes ≈ 6 KB
1 triệu vector               = 6 GB
1 million vectors            = 6 GB
100 triệu vector             = 600 GB   ← chỉ riêng dữ liệu thô, CHƯA tính đồ thị HNSW!
100 million vectors          = 600 GB   ← raw data only, NOT counting the HNSW graph!
```

Đồ thị HNSW còn ngốn thêm đáng kể nữa cho các liên kết.  
*The HNSW graph consumes considerably more for its links.*

Con số này có thể vượt quá RAM của máy.  
*This number may exceed the machine's RAM.*

Khi đó hệ điều hành phải đọc từ đĩa.  
*The operating system must then read from disk.*

**Tail latency** — độ trễ ở nhóm chậm nhất — phình lên khủng khiếp.  
***Tail latency** then balloons horribly.*

Đó chính là thời điểm bạn phải nghĩ tới **nén vector**.  
*That is exactly when you must think about **vector compression**.*

Bạn xem mục 4.2 để biết chi tiết.  
*See section 4.2 for details.*

Bạn cũng có thể nghĩ tới **DiskANN**.  
*You may also think about **DiskANN**.*

#### Bottleneck thứ hai: thời gian build index
*The second bottleneck: index build time*

HNSW build là `O(n · log n)`.  
*An HNSW build is `O(n · log n)`.*

Nó có thể tốn hàng giờ ở quy mô lớn.  
*It can take hours at large scale.*

Có ba cách tăng tốc.  
*There are three ways to speed it up.*

1. **Nạp toàn bộ dữ liệu xong rồi mới build index.**  
   *1. **Load all the data first, then build the index.***

   Cách này luôn nhanh hơn build trước rồi chèn dần.  
   *This is always faster than building first and inserting gradually.*

2. **Tăng `maintenance_work_mem`.**  
   *2. **Raise `maintenance_work_mem`.***

   Đây là lượng RAM mà Postgres được phép dùng cho các thao tác bảo trì.  
   *This is the RAM Postgres may use for maintenance operations.*

   Build index là một thao tác như vậy.  
   *An index build is one such operation.*

   Khuyến nghị là đặt nó đủ lớn để chứa **working set**.  
   *The recommendation is setting it large enough to hold the **working set**.*

   Working set là phần dữ liệu đang thực sự được đụng tới trong lúc build.  
   *The working set is the data actually touched during the build.*

   Bạn đừng vượt quá 50 tới 60% RAM của máy.  
   *Do not exceed 50 to 60% of the machine's RAM.*

   Vượt quá thì hệ điều hành hết chỗ thở.  
   *Beyond that the operating system runs out of breathing room.*

   Khi đó mọi thứ chậm lại.  
   *Everything then slows down.*

3. **Tăng `max_parallel_maintenance_workers`.**  
   *3. **Raise `max_parallel_maintenance_workers`.***

   Đây là số tiến trình build song song.  
   *This is the number of parallel build processes.*

#### Vertical scaling — làm trước, vì nó dễ
*Vertical scaling — do it first, because it is easy*

**Vertical scaling** nghĩa là mở rộng theo chiều dọc.  
***Vertical scaling** means growing upward.*

Bạn làm cho **một máy** mạnh hơn.  
*You make **one machine** stronger.*

Bạn thêm RAM.  
*You add RAM.*

Bạn thêm CPU.  
*You add CPU.*

Bạn đổi sang SSD nhanh hơn.  
*You swap in a faster SSD.*

Đây là thứ nên làm trước tiên.  
*This is what you should do first.*

Nó đi xa hơn nhiều người tưởng.  
*It goes further than most people assume.*

Máy cloud hiện nay có thể có vài TB RAM.  
*Today's cloud machines can have several TB of RAM.*

Cách này đơn giản.  
*This approach is simple.*

Nó không thay đổi kiến trúc.  
*It does not change the architecture.*

Nó không sinh bug mới.  
*It creates no new bugs.*

#### Horizontal scaling — khi hết đường vertical
*Horizontal scaling — when vertical runs out*

**Horizontal scaling** nghĩa là mở rộng theo chiều ngang.  
***Horizontal scaling** means growing outward.*

Bạn **thêm nhiều máy**.  
*You **add more machines**.*

Cách này phức tạp hơn hẳn.  
*This approach is considerably more complex.*

Bạn chỉ làm nó khi thật sự cần.  
*Do it only when you truly need to.*

Có ba công cụ chính.  
*There are three main tools.*

**a) Read replicas — bản sao chỉ đọc**

Đây là các máy giữ bản sao dữ liệu.  
*These are machines holding a copy of the data.*

Chúng chỉ phục vụ truy vấn đọc.  
*They serve read queries only.*

Tìm kiếm là công việc **read-heavy**.  
*Search is a **read-heavy** workload.*

Read-heavy nghĩa là chủ yếu đọc, ít ghi.  
*Read-heavy means mostly reads and few writes.*

Vậy nên cách này hiệu quả tuyệt vời.  
*This approach is therefore excellent.*

Bạn cần chịu tải gấp 5 lần.  
*Suppose you need five times the load capacity.*

Khi đó bạn thêm 5 replica.  
*You then add five replicas.*

**b) Partitioning — phân mảnh bảng**

Bạn chia một bảng lớn thành nhiều bảng con.  
*You split one large table into several child tables.*

Các bảng con nằm bên trong *cùng một* máy chủ.  
*The child tables live inside the *same* server.*

Việc chia tuân theo một quy tắc.  
*The split follows a rule.*

Đây là điểm mà staff engineer nói khác người mới.  
*Here is where a staff engineer differs from a beginner.*

**Hãy phân mảnh theo *cách dữ liệu được truy cập*.**  
***Partition by *how the data is accessed*.***

Bạn đừng phân mảnh theo kích thước.  
*Do not partition by size.*

Mọi truy vấn của bạn có thể đều có `WHERE tenant_id = ?`.  
*All your queries may carry `WHERE tenant_id = ?`.*

Khi đó hãy phân mảnh theo `tenant_id`.  
*Then partition by `tenant_id`.*

Postgres có thể **prune** toàn bộ các mảnh không liên quan.  
*Postgres can then **prune** every irrelevant partition.*

Prune nghĩa là cắt bỏ.  
*Pruning means cutting them away.*

Việc cắt bỏ diễn ra *trước khi* tìm kiếm.  
*The pruning happens *before* the search.*

Bạn xây index vector riêng cho từng mảnh.  
*You build a separate vector index per partition.*

Index đó nhỏ hơn.  
*That index is smaller.*

Index đó nằm gọn trong RAM.  
*That index fits in RAM.*

Index đó nhanh hơn nhiều.  
*That index is much faster.*

> Đây là một ghi chú kỹ thuật.  
> *Here is a technical note.*
>
> Một bảng không phân mảnh trong Postgres có giới hạn mặc định 32 TB.  
> *An unpartitioned table in Postgres has a default limit of 32 TB.*
>
> Bảng đã phân mảnh thì gần như không giới hạn.  
> *A partitioned table is nearly unlimited.*

**c) Sharding — phân mảnh qua nhiều máy**

Bạn chia dữ liệu ra **nhiều máy chủ khác nhau**.  
*You split the data across **several different servers**.*

Đây là bước phức tạp nhất.  
*This is the most complex step.*

Công cụ gồm **Citus**, **PgDog** và **pgvectorscale**.  
*The tools include **Citus**, **PgDog**, and **pgvectorscale**.*

Citus là extension biến Postgres thành cụm phân tán.  
*Citus is an extension turning Postgres into a distributed cluster.*

Bạn chỉ làm bước này khi một máy không còn chứa nổi dữ liệu.  
*Take this step only when one machine can no longer hold the data.*

### 4.2. 🧩 [Ngoài bài gốc] Quantization — đòn bẩy chi phí lớn nhất
*4.2. 🧩 [Ngoài bài gốc] Quantization — the biggest cost lever*

**Quantization** nghĩa là lượng tử hoá.  
***Quantization** means quantizing.*

Nó **giảm số bit dùng để lưu mỗi con số**.  
*It **reduces the number of bits used to store each number**.*

Bạn chấp nhận mất một chút độ chính xác.  
*You accept a small loss of accuracy.*

Đổi lại bạn có dung lượng nhỏ hơn nhiều.  
*In exchange you get far smaller storage.*

> Đây là một analogy.  
> *Here is an analogy.*
>
> Nó giống như lưu ảnh dưới dạng JPEG thay vì RAW.  
> *It resembles saving an image as JPEG instead of RAW.*
>
> File nhỏ đi hàng chục lần.  
> *The file shrinks by tens of times.*
>
> Mắt thường gần như không thấy khác biệt.  
> *The naked eye barely sees a difference.*
>
> Bạn phóng to soi kỹ thì đúng là có mất chi tiết.  
> *Zoom in closely and detail is indeed lost.*

| Kỹ thuật / Technique | Cắt được bao nhiêu dung lượng / Storage saved | Ảnh hưởng tới recall / Effect on recall |
|---|---|---|
| `halfvec` (16 bit thay vì 32 bit) / `halfvec` (16 bits instead of 32) | ~50% | rất nhỏ, thường không đo thấy / very small, usually unmeasurable |
| Binary quantization (1 bit mỗi số) / Binary quantization (1 bit per number) | tới ~32 lần / up to ~32 times | đáng kể, phải bù bằng bước re-rank / significant, offset by a re-rank step |
| Matryoshka (cắt bớt số chiều) / Matryoshka (trimming dimensions) | tuỳ tỷ lệ cắt / depends on the trim ratio | nhỏ, nếu model có hỗ trợ / small, when the model supports it |

#### Pattern kinh điển: two-stage retrieval (thô trước, tinh sau)
*A classic pattern: two-stage retrieval (coarse first, fine second)*

Đây là mô hình thiết kế mà bạn nên biết.  
*This is a design pattern worth knowing.*

Nó xuất hiện ở khắp nơi trong ngành tìm kiếm.  
*It appears everywhere in the search industry.*

**Giai đoạn 1 — recall, tức quét rộng / Stage 1 — recall, the wide sweep**

Bạn dùng index đã nén để lấy nhanh top 200 ứng viên.  
*You use a compressed index to grab the top 200 candidates fast.*

Ví dụ là index dùng binary quantization.  
*An example is an index using binary quantization.*

Bước này rẻ và nhanh.  
*This step is cheap and fast.*

Bước này hơi thô.  
*This step is a bit coarse.*

Mục tiêu duy nhất của nó là *đừng bỏ sót* thứ tốt.  
*Its only goal is *not missing* the good items.*

**Giai đoạn 2 — precision, tức lọc tinh / Stage 2 — precision, the fine filter**

Với 200 ứng viên đó, bạn tính lại khoảng cách bằng vector gốc đầy đủ.  
*For those 200 candidates, you recompute distances with the full original vectors.*

Bạn trộn thêm các tín hiệu nghiệp vụ.  
*You blend in business signals.*

Các tín hiệu đó gồm độ mới và độ phổ biến.  
*Those signals include recency and popularity.*

Chúng cũng gồm quyền truy cập của người dùng và tồn kho.  
*They also include user permissions and stock levels.*

Bạn chọn ra top 10 cuối cùng.  
*You pick the final top 10.*

Chỉ có 200 ứng viên nên bước này rất rẻ.  
*With only 200 candidates, this step is very cheap.*

Nó rẻ dù tính toán hoàn toàn chính xác.  
*It stays cheap despite fully exact computation.*

Kết quả rất đáng chú ý.  
*The result is remarkable.*

Chi phí gần bằng giai đoạn 1.  
*The cost is close to stage 1 alone.*

Chất lượng gần bằng exact search.  
*The quality is close to exact search.*

Đó là lý do mô hình này thắng thế.  
*That is why this pattern won.*

Đây là một mẹo triển khai.  
*Here is an implementation tip.*

**Hãy làm cả hai bước ngay trong một câu SQL.**  
***Do both steps inside a single SQL statement.***

Cách này giữ tính nhất quán.  
*This preserves consistency.*

Cách này tránh thêm round-trip.  
*This avoids an extra round-trip.*

### 4.3. Chi phí, độ trễ, độ tin cậy, giám sát
*4.3. Cost, latency, reliability, monitoring*

#### Cost (chi phí)
*Cost*

Postgres có sẵn pgvector trên hầu hết dịch vụ quản lý.  
*pgvector is available on most managed Postgres services.*

Ví dụ là Supabase, Neon, AWS RDS/Aurora và Google Cloud SQL.  
*Examples are Supabase, Neon, AWS RDS/Aurora, and Google Cloud SQL.*

Giá khởi điểm từ vài chục USD mỗi tháng.  
*Prices start from a few dozen USD per month.*

So với việc thêm một vector database chuyên dụng, bạn tiết kiệm nhiều thứ.  
*Compared with adding a dedicated vector database, you save on many fronts.*

Bạn không chỉ tiết kiệm hoá đơn hàng tháng.  
*You save more than the monthly bill.*

Giá trị lớn nhất nằm ở một điều khác.  
*The biggest value lies elsewhere.*

Đây là lập luận mà staff engineer đưa ra.  
*This is the argument a staff engineer makes.*

Ít ai khác nghĩ tới nó.  
*Few others think of it.*

**Bạn không thêm một hệ thống mới vào bức tranh.**  
***You do not add a new system to the picture.***

Mỗi hệ thống mới là một **failure domain** mới.  
*Every new system is a new **failure domain**.*

Failure domain là một chỗ nữa có thể sập độc lập.  
*A failure domain is one more place that can fail on its own.*

Mỗi hệ thống mới là một bề mặt on-call mới.  
*Every new system is a new on-call surface.*

Mỗi hệ thống mới là một thứ nữa phải backup.  
*Every new system is one more thing to back up.*

Mỗi hệ thống mới là một thứ nữa phải vá bảo mật.  
*Every new system is one more thing to patch.*

Mỗi hệ thống mới là một thứ nữa phải đào tạo người mới.  
*Every new system is one more thing to train newcomers on.*

Những chi phí này không nằm trên hoá đơn.  
*These costs do not appear on the invoice.*

Chúng có thật.  
*They are real.*

Chúng rất lớn.  
*They are very large.*

#### Latency (độ trễ) — và vì sao đừng bao giờ đo trung bình
*Latency — and why you should never measure the average*

**Latency** là thời gian từ lúc gửi truy vấn tới lúc nhận được kết quả.  
***Latency** is the time from sending a query to receiving the result.*

Hãy đo bằng **percentile** — bách phân vị.  
*Measure it with **percentiles**.*

Bạn đừng đo trung bình.  
*Do not measure the average.*

**p50** là giá trị mà 50% truy vấn nhanh hơn nó.  
***p50** is the value that 50% of queries beat.*

Đó chính là trung vị.  
*That is exactly the median.*

**p95** là giá trị mà 95% truy vấn nhanh hơn nó.  
***p95** is the value that 95% of queries beat.*

5% truy vấn chậm hơn nó.  
*Five percent of queries are slower.*

**p99** là giá trị mà 99% truy vấn nhanh hơn nó.  
***p99** is the value that 99% of queries beat.*

1% truy vấn chậm hơn nó.  
*One percent of queries are slower.*

Đây là nhóm "tail" — đuôi.  
*This is the "tail" group.*

Vì sao trung bình lừa dối bạn?  
*Why does the average deceive you?*

Giả sử 99 truy vấn chạy 10ms.  
*Suppose 99 queries take 10ms.*

Giả sử 1 truy vấn chạy 5 giây.  
*Suppose one query takes 5 seconds.*

Trung bình khi đó là khoảng 60ms.  
*The average is then about 60ms.*

Con số đó nghe rất ổn.  
*That number sounds fine.*

Có một người dùng thật đã chờ 5 giây.  
*A real user waited 5 seconds.*

Người đó có thể đã bỏ đi.  
*That person may have left.*

Hệ thống của bạn có hàng triệu truy vấn mỗi ngày.  
*Your system handles millions of queries per day.*

Khi đó "1% chậm" nghĩa là hàng chục nghìn người mỗi ngày.  
*Then "1% slow" means tens of thousands of people per day.*

**Trung bình che giấu nỗi đau.**  
***The average hides the pain.***

**Percentile phơi bày nó.**  
***The percentile exposes it.***

Với HNSW, tail latency phình lên khi `ef_search` quá cao.  
*With HNSW, tail latency balloons when `ef_search` is too high.*

Nó cũng phình lên khi đồ thị không còn vừa RAM.  
*It also balloons when the graph no longer fits in RAM.*

#### Monitoring recall — điểm phân biệt staff rõ nhất
*Monitoring recall — the clearest staff-level differentiator*

Đây là ý quan trọng nhất của cả Phần 4.  
*This is the most important idea in all of Part 4.*

Vậy nên tôi nói thật kỹ.  
*I will therefore spell it out carefully.*

Với ANN, **không có lỗi biên dịch** khi chất lượng tìm kiếm suy giảm.  
*With ANN, there is **no compile error** when search quality degrades.*

Không có exception nào.  
*There is no exception.*

Không có alert nào.  
*There is no alert.*

Hệ thống vẫn chạy.  
*The system keeps running.*

Hệ thống vẫn trả kết quả.  
*The system keeps returning results.*

Hệ thống vẫn nhanh.  
*The system stays fast.*

Chỉ là kết quả ngày càng kém liên quan.  
*The results merely grow less and less relevant.*

Người dùng không báo lỗi.  
*Users file no bug reports.*

Họ chỉ lặng lẽ dùng ít đi.  
*They just quietly use it less.*

Cho nên bạn **phải chủ động đo recall**.  
*You must therefore **measure recall proactively**.*

Cách đo là so kết quả có index với kết quả exact.  
*The method is comparing indexed results against exact results.*

Kết quả exact được dùng làm **ground truth**.  
*The exact results serve as the **ground truth**.*

Ground truth nghĩa là "sự thật nền" để đối chiếu.  
*Ground truth is the baseline truth for comparison.*

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép Postgres KHÔNG dùng index -> buộc phải exact search.
                                    -- force Postgres NOT to use the index -> exact search only.
                                    -- Kết quả này là ground truth (đáp án đúng tuyệt đối).
                                    -- This result is the ground truth (the absolutely correct answer).
SELECT id FROM documents ORDER BY embedding <=> :q LIMIT 10;
COMMIT;

-- Sau đó chạy lại đúng query đó với index bật bình thường,
-- Then run that same query again with the index enabled as usual,
-- rồi tính tỷ lệ trùng nhau giữa hai danh sách ID: đó là recall@10.
-- then compute the overlap ratio between the two ID lists: that is recall@10.
```

Đây là cách làm ở production.  
*Here is the production approach.*

Bạn lấy mẫu vài trăm truy vấn thật mỗi ngày.  
*You sample a few hundred real queries per day.*

Bạn chạy đối chiếu như trên trên một replica.  
*You run the comparison above on a replica.*

Mục đích là để không ảnh hưởng người dùng.  
*The purpose is avoiding impact on users.*

Bạn vẽ **recall@10 theo thời gian** lên dashboard.  
*You plot **recall@10 over time** on a dashboard.*

Đường đó có thể đi xuống.  
*That line may trend downward.*

Khi đó bạn biết mình cần tune lại hoặc reindex.  
*You then know you must retune or reindex.*

Bạn biết điều đó **trước khi** kinh doanh cảm nhận được.  
*You know it **before** the business feels it.*

Đây là thứ tôi khuyên bạn nói ra trong phỏng vấn.  
*This is something I recommend saying in an interview.*

Rất ít ứng viên nhắc tới nó.  
*Very few candidates mention it.*

Nó cho thấy bạn từng vận hành thật.  
*It shows you have actually operated a system.*

Nó cho thấy bạn không chỉ đọc tài liệu.  
*It shows you did more than read the docs.*

#### Failure modes — các kiểu hỏng cần chuẩn bị trước
*Failure modes — the breakages to prepare for*

**Failure mode** nghĩa là "cách mà hệ thống hỏng".  
*A **failure mode** is "the way a system breaks".*

Liệt kê trước các kiểu hỏng là công việc thiết kế.  
*Listing failure modes in advance is design work.*

Đó không phải sự bi quan.  
*It is not pessimism.*

1. **Đổi phiên bản embedding model.**  
   *1. **Changing the embedding model version.***

   Vector cũ và vector mới nằm trên hai không gian khác nhau.  
   *Old and new vectors live in two different spaces.*

   Đem so với nhau vẫn ra số.  
   *Comparing them still produces a number.*

   Kết quả là rác.  
   *The result is garbage.*

   Không có lỗi nào được báo.  
   *No error is reported.*

   Đây là failure mode nguy hiểm nhất trong danh sách này.  
   *This is the most dangerous failure mode on this list.*

2. **`VACUUM` trên bảng có index HNSW rất chậm.**  
   *2. **`VACUUM` on a table with an HNSW index is very slow.***

   `VACUUM` là tiến trình dọn dẹp của Postgres.  
   *`VACUUM` is the cleanup process of Postgres.*

   Nó thu hồi không gian của các dòng đã xoá.  
   *It reclaims the space of deleted rows.*

   Với HNSW nó có thể kéo dài rất lâu.  
   *With HNSW it can drag on for a very long time.*

   Cách xử lý thường dùng là reindex trước rồi mới vacuum.  
   *The common workaround is reindexing first, then vacuuming.*

3. **Pipeline sinh embedding lỗi âm thầm.**  
   *3. **The embedding pipeline fails silently.***

   Nó để lại các dòng `NULL` hoặc vector rác.  
   *It leaves behind `NULL` rows or garbage vectors.*

   Bạn xem mục 3.6 để biết chi tiết.  
   *See section 3.6 for details.*

4. **`SET` toàn cục rò rỉ qua connection pooler.**  
   *4. **A global `SET` leaks through the connection pooler.***

   Bạn xem mục 2.3 để biết chi tiết.  
   *See section 2.3 for details.*

5. **Lỗ hổng bảo mật của chính extension.**  
   *5. **A security hole in the extension itself.***

   Ví dụ là CVE-2026-3172 trong pgvector 0.8.0 và 0.8.1.  
   *An example is CVE-2026-3172 in pgvector 0.8.0 and 0.8.1.*

#### Ops (công cụ vận hành)
*Ops (operational tooling)*

**`pg_stat_statements`** là extension thống kê các câu query.  
***`pg_stat_statements`** is an extension that gathers query statistics.*

Nó cho biết query nào chạy nhiều nhất và tốn nhất.  
*It tells you which queries run most often and cost most.*

Bật nó lên là việc đầu tiên trên mọi Postgres production.  
*Enabling it is the first task on every production Postgres.*

**PgHero** là giao diện web xem nhanh tình trạng sức khoẻ database.  
***PgHero** is a web interface for a quick look at database health.*

### 4.4. Khi nào NÊN và khi nào KHÔNG NÊN dùng pgvector
*4.4. When you SHOULD and when you SHOULD NOT use pgvector*

Biết giới hạn của lựa chọn của chính mình là dấu hiệu rõ nhất của tư duy staff.  
*Knowing the limits of your own choice is the clearest sign of staff thinking.*

Người mới bảo vệ công cụ mình thích.  
*A beginner defends the tool they like.*

Người có kinh nghiệm nói rõ khi nào công cụ đó là lựa chọn sai.  
*An experienced person states when that tool is the wrong choice.*

**NÊN dùng pgvector khi / You SHOULD use pgvector when**

Vector search là **một tính năng bên trong** một ứng dụng vốn đã chạy Postgres.  
*Vector search is **a feature inside** an application already running Postgres.*

Bạn cần **lọc và join** vector với dữ liệu nghiệp vụ.  
*You need to **filter and join** vectors against business data.*

Đây là lý do mạnh nhất.  
*This is the strongest reason.*

Quy mô dưới khoảng 10 tới 50 triệu vector.  
*The scale is under roughly 10 to 50 million vectors.*

Bạn muốn ít hệ thống.  
*You want fewer systems.*

Bạn muốn ít chi phí vận hành.  
*You want lower operational cost.*

Bạn cần đảm bảo **ACID transaction**.  
*You need **ACID transaction** guarantees.*

ACID là nhóm tính chất đảm bảo dữ liệu luôn nhất quán kể cả khi có sự cố.  
*ACID is the set of properties keeping data consistent even during incidents.*

Vector và dữ liệu nghiệp vụ được ghi cùng lúc hoặc cùng không được ghi.  
*The vector and the business data are written together or not at all.*

**KHÔNG NÊN dùng pgvector khi / You should NOT use pgvector when**

Bạn hãy cân nhắc Pinecone, Weaviate, Qdrant, Milvus hoặc Vespa trong các trường hợp sau.  
*Consider Pinecone, Weaviate, Qdrant, Milvus, or Vespa in the following cases.*

Quy mô vượt 50 triệu tới hàng tỷ vector.  
*The scale exceeds 50 million up to billions of vectors.*

Đồng thời bạn cần phân tán trên nhiều vùng địa lý.  
*At the same time you need distribution across many geographic regions.*

Bạn cần **hybrid search** mạnh sẵn có.  
*You need strong built-in **hybrid search**.*

Hybrid search là kết hợp tìm theo vector với tìm theo từ khoá truyền thống.  
*Hybrid search combines vector search with traditional keyword search.*

Tìm theo từ khoá ở đây là BM25.  
*The keyword search here is BM25.*

Hệ thống trộn điểm số của hai bên lại.  
*The system blends the scores of both sides.*

Tổ chức của bạn vốn không dùng Postgres.  
*Your organization does not use Postgres at all.*

**Vector search chính là workload chính của sản phẩm.**  
***Vector search is the product's main workload.***

Nó không phải một tính năng phụ.  
*It is not a side feature.*

Khi đó một hệ chuyên dụng sẽ tối ưu tốt hơn ở mọi khía cạnh.  
*A dedicated system then optimizes better on every axis.*

> **Câu chốt tư duy nên thuộc lòng — a closing line worth memorizing**
>
> pgvector thắng ở sự đơn giản trong vận hành.  
> *pgvector wins on operational simplicity.*
>
> pgvector không thắng ở hiệu năng đỉnh.  
> *pgvector does not win on peak performance.*
>
> Hãy chọn nó khi vector search là một tính năng.  
> *Choose it when vector search is a feature.*
>
> Đừng chọn nó khi vector search là cả sản phẩm.  
> *Do not choose it when vector search is the whole product.*

### 4.5. Ảnh hưởng tổ chức & cách nói với người không kỹ thuật
*4.5. Organizational impact and how to speak to non-technical people*

Một staff engineer phải giải thích được quyết định kỹ thuật cho **stakeholder**.  
*A staff engineer must explain technical decisions to **stakeholders**.*

Stakeholder là các bên liên quan.  
*Stakeholders are the interested parties.*

Ví dụ là quản lý sản phẩm, sếp và phòng tài chính.  
*Examples are product managers, bosses, and the finance department.*

Bạn phải nói bằng ngôn ngữ của *họ*.  
*You must speak *their* language.*

Ngôn ngữ đó là **rủi ro và chi phí**.  
*That language is **risk and cost**.*

Ngôn ngữ đó không phải là thuật ngữ kỹ thuật.  
*That language is not technical jargon.*

**Ví dụ cách nói với PM hoặc sếp / A sample way to speak to a PM or a boss**

> "Chúng ta không cần mua thêm một database mới.  
> *"We do not need to buy another database.*
>
> Chúng ta cũng không cần vận hành thêm một database mới.  
> *We also do not need to operate another database.*
>
> Ta tận dụng hệ thống Postgres sẵn có.  
> *We reuse the Postgres system we already have.*
>
> Đội ngũ hiện tại đã biết cách vận hành nó.  
> *The current team already knows how to run it.*
>
> Ta không phải tuyển thêm người.  
> *We do not have to hire more people.*
>
> Ta không thêm một chỗ nữa có thể sập lúc nửa đêm.  
> *We do not add one more thing that can crash at midnight.*
>
> Chi phí thấp hơn.  
> *The cost is lower.*
>
> Rủi ro cũng thấp hơn.  
> *The risk is lower too.*
>
> Sau này ta có thể vượt khoảng 50 triệu bản ghi.  
> *Later we may exceed roughly 50 million records.*
>
> Ta cũng có thể cần phục vụ nhiều châu lục.  
> *We may also need to serve several continents.*
>
> Khi đó chúng ta sẽ phải tính chuyện chuyển đổi.  
> *We will then have to consider a migration.*
>
> Đó là bài toán của thành công.  
> *That is a problem of success.*
>
> Ta giải nó khi tới đó."  
> *We will solve it when we get there."*

Hãy chú ý cách nói trên.  
*Notice the wording above.*

Nó hoàn toàn không có chữ HNSW.  
*It contains no mention of HNSW.*

Nó không có chữ recall.  
*It contains no mention of recall.*

Nó cũng không có chữ embedding.  
*It contains no mention of embedding either.*

Nó nói bằng ba thứ mà người nghe quan tâm.  
*It speaks in the three things the listener cares about.*

Ba thứ đó là **tiền, rủi ro và tốc độ ra sản phẩm**.  
*Those three are **money, risk, and speed to market**.*

Kèm theo là một lời thừa nhận trung thực về giới hạn.  
*It also carries an honest admission of the limits.*

Điều này xây dựng lòng tin tốt hơn mọi lời hứa hẹn.  
*This builds trust better than any promise.*

**Ảnh hưởng tới roadmap — impact on the product roadmap**

Quyết định *chọn embedding model nào* là quyết định kiến trúc **nặng hơn** cả quyết định chọn index.  
*The choice of embedding model is a **heavier** architectural decision than the index choice.*

Đây là lý do.  
*Here is the reason.*

Đổi model đồng nghĩa với việc phải sinh lại embedding cho **toàn bộ** kho dữ liệu.  
*Changing the model means regenerating embeddings for **the entire** data store.*

Việc đó tốn tiền gọi API.  
*That costs API money.*

Việc đó tốn thời gian.  
*That costs time.*

Việc đó có thể buộc bạn ngừng dịch vụ.  
*That may force you to take the service down.*

Đây là việc staff cần làm sớm.  
*Here is what a staff engineer must do early.*

1. Chốt model càng sớm càng tốt.  
   *1. Lock in the model as early as possible.*

   Bạn chốt sau khi đã thử nghiệm đàng hoàng.  
   *You lock it in after proper experimentation.*

2. **Version hoá** cột embedding.  
   *2. **Version** the embedding column.*

   Ví dụ là thêm cột `embedding_v2` bên cạnh `embedding`.  
   *An example is adding an `embedding_v2` column next to `embedding`.*

   Nhờ vậy bạn chuyển đổi dần được.  
   *This lets you migrate gradually.*

3. Có kế hoạch **backfill**.  
   *3. Have a **backfill** plan.*

   Backfill là sinh lại dữ liệu cũ theo cách mới.  
   *Backfilling means regenerating old data the new way.*

   Có kế hoạch **blue-green deployment**.  
   *Have a **blue-green deployment** plan.*

   Blue-green nghĩa là chạy song song hai phiên bản.  
   *Blue-green means running two versions side by side.*

   Bạn chuyển lưu lượng dần từ cũ sang mới.  
   *You shift traffic gradually from old to new.*

   Nhờ vậy bạn quay đầu được nếu hỏng.  
   *This lets you roll back if things break.*

**Team topology — cấu trúc đội ngũ**

Giữ vector trong Postgres nghĩa là đội backend hiện tại tự lo được toàn bộ.  
*Keeping vectors in Postgres means the current backend team handles everything.*

Bạn không cần tuyển người biết một stack vector database riêng.  
*You do not need to hire someone who knows a separate vector database stack.*

Bạn không cần đào tạo lại.  
*You do not need to retrain anyone.*

Bạn không tạo ra một "ốc đảo tri thức".  
*You do not create a "knowledge island".*

Ốc đảo đó là thứ mà chỉ một người trong công ty hiểu.  
*That island is something only one person in the company understands.*

Đây là một lập luận về mặt tổ chức.  
*This is an organizational argument.*

Nó mạnh không kém lập luận kỹ thuật.  
*It is no weaker than the technical argument.*

Đây là kiểu lập luận rất được đánh giá cao trong phỏng vấn cấp staff.  
*This kind of argument is highly valued in staff-level interviews.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời của một staff engineer
*4.6. A sample system design question with a staff engineer's answer*

> **Đề bài — the problem statement**
>
> Hãy thiết kế hệ thống semantic search cho một sàn thương mại điện tử.  
> *Design a semantic search system for an e-commerce marketplace.*
>
> Sàn đó có 50 triệu sản phẩm.  
> *That marketplace has 50 million products.*
>
> Hệ thống phải hỗ trợ lọc theo danh mục, giá và tồn kho.  
> *The system must support filtering by category, price, and stock.*
>
> Yêu cầu p95 dưới 100ms.  
> *The requirement is p95 under 100ms.*
>
> Kiến trúc là multi-tenant.  
> *The architecture is multi-tenant.*

**Khung trả lời — 8 bước / The answer framework — 8 steps**

Bạn nhớ nói to các trade-off.  
*Remember to say the trade-offs out loud.*

**1. Làm rõ đề bài trước khi vẽ gì cả.**  
***1. Clarify the problem before drawing anything.***

Đây là bước mà ứng viên hay bỏ qua.  
*This is the step candidates often skip.*

Bỏ qua nó là mất điểm ngay lập tức.  
*Skipping it loses points immediately.*

Hãy hỏi QPS là bao nhiêu.  
*Ask what the QPS is.*

QPS là số truy vấn mỗi giây.  
*QPS is queries per second.*

Hãy hỏi catalog cập nhật thường xuyên thế nào.  
*Ask how often the catalog updates.*

Hãy hỏi mục tiêu recall là bao nhiêu.  
*Ask what the recall target is.*

Hãy hỏi về ngân sách.  
*Ask about the budget.*

Hãy hỏi có yêu cầu đa vùng địa lý không.  
*Ask whether multi-region is required.*

> *Vì sao bước này quan trọng / Why this step matters*
>
> Staff engineer không thiết kế cho đề bài tưởng tượng.  
> *A staff engineer does not design for an imagined problem.*
>
> Việc hỏi trước cho thấy bạn biết một điều.  
> *Asking first shows that you know one thing.*
>
> Câu trả lời đúng phụ thuộc vào ràng buộc.  
> *The right answer depends on the constraints.*

**2. Tầng embedding.**  
***2. The embedding layer.***

Bạn chọn model khoảng 1024 tới 1536 chiều.  
*Pick a model of roughly 1024 to 1536 dimensions.*

Lựa chọn đó cân bằng recall và chi phí.  
*That choice balances recall and cost.*

Bạn sinh embedding **bất đồng bộ** qua một hàng đợi.  
*Generate embeddings **asynchronously** through a queue.*

Việc đó chạy mỗi khi sản phẩm được thêm hoặc sửa.  
*It runs whenever a product is added or edited.*

Bạn tuyệt đối không gọi API embedding ngay trong luồng ghi dữ liệu.  
*Never call the embedding API inside the write path.*

Lý do là API bên ngoài chậm.  
*The reason is that an external API is slow.*

API bên ngoài cũng có thể lỗi.  
*An external API can also fail.*

Bạn version hoá embedding ngay từ ngày đầu.  
*Version your embeddings from day one.*

**3. Data model.**  
***3. The data model.***

Bạn dùng một bảng `products` chứa cả metadata lẫn cột `vector`.  
*Use one `products` table holding both the metadata and the `vector` column.*

Bạn **partition theo `tenant_id`** để prune sớm.  
***Partition by `tenant_id`** to prune early.*

Bạn đánh index B-tree thông thường trên `(category, price, in_stock)`.  
*Build an ordinary B-tree index on `(category, price, in_stock)`.*

**4. Index vector.**  
***4. The vector index.***

Bạn dùng HNSW với `vector_cosine_ops`.  
*Use HNSW with `vector_cosine_ops`.*

Bạn đặt `m` trong khoảng 16 tới 32.  
*Set `m` between 16 and 32.*

Bạn tune `ef_search` theo SLA p95.  
*Tune `ef_search` against the p95 SLA.*

Bạn bật `hnsw.iterative_scan = relaxed_order`.  
*Enable `hnsw.iterative_scan = relaxed_order`.*

Cách này chống overfiltering khi bộ lọc chặt.  
*This defends against overfiltering when the filter is tight.*

**5. Truy vấn.**  
***5. The query.***

Bạn dùng một câu SQL duy nhất.  
*Use a single SQL statement.*

Bạn lọc metadata trước để prune partition.  
*Filter the metadata first to prune partitions.*

Sau đó bạn dùng `ORDER BY <=>`.  
*Then use `ORDER BY <=>`.*

Sau đó bạn dùng `LIMIT`.  
*Then use `LIMIT`.*

Bạn có thể cần độ chính xác cao hơn.  
*You may need higher accuracy.*

Khi đó bạn thêm bước re-rank hai giai đoạn.  
*You then add a two-stage re-rank.*

**6. Scale và latency.**  
***6. Scale and latency.***

Bạn dùng read replica cho lưu lượng tìm kiếm.  
*Use read replicas for the search traffic.*

Bạn tính nhẩm dung lượng ngay tại chỗ.  
*Do the storage arithmetic on the spot.*

50 triệu nhân khoảng 6KB xấp xỉ **300GB** chỉ riêng vector thô.  
*50 million times about 6KB is roughly **300GB** for the raw vectors alone.*

Bạn cân nhắc `halfvec` hoặc binary quantization.  
*Consider `halfvec` or binary quantization.*

Bạn cũng có thể cân nhắc DiskANN.  
*You may also consider DiskANN.*

Mục đích là để vừa RAM và cắt chi phí.  
*The purpose is fitting RAM and cutting cost.*

Rồi bạn đo p95 thật chứ không đoán.  
*Then measure the real p95 instead of guessing.*

> *Mẹo phỏng vấn / An interview tip*
>
> Việc bạn **tự nhẩm ra con số 300GB** ngay tại chỗ gây ấn tượng mạnh.  
> ***Working out the 300GB figure** on the spot makes a strong impression.*
>
> Nó ấn tượng hơn mọi thuật ngữ bạn liệt kê.  
> *It impresses more than any list of terms.*
>
> Nó chứng tỏ bạn suy nghĩ bằng ràng buộc vật lý.  
> *It proves you think in physical constraints.*
>
> Nó chứng tỏ bạn không suy nghĩ bằng buzzword.  
> *It proves you do not think in buzzwords.*

**7. Vận hành.**  
***7. Operations.***

Bạn giám sát recall@10 bằng cách so định kỳ với exact search.  
*Monitor recall@10 by comparing periodically against exact search.*

Bạn giám sát p99 latency.  
*Monitor p99 latency.*

Bạn giám sát tỷ lệ dòng có embedding NULL.  
*Monitor the share of rows with a NULL embedding.*

Bạn có kế hoạch reindex.  
*Have a reindex plan.*

Bạn dùng blue-green khi đổi model.  
*Use blue-green when changing models.*

**8. Nói rõ khi nào ta nên bỏ pgvector.**  
***8. State clearly when to abandon pgvector.***

Hệ thống có thể tăng lên hàng trăm triệu sản phẩm.  
*The system may grow to hundreds of millions of products.*

Nó có thể phải phục vụ nhiều châu lục.  
*It may have to serve several continents.*

Nó có thể cần hybrid search nặng.  
*It may need heavy hybrid search.*

Khi đó bạn nói thẳng rằng mình sẽ cân nhắc Milvus hoặc Vespa.  
*You then say plainly that you would consider Milvus or Vespa.*

> **Đây là bước ghi điểm cao nhất — this is the highest-scoring step**
>
> Biết giới hạn của lựa chọn của chính mình chính là dấu hiệu staff.  
> *Knowing the limits of your own choice is exactly the staff signal.*
>
> Bạn phải nói ra giới hạn đó.  
> *You must say those limits out loud.*
>
> Người phỏng vấn không tìm người bảo vệ công cụ đến cùng.  
> *An interviewer is not looking for someone who defends a tool to the death.*
>
> Họ tìm người biết công cụ nào phù hợp với ràng buộc nào.  
> *They look for someone who knows which tool fits which constraints.*

### ✅ Self-check Phần 4
*✅ Self-check for Part 4*

**Câu 1.** Vì sao đo latency trung bình là sai lầm, và nên đo cái gì thay thế?  
***Question 1.** Why is measuring average latency a mistake, and what should you measure instead?*

> *Gợi ý đáp án / Suggested answer*
>
> Trung bình bị các truy vấn nhanh kéo xuống.  
> *The average is dragged down by the fast queries.*
>
> Nó che giấu nhóm chậm.  
> *It hides the slow group.*
>
> Bạn nên đo p50, p95 và p99.  
> *You should measure p50, p95, and p99.*
>
> Hệ thống có hàng triệu truy vấn mỗi ngày.  
> *A system handles millions of queries per day.*
>
> Khi đó "1% chậm" là hàng chục nghìn người thật mỗi ngày.  
> *Then "1% slow" is tens of thousands of real people per day.*

**Câu 2.** Vì sao phải chủ động giám sát recall, trong khi các chỉ số khác thì có alert tự động?  
***Question 2.** Why must you monitor recall proactively when other metrics have automatic alerts?*

> *Gợi ý đáp án / Suggested answer*
>
> Recall suy giảm **không gây lỗi**.  
> *Falling recall **causes no error**.*
>
> Hệ thống vẫn chạy.  
> *The system keeps running.*
>
> Hệ thống vẫn nhanh.  
> *The system stays fast.*
>
> Chỉ là kết quả kém liên quan dần.  
> *The results merely grow less relevant.*
>
> Không có exception nào để bắt.  
> *There is no exception to catch.*
>
> Bạn phải chủ động đo bằng cách so với exact search.  
> *You must measure proactively against exact search.*
>
> Exact search đóng vai trò ground truth.  
> *Exact search serves as the ground truth.*

**Câu 3.** Bạn có 50 triệu vector 1536 chiều. Ước lượng dung lượng thô và nêu hai cách giảm.  
***Question 3.** You have 50 million 1536-dimension vectors. Estimate the raw size and name two reductions.*

> *Gợi ý đáp án / Suggested answer*
>
> 50.000.000 × 1536 × 4 byte xấp xỉ **300 GB**.  
> *50,000,000 × 1536 × 4 bytes is roughly **300 GB**.*
>
> Con số đó chưa tính đồ thị HNSW.  
> *That figure excludes the HNSW graph.*
>
> Cách 1 là `halfvec`, cắt khoảng 50%.  
> *Option 1 is `halfvec`, cutting about 50%.*
>
> Cách 2 là binary quantization kèm re-rank, cắt tới khoảng 32 lần.  
> *Option 2 is binary quantization with re-rank, cutting up to about 32 times.*
>
> Cách 3 là Matryoshka, tức cắt bớt số chiều.  
> *Option 3 is Matryoshka, trimming the dimension count.*
>
> Cách 4 là chuyển sang DiskANN để dùng SSD thay RAM.  
> *Option 4 is moving to DiskANN to use SSD instead of RAM.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 WRAPPING UP FOR THE INTERVIEW (Interview cheatsheet)*

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn.  
> *This part is for a quick 20-minute review before an interview.*
>
> Nó cố tình viết ngắn và cô đặc.  
> *It is deliberately short and dense.*
>
> Nó ngược hẳn với bốn phần trên.  
> *It is the opposite of the four parts above.*
>
> Bạn có thể đọc một dòng ở đây mà thấy mơ hồ.  
> *You may read a line here and feel unsure.*
>
> Hãy quay lại đúng phần tương ứng phía trên.  
> *Go back to the matching part above.*
>
> Hãy đọc lại phần đó cho chậm.  
> *Reread that part slowly.*

### 5.1. Keywords bắt buộc nhớ
*5.1. Keywords you must remember*

| Thuật ngữ tiếng Anh / English term | Định nghĩa một dòng / One-line definition |
|---|---|
| **Embedding** | Dãy số biểu diễn ý nghĩa của một object; gần nhau = giống nghĩa. / A number sequence representing an object's meaning; near = alike in meaning. |
| **Vector / dimension** | Mảng số thực độ dài cố định; số phần tử chính là số chiều. / A fixed-length real-number array; the element count is the dimension count. |
| **Embedding model** | Mô hình AI biến chữ/ảnh thành vector; **không phải** pgvector làm việc này. / The AI model turning text or images into vectors; pgvector does **not** do this. |
| **Similarity search / k-NN** | Tìm k vector gần nhất với vector truy vấn. / Find the k vectors nearest the query vector. |
| **Exact search / brute force** | Quét hết, tính khoảng cách với mọi dòng; recall 100% nhưng `O(n·d)`. / Scan everything, distance to every row; 100% recall but `O(n·d)`. |
| **ANN** | *Approximate Nearest Neighbor* — đổi recall lấy tốc độ. / *Approximate Nearest Neighbor* — trades recall for speed. |
| **Recall** | % hàng xóm thật mà thuật toán tìm được; thước đo chất lượng của ANN. / The share of true neighbors found; the quality measure for ANN. |
| **pgvector** | Extension cho Postgres, thêm kiểu dữ liệu `vector`. / A Postgres extension adding the `vector` data type. |
| **HNSW** | Đồ thị nhiều tầng; query ~`O(log n)`; mặc định 2026. / A multi-layer graph; query ~`O(log n)`; the 2026 default. |
| **IVFFlat** | Chia list bằng k-means; build nhanh, cần dữ liệu để train. / Splits lists by k-means; fast builds, needs data to train. |
| **DiskANN / pgvectorscale** | Index dựa trên SSD và nén; tiết kiệm RAM, hỗ trợ tới 16000 chiều. / An SSD-plus-compression index; saves RAM, supports up to 16000 dimensions. |
| **Cosine / L2 / inner product** | Ba thước đo khoảng cách; toán tử `<=>` / `<->` / `<#>`. / Three distance measures; the `<=>`, `<->`, and `<#>` operators. |
| **Ops class** | Ví dụ `vector_cosine_ops`; **phải khớp** toán tử query. / For example `vector_cosine_ops`; it **must match** the query operator. |
| **m / ef_construction** | Tham số build HNSW: recall so với thời gian build và dung lượng. / HNSW build parameters: recall versus build time and size. |
| **ef_search** | Tham số query HNSW: núm xoay recall so với tốc độ, chỉnh được lúc chạy. / An HNSW query parameter: the recall-speed dial, tunable at runtime. |
| **lists / probes** | Tham số tương ứng của IVFFlat: số quận và số quận được thăm dò. / The IVFFlat counterparts: district count and probed district count. |
| **Overfiltering** | Filter chặt cộng ANN dẫn tới trả về thiếu kết quả so với `LIMIT`. / A tight filter plus ANN returns fewer results than `LIMIT`. |
| **Iterative index scan** | Cơ chế chống overfiltering từ pgvector 0.8: `hnsw.iterative_scan`. / The anti-overfiltering mechanism since pgvector 0.8: `hnsw.iterative_scan`. |
| **halfvec / binary quantization / Matryoshka** | Ba kỹ thuật nén vector để cắt RAM và chi phí. / Three vector compression techniques cutting RAM and cost. |
| **Two-stage retrieval** | Lấy thô top-N bằng index nén, rồi re-rank chính xác. / Grab a coarse top-N with a compressed index, then re-rank exactly. |
| **Curse of dimensionality** | Ở số chiều lớn mọi điểm gần như cách đều nhau; cây phân hoạch vô dụng. / At high dimensions all points are nearly equidistant; partition trees are useless. |
| **RAG** | *Retrieval-Augmented Generation*; vector search chính là bước "retrieval". / *Retrieval-Augmented Generation*; vector search is the retrieval step. |
| **Hybrid search** | Kết hợp vector search với tìm từ khoá (BM25) rồi trộn điểm. / Combines vector search with keyword search (BM25), then blends scores. |

### 5.2. Core concepts — nếu chỉ được nhớ mười điều
*5.2. Core concepts — if you can remember only ten things*

1. Vector search là tìm hàng xóm gần nhất trên "bản đồ ý nghĩa".  
   *1. Vector search is finding nearest neighbors on the "map of meaning".*

   **Khoảng cách nhỏ nghĩa là giống nghĩa.**  
   ***A small distance means alike in meaning.***

2. pgvector chỉ **lưu và tìm** vector.  
   *2. pgvector only **stores and searches** vectors.*

   Embedding do một model AI riêng sinh ra.  
   *A separate AI model generates the embeddings.*

   Kết quả dở về ngữ nghĩa thường do model, không do index.  
   *Semantically poor results usually come from the model, not the index.*

3. Mặc định pgvector làm **exact search với recall 100%**.  
   *3. By default pgvector does **exact search with 100% recall**.*

   Nó không cần index.  
   *It needs no index.*

   Index chỉ cần khi dữ liệu lớn.  
   *An index is needed only when the data grows large.*

4. Index ANN **bán tốc độ bằng cách lấy đi độ chính xác**.  
   *4. An ANN index **sells speed by taking accuracy**.*

   Đó là lựa chọn của bạn.  
   *That is your choice.*

   Đó không phải lỗi của nó.  
   *That is not its fault.*

5. **HNSW là mặc định.**  
   *5. **HNSW is the default.***

   Query xấp xỉ `O(log n)` và recall cao.  
   *Queries are about `O(log n)` with high recall.*

   Đổi lại build chậm và tốn RAM.  
   *In exchange builds are slow and RAM-hungry.*

6. **Ops class trong index phải khớp toán tử query.**  
   *6. **The index ops class must match the query operator.***

   Nếu không, Postgres âm thầm bỏ qua index.  
   *Otherwise Postgres silently ignores the index.*

7. Với embedding văn bản, bạn dùng **cosine** với `<=>`.  
   *7. For text embeddings, use **cosine** with `<=>`.*

   Bạn đã normalize thì `<#>` nhanh hơn mà tương đương.  
   *If you normalized, `<#>` is faster and equivalent.*

8. **Siêu năng lực của pgvector** là lọc và join vector với dữ liệu nghiệp vụ.  
   *8. **pgvector's superpower** is filtering and joining vectors with business data.*

   Việc đó gọn trong **một câu SQL, một round-trip, một transaction**.  
   *That fits in **one SQL statement, one round-trip, one transaction**.*

9. **Overfiltering** là bẫy kinh điển khi filter chặt.  
   *9. **Overfiltering** is the classic trap with tight filters.*

   Bạn bật `iterative_scan` để chống nó.  
   *Enable `iterative_scan` to counter it.*

10. Ở tầm staff, bottleneck là **RAM** và việc **giám sát recall**.  
    *10. At staff level the bottlenecks are **RAM** and **recall monitoring**.*

    Đòn bẩy là quantization, partitioning và two-stage re-rank.  
    *The levers are quantization, partitioning, and two-stage re-ranking.*

### 5.3. Mental model / analogy để nói cho trôi chảy
*5.3. Mental models and analogies for speaking fluently*

**"Bản đồ ý nghĩa" / "The map of meaning"**

Nó giải thích embedding cho người mới trong đúng một câu.  
*It explains embeddings to a beginner in exactly one sentence.*

Bạn dùng được nó cả với sếp không kỹ thuật.  
*You can use it even with a non-technical boss.*

**"Đường cao tốc nhiều tầng" / "Multi-layer highways"**

Nó giải thích cách HNSW đi từ tầng cao xuống tầng thấp.  
*It explains how HNSW moves from the high layers down to the low ones.*

**"Chia quận, chỉ vào vài quận gần" / "Split districts, enter only a few nearby"**

Nó giải thích IVFFlat.  
*It explains IVFFlat.*

Nó kèm luôn điểm yếu ở ranh giới quận.  
*It also carries the weakness at district boundaries.*

**"Thô trước, tinh sau" / "Coarse first, fine second"**

Nó giải thích two-stage retrieval.  
*It explains two-stage retrieval.*

**"Vector là một tính năng hay là cả sản phẩm?" / "Is the vector a feature or the whole product?"**

Đây là kim chỉ nam chọn giữa pgvector và vector DB chuyên dụng.  
*This is the compass for choosing between pgvector and a dedicated vector DB.*

**"Mục lục cuối sách" / "The index at the back of a book"**

Nó giải thích index nói chung.  
*It explains indexes in general.*

Nó kèm luôn cái giá của index.  
*It also carries the price of an index.*

Cái giá đó là tốn giấy và phải cập nhật.  
*That price is extra paper and mandatory updates.*

### 5.4. Code cần thuộc lòng
*5.4. Code you must memorize*

**(a) Vòng đời tối thiểu — interviewer rất hay bắt viết tại chỗ**  
*(a) The minimal life cycle — interviewers often ask you to write it on the spot*

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

**(b) Filtered search — câu "flex" thể hiện siêu năng lực của pgvector**  
*(b) Filtered search — the brag that shows pgvector's superpower*

```sql
SELECT id FROM items
WHERE tenant_id = 42 AND in_stock
ORDER BY embedding <=> :query_vec
LIMIT 10;
```

Bạn nói kèm một câu.  
*Say one line alongside it.*

"Với vector DB tách rời thì đây là hai round-trip và một vòng lặp thủ công."  
*"With a separate vector DB this is two round-trips and a manual loop."*

**(c) Exact k-NN cosine bằng NumPy**  
*(c) Exact cosine k-NN with NumPy*

Đoạn này chứng tỏ bạn hiểu bản chất.  
*This snippet proves you understand the essence.*

Nó chứng tỏ bạn không chỉ biết gọi thư viện.  
*It proves you do more than call a library.*

```python
def cosine_knn(q, corpus, k=10):
    q = q / (q @ q) ** 0.5                              # chuẩn hoá query
                                                        # normalize the query
    c = corpus / (corpus ** 2).sum(1, keepdims=True) ** 0.5   # chuẩn hoá từng dòng
                                                              # normalize each row
    dist = 1 - c @ q                                    # cosine distance
                                                        # cosine distance
    idx = dist.argsort()[:k]                            # k khoảng cách nhỏ nhất
                                                        # the k smallest distances
    return idx, dist[idx]
```

**(d) Đo recall — đoạn ít ai nhớ, nhưng gây ấn tượng mạnh**  
*(d) Measuring recall — few remember this one, but it impresses*

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact search làm ground truth
                                    -- force exact search as the ground truth
SELECT id FROM docs ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so độ trùng với kết quả có index -> recall@10
-- compare the overlap with the indexed result -> recall@10
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions with suggested answers*

**1. "Khác nhau giữa HNSW và IVFFlat là gì?"**  
***1. "What is the difference between HNSW and IVFFlat?"***

> HNSW là đồ thị nhiều tầng.  
> *HNSW is a multi-layer graph.*
>
> Query của nó xấp xỉ `O(log n)`.  
> *Its queries run at about `O(log n)`.*
>
> Recall của nó cao.  
> *Its recall is high.*
>
> Nó không cần train.  
> *It needs no training.*
>
> Nó xử lý dữ liệu thêm mới rất tốt.  
> *It handles newly added data very well.*
>
> Đổi lại nó build chậm và tốn RAM.  
> *In exchange it builds slowly and uses much RAM.*
>
> IVFFlat chia list bằng k-means.  
> *IVFFlat splits lists with k-means.*
>
> Nó build nhanh và tốn ít RAM.  
> *It builds fast and uses little RAM.*
>
> Nó cần dữ liệu sẵn để train.  
> *It needs existing data to train on.*
>
> Nó kém với dữ liệu thay đổi liên tục.  
> *It handles constantly changing data poorly.*
>
> Mặc định hiện nay là HNSW.  
> *The current default is HNSW.*

**2. [BẪY] "Vì sao kết quả tìm kiếm 'sai' sau khi tôi thêm index?"**  
***2. [TRAP] "Why are the search results 'wrong' after I added an index?"***

> Kết quả không sai.  
> *The results are not wrong.*
>
> ANN là *approximate* theo đúng thiết kế.  
> *ANN is *approximate* by design.*
>
> Bạn đã đổi recall lấy tốc độ.  
> *You traded recall for speed.*
>
> Cách xử lý đúng bắt đầu bằng việc **đo** recall.  
> *The right response starts by **measuring** recall.*
>
> Sau đó bạn tăng `ef_search`, cách này rẻ.  
> *Then raise `ef_search`, which is cheap.*
>
> Sau đó bạn mới tăng `m` và `ef_construction` rồi build lại.  
> *Only then raise `m` and `ef_construction` and rebuild.*
>
> Cách sau đắt hơn.  
> *That route is more expensive.*
>
> Bạn đừng chỉnh mò mà không đo.  
> *Do not tweak blindly without measuring.*

**3. [TRADE-OFF] "Tăng `ef_search` thì được gì mất gì?"**  
***3. [TRADE-OFF] "What do you gain and lose by raising `ef_search`?"***

> Recall tăng.  
> *Recall rises.*
>
> Latency cũng tăng.  
> *Latency rises too.*
>
> CPU và RAM tốn hơn.  
> *CPU and RAM usage grow.*
>
> Nó là núm xoay recall và tốc độ **ở thời điểm query**.  
> *It is the recall-speed dial **at query time**.*
>
> Bạn chỉnh được nó mà không cần build lại index.  
> *You can turn it without rebuilding the index.*
>
> Bạn tune nó theo SLA p95.  
> *Tune it against the p95 SLA.*

**4. "Nên dùng cosine hay L2?"**  
***4. "Should I use cosine or L2?"***

> Với embedding văn bản, bạn dùng cosine.  
> *For text embeddings, use cosine.*
>
> Lý do là ta quan tâm hướng ngữ nghĩa.  
> *The reason is our interest in semantic direction.*
>
> Ta không quan tâm độ dài vector.  
> *We do not care about vector length.*
>
> Bạn dùng L2 khi độ lớn của vector thực sự mang ý nghĩa.  
> *Use L2 when the vector magnitude genuinely carries meaning.*
>
> Bạn đã normalize về độ dài 1.  
> *Suppose you normalized to length 1.*
>
> Khi đó inner product tương đương cosine.  
> *Inner product is then equivalent to cosine.*
>
> Inner product lại tính nhanh hơn.  
> *Inner product also computes faster.*

**5. [BẪY] "Filter chặt mà trả về ít kết quả hơn `LIMIT`, vì sao?"**  
***5. [TRAP] "A tight filter returns fewer results than `LIMIT`. Why?"***

> Tên hiện tượng là overfiltering.  
> *The phenomenon is called overfiltering.*
>
> Index lấy khoảng `ef_search` ứng viên **trước**.  
> *The index grabs about `ef_search` candidates **first**.*
>
> `WHERE` lọc **sau**.  
> *`WHERE` filters **afterwards**.*
>
> Với bộ lọc hiếm thì gần hết ứng viên bị loại.  
> *With a rare filter, nearly all candidates are eliminated.*
>
> Bạn fix bằng `hnsw.iterative_scan = relaxed_order`.  
> *You fix it with `hnsw.iterative_scan = relaxed_order`.*
>
> Bạn kèm `max_scan_tuples` làm trần an toàn.  
> *Add `max_scan_tuples` as a safety ceiling.*
>
> Tính năng này có từ pgvector 0.8 trở lên.  
> *This feature exists from pgvector 0.8 onward.*

**6. [SCALE] "Một tỷ vector thì làm thế nào?"**  
***6. [SCALE] "What do you do with a billion vectors?"***

> Bạn bắt đầu bằng việc nêu bottleneck.  
> *Start by naming the bottleneck.*
>
> Bottleneck đó là RAM.  
> *That bottleneck is RAM.*
>
> RAM phải chứa cả vector thô lẫn đồ thị.  
> *RAM must hold both the raw vectors and the graph.*
>
> Sau đó bạn nêu quantization bằng `halfvec` hoặc binary.  
> *Then name quantization with `halfvec` or binary.*
>
> Bạn kèm two-stage re-rank.  
> *Pair it with two-stage re-ranking.*
>
> Bạn partition theo access pattern.  
> *Partition by access pattern.*
>
> Bạn dùng DiskANN qua `pgvectorscale`.  
> *Use DiskANN through `pgvectorscale`.*
>
> Bạn dùng read replica cho tải đọc.  
> *Use read replicas for the read load.*
>
> Rồi bạn nói thành thật một điều.  
> *Then say one honest thing.*
>
> Bạn cần đa vùng địa lý thì nên cân nhắc vector DB chuyên dụng.  
> *If you need multi-region, consider a dedicated vector DB.*

**7. [TRADE-OFF] "Khi nào KHÔNG dùng pgvector mà chọn Pinecone?"**  
***7. [TRADE-OFF] "When would you NOT use pgvector and pick Pinecone?"***

> Khi vector search là workload chính chứ không phải một tính năng.  
> *When vector search is the main workload rather than a feature.*
>
> Khi vượt khoảng 50 triệu vector và cần đa vùng.  
> *When you exceed roughly 50 million vectors and need multi-region.*
>
> Khi cần hybrid search mạnh sẵn có.  
> *When you need strong built-in hybrid search.*
>
> Khi tổ chức vốn không chạy Postgres.  
> *When the organization does not run Postgres at all.*
>
> pgvector thắng ở sự đơn giản trong vận hành.  
> *pgvector wins on operational simplicity.*
>
> pgvector không thắng ở hiệu năng đỉnh.  
> *pgvector does not win on peak performance.*

**8. "Đổi embedding model thì chuyện gì xảy ra?"**  
***8. "What happens when you change the embedding model?"***

> Vector cũ và vector mới không cùng một không gian.  
> *Old and new vectors do not share a space.*
>
> Bạn phải sinh lại embedding cho toàn bộ kho dữ liệu.  
> *You must regenerate embeddings for the entire corpus.*
>
> Cách làm là version hoá cột embedding.  
> *The method is versioning the embedding column.*
>
> Bạn backfill dần.  
> *You backfill gradually.*
>
> Bạn dùng blue-green khi chuyển.  
> *You use blue-green during the switch.*
>
> Đây là quyết định kiến trúc **nặng hơn** cả việc chọn index.  
> *This is a **heavier** architectural decision than choosing an index.*

**9. "Làm sao bạn biết chất lượng tìm kiếm đang xấu đi trên production?"**  
***9. "How do you know search quality is degrading in production?"***

> Câu này ít được hỏi trực tiếp.  
> *This question is rarely asked directly.*
>
> Nó cực kỳ ghi điểm khi bạn chủ động nói ra.  
> *It scores heavily when you raise it yourself.*
>
> Recall suy giảm **không gây lỗi**.  
> *Falling recall **causes no error**.*
>
> Vậy nên bạn phải chủ động đo.  
> *You must therefore measure proactively.*
>
> Bạn so kết quả có index với exact search.  
> *Compare the indexed results against exact search.*
>
> Exact search đóng vai trò ground truth.  
> *Exact search serves as the ground truth.*
>
> Bạn lấy mẫu định kỳ trên replica.  
> *Sample periodically on a replica.*
>
> Bạn vẽ recall@10 theo thời gian lên dashboard.  
> *Plot recall@10 over time on a dashboard.*

**10. [DATA MODELING] "Bạn thiết kế bảng thế nào cho multi-tenant?"**  
***10. [DATA MODELING] "How do you design the table for multi-tenant?"***

> Vector nằm cùng bảng với metadata.  
> *The vector lives in the same table as the metadata.*
>
> Bảng có cột `tenant_id NOT NULL`.  
> *The table has a `tenant_id NOT NULL` column.*
>
> Mọi query đều lọc theo cột đó.  
> *Every query filters on that column.*
>
> Bạn partition theo `tenant_id` để prune sớm.  
> *Partition by `tenant_id` to prune early.*
>
> Nhờ vậy index vector của mỗi partition đủ nhỏ để vừa RAM.  
> *Each partition's vector index then stays small enough for RAM.*
>
> Bạn đánh index B-tree trên các cột lọc thường dùng.  
> *Build B-tree indexes on the columns you filter by often.*

### 5.6. One-liner đắt giá — thả đúng lúc để ghi điểm
*5.6. High-value one-liners — drop them at the right moment*

Siêu năng lực của pgvector không phải là tốc độ.  
*pgvector's superpower isn't speed.*

Vector của bạn sống ngay cạnh dữ liệu nghiệp vụ.  
*Your vectors live next to your business data.*

Nhờ vậy filtered search chỉ là một câu SQL, một round-trip, một transaction.  
*So filtered search is one SQL query, one round-trip, one transaction.*

ANN là một cuộc đổi chác recall lấy tốc độ.  
*ANN is a recall-for-speed trade.*

Kỹ năng thật sự là giám sát recall trên production.  
*The real skill is monitoring recall in production.*

Lý do là không có lỗi biên dịch nào khi độ liên quan âm thầm xấu đi.  
*There's no compile error when relevance quietly degrades.*

Hãy chọn pgvector khi vector search là một tính năng.  
*Choose pgvector when vector search is a feature.*

Đừng chọn nó khi vector search là cả sản phẩm.  
*Don't choose it when vector search is the product.*

Quyết định kiến trúc nặng nhất không phải là index.  
*The heaviest architectural decision isn't the index.*

Quyết định đó là embedding model.  
*That decision is the embedding model.*

Đổi model nghĩa là phải re-embed toàn bộ kho dữ liệu.  
*Changing it means re-embedding the entire corpus.*

Ở quy mô một tỷ vector, bottleneck là RAM chứ không phải CPU.  
*At a billion vectors the bottleneck is RAM, not CPU.*

Lúc đó quantization và two-stage re-ranking không còn là tuỳ chọn.  
*That's when quantization and two-stage re-ranking stop being optional.*

Hãy partition theo access pattern.  
*Partition by access pattern.*

Đừng partition theo kích thước.  
*Don't partition by size.*

Nhờ vậy planner có thể prune trước cả khi index chạy.  
*So the planner can prune before the index even runs.*

---

## 📌 Ghi chú cuối — làm gì tiếp theo
*📌 Final notes — what to do next*

**1. Thực hành ngay, đừng chỉ đọc / 1. Practice now, do not just read**

Cách nhanh nhất để có một Postgres kèm pgvector là dùng **Docker**.  
*The fastest way to get Postgres with pgvector is **Docker**.*

Docker là công cụ chạy phần mềm trong "hộp" đóng gói sẵn.  
*Docker runs software inside a pre-packaged "box".*

Bạn không phải cài đặt lằng nhằng lên máy.  
*You avoid messy installation on your machine.*

```bash
docker run --name pgvec -e POSTGRES_PASSWORD=pass -p 5432:5432 -d pgvector/pgvector:pg17
```

Rồi bạn chạy lại **toàn bộ** code trong Phần 1 đến Phần 3.  
*Then rerun **all** the code from Part 1 through Part 3.*

Đọc lý thuyết mười lần không thay được một lần tự gõ `EXPLAIN ANALYZE`.  
*Reading theory ten times cannot replace typing `EXPLAIN ANALYZE` once yourself.*

Bạn phải nhìn thấy dòng `Index Scan using ..._hnsw...` bằng chính mắt mình.  
*You must see the `Index Scan using ..._hnsw...` line with your own eyes.*

**2. Kiểm chứng phiên bản trước khi phỏng vấn / 2. Verify the version before an interview**

pgvector phát triển rất nhanh.  
*pgvector develops very fast.*

Bản 0.8.5 là bản tính tới tháng 7/2026.  
*Version 0.8.5 is current as of July 2026.*

Bạn hãy liếc CHANGELOG trên GitHub `pgvector/pgvector`.  
*Glance at the CHANGELOG on GitHub at `pgvector/pgvector`.*

Các con số như giới hạn số chiều có thể đã đổi.  
*Numbers such as the dimension limit may have changed.*

Tên tham số cũng có thể đã đổi.  
*Parameter names may have changed too.*

Có thể đã có tính năng mới.  
*New features may exist.*

Nhắc được một tính năng mới ra là dấu hiệu bạn theo dõi công nghệ.  
*Mentioning a newly released feature signals that you follow the technology.*

Nó cho thấy bạn không học vẹt.  
*It shows you are not parroting.*

**3. Bài tập tự kiểm tra tổng hợp / 3. A comprehensive self-check exercise**

Hãy tự dựng lại toàn bộ ví dụ shop quần áo.  
*Rebuild the whole clothing shop example yourself.*

Bạn sinh embedding cho 1000 sản phẩm.  
*Generate embeddings for 1000 products.*

Bạn đo latency khi chưa có index.  
*Measure the latency without an index.*

Bạn đánh index HNSW rồi đo lại.  
*Build the HNSW index and measure again.*

Sau đó bạn **tự tính recall@10** so với exact search.  
*Then **compute recall@10 yourself** against exact search.*

Bạn nhìn thấy con số recall của chính mình bằng mắt.  
*You see your own recall figure with your own eyes.*

Khi đó mọi thứ trong Phần 3 và Phần 4 sẽ ăn sâu.  
*Everything in Parts 3 and 4 then sinks in deeply.*

Đọc không bao giờ làm được điều đó.  
*Reading never achieves that.*

**4. Chủ đề nên học tiếp, theo thứ tự / 4. Topics to learn next, in order**

> **RAG end-to-end**
>
> Bạn ghép vector search với một LLM.  
> *You combine vector search with an LLM.*
>
> Mục tiêu là làm chatbot tra cứu tài liệu nội bộ.  
> *The goal is a chatbot that looks up internal documents.*

> **Chunking strategies — chiến lược cắt đoạn**
>
> Đây là cách cắt tài liệu dài thành đoạn trước khi embed.  
> *This is how you split long documents into chunks before embedding.*
>
> Nó ảnh hưởng tới chất lượng còn nhiều hơn cả việc chọn index.  
> *It affects quality even more than the index choice.*

> **Hybrid search**
>
> Bạn kết hợp full-text search sẵn có của Postgres với vector search.  
> *You combine Postgres's built-in full-text search with vector search.*
>
> Full-text search ở đây dùng BM25.  
> *The full-text search here uses BM25.*
>
> Bạn trộn điểm bằng Reciprocal Rank Fusion.  
> *You blend the scores with Reciprocal Rank Fusion.*

> **Reranking models — model xếp hạng lại**
>
> Đây là model chuyên xếp hạng lại top-N kết quả.  
> *These models specialize in reordering the top-N results.*
>
> Đó chính là giai đoạn 2 của two-stage retrieval.  
> *That is exactly stage 2 of two-stage retrieval.*

> **Evaluation của retrieval — đánh giá chất lượng truy hồi**
>
> Các chỉ số gồm recall@k, MRR và nDCG.  
> *The metrics include recall@k, MRR, and nDCG.*
>
> Đây là cách đo chất lượng tìm kiếm một cách khoa học.  
> *This is how you measure search quality scientifically.*
>
> Rất ít kỹ sư có kỹ năng này.  
> *Very few engineers have this skill.*
>
> Nó tách biệt người đoán mò với người biết mình đang làm gì.  
> *It separates the guesser from the person who knows what they are doing.*
