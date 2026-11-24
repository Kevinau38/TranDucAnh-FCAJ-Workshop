---
title : "Trực quan hoá chi phí với AWS Cost Explorer"
weight : 1
chapter : false
---

# Trực quan hoá chi phí với AWS Cost Explorer

#### Tổng quan

Bạn không thể tối ưu thứ mà bạn không nhìn thấy. **AWS Cost Explorer** hiển thị chi phí và mức sử dụng của tài khoản AWS dưới dạng biểu đồ và bảng, giúp bạn tìm ra dịch vụ, Region hay loại sử dụng (usage type) nào làm tăng hoá đơn, và theo dõi chúng thay đổi theo thời gian. Đây là nội dung của trụ cột **Cost Optimization** trong AWS Well-Architected Framework: *quản lý tài chính trên đám mây* và *nắm rõ chi tiêu và mức sử dụng*.

Trong workshop này, bạn sẽ:

| Chương | Bạn học được |
|---|---|
| 1. Làm quen Cost Explorer | Đọc báo cáo chi phí: khoảng thời gian, độ chi tiết (granularity), kiểu biểu đồ và bảng chi tiết |
| 2. Lọc và nhóm dữ liệu | Thu hẹp báo cáo theo dịch vụ và chia nhỏ theo usage type hoặc Region |
| 3. Báo cáo đã lưu | Lưu báo cáo vào thư viện báo cáo rồi xoá |
| 4. Báo cáo độ bao phủ | Đọc báo cáo coverage của Savings Plans và Reserved Instances |
| 5. Cost allocation tag | Dùng tag để tách chi phí theo dự án hoặc môi trường |
| 6. Chi phí truyền dữ liệu | Hiểu chi phí truyền dữ liệu (data transfer) của các kiến trúc phổ biến |

#### Bạn cần chuẩn bị

- Một tài khoản AWS và user hoặc role có quyền mở **Billing and Cost Management** (ví dụ AWS managed policy `AWSBillingReadOnlyAccess`). Trong AWS Organizations, management account xem được chi phí của mọi member account.
- Workshop không tạo gì ngoài một báo cáo đã lưu mà bạn sẽ xoá ở cuối bài, nên **không phát sinh chi phí**: dùng Cost Explorer trên console là miễn phí.

{{% notice note %}}
Ảnh chụp lấy từ một tài khoản lab dùng chung nên số tiền và account ID đã được làm mờ. Số liệu của bạn sẽ khác.
{{% /notice %}}
