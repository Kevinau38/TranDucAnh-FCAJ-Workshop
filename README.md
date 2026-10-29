# TranDucAnh-FCAJ-Workshop

Workshops of the **AWS Study Group – First Cloud AI Journey (FCAJ)** that I, **Trần Đức Anh**, updated to the current AWS console, service versions, and group standards.

[Tiếng Việt bên dưới](#tiếng-việt)

## Workshops

| ID | Workshop | Status | Website |
| --- | --- | --- | --- |
| 000114 | [AWS RDS PostgreSQL (Postgres Database Workshop 1)](000114-postgres-database-workshop-1/) | In review | https://000114.awsfcaj.com |

## What "updated" means

For each workshop, I:

- Rebuilt the whole lab on a real AWS account with the current console and service versions, then retook every screenshot (1600x900, 100% scale, annotated).
- Rewrote the content in **English and Vietnamese**, and checked that both versions match line by line.
- Fixed outdated or incorrect steps, explained the concepts behind each step, and added troubleshooting notes found while doing the lab.
- Applied the FCAJ workshop rules: latest Hugo, `awsfcaj.com` domain, logo, Counter ID, footer, and page structure.

### 000114 – AWS RDS PostgreSQL

- Network: VPC with public and private subnets in two Availability Zones, S3 gateway endpoint instead of a NAT gateway, security groups that reference other security groups.
- Compute and database: Amazon Linux 2023 bastion host and app server, Amazon RDS for PostgreSQL 18.6 (`db.t4g.micro`, gp3).
- Connectivity: pgAdmin 4 through an SSH tunnel, and `psql` with TLS `verify-full` from a private app server using SSH ProxyJump.
- PostgreSQL basics: databases, schemas, tables, DML, roles and privileges, system catalog, session parameters, and queries.
- New architecture diagram with the current AWS Architecture Icons, and a clean-up chapter with the correct deletion order.

## Build locally

Each workshop is a [Hugo](https://gohugo.io/) site.

```bash
cd 000114-postgres-database-workshop-1
hugo server
```

## Credits

The original workshops were created by the **AWS Study Group / First Cloud Journey** community. This repository contains my updated versions. The official workshops are published on `awsfcaj.com`.

---

## Tiếng Việt

Các workshop của **AWS Study Group – First Cloud AI Journey (FCAJ)** do mình, **Trần Đức Anh**, cập nhật theo giao diện AWS console, phiên bản dịch vụ và quy chuẩn hiện tại của nhóm.

| ID | Workshop | Trạng thái | Website |
| --- | --- | --- | --- |
| 000114 | [AWS RDS PostgreSQL (Postgres Database Workshop 1)](000114-postgres-database-workshop-1/) | Đang review | https://000114.awsfcaj.com |

Với mỗi workshop, mình:

- Làm lại toàn bộ lab trên tài khoản AWS thật với console và phiên bản dịch vụ hiện tại, chụp lại toàn bộ hình (1600x900, scale 100%, có chú thích).
- Viết lại nội dung bằng **tiếng Anh và tiếng Việt**, rà soát để hai bản khớp nhau từng dòng.
- Sửa các bước lỗi thời hoặc sai, giải thích khái niệm ở từng bước và bổ sung phần xử lý lỗi gặp phải khi làm lab.
- Áp dụng quy chuẩn workshop của FCAJ: Hugo mới nhất, tên miền `awsfcaj.com`, logo, Counter ID, footer và cấu trúc trang.

Workshop gốc do cộng đồng **AWS Study Group / First Cloud Journey** xây dựng. Repo này chứa các phiên bản do mình cập nhật.
