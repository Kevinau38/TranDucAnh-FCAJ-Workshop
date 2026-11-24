---
title : "Hiệu năng Amazon EFS"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


Trong phần này, bạn ghi dữ liệu lên EFS file system mount tại `/efs` theo nhiều cách khác nhau. Mỗi thao tác NFS đều đi qua mạng tới EFS, nên một thao tác đơn lẻ mất nhiều thời gian hơn so với ổ đĩa local. EFS bù lại bằng cách phục vụ nhiều thao tác song song: càng dùng nhiều luồng, nhiều file và nhiều thư mục cùng lúc, bạn càng đạt throughput và IOPS cao hơn, cho tới giới hạn của file system.

File system của bài lab dùng throughput mode **Bursting**. Một file system nhỏ như thế này có thể burst tới khoảng 100 MiB/s (khoảng 110 MB/s), đây chính là mức trần bạn sẽ thấy ở một số bài đo. Với throughput **Elastic**, các bài đo này có thể đạt mức cao hơn nhiều.
