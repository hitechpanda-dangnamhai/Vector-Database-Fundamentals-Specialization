# PostgreSQL: Cơ sở dữ liệu quan hệ & Full-Text Search (tsvector / tsquery)
*PostgreSQL: Relational Databases and Full-Text Search (tsvector / tsquery)*
### Giáo trình SIÊU DỄ HIỂU — giải thích tận gốc mọi thuật ngữ | Basic → Staff
*An ULTRA-CLEAR course — every term explained from the ground up | Basic → Staff*

> **Bài giảng gốc — nguồn của giáo trình**  
> *Source lecture — where this course comes from*
>
> Bài giảng gốc là *"Refresh your knowledge of relational databases with a focus on PostgreSQL"*.  
> *The source lecture is "Refresh your knowledge of relational databases with a focus on PostgreSQL".*
>
> Bài giảng này thuộc Course 3, IBM Vector Database Fundamentals.  
> *This lecture belongs to Course 3, IBM Vector Database Fundamentals.*

> **Cách đọc giáo trình này**  
> *How to read this course*
>
> Bạn hãy đọc tuần tự từ trên xuống.  
> *Please read straight through from top to bottom.*
>
> Mọi từ chuyên ngành đều được giải thích **ngay lần đầu nó xuất hiện**.  
> *Every technical term gets explained the first time it appears.*
>
> Bạn không cần biết gì trước.  
> *You do not need any prior knowledge.*
>
> Bạn gặp một từ lạ chưa thấy giải thích.  
> *You may meet an unfamiliar term with no explanation.*
>
> Đó là lỗi của người viết.  
> *That is the writer's fault.*
>
> Đó không phải lỗi của bạn.  
> *That is not your fault.*

> **Ký hiệu trong bài**  
> *Symbols used in this course*
>
> 🧩 **[Ngoài bài gốc]** đánh dấu phần bài giảng gốc không nói.  
> *🧩 [Ngoài bài gốc] marks material the source lecture does not cover.*
>
> Một staff engineer bắt buộc phải biết phần đó.  
> *A staff engineer must know that material.*
>
> 💡 **Mẹo thực chiến** là kinh nghiệm khi làm thật ở production.  
> *💡 Mẹo thực chiến is practical experience from real production work.*
>
> Production — môi trường chạy thật — là hệ thống đang phục vụ người dùng thật.  
> *Production is the system that serves real users.*
>
> Production hỏng thì có người bị ảnh hưởng.  
> *A production failure affects real people.*
>
> ⚠️ **Chỗ khó** là chỗ nhiều người vấp.  
> *⚠️ Chỗ khó marks the places where many people stumble.*
>
> Vấp ở đây là chuyện bình thường.  
> *Stumbling here is normal.*

> **Cập nhật thông tin tới tháng 7/2026**  
> *Information updated through July 2026*
>
> Bản PostgreSQL ổn định mới nhất là **18.4**.  
> *The latest stable PostgreSQL release is 18.4.*
>
> Các nhánh 17, 16, 15 và 14 vẫn được hỗ trợ.  
> *The 17, 16, 15, and 14 branches are still supported.*
>
> **PostgreSQL 19 Beta 2** ra ngày 16/07/2026.  
> *PostgreSQL 19 Beta 2 came out on 16/07/2026.*
>
> Bản chính thức dự kiến ra vào tháng 9/2026.  
> *The final release is expected in September 2026.*
>
> Với full-text search — tìm kiếm toàn văn — **GIN vẫn là loại index chuẩn**.  
> *For full-text search, GIN is still the standard index type.*
>
> Cách làm được khuyến nghị hiện nay là dùng **generated column (STORED)**.  
> *The currently recommended approach uses a generated column with STORED.*
>
> Một lựa chọn khác là **functional index**.  
> *Another option is a functional index.*
>
> Bạn không nên tự viết trigger đồng bộ bằng tay.  
> *You should not hand-write a synchronization trigger yourself.*
>
> Các từ in đậm này sẽ được giải thích đầy đủ ở dưới.  
> *These bold terms get full explanations further down.*
>
> Bạn chưa cần hiểu chúng ngay lúc này.  
> *You do not need to understand them right now.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson map (Overview)*

### 0.1. Một câu tóm tắt siêu đơn giản
*0.1. One ultra-simple summary sentence*

**Bài này dạy bạn cách bắt cơ sở dữ liệu tìm chữ trong văn bản một cách thông minh.**  
*This lesson teaches you how to make a database search text intelligently.*

Người dùng gõ chữ "running".  
*A user types the word "running".*

Hệ thống vẫn tìm ra được những bài viết chỉ chứa chữ "ran".  
*The system still finds articles that contain only "ran".*

Hệ thống cũng tìm ra những bài viết chỉ chứa chữ "runs".  
*The system also finds articles that contain only "runs".*

Hệ thống tìm ra **rất nhanh**.  
*The system finds them very fast.*

Bạn có hàng chục triệu bài viết.  
*You have tens of millions of articles.*

Tốc độ vẫn nhanh như vậy.  
*The speed stays that fast.*

Bạn chưa hiểu vài từ ở trên.  
*You may not understand a few words above.*

Bạn chưa biết cơ sở dữ liệu là gì.  
*You may not know what a database is.*

Bạn chưa biết tại sao tìm chữ lại chậm.  
*You may not know why searching text is slow.*

Điều đó hoàn toàn bình thường.  
*That is completely normal.*

Điều đó cũng đúng như dự kiến.  
*That is also exactly as intended.*

Phần 1 sẽ dựng lại từng viên gạch một.  
*Part 1 will lay down each brick one by one.*

Phần 1 bắt đầu từ con số không.  
*Part 1 starts from zero.*

### 0.2. Bài này gồm hai tầng nội dung
*0.2. This lesson has two layers of content*

**Tầng thứ nhất là phần ôn lại nền móng.**  
*The first layer is a review of the foundations.*

Bạn học cách cơ sở dữ liệu quan hệ tổ chức dữ liệu.  
*You learn how a relational database organizes data.*

Bạn học bảng là gì.  
*You learn what a table is.*

Bạn học khóa chính và khóa ngoại là gì.  
*You learn what a primary key and a foreign key are.*

Bạn học PostgreSQL có những kiểu dữ liệu nào.  
*You learn which data types PostgreSQL offers.*

Bạn học lý do PostgreSQL đặc biệt so với các cơ sở dữ liệu khác.  
*You learn why PostgreSQL is special among databases.*

**Tầng thứ hai là trọng tâm thật sự của bài.**  
*The second layer is the real focus of this lesson.*

Đó là kỹ thuật **full-text search** — tìm kiếm toàn văn.  
*That technique is full-text search.*

PostgreSQL biến một đoạn văn bản thô thành một cấu trúc gọi là `tsvector`.  
*PostgreSQL turns raw text into a structure called `tsvector`.*

PostgreSQL so khớp cấu trúc đó với một truy vấn gọi là `tsquery`.  
*PostgreSQL matches that structure against a query called `tsquery`.*

Nhờ vậy bạn tìm được tài liệu theo **nghĩa của từ**.  
*This lets you find documents by the meaning of a word.*

Bạn không so từng ký tự một cách máy móc.  
*You do not compare character by character mechanically.*

### 0.3. Vấn đề thực tế mà bài này giải quyết
*0.3. The real-world problem this lesson solves*

Bạn có một ô tìm kiếm trên website.  
*You have a search box on a website.*

Người dùng gõ vào chữ "running".  
*A user types the word "running".*

Cách làm ngây thơ nhất là bảo cơ sở dữ liệu tìm những dòng chứa chuỗi ký tự *running*.  
*The most naive approach asks the database for rows containing the string "running".*

Cách này hỏng theo hai kiểu cùng lúc.  
*This approach fails in two ways at once.*

Nó **chậm khủng khiếp** khi dữ liệu lớn.  
*It is terribly slow on large data.*

Nó cũng **ngu ngốc**.  
*It is also dumb.*

Nó không biết "ran", "runs" và "running" là cùng một từ.  
*It does not know that "ran", "runs", and "running" are the same word.*

Nó lại tưởng "b**run**ch" có liên quan tới "run".  
*It wrongly thinks that "brunch" relates to "run".*

Full-text search sinh ra để chữa cả hai bệnh đó cùng lúc.  
*Full-text search exists to cure both problems at once.*

Toàn bộ Phần 1 sẽ mổ xẻ lý do cách ngây thơ kia hỏng.  
*All of Part 1 dissects why the naive approach fails.*

Sau đó Phần 1 mới giới thiệu liều thuốc.  
*Only then does Part 1 introduce the cure.*

### 0.4. Học xong bạn sẽ làm được gì
*0.4. What you will be able to do afterwards*

Bạn giải thích rành mạch cơ sở dữ liệu, DBMS và bảng.  
*You can clearly explain databases, DBMSs, and tables.*

Bạn giải thích rành mạch khóa chính, khóa ngoại và BLOB.  
*You can clearly explain primary keys, foreign keys, and BLOBs.*

Bạn giải thích được lý do PostgreSQL được gọi là "mở rộng được".  
*You can explain why PostgreSQL is called "extensible".*

Bạn chọn đúng kiểu dữ liệu của PostgreSQL cho từng nhu cầu.  
*You can pick the right PostgreSQL data type for each need.*

Bạn tránh được cái bẫy lưu tiền bằng số thực.  
*You can avoid the trap of storing money as a float.*

Bạn hiểu bản chất của lexeme, `tsvector`, `tsquery` và toán tử `@@`.  
*You understand what lexemes, `tsvector`, `tsquery`, and the `@@` operator really are.*

Bạn tự viết được câu lệnh tìm kiếm chạy được thật.  
*You can write a search statement that actually runs.*

Bạn đánh index đúng cách để tìm kiếm nhanh.  
*You can build the right index to make search fast.*

Bạn xếp hạng kết quả theo độ liên quan.  
*You can rank results by relevance.*

Bạn phân tích được khi nào PostgreSQL là đủ.  
*You can judge when PostgreSQL is enough.*

Bạn phân tích được khi nào cần tới hệ thống tìm kiếm chuyên dụng.  
*You can judge when a dedicated search system is needed.*

Bạn trả lời được câu hỏi thiết kế hệ thống về search trong phỏng vấn.  
*You can answer a system design interview question about search.*

### 0.5. Mạch kiến thức từ dễ tới khó
*0.5. The knowledge path from easy to hard*

🟢 **Basic** bắt đầu từ nền móng cơ sở dữ liệu.  
*🟢 Basic starts from the database foundations.*

Tiếp theo là lý do cách tìm bằng `LIKE` bị gãy.  
*Next comes the reason the `LIKE` approach breaks.*

Sau đó là khái niệm lexeme.  
*Then comes the concept of a lexeme.*

Cuối cùng là ví dụ "hello world" đầu tiên.  
*Finally comes the first "hello world" example.*

🟡 **Intermediate** mở ra bên trong PostgreSQL.  
*🟡 Intermediate opens up the inside of PostgreSQL.*

Bạn thấy văn bản đi qua những chặng nào.  
*You see which stages the text passes through.*

Bạn học cách chọn đúng hàm tạo truy vấn.  
*You learn to pick the right query-building function.*

Bạn học cách đánh index GIN.  
*You learn how to build a GIN index.*

Bạn học cách xếp hạng kết quả.  
*You learn how to rank results.*

Bạn học ba lỗi kinh điển.  
*You learn three classic mistakes.*

🔴 **Advanced** mổ ruột gan của inverted index.  
*🔴 Advanced dissects the inner workings of the inverted index.*

Bạn so sánh GIN với GiST.  
*You compare GIN against GiST.*

Bạn xem xét độ phức tạp thuật toán.  
*You examine algorithmic complexity.*

Bạn học cách chống lỗi chính tả.  
*You learn how to handle typos.*

Bạn học các trường hợp biên.  
*You learn the edge cases.*

🟣 **Staff** chạy thử với hàng chục triệu tài liệu.  
*🟣 Staff runs with tens of millions of documents.*

Bạn tìm xem hệ thống gãy ở đâu.  
*You find out where the system breaks.*

Bạn chọn giữa PostgreSQL và Elasticsearch.  
*You choose between PostgreSQL and Elasticsearch.*

Bạn học hybrid search — tìm kiếm lai.  
*You learn hybrid search.*

Hybrid search kết hợp tìm theo từ khóa với tìm theo ngữ nghĩa.  
*Hybrid search combines keyword search with semantic search.*

Bạn xem xét chi phí, vận hành và ảnh hưởng tổ chức.  
*You examine cost, operations, and organizational impact.*

Bạn làm một câu hỏi system design mẫu.  
*You work through a sample system design question.*

🎯 **Cheatsheet** gồm bảng thuật ngữ và ý cốt lõi.  
*🎯 The cheatsheet holds a term table and the core ideas.*

Cheatsheet cũng gồm code cần thuộc.  
*The cheatsheet also holds code worth memorizing.*

Cheatsheet có câu hỏi phỏng vấn kèm gợi ý trả lời.  
*The cheatsheet has interview questions with suggested answers.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

> Phần này viết cho người **chưa biết gì**.  
> *This part is written for people who know nothing yet.*
>
> Nó là phần dài nhất trong giáo trình.  
> *It is the longest part of the course.*
>
> Nó cũng là phần chậm nhất.  
> *It is also the slowest part.*
>
> Điều đó là cố ý.  
> *That is deliberate.*
>
> Bạn đã biết cơ sở dữ liệu quan hệ.  
> *You may already know relational databases.*
>
> Bạn vẫn nên lướt qua mục 1.4 trở đi.  
> *You should still skim from section 1.4 onward.*

### 1.1. Bắt đầu từ vấn đề: vì sao con người cần cơ sở dữ liệu
*1.1. Starting from the problem: why people need databases*

Hãy tưởng tượng bạn mở một cửa hàng bán quần áo online.  
*Imagine you open an online clothing shop.*

Ngày đầu tiên có 3 khách đặt hàng.  
*On the first day, 3 customers place orders.*

Bạn ghi vào một cuốn sổ tay.  
*You write them in a notebook.*

Bạn ghi tên khách, số điện thoại, món hàng và ngày giao.  
*You write the name, phone number, item, and delivery date.*

Không có vấn đề gì.  
*There is no problem.*

Sáu tháng sau bạn có 50.000 khách.  
*Six months later you have 50,000 customers.*

Giờ hãy thử trả lời một câu hỏi.  
*Now try to answer one question.*

*"Khách hàng nào đã mua áo khoác trong tháng trước và chưa được giao hàng?"*  
*"Which customers bought a jacket last month and have not received delivery?"*

Với cuốn sổ tay, bạn phải lật từng trang.  
*With the notebook, you must flip through every page.*

Bạn phải đọc từng dòng.  
*You must read every line.*

Việc đó mất vài ngày.  
*That takes several days.*

Bạn chắc chắn sẽ bỏ sót.  
*You will certainly miss some entries.*

Đó chính là **vấn đề** mà cơ sở dữ liệu sinh ra để giải.  
*That is exactly the problem databases exist to solve.*

Cơ sở dữ liệu lưu trữ lượng lớn dữ liệu.  
*A database stores large amounts of data.*

Việc **tìm, lọc và cập nhật** vẫn nhanh khi dữ liệu phình to.  
*Finding, filtering, and updating stay fast as the data grows.*

### 1.2. Các thuật ngữ nền móng — giải thích từng từ một
*1.2. The foundational terms — explained one word at a time*

**Database** — cơ sở dữ liệu — đọc là "đây-ta-bây".  
*Database is pronounced "đây-ta-bây" in Vietnamese.*

Database là một tập hợp dữ liệu được tổ chức có quy củ.  
*A database is a systematically organized collection of data.*

Dữ liệu đó được lưu trữ dưới dạng số hóa để dùng đi dùng lại nhiều lần.  
*The data is stored digitally for repeated reuse.*

Từ "có tổ chức" là mấu chốt.  
*The phrase "organized" is the crux.*

Một đống file lộn xộn trong máy tính không phải là database.  
*A messy pile of files on a computer is not a database.*

Một tập dữ liệu được sắp xếp theo cấu trúc rõ ràng mới là database.  
*Only data arranged in a clear structure counts as a database.*

**DBMS** là viết tắt của *Database Management System* — hệ quản trị cơ sở dữ liệu.  
*DBMS stands for Database Management System.*

DBMS là **phần mềm** đứng ra quản lý cái database đó.  
*A DBMS is the software that manages that database.*

Nó cho phép bạn truy cập, sửa, thêm và xóa dữ liệu.  
*It lets you access, edit, add, and delete data.*

Nó làm việc đó một cách nhanh chóng và an toàn.  
*It does this quickly and safely.*

Ví dụ là PostgreSQL, MySQL và Oracle Database.  
*Examples are PostgreSQL, MySQL, and Oracle Database.*

Hãy phân biệt cho rõ hai từ trên.  
*Please distinguish these two terms carefully.*

Rất nhiều người dùng lẫn lộn chúng.  
*Many people confuse them.*

**Database là dữ liệu.**  
*A database is the data.*

**DBMS là phần mềm trông coi dữ liệu đó.**  
*A DBMS is the software that looks after that data.*

Chuyện này giống như "sách" và "thủ thư".  
*This is like a "book" and a "librarian".*

Hai thứ đó khác nhau.  
*Those two things are different.*

Có một chi tiết đáng chú ý ở bên dưới.  
*There is a notable detail underneath.*

DBMS thật ra lưu dữ liệu thành các **file** trên ổ đĩa.  
*A DBMS actually stores data as files on disk.*

Các file đó giống file trong máy tính của bạn.  
*Those files are like the files on your own computer.*

DBMS lại **trình bày** cho bạn thấy dữ liệu dưới dạng **bảng**.  
*The DBMS presents that data to you as tables.*

Cách trình bày này giúp bạn dễ hình dung.  
*This presentation makes it easier to picture.*

Cách trình bày này cũng giúp bạn dễ truy vấn.  
*This presentation also makes it easier to query.*

Bạn làm việc với bảng.  
*You work with tables.*

Chuyện file nằm ở đâu trên đĩa là việc của DBMS.  
*Where the files sit on disk is the DBMS's job.*

**RDBMS** là viết tắt của *Relational Database Management System* — hệ quản trị cơ sở dữ liệu quan hệ.  
*RDBMS stands for Relational Database Management System.*

RDBMS là loại DBMS tổ chức dữ liệu thành các bảng **có quan hệ với nhau**.  
*An RDBMS is a DBMS that organizes data into related tables.*

Chữ "relational" — quan hệ — chính là điểm mấu chốt.  
*The word "relational" is the crux.*

Mục 1.3 sẽ cho bạn thấy "quan hệ" nghĩa là gì bằng ví dụ cụ thể.  
*Section 1.3 will show you what "relational" means through a concrete example.*

**Table** — bảng — là dữ liệu được xếp thành hàng và cột.  
*A table is data arranged in rows and columns.*

Nó y hệt một trang Excel.  
*It looks exactly like an Excel sheet.*

Mỗi **column** — cột — là một loại thông tin.  
*Each column is one kind of information.*

Ví dụ là cột "tên khách".  
*One example is the "customer name" column.*

Mỗi **row** — hàng — là một cá thể cụ thể.  
*Each row is one specific individual.*

Row còn được gọi là **record** — bản ghi.  
*A row is also called a record.*

Ví dụ là một khách hàng tên Lan.  
*One example is a customer named Lan.*

**Entity** — thực thể — là cách gọi mang tính khái niệm.  
*An entity is a conceptual way of naming something.*

Nó chỉ "một thứ có thật mà một hàng đang mô tả".  
*It refers to the real thing that a row describes.*

Một hàng trong bảng `customers` đại diện cho một con người có thật.  
*A row in the `customers` table represents a real person.*

Từ này hay xuất hiện trong tài liệu thiết kế.  
*This word appears often in design documents.*

Bạn nên quen mặt nó.  
*You should get familiar with it.*

**Schema** — lược đồ — đọc là "ski-ma".  
*Schema is pronounced "ski-ma".*

Schema là bản thiết kế của database.  
*A schema is the design blueprint of a database.*

Schema cho biết database có những bảng nào.  
*A schema tells you which tables the database has.*

Schema cho biết mỗi bảng có cột gì.  
*A schema tells you which columns each table has.*

Schema cho biết kiểu dữ liệu ra sao.  
*A schema tells you what the data types are.*

Bảng giống như căn nhà.  
*A table is like a house.*

Schema giống như bản vẽ kiến trúc.  
*A schema is like the architectural drawing.*

**Query** — truy vấn — là một câu lệnh bạn gửi cho DBMS.  
*A query is a statement you send to the DBMS.*

Bạn dùng nó để hỏi dữ liệu.  
*You use it to ask for data.*

Bạn cũng dùng nó để thay đổi dữ liệu.  
*You also use it to change data.*

**SQL** là viết tắt của *Structured Query Language*.  
*SQL stands for Structured Query Language.*

Người ta đọc nó là "ét-quờ-eo" hoặc "sí-cồ".  
*People pronounce it "ét-quờ-eo" or "sí-cồ".*

SQL là ngôn ngữ chuẩn để viết những câu truy vấn đó.  
*SQL is the standard language for writing those queries.*

Ví dụ là `SELECT name FROM customers;`.  
*One example is `SELECT name FROM customers;`.*

Câu đó nghĩa là "lấy cột name từ bảng customers".  
*That statement means "take the name column from the customers table".*

### 1.3. Khóa chính và khóa ngoại — trái tim của chữ "quan hệ"
*1.3. Primary keys and foreign keys — the heart of the word "relational"*

Đây là chỗ nhiều người mới bị rối.  
*This is where many beginners get confused.*

Vì vậy ta đi thật chậm bằng một ví dụ cụ thể.  
*So we go very slowly through a concrete example.*

Đó chính là ví dụ mà bài giảng gốc dùng.  
*It is exactly the example the source lecture uses.*

Ví dụ là một nhà bán lẻ — retailer.  
*The example is a retailer.*

Nhà bán lẻ này có bốn bảng.  
*This retailer has four tables.*

Bốn bảng là `customers`, `orders`, `orderline_items` và `items`.  
*The four tables are `customers`, `orders`, `orderline_items`, and `items`.*

Chúng lần lượt là khách hàng, đơn hàng, các dòng trong đơn hàng và mặt hàng.  
*They are customers, orders, order line items, and items respectively.*

**Primary key** — khóa chính — viết tắt là **PK**.  
*A primary key is abbreviated PK.*

PK là cột dùng để **định danh duy nhất** mỗi hàng trong bảng.  
*A PK is the column that uniquely identifies each row in a table.*

PK cũng có thể là một nhóm cột.  
*A PK can also be a group of columns.*

"Định danh duy nhất" nghĩa là không có hai hàng nào trùng giá trị ở cột đó.  
*"Uniquely identifies" means no two rows share a value in that column.*

Bạn nhìn vào giá trị đó.  
*You look at that value.*

Bạn biết chính xác đang nói tới hàng nào.  
*You know exactly which row you are talking about.*

Ví dụ trong bảng `customers`, cột `cust_id` là khóa chính.  
*In the `customers` table, the `cust_id` column is the primary key.*

Khách Lan có `cust_id = 101`.  
*Customer Lan has `cust_id = 101`.*

Khách Minh có `cust_id = 102`.  
*Customer Minh has `cust_id = 102`.*

Không bao giờ có hai khách cùng mang số 101.  
*Two customers never both carry the number 101.*

Vì sao ta không dùng luôn tên khách làm khóa chính?  
*Why not simply use the customer name as the primary key?*

Lý do thứ nhất là tên bị trùng.  
*The first reason is that names repeat.*

Có hàng nghìn người tên "Nguyễn Văn An".  
*There are thousands of people named "Nguyễn Văn An".*

Lý do thứ hai là tên còn có thể đổi.  
*The second reason is that names can change.*

Khóa chính cần một giá trị **không trùng và không đổi**.  
*A primary key needs a value that is unique and stable.*

Vì vậy người ta thường tạo hẳn một cột số riêng cho việc đó.  
*So people usually create a dedicated numeric column for it.*

**Foreign key** — khóa ngoại — viết tắt là **FK**.  
*A foreign key is abbreviated FK.*

FK là một cột trong bảng này chứa giá trị **trỏ tới khóa chính của một bảng khác**.  
*An FK is a column in one table holding a value that points to another table's primary key.*

Chính cái mũi tên trỏ qua trỏ lại này tạo ra **relationship** — mối quan hệ — giữa các bảng.  
*This back-and-forth arrow creates the relationship between tables.*

Đó là lý do người ta gọi loại cơ sở dữ liệu này là "quan hệ".  
*That is why people call this kind of database "relational".*

Cụ thể là bảng `orders` cũng có một cột tên `cust_id`.  
*Concretely, the `orders` table also has a column named `cust_id`.*

Ở đây nó **không phải** khóa chính.  
*Here it is not a primary key.*

Nó là khóa ngoại trỏ ngược về bảng `customers`.  
*It is a foreign key pointing back to the `customers` table.*

Một đơn hàng có `cust_id = 101`.  
*An order has `cust_id = 101`.*

Ta biết ngay đơn đó là của chị Lan.  
*We immediately know that order belongs to Lan.*

Cứ hình dung thế này để cho dễ nhớ.  
*Picture it this way to remember it easily.*

**Khóa chính giống số căn cước công dân.**  
*A primary key is like a national ID number.*

Mỗi người một số.  
*Each person has one number.*

Không ai trùng ai.  
*No two people share a number.*

**Khóa ngoại giống chỗ ghi "số căn cước của người nhận" trên một tờ biên nhận.**  
*A foreign key is like the "recipient's ID number" field on a receipt.*

Bản thân tờ biên nhận không phải là con người.  
*The receipt itself is not a person.*

Nó *tham chiếu* tới một con người cụ thể qua con số ấy.  
*It references one specific person through that number.*

Phép ví von này khớp ở đúng chỗ quan trọng nhất.  
*This analogy fits at the most important point.*

Bạn không cần chép lại toàn bộ thông tin cá nhân lên từng tờ biên nhận.  
*You do not need to copy all the personal details onto every receipt.*

Thông tin đó gồm tên, địa chỉ và số điện thoại.  
*Those details include the name, address, and phone number.*

Bạn chỉ ghi một con số.  
*You write down just one number.*

Bạn tra ngược lại khi cần.  
*You look it up again when you need it.*

Cơ sở dữ liệu quan hệ hoạt động y hệt như vậy.  
*A relational database works exactly the same way.*

Nó **không lặp lại dữ liệu**.  
*It does not repeat data.*

Nó **chỉ tham chiếu tới dữ liệu**.  
*It only references data.*

Việc không lặp lại có một lợi ích cụ thể.  
*Not repeating data brings one concrete benefit.*

Chị Lan đổi số điện thoại.  
*Lan changes her phone number.*

Bạn chỉ sửa **một chỗ duy nhất** trong bảng `customers`.  
*You edit exactly one place in the `customers` table.*

Trước đó bạn đã chép số điện thoại vào 500 đơn hàng.  
*Suppose you had copied that phone number into 500 orders.*

Khi đó bạn sẽ phải sửa 500 chỗ.  
*You would then have to edit 500 places.*

Bạn chắc chắn sẽ sót vài chỗ.  
*You would certainly miss a few.*

**JOIN** — dịch thô là "nối" — là phép toán trong SQL.  
*JOIN is an operation in SQL.*

JOIN dùng để ghép hai bảng lại với nhau.  
*JOIN combines two tables together.*

Nó ghép theo đúng cặp khóa chính và khóa ngoại đó.  
*It combines them through that primary key and foreign key pair.*

Nhờ vậy bạn trả lời được câu hỏi kiểu "khách nào đã đặt đơn nào".  
*This lets you answer questions like "which customer placed which order".*

Bạn chưa cần viết được JOIN ngay.  
*You do not need to write a JOIN yet.*

Bạn chỉ cần biết đó là công cụ khai thác quan hệ giữa các bảng.  
*You just need to know it is the tool for exploiting relationships between tables.*

### 1.4. BLOB — khi dữ liệu không phải là chữ và số
*1.4. BLOB — when the data is neither text nor numbers*

**BLOB** là viết tắt của *Binary Large Object* — đối tượng nhị phân lớn.  
*BLOB stands for Binary Large Object.*

BLOB là cách lưu những dữ liệu phức tạp ngay bên trong cơ sở dữ liệu.  
*A BLOB is a way to store complex data inside the database itself.*

Dữ liệu đó gồm ảnh, video, file PDF và file âm thanh.  
*That data includes images, video, PDF files, and audio files.*

BLOB lưu chúng dưới dạng **nhị phân**.  
*A BLOB stores them in binary form.*

**Nhị phân — binary —** ở đây nghĩa là dữ liệu được lưu nguyên xi.  
*Binary here means the data is stored exactly as it is.*

Dữ liệu nằm dưới dạng một chuỗi các bit 0 và 1.  
*The data sits as a string of 0 and 1 bits.*

Dữ liệu đó không được diễn giải thành chữ hay số.  
*That data is not interpreted as text or numbers.*

Cơ sở dữ liệu không hiểu nội dung bức ảnh là gì.  
*The database does not understand what the image shows.*

Nó chỉ giữ hộ bạn nguyên khối byte đó.  
*It just holds that block of bytes for you.*

Nó trả lại y nguyên khi bạn hỏi.  
*It returns it unchanged when you ask.*

🧩 **[Ngoài bài gốc]** Bài gốc chỉ giới thiệu rằng BLOB tồn tại.  
*🧩 [Beyond the source lesson] The source lesson only mentions that BLOBs exist.*

Ở thực chiến có một câu hỏi thú vị hơn.  
*In real work there is a more interesting question.*

Bạn *có nên* nhét ảnh vào database hay không?  
*Should you actually put images into the database?*

Câu trả lời của phần lớn hệ thống hiện đại là **không**.  
*Most modern systems answer no.*

Người ta lưu file lên một dịch vụ lưu trữ riêng.  
*People upload the file to a separate storage service.*

Một ví dụ là Amazon S3 — một kho chứa file trên đám mây.  
*One example is Amazon S3, a cloud file store.*

Trong database người ta chỉ lưu **đường dẫn** tới file đó.  
*In the database they store only the path to that file.*

Lý do thứ nhất là BLOB làm database phình to rất nhanh.  
*The first reason is that BLOBs bloat the database very fast.*

Database phình to khiến việc sao lưu chậm và tốn kém.  
*A bloated database makes backups slow and expensive.*

Lý do thứ hai là dịch vụ lưu trữ file chuyên dụng rẻ hơn nhiều.  
*The second reason is that a dedicated file storage service is far cheaper.*

Đây là một câu hỏi phỏng vấn hay gặp.  
*This is a common interview question.*

Bạn biết nói ra trade-off — sự đánh đổi — này.  
*You can articulate this trade-off.*

Đó là một điểm cộng.  
*That is a plus point.*

### 1.5. Các RDBMS phổ biến, và PostgreSQL đặc biệt ở chỗ nào
*1.5. Common RDBMSs, and what makes PostgreSQL special*

Bạn sẽ gặp những cái tên sau trong nghề.  
*You will meet the following names in this profession.*

Đó là **IBM Db2, Oracle Database, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, MariaDB, SAP HANA và Amazon RDS**.  
*They are IBM Db2, Oracle Database, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, MariaDB, SAP HANA, and Amazon RDS.*

Trong đó có loại thương mại.  
*Some of them are commercial.*

Loại thương mại bắt bạn trả tiền bản quyền.  
*Commercial ones charge you a license fee.*

Có loại **open-source** — mã nguồn mở.  
*Some of them are open-source.*

Loại này công khai mã nguồn.  
*This kind publishes its source code.*

Loại này cho bạn dùng miễn phí.  
*This kind is free to use.*

Có loại chạy trên máy chủ của chính công ty bạn.  
*Some run on your own company's servers.*

Có loại chạy trên đám mây.  
*Some run in the cloud.*

PostgreSQL thường được gọi tắt là **Postgres**.  
*PostgreSQL is often shortened to Postgres.*

Nó được xếp vào nhóm **ORDBMS**.  
*It belongs to the ORDBMS group.*

ORDBMS là viết tắt của *Object-Relational Database Management System*.  
*ORDBMS stands for Object-Relational Database Management System.*

Tiếng Việt dịch là hệ quản trị cơ sở dữ liệu quan hệ - đối tượng.  
*In Vietnamese it is called hệ quản trị cơ sở dữ liệu quan hệ - đối tượng.*

Nghĩa là nó vẫn là cơ sở dữ liệu quan hệ bình thường.  
*This means it is still an ordinary relational database.*

Nó được bổ sung thêm các tính năng theo hướng "đối tượng".  
*It gains extra features in the "object" direction.*

Trong tất cả những tính năng đó, thứ quan trọng nhất với chúng ta là tính mở rộng.  
*Among all those features, the most important one for us is extensibility.*

**PostgreSQL mở rộng được.**  
*PostgreSQL is extensible.*

Tiếng Anh gọi tính chất đó là **extensibility** — tính mở rộng.  
*In English that property is called extensibility.*

Cụ thể là bạn có thể cắm thêm vào Postgres những thứ nó vốn không có sẵn.  
*Concretely, you can plug into Postgres things it does not ship with.*

Bạn cắm thêm kiểu dữ liệu mới.  
*You plug in new data types.*

Bạn cắm thêm hàm mới.  
*You plug in new functions.*

Bạn cắm thêm cả **phương pháp đánh index mới**.  
*You even plug in new indexing methods.*

**Extension** — phần mở rộng — là gói tính năng do bên thứ ba viết.  
*An extension is a feature package written by a third party.*

Bạn cài nó thêm vào Postgres bằng một câu lệnh.  
*You install it into Postgres with a single statement.*

Cứ hình dung Postgres như một chiếc điện thoại.  
*Picture Postgres as a mobile phone.*

Phần cứng là cố định.  
*The hardware is fixed.*

Bạn cài thêm ứng dụng.  
*You install extra apps.*

Nhờ vậy nó làm được những việc mà nhà sản xuất chưa nghĩ tới.  
*It then does things the manufacturer never thought of.*

Có hai ví dụ trực tiếp liên quan tới khóa học này.  
*Two examples relate directly to this course.*

- **`pgvector`** là một extension thêm kiểu dữ liệu `vector` vào Postgres.  
  *`pgvector` is an extension that adds the `vector` data type to Postgres.*

  Nó dùng để làm **vector search** — tìm kiếm theo ngữ nghĩa.  
  *It is used for vector search.*

  Đó là nội dung của giáo trình trước.  
  *That is the content of the previous course.*

- **Full-text search** thêm hai kiểu dữ liệu `tsvector` và `tsquery`.  
  *Full-text search adds two data types, `tsvector` and `tsquery`.*

  Nó cũng thêm các phương pháp index tên là GIN và GiST.  
  *It also adds the indexing methods named GIN and GiST.*

  Nó dùng để tìm kiếm theo từ khóa.  
  *It is used for keyword search.*

  Đó là nội dung của giáo trình này.  
  *That is the content of this course.*

🧩 **[Ngoài bài gốc]** Bạn đừng coi "extensibility" là một chi tiết kỹ thuật vặt vãnh.  
*🧩 [Beyond the source lesson] Do not treat extensibility as a trivial technical detail.*

Nó là **lý do chiến lược** để một công ty chọn Postgres.  
*It is a strategic reason for a company to choose Postgres.*

Công ty cần tìm kiếm ngữ nghĩa.  
*A company needs semantic search.*

Công ty không phải mua thêm một cơ sở dữ liệu vector riêng.  
*The company does not have to buy a separate vector database.*

Công ty cũng không phải vận hành thêm hệ thống đó.  
*The company also does not have to operate that extra system.*

Bạn chỉ cắm thêm khả năng đó vào cái database bạn đã có.  
*You just plug that capability into the database you already have.*

Ít hệ thống hơn nghĩa là ít thứ để hỏng hơn.  
*Fewer systems means fewer things to break.*

Ít hệ thống hơn cũng nghĩa là ít người phải trực đêm hơn.  
*Fewer systems also means fewer people on night duty.*

Ít hệ thống hơn cũng nghĩa là ít tiền hơn.  
*Fewer systems also means less money.*

Luận điểm này sẽ quay lại rất mạnh ở Phần 4.  
*This argument comes back forcefully in Part 4.*

### 1.6. Các kiểu dữ liệu của PostgreSQL — nhớ theo nhóm, đừng học vẹt
*1.6. PostgreSQL data types — learn them by group, not by rote*

**Data type** — kiểu dữ liệu — là việc bạn khai báo trước cho database.  
*A data type is something you declare to the database in advance.*

Bạn cho database biết một cột sẽ chứa loại thông tin gì.  
*You tell the database what kind of information a column will hold.*

Loại đó có thể là số nguyên, chữ hoặc ngày tháng.  
*That kind may be an integer, text, or a date.*

Việc khai báo này giúp database lưu trữ hiệu quả hơn.  
*This declaration helps the database store data more efficiently.*

Việc khai báo này cũng ngăn dữ liệu rác lọt vào.  
*This declaration also keeps junk data out.*

Bạn không thể nhét chữ "abc" vào một cột khai là số.  
*You cannot push the text "abc" into a column declared as numeric.*

| Nhóm / Group | Các kiểu tiêu biểu / Typical types | Dùng cho / Used for |
|---|---|---|
| **Numeric** (số) / **Numeric** (numbers) | `integer`, `bigint`, `real`, `double precision`, `numeric` | Số lượng, tuổi, giá tiền / quantities, ages, prices |
| **Character** (chữ) / **Character** (text) | `text`, `varchar(n)`, `char(n)` | Tên, mô tả, nội dung bài viết / names, descriptions, article bodies |
| **Boolean** (đúng/sai) / **Boolean** (true/false) | `boolean` | Cờ bật/tắt, đã thanh toán hay chưa / on-off flags, paid status |
| **Date/Time** (ngày giờ) / **Date/Time** | `date`, `time`, `timestamp`, `timestamptz`, `interval` | Ngày đặt hàng, thời điểm log / order dates, log timestamps |
| **Array** (mảng) / **Array** | `text[]`, `integer[]` | Một cột chứa nhiều giá trị, ví dụ danh sách tag / one column holding many values, such as a tag list |
| **Enumerated** (`enum`) / **Enumerated** (`enum`) | `mood AS ENUM ('sad','ok','happy')` | Tập giá trị cố định, không được phép sai chính tả / a fixed value set with no room for typos |
| **Chuyên biệt** / **Specialized** | `money`, `point`/`polygon`, `inet`/`cidr`, `uuid`, `json`/`jsonb` | Tiền, tọa độ hình học, địa chỉ mạng, mã định danh, dữ liệu dạng JSON / money, geometric coordinates, network addresses, identifiers, JSON data |
| **Qua extension** / **Via extension** | `vector`, **`tsvector`/`tsquery`** | Tìm kiếm ngữ nghĩa / **tìm kiếm toàn văn** — semantic search / **full-text search** |

Ta giải thích vài từ trong bảng để không ai bị bỏ lại.  
*Let us explain a few terms in the table so nobody is left behind.*

**`integer` và `bigint`** đều là số nguyên.  
*Both `integer` and `bigint` are integer types.*

`bigint` chứa được số lớn hơn rất nhiều.  
*`bigint` holds far larger numbers.*

Chọn sai sẽ dẫn tới một sự cố kinh điển.  
*Choosing wrong leads to a classic incident.*

Hệ thống chạy êm vài năm.  
*The system runs smoothly for a few years.*

Sau đó nó đột nhiên gãy.  
*Then it suddenly breaks.*

Nguyên nhân là số đơn hàng vượt quá 2,1 tỷ.  
*The cause is the order count passing 2.1 billion.*

Đó là giới hạn của `integer`.  
*That is the limit of `integer`.*

**`real` và `double precision`** là số thực **dấu phẩy động**.  
*`real` and `double precision` are floating point real numbers.*

Tiếng Anh gọi là floating point.  
*The English term is floating point.*

"Dấu phẩy động" nghĩa là máy tính lưu số gần đúng.  
*"Floating point" means the computer stores an approximate number.*

Máy tính không lưu số tuyệt đối chính xác.  
*The computer does not store an exactly precise number.*

Đổi lại việc tính toán rất nhanh.  
*In exchange the computation is very fast.*

**`numeric`** cũng là số thực.  
*`numeric` is also a real number type.*

`numeric` lưu **chính xác tuyệt đối**.  
*`numeric` stores values with absolute precision.*

`numeric` chậm hơn một chút.  
*`numeric` is a little slower.*

**`varchar(n)` và `text`** đều lưu chữ.  
*Both `varchar(n)` and `text` store text.*

`varchar(n)` giới hạn tối đa `n` ký tự.  
*`varchar(n)` caps the length at `n` characters.*

`text` thì không giới hạn.  
*`text` has no cap.*

Trong Postgres, hai kiểu này gần như không khác nhau về hiệu năng.  
*In Postgres these two types differ almost not at all in performance.*

Vì vậy bạn cứ dùng `text` cho tiện.  
*So just use `text` for convenience.*

**`timestamptz`** là timestamp **có kèm múi giờ**.  
*`timestamptz` is a timestamp with a time zone.*

Chữ tz là viết tắt của time zone.  
*The letters tz stand for time zone.*

Đây là kiểu bạn nên dùng mặc định cho mọi mốc thời gian.  
*This is the type you should default to for every point in time.*

Lý do là hệ thống thật luôn có người dùng ở nhiều múi giờ.  
*The reason is that real systems always have users in many time zones.*

**`json` và `jsonb`** lưu dữ liệu dạng JSON.  
*`json` and `jsonb` store JSON data.*

JSON là một định dạng văn bản để biểu diễn dữ liệu có cấu trúc lồng nhau.  
*JSON is a text format for representing nested structured data.*

`jsonb` là bản đã được xử lý sẵn thành dạng nhị phân.  
*`jsonb` is the version pre-processed into binary form.*

Vì vậy `jsonb` truy vấn nhanh hơn.  
*So `jsonb` queries faster.*

Bạn gần như luôn chọn `jsonb`.  
*You almost always pick `jsonb`.*

⚠️ **Chỗ khó — cái bẫy tiền bạc kinh điển**  
*⚠️ A hard spot — the classic money trap*

Dữ liệu của bạn là **tiền**.  
*Your data is money.*

Hãy dùng `numeric` cho nó.  
*Please use `numeric` for it.*

Bạn cũng có thể dùng `money`.  
*You may also use `money`.*

Bạn **tuyệt đối không** dùng `real` hay `double precision`.  
*You absolutely must not use `real` or `double precision`.*

Lý do là số dấu phẩy động lưu gần đúng.  
*The reason is that floating point stores approximations.*

Phép tính `0.1 + 0.2` trong máy tính ra `0.30000000000000004`.  
*The computation `0.1 + 0.2` gives `0.30000000000000004` on a computer.*

Kết quả đó không phải `0.3`.  
*That result is not `0.3`.*

Sai số cộng dồn qua hàng triệu giao dịch.  
*The error accumulates across millions of transactions.*

Sổ sách kế toán sẽ lệch.  
*The accounting books will drift.*

Đây là lỗi mà rất nhiều lập trình viên mới mắc phải.  
*This is a mistake many new programmers make.*

Nó khó phát hiện.  
*It is hard to spot.*

Lúc test với vài dòng dữ liệu, mọi thứ trông vẫn đúng.  
*When you test with a few rows, everything still looks correct.*

### 1.7. Vấn đề thực tế: vì sao cách tìm kiếm ngây thơ lại hỏng
*1.7. The real-world problem: why the naive search approach breaks*

Giờ ta bước vào trọng tâm của bài.  
*Now we step into the heart of the lesson.*

Bạn có một bảng tên `articles` — bài viết.  
*You have a table named `articles`.*

Bảng này có một cột `body` kiểu `text`.  
*This table has a `body` column of type `text`.*

Cột đó chứa nội dung bài viết.  
*That column holds the article content.*

Người dùng gõ vào ô tìm kiếm chữ "running".  
*A user types the word "running" into the search box.*

Cách làm đầu tiên ai cũng nghĩ ra là dùng toán tử `LIKE` của SQL.  
*The first approach everyone thinks of uses the SQL `LIKE` operator.*

```sql
SELECT title FROM articles WHERE body LIKE '%running%';
```

**`LIKE`** là toán tử so khớp chuỗi ký tự trong SQL.  
*`LIKE` is the string-matching operator in SQL.*

Dấu **`%`** là **wildcard** — ký tự đại diện.  
*The `%` sign is a wildcard.*

Nó nghĩa là "chỗ này có thể là bất cứ thứ gì, dài ngắn tùy ý".  
*It means "anything can go here, of any length".*

Vậy `'%running%'` nghĩa là "chuỗi nào có chứa *running* ở đâu đó bên trong".  
*So `'%running%'` means "any string containing running somewhere inside".*

Trước và sau nó là gì cũng được.  
*Anything may come before it and after it.*

Câu lệnh này chạy được.  
*This statement does run.*

Nó lại hỏng theo ba cách khác nhau.  
*It breaks in three different ways.*

**Vấn đề thứ nhất — chậm và không thể mở rộng**  
*Problem one — slow and unable to scale*

Để hiểu lý do nó chậm, bạn cần biết **index** — chỉ mục — là gì.  
*To understand why it is slow, you need to know what an index is.*

Index là một cấu trúc dữ liệu phụ.  
*An index is an auxiliary data structure.*

Database dựng thêm nó bên cạnh bảng.  
*The database builds it alongside the table.*

Mục đích của nó là giúp tìm kiếm nhanh hơn.  
*Its purpose is to make searching faster.*

Nó hoàn toàn giống phần mục lục tra cứu ở cuối một cuốn sách.  
*It works just like the index at the back of a book.*

Loại index mặc định của hầu hết database tên là **B-tree** — cây B.  
*The default index type in most databases is called a B-tree.*

Nó sắp xếp giá trị theo thứ tự.  
*It keeps values in sorted order.*

Nhờ vậy nó tìm rất nhanh.  
*This makes it search very fast.*

Điều kiện là **bạn biết đoạn đầu của giá trị cần tìm**.  
*The condition is that you know the beginning of the value you want.*

Và đó chính là vấn đề ở đây.  
*And that is exactly the problem here.*

Chuỗi `'%running%'` có dấu `%` ở **đầu**.  
*The pattern `'%running%'` has a `%` at the start.*

Tức là bạn không biết chuỗi bắt đầu bằng gì.  
*This means you do not know what the string starts with.*

B-tree trở nên vô dụng.  
*The B-tree becomes useless.*

Chuyện này giống như bạn tra từ điển theo một cách rất kỳ.  
*This is like looking up a dictionary in a very odd way.*

Bạn chỉ biết "từ này có chứa vần *ung* ở giữa".  
*You only know that "this word has ung somewhere in the middle".*

Bạn buộc phải lật hết từ điển.  
*You are forced to flip through the whole dictionary.*

Index không dùng được.  
*The index becomes unusable.*

Khi đó database rơi vào chế độ **sequential scan** — quét tuần tự.  
*The database then falls into sequential scan mode.*

Người ta viết tắt nó là seq scan.  
*People abbreviate it as seq scan.*

Database đọc lần lượt **từng hàng một** trong bảng rồi so khớp.  
*The database reads every single row in turn and compares.*

Bảng có 10 triệu hàng.  
*Suppose the table has 10 million rows.*

Nó đọc đủ 10 triệu hàng.  
*It reads all 10 million rows.*

Thời gian chạy tỉ lệ thuận với **tổng số hàng**.  
*The runtime is proportional to the total row count.*

Bảng càng lớn càng chậm.  
*The bigger the table, the slower it gets.*

Không có cách nào cứu được.  
*There is no way to save it.*

**Vấn đề thứ hai — nó không hiểu biến thể của từ**  
*Problem two — it does not understand word variants*

Tiếng Anh biến đổi từ liên tục.  
*English inflects words constantly.*

Các từ "run", "runs", "ran" và "running" đều là cùng một hành động.  
*The words "run", "runs", "ran", and "running" all name the same action.*

`LIKE '%running%'` lại so khớp **từng ký tự một**.  
*`LIKE '%running%'` compares character by character instead.*

Vì vậy nó chỉ tìm ra đúng chuỗi "running".  
*So it only finds the exact string "running".*

Một bài viết hay ho có câu "he ran a marathon".  
*A great article may contain the sentence "he ran a marathon".*

Bài đó sẽ bị bỏ sót hoàn toàn.  
*That article gets missed completely.*

Nó đúng là thứ người dùng đang cần.  
*It is exactly what the user wants.*

**Vấn đề thứ ba — nó khớp nhầm**  
*Problem three — it matches the wrong things*

Bạn nới lỏng điều kiện thành `LIKE '%run%'`.  
*You loosen the condition to `LIKE '%run%'`.*

Bạn muốn bắt được nhiều biến thể hơn.  
*You want to catch more variants.*

Bạn lập tức khớp trúng cả "b**run**ch" — bữa xế.  
*You immediately also match "brunch".*

Bạn khớp trúng cả "p**run**e" — mận khô.  
*You also match "prune".*

Bạn khớp trúng cả "**run**ny nose" — sổ mũi.  
*You also match "runny nose".*

Kết quả trả về đầy rác.  
*The returned results are full of junk.*

Lý do là `LIKE` không biết đâu là ranh giới của một từ.  
*The reason is that `LIKE` does not know where a word boundary is.*

Với nó, mọi thứ chỉ là một dãy ký tự dài.  
*To it, everything is just one long character sequence.*

**Tóm lại**  
*In summary*

`LIKE` thất bại vì một lý do duy nhất.  
*`LIKE` fails for one single reason.*

Nó làm việc ở tầng **ký tự**.  
*It operates at the character level.*

Muốn tìm kiếm văn bản cho tử tế, ta cần một công cụ làm việc ở tầng **từ**.  
*For decent text search, we need a tool that operates at the word level.*

Công cụ đó phải hiểu rằng các biến thể của một từ thuộc về cùng một khái niệm.  
*That tool must understand that variants of a word belong to one concept.*

### 1.8. Analogy: mục lục cuối cuốn sách
*1.8. Analogy: the index at the back of a book*

Trước khi vào định nghĩa kỹ thuật, hãy dựng cho xong một phép ví von.  
*Before the technical definitions, let us finish building one analogy.*

Nó sẽ theo bạn suốt phần còn lại của giáo trình.  
*It will follow you through the rest of this course.*

Nó cũng sẽ theo bạn vào cả phòng phỏng vấn.  
*It will also follow you into the interview room.*

Bạn cầm một cuốn sách kỹ thuật dày 800 trang.  
*You hold an 800-page technical book.*

Bạn muốn biết chỗ nào nói về "database".  
*You want to know where it talks about "database".*

Có hai cách làm.  
*There are two approaches.*

**Cách một** là mở trang 1 rồi đọc từng dòng cho tới trang 800.  
*Approach one opens page 1 and reads every line up to page 800.*

Bạn ghi lại mọi chỗ thấy chữ "database".  
*You note every place you see the word "database".*

Cách này luôn cho kết quả đúng.  
*This approach always gives a correct result.*

Cách này mất cả ngày.  
*This approach takes all day.*

**Đây chính xác là `LIKE '%database%'`.**  
*This is exactly `LIKE '%database%'`.*

Đó là quét tuần tự toàn bộ.  
*That is a full sequential scan.*

**Cách hai** là lật ra trang Index ở cuối sách.  
*Approach two flips to the Index page at the back.*

Ở đó có sẵn dòng `database ....... 12, 45, 88`.  
*There sits the line `database ....... 12, 45, 88`.*

Bạn nhảy thẳng tới ba trang đó.  
*You jump straight to those three pages.*

Việc đó mất 5 giây.  
*That takes 5 seconds.*

Bây giờ là chi tiết quan trọng nhất của phép ví von.  
*Now comes the most important detail of the analogy.*

Nhiều người kể analogy xong rồi bỏ dở chỗ này.  
*Many people tell the analogy and then drop this part.*

Hãy để ý một điều.  
*Please notice one thing.*

**Thời gian tra mục lục không phụ thuộc vào việc cuốn sách dày bao nhiêu.**  
*The lookup time does not depend on how thick the book is.*

Sách có thể dày 800 trang.  
*The book may be 800 pages.*

Sách có thể dày 8.000 trang.  
*The book may be 8,000 pages.*

Bạn vẫn lật đúng một trang mục lục.  
*You still flip to exactly one index page.*

Sau đó bạn nhảy tới đúng số trang cần đọc.  
*Then you jump to exactly the pages you need.*

Thứ quyết định công sức của bạn là **số trang được liệt kê ra**.  
*What determines your effort is the number of pages listed.*

Đó chính là số kết quả khớp.  
*That is the number of matching results.*

Độ dày cuốn sách không quyết định điều đó.  
*The thickness of the book does not determine it.*

Full-text search làm y hệt như vậy.  
*Full-text search does exactly the same thing.*

Nó dựng sẵn một cấu trúc gọi là **inverted index** — chỉ mục ngược.  
*It pre-builds a structure called an inverted index.*

Đó là một bảng tra cứu ánh xạ **"từ" → "danh sách các tài liệu chứa từ đó"**.  
*It is a lookup table mapping a word to the list of documents containing it.*

Vì sao người ta gọi nó là "ngược"?  
*Why do people call it "inverted"?*

Cách lưu trữ thông thường là từ tài liệu suy ra các từ.  
*The normal storage direction goes from a document to its words.*

Nó trả lời câu hỏi "bài số 12 chứa những từ nào".  
*It answers "which words does article 12 contain".*

Inverted index lật ngược chiều đó lại.  
*The inverted index flips that direction around.*

Nó đi từ một từ suy ra danh sách tài liệu.  
*It goes from one word to a list of documents.*

Nó trả lời câu hỏi "từ *database* xuất hiện ở những bài nào".  
*It answers "which articles contain the word database".*

Chính chiều lật ngược này khiến việc tìm kiếm trở nên tức thì.  
*This flipped direction is what makes search instant.*

Full-text search có một thứ nữa.  
*Full-text search has one more thing.*

Mục lục sách vẫn còn thiếu thứ đó.  
*A book index still lacks that thing.*

Mục lục sách ghi "running" và "ran" thành hai mục riêng biệt.  
*A book index records "running" and "ran" as two separate entries.*

Full-text search khôn hơn.  
*Full-text search is smarter.*

Nó gộp cả hai về cùng một mục.  
*It merges both into a single entry.*

Cơ chế gộp đó tên là *stemming*.  
*That merging mechanism is called stemming.*

Ta sẽ mổ nó ngay bây giờ.  
*We will dissect it right now.*

### 1.9. Bộ thuật ngữ cốt lõi của full-text search
*1.9. The core vocabulary of full-text search*

Đây là 8 từ bạn phải nắm chắc.  
*These are the 8 words you must know solidly.*

Hãy đọc chậm, mỗi từ một lần.  
*Please read slowly, one word at a time.*

**Full-text search** viết tắt là **FTS** — tìm kiếm toàn văn.  
*Full-text search is abbreviated FTS.*

FTS là kỹ thuật tìm trong một tập tài liệu viết bằng ngôn ngữ tự nhiên.  
*FTS is a technique for searching a set of natural-language documents.*

Ngôn ngữ tự nhiên là tiếng người.  
*Natural language means human language.*

Nó không phải mã máy.  
*It is not machine code.*

FTS tìm ra những tài liệu khớp nhất với truy vấn của người dùng.  
*FTS finds the documents that best match the user's query.*

**Token** — đơn vị từ — là một mẩu văn bản được cắt ra từ câu.  
*A token is a piece of text cut out of a sentence.*

Nó thường là một từ.  
*It is usually one word.*

Hành động cắt đó gọi là **tokenize** — tách token.  
*That cutting action is called tokenizing.*

Ví dụ là câu "the quick fox".  
*One example is the sentence "the quick fox".*

Câu này được tokenize thành ba token là `the`, `quick` và `fox`.  
*This sentence tokenizes into three tokens: `the`, `quick`, and `fox`.*

**Stemming** nghĩa đen là rút về gốc từ.  
*Stemming literally means reducing a word to its stem.*

Chữ "stem" nghĩa là thân hoặc gốc cây.  
*The word "stem" means the trunk or root of a plant.*

Stemming là quá trình cắt bỏ các đuôi biến đổi của một từ.  
*Stemming is the process of cutting off a word's inflectional endings.*

Nó đưa từ về dạng gốc chung.  
*It brings the word back to a shared base form.*

Ví dụ là `running` rút thành `run`.  
*One example is `running` reducing to `run`.*

Ví dụ khác là `foxes` rút thành `fox`.  
*Another example is `foxes` reducing to `fox`.*

Ví dụ nữa là `databases` rút thành `databas`.  
*A further example is `databases` reducing to `databas`.*

Bạn có thể thấy lạ khi `databases` biến thành `databas`.  
*You may find it odd that `databases` becomes `databas`.*

Dạng đó thiếu mất chữ "e".  
*That form is missing the letter "e".*

Đây là một điểm quan trọng.  
*This is an important point.*

**Kết quả của stemming không nhất thiết là một từ có thật trong từ điển.**  
*The output of stemming is not necessarily a real dictionary word.*

Thuật toán chỉ cần đảm bảo một điều.  
*The algorithm only needs to guarantee one thing.*

Mọi biến thể của cùng một từ đều rút về **cùng một dạng**.  
*Every variant of the same word reduces to the same form.*

Dạng đó nhìn có thể kỳ cục.  
*That form may look strange.*

Cả "database" và "databases" đều ra `databas`.  
*Both "database" and "databases" come out as `databas`.*

Việc so khớp vì vậy vẫn đúng.  
*The matching therefore still works.*

**Lexeme** — đơn vị từ vựng — đọc là "léc-xim".  
*Lexeme is pronounced "léc-xim" in Vietnamese.*

Lexeme là kết quả cuối cùng sau khi một token đã được chuẩn hóa xong.  
*A lexeme is the final result after a token finishes normalization.*

Đây chính là **đơn vị tìm kiếm thật sự** của FTS.  
*This is the real unit of search in FTS.*

FTS không so ký tự.  
*FTS does not compare characters.*

FTS cũng không so token.  
*FTS does not compare tokens either.*

FTS so lexeme.  
*FTS compares lexemes.*

**Stop-word** — từ dừng — là những từ xuất hiện quá nhiều.  
*A stop-word is a word that appears far too often.*

Chúng không giúp phân biệt tài liệu nào với tài liệu nào.  
*They do not help tell one document from another.*

Ví dụ là "the", "is", "a", "and" và "of".  
*Examples are "the", "is", "a", "and", and "of".*

Chúng bị **loại bỏ hoàn toàn** khi xử lý.  
*They get removed entirely during processing.*

Lý do rất thực dụng.  
*The reason is very practical.*

Bạn giữ lại chữ "the".  
*Suppose you keep the word "the".*

Gần như 100% tài liệu đều chứa nó.  
*Nearly 100% of documents contain it.*

Danh sách tài liệu cho từ đó sẽ dài vô ích.  
*The document list for that word becomes uselessly long.*

Danh sách đó vừa tốn chỗ vừa không giúp tìm ra gì.  
*That list wastes space and helps find nothing.*

**`tsvector`** có chữ ts viết tắt của *text search*.  
*In `tsvector` the letters ts stand for text search.*

`tsvector` là một kiểu dữ liệu của PostgreSQL.  
*`tsvector` is a PostgreSQL data type.*

Nó dùng để lưu **danh sách các lexeme đã loại trùng và đã sắp xếp**.  
*It stores a deduplicated and sorted list of lexemes.*

Nó lưu kèm theo vị trí của chúng trong văn bản gốc.  
*It also stores their positions in the original text.*

Hãy nghĩ về nó như phiên bản đã được tiền xử lý sẵn của tài liệu.  
*Think of it as a pre-processed version of the document.*

Đó là dạng mà máy tìm kiếm đọc dễ nhất.  
*That is the form a search engine reads most easily.*

**`tsquery`** là kiểu dữ liệu lưu **truy vấn tìm kiếm**.  
*`tsquery` is the data type that stores a search query.*

Nó cũng ở dạng lexeme.  
*It is also in lexeme form.*

Các lexeme được nối với nhau bằng các toán tử logic.  
*The lexemes get joined by logical operators.*

Toán tử `&` là AND.  
*The `&` operator is AND.*

AND nghĩa là phải có cả hai.  
*AND means both must be present.*

Toán tử `|` là OR.  
*The `|` operator is OR.*

OR nghĩa là có một trong hai là được.  
*OR means either one is enough.*

Toán tử `!` là NOT.  
*The `!` operator is NOT.*

NOT nghĩa là không được chứa.  
*NOT means it must not be present.*

Toán tử `<->` là FOLLOWED BY.  
*The `<->` operator is FOLLOWED BY.*

FOLLOWED BY nghĩa là từ này phải đứng ngay trước từ kia.  
*FOLLOWED BY means this word must stand directly before the other one.*

**`@@`** là **toán tử match** — so khớp.  
*`@@` is the match operator.*

Nó đặt giữa một `tsvector` và một `tsquery`.  
*It sits between a `tsvector` and a `tsquery`.*

Nó trả lời đúng một câu hỏi.  
*It answers exactly one question.*

Câu hỏi đó là "tài liệu này có thỏa mãn truy vấn kia không?".  
*That question is "does this document satisfy that query?".*

Kết quả là `true` hoặc `false`.  
*The result is `true` or `false`.*

Cuối cùng là một phân biệt mà bài giảng gốc nhấn mạnh.  
*Finally comes a distinction the source lecture emphasizes.*

Đó là **RegEx so với FTS**.  
*That is RegEx versus FTS.*

**RegEx** là viết tắt của *Regular Expression* — biểu thức chính quy.  
*RegEx stands for Regular Expression.*

RegEx là công cụ tìm chuỗi theo một khuôn mẫu ký tự.  
*RegEx is a tool for finding strings by a character pattern.*

Ví dụ là "tìm mọi chuỗi bắt đầu bằng chữ hoa rồi tới 3 chữ số".  
*One example is "find every string starting with a capital letter then 3 digits".*

RegEx rất mạnh.  
*RegEx is very powerful.*

RegEx vẫn làm việc ở tầng **ký tự**.  
*RegEx still operates at the character level.*

Nó giống `LIKE` ở điểm này.  
*It resembles `LIKE` in this respect.*

FTS làm việc ở tầng **lexeme**.  
*FTS operates at the lexeme level.*

Đó là khác biệt về bản chất.  
*That is a difference in kind.*

Đó không phải khác biệt về mức độ.  
*That is not a difference in degree.*

### 1.10. Ví dụ chạy tay — tự mình biến một câu thành `tsvector`
*1.10. A hand-run example — turn a sentence into a `tsvector` yourself*

Lý thuyết đủ rồi.  
*That is enough theory.*

Giờ ta làm thủ công từng bước.  
*Now we work through it manually step by step.*

Bạn sẽ **nhìn thấy** dữ liệu biến đổi.  
*You will see the data transform.*

Bạn không chỉ nghe mô tả.  
*You do not merely hear a description.*

Câu đầu vào là **"The quick brown foxes are running"**.  
*The input sentence is "The quick brown foxes are running".*

**Bước 1 — Tokenize (tách thành token)**  
*Step 1 — Tokenize (split into tokens)*

PostgreSQL cắt câu theo khoảng trắng và dấu câu.  
*PostgreSQL splits the sentence on whitespace and punctuation.*

```
The | quick | brown | foxes | are | running
 ①      ②       ③       ④      ⑤       ⑥
```

Ta có sáu token.  
*We have six tokens.*

Ta ghi luôn số thứ tự vị trí của từng token.  
*We also record the position number of each token.*

Con số này sẽ có ích ở bước cuối.  
*This number becomes useful in the last step.*

**Bước 2 — Loại bỏ stop-word**  
*Step 2 — Remove stop-words*

Trong bộ từ điển tiếng Anh của Postgres, `the` và `are` nằm trong danh sách stop-word.  
*In the Postgres English dictionary, `the` and `are` are in the stop-word list.*

```
~~The~~ | quick | brown | foxes | ~~are~~ | running
           ②       ③       ④              ⑥
```

Còn lại bốn token.  
*Four tokens remain.*

Chúng là `quick` ở vị trí 2 và `brown` ở vị trí 3.  
*They are `quick` at position 2 and `brown` at position 3.*

Hai token còn lại là `foxes` ở vị trí 4 và `running` ở vị trí 6.  
*The other two are `foxes` at position 4 and `running` at position 6.*

Chú ý rằng **vị trí không bị đánh số lại**.  
*Notice that the positions do not get renumbered.*

Token `running` vẫn giữ số 6.  
*The token `running` keeps the number 6.*

Nó không tụt xuống 4.  
*It does not drop to 4.*

Điều này quan trọng cho một mục đích về sau.  
*This matters for a later purpose.*

Ta còn cần biết các từ có đứng cạnh nhau trong câu gốc hay không.  
*We still need to know whether words stood next to each other in the original sentence.*

**Bước 3 — Stemming (chuẩn hóa về lexeme)**  
*Step 3 — Stemming (normalize into lexemes)*

Bộ stemmer xử lý từng token.  
*The stemmer processes each token.*

| Token / Token | → Lexeme / → Lexeme | Ghi chú / Note |
|---|---|---|
| `quick` | `quick` | không có đuôi biến đổi, giữ nguyên / no inflectional ending, kept as is |
| `brown` | `brown` | giữ nguyên / kept as is |
| `foxes` | `fox` | cắt đuôi số nhiều `-es` / the plural ending `-es` is cut |
| `running` | `run` | cắt đuôi `-ning` / the ending `-ning` is cut |

**Bước 4 — Sắp xếp theo thứ tự chữ cái, loại trùng, gắn vị trí**  
*Step 4 — Sort alphabetically, deduplicate, attach positions*

Kết quả cuối cùng chính là `tsvector`.  
*The final result is the `tsvector`.*

```
'brown':3 'fox':4 'quick':2 'run':6
```

Hãy đọc kỹ dòng trên.  
*Please read the line above carefully.*

Mỗi phần tử gồm một lexeme trong dấu nháy.  
*Each element holds one lexeme in quotes.*

Theo sau là dấu hai chấm và vị trí của nó trong câu gốc.  
*A colon and its position in the original sentence follow.*

Danh sách được sắp theo thứ tự chữ cái.  
*The list is sorted alphabetically.*

Thứ tự đó là brown, fox, quick, run.  
*That order is brown, fox, quick, run.*

Danh sách không theo thứ tự trong câu.  
*The list does not follow the sentence order.*

Và đây là khoảnh khắc "à há" của cả bài học.  
*And here is the aha moment of the whole lesson.*

**Trong `tsvector` này có lexeme `run`.**  
*This `tsvector` contains the lexeme `run`.*

Câu gốc không hề chứa chuỗi ký tự "run" đứng riêng lẻ.  
*The original sentence never contains the standalone string "run".*

Câu gốc chỉ có "running".  
*The original sentence only has "running".*

Người dùng tìm "run".  
*A user searches for "run".*

Câu này vẫn khớp.  
*This sentence still matches.*

Đó chính xác là điều mà `LIKE` không bao giờ làm được.  
*That is exactly what `LIKE` can never do.*

### 1.11. Code "hello world" — chạy được thật, chú thích từng dòng
*1.11. The "hello world" code — it really runs, annotated line by line*

Bây giờ ta để PostgreSQL tự làm lại đúng bốn bước trên.  
*Now we let PostgreSQL redo those same four steps itself.*

Ta kiểm chứng xem kết quả có khớp với phần chạy tay không.  
*We check whether the result matches our hand-run version.*

```sql
-- ────────────────────────────────────────────────────────────
-- BƯỚC 1: Biến một đoạn văn bản thành tsvector
-- STEP 1: Turn a piece of text into a tsvector
-- to_tsvector nhận 2 tham số:
-- to_tsvector takes 2 arguments:
--   (a) tên "text search configuration" — ở đây là 'english',
--   (a) the name of the text search configuration, here 'english',
--       nói cho Postgres biết dùng bộ luật của ngôn ngữ nào để
--       which tells Postgres which language rule set to use for
--       tách từ, bỏ stop-word và stem;
--       tokenizing, removing stop-words, and stemming;
--   (b) đoạn văn bản cần xử lý.
--   (b) the piece of text to process.
-- ────────────────────────────────────────────────────────────
SELECT to_tsvector('english', 'The quick brown foxes are running');
-- Kết quả:  'brown':3 'fox':4 'quick':2 'run':6
-- Result:   'brown':3 'fox':4 'quick':2 'run':6
-- → đúng y hệt phần chạy tay ở mục 1.10.
-- → exactly the same as the hand-run version in section 1.10.


-- ────────────────────────────────────────────────────────────
-- BƯỚC 2: Biến truy vấn của người dùng thành tsquery
-- STEP 2: Turn the user query into a tsquery
-- Điểm mấu chốt: truy vấn cũng bị stem giống hệt tài liệu.
-- Key point: the query gets stemmed exactly like the document.
-- ────────────────────────────────────────────────────────────
SELECT to_tsquery('english', 'running');
-- Kết quả:  'run'
-- Result:   'run'
-- → người dùng gõ "running", nhưng thứ đem đi so khớp là 'run'.
-- → the user typed "running", yet the thing matched is 'run'.


-- ────────────────────────────────────────────────────────────
-- BƯỚC 3: So khớp bằng toán tử @@ → trả về true / false
-- STEP 3: Match with the @@ operator → returns true / false
-- Đọc câu này là: "cái tsvector bên trái có thỏa cái tsquery
-- Read this as: "does the tsvector on the left satisfy the tsquery
-- bên phải không?"
-- on the right?"
-- ────────────────────────────────────────────────────────────
SELECT to_tsvector('english', 'The quick brown foxes are running')
    @@ to_tsquery('english', 'running');
-- Kết quả:  t   (viết tắt của true — có khớp)
-- Result:   t   (short for true — it matches)


-- ────────────────────────────────────────────────────────────
-- BƯỚC 4: Áp dụng lên một bảng thật
-- STEP 4: Apply it to a real table
-- Với mỗi hàng trong bảng articles, Postgres biến cột body
-- For each row in the articles table, Postgres turns the body column
-- thành tsvector rồi so với tsquery. Hàng nào trả true thì lấy.
-- into a tsvector and compares it to the tsquery. Rows returning true are kept.
-- ────────────────────────────────────────────────────────────
SELECT title
FROM articles
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'running');
-- → trả về mọi bài viết có chứa run / runs / ran / running.
-- → returns every article containing run / runs / ran / running.
```

**Điểm cần khắc cốt ghi tâm từ đoạn code này**  
*The point to burn into your memory from this code*

Cả **tài liệu** lẫn **truy vấn** đều đi qua **cùng một bộ chuẩn hóa**.  
*Both the document and the query pass through the same normalizer.*

Chúng cùng tách token, cùng bỏ stop-word, cùng stem.  
*They share the same tokenizing, stop-word removal, and stemming.*

Nhờ vậy chúng "gặp nhau" ở tầng lexeme.  
*This lets them meet at the lexeme level.*

Bạn dùng config khác nhau cho hai bên.  
*Suppose you use different configurations on the two sides.*

Ví dụ là tài liệu dùng `'english'` còn truy vấn dùng `'simple'`.  
*For example the document uses `'english'` and the query uses `'simple'`.*

Khi đó chúng sẽ không gặp nhau.  
*They will then never meet.*

Kết quả sẽ sai một cách khó hiểu.  
*The result will be wrong in a baffling way.*

### 1.12. Nối lại với ví dụ trong bài giảng gốc
*1.12. Reconnecting with the example from the source lecture*

Bài gốc đưa ra một ví dụ.  
*The source lecture gives one example.*

Ví dụ đó là tìm cụm "hands-on data scientist".  
*That example searches for the phrase "hands-on data scientist".*

Cụ thể là ta kiểm tra xem tài liệu có chứa "hands-on" hay không.  
*Concretely, we check whether the document contains "hands-on".*

Ta cũng kiểm tra xem "data" có đứng ngay trước "scientist" hay không.  
*We also check whether "data" stands directly before "scientist".*

Rồi hệ thống trả về `t`.  
*The system then returns `t`.*

Ta hãy diễn giải lại bằng ngôn ngữ vừa học.  
*Let us restate that in the vocabulary we just learned.*

Tài liệu được lưu dưới dạng `tsvector`.  
*The document is stored as a `tsvector`.*

Cụm cần tìm được viết thành `tsquery`.  
*The phrase to find is written as a `tsquery`.*

`tsquery` đó dùng toán tử `<->`.  
*That `tsquery` uses the `<->` operator.*

Toán tử này yêu cầu "data" phải đứng liền trước "scientist".  
*This operator demands that "data" stand immediately before "scientist".*

Toán tử `@@` cho biết có khớp hay không.  
*The `@@` operator tells you whether it matches.*

Postgres đã ghi lại **vị trí** của từng lexeme ở bước 4 trong mục 1.10.  
*Postgres recorded the position of each lexeme in step 4 of section 1.10.*

Chính nhờ vậy nó mới trả lời được câu hỏi "hai từ này có đứng cạnh nhau không".  
*That alone lets it answer "do these two words stand next to each other".*

### 1.13. 🧩 [Ngoài bài gốc] — Ba điều bài gốc không nói mà bạn nên biết ngay từ Basic
*1.13. 🧩 [Beyond the source lesson] — three things the source lesson omits that you should know from Basic*

**Một: FTS không phải là "AI hiểu nghĩa".**  
*One: FTS is not an "AI that understands meaning".*

Nó vẫn chỉ là so khớp từ.  
*It is still just word matching.*

Nó chỉ so khớp *thông minh hơn* nhờ chuẩn hóa.  
*It merely matches more intelligently thanks to normalization.*

Nó tuyệt đối không biết "xe hơi" và "ô tô" là cùng một thứ.  
*It absolutely does not know that "xe hơi" and "ô tô" are the same thing.*

Lý do là hai cụm này ra hai lexeme khác nhau.  
*The reason is that these two phrases produce two different lexemes.*

Bạn muốn máy hiểu nghĩa.  
*You may want the machine to understand meaning.*

Khi đó bạn cần vector search.  
*You then need vector search.*

Phần 4 sẽ ghép hai thứ lại với nhau.  
*Part 4 will combine the two.*

**Hai: FTS có thể phá vỡ kỳ vọng của người dùng.**  
*Two: FTS can break user expectations.*

Một người tìm chính xác tên sản phẩm "iPhone 15 Pro".  
*Someone searches for the exact product name "iPhone 15 Pro".*

Người đó có thể nhận về kết quả lệch.  
*That person may get skewed results.*

Lý do là stemmer đôi khi cắt bậy các mã sản phẩm.  
*The reason is that a stemmer sometimes chops product codes wrongly.*

Có những cột chứa mã, tên riêng và username.  
*Some columns hold codes, proper names, and usernames.*

Với những cột đó, người ta thường dùng config `'simple'`.  
*For those columns, people usually use the `'simple'` configuration.*

Config `'simple'` không stem và không bỏ stop-word.  
*The `'simple'` configuration does not stem and does not drop stop-words.*

Nó chỉ chuyển văn bản thành chữ thường.  
*It only lowercases the text.*

Người ta không dùng `'english'` cho những cột đó.  
*People do not use `'english'` for those columns.*

**Ba: `to_tsvector` tính lại mỗi lần chạy.**  
*Three: `to_tsvector` recomputes on every run.*

Hãy nhìn lại câu lệnh ở BƯỚC 4 phía trên.  
*Look back at the statement in STEP 4 above.*

Mỗi lần bạn tìm kiếm, Postgres phải xử lý lại toàn bộ nội dung của **mọi hàng** trong bảng.  
*On every search, Postgres reprocesses the entire content of every row.*

Với bảng lớn thì đây là thảm họa hiệu năng.  
*On a large table this is a performance disaster.*

Cách chữa thứ nhất là dựng index.  
*The first cure is to build an index.*

Cách chữa thứ hai là lưu sẵn `tsvector` vào một cột.  
*The second cure is to store the `tsvector` in a column ahead of time.*

Đó là nội dung chính của Phần 2.  
*That is the main content of Part 2.*

### ✅ Self-check Phần 1
*✅ Self-check for Part 1*

Hãy trả lời bằng lời của chính bạn trước khi xem gợi ý.  
*Please answer in your own words before looking at the hints.*

**1. Nêu ba lý do khiến `WHERE body LIKE '%running%'` là lựa chọn tồi cho tìm kiếm văn bản.**  
*1. Give three reasons why `WHERE body LIKE '%running%'` is a poor choice for text search.*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> (a) Câu lệnh không dùng được B-tree index.  
> *(a) The statement cannot use a B-tree index.*
>
> Lý do là dấu `%` nằm ở đầu.  
> *The reason is the leading `%`.*
>
> Database rơi về sequential scan.  
> *The database falls back to a sequential scan.*
>
> Thời gian chạy tỉ lệ với tổng số hàng.  
> *The runtime scales with the total row count.*
>
> (b) Câu lệnh không hiểu biến thể từ.  
> *(b) The statement does not understand word variants.*
>
> Nó bỏ sót "ran" và "runs".  
> *It misses "ran" and "runs".*
>
> (c) Câu lệnh khớp nhầm vào giữa từ khác.  
> *(c) The statement matches inside other words by mistake.*
>
> Ví dụ là "brunch" khi bạn nới thành `%run%`.  
> *One example is "brunch" once you loosen it to `%run%`.*

**2. Lexeme là gì? Các từ "cats", "cat", "catty" sẽ về lexeme nào?**  
*2. What is a lexeme? Which lexemes do "cats", "cat", and "catty" reduce to?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Lexeme là dạng đã chuẩn hóa của một từ.  
> *A lexeme is the normalized form of a word.*
>
> Nó được dùng làm đơn vị so khớp của FTS.  
> *It serves as the matching unit of FTS.*
>
> Cả "cats" và "cat" đều về `cat`.  
> *Both "cats" and "cat" reduce to `cat`.*
>
> Từ "catty" là tính từ khác gốc nghĩa.  
> *The word "catty" is an adjective with a different sense root.*
>
> Stemmer tiếng Anh thường để nó thành `catti`.  
> *The English stemmer usually turns it into `catti`.*
>
> Đó là một lexeme riêng.  
> *That is a separate lexeme.*
>
> Điều này cho thấy stemming là *heuristic* — quy tắc gần đúng.  
> *This shows that stemming is a heuristic.*
>
> Stemming không hoàn hảo.  
> *Stemming is not perfect.*

**3. Dự đoán kết quả của `to_tsvector('english','The cats are sleeping')`. Vì sao "the" và "are" biến mất?**  
*3. Predict the result of `to_tsvector('english','The cats are sleeping')`. Why do "the" and "are" disappear?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Kết quả là `'cat':2 'sleep':4`.  
> *The result is `'cat':2 'sleep':4`.*
>
> Hai từ "the" và "are" là stop-word.  
> *The two words "the" and "are" are stop-words.*
>
> Chúng quá phổ biến.  
> *They are far too common.*
>
> Chúng không giúp phân biệt tài liệu nào với tài liệu nào.  
> *They do not help tell one document from another.*
>
> Vì vậy chúng bị loại bỏ.  
> *So they get removed.*
>
> Nhờ vậy index gọn hơn và nhanh hơn.  
> *This makes the index smaller and faster.*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

> Từ đây trở đi, giáo trình giả định bạn đã nắm chắc một số khái niệm.  
> *From here on, the course assumes you have a firm grip on several concepts.*
>
> Đó là lexeme, stemming, stop-word, `tsvector`, `tsquery` và `@@`.  
> *Those are lexeme, stemming, stop-word, `tsvector`, `tsquery`, and `@@`.*
>
> Bạn còn lăn tăn chỗ nào.  
> *You may still feel unsure about something.*
>
> Hãy quay lại mục 1.9 và 1.10.  
> *Please go back to sections 1.9 and 1.10.*
>
> Không có gì phải ngại.  
> *There is nothing to be embarrassed about.*

### 2.1. Bên trong PostgreSQL: văn bản đi qua những chặng nào
*2.1. Inside PostgreSQL: which stages the text passes through*

Ở Phần 1 ta đã chạy tay bốn bước.  
*In Part 1 we hand-ran four steps.*

Bây giờ hãy xem PostgreSQL gọi tên các bước đó là gì.  
*Now let us see what PostgreSQL calls those steps.*

Tên gọi này sẽ xuất hiện trong tài liệu chính thức.  
*These names appear in the official documentation.*

Tên gọi này cũng xuất hiện trong phỏng vấn.  
*These names also appear in interviews.*

**Text search configuration** — cấu hình tìm kiếm văn bản — là một "bộ luật xử lý ngôn ngữ" có tên.  
*A text search configuration is a named set of language processing rules.*

Bạn viết `to_tsvector('english', ...)`.  
*You write `to_tsvector('english', ...)`.*

Chuỗi `'english'` chính là tên của một configuration.  
*The string `'english'` is the name of one configuration.*

Mỗi configuration gồm hai bộ phận.  
*Each configuration has two parts.*

**Bộ phận thứ nhất là Parser** — bộ phân tích cú pháp.  
*The first part is the parser.*

Nhiệm vụ của nó là cắt văn bản thô thành các token.  
*Its job is to cut raw text into tokens.*

Nó còn **gán loại cho từng token**.  
*It also assigns a type to each token.*

Nó phân biệt từ thường, số, email, URL và tên máy chủ.  
*It distinguishes plain words, numbers, emails, URLs, and host names.*

Việc gán loại này quan trọng hơn bạn tưởng.  
*This type assignment matters more than you think.*

Một parser ngây thơ cắt theo dấu chấm và dấu @.  
*A naive parser splits on periods and the @ sign.*

Khi đó địa chỉ `lan@company.com` sẽ vỡ thành ba mảnh vô nghĩa.  
*The address `lan@company.com` then shatters into three meaningless pieces.*

Parser của Postgres nhận ra đó là một email.  
*The Postgres parser recognizes it as an email.*

Nó giữ nguyên địa chỉ đó thành một token duy nhất.  
*It keeps that address as one single token.*

URL, số phiên bản và đường dẫn file cũng được đối xử như vậy.  
*URLs, version numbers, and file paths get the same treatment.*

**Bộ phận thứ hai là Dictionaries** — các bộ từ điển.  
*The second part is the dictionaries.*

Từng token sau khi được parser gán loại sẽ đi qua một dãy từ điển.  
*Each typed token then travels through a chain of dictionaries.*

Mỗi từ điển có quyền làm một trong ba việc.  
*Each dictionary may do one of three things.*

Nó có thể biến token thành lexeme.  
*It may turn the token into a lexeme.*

Nó có thể vứt bỏ token.  
*It may throw the token away.*

Nó có thể chuyển token cho từ điển kế tiếp xử lý.  
*It may pass the token to the next dictionary.*

Có ba loại từ điển bạn cần biết.  
*There are three dictionary types you need to know.*

- **Stop-word dictionary** chứa danh sách các từ cần vứt bỏ.  
  *A stop-word dictionary holds the list of words to discard.*

  Ví dụ là "the", "is" và "a".  
  *Examples are "the", "is", and "a".*

- **Stemmer** là bộ rút gốc từ.  
  *A stemmer is the component that reduces words to their stems.*

  Postgres dùng thuật toán **Snowball**.  
  *Postgres uses the Snowball algorithm.*

  Snowball là một họ thuật toán stemming do Martin Porter phát triển.  
  *Snowball is a family of stemming algorithms developed by Martin Porter.*

  Nó có sẵn luật cho hàng chục ngôn ngữ.  
  *It ships with rules for dozens of languages.*

- **Synonym / thesaurus dictionary** — từ điển đồng nghĩa — là thành phần tùy chọn.  
  *A synonym or thesaurus dictionary is optional.*

  Nó cho phép bạn tự khai báo "những từ này coi như một".  
  *It lets you declare that certain words count as one.*

  Ví dụ là khai báo "supernovae" và "supernova stars" đều quy về lexeme `sn`.  
  *One example maps both "supernovae" and "supernova stars" to the lexeme `sn`.*

**Chọn sai configuration là ra kết quả sai.**  
*Choosing the wrong configuration produces wrong results.*

Hãy nhớ ba lựa chọn hay dùng nhất.  
*Please remember the three most common choices.*

| Config / Config | Nó làm gì / What it does | Dùng khi / Use it when |
|---|---|---|
| `'english'` | Bỏ stop-word tiếng Anh và stem theo luật tiếng Anh / drops English stop-words and stems by English rules | Nội dung văn xuôi tiếng Anh / English prose content |
| `'simple'` | Chỉ chuyển thành chữ thường. **Không** stem, **không** bỏ stop-word / only lowercases; no stemming, no stop-word removal | Mã sản phẩm, username, tag, tên riêng — những thứ không được phép biến dạng / product codes, usernames, tags, proper names — things that must not be distorted |
| `'vietnamese'` | Postgres **không có sẵn** config này / Postgres does not ship this configuration | Xem đoạn dưới / see the paragraph below |

Ta nói riêng về **tiếng Việt**.  
*Let us talk about Vietnamese separately.*

PostgreSQL không có bộ stemmer tiếng Việt tích hợp sẵn.  
*PostgreSQL has no built-in Vietnamese stemmer.*

Thật ra tiếng Việt cũng không cần stemming theo kiểu tiếng Anh.  
*Vietnamese does not really need English-style stemming.*

Lý do là từ tiếng Việt không biến đổi đuôi.  
*The reason is that Vietnamese words do not change their endings.*

Vấn đề của tiếng Việt lại nằm ở chỗ khác.  
*The Vietnamese problem lies elsewhere.*

Vấn đề thứ nhất là **dấu thanh**.  
*The first problem is diacritics.*

Người dùng hay gõ không dấu.  
*Users often type without diacritics.*

Vấn đề thứ hai là **từ ghép**.  
*The second problem is compound words.*

Cụm "cơ sở dữ liệu" là ba token.  
*The phrase "cơ sở dữ liệu" is three tokens.*

Cụm đó lại chỉ là một khái niệm.  
*That phrase is nonetheless a single concept.*

Cách xử lý thực dụng thứ nhất là dùng config `'simple'` kết hợp extension **`unaccent`**.  
*The first practical approach uses the `'simple'` configuration with the `unaccent` extension.*

`unaccent` là một extension bỏ dấu tiếng Việt.  
*`unaccent` is an extension that strips Vietnamese diacritics.*

Nó biến "tiếng" thành "tieng".  
*It turns "tiếng" into "tieng".*

Cách xử lý thứ hai là chuyển hẳn sang vector search cho phần ngữ nghĩa.  
*The second approach moves the semantic part entirely to vector search.*

Ta sẽ nói kỹ ở Phần 3 và Phần 4.  
*We will cover this in depth in Parts 3 and 4.*

### 2.2. Họ hàm tạo `tsquery` — chọn đúng cái cho đúng tình huống
*2.2. The `tsquery` builder family — pick the right one for the situation*

Ở Phần 1 ta mới dùng `to_tsquery`.  
*In Part 1 we only used `to_tsquery`.*

Thực tế Postgres có bốn hàm để tạo `tsquery`.  
*In reality Postgres has four functions for building a `tsquery`.*

**Chọn sai hàm là nguyên nhân số một khiến ô tìm kiếm bị lỗi 500 khi lên production.**  
*Picking the wrong function is the number one cause of 500 errors in a production search box.*

| Hàm / Function | Đầu vào phải trông như thế nào / What the input must look like | Dùng khi nào / When to use it |
|---|---|---|
| `to_tsquery` | Bắt buộc có toán tử: `'run & fast'` / operators are mandatory: `'run & fast'` | Khi **bạn** viết truy vấn và cần kiểm soát chính xác. **Không** đưa input thô của người dùng vào / when you write the query yourself and need exact control; never feed raw user input in |
| `plainto_tsquery` | Văn bản thô, tự động nối các từ bằng AND: `'run fast'` → `'run' & 'fast'` / raw text, words joined automatically with AND | Input đơn giản, muốn "phải có tất cả các từ" / simple input where all words must be present |
| `phraseto_tsquery` | Văn bản thô thành cụm từ liền kề: `'full text search'` → `'full' <-> 'text' <-> 'search'` / raw text into an adjacent phrase | Tìm đúng **cụm từ**, đúng thứ tự / finding an exact phrase in exact order |
| `websearch_to_tsquery` | Cú pháp kiểu Google: `'postgres -mysql "full text"'` / Google-style syntax | **Input trực tiếp từ ô tìm kiếm của người dùng** — an toàn nhất, cần PostgreSQL 11 trở lên / direct input from a user search box — the safest option, needs PostgreSQL 11 or newer |

⚠️ **Chỗ khó — và đây là chỗ rất nhiều người vấp**  
*⚠️ A hard spot — and this is where very many people stumble*

`to_tsquery` yêu cầu đầu vào phải đúng cú pháp của nó.  
*`to_tsquery` demands input in its own exact syntax.*

Người dùng thật thì gõ đủ thứ.  
*Real users type all sorts of things.*

Họ gõ khoảng trắng thừa và dấu chấm.  
*They type extra spaces and periods.*

Họ gõ dấu ngoặc lệch.  
*They type unbalanced parentheses.*

Họ gõ ký tự `#` trong "c# tips".  
*They type the `#` character in "c# tips".*

Mỗi lần như vậy `to_tsquery` sẽ **báo lỗi cú pháp**.  
*Each time, `to_tsquery` raises a syntax error.*

Bạn không bắt lỗi đó.  
*Suppose you do not catch that error.*

Server sẽ trả về lỗi 500 cho người dùng.  
*The server then returns a 500 error to the user.*

💡 **Mẹo thực chiến**  
*💡 A practical tip*

Hãy dùng **`websearch_to_tsquery`** cho bất kỳ ô tìm kiếm nào hướng tới người dùng cuối.  
*Please use `websearch_to_tsquery` for any end-user search box.*

Nó nuốt được mọi thứ lộn xộn mà không báo lỗi.  
*It swallows any mess without raising an error.*

Nó hỗ trợ đúng những quy ước người dùng đã quen từ Google.  
*It supports exactly the conventions users already know from Google.*

Đặt trong dấu ngoặc kép là tìm nguyên cụm.  
*Quotation marks mean an exact phrase.*

Đặt dấu trừ phía trước là loại trừ.  
*A leading minus sign means exclusion.*

Viết chữ OR là hoặc.  
*The word OR means either one.*

```sql
-- Người dùng gõ vào ô search:  postgres -mysql "full text"
-- The user types into the search box:  postgres -mysql "full text"
SELECT websearch_to_tsquery('english', 'postgres -mysql "full text"');
-- Kết quả: 'postgr' & !'mysql' & 'full' <-> 'text'
-- Result:  'postgr' & !'mysql' & 'full' <-> 'text'
--   'postgr'        → phải có (postgres đã bị stem)
--   'postgr'        → must be present (postgres has been stemmed)
--   !'mysql'        → không được có mysql
--   !'mysql'        → mysql must not be present
--   'full' <-> 'text' → phải có "full" đứng ngay trước "text"
--   'full' <-> 'text' → "full" must stand directly before "text"
```

### 2.3. Đánh index cho FTS — GIN là lựa chọn chuẩn
*2.3. Indexing for FTS — GIN is the standard choice*

Ta nhắc lại một ý từ mục 1.13.  
*Let us recall one point from section 1.13.*

Bạn không có index.  
*Suppose you have no index.*

Mỗi lần tìm kiếm, Postgres phải tính lại `to_tsvector` cho **từng hàng** trong bảng.  
*On every search, Postgres must recompute `to_tsvector` for every row in the table.*

Với 10 nghìn hàng thì không sao.  
*With 10 thousand rows this is fine.*

Với 10 triệu hàng thì mỗi truy vấn mất hàng chục giây.  
*With 10 million rows each query takes tens of seconds.*

**GIN** là viết tắt của *Generalized Inverted Index* — chỉ mục ngược tổng quát.  
*GIN stands for Generalized Inverted Index.*

GIN chính là hiện thực cụ thể của phép ví von "mục lục cuối sách" ở mục 1.8.  
*GIN is the concrete realization of the book-index analogy from section 1.8.*

Nó lưu ánh xạ từ mỗi lexeme tới danh sách các hàng chứa lexeme đó.  
*It stores a mapping from each lexeme to the list of rows containing it.*

Đây là loại index tiêu chuẩn cho FTS.  
*This is the standard index type for FTS.*

Có hai cách dựng index.  
*There are two ways to build the index.*

Bạn nên biết cả hai.  
*You should know both.*

Mỗi cách hợp với một tình huống.  
*Each way suits a different situation.*

**Cách 1 — Functional index (index trên biểu thức)**  
*Approach 1 — a functional index (an index on an expression)*

**Functional index** nghĩa là bạn đánh index không phải trên giá trị gốc của cột.  
*A functional index means you do not index the column's raw value.*

Bạn đánh index trên **kết quả của một hàm áp lên cột đó**.  
*You index the result of a function applied to that column.*

Ở đây ta index trên kết quả của `to_tsvector('english', body)`.  
*Here we index the result of `to_tsvector('english', body)`.*

```sql
-- Dựng index GIN trên biểu thức to_tsvector('english', body)
-- Build a GIN index on the expression to_tsvector('english', body)
CREATE INDEX articles_fts_idx          -- articles_fts_idx là tên ta đặt cho index
                                       -- articles_fts_idx is the name we give the index
ON articles                             -- trên bảng articles
                                        -- on the articles table
USING GIN (to_tsvector('english', body));  -- loại index là GIN, trên biểu thức này
                                           -- the index type is GIN, over this expression

-- Khi truy vấn, biểu thức trong WHERE PHẢI GIỐNG HỆT biểu thức lúc tạo index
-- When querying, the expression in WHERE MUST MATCH the one used at index creation
SELECT title
FROM articles
WHERE to_tsvector('english', body)     -- ← giống hệt dòng CREATE INDEX ở trên
                                       -- ← identical to the CREATE INDEX line above
   @@ websearch_to_tsquery('english', 'postgres index');
```

Ưu điểm thứ nhất là bạn không cần thêm cột nào.  
*The first advantage is that you need no extra column.*

Ưu điểm thứ hai là bạn không tốn thêm chỗ lưu `tsvector`.  
*The second advantage is that you spend no extra storage on the `tsvector`.*

Ưu điểm thứ ba là bạn không phải lo đồng bộ.  
*The third advantage is that you have no synchronization to worry about.*

Nhược điểm thứ nhất là câu truy vấn phải viết lặp lại đúng biểu thức đó.  
*The first drawback is that the query must repeat that exact expression.*

Nhược điểm thứ hai xuất hiện khi bạn muốn gộp nhiều cột.  
*The second drawback appears when you want to combine several columns.*

Khi đó biểu thức viết khá dài dòng.  
*The expression then becomes rather verbose.*

**Cách 2 — Generated column (cột được sinh tự động) kèm GIN**  
*Approach 2 — a generated column plus GIN*

**Generated column** là một cột mà bạn **không tự điền giá trị**.  
*A generated column is one whose value you never fill in yourself.*

Postgres tự tính nó ra từ các cột khác.  
*Postgres computes it from other columns.*

Postgres **tự cập nhật lại mỗi khi các cột nguồn thay đổi**.  
*Postgres updates it automatically whenever the source columns change.*

Từ khóa `STORED` nghĩa là giá trị đó được lưu thật lên đĩa.  
*The `STORED` keyword means the value is really written to disk.*

Postgres không tính lại nó mỗi lần đọc.  
*Postgres does not recompute it on every read.*

Tính năng này có từ PostgreSQL 12.  
*This feature exists from PostgreSQL 12 onward.*

```sql
ALTER TABLE articles                        -- sửa bảng articles
                                            -- alter the articles table
ADD COLUMN search_vector tsvector           -- thêm cột search_vector kiểu tsvector
                                            -- add a search_vector column of type tsvector
GENERATED ALWAYS AS (                       -- giá trị luôn được Postgres tự sinh
                                            -- the value is always generated by Postgres
    to_tsvector(
        'english',
        coalesce(title, '') || ' ' || coalesce(body, '')  -- ghép title + body
                                                          -- concatenate title and body
    )
) STORED;                                   -- lưu thật lên đĩa, tự cập nhật khi title/body đổi
                                            -- stored on disk, refreshed when title or body changes

-- Dựng index GIN trên chính cột đó
-- Build a GIN index directly on that column
CREATE INDEX articles_sv_idx ON articles USING GIN (search_vector);

-- Truy vấn giờ ngắn gọn hẳn
-- The query is now much shorter
SELECT title
FROM articles
WHERE search_vector @@ websearch_to_tsquery('english', 'postgres index');
```

Ta giải thích hai ký hiệu vừa xuất hiện.  
*Let us explain the two symbols that just appeared.*

**`||`** trong PostgreSQL là toán tử **nối chuỗi**.  
*In PostgreSQL, `||` is the string concatenation operator.*

Nó ghép hai đoạn text lại với nhau.  
*It joins two pieces of text together.*

**`coalesce(a, b)`** là hàm trả về `a` khi `a` không rỗng.  
*The function `coalesce(a, b)` returns `a` when `a` is not empty.*

Giá trị `a` có thể là **NULL**.  
*The value `a` may be NULL.*

Khi đó hàm trả về `b`.  
*The function then returns `b`.*

**NULL** — rỗng, không có giá trị — là một giá trị đặc biệt trong SQL.  
*NULL is a special value in SQL.*

Nó nghĩa là "ô này không có dữ liệu".  
*It means "this cell holds no data".*

Nó khác chuỗi rỗng `''`.  
*It differs from the empty string `''`.*

Chuỗi rỗng nghĩa là "có dữ liệu, và dữ liệu đó là không có ký tự nào".  
*An empty string means "there is data, and that data has zero characters".*

NULL nghĩa là "hoàn toàn không biết gì".  
*NULL means "nothing is known at all".*

⚠️ **Chỗ khó — và là lỗi khiến người ta mất cả buổi chiều để debug**  
*⚠️ A hard spot — the bug that costs people a whole afternoon of debugging*

Trong SQL, **NULL nuốt tất cả**.  
*In SQL, NULL swallows everything.*

Bất cứ phép toán nào có NULL tham gia đều cho kết quả NULL.  
*Any operation involving NULL produces NULL.*

Một bài viết nào đó không có `title`.  
*Suppose some article has no `title`.*

Cột `title` của nó là NULL.  
*Its `title` column is NULL.*

Bạn quên bọc `coalesce`.  
*You forget to wrap it in `coalesce`.*

Khi đó `title || ' ' || body` ra NULL.  
*The expression `title || ' ' || body` then evaluates to NULL.*

Điều đó kéo theo `search_vector` của cả hàng đó thành NULL.  
*That turns the whole row's `search_vector` into NULL.*

**Hàng đó biến mất hoàn toàn khỏi mọi kết quả tìm kiếm.**  
*That row vanishes completely from every search result.*

Nội dung body của nó vẫn chứa đúng từ khóa.  
*Its body content still contains the right keyword.*

Không có thông báo lỗi nào cả.  
*There is no error message at all.*

Nó chỉ lặng lẽ mất tích.  
*It simply disappears in silence.*

Vì vậy bạn hãy **luôn bọc `coalesce(col, '')` khi ghép cột**.  
*So please always wrap columns in `coalesce(col, '')` when concatenating.*

🧩 **[Ngoài bài gốc]** Ngày xưa Postgres chưa có generated column.  
*🧩 [Beyond the source lesson] In the old days Postgres had no generated columns.*

Khi đó người ta duy trì cột `tsvector` bằng **trigger**.  
*Back then people maintained the `tsvector` column with a trigger.*

Trigger là một đoạn code tự động chạy mỗi khi có thao tác thêm, sửa hoặc xóa trên bảng.  
*A trigger is code that runs automatically on every insert, update, or delete.*

Cách này chạy được.  
*This approach does work.*

Cách này lại dễ hỏng.  
*This approach breaks easily.*

Bạn quên viết trigger cho một đường cập nhật nào đó.  
*You may forget a trigger for some update path.*

Bạn cũng có thể sửa dữ liệu bằng câu lệnh thủ công.  
*You may also change data with a manual statement.*

Cột `tsvector` sẽ lệch khỏi nội dung thật ngay lập tức.  
*The `tsvector` column then drifts from the real content immediately.*

Không ai biết chuyện đó.  
*Nobody notices.*

Từ PostgreSQL 12 trở đi, **generated column STORED** đã thay thế hoàn toàn nhu cầu này.  
*From PostgreSQL 12 onward, a STORED generated column fully replaces that need.*

Best practice năm 2026 gồm hai vế.  
*The 2026 best practice has two halves.*

Bạn dùng functional index khi chỉ tìm trên một cột.  
*You use a functional index when searching a single column.*

Bạn dùng generated column khi cần gộp nhiều cột và gán trọng số.  
*You use a generated column when combining several columns and assigning weights.*

**Bạn đừng viết trigger sync bằng tay nữa.**  
*Please stop hand-writing synchronization triggers.*

Ngoại lệ chỉ dành cho những lý do rất đặc biệt.  
*The exception applies only to very special reasons.*

### 2.4. Ranking — sắp xếp kết quả theo độ liên quan
*2.4. Ranking — sorting results by relevance*

Toán tử `@@` chỉ trả lời "có khớp hay không".  
*The `@@` operator only answers "does it match or not".*

Có 5.000 bài viết cùng khớp.  
*Suppose 5,000 articles all match.*

Bạn cần biết bài nào **liên quan nhất** để đưa lên đầu.  
*You need to know which article is most relevant so you can put it first.*

Việc đó gọi là **ranking** — xếp hạng.  
*That task is called ranking.*

Độ liên quan gọi là **relevance**.  
*The degree of relatedness is called relevance.*

```sql
SELECT title,
       ts_rank(search_vector, query) AS rank    -- tính điểm liên quan cho từng hàng
                                                -- compute a relevance score for each row
FROM articles,
     websearch_to_tsquery('english', 'postgres performance') AS query
       -- Dòng trên đặt tsquery vào FROM và đặt tên là query, để
       -- The line above puts the tsquery in FROM and names it query, so that
       -- Postgres chỉ phải tính nó MỘT LẦN thay vì tính lại cho mỗi hàng.
       -- Postgres computes it ONCE instead of recomputing it per row.
WHERE search_vector @@ query                    -- lọc trước: chỉ giữ hàng khớp
                                                -- filter first: keep only matching rows
ORDER BY rank DESC                              -- sắp xếp điểm cao xuống thấp
                                                -- sort from high score to low
LIMIT 10;                                       -- chỉ lấy 10 kết quả đầu
                                                -- take only the first 10 results
```

Có hai hàm tính điểm bạn cần phân biệt.  
*There are two scoring functions you need to tell apart.*

- **`ts_rank`** chấm điểm dựa trên **tần suất** xuất hiện của các lexeme khớp.  
  *`ts_rank` scores based on the frequency of the matching lexemes.*

  Từ khóa xuất hiện càng nhiều lần thì điểm càng cao.  
  *The more times a keyword appears, the higher the score.*

- **`ts_rank_cd`** có chữ "cd" viết tắt của **cover density** — mật độ bao phủ.  
  *In `ts_rank_cd` the letters "cd" stand for cover density.*

  Ngoài tần suất, nó còn **thưởng điểm khi các từ khóa nằm gần nhau** trong tài liệu.  
  *Beyond frequency, it rewards keywords that sit close together in the document.*

  Cách chấm này hợp lý khi người dùng tìm một cụm từ.  
  *This scoring makes sense when the user searches for a phrase.*

  Một bài có "machine learning" nằm liền nhau đáng lẽ phải hơn.  
  *An article with "machine learning" side by side should rank higher.*

  Bài kia có chữ "machine" ở đầu và chữ "learning" ở cuối.  
  *The other article has "machine" at the start and "learning" at the end.*

**`setweight` — quyết định trường nào quan trọng hơn**  
*`setweight` — deciding which field matters more*

Trong thực tế, một từ khóa xuất hiện ở **tiêu đề** rõ ràng có ý nghĩa hơn.  
*In practice a keyword in the title clearly carries more meaning.*

Từ khóa đó nằm lọt thỏm giữa thân bài thì ít ý nghĩa hơn.  
*The same keyword buried in the body means less.*

`setweight` cho phép bạn gán **trọng số** cho từng nguồn văn bản.  
*`setweight` lets you assign a weight to each text source.*

Bốn trọng số là A, B, C và D.  
*The four weights are A, B, C, and D.*

Trọng số A là quan trọng nhất.  
*Weight A is the most important.*

Trọng số D là ít quan trọng nhất.  
*Weight D is the least important.*

```sql
ALTER TABLE articles
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    -- Tiêu đề: trọng số A (cao nhất)
    -- Title: weight A (the highest)
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    -- Thân bài: trọng số B
    -- Body: weight B
    setweight(to_tsvector('english', coalesce(body, '')), 'B') ||
    -- Tag: trọng số C. array_to_string biến mảng tag thành một chuỗi
    -- Tags: weight C. array_to_string turns the tag array into a single string
    setweight(to_tsvector('english', coalesce(array_to_string(tags, ' '), '')), 'C')
) STORED;
```

Kết quả là một bài có từ khóa trong tiêu đề sẽ được `ts_rank` chấm điểm cao hơn hẳn.  
*As a result, `ts_rank` gives a much higher score to an article with the keyword in its title.*

Bài chỉ có từ khóa trong thân sẽ xếp thấp hơn.  
*An article with the keyword only in its body ranks lower.*

Điều đáng nói là bạn đạt được điều đó **hoàn toàn bằng cấu hình trong database**.  
*The notable thing is that you achieve this entirely through database configuration.*

Bạn không phải viết một dòng code xử lý relevance nào ở tầng ứng dụng.  
*You write no relevance-handling code at the application layer.*

### 2.5. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh
*2.5. 🧩 [Beyond the source lesson] — three classic mistakes and how to avoid them*

**Lỗi 1 — Biểu thức lệch làm index bị bỏ qua**  
*Mistake 1 — a mismatched expression leaves the index unused*

Bạn tạo index bằng `GIN (to_tsvector('english', body))`.  
*You create the index with `GIN (to_tsvector('english', body))`.*

Trong câu truy vấn bạn lại viết `to_tsvector(body)`.  
*In the query you instead write `to_tsvector(body)`.*

Cách viết đó thiếu mất tham số config.  
*That form is missing the configuration argument.*

Với Postgres, đây là **hai biểu thức khác nhau**.  
*To Postgres these are two different expressions.*

Index vì vậy bị bỏ qua.  
*The index therefore gets skipped.*

Truy vấn rơi về sequential scan.  
*The query falls back to a sequential scan.*

Điều tệ nhất là **không có lỗi nào được báo ra**.  
*The worst part is that no error gets raised.*

Truy vấn vẫn trả về kết quả đúng.  
*The query still returns correct results.*

Nó chỉ chậm.  
*It is merely slow.*

Bạn sẽ chỉ phát hiện khi người dùng phàn nàn.  
*You only find out when users complain.*

Cách kiểm tra là dùng **`EXPLAIN ANALYZE`**.  
*The way to check is `EXPLAIN ANALYZE`.*

Đó là một câu lệnh yêu cầu Postgres chạy thật rồi kể lại chi tiết nó đã làm gì.  
*That statement asks Postgres to really run and then report exactly what it did.*

```sql
EXPLAIN ANALYZE
SELECT title FROM articles
WHERE to_tsvector('english', body) @@ websearch_to_tsquery('english', 'postgres');
```

Trong kết quả trả về, bạn tìm dòng có chữ **`Bitmap Index Scan`**.  
*In the output, look for the line containing `Bitmap Index Scan`.*

Đó là dấu hiệu index đang được dùng.  
*That is the sign the index is in use.*

Bạn lại thấy dòng **`Seq Scan`**.  
*You may instead see the line `Seq Scan`.*

Khi đó index đang bị bỏ qua.  
*The index is then being skipped.*

**Quy tắc vàng**  
*The golden rule*

Biểu thức trong `WHERE` phải **trùng khít từng chữ** với biểu thức lúc `CREATE INDEX`.  
*The expression in `WHERE` must match the one in `CREATE INDEX` word for word.*

Hai bên phải cùng tên config.  
*Both sides must use the same configuration name.*

Hai bên phải cùng dạng hai tham số.  
*Both sides must use the same two-argument form.*

**Lỗi 2 — Quên `coalesce`, NULL nuốt cả `tsvector`**  
*Mistake 2 — forgetting `coalesce`, so NULL swallows the whole `tsvector`*

Lỗi này đã được mô tả kỹ ở mục 2.3.  
*This mistake was described in detail in section 2.3.*

Triệu chứng đặc trưng rất dễ nhận.  
*The characteristic symptom is easy to spot.*

Một bài viết rõ ràng chứa từ khóa.  
*An article clearly contains the keyword.*

Bài đó không bao giờ xuất hiện trong kết quả.  
*That article never appears in the results.*

Bạn gặp triệu chứng này.  
*Suppose you meet this symptom.*

Việc đầu tiên nên làm là chạy `SELECT id FROM articles WHERE search_vector IS NULL;`.  
*The first thing to do is run `SELECT id FROM articles WHERE search_vector IS NULL;`.*

Câu lệnh đó cho bạn biết có bao nhiêu hàng đang bị mất tích.  
*That statement tells you how many rows have gone missing.*

**Lỗi 3 — Nhét input thô của người dùng vào `to_tsquery`**  
*Mistake 3 — feeding raw user input into `to_tsquery`*

Lỗi này đã được mô tả ở mục 2.2.  
*This mistake was described in section 2.2.*

Triệu chứng thứ nhất là log lỗi đầy dòng `syntax error in tsquery`.  
*The first symptom is an error log full of `syntax error in tsquery`.*

Triệu chứng thứ hai là một tỉ lệ nhỏ và dai dẳng các request tìm kiếm trả về lỗi 500.  
*The second symptom is a small, persistent share of search requests returning 500 errors.*

Cách chữa là đổi sang `websearch_to_tsquery`.  
*The fix is to switch to `websearch_to_tsquery`.*

Bạn cũng có thể đổi sang `plainto_tsquery`.  
*You may also switch to `plainto_tsquery`.*

### ✅ Self-check Phần 2
*✅ Self-check for Part 2*

**1. Với ô tìm kiếm của người dùng, bạn nên dùng hàm tạo `tsquery` nào, và vì sao KHÔNG dùng `to_tsquery`?**  
*1. Which `tsquery` builder should you use for a user search box, and why NOT `to_tsquery`?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Bạn dùng `websearch_to_tsquery`.  
> *You use `websearch_to_tsquery`.*
>
> Bạn cũng có thể dùng `plainto_tsquery`.  
> *You may also use `plainto_tsquery`.*
>
> Bạn không dùng `to_tsquery`.  
> *You do not use `to_tsquery`.*
>
> Lý do là nó đòi hỏi cú pháp toán tử chặt chẽ.  
> *The reason is its strict operator syntax.*
>
> Input thật của người dùng có khoảng trắng, dấu câu và ký tự lạ.  
> *Real user input has spaces, punctuation, and odd characters.*
>
> Những thứ đó gây syntax error.  
> *Those things cause a syntax error.*
>
> Syntax error dẫn tới lỗi 500.  
> *A syntax error leads to a 500 error.*

**2. Bạn muốn từ khóa khớp ở `title` được ưu tiên hơn khớp ở `body`. Dùng cơ chế gì?**  
*2. You want a keyword match in `title` to outrank one in `body`. Which mechanism do you use?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Bạn dùng `setweight`.  
> *You use `setweight`.*
>
> Bạn gán trọng số 'A' cho `to_tsvector` của title.  
> *You assign weight 'A' to the `to_tsvector` of the title.*
>
> Bạn gán trọng số 'B' cho body.  
> *You assign weight 'B' to the body.*
>
> Bạn nối hai phần bằng `||`.  
> *You join the two parts with `||`.*
>
> Sau đó bạn xếp hạng bằng `ts_rank`.  
> *Then you rank with `ts_rank`.*

**3. Index là `GIN(to_tsvector('english', body))`.**  
*3. The index is `GIN(to_tsvector('english', body))`.*

**Truy vấn lại viết `to_tsvector(body)`.**  
*The query instead writes `to_tsvector(body)`.*

**Chuyện gì xảy ra?**  
*What happens?*

**Bạn phát hiện lỗi này bằng cách nào?**  
*How do you detect this mistake?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Hai biểu thức không khớp nhau.  
> *The two expressions do not match.*
>
> Index vì vậy bị bỏ qua.  
> *The index therefore gets skipped.*
>
> Truy vấn rơi về sequential scan.  
> *The query falls back to a sequential scan.*
>
> Kết quả vẫn đúng.  
> *The results are still correct.*
>
> Truy vấn chậm dần theo kích thước bảng.  
> *The query slows down as the table grows.*
>
> Không có cảnh báo nào cả.  
> *There is no warning at all.*
>
> Bạn phát hiện bằng `EXPLAIN ANALYZE`.  
> *You detect it with `EXPLAIN ANALYZE`.*
>
> Bạn thấy `Seq Scan` thay cho `Bitmap Index Scan`.  
> *You see `Seq Scan` instead of `Bitmap Index Scan`.*
>
> Đó đúng là dấu hiệu của lỗi này.  
> *That is indeed the signature of this mistake.*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED (In depth)*

> Phần này nhìn xuống **bên dưới bề mặt**.  
> *This part looks beneath the surface.*
>
> Nó nói về cấu trúc dữ liệu thật.  
> *It covers the real data structures.*
>
> Nó nói về độ phức tạp thuật toán.  
> *It covers algorithmic complexity.*
>
> Nó nói về những đánh đổi bạn buộc phải chọn.  
> *It covers the trade-offs you are forced to make.*
>
> Đây là vùng kiến thức phân biệt người biết dùng công cụ với người hiểu công cụ.  
> *This knowledge separates people who can use a tool from people who understand it.*

### 3.1. Ruột gan của inverted index — vì sao GIN nhanh
*3.1. The guts of the inverted index — why GIN is fast*

Ta đã nói GIN là "mục lục cuối sách".  
*We said GIN is the index at the back of a book.*

Giờ hãy mở nó ra xem bên trong có gì.  
*Now let us open it up and look inside.*

GIN lưu một danh sách các lexeme đã sắp xếp.  
*GIN stores a sorted list of lexemes.*

Với mỗi lexeme, nó lưu kèm một **posting list** — danh sách đăng.  
*For each lexeme it also stores a posting list.*

Posting list chính là danh sách các hàng chứa lexeme đó.  
*A posting list is the list of rows containing that lexeme.*

Cụ thể hơn, posting list không lưu cả hàng.  
*More precisely, a posting list does not store the whole row.*

Nó chỉ lưu **row identifier** — mã định danh hàng.  
*It only stores the row identifier.*

Trong Postgres, mã này gọi là **TID** — *Tuple ID*.  
*In Postgres this identifier is called a TID, short for Tuple ID.*

TID là một cặp số.  
*A TID is a pair of numbers.*

Nó cho biết hàng đó nằm ở trang nào trên đĩa.  
*It tells you which page on disk holds that row.*

Nó cũng cho biết hàng đó ở vị trí thứ mấy trong trang.  
*It also tells you the row's slot position within that page.*

Hãy hình dung cấu trúc đó trong đầu.  
*Picture that structure in your head.*

```
lexeme 'databas'  →  [ hàng #12, hàng #45, hàng #88, hàng #301, ... ]
                     [ row  #12, row  #45, row  #88, row  #301, ... ]
lexeme 'index'    →  [ hàng #7,  hàng #45, hàng #200, ... ]
                     [ row  #7,  row  #45, row  #200, ... ]
lexeme 'postgr'   →  [ hàng #45, hàng #88, ... ]
                     [ row  #45, row  #88, ... ]
```

Bạn tìm `'postgr' & 'index'`.  
*You search for `'postgr' & 'index'`.*

Điều kiện đó đòi hỏi cả hai từ.  
*That condition demands both words.*

Postgres lấy hai posting list rồi **giao** chúng lại với nhau.  
*Postgres takes the two posting lists and intersects them.*

Kết quả ra ngay `[hàng #45]`.  
*The result is immediately `[row #45]`.*

Nó **không hề đụng vào** những hàng còn lại của bảng.  
*It never touches the remaining rows of the table.*

**Bây giờ tới phần độ phức tạp.**  
*Now we come to complexity.*

Trước hết ta giải thích ký hiệu.  
*First let us explain the notation.*

**Big-O** đọc là "bíc ô", viết là `O(...)`.  
*Big-O is pronounced "bíc ô" and written `O(...)`.*

Nó là cách mô tả *thời gian chạy của thuật toán tăng lên như thế nào khi dữ liệu lớn dần*.  
*It describes how an algorithm's runtime grows as the data grows.*

Ký hiệu `O(N)` nghĩa là thời gian tỉ lệ thuận với số lượng dữ liệu `N`.  
*The notation `O(N)` means the time is proportional to the data size `N`.*

Bạn gấp đôi dữ liệu.  
*You double the data.*

Thời gian cũng gấp đôi.  
*The time also doubles.*

Ký hiệu `O(1)` nghĩa là thời gian không đổi.  
*The notation `O(1)` means the time stays constant.*

Dữ liệu lớn cỡ nào cũng vậy.  
*This holds no matter how large the data grows.*

Ta so sánh hai cách tìm kiếm.  
*Let us compare the two search approaches.*

- **`LIKE '%x%'`** phải đọc từng hàng.  
  *`LIKE '%x%'` must read every row.*

  Độ phức tạp là **`O(N)`**.  
  *The complexity is `O(N)`.*

  Ở đây `N` là **tổng số hàng trong bảng**.  
  *Here `N` is the total number of rows in the table.*

  Bảng 10 triệu hàng chậm gấp 10 lần bảng 1 triệu hàng.  
  *A 10-million-row table is 10 times slower than a 1-million-row table.*

- **FTS với GIN** chỉ đọc những hàng nằm trong posting list.  
  *FTS with GIN reads only the rows in the posting list.*

  Thời gian phụ thuộc **số hàng khớp**.  
  *The time depends on the number of matching rows.*

  Cộng thêm chi phí tra cứu lexeme trong index.  
  *Add the cost of looking up the lexeme in the index.*

  Chi phí đó rất nhỏ, ở dạng logarit.  
  *That cost is very small and logarithmic.*

  Chỉ 50 bài viết chứa từ khóa.  
  *Suppose only 50 articles contain the keyword.*

  Bảng có 1 triệu hay 50 triệu hàng cũng gần như không khác nhau.  
  *A 1-million-row table and a 50-million-row table behave almost identically.*

**Đây là toàn bộ bí mật vì sao FTS giữ được tốc độ khi dữ liệu phình to.**  
*This is the whole secret of how FTS keeps its speed as data grows.*

Nó cũng ngay lập tức chỉ ra điểm yếu.  
*It also immediately reveals the weakness.*

Từ khóa người dùng tìm lại là một từ cực phổ biến.  
*The user's keyword may be an extremely common word.*

Từ đó khớp với 2 triệu hàng.  
*That word matches 2 million rows.*

Posting list khi đó dài 2 triệu phần tử.  
*The posting list is then 2 million entries long.*

FTS cũng chậm trong tình huống này.  
*FTS is slow in this situation too.*

Ta sẽ quay lại điểm này ở Phần 4.  
*We will return to this point in Part 4.*

**Mặt trái của GIN — chuyện xây và cập nhật**  
*The downside of GIN — building and updating it*

Xây GIN tốn kém hơn xây B-tree.  
*Building a GIN index costs more than building a B-tree.*

Lý do là nó phải bóc từng tài liệu ra thành nhiều lexeme rồi gộp lại.  
*The reason is that it must split each document into many lexemes and merge them.*

Có hai điều đáng nhớ.  
*There are two things worth remembering.*

- **`maintenance_work_mem`** là một tham số cấu hình.  
  *`maintenance_work_mem` is a configuration parameter.*

  Nó quy định lượng RAM mà Postgres được phép dùng cho các thao tác bảo trì.  
  *It sets how much RAM Postgres may use for maintenance operations.*

  Xây index là một trong các thao tác đó.  
  *Building an index is one of those operations.*

  GIN **rất nhạy** với tham số này.  
  *GIN is very sensitive to this parameter.*

  Bạn tăng nó lên.  
  *You raise it.*

  Thời gian xây index có thể rút từ hàng giờ xuống hàng phút.  
  *The index build time can drop from hours to minutes.*

- **`fastupdate`** là một cơ chế của GIN.  
  *`fastupdate` is a GIN mechanism.*

  Nó không cập nhật index ngay mỗi lần ghi.  
  *It does not update the index immediately on every write.*

  Nó gom các thay đổi vào một vùng đệm rồi gộp vào sau.  
  *It collects changes in a buffer and merges them later.*

  Nhờ vậy việc ghi nhanh hơn.  
  *Writing therefore gets faster.*

  Đổi lại truy vấn phải quét thêm vùng đệm đó.  
  *In exchange, queries must also scan that buffer.*

  Vùng đệm phình to mà chưa được dọn.  
  *The buffer may swell without being cleaned up.*

  Khi đó truy vấn chậm dần.  
  *Queries then get slower and slower.*

### 3.2. GIN so với GiST — câu hỏi phỏng vấn kinh điển
*3.2. GIN versus GiST — the classic interview question*

**GiST** là viết tắt của *Generalized Search Tree* — cây tìm kiếm tổng quát.  
*GiST stands for Generalized Search Tree.*

GiST là loại index thứ hai mà Postgres hỗ trợ cho FTS.  
*GiST is the second index type Postgres supports for FTS.*

Khác biệt cốt lõi là GiST **lossy** — có mất mát thông tin.  
*The core difference is that GiST is lossy.*

"Lossy" ở đây nghĩa là GiST không lưu chính xác từng lexeme.  
*"Lossy" here means GiST does not store each lexeme exactly.*

Nó lưu một **signature**.  
*It stores a signature.*

Signature là một dạng dấu vân tay nén lại của tập lexeme.  
*A signature is a compressed fingerprint of the lexeme set.*

Dấu vân tay này nhỏ gọn.  
*This fingerprint is compact.*

Dấu vân tay này có thể trùng nhau.  
*These fingerprints can collide.*

GiST vì vậy có thể báo "hàng này có vẻ khớp".  
*GiST may therefore report that a row seems to match.*

Thật ra hàng đó không khớp.  
*In reality that row does not match.*

Trường hợp đó gọi là **false positive** — dương tính giả.  
*That case is called a false positive.*

GiST có false positive.  
*GiST has false positives.*

Vì vậy sau khi tra index, Postgres buộc phải mở **heap** ra kiểm tra lại từng ứng viên.  
*So after consulting the index, Postgres must open the heap and recheck each candidate.*

**Heap** trong Postgres là nơi lưu dữ liệu thật của bảng.  
*The heap in Postgres is where the table's real data lives.*

Heap phân biệt với index.  
*The heap is distinct from the index.*

Bước kiểm tra lại này gọi là **recheck**.  
*This re-verification step is called a recheck.*

Đó chính là lý do GiST truy vấn chậm hơn GIN.  
*That is exactly why GiST queries slower than GIN.*

| Tiêu chí / Criterion | **GIN** | **GiST** |
|---|---|---|
| Bản chất / Nature | Inverted index, lưu chính xác / an inverted index storing exact data | Signature tree, lossy nên phải recheck / a lossy signature tree needing a recheck |
| Tốc độ truy vấn / Query speed | **Nhanh hơn** cho FTS / faster for FTS | Chậm hơn / slower |
| Tốc độ xây và ghi thêm / Build and write speed | Chậm hơn, ngốn RAM / slower, RAM-hungry | **Nhanh hơn**, nhẹ hơn / faster, lighter |
| Chịu cập nhật thường xuyên / Tolerance for frequent updates | Kém hơn / worse | **Tốt hơn** / better |
| Kích thước index / Index size | To hơn / larger | **Nhỏ hơn**, điều chỉnh được qua `siglen` / smaller, tunable via `siglen` |
| Nhạy với `maintenance_work_mem` / Sensitive to `maintenance_work_mem` | Có / yes | Không / no |
| Nên chọn khi / Choose it when | **Đọc nhiều, văn bản ít thay đổi** — tức đa số trường hợp / read-heavy with rarely changing text — most cases | Ghi rất nhiều, hoặc bị giới hạn dung lượng đĩa / write-heavy, or constrained on disk space |

**Chốt để nhớ**  
*The takeaway to remember*

Bạn mặc định chọn **GIN**.  
*You choose GIN by default.*

Bạn chỉ nghiêng về GiST trong hai trường hợp.  
*You lean toward GiST in only two cases.*

Trường hợp thứ nhất là bảng ghi rất nhiều so với đọc.  
*The first case is a table with far more writes than reads.*

Trường hợp thứ hai là dung lượng index trở thành ràng buộc thật sự.  
*The second case is when index size becomes a real constraint.*

### 3.3. Trade-off là gì, và ba cách duy trì `tsvector`
*3.3. What a trade-off is, and three ways to maintain a `tsvector`*

Trước khi so sánh, ta cần định nghĩa một từ.  
*Before comparing, we need to define one word.*

Từ này sẽ xuất hiện dày đặc từ đây tới hết giáo trình.  
*This word appears densely from here to the end of the course.*

**Trade-off** — sự đánh đổi — là tình huống bạn không thể có tất cả cùng lúc.  
*A trade-off is a situation where you cannot have everything at once.*

Bạn muốn được điều này.  
*You want one thing.*

Bạn phải chấp nhận mất điều kia.  
*You must accept losing another.*

Nhanh hơn thì tốn bộ nhớ hơn.  
*Faster costs more memory.*

Đơn giản hơn thì kém linh hoạt hơn.  
*Simpler means less flexible.*

Trong kỹ thuật, gần như **không có lựa chọn nào tốt tuyệt đối**.  
*In engineering there is almost never an absolutely best choice.*

Chỉ có lựa chọn phù hợp với ràng buộc cụ thể của bạn.  
*There is only the choice that fits your specific constraints.*

Khả năng gọi tên trade-off một cách rành mạch rất quan trọng.  
*The ability to name trade-offs clearly matters a lot.*

Đó chính là thứ người phỏng vấn tìm kiếm ở ứng viên senior trở lên.  
*That is exactly what interviewers look for in senior candidates and above.*

Ta áp dụng khái niệm này vào ba cách duy trì `tsvector`.  
*Let us apply this concept to the three ways of maintaining a `tsvector`.*

| Cách làm / Approach | Được gì / What you gain | Mất gì / What you lose |
|---|---|---|
| **Functional GIN index** `GIN(to_tsvector('english', body))` | Không nhân đôi dữ liệu, không thêm cột, không lo đồng bộ / no duplicated data, no extra column, no synchronization worries | Truy vấn phải lặp lại đúng biểu thức; gộp nhiều cột thì viết rườm rà / the query must repeat the exact expression, and combining columns gets verbose |
| **Generated column STORED** kèm GIN / **Generated column STORED** with GIN | Truy vấn gọn (`search_vector @@ q`), Postgres tự đồng bộ, gộp nhiều cột và gán `setweight` dễ dàng / compact queries, automatic synchronization, easy multi-column merging and `setweight` | Tốn thêm dung lượng đĩa để lưu `tsvector` / extra disk space to store the `tsvector` |
| **Trigger tự viết** / **Hand-written trigger** | Linh hoạt tối đa, làm được logic phức tạp tùy ý / maximum flexibility, any complex logic you like | Nhiều code, dễ quên đường cập nhật, lỗi xảy ra âm thầm — **nên né** / lots of code, easy to miss an update path, silent failures — best avoided |

Quy tắc thực chiến gồm ba vế.  
*The practical rule has three parts.*

Bạn cần gán trọng số cho nhiều trường.  
*You may need weights across several fields.*

Khi đó bạn dùng **generated column**.  
*You then use a generated column.*

Bạn chỉ tìm trên một trường và muốn tiết kiệm đĩa.  
*You may search a single field and want to save disk space.*

Khi đó bạn dùng **functional index**.  
*You then use a functional index.*

Bạn gần như không bao giờ cần trigger tay nữa.  
*You almost never need a hand-written trigger anymore.*

### 3.4. Tinh chỉnh relevance sâu hơn
*3.4. Tuning relevance more deeply*

**`ts_rank(weights, vector, query, normalization)`** là dạng đầy đủ với bốn tham số.  
*`ts_rank(weights, vector, query, normalization)` is the full four-argument form.*

- **`weights`** là mảng bốn số quyết định điểm cho từng nhãn trọng số.  
  *`weights` is an array of four numbers setting the score for each weight label.*

  Mặc định là `{0.1, 0.2, 0.4, 1.0}`.  
  *The default is `{0.1, 0.2, 0.4, 1.0}`.*

  Bốn số này tương ứng với D, C, B và A.  
  *These four numbers correspond to D, C, B, and A.*

  Một từ khớp ở nhãn A được tính điểm gấp 10 lần so với khớp ở nhãn D.  
  *A word matching at label A scores 10 times a match at label D.*

- **`normalization`** — chuẩn hóa — là một **bitmask**.  
  *`normalization` is a bitmask.*

  Bitmask là một số nguyên mà mỗi bit bật hoặc tắt một tùy chọn.  
  *A bitmask is an integer where each bit switches one option on or off.*

  Nó quyết định có chia điểm theo độ dài tài liệu hay không.  
  *It decides whether the score gets divided by document length.*

Vì sao ta cần chuẩn hóa theo độ dài?  
*Why do we need length normalization?*

Ta không chuẩn hóa.  
*Suppose we do not normalize.*

Một bài viết 10.000 chữ gần như luôn thắng một bài 300 chữ.  
*A 10,000-word article almost always beats a 300-word one.*

Lý do đơn giản là nó dài nên chứa nhiều từ khóa hơn.  
*The simple reason is that its length lets it hold more keywords.*

Lý do không phải là nó liên quan hơn.  
*The reason is not that it is more relevant.*

Cờ chuẩn hóa chia điểm cho độ dài để cân bằng lại.  
*The normalization flag divides the score by length to rebalance.*

Ví dụ là cờ `32`.  
*One example is flag `32`.*

Cờ này áp dụng công thức `rank / (rank + 1)`.  
*This flag applies the formula `rank / (rank + 1)`.*

Nó đưa điểm về khoảng từ 0 tới 1.  
*It maps the score into the range from 0 to 1.*

Nhờ vậy bạn dễ so sánh giữa các truy vấn.  
*This makes comparison across queries easier.*

⚠️ **Cảnh báo hiệu năng quan trọng**  
*⚠️ An important performance warning*

Để tính `ts_rank`, Postgres phải **mở `tsvector` của từng tài liệu khớp** ra và đọc.  
*To compute `ts_rank`, Postgres must open and read the `tsvector` of every matching document.*

Truy vấn khớp một triệu hàng.  
*Suppose the query matches one million rows.*

Postgres phải đọc một triệu `tsvector`.  
*Postgres must read one million `tsvector` values.*

Công việc này nặng về **I/O**.  
*This work is I/O heavy.*

I/O là viết tắt của *Input/Output*.  
*I/O stands for Input/Output.*

Đó là thao tác đọc và ghi dữ liệu từ đĩa.  
*It refers to reading and writing data on disk.*

Đây thường là chặng chậm nhất trong máy tính.  
*This is usually the slowest stage in a computer.*

Có ba cách giảm nhẹ.  
*There are three ways to ease it.*

(a) Bạn **lọc cứng trước khi xếp hạng**.  
*(a) You filter hard before ranking.*

Bạn thêm điều kiện về danh mục hoặc khoảng thời gian.  
*You add a category condition or a time range.*

Cách này thu hẹp tập ứng viên.  
*This narrows the candidate set.*

(b) Bạn **lưu sẵn `tsvector`** trong generated column.  
*(b) You pre-store the `tsvector` in a generated column.*

Nhờ vậy Postgres khỏi tính lại.  
*Postgres then avoids recomputing it.*

(c) Bạn chỉ xếp hạng **top-N**.  
*(c) You rank only the top N.*

Bạn không xếp hạng toàn bộ tập khớp.  
*You do not rank the entire matching set.*

### 3.5. Tìm theo tiền tố, theo cụm, và chống lỗi chính tả
*3.5. Prefix search, phrase search, and typo tolerance*

**Prefix matching** — khớp theo tiền tố — cho phép người dùng gõ nửa từ.  
*Prefix matching lets a user type half a word.*

Bạn dùng ký hiệu `:*`.  
*You use the `:*` notation.*

```sql
SELECT to_tsquery('english', 'postgr:*');
-- Khớp mọi lexeme bắt đầu bằng "postgr": postgres, postgresql, postgraduate...
-- Matches every lexeme starting with "postgr": postgres, postgresql, postgraduate...
-- Đây là cơ chế đằng sau tính năng gợi ý khi người dùng đang gõ dở.
-- This is the mechanism behind type-ahead suggestions.
```

**Phrase / proximity search** — tìm theo cụm hoặc theo khoảng cách.  
*Phrase and proximity search work by adjacency or distance.*

Bạn dùng `<->` để yêu cầu từ này đứng ngay trước từ kia.  
*You use `<->` to demand that one word stand directly before another.*

Bạn dùng `<N>` để yêu cầu hai từ cách nhau đúng N từ.  
*You use `<N>` to demand that two words sit exactly N words apart.*

```sql
SELECT to_tsvector('english', 'data science is fun')
    @@ phraseto_tsquery('english', 'data science');
-- phraseto_tsquery tự sinh ra 'data' <-> 'science'
-- phraseto_tsquery automatically produces 'data' <-> 'science'
-- → t (true), vì trong câu gốc "data" đứng ngay trước "science"
-- → t (true), because "data" stands directly before "science" in the source sentence

SELECT to_tsvector('english', 'science of data')
    @@ phraseto_tsquery('english', 'data science');
-- → f (false), vì thứ tự bị đảo. Đây chính là lúc thông tin VỊ TRÍ
-- → f (false), because the order is reversed. This is where the POSITION data
--   mà tsvector lưu ở mục 1.10 phát huy tác dụng.
--   stored by tsvector in section 1.10 proves its worth.
```

**Fuzzy search — chống lỗi chính tả bằng `pg_trgm`**  
*Fuzzy search — typo tolerance with `pg_trgm`*

Đây là một giới hạn quan trọng cần thuộc lòng.  
*Here is an important limitation to memorize.*

**FTS không sửa lỗi chính tả.**  
*FTS does not fix spelling mistakes.*

Người dùng gõ "postgrsql" và thiếu mất chữ e.  
*A user types "postgrsql" and drops the letter e.*

Stemmer sẽ tạo ra một lexeme khác hẳn `postgresql`.  
*The stemmer then produces a lexeme completely different from `postgresql`.*

Kết quả trả về là rỗng.  
*The result comes back empty.*

FTS chuẩn hóa **biến thể ngữ pháp**.  
*FTS normalizes grammatical variants.*

FTS không đoán **ý định** của người gõ sai.  
*FTS does not guess the intent behind a typo.*

Giải pháp là extension **`pg_trgm`**.  
*The solution is the `pg_trgm` extension.*

Extension này dựa trên khái niệm **trigram** — bộ ba ký tự.  
*This extension builds on the concept of a trigram.*

Trigram nghĩa là cắt một chuỗi thành mọi đoạn ba ký tự liên tiếp.  
*A trigram means cutting a string into every run of three consecutive characters.*

Ví dụ là chuỗi "postgres".  
*One example is the string "postgres".*

Chuỗi này cho ra `pos`, `ost`, `stg`, `tgr`, `gre` và `res`.  
*It produces `pos`, `ost`, `stg`, `tgr`, `gre`, and `res`.*

Hai chuỗi càng chia sẻ nhiều trigram chung thì càng giống nhau.  
*The more trigrams two strings share, the more alike they are.*

Một lỗi gõ nhỏ chỉ làm hỏng vài trigram.  
*A small typo only damages a few trigrams.*

Phần lớn trigram vẫn trùng nhau.  
*Most trigrams still overlap.*

```sql
-- Cài extension (chỉ cần chạy một lần cho mỗi database)
-- Install the extension (run once per database)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Dựng index GIN theo kiểu trigram trên cột title
-- Build a trigram-style GIN index on the title column
CREATE INDEX articles_title_trgm
ON articles
USING GIN (title gin_trgm_ops);   -- gin_trgm_ops báo cho GIN dùng luật trigram
                                  -- gin_trgm_ops tells GIN to use trigram rules

-- Tìm gần đúng
-- Approximate search
SELECT title,
       similarity(title, 'postgrsql') AS sim   -- similarity trả điểm giống nhau từ 0 tới 1
                                               -- similarity returns a likeness score from 0 to 1
FROM articles
WHERE title % 'postgrsql'      -- toán tử % nghĩa là "đủ giống theo ngưỡng trigram"
                               -- the % operator means "similar enough by the trigram threshold"
ORDER BY sim DESC;             -- giống nhất lên đầu
                               -- most similar first
```

🧩 **[Ngoài bài gốc]** Kiến trúc tìm kiếm ở production hiện đại thường xếp **ba tầng bổ trợ nhau**.  
*🧩 [Beyond the source lesson] Modern production search architecture usually stacks three complementary layers.*

**FTS** bắt đúng từ.  
*FTS catches the exact word.*

Nó chính xác, nhanh và giải thích được.  
*It is precise, fast, and explainable.*

**Trigram** cứu lỗi gõ.  
*Trigrams rescue typos.*

**Vector search** hiểu nghĩa gần.  
*Vector search understands near meanings.*

Ba tầng này không thay thế nhau.  
*These three layers do not replace one another.*

Mỗi tầng bắt một loại truy vấn mà hai tầng kia bỏ lọt.  
*Each layer catches a query type the other two miss.*

Phần 4 sẽ ghép chúng lại thành một kiến trúc hoàn chỉnh.  
*Part 4 will assemble them into one complete architecture.*

### 3.6. Edge case — các trường hợp biên phải thủ sẵn
*3.6. Edge cases — the boundary situations to prepare for*

**Edge case** — trường hợp biên — là những tình huống hiếm gặp.  
*An edge case is a rare situation.*

Chúng nằm ở rìa của những gì bạn dự tính.  
*They sit at the edge of what you planned for.*

Chúng xảy ra thì làm hệ thống hỏng hoặc cho kết quả sai.  
*When they happen, they break the system or produce wrong results.*

Người mới thường chỉ test "đường đi đẹp".  
*Beginners usually test only the happy path.*

Người có kinh nghiệm test các rìa.  
*Experienced people test the edges.*

Trong phỏng vấn, chủ động nêu edge case luôn ghi điểm.  
*In interviews, raising edge cases proactively always scores points.*

**NULL nuốt `tsvector`**  
*NULL swallows the `tsvector`*

Ta đã nói chuyện này ở mục 2.3.  
*We covered this in section 2.3.*

Ta nhắc lại vì đây là edge case gây đau đớn nhất trong FTS.  
*We repeat it because this is the most painful edge case in FTS.*

Bạn hãy luôn dùng `coalesce`.  
*Please always use `coalesce`.*

**Ngôn ngữ không phải tiếng Anh**  
*Languages other than English*

Bạn áp stemmer `'english'` lên tiếng Việt, tiếng Nhật hoặc tiếng Trung.  
*You may apply the `'english'` stemmer to Vietnamese, Japanese, or Chinese.*

Kết quả sẽ vô nghĩa.  
*The result will be meaningless.*

Với tiếng Việt, bạn cân nhắc `'simple'` kết hợp `unaccent`.  
*For Vietnamese, consider `'simple'` combined with `unaccent`.*

Cách này xử lý việc người dùng gõ không dấu.  
*This handles users typing without diacritics.*

Cách khác là chuyển sang vector search cho phần ngữ nghĩa.  
*Another approach moves the semantic part to vector search.*

Một bảng của bạn chứa nhiều ngôn ngữ.  
*Suppose one of your tables holds several languages.*

Hãy lưu tên config vào một cột riêng.  
*Please store the configuration name in a dedicated column.*

Sau đó bạn index theo biểu thức `GIN(to_tsvector(lang_col, body))`.  
*Then index on the expression `GIN(to_tsvector(lang_col, body))`.*

Cách này cho phép mỗi hàng dùng bộ luật ngôn ngữ của riêng nó.  
*This lets each row use its own language rule set.*

**Stop-word làm mất cả cụm từ**  
*Stop-words can erase an entire phrase*

Ban nhạc "The Who" là một ví dụ nổi tiếng.  
*The band "The Who" is a famous example.*

Cả hai từ đều là stop-word.  
*Both words are stop-words.*

Biểu thức `to_tsvector('english', 'The Who')` cho ra kết quả rỗng.  
*The expression `to_tsvector('english', 'The Who')` produces an empty result.*

Không cách nào tìm thấy ban nhạc này bằng config `'english'`.  
*There is no way to find this band with the `'english'` configuration.*

Với các trường chứa tên riêng, bạn dùng config `'simple'`.  
*For fields holding proper names, use the `'simple'` configuration.*

**Xây index làm khóa ghi**  
*Building an index locks writes*

Câu lệnh `CREATE INDEX` thông thường **khóa** bảng.  
*A plain `CREATE INDEX` statement locks the table.*

Nó chặn mọi thao tác ghi cho tới khi xây xong.  
*It blocks every write until the build finishes.*

Với bảng lớn, quá trình đó có thể kéo dài hàng giờ.  
*On a large table that process can take hours.*

Trên hệ thống đang chạy thật, bạn luôn dùng **`CREATE INDEX CONCURRENTLY`**.  
*On a live system, always use `CREATE INDEX CONCURRENTLY`.*

Nó xây chậm hơn.  
*It builds more slowly.*

Nó không chặn ghi.  
*It does not block writes.*

Tương tự, bạn có khi cần xây lại index.  
*Similarly, you may need to rebuild an index.*

Ví dụ là sau khi đổi config.  
*One example is after changing the configuration.*

Khi đó bạn dùng **`REINDEX INDEX CONCURRENTLY`**.  
*You then use `REINDEX INDEX CONCURRENTLY`.*

Lệnh này có từ PostgreSQL 12.  
*This command exists from PostgreSQL 12 onward.*

**Index bị bỏ qua âm thầm**  
*The index gets skipped silently*

Ta nhắc lại lần cuối vì tầm quan trọng của nó.  
*We repeat this one last time because of its importance.*

Vế trái của `@@` không phải đúng biểu thức đã index.  
*The left side of `@@` may not be the exact indexed expression.*

Khi đó Postgres lặng lẽ chuyển sang seq scan.  
*Postgres then quietly switches to a seq scan.*

Sau mỗi lần thay đổi câu truy vấn tìm kiếm, hãy chạy `EXPLAIN ANALYZE`.  
*After every change to a search query, please run `EXPLAIN ANALYZE`.*

Bạn xác nhận vẫn thấy `Bitmap Index Scan`.  
*You confirm that `Bitmap Index Scan` is still there.*

### 3.7. Bảng so sánh cốt lõi: LIKE/RegEx vs FTS vs Vector search
*3.7. The core comparison table: LIKE/RegEx vs FTS vs vector search*

Đây là bảng đáng chụp màn hình lưu lại.  
*This is a table worth screenshotting.*

Nó tóm tắt toàn bộ bức tranh tìm kiếm hiện đại.  
*It summarizes the entire modern search landscape.*

| | **LIKE / RegEx** | **FTS (`tsvector`)** | **Vector search (`pgvector`)** |
|---|---|---|---|
| Khớp theo / Matches on | Mẫu ký tự / character patterns | **Lexeme** — từ đã chuẩn hóa / lexemes, the normalized words | **Ngữ nghĩa** — embedding / meaning, via embeddings |
| "run" có tìm ra "running"? / Does "run" find "running"? | Không / no | **Có**, nhờ stemming / yes, thanks to stemming | Có, khi nghĩa đủ gần / yes, when the meanings are close enough |
| "áo phao" có tìm ra "đồ giữ ấm"? / Does "áo phao" find "đồ giữ ấm"? | Không / no | **Không**, khác lexeme hoàn toàn / no, the lexemes differ entirely | **Có**, vì nó hiểu nghĩa / yes, it understands meaning |
| Chống lỗi chính tả / Typo tolerance | Không / no | Không, cần `pg_trgm` / no, you need `pg_trgm` | Một phần / partial |
| Loại index / Index type | Kém, vì `%...%` thành seq scan / poor, since `%...%` forces a seq scan | **GIN** — inverted / GIN, an inverted index | **HNSW / IVFFlat** — tìm gần đúng / HNSW or IVFFlat, approximate search |
| Độ chính xác / Accuracy | Tuyệt đối / exact | Tuyệt đối theo lexeme / exact at the lexeme level | **Gần đúng** — approximate / approximate |
| Cần mô hình AI? / Needs an AI model? | Không / no | Không / no | **Có**, cần mô hình sinh embedding / yes, an embedding model is required |
| Giải thích được kết quả? / Are results explainable? | Có / yes | **Có** — "khớp vì chứa đúng từ" / yes — "it matched because it holds the word" | Khó / hard |

Ta giải thích hai từ mới trong bảng.  
*Let us explain two new terms from the table.*

**Embedding** nghĩa đen là *sự nhúng vào*.  
*Embedding literally means an act of embedding something.*

Embedding là việc biến một đoạn văn bản thành một dãy số dài.  
*An embedding turns a piece of text into a long sequence of numbers.*

Ví dụ là một dãy 768 số.  
*One example is a sequence of 768 numbers.*

Hai đoạn văn có nghĩa gần nhau sẽ cho hai dãy số gần nhau.  
*Two texts with close meanings produce two close number sequences.*

Chúng gần nhau trong một không gian toán học.  
*They sit close together in a mathematical space.*

**ANN** là viết tắt của *Approximate Nearest Neighbor* — láng giềng gần nhất gần đúng.  
*ANN stands for Approximate Nearest Neighbor.*

ANN là họ thuật toán tìm các dãy số gần nhất một cách nhanh.  
*ANN is the family of algorithms that finds the nearest number sequences quickly.*

Chúng chấp nhận sai sót nhỏ.  
*They accept small errors.*

**HNSW** và **IVFFlat** là hai thuật toán ANN mà `pgvector` cung cấp.  
*HNSW and IVFFlat are the two ANN algorithms `pgvector` provides.*

**Bài học rút ra**  
*The lesson to take away*

FTS mạnh khi người dùng gõ **đúng từ khóa**.  
*FTS is strong when the user types the right keyword.*

Người dùng có thể gõ ở bất kỳ biến thể nào.  
*The user may type any variant of it.*

FTS **không** hiểu từ đồng nghĩa.  
*FTS does not understand synonyms.*

Ngoại lệ là khi bạn tự cấu hình thesaurus.  
*The exception comes when you configure a thesaurus yourself.*

Chỗ trống đó chính là đất diễn của vector search.  
*That gap is exactly where vector search shines.*

Đó là lý do hybrid search tồn tại.  
*That is why hybrid search exists.*

### ✅ Self-check Phần 3
*✅ Self-check for Part 3*

**1. Bảng của bạn rất lớn.**  
*1. Your table is very large.*

**Vì sao GIN vẫn giữ được tốc độ?**  
*Why does GIN still keep its speed?*

**Vì sao `LIKE '%x%'` lại không giữ được?**  
*Why does `LIKE '%x%'` fail to keep its speed?*

**Độ phức tạp của mỗi bên là bao nhiêu?**  
*What is the complexity of each one?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> GIN là inverted index.  
> *GIN is an inverted index.*
>
> Nó tra thẳng posting list của lexeme.  
> *It looks up the lexeme's posting list directly.*
>
> Sau đó nó chỉ đọc các hàng khớp.  
> *Then it reads only the matching rows.*
>
> Thời gian phụ thuộc **số hàng khớp**.  
> *The time depends on the number of matching rows.*
>
> Thời gian không phụ thuộc tổng số hàng.  
> *The time does not depend on the total row count.*
>
> `LIKE '%x%'` không dùng được index.  
> *`LIKE '%x%'` cannot use an index.*
>
> Nó rơi về seq scan.  
> *It falls back to a seq scan.*
>
> Độ phức tạp là `O(N)` theo tổng số hàng.  
> *The complexity is `O(N)` over the total row count.*

**2. Bảng ghi rất nhiều, dung lượng index cần nhỏ, cần hỗ trợ tìm cụm — chọn GIN hay GiST?**  
*2. A write-heavy table, a small index budget, and phrase search support — do you choose GIN or GiST?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Bạn chọn GiST.  
> *You choose GiST.*
>
> Nó xây và cập nhật nhanh hơn.  
> *It builds and updates faster.*
>
> Index của nó nhỏ hơn.  
> *Its index is smaller.*
>
> Đổi lại truy vấn chậm hơn.  
> *In exchange the queries are slower.*
>
> Lý do là nó lossy nên phải recheck.  
> *The cause is its lossy nature, which forces a recheck.*
>
> Đây đúng là một trade-off.  
> *This really is a trade-off.*
>
> Bạn nêu rõ cái mất.  
> *You state the loss clearly.*
>
> Đó mới là câu trả lời hoàn chỉnh.  
> *Only that is a complete answer.*

**3. Người dùng hay gõ sai tên sản phẩm. FTS thuần giải quyết được không? Cần bổ sung gì?**  
*3. Users often misspell product names. Can plain FTS solve that? What do you need to add?*

> *Gợi ý đáp án*  
> *Suggested answer*
>
> Câu trả lời là không.  
> *The answer is no.*
>
> FTS chuẩn hóa biến thể ngữ pháp.  
> *FTS normalizes grammatical variants.*
>
> FTS không sửa lỗi gõ.  
> *FTS does not fix typos.*
>
> Bạn bổ sung extension `pg_trgm`.  
> *You add the `pg_trgm` extension.*
>
> Bạn dựng index với `gin_trgm_ops`.  
> *You build the index with `gin_trgm_ops`.*
>
> Bạn dùng toán tử `%` và hàm `similarity` để khớp gần đúng.  
> *You use the `%` operator and the `similarity` function for fuzzy matching.*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL (Systems thinking and technical leadership)*

> Ở ba phần trước, câu hỏi luôn là "làm thế nào".  
> *In the previous three parts, the question was always "how".*
>
> Ở phần này, câu hỏi đổi thành **"khi nào nên, khi nào không nên, và cái gì sẽ gãy trước"**.  
> *In this part the question becomes "when to, when not to, and what breaks first".*
>
> Đó là khác biệt cốt lõi giữa senior và staff.  
> *That is the core difference between senior and staff.*

### 4.1. Scale: từ vài nghìn tới vài chục triệu tài liệu
*4.1. Scale: from a few thousand to tens of millions of documents*

**Scale** — quy mô, mở rộng quy mô — nói về việc hệ thống còn chạy tốt hay không.  
*Scale is about whether the system still runs well.*

Câu hỏi đặt ra khi lượng dữ liệu hoặc lượng người dùng tăng lên nhiều lần.  
*The question arises when data volume or user count multiplies.*

Có hai kiểu mở rộng.  
*There are two kinds of scaling.*

Kiểu thứ nhất là **scale up / vertical scaling** — mở rộng dọc.  
*The first kind is scale up, or vertical scaling.*

Bạn mua máy mạnh hơn với nhiều RAM hơn và CPU nhanh hơn.  
*You buy a stronger machine with more RAM and a faster CPU.*

Kiểu thứ hai là **scale out / horizontal scaling** — mở rộng ngang.  
*The second kind is scale out, or horizontal scaling.*

Bạn thêm nhiều máy chạy song song.  
*You add more machines running in parallel.*

**Bottleneck** — điểm nghẽn — nghĩa đen là cổ chai.  
*Bottleneck literally means the neck of a bottle.*

Bottleneck là bộ phận chậm nhất trong hệ thống.  
*A bottleneck is the slowest component in a system.*

Nó quyết định tốc độ của cả dây chuyền.  
*It determines the speed of the whole chain.*

Cứ hình dung một chai nước.  
*Just picture a water bottle.*

Thân chai có thể to đến bất kỳ mức nào.  
*The body of the bottle can be as wide as you like.*

Nước chảy ra nhanh hay chậm là do cái cổ chai quyết định.  
*The neck alone decides how fast the water pours out.*

Tối ưu bất cứ chỗ nào **không phải** bottleneck đều là công cốc.  
*Optimizing anything that is not the bottleneck is wasted effort.*

**Câu hỏi ở tầm staff không phải "dùng GIN hay GiST".**  
*The staff-level question is not "GIN or GiST".*

Câu hỏi là "chỗ nào sẽ gãy khi hệ thống lớn lên".  
*The question is "what breaks as the system grows".*

Với FTS trong Postgres, có ba bottleneck chính.  
*For FTS in Postgres there are three main bottlenecks.*

Ta xếp chúng theo thứ tự thường gặp.  
*We list them in order of how often they appear.*

**Ta nói về điểm mạnh cấu trúc trước đã.**  
*Let us cover the structural strength first.*

GIN là inverted index.  
*GIN is an inverted index.*

Vì vậy thời gian truy vấn phụ thuộc số hàng khớp.  
*So query time depends on the number of matching rows.*

Thời gian không phụ thuộc tổng số hàng.  
*The time does not depend on the total row count.*

Điều đó có nghĩa là Postgres FTS chạy dưới một giây trên hàng triệu tài liệu.  
*This means Postgres FTS runs in under a second across millions of documents.*

Điều kiện là **truy vấn đủ chọn lọc**.  
*The condition is that the query be selective enough.*

Đây là lý do Postgres FTS thật sự thay thế được Elasticsearch.  
*This is why Postgres FTS genuinely replaces Elasticsearch.*

Điều đó đúng với phần lớn ứng dụng quy mô vừa và nhỏ.  
*That holds for most small and medium applications.*

**Bottleneck 1 — I/O của khâu xếp hạng**  
*Bottleneck 1 — I/O in the ranking stage*

Ta đã cảnh báo điều này ở mục 3.4.  
*We warned about this in section 3.4.*

Hàm `ts_rank` phải mở `tsvector` của **từng tài liệu khớp**.  
*The `ts_rank` function must open the `tsvector` of every matching document.*

Người dùng tìm một từ phổ biến khớp một triệu hàng.  
*A user searches a common word matching one million rows.*

Bạn phải đọc một triệu `tsvector` từ đĩa.  
*You must read one million `tsvector` values from disk.*

Đây thường là bottleneck xuất hiện **sớm nhất**.  
*This is usually the earliest bottleneck to appear.*

Nó cũng gây bất ngờ nhất.  
*It is also the most surprising one.*

Hệ thống chạy êm với các truy vấn hiếm.  
*The system runs smoothly on rare queries.*

Nó đột nhiên treo khi có người tìm một từ phổ thông.  
*It suddenly hangs when someone searches a common word.*

> **Cách chữa**  
> *The fix*
>
> Bạn lọc cứng bằng metadata **trước** khi xếp hạng.  
> *You filter hard on metadata before ranking.*
>
> Metadata đó gồm danh mục, khoảng thời gian và tenant.  
> *That metadata includes category, time range, and tenant.*
>
> Cách này thu hẹp tập ứng viên.  
> *This narrows the candidate set.*
>
> Bạn lưu sẵn `tsvector` trong generated column.  
> *You pre-store the `tsvector` in a generated column.*
>
> Bạn chỉ xếp hạng top-N.  
> *You rank only the top N.*

**Bottleneck 2 — write amplification**  
*Bottleneck 2 — write amplification*

**Write amplification** — khuếch đại ghi — là một hiện tượng cụ thể.  
*Write amplification is a specific phenomenon.*

Một thao tác ghi của bạn kéo theo nhiều thao tác ghi thật xuống đĩa.  
*One write of yours drags many real writes down to disk.*

Ở đây chuyện xảy ra như sau.  
*Here is how it plays out.*

Bạn sửa một bài viết.  
*You edit one article.*

Postgres không chỉ ghi lại hàng đó.  
*Postgres does not merely rewrite that row.*

Postgres còn phải cập nhật GIN index cho **mọi lexeme** trong bài.  
*Postgres must also update the GIN index for every lexeme in the article.*

Số mục cần cập nhật có thể lên tới hàng trăm.  
*The number of entries to update can reach into the hundreds.*

> **Cách chữa**  
> *The fix*
>
> Bạn bật `fastupdate` để gom thay đổi lại.  
> *You turn on `fastupdate` to batch the changes.*
>
> Bạn cân nhắc GiST khi ghi áp đảo đọc.  
> *You consider GiST when writes overwhelm reads.*
>
> Bạn tách hẳn nội dung tìm kiếm sang một bảng riêng.  
> *You split the searchable content into a separate table.*
>
> Nhờ vậy bảng chính không bị ảnh hưởng.  
> *The main table then stays unaffected.*

**Bottleneck 3 — thời gian xây index**  
*Bottleneck 3 — index build time*

GIN xây chậm.  
*GIN builds slowly.*

Câu lệnh `CREATE INDEX` thường khóa ghi.  
*The `CREATE INDEX` statement usually locks writes.*

> **Cách chữa**  
> *The fix*
>
> Bạn dùng `CREATE INDEX CONCURRENTLY`.  
> *You use `CREATE INDEX CONCURRENTLY`.*
>
> Bạn tăng `maintenance_work_mem` trước khi xây.  
> *You raise `maintenance_work_mem` before building.*

**Mở rộng ngang cho FTS**  
*Horizontal scaling for FTS*

Có hai kỹ thuật chính.  
*There are two main techniques.*

**Read replica** — bản sao chỉ đọc — là một bản sao của database.  
*A read replica is a copy of the database.*

Nó tự đồng bộ từ máy chính.  
*It synchronizes itself from the primary machine.*

Nó chỉ phục vụ đọc.  
*It serves reads only.*

Tìm kiếm là thao tác đọc thuần túy.  
*Search is a pure read operation.*

Vì vậy bạn đẩy toàn bộ lưu lượng search sang replica.  
*So you push all search traffic to the replica.*

Đây là cách giảm tải cực kỳ hiệu quả cho máy chính.  
*This is an extremely effective way to offload the primary.*

Máy chính vốn đã phải lo việc ghi.  
*The primary already has writes to handle.*

**Partitioning** — phân mảnh — là chia một bảng khổng lồ thành nhiều bảng con.  
*Partitioning splits one huge table into many child tables.*

Bạn chia theo một tiêu chí.  
*You split by one criterion.*

Tiêu chí thường gặp là theo thời gian, mỗi tháng một mảnh.  
*A common criterion is time, with one partition per month.*

Tiêu chí khác là theo **tenant**, mỗi khách hàng doanh nghiệp một mảnh.  
*Another criterion is tenant, with one partition per business customer.*

Mỗi mảnh có GIN index riêng.  
*Each partition has its own GIN index.*

Index của mỗi mảnh nhỏ hơn nhiều.  
*Each partition's index is far smaller.*

Truy vấn của bạn có kèm điều kiện lọc theo tiêu chí đó.  
*Your query carries a filter on that criterion.*

Khi đó bộ lập kế hoạch truy vấn sẽ **prune** — cắt tỉa.  
*The query planner then prunes.*

Nó bỏ qua hẳn những mảnh không liên quan.  
*It skips the irrelevant partitions entirely.*

Nó không thèm đụng tới chúng.  
*It never touches them at all.*

### 4.2. Quyết định kiến trúc: PostgreSQL FTS hay Elasticsearch?
*4.2. An architectural decision: PostgreSQL FTS or Elasticsearch?*

**Elasticsearch** là một công cụ tìm kiếm chuyên dụng.  
*Elasticsearch is a dedicated search engine.*

Bản mã nguồn mở tương đương là **OpenSearch**.  
*The equivalent open-source version is OpenSearch.*

Nó chạy như một hệ thống riêng biệt bên cạnh database của bạn.  
*It runs as a separate system alongside your database.*

Đây là lựa chọn mặc định của rất nhiều đội.  
*This is the default choice of many teams.*

Câu hỏi ở tầm staff là "có thật sự cần không".  
*The staff-level question is "do we really need it".*

| Tiêu chí / Criterion | **PostgreSQL FTS** | **Elasticsearch / OpenSearch** |
|---|---|---|
| Hạ tầng phải thêm / Extra infrastructure | **Không thêm gì** — dùng chính database đang có / nothing extra — it uses the database you already have | Một cụm máy riêng chạy trên JVM, cần người vận hành / a separate cluster on the JVM, needing operators |
| Đồng bộ dữ liệu / Data synchronization | **Không cần** — dữ liệu vốn đã ở đó / none needed — the data is already there | Phải xây pipeline đồng bộ, và pipeline này sẽ hỏng / you must build a sync pipeline, and it will break |
| Tính nhất quán / Consistency | **ACID, thấy ngay lập tức** / ACID, visible immediately | Nhất quán sau cùng — dữ liệu mới có độ trễ trước khi tìm được / eventually consistent, with a delay before new data is searchable |
| Nối với dữ liệu nghiệp vụ / Joining with business data | **Một câu SQL JOIN** / a single SQL JOIN | Hai hệ thống, phải viết code ghép ở tầng ứng dụng / two systems, joined by application code |
| Relevance nâng cao (BM25, analyzer phong phú) / Advanced relevance | Cơ bản (`ts_rank`) / basic, via `ts_rank` | **Mạnh hơn nhiều** / far stronger |
| Quy mô rất lớn, phân tích văn bản / Very large scale and text analytics | Khá / decent | **Vượt trội** / outstanding |
| Chi phí và độ phức tạp vận hành / Operational cost and complexity | **Thấp** / low | Cao / high |

Ta giải thích các từ trong bảng.  
*Let us explain the terms in the table.*

**JVM** là máy ảo Java.  
*The JVM is the Java Virtual Machine.*

Đó là nền tảng mà Elasticsearch chạy trên đó.  
*It is the platform Elasticsearch runs on.*

Bạn cần biết cách chỉnh bộ nhớ riêng cho nó.  
*You need to know how to tune its memory separately.*

**ACID** là viết tắt của bốn tính chất mà một giao dịch database phải đảm bảo.  
*ACID abbreviates four properties a database transaction must guarantee.*

*Atomicity* nghĩa là nguyên tử.  
*Atomicity means atomicity.*

Giao dịch làm hết hoặc không làm gì.  
*A transaction does everything or nothing.*

*Consistency* nghĩa là nhất quán.  
*Consistency means consistency.*

*Isolation* nghĩa là cô lập.  
*Isolation means isolation.*

Các giao dịch không giẫm chân nhau.  
*Transactions do not step on each other.*

*Durability* nghĩa là bền vững.  
*Durability means durability.*

Dữ liệu đã ghi thì không mất kể cả khi mất điện.  
*Written data survives even a power failure.*

**Eventually consistent** — nhất quán sau cùng — có nghĩa cụ thể.  
*Eventually consistent has a specific meaning.*

Dữ liệu mới ghi sẽ **cuối cùng** xuất hiện ở mọi nơi.  
*Newly written data eventually appears everywhere.*

Quá trình đó có độ trễ.  
*That process has a delay.*

Với search, điều này dẫn tới một hệ quả khó chịu.  
*For search this leads to an unpleasant consequence.*

Người dùng vừa đăng bài xong rồi tìm lại chưa thấy.  
*A user posts an article and cannot find it right after.*

Bạn phải thiết kế để né trải nghiệm đó.  
*You must design around that experience.*

**ETL** là viết tắt của *Extract - Transform - Load*.  
*ETL stands for Extract, Transform, Load.*

Đó là quy trình rút dữ liệu ra khỏi hệ này.  
*It is the process of extracting data from one system.*

Sau đó bạn biến đổi dữ liệu.  
*Then you transform the data.*

Cuối cùng bạn nạp nó vào hệ kia.  
*Finally you load it into another system.*

Đây chính là "pipeline đồng bộ" đã nhắc ở bảng trên.  
*This is exactly the sync pipeline mentioned in the table above.*

Bạn buộc phải xây và bảo trì nó khi chọn Elasticsearch.  
*You must build and maintain it once you choose Elasticsearch.*

**BM25** là một công thức tính điểm liên quan tiên tiến hơn `ts_rank`.  
*BM25 is a relevance scoring formula more advanced than `ts_rank`.*

Nó đang là tiêu chuẩn công nghiệp cho tìm kiếm theo từ khóa.  
*It is the industry standard for keyword search.*

Postgres lõi chưa có BM25.  
*Core Postgres does not have BM25 yet.*

Đã có extension mang nó vào.  
*An extension already brings it in.*

Bạn hãy xem phần ghi chú cuối.  
*Please see the closing notes.*

**On-call** — trực sự cố — là việc kỹ sư phải sẵn sàng bị gọi dậy giữa đêm.  
*On-call means an engineer must be ready to be woken at night.*

Kỹ sư bị gọi khi hệ thống hỏng.  
*The engineer gets called when the system breaks.*

Mỗi hệ thống bạn thêm vào kiến trúc là thêm một thứ có thể đánh thức ai đó lúc 3 giờ sáng.  
*Every system you add is one more thing that can wake someone at 3 a.m.*

Đây là chi phí thật.  
*This is a real cost.*

Ở tầm staff bạn phải tính nó vào quyết định.  
*At staff level you must factor it into the decision.*

> **Câu chốt tầm staff**  
> *The staff-level closing line*
>
> *"Với ứng dụng quy mô vừa và nhỏ, Postgres FTS xóa bỏ nhu cầu chạy thêm một cụm Elasticsearch."*  
> *"For small and medium applications, Postgres FTS removes the need for a separate Elasticsearch cluster."*
>
> *"Bạn dùng cùng một database."*  
> *"You use the same database."*
>
> *"Bạn có ACID."*  
> *"You get ACID."*
>
> *"Bạn join thẳng bằng SQL."*  
> *"You join directly with SQL."*
>
> *"Bạn không phải đồng bộ."*  
> *"You have nothing to synchronize."*
>
> *"Bạn không thêm hệ thống nào để trực đêm."*  
> *"You add no system to be on call for."*
>
> Bạn chọn Elasticsearch khi tìm kiếm **chính là sản phẩm** của bạn.  
> *You choose Elasticsearch when search is your product.*
>
> Bạn cũng chọn nó khi cần relevance và analyzer cao cấp.  
> *You also choose it when you need advanced relevance and analyzers.*
>
> Bạn cũng chọn nó khi quy mô thật sự rất lớn.  
> *You also choose it when the scale is genuinely very large.*

### 4.3. 🧩 [Ngoài bài gốc — ĐIỂM VÀNG] Hybrid Search: FTS kết hợp pgvector
*4.3. 🧩 [Beyond the source lesson — THE GOLDEN POINT] Hybrid search: FTS combined with pgvector*

Đây là chỗ toàn bộ khóa học hội tụ.  
*This is where the whole course converges.*

Đây cũng là nội dung dễ ghi điểm nhất trong phỏng vấn năm 2026.  
*It is also the easiest content to score points with in a 2026 interview.*

**Điểm mạnh và điểm mù của mỗi bên**  
*The strength and the blind spot of each side*

**FTS** tìm theo từ khóa, tức theo lexeme.  
*FTS searches by keyword, meaning by lexeme.*

Nó cực chính xác với thuật ngữ chuyên ngành và mã sản phẩm.  
*It is extremely precise with technical terms and product codes.*

Nó cũng cực chính xác với tên riêng và số phiên bản.  
*It is equally precise with proper names and version numbers.*

Nó nhanh và rẻ.  
*It is fast and cheap.*

Nó **giải thích được**.  
*It is explainable.*

Bạn luôn chỉ ra được "kết quả này lên đầu vì nó chứa đúng những từ này".  
*You can always say "this result ranks first because it holds exactly these words".*

Nó lại **mù ngữ nghĩa**.  
*It is nonetheless blind to meaning.*

Cụm "xe hơi" và "ô tô" là hai lexeme khác nhau.  
*The phrases "xe hơi" and "ô tô" are two different lexemes.*

Câu chuyện dừng ở đó.  
*The story ends there.*

**Vector search** tìm theo ngữ nghĩa.  
*Vector search searches by meaning.*

Nó hiểu được từ đồng nghĩa.  
*It understands synonyms.*

Nó hiểu được các cách diễn đạt khác nhau.  
*It understands different phrasings.*

Nó hay trượt ở đúng chỗ FTS mạnh.  
*It tends to miss exactly where FTS is strong.*

Một mã sản phẩm hiếm như "SKU-88XZ" gần như không có ý nghĩa ngữ nghĩa nào.  
*A rare product code like "SKU-88XZ" carries almost no semantic meaning.*

Embedding của nó vì vậy vô dụng.  
*Its embedding is therefore useless.*

Kết quả của nó khó giải thích cho người dùng.  
*Its results are hard to explain to users.*

Kết quả đó cũng khó giải thích cho chính đội kỹ thuật.  
*Those results are also hard to explain to the engineering team.*

Nhìn vào hai đoạn trên, kết luận là hiển nhiên.  
*Looking at the two paragraphs above, the conclusion is obvious.*

**Hai kỹ thuật này không cạnh tranh nhau.**  
*These two techniques do not compete.*

**Chúng bù khuyết cho nhau.**  
*They complement each other.*

Bạn chạy cả hai rồi hợp nhất kết quả.  
*You run both and then merge the results.*

Đó chính là **hybrid search**.  
*That is exactly hybrid search.*

**Cách hợp nhất phổ biến nhất là RRF.**  
*The most common merging method is RRF.*

**RRF** là viết tắt của *Reciprocal Rank Fusion* — hợp nhất theo nghịch đảo thứ hạng.  
*RRF stands for Reciprocal Rank Fusion.*

Nó hoạt động theo một ý tưởng đơn giản đến bất ngờ.  
*It works on a surprisingly simple idea.*

**Bạn đừng cộng điểm.**  
*Do not add up the scores.*

**Hãy cộng thứ hạng.**  
*Add up the ranks instead.*

Vì sao ta không cộng điểm?  
*Why do we not add the scores?*

Điểm của `ts_rank` và điểm khoảng cách vector nằm trên hai thang đo hoàn toàn khác nhau.  
*The `ts_rank` score and the vector distance score live on completely different scales.*

Không có cách nào so sánh trực tiếp cho công bằng.  
*There is no fair way to compare them directly.*

**Thứ hạng** thì luôn so sánh được.  
*Ranks, however, are always comparable.*

Đứng thứ nhất ở nhánh nào cũng là đứng thứ nhất.  
*First place in either branch is still first place.*

Công thức RRF rất gọn.  
*The RRF formula is very compact.*

Mỗi tài liệu nhận điểm `1 / (k + thứ_hạng)` từ mỗi nhánh.  
*Each document receives `1 / (k + rank)` from each branch.*

Sau đó bạn cộng các điểm đó lại.  
*Then you add those scores together.*

Hằng số `k` thường được chọn là 60.  
*The constant `k` is usually set to 60.*

Nó có tác dụng làm mượt.  
*It has a smoothing effect.*

Nó tránh việc vị trí số 1 áp đảo quá mức so với vị trí số 2.  
*It stops rank 1 from overwhelming rank 2.*

Tài liệu nào lọt vào top của **cả hai** nhánh sẽ có tổng điểm cao nhất.  
*A document reaching the top of both branches scores highest overall.*

Kết quả đó đúng như trực giác.  
*That result matches intuition.*

```sql
-- Ý tưởng: lấy top 50 từ nhánh keyword và top 50 từ nhánh semantic,
-- The idea: take the top 50 from the keyword branch and the top 50 from the semantic branch,
-- rồi hợp nhất thứ hạng bằng RRF.
-- then merge the ranks with RRF.

WITH kw AS (                                  -- WITH ... AS: đặt tên cho một truy vấn con
                                              -- WITH ... AS: give a name to a subquery
  -- ===== NHÁNH 1: KEYWORD (FTS) =====
  -- ===== BRANCH 1: KEYWORD (FTS) =====
  SELECT id,
         row_number() OVER (ORDER BY ts_rank(search_vector, q) DESC) AS r
         -- row_number() gán số thứ tự 1, 2, 3... theo điểm ts_rank giảm dần
         -- row_number() assigns 1, 2, 3... by descending ts_rank score
         -- → đây chính là "thứ hạng" của tài liệu ở nhánh keyword
         -- → this is the document's rank in the keyword branch
  FROM docs, websearch_to_tsquery('english', 'wireless headphones') q
  WHERE search_vector @@ q                    -- chỉ lấy tài liệu thật sự khớp từ khóa
                                              -- keep only documents that truly match the keywords
  LIMIT 50
),
sem AS (
  -- ===== NHÁNH 2: SEMANTIC (pgvector) =====
  -- ===== BRANCH 2: SEMANTIC (pgvector) =====
  SELECT id,
         row_number() OVER (ORDER BY embedding <=> :query_vec) AS r
         -- <=> là toán tử tính khoảng cách cosine giữa hai vector;
         -- <=> is the operator computing cosine distance between two vectors;
         -- càng NHỎ càng gần nghĩa, nên sắp tăng dần
         -- SMALLER means closer in meaning, so we sort ascending
         -- :query_vec là embedding của câu người dùng tìm, do ứng dụng truyền vào
         -- :query_vec is the embedding of the user query, passed in by the application
  FROM docs
  ORDER BY embedding <=> :query_vec
  LIMIT 50
)
-- ===== HỢP NHẤT BẰNG RRF =====
-- ===== MERGE WITH RRF =====
SELECT id,
       SUM(1.0 / (60 + r)) AS rrf_score       -- 60 là hằng số làm mượt của RRF
                                              -- 60 is the RRF smoothing constant
FROM (SELECT id, r FROM kw
      UNION ALL                                -- gộp hai danh sách lại, giữ cả trùng lặp
                                               -- merge both lists, keeping duplicates
      SELECT id, r FROM sem) t
GROUP BY id                                    -- gom theo tài liệu
                                               -- group by document
ORDER BY rrf_score DESC                        -- điểm hợp nhất cao nhất lên đầu
                                               -- highest fused score first
LIMIT 10;
```

**Vì sao staff bắt buộc phải biết điều này**  
*Why staff engineers must know this*

Hybrid search là **kiến trúc truy hồi mặc định** cho tìm kiếm hiện đại.  
*Hybrid search is the default retrieval architecture for modern search.*

Nó cũng là kiến trúc mặc định cho **RAG**.  
*It is also the default architecture for RAG.*

RAG là viết tắt của *Retrieval-Augmented Generation*.  
*RAG stands for Retrieval-Augmented Generation.*

Đó là kỹ thuật cho mô hình ngôn ngữ tra cứu tài liệu thật trước khi trả lời.  
*It is the technique of letting a language model look up real documents before answering.*

Mục đích là giảm bịa đặt.  
*The purpose is to reduce fabrication.*

Và đây là điểm mấu chốt.  
*And here is the crucial point.*

**`pgvector` cộng FTS làm được toàn bộ việc này bên trong một cái Postgres duy nhất.**  
*`pgvector` plus FTS does all of this inside a single Postgres.*

Bạn không cần chạy song song một cơ sở dữ liệu vector chuyên dụng.  
*You do not need a dedicated vector database running in parallel.*

Bạn cũng không cần một cụm Elasticsearch.  
*You also do not need an Elasticsearch cluster.*

Hãy để ý một điều.  
*Please notice one thing.*

Đây chính là lúc luận điểm "PostgreSQL mở rộng được" ở mục 1.5 trả cổ tức.  
*This is where the "PostgreSQL is extensible" argument from section 1.5 pays its dividend.*

Ở Phần 1, đó là một quyết định tưởng như hàn lâm.  
*In Part 1 it looked like an academic decision.*

Nó hóa ra quyết định toàn bộ hình dạng kiến trúc ở Phần 4.  
*It turns out to shape the entire architecture in Part 4.*

Nói được mạch liên kết này trong phỏng vấn là dấu hiệu rõ ràng của tư duy hệ thống.  
*Articulating this connection in an interview is a clear sign of systems thinking.*

### 4.4. Chi phí, vận hành, giám sát, và các kiểu hỏng
*4.4. Cost, operations, monitoring, and failure modes*

**Chi phí**  
*Cost*

FTS gần như **miễn phí về hạ tầng**.  
*FTS is nearly free in terms of infrastructure.*

Lý do là nó tận dụng Postgres bạn đã trả tiền.  
*The reason is that it reuses the Postgres you already pay for.*

Ta so sánh với việc vận hành một cụm Elasticsearch riêng.  
*Compare that with running a separate Elasticsearch cluster.*

Khoản tiết kiệm không chỉ là tiền máy chủ.  
*The saving is not only server money.*

Khoản tiết kiệm còn là **thời gian con người**.  
*The saving is also human time.*

Ít hệ thống hơn nghĩa là ít tài liệu vận hành hơn.  
*Fewer systems means less operational documentation.*

Ít hệ thống hơn cũng nghĩa là ít quy trình triển khai hơn.  
*Fewer systems also means fewer deployment procedures.*

Ít hệ thống hơn cũng nghĩa là ít bề mặt trực sự cố hơn.  
*Fewer systems also means less on-call surface.*

**Giám sát (monitoring)**  
*Monitoring*

Đây là những chỉ số bạn phải theo dõi.  
*These are the metrics you must track.*

- **p95 và p99 độ trễ truy vấn tìm kiếm**  
  *p95 and p99 search query latency*

  **p95** nghĩa là "95% số truy vấn nhanh hơn con số này".  
  *p95 means "95% of queries are faster than this number".*

  Người ta dùng p95 và p99 thay vì trung bình.  
  *People use p95 and p99 instead of the average.*

  Lý do là trung bình che giấu những trường hợp tệ nhất.  
  *The reason is that averages hide the worst cases.*

  Chính những trường hợp tệ nhất mới là thứ người dùng nhớ và phàn nàn.  
  *Those worst cases are exactly what users remember and complain about.*

- **Tỉ lệ truy vấn rơi về sequential scan**  
  *The share of queries falling back to a sequential scan*

  Đó là dấu hiệu index bị bỏ qua.  
  *That is the sign of an index being skipped.*

- **Kích thước GIN index**  
  *GIN index size*

  Nó phình theo thời gian và ăn đĩa.  
  *It swells over time and eats disk.*

- **Độ trễ ghi**  
  *Write latency*

  Chỉ số này giúp phát hiện write amplification do GIN.  
  *This metric helps detect write amplification caused by GIN.*

Ta dùng hai công cụ chính.  
*We use two main tools.*

**`pg_stat_statements`** là một extension thống kê mọi câu lệnh đã chạy.  
*`pg_stat_statements` is an extension that collects statistics on every statement executed.*

Nó tổng hợp thời gian và số lần gọi.  
*It aggregates timings and call counts.*

Công cụ thứ hai là `EXPLAIN ANALYZE` cho từng truy vấn nghi ngờ.  
*The second tool is `EXPLAIN ANALYZE` for each suspicious query.*

**Failure modes** — các kiểu hỏng.  
*Failure modes.*

Liệt kê trước những cách hệ thống có thể hỏng là một thói quen rất đặc trưng của staff engineer.  
*Listing the ways a system can break is a hallmark habit of a staff engineer.*

1. Biểu thức hoặc config bị lệch.  
*1. The expression or the configuration drifts.*

Index vì vậy bị bỏ qua.  
*The index therefore gets skipped.*

Độ trễ tăng dần một cách âm thầm.  
*Latency creeps up silently.*

Không có báo lỗi nào.  
*There is no error message.*

2. NULL nuốt `tsvector`.  
*2. NULL swallows the `tsvector`.*

Một số tài liệu biến mất khỏi kết quả tìm kiếm.  
*Some documents vanish from the search results.*

Không ai biết chuyện đó.  
*Nobody notices.*

Đây là kiểu hỏng nguy hiểm nhất.  
*This is the most dangerous failure mode.*

Lý do là hệ thống trông vẫn hoàn toàn bình thường.  
*The reason is that the system still looks completely normal.*

3. Truy vấn dùng một từ siêu phổ biến.  
*3. A query uses an extremely common word.*

Truy vấn đó khớp hàng triệu hàng.  
*That query matches millions of rows.*

Khâu xếp hạng ngốn I/O.  
*The ranking stage devours I/O.*

Cả database bị treo.  
*The whole database hangs.*

4. Bạn đổi text search configuration hoặc đổi ngôn ngữ.  
*4. You change the text search configuration or the language.*

Toàn bộ index cũ trở nên vô nghĩa.  
*The entire old index becomes meaningless.*

Bạn phải xây lại từ đầu.  
*You must rebuild it from scratch.*

5. Vùng đệm `fastupdate` của GIN phình to.  
*5. The GIN `fastupdate` buffer swells.*

Chuyện này xảy ra khi bạn ghi nhiều mà chưa **vacuum**.  
*This happens on heavy writes without a vacuum.*

Vacuum là thao tác dọn dẹp định kỳ của Postgres.  
*Vacuum is Postgres's periodic cleanup operation.*

Truy vấn khi đó chậm dần.  
*Queries then get slower and slower.*

### 4.5. Ảnh hưởng tới tổ chức và cách nói chuyện với người không rành kỹ thuật
*4.5. Organizational impact and how to talk to non-technical people*

**Stakeholder** — bên liên quan — là những người có lợi ích gắn với dự án.  
*A stakeholder is someone whose interests are tied to the project.*

Họ không nhất thiết hiểu kỹ thuật.  
*They do not necessarily understand the technology.*

Nhóm này gồm quản lý sản phẩm, giám đốc và bộ phận kinh doanh.  
*This group includes product managers, directors, and the business side.*

Một staff engineer phải trình bày được quyết định kỹ thuật bằng ngôn ngữ của họ.  
*A staff engineer must present technical decisions in their language.*

Ngôn ngữ đó là **rủi ro, chi phí và thời gian**.  
*That language is risk, cost, and time.*

Ngôn ngữ đó không phải là thuật ngữ kỹ thuật.  
*That language is not technical jargon.*

**Cách nói với quản lý sản phẩm hoặc sếp**  
*How to talk to a product manager or a boss*

> *"Chúng ta thêm được tính năng tìm kiếm mạnh mà **không phải dựng thêm hệ thống nào**."*  
> *"We can add powerful search without standing up any new system."*
>
> *"Chúng ta dùng chính cơ sở dữ liệu đang có."*  
> *"We use the database we already have."*
>
> *"Đội ngũ đã quen vận hành nó."*  
> *"The team already knows how to operate it."*
>
> *"Vì vậy rủi ro thấp."*  
> *"The risk is therefore low."*
>
> *"Chúng ta cũng không phát sinh chi phí hạ tầng."*  
> *"We also incur no new infrastructure cost."*
>
> *"Sau này lượng người dùng có thể tăng mạnh."*  
> *"User volume may grow sharply later."*
>
> *"Tìm kiếm có thể trở thành tính năng cốt lõi."*  
> *"Search may become a core feature."*
>
> *"Khi đó chúng ta sẽ cân nhắc một hệ thống chuyên dụng."*  
> *"We will then consider a dedicated system."*
>
> *"Đó là bài toán của lúc đã thành công."*  
> *"That is a problem for when we have already succeeded."*
>
> *"Đó không phải bài toán của bây giờ."*  
> *"That is not a problem for right now."*

Hãy để ý cách diễn đạt trên.  
*Please notice the phrasing above.*

Không có chữ nào là "GIN", "tsvector" hay "inverted index".  
*Not a single word is "GIN", "tsvector", or "inverted index".*

Thay vào đó là rủi ro thấp và không tốn thêm tiền.  
*Instead there is low risk and no extra money.*

Thay vào đó còn có một lộ trình rõ ràng cho tương lai.  
*Instead there is also a clear roadmap for the future.*

Đó là ngôn ngữ mà stakeholder nghe được.  
*That is the language a stakeholder can hear.*

**Ảnh hưởng tới roadmap**  
*Impact on the roadmap*

Quyết định về text search configuration có tầm ảnh hưởng rộng.  
*The text search configuration decision has broad impact.*

Quyết định đó gồm chọn ngôn ngữ nào.  
*That decision includes which language to choose.*

Quyết định đó gồm có dùng thesaurus hay không.  
*That decision includes whether to use a thesaurus.*

Quyết định đó gồm gán trọng số ra sao.  
*That decision includes how to assign weights.*

Nó ảnh hưởng tới chất lượng tìm kiếm của **toàn bộ sản phẩm**.  
*It affects the search quality of the entire product.*

Mỗi lần đổi config là bạn phải xây lại index cho toàn bảng.  
*Every configuration change forces a full-table index rebuild.*

Việc đó tốn thời gian và có rủi ro.  
*That costs time and carries risk.*

Vì vậy hãy đối xử với config như một **migration**.  
*So please treat the configuration like a migration.*

Migration là một thay đổi cấu trúc database.  
*A migration is a change to the database structure.*

Config cần có phiên bản.  
*The configuration needs versioning.*

Config cần có kế hoạch.  
*The configuration needs a plan.*

Config cần có phương án quay lui.  
*The configuration needs a rollback path.*

**Ảnh hưởng tới cấu trúc đội ngũ**  
*Impact on team structure*

Chọn FTS trong Postgres nghĩa là đội backend hiện tại tự lo được toàn bộ.  
*Choosing Postgres FTS means the current backend team handles everything itself.*

Chọn Elasticsearch nghĩa là ai đó phải học vận hành một cụm JVM.  
*Choosing Elasticsearch means someone must learn to operate a JVM cluster.*

Người đó phải trực sự cố cho cụm máy đó.  
*That person must go on call for that cluster.*

Về lâu dài bạn có thể phải tuyển người chuyên trách.  
*In the long run you may need to hire a specialist.*

Đây là một luận điểm **tổ chức**.  
*This is an organizational argument.*

Đây không phải luận điểm kỹ thuật.  
*This is not a technical argument.*

Biết đưa loại luận điểm này ra bàn là điều người ta chờ đợi ở tầm staff.  
*Bringing this kind of argument to the table is what people expect at staff level.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff engineer
*4.6. A sample system design question with a staff engineer's approach*

> **Đề bài**  
> *The prompt*
>
> *"Thiết kế chức năng tìm kiếm cho một trang tin tức hoặc thương mại điện tử."*  
> *"Design the search feature for a news site or an e-commerce site."*
>
> *"Trang này có 20 triệu bài viết hoặc sản phẩm."*  
> *"The site has 20 million articles or products."*
>
> *"Hệ thống phải hỗ trợ tìm theo từ khóa."*  
> *"The system must support keyword search."*
>
> *"Hệ thống phải chịu được lỗi chính tả."*  
> *"The system must tolerate typos."*
>
> *"Hệ thống phải hiểu được từ đồng nghĩa."*  
> *"The system must understand synonyms."*
>
> *"Hệ thống phải lọc theo danh mục và thời gian."*  
> *"The system must filter by category and time."*
>
> *"Độ trễ p95 phải dưới 150ms."*  
> *"The p95 latency must stay under 150ms."*
>
> *"Hệ thống phải hỗ trợ nhiều ngôn ngữ."*  
> *"The system must support multiple languages."*

Khung trả lời gồm bốn nhịp.  
*The answer framework has four beats.*

Bốn nhịp là làm rõ đề, thiết kế, mở rộng quy mô và nói rõ đánh đổi.  
*The four beats are clarify, design, scale, and state the trade-offs.*

Điều quan trọng nhất không phải là đưa ra "đáp án đúng".  
*The most important thing is not giving the right answer.*

Điều quan trọng nhất là **nói to những đánh đổi bạn đang cân nhắc**.  
*The most important thing is voicing the trade-offs you are weighing.*

**Nhịp 1 — Làm rõ đề (clarify)**  
*Beat 1 — clarify the problem*

Bạn đừng vẽ ngay.  
*Do not start drawing immediately.*

Hãy hỏi trước.  
*Please ask first.*

- QPS dự kiến là bao nhiêu?  
  *What is the expected QPS?*

  **QPS** là viết tắt của *Queries Per Second* — số truy vấn mỗi giây.  
  *QPS stands for Queries Per Second.*

- Nội dung được cập nhật thường xuyên tới mức nào?  
  *How often does the content get updated?*

  Câu trả lời quyết định việc chọn GIN hay GiST.  
  *The answer decides between GIN and GiST.*

- Ta ưu tiên **recall** hay **precision**?  
  *Do we prioritize recall or precision?*

  *Recall* nghĩa là không bỏ sót kết quả đúng.  
  *Recall means missing no correct result.*

  *Precision* nghĩa là không trả về kết quả rác.  
  *Precision means returning no junk result.*

  Hai thứ này đánh đổi lẫn nhau.  
  *These two trade off against each other.*

- Ngân sách hạ tầng ra sao?  
  *What is the infrastructure budget?*

- Có bắt buộc phải giải thích được vì sao một kết quả xếp hạng cao không?  
  *Must we be able to explain why a result ranks high?*

**Nhịp 2 — Mô hình dữ liệu**  
*Beat 2 — the data model*

Ta có một bảng `docs`.  
*We have one `docs` table.*

Bảng này chứa metadata gồm danh mục, thời gian và ngôn ngữ.  
*This table holds metadata: category, time, and language.*

Bảng này có cột `search_vector` là generated column.  
*This table has a `search_vector` generated column.*

Cột đó dùng `setweight` theo thứ tự title, body rồi tags.  
*That column uses `setweight` in the order title, body, then tags.*

Bảng này có cột `embedding` kiểu `vector`.  
*This table has an `embedding` column of type `vector`.*

Bạn partition bảng theo ngôn ngữ hoặc theo thời gian.  
*You partition the table by language or by time.*

**Nhịp 3 — Index**  
*Beat 3 — indexes*

Bạn dựng GIN trên `search_vector` cho nhánh từ khóa.  
*You build GIN on `search_vector` for the keyword branch.*

Bạn dựng GIN với `gin_trgm_ops` trên title cho nhánh chống lỗi chính tả.  
*You build GIN with `gin_trgm_ops` on the title for the typo-tolerance branch.*

Bạn dựng HNSW trên `embedding` cho nhánh ngữ nghĩa.  
*You build HNSW on `embedding` for the semantic branch.*

**Tất cả nằm trong cùng một Postgres.**  
*All of them live inside a single Postgres.*

**Nhịp 4 — Truy vấn theo kiểu hybrid**  
*Beat 4 — hybrid querying*

Ba nhánh chạy song song.  
*Three branches run in parallel.*

Nhánh FTS dùng `websearch_to_tsquery` cho an toàn với input người dùng.  
*The FTS branch uses `websearch_to_tsquery` for safety with user input.*

Nhánh trigram bắt lỗi gõ.  
*The trigram branch catches typos.*

Nhánh vector bắt từ đồng nghĩa.  
*The vector branch catches synonyms.*

Sau đó bạn hợp nhất ba nhánh bằng RRF.  
*Then you merge the three branches with RRF.*

Điểm mấu chốt về hiệu năng rất rõ.  
*The key performance point is clear.*

Bạn **lọc cứng theo danh mục và thời gian TRƯỚC**.  
*You filter hard by category and time FIRST.*

Nhờ vậy mỗi nhánh chỉ phải xếp hạng một tập nhỏ.  
*Each branch then ranks only a small set.*

**Nhịp 5 — Đạt mục tiêu độ trễ**  
*Beat 5 — hitting the latency target*

Bạn lọc chọn lọc trước khi xếp hạng.  
*You filter selectively before ranking.*

Bạn lưu sẵn `tsvector`.  
*You pre-store the `tsvector`.*

Bạn chỉ xếp hạng top-N.  
*You rank only the top N.*

Bạn đẩy lưu lượng tìm kiếm sang read replica.  
*You push search traffic to a read replica.*

Quan trọng nhất là bạn **đo p95 thật trên dữ liệu thật**.  
*Most importantly, you measure real p95 on real data.*

Bạn đừng suy đoán.  
*Do not speculate.*

**Nhịp 6 — Đa ngôn ngữ**  
*Beat 6 — multilingual support*

Bạn lưu tên config vào một cột.  
*You store the configuration name in a column.*

Bạn index theo `GIN(to_tsvector(lang_col, body))`.  
*You index on `GIN(to_tsvector(lang_col, body))`.*

Nhờ vậy mỗi hàng dùng bộ luật ngôn ngữ riêng.  
*Each row then uses its own language rule set.*

Bạn dùng mô hình embedding đa ngữ cho nhánh vector.  
*You use a multilingual embedding model for the vector branch.*

**Nhịp 7 — Vận hành**  
*Beat 7 — operations*

Bạn giám sát tỉ lệ rơi về seq scan.  
*You monitor the share of queries falling back to seq scan.*

Bạn giám sát p99, kích thước index và độ trễ ghi.  
*You monitor p99, index size, and write latency.*

Bạn dùng `REINDEX CONCURRENTLY` khi đổi config.  
*You use `REINDEX CONCURRENTLY` when changing the configuration.*

Bạn có khi phải đổi mô hình embedding.  
*You may need to change the embedding model.*

Khi đó bạn triển khai kiểu **blue-green**.  
*You then deploy in blue-green style.*

Bạn dựng song song bộ index mới.  
*You build the new index set in parallel.*

Bạn kiểm chứng xong mới chuyển toàn bộ lưu lượng sang.  
*You verify it before switching all traffic over.*

Lý do là embedding cũ và mới **không so sánh được với nhau**.  
*The reason is that old and new embeddings are not comparable.*

Trộn lẫn chúng sẽ cho kết quả sai hoàn toàn.  
*Mixing them produces completely wrong results.*

**Nhịp 8 — Biết giới hạn của lựa chọn mình đưa ra**  
*Beat 8 — knowing the limits of your own choice*

Bạn kết thúc bằng câu sau.  
*You close with the following line.*

> *"Tìm kiếm có thể trở thành workload chính."*  
> *"Search may become the primary workload."*
>
> *"Chúng ta có thể cần BM25 và analyzer cao cấp."*  
> *"We may need BM25 and advanced analyzers."*
>
> *"Quy mô có thể vượt xa mức hiện tại."*  
> *"The scale may grow far beyond today's level."*
>
> *"Khi đó tôi sẽ chuyển nhánh từ khóa sang Elasticsearch."*  
> *"I would then move the keyword branch to Elasticsearch."*
>
> *"Tôi sẽ giữ pgvector cho nhánh ngữ nghĩa."*  
> *"I would keep pgvector for the semantic branch."*

Bạn chủ động nêu ra khi nào giải pháp của chính mình không còn phù hợp.  
*You proactively state when your own solution stops fitting.*

Đó là dấu hiệu staff rõ ràng nhất trong một buổi phỏng vấn.  
*That is the clearest staff-level signal in an interview.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 WRAPPING UP FOR INTERVIEWS (Interview cheatsheet)*

> Phần này để ôn nhanh trước buổi phỏng vấn 30 phút.  
> *This part is for a quick 30-minute review before an interview.*
>
> Mọi thứ ở đây đã được giải thích đầy đủ ở trên.  
> *Everything here was fully explained above.*
>
> Đây chỉ là bản rút gọn để nhắc lại.  
> *This is only a condensed reminder.*

### 5.1. Keywords bắt buộc nhớ
*5.1. Keywords you must remember*

| Thuật ngữ tiếng Anh / English term | Định nghĩa một dòng / One-line definition |
|---|---|
| **DBMS / RDBMS** | Phần mềm quản lý cơ sở dữ liệu / loại tổ chức dữ liệu thành các bảng có quan hệ — database management software / the kind that organizes data into related tables |
| **Primary key / Foreign key** | Cột định danh duy nhất mỗi hàng / cột trỏ tới khóa chính của bảng khác — the column uniquely identifying each row / the column pointing at another table's primary key |
| **BLOB** | Binary Large Object — lưu ảnh, video, file dạng nhị phân trong database / storing images, video, and files in binary form inside the database |
| **ORDBMS / extensibility** | Quan hệ - đối tượng; PostgreSQL cắm thêm được kiểu dữ liệu, hàm, index, extension / object-relational; PostgreSQL accepts new types, functions, indexes, and extensions |
| **Full-text search (FTS)** | Tìm tài liệu ngôn ngữ tự nhiên theo lexeme, không theo ký tự / searching natural-language documents by lexeme rather than by character |
| **Token / Tokenize** | Mẩu văn bản cắt ra từ câu / hành động cắt đó — a piece of text cut from a sentence / the act of cutting it |
| **Stemming** | Rút từ về dạng gốc chung: "running" → "run" / reducing a word to its shared base form |
| **Lexeme** | Từ đã chuẩn hóa xong — đơn vị so khớp thật sự của FTS / a fully normalized word, the real matching unit of FTS |
| **Stop-word** | Từ quá phổ biến nên bị loại bỏ: "the", "is", "a" / a word too common to keep |
| **`tsvector`** | Danh sách lexeme đã loại trùng, sắp xếp, kèm vị trí — tài liệu đã tiền xử lý / a deduplicated, sorted lexeme list with positions — the pre-processed document |
| **`tsquery`** | Truy vấn dạng lexeme nối bằng `&` (AND), `\|` (OR), `!` (NOT), `<->` (liền kề) / a lexeme-form query joined by those operators |
| **`@@`** | Toán tử so khớp một `tsvector` với một `tsquery`, trả true/false / the operator matching a `tsvector` against a `tsquery` |
| **`to_tsvector` / `to_tsquery`** | Hàm tạo tsvector từ text / tạo tsquery từ chuỗi có toán tử — builds a tsvector from text / builds a tsquery from an operator string |
| **`plainto_` / `phraseto_` / `websearch_to_tsquery`** | Tự AND các từ / tìm nguyên cụm / cú pháp kiểu Google, an toàn với input người dùng — auto-ANDs words / finds an exact phrase / Google-style syntax, safe for user input |
| **Text search configuration** | Bộ luật xử lý ngôn ngữ: `'english'` (stem và bỏ stop-word), `'simple'` (chỉ lowercase) / a language rule set |
| **Parser / Dictionary** | Bộ tách token và gán loại / bộ biến token thành lexeme hoặc loại bỏ — splits and types tokens / turns tokens into lexemes or discards them |
| **Inverted index** | Bảng tra "từ → danh sách tài liệu chứa từ đó" — như mục lục cuối sách / a word-to-document lookup table, like a book index |
| **GIN / GiST** | Inverted index chính xác, chuẩn cho FTS / signature tree lossy, xây nhanh, index nhỏ — the exact inverted index standard for FTS / a lossy signature tree that builds fast and stays small |
| **Posting list** | Danh sách các hàng chứa một lexeme, lưu trong GIN / the list of rows holding one lexeme, stored in GIN |
| **`setweight` / `ts_rank` / `ts_rank_cd`** | Gán trọng số A-D / chấm điểm theo tần suất / chấm điểm có thưởng khi từ gần nhau — assigns weights A to D / scores by frequency / scores with a proximity bonus |
| **Functional index / Generated column** | Index trên biểu thức / cột Postgres tự sinh và tự cập nhật (`STORED`, PG 12+) — an index on an expression / a column Postgres generates and refreshes itself |
| **`coalesce`** | Trả về giá trị thay thế khi gặp NULL — bắt buộc dùng khi ghép cột thành tsvector / returns a fallback on NULL, mandatory when concatenating columns |
| **`pg_trgm` / trigram** | Extension khớp gần đúng chống lỗi chính tả, dựa trên các đoạn 3 ký tự / a fuzzy-matching extension based on three-character chunks |
| **Trade-off / Edge case / Bottleneck** | Sự đánh đổi / trường hợp biên / điểm nghẽn quyết định tốc độ cả hệ thống — a trade-off / an edge case / the bottleneck setting the whole system's pace |
| **Hybrid search / RRF** | Kết hợp FTS và vector search / hợp nhất bằng cách cộng nghịch đảo thứ hạng — combining FTS with vector search / fusing them by summing reciprocal ranks |
| **BM25** | Công thức chấm điểm liên quan tiên tiến, tiêu chuẩn công nghiệp cho keyword search / an advanced relevance formula, the industry standard for keyword search |

### 5.2. Core concepts — nếu chỉ được nhớ 10 điều
*5.2. Core concepts — if you only remember 10 things*

1. **FTS khớp theo lexeme**, không theo ký tự.  
*1. FTS matches by lexeme, not by character.*

Vì vậy "run" tìm ra được "running".  
*So "run" finds "running".*

`LIKE` vĩnh viễn không làm được điều đó.  
*`LIKE` can never do that.*

2. **`tsvector @@ tsquery` là trái tim của FTS.**  
*2. `tsvector @@ tsquery` is the heart of FTS.*

Cả tài liệu lẫn truy vấn đều đi qua **cùng một** bộ chuẩn hóa.  
*Both the document and the query pass through the same normalizer.*

Nhờ vậy chúng gặp nhau ở tầng lexeme.  
*They therefore meet at the lexeme level.*

3. **GIN inverted index** khiến thời gian truy vấn phụ thuộc **số hàng khớp**.  
*3. A GIN inverted index makes query time depend on the number of matching rows.*

Thời gian không phụ thuộc tổng số hàng.  
*The time does not depend on the total row count.*

Đó là lý do FTS vẫn nhanh khi bảng lớn.  
*That is why FTS stays fast on a large table.*

4. Cách làm chuẩn 2026 là **generated column STORED**.  
*4. The 2026 standard approach is a STORED generated column.*

Lựa chọn thay thế là **functional GIN index**.  
*The alternative is a functional GIN index.*

Bạn đừng viết trigger đồng bộ bằng tay nữa.  
*Please stop hand-writing synchronization triggers.*

5. Input của người dùng luôn đi qua **`websearch_to_tsquery`**.  
*5. User input always goes through `websearch_to_tsquery`.*

Bạn không bao giờ nhét nó thẳng vào `to_tsquery`.  
*You never feed it straight into `to_tsquery`.*

6. **`setweight` với A, B, C, D kết hợp `ts_rank`** điều khiển relevance ngay trong database.  
*6. `setweight` with A, B, C, D plus `ts_rank` controls relevance inside the database.*

Bạn không cần code ở tầng ứng dụng.  
*You need no application-layer code.*

7. Bạn **luôn dùng `coalesce(col, '')`** khi ghép nhiều cột.  
*7. Always use `coalesce(col, '')` when concatenating several columns.*

Một NULL là cả hàng biến mất khỏi kết quả.  
*A single NULL makes the whole row vanish from the results.*

Chuyện đó xảy ra âm thầm, không báo lỗi.  
*It happens silently, with no error.*

8. **FTS mù ngữ nghĩa.**  
*8. FTS is blind to meaning.*

Cụm "xe hơi" khác cụm "ô tô" với nó.  
*To it, "xe hơi" differs from "ô tô".*

FTS cũng **không sửa lỗi chính tả**.  
*FTS also does not fix typos.*

Bạn bù bằng vector search và `pg_trgm`.  
*You compensate with vector search and `pg_trgm`.*

9. **Postgres FTS thay được Elasticsearch** cho ứng dụng vừa và nhỏ.  
*9. Postgres FTS replaces Elasticsearch for small and medium applications.*

Bạn chọn Elasticsearch khi tìm kiếm là sản phẩm chính.  
*You choose Elasticsearch when search is the main product.*

Bạn cũng chọn nó khi quy mô rất lớn.  
*You also choose it at very large scale.*

10. **Extensibility của Postgres là một luận điểm chiến lược.**  
*10. Postgres extensibility is a strategic argument.*

Keyword, fuzzy và semantic search cùng nằm trong một database.  
*Keyword, fuzzy, and semantic search all live in one database.*

Bạn không thêm hệ thống nào cả.  
*You add no new system at all.*

### 5.3. Mental models — cách tư duy để trả lời trôi chảy
*5.3. Mental models — ways of thinking that make answers flow*

**"Mục lục cuối sách"** giải thích inverted index và GIN trong đúng một câu.  
*"The index at the back of a book" explains an inverted index and GIN in one sentence.*

Bạn nhớ nói thêm vế quan trọng.  
*Remember to add the important half.*

*Thời gian tra không phụ thuộc độ dày cuốn sách.*  
*Lookup time does not depend on the book's thickness.*

**"run = running = ran"** giải thích lexeme và stemming ngay lập tức.  
*"run = running = ran" explains lexemes and stemming instantly.*

Ai cũng hiểu câu đó.  
*Everyone understands it.*

**"Từ khóa và ý nghĩa"** là cách phân vai rất gọn.  
*"Keywords and meaning" is a compact way to split the roles.*

FTS bắt đúng từ.  
*FTS catches the exact word.*

Vector hiểu nghĩa.  
*Vectors understand meaning.*

Hybrid là cả hai.  
*Hybrid is both.*

**"Một database, ba tầng tìm kiếm"** tóm tắt kiến trúc.  
*"One database, three search layers" summarizes the architecture.*

Ba tầng là lexeme với FTS, lỗi gõ với trigram, ngữ nghĩa với vector.  
*The three layers are lexemes via FTS, typos via trigrams, and meaning via vectors.*

**"Search là một tính năng hay là cả sản phẩm?"** là kim chỉ nam.  
*"Is search a feature or the whole product?" is the guiding question.*

Câu đó giúp bạn quyết định giữa Postgres FTS và Elasticsearch.  
*It helps you decide between Postgres FTS and Elasticsearch.*

**"Mỗi hệ thống thêm vào là một người bị đánh thức lúc 3 giờ sáng."**  
*"Every added system is one more person woken at 3 a.m."*

Đó là cách nói về chi phí vận hành.  
*That is a way of framing operational cost.*

Ai cũng thấm câu đó.  
*That line lands with everyone.*

### 5.4. Code cần thuộc lòng
*5.4. Code you should memorize*

**(a) FTS tối thiểu kèm index — interviewer hay bảo viết tại chỗ**  
*(a) Minimal FTS with an index — interviewers often ask you to write this on the spot*

```sql
CREATE INDEX docs_fts ON docs USING GIN (to_tsvector('english', body));

SELECT title FROM docs
WHERE to_tsvector('english', body)
   @@ websearch_to_tsquery('english', 'postgres index');
```

**(b) Generated column có trọng số kèm xếp hạng — bản "production"**  
*(b) A weighted generated column with ranking — the production version*

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

**(c) Chạy tay `tsvector` — chứng tỏ bạn hiểu bản chất chứ không học vẹt**  
*(c) Hand-running a `tsvector` — proof you understand it rather than memorize it*

```
'The quick brown foxes are running'
  → tokenize:      The | quick | brown | foxes | are | running
  → tokenize:      The | quick | brown | foxes | are | running
  → bỏ stop-word:  (bỏ the, are)
  → drop stop-words: (drop the, are)
  → stemming:      foxes → fox,  running → run
  → stemming:      foxes → fox,  running → run
  → tsvector:      'brown':3 'fox':4 'quick':2 'run':6
  → tsvector:      'brown':3 'fox':4 'quick':2 'run':6
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions with suggested answers*

**1. "Khác nhau giữa FTS và `LIKE` là gì?"**  
*1. "What is the difference between FTS and `LIKE`?"*

> `LIKE '%x%'` so khớp ký tự.  
> *`LIKE '%x%'` matches characters.*
>
> Nó không dùng được B-tree index.  
> *It cannot use a B-tree index.*
>
> Nó rơi về seq scan với độ phức tạp `O(N)`.  
> *It falls back to a seq scan with `O(N)` complexity.*
>
> Nó không hiểu biến thể từ.  
> *It does not understand word variants.*
>
> Nó khớp nhầm vào giữa từ khác.  
> *It matches inside other words by mistake.*
>
> FTS so khớp lexeme.  
> *FTS matches lexemes.*
>
> FTS dùng GIN inverted index.  
> *FTS uses a GIN inverted index.*
>
> Thời gian vì vậy phụ thuộc số kết quả.  
> *The time therefore depends on the number of results.*
>
> Thời gian không phụ thuộc tổng số hàng.  
> *The time does not depend on the total row count.*
>
> FTS còn stem và bỏ stop-word.  
> *FTS also stems and drops stop-words.*

**2. "`tsvector` và `tsquery` là gì, `@@` làm gì?"**  
*2. "What are `tsvector` and `tsquery`, and what does `@@` do?"*

> `tsvector` là tài liệu đã tiền xử lý thành danh sách lexeme.  
> *A `tsvector` is a document pre-processed into a lexeme list.*
>
> Danh sách đó đã loại trùng, đã sắp xếp và có kèm vị trí.  
> *That list is deduplicated, sorted, and carries positions.*
>
> `tsquery` là truy vấn cũng ở dạng lexeme.  
> *A `tsquery` is a query also in lexeme form.*
>
> Các lexeme được nối bằng toán tử logic.  
> *The lexemes get joined by logical operators.*
>
> `@@` kiểm tra `tsvector` có thỏa `tsquery` hay không.  
> *`@@` checks whether the `tsvector` satisfies the `tsquery`.*

**3. [TRADE-OFF] "Chọn GIN hay GiST?"**  
*3. [TRADE-OFF] "Do you choose GIN or GiST?"*

> GIN chính xác và truy vấn nhanh.  
> *GIN is exact and queries fast.*
>
> GIN xây chậm, ngốn RAM và index to.  
> *GIN builds slowly, eats RAM, and produces a large index.*
>
> GIN là mặc định cho hệ đọc nhiều.  
> *GIN is the default for read-heavy systems.*
>
> GiST lossy nên phải recheck.  
> *GiST is lossy and therefore needs a recheck.*
>
> Truy vấn của GiST vì vậy chậm hơn.  
> *GiST queries are therefore slower.*
>
> Bù lại GiST xây nhanh và index nhỏ.  
> *In exchange GiST builds fast and stays small.*
>
> GiST chịu cập nhật tốt hơn.  
> *GiST tolerates updates better.*
>
> Bạn chọn GiST khi ghi nhiều hoặc bị giới hạn dung lượng.  
> *You choose GiST on write-heavy loads or under space constraints.*
>
> *Bạn luôn nói ra cả cái mất.*  
> *Always state the loss as well.*
>
> *Bạn đừng chỉ nói cái được.*  
> *Do not mention only the gain.*

**4. [BẪY] "Index FTS đã tạo rồi mà truy vấn vẫn chậm, vì sao?"**  
*4. [TRAP] "The FTS index exists, so why is the query still slow?"*

> Khả năng cao là biểu thức trong `WHERE` không trùng khít biểu thức lúc `CREATE INDEX`.  
> *Most likely the `WHERE` expression does not exactly match the `CREATE INDEX` one.*
>
> Có thể bạn thiếu tham số config.  
> *You may be missing the configuration argument.*
>
> Có thể bạn dùng dạng một tham số thay vì hai.  
> *You may be using the one-argument form instead of two.*
>
> Index bị bỏ qua âm thầm.  
> *The index gets skipped silently.*
>
> Không có báo lỗi nào.  
> *There is no error.*
>
> Bạn kiểm tra bằng `EXPLAIN ANALYZE`.  
> *You check with `EXPLAIN ANALYZE`.*
>
> Bạn tìm dòng `Bitmap Index Scan`.  
> *You look for the `Bitmap Index Scan` line.*
>
> Bạn thấy `Seq Scan` là dính lỗi này.  
> *Seeing `Seq Scan` means you have hit this bug.*

**5. [BẪY] "Một bài viết rõ ràng chứa từ khóa. Bài đó không bao giờ xuất hiện trong kết quả. Vì sao?"**  
*5. [TRAP] "An article clearly contains the keyword. It never appears in the results. Why?"*

> Gần như chắc chắn là một cột nguồn bị NULL khi ghép.  
> *Almost certainly a source column was NULL during concatenation.*
>
> Bạn đã quên `coalesce`.  
> *You forgot `coalesce`.*
>
> Cả `tsvector` của hàng đó thành NULL.  
> *The whole row's `tsvector` became NULL.*
>
> Hàng đó biến mất.  
> *That row disappeared.*
>
> Bạn kiểm tra bằng `WHERE search_vector IS NULL`.  
> *You check with `WHERE search_vector IS NULL`.*
>
> Bạn chữa bằng `coalesce(col, '')`.  
> *You fix it with `coalesce(col, '')`.*

**6. "Xử lý input thô của người dùng thế nào?"**  
*6. "How do you handle raw user input?"*

> Bạn dùng `websearch_to_tsquery`.  
> *You use `websearch_to_tsquery`.*
>
> Nó có cú pháp kiểu Google và an toàn với ký tự lạ.  
> *It has Google-style syntax and is safe with odd characters.*
>
> Bạn cũng có thể dùng `plainto_tsquery`.  
> *You may also use `plainto_tsquery`.*
>
> Bạn tuyệt đối không đưa input thô vào `to_tsquery`.  
> *You absolutely do not feed raw input into `to_tsquery`.*
>
> Cách đó gây syntax error và lỗi 500.  
> *That causes a syntax error and a 500 error.*

**7. "FTS có hiểu từ đồng nghĩa không? Cần thì làm sao?"**  
*7. "Does FTS understand synonyms? What do you do if you need that?"*

> Câu trả lời là không.  
> *The answer is no.*
>
> Ngoại lệ là khi bạn tự cấu hình thesaurus dictionary.  
> *The exception is configuring a thesaurus dictionary yourself.*
>
> Bạn muốn hiểu nghĩa thật sự.  
> *You may want real semantic understanding.*
>
> Khi đó bạn phải thêm vector search bằng `pgvector`.  
> *You then need to add vector search with `pgvector`.*
>
> Sau đó bạn kết hợp hai nhánh thành hybrid search.  
> *Then you combine the two branches into hybrid search.*
>
> Bạn hợp nhất chúng bằng RRF.  
> *You fuse them with RRF.*

**8. "FTS có sửa được lỗi chính tả không?"**  
*8. "Can FTS fix typos?"*

> Câu trả lời là không.  
> *The answer is no.*
>
> FTS chuẩn hóa biến thể ngữ pháp.  
> *FTS normalizes grammatical variants.*
>
> FTS không đoán ý định người gõ sai.  
> *FTS does not guess a misspeller's intent.*
>
> Bạn cần extension `pg_trgm` với index `gin_trgm_ops`.  
> *You need the `pg_trgm` extension with a `gin_trgm_ops` index.*
>
> Bạn dùng toán tử `%` và hàm `similarity`.  
> *You use the `%` operator and the `similarity` function.*

**9. [SCALE] "Có 20 triệu tài liệu, khi nào nên bỏ Postgres FTS để sang Elasticsearch?"**  
*9. [SCALE] "With 20 million documents, when should you leave Postgres FTS for Elasticsearch?"*

> Bạn chuyển khi tìm kiếm trở thành **workload** chính.  
> *You switch when search becomes the primary workload.*
>
> *Workload* là khối lượng công việc chính mà hệ thống phải gánh.  
> *A workload is the main body of work a system carries.*
>
> Bạn cũng chuyển khi cần BM25 và analyzer cao cấp.  
> *You also switch when you need BM25 and advanced analyzers.*
>
> Bạn cũng chuyển khi quy mô vượt xa mức Postgres phục vụ thoải mái.  
> *You also switch when the scale far exceeds what Postgres serves comfortably.*
>
> Trước ngưỡng đó, Postgres FTS thắng ở sự đơn giản trong vận hành.  
> *Below that threshold, Postgres FTS wins on operational simplicity.*
>
> Nó cũng thắng ở khả năng JOIN thẳng với dữ liệu nghiệp vụ.  
> *It also wins on joining directly with business data.*
>
> Nó cũng thắng ở tính nhất quán ACID.  
> *It also wins on ACID consistency.*
>
> Nó vẫn giữ được `pgvector` cho nhánh ngữ nghĩa trong cùng một hệ.  
> *It still keeps `pgvector` for the semantic branch in the same system.*

**10. [SCALE] "Bottleneck đầu tiên của FTS khi lớn lên là gì?"**  
*10. [SCALE] "What is the first FTS bottleneck as things grow?"*

> Thường là I/O của khâu xếp hạng.  
> *It is usually I/O in the ranking stage.*
>
> Hàm `ts_rank` phải mở `tsvector` của từng tài liệu khớp.  
> *The `ts_rank` function must open the `tsvector` of every matching document.*
>
> Một truy vấn với từ phổ biến khớp hàng triệu hàng.  
> *A query on a common word matches millions of rows.*
>
> Truy vấn đó sẽ ngốn I/O khủng khiếp.  
> *That query devours a terrifying amount of I/O.*
>
> Bạn chữa bằng cách lọc cứng theo metadata trước khi xếp hạng.  
> *You fix it by filtering hard on metadata before ranking.*
>
> Bạn lưu sẵn `tsvector`.  
> *You pre-store the `tsvector`.*
>
> Bạn chỉ xếp hạng top-N.  
> *You rank only the top N.*

### 5.6. Câu trả lời "one-liner" đắt giá
*5.6. High-value one-liner answers*

FTS khớp theo lexeme chứ không theo ký tự.  
*FTS matches lexemes, not characters.*

Đó là lý do "run" tìm ra "running".  
*That is why "run" finds "running".*

`LIKE` sẽ không bao giờ làm được điều đó.  
*`LIKE` never will.*

GIN inverted index khiến thời gian tìm kiếm tỉ lệ với số kết quả khớp.  
*A GIN inverted index makes search time scale with the number of matches.*

Thời gian đó không tỉ lệ với kích thước bảng.  
*That time does not scale with the size of the table.*

Với ứng dụng vừa và nhỏ, Postgres FTS xóa sổ cụm Elasticsearch của bạn.  
*For small-to-medium apps, Postgres FTS deletes your Elasticsearch cluster.*

Bạn dùng cùng một database với ACID.  
*You use the same database with ACID.*

Bạn chỉ cần một câu SQL join.  
*You need only one SQL join.*

Bạn không có gì để đồng bộ.  
*You have nothing to sync.*

Không ai bị gọi dậy lúc 3 giờ sáng.  
*Nobody gets paged at 3 a.m.*

FTS biết từ.  
*FTS knows words.*

Vector biết nghĩa.  
*Vectors know meaning.*

Hybrid search chỉ đơn giản là cả hai.  
*Hybrid search is just both.*

Bạn hợp nhất chúng bằng RRF bên trong một cái Postgres.  
*You fuse them with RRF inside one Postgres.*

Tính năng chiến lược không phải là `tsvector`.  
*The strategic feature is not `tsvector`.*

Tính năng chiến lược là việc Postgres đủ mở rộng được.  
*The strategic feature is Postgres being extensible enough.*

Nó làm được keyword, fuzzy và semantic search mà không thêm hệ thống nào.  
*It does keyword, fuzzy, and semantic search without adding a single new system.*

NULL không ném ra lỗi nào trong FTS.  
*NULL does not throw an error in FTS.*

Nó chỉ lặng lẽ xóa một hàng khỏi mọi kết quả tìm kiếm.  
*It just quietly deletes a row from every search result.*

Đó là lý do `coalesce` không phải là tùy chọn.  
*That is why `coalesce` is not optional.*

---

## 📌 Ghi chú cuối
*📌 Closing notes*

**Kiểm chứng phiên bản khi ôn**  
*Verify the version numbers while reviewing*

Tính tới tháng 7/2026, bản PostgreSQL ổn định mới nhất là **18.4**.  
*As of July 2026, the latest stable PostgreSQL release is 18.4.*

PostgreSQL 19 đang ở giai đoạn beta.  
*PostgreSQL 19 is in the beta stage.*

Beta 2 ra ngày 16/07/2026.  
*Beta 2 came out on 16/07/2026.*

Bản chính thức dự kiến ra vào tháng 9/2026.  
*The final release is expected in September 2026.*

Có hai mốc phiên bản cần nhớ.  
*There are two version milestones to remember.*

`websearch_to_tsquery` cần **PostgreSQL 11 trở lên**.  
*`websearch_to_tsquery` needs PostgreSQL 11 or newer.*

Generated column `STORED` cần **PostgreSQL 12 trở lên**.  
*A `STORED` generated column needs PostgreSQL 12 or newer.*

Tài liệu chính thức về FTS nằm ở `postgresql.org/docs/current/textsearch.html`.  
*The official FTS documentation lives at `postgresql.org/docs/current/textsearch.html`.*

**Thực hành — đây là phần quan trọng nhất**  
*Practice — this is the most important part*

Đọc xong giáo trình này bạn *biết*.  
*After reading this course you know the material.*

Bạn phải gõ tay thì mới *thuộc*.  
*You must type it by hand to truly own it.*

Đây là lộ trình gợi ý.  
*Here is a suggested path.*

1. Bạn dựng một Postgres bằng Docker.  
*1. Stand up a Postgres with Docker.*

Docker là một công cụ chạy phần mềm trong môi trường đóng gói sẵn.  
*Docker is a tool for running software in a pre-packaged environment.*

Bạn không phải cài đặt lằng nhằng.  
*You avoid messy installation.*

2. Bạn chạy lại đúng ba câu lệnh ở mục 1.11.  
*2. Rerun exactly the three statements from section 1.11.*

Ba câu đó là `to_tsvector`, `to_tsquery` và `@@`.  
*Those three are `to_tsvector`, `to_tsquery`, and `@@`.*

Bạn đối chiếu kết quả với phần chạy tay ở mục 1.10.  
*You compare the result with the hand-run version in section 1.10.*

3. Bạn tạo một bảng vài nghìn hàng.  
*3. Create a table with a few thousand rows.*

Bạn thêm generated column có `setweight`.  
*You add a generated column with `setweight`.*

Bạn dựng GIN index.  
*You build a GIN index.*

4. Bạn chạy `EXPLAIN ANALYZE`.  
*4. Run `EXPLAIN ANALYZE`.*

Bạn **tận mắt nhìn thấy** dòng `Bitmap Index Scan`.  
*You see the `Bitmap Index Scan` line with your own eyes.*

Sau đó bạn cố tình viết sai biểu thức.  
*Then you deliberately write the expression wrong.*

Bạn nhìn thấy nó chuyển thành `Seq Scan`.  
*You watch it turn into `Seq Scan`.*

Khoảnh khắc nhìn thấy sự khác biệt này có giá trị hơn đọc mười trang lý thuyết.  
*That moment is worth more than ten pages of theory.*

5. Bạn cài `pgvector`.  
*5. Install `pgvector`.*

Bạn tự viết một truy vấn hybrid RRF theo mẫu ở mục 4.3.  
*You write a hybrid RRF query yourself, following the template in section 4.3.*

**Nối mạch với giáo trình trước**  
*Linking back to the previous course*

Bài này dạy tìm kiếm theo **từ khóa**.  
*This lesson teaches keyword search.*

Giáo trình `pgvector` dạy tìm kiếm theo **ngữ nghĩa**.  
*The `pgvector` course teaches semantic search.*

Ghép hai bài lại chính là **hybrid search**.  
*Putting the two together is exactly hybrid search.*

Đó là kiến trúc truy hồi nền tảng cho search hiện đại.  
*That is the foundational retrieval architecture for modern search.*

Đó cũng là kiến trúc nền tảng cho RAG.  
*It is also the foundational architecture for RAG.*

**Học tiếp gì sau bài này**  
*What to learn next*

- Bạn so sánh RRF với weighted fusion — hợp nhất theo trọng số.  
  *Compare RRF against weighted fusion.*

  Bạn tìm hiểu khi nào dùng cái nào.  
  *You work out when to use which.*

- **Reranking model** là mô hình chấm điểm lại top-N kết quả.  
  *A reranking model rescores the top N results.*

  Mục đích là tăng độ chính xác ở đỉnh danh sách.  
  *The purpose is higher precision at the top of the list.*

- **BM25** và các extension mang nó vào Postgres.  
  *BM25 and the extensions that bring it into Postgres.*

  Ví dụ là ParadeDB và `pg_search`.  
  *Examples are ParadeDB and `pg_search`.*

- Bạn xây text search configuration và dictionary tùy biến cho **tiếng Việt**.  
  *Build a custom text search configuration and dictionary for Vietnamese.*

  Bạn kết hợp chúng với `unaccent`.  
  *You combine them with `unaccent`.*
