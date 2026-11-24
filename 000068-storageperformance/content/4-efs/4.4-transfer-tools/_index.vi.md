---
title : "So sánh các công cụ truyền file"
weight : 4
chapter : false
pre : " <b> 4.4 </b> "
---


Trong bước này, bạn copy 2.000 file mẫu (2 GB) từ ổ EBS sang EFS bằng bốn công cụ phổ biến và so sánh thời gian. Trước mỗi lần copy, page cache được xoá để mọi công cụ đều đọc file từ ổ EBS chứ không phải từ bộ nhớ.

1. Kiểm tra dữ liệu nguồn:

   ```bash
   du -csh /ebs/tutorial/data-1m/ && find /ebs/tutorial/data-1m/. -type f | wc -l
   ```

2. Copy các file bằng `rsync`, rồi bằng `cp`. Cả hai đều chạy một luồng:

   ```bash
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time rsync -r /ebs/tutorial/data-1m/ /efs/tutorial/rsync/
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time cp -r /ebs/tutorial/data-1m/* /efs/tutorial/cp/
   ```

3. Đặt số luồng bằng 4 luồng mỗi vCPU (8 luồng trên c7g.large), rồi copy các file bằng **fpsync**. fpsync dùng `fpart` để chia danh sách file thành nhiều phần (partition) và chạy một tiến trình `rsync` cho mỗi phần, nhiều tiến trình cùng lúc. Chạy hai lần, lần đầu với cấu hình mặc định, lần sau với `-f 250` (tối đa 250 file mỗi phần):

   ```bash
   threads=$(($(nproc --all) * 4)); echo $threads; sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time fpsync -n ${threads} /ebs/tutorial/data-1m/ /efs/tutorial/fpsync/
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time fpsync -n ${threads} -f 250 /ebs/tutorial/data-1m/ /efs/tutorial/fpsync2/
   ```

4. Copy các file bằng `cp` kết hợp GNU `parallel`, 8 lệnh copy cùng lúc:

   ```bash
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time find /ebs/tutorial/data-1m/. -type f | parallel --will-cite -j ${threads} cp {} /efs/tutorial/parallelcp
   ```

   - (1) `rsync`: khoảng 1 phút 31 giây.
   - (2) `cp`: khoảng 1 phút 9 giây.
   - (3) `fpsync` với cấu hình mặc định: khoảng 1 phút 38 giây, không nhanh hơn `rsync`.
   - (4) `fpsync -f 250`: khoảng 20 giây.
   - (5) `cp` với GNU `parallel`: khoảng 19 giây.

   ![rsync, cp, fpsync và parallel cp](/images/4-efs/4.4-transfer-tools.png)

Lần chạy `fpsync` đầu tiên là một bất ngờ đáng chú ý. Mặc định `fpart` cho tối đa 2.000 file vào một phần, nên cả 2.000 file mẫu nằm chung một phần và chỉ có một tiến trình `rsync` chạy: 8 luồng không hề được dùng. Với `-f 250`, danh sách được chia thành 8 phần, 8 tiến trình `rsync` chạy cùng lúc và việc copy nhanh hơn hơn bốn lần. Các công cụ đa luồng như fpsync, `mcp` hay GNU `parallel` tận dụng được thiết kế phân tán của EFS, nhưng chỉ khi công việc thực sự được chia nhỏ.

5. Bạn có thể mở thêm [Amazon EFS console](https://console.aws.amazon.com/efs/), chọn `FCJ-StoragePerf-EFS` và xem (1) performance mode và throughput mode. Tab **Monitoring** hiển thị throughput và IOPS của các bài đo bạn vừa chạy.

   ![Cấu hình của EFS file system](/images/4-efs/4-efs-console.png)
