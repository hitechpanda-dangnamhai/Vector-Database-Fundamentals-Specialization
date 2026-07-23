# Query Dữ Liệu vector — véc-tơ trong pgvector: Toán Tử distance — khoảng cách và threshold — ngưỡng
*Vector Data Queries in pgvector: Distance and Threshold Operators — Basic to Staff Course*

> **Nguồn gốc**  
> ***Source***

> Tài liệu dựa trên video *"Perform vector queries, including using pgvector for data queries"*.  
> *This material follows the video *"Perform vector queries, including using pgvector for data queries"*.*

> Video thuộc Course 3 của IBM Vector Database Fundamentals.  
> *The video belongs to Course 3 of IBM Vector Database Fundamentals.*

> Bài gốc trình bày các yêu cầu cho query vector.  
> *The original lesson presents the requirements for vector queries.*

> Bài gốc hướng dẫn cách viết `tsquery` cho `tsvector`.  
> *The original lesson explains `tsquery` construction for `tsvector`.*

> Bài gốc giới thiệu ba toán tử distance.  
> *The original lesson introduces three distance operators.*

> Ba toán tử đó là `<->`, `<#>` và `<=>`.  
> *The three operators are `<->`, `<#>`, and `<=>`.*

> Bài gốc trình bày search theo cosine hoặc distance.  
> *The original lesson presents searches with cosine or distance.*

> Bài gốc cũng trình bày giới hạn bằng threshold.  
> *The original lesson also presents threshold-based limits.*

> Tôi bám sát nội dung gốc.  
> *I closely follow the original content.*

> Tôi đào sâu thêm các điểm quan trọng.  
> *I explore the important points more deeply.*

> Nội dung mở rộng mang nhãn **[MỞ RỘNG NGOÀI BÀI GỐC]**.  
> *Extended content carries the label **[MỞ RỘNG NGOÀI BÀI GỐC]**.*

> **Vị trí trong series**  
> ***Position in the series***

> Đây là mảnh *"lấy dữ liệu ra ĐÚNG"*.  
> *This is the *"retrieve data CORRECTLY"* module.*

> Mảnh này hoàn tất bộ hands-on.  
> *This module completes the hands-on sequence.*

> Bộ hands-on bắt đầu với cài đặt ở bài #6.  
> *The hands-on sequence starts with installation in lesson #6.*

> Bộ hands-on tiếp tục với nạp dữ liệu ở bài #7.  
> *The hands-on sequence continues with data loading in lesson #7.*

> Bộ hands-on kết thúc với query trong bài này.  
> *The hands-on sequence ends with queries in this lesson.*

> Toán tử distance đã xuất hiện trong giáo trình pgvector khái niệm #3.  
> *Distance operators appeared in the pgvector concepts lesson #3.*

> `tsquery` đã xuất hiện trong giáo trình FTS #6.  
> *`tsquery` appeared in the FTS lesson #6.*

> Bài này tập trung vào cách viết query thực tế.  
> *This lesson focuses on practical query construction.*

> Bài này đào sâu phần threshold.  
> *This lesson explores threshold behavior in depth.*

> Các bài trước chưa trình bày kỹ phần này.  
> *The previous lessons did not cover this topic deeply.*

> **⚠️ Đính chính tới 2026**  
> ***⚠️ Corrections through 2026***

> Bài gốc có một số chỗ chưa chuẩn.  
> *The original lesson contains several inaccurate points.*

> 1. **`<=>` là cosine distance — khoảng cách cosin.**  
>    ***`<=>` is cosine distance.***

> `<=>` không phải cosine similarity — độ tương đồng cosin.  
> *`<=>` is not cosine similarity.*

> Bài gốc dùng cụm "cosine similarity to retrieve records".  
> *The original lesson uses the phrase "cosine similarity to retrieve records".*

> Toán tử này thực tế trả về distance.  
> *This operator actually returns distance.*

> Distance bằng `1 − similarity`.  
> *Distance equals `1 − similarity`.*

> Cosine distance có miền `[0,2]`.  
> *Cosine distance has the range `[0,2]`.*

> Bạn lấy similarity bằng `1 - (embedding <=> q)`.  
> *You obtain similarity with `1 - (embedding <=> q)`.*

> 2. **Threshold phụ thuộc metric — thước đo.**  
>    ***A threshold depends on the metric.***

> Threshold cũng phụ thuộc dataset — tập dữ liệu.  
> *A threshold also depends on the dataset.*

> Số "5" xuất hiện trong ví dụ L2 của bài gốc.  
> *The number "5" appears in the original L2 example.*

> Số "5" không phải một hằng số vạn năng.  
> *The number "5" is not a universal constant.*

> Mỗi metric có một thang giá trị riêng.  
> *Each metric has its own value scale.*

> Mỗi dataset cần một ngưỡng riêng.  
> *Each dataset needs its own threshold.*

> Cosine distance nằm trong miền `[0,2]`.  
> *Cosine distance lies within the range `[0,2]`.*

> L2 không có giới hạn trên.  
> *L2 has no upper bound.*

> 3. **Query có một bẫy liên quan đến index — chỉ mục.**  
>    ***Queries contain an index-related trap.***

> Query k-NN chạy nhanh nhờ `ORDER BY distance LIMIT k`.  
> *A k-NN query runs quickly with `ORDER BY distance LIMIT k`.*

> Threshold thuần chỉ dùng điều kiện `WHERE distance < x`.  
> *A bare threshold uses only the condition `WHERE distance < x`.*

> Threshold thuần không có `ORDER BY` hoặc `LIMIT`.  
> *A bare threshold has no `ORDER BY` or `LIMIT`.*

> Kiểu query này thường không dùng được ANN index.  
> *This query shape usually cannot use an ANN index.*

> PostgreSQL có thể thực hiện seq scan.  
> *PostgreSQL may perform a sequential scan.*

> Bài gốc không nhắc đến bẫy này.  
> *The original lesson does not mention this trap.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson Map (Overview)*

**Bài này dạy gì**  
***What this lesson teaches***

Bài này hướng dẫn cách viết query.  
*This lesson explains query construction.*

Query lấy các bản ghi giống nhất với một vector.  
*The query retrieves the records most similar to a vector.*

Bạn sử dụng đúng toán tử distance.  
*You use the correct distance operator.*

Bạn sử dụng đúng pattern — mẫu truy vấn.  
*You use the correct query pattern.*

Pattern có thể tìm một bản ghi đã tồn tại.  
*The pattern can find an existing record.*

Pattern có thể loại chính bản ghi nguồn.  
*The pattern can exclude the source record.*

Pattern có thể hiển thị điểm similarity.  
*The pattern can display the similarity score.*

Bạn dùng threshold để lọc kết quả.  
*You use a threshold to filter results.*

Threshold chỉ giữ các kết quả đủ gần.  
*The threshold keeps only sufficiently close results.*

Model — mô hình phải giống nhau.  
*The model must be the same.*

Số chiều của vector phải giống nhau.  
*The vector dimensions must be the same.*

**Vấn đề nó giải quyết**  
***The problem it solves***

Query vector có cú pháp gần giống SQL thông thường.  
*A vector query resembles ordinary SQL.*

Một số lỗi có thể làm kết quả mất ý nghĩa.  
*Several mistakes can make the results meaningless.*

Bạn có thể SELECT trực tiếp cột vector.  
*You may directly SELECT the vector column.*

Kết quả khi đó chỉ là một mảng số vô dụng.  
*The result is then only a useless numeric array.*

Bạn có thể dùng sai toán tử.  
*You may use the wrong operator.*

Bạn có thể dùng sai metric.  
*You may use the wrong metric.*

Bạn có thể đặt threshold tùy tiện.  
*You may choose an arbitrary threshold.*

Bạn có thể viết query làm index bị bỏ.  
*You may write a query that bypasses the index.*

Bài này giải thích từng lỗi.  
*This lesson explains each mistake.*

**Học xong bạn sẽ làm được**  
***What you can do after this lesson***

- Bạn có thể nêu yêu cầu về cùng model.  
  *You can state the same-model requirement.*

- Bạn có thể nêu yêu cầu về cùng length — độ dài.  
  *You can state the same-length requirement.*

- Bạn có thể giải thích lý do không SELECT raw vector.  
  *You can explain the reason for avoiding raw vector selection.*

- Bạn có thể dùng đúng toán tử `<->`.  
  *You can correctly use the `<->` operator.*

- Bạn có thể dùng đúng toán tử `<#>`.  
  *You can correctly use the `<#>` operator.*

- Bạn có thể dùng đúng toán tử `<=>`.  
  *You can correctly use the `<=>` operator.*

- Bạn có thể chọn toán tử phù hợp.  
  *You can choose the appropriate operator.*

- Bạn có thể viết query "tìm k bản ghi giống nhất với bản ghi X".  
  *You can write a "find the k records most similar to record X" query.*

- Bạn có thể loại chính bản ghi X.  
  *You can exclude record X itself.*

- Bạn có thể lấy giá trị distance.  
  *You can retrieve the distance value.*

- Bạn có thể lấy giá trị similarity.  
  *You can retrieve the similarity value.*

- Bạn có thể lọc đúng bằng threshold.  
  *You can filter correctly with a threshold.*

- Bạn có thể giải thích sự phụ thuộc vào metric.  
  *You can explain metric dependence.*

- Bạn có thể giải thích sự phụ thuộc vào dataset.  
  *You can explain dataset dependence.*

- Bạn có thể tránh bẫy index.  
  *You can avoid the index trap.*

- Bạn có thể dùng query pattern ở quy mô lớn.  
  *You can use query patterns at scale.*

- Bạn có thể dùng pre-filter — bộ lọc trước.  
  *You can use a pre-filter.*

- Bạn có thể dùng hybrid — kết hợp nhiều cách tìm kiếm.  
  *You can use hybrid retrieval.*

- Bạn có thể dùng pagination — phân trang.  
  *You can use pagination.*

- Bạn có thể hiệu chỉnh threshold bằng thực nghiệm.  
  *You can calibrate a threshold empirically.*

**Mạch basic → staff**  
***Basic-to-staff progression***

- 🟢 **Basic:** Bạn học yêu cầu cùng model.  
  *🟢 **Basic:** You learn the same-model requirement.*

- 🟢 **Basic:** Bạn học yêu cầu cùng length.  
  *🟢 **Basic:** You learn the same-length requirement.*

- 🟢 **Basic:** Bạn phân biệt `tsquery` cho text.  
  *🟢 **Basic:** You distinguish `tsquery` for text.*

- 🟢 **Basic:** Bạn phân biệt vector cho embedding — vector biểu diễn.  
  *🟢 **Basic:** You distinguish vectors for embeddings.*

- 🟢 **Basic:** Bạn viết query similarity đơn giản nhất.  
  *🟢 **Basic:** You write the simplest similarity query.*

- 🟡 **Intermediate:** Bạn học ba toán tử trong query.  
  *🟡 **Intermediate:** You learn the three query operators.*

- 🟡 **Intermediate:** Bạn học pattern "giống bản ghi có sẵn".  
  *🟡 **Intermediate:** You learn the "similar to an existing record" pattern.*

- 🟡 **Intermediate:** Pattern này dùng subquery — truy vấn con.  
  *🟡 **Intermediate:** This pattern uses a subquery.*

- 🟡 **Intermediate:** Bạn đưa distance vào SELECT.  
  *🟡 **Intermediate:** You include distance in SELECT.*

- 🟡 **Intermediate:** Bạn đưa similarity vào SELECT.  
  *🟡 **Intermediate:** You include similarity in SELECT.*

- 🟡 **Intermediate:** Bạn loại chính bản ghi nguồn.  
  *🟡 **Intermediate:** You exclude the source record.*

- 🟡 **Intermediate:** Bạn nhận diện các lỗi thường gặp.  
  *🟡 **Intermediate:** You identify common mistakes.*

- 🔴 **Advanced:** Bạn đào sâu threshold phụ thuộc metric.  
  *🔴 **Advanced:** You explore metric-dependent thresholds.*

- 🔴 **Advanced:** Bạn học cách hiệu chỉnh threshold.  
  *🔴 **Advanced:** You learn threshold calibration.*

- 🔴 **Advanced:** Bạn nhận diện bẫy index.  
  *🔴 **Advanced:** You identify the index trap.*

- 🔴 **Advanced:** Bạn đổi distance thành similarity phần trăm.  
  *🔴 **Advanced:** You convert distance into a similarity percentage.*

- 🔴 **Advanced:** Bạn kết hợp threshold với LIMIT.  
  *🔴 **Advanced:** You combine a threshold with LIMIT.*

- 🔴 **Advanced:** Bạn sử dụng iterative scan — quét lặp.  
  *🔴 **Advanced:** You use iterative scans.*

- 🔴 **Advanced:** Bạn xử lý các edge cases — trường hợp biên.  
  *🔴 **Advanced:** You handle edge cases.*

- 🟣 **Staff:** Bạn áp dụng query pattern ở quy mô lớn.  
  *🟣 **Staff:** You apply query patterns at scale.*

- 🟣 **Staff:** Bạn dùng pre-filter.  
  *🟣 **Staff:** You use pre-filters.*

- 🟣 **Staff:** Bạn dùng hybrid retrieval.  
  *🟣 **Staff:** You use hybrid retrieval.*

- 🟣 **Staff:** Bạn xử lý pagination.  
  *🟣 **Staff:** You handle pagination.*

- 🟣 **Staff:** Bạn calibrate threshold — hiệu chỉnh ngưỡng.  
  *🟣 **Staff:** You calibrate thresholds.*

- 🟣 **Staff:** Bạn sử dụng cache — bộ nhớ đệm.  
  *🟣 **Staff:** You use caching.*

- 🟣 **Staff:** Bạn monitor — giám sát hệ thống.  
  *🟣 **Staff:** You monitor the system.*

- 🟣 **Staff:** Bạn dùng cả mẫu giống lẫn mẫu khác.  
  *🟣 **Staff:** You use both similar and dissimilar samples.*

- 🟣 **Staff:** Bạn trả lời câu hỏi system design.  
  *🟣 **Staff:** You answer system design questions.*

- 🎯 **Cheatsheet:** Bạn ghi nhớ keywords.  
  *🎯 **Cheatsheet:** You memorize keywords.*

- 🎯 **Cheatsheet:** Bạn ghi nhớ core concepts.  
  *🎯 **Cheatsheet:** You memorize core concepts.*

- 🎯 **Cheatsheet:** Bạn ghi nhớ các đoạn code chính.  
  *🎯 **Cheatsheet:** You memorize the core code snippets.*

- 🎯 **Cheatsheet:** Bạn luyện câu hỏi phỏng vấn.  
  *🎯 **Cheatsheet:** You practice interview questions.*

- 🎯 **Cheatsheet:** Bạn ghi nhớ các one-liner.  
  *🎯 **Cheatsheet:** You memorize the one-liners.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

### 1.1. Hai yêu cầu bắt buộc khi query vector
*1.1. Two mandatory requirements for vector queries*

1. **Cùng model.**  
   ***Use the same model.***

Embedding lưu trong database — cơ sở dữ liệu phải dùng cùng model.  
*The embedding stored in the database must use the same model.*

Embedding của câu tìm kiếm phải dùng cùng model.  
*The search query embedding must use the same model.*

Hai embedding phải dùng cùng version — phiên bản.  
*Both embeddings must use the same version.*

Vector từ hai model nằm trong hai không gian khác nhau.  
*Vectors from two models occupy different spaces.*

Distance giữa hai không gian đó không có ý nghĩa.  
*Distance across those spaces has no meaning.*

Kết quả query khi đó sẽ hỗn loạn.  
*The query results will then be unreliable.*

Bài gốc nhấn mạnh đúng điểm này.  
*The original lesson correctly emphasizes this point.*

2. **Cùng số chiều hoặc length.**  
   ***Use the same dimension count or length.***

Hai vector phải có cùng độ dài.  
*Both vectors must have the same length.*

Phép tính distance cần điều kiện này.  
*The distance calculation requires this condition.*

Model quyết định số chiều của vector.  
*The model determines the vector dimension count.*

Bạn đổi model.  
*You change the model.*

Số chiều có thể thay đổi.  
*The dimension count may change.*

Bạn phải re-embed — tạo lại embedding cho toàn bộ dữ liệu.  
*You must re-embed the entire dataset.*

Giáo trình embeddings trình bày chi tiết nội dung này.  
*The embeddings lesson explains this topic in detail.*

### 1.2. Đừng SELECT raw vector — nó vô nghĩa với con người
*1.2. Do not SELECT raw vectors — they are meaningless to people*

```sql
SELECT embedding FROM reviews LIMIT 1;
-- => [0.0123, -0.87, 0.44, ... 512 số ...]   <- con người không đọc hiểu gì
-- => [0.0123, -0.87, 0.44, ... 512 numbers ...]   <- humans cannot understand this
```

Vector chỉ là một mảng float — số thực dấu phẩy động rất lớn.  
*A vector is only a very large float array.*

Raw vector không cung cấp thông tin hữu ích cho con người.  
*A raw vector provides no useful information to people.*

Việc lấy nguyên trạng vector làm sai mục đích query.  
*Retrieving the raw vector defeats the query purpose.*

Mục tiêu không phải là xem vector.  
*The goal is not vector inspection.*

Mục tiêu là so sánh các vector.  
*The goal is vector comparison.*

Phép so sánh giúp tìm các bản ghi giống nhau.  
*The comparison finds similar records.*

Query luôn tập trung vào distance hoặc similarity.  
*A query always focuses on distance or similarity.*

Query không tập trung vào bản thân cột vector.  
*A query does not focus on the vector column itself.*

### 1.3. Hai loại "search văn bản" — đừng nhầm
*1.3. Two types of "text search" — do not confuse them*

Bài gốc nhắc đến `tsquery`.  
*The original lesson mentions `tsquery`.*

Bài gốc cũng nhắc đến pgvector.  
*The original lesson also mentions pgvector.*

Hai cơ chế này rất dễ bị nhầm.  
*These two mechanisms are easy to confuse.*

Bảng sau trình bày sự khác biệt.  
*The following table shows the differences.*

|  | `tsquery` trên `tsvector` / `tsquery` on `tsvector` | pgvector (`<=>`...) / pgvector (`<=>`...) |
|---|---|---|
| Tìm theo / Search basis | **lexeme — từ khóa chuẩn hóa** / **normalized lexemes** | **ngữ nghĩa** bằng embedding / **meaning** through embeddings |
| Hợp cho / Best use | keyword search chính xác / exact keyword search | semantic search — tìm kiếm ngữ nghĩa / semantic search |
| "red" khớp "reddish"? / Does "red" match "reddish"? | có, do cùng lexeme / yes, through the same lexeme | có với nghĩa gần / yes, with similar meaning |
| "ô tô" khớp "xe hơi"? / Does "ô tô" match "xe hơi"? | không / no | **có** / **yes** |
| Chi tiết / Details | giáo trình FTS #6 / FTS lesson #6 | bài này / this lesson |

Bài gốc đưa ra một ví dụ `tsquery`.  
*The original lesson provides a `tsquery` example.*

Ví dụ dùng chuỗi *"a red mat costs as much as a blue mat"*.  
*The example uses the string *"a red mat costs as much as a blue mat"*.*

Query tìm cả `mat` lẫn `red`.  
*The query searches for both `mat` and `red`.*

Kết quả của phép tìm này là true.  
*The result of this search is true.*

Một yêu cầu khác cần `red` đứng ngay trước `mat`.  
*Another requirement places `red` immediately before `mat`.*

Yêu cầu này dùng toán tử **FOLLOWED BY**.  
*This requirement uses the **FOLLOWED BY** operator.*

Toán tử này dùng ký hiệu `<->` trong `tsquery`.  
*This operator uses the `<->` symbol in `tsquery`.*

```sql
SELECT to_tsvector('english','a red mat costs as much as a blue mat')
    @@ to_tsquery('english','red <-> mat');   -- red ngay trước mat? -> ở đây false (có "red mat" thì true)
    -- Is red immediately before mat? -> false here ("red mat" would be true)
```

⚠️ `<->` trong `tsquery` có nghĩa là FOLLOWED BY.  
*⚠️ `<->` in `tsquery` means FOLLOWED BY.*

`<->` của vector có nghĩa là L2 distance.  
*`<->` for vectors means L2 distance.*

Hai ngữ cảnh dùng cùng một ký hiệu.  
*Both contexts use the same symbol.*

Hai ngữ cảnh có ý nghĩa khác nhau.  
*The two contexts have different meanings.*

Phần còn lại của bài tập trung vào pgvector.  
*The rest of the lesson focuses on pgvector.*

Nội dung còn lại thuộc semantic search.  
*The remaining content concerns semantic search.*

### 1.4. Query similarity đơn giản nhất
*1.4. The simplest similarity query*

```sql
-- "5 review giống nhất với một embedding cho trước"
-- "5 reviews most similar to a given embedding"
SELECT id, content
FROM reviews
ORDER BY embedding <=> '[...embedding truy vấn...]'   -- <=> cosine distance
                                                     -- <=> cosine distance
LIMIT 5;
```

Mẫu cốt lõi là `ORDER BY <toán tử distance> LIMIT k`.  
*The core pattern is `ORDER BY <distance operator> LIMIT k`.*

`ORDER BY` mặc định dùng thứ tự **ASC**.  
*`ORDER BY` uses **ASC** by default.*

ASC đưa distance nhỏ nhất lên đầu.  
*ASC places the smallest distance first.*

Distance nhỏ nhất biểu thị kết quả giống nhất.  
*The smallest distance represents the most similar result.*

Mẫu này là khung xương của mọi vector query.  
*This pattern forms the backbone of every vector query.*

### ✅ Self-check Phần 1
*✅ Part 1 Self-check*

1. Vì sao embedding trong database phải dùng cùng model?  
   *Why must the database embedding use the same model?*

1a. Vì sao embedding của query phải dùng cùng model?  
   *Why must the query embedding use the same model?*

2. SELECT trực tiếp cột vector cho ra dữ liệu gì?  
   *What data does direct vector column selection return?*

2a. Vì sao query nên tập trung vào distance?  
   *Why should a query focus on distance?*

2b. Vì sao query không nên tập trung vào cột vector?  
   *Why should a query avoid focusing on the vector column?*

3. `<->` trong `tsquery` có nghĩa gì?  
   *What does `<->` mean in `tsquery`?*

3a. `<->` trong vector có nghĩa gì?  
   *What does `<->` mean for vectors?*

3b. Hai ý nghĩa này có giống nhau không?  
   *Are these two meanings the same?*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

### 2.1. Ba toán tử distance trong ngữ cảnh query
*2.1. Three distance operators in query contexts*

| Toán tử / Operator | Metric / Metric | Ghi chú / Notes | Ops class index / Ops class index |
|---|---|---|---|
| `<->` | L2 hoặc Euclidean distance / L2 or Euclidean distance | khoảng cách thẳng, không giới hạn trên / direct distance, no upper bound | `vector_l2_ops` |
| `<#>` | Negative inner product / Negative inner product | pgvector trả IP âm, ASC đưa kết quả gần lên trước / pgvector returns negative IP, ASC places close results first | `vector_ip_ops` |
| `<=>` | Cosine distance / Cosine distance | phổ biến nhất với text, miền `[0,2]` / most common for text, range `[0,2]` | `vector_cosine_ops` |

> **[Đính chính bài gốc]**  
> ***[Correction to the original lesson]***

> Bài gốc gọi query dùng `<=>` là "cosine similarity".  
> *The original lesson calls a query with `<=>` "cosine similarity".*

> `<=>` thực tế trả cosine distance.  
> *`<=>` actually returns cosine distance.*

> `<#>` trả negative inner product.  
> *`<#>` returns negative inner product.*

> `ORDER BY ... ASC` vẫn đưa kết quả gần nhất lên đầu.  
> *`ORDER BY ... ASC` still places the closest result first.*

> Bạn không nên ngạc nhiên trước các giá trị âm.  
> *You should not be surprised by negative values.*

**Chọn metric**  
***Metric selection***

Text embeddings mặc định phù hợp với cosine `<=>`.  
*Text embeddings generally suit cosine `<=>` by default.*

Vector có thể được normalize — chuẩn hóa về length bằng 1.  
*A vector can be normalized to a length of 1.*

Vector dạng này có thể dùng inner product `<#>`.  
*Such a vector can use inner product `<#>`.*

Inner product có thể chạy nhanh hơn.  
*Inner product can run faster.*

L2 phù hợp với vector có độ lớn mang ý nghĩa.  
*L2 suits vectors with meaningful magnitude.*

Giáo trình embeddings trình bày chi tiết lựa chọn này.  
*The embeddings lesson explains this choice in detail.*

### 2.2. Pattern kinh điển: "tìm giống một bản ghi ĐÃ CÓ" (bám ví dụ bài gốc)
*2.2. Classic pattern: "find records similar to an EXISTING record" (following the original example)*

Bài gốc yêu cầu lấy 5 review giống nhất.  
*The original lesson requests the five most similar reviews.*

Review nguồn có `id=1`.  
*The source review has `id=1`.*

Vector query là embedding của một dòng có sẵn.  
*The query vector is the embedding of an existing row.*

Query dùng subquery để lấy embedding này.  
*The query uses a subquery to retrieve this embedding.*

```sql
-- Euclidean
-- Euclidean
SELECT *
FROM reviews
WHERE id != 1                                              -- loại CHÍNH NÓ (distance=0)
                                                           -- exclude ITSELF (distance=0)
ORDER BY embedding <-> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;

-- Inner product
-- Inner product
SELECT *
FROM reviews
WHERE id != 1
ORDER BY embedding <#> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;

-- Cosine
-- Cosine
SELECT *
FROM reviews
WHERE id != 1
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;
```

Có hai điểm quan trọng.  
*There are two important points.*

- **`WHERE id != 1`** loại chính bản ghi nguồn.  
  ***`WHERE id != 1`** excludes the source record itself.*

- Bạn bỏ điều kiện này.  
  *You omit this condition.*

- Review 1 sẽ luôn đứng đầu.  
  *Review 1 will always appear first.*

- Distance từ review 1 tới chính nó bằng 0.  
  *The distance from review 1 to itself equals 0.*

- Kết quả đầu tiên khi đó trở nên vô dụng.  
  *The first result then becomes useless.*

- Bài gốc nêu đúng bước loại bản ghi nguồn.  
  *The original lesson correctly includes source-record exclusion.*

- **Subquery lấy embedding** bằng `(SELECT embedding FROM reviews WHERE id=1)`.  
  ***The subquery retrieves the embedding** with `(SELECT embedding FROM reviews WHERE id=1)`.*

- Embedding của dòng nguồn đi vào phép so sánh.  
  *The source-row embedding enters the comparison.*

### 2.3. Lấy giá trị distance/similarity ra để đọc
*2.3. Retrieve distance and similarity values for inspection*

Bạn thường muốn nhìn thấy điểm số.  
*You usually want to see the score.*

Thứ tự kết quả riêng lẻ chưa cung cấp đủ thông tin.  
*The result order alone does not provide enough information.*

```sql
SELECT id, content,
       embedding <=> :q            AS cosine_distance,     -- 0 = giống hệt, 2 = ngược
                                                               -- 0 = identical, 2 = opposite
       1 - (embedding <=> :q)      AS cosine_similarity     -- 1 = giống hệt, -1 = ngược
                                                               -- 1 = identical, -1 = opposite
FROM reviews
ORDER BY embedding <=> :q
LIMIT 5;
```

> **[MỞ RỘNG]**  
> ***[EXTENSION]***

> Bạn có thể hiển thị dạng "độ giống %".  
> *You can display a "similarity percentage".*

> Bạn dùng `1 - (embedding <=> q)` để lấy cosine similarity.  
> *You use `1 - (embedding <=> q)` to obtain cosine similarity.*

> Bạn nhân kết quả này với `100`.  
> *You multiply this result by `100`.*

> Bạn không nên hiển thị trực tiếp cosine distance.  
> *You should not directly display cosine distance.*

> Bạn không nên gọi cosine distance là "độ giống".  
> *You should not call cosine distance "similarity".*

> Hai khái niệm có hướng nghĩa ngược nhau.  
> *The two concepts have opposite directions.*

> Distance nhỏ biểu thị độ giống cao.  
> *A small distance indicates high similarity.*

### 2.4. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi thường gặp
*2.4. [EXTENSION BEYOND THE ORIGINAL LESSON] — Three common mistakes*

**Lỗi 1 — Quên loại chính bản ghi nguồn**  
***Mistake 1 — Forgetting to exclude the source record***

Bạn có thể quên điều kiện `WHERE id != 1`.  
*You may forget the `WHERE id != 1` condition.*

Kết quả đầu tiên sẽ là chính bản ghi nguồn.  
*The first result will be the source record itself.*

Bản ghi này có distance bằng `0`.  
*This record has a distance of `0`.*

Nó che mất một kết quả hữu ích.  
*It hides one useful result.*

**Lỗi 2 — Ops class index không khớp toán tử query**  
***Mistake 2 — The index ops class does not match the query operator***

Index có thể dùng `vector_cosine_ops`.  
*The index may use `vector_cosine_ops`.*

Query có thể dùng toán tử `<->`.  
*The query may use the `<->` operator.*

Hai lựa chọn này không khớp nhau.  
*These two choices do not match.*

Postgres sẽ bỏ qua index.  
*Postgres will ignore the index.*

Postgres có thể chạy seq scan âm thầm.  
*Postgres may silently run a sequential scan.*

Query sẽ chậm hơn.  
*The query will become slower.*

Toán tử phải khớp với ops class.  
*The operator must match the ops class.*

Bạn nên kiểm tra bằng `EXPLAIN ANALYZE`.  
*You should check with `EXPLAIN ANALYZE`.*

**Lỗi 3 — Nhầm distance với similarity khi diễn giải**  
***Mistake 3 — Confusing distance with similarity during interpretation***

Giá trị `<=>` nhỏ biểu thị độ giống cao.  
*A small `<=>` value indicates high similarity.*

Toán tử `<#>` trả negative inner product.  
*The `<#>` operator returns negative inner product.*

Giá trị này có thể là số âm.  
*This value can be negative.*

Bạn có thể coi giá trị lớn hơn là giống hơn.  
*You may treat a larger value as greater similarity.*

Bạn có thể dùng `ORDER BY DESC`.  
*You may use `ORDER BY DESC`.*

Cách sắp xếp này đảo ngược kết quả.  
*This ordering reverses the results.*

Thứ tự mặc định `ASC` phù hợp cho cả ba toán tử.  
*The default `ASC` order works for all three operators.*

pgvector được thiết kế theo quy tắc này.  
*pgvector follows this design rule.*

### ✅ Self-check Phần 2
*✅ Part 2 self-check*

1. Vì sao query tìm review giống `id=1` cần `WHERE id != 1`?  
*1. Why does a query for reviews similar to `id=1` need `WHERE id != 1`?*

2. Bạn đổi cosine distance thành "độ giống %" như thế nào?  
*2. How do you convert cosine distance into a similarity percentage?*

3. Toán tử `<#>` trả về giá trị gì?  
*3. What value does the `<#>` operator return?*

4. Vì sao query vẫn dùng `ORDER BY ASC`?  
*4. Why does the query still use `ORDER BY ASC`?*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED*

### 3.1. Threshold filtering — chỉ nhận kết quả "đủ gần"
*3.1. Threshold filtering — Accept only sufficiently close results*

`ORDER BY ... LIMIT 5` luôn trả tối đa 5 dòng.  
*`ORDER BY ... LIMIT 5` always returns up to five rows.*

Năm dòng này có thể không liên quan.  
*These five rows may be irrelevant.*

Chúng chỉ là những dòng ít xa nhất.  
*They are only the least distant rows.*

Threshold khắc phục vấn đề này.  
*A threshold solves this problem.*

Threshold chỉ chấp nhận kết quả trong ngưỡng.  
*A threshold accepts only results within the boundary.*

Distance có thể được dùng làm điều kiện.  
*Distance can serve as the condition.*

Similarity cũng có thể được dùng làm điều kiện.  
*Similarity can also serve as the condition.*

```sql
-- Chỉ nhận review có cosine distance < 0.3 (khá giống), tối đa 5
-- Accept only reviews with cosine distance < 0.3, with a maximum of 5
SELECT id, content, embedding <=> :q AS distance
FROM reviews
WHERE embedding <=> :q < 0.3          -- threshold
                                           -- threshold
ORDER BY embedding <=> :q
LIMIT 5;
```

Bài gốc dùng một ví dụ L2.  
*The original lesson uses an L2 example.*

Ví dụ dùng `WHERE embedding <-> q < 5`.  
*The example uses `WHERE embedding <-> q < 5`.*

Ý tưởng này đúng.  
*This idea is correct.*

Con số `5` cần được xem xét kỹ.  
*The number `5` needs careful evaluation.*

Mục 3.2 giải thích vấn đề này.  
*Section 3.2 explains this issue.*

### 3.2. [QUAN TRỌNG] Threshold phụ thuộc METRIC và DATASET
*3.2. [IMPORTANT] Threshold depends on the METRIC and DATASET*

Không có một ngưỡng vạn năng.  
*No universal threshold exists.*

Mỗi metric có một miền giá trị riêng.  
*Each metric has its own value range.*

| Metric / Metric | Miền giá trị / Value range | "Gần" nghĩa là / Meaning of "close" |
|---|---|---|
| Cosine distance (`<=>`) / Cosine distance (`<=>`) | **[0, 2]** / **[0, 2]** | thường `< 0.2–0.4` là rất giống / values below `0.2–0.4` often indicate high similarity |
| L2 (`<->`) / L2 (`<->`) | **[0, ∞)** không giới hạn / **[0, ∞)** without an upper bound | phụ thuộc thang và trạng thái normalize / depends on scale and normalization |
| Negative inner product (`<#>`) / Negative inner product (`<#>`) | phụ thuộc magnitude / depends on magnitude | khó đặt ngưỡng trước khi normalize / difficult to threshold before normalization |

Threshold `< 5` cho L2 chỉ phù hợp với dữ liệu cụ thể.  
*An L2 threshold below `5` fits only specific data.*

Threshold đó cũng phụ thuộc vào model cụ thể.  
*That threshold also depends on a specific model.*

Bạn có thể đổi model.  
*You may change the model.*

Bạn có thể thay đổi cách normalize.  
*You may change the normalization method.*

Khi đó con số cũ sẽ mất ý nghĩa.  
*The old number will then lose its meaning.*

Cosine distance có miền cố định `[0,2]`.  
*Cosine distance has a fixed `[0,2]` range.*

Đặc điểm này giúp đặt ngưỡng dễ hơn.  
*This property makes threshold selection easier.*

Ngưỡng cosine thường ổn định hơn ngưỡng L2.  
*Cosine thresholds are usually more stable than L2 thresholds.*

Bài gốc gợi ý một cách hiệu chỉnh.  
*The original lesson suggests a calibration method.*

Gợi ý đó chưa được giải thích rõ.  
*That suggestion lacks a clear explanation.*

Bạn chọn vài cặp biết trước là giống.  
*You select several known similar pairs.*

Bạn chọn vài cặp biết trước là khác.  
*You select several known dissimilar pairs.*

Bạn chạy query trên các cặp này.  
*You run queries on these pairs.*

Bạn ghi lại distance của từng cặp.  
*You record the distance of each pair.*

Hai nhóm sẽ tạo ra hai khoảng giá trị.  
*The two groups will produce two value ranges.*

Bạn chọn đường biên giữa hai nhóm.  
*You choose the boundary between the two groups.*

Bài gốc đề cập việc lấy cả bản ghi rất ít giống.  
*The original lesson mentions retrieving very dissimilar records.*

Bài gốc cũng đề cập bản ghi không giống.  
*The original lesson also mentions retrieving unrelated records.*

Mục đích là validate kết quả.  
*The purpose is result validation.*

Mục đích sâu hơn là calibrate ngưỡng.  
*The deeper purpose is threshold calibration.*

Các bản ghi đó không dành cho người dùng.  
*Those records are not intended for users.*

### 3.3. [BẪY INDEX] Threshold thuần không dùng được ANN index
*3.3. [INDEX TRAP] A bare threshold cannot use the ANN index*

Bài gốc bỏ qua bẫy này.  
*The original lesson omits this trap.*

```sql
-- ✅ Dùng ANN index (k-NN): ORDER BY + LIMIT
-- ✅ Use the ANN index for k-NN with ORDER BY and LIMIT
SELECT * FROM reviews ORDER BY embedding <=> :q LIMIT 5;

-- ✅ Vẫn dùng index: threshold + ORDER BY + LIMIT
-- ✅ Still use the index with threshold, ORDER BY, and LIMIT
SELECT * FROM reviews
WHERE embedding <=> :q < 0.3
ORDER BY embedding <=> :q
LIMIT 5;

-- ⚠️ CÓ THỂ seq scan: threshold THUẦN, không ORDER BY/LIMIT
-- ⚠️ MAY use a sequential scan with a bare threshold and no ORDER BY/LIMIT
SELECT * FROM reviews WHERE embedding <=> :q < 0.3;   -- tính distance MỌI dòng
                                                          -- calculate distance for EVERY row
```

HNSW được thiết kế cho k-nearest-neighbor.  
*HNSW is designed for k-nearest-neighbor search.*

IVFFlat cũng được thiết kế cho k-nearest-neighbor.  
*IVFFlat is also designed for k-nearest-neighbor search.*

Pattern này dùng `ORDER BY distance LIMIT k`.  
*This pattern uses `ORDER BY distance LIMIT k`.*

Các index này không dành cho range scan thuần.  
*These indexes are not designed for bare range scans.*

Điều kiện `WHERE distance < x` có thể đứng một mình.  
*The `WHERE distance < x` condition may appear alone.*

Trong trường hợp đó, Postgres phải tính distance cho mọi dòng.  
*In that case, Postgres must calculate distance for every row.*

Bạn muốn query nhanh.  
*You want a fast query.*

Bạn cũng muốn có threshold.  
*You also want a threshold.*

Bạn phải kèm `ORDER BY distance LIMIT k`.  
*You must include `ORDER BY distance LIMIT k`.*

Sau đó query mới lọc threshold.  
*The query then applies the threshold filter.*

Bạn nên xác nhận bằng `EXPLAIN ANALYZE`.  
*You should verify this with `EXPLAIN ANALYZE`.*

Kế hoạch thực thi cần hiển thị `Index Scan`.  
*The execution plan should show `Index Scan`.*

### 3.4. Threshold + filter + overfiltering
*3.4. Threshold, filter, and overfiltering*

Bạn có thể kết hợp threshold với filter nghiệp vụ.  
*You can combine a threshold with a business filter.*

Ví dụ là `WHERE category=...`.  
*An example is `WHERE category=...`.*

Cách kết hợp này có thể gây overfiltering.  
*This combination can cause overfiltering.*

ANN lấy các hàng xóm theo `ef_search` trước.  
*ANN first retrieves neighbors according to `ef_search`.*

Filter nghiệp vụ được áp dụng sau.  
*The business filter is applied afterward.*

Kết quả cuối có thể bị thiếu.  
*The final result set may be incomplete.*

Iterative scan có thể khắc phục vấn đề này.  
*Iterative scan can address this problem.*

pgvector 0.8+ hỗ trợ cơ chế này.  
*pgvector 0.8+ supports this mechanism.*

Giáo trình indexing giải thích chi tiết hơn.  
*The indexing lesson provides more detail.*

```sql
SET hnsw.iterative_scan = relaxed_order;
```

### 3.5. Edge cases
*3.5. Edge cases*

**Threshold quá chặt**  
***An overly strict threshold***

Threshold quá chặt có thể trả `0` kết quả.  
*An overly strict threshold may return zero results.*

Kết quả này vẫn hợp lệ.  
*This result is still valid.*

Không có bản ghi nào đủ gần.  
*No record is sufficiently close.*

Ứng dụng phải xử lý trạng thái này.  
*The application must handle this state.*

Ứng dụng nên báo "không tìm thấy".  
*The application should report "no results found".*

Ứng dụng không nên coi đây là lỗi.  
*The application should not treat this as an error.*

**Đổi model**  
***Changing the model***

Ngưỡng cũ có thể trở nên sai.  
*The old threshold may become invalid.*

Ngưỡng gắn với model.  
*The threshold is tied to the model.*

Ngưỡng cũng gắn với metric.  
*The threshold is also tied to the metric.*

Bạn phải re-embed sau khi đổi model.  
*You must re-embed after a model change.*

Bạn cũng phải hiệu chỉnh lại ngưỡng.  
*You must also recalibrate the threshold.*

**`<#>` với vector chưa normalize**  
***`<#>` with unnormalized vectors***

Magnitude có thể làm nhiễu inner product.  
*Magnitude can distort the inner product.*

Ngưỡng sẽ khó đáng tin cậy.  
*The threshold will be difficult to trust.*

Bạn nên normalize vector trước.  
*You should normalize the vectors first.*

**Vector NULL**  
***NULL vectors***

Vector `NULL` không xuất hiện trong kết quả distance.  
*A `NULL` vector does not appear in distance results.*

Bạn cần lọc các dòng này.  
*You need to filter these rows.*

Bạn cũng có thể backfill embedding.  
*You can also backfill the embeddings.*

**So similarity với distance trong `WHERE`**  
***Comparing similarity with distance in `WHERE`***

Bạn có thể viết ngưỡng similarity.  
*You can write a similarity threshold.*

Ví dụ là `WHERE 1-(embedding<=>q) > 0.7`.  
*An example is `WHERE 1-(embedding<=>q) > 0.7`.*

Điều kiện này tương đương một ngưỡng distance.  
*This condition equals a distance threshold.*

Ngưỡng tương đương là `WHERE embedding<=>q < 0.3`.  
*The equivalent threshold is `WHERE embedding<=>q < 0.3`.*

Cách viết theo distance hỗ trợ planner tốt hơn.  
*The distance form helps the planner more effectively.*

### ✅ Self-check Phần 3
*✅ Part 3 self-check*

1. Vì sao không có threshold vạn năng?  
*1. Why is there no universal threshold?*

2. Metric nào hỗ trợ ngưỡng ổn định hơn?  
*2. Which metric supports a more stable threshold?*

3. `WHERE embedding <=> q < 0.3` có dùng ANN index không?  
*3. Does `WHERE embedding <=> q < 0.3` use the ANN index?*

4. Query trên thiếu `ORDER BY` và `LIMIT`.  
*4. The query above lacks `ORDER BY` and `LIMIT`.*

5. Bạn nên viết query đó như thế nào?  
*5. How should you write that query?*

6. Bạn hiệu chỉnh threshold bằng mẫu giống như thế nào?  
*6. How do you calibrate a threshold with similar samples?*

7. Bạn hiệu chỉnh threshold bằng mẫu khác như thế nào?  
*7. How do you calibrate a threshold with dissimilar samples?*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL: Systems thinking and technical leadership*

### 4.1. Query pattern ở quy mô lớn
*4.1. Query patterns at large scale*

**Pre-filter trước khi rank**  
***Pre-filter before ranking***

Bạn đặt các filter có tính chọn lọc.  
*You apply selective filters.*

Ví dụ là `tenant_id`.  
*One example is `tenant_id`.*

Ví dụ khác là `category`.  
*Another example is `category`.*

Thời gian cũng có thể là một filter.  
*Time can also be a filter.*

Các filter thu hẹp tập ứng viên trước.  
*These filters reduce the candidate set first.*

Tập ứng viên nhỏ làm giảm chi phí distance.  
*A smaller candidate set reduces distance cost.*

Partition có thể hỗ trợ quá trình này.  
*Partitioning can support this process.*

Planner có thể prune dữ liệu sớm.  
*The planner can prune data early.*

**Hybrid query**  
***Hybrid query***

Hybrid query kết hợp vector search với full-text search.  
*A hybrid query combines vector search with full-text search.*

Vector search có thể dùng `<=>`.  
*Vector search can use `<=>`.*

Full-text search có thể dùng `@@`.  
*Full-text search can use `@@`.*

RRF có thể fuse hai danh sách kết quả.  
*RRF can fuse the two result lists.*

Kết quả thường tốt hơn keyword-only.  
*The result is often better than keyword-only search.*

Kết quả cũng thường tốt hơn semantic-only.  
*The result is also often better than semantic-only search.*

Giáo trình FTS giải thích phần này.  
*The FTS lesson explains this topic.*

Đây là default retrieval năm 2026.  
*This is the default retrieval approach in 2026.*

**Pagination vector kết quả**  
***Pagination for vector results***

`ORDER BY distance LIMIT k OFFSET n` vẫn hoạt động.  
*`ORDER BY distance LIMIT k OFFSET n` still works.*

OFFSET lớn gây tốn kém với ANN.  
*A large OFFSET is expensive with ANN.*

Hệ thống thực tế thường chỉ lấy top-k.  
*Real systems usually retrieve only the top-k.*

Sau đó hệ thống chạy rerank.  
*The system then performs reranking.*

Hệ thống thường tránh phân trang sâu.  
*The system usually avoids deep pagination.*

**Cache query embedding**  
***Cache query embeddings***

Cùng một câu truy vấn tạo ra cùng một embedding.  
*The same query produces the same embedding.*

Bạn có thể cache embedding theo hash.  
*You can cache the embedding by hash.*

Cách này tránh gọi lại model.  
*This approach avoids repeated model calls.*

Nó hữu ích trên đường xử lý nóng.  
*It is useful on the hot path.*

### 4.2. Hiệu chỉnh threshold một cách khoa học
*4.2. Calibrating thresholds scientifically*

Đây là phần nâng cấp từ ý bài gốc.  
*This section expands the original lesson's idea.*

Bài gốc gợi ý lấy bản ghi không giống để validate.  
*The original lesson suggests retrieving dissimilar records for validation.*

Gợi ý này còn mơ hồ.  
*This suggestion remains vague.*

Ở cấp staff, quy trình phải có hệ thống.  
*At staff level, the process must be systematic.*

1. Bạn dựng một tập vàng.  
*1. You build a golden set.*

Tập vàng chứa các cặp query–kết quả.  
*The golden set contains query-result pairs.*

Mỗi cặp đã được gán nhãn.  
*Each pair has a label.*

Một nhãn biểu thị liên quan.  
*One label indicates relevance.*

Một nhãn biểu thị không liên quan.  
*Another label indicates irrelevance.*

2. Bạn chạy query trên tập vàng.  
*2. You run queries on the golden set.*

Bạn ghi lại distance của cặp liên quan.  
*You record distances for relevant pairs.*

Bạn ghi lại distance của cặp không liên quan.  
*You record distances for irrelevant pairs.*

Bạn vẽ phân phối cho từng nhóm.  
*You plot a distribution for each group.*

3. Bạn chọn threshold tốt nhất.  
*3. You choose the best threshold.*

Threshold cần tách hai phân phối.  
*The threshold should separate the two distributions.*

Lựa chọn này tạo ra trade-off.  
*This choice creates a trade-off.*

Ngưỡng chặt làm tăng precision.  
*A strict threshold increases precision.*

Ngưỡng chặt cũng làm tăng bỏ sót.  
*A strict threshold also increases missed matches.*

Recall sẽ thấp hơn.  
*Recall will become lower.*

Ngưỡng lỏng làm tăng recall.  
*A loose threshold increases recall.*

Ngưỡng lỏng cũng đưa thêm kết quả rác.  
*A loose threshold also admits more irrelevant results.*

Bạn chọn theo nhu cầu sản phẩm.  
*You choose according to product needs.*

4. Bạn theo dõi threshold theo thời gian.  
*4. You monitor the threshold over time.*

Dữ liệu thay đổi sẽ yêu cầu hiệu chỉnh threshold.  
*Data changes require threshold recalibration.*

Model thay đổi cũng yêu cầu hiệu chỉnh threshold.  
*Model changes also require threshold recalibration.*

> **One-liner staff:**  
> ***Staff one-liner:***

> Threshold không phải một con số phép màu để đoán.  
> *A threshold is not a magic number for guessing.*

> Nó là đường biên được hiệu chỉnh bằng các cặp có nhãn.  
> *It is a boundary calibrated with labeled pairs.*

> Các cặp đó gồm mẫu giống và mẫu khác.  
> *Those pairs include similar and dissimilar samples.*

### 4.3. Correctness ở quy mô: ANN + threshold
*4.3. Correctness at scale: ANN plus threshold*

ANN là approximate.  
*ANN is approximate.*

Threshold được áp dụng trên kết quả xấp xỉ.  
*The threshold is applied to approximate results.*

ANN có thể bỏ sót một bản ghi phù hợp.  
*ANN may miss a qualifying record.*

Bản ghi đó có thể nằm trong ngưỡng thật.  
*That record may fall within the true threshold.*

ANN có thể không tìm ra bản ghi đó.  
*ANN may fail to retrieve that record.*

Một số hệ thống cần bảo đảm cao hơn.  
*Some systems need stronger guarantees.*

Compliance là một ví dụ.  
*Compliance is one example.*

Two-stage retrieval là một lựa chọn.  
*Two-stage retrieval is one option.*

Giai đoạn đầu dùng ANN với top-N rộng.  
*The first stage uses ANN with a broad top-N.*

Giai đoạn sau tính exact distance.  
*The second stage calculates exact distance.*

Giai đoạn sau áp dụng threshold.  
*The second stage applies the threshold.*

Bạn cũng phải monitor recall.  
*You must also monitor recall.*

Giáo trình indexing trình bày cách đo recall.  
*The indexing lesson explains recall measurement.*

Threshold không thể sửa một index có recall kém.  
*A threshold cannot fix an index with poor recall.*

### 4.4. Cost / latency / monitoring
*4.4. Cost, latency, and monitoring*

**Latency**  
***Latency***

`ef_search` cao thường cải thiện recall.  
*A high `ef_search` usually improves recall.*

`ef_search` cao cũng làm query chậm hơn.  
*A high `ef_search` also makes queries slower.*

Threshold không thay thế việc tune `ef_search`.  
*A threshold does not replace `ef_search` tuning.*

Bạn cần đo p95.  
*You need to measure p95.*

Bạn cũng cần đo p99.  
*You also need to measure p99.*

**Cost**  
***Cost***

Mỗi query real-time cần embedding cho câu truy vấn.  
*Each real-time query needs a query embedding.*

Quá trình này cần một model.  
*This process requires a model.*

Mỗi query cũng cần chạy search.  
*Each query also requires a search operation.*

Bạn nên cache embedding.  
*You should cache embeddings.*

Bạn nên giữ top-k nhỏ.  
*You should keep top-k small.*

**Monitoring**  
***Monitoring***

Bạn nên theo dõi phân phối distance theo thời gian.  
*You should track distance distributions over time.*

Drift có thể làm threshold cần thay đổi.  
*Drift may require a threshold change.*

Bạn nên theo dõi tỉ lệ query trả `0` kết quả.  
*You should track the rate of zero-result queries.*

Tỉ lệ cao có thể cho thấy threshold quá chặt.  
*A high rate may indicate an overly strict threshold.*

Bạn nên theo dõi tỉ lệ seq-scan fallback.  
*You should track the sequential-scan fallback rate.*

Query viết sai có thể làm mất index.  
*An incorrect query can bypass the index.*

### 4.5. Ảnh hưởng tổ chức và giải thích cho non-technical stakeholder
*4.5. Organizational impact and explanations for non-technical stakeholders*

**Nói với PM hoặc sếp**  
***Explanation for a PM or manager***

Kết quả search luôn được xếp hạng theo độ giống.  
*Search results are always ranked by similarity.*

`LIMIT` yêu cầu năm kết quả gần nhất.  
*`LIMIT` requests the five nearest results.*

Năm kết quả gần nhất chưa chắc là năm kết quả tốt.  
*The five nearest results are not necessarily good results.*

Threshold là một bộ lọc chất lượng.  
*A threshold is a quality filter.*

Threshold chỉ trả kết quả đủ giống.  
*The threshold returns only sufficiently similar results.*

Hệ thống có thể không trả kết quả.  
*The system may return no result.*

Lựa chọn này tốt hơn việc trả kết quả rác.  
*This choice is better than returning irrelevant results.*

Con số threshold được hiệu chỉnh trên dữ liệu thật.  
*The threshold value is calibrated on real data.*

Con số này không được đoán bừa.  
*This value is not guessed.*

Bạn phải chỉnh lại threshold sau khi đổi model AI.  
*You must recalibrate the threshold after changing the AI model.*

Cách giải thích nên tập trung vào chất lượng kết quả.  
*The explanation should focus on result quality.*

Cách giải thích cũng nên nêu trade-off precision/recall.  
*The explanation should also mention the precision-recall trade-off.*

**Roadmap**  
***Roadmap***

Golden set là một tài sản có thể tái sử dụng.  
*A golden set is a reusable asset.*

Golden set hỗ trợ hiệu chỉnh threshold.  
*The golden set supports threshold calibration.*

Golden set cũng hỗ trợ đo recall.  
*The golden set also supports recall measurement.*

Golden set giúp so sánh các model.  
*The golden set helps compare models.*

Tổ chức nên đầu tư xây dựng golden set.  
*The organization should invest in building a golden set.*

Tổ chức cũng nên duy trì golden set.  
*The organization should also maintain the golden set.*

**UX**  
***UX***

Trả `0` kết quả là một quyết định sản phẩm.  
*Returning zero results is a product decision.*

Quyết định này áp dụng cho trạng thái không có kết quả đủ gần.  
*This decision applies to a state with no sufficiently close result.*

Im lặng có thể tốt hơn gợi ý rác.  
*Silence may be better than an irrelevant suggestion.*

Staff engineer nên nêu rõ quyết định này với PM.  
*A staff engineer should clarify this decision with the PM.*

### 4.6. Câu hỏi system design mẫu + hướng trả lời staff
*4.6. Sample system design question and staff-level answer framework*

> **"Xây tính năng tìm sản phẩm tương tự cho e-commerce."**  
> ***"Build a similar-product feature for e-commerce."***

> Tính năng hiển thị tối đa 8 sản phẩm thực sự giống.  
> *The feature displays at most eight truly similar products.*

> Tính năng không hiển thị rác trong trạng thái thiếu kết quả phù hợp.  
> *The feature shows no irrelevant items without a valid match.*

> Tính năng lọc sản phẩm còn hàng.  
> *The feature filters for in-stock products.*

> Tính năng lọc sản phẩm cùng category.  
> *The feature filters for products in the same category.*

> Mục tiêu p95 nhỏ hơn `100ms`.  
> *The p95 target is below `100ms`.*

**Khung trả lời staff:**  
***Staff-level answer framework:***

1. **Clarify**  
*1. **Clarify***

Bạn cần xác định metric.  
*You need to identify the metric.*

Cosine phù hợp với embedding sản phẩm.  
*Cosine suits product embeddings.*

Bạn cần hỏi về golden set.  
*You need to ask about the golden set.*

Golden set hỗ trợ calibrate threshold.  
*The golden set supports threshold calibration.*

Bạn cần biết tần suất đổi model.  
*You need to know the model-change frequency.*

2. **Query shape**  
*2. **Query shape***

`WHERE in_stock AND category=:c AND id!=:self AND embedding <=> :q < :threshold ORDER BY embedding <=> :q LIMIT 8`  
*`WHERE in_stock AND category=:c AND id!=:self AND embedding <=> :q < :threshold ORDER BY embedding <=> :q LIMIT 8`*

Threshold ngăn kết quả rác.  
*The threshold blocks irrelevant results.*

`LIMIT` giới hạn số lượng.  
*`LIMIT` caps the result count.*

Filter nghiệp vụ áp dụng điều kiện sản phẩm.  
*Business filters apply product constraints.*

Iterative scan chống overfiltering.  
*Iterative scan reduces overfiltering.*

3. **Threshold**  
*3. **Threshold***

Bạn hiệu chỉnh threshold trên golden set.  
*You calibrate the threshold on the golden set.*

Golden set chứa các cặp giống.  
*The golden set contains similar pairs.*

Golden set cũng chứa các cặp khác.  
*The golden set also contains dissimilar pairs.*

Bạn chọn ngưỡng cân bằng precision với recall.  
*You choose a balance between precision and recall.*

Bạn version hóa threshold theo model.  
*You version the threshold by model.*

4. **Index & latency**  
*4. **Index and latency***

Bạn dùng HNSW với `vector_cosine_ops`.  
*You use HNSW with `vector_cosine_ops`.*

Bạn tune `ef_search` theo p95.  
*You tune `ef_search` according to p95.*

Query phải có `ORDER BY <=> LIMIT`.  
*The query must include `ORDER BY <=> LIMIT`.*

Pattern này giúp dùng index.  
*This pattern enables index usage.*

Bạn không dùng threshold thuần.  
*You do not use a bare threshold.*

5. **Correctness**  
*5. **Correctness***

Nhu cầu bảo đảm cao phù hợp với two-stage retrieval.  
*Stronger guarantees suit two-stage retrieval.*

ANN lấy một tập top-N rộng.  
*ANN retrieves a broad top-N set.*

Giai đoạn exact tính lại distance.  
*The exact stage recalculates distance.*

Giai đoạn exact áp dụng threshold.  
*The exact stage applies the threshold.*

6. **UX**  
*6. **UX***

Threshold có thể để lại `0` sản phẩm.  
*The threshold may leave zero products.*

Giao diện sẽ ẩn khối sản phẩm tương tự.  
*The interface will hide the similar-products section.*

Giao diện không hiển thị kết quả rác.  
*The interface does not show irrelevant results.*

7. **Monitor**  
*7. **Monitor***

Bạn monitor phân phối distance.  
*You monitor distance distributions.*

Bạn monitor tỉ lệ query có `0` kết quả.  
*You monitor the zero-result query rate.*

Bạn monitor p99.  
*You monitor p99.*

Bạn monitor recall trên golden set.  
*You monitor recall on the golden set.*

8. **Tự đánh giá**  
*8. **Self-evaluation***

`LIMIT` giới hạn số lượng.  
*`LIMIT` caps quantity.*

Threshold bảo đảm chất lượng.  
*The threshold protects quality.*

Hai cơ chế phối hợp tạo trải nghiệm tốt.  
*The two mechanisms create a good experience together.*

Bạn phải phân biệt "gần nhất" với "đủ tốt".  
*You must distinguish "nearest" from "good enough".*

Đây là tư duy cấp staff.  
*This is staff-level thinking.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 INTERVIEW CHEATSHEET*

### 5.1. Keywords bắt buộc nhớ
*5.1. Required keywords*

- **Same model / same length** là yêu cầu bắt buộc.  
  ***Same model / same length** is a mandatory requirement.*

- Yêu cầu này giúp distance có ý nghĩa.  
  *This requirement makes distance meaningful.*

- **`<->` / `<#>` / `<=>`** đại diện cho ba phép đo.  
  ***`<->` / `<#>` / `<=>`** represent three metrics.*

- Ba phép đo là L2, negative inner product và cosine distance.  
  *The three metrics are L2, negative inner product, and cosine distance.*

- **Ops class** gồm `vector_l2_ops`, `vector_ip_ops` và `vector_cosine_ops`.  
  ***Ops class** includes `vector_l2_ops`, `vector_ip_ops`, and `vector_cosine_ops`.*

- Ops class phải khớp với toán tử.  
  *The ops class must match the operator.*

- **k-NN query** dùng `ORDER BY distance LIMIT k`.  
  *A **k-NN query** uses `ORDER BY distance LIMIT k`.*

- Đây là khung xương của mọi vector query.  
  *This is the backbone of every vector query.*

- **Cosine distance vs similarity** là một điểm dễ nhầm.  
  ***Cosine distance vs similarity** is a common source of confusion.*

- `<=>` trả distance trong miền `[0,2]`.  
  *`<=>` returns distance in the `[0,2]` range.*

- Similarity bằng `1 - distance`.  
  *Similarity equals `1 - distance`.*

- **Threshold** lọc các kết quả đủ gần.  
  *A **threshold** filters sufficiently close results.*

- Threshold phụ thuộc metric.  
  *A threshold depends on the metric.*

- Threshold cũng phụ thuộc dataset.  
  *A threshold also depends on the dataset.*

- **`WHERE id != self`** loại bản ghi nguồn.  
  ***`WHERE id != self`** excludes the source record.*

- Điều kiện này dùng cho truy vấn tìm bản ghi giống nó.  
  *This condition applies to searches for records similar to themselves.*

- **Overfiltering** có thể làm thiếu kết quả.  
  ***Overfiltering** can cause missing results.*

- **Iterative scan** hỗ trợ filter chặt.  
  ***Iterative scan** supports restrictive filters.*

- Cấu hình liên quan là `hnsw.iterative_scan`.  
  *The related setting is `hnsw.iterative_scan`.*

- **Two-stage retrieval** dùng ANN ở giai đoạn đầu.  
  ***Two-stage retrieval** uses ANN in the first stage.*

- Giai đoạn sau dùng exact distance.  
  *The later stage uses exact distance.*

- Giai đoạn sau cũng áp dụng threshold.  
  *The later stage also applies the threshold.*

- Cách này tăng correctness.  
  *This approach improves correctness.*

- **Golden set** là một tập dữ liệu có nhãn.  
  *A **golden set** is a labeled dataset.*

- Golden set hỗ trợ hiệu chỉnh threshold.  
  *The golden set supports threshold calibration.*

- Golden set cũng hỗ trợ đo recall.  
  *The golden set also supports recall measurement.*

- **tsquery vs vector** biểu thị hai tầng tìm kiếm khác nhau.  
  ***tsquery vs vector** represents two different search levels.*

- tsquery hoạt động trên lexeme.  
  *tsquery operates on lexemes.*

- Vector search hoạt động trên semantic meaning.  
  *Vector search operates on semantic meaning.*

- Ký hiệu `<->` có hai nghĩa theo ngữ cảnh.  
  *The `<->` symbol has two context-dependent meanings.*

### 5.2. Core concepts — nếu chỉ nhớ vài điều
*5.2. Core concepts — Remember only these ideas*

1. Vector query dùng `ORDER BY <distance> LIMIT k`.  
*1. A vector query uses `ORDER BY <distance> LIMIT k`.*

Thứ tự `ASC` đưa kết quả gần nhất lên đầu.  
*The `ASC` order places the nearest result first.*

2. Bạn không nên `SELECT` raw vector.  
*2. You should not `SELECT` raw vectors.*

Raw vector không có ý nghĩa trực tiếp với con người.  
*Raw vectors have no direct meaning for humans.*

Query nên xoay quanh distance.  
*The query should focus on distance.*

Query cũng có thể xoay quanh similarity.  
*The query may also focus on similarity.*

3. Database embedding phải dùng cùng model với query embedding.  
*3. Database embeddings must use the same model as query embeddings.*

Hai phía cũng phải có cùng length.  
*Both sides must also have the same length.*

Vi phạm hai yêu cầu này làm kết quả vô nghĩa.  
*Violating these requirements makes the results meaningless.*

4. `<=>` là cosine distance.  
*4. `<=>` is cosine distance.*

Distance nhỏ biểu thị kết quả gần.  
*A small distance indicates a close result.*

Similarity bằng `1 - (<=>)`.  
*Similarity equals `1 - (<=>)`.*

5. Truy vấn có thể tìm các bản ghi giống bản ghi X.  
*5. A query can find records similar to record X.*

Query dùng subquery để lấy embedding của X.  
*The query uses a subquery to retrieve the embedding of X.*

Query cũng cần `WHERE id != X`.  
*The query also needs `WHERE id != X`.*

6. Threshold là bộ lọc chất lượng.  
*6. A threshold is a quality filter.*

`LIMIT` là giới hạn số lượng.  
*`LIMIT` is a quantity cap.*

Hai cơ chế giải quyết hai vấn đề khác nhau.  
*The two mechanisms solve different problems.*

7. Ngưỡng phụ thuộc metric.  
*7. A threshold depends on the metric.*

Ngưỡng cũng phụ thuộc dataset.  
*The threshold also depends on the dataset.*

Cosine distance có miền `[0,2]`.  
*Cosine distance has a `[0,2]` range.*

Cosine thường hỗ trợ ngưỡng ổn định hơn L2.  
*Cosine usually supports more stable thresholds than L2.*

L2 có miền trên không giới hạn.  
*L2 has no upper bound.*

8. Threshold thuần không dùng ANN index.  
*8. A bare threshold does not use the ANN index.*

Query nên kèm `ORDER BY distance LIMIT k`.  
*The query should include `ORDER BY distance LIMIT k`.*

9. ANN là approximate.  
*9. ANN is approximate.*

Threshold được áp dụng trên kết quả xấp xỉ.  
*The threshold is applied to approximate results.*

Two-stage exact phù hợp khi cần bảo đảm cao hơn.  
*Two-stage exact retrieval suits stronger guarantees.*

10. Golden set hỗ trợ hiệu chỉnh threshold.  
*10. A golden set supports threshold calibration.*

Golden set chứa các cặp giống có nhãn.  
*The golden set contains labeled similar pairs.*

Golden set cũng chứa các cặp khác có nhãn.  
*The golden set also contains labeled dissimilar pairs.*

Bạn không nên đoán ngưỡng.  
*You should not guess the threshold.*

### 5.3. Ideas / mental models
*5.3. Ideas and mental models*

- **"Gần nhất ≠ đủ tốt"** là mental model về LIMIT và threshold.  
  ***"Nearest ≠ good enough"** is a mental model for LIMIT and threshold.*

- `<=>` nhỏ biểu thị kết quả gần.  
  *A small `<=>` value indicates a close result.*

- `<#>` trả giá trị âm.  
  *`<#>` returns a negative value.*

- Bạn không nên đảo chiều sắp xếp.  
  *You should not reverse the sort order.*

- **"Ngưỡng là đường biên quyết định"** là mental model quan trọng.  
  ***"A threshold is a decision boundary"** is an important mental model.*

- Ngưỡng không phải một số phép màu.  
  *A threshold is not a magic number.*

- Bạn phải calibrate ngưỡng trên dữ liệu.  
  *You must calibrate the threshold on data.*

- **k-NN dùng index.**  
  ***k-NN uses an index.***

- **Range thuần dùng seq scan.**  
  ***A bare range query uses a sequential scan.***

- Query luôn cần `ORDER BY + LIMIT`.  
  *The query always needs `ORDER BY + LIMIT`.*

- Vector từ hai model không thể so sánh.  
  *Vectors from two models cannot be compared.*

### 5.4. Code cần thuộc lòng
*5.4. Code to memorize*

**(a) k-NN cơ bản:**  
***(a) Basic k-NN:***

```sql
SELECT id, content FROM reviews ORDER BY embedding <=> :q LIMIT 5;
```

**(b) Giống một bản ghi có sẵn:**  
***(b) Similar to an existing record:***

Query loại chính bản ghi nguồn.  
*The query excludes the source record.*

```sql
SELECT * FROM reviews
WHERE id != 1
ORDER BY embedding <=> (SELECT embedding FROM reviews WHERE id = 1)
LIMIT 5;
```

**(c) Threshold + LIMIT + similarity %:**  
***(c) Threshold + LIMIT + similarity percentage:***

Query này giữ index.  
*This query keeps the index usable.*

```sql
SELECT id, content, 1 - (embedding <=> :q) AS similarity
FROM reviews
WHERE embedding <=> :q < 0.3          -- threshold
                                           -- threshold
ORDER BY embedding <=> :q
LIMIT 5;
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions and suggested answers*

1. **"Viết query tìm 5 bản ghi giống nhất?"**  
*1. **"Write a query for the five most similar records."***

Dùng `SELECT ... ORDER BY embedding <=> :q LIMIT 5`.  
*Use `SELECT ... ORDER BY embedding <=> :q LIMIT 5`.*

Thứ tự `ASC` đưa kết quả gần nhất lên đầu.  
*The `ASC` order places the nearest result first.*

Query cũng có thể tìm các bản ghi giống một dòng có sẵn.  
*The query can also find records similar to an existing row.*

Trong trường hợp đó, query dùng subquery embedding.  
*In that case, the query uses an embedding subquery.*

Query cũng dùng `WHERE id != that`.  
*The query also uses `WHERE id != that`.*

2. **[BẪY] "`<=>` trả cosine similarity đúng không?"**  
*2. **[TRAP] "Does `<=>` return cosine similarity?"***

Không.  
*No.*

`<=>` trả cosine distance.  
*`<=>` returns cosine distance.*

Cosine distance bằng `1 − similarity`.  
*Cosine distance equals `1 − similarity`.*

Similarity bằng `1 - (embedding <=> q)`.  
*Similarity equals `1 - (embedding <=> q)`.*

3. **"Threshold để làm gì?"**  
*3. **"What does a threshold do?"***

Threshold lọc các kết quả đủ gần.  
*A threshold filters sufficiently close results.*

`LIMIT` chỉ giới hạn số lượng.  
*`LIMIT` only caps the quantity.*

Ngưỡng phụ thuộc metric.  
*The threshold depends on the metric.*

Ngưỡng cũng phụ thuộc dataset.  
*The threshold also depends on the dataset.*

Bạn phải hiệu chỉnh ngưỡng trên golden set.  
*You must calibrate the threshold on a golden set.*

Cosine có miền `[0,2]`.  
*Cosine has a `[0,2]` range.*

Cosine thường dễ đặt ngưỡng hơn L2.  
*Cosine usually supports easier threshold selection than L2.*

4. **[BẪY] "`WHERE embedding <=> q < 0.3` có nhanh không?"**  
*4. **[TRAP] "Is `WHERE embedding <=> q < 0.3` fast?"***

Threshold thuần dễ tạo seq scan.  
*A bare threshold can cause a sequential scan.*

Query này có thể tính distance cho mọi dòng.  
*This query may calculate distance for every row.*

Query phải kèm `ORDER BY embedding <=> q LIMIT k`.  
*The query must include `ORDER BY embedding <=> q LIMIT k`.*

Cấu trúc này cho phép dùng ANN index.  
*This structure enables ANN index usage.*

5. **[BẪY] "Kết quả đầu là chính review 1?"**  
*5. **[TRAP] "Is review 1 itself the first result?"***

Query đã quên `WHERE id != 1`.  
*The query omitted `WHERE id != 1`.*

Distance tới chính review 1 bằng 0.  
*The distance to review 1 itself equals zero.*

Review 1 luôn đứng đầu trong trường hợp này.  
*Review 1 always ranks first in this case.*

6. **[TRADE-OFF] "Threshold chặt hay lỏng?"**  
*6. **[TRADE-OFF] "Should the threshold be strict or loose?"***

Threshold chặt tạo precision cao.  
*A strict threshold produces high precision.*

Threshold chặt bỏ sót nhiều kết quả.  
*A strict threshold misses many results.*

Recall sẽ thấp hơn.  
*Recall will be lower.*

Query cũng dễ trả 0 kết quả.  
*The query can also return zero results.*

Threshold lỏng tạo recall cao.  
*A loose threshold produces high recall.*

Threshold lỏng có thể đưa rác vào kết quả.  
*A loose threshold can introduce noise into the results.*

Nhu cầu sản phẩm quyết định lựa chọn.  
*Product needs determine the choice.*

Golden set hỗ trợ calibrate ngưỡng.  
*A golden set supports threshold calibration.*

7. **[SCALE] "Sản phẩm tương tự không được hiện rác?"**  
*7. **[SCALE] "How do similar products avoid irrelevant results?"***

Threshold kiểm soát chất lượng.  
*The threshold controls quality.*

`LIMIT` kiểm soát số lượng.  
*`LIMIT` controls quantity.*

Filter nghiệp vụ giới hạn tập ứng viên.  
*Business filters restrict the candidate set.*

`iterative_scan` giảm nguy cơ overfiltering.  
*`iterative_scan` reduces the risk of overfiltering.*

Kết quả có thể bằng 0 sau threshold.  
*The result count may be zero after thresholding.*

Ứng dụng nên ẩn khối sản phẩm tương tự.  
*The application should hide the similar-products section.*

Ứng dụng không nên hiển thị rác.  
*The application should not display irrelevant items.*

8. **"ANN + threshold có đảm bảo đúng không?"**  
*8. **"Does ANN plus a threshold guarantee correctness?"***

Cơ chế này không bảo đảm tuyệt đối.  
*This mechanism does not provide an absolute guarantee.*

ANN là approximate.  
*ANN is approximate.*

ANN có thể bỏ sót kết quả thật.  
*ANN can miss a true result.*

Two-stage phù hợp với yêu cầu chắc chắn hơn.  
*Two-stage retrieval suits stronger correctness requirements.*

ANN lấy một tập top-N rộng.  
*ANN retrieves a broad top-N set.*

Giai đoạn exact tính lại distance.  
*The exact stage recalculates distance.*

Giai đoạn exact cũng áp dụng threshold.  
*The exact stage also applies the threshold.*

### 5.6. One-liner đắt giá
*5.6. Valuable one-liners*

- `ORDER BY distance LIMIT k` là khung xương của mọi vector query.  
  *`ORDER BY distance LIMIT k` is the backbone of every vector query.*

- Query dùng thứ tự tăng dần.  
  *The query uses ascending order.*

- Distance nhỏ biểu thị kết quả gần hơn.  
  *Smaller distance indicates a closer result.*

- `<=>` trả cosine distance.  
  *`<=>` returns cosine distance.*

- `<=>` không trả similarity.  
  *`<=>` does not return similarity.*

- Similarity bằng `1 - (embedding <=> q)`.  
  *Similarity equals `1 - (embedding <=> q)`.*

- `LIMIT` giới hạn số lượng.  
  *`LIMIT` caps the quantity.*

- Threshold bảo đảm chất lượng.  
  *A threshold guarantees quality.*

- Các kết quả gần nhất chưa chắc đủ tốt.  
  *The nearest results may not be good enough.*

- `WHERE distance < x` thuần bỏ qua ANN index.  
  *A bare `WHERE distance < x` bypasses the ANN index.*

- Query luôn cần `ORDER BY distance LIMIT k`.  
  *The query always needs `ORDER BY distance LIMIT k`.*

- Threshold là một đường biên quyết định.  
  *A threshold is a decision boundary.*

- Bạn calibrate nó trên các cặp giống có nhãn.  
  *You calibrate it on labeled similar pairs.*

- Bạn cũng dùng các cặp khác có nhãn.  
  *You also use labeled dissimilar pairs.*

- Bạn không nên đoán ngưỡng.  
  *You should not guess the threshold.*

- Query có thể tìm item giống một row có sẵn.  
  *A query can find items similar to an existing row.*

- Query phải loại row nguồn.  
  *The query must exclude the source row.*

- Distance của row nguồn tới chính nó bằng 0.  
  *The source row has zero distance from itself.*

---

### 📌 Ghi chú cuối
*📌 Final notes*

- `<=>` là cosine distance.  
  *`<=>` is cosine distance.*

- `<=>` không phải similarity.  
  *`<=>` is not similarity.*

- Threshold phụ thuộc metric.  
  *A threshold depends on the metric.*

- Threshold cũng phụ thuộc dataset.  
  *A threshold also depends on the dataset.*

- Threshold thuần không dùng ANN index.  
  *A bare threshold does not use the ANN index.*

- Query luôn cần `ORDER BY + LIMIT`.  
  *The query always needs `ORDER BY + LIMIT`.*

- Bạn phải nhớ `WHERE id != self`.  
  *You must remember `WHERE id != self`.*

- Trong lúc ôn, bạn nên xem pgvector README.  
  *During review, you should read the pgvector README.*

- README trình bày các toán tử.  
  *The README presents the operators.*

- README cũng trình bày các ops class.  
  *The README also presents the ops classes.*

- README cung cấp các ví dụ query.  
  *The README provides query examples.*

- Tài liệu pgvector 0.8+ giải thích iterative scan.  
  *The pgvector 0.8+ documentation explains iterative scan.*

- Tài liệu cũng giải thích tương tác với filter.  
  *The documentation also explains interaction with filters.*

- Bảng thực hành là `reviews(embedding vector(384))`.  
  *The practice table is `reviews(embedding vector(384))`.*

- Bảng này đã được nạp dữ liệu.  
  *This table already contains data.*

- Giáo trình bulk insert mô tả bước nạp này.  
  *The bulk insert lesson describes this loading step.*

- Bạn hãy chạy cả ba toán tử.  
  *Run all three operators.*

- Query cần tìm các review giống review `id=1`.  
  *The query should find reviews similar to review `id=1`.*

- Bạn hãy thêm cột `1-(<=>)`.  
  *Add the `1-(<=>)` column.*

- Cột này hiển thị similarity.  
  *This column displays similarity.*

- Bạn hãy thử threshold chặt.  
  *Try a strict threshold.*

- Bạn hãy thử threshold lỏng.  
  *Try a loose threshold.*

- Bạn hãy chạy `EXPLAIN ANALYZE`.  
  *Run `EXPLAIN ANALYZE`.*

- Kết quả cho biết lúc `Index Scan` biến mất.  
  *The result shows when `Index Scan` disappears.*

- Series này có đủ 8 mảnh.  
  *This series contains all eight pieces.*

- Chuỗi là embed → index → cài → nạp → query.  
  *The sequence is embed → index → install → load → query.*

- Chuỗi tiếp tục với store/query khái niệm.  
  *The sequence continues with conceptual store/query.*

- Phần này mang số `#3`.  
  *This part has the number `#3`.*

- Chuỗi kết thúc với FTS → chọn nền tảng.  
  *The sequence ends with FTS → platform selection.*

- Bạn đã viết được semantic search query đúng.  
  *You can now write correct semantic search queries.*

- Bạn cũng kiểm soát được chất lượng.  
  *You can also control quality.*

- Bạn nên học tiếp reranking sau retrieval.  
  *You should study reranking after retrieval next.*

- Cross-encoder có thể thực hiện reranking.  
  *A cross-encoder can perform reranking.*

- Bạn nên học tiếp query rewriting.  
  *You should study query rewriting next.*

- Bạn cũng nên học query expansion.  
  *You should also study query expansion.*

- Bạn nên học evaluation cho retrieval.  
  *You should study retrieval evaluation.*

- Evaluation cần một quy trình có hệ thống.  
  *Evaluation needs a systematic process.*

- Các metric gồm precision@k, recall@k và nDCG.  
  *The metrics include precision@k, recall@k, and nDCG.*

- Golden set hỗ trợ quá trình evaluation.  
  *A golden set supports the evaluation process.*
