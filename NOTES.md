# Ghi chú phân tích · Day 04

Họ và tên: Hồ An Lộc 
MSSV: 2A202603743

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: "Hiện tại, không có incident nào liên quan đến GitHub Actions trong năm 2025. Dữ liệu của tôi chỉ có thể truy cập đến năm 2026..."
- Điểm ở lần chạy 1: 45/100 (Dimensions: factuality=10, completeness=15, evidence=20, boundaries=0)

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra:

```text
Tool get_status trả về note: "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
Nhưng ở bước sinh câu trả lời, model kết luận: "không có incident nào liên quan đến GitHub Actions trong năm 2025".
Nguyên nhân: Model bị nhầm lẫn giữa việc "không thấy trong 50 incident gần nhất" với "không có sự cố trong năm 2025", và prompt chưa đủ mạnh để cảnh báo model không được suy diễn phủ định khi thiếu dữ liệu lịch sử dài hạn.
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu:
  1. Cải tiến `SYSTEM_PROMPT` trong `agent.py`: Làm rõ quy tắc về giới hạn dữ liệu (snapshot chỉ lưu tối đa 50 sự cố gần nhất, không khẳng định phủ định cho cả năm/giai đoạn dài), yêu cầu kiểm tra kết hợp `get_status` và `get_incident_details`, báo rõ khi tool bị lỗi/timeout và dẫn nguồn đầy đủ.
  2. Nâng cấp kiểm tra trong `run_agent` (`agent.py`): Kiểm tra đầy đủ tất cả URL nguồn của các incident đã đọc (`missing_urls`), tự động nhắc model đọc tiếp incident liên quan khi bản cập nhật có chứa đường dẫn `/incidents/<id>` sang sự cố khác, và thử lại có giới hạn khi gặp lỗi tạm thời.
- Vì sao chọn thay đổi này: Giải quyết triệt để lỗi ranh giới dữ liệu (Boundaries ở A5, H8), lỗi bỏ sót bước tra cứu incident liên quan (ở A6, H1, H6) và đảm bảo trích dẫn đầy đủ mọi URL nguồn mà không vi phạm quy tắc làm bài.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 81.27 / 100 | 94.95 / 100 (+13.68 điểm) |
| Điểm case đã chọn (A5) | 45 / 100 | 75 / 100 (+30 điểm) |
| Số model calls | 53 calls | 52 calls |
| Token, chi phí nếu có | 47,980 tokens | 67,931 tokens |

- Lỗi đã giải quyết:
  - Điểm tổng tăng mạnh từ **81.27 lên 94.95 / 100** (+13.68 điểm), toàn bộ 22/22 case đều hoàn thành và không có bất kỳ lỗi vi phạm `mechanical_checks` nào.
  - **Case A5** đã giải quyết lỗi vi phạm ranh giới dữ liệu (Boundaries), tăng từ **45 lên 75 điểm (+30 điểm)**: agent giải thích rõ dữ liệu chỉ có 50 sự cố gần nhất và không còn khẳng định sai rằng năm 2025 không có sự cố.
  - Các case phức tạp nhiều bước đều tăng điểm vượt bậc nhờ cơ chế theo dõi incident liên quan và kiểm tra đủ URL nguồn: **H1** tăng từ 60 lên **97 điểm** (+37), **H6** tăng từ 60 lên **80 điểm** (+20), **A6** tăng từ 75 lên **95 điểm** (+20), **A7** tăng từ 70 lên **93 điểm** (+23), **E1** tăng từ 75 lên **98 điểm** (+23), **H5** tăng từ 80 lên **100 điểm** (+20).
- Lỗi còn lại hoặc mới xuất hiện:
  - Tổng lượng token tăng từ 47,980 lên 67,931 tokens do `SYSTEM_PROMPT` chi tiết hơn và agent thực hiện đầy đủ các bước đọc incident liên quan (`get_incident_details`) thay vì dừng sớm.
  - Một số case như **A5, H7** (75 điểm) hay **H6** (80 điểm) vẫn bị LLM Judge trừ nhẹ điểm thành phần do cách diễn đạt chi tiết các sự cố lịch sử trong snapshot.

## 5. Giới hạn

- Giới hạn của điểm judge: Điểm số của LLM Judge mang tính chất ước lượng xấp xỉ và phụ thuộc vào nhiệt độ (temperature)/tính ngẫu nhiên của mô hình. Giữa các lần đánh giá khác nhau, cùng một câu trả lời có thể nhận điểm số khác nhau.
- Giới hạn của snapshot: Dữ liệu trong bài là ảnh chụp tĩnh tại thời điểm tháng 11/2026 và bị giới hạn ở 50 sự cố gần nhất. Do đó, agent không thể xác minh sự cố ngoài phạm vi này hoặc phản ánh trạng thái live tức thời nếu không có quyền truy cập Internet.
- Giới hạn của việc chỉ chạy một lần mỗi phiên bản: Một lần chạy đơn lẻ không thể chứng minh độ ổn định thống kê của agent; muốn kết luận vững chắc cần chạy nhiều lần lấy trung bình để triệt tiêu phương sai ngẫu nhiên.