---
date: '2024-12-20T21:14:00+08:00'
draft: false
title: '使用GitHub托管Hugo博客'
tags: ["Hugo", "github"]
---

### 新建一个GitHub仓库用于托管博客
直接去GitHub创建一个仓库即可，这里要注意GitHub page分为[user/org site和project site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-a-repository-for-your-site)，如果是user site，仓库名要符合`<user>.github.io`，因为我已经有了user site，所以在这里随便创建了一个名为`blog`的repo。  
把仓库克隆到本地，可能是因为proxy的原因，我刚开始通过https克隆报了很多奇奇怪怪的错，最后还是使用了ssh下载下来的（通过ssh连接GitHub可以看[这篇](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)）
```
git clone git@github.com:xliu-git/blog.git
```
### 安装Hugo  
Hugo是一个由Go生成的静态网站生成器，同类型的静态网站生成器还有Hexo、Jekyll等，都比较常见于个人博客，Hugo号称渲染速度更快， 我直接用的Hombrew装的，安装指南可以参考[Hugo文档](https://www.gohugo.org/)  
```
brew install Hugo
```
验证一下安装是否成功
```
hugo version
```

### 本地搭建博客
Hugo提供了很多的[博客模版](https://themes.gohugo.io/tags/blog/)，不同的模版在后续的配置中也有一些不同，可以直接参考模版的README来进行配置。
我使用的是模版`PaperMod`，进入本地仓库根目录，使用Hugo新建一个静态网站，因为`PaperMod`推荐使用yaml配置，所以加上了yaml option
```
hugo new site . --force --format yaml
```
通过git submodule来使用模版
```
git submodule add git@github.com:adityatelange/hugo-PaperMod.git themes/PaperMod
```
运行Hugo，根据运行提示点开相应的本地端口（e.g. http://localhost:1313/）
```
hugo server
```
Hugo会按照模版生成public文件夹，也就是一个完整的静态网站，通过调整config文件，可以实现一些模版里特定的[feature](https://github.com/adityatelange/hugo-PaperMod/wiki/Features)。通过Hugo new content可以发布文章，并在本地编辑md文件以及查看
```
hugo new content content/posts/my-first-post.md
```

### 添加持续部署脚本
GitHub Page的托管大概分两种，一种是在根目录或者/docs提供entry file，比如`index.html`、`index.md`、`README.md`；另一种是通过一个GitHub Actions workflow，提供一个部署脚本，Hugo提供了一个非常详细的指南关于[怎么在GitHub Page上托管Hugo项目](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
#### 在.github/workflows创建部署脚本hugo.yaml
```
mkdir -p .github/workflows
touch hugo.yaml
```
#### 复制粘贴到`.github/workflows/hugo.yaml`
```
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.137.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
#### 把部署模式改成GitHub Actions
找到相应仓库的Settings -> Pages -> Build and deployment设置成GitHub Actions

### 使用GitHub Page托管博客
把本地的更新push到托管仓库
```
git add -A
git commit -m "blog initialization"
git push
```
脚本中已经设置好了main分支的push会自动trigger GitHub Page的部署流程
```
# Runs on pushes targeting the default branch
  push:
    branches:
      - main
```
检查Actions部署成功之后，就可以在相应URL看到博客了
