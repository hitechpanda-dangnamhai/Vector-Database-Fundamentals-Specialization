# Cài & Dùng pgvector Thực Chiến: Install → Insert → Query → Node.js — Giáo trình Basic → Staff
*Practical pgvector: Install → Insert → Query → Node.js — A Basic-to-Staff Course*

> **Nguồn gốc — xuất xứ của bài học**  
> *Origin — where this lesson comes from*
>
> Bài gốc là video *"pgvector extension for PostgreSQL"*.  
> *The source lesson is the video "pgvector extension for PostgreSQL".*
>
> Video này thuộc Course 3, IBM Vector Database Fundamentals.  
> *This video belongs to Course 3, IBM Vector Database Fundamentals.*
>
> Bài gốc dạy theo kiểu hands-on — thực hành trực tiếp.  
> *The source lesson teaches in a hands-on style.*
>
> Bài gốc dạy yêu cầu cài đặt.  
> *The source lesson covers the installation requirements.*
>
> Bài gốc dạy cách cài trên Linux bằng git clone và make.  
> *The source lesson covers installing on Linux with git clone and make.*
>
> Bài gốc dạy cách verify — kiểm chứng kết quả cài đặt.  
> *The source lesson covers how to verify the installation.*
>
> Bài gốc dạy cách tạo bảng có cột vector — dãy số có thứ tự.  
> *The source lesson covers creating a table with a vector column.*
>
> Bài gốc dạy cách insert dữ liệu.  
> *The source lesson covers inserting data.*
>
> Bài gốc dạy cách query bằng `<=>`.  
> *The source lesson covers querying with `<=>`.*
>
> Bài gốc dạy cách dùng pgvector với Node.js.  
> *The source lesson covers using pgvector with Node.js.*
>
> Tôi giảng bám sát bài gốc.  
> *I follow the source lesson closely.*
>
> Sau đó tôi đào sâu thành hướng dẫn vận hành thật.  
> *Then I go deeper into a real operations guide.*
>
> Chỗ mở rộng được đánh dấu **[MỞ RỘNG NGOÀI BÀI GỐC]**.  
> *Extended parts carry the label **[MỞ RỘNG NGOÀI BÀI GỐC]**.*

> **Vị trí trong series — chỗ đứng của bài này**  
> *Place in the series — where this lesson sits*
>
> Đây là bản thực hành của giáo trình pgvector khái niệm.  
> *This is the hands-on version of the conceptual pgvector course.*
>
> Giáo trình khái niệm là bài #3.  
> *The conceptual course is lesson #3.*
>
> Bài #3 dạy lý do và cơ chế.  
> *Lesson #3 teaches the reasons and the mechanisms.*
>
> Cơ chế đó gồm HNSW, IVFFlat và trade-off — sự đánh đổi.  
> *Those mechanisms include HNSW, IVFFlat, and the trade-offs.*
>
> Bài này dạy phần tay chân.  
> *This lesson teaches the manual work.*
>
> Bài này dạy cách cài đặt.  
> *This lesson teaches how to install.*
>
> Bài này dạy các lệnh cần gõ.  
> *This lesson teaches which commands to type.*
>
> Bài này dạy cách kết nối app.  
> *This lesson teaches how to connect an app.*
>
> Tôi chỉ nhắc lại rất ngắn về cơ chế index — chỉ mục.  
> *I only recap the index mechanism very briefly.*
>
> Bạn hãy xem lại bài #3 để biết chi tiết.  
> *Please go back to lesson #3 for the details.*

> **⚠️ Đính chính & cập nhật tới 2026**  
> *⚠️ Corrections and updates through 2026*
>
> Bài gốc có vài chỗ đã cũ.  
> *A few parts of the source lesson are outdated.*
>
> Bài gốc có vài chỗ chưa chính xác.  
> *A few parts of the source lesson are inaccurate.*
>
> **1. PostgreSQL 12 không còn đủ.**  
> *1. PostgreSQL 12 is no longer enough.*
>
> pgvector bản mới đã bỏ hỗ trợ các phiên bản Postgres cũ.  
> *Newer pgvector versions dropped support for old Postgres releases.*
>
> Hiện nay bạn cần **PostgreSQL 13 trở lên**.  
> *Today you need PostgreSQL 13 or newer.*
>
> Bản prebuilt — dựng sẵn — được khuyến nghị dùng **PostgreSQL 15+**.  
> *The prebuilt packages recommend PostgreSQL 15 or newer.*
>
> Bạn đừng theo con số "12" trong bài gốc.  
> *Do not follow the number "12" from the source lesson.*
>
> **2. Build-from-source là cách khó nhất.**  
> *2. Building from source is the hardest path.*
>
> Build-from-source nghĩa là biên dịch từ mã nguồn bằng git clone và make.  
> *Build-from-source means compiling the code yourself with git clone and make.*
>
> Bài gốc chỉ dạy đúng cách này.  
> *The source lesson teaches only this one path.*
>
> Năm 2026 có nhiều cách dễ hơn nhiều.  
> *In 2026 there are much easier paths.*
>
> Cách thứ nhất là **Docker** với image `pgvector/pgvector:pg17`.  
> *The first path is Docker with the image `pgvector/pgvector:pg17`.*
>
> Cách thứ hai là **apt/yum** với lệnh `apt install postgresql-17-pgvector`.  
> *The second path is apt or yum with `apt install postgresql-17-pgvector`.*
>
> Cách thứ ba là dịch vụ **managed** — dịch vụ được quản lý sẵn.  
> *The third path is a managed service.*
>
> Supabase, Neon và RDS đã cài sẵn pgvector.  
> *Supabase, Neon, and RDS already ship with pgvector installed.*
>
> Trên các dịch vụ đó bạn chỉ cần chạy `CREATE EXTENSION`.  
> *On those services you only need to run `CREATE EXTENSION`.*
>
> Bạn nên ưu tiên các cách này.  
> *You should prefer these paths.*
>
> **3. `<=>` là cosine *distance*, không phải cosine *similarity*.**  
> *3. `<=>` is cosine distance, not cosine similarity.*
>
> Bài gốc gọi nó là "cosine similarity — độ tương đồng cosin".  
> *The source lesson calls it "cosine similarity".*
>
> Cách gọi đó chưa chuẩn.  
> *That naming is not correct.*
>
> Công thức là `cosine_distance = 1 − cosine_similarity`.  
> *The formula is `cosine_distance = 1 − cosine_similarity`.*
>
> Giá trị nhỏ hơn nghĩa là giống nhau hơn.  
> *A smaller value means more similar.*
>
> Vì vậy `ORDER BY ... <=> ... ASC` đưa kết quả giống nhất lên đầu.  
> *So `ORDER BY ... <=> ... ASC` puts the most similar result first.*
>
> **4. Node.js cần một bước nữa.**  
> *4. Node.js needs one more step.*
>
> Bạn chạy `require('pgvector/pg')` trước.  
> *You call `require('pgvector/pg')` first.*
>
> Sau đó bạn phải **đăng ký kiểu** với client.  
> *After that you must register the type with the client.*
>
> Chỉ khi đó driver mới parse `vector` thành mảng JS đúng cách.  
> *Only then does the driver parse `vector` into a JS array correctly.*

---

## Phần 0 — 🗺️ Bản đồ bài học (Overview)
*Part 0 — 🗺️ Lesson map (Overview)*

**Bài này dạy gì**  
*What this lesson teaches*

Bài này dạy toàn bộ vòng đời thực hành của pgvector.  
*This lesson covers the entire hands-on lifecycle of pgvector.*

Bạn sẽ cài extension — phần mở rộng — vào PostgreSQL.  
*You will install the extension into PostgreSQL.*

Bạn sẽ verify nó chạy được.  
*You will verify that it works.*

Bạn sẽ tạo bảng có cột `vector`.  
*You will create a table with a `vector` column.*

Bạn sẽ insert embedding — vector biểu diễn ngữ nghĩa.  
*You will insert embeddings.*

Bạn sẽ query k bản ghi gần nhất bằng toán tử distance — khoảng cách.  
*You will query the k nearest records with a distance operator.*

Bạn sẽ kết nối từ app Node.js hoặc Python.  
*You will connect from a Node.js app or a Python app.*

Kết thúc bài bạn có một database vector chạy được thật.  
*At the end you have a real working vector database.*

Bạn không chỉ hiểu lý thuyết.  
*You do not just understand the theory.*

**Vấn đề nó giải quyết**  
*The problem it solves*

Hiểu HNSW và cosine là một chuyện.  
*Understanding HNSW and cosine is one thing.*

Cài được pgvector và query từ code là chuyện khác.  
*Installing pgvector and querying from code is another thing.*

Đây là chỗ người mới hay tắc.  
*This is where beginners often get stuck.*

Người mới gặp lỗi khi build.  
*Beginners hit errors when they build.*

Người mới gặp lỗi `type "vector" does not exist`.  
*Beginners hit the error `type "vector" does not exist`.*

Người mới thấy index không được dùng.  
*Beginners find that the index is not used.*

Người mới thấy app không parse được vector.  
*Beginners find that the app cannot parse the vector.*

Bài này gỡ từng nút một.  
*This lesson untangles each knot one by one.*

**Học xong bạn sẽ làm được**  
*What you will be able to do*

Bạn nêu đúng yêu cầu cài đặt là PostgreSQL 13 trở lên.  
*You can state the install requirement of PostgreSQL 13 or newer.*

Bạn chọn được cách cài phù hợp trong Docker, apt, source, managed.  
*You can pick the right install path among Docker, apt, source, and managed.*

Bạn cài và verify được extension `vector`.  
*You can install and verify the `vector` extension.*

Bạn bật được extension `vector` trong một database.  
*You can enable the `vector` extension inside a database.*

Bạn tạo được bảng có cột `vector(n)` khớp số chiều model.  
*You can create a table with a `vector(n)` column matching the model dimensions.*

Bạn insert và query được bằng `<=>`, `<->`, `<#>`.  
*You can insert and query using `<=>`, `<->`, and `<#>`.*

Bạn kết nối được pgvector từ Node.js và Python.  
*You can connect to pgvector from Node.js and Python.*

Bạn đăng ký kiểu dữ liệu đúng cách.  
*You can register the data type correctly.*

Bạn xử lý được các gotcha cài đặt phổ biến.  
*You can handle the common installation gotchas.*

Bạn vận hành được ở production.  
*You can run it in production.*

Việc vận hành gồm Docker, `CREATE INDEX CONCURRENTLY` và version management.  
*Operations include Docker, `CREATE INDEX CONCURRENTLY`, and version management.*

**Mạch basic → staff**  
*The basic-to-staff path*

🟢 **Basic** bắt đầu từ yêu cầu PostgreSQL 13 trở lên.  
*🟢 Basic starts from the PostgreSQL 13+ requirement.*

Tiếp theo là cách cài dễ nhất bằng Docker hoặc managed.  
*Next comes the easiest install with Docker or a managed service.*

Sau đó là `CREATE EXTENSION` và bước verify.  
*Then come `CREATE EXTENSION` and the verify step.*

Cuối cùng là bảng, insert và query đầu tiên.  
*Finally come the first table, insert, and query.*

🟡 **Intermediate** so sánh mọi cách cài.  
*🟡 Intermediate compares every install path.*

Phần này dạy verify sâu hơn.  
*This part teaches deeper verification.*

Phần này dạy các toán tử distance.  
*This part teaches the distance operators.*

Phần này dạy client Node.js và Python.  
*This part teaches the Node.js and Python clients.*

Phần này dạy các gotcha cài đặt.  
*This part teaches the installation gotchas.*

🔴 **Advanced** mở nắp capo của build-from-source.  
*🔴 Advanced looks under the hood of build-from-source.*

Phần này nói về PGXS, file `.so` và cờ `-march=native`.  
*This part covers PGXS, the `.so` file, and the `-march=native` flag.*

Phần này nói về schema và `search_path`.  
*This part covers schema and `search_path`.*

Phần này nói về generated column.  
*This part covers generated columns.*

Phần này dùng `EXPLAIN ANALYZE` để chắc chắn index được dùng.  
*This part uses `EXPLAIN ANALYZE` to confirm the index is used.*

Phần này nói về việc upgrade extension.  
*This part covers upgrading the extension.*

🟣 **Staff** nói về cài đặt và vận hành production.  
*🟣 Staff covers production installation and operations.*

Chủ đề gồm Docker trong prod, migration, CI, security, reproducibility.  
*The topics include Docker in prod, migration, CI, security, and reproducibility.*

Phần này giải thích lý do không nên build source trên prod.  
*This part explains the reason for not building from source in prod.*

Phần này đưa ra câu hỏi ops và system design.  
*This part gives ops and system design questions.*

🎯 **Cheatsheet** gồm keywords và core concepts.  
*🎯 The cheatsheet holds keywords and core concepts.*

Cheatsheet cũng gồm code cần thuộc lòng.  
*The cheatsheet also holds code you should memorize.*

Cheatsheet có câu hỏi phỏng vấn và one-liner.  
*The cheatsheet has interview questions and one-liners.*

---

## Phần 1 — 🟢 BASIC (Nền tảng)
*Part 1 — 🟢 BASIC (Foundations)*

### 1.1. Yêu cầu cài đặt
*1.1. Installation requirements*

Bạn cần **PostgreSQL 13 trở lên**.  
*You need PostgreSQL 13 or newer.*

Bài gốc nói con số 12.  
*The source lesson mentions the number 12.*

Con số đó nay đã bị bỏ.  
*That number has now been dropped.*

Bản prebuilt khuyến nghị **PostgreSQL 15+**.  
*The prebuilt packages recommend PostgreSQL 15 or newer.*

Bạn hãy kiểm tra phiên bản bằng lệnh sau.  
*Please check your version with the following command.*

```sql
SELECT version();
```

Bạn lấy **pgvector** từ repo chính thức.  
*You get pgvector from the official repository.*

Địa chỉ repo là `https://github.com/pgvector/pgvector`.  
*The repository address is `https://github.com/pgvector/pgvector`.*

Bạn có thể chọn build từ source.  
*You may choose to build from source.*

Khi đó bạn cần trình biên dịch C tên `build-essential`.  
*In that case you need the C compiler package `build-essential`.*

Bạn cũng cần header của Postgres tên `postgresql-server-dev-XX`.  
*You also need the Postgres headers named `postgresql-server-dev-XX`.*

Thông thường bạn không cần build tay.  
*Normally you do not need to build by hand.*

Bạn hãy xem mục 1.2 để biết cách dễ hơn.  
*Please see section 1.2 for the easier paths.*

### 1.2. Cách cài DỄ NHẤT (khác bài gốc — nên dùng cái này trước)
*1.2. The EASIEST install (different from the source lesson — try this first)*

Bài gốc dạy build-from-source.  
*The source lesson teaches build-from-source.*

Cách đó khó và dễ lỗi.  
*That path is hard and error-prone.*

Với người mới, ba cách dưới đây gọn hơn nhiều.  
*For beginners, the three paths below are much simpler.*

**(a) Docker — nhanh nhất để học và thử (khuyến nghị)**  
*(a) Docker — the fastest way to learn and experiment (recommended)*

```bash
# Image chính thức đã cài sẵn pgvector, chọn phiên bản Postgres qua tag
# The official image ships with pgvector, pick the Postgres version by tag
docker run -d --name pgv \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  pgvector/pgvector:pg17
# Vào psql:
# Open psql:
docker exec -it pgv psql -U postgres
```

**(b) Gói hệ điều hành (production trên máy thật)**  
*(b) Operating system packages (production on real machines)*

```bash
# Debian/Ubuntu (thay 17 bằng phiên bản Postgres của bạn)
# Debian/Ubuntu (replace 17 with your own Postgres version)
sudo apt install postgresql-17-pgvector
# RHEL/Rocky/Fedora
# RHEL/Rocky/Fedora
sudo dnf install pgvector_17
```

**(c) Managed — không cài gì cả**  
*(c) Managed — nothing to install*

Supabase, Neon và AWS RDS/Aurora đã có sẵn pgvector.  
*Supabase, Neon, and AWS RDS/Aurora already include pgvector.*

Bạn chỉ cần làm bước `CREATE EXTENSION` ở mục 1.3.  
*You only need the `CREATE EXTENSION` step in section 1.3.*

### 1.3. Bật extension trong database + verify
*1.3. Enable the extension in a database and verify it*

Bạn có thể cài pgvector bằng bất kỳ cách nào.  
*You may install pgvector by any path.*

Sau khi cài, bạn phải *bật* extension trong **mỗi database** muốn dùng.  
*After installing, you must enable the extension in every database you want to use.*

```sql
-- Bật (idempotent: chạy lại không lỗi). Cần quyền CREATE hoặc superuser.
-- Enable it (idempotent: rerunning is safe). Requires CREATE rights or superuser.
CREATE EXTENSION IF NOT EXISTS vector;

-- Verify: extension đã có chưa & version bao nhiêu
-- Verify: whether the extension exists and which version it is
\dx vector
-- hoặc:
-- or:
SELECT extname, extversion FROM pg_extension WHERE extname = 'vector';
```

Bạn thấy dòng `vector | 0.8.x` trong kết quả.  
*You see the line `vector | 0.8.x` in the output.*

Đó là dấu hiệu thành công.  
*That is the sign of success.*

### 1.4. Tạo bảng có cột vector — số chiều PHẢI khớp model
*1.4. Create a table with a vector column — the dimensions MUST match the model*

```sql
CREATE TABLE items (
    id        serial PRIMARY KEY,
    content   text,
    embedding vector(512)     -- 512 = số chiều model sinh embedding của bạn
                              -- 512 = the dimension count of your embedding model
);
```

> Bài gốc nói đúng điểm cốt lõi.  
> *The source lesson gets the core point right.*
>
> **Độ dài vector 512 phải khớp output của model.**  
> *The vector length of 512 must match the model output.*
>
> Bạn đổi model.  
> *You change the model.*
>
> Khi đó số chiều cũng đổi theo.  
> *The dimension count then changes too.*
>
> Bạn phải sửa lại cột.  
> *You must alter the column.*
>
> Bạn cũng phải re-embed toàn bộ dữ liệu.  
> *You must also re-embed all the data.*
>
> Bạn nhét một vector 384 chiều vào `vector(512)`.  
> *You push a 384-dimension vector into `vector(512)`.*
>
> Postgres sẽ báo lỗi ngay lập tức.  
> *Postgres raises an error immediately.*

### 1.5. Insert & query đầu tiên
*1.5. Your first insert and query*

```sql
-- Insert: vector viết dạng chuỗi '[a,b,c,...]'
-- Insert: a vector is written as the string '[a,b,c,...]'
INSERT INTO items (content, embedding) VALUES
  ('the capital of the UK is London',        '[...512 số...]'),
  ('the German railways are Deutsche Bahn',  '[...512 số...]');

-- Query: tìm 1 câu GIỐNG NHẤT với embedding của câu tìm kiếm
-- Query: find the single sentence closest to the search embedding
SELECT content
FROM items
ORDER BY embedding <=> '[...embedding câu tìm kiếm...]'   -- <=> = cosine distance
                                                          -- <=> = cosine distance
LIMIT 1;
```

> **[Đính chính bài gốc]**  
> *[Correction to the source lesson]*
>
> Bài gốc gọi `<=>` là "cosine similarity".  
> *The source lesson calls `<=>` "cosine similarity".*
>
> Cách gọi chính xác là **cosine distance**.  
> *The accurate name is cosine distance.*
>
> Công thức là `cosine distance = 1 − cosine_similarity`.  
> *The formula is `cosine distance = 1 − cosine_similarity`.*
>
> Giá trị nhỏ hơn nghĩa là gần hơn.  
> *A smaller value means closer.*
>
> Vì vậy ta dùng `ORDER BY ... ASC`.  
> *So we use `ORDER BY ... ASC`.*
>
> `ASC` là chiều sắp xếp mặc định.  
> *`ASC` is the default sort direction.*
>
> Cách này đưa câu giống nhất lên đầu.  
> *This puts the most similar sentence first.*
>
> Bạn tìm câu về thủ đô nước Anh.  
> *You search for a sentence about the capital of England.*
>
> Kết quả sẽ là "the capital of the UK is London".  
> *The result will be "the capital of the UK is London".*

Trong thực tế embedding do model sinh ra.  
*In practice the model generates the embedding.*

Bạn hãy xem giáo trình embeddings để biết chi tiết.  
*Please see the embeddings course for the details.*

Bạn không gõ tay các con số này.  
*You do not type these numbers by hand.*

Ở đây `'[...]'` là chỗ cắm mảng float model trả về.  
*Here `'[...]'` is the slot for the float array the model returns.*

### ✅ Self-check Phần 1
*✅ Self-check for Part 1*

1. Phiên bản PostgreSQL tối thiểu cho pgvector hiện nay là bao nhiêu?  
*1. What is the minimum PostgreSQL version for pgvector today?*

Đáp án không phải con số trong bài gốc.  
*The answer is not the number from the source lesson.*

2. Bạn hãy nêu 2 cách cài pgvector dễ hơn build-from-source.  
*2. Name two pgvector install paths that are easier than build-from-source.*

3. `<=>` trả về similarity hay distance?  
*3. Does `<=>` return a similarity or a distance?*

Vậy `ORDER BY` nên dùng ASC hay DESC để giống nhất lên đầu?  
*So should `ORDER BY` use ASC or DESC to put the most similar first?*

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

### 2.1. Mọi cách cài — chọn cái nào khi nào
*2.1. Every install path — when to choose which*

| Cách / Method | Lệnh cốt lõi / Core command | Dùng khi / Use it when |
|---|---|---|
| **Docker** | `docker run pgvector/pgvector:pg17` | học, dev, CI — nhanh, sạch, reproducible / learning, dev, CI — fast, clean, reproducible |
| **apt/yum** | `apt install postgresql-17-pgvector` | production trên VM hoặc bare-metal có sẵn Postgres / production on a VM or bare-metal with Postgres already there |
| **Managed** | (có sẵn) → `CREATE EXTENSION vector` / (already there) → `CREATE EXTENSION vector` | Supabase, Neon, RDS — khỏi vận hành / Supabase, Neon, RDS — no ops needed |
| **Homebrew/conda/PGXN/pkg/APK** | tùy nền tảng / depends on the platform | môi trường dev đặc thù / specialized dev environments |
| **Build-from-source** | `git clone && make && make install` | phiên bản chưa có gói, hoặc cần custom / versions without a package, or custom builds |

### 2.2. Build-from-source — cách bài gốc dạy (khi bắt buộc)
*2.2. Build-from-source — the path the source lesson teaches (when you must)*

Bạn chỉ dùng cách này khi không có gói dựng sẵn cho môi trường của mình.  
*Use this path only when no prebuilt package exists for your environment.*

```bash
# Cần header Postgres + trình biên dịch trước
# You need the Postgres headers and a compiler first
sudo apt install build-essential git postgresql-server-dev-17

git clone https://github.com/pgvector/pgvector.git   # tải mã nguồn
                                                     # download the source code
cd pgvector                                           # vào thư mục
                                                      # enter the directory
make                                                  # biên dịch
                                                      # compile
sudo make install                                     # cài file .so + .sql vào Postgres
                                                      # install the .so and .sql files into Postgres
# Rồi vào psql chạy: CREATE EXTENSION IF NOT EXISTS vector;
# Then open psql and run: CREATE EXTENSION IF NOT EXISTS vector;
```

`make && make install` biên dịch extension.  
*`make && make install` compiles the extension.*

Hai lệnh này chép file vào thư mục Postgres.  
*These two commands copy the files into the Postgres directory.*

Sau đó bạn vẫn phải chạy **`CREATE EXTENSION`** trong database.  
*After that you still must run `CREATE EXTENSION` inside the database.*

Cài file không phải là bật extension.  
*Installing the files is not the same as enabling the extension.*

Đây là hai bước khác nhau.  
*These are two different steps.*

Người mới hay tưởng chúng là một.  
*Beginners often think they are one step.*

### 2.3. Ba toán tử distance — dùng đúng cái
*2.3. The three distance operators — pick the right one*

pgvector cung cấp ba toán tử.  
*pgvector provides three operators.*

Ba toán tử này khớp với ba metric — độ đo khoảng cách.  
*These three operators match three metrics.*

Ba toán tử cũng khớp với ba ops class — lớp toán tử khi đánh index.  
*The three operators also match three ops classes for indexing.*

| Toán tử / Operator | Metric / Metric | Ghi chú / Note | Ops class index / Index ops class |
|---|---|---|---|
| `<->` | Euclidean (L2) / Euclidean (L2) | khoảng cách thẳng / straight-line distance | `vector_l2_ops` |
| `<=>` | Cosine distance / Cosine distance | phổ biến nhất với text / the most common choice for text | `vector_cosine_ops` |
| `<#>` | Negative inner product / Negative inner product | pgvector trả IP âm để ASC nghĩa là gần / pgvector returns a negative IP so that ASC means closest | `vector_ip_ops` |

```sql
-- Ba kiểu query tương ứng:
-- The three matching query forms:
SELECT content FROM items ORDER BY embedding <-> '[...]' LIMIT 5;  -- L2
                                                                  -- L2
SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 5;  -- cosine
                                                                  -- cosine
SELECT content FROM items ORDER BY embedding <#> '[...]' LIMIT 5;  -- inner product
                                                                  -- inner product
```

Với text embeddings, bạn mặc định dùng **cosine** qua `<=>`.  
*For text embeddings, you use cosine by default through `<=>`.*

Bạn hãy xem giáo trình embeddings để biết chi tiết từng metric.  
*Please see the embeddings course for the details of each metric.*

### 2.4. Dùng pgvector với Node.js (bám bài gốc + sửa cho đúng)
*2.4. Using pgvector with Node.js (following the source lesson, with fixes)*

```bash
npm install pg pgvector
```

```javascript
const { Client } = require('pg');
const pgvector = require('pgvector/pg');   // helper cho kiểu vector
                                           // helper for the vector type

const client = new Client({ connectionString: 'postgresql://user:pass@localhost/db' });
await client.connect();

// QUAN TRỌNG: đăng ký kiểu vector với client thì mảng JS <-> vector mới parse đúng
// IMPORTANT: register the vector type with the client so JS arrays map to vectors correctly
await pgvector.registerType(client);       // (tên hàm theo README của package)
                                           // (function name follows the package README)

// Insert: dùng pgvector.toSql() để chuyển mảng JS -> literal vector
// Insert: use pgvector.toSql() to turn a JS array into a vector literal
const emb = [/* ...512 số... */];
await client.query(
  'INSERT INTO items (content, embedding) VALUES ($1, $2)',
  ['xin chào', pgvector.toSql(emb)]
);

// Query k gần nhất
// Query for the k nearest rows
const q = pgvector.toSql([/* ...embedding câu tìm... */]);
const res = await client.query(
  'SELECT content FROM items ORDER BY embedding <=> $1 LIMIT 1',
  [q]
);
console.log(res.rows[0].content);
await client.end();
```

> Bài gốc dừng ở `npm install pgvector`.  
> *The source lesson stops at `npm install pgvector`.*
>
> Bài gốc cũng dừng ở `require('pgvector/pg')`.  
> *The source lesson also stops at `require('pgvector/pg')`.*
>
> Bước bị thiếu là **đăng ký kiểu** bằng `registerType`.  
> *The missing step is registering the type with `registerType`.*
>
> Bạn bỏ qua bước này.  
> *You skip this step.*
>
> Khi đó driver `pg` trả về vector dưới dạng chuỗi thô.  
> *Then the `pg` driver returns the vector as a raw string.*
>
> Bạn phải tự parse chuỗi đó.  
> *You must parse that string yourself.*
>
> Bạn đăng ký kiểu xong.  
> *You finish registering the type.*
>
> Sau đó bạn làm việc với mảng số một cách tự nhiên.  
> *After that you work with number arrays naturally.*

### 2.5. Dùng pgvector với Python (thêm — chuẩn công nghiệp)
*2.5. Using pgvector with Python (added — the industry standard)*

```bash
pip install "psycopg[binary]" pgvector
```

```python
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("postgresql://user:pass@localhost/db")
register_vector(conn)                      # tương đương registerType bên Node
                                           # the equivalent of registerType in Node

conn.execute("INSERT INTO items (content, embedding) VALUES (%s, %s)",
             ("xin chào", emb))            # emb là numpy array / list, tự chuyển
                                           # emb is a numpy array or list, converted automatically
row = conn.execute(
    "SELECT content FROM items ORDER BY embedding <=> %s LIMIT 1", (query_vec,)
).fetchone()
print(row[0])
```

### 2.6. [MỞ RỘNG NGOÀI BÀI GỐC] — 3 lỗi cài đặt kinh điển
*2.6. [BEYOND THE SOURCE LESSON] — three classic installation errors*

**Lỗi 1 — `type "vector" does not exist` sau khi `CREATE EXTENSION`**  
*Error 1 — `type "vector" does not exist` after `CREATE EXTENSION`*

Extension nằm ở một schema khác.  
*The extension lives in a different schema.*

Schema đó không nằm trong `search_path` — đường dẫn tìm kiếm của Postgres.  
*That schema is not in the `search_path`.*

Bạn hãy kiểm tra và sửa bằng các lệnh sau.  
*Please check and fix it with the commands below.*

```sql
\dx vector                                   -- xem extension ở schema nào
                                             -- see which schema the extension lives in
SET search_path TO public, that_schema;      -- hoặc cài vào public từ đầu:
                                             -- or install it into public from the start:
-- DROP EXTENSION vector; CREATE EXTENSION vector SCHEMA public;
-- DROP EXTENSION vector; CREATE EXTENSION vector SCHEMA public;
```

**Lỗi 2 — Cài file rồi quên `CREATE EXTENSION`**  
*Error 2 — installing the files, then forgetting `CREATE EXTENSION`*

`make install` chỉ chép file.  
*`make install` only copies files.*

Bạn phải chạy `CREATE EXTENSION` trong *mỗi* database.  
*You must run `CREATE EXTENSION` in every database.*

Đây là hai bước riêng biệt.  
*These are two separate steps.*

**Lỗi 3 — Sai lib path khi có nhiều bản Postgres**  
*Error 3 — the wrong library path when several Postgres versions exist*

Bạn build source với header của PG 15.  
*You build from source with the PG 15 headers.*

Server của bạn lại chạy PG 17.  
*Your server, however, runs PG 17.*

File `.so` khi đó nằm sai chỗ.  
*The `.so` file then sits in the wrong place.*

Lệnh `CREATE EXTENSION` sẽ báo lỗi.  
*The `CREATE EXTENSION` command then fails.*

Bạn hãy kiểm tra bằng lệnh sau.  
*Please check with the command below.*

```bash
ls -l /usr/lib/postgresql/17/lib/vector.so   # Ubuntu
                                             # Ubuntu
```

Đây là thêm một lý do để dùng Docker hoặc gói cài sẵn.  
*This is one more reason to use Docker or a prebuilt package.*

Bạn nên tránh build tay.  
*You should avoid building by hand.*

### ✅ Self-check Phần 2
*✅ Self-check for Part 2*

1. Sau `make install`, bạn đã dùng được vector chưa?  
*1. After `make install`, can you already use vector?*

Bạn còn thiếu bước nào?  
*Which step are you still missing?*

2. Ba toán tử `<->`, `<=>`, `<#>` ứng với metric nào?  
*2. Which metric does each of `<->`, `<=>`, and `<#>` correspond to?*

3. Trong Node.js, vì sao bạn phải gọi `registerType(client)` trước khi query vector?  
*3. In Node.js, why must you call `registerType(client)` before querying vectors?*

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3 — 🔴 ADVANCED (In depth)*

### 3.1. Build-from-source dưới nắp capo
*3.1. Build-from-source under the hood*

**PGXS** là hệ thống build extension của Postgres.  
*PGXS is the extension build system of Postgres.*

Lệnh `make` gọi `pg_config`.  
*The `make` command calls `pg_config`.*

`pg_config` tìm header và thư viện của bản Postgres đang cài.  
*`pg_config` locates the headers and libraries of the installed Postgres.*

Sau đó `make` biên dịch mã C thành `vector.so`.  
*Then `make` compiles the C code into `vector.so`.*

`vector.so` là một shared library — thư viện dùng chung.  
*`vector.so` is a shared library.*

`make` cũng sinh ra các file `.sql` và `.control`.  
*`make` also produces the `.sql` and `.control` files.*

**`make install`** chép `vector.so` vào thư mục `lib/`.  
*`make install` copies `vector.so` into the `lib/` directory.*

Lệnh này chép `vector.control` và `vector--*.sql` vào thư mục `extension/`.  
*This command copies `vector.control` and `vector--*.sql` into the `extension/` directory.*

Từ đó lệnh `CREATE EXTENSION` mới nhìn thấy extension.  
*Only then does `CREATE EXTENSION` see the extension.*

**`-march=native` và bản portable là hai hướng khác nhau.**  
*`-march=native` and a portable build are two different directions.*

Build tay mặc định tối ưu cho CPU của máy build.  
*A manual build optimizes for the CPU of the build machine by default.*

Cờ tối ưu đó là `-march=native`.  
*That optimization flag is `-march=native`.*

Binary chạy nhanh hơn.  
*The binary runs faster.*

Binary lại **không chạy được trên CPU khác**.  
*The binary cannot run on a different CPU.*

Bạn bê binary sang máy khác.  
*You move the binary to another machine.*

Khi đó bạn gặp lỗi illegal instruction.  
*You then hit an illegal instruction error.*

Image Docker chính thức build với `OPTFLAGS=""`.  
*The official Docker image builds with `OPTFLAGS=""`.*

Cấu hình này tắt `-march=native`.  
*This setting turns off `-march=native`.*

Nhờ vậy image chạy được trên mọi CPU.  
*The image therefore runs on any CPU.*

Đây là sự đánh đổi một chút tốc độ lấy tính di động.  
*This trades a little speed for portability.*

Bạn hiểu điều này.  
*You understand this point.*

Binary build ở máy A crash ở máy B.  
*A binary built on machine A crashes on machine B.*

Khi đó bạn giải thích được lý do.  
*You can then explain the reason.*

### 3.2. Verify index THỰC SỰ được dùng — `EXPLAIN ANALYZE`
*3.2. Verify that the index is REALLY used — `EXPLAIN ANALYZE`*

Bạn cài xong pgvector.  
*You finish installing pgvector.*

Sau đó bạn tạo bảng.  
*Then you create the table.*

Sau đó bạn tạo index.  
*Then you create the index.*

Cơ chế HNSW và IVFFlat nằm ở giáo trình #3 và giáo trình indexing.  
*The HNSW and IVFFlat mechanisms live in lesson #3 and the indexing course.*

Việc *có index* không có nghĩa là *index được dùng*.  
*Having an index does not mean the index gets used.*

```sql
EXPLAIN ANALYZE
SELECT id FROM items ORDER BY embedding <=> '[...]' LIMIT 10;
```

Bạn thấy dòng **`Index Scan using ..._hnsw...`**.  
*You see the line `Index Scan using ..._hnsw...`.*

Đó là dấu hiệu tốt.  
*That is a good sign.*

Index đang chạy.  
*The index is being used.*

Bạn thấy dòng **`Seq Scan`**.  
*You see the line `Seq Scan`.*

Khi đó planner — bộ lập kế hoạch truy vấn — đã bỏ qua index.  
*In that case the planner skipped the index.*

Nguyên nhân thứ nhất là bảng quá nhỏ.  
*The first cause is a table that is too small.*

Với bảng nhỏ, seq scan rẻ hơn.  
*On a small table, a seq scan is cheaper.*

Nguyên nhân thứ hai là ops class không khớp toán tử.  
*The second cause is an ops class that does not match the operator.*

Nguyên nhân thứ ba là tham số `enable_seqscan` bị đặt khác.  
*The third cause is a different `enable_seqscan` setting.*

Bạn hãy đọc kỹ output trước khi kết luận là sai cấu hình.  
*Please read the output carefully before concluding that the setup is wrong.*

### 3.3. Generated column — tự động hóa embedding-adjacent (mẫu hay dùng)
*3.3. Generated columns — automating what sits next to the embedding (a common pattern)*

Vector của bạn sinh từ một nguồn cố định.  
*Your vectors come from a fixed source.*

Khi đó một cột thường kèm index là đủ.  
*In that case a plain column plus an index is enough.*

Bạn cần ghép thêm metadata — dữ liệu mô tả đi kèm.  
*You need to attach metadata as well.*

Bạn cũng cần đảm bảo tính nhất quán.  
*You also need to guarantee consistency.*

Khi đó người ta hay dùng một bảng gọn kèm ràng buộc số chiều.  
*People then use a compact table with a dimension constraint.*

```sql
CREATE TABLE docs (
    id        bigserial PRIMARY KEY,
    tenant_id bigint NOT NULL,
    content   text,
    embedding vector(1536)
);
CREATE INDEX ON docs (tenant_id);                              -- filter nhanh
                                                               -- fast filtering
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops); -- vector search
                                                               -- vector search
```

pgvector không tự sinh embedding.  
*pgvector does not generate embeddings itself.*

Bạn insert vector lấy từ model.  
*You insert vectors that come from a model.*

Bạn muốn database tự sinh embedding.  
*You may want the database to generate embeddings itself.*

Đó là tính năng của HeatWave.  
*That is a HeatWave feature.*

Bạn hãy xem giáo trình so sánh nền tảng.  
*Please see the platform comparison course.*

### 3.4. Nâng cấp pgvector đúng cách
*3.4. Upgrading pgvector the right way*

```sql
-- Sau khi cài bản mới (cùng cách cài ban đầu), trong mỗi database:
-- After installing the new version the same way as before, in every database:
ALTER EXTENSION vector UPDATE;
-- Kiểm version mới:
-- Check the new version:
SELECT extversion FROM pg_extension WHERE extname='vector';
```

Một số bản có ghi chú upgrade riêng.  
*Some releases carry their own upgrade notes.*

Ví dụ là việc tạo lại IVFFlat index sau khi lên từ bản rất cũ.  
*One example is re-creating the IVFFlat index after jumping from a very old version.*

Bạn hãy đọc CHANGELOG trước khi nâng cấp trên production.  
*Please read the CHANGELOG before upgrading in production.*

### 3.5. Edge cases
*3.5. Edge cases*

**Dimension mismatch khi insert — lệch số chiều lúc insert**  
*Dimension mismatch on insert*

Vector của bạn khác `vector(n)` của cột.  
*Your vector differs from the column's `vector(n)`.*

Postgres sẽ báo lỗi.  
*Postgres raises an error.*

Bạn hãy chuẩn hóa số chiều ở tầng app trước khi insert.  
*Please normalize the dimension count in the app layer before inserting.*

**`<#>` là *negative* inner product**  
*`<#>` is the negative inner product*

Bạn đừng ngạc nhiên khi thấy giá trị âm.  
*Do not be surprised by negative values.*

pgvector dùng số âm để ASC vẫn nghĩa là "gần nhất".  
*pgvector uses negative numbers so that ASC still means "closest".*

Postgres chỉ đánh index theo chiều ASC.  
*Postgres only indexes in ASC order.*

**`NULL` embedding**  
*A `NULL` embedding*

Hàng có `embedding IS NULL` không xuất hiện trong kết quả ANN — tìm kiếm gần đúng.  
*Rows with `embedding IS NULL` never appear in ANN results.*

Bạn nên có job kiểm tra và backfill — bù lại dữ liệu thiếu.  
*You should run a job to check and backfill them.*

**Bảng nhỏ dùng seq scan**  
*Small tables use a seq scan*

Đây không phải lỗi.  
*This is not a bug.*

Với n nhỏ, exact scan rẻ hơn.  
*For a small n, an exact scan is cheaper.*

Exact scan cũng cho recall 100%.  
*An exact scan also gives 100% recall.*

**Nhiều bản Postgres trên một host**  
*Several Postgres versions on one host*

File `.so` phải khớp bản Postgres đang chạy.  
*The `.so` file must match the running Postgres version.*

Bạn hãy xem lại lỗi 3 ở mục 2.6.  
*Please review error 3 in section 2.6.*

### ✅ Self-check Phần 3
*✅ Self-check for Part 3*

1. Vì sao binary pgvector build tay ở máy này có thể crash ở máy khác?  
*1. Why can a hand-built pgvector binary from one machine crash on another?*

Docker giải quyết chuyện đó ra sao?  
*How does Docker solve that?*

2. Bạn đã tạo HNSW index.  
*2. You have created an HNSW index.*

Bạn vẫn thấy `Seq Scan` trong `EXPLAIN ANALYZE`.  
*You still see `Seq Scan` in `EXPLAIN ANALYZE`.*

Bạn hãy nêu 2 nguyên nhân khả dĩ.  
*Please name two possible causes.*

3. Lệnh nâng cấp extension trong một database là gì?  
*3. What is the command to upgrade the extension inside a database?*

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4 — 🟣 STAFF LEVEL (Systems thinking and technical leadership)*

### 4.1. Đừng build-from-source trên production — reproducibility
*4.1. Do not build from source in production — reproducibility*

Bài gốc dạy git clone và make.  
*The source lesson teaches git clone and make.*

Ở tầm staff, **build tay trên server production là một anti-pattern**.  
*At staff level, building by hand on a production server is an anti-pattern.*

Kết quả phụ thuộc vào CPU và compiler của chính máy đó.  
*The result depends on the CPU and compiler of that machine.*

Nguyên nhân là cờ `-march=native`.  
*The cause is the `-march=native` flag.*

Kết quả đó khó tái lập.  
*That result is hard to reproduce.*

Kết quả đó cũng khó rollback — quay lui về bản cũ.  
*That result is also hard to roll back.*

Các node dễ bị lệch nhau.  
*The nodes easily drift apart.*

Bạn hãy dùng ba cách dưới đây thay thế.  
*Please use the three approaches below instead.*

**Docker image cố định tag** là cách thứ nhất.  
*A Docker image with a fixed tag is the first approach.*

Ví dụ là tag `pgvector/pgvector:pg17`.  
*One example is the tag `pgvector/pgvector:pg17`.*

Nhờ vậy mọi môi trường dev, staging và prod giống hệt nhau.  
*Every dev, staging, and prod environment is then identical.*

**Gói OS pin version** là cách thứ hai.  
*An OS package pinned to a version is the second approach.*

Ví dụ là `postgresql-17-pgvector=0.8.x`.  
*One example is `postgresql-17-pgvector=0.8.x`.*

Bạn quản lý nó qua công cụ cấu hình như Ansible hoặc Terraform.  
*You manage it through a configuration tool such as Ansible or Terraform.*

**Managed provider** là cách thứ ba.  
*A managed provider is the third approach.*

Nhà cung cấp lo phần binary cho bạn.  
*The provider takes care of the binary for you.*

Nguyên tắc là *pin phiên bản pgvector và Postgres một cách rõ ràng*.  
*The principle is to pin the pgvector and Postgres versions explicitly.*

Bạn coi pgvector như một dependency có version.  
*You treat pgvector as a dependency with a version.*

Bạn không coi nó là thứ "build lúc deploy".  
*You do not treat it as something to build at deploy time.*

### 4.2. Cài đặt & migration ở production
*4.2. Installation and migration in production*

**`CREATE EXTENSION` cần quyền cao.**  
*`CREATE EXTENSION` needs elevated privileges.*

Quyền đó là CREATE trên database hoặc quyền superuser.  
*That privilege is CREATE on the database or superuser rights.*

Bạn hãy đưa lệnh này vào migration script chạy một lần.  
*Please put this command into a migration script that runs once.*

Bạn đừng rải nó khắp nơi trong code app.  
*Do not scatter it across the app code.*

**Đánh index trên bảng lớn cần thận trọng.**  
*Indexing a large table needs care.*

Bạn hãy dùng **`CREATE INDEX CONCURRENTLY`**.  
*Please use `CREATE INDEX CONCURRENTLY`.*

Cách này không khóa write trong suốt quá trình build.  
*This approach does not lock writes during the build.*

Việc build index HNSW có thể kéo dài hàng giờ.  
*Building an HNSW index can take hours.*

**Nâng cấp cũng cần quy trình.**  
*Upgrades also need a process.*

Bạn chạy `ALTER EXTENSION vector UPDATE` trong cửa sổ bảo trì.  
*You run `ALTER EXTENSION vector UPDATE` inside a maintenance window.*

Bạn hãy test trên staging trước.  
*Please test on staging first.*

Bạn hãy đọc CHANGELOG để tìm ghi chú re-index.  
*Please read the CHANGELOG for any re-index notes.*

**Version skew — lệch phiên bản — là một rủi ro riêng.**  
*Version skew is a separate risk.*

Bạn hãy giữ pgvector đồng bộ trên primary và mọi replica.  
*Please keep pgvector in sync on the primary and every replica.*

Phiên bản khác nhau có thể gây lỗi trên replica.  
*Different versions can cause errors on a replica.*

Lỗi xảy ra khi replica áp WAL liên quan tới kiểu vector.  
*The error appears when a replica applies WAL involving the vector type.*

### 4.3. Security & multi-tenancy
*4.3. Security and multi-tenancy*

**Privilege tối thiểu là nguyên tắc đầu tiên.**  
*Least privilege is the first principle.*

App user chỉ cần quyền INSERT và SELECT trên bảng.  
*The app user only needs INSERT and SELECT on the table.*

App user không cần quyền superuser.  
*The app user does not need superuser rights.*

Bạn bật extension bằng migration do admin chạy.  
*You enable the extension through a migration run by an admin.*

Bạn không bật extension bằng app user.  
*You do not enable the extension with the app user.*

**Schema và `search_path` phải rõ ràng.**  
*The schema and `search_path` must be explicit.*

Bạn hãy xem lại lỗi 1 ở mục 2.6.  
*Please review error 1 in section 2.6.*

Cách này tránh lỗi `type vector does not exist`.  
*This avoids the `type vector does not exist` error.*

Lỗi đó hay xảy ra trong môi trường nhiều schema hoặc nhiều tenant.  
*That error is common in multi-schema or multi-tenant environments.*

**Connection pooler như PgBouncer cần thêm chú ý.**  
*A connection pooler such as PgBouncer needs extra attention.*

Lệnh `SET` toàn cục có thể rò qua connection được tái dùng.  
*A global `SET` can leak through a reused connection.*

Một ví dụ là `hnsw.ef_search`.  
*One example is `hnsw.ef_search`.*

Bạn hãy dùng `SET LOCAL` bên trong transaction.  
*Please use `SET LOCAL` inside a transaction.*

Nội dung này nối lại với giáo trình pgvector #3.  
*This point links back to pgvector lesson #3.*

### 4.4. CI/CD & test
*4.4. CI/CD and testing*

Dịch vụ Postgres trong CI nên dùng chính image `pgvector/pgvector:pgXX`.  
*The Postgres service in CI should use the same `pgvector/pgvector:pgXX` image.*

Nhờ vậy test vector chạy đúng môi trường như prod.  
*Vector tests then run in the same environment as prod.*

Migration của bạn nên idempotent — chạy lại không đổi kết quả.  
*Your migration should be idempotent.*

Lệnh `CREATE EXTENSION IF NOT EXISTS vector;` an toàn khi chạy lại.  
*The command `CREATE EXTENSION IF NOT EXISTS vector;` is safe to rerun.*

Bạn hãy test cả đường "index được dùng".  
*Please also test the "index is used" path.*

Bạn assert rằng output `EXPLAIN` có dòng `Index Scan`.  
*You assert that the `EXPLAIN` output contains `Index Scan`.*

Cách này tránh regression âm thầm về seq scan.  
*This prevents a silent regression back to seq scan.*

### 4.5. Ảnh hưởng tổ chức & giải thích cho non-technical stakeholder
*4.5. Organizational impact and explaining it to non-technical stakeholders*

**Bạn nói với PM hoặc sếp như sau.**  
*Here is how you talk to a PM or a manager.*

Thêm vector search *không cần dựng hệ thống mới*.  
*Adding vector search does not require a new system.*

Bạn chỉ bật một extension trên database sẵn có.  
*You just enable an extension on the existing database.*

Rủi ro ở mức thấp.  
*The risk is low.*

Với Docker hoặc managed, việc cài đặt tái lập được.  
*With Docker or a managed service, the installation is reproducible.*

Việc nâng cấp rõ ràng như mọi dependency.  
*Upgrades are as clear as with any dependency.*

Việc rollback cũng rõ ràng như vậy.  
*Rollbacks are just as clear.*

Cái cần đầu tư là quy trình.  
*What needs investment is the process.*

Quy trình gồm migration, đánh index không khóa và pin version.  
*The process covers migration, non-blocking indexing, and version pinning.*

Bạn không cần mua phần mềm mới.  
*You do not need to buy new software.*

Đây là cách framing bằng **rủi ro thấp cộng quy trình chuẩn**.  
*This frames the work as low risk plus a standard process.*

**Chuẩn hóa là việc tiếp theo.**  
*Standardization is the next task.*

Nhiều team trong công ty cùng dùng Postgres.  
*Several teams in the company all use Postgres.*

Khi đó bạn hãy thống nhất một cách cài duy nhất.  
*In that case, please agree on a single install path.*

Cách đó là một Docker tag hoặc một gói đã pinned.  
*That path is one Docker tag or one pinned package.*

Cách này giảm tình trạng "hoạt động trên máy tôi".  
*This reduces the "works on my machine" problem.*

Cách này cũng giảm lệch môi trường.  
*This also reduces environment drift.*

**Vận hành là việc cuối.**  
*Operations is the last task.*

Bạn coi pgvector là một dependency có vòng đời.  
*You treat pgvector as a dependency with a lifecycle.*

Bạn theo dõi CHANGELOG.  
*You follow the CHANGELOG.*

Bạn lên lịch nâng cấp.  
*You schedule upgrades.*

Bạn test trên staging.  
*You test on staging.*

### 4.6. Câu hỏi ops/system-design mẫu + hướng trả lời staff
*4.6. A sample ops/system-design question with a staff-level answer*

> **"Team muốn thêm pgvector vào một PostgreSQL production đang phục vụ khách hàng."**  
> *"The team wants to add pgvector to a production PostgreSQL that is serving customers."*
>
> **"Hệ thống không được downtime."**  
> *"The system must have no downtime."*
>
> **"Bạn triển khai thế nào?"**  
> *"How do you roll this out?"*

**Khung trả lời staff**  
*A staff-level answer framework*

1. **Clarify — làm rõ đề bài trước.**  
*1. Clarify the problem first.*

Hệ thống là self-host hay managed?  
*Is the system self-hosted or managed?*

Phiên bản Postgres là bao nhiêu?  
*Which Postgres version is it?*

Hệ thống có replica không?  
*Does the system have replicas?*

Bảng sẽ đánh index lớn cỡ nào?  
*How large is the table you will index?*

2. **Cài binary không gián đoạn.**  
*2. Install the binary without interruption.*

Với managed, bạn bật qua console hoặc qua `CREATE EXTENSION`.  
*On a managed service, you enable it through the console or `CREATE EXTENSION`.*

Với self-host, bạn cài gói `postgresql-XX-pgvector`.  
*On self-hosted machines, you install the `postgresql-XX-pgvector` package.*

Bạn không cần restart để có file.  
*You do not need a restart to get the files.*

Bạn **không** build tay trên prod.  
*You do not build by hand in prod.*

3. **Bật extension.**  
*3. Enable the extension.*

Bạn chạy `CREATE EXTENSION IF NOT EXISTS vector;`.  
*You run `CREATE EXTENSION IF NOT EXISTS vector;`.*

Lệnh này chạy qua migration do admin thực hiện.  
*This command runs through a migration performed by an admin.*

Lệnh này idempotent và rất nhanh.  
*This command is idempotent and very fast.*

4. **Thêm cột và backfill.**  
*4. Add the column and backfill.*

Bạn chạy `ALTER TABLE ADD COLUMN embedding vector(n)`.  
*You run `ALTER TABLE ADD COLUMN embedding vector(n)`.*

Cột này để nullable.  
*The column stays nullable.*

Vì vậy thao tác rất nhanh.  
*The operation is therefore very fast.*

Bạn backfill embedding theo lô ở nền qua queue.  
*You backfill the embeddings in batches in the background through a queue.*

Cách này không khóa bảng.  
*This approach does not lock the table.*

5. **Đánh index không khóa.**  
*5. Build the index without locking.*

Bạn chạy `CREATE INDEX CONCURRENTLY ... USING hnsw ...`.  
*You run `CREATE INDEX CONCURRENTLY ... USING hnsw ...`.*

Bạn chạy lệnh này sau khi backfill xong.  
*You run this command after the backfill finishes.*

Bạn canh tham số `maintenance_work_mem`.  
*You tune the `maintenance_work_mem` setting.*

6. **Verify.**  
*6. Verify.*

Bạn chạy `\dx vector`.  
*You run `\dx vector`.*

Bạn chạy `EXPLAIN ANALYZE` để chắc chắn có `Index Scan`.  
*You run `EXPLAIN ANALYZE` to confirm an `Index Scan`.*

7. **Replica và version.**  
*7. Replicas and versions.*

Bạn đảm bảo mọi replica dùng cùng version pgvector.  
*You make sure every replica runs the same pgvector version.*

8. **Rollback plan — kế hoạch lui.**  
*8. A rollback plan.*

Mọi thứ đều là migration và dependency đã pinned.  
*Everything is a migration and a pinned dependency.*

Vì vậy đường lui rất rõ ràng.  
*The way back is therefore very clear.*

Bạn drop index.  
*You drop the index.*

Bạn drop column.  
*You drop the column.*

Bạn có thể gỡ extension.  
*You can remove the extension.*

**Có kế hoạch lui là dấu hiệu của tư duy staff.**  
*Having a rollback plan is a sign of staff-level thinking.*

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN (Interview Cheatsheet)
*Part 5 — 🎯 WRAPPING UP FOR INTERVIEWS (Interview cheatsheet)*

### 5.1. Keywords bắt buộc nhớ
*5.1. Keywords you must remember*

**pgvector** là extension thêm kiểu `vector` cho PostgreSQL.  
*pgvector is the extension that adds the `vector` type to PostgreSQL.*

pgvector cũng thêm khả năng vector search.  
*pgvector also adds vector search.*

**`CREATE EXTENSION vector`** bật extension trong một database.  
*`CREATE EXTENSION vector` enables the extension inside one database.*

Bạn chạy lệnh này sau khi cài binary.  
*You run this command after installing the binary.*

**`vector(n)`** là kiểu cột lưu embedding n chiều.  
*`vector(n)` is the column type that stores an n-dimensional embedding.*

Số n phải khớp model.  
*The number n must match the model.*

**`<->`** là toán tử L2.  
*`<->` is the L2 operator.*

**`<=>`** là toán tử cosine distance.  
*`<=>` is the cosine distance operator.*

**`<#>`** là toán tử negative inner product.  
*`<#>` is the negative inner product operator.*

**ops class** gồm `vector_l2_ops`, `vector_cosine_ops`, `vector_ip_ops`.  
*The ops classes are `vector_l2_ops`, `vector_cosine_ops`, and `vector_ip_ops`.*

Ops class phải khớp toán tử khi bạn đánh index.  
*The ops class must match the operator when you build the index.*

**PGXS** là hệ build extension của Postgres.  
*PGXS is the extension build system of Postgres.*

Bạn dùng PGXS khi build từ source.  
*You use PGXS when you build from source.*

**`make && make install`** biên dịch và cài file extension.  
*`make && make install` compiles and installs the extension files.*

Hai lệnh này chưa bật extension.  
*These two commands do not enable the extension yet.*

**`\dx` và bảng `pg_extension`** dùng để verify extension và version.  
*`\dx` and the `pg_extension` table verify the extension and its version.*

**`ALTER EXTENSION vector UPDATE`** dùng để nâng cấp extension.  
*`ALTER EXTENSION vector UPDATE` upgrades the extension.*

**`CREATE INDEX CONCURRENTLY`** đánh index mà không khóa write.  
*`CREATE INDEX CONCURRENTLY` builds an index without locking writes.*

**`EXPLAIN ANALYZE`** kiểm tra index có thực sự được dùng không.  
*`EXPLAIN ANALYZE` checks whether the index is really used.*

Bạn nhìn vào `Index Scan` hoặc `Seq Scan` trong output.  
*You look for `Index Scan` or `Seq Scan` in the output.*

**registerType và register_vector** đăng ký kiểu vector ở client.  
*registerType and register_vector register the vector type on the client.*

Hai hàm này dành cho Node.js và Python.  
*These two functions are for Node.js and Python.*

**`pgvector/pgvector:pgXX`** là Docker image chính thức.  
*`pgvector/pgvector:pgXX` is the official Docker image.*

Image này đã cài sẵn pgvector.  
*This image ships with pgvector already installed.*

### 5.2. Core concepts — nếu chỉ nhớ vài điều
*5.2. Core concepts — if you only remember a few things*

1. Yêu cầu là **PostgreSQL 13 trở lên**.  
*1. The requirement is PostgreSQL 13 or newer.*

Con số đó không phải 12.  
*That number is not 12.*

Bản prebuilt khuyến nghị PostgreSQL 15 trở lên.  
*The prebuilt packages recommend PostgreSQL 15 or newer.*

2. Bạn cài binary trước.  
*2. You install the binary first.*

Sau đó bạn mới chạy `CREATE EXTENSION`.  
*Only then do you run `CREATE EXTENSION`.*

Đây là hai bước khác nhau.  
*These are two different steps.*

Bạn làm bước hai ở mỗi database.  
*You do the second step in every database.*

3. Các cách cài dễ là **Docker, apt/yum và managed**.  
*3. The easy install paths are Docker, apt/yum, and managed services.*

Build-from-source là phương án cuối cùng.  
*Build-from-source is the last resort.*

4. Cột `vector(n)` phải **khớp số chiều model**.  
*4. The `vector(n)` column must match the model dimensions.*

Bạn đổi model.  
*You change the model.*

Khi đó bạn phải sửa cột.  
*You then have to alter the column.*

Bạn cũng phải re-embed.  
*You also have to re-embed.*

5. `<=>` là **cosine distance**.  
*5. `<=>` is cosine distance.*

Giá trị nhỏ nghĩa là gần.  
*A small value means close.*

Vì vậy bạn viết `ORDER BY ... <=> ... LIMIT k` theo chiều ASC.  
*So you write `ORDER BY ... <=> ... LIMIT k` in ASC order.*

6. Ops class trong index **phải khớp** toán tử query.  
*6. The ops class in the index must match the query operator.*

Ngược lại, index sẽ bị bỏ qua.  
*Otherwise the index gets skipped.*

7. Client Node.js hoặc Python phải **đăng ký kiểu**.  
*7. The Node.js or Python client must register the type.*

Chỉ khi đó vector và mảng mới parse đúng.  
*Only then do vectors and arrays parse correctly.*

8. Bạn verify extension bằng `\dx`.  
*8. You verify the extension with `\dx`.*

Bạn verify index được dùng bằng `EXPLAIN ANALYZE`.  
*You verify index usage with `EXPLAIN ANALYZE`.*

9. Ở production, bạn **không build từ source**.  
*9. In production, you do not build from source.*

Bạn pin version qua Docker hoặc gói OS.  
*You pin the version through Docker or an OS package.*

Migration của bạn phải idempotent.  
*Your migration must be idempotent.*

Bạn dùng `CREATE INDEX CONCURRENTLY`.  
*You use `CREATE INDEX CONCURRENTLY`.*

10. Gotcha hàng đầu là `type "vector" does not exist`.  
*10. The top gotcha is `type "vector" does not exist`.*

Nguyên nhân nằm ở schema và `search_path`.  
*The cause lies in the schema and `search_path`.*

Gotcha tiếp theo là dimension mismatch.  
*The next gotcha is a dimension mismatch.*

Gotcha cuối là seq scan trên bảng nhỏ.  
*The last gotcha is a seq scan on a small table.*

### 5.3. Ideas / mental models
*5.3. Ideas and mental models*

**"Cài file khác với bật extension."**  
*"Installing the files is not the same as enabling the extension."*

Đây là hai bước.  
*These are two steps.*

Bạn đừng gộp chúng lại.  
*Do not merge them.*

**"Có index khác với dùng index."**  
*"Having an index is not the same as using it."*

Bạn hãy luôn chạy `EXPLAIN ANALYZE`.  
*Please always run `EXPLAIN ANALYZE`.*

**"Build tay dẫn tới crash lạ ở máy khác."**  
*"A hand-built binary leads to strange crashes on other machines."*

Bạn hãy dùng Docker hoặc gói để có tính di động.  
*Please use Docker or a package for portability.*

**"Pin version như mọi dependency khác."**  
*"Pin the version like any other dependency."*

pgvector không phải thứ để build lúc deploy.  
*pgvector is not something to build at deploy time.*

**"registerType là cây cầu."**  
*"registerType is the bridge."*

Không có nó, app nhận vector dưới dạng chuỗi.  
*Without it, the app receives the vector as a string.*

### 5.4. Code cần thuộc lòng
*5.4. Code you should memorize*

**(a) Cài, bật và tạo bảng — chuỗi lệnh cơ bản**  
*(a) Install, enable, and create a table — the basic command sequence*

```bash
docker run -d -e POSTGRES_PASSWORD=secret -p 5432:5432 pgvector/pgvector:pg17
```
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE items (id serial PRIMARY KEY, content text, embedding vector(512));
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
```

**(b) Insert và query**  
*(b) Insert and query*

```sql
INSERT INTO items (content, embedding) VALUES ('hello', '[...]');
SELECT content FROM items ORDER BY embedding <=> '[...]' LIMIT 1;
```

**(c) Node.js — đăng ký kiểu là bước hay bị quên**  
*(c) Node.js — registering the type is the step people forget*

```javascript
const pgvector = require('pgvector/pg');
await pgvector.registerType(client);
await client.query('INSERT INTO items (embedding) VALUES ($1)', [pgvector.toSql(vec)]);
```

### 5.5. Câu hỏi phỏng vấn thường gặp + gợi ý trả lời
*5.5. Common interview questions with suggested answers*

1. **"Cài pgvector cần gì và làm sao?"**  
*1. "What does installing pgvector need, and how do you do it?"*

Bạn cần PostgreSQL 13 trở lên.  
*You need PostgreSQL 13 or newer.*

Bạn cài binary bằng Docker, apt, yum hoặc source.  
*You install the binary with Docker, apt, yum, or source.*

Sau đó bạn chạy `CREATE EXTENSION IF NOT EXISTS vector` trong mỗi database.  
*Then you run `CREATE EXTENSION IF NOT EXISTS vector` in every database.*

Bạn verify bằng `\dx vector`.  
*You verify with `\dx vector`.*

2. **[BẪY] "`make install` xong là dùng được vector chưa?"**  
*2. [TRAP] "After `make install`, can you use vector already?"*

Câu trả lời là chưa.  
*The answer is no.*

Lệnh đó mới chỉ chép file.  
*That command only copies files.*

Bạn phải chạy `CREATE EXTENSION` trong database.  
*You must run `CREATE EXTENSION` in the database.*

Đây là hai bước riêng.  
*These are two separate steps.*

3. **[BẪY] "`<=>` trả về cosine similarity đúng không?"**  
*3. [TRAP] "Does `<=>` return cosine similarity?"*

Câu trả lời là không.  
*The answer is no.*

Toán tử đó trả về **cosine distance**.  
*That operator returns cosine distance.*

Công thức là `1 − similarity`.  
*The formula is `1 − similarity`.*

Giá trị nhỏ nghĩa là gần.  
*A small value means close.*

Vì vậy bạn dùng `ORDER BY ASC`.  
*So you use `ORDER BY ASC`.*

4. **"Bạn đã có index HNSW."**  
*4. "You already have an HNSW index."*

**"Vì sao query vẫn `Seq Scan`?"**  
*"Why does the query still show `Seq Scan`?"*

Nguyên nhân thứ nhất là bảng nhỏ.  
*The first cause is a small table.*

Với bảng nhỏ, seq scan rẻ hơn.  
*On a small table, a seq scan is cheaper.*

Nguyên nhân thứ hai là ops class không khớp toán tử.  
*The second cause is an ops class that does not match the operator.*

Nguyên nhân thứ ba là cost estimation.  
*The third cause is cost estimation.*

Bạn xác nhận bằng `EXPLAIN ANALYZE`.  
*You confirm with `EXPLAIN ANALYZE`.*

5. **[BẪY] "`type vector does not exist` sau CREATE EXTENSION?"**  
*5. [TRAP] "Why `type vector does not exist` after CREATE EXTENSION?"*

Extension nằm ở schema khác `search_path`.  
*The extension sits in a schema outside `search_path`.*

Bạn sửa `search_path`.  
*You fix the `search_path`.*

Hoặc bạn tạo extension ở schema `public`.  
*Alternatively you create the extension in the `public` schema.*

6. **[OPS] "Thêm pgvector vào Postgres production không downtime?"**  
*6. [OPS] "How do you add pgvector to production Postgres with no downtime?"*

Bạn cài bằng gói hoặc dùng managed.  
*You install with a package or use a managed service.*

Bạn không build tay.  
*You do not build by hand.*

Bạn chạy `CREATE EXTENSION` qua migration.  
*You run `CREATE EXTENSION` through a migration.*

Bạn chạy `ADD COLUMN` với cột nullable.  
*You run `ADD COLUMN` with a nullable column.*

Bạn backfill ở nền.  
*You backfill in the background.*

Bạn chạy `CREATE INDEX CONCURRENTLY`.  
*You run `CREATE INDEX CONCURRENTLY`.*

Bạn verify kết quả.  
*You verify the result.*

Bạn đồng bộ version trên replica.  
*You sync the version across replicas.*

7. **"Kết nối pgvector từ app thế nào?"**  
*7. "How do you connect to pgvector from an app?"*

Bạn cài client `pgvector` cho Node.  
*You install the `pgvector` client for Node.*

Bạn cài `pgvector[psycopg]` cho Python.  
*You install `pgvector[psycopg]` for Python.*

Bạn **đăng ký kiểu** bằng `registerType` hoặc `register_vector`.  
*You register the type with `registerType` or `register_vector`.*

Sau đó bạn query như bình thường với `$1` hoặc `%s`.  
*Then you query as usual with `$1` or `%s`.*

8. **[STAFF] "Nên build-from-source trên production không?"**  
*8. [STAFF] "Should you build from source in production?"*

Câu trả lời là không.  
*The answer is no.*

Kết quả phụ thuộc vào CPU.  
*The result depends on the CPU.*

Nguyên nhân là cờ `-march=native`.  
*The cause is the `-march=native` flag.*

Kết quả cũng khó tái lập.  
*The result is also hard to reproduce.*

Kết quả cũng khó rollback.  
*The result is also hard to roll back.*

Bạn hãy pin version qua Docker tag hoặc gói OS.  
*Please pin the version through a Docker tag or an OS package.*

### 5.6. One-liner đắt giá
*5.6. High-value one-liners*

Cài binary và bật extension là hai bước khác nhau.  
*Installing the binary and enabling the extension are two different steps.*

Bạn chạy `make install` rồi chạy `CREATE EXTENSION`.  
*You run `make install`, then you run `CREATE EXTENSION`.*

`<=>` là cosine distance chứ không phải similarity.  
*`<=>` is cosine distance, not similarity.*

Nhỏ hơn nghĩa là gần hơn.  
*Smaller is closer.*

Vì vậy bạn ORDER BY theo chiều tăng dần.  
*So you ORDER BY ascending.*

Có index không có nghĩa là index được dùng.  
*Having an index does not mean it is used.*

Bạn hãy luôn xác nhận bằng EXPLAIN ANALYZE.  
*Always confirm with EXPLAIN ANALYZE.*

Bạn đừng build pgvector từ source trên production.  
*Do not build pgvector from source in production.*

Bạn hãy pin một Docker tag hoặc một gói OS như mọi dependency khác.  
*Pin a Docker tag or an OS package like any other dependency.*

Bạn hãy đăng ký kiểu vector trong client.  
*Register the vector type in your client.*

Bạn bỏ qua bước đó.  
*You skip that step.*

Khi đó app nhận về một chuỗi thay vì một mảng.  
*Your app then gets back a string instead of an array.*

Trên một database đang chạy, bạn bật extension qua migration.  
*On a live database, you enable the extension via migration.*

Bạn thêm cột nullable rồi backfill.  
*You add a nullable column, then backfill.*

Bạn đánh index bằng CONCURRENTLY.  
*You build the index with CONCURRENTLY.*

Cách làm này cho zero downtime ngay từ thiết kế.  
*This approach gives zero downtime by construction.*

---

### 📌 Ghi chú cuối
*📌 Closing notes*

**Đính chính để nhớ đúng**  
*Corrections worth memorizing*

Yêu cầu là PostgreSQL **13 trở lên**.  
*The requirement is PostgreSQL 13 or newer.*

Con số đó không phải 12.  
*That number is not 12.*

`<=>` là cosine **distance**.  
*`<=>` is cosine distance.*

Build-from-source là cách khó nhất.  
*Build-from-source is the hardest path.*

Bạn hãy ưu tiên **Docker, apt hoặc managed**.  
*Please prefer Docker, apt, or a managed service.*

Bạn hãy nhớ **đăng ký kiểu** ở client.  
*Please remember to register the type on the client.*

**Kiểm chứng khi ôn**  
*What to verify while reviewing*

Bạn hãy xem README chính thức tại `github.com/pgvector/pgvector`.  
*Please read the official README at `github.com/pgvector/pgvector`.*

Ở đó có phiên bản Postgres tối thiểu.  
*It lists the minimum Postgres version.*

Ở đó có đủ các cách cài.  
*It lists all the install paths.*

Ở đó có API client mới nhất.  
*It lists the latest client API.*

Các hàm `registerType` và `toSql` có thể đổi theo bản.  
*The `registerType` and `toSql` functions can change between releases.*

**Thực hành**  
*Practice*

Bạn hãy chạy `docker run pgvector/pgvector:pg17`.  
*Please run `docker run pgvector/pgvector:pg17`.*

Bạn hãy làm trọn một vòng.  
*Please complete a full round trip.*

Bạn chạy `CREATE EXTENSION`.  
*You run `CREATE EXTENSION`.*

Bạn tạo bảng có cột `vector(384)`.  
*You create a table with a `vector(384)` column.*

Bạn insert vài embedding thật từ giáo trình embeddings.  
*You insert a few real embeddings from the embeddings course.*

Bạn chạy query `ORDER BY <=>`.  
*You run a query with `ORDER BY <=>`.*

Bạn chạy `EXPLAIN ANALYZE` để thấy `Index Scan`.  
*You run `EXPLAIN ANALYZE` to see an `Index Scan`.*

Sau đó bạn nối vào từ một script Node.js hoặc Python.  
*Then you connect from a Node.js or Python script.*

**Nối mạch series — đủ 6 mảnh**  
*Linking the series — all six pieces*

Mảnh thứ nhất là embed để tạo vector.  
*The first piece is embedding to create vectors.*

Mảnh thứ hai là index để làm nhanh.  
*The second piece is indexing to make it fast.*

Mảnh thứ ba là **cài và dùng pgvector**.  
*The third piece is installing and using pgvector.*

Đó chính là bài thực hành này.  
*That is this hands-on lesson.*

Mảnh thứ tư là store và query ở mức khái niệm trong bài #3.  
*The fourth piece is storing and querying at the conceptual level in lesson #3.*

Mảnh thứ năm là keyword và FTS.  
*The fifth piece is keyword search and FTS.*

Mảnh thứ sáu là chọn nền tảng.  
*The sixth piece is choosing a platform.*

Bạn giờ đã *hiểu* semantic search và RAG trên PostgreSQL.  
*You now understand semantic search and RAG on PostgreSQL.*

Bạn cũng đã *làm được* chúng.  
*You can also build them.*

**Học tiếp**  
*What to learn next*

Bạn hãy kết nối pgvector với framework như LangChain hoặc LlamaIndex.  
*Please connect pgvector to a framework such as LangChain or LlamaIndex.*

Bạn hãy xây pipeline embedding bất đồng bộ.  
*Please build an asynchronous embedding pipeline.*

Bạn hãy triển khai một RAG app end-to-end.  
*Please deploy an end-to-end RAG app.*
