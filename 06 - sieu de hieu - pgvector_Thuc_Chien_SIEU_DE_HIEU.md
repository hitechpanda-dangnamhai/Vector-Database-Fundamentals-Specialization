# Cài & Dùng pgvector Thực Chiến
## Giáo trình SIÊU DỄ HIỂU — Basic → Staff Engineer

> **Nguồn gốc:** Video *"pgvector extension for PostgreSQL"* — Course 3, IBM Vector Database Fundamentals. Bài gốc dạy phần tay chân: cài đặt trên Linux, tạo bảng có cột vector, insert, query bằng `<=>`, và kết nối từ Node.js.
>
> **Bài này khác bài pgvector lý thuyết ở chỗ nào:** Bài lý thuyết dạy *tại sao và cơ chế* (HNSW hoạt động ra sao, đánh đổi giữa tốc độ và độ chính xác). Bài này dạy *gõ lệnh nào, gõ ở đâu, gõ xong thì sao*. Đây là bài thực hành.
>
> **Cách đọc:** Tôi đi rất chậm và giải thích mọi thuật ngữ ngay lần đầu xuất hiện — kể cả những thứ dân trong nghề coi là hiển nhiên (terminal, `sudo`, Docker, port, schema...). Nếu bạn đã biết, cứ đọc lướt.
>
> **Quy ước ký hiệu:**
> - 🧩 **[Ngoài bài gốc]** = phần tôi bổ sung, video không nói, nhưng cần cho công việc thật.
> - ⚠️ **[Đính chính bài gốc]** = chỗ video nói chưa chính xác hoặc đã cũ.
> - 💡 = ý quan trọng cần khắc cốt.

---

## ⚠️ Bảy điểm cần đính chính & cập nhật (tra cứu ngày 23/07/2026)

Video gốc quay đã lâu và có vài chỗ nay không còn đúng. Tôi liệt kê trước để bạn khỏi học nhầm. Mỗi điểm đều được giải thích kỹ ở phần tương ứng bên dưới.

**Một — PostgreSQL 12 không còn đủ.** Video nói cần PostgreSQL 12 trở lên. README chính thức hiện ghi rõ pgvector hỗ trợ **PostgreSQL 13 trở lên**. Thực tế nên dùng 16, 17 hoặc 18 vì đó là những bản có gói cài sẵn và bản vá đầy đủ.

**Hai — Cách cài mà video dạy là cách KHÓ NHẤT.** Video dạy tải mã nguồn về rồi tự biên dịch (`git clone` + `make`). Cách này vẫn đúng, nhưng năm 2026 có ba cách dễ hơn hẳn: **Docker**, **gói hệ điều hành** (`apt install`), và **dịch vụ có sẵn** (Supabase, Neon, AWS RDS đã cài sẵn cho bạn). Người mới nên bắt đầu bằng Docker.

**Ba — `<=>` là cosine *distance*, không phải cosine *similarity*.** Video gọi nhầm. Hai thứ này **ngược chiều nhau**: similarity càng cao càng giống, distance càng thấp càng giống. Gọi nhầm tên dẫn tới sắp xếp ngược và ra kết quả sai hoàn toàn. Tôi giải thích kỹ ở Mục 1.8.

**Bốn — Tên hàm trong thư viện Node.js đã đổi.** Video (và nhiều tài liệu cũ) dùng `pgvector.registerType(client)` — số ít. README chính thức hiện dùng **`registerTypes(client)`** — **số nhiều**. Gõ tên cũ sẽ báo lỗi "không phải là hàm".

**Năm — Phiên bản hiện tại là pgvector 0.8.3.** Và có một điều quan trọng về bảo mật: bản **0.8.2 (tháng 2/2026) vá một lỗi tràn bộ nhớ đệm** trong quá trình xây index HNSW song song (mã lỗi CVE-2026-3172). Lỗi này có thể làm rò rỉ dữ liệu từ bảng khác hoặc làm sập server. Nếu bạn đang chạy bản cũ hơn 0.8.2, hãy nâng cấp. Tôi nói kỹ ở Mục 4.3.

**Sáu — pgvector có SÁU toán tử khoảng cách, không phải ba.** Video và nhiều tài liệu chỉ nhắc `<->`, `<=>`, `<#>`. Thực tế còn `<+>` (khoảng cách taxi), `<~>` và `<%>` (dành cho vector nhị phân). Bảng đầy đủ ở Mục 2.4.

**Bảy — Từ bản 0.8.0 có tính năng "iterative index scan" giải quyết một vấn đề rất đau.** Đó là chuyện lọc kết hợp tìm kiếm vector cho ra quá ít kết quả. Video không hề nhắc vì tính năng này ra sau. Đây là kiến thức thực chiến quan trọng, tôi dành hẳn Mục 3.4 cho nó.

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)

### Bài này dạy gì, nói bằng một câu

**Bài này dạy bạn cách biến một database PostgreSQL bình thường thành nơi cất giữ và tìm kiếm được các dãy số biểu diễn ý nghĩa — từ lúc cài đặt, tới lúc gõ câu lệnh đầu tiên, tới lúc kết nối được từ ứng dụng của bạn.**

Nếu bạn thấy cụm "dãy số biểu diễn ý nghĩa" hơi mơ hồ, đó là **embedding** — chủ đề của một bài khác trong series. Ở bài này bạn chỉ cần biết: **embedding là một dãy số dài, ví dụ 384 hoặc 1536 con số, do một model AI sinh ra để đại diện cho ý nghĩa của một câu chữ.** Hai câu giống nghĩa thì hai dãy số của chúng "gần nhau". Chỉ cần nhớ chừng đó là đủ theo hết bài này.

### Nó giải quyết nỗi đau gì

Hiểu lý thuyết là một chuyện. **Cài được và chạy được là chuyện hoàn toàn khác** — và đây là chỗ người mới tắc nhiều nhất.

Những câu hỏi kiểu này không có trong sách lý thuyết, nhưng chúng là thứ chặn bạn lại lúc 11 giờ đêm:

- Tôi gõ `CREATE EXTENSION vector` thì báo `could not open extension control file`. Nghĩa là sao?
- Tôi cài xong rồi mà vẫn báo `type "vector" does not exist`. Cài kiểu gì nữa?
- Tôi tạo index rồi mà truy vấn vẫn chậm như chưa tạo. Index có chạy không?
- App Node.js của tôi nhận về một chuỗi ký tự thay vì mảng số. Sai ở đâu?

Bài này gỡ từng nút một.

### Học xong bạn sẽ làm được

- Chọn đúng cách cài pgvector cho hoàn cảnh của mình, trong bốn cách, và biết vì sao.
- Cài được, kiểm chứng được là nó thực sự chạy, và hiểu vì sao "cài file" khác "bật extension".
- Tạo bảng có cột vector, thêm dữ liệu, và viết câu truy vấn tìm k bản ghi giống nhất.
- Đọc được kết quả `EXPLAIN ANALYZE` để biết index có thực sự được dùng hay không — kỹ năng phân biệt người biết dùng với người chỉ biết gõ theo hướng dẫn.
- Kết nối pgvector từ Node.js và Python đúng cách, không bị lỗi kiểu dữ liệu.
- Triển khai pgvector lên một database production đang phục vụ khách hàng mà không gây gián đoạn.

### Mạch kiến thức

- 🟢 **Basic** — Cài bằng Docker (dễ nhất) → bật extension → kiểm chứng → tạo bảng → thêm dữ liệu → truy vấn đầu tiên, kèm ví dụ tính tay để hiểu vì sao sắp xếp tăng dần.
- 🟡 **Intermediate** — Bốn cách cài và chọn cái nào khi nào → sáu toán tử khoảng cách → đánh index → kết nối từ Node.js và Python → ba lỗi cài đặt kinh điển.
- 🔴 **Advanced** — Bên trong quá trình biên dịch → `EXPLAIN ANALYZE` và quy tắc chính xác để index được dùng → vấn đề lọc kết hợp tìm kiếm vector và cách giải → nén vector để tiết kiệm bộ nhớ → nâng cấp extension → các trường hợp biên.
- 🟣 **Staff** — Vì sao tuyệt đối không biên dịch tay trên production → triển khai không gián đoạn → bảo mật, phân quyền, vá lỗi → kiểm thử tự động → nói chuyện với sếp → một câu hỏi vận hành mẫu.
- 🎯 **Cheatsheet** — Bảng từ khoá, ý cốt lõi, lệnh thuộc lòng, câu hỏi phỏng vấn.

---

## Phần 1 — 🟢 BASIC (Nền tảng)

> Phần này viết cho người **chưa từng cài extension nào cho PostgreSQL**, thậm chí chưa chắc đã dùng terminal nhiều. Nếu bạn đã quen, đọc lướt — nhưng đừng bỏ Mục 1.8, vì đó là chỗ dễ hiểu sai nhất trong toàn bài.

### 1.1. Nỗi đau: bạn đã có vector rồi, giờ cất ở đâu?

Giả sử bạn vừa học xong bài về embedding. Bạn viết được một đoạn code gọi model AI, đưa vào câu *"áo thun cotton nam"*, và nhận về một mảng 384 con số:

```
[0.021, -0.087, 0.044, 0.113, ..., -0.009]
```

Tuyệt. Nhưng giờ bạn có **10.000 sản phẩm**. Bạn cần cất 10.000 mảng số này ở đâu đó, rồi khi khách hàng gõ tìm kiếm, bạn phải tìm ra mảng nào gần nhất với mảng của câu tìm kiếm.

Cách ngây thơ nhất: lưu vào một file, đọc hết vào bộ nhớ, so sánh từng cái. Cách này chết ngay khi dữ liệu lớn, và tệ hơn nữa là **dữ liệu của bạn giờ nằm ở hai nơi** — thông tin sản phẩm (tên, giá, tồn kho) nằm trong database, còn vector nằm trong file. Muốn tìm "áo giống cái này, giá dưới 300 nghìn, còn hàng" thì bạn phải ghép thủ công hai nguồn. Cực kỳ phiền và dễ sai.

Lựa chọn thứ hai: dùng một **vector database** chuyên dụng (Pinecone, Weaviate, Qdrant...). Chúng làm việc này rất tốt. Nhưng bạn vừa thêm một hệ thống hoàn toàn mới vào kiến trúc — thêm một thứ phải cài, phải giám sát, phải trả tiền, phải đồng bộ với database chính, và phải có người trong đội biết vận hành.

Và đây là câu hỏi rất hợp lý: **"Tôi đã có PostgreSQL rồi. Sao không cất vector luôn vào đó?"**

Đó chính xác là điều pgvector cho phép bạn làm.

> **PostgreSQL (thường gọi tắt là Postgres):** một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở, miễn phí, cực kỳ phổ biến. "Quan hệ" nghĩa là nó lưu dữ liệu dưới dạng các **bảng** (table) gồm **hàng** (row — mỗi hàng là một bản ghi, ví dụ một sản phẩm) và **cột** (column — mỗi cột là một thuộc tính, ví dụ tên, giá).
>
> **Database (cơ sở dữ liệu):** trong Postgres, một server có thể chứa nhiều database tách biệt nhau. Ví dụ database `shop_production` và database `shop_test` nằm trên cùng một server nhưng hoàn toàn độc lập. Chi tiết này sẽ rất quan trọng ở Mục 1.5.

### 1.2. pgvector là gì

> **Extension (phần mở rộng):** một gói tính năng cắm thêm vào PostgreSQL để dạy cho nó làm việc mà nó vốn không biết làm. Postgres được thiết kế ngay từ đầu để mở rộng được — bạn có thể thêm kiểu dữ liệu mới, hàm mới, toán tử mới, thậm chí loại index mới, mà không cần sửa mã nguồn của Postgres.
>
> **pgvector:** extension dạy cho PostgreSQL ba thứ. Một, một **kiểu dữ liệu mới** tên là `vector` để lưu mảng số. Hai, các **toán tử** để đo khoảng cách giữa hai vector. Ba, các **loại index** chuyên dụng để tìm vector gần nhất thật nhanh.

**Analogy: pgvector giống như lắp một cái máy pha cà phê vào căn bếp có sẵn của bạn.**

Hãy đi hết phép ví von này, vì nó khớp ở nhiều điểm hơn bạn nghĩ.

Bạn đã có một căn bếp đầy đủ (PostgreSQL): có bếp nấu, tủ lạnh, bồn rửa, và bạn đã quen dùng nó hàng ngày. Giờ bạn muốn pha cà phê espresso. Bạn có hai lựa chọn: **xây một quán cà phê riêng** ở phòng bên cạnh (dựng một vector database chuyên dụng — mạnh hơn, nhưng tốn kém và phải quản lý thêm một không gian mới), hoặc **lắp thêm một cái máy pha vào bếp sẵn có** (cài pgvector).

Phép ví von khớp ở bốn điểm:

| Trong bếp | Trong kỹ thuật |
|---|---|
| Máy pha cà phê dùng chung điện nước với cả bếp | pgvector dùng chung mọi hạ tầng của Postgres: sao lưu, phân quyền, giao dịch |
| Bạn pha cà phê rồi rót vào cốc lấy từ tủ bếp, không cần chạy sang phòng khác | Bạn `JOIN` bảng vector với bảng sản phẩm trong cùng một câu lệnh |
| Máy pha rất giỏi việc pha cà phê, nhưng không thay được cả một quán chuyên nghiệp | pgvector đủ tốt cho phần lớn nhu cầu, nhưng ở quy mô cực lớn thì hệ chuyên dụng vẫn hơn |
| Mua máy về vẫn phải cắm điện mới chạy | Cài file vẫn phải bật extension mới dùng được — Mục 1.3 |

Lợi ích lớn nhất, và cũng là lý do pgvector phổ biến đến vậy, nằm ở dòng thứ hai: **vector nằm cùng chỗ với dữ liệu còn lại của bạn.** Câu truy vấn "tìm áo giống cái này, giá dưới 300 nghìn, còn hàng, thuộc shop đang hoạt động" viết được trong **một câu SQL duy nhất**, thay vì phải gọi hai hệ thống rồi tự ghép kết quả.

> **SQL (Structured Query Language):** ngôn ngữ tiêu chuẩn để ra lệnh cho database quan hệ. Câu lệnh SQL đọc gần như tiếng Anh: `SELECT tên FROM sản_phẩm WHERE giá < 300000` nghĩa là "lấy cột tên, từ bảng sản phẩm, với điều kiện giá nhỏ hơn 300000".
>
> **JOIN:** phép ghép dữ liệu từ nhiều bảng trong một câu truy vấn. Đây là thứ database quan hệ giỏi nhất, và là lợi thế lớn của pgvector so với vector database chuyên dụng.

💡 **Câu quan trọng nhất, sẽ nhắc lại nhiều lần trong bài:**

> **pgvector KHÔNG tạo ra embedding. Nó chỉ lưu và tìm.**
>
> Việc tạo ra dãy số là của model AI, hoàn toàn tách biệt. Bạn gọi model, lấy mảng số, rồi mới đưa mảng số đó cho pgvector cất giữ. Model là đầu bếp, pgvector là tủ lạnh.

Đây là hiểu nhầm phổ biến nhất của người mới, và cũng là một câu hỏi phỏng vấn hay gặp.

### 1.3. Điều quan trọng nhất Phần 1: cài file ≠ bật extension

Nếu bạn chỉ nhớ **một** điều từ toàn bộ Phần 1, hãy nhớ điều này. Nó là nguyên nhân của khoảng một nửa số câu hỏi "tôi cài rồi mà không chạy" trên các diễn đàn.

**Việc đưa pgvector vào dùng được gồm ĐÚNG HAI BƯỚC, hoàn toàn tách biệt:**

```
BƯỚC 1: CÀI FILE vào server
   → Chép các file của pgvector vào đúng thư mục mà PostgreSQL biết tìm.
   → Làm ở tầng HỆ ĐIỀU HÀNH (gõ lệnh trong terminal).
   → Làm MỘT LẦN cho cả server.
   → Sau bước này, Postgres mới chỉ "có sẵn file", chưa dùng được gì.

BƯỚC 2: BẬT EXTENSION trong database
   → Gõ lệnh SQL: CREATE EXTENSION vector;
   → Làm ở tầng DATABASE (gõ trong Postgres).
   → Làm RIÊNG cho TỪNG database muốn dùng.
   → Sau bước này mới thực sự dùng được.
```

**Vì sao lại tách làm hai bước?** Vì một server Postgres có thể chứa nhiều database, và không phải database nào cũng cần tính năng vector. Postgres để bạn tự quyết định bật ở đâu, thay vì tự động bật khắp nơi.

**Analogy: giống như cài một ứng dụng lên điện thoại.** Bước 1 là tải app từ cửa hàng về máy — giờ app đã nằm trong máy. Bước 2 là mở app ra và đăng nhập vào tài khoản. Tải về xong mà không mở thì app vẫn nằm im, không làm gì cho bạn cả.

💡 **Triệu chứng khi quên bước 2:** bạn cài xong, tự tin gõ `CREATE TABLE ... embedding vector(384)`, và nhận:

```
ERROR: type "vector" does not exist
```

Postgres đang nói thật lòng: trong database *này*, nó chưa biết `vector` là cái gì. File đã có trên server, nhưng bạn chưa bật nó lên trong database này.

### 1.4. Cách cài dễ nhất: Docker

⚠️ **[Đính chính bài gốc]** Video dạy cách tải mã nguồn về tự biên dịch. Với người mới, đó là cách dễ hỏng nhất và cho ít giá trị học tập nhất — bạn sẽ dành hai tiếng vật lộn với lỗi biên dịch thay vì học pgvector. Tôi dạy Docker trước; cách của video được giải thích đầy đủ ở Mục 2.2 cho khi bạn thực sự cần.

> **Docker:** công cụ đóng gói một phần mềm cùng **toàn bộ** những gì nó cần để chạy — hệ điều hành, thư viện, cấu hình — vào một gói duy nhất. Gói đó chạy giống hệt nhau trên mọi máy. Docker sinh ra để giết chết câu nói kinh điển "nhưng máy tôi chạy được mà".
>
> **Image (ảnh):** cái gói đó, ở trạng thái tĩnh — giống một file cài đặt.
>
> **Container:** một bản đang chạy của image — giống một chương trình đang mở. Từ một image bạn có thể chạy nhiều container.
>
> **Tag (nhãn):** phần sau dấu hai chấm trong tên image, cho biết phiên bản. Trong `pgvector/pgvector:pg18`, phần `pg18` là tag, nghĩa là "bản dành cho PostgreSQL 18".

Điều tuyệt vời: **dự án pgvector phát hành sẵn image Docker có Postgres và pgvector đã cài đặt hoàn chỉnh.** Bạn không phải cài gì cả. Một lệnh là xong.

```bash
docker run -d \
  --name pgv \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  pgvector/pgvector:pg18
```

Đọc từng phần của lệnh này — bạn sẽ gặp lại chúng suốt sự nghiệp:

| Phần | Nghĩa là gì |
|---|---|
| `docker run` | Tạo và khởi động một container mới |
| `-d` | Viết tắt của *detached* — chạy ngầm ở nền, trả lại quyền điều khiển terminal cho bạn. Không có nó, terminal sẽ bị "khoá" bởi log của Postgres |
| `--name pgv` | Đặt tên `pgv` cho container, để sau này gọi tên thay vì phải nhớ mã số dài |
| `-e POSTGRES_PASSWORD=secret` | `-e` là *environment variable* (biến môi trường) — truyền cấu hình vào bên trong container. Đây là mật khẩu của tài khoản quản trị `postgres`. **Trong công việc thật, đừng dùng `secret`** |
| `-p 5432:5432` | Mở **cổng** (port). Số bên trái là cổng trên máy bạn, bên phải là cổng bên trong container. 5432 là cổng mặc định của Postgres |
| `pgvector/pgvector:pg18` | Tên image và tag phiên bản |

> **Port (cổng):** một con số dùng để phân biệt các dịch vụ mạng chạy trên cùng một máy. Nếu địa chỉ IP là số nhà thì port là số phòng. Postgres theo quy ước ở phòng số 5432.
>
> **Terminal / shell / dòng lệnh:** cửa sổ đen nơi bạn gõ lệnh cho máy tính bằng chữ thay vì bấm chuột. Trên macOS là app "Terminal", trên Windows là PowerShell hoặc WSL, trên Linux là bất kỳ ứng dụng terminal nào.

**Bước tiếp theo: vào bên trong để gõ lệnh SQL.**

```bash
docker exec -it pgv psql -U postgres
```

Giải nghĩa: `docker exec` là "chạy một lệnh bên trong container đang chạy". `-it` gồm `-i` (interactive — cho phép bạn gõ vào) và `-t` (terminal — cấp một terminal giả để giao diện hiển thị đẹp). `pgv` là tên container. `psql -U postgres` là lệnh cần chạy bên trong.

> **psql:** công cụ dòng lệnh chính thức của PostgreSQL — nơi bạn gõ lệnh SQL và xem kết quả. `-U postgres` nghĩa là đăng nhập bằng tài khoản tên `postgres`, tài khoản quản trị mặc định.

Nếu thành công, bạn thấy dấu nhắc:

```
postgres=#
```

Dấu `#` ở cuối cho biết bạn đang đăng nhập với quyền quản trị cao nhất. (Người dùng thường sẽ thấy dấu `>` thay vì `#`.) Bạn đã sẵn sàng.

💡 **Ba lệnh Docker nên nhớ ngay từ đầu:**

```bash
docker ps                # xem các container đang chạy
docker stop pgv          # tạm dừng (dữ liệu vẫn còn nguyên)
docker start pgv         # chạy lại
docker rm -f pgv         # XOÁ container — dữ liệu bên trong MẤT HẾT
```

⚠️ Lệnh cuối cùng xoá sạch dữ liệu. Với việc học thì không sao — chạy lại `docker run` là có môi trường mới tinh. Với dữ liệu bạn muốn giữ, cần gắn thêm ổ đĩa lưu trữ bền vững (`-v`), là chủ đề của Mục 4.1.

### 1.5. Bật extension và kiểm chứng nó thật sự chạy

Bạn đang ở dấu nhắc `postgres=#`. Giờ là bước 2 trong hai bước ở Mục 1.3.

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Ba điều cần chú ý trong một dòng ngắn này.

**Thứ nhất, `IF NOT EXISTS` là thói quen tốt cần tập ngay.** Nó nghĩa là "nếu chưa có thì tạo, có rồi thì bỏ qua, đừng báo lỗi". Nhờ vậy chạy lại lệnh này lần thứ hai, thứ mười cũng không sao.

> **Idempotent (bất biến khi lặp):** tính chất của một thao tác mà chạy một lần hay chạy mười lần đều cho cùng kết quả. Đây là tính chất rất được coi trọng khi viết script triển khai tự động, vì script có thể bị chạy lại do lỗi mạng, do thử lại, hoặc do người khác vô tình chạy. Từ này sẽ quay lại ở Phần 4.

**Thứ hai, tên extension là `vector`, không phải `pgvector`.** Tên dự án trên GitHub là pgvector, nhưng tên extension bên trong Postgres là `vector`. Gõ `CREATE EXTENSION pgvector` sẽ báo lỗi không tìm thấy. Đây là một chỗ vấp rất phổ biến.

**Thứ ba, lệnh này cần quyền cao.** Bạn phải là superuser hoặc có quyền `CREATE` trên database.

> **Superuser:** tài khoản quản trị tối cao trong Postgres, làm được mọi thứ và bỏ qua mọi kiểm tra phân quyền. Tương đương `root` trên Linux. Trong Docker, tài khoản `postgres` mặc định là superuser. Trong production thì không nên để ứng dụng chạy bằng superuser — Mục 4.3.

**Kiểm chứng — đừng bao giờ bỏ bước này.**

Cách thứ nhất, dùng lệnh tắt của psql:

```
\dx vector
```

> Các lệnh bắt đầu bằng dấu `\` (như `\dx`, `\dt`, `\q`) là **lệnh tắt riêng của psql**, không phải SQL. `\dx` là *describe extensions*. `\dt` liệt kê bảng. `\q` là thoát.

Kết quả mong đợi:

```
                          List of installed extensions
  Name  | Version | Schema |                     Description
--------+---------+--------+------------------------------------------------------
 vector | 0.8.3   | public | vector data type and ivfflat and hnsw access methods
```

Cách thứ hai, dùng SQL thuần (dùng được từ code, không chỉ từ psql):

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname = 'vector';
```

> **`pg_extension`:** một bảng hệ thống — bảng do Postgres tự duy trì để ghi lại thông tin về chính nó. Postgres có hàng chục bảng như vậy, tất cả đều bắt đầu bằng `pg_`. Bạn đọc được nhưng không nên sửa.

Nếu thấy `vector | 0.8.3`, cả hai bước đã xong. Nếu ra kết quả rỗng, extension chưa được bật trong database này.

💡 **Nhắc lại điều dễ quên nhất:** `CREATE EXTENSION` chỉ tác dụng trong **database hiện tại**. Nếu bạn có ba database và cả ba đều cần vector, bạn phải chạy lệnh này **ba lần**, mỗi lần sau khi đã kết nối vào đúng database đó. Đổi database trong psql bằng `\c tên_database`.

### 1.6. Tạo bảng có cột vector

```sql
CREATE TABLE items (
    id        bigserial PRIMARY KEY,
    content   text,
    embedding vector(384)
);
```

Đọc từng dòng:

> **`bigserial`:** kiểu số nguyên lớn **tự động tăng**. Mỗi khi bạn thêm một hàng mới mà không chỉ định `id`, Postgres tự gán 1, 2, 3... Bạn không phải tự quản lý.
>
> **`PRIMARY KEY` (khoá chính):** cột định danh duy nhất cho mỗi hàng. Postgres đảm bảo không có hai hàng trùng giá trị và không hàng nào để trống.
>
> **`text`:** kiểu chuỗi ký tự, độ dài tuỳ ý. Trong Postgres, `text` không chậm hơn `varchar(n)` — cứ dùng `text` trừ khi bạn thực sự cần giới hạn độ dài.
>
> **`vector(384)`:** đây là kiểu dữ liệu mới mà pgvector vừa dạy cho Postgres. Số 384 là **số chiều** — cột này chỉ chấp nhận vector có đúng 384 con số.

⚠️ **Con số trong ngoặc phải khớp CHÍNH XÁC với model embedding của bạn.** Đây là điểm mà video gốc nhấn mạnh, và nhấn mạnh đúng.

| Model bạn dùng | Phải khai báo |
|---|---|
| `all-MiniLM-L6-v2` | `vector(384)` |
| TensorFlow Universal Sentence Encoder | `vector(512)` |
| OpenAI `text-embedding-3-small` | `vector(1536)` |
| OpenAI `text-embedding-3-large`, Google Gemini Embedding | `vector(3072)` |

Nhét vector 384 chiều vào cột `vector(512)` sẽ báo lỗi ngay lập tức:

```
ERROR: expected 512 dimensions, not 384
```

💡 Thật ra lỗi báo ngay như thế này là **điều tốt** — nó chặn dữ liệu sai từ cửa. Loại lỗi đáng sợ hơn nhiều là loại im lặng cho qua rồi làm hỏng kết quả về sau.

🧩 **[Ngoài bài gốc] — Có một cách khai báo không cố định số chiều.** README chính thức có nhắc: bạn dùng `vector` không kèm số:

```sql
CREATE TABLE embeddings (
    model_id  bigint,
    item_id   bigint,
    embedding vector,          -- không ghi số chiều -> chấp nhận mọi độ dài
    PRIMARY KEY (model_id, item_id)
);
```

Cách này hữu ích khi bạn cần lưu vector từ **nhiều model khác nhau** trong cùng một bảng — ví dụ khi đang chuyển đổi từ model cũ sang model mới. Nhưng có cái giá: bạn **không thể đánh index thẳng lên cột đó**, vì index cần biết trước số chiều. Bạn phải dùng index có điều kiện cho từng model. Với người mới, hãy cứ dùng `vector(n)` cố định.

### 1.7. Thêm dữ liệu

```sql
INSERT INTO items (content, embedding) VALUES
  ('thủ đô của nước Anh là London',        '[0.12, -0.45, 0.88, ...]'),
  ('đường sắt Đức tên là Deutsche Bahn',   '[0.91, 0.02, -0.31, ...]');
```

Điểm cần chú ý về cú pháp: **vector được viết như một chuỗi ký tự, đặt trong dấu nháy đơn, các số cách nhau bằng dấu phẩy, bọc trong ngoặc vuông.**

```
'[0.12, -0.45, 0.88]'
 ↑                  ↑
 nháy đơn của SQL, bắt buộc
```

Postgres tự nhận ra đây là kiểu `vector` nhờ ngữ cảnh (cột đích có kiểu `vector`) và tự chuyển đổi.

Tất nhiên **trong thực tế bạn không gõ tay 384 con số.** Model sinh ra chúng, code của bạn truyền vào. Mục 2.5 và 2.6 chỉ cách làm việc đó từ Node.js và Python.

### 1.8. Truy vấn đầu tiên — và chỗ dễ hiểu sai nhất toàn bài

Đây là câu lệnh trung tâm của cả bài:

```sql
SELECT content
FROM items
ORDER BY embedding <=> '[0.11, -0.44, 0.90, ...]'
LIMIT 1;
```

Đọc thành lời: *"Lấy cột `content` từ bảng `items`, sắp xếp theo khoảng cách cosine giữa cột `embedding` và vector tôi đưa vào, rồi trả về 1 hàng đầu tiên."*

Kết quả: câu có ý nghĩa gần nhất với vector tìm kiếm. Nếu vector tìm kiếm là của câu *"nước Anh có thủ đô là gì"*, bạn nhận về *"thủ đô của nước Anh là London"*.

Bây giờ là phần quan trọng nhất.

⚠️ **[Đính chính bài gốc] — `<=>` KHÔNG phải cosine similarity.**

Video gốc gọi `<=>` là "cosine similarity". Sai. README chính thức của pgvector ghi rõ nó là **cosine distance** — khoảng cách cosin. Và hai thứ này **ngược chiều nhau**:

```
cosine_distance = 1 − cosine_similarity
```

| | Ý nghĩa | Giống nhau nhất khi | Miền giá trị |
|---|---|---|---|
| **Cosine similarity** | Độ *giống nhau* | Giá trị **CAO** (= 1) | −1 đến 1 |
| **Cosine distance** (`<=>`) | *Khoảng cách* | Giá trị **THẤP** (= 0) | 0 đến 2 |

**Analogy để không bao giờ nhầm nữa:** hãy nghĩ về hai người bạn. "Độ thân thiết" là **similarity** — càng cao càng thân. "Khoảng cách nhà" là **distance** — càng gần càng thân. Cùng mô tả một mối quan hệ, nhưng thang đo ngược nhau.

💡 **Hệ quả thực tế:** vì `<=>` trả về *khoảng cách*, ta sắp xếp **tăng dần** để lấy cái giống nhất. `ORDER BY` trong SQL mặc định đã là tăng dần (`ASC`), nên bạn không cần viết gì thêm. Nhưng nếu bạn *tưởng* nó là similarity và viết thêm `DESC`, bạn sẽ nhận được đúng **những kết quả khác biệt nhất** — hoàn toàn ngược ý muốn. Và tệ hơn: câu lệnh vẫn chạy, không báo lỗi gì. Bạn chỉ đơn giản nhận kết quả rác.

**Ví dụ chạy tay — để bạn tự tay kiểm chứng chiều sắp xếp.**

Tôi dùng vector 2 chiều để tính được bằng bút giấy. Công thức:

```
cosine_similarity(A, B) = (A · B) / (|A| × |B|)
cosine_distance(A, B)   = 1 − cosine_similarity(A, B)
```

Giả sử trong bảng có ba hàng, và vector tìm kiếm là **Q = (1, 0)**:

| Hàng | Nội dung | Vector |
|---|---|---|
| 1 | "mèo con" | A = (2, 0) |
| 2 | "chó con" | B = (1, 1) |
| 3 | "xe tải" | C = (0, 3) |

**Tính cho hàng 1 (A):**
```
A · Q = 2×1 + 0×0 = 2
|A|   = sqrt(2² + 0²) = 2
|Q|   = sqrt(1² + 0²) = 1
similarity = 2 / (2 × 1) = 1.0
distance   = 1 − 1.0 = 0.0        ← khoảng cách BẰNG KHÔNG
```

**Tính cho hàng 2 (B):**
```
B · Q = 1×1 + 1×0 = 1
|B|   = sqrt(1 + 1) = 1.414
similarity = 1 / (1.414 × 1) = 0.707
distance   = 1 − 0.707 = 0.293
```

**Tính cho hàng 3 (C):**
```
C · Q = 0×1 + 3×0 = 0
|C|   = sqrt(0 + 9) = 3
similarity = 0 / (3 × 1) = 0.0
distance   = 1 − 0.0 = 1.0        ← khoảng cách LỚN NHẤT
```

**Bảng kết quả:**

| Nội dung | similarity | **distance (`<=>`)** | Giống nhất? |
|---|---|---|---|
| mèo con | 1.000 | **0.000** | ✅ Nhất |
| chó con | 0.707 | **0.293** | Nhì |
| xe tải | 0.000 | **1.000** | Bét |

Nhìn cột `distance`: **giống nhất có số nhỏ nhất.** Nên khi bạn `ORDER BY ... <=> ...` (tăng dần), "mèo con" lên đầu. Chính xác điều bạn muốn.

Nếu bạn thêm `DESC`, "xe tải" sẽ lên đầu. Đó là toàn bộ tác hại của việc gọi nhầm tên.

🧩 **[Ngoài bài gốc] — Muốn hiển thị similarity cho người dùng thì làm sao?**

Người dùng thích thấy "độ khớp 92%" hơn là "khoảng cách 0.08". README chính thức chỉ cách:

```sql
SELECT
    content,
    1 - (embedding <=> '[...]') AS cosine_similarity   -- đổi ngược về similarity
FROM items
ORDER BY embedding <=> '[...]'        -- ⚠️ VẪN sắp xếp bằng distance!
LIMIT 5;
```

⚠️ **Chú ý dòng `ORDER BY`.** Bạn *phải* sắp xếp bằng toán tử `<=>` thuần, **không** được sắp xếp bằng biểu thức `1 - (embedding <=> ...)`. Lý do rất cụ thể và tôi giải thích đầy đủ ở Mục 3.3: index của pgvector **chỉ được dùng khi `ORDER BY` là kết quả trực tiếp của một toán tử khoảng cách, sắp xếp tăng dần**. Viết thành biểu thức thì index bị bỏ qua hoàn toàn, và truy vấn của bạn chậm đi hàng nghìn lần mà không báo lỗi gì.

Cách viết ở trên là cách đúng: tính similarity ở phần `SELECT` để hiển thị, nhưng sắp xếp bằng distance thuần để index vẫn chạy.

### ✅ Self-check Phần 1

**Câu 1.** Phiên bản PostgreSQL tối thiểu để chạy pgvector là bao nhiêu?

<details>
<summary>Gợi ý đáp án</summary>

**PostgreSQL 13** trở lên (README chính thức ghi "supports Postgres 13+"). Con số 12 trong video đã cũ. Thực tế nên dùng 16/17/18 vì có gói cài sẵn và được vá lỗi đầy đủ.
</details>

**Câu 2.** Sau khi cài file pgvector vào server, bạn đã dùng được kiểu `vector` chưa? Còn thiếu gì?

<details>
<summary>Gợi ý đáp án</summary>

**Chưa.** Cài file chỉ là bước 1 — nó chép file vào thư mục Postgres. Còn thiếu bước 2: chạy `CREATE EXTENSION IF NOT EXISTS vector;` **trong từng database** muốn dùng. Quên bước này sẽ gặp lỗi `type "vector" does not exist`. Nhớ: cài app khác với mở app.
</details>

**Câu 3.** `<=>` trả về similarity hay distance? Để lấy kết quả giống nhất lên đầu thì `ORDER BY` tăng dần hay giảm dần?

<details>
<summary>Gợi ý đáp án</summary>

Trả về **cosine distance** (`= 1 − cosine similarity`). Vì là *khoảng cách*, nhỏ hơn nghĩa là gần hơn, nên sắp xếp **tăng dần (ASC)** — cũng là mặc định của SQL, không cần viết thêm. Thêm `DESC` sẽ cho ra chính xác những kết quả *khác biệt nhất*, mà không hề báo lỗi.
</details>

**Câu 4.** Cột của bạn khai `vector(1536)`. Bạn đổi model từ OpenAI sang `all-MiniLM-L6-v2` (384 chiều). Chuyện gì xảy ra khi insert?

<details>
<summary>Gợi ý đáp án</summary>

Báo lỗi ngay: `expected 1536 dimensions, not 384`. Bạn phải sửa định nghĩa cột **và** tạo lại toàn bộ embedding bằng model mới — vì vector của hai model khác nhau không so sánh được với nhau, kể cả nếu tình cờ cùng số chiều. Đây là một dự án di trú, không phải một dòng cấu hình.
</details>

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)

> Phần 1 cho bạn chạy được bằng con đường dễ nhất. Phần này mở rộng ra: **các cách cài khác và chọn cái nào khi nào**, **đầy đủ các toán tử**, **đánh index**, và **kết nối từ code thật**.

### 2.1. Bốn cách cài — chọn cái nào cho hoàn cảnh nào

| Cách | Lệnh cốt lõi | Dùng khi | Độ khó |
|---|---|---|---|
| **Docker** | `docker run pgvector/pgvector:pg18` | Học, phát triển, kiểm thử tự động | ⭐ Dễ nhất |
| **Gói hệ điều hành** | `sudo apt install postgresql-18-pgvector` | Production trên máy chủ thật đã có Postgres | ⭐⭐ Dễ |
| **Dịch vụ có sẵn** | Không cài gì, chỉ `CREATE EXTENSION` | Supabase, Neon, AWS RDS, Google Cloud SQL | ⭐ Dễ nhất |
| **Biên dịch từ mã nguồn** | `git clone && make && make install` | Khi không có gói sẵn cho môi trường của bạn | ⭐⭐⭐⭐ Khó nhất |

Ngoài bốn cách chính còn có Homebrew (macOS), PGXN, conda-forge, pkg (FreeBSD), APK (Alpine) — tuỳ nền tảng đặc thù.

**Cách 2 — gói hệ điều hành, chi tiết:**

```bash
# Debian / Ubuntu — thay 18 bằng phiên bản Postgres của bạn
sudo apt install postgresql-18-pgvector

# RHEL / Rocky Linux / Fedora
sudo dnf install pgvector_18
```

> **`sudo`:** viết tắt của *superuser do* — chạy lệnh với quyền quản trị hệ điều hành. Cần thiết vì bạn đang ghi file vào thư mục hệ thống mà người dùng thường không được ghi.
>
> **`apt` / `dnf`:** trình quản lý gói của hệ điều hành. Chúng tải phần mềm từ kho chính thức, tự xử lý các phần mềm phụ thuộc, và cho phép gỡ ra sạch sẽ. Đây là cách chuẩn để cài phần mềm trên Linux.

💡 **Chi tiết dễ vấp:** con số trong tên gói phải khớp phiên bản **Postgres** của bạn, không phải phiên bản pgvector. Đang chạy Postgres 16 mà cài `postgresql-18-pgvector` thì file rơi vào thư mục của Postgres 18 và server Postgres 16 của bạn không bao giờ thấy nó. Kiểm tra trước:

```sql
SELECT version();
```

**Cách 3 — dịch vụ có sẵn:** Supabase, Neon, AWS RDS/Aurora, Google Cloud SQL đều đã cài sẵn pgvector. Bạn bỏ qua hoàn toàn bước 1, chỉ cần chạy `CREATE EXTENSION IF NOT EXISTS vector;`. Đôi khi phải bật trong bảng điều khiển của nhà cung cấp trước.

⚠️ Lưu ý với dịch vụ có sẵn: phiên bản pgvector họ cung cấp **có thể cũ hơn hoặc mới hơn** bản chính thức. Kiểm tra bằng `SELECT extversion FROM pg_extension WHERE extname='vector';` và đọc changelog của nhà cung cấp, đừng giả định.

> **Changelog (nhật ký thay đổi):** file mà dự án phần mềm dùng để ghi lại từng phiên bản đã thêm gì, sửa gì, và có gì cần lưu ý khi nâng cấp. Đây là thứ đầu tiên cần đọc trước khi nâng phiên bản bất kỳ thư viện nào.

### 2.2. Biên dịch từ mã nguồn — cách của video gốc

Chỉ dùng khi thực sự không có gói sẵn. Nhưng bạn nên hiểu nó, vì đây là cách video dạy và là câu hỏi phỏng vấn hay gặp.

```bash
# ── Chuẩn bị: cài công cụ biên dịch và file header của Postgres ──
sudo apt install build-essential git postgresql-server-dev-18

# ── Tải mã nguồn về, ghim đúng phiên bản ──
cd /tmp
git clone --branch v0.8.3 https://github.com/pgvector/pgvector.git
cd pgvector

# ── Biên dịch ──
make

# ── Chép file đã biên dịch vào thư mục của Postgres ──
sudo make install

# ── Rồi vào psql chạy: CREATE EXTENSION IF NOT EXISTS vector;
```

Giải thích từng thứ:

> **Biên dịch (compile):** pgvector được viết bằng ngôn ngữ C — ngôn ngữ mà máy tính không đọc trực tiếp được. Biên dịch là quá trình dịch mã C thành mã máy mà CPU hiểu.
>
> **`build-essential`:** gói chứa trình biên dịch C và các công cụ đi kèm trên Ubuntu/Debian.
>
> **`postgresql-server-dev-18`:** gói chứa các **file header** của Postgres. Header là những file mô tả "Postgres có những hàm và cấu trúc dữ liệu nào" — trình biên dịch cần chúng để biết cách nối code của pgvector vào code của Postgres. Thiếu gói này, bạn gặp lỗi kinh điển `fatal error: postgres.h: No such file or directory`. Số 18 phải khớp phiên bản Postgres đang chạy.
>
> **`git clone`:** tải toàn bộ mã nguồn của một dự án từ GitHub về máy.
>
> **`--branch v0.8.3`:** ghim đúng phiên bản 0.8.3. **Rất nên làm.** Không có nó, bạn tải về bản mới nhất đang phát triển, có thể chưa ổn định — và quan trọng hơn, lần cài sau bạn sẽ nhận được phiên bản khác lần trước, không tái lập được.
>
> **`make`:** công cụ đọc file hướng dẫn (`Makefile`) rồi gọi trình biên dịch theo đúng trình tự.
>
> **`make install`:** chép các file kết quả vào đúng thư mục mà Postgres tìm.

💡 **Điểm cốt yếu, nhắc lại từ Mục 1.3:** `make install` **chỉ chép file**. Sau lệnh này bạn vẫn **chưa** dùng được kiểu `vector`. Vẫn phải vào psql chạy `CREATE EXTENSION`. Đây là câu hỏi bẫy rất hay gặp: *"`make install` xong là dùng được chưa?"* — **Chưa.**

### 2.3. Đánh index để tìm kiếm nhanh

Không có index, Postgres phải so sánh vector tìm kiếm với **từng hàng một** trong bảng. Với vài nghìn hàng thì không sao. Với vài triệu hàng thì mỗi truy vấn mất nhiều giây.

> **Index (chỉ mục):** cấu trúc dữ liệu phụ được xây sẵn để tìm kiếm nhanh, giống mục lục cuối một quyển sách dày. Không có mục lục thì phải lật từng trang.

pgvector có hai loại index. Cơ chế bên trong thuộc về bài lý thuyết; ở đây tôi chỉ cho bạn cú pháp và cách chọn.

```sql
-- HNSW — lựa chọn mặc định trong đa số trường hợp
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);

-- IVFFlat — xây nhanh hơn, tốn ít bộ nhớ hơn, nhưng tìm kém hơn
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

> **HNSW (Hierarchical Navigable Small World):** loại index xây một mạng lưới nhiều tầng nối các vector với nhau. Tìm kiếm nhanh và chính xác hơn IVFFlat, nhưng xây lâu hơn và tốn bộ nhớ hơn. Ưu điểm thực dụng lớn: **xây được ngay cả khi bảng còn rỗng.**
>
> **IVFFlat:** chia vector thành các nhóm, khi tìm chỉ xét vài nhóm gần nhất. Xây nhanh, nhẹ, nhưng chất lượng kém hơn. **Bắt buộc phải có dữ liệu trong bảng trước khi xây**, vì nó cần dữ liệu để phân nhóm. Xây trên bảng rỗng sẽ cho kết quả tệ.

⚠️ **`vector_cosine_ops` là phần rất hay bị làm sai, và làm sai thì không báo lỗi.**

> **Operator class (lớp toán tử, gọi tắt là ops class):** phần khai báo cho index biết nó phục vụ **phép đo khoảng cách nào**. Một index HNSW được xây cho cosine chỉ giúp cho truy vấn dùng cosine.

Quy tắc bắt buộc thuộc — **toán tử trong truy vấn phải khớp ops class của index:**

| Toán tử bạn dùng khi truy vấn | Ops class phải khai khi tạo index |
|---|---|
| `<->` (L2) | `vector_l2_ops` |
| `<=>` (cosine) | `vector_cosine_ops` |
| `<#>` (inner product) | `vector_ip_ops` |
| `<+>` (L1) | `vector_l1_ops` |

💡 **Nếu khai sai, chuyện gì xảy ra?** Index vẫn được tạo thành công. Truy vấn vẫn chạy đúng và trả về kết quả đúng. Chỉ có điều Postgres **âm thầm bỏ qua index** và quét toàn bảng. Bạn không nhận được lỗi nào, chỉ thấy truy vấn chậm bất thường. Đây là lý do Mục 3.3 (dùng `EXPLAIN ANALYZE`) là kỹ năng bắt buộc.

Bạn có thể tạo **nhiều index** trên cùng một cột nếu cần nhiều phép đo:

```sql
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON items USING hnsw (embedding vector_l2_ops);
```

Nhưng mỗi index tốn thêm dung lượng và làm chậm thao tác ghi. Chỉ tạo cái bạn thực sự dùng.

### 2.4. Sáu toán tử khoảng cách — không phải ba

⚠️ **[Ngoài bài gốc]** Video và phần lớn tài liệu chỉ nhắc ba toán tử. README chính thức liệt kê **sáu**:

| Toán tử | Tên | Dùng cho | Ghi chú |
|---|---|---|---|
| `<->` | Khoảng cách Euclid (L2) | `vector` | Khoảng cách "đường chim bay" |
| `<=>` | **Cosine distance** | `vector` | **Mặc định cho văn bản** |
| `<#>` | Inner product **âm** | `vector` | Xem cảnh báo bên dưới |
| `<+>` | Khoảng cách taxi (L1) | `vector` | Thêm từ bản 0.7.0 |
| `<~>` | Khoảng cách Hamming | `bit` | Cho vector nhị phân |
| `<%>` | Khoảng cách Jaccard | `bit` | Cho vector nhị phân |

⚠️ **`<#>` trả về giá trị ÂM — và đây là chủ ý, không phải lỗi.**

Lý do nằm ở một giới hạn kỹ thuật của Postgres: **index của Postgres chỉ hỗ trợ quét theo thứ tự tăng dần.** Nhưng inner product thì càng *cao* càng giống — ngược chiều. Giải pháp của pgvector là trả về giá trị âm của nó, để "giống nhất" trở thành "số nhỏ nhất", và sắp xếp tăng dần lại đúng.

Nên nếu bạn cần giá trị inner product thật, nhân với −1:

```sql
SELECT (embedding <#> '[...]') * -1 AS inner_product FROM items;
```

**Nên dùng cái nào?** Với embedding văn bản, mặc định là **cosine (`<=>`)**. Một ngoại lệ đáng biết mà README chỉ ra: nếu vector của bạn **đã được chuẩn hoá về độ dài 1** (embedding của OpenAI mặc định như vậy), thì inner product cho kết quả xếp hạng **giống hệt** cosine nhưng **tính nhanh hơn** — vì bỏ được phép chia. Đây là một tối ưu miễn phí nếu bạn biết chắc vector đã chuẩn hoá.

### 2.5. Kết nối từ Node.js

```bash
npm install pg pgvector
```

> **npm:** trình quản lý gói của JavaScript.
>
> **`pg`:** thư viện chuẩn để kết nối Node.js với PostgreSQL, thường gọi là *node-postgres*. Nó là **driver** — phần mềm biết cách nói chuyện với database qua mạng.
>
> **`pgvector`:** thư viện phụ trợ, dạy cho `pg` cách hiểu kiểu `vector`. Lưu ý số phiên bản của gói npm này (0.3.x) **không liên quan** đến số phiên bản của extension pgvector (0.8.3) — hai thứ khác nhau.

```javascript
const { Client } = require('pg');
const pgvector = require('pgvector/pg');   // lưu ý đường dẫn con '/pg'

// Chuỗi kết nối chứa: giao thức, tài khoản, mật khẩu, địa chỉ, tên database
const client = new Client({
  connectionString: 'postgresql://postgres:secret@localhost:5432/postgres'
});
await client.connect();

// ⭐ BƯỚC QUAN TRỌNG NHẤT — và là bước video gốc bỏ sót.
// ⚠️ Tên hàm là registerTypes (SỐ NHIỀU). Tài liệu cũ ghi registerType
//    (số ít) — gõ theo sẽ báo lỗi "is not a function".
await pgvector.registerTypes(client);

// Thêm dữ liệu
const emb = [/* ...384 số do model sinh ra... */];
await client.query(
  'INSERT INTO items (content, embedding) VALUES ($1, $2)',
  ['xin chào', pgvector.toSql(emb)]
);
//                  ↑ toSql() đổi mảng JS thành định dạng chuỗi '[a,b,c]' mà Postgres hiểu

// Truy vấn k gần nhất
const queryVec = pgvector.toSql([/* ...384 số của câu tìm kiếm... */]);
const res = await client.query(
  'SELECT content FROM items ORDER BY embedding <=> $1 LIMIT 5',
  [queryVec]
);
console.log(res.rows.map(r => r.content));

await client.end();   // đóng kết nối, tránh rò rỉ tài nguyên
```

**Vì sao `registerTypes` lại quan trọng đến vậy?**

Khi Postgres gửi dữ liệu về, nó gửi kèm một mã số cho biết đó là kiểu gì. Thư viện `pg` biết sẵn các kiểu chuẩn (số, chuỗi, ngày tháng), nhưng **không biết `vector`** — vì đó là kiểu do extension thêm vào, mỗi database lại có mã số khác nhau.

Không đăng ký, `pg` không biết xử lý sao nên trả về **chuỗi ký tự thô**:

```javascript
console.log(row.embedding);
// "[0.021,-0.087,0.044]"   ← một chuỗi, không phải mảng!
console.log(row.embedding.length);
// 21  ← độ dài CHUỖI, không phải số chiều. Bạn vừa gặp một bug rất khó hiểu.
```

Sau khi đăng ký, bạn nhận về mảng số đúng như mong đợi. Hàm `registerTypes` hoạt động bằng cách hỏi database mã số của kiểu `vector` rồi cài bộ chuyển đổi tương ứng — nên **nó phải chạy sau khi đã kết nối**, không thể chạy trước.

💡 **Nếu bạn dùng connection pool** (bể kết nối — tập nhiều kết nối tái sử dụng, cách chuẩn trong ứng dụng thật), phải đăng ký cho **mỗi** kết nối mới:

```javascript
pool.on('connect', async function (client) {
  await pgvector.registerTypes(client);
});
```

💡 **Về `$1`, `$2` trong câu lệnh:** đây là **tham số** (parameterized query). Bạn viết chỗ trống rồi truyền giá trị riêng, thay vì nối chuỗi. Đây không phải chuyện phong cách — nó là biện pháp bảo mật bắt buộc chống **SQL injection**, kiểu tấn công trong đó kẻ xấu nhét câu lệnh SQL vào ô nhập liệu để chiếm quyền database. **Không bao giờ nối chuỗi để tạo câu SQL.**

### 2.6. Kết nối từ Python

```bash
pip install "psycopg[binary]" pgvector
```

```python
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("postgresql://postgres:secret@localhost:5432/postgres")

# Tương đương registerTypes bên Node.js
register_vector(conn)

# Thêm dữ liệu — emb có thể là list Python hoặc numpy array, thư viện tự chuyển
conn.execute(
    "INSERT INTO items (content, embedding) VALUES (%s, %s)",
    ("xin chào", emb)          # %s là tham số trong psycopg (tương đương $1 bên Node)
)

# Truy vấn
row = conn.execute(
    "SELECT content FROM items ORDER BY embedding <=> %s LIMIT 1",
    (query_vec,)               # ⚠️ dấu phẩy cuối bắt buộc — nếu không Python
).fetchone()                   #    hiểu đây là chuỗi chứ không phải tuple một phần tử

print(row[0])
conn.commit()                  # xác nhận ghi dữ liệu
```

> **psycopg:** thư viện kết nối PostgreSQL chuẩn của Python. Bản `[binary]` cài kèm phần đã biên dịch sẵn, khỏi phải biên dịch lúc cài.
>
> **`commit()`:** trong Postgres, các thay đổi nằm trong một **giao dịch** (transaction) và chỉ được ghi vĩnh viễn khi bạn xác nhận. Quên `commit()` là lý do phổ biến khiến "code chạy không lỗi mà dữ liệu không thấy đâu".

🧩 **[Ngoài bài gốc] — Khi thêm hàng loạt, đừng insert từng dòng.** Với 100.000 sản phẩm, `INSERT` từng dòng sẽ mất hàng giờ. README chính thức khuyến nghị dùng `COPY`:

```python
with conn.cursor().copy("COPY items (content, embedding) FROM STDIN WITH (FORMAT BINARY)") as copy:
    copy.set_types(["text", "vector"])
    for content, emb in data:
        copy.write_row([content, emb])
```

`COPY` là kênh nạp dữ liệu hàng loạt tốc độ cao của Postgres, thường **nhanh hơn 10–100 lần** so với insert từng dòng.

💡 Và một quy tắc thứ tự rất quan trọng: **nạp dữ liệu trước, đánh index sau.** Xây index trên bảng rỗng rồi nhét dữ liệu vào từng chút một chậm hơn nhiều so với nhét hết dữ liệu rồi xây index một lần.

### 2.7. 🧩 [Ngoài bài gốc] Ba lỗi cài đặt kinh điển

#### Lỗi 1 — `type "vector" does not exist` dù đã `CREATE EXTENSION`

Đây là lỗi gây bối rối nhất, vì bạn *chắc chắn* đã chạy lệnh bật extension rồi.

**Nguyên nhân: extension nằm ở một schema không có trong `search_path`.**

> **Schema:** một "thư mục" bên trong database, dùng để nhóm các bảng và kiểu dữ liệu lại. Mặc định mọi thứ nằm trong schema tên `public`. Các hệ thống lớn thường tách nhiều schema, ví dụ mỗi khách hàng một schema.
>
> **`search_path`:** danh sách các schema mà Postgres sẽ lần lượt tìm khi bạn viết một cái tên không kèm schema. Giống biến `PATH` trên hệ điều hành. Nếu kiểu `vector` nằm trong schema `extensions` mà `search_path` chỉ có `public`, thì Postgres tìm không ra và báo "không tồn tại" — dù nó tồn tại ngay đó, chỉ ở chỗ khác.

**Chẩn đoán và sửa:**

```sql
-- Xem extension đang nằm ở schema nào
\dx vector

-- Xem search_path hiện tại
SHOW search_path;

-- Cách sửa nhanh (chỉ có tác dụng trong phiên làm việc này)
SET search_path TO public, extensions;

-- Cách sửa lâu dài cho một database
ALTER DATABASE ten_database SET search_path TO public, extensions;
```

Cách phòng tốt nhất là chỉ định rõ ràng ngay từ đầu:

```sql
CREATE EXTENSION IF NOT EXISTS vector SCHEMA public;
```

#### Lỗi 2 — Quên hẳn bước `CREATE EXTENSION`

Đã nói ở Mục 1.3 và 2.2, nhắc lại vì nó thực sự là lỗi số một. Đặc biệt hay xảy ra khi bạn có nhiều database: bạn bật ở `postgres` (database mặc định) rồi ứng dụng lại kết nối vào `myapp`. Extension phải bật riêng cho **từng** database.

```sql
\c myapp                                      -- chuyển sang database myapp
CREATE EXTENSION IF NOT EXISTS vector;        -- bật ở đây nữa
```

#### Lỗi 3 — Nhiều bản Postgres trên cùng một máy, file rơi nhầm chỗ

Máy bạn có Postgres 15 và Postgres 18 cùng cài. Bạn biên dịch pgvector, nhưng `make` chọn nhầm bản 15, trong khi server đang chạy là bản 18. File `vector.so` rơi vào thư mục của bản 15 và bản 18 không bao giờ thấy nó.

**Chẩn đoán:**

```bash
# Kiểm tra file có ở đúng thư mục của phiên bản đang chạy không (Ubuntu)
ls -l /usr/lib/postgresql/18/lib/vector.so
```

**Sửa** — chỉ rõ cho `make` biết dùng bản nào:

```bash
export PG_CONFIG=/usr/lib/postgresql/18/bin/pg_config
make clean && make && sudo make install
```

> **`pg_config`:** chương trình nhỏ đi kèm mỗi bản cài Postgres, trả lời câu hỏi "bản Postgres này để file ở đâu, biên dịch với tuỳ chọn gì". `make` gọi nó để biết chép file vào đâu. Nhiều bản Postgres nghĩa là nhiều `pg_config`, và `make` sẽ chọn cái nào tìm thấy trước.

💡 Ba lỗi trên đều là lý do để dùng Docker hoặc gói hệ điều hành thay vì tự biên dịch. Docker không có chuyện nhiều bản lẫn lộn — mỗi container chỉ có đúng một Postgres.

### ✅ Self-check Phần 2

**Câu 1.** Bạn chạy `make install` thành công. Đã dùng được kiểu `vector` chưa?

<details>
<summary>Gợi ý đáp án</summary>

**Chưa.** `make install` chỉ chép file `.so` và `.sql` vào thư mục Postgres. Vẫn phải chạy `CREATE EXTENSION IF NOT EXISTS vector;` trong **từng database** muốn dùng. Đây là hai bước khác nhau ở hai tầng khác nhau.
</details>

**Câu 2.** Bạn tạo index với `vector_l2_ops` nhưng truy vấn bằng `<=>`. Chuyện gì xảy ra?

<details>
<summary>Gợi ý đáp án</summary>

Index được tạo bình thường, truy vấn chạy bình thường và **trả về kết quả đúng** — nhưng Postgres **âm thầm bỏ qua index** và quét toàn bảng, nên chậm hơn hàng nghìn lần. Không có lỗi nào được báo. Toán tử truy vấn phải khớp ops class: `<=>` cần `vector_cosine_ops`.
</details>

**Câu 3.** Trong Node.js, vì sao phải gọi `registerTypes(client)` trước khi truy vấn? Và tên hàm cũ trong tài liệu cũ là gì?

<details>
<summary>Gợi ý đáp án</summary>

Vì thư viện `pg` không biết sẵn kiểu `vector` (kiểu do extension thêm vào, mã số khác nhau ở mỗi database). Không đăng ký, nó trả về **chuỗi ký tự thô** `"[0.1,0.2,...]"` thay vì mảng số — và `.length` sẽ cho độ dài chuỗi chứ không phải số chiều. Tài liệu cũ ghi `registerType` số ít; tên hiện tại là **`registerTypes`** số nhiều. Với connection pool, phải đăng ký cho mỗi kết nối mới qua `pool.on('connect', ...)`.
</details>

**Câu 4.** Vì sao `<#>` trả về giá trị âm?

<details>
<summary>Gợi ý đáp án</summary>

Vì index của Postgres chỉ quét được theo thứ tự **tăng dần**, trong khi inner product càng *cao* càng giống — ngược chiều. pgvector trả về giá trị âm để "giống nhất" thành "nhỏ nhất", nhờ đó sắp xếp tăng dần vẫn đúng và index vẫn dùng được. Muốn giá trị thật thì nhân với −1.
</details>

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)

> Phần này nhìn xuống bên dưới bề mặt: quá trình biên dịch thực sự làm gì, làm sao biết chắc index được dùng, vấn đề lọc kết hợp tìm kiếm vector, và cách nén vector khi bộ nhớ thành nút thắt.
>
> Hai từ dùng liên tục từ đây:
>
> **Trade-off (đánh đổi):** tình huống mà cải thiện mặt này bắt buộc phải hy sinh mặt khác. Ví dụ trong bài này: index HNSW tìm nhanh hơn nhưng tốn RAM và xây lâu hơn. Không có lựa chọn tốt ở mọi mặt — kỹ năng của staff engineer là *nói rõ* mình đánh đổi cái gì lấy cái gì.
>
> **Edge case (trường hợp biên):** tình huống hiếm, nằm ở rìa, thường bị bỏ quên khi viết code — và thường là thứ làm sập hệ thống trong production.

### 3.1. Bên trong quá trình biên dịch

Hiểu chỗ này giải thích được một hiện tượng bí ẩn: *"file tôi biên dịch ở máy A chạy được, bê sang máy B thì sập."*

**Chuyện gì thực sự xảy ra khi bạn gõ `make`:**

```
make
 └─→ gọi pg_config để hỏi:
       "Header của Postgres ở đâu?"        → /usr/include/postgresql/18/server
       "Thư viện extension để ở đâu?"      → /usr/lib/postgresql/18/lib
       "Postgres được biên dịch với cờ gì?" → ...
 └─→ biên dịch mã C thành vector.so

make install
 └─→ chép vector.so                        → /usr/lib/postgresql/18/lib/
 └─→ chép vector.control, vector--*.sql    → /usr/share/postgresql/18/extension/
```

> **PGXS (PostgreSQL Extension Building Infrastructure):** hệ thống build chuẩn của Postgres dành cho extension. Nó là lý do bạn chỉ cần gõ `make` mà không phải tự viết lệnh biên dịch — PGXS lo hết việc tìm đường dẫn và cờ biên dịch.
>
> **`.so` (shared object):** file thư viện dùng chung trên Linux, tương đương `.dll` trên Windows. Đây là mã máy mà Postgres nạp vào lúc chạy. Đây chính là "phần thịt" của extension.
>
> **`.control` và `.sql`:** file `vector.control` khai báo thông tin extension (tên, phiên bản mặc định). Các file `vector--0.8.3.sql` chứa lệnh SQL định nghĩa kiểu, toán tử, hàm. Khi bạn gõ `CREATE EXTENSION vector`, Postgres đọc chính các file này rồi thực thi.

Giờ mọi thứ khớp lại: `CREATE EXTENSION` chỉ hoạt động khi các file trên đã nằm đúng chỗ. Nếu chưa, bạn gặp `could not open extension control file` — Postgres đang nói: "tôi tìm ở thư mục extension của mình mà không thấy file nào tên `vector.control`".

**Và đây là điều quan trọng nhất của mục này: `-march=native`.**

> **`-march=native`:** một cờ biên dịch nói với trình biên dịch: *"hãy tối ưu tối đa cho đúng con CPU đang có ở máy này."* Trình biên dịch sẽ dùng cả những lệnh máy đặc biệt mà CPU này hỗ trợ — nhanh hơn, nhưng **CPU khác có thể không hiểu những lệnh đó.**

README chính thức của pgvector nói rõ: mặc định pgvector biên dịch với `-march=native` trên một số nền tảng để đạt hiệu năng tốt nhất. Hệ quả trực tiếp:

```
Biên dịch trên máy A (CPU đời mới)  →  vector.so dùng lệnh máy đời mới
Bê sang máy B (CPU đời cũ hơn)      →  ILLEGAL INSTRUCTION → Postgres sập
```

Đây chính là lời giải cho hiện tượng "chạy được trên máy tôi". Và nó không sập lúc cài — nó sập lúc chạy, khi ai đó thực hiện một truy vấn vector, có thể là giữa đêm.

**Cách khắc phục** — README chỉ rõ:

```bash
make OPTFLAGS=""     # tắt -march=native, đánh đổi chút tốc độ lấy tính di động
```

Và đây cũng là điều mà **image Docker chính thức làm sẵn** cho bạn: nó được biên dịch ở chế độ di động để chạy trên mọi CPU. Một lý do nữa để dùng Docker thay vì tự biên dịch — mục 4.1 sẽ nói kỹ.

### 3.2. `EXPLAIN ANALYZE` — kỹ năng phân biệt người biết dùng với người gõ theo hướng dẫn

💡 **Nguyên tắc nền tảng: CÓ index không có nghĩa là index ĐƯỢC DÙNG.**

Postgres có một bộ phận gọi là **query planner** (bộ lập kế hoạch truy vấn). Với mỗi câu lệnh, nó ước lượng chi phí của nhiều cách thực hiện rồi chọn cách nó *cho là* rẻ nhất. Nó hoàn toàn có thể quyết định bỏ qua index của bạn.

> **`EXPLAIN`:** cho xem *kế hoạch* Postgres định dùng, không thực sự chạy truy vấn.
>
> **`EXPLAIN ANALYZE`:** thực sự **chạy** truy vấn rồi cho xem kế hoạch kèm thời gian thật của từng bước. Đây là công cụ chẩn đoán hiệu năng số một của Postgres.

```sql
EXPLAIN ANALYZE
SELECT id FROM items
ORDER BY embedding <=> '[...]'
LIMIT 10;
```

**Đọc kết quả — chỉ cần tìm một dòng:**

```
✅ TỐT — index đang chạy:
   Index Scan using items_embedding_idx on items  (cost=... rows=10 ...)

❌ VẤN ĐỀ — index bị bỏ qua:
   Seq Scan on items  (cost=... rows=1000000 ...)
```

> **Seq Scan (sequential scan — quét tuần tự):** đọc lần lượt **từng hàng** trong bảng. Với bảng nhỏ thì đây là cách nhanh nhất. Với bảng triệu hàng thì đây là thảm hoạ.
>
> **Index Scan:** dùng index để nhảy thẳng đến các hàng liên quan.

**Bốn nguyên nhân khiến index bị bỏ qua, xếp theo tần suất:**

**Một — Câu truy vấn viết không đúng dạng mà index hỗ trợ.** Đây là nguyên nhân bất ngờ nhất và README nói rất rõ. Index vector **chỉ được dùng khi thoả đủ ba điều kiện**:

1. Câu lệnh có `ORDER BY`
2. Câu lệnh có `LIMIT`
3. Phần `ORDER BY` là **kết quả trực tiếp của một toán tử khoảng cách**, sắp xếp **tăng dần** — không được là biểu thức bọc quanh nó

Ví dụ trực tiếp từ README:

```sql
-- ✅ DÙNG được index
ORDER BY embedding <=> '[3,1,2]' LIMIT 5;

-- ❌ KHÔNG dùng được index — vì ORDER BY là một biểu thức, không phải toán tử thuần
ORDER BY 1 - (embedding <=> '[3,1,2]') DESC LIMIT 5;
```

Dòng thứ hai chính là cái bẫy tôi cảnh báo ở Mục 1.8: người ta muốn sắp xếp theo "độ giống" nên đổi distance thành similarity rồi sắp giảm dần. Câu lệnh chạy đúng, kết quả đúng, nhưng index bị vứt bỏ hoàn toàn và truy vấn chậm đi hàng nghìn lần. **Tính similarity ở phần `SELECT` để hiển thị, nhưng luôn sắp xếp bằng toán tử thuần ở `ORDER BY`.**

**Hai — Bảng quá nhỏ.** Với vài nghìn hàng, planner tính ra quét thẳng còn rẻ hơn đi qua index. **Đây không phải lỗi** — đó là quyết định đúng. Và quét thẳng còn cho độ chính xác 100%, trong khi index xấp xỉ có thể bỏ sót.

**Ba — Ops class không khớp toán tử.** Đã nói ở Mục 2.3.

**Bốn — Thống kê của bảng đã cũ.** Planner ra quyết định dựa trên thống kê được thu thập định kỳ. Nếu bạn vừa nạp một triệu hàng, thống kê chưa kịp cập nhật:

```sql
ANALYZE items;      -- cập nhật thống kê ngay
```

💡 **Mẹo chẩn đoán rất hữu ích từ README:** khi nghi ngờ planner chọn sai, hãy ép nó thử cách khác trong một giao dịch tạm:

```sql
BEGIN;
SET LOCAL enable_seqscan = off;    -- cấm quét tuần tự, buộc dùng index
EXPLAIN ANALYZE SELECT ...;
COMMIT;
```

> **`SET LOCAL`:** đặt cấu hình **chỉ trong giao dịch hiện tại**, tự khôi phục khi kết thúc. Khác với `SET` thường — vốn tác động cả phiên kết nối và có thể rò rỉ sang các truy vấn sau. Chi tiết này rất quan trọng khi dùng connection pool, sẽ nói ở Mục 4.3.

Nếu ép dùng index mà nhanh hơn hẳn, bạn biết planner đã ước lượng sai. Nếu chậm hơn, planner đã đúng và bạn nên để yên.

### 3.3. 🧩 [Ngoài bài gốc] Vấn đề lớn nhất khi kết hợp lọc với tìm kiếm vector

Đây là mục quan trọng nhất Phần 3, và video gốc hoàn toàn không nhắc — vì lời giải chỉ xuất hiện từ pgvector 0.8.0.

**Tình huống:** bạn viết một câu truy vấn rất bình thường — tìm sản phẩm giống nhất, nhưng chỉ trong một danh mục:

```sql
SELECT * FROM items
WHERE category_id = 123
ORDER BY embedding <=> '[...]'
LIMIT 5;
```

Bạn mong 5 kết quả. Bạn nhận về **1 kết quả**. Hoặc không có gì cả.

**Vì sao?** Vì với index xấp xỉ, **việc lọc được áp dụng SAU khi quét index, không phải trước.**

Trình tự thực tế:

```
1. Index HNSW quét và lấy ra ~40 vector gần nhất  (40 là giá trị mặc định của hnsw.ef_search)
2. Postgres lọc 40 cái đó bằng điều kiện category_id = 123
3. Nếu danh mục 123 chỉ chiếm 10% dữ liệu → trung bình chỉ 4 cái sống sót
4. Bạn xin 5, nhận được 4. Hoặc ít hơn.
```

README nói chính xác con số này: với `hnsw.ef_search` mặc định là 40, nếu điều kiện lọc khớp 10% số hàng, trung bình chỉ khoảng 4 hàng qua được cửa.

Đây là loại lỗi cực kỳ khó chịu vì **không có lỗi nào được báo** — bạn chỉ đơn giản nhận được ít kết quả hơn mong đợi, và có thể mất nhiều tháng mới nhận ra.

**Lời giải một: iterative index scan** (từ pgvector 0.8.0).

> **Iterative index scan (quét index lặp):** thay vì quét index đúng một lần rồi dừng, Postgres **tự động quét thêm** cho tới khi đủ số kết quả cần (hoặc chạm giới hạn an toàn).

```sql
-- Bật, giữ đúng thứ tự khoảng cách tuyệt đối
SET hnsw.iterative_scan = strict_order;

-- Hoặc: cho phép thứ tự lệch chút, đổi lại tìm được nhiều kết quả hơn
SET hnsw.iterative_scan = relaxed_order;
```

Trade-off giữa hai chế độ: `strict_order` đảm bảo kết quả sắp đúng thứ tự khoảng cách; `relaxed_order` cho phép hơi lệch nhưng bao phủ tốt hơn. Với tìm kiếm sản phẩm, `relaxed_order` thường là lựa chọn hợp lý — lệch một hai vị trí không ai nhận ra, nhưng thiếu kết quả thì rất rõ.

Có hai van an toàn để quá trình quét thêm không chạy mãi:

```sql
SET hnsw.max_scan_tuples = 20000;      -- tối đa duyệt bao nhiêu hàng (mặc định 20.000)
SET hnsw.scan_mem_multiplier = 2;      -- cho phép dùng gấp mấy lần work_mem
```

**Lời giải hai: đánh index lên cột lọc.** README khuyên đây là chỗ nên bắt đầu:

```sql
CREATE INDEX ON items (category_id);
```

Nếu điều kiện lọc chỉ khớp một tỉ lệ nhỏ số hàng, Postgres có thể lọc trước rồi quét chính xác trên phần còn lại — vừa nhanh vừa cho độ chính xác 100%. **Index chính xác thắng index xấp xỉ khi điều kiện lọc rất hẹp.**

**Lời giải ba: index có điều kiện**, khi bạn chỉ lọc theo vài giá trị cố định:

```sql
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops) WHERE (category_id = 123);
```

**Lời giải bốn: chia bảng theo phân vùng**, khi bạn lọc theo rất nhiều giá trị khác nhau (ví dụ mỗi khách hàng một phân vùng).

💡 **Vì sao mục này đáng giá trong phỏng vấn:** rất nhiều người biết viết truy vấn vector cơ bản, nhưng rất ít người biết rằng thêm một mệnh đề `WHERE` vào có thể âm thầm làm hỏng kết quả. Nói được điều này — và nói được bốn hướng giải — cho thấy bạn đã thực sự vận hành hệ thống chứ không chỉ đọc tài liệu.

### 3.4. 🧩 [Ngoài bài gốc] Nén vector khi bộ nhớ thành nút thắt

Mỗi vector `vector(n)` chiếm `4 × n + 8` byte. Với 1536 chiều là khoảng 6 KB mỗi hàng. Nhân với 10 triệu hàng ra khoảng **60 GB** — chưa tính index. Đây là lúc bộ nhớ trở thành ràng buộc thật.

pgvector cho ba công cụ để giảm.

**Một — `halfvec`: dùng số thực nửa độ chính xác.**

```sql
CREATE TABLE items (id bigserial PRIMARY KEY, embedding halfvec(1536));
```

Mỗi phần tử chỉ chiếm 2 byte thay vì 4, tức **giảm gần một nửa dung lượng**. Độ chính xác giảm rất ít vì embedding vốn không cần nhiều chữ số thập phân đến vậy.

Bạn cũng có thể giữ nguyên cột `vector` mà chỉ đánh index ở nửa độ chính xác — được index nhỏ mà vẫn giữ dữ liệu gốc đầy đủ:

```sql
CREATE INDEX ON items USING hnsw ((embedding::halfvec(1536)) halfvec_l2_ops);
```

**Hai — Lượng tử hoá nhị phân: giảm cực mạnh, kèm bước xếp hạng lại.**

> **Binary quantization (lượng tử hoá nhị phân):** biến mỗi số thực thành đúng **một bit** — dương thành 1, âm thành 0. Dung lượng giảm khoảng **32 lần**. Đổi lại mất nhiều thông tin, nên phải có bước sửa sai.

Mẫu chuẩn là **tìm rộng bằng bản nén, rồi xếp hạng lại bằng bản gốc**:

```sql
SELECT * FROM (
    -- Bước 1: dùng index nhị phân siêu nhẹ để lấy nhanh 20 ứng viên
    SELECT * FROM items
    ORDER BY binary_quantize(embedding)::bit(1536) <~> binary_quantize('[...]')
    LIMIT 20
)
-- Bước 2: xếp hạng lại 20 ứng viên đó bằng vector GỐC, lấy 5 cái tốt nhất
ORDER BY embedding <=> '[...]'
LIMIT 5;
```

Đây là một mẫu thiết kế rất hay và đáng nhớ: **lọc thô nhanh, tinh chỉnh chính xác trên tập nhỏ.** Bạn được tốc độ của bản nén và độ chính xác gần bằng bản gốc.

**Ba — Giới hạn số chiều đánh index cần biết.** README ghi rõ:

| Kiểu | Số chiều tối đa index được |
|---|---|
| `vector` | 2.000 |
| `halfvec` | 4.000 |
| `bit` | 64.000 |

⚠️ **Đây là một cái bẫy thật.** Kiểu `vector` lưu được tới 16.000 chiều, nhưng **index HNSW chỉ đánh được tới 2.000 chiều**. Nếu bạn dùng OpenAI `text-embedding-3-large` (3072 chiều) và cố tạo index HNSW thẳng lên cột `vector(3072)`, bạn sẽ gặp lỗi. Lời giải: dùng `halfvec` (tới 4.000 chiều), hoặc giảm số chiều ngay từ lúc gọi model.

### 3.5. Nâng cấp extension

```bash
# Bước 1: cài phiên bản mới, DÙNG ĐÚNG CÁCH đã cài lần đầu
#   Docker  → đổi tag rồi tạo container mới
#   apt     → sudo apt upgrade postgresql-18-pgvector
#   source  → git pull, checkout tag mới, make, make install
```

```sql
-- Bước 2: trong TỪNG database, cập nhật extension
ALTER EXTENSION vector UPDATE;

-- Bước 3: kiểm chứng
SELECT extversion FROM pg_extension WHERE extname = 'vector';
```

Vẫn là mô hình hai bước quen thuộc: cài file ở tầng hệ điều hành, cập nhật ở tầng database.

⚠️ **Luôn đọc changelog trước khi nâng trên production.** Một số bản có ghi chú đặc biệt, ví dụ yêu cầu xây lại index IVFFlat sau khi nâng từ bản rất cũ. Bỏ qua ghi chú này có thể khiến kết quả tìm kiếm sai mà không báo lỗi.

### 3.6. 🧩 [Ngoài bài gốc] Các trường hợp biên phải thủ sẵn

**1. Vector `NULL` và vector 0 không được đánh index.**

README nói rõ: hàng có `embedding IS NULL` **không xuất hiện** trong kết quả tìm kiếm bằng index. Và với cosine distance, vector toàn số 0 cũng vậy — hợp lý về mặt toán, vì vector 0 không có hướng nên góc không xác định.

Hệ quả thực tế: nếu bạn thêm cột embedding vào bảng có sẵn (`ALTER TABLE ADD COLUMN`), **mọi hàng cũ đều có `NULL`** và biến mất khỏi kết quả tìm kiếm cho tới khi bạn nạp bù. Cần một công việc kiểm tra định kỳ:

```sql
SELECT count(*) FROM items WHERE embedding IS NULL;
```

Con số này phải bằng 0, hoặc phải giải thích được vì sao khác 0.

**2. `NaN` và vô cực bị từ chối.**

README ghi: mọi phần tử của `vector` phải là số hữu hạn — không chấp nhận `NaN`, `Infinity`, `-Infinity`.

> **`NaN` (Not a Number):** giá trị đặc biệt biểu thị kết quả tính toán không hợp lệ, ví dụ 0 chia 0. Tính chất nguy hiểm của nó là lây lan — bất kỳ phép tính nào có `NaN` tham gia cũng cho ra `NaN`.

Đây là một trong những trường hợp hiếm hoi mà việc pgvector nghiêm khắc lại là **món quà**: nó chặn dữ liệu hỏng ngay tại cửa thay vì để nó âm thầm phá kết quả về sau. Nhưng ứng dụng của bạn phải sẵn sàng bắt lỗi này.

**3. Bảng nhỏ dùng quét tuần tự — không phải lỗi.**

Nhắc lại từ Mục 3.2 vì rất nhiều người báo nhầm đây là bug. Với bảng nhỏ, quét thẳng vừa nhanh hơn vừa cho độ chính xác tuyệt đối.

**4. Dọn dẹp index HNSW rất chậm.**

> **VACUUM:** tiến trình dọn dẹp của Postgres, thu hồi không gian của các hàng đã xoá hoặc đã cập nhật.

README cảnh báo VACUUM trên index HNSW có thể mất rất lâu, và gợi ý xây lại index trước cho nhanh hơn:

```sql
REINDEX INDEX CONCURRENTLY index_name;
VACUUM table_name;
```

**5. Docker cộng với xây index song song cần thêm bộ nhớ chia sẻ.**

README có một lưu ý dễ bỏ qua: nếu bạn tăng `maintenance_work_mem`, phải tăng `--shm-size` của container tương ứng, nếu không việc xây index HNSW song song sẽ lỗi.

```bash
docker run --shm-size=1g ...
```

> **`shm`:** viết tắt của *shared memory* — vùng nhớ mà nhiều tiến trình cùng truy cập được. Docker mặc định cấp rất ít (64 MB), đủ cho việc thường nhưng không đủ cho xây index song song.

**6. Cùng một tên nhưng khác phiên bản giữa các máy chủ.**

Nếu bạn có máy chủ chính và máy chủ bản sao chạy hai phiên bản pgvector khác nhau, có thể phát sinh lỗi khi bản sao áp dụng nhật ký ghi. Giữ đồng bộ — chi tiết ở Mục 4.2.

### ✅ Self-check Phần 3

**Câu 1.** Vì sao file pgvector tự biên dịch ở máy này có thể làm sập Postgres ở máy khác? Docker giải quyết bằng cách nào?

<details>
<summary>Gợi ý đáp án</summary>

Vì pgvector mặc định biên dịch với cờ `-march=native`, tối ưu cho đúng CPU của máy biên dịch và có thể dùng những lệnh máy mà CPU khác không hiểu — gây lỗi *illegal instruction* lúc chạy, không phải lúc cài. Khắc phục thủ công: `make OPTFLAGS=""`. Docker giải quyết vì image chính thức đã được biên dịch sẵn ở chế độ di động, chạy được trên mọi CPU.
</details>

**Câu 2.** Bạn có index HNSW nhưng `EXPLAIN ANALYZE` vẫn hiện `Seq Scan`. Nêu ba nguyên nhân.

<details>
<summary>Gợi ý đáp án</summary>

(a) Câu truy vấn không đúng dạng — thiếu `ORDER BY` hoặc `LIMIT`, hoặc `ORDER BY` là một *biểu thức* thay vì toán tử khoảng cách thuần tăng dần (ví dụ `ORDER BY 1 - (embedding <=> ...) DESC`). (b) Bảng quá nhỏ nên planner tính ra quét thẳng rẻ hơn — đây không phải lỗi. (c) Ops class của index không khớp toán tử truy vấn. Thêm: thống kê bảng đã cũ, chạy `ANALYZE items;`.
</details>

**Câu 3.** Truy vấn của bạn có `WHERE category_id = 123` và trả về ít kết quả hơn `LIMIT`. Vì sao, và sửa thế nào?

<details>
<summary>Gợi ý đáp án</summary>

Vì với index xấp xỉ, **lọc được áp dụng SAU khi quét index**. HNSW lấy ra khoảng 40 ứng viên (mặc định `hnsw.ef_search`), rồi mới lọc — nếu danh mục chiếm 10% dữ liệu thì trung bình chỉ còn 4 cái. Bốn hướng sửa: bật `hnsw.iterative_scan` (0.8.0+), đánh index lên cột lọc, dùng index có điều kiện, hoặc chia bảng theo phân vùng.
</details>

**Câu 4.** Bạn dùng model 3072 chiều và muốn đánh index HNSW. Vấn đề là gì?

<details>
<summary>Gợi ý đáp án</summary>

Kiểu `vector` chỉ **đánh index** được tối đa **2.000 chiều** (dù lưu được tới 16.000). Lời giải: chuyển sang `halfvec` (index được tới 4.000 chiều, đồng thời giảm gần nửa dung lượng), dùng lượng tử hoá nhị phân kèm bước xếp hạng lại, hoặc giảm số chiều ngay khi gọi model nếu model hỗ trợ.
</details>

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & vận hành)

> Phần này khác các phần trên ở một điểm căn bản: không còn là "làm sao cho nó chạy", mà là **"làm sao cho nó chạy trên hệ thống đang phục vụ khách hàng thật, không ai bị gián đoạn, và nếu hỏng thì lui lại được"**.
>
> Vài từ dùng liên tục:
>
> **Production (môi trường thật):** hệ thống đang phục vụ người dùng thật. Đối lập với môi trường phát triển trên máy lập trình viên và môi trường thử nghiệm (staging).
>
> **Downtime (thời gian chết):** khoảng thời gian hệ thống không phục vụ được. Với sàn thương mại điện tử, mỗi phút downtime là tiền mất thật.
>
> **Rollback (lui lại):** khả năng quay về trạng thái trước khi thay đổi. Một staff engineer luôn có kế hoạch lui trước khi tiến.

### 4.1. Vì sao tuyệt đối không biên dịch tay trên production

Video gốc dạy `git clone` + `make` + `make install`. Với việc học thì được. Làm thế trên máy chủ production là một **anti-pattern** — cách làm nhìn có vẻ hợp lý nhưng gây hại về lâu dài.

Bốn lý do, xếp theo mức độ nghiêm trọng:

**Một — Kết quả không tái lập được.** Như phân tích ở Mục 3.1, kết quả biên dịch phụ thuộc vào CPU, trình biên dịch, và các gói header có trên máy đó vào thời điểm đó. Biên dịch lại sáu tháng sau trên máy khác cho ra một file khác. Bạn không còn biết chính xác cái gì đang chạy trên hệ thống của mình.

**Hai — Không lui lại sạch được.** Cài bằng `apt` thì gỡ bằng `apt remove` — sạch sẽ, có lưu vết. Biên dịch tay thì bạn phải tự nhớ nó đã rải file vào những thư mục nào.

**Ba — Các máy chủ bị lệch nhau.** Bạn có ba máy chủ, biên dịch trên từng cái vào ba thời điểm khác nhau. Giờ chúng chạy ba bản khác nhau và bạn không biết. Kết hợp với cảnh báo `-march=native` ở Mục 3.1, đây là công thức tạo ra loại lỗi chỉ xảy ra trên một máy và không ai tái hiện được.

**Bốn — Máy chủ production không nên có trình biên dịch.** Có sẵn `gcc`, `make`, `git` trên production là mở rộng bề mặt tấn công: kẻ xâm nhập được vào máy sẽ có đủ công cụ để biên dịch mã độc ngay tại chỗ.

**Cách làm đúng: coi pgvector như mọi thư viện khác — có phiên bản, được ghim rõ ràng.**

```yaml
# Docker Compose — mọi môi trường dùng chung MỘT tag chính xác
services:
  db:
    image: pgvector/pgvector:0.8.3-pg18     # ghim CẢ pgvector LẪN Postgres
    shm_size: 1gb                            # cần cho xây index HNSW song song (Mục 3.6)
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - pgdata:/var/lib/postgresql/data      # dữ liệu sống sót khi container bị xoá
volumes:
  pgdata:
```

💡 **Chú ý tag `0.8.3-pg18` thay vì chỉ `pg18`.** Tag `pg18` là tag "trôi" — hôm nay trỏ tới pgvector 0.8.3, vài tháng sau nhà phát hành cập nhật thì nó trỏ tới bản khác. Bạn dựng lại môi trường và bỗng nhiên chạy phiên bản khác mà không hề sửa dòng cấu hình nào. **Ghim đủ cả hai số** thì môi trường mới thực sự tái lập được.

Với máy chủ thật không dùng Docker, ghim phiên bản gói qua công cụ quản lý cấu hình:

```yaml
# Ansible
- name: Cài pgvector với phiên bản cố định
  apt:
    name: postgresql-18-pgvector=0.8.3-1.pgdg22.04+1
    state: present
```

> **Volume trong Docker:** vùng lưu trữ tồn tại độc lập với container. Không có nó, một lệnh `docker rm` là mất sạch dữ liệu. Bắt buộc phải có khi dùng Docker cho bất cứ việc gì ngoài học tập.

### 4.2. Triển khai lên database đang chạy — không gián đoạn

Đây là tình huống thực tế phổ biến nhất và cũng là câu hỏi phỏng vấn hay gặp nhất về chủ đề này.

**Bước 1 — Cài file, không cần khởi động lại.** Cài gói qua `apt`/`dnf` chỉ chép file vào thư mục, **không cần khởi động lại Postgres**. Với dịch vụ có sẵn thì bỏ qua hẳn bước này.

**Bước 2 — Bật extension, qua migration.**

> **Migration (di trú):** file script chứa các thay đổi cấu trúc database, được lưu cùng mã nguồn, đánh số thứ tự và chạy tự động khi triển khai. Đây là cách chuẩn để thay đổi database — vì nó được lưu vết, được review như code, và chạy giống hệt nhau ở mọi môi trường.

```sql
-- migrations/001_enable_pgvector.sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Ba điều cần chú ý: lệnh chạy rất nhanh và **không khoá bảng nào**; nó **idempotent** nhờ `IF NOT EXISTS`; và nó **cần quyền cao** nên phải chạy bằng tài khoản quản trị, không phải tài khoản ứng dụng.

**Bước 3 — Thêm cột, để trống.**

```sql
ALTER TABLE products ADD COLUMN embedding vector(1536);
```

💡 **Vì sao bước này an toàn?** Vì cột mới cho phép `NULL` và không có giá trị mặc định, Postgres hiện đại chỉ ghi lại thông tin mô tả cấu trúc, **không viết lại từng hàng dữ liệu**. Thao tác xong gần như tức thì kể cả trên bảng nhiều triệu hàng.

⚠️ **Ngược lại, thêm cột kèm `NOT NULL` hoặc giá trị mặc định phức tạp có thể buộc Postgres viết lại toàn bộ bảng và khoá nó suốt quá trình** — nghĩa là downtime. Chi tiết này đáng giá cả một sự cố.

**Bước 4 — Cập nhật luồng ghi TRƯỚC khi nạp bù.**

Đây là thứ tự dễ làm sai nhất. Trong lúc bạn nạp bù dữ liệu cũ, sản phẩm mới vẫn liên tục được thêm vào. Nếu luồng thêm sản phẩm chưa được cập nhật để sinh embedding, mọi bản ghi tạo ra trong khoảng thời gian đó sẽ **vĩnh viễn thiếu vector** — và bạn sẽ không phát hiện ra cho tới rất lâu sau.

**Bước 5 — Nạp bù, chạy nền có giới hạn tốc độ.**

> **Backfill (nạp bù):** tiến trình chạy nền lấp đầy dữ liệu cho những hàng đã tồn tại từ trước.

```python
BATCH = 500
while True:
    rows = fetch_rows_where_embedding_is_null(limit=BATCH)
    if not rows:
        break

    embeddings = model.encode([r.content for r in rows])   # gọi model theo lô
    update_embeddings(rows, embeddings)                     # ghi theo lô

    time.sleep(0.5)   # CỐ Ý nghỉ giữa các lô.
    # Không có dòng này, tiến trình nạp bù cạnh tranh tài nguyên với
    # người dùng thật và làm chậm cả hệ thống.
    # Chậm mà êm tốt hơn nhanh mà gây sự cố.
```

**Bước 6 — Đánh index, không khoá ghi.**

```sql
CREATE INDEX CONCURRENTLY idx_products_embedding
ON products USING hnsw (embedding vector_cosine_ops);
```

> **`CREATE INDEX CONCURRENTLY`:** biến thể xây index **không khoá thao tác ghi** vào bảng. Đánh đổi: chậm hơn (Postgres phải quét bảng hai lần) và không chạy được bên trong một giao dịch.

Vì sao bắt buộc trong production: xây index HNSW trên bảng nhiều triệu hàng có thể mất **hàng giờ**. `CREATE INDEX` thường sẽ khoá mọi thao tác ghi suốt thời gian đó — khách hàng không đặt được hàng trong nhiều giờ.

⚠️ **Rủi ro cần biết:** nếu lệnh này thất bại giữa chừng, nó để lại một index ở trạng thái **không hợp lệ** — vẫn chiếm dung lượng nhưng không dùng được. Phải kiểm tra và dọn:

```sql
SELECT indexrelid::regclass FROM pg_index WHERE NOT indisvalid;
DROP INDEX CONCURRENTLY ten_index_hong;
```

Hai chỉnh sửa giúp xây nhanh hơn nhiều (theo README chính thức):

```sql
SET maintenance_work_mem = '8GB';           -- index xây nhanh hẳn khi vừa bộ nhớ
SET max_parallel_maintenance_workers = 7;   -- dùng nhiều nhân CPU cùng lúc
```

⚠️ Đừng đặt `maintenance_work_mem` cao tới mức cạn bộ nhớ máy chủ. Và nếu chạy trong Docker, nhớ tăng `shm_size` tương ứng (Mục 3.6).

Theo dõi tiến độ trong lúc chờ:

```sql
SELECT phase, round(100.0 * blocks_done / nullif(blocks_total, 0), 1) AS "%"
FROM pg_stat_progress_create_index;
```

**Bước 7 — Kiểm chứng ba thứ, rồi mới bật tính năng.**

```sql
\dx vector                                              -- extension đúng phiên bản?
SELECT count(*) FROM products WHERE embedding IS NULL;  -- còn hàng nào chưa nạp bù?
EXPLAIN ANALYZE SELECT ... ORDER BY embedding <=> ...;  -- index có được dùng không?
```

**Kế hoạch lui, chuẩn bị trước khi bắt đầu:**

```sql
DROP INDEX CONCURRENTLY idx_products_embedding;   -- lui bước 6
ALTER TABLE products DROP COLUMN embedding;       -- lui bước 3
-- Extension để lại cũng vô hại, không cần gỡ
```

💡 Vì mọi bước đều là thao tác **cộng thêm** chứ không sửa dữ liệu cũ, việc lui lại rất sạch. **Đó không phải may mắn — đó là do trình tự các bước được thiết kế như vậy.** Nói được điều này trong phỏng vấn là dấu hiệu tư duy staff.

### 4.3. Bảo mật, phân quyền, và vá lỗi

**Nguyên tắc quyền tối thiểu.**

> **Principle of least privilege (quyền tối thiểu):** mỗi thành phần chỉ được cấp đúng quyền nó cần, không hơn. Để ứng dụng chạy bằng superuser nghĩa là một lỗ hổng nhỏ có thể trở thành mất toàn bộ database.

```sql
-- Tài khoản ứng dụng: chỉ đọc và ghi trên bảng cần thiết
CREATE USER app_user WITH PASSWORD '...';
GRANT SELECT, INSERT, UPDATE ON products TO app_user;

-- KHÔNG cấp quyền tạo extension cho tài khoản ứng dụng.
-- CREATE EXTENSION do quản trị viên chạy qua migration, một lần.
```

**Bẫy rất cụ thể với connection pool.**

> **Connection pool (bể kết nối):** tập các kết nối database được giữ sẵn và tái sử dụng, thay vì mở mới mỗi lần. PgBouncer là công cụ phổ biến nhất.

Vấn đề: các tham số đặt bằng `SET` sẽ **bám vào kết nối**. Với connection pool, kết nối đó sau này được giao cho một yêu cầu khác — và tham số của bạn **rò rỉ sang** yêu cầu đó.

```sql
-- ❌ SAI khi có connection pool
SET hnsw.ef_search = 200;
SELECT ...;
-- Kết nối trả về pool, vẫn mang ef_search = 200.
-- Yêu cầu tiếp theo nhận kết nối này sẽ chậm bất thường mà không ai hiểu vì sao.

-- ✅ ĐÚNG — dùng SET LOCAL trong giao dịch
BEGIN;
SET LOCAL hnsw.ef_search = 200;   -- tự khôi phục khi giao dịch kết thúc
SELECT ...;
COMMIT;
```

Loại lỗi này rất khó chẩn đoán vì nó không tái hiện nhất quán — chỉ những yêu cầu tình cờ nhận đúng kết nối "bị nhiễm" mới bị ảnh hưởng.

**Vá lỗi bảo mật — bài học cụ thể từ chính năm 2026.**

Đầu năm 2026, pgvector phát hành bản **0.8.2** để vá một lỗi tràn bộ nhớ đệm xảy ra khi **xây index HNSW song song** (mã lỗi CVE-2026-3172). Lỗi này có thể làm **rò rỉ dữ liệu từ các bảng khác** hoặc làm sập máy chủ database. Nhóm phát triển khuyến nghị người dùng nâng cấp.

> **CVE (Common Vulnerabilities and Exposures):** hệ thống định danh chuẩn quốc tế cho các lỗ hổng bảo mật đã công bố. Mỗi lỗ hổng có một mã duy nhất để cả ngành cùng tham chiếu.

> **Dependency (thành phần phụ thuộc):** bất kỳ phần mềm bên ngoài nào mà hệ thống của bạn cần để chạy — thư viện, extension, dịch vụ. Điểm quan trọng: mỗi dependency đều có phiên bản, có lỗi, và có bản vá; nên nó cần được quản lý chứ không phải cài một lần rồi quên.

💡 **Bài học cốt lõi:** pgvector là một dependency có vòng đời, giống mọi thư viện khác. Nó cần được **theo dõi, đánh phiên bản, và vá lỗi**. Rất nhiều đội cài extension một lần rồi quên nó tồn tại trong hai năm — cho tới khi có sự cố như trên.

Việc cụ thể nên làm: theo dõi changelog chính thức, đặt lịch rà soát phiên bản mỗi quý, và kiểm tra phiên bản đang chạy trên **tất cả** máy chủ, kể cả bản sao.

**Đồng bộ phiên bản giữa máy chủ chính và bản sao.**

> **Replica (bản sao):** máy chủ database sao chép dữ liệu từ máy chủ chính, dùng để chia tải đọc hoặc dự phòng.
>
> **WAL (Write-Ahead Log — nhật ký ghi trước):** cơ chế trong đó Postgres ghi mọi thay đổi vào nhật ký trước khi áp dụng. Bản sao đọc nhật ký này để tự cập nhật theo.

Vì bản sao phải áp dụng các bản ghi nhật ký liên quan đến kiểu `vector`, phiên bản pgvector giữa máy chính và mọi bản sao phải **giống nhau**. Lệch phiên bản có thể gây lỗi khi áp dụng nhật ký — sự cố khó chẩn đoán vì nó chỉ xuất hiện ở bản sao.

### 4.4. Kiểm thử tự động

> **CI/CD (Continuous Integration / Continuous Deployment):** hệ thống tự động chạy kiểm thử mỗi khi có thay đổi mã nguồn, và tự động triển khai nếu kiểm thử đạt.

**Quy tắc một: môi trường kiểm thử dùng đúng image như production.**

```yaml
# GitHub Actions
services:
  postgres:
    image: pgvector/pgvector:0.8.3-pg18     # GIỐNG HỆT production
```

Nếu kiểm thử chạy trên Postgres thường không có pgvector, các bài kiểm thử liên quan đến vector sẽ bị bỏ qua âm thầm — và bạn phát hiện lỗi ở production.

**Quy tắc hai: kiểm thử cả việc index có được dùng hay không.**

Đây là ý tưởng ít người nghĩ tới nhưng rất giá trị. Một thay đổi nhìn vô hại ở tầng ứng dụng — ví dụ ai đó sửa `ORDER BY` thành biểu thức để hiển thị điểm phần trăm — có thể khiến index bị bỏ qua (Mục 3.2). Truy vấn vẫn trả kết quả **đúng**, kiểm thử chức năng vẫn xanh, chỉ có hiệu năng sụp đổ. Không ai phát hiện cho tới khi khách hàng phàn nàn.

```python
def test_vector_index_is_actually_used():
    plan = db.execute(
        "EXPLAIN SELECT id FROM items ORDER BY embedding <=> %s LIMIT 5",
        (sample_vector,)
    ).fetchall()
    plan_text = "\n".join(row[0] for row in plan)

    assert "Index Scan" in plan_text, f"Index không được dùng! Kế hoạch:\n{plan_text}"
    assert "Seq Scan" not in plan_text
```

💡 Bài kiểm thử này bắt được loại lỗi mà không bài kiểm thử chức năng nào bắt được: **hồi quy về hiệu năng**. Đề xuất nó trong buổi phỏng vấn là một điểm cộng rõ ràng.

**Quy tắc ba: migration phải idempotent.** Dùng `IF NOT EXISTS`, `IF EXISTS`. Vì trong thực tế migration *sẽ* bị chạy lại — do lỗi mạng, do thử lại tự động, do người khác vô tình.

### 4.5. Nói chuyện với người không rành kỹ thuật

Nếu bạn không thuyết phục được quản lý sản phẩm hoặc giám đốc, bạn sẽ không được quyết định. Nguyên tắc: **bỏ hết thuật ngữ, quy về ba thứ họ quan tâm — rủi ro, thời gian, tiền.**

Một đoạn có thể dùng gần như nguyên văn:

> *"Để thêm tính năng tìm kiếm thông minh, chúng ta **không cần dựng hệ thống mới**. Database hiện tại đã hỗ trợ sẵn, chỉ cần bật thêm một tính năng lên — giống cài thêm ứng dụng vào điện thoại đã có, chứ không phải mua điện thoại mới.*
>
> *Về **rủi ro**: thấp. Mọi thay đổi đều là thêm vào, không sửa dữ liệu đang có, nên nếu có vấn đề thì gỡ ra được sạch sẽ trong vài phút. Khách hàng không bị gián đoạn ở bất kỳ bước nào — tôi đã sắp trình tự để đảm bảo điều đó.*
>
> *Về **thời gian**: phần bật tính năng mất một buổi. Phần lâu hơn là xử lý dữ liệu cũ đã có, chạy nền khoảng hai ngày mà không ảnh hưởng ai.*
>
> *Cái cần đầu tư không phải là mua phần mềm — nó miễn phí và mã nguồn mở. Cái cần đầu tư là **quy trình**: đảm bảo mọi máy chủ chạy đúng một phiên bản, có lịch cập nhật bản vá bảo mật, và có kiểm thử tự động. Khoảng ba ngày công thiết lập ban đầu, sau đó gần như không tốn gì."*

Hãy để ý: đoạn trên không có một thuật ngữ nào chưa giải thích. Không "extension", không "index", không "HNSW". Nhưng nó truyền tải đủ mức rủi ro, mốc thời gian, và điểm cần đầu tư.

**Hai điều nên chủ động nêu ở tầm tổ chức:**

**Một — Chuẩn hoá cách cài trong toàn công ty.** Nếu ba đội cùng dùng Postgres, hãy thống nhất một cách cài duy nhất và một phiên bản duy nhất. Không thì bạn sẽ có đội dùng Docker, đội dùng `apt`, đội tự biên dịch — ba phiên bản, ba kiểu lỗi, và không ai giúp được ai khi có sự cố.

**Hai — pgvector cần một người chịu trách nhiệm có tên.** Không phải công việc toàn thời gian, nhưng phải có người cụ thể gắn với việc theo dõi changelog và lên lịch nâng cấp. Sự cố CVE ở Mục 4.3 cho thấy vì sao: nếu không ai theo dõi, bản vá ra mắt mà hệ thống vẫn chạy bản lỗi hàng năm trời.

### 4.6. Câu hỏi vận hành mẫu + hướng trả lời

> **Đề bài:** *"Đội muốn thêm pgvector vào một PostgreSQL production đang phục vụ khách hàng. Bảng chính có 20 triệu hàng. Không được có downtime. Bạn triển khai thế nào?"*

Chú ý bước 1 và bước 9 — đó là hai bước ứng viên hay bỏ qua nhất, và cũng là hai bước ghi điểm nhất.

**Bước 1 — Làm rõ đề trước khi vẽ gì cả.**

- Postgres tự vận hành hay dịch vụ có sẵn? (Quyết định cách cài file.)
- Phiên bản Postgres hiện tại? (Phải từ 13 trở lên.)
- Có bản sao không, mấy cái? (Ảnh hưởng kế hoạch đồng bộ phiên bản.)
- Model embedding nào, bao nhiêu chiều? (Trên 2.000 chiều thì phải tính chuyện `halfvec`.)
- Truy vấn có kèm điều kiện lọc không? (Nếu có thì phải xử lý vấn đề ở Mục 3.3 ngay từ thiết kế.)
- Có được phép có cửa sổ bảo trì không, hay tuyệt đối không gián đoạn?

**Bước 2 — Cài file, không khởi động lại.** Dịch vụ có sẵn thì bật qua bảng điều khiển. Tự vận hành thì `apt install postgresql-18-pgvector` với phiên bản ghim rõ. **Không biên dịch tay trên production** — nêu lý do `-march=native` và tính tái lập.

**Bước 3 — Bật extension qua migration**, chạy bằng tài khoản quản trị. Nhanh, không khoá gì.

**Bước 4 — Thêm cột cho phép trống.** Gần như tức thì vì Postgres không viết lại bảng. Nêu rõ vì sao *không* thêm `NOT NULL` ở bước này.

**Bước 5 — Cập nhật luồng ghi TRƯỚC, rồi mới nạp bù.** Nêu rõ lý do thứ tự: nếu ngược lại, mọi bản ghi tạo trong lúc nạp bù sẽ vĩnh viễn thiếu vector.

**Bước 6 — Đánh index bằng `CREATE INDEX CONCURRENTLY`, SAU khi nạp bù xong.** Xây sau khi có dữ liệu nhanh hơn nhiều. Tăng `maintenance_work_mem` và số tiến trình song song. Theo dõi qua `pg_stat_progress_create_index`. Chuẩn bị sẵn quy trình dọn nếu lệnh thất bại giữa chừng.

**Bước 7 — Kiểm chứng ba thứ** trước khi bật tính năng, cộng thêm kiểm tra phiên bản pgvector trên **mọi bản sao**.

**Bước 8 — Kế hoạch lui, viết ra TRƯỚC khi chạy bước đầu tiên.** Vì mọi bước đều là cộng thêm chứ không sửa dữ liệu cũ, lui lại rất sạch: bỏ index, bỏ cột. Extension để lại vô hại.

**Bước 9 — Điều kiện nào khiến tôi làm khác đi.** Nếu bảng lớn hơn nhiều, ví dụ 500 triệu hàng, tôi cân nhắc `halfvec` hoặc lượng tử hoá nhị phân ngay từ đầu vì bộ nhớ cho index sẽ thành nút thắt. Nếu truy vấn luôn kèm điều kiện lọc rất hẹp, tôi có thể **không cần index xấp xỉ chút nào** — index thường trên cột lọc cộng quét chính xác trên phần còn lại vừa nhanh vừa cho độ chính xác tuyệt đối.

💡 **Vì sao bước 9 là dấu hiệu của staff:** nêu rõ *điều kiện nào sẽ khiến mình đổi phương án* cho thấy bạn hiểu quyết định của mình dựa trên cái gì, thay vì học thuộc một quy trình cố định. Người phỏng vấn tìm chính xác điều này.

### ✅ Self-check Phần 4

**Câu 1.** Nêu ba lý do không nên biên dịch pgvector từ mã nguồn trên máy chủ production.

<details>
<summary>Gợi ý đáp án</summary>

(a) Không tái lập được — kết quả phụ thuộc CPU, trình biên dịch và các gói có trên máy tại thời điểm đó. (b) Không lui lại sạch được vì không có trình quản lý gói để gỡ. (c) Các máy chủ dễ lệch phiên bản nhau mà không ai biết, cộng với `-march=native` tạo ra lỗi chỉ xảy ra trên một máy. (d) Máy production không nên có sẵn trình biên dịch vì lý do an ninh. Cách đúng: ghim tag Docker đầy đủ (`0.8.3-pg18`) hoặc ghim phiên bản gói OS.
</details>

**Câu 2.** Vì sao phải dùng `CREATE INDEX CONCURRENTLY` trên production? Rủi ro của nó là gì?

<details>
<summary>Gợi ý đáp án</summary>

Vì `CREATE INDEX` thường **khoá mọi thao tác ghi** vào bảng suốt thời gian xây, mà xây HNSW trên bảng nhiều triệu hàng có thể mất hàng giờ — khách hàng không ghi được dữ liệu suốt thời gian đó. `CONCURRENTLY` không khoá ghi. Rủi ro: chậm hơn (quét bảng hai lần), không chạy được trong giao dịch, và nếu thất bại giữa chừng để lại index **không hợp lệ** vẫn chiếm dung lượng — phải kiểm tra `pg_index WHERE NOT indisvalid` và dọn bằng `DROP INDEX CONCURRENTLY`.
</details>

**Câu 3.** Bạn dùng PgBouncer. Vì sao `SET hnsw.ef_search = 200` nguy hiểm, và thay bằng gì?

<details>
<summary>Gợi ý đáp án</summary>

Vì `SET` bám vào **kết nối**, mà connection pool tái sử dụng kết nối cho các yêu cầu khác nhau — tham số sẽ **rò rỉ** sang những yêu cầu sau, khiến chúng chậm bất thường mà không ai hiểu vì sao. Đặc biệt khó chẩn đoán vì không tái hiện nhất quán. Thay bằng `SET LOCAL` bên trong `BEGIN ... COMMIT` — nó tự khôi phục khi giao dịch kết thúc.
</details>

**Câu 4.** Vì sao nên viết một bài kiểm thử tự động kiểm tra `EXPLAIN` có chứa `Index Scan`?

<details>
<summary>Gợi ý đáp án</summary>

Vì đây là loại lỗi mà kiểm thử chức năng **không bắt được**: một thay đổi ở tầng ứng dụng (ví dụ sửa `ORDER BY` thành biểu thức để hiển thị phần trăm) khiến index bị bỏ qua, nhưng truy vấn vẫn trả về **kết quả đúng**. Kiểm thử chức năng vẫn xanh, chỉ có hiệu năng sụp đổ hàng nghìn lần. Đây là bài kiểm thử chống **hồi quy về hiệu năng**.
</details>

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)

> Phần này để ôn nhanh trong 20 phút trước buổi phỏng vấn. Nếu đã đọc bốn phần trên, đây chỉ là gợi nhớ.

### 5.1. Bảng thuật ngữ bắt buộc nhớ

| Thuật ngữ | Định nghĩa một dòng |
|---|---|
| **pgvector** | Extension thêm kiểu `vector` và khả năng tìm kiếm vector cho PostgreSQL |
| **Extension** | Gói tính năng cắm thêm vào Postgres để dạy nó làm việc nó vốn không biết |
| **`CREATE EXTENSION vector`** | Bật extension **trong một database** — bước 2, sau khi đã cài file |
| **`vector(n)`** | Kiểu cột lưu embedding n chiều; n phải khớp chính xác với model |
| **`halfvec(n)`** | Vector nửa độ chính xác — 2 byte/phần tử, giảm gần nửa dung lượng |
| **`<->` / `<=>` / `<#>` / `<+>`** | L2 / **cosine distance** / inner product âm / L1 |
| **`<~>` / `<%>`** | Hamming / Jaccard — dành cho vector nhị phân kiểu `bit` |
| **Ops class** | `vector_l2_ops` / `vector_cosine_ops` / `vector_ip_ops` — **phải khớp** toán tử truy vấn |
| **HNSW** | Index đồ thị nhiều tầng; tìm nhanh và chính xác hơn, xây lâu, tốn RAM; xây được trên bảng rỗng |
| **IVFFlat** | Index chia nhóm; xây nhanh, nhẹ, tìm kém hơn; **bắt buộc phải có dữ liệu trước** |
| **PGXS** | Hệ thống build extension của Postgres, dùng khi biên dịch từ mã nguồn |
| **`pg_config`** | Chương trình cho biết bản Postgres này để file ở đâu — `make` gọi nó |
| **`.so`** | File thư viện dùng chung — "phần thịt" của extension |
| **`-march=native`** | Cờ biên dịch tối ưu cho CPU máy hiện tại; nguyên nhân lỗi *illegal instruction* ở máy khác |
| **`\dx` / `pg_extension`** | Hai cách kiểm tra extension và phiên bản |
| **`ALTER EXTENSION vector UPDATE`** | Nâng cấp extension trong một database (sau khi đã cài file bản mới) |
| **`EXPLAIN ANALYZE`** | Chạy truy vấn và cho xem kế hoạch thật — công cụ chẩn đoán số một |
| **Index Scan / Seq Scan** | Dùng index / quét toàn bảng |
| **`CREATE INDEX CONCURRENTLY`** | Xây index không khoá thao tác ghi — bắt buộc trên production |
| **`search_path`** | Danh sách schema Postgres tìm khi tên không kèm schema; thủ phạm lỗi `type "vector" does not exist` |
| **`registerTypes` / `register_vector`** | Đăng ký kiểu vector ở client Node.js / Python — **`registerTypes` số nhiều** |
| **Iterative index scan** | Từ 0.8.0 — tự quét thêm khi lọc làm mất kết quả |
| **`SET LOCAL`** | Đặt tham số chỉ trong giao dịch; bắt buộc khi dùng connection pool |
| **`pgvector/pgvector:0.8.3-pg18`** | Image Docker chính thức, ghim đủ cả hai phiên bản |

### 5.2. Mười ý cốt lõi

1. **Cài file và bật extension là HAI bước khác nhau.** `make install` (hoặc `apt install`) rồi mới `CREATE EXTENSION`, và phải bật ở **từng database**.
2. Yêu cầu tối thiểu: **PostgreSQL 13+** (không phải 12). Thực tế nên dùng 16/17/18.
3. Cách cài dễ nhất là **Docker** hoặc **dịch vụ có sẵn**. Biên dịch từ mã nguồn là phương án cuối cùng.
4. Cột `vector(n)` phải **khớp chính xác số chiều model**. Đổi model là phải sửa cột và tạo lại toàn bộ vector.
5. **`<=>` là cosine DISTANCE, không phải similarity.** Nhỏ = gần. Sắp xếp **tăng dần**.
6. **Ops class của index phải khớp toán tử truy vấn**, nếu không index bị bỏ qua âm thầm, không báo lỗi.
7. **Có index không có nghĩa index được dùng.** Luôn kiểm bằng `EXPLAIN ANALYZE`. Index chỉ chạy khi có `ORDER BY` + `LIMIT` + toán tử khoảng cách **thuần** tăng dần.
8. Client phải **đăng ký kiểu** (`registerTypes`/`register_vector`), nếu không app nhận về chuỗi thay vì mảng.
9. Thêm `WHERE` vào truy vấn vector có thể làm **mất kết quả** — vì lọc áp dụng *sau* khi quét index. Giải bằng iterative scan hoặc index trên cột lọc.
10. Production: **không biên dịch tay**, ghim phiên bản đầy đủ, migration idempotent, `CREATE INDEX CONCURRENTLY`, và **theo dõi bản vá bảo mật**.

### 5.3. Mental models — cách tư duy để trả lời trôi chảy

- **"Cài app khác với mở app"** — cài file và `CREATE EXTENSION` là hai bước, đừng gộp.
- **"Có index khác với dùng index"** — luôn xác nhận bằng `EXPLAIN ANALYZE`.
- **"Similarity là độ thân thiết, distance là khoảng cách nhà"** — hai thang đo ngược chiều, đừng nhầm.
- **"Model là đầu bếp, pgvector là tủ lạnh"** — pgvector không tạo embedding, chỉ lưu và tìm.
- **"Biên dịch tay = sập ở máy khác"** — vì `-march=native`; dùng Docker hoặc gói OS.
- **"Ghim phiên bản như mọi thư viện"** — pgvector là dependency có vòng đời, không phải thứ build lúc triển khai.
- **"Đăng ký kiểu là cây cầu"** — không có nó, app nhận chuỗi ký tự thay vì mảng số.
- **"Chỉ cộng thêm, không sửa dữ liệu cũ"** — đó là lý do kế hoạch triển khai lui lại được sạch sẽ.

### 5.4. Lệnh cần thuộc lòng

**(a) Từ số 0 đến chạy được, bằng Docker:**

```bash
docker run -d --name pgv -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg18
docker exec -it pgv psql -U postgres
```

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id bigserial PRIMARY KEY, content text, embedding vector(384));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
```

**(b) Thêm và truy vấn:**

```sql
INSERT INTO items (content, embedding) VALUES ('xin chào', '[0.1, 0.2, ...]');

SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 5;

-- Muốn hiển thị similarity mà VẪN dùng được index:
SELECT content, 1 - (embedding <=> '[...]') AS similarity
FROM items
ORDER BY embedding <=> '[...]'      -- sắp xếp bằng toán tử THUẦN
LIMIT 5;
```

**(c) Kiểm chứng — ba lệnh phải thuộc:**

```sql
\dx vector                                                    -- extension có chưa, bản nào
SELECT count(*) FROM items WHERE embedding IS NULL;           -- dữ liệu có đủ không
EXPLAIN ANALYZE SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;  -- index có chạy không
```

**(d) Node.js — chú ý `registerTypes` số nhiều:**

```javascript
const pgvector = require('pgvector/pg');
await pgvector.registerTypes(client);                    // BẮT BUỘC, sau khi connect
await client.query('INSERT INTO items (embedding) VALUES ($1)', [pgvector.toSql(vec)]);
```

**(e) Python:**

```python
from pgvector.psycopg import register_vector
register_vector(conn)
conn.execute("SELECT content FROM items ORDER BY embedding <=> %s LIMIT 5", (vec,))
```

**(f) Triển khai lên production:**

```sql
CREATE EXTENSION IF NOT EXISTS vector;                  -- qua migration, tài khoản admin
ALTER TABLE products ADD COLUMN embedding vector(1536); -- nullable, nhanh
-- (cập nhật luồng ghi, rồi nạp bù theo lô có nghỉ)
CREATE INDEX CONCURRENTLY ON products USING hnsw (embedding vector_cosine_ops);
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời

**1. "Cài pgvector cần gì và làm thế nào?"**

PostgreSQL 13 trở lên. Hai bước: cài file (Docker, `apt`, dịch vụ có sẵn, hoặc biên dịch từ mã nguồn), rồi `CREATE EXTENSION IF NOT EXISTS vector;` trong **từng database**. Kiểm chứng bằng `\dx vector`. Điểm ghi thêm: nói rõ nên chọn Docker hoặc gói OS thay vì biên dịch tay, và giải thích được vì sao.

**2. [BẪY] "`make install` xong là dùng được kiểu `vector` chưa?"**

**Chưa.** `make install` chỉ chép file `.so` và `.sql` vào thư mục Postgres. Vẫn phải `CREATE EXTENSION` trong database. Hai bước ở hai tầng khác nhau — tầng hệ điều hành và tầng database.

**3. [BẪY] "`<=>` trả về cosine similarity, đúng không?"**

Không. Nó trả về **cosine distance**, bằng `1 − cosine similarity`. Nhỏ hơn nghĩa là gần hơn, nên sắp xếp **tăng dần** (mặc định của SQL) để lấy kết quả giống nhất. Thêm `DESC` sẽ cho ra chính xác những kết quả khác biệt nhất — mà không hề báo lỗi.

**4. [BẪY] "Tôi `CREATE EXTENSION` rồi mà vẫn báo `type "vector" does not exist`."**

Ba khả năng: (a) extension nằm ở schema không có trong `search_path` — kiểm bằng `\dx vector` và `SHOW search_path`; (b) bật ở database khác với database ứng dụng đang kết nối; (c) có nhiều bản Postgres trên máy và file rơi vào thư mục của bản không chạy.

**5. "Vì sao có index HNSW mà truy vấn vẫn `Seq Scan`?"**

Bốn nguyên nhân: (a) truy vấn không đúng dạng — thiếu `ORDER BY`/`LIMIT`, hoặc `ORDER BY` là biểu thức thay vì toán tử thuần tăng dần; (b) bảng quá nhỏ nên quét thẳng thực sự rẻ hơn — **đây không phải lỗi**; (c) ops class không khớp toán tử; (d) thống kê bảng đã cũ, chạy `ANALYZE`. Xác nhận bằng `EXPLAIN ANALYZE`, và có thể thử ép bằng `SET LOCAL enable_seqscan = off`.

**6. [BẪY] "Tôi thêm `WHERE category_id = 123` vào truy vấn vector và bị mất kết quả. Vì sao?"**

Vì với index xấp xỉ, **lọc được áp dụng SAU khi quét index**. HNSW lấy khoảng 40 ứng viên (mặc định `hnsw.ef_search`), rồi mới lọc — nếu danh mục chiếm 10% dữ liệu thì trung bình chỉ còn khoảng 4 cái sống sót. Bốn hướng giải: bật `hnsw.iterative_scan` (từ 0.8.0), đánh index thường lên cột lọc, dùng index có điều kiện, hoặc chia bảng theo phân vùng.

**7. "Kết nối pgvector từ ứng dụng thế nào?"**

Cài thư viện client (`pgvector` cho Node.js, `pgvector[psycopg]` cho Python), **đăng ký kiểu** ngay sau khi kết nối (`registerTypes` — số nhiều — hoặc `register_vector`), rồi truy vấn bình thường với tham số `$1`/`%s`. Không đăng ký thì app nhận về chuỗi ký tự thay vì mảng số. Với connection pool, phải đăng ký cho mỗi kết nối mới qua `pool.on('connect', ...)`.

**8. [OPS] "Thêm pgvector vào Postgres production không downtime?"**

Cài gói hoặc dùng dịch vụ có sẵn (không biên dịch tay) → `CREATE EXTENSION` qua migration bằng tài khoản admin → `ADD COLUMN` cho phép trống (nhanh, không viết lại bảng) → **cập nhật luồng ghi trước** → nạp bù theo lô có nghỉ → `CREATE INDEX CONCURRENTLY` → kiểm chứng ba thứ → đồng bộ phiên bản trên mọi bản sao. Và có kế hoạch lui viết sẵn từ đầu.

**9. [STAFF] "Có nên biên dịch pgvector từ mã nguồn trên production không?"**

Không. Lý do: `-march=native` khiến file chỉ chạy đúng trên CPU giống máy biên dịch (lỗi *illegal instruction* ở máy khác); không tái lập được; không lui lại sạch; các máy dễ lệch phiên bản; và máy production không nên có sẵn trình biên dịch. Thay bằng ghim tag Docker đầy đủ (`0.8.3-pg18`, không phải `pg18` trôi) hoặc ghim phiên bản gói OS qua công cụ quản lý cấu hình.

**10. [STAFF] "Bạn quản lý vòng đời của pgvector thế nào?"**

Coi nó như mọi dependency có phiên bản: ghim rõ, theo dõi changelog, rà soát định kỳ mỗi quý, kiểm thử nâng cấp trên staging trước, và giữ đồng bộ phiên bản trên mọi máy chủ kể cả bản sao. Ví dụ cụ thể để dẫn chứng: bản 0.8.2 đầu năm 2026 vá một lỗi tràn bộ nhớ đệm trong xây index HNSW song song có thể làm rò rỉ dữ liệu hoặc sập server — đội nào không theo dõi thì chạy bản lỗi rất lâu mà không biết.

**11. "Model của bạn cho vector 3072 chiều. Có vấn đề gì không?"**

Có. Kiểu `vector` lưu được tới 16.000 chiều nhưng **index HNSW chỉ đánh được tới 2.000 chiều**. Giải bằng `halfvec` (tới 4.000 chiều, đồng thời giảm gần nửa dung lượng), lượng tử hoá nhị phân kèm bước xếp hạng lại, hoặc giảm số chiều ngay khi gọi model nếu model hỗ trợ.

### 5.6. Câu chốt "one-liner" đắt giá

- *"Installing the binary and enabling the extension are two different steps — `make install`, then `CREATE EXTENSION`, in every database."*
- *"`<=>` is cosine distance, not similarity — smaller is closer, so you ORDER BY ascending."*
- *"Having an index doesn't mean it's used — always confirm with EXPLAIN ANALYZE."*
- *"The index only kicks in with ORDER BY plus LIMIT on a bare distance operator — wrap it in an expression and you silently lose it."*
- *"Don't build pgvector from source in production; pin a Docker tag or an OS package like any other dependency."*
- *"Register the vector type in your client, or your app gets back a string instead of an array."*
- *"Filtering is applied after the index scan, which is why adding a WHERE clause can quietly cost you results."*
- *"On a live database: extension via migration, column nullable then backfill, index CONCURRENTLY — zero downtime by construction, and rollback by construction too."*

---

## 📌 Ghi chú cuối

### Bài tập thực hành — làm trọn một vòng trong 30 phút

Đây là bài tập giá trị nhất bạn có thể làm sau khi đọc xong. Nó biến kiến thức trên giấy thành kiến thức thật.

```bash
# 1. Dựng môi trường
docker run -d --name pgv -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg18
docker exec -it pgv psql -U postgres
```

```sql
-- 2. Bật và kiểm chứng
CREATE EXTENSION IF NOT EXISTS vector;
\dx vector

-- 3. Tạo bảng (384 chiều, khớp all-MiniLM-L6-v2)
CREATE TABLE items (id bigserial PRIMARY KEY, content text, embedding vector(384));
```

Rồi từ một script Python hoặc Node.js: sinh embedding thật cho 20 câu bằng model từ giáo trình embedding, nhét vào bảng, và chạy một truy vấn tìm kiếm. Sau đó:

```sql
-- 4. Đánh index và kiểm tra nó có thực sự được dùng
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
EXPLAIN ANALYZE SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 5;
```

💡 **Với 20 hàng, nhiều khả năng bạn sẽ thấy `Seq Scan` chứ không phải `Index Scan`** — và điều đó **đúng**, không phải lỗi. Bảng quá nhỏ nên quét thẳng rẻ hơn (Mục 3.2). Hãy thử nhét vào 100.000 hàng rồi chạy lại: lúc đó bạn sẽ thấy `Index Scan`. **Tự tay chứng kiến sự chuyển đổi này là cách hiểu Mục 3.2 sâu hơn mọi đoạn văn tôi viết.**

### Nối mạch với các bài khác trong series

```
   ┌──────────────────┐   ┌───────────────────┐   ┌──────────────────┐
   │  Bài embedding   │   │  Bài pgvector     │   │    BÀI NÀY       │
   │  Tạo ra vector   │ → │  lý thuyết        │ → │  Cài & dùng      │
   │  (model AI)      │   │  (HNSW, cosine)   │   │  (tay chân)      │
   └──────────────────┘   └───────────────────┘   └──────────────────┘
            │                       │                      │
            └───────────────────────┴──────────────────────┘
                                    ↓
                    Semantic search / RAG chạy được thật
```

Hai từ trong sơ đồ chưa được giải thích, giải thích luôn ở đây:

> **Semantic search (tìm kiếm ngữ nghĩa):** tìm kiếm theo **ý nghĩa** thay vì theo mặt chữ. Gõ "áo phông" vẫn ra "áo thun" — đúng nỗi đau mở đầu ở Mục 1.1.
>
> **RAG (Retrieval-Augmented Generation — "sinh văn bản có tra cứu bổ trợ"):** cách làm chatbot trả lời dựa trên tài liệu riêng của bạn, gồm hai bước. Bước một, khi người dùng hỏi, hệ thống *tra cứu* trong kho tài liệu công ty để tìm vài đoạn liên quan nhất — và nó tra bằng đúng cơ chế bạn vừa học trong bài này. Bước hai, nó đưa các đoạn tìm được cùng câu hỏi cho một model ngôn ngữ lớn để *viết ra* câu trả lời. Nhờ vậy chatbot trả lời dựa trên tài liệu thật thay vì bịa.

Bài embedding cho bạn *nguyên liệu*. Bài pgvector lý thuyết cho bạn *hiểu vì sao*. Bài này cho bạn *làm được*. Ba mảnh ghép mới thành một hệ thống hoàn chỉnh.

### Nhớ kiểm chứng lại khi ôn

- **Phiên bản pgvector hiện tại là 0.8.3.** Bản 0.8.2 vá lỗi bảo mật CVE-2026-3172; đừng chạy bản cũ hơn.
- **README chính thức là nguồn đúng nhất:** `github.com/pgvector/pgvector`. Phiên bản Postgres tối thiểu, danh sách tag Docker, và giới hạn số chiều đều có thể đổi.
- **Tên hàm client có thể đổi theo phiên bản.** Hiện tại là `registerTypes` (số nhiều) bên Node.js — tài liệu cũ ghi `registerType`. Luôn kiểm README của `pgvector-node` trước khi viết code.
- **Đọc changelog trước mỗi lần nâng cấp.** Một số bản có ghi chú yêu cầu xây lại index.

### Học tiếp gì sau bài này

- **Hybrid search (tìm kiếm lai)** — kết hợp tìm kiếm vector với **full-text search** (tìm theo từ khoá — Postgres có sẵn, giỏi bắt chính xác mã sản phẩm, tên riêng, số hiệu, những thứ vector lại yếu). Hai kiểu bù trừ cho nhau nên kết hợp luôn tốt hơn từng cái riêng lẻ. Cách trộn hai bảng xếp hạng gọi là **Reciprocal Rank Fusion** — README có ví dụ sẵn.
- **Tinh chỉnh HNSW** — ba tham số `m`, `ef_construction`, `ef_search` ảnh hưởng thế nào tới đánh đổi giữa tốc độ, độ chính xác và thời gian xây index.
- **Đo độ bao phủ trong production** — **recall** (độ bao phủ) là tỉ lệ kết quả đúng mà hệ thống thực sự tìm ra được; recall 0.95 nghĩa là trong 100 kết quả lẽ ra phải thấy thì tìm được 95, bỏ sót 5. Đo bằng cách so kết quả tìm xấp xỉ với tìm chính xác qua `SET LOCAL enable_indexscan = off`, để biết index của bạn đang bỏ sót bao nhiêu.
- **Mở rộng ngang (horizontal scaling)** — khi một máy chủ không còn đủ, chia dữ liệu ra nhiều máy. Citus và PgDog là hai công cụ làm việc này cho Postgres.
- **Ghép với framework** — LangChain, LlamaIndex đều có sẵn tích hợp pgvector cho ứng dụng RAG.

---

*Giáo trình soạn theo INSTRUCTION "giáo trình siêu dễ hiểu". Thông tin kỹ thuật tra cứu và kiểm chứng ngày 23/07/2026 từ README chính thức của pgvector và pgvector-node. Các con số phiên bản nên được kiểm tra lại trước khi dùng cho quyết định thật.*
