---
title : "Create the Hugo site"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---


You create the sample site with the official [Hugo quick start](https://gohugo.io/getting-started/quick-start/) and the **Ananke** theme.

1. In a working folder, run:
   ```bash
   hugo new project quickstart
   cd quickstart
   git init
   git submodule add https://github.com/gohugo-ananke/ananke themes/ananke
   echo "theme = 'ananke'" >> hugo.toml
   hugo new content content/posts/my-first-post.md
   ```
2. Open `hugo.toml` and change the title:
   ```toml
   baseURL = 'https://example.org/'
   locale = 'en-us'
   title = 'FCJ Hugo on GitHub Pages'
   theme = 'ananke'
   ```
3. Open `content/posts/my-first-post.md`, set `draft = false` and add a line of text:
   ```markdown
   +++
   date = '2026-11-17T13:14:46+07:00'
   draft = false
   title = 'My First Post'
   +++
   This site is built with **Hugo** and published with **GitHub Pages**.
   ```
4. Create a `.gitignore` file in the `quickstart` folder so build output is not committed with the source:
   ```text
   public/
   resources/_gen/
   .hugo_build.lock
   ```
5. Preview the site locally:
   ```bash
   hugo server
   ```
   Open `http://localhost:1313/` in your browser, then press **Ctrl+C** in the terminal to stop the server.

{{% notice info %}}
`hugo server` builds the site in memory. The `hugo` command (without `server`) writes the finished site into the `public/` folder; you use it in sections 3 and 4.
{{% /notice %}}
