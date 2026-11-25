---
title : "Giữ volume và backup"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Trước khi `prod-postgres` ghi bất cứ thứ gì lên volume dùng chung, instance này giành reservation **Write Exclusive**. Từ lúc đó volume chỉ nhận lệnh ghi từ `prod-postgres`, dù `test-postgres` cũng đang gắn vào cùng volume.
