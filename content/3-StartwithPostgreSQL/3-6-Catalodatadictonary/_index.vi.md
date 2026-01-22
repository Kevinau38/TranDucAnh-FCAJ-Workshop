---
title : "Catalog and Data dictionary"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 3.6. </b> "
---

Chương này giới thiệu PostgreSQL [Data Dictionary](https://www.postgresql.org/docs/11/ddl-schemas.html#DDL-SCHEMAS-CATALOG).

 #### The pg_catalog schema

 Ngoài các public schemas do người dùng tạo, mỗi cơ sở dữ liệu còn chứa một pg_catalog schema, chứa các bảng hệ thống và tất cả các kiểu dữ liệu, hàm và toán tử tích hợp sẵn.

 Đây là một tập hợp các bảng được sử dụng để lưu trữ siêu dữ liệu động và tĩnh cho cơ sở dữ liệu PostgreSQL và có thể được coi là “từ điển dữ liệu” cho cơ sở dữ liệu. Các bảng này được sử dụng cho các hoạt động thuộc loại “sổ sách kế toán” nội bộ. Tất cả các bảng danh mục Hệ thống đều bắt đầu bằng tiền tố pg_* và có thể tìm thấy trong [pg_catalog schema](https://www.postgresql.org/docs/11/catalogs-overview.html)

 1. Trong pgAdmin, hãy mở rộng nút **rdspg-fcj-labs**, sau đó mở rộng nút **Databases**, sau đó mở rộng nút **first_database**, sau đó mở rộng nút **Catalogs**, sau đó mở rộng **PostgreSQL Catalog**, sau đó mở rộng **Tables**.
  ![catalog](/images/2/2-6/43.png)

 Bạn có thể xem các bảng trong [pg_catalog schema](https://www.postgresql.org/docs/11/catalogs-overview.html).

 2. Nhấp vào **first_database**, sau đó chọn biểu tượng **Query Tool**

 3. Dán và chạy lệnh này để xem ví dụ về truy vấn trực tiếp **pg_catalog**: s
  ```select * from pg_tables where schemaname='pg_catalog';```
  ![catalog](/images/2/2-6/44.png)

#### The Statistics Collector Views (also part of pg_catalog)

 Trình thu thập số liệu thống kê là một hệ thống con đặc biệt thu thập thông tin động thời gian chạy về các hoạt động hiện tại trong phiên bản cơ sở dữ liệu. Ví dụ: [số lượt xem của người thu thập số liệu thống kê](https://www.postgresql.org/docs/11/monitoring-stats.html#MONITORING-STATS-DYNAMIC-VIEWS-TABLE) rất hữu ích để xác định tần suất truy cập một bảng cụ thể và liệu bảng có được quét hoặc truy cập bằng chỉ mục hay không.
 1. Dán và chạy lệnh này để xem ví dụ về một trong các chế độ xem **Statistics Collector**:

  ```SELECT * FROM pg_stat_activity WHERE STATE = 'active';```

  ![catalog](/images/2/2-6/45.png)

2.  Nhấp vào nút **rds-pg-labs**. Sau đó nhấp vào **Dashboard**. Sau đó nhấp vào **Sessions** trong phần Hoạt động của máy chủ.
  ![catalog](/images/2/2-6/46.png)

  Tại đây, pgAdmin đang hiển thị cho bạn thông tin **pg_stat_activity** tương tự cho cụm PostgreSQL của bạn.

 {{% notice tip%}}
 [Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html) là một cách tuyệt vời để trực quan hóa hoạt động hiện tại và lịch sử cũng như đợi các sự kiện. Chúng tôi khuyến khích bạn bắt đầu với Thông tin chi tiết về hiệu suất để chẩn đoán liên quan đến hiệu suất.
 {{% /notice %}}

 3. Dán và chạy lệnh này để xem ví dụ về một trong các chế độ xem **Statistics Collector**:
  ```select * from information_schema.tables;```
   ![catalog](/images/2/2-6/48.png) 