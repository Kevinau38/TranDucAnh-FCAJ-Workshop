---
title : "Cài đặt Git và Hugo"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


Bạn cần một tài khoản GitHub, cùng với **Git** và **Hugo extended** trên máy tính.

1. Cài đặt Git.
   - Windows: cài [Git for Windows](https://git-scm.com/downloads/win). Bộ cài có sẵn **Git Credential Manager**, nên lần `git push` đầu tiên sẽ mở trình duyệt để bạn đăng nhập GitHub. Bạn không cần tạo personal access token.
   - macOS: `brew install git`. Linux: cài `git` bằng trình quản lý gói của bản phân phối.
2. Cài Hugo bản **extended**, phiên bản 0.158.0 trở lên (theme Ananke dùng trong bài cần phiên bản này). Xem [Installation](https://gohugo.io/installation/) để biết mọi cách cài.
   ```bash
   # Windows
   winget install Hugo.Hugo.Extended
   # macOS
   brew install hugo
   ```
3. Mở terminal mới và kiểm tra hai công cụ:
   ```bash
   git --version
   hugo version
   ```
   `hugo version` phải hiển thị `+extended` và phiên bản từ `v0.158.0` trở lên.
4. Khai báo thông tin của bạn cho Git. Các giá trị này xuất hiện trong commit:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "<username>@users.noreply.github.com"
   ```

{{% notice warning %}}
Không bao giờ dán personal access token vào URL clone, script hay khung chat. Ai thấy token đều có thể thao tác trên GitHub dưới danh nghĩa của bạn. Nếu bắt buộc dùng token, hãy để Git Credential Manager lưu nó.
{{% /notice %}}
