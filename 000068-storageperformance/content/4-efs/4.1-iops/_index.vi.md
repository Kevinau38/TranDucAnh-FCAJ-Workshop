---
title : "Tối ưu IOPS"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


Trong bước này, bạn tạo 1.024 file rỗng trên EFS theo ba cách khác nhau. Tạo file là thao tác metadata, nên bài đo này đo IOPS chứ không phải throughput.

1. Chạy ba lệnh sau. Mỗi lệnh tạo một thư mục ngẫu nhiên riêng trong `/efs/tutorial/touch`:

   ```bash
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}; time for i in {1..1024}; do touch /efs/tutorial/touch/${directory}/test-1.3-$i; done
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}; time seq 1 1024 | parallel --will-cite -j 128 touch /efs/tutorial/touch/${directory}/test-1.4-{}
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}/{1..32}; time seq 1 32 | parallel --will-cite -j 32 touch /efs/tutorial/touch/${directory}/{}/test1.5{1..32}
   ```

   - (1) Mỗi lần một lệnh `touch`, trong một thư mục: khoảng 9 giây.
   - (2) 128 lệnh `touch` cùng lúc, trong một thư mục: khoảng 5 giây.
   - (3) 32 job, mỗi job tạo 32 file trong thư mục riêng: dưới 1 giây.

   ![Tạo 1.024 file theo ba cách](/images/4-efs/4.1-touch.png)

Cách thứ ba nhanh hơn cách đầu tiên hơn 15 lần. Nó chạy nhiều thao tác song song, chia các thao tác ra nhiều thư mục để chúng không tranh nhau cùng một thư mục, và chỉ khởi động 32 tiến trình thay vì 1.024. Cách tốt nhất để dùng EFS là làm việc với nhiều luồng và nhiều thư mục cùng lúc.
