# CAPSTONE — Vector Database với PostgreSQL: Tổng hợp toàn khóa
### Giáo trình SIÊU DỄ HIỂU — giải thích tận gốc mọi thuật ngữ | Basic → Staff

> **Bài giảng gốc:** *"Course wrap-up"* — Course 3, IBM Vector Database Fundamentals. Video gốc **không có kiến thức kỹ thuật mới**, nó chỉ tổng kết lại toàn khóa.
>
> **Vì vậy giáo trình này được viết theo hướng khác:** thay vì lặp lại tám bài đã học, nó **khâu cả tám bài thành một hệ thống duy nhất**. Bạn sẽ có một tấm bản đồ tổng, hiểu được các mảnh ghép nối với nhau ra sao, một dự án capstone để xây, và một bản cheatsheet ôn nhanh cả khóa.
>
> **Điểm quan trọng về cách viết:** giáo trình này **tự chứa**. Mọi thuật ngữ đều được giải thích lại từ đầu, ngay lần đầu xuất hiện. Nghĩa là bạn ôn được cả khóa chỉ bằng một file này — không cần mở tám file kia ra tra. Nếu muốn đào sâu chỗ nào thì mới quay lại bài tương ứng.
>
> **Toàn bộ nội dung ở đây là 🧩 [Ngoài bài gốc]**, vì bài gốc chỉ là lời chúc mừng và tóm tắt. Nên từ đây mình sẽ không đánh dấu ký hiệu đó nữa mà chỉ dùng ⚠️ **Chỗ khó**, 💡 **Mẹo thực chiến**, và ❗ **[Cập nhật 2026]**.
>
> **Tám bài của khóa, theo thứ tự học đề xuất:**
> 1. Tạo vector (embeddings) · 2. Đánh index cho nhanh · 3. Chọn nền tảng · 4. Thiết kế lưu trữ (pgvector khái niệm) · 5. Cài đặt pgvector · 6. Nạp dữ liệu quy mô lớn · 7. Truy vấn và ngưỡng lọc · 8. Full-text search và hybrid

---

## Phần 0 — 🗺️ Bản đồ toàn khóa: tám mảnh là MỘT hệ thống

### 0.1. Cả khóa học trả lời đúng một câu hỏi

Tám bài học nghe có vẻ là tám chủ đề rời rạc, nhưng thực ra chúng cùng trả lời **một câu hỏi duy nhất**:

> ***"Làm sao biến PostgreSQL — một cơ sở dữ liệu bình thường — thành một hệ thống tìm kiếm theo ý nghĩa, chạy được thật ở production?"***

Hai từ trong câu trên cần giải nghĩa ngay, vì chúng là mục tiêu của cả khóa.

**Tìm kiếm theo ý nghĩa** (*semantic search*) là kiểu tìm kiếm mà khi bạn gõ "đồ mặc cho trời lạnh", hệ thống trả về "áo phao giữ ấm" — dù hai câu này **không có một chữ nào chung**. Nó hiểu ý, không so chữ.

**Production** (dịch: *môi trường chạy thật*) là hệ thống đang phục vụ người dùng thật, tiền thật, và hỏng là có người bị ảnh hưởng. Khác với môi trường thử nghiệm ở chỗ: nó phải nhanh, phải đúng, phải chịu được tải, và phải sửa được khi hỏng lúc 3 giờ sáng.

### 0.2. Đường ống dữ liệu — tám trạm nối tiếp nhau

```
     [1] TẠO VECTOR          [2] ĐÁNH INDEX          [3] CHỌN NỀN TẢNG
   chữ/ảnh → dãy số        làm việc tìm nhanh      Postgres? MySQL? MariaDB?
     (dùng model AI)         (HNSW / IVFFlat)          → chọn pgvector
          │                        │                         │
          └───────────┬────────────┴────────────┬────────────┘
                      ▼                         ▼
              [5] CÀI pgvector          [4] THIẾT KẾ BẢNG
           (Docker / gói OS / cloud)   (cột vector + dữ liệu nghiệp vụ)
                      │
                      ▼
          [6] NẠP DỮ LIỆU ────► [7] TRUY VẤN ────► [8] KẾT HỢP TỪ KHÓA
        (COPY, nạp trước         (ORDER BY <=>       (full-text search
         index sau)               LIMIT + ngưỡng)     hợp nhất bằng RRF)
                                        │
                                        ▼
                        HỆ TÌM KIẾM NGỮ NGHĨA / RAG
```

**Đọc bản đồ thành lời:** Bạn **tạo** vector bằng một mô hình AI (1), chọn cách **đánh index** để tìm cho nhanh (2), chọn **nền tảng** để chứa (3), **thiết kế bảng** (4), **cài** phần mở rộng (5), **nạp** dữ liệu vào một cách hiệu quả (6), **truy vấn** đúng cách có kiểm soát chất lượng (7), và tùy chọn **ghép thêm tìm kiếm từ khóa** thành hybrid (8).

Kết quả cuối cùng là một hệ **retrieval** (dịch: *truy hồi* — tìm và lấy ra thông tin liên quan) hoàn chỉnh, và đó chính là nền móng của mọi ứng dụng **RAG** (viết tắt của *Retrieval-Augmented Generation*, dịch: *sinh văn bản có tra cứu*) — kỹ thuật cho mô hình ngôn ngữ đọc tài liệu thật trước khi trả lời, để giảm bịa đặt.

### 0.3. Ba câu chốt xuyên suốt cả khóa

Nếu quên hết mọi thứ khác, ba câu này vẫn phải nhớ:

**Một — embedding biến dữ liệu phức tạp thành thứ so sánh được.** Không chỉ chữ: ảnh, âm thanh, video đều biến thành vector được.

**Hai — cosine gần 1 nghĩa là giống nhau, nhưng toán tử `<=>` trả về *khoảng cách*, tức là `1 trừ đi độ giống`.** Câu này chứa cái bẫy phổ biến nhất của cả khóa, và mục 1.4 sẽ làm rõ.

**Ba — pgvector dùng cho dữ liệu không phải chữ nữa, khác hẳn `tsvector` của full-text search vốn chỉ xử lý từ khóa.** Hai kiểu dữ liệu này trùng chữ cái nhưng không liên quan gì tới nhau.

### 0.4. Cấu trúc của giáo trình này

- 🟢 **Basic** — cả hệ thống giải thích cho người mới, một phép ví von dựng đầy đủ, và **một ví dụ chạy tay đi hết cả tám trạm bằng số thật**.
- 🟡 **Intermediate** — đi dọc đường ống, mỗi trạm một quyết định cốt lõi kèm lý do, và ba "sợi chỉ đỏ" xuyên qua nhiều trạm.
- 🔴 **Advanced** — năm chủ đề không thuộc riêng bài nào: đánh đổi giữa nhanh và đúng, vì sao không tìm kiếm vector bằng cách thông thường được, luận điểm "một Postgres cho tất cả", chi phí ẩn của việc đổi mô hình, và cách hợp nhất kết quả hybrid.
- 🟣 **Staff** — một câu hỏi system design gộp mọi quyết định của khóa, bản đặc tả dự án capstone để đưa vào hồ sơ, và cách trình bày với người không rành kỹ thuật.
- 🎯 **Master cheatsheet** — ôn nhanh cả khóa trong một trang.

---

## Phần 1 — 🟢 BASIC: Cả hệ thống trong một đoạn, rồi đi từng bước

### 1.1. Toàn bộ khóa học giải thích cho người chưa biết gì

Đọc chậm đoạn này, rồi mục 1.2 sẽ mổ từng câu ra.

> Máy tính không hiểu chữ nghĩa hay hình ảnh. Vì vậy ta dùng một **mô hình AI** để biến chúng thành **embedding** — những dãy số nắm bắt được ý nghĩa, sao cho hai nội dung giống nhau về nghĩa sẽ cho ra hai dãy số nằm gần nhau. Ta lưu những dãy số đó vào **PostgreSQL** nhờ phần mở rộng **pgvector**, và dựng **index** để tìm được "hàng xóm gần nhất" thật nhanh dù có hàng triệu dãy số. Khi người dùng đặt câu hỏi, ta biến câu hỏi đó thành vector rồi tìm những bản ghi có vector gần nhất — đó chính là **tìm kiếm theo ý nghĩa**. Ghép thêm **tìm kiếm theo từ khóa** vào nữa thì ra kết quả tốt nhất.

Nếu đoạn trên có từ nào bạn chưa hiểu, hoàn toàn bình thường — mục 1.3 sẽ giải thích từng từ một.

### 1.2. Analogy: người thủ thư và tấm bản đồ tọa độ

Trước khi vào định nghĩa, hãy dựng một phép ví von cho thật đầy đủ. Nó sẽ theo bạn suốt bài và cả trong phòng phỏng vấn.

Hình dung một thư viện khổng lồ với 20 triệu cuốn sách.

**Cách làm cũ — tra theo từ khóa.** Thư viện có một cuốn mục lục ghi "sách nào chứa từ nào". Bạn hỏi "sách về *ô tô*", thủ thư tra mục lục và đưa bạn mọi cuốn có chữ "ô tô". Cách này chính xác và nhanh — nhưng nếu có một cuốn tuyệt hay tên là *"Lịch sử ngành xe hơi Việt Nam"* thì bạn **không bao giờ tìm ra**, vì nó không chứa chữ "ô tô".

**Cách làm mới — sắp sách theo tọa độ ý nghĩa.** Bây giờ tưởng tượng thư viện gắn cho mỗi cuốn sách một **tọa độ**, giống như tọa độ trên bản đồ. Tọa độ này không phản ánh vị trí trên kệ, mà phản ánh **nội dung**: mọi cuốn về phương tiện giao thông nằm quanh một vùng, mọi cuốn về nấu ăn nằm ở vùng khác.

Giờ khi bạn hỏi "sách về ô tô", thủ thư không tra chữ nữa. Ông ta tính ra tọa độ của **câu hỏi** rồi đi tìm những cuốn sách có tọa độ **gần nhất**. Cuốn *"Lịch sử ngành xe hơi"* nằm ngay cạnh đó, nên bạn tìm ra ngay — dù không chung một chữ nào.

**Ánh xạ sang kỹ thuật:**

| Trong thư viện | Trong hệ thống thật |
|---|---|
| Tọa độ của một cuốn sách | **Embedding** — vector biểu diễn nội dung |
| Người gắn tọa độ cho sách | **Mô hình AI** (trạm 1) |
| Cách sắp xếp để tra tọa độ cho nhanh | **Index** HNSW (trạm 2) |
| Cái kệ chứa cả sách lẫn tọa độ | **PostgreSQL + pgvector** (trạm 3, 4, 5) |
| Việc gắn tọa độ cho 20 triệu cuốn ban đầu | **Nạp dữ liệu** (trạm 6) |
| Thủ thư đi tìm cuốn gần nhất | **Truy vấn** (trạm 7) |
| Dùng cả mục lục từ khóa lẫn tọa độ | **Hybrid search** (trạm 8) |

**Và đây là ba chỗ phép ví von khớp chính xác nhất** — chú ý cả ba, vì mỗi chỗ tương ứng với một bài học lớn:

*Thứ nhất:* nếu hai thư viện dùng **hai hệ tọa độ khác nhau**, tọa độ của thư viện này đem sang thư viện kia là vô nghĩa. Đó chính là quy tắc "phải dùng cùng một mô hình".

*Thứ hai:* việc gắn tọa độ cho 20 triệu cuốn sách **tốn công hơn nhiều** so với việc tra một cuốn. Đó là lý do chi phí thật của hệ thống nằm ở khâu tạo embedding, không phải khâu tìm kiếm.

*Thứ ba:* nếu đổi sang hệ tọa độ mới tốt hơn, bạn phải **gắn lại tọa độ cho cả 20 triệu cuốn**. Đó chính là re-embedding — quyết định nặng nhất trong toàn bộ kiến trúc (mục 3.4).

### 1.3. Từ điển thuật ngữ nền móng

Đây là những từ sẽ xuất hiện liên tục. Đọc chậm, mỗi từ một lần.

**Vector** là một **dãy số có thứ tự**, ví dụ `[0.12, -0.85, 0.33]`. Số lượng phần tử trong dãy gọi là **số chiều (dimension)**. Các mô hình thực tế tạo ra vector 384, 768, 1024, 1536 hoặc 3072 chiều.

**Embedding** (nghĩa đen: *sự nhúng vào*) là kết quả của việc biến một nội dung thành vector, sao cho **nội dung gần nghĩa cho ra vector gần nhau**. Đây chính là "tọa độ" trong phép ví von thư viện.

**Model** (dịch: *mô hình*) là chương trình AI làm việc biến nội dung thành embedding. Nó quyết định "hệ tọa độ" trông như thế nào.

**Distance** (dịch: *khoảng cách*) là con số đo hai vector cách nhau bao xa. **Càng nhỏ càng giống.**

**Similarity** (dịch: *độ tương đồng*) là con số đo hai vector giống nhau tới đâu. **Càng lớn càng giống.** Hai khái niệm này ngược nhau, và mục 1.4 sẽ chỉ ra vì sao lẫn lộn chúng là lỗi phổ biến nhất.

**k-NN** (viết tắt của *k-Nearest Neighbors*, dịch: *k láng giềng gần nhất*) là bài toán: cho một vector, tìm k vector gần nó nhất.

**ANN** (viết tắt của *Approximate Nearest Neighbor*, dịch: *láng giềng gần nhất gần đúng*) là cách giải bài toán trên **nhanh nhưng chấp nhận thỉnh thoảng bỏ sót**. Đây là đánh đổi trung tâm của toàn bộ lĩnh vực, và mục 3.1 dành riêng cho nó.

**Index** (dịch: *chỉ mục*) là cấu trúc dữ liệu phụ mà database dựng thêm để tìm kiếm nhanh hơn — giống mục lục ở cuối một cuốn sách.

**Extension** (dịch: *phần mở rộng*) là gói tính năng cài thêm vào PostgreSQL để nó làm được việc mà nó vốn không làm được. **pgvector** là extension thêm khả năng lưu và tìm vector.

**Full-text search** (viết tắt **FTS**, dịch: *tìm kiếm toàn văn*) là tìm kiếm theo **từ khóa** — cách làm cũ trong phép ví von thư viện. PostgreSQL có sẵn khả năng này qua kiểu `tsvector`.

**Hybrid search** (dịch: *tìm kiếm lai*) là chạy cả hai cách — theo từ khóa và theo ý nghĩa — rồi hợp nhất kết quả.

⚠️ **Chỗ khó cần nhớ ngay: `tsvector` KHÔNG phải `vector`.**

| | `tsvector` | `vector` |
|---|---|---|
| Có sẵn trong Postgres? | **Có** | Không, cần cài pgvector |
| Lưu gì | Danh sách **từ đã chuẩn hóa** | Dãy **số thực** (embedding) |
| Dùng cho | Tìm theo **từ khóa** | Tìm theo **ý nghĩa** |
| Ví dụ nội dung | `'brown':3 'fox':4 'run':6` | `[0.12, -0.85, 0.33, ...]` |

Hai thứ này chỉ trùng bốn chữ cái. Nhầm chúng là một trong những câu hỏi bẫy hay gặp nhất.

### 1.4. Ví dụ chạy tay — đi hết cả tám trạm bằng số thật

Đây là phần quan trọng nhất của Phần 1. Ta sẽ chạy toàn bộ đường ống bằng tay, với dữ liệu tí hon để tính được bằng giấy bút.

**Trạm 1 — Tạo vector.**

Ta có ba tài liệu. Mô hình AI biến chúng thành vector 2 chiều (thực tế là hàng trăm chiều, nhưng phép toán y hệt):

| Tài liệu | Nội dung | Vector do mô hình sinh ra |
|---|---|---|
| D1 | "áo khoác mùa đông" | `[0.9, 0.1]` |
| D2 | "áo phao giữ ấm" | `[0.8, 0.2]` |
| D3 | "máy xay sinh tố" | `[0.1, 0.9]` |

Chú ý D1 và D2 có số liệu gần nhau (cùng nói về đồ mặc ấm), còn D3 khác hẳn. Mô hình đã đặt chúng vào đúng vùng trên "bản đồ ý nghĩa".

**Trạm 4 — Thiết kế bảng.** Ta cần một bảng có cột vector 2 chiều, đặt cạnh cột nội dung:

```
docs(id, content, embedding vector(2))
```

**Trạm 6 — Nạp dữ liệu.** Ba dòng vào bảng. (Ở quy mô thật thì đây là chỗ dùng lệnh `COPY`, xem mục 2.6.)

**Trạm 7 — Truy vấn.** Người dùng gõ: **"đồ mặc cho trời lạnh"**. Ta đưa câu này qua **cùng mô hình** ở trạm 1, được vector truy vấn:

```
q = [0.85, 0.15]
```

Bây giờ tính **cosine distance** từ `q` tới từng tài liệu. Cosine đo **góc** giữa hai vector và bỏ qua độ dài của chúng.

Công thức: `cosine similarity = (tích vô hướng) / (độ dài A × độ dài B)`, rồi `cosine distance = 1 - similarity`.

**Tính cho D1 = [0.9, 0.1]:**
```
Tích vô hướng:  0.85×0.9 + 0.15×0.1 = 0.765 + 0.015 = 0.780
Độ dài q:       √(0.85² + 0.15²) = √(0.7225 + 0.0225) = √0.745 ≈ 0.863
Độ dài D1:      √(0.9² + 0.1²)   = √(0.81 + 0.01)     = √0.82  ≈ 0.906
Similarity:     0.780 / (0.863 × 0.906) = 0.780 / 0.782 ≈ 0.997
Distance:       1 - 0.997 = 0.003        ← rất gần
```

**Tính cho D2 = [0.8, 0.2]:**
```
Tích vô hướng:  0.85×0.8 + 0.15×0.2 = 0.680 + 0.030 = 0.710
Độ dài D2:      √(0.64 + 0.04) = √0.68 ≈ 0.825
Similarity:     0.710 / (0.863 × 0.825) = 0.710 / 0.712 ≈ 0.997
Distance:       1 - 0.997 = 0.003        ← cũng rất gần
```

**Tính cho D3 = [0.1, 0.9]:**
```
Tích vô hướng:  0.85×0.1 + 0.15×0.9 = 0.085 + 0.135 = 0.220
Độ dài D3:      √(0.01 + 0.81) = √0.82 ≈ 0.906
Similarity:     0.220 / (0.863 × 0.906) = 0.220 / 0.782 ≈ 0.281
Distance:       1 - 0.281 = 0.719        ← xa hơn khoảng hai trăm lần
```

**Kết quả xếp hạng:** D1 (0.003) và D2 (0.003) đứng đầu, D3 (0.719) đứng cuối.

Bây giờ hãy dừng lại và nhìn kỹ điều vừa xảy ra — đây là **phép màu của cả khóa học gói trong một dòng**:

Câu người dùng gõ là **"đồ mặc cho trời lạnh"**. Tài liệu D2 là **"áo phao giữ ấm"**. Hai câu này **không chung một chữ nào** — không có "đồ", không có "lạnh", không có "áo phao". Tìm kiếm theo từ khóa sẽ trả về con số không tròn trĩnh. Nhưng vì mô hình đã đặt hai nội dung này gần nhau trên bản đồ ý nghĩa, mấy phép nhân và cộng đơn giản ở trên tìm ra nó ngay.

**Trạm 7 (tiếp) — Ngưỡng lọc.** Nếu ta đặt ngưỡng ở **0.3** thì D3 bị loại, và hệ thống trả về đúng 2 kết quả liên quan thay vì 3 kết quả trong đó có một cái nói về máy xay sinh tố.

**Trạm 2 — Đánh index.** Với 3 dòng thì không cần. Nhưng nếu là 20 triệu dòng, việc tính khoảng cách cho **từng dòng một** như ta vừa làm sẽ mất hàng chục giây mỗi truy vấn. Index HNSW giải quyết đúng chỗ đó — nó cho phép nhảy thẳng tới vùng lân cận thay vì duyệt hết.

**Trạm 8 — Hybrid.** Nếu người dùng gõ một mã sản phẩm như `SKU-88XZ`, tìm theo ý nghĩa sẽ trượt (vì mã sản phẩm gần như không có ý nghĩa ngữ nghĩa nào). Lúc đó nhánh tìm theo từ khóa cứu bạn. Chạy cả hai rồi hợp nhất — đó là hybrid, và mục 3.5 sẽ tính tay phép hợp nhất này.

### 1.5. Code end-to-end nhỏ nhất — cả tám trạm trong 15 dòng

```sql
-- ═══ TRẠM 5: CÀI EXTENSION ═══
-- Chỉ chạy một lần cho mỗi database. IF NOT EXISTS = có rồi thì bỏ qua.
CREATE EXTENSION IF NOT EXISTS vector;


-- ═══ TRẠM 4: THIẾT KẾ BẢNG ═══
CREATE TABLE docs (
    id        bigserial PRIMARY KEY,  -- số tự tăng làm khóa chính
    content   text,                   -- nội dung gốc, để HIỂN THỊ cho người dùng
    embedding vector(384)             -- vector 384 chiều
                                      -- ⚠️ số 384 phải KHỚP số chiều của mô hình
);
-- Chú ý: cột embedding nằm CÙNG BẢNG với content.
-- Đây chính là siêu năng lực của pgvector — xem mục 3.3.


-- ═══ TRẠM 6: NẠP DỮ LIỆU ═══
-- Ở đây chèn tay cho dễ hiểu. Production dùng COPY, và nạp TRƯỚC index (mục 2.6).
INSERT INTO docs (content, embedding)
VALUES ('áo khoác mùa đông', '[0.9, 0.1, ...]'),
       ('áo phao giữ ấm',    '[0.8, 0.2, ...]');


-- ═══ TRẠM 2: ĐÁNH INDEX (sau khi nạp xong) ═══
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
--                          ^^^^            ^^^^^^^^^^^^^^^^^
--                    thuật toán HNSW   "ops class": báo cho index biết
--                                       sẽ dùng với thước đo cosine.
--                    ⚠️ ops class PHẢI khớp toán tử dùng lúc truy vấn.


-- ═══ TRẠM 7: TRUY VẤN ═══
SELECT content,
       1 - (embedding <=> :q) AS similarity  -- đổi distance → similarity để hiển thị
FROM docs
WHERE embedding <=> :q < 0.3                 -- NGƯỠNG: lọc chất lượng
ORDER BY embedding <=> :q                    -- sắp gần nhất lên đầu (tăng dần)
LIMIT 5;                                     -- LIMIT: giới hạn số lượng
```

Về ký hiệu `:q` — đó là **tham số truy vấn**, chỗ mà ứng dụng của bạn điền vector thật vào lúc chạy. Và điều quan trọng nhất: vector đó phải do **cùng một mô hình** đã dùng để tạo embedding cho bảng `docs` sinh ra (trạm 1).

Đoạn code trên chính là cả đường ống thu nhỏ. Tám trạm, mười lăm dòng.

### 1.6. Hai điều nên khắc cốt ghi tâm ngay từ Basic

**Một: pgvector không tự sinh embedding.** Nó chỉ **lưu** và **tìm**. Việc biến chữ thành vector là của mô hình AI, chạy ở tầng ứng dụng. Rất nhiều người mới tưởng cài pgvector xong là database tự biết biến chữ thành vector — không phải.

**Hai: cột vector là nguyên liệu tính toán, không phải thông tin hiển thị.** Đừng bao giờ `SELECT embedding` ra để xem — bạn sẽ nhận về mấy trăm con số không nói lên điều gì, vì từng chiều riêng lẻ không mang ý nghĩa mà con người đọc được. Ý nghĩa chỉ tồn tại trong **quan hệ giữa các vector**, tức là trong khoảng cách.

### ✅ Self-check Phần 1

**1. Kể tên bốn bước từ "một câu chữ" tới "kết quả tìm kiếm theo ý nghĩa".**
> *Gợi ý đáp án:* (a) Mô hình AI biến câu chữ thành embedding; (b) lưu embedding vào cột `vector` trong Postgres; (c) dựng index HNSW để tìm nhanh; (d) khi có câu hỏi thì embed câu hỏi bằng **cùng mô hình** rồi tìm các vector gần nhất bằng `ORDER BY ... LIMIT k`.

**2. Vì sao câu truy vấn bắt buộc phải dùng cùng mô hình với dữ liệu trong bảng?**
> *Gợi ý đáp án:* Mỗi mô hình dựng một hệ tọa độ riêng. Vector từ hai mô hình khác nhau nằm trên hai bản đồ khác nhau nên khoảng cách giữa chúng vô nghĩa. Nguy hiểm nhất là khi hai mô hình tình cờ cùng số chiều — database không báo lỗi, kết quả cứ sai âm thầm.

**3. Trong đoạn code ở mục 1.5, dòng nào là "index" và dòng nào là "ngưỡng"?**
> *Gợi ý đáp án:* Index là `CREATE INDEX ... USING hnsw (embedding vector_cosine_ops)`. Ngưỡng là `WHERE embedding <=> :q < 0.3`. Lưu ý phân biệt ngưỡng (chất lượng) với `LIMIT 5` (số lượng).

**4. `tsvector` và `vector` khác nhau ra sao?**
> *Gợi ý đáp án:* `tsvector` có sẵn trong Postgres, lưu danh sách từ đã chuẩn hóa, dùng cho tìm kiếm **từ khóa**. `vector` đến từ extension pgvector, lưu dãy số thực (embedding), dùng cho tìm kiếm **ý nghĩa**. Chỉ trùng chữ cái.

---

## Phần 2 — 🟡 INTERMEDIATE: Đi dọc đường ống, mỗi trạm một quyết định

> Mỗi trạm trong đường ống tương ứng với **đúng một câu hỏi phải trả lời**. Nắm được chuỗi tám câu hỏi này là nắm được cả khóa. Ở đây ta đi từng trạm, giải thích **vì sao** câu trả lời lại là như vậy — chứ không chỉ liệt kê kết luận.

### 2.1. Trạm 1 — TẠO VECTOR: chọn mô hình nào?

**Câu hỏi cốt lõi:** dùng mô hình nào để biến nội dung thành embedding?

Trước hết, một phân biệt quan trọng về mặt khái niệm mà người phỏng vấn hay hỏi.

**Static embedding** (dịch: *embedding tĩnh*) là thế hệ cũ, ví dụ Word2Vec. Mỗi từ có **đúng một vector cố định**, bất kể nó xuất hiện ở đâu. Nghĩa là từ "đá" trong "hòn đá" và trong "đá bóng" nhận cùng một vector — dù hai nghĩa hoàn toàn khác nhau.

**Contextual embedding** (dịch: *embedding theo ngữ cảnh*) là thế hệ hiện tại, dựa trên kiến trúc **transformer**. Vector của một từ **thay đổi theo câu chứa nó**, nhờ một cơ chế gọi là **attention** (dịch: *sự chú ý*) — mô hình "nhìn" các từ xung quanh để quyết định nghĩa. Nhờ vậy "đá" trong hai câu trên cho hai vector khác nhau.

Nêu được phân biệt này trong phỏng vấn là điểm cộng, vì nó cho thấy bạn hiểu **vì sao** mô hình hiện đại tốt hơn, chứ không chỉ biết tên chúng.

❗ **[Cập nhật 2026]** Danh sách mô hình khuyến nghị đã đổi khá nhiều so với thời điểm khóa học được quay. Tình hình giữa 2026:

| Nhu cầu | Lựa chọn hiện tại | Ghi chú |
|---|---|---|
| **Tự chạy, đa ngôn ngữ** | **Qwen3-Embedding** (bản 8B, 4B, hoặc 0.6B) | Giấy phép Apache 2.0, dùng thương mại thoải mái; đang là đường đi mặc định cho đội có GPU |
| **Đa ngôn ngữ, mã nguồn mở tối đa** | **Llama-Embed-Nemotron-8B** của NVIDIA | Dẫn đầu bảng xếp hạng đa ngữ, trọng số mở hoàn toàn |
| **Dùng API, chất lượng cao** | **Gemini Embedding** của Google, hoặc **Voyage** | Không phải tự vận hành, nhưng dữ liệu ra ngoài |
| **Đa phương thức (chữ + ảnh)** | **Cohere Embed v4** hoặc **Jina v4/v5** | Nhúng cả PDF, slide, ảnh mà không cần pipeline OCR riêng |
| **Rẻ nhất, chấp nhận chất lượng vừa** | **OpenAI text-embedding-3-small** | Vẫn là lựa chọn giá tốt nhất cho đội eo hẹp ngân sách |

**MTEB** (viết tắt của *Massive Text Embedding Benchmark*) là bộ tiêu chuẩn để so sánh các mô hình embedding, và **MMTEB** là phiên bản đa ngôn ngữ của nó, phủ hơn 250 ngôn ngữ.

⚠️ **Ba cảnh báo khi đọc bảng xếp hạng MTEB.** Thứ nhất, điểm số là **do chính nhà phát triển tự báo cáo**. Thứ hai, MTEB đã có phiên bản 2 và **điểm giữa hai phiên bản không so sánh trực tiếp được**. Thứ ba — và quan trọng nhất — **điểm chung chung không phải lúc nào cũng đúng với dữ liệu của bạn**; hãy tự đo trên chính dữ liệu của mình.

💡 **Mẹo cho người làm dữ liệu tiếng Việt:** đã có một bộ tiêu chuẩn riêng tên là **VN-MTEB** dành cho tiếng Việt. Trước khi chọn mô hình cho một sản phẩm tiếng Việt, đây là chỗ đáng tra đầu tiên — vì thứ hạng trên bảng tiếng Anh không nói lên nhiều điều về tiếng Việt.

**Về số chiều:** với đa số ứng dụng, khoảng **768 tới 1024 chiều** là điểm cân bằng tốt nhất giữa độ chính xác và chi phí. Có một kỹ thuật tên là **Matryoshka** (đặt theo tên búp bê gỗ Nga lồng nhau) cho phép cắt bớt số chiều — ví dụ xuống 256 — mà chỉ mất khoảng 2–3% độ chính xác. Rất đáng cân nhắc khi RAM là ràng buộc.

**Chốt trạm 1:** chọn mô hình theo ngôn ngữ và ràng buộc dữ liệu, không theo thứ hạng chung chung. Và ghi nhớ điều quan trọng nhất: **đổi mô hình nghĩa là phải làm lại toàn bộ vector** (mục 3.4).

### 2.2. Trạm 2 — ĐÁNH INDEX: dùng loại nào?

**Câu hỏi cốt lõi:** làm sao tìm nhanh trong hàng triệu vector?

**Cách chính xác tuyệt đối** là so vector truy vấn với **từng vector một** trong kho — gọi là **exact search** (*tìm chính xác*). Nó luôn đúng 100%, nhưng thời gian tỉ lệ thuận với số dòng.

Ký hiệu để mô tả điều đó là **Big-O**: `O(n)` nghĩa là thời gian tăng tỉ lệ thuận với lượng dữ liệu `n` — gấp đôi dữ liệu thì gấp đôi thời gian.

Với hàng triệu vector, `O(n)` là không chấp nhận được. Nên ta dùng **ANN** — chấp nhận thỉnh thoảng bỏ sót để đổi lấy tốc độ nhanh hơn hàng nghìn lần.

Ba lựa chọn chính:

**HNSW** (viết tắt của *Hierarchical Navigable Small World*) dựng một **đồ thị nhiều tầng**. Tầng trên cùng thưa, cho phép nhảy những bước dài; các tầng dưới dày dần, cho phép dò kỹ. Cứ hình dung như đi từ đường cao tốc xuống quốc lộ rồi vào ngõ nhỏ. Độ phức tạp khoảng `O(log n)` — dữ liệu gấp mười lần thì thời gian chỉ tăng một chút.
> *Được:* độ chính xác cao, truy vấn nhanh. *Mất:* **ngốn RAM** vì đồ thị phải nằm trong bộ nhớ, và xây index chậm.

**IVFFlat** (viết tắt của *Inverted File with Flat compression*) **chia không gian thành các cụm**, mỗi cụm có một điểm trung tâm gọi là **centroid**. Khi truy vấn, nó chỉ tìm trong vài cụm gần nhất.
> *Được:* xây nhanh, nhẹ RAM. *Mất:* độ chính xác thấp hơn HNSW, và **cần có sẵn dữ liệu để chia cụm** — nên không hợp với bảng còn rỗng hoặc dữ liệu thay đổi liên tục.

**DiskANN** thiết kế để chạy chủ yếu trên **SSD** (ổ cứng thể rắn) thay vì RAM. Trong Postgres, nó đến qua extension `pgvectorscale`.
> *Được:* rẻ hơn nhiều ở quy mô hàng trăm triệu vector. *Mất:* chậm hơn HNSW khi dữ liệu còn vừa RAM.

**Chốt trạm 2:**

| Quy mô | Lựa chọn |
|---|---|
| Dưới ~50 nghìn vector | **Không cần index** — exact search vẫn nhanh và cho kết quả chính xác 100% |
| Vừa (tới vài chục triệu) | **HNSW** — mặc định đúng trong đa số trường hợp |
| RAM eo hẹp, dữ liệu tĩnh | IVFFlat hoặc DiskANN |

💡 Điểm đáng nhớ: **đừng vội đánh index khi chưa cần**. Với vài nghìn dòng, exact search vừa nhanh vừa chính xác tuyệt đối. Tối ưu sớm là lãng phí.

### 2.3. Trạm 3 — CHỌN NỀN TẢNG: cơ sở dữ liệu quan hệ hay hệ chuyên dụng?

**Câu hỏi cốt lõi:** đặt vector ở đâu?

Hai hướng: cắm vector vào **cơ sở dữ liệu quan hệ** đang chạy (PostgreSQL với pgvector, MariaDB với kiểu `VECTOR`, MySQL với HeatWave), hoặc dựng một **cơ sở dữ liệu vector chuyên dụng** (Pinecone, Weaviate, Qdrant, Milvus).

**Kim chỉ nam để quyết định, gói trong một câu hỏi:** ***"Tìm kiếm vector là một tính năng, hay chính là sản phẩm?"***

| Chọn cơ sở dữ liệu quan hệ khi | Chọn hệ chuyên dụng khi |
|---|---|
| Vector là **một tính năng** trong app đã chạy | Vector search **chính là sản phẩm** |
| Dưới khoảng 10–50 triệu vector | Từ hàng trăm triệu tới hàng tỷ |
| Cần lọc và nối với dữ liệu nghiệp vụ | Cần phân tán nhiều vùng địa lý |
| Đội nhỏ, muốn ít hệ thống | Có đội chuyên trách hạ tầng tìm kiếm |

Con số "10–50 triệu" nghe nhỏ nhưng thực ra **bao trùm phần lớn** hệ thống tìm kiếm và RAG trong doanh nghiệp.

❗ **[Cập nhật 2026]** Có một xu hướng đáng nói: sau làn sóng ai cũng dựng vector database riêng giai đoạn 2023–2024, **khá nhiều đội đã quay ngược về PostgreSQL**. Lý do không phải pgvector nhanh hơn — nó không nhanh hơn. Lý do là **vận hành thêm một database là cái giá thật mà người ta đã đánh giá thấp**.

**Chốt trạm 3:** với bối cảnh của khóa này — đã có Postgres, vector là tính năng, quy mô vừa — câu trả lời là **pgvector**.

### 2.4. Trạm 4 — THIẾT KẾ BẢNG: schema như thế nào?

**Câu hỏi cốt lõi:** đặt cột vector ở đâu và kèm những gì?

**Schema** (đọc "ski-ma", dịch: *lược đồ*) là bản thiết kế của database: có bảng nào, mỗi bảng có cột gì, kiểu dữ liệu ra sao.

Nguyên tắc: **đặt cột `vector(n)` ngay trong bảng nghiệp vụ, cạnh các cột bình thường** — không tách sang bảng riêng, không tách sang hệ thống riêng.

```sql
CREATE TABLE products (
    id         bigserial PRIMARY KEY,
    tenant_id  bigint,          -- khách hàng doanh nghiệp nào (dùng để lọc và phân mảnh)
    name       text,
    price      numeric,         -- dùng numeric cho tiền, KHÔNG dùng float
    in_stock   boolean,         -- dữ liệu nghiệp vụ...
    lang       text,            -- ...nằm ngay cạnh...
    embedding  vector(1024),    -- ...vector. Cùng một bảng.
    model_name text             -- 💡 ghi rõ mô hình nào sinh ra vector này
);
```

Hai chi tiết đáng chú ý trong đoạn trên.

**`model_name`** — cột này không có trong bài gốc nhưng đáng thêm vào. Nó ghi lại mô hình và phiên bản đã sinh ra vector, để sau này bạn biết chính xác hàng nào cần làm lại khi đổi mô hình. Nghe thừa lúc mới làm, nhưng nó cứu bạn ở mục 3.4.

**Số `1024` phải khớp đúng số chiều mô hình sinh ra.** Khai sai là `INSERT` báo lỗi ngay.

**Vì sao đặt cùng bảng lại quan trọng tới vậy?** Vì nó cho phép **hybrid query** — một câu SQL vừa tìm theo ý nghĩa vừa lọc theo nghiệp vụ:

```sql
SELECT id, name FROM products
WHERE in_stock = true AND price < 50      -- lọc nghiệp vụ
ORDER BY embedding <=> :q                  -- xếp theo độ giống
LIMIT 10;
```

Mục 3.3 sẽ phân tích vì sao câu lệnh tưởng như bình thường này lại là siêu năng lực lớn nhất của cả kiến trúc.

### 2.5. Trạm 5 — CÀI ĐẶT: cài kiểu gì?

**Câu hỏi cốt lõi:** làm sao có pgvector trên máy chủ?

Ba cách, theo thứ tự ưu tiên:

**Dịch vụ vận hành sẵn (managed)** — Supabase, Neon, AWS RDS, Google Cloud SQL đều có sẵn pgvector. Bạn chỉ cần bật lên. Đây là cách ít công nhất.

**Gói cài đặt của hệ điều hành hoặc Docker** — dùng ảnh `pgvector/pgvector` hoặc cài qua `apt`/`yum`. Đây là cách chuẩn khi tự vận hành.

**Biên dịch từ mã nguồn** — chỉ khi thật sự cần một phiên bản chưa được đóng gói. **Đừng làm việc này trên máy chủ production.**

⚠️ **Chỗ khó — một phân biệt mà rất nhiều người vấp.** Có **hai bước riêng biệt**, và làm bước một không có nghĩa là xong bước hai:

1. **Cài file lên máy chủ** (qua Docker, apt, hoặc managed) — đưa mã của extension vào hệ thống.
2. **Kích hoạt trong từng database** bằng `CREATE EXTENSION vector;` — bật nó lên cho database cụ thể đó.

Triệu chứng khi quên bước 2: lỗi `type "vector" does not exist` dù bạn chắc chắn đã cài. Và lưu ý extension được bật **theo từng database**, nên nếu có nhiều database thì phải chạy lệnh đó ở mỗi cái.

💡 **Mẹo thực chiến:** hãy **ghim (pin) phiên bản** — dùng `pgvector/pgvector:pg17-0.8.0` chứ đừng dùng thẻ `latest`. Và hãy đưa `CREATE EXTENSION` vào **migration** (một tập lệnh thay đổi cấu trúc database có phiên bản, chạy tự động khi triển khai) thay vì gõ tay, để mọi môi trường luôn giống nhau. Nếu có **replica** (bản sao chỉ đọc của database), phiên bản extension trên replica phải khớp với máy chính.

### 2.6. Trạm 6 — NẠP DỮ LIỆU: chèn kiểu gì cho nhanh?

**Câu hỏi cốt lõi:** làm sao đưa 20 triệu vector vào bảng mà không mất cả tuần?

**Chốt thứ nhất: dùng `COPY`, không dùng `INSERT` từng dòng.**

`INSERT` từng dòng nghĩa là mỗi dòng phải đi qua toàn bộ chi phí giao tiếp giữa ứng dụng và database: gửi câu lệnh, phân tích cú pháp, lập kế hoạch, ghi nhật ký giao dịch. Nhân chi phí đó với 20 triệu lần là ra một con số khủng khiếp.

**`COPY`** là lệnh nạp hàng loạt của Postgres: nó nhận cả một luồng dữ liệu và ghi thẳng vào bảng, bỏ qua gần hết chi phí trên. Thường nhanh hơn `INSERT` từng dòng **hàng chục lần**.

**Chốt thứ hai — và đây là chỗ hay bị làm ngược: NẠP TRƯỚC, ĐÁNH INDEX SAU.**

Vì sao? Vì nếu bảng đã có index HNSW, thì **mỗi dòng bạn chèn vào đều buộc Postgres phải cập nhật lại đồ thị HNSW** — chèn thêm nút, nối lại các cạnh. Làm việc đó 20 triệu lần là vô cùng lãng phí.

Ngược lại, nếu bạn nạp hết vào bảng chưa có index rồi mới `CREATE INDEX` một lần, Postgres xây đồ thị **một lượt** với toàn bộ dữ liệu trong tay — nhanh hơn rất nhiều và cho ra cấu trúc tốt hơn.

Cứ hình dung: sắp xếp lại cả tủ sách **một lần** sau khi mua xong 500 cuốn, so với việc **xếp lại cả tủ sau mỗi lần mua một cuốn**. Phép ví von này khớp chính xác ở chỗ quan trọng: chi phí không nằm ở việc bỏ sách vào tủ, mà ở việc **duy trì trật tự** sau mỗi lần bỏ vào.

**Chốt thứ ba: làm cho quy trình nạp *chạy lại được*.**

**Idempotent** (dịch: *bất biến khi lặp lại*) nghĩa là chạy một lần hay chạy mười lần đều cho cùng kết quả — không nhân đôi dữ liệu. Điều này cực kỳ quan trọng vì tiến trình nạp dữ liệu lớn **sẽ đứt giữa chừng** (mất mạng, hết bộ nhớ, ai đó bấm Ctrl+C), và bạn cần chạy lại được mà không sợ.

Cách làm phổ biến: nạp vào một **bảng tạm** trước, rồi **upsert** sang bảng thật. **Upsert** là ghép của *update* và *insert*: nếu bản ghi đã có thì cập nhật, chưa có thì thêm mới.

### 2.7. Trạm 7 — TRUY VẤN: lấy ra thế nào cho đúng?

**Câu hỏi cốt lõi:** viết câu truy vấn ra sao để vừa nhanh vừa đúng?

**Khung xương của mọi truy vấn vector:**

```sql
ORDER BY embedding <=> :q    -- sắp theo khoảng cách, tăng dần (nhỏ = gần)
LIMIT k                       -- lấy k cái đầu
```

Ba toán tử khoảng cách của pgvector:

| Toán tử | Thước đo | Ops class tương ứng |
|---|---|---|
| `<=>` | Cosine distance — **mặc định cho văn bản** | `vector_cosine_ops` |
| `<->` | Khoảng cách Euclid (L2) | `vector_l2_ops` |
| `<#>` | Tích vô hướng **âm** | `vector_ip_ops` |

⚠️ **Bẫy thứ nhất — `<=>` trả về *distance*, không phải *similarity*.** Distance = 0 nghĩa là **giống hệt**; distance = 2 nghĩa là **ngược hẳn**. Muốn hiển thị "độ giống %" cho người dùng thì phải đổi: `1 - (embedding <=> :q)`. Nếu hiển thị thẳng distance dưới nhãn "độ giống", kết quả giống hệt sẽ hiện thành **0%**.

⚠️ **Bẫy thứ hai — `LIMIT` không đảm bảo chất lượng.** `LIMIT 5` **luôn** trả về đủ 5 dòng, kể cả khi cả 5 đều chẳng liên quan gì — chúng chỉ là "ít xa nhất trong đám xa". Muốn đảm bảo chất lượng phải thêm **ngưỡng (threshold)**:

```sql
WHERE embedding <=> :q < 0.3     -- ngưỡng: đảm bảo CHẤT LƯỢNG
ORDER BY embedding <=> :q
LIMIT 5;                          -- LIMIT: giới hạn SỐ LƯỢNG
```

**Câu để nhớ suốt đời: `LIMIT` giới hạn SỐ LƯỢNG, ngưỡng đảm bảo CHẤT LƯỢNG.**

⚠️ **Bẫy thứ ba — ngưỡng đứng một mình sẽ làm mất index.** Câu `WHERE embedding <=> :q < 0.3` mà **không** kèm `ORDER BY` và `LIMIT` buộc Postgres tính khoảng cách cho **mọi dòng**. Lý do: index ANN được thiết kế cho bài toán "tìm k cái gần nhất", không phải "tìm mọi thứ trong bán kính x". Luôn kèm `ORDER BY ... LIMIT k`.

⚠️ **Bẫy thứ tư — khi tìm "giống với một bản ghi có sẵn", nhớ loại chính nó.** Khoảng cách từ một vector tới chính nó bằng 0, nên nó luôn đứng đầu: `WHERE id != :current_id`.

### 2.8. Trạm 8 — HYBRID: ghép tìm theo từ khóa vào

**Câu hỏi cốt lõi:** làm sao bù điểm mù của tìm kiếm ngữ nghĩa?

Vấn đề: tìm theo ý nghĩa rất giỏi bắt ý, nhưng **hay trượt ở chỗ cần chính xác từng chữ** — mã sản phẩm `SKU-88XZ`, tên riêng hiếm, số phiên bản. Những thứ này gần như không mang ý nghĩa ngữ nghĩa nào để mô hình nắm bắt.

Tìm theo **từ khóa** thì ngược lại: chính xác tuyệt đối với mã và tên riêng, nhưng mù ngữ nghĩa ("ô tô" không tìm ra "xe hơi").

Giải pháp: **chạy cả hai, rồi hợp nhất kết quả.** Đó là hybrid search.

Ở phía từ khóa, PostgreSQL dùng kiểu **`tsvector`** (danh sách từ đã chuẩn hóa), truy vấn bằng **`tsquery`**, so khớp bằng toán tử **`@@`**, và đánh index bằng **GIN** — một loại **inverted index** (*chỉ mục ngược*), tức bảng tra "từ → danh sách tài liệu chứa từ đó", đúng như mục lục cuối sách.

Ở phía hợp nhất, kỹ thuật chuẩn là **RRF** (viết tắt của *Reciprocal Rank Fusion*, dịch: *hợp nhất theo nghịch đảo thứ hạng*). Mục 3.5 sẽ tính tay phép này để bạn thấy nó hoạt động ra sao.

### 2.9. Ba sợi chỉ đỏ xuyên qua nhiều trạm

Đây là phần phân biệt người học thuộc từng bài với người hiểu hệ thống. Có ba sự thật **không thuộc riêng trạm nào** mà xuyên qua nhiều trạm cùng lúc.

**Sợi chỉ thứ nhất — "cùng một mô hình" xuất hiện ở ba trạm.** Ở trạm 1 khi tạo vector, ở trạm 4 khi khai số chiều của cột cho khớp, và ở trạm 7 khi embed câu truy vấn. Ba chỗ khác nhau, cùng một ràng buộc. Vi phạm ở bất kỳ chỗ nào cũng làm hỏng toàn bộ.

**Sợi chỉ thứ hai — "nạp trước index sau" (trạm 6) chỉ có nghĩa nếu hiểu trạm 2.** Nếu bạn không biết HNSW là một đồ thị phải duy trì mỗi lần chèn, lời khuyên ở trạm 6 nghe như mê tín. Biết rồi thì nó hiển nhiên. Đây là ví dụ điển hình của việc **hiểu cơ chế giúp nhớ quy tắc mà không cần học thuộc**.

**Sợi chỉ thứ ba — ngưỡng ở trạm 7 phụ thuộc thước đo chọn ở trạm 1 và 4.** Cosine distance luôn nằm trong `[0, 2]`, nên ngưỡng 0.3 có ý nghĩa ổn định. Nhưng L2 **không có giới hạn trên** — một ngưỡng như "5" có thể quá chặt với bộ dữ liệu này và quá lỏng với bộ khác. Chọn thước đo ở đầu đường ống quyết định cách bạn lọc ở cuối đường ống.

### ✅ Self-check Phần 2

**1. Với dữ liệu tiếng Việt nhạy cảm (không được gửi ra API bên ngoài), quyết định ở trạm 1 và trạm 3 nên là gì?**
> *Gợi ý đáp án:* Trạm 1 — chọn mô hình **tự chạy được** và mạnh về đa ngôn ngữ (Qwen3-Embedding hoặc Llama-Embed-Nemotron), kiểm tra thêm trên bảng VN-MTEB. Trạm 3 — **pgvector tự vận hành**, vì ràng buộc quyền riêng tư loại ngay mọi dịch vụ đám mây.

**2. Vì sao "nạp trước, đánh index sau" liên quan tới kiến thức ở trạm 2?**
> *Gợi ý đáp án:* Vì HNSW là một đồ thị phải cập nhật lại sau **mỗi** lần chèn. Nạp vào bảng chưa có index rồi xây một lượt sẽ nhanh hơn rất nhiều và cho cấu trúc tốt hơn — giống xếp lại tủ sách một lần thay vì sau mỗi cuốn.

**3. Một câu hybrid gồm hai nhánh nào, và hợp nhất bằng gì?**
> *Gợi ý đáp án:* Nhánh từ khóa (`tsvector` với toán tử `@@`, index GIN) và nhánh ngữ nghĩa (`vector` với toán tử `<=>`, index HNSW). Hợp nhất bằng **RRF** — cộng nghịch đảo thứ hạng từ hai nhánh.

**4. `LIMIT` và ngưỡng khác nhau ở chỗ nào?**
> *Gợi ý đáp án:* `LIMIT` giới hạn **số lượng**; ngưỡng đảm bảo **chất lượng**. `LIMIT 5` luôn trả đủ 5 dòng kể cả khi cả 5 đều là rác. Hệ thống tốt cần cả hai, và ngưỡng phải kèm `ORDER BY ... LIMIT` để không mất index.

---

## Phần 3 — 🔴 ADVANCED: Năm chủ đề xuyên suốt

> Đây là những ý **không thuộc riêng bài nào**. Chính chúng phân biệt người học thuộc từng bài với người hiểu cả hệ thống — và chính chúng là thứ người phỏng vấn dò tìm.

### 3.1. Đánh đổi giữa nhanh và đúng: recall là gì và đo nó ra sao

Sự thật xuyên suốt từ trạm 2 tới trạm 7: **ANN đổi độ chính xác lấy tốc độ.**

Không có index thì tìm chính xác tuyệt đối nhưng chậm `O(n)`. Có index HNSW hoặc IVFFlat thì nhanh hơn hàng nghìn lần nhưng **có thể bỏ sót** vài hàng xóm thật.

Mức độ bỏ sót đó được đo bằng **recall** (dịch: *độ bao phủ*).

**Ví dụ chạy tay — tính recall@10 bằng tay.**

Giả sử bạn muốn lấy 10 kết quả gần nhất cho một câu truy vấn.

*Bước 1 — chạy tìm chính xác* (tắt index, hoặc chạy trên bản sao nhỏ) để biết **đáp án đúng**:
```
Đáp án đúng (10 id gần nhất):  [7, 12, 45, 88, 91, 103, 150, 202, 310, 415]
```

*Bước 2 — chạy tìm bằng index ANN:*
```
Kết quả ANN trả về:  [7, 12, 45, 88, 91, 103, 150, 202, 500, 612]
                                                       ^^^  ^^^
                                                    hai id lạ
```

*Bước 3 — đếm số phần tử trùng nhau:* có **8** id xuất hiện ở cả hai danh sách (7, 12, 45, 88, 91, 103, 150, 202).

*Bước 4 — tính:*
```
recall@10 = 8 / 10 = 0.80  (tức 80%)
```

Nghĩa là index đang bỏ sót **20%** kết quả đúng. Với gợi ý sản phẩm thì chấp nhận được; với hệ thống rà soát pháp lý thì không.

**Cách cải thiện recall:** tăng tham số **`ef_search`** — số ứng viên mà index xét trước khi dừng. Cao hơn thì recall tốt hơn nhưng truy vấn chậm hơn. Đây là **cái núm vặn chính** trên đường cong đánh đổi.

```sql
SET hnsw.ef_search = 100;   -- mặc định là 40; tăng lên thì chính xác hơn, chậm hơn
```

⚠️ **Chỗ khó — và đây là điều quan trọng nhất trong mục này.** Recall tụt **không gây lỗi, không làm chậm, không có cảnh báo nào**. Hệ thống vẫn trả đủ số kết quả, vẫn nhanh, mọi biểu đồ giám sát vẫn xanh. Nó chỉ lặng lẽ trả về những kết quả tệ hơn. **Nếu bạn không chủ động đo, bạn sẽ không bao giờ biết.**

Vì vậy, câu trả lời ở tầm staff không phải "tôi dùng HNSW" mà là: ***"tôi biết mình đang ở đâu trên đường cong recall–tốc độ, và tôi đo nó định kỳ bằng cách so kết quả ANN với kết quả chính xác trên một tập mẫu."***

### 3.2. Vì sao không thể tìm vector bằng cách thông thường

Đây là câu hỏi bẫy hay gặp: *"Sao không dùng B-tree cho vector, như mọi cột khác?"*

**B-tree** là loại index mặc định của mọi cơ sở dữ liệu. Nó hoạt động bằng cách **sắp xếp giá trị theo thứ tự** rồi tìm kiếm nhị phân — chia đôi liên tục, nên rất nhanh.

**Binary search** (dịch: *tìm kiếm nhị phân*) là ý tưởng: muốn tìm một số trong danh sách đã sắp xếp, hãy nhìn vào giữa; nếu số cần tìm lớn hơn thì bỏ nửa trái, nhỏ hơn thì bỏ nửa phải. Mỗi bước loại được một nửa.

**Vì sao cách đó không áp dụng được cho vector? Hai lý do.**

**Lý do thứ nhất: không có "một trục" để sắp xếp.** Với số hay chữ, bạn sắp theo đúng một chiều — 1 < 2 < 3. Nhưng vector 1024 chiều thì sắp theo chiều nào? Sắp theo chiều thứ nhất thì hai vector giống nhau về 1023 chiều còn lại có thể nằm cách xa nhau trong danh sách. **Khái niệm "thứ tự" đơn giản là không tồn tại trong không gian nhiều chiều.**

**Lý do thứ hai: lời nguyền số chiều.**

**Curse of dimensionality** (dịch: *lời nguyền số chiều*) là hiện tượng: khi số chiều tăng lên, **mọi điểm đều trở nên gần như cách đều nhau**, khiến khái niệm "gần nhất" mất dần ý nghĩa.

Hình dung dần dần cho dễ thấy: trên **một đường thẳng** (1 chiều), một điểm chỉ có 2 hàng xóm — trái và phải. Trên **mặt phẳng** (2 chiều), nó có hàng xóm quanh 360 độ. Trong **không gian ba chiều**, hàng xóm bủa vây mọi hướng. Cứ thêm chiều thì "xung quanh" lại phình ra theo cấp số nhân, trong khi số điểm dữ liệu của bạn thì không đổi. Kết quả là dữ liệu trở nên **cực kỳ thưa thớt**, và khoảng cách giữa điểm gần nhất và điểm xa nhất co lại gần bằng nhau.

**Kết luận:** không sắp thứ tự được, cộng với lời nguyền số chiều, nên **buộc phải chấp nhận tìm gần đúng** — và đó chính là lý do tồn tại của cả họ thuật toán ANN ở trạm 2.

Đây là một câu trả lời rất ghi điểm, vì nó cho thấy bạn hiểu **vì sao ngành này phải phát minh ra HNSW**, chứ không chỉ biết dùng nó.

### 3.3. Luận điểm "một Postgres cho tất cả"

Sự thật xuyên suốt từ trạm 3 và trạm 8: **siêu năng lực của pgvector không phải tốc độ đỉnh — mà là vector sống cạnh dữ liệu nghiệp vụ.**

Hãy lấy một yêu cầu hết sức bình thường: *"Tìm 10 sản phẩm giống cái này nhất, nhưng chỉ trong số còn hàng và giá dưới 50 đô."*

**Với pgvector — một câu SQL:**

```sql
SELECT id, name FROM products
WHERE in_stock = true AND price < 50
ORDER BY embedding <=> :q
LIMIT 10;
```

Một câu lệnh, một lần gọi mạng, một **transaction** (dịch: *giao dịch* — một nhóm thao tác được database đảm bảo hoặc làm hết hoặc không làm gì). Kết quả luôn nhất quán vì cả điều kiện lọc lẫn vector đều đọc từ cùng một ảnh chụp dữ liệu tại một thời điểm.

**Với hệ vector tách rời — ba bước:**

```
Bước 1: gửi vector sang hệ vector → nhận về danh sách ID
Bước 2: gửi danh sách ID sang database chính → hỏi cái nào còn hàng, giá dưới 50
Bước 3: ghép kết quả trong code ứng dụng
```

Ba vấn đề, và không cái nào nhỏ:

**Một — hai lần gọi mạng.** Mỗi lần gọi qua mạng gọi là một **round-trip** (*một vòng đi-về*), tốn vài mili-giây. Hai round-trip tuần tự thì độ trễ cộng dồn.

**Hai — bài toán "lọc sau" không có lời giải đẹp.** Bạn lấy top 10 từ hệ vector, lọc theo điều kiện, và phát hiện chỉ 2 cái thỏa. Giờ làm gì? Xin top 100? Rồi top 1000? Đây là vấn đề nổi tiếng trong ngành. Còn khi vector nằm cùng bảng, **bộ tối ưu truy vấn của database tự lo chuyện này**.

**Ba — đồng bộ dữ liệu.** Mỗi lần thêm, sửa, xóa sản phẩm là phải nhớ cập nhật cả hệ vector. Quên một đường là hai hệ lệch nhau, và triệu chứng sẽ là "sản phẩm đã xóa vẫn hiện trong kết quả" — nguồn lỗi kinh điển của mọi kiến trúc hai hệ thống.

**Kim chỉ nam:** vector là **tính năng** → Postgres. Vector là **sản phẩm**, hoặc hàng tỷ vector, hoặc nhiều vùng địa lý → hệ chuyên dụng.

### 3.4. Chi phí ẩn lớn nhất: đổi mô hình nghĩa là làm lại tất cả

Sự thật xuyên suốt từ trạm 1, 3 và 6, và đây là **quyết định nặng nhất trong toàn bộ kiến trúc** — nặng hơn cả việc chọn loại index.

Nhắc lại nguyên nhân gốc: vector do hai mô hình khác nhau nằm trên hai "bản đồ" khác nhau, nên **không so sánh được**. Hệ quả: đổi mô hình embedding nghĩa là phải **sinh lại toàn bộ vector cho cả kho** — gọi là **re-embed**.

**Ví dụ chạy tay — tính chi phí re-embed bằng số.**

Giả sử kho của bạn có **20 triệu đoạn văn bản**, mỗi đoạn trung bình 200 **token** (token là đơn vị mà mô hình đếm, xấp xỉ ba phần tư một từ tiếng Anh).

```
Tổng token cần xử lý:  20.000.000 × 200 = 4 tỷ token

Nếu dùng API giá 0,02 đô mỗi triệu token:
    4.000 triệu token × 0,02 đô = 80 đô
    → nghe rẻ, nhưng...

Nếu dùng mô hình chất lượng cao hơn, giá 0,15 đô mỗi triệu token:
    4.000 × 0,15 = 600 đô

Cộng thêm những khoản không ai nhớ tính:
  · Thời gian xử lý: vài ngày tới vài tuần tùy thông lượng
  · Phải chạy SONG SONG hai bộ index (cũ và mới) trong lúc chuyển đổi
    → tăng gấp đôi RAM tạm thời
  · Công sức kiểm chứng chất lượng trước khi chuyển
  · Phải hiệu chỉnh lại NGƯỠNG (mục 2.7) vì ngưỡng gắn với mô hình
```

Con số tiền có thể không lớn, nhưng **thời gian, rủi ro và công sức thì lớn** — và đó mới là thứ khiến các đội trì hoãn việc nâng cấp mô hình suốt nhiều năm dù biết có mô hình tốt hơn.

**Bốn cách giữ đường lui — nên làm ngay từ ngày đầu:**

1. **Chốt mô hình sớm và cân nhắc kỹ**, vì đổi về sau rất đắt.
2. **Ghi phiên bản mô hình** vào một cột bên cạnh mỗi vector (như mục 2.4), để biết hàng nào cần làm lại.
3. **Đánh phiên bản cho cột** — thêm `embedding_v2` bên cạnh `embedding` thay vì ghi đè, rồi chuyển đổi kiểu **blue-green**: dựng song song bộ mới, kiểm chứng xong mới chuyển toàn bộ lưu lượng sang, và giữ bộ cũ để quay lui nếu có sự cố.
4. **Ưu tiên tự sinh embedding (BYO) thay vì để database sinh hộ.** Có những nền tảng — như MySQL HeatWave — tự sinh embedding cho bạn. Rất tiện, nhưng vector của bạn khi đó bị buộc vào mô hình của nhà cung cấp đó. **Tiện lợi đi kèm sợi xích.**

### 3.5. Hybrid search và cách hợp nhất bằng RRF

Sự thật xuyên suốt từ trạm 8 và trạm 1: **tìm theo từ khóa và tìm theo ý nghĩa bù khuyết cho nhau.**

| | **Tìm theo từ khóa (FTS)** | **Tìm theo ý nghĩa (vector)** |
|---|---|---|
| Mạnh ở | Thuật ngữ, mã sản phẩm, tên riêng | Từ đồng nghĩa, cách diễn đạt khác |
| Yếu ở | **Mù ngữ nghĩa** ("ô tô" ≠ "xe hơi") | Trượt thuật ngữ hiếm và mã sản phẩm |
| Giải thích được kết quả? | **Có** — "vì chứa đúng từ này" | Khó |
| Cần mô hình AI? | Không | Có |

Nhìn bảng là thấy ngay: chúng **không cạnh tranh, chúng bổ sung**. Hybrid = chạy cả hai rồi hợp nhất.

**Nhưng hợp nhất bằng cách nào?** Đây là chỗ tinh tế. Điểm của `ts_rank` (nhánh từ khóa) và khoảng cách cosine (nhánh vector) nằm trên **hai thang đo hoàn toàn khác nhau** — cộng chúng lại là vô nghĩa, giống như cộng cân nặng với chiều cao.

**RRF** giải quyết bằng một ý tưởng đơn giản đến bất ngờ: **đừng cộng điểm, hãy cộng thứ hạng.** Vì thứ hạng thì luôn so sánh được — đứng thứ nhất ở nhánh nào cũng là đứng thứ nhất.

Công thức: mỗi tài liệu nhận `1 / (k + thứ_hạng)` từ mỗi nhánh, rồi cộng lại. Hằng số `k` thường chọn 60, có tác dụng làm mượt để vị trí số 1 không áp đảo quá mức so với vị trí số 2.

**Ví dụ chạy tay — tính RRF bằng tay.**

Giả sử hai nhánh trả về hai danh sách xếp hạng như sau:

```
Nhánh TỪ KHÓA:            Nhánh NGỮ NGHĨA:
  hạng 1: tài liệu A        hạng 1: tài liệu C
  hạng 2: tài liệu B        hạng 2: tài liệu A
  hạng 3: tài liệu C        hạng 3: tài liệu D
```

Tính điểm RRF cho từng tài liệu (dùng k = 60):

```
Tài liệu A:  có ở cả hai nhánh — hạng 1 và hạng 2
             1/(60+1) + 1/(60+2) = 0,01639 + 0,01613 = 0,03252   ← cao nhất

Tài liệu C:  có ở cả hai nhánh — hạng 3 và hạng 1
             1/(60+3) + 1/(60+1) = 0,01587 + 0,01639 = 0,03226

Tài liệu B:  chỉ có ở nhánh từ khóa — hạng 2
             1/(60+2) = 0,01613

Tài liệu D:  chỉ có ở nhánh ngữ nghĩa — hạng 3
             1/(60+3) = 0,01587
```

**Xếp hạng cuối cùng: A → C → B → D.**

Hãy để ý điều RRF vừa làm được, vì nó rất hợp trực giác: **tài liệu nào lọt vào top của CẢ HAI nhánh sẽ lên đầu.** Tài liệu A không đứng nhất ở nhánh nào cả, nhưng nó có mặt ở vị trí cao trong cả hai — và đó chính là tín hiệu mạnh nhất cho thấy nó thật sự liên quan. Còn B và D, dù mỗi cái đứng khá cao trong nhánh của mình, vẫn xếp sau vì chỉ một nhánh "bỏ phiếu" cho chúng.

Và điểm mấu chốt về mặt kiến trúc: **pgvector cộng full-text search làm được toàn bộ chuyện này bên trong một Postgres duy nhất** — không cần chạy song song một vector database và một cụm Elasticsearch. Đây là lúc luận điểm ở mục 3.3 trả cổ tức.

### 3.6. Tổng hợp các trường hợp biên của cả khóa

**Edge case** (dịch: *trường hợp biên*) là những tình huống hiếm gặp nhưng gây hỏng hoặc sai. Chủ động nêu chúng trong phỏng vấn luôn ghi điểm.

| Trường hợp biên | Triệu chứng | Cách chữa |
|---|---|---|
| Trộn vector từ hai mô hình **cùng số chiều** | Kết quả sai âm thầm, không báo lỗi | Ghi phiên bản mô hình vào cột riêng |
| Ops class không khớp toán tử | Index bị bỏ qua, chậm dần, không cảnh báo | `EXPLAIN ANALYZE`, tìm `Index Scan` |
| Ngưỡng đứng một mình | Quét toàn bảng | Luôn kèm `ORDER BY ... LIMIT k` |
| Quên loại chính bản ghi nguồn | Kết quả đầu luôn là chính nó | `WHERE id != :self` |
| Ngưỡng quá chặt | Trả về 0 kết quả | Xử lý ở tầng ứng dụng: ẩn khối, đừng báo lỗi |
| Lọc mạnh cộng ANN (**overfiltering**) | Thiếu kết quả dù kho vẫn còn | Bật `hnsw.iterative_scan`, hoặc tăng `ef_search` |
| Cột vector bị NULL | Hàng đó biến mất khỏi mọi kết quả | Theo dõi số hàng `embedding IS NULL` |
| `CREATE INDEX` khóa ghi trên bảng lớn | Không ghi được trong lúc xây | Dùng `CREATE INDEX CONCURRENTLY` |
| Đổi mô hình mà quên hiệu chỉnh ngưỡng | Chất lượng tụt sau khi "nâng cấp" | Hiệu chỉnh lại ngưỡng trên tập mẫu đã gán nhãn |

### ✅ Self-check Phần 3

**1. "ANN đổi độ chính xác lấy tốc độ" thể hiện ở những trạm nào, và đo bằng cách nào?**
> *Gợi ý đáp án:* Thể hiện ở trạm 2 (chọn index) và trạm 7 (ngưỡng áp lên kết quả gần đúng nên có thể sót). Đo bằng **recall@k**: chạy tìm chính xác trên một tập mẫu để có đáp án đúng, chạy ANN, rồi đếm tỉ lệ trùng nhau. Núm vặn để cải thiện là `ef_search`.

**2. Vì sao không dùng B-tree cho vector được?**
> *Gợi ý đáp án:* Hai lý do. B-tree cần **một trục để sắp thứ tự**, mà không gian nhiều chiều không có khái niệm thứ tự như vậy. Cộng thêm **lời nguyền số chiều** — càng nhiều chiều thì mọi điểm càng trở nên cách đều nhau. Vì vậy buộc phải dùng ANN.

**3. Vì sao re-embedding là quyết định nặng hơn chọn index?**
> *Gợi ý đáp án:* Đổi index chỉ cần xây lại cấu trúc phụ từ dữ liệu sẵn có. Đổi mô hình embedding thì phải **sinh lại toàn bộ vector cho cả kho** — tốn tiền gọi mô hình, tốn nhiều ngày xử lý, phải chạy song song hai bộ index, và phải hiệu chỉnh lại cả ngưỡng.

**4. RRF hợp nhất kết quả như thế nào, và vì sao không cộng điểm trực tiếp?**
> *Gợi ý đáp án:* RRF cộng `1/(60 + thứ_hạng)` từ mỗi nhánh. Không cộng điểm trực tiếp được vì `ts_rank` và khoảng cách cosine nằm trên hai thang đo khác nhau, không so sánh được. Tài liệu lọt vào top của cả hai nhánh sẽ có tổng điểm cao nhất — đúng như trực giác.

---

## Phần 4 — 🟣 STAFF LEVEL: Thiết kế hệ hoàn chỉnh & dự án capstone

### 4.1. Câu hỏi system design gộp mọi quyết định của khóa

> **Đề bài:** *"Thiết kế hệ tìm kiếm ngữ nghĩa / RAG cho 20 triệu tài liệu. Đa ngôn ngữ, có tiếng Việt. Dữ liệu khách hàng nhạy cảm, không được gửi ra API bên ngoài. Độ trễ p95 dưới 150ms. Cập nhật hàng ngày, không được ngừng dịch vụ. Đội backend nhỏ, đang dùng PostgreSQL."*

Đề bài này gộp **cả tám trạm**. Điều quan trọng nhất không phải đưa ra "đáp án đúng", mà là **nói to quá trình loại trừ và những đánh đổi bạn đang cân nhắc**.

**Nhịp 1 — Làm rõ đề trước khi vẽ.** Đừng nhảy vào thiết kế ngay. Hỏi trước:
- **QPS** (viết tắt của *Queries Per Second*, số truy vấn mỗi giây) dự kiến bao nhiêu?
- Mục tiêu recall là bao nhiêu — 90% có đủ không, hay phải 99%?
- Ràng buộc tuân thủ cụ thể là gì: dữ liệu không được ra khỏi công ty, hay chỉ không được ra khỏi lãnh thổ?
- Ngân sách hạ tầng? Tần suất cập nhật nội dung?

**Nhịp 2 — Trạm 1, chọn mô hình.** Dữ liệu nhạy cảm cộng đa ngôn ngữ dẫn tới: **mô hình tự chạy trong nhà**, mạnh về đa ngữ (Qwen3-Embedding hoặc Llama-Embed-Nemotron), và kiểm chứng thêm trên bảng **VN-MTEB** cho phần tiếng Việt.

Ở đây cần nói thêm về **chunking** (dịch: *chia đoạn*) — tài liệu dài phải cắt thành các đoạn nhỏ trước khi embed, vì mô hình có giới hạn độ dài đầu vào, và vì một vector đại diện cho 50 trang thì quá mơ hồ để tìm kiếm chính xác. Kèm theo: xử lý theo **batch** (*theo lô*) trên GPU để tăng thông lượng, và cache theo mã băm để không embed lại nội dung trùng.

**Nhịp 3 — Trạm 3, chọn nền tảng.** 20 triệu vector nằm **dưới ngưỡng** cần hệ chuyên dụng; vector là tính năng chứ không phải sản phẩm; công ty đã chạy Postgres và đội nhỏ. Kết luận: **pgvector tự vận hành**. Và ràng buộc quyền riêng tư đã loại mọi dịch vụ đám mây **trước khi** ta kịp bàn tới hiệu năng.

Đây là một điểm rất đặc trưng của tư duy staff: **tìm ràng buộc cứng trước, rồi mới tối ưu trong phần không gian còn lại.** Tối ưu xong mới phát hiện phương án vi phạm quy định là cách lãng phí vài tuần của cả đội.

**Nhịp 4 — Trạm 4, thiết kế bảng.**

```sql
CREATE TABLE docs (
    id         bigserial,
    tenant_id  bigint,           -- khách hàng doanh nghiệp nào
    lang       text,             -- ngôn ngữ, dùng để lọc và phân mảnh
    content    text,
    embedding  vector(1024),
    model_name text,             -- phiên bản mô hình, để biết hàng nào cần re-embed
    updated_at timestamptz
) PARTITION BY LIST (tenant_id);
```

**Partition** (dịch: *phân mảnh*) là chia một bảng khổng lồ thành nhiều bảng con theo một tiêu chí. Khi truy vấn có điều kiện theo tiêu chí đó, bộ lập kế hoạch sẽ **prune** — bỏ qua hẳn những mảnh không liên quan. Với hệ nhiều khách hàng, phân mảnh theo `tenant_id` vừa tăng tốc vừa cô lập dữ liệu giữa các khách.

Thêm index B-tree thông thường trên các cột dùng để lọc (`lang`, `updated_at`).

**Nhịp 5 — Trạm 5, cài đặt.** Docker với thẻ phiên bản được ghim, hoặc gói của hệ điều hành; `CREATE EXTENSION` đưa vào migration; phiên bản trên replica phải khớp máy chính.

**Nhịp 6 — Trạm 6, nạp dữ liệu.** Nạp lần đầu bằng **`COPY`** vào bảng **chưa có index**; quy trình phải **chạy lại được** (bảng tạm rồi upsert); cập nhật hàng ngày qua **hàng đợi bất đồng bộ** để không chặn đường ghi chính.

**Nhịp 7 — Trạm 2, đánh index.** **HNSW** với `vector_cosine_ops`, xây **sau khi** nạp xong, dùng `CREATE INDEX CONCURRENTLY` để không khóa ghi, và tăng **`maintenance_work_mem`** (tham số quy định lượng RAM cho các thao tác bảo trì) trước khi xây để rút ngắn thời gian đáng kể. Nếu RAM căng thì cân nhắc **quantization** (giảm độ chính xác từng số để tiết kiệm bộ nhớ) hoặc DiskANN.

**Nhịp 8 — Trạm 7, truy vấn.**

```sql
WITH nearest AS MATERIALIZED (
    SELECT id, content, embedding <=> :q AS distance
    FROM docs
    WHERE tenant_id = :tenant AND lang = :lang    -- lọc trước để thu hẹp
    ORDER BY distance
    LIMIT 50                                       -- lấy rộng
)
SELECT * FROM nearest
WHERE distance < :threshold                        -- ngưỡng chất lượng
ORDER BY distance
LIMIT 10;
```

Chỉnh `ef_search` theo mục tiêu p95; bật `hnsw.iterative_scan` vì có lọc mạnh; ngưỡng được **hiệu chỉnh trên golden set** (mục 4.3), không đoán.

**Nhịp 9 — Trạm 8, hybrid.** Thêm cột `tsvector` với index GIN cho nhánh từ khóa, hợp nhất với nhánh vector bằng RRF. Với tiếng Việt, lưu ý PostgreSQL **không có bộ xử lý ngôn ngữ tiếng Việt sẵn** — nên dùng cấu hình `simple` kết hợp extension `unaccent` (bỏ dấu, để người gõ không dấu vẫn tìm được), hoặc dựa nhiều hơn vào nhánh vector.

**Nhịp 10 — Vận hành.** Giám sát recall@k so với kết quả chính xác, p99, độ trễ nhân bản sang replica, và **drift** — phân bố khoảng cách dịch chuyển theo thời gian, dấu hiệu cần hiệu chỉnh lại ngưỡng. Khi đổi mô hình thì triển khai blue-green với cột được đánh phiên bản.

**Nhịp 11 — Tự đánh giá, và đây là nhịp quan trọng nhất.**

> *"Điểm nghẽn của hệ này là khâu **sinh embedding**, không phải khâu chèn hay khâu tìm kiếm — nên đó là chỗ tôi sẽ đầu tư GPU và tối ưu batch trước tiên.*
>
> *Ràng buộc quyền riêng tư **ép** tôi tự vận hành, bất kể điểm benchmark nói gì.*
>
> *Với 20 triệu vector tôi chọn pgvector, nhưng tôi biết rõ **ngưỡng** nào sẽ khiến tôi đổi ý: khi lên hàng trăm triệu vector hoặc phải phục vụ nhiều vùng địa lý. Trước ngưỡng đó, tôi sẽ thử pgvectorscale với DiskANN đã, rồi mới tính tới việc tách hệ."*

**Biết lý do và biết giới hạn của mọi lựa chọn mình đưa ra** — đó là thứ phân biệt rõ nhất giữa một câu trả lời khá và một câu trả lời ở tầm staff.

### 4.2. Hiệu chỉnh ngưỡng bằng golden set

Ngưỡng lọc xuất hiện ở trạm 7, nhưng cách xác định nó là một quy trình đáng nói riêng, vì đây là chỗ nhiều đội làm ẩu nhất — họ đoán một con số rồi để đó mãi.

**Golden set** (dịch: *tập vàng*) là bộ dữ liệu gồm các cặp truy vấn – kết quả **đã được con người gán nhãn** liên quan hay không liên quan.

**Quy trình bốn bước:**

1. **Xây tập vàng** — vài trăm cặp là đủ để bắt đầu. Lấy từ nhật ký tìm kiếm thật rồi nhờ người trong đội hoặc bộ phận nghiệp vụ gán nhãn.
2. **Đo khoảng cách** của nhóm liên quan và nhóm không liên quan, ví dụ:
   ```
   Nhóm LIÊN QUAN:      0.05  0.09  0.14  0.18  0.22  0.27
                                      ⬇ khoảng trống ⬇
   Nhóm KHÔNG LIÊN QUAN: 0.48  0.55  0.62  0.77  0.91
   ```
3. **Chọn ngưỡng nằm trong khoảng trống** — ở đây khoảng 0.35.
4. **Chạy lại định kỳ**, và **bắt buộc chạy lại mỗi khi đổi mô hình**.

Cần hiểu rõ mình đang vặn cái gì. **Precision** (dịch: *độ chính xác*) là tỉ lệ kết quả trả về mà đúng; **recall** là tỉ lệ kết quả đúng mà tìm được. Hai chỉ số này đánh đổi lẫn nhau, và ngưỡng chính là cái núm vặn: **chặt** thì precision cao nhưng bỏ sót nhiều và hay trả về 0 kết quả; **lỏng** thì recall cao nhưng lẫn rác.

💡 **Chẩn đoán quan trọng:** nếu hai nhóm **chồng lấn** nhau — nhóm liên quan có cặp ở 0.52 trong khi nhóm không liên quan có cặp ở 0.41 — thì **không tồn tại ngưỡng nào tách được chúng**. Khi đó vấn đề nằm ở **mô hình embedding không phù hợp với dữ liệu của bạn**, chứ không nằm ở ngưỡng. Nhận ra điều này cứu bạn khỏi hàng tuần chỉnh ngưỡng vô ích.

Và điểm ở tầm staff: golden set **không chỉ để đặt ngưỡng**. Nó còn dùng để đo recall của index, để so sánh hai mô hình embedding, và để trả lời câu hỏi "thay đổi này làm chất lượng tốt lên hay tệ đi". Vì vậy nó xứng đáng được xây dựng và duy trì như một phần chính thức của sản phẩm.

### 4.3. Dự án capstone — xây để đưa vào hồ sơ

Video gốc khuyên nên chia sẻ bài thực hành lên một kho mã công khai để nhà tuyển dụng xem. Đây là bản đặc tả một dự án trọn vẹn đáng để xây.

> **Tên dự án: "Hệ tìm kiếm ngữ nghĩa và hybrid trên PostgreSQL"**

**Mục tiêu:** tìm kiếm theo ý nghĩa trên một tập dữ liệu thật (một phần Wikipedia, hoặc tập câu hỏi thường gặp, hoặc danh mục sản phẩm), hỗ trợ cả tìm theo từ khóa lẫn theo ngữ nghĩa.

**Công nghệ:** PostgreSQL với pgvector chạy trong Docker; một mô hình embedding tự chạy; một API bằng FastAPI hoặc Express.

**Tám thành phần bắt buộc — đúng tám trạm của khóa:**

| # | Thành phần | Chứng minh bạn nắm được gì |
|---|---|---|
| 1 | Pipeline chia đoạn và embed theo lô | Trạm 1 — hiểu chunking và batch |
| 2 | Schema có cột `vector(n)`, metadata, và `tsvector` với index GIN | Trạm 4, 8 |
| 3 | Nạp bằng `COPY`, nạp trước index sau, chạy lại được | Trạm 6 — hiểu vì sao, không chỉ biết làm |
| 4 | Index HNSW, có chỉnh `ef_search` | Trạm 2 |
| 5 | Endpoint truy vấn: k-NN + ngưỡng + lọc nghiệp vụ | Trạm 7 |
| 6 | Endpoint hybrid: vector cộng từ khóa, hợp nhất bằng RRF | Trạm 8 |
| 7 | `EXPLAIN ANALYZE` chứng minh index được dùng, và đo recall@k trên golden set nhỏ | Tư duy đo lường |
| 8 | README bàn về **đánh đổi** | Tư duy staff |

**Và đây là phần thật sự gây ấn tượng — mục số 8.**

Hầu hết dự án trên hồ sơ đều dừng ở "chạy được". Thứ khiến người tuyển dụng dừng lại là một README trả lời được những câu hỏi kiểu:

- Vì sao chọn HNSW mà không phải IVFFlat? Chọn ở quy mô nào thì đổi ý?
- Ngưỡng đặt bằng bao nhiêu, và **hiệu chỉnh ra sao**?
- recall@10 đo được là bao nhiêu, và đánh đổi với độ trễ thế nào?
- **Khi nào thì không nên dùng pgvector?**

Câu cuối cùng là câu quan trọng nhất. Một người viết được phần "khi nào giải pháp của tôi không còn phù hợp" đang thể hiện đúng thứ mà tầm staff cần: **biết giới hạn của lựa chọn của mình**. Điều đó thuyết phục hơn mọi chứng chỉ.

### 4.4. Cách trình bày với người không rành kỹ thuật

**Stakeholder** (dịch: *bên liên quan*) là những người có lợi ích gắn với dự án nhưng không hiểu kỹ thuật: quản lý sản phẩm, giám đốc, bộ phận pháp chế. Một staff engineer phải trình bày quyết định kỹ thuật bằng ngôn ngữ của họ — tức là bằng **rủi ro, chi phí, thời gian và ràng buộc pháp lý**.

Một cách diễn đạt gói gọn cả khóa:

> *"Chúng ta thêm được khả năng tìm kiếm theo ý nghĩa vào chính cơ sở dữ liệu đang chạy — **không phải dựng hệ thống mới**. Đội hiện tại vận hành được, dữ liệu vẫn nằm nguyên một chỗ, nên rủi ro và chi phí đều thấp.*
>
> *Chi phí lớn nhất không phải là chỗ lưu trữ, mà là hai thứ: một, tiền và thời gian để AI tạo ra dữ liệu tìm kiếm cho toàn bộ kho; hai, nếu sau này ta đổi mô hình AI thì phải **xử lý lại toàn bộ dữ liệu từ đầu**.*
>
> *Vì vậy tôi đề xuất chốt lựa chọn mô hình sớm, và xây quy trình sao cho chạy lại được nhiều lần — để nếu buộc phải làm lại thì đó là việc tốn thời gian máy, không phải tốn thời gian người."*

Hãy để ý: không có chữ nào là "HNSW", "cosine", hay "recall". Thay vào đó là chi phí rõ ràng, rủi ro thấp, và một đề xuất cụ thể. Đó là ngôn ngữ mà stakeholder nghe được và ra quyết định được.

**Về cấu trúc đội ngũ:** chọn pgvector nghĩa là **một đội database lo tất**. Tách hệ vector riêng thường kéo theo kỹ năng mới, quy trình trực sự cố mới, và về lâu dài có thể là một vị trí tuyển dụng mới. Đây là lập luận **tổ chức**, không phải kỹ thuật — và biết đưa nó ra bàn đúng lúc là điều người ta chờ đợi ở tầm staff.

**Về chuẩn hóa toàn công ty:** nếu nhiều đội cùng cần vector, hãy chọn **một** mô hình chung thay vì để mỗi đội tự chọn một hệ khác nhau. Sự phân mảnh này ban đầu trông vô hại, nhưng sau hai năm nó biến thành năm hệ thống mà không ai nắm hết.

---

## Phần 5 — 🎯 MASTER CHEATSHEET: Ôn nhanh CẢ KHÓA

> Nếu chỉ kịp đọc một phần trước buổi phỏng vấn, đọc phần này.

### 5.1. Bảng thuật ngữ toàn khóa

| Thuật ngữ | Định nghĩa một dòng |
|---|---|
| **Embedding** | Vector mã hóa ý nghĩa; nội dung gần nghĩa cho vector gần nhau |
| **Dimension (số chiều)** | Số phần tử trong vector; phải khớp giữa mô hình và cột trong bảng |
| **Static vs contextual** | Word2Vec cho mỗi từ một vector cố định; transformer đổi vector theo ngữ cảnh nhờ attention |
| **Cosine distance** | Đo góc, bỏ qua độ lớn; miền `[0, 2]`; **`<=>` trả distance, không phải similarity** |
| **L2 / inner product** | `<->` khoảng cách Euclid (không giới hạn trên) / `<#>` tích vô hướng **âm** |
| **k-NN / ANN** | Tìm k gần nhất / tìm gần đúng, đổi độ chính xác lấy tốc độ |
| **Recall@k** | Tỉ lệ kết quả đúng mà ANN tìm được so với tìm chính xác |
| **HNSW** | Đồ thị nhiều tầng, ~`O(log n)`, recall cao, **ngốn RAM** — mặc định |
| **IVFFlat** | Chia cụm theo centroid, nhẹ RAM, cần sẵn dữ liệu để chia, recall thấp hơn |
| **DiskANN / pgvectorscale** | Chạy trên SSD, rẻ hơn ở quy mô hàng trăm triệu vector |
| **Curse of dimensionality** | Nhiều chiều thì mọi điểm gần như cách đều nhau → không sắp thứ tự được → buộc dùng ANN |
| **pgvector** | Extension thêm kiểu `vector` cho PostgreSQL; dòng 0.8.x (2026) |
| **Ops class** | `vector_cosine_ops` / `vector_l2_ops` / `vector_ip_ops` — **phải khớp toán tử truy vấn** |
| **`ef_search`** | Số ứng viên index xét trước khi dừng — núm vặn giữa recall và tốc độ |
| **`tsvector` / `tsquery` / `@@`** | Kiểu và toán tử của full-text search — **lexeme, khác hoàn toàn `vector`** |
| **GIN / inverted index** | Bảng tra "từ → tài liệu chứa từ đó", như mục lục cuối sách |
| **Hybrid search / RRF** | Kết hợp từ khóa và ngữ nghĩa; hợp nhất bằng cộng `1/(60 + thứ_hạng)` |
| **`COPY`** | Lệnh nạp hàng loạt nhanh nhất; nhanh hơn `INSERT` từng dòng hàng chục lần |
| **Idempotent / upsert** | Chạy lại nhiều lần vẫn ra một kết quả; bảng tạm rồi cập nhật-hoặc-thêm |
| **Threshold (ngưỡng)** | Bộ lọc chất lượng; phụ thuộc thước đo và bộ dữ liệu; phải hiệu chỉnh |
| **Golden set** | Tập cặp đã gán nhãn để hiệu chỉnh ngưỡng và đo recall |
| **Re-embedding** | Đổi mô hình = sinh lại toàn bộ vector — quyết định nặng nhất của cả stack |
| **BYO vs in-DB embedding** | Tự sinh vector bên ngoài (linh hoạt) / để database sinh (tiện nhưng khóa chặt) |
| **Chunking** | Cắt tài liệu dài thành đoạn nhỏ trước khi embed |
| **Overfiltering** | Lọc mạnh cộng ANN làm thiếu kết quả; chữa bằng `iterative_scan` |
| **Blue-green** | Dựng song song bộ mới, kiểm chứng xong mới chuyển lưu lượng, giữ bộ cũ để quay lui |
| **Drift** | Phân bố khoảng cách dịch chuyển theo thời gian → ngưỡng cần hiệu chỉnh lại |
| **RAG** | Cho mô hình ngôn ngữ tra tài liệu thật trước khi trả lời, để giảm bịa đặt |

### 5.2. Mười hai ý cốt lõi — nếu chỉ nhớ ngần này

1. **Mô hình tạo vector; pgvector chỉ lưu và tìm** — nó không tự sinh embedding.
2. **Static so với contextual**: transformer thắng vì hiểu ngữ cảnh nhờ cơ chế attention.
3. **Cosine đo hướng, bỏ qua độ dài**; `<=>` trả **distance**, similarity = `1 - distance`.
4. **Không index = chính xác 100% nhưng `O(n)`**; có index = ANN, đổi độ chính xác lấy tốc độ.
5. **HNSW là mặc định** (recall cao, tốn RAM); IVFFlat nhẹ hơn nhưng cần sẵn dữ liệu.
6. **Không dùng B-tree cho vector được** — không có trục để sắp thứ tự, cộng lời nguyền số chiều.
7. **Siêu năng lực của pgvector: vector nằm cạnh dữ liệu nghiệp vụ** → hybrid query một câu SQL.
8. **Vector là tính năng → Postgres; là sản phẩm hoặc hàng tỷ vector → hệ chuyên dụng.**
9. **`COPY` nhanh nhất; nạp trước, index sau**; quy trình phải chạy lại được.
10. **Truy vấn: `ORDER BY <=> LIMIT k` (số lượng) cộng ngưỡng (chất lượng)**; nhớ loại chính nó; ngưỡng đứng một mình sẽ mất index.
11. **Hybrid = từ khóa cộng ngữ nghĩa, hợp nhất bằng RRF** — kiến trúc mặc định 2026.
12. **Đổi mô hình = làm lại toàn bộ vector** — quyết định nặng nhất; và **quyền riêng tư có thể quyết định thay bạn** bất kể benchmark nói gì.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Thư viện gắn tọa độ cho sách"** — giải thích embedding và semantic search trong 15 giây cho bất kỳ ai.
- **"Mục lục cuối sách"** — giải thích inverted index và GIN trong một câu.
- **"Distance là khoảng cách trên bản đồ, similarity là điểm số"** — 0 mét là đang đứng tại đó; điểm cao là tốt.
- **"Gần nhất không có nghĩa là đủ tốt"** — phân biệt `LIMIT` với ngưỡng.
- **"Xếp lại tủ sách một lần, không phải sau mỗi cuốn"** — vì sao nạp trước index sau.
- **"Vector là tính năng hay là sản phẩm?"** — kim chỉ nam chọn nền tảng.
- **"Tiện lợi đi kèm sợi xích"** — in-database embedding đổi tiện lợi lấy lock-in.
- **"Ngưỡng là đường biên quyết định, không phải con số phép màu"** — nó được đo, không được đoán.

### 5.4. Code thuộc lòng — cả vòng đời trong một khối

```sql
-- ═══ CÀI + SCHEMA + INDEX ═══
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE docs (
  id bigserial PRIMARY KEY,
  content text,
  embedding vector(1536)
);

CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);  -- SAU khi nạp


-- ═══ NẠP NHANH ═══
\copy docs (content, embedding) FROM 'data.csv' WITH (FORMAT csv, HEADER true)


-- ═══ TRUY VẤN: k-NN + ngưỡng + độ giống % ═══
WITH nearest AS MATERIALIZED (
  SELECT content, embedding <=> :q AS distance
  FROM docs ORDER BY distance LIMIT 20
)
SELECT content, 1 - distance AS similarity
FROM nearest WHERE distance < 0.3 ORDER BY distance LIMIT 5;


-- ═══ HYBRID: hợp nhất bằng RRF ═══
WITH kw AS (
  SELECT id, row_number() OVER (ORDER BY ts_rank(sv, q) DESC) AS r
  FROM docs, websearch_to_tsquery('english', :txt) q
  WHERE sv @@ q LIMIT 50
),
sem AS (
  SELECT id, row_number() OVER (ORDER BY embedding <=> :qv) AS r
  FROM docs ORDER BY embedding <=> :qv LIMIT 50
)
SELECT id, SUM(1.0 / (60 + r)) AS rrf
FROM (SELECT * FROM kw UNION ALL SELECT * FROM sem) t
GROUP BY id ORDER BY rrf DESC LIMIT 10;
```

```python
# ═══ TẠO EMBEDDING — nhớ dùng CÙNG mô hình cho cả kho lẫn câu truy vấn ═══
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")   # đổi mô hình = re-embed tất cả
vec = model.encode(text, normalize_embeddings=True)  # normalize để cosine ổn định
```

```sql
-- ═══ BA LỆNH CHẨN ĐOÁN PHẢI NHỚ ═══
EXPLAIN ANALYZE SELECT ...;              -- tìm "Index Scan"; thấy "Seq Scan" là có vấn đề
SET hnsw.ef_search = 100;                -- tăng recall, đổi lại chậm hơn
SET hnsw.iterative_scan = relaxed_order; -- chống overfiltering khi lọc mạnh
```

### 5.5. Mười hai câu phỏng vấn trải cả khóa

**1. "Embedding là gì?"**
> Vector mã hóa ý nghĩa, sao cho nội dung gần nghĩa cho vector gần nhau. **Ghi điểm thêm** bằng cách nêu phân biệt static (Word2Vec, mỗi từ một vector cố định) với contextual (transformer, vector đổi theo ngữ cảnh nhờ attention).

**2. "Cosine hay L2?"**
> Cosine đo góc, bỏ qua độ lớn — mặc định cho văn bản, và miền cố định `[0,2]` nên dễ đặt ngưỡng. L2 khi độ lớn có ý nghĩa. Nếu vector đã chuẩn hóa thì hai cái cho cùng thứ tự xếp hạng.

**3. [BẪY] "`<=>` trả về cosine similarity đúng không?"**
> Không — nó trả **distance**, nhỏ là giống. Similarity = `1 - (embedding <=> q)`. Hiển thị nhầm thì kết quả giống hệt sẽ hiện thành 0%.

**4. "HNSW so với IVFFlat?"**
> HNSW: đồ thị nhiều tầng, `~O(log n)`, recall cao, tốn RAM, xây chậm. IVFFlat: chia cụm, nhẹ, xây nhanh, nhưng cần sẵn dữ liệu để chia cụm và recall thấp hơn. Mặc định chọn HNSW.

**5. [BẪY] "Sao không dùng B-tree cho vector?"**
> Vì không có một trục nào để sắp thứ tự trong không gian nhiều chiều, cộng với lời nguyền số chiều khiến mọi điểm gần như cách đều nhau. Vì vậy ngành này buộc phải chấp nhận tìm gần đúng (ANN).

**6. "pgvector hay Pinecone?"**
> Kim chỉ nam: vector là **tính năng** hay là **sản phẩm**. Postgres thắng ở sự đơn giản trong vận hành và khả năng hybrid query một câu SQL; hệ chuyên dụng thắng ở quy mô đỉnh và phân tán nhiều vùng.

**7. [BẪY] "`tsvector` chính là vector search phải không?"**
> Không. `tsvector` là full-text search — lưu danh sách từ đã chuẩn hóa, tìm theo từ khóa. Semantic search cần kiểu `vector` của extension pgvector. Chỉ trùng chữ cái.

**8. "Nạp hàng loạt nhanh nhất bằng cách nào?"**
> `COPY` vào bảng **chưa có index**, rồi mới `CREATE INDEX`. Lý do: HNSW là đồ thị phải cập nhật sau mỗi lần chèn, nên xây một lượt sau khi nạp xong sẽ nhanh hơn nhiều. Và quy trình phải chạy lại được (bảng tạm rồi upsert).

**9. [BẪY] "`WHERE embedding <=> q < 0.3` có nhanh không?"**
> Không, nếu đứng một mình — index ANN chỉ tối ưu cho k-NN, còn lọc theo bán kính buộc tính khoảng cách mọi dòng. Phải kèm `ORDER BY ... LIMIT k`; cách chuẩn theo tài liệu pgvector là dùng CTE `MATERIALIZED` với điều kiện lọc đặt bên ngoài.

**10. "Đổi mô hình embedding thì sao?"**
> Phải **re-embed toàn bộ kho** — vector hai mô hình nằm trên hai không gian khác nhau. Nêu thêm: đánh phiên bản cho cột, chuyển đổi blue-green, và nhớ hiệu chỉnh lại ngưỡng vì ngưỡng gắn với mô hình.

**11. [SCALE] "Hàng tỷ vector mà RAM có hạn thì sao?"**
> Lượng tử hóa để giảm bộ nhớ; chuyển sang DiskANN qua pgvectorscale để chạy trên SSD; phân mảnh bảng; truy hồi hai tầng (ANN lấy rộng rồi tính chính xác trên tập nhỏ). Nếu còn cần đa vùng địa lý nữa thì lúc đó mới cân nhắc hệ chuyên dụng.

**12. "Hybrid search là gì và vì sao dùng RRF?"**
> Chạy cả nhánh từ khóa lẫn nhánh ngữ nghĩa rồi hợp nhất. Dùng RRF vì điểm của hai nhánh nằm trên hai thang đo khác nhau, không cộng trực tiếp được — nên cộng **nghịch đảo thứ hạng** thay vì cộng điểm. Tài liệu lọt top ở cả hai nhánh sẽ lên đầu.

### 5.6. Câu trả lời "one-liner" đắt giá

- *"The model makes the vector, the database stores it — pgvector never embeds anything itself."*
- *"You can't binary-search vectors: there's no single axis to sort along, so the whole field approximates instead."*
- *"pgvector wins on operational simplicity, not peak speed — pick it when vector search is a feature, not the product."*
- *"COPY into an unindexed table, then build the index — never make Postgres maintain an HNSW graph mid-load."*
- *"LIMIT caps how many; the threshold guarantees how good — and a bare threshold quietly drops the index."*
- *"Changing the embedding model means re-embedding everything — it's the heaviest decision in the entire stack."*
- *"FTS knows words, vectors know meaning — hybrid is both, fused with RRF, inside one Postgres."*
- *"Recall failures are silent: nothing errors, nothing slows down, the results just quietly get worse."*
- *"Sometimes privacy picks the architecture before performance even enters the conversation."*

---

## 📌 Ghi chú cuối

**Bạn đã đi hết một chặng đường dài.** Từ "vector là gì" tới "thiết kế và vận hành một hệ tìm kiếm ngữ nghĩa chạy thật trên PostgreSQL". Chín file giáo trình phủ trọn: tạo vector → đánh index → chọn nền tảng → thiết kế lưu trữ → cài đặt → nạp dữ liệu → truy vấn → hybrid → tổng hợp.

**Cách ôn hiệu quả nhất:** đọc Phần 0 để lấy lại bản đồ tổng, đọc Phần 3 để nắm những sợi chỉ xuyên suốt (đây là phần phân biệt bạn với ứng viên khác), rồi dùng Phần 5 để nước rút trước phỏng vấn. Chỉ mở tám file kia ra khi cần đào sâu một chỗ cụ thể.

**Kiểm chứng lại trước khi phỏng vấn.** Mảng này đổi rất nhanh — đặc biệt là **danh sách mô hình embedding**, vốn thay đổi gần như hàng quý. Trước buổi phỏng vấn, hãy xem lại README của pgvector (cú pháp, ops class, các mẫu truy vấn khuyến nghị), bảng xếp hạng MTEB hoặc MMTEB (mô hình), và tài liệu Postgres về full-text search. Điều **không** đổi là cách tư duy: đường ống tám trạm, các đánh đổi, và khung quyết định.

**Việc đáng làm nhất sau khóa học này:** xây dự án capstone ở mục 4.3 và đưa lên một kho mã công khai. Một repo "tìm kiếm ngữ nghĩa và hybrid" với README bàn về **đánh đổi và giới hạn** là minh chứng kỹ năng thuyết phục hơn mọi chứng chỉ — vì nó cho thấy bạn không chỉ làm cho chạy được, mà hiểu vì sao mình chọn như vậy và khi nào thì nên chọn khác.

**Học tiếp gì sau khóa này:**
- **RAG đầy đủ từ đầu tới cuối** — truy hồi, xếp hạng lại, rồi để mô hình ngôn ngữ tổng hợp câu trả lời.
- **Reranking bằng cross-encoder** — một mô hình chính xác hơn chấm lại top-N sau khi truy hồi, nâng đáng kể chất lượng ở đỉnh danh sách.
- **Chiến lược chunking** — cắt tài liệu ra sao ảnh hưởng tới chất lượng tìm kiếm nhiều hơn người ta tưởng.
- **Đánh giá truy hồi có hệ thống** — precision@k, recall@k, nDCG, MRR trên golden set.
- **Embedding đa phương thức** — một không gian vector chung cho cả chữ, ảnh, âm thanh và video.
- **Phân tán vector qua nhiều máy** — Citus hoặc pgvectorscale khi vượt quá khả năng một máy chủ.

**Chúc mừng bạn đã hoàn thành.**
