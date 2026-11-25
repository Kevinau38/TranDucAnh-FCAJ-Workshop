---
title : "Tạo volume io2"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

1. Trong EC2 console, mở **Volumes** và chọn **Create volume**.
   - (1) **Volume type**: **Provisioned IOPS SSD (io2)**.
   - (2) **Size**: `4` GiB, mức nhỏ nhất của `io2`.
   - (3) **IOPS**: `100`, mức nhỏ nhất. Bài lab chỉ chuyển vài kilobyte dữ liệu.
   - (4) **Availability Zone**: `ap-northeast-1a`, zone của hai subnet.

   ![Cấu hình volume](/images/3-shared-volume/3.1-volume-settings.png)

2. Cuộn xuống:
   - (1) Chọn **Enable Multi-Attach**.
   - (2) Chọn **Add new tag**, nhập **Key** `Name`, **Value** `shared-io2`.
   - Chọn **Create volume**.

   ![Multi-Attach và tag](/images/3-shared-volume/3.1-multi-attach.png)

Console có hiện ghi chú về Windows Server Failover Clustering. Ghi chú này không áp dụng cho instance Linux.
