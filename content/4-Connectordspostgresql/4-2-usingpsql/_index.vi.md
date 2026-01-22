---
title : "Sử dụng EC2 instance để kết nối với RDS PostgreSQL"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 4.2 </b> "
---

#### Để kết nối đến RDS PostgreSQL DB bằng EC2 instance, hãy làm theo các bước sau:

1. Kết nối đến EC2 instance (App server) bằng MobaXterm.
2. Tải và cài đặt bản cập nhật hệ điều hành linux.
 Bạn có thể sử dụng lệnh sau để tải và cài đặt:
 ```
 cat /etc/os-release 
 cat /etc/system-release
 sudo yum update -y

 ``` 
 ![connecting](/images/4/4-2/1.png)
 
 3. Thêm kho lưu trữ bổ sung PostgreSQL Amazon

 PostgreSQL is part of the amazon extras library.\
 Để kiểm tra phiên bản postgresql có sẵn trong kho lưu trữ bổ sung của Amazon:
 ``` 
 amazon-linux-extras | grep postgresql
 ```
 ![connecting](/images/4/4-2/3.png)

 Để kích hoạt kho lưu trữ bổ sung của Amazon:
 ``` 
 sudo amazon-linux-extras enable postgresql14
 ```

 4. Cài đặt psql client:

 ``` sudo amazon-linux-extras install postgresql14 ```

 5. Cài đặt pgbench:

 ``` sudo yum install postgresql-contrib ```

 6. Kiểm tra **psql** & **pgbench**
```
psql --version
pgbench --version
```


  ![connecting](/images/4/4-2/5.png)

7. Kết nối đến RDS PostgreSQL:

```psql -h rdspg-fcj-labs.cssuddr073hp.us-east-1.rds.amazonaws.com -U masteruser -d pglab```

- -h: RDS PostgreSQL endpoint
- -U: username for the master user of your database instance
- -d: name for database 



