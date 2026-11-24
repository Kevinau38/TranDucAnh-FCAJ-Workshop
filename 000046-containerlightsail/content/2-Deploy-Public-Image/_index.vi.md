---
title : "Triển khai public image"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

**Deployment** mô tả các container chạy trên service: image, các cổng mở, và container nào nhận lưu lượng của public domain. Mỗi lần lưu deployment, Lightsail tạo một **version** mới.

1. Khi service ở trạng thái **Ready**, mở tab **Deployments** và chọn **Create your first deployment**.

![Tạo deployment đầu tiên](/images/2/1-ready.png)

2. Điền thông tin container:
   - **Container name**: `web`
   - **Image**: `nginx:stable-alpine`. Nếu không ghi tên registry, Lightsail kéo image từ Docker Hub.
   - Chọn **Add open ports**, nhập `80` và giữ **HTTP**.

![Cấu hình container](/images/2/2-container.png)

3. Ở mục **Public endpoint**, chọn `web`. Cổng là `80` và health check path là `/`. Lightsail chuyển lưu lượng của public domain tới container này và kiểm tra sức khoẻ qua đường dẫn này. Chọn **Save and deploy**.

![Public endpoint](/images/2/3-endpoint.png)

4. Trạng thái chuyển sang **Deploying** (version 1). Cần khoảng 3–5 phút để chuyển sang **Running**.

![Đang triển khai](/images/2/4-deploying.png)

5. Mở **Public domain** trên trình duyệt, bạn sẽ thấy trang **Welcome to nginx!**. Bạn cũng có thể kiểm tra từ CloudShell:

```bash
URL=$(aws lightsail get-container-services --region ap-southeast-1 \
  --service-name fcj-container --query 'containerServices[0].url' --output text)
echo $URL
curl -s $URL | grep -E '<title>|<h1>'
```

![Kết quả từ nginx](/images/2/5-curl-nginx.png)
