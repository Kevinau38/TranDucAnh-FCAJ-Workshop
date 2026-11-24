---
title : "Integrate with the Well-Architected Tool"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Service Screener can write its results into a workload of the **AWS Well-Architected Tool**: it answers the review questions it can check automatically, marks the risks, and saves a milestone. You then finish the review in the console.

1. Run the scan again with the `WA` option. `reportName` is the workload name (an existing name is updated). `newMileStone: 1` saves a new milestone on every run.

```bash
screener --regions ap-southeast-1 --services s3,ec2 \
  --tags Project=fcj-screener --disable-custom-pages 1 --beta 1 \
  --others '{"WA": {"region": "ap-southeast-1", "reportName": "fcj-screener", "newMileStone": 1}}'
```

{{% notice note %}}
If Service Screener asks `Resume from where it left off? [y/n]`, answer `n`.
{{% /notice %}}

2. Open the [Well-Architected Tool](https://console.aws.amazon.com/wellarchitected/home?region=ap-southeast-1) in Singapore and choose **Workloads**. Search for `fcj`. The workload `fcj-screener` shows the questions answered and the high and medium risks.

![Workloads](/images/5/1-workloads.png)

3. Open the workload. The **Overview** shows how many of the 57 questions Service Screener answered and how many risks it found.

![Workload overview](/images/5/2-overview.png)

4. The **Milestones** tab shows the milestone that Service Screener saved, named `SS-<date>-<number>`. Milestones let you compare reviews over time.

![Milestones](/images/5/3-milestones.png)

5. In the left menu, choose **AWS Well-Architected Framework**, then the **Improvement plan** tab. Each item is a review question with its risk level and recommended improvements.

![Improvement plan](/images/5/4-improvement.png)

{{% notice tip %}}
The download from chapter 4 also contains a PDF of the Well-Architected report (`WAFR_<name>_....pdf`) when you use this option. AWS Partners can add a Migration Evaluation ID with `--others '{"mpe": {"id": "..."}}'`, alone or together with `WA`.
{{% /notice %}}
