---
title : "Gắn volume vào hai instance"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

1. Chọn `shared-io2`, chọn **Actions** → **Attach volume**. Mở danh sách **Instance**:
   - (1) Volume nằm ở `ap-northeast-1a`.
   - (2) Danh sách có cả `prod-postgres` và `test-postgres`. Console chỉ liệt kê các instance cùng Availability Zone với volume, bất kể chúng thuộc VPC nào.

   ![Các instance cùng zone](/images/3-shared-volume/3.2-instance-list.png)

2. Gắn volume vào instance production:
   - (1) **Instance**: `prod-postgres`.
   - (2) **Device name**: `/dev/sdf`.
   - (3) Chọn **Attach volume**.

   ![Gắn vào prod-postgres](/images/3-shared-volume/3.2-attach-prod.png)

3. Chọn lại `shared-io2`, chọn **Actions** → **Attach volume**:
   - (1) **Instance**: `test-postgres`.
   - (2) **Device name**: `/dev/sdf`.
   - (3) **Attached instances** đã có `prod-postgres`.
   - Chọn **Attach volume**.

   ![Gắn vào test-postgres](/images/3-shared-volume/3.2-attach-test.png)

4. Chọn `shared-io2` và xem tab **Details**:
   - (1) `io2`, 4 GiB, 100 IOPS.
   - (2) **Volume state**: **In-use**.
   - (3) **Multi-Attach enabled**: **Yes**.
   - (4) **Attached resources**: **2 instances**.

   ![Volume gắn vào hai instance](/images/3-shared-volume/3.2-attached.png)

Trên instance Nitro, device name bạn chọn chỉ là nhãn. Trong hệ điều hành, volume hiện ra dưới dạng thiết bị NVMe, trong bài lab là `/dev/nvme1n1`.
