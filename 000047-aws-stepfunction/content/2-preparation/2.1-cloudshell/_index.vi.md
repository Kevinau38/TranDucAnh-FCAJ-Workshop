---
title : "Mở AWS CloudShell"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

AWS CloudShell là shell chạy trên trình duyệt, đã cài sẵn AWS CLI, AWS SAM CLI và Node.js. CloudShell dùng quyền của người đang đăng nhập console, nên bạn không cần IAM user hay access key.

1. Đăng nhập AWS Management Console và chuyển sang Region **Asia Pacific (Tokyo) ap-northeast-1**.
2. Chọn biểu tượng **CloudShell** trên thanh điều hướng.
3. Khi dấu nhắc lệnh xuất hiện, kiểm tra các công cụ:

   ```bash
   sam --version; node --version; aws --version
   ```

   - (1) AWS SAM CLI, Node.js và AWS CLI v2 đã sẵn sàng.

   ![CloudShell](/images/2-preparation/2.1-cloudshell.png)

{{% notice note %}}
File trong thư mục home của CloudShell được giữ lại giữa các phiên trong cùng Region, nên bạn có thể đóng tab và làm tiếp sau.
{{% /notice %}}
