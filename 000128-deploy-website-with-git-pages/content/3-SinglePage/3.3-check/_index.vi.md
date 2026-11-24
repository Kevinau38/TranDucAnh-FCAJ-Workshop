---
title : "Kiểm tra kết quả"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


1. Chọn tab **Actions**. GitHub đã tạo một lần chạy **pages build and deployment** cho lần push của bạn. Chờ đến khi có dấu tích xanh, khoảng một phút.

![pages build and deployment](/images/3-SinglePage/3.3-actions.png)

2. Mở `https://<username>.github.io/` trên trình duyệt. Website hiển thị tiêu đề và bài **My First Post**.

![User site](/images/3-SinglePage/3.3-site.png)

{{% notice info %}}
Nếu thấy trang 404, chờ một đến hai phút rồi tải lại. Nếu trang không có định dạng, hãy kiểm tra bạn đã build với `--baseURL https://<username>.github.io/`.
{{% /notice %}}
