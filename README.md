# missing

为 daozhanglv 准备的个人技术博客。Hugo + GitHub Pages，原生模板，无第三方主题、前端构建依赖、跟踪脚本。米白色、黑色细线与蓝色强调，参考 ArchLinuxStudio 的编辑式排版，重新实现为个人博客。

## 本地运行

安装 Hugo 0.167.0（与 Actions 保持一致；macOS 可以使用 `brew install hugo`），在本目录运行：

```bash
hugo server -D
```

打开 http://localhost:1313/。正式构建使用 `hugo --minify`，生成目录为 `public/`，无需提交该目录。

## 首次部署到 GitHub Pages

1. 登录 https://github.com/daozhanglv，创建名为 `daozhanglv.github.io` 的公开仓库。不要预先添加 README，方便直接推送。
2. 把本目录的全部文件（包含隐藏的 `.github/` 和 `.gitignore`）提交到仓库：

```bash
git init -b main
git add .
git commit -m "Create missing Hugo blog"
git remote add origin https://github.com/daozhanglv/daozhanglv.github.io.git
git push -u origin main
```

3. 在仓库 **Settings → Pages → Build and deployment → Source** 选择 **GitHub Actions**。
4. 在 **Actions → Build and deploy Hugo → Run workflow** 手动运行一次（如果首次 push 已成功部署，无需重复）。
5. 部署成功后访问 https://daozhanglv.github.io/。

如果这个仓库已存在，请先备份并检查原有内容，不要直接覆盖。也可以使用其他仓库名，访问地址将变为 `https://daozhanglv.github.io/仓库名/`；修改 `hugo.toml` 的 `baseURL`。Actions 构建会自动使用 Pages 返回的实际地址，导航和样式也支持子路径。

自动部署参照 Hugo 官方指南：https://gohugo.io/host-and-deploy/deploy-to-github-pages/

## 写文章

```bash
hugo new content posts/my-note.md
hugo server -D
```

修改 `content/posts/my-note.md`，填写标题、description、tags 和正文。发布前设置 `draft: false`；未来日期的文章默认不会输出。推送到 main 后自动重新部署。

三篇起始文章均标记为「示例文章」，不是作者已发表的个人经历。正式上线时删除或替换它们，自己的新文章无需添加 `example: true`。

## 修改个人信息

- `hugo.toml`：博客名、作者、说明、GitHub 链接与站点地址。
- `content/about.md`：个人介绍。
- `layouts/index.html`：首页介绍与主标题。
- `static/css/main.css`：颜色与排版。
- `static/favicon.svg`：浏览器图标。

功能：文章分页、标签索引、年份归档、阅读目录、相邻文章导航、RSS、sitemap、robots.txt、404 页、响应式布局、代码高亮。

## 自定义域名

在仓库 Pages 设置中配置域名，按 GitHub 提示添加 DNS 记录，再修改 `baseURL`。本项目没有预设自定义域名或 CNAME。
