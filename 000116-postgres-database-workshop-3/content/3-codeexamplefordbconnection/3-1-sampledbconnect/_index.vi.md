---
title : "Kết nối database đơn giản"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Tổng quan

Cách kết nối đơn giản nhất: chương trình đọc endpoint, tên người dùng và mật khẩu từ các biến môi trường chuẩn của PostgreSQL (`PGHOST`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`). Tuỳ chọn `ssl.ca` buộc `pg` kiểm tra chứng chỉ của server do RDS ký và khớp với tên host.

#### Tạo chương trình

1. Tạo file `simple-connect.js`:

    ```bash
    cat > simple-connect.js <<'EOF'
    // 3.1 Simple connection: settings come from environment variables
    const fs = require('fs');
    const { Client } = require('pg');

    const client = new Client({
      host: process.env.PGHOST,
      port: Number(process.env.PGPORT || 5432),
      user: process.env.PGUSER,
      password: process.env.PGPASSWORD,
      database: process.env.PGDATABASE || 'postgres',
      // Verify the server certificate and host name with the RDS CA bundle
      ssl: { ca: fs.readFileSync('global-bundle.pem', 'utf8') },
    });

    (async () => {
      await client.connect();
      const { rows } = await client.query(
        `SELECT current_user, current_database(), inet_server_addr() AS server_ip,
                s.version AS tls_version, split_part(version(), ' ', 2) AS pg_version
           FROM pg_stat_ssl s WHERE s.pid = pg_backend_pid()`);
      console.table(rows);
      await client.end();
    })().catch((err) => { console.error('Connection failed:', err.message); process.exit(1); });
    EOF
    ```

    Câu truy vấn đọc `pg_stat_ssl` của chính kết nối này, nên kết quả chứng minh phiên làm việc đã được mã hoá.

#### Chạy chương trình

2. Đưa thông tin kết nối vào biến môi trường rồi chạy chương trình. Mật khẩu lấy từ secret bạn đã đọc ở [2.2](../../2-preparation/2-2-ssm/):

    ```bash
    cd ~/rdspg-dev-app
    SECRET_ARN=$(aws rds describe-db-instances --db-instance-identifier rdspg-dev-db \
        --query 'DBInstances[0].MasterUserSecret.SecretArn' --output text)
    export PGHOST=$(aws rds describe-db-instances --db-instance-identifier rdspg-dev-db \
        --query 'DBInstances[0].Endpoint.Address' --output text) PGUSER=postgres PGDATABASE=pglab
    export PGPASSWORD=$(aws secretsmanager get-secret-value --secret-id "$SECRET_ARN" \
        --query SecretString --output text | jq -r .password)
    npm run simple
    ```

    - (1) `npm run simple` chạy `simple-connect.js`.
    - (2) Bạn đã kết nối bằng user `postgres` vào database `pglab`.
    - (3) Kết nối dùng **TLS 1.3**.
    - (4) Server chạy PostgreSQL **18.3**.

    ![Kết nối đơn giản](/images/3/3-1/1.png)

{{% notice warning %}}
Cách này chạy được, nhưng mật khẩu giờ nằm trong môi trường của shell, và trong bất kỳ file `.env` hay cấu hình triển khai nào bạn chép nó vào. Khi RDS xoay vòng mật khẩu (mỗi 7 ngày), chương trình sẽ lỗi cho tới khi có người chép mật khẩu mới. Mục 3.2 bỏ hẳn mật khẩu khỏi cấu hình.
{{% /notice %}}
