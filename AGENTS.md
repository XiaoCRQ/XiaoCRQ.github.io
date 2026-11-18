# AGENTS.md

给在此仓库工作的 AI 助手的说明。

## 这是什么

基于 [fuwari](https://github.com/saicaca/fuwari) 主题的 Astro 5 静态博客，部署在 GitHub Pages。

- 线上地址：https://xiaocrq.github.io/
- 仓库：`XiaoCRQ/XiaoCRQ.github.io`
- 这是**用户主页站**（仓库名等于 `<username>.github.io`），因此 `base` 必须是 `/`，不要改动。改了 `base` 会让图片、内链、Pagefind 索引全部 404。

## 环境

- Node ≥ 22，包管理器 **只能** 用 pnpm（`package.json` 有 `preinstall: npx only-allow pnpm`，且锁定 `pnpm@9.14.4`）。用 npm/yarn 会直接失败。
- 依赖已装在本地 `node_modules/`。

## 常用命令

```bash
pnpm dev        # 本地开发，默认 http://localhost:4321
pnpm build      # astro build + pagefind 建搜索索引，输出到 dist/
pnpm preview    # 预览 dist/
pnpm new-post  <文件名>   # 在 src/content/posts/ 生成带 frontmatter 的模板
pnpm check      # astro check 类型检查
```

改完必须跑一次 `pnpm build` 再交付——它同时验证 Astro 构建和 Pagefind 索引两步。

## 关键文件

| 路径 | 作用 |
|---|---|
| `astro.config.mjs` | `site` 必须是 `https://xiaocrq.github.io/`；`base` 保持 `/`；`trailingSlash: "always"`（URL 一律带尾斜杠） |
| `src/config.ts` | 站点标题（`title: "XiaoCRQ"`）、副标题、`lang: "en"`（界面文案是英文，正文内容是中文）、主题色、导航栏、侧边栏资料卡片、许可证 |
| `src/content/posts/` | 文章，每篇一个 `.md`，URL 是 `/posts/<文件名>/`（不是根路径）。**目前是空的**，只有 `.gitkeep` 占位 |
| `src/content/spec/about.md` | About 页正文（页面标题来自 i18n，正文里的 `# About` 是手写的） |
| `src/assets/images/avatar.jpg` | 头像（构建时由 Astro 转 webp 优化） |
| `.github/workflows/deploy.yml` | 推送到 `main` 自动构建 + 部署到 Pages |

文章 frontmatter 字段：`title` `published` `description` `image` `tags` `category` `draft` `lang`。`draft: true` 的文章不会出现在构建产物里。

## 部署

- 推送 `main` → Actions 跑 `withastro/action@v5` 构建 → `actions/deploy-pages@v4` 发布。
- 也可以手动跑：GitHub → Actions → "Deploy to GitHub Pages" → Run workflow。
- **本机 SSH 对 GitHub 认证是失败的**（`git@github.com: Permission denied (publickey)`）。不要反复尝试 push，把命令交给用户执行：

  ```bash
  cd /home/XiaoCRQ/Work/Program/Project/Blog && git push
  ```

- 部署失败时旧版本仍留在 `main_back` 分支（这是替换前的旧 HTML 首页，只有 63 个文件的静态站，与当前代码无共同历史）。需要回滚：Pages 设置里把 Source 改回 `Deploy from a branch` 并选 `main_back`。

## 已知坑

- **Pagefind 对 `zh-cn` 不做 stemming**：当前 `lang: "en"` 不受影响；一旦把 `lang` 改回 `zh_CN`，中文搜索变成字面匹配，构建日志会出现 `Note: Pagefind doesn't support stemming for the language zh-cn.`，属正常现象。
- 站内链接写相对/根路径即可，fuwari 会自动加 `base`；外部链接要加 `external: true` 才会新开标签页。
- `.gitignore` 已排除 `dist/`、`node_modules/`、`.astro/`、`.env`。不要把这些提交上去。
- **本地删掉文章后，构建仍会生成旧文章页面并报 `Image file not found`**：Astro 5 的内容缓存是 `node_modules/.astro/data-store.json`，删 `.astro/` 没用。清缓存：`mv node_modules/.astro /tmp/astro-cache-backup`，再重新 `pnpm build`。
- **同一个坑在 CI 上更隐蔽**：`withastro/action@v5` 默认 `cache: true`，缓存 `node_modules/.astro`，key 是 `astro-cache-<os>-<sha>` 但 `restore-keys` 是前缀 `astro-cache-<os>-`，也就是**不管内容变没变都命中上一次的缓存**。删文章后 Actions 会拿旧 data-store 去渲染已删除的文章，报 `[ERROR] Image file not found: src/content/posts/guide/cover.jpeg` + `file is not a function` 然后 exit 1。所以 `deploy.yml` 里明确设了 `cache: false`，不要打开。
- 文章集合为空时构建会输出 `The collection "posts" does not exist or is empty`，只是警告，exit code 仍为 0。`src/content/posts/.gitkeep` 用来保住空目录，不要删。
- `src/assets/images/demo-banner.png`（878KB）和 `demo-avatar.png`（407KB）已不再被引用，删除前先问。
- `vercel.json` 是模板遗留文件，GitHub Pages 用不到，无害。
- 给用户的写作手册在 `usage.md`，改写作相关规则时两处一起更新。

## 协作偏好

用户是编程新手，中文交流。改配置时：先解释这一项是干什么的、有什么取舍，让用户选择后再动手；一次只推进一件事，改完报告验证结果（构建输出、线上状态）。
