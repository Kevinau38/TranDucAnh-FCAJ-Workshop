---
title : "Tối ưu thao tác với file nhỏ"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


Với object nhỏ, thời gian chủ yếu tốn cho request chứ không phải dữ liệu. Cách để chuyển nhiều object nhỏ nhanh hơn là có nhiều request cùng chạy một lúc, để đạt nhiều **giao dịch mỗi giây (TPS)** hơn.

1. Tạo 500 file 1 KB trong thư mục `small/`:

   ```bash
   seq 1 500 > object_ids && dd if=/dev/urandom of=1KB.file bs=1 count=0 seek=1K 2>/dev/null
   mkdir -p small && for i in $(seq 1 500); do cp 1KB.file small/object-$i; done; ls small | wc -l
   ```

2. Upload thư mục ba lần với 1, 10 và 50 request đồng thời:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && time aws s3 cp small/ s3://${bucket}/run1/ --recursive --only-show-errors
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 cp small/ s3://${bucket}/run2/ --recursive --only-show-errors
   aws configure set default.s3.max_concurrent_requests 50 && time aws s3 cp small/ s3://${bucket}/run3/ --recursive --only-show-errors
   ```

   - (1) Với 1 request: khoảng 18 giây, tức khoảng 28 object mỗi giây.
   - (2) Với 10 request: khoảng 2,3 giây, tức khoảng 220 object mỗi giây.
   - (3) Với 50 request: không nhanh hơn 10 request.

   ![Upload 500 file nhỏ với 1, 10 và 50 luồng](/images/3-s3/3.3-small-files.png)

Từ 10 lên 50 request, thời gian không đổi: giới hạn lúc này là một tiến trình CLI duy nhất và CPU của instance, không phải Amazon S3. Muốn nhanh hơn nữa, bạn cần chia việc ra nhiều tiến trình hoặc nhiều instance.

{{% notice note %}}
Phiên bản gốc của bước này chạy một lệnh `aws s3 cp` cho mỗi object bằng GNU `parallel` (`parallel -a object_ids -j 5 aws s3 cp ...`). Trên instance 2 vCPU, cách đó mất khoảng 2 phút cho 500 object dù tăng số job bao nhiêu, vì khởi động 500 tiến trình AWS CLI đã dùng hết CPU. Một tiến trình với nhiều request đồng thời hiệu quả hơn nhiều.
{{% /notice %}}
