# 使用说明与备忘录

这个文件是给你自己看的：怎么发文章、怎么写、哪些坑踩过。配置相关的技术说明在 `AGENTS.md`。

## 一、站点速查

| 项目 | 值 |
|---|---|
| 线上地址 | https://xiaocrq.github.io/ |
| 仓库 | `XiaoCRQ/XiaoCRQ.github.io`（SSH：`git@github.com:XiaoCRQ/XiaoCRQ.github.io.git`） |
| 本地目录 | `/home/XiaoCRQ/Work/Program/Project/Blog` |
| 主题 | fuwari（Astro 5 静态站） |
| 分支 | `main` = 博客源码；`main_back` = 换主题前的旧 HTML 首页（别删） |
| 包管理器 | 只能用 pnpm（`package.json` 里 `preinstall` 强制检查，npm/yarn 直接报错） |

## 二、目录速查

| 路径 | 作用 |
|---|---|
| `src/content/posts/` | **文章都在这里**，一篇一个 `.md` 文件 |
| `src/content/spec/about.md` | 「关于」页正文 |
| `src/config.ts` | 站点标题、副标题、语言、主题色、导航栏、侧边栏个人信息、社交链接 |
| `astro.config.mjs` | `site`（正式网址）、`base`、构建插件配置 |
| `src/assets/images/avatar.jpg` | 头像（换成别的图时替换这个文件，或改 `config.ts` 里的路径） |
| `public/` | 原样复制的静态文件，按 `/文件名` 访问 |
| `.github/workflows/deploy.yml` | 自动部署配置，一般不用动 |
| `dist/` | 构建产物，**不要提交**，已忽略 |
| `AGENTS.md` / `usage.md` | 给 AI 助手的说明 / 给你自己的说明 |

## 三、发一篇文章（完整流程）

```bash
cd /home/XiaoCRQ/Work/Program/Project/Blog

pnpm new-post acm-first-contest   # 生成 src/content/posts/acm-first-contest.md
pnpm dev                          # 预览：http://localhost:4321 ，改文件自动刷新
# 写完后 Ctrl+C 停掉 dev
pnpm build                        # 可选，但建议跑一次确认能构建成功
git add -A
git commit -m "post: 第一次 ACM 比赛"
git push                          # 推上去后 GitHub Actions 自动构建并发布，约 1 分钟
```

也可以不用脚本，直接在 `src/content/posts/` 里新建 `.md` 文件，把下面这段 frontmatter 粘到最上面。

### 文件名就是网址

文件 `acm-first-contest.md` → 网址 `https://xiaocrq.github.io/posts/acm-first-contest/`

- 用英文小写 + 连字符，别用中文和空格（中文文件名会导致链接难看的编码）
- **上线后不要改文件名**，否则旧链接全部 404
- 子目录也会保留在网址里：`posts/2026/acm.md` → `/posts/2026/acm/`

## 四、frontmatter 字段

```markdown
---
title: 第一次参加 ACM 比赛
published: 2026-11-18
updated: 2026-11-20
description: 记录第一次参加 ACM 校赛的过程和踩过的坑。
image: ./cover.jpg
tags: [ACM, C++, 比赛]
category: 比赛记录
draft: false
lang: ''
---
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `title` | 是 | 文章标题 |
| `published` | 是 | 发布日期，格式必须 `YYYY-MM-DD`（写错构建会失败） |
| `updated` | 否 | 更新时间，有的话文章页会显示"已更新" |
| `description` | 否 | 摘要，用于列表页和分享卡片；留空则自动取正文第一段 |
| `image` | 否 | 封面图，相对路径 `./cover.jpg`（图放在同目录）或以 `/` 开头的 `public/` 路径 |
| `tags` | 否 | 标签数组，显示在文章里，可点击筛选 |
| `category` | 否 | 分类，单个字符串 |
| `draft` | 否 | `true` 时**不会**被构建出来，等于不发布；本地 dev 也不显示 |
| `lang` | 否 | 留空即使用站点语言（`zh_CN`） |

## 五、写作语法

### 基础

普通 Markdown：`#` 一级标题（正文里一般从 `##` 开始）、`**粗体**`、`*斜体*`、`` `行内代码` ``、`[链接](https://example.com)`、`> 引用`、`- 列表`、`1. 有序列表`、`---` 分隔线。

标题会自动生成右侧目录（TOC），只收集 `##` / `###`（`src/config.ts` 里 `toc.depth: 2`）。

### 代码块

````markdown
```cpp title="main.cpp"
#include <bits/stdc++.h>
using namespace std;
int main() { return 0; }
```
````

- 语言名会自动显示成右上角的小徽章
- `title="main.cpp"`：显示文件名/标题；`frame="none"` 去掉外框，`frame="code"` 强制编辑器样式
- 加 `showLineNumbers`：显示行号
- 加 `{1,4,7-8}`：高亮这些行；`ins={3}` 标记新增行（绿色），`del={2}` 标记删除行（红色）
- 加 `collapse={1-5}`：把第 1–5 行折叠起来，点开才显示

### 数学公式

行内：`$O(n \log n)$` → $O(n \log n)$

块级：

```markdown
$$
T(n) = 2T(n/2) + O(n)
$$
```

### 提示框

支持 `note` `tip` `important` `warning` `caution` 五种：

```markdown
:::tip[小技巧]
可以自定义标题。
:::
```

也支持 GitHub 原生写法：

```markdown
> [!NOTE]
> 这样写也行。
```

### 其他

- GitHub 仓库卡片：`::github{repo="XiaoCRQ/XiaoCRQ"}`（加载时从 GitHub API 拉信息）
- 折叠文字：`:spoiler[这里藏着一句话]`
- 图片：`![图片说明](./cover.jpg)`（放在文章同目录）或 `![图片说明](/images/xxx.png)`（放在 `public/images/`）
- 视频：直接从 B 站/YouTube 复制 iframe 嵌入代码粘进来即可

```html
<iframe width="100%" height="468" src="//player.bilibili.com/player.html?bvid=BV1fK4y1s7Qf&p=1" frameborder="0" allowfullscreen="true"></iframe>
```

## 六、改站点信息

全都在 `src/config.ts`：

| 想改什么 | 改哪个 |
|---|---|
| 浏览器标签页 / 左上角标题 | `siteConfig.title`（现在是 `XiaoCRQ`） |
| 标题下面那行小字 | `siteConfig.subtitle` |
| 界面语言 | `siteConfig.lang` |
| 主题色 | `siteConfig.themeColor.hue`（0 红 / 200 青 / 250 蓝 / 345 粉） |
| 顶部导航 | `navBarConfig.links` |
| 侧边栏头像、名字、简介 | `profileConfig` |
| 侧边栏社交图标 | `profileConfig.links`（图标名去 https://icones.js.org/ 查） |
| 文章版权声明 | `licenseConfig` |

改完 `pnpm dev` 看效果，满意就 commit + push。

## 七、备忘录 / 踩过的坑

- **删了文章但构建还在生成旧页面**：Astro 的内容缓存不在 `.astro/`，而在 `node_modules/.astro/data-store.json`。清缓存：
  ```bash
  mv node_modules/.astro /tmp/astro-cache-$(date +%s)
  ```
  然后重新 `pnpm build`。换电脑/CI 上不会遇到，只有本地删文章才会。
- **`The collection "posts" does not exist or is empty`**：文章全删空后构建会刷这行警告，只是警告，构建仍然成功（exit 0），不用管。`src/content/posts/.gitkeep` 是用来保住这个空目录的，别删。
- **中文搜索搜不全**：Pagefind 对 `zh-cn` 不做词干处理（构建日志里的 `doesn't support stemming`），所以搜"比赛"不会命中"比赛的"。属于正常现象。
- **网址一律带尾斜杠**：`/posts/xxx/` 能开，`/posts/xxx` 会 404。这是 `trailingSlash: "always"` 决定的，分享链接时注意。
- **推送只有你能做**：这台机器上 `git push` 会因为 SSH key 未授权而失败（`Permission denied (publickey)`）。要用 GitHub 上已配置的 SSH key 或令牌。AI 助手只能改代码，推送要你自己执行。
- **部署状态在哪看**：https://github.com/XiaoCRQ/XiaoCRQ.github.io/actions —— 绿色 = 已上线，红色 = 构建失败，点进去能看到具体报错。
- **想回滚到旧首页**：Settings → Pages → Source 改回 `Deploy from a branch` + 选 `main_back`。
- **`pnpm install` 报 `only-allow` 错误**：说明你用了 npm/yarn，换回 `pnpm install`。
- **依赖装不上/很奇怪**：先 `pnpm install`，别手动改 `pnpm-lock.yaml`。

## 八、待办 / 想法

- [ ] 换掉模板自带的 `src/assets/images/demo-banner.png`（898KB）
- [ ] 要不要加评论系统（fuwari 本身不带）
- [ ] 要不要绑自定义域名
- [ ] 指定 favicon（`src/config.ts` 的 `siteConfig.favicon`）
