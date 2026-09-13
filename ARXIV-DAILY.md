# 每日 arXiv 论文动态接入

`arxiv-daily-researcher`（fork 仓库）每日生成 HTML 报告，推送到本站 `source` 分支的 `source/arxiv/reports/`，并重建 `source/arxiv/index.html`；随后本站 `deploy.yml` 自动重新构建上线。

本站是公开展示层，只保留最近 30 天报告和公开目录。完整报告历史、SQLite 备份、私有检索索引与人工知识沉淀放在私有仓库 `pengzhao311/quantum-lab`。

## 一、在 pengzhao311.github.io 侧（本仓库）

无需额外配置——`source/arxiv/` 已被 Hexo 原样复制进产物，`deploy.yml` 会在 `source` 分支有新提交时自动重建。

## 二、在 arxiv-daily-researcher fork 侧

### 1. 建 Fine-grained PAT

在 `pengzhao311` 账户下：**Settings → Developer settings → Fine-grained tokens → Generate new token**：

- Repository access: 仅选 `pengzhao311/pengzhao311.github.io`
- Permissions: `Contents` → `Read and write`
- 存到 fork 仓库 **Settings → Secrets and variables → Actions**，名称为 `PAGES_PAT`

如需同时归档到私有 `quantum-lab`，再建一个 token：

- Repository access: 仅选 `pengzhao311/quantum-lab`
- Permissions: `Contents` → `Read and write`
- 存到 fork 仓库 **Settings → Secrets and variables → Actions**，名称为 `QUANTUM_LAB_PAT`

### 2. 改 `.github/workflows/daily-run.yml`

把发布那一步替换为（推送到本站 `source` 分支，触发重建）：

```yaml
      - name: Publish reports to site source branch
        if: always()
        env:
          PAGES_PAT: ${{ secrets.PAGES_PAT }}
        run: |
          set -euo pipefail
          SITE=$(mktemp -d)
          git clone --depth 1 --branch source \
            "https://x-access-token:${PAGES_PAT}@github.com/pengzhao311/pengzhao311.github.io.git" "$SITE"

          mkdir -p "$SITE/source/arxiv/reports"
          if [ -d data/reports/daily_research/html ]; then
            cp -a data/reports/daily_research/html/. "$SITE/source/arxiv/reports/" || true
          fi

          python scripts/build_site_index.py "$SITE/source/arxiv" --max-age-days 30

          cd "$SITE"
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          if git diff --cached --quiet; then
            echo "No new reports."
          else
            git commit -m "Update arXiv reports: $(date -u +'%Y-%m-%d %H:%M UTC')"
            git push origin source
          fi
```

> `scripts/build_site_index.py` 也放在 fork 仓库的 `scripts/` 下（与本仓库同款）。

### 3. 触发链

```
fork 每日任务
  → 推最近 30 天公开报告与 index.html 到本站 source/arxiv
  → 推完整历史、SQLite 备份与私有索引到 quantum-lab/arxiv
  → 本站 source 分支有提交
  → 触发 deploy.yml → hexo generate → 部署 gh-pages → 线上 /arxiv/ 更新
```

## 注意事项

- PAT 只授 `Contents: Read and write` 于本仓库，泄露影响面最小。
- 报告文件名 `ARXIV_Report_<时间戳>.html`，脚本按文件名日期倒序排列。
- 本站默认保留最近 30 天公开 HTML；完整历史以 `quantum-lab` 为准。
- 不要往 `gh-pages` 分支手动 push；一切产物以 `source` + `deploy.yml` 为准。
