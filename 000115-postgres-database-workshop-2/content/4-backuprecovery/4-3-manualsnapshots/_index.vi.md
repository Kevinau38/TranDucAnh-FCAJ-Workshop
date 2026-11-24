---
title : "Snapshot thủ công"
weight : 3
chapter : false
pre : " <b> 4.3. </b> "
---

#### Tổng quan

**Manual snapshot** là bản sao lưu bạn tự chụp, ví dụ ngay trước một lần phát hành hay một thay đổi dữ liệu có rủi ro. RDS giữ nó đến khi bạn xóa, kể cả sau khi DB instance đã bị xóa. Snapshot chứa toàn bộ DB instance với mọi cơ sở dữ liệu, không chỉ một cơ sở dữ liệu.

#### Chụp manual snapshot

1. Trong [danh sách cơ sở dữ liệu](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Chọn `rdspg-lab-db`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Take snapshot**.

    ![Take snapshot](/images/4/4-3/1.png)

2. Trên trang **Take DB snapshot**:
    - (1) **Snapshot type**: **DB instance**, `rdspg-lab-db`
    - (2) **Snapshot name**: `rdspg-lab-db-manual-snapshot`
    - (3) Chọn **Take snapshot**.

    ![Snapshot name](/images/4/4-3/2.png)

Snapshot mất vài phút. Trong lúc đó, trạng thái của `rdspg-lab-db` là **Backing-up** và cơ sở dữ liệu vẫn hoạt động bình thường. Đợi đến khi trạng thái của snapshot là **Available**.

{{% notice tip %}}
Với DB instance Single-AZ, vài giây đầu khi chụp snapshot có thể làm I/O chậm lại chốc lát. Với DB instance Multi-AZ, RDS chụp snapshot từ standby, nên primary không bị ảnh hưởng.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Chụp manual snapshot bằng AWS CLI" %}}
```bash
aws rds create-db-snapshot \
    --db-instance-identifier rdspg-lab-db \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot

aws rds wait db-snapshot-available \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot
```
{{% /expand %}}
