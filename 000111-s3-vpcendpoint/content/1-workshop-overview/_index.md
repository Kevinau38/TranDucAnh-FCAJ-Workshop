---
title : "Workshop overview"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### VPC endpoints

A VPC endpoint is a virtual device that lets resources in your VPC (and networks connected to it) reach supported AWS services **privately**. Traffic between the endpoint and the service stays on the AWS network.

| | Gateway endpoint | Interface endpoint |
|---|---|---|
| Services | Amazon S3, DynamoDB | Most AWS services, including S3 |
| How it works | Prefix-list route in a route table | ENIs with private IPs in your subnets |
| Reachable from on-premises | No | Yes (VPN, Direct Connect, peering) |
| DNS | Uses the public S3 names | Endpoint-specific DNS names, or private DNS / Route 53 |
| Cost | No hourly charge | Per hour per AZ + per GB |

#### What you will do

1. Deploy the base network with AWS CloudFormation.
2. Create an **S3 Gateway endpoint** and upload a file from a private instance that has no Internet access.
3. Connect the simulated on-premises network, create an **S3 Interface endpoint**, and upload a file over the private link.
4. Use **Route 53 Resolver** (inbound + outbound endpoints and a forwarding rule) so on-premises clients resolve the normal S3 hostname to the endpoint's private IPs.
5. Attach a **VPC endpoint policy** that only allows one bucket.
6. Clean up every resource.

The lab takes about **60–90 minutes**.
