---
title : "Tạo EC2 instance cho App Server"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---

#### Tạo EC2 instance cho App Server theo các bước sau:
1. Truy cập **Amazon EC2** [console](https://console.aws.amazon.com/ec2/).
2. Click **Launch Instance**.

    ![Create EC2 instance](/images/preparation/ec2/1.png)

3. Chọn **AMI**. Với App Server, bạn có thể chọn **Linux AMI**, chẳng hạn như **Amazon Linux 2**.
4. Chọn **instance type**. Loại instance mà bạn chọn sẽ phụ thuộc vào yêu cầu của App Server. Ví dụ, nếu bạn đang chạy website với lưu lượng truy cập cao, bạn sẽ cần instance type lớn với nhiều CPU và memory hơn.

    ![Create EC2 instance](/images/preparation/ec2/2.png)

    ![Create EC2 instance](/images/preparation/ec2/4.png)

5. Tại mục **Key Pair**, chọn keypair mà bạn đã tạo trước đó hoặc click **Create new key pair** để tạo 1 keypair mới.

    ![Create EC2 instance](/images/preparation/ec2/5.png)

6. Định cấu hình chi tiết của instance. Điều này bao gồm những thứ như số lượng instance sẽ khởi chạy, cấu hình mạng và cấu hình lưu trữ.
7.  Tại mục **Network settings**
    - Chọn **VPC** chứa EC2 App Server
    - Chọn **Subnet**
    - **Enable** Auto-assign public IP
    - Thêm **security group** cho EC2 App Server mà bạn đã tạo trước đó. Security group là 1 tường lửa kiểm soát lưu lượng đi với đến EC2 của bạn

    ![Create EC2 instance](/images/preparation/ec2/6.png)

8. **Đánh giá** và **khởi tạo** instance
    ![Create 7C2 instance](/images/preparation/ec2/7.png)

*Sau khi khởi chạy instance, bạn có thể kết nối với instance đó bằng ứng dụng SSH client, chẳng hạn như MobaXterm hoặc PuTTY. Sau khi kết nối, bạn có thể cài đặt app server và triển khai ứng dụng của mình.* 