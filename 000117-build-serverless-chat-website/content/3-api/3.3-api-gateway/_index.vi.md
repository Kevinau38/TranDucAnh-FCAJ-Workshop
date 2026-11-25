---
title : "Đặt API Gateway trước Lambda"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

1. Mở console **API Gateway**, chọn **Create an API**, rồi **Build** ở mục **REST API**.
   - (1) **New API**.
   - (2) **API name**: `Chat-API`.
   - (3) **Endpoint type**: **Regional**. Website đã đi qua CloudFront nên API không cần loại edge-optimized.
   - Chọn **Create API**.

   ![Tạo API](/images/3-api/3.3-create-api.png)

2. Chọn **Create resource**:
   - (1) Bật **Proxy resource**.
   - (2) **Resource name**: `{proxy+}`.
   - (3) Chọn **CORS (Cross Origin Resource Sharing)**. API Gateway thêm method `OPTIONS` để trình duyệt được gọi API từ domain của CloudFront.
   - (4) Chọn **Create resource**.

   ![Proxy resource](/images/3-api/3.3-proxy-resource.png)

3. Chọn method `ANY` dưới `/{proxy+}`, chọn **Edit integration**:
   - (1) **Integration type**: **Lambda function**, bật **Lambda proxy integration**. API Gateway chuyển nguyên request cho Lambda và trả về đúng `statusCode`, `headers`, `body` mà function tạo ra.
   - (2) **Lambda function**: `Chat-Proxy`.
   - Chọn **Save**. API Gateway tự thêm quyền gọi function.

   ![Integration](/images/3-api/3.3-integration.png)

4. Mở tab **Test** của `ANY`, chọn method **GET**, nhập `conversations` vào **proxy** rồi chọn **Test**.
   - (1) **Status** là `200`.
   - (2) **Response body** là nội dung của `conversations.json`.

   ![Test API](/images/3-api/3.3-test-api.png)
