---
title : "Put Amazon CloudFront in front of the site"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

Finally you give the site an HTTPS address with **Amazon CloudFront** and close the load balancer to everyone else. After this step the only public entry point is CloudFront.

| Setting | Value | Why |
|---|---|---|
| Origin | `wordpress-alb`, HTTP only | The load balancer listens on port 80 |
| Viewer protocol policy | Redirect HTTP to HTTPS | Visitors always use HTTPS |
| Cache policy | `Managed-CachingDisabled` | WordPress pages are dynamic; caching is off to keep the lab simple |
| Origin request policy | `Managed-AllViewerAndCloudFrontHeaders-2022-06` | Forwards the `Host` header and `CloudFront-Forwarded-Proto`, so `wp-config.php` builds HTTPS links with the CloudFront domain |
| `ALB-SG` inbound | Managed prefix list `com.amazonaws.global.cloudfront.origin-facing` | Only CloudFront can reach the load balancer |

#### Create the distribution

1. In **AWS CloudShell**, run:

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

2. Open the [CloudFront console](https://console.aws.amazon.com/cloudfront/) and select the new distribution. (1) Copy the **Distribution domain name**. (2) **Last modified** shows **Deploying** for a few minutes.

   ![CloudFront distribution](/images/6-cloudfront/6.1-cf-dist.png)

3. On the **Behaviors** tab, (1) the default behavior sends requests to `wordpress-alb`, redirects HTTP to HTTPS and uses the two managed policies.

   ![Behavior](/images/6-cloudfront/6.1-cf-behavior.png)

#### Allow only CloudFront to reach the load balancer

1. Replace the rule `HTTP from 0.0.0.0/0` in `ALB-SG` with a rule for the CloudFront managed prefix list:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   PL=$(aws ec2 describe-managed-prefix-lists --filters Name=prefix-list-name,Values=com.amazonaws.global.cloudfront.origin-facing --query "PrefixLists[0].PrefixListId" --output text)
   ALBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=ALB-SG --query "SecurityGroups[0].GroupId" --output text)
   aws ec2 authorize-security-group-ingress --group-id $ALBSG --ip-permissions "IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds=[{PrefixListId=$PL,Description='HTTP from CloudFront only'}]" --query Return
   aws ec2 revoke-security-group-ingress --group-id $ALBSG --protocol tcp --port 80 --cidr 0.0.0.0/0 --query Return
   ```

2. Open **Security Groups** → `ALB-SG` → **Inbound rules**. (1) The only rule is HTTP 80 from the prefix list `pl-...` (`com.amazonaws.global.cloudfront.origin-facing`).

   ![ALB-SG restricted to CloudFront](/images/6-cloudfront/6.2-alb-sg-cf.png)

#### Test

1. When the distribution status is **Deployed**, run:

   ```bash
   CF=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].DomainName" --output text)
   ALBDNS=$(aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].DNSName" --output text)
   curl -s https://$CF/ | grep -o '<title>.*</title>'
   curl -s -m 8 -o /dev/null -w 'ALB direct: %{http_code}\n' http://$ALBDNS/ || echo 'ALB direct: blocked'
   echo https://$CF/
   ```

   - (1) The site answers through CloudFront over HTTPS.
   - (2) A direct request to the load balancer times out, because `ALB-SG` accepts only CloudFront.

   ![Test](/images/6-cloudfront/6.3-verify.png)

2. Open the CloudFront address in your browser and select the post. (1) WordPress builds its links with the CloudFront domain and HTTPS.

   ![WordPress through CloudFront](/images/6-cloudfront/6.4-site-cf.png)
