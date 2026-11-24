---
title : "Test the Gateway endpoint"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


#### Create an S3 bucket

1. Open the **S3 console → Create bucket**.
2. Keep **General purpose**, enter a globally unique **Bucket name** (for example `fcj-s3vpce-<your-name>`), keep the other settings and choose **Create bucket**.

![bucket name](/images/3-s3-vpc/3.2-bucket-name.png)
![bucket created](/images/3-s3-vpc/3.2-bucket-done.png)

#### Connect to the instance with Session Manager

1. Open **Systems Manager → Session Manager → Start session**.
2. Select **Test-Gateway-Endpoint** (it runs in VPC Cloud) and choose **Start session**.

![ssm target](/images/3-s3-vpc/3.2-ssm-target.png)

{{% notice note %}}
Session Manager reaches this instance through the SSM interface endpoints created by the base stack – no SSH key, no public IP, no bastion host.
{{% /notice %}}

#### Upload a file through the gateway endpoint

Run the following commands in the session (replace the bucket name and Region with yours):

```bash
cd ~ && fallocate -l 1G testfile.xyz && ls -lh testfile.xyz
aws s3 cp testfile.xyz s3://<your-bucket-name> --region <your-region>
curl -s -m 5 -o /dev/null https://aws.amazon.com || echo "No Internet access from this subnet"
```

The 1 GB file is uploaded successfully, while the request to a public website times out – proving the upload went through the **gateway endpoint**, not the Internet.

![upload](/images/3-s3-vpc/3.2-upload.png)

#### Check the object

Open your bucket in the S3 console. **testfile.xyz** (1 GB) is in the bucket.

![object](/images/3-s3-vpc/3.2-object.png)

Close the session window when you are done.
