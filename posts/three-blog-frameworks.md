---
title: 用三个静态站点框架搭建博客的完整记录
date: 2026-05-24
author: xufanchn
tags: [博客, VitePress, Hugo, Hexo, GitHub Actions, GitHub Pages]
description: 从需求分析到方案设计再到最终实现，完整记录用 VitePress、Hugo、Hexo 三个框架搭建博客的过程，以及如何用一个仓库统一管理所有文章。
---

# 用三个静态站点框架搭建博客的完整记录

## 起因

一直想搭建个人技术博客，但在选择框架时犯了难。目前主流的静态站点生成器各有特色：

- **VitePress**：Vue 生态，Markdown 优先，配置简洁，构建飞速
- **Hugo**：Go 编写，构建极快，主题丰富，单二进制无依赖
- **Hexo**：Node.js 生态，插件多，中文社区活跃，入门友好

与其纠结选哪个，不如三个都搭一遍。正好对比一下各自的开发体验、构建速度、部署流程。

## 需求梳理

在动手之前，先明确要解决的问题：

1. **零本地依赖**：不在本地安装任何框架 CLI，全部通过 GitHub Actions 构建
2. **内容统一管理**：三个博客内容相同，不能每次更新都要改三个仓库
3. **自动部署**：推送文章后自动更新所有站点
4. **可对比**：同一个主题、同一篇文章，方便对比框架差异

## 方案设计

核心思路是**内容与站点分离**：

```
blog-content/                 ← 唯一的文章仓库（你只操作这个）
  └── posts/
       ├── webterm-intro.md
       └── three-blog-frameworks.md

         │  每 15 分钟自动同步
         ▼
  ┌──────────┬──────────┬──────────┐
  │ Vitepress│   Hugo   │   Hexo   │
  │ 工作流拉取│ 工作流拉取│ 工作流拉取│
  │ 构建部署  │ 构建部署  │ 构建部署  │
  └──────────┴──────────┴──────────┘
```

每个博客站点的 GitHub Actions 工作流做了三件事：

1. **Checkout 博客仓库**：拉取站点自身的配置和主题
2. **Checkout blog-content**：拉取共享的文章内容
3. **复制 + 构建 + 部署**：把文章复制到框架要求的目录，构建后部署到 GitHub Pages

这样只需往 `blog-content` 推送一次文章，三个站点就能自动同步更新。

## 实现过程

### 1. 创建仓库

在 GitHub 上创建四个仓库：

| 仓库 | 用途 |
|---|---|
| `blog-content` | 文章内容，唯一需要手动操作的仓库 |
| `blog-vitepress` | VitePress 站点 |
| `blog-hugo` | Hugo 站点 |
| `blog-hexo` | Hexo 站点 |

### 2. 搭建各框架站点

三个框架的搭建过程各有特点。

**VitePress** 最简洁，`package.json` 加一个依赖就能跑：

```json
{
  "scripts": {
    "dev": "vitepress dev",
    "build": "vitepress build"
  },
  "dependencies": {
    "vitepress": "^1.6.4"
  }
}
```

配置文件 `config.mts` 也很直观，导航栏、侧边栏、搜索、暗色模式都是声明式配置。VitePress 本身偏向文档站，改造成博客风格需要手动调整首页布局。

**Hugo** 通过 `hugo new site` 初始化，选择 PaperMod 主题。Hugo 的配置用 TOML 格式，主题通过 Git Submodule 引入。文章存放在 `content/posts/`，URL 路径由文件名自动生成。Hugo 的构建速度是三家中最快的——因为它是 Go 写的静态二进制。

**Hexo** 通过 `hexo init` 初始化，默认 landscape 主题。配置用 YAML，文章放在 `source/_posts/`。Hexo 生态成熟，插件和主题数量最多，但 Node.js 依赖也最重。

### 3. 配置 GitHub Actions

每个博客的部署流程本质相同，差异只在构建命令和输出目录：

```yaml
# 核心步骤（以 Hugo 为例）
- name: Checkout blog-content
  uses: actions/checkout@v4
  with:
    repository: xufanchn/blog-content
    path: blog-content

- name: Sync posts
  run: |
    rm -rf content/posts/*
    cp blog-content/posts/*.md content/posts/

- name: Build
  run: hugo --minify
```

触发条件设置了三种：
- `push`：推送站点配置变更时立即构建
- `schedule`：每 15 分钟定时检查 blog-content 是否有新文章
- `workflow_dispatch`：手动触发，着急的时候用

### 4. 启用 GitHub Pages

通过 GitHub API 为三个仓库启用 Pages，选择 "GitHub Actions" 作为部署源：

```bash
gh api repos/xufanchn/blog-vitepress/pages \
  -X POST -F "build_type=workflow"
```

### 5. 文章格式统一

为了让同一份 `.md` 文件在三个框架中都能正确渲染，使用 YAML frontmatter 格式：

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

VitePress 和 Hexo 原生支持 YAML frontmatter，Hugo 也支持（默认是 TOML，但同样能解析 YAML 包裹的 `---` 块）。

## 遇到的问题

### GitHub Pages 部署 404

首次启用 Pages 后立即触发 workflow，deploy 步骤报 404。原因是 Pages 配置还没完全生效，workflow 就跑完了。解决方法是等待几秒后重跑 workflow。

### 网络超时

GitHub API 偶尔 TLS 握手超时，属于网络波动，不影响最终结果。

### 框架差异

同一个 frontmatter 字段在不同框架中的行为略有不同：
- `date` 字段：VitePress 仅用于排序，Hugo 需要严格日期格式，Hexo 使用 Moment.js 解析
- `tags` 字段：Hugo 展示在文章底部，Hexo 生成标签云，VitePress 默认不展示

这些差异恰好体现了框架设计理念的不同，也是做这个对比的意义所在。

## 最终效果

- **写文章**：在 `blog-content` 仓库的 `posts/` 下新建 `.md` 文件，推送即可
- **自动部署**：15 分钟内三个站点自动更新，也可手动触发立即部署
- **零本地依赖**：全程不需要安装 Node.js、Hugo、Hexo

三个站点的对比：

| 维度 | VitePress | Hugo + PaperMod | Hexo + Landscape |
|---|---|---|---|
| 构建速度 | ~3s | ~1s | ~5s |
| 配置复杂度 | 低 | 中 | 中 |
| 主题定制 | JS/TS 编程 | TOML + HTML 模板 | YAML + EJS 模板 |
| 搜索 | 内置本地搜索 | 需额外配置 | 需插件 |
| 暗色模式 | 内置 | 内置 | 需插件 |
| 中文支持 | 好 | 好 | 好 |

## 总结

用三个框架搭建同一个博客的过程，本身就是一次很好的学习体验。最终的架构做到了"只维护一个仓库，同步三个站点"的目标。

如果你也在纠结选哪个框架，不妨也这样搭一遍——实际用过之后的感受，比看再多的对比文章都直观。

---

- **博客内容仓库**：[blog-content](https://github.com/xufanchn/blog-content)
- **VitePress 站点**：[xufanchn.github.io/blog-vitepress](https://xufanchn.github.io/blog-vitepress/)
- **Hugo 站点**：[xufanchn.github.io/blog-hugo](https://xufanchn.github.io/blog-hugo/)
- **Hexo 站点**：[xufanchn.github.io/blog-hexo](https://xufanchn.github.io/blog-hexo/)
