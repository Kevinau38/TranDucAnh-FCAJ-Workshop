---
title : "Chuyển sang Multi-AZ DB cluster bằng read replica"
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Tổng quan

**Multi-AZ DB cluster** có một DB instance writer và **hai standby đọc được**, mỗi instance nằm ở một Availability Zone khác nhau. So với Multi-AZ DB instance (một primary và một standby không đọc được), cluster mang lại:

| | Multi-AZ DB instance | Multi-AZ DB cluster |
|---|---|---|
| DB instance | 1 primary + 1 standby | 1 writer + 2 standby đọc được |
| Đọc từ standby | Không | Có, qua **reader endpoint** |
| Thời gian failover thường gặp | 60 đến 120 giây | Dưới 35 giây |
| Lưu trữ | EBS | SSD NVMe cục bộ + EBS |
| Instance class | Tất cả | `m5d`, `m6gd`, `m6id`, `r5d`, `r6gd`, `r6id`, `c6gd`... |

Để chuyển một DB instance Single-AZ hoặc Multi-AZ sang Multi-AZ DB cluster với thời gian gián đoạn ngắn, bạn tạo một **Multi-AZ DB cluster read replica** từ DB instance, chờ nó bắt kịp dữ liệu, dừng ghi, rồi **promote** cluster.

{{% notice warning %}}
Multi-AZ DB cluster cần subnet ở **ba** Availability Zone trong DB subnet group. Nếu `rdspg-lab-subnet-group` chỉ có hai subnet, hãy thêm private subnet thứ ba trước (xem ghi chú bên dưới). Cluster chạy ba DB instance không thuộc free tier: hãy xoá nó ngay sau khi xong bước 5.5.
{{% /notice %}}

{{% notice note %}}
Nếu console báo *Can't create a Multi AZ DB cluster because there aren't enough subnets available*, hãy thêm một private subnet ở Availability Zone thứ ba bằng AWS CloudShell, rồi chọn lại **Create read replica**:

```bash
VPC=<ID của rdspg-lab-vpc>
RT=<ID của private route table>
PRI3=$(aws ec2 create-subnet --vpc-id $VPC --cidr-block 10.0.160.0/20 \
    --availability-zone <AZ thứ ba> \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=rdspg-lab-subnet-private3}]' \
    --query Subnet.SubnetId --output text)
aws ec2 associate-route-table --route-table-id $RT --subnet-id $PRI3
aws rds modify-db-subnet-group --db-subnet-group-name rdspg-lab-subnet-group \
    --subnet-ids <private subnet 1> <private subnet 2> $PRI3
```
{{% /notice %}}

#### Tạo Multi-AZ DB cluster read replica

1. Trong danh sách database, chọn `rdspg-lab-db`, rồi chọn **Actions** > **Create read replica**, như ở bước 5.1.

2. Kéo xuống phần **Availability** trước, vì lựa chọn này thay đổi phần còn lại của trang:
    - (1) Chọn **Multi-AZ DB cluster deployment (3 instances)**.

    ![Multi-AZ DB cluster](/images/5/5-4/1.png)

3. Kéo ngược lên:
    - (1) **DB cluster identifier**: `rdspg-lab-cluster`
    - (2) **DB instance class**: **Compute optimized classes**
    - (3) **Instance type**: `db.c6gd.medium`, class nhỏ nhất cho Multi-AZ DB cluster

    ![Cấu hình cluster](/images/5/5-4/2.png)

4. Ở phần **Connectivity**, kiểm tra **DB subnet group** là `rdspg-lab-subnet-group`, **Public access** là **Not publicly accessible**, và security group là `rds-sg`. Chọn **Create read replica**.

5. Trong danh sách database:
    - (1) `rdspg-lab-cluster` xuất hiện dưới `rdspg-lab-db` với role **Multi-AZ DB cluster** và size **3 instances**.
    - (2) RDS tạo ba **Reader instance** `db.c6gd.medium` ở ba Availability Zone.

    ![Đang tạo cluster](/images/5/5-4/3.png)

    Việc tạo cluster mất khoảng 20 đến 30 phút.

6. Khi **Status** của `rdspg-lab-cluster` là **Available**, chọn cluster và mở tab **Connectivity & security**:
    - (1) Endpoint **Writer** (`rdspg-lab-cluster.cluster-...`) luôn trỏ tới DB instance writer. Dùng nó cho cả đọc và ghi.
    - (2) Endpoint **Reader** (`rdspg-lab-cluster.cluster-ro-...`) phân phối kết nối tới các standby đọc được.

    ![Endpoint của cluster](/images/5/5-4/4.png)

7. Mở tab **Replication**:
    - (1) Cluster có role **Multi-AZ DB replica cluster**, các DB instance nằm ở ba Availability Zone, nguồn là `rdspg-lab-db`, trạng thái **Replicating**.

    ![Replication của cluster](/images/5/5-4/5.png)

#### Promote Multi-AZ DB cluster

Trong một lần chuyển đổi thật, lúc này bạn dừng ghi vào `rdspg-lab-db`, chờ **Replica lag** về 0, rồi promote cluster và trỏ ứng dụng sang writer endpoint.

8. Trong danh sách database:
    - (1) Chọn `rdspg-lab-cluster`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Promote**.

    ![Promote cluster](/images/5/5-4/6.png)

9. Trên trang **Promote read replica: rdspg-lab-cluster**:
    - (1) Đọc thông báo: replication dừng ngay khi bắt đầu promote.
    - (2) Chọn **Promote read replica**.

    ![Promote read replica](/images/5/5-4/7.png)

10. **Status** của `rdspg-lab-cluster` chuyển sang **Promoting**.

    ![Promoting](/images/5/5-4/8.png)

11. Sau vài phút, việc promote hoàn tất:
    - (1) `rdspg-lab-cluster` không còn nằm dưới `rdspg-lab-db`. Giờ đây nó là một **Multi-AZ DB cluster** độc lập, nhận được lệnh ghi. RDS tạo bản backup đầu tiên (**Backing-up**), rồi **Status** chuyển thành **Available**.
    - (2) Role của `rdspg-lab-db` đổi từ **Primary** thành **Instance**, vì nó không còn replica nào.

    ![Cluster sau khi promote](/images/5/5-4/9.png)

{{% notice tip %}}
Bạn cũng có thể chuyển DB instance sang Multi-AZ DB cluster từ snapshot (**Restore snapshot** > **Multi-AZ DB cluster**). Cách dùng read replica giữ thời gian gián đoạn chỉ trong vài phút promote.
{{% /notice %}}

#### (Tuỳ chọn) AWS CLI

{{%expand "Tạo và promote Multi-AZ DB cluster read replica bằng AWS CLI" %}}
```bash
aws rds create-db-cluster \
    --db-cluster-identifier rdspg-lab-cluster \
    --replication-source-identifier <ARN của rdspg-lab-db> \
    --engine postgres \
    --db-cluster-instance-class db.c6gd.medium \
    --storage-type gp3 \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID của rds-sg> \
    --no-publicly-accessible

aws rds wait db-cluster-available --db-cluster-identifier rdspg-lab-cluster

aws rds promote-read-replica-db-cluster --db-cluster-identifier rdspg-lab-cluster
```
{{% /expand %}}
