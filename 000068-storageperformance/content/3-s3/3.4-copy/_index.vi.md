---
title : "Tối ưu thao tác copy"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---


Để copy một object sang vị trí khác trong S3, bạn không cần tải nó về trước. Amazon S3 có thể copy dữ liệu ngay bên trong dịch vụ (**PUT Copy**, tức các API `CopyObject` và `UploadPartCopy`), nên instance chỉ gửi lời gọi API còn dữ liệu không rời khỏi S3.

1. Copy object 5 GB `upload1.test` sang prefix `copy/` theo ba cách:

   ```bash
   aws configure set default.s3.max_concurrent_requests 10
   time (aws s3 cp s3://$bucket/upload1.test 5GB.file --no-progress; aws s3 cp 5GB.file s3://$bucket/copy/5GB.file --no-progress)
   time aws s3api copy-object --copy-source $bucket/upload1.test --bucket $bucket --key copy/5GB-2.file --query CopyObjectResult.ETag --output text
   time aws s3 cp s3://$bucket/upload1.test s3://$bucket/copy/5GB-3.file --no-progress
   ```

   - (1) Tải về instance rồi upload lại: khoảng 1 phút 20 giây. Có 10 GB dữ liệu đi giữa instance và S3.
   - (2) `aws s3api copy-object`: khoảng 1 phút 17 giây. Việc copy diễn ra trong S3, nhưng chỉ bằng một request duy nhất.
   - (3) `aws s3 cp` từ S3 sang S3: khoảng 10 giây. AWS CLI chia việc copy thành nhiều phần và copy tới 10 phần cùng lúc bằng `UploadPartCopy`.

   ![Ba cách copy một object 5 GB](/images/3-s3/3.4-copy.png)
2. Mở [Amazon S3 console](https://console.aws.amazon.com/s3/) và chọn bucket của bài lab. (1) Các prefix `copy/`, `parallel/`, `run1/` tới `run3/`, `sync1/` và `sync2/` chứa các object bạn đã upload trong phần này, bên cạnh bốn object `upload*.test`.

   ![Các object trong bucket của bài lab](/images/3-s3/3.4-bucket.png)

Cách (2) và (3) đều copy bên trong S3, nên không có dữ liệu đi qua instance và không phát sinh phí truyền dữ liệu. Cách (3) nhanh nhất vì kết hợp copy phía server với nhiều phần song song. Một request `CopyObject` còn giới hạn object tối đa 5 GB, trong khi multipart copy dùng được cho object tới 5 TB.

{{% notice note %}}
Với multipart copy, AWS CLI copy cả tag của object, nên role cần quyền `s3:GetObjectTagging` và `s3:PutObjectTagging` trên bucket. Role của bài lab đã có sẵn hai quyền này.
{{% /notice %}}
