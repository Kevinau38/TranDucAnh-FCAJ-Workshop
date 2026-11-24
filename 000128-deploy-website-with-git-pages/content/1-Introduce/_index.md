---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### GitHub Pages

GitHub Pages serves the HTML, CSS and JavaScript files of a repository as a website over HTTPS. Public repositories on a free GitHub account can use it at no cost. There are two kinds of site:

| Site type | Repository name | Address |
|---|---|---|
| User (or organization) site | exactly `<username>.github.io` | `https://<username>.github.io/` |
| Project site | any name, for example `workshop1` | `https://<username>.github.io/workshop1/` |

Each account has one user site and can have many project sites.

#### Publishing source

In **Settings, Pages** of a repository you choose how the site is published:

- **Deploy from a branch**: GitHub publishes the files that are already in a branch and folder (for example `main` and `/ (root)`). You must commit the finished HTML files.
- **GitHub Actions**: a workflow in `.github/workflows/` produces the site and deploys it. The workflow can simply upload files you built yourself (**Static HTML**), or build the site from source on every push (**Hugo**).

#### Hugo and baseURL

Hugo writes the finished website into the `public/` folder. Links inside the site depend on the **baseURL** setting, so a site that will live at `https://<username>.github.io/workshop1/` must be built with that baseURL, otherwise CSS and links point to the wrong place. In section 5 the Hugo workflow sets baseURL for you from the Pages address.

#### What you will do

| Section | Repository | Publishing source | What is pushed |
|---|---|---|---|
| 3 | `<username>.github.io` | Deploy from a branch | built `public/` files |
| 4 | `workshop1` | GitHub Actions, Static HTML | built `public/` files |
| 5 | `workshop2` | GitHub Actions, Hugo | Hugo source code |
