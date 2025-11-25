> [!NOTE]
> This workflow got merged into the Pelican project, you should [use the official version of the workflow](https://docs.getpelican.com/en/latest/tips.html#publishing-to-github-pages-using-a-custom-github-actions-workflow) instead.
> I sometimes still use this repo to test new features ahead of merging
> them into Pelican.

Use GitHub Actions to Deploy a [Pelican](https://getpelican.com/) Site to GitHub Pages
--------------------------------------------------------------------------------------

This repo contains a [reusable GitHub Actions workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for deploying a [Pelican](https://getpelican.com/) site to [GitHub Pages](https://pages.github.com/): push the source files of your site to a branch of your GitHub repo and GitHub Actions runs Pelican to build your output files and deploys them to your GitHub Pages site. No need to run Pelican locally or commit your output files to git. You can even edit your site's source files using GitHub's web interface and the updated output files will be built on GitHub Actions and deployed. There's a [demo repo](https://github.com/seanh/pelican-github-pages-demo/) showing the workflow in use.

Explanation
-----------

GitHub has two ways to deploy a repo's GitHub Pages site:

1. You can **Deploy from a branch** (also called the "Classic Pages experience"). This uses the [Jekyll](https://jekyllrb.com/) static site generator. If you want to use a different static site generator you have to run your generator locally and push the generated output files to a branch. Or you can write a GitHub Actions workflow that runs your generator and pushes the output files to a branch for you.
2. Alternatively, you can use **GitHub Actions** to deploy your site directly, which enables deeper integration of non-Jekyll site generators. This is the recommended way to use non-Jekyll site generators with GitHub Pages. You just push your Pelican source content files to a branch of your repo (such as the `main` branch) and a GitHub Actions workflow runs automatically on each push, builds your site with Pelican, and deploys the output files to GitHub Pages. No need to run Pelican locally or commit your output files to Git. This is what the GitHub Actions workflow in this repo does.

See the [GitHub Pages docs about publishing sources](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) for more about the two ways of publishing a GitHub Pages site.

Usage
-----

1. Enable GitHub Pages in your repo: go to **Settings &rarr; Pages** and choose **GitHub Actions** for the **Source** setting:

   ![GitHub Pages deployment settings](/settings.png)

2. Commit a `.github/workflows/github_pages.yml` file to your repo with these contents:

   ```yaml
   name: Deploy to GitHub Pages
   on:
     push:
       branches: ["main"]
     workflow_dispatch:
   jobs:
     deploy:
       uses: "seanh/ghp-pelican/.github/workflows/github_pages.yml@main"
       permissions:
         contents: "read"
         pages: "write"
         id-token: "write"
       with:
         settings: "publishconf.py"
   ```

   You may want to replace the `@main` with the ID of a specific commit in this repo in order to pin the version of the reusable workflow that you're using. For example: `uses: seanh/ghp-pelican/.github/workflows/github_pages.yml@2925832191ca1cdba8852582d222ebd723358df2`. If you do this you might want to get Dependabot to send you automated pull requests to update that commit ID whenever new versions of this workflow are published, like so:

   ```yaml
   # .github/dependabot.yml
   version: 2
   updates:
     - package-ecosystem: "github-actions"
       directory: "/"
       schedule:
         interval: "monthly"
   ```

   See [GitHub's docs about using Dependabot to keep your actions up to date](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot).

3. Go to the **Actions** tab in your repo (`https://github.com/YOUR_USER/YOUR_REPO/actions`) and you should see a **Deploy to GitHub Pages** action running.

4. Once the action completes you should see your Pelican site deployed at your repo's GitHub Pages URL: `https://YOUR_USER.github.io.` for a user or organization site, or `https://YOUR_USER.github.io/YOUR_REPO` for a project site.

Notes:

* **You don't need to set `SITEURL` or `FEED_DOMAIN`** in your Pelican config file: the workflow will set them correctly for you.

* **You don't need to commit the output directory to git**: the workflow will run Pelican to build the output directory for you on GitHub Actions.

A number of optional inputs can be added to the `with:` block when calling the workflow, for example:

```yaml
with:
  settings: "publishconf.py"
  requirements: "pelican[markdown] typogrify"
  theme: "https://github.com/seanh/sidecar.git"
  python: "3.12"
```

Here's the complete list of workflow inputs:

| Name | Required | Description | Type | Default |
| --- | --- | --- | --- | --- |
| `settings` | Yes | The path to your Pelican settings file (the `pelican` command's `--settings` option), for example: `"publishconf.py"` | string |  |
| `requirements` | No | The Python requirements to install, for example to enable markdown and typogrify use: `"pelican[markdown] typogrify"` or if you have a requirements file: `"-r requirements.txt"` | string | `"pelican"` |
| `output-path` | No | Where to output the generated files (the `pelican` command's `--output` option) | string | `"output/"` |
| `theme` | No | The GitHub repo URL of a custom theme to use, for example: `"https://github.com/seanh/sidecar.git"` | string | |
| `python` | No | The version of Python to use to build the site, for example: `"3.12"` (to use the most recent version of Python 3.12, this is faster) or `"3.12.1"` (to use an exact version, slower) | string | `"3.12"` |
| `siteurl` | No | The base URL of your web site (Pelican's `SITEURL` setting). If not passed this will default to the URL of your GitHub Pages site, which is correct in most cases. | string | The URL of your GitHub Pages site. |
| `feed_domain` | No | The domain to be prepended to feed URLs (Pelican's `FEED_DOMAIN` setting). If not passed this will default to the URL of your GitHub Pages site, which is correct in most cases. | string | The URL of your GitHub Pages site. |
| `deploy` | No | Whether to deploy the site to GitHub Pages. If this is `false` then your site will be built with Pelican but not deployed, which can be useful to test changes in pull requests before merging them (see below). | bool | `true` |
| `stork` | No | Whether to install [Stork](https://docs.rs/stork/latest/stork/) on the runner to be able to build a site with Stork search enabled | bool | `false` |

### Test building your site without deploying it

You can just test building your site with Pelican and not deploy it to GitHub Pages by passing `deploy: false` to the workflow. For example you might want to do this when commits are pushed to pull requests before merging the PR:

```yaml
name: Test Pelican build
on:
  push:
  workflow_dispatch:
jobs:
  deploy:
    uses: "seanh/ghp-pelican/.github/workflows/github_pages.yml@main"
    permissions:
      contents: "read"
      pages: "write"
      id-token: "write"
    with:
      settings: "publishconf.py"
      deploy: false
```

### "Insecure content" warnings from browsers

If your site uses `https://` and is broken because the browser is blocking network requests (for example for CSS files) due to "insecure content" this may be because GitHub Pages is generating `http://` URLs for your site.

To fix this go into your site repo's settings and enable the **Enforce HTTPS** setting: go to **Settings → Pages** and check **Enforce HTTPS**. Then re-run the workflow to re-deploy your site. Alternatively, you can use the workflow’s `siteurl` and `feed_domain` settings.
