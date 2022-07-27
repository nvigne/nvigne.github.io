---
title: "Host a Hugo website on Github pages"
date: 2022-07-27T17:34:43+02:00
draft: true
---

[GitHub pages](https://pages.github.com/) is a very simple way to host a static website. You can simply commit your html static page and GitHub will serve it as a static content from a github subdomain using your username.

You can even setup your [own custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site) if you own one.

The question is how can I host a static website that is generated like [Hugo](https://gohugo.io/) - for people who don't know, Hugo is a very light and flexible static site generators.

# Setup your Hugo repository
The first step that is common to any website would be to host your Hugo codesource in a GitHub repository.

To enable GitHub pages, you need to create a repository named `username.github.io` and must be a public repository.

In this article, I'm assuming that you have push your Hugo codesource to this repository and know how Git works.

# Default GitHub action
By default when you create a GitHub pages repository, your repository will have a new Action added nammed `pages-build=deployement`. This action cannot be modify or deleted. We cannot see the definition, but we can have access to the logs to analyze what the action does.

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

# Create our own GitHub action
We can't use the default GitHub action as we don't have a way to build our Hugo website, so let's build our own action.

## First step: build Hugo
In order to build our Hugo website, we need to first install Hugo on the machine, hopefully, there is already an existing action for that: `peaceiris/actions-hugo@v2` where you can specifiy the Hugo version to install.

After installing Hugo on the machine, we can actually build our website, calling `hugo --minify`

Now, we need to bundle the output into an archive and upload the artifact.





