---
title : "Push the built site"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


1. Rebuild the site with the project address. The path `/workshop1/` at the end is required, otherwise links and CSS point to the root of your user site.
   ```bash
   # in the quickstart folder, rebuild for the project path
   hugo --baseURL https://<username>.github.io/workshop1/

   cd ..
   git clone https://github.com/<username>/workshop1.git
   cp -r quickstart/public/* workshop1/

   cd workshop1
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git push -u origin main
   ```
2. Refresh the `workshop1` repository on GitHub and check that it contains `index.html` and the other built files.

{{% notice info %}}
Run `hugo` again for every repository. The `public/` folder from section 3 was built for a different address and must not be reused.
{{% /notice %}}
