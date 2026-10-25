# blog

Cyanote 博客的文章与发布仓库。

- 线上站点：**https://lyrissonare.github.io/blog/**
- 博客系统：[Lyrissonare/Cyanote_Lite](https://github.com/Lyrissonare/Cyanote_Lite)

## 目录

```
content/
├─ posts/*.md       # 文章（Hexo 式 front-matter，文件名为 slug）
├─ settings.json    # 站点设置：关于页、公告、友链、首页条数
└─ index.json       # 构建产物，勿手改
.github/workflows/
└─ deploy.yml       # 拉取 Cyanote_Lite 引擎并部署到 Pages
```

## 写文章与发布

- **Windows 发布器**：用 Cyanote Lite 的 `CyanotePublisher` 选择 `.md`，一键推送
- **网页 / 本地**：新增或修改 `content/posts/*.md`，提交到 `main`

推送后，`deploy-blog` 工作流会自动使用最新版的 Cyanote Lite 引擎重建 `content/index.json`、组装站点并发布到 GitHub Pages。

系统代码升级后，到 **Actions → deploy-blog → Run workflow** 手动部署一次即可。

草稿（front-matter 中 `status: draft`）不会进入公开索引，但文件仍会保留在本仓库中。

## 本地预览

把本仓库的 `content/` 放进 Cyanote_Lite 的 `content/`，然后在系统仓库运行：

```bash
npm run dev   # http://localhost:3820
```
