# Guts' Blog · Hexo 使用指南

这是位于 `D:\MyBlog` 的 Hexo 博客项目。本文依据 [Hexo 官方文档](https://hexo.io/zh-cn/docs/) 整理，并结合本项目配置编写，所有终端命令均适用于 **Windows CMD**。

- 博客地址：[https://zooeyq.github.io](https://zooeyq.github.io/)
- 源码仓库：[zooeyq/zooeyq.github.io](https://github.com/zooeyq/zooeyq.github.io)
- 框架：Hexo 8；主题：Landscape。
- 发布方式：推送源码到 `main`，由 GitHub Actions 生成网站并部署。
- 文档核对日期：2026-10-19。后续修改配置时，请同步更新本说明。

## 目录

- [1. 环境准备](#1-环境准备)
- [2. 项目目录](#2-项目目录)
- [3. 从草稿开始写作](#3-从草稿开始写作)
- [4. 文章信息与正文](#4-文章信息与正文)
- [5. 图片与资源文件夹](#5-图片与资源文件夹)
- [6. 本地预览与构建](#6-本地预览与构建)
- [7. 草稿转为正式文章](#7-草稿转为正式文章)
- [8. 发布到 GitHub Pages](#8-发布到-github-pages)
- [9. 修改文章与站点配置](#9-修改文章与站点配置)
- [10. 常见问题](#10-常见问题)
- [11. 常用命令速查](#11-常用命令速查)
- [12. 官方参考资料](#12-官方参考资料)

## 1. 环境准备

本项目已初始化，不需要再次运行 `hexo init`。每次打开 CMD 后，先进入项目目录：

```cmd
cd /d D:\MyBlog
```

`/d` 用于同时切换盘符和目录。

检查环境：

```cmd
node -v
npm -v
git --version
```

Hexo 8 要求 Node.js 至少为 **20.19.0**；本项目的自动构建使用 Node.js 22，本地可选用同一主版本。具体要求见 [官方安装文档的版本限制](https://hexo.io/zh-cn/docs/#Node-js-版本限制)。

首次克隆项目，或本地尚未安装依赖时执行：

```cmd
npm ci
```

已有正常可用的 `node_modules` 时，不必每次重复安装。`npm ci` 会按 `package-lock.json` 安装，并重建依赖目录。

本文使用 `npx hexo` 调用项目依赖中的 Hexo，无需额外全局安装。可查看版本：

```cmd
npx hexo version
```

## 2. 项目目录

```text
D:\MyBlog\
├── README.md                 # 本使用指南
├── _config.yml               # 网站主配置
├── _config.landscape.yml     # Landscape 主题配置覆盖
├── package.json              # 依赖和快捷命令
├── package-lock.json         # 依赖版本锁定文件
├── scaffolds/                # 新建文章、草稿、页面的模板
├── source/
│   ├── _drafts/              # 草稿
│   ├── _posts/               # 正式文章及其资源目录
│   └── images/               # 可自行创建，存放公共图片
├── themes/                   # 本地主题目录；主题也可作为依赖安装
├── public/                   # 生成的静态网站，不直接编辑
├── node_modules/             # 安装的依赖，不直接编辑
└── .github/workflows/
    └── pages.yml             # 自动构建和部署配置
```

写文章修改 `source`，调整网站修改配置文件。不要直接修改 `public` 中的网页，下次生成时会被覆盖。

## 3. 从草稿开始写作

### 新建草稿

下面用 `my-new-post` 作为新文章的文件名示例，可替换成自己的英文名称：

```cmd
cd /d D:\MyBlog
npx hexo new draft "my-new-post"
```

本项目启用了 `post_asset_folder: true`，通过 Hexo 新建时会同时创建：

```text
source/_drafts/
├── my-new-post.md
└── my-new-post/
```

Markdown 文件用于写作，同名文件夹用于图片等资源。空文件夹通常不会被 Git 单独记录。

用编辑器打开 `.md` 写内容并保存。建议文件名使用英文小写和短横线，文首的 `title` 可以写中文。已有同名草稿时，直接编辑原文件，不要重复新建或使用覆盖选项。

当前 Maven 草稿已经存在，文件为 `source/_drafts/maven-learning-notes.md`，无需重新创建。

### 其他创建方式

直接创建正式文章：

```cmd
npx hexo new post "another-post"
```

创建独立页面，例如“关于”：

```cmd
npx hexo new page "about"
```

页面通常位于 `source/about/index.md`，访问路径为 `/about/`。是否显示在导航菜单中，需要在主题配置中另行设置。

官方说明：[写作](https://hexo.io/zh-cn/docs/writing)。

## 4. 文章信息与正文

### Front-matter

文章最上面的两行 `---` 之间是 YAML 格式的文章信息，例如：

```yaml
---
title: "我的 Maven 学习笔记"
date: 2026-10-19 23:47:00
categories:
  - 技术笔记
tags:
  - Java
  - Maven
---
```

| 字段 | 用途 |
| --- | --- |
| `title` | 页面显示的标题 |
| `date` | 文章日期，影响排序及本项目的日期路径 |
| `updated` | 可选，明确指定更新时间 |
| `categories` | 分类；普通列表中的多个项目表示逐级分类 |
| `tags` | 标签列表，通常是并列关系 |
| `permalink` | 可选，单独指定文章链接 |

YAML 使用空格缩进，不要使用 Tab；同级标签的缩进应一致。标题带有冒号等符号时，建议加引号。

本项目当前的 `scaffolds/draft.md` 没有日期字段，新建草稿后可手动补充 `date`。以后想自动包含日期，可以编辑该模板加入 `date: {{ date }}`；这只影响后续新建的文件。

字段说明：[Front-matter](https://hexo.io/zh-cn/docs/front-matter)。

### Markdown 写作示例

````markdown
这里写开头的简短介绍。

<!-- more -->

## 第一个小节

普通正文，**重点内容**，以及 `行内代码`。

- 一个要点
- 另一个要点

[Hexo 官网](https://hexo.io/)

```java
System.out.println("Hello, Hexo!");
```
````

`<!-- more -->` 标记首页摘要的截断位置，具体展示由主题决定；正文仍会完整显示。文章标题由 `title` 提供，正文一般从二级标题 `##` 开始即可。Hexo 的额外语法见 [标签插件](https://hexo.io/zh-cn/docs/tag-plugins)。

## 5. 图片与资源文件夹

### 每篇文章自己的图片

例如把图片放到：

```text
source/_drafts/my-new-post/maven-flow.png
```

在该文章中引用：

```text
{% asset_img maven-flow.png Maven依赖解析示意图 %}
```

这是 Hexo 的资源标签，可以处理文章页、首页摘要等位置的图片路径。文件名建议使用英文，避免空格；它需要通过 Hexo 渲染，普通 Markdown 编辑器未必能直接预览。

不要把 `D:\MyBlog\...` 这样的本机绝对路径写进博客图片链接，访问线上网站的人无法读取你的磁盘。

### 多篇文章共用的图片

公共图片可放在 `source/images/`，例如 `source/images/logo.png`，引用方式为：

```markdown
![博客标志](/images/logo.png)
```

上面的根路径写法适用于本站当前部署在域名根目录的情况。

### 使用普通 Markdown 引用文章图片

官方也提供 `hexo-renderer-marked` 的资源路径处理选项：

```yaml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

当前项目没有显式配置这一组 `marked` 选项，建议先使用 `asset_img`。若以后启用，合并到现有配置中，检查文章页和首页后再发布。

官方说明：[资源文件夹](https://hexo.io/zh-cn/docs/asset-folders)。

## 6. 本地预览与构建

### 包含草稿的预览

```cmd
cd /d D:\MyBlog
npx hexo server --draft
```

打开 [http://localhost:4000/](http://localhost:4000/)，从首页点击文章。保持 CMD 窗口运行，编辑保存后刷新浏览器查看变化，结束时按 `Ctrl+C`。

### 只预览正式内容

```cmd
npx hexo server
```

本项目 `render_drafts: false`，默认不显示草稿。`--draft` 是临时预览参数，通常不需要为了本地查看而修改全站设置。

### 端口被占用

换一个端口：

```cmd
npx hexo server --draft -p 4001
```

然后访问 [http://localhost:4001/](http://localhost:4001/)。本地预览服务不是线上部署。官方说明：[服务器](https://hexo.io/zh-cn/docs/server)。

### 生成和清理

生成静态网站：

```cmd
npm run build
```

输出在 `public`。遇到旧页面残留、重命名文章或刚检查过草稿构建时，可停止预览服务，再执行：

```cmd
npm run clean
npm run build
```

`clean` 清除 `public` 和 `db.json`，不会删除文章源文件。不要手动编辑生成文件。命令定义见 [Hexo 指令](https://hexo.io/zh-cn/docs/commands)。

## 7. 草稿转为正式文章

检查标题、日期、正文、代码和图片后，停止预览服务，再执行：

```cmd
npx hexo publish my-new-post
```

参数使用草稿文件名，不带 `.md`，不要换成修改后的中文显示标题。

Hexo 会将文章移至 `source/_posts/`，当前资源配置下也会处理同名资源目录。完成后检查文件位置和图片：

```text
source/_posts/
├── my-new-post.md
└── my-new-post/
    └── maven-flow.png
```

再运行普通预览，确认文章出现：

```cmd
npm run clean
npm run build
npx hexo server
```

`hexo publish` 只在本地将草稿转为正式文章，**不会上传到 GitHub，也不会直接更新线上网站**。规则见 [官方草稿说明](https://hexo.io/zh-cn/docs/writing#草稿)。

## 8. 发布到 GitHub Pages

### 本项目的实际流程

```text
修改文章或配置 → 本地检查 → Git 提交 → 推送 main
    → GitHub Actions 构建 → 部署 GitHub Pages
```

根据 `.github/workflows/pages.yml`，推送到 `main` 后，工作流会使用 Node.js 22，执行 `npm ci` 和 `npm run build`，上传 `public` 并部署。

`_config.yml` 中的 `deploy.type` 为空，因此本项目虽然有 `npm run deploy` 快捷命令，但尚未配置 Hexo 的一键部署器。日常上线按下面的 Git 推送流程操作即可。

### 提交前检查

```cmd
cd /d D:\MyBlog
git status --short
git branch --show-current
git diff
```

确认当前分支为 `main`，并检查要提交的变更。若分支或远程状态不符合预期，先处理对应问题。

### 暂存、提交与推送

以下以 `my-new-post` 为例，只暂存准备发布的文章：

```cmd
git add -- source/_posts/my-new-post.md
```

如果资源目录中有图片等文件，同时暂存：

```cmd
git add -- source/_posts/my-new-post/
```

空资源目录不会被 Git 跟踪，可跳过上一步。如果草稿以前提交过，则发布时旧路径的删除也需要暂存。先查看：

```cmd
git ls-files -- source/_drafts/my-new-post.md
```

只有输出该路径时，再运行：

```cmd
git add -u -- source/_drafts/my-new-post.md
```

若旧草稿资源也曾被跟踪，同样暂存旧资源路径的删除：

```cmd
git add -u -- source/_drafts/my-new-post/
```

确认暂存区只包含本次计划提交的内容：

```cmd
git diff --cached --stat
git diff --cached
git commit -m "docs: publish my-new-post"
git push origin main
```

修改配置或 README 时，按实际需要额外暂存对应文件。`public`、`node_modules` 和 `db.json` 已被忽略，无需提交。

### 确认上线结果

1. 打开 [GitHub Actions](https://github.com/zooeyq/zooeyq.github.io/actions)。
2. 等待 `Deploy Hexo to GitHub Pages` 成功完成。
3. 打开 [博客](https://zooeyq.github.io/)，检查文章和图片。

如果部署失败，查看工作流日志；仓库 Settings → Pages → Build and deployment → Source 应选择 **GitHub Actions**。这是远端设置要求，本说明未核查当前远端设置。配置方法参见 [Hexo 官方 GitHub Pages 指南](https://hexo.io/zh-cn/docs/github-pages)。

草稿不会出现在普通网站构建中；但如果将草稿源码提交到公开仓库，别人仍可从仓库读取。不要把需要保密的内容提交到公开源码仓库。

## 9. 修改文章与站点配置

### 修改已经发布的文章

直接编辑 `source/_posts/` 内的原文章，保存后本地预览、构建、提交并推送即可，不必再次 `hexo publish`。

本项目使用 `:year/:month/:day/:title/` 生成文章路径。显示标题通常来自 `title` 字段，而路径中的 `:title` 通常来自文章文件名对应的 slug。发布后修改日期或文件名可能改变链接，操作前应考虑旧链接；也可以使用文章的 `permalink` 固定地址。参见 [永久链接](https://hexo.io/zh-cn/docs/permalinks)。

### 常用站点配置

编辑项目根目录的 `_config.yml`：

| 当前设置 | 作用 |
| --- | --- |
| `title: Guts' Blog` | 博客名称 |
| `author: Guts` | 作者 |
| `language: zh-CN` | 站点语言 |
| `timezone: Asia/Shanghai` | 时区 |
| `url: https://zooeyq.github.io` | 线上网址 |
| `theme: landscape` | 当前主题 |
| `post_asset_folder: true` | 新建文章时创建资源目录 |
| `render_drafts: false` | 默认不生成草稿 |
| `future: true` | 允许生成日期在未来的正式文章 |

主题外观和菜单等优先在 `_config.landscape.yml` 中配置，选项以主题支持的内容为准。修改配置后重启预览服务，必要时清理再生成。详情见 [配置文档](https://hexo.io/zh-cn/docs/configuration)。

### 自定义新文章模板

`scaffolds/post.md`、`scaffolds/draft.md`、`scaffolds/page.md` 分别用于新建正式文章、草稿和页面。模板可使用 `{{ title }}`、`{{ date }}` 等变量。例如，希望新草稿自动带日期和分类占位，可以自行将 `scaffolds/draft.md` 调整为：

```yaml
---
title: {{ title }}
date: {{ date }}
categories:
tags:
---
```

本指南仅展示修改方式，没有替你更改模板。已有文章也不会因模板更新而自动改变。

## 10. 常见问题

| 现象 | 检查与处理 |
| --- | --- |
| CMD 提示找不到 `node`、`npm` 或 `git` | 检查安装及 Path，重新打开 CMD |
| 找不到 Hexo 或依赖 | 确认位于 `D:\MyBlog`，依赖缺失时运行 `npm ci` |
| 草稿不显示 | 使用 `npx hexo server --draft`，检查文件是否位于 `_drafts` |
| 没有同名资源文件夹 | 检查 `post_asset_folder`；手动写入 `.md` 不会触发新建命令，可补建同名目录 |
| 图片在文章页正常、首页不正常 | 优先使用 `asset_img`，检查资源文件位置 |
| 本地正常、线上图片缺失 | 检查图片是否已提交，以及文件名大小写是否完全一致；线上构建使用 Linux |
| 4000 端口被占用 | 停止旧服务或改用 `-p 4001` |
| 修改配置后没变化 | 停止服务，清理后重新启动，刷新浏览器 |
| YAML 报错 | 检查缩进、英文冒号及其后的空格、引号和列表层级 |
| `hexo publish` 后线上没变化 | 还需要提交、推送并等待 Actions 完成 |
| 推送被拒绝 | 阅读 Git 错误，确认认证或远程是否有新提交；先同步并解决冲突，不要直接强制推送 |
| 正常构建还残留旧文章 | 执行 `npm run clean` 再 `npm run build` |

需要更多诊断信息时可运行 `npx hexo generate --debug`，并参阅 [Hexo 官方问题解答](https://hexo.io/zh-cn/docs/troubleshooting)。

## 11. 常用命令速查

先运行 `cd /d D:\MyBlog`，再按需要选择命令：

| 目的 | CMD 命令 |
| --- | --- |
| 新建草稿 | `npx hexo new draft "my-new-post"` |
| 新建正式文章 | `npx hexo new post "my-new-post"` |
| 新建独立页面 | `npx hexo new page "about"` |
| 预览草稿 | `npx hexo server --draft` |
| 预览正式文章 | `npm run server` |
| 草稿转正式文章 | `npx hexo publish my-new-post` |
| 生成静态网站 | `npm run build` |
| 清理生成结果和缓存 | `npm run clean` |
| 查看 Hexo 版本 | `npx hexo version` |
| 停止预览服务 | `Ctrl+C` |

快捷命令来自本项目的 `package.json`。`npm run server -- --draft` 也可以预览草稿，其中额外的 `--` 用于将参数传给 Hexo。

## 12. 官方参考资料

本文将官方用法改写为适合本仓库的操作步骤；仓库路径、快捷命令和自动部署方式依据本地文件核对。

- [Hexo 文档首页与安装](https://hexo.io/zh-cn/docs/)
- [指令](https://hexo.io/zh-cn/docs/commands)
- [写作与草稿](https://hexo.io/zh-cn/docs/writing)
- [Front-matter](https://hexo.io/zh-cn/docs/front-matter)
- [资源文件夹](https://hexo.io/zh-cn/docs/asset-folders)
- [标签插件](https://hexo.io/zh-cn/docs/tag-plugins)
- [服务器](https://hexo.io/zh-cn/docs/server)
- [网站与主题配置](https://hexo.io/zh-cn/docs/configuration)
- [永久链接](https://hexo.io/zh-cn/docs/permalinks)
- [GitHub Pages 部署](https://hexo.io/zh-cn/docs/github-pages)

Hexo 官方文档采用 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可；本文包含对其内容的概括和针对本项目的补充说明。
