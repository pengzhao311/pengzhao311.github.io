# 课题组网站（pengzhao311.github.io）

基于 **Hexo 7.3 + NexT 8.20（Gemini 主题）** 构建，GitHub Actions 自动构建并部署到 GitHub Pages。包含：首页、研究方向、成员、论文列表，以及**每日 arXiv 论文动态**。

## 分支结构

| 分支 | 内容 | 说明 |
| --- | --- | --- |
| `source`（默认） | Hexo 工程源码 | 改内容都在这 |
| `gh-pages` | 构建产物 | `deploy.yml` 自动生成，**勿手动修改** |

## 目录说明

```
├── _config.yml            # Hexo 站点配置（标题/URL/主题等）
├── _config.next.yml       # NexT 主题配置（Gemini 方案、菜单、暗色模式）
├── source/
│   ├── _posts/            # 博客文章
│   ├── members/           # 成员页（占位）
│   ├── research/          # 研究方向页（占位）
│   ├── publications/      # 论文列表页（占位）
│   ├── arxiv/             # 每日论文动态（index.html 由脚本生成，reports/ 存日报）
│   └── _data/languages.yml # 自定义菜单中文标签
├── scripts/build_site_index.py  # 生成「论文动态」列表页
└── .github/workflows/deploy.yml # 构建 + 部署到 gh-pages
```

## 本地开发

```bash
npm install          # 安装依赖（需 Node 18+）
npx hexo server      # 本地预览 http://localhost:4000
npx hexo generate    # 生成静态产物到 public/
```

## 部署

1. GitHub 仓库 **Settings → Pages**：Source 选 `Deploy from a branch`，Branch 选 `gh-pages` / `/ (root)`。
2. 推送到 `source` 分支即触发 `deploy.yml`，自动构建并部署。
3. 线上地址：https://pengzhao311.github.io/

## 每日论文动态

见 [ARXIV-DAILY.md](ARXIV-DAILY.md)。

## 修改站点信息（占位 → 真实）

1. **站点标题/副标题/作者**：改 `_config.yml` 的 `title` / `subtitle` / `author`。
2. **成员/方向/论文列表**：直接编辑 `source/members/index.md`、`source/research/index.md`、`source/publications/index.md`。
3. **菜单文字**：`source/_data/languages.yml`。
4. **新增导航项**：`_config.next.yml` 的 `menu`。

## 首次发布（把工程推到 source 分支）

```bash
git init
git checkout -b source
git add -A
git commit -m "Rebuild site: Hexo 7.3 + NexT 8.20 (Gemini)"
git remote add origin https://github.com/pengzhao311/pengzhao311.github.io.git
git push -u origin source
```

推送后到 **Settings → Branches** 把默认分支设为 `source`；再到 **Settings → Pages** 选 `gh-pages` 分支部署。首次 `deploy.yml` 运行会清空旧 `gh-pages` 内容并写入新站点。
