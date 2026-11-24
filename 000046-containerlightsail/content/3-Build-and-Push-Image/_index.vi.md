---
title : "Build và push image của bạn"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

Bây giờ bạn build một image thay trang chào mặc định của nginx bằng trang của riêng bạn, rồi push lên container service. Bạn làm việc này trong **AWS CloudShell**, nơi đã có sẵn Docker và AWS CLI.

1. Mở [CloudShell](https://console.aws.amazon.com/cloudshell/home?region=ap-southeast-1) ở **ap-southeast-1**.

2. Cài **lightsailctl**, plugin mà AWS CLI dùng để push image lên Lightsail. CloudShell giữ lại thư mục home, nên bạn cài vào `~/.local/bin`, thư mục đã có sẵn trong `PATH`.

```bash
mkdir -p ~/.local/bin
curl -sSL https://s3.us-west-2.amazonaws.com/lightsailctl/latest/linux-amd64/lightsailctl \
  -o ~/.local/bin/lightsailctl
chmod +x ~/.local/bin/lightsailctl
lightsailctl --version
```

3. Tạo một thư mục chứa trang web và `Dockerfile`. Image dựa trên nginx từ **Amazon ECR Public Gallery** và chép trang của bạn đè lên trang mặc định.

```bash
mkdir -p ~/fcj-nginx && cd ~/fcj-nginx

cat > index.html <<'EOF'
<!DOCTYPE html>
<html><head><meta charset="utf-8"><title>FCJ Lightsail Container</title></head>
<body style="font-family:sans-serif;text-align:center;margin-top:15%">
<h1>Welcome to the Amazon Lightsail Container workshop</h1>
<p>This page is served by an image built in AWS CloudShell.</p>
</body></html>
EOF

cat > Dockerfile <<'EOF'
FROM public.ecr.aws/nginx/nginx:stable-alpine
COPY index.html /usr/share/nginx/html/index.html
EOF
```

4. Build image và chạy thử trên cổng 8080. Lệnh `curl` phải in ra dòng `<h1>` của trang bạn vừa tạo.

```bash
docker build -t fcj-nginx:v1 .
docker run -d --name fcj-test -p 8080:80 fcj-nginx:v1
sleep 2 && curl -s localhost:8080 | grep h1
docker images fcj-nginx
```

![Build và chạy thử image](/images/3/1-build.png)

5. Xoá container chạy thử rồi push image lên container service. `--label` là tên của image bên trong Lightsail.

```bash
docker rm -f fcj-test
aws lightsail push-container-image --region ap-southeast-1 \
  --service-name fcj-container --label fcj-nginx --image fcj-nginx:v1
```

Lệnh kết thúc bằng `Image "fcj-nginx:v1" registered.` cùng tên dùng trong deployment, ví dụ `:fcj-container.fcj-nginx.6`. Số ở cuối tăng lên mỗi lần bạn push với cùng label.

![Push image xong](/images/3/2-push.png)

6. Trên console, mở tab **Images** của `fcj-container`. Image vừa push nằm trong danh sách **Stored images**.

![Stored images](/images/3/3-images.png)

{{% notice tip %}}
Bạn cũng có thể push image lên **Amazon ECR** hoặc dùng bất kỳ public registry nào. Image lưu trong Lightsail không mất phí và bị xoá cùng container service.
{{% /notice %}}
