---
title : "Create the Auto Scaling group and load balancer"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Now that one instance runs WordPress correctly, you turn it into a highly available web tier:

| Resource | Name | Notes |
|---|---|---|
| AMI | `wordpress-ami` | Image of `wordpress-setup` with Apache, PHP, WordPress and `wp-config.php` |
| Launch template | `wordpress-lt` | AMI, `t4g.micro`, `WebServer-SG`, `FCJ-WordPress-EC2-Role`, IMDSv2, **no public IP** |
| Target group | `wordpress-tg` | HTTP 80, health check path `/health.html` |
| Application Load Balancer | `wordpress-alb` | Internet-facing, in the **two public subnets**, `ALB-SG` |
| Auto Scaling group | `wordpress-asg` | In the **two private subnets**, minimum 1, desired 2, maximum 3, ELB health checks, target tracking on 50% CPU |

{{% notice info %}}
The load balancer is the only resource in the public subnets. The instances that the Auto Scaling group launches stay in the private subnets and receive traffic only from `ALB-SG`.
{{% /notice %}}

#### Create the resources

1. In **AWS CloudShell**, run the following script. It creates the AMI and waits until it is available (about 5 minutes), then creates the launch template, target group, load balancer, listener, Auto Scaling group and scaling policy:

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

2. The script prints a line for each resource that it creates.

   ![Script output](/images/4-autoscaling/4.0-cli-output.png)

#### Check the resources

1. **AMI** – open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **AMIs**. (1) `wordpress-ami` is available and (2) its architecture is `arm64`.

   ![AMI](/images/4-autoscaling/4.1-ami.png)

2. **Target group** – open **Target groups** → `wordpress-tg` → **Targets**. After about two minutes, (1) the two instances from the Auto Scaling group, one in each Availability Zone, are **Healthy**.

   ![Healthy targets](/images/4-autoscaling/4.3-tg-healthy.png)

3. **Load balancer** – open **Load balancers**. (1) `wordpress-alb` is **Active**, **Internet-facing** and uses **2 Availability Zones**.

   ![Load balancer](/images/4-autoscaling/4.4-alb.png)

4. **Auto Scaling group** – open **Auto Scaling groups** → `wordpress-asg` → **Instance management**. (1) The desired capacity is 2 and the scaling limits are 1–3. (2) Both instances are **InService** and **Healthy**.

   ![Auto Scaling group](/images/4-autoscaling/4.5-asg.png)

#### Test the website and remove the setup instance

1. Get the DNS name of the load balancer and open it in your browser:

   ```bash
   aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].DNSName" --output text
   ```

2. The WordPress site opens with (1) the title **FCJ WordPress on AWS** and (2) the first post **Hello world!**.

   ![WordPress through the load balancer](/images/4-autoscaling/4.6-site-alb.png)

3. The instance `wordpress-setup` is no longer needed, because the Auto Scaling group runs copies of its AMI. Terminate it:

   ```bash
   aws ec2 terminate-instances --instance-ids $IID --query "TerminatingInstances[0].CurrentState.Name" --output text
   ```

{{% notice note %}}
WordPress stores its posts and settings in RDS, so every instance in the Auto Scaling group shows the same content. Uploaded media files are stored on the instance disk; for a production site you would store them on Amazon EFS or Amazon S3.
{{% /notice %}}
