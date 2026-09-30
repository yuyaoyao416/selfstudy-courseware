# self-study 课件静态站点

self-study 工作区（`E:\Deepseek Harness\self-study`）产出的 HTML 课件部署包。
只含课件本身（HTML + 样式 + 脚本 + 配图，约 9MB / 112 个文件 / 11 个页面），
**不含** `reference/`（老师课件 PDF，版权内容）、学习进度、学习记录、会话摘要等个人数据。

## 目录结构

```
index.html                  课程总览（4 个科目入口）
.nojekyll                   告诉 GitHub Pages 原样托管，别用 Jekyll 处理
learning/
  assets/                   共享主题层（Sayo UI + learn-theme）
  subjects/<科目>/
    index.html              科目主页
    lessons/0001-*.html     课时课件
    assets/                 科目级 css/js 与配图
```

## 部署到 GitHub Pages

1. 在 GitHub 建一个**公开**仓库（私有仓库开 Pages 需要 Pro；本包已剔除个人数据，公开安全）
2. 在本目录执行：

```powershell
git init
git add -A
git commit -m "courseware"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

3. 仓库网页 → Settings → Pages → Build and deployment → Source 选 `Deploy from a branch`
   → Branch 选 `main` / `/(root)` → Save
4. 一两分钟后访问 `https://<用户名>.github.io/<仓库名>/`

限制参考：[GitHub Pages 官方限制](https://docs.github.com/zh/pages/getting-started-with-github-pages/github-pages-limits)
（源仓库建议 ≤1GB、单文件 ≤100MB、月带宽软限 100GB、每小时构建 ≤10 次）

## 部署到 Vercel

```powershell
npx vercel login      # 浏览器授权（首次）
npx vercel --prod     # 本目录直接部署，得到 xxx.vercel.app
```

免费 Hobby 套餐自带 HTTPS 和全球 CDN。如需"只自己看"，可在项目设置里开启
Production Deployment Protection（Vercel 已宣布该保护对所有套餐免费）。

## 更新课件（已自动化）

学习工作区里生成了新课文/新科目后，**什么也不用做**——每次课程更新后，学习 Agent 会主动运行 `sync-courseware.ps1` 把更新送到 GitHub。

手动触发（想立刻看到效果时）：

```powershell
powershell -File "E:\Deepseek Harness\杂项\sync-courseware.ps1"
```

脚本做的事：重新打包（自动过滤个人数据 + 全量引用自检）→ git 提交 → push；
git 通道不通时自动降级到 GitHub API 增量推送（复用远端已有 blob，只传变化的文件）；
全部失败时改动存本地，下次运行自动补推，不丢数据。
运行日志：`E:\Deepseek Harness\杂项\sync-log.txt`。

GitHub 收到更新后自动重新发布，1~2 分钟后网站就是新版。

**网络说明**：本机 git 已配置 `http.proxy=http://127.0.0.1:7890`（GitHub 直连不通，走本机代理）。
代理没开时推送会失败并自动转为本地暂存，等代理恢复后下一次运行自动补推。

脚本做的事（详见脚本头部注释）：拷课件 → 把根页 `.learning/` 引用改写为 `learning/`
（部署副本不带隐藏点目录）→ 摘掉科目页上指向个人笔记的附件链接 → 写 `.nojekyll`
→ 自检所有 HTML 本地引用可解析。
