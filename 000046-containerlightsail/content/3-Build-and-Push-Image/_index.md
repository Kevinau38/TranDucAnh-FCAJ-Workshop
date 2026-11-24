---
title : "Build and push your image"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

Now you build an image that replaces the nginx welcome page with your own page, and push it to the container service. You do this in **AWS CloudShell**, which has Docker and the AWS CLI.

1. Open [CloudShell](https://console.aws.amazon.com/cloudshell/home?region=ap-southeast-1) in **ap-southeast-1**.

2. Install **lightsailctl**, the plugin that the AWS CLI uses to push images to Lightsail. CloudShell keeps your home folder, so you install it in `~/.local/bin`, which is already in the `PATH`.

```bash
mkdir -p ~/.local/bin
curl -sSL https://s3.us-west-2.amazonaws.com/lightsailctl/latest/linux-amd64/lightsailctl \
  -o ~/.local/bin/lightsailctl
chmod +x ~/.local/bin/lightsailctl
lightsailctl --version
```

3. Create a folder with a web page and a `Dockerfile`. The image starts from nginx in the **Amazon ECR Public Gallery** and copies your page over the default one.

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

4. Build the image and test it locally on port 8080. The `curl` command must print the `<h1>` line of your page.

```bash
docker build -t fcj-nginx:v1 .
docker run -d --name fcj-test -p 8080:80 fcj-nginx:v1
sleep 2 && curl -s localhost:8080 | grep h1
docker images fcj-nginx
```

![Image built and tested](/images/3/1-build.png)

5. Remove the test container and push the image to the container service. `--label` is a name for the image inside Lightsail.

```bash
docker rm -f fcj-test
aws lightsail push-container-image --region ap-southeast-1 \
  --service-name fcj-container --label fcj-nginx --image fcj-nginx:v1
```

The command ends with `Image "fcj-nginx:v1" registered.` and the name to use in deployments, for example `:fcj-container.fcj-nginx.6`. The number at the end grows each time you push with the same label.

![Image pushed](/images/3/2-push.png)

6. In the console, open the **Images** tab of `fcj-container`. The pushed image is listed under **Stored images**.

![Stored images](/images/3/3-images.png)

{{% notice tip %}}
You can also push images to **Amazon ECR** or use any public registry. Stored images in Lightsail are free and are deleted with the container service.
{{% /notice %}}
