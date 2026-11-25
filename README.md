# TranDucAnh-FCAJ-Workshop

Workshops of the **AWS Study Group – First Cloud AI Journey (FCAJ)** that I, **Trần Đức Anh**, updated to the current AWS console, service versions, and group standards.

[Tiếng Việt bên dưới](#tiếng-việt)

## Workshops

| ID | Workshop | Status | Website |
| --- | --- | --- | --- |
| 000008 | [Amazon CloudWatch Workshop](000008-cloudwatch/) | Published | https://000008.awsfcaj.com |
| 000021 | [WordPress on AWS](000021-wordpresshaonaws/) | Published | https://000021.awsfcaj.com |
| 000034 | [Cost Visualization with AWS Cost Explorer](000034-costvisualization/) | Published | https://000034.awsfcaj.com |
| 000042 | [Savings Plans, Reserved Instances and Reserved DB Instances](000042-purchasesavingplans/) | Published | https://000042.awsfcaj.com |
| 000045 | [Amazon Lightsail Workshop](000045-lightsail-workshop/) | Published | https://000045.awsfcaj.com |
| 000046 | [Amazon Lightsail Container](000046-containerlightsail/) | Published | https://000046.awsfcaj.com |
| 000068 | [Storage Performance Lab (Amazon S3 and Amazon EFS)](000068-storageperformance/) | Published | https://000068.awsfcaj.com |
| 000069 | [Amazon S3 Security Best Practices](000069-s3security/) | Published | https://000069.awsfcaj.com |
| 000101 | [AWS Service Screener](000101-aws-service-screeneer/) | Published | https://000101.awsfcaj.com |
| 000111 | [Secure Hybrid Access to Amazon S3 using VPC Endpoints](000111-s3-vpcendpoint/) | Published | https://000111.awsfcaj.com |
| 000114 | [AWS RDS PostgreSQL (Postgres Database Workshop 1)](000114-postgres-database-workshop-1/) | Published | https://000114.awsfcaj.com |
| 000115 | [AWS RDS PostgreSQL Foundation (Postgres Database Workshop 2)](000115-postgres-database-workshop-2/) | Published | https://000115.awsfcaj.com |
| 000116 | [AWS RDS PostgreSQL for Developers (Postgres Database Workshop 3)](000116-postgres-database-workshop-3/) | Published | https://000116.awsfcaj.com |
| 000117 | [Build a Complete Serverless Chat Website](000117-build-serverless-chat-website/) | Published | https://000117.awsfcaj.com |
| 000128 | [Deploy a Hugo Website with GitHub Pages](000128-deploy-website-with-git-pages/) | Published | https://000128.awsfcaj.com |
| 000142 | [Share Data Across VPCs with EBS Multi-Attach and NVMe Reservations](000142-nvme-reservation-sharedataaccrossvpc/) | Published | https://000142.awsfcaj.com |

## What "updated" means

For each workshop, I:

- Went through the lab again with the current AWS console, service versions, and tools, and retook the screenshots where the UI or the steps had changed.
- Rewrote the content in **English and Vietnamese**, and checked that both versions match.
- Fixed outdated or incorrect steps, explained the concepts behind each step, and added troubleshooting notes and a clean-up chapter where the lab creates resources.
- Redrew the architecture diagram with the current AWS Architecture Icons (every workshop that has an architecture; 000034 and 000042 are console-only and have none).
- Applied the FCAJ workshop rules: latest Hugo, `awsfcaj.com` domain, logo, Counter ID, footer, and page structure.

What changed in each workshop is listed below.

### 000008 – Amazon CloudWatch Workshop

- New CloudFormation template: three Amazon Linux 2023 arm64 `t4g.micro` instances with a least-privilege IAM role and the CloudWatch agent.
- Metrics: search and math expressions, dynamic labels; Logs Insights queries and a metric filter.
- Alarm with SNS email notification, and a CloudWatch dashboard.

### 000021 – WordPress on AWS

- Fully private design: EC2 Auto Scaling group and Amazon RDS for MySQL 8.4 Multi-AZ in private subnets, no NAT gateway.
- S3 gateway endpoint, Session Manager through interface endpoints, database password managed by AWS Secrets Manager.
- Only the Application Load Balancer is public, placed behind Amazon CloudFront and restricted with the CloudFront prefix list.
- WordPress installed with WP-CLI before the ALB is attached; snapshot and restore of the database.

### 000034 – Cost Visualization with AWS Cost Explorer

- Cost Explorer basics, filtering and grouping, saved reports.
- Savings Plans and Reserved Instance coverage reports, cost allocation tags, and data transfer costs.

### 000042 – Savings Plans, Reserved Instances and Reserved DB Instances

- Savings Plans types and recommendations, the purchase pages, EC2 Reserved Instances, and reserved DB instances.
- The lab never completes a purchase: no resources are created and there is no cost.

### 000045 – Amazon Lightsail Workshop

- WordPress on Ubuntu 24.04 with a Lightsail managed MySQL database, and a PrestaShop blueprint (Singapore Region).
- Firewall rules, snapshots, migration to a larger instance with the static IP moved over, and a CPU alarm.

### 000046 – Amazon Lightsail Container

- Lightsail container service (Nano, 1 node), deploying nginx from Docker Hub.
- Building an image in AWS CloudShell, pushing it with `push-container-image`, and deploying it as version 2.

### 000068 – Storage Performance Lab

- New CloudFormation template with an IAM role instead of an IAM user (Tokyo Region).
- Amazon S3: multi-threaded uploads, `sync`, small objects, server-side copy.
- Amazon EFS: small files, I/O size and sync, multi-threaded writes, and copy tools (`rsync`, `cp`, `fpsync`, `parallel`).

### 000069 – Amazon S3 Security Best Practices

- One CloudFormation template; access through an IAM role and Session Manager.
- Require HTTPS, SSE-S3, block public ACLs, S3 Block Public Access, restrict access to a VPC endpoint, an AWS Config rule, and IAM Access Analyzer.

### 000101 – AWS Service Screener

- Service Screener v2 run from AWS CloudShell (Singapore Region) as a read-only scan of a tagged demo bucket.
- Reading the HTML report, the AWS Well-Architected Tool integration, and troubleshooting.

### 000111 – Secure Hybrid Access to Amazon S3 using VPC Endpoints

- Gateway and interface endpoints for S3, with a simulated on-premises VPC connected through VPC peering.
- Route 53 Resolver inbound and outbound endpoints, endpoint policies, and SSM interface endpoints.

### 000114 – AWS RDS PostgreSQL

- Network: VPC with public and private subnets in two Availability Zones, S3 gateway endpoint instead of a NAT gateway, security groups that reference other security groups.
- Compute and database: Amazon Linux 2023 bastion host and app server, Amazon RDS for PostgreSQL 18.6 (`db.t4g.micro`, gp3).
- Connectivity: pgAdmin 4 through an SSH tunnel, and `psql` with TLS `verify-full` from a private app server using SSH ProxyJump.
- PostgreSQL basics: databases, schemas, tables, DML, roles and privileges, system catalog, session parameters, and queries.
- Redrawn architecture diagram, and a clean-up chapter with the correct deletion order.

### 000115 – AWS RDS PostgreSQL Foundation

- Three Availability Zones with private subnets, bastion host with SSH ProxyJump, S3 gateway endpoint.
- Major version upgrade from PostgreSQL 17 to 18; monitoring with CloudWatch, Performance Insights, and `pgbench`.
- Automated backups, snapshots, point-in-time recovery, and AWS Backup.
- Read replicas, Multi-AZ DB cluster, parameter groups, and a Multi-AZ failover test.

### 000116 – AWS RDS PostgreSQL for Developers

- Node.js app using `pg` over TLS `verify-full`.
- RDS-managed secret in AWS Secrets Manager, read through an IAM role; AWS SDK for JavaScript v3 to find the endpoint and the secret.
- Express + Handlebars CRUD app, accessed with Session Manager (no SSH).

### 000117 – Build a Complete Serverless Chat Website

- Static site in a private S3 bucket served through CloudFront with Origin Access Control (no public bucket).
- API Gateway REST API and Node.js Lambda functions (`nodejs26.x`) with `FCJ-*` roles and a permissions boundary; DynamoDB on-demand tables with a GSI.
- Split the proxy into one function per route with mapping templates and the generated JavaScript SDK.
- Cognito user pool (SPA client), Cognito authorizer on every method, user name taken from the ID token; CachingOptimized with invalidation.
- New architecture diagram and a CLI clean-up script.

### 000128 – Deploy a Hugo Website with GitHub Pages

- User site published from a branch, the Static HTML workflow, and a Hugo workflow for CI/CD.
- Git Credential Manager instead of a personal access token; Hugo quick start with the Ananke theme.

### 000142 – Share Data Across VPCs with EBS Multi-Attach and NVMe Reservations

- Two isolated VPCs (prod and test) with private subnets in the same Availability Zone, no NAT, no peering; Session Manager through interface endpoints, S3 gateway endpoint for `dnf`.
- Amazon Linux 2023 instances with PostgreSQL 18, one Multi-Attach enabled `io2` volume attached to both.
- Tested NVMe reservation flow with `nvme-cli`: register keys, Write Exclusive reservation, write blocked on the other instance (Reservation Conflict), read-only restore, hand over write access.
- New architecture diagram and a CLI clean-up chapter.

## Build locally

Each workshop is a [Hugo](https://gohugo.io/) site. Go into the workshop folder and run the Hugo server, for example:

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
| 000008 | [Amazon CloudWatch Workshop](000008-cloudwatch/) | Đã xuất bản | https://000008.awsfcaj.com |
| 000021 | [WordPress on AWS](000021-wordpresshaonaws/) | Đã xuất bản | https://000021.awsfcaj.com |
| 000034 | [Cost Visualization with AWS Cost Explorer](000034-costvisualization/) | Đã xuất bản | https://000034.awsfcaj.com |
| 000042 | [Savings Plans, Reserved Instances and Reserved DB Instances](000042-purchasesavingplans/) | Đã xuất bản | https://000042.awsfcaj.com |
| 000045 | [Amazon Lightsail Workshop](000045-lightsail-workshop/) | Đã xuất bản | https://000045.awsfcaj.com |
| 000046 | [Amazon Lightsail Container](000046-containerlightsail/) | Đã xuất bản | https://000046.awsfcaj.com |
| 000068 | [Storage Performance Lab (Amazon S3 và Amazon EFS)](000068-storageperformance/) | Đã xuất bản | https://000068.awsfcaj.com |
| 000069 | [Amazon S3 Security Best Practices](000069-s3security/) | Đã xuất bản | https://000069.awsfcaj.com |
| 000101 | [AWS Service Screener](000101-aws-service-screeneer/) | Đã xuất bản | https://000101.awsfcaj.com |
| 000111 | [Secure Hybrid Access to Amazon S3 using VPC Endpoints](000111-s3-vpcendpoint/) | Đã xuất bản | https://000111.awsfcaj.com |
| 000114 | [AWS RDS PostgreSQL (Postgres Database Workshop 1)](000114-postgres-database-workshop-1/) | Đã xuất bản | https://000114.awsfcaj.com |
| 000115 | [AWS RDS PostgreSQL Foundation (Postgres Database Workshop 2)](000115-postgres-database-workshop-2/) | Đã xuất bản | https://000115.awsfcaj.com |
| 000116 | [AWS RDS PostgreSQL for Developers (Postgres Database Workshop 3)](000116-postgres-database-workshop-3/) | Đã xuất bản | https://000116.awsfcaj.com |
| 000117 | [Build a Complete Serverless Chat Website](000117-build-serverless-chat-website/) | Đã xuất bản | https://000117.awsfcaj.com |
| 000128 | [Deploy a Hugo Website with GitHub Pages](000128-deploy-website-with-git-pages/) | Đã xuất bản | https://000128.awsfcaj.com |
| 000142 | [Share Data Across VPCs with EBS Multi-Attach and NVMe Reservations](000142-nvme-reservation-sharedataaccrossvpc/) | Đã xuất bản | https://000142.awsfcaj.com |

### "Cập nhật" nghĩa là gì

Với mỗi workshop, mình:

- Làm lại lab với console, phiên bản dịch vụ và công cụ hiện tại của AWS, chụp lại hình ở những chỗ giao diện hoặc các bước đã thay đổi.
- Viết lại nội dung bằng **tiếng Anh và tiếng Việt**, rà soát để hai bản khớp nhau.
- Sửa các bước lỗi thời hoặc sai, giải thích khái niệm ở từng bước, bổ sung phần xử lý lỗi và chương dọn dẹp tài nguyên ở những lab có tạo tài nguyên.
- Vẽ lại sơ đồ kiến trúc bằng bộ AWS Architecture Icons mới nhất (mọi workshop có kiến trúc; 000034 và 000042 chỉ thao tác trên console nên không có sơ đồ).
- Áp dụng quy chuẩn workshop của FCAJ: Hugo mới nhất, tên miền `awsfcaj.com`, logo, Counter ID, footer và cấu trúc trang.

### Nội dung cập nhật từng workshop

- **000008 – Amazon CloudWatch:** template CloudFormation mới (3 instance Amazon Linux 2023 arm64 `t4g.micro`, IAM role quyền tối thiểu, CloudWatch agent); metric search và math expression, dynamic label; Logs Insights, metric filter; alarm gửi email qua SNS; dashboard.
- **000021 – WordPress on AWS:** thiết kế private hoàn toàn: EC2 Auto Scaling group và RDS for MySQL 8.4 Multi-AZ trong private subnet, không NAT gateway; S3 gateway endpoint; Session Manager qua interface endpoint; mật khẩu database do Secrets Manager quản lý; chỉ ALB là public, đặt sau CloudFront và giới hạn bằng prefix list của CloudFront; cài WordPress bằng WP-CLI trước khi gắn ALB; snapshot và restore database.
- **000034 – Cost Explorer:** kiến thức cơ bản, filter và group, saved report, coverage report của Savings Plans và RI, cost allocation tag, chi phí data transfer.
- **000042 – Savings Plans và Reserved Instances:** các loại Savings Plans và recommendation, trang mua, EC2 Reserved Instances, reserved DB instance; lab không bao giờ bấm mua nên không tạo tài nguyên và không tốn phí.
- **000045 – Amazon Lightsail:** WordPress trên Ubuntu 24.04 với database MySQL managed của Lightsail, blueprint PrestaShop (Singapore); firewall, snapshot, chuyển sang instance lớn hơn và gắn lại static IP, alarm CPU.
- **000046 – Lightsail Container:** container service (Nano, 1 node), deploy nginx từ Docker Hub; build image trong CloudShell, đẩy lên bằng `push-container-image` và deploy thành version 2.
- **000068 – Storage Performance:** template CloudFormation mới, dùng IAM role thay cho IAM user (Tokyo); S3: upload đa luồng, `sync`, object nhỏ, server-side copy; EFS: file nhỏ, I/O size và sync, ghi đa luồng, các công cụ copy (`rsync`, `cp`, `fpsync`, `parallel`).
- **000069 – S3 Security:** một template CloudFormation, truy cập qua IAM role và Session Manager; bắt buộc HTTPS, SSE-S3, chặn public ACL, S3 Block Public Access, giới hạn truy cập qua VPC endpoint, AWS Config rule, IAM Access Analyzer.
- **000101 – AWS Service Screener:** chạy Service Screener v2 từ CloudShell (Singapore) để quét read-only một bucket demo có tag; đọc báo cáo HTML, tích hợp Well-Architected Tool, xử lý lỗi.
- **000111 – S3 qua VPC Endpoint:** gateway và interface endpoint cho S3, VPC giả lập on-premises nối bằng VPC peering; Route 53 Resolver inbound và outbound endpoint, endpoint policy, SSM interface endpoint.
- **000114 – AWS RDS PostgreSQL:** VPC public/private subnet ở 2 AZ, S3 gateway endpoint thay NAT gateway; bastion và app server Amazon Linux 2023, RDS for PostgreSQL 18.6 (`db.t4g.micro`, gp3); pgAdmin 4 qua SSH tunnel, `psql` TLS `verify-full` qua SSH ProxyJump; kiến thức PostgreSQL cơ bản; sơ đồ kiến trúc vẽ lại và chương dọn dẹp đúng thứ tự xoá.
- **000115 – RDS PostgreSQL Foundation:** 3 AZ với private subnet, bastion và SSH ProxyJump, S3 gateway endpoint; nâng cấp PostgreSQL 17 lên 18; giám sát bằng CloudWatch, Performance Insights, `pgbench`; backup tự động, snapshot, PITR, AWS Backup; read replica, Multi-AZ DB cluster, parameter group, thử failover Multi-AZ.
- **000116 – RDS PostgreSQL for Developers:** ứng dụng Node.js dùng `pg` qua TLS `verify-full`; secret do RDS quản lý trong Secrets Manager, đọc qua IAM role; AWS SDK for JavaScript v3 để lấy endpoint và secret; ứng dụng CRUD Express + Handlebars, truy cập bằng Session Manager (không SSH).
- **000117 – Serverless Chat Website:** website tĩnh trong bucket S3 private, phát hành qua CloudFront với Origin Access Control; API Gateway REST và Lambda Node.js (`nodejs26.x`) với role `FCJ-*` có permissions boundary; DynamoDB on-demand có GSI; tách proxy thành nhiều function với mapping template và JavaScript SDK sinh từ API; Cognito user pool (SPA), authorizer cho API, tên người dùng lấy từ ID token; bật CachingOptimized và invalidation; sơ đồ kiến trúc mới và script dọn dẹp bằng CLI.
- **000128 – Hugo với GitHub Pages:** user site publish từ branch, workflow Static HTML và workflow Hugo cho CI/CD; dùng Git Credential Manager thay cho personal access token; Hugo quick start với theme Ananke.
- **000142 – EBS Multi-Attach và NVMe Reservation:** hai VPC tách biệt (prod và test) với private subnet cùng Availability Zone, không NAT, không peering; Session Manager qua interface endpoint, S3 gateway endpoint cho `dnf`; instance Amazon Linux 2023 chạy PostgreSQL 18, một volume `io2` bật Multi-Attach gắn vào cả hai; chạy thử luồng NVMe reservation bằng `nvme-cli`: đăng ký key, reservation Write Exclusive, lệnh ghi từ instance còn lại bị chặn (Reservation Conflict), restore từ mount chỉ đọc, chuyển quyền ghi; sơ đồ kiến trúc mới và chương dọn dẹp bằng CLI.

### Chạy trên máy

Mỗi workshop là một site [Hugo](https://gohugo.io/). Vào thư mục workshop và chạy `hugo server` (xem ví dụ ở phần tiếng Anh).

Workshop gốc do cộng đồng **AWS Study Group / First Cloud Journey** xây dựng. Repo này chứa các phiên bản do mình cập nhật.
