---
title : "Tạo website Hugo"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---


Bạn tạo website mẫu theo [Hugo quick start](https://gohugo.io/getting-started/quick-start/) chính thức với theme **Ananke**.

1. Trong một thư mục làm việc, chạy:
   ```bash
   hugo new project quickstart
   cd quickstart
   git init
   git submodule add https://github.com/gohugo-ananke/ananke themes/ananke
   echo "theme = 'ananke'" >> hugo.toml
   hugo new content content/posts/my-first-post.md
   ```
2. Mở `hugo.toml` và đổi tiêu đề:
   ```toml
   baseURL = 'https://example.org/'
   locale = 'en-us'
   title = 'FCJ Hugo on GitHub Pages'
   theme = 'ananke'
   ```
3. Mở `content/posts/my-first-post.md`, đặt `draft = false` và thêm một dòng nội dung:
   ```markdown
   +++
   date = '2026-11-17T13:14:46+07:00'
   draft = false
   title = 'My First Post'
   +++
   This site is built with **Hugo** and published with **GitHub Pages**.
   ```
4. Tạo file `.gitignore` trong thư mục `quickstart` để không commit kết quả build cùng source:
   ```text
   public/
   resources/_gen/
   .hugo_build.lock
   ```
5. Xem trước website trên máy:
   ```bash
   hugo server
   ```
   Mở `http://localhost:1313/` trên trình duyệt, sau đó nhấn **Ctrl+C** trong terminal để dừng server.

{{% notice info %}}
`hugo server` build website trong bộ nhớ. Lệnh `hugo` (không có `server`) ghi website hoàn chỉnh vào thư mục `public/`; bạn sẽ dùng lệnh này ở phần 3 và 4.
{{% /notice %}}
