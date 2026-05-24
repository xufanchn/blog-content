# Blog Content

博客文章的统一存储仓库。推送 `.md` 到 `posts/` 后自动同步到博客站点。

**同步站点：**
- [xf / blog](https://xufanchn.github.io/blog/) — 个人博客（Hugo）

## 使用方法

1. 在 `posts/` 目录下新建 Markdown 文件，使用 YAML frontmatter：

```markdown
---
title: 文章标题
date: 2026-05-24
author: xufanchn
tags: [标签1, 标签2]
description: 文章摘要
---

正文内容...
```

2. 推送后，博客站点每 5 分钟自动同步更新

3. 如需立即更新，手动触发博客的 `Deploy` Action

## 相关项目

- [webterm-docs](https://github.com/xufanchn/webterm-docs) — WebTerm 产品文档（独立维护）
