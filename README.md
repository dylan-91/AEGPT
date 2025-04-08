# KẾ HOẠCH TRIỂN KHAI CHATBOT NỘI BỘ SỬ DỤNG CUSTOM LLM MODEL

## 1. Mục tiêu triển khai

Xây dựng hệ thống **chatbot nội bộ thông minh**, hỗ trợ nhân viên tra cứu tài liệu, quy trình và nghiệp vụ nội bộ theo từng phòng ban. Hệ thống sử dụng mô hình LLM mã nguồn mở, tối ưu chi phí và đảm bảo bảo mật dữ liệu nội bộ.

---

## 2. Yêu cầu phần cứng máy chủ

| Hạng mục         | Cấu hình khuyến nghị                        |
|------------------|---------------------------------------------|
| **CPU**          | (4 cores trở lên)     |
| **RAM**          | Tối thiểu 16GB                              |
| **GPU**          | 12GB (ưu tiên ≥ 16GB VRAM) |
| **Ổ cứng**       | SSD ≥ 512GB                                 |
| **Hệ điều hành** | Windows        |
| **System**       | Python, API sẽ dùng Flask                 |

> Ghi chú: Có thể chạy bằng CPU nếu dùng mô hình GGUF quantized (Q4_K_M), tuy nhiên tốc độ phản hồi sẽ chậm hơn so với dùng GPU.

> Với cấu hình như yêu cầu, tương lai sắp tới, chúng ta có thể xây dựng hệ thống AI API cho hệ thống với nhiều chức năng:  

    - Nhận diện, so sánh hoặc các tác vụ liên quan hình ảnh
    - Speech to text, recap meeting...
    - Classification
    - Predict future values
    - ...
---

## 3. Tính năng hệ thống

- Giao diện web nội bộ cho phép nhân viên:
  - Nhập câu hỏi
  - Nhận câu trả lời ngắn gọn, chính xác
- Cho phép **tải lên tài liệu nội bộ** theo từng **phòng ban**
- Tự động xử lý tài liệu:
  - Chia nhỏ nội dung
  - Sinh vector và lưu trữ bằng ChromaDB
- Khi người dùng hỏi:
  - Truy tìm nội dung liên quan (top-k)
  - Nếu có, chèn vào prompt trả lời bằng **mô hình có context**
  - Nếu không có, chuyển sang **mô hình gốc để trả lời chung**
- Trả lời bằng tiếng Việt, có thể dẫn link tài liệu gốc (nếu có), cho phép users nhập prompt cài đặt phong cách trả lời.

---

## 4. Thành phần kỹ thuật

| Thành phần     | Công nghệ sử dụng                                       |
|----------------|--------------------------------------------------------|
| **LLM Model**  | Gemma:7b or Mistral 7B (định dạng GGUF, quantized Q4_K_M) , đã Fine Tuning         |
| **Runtime**    | [Ollama](https://ollama.com)                           |
| **Vector Store** | [ChromaDB](https://www.trychroma.com)              |
| **Embedding Model** | `sentence-transformers` (ViT5 hoặc multilingual) |
| **Truy vấn thông minh** | Semantic search + lọc theo trọng số          |
| **Frontend**   | Web app: .NET MVC (local web đã có)        |
| **Backend**    | Python (REST API tích hợp Ollama và ChromaDB)          |

---

## 5. Luồng xử lý thông minh (RAG hai tầng)
flowchart TD
    A[Người dùng đặt câu hỏi] --> B[Tìm vector gần nhất trong Chroma]
    B --> |score > 0.7| C[Sử dụng Modelfile 2: Trả lời có context]
    B --> |score < 0.7| D[Sử dụng Modelfile 1: Trả lời chung]

## 6. Thời gian triển khai (ước tính)

### Chi tiết thời gian theo giai đoạn:

| Giai đoạn                                | Công việc cụ thể                                             | Thời gian ước tính |
|------------------------------------------|--------------------------------------------------------------|--------------------|
| **Giai đoạn 1**: Cài đặt môi trường      | Cài đặt Ollama, mô hình Mistral, thư viện embedding          | 1 ngày             |
| **Giai đoạn 2**: Xử lý dữ liệu nội bộ    | Chia nhỏ dữ liệu, tạo embedding, lưu trữ vào ChromaDB        | 2–3 ngày           |
| **Giai đoạn 3**: Xây dựng API            | Xử lý truy vấn, lọc vector theo trọng số, kết nối với Ollama | 2 ngày             |
| **Giai đoạn 4**: Tích hợp giao diện      | Tạo giao diện web, gọi API backend, hiển thị trả lời         | 1–2 ngày           |
|                                          | **Tổng thời gian triển khai**                                | **6–8 ngày làm việc** |