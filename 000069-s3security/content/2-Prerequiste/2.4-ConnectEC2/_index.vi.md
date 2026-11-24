---
title : "Kết nối bằng Session Manager"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---


#### Kết nối instance bằng Session Manager

1. Trong EC2 console, mở **Instances**, chọn `SID-security-instance` và chọn **Connect**.
2. Tại tab **Session Manager** (hiển thị là **SSM Session Manager**), chọn **Connect**.

![Kết nối bằng Session Manager](/images/2-prerequisite/2.4-connect.png)

3. Một terminal mở ra trên trình duyệt. Nạp biến môi trường của lab và liệt kê bucket:

```bash
cd ~ && source /etc/profile.d/sid-lab.sh && echo $bucket && aws s3 ls s3://$bucket --recursive
```

Instance in ra tên bucket và object thử nghiệm `app1/file1`.

![Shell của Session Manager](/images/2-prerequisite/2.4-shell.png)

{{% notice tip %}}
Nếu session bị đóng, hãy kết nối lại và chạy `cd ~ && source /etc/profile.d/sid-lab.sh` trước khi chạy các lệnh tiếp theo. File này khai báo biến `$bucket` và Region mặc định cho AWS CLI.
{{% /notice %}}
