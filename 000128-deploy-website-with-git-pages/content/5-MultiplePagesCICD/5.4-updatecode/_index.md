---
title : "Update the source code"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---


1. The commit in step 5.2 was made on GitHub, so first bring it to your computer. In the `quickstart` folder, run:
   ```bash
   git pull
   ```
2. Open `content/posts/my-first-post.md` and add a new line at the end, for example:
   ```markdown
   This line was added later. GitHub Actions rebuilt and redeployed the site automatically after `git push`.
   ```
3. Commit and push the change:
   ```bash
   git add .
   git commit -m "Edit Source Code"
   git push
   ```
You only pushed Markdown. You did not run `hugo` and did not touch any HTML.
