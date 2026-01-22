---
title : "Tạo EC2 instance bastion h "
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 2.6. </b> "
---

{{% notice note %}}
Chúng ta sẽ kết nối đến instance RDS PostgreSQL bằng pgAdmin4 thông qua EC2 bastion host
{{% /notice %}}
#### Tạo EC2 instance cho bastion host, làm theo các bước dưới đây
1. Truy cập giao diện **Amazon EC2** [console](https://console.aws.amazon.com/ec2/).
2. Click **Launch Instance**.

    ![Create bastion host](/images/preparation/ec2/1.png)
3. Đặt tên cho EC2 bastion.

    ![Create bastion host](/images/preparation/bastionhost/ec2bastion.png)

4. Chọn **AMI**. Bạn có thể chọn **Amazon Linux 2** cho app server
    ![Create bastion host](/images/preparation/bastionhost/ec2bastion1.png)
5. Choose an **instance type**. The instance type that you choose will depend on the requirements of your app server. For example, if you are running a high-traffic website, you will need a larger instance type with more CPU and memory.
5. Chọn **instance type**. Instance type bạn chọn sẽ tùy thuộc vào yêu cầu của app server. Ví dụ: nếu bạn đang chạy một trang web có lưu lượng truy cập cao, bạn sẽ cần instance type lớn hơn với nhiều CPU và bộ nhớ hơn.
    ![Create bastion host](/images/preparation/bastionhost/ec2bastion2.png)

6. Tại mục **Key Pair**, chọn keyair mà bạn đã tạo hoặc click vào **Create new key pair** để tạo 1 keypair mới
    ![Create bastion host](/images/preparation/bastionhost/ec2bastion3.png)

7.  Tại mục **Network settings**
    - Chọn **VPC** chứa EC2 app server
    - Chọn **Subnet**
    - Chọn **Enable** Auto-assign public IP
    - Thêm **security group** cho EC2 app server mà bạn đã tạo trước đó. *SG là một 1 firewall kiểm soát lưu lượng đi và đến instance của bạn*
    ![Create bastion host](/images/preparation/bastionhost/ec2bastion4.png)

8. **Review** và **launch** instance

    ![Create bastion host](/images/preparation/bastionhost/ec2bastion5.png)

