---
title : "Dọn dẹp tài nguyên"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Xoá container service sẽ xoá luôn các deployment và stored image của nó.

1. Trên trang `fcj-container`, mở menu (⋮) bên cạnh các tab và chọn **Delete**.

![Menu Delete](/images/5/1-menu.png)

2. Chọn **Delete container service**.

![Delete container service](/images/5/2-delete.png)

3. Xác nhận bằng **Yes, delete**.

![Xác nhận](/images/5/3-confirm.png)

4. Trong CloudShell, xoá image cục bộ, thư mục làm việc và lightsailctl. Lệnh cuối phải trả về `[]`.

```bash
aws lightsail delete-container-service --region ap-southeast-1 --service-name fcj-container
docker rmi -f fcj-nginx:v1
rm -rf ~/fcj-nginx ~/.local/bin/lightsailctl
aws lightsail get-container-services --region ap-southeast-1 \
  --query "containerServices[?containerServiceName=='fcj-container'].state"
```
