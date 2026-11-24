---
title : "Triển khai CloudFormation template"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


1. Tải template [fcj-storage-performance.yaml](/files/fcj-storage-performance.yaml) về máy.

2. Mở [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/) và chọn Region bạn dùng cho bài lab. Chọn **Create stack** → **With new resources (standard)**.

3. Tại trang **Create stack**:
   - (1) Chọn **Upload a template file**.
   - (2) Chọn **Choose file** rồi chọn file `fcj-storage-performance.yaml`.
   - (3) Chọn **Next**.

   ![Upload template](/images/2-preparation/2.1-upload-template.png)

4. Tại trang **Specify stack details**:
   - (1) **Stack name**: `FCJ-StoragePerf`.
   - (2) **InstanceType**: giữ `c7g.large`.
   - (3) **PermissionsBoundaryArn**: để trống. Chỉ điền nếu tài khoản của bạn bắt buộc gắn permissions boundary cho IAM role mới (tài khoản lab dùng chung khi chụp ảnh có yêu cầu này).
   - Chọn **Next**.

   ![Stack details](/images/2-preparation/2.1-stack-details.png)

5. Tại trang **Configure stack options**, giữ mặc định. Ở cuối trang, (1) tick **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, rồi (2) chọn **Next**.

   ![Capabilities](/images/2-preparation/2.1-capabilities.png)

6. Tại trang **Review and create**, chọn **Submit**.

   ![Submit](/images/2-preparation/2.1-submit.png)

7. Chờ stack chuyển sang **CREATE_COMPLETE**, khoảng 3 phút. Instance tạo xong các file mẫu trên EBS và mount EFS rồi mới báo thành công cho CloudFormation.

   ![CREATE_COMPLETE](/images/2-preparation/2.1-complete.png)

8. Mở tab (1) **Outputs**. (2) Ghi lại tên bucket, file system ID và instance ID.

   ![Outputs](/images/2-preparation/2.1-outputs.png)

Template tạo các tài nguyên sau:

| Tài nguyên | Tên | Ghi chú |
|---|---|---|
| VPC, public subnet, internet gateway | `FCJ-StoragePerf-*` | `10.40.0.0/16`, một subnet ở Availability Zone đầu tiên |
| S3 gateway endpoint | | Lưu lượng S3 từ instance đi trong mạng AWS |
| Security group | `FCJ-StoragePerf-Instance-SG`, `FCJ-StoragePerf-EFS-SG` | Instance không có rule inbound; EFS chỉ nhận NFS (2049) từ instance |
| EC2 instance | `FCJ-StoragePerf-Instance` | Amazon Linux 2023 arm64, ổ root 30 GB, ổ dữ liệu 20 GB mount tại `/ebs` chứa 2.000 file 1 MB |
| EFS file system | `FCJ-StoragePerf-EFS` | General Purpose, Bursting, mount tại `/efs` |
| S3 bucket | `fcj-storageperf-labbucket-...` | Bật Block Public Access |
| IAM role | `FCJ-StoragePerf-EC2-Role-<region>` | Session Manager, và quyền đọc/ghi chỉ trên bucket của bài lab |
