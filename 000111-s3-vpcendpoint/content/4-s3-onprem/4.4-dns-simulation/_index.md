---
title : "On-premises DNS simulation"
weight : 4
chapter : false
pre : " <b> 4.4 </b> "
---


Passing `--endpoint-url` to every command is not practical for real applications. AWS recommends using **DNS**: on-premises resolvers forward queries for `s3.<region>.amazonaws.com` to a Route 53 Resolver **inbound endpoint**, which answers from a private hosted zone that points to the interface endpoint.

In this lab, the outbound endpoint in VPC On-prem plays the role of the corporate DNS server that forwards queries.

#### Create alias records for the interface endpoint

1. Open **Route 53 → Hosted zones** and open the private zone **s3.&lt;region&gt;.amazonaws.com** (created by the DNS stack). Choose **Create record**.

![hosted zone](/images/4-s3-onprem/4.4-hosted-zone.png)

2. Record 1 – leave **Record name** empty (zone apex), **Record type** A:
   + Turn on **Alias**
   + **Route traffic to**: **Alias to VPC endpoint**, choose your Region
   + Choose the Regional DNS name of **s3-interface-endpoint** (`*.vpce-...s3.<region>.vpce.amazonaws.com`)

![record 1](/images/4-s3-onprem/4.4-record-1.png)

3. Choose **Add another record**. Record 2 – **Record name** `*`, type A, **Alias** to the same VPC endpoint. The wildcard covers virtual-hosted bucket names such as `<bucket>.s3.<region>.amazonaws.com`.

![record 2](/images/4-s3-onprem/4.4-record-2.png)

4. Choose **Create records**. The zone now has the two alias records.

![records](/images/4-s3-onprem/4.4-records-done.png)

#### Create a Resolver forwarding rule

1. Open **Route 53 → Resolver → Inbound endpoints** and open **fcj-inbound-cloud**. Note the two **IP addresses** (one per subnet).

![inbound ips](/images/4-s3-onprem/4.4-inbound-ips.png)

2. Go to **Resolver → Rules → Create rule**:
   + **Name**: `fcj-s3-forward-rule`
   + **Rule type**: **Forward**
   + **Domain name**: `s3.<region>.amazonaws.com`
   + **VPCs that use this rule**: **VPC On-prem**

![rule 1](/images/4-s3-onprem/4.4-rule-1.png)

   + **Outbound endpoint**: **fcj-outbound-onprem**
   + **Target IP addresses**: the two inbound endpoint IPs, port **53**

![rule 2](/images/4-s3-onprem/4.4-rule-2.png)

3. Choose **Submit**. The rule status becomes **Complete**.

![rule done](/images/4-s3-onprem/4.4-rule-done.png)

#### Test

Back in the **Test-Interface-Endpoint** session, run:

```bash
getent hosts <your-bucket-name>.s3.<region>.amazonaws.com
aws s3 ls s3://<your-bucket-name> --region <your-region>
```

The **standard** S3 hostname now resolves to the private IPs of the interface endpoint, and the AWS CLI works **without** `--endpoint-url`.

![test](/images/4-s3-onprem/4.4-test.png)
