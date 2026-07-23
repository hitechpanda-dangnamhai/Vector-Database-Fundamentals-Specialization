# RDBMS Nào Hỗ Trợ Vector Search? PostgreSQL vs MySQL vs MariaDB — Giáo trình Basic → Staff
*Which RDBMS Supports Vector Search? PostgreSQL vs MySQL vs MariaDB — A Basic to Staff Course*

> **Nguồn gốc — origin of this lesson**
>
> Bài này dựa trên video *"RDBMS that support vector database searches"*.  
> *This lesson is based on the video "RDBMS that support vector database searches".*
>
> Video nằm trong Course 3, IBM Vector Database Fundamentals.  
> *The video belongs to Course 3, IBM Vector Database Fundamentals.*
>
> Bài gốc so sánh khả năng hỗ trợ vector của ba nền tảng.  
> *The original lesson compares vector support across three platforms.*
>
> Ba nền tảng đó là **PostgreSQL (pgvector)**, **MySQL HeatWave** và **MariaDB Vector**.  
> *Those three platforms are **PostgreSQL (pgvector)**, **MySQL HeatWave**, and **MariaDB Vector**.*
>
> Tôi giảng bám sát bài gốc.  
> *I follow the original lesson closely.*
>
> Sau đó tôi đào sâu thành một khung quyết định chọn nền tảng.  
> *Then I go deeper into a decision framework for platform choice.*
>
> Chỗ mở rộng được đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.  
> *Extended parts are marked **[MỞ RỘNG NGOÀI BÀI GỐC]**.*

> **Vị trí trong series — position in the series**
>
> Đây là mảnh *"chọn nhà cho vector"*.  
> *This is the piece about choosing a home for vectors.*
>
> Bốn mảnh trước đi theo thứ tự embed, index, store/query, keyword/FTS.  
> *The four earlier pieces run in this order: embed, index, store/query, keyword/FTS.*
>
> Mảnh embed tạo ra vector.  
> *The embed piece creates vectors.*
>
> Mảnh index làm cho tìm kiếm nhanh hơn.  
> *The index piece makes search faster.*
>
> Mảnh store/query dùng pgvector.  
> *The store/query piece uses pgvector.*
>
> Bài này lùi lại một bước.  
> *This lesson steps back one level.*
>
> Bài này hỏi một câu hỏi chiến lược.  
> *This lesson asks a strategic question.*
>
> *Nên đặt vector ở RDBMS nào?*  
> *Which RDBMS should host the vectors?*
>
> *Hay nên đặt vector ở một vector DB riêng?*  
> *Or should the vectors live in a separate vector DB?*

> **⚠️ Đính chính & cập nhật tới 2026 — corrections and updates through 2026**
>
> Bài gốc đã lỗi thời ở nhiều chỗ.  
> *The original lesson is outdated in many places.*
>
> **1.** MariaDB Vector đã đạt **GA** — general availability, tức bản phát hành chính thức.  
> ***1.** MariaDB Vector has reached **GA**, the official general availability release.*
>
> Cột mốc này nằm ở bản **11.8 LTS**, tháng 6/2025.  
> *This milestone landed in version **11.8 LTS**, June 2025.*
>
> Bài gốc dùng thì tương lai cho tính năng này.  
> *The original lesson uses the future tense for this feature.*
>
> Bài gốc viết "will store..." và "will provide...".  
> *The original lesson writes "will store..." and "will provide...".*
>
> Lúc đó MariaDB Vector còn ở bản preview.  
> *At that time MariaDB Vector was still in preview.*
>
> **2.** Bài gốc nói MariaDB lưu primary data và vector data riêng biệt ở một database đặc biệt.  
> ***2.** The original lesson says MariaDB stores primary data and vector data separately in a special database.*
>
> Ý này **SAI** với bản GA.  
> *This claim is **wrong** for the GA release.*
>
> Thực tế embedding nằm trong **cột `VECTOR` ngay trong bảng**.  
> *In reality the embedding sits in a **`VECTOR` column inside the table**.*
>
> Embedding nằm cùng chỗ với dữ liệu nghiệp vụ.  
> *The embedding lives in the same place as the business data.*
>
> Bạn search bằng SQL.  
> *You search with SQL.*
>
> Đây là một *ưu điểm*.  
> *This is an *advantage*.*
>
> Ưu điểm đó là hybrid query trong một câu.  
> *That advantage is a hybrid query in a single statement.*
>
> Đây không phải sự tách rời.  
> *This is not a separation.*
>
> **3.** PlanetScale (Vitess) đã có vector index từ tháng 3/2025.  
> ***3.** PlanetScale (Vitess) has had a vector index since March 2025.*
>
> PlanetScale dùng thuật toán **SPANN**.  
> *PlanetScale uses the **SPANN** algorithm.*
>
> Bài gốc nói tính năng này sắp ra.  
> *The original lesson says this feature is coming soon.*
>
> **4.** Bài gốc gọi `tsvector` là một kiểu dữ liệu vector của PostgreSQL.  
> ***4.** The original lesson calls `tsvector` a vector data type of PostgreSQL.*
>
> Cách gọi này gây nhầm lẫn.  
> *This wording is misleading.*
>
> `tsvector` phục vụ **full-text search**.  
> *`tsvector` serves **full-text search**.*
>
> `tsvector` chứa lexeme.  
> *`tsvector` holds lexemes.*
>
> Kiểu `vector` của pgvector chứa embedding.  
> *The `vector` type from pgvector holds embeddings.*
>
> Hai kiểu này khác hẳn nhau.  
> *These two types are entirely different.*
>
> Bạn đừng lẫn hai thứ này.  
> *Do not confuse the two.*
>
> **5.** MySQL Community Server 9.0 trở lên có kiểu `VECTOR`.  
> ***5.** MySQL Community Server 9.0 and later has the `VECTOR` type.*
>
> Bản này cũng có distance function.  
> *This edition also has distance functions.*
>
> **ANN vector index** chủ yếu đến qua **HeatWave**.  
> *The **ANN vector index** comes mainly through **HeatWave**.*
>
> HeatWave là dịch vụ managed và proprietary.  
> *HeatWave is a managed, proprietary service.*
>
> Đây là điểm khác biệt then chốt so với MariaDB.  
> *This is the key difference from MariaDB.*
>
> MariaDB đi theo hướng native và mã nguồn mở.  
> *MariaDB goes the native, open source route.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson map (Overview)*

**Bài này dạy gì / What this lesson teaches**

Các relational database đang thêm khả năng lưu vector.  
*Relational databases are adding the ability to store vectors.*

Ba database đó là PostgreSQL, MySQL và MariaDB.  
*Those three databases are PostgreSQL, MySQL, and MariaDB.*

Chúng cũng thêm khả năng tìm vector.  
*They also add the ability to search vectors.*

Bạn không cần dựng một vector database riêng nữa.  
*You no longer need to set up a separate vector database.*

Chúng làm điều đó theo **ba cách kiến trúc khác nhau**.  
*They do this in **three different architectural ways**.*

Ba cách đó là extension, native type và managed service.  
*Those three ways are extension, native type, and managed service.*

Lựa chọn kiến trúc ảnh hưởng lớn tới chi phí.  
*The architectural choice strongly affects cost.*

Lựa chọn đó ảnh hưởng lớn tới vận hành.  
*That choice strongly affects operations.*

Lựa chọn đó tác động tới vendor lock-in — khóa nhà cung cấp.  
*That choice also drives vendor lock-in.*

**Vấn đề nó giải quyết / The problem it solves**

Bạn đã có một RDBMS chạy production.  
*You already run an RDBMS in production.*

Giờ bạn cần semantic search — tìm kiếm theo ngữ nghĩa.  
*Now you need semantic search.*

Bạn cũng có thể cần RAG.  
*You may also need RAG.*

Câu hỏi thực tế rất đơn giản.  
*The practical question is simple.*

*Bạn cắm vector vào chính database sẵn có?*  
*Do you plug vectors into the database you already have?*

*Hay bạn mua và dựng Pinecone hoặc Weaviate riêng?*  
*Or do you buy and run a separate Pinecone or Weaviate?*

Bài này cho bạn một khung để trả lời.  
*This lesson gives you a framework for the answer.*

Bài này chỉ ra điểm mạnh của mỗi RDBMS.  
*This lesson shows the strengths of each RDBMS.*

Bài này cũng chỉ ra điểm yếu của mỗi RDBMS.  
*This lesson also shows the weaknesses of each RDBMS.*

**Học xong bạn sẽ làm được / What you will be able to do**

Bạn sẽ giải thích được ba mô hình hỗ trợ vector.  
*You will be able to explain the three models of vector support.*

Mô hình **extension** có ví dụ là pgvector.  
*The **extension** model is exemplified by pgvector.*

Mô hình **native** có ví dụ là MariaDB Vector.  
*The **native** model is exemplified by MariaDB Vector.*

Mô hình **managed service** có ví dụ là MySQL HeatWave.  
*The **managed service** model is exemplified by MySQL HeatWave.*

Bạn sẽ viết được code lưu vector trên PostgreSQL, MariaDB và MySQL.  
*You will be able to write code that stores vectors on PostgreSQL, MariaDB, and MySQL.*

Bạn sẽ viết được code query vector trên cả ba nền tảng.  
*You will be able to write code that queries vectors on all three platforms.*

Bạn sẽ phân biệt được "bring-your-own-embedding" với "in-database embedding generation".  
*You will be able to distinguish "bring-your-own-embedding" from "in-database embedding generation".*

Bạn sẽ xây được khung quyết định giữa RDBMS-native và dedicated vector DB.  
*You will be able to build a decision framework between RDBMS-native and a dedicated vector DB.*

Bạn sẽ xây được khung quyết định giữa managed và self-host.  
*You will be able to build a decision framework between managed and self-host.*

Bạn sẽ trả lời được câu hỏi system design về chọn nền tảng vector cho một use case.  
*You will be able to answer a system design question about choosing a vector platform for a use case.*

**Mạch basic → staff / The basic to staff progression**

🟢 **Basic**

Phần này nêu lý do RDBMS thêm vector.  
*This part gives the reason an RDBMS adds vectors.*

Phần này trình bày ba mô hình hỗ trợ.  
*This part presents the three support models.*

Phần này đưa ra một analogy.  
*This part offers an analogy.*

Phần này xây một mental model chung.  
*This part builds a shared mental model.*

🟡 **Intermediate**

Phần này đi qua từng nền tảng với code hiện hành.  
*This part walks through each platform with current code.*

Nền tảng thứ nhất là PostgreSQL với pgvector.  
*The first platform is PostgreSQL with pgvector.*

Nền tảng thứ hai là MySQL HeatWave cùng hệ sinh thái của nó.  
*The second platform is MySQL HeatWave with its ecosystem.*

Nền tảng thứ ba là MariaDB Vector.  
*The third platform is MariaDB Vector.*

Phần này so sánh BYO-embedding với in-DB embedding.  
*This part compares BYO-embedding with in-DB embedding.*

Phần này liệt kê các lỗi thường gặp.  
*This part lists the common mistakes.*

🔴 **Advanced**

Phần này mở nắp capo xem kiến trúc bên trong.  
*This part opens the hood to look at the internal architecture.*

Phần này so sánh extension, native engine và managed service.  
*This part compares extension, native engine, and managed service.*

Phần này nêu thuật toán index của mỗi bên.  
*This part names the index algorithm on each side.*

Các thuật toán đó là HNSW, SPANN và ScaNN.  
*Those algorithms are HNSW, SPANN, and ScaNN.*

Phần này nói về giới hạn chiều.  
*This part covers dimension limits.*

Phần này nói về SIMD.  
*This part covers SIMD.*

Phần này đính chính ý "lưu riêng biệt".  
*This part corrects the claim about separate storage.*

🟣 **Staff**

Phần này xây khung quyết định giữa RDBMS và dedicated vector DB.  
*This part builds the decision framework between an RDBMS and a dedicated vector DB.*

Phần này bàn về managed và self-host.  
*This part discusses managed and self-host.*

Phần này bàn về lock-in và migration.  
*This part discusses lock-in and migration.*

Phần này bàn về cost.  
*This part discusses cost.*

Phần này bàn về ảnh hưởng tới tổ chức.  
*This part discusses organizational impact.*

Phần này đưa ra câu hỏi system design.  
*This part presents a system design question.*

🎯 **Cheatsheet**

Phần này gom keywords.  
*This part collects keywords.*

Phần này gom core concepts.  
*This part collects core concepts.*

Phần này gom code cần thuộc lòng.  
*This part collects the code you must memorize.*

Phần này gom câu hỏi phỏng vấn.  
*This part collects interview questions.*

Phần này gom các one-liner.  
*This part collects the one-liners.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

### 1.1. Vấn đề thực tế: có nên dựng thêm một database chỉ để chứa vector?
*1.1. A real problem: should you set up another database just to hold vectors?*

Bạn có một app chạy trên PostgreSQL hoặc MySQL đã nhiều năm.  
*You have an app running on PostgreSQL or MySQL for many years.*

App đó lưu user, order, product và nhiều thứ khác.  
*That app stores users, orders, products, and many other things.*

Giờ sếp muốn thêm tính năng tìm sản phẩm tương tự theo hình ảnh.  
*Now your boss wants a feature that finds similar products by image.*

Tính năng đó cần vector search.  
*That feature needs vector search.*

Bạn có hai hướng đi.  
*You have two directions.*

**Hướng A / Direction A**

Hướng A là dựng thêm một **vector database chuyên dụng**.  
*Direction A is to set up a **dedicated vector database**.*

Ví dụ là Pinecone, Weaviate, Qdrant và Milvus.  
*Examples are Pinecone, Weaviate, Qdrant, and Milvus.*

Đây là một hệ thống mới phải học.  
*This is a new system you must learn.*

Bạn phải vận hành hệ thống đó.  
*You must operate that system.*

Bạn phải đồng bộ dữ liệu qua lại giữa hai nơi.  
*You must sync data back and forth between two places.*

**Hướng B / Direction B**

Hướng B là thêm khả năng vector vào **chính RDBMS đang chạy**.  
*Direction B is to add vector capability to **the RDBMS you already run**.*

Vector nằm cạnh business data.  
*The vectors sit next to the business data.*

Bạn dùng SQL quen thuộc.  
*You use familiar SQL.*

Bạn không thêm hệ thống nào.  
*You add no new system.*

Bài này nói về hướng B.  
*This lesson is about direction B.*

Các RDBMS lớn giờ đều hỗ trợ vector.  
*The major RDBMSs now all support vectors.*

Chúng chỉ khác nhau ở *cách* hỗ trợ.  
*They differ only in *how* they support it.*

### 1.2. Analogy: nâng cấp căn nhà vs mua nhà thứ hai
*1.2. Analogy: upgrading your house vs buying a second house*

**Dedicated vector DB**

Dedicated vector DB giống như mua thêm một căn nhà thứ hai.  
*A dedicated vector DB is like buying a second house.*

Căn nhà đó chỉ để chứa vector.  
*That house only holds vectors.*

Nó rộng rãi và tối ưu cho vector.  
*It is roomy and optimized for vectors.*

Bạn phải đi lại giữa hai căn nhà.  
*You must travel between the two houses.*

Việc đi lại này chính là đồng bộ dữ liệu.  
*This travel is exactly data synchronization.*

Bạn trả hai lần tiền điện.  
*You pay two electricity bills.*

Bạn trông coi hai căn nhà.  
*You watch over two houses.*

**RDBMS + vector**

RDBMS cộng vector giống như cơi nới thêm một phòng.  
*An RDBMS plus vectors is like adding one extra room.*

Căn phòng đó nằm trong căn nhà sẵn có.  
*That room sits inside the house you already own.*

Mọi thứ nằm dưới một mái nhà.  
*Everything stays under one roof.*

Bạn chỉ đi lại trong nhà.  
*You only move inside the house.*

Việc đi lại đó chính là một câu SQL.  
*That movement is exactly one SQL statement.*

Cách này đủ tốt cho phần lớn nhu cầu.  
*This approach is good enough for most needs.*

Có ba cách cơi nới.  
*There are three ways to extend.*

| Mô hình / Model | Ví dụ / Example | Nghĩa là / Meaning |
|---|---|---|
| **Extension** / **Extension** | PostgreSQL + **pgvector** / PostgreSQL + **pgvector** | cài thêm gói vào server sẵn có để có kiểu `vector` / install an extra package into the existing server to get the `vector` type |
| **Native** / **Native** | **MariaDB Vector** / **MariaDB Vector** | kiểu `VECTOR` được xây thẳng vào engine, khỏi cài gì / the `VECTOR` type is built straight into the engine, with nothing to install |
| **Managed service** / **Managed service** | **MySQL HeatWave** / **MySQL HeatWave** | dịch vụ đám mây làm sẵn cả vector *và* sinh embedding / a cloud service that provides both vectors *and* embedding generation |

### 1.3. Định nghĩa thuật ngữ (lần đầu xuất hiện)
*1.3. Term definitions (first appearance)*

> **Vector / embedding — vector, embedding**
>
> Vector là một mảng số.  
> *A vector is an array of numbers.*
>
> Mảng số đó biểu diễn dữ liệu phức tạp.  
> *That array of numbers represents complex data.*
>
> Dữ liệu đó có thể là ảnh, text hoặc audio.  
> *That data can be an image, text, or audio.*
>
> Mục đích là so sánh theo *độ giống nhau*.  
> *The purpose is comparison by *similarity*.*
>
> Bạn đã học phần này ở giáo trình embeddings.  
> *You already learned this in the embeddings course.*

> **Vector similarity search — tìm kiếm theo độ tương đồng vector**
>
> Vector similarity search tìm vector gần nhất theo distance.  
> *Vector similarity search finds the nearest vectors by distance.*
>
> Các distance thường dùng là cosine, Euclidean/L2, Manhattan và Jaccard.  
> *Common distances are cosine, Euclidean/L2, Manhattan, and Jaccard.*

> **Extension — gói mở rộng**
>
> Extension là gói cài thêm vào một database.  
> *An extension is a package installed into a database.*
>
> Gói đó mang lại tính năng mới.  
> *That package brings new features.*
>
> pgvector là một ví dụ.  
> *pgvector is one example.*
>
> Bạn cần chạy `CREATE EXTENSION`.  
> *You need to run `CREATE EXTENSION`.*

> **Native support — hỗ trợ gốc**
>
> Native support là tính năng được xây thẳng vào lõi database.  
> *Native support is a feature built directly into the database core.*
>
> Bạn dùng ngay không cần cài gì.  
> *You use it immediately without installing anything.*
>
> Kiểu `VECTOR` của MariaDB là một ví dụ.  
> *MariaDB's `VECTOR` type is one example.*

> **Managed database service — dịch vụ database được quản lý**
>
> Managed database service là database chạy bởi nhà cung cấp đám mây.  
> *A managed database service is a database run by a cloud provider.*
>
> Nhà cung cấp cũng vận hành database đó.  
> *The provider also operates that database.*
>
> MySQL HeatWave của Oracle là một ví dụ.  
> *Oracle's MySQL HeatWave is one example.*
>
> Bạn không quản lý hạ tầng.  
> *You do not manage the infrastructure.*

> **OLTP (Online Transaction Processing) — xử lý giao dịch trực tuyến**
>
> OLTP là xử lý giao dịch thời gian thực.  
> *OLTP is real-time transaction processing.*
>
> Ví dụ là đặt hàng và thanh toán.  
> *Examples are placing orders and making payments.*
>
> Đây là thế mạnh truyền thống của RDBMS.  
> *This is the traditional strength of an RDBMS.*

> **In-database embedding — sinh embedding ngay trong database**
>
> Database *tự* sinh embedding từ text.  
> *The database generates embeddings from text *by itself*.*
>
> Nó dùng một LLM tích hợp.  
> *It uses an integrated LLM.*
>
> HeatWave GenAI là một ví dụ.  
> *HeatWave GenAI is one example.*
>
> Cách này khác với "bring-your-own".  
> *This differs from "bring-your-own".*
>
> Với "bring-your-own", bạn tự sinh vector bên ngoài.  
> *With "bring-your-own", you generate the vectors outside.*
>
> Sau đó bạn nhét vector vào database.  
> *Then you push the vectors into the database.*

### 1.4. Mental model chung: mọi nền tảng đều làm cùng một việc
*1.4. A shared mental model: every platform does the same job*

Luồng làm việc luôn giống nhau.  
*The workflow is always the same.*

Điều này đúng với extension, native và managed.  
*This holds for extension, native, and managed.*

```
1. Tạo cột kiểu vector (độ dài = số chiều của model)   -> vector(1536) / VECTOR(1536)
   Create a vector column (length = the model's dimension count)
2. Sinh embedding (bằng model) rồi INSERT vào cột đó
   Generate the embedding with a model, then INSERT it into that column
3. Tạo index (thường HNSW) để tìm nhanh
   Create an index (usually HNSW) for fast search
4. Query: ORDER BY <distance>(cột, query_vector) LIMIT k
   Query: ORDER BY <distance>(column, query_vector) LIMIT k
```

Bạn học kỹ một nền tảng.  
*You learn one platform thoroughly.*

Sau đó ba nền tảng chỉ khác nhau ở *cú pháp*.  
*After that, the three platforms differ only in *syntax*.*

*Ý tưởng* của chúng không khác nhau.  
*Their *ideas* do not differ.*

Bài gốc nói đúng ở điểm này.  
*The original lesson is right on this point.*

Hầu hết RDBMS giờ cung cấp add-on hoặc wrapper.  
*Most RDBMSs now provide an add-on or a wrapper.*

Add-on đó mang lại kiểu vector.  
*That add-on brings a vector type.*

Add-on đó cho phép tìm theo similarity.  
*That add-on enables similarity search.*

Bạn không phải tự tính distance bằng tay.  
*You do not have to compute distances by hand.*

### ✅ Self-check Phần 1
*✅ Self-check for Part 1*

1. Ba mô hình hỗ trợ vector của RDBMS là gì?  
   *1. What are the three models of vector support in an RDBMS?*

   Cho một ví dụ mỗi loại.  
   *Give one example of each.*

2. Ưu điểm lớn nhất của việc cơi nới RDBMS là gì?  
   *2. What is the biggest advantage of extending your RDBMS?*

   Bạn hãy so sánh với việc mua một vector DB riêng.  
   *Compare it with buying a separate vector DB.*

3. "In-database embedding" khác "bring-your-own embedding" ở chỗ nào?  
   *3. How does "in-database embedding" differ from "bring-your-own embedding"?*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

### 2.1. PostgreSQL + pgvector (mô hình extension)
*2.1. PostgreSQL + pgvector (the extension model)*

PostgreSQL là một RDBMS mã nguồn mở mạnh.  
*PostgreSQL is a powerful open source RDBMS.*

Uber, Netflix, Instagram và Spotify đều dùng nó.  
*Uber, Netflix, Instagram, and Spotify all use it.*

Bài gốc nêu các tên này.  
*The original lesson names these companies.*

PostgreSQL hỗ trợ vector qua **extension pgvector**.  
*PostgreSQL supports vectors through the **pgvector extension**.*

Bạn cài pgvector lên server.  
*You install pgvector on the server.*

Sau đó bạn có kiểu `vector`.  
*Then you get the `vector` type.*

> **[Đính chính bài gốc] — correction to the original lesson**
>
> Bài gốc nói PostgreSQL cung cấp `tsvector` như một kiểu dữ liệu vector.  
> *The original lesson says PostgreSQL offers `tsvector` as a vector data type.*
>
> Bạn hãy cẩn thận với câu này.  
> *Be careful with that claim.*
>
> `tsvector` phục vụ **full-text search**.  
> *`tsvector` serves **full-text search**.*
>
> `tsvector` là một danh sách lexeme.  
> *`tsvector` is a list of lexemes.*
>
> Bạn xem lại giáo trình FTS để rõ hơn.  
> *See the FTS course for more detail.*
>
> `tsvector` **không phải** kiểu để lưu embedding.  
> *`tsvector` is **not** the type for storing embeddings.*
>
> Kiểu lưu embedding là `vector`.  
> *The type for storing embeddings is `vector`.*
>
> **pgvector** cung cấp kiểu `vector`.  
> ***pgvector** provides the `vector` type.*
>
> Hai thứ này khác nhau hoàn toàn.  
> *These two things are completely different.*
>
> Bài gốc phân biệt đúng ở câu ngay sau đó.  
> *The original lesson makes the right distinction in the very next sentence.*
>
> Câu đó nói pgvector cho phép xử lý vector phi-text.  
> *That sentence says pgvector handles non-text vectors.*
>
> Câu trước vẫn dễ gây nhầm.  
> *The earlier sentence is still misleading.*

```sql
-- 1) Thêm extension (chỉ cần một lần mỗi database)
-- 1) Add the extension (only once per database)
CREATE EXTENSION IF NOT EXISTS vector;

-- 2) Bảng với cột vector; độ dài (512) = số chiều model tạo embedding
-- 2) A table with a vector column; the length (512) = the embedding model's dimension count
CREATE TABLE items (
    id        bigserial PRIMARY KEY,
    content   text,
    embedding vector(512)
);

-- 3) Index HNSW (khớp ops class với toán tử query)
-- 3) An HNSW index (match the ops class to the query operator)
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);

-- 4) Query k gần nhất
-- 4) Query the k nearest rows
SELECT id, content
FROM items
ORDER BY embedding <=> '[...512 số...]'
LIMIT 5;
```

**Đặc điểm / Characteristics**

pgvector đi theo hướng BYO-embedding.  
*pgvector follows the BYO-embedding route.*

pgvector không sinh vector.  
*pgvector does not generate vectors.*

Hybrid query ở đây rất mạnh.  
*Hybrid query here is very strong.*

Bạn join và filter với business data trong một câu SQL.  
*You join and filter against business data in one SQL statement.*

Hệ sinh thái managed rất rộng.  
*The managed ecosystem is very broad.*

Ví dụ là Supabase, Neon và RDS/Aurora.  
*Examples are Supabase, Neon, and RDS/Aurora.*

Bạn xem chi tiết ở giáo trình pgvector.  
*See the pgvector course for the details.*

### 2.2. MySQL & MySQL HeatWave (mô hình managed service)
*2.2. MySQL & MySQL HeatWave (the managed service model)*

Bài gốc tập trung vào **MySQL HeatWave**.  
*The original lesson focuses on **MySQL HeatWave**.*

HeatWave là dịch vụ *fully managed* của Oracle.  
*HeatWave is a *fully managed* service from Oracle.*

Nó gộp transaction, analytics và machine learning trong một service.  
*It combines transactions, analytics, and machine learning in one service.*

Phần transaction chính là OLTP.  
*The transaction part is exactly OLTP.*

Điểm khác biệt lớn nằm ở ba chỗ.  
*The big differences sit in three places.*

**In-database embedding (HeatWave GenAI)**

HeatWave có thể tự sinh embedding real-time từ text.  
*HeatWave can generate embeddings from text in real time.*

Nó dùng một LLM tích hợp.  
*It uses an integrated LLM.*

Bạn không cần gọi model bên ngoài.  
*You do not need to call an external model.*

Đây là differentiator so với pgvector và MariaDB.  
*This is a differentiator against pgvector and MariaDB.*

Hai nền tảng kia vốn theo BYO-embedding.  
*Those two platforms follow BYO-embedding.*

**Nơi chạy / Where it runs**

HeatWave chạy trên OCI — Oracle Cloud.  
*HeatWave runs on OCI, the Oracle Cloud.*

HeatWave cũng chạy trên AWS.  
*HeatWave also runs on AWS.*

Có một bản dùng thử giới hạn.  
*A limited trial edition is available.*

**Tài nguyên / Resources**

HeatWave cần tài nguyên lớn cho scalability.  
*HeatWave needs large resources for scalability.*

Nó cũng cần tài nguyên lớn cho throughput.  
*It also needs large resources for throughput.*

Vì thế nó hợp với môi trường cloud.  
*It therefore fits a cloud environment.*

```sql
-- Ví dụ (bám bài gốc): bảng T với cột val lưu vector 5 phần tử
-- Example (following the original lesson): table T with column val storing a 5-element vector
CREATE TABLE t (
    id  INT PRIMARY KEY,
    val VECTOR(5)
);

-- Chèn vector (MySQL 9.x dùng STRING_TO_VECTOR để parse text -> vector)
-- Insert a vector (MySQL 9.x uses STRING_TO_VECTOR to parse text -> vector)
INSERT INTO t (id, val) VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));

-- Đo distance (ANN index tối ưu có ở HeatWave)
-- Measure distance (the optimized ANN index lives in HeatWave)
SELECT id FROM t
ORDER BY DISTANCE(val, STRING_TO_VECTOR('[...]'), 'COSINE')
LIMIT 5;
```

> **[MỞ RỘNG — cập nhật] — extension and update**
>
> Đây là cảnh báo về MySQL.  
> *This is a warning about MySQL.*
>
> Community Server 9.0 trở lên có kiểu `VECTOR`.  
> *Community Server 9.0 and later has the `VECTOR` type.*
>
> Bản này cũng có distance function.  
> *This edition also has distance functions.*
>
> ANN vector *index* chủ yếu đến qua HeatWave.  
> *The ANN vector *index* comes mainly through HeatWave.*
>
> HeatWave là dịch vụ managed và proprietary.  
> *HeatWave is a managed, proprietary service.*
>
> Hệ sinh thái MySQL còn bị phân mảnh.  
> *The MySQL ecosystem is also fragmented.*
>
> **PlanetScale/Vitess** đã có vector index từ 3/2025.  
> ***PlanetScale/Vitess** has had a vector index since March 2025.*
>
> PlanetScale dùng thuật toán **SPANN**.  
> *PlanetScale uses the **SPANN** algorithm.*
>
> **Google Cloud SQL for MySQL** có vector index từ 2024.  
> ***Google Cloud SQL for MySQL** has had a vector index since 2024.*
>
> Nền tảng này dùng thuật toán **ScaNN**.  
> *This platform uses the **ScaNN** algorithm.*
>
> **TiDB** đang ở bản beta.  
> ***TiDB** is at the beta stage.*
>
> Amazon RDS for MySQL chưa có vector index.  
> *Amazon RDS for MySQL has no vector index yet.*
>
> Amazon RDS for MariaDB 11.8 thì đã có.  
> *Amazon RDS for MariaDB 11.8 does have one.*
>
> Bài gốc nói PlanetScale sắp giới thiệu vector.  
> *The original lesson says PlanetScale will soon introduce vectors.*
>
> Tính năng đó nay đã có.  
> *That feature exists today.*

### 2.3. MariaDB Vector (mô hình native)
*2.3. MariaDB Vector (the native model)*

MariaDB Server là một fork của MySQL.  
*MariaDB Server is a fork of MySQL.*

Chính những người tạo MySQL đã lập ra nó.  
*The original creators of MySQL founded it.*

**MariaDB Vector** đưa vector similarity search thẳng vào lõi.  
***MariaDB Vector** puts vector similarity search straight into the core.*

Ở đây không có extension.  
*There is no extension here.*

Ở đây cũng không có plugin.  
*There is no plugin here either.*

> **[Đính chính quan trọng] — an important correction**
>
> Bài gốc nói MariaDB lưu primary data và vectorized data *riêng biệt*.  
> *The original lesson says MariaDB stores primary data and vectorized data *separately*.*
>
> Bài gốc nói dữ liệu vector nằm ở một database *đặc biệt*.  
> *The original lesson says the vector data lives in a *special* database.*
>
> **Bản GA không như vậy.**  
> ***The GA release does not work this way.***
>
> Embedding được lưu trong cột `VECTOR(n)` ngay trong bảng bình thường.  
> *The embedding is stored in a `VECTOR(n)` column inside an ordinary table.*
>
> Cột đó nằm cạnh dữ liệu nghiệp vụ.  
> *That column sits next to the business data.*
>
> Bạn search bằng SQL.  
> *You search with SQL.*
>
> Đây đúng triết lý "một database cho tất cả".  
> *This matches the "one database for everything" philosophy.*
>
> Đây là một *ưu điểm*.  
> *This is an *advantage*.*
>
> Đây không phải sự tách rời.  
> *This is not a separation.*
>
> Mô tả cũ có thể ứng với bản preview.  
> *The old description may apply to the preview release.*
>
> Mô tả cũ cũng có thể ứng với thiết kế sớm.  
> *The old description may also apply to an early design.*
>
> Mọi thứ nay đã khác.  
> *Everything is different today.*

```sql
-- Native: KHÔNG cần CREATE EXTENSION / INSTALL PLUGIN
-- Native: NO need for CREATE EXTENSION / INSTALL PLUGIN
CREATE TABLE chunks (
    id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    content   TEXT NOT NULL,
    embedding VECTOR(1536) NOT NULL,
    VECTOR INDEX (embedding) DISTANCE=cosine       -- index HNSW, chọn metric ngay
                                                   -- an HNSW index, with the metric chosen right here
) ENGINE=InnoDB;

-- Chèn: VEC_FromText parse chuỗi -> vector
-- Insert: VEC_FromText parses a string -> vector
INSERT INTO chunks (content, embedding)
VALUES ('MariaDB lưu embedding native', VEC_FromText('[...1536 số...]'));

-- Query: BẮT BUỘC có ORDER BY VEC_DISTANCE_*() + LIMIT thì index mới được dùng
-- Query: ORDER BY VEC_DISTANCE_*() plus LIMIT is REQUIRED for the index to be used
SELECT id, content
FROM chunks
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 3;
```

**Đặc điểm (cập nhật 2026) / Characteristics (updated 2026)**

MariaDB Vector đạt GA từ bản **11.8 LTS**.  
*MariaDB Vector reached GA in version **11.8 LTS**.*

Tính năng này nằm trong **Community Server** miễn phí.  
*This feature ships in the free **Community Server**.*

Nó không giới hạn ở bản enterprise.  
*It is not limited to the enterprise edition.*

Điểm này khác MySQL.  
*This point differs from MySQL.*

MySQL đặt index trong HeatWave proprietary.  
*MySQL keeps the index inside the proprietary HeatWave.*

MariaDB dùng **HNSW**.  
*MariaDB uses **HNSW**.*

MariaDB hỗ trợ **cosine** và **Euclidean**.  
*MariaDB supports **cosine** and **Euclidean**.*

Số chiều tối đa là **16.383**.  
*The maximum dimension count is **16,383**.*

Tính năng này có trên **Amazon RDS for MariaDB 11.8**.  
*This feature is available on **Amazon RDS for MariaDB 11.8**.*

MariaDB tối ưu **SIMD**.  
*MariaDB optimizes for **SIMD**.*

Các tập lệnh gồm AVX2, AVX512 và NEON.  
*The instruction sets include AVX2, AVX512, and NEON.*

MariaDB **không** tự sinh embedding.  
*MariaDB does **not** generate embeddings itself.*

Nó theo hướng BYO giống pgvector.  
*It follows the BYO route like pgvector.*

MariaDB tích hợp LangChain, LlamaIndex và Spring AI.  
*MariaDB integrates with LangChain, LlamaIndex, and Spring AI.*

### 2.4. Bảng so sánh nhanh ba nền tảng
*2.4. Quick comparison of the three platforms*

| | PostgreSQL + pgvector | MySQL HeatWave | MariaDB Vector |
|---|---|---|---|
| Mô hình / Model | Extension / Extension | Managed service / Managed service | **Native** (trong lõi) / **Native** (in the core) |
| Cần cài thêm? / Extra install needed? | `CREATE EXTENSION vector` / `CREATE EXTENSION vector` | Không (managed) / No (managed) | **Không** (built-in) / **No** (built-in) |
| Mã nguồn mở & miễn phí? / Open source and free? | Có / Yes | Không (proprietary/cloud) / No (proprietary/cloud) | **Có** (Community) / **Yes** (Community) |
| Sinh embedding trong DB? / Embedding generation in the DB? | Không (BYO) / No (BYO) | **Có (HeatWave GenAI)** / **Yes (HeatWave GenAI)** | Không (BYO) / No (BYO) |
| Index / Index | HNSW, IVFFlat, (DiskANN) / HNSW, IVFFlat, (DiskANN) | HNSW (HeatWave) / HNSW (HeatWave) | HNSW / HNSW |
| Distance / Distance | cosine/L2/ip / cosine/L2/ip | cosine/L2... / cosine/L2... | cosine/L2 / cosine/L2 |
| Self-host được? / Self-hostable? | **Có** / **Yes** | Không (cloud-only) / No (cloud-only) | **Có** / **Yes** |
| Hybrid query (vector + SQL) / Hybrid query (vector + SQL) | **Có, 1 câu** / **Yes, one statement** | Có / Yes | **Có, 1 câu** / **Yes, one statement** |

### 2.5. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp
*2.5. [MỞ RỘNG NGOÀI BÀI GỐC] — three common mistakes*

**Lỗi 1 — MariaDB: quên `LIMIT` nên index không được dùng**  
*Mistake 1 — MariaDB: forgetting `LIMIT` leaves the index unused*

Vector index của MariaDB chỉ kích hoạt trong một điều kiện.  
*The MariaDB vector index activates under one condition only.*

Câu query phải có `ORDER BY VEC_DISTANCE_*()`.  
*The query must contain `ORDER BY VEC_DISTANCE_*()`.*

Câu query cũng phải có `LIMIT`.  
*The query must also contain `LIMIT`.*

Thiếu `LIMIT` sẽ dẫn tới full scan.  
*A missing `LIMIT` leads to a full scan.*

pgvector cũng ưa có `LIMIT` để tối ưu.  
*pgvector also prefers a `LIMIT` for optimization.*

**Lỗi 2 — Trộn embedding từ hai model khác nhau trong cùng một cột**  
*Mistake 2 — mixing embeddings from two different models in one column*

Model A tạo vector 512 chiều.  
*Model A produces 512-dimension vectors.*

Model B tạo vector 1536 chiều.  
*Model B produces 1536-dimension vectors.*

Hai loại vector này không cùng một không gian.  
*These two kinds of vectors do not share a space.*

Distance giữa chúng trở nên vô nghĩa.  
*The distance between them becomes meaningless.*

Lệch chiều còn gây lỗi insert.  
*A dimension mismatch also causes an insert error.*

Toàn bộ cột phải dùng cùng một model.  
*The whole column must use the same model.*

Toàn bộ cột phải dùng cùng một version.  
*The whole column must use the same version.*

Điều này đúng cho cả ba nền tảng.  
*This holds for all three platforms.*

Đây là chỗ nối lại với bài embeddings.  
*This is the link back to the embeddings lesson.*

**Lỗi 3 — Nhầm `tsvector` với `vector`**  
*Mistake 3 — confusing `tsvector` with `vector`*

`tsvector` thuộc về full-text search của Postgres.  
*`tsvector` belongs to Postgres full-text search.*

`vector` thuộc về embedding.  
*`vector` belongs to embeddings.*

Semantic search cần kiểu `vector` của pgvector.  
*Semantic search needs the `vector` type from pgvector.*

Semantic search không dùng `tsvector`.  
*Semantic search does not use `tsvector`.*

`tsvector` làm việc ở tầng keyword và lexeme.  
*`tsvector` operates at the keyword and lexeme level.*

Nhiều người đọc lướt bài gốc.  
*Many people skim the original lesson.*

Họ tưởng Postgres đã sẵn có vector search qua `tsvector`.  
*They assume Postgres already has vector search through `tsvector`.*

Điều đó không đúng.  
*That is not true.*

### ✅ Self-check Phần 2
*✅ Self-check for Part 2*

1. Khác biệt then chốt giữa MySQL HeatWave và MariaDB Vector là gì?  
   *1. What is the key difference between MySQL HeatWave and MariaDB Vector?*

   Bạn hãy xét hai trục "sinh embedding" và "mã nguồn mở".  
   *Consider the two axes of embedding generation and open source.*

2. Trên MariaDB, điều kiện để vector index được dùng khi query là gì?  
   *2. On MariaDB, what condition lets a query use the vector index?*

3. `tsvector` và `vector` trong PostgreSQL khác nhau thế nào?  
   *3. How do `tsvector` and `vector` differ in PostgreSQL?*

   Cái nào phục vụ semantic search?  
   *Which one serves semantic search?*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED (In depth)*

### 3.1. Ba mô hình kiến trúc — đánh đổi dưới nắp capo
*3.1. Three architecture models — the trade-offs under the hood*

**Extension (pgvector)**

*Ưu điểm / Advantages*

pgvector linh hoạt tối đa.  
*pgvector is maximally flexible.*

pgvector là mã nguồn mở.  
*pgvector is open source.*

pgvector cập nhật độc lập với Postgres core.  
*pgvector updates independently of the Postgres core.*

pgvector có nhiều loại index.  
*pgvector has many index types.*

Các loại đó gồm HNSW, IVFFlat và DiskANN qua pgvectorscale.  
*Those types include HNSW, IVFFlat, and DiskANN through pgvectorscale.*

Hệ sinh thái managed của nó rất rộng.  
*Its managed ecosystem is very broad.*

*Nhược điểm / Drawbacks*

Bạn phải cài pgvector.  
*You must install pgvector.*

Bạn phải quản version của pgvector.  
*You must manage the pgvector version.*

Tính năng phụ thuộc vào phiên bản extension.  
*The available features depend on the extension version.*

Một số managed provider khóa version cũ.  
*Some managed providers pin an old version.*

**Native (MariaDB)**

*Ưu điểm / Advantages*

Bạn không phải cài gì.  
*You install nothing.*

Điều này giảm ma sát vận hành.  
*This reduces operational friction.*

MariaDB tối ưu sâu ở tầng engine.  
*MariaDB optimizes deeply at the engine layer.*

Ví dụ là SIMD với AVX và NEON.  
*One example is SIMD with AVX and NEON.*

Tính năng đi cùng release của database.  
*Features ship with the database release.*

*Nhược điểm / Drawbacks*

Bạn bị buộc vào nhịp release của database.  
*You are tied to the database release cadence.*

MariaDB có ít "món" hơn extension.  
*MariaDB offers fewer options than an extension.*

Ví dụ là chỉ có HNSW với cosine và L2.  
*For example, only HNSW with cosine and L2 is available.*

**Managed (HeatWave)**

*Ưu điểm / Advantages*

Bạn không vận hành hạ tầng.  
*You do not operate the infrastructure.*

HeatWave có **in-database embedding**.  
*HeatWave has **in-database embedding**.*

Bạn khỏi gọi model bên ngoài.  
*You avoid calling an external model.*

HeatWave gộp OLTP, analytics và ML.  
*HeatWave combines OLTP, analytics, and ML.*

*Nhược điểm / Drawbacks*

HeatWave gây **vendor lock-in**.  
*HeatWave creates **vendor lock-in**.*

Nó chạy trên OCI và AWS.  
*It runs on OCI and AWS.*

Nó là dịch vụ proprietary.  
*It is a proprietary service.*

Chi phí tính theo dịch vụ.  
*Cost is charged per service.*

Bạn không self-host được.  
*You cannot self-host it.*

Việc rời đi rất khó.  
*Leaving is hard.*

### 3.2. Thuật toán index — không phải ai cũng HNSW
*3.2. Index algorithms — not everyone uses HNSW*

Bài gốc chỉ nhắc IVFFlat và HNSW của pgvector.  
*The original lesson mentions only IVFFlat and HNSW from pgvector.*

Ở tầm landscape 2026, các nền tảng dùng thuật toán khác nhau.  
*Across the 2026 landscape, platforms use different algorithms.*

| Nền tảng / Platform | Thuật toán ANN / ANN algorithm |
|---|---|
| pgvector, MariaDB, phần lớn MySQL forks / pgvector, MariaDB, most MySQL forks | **HNSW** (graph nhiều tầng) / **HNSW** (a multi-layer graph) |
| PlanetScale (Vitess) / PlanetScale (Vitess) | **SPANN** (Space-Partitioned ANN, lai cụm với graph, hợp disk) / **SPANN** (Space-Partitioned ANN, a cluster-graph hybrid suited to disk) |
| Google Cloud SQL for MySQL / Google Cloud SQL for MySQL | **ScaNN** (Google, quantization với phân vùng) / **ScaNN** (Google, quantization plus partitioning) |
| Timescale pgvectorscale / Timescale pgvectorscale | **DiskANN** (SSD-based) / **DiskANN** (SSD-based) |

**Ý nghĩa staff / What this means at staff level**

"Vector search" không phải một thứ đồng nhất.  
*"Vector search" is not a single uniform thing.*

Thuật toán index quyết định trade-off.  
*The index algorithm decides the trade-off.*

Trade-off đó gồm memory, disk, recall và tốc độ.  
*That trade-off covers memory, disk, recall, and speed.*

HNSW chạy in-RAM.  
*HNSW runs in RAM.*

HNSW cho recall cao.  
*HNSW gives high recall.*

SPANN và DiskANN thân thiện với disk.  
*SPANN and DiskANN are disk-friendly.*

Chúng tiết kiệm RAM ở quy mô lớn.  
*They save RAM at large scale.*

Bạn xem cơ chế HNSW và IVFFlat ở giáo trình indexing.  
*See the indexing course for the mechanics of HNSW and IVFFlat.*

### 3.3. Giới hạn chiều & SIMD — chi tiết kỹ thuật hay bị bỏ qua
*3.3. Dimension limits and SIMD — technical details people often skip*

**Giới hạn chiều / Dimension limits**

Index HNSW và IVFFlat của pgvector tối đa **2000 chiều**.  
*The HNSW and IVFFlat indexes in pgvector top out at **2000 dimensions**.*

Bạn dùng `halfvec` cho số chiều cao hơn.  
*Use `halfvec` for higher dimension counts.*

Kiểu `VECTOR` của MariaDB đạt tới **16.383 chiều**.  
*The `VECTOR` type in MariaDB reaches **16,383 dimensions**.*

Model OpenAI 3-large có 3072 chiều.  
*The OpenAI 3-large model has 3072 dimensions.*

Với model đó, khác biệt này rất quan trọng.  
*For that model, this difference matters a lot.*

**SIMD**

MariaDB Vector tự dùng lệnh vector hóa phần cứng.  
*MariaDB Vector automatically uses hardware vectorization instructions.*

Trên Intel, đó là AVX2 và AVX512.  
*On Intel, those are AVX2 and AVX512.*

Trên ARM, đó là NEON.  
*On ARM, that is NEON.*

Các lệnh này tính distance rất nhanh.  
*These instructions compute distances very fast.*

Bạn không cần cấu hình gì.  
*You need no configuration.*

Đây là một tối ưu low-level.  
*This is a low-level optimization.*

Một số extension chưa có tối ưu này mặc định.  
*Some extensions lack this optimization by default.*

**Hệ quả / Consequence**

Các nền tảng đều nói "hỗ trợ vector".  
*Every platform claims "vector support".*

Chi tiết engine vẫn tạo ra khác biệt hiệu năng thật.  
*Engine details still create real performance differences.*

Chi tiết đó gồm giới hạn chiều, SIMD và thuật toán.  
*Those details include dimension limits, SIMD, and the algorithm.*

### 3.4. Đính chính "lưu riêng biệt" & vì sao "cùng bảng" là ưu điểm
*3.4. Correcting "separate storage" and why "same table" is an advantage*

Bài gốc mô tả MariaDB lưu vector ở một database riêng.  
*The original lesson describes MariaDB storing vectors in a separate database.*

Thực tế bản GA lưu vector **cùng bảng**.  
*In reality the GA release stores vectors in the **same table**.*

Đó là một điểm mạnh.  
*That is a strength.*

Bạn chạy được **hybrid query** trong một câu.  
*You can run a **hybrid query** in a single statement.*

```sql
-- Sản phẩm giống nhất, NHƯNG còn hàng và dưới 50$ — một câu, một transaction
-- The most similar products, BUT in stock and under $50 — one statement, one transaction
SELECT id, name FROM products
WHERE in_stock = 1 AND price < 50
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]'))
LIMIT 10;
```

Bạn có thể tách vector sang một hệ riêng.  
*You can split vectors into a separate system.*

Hệ riêng đó là một dedicated vector DB.  
*That separate system is a dedicated vector DB.*

Khi đó bạn query vector trước.  
*In that case you query the vectors first.*

Bạn nhận về một danh sách IDs.  
*You receive a list of IDs.*

Bạn query lại DB nghiệp vụ để lọc giá và tồn kho.  
*You query the business DB again to filter by price and stock.*

Bạn phải nuôi hai hệ thống.  
*You must maintain two systems.*

Bạn tốn hai round-trip.  
*You spend two round-trips.*

Việc đồng bộ trở nên phức tạp.  
*Synchronization becomes complex.*

Vector nằm cùng bảng với business data.  
*The vectors live in the same table as the business data.*

Đây chính là lý do RDBMS-native hấp dẫn.  
*This is exactly why RDBMS-native is attractive.*

### 3.5. Edge cases
*3.5. Edge cases*

**Đồng bộ khi tách hệ / Synchronization when systems are split**

Đôi khi bạn buộc phải dùng một vector DB riêng.  
*Sometimes you are forced to use a separate vector DB.*

Mỗi lệnh create phải sync sang hệ vector.  
*Every create must sync to the vector system.*

Mỗi lệnh update phải sync sang hệ vector.  
*Every update must sync to the vector system.*

Mỗi lệnh delete phải sync sang hệ vector.  
*Every delete must sync to the vector system.*

Đây là nguồn bug kinh điển.  
*This is a classic source of bugs.*

Dữ liệu hai bên dễ lệch nhau.  
*The data on the two sides drifts apart easily.*

**Distance metric khác nhau giữa các nền tảng / Distance metrics differ across platforms**

Các nền tảng đều nói "cosine".  
*Every platform says "cosine".*

Có nền tảng trả về *distance*.  
*Some platforms return a *distance*.*

Với distance, số nhỏ nghĩa là gần.  
*With a distance, a small number means close.*

Có nền tảng trả về *similarity*.  
*Other platforms return a *similarity*.*

Với similarity, số lớn nghĩa là gần.  
*With a similarity, a large number means close.*

Bạn đọc kỹ tài liệu.  
*Read the documentation carefully.*

Nếu không, `ORDER BY` của bạn sẽ ngược.  
*Otherwise your `ORDER BY` will run backwards.*

**Chiều vượt giới hạn index / Dimensions beyond the index limit**

Bạn vẫn insert được vector đó.  
*You can still insert that vector.*

Database vẫn lưu được nó.  
*The database still stores it.*

Database không index được nó.  
*The database cannot index it.*

Query sẽ âm thầm chuyển sang seq scan.  
*The query silently falls back to a sequential scan.*

Seq scan rất chậm.  
*A sequential scan is very slow.*

**Managed lock-in / Managed lock-in**

Dữ liệu và embedding nằm ở HeatWave.  
*Your data and embeddings live inside HeatWave.*

Việc bê nguyên chúng sang nơi khác rất khó.  
*Moving them elsewhere as-is is hard.*

Bạn hãy tính chi phí rời đi trước khi cam kết.  
*Estimate the exit cost before you commit.*

### ✅ Self-check Phần 3
*✅ Self-check for Part 3*

1. Nêu một ưu điểm và một nhược điểm của mỗi mô hình.  
   *1. Name one advantage and one drawback of each model.*

   Ba mô hình là extension, native và managed.  
   *The three models are extension, native, and managed.*

2. Không phải nền tảng nào cũng dùng HNSW.  
   *2. Not every platform uses HNSW.*

   Bạn hãy kể 2 thuật toán ANN khác.  
   *Name two other ANN algorithms.*

   Bạn hãy nêu nền tảng dùng chúng.  
   *Name the platforms that use them.*

3. Vì sao "vector cùng bảng với business data" tốt hơn cho hybrid query?  
   *3. Why is "vectors in the same table as business data" better for hybrid queries?*

   Bạn hãy so sánh với phương án tách hệ riêng.  
   *Compare it with the split-system option.*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL (Systems thinking and technical leadership)*

### 4.1. Khung quyết định: RDBMS-native vs Dedicated Vector DB
*4.1. Decision framework: RDBMS-native vs a dedicated vector DB*

Đây là một quyết định kiến trúc thật.  
*This is a real architectural decision.*

Bài gốc không chạm tới quyết định này.  
*The original lesson does not touch this decision.*

Bài gốc chỉ liệt kê các RDBMS.  
*The original lesson only lists RDBMSs.*

Staff phải biết cả phương án dedicated.  
*A staff engineer must also know the dedicated option.*

Ví dụ là Pinecone, Weaviate, Qdrant và Milvus.  
*Examples are Pinecone, Weaviate, Qdrant, and Milvus.*

Staff phải biết khi nào chọn cái nào.  
*A staff engineer must know when to pick which.*

**Chọn RDBMS-native khi / Choose RDBMS-native when**

Các lựa chọn ở đây là pgvector, MariaDB và HeatWave.  
*The options here are pgvector, MariaDB, and HeatWave.*

Vector chỉ là *một tính năng trong* app đã chạy RDBMS.  
*Vectors are just *one feature inside* an app already running an RDBMS.*

Bạn cần hybrid query trong một transaction ACID.  
*You need a hybrid query inside one ACID transaction.*

Hybrid query đó gồm vector cùng filter và join business data.  
*That hybrid query combines vectors with filters and joins on business data.*

Số vector dưới khoảng 10–50 triệu.  
*The vector count is under roughly 10–50 million.*

Ngưỡng này bao trùm đa số RAG và search B2B.  
*This threshold covers most B2B RAG and search workloads.*

Bạn muốn ít hệ thống.  
*You want fewer systems.*

Bạn muốn ít chi phí.  
*You want lower cost.*

Đội hiện tại tự vận hành được.  
*Your current team can operate it.*

**Chọn Dedicated Vector DB khi / Choose a dedicated vector DB when**

Vector search **là sản phẩm chính**.  
*Vector search **is the main product**.*

Vector search không phải tính năng phụ.  
*Vector search is not a side feature.*

Số vector vượt khoảng 50 triệu tới hàng tỷ.  
*The vector count exceeds roughly 50 million up to billions.*

Bạn cần globally distributed multi-region.  
*You need a globally distributed multi-region setup.*

Bạn cần tính năng chuyên sâu.  
*You need specialized features.*

Ví dụ là hybrid dense cộng sparse built-in.  
*One example is built-in hybrid dense plus sparse.*

Ví dụ khác là multi-tenancy nâng cao.  
*Another example is advanced multi-tenancy.*

Ví dụ khác nữa là filtering phức tạp ở quy mô lớn.  
*Yet another example is complex filtering at large scale.*

Tổ chức không dùng một RDBMS phù hợp.  
*Your organization does not run a suitable RDBMS.*

Hoặc tổ chức muốn tách hoàn toàn workload vector.  
*Or your organization wants to fully isolate the vector workload.*

> **One-liner staff — the staff-level one-liner**
>
> Đặt vector cạnh business data thắng ở operational simplicity.  
> *Putting vectors next to business data wins on operational simplicity.*
>
> Tách ra một vector DB riêng thắng ở peak scale.  
> *Splitting into a separate vector DB wins at peak scale.*
>
> Bạn chọn theo một câu hỏi duy nhất.  
> *You choose by a single question.*
>
> Vector là feature hay là product?  
> *Are vectors a feature or the product?*

### 4.2. Managed vs Self-host — trục quyết định thứ hai
*4.2. Managed vs self-host — the second decision axis*

| | Self-host (pgvector, MariaDB Community) | Managed (HeatWave, Supabase, Neon, RDS) |
|---|---|---|
| Vận hành / Operations | tự lo patch, backup, scale / you handle patching, backup, scaling | nhà cung cấp lo / the provider handles it |
| Chi phí / Cost | rẻ về license, tốn nhân lực / cheap on licensing, costly in staff time | trả theo dịch vụ / pay per service |
| Kiểm soát & dữ liệu / Control and data | **toàn quyền, dữ liệu ở nhà** / **full control, data stays in house** | tiện nhưng dữ liệu ở cloud bên thứ ba / convenient but data sits in a third-party cloud |
| Lock-in / Lock-in | thấp (mã nguồn mở) / low (open source) | **cao** (đặc biệt HeatWave proprietary) / **high** (especially proprietary HeatWave) |
| Tính năng ăn liền / Out-of-the-box features | tự lắp / you assemble them yourself | in-DB embedding, auto-scale sẵn / in-DB embedding and auto-scale included |

Ràng buộc privacy và compliance đôi khi *ép* bạn self-host.  
*Privacy and compliance constraints sometimes *force* you to self-host.*

Lý do là dữ liệu khách hàng không được rời hệ thống.  
*The reason is that customer data may not leave the system.*

Ràng buộc này thắng mọi tiện lợi của managed.  
*This constraint outweighs every convenience of managed services.*

Staff phải nêu ràng buộc này thật sớm.  
*A staff engineer must raise this constraint very early.*

Đây là chỗ nối lại luận điểm ở giáo trình embeddings.  
*This links back to the argument in the embeddings course.*

### 4.3. Lock-in & migration — chi phí ẩn lớn nhất
*4.3. Lock-in and migration — the largest hidden cost*

**In-database embedding tiện nhưng khóa chặt / In-database embedding is convenient but binding**

HeatWave GenAI sinh embedding bằng model của Oracle.  
*HeatWave GenAI generates embeddings with Oracle's model.*

Bạn đổi sang nền tảng khác.  
*You move to another platform.*

Khi đó bạn phải re-embed toàn bộ dữ liệu bằng model khác.  
*You must then re-embed all your data with a different model.*

Vector từ hai model không tương thích về không gian.  
*Vectors from two models are not compatible in space.*

BYO-embedding linh hoạt hơn nhiều.  
*BYO-embedding is far more flexible.*

pgvector và MariaDB đi theo hướng này.  
*pgvector and MariaDB follow this route.*

Bạn kiểm soát model của mình.  
*You control your own model.*

**Đổi RDBMS là một migration nặng / Switching RDBMS is a heavy migration**

Bạn không chỉ chuyển data.  
*You do not just move data.*

Bạn phải chuyển cả cú pháp.  
*You must also convert the syntax.*

pgvector dùng toán tử `<=>`.  
*pgvector uses the `<=>` operator.*

MariaDB dùng `VEC_DISTANCE_COSINE`.  
*MariaDB uses `VEC_DISTANCE_COSINE`.*

MySQL dùng `DISTANCE()`.  
*MySQL uses `DISTANCE()`.*

Bạn phải dựng lại index.  
*You must rebuild the indexes.*

Bạn có thể phải re-embed.  
*You may also have to re-embed.*

**Chiến lược / Strategy**

Bạn ưu tiên chuẩn mở.  
*Prefer open standards.*

Bạn ưu tiên BYO-embedding.  
*Prefer BYO-embedding.*

Hai điều này giữ cho bạn đường lui.  
*These two keep an exit path open.*

Bạn version hóa embedding.  
*Version your embeddings.*

Bạn tránh phụ thuộc tính năng độc quyền của một vendor.  
*Avoid depending on a single vendor's proprietary features.*

Lời khuyên này áp dụng khi tính di động quan trọng.  
*This advice applies when portability matters.*

### 4.4. Cost, reliability, monitoring
*4.4. Cost, reliability, monitoring*

**Cost drivers — các yếu tố đẩy chi phí**

Yếu tố thứ nhất là hạ tầng.  
*The first factor is infrastructure.*

HNSW in-memory ngốn nhiều RAM.  
*In-memory HNSW consumes a lot of RAM.*

Yếu tố thứ hai là phí managed service theo mức dùng.  
*The second factor is usage-based managed service fees.*

Yếu tố thứ ba là phí sinh embedding.  
*The third factor is the cost of generating embeddings.*

Với in-DB embedding, phí này gộp vào phí dịch vụ.  
*With in-DB embedding, this cost folds into the service fee.*

Yếu tố thứ tư là việc re-embed khi đổi model.  
*The fourth factor is re-embedding when you change models.*

**Reliability — độ tin cậy**

RDBMS-native thừa hưởng HA của database mẹ.  
*RDBMS-native inherits the HA of the parent database.*

Nó cũng thừa hưởng backup và replication trưởng thành.  
*It also inherits mature backup and replication.*

Đây là lợi thế lớn so với các vector DB non trẻ.  
*This is a big advantage over young vector DBs.*

Bạn chỉ trông một hệ thống.  
*You watch only one system.*

Bạn không phải trông hai hệ thống.  
*You do not watch two systems.*

**Monitoring — giám sát**

Chỉ số thứ nhất là recall@k.  
*The first metric is recall@k.*

Bạn so kết quả exact với kết quả từ index.  
*Compare exact results against index results.*

Chỉ số thứ hai là latency p95 và p99.  
*The second metric is p95 and p99 latency.*

Chỉ số thứ ba là index size cùng mức RAM.  
*The third metric is index size and RAM usage.*

Chỉ số thứ tư chỉ áp dụng khi bạn tách hệ.  
*The fourth metric applies only when systems are split.*

Đó là độ trễ đồng bộ giữa RDBMS và vector DB.  
*It is the sync lag between the RDBMS and the vector DB.*

Đó cũng là độ lệch dữ liệu giữa hai hệ.  
*It is also the data drift between the two systems.*

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder
*4.5. Organizational impact and explaining it to non-technical stakeholders*

**Nói với PM hoặc sếp / Talking to a PM or a manager**

"Chúng ta có thể thêm tìm-kiếm-theo-nghĩa.  
*"We can add meaning-based search.*

Chúng ta *không dựng hệ thống mới*.  
*We *build no new system*.*

Chúng ta cắm nó vào database sẵn có.  
*We plug it into the database we already have.*

Database đó là Postgres hoặc MariaDB.  
*That database is Postgres or MariaDB.*

Đội hiện tại vận hành được.  
*The current team can operate it.*

Rủi ro và chi phí đều thấp.  
*Risk and cost are both low.*

Dữ liệu ở nguyên một chỗ.  
*The data stays in one place.*

Chúng ta cũng có thể chọn dịch vụ managed như HeatWave.  
*We could also pick a managed service like HeatWave.*

Cách đó nhanh và có sẵn AI.  
*That path is fast and comes with AI built in.*

Cách đó khóa chúng ta vào một nhà cung cấp.  
*That path locks us into one vendor.*

Dữ liệu sẽ ra cloud của họ.  
*Our data goes into their cloud.*

Sau này vector search có thể thành một workload khổng lồ.  
*Later, vector search may become a huge workload.*

Lúc đó chúng ta cân nhắc một hệ chuyên dụng.  
*At that point we consider a dedicated system.*

Đó là bài toán của thành công."  
*That is a problem of success."*

Bạn framing câu chuyện bằng bốn trục.  
*Frame the story along four axes.*

Bốn trục đó là **rủi ro, chi phí, lock-in và privacy**.  
*Those four axes are **risk, cost, lock-in, and privacy**.*

**Team topology — cấu trúc đội**

RDBMS-native nghĩa là *một* đội database lo tất.  
*RDBMS-native means *one* database team handles everything.*

Việc tách vector DB riêng thường kéo theo nhu cầu kỹ năng mới.  
*Splitting off a vector DB usually brings a need for new skills.*

Nó cũng kéo theo một lịch on-call mới.  
*It also brings a new on-call rotation.*

Đây là một lập luận tổ chức.  
*This is an organizational argument.*

Đây không chỉ là lập luận kỹ thuật.  
*This is not merely a technical argument.*

**Chuẩn hóa toàn công ty / Company-wide standardization**

Nhiều team trong công ty đều cần vector.  
*Many teams in the company need vectors.*

Bạn nên chọn *một* mô hình chung.  
*You should pick *one* shared model.*

Ví dụ là pgvector cho mọi service dùng Postgres.  
*One example is pgvector for every Postgres-based service.*

Cách này giảm phân mảnh.  
*This approach reduces fragmentation.*

Mỗi team một vector DB gây phân mảnh nhiều hơn.  
*One vector DB per team causes much more fragmentation.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff
*4.6. A sample system design question with a staff-level answer*

> **Câu hỏi — the question**
>
> Công ty bạn chạy PostgreSQL cho core app.  
> *Your company runs PostgreSQL for the core app.*
>
> Bạn cần thêm semantic search cho 20 triệu tài liệu.  
> *You need to add semantic search over 20 million documents.*
>
> Dữ liệu khách hàng rất nhạy cảm.  
> *The customer data is highly sensitive.*
>
> Dữ liệu đó không được gửi ra API bên ngoài.  
> *That data may not be sent to any external API.*
>
> Team backend của bạn khá nhỏ.  
> *Your backend team is quite small.*
>
> Bạn chọn nền tảng vector thế nào?  
> *How do you choose the vector platform?*

**Khung trả lời staff / The staff-level answer framework**

Bạn nói to mọi trade-off.  
*Say every trade-off out loud.*

1. **Clarify — làm rõ đề bài**  
   *1. **Clarify** the problem.*

   Bạn hỏi về QPS.  
   *Ask about QPS.*

   Bạn hỏi về recall target.  
   *Ask about the recall target.*

   Bạn hỏi về tốc độ tăng trưởng dữ liệu.  
   *Ask about data growth.*

   Bạn hỏi về ràng buộc compliance cụ thể.  
   *Ask about the specific compliance constraints.*

   Bạn hỏi về ngân sách.  
   *Ask about the budget.*

2. **Loại phương án theo ràng buộc — eliminate options by constraint**  
   *2. **Eliminate** options using the constraints.*

   Dữ liệu nhạy cảm cộng team nhỏ tạo ra hai ràng buộc cứng.  
   *Sensitive data plus a small team creates two hard constraints.*

   Bạn loại managed proprietary như HeatWave.  
   *Rule out proprietary managed services like HeatWave.*

   Bạn loại luôn API embedding bên ngoài.  
   *Rule out external embedding APIs as well.*

   Bạn nghiêng về self-host pgvector.  
   *Lean toward self-hosted pgvector.*

   Bạn ghép nó với một model embedding self-host.  
   *Pair it with a self-hosted embedding model.*

   Cách này giữ dữ liệu ở nhà.  
   *This keeps the data in house.*

   Cách này tận dụng Postgres sẵn có.  
   *This reuses the Postgres you already run.*

   Team đã biết Postgres rồi.  
   *The team already knows Postgres.*

3. **Vì sao không dùng dedicated vector DB — why not a dedicated vector DB**  
   *3. **Why not** a dedicated vector DB.*

   20 triệu vector nằm dưới ngưỡng cần Pinecone hoặc Milvus.  
   *20 million vectors sit below the threshold for Pinecone or Milvus.*

   Vector ở đây là một feature.  
   *Vectors here are a feature.*

   Vector ở đây không phải product.  
   *Vectors here are not the product.*

   Bạn tránh thêm một hệ thống.  
   *You avoid adding another system.*

   Bạn tránh luôn gánh nặng đồng bộ.  
   *You also avoid the synchronization burden.*

   Hybrid query làm ngay trong SQL.  
   *The hybrid query happens right inside SQL.*

   Query đó lọc theo tenant và theo quyền.  
   *That query filters by tenant and by permission.*

4. **Kiến trúc — the architecture**  
   *4. **The architecture**.*

   Bạn dùng cột `vector` với index HNSW.  
   *Use a `vector` column with an HNSW index.*

   Embedding sinh bởi một model self-host.  
   *A self-hosted model generates the embeddings.*

   Ví dụ là BGE-M3 hoặc MiniLM.  
   *Examples are BGE-M3 or MiniLM.*

   Bạn ghép thêm FTS khi cần tìm theo keyword.  
   *Add FTS when keyword search is needed.*

   Bạn partition dữ liệu theo tenant.  
   *Partition the data by tenant.*

5. **Migration và lock-in — migration and lock-in**  
   *5. **Migration and lock-in**.*

   Bạn dùng BYO-embedding cùng chuẩn mở.  
   *Use BYO-embedding together with open standards.*

   Cách này giữ đường lui.  
   *This keeps an exit path open.*

   Bạn version hóa embedding.  
   *Version your embeddings.*

6. **Khi nào đổi ý — when to change your mind**  
   *6. **When to change your mind**.*

   Dữ liệu có thể tăng tới hàng trăm triệu vector.  
   *The data may grow to hundreds of millions of vectors.*

   Hệ thống có thể cần chạy đa vùng.  
   *The system may need to run multi-region.*

   Lúc đó bạn cân nhắc tách sang hệ dedicated.  
   *At that point consider splitting to a dedicated system.*

   Bạn nêu rõ *ngưỡng* kích hoạt quyết định đó.  
   *State the *threshold* that triggers that decision.*

7. **Chốt — the closing statement**  
   *7. **The closing statement**.*

   "Tôi chọn RDBMS-native self-host.  
   *"I choose self-hosted RDBMS-native.*

   Lý do thứ nhất là ràng buộc *privacy*.  
   *The first reason is the *privacy* constraint.*

   Lý do thứ hai là *quy mô hiện tại*.  
   *The second reason is the *current scale*.*

   Nó không phải lựa chọn luôn tốt nhất.  
   *It is not always the best choice.*

   Tôi biết ngưỡng nào sẽ khiến tôi đổi."  
   *I know the threshold that would make me switch."*

   Bạn biết lý do của lựa chọn.  
   *You know the reason for your choice.*

   Bạn biết giới hạn của lựa chọn.  
   *You know the limits of your choice.*

   Đây chính là dấu hiệu staff.  
   *This is exactly the staff-level signal.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 WRAPPING UP FOR THE INTERVIEW (Interview cheatsheet)*

### 5.1. Keywords bắt buộc nhớ
*5.1. Keywords you must remember*

**pgvector**

pgvector là extension cho PostgreSQL.  
*pgvector is an extension for PostgreSQL.*

Nó thêm kiểu `vector`.  
*It adds the `vector` type.*

**MariaDB Vector**

MariaDB Vector là hỗ trợ vector **native**.  
*MariaDB Vector is **native** vector support.*

Nó cung cấp kiểu `VECTOR`.  
*It provides the `VECTOR` type.*

Nó đạt GA từ bản 11.8 LTS.  
*It reached GA in version 11.8 LTS.*

Nó là mã nguồn mở.  
*It is open source.*

**MySQL HeatWave**

MySQL HeatWave là dịch vụ managed của Oracle.  
*MySQL HeatWave is Oracle's managed service.*

Nó có **in-database embedding**.  
*It has **in-database embedding**.*

Tính năng đó mang tên HeatWave GenAI.  
*That feature is called HeatWave GenAI.*

**Extension / Native / Managed**

Đây là ba mô hình hỗ trợ vector của RDBMS.  
*These are the three models of RDBMS vector support.*

**BYO-embedding**

Bạn tự sinh vector bên ngoài.  
*You generate the vectors outside.*

Sau đó bạn nhét vector vào database.  
*Then you push the vectors into the database.*

pgvector và MariaDB đi theo hướng này.  
*pgvector and MariaDB follow this route.*

**In-database embedding**

Database tự sinh vector bằng LLM tích hợp.  
*The database generates vectors with an integrated LLM.*

HeatWave đi theo hướng này.  
*HeatWave follows this route.*

**Hybrid query**

Hybrid query gộp vector search với filter và join SQL.  
*A hybrid query combines vector search with SQL filters and joins.*

Tất cả nằm trong một câu.  
*It all fits in one statement.*

Tất cả nằm trong một transaction.  
*It all fits in one transaction.*

**HNSW / SPANN / ScaNN / DiskANN**

Đây là các thuật toán ANN.  
*These are ANN algorithms.*

Các nền tảng khác nhau dùng thuật toán khác nhau.  
*Different platforms use different algorithms.*

**Dedicated vector DB**

Ví dụ là Pinecone, Weaviate, Qdrant và Milvus.  
*Examples are Pinecone, Weaviate, Qdrant, and Milvus.*

Đây là hệ chuyên dụng cho vector.  
*These are systems dedicated to vectors.*

**Vendor lock-in**

Bạn bị khóa vào một nhà cung cấp.  
*You are locked into a single vendor.*

Rủi ro này cao nhất với managed proprietary.  
*This risk is highest with proprietary managed services.*

**Distance metrics**

Các metric gồm cosine, Euclidean (L2), Manhattan và Jaccard.  
*The metrics include cosine, Euclidean (L2), Manhattan, and Jaccard.*

**Cú pháp mỗi nền tảng / Syntax per platform**

MariaDB dùng `VEC_FromText` và `VEC_DISTANCE_COSINE`.  
*MariaDB uses `VEC_FromText` and `VEC_DISTANCE_COSINE`.*

MySQL dùng `STRING_TO_VECTOR` và `DISTANCE`.  
*MySQL uses `STRING_TO_VECTOR` and `DISTANCE`.*

pgvector dùng toán tử `<=>`.  
*pgvector uses the `<=>` operator.*

**tsvector ≠ vector**

`tsvector` là lexeme của FTS.  
*`tsvector` holds FTS lexemes.*

`vector` là embedding.  
*`vector` holds embeddings.*

Bạn đừng nhầm hai thứ này.  
*Do not confuse the two.*

### 5.2. Core concepts — nếu chỉ nhớ vài điều
*5.2. Core concepts — if you remember only a few things*

1. RDBMS thêm vector để bạn **khỏi dựng vector DB riêng**.  
   *1. An RDBMS adds vectors so you **skip building a separate vector DB**.*

   Có ba cách làm điều đó.  
   *There are three ways to do it.*

   Ba cách đó là extension, native và managed.  
   *Those three ways are extension, native, and managed.*

2. **pgvector là extension.**  
   *2. **pgvector is an extension.***

   **MariaDB là native.**  
   ***MariaDB is native.***

   **HeatWave là managed.**  
   ***HeatWave is managed.***

   HeatWave còn tự sinh embedding.  
   *HeatWave additionally generates embeddings.*

3. RDBMS-native có một siêu năng lực chung.  
   *3. RDBMS-native has one shared superpower.*

   Đó là **hybrid query** trong một câu SQL.  
   *That is a **hybrid query** in one SQL statement.*

   Câu đó gộp vector với business data.  
   *That statement combines vectors with business data.*

4. MariaDB Vector đã **GA và mã nguồn mở**.  
   *4. MariaDB Vector is **GA and open source**.*

   Nó lưu embedding **cùng bảng**.  
   *It stores embeddings in the **same table**.*

   Nó không lưu riêng như mô tả cũ.  
   *It does not store them separately as the old description says.*

   ANN index của MySQL chủ yếu qua HeatWave.  
   *The MySQL ANN index comes mainly through HeatWave.*

   HeatWave là proprietary.  
   *HeatWave is proprietary.*

5. Không phải nền tảng nào cũng dùng HNSW.  
   *5. Not every platform uses HNSW.*

   PlanetScale dùng SPANN.  
   *PlanetScale uses SPANN.*

   Cloud SQL dùng ScaNN.  
   *Cloud SQL uses ScaNN.*

6. **BYO-embedding linh hoạt hơn.**  
   *6. **BYO-embedding is more flexible.***

   **In-DB embedding tiện hơn.**  
   ***In-DB embedding is more convenient.***

   In-DB embedding khóa bạn chặt hơn.  
   *In-DB embedding locks you in more tightly.*

7. RDBMS-native thắng ở operational simplicity.  
   *7. RDBMS-native wins on operational simplicity.*

   Dedicated vector DB thắng ở peak scale.  
   *A dedicated vector DB wins at peak scale.*

8. Quyết định có bốn trục.  
   *8. The decision has four axes.*

   Trục (a) là feature hay product.  
   *Axis (a) is feature versus product.*

   Trục (b) là quy mô.  
   *Axis (b) is scale.*

   Trục (c) là managed hay self-host.  
   *Axis (c) is managed versus self-host.*

   Trục (d) là privacy cùng lock-in.  
   *Axis (d) is privacy and lock-in.*

9. Đổi nền tảng hoặc đổi model kéo theo re-embed.  
   *9. Switching platform or model forces a re-embed.*

   Việc đó cũng kéo theo đổi cú pháp.  
   *It also forces a syntax change.*

   Đây là chi phí ẩn rất lớn.  
   *This is a very large hidden cost.*

   Bạn ưu tiên chuẩn mở để giữ đường lui.  
   *Prefer open standards to keep an exit path.*

10. `tsvector` phục vụ FTS.  
    *10. `tsvector` serves FTS.*

    `vector` phục vụ embedding.  
    *`vector` serves embeddings.*

    Semantic search cần `vector`.  
    *Semantic search needs `vector`.*

### 5.3. Ideas / mental models
*5.3. Ideas and mental models*

**"Cơi nới phòng vs mua nhà thứ hai"**  
*"Adding a room vs buying a second house"*

Đây là hình ảnh cho RDBMS-native đối lại dedicated vector DB.  
*This is the image for RDBMS-native versus a dedicated vector DB.*

**"Feature hay product?"**  
*"A feature or the product?"*

Đây là kim chỉ nam chọn giữa RDBMS và Pinecone.  
*This is the compass for choosing between an RDBMS and Pinecone.*

**"Extension = cài thêm, native = xây sẵn, managed = thuê trọn"**  
*"Extension = install it, native = built in, managed = rent it all"*

Đây là cách nhớ ba mô hình.  
*This is the way to remember the three models.*

**"Tiện đi kèm xích"**  
*"Convenience comes with a leash"*

In-DB embedding của HeatWave đánh đổi bằng lock-in.  
*HeatWave's in-DB embedding trades convenience for lock-in.*

**"Một hệ để trông, không phải hai"**  
*"One system to watch, not two"*

Đây là lợi thế vận hành của mô hình native.  
*This is the operational advantage of the native model.*

### 5.4. Code cần thuộc lòng
*5.4. Code you must memorize*

**(a) pgvector (extension)**

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE t (id bigserial PRIMARY KEY, embedding vector(1536));
CREATE INDEX ON t USING hnsw (embedding vector_cosine_ops);
SELECT id FROM t ORDER BY embedding <=> '[...]' LIMIT 5;
```

**(b) MariaDB (native)**

```sql
CREATE TABLE t (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  embedding VECTOR(1536) NOT NULL,
  VECTOR INDEX (embedding) DISTANCE=cosine
) ENGINE=InnoDB;
SELECT id FROM t
ORDER BY VEC_DISTANCE_COSINE(embedding, VEC_FromText('[...]')) LIMIT 5;  -- LIMIT bắt buộc
                                                                        -- LIMIT is required
```

**(c) MySQL (9.x / HeatWave)**

```sql
CREATE TABLE t (id INT PRIMARY KEY, val VECTOR(5));
INSERT INTO t VALUES (1, STRING_TO_VECTOR('[1,2,3,4,5]'));
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions with suggested answers*

1. **"PostgreSQL, MySQL, MariaDB hỗ trợ vector khác nhau thế nào?"**  
   *1. **"How do PostgreSQL, MySQL, and MariaDB differ in vector support?"***

   Postgres đi theo mô hình extension.  
   *Postgres follows the extension model.*

   Extension đó là pgvector.  
   *That extension is pgvector.*

   MariaDB đi theo mô hình native.  
   *MariaDB follows the native model.*

   Nó có kiểu `VECTOR` và đã GA từ 11.8.  
   *It has the `VECTOR` type and reached GA in 11.8.*

   Nó là mã nguồn mở.  
   *It is open source.*

   MySQL Community có kiểu `VECTOR`.  
   *MySQL Community has the `VECTOR` type.*

   ANN index của nó chủ yếu qua HeatWave.  
   *Its ANN index comes mainly through HeatWave.*

   HeatWave là managed và có in-DB embedding.  
   *HeatWave is managed and offers in-DB embedding.*

2. **[BẪY] "PostgreSQL đã có vector search sẵn qua `tsvector` đúng không?"**  
   *2. **[TRAP] "PostgreSQL already has vector search through `tsvector`, right?"***

   Không đúng.  
   *No.*

   `tsvector` là full-text search với lexeme.  
   *`tsvector` is full-text search over lexemes.*

   Semantic vector search cần extension **pgvector**.  
   *Semantic vector search needs the **pgvector** extension.*

   Extension đó cung cấp kiểu `vector`.  
   *That extension provides the `vector` type.*

3. **[TRADE-OFF] "RDBMS-native hay dedicated vector DB như Pinecone?"**  
   *3. **[TRADE-OFF] "RDBMS-native or a dedicated vector DB like Pinecone?"***

   Bạn chọn native khi vector là một feature.  
   *Choose native when vectors are a feature.*

   Bạn chọn native khi số vector dưới khoảng 50 triệu.  
   *Choose native when the vector count is under roughly 50 million.*

   Bạn chọn native khi cần hybrid SQL.  
   *Choose native when you need hybrid SQL.*

   Bạn chọn native khi muốn ít hệ thống.  
   *Choose native when you want fewer systems.*

   Bạn chọn dedicated khi vector là product.  
   *Choose dedicated when vectors are the product.*

   Bạn chọn dedicated khi quy mô rất lớn hoặc đa vùng.  
   *Choose dedicated at very large scale or multi-region.*

   Bạn chọn dedicated khi cần tính năng chuyên sâu.  
   *Choose dedicated when you need specialized features.*

4. **"HeatWave khác pgvector và MariaDB ở chỗ nào?"**  
   *4. **"How does HeatWave differ from pgvector and MariaDB?"***

   HeatWave là managed.  
   *HeatWave is managed.*

   HeatWave **tự sinh embedding** trong DB.  
   *HeatWave **generates embeddings** inside the DB.*

   HeatWave là proprietary.  
   *HeatWave is proprietary.*

   HeatWave chỉ chạy trên cloud.  
   *HeatWave runs only in the cloud.*

   Mức lock-in của nó rất cao.  
   *Its lock-in level is very high.*

   pgvector và MariaDB self-host được.  
   *pgvector and MariaDB can be self-hosted.*

   Chúng là mã nguồn mở.  
   *They are open source.*

   Chúng đi theo BYO-embedding.  
   *They follow BYO-embedding.*

5. **[TRADE-OFF] "In-database embedding có gì hay và gì dở?"**  
   *5. **[TRADE-OFF] "What is good and bad about in-database embedding?"***

   Điểm hay là sự tiện lợi.  
   *The good part is convenience.*

   Bạn khỏi gọi model bên ngoài.  
   *You skip calling an external model.*

   Điểm dở là bị khóa vào model của vendor.  
   *The bad part is lock-in to the vendor's model.*

   Đổi nền tảng buộc bạn re-embed.  
   *Switching platforms forces a re-embed.*

   Cách này kém linh hoạt hơn BYO.  
   *This route is less flexible than BYO.*

6. **[BẪY] "MariaDB lưu vector ở một database riêng phải không?"**  
   *6. **[TRAP] "MariaDB stores vectors in a separate database, right?"***

   Không đúng với bản GA.  
   *Not for the GA release.*

   Embedding nằm trong cột `VECTOR` **cùng bảng** với business data.  
   *The embedding sits in a `VECTOR` column in the **same table** as the business data.*

   Nhờ đó bạn chạy hybrid query một câu.  
   *This lets you run a one-statement hybrid query.*

   Mô tả "lưu riêng" là mô tả cũ.  
   *The "separate storage" description is outdated.*

7. **[SCALE] "Công ty dùng Postgres, cần vector cho 20M docs, dữ liệu nhạy cảm?"**  
   *7. **[SCALE] "The company runs Postgres, needs vectors for 20M docs, with sensitive data?"***

   Bạn chọn self-host pgvector.  
   *Choose self-hosted pgvector.*

   Bạn ghép với một model embedding self-host.  
   *Pair it with a self-hosted embedding model.*

   Cách này tận dụng DB sẵn có.  
   *This reuses the database you already have.*

   Dữ liệu ở nguyên trong nhà.  
   *The data stays in house.*

   Bạn tránh được lock-in.  
   *You avoid lock-in.*

   Cấu hình này đủ cho 20 triệu vector.  
   *This setup is enough for 20 million vectors.*

   Bạn nêu rõ ngưỡng sẽ khiến mình đổi sang dedicated.  
   *State the threshold that would move you to a dedicated system.*

8. **"Không phải nền tảng nào cũng HNSW — đúng không?"**  
   *8. **"Not every platform uses HNSW — correct?"***

   Đúng.  
   *Correct.*

   pgvector, MariaDB và nhiều MySQL fork dùng HNSW.  
   *pgvector, MariaDB, and many MySQL forks use HNSW.*

   PlanetScale dùng SPANN.  
   *PlanetScale uses SPANN.*

   Cloud SQL dùng ScaNN.  
   *Cloud SQL uses ScaNN.*

   pgvectorscale dùng DiskANN.  
   *pgvectorscale uses DiskANN.*

### 5.6. One-liner đắt giá
*5.6. High-value one-liners*

Mọi RDBMS lớn giờ đều nói được tiếng vector.  
*Every major RDBMS now speaks vector.*

Câu hỏi không còn là nó có làm được hay không.  
*The question is no longer whether it can.*

Câu hỏi là extension, native hay managed.  
*The question is extension, native, or managed.*

Câu hỏi là mỗi lựa chọn tốn của bạn bao nhiêu lock-in.  
*The question is what each one costs you in lock-in.*

RDBMS-native thắng ở operational simplicity.  
*RDBMS-native wins on operational simplicity.*

Dedicated vector DB thắng ở peak scale.  
*Dedicated vector DBs win on peak scale.*

Bạn chọn theo việc vector search là feature hay là product.  
*Pick by whether vector search is a feature or the product.*

HeatWave sinh embedding hộ bạn.  
*HeatWave generates embeddings for you.*

Điều đó rất tiện.  
*That is convenient.*

Sự tiện lợi đó là một sợi xích.  
*That convenience is a leash.*

Bạn đổi nền tảng thì phải re-embed toàn bộ.  
*Switch platforms and you re-embed everything.*

MariaDB Vector giữ embedding trong một cột cạnh dữ liệu của bạn.  
*MariaDB Vector keeps embeddings in a column next to your data.*

Nhờ vậy "tìm sản phẩm tương tự dưới 50$ còn hàng" chỉ là một câu SQL.  
*So "find similar products under $50 in stock" is one SQL statement.*

Câu đó cũng chỉ là một transaction.  
*It is also one transaction.*

`tsvector` là keyword.  
*`tsvector` is keywords.*

`vector` là nghĩa.  
*`vector` is meaning.*

Nhầm hai thứ này là cách trượt phỏng vấn vector search nhanh nhất.  
*Confusing them is the fastest way to fail a vector-search interview.*

Đôi khi chính privacy chọn nền tảng thay bạn.  
*Sometimes privacy picks the platform for you.*

Dữ liệu có thể không được rời khỏi tòa nhà.  
*The data may not be allowed to leave the building.*

Khi đó dịch vụ cloud managed bị loại từ đầu.  
*Managed cloud services are then off the table from the start.*

Hiệu năng thậm chí chưa kịp được đem ra cân.  
*Performance never even gets weighed.*

---

### 📌 Ghi chú cuối
*📌 Final notes*

**Đính chính để nhớ đúng / Corrections to memorize**

MariaDB Vector đã GA ở bản 11.8 LTS.  
*MariaDB Vector reached GA in version 11.8 LTS.*

MariaDB Vector lưu embedding **cùng bảng**.  
*MariaDB Vector stores embeddings in the **same table**.*

Nó không lưu riêng biệt.  
*It does not store them separately.*

PlanetScale đã có vector với thuật toán SPANN.  
*PlanetScale already has vectors using the SPANN algorithm.*

`tsvector` của FTS khác `vector` của embedding.  
*The FTS `tsvector` differs from the embedding `vector`.*

ANN index của MySQL chủ yếu đến qua HeatWave.  
*The MySQL ANN index comes mainly through HeatWave.*

**Kiểm chứng khi ôn / Verify while revising**

Mảng này đổi rất nhanh.  
*This area changes very fast.*

Trước phỏng vấn bạn xem tài liệu chính thức của pgvector.  
*Before an interview, read the official pgvector documentation.*

Bạn cũng xem tài liệu MariaDB bản 11.8 trở lên.  
*Also read the MariaDB documentation for 11.8 and later.*

Bạn cũng xem tài liệu MySQL và HeatWave.  
*Also read the MySQL and HeatWave documentation.*

Mục tiêu là cập nhật cú pháp và tính năng.  
*The goal is to refresh the syntax and the features.*

**Thực hành / Practice**

Bạn dựng thử pgvector bằng Docker image `pgvector/pgvector`.  
*Spin up pgvector with the `pgvector/pgvector` Docker image.*

Bạn dựng thử MariaDB 11.8 bằng Docker.  
*Spin up MariaDB 11.8 with Docker.*

Bạn tạo cùng một bảng vector trên mỗi bên.  
*Create the same vector table on each side.*

Bạn chạy cùng một query trên cả hai.  
*Run the same query on both.*

Bạn so cú pháp `<=>` với `VEC_DISTANCE_COSINE`.  
*Compare the `<=>` syntax against `VEC_DISTANCE_COSINE`.*

Bạn sẽ cảm nhận rõ một điều.  
*You will clearly feel one thing.*

Ý tưởng giống nhau.  
*The idea is the same.*

Cú pháp thì khác nhau.  
*The syntax is different.*

**Nối mạch series (đủ 5 mảnh) / Connecting the series (all five pieces)**

Mảnh 1 là embed để tạo vector.  
*Piece 1 is embed, which creates vectors.*

Mảnh 2 là index để làm nhanh.  
*Piece 2 is index, which makes things fast.*

Mảnh 3 là store/query với pgvector.  
*Piece 3 is store/query with pgvector.*

Mảnh 4 là keyword/FTS.  
*Piece 4 is keyword/FTS.*

Mảnh 5 là **chọn nền tảng**, tức bài này.  
*Piece 5 is **choosing the platform**, which is this lesson.*

Toàn bộ năm mảnh phục vụ một mục tiêu.  
*All five pieces serve one goal.*

Bạn build và vận hành semantic search hoặc RAG có hiểu biết.  
*You build and operate semantic search or RAG with real understanding.*

**Học tiếp / What to learn next**

Bạn học benchmark có hệ thống giữa các nền tảng.  
*Learn systematic benchmarking across platforms.*

Hai bộ tham khảo là ann-benchmarks và Big Vector Search Benchmark.  
*Two references are ann-benchmarks and the Big Vector Search Benchmark.*

Bạn học chiến lược migration giữa các vector store.  
*Learn migration strategies between vector stores.*

Bạn học hybrid dense cộng sparse.  
*Learn hybrid dense plus sparse.*

Hai ví dụ là BGE-M3 và Cohere embed-v4.  
*Two examples are BGE-M3 and Cohere embed-v4.*
