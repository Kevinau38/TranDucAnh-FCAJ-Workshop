---
title : "Deploying AWS FCJ User Management System"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Overview

FCJ User Management is a small web application to list, search, add, edit, and delete users. It is built with:

- **Node.js** and **Express**, the web server;
- **Handlebars** (`express-handlebars`), the HTML templates;
- **Amazon RDS for PostgreSQL**, the data, through the module `db.js` from [3.2](../3-codeexamplefordbconnection/3-2-autofetchingcredential/): no endpoint and no password in the code.

Every query uses parameters (`$1`, `$2`, ...), so user input never becomes part of the SQL text.

#### 1. Create the table

1. Create `schema.sql`. The table `fcj_user` has an identity column, a unique email, and a `CHECK` on the status:

    ```bash
    cat > schema.sql <<'EOF'
    CREATE TABLE IF NOT EXISTS fcj_user (
        id         integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        first_name varchar(45)  NOT NULL,
        last_name  varchar(45)  NOT NULL,
        email      varchar(150) NOT NULL UNIQUE,
        phone      varchar(15)  NOT NULL,
        courses    varchar(150) NOT NULL,
        status     varchar(10)  NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive'))
    );

    INSERT INTO fcj_user (first_name, last_name, email, phone, courses, status) VALUES
    ('An',   'Nguyen', 'an.nguyen@example.com',   '0901000001', 'AWS Certified Solutions Architect - Associate', 'active'),
    ('Binh', 'Tran',   'binh.tran@example.com',   '0901000002', 'AWS Certified Developer - Associate',           'active'),
    ('Chi',  'Le',     'chi.le@example.com',      '0901000003', 'AWS Certified Data Engineer - Associate',       'inactive'),
    ('Dung', 'Pham',   'dung.pham@example.com',   '0901000004', 'AWS Certified Advanced Networking - Specialty', 'active'),
    ('Giang','Hoang',  'giang.hoang@example.com', '0901000005', 'AWS Certified Security - Specialty',            'active')
    ON CONFLICT (email) DO NOTHING;
    EOF
    ```

2. Load it with `psql`. `PGSSLMODE=verify-full` makes `psql` verify the server certificate like the Node.js code does:

    ```bash
    cd ~/rdspg-dev-app
    read -r PGHOST SECRET_ARN <<< $(aws rds describe-db-instances --db-instance-identifier rdspg-dev-db \
        --query 'DBInstances[0].[Endpoint.Address,MasterUserSecret.SecretArn]' --output text)
    export PGHOST PGUSER=postgres PGDATABASE=pglab PGSSLMODE=verify-full \
        PGSSLROOTCERT=$HOME/rdspg-dev-app/global-bundle.pem \
        PGPASSWORD=$(aws secretsmanager get-secret-value --secret-id "$SECRET_ARN" \
        --query SecretString --output text | jq -r .password)
    psql -f schema.sql && psql -c 'SELECT id, first_name, last_name, email, status FROM fcj_user;'
    unset PGPASSWORD
    ```

    - (1) `CREATE TABLE` and `INSERT 0 5`: the table and 5 sample users are created.
    - (2) `SELECT` returns the 5 users.

    ![Create the table](/images/4/1.png)

#### 2. Create the application

3. Create `app.js`:

    ```bash
    cat > app.js <<'EOF'
    // 4. FCJ User Management: Express + Handlebars + RDS for PostgreSQL
    const express = require('express');
    const { engine } = require('express-handlebars');
    const { createPool } = require('./db');

    const app = express();
    app.engine('hbs', engine({ extname: '.hbs', helpers: { eq: (a, b) => a === b } }));
    app.set('view engine', 'hbs');
    app.use(express.urlencoded({ extended: false }));

    let pool;
    const STATUSES = ['active', 'inactive'];

    app.get('/', async (req, res, next) => {
      try {
        const q = (req.query.q || '').trim();
        const { rows } = await pool.query(
          `SELECT * FROM fcj_user
            WHERE $1 = '' OR first_name ILIKE '%' || $1 || '%' OR last_name ILIKE '%' || $1 || '%'
                  OR email ILIKE '%' || $1 || '%'
            ORDER BY id`, [q]);
        res.render('home', { users: rows, q, msg: req.query.msg });
      } catch (err) { next(err); }
    });

    app.get('/add', (req, res) => res.render('form', { title: 'Add user', action: '/add', user: { status: 'active' }, STATUSES }));

    app.post('/add', async (req, res, next) => {
      try {
        const { first_name, last_name, email, phone, courses, status } = req.body;
        await pool.query(
          'INSERT INTO fcj_user (first_name, last_name, email, phone, courses, status) VALUES ($1,$2,$3,$4,$5,$6)',
          [first_name, last_name, email, phone, courses, status]);
        res.redirect('/?msg=User added');
      } catch (err) { next(err); }
    });

    app.get('/edit/:id', async (req, res, next) => {
      try {
        const { rows } = await pool.query('SELECT * FROM fcj_user WHERE id = $1', [req.params.id]);
        if (!rows.length) return res.redirect('/?msg=User not found');
        res.render('form', { title: 'Edit user', action: `/edit/${rows[0].id}`, user: rows[0], STATUSES });
      } catch (err) { next(err); }
    });

    app.post('/edit/:id', async (req, res, next) => {
      try {
        const { first_name, last_name, email, phone, courses, status } = req.body;
        await pool.query(
          'UPDATE fcj_user SET first_name=$1, last_name=$2, email=$3, phone=$4, courses=$5, status=$6 WHERE id=$7',
          [first_name, last_name, email, phone, courses, status, req.params.id]);
        res.redirect('/?msg=User updated');
      } catch (err) { next(err); }
    });

    app.post('/delete/:id', async (req, res, next) => {
      try {
        await pool.query('DELETE FROM fcj_user WHERE id = $1', [req.params.id]);
        res.redirect('/?msg=User deleted');
      } catch (err) { next(err); }
    });

    app.use((err, req, res, next) => {
      console.error(err);
      res.status(500).render('error', { message: err.detail || err.message });
    });

    (async () => {
      pool = await createPool();
      const { rows } = await pool.query('SELECT current_user, inet_server_addr() AS server_ip');
      console.log(`Connected to RDS as ${rows[0].current_user} (${rows[0].server_ip})`);
      const port = Number(process.env.PORT || 3000);
      app.listen(port, () => console.log(`FCJ User Management listening on port ${port}`));
    })().catch((err) => { console.error('Start failed:', err.name, err.message); process.exit(1); });
    EOF
    ```

4. Create the templates:

    ```bash
    mkdir -p views/layouts
    cat > views/layouts/main.hbs <<'EOF'
    <!doctype html>
    <html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
    <title>FCJ User Management</title>
    <style>
     body{margin:0;font-family:system-ui,Segoe UI,Arial,sans-serif;background:#f4f6f8;color:#232f3e}
     header{background:#232f3e;color:#fff;padding:14px 32px;display:flex;align-items:center;gap:12px}
     header b{color:#ff9900} main{max-width:1100px;margin:24px auto;padding:0 16px}
     .card{background:#fff;border-radius:8px;box-shadow:0 1px 3px rgba(0,0,0,.12);padding:20px}
     table{width:100%;border-collapse:collapse} th,td{text-align:left;padding:10px;border-bottom:1px solid #e5e7eb}
     th{background:#f8fafc;font-size:13px;text-transform:uppercase;color:#475569}
     .btn{display:inline-block;padding:7px 14px;border-radius:6px;border:1px solid #cbd5e1;background:#fff;color:#232f3e;text-decoration:none;cursor:pointer;font-size:14px}
     .btn.primary{background:#ff9900;border-color:#ff9900;color:#111;font-weight:600} .btn.danger{color:#b91c1c}
     .bar{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px;gap:12px}
     input,select{padding:8px;border:1px solid #cbd5e1;border-radius:6px;font-size:14px;width:100%;box-sizing:border-box}
     form.grid{display:grid;grid-template-columns:1fr 1fr;gap:14px} label{font-size:13px;color:#475569}
     .tag{padding:2px 8px;border-radius:10px;font-size:12px} .active{background:#dcfce7;color:#166534} .inactive{background:#fee2e2;color:#991b1b}
     .msg{background:#e0f2fe;color:#075985;padding:10px 14px;border-radius:6px;margin-bottom:14px} td form{display:inline}
    </style></head>
    <body><header><span style="font-size:20px">&#9729;</span><div><b>FCJ</b> User Management <small style="opacity:.7">· Node.js · Express · Amazon RDS for PostgreSQL</small></div></header>
    <main>{{{body}}}</main></body></html>
    EOF
    cat > views/home.hbs <<'EOF'
    {{#if msg}}<div class="msg">{{msg}}</div>{{/if}}
    <div class="card">
     <div class="bar">
      <form method="get" action="/" style="display:flex;gap:8px;flex:1;max-width:420px">
       <input name="q" value="{{q}}" placeholder="Search by name or email"><button class="btn">Search</button>
      </form>
      <a class="btn primary" href="/add">+ Add user</a>
     </div>
     <table>
      <tr><th>ID</th><th>First name</th><th>Last name</th><th>Email</th><th>Phone</th><th>Course</th><th>Status</th><th></th></tr>
      {{#each users}}
      <tr><td>{{id}}</td><td>{{first_name}}</td><td>{{last_name}}</td><td>{{email}}</td><td>{{phone}}</td><td>{{courses}}</td>
       <td><span class="tag {{status}}">{{status}}</span></td>
       <td style="white-space:nowrap"><a class="btn" href="/edit/{{id}}">Edit</a>
        <form method="post" action="/delete/{{id}}"><button class="btn danger">Delete</button></form></td></tr>
      {{else}}<tr><td colspan="8">No users found.</td></tr>{{/each}}
     </table>
    </div>
    EOF
    cat > views/form.hbs <<'EOF'
    <div class="card">
     <h2 style="margin-top:0">{{title}}</h2>
     <form class="grid" method="post" action="{{action}}">
      <div><label>First name</label><input name="first_name" value="{{user.first_name}}" required maxlength="45"></div>
      <div><label>Last name</label><input name="last_name" value="{{user.last_name}}" required maxlength="45"></div>
      <div><label>Email</label><input name="email" type="email" value="{{user.email}}" required maxlength="150"></div>
      <div><label>Phone</label><input name="phone" value="{{user.phone}}" required maxlength="15"></div>
      <div><label>Course</label><input name="courses" value="{{user.courses}}" required maxlength="150"></div>
      <div><label>Status</label><select name="status">{{#each STATUSES}}<option {{#if (eq this ../user.status)}}selected{{/if}}>{{this}}</option>{{/each}}</select></div>
      <div style="grid-column:1/3"><button class="btn primary">Save</button> <a class="btn" href="/">Cancel</a></div>
     </form>
    </div>
    EOF
    cat > views/error.hbs <<'EOF'
    <div class="card"><h2 style="margin-top:0;color:#b91c1c">Something went wrong</h2><p>{{message}}</p><a class="btn" href="/">Back</a></div>
    EOF
    ```

#### 3. Start the application

5. Start the web server:

    ```bash
    npm start
    ```

    - (1) `npm start` runs `app.js`.
    - (2) The application connected to RDS as `postgres` and listens on port **3000**.

    ![npm start](/images/4/2.png)

    Keep the terminal open: the application stops when you press `Ctrl+C` or close the session.

6. In the [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#Instances:), select `rdspg-dev-server` and copy its **Public IPv4 address**. Open `http://<public-ip>:3000` in your browser. The security group `dev-server-sg` allows port 3000 only from your IP address.

#### 4. Use the application

7. The home page:
    - (1) The 5 users come from the table `fcj_user` in RDS.
    - (2) Choose **+ Add user**.

    ![Home](/images/4/3.png)

8. Fill in the form (1), then choose **Save** (2):

    ![Add user](/images/4/4.png)

9. Back on the home page:
    - (1) The message **User added**.
    - (2) The new user has the ID `6`.
    - (3) Choose **Edit** on this user.

    ![User added](/images/4/5.png)

10. Change **Status** to `inactive` (1), then choose **Save** (2):

    ![Edit user](/images/4/6.png)

11. Search and delete:
    - (1) Type part of the name in the search box, then choose **Search**.
    - (2) The user now has the status `inactive`.
    - (3) Choose **Delete**.

    ![Search](/images/4/7.png)

12. (1) The message **User deleted**: the row is gone from the table.

    ![User deleted](/images/4/8.png)

{{% notice note %}}
Try to add a user with an email that already exists: the `UNIQUE` constraint rejects it, and the application shows the error from PostgreSQL. Press `Ctrl+C` in the terminal to stop the application when you finish.
{{% /notice %}}
