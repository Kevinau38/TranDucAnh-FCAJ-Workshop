---
title : "Các loại Reserved Instance"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Offering class

| | Standard RI | Convertible RI |
|---|---|---|
| Giảm giá so với On-Demand (tối đa) | 72% | 66% |
| Đổi Availability Zone, phạm vi (scope), network platform hoặc instance size (cùng family, Linux/UNIX) | ✓ (modify) | ✓ (modify) |
| Đổi instance family, hệ điều hành, tenancy hoặc hình thức thanh toán | — | ✓ (exchange lấy Convertible RI khác có giá trị bằng hoặc lớn hơn) |
| Bán lại trên Reserved Instance Marketplace | ✓ | — |

#### Phạm vi (scope)

- **Regional**: giảm giá áp dụng cho phần sử dụng khớp cấu hình ở mọi Availability Zone trong Region và, với Linux/UNIX tenancy mặc định, cho mọi size trong cùng instance family (size flexibility). Không giữ chỗ capacity.
- **Zonal**: giảm giá chỉ áp dụng trong một Availability Zone, và AWS **giữ chỗ capacity** cho instance ở đó.

#### Hình thức thanh toán và thời hạn

RI dùng các hình thức thanh toán giống Savings Plans (**All Upfront**, **Partial Upfront**, **No Upfront**) với thời hạn **1 năm** hoặc **3 năm**. Bạn cũng có thể mua Standard RI có thời hạn còn lại ngắn hơn từ khách hàng khác trên **Reserved Instance Marketplace**.

{{% notice warning %}}
Với **No Upfront** và **Partial Upfront**, bạn bị tính phí cho mọi giờ trong thời hạn, dù có instance khớp cấu hình đang chạy hay không.
{{% /notice %}}
