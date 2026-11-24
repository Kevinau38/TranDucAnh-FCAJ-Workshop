---
title : "Tích hợp với Well-Architected Tool"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Service Screener có thể ghi kết quả vào một workload trong **AWS Well-Architected Tool**: nó trả lời các câu hỏi review mà nó kiểm tra được tự động, đánh dấu rủi ro và lưu một milestone. Sau đó bạn hoàn thiện bài review trên console.

1. Chạy quét lại với tuỳ chọn `WA`. `reportName` là tên workload (nếu tên đã tồn tại thì workload đó được cập nhật). `newMileStone: 1` lưu một milestone mới sau mỗi lần chạy.

```bash
screener --regions ap-southeast-1 --services s3,ec2 \
  --tags Project=fcj-screener --disable-custom-pages 1 --beta 1 \
  --others '{"WA": {"region": "ap-southeast-1", "reportName": "fcj-screener", "newMileStone": 1}}'
```

{{% notice note %}}
Nếu Service Screener hỏi `Resume from where it left off? [y/n]`, trả lời `n`.
{{% /notice %}}

2. Mở [Well-Architected Tool](https://console.aws.amazon.com/wellarchitected/home?region=ap-southeast-1) ở Singapore và chọn **Workloads**. Tìm `fcj`. Workload `fcj-screener` hiển thị số câu hỏi đã trả lời và số rủi ro mức cao, mức trung bình.

![Workloads](/images/5/1-workloads.png)

3. Mở workload. Tab **Overview** cho biết Service Screener đã trả lời bao nhiêu trong 57 câu hỏi và tìm thấy bao nhiêu rủi ro.

![Tổng quan workload](/images/5/2-overview.png)

4. Tab **Milestones** hiển thị milestone mà Service Screener đã lưu, tên dạng `SS-<ngày>-<số>`. Milestone giúp bạn so sánh các lần review theo thời gian.

![Milestones](/images/5/3-milestones.png)

5. Ở menu bên trái, chọn **AWS Well-Architected Framework**, rồi chọn tab **Improvement plan**. Mỗi mục là một câu hỏi review kèm mức rủi ro và các đề xuất cải thiện.

![Improvement plan](/images/5/4-improvement.png)

{{% notice tip %}}
Khi dùng tuỳ chọn này, file tải về ở chương 4 còn có bản PDF của báo cáo Well-Architected (`WAFR_<tên>_....pdf`). AWS Partner có thể thêm Migration Evaluation ID bằng `--others '{"mpe": {"id": "..."}}'`, dùng riêng hoặc cùng với `WA`.
{{% /notice %}}
