# Indexing cho Vector Search: từ Binary Search tới IVFFlat & HNSW — Giáo trình siêu dễ hiểu, Basic → Staff

> **Bài giảng gốc:** bài đọc *"Creating Indexes for Vector Search Optimization"* (tác giả Lavanya T S) — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy ba thứ: định nghĩa database index, index tiết kiệm thời gian ra sao (linear search so với binary search), và hai loại index của pgvector (IVFFlat, HNSW) kèm tham số. Tôi giảng bám sát rồi đào sâu; chỗ đi xa hơn bài gốc được đánh dấu 🧩 **[Ngoài bài gốc]**.
>
> **Vị trí trong series:** đây là mảnh **"làm cho search NHANH"**. Mạch của cả series: embed (tạo vector) → **index (bài này)** → store & query (pgvector) → keyword/full-text search → hybrid. Bài này zoom vào đúng một câu hỏi: *khi có hàng triệu vector, làm sao tìm nhanh mà không phải quét hết?*
>
> **Có trùng với giáo trình pgvector trước không?** Có một phần (HNSW/IVFFlat). Nhưng ở đây tôi tiếp cận từ **lý thuyết indexing** — bắt đầu từ binary search, đi tới chỗ nó *gãy*, rồi mới tới ANN. Mắt xích đó là thứ bài gốc bỏ qua và cũng là thứ hay bị hỏi nhất trong phỏng vấn. Giáo trình này **tự đứng độc lập được**: mọi thuật ngữ vẫn được giải thích lại từ đầu, bạn không cần đọc bài trước.
>
> **⚠️ Ba đính chính trong bài gốc — đọc kỹ để đi phỏng vấn không nói sai:**
> 1. **HNSW = Hierarchical Navigable Small *World*** (thế giới nhỏ), chứ không phải "Small Words". Bài gốc viết sai chính tả thuật ngữ. Chữ *Small World* là một khái niệm có thật trong lý thuyết mạng, tôi sẽ giải thích ở mục 3.1.
> 2. Bài gốc nói cạnh của HNSW "là khoảng cách Euclidean" — **không đúng**. HNSW dùng **bất kỳ thước đo khoảng cách nào bạn cấu hình**; chính ví dụ trong bài gốc dùng `vector_cosine_ops`, tức là cosine chứ không phải Euclidean.
> 3. Hình HNSW trong bài gốc đánh nhãn tầng bị lỗi ("Layer 3, 2, 3, 0"). Đúng phải là **Layer 3 → 2 → 1 → 0**, từ trên xuống dưới.
>
> **Cập nhật tới tháng 7/2026:** pgvector đang ở nhánh **0.8.x**, bản mới nhất là **v0.8.5**. HNSW là index mặc định được khuyên dùng. Lưu ý vận hành: bản **0.8.2** vá lỗ hổng CVE-2026-3172 (tràn bộ đệm khi build index HNSW song song) — nếu đang chạy 0.8.0/0.8.1 thì nên nâng cấp.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### Bài này dạy gì — nói bằng một câu đơn giản nhất

**Bài này dạy bạn vì sao việc tìm kiếm trong một đống dữ liệu khổng lồ lại có thể nhanh, và vì sao mẹo làm-cho-nhanh quen thuộc mấy chục năm nay bỗng nhiên *vô dụng* khi ta chuyển sang tìm kiếm bằng vector — nên người ta phải nghĩ ra hai cách hoàn toàn mới.**

Nếu ngay lúc này bạn chưa biết "index" là gì, "binary search" là gì, hay "vector" là gì — hoàn toàn bình thường. Phần 1 dựng lại từ con số 0 và không giả định bạn biết trước bất cứ điều gì.

### Vấn đề nó giải quyết

Bạn có 100 triệu vector và cần tìm cái giống nhất với một câu truy vấn. Cách ngây thơ là so sánh với từng cái một — 100 triệu phép so cho **mỗi** lần bấm nút tìm kiếm. Bất khả thi nếu bạn muốn trả lời trong một phần mười giây.

Index giải quyết chuyện đó. Nhưng câu chuyện có một khúc quanh thú vị: **index truyền thống (thứ đã phục vụ ngành database suốt 50 năm) không dùng được cho vector.** Hiểu *chính xác vì sao nó không dùng được* là chỗ phân biệt người thực sự hiểu với người học thuộc lòng — và đó là trọng tâm của giáo trình này.

### Học xong bạn sẽ làm được

- Định nghĩa được database index, và **tính ra con số cụ thể** chi phí của linear search `(n+1)/2` so với binary search `O(log n)`.
- Giải thích được **vì sao B-tree và binary search không dùng được cho vector nhiều chiều** — mắt xích cốt lõi mà bài gốc bỏ qua.
- Mô tả được cơ chế bên trong của **IVFFlat** (centroid / list / probe) và **HNSW** (đồ thị nhiều tầng), kèm độ phức tạp Big-O.
- Viết được câu lệnh tạo index và tinh chỉnh `lists`/`probes`, `m`/`ef_construction`/`ef_search`.
- Chọn đúng index theo quy mô, phân tích trade-off giữa độ chính xác – tốc độ – bộ nhớ, và trả lời được câu hỏi phỏng vấn dạng system design.

### Mạch kiến thức: basic → staff

- 🟢 **Basic** — Index là gì (analogy mục lục sách) → linear search tốn `(n+1)/2` → sắp xếp rồi binary search → **ví dụ tìm "Jack" chạy tay từng bước** → B-tree.
- 🟡 **Intermediate** — **Vì sao index kinh điển gãy với vector** (mắt xích quan trọng nhất) → ANN là gì → IVFFlat với centroid/list/probe + code + exact vs approximate + ba lỗi thường gặp.
- 🔴 **Advanced** — Bên trong HNSW, ví dụ "tìm số 12" chạy tay, bảng Big-O đối chiếu ba phương pháp, lời nguyền số chiều, DiskANN, tự viết lại thuật toán, các trường hợp biên.
- 🟣 **Staff** — Chọn index theo quy mô thật, chi phí build, giám sát recall, giới hạn bộ nhớ, khi nào **không** nên đánh index, tìm kiếm có lọc, và câu hỏi system design mẫu.
- 🎯 **Cheatsheet** — Từ khoá, ý cốt lõi, code cần thuộc, câu hỏi phỏng vấn và các câu chốt "đắt giá".

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người chưa biết gì. Tôi đi rất chậm và giải thích cả những thứ mà dân trong nghề coi là hiển nhiên. Nếu bạn đã quen với database, đừng bỏ mục 1.4 — nó là bản lề của toàn bộ giáo trình.

### 1.1. Bắt đầu từ nỗi đau: tìm một cái tên trong một danh sách

Trước khi nói tới giải pháp, ta phải cảm được vấn đề. Không thấy nỗi đau thì không nhớ nổi liều thuốc.

Vài từ cần giải thích trước đã:

- **Database** (cơ sở dữ liệu) — nơi lưu trữ dữ liệu có tổ chức của một ứng dụng, thay vì để rải rác trong các file lẻ.
- **Table** (bảng) — cách dữ liệu được tổ chức bên trong database: có các cột và các dòng, giống một sheet Excel.
- **Row** / **record** (dòng / bản ghi) — một dòng trong bảng, chứa thông tin về một đối tượng cụ thể (một người dùng, một sản phẩm).
- **Query** (truy vấn) — câu lệnh hỏi database: "cho tôi những dòng thoả điều kiện này".

Bây giờ hãy tưởng tượng bạn có một bảng chứa **20 cái tên**, và người dùng muốn tìm tên **"Jack"**. Cách ngây thơ nhất mà máy tính có thể làm là: đọc dòng 1, không phải Jack; đọc dòng 2, không phải; đọc dòng 3... cho tới khi gặp.

Nếu Jack nằm ở dòng 20 — dòng cuối cùng — máy phải đọc **cả 20 dòng**.

Với 20 dòng thì chẳng sao. Nhưng hãy thử với 1 triệu dòng. Và rồi nhân nó với hàng nghìn người dùng cùng bấm tìm kiếm mỗi giây. Đó là lúc hệ thống của bạn sụp.

Vấn đề gói gọn trong một câu: **làm sao tìm mà không phải nhìn hết?**

### 1.2. Index là gì — định nghĩa và analogy

**Database index** (chỉ mục cơ sở dữ liệu) là **một cấu trúc dữ liệu phụ, được dựng thêm bên cạnh bảng, nhằm tăng tốc việc tìm kiếm**. Nó chứa một bản sao đã được tổ chức lại của một phần dữ liệu trong bảng, sắp xếp theo cách giúp định vị nhanh dòng cần tìm.

("**Cấu trúc dữ liệu**" chỉ đơn giản là cách sắp xếp dữ liệu trong bộ nhớ để làm một việc gì đó cho hiệu quả. Một danh sách là một cấu trúc dữ liệu; một cái cây cũng vậy.)

Bây giờ tới analogy — và tôi sẽ dựng nó thật đầy đủ chứ không nói lướt, vì đây là hình ảnh mà bạn sẽ dùng để giải thích cho cả sếp lẫn người phỏng vấn.

> **Hãy nghĩ tới phần mục lục tra cứu ở cuối một cuốn sách dày** (tiếng Anh gọi đúng là *index*, cùng một từ, không phải trùng hợp).
>
> Bạn cầm một cuốn sách 800 trang về database và muốn biết cuốn này nói gì về "transaction". Bạn có hai lựa chọn:
>
> **Cách 1:** mở trang 1, đọc, không thấy; sang trang 2, đọc... Bạn sẽ mất cả buổi chiều.
>
> **Cách 2:** lật ra mục lục tra cứu ở cuối sách, dò tới chữ T, thấy dòng *"transaction → trang 12, 45, 301"*, và nhảy thẳng tới đó. Mất mười giây.

Bây giờ là phần quan trọng — **chỉ ra chính xác chỗ analogy này khớp với kỹ thuật**, vì analogy chỉ có giá trị khi bạn biết nó khớp ở đâu:

| Trong cuốn sách | Trong database |
|---|---|
| Mục lục tra cứu ở cuối sách | Index |
| Đọc từ trang 1 tới khi thấy | Sequential scan / linear search |
| Nhảy thẳng tới trang 301 | Index scan |
| Mục lục **chiếm thêm mấy chục trang giấy** | Index **chiếm thêm dung lượng đĩa và RAM** |
| Sửa nội dung sách thì **phải in lại mục lục** | Mỗi lần thêm/sửa/xoá dòng, database **phải cập nhật index** → ghi chậm đi một chút |
| Mục lục cho sách 10 trang thì **vô nghĩa** | Index trên bảng nhỏ **không đáng** |

Hai dòng cuối là chỗ mà người mới hay bỏ qua, nên tôi nhấn mạnh: **index không miễn phí**. Ý tưởng cốt lõi của toàn bộ chuyện này là:

> **Trả một chút chi phí lưu trữ và một chút chi phí lúc ghi dữ liệu, để đổi lấy tốc độ tìm kiếm nhanh gấp hàng nghìn lần.**

Và như chính bài gốc nhấn mạnh trong phần tóm tắt: **hiệu quả của index chỉ thực sự lộ ra trên tập dữ liệu lớn.** Ghi nhớ câu này, ta sẽ dùng lại nó ở Phần 4 để trả lời một câu hỏi bẫy kinh điển.

### 1.3. Linear search tốn bao nhiêu — tính bằng con số

**Linear search** (tìm kiếm tuyến tính, còn gọi là *sequential scan* — quét tuần tự) là cách làm ở mục 1.1: so từng dòng một, từ đầu tới cuối, cho tới khi tìm thấy.

Ta tính chi phí trung bình của nó. Gọi `n` là số dòng.

- Nếu may mắn, thứ bạn tìm nằm ở dòng đầu → đọc **1** dòng.
- Nếu xui, nó nằm ở dòng cuối → đọc **n** dòng.
- Nếu nó **không tồn tại**, bạn buộc phải đọc hết **n** dòng mới dám kết luận.

Trung bình cho các trường hợp tìm thấy, số dòng phải đọc là:

```
(1 + n) / 2
```

(Vì các vị trí từ 1 đến n có khả năng như nhau, nên trung bình cộng của chúng là `(1+n)/2`.)

Hãy thay số vào cho thật cụ thể — đây là con số bạn nên thuộc lòng để nói trong phỏng vấn:

| Số dòng `n` | Trung bình phải đọc |
|---|---|
| 20 | ~10 dòng |
| 1.000 | ~500 dòng |
| **1.000.000** | **~500.000 dòng** |
| 100.000.000 | ~50.000.000 dòng |

Nửa triệu lần đọc cho **một** lần tìm kiếm. Bây giờ nhân với 1000 người dùng mỗi giây và bạn sẽ hiểu vì sao "không có index" là bất khả thi ở quy mô lớn.

Ký hiệu chuyên môn cho chi phí này là **`O(n)`** — tôi sẽ giải thích ký hiệu `O` ngay ở mục sau khi ta có cái để so sánh.

### 1.4. Sắp xếp + binary search — ví dụ tìm "Jack" chạy tay

Bây giờ ta làm một việc rất đơn giản nhưng thay đổi mọi thứ: **sắp xếp danh sách theo bảng chữ cái**.

Đây là 20 cái tên sau khi sắp xếp:

| Vị trí | Tên | | Vị trí | Tên |
|---|---|---|---|---|
| 1 | Alexander | | 11 | **Jack** |
| 2 | Amelia | | 12 | James |
| 3 | Ava | | 13 | Liam |
| 4 | Benjamin | | 14 | Lucas |
| 5 | Charlotte | | 15 | Mia |
| 6 | Daniel | | 16 | Michael |
| 7 | Emily | | 17 | Noah |
| 8 | Emma | | 18 | Oliver |
| 9 | Grace | | 19 | Sophia |
| 10 | Isabella | | 20 | William |

**Binary search** (tìm kiếm nhị phân) hoạt động theo một ý tưởng duy nhất: **luôn nhìn vào phần tử ở giữa, rồi vứt bỏ một nửa danh sách.**

Ta chạy tay từng bước để tìm "Jack". Tôi sẽ ghi rõ phạm vi còn lại sau mỗi bước.

**Bước 1.** Phạm vi: vị trí 1–20. Điểm giữa là vị trí 10 → **Isabella**.
So sánh: "Jack" đứng **sau** "Isabella" trong bảng chữ cái (chữ J sau chữ I).
→ Kết luận: Jack chắc chắn nằm ở nửa sau. **Vứt bỏ vị trí 1–10.** *Vừa loại 10 tên chỉ bằng một phép so sánh.*
Phạm vi còn lại: 11–20 (10 tên).

**Bước 2.** Điểm giữa của 11–20 là vị trí 15 → **Mia**.
So sánh: "Jack" đứng **trước** "Mia" (J trước M).
→ Vứt bỏ vị trí 15–20.
Phạm vi còn lại: 11–14 (4 tên).

**Bước 3.** Điểm giữa của 11–14 là vị trí 12 → **James**.
So sánh: "Jack" đứng **trước** "James" — hai tên cùng bắt đầu bằng "Ja", nhưng chữ thứ ba là "c" đứng trước "m".
→ Vứt bỏ vị trí 12–14.
Phạm vi còn lại: 11 (1 tên).

**Bước 4.** Điểm giữa của phạm vi chỉ còn một phần tử là vị trí 11 → **Jack**. ✅ Tìm thấy.

**Bốn bước.** So với 20 lần đọc của linear search.

Bây giờ tới điều thực sự đáng kinh ngạc. Vì mỗi bước cắt đôi phạm vi, số bước cần thiết là **log₂(n)** — logarit cơ số 2 của n, tức là "phải chia đôi bao nhiêu lần thì n về còn 1".

| Số phần tử `n` | Linear search (trung bình) | Binary search (số bước) |
|---|---|---|
| 20 | ~10 | ~5 |
| 1.000 | ~500 | ~10 |
| **1.000.000** | **~500.000** | **~20** |
| 1.000.000.000 | ~500.000.000 | ~30 |

Hãy nhìn dòng in đậm cho kỹ. **Một triệu tên, hai mươi bước.** (Vì 2²⁰ ≈ 1.048.576.) Và khi dữ liệu tăng gấp một nghìn lần nữa, lên một tỷ, số bước chỉ tăng từ 20 lên 30.

Ký hiệu chuyên môn: đây là **`O(log n)`**.

Nhân tiện giải thích luôn ký hiệu **Big-O** (đọc là "O lớn") vì ta sẽ dùng nó suốt phần còn lại: nó **không** cho biết chương trình chạy bao nhiêu giây; nó cho biết **thời gian chạy tăng lên thế nào khi dữ liệu to ra**. Và cái xu hướng đó mới là thứ quyết định hệ thống sống hay chết ở quy mô lớn.

- `O(n)` — dữ liệu gấp 1000 lần thì chậm gấp 1000 lần. Chết ở quy mô lớn.
- `O(log n)` — dữ liệu gấp 1000 lần thì chỉ chậm thêm khoảng 10 lần. Đây là con số vàng.

### 1.5. ⭐ Điều kiện tiên quyết của binary search — mắt xích quan trọng nhất cả bài

Hãy dừng lại và hỏi một câu mà rất ít người học đặt ra: **vì sao binary search làm được cái trò cắt đôi đó?**

Câu trả lời: vì ở **mỗi bước**, ta trả lời được câu hỏi *"mục tiêu nằm trước hay sau điểm giữa?"*

Và ta trả lời được câu hỏi đó chỉ vì các cái tên **nằm trên một trục có thứ tự** — bảng chữ cái. Với hai cái tên bất kỳ, luôn tồn tại một đáp án dứt khoát cho câu "cái nào đứng trước".

Thuật ngữ toán học cho tính chất này là **total order** (thứ tự toàn phần / thứ tự tuyến tính): mọi cặp phần tử đều so sánh được với nhau bằng "nhỏ hơn / lớn hơn", và các phần tử xếp thành **một đường thẳng duy nhất**.

Chính vì có đường thẳng đó nên khái niệm "điểm giữa" mới tồn tại. Và chính vì "điểm giữa" tồn tại nên mới cắt đôi được.

> **🔑 [GHI NHỚ — mắt xích quan trọng nhất của cả giáo trình]**
>
> Binary search hoạt động được **chỉ khi** dữ liệu sắp xếp được trên một trục duy nhất.
>
> Điều kiện này chính là chỗ sẽ **vỡ** khi ta chuyển sang vector nhiều chiều. Hãy giữ ý này trong đầu — mục 2.1 sẽ cho bạn thấy nó vỡ như thế nào và vì sao cả một ngành công nghiệp phải nghĩ ra cách hoàn toàn khác.

### 1.6. Index kinh điển trong database = B-tree

Binary search ở trên hoạt động trên một mảng đã sắp xếp nằm trong bộ nhớ. Database thực tế cần một phiên bản mạnh hơn, vì dữ liệu nằm trên đĩa và liên tục bị thêm/xoá. Phiên bản đó tên là **B-tree**.

**B-tree** (cây B — chữ B thường được hiểu là *balanced*, cân bằng) là **một cấu trúc cây giữ dữ liệu ở dạng đã sắp xếp, và tự cân bằng để mọi nhánh đều có độ sâu xấp xỉ nhau**.

Bạn có thể hình dung nó đúng như "binary search được đóng gói thành cấu trúc cây": mỗi lần đi xuống một tầng của cây là một lần loại bỏ phần lớn dữ liệu, y hệt như cắt đôi. Nhờ vậy nó cũng cho `O(log n)`.

B-tree là loại index **mặc định** trong mọi hệ quản trị database quan hệ (**RDBMS** — viết tắt của *Relational Database Management System*, tức hệ quản trị cơ sở dữ liệu quan hệ, ví dụ PostgreSQL, MySQL). Khi bạn gõ `CREATE INDEX` mà không nói gì thêm, bạn đang tạo một B-tree.

```sql
-- Index kinh điển: B-tree trên các cột SẮP THỨ TỰ ĐƯỢC

CREATE INDEX ON users (email);
-- Từ giờ câu "SELECT * FROM users WHERE email = 'a@b.com'" chạy O(log n)
-- thay vì quét cả bảng.

CREATE INDEX ON orders (created_at);
-- B-tree còn giỏi RANGE SCAN (quét theo khoảng): vì dữ liệu đã sắp thứ tự,
-- câu "đơn hàng trong 7 ngày qua" chỉ cần tìm điểm bắt đầu rồi đọc liên tiếp.
```

B-tree tuyệt vời cho dữ liệu **một chiều và sắp thứ tự được**: số, chuỗi ký tự, ngày tháng. Nó đã phục vụ ngành database suốt nửa thế kỷ.

Nhưng vector thì không phải loại dữ liệu đó. Và đó chính là lý do pgvector phải mang tới những loại index hoàn toàn khác — nội dung của Phần 2.

### 🧩 [Ngoài bài gốc] — Hai điều bài gốc không nói về index

**1. Index làm chậm việc ghi dữ liệu.** Bài gốc chỉ nói index tốn thêm chỗ. Nhưng cái giá thật sự khó chịu hơn: mỗi lần bạn `INSERT`, `UPDATE` hay `DELETE` một dòng, database phải **cập nhật tất cả index** liên quan tới dòng đó. Một bảng có 8 index thì mỗi lần ghi là 9 lần ghi thật sự (1 vào bảng + 8 vào index). Hiện tượng này gọi là **write amplification** (khuếch đại ghi).

Hệ quả thực tế: một hệ thống ghi nhiều mà đánh index bừa bãi sẽ *chậm đi*, không phải nhanh lên. "Đánh index cho mọi cột" là một trong những lời khuyên tệ nhất trong nghề.

**2. Index có thể bị "phình" (bloat).** Sau nhiều lần xoá và cập nhật, index tích tụ các phần không còn dùng nhưng vẫn chiếm chỗ. Trong Postgres, đây là lý do người ta phải chạy `REINDEX` định kỳ. Với index vector, chuyện này còn khó chịu hơn nữa — ta sẽ gặp lại ở mục 4.2.

### ✅ Self-check Phần 1

Trả lời trong đầu trước, rồi mở gợi ý ra so.

**Câu 1.** Linear search trên 1 triệu dòng trung bình phải đọc bao nhiêu dòng? Binary search cần bao nhiêu bước?
> *Gợi ý đáp án:* Linear ≈ `(1.000.000+1)/2` ≈ **500.000 dòng**. Binary search ≈ `log₂(1.000.000)` ≈ **20 bước**. Khác nhau khoảng 25.000 lần — và khoảng cách này càng giãn ra khi dữ liệu càng lớn.

**Câu 2.** Vì sao binary search *bắt buộc* dữ liệu phải được sắp xếp?
> *Gợi ý đáp án:* Vì nó cần trả lời câu "mục tiêu nằm trước hay sau điểm giữa" ở mỗi bước. Câu đó chỉ trả lời được khi tồn tại **thứ tự toàn phần** trên một trục duy nhất. Không có thứ tự thì không có khái niệm "điểm giữa", nên không cắt đôi được, và việc một phần tử nào đó lớn hơn phần tử giữa cũng không cho ta biết gì về vị trí của nó.

**Câu 3.** B-tree hợp với kiểu dữ liệu nào? Cho một ví dụ cột nên đánh B-tree và một ví dụ không nên.
> *Gợi ý đáp án:* Hợp với dữ liệu một chiều, sắp thứ tự được: số, chuỗi, ngày tháng. Nên đánh: `email`, `created_at`, `user_id`. Không nên: cột `embedding` kiểu vector (lý do ở Phần 2), hoặc một cột mà mọi dòng đều có cùng giá trị (index vô ích vì không loại bỏ được gì).

**Câu 4.** Vì sao "đánh index cho mọi cột" là lời khuyên tệ?
> *Gợi ý đáp án:* Vì mỗi index tốn thêm dung lượng và làm **mọi thao tác ghi chậm đi** (write amplification). Index chỉ đáng khi cột đó thực sự được dùng để tìm kiếm, và khi bảng đủ lớn để lợi ích vượt chi phí.

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Phần này giả định bạn đã nắm: index là gì, linear search tốn `O(n)`, binary search cho `O(log n)`, và điều kiện tiên quyết của nó là dữ liệu phải sắp thứ tự được. Nếu còn lấn cấn, hãy quay lại mục 1.5 — mọi thứ dưới đây mọc lên từ đó.

### 2.1. ⭐ [MẮT XÍCH CỐT LÕI] Vì sao B-tree và binary search GÃY với vector

Bài gốc nhảy thẳng từ binary search sang IVFFlat/HNSW mà không giải thích *vì sao cần một loại index mới*. Đây chính là chỗ phân biệt người hiểu bản chất với người học vẹt, và là câu hỏi mà người phỏng vấn giỏi rất hay dùng để dò xem bạn có thực sự hiểu hay không.

Trước hết, nhắc lại nhanh **vector** là gì để giáo trình này đứng độc lập được:

- **Vector** (trong ngữ cảnh này còn gọi là **embedding**) là **một dãy số biểu diễn ý nghĩa của một vật** — một đoạn chữ, một tấm ảnh, một đoạn âm thanh. Ví dụ câu "áo khoác mùa đông" có thể biến thành `[0.12, -0.98, 0.33, ...]` gồm 384 con số.
- Số lượng con số trong dãy gọi là **dimension** (số chiều). Model `all-MiniLM-L6-v2` cho 384 chiều; `text-embedding-3-small` của OpenAI cho 1536 chiều.
- Quy tắc vàng: **hai vật giống nhau về ý nghĩa thì hai vector của chúng nằm gần nhau**. Nên bài toán "tìm thứ giống nhất" biến thành bài toán "tìm điểm gần nhất trong không gian".
- Bài toán đó có tên: **Nearest Neighbor Search** (tìm hàng xóm gần nhất), viết tắt **NN search**. Khi muốn lấy đúng `k` cái gần nhất thì gọi là **k-NN**.

Bây giờ, ba lý do vì sao công cụ cũ không dùng được.

#### Vấn đề 1 — Vector không có "thứ tự tuyến tính"

Đây là hệ quả trực tiếp của mắt xích ở mục 1.5.

Bạn sắp xếp tên theo bảng chữ cái được, vì tên nằm trên **một** trục. Nhưng vector `[0.2, -0.7, 0.4, ...]` gồm 384 con số thì sắp xếp theo... **chiều nào**?

Giả sử bạn quyết định sắp theo chiều thứ nhất. Hãy xem chuyện gì xảy ra với một ví dụ 2 chiều cho dễ nhìn:

| Vật | Vector |
|---|---|
| A | (0.1, 0.9) |
| B | (0.2, 0.9) |
| C | (0.15, 0.1) |

Sắp theo chiều thứ nhất (x): A (0.1) → C (0.15) → B (0.2). Theo thứ tự này, **C nằm giữa A và B**.

Nhưng khoảng cách thật thì sao? A và B cách nhau 0.1 (rất gần — chúng gần như trùng nhau). Còn C thì cách cả hai khoảng 0.8 (rất xa). **Cái nằm "ở giữa" theo thứ tự sắp xếp lại là cái xa nhất trong thực tế.**

Và đó mới là 2 chiều. Với 384 chiều, tình hình còn tệ hơn nhiều bậc.

Kết luận: **không tồn tại một trục duy nhất nào để sắp mọi vector sao cho "gần trên trục" đồng nghĩa với "gần trong thực tế".** Binary search mất sạch điều kiện tiên quyết. Cắt đôi danh sách chẳng cho bạn thông tin gì về việc hàng xóm thật nằm ở nửa nào.

#### Vấn đề 2 — Câu hỏi đặt ra hoàn toàn khác

Ngay cả khi bỏ qua vấn đề 1, B-tree vẫn không phải công cụ đúng, vì nó được thiết kế để trả lời một loại câu hỏi khác hẳn.

| B-tree trả lời được | Vector search cần trả lời |
|---|---|
| "Có dòng nào **bằng đúng** giá trị X không?" | |
| "Những dòng nào nằm **trong khoảng** [a, b]?" | |
| | **"Dòng nào GẦN NHẤT với X về khoảng cách?"** |

Hai câu đầu là bài toán *khớp* và *khoảng*. Câu thứ ba là bài toán *nearest neighbor*. Chúng khác nhau về bản chất: với hai câu đầu, bạn biết trước tiêu chí và chỉ cần kiểm tra từng ứng viên có thoả hay không. Với câu thứ ba, bạn **không thể biết một ứng viên có phải đáp án hay không cho tới khi đã xem hết những cái khác** — vì "gần nhất" là một tính chất tương đối, phụ thuộc vào toàn bộ tập dữ liệu.

Đó là một bài toán khó hơn hẳn về mặt bản chất.

#### Vấn đề 3 — Lời nguyền số chiều

Câu hỏi rất hợp lý mà một người thông minh sẽ đặt ra ở đây: "Được rồi, không sắp trên một trục được. Nhưng trong đồ hoạ máy tính người ta có **kd-tree** (cây chia không gian thành các nửa theo lần lượt từng chiều) để tìm hàng xóm gần nhất trong không gian 2D/3D mà. Sao không dùng luôn?"

Người ta đã thử. Và nó thất bại, vì một hiện tượng tên là **curse of dimensionality** (lời nguyền số chiều).

Nội dung của lời nguyền, nói ngắn: **khi số chiều lớn (hàng trăm tới hàng nghìn), mọi điểm trở nên gần như cách đều nhau.** Khoảng cách tới hàng xóm gần nhất và khoảng cách tới điểm xa nhất xích lại gần bằng nhau, nên khái niệm "gần" mất dần ý nghĩa phân biệt.

Hệ quả kỹ thuật: kd-tree phải duyệt gần hết các nhánh mới dám khẳng định đã tìm đúng — tức là nó **sập trở về `O(n)`**, không nhanh hơn brute force là bao. Ta sẽ đào sâu hiện tượng này ở mục 3.3.

#### Kết luận — và bước ngoặt của cả câu chuyện

Ba vấn đề trên dồn ngành này vào một quyết định lớn: **từ bỏ việc tìm chính xác tuyệt đối.**

Đây là bước ngoặt đáng nhớ nhất trong toàn bộ chủ đề. Thay vì cố tìm hàng xóm gần nhất *thật*, ta chấp nhận tìm hàng xóm gần nhất *gần đúng* — nhanh hơn hàng nghìn lần và sai sót đủ nhỏ để không ai để ý.

Cách tiếp cận đó có tên: **ANN** (viết tắt của *Approximate Nearest Neighbor*, hàng xóm gần nhất **xấp xỉ**).

Và **IVFFlat** cùng **HNSW** — hai nhân vật chính của bài gốc — chính là hai cách hiện thực hoá ý tưởng ANN. Chúng đặt cược khác nhau, và cái giá phải trả cũng khác nhau. Phần còn lại của giáo trình là câu chuyện về hai cách đặt cược đó.

### 2.2. ANN và recall — hai từ phải hiểu trước khi đi tiếp

- **ANN** (*Approximate Nearest Neighbor*) — nhóm thuật toán chấp nhận **có thể bỏ sót** vài hàng xóm thật để đổi lấy tốc độ. Ngược với ANN là **exact search** (tìm chính xác tuyệt đối).
- **Recall** (độ bao phủ) — thước đo chất lượng của ANN. Định nghĩa: trong `k` hàng xóm thật sự gần nhất, thuật toán tìm được bao nhiêu phần trăm.
  > Ví dụ cụ thể: bạn hỏi 10 kết quả gần nhất. So với đáp án chính xác, thuật toán tìm đúng 9 cái, còn 1 cái nó bỏ sót và thay bằng một kết quả xa hơn. → **recall@10 = 90%**.

Từ đây trở đi, hãy ghi vào đầu câu này, vì nó là linh hồn của cả bài (và chính bài gốc cũng chốt đúng ý này trong phần tóm tắt):

> **Index ANN không làm search nhanh lên một cách miễn phí. Nó BÁN tốc độ cho bạn, và cái giá là độ chính xác. Chọn index và tinh chỉnh tham số chính là việc cân bằng ba thứ: độ chính xác ↔ tốc độ ↔ tài nguyên.**

### 2.3. IVFFlat — "chia cụm rồi chỉ tìm trong vài cụm"

**IVFFlat** = *Inverted File with Flat compression* (tệp đảo, vector lưu nguyên bản không nén).

#### Ý tưởng, kể bằng analogy

Bạn đang đứng ở một điểm trong thành phố và muốn tìm quán phở gần nhất. Cách ngu ngốc là đo khoảng cách tới từng quán phở trong toàn thành phố. Cách khôn ngoan:

1. Chia thành phố thành các **quận**, mỗi quận có một điểm trung tâm.
2. So vị trí của bạn với **trung tâm của các quận** — chỉ vài chục phép so, rất rẻ.
3. Chọn ra quận có trung tâm gần bạn nhất.
4. Chỉ đo khoảng cách tới các quán phở **bên trong quận đó**. Bỏ qua toàn bộ phần còn lại của thành phố.

Nếu thành phố có 1 triệu quán chia đều vào 1000 quận, bạn vừa giảm từ 1 triệu phép so xuống còn khoảng `1000 (so với tâm quận) + 1000 (quét trong quận)` = **2000 phép so**. Nhanh hơn 500 lần.

IVFFlat làm đúng như vậy trên không gian vector. Bây giờ là phần **chỉ rõ chỗ khớp** giữa analogy và thuật ngữ kỹ thuật:

| Trong analogy | Thuật ngữ kỹ thuật |
|---|---|
| Quận | **list** (cụm) — số lượng do tham số `lists` quyết định |
| Trung tâm quận | **centroid** (tâm cụm) — một vector đại diện cho cả cụm |
| Việc vẽ ranh giới quận | thuật toán **k-means clustering** |
| Số quận bạn chịu khó ghé xem | tham số **`probes`** |

**k-means clustering** (phân cụm k-means) là một thuật toán tự động nhóm các điểm gần nhau thành `k` cụm. Cách nó chạy rất trực quan: chọn đại `k` điểm làm tâm, gán mỗi điểm dữ liệu vào cụm có tâm gần nó nhất, rồi tính lại tâm bằng cách lấy trung bình các điểm trong cụm; lặp lại tới khi ổn định.

#### Hai tham số bắt buộc phải hiểu

**`lists`** — số cụm sẽ chia. Đây là tham số **lúc build index**.
> Công thức gợi ý chính thức: `số_dòng / 1000` nếu bảng có tối đa 1 triệu dòng; `sqrt(số_dòng)` nếu trên 1 triệu dòng.
> Ví dụ: 1 triệu vector → `lists = 1000`, mỗi cụm chứa trung bình 1000 phần tử.

**`probes`** — số cụm sẽ được ghé xem lúc tìm kiếm. Đây là tham số **lúc query**, mặc định là **1**.
> Cao hơn → xét thêm các cụm lân cận → **recall tốt hơn** nhưng **chậm hơn**. Gợi ý khởi đầu: `sqrt(lists)`.

Hai tham số này nằm ở hai thời điểm khác nhau, và đó là điều cần nhớ: `lists` bạn chỉ chọn được một lần lúc build (muốn đổi phải build lại); `probes` bạn xoay được bất cứ lúc nào lúc chạy.

#### Code

```sql
-- Tạo IVFFlat index.
-- ⚠️ LƯU Ý SỐNG CÒN: phải tạo SAU KHI bảng đã có dữ liệu.
--    Lý do: k-means cần dữ liệu thật để tìm tâm cụm. Xem mục 2.5, Lỗi 2.
CREATE INDEX ON sentence USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
--          ^^^^^^^^^^^^^^^^^^  "ops class": phải khớp với toán tử bạn dùng lúc query.
--                              vector_cosine_ops đi với toán tử <=> (cosine distance).

-- Lúc tìm kiếm: đặt số cụm sẽ dò.
SET ivfflat.probes = 32;      -- probes cao = recall cao, tốc độ giảm

SELECT * FROM sentence
ORDER BY embedding <=> '[...]'   -- <=> là toán tử tính cosine distance
LIMIT 10;                        -- ORDER BY ... LIMIT k = "cho tôi k cái giống nhất"
```

Một thuật ngữ mới trong đoạn trên: **ops class** (viết đầy đủ là *operator class*, lớp toán tử) là cách bạn nói cho Postgres biết index này được xây theo thước đo khoảng cách nào. Ba lựa chọn tương ứng ba toán tử:

| Ops class | Toán tử | Thước đo |
|---|---|---|
| `vector_cosine_ops` | `<=>` | cosine distance — phổ biến nhất với văn bản |
| `vector_l2_ops` | `<->` | Euclidean (L2) — khoảng cách đường chim bay |
| `vector_ip_ops` | `<#>` | negative inner product — nhanh nhất khi vector đã chuẩn hoá |

Chúng **phải khớp nhau**, nếu không index sẽ bị bỏ qua trong im lặng (mục 2.5, Lỗi 3).

#### Điểm yếu chí mạng của IVFFlat

**1. Hàng xóm nằm sát ranh giới cụm dễ bị bỏ sót.** Đây là điểm yếu mang tính cấu trúc, không sửa được bằng tham số.

> Quay lại analogy quán phở: bạn ở sát mép Quận 1, và có một quán phở tuyệt vời chỉ cách bạn 50 mét — nhưng nó nằm bên kia đường, thuộc Quận 3. Nếu bạn chỉ ghé Quận 1 (`probes = 1`), quán đó **hoàn toàn vô hình** với bạn, dù nó là quán gần nhất trong thực tế.

Cách giảm nhẹ: tăng `probes` để ghé thêm các quận lân cận. Nhưng lưu ý, đó chỉ là *giảm nhẹ* chứ không phải *xoá bỏ* — luôn tồn tại một ranh giới nào đó ở xa hơn.

**2. IVFFlat mang tính tĩnh.** Sau khi build xong, nếu bạn thêm hàng triệu dòng mới, chúng vẫn bị gán vào các centroid **cũ**. Phân hoạch dần lệch đi so với hình dạng thật của dữ liệu, và recall tụt xuống **một cách âm thầm, không có lỗi nào báo**. Cách chữa duy nhất là `REINDEX` (dựng lại index từ đầu) định kỳ. Với hệ thống có dữ liệu thay đổi liên tục, đây là một gánh nặng vận hành thật sự.

### 2.4. Exact vs Approximate — điều mà người mới hiểu nhầm nhiều nhất

Hãy đặt hai thứ cạnh nhau cho thật rõ:

| | Không có index | Có index ANN |
|---|---|---|
| Cách làm | Quét hết, tính khoảng cách từng dòng (**brute force**) | Chỉ xét một phần dữ liệu theo cấu trúc index |
| Recall | **100%** — luôn đúng tuyệt đối | **dưới 100%** — có thể sót vài hàng xóm thật |
| Tốc độ | `O(n · d)` — chậm khi dữ liệu lớn | nhanh hơn hàng trăm tới hàng nghìn lần |
| Bộ nhớ thêm | không | có (đáng kể với HNSW) |

(`n` là số vector, `d` là số chiều.)

**Điểm mấu chốt cần khắc cốt:** khi kết quả sau khi đánh index "khác" so với trước, **đó không phải bug**. Đó là *by design* (đúng theo thiết kế) — bạn đã đồng ý với cuộc đánh đổi này ngay từ lúc gõ `CREATE INDEX`.

Kịch bản dở khóc dở cười xảy ra ở khắp nơi: người ta test trên máy dev với 500 dòng (chưa có index → exact → đúng 100%), rồi lên production 5 triệu dòng có index HNSW, và hoảng lên vì "kết quả sai". Không có gì sai cả. Chỉ là không ai nói cho họ biết họ đã mua cái gì bằng cái gì.

### 2.5. 🧩 [Ngoài bài gốc] — Ba lỗi kinh điển và cách tránh

#### Lỗi 1 — Đánh index quá sớm trên dữ liệu nhỏ

Với vài nghìn tới vài chục nghìn vector, brute force exact **đã đủ nhanh** (thường dưới 10 mili-giây) và cho recall 100%. Đánh index ANN lúc này chỉ tổ làm giảm độ chính xác mà lợi ích tốc độ không đáng kể.

Đây chính là ý mà bài gốc chốt trong phần tóm tắt: *hiệu quả của index chỉ thấy rõ trên tập dữ liệu lớn.* Và nó dẫn tới một câu trả lời phỏng vấn rất ghi điểm mà ta sẽ dùng lại ở Phần 4: **"không phải mọi thứ đều cần index."**

#### Lỗi 2 — Tạo IVFFlat khi bảng còn rỗng

IVFFlat **bắt buộc phải có dữ liệu sẵn** lúc build, vì nó cần chạy k-means để tìm centroid. Build index trên bảng rỗng rồi mới đổ dữ liệu vào → các centroid được tính trên hư không → phân hoạch vô nghĩa → recall thảm hại.

Và cũng như mọi lỗi khác trong danh sách này: **không có thông báo lỗi nào**. Câu lệnh chạy thành công, index tồn tại, query vẫn chạy. Chỉ có chất lượng là rác.

Đây là điểm khác biệt lớn với HNSW — HNSW xây đồ thị bằng cách chèn dần từng node, nên nó tạo được cả khi bảng rỗng và lớn lên tự nhiên cùng dữ liệu.

#### Lỗi 3 — Ops class không khớp toán tử query

Bạn tạo index bằng `vector_cosine_ops` (dành cho `<=>`) nhưng lại query bằng `<->` (L2). Chuyện gì xảy ra?

**Postgres im lặng bỏ qua index của bạn** và rơi về sequential scan. Không lỗi, không cảnh báo. Chỉ là một query đáng lẽ chạy 5 mili-giây giờ chạy 5 giây — và nó thường chỉ lộ ra vào đúng lúc đông người dùng nhất.

Cách kiểm tra: dùng **`EXPLAIN ANALYZE`**, lệnh cho biết Postgres *thực sự* thực thi query bằng cách nào.

```sql
EXPLAIN ANALYZE
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

Trong kết quả trả về, hãy tìm:
- Thấy **`Index Scan using ..._hnsw...`** hoặc `..._ivfflat...` → index đang được dùng. Tốt.
- Thấy **`Seq Scan`** → index đang bị bỏ qua. Phải sửa.

Thói quen chạy `EXPLAIN ANALYZE` sau mỗi lần đánh index là thứ phân biệt người cẩn thận với người đoán mò. Nó tốn của bạn 10 giây và cứu bạn khỏi những đêm không ngủ.

### ✅ Self-check Phần 2

**Câu 1.** Nêu ít nhất hai lý do vì sao binary search / B-tree không dùng được để tìm hàng xóm gần nhất của vector.
> *Gợi ý đáp án:* (1) Vector nhiều chiều **không có thứ tự tuyến tính trên một trục duy nhất** để sắp xếp — sắp theo chiều nào cũng làm hai vector thực sự gần nhau bị đẩy ra xa nhau trong thứ tự. (2) Câu hỏi khác bản chất: B-tree trả lời "bằng đúng X" hoặc "trong khoảng [a,b]", còn vector search hỏi "gần nhất". (3) Lời nguyền số chiều làm cây phân hoạch không gian sập về `O(n)`.

**Câu 2.** Trong IVFFlat, `lists` và `probes` mỗi cái điều khiển gì? Tăng `probes` thì được gì mất gì?
> *Gợi ý đáp án:* `lists` = số cụm, chọn **lúc build**, muốn đổi phải build lại. `probes` = số cụm được ghé xem, chỉnh **lúc query**. Tăng `probes` → recall tăng (bớt bỏ sót hàng xóm ở cụm bên cạnh) nhưng latency tăng vì phải quét nhiều vector hơn.

**Câu 3.** Vì sao không nên tạo IVFFlat index khi bảng còn rỗng? HNSW có bị vấn đề này không?
> *Gợi ý đáp án:* IVFFlat cần dữ liệu để chạy k-means tìm centroid; bảng rỗng → phân cụm vô nghĩa → recall tệ, mà không báo lỗi gì. HNSW **không** bị, vì nó xây đồ thị bằng cách chèn dần từng node.

**Câu 4.** Bạn có 8000 vector. Có nên đánh index ANN không?
> *Gợi ý đáp án:* Không. Brute force exact ở quy mô này đã đủ nhanh và cho recall 100%. Đánh index chỉ làm giảm độ chính xác mà đổi lại rất ít tốc độ. Index chỉ đáng khi dữ liệu đủ lớn.

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Ở đây ta chui xuống bên dưới bề mặt. Tôi nói trước: mục 3.1 và 3.3 là hai chỗ khó nhất giáo trình. Nhưng tôi sẽ chạy tay từng bước và giải thích cả những ký hiệu mà sách hay bỏ qua, nên bạn cứ đi chậm.

### 3.1. HNSW — đồ thị nhiều tầng với "đường cao tốc"

**HNSW** = *Hierarchical Navigable Small World* — có phân tầng (hierarchical), đi lại được (navigable), trên một mạng lưới "thế giới nhỏ" (small world).

Ba từ đó đều mang nghĩa cụ thể, và tôi giải thích luôn từ khó nhất:

**Small world** (thế giới nhỏ) là một khái niệm có thật trong lý thuyết mạng, chỉ loại mạng lưới mà **hai điểm bất kỳ chỉ cách nhau vài bước nhảy**, dù mạng có rất nhiều điểm. Bạn có thể đã nghe ý tưởng "sáu độ phân cách" — rằng hai người bất kỳ trên Trái Đất chỉ cách nhau khoảng sáu mối quen biết. Đó chính là một mạng small world. Bí quyết của loại mạng này: đa số liên kết là **liên kết gần** (bạn bè hàng xóm), nhưng có **một số ít liên kết đi rất xa** (một người bạn ở nước ngoài) — và chính vài liên kết xa đó rút ngắn mọi khoảng cách một cách kỳ diệu.

HNSW xây dựng có chủ đích một mạng như vậy giữa các vector.

Vài thuật ngữ nữa cần có trước khi đi tiếp:

- **Graph** (đồ thị) — cấu trúc gồm các **node** (nút) nối với nhau bằng **edge** (cạnh). Ở đây mỗi node là một vector, mỗi cạnh nghĩa là "hai vector này là hàng xóm gần của nhau".
- **Greedy search** (tìm kiếm tham lam) — chiến lược đi từng bước, mỗi bước luôn chọn nước đi *có vẻ* tốt nhất ngay lúc đó, không nhìn xa hơn. Đơn giản và nhanh, nhưng có nhược điểm ta sẽ nói ở cuối mục.

#### Cấu trúc: nhiều tầng chồng lên nhau

- **Tầng dưới cùng (Layer 0)** chứa **mọi** node, các cạnh nối những node gần nhau — toàn "ngõ nhỏ", mỗi bước đi rất ngắn.
- **Các tầng trên** chỉ chứa một tập con ngày càng ít node, và các cạnh ở đó nối những node cách nhau rất xa — "đường cao tốc".
- Node được đưa lên tầng nào là **chọn ngẫu nhiên theo phân phối mũ**: đa số node chỉ nằm ở tầng đáy, càng lên cao càng thưa.

> **Analogy giao thông (dựng đầy đủ):** hãy tưởng tượng bạn cần đi từ Hà Nội tới một số nhà cụ thể trong một con hẻm ở Sài Gòn.
>
> Bạn không bò theo ngõ nhỏ suốt 1700 km. Bạn bay một chuyến (tầng cao, một bước nhảy cực xa) để tới đúng thành phố. Rồi bắt taxi trên đường lớn (tầng giữa) để tới đúng quận. Rồi cuối cùng mới đi bộ vào từng hẻm (tầng đáy) để tìm đúng số nhà.
>
> **Chỗ khớp với kỹ thuật:** mỗi tầng của HNSW làm đúng một việc — thu hẹp phạm vi theo một cấp độ. Tầng cao đưa bạn về đúng *vùng* của không gian vector; tầng đáy tinh chỉnh tới đúng *hàng xóm*. Và cũng như trong đời thật, số chặng bạn phải đi **không tăng tuyến tính theo khoảng cách** — đi xa gấp mười lần không có nghĩa là mất mười lần số chặng.

#### 🧩 [Ngoài bài gốc] Vì sao cấu trúc này cho `O(log n)` — mối liên hệ với skip list

Đây là một hiểu biết mà rất ít người nhắc tới nhưng làm mọi thứ sáng hẳn ra.

HNSW về bản chất là **skip list được tổng quát hoá từ danh sách một chiều lên đồ thị nhiều chiều**. Skip list là một cấu trúc dữ liệu kinh điển: một danh sách đã sắp xếp, bên trên chồng thêm các "tầng" chứa một phần ngẫu nhiên các phần tử để nhảy cóc cho nhanh. Nó cho `O(log n)` bằng đúng cơ chế: mỗi tầng cao hơn có số node ít đi khoảng một nửa, nên số tầng là `log n`, và ở mỗi tầng bạn chỉ đi vài bước.

HNSW áp dụng y hệt ý tưởng đó, chỉ thay "danh sách đã sắp xếp" bằng "đồ thị hàng xóm gần" — thứ dùng được cả khi dữ liệu **không sắp thứ tự được** (nhớ mục 1.5 và 2.1: đó chính là vấn đề gốc rễ của vector).

Nói được câu này trong phỏng vấn là dấu hiệu bạn hiểu chứ không thuộc: *"HNSW là skip list cho không gian mà bạn không thể sắp xếp."*

#### Ví dụ chạy tay: tìm số 12

Bài gốc có một ví dụ tìm số 12 nhưng nhãn tầng bị lỗi. Tôi dựng lại cho đúng và chạy từng bước.

Ta dùng các con số làm "vector 1 chiều" cho dễ hình dung (khoảng cách giữa hai số chính là hiệu tuyệt đối của chúng). Tập dữ liệu ở tầng đáy: **2, 4, 8, 12, 14, 16, 19**.

Cấu trúc nhiều tầng (node càng lên cao càng ít):

```
Layer 3 (đỉnh):   2
Layer 2:          2 ────────────── 14
Layer 1:          2 ────── 8 ───── 14 ────── 19
Layer 0 (đáy):    2 ── 4 ── 8 ── 12 ── 14 ── 16 ── 19
```

**Nhiệm vụ: tìm số gần 12 nhất. Điểm xuất phát: node 2 ở Layer 3.**

**Bước 1 — Layer 3.** Ta đang ở node 2. Tầng này không có node nào khác để đi. → Tụt xuống Layer 2, vẫn ở node 2.

**Bước 2 — Layer 2.** Ta ở node 2, hàng xóm duy nhất là 14.
- Khoảng cách hiện tại: `|2 − 12| = 10`
- Nếu đi tới 14: `|14 − 12| = 2`
- 2 nhỏ hơn 10 → **đi tới 14**. Ở 14 rồi, không còn hàng xóm nào tốt hơn ở tầng này. → Tụt xuống Layer 1, ở node 14.

*Hãy để ý điều vừa xảy ra: chỉ một bước nhảy đã đưa ta từ đầu này sang gần cuối tập dữ liệu. Đó là "đường cao tốc".*

**Bước 3 — Layer 1.** Ta ở node 14, hàng xóm là 8 và 19.
- Hiện tại: `|14 − 12| = 2`
- Đi tới 8: `|8 − 12| = 4` → tệ hơn.
- Đi tới 19: `|19 − 12| = 7` → tệ hơn.
- Không nước đi nào cải thiện → **đứng yên**. → Tụt xuống Layer 0, ở node 14.

**Bước 4 — Layer 0.** Ta ở node 14, hàng xóm là 12 và 16.
- Đi tới 12: `|12 − 12| = 0` → tốt hơn hẳn.
- **Đi tới 12.** Hàng xóm của 12 là 8 (`|8−12| = 4`) và 14 (`= 2`), đều tệ hơn.
- Không cải thiện được nữa → **dừng. Đáp án: 12.** ✅

Tổng cộng khoảng **4 bước** thay vì quét cả 7 phần tử. Với 7 phần tử thì lợi ích không rõ, nhưng hãy phóng to: với **1 triệu** vector, HNSW cần khoảng **20 bước** thay vì 1 triệu phép so. Đó chính là `O(log n)`.

#### Nhược điểm của greedy search — và vì sao `ef_search` tồn tại

Bạn có để ý ở Bước 3, thuật toán đã **đứng yên** vì không hàng xóm nào tốt hơn? Đó là điểm yếu cố hữu của greedy search: nó có thể mắc kẹt ở một **local minimum** (cực tiểu địa phương — một điểm mà mọi hướng đi xung quanh đều tệ hơn, nhưng nó không phải điểm tốt nhất trên toàn cục).

Trong ví dụ trên ta may mắn vì tầng dưới cứu được. Nhưng ở không gian nhiều chiều thật, điều này xảy ra thường xuyên.

Cách khắc phục: thay vì chỉ giữ **một** node hiện tại, thuật toán giữ **một danh sách các ứng viên tốt nhất** và mở rộng tìm kiếm từ tất cả chúng. Độ dài danh sách đó chính là **`ef_search`**.

Bây giờ thì tham số này không còn là một con số bí ẩn nữa — bạn hiểu chính xác nó mua cho bạn cái gì: **`ef_search` là số đường thoát hiểm khỏi việc mắc kẹt.** Càng nhiều đường thoát thì càng khó bỏ sót hàng xóm thật, nhưng càng tốn thời gian.

#### Ba tham số của HNSW

```sql
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**`m`** — số cạnh tối đa mà mỗi node được nối ở mỗi tầng (mặc định 16). Cao hơn → đồ thị dày hơn, nhiều đường đi hơn → recall tốt hơn, nhưng index to hơn và build lâu hơn. Đây là tham số **lúc build**.

**`ef_construction`** — số ứng viên hàng xóm được xem xét *khi xây* đồ thị, mỗi lần chèn một node mới (mặc định 64; bài gốc cũng nhắc con số này). Cao hơn → đồ thị chất lượng hơn → recall tốt hơn, nhưng **build chậm hơn**. Cũng là tham số **lúc build**.

**`ef_search`** — số ứng viên được giữ *khi tìm kiếm* (mặc định 40). Bài gốc **không nhắc tới tham số này**, nhưng trong thực tế nó là núm xoay quan trọng nhất, vì nó là thứ duy nhất bạn chỉnh được **lúc chạy** mà không phải build lại index:

```sql
SET hnsw.ef_search = 100;   -- tăng recall khi cần độ chính xác cao hơn
```

Cách nhớ gọn ba tham số: **`m` và `ef_construction` là những gì bạn quyết một lần lúc xây; `ef_search` là cái núm bạn xoay hằng ngày.**

> **🧩 Mẹo staff — một cái bẫy production thật:** trên hệ thống thật, hãy dùng `SET LOCAL` bên trong `BEGIN ... COMMIT` thay vì `SET` toàn cục.
>
> Lý do: `SET` gắn giá trị vào **connection** (kết nối tới database). Ứng dụng thật hầu như luôn dùng **connection pool** (bể kết nối — một tập kết nối tạo sẵn, được **dùng lại luân phiên** cho nhiều request khác nhau, phổ biến nhất là PgBouncer). Bạn `SET ef_search = 500` cho một query nặng; kết nối đó quay về pool; request của người dùng khác vớ đúng nó và bỗng dưng chạy chậm gấp 10 lần. Đây là loại bug ngẫu nhiên, không tái hiện được trên máy dev, và cực kỳ khó truy.

### 3.2. Big-O: brute force vs IVFFlat vs HNSW

Đây là bảng mà người phỏng vấn rất hay yêu cầu bạn dựng lại từ đầu.

| Phương pháp | Độ phức tạp query | Thời gian build | Bộ nhớ | Cần data để build? | Chịu data động |
|---|---|---|---|---|---|
| Brute force (không index) | `O(n · d)` | không có | thấp | — | tốt (recall luôn 100%) |
| **IVFFlat** | ~`O(lists·d + (n/lists)·probes·d)` | **nhanh** | **thấp** | **có** (k-means) | kém (cụm lệch dần) |
| **HNSW** | ~**`O(log n · d)`** | chậm | cao (đồ thị nằm trong RAM) | không | **tốt** |

(`n` = số vector, `d` = số chiều.)

Đọc bảng này theo kiểu "ai đặt cược vào cái gì":

- **Brute force** không đặt cược gì cả. Nó trả giá bằng thời gian query, nhưng đổi lại luôn đúng và không tốn công vận hành.
- **IVFFlat** đặt cược rằng *dữ liệu tụ thành cụm rõ ràng và không thay đổi nhiều*. Nếu cược đúng, bạn được index nhẹ và build nhanh.
- **HNSW** đặt cược rằng *bạn có đủ RAM và đủ kiên nhẫn lúc build*. Nếu cược đúng, bạn được query nhanh nhất và recall cao nhất.

#### 🧩 [Ngoài bài gốc] Vì sao công thức gợi ý `lists ≈ sqrt(n)`?

Con số này thường được đưa ra như một quy tắc phải học thuộc. Nhưng nó suy ra được, và biết cách suy ra là điểm cộng lớn.

Nhìn vào chi phí query của IVFFlat với `probes = 1`, nó gồm hai phần cộng lại:

```
Phần 1: so query với tất cả centroid   →  chi phí tỷ lệ với  lists
Phần 2: quét hết vector trong 1 cụm    →  chi phí tỷ lệ với  n / lists
```

Hai phần này **kéo ngược chiều nhau**: chia càng nhiều cụm thì phần 1 tăng nhưng phần 2 giảm, và ngược lại. Tổng chi phí đạt nhỏ nhất khi hai phần **cân bằng nhau**:

```
lists = n / lists   →   lists² = n   →   lists = sqrt(n)
```

Đúng bằng công thức gợi ý. Còn quy tắc `n/1000` cho bảng dưới 1 triệu dòng chỉ là một biến thể thực dụng, nhắm tới việc mỗi cụm chứa khoảng 1000 phần tử — đủ nhỏ để quét nhanh, đủ lớn để k-means không bị nhiễu.

Nói được đoạn suy luận này trong phỏng vấn có sức nặng hơn nhiều so với việc đọc thuộc con số.

### 3.3. Curse of dimensionality — vì sao NN "khó" ở số chiều cao

Ta đã nhắc hiện tượng này ở mục 2.1. Giờ đào sâu, vì nói được nó trong phỏng vấn là điểm cộng lớn.

Ở 2 hay 3 chiều, việc chia không gian thành các ô (kd-tree) hoạt động rất tốt. Nhưng khi `d` lên tới hàng trăm hoặc hàng nghìn, ba chuyện xảy ra:

**1. Thể tích không gian tăng theo hàm mũ, nên dữ liệu trở nên cực kỳ thưa.**
> Ví dụ cụ thể để cảm nhận: chia mỗi chiều thành 10 khoảng. Với 2 chiều bạn có 10² = 100 ô. Với 10 chiều đã là 10¹⁰ = 10 tỷ ô. Với 384 chiều thì số ô lớn hơn số nguyên tử trong vũ trụ quan sát được. Bạn có 1 triệu điểm dữ liệu rải vào đó — gần như mọi ô đều rỗng, và hai điểm bất kỳ gần như chắc chắn nằm ở hai ô khác nhau. Việc "chia ô" mất sạch ý nghĩa.

**2. Khoảng cách giữa điểm gần nhất và điểm xa nhất co lại gần bằng nhau.** Đây là phần phản trực giác nhất. Với số chiều rất cao, tỷ lệ `(khoảng cách xa nhất / khoảng cách gần nhất)` tiến dần về 1. Nói cách khác, **khái niệm "gần nhất" mất dần khả năng phân biệt**.
> Vì sao lại thế? Trực giác: khoảng cách là tổng đóng góp của tất cả các chiều. Với hàng nghìn chiều, theo luật số lớn, tổng đó có xu hướng hội tụ về một giá trị trung bình chung cho mọi cặp điểm — các chênh lệch ngẫu nhiên ở từng chiều triệt tiêu lẫn nhau.

**3. Vì hai lý do trên, cây phân hoạch phải duyệt gần hết các nhánh** mới dám khẳng định đã tìm đúng → nó **sập về `O(n)`**, tức không nhanh hơn brute force.

Đó là lý do ngành này chuyển sang **approximate**, và chuyển sang các cấu trúc kiểu **đồ thị** (HNSW) hoặc **cụm** (IVFFlat) thay vì cây chính xác.

> **Câu chốt đáng nhớ:** *"Ở số chiều cao, việc chia nhỏ không gian trở nên vô nghĩa — nên thay vì chia không gian, ta xây một mạng lưới đường đi trong không gian đó."* Đó chính xác là điều HNSW làm.

### 3.4. 🧩 [Ngoài bài gốc] DiskANN — "người thứ ba" mà bài gốc không nhắc

Bài gốc chỉ nêu IVFFlat và HNSW. Ở thời điểm 2026 có thêm một lựa chọn thứ ba đáng biết: **DiskANN**, đến với Postgres qua extension **`pgvectorscale`** của Timescale.

Trước hết hai từ cần phân biệt:
- **RAM** (bộ nhớ trong) — rất nhanh, nhưng đắt và có giới hạn (một máy chủ thường vài chục tới vài trăm GB).
- **SSD** (ổ cứng thể rắn) — chậm hơn RAM khoảng vài chục lần, nhưng rẻ hơn nhiều và dung lượng lớn hơn nhiều.

HNSW **đặt cược rằng toàn bộ đồ thị vừa trong RAM**. Khi cược đó không còn đúng — ví dụ 100 triệu vector — hệ thống phải liên tục đọc từ đĩa một cách ngẫu nhiên và hiệu năng sụp đổ.

DiskANN được thiết kế cho đúng tình huống đó: nó **cố tình giữ phần lớn đồ thị trên SSD**, tổ chức dữ liệu sao cho mỗi bước tìm kiếm chỉ cần rất ít lần đọc đĩa, kết hợp với **binary quantization** (nén mỗi con số xuống còn 1 bit) để phần cần nằm trong RAM trở nên rất nhỏ.

Hai điểm bán hàng đáng nhớ:
1. Cùng một tập dữ liệu, index có thể nhỏ hơn HNSW nhiều lần.
2. Hỗ trợ vector tới **16000 chiều** một cách tự nhiên — trong khi HNSW và IVFFlat của pgvector giới hạn **2000 chiều** (xem mục 3.6).

> Câu chốt để nói trong phỏng vấn: *"HNSW đặt cược rằng đồ thị vừa RAM. DiskANN là câu trả lời cho lúc cược đó sai."*

### 3.5. Tự implement để hiểu tận gốc

Cách chắc chắn nhất để thực sự hiểu một thuật toán là tự viết lại phiên bản đơn giản của nó.

#### (a) Binary search — nền tảng của cả bài

```python
def binary_search(sorted_names, target):
    """Trả về (vị trí tìm thấy, số bước đã đi). Trả về -1 nếu không có."""
    lo, hi = 0, len(sorted_names) - 1   # lo/hi là hai đầu của phạm vi còn lại
    steps = 0

    while lo <= hi:                     # còn phạm vi để tìm thì còn lặp
        steps += 1
        mid = (lo + hi) // 2            # điểm giữa; // là phép chia lấy phần nguyên

        if sorted_names[mid] == target:
            return mid, steps           # trúng đích

        elif sorted_names[mid] < target:
            lo = mid + 1                # mục tiêu nằm bên PHẢI -> vứt bỏ nửa trái

        else:
            hi = mid - 1                # mục tiêu nằm bên TRÁI  -> vứt bỏ nửa phải

    return -1, steps                    # hết phạm vi mà chưa thấy -> không tồn tại


names = sorted(["Alexander", "Amelia", "Ava", "Benjamin", "Charlotte",
                "Daniel", "Emily", "Emma", "Grace", "Isabella",
                "Jack", "James", "Liam", "Lucas", "Mia",
                "Michael", "Noah", "Oliver", "Sophia", "William"])

print(binary_search(names, "Jack"))     # -> (10, 4): vị trí 10 (đếm từ 0), đúng 4 bước
```

Con số 4 bước này khớp chính xác với phần chạy tay ở mục 1.4. Hãy thử đổi tên cần tìm và xem số bước thay đổi thế nào — nó luôn nằm quanh 4–5, không bao giờ tới 20.

#### (b) MiniIVF — tự tay thấy trade-off của `probes`

```python
import numpy as np
from sklearn.cluster import KMeans

class MiniIVF:
    def build(self, corpus, n_lists=16):
        """Giai đoạn BUILD: chạy k-means để chia cụm và tính tâm."""
        self.corpus = corpus
        km = KMeans(n_clusters=n_lists, n_init="auto").fit(corpus)
        self.centroids = km.cluster_centers_   # toạ độ tâm của mỗi cụm
        self.assign    = km.labels_            # mỗi vector thuộc cụm số mấy
        return self

    def search(self, q, k=5, probes=2):
        """Giai đoạn QUERY: chỉ tìm trong `probes` cụm gần nhất."""
        # Bước 1: so query với các TÂM cụm. Rẻ: chỉ n_lists phép so, không phải n.
        dist_to_centroids = np.linalg.norm(self.centroids - q, axis=1)
        near = np.argsort(dist_to_centroids)[:probes]   # chọn `probes` cụm gần nhất

        # Bước 2: brute force CHỈ trong các cụm đã chọn, bỏ qua toàn bộ phần còn lại.
        cand = np.where(np.isin(self.assign, near))[0]  # chỉ số các vector ứng viên
        d = np.linalg.norm(self.corpus[cand] - q, axis=1)
        top = np.argsort(d)[:k]
        return cand[top], d[top]
```

**Bài tập tôi thực sự khuyên bạn làm** (mất 5 phút và đáng giá hơn cả giờ đọc lý thuyết):

1. Sinh ngẫu nhiên 10.000 vector 128 chiều.
2. Tính đáp án đúng bằng brute force (`np.linalg.norm` với toàn bộ corpus).
3. Chạy `MiniIVF.search` với `probes = 1`, rồi `probes = 4`, rồi `probes = 16`.
4. Với mỗi lần, đếm xem trong 5 kết quả trả về có bao nhiêu cái trùng với đáp án đúng.

Con số đó chính là **recall@5** của bạn. Bạn sẽ **tự mắt thấy** nó tăng dần khi `probes` tăng, và thấy thời gian chạy cũng tăng theo. Đọc mười lần về trade-off recall–tốc độ không bằng nhìn thấy nó xảy ra một lần trên màn hình của chính mình.

### 3.6. Edge cases — các trường hợp biên phải biết

**Edge case** (trường hợp biên) nghĩa là: tình huống nằm ở rìa điều kiện bình thường, ít gặp nhưng khi gặp thì làm hệ thống hỏng theo cách bất ngờ. Người viết code tốt là người nghĩ tới chúng *trước khi* chúng xảy ra.

**1. Giới hạn 2000 chiều.** Kiểu `vector` của pgvector **lưu** được tới 16000 chiều, nhưng index HNSW và IVFFlat chỉ **index** được tối đa **2000 chiều**:

```sql
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
-- ERROR: column cannot have more than 2000 dimensions for hnsw index
```

Model `text-embedding-3-large` của OpenAI cho 3072 chiều → dính lỗi này ngay. Ba cách xử lý:
- **`halfvec`** — lưu mỗi số bằng 16 bit thay vì 32 bit; index được tới khoảng 4000 chiều, dung lượng giảm một nửa, mất mát recall thường không đo thấy:
  ```sql
  CREATE INDEX ON docs USING hnsw ((embedding::halfvec(3072)) halfvec_cosine_ops);
  --                                          ^^ :: là cú pháp ép kiểu của Postgres
  ```
- **Matryoshka** (đặt theo tên búp bê gỗ Nga lồng nhau) — kỹ thuật huấn luyện sao cho thông tin quan trọng dồn về đầu vector, nhờ đó cắt bớt đuôi (3072 → 1024) mà mất rất ít chất lượng.
- **DiskANN** qua `pgvectorscale` — hỗ trợ tới 16000 chiều, không cần thủ thuật.

**2. IVFFlat sau khi nạp thêm nhiều dữ liệu** → centroid lệch so với hình dạng thật của dữ liệu → recall tụt âm thầm → cần `REINDEX` định kỳ.

**3. `VACUUM` trên bảng có index HNSW rất chậm.** (`VACUUM` là tiến trình dọn dẹp của Postgres, thu hồi không gian của các dòng đã xoá.) Cách xử lý thường dùng: reindex trước rồi mới vacuum.

**4. Recall tụt trong im lặng** khi tham số đặt quá thấp. Không có lỗi nào báo. Bắt buộc phải **đo**, không được đoán (mục 4.3).

**5. Ops class sai** → index bị bỏ qua, rơi về sequential scan âm thầm (mục 2.5).

**6. Giá trị `NULL`.** Những dòng có `embedding IS NULL` sẽ **không bao giờ** xuất hiện trong kết quả — chúng vô hình. Hay xảy ra khi pipeline sinh embedding lỗi ở vài dòng. Nên có kiểm tra định kỳ:
```sql
SELECT count(*) FROM items WHERE embedding IS NULL;
```

### ✅ Self-check Phần 3

**Câu 1.** HNSW có độ phức tạp query khoảng bao nhiêu? Giải thích bằng analogy vì sao "đồ thị nhiều tầng" cho tốc độ đó.
> *Gợi ý đáp án:* ~`O(log n · d)`. Vì mỗi tầng thu hẹp phạm vi một cấp: tầng cao nhảy xa để về đúng vùng, tầng thấp tinh chỉnh tới đúng điểm — giống bay chuyến quốc tế rồi mới bắt taxi rồi mới đi bộ. Số tầng tỷ lệ với `log n`, và ở mỗi tầng chỉ đi vài bước.

**Câu 2.** `ef_construction` và `ef_search` khác nhau chỗ nào?
> *Gợi ý đáp án:* `ef_construction` là tham số **lúc build** — số ứng viên xét khi chèn mỗi node vào đồ thị; muốn đổi phải build lại index. `ef_search` là tham số **lúc query** — số ứng viên giữ trong lúc tìm kiếm; chỉnh được ngay lúc chạy, và nó là núm xoay recall–tốc độ dùng hằng ngày.

**Câu 3.** "Curse of dimensionality" làm kd-tree hỏng như thế nào ở số chiều cao?
> *Gợi ý đáp án:* Ở chiều cao, thể tích tăng theo hàm mũ nên dữ liệu cực thưa, và khoảng cách gần nhất với xa nhất co lại gần bằng nhau. Hệ quả là cây không loại bỏ được nhánh nào một cách chắc chắn, phải duyệt gần hết → sập về `O(n)`, không nhanh hơn brute force.

**Câu 4.** Vì sao công thức gợi ý cho `lists` lại là `sqrt(n)`?
> *Gợi ý đáp án:* Chi phí query IVFFlat gồm hai phần kéo ngược chiều nhau: so với centroid (tỷ lệ với `lists`) và quét trong cụm (tỷ lệ với `n/lists`). Tổng nhỏ nhất khi hai phần cân bằng: `lists = n/lists` → `lists = sqrt(n)`.

**Câu 5.** Greedy search có nhược điểm gì, và tham số nào của HNSW dùng để bù đắp?
> *Gợi ý đáp án:* Nó có thể mắc kẹt ở **local minimum** — một điểm mà mọi hướng xung quanh đều tệ hơn nhưng không phải điểm tốt nhất toàn cục. `ef_search` bù đắp bằng cách giữ nhiều ứng viên cùng lúc thay vì chỉ một, tạo thêm "đường thoát".

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)

> Đây là phần phân biệt senior với staff. Senior trả lời *"index nào và dùng thế nào"*. Staff trả lời *"ở quy mô và ràng buộc CỦA TÔI thì cái nào, nó gãy ở đâu, tốn bao nhiêu tiền, ai phải trực đêm vì nó, và khi nào ta nên bỏ lựa chọn hiện tại"*.
>
> Vài từ vựng nghề nghiệp cần giải thích trước:
> - **Staff Engineer** — cấp bậc kỹ sư cao hơn senior. Đặc trưng không phải code giỏi hơn, mà là **phạm vi ảnh hưởng**: quyết định kiến trúc, đánh đổi dài hạn, ảnh hưởng tới nhiều team.
> - **Production** — môi trường chạy thật, phục vụ người dùng thật (đối lập với dev/staging là môi trường thử nghiệm).
> - **SLA** (viết tắt của *Service Level Agreement*, cam kết mức dịch vụ) — lời hứa đo được về chất lượng, ví dụ "95% truy vấn phải trả lời dưới 100 mili-giây".
> - **Latency** (độ trễ) — thời gian từ lúc gửi truy vấn tới lúc nhận kết quả.
> - **QPS** (*Queries Per Second*) — số truy vấn mỗi giây mà hệ thống phải chịu.

### 4.1. Chọn index theo quy mô — quyết định thật ở tầm staff

**Câu hỏi đúng không phải "IVFFlat hay HNSW" một cách trừu tượng, mà là "ở quy mô và ràng buộc của tôi thì cái nào".** Người mới tìm "cái tốt nhất"; người có kinh nghiệm hỏi "tốt nhất *cho ràng buộc nào*".

Đây là bảng quyết định tôi thực sự dùng:

| Tình huống | Lựa chọn | Lý do |
|---|---|---|
| **Dưới ~10.000–50.000 vector** | **Đừng đánh index** | Brute force exact đã đủ nhanh (vài mili-giây), recall 100%, không phải tune gì cả. Đánh index lúc này chỉ tự chuốc recall thấp mà chẳng được bao nhiêu tốc độ. |
| **Vài trăm nghìn tới vài triệu** | **HNSW** | Mặc định. Recall cao, latency thấp, chịu dữ liệu thêm mới tốt. Chấp nhận build lâu và tốn RAM. |
| **RAM là ràng buộc chính, dữ liệu khá tĩnh, cần build lại thường xuyên** | **IVFFlat** | Nhẹ, build nhanh. Chấp nhận recall thấp hơn và phải reindex khi dữ liệu đổi nhiều. |
| **Dataset khổng lồ (hàng trăm triệu), RAM không đủ** | **DiskANN** (`pgvectorscale`) | Đưa đồ thị lên SSD + nén, cắt mạnh chi phí RAM. |
| **Vector trên 2000 chiều** | `halfvec` + HNSW, hoặc DiskANN | Vì giới hạn 2000 chiều của HNSW/IVFFlat. |

Dòng đầu tiên là dòng quan trọng nhất và cũng là dòng hay bị bỏ qua nhất. **"Không phải mọi thứ đều cần index"** — nêu được điều này trong phỏng vấn là tín hiệu tư duy staff rõ ràng, vì nó cho thấy bạn tối ưu cho *kết quả*, không phải cho việc thể hiện kiến thức.

### 4.2. Chi phí build, bộ nhớ, và bottleneck vận hành

**Bottleneck** (điểm nghẽn) là bộ phận chạm giới hạn trước tiên và kéo cả hệ thống chậm lại. Tìm đúng bottleneck quan trọng hơn tối ưu mọi thứ.

#### Bottleneck 1 — RAM. Hãy tính bằng con số.

Với HNSW, đồ thị phải nằm gọn trong RAM thì mới nhanh. Hãy tính cho cụ thể:

```
Mỗi con số trong vector       = 4 byte (kiểu float 32 bit)
Một vector 1536 chiều         = 1536 × 4 = 6.144 byte ≈ 6 KB
1 triệu vector                = 6 GB
100 triệu vector              = 600 GB   ← chỉ riêng vector thô!
```

Và đó **chưa tính đồ thị HNSW**. Ước lượng nhanh cho phần đồ thị: mỗi node lưu tối đa khoảng `2m` liên kết ở tầng đáy, mỗi liên kết khoảng 4 byte. Với `m = 16`:

```
100.000.000 node × 32 liên kết × 4 byte ≈ 12,8 GB   (chỉ riêng phần liên kết)
```

Kết luận: ở quy mô này **RAM là ràng buộc số một**, không phải CPU. Đó là lúc phải cân nhắc **quantization** (nén vector: `halfvec` cắt 50%, binary quantization cắt tới ~32 lần, kèm bước re-rank chính xác trên top-N để bù lại chất lượng) hoặc chuyển sang **DiskANN**.

#### Bottleneck 2 — Thời gian build

HNSW build là `O(n log n)`, có thể mất **hàng giờ** ở quy mô lớn. Bốn cách tăng tốc:

1. **Nạp toàn bộ dữ liệu xong rồi mới build index.** Luôn nhanh hơn nhiều so với build trước rồi chèn dần từng dòng.
2. **Tăng `maintenance_work_mem`** — lượng RAM Postgres được phép dùng cho thao tác bảo trì như build index. Đặt đủ lớn để phần đang xử lý vừa trong RAM, nhưng đừng vượt 50–60% RAM máy kẻo hệ điều hành hết chỗ thở.
3. **Tăng `max_parallel_maintenance_workers`** — số tiến trình build song song.
4. **Dùng `CREATE INDEX CONCURRENTLY`** — build index mà **không khoá thao tác ghi** lên bảng. Đánh đổi: chậm hơn (phải quét bảng hai lượt) và có thể thất bại giữa chừng để lại một index hỏng cần dọn. Nhưng trên production, đây gần như luôn là lựa chọn đúng, vì cách thông thường sẽ chặn mọi `INSERT`/`UPDATE` trong suốt hàng giờ.

```sql
CREATE INDEX CONCURRENTLY items_emb_idx
ON items USING hnsw (embedding vector_cosine_ops);
-- Không khoá write. Bắt buộc dùng trên bảng đang phục vụ người dùng thật.
```

#### Bottleneck 3 — Dữ liệu động

- **IVFFlat** lệch cụm khi ghi nhiều → cần lịch reindex định kỳ.
- **HNSW** chịu insert tốt hơn nhiều, nhưng `VACUUM` trên nó rất chậm → cần chiến lược bảo trì riêng.

### 4.3. Monitoring recall — thứ phân biệt staff rõ nhất

Đây là ý quan trọng nhất của cả Phần 4, nên tôi nói thật kỹ.

Với ANN, **không có lỗi biên dịch, không có exception, không có cảnh báo nào** khi chất lượng tìm kiếm suy giảm. Hệ thống vẫn chạy, vẫn nhanh, vẫn trả về đúng số lượng kết quả — chỉ là các kết quả đó ngày càng kém liên quan. Người dùng không báo lỗi; họ chỉ lặng lẽ dùng ít đi.

So sánh cho rõ: nếu database chậm, bạn có biểu đồ latency báo động. Nếu service sập, bạn có alert. Nhưng nếu recall tụt từ 95% xuống 70%, **tuyệt đối không có gì xảy ra** ngoài việc sản phẩm của bạn âm thầm tệ đi.

Cho nên bạn **phải chủ động đo**. Cách đo: so kết quả có index với kết quả exact — dùng exact làm **ground truth** (sự thật nền, tức đáp án đúng để đối chiếu).

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép Postgres KHÔNG dùng index -> buộc phải exact search.
                                    -- Kết quả này là ground truth (đúng tuyệt đối).
SELECT id FROM items ORDER BY embedding <=> :q LIMIT 10;
COMMIT;

-- Sau đó chạy lại đúng query đó với index bật bình thường,
-- rồi tính tỷ lệ trùng nhau giữa hai danh sách ID -> đó là recall@10.
```

**Cách làm ở production:** lấy mẫu vài trăm truy vấn thật mỗi ngày, chạy đối chiếu như trên **trên một read replica** (bản sao chỉ đọc — để việc đo không ảnh hưởng người dùng thật), rồi vẽ **recall@10 theo thời gian** lên dashboard.

Khi đường đó đi xuống, bạn biết cần tăng `ef_search`/`probes`/`m` hoặc reindex — **trước khi** phía kinh doanh cảm nhận được vấn đề.

> **Câu chốt để nói trong phỏng vấn:** *"Không đo recall là bay mù. Đây là hệ thống duy nhất tôi biết mà chất lượng có thể sụp hoàn toàn trong khi mọi biểu đồ vẫn xanh."*

### 4.4. 🧩 [Ngoài bài gốc] Filtered search và overfiltering

Trong thực tế, gần như mọi tìm kiếm đều đi kèm điều kiện lọc: "tìm sản phẩm giống nhất **nhưng chỉ trong danh mục này, của khách hàng này, còn hàng**".

Và ở đây có một cái bẫy mà người rất giỏi cũng vấp, vì thứ tự thực hiện bên trong không như bạn tưởng:

**Index ANN lấy khoảng `ef_search` ứng viên gần nhất TRƯỚC, rồi Postgres mới áp mệnh đề `WHERE` lên nhóm ứng viên đó SAU.**

Hệ quả: nếu điều kiện lọc của bạn hiếm (ví dụ danh mục đó chỉ chiếm 0,5% dữ liệu), thì trong 40 ứng viên ban đầu có thể chỉ 1–2 cái sống sót qua bộ lọc. Bạn yêu cầu `LIMIT 10` nhưng nhận về 2 dòng — dù trong bảng rõ ràng có hàng nghìn dòng thoả điều kiện.

Tên chuyên môn của hiện tượng: **overfiltering** (lọc quá tay).

Cách chữa — **iterative index scan** (quét index lặp lại), tính năng chủ lực từ pgvector 0.8.0:

```sql
SET hnsw.iterative_scan = relaxed_order;  -- cho phép index tự động quét thêm
                                          -- cho tới khi gom đủ số kết quả yêu cầu
SET hnsw.max_scan_tuples = 20000;         -- trần an toàn: quét thêm tối đa 20.000 dòng
                                          -- rồi dừng, tránh một query xấu quét cả bảng
```

Hai lựa chọn giá trị: `strict_order` giữ đúng thứ tự khoảng cách tuyệt đối; `relaxed_order` cho phép thứ tự xê dịch chút ít nhưng nhanh hơn đáng kể, thường giữ được 95–99% chất lượng. Với đa số hệ thống thật, `relaxed_order` là lựa chọn đúng.

**Cách phòng tốt hơn ở tầng thiết kế:** nếu bạn *biết trước* mọi query đều lọc theo `tenant_id`, hãy **partition** bảng theo cột đó (chia bảng lớn thành nhiều bảng con theo quy tắc). Khi ấy Postgres **prune** (cắt bỏ) toàn bộ các mảnh không liên quan *trước khi* index vector chạy, và mỗi mảnh có index riêng, nhỏ hơn, vừa RAM hơn.

> **Nguyên tắc nên thuộc:** *partition theo cách dữ liệu được truy cập, không phải theo kích thước.*

### 4.5. Ảnh hưởng tổ chức & cách nói với người không kỹ thuật

Một staff engineer phải giải thích được quyết định kỹ thuật cho **stakeholder** (các bên liên quan — quản lý sản phẩm, sếp, phòng tài chính) bằng ngôn ngữ của *họ*: **chi phí và rủi ro**, không phải thuật ngữ.

**Ví dụ cách nói với PM hoặc sếp:**

> *"Index cho tìm kiếm giống như mục lục ở cuối một cuốn sách: ta trả một chút chi phí (thêm bộ nhớ, thêm thời gian lúc xây dựng) để đổi lấy tốc độ tìm nhanh gấp hàng nghìn lần. Riêng với tìm kiếm bằng vector, ta còn phải đánh đổi thêm một chút độ chính xác nữa — nhưng đó là thứ điều chỉnh được bằng vài núm vặn, tuỳ theo mức độ chính xác mà sản phẩm cần. Với lượng dữ liệu hiện tại thì chưa cần; nó chỉ đáng làm khi ta lớn lên."*

Chú ý: đoạn trên không có chữ HNSW, không có recall, không có `O(log n)`. Nó nói bằng ba thứ người nghe quan tâm — **tiền, tốc độ, và một sự đánh đổi rõ ràng**. Và nó kết thúc bằng một lời thừa nhận trung thực về việc "chưa cần ngay", điều xây dựng lòng tin tốt hơn mọi lời hứa.

**Ảnh hưởng tới roadmap:** chọn và tune index **không phải việc làm một lần rồi quên**. Recall và latency cần được đo liên tục; index cần được dựng lại khi dữ liệu đổi nhiều. Hãy coi đây là **hạ tầng có vòng đời**, không phải một thứ "set-and-forget". Nói được điều này với PM giúp bạn xin được ngân sách thời gian cho việc bảo trì — thứ mà các team thường quên và phải trả giá sau đó.

**Team topology (cấu trúc đội ngũ):** vì tất cả các index này nằm ngay trong Postgres, đội backend hiện tại tự vận hành được. Không phải thêm một hệ thống tìm kiếm riêng, không phải tuyển người biết một stack mới, không tạo ra "ốc đảo tri thức" mà chỉ một người trong công ty hiểu. Đây là lập luận tổ chức mạnh không kém lập luận kỹ thuật — và là kiểu lập luận rất được đánh giá cao ở phỏng vấn cấp staff.

### 4.6. Câu hỏi system design mẫu + hướng trả lời của staff

> **Đề bài:** *"Thiết kế vector search cho 100 triệu embedding, yêu cầu p95 dưới 100ms, có lọc theo tenant và category, dữ liệu cập nhật hàng ngày, ngân sách RAM hạn chế."*

Trước hết, một khái niệm cần giải thích vì nó nằm ngay trong đề bài: **p95** là **bách phân vị thứ 95** — giá trị mà 95% truy vấn nhanh hơn nó. Người ta đo p50/p95/p99 thay vì đo trung bình, vì trung bình bị các truy vấn nhanh kéo xuống và **che giấu** nhóm chậm. Nếu 99 truy vấn chạy 10ms và 1 truy vấn chạy 5 giây, trung bình là ~60ms nghe rất ổn — nhưng đã có một người dùng thật chờ 5 giây và có thể đã bỏ đi.

**Khung trả lời — 7 bước, và nhớ nói to các trade-off:**

**1. Làm rõ đề bài trước khi vẽ gì cả.** Hỏi: QPS bao nhiêu? Mục tiêu recall là bao nhiêu? Tần suất cập nhật cụ thể? Ngân sách RAM cụ thể là bao nhiêu GB? Vector bao nhiêu chiều?
> *Vì sao quan trọng:* staff engineer không thiết kế cho đề bài tưởng tượng. Việc hỏi trước cho thấy bạn hiểu rằng câu trả lời đúng **phụ thuộc vào ràng buộc**, và đây là bước ứng viên hay bỏ qua rồi mất điểm ngay lập tức.

**2. Chọn index theo ràng buộc — và tính nhẩm ngay tại chỗ.** 100 triệu × 1536 chiều × 4 byte ≈ **600 GB** chỉ riêng vector thô → HNSW thuần **không vừa RAM** với ngân sách hạn chế. Vậy hai hướng: **DiskANN (`pgvectorscale`)**, hoặc **HNSW kèm quantization** (`halfvec` hoặc binary) cộng **two-stage re-rank** (lấy thô top-N bằng index nén, rồi tính lại chính xác trên N ứng viên đó).
> *Mẹo phỏng vấn:* việc bạn **tự nhẩm ra con số 600 GB** ngay tại chỗ gây ấn tượng mạnh hơn mọi thuật ngữ bạn liệt kê. Nó chứng tỏ bạn suy nghĩ bằng ràng buộc vật lý chứ không bằng buzzword.

**3. Xử lý filter.** Partition theo `tenant_id` (và có thể `category`) để planner prune sớm; bật `iterative_scan` chống overfiltering; index B-tree thông thường trên các cột lọc.

**4. Build và cập nhật.** Nạp dữ liệu xong rồi mới build; dùng `CREATE INDEX CONCURRENTLY` để không khoá ghi; có lịch reindex cho phần dữ liệu động. Lưu ý HNSW chịu insert hằng ngày tốt hơn IVFFlat — với yêu cầu "cập nhật hàng ngày" của đề bài, đây là một điểm cộng nghiêng về HNSW.

**5. Latency.** Tune `ef_search` theo SLA p95 — bắt đầu thấp rồi tăng dần tới khi recall đạt yêu cầu, đừng đặt cao ngay từ đầu. Dùng read replica cho lưu lượng tìm kiếm.

**6. Giám sát.** recall@10 (so exact định kỳ trên replica), p99 latency, kích thước index và RAM, tỷ lệ query bị overfiltering.

**7. Nói rõ khi nào KHÔNG cần ANN.** Nếu kiến trúc là multi-tenant và mỗi tenant chỉ có vài nghìn vector, thì sau khi partition theo tenant, **exact search trong từng partition còn vừa nhanh vừa chính xác 100%** — không cần index ANN chút nào.
> **Đây là bước ghi điểm cao nhất.** Nêu được rằng giải pháp phức tạp có thể không cần thiết, sau khi đã chứng minh mình hiểu nó, là dấu hiệu staff rõ ràng nhất. Người phỏng vấn không tìm người biết nhiều thuật toán nhất; họ tìm người biết khi nào **không** dùng chúng.

### ✅ Self-check Phần 4

**Câu 1.** Bạn có bảng 5.000 vector. Nên đánh index gì?
> *Gợi ý đáp án:* Không đánh index gì cả. Brute force exact ở quy mô này chạy trong vài mili-giây và cho recall 100%. Đánh index ANN chỉ làm giảm độ chính xác mà lợi ích tốc độ không đáng kể.

**Câu 2.** Vì sao `CREATE INDEX CONCURRENTLY` gần như luôn là lựa chọn đúng trên production?
> *Gợi ý đáp án:* Vì `CREATE INDEX` thông thường **khoá mọi thao tác ghi** lên bảng trong suốt quá trình build — với HNSW ở quy mô lớn có thể là hàng giờ, tức là hàng giờ ứng dụng không ghi được. `CONCURRENTLY` chậm hơn và có thể thất bại giữa chừng, nhưng không chặn người dùng.

**Câu 3.** Vì sao phải chủ động giám sát recall trong khi các chỉ số khác đều có alert tự động?
> *Gợi ý đáp án:* Vì recall suy giảm **không gây lỗi**. Hệ thống vẫn chạy, vẫn nhanh, vẫn trả đủ số kết quả — chỉ là chúng kém liên quan dần. Không có exception nào để bắt, nên phải chủ động đo bằng cách so với exact search làm ground truth.

**Câu 4.** Bạn có 100 triệu vector 1536 chiều và máy chủ 128 GB RAM. Vấn đề gì sẽ xảy ra và hướng xử lý?
> *Gợi ý đáp án:* 100M × 1536 × 4 byte ≈ **600 GB** vector thô, cộng thêm khoảng 13 GB cho liên kết đồ thị — vượt xa 128 GB RAM. HNSW thuần sẽ phải đọc đĩa liên tục và tail latency sụp đổ. Hướng xử lý: quantization (`halfvec` hoặc binary + two-stage re-rank), DiskANN qua `pgvectorscale`, hoặc partition theo access pattern để mỗi partition có index nhỏ vừa RAM.

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn. Nó cố tình viết ngắn và cô đặc — ngược hẳn với bốn phần trên. Nếu một dòng nào đọc mà thấy mơ hồ, hãy quay lại đúng mục tương ứng đọc lại cho chậm.

### 5.1. Keywords bắt buộc nhớ

| Thuật ngữ tiếng Anh | Định nghĩa một dòng |
|---|---|
| **Index** | Cấu trúc dữ liệu phụ giúp tăng tốc tìm kiếm; như mục lục cuối sách. |
| **Linear search / sequential scan** | Quét từng dòng; trung bình `(n+1)/2`, tệ nhất `n`. |
| **Binary search** | Chia đôi liên tục trên dữ liệu đã sắp xếp; `O(log n)`. |
| **Total order** | Thứ tự toàn phần trên một trục — **điều kiện tiên quyết** của binary search. |
| **B-tree** | Index kinh điển của RDBMS cho dữ liệu một chiều, sắp thứ tự được. |
| **Write amplification** | Mỗi index thêm vào làm mọi thao tác ghi chậm đi. |
| **Vector / embedding** | Dãy số biểu diễn ý nghĩa của một vật; gần nhau = giống nghĩa. |
| **Nearest Neighbor (NN) / k-NN** | Tìm k vector gần nhất theo khoảng cách. |
| **ANN** | *Approximate Nearest Neighbor* — đổi recall lấy tốc độ. |
| **Recall** | % hàng xóm thật mà thuật toán tìm được; thước đo chất lượng ANN. |
| **Curse of dimensionality** | Ở chiều cao mọi điểm gần như cách đều nhau → cây phân hoạch sập về `O(n)`. |
| **IVFFlat** | *Inverted File Flat*; chia `lists` cụm bằng k-means; `probes` = số cụm dò. |
| **Centroid** | Tâm cụm; query so với centroid để chọn cụm cần tìm. |
| **k-means** | Thuật toán phân cụm dùng để chia list cho IVFFlat. |
| **HNSW** | *Hierarchical Navigable Small **World***; đồ thị nhiều tầng; ~`O(log n)`. |
| **Small world** | Mạng mà hai điểm bất kỳ chỉ cách nhau vài bước nhảy. |
| **Greedy search / local minimum** | Đi từng bước chọn nước tốt nhất tức thời; có thể mắc kẹt. |
| **`m` / `ef_construction`** | Tham số **lúc build** HNSW: số cạnh mỗi tầng / số ứng viên khi chèn. |
| **`ef_search`** | Tham số **lúc query** HNSW: núm xoay recall–tốc độ, chỉnh được lúc chạy. |
| **`lists` / `probes`** | Tham số IVFFlat, tương ứng lúc build / lúc query. |
| **Ops class** | `vector_cosine_ops`↔`<=>`, `vector_l2_ops`↔`<->`, `vector_ip_ops`↔`<#>`. |
| **Overfiltering** | ANN lấy ứng viên trước, `WHERE` lọc sau → trả về thiếu so với `LIMIT`. |
| **Iterative index scan** | Cơ chế chống overfiltering (pgvector 0.8+). |
| **DiskANN / pgvectorscale** | Index dựa trên SSD + nén; tiết kiệm RAM, hỗ trợ tới 16000 chiều. |
| **Quantization** | Giảm số bit mỗi con số (`halfvec`, binary) để cắt RAM. |
| **Two-stage retrieval** | Lấy thô top-N bằng index nén, rồi re-rank chính xác. |
| **`CREATE INDEX CONCURRENTLY`** | Build index mà không khoá thao tác ghi — bắt buộc trên production. |

### 5.2. Core concepts — nếu chỉ được nhớ mười điều

1. Index đổi một chút chi phí lưu trữ và ghi để lấy tốc độ tìm kiếm khổng lồ — **và chỉ đáng khi dữ liệu lớn**.
2. Linear `O(n)` → binary/B-tree `O(log n)`, nhưng **binary search bắt buộc dữ liệu phải sắp thứ tự được trên một trục**.
3. **Vector nhiều chiều không có trục để sắp** → B-tree/binary search vô dụng → buộc phải dùng ANN. *(Đây là câu quan trọng nhất cả bài.)*
4. ANN đổi **recall lấy tốc độ**; không index = exact = recall 100% nhưng `O(n·d)`.
5. **IVFFlat:** chia cụm bằng k-means, dò `probes` cụm. Build nhanh, nhẹ — nhưng cần dữ liệu sẵn, sót hàng xóm ở ranh giới cụm, và ngại dữ liệu động.
6. **HNSW:** đồ thị nhiều tầng, ~`O(log n)`. Recall cao, chịu dữ liệu động tốt — nhưng tốn RAM và build lâu.
7. `probes`/`ef_search` là núm xoay **lúc query**; `lists`/`m`/`ef_construction` là quyết định **lúc build**.
8. **Dữ liệu nhỏ → đừng đánh index.** Exact vừa nhanh vừa chính xác 100%.
9. **Ops class phải khớp toán tử**, nếu không index bị bỏ qua trong im lặng (seq scan).
10. Ở quy mô lớn: **RAM là bottleneck** → quantization/DiskANN; và **phải đo recall**, vì ANN xuống cấp mà không báo lỗi.

### 5.3. Mental model / analogy để nói cho trôi chảy

- **"Mục lục cuối sách"** → index là gì, trong đúng một câu. Dùng được cả với sếp không kỹ thuật.
- **"Sắp được thì binary search được"** → giải thích trong một câu vì sao vector cần loại index khác.
- **"Chia quận, chỉ ghé vài quận"** → IVFFlat, kèm điểm yếu ở ranh giới cụm (quán phở bên kia đường).
- **"Bay, rồi taxi, rồi đi bộ"** → HNSW đi từ tầng cao xuống tầng thấp.
- **"HNSW là skip list cho không gian không sắp xếp được"** → câu nói chứng tỏ hiểu sâu.
- **"ANN xuống cấp trong im lặng"** → vì sao phải giám sát recall.

### 5.4. Code cần thuộc lòng

**(a) Hai câu tạo index — interviewer rất hay bắt viết tại chỗ:**

```sql
CREATE INDEX ON items USING hnsw    (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

SET hnsw.ef_search = 100;    -- núm xoay recall lúc query (HNSW)
SET ivfflat.probes  = 32;    -- núm xoay recall lúc query (IVFFlat)
```

**(b) Binary search viết từ số 0:**

```python
def binary_search(a, x):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == x: return mid
        if a[mid] < x:  lo = mid + 1     # vứt nửa trái
        else:           hi = mid - 1     # vứt nửa phải
    return -1
```

**(c) Đo recall — đoạn ít ai nhớ nhưng gây ấn tượng mạnh:**

```sql
BEGIN;
SET LOCAL enable_indexscan = off;   -- ép exact search làm ground truth
SELECT id FROM items ORDER BY embedding <=> :q LIMIT 10;
COMMIT;
-- so độ trùng top-k với kết quả có index -> recall@10
```

**(d) Build an toàn trên production:**

```sql
CREATE INDEX CONCURRENTLY items_emb_idx
ON items USING hnsw (embedding vector_cosine_ops);
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Index tiết kiệm thời gian như thế nào?"**
> Linear search là `O(n)`, trung bình đọc `(n+1)/2` dòng. Binary search / B-tree là `O(log n)`. Với 1 triệu dòng: ~500.000 lần đọc so với ~20 bước. Cái giá phải trả là thêm dung lượng và ghi chậm hơn.

**2. [MẤU CHỐT] "Vì sao không dùng B-tree cho vector search?"**
> Ba lý do. (a) Vector nhiều chiều **không có thứ tự tuyến tính trên một trục duy nhất** để sắp xếp — sắp theo chiều nào cũng làm hai vector thực sự gần nhau bị đẩy xa nhau trong thứ tự, nên binary search mất điều kiện tiên quyết. (b) Câu hỏi khác bản chất: B-tree trả lời "bằng đúng X / trong khoảng [a,b]", còn vector search hỏi "gần nhất". (c) Curse of dimensionality làm cây phân hoạch không gian sập về `O(n)`. → Vì vậy ta chuyển sang ANN.

**3. "IVFFlat với HNSW khác nhau thế nào?"**
> IVFFlat: chia cụm bằng k-means, tham số `lists`/`probes`; build nhanh, nhẹ RAM — nhưng cần dữ liệu để train, sót hàng xóm ở ranh giới cụm, kém với dữ liệu động. HNSW: đồ thị nhiều tầng, ~`O(log n)`, recall cao, chịu dữ liệu động tốt — nhưng build lâu và tốn RAM. Mặc định hiện nay là HNSW.

**4. [TRADE-OFF] "Tăng `probes` hoặc `ef_search` thì được gì mất gì?"**
> Recall tăng, latency tăng. Đây là núm xoay recall–tốc độ **lúc query**, chỉnh được mà không phải build lại index. Tune theo SLA p95, và tune bằng cách **đo** chứ không đoán.

**5. [BẪY] "Vì sao kết quả 'sai' sau khi tôi đánh index?"**
> Không sai — ANN là *approximate* theo đúng thiết kế, bạn đã đổi recall lấy tốc độ ngay lúc gõ `CREATE INDEX`. Xử lý đúng theo thứ tự: đo recall trước → tăng `ef_search`/`probes` (rẻ) → cuối cùng mới tăng `m`/`ef_construction` và build lại (đắt).

**6. [BẪY] "Có nên đánh index cho bảng 5.000 vector không?"**
> Không. Brute force exact đã đủ nhanh và cho recall 100%. Index chỉ đáng khi dữ liệu đủ lớn. Không phải mọi thứ đều cần index.

**7. [SCALE] "100 triệu vector nhưng RAM hạn chế thì sao?"**
> Nhẩm trước: 100M × 1536 × 4 byte ≈ 600 GB vector thô → HNSW thuần không vừa RAM. Hướng: quantization (`halfvec`/binary) kèm two-stage re-rank; hoặc DiskANN qua `pgvectorscale`; partition theo access pattern; bật `iterative_scan` cho filter; và đo recall lẫn p95 để xác nhận.

**8. "Vì sao không nên tạo IVFFlat khi bảng còn rỗng?"**
> Vì nó cần dữ liệu để chạy k-means tìm centroid; bảng rỗng → phân cụm vô nghĩa → recall tệ, mà **không có lỗi nào báo**. HNSW không bị vấn đề này vì nó chèn dần từng node.

**9. "Làm sao bạn biết chất lượng search đang xấu đi trên production?"**
> Câu này ít được hỏi trực tiếp nhưng cực kỳ ghi điểm khi bạn chủ động nêu: recall suy giảm **không gây lỗi**, nên phải chủ động đo bằng cách so kết quả có index với exact search làm ground truth, lấy mẫu định kỳ trên read replica, và vẽ recall@10 theo thời gian lên dashboard.

**10. "Bạn build index trên bảng đang chạy production thế nào?"**
> `CREATE INDEX CONCURRENTLY` để không khoá thao tác ghi. Chậm hơn và có thể thất bại giữa chừng (để lại index hỏng cần dọn), nhưng cách thông thường sẽ chặn mọi `INSERT`/`UPDATE` suốt hàng giờ với HNSW ở quy mô lớn.

### 5.6. One-liner đắt giá — thả đúng lúc để ghi điểm

- *"An index trades a little storage and write time for a massive drop in search time — but it only pays off at scale."*
- *"Binary search needs a line to sort along; vectors don't have one, so we approximate instead of sorting."*
- *"HNSW is a skip list for a space you cannot sort."*
- *"ANN trades recall for speed — the real skill is measuring recall, because it degrades silently with no error to catch."*
- *"IVFFlat clusters and probes; HNSW climbs a multi-layer highway — same goal, different bet on memory versus build time."*
- *"Not everything needs an index — on small data, exact search is both faster to reason about and 100% correct."*
- *"At a hundred million vectors the bottleneck is RAM, not CPU — that's when quantization stops being optional."*

---

## 📌 Ghi chú cuối — làm gì tiếp theo

**1. Ba đính chính cần nhớ đúng khi đi phỏng vấn:**
- HNSW = *Small **World***, không phải "Small Words".
- Cạnh trong HNSW nối theo **thước đo khoảng cách bạn cấu hình** (cosine / L2 / inner product), không riêng gì Euclidean.
- Giới hạn **2000 chiều** cho index HNSW/IVFFlat — dùng `halfvec` hoặc DiskANN nếu vector cao chiều hơn.

**2. Thực hành ngay, đừng chỉ đọc.** Cách nhanh nhất để có Postgres kèm pgvector là dùng **Docker** (công cụ chạy phần mềm trong "hộp" đóng gói sẵn):

```bash
docker run --name pgvec -e POSTGRES_PASSWORD=pass -p 5432:5432 -d pgvector/pgvector:pg17
```

Rồi làm đúng chuỗi việc này, theo thứ tự:
1. Nạp khoảng 100.000 vector.
2. Đo latency khi **chưa** có index.
3. Tạo IVFFlat, đo lại. Tạo HNSW, đo lại.
4. Chạy `EXPLAIN ANALYZE` để **nhìn thấy** dòng `Index Scan using ...` bằng chính mắt mình.
5. Tắt index (`SET LOCAL enable_indexscan = off`) để lấy ground truth, rồi **tự tính recall@10** của mình.
6. Vặn `probes` và `ef_search` lên xuống, ghi lại cặp số (recall, latency) sau mỗi lần.

Bước 6 là bước quan trọng nhất. Khi bạn nhìn thấy đường cong đánh đổi bằng số liệu của chính mình, toàn bộ Phần 3 và Phần 4 sẽ ăn sâu theo cách mà đọc không bao giờ làm được.

**3. Kiểm chứng phiên bản trước khi ôn thi.** pgvector phát triển nhanh (v0.8.5 tính tới tháng 7/2026; HNSW là mặc định; iterative scan có từ 0.8.0; DiskANN qua `pgvectorscale`). Liếc CHANGELOG trên GitHub `pgvector/pgvector` — con số giới hạn chiều, tên tham số, hay tính năng mới có thể đã thay đổi.

**4. Nối lại mạch của cả series** (bốn mảnh, tất cả nằm trong một Postgres duy nhất):
> embed (tạo vector) → **index (bài này: làm search nhanh)** → store & query (pgvector) → keyword/full-text search → hybrid.

**5. Chủ đề nên học tiếp, theo thứ tự:**
- **Quantization sâu hơn** — scalar, product quantization (PQ), và binary; hiểu vì sao PQ vẫn là nền tảng của nhiều hệ thống lớn.
- **Two-stage retrieval và reranking model** — cách lấy lại chất lượng đã mất khi dùng index nén.
- **Sharding vector** — Citus, pgvectorscale, khi một máy không còn chứa nổi.
- **Benchmark có hệ thống** — bộ `ann-benchmarks` là chuẩn công khai của ngành để so recall/latency giữa các thuật toán. Đọc kết quả của nó cho bạn trực giác về con số thật mà không cần tự chạy.
- **Hybrid search** — kết hợp full-text search của Postgres (BM25) với vector search, rồi trộn điểm bằng Reciprocal Rank Fusion.
