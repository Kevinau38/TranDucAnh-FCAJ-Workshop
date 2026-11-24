---
title : "Xử lý sự cố và best practice"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Các lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `screener: command not found` | Alias bị mất khi mở terminal hoặc phiên mới. Quay lại thư mục `service-screener-v2` và chạy `alias screener='python3 $(pwd)/main.py'`, hoặc làm lại chương 2 nếu `/tmp` đã bị xoá. |
| `CredentialRetrievalError` khi quét lâu | CloudShell cấp credential qua một proxy cục bộ, proxy này có thể lỗi khi tải cao. Trước khi quét, chạy `eval "$(aws configure export-credentials --format env)"`. Credential này hết hạn cùng phiên CloudShell (thường 1–3 giờ). Sau đó chạy lại lệnh quét và trả lời `y` để tiếp tục. |
| `AccessDenied` ở một số check | User thiếu quyền đọc với dịch vụ đó. Kiểm tra bằng `aws sts get-caller-identity` và thêm `ReadOnlyAccess`. |
| Quét quá lâu | Thu hẹp phạm vi bằng `--regions`, `--services` hoặc `--tags`, hoặc dùng `--disable-custom-pages 1` (tiết kiệm khoảng 2–3 phút). |
| `--beta 1` vẫn hiện báo cáo kiểu cũ | Chưa build Cloudscape UI. Làm lại bước 5 của chương 2. |

#### Best practice

- **Quét ở nơi bạn có workload.** Region không được quét là chưa được đánh giá, không có nghĩa là an toàn.
- **Chạy định kỳ**, ví dụ sau mỗi lần release hoặc mỗi tháng, rồi so sánh kết quả, hoặc dùng milestone của Well-Architected.
- **Sửa theo thứ tự**: mức High trước, rồi Quick Win, rồi phần còn lại.
- **Ẩn các finding đã chấp nhận** bằng suppression file, để báo cáo chỉ hiện vấn đề mới.
- **Giữ báo cáo ở chế độ riêng tư.** Lưu trên máy của bạn hoặc nơi riêng tư, không bao giờ đưa lên website công khai.
