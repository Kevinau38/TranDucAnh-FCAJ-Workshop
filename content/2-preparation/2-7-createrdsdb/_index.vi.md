---
title : "Tạo RDS PostgreSQL DB"
date :  "`r Sys.Date()`" 
weight : 7
chapter : false
pre : " <b> 2.7. </b> "
---


#### Để tạo 1 RDS PostgreSQL DB, làm theo các bước dưới đây
 
1. Truy cập đến giao diện **Amazon RDS** [console](https://console.aws.amazon.com/rds/).
    ![Create EC2 SG](/images/preparation/db/1.png)

2. Click **Create database**.
3. Tại mục **Database engine**, chọn **PostgreSQL**.
4. Tại mục **Version**, chọn version PostgreSQL bạn muốn sử dụng.

    ![Create EC2 SG](/images/preparation/db/2.png)

5. Tại mục **Template**, chọn template cho db instance của bạn. Template là cấu hình được khởi tạo trước cho DB instance

    ![Create EC2 SG](/images/preparation/db/3.png)

6. Tại mục **Availability** và **Durability**, DB instance identifier, đặt tên cho Tại mục DB instance.

    ![Create EC2 SG](/images/preparation/db/4.png)

7. Tại mục **DB instance identifier**, nhập một tên duy nhất cho rds instance.
8. Tại mục **Master username**, nhập tên người dùng cho người dùng chính của rds instance. 
9. Tại mục **Master password**, nhập mật khẩu cho người dùng chính của rds instance.

    ![Create EC2 SG](/images/preparation/db/5.png)

10. Tại mục **DB instance class**, chọn instance class mà bạn muốn sử dụng cho db instance của mình. Instance class sẽ xác định số CPU và bộ nhớ được phân bổ cho db instance của bạn.

![Create EC2 SG](/images/preparation/db/6.png)

11. Tại mục **Storage**, chọn dung lượng lưu trữ mà bạn muốn phân bổ cho db instance của mình.

    ![Create EC2 SG](/images/preparation/db/7.png)

12. Tại mục **Connectivity**, chọn **Connect to a EC2 compute resource**, sau đó chọn **App server** instance mà bạn đã tạo.

    ![Create EC2 SG](/images/preparation/db/8.png)

13. Tại mục **DB subnet group**, chọn DB subnet group mà bạn muốn sử dụng cho DB instance của mình.
14. Tại mục **VPC security groups**, chọn security group mà bạn muốn sử dụng cho DB instance của mình.

    ![Create EC2 SG](/images/preparation/db/9.png)

15. Click vào **Additional configuration**, Nhập **5432** cho **database port**

    ![Create EC2 SG](/images/preparation/db/10.png)

16. Tại mục **Database authentication**, chọn phương thức mà bạn muốn sử dụng

    ![Create EC2 SG](/images/preparation/db/11.png)

17. Tại mục **Monitoring**, chọn các tùy chọn giám sát mà bạn muốn sử dụng cho phiên bản cơ sở dữ liệu của mình

    ![Create EC2 SG](/images/preparation/db/12.png)

18. Click vào **Additional configuration**:
- Tại mục **Database option**, điền tên cho cơ sở dữ liệu bạn muốn tạo sau khi khởi chạy RDS PostgreSQL 
- Tại mục **DB parameter group**, giữ cấu hình mặc định

    ![Create EC2 SG](/images/preparation/db/13.png)

19. Tại mục **Backup**, chọn các tùy chọn sao lưu mà bạn muốn sử dụng cho db instance của mình.

    ![Create EC2 SG](/images/preparation/db/14.png)

20. Click **Create database**.

Database instance của bạn sẽ được tạo và hiển thị trong danh sách các database instance. Bạn có thể kết nối với nó bằng ứng dụng PostgreSQL client, chẳng hạn như psql, pgAdmin4. Để thực hiện việc này, bạn sẽ cần hostname, port, tên người dùng và mật khẩu cho db instance của bạn.

**Dưới đây là một số điều cần lưu ý khi tạo cơ sở dữ liệu RDS PostgreSQL:**
{{%expand "Click here" %}}
- Bạn chỉ có thể tạo một RDS PostgreSQL DB trong VPC nằm trong cùng AWS region với công cụ cơ sở dữ liệu mà bạn dự định sử dụng.
- Bạn phải chọn instance class tương thích với phiên bản PostgreSQL mà bạn muốn sử dụng.
- Bạn có thể chọn tạo db instance của mình trong một Availability Zone (AZ) hoặc trong nhiều AZ. Nếu chọn tạo db instance trong nhiều AZ, bạn sẽ có thể tận dụng các tính năng high availability và disaster recovery
- Bạn có thể chọn bật mã hóa cho db instance của mình. Điều này sẽ mã hóa dữ liệu của bạn khi lưu trữ và truyền tải.
- Bạn có thể chọn bật sao lưu cho db instance    của mình. Điều này sẽ cho phép bạn khôi phục cơ sở dữ liệu của mình về thời điểm trước đó nếu có sự cố.
{{% /expand%}}


