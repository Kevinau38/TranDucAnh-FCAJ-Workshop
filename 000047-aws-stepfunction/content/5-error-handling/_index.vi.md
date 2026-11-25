---
title : "Xử lý lỗi"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Hiện workflow chưa chịu được lỗi. Mặc định, khi một state lỗi thì cả execution thất bại. Lỗi có thể đến từ:

- definition của state machine (ví dụ state Choice không có quy tắc nào khớp),
- task (ví dụ Lambda function ném exception),
- sự cố tạm thời (ví dụ mạng hoặc Lambda bị giới hạn tốc độ).

Step Functions có hai công cụ: **Retry** chạy lại state bị lỗi, **Catch** chuyển sang state khác khi gặp một lỗi cụ thể.
