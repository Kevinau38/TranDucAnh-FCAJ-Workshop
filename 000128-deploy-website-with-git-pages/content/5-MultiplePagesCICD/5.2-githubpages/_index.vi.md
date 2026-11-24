---
title : "Thêm workflow Hugo"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---


1. Trong `workshop2`, chọn **Settings**, rồi **Pages**, và đặt **Source** là **GitHub Actions** (1). Workflow Hugo không nằm trong các thẻ gợi ý, nên chọn **browse all workflows** (2).

![Browse all workflows](/images/5-MultiplePagesCICD/5.2-actions-source.png)

2. Ở ô tìm kiếm, nhập `hugo` (1). Trên thẻ **Hugo**, chọn **Configure** (2).

![Workflow Hugo](/images/5-MultiplePagesCICD/5.2-search-hugo.png)

3. GitHub mở file `.github/workflows/hugo.yml` (1).

![hugo.yml](/images/5-MultiplePagesCICD/5.2-hugo-editor.png)

4. Cuộn xuống job `build` và đổi `HUGO_VERSION` sang phiên bản mà theme Ananke hỗ trợ, ví dụ `0.170.0` hoặc phiên bản mà `hugo version` hiển thị trên máy bạn (1). Giá trị mặc định trong template cũ hơn và build sẽ lỗi. Sau đó chọn **Commit changes...** (2) và commit thẳng vào `main`.

![HUGO_VERSION](/images/5-MultiplePagesCICD/5.2-hugo-version.png)

Các phần quan trọng của workflow:
```yaml
    env:
      HUGO_VERSION: 0.170.0
...
      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
```
Workflow checkout repository kèm submodule nên có sẵn theme, và lấy baseURL từ cấu hình Pages, nên bạn không cần tự gõ đường dẫn project.
