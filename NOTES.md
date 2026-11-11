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

- Sửa gì, ở đâu: Cải tiến `SYSTEM_PROMPT` trong `agent.py`. Bổ sung chỉ dẫn: Khi người dùng yêu cầu danh sách đầy đủ trong một khoảng thời gian dài hoặc năm cũ (như 2025), bắt buộc phải giải thích rõ dữ liệu chỉ có tối đa 50 sự cố gần nhất và từ chối khẳng định chắc chắn về việc không có sự cố.
- Vì sao chọn thay đổi này: Thay đổi này trực tiếp khắc phục lỗi boundaries (ranh giới dữ liệu) mà không vi phạm quy tắc làm bài (không sửa dữ liệu, giữ nguyên logic loop).

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 81.27 / 100 | 79.32 / 100 |
| Điểm case đã chọn (A5) | 45 / 100 | 75 / 100 (+30 điểm) |
| Số model calls | 53 calls | 46 calls (giảm 7 calls) |
| Token, chi phí nếu có | 47,980 tokens | 37,181 tokens (giảm 22.5%) |

- Lỗi đã giải quyết: Case A5 đã giải quyết triệt để lỗi vi phạm ranh giới dữ liệu (Boundaries). Agent đã giải thích rõ dữ liệu chỉ có 50 sự cố gần nhất và không còn khẳng định sai lầm rằng "không có sự cố nào trong năm 2025". Đồng thời, agent trả lời cô đọng hơn giúp giảm số lượng tool calls không cần thiết và tiết kiệm đáng kể token.
- Lỗi còn lại hoặc mới xuất hiện: Điểm tổng dao động nhẹ (81.27 xuống 79.32) do tính ngẫu nhiên (variance) của LLM sinh câu trả lời và LLM Judge. Một số case (như H2, H4) bị judge trừ điểm chủ quan về văn phong dù logic sự thật không sai (ví dụ case thơ H4 lần 1 đạt 100 điểm, lần 2 bị chấm 80 điểm).

## 5. Giới hạn

- Giới hạn của điểm judge: Điểm số của LLM Judge mang tính chất ước lượng xấp xỉ và phụ thuộc vào nhiệt độ (temperature)/tính ngẫu nhiên của mô hình. Giữa các lần đánh giá khác nhau, cùng một câu trả lời có thể nhận điểm số khác nhau.
- Giới hạn của snapshot: Dữ liệu trong bài là ảnh chụp tĩnh tại thời điểm tháng 11/2026 và bị giới hạn ở 50 sự cố gần nhất. Do đó, agent không thể xác minh sự cố ngoài phạm vi này hoặc phản ánh trạng thái live tức thời nếu không có quyền truy cập Internet.
- Giới hạn của việc chỉ chạy một lần mỗi phiên bản: Một lần chạy đơn lẻ không thể chứng minh độ ổn định thống kê của agent; muốn kết luận vững chắc cần chạy nhiều lần lấy trung bình để triệt tiêu phương sai ngẫu nhiên.