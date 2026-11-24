---
title : "Push website đã build"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


1. Build lại website với địa chỉ của project site. Đường dẫn `/workshop1/` ở cuối là bắt buộc, nếu không link và CSS sẽ trỏ về gốc của user site.
   ```bash
   # in the quickstart folder, rebuild for the project path
   hugo --baseURL https://<username>.github.io/workshop1/

   cd ..
   git clone https://github.com/<username>/workshop1.git
   cp -r quickstart/public/* workshop1/

   cd workshop1
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git push -u origin main
   ```
2. Tải lại repository `workshop1` trên GitHub và kiểm tra repository đã có `index.html` cùng các file đã build khác.

{{% notice info %}}
Hãy chạy lại `hugo` cho từng repository. Thư mục `public/` ở phần 3 được build cho một địa chỉ khác nên không dùng lại được.
{{% /notice %}}
