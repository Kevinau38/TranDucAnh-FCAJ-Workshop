---
title : "Auto-fetching instance credentials and connecting"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Overview

This time the code needs only the identifier of the DB instance. With the temporary credentials of the IAM role `FCJ-rdspg-dev-ec2-role`, it:

1. calls **`DescribeDBInstances`** to get the endpoint, the port, and the ARN of the secret (`MasterUserSecret.SecretArn`);
2. calls **`GetSecretValue`** to get the user name and the password;
3. opens a connection pool with TLS verification.

`pg` accepts a function as `password`, and calls it every time it opens a new connection. The function returns the password from a 5-minute cache, so after a rotation the pool picks up the new password without a restart.

#### Create the module and the program

1. Create `db.js`. Chapter 4 reuses it:

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

2. Create `auto-connect.js`:

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

#### Run it

3. Remove every connection setting from the shell, then run the program:

    ```bash
    unset PGHOST PGUSER PGPASSWORD PGDATABASE SECRET_ARN; env | grep -c '^PG'
    npm run auto
    ```

    - (1) No `PG*` variable is left: the count is `0`.
    - (2) `npm run auto` starts `auto-connect.js`.
    - (3) The program found the endpoint and the secret `rds!db-<id>` by itself.
    - (4) It is connected as `postgres` over **TLS 1.3**.

    ![Auto-fetching](/images/3/3-2/1.png)

{{% notice tip %}}
`AWS_REGION` (set in `~/.bashrc` in 2.1) tells the SDK which Region to call. The SDK takes the credentials of the role from the instance metadata service; there are no access keys anywhere. If you see `AccessDeniedException`, check the inline policy `rdspg-dev-app-access` of the role.
{{% /notice %}}
