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
```


## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |  |  |
| Điểm case đã chọn |  |  |
| Số model calls |  |  |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết:
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.
