---
title : "Điều phối các service"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

Cách đơn giản nhất để nối các service là đặt logic vào service Account Applications: lưu đơn mới xong thì gọi Data Checking hai lần, rồi đánh dấu hoặc duyệt đơn. Cách này chạy được, nhưng service nào cũng phải tự xử lý timeout, chạy lại và lỗi, và không ai nhìn thấy được hình dạng quy trình hay lịch sử xử lý của từng đơn.

**AWS Step Functions** giải quyết việc này. Bạn mô tả workflow dưới dạng **state machine**: một tập state có tên, một state bắt đầu và các bước chuyển giữa chúng. Step Functions chạy từng bước, chuyển dữ liệu từ bước này sang bước sau, chạy lại khi cần và lưu lịch sử trực quan của mọi execution.

Trong phần này bạn sẽ:

1. Tạo state machine đầu tiên với các state **Pass** trên console.
2. Thay chúng bằng state **Task** gọi function Data Checking.
3. Cấp quyền gọi Lambda cho state machine, lần này bằng AWS SAM.
4. Điều khiển dữ liệu truyền giữa các state bằng **ResultPath**.
5. Thêm state **Choice** để rẽ nhánh những đơn cần xem xét.
