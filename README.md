# 🎓 UIT Chatbot - Advanced RAG QA System


## 📌 Giới thiệu

UIT Chatbot là một hệ thống hỏi đáp học vụ được thiết kế dành riêng cho sinh viên Trường Đại học Công nghệ Thông tin (UIT) nhằm hỗ trợ giải đáp nhanh chóng và chính xác các vấn đề liên quan đến học vụ như điếm số, quy chế, học phí và các thủ tục hành chính khác.

Hệ thống được xây dựng dựa trên kiến trúc **RAG (Retrieval-Augmented Generation)** nâng cao, kết hợp giữa truy xuất văn bản hiệu quả và mô hình ngôn ngữ lớn để sinh câu trả lời tự nhiên, chính xác, có dẫn nguồn. Ngoài ra, hệ thống còn cải tiến các thành phần truy xuất nhằm tối ưu hiệu năng và độ chính xác trong môi trường tiếng Việt.


## 📚 Bộ dữ liệu

Hệ thống sử dụng bộ dữ liệu **UIT-QA**, gồm:

- **7029 câu hỏi - trả lời** liên quan đến các chủ đề học vụ phổ biến.
- **Tài liệu học vụ chính thức** từ trang web nhà trường: quy chế đào tạo, hướng dẫn học vụ, thông báo,...
- **Bộ dữ liệu embedding fine-tuning** nhằm huấn luyện lại các mô hình nhúng để tối ưu khả năng truy xuất.



## ⚙️ Kiến trúc hệ thống (Advanced RAG)

![Hình ảnh kiến trúc hệ thống](path/to/architecture.png) <!-- Thay bằng link ảnh nếu có -->

So với kiến trúc RAG tiêu chuẩn, hệ thống tích hợp **các module** sau:

- Query Transform giúp cải thiện biểu diễn ngữ nghĩa của truy vấn bằng cách tận dụng ngữ cảnh từ các lượt hỏi trước, đảm bảo mạch hội thoại trong các phiên hỏi đáp liên tục.
- Query Router có nhiệm vụ phân loại truy vấn thành small talk hoặc truy vấn chuyên ngành, từ đó định hướng xử lý phù hợp nhằm tối ưu hiệu suất và chất lượng phản hồi.
- Post Retrieval thực hiện lọc, sắp xếp và tái tổ chức các đoạn văn bản truy xuất để chọn ra ngữ cảnh phù hợp nhất đưa vào mô hình ngôn ngữ lớn, đảm bảo câu trả lời sinh ra chính xác và mạch lạc.


## 🔍 Các giai đoạn chính của hệ thống

### 1. 🧱 Indexing & Embedding

- Tài liệu được xử lý bằng hai kỹ thuật:
  - **Semantic Chunking**: phân đoạn dựa trên nội dung ngữ nghĩa.
  - **Recursive Chunking**: phân đoạn theo cấu trúc câu và độ dài cố định.
- Các chunk được biểu diễn thành vector nhờ hai mô hình:
  - `Halong_Embedding`
  - `VietNamese-phobert-base`
- Ngoài ra, hai mô hình trên được **fine-tune** trên tập truy xuất UIT-QA nhằm cải thiện độ tương đồng ngữ nghĩa trong ngữ cảnh học vụ.


### 2. 🔎 Retrieval

- Truy xuất các đoạn văn bản liên quan bằng cách tính tương đồng cosine giữa embedding của câu hỏi và các đoạn văn.
- Triển khai kỹ thuật **hybrid search**: kết hợp semantic search và BM25 search để nâng cao độ phủ.


### 3. 🧠 Generation (QA)

- Các đoạn văn đã truy xuất được đưa vào prompt kèm theo câu hỏi người dùng.
- Mô hình LLM (GPT-4o) sinh câu trả lời đầy đủ.
- Kỹ thuật **Chain-of-Thought** được tích hợp để hướng dẫn mô hình suy luận từng bước trước khi trả lời.


## 📊 Đánh giá thực nghiệm

#### Hệ thống được đánh giá với hai kỹ thuật phân đoạn tài liệu:

- **Recursive Chunking**: chia đoạn văn theo độ dài cố định (theo số từ hoặc câu).
- **Semantic Chunking**: phân đoạn theo cấu trúc ngữ nghĩa (VD: theo tiêu đề, mục lục, hoặc phân đoạn theo logic nội dung).

Bảng dưới đây trình bày kết quả so sánh dựa trên các chỉ số:

| Chunking Method    | Embedding Model     | HitRate@10 | MAP@10 | MRR@10 |
|--------------------|---------------------|-------------|--------|--------|
| Recursive Chunking | Halong_embedding    | **0.9110**      | **0.8497** | **0.6721* |
| Semantic Chunking  | Halong_embedding    | 0.9075  | 0.8121 | 0.6037 |
| Recursive Chunking | VietNamese-phobert-base | 0.9110      | 0.8209 | 0.6358 |
| Semantic Chunking  | VietNamese-phobert-base | 0.8973      | 0.8237 | 0.6139 |

> ✅ **Kết luận**: Recursive Chunking chứng tỏ sự phù hợp vượt trội đối với các văn bản học vụ nhờ khả năng tách theo cấu trúc logic – ví dụ phân mục thông báo, chương, mục, tiêu đề – vốn thường xuất hiện rõ ràng trong quy chế đào tạo, hướng dẫn học vụ và thông báo chính thức. Việc chia tài liệu theo từng khối cố định này không chỉ giúp bảo toàn mạch nội dung và ngữ cảnh chuyên sâu của mỗi phần, mà còn tối ưu hóa quá trình truy xuất bằng cách giữ nguyên cấu trúc thông tin quan trọng. Thực nghiệm cho thấy với Halong Embedding và VietNamese-phobert-base, Recursive Chunking cả hai đều nâng cao đáng kể HitRate@10 và MAP@10 so với Semantic Chunking, đồng thời duy trì MRR@10 ở mức cạnh tranh, qua đó khẳng định đây là giải pháp lý tưởng cho hệ thống RAG trong bối cảnh xử lý dữ liệu học vụ.

#### Hệ thống được đánh giá giai đoạn trước và sau khi thực hiện finetune hai mô hình embedding là **Halong_embedding** và **VietNamese-phobert-base** với dữ liệu truy xuất **UIT-QA**:

Bảng dưới đây trình bày kết quả so sánh dựa trên các chỉ số:


| Mô hình Embedding    | Fine-tune | HitRate@10 | MAP@10 | MRR@10 |
|----------------------|-----------|-------------|--------|--------|
| Halong_embedding     | ❌        | 0.9110      | **0.8497** | 0.6721 |
| Halong_embedding     | ✅        | **0.9349**  | 0.8386 | **0.6862** |
| VietNamese-phobert-base  | ❌        | 0.9110      | 0.8209 | 0.6358 |
| VietNamese-phobert-base  | ✅        | 0.9212      | 0.8121 | 0.6540 |

> ✅ **Kết luận**: Fine‑tuning embedding trên dữ liệu học vụ là bước thiết yếu giúp hệ thống RAG nắm bắt chính xác các thuật ngữ chuyên ngành, cải thiện đáng kể HitRate@10 mà vẫn duy trì MAP@10 và MRR@10. Halong_embedding hưởng lợi nhiều nhất với mức tăng rõ rệt, trong khi Vietnamese-phobert-base cũng tiến bộ nhưng cần thêm dữ liệu hoặc điều chỉnh siêu tham số để bắt kịp. Vì vậy, muốn tối ưu hiệu quả truy xuất, việc fine‑tune embedding thực sự cần thiết nếu có đủ tài nguyên và dữ liệu cho việc này.


