---
title : "Dùng AWS Config rule để phát hiện bucket công khai"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---


#### Dùng AWS Config rule để phát hiện bucket công khai

**AWS Config** ghi lại cấu hình của tài nguyên và đánh giá cấu hình đó theo các rule. Managed rule **s3-bucket-public-read-prohibited** kiểm tra thiết lập Block Public Access, bucket policy và bucket ACL, và đánh dấu mọi bucket cho phép đọc công khai.

#### Thiết lập AWS Config

{{% notice info %}}
Nếu AWS Config đã ghi nhận tại Region này, hãy chuyển thẳng tới phần **Thêm rule**. Không thay đổi recorder mà người khác đang dùng trong tài khoản dùng chung.
{{% /notice %}}

1. Mở [AWS Config console](https://console.aws.amazon.com/config/home) và chọn **Get started**.

![Trang chủ AWS Config](/images/8-config/8-config-home.png)

2. Tại **Recording strategy**, chọn **Specific resource types** để giữ chi phí thấp.

![Specific resource types](/images/8-config/8-settings.png)

3. Tại **Resource type**, chọn **AWS S3 Bucket** với tần suất **Continuous**. Giữ lựa chọn **Use an existing AWS Config service-linked role**.

![Ghi nhận S3 bucket](/images/8-config/8-resource-type.png)

4. Tại **Delivery channel**, chọn **Create a bucket** và nhập tên, ví dụ `fcj-config-lab-<suffix>`, rồi chọn **Next**. Bỏ qua trang rules bằng **Next** và chọn **Confirm**.

![Bucket cho delivery channel](/images/8-config/8-delivery.png)

#### Thêm rule

5. Trong AWS Config console, chọn **Rules** → **Add rule**. Giữ **Add AWS managed rule**, tìm `s3-bucket-public-read-prohibited`, chọn rule và chọn **Next**.

![Chọn managed rule](/images/8-config/8-rule-select.png)

6. Giữ tên và thiết lập mặc định, chọn **Next**, rồi chọn **Save**.

![Cấu hình rule](/images/8-config/8-rule-config.png)

#### Chủ động làm bucket công khai

7. Trong S3 console, mở bucket lab → **Permissions** → **Access control list (ACL)** → **Edit**.
8. Với **Everyone (public access)**, tích **Objects – List**, tích **I understand the effects of these changes on my objects and buckets** và chọn **Save changes**.

![Cấp quyền list công khai](/images/8-config/8-acl-edit.png)

#### Đánh giá rule

9. Quay lại AWS Config, mở rule **s3-bucket-public-read-prohibited**, chọn **Actions** → **Re-evaluate**.

![Đánh giá lại rule](/images/8-config/8-reevaluate.png)

10. Sau khoảng một phút, cuộn xuống **Resources in scope** và chọn **All**. Bucket lab ở trạng thái **Noncompliant** với ghi chú *The S3 bucket ACL allows public read access*, còn bucket private dùng cho Config ở trạng thái **Compliant**.

![Bucket không tuân thủ](/images/8-config/8-noncompliant.png)

11. Gỡ quyền công khai: chỉnh lại bucket ACL, bỏ chọn **Everyone (public access) – List** và chọn **Save changes**.

{{% notice tip %}}
Ở các tài khoản có guardrail (ví dụ AWS Organizations có cơ chế tự động khắc phục), ACL công khai có thể bị gỡ và Block Public Access được bật lại sau vài phút. Đó là hành vi mong muốn trong production; kết quả rule ghi nhận trước khi bị khắc phục là đủ cho bài lab này.
{{% /notice %}}
