---
title : "Publish the site with CloudFront"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

1. Open the **CloudFront** console and choose **Create distribution**.
   - (1) On **Choose a plan**, select **Pay as you go**, then choose **Choose pay-as-you-go** at the bottom. You only pay for the requests of the lab.

   ![Pay as you go](/images/2-static-website/2.2-cf-plan.png)

2. On **Get started**:
   - (1) **Distribution name**: `fcj-chat-app`.
   - (2) **Distribution type**: **Single website configuration**.
   - Choose **Next**.

   ![Distribution name](/images/2-static-website/2.2-cf-name.png)

3. On **Specify origin**, keep **Amazon S3** as origin type.
   - (1) **S3 origin**: choose **Browse S3**, filter by the bucket name and select your bucket.
   - (2) Keep **Allow private S3 bucket access to CloudFront** selected. CloudFront creates an Origin Access Control and adds a bucket policy that only allows this distribution.

   ![S3 origin](/images/2-static-website/2.2-cf-origin.png)

4. Scroll down:
   - (1) **Cache settings**: **Customize cache settings**.
   - (2) **Cache policy**: **CachingDisabled**. While you build the site you upload new files often, so CloudFront should always read the latest version. You turn caching on in part 7.
   - Choose **Next**.

   ![Cache settings](/images/2-static-website/2.2-cf-cache.png)

5. On **Enable security**:
   - (1) Select **Do not enable security protections**. AWS WAF is billed per month and is not needed for this lab.
   - Choose **Next**, review and choose **Create distribution**.

   ![No WAF](/images/2-static-website/2.2-cf-waf.png)

6. Wait until **Last modified** shows a date instead of **Deploying** (about 5 minutes). Save the distribution ID and domain name in CloudShell:

   ```bash
   echo 'export DIST_ID=<distribution-id>' >> ~/chat.env && source ~/chat.env
   echo "export CF_DOMAIN=$(aws cloudfront get-distribution --id $DIST_ID --query Distribution.DomainName --output text)" >> ~/chat.env && source ~/chat.env
   ```

7. Open `https://<distribution-domain>/chats.html` in the browser.
   - (1) The navigation bar of the chat app appears.
   - (2) There is no conversation yet, only the **Start a new chat** button.

   ![Site through CloudFront](/images/2-static-website/2.2-site-v1.png)

8. Compare the two paths to the same file:

   ```bash
   curl -s -o /dev/null -w 'CloudFront: %{http_code}\n' https://$CF_DOMAIN/chats.html
   curl -s -o /dev/null -w 'S3 direct:  %{http_code}\n' https://$BUCKET.s3.ap-northeast-1.amazonaws.com/chats.html
   ```

   - (1) CloudFront returns `200`, while the direct S3 URL returns `403`. Only the distribution can read the bucket.

   ![Access check](/images/2-static-website/2.2-access-check.png)
