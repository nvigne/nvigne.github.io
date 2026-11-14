# Nicolas Vignes Blog

A static Hugo blog using PaperMod. Markdown articles are rendered at build time;
the published site requires no backend.

## Local development

Install Hugo **extended 0.170.0** (the version pinned in the deployment workflow).
Clone with the theme submodule:

```sh
git clone --recurse-submodules https://github.com/nvigne/nvigne.github.io.git
cd nvigne.github.io
```

For an existing checkout, run `git submodule update --init --recursive`.

Create an article with `hugo new content posts/my-article.md`, then preview drafts
with `hugo server --buildDrafts`. New articles start with `draft: true`; change it
to `false` when ready to publish. Article files live in `content/posts`.

Build the production site with `hugo --minify --panicOnWarning`. Generated output
in `public`, the resource cache, and Hugo's build lock are ignored by Git.

## Deployment

Pushes to `main` build and deploy through the official GitHub Pages artifact and
deployment actions. The workflow can also be started manually.

In **Settings > Pages**, use **GitHub Actions** as the source and retain the custom
domain `blog.nicolasvignes.fr`. The local `baseURL` matches that domain; deployments
use the URL supplied by GitHub Pages. The root `CNAME` records the intended domain;
the custom domain for Actions deployments is managed in Pages settings.

PaperMod is pinned as a Git submodule, preserving the theme while avoiding a
vendored copy or requiring Go/Hugo Modules. To update it deliberately, run
`git submodule update --remote themes/PaperMod`, rebuild, and commit the updated
submodule reference. Update the Hugo version in the workflow and this document
together.
