# Jiaxin's Notes

基于 [Hugo](https://gohugo.io) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 的技术博客,部署在 GitHub Pages。极简黑白文字优先风格,主题方向是 DNA sequence modeling。

---

## 0. 一次性环境准备(在你 Mac 上)

```bash
# 装 Hugo extended(本地预览要用,推荐用 Homebrew)
brew install hugo

# 验证 — 任何 v0.130+ 都行
hugo version
```

> 不装 Hugo 也能用 — 只是没法本地预览,只能 push 后看 GitHub Action 跑出来的结果。

---

## 1. 把这个文件夹推到 GitHub

仓库名**建议**叫 `<你的GitHub用户名>.github.io`(例如 `jiaxinli.github.io`)。这样部署后地址就是 `https://你的用户名.github.io/`,不需要 baseURL 加子路径。

```bash
cd "/Users/jiaxinli/Documents/Claude/Projects/personal_website"

# 这里之前在沙箱里残留过半个坏的 .git,先清掉再重来
rm -rf .git

# 重新初始化
git init -b main
git add .
git commit -m "init: Hugo + PaperMod 极简黑白博客骨架"

# 在 GitHub 网页上新建一个空仓库 <用户名>.github.io(不要勾 README/license)
# 然后:
git remote add origin git@github.com:你的用户名/你的用户名.github.io.git
git push -u origin main
```

---

## 2. 在 GitHub 上启用 Pages

仓库页面 → **Settings** → **Pages** → **Build and deployment** →
**Source** 选 **GitHub Actions**(不要选 Branch)。

第一次 push 之后,`Actions` 标签页会有一条 *Deploy Hugo site to Pages* workflow 在跑,大约 1 分钟内完成,完成后再访问 `https://你的用户名.github.io/`。

---

## 3. 修改成你自己的内容

把这几处的占位符换掉:

| 文件 | 改什么 |
| --- | --- |
| `hugo.toml` | `baseURL`、`title`、`params.author`、`params.socialIcons[].url` 里的 `YOUR_USERNAME` |
| `content/about/index.md` | About 页本身的文字、邮箱、社交链接 |
| `content/posts/hello-world.md` | 第一篇示例文章 — 留作参考也行,直接删了重写也行 |

---

## 4. 日常写作

```bash
# 新建一篇草稿(draft: true,不会出现在站点上)
hugo new content posts/dnabert-2-notes.md

# 本地实时预览(包括 draft)
hugo server -D

# 浏览器打开 http://localhost:1313/
```

文章写好后,把开头 frontmatter 里的 `draft: true` 改成 `draft: false`,然后:

```bash
git add content/posts/dnabert-2-notes.md
git commit -m "post: notes on DNABERT-2"
git push
```

push 完 GitHub Action 会自动构建部署,一两分钟后线上就能看到。

---

## 5. 文章 frontmatter 速查

```yaml
---
title: "标题"
date: 2026-05-08T10:00:00+08:00
draft: false
tags: ["dna-modeling", "tokenization"]
categories: ["paper-notes"]
summary: "在文章列表页显示的摘要"
ShowToc: true            # 文章右侧 / 顶部目录
TocOpen: false           # 默认收起
math: true               # 这篇要不要加载 KaTeX 渲染数学公式
cover:                   # 可选封面图
  image: "cover.png"     # 放在文章同目录(page bundle)
  alt: ""
  caption: ""
---
```

要写公式时把 `math: true`,然后用 `$E = mc^2$`、`$$\int_0^1 f(x)\,dx$$`。
(KaTeX 的引入需要再加一个 partial — 等你真要用了再说,默认是关的。)

---

## 6. 主题升级

PaperMod 是直接拷贝在 `themes/PaperMod/` 下的(**不是** submodule —— 沙箱限制导致没法用 submodule 装,但平时无影响)。要升级:

```bash
# 备份你写的 custom.css 别覆盖了
cp assets/css/extended/custom.css /tmp/custom.css.bak

rm -rf themes/PaperMod
git clone --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
rm -rf themes/PaperMod/.git themes/PaperMod/.github themes/PaperMod/exampleSite themes/PaperMod/images

git add themes/PaperMod
git commit -m "chore: bump PaperMod"
```

如果以后想转成 submodule(让升级更优雅),在 Mac 上跑:

```bash
rm -rf themes/PaperMod
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git commit -m "chore: pin PaperMod as submodule"
```

之后升级就是 `cd themes/PaperMod && git pull origin master`。

---

## 7. 目录结构一览

```
personal_website/
├─ hugo.toml                       # 站点配置(baseURL、菜单、PaperMod 参数)
├─ archetypes/default.md           # `hugo new` 用的模板
├─ content/
│  ├─ about/index.md               # About 页
│  ├─ archives.md                  # 归档页(按年列出所有文章)
│  ├─ search.md                    # 搜索页(基于 Fuse.js)
│  └─ posts/
│     └─ hello-world.md            # 第一篇示例文章
├─ assets/css/extended/custom.css  # 黑白极简覆盖样式
├─ static/                         # 直接拷贝到根的静态资源(favicon 等)
├─ themes/PaperMod/                # 主题
└─ .github/workflows/hugo.yml      # 自动部署
```

---

## 8. 想做的小调整速查

- **首页改成"个人简介卡片"风格**:`hugo.toml` 里 `[params.profileMode]` 把 `enabled = true`。
- **关闭 dark mode 切换**:`disableThemeToggle = true`。
- **加评论(giscus)**:看 PaperMod wiki [Comments](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#adding-custom-comments)。
- **加 Google Analytics / Umami**:`hugo.toml` 顶部加 `googleAnalytics = "G-XXXX"`,或自己塞 partial。
- **改字体**:编辑 `assets/css/extended/custom.css` 里 `body` 和 `.post-content` 的 `font-family`。

---

## 9. 出问题排查

- **本地 `hugo server` 报 `theme "PaperMod" not found`** — 你大概在用 sandbox 里那份残缺的 themes 目录;在 Mac 上重新 `git clone` 一遍主题进 `themes/PaperMod/`。
- **GitHub Action 失败** — 看 Actions 日志里 `Build with Hugo` 那一步;90% 的报错是 frontmatter YAML 缩进错了。
- **部署完了访问 404** — 大概率是 Pages source 没切到 GitHub Actions,或者 baseURL 还没改。
