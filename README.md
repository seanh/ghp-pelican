Use GitHub Actions to Deploy a Pelican Site to GitHub Pages
===========================================================

This repo contains a [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for automatically publishing a [Pelican](https://getpelican.com/) site to [GitHub Pages](https://pages.github.com/) using [GitHub Actions](https://github.com/features/actions).

There's a [demo repo](https://github.com/seanh/pelican-github-pages-demo/) showing the workflow in use.

Usage
-----

1. Create a Pelican site in your GitHub repo, for example by following [Pelican's quickstart](https://docs.getpelican.com/en/latest/quickstart.html).

   * Your repo needs to contain at least a `publishconf.py` file and a `content/` directory.

   * **You don't need to set `SITEURL`** in your `publishconf.py` file: the workflow will set it for you.

   * **You don't need to commit the `output/` directory to git**: the workflow will build the output files for you on GitHub Actions.
   
2. Enable GitHub Pages in your repo: go to **Settings &rarr; Pages** and choose **GitHub Actions** as the **Source**:

   ![GitHub Pages deployment settings](/settings.png)

3. Add a `.github/workflows/pelican.yml` file to your repo with these contents:

   ```yaml
   name: Deploy Pelican site to GitHub Pages
   on:
     push:
       branches: ["main"]
     workflow_dispatch:
   jobs:
     Deploy:
       uses: seanh/pelican-github-pages/.github/workflows/pelican.yml@main
       permissions:
         contents: read
         pages: write
         id-token: write
   ```

   You may want to replace the `@main` with the ID of a specific commit in this repo in order to pin the version of the reusable workflow that you're using. For example: `uses: seanh/pelican-github-pages/.github/workflows/pelican.yml@0cc99c14466712c5bc2406660cd988541afcd767`.

   Alternatively you can copy-paste the jobs from this repo's `pelican.yml` workflow into your own repo's workflow, instead of using it as a reusable workflow.

5. Go to the **Actions** tab in your repo (`https://github.com/YOUR_USER/YOUR_REPO/actions`) and you should see a **Deploy Pelican Site to GitHub Pages** action running.

6. Once the action completes you should see your Pelican site deployed at your repo's GitHub Pages URL: `https://YOUR_USER.github.io.` for a user or organization site, or `https://YOUR_USER.github.io/YOUR_REPO` for a project site.

Explanation
-----------

GitHub has two ways to deploy a repo's GitHub Pages site:

1. You can **Deploy from a branch** (also called the "Classic Pages experience"). This uses [Jekyll](https://jekyllrb.com/) to build your site. If you want to use a different static site generator you have to run the generator locally and push the generated output files to the repo's `gh-pages` branch. This is what the [Publishing to GitHub section](https://docs.getpelican.com/en/latest/tips.html#publishing-to-github) in Pelican's docs uses.
2. Or you can use **GitHub Actions** to deploy your site, which enables deeper integration of non-Jekyll site generators: you just push your Pelican source content files to your repo's `main` branch and a GitHub Actions workflow runs automatically on each push, builds your site with Pelican, and deploys the output files to GitHub Pages. You can even edit your site's content using GitHub's web interface and the changes will be automatically deployed. This is what this repo uses.

See the [GitHub Pages docs about publishing sources](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) for more about the two ways of publishing a GitHub Pages site.
