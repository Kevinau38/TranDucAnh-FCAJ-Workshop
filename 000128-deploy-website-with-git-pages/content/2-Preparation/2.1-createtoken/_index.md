---
title : "Install Git and Hugo"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


You need a GitHub account, **Git** and **Hugo extended** on your computer.

1. Install Git.
   - Windows: install [Git for Windows](https://git-scm.com/downloads/win). It includes **Git Credential Manager**, so the first `git push` opens a browser window where you sign in to GitHub. You do not need to create a personal access token.
   - macOS: `brew install git`. Linux: install `git` from your package manager.
2. Install Hugo **extended**, version 0.158.0 or later (the Ananke theme used in this lab needs it). See [Installation](https://gohugo.io/installation/) for every option.
   ```bash
   # Windows
   winget install Hugo.Hugo.Extended
   # macOS
   brew install hugo
   ```
3. Open a new terminal and check both tools:
   ```bash
   git --version
   hugo version
   ```
   `hugo version` must show `+extended` and a version of at least `v0.158.0`.
4. Tell Git who you are. These values appear in your commits:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "<username>@users.noreply.github.com"
   ```

{{% notice warning %}}
Never paste a personal access token into a clone URL, a script or a chat. Anyone who sees it can act as you on GitHub. If you must use a token, let Git Credential Manager store it.
{{% /notice %}}
