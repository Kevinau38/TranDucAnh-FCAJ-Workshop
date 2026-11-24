---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### GitHub Pages

GitHub Pages phục vụ các file HTML, CSS và JavaScript trong một repository thành website qua HTTPS. Repository public trên tài khoản GitHub miễn phí được dùng tính năng này mà không mất phí. Có hai loại site:

| Loại site | Tên repository | Địa chỉ |
|---|---|---|
| User (hoặc organization) site | chính xác `<username>.github.io` | `https://<username>.github.io/` |
| Project site | tên bất kỳ, ví dụ `workshop1` | `https://<username>.github.io/workshop1/` |

Mỗi tài khoản có một user site và có thể có nhiều project site.

#### Nguồn publish

Trong **Settings, Pages** của repository, bạn chọn cách website được publish:

- **Deploy from a branch**: GitHub publish các file có sẵn trong một branch và thư mục (ví dụ `main` và `/ (root)`). Bạn phải commit sẵn các file HTML đã build.
- **GitHub Actions**: một workflow trong `.github/workflows/` tạo ra website và deploy. Workflow có thể chỉ upload các file bạn đã tự build (**Static HTML**), hoặc tự build website từ source mỗi lần push (**Hugo**).

#### Hugo và baseURL

Hugo ghi website hoàn chỉnh vào thư mục `public/`. Các đường dẫn trong website phụ thuộc vào thiết lập **baseURL**, nên một website sẽ chạy tại `https://<username>.github.io/workshop1/` phải được build với baseURL đó, nếu không CSS và link sẽ trỏ sai chỗ. Ở phần 5, workflow Hugo tự đặt baseURL theo địa chỉ của Pages.

#### Bạn sẽ làm gì

| Phần | Repository | Nguồn publish | Nội dung push |
|---|---|---|---|
| 3 | `<username>.github.io` | Deploy from a branch | các file đã build trong `public/` |
| 4 | `workshop1` | GitHub Actions, Static HTML | các file đã build trong `public/` |
| 5 | `workshop2` | GitHub Actions, Hugo | source code Hugo |
