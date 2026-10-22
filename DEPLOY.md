# Deploying to GitHub Pages

The site is a [Zola](https://www.getzola.org/) project. `.github/workflows/pages.yml` builds it and publishes it with the official Pages actions. It runs on every push to `main` and can also be started by hand (Actions → "Deploy to GitHub Pages" → Run workflow).

Fork deployment: https://brijesh-thakkar.github.io/unified-website/

## How the workflow works

1. **configure-pages** reads the repo's Pages settings and outputs `base_url` and `base_path` (for example `/unified-website`).
2. Zola 0.21.0 is installed. It is pinned because Zola 0.22 and later reject `[markdown.highlighting] enabled = true` in `config.toml`.
3. `zola build --base-url <base_url>/` writes the site to `public/`. The templates build relative asset paths, so CSS, JS and images work under the subpath.
4. Markdown content contains root-absolute links such as `/docs/peers/`. A `sed` step prefixes these `href`/`src` values with `base_path`. It does nothing if the site is served from a domain root.
5. **upload-pages-artifact** packages `public/`, and **deploy-pages** publishes it in a second job that uses the `github-pages` environment.

The existing workflows (`deploy.yml`, `preview.yml`, and the others) use a self-hosted runner and Filebase/IPFS. They are untouched. `pages.yml` runs on `ubuntu-latest` and is independent of them.

## Enabling it on seetadev/unified-website

1. Merge `pages.yml` into `main`.
2. In the repo go to **Settings → Pages → Build and deployment → Source** and choose **GitHub Actions**. Or run:
   ```
   gh api -X POST repos/seetadev/unified-website/pages -f build_type=workflow
   ```
3. Push to `main`, or run the workflow manually. The site appears at https://seetadev.github.io/unified-website/. The `base_path` is detected automatically, so no edits are needed.
4. If `deploy.yml` also fires on push and queues forever because it needs the self-hosted runner, disable it or change its `runs-on` for that repo.
5. For a custom domain, set it under Settings → Pages. `base_path` becomes empty and the link-rewrite step is skipped.
