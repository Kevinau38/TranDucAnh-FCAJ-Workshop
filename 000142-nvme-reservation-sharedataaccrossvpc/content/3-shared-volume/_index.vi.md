---
title : "Tạo volume dùng chung"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

Trong phần này bạn tạo một volume `io2` bật Multi-Attach và gắn nó vào `prod-postgres` và `test-postgres`. Hai instance nằm ở hai VPC khác nhau nhưng cùng Availability Zone với volume, nên đều gắn được.
