---
title : "Triển khai project site bằng workflow Hugo (CI/CD)"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Ở phần 3 và 4, bạn tự build website và push HTML. Ở phần này, bạn chỉ push **source code Hugo**. Workflow **Hugo** cài Hugo trên runner của GitHub, build website với baseURL đúng và deploy. Mỗi lần `git push` sau đó, website được build và deploy lại tự động, đây chính là một pipeline CI/CD nhỏ.
