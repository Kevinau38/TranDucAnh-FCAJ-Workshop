---
title : "Tạo Auto Scaling group và load balancer"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Khi một instance đã chạy WordPress đúng, bạn biến nó thành một tầng web có tính sẵn sàng cao:

| Tài nguyên | Tên | Ghi chú |
|---|---|---|
| AMI | `wordpress-ami` | Image của `wordpress-setup` gồm Apache, PHP, WordPress và `wp-config.php` |
| Launch template | `wordpress-lt` | AMI, `t4g.micro`, `WebServer-SG`, `FCJ-WordPress-EC2-Role`, IMDSv2, **không có IP public** |
| Target group | `wordpress-tg` | HTTP 80, đường dẫn health check `/health.html` |
| Application Load Balancer | `wordpress-alb` | Internet-facing, nằm trong **hai public subnet**, dùng `ALB-SG` |
| Auto Scaling group | `wordpress-asg` | Nằm trong **hai private subnet**, tối thiểu 1, mong muốn 2, tối đa 3, health check theo ELB, target tracking theo CPU 50% |

{{% notice info %}}
Load balancer là tài nguyên duy nhất nằm trong public subnet. Các instance do Auto Scaling group khởi tạo vẫn nằm trong private subnet và chỉ nhận traffic từ `ALB-SG`.
{{% /notice %}}

#### Tạo các tài nguyên

1. Trong **AWS CloudShell**, chạy script sau. Script tạo AMI và chờ đến khi AMI sẵn sàng (khoảng 5 phút), sau đó tạo launch template, target group, load balancer, listener, Auto Scaling group và scaling policy:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   IID=$(aws ec2 describe-instances --filters Name=tag:Name,Values=wordpress-setup Name=instance-state-name,Values=running --query "Reservations[0].Instances[0].InstanceId" --output text)
   PRIV=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values='*private*' --query 'Subnets[].SubnetId' --output text | tr '\t' ',')
   PUB=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values='*public*' --query 'Subnets[].SubnetId' --output text)
   sg(){ aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=$1 --query 'SecurityGroups[0].GroupId' --output text; }
   WEBSG=$(sg WebServer-SG); ALBSG=$(sg ALB-SG)
   AMI=$(aws ec2 create-image --instance-id $IID --name wordpress-ami --description 'WordPress on Amazon Linux 2023' --tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=wordpress-ami}]' --query ImageId --output text)
   echo "AMI $AMI"; aws ec2 wait image-available --image-ids $AMI; echo "AMI available"
   aws ec2 create-launch-template --launch-template-name wordpress-lt --version-description v1 --launch-template-data "{\"ImageId\":\"$AMI\",\"InstanceType\":\"t4g.micro\",\"SecurityGroupIds\":[\"$WEBSG\"],\"IamInstanceProfile\":{\"Name\":\"FCJ-WordPress-EC2-Role\"},\"MetadataOptions\":{\"HttpTokens\":\"required\",\"HttpEndpoint\":\"enabled\"},\"TagSpecifications\":[{\"ResourceType\":\"instance\",\"Tags\":[{\"Key\":\"Name\",\"Value\":\"wordpress-asg\"}]}]}" --query 'LaunchTemplate.LaunchTemplateName' --output text
   TG=$(aws elbv2 create-target-group --name wordpress-tg --protocol HTTP --port 80 --vpc-id $VPC --target-type instance --health-check-path /health.html --health-check-interval-seconds 15 --healthy-threshold-count 2 --query 'TargetGroups[0].TargetGroupArn' --output text); echo TG-OK
   ALB=$(aws elbv2 create-load-balancer --name wordpress-alb --type application --scheme internet-facing --subnets $PUB --security-groups $ALBSG --query 'LoadBalancers[0].LoadBalancerArn' --output text); echo ALB-OK
   aws elbv2 create-listener --load-balancer-arn $ALB --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=$TG --query 'Listeners[0].Port' --output text
   aws autoscaling create-auto-scaling-group --auto-scaling-group-name wordpress-asg --launch-template LaunchTemplateName=wordpress-lt,Version='$Latest' --min-size 1 --max-size 3 --desired-capacity 2 --vpc-zone-identifier $PRIV --target-group-arns $TG --health-check-type ELB --health-check-grace-period 120 && echo ASG-OK
   aws autoscaling put-scaling-policy --auto-scaling-group-name wordpress-asg --policy-name cpu50 --policy-type TargetTrackingScaling --target-tracking-configuration '{"PredefinedMetricSpecification":{"PredefinedMetricType":"ASGAverageCPUUtilization"},"TargetValue":50}' --query PolicyName --output text
   ```

2. Script in ra một dòng cho mỗi tài nguyên được tạo.

   ![Kết quả script](/images/4-autoscaling/4.0-cli-output.png)

#### Kiểm tra các tài nguyên

1. **AMI** – mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **AMIs**. (1) `wordpress-ami` đã sẵn sàng và (2) kiến trúc là `arm64`.

   ![AMI](/images/4-autoscaling/4.1-ami.png)

2. **Target group** – mở **Target groups** → `wordpress-tg` → **Targets**. Sau khoảng hai phút, (1) hai instance của Auto Scaling group, mỗi instance ở một Availability Zone, đều **Healthy**.

   ![Target khoẻ mạnh](/images/4-autoscaling/4.3-tg-healthy.png)

3. **Load balancer** – mở **Load balancers**. (1) `wordpress-alb` ở trạng thái **Active**, loại **Internet-facing** và dùng **2 Availability Zone**.

   ![Load balancer](/images/4-autoscaling/4.4-alb.png)

4. **Auto Scaling group** – mở **Auto Scaling groups** → `wordpress-asg` → **Instance management**. (1) Desired capacity là 2 và giới hạn scaling là 1–3. (2) Cả hai instance đều **InService** và **Healthy**.

   ![Auto Scaling group](/images/4-autoscaling/4.5-asg.png)

#### Kiểm tra website và xoá instance cài đặt

1. Lấy DNS name của load balancer và mở trên trình duyệt:

   ```bash
   aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].DNSName" --output text
   ```

2. Website WordPress hiển thị (1) tiêu đề **FCJ WordPress on AWS** và (2) bài viết đầu tiên **Hello world!**.

   ![WordPress qua load balancer](/images/4-autoscaling/4.6-site-alb.png)

3. Instance `wordpress-setup` không còn cần thiết vì Auto Scaling group đã chạy các bản sao từ AMI của nó. Hãy terminate instance này:

   ```bash
   aws ec2 terminate-instances --instance-ids $IID --query "TerminatingInstances[0].CurrentState.Name" --output text
   ```

{{% notice note %}}
WordPress lưu bài viết và cấu hình trong RDS, vì vậy mọi instance trong Auto Scaling group đều hiển thị cùng một nội dung. File media tải lên được lưu trên ổ đĩa của instance; với website production, bạn nên lưu chúng trên Amazon EFS hoặc Amazon S3.
{{% /notice %}}
