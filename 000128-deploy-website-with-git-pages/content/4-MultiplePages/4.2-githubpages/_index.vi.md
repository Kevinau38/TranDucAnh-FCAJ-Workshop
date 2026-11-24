---
title : "Thêm workflow Static HTML"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


1. Trong repository `workshop1`, chọn **Settings**, rồi **Pages**.
2. Mở danh sách **Source** và chọn **GitHub Actions** (1).

![Chọn GitHub Actions](/images/4-MultiplePages/4.2-source-dropdown.png)

3. GitHub hiển thị nguồn là **GitHub Actions** (1) và gợi ý một số workflow. Trên thẻ **Static HTML**, chọn **Configure** (2).

![Static HTML, Configure](/images/4-MultiplePages/4.2-static-configure.png)

4. GitHub mở trình soạn thảo với file `.github/workflows/static.yml` (1). Workflow chạy mỗi khi có push lên `main`, upload toàn bộ repository làm website và deploy lên Pages. Bạn không cần sửa gì. Chọn **Commit changes...** (2).

![static.yml](/images/4-MultiplePages/4.2-static-editor.png)

5. Giữ commit message **Create static.yml** (1), giữ **Commit directly to the main branch** và chọn **Commit changes** (2).

![Commit static.yml](/images/4-MultiplePages/4.2-commit-dialog.png)

Commit này cũng là một lần push lên `main`, nên workflow chạy ngay.
