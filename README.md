> [!IMPORTANT]
> This workflow got merged into the Pelican project, you should [use the official version of the workflow](https://docs.getpelican.com/en/latest/tips.html#publishing-to-github-pages-using-a-custom-github-actions-workflow) instead.

Use GitHub Actions to Deploy a [Pelican](https://getpelican.com/) Site to GitHub Pages
--------------------------------------------------------------------------------------

This repo contains a [reusable GitHub Actions workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for deploying a [Pelican](https://getpelican.com/) site to [GitHub Pages](https://pages.github.com/): push the source files of your site to a branch of your GitHub repo and GitHub Actions runs Pelican to build your output files and deploys them to your GitHub Pages site. You can even edit your site's content using GitHub's web interface and the changes will be automatically deployed. There's a [demo repo](https://github.com/seanh/pelican-github-pages-demo/) showing the workflow in use.

Explanation
-----------

GitHub has two ways to deploy a repo's GitHub Pages site:

1. You can **Deploy from a branch** (also called the "Classic Pages experience"). This uses the [Jekyll](https://jekyllrb.com/) static site generator. If you want to use a different static site generator you have to run your generator locally and push the generated output files to a branch. Or you can write a GitHub Actions workflow that run your generator and pushes the output files to a branch for you.
2. Alternatively, you can use **GitHub Actions** to deploy your site directly, which enables deeper integration of non-Jekyll site generators. This is the recommended way to use non-Jekyll site generators with GitHub Pages. You just push your Pelican source content files to a branch of your repo (such as the `main` branch) and a GitHub Actions workflow runs automatically on each push, builds your site with Pelican, and deploys the output files to GitHub Pages. This is what the GitHub Actions workflow in this repo does.

See the [GitHub Pages docs about publishing sources](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) for more about the two ways of publishing a GitHub Pages site.


Usage
-----

1. Create a Pelican site in your GitHub repo, for example by following [Pelican's quickstart](https://docs.getpelican.com/en/latest/quickstart.html).

   * Your repo needs to contain at least a Pelican config file and a content directory.

   * **You don't need to set `SITEURL`** in your Pelican config file: the workflow will set it correctly for you.

   * **You don't need to set `FEED_DOMAIN`** in your Pelican config file: the workflow will set it correctly for you.

   * **You don't need to commit the output directory to git**: the workflow will build the output files for you and deploy them to GitHub Pages directly.
   
2. Enable GitHub Pages in your repo: go to **Settings &rarr; Pages** and choose **GitHub Actions** as the **Source**:

   ![GitHub Pages deployment settings](/settings.png)

3. Add a `.github/workflows/pelican.yml` file to your repo with these contents:

   ```yaml
   name: Deploy
   on:
     push:
       branches: ["main"]
     workflow_dispatch:
   jobs:
     Deploy:
       uses: seanh/ghp-pelican/.github/workflows/pelican.yml@main
       permissions:
         contents: read
         pages: write
         id-token: write
    with:
      settings: "publishconf.py"
   ```

   You may want to replace the `@main` with the ID of a specific commit in this repo in order to pin the version of the reusable workflow that you're using. For example: `uses: seanh/ghp-pelican/.github/workflows/pelican.yml@0cc99c14466712c5bc2406660cd988541afcd767`. If you do this you might want to get Dependabot to send you automated pull requests to update that commit ID whenever new versions of this workflow are published, like so:

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

5. Go to the **Actions** tab in your repo (`https://github.com/YOUR_USER/YOUR_REPO/actions`) and you should see a **Deploy Pelican Site to GitHub Pages** action running.

6. Once the action completes you should see your Pelican site deployed at your repo's GitHub Pages URL: `https://YOUR_USER.github.io.` for a user or organization site, or `https://YOUR_USER.github.io/YOUR_REPO` for a project site.

A number of optional inputs can be added to the ``with:`` block when calling the workflow, for example:

```yaml
with:
  settings: "publishconf.py"
  requirements: "pelican[markdown] typogrify"
  theme: "https://github.com/seanh/sidecar.git"
  python: "3.12"
```


Here's the complete list of workflow inputs:

<dl>
  <dt><code>settings</code></dt>
  <dd>The path to your Pelican settings file (the <code>pelican</code>
  command's <code>--settings</code> option), for example:
  <code>"publishconf.py"</code>. This is the only required setting.</dd>

  <dt><code>requirements</code></dt>
  <dd>The Python requirements to install.
  For example to enable Markdown and Typogrify use: <code>"pelican[markdown] typogrify"</code>.
  If you have a requirements file use: <code>"-r requirements.txt"</code>.
  Default: <code>"pelican"</code>.</dd>

  <dt><code>theme</code></dt>
  <dd>The GitHub repo URL of a custom theme to use, for example: <code>"https://github.com/seanh/sidecar.git"</code>.</dd>

  <dt><code>python</code></dt>
  <dd>The version of Python to use to build the site, for example: <code>"3.12"</code> (to use the most recent version of Python 3.12, this is faster) or <code>"3.12.1"</code> (to use an exact version, slower).
  Default: <code>"3.12"</code>.</dd>

  <dt><code>siteurl</code></dt>
  <dd>The base URL of your web site (Pelican's <code>SITEURL</code> setting).
  If not passed this will default to the URL of your GitHub Pages site, which is correct in most cases.</dd>

  <dt><code>feed_domain</code></dt>
  <dd>The domain to be prepended to feed URLs (Pelican's <code>FEED_DOMAIN</code> setting).
  If not passed this will default to the URL of your GitHub Pages site, which is correct in most cases.</dd>

  <dt><code>output-path</code></dt>
  <dd>Where to output the generated files (the <code>pelican</code> command's <code>--output</code> option).
  Default: <code>"output/"</code>.</dd>
</dl>
