---
title: 你好，世界
description: 这是本站的第一篇文章，用来验证站点和文章列表正常工作。
pubDatetime: 2026-10-23T00:00:00Z
featured: false
draft: true
tags:
  - 随笔
---

欢迎来到我的个人网站。

这篇文章是初始化时创建的示例，可以直接删除，也可以把它改成你的第一篇文章。

## 如何添加新文章

在 `src/content/posts/` 目录下新建一个 `.md` 文件即可，例如 `my-post.md`，站点会自动生成 `/posts/my-post` 页面。

文章开头的 frontmatter 至少需要 `title`、`description`、`pubDatetime` 三个字段：

```yaml
---
title: 文章标题
description: 文章摘要，会显示在列表和搜索结果中
pubDatetime: 2026-10-23T10:00:00Z
tags:
  - 标签
---
```

> [!TIP]
> 不需要在正文里再写一级标题（`#`），页面标题会自动使用 frontmatter 中的 `title`。正文从 `##` 开始。

更多说明见 `docs/project-structure.md`。