---
title : "Tự động lấy thông tin đăng nhập và kết nối"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Tổng quan

Lần này code chỉ cần tên định danh (identifier) của DB instance. Với thông tin xác thực tạm thời của IAM role `FCJ-rdspg-dev-ec2-role`, code sẽ:

1. gọi **`DescribeDBInstances`** để lấy endpoint, port và ARN của secret (`MasterUserSecret.SecretArn`);
2. gọi **`GetSecretValue`** để lấy tên người dùng và mật khẩu;
3. mở một connection pool có kiểm tra TLS.

`pg` cho phép truyền một hàm vào `password` và gọi hàm đó mỗi khi mở kết nối mới. Hàm trả về mật khẩu từ bộ nhớ đệm 5 phút, nên sau khi xoay vòng mật khẩu, pool tự nhận mật khẩu mới mà không cần khởi động lại.

#### Tạo module và chương trình

1. Tạo file `db.js`. Chương 4 dùng lại file này:

    ```bash
    cat > db.js <<'EOF'
    // Finds the endpoint and the credentials of the DB instance at run time
    const fs = require('fs');
    const { Pool } = require('pg');
    const { RDSClient, DescribeDBInstancesCommand } = require('@aws-sdk/client-rds');
    const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

    const region = process.env.AWS_REGION;
    const dbInstanceId = process.env.DB_INSTANCE_ID || 'rdspg-dev-db';
    const rds = new RDSClient({ region });
    const sm = new SecretsManagerClient({ region });

    // 1. Endpoint, port, and ARN of the secret that RDS manages for the master user
    async function describeInstance() {
      const out = await rds.send(new DescribeDBInstancesCommand({ DBInstanceIdentifier: dbInstanceId }));
      const db = out.DBInstances[0];
      return { host: db.Endpoint.Address, port: db.Endpoint.Port, secretArn: db.MasterUserSecret.SecretArn };
    }

    // 2. User name and password from Secrets Manager, cached for 5 minutes
    let cache = { at: 0, value: null };
    async function getCredentials(secretArn) {
      if (cache.value && Date.now() - cache.at < 5 * 60 * 1000) return cache.value;
      const out = await sm.send(new GetSecretValueCommand({ SecretId: secretArn }));
      const { username, password } = JSON.parse(out.SecretString);
      cache = { at: Date.now(), value: { username, password } };
      return cache.value;
    }

    // 3. A connection pool. pg calls password() for each new connection, so a rotated
    //    password is picked up without restarting the application.
    async function createPool(database = process.env.PGDATABASE || 'pglab') {
      const { host, port, secretArn } = await describeInstance();
      const { username } = await getCredentials(secretArn);
      return new Pool({
        host, port, database, user: username,
        password: async () => (await getCredentials(secretArn)).password,
        ssl: { ca: fs.readFileSync(__dirname + '/global-bundle.pem', 'utf8') },
        max: 5,
      });
    }

    module.exports = { describeInstance, createPool };
    EOF
    ```

2. Tạo file `auto-connect.js`:

    ```bash
    cat > auto-connect.js <<'EOF'
    // 3.2 Auto-fetching: no endpoint and no password in the code or in the environment
    const { describeInstance, createPool } = require('./db');

    (async () => {
      const info = await describeInstance();
      console.log('Endpoint found :', info.host + ':' + info.port);
      console.log('Secret found   :', info.secretArn.split(':secret:')[1]);
      const pool = await createPool('postgres');
      const { rows } = await pool.query(
        `SELECT current_user, inet_server_addr() AS server_ip, s.version AS tls_version
           FROM pg_stat_ssl s WHERE s.pid = pg_backend_pid()`);
      console.table(rows);
      await pool.end();
    })().catch((err) => { console.error('Failed:', err.name, err.message); process.exit(1); });
    EOF
    ```

#### Chạy chương trình

3. Xoá mọi thông tin kết nối khỏi shell rồi chạy chương trình:

    ```bash
    unset PGHOST PGUSER PGPASSWORD PGDATABASE SECRET_ARN; env | grep -c '^PG'
    npm run auto
    ```

    - (1) Không còn biến `PG*` nào: kết quả đếm là `0`.
    - (2) `npm run auto` chạy `auto-connect.js`.
    - (3) Chương trình tự tìm được endpoint và secret `rds!db-<id>`.
    - (4) Đã kết nối bằng `postgres` qua **TLS 1.3**.

    ![Tự động lấy thông tin](/images/3/3-2/1.png)

{{% notice tip %}}
`AWS_REGION` (đã đặt trong `~/.bashrc` ở mục 2.1) cho SDK biết cần gọi Region nào. SDK lấy thông tin xác thực của role từ instance metadata service; không có access key ở đâu cả. Nếu gặp `AccessDeniedException`, hãy kiểm tra inline policy `rdspg-dev-app-access` của role.
{{% /notice %}}
