---
title : "Triển khai hệ thống AWS FCJ User Management"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Tổng quan

FCJ User Management là một ứng dụng web nhỏ để xem, tìm kiếm, thêm, sửa và xoá người dùng. Ứng dụng được xây dựng với:

- **Node.js** và **Express**: web server;
- **Handlebars** (`express-handlebars`): giao diện HTML;
- **Amazon RDS for PostgreSQL**: dữ liệu, truy cập qua module `db.js` ở mục [3.2](../3-codeexamplefordbconnection/3-2-autofetchingcredential/), nên không có endpoint hay mật khẩu nào trong code.

Mọi câu truy vấn đều dùng tham số (`$1`, `$2`, ...), nên dữ liệu người dùng nhập không bao giờ trở thành một phần của câu SQL.

#### 1. Tạo bảng

1. Tạo file `schema.sql`. Bảng `fcj_user` có cột identity, email không trùng và ràng buộc `CHECK` cho trạng thái:

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

2. Nạp file bằng `psql`. `PGSSLMODE=verify-full` buộc `psql` kiểm tra chứng chỉ server giống như code Node.js:

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

    - (1) `CREATE TABLE` và `INSERT 0 5`: đã tạo bảng và 5 người dùng mẫu.
    - (2) `SELECT` trả về 5 người dùng.

    ![Tạo bảng](/images/4/1.png)

#### 2. Tạo ứng dụng

3. Tạo file `app.js`:

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

4. Tạo các template giao diện:

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

#### 3. Khởi động ứng dụng

5. Khởi động web server:

    ```bash
    npm start
    ```

    - (1) `npm start` chạy `app.js`.
    - (2) Ứng dụng đã kết nối RDS bằng `postgres` và lắng nghe ở cổng **3000**.

    ![npm start](/images/4/2.png)

    Giữ terminal mở: ứng dụng dừng khi bạn nhấn `Ctrl+C` hoặc đóng phiên.

6. Trong [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#Instances:), chọn `rdspg-dev-server` và sao chép **Public IPv4 address**. Mở `http://<public-ip>:3000` trên trình duyệt. Security group `dev-server-sg` chỉ cho phép cổng 3000 từ địa chỉ IP của bạn.

#### 4. Sử dụng ứng dụng

7. Trang chủ:
    - (1) 5 người dùng lấy từ bảng `fcj_user` trong RDS.
    - (2) Chọn **+ Add user**.

    ![Trang chủ](/images/4/3.png)

8. Điền form (1), rồi chọn **Save** (2):

    ![Thêm người dùng](/images/4/4.png)

9. Quay về trang chủ:
    - (1) Thông báo **User added**.
    - (2) Người dùng mới có ID `6`.
    - (3) Chọn **Edit** ở người dùng này.

    ![Đã thêm](/images/4/5.png)

10. Đổi **Status** thành `inactive` (1), rồi chọn **Save** (2):

    ![Sửa người dùng](/images/4/6.png)

11. Tìm kiếm và xoá:
    - (1) Nhập một phần tên vào ô tìm kiếm, rồi chọn **Search**.
    - (2) Người dùng giờ có trạng thái `inactive`.
    - (3) Chọn **Delete**.

    ![Tìm kiếm](/images/4/7.png)

12. (1) Thông báo **User deleted**: dòng dữ liệu đã bị xoá khỏi bảng.

    ![Đã xoá](/images/4/8.png)

{{% notice note %}}
Thử thêm một người dùng có email đã tồn tại: ràng buộc `UNIQUE` sẽ từ chối, và ứng dụng hiển thị lỗi từ PostgreSQL. Nhấn `Ctrl+C` trong terminal để dừng ứng dụng khi làm xong.
{{% /notice %}}
