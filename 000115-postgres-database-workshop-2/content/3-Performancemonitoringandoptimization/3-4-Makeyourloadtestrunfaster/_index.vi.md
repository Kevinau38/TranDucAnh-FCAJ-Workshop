---
title : "Làm bài kiểm tra tải chạy nhanh hơn"
weight : 4
chapter : false
pre : " <b> 3.4. </b> "
---

#### Tổng quan

Database Insights cho thấy bài kiểm tra tải bị giới hạn bởi CPU. Để gỡ điểm nghẽn này, bạn **scale up** DB instance, còn gọi là *mở rộng theo chiều dọc*: đổi instance class sang loại có nhiều vCPU và bộ nhớ hơn, chạy lại cùng bài kiểm tra tải, rồi thu nhỏ lại.

| Instance class | vCPU | Bộ nhớ |
|---|---|---|
| `db.t4g.micro` | 2 | 1 GiB |
| `db.t4g.xlarge` | 4 | 16 GiB |

`db.t4g.xlarge` không thuộc free tier và tính phí theo giờ, vì vậy chỉ giữ nó trong thời gian kiểm tra.

{{% notice warning %}}
Đổi instance class sẽ khởi động lại cơ sở dữ liệu. Với **Apply immediately**, cơ sở dữ liệu ngừng phục vụ vài phút. Trong môi trường production, hãy lên lịch thay đổi này trong maintenance window, hoặc dùng triển khai Multi-AZ để rút ngắn thời gian gián đoạn.
{{% /notice %}}

#### Scale up DB instance

1. Chọn `rdspg-lab-db`, rồi **Modify**. Trong **Instance configuration**:
    - (1) **Instance type**: `db.t4g.xlarge` (4 vCPU, 16 GiB RAM).

    ![Instance type](/images/3/3-4/1.png)

2. Chọn **Continue**:
    - (1) Kiểm tra thay đổi: **DB instance class** `db.t4g.micro` → `db.t4g.xlarge`.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Summary of modifications](/images/3/3-4/2.png)

3. Đợi khoảng 10 phút cho đến khi cột **Size** hiện `db.t4g.xlarge` và **Status** là **Available**.

#### Chạy lại kiểm tra tải

4. Trên App server, chạy đúng lệnh ở bước 3.2:

    ```bash
    pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
    ```

    - (1) Độ trễ trung bình giảm từ khoảng 148 ms xuống khoảng **18 ms**.
    - (2) Thông lượng tăng từ khoảng 337 lên khoảng **2.780 tps**, nhanh hơn khoảng 8 lần.

    ![Load test on db.t4g.xlarge](/images/3/3-4/3.png)

Mức cải thiện lớn hơn nhiều so với số vCPU tăng thêm. Với 16 GiB bộ nhớ, toàn bộ dữ liệu thử (khoảng 750 MB) nằm gọn trong shared buffers, nên PostgreSQL gần như không phải đọc từ bộ lưu trữ. Ngoài ra `db.t4g.xlarge` khởi đầu với nhiều CPU credit hơn `db.t4g.micro`.

#### Thu nhỏ lại

5. Để giữ chi phí thấp, sửa `rdspg-lab-db` lần nữa và đổi instance type về `db.t4g.micro`, với **Apply immediately**.

{{% notice tip %}}
Scale up là cách nhanh nhất để gỡ điểm nghẽn CPU hoặc bộ nhớ, nhưng có giới hạn và tốn thêm chi phí mỗi giờ. Trước khi scale up, hãy dùng Database Insights để chắc rằng điểm nghẽn thực sự nằm ở instance, chứ không phải do thiếu index hay câu truy vấn viết chưa tốt.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Đổi instance class bằng AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.xlarge \
    --apply-immediately

# Sau khi kiểm tra tải
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.micro \
    --apply-immediately
```
{{% /expand %}}
