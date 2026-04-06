# Individual Report: Lab 3 - Chatbot vs ReAct Agent

**Student Name:** Lê Minh Khang  
**Student ID:** 2A202600102  
**Date:** 6/4/2026  

---

## I. Technical Contribution (15 Points)

Mô tả đóng góp cụ thể vào mã nguồn (ví dụ: triển khai công cụ cụ thể, sửa bộ phân giải, v.v.).

- **Modules Implemented:** `src/tools/oxford_tool.py`

- **Code Highlights:**  

- **Documentation:**  
Khi LLM cần tra cứu định nghĩa chính xác của một từ vựng, nó sẽ gọi công cụ Oxford API. Điều này giúp Agent cung cấp kiến thức chuẩn xác thay vì tự suy diễn nghĩa của từ.

---

## II. Debugging Case Study (10 Points)

Phân tích một sự cố lỗi cụ thể gặp phải trong quá trình làm lab thông qua hệ thống logging.

- **Problem Description:**  
Agent rơi vào vòng lặp (loop) khi cố gắng liệt kê các bộ thẻ nhớ, liên tục gọi `list_flashcard_sets()` nhưng gặp lỗi kỹ thuật, dẫn đến việc chạm ngưỡng tối đa (max steps) và báo lỗi định dạng.

- **Log Source:**  
`logs/2026-04-06.log`, mốc thời gian khoảng `2026-04-06T10:29:....`

- **Diagnosis:**  
  - Qua log, hệ thống báo:  
    `TypeError: list_sets_func() takes 0 positional arguments but 1 was given.`  
  - Nguyên nhân do LLM tự động truyền tham số vào hàm trong khi hàm này không yêu cầu đầu vào.  
  - Ngoài ra, tại Step 2 (log `10:07:12`), lỗi thiếu argument `front` và `back` cho thấy việc ánh xạ tham số từ chuỗi JSON sang hàm Python chưa đồng bộ.

- **Solution:**  
  - Cập nhật mô tả công cụ trong prompt để chỉ rõ `list_flashcard_sets` không nhận tham số.  
  - Điều chỉnh logic xử lý tại `agent.py` để unpack các tham số từ Action một cách chính xác hơn.  

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

Phản hồi về sự khác biệt trong khả năng lập luận.

- **Reasoning:**  
Khối Thought giúp Agent chia nhỏ vấn đề thành các bước logic (ví dụ: tạo set trước, thêm card sau). Chatbot thông thường thường trả lời trực tiếp dựa trên dữ liệu có sẵn, dễ dẫn đến sai sót nếu thiếu thông tin thời gian thực. ReAct cung cấp minh chứng rõ ràng cho "tại sao" Agent lại thực hiện hành động đó.

- **Reliability:**  
Agent thực tế hoạt động kém hơn Chatbot khi:  
  - Task quá đơn giản: Gây lãng phí tài nguyên và thời gian (latency cao) cho các câu chào hỏi hoặc yêu cầu đơn giản.  
  - Input thô tục: Như trong log `07:35:03`, Chatbot phản ứng linh hoạt hơn, trong khi Agent có thể bị "vấp" nếu không có kịch bản xử lý ngoại lệ tốt.  

- **Observation:**  
Phản hồi từ môi trường là "la bàn" cho Agent:  
  - Giúp Agent nhận ra lỗi (như lỗi thiếu tham số ở Step 2) để tìm cách khắc phục ở Step 3.  
  - Nếu Observation bị nhiễu hoặc trả về thông báo lỗi không rõ ràng, Agent sẽ dễ rơi vào vòng lặp vô tận.  

---

## IV. Future Improvements (5 Points)

Làm thế nào để mở rộng hệ thống này lên mức độ sản phẩm (production-level)?

- **Scalability:**  
Sử dụng hàng đợi bất đồng bộ (async queue) và kiến trúc microservices cho các công cụ để xử lý nhiều yêu cầu song song mà không làm nghẽn hệ thống chính.

- **Safety:**  
Triển khai một "Supervisor LLM" hoặc bộ lọc tiền xử lý để kiểm tra tính hợp lệ của Action, ngăn chặn các hành động lặp lại vô ích và lọc các nội dung nhạy cảm/thô tục từ người dùng.

- **Performance:**  
  - Sử dụng Vector Database để truy xuất công cụ (Tool Retrieval) khi số lượng công cụ lên tới hàng trăm.  
  - Lưu bộ nhớ đệm (Cache) các kết quả từ API (như Oxford) để giảm chi phí và tăng tốc độ phản hồi.  