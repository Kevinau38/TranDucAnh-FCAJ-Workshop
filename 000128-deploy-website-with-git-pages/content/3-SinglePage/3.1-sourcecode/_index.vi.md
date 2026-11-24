---
title : "Push website đã build"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Build website với địa chỉ của user site, clone repository trống bên cạnh thư mục `quickstart`, chép các file trong `public/` vào và push:
   ```bash
   # in the quickstart folder
   hugo --baseURL https://<username>.github.io/

   # in a separate folder, clone the empty repository
   cd ..
   git clone https://github.com/<username>/<username>.github.io.git
   cp -r quickstart/public/* <username>.github.io/

   cd <username>.github.io
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git push -u origin main
   ```
   Trên Windows PowerShell, dùng `Copy-Item -Recurse quickstart\public\* <username>.github.io\` thay cho `cp`.
2. Lần `git push` đầu tiên, Git Credential Manager mở một cửa sổ trình duyệt. Đăng nhập GitHub và cho phép truy cập.
3. Tải lại trang repository trên GitHub. Repository giờ chứa website đã build: `index.html`, `404.html`, `sitemap.xml` và các thư mục `posts`, `tags`, `categories`.

![Các file đã build trong repository](/images/3-SinglePage/3.1-repo-pushed.png)
