---
title : "Dùng IAM Access Analyzer cho S3"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---


#### Dùng IAM Access Analyzer cho S3

**IAM Access Analyzer** dùng automated reasoning để tìm các tài nguyên – S3 bucket, IAM role, KMS key và nhiều loại khác – có thể bị truy cập từ **bên ngoài zone of trust** (tài khoản hoặc tổ chức của bạn). Với S3, Access Analyzer phân tích bucket policy, ACL, access point và thiết lập Block Public Access, rồi tạo **finding** cho mỗi bucket được chia sẻ công khai hoặc với tài khoản khác.

#### Tạo analyzer

1. Mở [IAM console](https://console.aws.amazon.com/iam/home) → **Access analyzer** → **Analyzer settings** → **Create analyzer**.
2. Giữ **Resource analysis – External access** (không tính thêm phí), nhập tên `FCJ-S3-Analyzer`. Zone of trust là **current account**.

![Tạo analyzer](/images/9-analyzer/9-create.png)

3. Chọn **Create analyzer**.

![Nút Create analyzer](/images/9-analyzer/9-create-btn.png)

#### Xem các finding

4. Trang **External access** mở ra. Lần quét đầu tiên mất vài phút. Lọc finding theo **Resource Type = AWS::S3::Bucket**.

![Finding của S3](/images/9-analyzer/9-findings.png)

Vì bạn đã gỡ ACL công khai và bucket policy ở các phần trước, bucket lab **không có finding nào ở trạng thái Active**: bucket không được chia sẻ công khai hay với tài khoản AWS nào khác.

{{% notice info %}}
Cách đọc một finding:
+ **Active** – bucket có thể bị truy cập từ bên ngoài zone of trust. Nếu đây là truy cập có chủ đích, hãy **archive** finding; nếu không, hãy gỡ câu lệnh policy, quyền ACL hoặc access point gây ra nó.
+ **Resolved** – quyền truy cập đã bị gỡ và analyzer không còn thấy nữa.
+ **Archived** – bạn đã đánh dấu truy cập là có chủ đích.

S3 console cũng hiển thị kết quả này tại tab **Permissions** của bucket (**Access finding**) và tại **External access summary** trong danh sách bucket.
{{% /notice %}}

{{% notice tip %}}
Trong tài khoản dùng chung, bạn có thể thấy finding của các IAM role thuộc người khác. Không chỉnh sửa chúng; hãy lọc theo loại tài nguyên hoặc tên tài nguyên của bạn.
{{% /notice %}}
