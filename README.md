# missing

daozhanglv 的个人技术博客：<https://daozhanglv.github.io/>

记录我在做的项目和学习过程中的笔记。Hugo + GitHub Pages，原生模板，无第三方主题、前端构建依赖、跟踪脚本。

## 内容

| 类型 | 语言 | 说明 |
|---|---|---|
| 项目文章 | 英文 | 每个项目到达一个里程碑时写一篇：为什么做、怎么设计、还差什么 |
| 学习笔记 | 中文 | 系统、工具与编程问题的记录 |

目前写到的项目：

- [mac-autosync](https://github.com/daozhanglv/mac-autosync)：每晚自动把 Mac 的配置文件备份到 Git 仓库的模板。文章：[mac-autosync: a Mac config backup I never have to think about](https://daozhanglv.github.io/posts/mac-autosync/)
- [WortSchatz](https://github.com/daozhanglv/wortschatz)：德语词汇训练工具（文章待写）

## 写文章

```bash
hugo new content posts/my-note.md
hugo server -D
```

修改 `content/posts/my-note.md`，填写标题、description、tags 和正文。发布前设置 `draft: false`；日期在未来的文章不会输出。

推送到 `main` 后，GitHub Actions（`.github/workflows/hugo.yaml`）自动构建并部署，一两分钟后上线。部署结果可在仓库的 Actions 页面查看。

## 本地运行

安装 Hugo 0.167.0（与 Actions 保持一致；macOS 可以使用 `brew install hugo`），在本目录运行：

```bash
hugo server -D
```

打开 http://localhost:1313/。正式构建使用 `hugo --minify`，生成目录为 `public/`，无需提交该目录。

## 目录结构

- `content/posts/`：文章，一篇一个 Markdown 文件。
- `content/about.md`：关于页。
- `hugo.toml`：博客名、作者、说明、GitHub 链接与站点地址。
- `layouts/`：页面模板；`layouts/index.html` 是首页。
- `static/css/main.css`：颜色与排版。
- `static/favicon.svg`：浏览器图标。
- `archetypes/default.md`：新文章的模板。

功能：文章分页、标签索引、年份归档、阅读目录、相邻文章导航、RSS、sitemap、robots.txt、404 页、响应式布局、代码高亮。

## 自定义域名

目前使用默认地址，没有配置自定义域名或 CNAME。需要时在仓库 Pages 设置中配置域名，按 GitHub 提示添加 DNS 记录，再修改 `hugo.toml` 的 `baseURL`。
