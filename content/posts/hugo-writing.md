---
title: '用 Markdown 写下你的下一篇文章'
date: 2026-10-09T09:00:00+08:00
draft: false
description: '从创建文件到发布：这个博客的日常写作流程，只需要 Markdown 和一次提交。'
tags: ['Hugo', '写作']
example: true
---
## 创建文章

在博客项目根目录执行：

```bash
hugo new content posts/my-first-post.md
```

Hugo 会根据 `archetypes/default.md` 创建一个包含标题、时间和草稿状态的文件。

## 写入内容

文章顶部的 front matter 保存元数据，下面是 Markdown 正文。

```yaml
title: '我的第一篇文章'
description: '一句话说明文章在讨论什么。'
draft: false
tags: ['Linux']
```

文件里的日期应该保留。上线之前，记得把 `draft` 改为 `false`。

## 本地预览

```bash
hugo server -D
```

打开终端显示的本地地址，检查文章排版。`-D` 会显示草稿，方便修改。

## 发布更新

提交到仓库的 `main` 分支后，GitHub Actions 会构建并部署整个站点。

```bash
git add .
git commit -m "Add a new post"
git push
```

在 GitHub 的 Actions 页面可以查看部署结果。首次发布前，先按照项目 README 启用 GitHub Pages。
