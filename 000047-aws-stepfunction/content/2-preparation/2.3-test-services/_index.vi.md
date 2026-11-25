---
title : "Kiểm tra các service mẫu"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Gọi trực tiếp các function bằng `aws lambda invoke`. AWS CLI v2 mặc định coi payload là base64, nên mỗi lệnh thêm `--cli-binary-format raw-in-base64-out` để gửi JSON thường.

#### Account Applications

1. Gửi một đơn, lưu ID rồi đánh dấu đơn cần xem xét:

```bash
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "123 Enterprise Street" }' out.json && cat out.json; echo
ID=$(jq -r .id out.json); echo $ID
aws lambda invoke --function-name sfn-workshop-FlagApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\", \"flagType\": \"REVIEW\" }" out.json && cat out.json; echo
```

   - (1) Đơn mới có trạng thái `SUBMITTED` và ID dạng `application_<uuid>`.
   - (2) ID được lưu vào biến `ID`.
   - (3) Đơn chuyển sang `FLAGGED_FOR_REVIEW`.

   ![Gửi và đánh dấu](/images/2-preparation/2.3-submit-flag.png)

2. Liệt kê đơn cần xem xét rồi duyệt đơn:

```bash
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "FLAGGED_FOR_REVIEW" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-ApproveApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\" }" out.json && cat out.json; echo
```

   - (1) `FindApplications` trả về đơn đang được đánh dấu.
   - (2) Đơn chuyển sang `APPROVED`.

   ![Tìm và duyệt](/images/2-preparation/2.3-find-approve.png)

#### Data Checking

3. Kiểm tra hai tên và hai địa chỉ:

```bash
for p in '{"command": "CHECK_NAME", "data": { "name": "Spock" } }' \
         '{"command": "CHECK_NAME", "data": { "name": "evil Spock" } }' \
         '{"command": "CHECK_ADDRESS", "data": { "address": "123 Street" } }' \
         '{"command": "CHECK_ADDRESS", "data": { "address": "DoesntMatchAddressPattern" } }'; do
  echo "$p"; aws lambda invoke --function-name sfn-workshop-DataChecking --cli-binary-format raw-in-base64-out --payload "$p" out.json >/dev/null && cat out.json; echo
done
```

   - (1) `Spock` là tên hợp lệ: `flagged` là `false`.
   - (2) Tên có chữ `evil` bị đánh dấu.
   - (3) `123 Street` khớp mẫu địa chỉ (số và chữ).
   - (4) `DoesntMatchAddressPattern` bị đánh dấu.

   ![Data checking](/images/2-preparation/2.3-data-checking.png)

Các service đã chạy, nhưng chưa có gì nối chúng lại. Vẫn phải có người gọi bước kiểm tra, đọc kết quả rồi quyết định đánh dấu hay duyệt đơn. Ở phần tiếp theo, Step Functions sẽ đảm nhận việc này.
