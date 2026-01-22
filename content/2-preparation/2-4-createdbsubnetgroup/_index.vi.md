---
title : "Tạo DB Subnet Group"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---


#### Tạo DB Subnet Group trên giao diện AWS console theo các bước dưới đây:

1. Truy cập Amazon RDS [console](https://console.aws.amazon.com/rds/)
2. Tại bảng điều hướng, chọn **Subnet groups** và click **Create DB Subnet Group**.

    ![Create EC2 SG](/images/preparation/4/1.png)

3. Tại mục **Name** và **Description**, nhập tên và mô tả cho DB Subnet Group của bạn.
4. Tại mục **VPC**, chọn VPC mà bạn muốn tạo DB Subnet Group.

    ![Create EC2 SG](/images/preparation/4/2.png)

5. Chọn các subnet mà bạn muốn đưa vào DB Subnet Group. Đảm bảo rằng subnet bạn chọn phải nằm ít nhất trên 2 Az
6. Click **Create**.

    ![Create EC2 SG](/images/preparation/4/3.png)

***DB Subnet Group sẽ được tạo và hiển thị trong danh sách DB Subnet Group***

**Dưới đây là một số lưu ý thêm khi bạn tạo DB Subnet Group:**
- Bạn chỉ có thể tạo DB Subnet Group tại cùng VPC với cơ sở dữ liệu bạn đã tạo.
- Bạn phải có ít nhất 1 subnet ở mỗi AZ bạn muốn triển khai DB instance của bạn. 
- Bạn chỉ có thể cấu hình DB subnet group 1 lần khi khởi tạo. Nếu bạn muốn thay đổi subnet trong DB subnet group của bạn, bạn phải tạo lại 1 DB subnet group mới.
- Bạn có thể sử dụng DB subnet group để tạo DB instance ở bất cứ AZ nào nằm trong VPC