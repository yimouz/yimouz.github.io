---

title: Hexo 单仓库双分支 配合action
date: 2026-01-30 14:00:21
tags: 杂记
categories: 杂记
author: 是啊蓁阿
summary: Hexo 单仓库双分支：摆脱本地hexo、node
  

---
# Hexo 单仓库双分支：摆脱本地hexo、node

本方案旨在实现一个**完全脱离本地环境**、随时随地可写作的现代化博客系统。你只需要一个 GitHub 账号，即可通过网页或任何 Git 客户端发布文章。

## 1. 核心架构：单仓库双分支

同一个 GitHub 仓库的两个分支，实现“源码”与“网页”的分离：

| 分支名 | 作用 | 内容 | 权限 |
| :--- | :--- | :--- | :--- |
| **`dev`** | **后台 (源码)** | Markdown 文章、配置文件、`package.json` | **默认分支**，你在此写作 |
| **`master`** | **前台 (网页)** | HTML/CSS/JS (由 Action 自动生成) | 仅由 Robot 推送，**勿手动修改** |

---

## 2. 初始环境搭建 (只需做一次)

### 第一步：准备仓库
1.  在 GitHub 创建一个新仓库，名称必须为：`你的用户名.github.io`。
2.  仓库设置为 **Public**（公开）。

### 第二步：初始化分支
在本地或者直接在 GitHub 网页上操作：
1.  确保你当前处于一个干净的分支（建议命名为 `dev`）。
2.  将 Hexo 源码（`source/`, `themes/`, `_config.yml`, `package.json` 等）全部推送到 `dev`。
3.  在 GitHub 仓库设置 (**Settings -> General**) 中，将 **Default branch** 修改为 **`dev`**。
4.  (可选) 删除远程的 `master` 分支（如果有），让 Action 稍后自动重建一个干净的。

### 第三步：配置自动化 (GitHub Actions)
在 `dev` 分支根目录下，创建文件 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hexo Blog

on:
  push:
    branches:
      - dev  # 监听源码分支的变动

permissions:
  contents: write # 必须赋予写权限

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source
        uses: actions/checkout@v4
        with:
          ref: dev          # 拉取源码
          submodules: true  # 兼容主题 submodule
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20' # 使用现代 Node 版本

      - name: Upgrade npm & Install Hexo CLI
        run: |
          npm install -g npm@latest
          npm install -g hexo-cli

      - name: Install Dependencies
        run: npm install --no-optional

      - name: Build Hexo
        run: hexo generate

      - name: Deploy to Master
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: master  # 发布到 master 分支
          commit_message: "Auto deploy: ${{ github.event.head_commit.message }}"
```

### 第四步：开启 GitHub Pages
等待第一次 Action 跑通（显示绿色 ✅）后：
1.  进入仓库 **Settings -> Pages**。
2.  **Source** 选择 `Deploy from a branch`。
3.  **Branch** 选择 `master` 分支（根目录 `/root`）。
4.  保存。

---

## 3. 日常写作流程 (完全云端化)

你可以选择以下任意一种方式写文章，**无需本地安装 Node.js/Hexo**。

### 方式 A：GitHub 网页版 (最简单)
1.  打开你的仓库首页（默认就是 `dev` 分支）。
2.  进入 `source/_posts/` 目录。
3.  点击 **Add file -> Create new file**。
4.  文件名填 `我的新文章.md`，内容写入：
    ```markdown
    ---
    title: 我的新文章
    date: 2024-03-21 12:00:00
    tags: [生活, 记录]
    ---
    
    这里是正文内容...
    ```
5.  点击底部的 **Commit changes**。
6.  **完成！** 等待 1-2 分钟，你的博客 `https://你的用户名.github.io` 就会自动更新。

### 方式 B：VS Code 网页版
1.  在你的仓库页面，直接按键盘上的 **`.` (句号)** 键。
2.  这会打开一个网页版的 VS Code 编辑器。
3.  像在本地一样写文章、管理文件。
4.  写完后在左侧 Git 栏提交并推送。

---

## 4. 避坑指南

1.  **依赖管理**：
    *   确保 `package.json` 里的 `hexo` 版本是新的（建议 `^7.0.0`）。
    *   **不要上传** `package-lock.json`（除非你很确定环境一致），让 Action 每次自己计算依赖，可以避免很多 npm 兼容性报错。

2.  **图片处理**：
    *   在网页端写文章时，上传图片比较麻烦。
    *   建议使用 **图床**（如 SM.MS, Imgur, 或自己搭建的 OSS），在文章里直接用 Markdown 链接 `![](图片URL)` 引用。

3.  **主题修改**：
    *   如果需要修改主题配置（如菜单、侧边栏），直接编辑 `_config.yml` 或主题目录下的配置文件并提交即可，Action 会自动重新编译生效。
