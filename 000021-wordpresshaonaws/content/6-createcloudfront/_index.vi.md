---
title : "Đặt Amazon CloudFront phía trước website"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

Cuối cùng, bạn cung cấp địa chỉ HTTPS cho website bằng **Amazon CloudFront** và đóng load balancer với mọi nguồn khác. Sau bước này, điểm truy cập public duy nhất là CloudFront.

| Cấu hình | Giá trị | Lý do |
|---|---|---|
| Origin | `wordpress-alb`, chỉ HTTP | Load balancer lắng nghe ở cổng 80 |
| Viewer protocol policy | Redirect HTTP to HTTPS | Người dùng luôn dùng HTTPS |
| Cache policy | `Managed-CachingDisabled` | Các trang WordPress là nội dung động; tắt cache để bài lab đơn giản |
| Origin request policy | `Managed-AllViewerAndCloudFrontHeaders-2022-06` | Chuyển tiếp header `Host` và `CloudFront-Forwarded-Proto`, để `wp-config.php` tạo link HTTPS với tên miền CloudFront |
| Inbound của `ALB-SG` | Managed prefix list `com.amazonaws.global.cloudfront.origin-facing` | Chỉ CloudFront mới truy cập được load balancer |

#### Tạo distribution

1. Trong **AWS CloudShell**, chạy:

   ```bash
   ALBDNS=$(aws elbv2 describe-load-balancers --names wordpress-alb --query 'LoadBalancers[0].DNSName' --output text)
   CP=$(aws cloudfront list-cache-policies --type managed --query "CachePolicyList.Items[?CachePolicy.CachePolicyConfig.Name=='Managed-CachingDisabled'].CachePolicy.Id" --output text)
   ORP=$(aws cloudfront list-origin-request-policies --type managed --query "OriginRequestPolicyList.Items[?OriginRequestPolicy.OriginRequestPolicyConfig.Name=='Managed-AllViewerAndCloudFrontHeaders-2022-06'].OriginRequestPolicy.Id" --output text)
   cat > cf.json <<JSON
   {"CallerReference":"wordpress-$(date +%s)","Comment":"WordPress on AWS (FCJ)","Enabled":true,"PriceClass":"PriceClass_200",
    "Origins":{"Quantity":1,"Items":[{"Id":"wordpress-alb","DomainName":"$ALBDNS",
      "CustomOriginConfig":{"HTTPPort":80,"HTTPSPort":443,"OriginProtocolPolicy":"http-only"}}]},
    "DefaultCacheBehavior":{"TargetOriginId":"wordpress-alb","ViewerProtocolPolicy":"redirect-to-https",
      "AllowedMethods":{"Quantity":7,"Items":["GET","HEAD","OPTIONS","PUT","POST","PATCH","DELETE"],"CachedMethods":{"Quantity":2,"Items":["GET","HEAD"]}},
      "CachePolicyId":"$CP","OriginRequestPolicyId":"$ORP","Compress":true}}
   JSON
   aws cloudfront create-distribution --distribution-config file://cf.json --query 'Distribution.[Id,DomainName,Status]' --output text
   ```

2. Mở [CloudFront console](https://console.aws.amazon.com/cloudfront/) và chọn distribution vừa tạo. (1) Sao chép **Distribution domain name**. (2) **Last modified** hiển thị **Deploying** trong vài phút.

   ![CloudFront distribution](/images/6-cloudfront/6.1-cf-dist.png)

3. Ở tab **Behaviors**, (1) behavior mặc định gửi request đến `wordpress-alb`, chuyển HTTP sang HTTPS và dùng hai managed policy ở trên.

   ![Behavior](/images/6-cloudfront/6.1-cf-behavior.png)

#### Chỉ cho phép CloudFront truy cập load balancer

1. Thay rule `HTTP từ 0.0.0.0/0` trong `ALB-SG` bằng rule cho managed prefix list của CloudFront:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   PL=$(aws ec2 describe-managed-prefix-lists --filters Name=prefix-list-name,Values=com.amazonaws.global.cloudfront.origin-facing --query "PrefixLists[0].PrefixListId" --output text)
   ALBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=ALB-SG --query "SecurityGroups[0].GroupId" --output text)
   aws ec2 authorize-security-group-ingress --group-id $ALBSG --ip-permissions "IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds=[{PrefixListId=$PL,Description='HTTP from CloudFront only'}]" --query Return
   aws ec2 revoke-security-group-ingress --group-id $ALBSG --protocol tcp --port 80 --cidr 0.0.0.0/0 --query Return
   ```

2. Mở **Security Groups** → `ALB-SG` → **Inbound rules**. (1) Chỉ còn một rule HTTP 80 từ prefix list `pl-...` (`com.amazonaws.global.cloudfront.origin-facing`).

   ![ALB-SG chỉ cho CloudFront](/images/6-cloudfront/6.2-alb-sg-cf.png)

#### Kiểm tra

1. Khi trạng thái distribution là **Deployed**, chạy:

   ```bash
   CF=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].DomainName" --output text)
   ALBDNS=$(aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].DNSName" --output text)
   curl -s https://$CF/ | grep -o '<title>.*</title>'
   curl -s -m 8 -o /dev/null -w 'ALB direct: %{http_code}\n' http://$ALBDNS/ || echo 'ALB direct: blocked'
   echo https://$CF/
   ```

   - (1) Website trả lời qua CloudFront bằng HTTPS.
   - (2) Request trực tiếp đến load balancer bị timeout, vì `ALB-SG` chỉ chấp nhận CloudFront.

   ![Kiểm tra](/images/6-cloudfront/6.3-verify.png)

2. Mở địa chỉ CloudFront trên trình duyệt và chọn bài viết. (1) WordPress tạo link với tên miền CloudFront và HTTPS.

   ![WordPress qua CloudFront](/images/6-cloudfront/6.4-site-cf.png)
