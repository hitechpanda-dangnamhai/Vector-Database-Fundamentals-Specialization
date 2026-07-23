# Indexing cho Vector Search: Từ Binary Search tới IVFFlat & HNSW — Giáo trình Basic → Staff
*Indexing for Vector Search: From Binary Search to IVFFlat & HNSW — A Basic → Staff Course*

> **Nguồn gốc — xuất xứ tài liệu**
>
> Tài liệu gốc là một bài reading.  
> *The original material is a reading.*
>
> Tên bài là "Creating Indexes for Vector Search Optimization".  
> *Its title is "Creating Indexes for Vector Search Optimization".*
>
> Tác giả là Lavanya T S.  
> *The author is Lavanya T S.*
>
> Bài thuộc Course 3, IBM Vector Database Fundamentals.  
> *The reading belongs to Course 3, IBM Vector Database Fundamentals.*
>
> Bài gốc dạy định nghĩa database index — chỉ mục cơ sở dữ liệu.  
> *The reading teaches the definition of a database index.*
>
> Bài gốc dạy cách index tiết kiệm thời gian.  
> *The reading teaches how an index saves time.*
>
> Phần đó so sánh linear search với binary search.  
> *That part compares linear search against binary search.*
>
> Bài gốc dạy hai index của pgvector.  
> *The reading teaches two pgvector indexes.*
>
> Hai index đó là IVFFlat và HNSW.  
> *Those two indexes are IVFFlat and HNSW.*
>
> Bài gốc dạy cả tham số của chúng.  
> *The reading also covers their parameters.*
>
> Tôi giảng bám sát bài gốc.  
> *I follow the original reading closely.*
>
> Sau đó tôi đào sâu thêm.  
> *After that I go deeper.*
>
> Chỗ mở rộng mang nhãn **[MỞ RỘNG NGOÀI BÀI GỐC]**.  
> *Extended sections carry the label **[MỞ RỘNG NGOÀI BÀI GỐC]**.*

> **Vị trí trong series — chỗ đứng của bài này**
>
> Đây là mảnh "làm cho search NHANH".  
> *This is the piece about making search FAST.*
>
> Bộ ba bài trước gồm embed, store & search và keyword.  
> *The previous three lessons were embed, store & search, and keyword.*
>
> Bài embed dạy cách tạo vector.  
> *The embed lesson taught how to create vectors.*
>
> Bài store & search dùng pgvector.  
> *The store & search lesson used pgvector.*
>
> Bài keyword dùng FTS — tìm kiếm toàn văn.  
> *The keyword lesson used FTS, or full-text search.*
>
> Bài này zoom vào một câu hỏi duy nhất.  
> *This lesson zooms into a single question.*
>
> Bạn có hàng triệu vector trong database.  
> *You have millions of vectors in the database.*
>
> Làm sao tìm nhanh mà không quét hết?  
> *How do you search fast without scanning everything?*
>
> Một phần bài này trùng với giáo trình pgvector đầu tiên.  
> *Part of this lesson overlaps with the first pgvector course.*
>
> Phần trùng đó là HNSW và IVFFlat.  
> *That overlapping part is HNSW and IVFFlat.*
>
> Ở đây tôi tiếp cận từ **lý thuyết indexing**.  
> *Here I approach it from **indexing theory**.*
>
> Reading bỏ qua một mắt xích quan trọng.  
> *The reading skips one important link.*
>
> Tôi bổ sung mắt xích đó.  
> *I fill in that link.*
>
> Tôi không lặp lại nội dung cũ.  
> *I do not repeat the old content.*

> **⚠️ Đính chính nhỏ trong reading — sửa lỗi để đi phỏng vấn**
>
> Bạn cần biết ba đính chính dưới đây.  
> *You need to know the three corrections below.*
>
> Chúng giúp bạn nói đúng khi đi phỏng vấn.  
> *They help you speak accurately in an interview.*
>
> **Đính chính 1 / Correction 1**
>
> HNSW viết đầy đủ là Hierarchical Navigable Small **World**.  
> *HNSW stands for Hierarchical Navigable Small **World**.*
>
> Small World nghĩa là thế giới nhỏ.  
> *Small World refers to the small-world graph idea.*
>
> Reading viết "Small Words".  
> *The reading writes "Small Words".*
>
> Đó là một lỗi chính tả thuật ngữ.  
> *That is a spelling error in the technical term.*
>
> **Đính chính 2 / Correction 2**
>
> Reading nói các link của HNSW là khoảng cách Euclidean.  
> *The reading says HNSW links are Euclidean distances.*
>
> Thực ra HNSW dùng bất kỳ distance metric nào.  
> *In reality HNSW uses any distance metric.*
>
> Bạn tự cấu hình metric đó.  
> *You configure that metric yourself.*
>
> Chính ví dụ của reading dùng `vector_cosine_ops`.  
> *The reading's own example uses `vector_cosine_ops`.*
>
> Ops class đó ứng với cosine.  
> *That ops class maps to cosine.*
>
> Nó không phải Euclidean.  
> *It is not Euclidean.*
>
> Cạnh graph nối theo độ gần.  
> *Graph edges connect points by closeness.*
>
> Độ gần tính theo metric đã chọn.  
> *Closeness is measured with the chosen metric.*
>
> **Đính chính 3 / Correction 3**
>
> Hình HNSW trong reading đánh nhãn tầng sai.  
> *The HNSW figure in the reading labels its layers wrongly.*
>
> Hình ghi "Layer 3, 2, 3, 0".  
> *The figure reads "Layer 3, 2, 3, 0".*
>
> Nhãn đúng là Layer 3 → 2 → 1 → 0.  
> *The correct labels are Layer 3 → 2 → 1 → 0.*
>
> Thứ tự này chạy từ trên xuống dưới.  
> *This order runs from top to bottom.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson map (Overview)*

**Bài này dạy gì / What this lesson teaches**

Bài này định nghĩa index — chỉ mục.  
*This lesson defines the index.*

Bài này giải thích vai trò của index.  
*This lesson explains the role of an index.*

Không có index, việc tìm kiếm là quét từng cái.  
*Without an index, searching means scanning item by item.*

Cách quét đó chậm khủng khiếp trên dữ liệu lớn.  
*That scan is terribly slow on large data.*

Có index, việc tìm kiếm thành nhảy thẳng tới nơi cần.  
*With an index, searching becomes a jump straight to the target.*

Cách nhảy đó rất nhanh.  
*That jump is very fast.*

Ta bắt đầu từ index kinh điển.  
*We start from the classic indexes.*

Index kinh điển gồm binary search và B-tree.  
*The classic indexes are binary search and the B-tree.*

Chúng phục vụ dữ liệu 1 chiều.  
*They serve one-dimensional data.*

Chúng gãy với vector nhiều chiều.  
*They break down on high-dimensional vectors.*

Sau đó ta học hai index chuyên cho vector.  
*Next we study two indexes built for vectors.*

Hai index đó là **IVFFlat** và **HNSW**.  
*Those two indexes are **IVFFlat** and **HNSW**.*

Ta cũng học cách tune tham số của chúng.  
*We also learn how to tune their parameters.*

**Vấn đề nó giải quyết / The problem it solves**

Bạn cần tìm 1 vector giống nhất trong 100 triệu vector.  
*You need the single closest vector among 100 million vectors.*

Cách brute-force so từng cái một.  
*The brute-force way compares them one by one.*

Chi phí của nó là `O(n)`.  
*Its cost is `O(n)`.*

Cách đó bất khả thi trong real-time.  
*That approach is infeasible in real time.*

Index approximate nearest neighbor (ANN) — láng giềng gần nhất xấp xỉ — kéo chi phí về gần `O(log n)`.  
*An approximate nearest neighbor (ANN) index brings the cost near `O(log n)`.*

ANN đánh đổi một chút độ chính xác.  
*ANN trades away a little accuracy.*

Độ chính xác đó gọi là recall — độ bao phủ kết quả đúng.  
*That accuracy measure is called recall.*

Đổi lại tốc độ tăng gấp hàng nghìn lần.  
*In exchange the speed rises a thousandfold.*

**Học xong bạn sẽ làm được / What you will be able to do**

Bạn định nghĩa được database index.  
*You can define a database index.*

Bạn tính được chi phí linear search `(n+1)/2`.  
*You can compute the linear search cost `(n+1)/2`.*

Bạn tính được chi phí binary search `O(log n)`.  
*You can compute the binary search cost `O(log n)`.*

Bạn giải thích được điểm gãy của B-tree với vector nhiều chiều.  
*You can explain why the B-tree breaks on high-dimensional vectors.*

Bạn giải thích được điểm gãy của binary search.  
*You can explain why binary search breaks too.*

Đây là mắt xích cốt lõi của cả bài.  
*This is the core link of the whole lesson.*

Bạn mô tả được cơ chế IVFFlat.  
*You can describe the IVFFlat mechanism.*

Cơ chế đó gồm centroid, list và probe.  
*That mechanism involves centroids, lists, and probes.*

Bạn mô tả được cơ chế HNSW.  
*You can describe the HNSW mechanism.*

HNSW là một graph nhiều tầng.  
*HNSW is a multi-layer graph.*

Bạn nêu được Big-O của cả hai.  
*You can state the Big-O of both.*

Bạn viết được câu lệnh tạo index.  
*You can write the index creation statement.*

Bạn tune được `lists` và `probes`.  
*You can tune `lists` and `probes`.*

Bạn tune được `m`, `ef_construction` và `ef_search`.  
*You can tune `m`, `ef_construction`, and `ef_search`.*

Bạn chọn được index đúng theo quy mô.  
*You can pick the right index for a given scale.*

Bạn phân tích được trade-off giữa recall, speed và memory.  
*You can analyze the trade-off among recall, speed, and memory.*

Bạn trả lời được câu hỏi system design.  
*You can answer a system design question.*

**Mạch basic → staff / The basic → staff track**

🟢 **Basic — nền tảng**

Phần này định nghĩa index.  
*This part defines the index.*

Phần này dùng analogy cuốn sách.  
*This part uses the book analogy.*

Phần này tính linear search `(n+1)/2`.  
*This part computes linear search at `(n+1)/2`.*

Phần này giới thiệu sorted list và binary search.  
*This part introduces the sorted list and binary search.*

Phần này chạy tay ví dụ tìm "Jack".  
*This part hand-traces the "Jack" lookup example.*

Phần này giới thiệu B-tree.  
*This part introduces the B-tree.*

🟡 **Intermediate — vận dụng**

Phần này giải thích điểm gãy của index kinh điển với vector.  
*This part explains why classic indexes break on vectors.*

Phần này giới thiệu ANN.  
*This part introduces ANN.*

Phần này mô tả IVFFlat với centroid, list và probe.  
*This part describes IVFFlat with centroids, lists, and probes.*

Phần này có code minh họa.  
*This part includes sample code.*

Phần này so sánh exact với approximate.  
*This part compares exact against approximate.*

Phần này liệt kê các lỗi thường gặp.  
*This part lists the common mistakes.*

🔴 **Advanced — chuyên sâu**

Phần này mổ xẻ internals của HNSW.  
*This part dissects the internals of HNSW.*

Phần này chạy ví dụ "tìm 12".  
*This part walks through the "find 12" example.*

Phần này so Big-O của IVFFlat, HNSW và brute-force.  
*This part compares the Big-O of IVFFlat, HNSW, and brute-force.*

Phần này bàn về curse of dimensionality.  
*This part discusses the curse of dimensionality.*

Phần này giới thiệu DiskANN.  
*This part introduces DiskANN.*

Phần này tune sâu các tham số.  
*This part tunes the parameters in depth.*

Phần này hướng dẫn tự implement.  
*This part guides you through a hand-rolled implementation.*

Phần này liệt kê edge case.  
*This part lists the edge cases.*

🟣 **Staff — tư duy hệ thống**

Phần này chọn index ở quy mô lớn.  
*This part picks indexes at large scale.*

Phần này bàn về build cost.  
*This part discusses build cost.*

Phần này bàn về recall monitoring.  
*This part discusses recall monitoring.*

Phần này bàn về memory.  
*This part discusses memory.*

Phần này chỉ ra khi nào exact tốt hơn.  
*This part shows when exact search is better.*

Phần này bàn về filtered search.  
*This part discusses filtered search.*

Phần này đưa câu hỏi system design.  
*This part poses a system design question.*

🎯 **Cheatsheet — chốt lại**

Phần này liệt kê keywords.  
*This part lists the keywords.*

Phần này liệt kê core concepts.  
*This part lists the core concepts.*

Phần này liệt kê code cần thuộc lòng.  
*This part lists the code you must memorize.*

Phần này liệt kê câu hỏi phỏng vấn.  
*This part lists the interview questions.*

Phần này có các one-liner.  
*This part includes the one-liners.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

### 1.1. Index là gì (định nghĩa + analogy)
*1.1. What an index is (definition + analogy)*

**Database index / Database index**

Database index là một cấu trúc dữ liệu.  
*A database index is a data structure.*

Nó tăng tốc việc tìm kiếm.  
*It speeds up searching.*

Nó tăng tốc việc truy xuất.  
*It speeds up retrieval.*

Nó chứa một bản sao đã tổ chức.  
*It holds an organized copy.*

Bản sao đó lấy từ một phần dữ liệu của bảng.  
*That copy comes from part of the table's data.*

Nó giúp định vị nhanh dòng cần tìm.  
*It locates the target row quickly.*

**Analogy — bài gốc dùng / Analogy — used by the reading**

Index giống mục lục tra cứu cuối một cuốn sách.  
*An index is like the lookup index at the back of a book.*

Bạn muốn tìm chủ đề "database".  
*You want to find the topic "database".*

Bạn không đọc cả cuốn sách.  
*You do not read the whole book.*

Bạn tra mục lục.  
*You check the index.*

Mục lục ghi "database → trang 12, 45".  
*The index says "database → pages 12, 45".*

Bạn nhảy thẳng tới trang đó.  
*You jump straight to those pages.*

Database index là "mục lục" của bảng.  
*A database index is the "book index" of a table.*

Ý tưởng cốt lõi rất đơn giản.  
*The core idea is very simple.*

Bạn trả một chút chi phí lưu trữ.  
*You pay a little storage cost.*

Bạn trả thêm chi phí cập nhật.  
*You also pay an update cost.*

Bạn đổi lấy tốc độ tìm kiếm khổng lồ.  
*You get an enormous search speedup in return.*

### 1.2. Vì sao index tiết kiệm thời gian — linear search tốn thế nào
*1.2. Why an index saves time — the cost of linear search*

Linear search — quét tuyến tính — so từng dòng một.  
*Linear search compares the rows one by one.*

Nó dừng lại tại dòng tìm thấy.  
*It stops at the matching row.*

Trung bình bạn phải đọc `(n + 1) / 2` dòng.  
*On average you read `(n + 1) / 2` rows.*

Gặp may, bạn thấy kết quả sớm.  
*If you are lucky, you hit the result early.*

Gặp xui, bạn thấy kết quả ở cuối.  
*If you are unlucky, the result sits at the end.*

Trường hợp tệ nhất tốn `n` dòng.  
*The worst case costs `n` rows.*

Kết quả nằm ở dòng cuối cùng.  
*The result lies in the very last row.*

Kết quả cũng có thể không tồn tại.  
*The result may also not exist at all.*

Với `n = 1.000.000`, trung bình bạn đọc **~500.000 dòng**.  
*With `n = 1,000,000`, you read about **500,000 rows** on average.*

Con số đó chỉ dành cho *một* lần tìm.  
*That number covers just *one* lookup.*

Hệ thống nhận hàng nghìn query mỗi giây.  
*The system takes thousands of queries per second.*

Hệ thống sẽ sập.  
*The system will collapse.*

Trên dữ liệu lớn, phương án "không index" là bất khả thi.  
*On large data, the "no index" option is infeasible.*

Summary bài gốc nhấn mạnh đúng ý này.  
*The reading's summary stresses exactly this point.*

Hiệu quả của index chỉ thấy rõ trên tập dữ liệu lớn.  
*The benefit of an index shows clearly only on large datasets.*

### 1.3. Ví dụ chạy tay — tìm "Jack" (bám ví dụ bài gốc)
*1.3. Hand-traced example — finding "Jack" (following the reading)*

**Không index, danh sách unsorted, "Jack" ở vị trí 20/20 / No index, unsorted list, "Jack" at position 20 of 20**

> Danh sách chạy Emily → James → Olivia → ...  
> *The list runs Emily → James → Olivia → ...*
>
> Bạn phải đọc cả 20 dòng.  
> *You must read all 20 rows.*
>
> "Jack" nằm ở dòng cuối.  
> *"Jack" sits in the last row.*
>
> Với danh sách chưa sắp xếp, không có cách nào nhanh hơn.  
> *For an unsorted list, no faster method exists.*

**Có sorted index theo bảng chữ cái, cộng binary search / With an alphabetical sorted index plus binary search**

Danh sách đã sort gồm Alexander, Amelia, Ava, Benjamin, Charlotte, Daniel, Emily, Emma, Grace, **Isabella**, Jack, James, Lucas, **Michael**, Mia... William.  
*The sorted list holds Alexander, Amelia, Ava, Benjamin, Charlotte, Daniel, Emily, Emma, Grace, **Isabella**, Jack, James, Lucas, **Michael**, Mia... William.*

Danh sách có 20 tên.  
*The list has 20 names.*

Binary search chia đôi liên tục.  
*Binary search halves the range again and again.*

**Bước 1 / Step 1**

Bạn chia đôi 20 tên.  
*You halve the 20 names.*

Tên đầu nửa sau là **Isabella**.  
*The first name of the second half is **Isabella**.*

Isabella bắt đầu bằng chữ I.  
*Isabella starts with the letter I.*

"Jack" bắt đầu bằng chữ J.  
*"Jack" starts with the letter J.*

Chữ J đứng sau chữ I.  
*The letter J comes after the letter I.*

Vậy "Jack" chắc chắn nằm ở nửa sau.  
*So "Jack" certainly lies in the second half.*

Nửa sau có 10 tên từ Isabella tới William.  
*The second half holds 10 names from Isabella to William.*

Bạn loại nửa đầu.  
*You drop the first half.*

Bạn vừa loại 10 tên trong 1 bước.  
*You just eliminated 10 names in one step.*

**Bước 2 / Step 2**

Bạn chia đôi 10 tên còn lại.  
*You halve the remaining 10 names.*

Điểm giữa là **Michael**.  
*The midpoint is **Michael**.*

Chữ J đứng trước chữ M.  
*The letter J comes before the letter M.*

Vậy "Jack" nằm ở nửa đầu.  
*So "Jack" lies in the first half.*

Nửa đầu gồm Isabella, Jack, James, Lucas và Mia.  
*The first half holds Isabella, Jack, James, Lucas, and Mia.*

Bạn loại nửa sau.  
*You drop the second half.*

**Bước 3 / Step 3**

Bạn chia đôi tiếp.  
*You halve the range again.*

Ranh giới nằm giữa Isabella và Jack.  
*The boundary falls between Isabella and Jack.*

"Jack" đứng sau Isabella.  
*"Jack" comes after Isabella.*

Phần còn lại gồm Jack, James, Lucas và Mia.  
*The remainder holds Jack, James, Lucas, and Mia.*

**Bước 4 / Step 4**

Bạn lấy điểm giữa.  
*You take the midpoint.*

Bạn chạm đúng **Jack**. ✅  
*You land exactly on **Jack**. ✅*

Bạn tốn 4 bước.  
*You spent 4 steps.*

Cách cũ tốn 20 lần đọc.  
*The old way cost 20 reads.*

Với `n` phần tử, binary search cần **~log₂(n)** bước.  
*With `n` items, binary search needs about **log₂(n)** steps.*

20 tên cần khoảng 5 bước.  
*20 names need about 5 steps.*

**1 triệu** tên chỉ cần **~20 bước**.  
*One **million** names need only about **20 steps**.*

Lý do là 2²⁰ ≈ 1.048.576.  
*The reason is 2²⁰ ≈ 1,048,576.*

Đây là sức mạnh của `O(log n)`.  
*This is the power of `O(log n)`.*

### 1.4. Điều kiện tiên quyết của binary search — phải SẮP XẾP ĐƯỢC
*1.4. The precondition of binary search — the data must be SORTABLE*

Tên sắp thứ tự trên một trục.  
*Names are ordered along one axis.*

Trục đó là bảng chữ cái.  
*That axis is the alphabet.*

Nhờ vậy binary search chạy được.  
*Thanks to that, binary search works.*

Ở mỗi bước bạn trả lời một câu hỏi.  
*At each step you answer one question.*

Câu hỏi là "mục tiêu nằm trước hay sau điểm giữa?".  
*The question is "does the target sit before or after the midpoint?".*

Bạn trả lời được nhờ thứ tự tuyến tính.  
*You can answer it thanks to a linear ordering.*

Thứ tự tuyến tính có tên tiếng Anh là total order.  
*The English name for that linear ordering is total order.*

> **Ghi nhớ — mắt xích quan trọng nhất cả bài**  
> *Remember — the most important link in this lesson*
>
> Điều kiện "sắp xếp được trên một trục" sẽ vỡ.  
> *The "sortable along one axis" condition will break.*
>
> Nó vỡ với vector nhiều chiều.  
> *It breaks on high-dimensional vectors.*
>
> Bạn hãy giữ ý này trong đầu.  
> *Keep this idea in mind.*
>
> Phần 2 sẽ giải thích nguyên nhân.  
> *Part 2 will explain the reason.*
>
> Phần 2 cũng chỉ ra cách làm khác.  
> *Part 2 will also show the alternative approach.*

### 1.5. Index kinh điển trong DB = B-tree
*1.5. The classic index in a DB is the B-tree*

Trong RDBMS, index mặc định là **B-tree**.  
*In an RDBMS, the default index is the **B-tree**.*

B-tree là một cây cân bằng.  
*A B-tree is a balanced tree.*

Nó cho phép tìm ở `O(log n)`.  
*It supports lookups at `O(log n)`.*

Nó cho phép `range scan` ở `O(log n)`.  
*It supports a `range scan` at `O(log n)`.*

Nó giống một "binary search có cấu trúc cây".  
*It works like "binary search in tree form".*

Nó chạy trên dữ liệu sắp thứ tự được.  
*It runs on sortable data.*

Kiểu dữ liệu đó gồm số, chuỗi và ngày tháng.  
*Such data types include numbers, strings, and dates.*

Câu lệnh `CREATE INDEX ON users(email)` tạo index cho email.  
*The statement `CREATE INDEX ON users(email)` builds an index on email.*

Sau đó việc tra email cực nhanh.  
*After that, email lookups are extremely fast.*

```sql
-- Index kinh điển: B-tree trên cột sắp thứ tự được
-- Classic index: a B-tree on a sortable column
CREATE INDEX ON users (email);           -- tìm theo email: O(log n)
-- lookup by email: O(log n)
CREATE INDEX ON orders (created_at);     -- range: "đơn trong 7 ngày qua": nhanh
-- range: "orders in the last 7 days" is fast
```

B-tree tuyệt vời cho dữ liệu **1 chiều**.  
*The B-tree is excellent for **one-dimensional** data.*

Dữ liệu đó phải sắp thứ tự được.  
*That data must be sortable.*

Vector không có tính chất đó.  
*Vectors do not have that property.*

Vì vậy pgvector cần index khác.  
*For that reason pgvector needs a different index.*

### ✅ Self-check Phần 1
*✅ Self-check for Part 1*

Linear search trên 1 triệu dòng trung bình đọc bao nhiêu dòng?  
*How many rows does linear search read on average over one million rows?*

Binary search cần bao nhiêu bước?  
*How many steps does binary search need?*

Vì sao binary search bắt buộc dữ liệu phải sắp xếp?  
*Why does binary search require sorted data?*

B-tree hợp với kiểu dữ liệu nào?  
*Which data types suit a B-tree?*

Hãy cho một ví dụ cột nên đánh B-tree.  
*Give one example of a column that deserves a B-tree.*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

### 2.1. [MẮT XÍCH CỐT LÕI] Vì sao B-tree/binary search GÃY với vector
*2.1. [THE CORE LINK] Why B-tree and binary search BREAK on vectors*

Bài gốc nhảy thẳng từ binary search sang IVFFlat và HNSW.  
*The reading jumps straight from binary search to IVFFlat and HNSW.*

Bài gốc không giải thích nhu cầu về index mới.  
*The reading never explains the need for a new index.*

Chỗ này phân biệt người hiểu bản chất.  
*This point separates the people who truly understand.*

Chỗ này cũng lộ ra người học vẹt.  
*This point also exposes the rote learners.*

**Vấn đề 1 — vector không có thứ tự tuyến tính / Problem 1 — vectors have no linear ordering**

Bạn sort tên theo bảng chữ cái được.  
*You can sort names alphabetically.*

Một vector có dạng `[0.2, -0.7, 0.4, ...]`.  
*A vector looks like `[0.2, -0.7, 0.4, ...]`.*

Vector đó có 384 chiều.  
*That vector has 384 dimensions.*

Bạn sort nó theo chiều nào?  
*Along which dimension do you sort it?*

Giả sử bạn sort theo chiều 1.  
*Suppose you sort along dimension 1.*

Hai vector gần nhau về nghĩa có thể cách xa nhau ở chiều 1.  
*Two semantically close vectors may sit far apart on dimension 1.*

**Không tồn tại một trục duy nhất** cho mọi vector.  
***No single axis exists** for all vectors.*

Trên trục lý tưởng, "gần trên trục" phải bằng "gần thật sự".  
*On an ideal axis, "close on the axis" would equal "truly close".*

Binary search mất điều kiện tiên quyết ở mục 1.4.  
*Binary search loses the precondition from section 1.4.*

**Vấn đề 2 — câu hỏi khác hẳn / Problem 2 — the question itself differs**

B-tree trả lời câu hỏi về giá trị bằng đúng X.  
*A B-tree answers questions about an exact value X.*

B-tree cũng trả lời câu hỏi về khoảng [a,b].  
*A B-tree also answers questions about a range [a,b].*

Vector search hỏi một câu khác hẳn.  
*Vector search asks a completely different question.*

Câu hỏi là "vector nào GẦN NHẤT về khoảng cách".  
*The question is "which vector is CLOSEST by distance".*

Đây là bài toán nearest neighbor — láng giềng gần nhất.  
*This is the nearest neighbor problem.*

Đây không phải bài toán khớp giá trị.  
*This is not an exact-match problem.*

Đây cũng không phải bài toán khoảng.  
*This is not a range problem either.*

**Vấn đề 3 — curse of dimensionality / Problem 3 — the curse of dimensionality**

Người ta từng thử cây phân hoạch không gian cho bài toán NN.  
*People once tried space-partitioning trees for the NN problem.*

Cây đó tên là kd-tree.  
*That tree is called a kd-tree.*

Số chiều thực tế lên tới hàng trăm hoặc hàng nghìn.  
*Real dimensionality reaches hundreds or thousands.*

Ở số chiều lớn, mọi điểm gần như cách đều nhau.  
*At high dimensionality, all points sit nearly equidistant.*

Cây phải duyệt gần hết dữ liệu.  
*The tree must traverse almost all the data.*

Lúc đó nó không nhanh hơn brute-force.  
*At that point it is no faster than brute-force.*

Phần 3 sẽ đào sâu điểm này.  
*Part 3 will dig into this point.*

**Kết luận / Conclusion**

Với vector nhiều chiều, ta từ bỏ tìm chính xác tuyệt đối.  
*For high-dimensional vectors, we give up perfectly exact search.*

Ta chuyển sang **Approximate Nearest Neighbor (ANN)**.  
*We move to **Approximate Nearest Neighbor (ANN)**.*

Ta chấp nhận bỏ sót vài hàng xóm thật.  
*We accept missing a few true neighbors.*

Đổi lại ta có tốc độ.  
*In return we gain speed.*

IVFFlat và HNSW là hai cách làm ANN.  
*IVFFlat and HNSW are two ways to do ANN.*

### 2.2. IVFFlat — "chia cụm rồi chỉ tìm trong vài cụm"
*2.2. IVFFlat — "cluster first, then search only a few clusters"*

IVFFlat viết đầy đủ là **Inverted File with Flat compression**.  
*IVFFlat stands for **Inverted File with Flat compression**.*

Cơ chế dưới đây bám bài gốc.  
*The mechanism below follows the reading.*

Bài gốc minh họa bằng hình 3 centroid.  
*The reading illustrates it with a 3-centroid figure.*

**1. Build — dựng index / 1. Build**

Bạn chạy một thuật toán clustering.  
*You run a clustering algorithm.*

Thuật toán đó là k-means.  
*That algorithm is k-means.*

Nó chia toàn bộ vector thành `n` **list**.  
*It splits all vectors into `n` **lists**.*

Mỗi list là một cụm.  
*Each list is one cluster.*

Cụm nằm quanh một **centroid** — tâm cụm.  
*The cluster forms around a **centroid**.*

**2. Search — tìm kiếm / 2. Search**

Bạn so query với các centroid.  
*You compare the query against the centroids.*

Bạn chọn centroid gần nhất.  
*You pick the nearest centroid.*

Bạn tìm brute-force bên trong cụm của centroid đó.  
*You brute-force search inside that centroid's cluster.*

Bạn không quét cả triệu vector.  
*You do not scan the whole million vectors.*

Trong hình bài gốc, Search Vector có màu tím.  
*In the reading's figure, the Search Vector is purple.*

Hệ thống so nó với Centroid 1, 2 và 3.  
*The system compares it against Centroid 1, 2, and 3.*

Nó gần **Centroid 3** nhất.  
*It is closest to **Centroid 3**.*

Hệ thống chỉ lục trong cụm màu xanh lá.  
*The system searches only the green cluster.*

**Hai tham số phải hiểu / Two parameters you must understand**

`lists` là số cụm.  
*`lists` is the number of clusters.*

Bài gốc và pgvector gợi ý công thức `rows/1000`.  
*The reading and pgvector suggest the formula `rows/1000`.*

Công thức đó dùng cho bảng ≤ 1 triệu rows.  
*That formula applies to tables with ≤ 1 million rows.*

1 triệu vector cho ra 1000 list.  
*One million vectors yield 1000 lists.*

Mỗi list chứa khoảng 1000 phần tử.  
*Each list holds about 1000 items.*

Trên 1 triệu rows, bạn dùng `sqrt(rows)`.  
*Above one million rows, you use `sqrt(rows)`.*

`probes` là số cụm dò khi search.  
*`probes` is the number of clusters probed during a search.*

Giá trị mặc định là 1.  
*The default value is 1.*

Giá trị cao hơn xét thêm cụm lân cận.  
*A higher value examines more neighboring clusters.*

Recall khi đó tốt hơn.  
*Recall then improves.*

Tốc độ khi đó chậm hơn.  
*Speed then drops.*

Giá trị khởi đầu gợi ý là `sqrt(lists)`.  
*The suggested starting value is `sqrt(lists)`.*

**Code — bám bài gốc / Code — following the reading**

```sql
-- Tạo IVFFlat index. LƯU Ý: phải tạo SAU khi bảng đã có dữ liệu (cần data để k-means).
-- Create the IVFFlat index. NOTE: build it AFTER the table has data, since k-means needs data.
CREATE INDEX ON sentence USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Khi search, đặt số cụm dò. probes cao = recall cao, tốc độ giảm.
-- At search time, set the probe count. Higher probes means higher recall and lower speed.
SET ivfflat.probes = 32;

SELECT * FROM sentence ORDER BY embedding <=> '[...]' LIMIT 10;
```

**Điểm yếu IVFFlat / The weakness of IVFFlat**

Một hàng xóm thật có thể nằm **sát ranh giới cụm**.  
*A true neighbor may sit **right at a cluster boundary**.*

Hàng xóm đó dễ bị bỏ sót.  
*That neighbor is easily missed.*

Nó nằm ở cụm bên cạnh.  
*It lives in the adjacent cluster.*

`probes` không dò tới cụm đó.  
*`probes` never reaches that cluster.*

IVFFlat còn có tính tĩnh.  
*IVFFlat is also static.*

Bạn thêm nhiều dữ liệu mới sau khi build.  
*You add a lot of new data after the build.*

Phân hoạch cụm khi đó bị lệch.  
*The cluster partitioning then goes off balance.*

Recall tụt xuống.  
*Recall drops.*

Bạn phải chạy `REINDEX`.  
*You must run `REINDEX`.*

### 2.3. Exact vs Approximate — điều junior hay hiểu nhầm
*2.3. Exact vs approximate — the point juniors misread*

**Không index → exact search / No index → exact search**

Không có index, pgvector chạy exact search — tìm chính xác.  
*Without an index, pgvector runs an exact search.*

pgvector quét hết bảng.  
*pgvector scans the entire table.*

Nó tính distance cho từng vector.  
*It computes the distance for every vector.*

Recall đạt 100%.  
*Recall reaches 100%.*

Kết quả luôn đúng.  
*The result is always correct.*

Chi phí là `O(n)`.  
*The cost is `O(n)`.*

Cách này chậm trên dữ liệu lớn.  
*This approach is slow on large data.*

**Có ANN index → approximate search / With an ANN index → approximate search**

Có ANN index, pgvector chạy approximate search — tìm xấp xỉ.  
*With an ANN index, pgvector runs an approximate search.*

Cách này nhanh.  
*This approach is fast.*

Chi phí gần `~log n` hoặc tính theo cụm.  
*The cost is near `~log n` or scales by cluster.*

Recall thấp hơn 100%.  
*Recall falls below 100%.*

Kết quả có thể thiếu vài hàng xóm thật.  
*The result may miss a few true neighbors.*

Đây là thiết kế có chủ đích.  
*This is by design.*

Đây không phải bug.  
*This is not a bug.*

> Summary bài gốc nói đúng.  
> *The reading's summary gets this right.*
>
> Mỗi phương pháp có điểm mạnh riêng.  
> *Each method has its own strength.*
>
> Mỗi phương pháp cũng có trade-off — đánh đổi riêng.  
> *Each method also carries its own trade-off.*
>
> Bạn chọn index và tune tham số.  
> *You pick an index and tune the parameters.*
>
> Việc đó cân bằng accuracy, speed và resource.  
> *That work balances accuracy, speed, and resources.*
>
> Câu này là linh hồn của cả bài.  
> *This sentence is the soul of the whole lesson.*

### 2.4. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp
*2.4. [BEYOND THE ORIGINAL READING] — 3 common mistakes*

**Lỗi 1 — đánh index quá sớm trên dữ liệu nhỏ / Mistake 1 — indexing too early on small data**

Bạn có vài nghìn tới vài chục nghìn vector.  
*You have a few thousand to a few tens of thousands of vectors.*

Brute-force exact đã đủ nhanh.  
*Exact brute-force is already fast enough.*

Nó cho recall 100%.  
*It delivers 100% recall.*

ANN index lúc này giảm độ chính xác.  
*An ANN index at this stage lowers accuracy.*

Nó không lợi tốc độ đáng kể.  
*It brings no meaningful speed gain.*

Index chỉ đáng giá trên dữ liệu đủ lớn.  
*An index pays off only on sufficiently large data.*

Summary bài gốc nói cùng ý đó.  
*The reading's summary states the same point.*

**Lỗi 2 — tạo IVFFlat khi bảng còn rỗng / Mistake 2 — creating IVFFlat on an empty table**

IVFFlat cần dữ liệu cho k-means.  
*IVFFlat needs data for k-means.*

k-means dùng dữ liệu để tìm centroid.  
*k-means uses the data to locate centroids.*

Bảng rỗng cho ra phân cụm vô nghĩa.  
*An empty table produces meaningless clusters.*

Recall khi đó rất tệ.  
*Recall then turns out terrible.*

HNSW khác IVFFlat ở điểm này.  
*HNSW differs from IVFFlat on this point.*

Bạn tạo được HNSW cả khi bảng rỗng.  
*You can build HNSW even on an empty table.*

**Lỗi 3 — sai ops class so với toán tử query / Mistake 3 — ops class mismatched with the query operator**

Bạn tạo index `vector_cosine_ops`.  
*You create a `vector_cosine_ops` index.*

Bạn query bằng toán tử `<->`.  
*You query with the `<->` operator.*

Toán tử `<->` là L2.  
*The `<->` operator means L2.*

Postgres sẽ **bỏ qua index**.  
*Postgres will **ignore the index**.*

Query rơi về seq scan.  
*The query falls back to a seq scan.*

Ops class phải khớp toán tử.  
*The ops class must match the operator.*

`vector_cosine_ops` đi với `<=>`.  
*`vector_cosine_ops` pairs with `<=>`.*

`vector_l2_ops` đi với `<->`.  
*`vector_l2_ops` pairs with `<->`.*

`vector_ip_ops` đi với `<#>`.  
*`vector_ip_ops` pairs with `<#>`.*

Bạn kiểm tra bằng `EXPLAIN ANALYZE`.  
*You verify it with `EXPLAIN ANALYZE`.*

### ✅ Self-check Phần 2
*✅ Self-check for Part 2*

Hãy nêu 2 lý do binary search không tìm được nearest neighbor của vector.  
*Give 2 reasons binary search cannot find the nearest neighbor of a vector.*

Hãy nêu 2 lý do tương tự cho B-tree.  
*Give 2 similar reasons for the B-tree.*

Trong IVFFlat, `lists` điều khiển cái gì?  
*In IVFFlat, what does `lists` control?*

Trong IVFFlat, `probes` điều khiển cái gì?  
*In IVFFlat, what does `probes` control?*

Tăng `probes` thì bạn được gì và mất gì?  
*What do you gain and lose by raising `probes`?*

Vì sao bạn không nên tạo IVFFlat index khi bảng còn rỗng?  
*Why should you avoid creating an IVFFlat index on an empty table?*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED (In depth)*

### 3.1. HNSW — graph nhiều tầng "đường cao tốc"
*3.1. HNSW — the multi-layer "highway" graph*

HNSW viết đầy đủ là **Hierarchical Navigable Small World**.  
*HNSW stands for **Hierarchical Navigable Small World**.*

Cơ chế của nó như sau.  
*Its mechanism works as follows.*

**Cấu trúc graph / The graph structure**

Bạn xây một **graph nhiều tầng**.  
*You build a **multi-layer graph**.*

Mỗi node là một vector.  
*Each node is one vector.*

Cạnh nối một node với các hàng xóm gần nó.  
*An edge links a node to its nearby neighbors.*

Độ gần tính theo distance metric đã chọn.  
*Closeness follows the chosen distance metric.*

Metric đó có thể là cosine, L2 hoặc ip.  
*That metric may be cosine, L2, or ip.*

Reading viết nhầm thành Euclidean.  
*The reading mistakenly writes Euclidean.*

**Tầng trên cùng / The top layer**

Tầng trên cùng có ít node.  
*The top layer holds few nodes.*

Cạnh ở tầng này là "đường dài".  
*Edges on this layer are "long roads".*

Chúng nhảy xa và nhanh.  
*They jump far and fast.*

Chúng đưa bạn về đúng vùng.  
*They carry you into the right region.*

**Tầng dưới cùng / The bottom layer**

Tầng dưới cùng chứa mọi node.  
*The bottom layer holds every node.*

Cạnh ở tầng này là "đường ngắn".  
*Edges on this layer are "short roads".*

Chúng tinh chỉnh tới hàng xóm chính xác.  
*They refine the path to the exact neighbor.*

**Search / Search**

Search bắt đầu ở tầng trên.  
*The search starts at the top layer.*

Nó đi tới node gần query nhất.  
*It moves to the node closest to the query.*

Cách đi đó gọi là tham lam hay greedy.  
*That movement is called greedy.*

Sau đó nó tụt xuống tầng dưới.  
*Then it drops to the layer below.*

Nó lặp lại quá trình tới tầng đáy.  
*It repeats the process down to the bottom layer.*

**Ví dụ "tìm 12" của reading / The reading's "find 12" example**

Reading có một ví dụ tên "tìm 12".  
*The reading gives an example called "find 12".*

Tôi diễn giải lại ví dụ đó.  
*I restate that example here.*

Tôi cũng sửa nhãn tầng cho đúng.  
*I also fix the layer labels.*

Danh sách ở tầng đáy là 2, 4, 8, 12, 14, 16, 19.  
*The bottom-layer list is 2, 4, 8, 12, 14, 16, 19.*

Layer 3 là tầng đỉnh.  
*Layer 3 is the top layer.*

Tầng này chỉ có node 4.  
*This layer holds only node 4.*

Từ entry point bạn nhảy tới 4.  
*From the entry point you jump to 4.*

Layer 2 có node 4 và 16.  
*Layer 2 holds node 4 and node 16.*

Số 12 nằm giữa hai node đó.  
*The number 12 lies between those two nodes.*

Bạn đi về phía phù hợp.  
*You move toward the correct side.*

Layer 1 có node 4, 12 và 16.  
*Layer 1 holds node 4, 12, and 16.*

Bạn thấy node 12.  
*You spot node 12.*

Layer 0 là tầng đáy.  
*Layer 0 is the bottom layer.*

Bạn dừng lại ở **12**. ✅  
*You stop at **12**. ✅*

**Analogy / Analogy**

Analogy ở đây là bản đồ giao thông.  
*The analogy here is a transport map.*

Tuyến bay quốc tế là tầng cao.  
*The international flight is the high layer.*

Nó đưa bạn về đúng thành phố.  
*It brings you to the right city.*

Taxi là tầng thấp.  
*The taxi is the low layer.*

Nó đưa bạn tới đúng địa chỉ.  
*It brings you to the exact address.*

Nhờ vậy search tốn khoảng **`O(log n)`**.  
*Thanks to this, the search costs about **`O(log n)`**.*

**Code — bám bài gốc / Code — following the reading**

```sql
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**Tham số `m` / The `m` parameter**

`m` là số cạnh tối đa của mỗi node ở mỗi tầng.  
*`m` is the maximum number of edges per node on each layer.*

Giá trị cao cho recall tốt hơn.  
*A high value gives better recall.*

Graph khi đó to hơn.  
*The graph then grows larger.*

Chi phí build tăng lên.  
*The build cost rises.*

Mức RAM cũng tăng lên.  
*RAM usage rises too.*

**Tham số `ef_construction` / The `ef_construction` parameter**

`ef_construction` là số ứng viên hàng xóm xét khi xây graph.  
*`ef_construction` is the number of neighbor candidates examined during the build.*

Reading dùng 64 ứng viên cho mỗi lần chèn.  
*The reading uses 64 candidates per insertion.*

Giá trị cao cho graph chất lượng hơn.  
*A high value gives a better quality graph.*

Recall khi đó tốt hơn.  
*Recall then improves.*

Việc build khi đó **chậm hơn**.  
*The build then runs **slower**.*

**Tham số `ef_search` / The `ef_search` parameter**

`ef_search` là tham số query-time — tham số lúc chạy query.  
*`ef_search` is a query-time parameter.*

Reading không nhắc tới nó.  
*The reading never mentions it.*

Nó là số ứng viên xét khi tìm.  
*It is the number of candidates examined during a search.*

Giá trị mặc định là 40.  
*The default value is 40.*

Giá trị cao cho recall tốt hơn.  
*A high value gives better recall.*

Tốc độ khi đó chậm hơn.  
*Speed then drops.*

Đây là núm xoay recall–speed quan trọng nhất lúc chạy.  
*This is the most important recall–speed knob at runtime.*

```sql
SET hnsw.ef_search = 100;   -- tăng recall khi cần
-- raise recall when you need it
```

### 3.2. Big-O: brute-force vs IVFFlat vs HNSW
*3.2. Big-O: brute-force vs IVFFlat vs HNSW*

| Phương pháp / Method | Query complexity / Query complexity | Build / Build | Memory / Memory | Cần data để build? / Needs data to build? | Data động / Dynamic data |
|---|---|---|---|---|---|
| Brute-force, không index / Brute-force, no index | `O(n·d)` | 0 | thấp / low | — | tốt, recall 100% / good, 100% recall |
| **IVFFlat** | ~`O(lists·d + (n/lists)·probes·d)` | **nhanh / fast** | **thấp / low** | **có, cần k-means / yes, k-means needs it** | kém, lệch cụm / poor, clusters drift |
| **HNSW** | ~**`O(log n · d)`** | chậm / slow | cao, graph nằm in-RAM / high, graph held in RAM | không / no | **tốt / good** |

Ở đây `n` là số vector.  
*Here `n` is the number of vectors.*

Ở đây `d` là số chiều.  
*Here `d` is the number of dimensions.*

**Chốt lại / The takeaway**

HNSW nhanh nhất.  
*HNSW is the fastest.*

HNSW cho recall cao nhất.  
*HNSW gives the highest recall.*

HNSW tốn nhiều RAM.  
*HNSW consumes a lot of RAM.*

HNSW build lâu.  
*HNSW takes long to build.*

IVFFlat nhẹ.  
*IVFFlat is lightweight.*

IVFFlat build nhanh.  
*IVFFlat builds fast.*

IVFFlat cho recall kém hơn.  
*IVFFlat gives lower recall.*

IVFFlat ngại data động.  
*IVFFlat copes poorly with dynamic data.*

Người phỏng vấn hay hỏi bảng này.  
*Interviewers ask about this table often.*

### 3.3. Curse of dimensionality — vì sao NN "khó" ở chiều cao
*3.3. The curse of dimensionality — why NN is "hard" in high dimensions*

Ở 2D và 3D, việc chia không gian thành ô rất hiệu quả.  
*In 2D and 3D, splitting space into cells works very well.*

Cấu trúc đó là kd-tree.  
*That structure is the kd-tree.*

Số chiều `d` có thể lên hàng trăm hoặc hàng nghìn.  
*The dimension count `d` may reach hundreds or thousands.*

Khi đó ba vấn đề xuất hiện.  
*Three problems then appear.*

Thể tích không gian tăng theo hàm mũ.  
*The volume of the space grows exponentially.*

Dữ liệu trở nên *thưa*.  
*The data becomes *sparse*.*

Khoảng cách tới điểm gần nhất và điểm xa nhất co lại gần nhau.  
*The distances to the nearest and farthest points converge.*

Khái niệm "gần nhất" mất ý nghĩa phân biệt.  
*The notion of "nearest" loses its discriminating power.*

Cây phân hoạch phải duyệt gần hết nhánh.  
*The partition tree must traverse nearly every branch.*

Chi phí sập về `O(n)`.  
*The cost collapses back to `O(n)`.*

Vì vậy ta cần cách tiếp cận **approximate**.  
*For that reason we need an **approximate** approach.*

Ta cần cấu trúc kiểu graph.  
*We need a graph-style structure.*

HNSW là ví dụ của cấu trúc graph.  
*HNSW is an example of a graph structure.*

Ta cũng có thể dùng cấu trúc cụm.  
*We may also use a cluster structure.*

IVFFlat là ví dụ của cấu trúc cụm.  
*IVFFlat is an example of a cluster structure.*

Ta không dùng cây chính xác.  
*We do not use an exact tree.*

Bạn nêu được "curse of dimensionality" khi phỏng vấn.  
*You can name the "curse of dimensionality" in an interview.*

Đó là một điểm cộng lớn.  
*That is a big plus.*

### 3.4. [MỞ RỘNG] DiskANN — "người thứ ba" ngoài bài gốc
*3.4. [EXTENSION] DiskANN — the "third player" beyond the reading*

Bài gốc chỉ nêu IVFFlat và HNSW.  
*The reading covers only IVFFlat and HNSW.*

Ở tầm 2026, ta có thêm **DiskANN**.  
*As of 2026, we also have **DiskANN**.*

Nó đến qua extension `pgvectorscale` của Timescale.  
*It arrives through Timescale's `pgvectorscale` extension.*

DiskANN giữ phần lớn graph trên **SSD**.  
*DiskANN keeps most of the graph on **SSD**.*

Nó không giữ graph trên RAM.  
*It does not keep the graph in RAM.*

Nó dùng thêm binary quantization — lượng tử hóa nhị phân.  
*It also applies binary quantization.*

Index của nó nhỏ hơn HNSW nhiều lần.  
*Its index is many times smaller than an HNSW index.*

Nó hỗ trợ vector tới 16000 chiều native.  
*It natively supports vectors up to 16000 dimensions.*

HNSW và IVFFlat của pgvector giới hạn **2000 chiều** khi index.  
*The pgvector HNSW and IVFFlat indexes cap out at **2000 dimensions**.*

Đây là một cú vấp kinh điển.  
*This is a classic stumbling block.*

Model OpenAI 3-large có 3072 chiều.  
*The OpenAI 3-large model has 3072 dimensions.*

Workaround là kiểu dữ liệu `halfvec`.  
*The workaround is the `halfvec` type.*

Bạn biết DiskANN.  
*You know DiskANN.*

Bạn biết giới hạn 2000 chiều.  
*You know the 2000-dimension limit.*

Đó là dấu hiệu bạn cập nhật kiến thức.  
*That signals you keep your knowledge current.*

### 3.5. Tự implement để hiểu tận gốc
*3.5. Implement it yourself to understand it fully*

**(a) Binary search — nền tảng bài gốc / (a) Binary search — the reading's foundation**

```python
def binary_search(sorted_names, target):
    lo, hi = 0, len(sorted_names) - 1
    steps = 0
    while lo <= hi:
        steps += 1
        mid = (lo + hi) // 2
        if sorted_names[mid] == target:
            return mid, steps                 # tìm thấy + số bước
            # found it, plus the step count
        elif sorted_names[mid] < target:
            lo = mid + 1                       # loại nửa trái
            # drop the left half
        else:
            hi = mid - 1                       # loại nửa phải
            # drop the right half
    return -1, steps

names = sorted(["Emily","James","Olivia","Jack","Isabella","Michael","Mia","William"])
print(binary_search(names, "Jack"))            # ~3 bước thay vì quét tuyến tính
# about 3 steps instead of a linear scan
```

**(b) IVF mini — thấy trade-off `probes` / (b) A mini IVF — see the `probes` trade-off**

```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def build(self, corpus, n_lists=16):
        self.corpus = corpus
        km = KMeans(n_clusters=n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_     # tâm mỗi cụm
        # the centroid of each cluster
        self.assign = km.labels_                 # vector -> cụm nào
        # which cluster each vector belongs to
        return self

    def search(self, q, k=5, probes=2):
        # 1) chọn `probes` cụm gần query (thay vì quét cả corpus)
        # 1) pick the `probes` clusters nearest the query, instead of scanning the corpus
        near = np.argsort(np.linalg.norm(self.centroids - q, axis=1))[:probes]
        # 2) brute-force chỉ trong các cụm đã chọn
        # 2) brute-force only inside the chosen clusters
        cand = np.where(np.isin(self.assign, near))[0]
        d = np.linalg.norm(self.corpus[cand] - q, axis=1)
        top = np.argsort(d)[:k]
        return cand[top], d[top]
# Chạy với probes=1 vs probes=8 để tự thấy: probes nhỏ nhanh nhưng đôi khi trượt.
# Run it with probes=1 and probes=8 to see it yourself.
# A small probes value is fast. It sometimes misses the true neighbor.
```

### 3.6. Edge cases
*3.6. Edge cases*

**Giới hạn 2000 chiều / The 2000-dimension limit**

Index HNSW và IVFFlat có giới hạn **2000 chiều**.  
*The HNSW and IVFFlat indexes cap out at **2000 dimensions**.*

Với model chiều cao, bạn dùng `halfvec`.  
*For high-dimensional models, you use `halfvec`.*

`halfvec` hỗ trợ tới khoảng 4000 chiều.  
*`halfvec` supports up to about 4000 dimensions.*

Bạn cũng có thể dùng DiskANN.  
*You may also use DiskANN.*

**IVFFlat sau khi nạp thêm nhiều data / IVFFlat after loading a lot of new data**

Bạn nạp thêm nhiều data vào bảng.  
*You load a lot of new data into the table.*

Centroid bị lệch.  
*The centroids drift.*

Recall tụt xuống.  
*Recall drops.*

Bạn phải chạy `REINDEX`.  
*You must run `REINDEX`.*

**VACUUM trên HNSW / VACUUM on HNSW**

Lệnh `VACUUM` trên HNSW rất chậm.  
*The `VACUUM` command runs very slowly on HNSW.*

Bạn nên reindex trước.  
*You should reindex first.*

Sau đó bạn mới vacuum.  
*Only then do you vacuum.*

**Recall âm thầm tụt / Recall drops silently**

Tham số quá thấp làm recall âm thầm tụt.  
*Parameters set too low make recall drop silently.*

Bạn phải *đo* recall.  
*You must *measure* recall.*

Phần 4 hướng dẫn cách đo.  
*Part 4 shows how to measure it.*

Bạn không nên đoán.  
*You should not guess.*

**Sai ops class / A wrong ops class**

Sai ops class gây hậu quả nặng.  
*A wrong ops class carries heavy consequences.*

Postgres bỏ qua index.  
*Postgres skips the index.*

Query rơi về seq scan một cách âm thầm.  
*The query falls back to a seq scan silently.*

### ✅ Self-check Phần 3
*✅ Self-check for Part 3*

HNSW search tốn khoảng `O(?)`.  
*An HNSW search costs about `O(?)`.*

Vì sao "graph nhiều tầng" cho tốc độ đó?  
*Why does the "multi-layer graph" deliver that speed?*

Hãy giải thích bằng analogy.  
*Explain it with an analogy.*

`ef_construction` chạy ở build-time.  
*`ef_construction` acts at build time.*

`ef_search` chạy ở query-time.  
*`ef_search` acts at query time.*

Hai tham số này khác nhau chỗ nào?  
*Where do these two parameters differ?*

"Curse of dimensionality" làm hỏng kd-tree ở chiều cao.  
*The "curse of dimensionality" breaks the kd-tree in high dimensions.*

Nó hỏng theo cơ chế nào?  
*By what mechanism does it break?*

Câu trả lời cũng áp dụng cho mọi exact tree.  
*The same answer applies to every exact tree.*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL (Systems thinking & technical leadership)*

### 4.1. Chọn index theo quy mô — quyết định thật ở tầm staff
*4.1. Choosing an index by scale — the real staff-level decision*

Câu hỏi không phải "IVFFlat hay HNSW" một cách trừu tượng.  
*The question is not "IVFFlat or HNSW" in the abstract.*

Câu hỏi thật nói về quy mô CỦA TÔI.  
*The real question is about MY scale.*

Câu hỏi thật nói về ràng buộc CỦA TÔI.  
*The real question is about MY constraints.*

Index nào hợp với hai thứ đó?  
*Which index fits those two things?*

**Dữ liệu nhỏ — dưới khoảng 10K tới 50K vector / Small data — under roughly 10K to 50K vectors**

Bạn **đừng đánh index**.  
*You should **not build an index**.*

Brute-force exact đủ nhanh.  
*Exact brute-force is fast enough.*

Recall đạt 100%.  
*Recall reaches 100%.*

Bạn không phải lo việc tune.  
*You have no tuning to worry about.*

Đánh index sớm chỉ tự chuốc recall thấp.  
*Indexing early only buys you lower recall.*

Summary bài gốc nói cùng ý đó.  
*The reading's summary makes the same point.*

Hiệu quả index chỉ thấy rõ trên tập lớn.  
*The benefit of an index shows only on large datasets.*

**Quy mô trung bình — tới khoảng vài triệu vector / Medium scale — up to a few million vectors**

**HNSW** là lựa chọn mặc định.  
***HNSW** is the default choice.*

Nó cho recall cao.  
*It gives high recall.*

Nó cho latency thấp.  
*It gives low latency.*

Nó chịu data động tốt.  
*It handles dynamic data well.*

Bạn chấp nhận thời gian build lâu.  
*You accept a long build time.*

Bạn chấp nhận mức RAM cao.  
*You accept high RAM usage.*

**RAM hạn chế, build lại thường xuyên, dữ liệu khá tĩnh / Tight RAM, frequent rebuilds, fairly static data**

Bạn chọn **IVFFlat**.  
*You pick **IVFFlat**.*

IVFFlat nhẹ.  
*IVFFlat is lightweight.*

IVFFlat build nhanh.  
*IVFFlat builds fast.*

Với dataset khổng lồ, bạn chọn **DiskANN**.  
*For a huge dataset, you pick **DiskANN**.*

DiskANN chạy trên nền SSD.  
*DiskANN runs on SSD.*

**Vector trên 2000 chiều / Vectors above 2000 dimensions**

Bạn dùng `halfvec` kèm HNSW.  
*You use `halfvec` together with HNSW.*

Bạn cũng có thể dùng DiskANN.  
*You may also use DiskANN.*

### 4.2. Build cost, memory, và bottleneck vận hành
*4.2. Build cost, memory, and operational bottlenecks*

**Build time — thời gian dựng index / Build time**

HNSW build ở mức `O(n log n)`.  
*An HNSW build costs `O(n log n)`.*

Ở quy mô lớn, việc build tốn hàng giờ.  
*At large scale, the build takes hours.*

Bạn nạp hết data trước.  
*You load all the data first.*

Bạn build index sau.  
*You build the index afterwards.*

Bạn tăng `maintenance_work_mem`.  
*You raise `maintenance_work_mem`.*

Giá trị đó nên ở mức 50–60% RAM trở xuống.  
*That value should stay at or below 50–60% of RAM.*

Bạn tăng `max_parallel_maintenance_workers`.  
*You raise `max_parallel_maintenance_workers`.*

Bạn dùng **`CREATE INDEX CONCURRENTLY`**.  
*You use **`CREATE INDEX CONCURRENTLY`**.*

Lệnh đó không khóa write.  
*That command does not lock writes.*

**Memory — bộ nhớ / Memory**

HNSW graph phải nằm gọn trong RAM.  
*The HNSW graph must fit entirely in RAM.*

Nhờ vậy nó mới nhanh.  
*Only then does it stay fast.*

Phép tính là 100M vector × 1536 chiều × 4 byte.  
*The arithmetic is 100M vectors × 1536 dimensions × 4 bytes.*

Kết quả xấp xỉ 600GB.  
*The result is roughly 600GB.*

Con số đó *chỉ* tính raw vectors.  
*That figure covers the raw vectors *only*.*

Graph còn có overhead riêng.  
*The graph carries its own overhead on top.*

RAM là ràng buộc số một.  
*RAM is the number one constraint.*

Bạn cân nhắc **quantization** — lượng tử hóa.  
*You consider **quantization**.*

Lựa chọn gồm halfvec và binary.  
*The options include halfvec and binary.*

Bạn cũng có thể cân nhắc DiskANN.  
*You may also consider DiskANN.*

**Data động — dữ liệu thay đổi liên tục / Dynamic data**

IVFFlat lệch cụm khi bạn ghi nhiều.  
*IVFFlat clusters drift when you write a lot.*

Bạn cần một lịch reindex.  
*You need a reindex schedule.*

HNSW chịu insert tốt hơn.  
*HNSW handles inserts better.*

`VACUUM` trên HNSW lại chậm.  
*`VACUUM` on HNSW is slow, however.*

### 4.3. Monitoring recall — thứ phân biệt staff
*4.3. Monitoring recall — what sets a staff engineer apart*

ANN không có kết quả "đúng/sai" nhị phân.  
*ANN has no binary "right or wrong" outcome.*

Bạn phải **đo recall** định kỳ.  
*You must **measure recall** on a regular schedule.*

Bạn so kết quả có index với kết quả exact.  
*You compare the indexed result against the exact result.*

Kết quả exact là ground truth — chuẩn vàng.  
*The exact result is the ground truth.*

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact search làm chuẩn vàng
-- force an exact search as the gold standard
SELECT id FROM items ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so overlap với kết quả có index -> recall@10; dashboard theo thời gian
-- compare the overlap with the indexed result to get recall@10
-- then track that number on a dashboard over time
```

Recall tụt là một tín hiệu.  
*Falling recall is a signal.*

Bạn tăng `ef_search`, `probes` hoặc `m`.  
*You raise `ef_search`, `probes`, or `m`.*

Bạn cũng có thể reindex.  
*You may also reindex.*

**Không đo recall là bay mù.**  
***Not measuring recall means flying blind.***

Chất lượng search có thể xuống dốc.  
*Search quality may decline steadily.*

Hệ thống không báo lỗi nào cả.  
*The system raises no error at all.*

### 4.4. Filtered search & overfiltering [MỞ RỘNG]
*4.4. Filtered search & overfiltering [EXTENSION]*

Bạn kết hợp ANN với mệnh đề `WHERE`.  
*You combine ANN with a `WHERE` clause.*

Ví dụ là `category='rare'`.  
*An example is `category='rare'`.*

Index lấy `ef_search` hàng xóm *trước*.  
*The index fetches `ef_search` neighbors *first*.*

Index lọc *sau*.  
*The index filters *afterwards*.*

Filter hiếm cho ra ít kết quả hơn `LIMIT`.  
*A rare filter returns fewer rows than `LIMIT`.*

Hiện tượng này gọi là **overfiltering** — lọc quá tay.  
*This phenomenon is called **overfiltering**.*

Bạn khắc phục bằng **iterative index scan**.  
*You fix it with an **iterative index scan**.*

Tính năng đó có từ pgvector 0.8.  
*That feature arrived in pgvector 0.8.*

```sql
SET hnsw.iterative_scan = relaxed_order;
SET hnsw.max_scan_tuples = 20000;
```

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder
*4.5. Organizational impact & explaining it to a non-technical stakeholder*

**Nói với PM và sếp / Talking to your PM and your manager**

Index cho search giống mục lục cuốn sách.  
*A search index is like the index of a book.*

Bạn trả một chút chi phí bộ nhớ.  
*You pay a little memory cost.*

Bạn trả một chút thời gian xây dựng.  
*You pay a little build time.*

Bạn đổi lấy tìm kiếm nhanh gấp hàng nghìn lần.  
*You get search that is a thousand times faster.*

Với vector, ta đánh đổi thêm một chút độ chính xác.  
*With vectors, we also trade away a little accuracy.*

Ta lấy tốc độ về.  
*We take speed in return.*

Ta điều chỉnh mức đánh đổi bằng vài núm vặn.  
*We adjust that trade-off with a few knobs.*

Mức điều chỉnh tùy nhu cầu.  
*The setting depends on the need.*

Trên dữ liệu nhỏ, bạn chưa cần index.  
*On small data, you do not need an index yet.*

Index chỉ đáng giá trên dữ liệu lớn.  
*An index pays off only on large data.*

Cách framing này dựa trên chi phí, tốc độ và đánh đổi độ chính xác.  
*This framing rests on cost, speed, and the accuracy trade-off.*

**Roadmap / Roadmap**

Việc chọn và tune index không phải việc one-off — làm một lần rồi thôi.  
*Choosing and tuning an index is not a one-off job.*

Bạn đo recall liên tục.  
*You measure recall continuously.*

Bạn đo latency liên tục.  
*You measure latency continuously.*

Bạn reindex khi data đổi.  
*You reindex when the data changes.*

Bạn nên coi đây là hạ tầng có vòng đời.  
*You should treat this as infrastructure with a lifecycle.*

Đây không phải thứ "set-and-forget".  
*This is not a "set-and-forget" thing.*

**Team / Team**

Mọi index này nằm trong Postgres.  
*All of these indexes live inside Postgres.*

Backend team hiện tại tự vận hành được.  
*Your current backend team can operate them alone.*

Bạn không cần thêm hệ thống search riêng.  
*You need no separate search system.*

Điểm này nối lại luận điểm "một Postgres cho tất cả".  
*This point ties back to the "one Postgres for everything" argument.*

Luận điểm đó xuyên suốt cả series.  
*That argument runs through the whole series.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff
*4.6. A sample system design question + a staff-level answer*

> **Đề bài — câu hỏi system design mẫu**  
> *The brief — a sample system design question*
>
> Bạn hãy thiết kế vector search cho 100 triệu embedding.  
> *Design vector search for 100 million embeddings.*
>
> Chỉ tiêu p95 phải dưới 100ms.  
> *The p95 target must stay under 100ms.*
>
> Hệ thống có filter theo tenant và category.  
> *The system filters by tenant and category.*
>
> Dữ liệu cập nhật hàng ngày.  
> *The data updates daily.*
>
> Ngân sách RAM bị hạn chế.  
> *The RAM budget is limited.*

Khung trả lời ở tầm staff gồm bảy bước.  
*The staff-level answer framework has seven steps.*

Bạn nói to từng trade-off.  
*You say every trade-off out loud.*

**Bước 1 — Clarify (làm rõ đề bài) / Step 1 — Clarify**

QPS là bao nhiêu?  
*What is the QPS?*

Recall target là bao nhiêu?  
*What is the recall target?*

Tần suất update ra sao?  
*How often does the data update?*

Ngân sách RAM cụ thể là bao nhiêu?  
*What exactly is the RAM budget?*

Vector có bao nhiêu chiều?  
*How many dimensions do the vectors have?*

**Bước 2 — Chọn index theo ràng buộc / Step 2 — Pick the index from the constraints**

RAM hạn chế đi kèm 100 triệu vector.  
*A tight RAM budget comes with 100 million vectors.*

HNSW thuần có thể không vừa RAM.  
*Plain HNSW may not fit in RAM.*

Bạn cân nhắc **DiskANN** qua `pgvectorscale`.  
*You consider **DiskANN** through `pgvectorscale`.*

Bạn cũng có thể dùng **HNSW kèm quantization**.  
*You may also use **HNSW with quantization**.*

Lựa chọn quantization gồm halfvec và binary.  
*The quantization options include halfvec and binary.*

Bạn thêm bước two-stage re-rank — xếp hạng lại hai vòng.  
*You add a two-stage re-rank.*

Vòng thô chạy trên index nén.  
*The coarse stage runs on the compressed index.*

Vòng tinh chạy exact trên top-N.  
*The fine stage runs exact search on the top-N.*

**Bước 3 — Filter / Step 3 — Filter**

Bạn partition theo tenant và category.  
*You partition by tenant and category.*

Planner khi đó prune sớm.  
*The planner then prunes early.*

Bạn bật `iterative_scan`.  
*You enable `iterative_scan`.*

Tính năng đó chống overfiltering.  
*That feature guards against overfiltering.*

**Bước 4 — Build và update / Step 4 — Build and update**

Bạn nạp data trước.  
*You load the data first.*

Bạn build index sau.  
*You build the index afterwards.*

Bạn dùng `CREATE INDEX CONCURRENTLY`.  
*You use `CREATE INDEX CONCURRENTLY`.*

Bạn lập lịch reindex cho phần data động.  
*You schedule a reindex for the dynamic portion.*

HNSW chịu insert hàng ngày tốt hơn IVFFlat.  
*HNSW handles daily inserts better than IVFFlat.*

**Bước 5 — Latency / Step 5 — Latency**

Bạn tune `ef_search` theo SLA p95.  
*You tune `ef_search` against the p95 SLA.*

Bạn dùng read replica cho search traffic.  
*You use read replicas for search traffic.*

**Bước 6 — Monitoring / Step 6 — Monitoring**

Bạn đo recall@10.  
*You measure recall@10.*

Bạn so với exact định kỳ.  
*You compare against exact search on a schedule.*

Bạn đo p99 latency.  
*You measure p99 latency.*

Bạn đo RAM và index size.  
*You measure RAM and index size.*

Bạn đo tỉ lệ overfiltering.  
*You measure the overfiltering rate.*

**Bước 7 — Khi nào không cần ANN / Step 7 — When ANN is unnecessary**

Một tenant chỉ có vài nghìn vector.  
*One tenant holds only a few thousand vectors.*

Exact per-tenant khi đó còn nhanh hơn.  
*Per-tenant exact search is then even faster.*

Nó cũng chính xác hơn.  
*It is also more accurate.*

**Không phải mọi thứ đều cần index.**  
***Not everything needs an index.***

Bạn nêu được điều này.  
*You can state this point.*

Đó là dấu hiệu của tư duy staff.  
*That is the mark of staff-level thinking.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 WRAPPING UP FOR THE INTERVIEW (Interview Cheatsheet)*

### 5.1. Keywords bắt buộc nhớ
*5.1. Keywords you must memorize*

**Index — chỉ mục / Index**

Index là một cấu trúc dữ liệu.  
*An index is a data structure.*

Nó tăng tốc việc tìm kiếm.  
*It speeds up searching.*

Nó giống mục lục cuối sách.  
*It is like the index at the back of a book.*

**Linear search — quét tuyến tính / Linear search**

Linear search quét từng dòng.  
*Linear search scans row by row.*

Chi phí trung bình là `(n+1)/2`.  
*The average cost is `(n+1)/2`.*

Chi phí tệ nhất là `n`.  
*The worst case cost is `n`.*

**Binary search — tìm kiếm nhị phân / Binary search**

Binary search chia đôi liên tục.  
*Binary search halves the range repeatedly.*

Nó chạy trên dữ liệu đã sort.  
*It runs on sorted data.*

Chi phí là `O(log n)`.  
*The cost is `O(log n)`.*

**B-tree / B-tree**

B-tree là index kinh điển của RDBMS.  
*The B-tree is the classic RDBMS index.*

Nó phục vụ dữ liệu 1 chiều sắp thứ tự được.  
*It serves one-dimensional sortable data.*

**Nearest Neighbor (NN) và k-NN / Nearest Neighbor (NN) and k-NN**

k-NN tìm k vector gần nhất.  
*k-NN finds the k closest vectors.*

Độ gần đo theo distance.  
*Closeness is measured by distance.*

**ANN (Approximate NN) / ANN (Approximate NN)**

ANN đổi recall lấy tốc độ.  
*ANN trades recall for speed.*

Bạn cần nó khi vector có nhiều chiều.  
*You need it when vectors have many dimensions.*

**Recall — độ bao phủ / Recall**

Recall là phần trăm hàng xóm thật mà search tìm được.  
*Recall is the percentage of true neighbors the search finds.*

Nó là thước đo chất lượng của ANN.  
*It is the quality metric for ANN.*

**Curse of dimensionality — lời nguyền số chiều / Curse of dimensionality**

Ở chiều cao, khoảng cách co lại gần nhau.  
*At high dimensions, distances converge.*

Cây và exact index hỏng ở đó.  
*Trees and exact indexes break down there.*

**IVFFlat / IVFFlat**

IVFFlat viết đầy đủ là Inverted File Flat.  
*IVFFlat stands for Inverted File Flat.*

Nó chia dữ liệu thành `lists` cụm bằng k-means.  
*It splits the data into `lists` clusters with k-means.*

`probes` là số cụm dò khi search.  
*`probes` is the number of clusters probed at search time.*

**Centroid — tâm cụm / Centroid**

Centroid là tâm của một cụm.  
*A centroid is the center of a cluster.*

Query so với centroid để chọn cụm cần tìm.  
*The query compares against centroids to pick the cluster to search.*

**HNSW / HNSW**

HNSW viết đầy đủ là Hierarchical Navigable Small **World**.  
*HNSW stands for Hierarchical Navigable Small **World**.*

Nó là một graph nhiều tầng.  
*It is a multi-layer graph.*

Chi phí search khoảng `O(log n)`.  
*The search cost is about `O(log n)`.*

**`m`, `ef_construction`, `ef_search` / `m`, `ef_construction`, `ef_search`**

`m` là số cạnh mỗi tầng.  
*`m` is the edge count per layer.*

`ef_construction` là số ứng viên lúc build.  
*`ef_construction` is the candidate count at build time.*

`ef_search` là số ứng viên lúc query.  
*`ef_search` is the candidate count at query time.*

**`lists` và `probes` / `lists` and `probes`**

`lists` và `probes` là tham số của IVFFlat.  
*`lists` and `probes` are IVFFlat parameters.*

**DiskANN và pgvectorscale / DiskANN and pgvectorscale**

DiskANN là index chạy trên nền SSD.  
*DiskANN is an SSD-based index.*

Nó tiết kiệm RAM.  
*It saves RAM.*

Nó hỗ trợ chiều tới 16000.  
*It supports up to 16000 dimensions.*

**ops class — lớp toán tử / ops class**

`vector_cosine_ops` đi với `<=>`.  
*`vector_cosine_ops` pairs with `<=>`.*

`vector_l2_ops` đi với `<->`.  
*`vector_l2_ops` pairs with `<->`.*

`vector_ip_ops` đi với `<#>`.  
*`vector_ip_ops` pairs with `<#>`.*

**Iterative scan / Iterative scan**

Iterative scan chống overfiltering.  
*Iterative scan guards against overfiltering.*

Nó dùng khi bạn kết hợp ANN với `WHERE`.  
*You use it when combining ANN with `WHERE`.*

Tính năng này có từ pgvector 0.8.  
*This feature arrived in pgvector 0.8.*

### 5.2. Core concepts — nếu chỉ nhớ vài điều
*5.2. Core concepts — if you remember only a few things*

**Điều 1 / Point 1**

Index đổi chút chi phí lưu trữ và build lấy tốc độ tìm kiếm khổng lồ.  
*An index trades a little storage and build cost for enormous search speed.*

Nó chỉ đáng giá khi dữ liệu lớn.  
*It only pays off on large data.*

**Điều 2 / Point 2**

Linear search tốn `O(n)`.  
*Linear search costs `O(n)`.*

Binary search và B-tree tốn `O(log n)`.  
*Binary search and the B-tree cost `O(log n)`.*

**Binary search cần dữ liệu sắp thứ tự được.**  
***Binary search needs sortable data.***

**Điều 3 / Point 3**

**Vector nhiều chiều không sắp thứ tự trên một trục.**  
***High-dimensional vectors have no single sorting axis.***

B-tree và binary search khi đó vô dụng.  
*The B-tree and binary search are useless there.*

Bạn cần ANN.  
*You need ANN.*

**Điều 4 / Point 4**

ANN đổi **recall lấy tốc độ**.  
*ANN trades **recall for speed**.*

Exact search không dùng index.  
*Exact search uses no index.*

Nó cho recall 100%.  
*It gives 100% recall.*

Chi phí của nó là `O(n)`.  
*Its cost is `O(n)`.*

**Điều 5 / Point 5**

**IVFFlat** chia cụm theo centroid và list.  
***IVFFlat** clusters the data by centroid and list.*

Nó dò `probes` cụm khi search.  
*It probes `probes` clusters during a search.*

Nó build nhanh và nhẹ.  
*It builds fast and stays light.*

Nó cần sẵn data để build.  
*It needs data present to build.*

Nó ngại data động.  
*It copes poorly with dynamic data.*

**Điều 6 / Point 6**

**HNSW** là graph nhiều tầng.  
***HNSW** is a multi-layer graph.*

Chi phí search khoảng `O(log n)`.  
*The search cost is about `O(log n)`.*

Nó cho recall cao.  
*It gives high recall.*

Nó chịu data động tốt.  
*It handles dynamic data well.*

Nó tốn RAM.  
*It consumes RAM.*

Nó build lâu.  
*It takes long to build.*

**Điều 7 / Point 7**

`probes` và `ef_search` là núm xoay recall–speed lúc query.  
*`probes` and `ef_search` are the recall–speed knobs at query time.*

`lists`, `m` và `ef_construction` là núm xoay lúc build.  
*`lists`, `m`, and `ef_construction` are the knobs at build time.*

**Điều 8 / Point 8**

Với dữ liệu nhỏ, bạn **đừng đánh index**.  
*On small data, you should **not build an index**.*

Exact search đủ nhanh.  
*Exact search is fast enough.*

Nó còn chính xác hơn.  
*It is also more accurate.*

**Điều 9 / Point 9**

Ops class phải khớp toán tử.  
*The ops class must match the operator.*

Nếu không, index bị bỏ qua.  
*Otherwise the index is ignored.*

Query rơi về seq scan một cách âm thầm.  
*The query falls back to a seq scan silently.*

**Điều 10 / Point 10**

Ở quy mô lớn, RAM là bottleneck.  
*At large scale, RAM is the bottleneck.*

Bạn dùng quantization hoặc DiskANN.  
*You use quantization or DiskANN.*

Bạn phải **đo recall**.  
*You must **measure recall**.*

ANN xuống cấp mà không báo lỗi.  
*ANN degrades without raising any error.*

### 5.3. Ideas / mental models
*5.3. Ideas / mental models*

**"Mục lục cuối sách" / "The index at the back of a book"**

Câu này nói index là gì trong 1 câu.  
*This phrase says what an index is in one sentence.*

**"Sắp được thì binary search được" / "If you can sort it, you can binary search it"**

Câu này giải thích vì sao vector cần index khác.  
*This phrase explains why vectors need a different index.*

**"Chia quận, chỉ vào vài quận" / "Divide into districts, then visit only a few"**

Câu này mô tả IVFFlat.  
*This phrase describes IVFFlat.*

Nó cũng gợi nhớ điểm yếu ranh giới cụm.  
*It also recalls the cluster-boundary weakness.*

**"Đường cao tốc nhiều tầng" / "A multi-layer highway"**

Câu này mô tả cách HNSW navigate.  
*This phrase describes how HNSW navigates.*

Nó đi từ tầng cao xuống tầng thấp.  
*It travels from the high layer down to the low layer.*

**"ANN xuống cấp trong im lặng" / "ANN degrades in silence"**

Câu này giải thích vì sao phải monitor recall.  
*This phrase explains why you must monitor recall.*

### 5.4. Code cần thuộc lòng
*5.4. Code you should know by heart*

**(a) Hai câu tạo index — interviewer hay bắt viết / (a) The two index statements — interviewers often ask for these**

```sql
CREATE INDEX ON items USING hnsw    (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
SET hnsw.ef_search = 100;      -- query-time recall knob
-- núm xoay recall lúc chạy query
SET ivfflat.probes  = 32;
```

**(b) Binary search từ số 0 / (b) Binary search from scratch**

```python
def binary_search(a, x):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == x: return mid
        if a[mid] < x:  lo = mid + 1
        else:           hi = mid - 1
    return -1
```

**(c) Đo recall — so ANN với exact / (c) Measuring recall — comparing ANN against exact**

```sql
-- exact (ground truth): SET LOCAL enable_indexscan = off; trong BEGIN...COMMIT
-- exact (ground truth): SET LOCAL enable_indexscan = off; inside BEGIN...COMMIT
-- rồi so overlap top-k với kết quả có index -> recall@k
-- then compare the top-k overlap against the indexed result -> recall@k
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions + suggested answers*

**Câu 1 — "Index tiết kiệm thời gian thế nào?" / Question 1 — "How does an index save time?"**

Linear search tốn `O(n)`.  
*Linear search costs `O(n)`.*

Trung bình nó đọc `(n+1)/2` dòng.  
*On average it reads `(n+1)/2` rows.*

Binary search và B-tree tốn `O(log n)`.  
*Binary search and the B-tree cost `O(log n)`.*

Với 1 triệu dòng, cách cũ đọc khoảng 500K lần.  
*On one million rows, the old way makes about 500K reads.*

Cách mới chỉ tốn khoảng 20 bước.  
*The new way takes only about 20 steps.*

**Câu 2 — [MẤU CHỐT] "Vì sao không dùng B-tree cho vector search?" / Question 2 — [KEY] "Why not use a B-tree for vector search?"**

Vector nhiều chiều không có thứ tự tuyến tính trên một trục.  
*High-dimensional vectors have no linear order along one axis.*

Bạn không sort chúng được.  
*You cannot sort them.*

Câu hỏi ở đây là "gần nhất" hay NN.  
*The question here is "nearest", or NN.*

Nó không phải khớp giá trị hay khoảng.  
*It is not an exact match or a range.*

Curse of dimensionality làm cây phân hoạch sập về `O(n)`.  
*The curse of dimensionality collapses partition trees back to `O(n)`.*

Vì vậy bạn cần ANN.  
*For that reason you need ANN.*

**Câu 3 — "IVFFlat vs HNSW?" / Question 3 — "IVFFlat vs HNSW?"**

IVFFlat chia cụm quanh centroid.  
*IVFFlat clusters around centroids.*

Tham số của nó là `lists` và `probes`.  
*Its parameters are `lists` and `probes`.*

Nó build nhanh và nhẹ.  
*It builds fast and stays light.*

Nó cần sẵn data.  
*It needs data present.*

Nó kém với data động.  
*It is weak with dynamic data.*

Recall của nó thấp hơn.  
*Its recall is lower.*

HNSW là graph nhiều tầng.  
*HNSW is a multi-layer graph.*

Chi phí search khoảng `O(log n)`.  
*The search cost is about `O(log n)`.*

Recall của nó cao.  
*Its recall is high.*

Nó chịu data động tốt.  
*It handles dynamic data well.*

Nó build lâu và tốn RAM.  
*It builds slowly and eats RAM.*

Lựa chọn mặc định nay là HNSW.  
*The default choice today is HNSW.*

**Câu 4 — [TRADE-OFF] "Tăng `probes` hoặc `ef_search` được gì mất gì?" / Question 4 — [TRADE-OFF] "What do you gain and lose by raising `probes` or `ef_search`?"**

Recall tăng lên.  
*Recall rises.*

Latency cũng tăng lên.  
*Latency rises too.*

Đây là núm xoay recall–speed lúc query.  
*This is the recall–speed knob at query time.*

Bạn tune nó theo SLA.  
*You tune it against the SLA.*

**Câu 5 — [BẪY] "Kết quả 'sai' sau khi đánh index?" / Question 5 — [TRAP] "The results are 'wrong' after indexing?"**

Kết quả không sai.  
*The results are not wrong.*

ANN vốn là approximate.  
*ANN is approximate by nature.*

Nó đổi recall lấy tốc độ.  
*It trades recall for speed.*

Bạn tăng tham số để cải thiện.  
*You raise the parameters to improve it.*

Bạn đo recall để định lượng.  
*You measure recall to quantify it.*

**Câu 6 — [BẪY] "Nên đánh index cho bảng 5000 vector không?" / Question 6 — [TRAP] "Should you index a table of 5000 vectors?"**

Câu trả lời là không.  
*The answer is no.*

Exact brute-force đủ nhanh.  
*Exact brute-force is fast enough.*

Nó cho recall 100%.  
*It gives 100% recall.*

Index chỉ đáng giá khi dữ liệu lớn.  
*An index pays off only on large data.*

**Câu 7 — [SCALE] "100M vector, RAM hạn chế?" / Question 7 — [SCALE] "100M vectors on a tight RAM budget?"**

HNSW thuần có thể không vừa RAM.  
*Plain HNSW may not fit in RAM.*

Bạn dùng quantization kèm two-stage re-rank.  
*You use quantization with a two-stage re-rank.*

Lựa chọn quantization gồm halfvec và binary.  
*The quantization options include halfvec and binary.*

Bạn cũng có thể chọn DiskANN.  
*You may also pick DiskANN.*

Bạn partition dữ liệu cho phần filter.  
*You partition the data for the filter.*

Bạn bật `iterative_scan`.  
*You enable `iterative_scan`.*

Bạn đo recall và p95.  
*You measure recall and p95.*

**Câu 8 — "Vì sao IVFFlat không nên tạo khi bảng rỗng?" / Question 8 — "Why avoid creating IVFFlat on an empty table?"**

IVFFlat cần data cho k-means.  
*IVFFlat needs data for k-means.*

k-means dùng data để tìm centroid.  
*k-means uses the data to locate centroids.*

Bảng rỗng cho ra phân cụm vô nghĩa.  
*An empty table yields meaningless clusters.*

HNSW thì tạo được cả khi bảng rỗng.  
*HNSW, by contrast, builds fine on an empty table.*

### 5.6. One-liner đắt giá
*5.6. High-value one-liners*

Index đổi một chút chi phí lưu trữ và thời gian build.  
*An index trades a little storage and build time.*

Nó đổi lấy mức giảm khổng lồ về thời gian tìm kiếm.  
*It buys a massive drop in search time.*

Nó chỉ đáng giá ở quy mô lớn.  
*It only pays off at scale.*

Binary search cần một trục để sắp thứ tự.  
*Binary search needs a line to sort along.*

Vector không có trục đó.  
*Vectors do not have one.*

Vì vậy ta xấp xỉ thay vì sắp thứ tự.  
*So we approximate instead of sorting.*

ANN đổi recall lấy tốc độ.  
*ANN trades recall for speed.*

Kỹ năng thật nằm ở việc đo recall.  
*The real skill is measuring recall.*

Recall xuống cấp trong im lặng.  
*Recall degrades silently.*

Không có lỗi nào để bắt nó.  
*No error appears to catch it.*

IVFFlat chia cụm rồi dò cụm.  
*IVFFlat clusters and probes.*

HNSW leo một đường cao tốc nhiều tầng.  
*HNSW climbs a multi-layer highway.*

Hai bên có cùng mục tiêu.  
*Both share the same goal.*

Hai bên đặt cược khác nhau giữa memory và build time.  
*They bet differently on memory versus build time.*

Không phải mọi thứ đều cần index.  
*Not everything needs an index.*

Trên dữ liệu nhỏ, exact search dễ suy luận hơn.  
*On small data, exact search is easier to reason about.*

Nó cũng đúng 100%.  
*It is also 100% correct.*

---

### 📌 Ghi chú cuối
*📌 Closing notes*

**Đính chính để nhớ đúng / Corrections to memorize**

HNSW là Small **World**.  
*HNSW is Small **World**.*

Nó không phải Small Words.  
*It is not Small Words.*

Cạnh HNSW theo **metric bạn cấu hình**.  
*HNSW edges follow **the metric you configure**.*

Nó không riêng Euclidean.  
*It is not Euclidean alone.*

Index HNSW và IVFFlat giới hạn **2000 chiều**.  
*The HNSW and IVFFlat indexes cap at **2000 dimensions**.*

Với chiều cao hơn, bạn dùng `halfvec`.  
*For higher dimensions, you use `halfvec`.*

Bạn cũng có thể dùng DiskANN.  
*You may also use DiskANN.*

**Kiểm chứng khi ôn / Verify while you revise**

pgvector hiện ở phiên bản **v0.8.x**.  
*pgvector currently sits at version **v0.8.x**.*

Phiên bản này lấy HNSW làm mặc định.  
*This version treats HNSW as the default.*

Phiên bản này có iterative scan.  
*This version includes iterative scan.*

DiskANN đến qua `pgvectorscale`.  
*DiskANN arrives through `pgvectorscale`.*

Bạn xem CHANGELOG trên GitHub `pgvector/pgvector`.  
*Check the CHANGELOG on GitHub at `pgvector/pgvector`.*

**Thực hành / Practice**

Bạn dựng Postgres kèm pgvector.  
*Set up Postgres with pgvector.*

Bạn nạp khoảng 100K vector.  
*Load about 100K vectors.*

Bạn tạo cả IVFFlat và HNSW.  
*Create both IVFFlat and HNSW.*

Bạn chạy `EXPLAIN ANALYZE`.  
*Run `EXPLAIN ANALYZE`.*

Bạn sẽ thấy dòng `Index Scan`.  
*You will see the `Index Scan` line.*

Bạn tắt index bằng `enable_indexscan=off`.  
*Turn the index off with `enable_indexscan=off`.*

Bạn so top-k để đo recall.  
*Compare the top-k results to measure recall.*

Bạn vặn `probes` và `ef_search`.  
*Turn the `probes` and `ef_search` knobs.*

Bạn sẽ *thấy* trade-off recall–latency.  
*You will *see* the recall–latency trade-off.*

**Nối mạch series — đủ 4 mảnh / Connecting the series — all 4 pieces**

Mảnh embed tạo vector.  
*The embed piece creates vectors.*

Mảnh **index** làm search nhanh.  
*The **index** piece makes search fast.*

Bài này chính là mảnh index.  
*This lesson is that index piece.*

Mảnh store & query dùng pgvector.  
*The store & query piece uses pgvector.*

Mảnh keyword dùng FTS.  
*The keyword piece uses FTS.*

Cuối cùng là hybrid.  
*Hybrid comes last.*

Toàn bộ nằm trong một Postgres.  
*All of it lives inside one Postgres.*

**Học tiếp / What to study next**

Bạn học tiếp về quantization.  
*Study quantization next.*

Các dạng gồm scalar, product và binary.  
*The variants include scalar, product, and binary.*

Bạn học tiếp về two-stage retrieval và reranking.  
*Study two-stage retrieval and reranking next.*

Bạn học tiếp về sharding vector.  
*Study vector sharding next.*

Công cụ gồm Citus và `pgvectorscale`.  
*The tools include Citus and `pgvectorscale`.*

Bạn học cách benchmark recall và latency có hệ thống.  
*Learn to benchmark recall and latency systematically.*

Bộ công cụ tham khảo là ann-benchmarks.  
*The reference toolkit is ann-benchmarks.*
