---
title : "Tối ưu throughput"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


Trong bước này, bạn upload một file 5 GB với số request song song tăng dần, rồi upload cùng lượng dữ liệu đó dưới dạng năm file riêng biệt.

#### Upload một file lớn

1. Cấu hình AWS CLI chỉ gửi một request mỗi lúc, và upload file lớn hơn 64 MB thành các phần 16 MB. Sau đó tạo file thử nghiệm 5 GB:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && aws configure set default.s3.multipart_threshold 64MB && aws configure set default.s3.multipart_chunksize 16MB && cat ~/.aws/config
   dd if=/dev/urandom of=5GB.file bs=1 count=0 seek=5G && ls -lh 5GB.file
   ```

   - (1) Ba cấu hình được ghi vào file `~/.aws/config`.
   - (2) Lệnh `dd` với `count=0 seek=5G` tạo ngay file 5 GB (sparse file), nên bạn không phải chờ ghi dữ liệu ngẫu nhiên.

   ![Cấu hình S3 của AWS CLI](/images/3-s3/3.1-config.png)

2. Upload file bốn lần, với 1, 2, 10 và 20 request đồng thời:

   ```bash
   time aws s3 cp 5GB.file s3://${bucket}/upload1.test --no-progress
   aws configure set default.s3.max_concurrent_requests 2 && time aws s3 cp 5GB.file s3://${bucket}/upload2.test --no-progress
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 cp 5GB.file s3://${bucket}/upload3.test --no-progress
   aws configure set default.s3.max_concurrent_requests 20 && time aws s3 cp 5GB.file s3://${bucket}/upload4.test --no-progress
   ```

   Trong ảnh chụp, lần upload mất khoảng 1 phút với 1 request, 31 giây với 2 request, và khoảng 9 giây với cả 10 lẫn 20 request.

   ![Upload với 1, 2, 10 và 20 luồng](/images/3-s3/3.1-uploads.png)

File 5 GB được chia thành 320 phần 16 MB. Với một request, các phần được gửi lần lượt từng phần một. Với nhiều request, nhiều phần được gửi cùng lúc nên upload nhanh hơn nhiều. Từ 10 lên 20 request thì không nhanh thêm: lúc này chính AWS CLI (và CPU của instance 2 vCPU) đã trở thành giới hạn. Các công cụ viết riêng cho throughput cao, như AWS SDK với CRT transfer client, có thể dùng hàng trăm kết nối và đi xa hơn nữa.

#### Upload nhiều file cùng lúc

3. Tạo file 1 GB và upload nó năm lần song song bằng GNU `parallel`. Năm tiến trình AWS CLI chạy cùng lúc, mỗi tiến trình dùng 20 request đồng thời:

   ```bash
   dd if=/dev/urandom of=1GB.file bs=1 count=0 seek=1G && ls -lh 1GB.file
   time seq 1 5 | parallel --will-cite -j 5 aws s3 cp 1GB.file s3://${bucket}/parallel/object{}.test --no-progress
   ```

   - (1) `-j 5` chạy năm lệnh upload cùng lúc.
   - (2) Cùng 5 GB dữ liệu được upload xong trong khoảng 7 giây.

   ![Upload song song năm file 1 GB](/images/3-s3/3.1-parallel.png)

Song song hoá cũng tốn thêm request. Mỗi phần là một request PUT riêng, tính phí theo mỗi 1.000 request: một file 5 GB chia thành các phần 16 MB là 320 request PUT thay vì 1. Bạn trả thêm một ít tiền request để đổi lấy tốc độ.
