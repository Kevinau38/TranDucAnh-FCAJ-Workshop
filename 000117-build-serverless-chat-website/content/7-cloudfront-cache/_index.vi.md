---
title : "Bật cache cho CloudFront"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Trong lúc xây dựng, distribution dùng **CachingDisabled** để mỗi lần upload là thấy ngay. Website đã hoàn chỉnh nên giờ có thể cache file tĩnh ở các edge location. Website gọi API trực tiếp qua API Gateway nên cache website không ảnh hưởng dữ liệu chat.

1. Chuyển default behavior sang managed policy **CachingOptimized** (ID `658327ea-f89d-4fab-a63d-7e88639e58f6`):

```bash
source ~/chat.env; cd ~
aws cloudfront get-distribution-config --id $DIST_ID > dist.json && ETAG=$(jq -r .ETag dist.json) && jq '.DistributionConfig.DefaultCacheBehavior.CachePolicyId="658327ea-f89d-4fab-a63d-7e88639e58f6" | .DistributionConfig' dist.json > dist-new.json && aws cloudfront update-distribution --id $DIST_ID --if-match $ETAG --distribution-config file://dist-new.json --query 'Distribution.[Status,DistributionConfig.DefaultCacheBehavior.CachePolicyId]' --output text
```

   - (1) Đọc cấu hình hiện tại kèm `ETag`, đổi `CachePolicyId` rồi gửi lại.
   - (2) Distribution ở trạng thái `InProgress` với cache policy mới.

   ![Cache policy](/images/7-cloudfront-cache/7-cache-policy.png)

2. Trong console CloudFront, mở tab **Behaviors** (1) của distribution. Default behavior giờ dùng **Managed-CachingOptimized** (2).

   ![Behavior](/images/7-cloudfront-cache/7-behavior.png)

3. Khi deploy xong, tạo invalidation để các edge location bỏ bản cũ, rồi gọi cùng một trang ba lần:

```bash
aws cloudfront wait distribution-deployed --id $DIST_ID
aws cloudfront create-invalidation --distribution-id $DIST_ID --paths '/*' --query Invalidation.Status --output text
sleep 60; for i in 1 2 3; do curl -sI https://$CF_DOMAIN/index.html | grep -i x-cache; done
```

   - (1) Request đầu là `Miss`, CloudFront đọc file từ S3. Các request sau là `Hit` từ cache.

   ![Cache hit](/images/7-cloudfront-cache/7-cache-hit.png)

{{% notice note %}}
Từ giờ, sau khi upload file đã sửa, hãy tạo invalidation cho đường dẫn của file đó (ví dụ `/js/site.js`), nếu không người dùng vẫn nhận bản cache cho đến khi hết hạn.
{{% /notice %}}
