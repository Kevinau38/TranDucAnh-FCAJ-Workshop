---
title : "Sửa tham số"
weight : 2
chapter : false
pre : " <b> 6.2. </b> "
---

#### Đổi hai tham số

1. Chọn `rdspg-lab-pg18`, rồi chọn **Edit**. Nhập `log_connections` vào **Filter Parameters**:
    - (1) Chỉ còn `log_connections`. Trong PostgreSQL 18, **Data type** của nó là **list**: bạn chọn những giai đoạn nào của kết nối được ghi log.
    - (2) Trong **Value**, nhập `receipt,authentication,authorization`.

    ![log_connections](/images/6/6-2/1.png)

{{% notice info %}}
Trước PostgreSQL 18, `log_connections` là kiểu Boolean (`1` hoặc `0`). Giá trị `receipt,authentication,authorization` có tác dụng như giá trị `1` cũ. Giá trị `all` ghi thêm thời gian thiết lập của mỗi kết nối.
{{% /notice %}}

2. Thay bộ lọc bằng `log_min_duration_statement`:
    - (1) Chỉ còn `log_min_duration_statement`. Giá trị tính bằng mili giây.
    - (2) Trong **Value**, nhập `4000`.
    - (3) Chọn **Save Changes**. RDS lưu cả hai giá trị.

    ![log_min_duration_statement](/images/6/6-2/2.png)

#### So sánh với parameter group mặc định

3. Chọn **Parameter groups**, chọn `rdspg-lab-pg18`, rồi **Actions** > **Compare**:
    - (1) Chỉ hai tham số bạn đã đổi khác với `default.postgres18`.

    ![Compare](/images/6/6-2/3.png)

    Chọn **Close**.

#### (Tùy chọn) AWS CLI

{{%expand "Sửa tham số bằng AWS CLI" %}}
```bash
aws rds modify-db-parameter-group \
    --db-parameter-group-name rdspg-lab-pg18 \
    --parameters \
        "ParameterName=log_connections,ParameterValue='receipt,authentication,authorization',ApplyMethod=immediate" \
        "ParameterName=log_min_duration_statement,ParameterValue=4000,ApplyMethod=immediate"

aws rds describe-db-parameters \
    --db-parameter-group-name rdspg-lab-pg18 \
    --source user \
    --query 'Parameters[].[ParameterName,ParameterValue,ApplyType]' \
    --output table
```
{{% /expand %}}
