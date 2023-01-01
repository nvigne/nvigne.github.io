---
title: "Host a Hugo website on Github pages"
date: 2023-01-01T19:10:43+02:00
draft: false
---

[GitHub pages](https://pages.github.com/) is a very simple way to host a static website. You can simply commit your html static page and GitHub will serve it as a static content from a github subdomain using your username.

You can even setup your [own custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site) if you own one.

The question is how can I host a static website that is generated like [Hugo](https://gohugo.io/) - for people who don't know, Hugo is a very light and flexible static site generators.

In the following article, I will assume that you are familiar with Hugo, GitHub and especially GitHub actions.

# Setup your Hugo repository
The first step that is common to any website would be to host your Hugo codesource in a GitHub repository.

To enable GitHub pages, you need to create a repository named `username.github.io` and must be a public repository.

In this article, I'm assuming that you have push your Hugo codesource to this repository and know how Git works.

# Default GitHub action for GitHub pages
By default when you create a GitHub pages repository, your repository will have a new Action added nammed `pages-build-deployement`. This action cannot be modify or deleted. We cannot see the definition, but we can have access to the logs to analyze what the action does.

This action is run every time there is a push on the default branch of your repository

The action is divided in three steps:
- Build
- Deploy
- report-build-status
We will focus on the first two that we are interesting in.

## Build
The build action is meant for building [Jekyll](https://jekyllrb.com/) project (other static site generators), as we are not using Jekyll this action does nothing.

But it does bundle the output directory content into a tar file and upload the artifact.

## Deploy
The deploy action deployes the generated content to GitHub pages, we can see that it is using the `actions\deploy-pages@v1` action. 

There are no parameter passed to this action, the action is taking the first artifact uploaded in the build to be deployed.

# Create our own GitHub action
We can't use the default GitHub action as we don't have a way to build our Hugo website, so let's build our own action.

## First step: build Hugo
In order to build our Hugo website, we need to first install Hugo on the machine, hopefully, there is already an existing action for that: `peaceiris/actions-hugo@v2` where you can specifiy the Hugo version to install.

After installing Hugo on the machine, we can actually build our website, calling `hugo --minify`

Now, we need to bundle the output into an archive and upload the artifact. we can use a simple tar command to create the archive. Here is an example: `tar --dereference --hard-dereference --directory ./public -cvf artifact.tar .`

Finally upload the archive as an artifact, we can use the default GitHub action `actions/upload-artifact@main`

If we resume our first step, we need to install Hugo, build our website with Hugo, then create an archive and upload it as an artifact. Your GitHub action should looks like the below:

```yaml
build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify
        
      - name: Archive build output
        run: tar --dereference --hard-dereference --directory ./public -cvf artifact.tar .
      - name: Upload Artifact
        uses: actions/upload-artifact@main
        with:
          name: github-pages
          path: ./artifact.tar
          if-no-files-found: warn
```

## Second step: Deploy pages
We have now our Hugo website upload as a GitHub artifact. As we noticed earlier the GitHub action deploy is automatically picked to be deployed.

There are few things to know, you need additional permissions to read the artifact content and deploy the page, you need the additional permissions
```yaml
permissions:
      id-token: write
      contents: read
      pages: write
```

You also need to setup the environment proprerties of this step to indicates that you deploy to the `github-pages` environment. This was discussed in [this issue](https://github.com/actions/deploy-pages/issues/9). With the support of deploying Github pages from custom actions, it is documented [here](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow).

```yaml 
 environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
```

Once we have everything setup, we call call the actual action `actions/deploy-pages@v1`

Your yaml for the second step should looks like this:

```yaml 
deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pages: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v1
```

## Final yaml for your GitHub action
Your final yaml should looks like the following:

```yml
name: Deploy Github Pages

on:
  push:
    branches: [ "main" ]

  workflow_dispatch:

jobs:
  # Build the content for GitHub Pages
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify
        
      - name: Archive build output
        run: tar --dereference --hard-dereference --directory ./public -cvf artifact.tar .
      - name: Upload Artifact
        uses: actions/upload-artifact@main
        with:
          name: github-pages
          path: ./artifact.tar
          if-no-files-found: warn

  # Deploy to Github pages
  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pages: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v1
```
You can try to run it directly from the Actions menu.

# Switch to deploy using Actions
By default, when pushing a new commit to your main branch, it will trigger the default github page deployment action. We want to use instead use the action that we just created.

You can go to your repository, in Settings under Pages section. You can select the source for build and deployment and select `GitHub Actions`.

You can now run the Action and see your website deployed on your GitHub pages.
