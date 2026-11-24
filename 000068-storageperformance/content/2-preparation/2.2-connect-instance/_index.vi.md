---
title : "Kết nối tới instance"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---


1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances**, chọn `FCJ-StoragePerf-Instance` rồi chọn **Connect**.

2. Tại trang **Connect to instance**, (1) chọn **SSM Session Manager**, rồi (2) chọn **Connect**.

   ![Kết nối bằng Session Manager](/images/2-preparation/2.2-connect.png)

3. Một terminal mở ra trên trình duyệt với user `ssm-user`. Mở login shell để nạp các biến của bài lab, và chuyển về thư mục home:

   ```bash
   bash -l
   cd ~
   ```

4. Kiểm tra môi trường:

   ```bash
   echo $bucket; ls /ebs/tutorial/data-1m | wc -l; du -sh /ebs/tutorial/data-1m; df -h /ebs /efs
   ```

   - (1) Biến `$bucket` chứa tên bucket của bài lab.
   - (2) Thư mục `/ebs/tutorial/data-1m` có 2.000 file, khoảng 2 GB, và EFS đã được mount tại `/efs`.

   ![Kiểm tra môi trường](/images/2-preparation/2.2-verify.png)

{{% notice note %}}
Ảnh chụp terminal trong bài được hiển thị với nền sáng cho dễ đọc. Terminal Session Manager của bạn sẽ có nền tối.
{{% /notice %}}

{{% notice tip %}}
Nếu phiên kết nối bị đóng, hãy kết nối lại và chạy `bash -l` và `cd ~` trước khi làm tiếp. Các giá trị `aws configure set` ở phần 3 được lưu trong `~/.aws/config` nên vẫn còn.
{{% /notice %}}
