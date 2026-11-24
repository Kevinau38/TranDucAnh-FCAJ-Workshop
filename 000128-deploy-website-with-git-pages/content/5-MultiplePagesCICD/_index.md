---
title : "Deploy a project site with the Hugo workflow (CI/CD)"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

In sections 3 and 4 you built the site yourself and pushed HTML. Here you push only the **Hugo source code**. The **Hugo** workflow installs Hugo on a GitHub runner, builds the site with the right baseURL and deploys it. Every later `git push` rebuilds and redeploys the site automatically, which is a small CI/CD pipeline.
