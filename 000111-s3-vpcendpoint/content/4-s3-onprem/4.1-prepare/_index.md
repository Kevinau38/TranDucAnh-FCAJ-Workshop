---
title : "Prepare the environment"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


#### Deploy the DNS stack

The second template creates the Route 53 building blocks used in section 4.4:

+ A **private hosted zone** named `s3.<region>.amazonaws.com`, associated with VPC Cloud.
+ A Route 53 Resolver **inbound endpoint** in VPC Cloud – it answers DNS queries that come from outside the VPC.
+ A Route 53 Resolver **outbound endpoint** in VPC On-prem – it forwards selected DNS queries to another resolver.

1. In the CloudFormation console choose **Create stack → With new resources (standard)**, upload **fcj-s3vpce-dns.yaml**, name the stack **fcj-s3vpce-dns** and create it.
2. The stack takes 3–5 minutes. You can continue with the next steps while it is being created.

![dns stack](/images/4-s3-onprem/4.1-dns-stack.png)

{{% notice info %}}
The DNS template imports the VPC, subnet and security-group IDs exported by **fcj-s3vpce-base**, so you do not need to enter any parameters.
{{% /notice %}}

#### Route the on-premises network to VPC Cloud

The base stack already created the **VPC peering** connection (*Hybrid link On-prem to Cloud*) and the route from VPC Cloud back to 172.16.0.0/16. You now finish the "hybrid link" by adding the route on the on-premises side.

1. Open **VPC console → Route tables**, select **RT On-prem**, open the **Routes** tab and choose **Edit routes**.

![rt onprem](/images/4-s3-onprem/4.1-rt-onprem.png)

2. Choose **Add route**:
   + **Destination**: `10.10.0.0/16` (VPC Cloud)
   + **Target**: **Peering Connection** → `pcx-...` (Hybrid link On-prem to Cloud)
3. Choose **Save changes**.

![add route](/images/4-s3-onprem/4.1-add-route.png)

4. The route `10.10.0.0/16 → pcx-...` is **Active**.

![route done](/images/4-s3-onprem/4.1-route-done.png)
