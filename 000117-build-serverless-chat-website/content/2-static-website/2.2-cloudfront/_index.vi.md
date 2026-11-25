---
title : "Phát hành website qua CloudFront"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

1. Mở console **CloudFront**, chọn **Create distribution**.
   - (1) Ở **Choose a plan**, chọn **Pay as you go**, rồi chọn **Choose pay-as-you-go** ở cuối trang. Bạn chỉ trả tiền theo lượng request của bài lab.

   ![Pay as you go](/images/2-static-website/2.2-cf-plan.png)

2. Ở **Get started**:
   - (1) **Distribution name**: `fcj-chat-app`.
   - (2) **Distribution type**: **Single website configuration**.
   - Chọn **Next**.

   ![Tên distribution](/images/2-static-website/2.2-cf-name.png)

3. Ở **Specify origin**, giữ origin type là **Amazon S3**.
   - (1) **S3 origin**: chọn **Browse S3**, lọc theo tên bucket rồi chọn bucket của bạn.
   - (2) Giữ chọn **Allow private S3 bucket access to CloudFront**. CloudFront tự tạo Origin Access Control và thêm bucket policy chỉ cho phép đúng distribution này.

   ![S3 origin](/images/2-static-website/2.2-cf-origin.png)

4. Cuộn xuống:
   - (1) **Cache settings**: **Customize cache settings**.
   - (2) **Cache policy**: **CachingDisabled**. Trong lúc xây dựng website bạn upload file mới liên tục, nên CloudFront cần luôn đọc bản mới nhất. Phần 7 sẽ bật lại cache.
   - Chọn **Next**.

   ![Cache settings](/images/2-static-website/2.2-cf-cache.png)

5. Ở **Enable security**:
   - (1) Chọn **Do not enable security protections**. AWS WAF tính phí theo tháng và không cần cho bài lab.
   - Chọn **Next**, xem lại rồi chọn **Create distribution**.

   ![Không dùng WAF](/images/2-static-website/2.2-cf-waf.png)

6. Đợi đến khi **Last modified** hiện ngày giờ thay cho **Deploying** (khoảng 5 phút). Lưu ID và domain của distribution trong CloudShell:

   ```bash
   echo 'export DIST_ID=<distribution-id>' >> ~/chat.env && source ~/chat.env
   echo "export CF_DOMAIN=$(aws cloudfront get-distribution --id $DIST_ID --query Distribution.DomainName --output text)" >> ~/chat.env && source ~/chat.env
   ```

7. Mở `https://<distribution-domain>/chats.html` trên trình duyệt.
   - (1) Thanh điều hướng của ứng dụng chat hiện ra.
   - (2) Chưa có cuộc trò chuyện nào, chỉ có nút **Start a new chat**.

   ![Website qua CloudFront](/images/2-static-website/2.2-site-v1.png)

8. So sánh hai đường truy cập vào cùng một file:

   ```bash
   curl -s -o /dev/null -w 'CloudFront: %{http_code}\n' https://$CF_DOMAIN/chats.html
   curl -s -o /dev/null -w 'S3 direct:  %{http_code}\n' https://$BUCKET.s3.ap-northeast-1.amazonaws.com/chats.html
   ```

   - (1) CloudFront trả `200`, còn URL S3 trực tiếp trả `403`. Chỉ distribution mới đọc được bucket.

   ![Kiểm tra truy cập](/images/2-static-website/2.2-access-check.png)
