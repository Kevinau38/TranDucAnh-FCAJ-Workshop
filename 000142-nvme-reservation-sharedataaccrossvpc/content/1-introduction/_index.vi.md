---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Multi-Attach

Một volume bật Multi-Attach có thể gắn cùng lúc vào tối đa **16 instance**. Các quy tắc chính:

| Quy tắc | Chi tiết |
|---|---|
| Loại volume | Chỉ Provisioned IOPS SSD: `io2` (khuyến nghị) hoặc `io1` |
| Instance | Phải chạy trên **Nitro System** và nằm **cùng Availability Zone** với volume. Có thể nằm ở VPC hoặc subnet khác nhau |
| Volume khởi động | Volume Multi-Attach không dùng làm boot volume được, và không bật Multi-Attach lúc launch instance được |
| File system | File system thông thường như **XFS** và **ext4** không được thiết kế để nhiều máy chủ mount cùng lúc. Hãy dùng cluster file system, hoặc bảo đảm chỉ một instance mount file system ở chế độ đọc ghi |
| Chi phí | Không tính thêm phí. Bạn trả giá bình thường của volume `io2` và số IOPS đã cấp |

Trong bài lab, volume được format XFS. Instance production mount volume, ghi file backup rồi unmount. Sau đó instance test mount ở chế độ chỉ đọc. Hai instance không bao giờ ghi vào file system cùng một lúc.

#### NVMe reservation

NVMe reservation là một phần của chuẩn NVMe. Trước hết mỗi instance **đăng ký** (register) một key trên volume. Instance đã đăng ký có thể **giành** (acquire) một reservation thuộc một loại nhất định. Volume áp dụng reservation đó cho mọi lệnh I/O nó nhận được.

| Lệnh | Lệnh `nvme-cli` | Mục đích |
|---|---|---|
| Reservation Register | `nvme resv-register` | Đăng ký, huỷ đăng ký hoặc thay key của instance |
| Reservation Acquire | `nvme resv-acquire` | Giành reservation, hoặc giành lại reservation của key khác |
| Reservation Release | `nvme resv-release` | Nhả hoặc xoá reservation |
| Reservation Report | `nvme resv-report` | Xem các key đã đăng ký và ai đang giữ reservation |

Loại reservation (`--rtype`) quyết định các instance khác được làm gì:

| rtype | Loại | Instance giữ | Instance khác đã đăng ký | Instance chưa đăng ký |
|---|---|---|---|---|
| 1 | Write Exclusive | Đọc và ghi | Chỉ đọc | Chỉ đọc |
| 2 | Exclusive Access | Đọc và ghi | Không truy cập | Không truy cập |
| 3 | Write Exclusive - Registrants Only | Đọc và ghi | Đọc và ghi | Chỉ đọc |
| 4 | Exclusive Access - Registrants Only | Đọc và ghi | Đọc và ghi | Không truy cập |
| 5 | Write Exclusive - All Registrants | Đọc và ghi | Đọc và ghi | Chỉ đọc |
| 6 | Exclusive Access - All Registrants | Đọc và ghi | Đọc và ghi | Không truy cập |

Bài lab dùng loại **1, Write Exclusive**: instance production là bên duy nhất được ghi, còn instance test vẫn đọc được file backup.

#### Yêu cầu

- NVMe reservation hoạt động trên volume `io2` bật Multi-Attach tạo sau ngày 18/09/2023, và volume `io1` tạo sau ngày 01/10/2026. Với volume cũ hơn, hãy gỡ volume khỏi tất cả instance rồi gắn lại.
- Hệ điều hành hỗ trợ: Amazon Linux 2 trở lên, RHEL 8.3 trở lên, SUSE Linux Enterprise 12 SP3 trở lên, Windows Server 2016 trở lên. Bài lab dùng **Amazon Linux 2023**.
- Dùng NVMe reservation không tốn thêm phí.
