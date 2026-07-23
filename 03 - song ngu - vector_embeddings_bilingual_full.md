# Vector Embeddings: Biến chữ thành số và đo độ giống nhau
*Vector Embeddings: Turning Text into Numbers and Measuring Similarity*

## Giáo trình SIÊU DỄ HIỂU: Basic đến Staff Engineer
*An ULTRA-EASY Textbook: Basic to Staff Engineer*

> **Nguồn gốc — tài liệu ban đầu**  
> *Origin — source material*
>
> Video có tên *"Vector embeddings in relational databases"*.  
> *The video is titled "Vector embeddings in relational databases".*
>
> Video thuộc Course 3 của IBM Vector Database Fundamentals.  
> *The video belongs to Course 3 of IBM Vector Database Fundamentals.*
>
> **Cách đọc giáo trình này — hướng dẫn đọc**  
> *How to read this textbook — reading guidance*
>
> Tôi sẽ trình bày rất chậm.  
> *I will explain everything very slowly.*
>
> Một từ chuyên ngành có thể xuất hiện lần đầu.  
> *A technical term may appear for the first time.*
>
> Một từ tiếng Anh cũng có thể xuất hiện lần đầu.  
> *An English term may also appear for the first time.*
>
> Khi đó, tôi sẽ dừng lại.  
> *At that point, I will pause.*
>
> Tôi sẽ giải thích từ đó ngay tại chỗ.  
> *I will explain that term immediately.*
>
> Sau đó, tôi mới tiếp tục.  
> *After that, I will continue.*
>
> Một số chỗ có thể quá dài dòng với bạn.  
> *Some parts may feel too detailed for you.*
>
> Bạn có thể nhảy qua những chỗ đó.  
> *You may skip those parts.*
>
> Người mới nên đọc tuần tự.  
> *Beginners should read in order.*
>
> Người mới không nên nhảy cóc.  
> *Beginners should not skip ahead.*
>
> **Quy ước ký hiệu — ý nghĩa các biểu tượng**  
> *Symbol conventions — meanings of the icons*
>
> - 🧩 **[Ngoài bài gốc]** là phần bổ sung của tôi.  
>   *🧩 **[Beyond the original lesson]** marks my added material.*
>
> - Video gốc không nói phần này.  
>   *The original video does not cover this material.*
>
> - Một staff engineer bắt buộc phải biết phần này.  
>   *A staff engineer must know this material.*
>
> - ⚠️ **[Đính chính bài gốc]** đánh dấu nội dung chưa chính xác.  
>   *⚠️ **[Correction to the original lesson]** marks inaccurate content.*
>
> - Biểu tượng này cũng đánh dấu nội dung đã cũ.  
>   *This icon also marks outdated content.*
>
> - 💡 đánh dấu một mẹo quan trọng.  
>   *💡 marks an important tip.*
>
> - 💡 cũng đánh dấu một ý cần ghi nhớ.  
>   *💡 also marks an idea worth remembering.*

---

## ⚠️ Bốn điểm cần đính chính và cập nhật đến tháng 7/2026
*⚠️ Four Corrections and Updates as of July 2026*

Trước tiên, tôi sẽ liệt kê các chỗ đã cũ.  
*First, I will list the outdated parts.*

Tôi cũng sẽ liệt kê các chỗ chưa đúng.  
*I will also list the inaccurate parts.*

Mục tiêu là giúp bạn tránh học nhầm.  
*The goal is to help you avoid learning incorrect information.*

Bạn có thể đọc lướt phần này.  
*You may skim this section.*

Các phần sau sẽ giải thích kỹ từng điểm.  
*Later sections will explain each point carefully.*

**Điểm thứ nhất liên quan đến tên thư viện JavaScript.**  
*The first point concerns the JavaScript library name.*

Video dùng gói `@xenova/transformers`.  
*The video uses the `@xenova/transformers` package.*

Gói này đã đổi tên thành `@huggingface/transformers`.  
*This package is now named `@huggingface/transformers`.*

Gói mới đã lên phiên bản 4.  
*The new package has reached version 4.*

Phiên bản 4 ra mắt vào tháng 2/2026.  
*Version 4 launched in February 2026.*

Phiên bản này hỗ trợ card đồ hoạ trong trình duyệt.  
*This version supports browser-based graphics cards.*

Tên cũ vẫn có thể cài đặt.  
*The old name can still be installed.*

Tên cũ đã ngừng cập nhật.  
*The old name no longer receives updates.*

Tên mới có thể tạo một điểm cộng khi phỏng vấn.  
*The new name can create a small interview advantage.*

Điểm cộng này nhỏ.  
*This advantage is small.*

Điểm cộng này có thật.  
*This advantage is real.*

**Điểm thứ hai liên quan đến số chiều của vector.**  
*The second point concerns vector dimensions.*

Video nói vector toán học chỉ có tối đa 3 chiều.  
*The video says mathematical vectors have at most three dimensions.*

Câu đó sai.  
*That statement is wrong.*

Toán học cho phép không gian vector có số chiều tuỳ ý.  
*Mathematics allows vector spaces with arbitrary dimensions.*

Không gian 10 chiều là hợp lệ.  
*A 10-dimensional space is valid.*

Không gian 384 chiều là hợp lệ.  
*A 384-dimensional space is valid.*

Không gian 3072 chiều cũng hợp lệ.  
*A 3072-dimensional space is also valid.*

Thị giác con người bị giới hạn ở 3 chiều.  
*Human vision is limited to three dimensions.*

Toán học không có giới hạn đó.  
*Mathematics has no such limit.*

Mục 1.4 sẽ giải thích kỹ điểm này.  
*Section 1.4 will explain this point carefully.*

**Điểm thứ ba liên quan đến cosine similarity — độ tương đồng cosin.**  
*The third point concerns cosine similarity.*

Video nói cosine similarity chạy từ 0 đến 1.  
*The video says cosine similarity ranges from 0 to 1.*

Câu đó chưa chính xác.  
*That statement is not fully accurate.*

Miền giá trị toán học là từ −1 đến 1.  
*The mathematical range is from −1 to 1.*

Khoảng 0 đến 1 chỉ xuất hiện trong một trường hợp riêng.  
*The 0-to-1 range appears only in a special case.*

Đây là một câu hỏi bẫy phổ biến khi phỏng vấn.  
*This is a common interview trap question.*

Mục 1.6 sẽ giải thích chi tiết.  
*Section 1.6 will explain it in detail.*

**Điểm thứ tư liên quan đến danh sách model — mô hình.**  
*The fourth point concerns the model list.*

Danh sách model trong video đã cũ.  
*The model list in the video is outdated.*

Video nhắc Word2Vec.  
*The video mentions Word2Vec.*

Video nhắc GloVe.  
*The video mentions GloVe.*

Video nhắc Paragram.  
*The video mentions Paragram.*

Video nhắc Universal Sentence Encoder.  
*The video mentions Universal Sentence Encoder.*

Các model này có giá trị lịch sử.  
*These models have historical value.*

Chúng giúp bạn hiểu nguyên lý.  
*They help you understand the principles.*

Hầu như không ai dùng chúng cho hệ thống mới năm 2026.  
*Almost nobody uses them for new systems in 2026.*

Mục 2.3 có bảng cập nhật.  
*Section 2.3 contains an updated table.*

---

## Phần 0: 🗺️ Bản đồ bài học
*Part 0: 🗺️ Lesson Map*

### Bài này dạy gì trong một câu
*What This Lesson Teaches in One Sentence*

Bài này dạy cách biến một câu chữ thành một dãy số.  
*This lesson teaches how to turn a sentence into a number sequence.*

Hai câu giống nghĩa sẽ tạo ra hai dãy số gần nhau.  
*Two semantically similar sentences produce nearby number sequences.*

Bài này cũng dạy cách đo mức độ gần nhau.  
*This lesson also teaches how to measure that closeness.*

Một số từ trong câu trên có thể còn mơ hồ.  
*Some terms in the sentence above may still feel unclear.*

Bạn có thể chưa hiểu khái niệm gần nhau.  
*You may not yet understand the idea of closeness.*

Bạn cũng có thể chưa hiểu quan hệ giữa số với nghĩa.  
*You may also not understand the relationship between numbers and meaning.*

Điều đó hoàn toàn bình thường.  
*That is completely normal.*

Điều đó đúng với dự đoán của bài học.  
*That matches the lesson's expectation.*

Phần 1 sẽ lấp các khoảng trống này.  
*Part 1 will fill these gaps.*

Bạn không cần kiến thức nền trước khi bắt đầu.  
*You need no prior knowledge before starting.*

### Nó giải quyết nỗi đau gì
*What Pain Does It Solve?*

Máy tính không hiểu chữ.  
*Computers do not understand text.*

Với máy tính, chữ chỉ là một dãy ký tự.  
*To a computer, text is only a character sequence.*

Máy tính có thể so sánh hai dãy ký tự.  
*Computers can compare two character sequences.*

Máy tính biết hai dãy có giống hệt nhau hay không.  
*Computers know whether two sequences are identical.*

Máy tính không biết "áo thun" là "áo phông".  
*Computers do not know that "áo thun" means "áo phông".*

Nhiều tính năng hiện đại yêu cầu máy hiểu nghĩa.  
*Many modern features require machines to understand meaning.*

Một ví dụ là ô tìm kiếm thông minh.  
*One example is a smart search box.*

Ô này có thể nằm trong website bán hàng.  
*This box may appear on an e-commerce website.*

Một ví dụ khác là tính năng sản phẩm tương tự.  
*Another example is a similar-products feature.*

Chatbot tài liệu công ty cũng cần hiểu nghĩa.  
*Company-document chatbots also need semantic understanding.*

Ta phải chuyển ý nghĩa thành con số.  
*We must convert meaning into numbers.*

Máy tính có thể tính toán con số.  
*Computers can calculate numbers.*

Máy tính cũng có thể so sánh con số.  
*Computers can also compare numbers.*

**Embedding — phép nhúng vector chính là cây cầu đó.**  
*Embedding is exactly that bridge.*

Embedding biến ý nghĩa thành số.  
*Embedding converts meaning into numbers.*

Nội dung giống nghĩa sẽ tạo ra các dãy số gần nhau.  
*Semantically similar content produces nearby number sequences.*

Các dãy số này nằm trong một không gian.  
*These number sequences exist in a space.*

Ta có thể đo mức độ gần nhau.  
*We can measure their closeness.*

Một công thức toán rất đơn giản làm việc đó.  
*A very simple mathematical formula does that.*

Bạn sẽ tính công thức bằng tay ở Mục 1.7.  
*You will calculate the formula by hand in Section 1.7.*

### Học xong bạn sẽ làm được
*What You Will Be Able to Do*

- Bạn có thể giải thích embedding bằng lời của mình.  
  *You can explain embedding in your own words.*

- Bạn có thể giải thích khoảng cách giữa hai vector.  
  *You can explain the distance between two vectors.*

- Bạn có thể giải thích quan hệ giữa khoảng cách với độ giống nghĩa.  
  *You can explain the relationship between distance and semantic similarity.*

- Bạn có thể phân biệt static embedding — embedding tĩnh.  
  *You can distinguish static embedding.*

- Bạn có thể phân biệt contextual embedding — embedding theo ngữ cảnh.  
  *You can distinguish contextual embedding.*

- Kiến thức này có thể gây ấn tượng trong phỏng vấn.  
  *This knowledge can impress an interviewer.*

- Bạn có thể viết code tạo embedding bằng JavaScript.  
  *You can write JavaScript code that creates embeddings.*

- Code đó có thể chạy ngay trong trình duyệt.  
  *That code can run directly in a browser.*

- Bạn có thể viết code JavaScript với TensorFlow.  
  *You can write JavaScript code with TensorFlow.*

- Bạn có thể viết code Python theo chuẩn ngành.  
  *You can write industry-standard Python code.*

- Bạn có thể tính cosine similarity bằng tay.  
  *You can calculate cosine similarity by hand.*

- Bạn có thể tự viết hàm đó từ số 0.  
  *You can implement that function from scratch.*

- Interviewer thường yêu cầu bài tập này.  
  *Interviewers often request this exercise.*

- Bạn có thể chọn model embedding phù hợp cho dự án thật.  
  *You can choose a suitable embedding model for a real project.*

- Bạn có thể giải thích tác động của việc đổi model.  
  *You can explain the impact of changing models.*

- Đổi model là một quyết định kiến trúc nặng ký.  
  *Changing models is a major architectural decision.*

- Bạn có thể trả lời câu hỏi system design.  
  *You can answer a system design question.*

- Câu hỏi đó liên quan đến pipeline — chuỗi xử lý embedding.  
  *That question concerns an embedding pipeline.*

- Quy mô có thể đạt hàng chục triệu tài liệu.  
  *The scale can reach tens of millions of documents.*

### Mạch kiến thức từ đầu đến cuối
*The Knowledge Path from Start to Finish*

- 🟢 **Basic** bắt đầu từ nỗi đau tìm kiếm theo từ khoá.  
  *🟢 **Basic** starts with keyword-search pain.*

- Phần này dùng analogy — phép ví von bản đồ ý nghĩa.  
  *This part uses a meaning-map analogy.*

- Phần này định nghĩa embedding.  
  *This part defines embedding.*

- Phần này giải thích ai tạo embedding.  
  *This part explains who creates embeddings.*

- Phần này đo độ giống bằng cosine.  
  *This part measures similarity with cosine.*

- Phần này có một ví dụ tính tay.  
  *This part includes a hand calculation.*

- Phần này có code "hello world".  
  *This part includes "hello world" code.*

- 🟡 **Intermediate** giải thích quá trình tạo embedding.  
  *🟡 **Intermediate** explains the embedding process.*

- Phần này phân biệt static với contextual.  
  *This part distinguishes static and contextual embeddings.*

- Phần này trình bày bức tranh model năm 2026.  
  *This part presents the 2026 model landscape.*

- Phần này có code thật bằng ba ngôn ngữ.  
  *This part contains real code in three languages.*

- Phần này giải thích pooling — phép gộp.  
  *This part explains pooling.*

- Phần này giải thích normalize — chuẩn hoá.  
  *This part explains normalization.*

- Phần này chỉ ra ba lỗi kinh điển.  
  *This part identifies three classic mistakes.*

- 🔴 **Advanced** trình bày toán học phía sau cosine.  
  *🔴 **Advanced** presents the mathematics behind cosine.*

- Phần này có một chứng minh ngắn.  
  *This part contains a short proof.*

- Phần này giải thích lý do chuẩn hoá vector.  
  *This part explains vector normalization.*

- Phần này trình bày số chiều.  
  *This part discusses dimensions.*

- Phần này giới thiệu kỹ thuật Matryoshka.  
  *This part introduces the Matryoshka technique.*

- Phần này tự viết thuật toán từ số 0.  
  *This part implements the algorithm from scratch.*

- Phần này xử lý các trường hợp biên.  
  *This part handles edge cases.*

- 🟣 **Staff** trình bày embedding ở quy mô lớn.  
  *🟣 **Staff** presents embeddings at large scale.*

- Quy mô có thể đạt hàng chục triệu bản ghi.  
  *The scale can reach tens of millions of records.*

- Phần này tính chi phí thật bằng tiền.  
  *This part calculates real monetary costs.*

- Phần này trình bày quy trình chọn model.  
  *This part presents a model-selection process.*

- Phần này đưa ra kế hoạch đổi model an toàn.  
  *This part provides a safe model-migration plan.*

- Kế hoạch đó tránh làm sập hệ thống.  
  *That plan avoids system failure.*

- Phần này trình bày hoạt động giám sát.  
  *This part presents monitoring practices.*

- Phần này trình bày các kiểu hỏng.  
  *This part presents failure modes.*

- Phần này hướng dẫn nói chuyện với lãnh đạo không rành kỹ thuật.  
  *This part guides communication with nontechnical leaders.*

- Phần này có một câu hỏi system design đầy đủ.  
  *This part contains a complete system design question.*

- 🎯 **Cheatsheet** có bảng từ khoá.  
  *🎯 **Cheatsheet** contains a keyword table.*

- Phần này tóm các ý cốt lõi.  
  *This part summarizes the core ideas.*

- Phần này có code cần thuộc lòng.  
  *This part contains essential code.*

- Phần này có câu hỏi phỏng vấn.  
  *This part contains interview questions.*

- Phần này có gợi ý trả lời.  
  *This part contains answer hints.*

### Bài này nằm ở đâu trong bức tranh lớn
*Where This Lesson Fits in the Big Picture*

Bạn có thể đã học bài **pgvector** trước đó.  
*You may have studied the **pgvector** lesson earlier.*

Bài đó có một luận điểm quan trọng.  
*That lesson contains an important point.*

Pgvector không tự tạo embedding.  
*Pgvector does not create embeddings by itself.*

Một model phải tạo embedding.  
*A model must create the embeddings.*

Bài này giới thiệu model đó.  
*This lesson introduces that model.*

Toàn bộ chuỗi có dạng sau.  
*The complete flow looks like this.*

```
   văn bản
   text
      ↓  ← BÀI NÀY: model biến chữ thành vector
      ↓  ← THIS LESSON: the model turns text into vectors
   vector số
   numeric vector
      ↓  ← Bài pgvector: lưu vector trong database
      ↓  ← pgvector lesson: store vectors in the database
      ↓  ← Bài pgvector: tìm vector gần nhất
      ↓  ← pgvector lesson: find the nearest vector
   kết quả tìm kiếm theo nghĩa
   semantic search result
      ↓
   Semantic search / RAG / recommendation
```

Sơ đồ trên có bốn thuật ngữ chưa được giải thích.  
*The diagram contains four unexplained terms.*

Tôi sẽ giải thích chúng ngay tại đây.  
*I will explain them here.*

Bạn không cần đoán bất kỳ thuật ngữ nào.  
*You do not need to guess any term.*

> **pgvector — phần mở rộng vector cho PostgreSQL**  
> *pgvector — a vector extension for PostgreSQL*
>
> Pgvector là một phần mở rộng của PostgreSQL.  
> *Pgvector is a PostgreSQL extension.*
>
> Nó giúp database lưu vector.  
> *It helps the database store vectors.*
>
> Nó cũng giúp database tìm vector gần nhất rất nhanh.  
> *It also helps the database find nearest vectors quickly.*
>
> Pgvector không tự tạo vector.  
> *Pgvector does not create vectors itself.*
>
> Điểm này rất quan trọng.  
> *This point is very important.*
>
> Mục 1.5 sẽ giải thích kỹ hơn.  
> *Section 1.5 will explain it further.*
>
> **Semantic search — tìm kiếm ngữ nghĩa**  
> *Semantic search — meaning-based search*
>
> Semantic search tìm kiếm theo ý nghĩa.  
> *Semantic search searches by meaning.*
>
> Nó không chỉ tìm theo mặt chữ.  
> *It does not search only by surface text.*
>
> Người dùng có thể gõ "áo phông".  
> *A user may type "áo phông".*
>
> Hệ thống vẫn có thể trả về "áo thun".  
> *The system can still return "áo thun".*
>
> Máy hiểu hai từ này cùng nghĩa.  
> *The machine understands their shared meaning.*
>
> Tìm kiếm theo từ khoá không làm được việc đó.  
> *Keyword search cannot do that.*
>
> **RAG — sinh văn bản có tra cứu bổ trợ**  
> *RAG — Retrieval-Augmented Generation*
>
> RAG là một cách xây dựng chatbot.  
> *RAG is a way to build chatbots.*
>
> Chatbot sẽ trả lời dựa trên tài liệu riêng.  
> *The chatbot answers from private documents.*
>
> RAG có hai bước.  
> *RAG has two steps.*
>
> Bước một là retrieval — truy xuất.  
> *Step one is retrieval.*
>
> Người dùng đặt một câu hỏi.  
> *A user asks a question.*
>
> Hệ thống tìm trong kho tài liệu công ty.  
> *The system searches the company document store.*
>
> Hệ thống chọn vài đoạn liên quan nhất.  
> *The system selects the most relevant passages.*
>
> Kỹ thuật embedding của bài này hỗ trợ quá trình đó.  
> *This lesson's embedding technique supports that process.*
>
> Bước hai là generation — sinh văn bản.  
> *Step two is generation.*
>
> Hệ thống đưa các đoạn tìm được vào model ngôn ngữ lớn.  
> *The system sends the retrieved passages to a large language model.*
>
> Hệ thống cũng đưa câu hỏi vào model.  
> *The system also sends the question to the model.*
>
> Model viết ra câu trả lời.  
> *The model writes the answer.*
>
> Chatbot dựa trên tài liệu thật của bạn.  
> *The chatbot relies on your real documents.*
>
> Cách này giảm nguy cơ chatbot bịa.  
> *This approach reduces chatbot fabrication.*
>
> **Recommendation — gợi ý**  
> *Recommendation — suggestion feature*
>
> Recommendation tạo ra mục "sản phẩm tương tự".  
> *Recommendation creates a "similar products" section.*
>
> Nó cũng tạo ra mục "có thể bạn cũng thích".  
> *It also creates a "you may also like" section.*
>
> Cách dùng embedding rất trực tiếp.  
> *The embedding approach is very direct.*
>
> Hệ thống lấy sản phẩm người dùng đang xem.  
> *The system takes the product being viewed.*
>
> Hệ thống tìm các sản phẩm có vector gần nhất.  
> *The system finds products with the nearest vectors.*

---

## Phần 1: 🟢 BASIC
*Part 1: 🟢 BASIC*

> Phần này dành cho người chưa biết chủ đề.  
> *This part is for people new to the topic.*
>
> Đây là phần dài nhất của giáo trình.  
> *This is the longest part of the textbook.*
>
> Đây cũng là phần chậm nhất.  
> *This is also the slowest part.*
>
> Người có nền tảng có thể đọc lướt.  
> *Readers with prior knowledge may skim it.*
>
> Bạn không nên bỏ Mục 1.7.  
> *You should not skip Section 1.7.*
>
> Mục đó có ví dụ tính tay.  
> *That section contains a hand calculation.*
>
> Ví dụ đó giúp mọi ý khớp lại.  
> *That example makes every idea fit together.*

### 1.1. Nỗi đau: máy tính không đọc chữ như con người
*1.1. The Pain: Computers Do Not Read Like Humans*

Hãy tưởng tượng một website bán quần áo.  
*Imagine an online clothing store.*

Bạn đang xây dựng ô tìm kiếm cho website đó.  
*You are building its search box.*

Kho hàng có một sản phẩm.  
*The inventory contains a product.*

Tên sản phẩm là **"áo thun cotton nam"**.  
*The product name is **"áo thun cotton nam"**.*

Một khách hàng nhập **"áo phông nam"**.  
*A customer enters **"áo phông nam"**.*

Bạn biết hai cụm chỉ cùng một thứ.  
*You know the two phrases describe the same item.*

Tôi cũng biết điều đó.  
*I know that too.*

"Áo thun" là một cách gọi.  
*"Áo thun" is one regional expression.*

"Áo phông" là một cách gọi khác.  
*"Áo phông" is another regional expression.*

Hai cách gọi phổ biến ở hai miền.  
*The two expressions are common in different regions.*

Máy tính không hiểu điều đó.  
*The computer does not understand that.*

Với máy tính, hai tên là hai chuỗi khác nhau.  
*To the computer, the two names are different strings.*

> **String — chuỗi ký tự**  
> *String — character sequence*
>
> Trong lập trình, string là một dãy ký tự.  
> *In programming, a string is a character sequence.*
>
> Dãy này có thể chứa chữ cái.  
> *This sequence may contain letters.*
>
> Dãy này có thể chứa số.  
> *This sequence may contain numbers.*
>
> Dãy này có thể chứa dấu cách.  
> *This sequence may contain spaces.*
>
> Các ký tự được xếp cạnh nhau.  
> *The characters are arranged next to one another.*
>
> Máy tính lưu chúng như một dãy.  
> *The computer stores them as a sequence.*
>
> Máy tính không lưu chữ "áo" như khái niệm cái áo.  
> *The computer does not store "áo" as the concept of a shirt.*
>
> Nó lưu một dãy mã số.  
> *It stores a sequence of numeric codes.*
>
> Mỗi mã số đại diện cho một ký tự.  
> *Each numeric code represents a character.*
>
> Máy tính hoàn toàn không biết cái áo là gì.  
> *The computer has no idea what a shirt is.*

Bạn có thể viết một câu lệnh tìm kiếm truyền thống.  
*You can write a traditional search query.*

Câu lệnh có thể chạy trong database.  
*The query can run in a database.*

Ví dụ là câu lệnh sau.  
*The following query is an example.*

```sql
SELECT * FROM products WHERE name LIKE '%áo phông%';
```

> **SQL — ngôn ngữ truy vấn có cấu trúc**  
> *SQL — Structured Query Language*
>
> SQL là viết tắt của *Structured Query Language*.  
> *SQL stands for Structured Query Language.*
>
> SQL là ngôn ngữ tiêu chuẩn cho database quan hệ.  
> *SQL is the standard language for relational databases.*
>
> Bạn dùng SQL để hỏi dữ liệu.  
> *You use SQL to query data.*
>
> Câu lệnh trên lấy tất cả dữ liệu từ bảng `products`.  
> *The query retrieves all data from the `products` table.*
>
> Điều kiện nằm trên cột `name`.  
> *The condition applies to the `name` column.*
>
> Cột đó phải chứa cụm "áo phông".  
> *That column must contain the phrase "áo phông".*
>
> Cụm này có thể nằm ở bất kỳ vị trí nào.  
> *The phrase may appear at any position.*
>
> **Relational database — database quan hệ**  
> *Relational database — relational data store*
>
> RDBMS là viết tắt của Relational Database Management System.  
> *RDBMS stands for Relational Database Management System.*
>
> Loại database này lưu dữ liệu dưới dạng bảng.  
> *This database type stores data in tables.*
>
> Mỗi bảng có nhiều hàng.  
> *Each table has many rows.*
>
> Mỗi hàng là một bản ghi.  
> *Each row is a record.*
>
> Một bản ghi có thể đại diện cho một sản phẩm.  
> *A record may represent a product.*
>
> Mỗi bảng cũng có nhiều cột.  
> *Each table also has many columns.*
>
> Column có nghĩa là cột.  
> *Column means a table column.*
>
> Field cũng có nghĩa là cột dữ liệu.  
> *Field also means a data column.*
>
> Mỗi cột lưu một thuộc tính.  
> *Each column stores an attribute.*
>
> Ví dụ là tên, giá và màu.  
> *Examples include name, price, and color.*
>
> PostgreSQL là một RDBMS.  
> *PostgreSQL is an RDBMS.*
>
> MySQL là một RDBMS.  
> *MySQL is an RDBMS.*
>
> SQL Server cũng là một RDBMS.  
> *SQL Server is also an RDBMS.*
>
> **`LIKE '%...%'` — phép so khớp chuỗi trong SQL**  
> *`LIKE '%...%'` — SQL string matching*
>
> `LIKE` so khớp chuỗi ký tự.  
> *`LIKE` matches character strings.*
>
> Dấu `%` đại diện cho bất kỳ nội dung nào.  
> *The `%` symbol represents any content.*
>
> `'%áo phông%'` có nghĩa là chứa cụm "áo phông".  
> *`'%áo phông%'` means containing the phrase "áo phông".*
>
> Cụm đó có thể nằm ở bất kỳ vị trí nào.  
> *The phrase may appear anywhere.*

Câu lệnh này không trả về kết quả.  
*This query returns no result.*

Tên sản phẩm không chứa cụm "áo phông".  
*The product name does not contain "áo phông".*

Khách hàng thấy một trang trắng.  
*The customer sees an empty page.*

Khách hàng rời khỏi website.  
*The customer leaves the website.*

Bạn vừa mất một đơn hàng.  
*You have just lost an order.*

Máy tính không biết hai từ đồng nghĩa.  
*The computer does not know the two terms are synonyms.*

Bạn có thể nghĩ đến full-text search — tìm kiếm toàn văn.  
*You may think about full-text search.*

> **Full-text search — tìm kiếm toàn văn**  
> *Full-text search — full-text retrieval*
>
> Full-text search nâng cao hơn `LIKE`.  
> *Full-text search is more advanced than `LIKE`.*
>
> Nó cắt câu thành các từ.  
> *It splits a sentence into words.*
>
> Nó đưa từng từ về dạng gốc.  
> *It reduces each word to a root form.*
>
> Ví dụ tiếng Anh là "running" thành "run".  
> *An English example is "running" becoming "run".*
>
> Dạng gốc này gọi là lexeme — từ vị.  
> *This root form is called a lexeme.*
>
> Hệ thống có thể tìm "run" từ truy vấn "running".  
> *The system can find "run" from the query "running".*

Full-text search giỏi hơn `LIKE`.  
*Full-text search is better than `LIKE`.*

Full-text search vẫn thất bại trong trường hợp này.  
*Full-text search still fails in this case.*

"Thun" với "phông" không cùng một từ gốc.  
*"Thun" and "phông" do not share a root form.*

Chúng là hai từ khác nhau về mặt chữ.  
*They are different words on the surface.*

Chúng chỉ giống nhau về ý nghĩa.  
*They are similar only in meaning.*

Full-text search làm việc ở tầng từ.  
*Full-text search operates at the word level.*

Nó không làm việc ở tầng nghĩa.  
*It does not operate at the meaning level.*

Đây là giới hạn của kỹ thuật tìm kiếm dựa trên chữ.  
*This is the limit of text-based search techniques.*

| Kỹ thuật / Technique | Tầng hoạt động / Operating level | "áo thun" ≈ "áo phông"? |
|---|---|---|
| `LIKE` | ký tự / characters | ❌ Không / No |
| Full-text search | từ, lexeme / words, lexemes | ❌ Không / No |
| **Embedding** | **ý nghĩa / meaning** | ✅ Có / Yes |

Ta cần một cách biểu diễn ý nghĩa.  
*We need a way to represent meaning.*

Máy tính phải tính toán được cách biểu diễn đó.  
*The computer must be able to calculate with that representation.*

Cách biểu diễn đó chính là embedding.  
*That representation is embedding.*

### 1.2. Analogy: tấm bản đồ ý nghĩa
*1.2. Analogy: The Map of Meaning*

Trước tiên, bạn cần một hình ảnh trong đầu.  
*First, you need a mental image.*

Định nghĩa suông thường rất dễ quên.  
*A bare definition is often easy to forget.*

Hãy tưởng tượng một tấm bản đồ khổng lồ.  
*Imagine a gigantic map.*

Đây không phải bản đồ địa lý.  
*This is not a geographic map.*

Đây là bản đồ của ý nghĩa.  
*This is a map of meaning.*

Mỗi từ nằm tại một vị trí.  
*Each word occupies a position.*

Mỗi câu cũng nằm tại một vị trí.  
*Each sentence also occupies a position.*

Mỗi đoạn văn cũng nằm tại một vị trí.  
*Each paragraph also occupies a position.*

Một quy tắc duy nhất quyết định các vị trí.  
*A single rule determines the positions.*

> Nội dung giống nghĩa được đặt gần nhau.  
> *Semantically similar content is placed nearby.*
>
> Nội dung khác nghĩa được đặt xa nhau.  
> *Semantically different content is placed far apart.*

"Áo thun" nằm gần "áo phông".  
*"Áo thun" lies near "áo phông".*

"T-shirt" cũng nằm gần hai cụm đó.  
*"T-shirt" also lies near those phrases.*

"Áo cotton ngắn tay" cũng nằm trong cụm đó.  
*"Áo cotton ngắn tay" also belongs to that cluster.*

Các cụm này tạo thành một nhóm nhỏ.  
*These phrases form a small group.*

Nhóm đó nằm tại một góc của bản đồ.  
*That group lies in one area of the map.*

"Xe máy" nằm ở một góc khác.  
*"Xe máy" lies in another area.*

"Ô tô" nằm gần "xe máy".  
*"Ô tô" lies near "xe máy".*

"Xe hơi" cũng nằm gần hai từ đó.  
*"Xe hơi" also lies near those terms.*

Cụm phương tiện nằm xa cụm áo.  
*The vehicle cluster lies far from the clothing cluster.*

"Mèo" với "chó" có khoảng cách vừa phải.  
*"Mèo" and "chó" have a moderate distance.*

Chúng không sát nhau.  
*They are not extremely close.*

Mèo không phải chó.  
*A cat is not a dog.*

Chúng vẫn gần nhau hơn mèo với ô tô.  
*They are still closer than a cat and a car.*

Mèo với chó đều là thú cưng bốn chân.  
*Cats and dogs are both four-legged pets.*

Mỗi vị trí trên bản đồ có một toạ độ.  
*Every map position has coordinates.*

Bản đồ địa lý dùng hai con số.  
*Geographic maps use two numbers.*

Hai con số đó là kinh độ với vĩ độ.  
*Those two numbers are longitude and latitude.*

Bản đồ ý nghĩa cũng dùng một bộ số.  
*The meaning map also uses a number set.*

Bộ số này có nhiều giá trị hơn.  
*This number set has more values.*

Nó có thể chứa vài trăm số.  
*It may contain several hundred numbers.*

**Bộ số toạ độ đó chính là embedding.**  
*That coordinate number set is the embedding.*

Đó là toàn bộ ý tưởng.  
*That is the whole idea.*

Phép ví von khớp với kỹ thuật ở bốn điểm.  
*The analogy matches the technique in four ways.*

| Trên bản đồ / On a map | Trong kỹ thuật / In the technique |
|---|---|
| Toạ độ của một điểm / Coordinates of a point | Embedding của một câu / Embedding of a sentence |
| Số toạ độ để định vị / Number of coordinates | Số chiều của embedding, thường 384–3072 / Embedding dimensions, usually 384–3072 |
| Hai điểm gần nhau / Two nearby points | Hai câu giống nghĩa / Two semantically similar sentences |
| Đo khoảng cách / Measuring distance | Tính cosine similarity / Calculating cosine similarity |
| Người vẽ bản đồ / The mapmaker | Model AI đã được huấn luyện / A trained AI model |

Dòng cuối của bảng rất quan trọng.  
*The final row of the table is very important.*

Nhiều người thường bỏ qua điểm này.  
*Many people often overlook this point.*

Một người phải vẽ bản đồ.  
*Someone must draw the map.*

Bản đồ không tự sinh ra.  
*The map does not create itself.*

Một model AI đóng vai trò người vẽ.  
*An AI model acts as the mapmaker.*

Mục 1.5 sẽ nói về model đó.  
*Section 1.5 will discuss that model.*

### 1.3. Định nghĩa: vector là gì
*1.3. Definition: What Is a Vector?*

Từ "vector" nghe rất toán học.  
*The word "vector" sounds very mathematical.*

Nhiều người cảm thấy sợ từ này.  
*Many people feel intimidated by this word.*

Trong ngữ cảnh này, vector rất đơn giản.  
*In this context, a vector is very simple.*

> **Vector — danh sách số có thứ tự**  
> *Vector — an ordered list of numbers*
>
> Vector là một danh sách các con số.  
> *A vector is a list of numbers.*
>
> Các con số được xếp theo thứ tự.  
> *The numbers are arranged in order.*
>
> Định nghĩa chỉ có vậy.  
> *That is the entire definition.*
>
> Không có điều gì bí ẩn hơn.  
> *There is nothing more mysterious.*

`[2, 5, -1]` là một vector 3 chiều.  
*`[2, 5, -1]` is a three-dimensional vector.*

`[0.3, -0.8, 0.1, 0.9, 0.02]` là vector 5 chiều.  
*`[0.3, -0.8, 0.1, 0.9, 0.02]` is a five-dimensional vector.*

Trong lập trình, vector thường được gọi là array — mảng.  
*In programming, a vector is often called an array.*

Nó cũng có thể được gọi là list — danh sách.  
*It may also be called a list.*

Khái niệm vẫn giống nhau.  
*The concept remains the same.*

Tên gọi phụ thuộc vào ngôn ngữ.  
*The name depends on the language.*

Bạn cần nhớ hai điều ở tầng basic.  
*You need to remember two things at the basic level.*

Điều thứ nhất liên quan đến thứ tự.  
*The first point concerns order.*

Thứ tự có ý nghĩa.  
*Order carries meaning.*

Vector `[1, 2]` khác vector `[2, 1]`.  
*Vector `[1, 2]` differs from vector `[2, 1]`.*

Vector không phải một túi số lộn xộn.  
*A vector is not an unordered bag of numbers.*

Vị trí thứ nhất luôn mang một loại thông tin.  
*The first position always carries one information type.*

Vị trí thứ hai mang một loại thông tin khác.  
*The second position carries another information type.*

Điều thứ hai liên quan đến cách hình dung vector.  
*The second point concerns vector intuition.*

Bạn có thể xem vector như một mũi tên.  
*You can view a vector as an arrow.*

Mũi tên đó chỉ một hướng.  
*That arrow points in a direction.*

Bạn cũng có thể xem vector như một điểm.  
*You can also view a vector as a point.*

Điểm đó nằm trên một bản đồ.  
*That point lies on a map.*

Vector `[2, 1]` có thể biểu diễn một vị trí.  
*Vector `[2, 1]` can represent a position.*

Vị trí đó nằm bên phải 2 đơn vị.  
*That position lies two units to the right.*

Vị trí đó nằm phía trên 1 đơn vị.  
*That position lies one unit upward.*

Vector này cũng có thể biểu diễn một mũi tên.  
*This vector can also represent an arrow.*

Mũi tên xuất phát từ gốc.  
*The arrow starts at the origin.*

Mũi tên chỉ về hướng phải chếch lên.  
*The arrow points diagonally upward to the right.*

Hai cách hiểu này tương đương.  
*These two interpretations are equivalent.*

Cả hai cách đều hữu ích.  
*Both interpretations are useful.*

Cách hiểu điểm hỗ trợ hình dung bản đồ.  
*The point interpretation supports the map analogy.*

Cách hiểu mũi tên sẽ hữu ích ở Mục 1.6.  
*The arrow interpretation will help in Section 1.6.*

Mục đó sẽ nói về góc.  
*That section will discuss angles.*

### 1.4. Định nghĩa: embedding với số chiều
*1.4. Definition: Embedding and Dimensions*

Bây giờ, ta sẽ ghép hai mảnh kiến thức.  
*Now, we will combine two knowledge pieces.*

> **Embedding — phép nhúng vector**  
> *Embedding — vector embedding*
>
> Embedding là một vector.  
> *An embedding is a vector.*
>
> Vector đó là một dãy số.  
> *That vector is a number sequence.*
>
> Dãy số biểu diễn ý nghĩa của một đối tượng.  
> *The sequence represents the meaning of an object.*
>
> Từ "embed" có nghĩa là "nhúng vào".  
> *The word "embed" means "to place inside".*
>
> Ta nhúng câu chữ vào một không gian số.  
> *We embed text into a numeric space.*
>
> Đối tượng không nhất thiết phải là chữ.  
> *The object does not have to be text.*
>
> Đối tượng có thể là một câu.  
> *The object may be a sentence.*
>
> Đối tượng có thể là một đoạn văn.  
> *The object may be a paragraph.*
>
> Đối tượng có thể là một tấm ảnh.  
> *The object may be an image.*
>
> Đối tượng có thể là một đoạn video.  
> *The object may be a video clip.*
>
> Đối tượng có thể là một file âm thanh.  
> *The object may be an audio file.*
>
> Đối tượng cũng có thể là dữ liệu cảm biến.  
> *The object may also be sensor data.*
>
> Một model phù hợp phải xử lý được loại dữ liệu đó.  
> *A suitable model must support that data type.*

Ví dụ sau được rút gọn cho dễ nhìn.  
*The following example is shortened for readability.*

```
"áo thun cotton"  →  [ 0.21, -0.87,  0.44,  0.03, ... ]   (384 số)
"áo thun cotton"  →  [ 0.21, -0.87,  0.44,  0.03, ... ]   (384 numbers)
"áo phông"        →  [ 0.19, -0.83,  0.47,  0.05, ... ]   (384 số)
"áo phông"        →  [ 0.19, -0.83,  0.47,  0.05, ... ]   (384 numbers, very similar)
"xe máy Honda"    →  [-0.62,  0.11, -0.35,  0.88, ... ]   (384 số)
"xe máy Honda"    →  [-0.62,  0.11, -0.35,  0.88, ... ]   (384 numbers, very different)
```

Hãy quan sát kỹ ba ví dụ.  
*Look closely at the three examples.*

Hai dòng đầu có nhiều con số gần nhau.  
*The first two lines contain many nearby values.*

Vị trí đầu có 0.21 với 0.19.  
*The first position has 0.21 and 0.19.*

Vị trí tiếp theo có −0.87 với −0.83.  
*The next position has −0.87 and −0.83.*

Dòng thứ ba có các con số khác hẳn.  
*The third line contains very different values.*

Đây là hình dạng số của sự gần nghĩa.  
*This is the numeric form of semantic closeness.*

> **Dimension — số chiều**  
> *Dimension — vector length*
>
> Dimension là độ dài của dãy số.  
> *Dimension is the length of the number sequence.*
>
> Nó cho biết dãy có bao nhiêu số.  
> *It tells how many numbers the sequence contains.*
>
> Mỗi model tạo embedding có số chiều cố định.  
> *Each embedding model produces a fixed dimension.*

Bạn nên nhớ một số giá trị thực tế.  
*You should remember several real values.*

| Model | Số chiều / Dimensions |
|---|---|
| `all-MiniLM-L6-v2`, model nhẹ để học / lightweight learning model | 384 |
| TensorFlow Universal Sentence Encoder | 512 |
| OpenAI `text-embedding-3-small` | 1536 |
| OpenAI `text-embedding-3-large` | 3072 |
| Google Gemini Embedding | 3072 |

⚠️ **[Đính chính bài gốc]**  
*⚠️ **[Correction to the original lesson]***

Video gốc nói sai tại điểm này.  
*The original video is wrong at this point.*

Sai sót này gây hiểu lầm nghiêm trọng.  
*This error causes serious confusion.*

Video nói vector toán học bị giới hạn ở 3 chiều.  
*The video says mathematical vectors are limited to three dimensions.*

Video nói vector AI có thể có nhiều chiều hơn.  
*The video says AI vectors can have more dimensions.*

Câu này ngược với sự thật.  
*This statement reverses the truth.*

Toán học chưa bao giờ giới hạn vector ở 3 chiều.  
*Mathematics has never limited vectors to three dimensions.*

Không gian vector *n* chiều là một khái niệm chuẩn.  
*An *n*-dimensional vector space is a standard concept.*

*n* có thể là bất kỳ số nguyên dương nào.  
*The value *n* may be any positive integer.*

Khái niệm này có nền tảng chặt chẽ từ thế kỷ 19.  
*This concept has had rigorous foundations since the nineteenth century.*

Nó xuất hiện rất lâu trước máy tính.  
*It appeared long before computers.*

Bạn có thể làm toán trong không gian 384 chiều.  
*You can do mathematics in 384-dimensional space.*

Bạn cũng có thể dùng không gian 1 triệu chiều.  
*You can also use a one-million-dimensional space.*

Các công thức vẫn hoạt động bình thường.  
*The formulas still work normally.*

Trí tưởng tượng thị giác của con người mới bị giới hạn.  
*Human visual imagination is the limited part.*

Con người sống trong không gian 3 chiều.  
*Humans live in three-dimensional space.*

Não người khó hình dung từ 4 chiều trở lên.  
*The human brain struggles to visualize four or more dimensions.*

Không hình dung được không có nghĩa là không tính được.  
*Inability to visualize does not imply inability to calculate.*

💡 **Cách tránh bị rối**  
*💡 **A Way to Avoid Confusion***

Bạn không nên cố tưởng tượng không gian 384 chiều.  
*You should not try to visualize 384-dimensional space.*

Bạn sẽ không làm được việc đó.  
*You will not be able to do that.*

Bạn cũng không cần làm việc đó.  
*You do not need to do that.*

Hãy dùng không gian 2 chiều để lấy trực giác.  
*Use two-dimensional space for intuition.*

Bạn cũng có thể dùng không gian 3 chiều.  
*You may also use three-dimensional space.*

Công thức toán mở rộng lên 384 chiều một cách máy móc.  
*The mathematical formula extends mechanically to 384 dimensions.*

Khoảng cách 2 chiều dùng công thức `sqrt(x² + y²)`.  
*Two-dimensional distance uses `sqrt(x² + y²)`.*

Đây là định lý Pythagoras.  
*This is the Pythagorean theorem.*

Bạn đã học nó ở cấp 2.  
*You learned it in secondary school.*

Không gian 384 chiều dùng công thức tương tự.  
*The 384-dimensional space uses a similar formula.*

Công thức là `sqrt(x₁² + x₂² + ... + x₃₈₄²)`.  
*The formula is `sqrt(x₁² + x₂² + ... + x₃₈₄²)`.*

Công thức chỉ có thêm nhiều số hạng.  
*The formula only contains more terms.*

Không có phép màu nào ở đây.  
*There is no magic here.*
### 1.5. Ai tạo ra embedding: model
*1.5. Who creates embeddings: the model*

Ở Mục 1.2, tôi đã nhắc đến người vẽ bản đồ ý nghĩa.  
*Section 1.2 mentioned the creator of the meaning map.*

Bây giờ, chúng ta sẽ nói về người đó.  
*Now we will discuss that creator.*

> **Model — mô hình**  
> ***Model — a computational model***
>
> Trong machine learning, model là một chương trình.  
> *In machine learning, a model is a program.*
>
> Model đã học từ dữ liệu.  
> *The model learned from data.*
>
> Model học cách thực hiện một công việc.  
> *The model learns how to perform a task.*
>
> Lập trình viên không viết thủ công từng bước cho model.  
> *A programmer does not manually specify every step for the model.*

> **Machine learning — học máy**  
> ***Machine learning — machine learning***
>
> Machine learning là một cách xây dựng phần mềm.  
> *Machine learning is a way to build software.*
>
> Người phát triển không viết mọi quy tắc cụ thể.  
> *The developer does not write every specific rule.*
>
> Người phát triển cung cấp rất nhiều ví dụ cho máy.  
> *The developer provides many examples to the machine.*
>
> Máy tự rút ra quy luật từ các ví dụ.  
> *The machine derives patterns from the examples.*
>
> Ví dụ, bạn có thể xây dựng bộ lọc spam.  
> *For example, you can build a spam filter.*
>
> Bạn không cần viết luật về chữ "trúng thưởng".  
> *You do not need a rule about the phrase "trúng thưởng".*
>
> Bạn có thể cung cấp 1 triệu email đã dán nhãn.  
> *You can provide one million labeled emails.*
>
> Mỗi email có nhãn spam hoặc không spam.  
> *Each email has a spam or non-spam label.*
>
> Máy sẽ tự tìm ra các dấu hiệu.  
> *The machine will discover the signals itself.*

> **Pretrained — đã huấn luyện sẵn**  
> ***Pretrained — already trained***
>
> Một pretrained model đã được người khác huấn luyện.  
> *A pretrained model has already been trained by others.*
>
> Google có thể huấn luyện model.  
> *Google may train the model.*
>
> OpenAI cũng có thể huấn luyện model.  
> *OpenAI may also train the model.*
>
> Meta cũng có thể huấn luyện model.  
> *Meta may also train the model.*
>
> Cộng đồng mã nguồn mở cũng có thể huấn luyện model.  
> *The open-source community may also train the model.*
>
> Quá trình huấn luyện dùng hàng tỉ mẫu dữ liệu.  
> *The training process uses billions of data samples.*
>
> Quá trình này có thể tốn hàng triệu đô tiền điện.  
> *This process may cost millions of dollars in electricity.*
>
> Quá trình này cũng cần nhiều card đồ hoạ.  
> *This process also requires many graphics cards.*
>
> Bạn chỉ cần tải model về.  
> *You only need to download the model.*
>
> Sau đó, bạn có thể sử dụng model.  
> *Then you can use the model.*
>
> Thông thường, bạn không tự huấn luyện model embedding.  
> *Normally, you do not train an embedding model yourself.*
>
> Một lab nghiên cứu có thể tự huấn luyện model embedding.  
> *A research lab may train its own embedding model.*

Model embedding học một bản đồ ý nghĩa.  
*An embedding model learns a meaning map.*

Quá trình này nghe rất khó tin.  
*This process sounds unbelievable.*

Nguyên lý của nó lại rất đơn giản.  
*Its principle is very simple.*

Model đọc một lượng văn bản khổng lồ.  
*The model reads a massive amount of text.*

Lượng văn bản này có thể gần bằng toàn bộ internet.  
*This amount of text may approach the size of the entire internet.*

Model học theo một nguyên tắc.  
*The model learns through one principle.*

> **Một số từ thường xuất hiện trong ngữ cảnh giống nhau.**  
> ***Some words often appear in similar contexts.***
>
> **Các từ đó thường có nghĩa giống nhau.**  
> ***Those words often have similar meanings.***

Hãy xem xét hàng tỉ câu tiếng Việt trên mạng.  
*Consider the billions of Vietnamese sentences online.*

Từ "áo thun" thường xuất hiện cạnh một số từ.  
*The phrase "áo thun" often appears near certain words.*

Từ "áo phông" cũng thường xuất hiện cạnh các từ đó.  
*The phrase "áo phông" also often appears near those words.*

Các ví dụ là "mặc", "size" và "cotton".  
*Examples include "mặc", "size", and "cotton".*

Các ví dụ khác là "ngắn tay" và "giặt máy".  
*Other examples include "ngắn tay" and "giặt máy".*

Từ "xe máy" xuất hiện trong ngữ cảnh khác.  
*The phrase "xe máy" appears in a different context.*

Nó thường xuất hiện cạnh từ "xăng".  
*It often appears near the word "xăng".*

Nó cũng thường xuất hiện cạnh cụm "biển số".  
*It also often appears near the phrase "biển số".*

Cụm "đội mũ bảo hiểm" cũng thường xuất hiện gần nó.  
*The phrase "đội mũ bảo hiểm" also often appears near it.*

Không ai cần dạy model câu "áo thun là áo phông".  
*Nobody needs to teach the model the sentence "áo thun là áo phông".*

Model tự rút ra mối quan hệ đó.  
*The model derives that relationship itself.*

Model quan sát cách sử dụng của hai từ.  
*The model observes the usage of both phrases.*

Hai từ thường xuất hiện trong cùng hoàn cảnh.  
*The two phrases often appear in the same situations.*

Ý tưởng này đã xuất hiện từ lâu trong ngôn ngữ học.  
*This idea has existed in linguistics for a long time.*

Nó thường được tóm gọn bằng một câu.  
*It is often summarized in one sentence.*

> **Ngữ cảnh xung quanh giúp bạn hiểu một từ.**  
> ***The surrounding context helps you understand a word.***

💡 **Đây là câu quan trọng nhất của cả bài.**  
*💡 **This is the most important statement in the entire lesson.***

Nó sẽ xuất hiện lại ở Phần 4.  
*It will appear again in Part 4.*

> **MODEL tạo ra embedding.**  
> ***The MODEL creates embeddings.***
>
> **DATABASE chỉ lưu embedding.**  
> ***The DATABASE only stores embeddings.***
>
> **DATABASE cũng tìm embedding.**  
> ***The DATABASE also searches embeddings.***

Đây là hai công việc hoàn toàn tách biệt.  
*These are two completely separate jobs.*

Hai thành phần khác nhau đảm nhiệm hai công việc.  
*Two different components handle the two jobs.*

> **pgvector — phần mở rộng vector cho PostgreSQL**  
> ***pgvector — a vector extension for PostgreSQL***
>
> pgvector là một extension của PostgreSQL.  
> *pgvector is an extension for PostgreSQL.*
>
> Extension này cho phép database lưu cột kiểu vector.  
> *This extension lets the database store vector columns.*
>
> Extension này cũng tìm vector gần nhất rất nhanh.  
> *This extension also finds nearest vectors very quickly.*
>
> pgvector không sinh ra embedding.  
> *pgvector does not create embeddings.*
>
> Bạn phải tự gọi model.  
> *You must call the model yourself.*
>
> Model trả về một vector.  
> *The model returns a vector.*
>
> Sau đó, bạn đưa vector vào pgvector.  
> *Then you send the vector to pgvector.*
>
> pgvector sẽ lưu vector đó.  
> *pgvector will store that vector.*

Một analogy có thể giúp bạn ghi nhớ.  
*An analogy can help you remember.*

**Model là đầu bếp.**  
*The model is the chef.*

**Database là cái tủ lạnh.**  
*The database is the refrigerator.*

Đầu bếp nấu ra món ăn.  
*The chef prepares the food.*

Tủ lạnh cất món ăn.  
*The refrigerator stores the food.*

Tủ lạnh giúp bạn lấy món ăn nhanh.  
*The refrigerator helps you retrieve the food quickly.*

Cái tủ lạnh không thể tự nấu ăn.  
*The refrigerator cannot cook by itself.*

Ý tưởng ngược lại thật vô lý.  
*The opposite idea is absurd.*

Tuy nhiên, người mới thường có hiểu nhầm này.  
*However, beginners often have this misconception.*

Đây cũng là một câu hỏi phỏng vấn phổ biến.  
*This is also a common interview question.*

### 1.6. Đo "gần nhau" như thế nào: cosine similarity
*1.6. How to measure closeness: cosine similarity*

Chúng ta đã có embedding.  
*We now have embeddings.*

Bây giờ, chúng ta cần đo khoảng cách giữa hai embedding.  
*Now we need to measure the closeness of two embeddings.*

Một phương pháp phổ biến là cosine similarity.  
*One common method is cosine similarity.*

> **Cosine similarity — độ tương đồng cosin**  
> ***Cosine similarity — cosine similarity***
>
> Cosine similarity là một con số.  
> *Cosine similarity is a number.*
>
> Con số này đo mức độ giống nhau giữa hai vector.  
> *This number measures the similarity between two vectors.*
>
> Phép đo sử dụng góc giữa hai vector.  
> *The measurement uses the angle between the two vectors.*

Cosine là hàm cosin trong lượng giác.  
*Cosine is the cosine function from trigonometry.*

Bạn đã học hàm này ở trường phổ thông.  
*You learned this function in secondary school.*

Bạn không cần nhớ kiến thức lượng giác.  
*You do not need to remember trigonometry.*

Bạn chỉ cần nhớ một tính chất.  
*You only need to remember one property.*

Cosin biểu thị mức độ chụm của hai hướng.  
*Cosine represents the alignment of two directions.*

Ở Mục 1.3, vector được mô tả như một mũi tên.  
*Section 1.3 described a vector as an arrow.*

Bây giờ, chúng ta sẽ dùng cách hiểu đó.  
*Now we will use that interpretation.*

Hãy hình dung hai mũi tên có cùng điểm gốc.  
*Imagine two arrows with the same starting point.*

- Hai mũi tên chỉ cùng một hướng.  
  *The two arrows point in the same direction.*

- Góc giữa chúng gần bằng 0°.  
  *The angle between them is close to 0°.*

- Cosine similarity gần bằng **1**.  
  *Cosine similarity is close to **1**.*

- Hai vector rất giống nhau.  
  *The two vectors are very similar.*

- Hai mũi tên vuông góc với nhau.  
  *The two arrows are perpendicular.*

- Góc giữa chúng bằng 90°.  
  *The angle between them is 90°.*

- Cosine similarity bằng **0**.  
  *Cosine similarity equals **0**.*

- Hai vector không liên quan.  
  *The two vectors are unrelated.*

- Hai mũi tên chỉ ngược hướng nhau.  
  *The two arrows point in opposite directions.*

- Góc giữa chúng bằng 180°.  
  *The angle between them is 180°.*

- Cosine similarity bằng **−1**.  
  *Cosine similarity equals **−1**.*

- Hai vector biểu thị ý nghĩa đối lập.  
  *The two vectors represent opposite meanings.*

Đây là bảng bạn nên ghi nhớ.  
*You should remember this table.*

| Góc giữa hai vector / Angle between vectors | Cosine similarity / Cosine similarity | Ý nghĩa với văn bản / Meaning for text |
|---|---|---|
| 0° (cùng hướng) / 0° (same direction) | 1.0 / 1.0 | Cùng nghĩa / Same meaning |
| ~30° / ~30° | ~0.87 / ~0.87 | Rất liên quan / Highly related |
| ~60° / ~60° | ~0.5 / ~0.5 | Hơi liên quan / Somewhat related |
| 90° (vuông góc) / 90° (perpendicular) | 0.0 / 0.0 | Không liên quan / Unrelated |
| 180° (ngược hướng) / 180° (opposite directions) | −1.0 / −1.0 | Đối lập / Opposite |

Đây là ý then chốt.  
*This is the key idea.*

Nó cũng là một câu trả lời phỏng vấn hữu ích.  
*It is also a useful interview answer.*

> 💡 **Cosine chỉ quan tâm đến hướng.**  
> *💡 **Cosine only considers direction.***
>
> **Cosine không quan tâm đến độ dài của mũi tên.**  
> ***Cosine does not consider the length of the arrow.***

Đặc tính này rất phù hợp với văn bản.  
*This property suits text very well.*

Hãy xem hai câu sau.  
*Consider the following two sentences.*

- Câu A là *"Con mèo đang ngủ."*  
  *Sentence A is "Con mèo đang ngủ."*

- Câu B là *"Con mèo đang ngủ say sưa trên chiếc ghế sofa màu xám ở phòng khách."*  
  *Sentence B is "Con mèo đang ngủ say sưa trên chiếc ghế sofa màu xám ở phòng khách."*

Câu B dài hơn câu A rất nhiều.  
*Sentence B is much longer than sentence A.*

Một số cách biểu diễn tạo vector dài hơn cho câu dài.  
*Some representations create longer vectors for longer sentences.*

Tuy nhiên, hai câu có cùng chủ đề.  
*However, the two sentences have the same topic.*

Chủ đề của cả hai câu là giấc ngủ của con mèo.  
*The topic of both sentences is a cat's sleep.*

Cosine bỏ qua độ dài.  
*Cosine ignores length.*

Cosine chỉ so sánh hướng.  
*Cosine compares only direction.*

Vì thế, cosine nhận ra hai câu cùng chủ đề.  
*Therefore, cosine recognizes the shared topic.*

Một phép đo khác có thể xét độ dài.  
*Another metric may consider length.*

Phép đo đó có thể cho kết luận sai.  
*That metric may produce an incorrect conclusion.*

Nó có thể xem hai câu rất khác nhau.  
*It may view the two sentences as very different.*

Nguyên nhân chỉ là sự khác biệt về độ dài.  
*The only cause is the difference in length.*

Đó là lý do cosine thường được dùng cho văn bản.  
*That is why cosine is commonly used for text.*

⚠️ **[Đính chính bài gốc]**  
*⚠️ **[Correction to the original lesson]***

Video nói cosine similarity chạy từ 0 đến 1.  
*The video says cosine similarity ranges from 0 to 1.*

Câu đó chưa chính xác về mặt toán học.  
*That statement is mathematically inaccurate.*

Miền giá trị đầy đủ là từ −1 đến 1.  
*The full range is from −1 to 1.*

Cosin của mọi góc đều nằm trong khoảng đó.  
*The cosine of every angle lies within that range.*

Cosine chỉ nằm từ 0 đến 1 trong một trường hợp riêng.  
*Cosine lies between 0 and 1 only in a special case.*

Mọi thành phần của hai vector phải không âm.  
*Every component of both vectors must be nonnegative.*

Khi đó, hai vector không chứa số âm.  
*In that case, the two vectors contain no negative values.*

Hai mũi tên không thể chỉ ngược hướng.  
*The two arrows cannot point in opposite directions.*

Góc giữa chúng luôn nhỏ hơn hoặc bằng 90°.  
*The angle between them is always at most 90°.*

Embedding hiện đại có chứa số âm.  
*Modern embeddings contain negative values.*

Ví dụ ở Mục 1.4 có giá trị `-0.87`.  
*The example in Section 1.4 contains the value `-0.87`.*

Về lý thuyết, cosine có thể cho giá trị âm.  
*In theory, cosine can produce a negative value.*

Các câu thật hiếm khi tạo giá trị âm mạnh.  
*Real sentences rarely produce strongly negative values.*

Tuy nhiên, cosine không luôn nằm trong `[0,1]`.  
*However, cosine does not always lie within `[0,1]`.*

Khẳng định đó sai về mặt toán học.  
*That claim is mathematically false.*

Đây là một câu hỏi bẫy phổ biến trong phỏng vấn.  
*This is a common trick question in interviews.*

Bạn nên trả lời **`[−1, 1]`**.  
*You should answer **`[−1, 1]`**.*

### 1.7. Ví dụ chạy tay: tính cosine similarity bằng bút và giấy
*1.7. Manual example: calculating cosine similarity with pen and paper*

Đây là mục quan trọng nhất của Phần 1.  
*This is the most important section in Part 1.*

Bạn không nên đọc lướt mục này.  
*You should not skim this section.*

Hãy lấy giấy ra.  
*Take out a sheet of paper.*

Hãy tính theo từng bước.  
*Follow each calculation step.*

Bạn chỉ cần tự tính một lần.  
*You only need to calculate it once yourself.*

Sau đó, cosine similarity sẽ không còn đáng sợ.  
*Afterward, cosine similarity will no longer feel intimidating.*

Ví dụ này dùng vector 2 chiều.  
*This example uses two-dimensional vectors.*

Embedding thật có thể có 384 chiều.  
*Real embeddings may have 384 dimensions.*

Toán học trong hai trường hợp hoàn toàn giống nhau.  
*The mathematics is identical in both cases.*

Ví dụ 2 chiều chỉ có ít số hạng hơn.  
*The two-dimensional example simply has fewer terms.*

Các con số dưới đây được tạo để dễ tính.  
*The following numbers were chosen for easy calculation.*

Chúng không phải embedding thật.  
*They are not real embeddings.*

**Công thức**  
*The formula*

```
                        A · B
cosine_similarity(A, B) = ───────────
                        |A| × |B|

trong đó:
where:
    A · B  =  a₁×b₁ + a₂×b₂          ← gọi là DOT PRODUCT (tích vô hướng)
                                          called the DOT PRODUCT
    |A|    =  sqrt(a₁² + a₂²)         ← gọi là MAGNITUDE / NORM (độ dài của vector)
                                          called the MAGNITUDE / NORM
```

> **Dot product — tích vô hướng**  
> ***Dot product — dot product***
>
> Dot product có ký hiệu là dấu chấm `·`.  
> *The dot product uses the symbol `·`.*
>
> Bạn nhân các cặp số ở cùng vị trí.  
> *You multiply number pairs at matching positions.*
>
> Sau đó, bạn cộng mọi kết quả.  
> *Then you add all the products.*
>
> Kết quả cuối cùng là một số duy nhất.  
> *The final result is a single number.*
>
> Hãy dùng hai vector `[2,1]` và `[4,2]`.  
> *Use the vectors `[2,1]` and `[4,2]`.*
>
> Cặp đầu tiên cho kết quả `2×4=8`.  
> *The first pair gives `2×4=8`.*
>
> Cặp thứ hai cho kết quả `1×2=2`.  
> *The second pair gives `1×2=2`.*
>
> Tổng của hai kết quả bằng `10`.  
> *The sum of the two products equals `10`.*

> **Magnitude / norm — độ dài của vector**  
> ***Magnitude / norm — vector length***
>
> Magnitude có ký hiệu bằng hai vạch đứng `|A|`.  
> *Magnitude uses the notation `|A|`.*
>
> Magnitude là độ dài của mũi tên.  
> *Magnitude is the length of the arrow.*
>
> Công thức dựa trên định lý Pythagoras.  
> *The formula uses the Pythagorean theorem.*
>
> Bạn bình phương từng số.  
> *You square each number.*
>
> Sau đó, bạn cộng các kết quả.  
> *Then you add the results.*
>
> Cuối cùng, bạn lấy căn bậc hai.  
> *Finally, you take the square root.*
>
> Ký hiệu `sqrt` là viết tắt của *square root*.  
> *The notation `sqrt` means *square root*.*
>
> Cụm này có nghĩa là căn bậc hai.  
> *This phrase means the square root.*

**Dữ liệu ví dụ**  
*Example data*

Ví dụ có ba embedding từ 2 chiều.  
*The example contains three two-dimensional word embeddings.*

- **A = "mèo" = (2, 1)**  
  ***A = "mèo" = (2, 1)***

- **B = "miu" = (4, 2)**  
  ***B = "miu" = (4, 2)***

"Miu" là từ đồng nghĩa của "mèo".  
*"Miu" is a synonym for "mèo".*

- **C = "ô tô" = (−1, 3)**  
  ***C = "ô tô" = (−1, 3)***

"Ô tô" không liên quan đến "mèo".  
*"Ô tô" is unrelated to "mèo".*

---

**Bước 1: Tính similarity giữa A và B**  
*Step 1: Calculate the similarity between A and B*

A biểu thị từ "mèo".  
*A represents the word "mèo".*

B biểu thị từ "miu".  
*B represents the word "miu".*

Tính dot product.  
*Calculate the dot product.*

```
A · B = 2×4 + 1×2
      = 8   + 2
      = 10
```

Tính độ dài của A.  
*Calculate the length of A.*

```
|A| = sqrt(2² + 1²) = sqrt(4 + 1) = sqrt(5) ≈ 2.236
```

Tính độ dài của B.  
*Calculate the length of B.*

```
|B| = sqrt(4² + 2²) = sqrt(16 + 4) = sqrt(20) ≈ 4.472
```

Thay các kết quả vào công thức.  
*Substitute the results into the formula.*

```
cosine = 10 / (2.236 × 4.472)
       = 10 / 10.0
       = 1.00
```

**Kết quả là 1.00.** ✅  
*The result is 1.00.* ✅

Hai vector giống nhau hoàn toàn về hướng.  
*The two vectors have exactly the same direction.*

Tại sao kết quả bằng đúng 1?  
*Why does the result equal exactly 1?*

B = (4, 2) chính là A nhân đôi.  
*B = (4, 2) is exactly twice A.*

Phép nhân tạo ra `(2×2, 1×2)`.  
*The multiplication produces `(2×2, 1×2)`.*

Kết quả của phép nhân là `(4, 2)`.  
*The multiplication result is `(4, 2)`.*

Hai mũi tên chỉ đúng cùng một hướng.  
*The two arrows point in exactly the same direction.*

Chúng chỉ khác nhau về độ dài.  
*They differ only in length.*

B dài gấp đôi A.  
*B is twice as long as A.*

Ví dụ này cho thấy một tính chất của cosine.  
*This example demonstrates one property of cosine.*

Cosine không quan tâm đến độ dài.  
*Cosine does not consider length.*

Mũi tên B dài gấp đôi mũi tên A.  
*Arrow B is twice as long as arrow A.*

Kết quả vẫn bằng 1.00 tuyệt đối.  
*The result still equals exactly 1.00.*

Công thức chia cho `|A| × |B|`.  
*The formula divides by `|A| × |B|`.*

Phép chia loại bỏ ảnh hưởng của độ dài.  
*The division removes the effect of length.*

Kết quả chỉ còn phản ánh hướng.  
*The result reflects only direction.*

---

**Bước 2: Tính similarity giữa A và C**  
*Step 2: Calculate the similarity between A and C*

A biểu thị từ "mèo".  
*A represents the word "mèo".*

C biểu thị từ "ô tô".  
*C represents the phrase "ô tô".*

Tính dot product.  
*Calculate the dot product.*

```
A · C = 2×(−1) + 1×3
      = −2     + 3
      = 1
```

Tính độ dài của C.  
*Calculate the length of C.*

```
|C| = sqrt((−1)² + 3²) = sqrt(1 + 9) = sqrt(10) ≈ 3.162
```

Độ dài của A là 2.236.  
*The length of A is 2.236.*

Giá trị này đã được tính ở trên.  
*This value was calculated above.*

Thay các giá trị vào công thức.  
*Substitute the values into the formula.*

```
cosine = 1 / (2.236 × 3.162)
       = 1 / 7.07
       ≈ 0.14
```

**Kết quả là 0.14.** ✅  
*The result is 0.14.* ✅

Hai vector gần như không liên quan.  
*The two vectors are almost unrelated.*

Con số 0.14 rất gần 0.  
*The value 0.14 is very close to 0.*

Ở Mục 1.6, cosine gần 0 có một ý nghĩa.  
*Section 1.6 explained the meaning of cosine values near 0.*

Hai mũi tên gần vuông góc.  
*The two arrows are nearly perpendicular.*

Hai ý nghĩa gần như không liên quan.  
*The two meanings are almost unrelated.*

Kết quả này đúng với kỳ vọng.  
*This result matches our expectation.*

"Mèo" không liên quan đến "ô tô".  
*"Mèo" is unrelated to "ô tô".*

---

**Tổng kết ví dụ chạy tay**  
*Summary of the manual example*

| Cặp / Pair | Dot product / Dot product | Cosine / Cosine | Diễn giải / Interpretation |
|---|---|---|---|
| mèo – miu / mèo – miu | 10 / 10 | **1.00** / **1.00** | Đồng nghĩa ✅ / Synonymous ✅ |
| mèo – ô tô / mèo – ô tô | 1 / 1 | **0.14** / **0.14** | Không liên quan ✅ / Unrelated ✅ |

💡 **Bạn vừa thực hiện chính xác phép tính của pgvector.**  
*💡 **You just performed the exact calculation used by pgvector.***

pgvector chạy phép tính này trong semantic search.  
*pgvector runs this calculation during semantic search.*

Bạn thường kích hoạt nó bằng một câu truy vấn.  
*You usually trigger it with a query.*

Ví dụ này chỉ sử dụng 2 chiều.  
*This example uses only 2 dimensions.*

pgvector có thể sử dụng 384 chiều.  
*pgvector may use 384 dimensions.*

pgvector cũng có thể sử dụng 1536 chiều.  
*pgvector may also use 1536 dimensions.*

pgvector thực hiện phép tính hàng triệu lần mỗi giây.  
*pgvector performs the calculation millions of times per second.*

Bản chất toán học vẫn không thay đổi.  
*The mathematical principle remains unchanged.*

Bạn hiểu ví dụ này.  
*You understand this example.*

Khi đó, bạn đã hiểu lõi của vector search.  
*At that point, you understand the core of vector search.*

### 1.8. Code "hello world": tạo embedding thật và đo similarity
*1.8. "Hello world" code: create real embeddings and measure similarity*

Bây giờ, chúng ta sẽ chạy code thật.  
*Now, we will run real code.*

Ví dụ này sử dụng JavaScript.  
*This example uses JavaScript.*

Ví dụ chạy trực tiếp trên máy của bạn.  
*The example runs directly on your computer.*

Ví dụ cũng có thể chạy trong trình duyệt.  
*The example can also run in a browser.*

Ví dụ không cần API key.  
*The example does not require an API key.*

Ví dụ không tốn tiền.  
*The example costs nothing.*

> **API key — khóa truy cập**  
> *API key — access key*
>
> API key là một chuỗi ký tự bí mật.  
> *An API key is a secret character string.*
>
> Nó có vai trò giống mật khẩu.  
> *It works like a password.*
>
> Nó chứng minh danh tính của bạn với dịch vụ.  
> *It proves your identity to a service.*
>
> Dịch vụ cũng dùng nó để tính phí.  
> *The service also uses it for billing.*
>
> Ví dụ này chạy model ngay trên máy bạn.  
> *This example runs the model on your computer.*
>
> Vì thế, ví dụ này không cần API key.  
> *Therefore, this example needs no API key.*

**Chuẩn bị**  
*Preparation*

Bạn hãy cài thư viện bằng lệnh sau.  
*Install the library with the following command.*

Bạn chạy lệnh trong terminal.  
*Run the command in a terminal.*

```bash
npm install @huggingface/transformers
```

> **npm — Node Package Manager**  
> *npm — Node Package Manager*
>
> npm là công cụ quản lý package cho JavaScript.  
> *npm is a package manager for JavaScript.*
>
> Lệnh `npm install <tên>` tải package về dự án.  
> *The `npm install <name>` command downloads a package into a project.*
>
> **Library — thư viện**  
> *Library — reusable code collection*
>
> Library là code do người khác viết sẵn.  
> *A library is reusable code written by others.*
>
> Bạn dùng lại library trong dự án.  
> *You reuse a library in a project.*
>
> Bạn không phải viết lại mọi thứ.  
> *You do not need to rewrite everything.*
>
> **Package — gói phần mềm**  
> *Package — software package*
>
> Package là một đơn vị phân phối code.  
> *A package is a unit for distributing code.*
>
> Trong ngữ cảnh này, library và package gần nghĩa nhau.  
> *In this context, library and package have similar meanings.*
>
> **Hugging Face**  
> *Hugging Face*
>
> Hugging Face là một công ty AI.  
> *Hugging Face is an AI company.*
>
> Hugging Face cũng là một nền tảng cộng đồng.  
> *Hugging Face is also a community platform.*
>
> Nền tảng này giống GitHub dành cho AI.  
> *The platform resembles GitHub for AI.*
>
> Mọi người chia sẻ hàng trăm nghìn model tại đó.  
> *People share hundreds of thousands of models there.*
>
> Nhiều model đã được huấn luyện sẵn.  
> *Many models are pretrained.*
>
> Bạn có thể tải nhiều model miễn phí.  
> *You can download many models for free.*
>
> **Transformers.js**  
> *Transformers.js*
>
> Transformers.js là một library của Hugging Face.  
> *Transformers.js is a Hugging Face library.*
>
> Library này chạy model AI bằng JavaScript.  
> *This library runs AI models with JavaScript.*
>
> Model có thể chạy trên máy của bạn.  
> *The model can run on your computer.*
>
> Model cũng có thể chạy trong trình duyệt.  
> *The model can also run in a browser.*
>
> Cách chạy này không cần server.  
> *This execution method needs no server.*
>
> Tên package cũ là `@xenova/transformers`.  
> *The old package name is `@xenova/transformers`.*
>
> Video gốc sử dụng tên cũ.  
> *The original video uses the old name.*
>
> Tên package mới là `@huggingface/transformers`.  
> *The new package name is `@huggingface/transformers`.*
>
> Package hiện ở phiên bản 4.  
> *The package is currently at version 4.*

**Code**  
*Code*

```javascript
// Nhập hàm pipeline từ thư viện.
// Import the pipeline function from the library.
// Hàm này tạo một dây chuyền xử lý.
// This function creates a processing pipeline.
// Nhập hàm cos_sim từ thư viện.
// Import the cos_sim function from the library.
// Hàm này tính cosine similarity.
// This function calculates cosine similarity.
import { pipeline, cos_sim } from '@huggingface/transformers';

// ─── BƯỚC 1: Nạp model ───────────────────────────────────────────
// ─── STEP 1: Load the model ──────────────────────────────────────
// pipeline() tạo một dây chuyền xử lý hoàn chỉnh.
// pipeline() creates a complete processing pipeline.
// Nó cắt văn bản thành token.
// It splits text into tokens.
// Nó chạy model.
// It runs the model.
// Nó trả về các con số.
// It returns numbers.
const extractor = await pipeline(
  'feature-extraction',          // TÊN CÔNG VIỆC: trích xuất đặc trưng
                                 // TASK NAME: feature extraction
                                 // Đây là tên chính thức của việc tạo embedding.
                                 // This is the official name for embedding creation.
  'Xenova/all-MiniLM-L6-v2'      // TÊN MODEL: model nhẹ khoảng 23 MB
                                 // MODEL NAME: a lightweight model of about 23 MB
                                 // Model tạo vector 384 chiều.
                                 // The model creates 384-dimensional vectors.
                                 // Model chạy được trong browser.
                                 // The model can run in a browser.
);
// Lần chạy đầu tiên tải model từ mạng.
// The first run downloads the model from the internet.
// Quá trình này có thể mất vài giây.
// This process can take several seconds.
// Các lần sau dùng model trong cache.
// Later runs use the model from the cache.
// Khi đó, chương trình chạy ngay lập tức.
// The program then starts immediately.

// ─── BƯỚC 2: Biến câu thành vector ───────────────────────────────
// ─── STEP 2: Convert sentences into vectors ──────────────────────
// pooling:'mean' là một tham số bắt buộc.
// pooling:'mean' is a required parameter.
// normalize:true là một tham số bắt buộc.
// normalize:true is a required parameter.
// Bạn cần cả hai tham số.
// You need both parameters.
// Thiếu chúng, bạn không nhận được vector cả câu.
// Without them, you do not receive a sentence vector.
// Mục 2.5 giải thích chi tiết lý do.
// Section 2.5 explains the reason in detail.
// Đây là lỗi phổ biến nhất của người mới.
// This is the most common beginner mistake.
const a = await extractor('con mèo đang ngủ', {
  pooling: 'mean',      // gộp vector của từng từ thành một vector cho cả câu
                        // merge token vectors into one sentence vector
  normalize: true       // co vector về độ dài đúng bằng 1
                        // scale the vector to a length of exactly 1
});

const b = await extractor('chú miu đang thiu thiu', {
  pooling: 'mean',
  normalize: true
});

const c = await extractor('giá xăng hôm nay tăng', {  // câu không liên quan dùng để đối chứng
                                                     // an unrelated sentence used as a control
  pooling: 'mean',
  normalize: true
});

// ─── BƯỚC 3: Xem kết quả vừa nhận ────────────────────────────────
// ─── STEP 3: Inspect the returned result ─────────────────────────
console.log(a.dims);           // [1, 384] nghĩa là một câu có 384 chiều
                               // [1, 384] means one sentence has 384 dimensions
console.log(a.data.length);    // 384 nghĩa là có đúng 384 con số
                               // 384 means there are exactly 384 numbers
console.log(a.data.slice(0, 5)); // lấy 5 số đầu
                                 // get the first 5 numbers
                                 // Ví dụ: [0.031, -0.084, 0.012, ...]
                                 // Example: [0.031, -0.084, 0.012, ...]

// ─── BƯỚC 4: Đo độ giống nhau ────────────────────────────────────
// ─── STEP 4: Measure similarity ──────────────────────────────────
console.log(cos_sim(a.data, b.data));   // khoảng 0.65
                                        // about 0.65
                                        // Hai câu cùng nói về mèo ngủ.
                                        // Both sentences describe a sleeping cat.
console.log(cos_sim(a.data, c.data));   // khoảng 0.05
                                        // about 0.05
                                        // Hai câu gần như không liên quan.
                                        // The two sentences are almost unrelated.
```

**Đọc lại kết quả để thấy điều đặc biệt**  
*Review the result to see the key effect*

Câu thứ nhất là `"con mèo đang ngủ"`.  
*The first sentence is `"con mèo đang ngủ"`.*

Câu thứ hai là `"chú miu đang thiu thiu"`.  
*The second sentence is `"chú miu đang thiu thiu"`.*

Hai câu gần như không có từ chung.  
*The two sentences share almost no words.*

Chúng chỉ cùng chứa từ `"đang"`.  
*They only share the word `"đang"`.*

`LIKE` không thể khớp hai câu này.  
*`LIKE` cannot match these sentences.*

Full-text search cũng không thể khớp chúng.  
*Full-text search cannot match them either.*

Cosine similarity giữa chúng đạt khoảng 0.65.  
*Their cosine similarity reaches about 0.65.*

Giá trị này cao hơn rõ rệt so với 0.05.  
*This value is clearly higher than 0.05.*

Giá trị 0.05 thuộc câu về giá xăng.  
*The 0.05 value belongs to the sentence about fuel prices.*

Máy tính đã nhận ra hai câu cùng nghĩa.  
*The computer recognized their similar meaning.*

Máy tính không cần nhiều từ chung.  
*The computer needed no substantial word overlap.*

Đây là bức tường chúng ta vừa phá vỡ.  
*This is the barrier we just broke.*

**Kiểu dữ liệu của `a.data`**  
*The data type of `a.data`*

Kiểu dữ liệu của `a.data` là `Float32Array`.  
*The data type of `a.data` is `Float32Array`.*

Nó là một mảng số thực.  
*It is an array of floating-point numbers.*

Mỗi số sử dụng 32 bit.  
*Each number uses 32 bits.*

> **Float — số thực dấu chấm động**  
> *Float — floating-point number*
>
> Float là kiểu số có phần thập phân.  
> *A float is a number type with a decimal part.*
>
> Giá trị `0.031` là một ví dụ.  
> *The value `0.031` is one example.*
>
> Con số 32 biểu thị 32 bit cho mỗi số.  
> *The number 32 represents 32 bits per value.*
>
> 32 bit bằng 4 byte.  
> *Thirty-two bits equal 4 bytes.*
>
> Một embedding 384 chiều có 384 số.  
> *A 384-dimensional embedding contains 384 numbers.*
>
> Nó cần `384 × 4 = 1536 byte`.  
> *It requires `384 × 4 = 1536 bytes`.*
>
> Dung lượng này xấp xỉ 1.5 KB.  
> *This size is approximately 1.5 KB.*
>
> Con số này trông rất nhỏ.  
> *This number looks very small.*
>
> 100 triệu tài liệu cần khoảng 150 GB.  
> *One hundred million documents require about 150 GB.*
>
> Khi đó, dung lượng trở thành vấn đề kiến trúc.  
> *At that scale, storage becomes an architectural issue.*
>
> Phần 4 sẽ thảo luận vấn đề này.  
> *Part 4 will discuss this issue.*

### 1.9. 🧩 [Ngoài bài gốc] Hai điều video gốc chưa nói hết
*1.9. 🧩 [Beyond the original lesson] Two points the original video did not fully explain*

**Điều thứ nhất: con người không thể đọc embedding.**  
*First point: humans cannot read embeddings.*

Điều đó hoàn toàn bình thường.  
*That is completely normal.*

Video gốc đưa ra một nhận xét đúng.  
*The original video makes a correct observation.*

Embedding không tồn tại dưới dạng dễ suy luận.  
*An embedding does not exist in an easily interpretable form.*

Tôi sẽ giải thích rõ hơn về điều này.  
*I will explain this point more clearly.*

Bạn có thể nhìn vào chiều thứ 47.  
*You can inspect dimension 47.*

Giá trị tại đó có thể là 0.31.  
*Its value may be 0.31.*

Con số đó không có ý nghĩa riêng.  
*That number has no standalone meaning.*

Nó không biểu thị độ vui vẻ.  
*It does not represent happiness.*

Nó cũng không biểu thị mức liên quan đến động vật.  
*It also does not represent animal relevance.*

Model tự sắp xếp thông tin trong quá trình huấn luyện.  
*The model organizes information during training.*

Model sử dụng cách biểu diễn riêng.  
*The model uses its own representation.*

Không ai biết chính xác ý nghĩa của từng chiều.  
*No one knows the exact meaning of each dimension.*

Ngay cả người tạo model cũng không biết.  
*Even the model creators do not know.*

Toàn bộ 384 con số cùng nhau mới mang ý nghĩa.  
*All 384 numbers together carry meaning.*

Ý nghĩa xuất hiện qua phép so sánh embedding.  
*Meaning emerges through embedding comparison.*

💡 **Bạn không đọc embedding.**  
*💡 **You do not read embeddings.***

💡 **Bạn so sánh embedding.**  
*💡 **You compare embeddings.***

Bạn không nên diễn giải từng con số.  
*You should not interpret each number.*

**Điều thứ hai: vector search là một lĩnh vực riêng.**  
*Second point: vector search is a separate field.*

Video gốc đề cập một cách làm thủ công.  
*The original video mentions a manual approach.*

Cách đó sử dụng SQL thuần.  
*That approach uses plain SQL.*

Bạn lấy toàn bộ vector khỏi database.  
*You retrieve every vector from the database.*

Bạn tính cosine cho từng vector.  
*You calculate cosine for each vector.*

Sau đó, bạn sắp xếp toàn bộ kết quả.  
*You then sort all results.*

Cách làm này tốn rất nhiều bộ nhớ.  
*This approach consumes a large amount of memory.*

Cách làm này cũng rất chậm.  
*This approach is also very slow.*

Nhận xét của video hoàn toàn chính xác.  
*The video's observation is completely correct.*

Chúng ta cần đào sâu nhận xét này.  
*We need to examine this observation more deeply.*

Hãy xét một phép tính cụ thể.  
*Consider a concrete calculation.*

Bạn có 10 triệu sản phẩm.  
*You have 10 million products.*

Mỗi sản phẩm có một embedding 384 chiều.  
*Each product has a 384-dimensional embedding.*

Người dùng nhập một từ khóa tìm kiếm.  
*The user enters a search term.*

Cách thủ công cần thực hiện các bước sau.  
*The manual approach requires the following steps.*

- Hệ thống đọc 15 GB dữ liệu từ đĩa.  
  *The system reads 15 GB of data from disk.*

- Con số này đến từ `10 triệu × 1.5 KB`.  
  *This number comes from `10 million × 1.5 KB`.*

- Hệ thống thực hiện 10 triệu phép tính cosine.  
  *The system performs 10 million cosine calculations.*

- Mỗi phép tính cần khoảng 1150 phép toán.  
  *Each calculation needs about 1,150 operations.*

- Tổng số đạt khoảng 11.5 tỉ phép tính.  
  *The total reaches about 11.5 billion operations.*

- Hệ thống sắp xếp 10 triệu kết quả.  
  *The system sorts 10 million results.*

- Hệ thống lấy ra top 10.  
  *The system returns the top 10.*

Hệ thống phải hoàn thành mọi việc dưới 100 mili giây.  
*The system must finish everything within 100 milliseconds.*

Yêu cầu này áp dụng cho mỗi lượt tìm kiếm.  
*This requirement applies to every search.*

Yêu cầu này cũng áp dụng cho mỗi người dùng.  
*This requirement also applies to every user.*

Cách làm thủ công không thể đáp ứng yêu cầu.  
*The manual approach cannot meet the requirement.*

Đó là lý do ANN index tồn tại.  
*That is why ANN indexes exist.*

> **ANN — Approximate Nearest Neighbor**  
> *ANN — Approximate Nearest Neighbor*
>
> ANN có nghĩa là láng giềng gần nhất xấp xỉ.  
> *ANN means approximate nearest neighbor.*
>
> Từ khóa quan trọng là xấp xỉ.  
> *The important word is approximate.*
>
> Hệ thống không kiểm tra hết 10 triệu vector.  
> *The system does not inspect all 10 million vectors.*
>
> Hệ thống bỏ qua phần lớn vector.  
> *The system skips most vectors.*
>
> Kết quả có thể sai một phần nhỏ.  
> *The result may contain a small error.*
>
> Đổi lại, tốc độ tăng hàng nghìn lần.  
> *In return, speed improves by thousands of times.*
>
> **Index — chỉ mục**  
> *Index — search index*
>
> Index là một cấu trúc dữ liệu phụ.  
> *An index is an auxiliary data structure.*
>
> Hệ thống xây index trước khi tìm kiếm.  
> *The system builds the index before searching.*
>
> Index giúp tìm kiếm nhanh.  
> *The index enables fast search.*
>
> Index giống mục lục của một quyển sách dày.  
> *An index resembles the index of a thick book.*
>
> Không có mục lục, bạn phải lật từng trang.  
> *Without an index, you must inspect every page.*
>
> Có mục lục, bạn tìm trang cần thiết rất nhanh.  
> *With an index, you find the required page quickly.*

Bài pgvector đi sâu vào ANN index.  
*The pgvector lesson explores ANN indexes in depth.*

Bài này tập trung vào việc sản xuất vector.  
*This lesson focuses on producing vectors.*

ANN index tập trung vào việc tìm vector nhanh.  
*An ANN index focuses on finding vectors quickly.*

Đây là hai bài toán khác nhau.  
*These are two different problems.*

Chúng sử dụng hai nhóm kỹ thuật khác nhau.  
*They use two different groups of techniques.*

### ✅ Self-check Phần 1
*✅ Part 1 self-check*

Bạn hãy tự trả lời trước.  
*Answer the questions yourself first.*

Sau đó, bạn hãy xem gợi ý.  
*Then, review the suggested answers.*

**Câu 1. Cosine similarity đo hướng hay độ dài?**  
*Question 1. Does cosine similarity measure direction or length?*

Đặc tính này phù hợp với văn bản như thế nào?  
*How does this property suit text?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Cosine similarity đo hướng.  
*Cosine similarity measures direction.*

Nó hoàn toàn bỏ qua độ dài.  
*It completely ignores length.*

Công thức chia cho tích `|A| × |B|`.  
*The formula divides by the product `|A| × |B|`.*

Phép chia triệt tiêu ảnh hưởng của độ dài.  
*The division removes the effect of length.*

Một câu dài có thể cùng chủ đề với câu ngắn.  
*A long sentence can share a topic with a short sentence.*

Ta muốn máy nhận ra chủ đề giống nhau.  
*We want the computer to recognize similar topics.*

Ta không muốn phạt câu dài.  
*We do not want to penalize a long sentence.*

Ở Mục 1.7, vector B dài gấp đôi A.  
*In Section 1.7, vector B is twice as long as A.*

Cosine similarity vẫn bằng đúng 1.00.  
*Cosine similarity still equals exactly 1.00.*

</details>

**Câu 2. Hai câu cùng nghĩa cho cosine gần giá trị nào?**  
*Question 2. What cosine value indicates similar meaning?*

Hai câu không liên quan cho cosine gần giá trị nào?  
*What cosine value indicates unrelated sentences?*

Miền giá trị đầy đủ của cosine similarity là gì?  
*What is the full range of cosine similarity?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Hai câu cùng nghĩa cho giá trị gần 1.  
*Sentences with similar meaning produce a value near 1.*

Hai câu không liên quan cho giá trị gần 0.  
*Unrelated sentences produce a value near 0.*

Miền giá trị đầy đủ là `[−1, 1]`.  
*The full range is `[−1, 1]`.*

Miền giá trị không phải `[0, 1]`.  
*The range is not `[0, 1]`.*

Cosine chỉ nằm trong `[0, 1]` trong trường hợp riêng.  
*Cosine stays within `[0, 1]` only in a special case.*

Mọi thành phần của hai vector phải không âm.  
*Every component of both vectors must be nonnegative.*

Đây là một câu hỏi bẫy trong phỏng vấn.  
*This is a common interview trap.*

</details>

**Câu 3. Vì sao embedding không đọc được?**  
*Question 3. Why are embeddings unreadable?*

Vì sao embedding vẫn cực kỳ hữu dụng?  
*Why are embeddings still extremely useful?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Công dụng của embedding không nằm ở việc đọc.  
*The value of an embedding does not come from reading it.*

Công dụng nằm ở việc so sánh.  
*Its value comes from comparison.*

Từng chiều riêng lẻ không có nghĩa dễ diễn giải.  
*Each individual dimension has no easily interpretable meaning.*

Toàn bộ dãy số cùng nhau mã hóa ngữ nghĩa.  
*The complete sequence encodes meaning.*

Ngữ nghĩa xuất hiện qua phép đo khoảng cách.  
*Meaning appears through distance measurement.*

Ta không cần hiểu cách model vẽ bản đồ.  
*We do not need to understand the model's mapping process.*

Ta chỉ cần biết ý nghĩa của khoảng cách.  
*We only need to understand the meaning of distance.*

Hai điểm gần nhau biểu thị hai nội dung gần nghĩa.  
*Nearby points represent semantically similar content.*

</details>

**Câu 4. pgvector có tự tạo embedding từ câu văn không?**  
*Question 4. Does pgvector create an embedding from text?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Câu trả lời là sai.  
*The answer is false.*

pgvector chỉ lưu trữ vector.  
*pgvector only stores vectors.*

pgvector cũng tìm kiếm vector.  
*pgvector also searches vectors.*

Bạn phải tự gọi một embedding model.  
*You must call an embedding model yourself.*

Model biến câu văn thành vector.  
*The model converts text into a vector.*

Sau đó, bạn đưa vector vào pgvector.  
*You then send the vector to pgvector.*

Model là đầu bếp.  
*The model is the chef.*

Database là tủ lạnh.  
*The database is the refrigerator.*

</details>

---

## Phần 2 — 🟡 INTERMEDIATE (Vận dụng)
*Part 2 — 🟡 INTERMEDIATE (Application)*

> Từ phần này, tôi giả định bạn đã hiểu Phần 1.  
> *From this point, I assume you understand Part 1.*
>
> Embedding là một dãy số mã hóa ý nghĩa.  
> *An embedding is a number sequence that encodes meaning.*
>
> Model tạo ra embedding.  
> *A model creates embeddings.*
>
> Cosine đo góc giữa hai vector.  
> *Cosine measures the angle between two vectors.*
>
> Phần này giải thích cách model tạo embedding.  
> *This part explains how a model creates embeddings.*
>
> Phần này cũng hướng dẫn cách dùng embedding đúng.  
> *This part also explains correct embedding usage.*

### 2.1. Hai thế hệ embedding: static và contextual
*2.1. Two embedding generations: static and contextual*

Đây là sự phân biệt quan trọng nhất của bài.  
*This is the most important distinction in the lesson.*

Video gốc đã gộp hai thế hệ vào nhau.  
*The original video combines two generations.*

Cách trình bày đó gây hiểu lầm.  
*That presentation creates confusion.*

Video đặt Word2Vec cạnh GPT.  
*The video places Word2Vec beside GPT.*

Cách đặt này tạo cảm giác chúng cùng loại.  
*This placement suggests the same category.*

Chúng không cùng loại.  
*They do not belong to the same category.*

Chúng cách nhau một thế hệ công nghệ.  
*They are separated by a technology generation.*

#### Thế hệ 1: Static embedding — embedding tĩnh
*Generation 1: Static embedding*

> **Static embedding — embedding tĩnh**  
> *Static embedding — fixed embedding*
>
> Mỗi từ có đúng một vector cố định.  
> *Each word has exactly one fixed vector.*
>
> Vector đó không bao giờ thay đổi.  
> *That vector never changes.*
>
> Ngữ cảnh không làm vector thay đổi.  
> *Context does not change the vector.*
>
> Word2Vec là một đại diện.  
> *Word2Vec is one representative.*
>
> Google công bố Word2Vec năm 2013.  
> *Google released Word2Vec in 2013.*
>
> GloVe là một đại diện khác.  
> *GloVe is another representative.*
>
> Stanford công bố GloVe năm 2014.  
> *Stanford released GloVe in 2014.*
>
> Paragram cũng thuộc thế hệ này.  
> *Paragram also belongs to this generation.*
>
> Video gốc nhắc cả ba model.  
> *The original video mentions all three models.*

Cơ chế hoạt động rất trực tiếp.  
*The mechanism is very direct.*

Model học sẵn một bảng tra cứu khổng lồ.  
*The model learns a huge lookup table.*

Mỗi hàng tương ứng với một từ.  
*Each row corresponds to one word.*

Mỗi hàng chứa vector của từ đó.  
*Each row contains the word's vector.*

Số hàng bằng số từ trong từ điển.  
*The row count equals the vocabulary size.*

Bạn muốn lấy embedding của từ `"mèo"`.  
*You want the embedding for `"mèo"`.*

Model tra hàng mang tên `"mèo"`.  
*The model looks up the row named `"mèo"`.*

Model trả về vector trong hàng đó.  
*The model returns the vector in that row.*

Quy trình kết thúc tại đó.  
*The process ends there.*

Model học bằng co-occurrence.  
*The model learns through co-occurrence.*

> **Co-occurrence — đồng xuất hiện**  
> *Co-occurrence — joint occurrence*
>
> Co-occurrence là hiện tượng hai từ thường xuất hiện gần nhau.  
> *Co-occurrence means two words often appear near each other.*
>
> Model quét hàng tỉ câu.  
> *The model scans billions of sentences.*
>
> Model ghi lại thống kê về các từ xung quanh.  
> *The model records statistics about surrounding words.*
>
> Từ `"mèo"` thường xuất hiện gần `"kêu"`.  
> *The word `"mèo"` often appears near `"kêu"`.*
>
> Nó cũng thường xuất hiện gần `"meo"`.  
> *It also often appears near `"meo"`.*
>
> Các từ khác là `"chuột"` và `"lông"`.  
> *Other words include `"chuột"` and `"lông"`.*
>
> Hai từ có hàng xóm giống nhau.  
> *Two words have similar neighbors.*
>
> Model gán cho chúng vector giống nhau.  
> *The model assigns them similar vectors.*

Static embedding có một vấn đề chí mạng.  
*Static embeddings have a fatal limitation.*

Nó không phân biệt từ đồng âm khác nghĩa.  
*It cannot distinguish polysemous words.*

Ví dụ kinh điển là từ tiếng Anh `"bank"`.  
*The classic example is the English word `"bank"`.*

- `"I sat on the river bank"`.  
  *`"I sat on the river bank"`.*

- Trong câu này, `bank` có nghĩa là bờ sông.  
  *In this sentence, `bank` means a riverbank.*

- `"I opened a bank account"`.  
  *`"I opened a bank account"`.*

- Trong câu này, `bank` có nghĩa là ngân hàng.  
  *In this sentence, `bank` means a financial institution.*

Hai nghĩa hoàn toàn khác nhau.  
*The two meanings are completely different.*

Word2Vec chỉ có một hàng cho từ `"bank"`.  
*Word2Vec has only one row for `"bank"`.*

Nó phải trả cùng một vector cho hai nghĩa.  
*It must return the same vector for both meanings.*

Vector bị trung bình hóa giữa hai nghĩa.  
*The vector becomes an average of both meanings.*

Nó không đại diện chính xác cho nghĩa bờ sông.  
*It does not accurately represent the riverbank meaning.*

Nó cũng không đại diện chính xác cho nghĩa ngân hàng.  
*It also does not accurately represent the financial meaning.*

Tiếng Việt cũng có nhiều trường hợp tương tự.  
*Vietnamese also has many similar cases.*

`"Đường"` có thể chỉ đường đi.  
*`"Đường"` can mean a road.*

`"Đường"` cũng có thể chỉ đường ăn.  
*`"Đường"` can also mean sugar.*

`"Chín"` có thể chỉ số 9.  
*`"Chín"` can represent the number nine.*

`"Chín"` cũng có thể chỉ trạng thái đã chín.  
*`"Chín"` can also mean ripe.*

`"Bàn"` có thể chỉ một đồ vật.  
*`"Bàn"` can refer to a table.*

`"Bàn"` cũng có thể chỉ hành động thảo luận.  
*`"Bàn"` can also mean to discuss.*

Cách nhớ ngắn gọn là `một từ một nghĩa`.  
*The simple memory phrase is `one word, one meaning`.*

Static embedding hoạt động theo giả định đó.  
*Static embeddings operate under that assumption.*

Ngôn ngữ thực tế không hoạt động như vậy.  
*Real language does not work that way.*

#### Thế hệ 2: Contextual embedding — embedding theo ngữ cảnh
*Generation 2: Contextual embedding*

> **Contextual embedding — embedding theo ngữ cảnh**  
> *Contextual embedding — context-dependent embedding*
>
> Vector của một từ phụ thuộc vào toàn bộ câu.  
> *A word vector depends on the entire sentence.*
>
> Cùng một từ có thể nhận nhiều vector.  
> *The same word can receive many vectors.*
>
> Ngữ cảnh khác tạo ra vector khác.  
> *Different contexts create different vectors.*
>
> BERT là một đại diện.  
> *BERT is one representative.*
>
> Google công bố BERT năm 2018.  
> *Google released BERT in 2018.*
>
> GPT của OpenAI cũng thuộc nhóm này.  
> *OpenAI's GPT also belongs to this group.*
>
> Universal Sentence Encoder cũng thuộc nhóm này.  
> *Universal Sentence Encoder also belongs to this group.*
>
> Các model hiện đại năm 2026 đều dùng cách này.  
> *Modern models in 2026 use this approach.*
>
> Chúng dựa trên kiến trúc transformer.  
> *They rely on the transformer architecture.*
>
> **Transformer**  
> *Transformer*
>
> Transformer là một kiến trúc mạng nơ-ron.  
> *A transformer is a neural network architecture.*
>
> Kiến trúc này được công bố năm 2017.  
> *The architecture was introduced in 2017.*
>
> Nó đứng sau hầu hết tiến bộ AI gần đây.  
> *It powers most recent AI advances.*
>
> ChatGPT sử dụng transformer.  
> *ChatGPT uses transformers.*
>
> Claude sử dụng transformer.  
> *Claude uses transformers.*
>
> Gemini cũng sử dụng transformer.  
> *Gemini also uses transformers.*
>
> Cơ chế cốt lõi là self-attention.  
> *The core mechanism is self-attention.*
>
> **Self-attention — tự chú ý**  
> *Self-attention — contextual attention mechanism*
>
> Mỗi từ có thể nhìn các từ khác trong câu.  
> *Each word can inspect other words in the sentence.*
>
> Từ đó tự chọn từ cần chú ý.  
> *It then selects the relevant words.*
>
> Mục tiêu là hiểu nghĩa của chính nó.  
> *The goal is to understand its own meaning.*
>
> Hãy xét từ `"bank"` trong câu về bờ sông.  
> *Consider `"bank"` in the river sentence.*
>
> Self-attention cho `"bank"` nhìn thấy `"river"`.  
> *Self-attention lets `"bank"` see `"river"`.*
>
> Model nhận ra ngữ cảnh sông.  
> *The model recognizes the river context.*
>
> Model điều chỉnh vector về nghĩa bờ sông.  
> *The model adjusts the vector toward the riverbank meaning.*

Contextual embedding giải được vấn đề của static embedding.  
*Contextual embeddings solve the static embedding problem.*

Hai chữ `"bank"` nhận hai vector khác nhau.  
*The two `"bank"` occurrences receive different vectors.*

Mỗi vector phản ánh đúng ngữ cảnh.  
*Each vector reflects the correct context.*

Contextual embedding còn nhận ra nhiều sắc thái.  
*Contextual embeddings also capture many nuances.*

Static embedding hoàn toàn bỏ lỡ các sắc thái này.  
*Static embeddings completely miss these nuances.*

- **Phủ định**  
  ***Negation***

- Câu thứ nhất là `"phim này không dở"`.  
  *The first sentence is `"phim này không dở"`.*

- Câu thứ hai là `"phim này khá hay"`.  
  *The second sentence is `"phim này khá hay"`.*

- Contextual embedding nhận ra hai câu gần nghĩa.  
  *Contextual embeddings recognize their similar meaning.*

- Static embedding thấy từ `"dở"`.  
  *Static embeddings see the word `"dở"`.*

- Nó đẩy câu thứ nhất về phía tiêu cực.  
  *It pushes the first sentence toward a negative meaning.*

- **Toàn bộ câu**  
  ***The complete sentence***

- Static embedding cần lấy trung bình vector từ.  
  *Static embeddings require averaging word vectors.*

- Cách này làm mất thông tin về trật tự.  
  *This approach loses order information.*

- Cách này cũng làm mất quan hệ giữa các từ.  
  *This approach also loses relationships between words.*

- Transformer xử lý cả câu như một thể thống nhất.  
  *A transformer processes the sentence as one unit.*

#### Bảng so sánh
*Comparison table*

| Tiêu chí / Criterion | Static (Word2Vec, GloVe) | Contextual (BERT, GPT, model 2026) |
|---|---|---|
| Số vector cho một từ / Vectors per word | 1, cố định vĩnh viễn / 1, permanently fixed | Vô số, tùy ngữ cảnh / Unlimited, context-dependent |
| Từ `bank` có hai nghĩa / Two meanings of `bank` | Cùng 1 vector ❌ / Same vector ❌ | 2 vector khác nhau ✅ / Two different vectors ✅ |
| Hiểu phủ định và sắc thái / Understands negation and nuance | Không / No | Có / Yes |
| Cách lấy vector / Vector retrieval | Tra bảng, cực nhanh / Table lookup, extremely fast | Chạy model, chậm hơn nhiều / Model inference, much slower |
| Chi phí tính toán / Compute cost | Rất thấp / Very low | Cao, chạy model cho từng input / High, model inference for every input |
| Kích thước / Size | Nhỏ / Small | Hàng trăm MB đến hàng chục GB / Hundreds of MB to tens of GB |
| Vị thế năm 2026 / Status in 2026 | Giá trị lịch sử / Historical value | Tiêu chuẩn thực tế / Practical standard |

🧩 **[Ngoài bài gốc] Giá trị của mục này trong phỏng vấn**  
*🧩 **[Beyond the original lesson] Interview value of this section***

Nhà tuyển dụng có thể hỏi `"embedding là gì?"`.  
*An interviewer may ask, `"What is an embedding?"`.*

Câu trả lời tầm thường là `"một mảng số"`.  
*The ordinary answer is `"an array of numbers"`.*

Hầu hết ứng viên đều nói được câu đó.  
*Most candidates can give that answer.*

Câu đó không chứng minh hiểu biết sâu.  
*That answer does not demonstrate deep understanding.*

Câu trả lời senior hoặc staff cần đầy đủ hơn.  
*A senior or staff answer needs more depth.*

Embedding là một mảng số mã hóa ngữ nghĩa.  
*An embedding is a numeric array that encodes meaning.*

Bạn cần phân biệt static embedding.  
*You need to distinguish static embeddings.*

Static embedding dùng một vector cố định cho mỗi từ.  
*Static embeddings use one fixed vector per word.*

Word2Vec là một ví dụ.  
*Word2Vec is one example.*

Bạn cũng cần phân biệt contextual embedding.  
*You also need to distinguish contextual embeddings.*

Contextual embedding đến từ transformer.  
*Contextual embeddings come from transformers.*

Vector phụ thuộc vào ngữ cảnh.  
*The vector depends on context.*

Self-attention tạo khả năng đó.  
*Self-attention enables this capability.*

Transformer tính embedding theo điều kiện ngữ cảnh.  
*Transformers compute context-conditioned embeddings.*

Đó là lý do transformer vượt trội.  
*That is why transformers perform better.*

Câu trả lời này thể hiện hiểu biết về lịch sử.  
*This answer demonstrates historical understanding.*

Câu trả lời này cũng thể hiện hiểu biết về cơ chế.  
*This answer also demonstrates mechanistic understanding.*

Nó không chỉ là một định nghĩa học thuộc.  
*It is not merely a memorized definition.*

### 2.2. Từ chữ đến vector: chuyện gì xảy ra bên trong
*2.2. From text to vectors: what happens inside*

Ở Mục 1.8, bạn gọi `extractor(text)`.  
*In Section 1.8, you call `extractor(text)`.*

Bạn nhận về 384 con số.  
*You receive 384 numbers.*

Bên trong hộp đen có bốn bước.  
*The black box contains four steps.*

Việc hiểu chúng giúp bạn tránh ba lỗi.  
*Understanding them helps you avoid three mistakes.*

Mục 2.5 sẽ trình bày ba lỗi đó.  
*Section 2.5 will present those three mistakes.*

```
   "con mèo đang ngủ"
          │
          ▼  BƯỚC 1: TOKENIZE (cắt thành token)
             STEP 1: TOKENIZE (split into tokens)
   ["con", "mèo", "đang", "ngủ"]
          │
          ▼  BƯỚC 2: CHẠY TRANSFORMER
             STEP 2: RUN THE TRANSFORMER
   [vec_con, vec_mèo, vec_đang, vec_ngủ]     ← MỘT vector cho MỖI token!
                                               ONE vector for EACH token!
          │
          ▼  BƯỚC 3: POOLING (gộp lại)
             STEP 3: POOLING (merge vectors)
   [vec_cả_câu]                               ← chỉ còn MỘT vector
                                               only ONE vector remains
          │
          ▼  BƯỚC 4: NORMALIZE (chuẩn hóa)
             STEP 4: NORMALIZE
   [vec_cả_câu, độ dài = 1]                   ← sẵn sàng để so sánh
   [sentence_vector, length = 1]              ← ready for comparison
```

**Bước 1: Tokenize**  
*Step 1: Tokenize*

> **Token**  
> *Token*
>
> Token là đơn vị nhỏ nhất mà model xử lý.  
> *A token is the smallest unit processed by a model.*
>
> Token không hoàn toàn giống một từ.  
> *A token is not always a complete word.*
>
> Token có thể là một từ.  
> *A token can be a word.*
>
> Token có thể là một phần của từ.  
> *A token can be part of a word.*
>
> Token cũng có thể là một dấu câu.  
> *A token can also be punctuation.*
>
> Hãy xét từ tiếng Anh `"unhappiness"`.  
> *Consider the English word `"unhappiness"`.*
>
> Tokenizer thường cắt nó thành ba token.  
> *A tokenizer often splits it into three tokens.*
>
> Ba token là `un`, `happi` và `ness`.  
> *The three tokens are `un`, `happi`, and `ness`.*
>
> Cách cắt này giúp model xử lý từ mới.  
> *This splitting method helps the model process new words.*
>
> Model có thể chưa từng thấy từ hoàn chỉnh.  
> *The model may never have seen the complete word.*
>
> **Tokenizer — bộ tách token**  
> *Tokenizer — token splitting component*
>
> Tokenizer thực hiện việc cắt văn bản.  
> *A tokenizer splits text.*
>
> Mỗi model có tokenizer riêng.  
> *Each model has its own tokenizer.*
>
> Tokenizer luôn đi kèm model.  
> *The tokenizer always accompanies the model.*
>
> Tokenizer của model khác có thể tạo kết quả sai.  
> *A tokenizer from another model can produce incorrect results.*

💡 Quy tắc sau giúp bạn ước lượng chi phí.  
*The following rule helps you estimate cost.*

**1 token ≈ 0.75 từ tiếng Anh.**  
*One token is approximately 0.75 English words.*

Nói cách khác, 1000 token tương đương khoảng 750 từ.  
*In other words, 1,000 tokens equal about 750 words.*

Tiếng Việt thường dùng nhiều token hơn.  
*Vietnamese usually uses more tokens.*

Cùng một nội dung có thể tốn gấp 1.5–2 lần.  
*The same content can require 1.5–2 times more tokens.*

Lý do nằm ở cách tối ưu tokenizer.  
*The reason lies in tokenizer optimization.*

Phần lớn tokenizer được tối ưu cho tiếng Anh.  
*Most tokenizers are optimized for English.*

Chi tiết này ảnh hưởng trực tiếp đến hóa đơn.  
*This detail directly affects your bill.*

Phần 4 sẽ dùng lại chi tiết này.  
*Part 4 will use this detail again.*

**Bước 2: Chạy transformer**  
*Step 2: Run the transformer*

Đây là bước quan trọng nhất.  
*This is the most important step.*

Bước này thường gây bất ngờ cho người mới.  
*This step often surprises beginners.*

> ⚠️ Transformer trả về một vector cho mỗi token.  
> *⚠️ A transformer returns one vector for each token.*
>
> Transformer không trả về một vector cho cả câu.  
> *A transformer does not return one vector for the whole sentence.*

Câu trên có 4 token.  
*The sentence above has four tokens.*

Transformer tạo ra **4 vector**.  
*The transformer produces **four vectors**.*

Mỗi vector có 384 chiều.  
*Each vector has 384 dimensions.*

Theo ngôn ngữ kỹ thuật, kết quả có một shape.  
*In technical language, the result has a shape.*

Shape đó là `[1, 4, 384]`.  
*That shape is `[1, 4, 384]`.*

> **Shape — hình dạng**  
> *Shape — data dimensions*
>
> Shape mô tả kích thước của một khối dữ liệu nhiều tầng.  
> *A shape describes the dimensions of a multilayer data block.*
>
> `[1, 4, 384]` biểu thị 1 câu.  
> *`[1, 4, 384]` represents one sentence.*
>
> Câu đó có 4 token.  
> *That sentence has four tokens.*
>
> Mỗi token có 384 số.  
> *Each token has 384 numbers.*
>
> **Tensor — khối số nhiều chiều**  
> *Tensor — a multidimensional block of numbers*
>
> Tensor là tên chung cho khối số nhiều chiều.  
> *A tensor is a general name for a multidimensional number block.*
>
> Vector là tensor 1 chiều.  
> *A vector is a one-dimensional tensor.*
>
> Ma trận là tensor 2 chiều.  
> *A matrix is a two-dimensional tensor.*
>
> `[1, 4, 384]` là tensor 3 chiều.  
> *`[1, 4, 384]` is a three-dimensional tensor.*
>
> Thuật ngữ này xuất hiện khắp nơi trong AI.  
> *This term appears everywhere in AI.*
>
> Bạn nên làm quen với thuật ngữ này.  
> *You should become familiar with this term.*

Bạn cần một vector đại diện cho cả câu.  
*You need one vector to represent the whole sentence.*

Bạn không cần 4 vector riêng biệt.  
*You do not need four separate vectors.*

Vì vậy, pipeline cần bước 3.  
*Therefore, the pipeline needs step 3.*

**Bước 3: Pooling**  
*Step 3: Pooling*

> **Pooling — phép gộp vector**  
> *Pooling — vector aggregation*
>
> Pooling biến nhiều vector thành một vector duy nhất.  
> *Pooling converts many vectors into one vector.*
>
> **Mean pooling — gộp bằng trung bình**  
> *Mean pooling — aggregation by averaging*
>
> Mean pooling là cách phổ biến nhất.  
> *Mean pooling is the most common method.*
>
> Nó lấy trung bình cộng của mọi vector token.  
> *It takes the arithmetic mean of all token vectors.*
>
> Phép tính được thực hiện riêng cho từng chiều.  
> *The calculation is performed separately for each dimension.*

Ví dụ sau dùng vector 3 chiều.  
*The following example uses three-dimensional vectors.*

Cách này giúp ví dụ dễ quan sát.  
*This choice makes the example easier to inspect.*

```
vec_con   = [0.2, 0.8, 0.1]
vec_mèo   = [0.6, 0.4, 0.3]
vec_đang  = [0.1, 0.2, 0.5]
vec_ngủ   = [0.5, 0.6, 0.1]
──────────────────────────────  cộng theo cột rồi chia 4
                                add each column and divide by 4
mean pool = [0.35, 0.5, 0.25]     ← (0.2+0.6+0.1+0.5)/4 = 0.35, v.v.
                                         (0.2+0.6+0.1+0.5)/4 = 0.35, etc.
```

Quy trình chỉ đơn giản như vậy.  
*The process is that simple.*

Một số model dùng cách khác.  
*Some models use another method.*

Model đó lấy vector của token đặc biệt `[CLS]`.  
*Such a model uses the vector of a special `[CLS]` token.*

Token `[CLS]` nằm ở đầu câu.  
*The `[CLS]` token appears at the start of the sentence.*

Token này được huấn luyện để tóm tắt cả câu.  
*This token is trained to summarize the whole sentence.*

Mean pooling vẫn là cách mặc định phổ biến nhất.  
*Mean pooling remains the most common default method.*

Tham số `pooling: 'mean'` thực hiện cách này.  
*The `pooling: 'mean'` parameter performs this method.*

Mục 1.8 đã dùng tham số đó.  
*Section 1.8 used that parameter.*

**Bước 4: Normalize**  
*Step 4: Normalize*

> **Normalize — chuẩn hóa**  
> *Normalize — normalization*
>
> Ở đây, normalize có nghĩa là L2 normalization.  
> *Here, normalize means L2 normalization.*
>
> Phép này chia mọi thành phần cho độ dài vector.  
> *This operation divides every component by the vector length.*
>
> Vector kết quả có độ dài đúng bằng 1.  
> *The resulting vector has a length of exactly 1.*
>
> Về hình học, hướng mũi tên được giữ nguyên.  
> *Geometrically, the arrow direction remains unchanged.*
>
> Phép chuẩn hóa chỉ co hoặc giãn mũi tên.  
> *Normalization only shrinks or stretches the arrow.*
>
> Mũi tên cuối cùng dài đúng 1 đơn vị.  
> *The final arrow is exactly one unit long.*

Ví dụ sau dùng vector `A = (3, 4)`.  
*The following example uses the vector `A = (3, 4)`.*

```
Độ dài:      |A| = sqrt(3² + 4²) = sqrt(9+16) = sqrt(25) = 5
Length:      |A| = sqrt(3² + 4²) = sqrt(9+16) = sqrt(25) = 5
Chuẩn hóa:   A_norm = (3/5, 4/5) = (0.6, 0.8)
Normalize:   A_norm = (3/5, 4/5) = (0.6, 0.8)
Kiểm tra:    sqrt(0.6² + 0.8²) = sqrt(0.36 + 0.64) = sqrt(1) = 1  ✅
Check:       sqrt(0.6² + 0.8²) = sqrt(0.36 + 0.64) = sqrt(1) = 1  ✅
```

Bước này có hai lý do rất thực dụng.  
*This step has two very practical reasons.*

**Lý do một: phép so sánh trở nên rẻ hơn.**  
*Reason one: comparison becomes cheaper.*

Hãy nhìn lại công thức cosine tại Mục 1.7.  
*Look again at the cosine formula in Section 1.7.*

```
cosine(A, B) = (A · B) / (|A| × |B|)
```

Giả sử A đã được chuẩn hóa.  
*Suppose A has been normalized.*

Khi đó, `|A| = 1`.  
*In that case, `|A| = 1`.*

Giả sử B cũng đã được chuẩn hóa.  
*Suppose B has also been normalized.*

Khi đó, `|B| = 1`.  
*In that case, `|B| = 1`.*

Mẫu số lúc này bằng 1.  
*The denominator now equals 1.*

Công thức được rút gọn như sau.  
*The formula simplifies as follows.*

```
cosine(A, B) = A · B        ← chỉ còn dot product!
                                 only the dot product remains!
```

Mỗi phép so sánh không còn hai phép căn bậc hai.  
*Each comparison no longer needs two square roots.*

Mỗi phép so sánh cũng không còn một phép chia.  
*Each comparison also no longer needs one division.*

Mức tiết kiệm này có vẻ nhỏ.  
*This saving may appear small.*

Hệ thống có thể thực hiện hàng triệu phép so sánh mỗi giây.  
*A system may perform millions of comparisons per second.*

Khi đó, khác biệt hiệu năng trở nên rất lớn.  
*The performance difference then becomes very large.*

Vì vậy, vector database thường khuyến nghị chuẩn hóa trước.  
*Therefore, vector databases often recommend prior normalization.*

Bạn nên lưu các vector đã chuẩn hóa.  
*You should store normalized vectors.*

**Lý do hai: mọi phép đo trở nên nhất quán.**  
*Reason two: every measurement becomes consistent.*

Database có thể chứa vector đã chuẩn hóa.  
*The database may contain normalized vectors.*

Database cũng có thể chứa vector chưa chuẩn hóa.  
*The database may also contain unnormalized vectors.*

Sự pha trộn này làm sai thứ tự xếp hạng.  
*This mixture distorts the result ranking.*

Lỗi đó thường rất khó phát hiện.  
*That error is often very difficult to detect.*

Chuẩn hóa toàn bộ vector là cách phòng tránh.  
*Normalizing every vector is a preventive measure.*

### 2.3. 🧩 [Ngoài bài gốc] Bức tranh model năm 2026: ai đang dùng gì
*2.3. 🧩 [Beyond the original lesson] The 2026 model landscape: who uses what*

Video gốc liệt kê Word2Vec, GloVe, Paragram và USE.  
*The original video lists Word2Vec, GloVe, Paragram, and USE.*

Video cũng nhắc tới OpenAI và Google.  
*The video also mentions OpenAI and Google.*

Danh sách đó đã cũ.  
*That list is outdated.*

Bảng sau cập nhật bức tranh vào tháng 7/2026.  
*The following table updates the landscape for July 2026.*

Tôi đã tra cứu lại trước khi viết.  
*I checked the information again before writing.*

| Model / Model | Kiểu / Type | Số chiều / Dimensions | Điểm mạnh / Strength |
|---|---|---|---|
| **Google Gemini Embedding** / **Google Gemini Embedding** | API trả phí / Paid API | 3072 / 3072 | Dẫn đầu bảng xếp hạng tiếng Anh trong phần lớn 2026. Hỗ trợ multimodal cho text, ảnh, video, audio và PDF trong cùng không gian vector. / Led English rankings for much of 2026. Supports multimodal text, images, video, audio, and PDF in one vector space. |
| **Cohere `embed-v4`** / **Cohere `embed-v4`** | API trả phí / Paid API | Tùy chỉnh / Configurable | Hỗ trợ hơn 100 ngôn ngữ. Đây là model đầu tiên đưa multimodal vào production ở quy mô lớn. / Supports more than 100 languages. It was the first model to bring multimodal embeddings into large-scale production. |
| **OpenAI `text-embedding-3-small`** / **OpenAI `text-embedding-3-small`** | API trả phí / Paid API | 1536 / 1536 | Rẻ, đơn giản và có tài liệu tốt. Đây là lựa chọn khởi đầu an toàn cho tiếng Anh. / Affordable, simple, and well documented. It is a safe starting choice for English. |
| **OpenAI `text-embedding-3-large`** / **OpenAI `text-embedding-3-large`** | API trả phí / Paid API | 3072 / 3072 | Mạnh hơn bản small. Model này rất phổ biến trong doanh nghiệp. / Stronger than the small version. This model is very popular in enterprises. |
| **Voyage `voyage-3.x/4`** / **Voyage `voyage-3.x/4`** | API trả phí / Paid API | Tùy chỉnh / Configurable | Tối ưu riêng cho tìm kiếm. Các bản chuyên ngành luật, tài chính hoặc code vượt model chung 10–15% trong lĩnh vực tương ứng. / Optimized specifically for retrieval. Domain versions for law, finance, or code outperform general models by 10–15% in their fields. |
| **Qwen3-Embedding-8B** (Alibaba) / **Qwen3-Embedding-8B** (Alibaba) | Mã nguồn mở / Open source | 4096 / 4096 | Rất mạnh về đa ngôn ngữ. Model đã vượt OpenAI trên benchmark. Đây là lựa chọn self-host mặc định năm 2026. / Very strong for multilingual use. The model has surpassed OpenAI on benchmarks. It is the default self-hosted choice in 2026. |
| **BGE-M3** (BAAI) / **BGE-M3** (BAAI) | Mã nguồn mở / Open source | 1024 / 1024 | Là chuẩn mực mã nguồn mở cho hơn 100 ngôn ngữ. Model xuất đồng thời vector dày và vector thưa. / An open-source standard for more than 100 languages. The model produces dense and sparse vectors together. |
| **Jina v5-text-small** / **Jina v5-text-small** | Mã nguồn mở / Open source | Tùy chỉnh / Configurable | Có tỉ lệ chất lượng trên kích thước tốt nhất. Model mạnh nhưng nhẹ. / Offers the best quality-to-size ratio. The model is powerful yet lightweight. |
| **`all-MiniLM-L6-v2`** / **`all-MiniLM-L6-v2`** | Mã nguồn mở / Open source | 384 / 384 | Chỉ khoảng 23 MB. Model chạy được trong trình duyệt. Đây là model mặc định để học và làm demo. / Only about 23 MB. The model runs in a browser. It is the default model for learning and demos. |
| **TensorFlow USE** / **TensorFlow USE** | Mã nguồn mở / Open source | 512 / 512 | Video gốc sử dụng model này. Model đã cũ. Bạn không nên chọn nó cho hệ thống mới. / The original video uses this model. The model is outdated. You should not choose it for a new system. |

> **API — Application Programming Interface**  
> *API — Application Programming Interface*
>
> Trong ngữ cảnh này, bạn gửi dữ liệu qua internet.  
> *In this context, you send data through the internet.*
>
> Dữ liệu được gửi tới server của một công ty.  
> *The data is sent to a company's server.*
>
> Công ty xử lý dữ liệu.  
> *The company processes the data.*
>
> Công ty trả kết quả cho bạn.  
> *The company returns the result to you.*
>
> Công ty tính tiền theo lượng sử dụng.  
> *The company charges you according to usage.*
>
> Bạn không cần một máy mạnh.  
> *You do not need a powerful machine.*
>
> Dữ liệu phải rời khỏi hệ thống của bạn.  
> *The data must leave your system.*
>
> **Mã nguồn mở / self-host — tự vận hành**  
> *Open source / self-host — operate it yourself*
>
> Bạn tải model về máy chủ của mình.  
> *You download the model to your own server.*
>
> Bạn chạy model trên máy chủ đó.  
> *You run the model on that server.*
>
> Bạn không trả tiền theo từng lượt dùng.  
> *You do not pay per request.*
>
> Dữ liệu không rời khỏi hệ thống.  
> *The data does not leave the system.*
>
> Bạn phải tự quản lý hạ tầng.  
> *You must manage the infrastructure yourself.*
>
> Hạ tầng thường cần máy có GPU.  
> *The infrastructure usually needs a machine with a GPU.*
>
> **GPU — Graphics Processing Unit**  
> *GPU — Graphics Processing Unit*
>
> GPU là card đồ họa.  
> *A GPU is a graphics card.*
>
> GPU vốn được tạo ra cho trò chơi điện tử.  
> *GPUs were originally created for video games.*
>
> GPU thực hiện hàng nghìn phép tính song song rất tốt.  
> *A GPU performs thousands of parallel calculations very well.*
>
> Vì vậy, GPU trở thành phần cứng quan trọng cho AI.  
> *Therefore, GPUs became important hardware for AI.*
>
> GPU giúp model chạy với tốc độ chấp nhận được.  
> *A GPU helps a model run at an acceptable speed.*
>
> **Multimodal — đa phương thức**  
> *Multimodal — multiple data modalities*
>
> Model multimodal xử lý được nhiều loại dữ liệu.  
> *A multimodal model processes several data types.*
>
> Các loại đó gồm chữ, ảnh và âm thanh.  
> *Those types include text, images, and audio.*
>
> Model nhúng chúng vào cùng một không gian vector.  
> *The model embeds them into the same vector space.*
>
> Khả năng này tạo ra một hệ quả rất mạnh.  
> *This capability creates a very powerful result.*
>
> Bạn có thể nhập `"con mèo vàng đang ngủ"`.  
> *You can enter `"a sleeping yellow cat"`.*
>
> Hệ thống có thể tìm một tấm ảnh phù hợp.  
> *The system can find a matching image.*
>
> Chữ nằm trên cùng bản đồ với ảnh.  
> *Text lies on the same map as images.*

**Quy tắc chọn nhanh**  
*Quick selection rules*

Bạn có thể cần quyết định trong 30 giây.  
*You may need to decide within 30 seconds.*

| Tình huống / Situation | Chọn / Choose |
|---|---|
| Chỉ dùng tiếng Anh. Bạn muốn cách đơn giản và chạy ngay. / You use only English. You want a simple option that works immediately. | OpenAI `text-embedding-3-small` / OpenAI `text-embedding-3-small` |
| Có tiếng Việt hoặc nhiều ngôn ngữ. / You have Vietnamese or multiple languages. | BGE-M3, Qwen3-Embedding hoặc Cohere `embed-v4` / BGE-M3, Qwen3-Embedding, or Cohere `embed-v4` |
| Dữ liệu nhạy cảm không được rời khỏi công ty. / Sensitive data must not leave the company. | Bắt buộc self-host bằng Qwen3-Embedding hoặc BGE-M3. / Self-hosting with Qwen3-Embedding or BGE-M3 is mandatory. |
| Chạy trong trình duyệt, máy yếu hoặc dùng để học. / Run in a browser, on a weak machine, or for learning. | `all-MiniLM-L6-v2` / `all-MiniLM-L6-v2` |
| Cần tìm ảnh bằng chữ hoặc tìm chữ bằng ảnh. / Need text-to-image or image-to-text retrieval. | Gemini Embedding hoặc Cohere `embed-v4` / Gemini Embedding or Cohere `embed-v4` |

⚠️ **Cảnh báo về độ tươi của thông tin**  
*⚠️ Information freshness warning*

Bảng xếp hạng embedding thay đổi gần như mỗi tháng.  
*Embedding rankings change almost every month.*

Bảng trên đúng tại thời điểm tháng 7/2026.  
*The table above is accurate as of July 2026.*

Bạn nên kiểm tra trước khi chọn model cho dự án thật.  
*You should check again before choosing a model for a real project.*

Bạn cũng nên kiểm tra trước một buổi phỏng vấn.  
*You should also check before an interview.*

Hãy mở bảng xếp hạng MTEB trên Hugging Face.  
*Open the MTEB leaderboard on Hugging Face.*

Hãy xem tình hình mới nhất tại đó.  
*Check the latest situation there.*

Bạn không nên chỉ nhìn vị trí trên bảng xếp hạng.  
*You should not rely only on leaderboard position.*

Phần 4 sẽ giải thích lý do.  
*Part 4 will explain the reason.*

### 2.4. Code thật: ba con đường
*2.4. Real code: three paths*

Ba đoạn code sau thực hiện cùng một việc.  
*The following three code examples perform the same task.*

Chúng biến chữ thành vector.  
*They convert text into vectors.*

Mỗi đoạn phù hợp với một hoàn cảnh khác nhau.  
*Each example suits a different situation.*

Bạn nên đọc cả ba đoạn.  
*You should read all three examples.*

Khi đó, bạn sẽ biết lúc nào nên dùng từng cách.  
*You will then know when to use each approach.*

#### (a) JavaScript + Transformers.js: chạy local, miễn phí, không cần API key
*(a) JavaScript + Transformers.js: run locally, free, with no API key*

Đây là bản cập nhật của cách làm trong video gốc.  
*This is an updated version of the method in the original video.*

```javascript
// npm install @huggingface/transformers      (tên gói MỚI, video gốc dùng @xenova/transformers)
// npm install @huggingface/transformers      (NEW package name, the original video uses @xenova/transformers)
import { pipeline } from '@huggingface/transformers';

// Nạp model MỘT LẦN duy nhất.
// Load the model only ONCE.
// Dùng lại model cho mọi câu.
// Reuse the model for every sentence.
// ⚠️ LỖI HAY GẶP: gọi pipeline() bên trong hàm embed().
// ⚠️ COMMON MISTAKE: calling pipeline() inside the embed() function.
// Cách đó nạp lại model sau mỗi lần gọi.
// That approach reloads the model after every call.
// Chương trình có thể chậm hơn hàng trăm lần.
// The program can become hundreds of times slower.
// Luôn nạp model bên ngoài hàm.
// Always load the model outside the function.
const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');

// Hàm tiện ích nhận vào một chuỗi.
// This utility function accepts a string.
// Hàm trả về một mảng thường gồm 384 số.
// The function returns a regular array of 384 numbers.
async function embed(text) {
  const out = await extractor(text, {
    pooling: 'mean',     // gộp vector từng token thành một vector câu
                         // merge token vectors into one sentence vector
                         // Xem Mục 2.2, bước 3.
                         // See Section 2.2, step 3.
    normalize: true      // đưa độ dài về 1
                         // scale the length to 1
                         // Xem Mục 2.2, bước 4.
                         // See Section 2.2, step 4.
  });
  return Array.from(out.data);   // đổi Float32Array thành mảng JavaScript thường
                                 // convert Float32Array into a regular JavaScript array
                                 // Mảng này dễ lưu hoặc gửi.
                                 // This array is easy to store or send.
}

// Xử lý NHIỀU câu cùng lúc.
// Process MANY sentences at once.
// Cách này nhanh hơn gọi từng câu riêng lẻ.
// This approach is much faster than processing sentences separately.
// Phần 4 giải thích lý do.
// Part 4 explains the reason.
// Đây là tối ưu quan trọng nhất cho dữ liệu lớn.
// This is the most important optimization for large datasets.
const vectors = await Promise.all([
  embed('áo thun cotton nam'),
  embed('áo phông nam'),
  embed('xe máy Honda')
]);

console.log(vectors[0].length);   // 384
                                 // 384
```

**Khi nào dùng**  
*When to use it*

Dùng cách này để làm demo.  
*Use this approach for demos.*

Bạn có thể chạy nó trong trình duyệt.  
*You can run it in a browser.*

Nó cũng phù hợp với ứng dụng nhỏ.  
*It also suits small applications.*

Cách này không phụ thuộc dịch vụ bên ngoài.  
*This approach does not depend on an external service.*

#### (b) JavaScript + TensorFlow.js Universal Sentence Encoder: đúng như video gốc
*(b) JavaScript + TensorFlow.js Universal Sentence Encoder: the original video approach*

```javascript
// npm install @tensorflow/tfjs @tensorflow-models/universal-sentence-encoder
// npm install @tensorflow/tfjs @tensorflow-models/universal-sentence-encoder
import * as use from '@tensorflow-models/universal-sentence-encoder';

// Tải model USE đã huấn luyện sẵn.
// Load the pretrained USE model.
// Model có kích thước khoảng 25 MB.
// The model is about 25 MB.
// Model chỉ được tải một lần.
// The model is downloaded only once.
// Sau đó, model được lưu trong cache.
// The model is then stored in the cache.
const model = await use.load();

// USE nhận vào một MẢNG câu.
// USE accepts an ARRAY of sentences.
// USE trả về một tensor.
// USE returns a tensor.
// USE tự thực hiện pooling bên trong.
// USE performs pooling internally.
// Bạn không cần chỉ định pooling.
// You do not need to specify pooling.
const embeddings = await model.embed([
  'áo thun cotton nam',
  'áo phông nam'
]);

console.log(embeddings.shape);              // [2, 512] -> 2 câu, mỗi câu 512 chiều
                                             // [2, 512] -> 2 sentences, 512 dimensions per sentence

// .array() chuyển tensor thành mảng JavaScript thường.
// .array() converts the tensor into a regular JavaScript array.
// Mảng này có thể được dùng tiếp.
// This array can be used in later processing.
const arr = await embeddings.array();
console.log(arr[0].length);                 // 512
                                             // 512

// ⚠️ TensorFlow.js yêu cầu giải phóng bộ nhớ thủ công.
// ⚠️ TensorFlow.js requires manual memory release.
// TF.js cấp phát bộ nhớ ngoài trình dọn rác JavaScript.
// TF.js allocates memory outside the JavaScript garbage collector.
// Việc quên dispose() gây rò rỉ bộ nhớ.
// Forgetting dispose() causes a memory leak.
// Chương trình có thể sập sau thời gian dài.
// The program can crash after running for a long time.
embeddings.dispose();
```

> **Universal Sentence Encoder — USE**  
> *Universal Sentence Encoder — USE*
>
> USE là một model của Google.  
> *USE is a model from Google.*
>
> Model tạo embedding 512 chiều cho cả câu.  
> *The model creates a 512-dimensional embedding for a whole sentence.*
>
> Video gốc sử dụng model này.  
> *The original video uses this model.*
>
> Model vẫn hoạt động tốt.  
> *The model still works well.*
>
> Chất lượng của nó đã lạc hậu vào năm 2026.  
> *Its quality is outdated in 2026.*
>
> Bạn chỉ nên dùng nó khi bảo trì hệ thống cũ.  
> *You should use it only when maintaining a legacy system.*
>
> **Tensor `.dispose()`**  
> *Tensor `.dispose()`*
>
> Video gốc không nhắc chi tiết này.  
> *The original video does not mention this detail.*
>
> Người mới thường bỏ sót nó.  
> *Beginners often overlook it.*
>
> TensorFlow.js quản lý bộ nhớ thủ công.  
> *TensorFlow.js manages memory manually.*
>
> TensorFlow.js không tự dọn bộ nhớ đó.  
> *TensorFlow.js does not clean up that memory automatically.*

**Khi nào dùng**  
*When to use it*

Bạn có thể đã dùng hệ sinh thái TensorFlow.  
*You may already use the TensorFlow ecosystem.*

Khi đó, cách này phù hợp.  
*In that case, this approach is suitable.*

Cách này cũng phù hợp khi bảo trì hệ thống cũ.  
*This approach also suits legacy-system maintenance.*

#### (c) Python: con đường chuẩn của ngành
*(c) Python: the industry-standard path*

Trong công việc thật, phần lớn pipeline embedding dùng Python.  
*In real work, most embedding pipelines use Python.*

Bạn có thể phỏng vấn cho vị trí backend hoặc ML.  
*You may interview for a backend or ML position.*

Interviewer thường mong bạn viết bằng Python.  
*The interviewer will often expect Python.*

```python
# ── Cách 1: chạy local, miễn phí ─────────────────────────────────
# ── Method 1: run locally for free ──────────────────────────────
# pip install sentence-transformers
# pip install sentence-transformers
from sentence_transformers import SentenceTransformer

# Nạp model một lần.
# Load the model once.
# Dùng lại model cho các lần sau.
# Reuse the model for later calls.
# Cách này giống lưu ý trong ví dụ JavaScript.
# This approach matches the JavaScript example note.
model = SentenceTransformer("all-MiniLM-L6-v2")

# encode() tự thực hiện tokenize.
# encode() performs tokenization automatically.
# encode() tự chạy model.
# encode() runs the model automatically.
# encode() tự thực hiện pooling.
# encode() performs pooling automatically.
# normalize_embeddings=True tương đương normalize:true bên JavaScript.
# normalize_embeddings=True is equivalent to normalize:true in JavaScript.
vec = model.encode("áo thun cotton nam", normalize_embeddings=True)
print(vec.shape)        # (384,) -> một vector 384 chiều
                        # (384,) -> one 384-dimensional vector

# XỬ LÝ THEO LÔ: đưa vào cả DANH SÁCH câu.
# BATCH PROCESSING: pass an entire LIST of sentences.
# Không gọi riêng từng câu.
# Do not call the model separately for each sentence.
# Đây là tối ưu quan trọng nhất.
# This is the most important optimization.
# Cách này tận dụng khả năng song song của GPU hoặc CPU.
# This approach uses GPU or CPU parallelism.
# Vì vậy, tốc độ tăng lên nhiều lần.
# Therefore, the speed increases many times.
texts = ["áo thun cotton nam", "áo phông nam", "xe máy Honda"]
vecs = model.encode(
    texts,
    normalize_embeddings=True,
    batch_size=32,           # xử lý 32 câu mỗi lượt
                             # process 32 sentences per batch
    show_progress_bar=True   # hiện thanh tiến trình
                             # show a progress bar
                             # Tính năng này hữu ích với hàng triệu dòng.
                             # This feature is useful for millions of rows.
)
print(vecs.shape)       # (3, 384) -> 3 câu, mỗi câu 384 chiều
                        # (3, 384) -> 3 sentences, 384 dimensions per sentence


# ── Cách 2: gọi API OpenAI ───────────────────────────────────────
# ── Method 2: call the OpenAI API ───────────────────────────────
# pip install openai
# pip install openai
from openai import OpenAI

client = OpenAI()   # tự đọc khóa từ biến môi trường OPENAI_API_KEY
                    # automatically read the key from OPENAI_API_KEY

resp = client.embeddings.create(
    model="text-embedding-3-small",       # ⚠️ GHIM TÊN MODEL RÕ RÀNG
                                           # ⚠️ PIN THE MODEL NAME EXPLICITLY
                                           # Mục 4.5 giải thích lý do.
                                           # Section 4.5 explains the reason.
    input=["áo thun cotton nam", "áo phông nam"]   # gửi cả lô trong một lần gọi mạng
                                                      # send the whole batch in one network call
)

# Kết quả trả về là một danh sách.
# The result is a list.
# Thứ tự kết quả khớp với thứ tự input.
# The result order matches the input order.
vec1 = resp.data[0].embedding    # list gồm 1536 số thực
                                 # a list of 1,536 floating-point numbers
vec2 = resp.data[1].embedding
print(len(vec1))                 # 1536
                                 # 1536
print(resp.usage.total_tokens)   # số token đã dùng
                                 # number of tokens used
                                 # Đây là lượng bị tính tiền.
                                 # This is the billable amount.
```

> **Biến môi trường — environment variable**  
> *Environment variable — external configuration value*
>
> Biến môi trường lưu thông tin bí mật bên ngoài code.  
> *An environment variable stores secrets outside the code.*
>
> API key là một ví dụ.  
> *An API key is one example.*
>
> Bạn đặt giá trị trong cấu hình hệ thống.  
> *You place the value in system configuration.*
>
> Chương trình đọc giá trị lúc chạy.  
> *The program reads the value at runtime.*
>
> Bạn không được viết API key trực tiếp trong code.  
> *You must not write an API key directly in code.*
>
> Bạn cũng không được đẩy key lên GitHub.  
> *You must not push the key to GitHub.*
>
> Một số bot chuyên quét GitHub.  
> *Some bots continuously scan GitHub.*
>
> Chúng tìm các key bị lộ.  
> *They look for exposed keys.*
>
> Chúng có thể dùng hết tiền trong vài phút.  
> *They can spend all your funds within minutes.*
>
> **Batch — lô**  
> *Batch — a group of inputs*
>
> Batch gom nhiều đầu vào vào cùng một lượt xử lý.  
> *A batch groups many inputs into one processing run.*
>
> Cách này thay thế việc xử lý từng đầu vào riêng lẻ.  
> *This approach replaces separate processing for each input.*
>
> Đây là kỹ thuật tối ưu quan trọng nhất cho dữ liệu lớn.  
> *This is the most important optimization for large datasets.*
>
> Mục 4.1 sẽ giải thích kỹ hơn.  
> *Section 4.1 will explain it in more detail.*

Video gốc nhắc đến OpenAI và Google.  
*The original video mentions OpenAI and Google.*

Bạn cần lấy API key từ website của họ.  
*You need to obtain an API key from their websites.*

Sau đó, bạn kết nối bằng key đó.  
*You then connect with that key.*

Thông tin này là chính xác.  
*This information is correct.*

Cách dùng API tạo ra nhiều hệ quả.  
*API usage creates many consequences.*

Các hệ quả liên quan đến chi phí.  
*Some consequences concern cost.*

Các hệ quả khác liên quan đến quyền riêng tư.  
*Other consequences concern privacy.*

Độ tin cậy cũng là một vấn đề.  
*Reliability is also a concern.*

Phần 4 sẽ phân tích kỹ các vấn đề này.  
*Part 4 will analyze these issues in detail.*

### 2.5. 🧩 [Ngoài bài gốc] Ba lỗi kinh điển khi làm việc với embedding
*2.5. 🧩 [Beyond the original lesson] Three classic embedding mistakes*

Ba lỗi này chiếm phần lớn thời gian debug của người mới.  
*These three mistakes consume most beginner debugging time.*

Cả ba lỗi đều có cùng biểu hiện nguy hiểm.  
*All three mistakes share the same dangerous behavior.*

Chương trình vẫn chạy.  
*The program still runs.*

Chương trình không báo lỗi.  
*The program reports no error.*

Kết quả lại không chính xác.  
*The result is still incorrect.*

Đây là loại bug khó chịu nhất.  
*This is the most frustrating type of bug.*

> **Debug — gỡ lỗi**  
> *Debug — troubleshooting*
>
> Debug là quá trình truy tìm nguyên nhân chương trình chạy sai.  
> *Debugging is the process of finding why a program behaves incorrectly.*
>
> Loại bug dễ nhất làm chương trình sập ngay.  
> *The easiest bug type crashes the program immediately.*
>
> Lỗi đó chỉ thẳng vào vị trí hỏng.  
> *That failure points directly to the broken location.*
>
> Loại bug khó nhất vẫn cho chương trình chạy.  
> *The hardest bug type lets the program continue running.*
>
> Chương trình chỉ tạo ra kết quả sai.  
> *The program only produces an incorrect result.*
>
> Bạn không biết vị trí cần kiểm tra.  
> *You do not know where to investigate.*
>
> Cả ba lỗi sau thuộc loại khó này.  
> *All three following mistakes belong to this difficult type.*
>
> **Production — môi trường thật**  
> *Production — the live environment*
>
> Production là hệ thống phục vụ người dùng thật.  
> *Production is the system serving real users.*
>
> Môi trường này khác máy thử nghiệm của lập trình viên.  
> *This environment differs from a developer's test machine.*
>
> Lỗi trong production được khách hàng nhìn thấy.  
> *Customers see errors in production.*

#### Lỗi 1: Quên pooling và không hiểu shape lạ
*Error 1: Forgetting pooling and misunderstanding an unusual shape*

**Triệu chứng**  
*Symptom*

Bạn embed hai câu.  
*You embed two sentences.*

Bạn in kích thước của kết quả.  
*You print the result dimensions.*

Kết quả đầu tiên là `[1, 7, 384]`.  
*The first result is `[1, 7, 384]`.*

Kết quả thứ hai là `[1, 12, 384]`.  
*The second result is `[1, 12, 384]`.*

Bạn mong đợi shape `[1, 384]`.  
*You expected the shape `[1, 384]`.*

Sau đó, bạn tính cosine.  
*You then calculate cosine similarity.*

Chương trình có thể báo lỗi.  
*The program may report an error.*

Chương trình cũng có thể trả một số vô nghĩa.  
*The program may also return a meaningless number.*

**Nguyên nhân**  
*Cause*

Mục 2.2 đã giải thích nguyên nhân ở bước 2.  
*Section 2.2 explained the cause in step 2.*

Transformer trả về một vector cho mỗi token.  
*A transformer returns one vector for each token.*

Câu đầu tiên có 7 token.  
*The first sentence has seven tokens.*

Vì vậy, câu đầu tiên tạo ra 7 vector.  
*Therefore, the first sentence produces seven vectors.*

Câu thứ hai có 12 token.  
*The second sentence has twelve tokens.*

Vì vậy, câu thứ hai tạo ra 12 vector.  
*Therefore, the second sentence produces twelve vectors.*

Hai câu có độ dài khác nhau.  
*The two sentences have different lengths.*

Số vector của chúng cũng khác nhau.  
*Their vector counts are also different.*

Bạn không thể so sánh trực tiếp hai kết quả này.  
*You cannot directly compare these two results.*

**Cách sửa**  
*The fix*

```javascript
// SAI ❌
// WRONG ❌
const a = await extractor('con mèo đang ngủ');
console.log(a.dims);   // [1, 5, 384] -> 5 vector, không phải 1!
                       // [1, 5, 384] -> 5 vectors, not 1!

// ĐÚNG ✅
// CORRECT ✅
const a = await extractor('con mèo đang ngủ', { pooling: 'mean', normalize: true });
console.log(a.dims);   // [1, 384] -> đúng 1 vector cho cả câu
                       // [1, 384] -> exactly 1 vector for the whole sentence
```

```python
# Python: sentence-transformers tự pooling giúp bạn, nhưng vẫn nên normalize
# Python: sentence-transformers performs pooling for you, but normalization is still recommended
vec = model.encode(text, normalize_embeddings=True)   # ✅
```

💡 **Mẹo phòng bệnh**  
*💡 Preventive tip*

Bạn hãy in kích thước ngay sau lần tạo embedding đầu tiên.  
*Print the shape immediately after the first embedding operation.*

Kích thước phải bằng đúng số chiều bạn kỳ vọng.  
*The shape must match your expected dimension.*

Ba giây kiểm tra có thể tiết kiệm ba giờ debug.  
*Three seconds of checking can save three hours of debugging.*

#### Lỗi 2 — Trộn embedding từ hai model khác nhau
*Error 2: mixing embeddings from different models*

**Triệu chứng**  
*Symptom*

Kết quả tìm kiếm chứa các mục hoàn toàn ngẫu nhiên.  
*The search results contain completely random items.*

Database trông giống như đã bị hỏng.  
*The database appears to be broken.*

**Nguyên nhân**  
*Cause*

Database chứa vector do model A tạo ra.  
*The database contains vectors from model A.*

Database cũng chứa vector do model B tạo ra.  
*The database also contains vectors from model B.*

Hãy nhớ lại analogy bản đồ ở Mục 1.2.  
*Recall the map analogy from Section 1.2.*

Mỗi model vẽ một tấm bản đồ riêng.  
*Each model draws its own map.*

Mỗi tấm bản đồ dùng một hệ quy chiếu riêng.  
*Each map uses its own coordinate system.*

Toạ độ `(2, 1)` có một nghĩa trên bản đồ A.  
*The coordinate `(2, 1)` has one meaning on map A.*

Toạ độ `(2, 1)` có nghĩa khác trên bản đồ B.  
*The coordinate `(2, 1)` has another meaning on map B.*

Hai toạ độ đó chỉ vào hai vị trí hoàn toàn khác nhau.  
*Those coordinates point to completely different locations.*

Việc so sánh chúng hoàn toàn vô nghĩa.  
*Comparing them is completely meaningless.*

Bạn có thể so sánh tình huống này với hai địa chỉ.  
*You can compare this situation with two addresses.*

Địa chỉ đầu tiên là số nhà 15 tại Hà Nội.  
*The first address is house number 15 in Hanoi.*

Địa chỉ thứ hai là số nhà 15 tại TP.HCM.  
*The second address is house number 15 in Ho Chi Minh City.*

Hai địa chỉ dùng cùng một con số.  
*The two addresses use the same number.*

Hai địa chỉ không liên quan đến nhau.  
*The two addresses are unrelated.*

Trường hợp dễ phát hiện có số chiều khác nhau.  
*The easy case has different dimensions.*

Ví dụ là 384 chiều và 1536 chiều.  
*Examples are 384 dimensions and 1536 dimensions.*

Code sẽ báo lỗi ngay trong trường hợp này.  
*The code immediately reports an error in this case.*

Trường hợp nguy hiểm có hai model cùng số chiều.  
*The dangerous case has two models with equal dimensions.*

Code vẫn chạy trơn tru.  
*The code still runs smoothly.*

Code không báo bất kỳ lỗi nào.  
*The code reports no errors.*

Kết quả cuối cùng chỉ là dữ liệu rác.  
*The final result is only garbage data.*

Hệ thống không thể tự động phát hiện lỗi này.  
*The system cannot automatically detect this error.*

Bạn phải duy trì kỷ luật dữ liệu nghiêm ngặt.  
*You must maintain strict data discipline.*

**Cách phòng**  
*Prevention*

```python
# Lưu tên model + phiên bản NGAY BÊN CẠNH vector trong database
# Store the model name and version DIRECTLY BESIDE the vector in the database
{
    "id": 12345,
    "text": "áo thun cotton nam",
    "embedding": [0.21, -0.87, ...],
    "embedding_model": "all-MiniLM-L6-v2",    # ← dòng này cứu bạn sau này
                                                     # ← this line will save you later
    "embedding_dim": 384
}
```

Quy tắc sắt áp dụng cho toàn bộ một index.  
*An iron rule applies to an entire index.*

Mọi dữ liệu phải dùng chung một model.  
*All data must use the same model.*

Mọi dữ liệu phải dùng chung một phiên bản.  
*All data must use the same version.*

Quy tắc này không có ngoại lệ.  
*This rule has no exceptions.*

Hệ quả tiếp theo rất nặng ký.  
*The next consequence is significant.*

Bạn đổi model.  
*You change the model.*

Khi đó, bạn phải tạo lại toàn bộ embedding.  
*You must then regenerate every embedding.*

Mười triệu tài liệu tạo thành một khối lượng rất lớn.  
*Ten million documents create a massive workload.*

Công việc này không phải một dòng cấu hình.  
*This task is not a configuration line.*

Công việc này là một dự án nhiều ngày.  
*This task is a multi-day project.*

Dự án này tiêu tốn tiền thật.  
*This project costs real money.*

Mục 4.3 trình bày cách chuyển đổi an toàn.  
*Section 4.3 presents a safe migration method.*

Cách đó không làm sập hệ thống.  
*That method does not bring down the system.*

#### Lỗi 3 — Dùng dot product mà quên chuẩn hoá
*Error 3: using dot product without normalization*

**Triệu chứng**  
*Symptom*

Kết quả tìm kiếm luôn ưu tiên các tài liệu dài.  
*Search results always favor long documents.*

Nội dung của chúng có thể không liên quan.  
*Their content may be irrelevant.*

**Nguyên nhân**  
*Cause*

Mục 2.2 đã nêu một điều kiện quan trọng.  
*Section 2.2 stated an important condition.*

Vector phải được chuẩn hoá trước.  
*Vectors must be normalized first.*

Sau đó, cosine bằng dot product.  
*Cosine then equals dot product.*

Nhiều người chỉ nhớ lợi ích tốc độ.  
*Many people remember only the speed benefit.*

Họ dùng dot product để tìm nhanh hơn.  
*They use dot product for faster search.*

Họ quên bước chuẩn hoá bắt buộc.  
*They forget the required normalization step.*

Dot product chưa chuẩn hoá chịu ảnh hưởng của độ dài vector.  
*An unnormalized dot product depends on vector magnitude.*

Văn bản dài thường tạo vector có độ dài lớn hơn.  
*Long text often creates vectors with greater magnitude.*

Các tài liệu dài dòng sẽ được xếp hạng cao hơn.  
*Verbose documents will receive higher rankings.*

Chúng có thể kém liên quan hơn nhiều.  
*They may be far less relevant.*

Cosine được dùng để tránh chính vấn đề này.  
*Cosine is used to avoid this exact problem.*

**Cách sửa**  
*The fix*

Bạn phải chọn một trong hai cách.  
*You must choose one of two approaches.*

Bạn phải giữ cách đó nhất quán trong toàn hệ thống.  
*You must keep that approach consistent across the system.*

- Chuẩn hoá tất cả vector trước khi lưu.  
  *Normalize all vectors before storage.*

- Sau đó, dùng dot product để đạt tốc độ cao nhất.  
  *Then use dot product for maximum speed.*

- Cách khác là không chuẩn hoá vector.  
  *The alternative is leaving vectors unnormalized.*

- Khi đó, luôn dùng công thức cosine đầy đủ.  
  *Always use the full cosine formula in that case.*

Bạn tuyệt đối không được trộn lẫn hai cách.  
*You must never mix the two approaches.*

### 2.6. Chọn phép đo nào cho việc gì
*2.6. Choosing a metric for each task*

Cosine không phải phép đo duy nhất.  
*Cosine is not the only metric.*

Bạn sẽ thường gặp ba phép đo sau.  
*You will often encounter the following three metrics.*

| Phép đo / Metric | Công thức / Formula | Đo cái gì / What it measures | Dùng khi / Use case | Toán tử pgvector / pgvector operator |
|---|---|---|---|---|
| **Cosine similarity** | `(A·B)/(\|A\|\|B\|)` | Góc giữa hai vector / The angle between two vectors | **Mặc định cho văn bản** / **Default for text** | `<=>` |
| **Dot / inner product** | `Σ aᵢbᵢ` | Góc và độ dài / Angle plus magnitude | Vector đã chuẩn hoá / Normalized vectors | `<#>` |
| **Euclidean (L2)** | `sqrt(Σ(aᵢ−bᵢ)²)` | Khoảng cách đường chim bay / Straight-line distance | Độ lớn vector có ý nghĩa / Vector magnitude carries meaning | `<->` |

> **Euclidean distance — khoảng cách Euclid**  
> *Euclidean distance*
>
> Euclidean distance còn được gọi là khoảng cách L2.  
> *Euclidean distance is also called L2 distance.*
>
> Nó đo khoảng cách đường chim bay giữa hai điểm.  
> *It measures the straight-line distance between two points.*
>
> Đây là định lý Pythagoras mở rộng.  
> *It is an extension of the Pythagorean theorem.*
>
> Điểm đầu tiên có toạ độ `(0,0)`.  
> *The first point has coordinates `(0,0)`.*
>
> Điểm thứ hai có toạ độ `(3,4)`.  
> *The second point has coordinates `(3,4)`.*
>
> Khoảng cách bằng `sqrt(3² + 4²) = 5`.  
> *The distance equals `sqrt(3² + 4²) = 5`.*

Một điểm dễ gây nhầm cần được ghi nhớ.  
*One confusing point deserves attention.*

Similarity và distance chạy theo hai chiều ngược nhau.  
*Similarity and distance move in opposite directions.*

- **Similarity — độ tương đồng** càng cao thì càng giống.  
  *Higher similarity means greater likeness.*

- Cosine similarity bằng 1 biểu thị mức giống cao nhất.  
  *A cosine similarity of 1 indicates maximum likeness.*

- **Distance — khoảng cách** càng thấp thì càng giống.  
  *Lower distance means greater likeness.*

- Khoảng cách bằng 0 biểu thị hai điểm trùng nhau.  
  *A distance of 0 indicates identical points.*

pgvector làm việc với distance.  
*pgvector works with distance.*

Toán tử `<=>` trả về cosine distance.  
*The `<=>` operator returns cosine distance.*

Cosine distance được tính bằng công thức sau.  
*Cosine distance uses the following formula.*

```
cosine_distance = 1 − cosine_similarity        (miền giá trị: từ 0 đến 2)
cosine_distance = 1 − cosine_similarity        (range: 0 to 2)
```

Bạn có thể viết truy vấn sau.  
*You can write the following query.*

`ORDER BY embedding <=> query_vector`  
*`ORDER BY embedding <=> query_vector`*

Truy vấn sắp xếp kết quả theo thứ tự tăng dần.  
*The query sorts results in ascending order.*

Các vector giống nhất xuất hiện trước.  
*The most similar vectors appear first.*

Mức giống cao nhất tương ứng khoảng cách nhỏ nhất.  
*Maximum similarity corresponds to minimum distance.*

Nhầm chiều sẽ đảo ngược toàn bộ kết quả tìm kiếm.  
*Reversing the direction will invert all search results.*

Đây là một lỗi rất phổ biến.  
*This is a very common mistake.*

### ✅ Self-check Phần 2
*✅ Part 2 self-check*

**Câu 1**  
*Question 1*

Từ `bank` xuất hiện trong cụm `river bank`.  
*The word `bank` appears in `river bank`.*

Từ đó cũng xuất hiện trong cụm `bank account`.  
*The same word appears in `bank account`.*

Static embedding tạo ra bao nhiêu vector?  
*How many vectors does static embedding produce?*

Contextual embedding tạo ra bao nhiêu vector?  
*How many vectors does contextual embedding produce?*

Lý do là gì?  
*What is the reason?*

<details>
<summary>Gợi ý đáp án / Answer hint</summary>

Static embedding tạo ra một vector duy nhất.  
*Static embedding produces only one vector.*

Nó chỉ có một hàng cho mỗi từ.  
*It has only one row for each word.*

Hai nghĩa bị trộn vào cùng một vector.  
*Both meanings become mixed into one vector.*

Vector đó không đại diện chính xác cho nghĩa nào.  
*That vector accurately represents neither meaning.*

Contextual embedding tạo ra hai vector khác nhau.  
*Contextual embedding produces two different vectors.*

Self-attention xem các từ xung quanh `bank`.  
*Self-attention examines the words around `bank`.*

Một ngữ cảnh chứa từ `river`.  
*One context contains the word `river`.*

Ngữ cảnh khác chứa từ `account`.  
*The other context contains the word `account`.*

Model điều chỉnh vector theo từng ngữ cảnh.  
*The model adjusts the vector for each context.*

</details>

**Câu 2**  
*Question 2*

Tại sao embedding của câu bắt buộc phải dùng pooling?  
*Why must sentence embeddings use pooling?*

<details>
<summary>Gợi ý đáp án / Answer hint</summary>

Transformer trả về một vector cho mỗi token.  
*A transformer returns one vector for each token.*

Nó không trả về một vector cho cả câu.  
*It does not return one vector for the whole sentence.*

Kết quả có shape `[1, số_token, 384]`.  
*The result has shape `[1, token_count, 384]`.*

Hai câu có thể chứa số token khác nhau.  
*Two sentences may contain different token counts.*

Hai kết quả đó không thể được so sánh trực tiếp.  
*Those results cannot be compared directly.*

Pooling gộp các vector token.  
*Pooling combines the token vectors.*

Mean pooling thường lấy giá trị trung bình.  
*Mean pooling usually takes the average value.*

Kết quả là một vector đại diện cho cả câu.  
*The result is one vector for the whole sentence.*

</details>

**Câu 3**  
*Question 3*

Vector đã được chuẩn hoá.  
*The vectors are normalized.*

Tại sao cosine similarity rút gọn thành dot product?  
*Why does cosine similarity reduce to dot product?*

Việc đó mang lại lợi ích gì?  
*What benefit does this provide?*

<details>
<summary>Gợi ý đáp án / Answer hint</summary>

Chuẩn hoá làm `|A| = |B| = 1`.  
*Normalization makes `|A| = |B| = 1`.*

Mẫu số `|A| × |B|` bằng 1.  
*The denominator `|A| × |B|` equals 1.*

Công thức chỉ còn tử số.  
*The formula retains only the numerator.*

Tử số chính là dot product.  
*The numerator is the dot product.*

Phép tính loại bỏ hai căn bậc hai.  
*The calculation removes two square roots.*

Phép tính cũng loại bỏ một phép chia.  
*The calculation also removes one division.*

Một lần tiết kiệm không tạo khác biệt lớn.  
*One saved operation makes little difference.*

Hàng triệu lần tạo ra khác biệt rất lớn.  
*Millions of repetitions create a major difference.*

</details>

**Câu 4**  
*Question 4*

Bạn đã embed 1 triệu tài liệu bằng model A.  
*You embedded one million documents with model A.*

Bây giờ bạn muốn dùng model B tốt hơn.  
*You now want to use a better model B.*

Bạn phải làm gì với dữ liệu cũ?  
*What must you do with the old data?*

<details>
<summary>Gợi ý đáp án / Answer hint</summary>

Bạn phải tạo lại toàn bộ embedding.  
*You must regenerate every embedding.*

Mọi embedding mới phải dùng model B.  
*Every new embedding must use model B.*

Vector cũ nằm trên bản đồ của model A.  
*Old vectors lie on the map of model A.*

Vector mới nằm trên bản đồ của model B.  
*New vectors lie on the map of model B.*

Hai nhóm vector không thể được so sánh.  
*The two vector groups cannot be compared.*

Số chiều giống nhau không thay đổi điều đó.  
*Equal dimensions do not change that fact.*

Đây là một dự án tốn thời gian.  
*This is a time-consuming project.*

Dự án này cũng tiêu tốn tiền bạc.  
*This project also costs money.*

Mục 4.3 trình bày phương pháp chuyển đổi an toàn.  
*Section 4.3 presents a safe migration method.*

</details>

---

## Phần 3 — 🔴 ADVANCED (Chuyên sâu)
*Part 3: 🔴 ADVANCED*

> Phần này nhìn xuống bên dưới bề mặt.  
> *This part looks beneath the surface.*
>
> Phần này trình bày toán học của cosine.  
> *This part presents the mathematics of cosine.*
>
> Phần này trình bày độ phức tạp tính toán.  
> *This part presents computational complexity.*
>
> Phần này phân tích sự đánh đổi về số chiều.  
> *This part analyzes dimensional trade-offs.*
>
> Phần này cũng trình bày các trường hợp biên.  
> *This part also presents edge cases.*
>
> Các trường hợp đó có thể làm hỏng hệ thống lúc 3 giờ sáng.  
> *Those cases can break your system at 3 a.m.*
>
> Trước tiên, ta cần hiểu hai thuật ngữ.  
> *First, we need to understand two terms.*
>
> Hai thuật ngữ này sẽ xuất hiện liên tục.  
> *These two terms will appear repeatedly.*
>
> **Trade-off — sự đánh đổi**  
> *Trade-off*
>
> Trade-off là một tình huống bắt buộc phải hy sinh.  
> *A trade-off is a situation requiring sacrifice.*
>
> Bạn cải thiện một mặt.  
> *You improve one aspect.*
>
> Khi đó, một mặt khác trở nên kém hơn.  
> *Another aspect then becomes worse.*
>
> Không có lựa chọn tốt nhất ở mọi mặt.  
> *No choice is best in every aspect.*
>
> Ví dụ là việc tăng số chiều embedding.  
> *An example is increasing embedding dimensions.*
>
> Số chiều cao hơn có thể cải thiện chất lượng tìm kiếm.  
> *Higher dimensions can improve search quality.*
>
> Số chiều cao hơn cũng tốn nhiều RAM hơn.  
> *Higher dimensions also consume more RAM.*
>
> Việc tìm kiếm cũng trở nên chậm hơn.  
> *Search also becomes slower.*
>
> Bạn không thể đồng thời đạt mọi lợi ích.  
> *You cannot achieve every benefit simultaneously.*
>
> Staff engineer phải nhận ra các trade-off.  
> *A staff engineer must recognize trade-offs.*
>
> Staff engineer cũng phải trình bày lựa chọn rõ ràng.  
> *A staff engineer must also explain choices clearly.*
>
> Họ phải nêu rõ phần được hy sinh.  
> *They must identify the sacrificed aspect.*
>
> Họ cũng phải nêu rõ phần nhận lại.  
> *They must also identify the gained aspect.*
>
> Họ không giả vờ có giải pháp hoàn hảo.  
> *They do not pretend to have a perfect solution.*
>
> **Edge case — trường hợp biên**  
> *Edge case*
>
> Edge case là một tình huống hiếm gặp.  
> *An edge case is a rare situation.*
>
> Nó nằm ở rìa của hành vi bình thường.  
> *It lies at the edge of normal behavior.*
>
> Lập trình viên thường bỏ quên nó.  
> *Developers often overlook it.*
>
> Nó thường làm sập hệ thống trong production.  
> *It often crashes systems in production.*
>
> Một ví dụ là chuỗi đầu vào rỗng.  
> *An example is an empty input string.*

### 3.1. Toán học của cosine — vì sao nó đo được hướng
*3.1. Cosine mathematics: why it measures direction*

Phần 1 đã giới thiệu công thức cosine.  
*Part 1 introduced the cosine formula.*

Khi đó, bạn được yêu cầu tin vào công thức.  
*You were asked to trust the formula then.*

Bây giờ, ta sẽ chứng minh công thức đó.  
*We will now prove that formula.*

Điểm khởi đầu là định nghĩa hình học của dot product.  
*The starting point is the geometric definition of dot product.*

```
A · B = |A| × |B| × cos(θ)
```

Ký hiệu `θ` được đọc là theta.  
*The symbol `θ` is pronounced theta.*

Theta là một chữ cái Hy Lạp.  
*Theta is a Greek letter.*

Ký hiệu này thường đại diện cho một góc.  
*This symbol commonly represents an angle.*

Ở đây, theta là góc giữa hai vector.  
*Here, theta is the angle between two vectors.*

Công thức mô tả dot product của hai vector.  
*The formula describes the dot product of two vectors.*

Dot product bằng tích của hai độ dài.  
*The dot product equals the product of two magnitudes.*

Kết quả đó được nhân với cosin của góc.  
*That result is multiplied by the cosine of the angle.*

Đây là một định lý hình học vector cơ bản.  
*This is a fundamental theorem of vector geometry.*

Định lý đúng trong mọi số chiều.  
*The theorem holds in every dimension.*

Nó vẫn đúng trong không gian 384 chiều.  
*It still holds in 384-dimensional space.*

Con người không thể hình dung không gian đó.  
*Humans cannot visualize that space.*

Khả năng hình dung không ảnh hưởng đến tính đúng đắn.  
*Visualization ability does not affect correctness.*

Bây giờ, ta thực hiện một phép biến đổi đại số.  
*We now perform an algebraic transformation.*

Ta chia hai vế cho `|A| × |B|`.  
*We divide both sides by `|A| × |B|`.*

```
   A · B
──────────── = cos(θ)
 |A| × |B|
```

Vế trái là công thức cosine similarity.  
*The left side is the cosine similarity formula.*

Bạn đã dùng công thức này ở Mục 1.7.  
*You used this formula in Section 1.7.*

Vế phải là cosin của góc.  
*The right side is the cosine of the angle.*

💡 **Đây là lời giải thích sâu nhất cho tính chất bỏ qua độ dài.**  
*💡 This is the deepest explanation of magnitude independence.*

Phép chia dùng mẫu số `|A| × |B|`.  
*The division uses the denominator `|A| × |B|`.*

Mẫu số triệt tiêu chính xác hai thừa số độ dài.  
*The denominator exactly cancels both magnitude factors.*

Hai thừa số đó nằm trong định nghĩa dot product.  
*Those factors appear in the dot product definition.*

Sau phép triệt tiêu, chỉ còn `cos(θ)`.  
*Only `cos(θ)` remains after cancellation.*

Đại lượng này chỉ phụ thuộc vào góc.  
*This quantity depends only on the angle.*

Không ai thiết kế một mẹo bỏ qua độ dài.  
*No one designed a trick to ignore magnitude.*

Độ dài biến mất tự nhiên qua phép chia.  
*Magnitude disappears naturally through division.*

Ta có thể suy ra ngay miền giá trị.  
*We can immediately derive the value range.*

Cosin của mọi góc nằm trong `[−1, 1]`.  
*The cosine of every angle lies within `[−1, 1]`.*

Cosine similarity cũng nằm trong `[−1, 1]`.  
*Cosine similarity also lies within `[−1, 1]`.*

Đây là chứng minh một dòng cho Mục 1.6.  
*This is a one-line proof for Section 1.6.*

### 3.2. Vì sao chuẩn hoá làm cosine và Euclidean tương đương
*3.2. Why normalization makes cosine and Euclidean equivalent*

Đây là một kết quả rất hữu ích.  
*This is a very useful result.*

Nó cũng thường xuất hiện trong phỏng vấn.  
*It also appears frequently in interviews.*

Nhiều hệ thống dùng L2.  
*Many systems use L2.*

Nhiều hệ thống khác dùng dot product.  
*Many other systems use dot product.*

Kết quả top-k của chúng vẫn có thể giống nhau.  
*Their top-k results can still be identical.*

Kết quả này giải thích nguyên nhân.  
*This result explains the reason.*

Giả sử A đã được chuẩn hoá.  
*Assume A is normalized.*

Giả sử B cũng đã được chuẩn hoá.  
*Assume B is also normalized.*

Khi đó, `|A| = |B| = 1`.  
*Then `|A| = |B| = 1`.*

Ta tính bình phương khoảng cách Euclid.  
*We calculate the squared Euclidean distance.*

```
|A − B|²  =  |A|² + |B|² − 2(A · B)          ← khai triển đại số cơ bản
                                                   ← basic algebraic expansion
          =    1  +   1  − 2·cos(θ)           ← thay |A|=|B|=1 và A·B = cos(θ)
                                                   ← substitute |A|=|B|=1 and A·B = cos(θ)
          =    2 − 2·cos(θ)
```

Kết quả cuối cùng là `2 − 2·cos(θ)`.  
*The final result is `2 − 2·cos(θ)`.*

Khoảng cách Euclid giảm nghiêm ngặt theo cosine similarity.  
*Euclidean distance strictly decreases with cosine similarity.*

Cosine càng lớn thì khoảng cách càng nhỏ.  
*Greater cosine means smaller distance.*

Quy luật này không có ngoại lệ.  
*This rule has no exceptions.*

> **Monotonically equivalent — đơn điệu tương đương**  
> *Monotonically equivalent*
>
> Hai phép đo có thể thay đổi cùng chiều.  
> *Two metrics can change in the same direction.*
>
> Chúng cũng có thể thay đổi ngược chiều.  
> *They can also change in opposite directions.*
>
> Quan hệ đó phải luôn nhất quán.  
> *That relationship must always remain consistent.*
>
> Hệ quả thực dụng nằm ở thứ tự xếp hạng.  
> *The practical consequence concerns ranking order.*
>
> Hai phép đo tạo ra cùng một thứ tự.  
> *The two metrics produce the same order.*

Vector đã chuẩn hoá tạo ra một kết quả quan trọng.  
*Normalized vectors produce an important result.*

Bạn tìm 10 vector có cosine similarity cao nhất.  
*You find the ten vectors with highest cosine similarity.*

Bạn cũng tìm 10 vector có Euclidean distance thấp nhất.  
*You also find the ten vectors with lowest Euclidean distance.*

Hai cách cho cùng một danh sách.  
*Both approaches produce the same list.*

Hai cách cũng cho cùng một thứ tự.  
*Both approaches also produce the same order.*

Chỉ các con số điểm khác nhau.  
*Only the score values differ.*

💡 **Ứng dụng thực tế**  
*💡 Practical application*

Bạn có thể chuẩn hoá toàn bộ vector.  
*You can normalize every vector.*

Sau đó, bạn có thể dùng dot product.  
*You can then use dot product.*

Cách này mang lại tốc độ của dot product.  
*This approach provides dot product speed.*

Chất lượng xếp hạng không bị mất.  
*Ranking quality is not lost.*

Interviewer có thể hỏi về cosine và L2.  
*An interviewer may ask about cosine and L2.*

Một câu trả lời tốt cần nêu điều kiện chuẩn hoá.  
*A strong answer should mention the normalization condition.*

Vector đã chuẩn hoá làm hai phép đo tương đương về xếp hạng.  
*Normalized vectors make both metrics ranking-equivalent.*

Khi đó, bạn chọn theo hiệu năng.  
*You then choose based on performance.*

Bạn không cần chọn theo chất lượng.  
*You do not need to choose based on quality.*

### 3.3. Độ phức tạp tính toán — vì sao tìm kiếm chính xác không khả thi
*3.3. Computational complexity: why exact search is impractical*

> **Big-O**  
> *Big-O*
>
> Big-O mô tả tốc độ tăng của chi phí tính toán.  
> *Big-O describes the growth rate of computational cost.*
>
> Dữ liệu tăng lên theo một mức nhất định.  
> *Data increases by a given amount.*
>
> Big-O mô tả mức tăng tương ứng của chi phí.  
> *Big-O describes the corresponding cost increase.*
>
> Ký hiệu này bỏ qua các hằng số.  
> *This notation ignores constant factors.*
>
> `O(n)` biểu thị mức tăng tuyến tính.  
> *`O(n)` represents linear growth.*
>
> Dữ liệu tăng gấp đôi.  
> *The data doubles.*
>
> Thời gian cũng tăng gấp đôi.  
> *The time also doubles.*
>
> `O(n²)` biểu thị mức tăng bình phương.  
> *`O(n²)` represents quadratic growth.*
>
> Dữ liệu tăng gấp đôi.  
> *The data doubles.*
>
> Thời gian tăng gấp bốn lần.  
> *The time increases fourfold.*
>
> `O(1)` biểu thị thời gian không đổi.  
> *`O(1)` represents constant time.*
>
> Kích thước dữ liệu không làm thay đổi thời gian.  
> *Data size does not change the time.*

Các thao tác trong bài có chi phí sau.  
*The operations in this lesson have the following costs.*

| Thao tác / Operation | Big-O | Nghĩa là / Meaning |
|---|---|---|
| Tính cosine giữa hai vector d chiều / Compute cosine between two d-dimensional vectors | `O(d)` | Phải đọc tất cả d số một lần / Every one of the d values must be read once |
| Tìm gần nhất bằng vét cạn trong n vector / Brute-force nearest search across n vectors | `O(n × d)` | Tính cosine với từng vector / Compute cosine against every vector |
| Tìm gần nhất bằng ANN index / Nearest search with an ANN index | `≈ O(log n × d)` | Chỉ đọc một phần rất nhỏ / Read only a very small subset |

Các con số cụ thể cho thấy khác biệt rõ ràng.  
*Concrete numbers reveal the difference clearly.*

Giả sử `n = 10 triệu` vector.  
*Assume `n = 10 million` vectors.*

Giả sử `d = 384` chiều.  
*Assume `d = 384` dimensions.*

```
Vét cạn:      10.000.000 × 384  ≈  3.8 tỉ phép tính mỗi lượt tìm kiếm
Brute force:  10,000,000 × 384  ≈  3.8 billion operations per search

ANN (HNSW):   log₂(10.000.000) ≈ 23, và 23 × 384 × (hệ số ~100)
ANN (HNSW):   log₂(10,000,000) ≈ 23, then 23 × 384 × (factor ~100)

              ≈ vài trăm nghìn phép tính
              ≈ a few hundred thousand operations
```

Khác biệt đạt khoảng 10.000 lần.  
*The difference reaches about 10,000 times.*

Đây không phải một tối ưu nhỏ.  
*This is not a minor optimization.*

Đây là ranh giới giữa khả thi và bất khả thi.  
*This is the boundary between feasible and infeasible.*

> **HNSW — Hierarchical Navigable Small World**  
> *HNSW: Hierarchical Navigable Small World*
>
> HNSW là thuật toán ANN phổ biến nhất hiện nay.  
> *HNSW is the most popular ANN algorithm today.*
>
> pgvector sử dụng thuật toán này.  
> *pgvector uses this algorithm.*
>
> Hầu hết vector database cũng sử dụng nó.  
> *Most vector databases also use it.*
>
> Thuật toán xây một mạng lưới nhiều tầng.  
> *The algorithm builds a multilayer network.*
>
> Mạng lưới kết nối các vector với nhau.  
> *The network connects vectors together.*
>
> Các tầng trên có mật độ thưa.  
> *Upper layers are sparse.*
>
> Chúng hỗ trợ các bước nhảy xa.  
> *They support long jumps.*
>
> Các tầng dưới có mật độ dày.  
> *Lower layers are dense.*
>
> Chúng hỗ trợ tìm kiếm chính xác hơn.  
> *They support more precise search.*
>
> Quá trình giống một hành trình nhiều chặng.  
> *The process resembles a multistage journey.*
>
> Bạn bắt đầu tại sân bay quốc tế.  
> *You start at an international airport.*
>
> Sau đó, bạn đến sân bay nội địa.  
> *You then reach a domestic airport.*
>
> Tiếp theo, bạn đi xe buýt.  
> *Next, you take a bus.*
>
> Cuối cùng, bạn đi bộ.  
> *Finally, you walk.*
>
> Mỗi tầng đưa bạn đến gần mục tiêu hơn.  
> *Each layer brings you closer to the target.*

Tốc độ cao có một cái giá.  
*High speed has a cost.*

Cái giá nằm trong từ approximate.  
*The cost lies in the word approximate.*

Approximate có nghĩa là xấp xỉ.  
*Approximate means inexact.*

ANN có thể bỏ sót một số kết quả đúng.  
*ANN may miss some correct results.*

> **Recall — độ bao phủ**  
> *Recall*
>
> Recall đo tỉ lệ kết quả đúng được tìm thấy.  
> *Recall measures the proportion of correct results found.*
>
> Recall bằng 0.95 mang một ý nghĩa cụ thể.  
> *A recall of 0.95 has a specific meaning.*
>
> Hệ thống đáng lẽ phải tìm thấy 100 kết quả.  
> *The system should have found 100 results.*
>
> Hệ thống thực tế tìm thấy 95 kết quả.  
> *The system actually finds 95 results.*
>
> Hệ thống bỏ sót 5 kết quả.  
> *The system misses five results.*
>
> **Recall@k**  
> *Recall@k*
>
> Recall@k chỉ xét k kết quả đầu tiên.  
> *Recall@k considers only the first k results.*
>
> Recall@10 là chỉ số phổ biến nhất cho tìm kiếm.  
> *Recall@10 is the most common search metric.*
>
> Người dùng hiếm khi xem quá 10 kết quả.  
> *Users rarely inspect more than ten results.*

Đây là một trade-off kinh điển.  
*This is a classic trade-off.*

Bạn có thể chỉnh các tham số HNSW.  
*You can tune the HNSW parameters.*

Các tham số đổi tốc độ lấy recall.  
*The parameters trade speed for recall.*

Recall 0.99 thường làm tìm kiếm chậm hơn.  
*A recall of 0.99 usually makes search slower.*

Recall 0.90 có thể làm tìm kiếm nhanh hơn nhiều.  
*A recall of 0.90 can make search much faster.*

Không có cấu hình tốt nhất ở cả hai mặt.  
*No configuration is best in both aspects.*

Điểm cân bằng phụ thuộc vào bài toán.  
*The balance point depends on the use case.*

Tìm kiếm sản phẩm có thể bỏ sót một kết quả.  
*Product search may tolerate one missed result.*

Tra cứu hồ sơ y tế có yêu cầu khác.  
*Medical record retrieval has different requirements.*

Một kết quả bị bỏ sót có thể gây hậu quả lớn.  
*One missed result can cause serious consequences.*

### 3.4. Số chiều: nhiều hơn có phải luôn tốt hơn?
*3.4. Dimensions: is more always better?*

Trực giác đầu tiên nghe có vẻ hợp lý.  
*The first intuition sounds reasonable.*

Nhiều chiều hơn có thể chứa nhiều thông tin hơn.  
*More dimensions can hold more information.*

Nhiều thông tin hơn có thể cải thiện tìm kiếm.  
*More information can improve search.*

Điều này đúng đến một mức nhất định.  
*This is true only up to a point.*

Cái giá tăng tuyến tính trên mọi mặt.  
*The cost increases linearly in every aspect.*

| Số chiều / Dimensions | Dung lượng mỗi vector / Size per vector | 10 triệu vector / 10 million vectors | Chi phí một cosine / Cost per cosine |
|---|---|---|---|
| 384 | 1.5 KB | ~15 GB | 1× |
| 1024 | 4 KB | ~40 GB | 2.7× |
| 1536 | 6 KB | ~60 GB | 4× |
| 3072 | 12 KB | ~120 GB | 8× |

Cột thứ ba cần được chú ý đặc biệt.  
*The third column deserves special attention.*

ANN index cần nằm trong RAM.  
*An ANN index must reside in RAM.*

Yêu cầu này giúp tìm kiếm nhanh.  
*This requirement enables fast search.*

> **RAM — Random Access Memory**  
> *RAM: Random Access Memory*
>
> RAM là bộ nhớ trong của máy chủ.  
> *RAM is a server's working memory.*
>
> RAM nhanh hơn ổ cứng hàng trăm lần.  
> *RAM is hundreds of times faster than storage drives.*
>
> RAM cũng đắt hơn rất nhiều.  
> *RAM is also much more expensive.*
>
> Dung lượng RAM thường nhỏ hơn nhiều.  
> *RAM capacity is usually much smaller.*
>
> Index có thể không vừa trong RAM.  
> *The index may not fit in RAM.*
>
> Khi đó, hệ thống phải đọc dữ liệu từ đĩa.  
> *The system must then read data from disk.*
>
> Tốc độ tìm kiếm sẽ sụp đổ.  
> *Search performance will collapse.*

Mười lăm GB phù hợp với máy chủ tầm trung.  
*Fifteen gigabytes fits a mid-range server.*

Một trăm hai mươi GB cần máy chủ RAM lớn.  
*One hundred twenty gigabytes requires a high-memory server.*

Máy chủ đó có thể đắt hơn nhiều lần.  
*That server can cost several times more.*

Một lựa chọn khác là chia dữ liệu ra nhiều máy.  
*Another option is distributing data across multiple machines.*

Giải pháp đó tạo thêm một tầng phức tạp.  
*That solution adds another layer of complexity.*

Số chiều embedding là một quyết định kiến trúc.  
*Embedding dimension is an architectural decision.*

Nó không phải một tham số kỹ thuật nhỏ.  
*It is not a minor technical parameter.*

Một kỹ thuật có thể giảm bớt thế lưỡng nan này.  
*One technique can reduce this dilemma.*

#### Matryoshka — cắt bớt chiều mà gần như không mất chất lượng
*Matryoshka: reducing dimensions with almost no quality loss*

> **Matryoshka Representation Learning — MRL**  
> *Matryoshka Representation Learning: MRL*
>
> MRL là một kỹ thuật huấn luyện model.  
> *MRL is a model training technique.*
>
> Kỹ thuật dồn thông tin quan trọng vào các chiều đầu.  
> *The technique packs important information into early dimensions.*
>
> Tên kỹ thuật bắt nguồn từ búp bê Matryoshka của Nga.  
> *The technique takes its name from Russian Matryoshka dolls.*
>
> Búp bê lớn chứa một búp bê nhỏ.  
> *A large doll contains a smaller doll.*
>
> Mỗi búp bê nhỏ vẫn là một hình thể hoàn chỉnh.  
> *Each smaller doll remains a complete figure.*

Kỹ thuật mang lại một hệ quả cực kỳ hữu dụng.  
*The technique provides an extremely useful consequence.*

Bạn có thể cắt vector 3072 chiều xuống 1024 chiều.  
*You can truncate a 3072-dimensional vector to 1024 dimensions.*

Bạn cũng có thể cắt nó xuống 512 chiều.  
*You can also truncate it to 512 dimensions.*

Bạn chỉ cần loại bỏ phần đuôi.  
*You only need to discard the tail.*

Chất lượng tìm kiếm chỉ giảm rất ít.  
*Search quality decreases only slightly.*

Model thông thường không được huấn luyện theo MRL.  
*A conventional model is not trained with MRL.*

Việc cắt chiều sẽ phá hỏng vector của model đó.  
*Dimension truncation will damage that model's vectors.*

Thông tin được phân bố trên toàn bộ các chiều.  
*Information is distributed across all dimensions.*

Việc bỏ bất kỳ phần nào cũng gây mất mát lớn.  
*Discarding any part causes substantial information loss.*
Con số thực tế đã được báo cáo.  
*Real-world figures have been reported.*

Việc cắt xuống 256 chiều chỉ giảm độ chính xác khoảng 2–3%.  
*Truncation to 256 dimensions reduces accuracy by only about 2–3%.*

Chi phí lưu trữ giảm khoảng 4 lần.  
*Storage cost decreases by about four times.*

Hệ thống có thể chứa hàng trăm triệu vector.  
*The system may contain hundreds of millions of vectors.*

Khoản tiết kiệm khi đó sẽ rất lớn.  
*The resulting savings will be substantial.*

Mức mất mát chất lượng vẫn rất nhỏ.  
*The quality loss remains very small.*

Code minh hoạ nằm bên dưới.  
*The example code appears below.*

```python
# Với model hỗ trợ MRL (OpenAI text-embedding-3, Gemini Embedding):
# With an MRL-capable model such as OpenAI text-embedding-3 or Gemini Embedding:
resp = client.embeddings.create(
    model="text-embedding-3-large",
    input="áo thun cotton nam",
    dimensions=512          # ← yêu cầu 512 chiều thay vì 3072 mặc định
                            # ← request 512 dimensions instead of the default 3072
)
# API trả về vector 512 chiều, đã được cắt và chuẩn hoá lại đúng cách.
# The API returns a properly truncated and renormalized 512-dimensional vector.

# ⚠️ CẢNH BÁO: chỉ làm được với model được HUẤN LUYỆN theo MRL.
# ⚠️ WARNING: this works only with a model TRAINED using MRL.
#    Tự tay cắt vector[:512] của một model thường sẽ phá hỏng nó.
#    Manually slicing vector[:512] from a conventional model will damage it.
```

Một số model hỗ trợ MRL vào năm 2026.  
*Several models support MRL in 2026.*

OpenAI `text-embedding-3-*` hỗ trợ kỹ thuật này.  
*OpenAI `text-embedding-3-*` supports this technique.*

Google Gemini Embedding cũng hỗ trợ kỹ thuật này.  
*Google Gemini Embedding also supports this technique.*

Một số model mã nguồn mở mới cũng hỗ trợ MRL.  
*Some newer open-source models also support MRL.*

💡 Kiến thức này thể hiện tư duy cấp staff.  
*💡 This knowledge demonstrates staff-level thinking.*

Nó cung cấp một núm xoay cho trade-off chi phí–chất lượng.  
*It provides a control for the cost-quality trade-off.*

Bạn không phải chọn cứng một trong hai phía.  
*You do not need to choose one side rigidly.*

Sếp có thể yêu cầu giảm 60% chi phí hạ tầng search.  
*Management may request a 60% reduction in search infrastructure cost.*

Bạn có thể đề xuất giảm từ 3072 xuống 1024 chiều.  
*You can propose reducing dimensions from 3072 to 1024.*

MRL có thể hỗ trợ giải pháp đó.  
*MRL can support that solution.*

Recall có thể giảm khoảng 1–2%.  
*Recall may decrease by about 1–2%.*

Bạn phải đo mức giảm trên dữ liệu thật.  
*You must measure the reduction on real data.*

Đó là câu trả lời của một staff engineer.  
*That is a staff engineer's answer.*

### 3.5. Tự viết lại cosine similarity từ số 0
*3.5. Implementing cosine similarity from scratch*

Interviewer thường yêu cầu viết hàm này tại chỗ.  
*Interviewers often ask candidates to write this function on the spot.*

Hàm này khá ngắn.  
*This function is fairly short.*

Hàm vẫn bộc lộ mức hiểu bản chất của bạn.  
*It still reveals your understanding of the fundamentals.*

Hàm cũng bộc lộ khả năng xử lý trường hợp biên.  
*It also reveals your handling of edge cases.*

#### Bản Python thuần — không thư viện
*Pure Python version without libraries*

```python
def cosine_similarity(a, b):
    """
    Tính cosine similarity giữa hai vector (list số).
    Đây là bản viết tay để thể hiện hiểu bản chất — production dùng NumPy.
    """
    # Kiểm tra đầu vào trước: hai vector khác chiều là lỗi logic,
    # Validate the input first: vectors with different dimensions indicate a logic error,
    # phải báo lỗi rõ ràng chứ không được im lặng cho ra số sai.
    # so report it clearly instead of silently returning an incorrect value.
    if len(a) != len(b):
        raise ValueError(f"Số chiều không khớp: {len(a)} vs {len(b)}")

    # Tính cả ba đại lượng trong MỘT vòng lặp duy nhất, thay vì ba vòng.
    # Compute all three values in ONE loop instead of three loops.
    # Chi tiết nhỏ này là điểm cộng: nó giảm số lần duyệt mảng từ 3 xuống 1.
    # This small detail is a plus because it reduces array scans from three to one.
    dot = 0.0      # tích vô hướng: Σ aᵢ × bᵢ
                   # dot product: Σ aᵢ × bᵢ
    norm_a = 0.0   # bình phương độ dài của a: Σ aᵢ²
                   # squared magnitude of a: Σ aᵢ²
    norm_b = 0.0   # bình phương độ dài của b: Σ bᵢ²
                   # squared magnitude of b: Σ bᵢ²

    for x, y in zip(a, b):     # zip ghép từng cặp phần tử cùng vị trí
                                # zip pairs elements at matching positions
        dot += x * y
        norm_a += x * x
        norm_b += y * y

    # EDGE CASE quan trọng nhất: vector 0 (mọi phần tử bằng 0).
    # Most important EDGE CASE: the zero vector, whose elements are all zero.
    # Nó xảy ra khi input rỗng hoặc bước encode lỗi.
    # It occurs with empty input or a failed encoding step.
    # Vector 0 KHÔNG có hướng, nên cosine không xác định về mặt toán học.
    # The zero vector has NO direction, so cosine is mathematically undefined.
    # Không guard dòng này = chương trình sập vì chia cho 0.
    # Without this guard, the program crashes because of division by zero.
    denominator = (norm_a ** 0.5) * (norm_b ** 0.5)
    if denominator == 0:
        return 0.0             # quy ước: coi như không liên quan
                               # convention: treat the vectors as unrelated

    return dot / denominator
```

#### Bản NumPy — dùng trong thực tế
*NumPy version for practical use*

```python
import numpy as np

def cosine_similarity_np(a, b) -> float:
    a = np.asarray(a, dtype=np.float32)
    b = np.asarray(b, dtype=np.float32)

    denom = np.linalg.norm(a) * np.linalg.norm(b)   # norm() = độ dài vector
                                                    # norm() = vector magnitude
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
    # Normalize the query to unit length
    q = query / np.linalg.norm(query)

    # Chuẩn hoá TỪNG HÀNG của corpus.
    # Normalize EACH ROW of the corpus.
    # axis=1 nghĩa là tính norm theo chiều ngang (mỗi hàng một norm).
    # axis=1 calculates the norm horizontally, with one norm per row.
    # keepdims=True giữ shape (n,1) để phép chia broadcast đúng theo hàng.
    # keepdims=True preserves shape (n,1) for correct row-wise broadcasting.
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)

    # ⭐ MẤU CHỐT HIỆU NĂNG: một phép nhân ma trận thay cho vòng lặp n lần.
    # ⭐ PERFORMANCE KEY: one matrix multiplication replaces n loop iterations.
    # Vì cả hai đã chuẩn hoá, dot product CHÍNH LÀ cosine (chứng minh ở Mục 2.2).
    # Both inputs are normalized, so dot product IS cosine, as shown in Section 2.2.
    # NumPy đẩy phép này xuống thư viện đại số tuyến tính viết bằng C,
    # NumPy delegates this operation to a linear algebra library written in C,
    # chạy nhanh hơn vòng lặp Python khoảng 100 lần.
    # which runs about 100 times faster than a Python loop.
    sims = c @ q                      # kết quả shape (n,)
                                        # result shape: (n,)

    # argsort trả về CHỈ SỐ sắp xếp tăng dần.
    # argsort returns INDICES in ascending order.
    # Dấu trừ trước sims biến nó thành sắp xếp giảm dần -> lấy điểm CAO nhất.
    # The minus sign before sims creates descending order and selects the HIGHEST scores.
    idx = np.argsort(-sims)[:k]
    return idx, sims[idx]
```

#### Bản JavaScript thuần
*Pure JavaScript version*

```javascript
function cosineSimilarity(a, b) {
  if (a.length !== b.length) {
    throw new Error(`Số chiều không khớp: ${a.length} vs ${b.length}`);
  }

  let dot = 0, normA = 0, normB = 0;

  // Vòng lặp chỉ số (không phải for...of) vì nhanh hơn đáng kể
  // An index loop is significantly faster than for...of
  // với Float32Array — kiểu dữ liệu mà Transformers.js trả về.
  // with Float32Array, the data type returned by Transformers.js.
  for (let i = 0; i < a.length; i++) {
    dot   += a[i] * b[i];
    normA += a[i] * a[i];
    normB += b[i] * b[i];
  }

  const denom = Math.sqrt(normA) * Math.sqrt(normB);
  return denom === 0 ? 0 : dot / denom;   // guard chia 0
                                               // guard against division by zero
}
```

> ⚠️ Ba hàm trên rất hữu ích cho việc học.  
> *⚠️ The three functions above are highly useful for learning.*
>
> Chúng cũng phù hợp với bài phỏng vấn.  
> *They are also suitable for interview exercises.*
>
> Bạn không nên dùng chúng cho hàng triệu vector trong production.  
> *You should not use them for millions of vectors in production.*
>
> Mục 3.3 đã giải thích nguyên nhân.  
> *Section 3.3 explained the reason.*
>
> Tìm kiếm vét cạn có độ phức tạp `O(n×d)`.  
> *Brute-force search has `O(n×d)` complexity.*
>
> Chi phí đó quá cao.  
> *That cost is too high.*
>
> Production sử dụng ANN index.  
> *Production systems use an ANN index.*
>
> Các lựa chọn gồm pgvector, FAISS hoặc Qdrant.  
> *Options include pgvector, FAISS, or Qdrant.*
>
> Interviewer có thể yêu cầu dùng hàm này với 10 triệu tài liệu.  
> *An interviewer may request this function for 10 million documents.*
>
> Đó là một câu hỏi bẫy.  
> *That is a trick question.*
>
> Bạn nên đề xuất ANN index.  
> *You should propose an ANN index.*
>
> Bạn cũng nên nêu ước lượng tại Mục 3.3.  
> *You should also cite the estimate from Section 3.3.*
>
> Câu trả lời đó sẽ giúp bạn ghi điểm.  
> *That answer will help you stand out.*

### 3.6. 🧩 [Ngoài bài gốc] Các trường hợp biên phải thủ sẵn
*3.6. 🧩 [Beyond the original lesson] Edge cases you must anticipate*

Danh sách này chứa các tình huống có thể phá hỏng production.  
*This list contains situations that can break production.*

Tác giả đã trực tiếp gặp một số tình huống.  
*The author has encountered some of these situations directly.*

Tác giả cũng đã thấy người khác gặp chúng.  
*The author has also seen others encounter them.*

**1. Vector 0 hoặc NaN.**  
***1. The zero vector or NaN.***

> **NaN — Not a Number**  
> *NaN: Not a Number*
>
> NaN là một giá trị đặc biệt.  
> *NaN is a special value.*
>
> Nó biểu thị một kết quả tính toán không hợp lệ.  
> *It represents an invalid computation result.*
>
> Ví dụ là phép tính 0 chia 0.  
> *An example is zero divided by zero.*
>
> NaN có một tính chất nguy hiểm.  
> *NaN has a dangerous property.*
>
> NaN có thể lây lan qua các phép tính.  
> *NaN can propagate through calculations.*
>
> Mọi phép tính chứa NaN đều cho ra NaN.  
> *Every calculation containing NaN produces NaN.*
>
> Một vector NaN có thể lọt vào database.  
> *A NaN vector can enter the database.*
>
> Vector đó có thể phá hỏng toàn bộ thứ tự xếp hạng.  
> *That vector can corrupt the entire ranking.*

Chuỗi rỗng là một nguyên nhân thường gặp.  
*An empty string is a common cause.*

Chuỗi chỉ chứa dấu cách cũng là một nguyên nhân.  
*A whitespace-only string is another cause.*

Lỗi mạng có thể làm API trả dữ liệu thiếu.  
*A network failure can make the API return incomplete data.*

Bạn phải kiểm tra trước khi ghi database.  
*You must validate before writing to the database.*

Bạn không nên chờ đến lúc đọc dữ liệu.  
*You should not wait until data retrieval.*

```python
import numpy as np

def is_valid_embedding(vec, expected_dim):
    if vec is None:                       return False
    if len(vec) != expected_dim:          return False   # sai số chiều
                                                          # incorrect dimension count
    if np.isnan(vec).any():               return False   # chứa NaN
                                                          # contains NaN
    if np.isinf(vec).any():               return False   # chứa vô cực
                                                          # contains infinity
    if np.linalg.norm(vec) < 1e-8:        return False   # vector gần như bằng 0
                                                          # vector is nearly zero
    return True
```

**2. Truncation — văn bản dài bị cắt cụt âm thầm.**  
***2. Truncation: long text is silently cut off.***

> **Truncation — cắt cụt**  
> *Truncation: cutting off excess input*
>
> Mọi model embedding đều có giới hạn token đầu vào.  
> *Every embedding model has an input token limit.*
>
> Giới hạn thường là 256 token.  
> *The limit is often 256 tokens.*
>
> Một số model có giới hạn 512 token.  
> *Some models have a 512-token limit.*
>
> Một số model có giới hạn 8192 token.  
> *Some models have an 8192-token limit.*
>
> Model quyết định giới hạn cụ thể.  
> *The specific model determines the limit.*
>
> Văn bản vượt giới hạn sẽ mất phần đuôi.  
> *Text beyond the limit loses its tail.*
>
> Việc cắt xảy ra trước bước xử lý của model.  
> *The truncation occurs before model processing.*

Đây là edge case nguy hiểm nhất.  
*This is the most dangerous edge case.*

Quá trình cắt cụt hoàn toàn im lặng.  
*The truncation process is completely silent.*

Hệ thống không báo lỗi.  
*The system reports no error.*

Hệ thống cũng không đưa cảnh báo.  
*The system also provides no warning.*

Bạn có thể gửi vào một tài liệu 50 trang.  
*You may submit a 50-page document.*

Model chỉ đọc hai trang đầu.  
*The model reads only the first two pages.*

Model loại bỏ 48 trang còn lại.  
*The model discards the remaining 48 pages.*

Model vẫn trả về một vector bình thường.  
*The model still returns a normal-looking vector.*

Bạn có thể lưu vector đó vào database.  
*You may store that vector in the database.*

Bạn có thể tin rằng toàn bộ tài liệu đã được index.  
*You may believe the whole document was indexed.*

Sáu tháng sau, một người phát hiện vấn đề.  
*Six months later, someone discovers the problem.*

Tìm kiếm không trả về nội dung cuối tài liệu.  
*Search never returns content from the document's end.*

Chunking là cách xử lý vấn đề này.  
*Chunking handles this problem.*

> **Chunking — chia đoạn**  
> *Chunking: splitting a document into smaller segments*
>
> Chunking cắt tài liệu dài thành nhiều đoạn nhỏ.  
> *Chunking splits a long document into smaller segments.*
>
> Hệ thống embed từng đoạn riêng biệt.  
> *The system embeds each segment separately.*
>
> Hệ thống lưu mọi đoạn vào database.  
> *The system stores every segment in the database.*
>
> Truy vấn sẽ tìm đoạn liên quan nhất.  
> *A query finds the most relevant segment.*
>
> Truy vấn không tìm toàn bộ tài liệu như một khối.  
> *A query does not search the whole document as one block.*

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
                                       # step size is smaller than the chunk size
    chunks = []
    for i in range(0, len(words), step):
        chunk = " ".join(words[i : i + max_tokens])
        if chunk.strip():              # bỏ qua đoạn rỗng ở cuối
                                       # skip an empty trailing chunk
            chunks.append(chunk)
    return chunks
```

Một cách tốt hơn là cắt theo ranh giới ngữ nghĩa.  
*A better method splits at semantic boundaries.*

Ranh giới có thể là cuối đoạn văn.  
*A boundary can be the end of a paragraph.*

Ranh giới cũng có thể là cuối một mục.  
*A boundary can also be the end of a section.*

Cách này tốt hơn việc đếm từ máy móc.  
*This method is better than mechanical word counting.*

Việc cắt giữa câu làm hỏng ngữ nghĩa.  
*Splitting inside a sentence damages semantics.*

Cả hai nửa đều mất một phần ngữ cảnh.  
*Both halves lose part of their context.*

Chiến lược chunking là một chủ đề riêng.  
*Chunking strategy is a separate topic.*

Nó ảnh hưởng lớn đến chất lượng RAG.  
*It strongly affects RAG quality.*

**3. Sai ngôn ngữ.**  
***3. Wrong language.***

Nhiều model phổ biến chủ yếu học từ tiếng Anh.  
*Many popular models are trained mainly on English.*

Bạn vẫn có thể đưa tiếng Việt vào model.  
*You can still send Vietnamese text to the model.*

Model vẫn trả về một vector bình thường.  
*The model still returns a normal-looking vector.*

Chất lượng ngữ nghĩa có thể kém hơn nhiều.  
*Semantic quality may be much worse.*

Hệ thống vẫn không báo lỗi.  
*The system still reports no error.*

Bạn nên tự tạo một bộ 20–30 cặp câu tiếng Việt.  
*You should create a set of 20–30 Vietnamese sentence pairs.*

Bạn phải biết chắc các cặp này đồng nghĩa.  
*You must know that these pairs are synonymous.*

Sau đó, bạn đo cosine cho từng cặp.  
*Then you measure cosine for each pair.*

Model tốt phải cho điểm cao với cặp đồng nghĩa.  
*A good model must score synonymous pairs highly.*

Điểm đó phải cao hơn rõ rệt so với cặp ngẫu nhiên.  
*Those scores must clearly exceed random-pair scores.*

Điểm lẫn lộn cho thấy model không phù hợp.  
*Mixed scores indicate an unsuitable model.*

Bạn nên chuyển sang model đa ngôn ngữ.  
*You should switch to a multilingual model.*

Các lựa chọn gồm BGE-M3 hoặc Qwen3-Embedding.  
*Options include BGE-M3 or Qwen3-Embedding.*

Cohere `embed-v4` cũng là một lựa chọn.  
*Cohere `embed-v4` is another option.*

**4. Cosine ra giá trị âm.**  
***4. Cosine produces a negative value.***

Giá trị âm hoàn toàn hợp lệ về toán học.  
*A negative value is mathematically valid.*

Mục 3.1 đã giải thích điều này.  
*Section 3.1 explained this fact.*

Code có thể ngầm giả định miền `[0,1]`.  
*Code may implicitly assume the `[0,1]` range.*

Ví dụ là cách hiển thị phần trăm cho người dùng.  
*An example is displaying a percentage to users.*

Hệ thống có thể hiển thị độ khớp bằng −12%.  
*The system may display a match score of −12%.*

Bạn có thể dùng `max(0, sim)`.  
*You can use `max(0, sim)`.*

Một lựa chọn khác là cosine distance.  
*Another option is cosine distance.*

**5. Trộn vector đã chuẩn hoá và chưa chuẩn hoá.**  
***5. Mixing normalized and unnormalized vectors.***

Lỗi này đã xuất hiện trong Mục 2.5.  
*This error appeared in Section 2.5.*

Nó thuộc nhóm lỗi âm thầm gây sai.  
*It belongs to the class of silent errors.*

Lỗi thường xảy ra trong hệ thống có nhiều luồng ghi.  
*The error often occurs in systems with multiple write paths.*

Các team khác nhau có thể viết từng luồng.  
*Different teams may implement each path.*

Bạn phải chuẩn hoá tập trung tại một vị trí.  
*You must centralize normalization in one location.*

Vị trí đó nằm ngay trước bước ghi database.  
*That location sits immediately before the database write.*

Bạn không nên để mỗi nơi tự chuẩn hoá.  
*You should not let each path normalize independently.*

**6. Nhà cung cấp âm thầm đổi phiên bản model.**  
***6. The provider silently changes the model version.***

Đây là edge case nguy hiểm nhất.  
*This is the most dangerous edge case.*

Nó cũng là edge case bất ngờ nhất.  
*It is also the most surprising edge case.*

Mục 4.5 sẽ giải thích chi tiết.  
*Section 4.5 will explain it in detail.*

Vấn đề này thuộc cấp độ kiến trúc.  
*This issue belongs at the architectural level.*

### ✅ Self-check Phần 3
*✅ Part 3 self-check*

**Câu 1.**  
***Question 1.***

Phép chia cho `|A| × |B|` loại bỏ độ dài như thế nào?  
*How does division by `|A| × |B|` remove magnitude?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Định nghĩa hình học là `A · B = |A| × |B| × cos(θ)`.  
*The geometric definition is `A · B = |A| × |B| × cos(θ)`.*

Ta chia cả hai vế cho `|A| × |B|`.  
*We divide both sides by `|A| × |B|`.*

Hai thừa số độ dài sẽ bị triệt tiêu.  
*The two magnitude factors cancel out.*

Kết quả chỉ còn `cos(θ)`.  
*Only `cos(θ)` remains.*

Đại lượng này chỉ phụ thuộc vào góc.  
*This quantity depends only on the angle.*

Độ dài không bị bỏ qua một cách nhân tạo.  
*Magnitude is not ignored artificially.*

Phép chia khử độ dài một cách tự nhiên.  
*The division removes magnitude naturally.*

</details>

**Câu 2.**  
***Question 2.***

Vector đã chuẩn hoá có thứ hạng cosine khác L2 không?  
*Do normalized vectors rank differently under cosine and L2?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Thứ hạng không khác nhau.  
*The rankings do not differ.*

Công thức là `|A−B|² = 2 − 2·cos(θ)`.  
*The formula is `|A−B|² = 2 − 2·cos(θ)`.*

Công thức đúng với hai vector đã chuẩn hoá.  
*The formula holds for two normalized vectors.*

Khoảng cách là một hàm giảm nghiêm ngặt của cosine.  
*Distance is a strictly decreasing function of cosine.*

Hai phép đo đơn điệu tương đương.  
*The two measures are monotonically equivalent.*

Chúng tạo cùng một thứ tự xếp hạng.  
*They produce the same ranking order.*

Giá trị điểm của chúng vẫn khác nhau.  
*Their score values still differ.*

</details>

**Câu 3.**  
***Question 3.***

Matryoshka embedding cho phép làm gì?  
*What does Matryoshka embedding allow?*

Tại sao model thường không hỗ trợ cách đó?  
*Why do conventional models not support that method?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Matryoshka cho phép cắt cụt vector.  
*Matryoshka allows vector truncation.*

Ví dụ là cắt từ 3072 xuống 512 chiều.  
*An example is reducing 3072 dimensions to 512.*

Việc cắt giúp tiết kiệm dung lượng lưu trữ.  
*The reduction saves storage capacity.*

Việc cắt cũng tăng tốc tìm kiếm.  
*The reduction also accelerates search.*

Độ chính xác chỉ giảm vài phần trăm.  
*Accuracy decreases by only a few percentage points.*

Model thường phân bố thông tin trên mọi chiều.  
*Conventional models distribute information across all dimensions.*

Việc bỏ phần đuôi gây mất mát lớn.  
*Discarding the tail causes substantial loss.*

Model MRL được huấn luyện có chủ đích.  
*An MRL model is trained intentionally.*

Nó dồn thông tin quan trọng vào các chiều đầu.  
*It packs important information into early dimensions.*

</details>

**Câu 4.**  
***Question 4.***

Tại sao truncation nguy hiểm hơn vector NaN?  
*Why is truncation more dangerous than a NaN vector?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

NaN thường gây lỗi rõ ràng.  
*NaN often causes an obvious error.*

NaN cũng dễ được phát hiện bằng kiểm tra.  
*NaN is also easy to detect through validation.*

Truncation diễn ra hoàn toàn im lặng.  
*Truncation occurs completely silently.*

Model vẫn trả về một vector hợp lệ.  
*The model still returns a valid vector.*

Vector đó chỉ mã hoá phần đầu tài liệu.  
*That vector encodes only the document's beginning.*

Hệ thống có thể chạy bình thường nhiều tháng.  
*The system may run normally for many months.*

Nội dung cuối tài liệu không bao giờ được tìm thấy.  
*Content near the document's end is never found.*

Chunking là cách xử lý vấn đề này.  
*Chunking is the solution to this problem.*

Bạn cũng nên theo dõi tỉ lệ input bị cắt.  
*You should also monitor the input truncation rate.*

</details>

---

## Phần 4 — 🟣 STAFF LEVEL (Tư duy hệ thống & lãnh đạo kỹ thuật)
*Part 4: 🟣 STAFF LEVEL (Systems thinking and technical leadership)*

> Đây là phần phân biệt senior engineer với staff engineer.  
> *This section distinguishes a senior engineer from a staff engineer.*
>
> Senior engineer giải quyết được bài toán kỹ thuật.  
> *A senior engineer can solve a technical problem.*
>
> Staff engineer nhận ra bài toán đáng giải.  
> *A staff engineer identifies the problem worth solving.*
>
> Staff engineer hiểu cái giá của giải pháp.  
> *A staff engineer understands the solution's cost.*
>
> Staff engineer giải thích được vấn đề cho người không viết code.  
> *A staff engineer can explain the issue to non-programmers.*
>
> Một số thuật ngữ sẽ xuất hiện liên tục từ đây.  
> *Several terms will appear repeatedly from this point.*
>
> **Scale — quy mô**  
> *Scale*
>
> Scale mô tả sự tăng mạnh của dữ liệu hoặc lưu lượng.  
> *Scale describes major growth in data or traffic.*
>
> Scale dọc sử dụng một máy mạnh hơn.  
> *Vertical scaling uses a more powerful machine.*
>
> Scale ngang sử dụng nhiều máy hơn.  
> *Horizontal scaling uses more machines.*
>
> Scale dọc có cách triển khai đơn giản hơn.  
> *Vertical scaling has a simpler implementation.*
>
> Scale dọc có một giới hạn cứng.  
> *Vertical scaling has a hard limit.*
>
> Scale ngang không có giới hạn tương tự.  
> *Horizontal scaling has no similar limit.*
>
> Scale ngang tạo ra nhiều độ phức tạp hơn.  
> *Horizontal scaling creates much more complexity.*
>
> **Bottleneck — điểm nghẽn**  
> *Bottleneck*
>
> Bottleneck là khâu chậm nhất trong toàn hệ thống.  
> *A bottleneck is the slowest stage in the whole system.*
>
> Nó quyết định tốc độ chung của hệ thống.  
> *It determines the system's overall speed.*
>
> Tối ưu khâu khác không cải thiện bottleneck.  
> *Optimizing another stage does not improve the bottleneck.*
>
> Công sức đó thường bị lãng phí.  
> *That effort is usually wasted.*
>
> **Latency — độ trễ**  
> *Latency*
>
> Latency là thời gian từ yêu cầu đến kết quả.  
> *Latency is the time from request to result.*
>
> Đơn vị thường dùng là mili giây.  
> *The common unit is milliseconds.*
>
> **Throughput — thông lượng**  
> *Throughput*
>
> Throughput là số công việc được xử lý trong một khoảng thời gian.  
> *Throughput is the amount of work processed within a time period.*
>
> Latency khác throughput.  
> *Latency differs from throughput.*
>
> Hai chỉ số này thường có sự đánh đổi.  
> *These two metrics often involve a trade-off.*

### 4.1. Bottleneck ở đâu: không phải công thức cosine
*4.1. Where the bottleneck is: not the cosine formula*

Người mới thường nghĩ phép tính cosine là khâu chậm.  
*Beginners often think cosine computation is the slow stage.*

Nhận định đó không đúng.  
*That assumption is incorrect.*

Trong bài pgvector, bottleneck nằm ở lưu trữ.  
*In the pgvector lesson, the bottleneck lies in storage.*

Bottleneck cũng nằm ở quá trình tìm kiếm.  
*The bottleneck also lies in search.*

RAM phải chứa index.  
*RAM must hold the index.*

Hệ thống cũng cần thời gian để duyệt index.  
*The system also needs time to traverse the index.*

Trong bài này, bottleneck nằm ở việc sản xuất vector.  
*In this lesson, the bottleneck lies in vector production.*

Thời gian chạy model là bottleneck đầu tiên.  
*Model execution time is the first bottleneck.*

Chi phí API là bottleneck thứ hai.  
*API cost is the second bottleneck.*

Embedding truy vấn thời gian thực là bottleneck thứ ba.  
*Real-time query embedding is the third bottleneck.*

Bốn kỹ thuật sau giải quyết các bottleneck này.  
*The following four techniques address these bottlenecks.*

Các kỹ thuật được xếp theo mức độ quan trọng.  
*The techniques are ordered by importance.*

#### Kỹ thuật 1 — Xử lý theo lô
*Technique 1: batching*

Embedding từng câu gây lãng phí rất lớn.  
*Embedding one sentence at a time causes major waste.*

GPU được thiết kế cho tính toán song song quy mô lớn.  
*GPUs are designed for large-scale parallel computation.*

Một câu chỉ sử dụng khoảng 1% năng lực GPU.  
*One sentence uses only about 1% of GPU capacity.*

Phần lớn thời gian dành cho chi phí khởi động cố định.  
*Most time goes to fixed startup overhead.*

Bạn có thể gom 32–256 câu thành một batch.  
*You can group 32–256 sentences into one batch.*

Model xử lý cả batch trong một lượt.  
*The model processes the whole batch in one pass.*

Tổng thời gian gần như không tăng.  
*Total time barely increases.*

Số câu được xử lý tăng hàng chục lần.  
*The number of processed sentences increases by dozens of times.*

```python
# ❌ CHẬM: 1 triệu lần gọi model, mỗi lần lãng phí chi phí khởi động
# ❌ SLOW: one million model calls, with startup overhead wasted each time
for doc in documents:
    vec = model.encode(doc)
    save_to_db(vec)

# ✅ NHANH: chia thành các lô, số lần gọi giảm hàng chục lần
# ✅ FAST: split inputs into batches and reduce calls by dozens of times
BATCH = 128
for i in range(0, len(documents), BATCH):
    batch = documents[i : i + BATCH]
    vecs = model.encode(batch, normalize_embeddings=True)   # 1 lần gọi cho 128 câu
                                                               # one call for 128 sentences
    save_batch_to_db(vecs)                                   # 1 lần ghi cho 128 vector
                                                               # one write for 128 vectors
```

💡 Batching còn giúp giảm số lượt gọi API.  
*💡 Batching also reduces the number of API calls.*

Mỗi lượt gọi API phải đi qua mạng.  
*Every API call must travel through the network.*

Chuyến đi này thường mất 50–200ms.  
*This round trip usually takes 50–200ms.*

Một batch 128 câu chỉ cần một lượt gọi.  
*A batch of 128 sentences needs only one call.*

Cách này loại bỏ 127 chuyến đi mạng.  
*This method removes 127 network round trips.*

#### Kỹ thuật 2 — Nhớ đệm
*Technique 2: caching*

Embedding là một hàm xác định.  
*Embedding is a deterministic function.*

Cùng văn bản sẽ cho cùng vector.  
*The same text produces the same vector.*

Điều này yêu cầu cùng model.  
*This requires the same model.*

Điều này cũng yêu cầu cùng phiên bản.  
*This also requires the same version.*

Tính lại lần thứ hai là lãng phí hoàn toàn.  
*Computing it a second time is completely wasteful.*

```python
import hashlib, json

def get_embedding_cached(text, model_name, cache):
    # Tạo "vân tay" duy nhất từ nội dung + tên model.
    # Create a unique fingerprint from the content and model name.
    # Phải có model_name trong khoá! Nếu không, đổi model sẽ lấy nhầm
    # The key must include model_name! Otherwise, a model change will retrieve
    # vector cũ trong cache — một bug cực kỳ khó tìm.
    # an old cached vector, which creates an extremely difficult bug.
    key = hashlib.sha256(f"{model_name}::{text}".encode()).hexdigest()

    if key in cache:
        return cache[key]              # tiết kiệm 1 lần gọi model + 1 lần trả tiền
                                       # save one model call and one paid request

    vec = model.encode(text, normalize_embeddings=True)
    cache[key] = vec
    return vec
```

> **Hash — băm**  
> *Hash*
>
> Hash biến dữ liệu thành một chuỗi có độ dài cố định.  
> *A hash converts data into a fixed-length string.*
>
> Chuỗi này đóng vai trò như một vân tay.  
> *This string acts like a fingerprint.*
>
> Cùng đầu vào luôn tạo cùng một hash.  
> *The same input always creates the same hash.*
>
> Hai đầu vào khác nhau gần như luôn tạo hash khác nhau.  
> *Different inputs almost always create different hashes.*
>
> Hash phù hợp làm cache key.  
> *A hash works well as a cache key.*
>
> Nó ngắn gọn.  
> *It is compact.*
>
> Nó cũng được so sánh rất nhanh.  
> *It is also compared very quickly.*

Mức tiết kiệm thực tế lớn hơn nhiều người nghĩ.  
*Real-world savings are larger than many people expect.*

Câu truy vấn lặp lại tạo cơ hội tiết kiệm đầu tiên.  
*Repeated queries create the first saving opportunity.*

Một số từ khoá phổ biến chiếm phần lớn lưu lượng.  
*A few popular keywords account for most traffic.*

Cache các từ khoá đó gần như không tốn chi phí.  
*Caching those keywords costs almost nothing.*

Nội dung không đổi tạo cơ hội tiết kiệm thứ hai.  
*Unchanged content creates the second saving opportunity.*

Giá sản phẩm có thể thay đổi.  
*A product price may change.*

Mô tả sản phẩm có thể giữ nguyên.  
*The product description may remain unchanged.*

Khi đó hệ thống không cần embedding lại mô tả.  
*The system does not need to re-embed the description.*

#### Kỹ thuật 3 — Tách đường nóng và đường nguội
*Technique 3: separating the hot path from the cold path*

Hệ thống có hai luồng embedding khác nhau.  
*The system has two different embedding flows.*

Mỗi luồng có một yêu cầu riêng.  
*Each flow has its own requirement.*

|  | Đường nguội / Cold path | Đường nóng / Hot path |
|---|---|---|
| Việc gì / Task | Embed tài liệu để nạp vào kho / Embed documents for indexing | Embed truy vấn người dùng / Embed the user's query |
| Ai chờ / Waiting party | Không ai / Nobody | Người dùng / The user |
| Ưu tiên / Priority | Throughput cao / High throughput | Latency thấp / Low latency |
| Mức chấp nhận / Acceptable target | Chạy nền vài giờ / Several background hours | Tối đa khoảng 50ms / About 50ms maximum |
| Cách làm / Method | Batch lớn qua queue / Large batches through a queue | Batch nhỏ, self-host, cache mạnh / Small batches, self-hosting, strong caching |

> **Queue — hàng đợi bất đồng bộ**  
> *Queue*
>
> Queue xếp công việc vào một hàng chờ.  
> *A queue places work into a waiting line.*
>
> Worker lấy công việc ra xử lý dần.  
> *Workers take jobs from the queue gradually.*
>
> Người gửi không phải chờ kết quả.  
> *The sender does not need to wait for the result.*
>
> Queue giúp hệ thống chịu tải đột biến.  
> *A queue helps the system absorb traffic spikes.*
>
> Ví dụ là 10.000 tài liệu xuất hiện cùng lúc.  
> *An example is 10,000 documents arriving at once.*
>
> Các tài liệu sẽ xếp hàng.  
> *The documents will wait in the queue.*
>
> Hệ thống không bị sập ngay lập tức.  
> *The system does not crash immediately.*

💡 Nhiều ứng viên chỉ nghĩ đến embedding tài liệu.  
*💡 Many candidates think only about document embedding.*

Họ quên rằng truy vấn cũng cần embedding.  
*They forget that queries also need embedding.*

Embedding truy vấn nằm trên đường nóng.  
*Query embedding sits on the hot path.*

API bên ngoài tạo thêm độ trễ mạng.  
*An external API adds network latency.*

Mỗi lượt tìm kiếm có thể chậm thêm 50–200ms.  
*Every search may become 50–200ms slower.*

Người dùng có thể cảm nhận độ trễ đó.  
*Users can perceive that delay.*

Đây là lý do mạnh để self-host model truy vấn.  
*This is a strong reason to self-host the query model.*

Đường indexing vẫn có thể sử dụng API.  
*The indexing path may still use an API.*

#### Kỹ thuật 4 — Tính tiền trước khi hứa
*Technique 4: calculating cost before making promises*

Đây là một kỹ năng đặc trưng của staff engineer.  
*This is a characteristic staff engineer skill.*

Mọi quyết định kỹ thuật cần được quy đổi thành tiền.  
*Every technical decision should be translated into money.*

Ví dụ sau sử dụng 100 triệu tài liệu.  
*The following example uses 100 million documents.*

Mỗi tài liệu có trung bình 200 token.  
*Each document averages 200 tokens.*

Model API có giá khoảng 0.02 USD mỗi triệu token.  
*The API model costs about USD 0.02 per million tokens.*

Bạn phải kiểm tra lại mức giá hiện hành.  
*You must verify the current price.*

Giá có thể thay đổi.  
*The price can change.*

```text
Tổng token = 100.000.000 × 200 = 20.000.000.000 token = 20.000 triệu token
Total tokens = 100,000,000 × 200 = 20,000,000,000 tokens = 20,000 million tokens

Chi phí = 20.000 × 0.02 USD ≈ 400 USD
Cost = 20,000 × USD 0.02 ≈ USD 400

Con số này áp dụng cho một lần embedding toàn bộ dữ liệu.
This figure applies to one full embedding pass.
```

400 USD có vẻ không quá lớn.  
*USD 400 may not seem very large.*

Đó chỉ là chi phí cho một lần.  
*That is the cost of only one pass.*

Đổi model sẽ tạo thêm 400 USD.  
*Changing the model creates another USD 400 cost.*

Model đắt hơn 10 lần có thể xuất hiện dễ dàng.  
*A model costing ten times more is easy to encounter.*

Khi đó chi phí thành 4.000 USD mỗi lần.  
*The cost then becomes USD 4,000 per pass.*

Tiếng Việt thường tốn nhiều token hơn tiếng Anh.  
*Vietnamese often consumes more tokens than English.*

Mức tăng có thể từ 1,5 đến 2 lần.  
*The increase may range from 1.5 to 2 times.*

Tokenizer thường được tối ưu cho tiếng Anh.  
*Tokenizers are usually optimized for English.*

Bạn phải nhân mức ước tính ban đầu lên tương ứng.  
*You must scale the original estimate accordingly.*

Ước tính trên chưa gồm truy vấn hằng ngày.  
*The estimate above excludes daily queries.*

Nó cũng chưa gồm chi phí lưu trữ.  
*It also excludes storage cost.*

Chi phí RAM cho index cũng chưa được tính.  
*RAM cost for the index is also excluded.*

💡 Staff engineer cần nêu ba con số trước khi chọn model.  
*💡 A staff engineer should state three figures before choosing a model.*

Con số đầu tiên là chi phí embedding corpus ban đầu.  
*The first figure is the initial corpus embedding cost.*

Con số thứ hai là chi phí embedding lại.  
*The second figure is the re-embedding cost.*

Con số thứ ba là chi phí truy vấn hằng tháng.  
*The third figure is the monthly query cost.*

Ba con số này thường quan trọng hơn benchmark.  
*These three figures often matter more than benchmark scores.*

### 4.2. Chọn model — quy trình đúng
*4.2. Choosing a model: the correct process*

Sai lầm phổ biến nhất bắt đầu từ bảng MTEB.  
*The most common mistake starts with the MTEB leaderboard.*

Nhiều người chọn model đứng đầu.  
*Many people select the top-ranked model.*

Sau đó họ kết thúc quá trình đánh giá.  
*They then end the evaluation process.*

> **MTEB — Massive Text Embedding Benchmark**  
> *MTEB: Massive Text Embedding Benchmark*
>
> MTEB là bộ benchmark tiêu chuẩn cho embedding model.  
> *MTEB is a standard benchmark suite for embedding models.*
>
> Hugging Face duy trì bộ benchmark này.  
> *Hugging Face maintains this benchmark suite.*
>
> Bảng tiếng Anh chứa 56 tập dữ liệu.  
> *The English leaderboard contains 56 datasets.*
>
> Các tập dữ liệu bao phủ tám loại nhiệm vụ.  
> *The datasets cover eight task types.*
>
> Các nhiệm vụ gồm tìm kiếm, phân loại và gom cụm.  
> *The tasks include retrieval, classification, and clustering.*
>
> Xếp hạng lại cũng là một nhiệm vụ.  
> *Reranking is also one task.*
>
> MMTEB là bảng benchmark đa ngôn ngữ.  
> *MMTEB is the multilingual benchmark leaderboard.*
>
> MMTEB chứa 131 nhiệm vụ.  
> *MMTEB contains 131 tasks.*
>
> Các nhiệm vụ bao phủ hơn 250 ngôn ngữ.  
> *The tasks cover more than 250 languages.*
>
> **Benchmark — bộ kiểm tra chuẩn hoá**  
> *Benchmark*
>
> Benchmark giúp so sánh các model công bằng hơn.  
> *A benchmark helps compare models more fairly.*
>
> Mọi model được đánh giá trên cùng tiêu chí.  
> *Every model is evaluated using the same criteria.*

MTEB rất hữu ích.  
*MTEB is very useful.*

Bạn vẫn phải biết ba giới hạn của nó.  
*You still need to understand its three limitations.*

**Một là điểm số do nhà cung cấp tự nộp.**  
***First, providers submit their own scores.***

Mã đánh giá là mã nguồn mở.  
*The evaluation code is open source.*

Không có bên thứ ba độc lập kiểm chứng mọi kết quả.  
*No independent third party verifies every result.*

Bạn nên đọc thông cáo của hãng với thái độ thận trọng.  
*You should read vendor announcements cautiously.*

**Hai là benchmark sử dụng dữ liệu chung.**  
***Second, benchmarks use general data.***

Benchmark không sử dụng dữ liệu riêng của bạn.  
*Benchmarks do not use your own data.*

Model tốt cho tin tức tiếng Anh có thể kém cho tiếng Việt.  
*A model strong on English news may perform poorly on Vietnamese.*

Mô tả sản phẩm có đặc điểm riêng.  
*Product descriptions have unique characteristics.*

Văn bản pháp lý cũng có đặc điểm riêng.  
*Legal text also has unique characteristics.*

Chênh lệch chất lượng có thể rất lớn.  
*The quality difference can be substantial.*

**Ba là các bảng không luôn so sánh trực tiếp được.**  
***Third, leaderboards are not always directly comparable.***

MTEB v1 sử dụng một bộ dữ liệu.  
*MTEB v1 uses one dataset collection.*

MTEB v2 sử dụng một bộ dữ liệu khác.  
*MTEB v2 uses a different dataset collection.*

Bảng tiếng Anh có cách tính riêng.  
*The English leaderboard has its own scoring method.*

Bảng đa ngôn ngữ có cách tính khác.  
*The multilingual leaderboard uses another method.*

So sánh chéo hai bảng là sai.  
*Cross-comparing the two leaderboards is incorrect.*

Quy trình đúng gồm bốn bước.  
*The correct process contains four steps.*

Bạn phải thực hiện chúng theo thứ tự.  
*You must perform them in order.*

**Bước 1 — Liệt kê ràng buộc trước khi xem điểm.**  
***Step 1: list constraints before viewing scores.***

Bạn nên sắp xếp ràng buộc theo mức ưu tiên.  
*You should order constraints by priority.*

| Ràng buộc / Constraint | Câu hỏi cần trả lời / Question |
|---|---|
| Loại dữ liệu / Data type | Chỉ có text hay có cả ảnh và PDF? / Is the data text-only, or does it include images and PDFs? |
| Ngôn ngữ / Language | Có tiếng Việt không? Có bao nhiêu ngôn ngữ? / Is Vietnamese included? How many languages exist? |
| Quyền riêng tư / Privacy | Dữ liệu có được phép rời hệ thống không? / May the data leave the system? |
| Độ trễ / Latency | Đường nóng cần bao nhiêu mili giây? / How many milliseconds does the hot path require? |
| Ngân sách / Budget | Ngân sách tháng cho embedding và hạ tầng là bao nhiêu? / What is the monthly budget for embedding and infrastructure? |

Ràng buộc quyền riêng tư có thể cấm gửi dữ liệu ra ngoài.  
*A privacy constraint may forbid external data transfer.*

Khi đó mọi API model bị loại.  
*Every API model is then eliminated.*

Thứ hạng benchmark không thay đổi quyết định này.  
*Benchmark rank does not change this decision.*

Nhận ra sớm giúp tiết kiệm nhiều tuần.  
*Early recognition saves many weeks.*

**Bước 2 — Lập danh sách ngắn gồm 2–3 model.**  
***Step 2: create a shortlist of 2–3 models.***

Bạn có thể sử dụng MTEB hoặc MMTEB.  
*You can use MTEB or MMTEB.*

Mọi model phải thoả mãn các ràng buộc.  
*Every model must satisfy the constraints.*

**Bước 3 — Đo trên dữ liệu thật.**  
***Step 3: measure performance on real data.***

Đây là bước quyết định.  
*This is the deciding step.*

Nó cũng thường bị bỏ qua nhất.  
*It is also the most frequently skipped step.*

Bạn cần xây dựng một tập vàng.  
*You need to build a gold set.*

Tập vàng chứa 50–200 truy vấn thật.  
*The gold set contains 50–200 real queries.*

Mỗi truy vấn có nhãn kết quả đúng.  
*Each query has labels for correct results.*

Nhãn chỉ ra tài liệu phải xuất hiện.  
*The labels identify documents that should appear.*

Bạn chạy từng model trên tập này.  
*You run each model on this set.*

Sau đó bạn đo các chỉ số phù hợp.  
*Then you measure suitable metrics.*

> **nDCG — normalized Discounted Cumulative Gain**  
> *nDCG: normalized Discounted Cumulative Gain*
>
> nDCG đo chất lượng xếp hạng.  
> *nDCG measures ranking quality.*
>
> Chỉ số này tính đến vị trí kết quả.  
> *This metric considers result position.*
>
> Kết quả đúng ở hạng 1 có giá trị cao hơn.  
> *A correct result at rank 1 has more value.*
>
> Cùng kết quả ở hạng 9 có giá trị thấp hơn.  
> *The same result at rank 9 has less value.*
>
> nDCG phản ánh sát trải nghiệm người dùng.  
> *nDCG closely reflects user experience.*
>
> **MRR — Mean Reciprocal Rank**  
> *MRR: Mean Reciprocal Rank*
>
> MRR là trung bình nghịch đảo của thứ hạng đúng đầu tiên.  
> *MRR is the average reciprocal rank of the first correct result.*
>
> Kết quả đúng luôn ở hạng 1 tạo MRR bằng 1.  
> *A correct result always at rank 1 produces an MRR of 1.*
>
> MRR phù hợp với bài toán cần một đáp án đúng.  
> *MRR suits tasks requiring one correct answer.*
>
> Hỏi đáp là một ví dụ.  
> *Question answering is one example.*
>
> Recall@k đã được giải thích tại Mục 3.3.  
> *Recall@k was explained in Section 3.3.*

**Bước 4 — Cân nhắc fine-tune cho lĩnh vực hẹp.**  
***Step 4: consider fine-tuning for a narrow domain.***

> **Fine-tune — tinh chỉnh**  
> *Fine-tune*
>
> Fine-tune bắt đầu từ một model đã huấn luyện.  
> *Fine-tuning starts from a pretrained model.*
>
> Model được huấn luyện thêm trên dữ liệu chuyên ngành.  
> *The model receives additional training on domain data.*
>
> Cách này rẻ hơn huấn luyện từ đầu.  
> *This method is much cheaper than training from scratch.*

Một số lĩnh vực có ngôn ngữ rất đặc thù.  
*Some domains use highly specialized language.*

Pháp lý là một ví dụ.  
*Law is one example.*

Y tế là một ví dụ khác.  
*Medicine is another example.*

Mã nguồn cũng là một lĩnh vực đặc thù.  
*Source code is also a specialized domain.*

Fine-tune thường cải thiện 10–30% trong các lĩnh vực này.  
*Fine-tuning often improves performance by 10–30% in these domains.*

Nó cũng tạo thêm một tầng vận hành.  
*It also creates another operational layer.*

Bạn phải lưu dữ liệu huấn luyện.  
*You must retain the training data.*

Bạn phải lặp lại quy trình khi đổi model nền.  
*You must repeat the process after changing the base model.*

Bạn phải tự chịu trách nhiệm về chất lượng.  
*You must take responsibility for quality.*

Bạn không nên bắt đầu bằng fine-tune.  
*You should not begin with fine-tuning.*

Bạn chỉ nên dùng nó khi model có sẵn không đủ tốt.  
*You should use it only when existing models are insufficient.*

💡 MTEB giúp thu hẹp danh sách ứng viên.  
*💡 MTEB narrows the candidate shortlist.*

Dữ liệu có nhãn của bạn tạo ra quyết định cuối cùng.  
*Your labeled data produces the final decision.*

Thứ hạng benchmark chỉ là giả định ban đầu.  
*Benchmark rank is only an initial prior.*

Nó không phải một phán quyết.  
*It is not a verdict.*

### 4.3. Đổi model — quyết định kiến trúc nặng ký nhất
*4.3. Changing models: the heaviest architectural decision*

Nhiều đội ngũ gặp khó khăn tại bước này.  
*Many teams struggle at this step.*

Đây cũng là chủ đề system design phổ biến.  
*This is also a common system design topic.*

Đổi embedding model yêu cầu tạo lại toàn bộ vector.  
*Changing the embedding model requires regenerating every vector.*

Mục 2.5 đã giải thích nguyên nhân.  
*Section 2.5 explained the reason.*

Vector cũ nằm trên một bản đồ.  
*Old vectors exist on one map.*

Vector mới nằm trên một bản đồ khác.  
*New vectors exist on another map.*

100 triệu tài liệu có thể cần nhiều ngày xử lý.  
*One hundred million documents may require many processing days.*

Chi phí có thể lên đến hàng nghìn USD.  
*The cost may reach thousands of US dollars.*

Hệ thống vẫn phải phục vụ người dùng trong thời gian đó.  
*The system must still serve users during that period.*

Chiến lược an toàn gồm bốn phần.  
*The safe strategy contains four parts.*

**Một — Đánh phiên bản cho cột dữ liệu.**  
***First: version the data columns.***

Bạn không nên ghi đè cột cũ.  
*You should not overwrite the old column.*

Bạn nên thêm một cột mới song song.  
*You should add a new parallel column.*

```sql
ALTER TABLE products ADD COLUMN embedding_v2 vector(1024);
-- Cột embedding_v1 vẫn nguyên vẹn và vẫn đang phục vụ người dùng.
-- The embedding_v1 column remains intact and continues serving users.
-- Nếu model mới hoá ra tệ hơn, bạn chỉ việc bỏ cột v2 đi. Không mất gì.
-- If the new model performs worse, you can drop v2 without losing anything.
```

**Hai — Backfill chạy nền.**  
***Second: run a background backfill.***

> **Backfill — nạp bù**  
> *Backfill*
>
> Backfill lấp dữ liệu cho các bản ghi đã tồn tại.  
> *A backfill fills data for existing records.*
>
> Nó chạy như một tiến trình nền.  
> *It runs as a background process.*
>
> Tiến trình phải chạy chậm rãi.  
> *The process must run gradually.*
>
> Tiến trình cũng phải có giới hạn tốc độ.  
> *The process must also have rate limits.*
>
> Nó không được tranh tài nguyên với production.  
> *It must not compete with production workloads.*

**Ba — Ghi kép trong giai đoạn chuyển tiếp.**  
***Third: use dual writes during the transition.***

Mọi tài liệu mới phải ghi vào cả hai cột.  
*Every new document must write to both columns.*

Quy tắc này áp dụng trong thời gian backfill.  
*This rule applies during the backfill.*

Thiếu dual-write tạo ra một vùng dữ liệu trống.  
*Missing dual writes creates a data gap.*

Vùng dữ liệu này rất khó phát hiện.  
*This data gap is very difficult to detect.*

**Bốn — Chuyển đổi theo kiểu blue-green.**  
***Fourth: perform a blue-green transition.***

> **Blue-green deployment — triển khai xanh dương và xanh lá**  
> *Blue-green deployment*
>
> Kỹ thuật này duy trì hai môi trường song song.  
> *This technique maintains two parallel environments.*
>
> Môi trường xanh dương đang phục vụ production.  
> *The blue environment serves production.*
>
> Môi trường xanh lá chứa phiên bản mới.  
> *The green environment contains the new version.*
>
> Bạn kiểm tra kỹ môi trường mới.  
> *You thoroughly test the new environment.*
>
> Sau đó bạn chuyển lưu lượng bằng một thao tác.  
> *Then you switch traffic with one operation.*
>
> Bạn có thể chuyển ngược lại ngay khi có lỗi.  
> *You can immediately switch back after a failure.*

Với embedding, bạn bắt đầu bằng 5–10% lưu lượng thật.  
*For embeddings, you start with 5–10% of real traffic.*

Phần lưu lượng này sử dụng cột v2.  
*This traffic portion uses the v2 column.*

Bạn đo chất lượng bằng dữ liệu thật.  
*You measure quality using real data.*

Tỉ lệ click là một chỉ số.  
*Click-through rate is one metric.*

Tỉ lệ tìm kiếm không có kết quả là chỉ số khác.  
*The zero-result rate is another metric.*

Sau đó bạn tăng lưu lượng dần lên 100%.  
*Then you gradually increase traffic to 100%.*

💡 Đổi model là một dự án tốn kém.  
*💡 Changing models is an expensive project.*

Bạn nên đầu tư nhiều thời gian ở giai đoạn thiết kế.  
*You should invest more time during the design phase.*

Một tuần đánh giá ban đầu thường rất rẻ.  
*One week of initial evaluation is usually inexpensive.*

Một dự án di trú kéo dài một tháng rất đắt.  
*A month-long migration project is very expensive.*

### 4.4. Chi phí, độ tin cậy, giám sát
*4.4. Cost, reliability, and monitoring*

#### Những thứ đẩy chi phí lên
*Factors that increase cost*

| Yếu tố / Factor | Cách kiểm soát / Control |
|---|---|
| Số token đã embed / Embedded token count | Chunk hợp lý. Loại bỏ menu, footer và boilerplate. / Use sensible chunks. Remove menus, footers, and boilerplate. |
| Số lần embed lại / Re-embedding count | Chốt model sớm. Đánh phiên bản cẩn thận. / Select the model early. Version it carefully. |
| Số chiều vector / Vector dimensions | Dùng Matryoshka khi model hỗ trợ. / Use Matryoshka when supported. |
| Truy vấn lặp / Repeated queries | Cache theo hash. / Cache by hash. |
| Token tiếng Việt / Vietnamese tokens | Chọn model có tokenizer tốt cho tiếng Việt. / Choose a model with a strong Vietnamese tokenizer. |

#### Độ tin cậy
*Reliability*

> **Reliability — độ tin cậy**  
> *Reliability*
>
> Reliability là khả năng hệ thống tiếp tục hoạt động đúng.  
> *Reliability is the system's ability to keep working correctly.*
>
> Khả năng này phải tồn tại cả khi có sự cố.  
> *This ability must persist during failures.*
>
> **Rate limit — giới hạn tốc độ**  
> *Rate limit*
>
> Rate limit giới hạn số lượt gọi API.  
> *A rate limit restricts the number of API calls.*
>
> Giới hạn áp dụng trong một khoảng thời gian.  
> *The limit applies within a time window.*
>
> Nhà cung cấp áp dụng giới hạn này.  
> *The provider enforces this limit.*
>
> Vượt giới hạn làm yêu cầu bị từ chối.  
> *Exceeding the limit causes request rejection.*
>
> **Retry with backoff — thử lại có giãn cách**  
> *Retry with backoff*
>
> Hệ thống thử lại sau một lỗi tạm thời.  
> *The system retries after a temporary failure.*
>
> Thời gian chờ tăng sau mỗi lần thất bại.  
> *The waiting time increases after each failure.*
>
> Ví dụ là 1, 2, 4 và 8 giây.  
> *Examples are 1, 2, 4, and 8 seconds.*
>
> Thử lại liên tục có thể làm hệ thống quá tải hơn.  
> *Immediate repeated retries can worsen system overload.*

```python
import time, random

def embed_with_retry(text, max_retries=5):
    for attempt in range(max_retries):
        try:
            return client.embeddings.create(model=MODEL, input=text)
        except RateLimitError:
            # Chờ tăng theo cấp số nhân: 1s, 2s, 4s, 8s, 16s
            # Wait with exponential growth: 1s, 2s, 4s, 8s, 16s
            # Cộng thêm một chút ngẫu nhiên ("jitter") để tránh việc
            # Add a small random amount called jitter to prevent
            # hàng nghìn tiến trình cùng thử lại đúng một thời điểm.
            # thousands of processes from retrying at the same moment.
            wait = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait)
    raise Exception("Đã hết số lần thử lại")
```

Đường nóng cần một phương án dự phòng.  
*The hot path needs a fallback.*

API embedding có thể ngừng hoạt động.  
*The embedding API may become unavailable.*

Hệ thống tìm kiếm không nên chết theo.  
*The search system should not fail with it.*

Bạn có thể giữ một model nhỏ self-host.  
*You can maintain a small self-hosted model.*

Model đó chỉ được dùng tạm thời.  
*That model is used only temporarily.*

Một lựa chọn khác là full-text search.  
*Another option is full-text search.*

Chất lượng kết quả sẽ kém hơn.  
*Result quality will be lower.*

Hệ thống vẫn còn sử dụng được.  
*The system remains usable.*

Điều đó tốt hơn một trang lỗi.  
*That is better than an error page.*

#### Giám sát — bốn chỉ số phải theo dõi
*Monitoring: four required metrics*

> **Monitoring — giám sát**  
> *Monitoring*
>
> Monitoring thu thập số liệu từ hệ thống đang chạy.  
> *Monitoring collects metrics from a running system.*
>
> Số liệu giúp phát hiện vấn đề sớm.  
> *Metrics help detect problems early.*
>
> Mục tiêu là phát hiện trước người dùng.  
> *The goal is detection before users notice.*

| Chỉ số / Metric | Lý do / Reason |
|---|---|
| Tỉ lệ input bị truncation / Input truncation rate | Phát hiện việc cắt cụt âm thầm. / Detect silent truncation. |
| Tỉ lệ encode lỗi hoặc vector rỗng / Encode failure or empty-vector rate | Phát hiện dữ liệu hỏng trong kho. / Detect corrupted stored data. |
| Recall@k định kỳ trên tập vàng / Periodic Recall@k on the gold set | Phát hiện chất lượng search suy giảm. / Detect declining search quality. |
| Chi phí token hằng ngày / Daily token cost | Phát hiện rò rỉ chi phí sớm. / Detect cost leakage early. |

> **Drift — trôi dạt**  
> *Drift*
>
> Drift là hiện tượng chất lượng model giảm theo thời gian.  
> *Drift is a gradual decline in model quality over time.*
>
> Model có thể không thay đổi.  
> *The model may remain unchanged.*
>
> Dữ liệu vẫn có thể thay đổi.  
> *The data may still change.*
>
> Cách người dùng gõ cũng có thể thay đổi.  
> *User query behavior may also change.*
>
> Một sàn thương mại có thể thêm ngành hàng mới.  
> *An e-commerce platform may add a new category.*
>
> Ngành hàng mới tạo ra thuật ngữ mới.  
> *The new category creates new terminology.*
>
> Model có thể chưa gặp các thuật ngữ đó.  
> *The model may not have seen those terms.*
>
> Bạn phải đo recall định kỳ trên tập vàng.  
> *You must measure recall periodically on the gold set.*
>
> Một lần đo lúc ra mắt là chưa đủ.  
> *One launch-time measurement is insufficient.*

#### Các kiểu hỏng — xếp theo mức độ nguy hiểm
*Failure modes ordered by danger*

**1. Nhà cung cấp âm thầm đổi phiên bản model.**  
***1. The provider silently changes the model version.***

⚠️ Đây là kiểu hỏng nguy hiểm nhất.  
*⚠️ This is the most dangerous failure mode.*

Bạn gọi API bằng tên `"text-embedding-3-small"`.  
*You call the API with the name `"text-embedding-3-small"`.*

Nhà cung cấp có thể cập nhật model đứng sau tên đó.  
*The provider may update the model behind that name.*

Từ thời điểm đó, model tạo ra vector mới.  
*From that moment, the model creates new vectors.*

Vector mới không còn chung không gian với vector cũ.  
*The new vectors no longer share the old vector space.*

Vector cũ vẫn nằm trong database.  
*The old vectors remain in the database.*

Nhà cung cấp có thể không gửi thông báo.  
*The provider may send no notice.*

Hệ thống cũng không báo lỗi.  
*The system also reports no error.*

Chất lượng tìm kiếm chỉ giảm dần.  
*Search quality simply declines over time.*

Bạn có thể không hiểu nguyên nhân.  
*You may not understand the cause.*

Bạn nên ghim phiên bản model rõ ràng.  
*You should pin the model version explicitly.*

Cách này chỉ áp dụng khi nhà cung cấp hỗ trợ.  
*This method applies only when the provider supports it.*

Bạn phải lưu tên model cạnh mỗi vector.  
*You must store the model name beside every vector.*

Bạn cũng phải lưu phiên bản model.  
*You must also store the model version.*

Bạn phải giám sát recall trên tập vàng.  
*You must monitor recall on the gold set.*

Việc giám sát giúp phát hiện sự cố này.  
*Monitoring helps detect this failure.*

**2. Tài liệu dài bị cắt cụt.**  
***2. Long documents are truncated.***

Mục 3.6 đã giải thích vấn đề này.  
*Section 3.6 explained this problem.*

Chunking là biện pháp phòng ngừa đầu tiên.  
*Chunking is the first preventive measure.*

Giám sát tỉ lệ truncation là biện pháp thứ hai.  
*Monitoring the truncation rate is the second measure.*

**3. Dùng model sai ngôn ngữ.**  
***3. Using a model for the wrong language.***

Bạn nên tự kiểm tra bằng các cặp câu tiếng Việt.  
*You should test with Vietnamese sentence pairs.*

Các cặp câu phải thực sự đồng nghĩa.  
*The sentence pairs must be truly synonymous.*

**4. Trộn vector chuẩn hoá và chưa chuẩn hoá.**  
***4. Mixing normalized and unnormalized vectors.***

Bạn phải chuẩn hoá tại một vị trí tập trung.  
*You must normalize in one centralized location.*

### 4.5. Nói chuyện với người không rành kỹ thuật
*4.5. Communicating with non-technical people*

Kỹ năng này phân biệt staff engineer với senior engineer giỏi.  
*This skill distinguishes a staff engineer from a strong senior engineer.*

Bạn phải giải thích được quyết định cho quản lý sản phẩm.  
*You must explain decisions to product managers.*

Bạn cũng phải giải thích được cho giám đốc.  
*You must also explain them to executives.*

Thiếu khả năng này sẽ làm bạn mất quyền quyết định.  
*Lacking this ability will cost you decision authority.*

Nguyên tắc là loại bỏ thuật ngữ không cần thiết.  
*The principle is to remove unnecessary terminology.*

Bạn nên quy vấn đề về ba yếu tố.  
*You should reduce the issue to three factors.*

Yếu tố đầu tiên là tiền.  
*The first factor is money.*

Yếu tố thứ hai là rủi ro.  
*The second factor is risk.*

Yếu tố thứ ba là tốc độ ra sản phẩm.  
*The third factor is time to market.*

Ví dụ sau có thể được dùng gần như nguyên văn.  
*The following example can be used almost verbatim.*

> Embedding giúp ô tìm kiếm hiểu ý khách hàng.  
> *Embedding helps the search box understand customer intent.*
>
> Nó không chỉ so khớp mặt chữ.  
> *It does not merely match surface text.*
>
> Khách có thể gõ “áo phông”.  
> *A customer may type “áo phông”.*
>
> Hệ thống vẫn tìm được “áo thun”.  
> *The system can still find “áo thun”.*
>
> Việc chọn công cụ tạo ra ba trade-off.  
> *Choosing the tool creates three trade-offs.*
>
> Trade-off đầu tiên liên quan đến chi phí.  
> *The first trade-off concerns cost.*
>
> Dịch vụ bên ngoài tính tiền theo lượng dùng.  
> *An external service charges by usage.*
>
> Tự vận hành cần đầu tư máy chủ ban đầu.  
> *Self-hosting requires initial server investment.*
>
> Chi phí biên về sau có thể thấp hơn.  
> *Marginal cost may become lower later.*
>
> Trade-off thứ hai liên quan đến quyền riêng tư.  
> *The second trade-off concerns privacy.*
>
> Dịch vụ ngoài nhận dữ liệu của khách hàng.  
> *An external service receives customer data.*
>
> Dữ liệu được gửi tới máy chủ công ty khác.  
> *The data is sent to another company's servers.*
>
> Dữ liệu nhạy cảm có thể cấm cách làm này.  
> *Sensitive data may prohibit this approach.*
>
> Trade-off thứ ba liên quan đến tốc độ ra mắt.  
> *The third trade-off concerns launch speed.*
>
> Dịch vụ ngoài có thể chạy trong một tuần.  
> *An external service may be ready within one week.*
>
> Tự vận hành có thể cần khoảng một tháng.  
> *Self-hosting may require about one month.*
>
> Một điểm khác cần được biết trước.  
> *Another point must be understood in advance.*
>
> Đổi lựa chọn sau này rất tốn kém.  
> *Changing the choice later is very expensive.*
>
> Hệ thống phải xử lý lại toàn bộ dữ liệu.  
> *The system must process all data again.*
>
> Quy mô hiện tại có thể cần khoảng hai tuần công.  
> *The current scale may require about two weeks of work.*
>
> Chi phí có thể lên đến vài nghìn USD.  
> *The cost may reach several thousand US dollars.*
>
> Tôi đề nghị thêm một tuần đánh giá ngay bây giờ.  
> *I recommend adding one week of evaluation now.*
>
> Cách này rẻ hơn việc làm lại sau.  
> *This approach is cheaper than rebuilding later.*

Đoạn giải thích trên không dùng thuật ngữ chưa được giải thích.  
*The explanation above uses no unexplained terminology.*

Nó không nhắc đến vector.  
*It does not mention vectors.*

Nó không nhắc đến cosine.  
*It does not mention cosine.*

Nó cũng không nhắc đến MTEB.  
*It also does not mention MTEB.*

Đoạn đó vẫn truyền tải đủ ba trade-off.  
*The passage still communicates all three trade-offs.*

Nó cũng nêu một cảnh báo rủi ro quan trọng.  
*It also states one important risk warning.*

Về mặt tổ chức, staff engineer nên chủ động nêu hai điểm.  
*Organizationally, a staff engineer should proactively raise two points.*

**Một là quyền riêng tư đôi khi quan trọng hơn benchmark.**  
***First, privacy sometimes matters more than benchmarks.***

Một công ty có thể xử lý dữ liệu y tế.  
*A company may process medical data.*

Một công ty khác có thể xử lý dữ liệu tài chính.  
*Another company may process financial data.*

Dữ liệu cá nhân cũng có thể thuộc phạm vi GDPR.  
*Personal data may also fall under GDPR.*

Việc gửi dữ liệu tới API bên thứ ba có thể vi phạm quy định.  
*Sending data to a third-party API may violate regulations.*

Khi đó công ty bắt buộc phải self-host.  
*The company must then self-host.*

Model self-host có thể yếu hơn một chút.  
*The self-hosted model may be slightly weaker.*

Ràng buộc pháp lý vẫn quan trọng hơn benchmark.  
*The legal constraint still matters more than benchmark rank.*

Bạn phải nêu ràng buộc này từ cuộc họp đầu tiên.  
*You must raise this constraint in the first meeting.*

Bạn không nên chờ đến lúc hệ thống đã hoàn thành.  
*You should not wait until the system is complete.*

> **GDPR — luật bảo vệ dữ liệu cá nhân của châu Âu**  
> *GDPR: the European personal data protection law*
>
> GDPR quy định chặt chẽ việc xử lý dữ liệu cá nhân.  
> *GDPR tightly regulates personal data processing.*
>
> GDPR cũng quy định việc chuyển giao dữ liệu.  
> *GDPR also regulates data transfers.*
>
> Nhiều quốc gia có luật tương tự.  
> *Many countries have similar laws.*
>
> Việt Nam có quy định về bảo vệ dữ liệu cá nhân.  
> *Vietnam has personal data protection regulations.*

**Hai là pipeline embedding nên trở thành hạ tầng dùng chung.**  
***Second, the embedding pipeline should become shared infrastructure.***

Ba team có thể cùng cần embedding.  
*Three teams may all need embeddings.*

Bạn không nên để mỗi team tự viết một pipeline.  
*You should not let each team build its own pipeline.*

Công ty sẽ nhanh chóng có ba model khác nhau.  
*The company will quickly have three different models.*

Công ty cũng có ba chiến lược chunking khác nhau.  
*The company will also have three different chunking strategies.*

Ba hoá đơn riêng biệt cũng sẽ xuất hiện.  
*Three separate bills will also appear.*

Kết quả giữa các team sẽ không so sánh được.  
*Results across teams will not be comparable.*

Bạn nên xây một dịch vụ nội bộ dùng chung.  
*You should build one shared internal service.*

Dịch vụ phải có phiên bản rõ ràng.  
*The service must have explicit versioning.*

Dịch vụ nên có cache chung.  
*The service should have a shared cache.*

Dịch vụ cũng cần hệ thống monitoring chung.  
*The service also needs shared monitoring.*

### 4.6. Câu hỏi system design mẫu và hướng trả lời
*4.6. Sample system design question and answer framework*

> **Đề bài**  
> *Prompt*
>
> Hãy thiết kế một hệ thống tạo embedding.  
> *Design a system that generates embeddings.*
>
> Hệ thống cũng phải phục vụ semantic search.  
> *The system must also serve semantic search.*
>
> Kho dữ liệu chứa 50 triệu tài liệu.  
> *The corpus contains 50 million documents.*
>
> Tài liệu mới được thêm theo thời gian thực.  
> *New documents arrive in real time.*
>
> Nội dung gồm tiếng Anh và tiếng Việt.  
> *The content includes English and Vietnamese.*
>
> Dữ liệu là hồ sơ khách hàng.  
> *The data consists of customer records.*
>
> Dữ liệu có tính nhạy cảm.  
> *The data is sensitive.*

Một staff engineer nên trả lời theo một khung rõ ràng.  
*A staff engineer should answer with a clear framework.*

Khung bắt đầu bằng việc làm rõ đề.  
*The framework starts with clarifying the problem.*

Sau đó bạn chọn giải pháp.  
*Then you choose a solution.*

Tiếp theo bạn thiết kế kiến trúc.  
*Next you design the architecture.*

Bạn phải tính quy mô.  
*You must calculate scale.*

Cuối cùng bạn phân tích rủi ro.  
*Finally you analyze risk.*

Mỗi bước cần nêu rõ trade-off.  
*Each step should state its trade-off clearly.*

**Bước 1 — Làm rõ đề trước khi vẽ kiến trúc.**  
***Step 1: clarify the problem before drawing the architecture.***

Ứng viên thường bỏ qua bước này.  
*Candidates often skip this step.*

Việc bỏ qua gây mất điểm lớn.  
*Skipping it causes a major score loss.*

Bạn cần biết số truy vấn mỗi giây lúc cao điểm.  
*You need the peak queries per second.*

Bạn cần biết tốc độ tài liệu mới xuất hiện.  
*You need the new-document arrival rate.*

Tốc độ có thể là hàng trăm tài liệu mỗi ngày.  
*The rate may be hundreds of documents per day.*

Tốc độ cũng có thể là hàng triệu tài liệu mỗi ngày.  
*The rate may also be millions of documents per day.*

Bạn cần biết độ dài trung bình của tài liệu.  
*You need the average document length.*

Bạn cần biết mục tiêu chất lượng.  
*You need the quality target.*

Recall@10 có thể là chỉ số mục tiêu.  
*Recall@10 may be the target metric.*

Tỉ lệ click cũng có thể là chỉ số mục tiêu.  
*Click-through rate may also be the target metric.*

Bạn phải làm rõ ràng buộc pháp lý.  
*You must clarify legal constraints.*

Bạn phải biết dữ liệu có được ra ngoài lãnh thổ hay không.  
*You must know whether data may leave the territory.*

**Bước 2 — Chọn model và giải thích lý do.**  
***Step 2: choose the model and explain why.***

Dữ liệu nhạy cảm tạo ra một ràng buộc cứng.  
*Sensitive data creates a hard constraint.*

Hệ thống cũng cần hỗ trợ tiếng Việt.  
*The system also needs Vietnamese support.*

Hai điều kiện hướng tới model đa ngôn ngữ self-host.  
*These two conditions point to a self-hosted multilingual model.*

Qwen3-Embedding là một ví dụ.  
*Qwen3-Embedding is one example.*

BGE-M3 là một lựa chọn khác.  
*BGE-M3 is another option.*

API bên ngoài không phải lựa chọn mặc định.  
*An external API is not the default choice.*

Quyền riêng tư là yếu tố quyết định.  
*Privacy is the deciding factor.*

Điểm benchmark không phải yếu tố quyết định.  
*Benchmark score is not the deciding factor.*

Bạn có thể chấp nhận model yếu hơn vài điểm.  
*You may accept a model that scores a few points lower.*

Đổi lại, dữ liệu không rời hạ tầng công ty.  
*In return, data never leaves company infrastructure.*

Bạn vẫn phải đo hai hoặc ba model ứng viên.  
*You must still evaluate two or three candidate models.*

Việc đo phải dùng truy vấn thật.  
*The evaluation must use real queries.*

**Bước 3 — Thiết kế đường nạp dữ liệu.**  
***Step 3: design the ingestion path.***

Đây là đường nguội.  
*This is the cold path.*

```text
Tài liệu mới
New document
    ↓
Hàng đợi
Queue
    ↓
Worker chunking
Chunking worker
    ↓
Worker embedding trên GPU theo batch
GPU embedding worker with batching
    ↓
Normalize
Normalize
    ↓
PostgreSQL + pgvector
PostgreSQL + pgvector
    ↓
Vector + metadata
Vector + metadata
```

> **Worker — tiến trình thợ**  
> *Worker*
>
> Worker là một chương trình chạy nền.  
> *A worker is a background program.*
>
> Worker liên tục lấy việc từ queue.  
> *A worker continuously takes jobs from the queue.*
>
> Worker xử lý từng công việc.  
> *The worker processes each job.*
>
> Bạn có thể tăng gấp đôi số worker.  
> *You can double the number of workers.*
>
> Throughput khi đó có thể tăng gần gấp đôi.  
> *Throughput may then nearly double.*
>
> Đây là scale ngang trong thực tế.  
> *This is horizontal scaling in practice.*

Queue giúp hệ thống chịu được tải đột biến.  
*A queue helps the system absorb traffic spikes.*

Khách hàng có thể nhập 100.000 tài liệu cùng lúc.  
*A customer may upload 100,000 documents at once.*

Các tài liệu sẽ xếp hàng chờ.  
*The documents will wait in the queue.*

Cụm GPU không bị sập vì tải tức thời.  
*The GPU cluster avoids crashing from the immediate load.*

Chunking phải đứng trước embedding.  
*Chunking must precede embedding.*

Thứ tự này ngăn truncation âm thầm.  
*This order prevents silent truncation.*

Mục 3.6 đã phân tích vấn đề đó.  
*Section 3.6 analyzed that problem.*

**Bước 4 — Thiết kế đường truy vấn.**  
***Step 4: design the query path.***

Đây là đường nóng.  
*This is the hot path.*

```text
Người dùng gõ truy vấn
The user enters a query
    ↓
Kiểm tra cache bằng hash của truy vấn
Check the cache using the query hash
    ↓
Cache miss
Cache miss
    ↓
Embed bằng model self-host khoảng 10ms
Embed with a self-hosted model in about 10ms
    ↓
Tìm bằng HNSW trong pgvector
Search with HNSW in pgvector
    ↓
Tuỳ chọn: kết hợp full-text search
Optional: combine full-text search
    ↓
Trả kết quả
Return results
```

Self-host phù hợp với đường truy vấn vì latency.  
*Self-hosting suits the query path because of latency.*

API có thể mất 50–200ms cho phần mạng.  
*An API may spend 50–200ms on network travel.*

Người dùng có thể cảm nhận độ trễ đó.  
*Users may perceive that delay.*

Model nội bộ trên GPU có thể mất 5–15ms.  
*An internal GPU model may take 5–15ms.*

Cache phù hợp vì truy vấn thường lặp lại.  
*Caching fits because queries often repeat.*

Một số từ khoá phổ biến chiếm phần lớn lưu lượng.  
*A few popular keywords account for most traffic.*

Embedding mạnh về tìm kiếm theo nghĩa.  
*Embeddings are strong at semantic retrieval.*

Embedding có thể yếu với mã sản phẩm.  
*Embeddings may be weak with product codes.*

Embedding cũng có thể yếu với tên riêng.  
*Embeddings may also be weak with proper names.*

Số hiệu là một điểm yếu khác.  
*Serial numbers are another weakness.*

Full-text search xử lý tốt các trường hợp chính xác này.  
*Full-text search handles these exact cases well.*

Việc kết hợp hai cách gọi là hybrid search.  
*Combining both methods is called hybrid search.*

Hybrid search thường tốt hơn từng cách riêng lẻ.  
*Hybrid search is often better than either method alone.*

**Bước 5 — Tính quy mô và chi phí.**  
***Step 5: calculate scale and cost.***

Kho có 50 triệu tài liệu.  
*The corpus contains 50 million documents.*

Mỗi tài liệu tạo trung bình 1,5 chunk.  
*Each document creates an average of 1.5 chunks.*

Tổng số là khoảng 75 triệu vector.  
*The total is about 75 million vectors.*

Vector 1024 chiều chiếm khoảng 4 KB.  
*A 1024-dimensional vector uses about 4 KB.*

75 triệu vector cần khoảng 300 GB.  
*Seventy-five million vectors require about 300 GB.*

Dung lượng này không vừa RAM máy chủ thông thường.  
*This size does not fit in ordinary server RAM.*

Có hai hướng xử lý chính.  
*There are two main approaches.*

Hướng đầu tiên dùng Matryoshka.  
*The first approach uses Matryoshka.*

Bạn có thể giảm xuống 512 chiều.  
*You may reduce the vectors to 512 dimensions.*

Bạn cũng có thể giảm xuống 256 chiều.  
*You may also reduce them to 256 dimensions.*

Dung lượng khi đó còn khoảng 75–150 GB.  
*Storage then drops to about 75–150 GB.*

Độ chính xác có thể giảm 2–3%.  
*Accuracy may decrease by 2–3%.*

Hướng thứ hai sử dụng nhiều máy.  
*The second approach uses multiple machines.*

Cách này giữ nguyên số chiều.  
*This method preserves the original dimensions.*

Chất lượng được giữ nguyên.  
*Quality remains unchanged.*

Độ phức tạp vận hành sẽ tăng.  
*Operational complexity will increase.*

Bạn phải đo trước khi chọn.  
*You must measure before choosing.*

Bài toán có thể chấp nhận mất 2–3% recall.  
*The use case may tolerate a 2–3% recall loss.*

Khi đó Matryoshka là phương án đơn giản hơn.  
*Matryoshka is then the simpler option.*

**Bước 6 — Thiết kế phiên bản và kế hoạch di trú.**  
***Step 6: design versioning and migration plans.***

Cột vector phải có phiên bản từ ngày đầu.  
*The vector column must be versioned from day one.*

Tên model phải được lưu cạnh mỗi bản ghi.  
*The model name must be stored beside each record.*

Quy trình backfill phải được chuẩn bị sẵn.  
*The backfill process must be prepared in advance.*

Bạn có thể chưa định đổi model ngay.  
*You may not plan to change the model immediately.*

Một ngày nào đó việc đổi model vẫn sẽ xảy ra.  
*One day, a model change will still occur.*

Chuẩn bị trước rẻ hơn chữa cháy.  
*Preparation is cheaper than emergency repair.*

**Bước 7 — Thiết kế monitoring.**  
***Step 7: design monitoring.***

Bạn phải theo dõi tỉ lệ tài liệu bị truncation.  
*You must monitor the document truncation rate.*

Bạn phải theo dõi tỉ lệ lỗi encode.  
*You must monitor the encoding error rate.*

Bạn nên đo recall@10 hằng tuần.  
*You should measure Recall@10 weekly.*

Phép đo phải sử dụng tập vàng.  
*The measurement must use the gold set.*

Bạn phải theo dõi latency đường nóng ở p95.  
*You must monitor hot-path latency at p95.*

Bạn cũng phải theo dõi chi phí GPU mỗi ngày.  
*You must also monitor daily GPU cost.*

**Bước 8 — Nêu điều kiện làm thay đổi quyết định.**  
***Step 8: state the condition that changes the decision.***

Ràng buộc quyền riêng tư có thể không tồn tại.  
*The privacy constraint may not exist.*

Đội ngũ nhỏ có thể không muốn vận hành GPU.  
*A small team may not want to operate GPUs.*

Khi đó API đơn giản hơn nhiều.  
*An API is then much simpler.*

Bạn có thể chọn API trong trường hợp đó.  
*You may choose an API in that case.*

Quyền riêng tư là yếu tố quyết định của bài này.  
*Privacy is the deciding factor in this problem.*

Quyết định phải thay đổi khi yếu tố đó thay đổi.  
*The decision must change when that factor changes.*

💡 Bước 8 thể hiện tư duy cấp staff.  
*💡 Step 8 demonstrates staff-level thinking.*

Bạn nêu rõ điều kiện khiến mình đổi ý.  
*You state the condition that would change your mind.*

Điều này chứng minh bạn hiểu cơ sở của quyết định.  
*This proves that you understand the basis of the decision.*

Bạn không chỉ học thuộc một kiến trúc.  
*You are not merely memorizing one architecture.*

Người phỏng vấn thường tìm kiếm chính năng lực này.  
*Interviewers often look for exactly this ability.*

### ✅ Self-check Phần 4
*✅ Part 4 self-check*

**Câu 1.**  
***Question 1.***

Bottleneck của hệ thống embedding nằm ở đâu?  
*Where is the bottleneck in an embedding system?*

Hãy nêu hai kỹ thuật quan trọng nhất.  
*Name the two most important techniques.*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Bottleneck không nằm ở phép tính cosine.  
*The bottleneck does not lie in cosine computation.*

Nó nằm ở khâu sản xuất vector.  
*It lies in vector production.*

Thời gian chạy model là một phần.  
*Model execution time is one part.*

Chi phí API là một phần khác.  
*API cost is another part.*

Embedding truy vấn thời gian thực cũng tạo latency.  
*Real-time query embedding also creates latency.*

Batching là kỹ thuật quan trọng đầu tiên.  
*Batching is the first important technique.*

Nó gom nhiều câu vào một lượt gọi.  
*It groups many sentences into one call.*

Nó tận dụng khả năng song song của GPU.  
*It uses GPU parallelism.*

Caching là kỹ thuật quan trọng thứ hai.  
*Caching is the second important technique.*

Embedding là một hàm xác định.  
*Embedding is a deterministic function.*

Cùng input luôn tạo cùng output.  
*The same input always creates the same output.*

Tính lại là lãng phí.  
*Recomputation is wasteful.*

</details>

**Câu 2.**  
***Question 2.***

Tại sao không nên chọn model đứng đầu MTEB?  
*Why should you not simply choose the top MTEB model?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Có ba lý do chính.  
*There are three main reasons.*

Nhà cung cấp tự nộp điểm số.  
*Providers submit their own scores.*

Không có kiểm chứng độc lập cho mọi kết quả.  
*There is no independent verification for every result.*

Benchmark dùng dữ liệu chung.  
*Benchmarks use general data.*

Benchmark không dùng dữ liệu của bạn.  
*Benchmarks do not use your data.*

Model tốt cho tiếng Anh có thể kém cho tiếng Việt.  
*A model strong in English may perform poorly in Vietnamese.*

Ràng buộc cứng có thể loại model đứng đầu.  
*Hard constraints may eliminate the top model.*

Quyền riêng tư là một ràng buộc.  
*Privacy is one constraint.*

Latency là một ràng buộc khác.  
*Latency is another constraint.*

Ngân sách cũng có thể quyết định.  
*Budget may also decide.*

Quy trình đúng bắt đầu bằng ràng buộc.  
*The correct process starts with constraints.*

Sau đó bạn lập danh sách ngắn bằng MTEB.  
*Then you create a shortlist using MTEB.*

Cuối cùng bạn đo trên truy vấn thật có nhãn.  
*Finally you evaluate on real labeled queries.*

</details>

**Câu 3.**  
***Question 3.***

Kiểu hỏng nào nguy hiểm nhất trong hệ thống embedding?  
*Which failure mode is most dangerous in an embedding system?*

Cách phòng là gì?  
*What is the prevention method?*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Nhà cung cấp có thể âm thầm đổi model.  
*The provider may silently change the model.*

Tên API có thể vẫn giữ nguyên.  
*The API name may remain unchanged.*

Vector mới sẽ nằm trong không gian khác.  
*New vectors will exist in another space.*

Vector cũ vẫn nằm trong database.  
*Old vectors will remain in the database.*

Chất lượng tìm kiếm giảm dần.  
*Search quality will gradually decline.*

Hệ thống có thể không báo lỗi.  
*The system may report no error.*

Bạn phải ghim phiên bản model.  
*You must pin the model version.*

Bạn phải lưu tên cùng phiên bản cạnh vector.  
*You must store the name and version beside the vector.*

Bạn cũng phải đo recall@k định kỳ.  
*You must also measure Recall@k periodically.*

</details>

**Câu 4.**  
***Question 4.***

Hãy giải thích chi phí đổi model bằng ba câu.  
*Explain the cost of changing models in three sentences.*

Đối tượng nghe là một giám đốc không rành kỹ thuật.  
*The audience is a non-technical executive.*

<details>
<summary>Gợi ý đáp án / Suggested answer</summary>

Mỗi công cụ mã hoá ý nghĩa theo một cách riêng.  
*Each tool encodes meaning in its own way.*

Hai cách giống hai hệ tốc ký khác nhau.  
*The two methods resemble different shorthand systems.*

Ghi chú của hệ này không dùng được trong hệ kia.  
*Notes from one system do not work in the other.*

Đổi công cụ yêu cầu xử lý lại toàn bộ dữ liệu.  
*Changing tools requires processing all data again.*

Quy mô hiện tại có thể cần X ngày công.  
*The current scale may require X workdays.*

Chi phí có thể là Y USD.  
*The cost may be Y US dollars.*

Một tuần đánh giá thêm hôm nay sẽ rẻ hơn.  
*One extra evaluation week today will be cheaper.*

Cách đó tránh việc làm lại sau sáu tháng.  
*That approach avoids rebuilding after six months.*

</details>

---

## Phần 5 — 🎯 CHỐT LẠI ĐỂ ĐI PHỎNG VẤN
*Part 5: 🎯 Final interview review*

> Phần này dùng để ôn nhanh trong 20 phút.  
> *This section supports a quick 20-minute review.*
>
> Bạn nên đọc nó trước buổi phỏng vấn.  
> *You should read it before an interview.*
>
> Bốn phần trước đã giải thích chi tiết.  
> *The previous four parts provided detailed explanations.*
>
> Phần này chỉ đóng vai trò gợi nhớ.  
> *This section serves only as a memory aid.*

### 5.1. Bảng thuật ngữ bắt buộc nhớ
*5.1. Required terminology table*

Bảng giữ nguyên thuật ngữ tiếng Anh.  
*The table preserves the English terminology.*

Phỏng vấn kỹ thuật thường sử dụng tiếng Anh.  
*Technical interviews commonly use English.*

| Thuật ngữ / Term | Định nghĩa / Definition |
|---|---|
| **Embedding** | Dãy số nhiều chiều mã hoá ý nghĩa. Nội dung giống nghĩa tạo vector gần nhau. / A multidimensional number sequence that encodes meaning. Similar meanings create nearby vectors. |
| **Dimension** | Số phần tử trong vector. MiniLM có 384. USE có 512. OpenAI có 1536 hoặc 3072. / The number of elements in a vector. MiniLM has 384. USE has 512. OpenAI has 1536 or 3072. |
| **Static embedding** | Một vector cố định cho mỗi từ. Word2Vec và GloVe là ví dụ. / One fixed vector per word. Word2Vec and GloVe are examples. |
| **Contextual embedding** | Vector phụ thuộc ngữ cảnh. Transformer tạo vector khác nhau cho hai nghĩa của “bank”. / A context-dependent vector. A transformer creates different vectors for two meanings of “bank.” |
| **Transformer / self-attention** | Kiến trúc cho phép mỗi token quan sát token khác. / An architecture that lets each token observe other tokens. |
| **Token / tokenizer** | Token là đơn vị model xử lý. Tokenizer cắt văn bản thành token. / A token is a unit processed by the model. A tokenizer splits text into tokens. |
| **Pooling** | Phép gộp nhiều vector token thành một vector câu. Mean pooling dùng trung bình cộng. / The operation that combines token vectors into one sentence vector. Mean pooling uses the arithmetic mean. |
| **Normalize (L2)** | Phép đưa vector về độ dài 1. Sau đó dot product tương đương cosine. / The operation that gives a vector unit length. Dot product then equals cosine. |
| **Dot product** | Công thức `Σ aᵢbᵢ`. Nó nhân từng cặp rồi cộng lại. / The formula `Σ aᵢbᵢ`. It multiplies matching pairs and sums them. |
| **Magnitude / norm** | Công thức `sqrt(Σ aᵢ²)`. Nó đo độ dài vector. / The formula `sqrt(Σ aᵢ²)`. It measures vector magnitude. |
| **Cosine similarity** | Công thức `(A·B)/(\|A\|\|B\|)`. Nó đo góc. Miền là `[−1,1]`. / The formula `(A·B)/(\|A\|\|B\|)`. It measures angle. Its range is `[−1,1]`. |
| **Cosine distance** | Công thức `1 − cosine_similarity`. Miền là `[0,2]`. Điểm nhỏ hơn nghĩa là gần hơn. / The formula `1 − cosine_similarity`. Its range is `[0,2]`. A smaller score means greater similarity. |
| **Euclidean distance** | Công thức `sqrt(Σ(aᵢ−bᵢ)²)`. Nó đo khoảng cách đường thẳng. / The formula `sqrt(Σ(aᵢ−bᵢ)²)`. It measures straight-line distance. |
| **Feature-extraction pipeline** | Tên tác vụ tạo embedding trong Transformers.js. / The embedding task name in Transformers.js. |
| **Transformers.js** | Thư viện chạy Hugging Face model bằng JavaScript. Gói mới là `@huggingface/transformers`. / A library that runs Hugging Face models with JavaScript. The new package is `@huggingface/transformers`. |
| **Universal Sentence Encoder** | Model Google 512 chiều. Model này xuất hiện trong video gốc. Nó đã cũ. / A 512-dimensional Google model. It appears in the original video. It is now old. |
| **MTEB / MMTEB** | Benchmark cho embedding model. MTEB có 56 tập tiếng Anh. MMTEB có 131 nhiệm vụ đa ngôn ngữ. / Benchmarks for embedding models. MTEB has 56 English datasets. MMTEB has 131 multilingual tasks. |
| **Recall@k / nDCG / MRR** | Ba chỉ số đánh giá search. nDCG tính đến vị trí kết quả. / Three search evaluation metrics. nDCG considers result position. |
| **Matryoshka (MRL)** | Kỹ thuật cho phép cắt vector với mất mát nhỏ. / A technique that allows vector truncation with little loss. |
| **Chunking / truncation** | Chunking chia tài liệu thành đoạn. Truncation cắt bỏ phần vượt giới hạn token. / Chunking splits documents into segments. Truncation removes input beyond the token limit. |
| **ANN / HNSW** | ANN tìm láng giềng gần nhất xấp xỉ. HNSW là thuật toán phổ biến. / ANN performs approximate nearest-neighbor search. HNSW is a common algorithm. |
| **Re-embedding** | Quá trình tạo lại toàn bộ vector sau khi đổi model. / The process of regenerating every vector after a model change. |
| **Hybrid search** | Cách kết hợp vector search với full-text search. / A method that combines vector search with full-text search. |

### 5.2. Mười ý cốt lõi
*5.2. Ten core ideas*

**1.** Embedding biến ý nghĩa thành số.  
***1.*** *Embedding converts meaning into numbers.*

Nội dung giống nghĩa tạo vector gần nhau.  
*Similar meanings create nearby vectors.*

Máy tính có thể so sánh chúng bằng toán học.  
*Computers can compare them mathematically.*

**2.** Model tạo embedding.  
***2.*** *The model creates embeddings.*

Database chỉ lưu và tìm vector.  
*The database only stores and searches vectors.*

pgvector không tự tạo vector.  
*pgvector does not create vectors.*

Model giống đầu bếp.  
*The model resembles a chef.*

Database giống tủ lạnh.  
*The database resembles a refrigerator.*

**3.** Static và contextual là phân biệt cốt lõi.  
***3.*** *Static and contextual embeddings form a core distinction.*

Transformer thắng nhờ self-attention.  
*Transformers win because of self-attention.*

Vector của transformer phụ thuộc ngữ cảnh.  
*A transformer vector depends on context.*

**4.** Cosine đo hướng.  
***4.*** *Cosine measures direction.*

Cosine bỏ qua độ dài.  
*Cosine ignores magnitude.*

Đặc tính này phù hợp với văn bản.  
*This property suits text.*

Miền giá trị là `[−1,1]`.  
*The value range is `[−1,1]`.*

Miền không phải `[0,1]`.  
*The range is not `[0,1]`.*

**5.** Transformer tạo một vector cho mỗi token.  
***5.*** *A transformer creates one vector per token.*

Bạn cần pooling cho vector câu.  
*You need pooling for a sentence vector.*

Bạn cũng cần normalize.  
*You also need normalization.*

**6.** Vector đã normalize làm cosine bằng dot product.  
***6.*** *Normalized vectors make cosine equal dot product.*

Cosine và L2 tạo cùng thứ tự xếp hạng.  
*Cosine and L2 create the same ranking order.*

**7.** Vector từ hai model không thể so sánh.  
***7.*** *Vectors from two models cannot be compared.*

Hai phiên bản khác nhau cũng có thể không tương thích.  
*Two different versions may also be incompatible.*

Đổi model yêu cầu embedding lại toàn bộ dữ liệu.  
*Changing models requires re-embedding all data.*

**8.** Quá trình chọn model bắt đầu bằng ràng buộc.  
***8.*** *The model selection process begins with constraints.*

MTEB giúp lập danh sách ngắn.  
*MTEB helps create a shortlist.*

Dữ liệu thật quyết định lựa chọn.  
*Real data determines the choice.*

**9.** Hệ thống lớn cần batch.  
***9.*** *Large systems need batching.*

Hệ thống lớn cũng cần cache.  
*Large systems also need caching.*

Tài liệu dài cần chunking.  
*Long documents need chunking.*

Chi phí token phải được quy đổi thành tiền.  
*Token cost must be translated into money.*

Latency cũng là một ràng buộc thật.  
*Latency is also a real constraint.*

**10.** Pipeline bắt đầu từ text.  
***10.*** *The pipeline starts with text.*

Model tạo embedding.  
*The model creates an embedding.*

ANN lưu và tìm vector.  
*ANN stores and searches vectors.*

Hybrid search kết hợp full-text search.  
*Hybrid search combines full-text search.*

RAG có thể sử dụng kết quả cuối cùng.  
*RAG can use the final results.*

### 5.3. Mental models
*5.3. Mental models*

- **Bản đồ ý nghĩa** mô tả embedding như toạ độ.  
  *The meaning map describes an embedding as coordinates.*

- Nội dung giống nghĩa nằm gần nhau.  
  *Similar meanings lie close together.*

- **Model là đầu bếp.**  
  *The model is the chef.*

- **Database là tủ lạnh.**  
  *The database is the refrigerator.*

- Hình ảnh này giải thích pgvector không tạo embedding.  
  *This image explains why pgvector does not create embeddings.*

- **Cosine so hướng.**  
  *Cosine compares direction.*

- **Cosine không so độ dài.**  
  *Cosine does not compare magnitude.*

- Câu này giải thích tính phù hợp với văn bản.  
  *This sentence explains its suitability for text.*

- **“Bank” có hai nghĩa.**  
  *“Bank” has two meanings.*

- Ví dụ này minh hoạ static và contextual.  
  *This example illustrates static and contextual embeddings.*

- **Hai model tạo hai bản đồ khác nhau.**  
  *Two models create two different maps.*

- Cùng toạ độ có thể chỉ hai nơi khác nhau.  
  *The same coordinates may point to different places.*

- Hình ảnh này giải thích việc không được trộn model.  
  *This image explains why models must not be mixed.*

- **MTEB là giả định ban đầu.**  
  *MTEB is an initial prior.*

- **Dữ liệu của bạn là phán quyết.**  
  *Your data is the verdict.*

- Câu này thể hiện tư duy chín chắn.  
  *This sentence demonstrates mature judgment.*

- **Đường nóng và đường nguội** là hai luồng khác nhau.  
  *The hot path and cold path are different flows.*

- Truy vấn cũng cần embedding.  
  *Queries also need embeddings.*

- Đường truy vấn phải có latency thấp.  
  *The query path must have low latency.*

### 5.4. Code cần thuộc lòng
*5.4. Code to memorize*

**(a) Tạo embedding bằng JavaScript**  
***(a) Creating an embedding with JavaScript***

```javascript
import { pipeline } from '@huggingface/transformers';
const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
const out = await extractor(text, { pooling: 'mean', normalize: true });
const vec = Array.from(out.data);   // 384 số
                                         // 384 numbers
```

**(b) Tạo embedding bằng Python**  
***(b) Creating an embedding with Python***

```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-MiniLM-L6-v2")
vec = model.encode(text, normalize_embeddings=True)          # một câu
                                                               # one sentence
vecs = model.encode(texts, normalize_embeddings=True, batch_size=32)   # cả lô
                                                                            # a full batch
```

**(c) Viết cosine similarity từ số 0**  
***(c) Writing cosine similarity from scratch***

Interviewer thường yêu cầu viết hàm này.  
*Interviewers often ask candidates to write this function.*

```python
def cosine(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    na  = sum(x * x for x in a) ** 0.5
    nb  = sum(y * y for y in b) ** 0.5
    return dot / (na * nb) if na and nb else 0.0
    #                          ↑ guard vector 0 — nhớ nói ra edge case này,
    #                          ↑ guard the zero vector and mention this edge case,
    #                            đây là chi tiết phân biệt ứng viên cẩn thận
    #                            because this detail distinguishes careful candidates
```

**(d) Tìm top-k bằng NumPy**  
***(d) Finding top-k with NumPy***

Câu hỏi thường yêu cầu cách làm nhanh hơn.  
*The question often asks for a faster method.*

```python
import numpy as np
def top_k(query, corpus, k=5):
    q = query / np.linalg.norm(query)
    c = corpus / np.linalg.norm(corpus, axis=1, keepdims=True)
    sims = c @ q                      # một phép nhân ma trận thay cho vòng lặp
                                        # one matrix multiplication replaces a loop
    idx = np.argsort(-sims)[:k]
    return idx, sims[idx]
```

### 5.5. Câu hỏi phỏng vấn thường gặp và gợi ý trả lời
*5.5. Common interview questions and suggested answers*

**1. “Embedding là gì?”**  
***1. “What is an embedding?”***

Embedding là một dãy số nhiều chiều.  
*An embedding is a multidimensional sequence of numbers.*

Nó mã hoá ý nghĩa của một đối tượng.  
*It encodes the meaning of an object.*

Nội dung giống nghĩa nằm gần nhau trong không gian.  
*Semantically similar content lies close together in the space.*

Một câu trả lời tốt nên nhắc static embedding.  
*A strong answer should mention static embeddings.*

Word2Vec tạo một vector cố định cho mỗi từ.  
*Word2Vec creates one fixed vector per word.*

Một câu trả lời tốt cũng nên nhắc contextual embedding.  
*A strong answer should also mention contextual embeddings.*

Transformer tạo vector phụ thuộc ngữ cảnh.  
*A transformer creates context-dependent vectors.*

Self-attention giúp tạo ra sự khác biệt này.  
*Self-attention enables this difference.*

Đây là một lý do transformer vượt static embedding.  
*This is one reason transformers outperform static embeddings.*

**2. “Vì sao dùng cosine thay vì Euclidean cho văn bản?”**  
***2. “Why use cosine instead of Euclidean distance for text?”***

Cosine đo hướng.  
*Cosine measures direction.*

Cosine bỏ qua độ dài.  
*Cosine ignores magnitude.*

Với văn bản, hướng thường phản ánh chủ đề.  
*For text, direction often reflects topic.*

Độ dài vector có thể phản ánh độ dài câu.  
*Vector magnitude may reflect sentence length.*

Vector đã chuẩn hoá tạo một kết quả quan trọng.  
*Normalized vectors create an important result.*

Cosine và L2 tạo cùng thứ tự xếp hạng.  
*Cosine and L2 create the same ranking order.*

Khi đó lựa chọn dựa trên hiệu năng.  
*The choice then depends on performance.*

Lựa chọn không còn dựa trên chất lượng xếp hạng.  
*The choice no longer depends on ranking quality.*

**3. [BẪY] “Cosine similarity chạy từ 0 đến 1, đúng không?”**  
***3. [TRICK] “Cosine similarity ranges from 0 to 1, right?”***

Câu trả lời là không hoàn toàn đúng.  
*The answer is not entirely correct.*

Miền giá trị toán học là `[−1,1]`.  
*The mathematical range is `[−1,1]`.*

Cosin của mọi góc nằm trong miền đó.  
*The cosine of every angle lies in that range.*

Giá trị chỉ nằm trong `[0,1]` ở một trường hợp riêng.  
*Values lie in `[0,1]` only in a special case.*

Mọi thành phần của hai vector phải không âm.  
*Every component of both vectors must be nonnegative.*

Embedding hiện đại thường có thành phần âm.  
*Modern embeddings often contain negative components.*

Cosine vì thế có thể mang giá trị âm.  
*Cosine can therefore have a negative value.*

Các cặp văn bản thực sự giống nhau thường có điểm dương cao.  
*Truly similar text pairs usually have high positive scores.*

**4. [BẪY] “Hai câu tạo shape khác nhau. Vì sao?”**  
***4. [TRICK] “Two sentences produce different shapes. Why?”***

Nguyên nhân là thiếu pooling.  
*The cause is missing pooling.*

Feature-extraction trả một vector cho mỗi token.  
*Feature extraction returns one vector per token.*

Shape có dạng `[1, số_token, d]`.  
*The shape is `[1, token_count, d]`.*

Hai câu có thể có số token khác nhau.  
*Two sentences may have different token counts.*

JavaScript cần `pooling: 'mean'`.  
*JavaScript needs `pooling: 'mean'`.*

JavaScript cũng cần `normalize: true`.  
*JavaScript also needs `normalize: true`.*

Python có thể dùng `sentence-transformers`.  
*Python can use `sentence-transformers`.*

Bạn nên đặt `normalize_embeddings=True`.  
*You should set `normalize_embeddings=True`.*

**5. “pgvector tự tạo embedding, đúng không?”**  
***5. “pgvector creates embeddings by itself, right?”***

Câu trả lời là không.  
*The answer is no.*

pgvector chỉ lưu vector.  
*pgvector only stores vectors.*

Nó cũng tìm vector gần nhất bằng ANN index.  
*It also finds nearest vectors with an ANN index.*

Một embedding model tạo ra vector.  
*An embedding model creates the vector.*

Bạn gọi model trước.  
*You call the model first.*

Sau đó bạn lấy vector.  
*Then you receive the vector.*

Cuối cùng bạn đưa vector vào pgvector.  
*Finally you send the vector to pgvector.*

Model là đầu bếp.  
*The model is the chef.*

Database là tủ lạnh.  
*The database is the refrigerator.*

**6. “Bạn tìm được model tốt hơn. Dữ liệu cũ phải làm gì?”**  
***6. “You found a better model. What happens to old data?”***

Bạn phải embedding lại toàn bộ dữ liệu.  
*You must re-embed all data.*

Vector cũ nằm trong một không gian.  
*Old vectors exist in one space.*

Vector mới nằm trong không gian khác.  
*New vectors exist in another space.*

Hai nhóm không thể so sánh trực tiếp.  
*The two groups cannot be compared directly.*

Số chiều giống nhau không giải quyết vấn đề.  
*Matching dimensions do not solve the problem.*

Cách an toàn bắt đầu bằng một cột mới có phiên bản.  
*The safe method begins with a new versioned column.*

Hai cột chạy song song.  
*The two columns run in parallel.*

Backfill chạy nền.  
*The backfill runs in the background.*

Backfill phải có giới hạn tốc độ.  
*The backfill must have rate limits.*

Dữ liệu mới phải được dual-write.  
*New data must use dual writes.*

Lưu lượng được chuyển dần theo blue-green.  
*Traffic shifts gradually with blue-green deployment.*

Đây là dự án kéo dài nhiều ngày.  
*This is a multi-day project.*

Nó không phải một dòng cấu hình.  
*It is not one configuration line.*

Bạn nên đầu tư kỹ hơn khi chọn model ban đầu.  
*You should invest more effort in initial model selection.*

**7. [TRADE-OFF] “Self-host model hay dùng API?”**  
***7. [TRADE-OFF] “Should you self-host the model or use an API?”***

API không yêu cầu bạn vận hành hạ tầng model.  
*An API does not require you to operate model infrastructure.*

Nhà cung cấp thường cập nhật dịch vụ.  
*The provider usually updates the service.*

API giúp ra mắt nhanh.  
*An API supports a fast launch.*

API tính tiền theo lượng dùng.  
*An API charges by usage.*

Hệ thống phụ thuộc vào bên thứ ba.  
*The system depends on a third party.*

Rate limit có thể ảnh hưởng hệ thống.  
*Rate limits may affect the system.*

Downtime của nhà cung cấp cũng có thể ảnh hưởng.  
*Provider downtime may also affect the system.*

Nhà cung cấp có thể đổi phiên bản âm thầm.  
*The provider may silently change versions.*

Latency mạng của API thường cao hơn.  
*API network latency is usually higher.*

Dữ liệu phải rời khỏi hệ thống của bạn.  
*Data must leave your system.*

Self-host có chi phí biên thấp hơn.  
*Self-hosting has a lower marginal cost.*

Dữ liệu được giữ trong hạ tầng nội bộ.  
*Data remains inside internal infrastructure.*

Latency đường truy vấn có thể thấp hơn nhiều.  
*Query-path latency may be much lower.*

Self-host yêu cầu GPU.  
*Self-hosting requires GPUs.*

Bạn phải tự cập nhật model.  
*You must update the model yourself.*

Bạn cũng cần người có kinh nghiệm vận hành.  
*You also need experienced operators.*

Quyền riêng tư thường là yếu tố quyết định.  
*Privacy is often the deciding factor.*

Dữ liệu có thể bị cấm rời công ty.  
*Data may be prohibited from leaving the company.*

Khi đó benchmark không còn quyết định lựa chọn.  
*Benchmark rank no longer determines the choice.*

**8. [SCALE] “Bạn cần embed 100 triệu tài liệu. Bạn tối ưu những gì?”**  
***8. [SCALE] “You need to embed 100 million documents. What do you optimize?”***

Bạn nên ưu tiên theo mức tác động.  
*You should prioritize by impact.*

Ưu tiên đầu tiên là chunking hợp lý.  
*The first priority is sensible chunking.*

Chunking giúp tránh truncation.  
*Chunking prevents truncation.*

Chunking cũng giúp tránh embedding nội dung rác.  
*Chunking also prevents embedding useless content.*

Ưu tiên thứ hai là batching.  
*The second priority is batching.*

Mỗi batch có thể chứa 32–256 câu.  
*Each batch may contain 32–256 sentences.*

Batching giúp tận dụng GPU.  
*Batching improves GPU utilization.*

Ưu tiên thứ ba là cache theo hash nội dung.  
*The third priority is caching by content hash.*

Cache giúp tránh tính lại embedding.  
*Caching prevents repeated embedding computation.*

Ưu tiên thứ tư là cân nhắc Matryoshka.  
*The fourth priority is considering Matryoshka.*

Kỹ thuật này có thể giảm số chiều vector.  
*This technique can reduce vector dimensions.*

Nó phù hợp khi RAM cho index là ràng buộc.  
*It is suitable when index RAM is a constraint.*

Ưu tiên thứ năm là xử lý bất đồng bộ.  
*The fifth priority is asynchronous processing.*

Bạn nên sử dụng queue.  
*You should use a queue.*

Queue giúp hệ thống chịu tải đột biến.  
*A queue helps the system absorb traffic spikes.*

Bạn phải tính chi phí trước mọi cam kết.  
*You must calculate cost before making any commitment.*

100 triệu tài liệu có thể chứa 200 token mỗi tài liệu.  
*One hundred million documents may contain 200 tokens each.*

Tổng số khi đó là 20 tỉ token.  
*The total is then 20 billion tokens.*

Bạn nhân con số này với đơn giá model.  
*You multiply this figure by the model price.*

Kết quả là chi phí cụ thể.  
*The result is a concrete cost.*

**9. “Bạn chọn embedding model thế nào?”**  
***9. “How do you choose an embedding model?”***

Bạn phải liệt kê các ràng buộc trước.  
*You must list the constraints first.*

Loại dữ liệu là ràng buộc đầu tiên.  
*Data type is the first constraint.*

Ngôn ngữ là ràng buộc thứ hai.  
*Language is the second constraint.*

Quyền riêng tư là ràng buộc thứ ba.  
*Privacy is the third constraint.*

Latency là ràng buộc thứ tư.  
*Latency is the fourth constraint.*

Ngân sách là ràng buộc thứ năm.  
*Budget is the fifth constraint.*

Ràng buộc cứng có thể loại model đứng đầu.  
*A hard constraint may eliminate the top-ranked model.*

Sau đó bạn lập danh sách ngắn gồm 2–3 model.  
*Then you create a shortlist of 2–3 models.*

Bạn có thể sử dụng MTEB hoặc MMTEB.  
*You can use MTEB or MMTEB.*

Tiếp theo bạn đo trên tập truy vấn thật.  
*Next you evaluate on real queries.*

Tập truy vấn phải có nhãn.  
*The query set must have labels.*

Bạn nên đo recall@k.  
*You should measure Recall@k.*

Bạn cũng nên đo nDCG.  
*You should also measure nDCG.*

Đây là bước quyết định.  
*This is the deciding step.*

Cuối cùng bạn cân nhắc fine-tune.  
*Finally you consider fine-tuning.*

Fine-tune phù hợp với lĩnh vực rất hẹp.  
*Fine-tuning suits a very narrow domain.*

MTEB chỉ là giả định ban đầu.  
*MTEB is only an initial prior.*

MTEB không phải phán quyết cuối cùng.  
*MTEB is not the final verdict.*

**10. [BẪY] “Hệ thống tìm kiếm ngày càng kém. Không ai thay đổi gì. Chuyện gì xảy ra?”**  
***10. [TRICK] “Search quality keeps declining. Nobody changed anything. What happened?”***

Có vài khả năng chính.  
*There are several main possibilities.*

Khả năng đầu tiên là nhà cung cấp đổi model âm thầm.  
*The first possibility is a silent provider model change.*

Tên API có thể vẫn giữ nguyên.  
*The API name may remain unchanged.*

Vector mới có thể lệch khỏi vector cũ.  
*New vectors may diverge from old vectors.*

Bạn nên ghim phiên bản model.  
*You should pin the model version.*

Bạn cũng nên lưu tên model cạnh mỗi vector.  
*You should also store the model name beside every vector.*

Khả năng thứ hai là drift.  
*The second possibility is drift.*

Dữ liệu có thể đã thay đổi.  
*The data may have changed.*

Cách người dùng gõ cũng có thể thay đổi.  
*User query behavior may also have changed.*

Khả năng thứ ba là truncation âm thầm.  
*The third possibility is silent truncation.*

Tài liệu mới có thể dài hơn trước.  
*New documents may be longer than before.*

Model có thể bỏ mất phần đuôi.  
*The model may discard the tail.*

Ba vấn đề này cần monitoring định kỳ.  
*These three problems require periodic monitoring.*

Bạn phải đo recall@k trên tập vàng.  
*You must measure Recall@k on the gold set.*

Một lần đo lúc ra mắt là chưa đủ.  
*One launch-time measurement is insufficient.*

### 5.6. Câu chốt one-liner đắt giá
*5.6. Valuable one-liners*

Đây là các câu ngắn.  
*These are short statements.*

Mỗi câu thể hiện chiều sâu kỹ thuật.  
*Each statement demonstrates technical depth.*

Bạn nên sử dụng chúng đúng lúc.  
*You should use them at the right moment.*

- Model tạo vector.  
  *The model makes the vector.*

- Database lưu vector.  
  *The database stores the vector.*

- pgvector không tự tạo embedding.  
  *pgvector never creates embeddings by itself.*

- Cosine đo hướng.  
  *Cosine measures direction.*

- Cosine không đo độ dài.  
  *Cosine does not measure magnitude.*

- Đặc tính này phù hợp với ý nghĩa.  
  *This property is exactly what meaning requires.*

- Static embedding tạo một vector cho mỗi từ.  
  *Static embeddings create one vector per word.*

- Contextual embedding tạo vector theo ngữ cảnh.  
  *Contextual embeddings create vectors in context.*

- Đây là lý do transformer chiến thắng.  
  *This is why transformers won.*

- Hai model tạo hai không gian khác nhau.  
  *Two models create two different spaces.*

- Đổi embedding model yêu cầu embedding lại toàn bộ dữ liệu.  
  *Changing an embedding model requires re-embedding all data.*

- MTEB giúp thu hẹp danh sách.  
  *MTEB narrows the shortlist.*

- Truy vấn có nhãn của bạn quyết định lựa chọn.  
  *Your labeled queries determine the choice.*

- Quyền riêng tư có thể quan trọng hơn benchmark.  
  *Privacy can override the benchmark.*

- Dữ liệu có thể không được phép rời công ty.  
  *Data may not be allowed to leave the company.*

- Khi đó bạn có thể self-host một model yếu hơn chút ít.  
  *You may then self-host a slightly weaker model.*

- Truncation là bug im lặng nhất trong hệ thống.  
  *Truncation is the quietest bug in the stack.*

- Hệ thống không báo lỗi.  
  *The system reports no error.*

- Hệ thống cũng không đưa cảnh báo.  
  *The system also provides no warning.*

- Một phần tài liệu chỉ âm thầm biến mất khỏi index.  
  *Part of the document silently disappears from the index.*

- Truy vấn cũng cần một embedding.  
  *A query also needs an embedding.*

- Embedding truy vấn nằm trên đường nóng.  
  *Query embedding sits on the hot path.*

---

## 📌 Ghi chú cuối
*📌 Final notes*

### Ba việc nên làm ngay sau khi đọc xong
*Three actions to take immediately after reading*

**Một — Chạy thật một lần.**  
***First: run a real example once.***

Bạn có thể cài `sentence-transformers` cho Python.  
*You can install `sentence-transformers` for Python.*

Bạn cũng có thể cài `@huggingface/transformers` cho JavaScript.  
*You can also install `@huggingface/transformers` for JavaScript.*

Sau đó bạn embed ba câu.  
*Then you embed three sentences.*

Hai câu phải đồng nghĩa.  
*Two sentences must be synonymous.*

Một câu phải không liên quan.  
*One sentence must be unrelated.*

Bạn tự tính cosine bằng hàm tại Mục 5.4.  
*You calculate cosine with the function in Section 5.4.*

Bạn kiểm tra kết quả bằng mắt.  
*You inspect the result directly.*

Hai câu đồng nghĩa phải có điểm cao hơn rõ rệt.  
*The synonymous sentences must score clearly higher.*

Bài thực hành này mất khoảng 15 phút.  
*This exercise takes about 15 minutes.*

Nó biến kiến thức lý thuyết thành kiến thức thật.  
*It turns theoretical knowledge into real knowledge.*

**Hai — Tự kiểm tra chất lượng tiếng Việt.**  
***Second: evaluate Vietnamese quality yourself.***

Bạn viết 20 cặp câu tiếng Việt đồng nghĩa.  
*You write 20 synonymous Vietnamese sentence pairs.*

Bạn phải biết chắc chúng đồng nghĩa.  
*You must know that they are truly synonymous.*

Bạn cũng tạo 20 cặp ngẫu nhiên.  
*You also create 20 random pairs.*

Sau đó bạn đo cosine cho cả 40 cặp.  
*Then you measure cosine for all 40 pairs.*

Hai nhóm phải tách biệt rõ ràng.  
*The two groups must separate clearly.*

Điểm lẫn lộn cho thấy model không phù hợp.  
*Mixed scores indicate an unsuitable model.*

Đây là phiên bản nhỏ của Bước 3 tại Mục 4.2.  
*This is a small-scale version of Step 3 in Section 4.2.*

**Ba — Đóng vòng với pgvector.**  
***Third: complete the loop with pgvector.***

Bạn lấy vector vừa tạo.  
*You take the generated vector.*

Bạn lưu nó vào PostgreSQL có pgvector.  
*You store it in PostgreSQL with pgvector.*

Sau đó bạn chạy một semantic search query.  
*Then you run a semantic search query.*

Toàn bộ chuỗi gồm embed, lưu và tìm.  
*The complete chain includes embedding, storage, and search.*

Bạn cần thấy chuỗi chạy từ đầu đến cuối.  
*You need to see the chain run end to end.*

Khi đó mọi mảnh ghép sẽ khớp lại.  
*The pieces will then fit together.*

### Nối mạch với các bài khác trong series
*Connecting this lesson with other lessons in the series*

```text
   ┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐
   │   BÀI NÀY       │   │  BÀI pgvector    │   │   BÀI FTS       │
   │  THIS LESSON    │   │  pgvector LESSON │   │  FTS LESSON     │
   │  Tạo vector     │ → │  Lưu & tìm vector│ + │  Tìm theo từ khoá│
   │  Create vectors │   │  Store & search  │   │  Keyword search │
   │  (embedding)    │   │  (ANN, HNSW)     │   │  (lexeme)       │
   └─────────────────┘   └──────────────────┘   └─────────────────┘
              ↓                    ↓                     ↓
         ══════════════════════════════════════════════════
                    HYBRID SEARCH → SEMANTIC SEARCH → RAG
         ══════════════════════════════════════════════════
                        Tất cả chạy trên một PostgreSQL
                        Everything runs on one PostgreSQL
```

Có ba mảnh ghép.  
*There are three pieces.*

Cả ba sử dụng một database.  
*All three use one database.*

Embedding hiểu ý nghĩa.  
*Embeddings understand meaning.*

Full-text search bắt chính xác từ khoá.  
*Full-text search captures exact keywords.*

Nó cũng bắt mã sản phẩm.  
*It also captures product codes.*

Nó cũng bắt tên riêng.  
*It also captures proper names.*

Kết hợp hai cách thường tạo kết quả tốt hơn.  
*Combining both methods usually creates better results.*

Mỗi cách riêng lẻ thường kém hơn.  
*Either method alone is usually weaker.*

### Nhớ kiểm chứng lại khi ôn
*Remember to verify updates during review*

Tên gói JavaScript hiện là `@huggingface/transformers`.  
*The current JavaScript package name is `@huggingface/transformers`.*

Tài liệu gốc ghi phiên bản 4.  
*The original material states version 4.*

Phiên bản đó ra mắt vào tháng 2/2026.  
*That version launched in February 2026.*

Nó hỗ trợ WebGPU.  
*It supports WebGPU.*

Tên cũ là `@xenova/transformers`.  
*The old name is `@xenova/transformers`.*

Tên cũ vẫn có thể được cài đặt.  
*The old package can still be installed.*

Tên cũ đã ngừng phát triển.  
*The old package is no longer developed.*

Bức tranh model thay đổi hằng tháng.  
*The model landscape changes monthly.*

Bảng tại Mục 2.3 đúng vào tháng 7/2026.  
*The table in Section 2.3 was current in July 2026.*

Bạn nên kiểm tra MTEB trước cuộc phỏng vấn.  
*You should check MTEB before an interview.*

Bảng xếp hạng nằm trên Hugging Face.  
*The leaderboard is hosted on Hugging Face.*

Giá API cũng thay đổi liên tục.  
*API prices also change continuously.*

Các con số tại Mục 4.1 chỉ minh hoạ cách tính.  
*The figures in Section 4.1 only illustrate the calculation method.*

Bạn không nên trích dẫn chúng như giá hiện hành.  
*You should not cite them as current prices.*

Bạn phải kiểm tra bảng giá mới nhất khi cần số thật.  
*You must check the latest pricing table when real figures are needed.*

### Học tiếp gì sau bài này
*What to learn next*

- **Chiến lược chunking cho RAG**  
  *Chunking strategies for RAG*

- Cách chia tài liệu ảnh hưởng rất lớn đến chất lượng.  
  *Document splitting strongly affects quality.*

- Ảnh hưởng này đôi khi lớn hơn việc chọn model.  
  *This impact sometimes exceeds model selection.*

- **Reranking bằng cross-encoder**  
  *Reranking with a cross-encoder*

- ANN có thể lọc ra 100 ứng viên.  
  *ANN may retrieve 100 candidates.*

- Cross-encoder tinh chỉnh thứ tự của các ứng viên.  
  *A cross-encoder refines the candidate ranking.*

- Kỹ thuật này cải thiện top 10 đáng kể.  
  *This technique substantially improves the top 10.*

- **Multimodal embedding**  
  *Multimodal embeddings*

- Kỹ thuật này nhúng text và ảnh vào cùng không gian.  
  *This technique embeds text and images in the same space.*

- Nó cho phép tìm ảnh bằng text.  
  *It enables image search with text.*

- **Đánh giá hệ thống retrieval**  
  *Retrieval system evaluation*

- Các chỉ số gồm recall@k, nDCG và MRR.  
  *Metrics include Recall@k, nDCG, and MRR.*

- Bạn cũng cần biết cách xây tập vàng đáng tin cậy.  
  *You also need to know how to build a trustworthy gold set.*

---

*Giáo trình được soạn theo yêu cầu “giáo trình siêu dễ hiểu”.*  
*This course was written under the “super easy-to-understand course” instruction.*

*Thông tin được cập nhật tới tháng 7/2026.*  
*The information was updated through July 2026.*

*Bảng xếp hạng model có thể thay đổi.*  
*Model leaderboards may change.*

*Tên gói phần mềm có thể thay đổi.*  
*Software package names may change.*

*Giá API cũng có thể thay đổi.*  
*API prices may also change.*

*Bạn nên kiểm chứng lại trước một quyết định thật.*  
*You should verify the information before a real decision.*
