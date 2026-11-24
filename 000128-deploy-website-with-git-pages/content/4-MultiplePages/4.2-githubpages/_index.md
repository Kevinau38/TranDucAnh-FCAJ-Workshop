---
title : "Add the Static HTML workflow"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


1. In the `workshop1` repository, choose **Settings**, then **Pages**.
2. Open the **Source** list and choose **GitHub Actions** (1).

![Choose GitHub Actions](/images/4-MultiplePages/4.2-source-dropdown.png)

3. GitHub shows **GitHub Actions** as the source (1) and suggests workflows. On the **Static HTML** card, choose **Configure** (2).

![Static HTML, Configure](/images/4-MultiplePages/4.2-static-configure.png)

4. GitHub opens an editor with the file `.github/workflows/static.yml` (1). The workflow runs on every push to `main`, uploads the whole repository as the site and deploys it to Pages. You do not need to change it. Choose **Commit changes...** (2).

![static.yml](/images/4-MultiplePages/4.2-static-editor.png)

5. Keep the commit message **Create static.yml** (1), keep **Commit directly to the main branch** and choose **Commit changes** (2).

![Commit static.yml](/images/4-MultiplePages/4.2-commit-dialog.png)

The commit is a push to `main`, so the workflow starts at once.
