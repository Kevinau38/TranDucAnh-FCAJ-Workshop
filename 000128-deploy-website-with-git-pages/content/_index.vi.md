---
title : "Triển khai Website Hugo với GitHub Pages"
weight : 1
chapter : false
---

# Triển khai Website Hugo với GitHub Pages

#### Tổng quan

**GitHub Pages** cho phép host website tĩnh miễn phí, trực tiếp từ một GitHub repository. **Hugo** là công cụ tạo website tĩnh rất nhanh, chuyển các file Markdown thành một website hoàn chỉnh; chính các workshop của AWS Study Group mà bạn đang đọc cũng được build bằng Hugo.

Trong bài lab này, bạn sẽ tạo một website Hugo nhỏ và đưa lên GitHub Pages theo ba cách, từ đơn giản nhất đến một pipeline CI/CD hoàn chỉnh:

1. Một **user site** tại `https://<username>.github.io/`, publish trực tiếp từ một branch.
2. Một **project site** tại `https://<username>.github.io/workshop1/`, publish bằng GitHub Actions workflow **Static HTML**.
3. Một **project site** tại `https://<username>.github.io/workshop2/`, nơi GitHub Actions workflow **Hugo** tự build và deploy website mỗi khi bạn push source code.

#### Kiến trúc

![Kiến trúc bài lab](/images/architecture.png)

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Bạn không còn phải gắn personal access token vào URL khi clone: Git Credential Manager sẽ đăng nhập GitHub qua trình duyệt. Website mẫu giờ được tạo bằng Hugo quick start chính thức với theme Ananke. Toàn bộ bài lab miễn phí, và bạn sẽ xoá ba repository ở bước cuối.
{{% /notice %}}
