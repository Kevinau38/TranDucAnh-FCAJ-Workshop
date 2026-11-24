---
title : "Báo cáo độ bao phủ"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Độ bao phủ (coverage) và mức sử dụng (utilization)

Khi dùng Savings Plans hoặc Reserved Instances (xem workshop [000042](https://000042.awsfcaj.com/)), có hai câu hỏi quan trọng:

| Báo cáo | Câu hỏi | Mục tiêu |
|---|---|---|
| **Coverage** | Bao nhiêu phần sử dụng đủ điều kiện được cam kết bao phủ? Phần còn lại tính giá On-Demand. | Cao, với workload ổn định |
| **Utilization** | Tôi dùng thực tế bao nhiêu phần cam kết đã mua? Phần không dùng là lãng phí. | Gần 100% |

#### Coverage của Savings Plans

1. Trong **Cost Explorer Saved Reports**, mở báo cáo mặc định **Coverage report** (loại *Savings Plans coverage*):
    - (1) Đường dẫn (breadcrumb) cho thấy báo cáo thuộc **Savings Plans**.
    - (2) **On-Demand spend not covered**, **Average coverage** và **Potential monthly savings**. Tài khoản này không có Savings Plan nên coverage trống.
    - (3) **Coverage target (%)** vẽ đường mục tiêu nét đứt trên biểu đồ; **Granularity** là theo ngày hoặc theo tháng.

    ![Coverage của Savings Plans](/images/4/4-1.png)

2. Cuộn xuống:
    - (1) **Savings Plans coverage breakdown**: với mỗi dịch vụ, instance family và Region, phần chi phí được Savings Plans bao phủ và phần On-Demand. Lambda và DynamoDB xuất hiện vì Compute và Database Savings Plans có thể bao phủ chúng.
    - (2) Lọc báo cáo theo linked account, Region, instance family, dịch vụ hoặc cost category.

    ![Chi tiết coverage](/images/4/4-2.png)

#### Coverage của Reserved Instances

3. Quay lại thư viện báo cáo và mở **RI Coverage**:
    - (1) Báo cáo reservation tách theo **Service**: EC2, RDS, ElastiCache, OpenSearch Service, Redshift và các dịch vụ khác.
    - (2) Tài khoản này không có reservation nên không có dữ liệu coverage.

    ![Coverage của RI](/images/4/4-3.png)

**Utilization report** (Savings Plans) và **RI Utilization** dùng tương tự. Hãy xem chúng cùng với phần đề xuất ở chương 2 của workshop 000042 trước khi mua một cam kết mới.
