---
title : "Mở rộng theo chiều dọc"
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Tổng quan

Bạn đã mở rộng `rdspg-lab-db` theo chiều dọc ở bước [3.4](../../3-performancemonitoringandoptimization/3-4-makeyourloadtestrunfaster/): đổi instance class từ `db.t4g.micro` sang `db.t4g.xlarge`, thông lượng của bài kiểm tra tải tăng từ khoảng 337 lên khoảng 2.780 giao dịch mỗi giây, rồi đổi class lại.

Trang này tóm tắt những điều cần biết trước khi scale up hoặc scale down trong production.

| Thay đổi | Gián đoạn | Cách giảm gián đoạn |
|---|---|---|
| Instance class (CPU, bộ nhớ) | Vài phút, DB instance khởi động lại | Multi-AZ: RDS đổi standby trước, rồi failover (khoảng 1 phút) |
| Dung lượng lưu trữ (tăng) | Không | Storage autoscaling tự tăng dung lượng |
| IOPS và throughput (`gp3`, `io2`) | Không | — |
| Loại lưu trữ (`gp2` → `gp3`) | Không, nhưng hiệu năng có thể giảm trong lúc đổi | — |

{{% notice note %}}
Bạn có thể tăng dung lượng lưu trữ nhưng không thể giảm. Sau mỗi lần đổi lưu trữ, bạn phải đợi ít nhất 6 giờ, hoặc đến khi quá trình tối ưu lưu trữ xong, mới đổi được lần tiếp theo.
{{% /notice %}}

#### Các bước

1. Chọn DB instance, rồi **Modify**.
2. Trong **Instance configuration**, chọn **Instance type** mới. Trong **Storage**, đổi dung lượng hoặc IOPS nếu cần.
3. Chọn **Continue**, kiểm tra **Summary of modifications**, rồi chọn **Apply immediately** hoặc **Apply during the next scheduled maintenance window**.
4. Chọn **Modify DB instance** và đợi đến khi trạng thái trở lại **Available**.

{{% notice tip %}}
Hãy theo dõi CloudWatch Database Insights vài ngày trước khi chọn instance class nhỏ hơn. Chỉ scale down khi mức sử dụng CPU và tải cơ sở dữ liệu luôn thấp, và dữ liệu vẫn nằm vừa trong bộ nhớ.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Đổi instance class và lưu trữ bằng AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.small \
    --allocated-storage 30 \
    --max-allocated-storage 100 \
    --apply-immediately
```
`--max-allocated-storage` bật storage autoscaling đến tối đa 100 GiB.
{{% /expand %}}
