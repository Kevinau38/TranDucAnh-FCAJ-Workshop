---
title : "Simple database connection"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Overview

The simplest way to connect: the program reads the endpoint, the user name, and the password from the standard PostgreSQL environment variables (`PGHOST`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`). The `ssl.ca` option makes `pg` verify that the certificate of the server is signed by RDS and matches the host name.

#### Create the program

1. Create `simple-connect.js`:

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

    The query reads `pg_stat_ssl` for its own connection, so the output proves that the session is encrypted.

#### Run it

2. Put the connection settings in environment variables, then run the program. The password comes from the secret that you read in [2.2](../../2-preparation/2-2-ssm/):

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

    - (1) `npm run simple` starts `simple-connect.js`.
    - (2) You are connected as `postgres` to the database `pglab`.
    - (3) The connection uses **TLS 1.3**.
    - (4) The server runs PostgreSQL **18.3**.

    ![Simple connection](/images/3/3-1/1.png)

{{% notice warning %}}
This works, but the password now lives in the environment of your shell, and in any `.env` file or deployment setting where you copy it. When RDS rotates the password (every 7 days), the program fails until someone copies the new one. Section 3.2 removes the password from the configuration.
{{% /notice %}}
