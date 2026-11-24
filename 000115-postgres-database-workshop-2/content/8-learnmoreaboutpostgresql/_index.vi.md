---
title : "Tìm hiểu thêm về PostgreSQL"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Benchmark PostgreSQL bằng pgbench

`pgbench` đi kèm PostgreSQL và là cách nhanh nhất để tạo tải lặp lại được lên database. Công cụ mở nhiều phiên client, chạy cùng một kịch bản giao dịch trong mỗi phiên nhanh nhất có thể, rồi báo cáo thông lượng (**tps**, số giao dịch mỗi giây) và độ trễ. `pgbench` không đo CPU, bộ nhớ hay disk I/O của server: hãy theo dõi các chỉ số đó trong CloudWatch và Performance Insights khi chạy test, như bạn đã làm ở chương 3.

#### Các kịch bản có sẵn

| Kịch bản | Tuỳ chọn | Mỗi giao dịch làm gì |
|---|---|---|
| `tpcb-like` (mặc định) | `-b tpcb-like` | 3 `UPDATE`, 1 `SELECT` và 1 `INSERT`, dựa trên TPC-B |
| `simple-update` | `-N` hoặc `-b simple-update` | Như `tpcb-like` nhưng không cập nhật `pgbench_tellers` và `pgbench_branches`, nên ít tranh chấp khoá hơn |
| `select-only` | `-S` hoặc `-b select-only` | Một `SELECT` theo khoá chính. Hữu ích để test đọc, ví dụ trên read replica |
| Kịch bản của bạn | `-f script.sql` | Bất kỳ câu SQL nào bạn viết, với biến như `\set aid random(1, 100000 * :scale)` |

#### Khởi tạo dữ liệu test

Lệnh bạn đã chạy ở chương 3:

```bash
pgbench -i --fillfactor=90 --scale=50 pglab
```

| Tuỳ chọn | Ý nghĩa |
|---|---|
| `-i` (`--initialize`) | Tạo các bảng `pgbench_accounts`, `pgbench_branches`, `pgbench_tellers` và `pgbench_history`. **Lệnh xoá các bảng trùng tên trước.** |
| `-F` (`--fillfactor=90`) | Tạo bảng với fill factor 90: mỗi trang chỉ được lấp đầy 90%, phần trống giúp `UPDATE` giữ phiên bản dòng mới trong cùng trang (HOT update) |
| `-s` (`--scale=50`) | Nhân kích thước dữ liệu: 50 branch, 500 teller và 5.000.000 account (khoảng 750 MB) |
| `pglab` | Database sử dụng. Database phải tồn tại sẵn |

#### Chạy test

```bash
pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
```

| Tuỳ chọn | Ý nghĩa |
|---|---|
| `-M` (`--protocol=prepared`) | Gửi câu lệnh dưới dạng prepared statement, server chỉ phân tích và lập kế hoạch một lần cho mỗi phiên |
| `-P 30` (`--progress=30`) | In một dòng tiến độ (tps, độ trễ, độ lệch chuẩn) mỗi 30 giây |
| `-T` (`--time=300`) | Chạy test trong 300 giây. Dùng `-t` nếu muốn chạy số giao dịch cố định cho mỗi client |
| `-c` (`--client=50`) | Mở 50 phiên client tới database |
| `-j` (`--jobs=2`) | Dùng 2 luồng trong `pgbench`. Đặt bằng số vCPU của máy client, không phải bằng số client |
| `pglab` | Database cần kết nối |

Host, user và mật khẩu lấy từ `PGHOST`, `PGUSER` và `~/.pgpass`. Nếu không có, hãy thêm `-h <endpoint> -U postgres`. Lưu ý `-p` là **port**, không phải protocol.

#### Mẹo

- **Mỗi lần chỉ đổi một yếu tố.** Chạy mỗi test ít nhất hai lần, và chỉ so sánh các test khác nhau đúng một cấu hình: instance class, một tham số, hoặc số client.
- **Tăng số client từng bước** (ví dụ 10, 25, 50, 100). Khi tps không tăng nữa và độ trễ tăng lên, bạn đã chạm giới hạn của DB instance. `db.t4g.micro` chỉ cho khoảng 80 kết nối.
- **Để ý burst credit.** Class `t` chạy bằng CPU credit. Test dài có thể dùng hết credit, khi đó kết quả sẽ giảm.
- **Không test trên production.** `pgbench -i` xoá bảng, và tải test có thể làm chậm người dùng thật.

#### Tìm hiểu thêm

- [Amazon RDS for PostgreSQL User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html)
- [Tài liệu pgbench](https://www.postgresql.org/docs/current/pgbench.html)
- [Best practices for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [Amazon RDS Proxy](https://aws.amazon.com/rds/proxy/) để gom kết nối và failover nhanh hơn
