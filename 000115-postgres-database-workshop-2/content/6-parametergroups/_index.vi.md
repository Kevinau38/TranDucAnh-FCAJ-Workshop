---
title : "Nhóm tham số (parameter group)"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Tổng quan

Trên máy chủ PostgreSQL của riêng bạn, bạn đổi cấu hình trong `postgresql.conf`. Trên Amazon RDS, bạn không truy cập được tệp đó. Thay vào đó, cấu hình đến từ một **DB parameter group**: tập hợp có tên các tham số engine mà bạn gắn cho một hoặc nhiều DB instance.

| Parameter group | Sửa được không? | Dùng khi |
|---|---|---|
| Mặc định (`default.postgres18`) | Không | Tự gắn khi bạn không chọn group |
| Tùy chỉnh (`rdspg-lab-pg18`) | Có | Cấu hình riêng của bạn, dùng chung cho mọi DB instance gắn group đó |

Mỗi tham số có một **apply type**:

- **Dynamic**: RDS áp dụng giá trị mới trong vài giây, không cần khởi động lại.
- **Static**: giá trị mới chỉ có hiệu lực sau khi **reboot** DB instance.

Trong chương này, bạn tạo parameter group tùy chỉnh cho PostgreSQL 18, bật hai tham số ghi log, gắn group cho `rdspg-lab-db` và kiểm tra giá trị mới bằng `SHOW`.

| Tham số | Mặc định | Giá trị mới | Tác dụng |
|---|---|---|---|
| `log_connections` | rỗng (tắt) | `receipt,authentication,authorization` | Ghi log mọi lần kết nối, bước xác thực, cơ sở dữ liệu và người dùng của phiên |
| `log_min_duration_statement` | `-1` (tắt) | `4000` | Ghi log mọi câu lệnh chạy từ 4 giây trở lên, kèm thời gian chạy |

{{% notice tip %}}
Hãy dùng một parameter group tùy chỉnh cho mỗi phiên bản engine và mỗi môi trường (ví dụ `prod-pg18` và `dev-pg18`). Đừng sửa parameter group mà production đang dùng trước khi thử thay đổi trên một DB instance khác.
{{% /notice %}}
