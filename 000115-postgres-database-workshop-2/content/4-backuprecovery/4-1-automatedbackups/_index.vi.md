---
title : "Sao lưu tự động (automated backup)"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Tổng quan

RDS chụp snapshot đầy đủ của ổ lưu trữ ngay sau khi tạo DB instance, sau đó mỗi ngày một snapshot trong **backup window**. Giữa các snapshot, RDS tải nhật ký giao dịch (WAL) lên Amazon S3 mỗi 5 phút. Kết hợp lại, bạn có thể khôi phục cơ sở dữ liệu về bất kỳ giây nào giữa **earliest restorable time** và **latest restorable time**.

#### Xem automated backup

1. Trong [Amazon RDS console](https://console.aws.amazon.com/rds/home#databases:), chọn `rdspg-lab-db`, rồi mở tab **Maintenance & backups**:
    - (1) **Automated backups** đang bật với thời gian lưu **1 ngày**, giá trị bạn chọn khi tạo cơ sở dữ liệu.
    - (2) **Latest restore time** thường cách hiện tại chưa đến 5 phút.
    - (3) **Backup window** là khoảng 30 phút sao lưu hằng ngày, tính theo UTC. RDS chọn ngẫu nhiên vì bạn chưa đặt.
    - (4) **Snapshots** liệt kê các automated snapshot: snapshot đầu tiên sau khi tạo cơ sở dữ liệu, các snapshot của lần nâng cấp phiên bản phụ và chính, và `rds:preupgrade-...`, snapshot RDS chụp ngay trước khi nâng cấp phiên bản chính ở bước 2.2.

    ![Maintenance and backups](/images/4/4-1/1.png)

#### (Tùy chọn) Kiểm tra bản sao lưu bằng AWS CLI

2. Mở **AWS CloudShell** và chạy:

    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBInstances[0].[BackupRetentionPeriod,PreferredBackupWindow,LatestRestorableTime]' \
        --output table

    aws rds describe-db-snapshots \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBSnapshots[].[DBSnapshotIdentifier,SnapshotType,Status]' \
        --output table
    ```

    - (1) Thời gian lưu (`1`), backup window và latest restorable time theo UTC giống trên console.
    - (2) Mọi snapshot đều có loại `automated`. Snapshot do bạn tự chụp có loại `manual`.

    ![AWS CLI](/images/4/4-1/2.png)

{{% notice note %}}
Bạn không thể tự xóa automated snapshot. RDS xóa nó khi quá thời gian lưu. Khi xóa DB instance, bạn có thể chọn giữ automated backup cho phần còn lại của thời gian lưu.
{{% /notice %}}
