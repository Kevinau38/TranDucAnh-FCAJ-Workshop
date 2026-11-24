---
title : "Push the built site"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Build the site with the address of the user site, clone the empty repository next to the `quickstart` folder, copy the files from `public/` into it and push:
   ```bash
   # in the quickstart folder
   hugo --baseURL https://<username>.github.io/

   # in a separate folder, clone the empty repository
   cd ..
   git clone https://github.com/<username>/<username>.github.io.git
   cp -r quickstart/public/* <username>.github.io/

   cd <username>.github.io
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git push -u origin main
   ```
   On Windows PowerShell, use `Copy-Item -Recurse quickstart\public\* <username>.github.io\` instead of `cp`.
2. The first `git push` opens a browser window from Git Credential Manager. Sign in to GitHub and authorize it.
3. Refresh the repository page on GitHub. It now contains the built website: `index.html`, `404.html`, `sitemap.xml` and the `posts`, `tags` and `categories` folders.

![Built files in the repository](/images/3-SinglePage/3.1-repo-pushed.png)
