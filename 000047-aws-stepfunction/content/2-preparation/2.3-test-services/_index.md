---
title : "Test the sample services"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Call the functions directly with `aws lambda invoke`. AWS CLI v2 expects the payload in base64 by default, so every command adds `--cli-binary-format raw-in-base64-out` to send plain JSON.

#### Account Applications

1. Submit an application, save its ID, then flag it for review:

```bash
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "123 Enterprise Street" }' out.json && cat out.json; echo
ID=$(jq -r .id out.json); echo $ID
aws lambda invoke --function-name sfn-workshop-FlagApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\", \"flagType\": \"REVIEW\" }" out.json && cat out.json; echo
```

   - (1) The new application has the state `SUBMITTED` and an ID such as `application_<uuid>`.
   - (2) The ID is saved in the variable `ID`.
   - (3) The application is now `FLAGGED_FOR_REVIEW`.

   ![Submit and flag](/images/2-preparation/2.3-submit-flag.png)

2. List the flagged applications, then approve the application:

```bash
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "FLAGGED_FOR_REVIEW" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-ApproveApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\" }" out.json && cat out.json; echo
```

   - (1) `FindApplications` returns the flagged application.
   - (2) The application is now `APPROVED`.

   ![Find and approve](/images/2-preparation/2.3-find-approve.png)

#### Data Checking

3. Check two names and two addresses:

```bash
for p in '{"command": "CHECK_NAME", "data": { "name": "Spock" } }' \
         '{"command": "CHECK_NAME", "data": { "name": "evil Spock" } }' \
         '{"command": "CHECK_ADDRESS", "data": { "address": "123 Street" } }' \
         '{"command": "CHECK_ADDRESS", "data": { "address": "DoesntMatchAddressPattern" } }'; do
  echo "$p"; aws lambda invoke --function-name sfn-workshop-DataChecking --cli-binary-format raw-in-base64-out --payload "$p" out.json >/dev/null && cat out.json; echo
done
```

   - (1) `Spock` is a valid name: `flagged` is `false`.
   - (2) A name containing `evil` is flagged.
   - (3) `123 Street` matches the address pattern (number and word).
   - (4) `DoesntMatchAddressPattern` is flagged.

   ![Data checking](/images/2-preparation/2.3-data-checking.png)

The services work, but nothing connects them yet. Someone still has to call the data check, read the result and decide whether to flag or approve the application. In the next part, Step Functions takes over this job.
