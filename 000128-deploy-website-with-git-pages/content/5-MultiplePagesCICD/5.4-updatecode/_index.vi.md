---
title : "Cập nhật source code"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---


1. Commit ở bước 5.2 được tạo trên GitHub, nên trước tiên hãy kéo commit đó về máy. Trong thư mục `quickstart`, chạy:
   ```bash
   git pull
   ```
2. Mở `content/posts/my-first-post.md` và thêm một dòng mới ở cuối, ví dụ:
   ```markdown
   This line was added later. GitHub Actions rebuilt and redeployed the site automatically after `git push`.
   ```
3. Commit và push thay đổi:
   ```bash
   git add .
   git commit -m "Edit Source Code"
   git push
   ```
Bạn chỉ push Markdown. Bạn không chạy `hugo` và không đụng đến file HTML nào.
