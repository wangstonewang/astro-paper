# AstroPaper 项目结构与内容管理

> 适用于本仓库（AstroPaper v6，Astro 7 + Tailwind CSS 4）。
> 整理时间：2026-10-23。

## 1. 模板概览

这是一个静态博客模板。核心思路是：**内容（Markdown 文件）驱动网站页面的静态生成**，`npm run build` 后输出纯静态 HTML。

架构分为两层：

- **内容层**：`src/content/` 下的 Markdown / MDX 文件 → 每篇文章对应一个页面
- **表现层**：`src/pages/` 负责路由，`src/layouts/` + `src/components/` 负责 UI

## 2. 目录结构

```text
astropaper/
├── astro-paper.config.ts   ★ 用户配置（站点名、作者、社交链接、分页数等）
├── astro.config.ts            Astro 底层配置（插件、Markdown 渲染、字体）
├── package.json               npm 脚本和依赖
├── public/                   静态资源原样发布（favicon、默认 OG 图）
│   ├── favicon.svg
│   └── default-og.jpg
└── src/
    ├── content/              ★★ 你的内容都在这里
    │   ├── posts/              所有博客文章（.md / .mdx）
    │   └── pages/about.md      "关于"页面的内容
    ├── pages/                  路由（文件路径 = URL 路径）
    ├── layouts/                页面骨架（Layout / PostLayout）
    ├── components/             可复用 UI（Header、Footer、Card、Tag…）
    ├── assets/                 需要构建优化的图片、图标
    │   ├── images/
    │   └── icons/
    ├── styles/                 Tailwind、主题色、排版样式
    ├── utils/                  工具函数（排序、过滤草稿、取标签…）
    ├── i18n/                   界面文案（当前只有 en）
    ├── types/config.ts         配置的类型定义
    ├── config.ts               内部配置合并（不建议直接改）
    └── content.config.ts       内容集合的 schema（frontmatter 字段校验）
```

## 3. 关键目录说明

| 目录 / 文件 | 作用 |
|---|---|
| `src/content/posts/` | **放文章的地方**。可以创建子目录分类，子目录名会进入 URL |
| `src/content/pages/about.md` | 关于页正文，改这里就能替换模板介绍 |
| `src/pages/` | 页面路由：`index.astro`→首页、`about.astro`→/about、`posts/[...page].astro`→文章列表分页、`posts/[...slug]/index.astro`→单篇文章、`tags/`→标签页、`archives/`→归档、`search.astro`→搜索、`rss.xml.ts`→RSS、`robots.txt.ts`→robots |
| `src/layouts/` | `Layout.astro` 是全站骨架（head/SEO/主题切换），`PostLayout.astro` 是文章页骨架 |
| `src/components/` | 头部导航、卡片、分页、标签、分享按钮等组件 |
| `src/assets/` | 会被 Astro 处理和哈希化命名的图片、图标，适合放正文图片 |
| `public/` | 不经过处理、原样复制的内容，适合 favicon、robots 附加文件等 |
| `src/styles/theme.css` | 站点主色调定义（CSS 变量） |
| `astro-paper.config.ts` | **最常改的配置文件**：站点 URL、标题、作者、每页文章数、社交链接、编辑链接等 |
| `src/content.config.ts` | 校验文章 frontmatter；想加自定义字段也在这里加 |
| `.github/` | GitHub Actions（自动部署流程参考） |

模板自带的 `how-to-*.md`、`examples/`、`_releases/`、`_color-schemes/` 都是**示例内容**，正式上线时应删除或替换。

以 `_` 开头的目录和文件不会生成路由。

## 4. 如何添加自己的文章

### 4.1 新建文件

在 `src/content/posts/` 下新建 `.md` 或 `.mdx` 文件：

```bash
src/content/posts/my-first-post.md        →  /posts/my-first-post
src/content/posts/2026/my-post.md         →  /posts/2026/my-post
src/content/posts/_drafts/idea.md         →  不生成页面（下划线开头）
```

> `_` 开头的文件和目录会被当成"私有资源"：不生成路由，可用来放草稿或共用图片。

### 4.2 frontmatter（文件头部的元信息）

只有 **`title`、`description`、`pubDatetime`** 三项必填：

```yaml
---
title: 我的第一篇文章
description: 用一句话概括这篇文章（用于列表摘要和 SEO）
pubDatetime: 2026-10-22T10:00:00Z
featured: true            # 是否显示在首页推荐区
draft: false              # true = 草稿，构建时不显示
tags:
  - 随笔
  - Astro
ogImage: ./assets/cover.png   # 可选，相对当前 md 文件
---
```

常用可选字段：

| 字段 | 说明 |
|---|---|
| `author` | 作者，默认取站点作者 |
| `modDatetime` | 修改时间 |
| `canonicalURL` | 原文地址（文章同时发布在别处时使用） |
| `timezone` | 单篇文章的时区，优先级高于全局配置 |
| `hideEditPost` | 隐藏该文章的"编辑此文"按钮 |

完整字段表见模板文章 `src/content/posts/adding-new-post.mdx`。

### 4.3 正文写作约定

- **不要再写 `#` 一级标题**：标题由 frontmatter 的 `title` 渲染，正文从 `##` 开始。
- 需要目录：在正文里写一个 `## Table of contents`，会自动生成可折叠目录。
- 提示框（callout）：

  ```md
  > [!TIP]
  > 这是一条提示。
  ```

  支持的类型：`NOTE`、`ABSTRACT`、`INFO`、`TODO`、`TIP`、`SUCCESS`、`QUESTION`、`WARNING`、`FAILURE`、`DANGER`、`BUG`、`EXAMPLE`、`QUOTE`。
- `.mdx` 文件可以在文章中直接引入组件、使用 JSX 语法；纯写文章用 `.md` 就够了。
- 草稿：`draft: true`，或者放到 `_` 开头的目录里。

### 4.4 本地预览

```bash
npm run dev       # 开发服务器，默认 http://localhost:4321
npm run build     # 类型检查 + 构建 + 生成搜索索引（Pagefind）
npm run preview   # 预览构建结果（搜索功能只有 build 后才能用）
```

## 5. 如何添加图片

按用途二选一：

### A. 正文配图 → `src/assets/images/`（推荐）

图片会被 Astro 压缩、优化并生成哈希文件名。在 Markdown 里用**相对路径**引用：

```md
![图片说明](../../assets/images/my-photo.jpg)
```

或者在文章目录里建一个 `assets/` 子目录与 md 放在一起：

```md
![图片说明](./assets/my-photo.jpg)
```

### B. 固定路径资源 → `public/`

图片原样复制、不做优化，用绝对路径引用：

```md
![图片说明](/my-photo.png)
```

适合 favicon、logo、需要固定 URL 的文件。

### 社交分享图（OG image）

frontmatter 里的 `ogImage` 可以写本地相对路径或完整 URL；不写会自动生成（对应 `features.dynamicOgImage: true`）。

## 6. 正式做个人站前建议修改的几处

1. **`astro-paper.config.ts`**
   - `site.url`、`title`、`description`、`author`、`profile`
   - `lang` 改成 `"zh-CN"`，`timezone` 改成 `"Asia/Shanghai"`
   - `socials` 换成自己的 GitHub / X / 邮箱
   - `editPost.enabled` 改成 `false`（或指向自己的仓库）
   - `posts.perPage` / `perIndex`：每页文章数
2. **`src/content/pages/about.md`** → 写自我介绍
3. **删除示例文章**：`src/content/posts/` 下除自己文章外的 `.md/.mdx`，以及 `_releases/`、`_color-schemes/`、`examples/`
4. **换 favicon / 默认分享图**：`public/favicon.svg`、`public/default-og.jpg`
5. **主题色**（可选）：`src/styles/theme.css`

> 关于删除示例文章对后续模板更新的影响，见 `docs/updating-template.md`。