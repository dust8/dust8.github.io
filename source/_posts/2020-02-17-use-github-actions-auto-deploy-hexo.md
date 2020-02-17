---
title: 使用Github Actions 来自动部署 Hexo 博客
date: 2020-02-17 05:15:31
tags:
  - Github Ations
  - Hexo
---

原理是把 `Hexo` 的源文件和 `Actions` 放到 `blog-source` 分支, 生成的静态文件放到主分支. 这样 `Actions` 里面监听到 `blog-source` 分支的推送就构建环境生成静态文件, 然后推送到主分支, 这样就可以了.
好处有几个:

- 源代码不用带着跑, 担心丢失, 一直在分支里面
- 不用手动生成静态文件, 部署, 专注于写作
- 不需要用第三方自动构建工具了

## Github Actions 配置

在源文件分支新建 `.github/workflows/ci.yml`

```yaml
name: 使用 Actions 自动部署 Hexo 博客

# 当推送的分支是源文件分支时才操作
on:
  push:
    branches:
      - blog-source

env:
  GIT_NAME: xxx
  GIT_EMAIL: xxx@users.noreply.github.com
  # 放源文件的分支
  BRANCH_SOURCE: blog-source

jobs:
  build:
    name: 构建与部署博客
    runs-on: ubuntu-latest
    steps:
      - name: 切换到源文件分支
        uses: actions/checkout@v2
        with:
          ref: ${{ env.BRANCH_SOURCE }}

      - name: 配置 Git 环境
        run: |
          git config --global user.name "$GIT_NAME"
          git config --global user.email "$GIT_EMAIL"

      - name: 设置 NodeJS
        uses: actions/setup-node@v1
        with:
          node-version: "10.x"

      - name: 安装 Hexo 及其相关插件
        run: |
          export TZ='Asia/Shanghai'
          npm install hexo-cli -g
          npm install hexo-generator-feed --save

      - name: 生成静态文件
        run: hexo g

      - name: 提交静态文件
        run: |
          cd ./public
          git init
          git add --all .
          git commit -m "Actions CI Auto Builder"

      - name: 推送静态文件
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          force: true
          directory: public
```

`GITHUB_TOKEN`, 在 [https://github.com/settings/tokens](https://github.com/settings/tokens) ,设置名字为 `GITHUB_TOKEN` , 然后勾选 `repo, admin:repo_hook, workflow` 等选项，最后点击 `Generate token` 即可。

## 参考链接

- [GitHub Actions Documentation](https://help.github.com/en/actions)
- [Marketplace for Actions](https://github.com/marketplace?utf8=%E2%9C%93&type=actions)
- [GitHub Action for GitHub Push](https://github.com/marketplace/actions/github-push)
- [git add 出现 "fatal: in unpopulated submodule XXX" 错误](https://blog.csdn.net/zixiao217/article/details/86693854)
