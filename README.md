# self-study 课件静态站点

self-study 工作区（`E:\Deepseek Harness\self-study`）产出的 HTML 课件部署包。
只含课件本身（HTML + 样式 + 脚本 + 配图，约 9MB / 111 个文件 / 10 个页面），
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

## 更新课件后重新部署

学习工作区里生成了新课文/新科目后，重跑打包脚本，再二选一推送：

```powershell
# 1. 重新打包（自动过滤个人数据 + 全量引用自检）
pwsh -File "E:\Deepseek Harness\杂项\package-site.ps1"

# 2a. 网络正常时：git 推送
cd "E:\Deepseek Harness\杂项\selfstudy-site"
git add -A; git commit -m "update courseware"; git push

# 2b. git 通道被掐断时：API 推送（复用远端已有 blob，只传变化的文件，小请求扛抖动）
pwsh -File "E:\Deepseek Harness\杂项\fix-repo-via-api.ps1"
```

GitHub 收到更新后自动重新发布，1~2 分钟后网站就是新版。

**历史说明**：远端主提交由 API 创建，与本地提交历史不一致（两边内容完全相同）。
网络恢复后执行一次 `git push -f origin main`，本地历史即接管，之后走 2a 即可。
API 脚本会逐文件比对 blob sha：内容没变的直接复用远端对象，只上传真正变化的文件。

脚本做的事（详见脚本头部注释）：拷课件 → 把根页 `.learning/` 引用改写为 `learning/`
（部署副本不带隐藏点目录）→ 摘掉科目页上指向个人笔记的附件链接 → 写 `.nojekyll`
→ 自检所有 HTML 本地引用可解析。
