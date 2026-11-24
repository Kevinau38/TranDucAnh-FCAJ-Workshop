---
title : "Push the source code"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---


1. The `quickstart` folder is already a Git repository, with the Ananke theme added as a **submodule**. Push it to `workshop2`:
   ```bash
   # in the quickstart folder (already a Git repository from step 2.3)
   git add .
   git commit -m "Add Source Code"
   git branch -M main
   git remote add origin https://github.com/<username>/workshop2.git
   git push -u origin main
   ```
2. Refresh the `workshop2` repository. It contains the Hugo source: `archetypes`, `content/posts`, `themes`, `.gitignore`, `.gitmodules` and `hugo.toml`. There is no `public/` folder, because `.gitignore` excludes it.

![Hugo source in workshop2](/images/5-MultiplePagesCICD/5.1-repo-source.png)
