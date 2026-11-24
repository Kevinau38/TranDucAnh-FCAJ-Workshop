---
title : "Push source code"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---


1. Thư mục `quickstart` đã là một Git repository, với theme Ananke được thêm dưới dạng **submodule**. Push lên `workshop2`:
   ```bash
   # in the quickstart folder (already a Git repository from step 2.3)
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git remote add origin https://github.com/<username>/workshop2.git
   git push -u origin main
   ```
2. Tải lại repository `workshop2`. Repository chứa source Hugo: `archetypes`, `content/posts`, `themes`, `.gitignore`, `.gitmodules` và `hugo.toml`. Không có thư mục `public/`, vì `.gitignore` đã loại bỏ nó.

![Source Hugo trong workshop2](/images/5-MultiplePagesCICD/5.1-repo-source.png)
