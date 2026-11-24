---
title : "Nâng cấp phiên bản engine"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### Tổng quan

Bạn nâng cấp `rdspg-lab-db` qua hai bước:

| Bước | Từ | Lên | RDS làm gì |
|---|---|---|---|
| Nâng cấp phiên bản phụ | 17.9 | 17.11 | Chụp snapshot, nâng cấp engine và khởi động lại cơ sở dữ liệu |
| Nâng cấp phiên bản chính | 17.11 | 18.6 | Chụp snapshot trước nâng cấp, chạy `pg_upgrade`, chuyển sang parameter group `default.postgres18` và khởi động lại cơ sở dữ liệu |

Trong mỗi lần nâng cấp, cơ sở dữ liệu ngừng phục vụ vài phút.

{{% notice warning %}}
Nếu DB instance có read replica, với nâng cấp phiên bản phụ hãy nâng cấp replica trước instance nguồn. Với nâng cấp phiên bản chính, RDS nâng cấp các read replica cùng Region đồng thời với instance nguồn.
{{% /notice %}}

#### Nâng cấp phiên bản phụ (17.9 → 17.11)

1. Chọn `rdspg-lab-db`, rồi chọn **Modify** (như ở bước 2.1). Trên trang **Modify DB instance: rdspg-lab-db**, trong **Settings**:
    - (1) Mở danh sách **DB engine version**. Danh sách chỉ hiện các phiên bản mà 17.9 có thể nâng lên.
    - (2) Chọn **17.11**.

    ![DB engine version](/images/2/2-2/1.png)

2. Kéo xuống phần **Maintenance**:
    - (1) Chọn **Enable auto minor version upgrade**, để RDS tự cài các phiên bản phụ sau này của PostgreSQL 18.
    - (2) Giữ nguyên **DB instance maintenance window**.

    ![Maintenance](/images/2/2-2/2.png)

3. Chọn **Continue**. Trên trang tóm tắt:
    - (1) Kiểm tra thay đổi: **Engine version** 17.9 → 17.11, **Auto minor version upgrade** No → Yes.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Summary of modifications](/images/2/2-2/3.png)

{{% notice info %}}
Nếu chọn **Apply during the next scheduled maintenance window**, RDS đợi đến maintenance window mới áp dụng thay đổi. Trong workshop này bạn áp dụng ngay để xem kết quả.
{{% /notice %}}

4. **Status** của `rdspg-lab-db` chuyển sang **Backing-up**, rồi **Upgrading**.

    ![Upgrading](/images/2/2-2/4.png)

5. Sau khoảng 10 phút, **Status** trở lại **Available**. Mở tab **Logs & events**:
    - (1) Trong **Recent events**, sự kiện *Database instance engine minor version upgrade complete. Previous version: 17.9.R2. New version: 17.11.R1.* xác nhận đã nâng cấp xong.

    ![Minor version upgrade event](/images/2/2-2/5.png)

#### Nâng cấp phiên bản chính (17.11 → 18.6)

6. Chọn **Modify** lần nữa. Trong **Settings**:
    - (1) **DB engine version**: **18.6**. Từ 17.11, đích nâng cấp phiên bản chính duy nhất là 18.6.

    ![DB engine version 18.6](/images/2/2-2/6.png)

7. Kéo xuống **Additional configuration** > **Database options**:
    - (1) **DB parameter group** đã đổi thành `default.postgres18`. Mỗi phiên bản chính cần parameter group thuộc family riêng.

    ![DB parameter group](/images/2/2-2/7.png)

{{% notice note %}}
Nếu bạn dùng parameter group tùy chỉnh, hãy tạo một bản cho family mới (ví dụ `postgres18`) và chọn nó ở đây. Bạn làm việc với parameter group tùy chỉnh ở chương 6.
{{% /notice %}}

8. Chọn **Continue**:
    - (1) Đọc cảnh báo: nâng cấp phiên bản chính không thể hoàn tác, và cơ sở dữ liệu ngừng phục vụ vài phút.

    ![Upgrade warning](/images/2/2-2/8.png)

9. Trên cùng trang:
    - (1) Kiểm tra thay đổi: **Engine version** 17.11 → 18.6, **DB parameter group** `default.postgres17` → `default.postgres18`.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Summary of the major version upgrade](/images/2/2-2/9.png)

10. **Status** chuyển sang **Upgrading**. Nâng cấp phiên bản chính mất khoảng 15 đến 20 phút.

    ![Major version upgrading](/images/2/2-2/10.png)

{{% notice info %}}
Trước khi nâng cấp phiên bản chính, RDS chạy kiểm tra trước (pre-check) và tự chụp snapshot tên `rds:preupgrade-rdspg-lab-db-17-11-to-18-6-...`. Nếu nâng cấp thất bại, bạn có thể khôi phục cơ sở dữ liệu từ snapshot này.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Nâng cấp phiên bản engine bằng AWS CLI" %}}
```bash
# Liệt kê các phiên bản mà DB instance có thể nâng lên
aws rds describe-db-engine-versions \
    --engine postgres --engine-version 17.11 \
    --query 'DBEngineVersions[0].ValidUpgradeTarget[].[EngineVersion,IsMajorVersionUpgrade]' \
    --output table

# Nâng cấp phiên bản phụ
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --engine-version 17.11 \
    --auto-minor-version-upgrade \
    --apply-immediately

# Nâng cấp phiên bản chính
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --engine-version 18.6 \
    --db-parameter-group-name default.postgres18 \
    --allow-major-version-upgrade \
    --apply-immediately
```
{{% /expand %}}
