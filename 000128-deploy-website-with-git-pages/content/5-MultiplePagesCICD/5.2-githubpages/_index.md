---
title : "Add the Hugo workflow"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---


1. In `workshop2`, choose **Settings**, then **Pages**, and set **Source** to **GitHub Actions** (1). The Hugo workflow is not on the suggested cards, so choose **browse all workflows** (2).

![Browse all workflows](/images/5-MultiplePagesCICD/5.2-actions-source.png)

2. In the search box, type `hugo` (1). On the **Hugo** card, choose **Configure** (2).

![Hugo workflow](/images/5-MultiplePagesCICD/5.2-search-hugo.png)

3. GitHub opens `.github/workflows/hugo.yml` (1).

![hugo.yml](/images/5-MultiplePagesCICD/5.2-hugo-editor.png)

4. Scroll to the `build` job and change `HUGO_VERSION` to a version that the Ananke theme supports, for example `0.170.0` or the version that `hugo version` showed on your computer (1). The default value in the template is older and the build fails with it. Then choose **Commit changes...** (2) and commit directly to `main`.

![HUGO_VERSION](/images/5-MultiplePagesCICD/5.2-hugo-version.png)

The important parts of the workflow are:
```yaml
    env:
      HUGO_VERSION: 0.170.0
...
      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
```
The workflow checks out the repository with its submodules, so the theme is available, and it takes the baseURL from the Pages settings, so you never have to type the project path.
