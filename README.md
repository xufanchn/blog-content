# Blog Content

所有博客文章的统一存储仓库。推送 `.md` 文件到 `posts/` 目录后，三个博客站点会自动同步更新。

**支持的博客站点：**
- [VitePress](https://xufanchn.github.io/blog-vitepress/)
- [Hugo](https://xufanchn.github.io/blog-hugo/)
- [Hexo](https://xufanchn.github.io/blog-hexo/)

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

2. 推送后，三个博客站点会在 15 分钟内自动更新

3. 也可以手动触发各博客站点的 `Deploy` Action 立即更新
