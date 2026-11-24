---
title : "Tạo parameter group tùy chỉnh"
weight : 1
chapter : false
pre : " <b> 6.1. </b> "
---

#### Tạo parameter group

1. Trong [Amazon RDS console](https://console.aws.amazon.com/rds/home):
    - (1) Chọn **Parameter groups** ở thanh điều hướng.
    - (2) Chọn **Create parameter group**.

    ![Parameter groups](/images/6/6-1/1.png)

2. Trong **Parameter group details**:
    - (1) **Parameter group name**: `rdspg-lab-pg18`
    - (2) **Description**: `Custom parameters for rdspg-lab-db (PostgreSQL 18)`
    - (3) **Engine type**: **PostgreSQL**
    - (4) **Parameter group family**: `postgres18`. Family phải khớp với phiên bản chính của DB instance.
    - (5) **Type**: **DB Parameter Group**

    Chọn **Create** ở cuối trang.

    ![Parameter group details](/images/6/6-1/2.png)

3. Trên tab **Custom**:
    - (1) `rdspg-lab-pg18` xuất hiện với family `postgres18`. Group mới có giá trị giống group mặc định.

    ![Custom parameter groups](/images/6/6-1/3.png)

{{% notice note %}}
Khi nâng DB instance lên phiên bản chính mới, bạn cần parameter group thuộc family mới. Đó là lý do lần nâng cấp phiên bản chính ở bước 2.2 đổi parameter group của `rdspg-lab-db` thành `default.postgres18`.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Tạo parameter group bằng AWS CLI" %}}
```bash
aws rds create-db-parameter-group \
    --db-parameter-group-name rdspg-lab-pg18 \
    --db-parameter-group-family postgres18 \
    --description "Custom parameters for rdspg-lab-db (PostgreSQL 18)"
```
{{% /expand %}}
