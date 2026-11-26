# missing

daozhanglv 的个人作品集与技术博客：<https://daozhanglv.github.io/>

面向 Backend / Java 求职展示与日常工程笔记。站点界面与文章均为英文。采用 Hugo + GitHub Pages 原生模板构建，零第三方主题、零前端框架包袱、零跟踪脚本。

## 内容与结构

| 页面 / 类型 | 语言 | 说明 |
|---|---|---|
| `/projects/` 作品集 | 英文 | 核心工程项目的系统架构图、关键设计与里程碑文章索引 |
| `/resume/` 在线简历 | 英文 | 单页求职简历，支持一键 `Print / Save PDF` 打印导出标准 A4/Letter PDF |
| `/posts/` 项目与笔记 | 英文 | 项目里程碑复盘（为什么做、怎么设计、踩过什么坑、还差什么）与技术笔记 |

目前展示的项目：

- [WortSchatz](https://github.com/daozhanglv/wortschatz)：基于 Java 21 / Spring Boot 4 / PostgreSQL 17 的德语词汇训练后端。
  - 系列第 1 篇：[WortSchatz (1): building the data foundation with Spring Boot and PostgreSQL](https://daozhanglv.github.io/posts/wortschatz-1-data-foundation/)
- [mac-autosync](https://github.com/daozhanglv/mac-autosync)：每晚自动把 Mac 配置文件与 Homebrew 包列表备份到私有 Git 仓库的双层防泄漏工具。
  - 设计复盘：[mac-autosync: a Mac config backup I never have to think about](https://daozhanglv.github.io/posts/mac-autosync/)

## 写文章与维护作品集

```bash
hugo new content posts/my-note.md
hugo server -D
```

1. **填写 Frontmatter**：在 `content/posts/my-note.md` 中填写 `title`、`description`、`tags`；如果属于某个连载项目，填写 `series: ['WortSchatz']` 即可自动渲染文章顶部的系列目录导航。发布前将 `draft` 设为 `false`。
2. **固定源码链接版本**：里程碑文章中引用其他仓库源码时，建议使用当时的 Commit SHA 或 Tag（如 `blob/v0.1.0/...` 而非 `blob/main/...`），避免后续主分支重构导致文章链接 404。
3. **插入架构图**：正文支持原生 `mermaid` 代码块（仅在包含 `mermaid` 代码块的文章按需加载渲染脚本），作品集页支持在 `static/diagrams/` 放置 SVG 并在 `data/projects.yaml` 引用。
4. **更新作品集与简历**：
   - 修改 `data/projects.yaml` 可同时更新首页 Selected Projects、`/projects/` 详情与 `/resume/` 项目栏；支持在 `posts` 数组中挂载多篇里程碑文章。
   - 修改 `data/resume.yaml` 可补充教育背景（`education`）与工作经历（`experience`），取消注释后 `/resume/` 页面会自动渲染对应区块。
   - 修改 `hugo.toml` 的 `[params]` 可配置 `status`（求职状态）、`linkedin`、`location`、`resume_pdf` 等可选字段。

推送到 `main` 后，GitHub Actions（`.github/workflows/hugo.yaml`）会自动构建、校验全部站内链接与 RSS 格式，并在一两分钟内完成部署。

## 本地运行

安装 Hugo 0.167.0（与 Actions 保持一致；macOS 可使用 `brew install hugo`），在本目录运行：

```bash
hugo server -D
```

打开 http://localhost:1313/。正式构建使用 `hugo --gc --minify`，生成目录为 `public/`（已在 `.gitignore` 中忽略）。

## 目录结构

- `content/posts/`：技术文章，一篇一个 Markdown 文件。
- `content/projects.md` & `data/projects.yaml`：作品集页面与结构化项目数据（支持架构图与多篇里程碑文章）。
- `content/resume.md` & `data/resume.yaml`：在线简历页面与结构化简历数据（含 `@media print` 打印样式）。
- `content/about.md`：关于与求职联系页。
- `hugo.toml`：站点定位、求职信息、分类（`tags` / `series`）与代码高亮配置。
- `layouts/`：原生 Hugo 模板与构建期 Markdown Render Hooks（代码块语言栏与复制、外链安全属性、Mermaid 图表、纯文章全文 RSS）。
- `assets/css/main.css`：满足 WCAG AA 对比度的色彩系统、响应式排版、Chroma 代码高亮与打印样式。
- `static/`：`favicon.svg`、Open Graph 社交分享卡片 `og-card.png` 及项目架构图 `diagrams/*.svg`。

## 自定义域名

目前使用默认 GitHub Pages 地址。后续如需绑定自定义域名，在仓库 Pages 设置中添加域名与 DNS 记录，并同步更新 `hugo.toml` 的 `baseURL` 即可。
