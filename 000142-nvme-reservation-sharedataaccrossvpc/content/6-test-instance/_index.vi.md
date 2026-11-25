---
title : "Restore trên instance test"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

Trong phần này bạn làm việc trên `test-postgres`. Bạn đăng ký key thứ hai, kiểm chứng volume chặn lệnh ghi từ instance này, và restore bản backup từ một mount chỉ đọc.

Mở phiên Session Manager tới `test-postgres`, chạy `bash -l` và `cd ~`, rồi cài các gói giống bước 4.1:

```bash
sudo dnf install -y -q nvme-cli postgresql18-server
```
