---
title : "Các loại Savings Plans"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---

#### Bốn loại Savings Plans

1. Mở [Savings Plans console](https://console.aws.amazon.com/costmanagement/home#/savings-plans/overview) (**Billing and Cost Management** > **Savings Plans** > **Overview**):
    - (1) Menu **Savings Plans**: Overview, Inventory, Recommendations, Purchase Analyzer, báo cáo Utilization và Coverage, Purchase Savings Plans và Cart.
    - (2) AWS cung cấp bốn loại Savings Plans.
    - (3) **Purchase a Savings Plan** mở trang mua mà bạn dùng ở chương 3.

    ![Savings Plans overview](/images/1/1-1.png)

| Loại | Giảm giá (tối đa) | Áp dụng cho | Độ linh hoạt |
|---|---|---|---|
| **Compute Savings Plans** | 66% | EC2 instance, AWS Fargate, AWS Lambda | Mọi instance family, size, Availability Zone, Region, hệ điều hành hoặc tenancy |
| **EC2 Instance Savings Plans** | 72% | EC2 instance | Một instance family trong một Region (ví dụ M7g ở Singapore); mọi size, hệ điều hành hoặc tenancy trong family đó |
| **SageMaker AI Savings Plans** | 64% | Instance của Amazon SageMaker AI | Mọi instance family, size, Region hoặc thành phần (notebook, training, inference) |
| **Database Savings Plans** | 35% | Amazon Aurora, Amazon RDS, Amazon DynamoDB, Amazon ElastiCache, Amazon DocumentDB, Amazon Neptune, Amazon OpenSearch Service và các cơ sở dữ liệu được quản lý khác | Mọi engine, instance family, size, kiểu triển khai hoặc Region được hỗ trợ |

{{% notice tip %}}
Compute Savings Plans linh hoạt nhất; EC2 Instance Savings Plans giảm giá nhiều nhất nếu bạn biết chắc sẽ dùng instance family nào. Database Savings Plans chỉ có thời hạn 1 năm với hình thức No Upfront.
{{% /notice %}}
