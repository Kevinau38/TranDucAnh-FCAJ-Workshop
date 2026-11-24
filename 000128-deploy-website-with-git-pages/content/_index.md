---
title : "Deploy Hugo Website with GitHub Pages"
weight : 1
chapter : false
---

# Deploy Hugo Website with GitHub Pages

#### Overview

**GitHub Pages** hosts static websites for free, straight from a GitHub repository. **Hugo** is a fast static site generator that turns Markdown files into a complete website; the AWS Study Group workshops you are reading are built with Hugo.

In this lab you build a small Hugo site and publish it to GitHub Pages in three different ways, from the simplest to a full CI/CD pipeline:

1. A **user site** at `https://<username>.github.io/`, published from a branch.
2. A **project site** at `https://<username>.github.io/workshop1/`, published by the **Static HTML** GitHub Actions workflow.
3. A **project site** at `https://<username>.github.io/workshop2/`, where the **Hugo** GitHub Actions workflow builds and deploys the site every time you push source code.

#### Architecture

![Lab architecture](/images/architecture.png)

{{% notice info %}}
This is an updated version of the workshop. You no longer put a personal access token in the clone URL: Git Credential Manager signs you in to GitHub through the browser. The sample site is now created with the official Hugo quick start and the Ananke theme. Everything in this lab is free, and you delete the three repositories at the end.
{{% /notice %}}
