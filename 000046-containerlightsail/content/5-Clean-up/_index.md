---
title : "Clean up"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Deleting the container service also deletes its deployments and stored images.

1. On the page of `fcj-container`, open the menu (⋮) next to the tabs and choose **Delete**.

![Delete menu](/images/5/1-menu.png)

2. Choose **Delete container service**.

![Delete container service](/images/5/2-delete.png)

3. Confirm with **Yes, delete**.

![Confirm](/images/5/3-confirm.png)

4. In CloudShell, remove the local image, the working folder and lightsailctl. The last command must return `[]`.

```bash
aws lightsail delete-container-service --region ap-southeast-1 --service-name fcj-container
docker rmi -f fcj-nginx:v1
rm -rf ~/fcj-nginx ~/.local/bin/lightsailctl
aws lightsail get-container-services --region ap-southeast-1 \
  --query "containerServices[?containerServiceName=='fcj-container'].state"
```
