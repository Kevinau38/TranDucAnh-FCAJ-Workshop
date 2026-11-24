---
title : "Dọn dẹp tài nguyên"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Xoá toàn bộ tài nguyên của bài lab để không phải trả thêm phí. Bạn có thể dùng console hoặc CloudShell.

#### Dùng console

1. **Instances**: với mỗi instance (`fcj-wordpress-2gb`, `fcj-prestashop`, và `fcj-wordpress` nếu vẫn còn), mở menu thao tác (⋮) và chọn **Delete**.

![Xoá instance](/images/8/1-delete-menu.png)

2. Xác nhận bằng **Yes, delete**. Alarm của instance bị xoá cùng instance.

![Xác nhận xoá](/images/8/2-confirm.png)

3. **Databases**: mở `fcj-lightsail-db`, chọn tab **Delete** và xoá database. Bạn không cần tạo final snapshot.

4. **Snapshots**: xoá `fcj-wordpress-snap-1`.

5. **Networking**: xoá các static IP `fcj-wordpress-ip` và `fcj-prestashop-ip`. Static IP không gắn với instance sẽ bị tính phí.

6. **Alarm notifications**: nếu đã thêm email contact, hãy xoá nó.

#### Dùng CloudShell

```bash
aws lightsail delete-instance --instance-name fcj-wordpress-2gb
aws lightsail delete-instance --instance-name fcj-prestashop
aws lightsail delete-instance --instance-name fcj-wordpress   # nếu chưa xoá ở chương 6
aws lightsail delete-relational-database --relational-database-name fcj-lightsail-db --skip-final-snapshot
aws lightsail delete-instance-snapshot --instance-snapshot-name fcj-wordpress-snap-1
aws lightsail release-static-ip --static-ip-name fcj-wordpress-ip
aws lightsail release-static-ip --static-ip-name fcj-prestashop-ip
rm -f lsssh.sh /tmp/ad.json /tmp/lskey /tmp/lskey-cert.pub
```

Kiểm tra không còn tài nguyên nào. Mọi lệnh phải trả về `[]`:

```bash
aws lightsail get-instances --query 'instances[].name'
aws lightsail get-relational-databases --query 'relationalDatabases[].name'
aws lightsail get-static-ips --query 'staticIps[].name'
aws lightsail get-instance-snapshots --query 'instanceSnapshots[].name'
aws lightsail get-alarms --query 'alarms[].name'
```
