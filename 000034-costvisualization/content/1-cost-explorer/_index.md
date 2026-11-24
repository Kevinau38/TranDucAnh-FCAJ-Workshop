---
title : "Cost Explorer basics"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Open Cost Explorer

1. Open the [Billing and Cost Management console](https://console.aws.amazon.com/costmanagement/home#/cost-explorer) and choose **Cost Explorer**:
    - (1) **Cost Explorer** in the navigation pane opens a new cost and usage report.
    - (2) **Cost and usage overview**: the total cost, the average cost per period, and the number of services in the report.
    - (3) **Report parameters**: by default, the last 6 months, **Monthly** granularity, grouped by **Service**.

    ![Cost Explorer](/images/1/1-1.png)

{{% notice info %}}
The first time Cost Explorer is enabled in an account, AWS prepares the data of the current month and the last 12 months, which can take up to 24 hours. After that, Cost Explorer refreshes the cost data at least once every 24 hours.
{{% /notice %}}

2. Scroll down to the **Cost and usage graph**:
    - (1) One stacked bar per month: each color is a service.
    - (2) The legend shows the top services; the smaller ones are grouped as **Others**.

    ![Cost and usage graph](/images/1/1-2.png)

3. Scroll down to the **Cost and usage breakdown**:
    - (1) The cost of each service per month, and the total of the period. The first row is the total of the account.
    - (2) **Download as CSV** exports the table, for example to share it or to build your own charts.

    ![Cost and usage breakdown](/images/1/1-3.png)

#### Change the granularity and the chart type

4. In **Report parameters**:
    - (1) Set **Granularity** to **Daily**. Daily data shows spikes that a monthly view hides: here, a service that is billed once a month on the same day.
    - (2) Switch the chart between **bar**, **line**, and **stacked bar**. The line chart makes trends easier to compare.

    ![Daily, line chart](/images/1/1-4.png)

{{% notice tip %}}
**Hourly** granularity and **resource-level** data (the cost of each EC2 instance, for example) are optional features that you turn on in **Cost Management preferences**. They cover the last 14 days and are billed per usage record, so turn them on only if you need them.
{{% /notice %}}
