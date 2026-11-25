---
title : "Tạo database production"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

1. Khởi tạo PostgreSQL, khởi động dịch vụ, rồi tạo database `test_erp` với bảng `clients` gồm ba dòng:

   ```bash
   sudo postgresql-setup --initdb && sudo systemctl enable --now postgresql && systemctl is-active postgresql
   sudo -u postgres psql -c "CREATE DATABASE test_erp;"
   sudo -u postgres psql -d test_erp -c "CREATE TABLE clients (id SERIAL PRIMARY KEY, first_name VARCHAR(50), last_name VARCHAR(50), role VARCHAR(50));" -c "INSERT INTO clients (first_name, last_name, role) VALUES ('John', 'Smith', 'CEO'), ('Jane', 'Doe', 'CTO'), ('Minh', 'Nguyen', 'DBA');" -c "SELECT * FROM clients;"
   ```

   - (1) Dịch vụ PostgreSQL ở trạng thái `active`.
   - (2) Database `test_erp` đã được tạo.
   - (3) Bảng `clients` có ba dòng.

   ![Database PostgreSQL](/images/4-prod-instance/4.2-postgresql.png)

Mặc định PostgreSQL chỉ lắng nghe trên `localhost`. Bài lab không kết nối tới PostgreSQL qua mạng nên bạn không cần sửa `postgresql.conf` hay `pg_hba.conf`.
