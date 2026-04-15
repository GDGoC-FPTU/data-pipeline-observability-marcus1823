# Experiment Report: Data Quality Impact on AI Agent

**Student ID:** marcuschill1823@gmail.com - 2A202600438
**Name:** Trương Minh Tiền
**Date:** 15-04-2025

---

## 1. Kết quả thí nghiệm

Chạy `agent_simulation.py` với 2 bộ dữ liệu và ghi lại kết quả:

| Scenario                          | Agent Response                                                          | Accuracy (1-10) | Notes                                                                                                                                                                   |
| --------------------------------- | ----------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Clean Data (`processed_data.csv`) | Agent: Based on my data, the best choice is Laptop at $1200.            | 9               | Kết quả đúng. Laptop là sản phẩm electronics có giá cao nhất sau khi validate & transform (chỉ còn 3 records hợp lệ: Laptop, Chair, Monitor).                           |
| Garbage Data (`garbage_data.csv`) | Agent: Based on my data, the best choice is Nuclear Reactor at $999999. | 2               | Kết quả sai lệch. Agent bị đánh lừa bởi outlier "Nuclear Reactor" giá $999,999 — một record hoàn toàn vô lý nhưng vẫn được chấp nhận vì dữ liệu chưa qua bước validate. |

---

## 2. Phân tích & nhận xét (Phan tich & nhan xet)

### Tại sao Agent trả lời sai khi dùng Garbage Data? (Tai sao Agent tra loi sai?)

Khi sử dụng `garbage_data.csv`, Agent trả lời sai vì dữ liệu đầu vào chứa nhiều vấn đề nghiêm trọng mà không hề được kiểm tra (validate) trước khi đưa vào pipeline:

- **Duplicate IDs:** Có 2 records cùng `id=1` (Laptop và Banana). Nếu Agent dùng ID làm khoá để tra cứu, nó sẽ bị nhầm lẫn giữa các sản phẩm hoàn toàn khác nhau, dẫn đến trả về sai thực thể.
- **Wrong data types:** Record "Broken Chair" có `price = "ten dollars"` (chuỗi thay vì số). Điều này có thể gây crash khi tính toán (`idxmax`, so sánh giá trị) hoặc bị bỏ qua im lặng, làm mất dữ liệu mà người dùng không hay biết.
- **Extreme outliers:** "Nuclear Reactor" có giá $999,999 — giá trị cực kỳ bất thường so với các sản phẩm khác (vài chục đến vài nghìn đô). Vì Agent dùng `idxmax()` để chọn sản phẩm "tốt nhất" theo giá, nó ngay lập tức bị đánh lừa bởi outlier này và đưa ra đề xuất vô nghĩa.
- **Null values:** Record "Ghost Item" có `id = None` và `category = None`. Các giá trị null sẽ gây lỗi khi filter hoặc so sánh, làm giảm độ tin cậy của kết quả và có thể khiến pipeline báo lỗi bất ngờ.

Tất cả những vấn đề trên đều sẽ bị chặn bằng bước **validate** trong ETL pipeline (loại bỏ price <= 0, category rỗng, chuẩn hoá kiểu dữ liệu, phát hiện outlier). Khi bỏ qua bước này, "rác vào = rác ra" (garbage in, garbage out) — dù Agent có prompt tốt đến đâu cũng không thể cứu được chất lượng đầu ra.

---

## 3. Kết luận

**Quality Data > Quality Prompt?** **Đồng ý.**

Thí nghiệm cho thấy rõ: cùng một Agent, cùng một prompt ("What is the best electronic product?"), nhưng chỉ cần thay đổi chất lượng dữ liệu đầu vào thì kết quả chuyển từ đúng (Laptop $1200) sang sai hoàn toàn (Nuclear Reactor $999,999). Một prompt được thiết kế tốt cũng không thể bù đắp cho dữ liệu bị nhiễm bẩn, vì LLM/Agent chỉ "suy luận" trên những gì nó được cung cấp. Do đó, đầu tư vào **Data Pipeline** và **Data Observability** (validation, monitoring, logging) là nền móng bắt buộc trước khi nói đến tối ưu prompt hay fine-tune model.
