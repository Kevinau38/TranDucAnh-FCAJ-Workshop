---
title : "Create a container service"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

1. Open the [Lightsail console](https://lightsail.aws.amazon.com/), choose **Containers** in the left menu, then **Create container service**.

2. Check that the location is **Singapore, all zones (ap-southeast-1)**. If not, choose **Change AWS Region**.

![Create a container service](/images/1/1-location.png)

3. Under **Choose the power**, choose **Nano** (512 MB RAM, 0.25 vCPU, USD 7 per node per month). Keep the scale at **×1**. The cost of the service is power × scale.

![Power and scale](/images/1/2-power.png)

{{% notice info %}}
Each node runs a copy of every container in your deployment. With a scale of 2 or more, Lightsail spreads the traffic across the nodes, so the application keeps running if one node fails.
{{% /notice %}}

4. Do not set up a deployment now; you do it in the next chapter. Under **Identify your service**, enter `fcj-container`. The name becomes part of the public domain.

![Service name](/images/1/3-name.png)

5. Check the **Summary** and choose **Create container service**.

![Create](/images/1/4-create.png)

6. The service shows **Pending** while Lightsail creates its resources. It becomes **Ready** in a few minutes. The **Public domain** is already shown.

![Service pending](/images/1/5-pending.png)
