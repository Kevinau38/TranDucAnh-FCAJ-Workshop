---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### Hiệu năng Amazon S3

Amazon S3 tự động mở rộng để đáp ứng số lượng request lớn. Một ứng dụng có thể đạt ít nhất **3.500 request PUT/COPY/POST/DELETE** và **5.500 request GET/HEAD** mỗi giây **cho mỗi prefix**, và số prefix trong một bucket là không giới hạn. Muốn nhanh hơn, bạn **song song hoá**: chia object lớn thành nhiều phần và upload các phần cùng lúc (multipart upload), phân tán object qua nhiều prefix, hoặc chạy nhiều tác vụ truyền cùng lúc. Các workload data lake lớn dùng cách này để tận dụng hết băng thông mạng của instance, lên tới 100 Gb/s trên một EC2 instance.

AWS CLI làm việc này tự động với các lệnh `aws s3` (`cp`, `sync`, `mv`). Ba cấu hình quyết định CLI chạy mạnh tới đâu:

| Cấu hình | Ý nghĩa |
|---|---|
| `max_concurrent_requests` | Số request CLI gửi cùng lúc (mặc định 10) |
| `multipart_threshold` | Object lớn hơn ngưỡng này được upload thành nhiều phần (mặc định 8 MB) |
| `multipart_chunksize` | Kích thước mỗi phần (mặc định 8 MB) |

#### Hiệu năng Amazon EFS

Amazon EFS là file system NFS dạng serverless, tự co giãn. Hiệu năng của EFS phụ thuộc vào cách bạn tạo nó:

| Tuỳ chọn | Lựa chọn |
|---|---|
| Storage class | **EFS Standard** (dữ liệu lưu ở nhiều Availability Zone) hoặc **EFS One Zone** (một Availability Zone); mỗi loại có thêm lớp Infrequent Access và Archive với độ trễ byte đầu tiên cao hơn |
| Performance mode | **General Purpose** (độ trễ thấp nhất, được khuyến nghị) hoặc **Max I/O** (chỉ dành cho workload cũ cần song song cực lớn) |
| Throughput mode | **Elastic** (tự co giãn theo workload, trả tiền theo GB truyền), **Provisioned** (throughput cố định) hoặc **Bursting** (throughput tăng theo lượng dữ liệu lưu trữ, có burst credit) |

Vì mỗi thao tác NFS đều đi qua mạng, EFS có độ trễ trên mỗi thao tác cao hơn ổ EBS gắn trực tiếp. Cách để đạt throughput và IOPS cao trên EFS cũng giống S3: **nhiều luồng, nhiều file, nhiều thư mục** cùng lúc.

File system của bài lab dùng **EFS Standard**, **General Purpose** và **Bursting**. File system mới tạo có đủ burst credit cho cả bài lab.
