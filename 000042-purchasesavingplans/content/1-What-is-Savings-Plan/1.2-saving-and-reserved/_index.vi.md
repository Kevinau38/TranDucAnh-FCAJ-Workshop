---
title : "Savings Plans và Reserved Instances"
weight : 2
chapter : false
pre : " <b> 1.2. </b> "
---

#### So sánh Savings Plans và Reserved Instances

Cả hai mô hình đều giảm giá cho phần sử dụng ổn định để đổi lấy cam kết 1 hoặc 3 năm. Savings Plans cam kết **số tiền mỗi giờ**; Reserved Instances cam kết **một cấu hình instance cụ thể**.

| | Compute Savings Plans | EC2 Instance Savings Plans | Convertible RI | Standard RI |
|---|---|---|---|---|
| Giảm giá so với On-Demand (tối đa) | 66% | 72% | 66% | 72% |
| Áp dụng cho mọi instance family | ✓ | — | Phải exchange | — |
| Áp dụng cho mọi instance size | ✓ | ✓ | ✓ (phạm vi Regional) | ✓ (phạm vi Regional) |
| Áp dụng cho mọi hệ điều hành hoặc tenancy | ✓ | ✓ | Phải exchange | — |
| Áp dụng ở mọi Region | ✓ | — | — | — |
| Áp dụng cho AWS Fargate và AWS Lambda | ✓ | — | — | — |
| Giữ chỗ capacity trong một Availability Zone | — | — | ✓ (zonal) | ✓ (zonal) |
| Bán lại được trên Reserved Instance Marketplace | — | — | — | ✓ |
| Thời hạn | 1 hoặc 3 năm | 1 hoặc 3 năm | 1 hoặc 3 năm | 1 hoặc 3 năm |

{{% notice info %}}
AWS khuyến nghị dùng Savings Plans cho phần lớn các cam kết mới về compute: bạn được mức giảm giá như Reserved Instances mà không bị gắn với một cấu hình instance. Reserved Instances vẫn hữu ích khi bạn cần **giữ chỗ capacity** trong một Availability Zone, hoặc cho các dịch vụ mà Savings Plans không áp dụng.
{{% /notice %}}
