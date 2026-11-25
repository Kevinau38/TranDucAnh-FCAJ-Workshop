---
title : "Turn on CloudFront caching"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

While building, the distribution used **CachingDisabled** so every upload was visible at once. The website is finished, so the static files can now be cached at the edge locations. The API is called directly through API Gateway, so caching the site does not affect the chat data.

1. Switch the default behavior to the managed policy **CachingOptimized** (ID `658327ea-f89d-4fab-a63d-7e88639e58f6`):

```bash
source ~/chat.env; cd ~
aws cloudfront get-distribution-config --id $DIST_ID > dist.json && ETAG=$(jq -r .ETag dist.json) && jq '.DistributionConfig.DefaultCacheBehavior.CachePolicyId="658327ea-f89d-4fab-a63d-7e88639e58f6" | .DistributionConfig' dist.json > dist-new.json && aws cloudfront update-distribution --id $DIST_ID --if-match $ETAG --distribution-config file://dist-new.json --query 'Distribution.[Status,DistributionConfig.DefaultCacheBehavior.CachePolicyId]' --output text
```

   - (1) Read the current configuration with its `ETag`, change `CachePolicyId` and send it back.
   - (2) The distribution is `InProgress` with the new cache policy.

   ![Cache policy](/images/7-cloudfront-cache/7-cache-policy.png)

2. In the CloudFront console, open the **Behaviors** tab (1) of the distribution. The default behavior now uses **Managed-CachingOptimized** (2).

   ![Behavior](/images/7-cloudfront-cache/7-behavior.png)

3. When the deployment is done, create an invalidation so that the edge locations drop old copies, then request the same page three times:

```bash
aws cloudfront wait distribution-deployed --id $DIST_ID
aws cloudfront create-invalidation --distribution-id $DIST_ID --paths '/*' --query Invalidation.Status --output text
sleep 60; for i in 1 2 3; do curl -sI https://$CF_DOMAIN/index.html | grep -i x-cache; done
```

   - (1) The first request is a `Miss` and CloudFront reads the file from S3. The next requests are a `Hit` from the cache.

   ![Cache hit](/images/7-cloudfront-cache/7-cache-hit.png)

{{% notice note %}}
From now on, after you upload a changed file, create an invalidation for its path (for example `/js/site.js`), or the users keep getting the cached copy until it expires.
{{% /notice %}}
