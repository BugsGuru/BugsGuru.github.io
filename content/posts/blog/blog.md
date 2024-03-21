---
title: "How to build the blog"
date: 2024-03-20

categories: [部署搭建]
tags: ["blog"]
keywords: ["blog","hugo","PaperMod"]

description: "How to build blog site with Hugo and PaperMod"
---

## 搭建流程
```sh
# Install hugo, build from source(go)：
go install -tags extended github.com/gohugoio/hugo@latest
hugo version

# Create a new site
hugo new site MyFreshWebsite --format yaml # replace MyFreshWebsite with name of your website

# Install theme: PaperMod
# Inside the folder of your Hugo site `MyFreshWebsite`, run:
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)


# In `config.yml` add:
# theme: ["PaperMod"]

# Creat posts inside the folder of your Hugo site `MyFreshWebsite/content/`

```

## 参考
- [Hugo文档](https://gohugo.io/installation/)
- [PaperMod文档](https://adityatelange.github.io/hugo-PaperMod/)
- [PaperMod配置参数](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-variables/)
- [PaperModExampleSite](https://github.com/adityatelange/hugo-PaperMod/tree/exampleSite)
- [ThisSiteSourceCode](https://github.com/BugsGuru/BugsGuru.github.io)
