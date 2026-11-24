---
title : "Tạo snapshot"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

**Snapshot** là bản sao tại một thời điểm của các ổ đĩa của instance. Bạn có thể dùng nó để khôi phục instance khi thao tác nhầm, tạo bản sao của instance, hoặc chuyển sang gói lớn hơn (chương tiếp theo). Snapshot được tính phí theo dung lượng lưu trữ; các snapshot sau chỉ lưu phần dữ liệu thay đổi.

1. Mở instance `fcj-wordpress` và chọn tab **Snapshots**.

2. Ở mục **Manual snapshots**, chọn **Create snapshot**. Nhập `fcj-wordpress-snap-1` và chọn **Create**.

![Tạo snapshot](/images/5/1-create-snapshot.png)

3. Snapshot hiển thị **Snapshotting...** trong vài phút. Instance vẫn chạy bình thường trong lúc chụp snapshot.

![Snapshot đang tạo](/images/5/2-snapshotting.png)

{{% notice info %}}
**Automatic snapshots** chụp snapshot mỗi ngày vào giờ bạn chọn và giữ 7 bản gần nhất. Hãy bật tính năng này cho instance production. Nội dung WordPress nằm trong managed database, vốn được Lightsail tự động sao lưu; database có snapshot riêng ở tab **Snapshots & restore**.
{{% /notice %}}
