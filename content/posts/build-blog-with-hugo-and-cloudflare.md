+++
title = '用 Hugo + Cloudflare Pages 从零搭建并上线个人博客'
date = 2026-05-30T10:00:00+08:00
draft = false
tags = ['Hugo', 'Cloudflare', '建站', '教程']
categories = ['建站']
summary = '从安装 Hugo、套用 PaperMod 主题，到推送 GitHub、Cloudflare Pages 自动部署、绑定自己的域名——一篇完全可照做的端到端建站实操，也是这个博客自己的诞生过程。'
ShowToc = true
TocOpen = true
+++


> 这是 《Cloudflare 完全指南》 第 3 部分"路线 A"的**完整手把手实操**。跟着从头做到尾，你会得到一个：**全球 CDN 加速、自动 HTTPS、每次 `git push` 自动部署、绑定你自己域名**的个人博客——**月成本 ≈ 一个域名的钱**（托管、构建、流量、证书全免费）。
>
> **技术栈**：Hugo（静态网站生成器，飞快）+ PaperMod（最流行的博客主题）+ GitHub（存代码）+ Cloudflare Pages（构建并全球托管）+ Cloudflare 域名（可选，最后一步）。
>
> **你需要准备**：① 一台电脑（本文以 **Windows 11 + PowerShell** 为主，附 macOS/Linux 命令）；② 一个 **GitHub 账号**（免费，[github.com](https://github.com) 注册）；③ 一个 **Cloudflare 账号**；④（可选，最后一步用）一个**已接入 Cloudflare 的域名**——接入方法见主指南第 2 部分。
>
> **预计耗时**：第一次走完约 30–60 分钟。

## 目录

- [你将搭建什么](#你将搭建什么)
- [先记住：三个最容易翻车的点](#先记住三个最容易翻车的点)
- [阶段一·安装 Hugo 与 Git](#阶段一安装-hugo-与-git)
- [阶段二·本地把博客搭出来](#阶段二本地把博客搭出来)
- [阶段三·推送到 GitHub](#阶段三推送到-github)
- [阶段四·部署到 Cloudflare Pages](#阶段四部署到-cloudflare-pages)
- [阶段五·绑定你自己的域名](#阶段五绑定你自己的域名)
- [阶段六·日常写作工作流](#阶段六日常写作工作流)
- [阶段七·进阶优化](#阶段七进阶优化)
- [阶段八·常见报错排查](#阶段八常见报错排查)
- [附录·完整命令速查](#附录完整命令速查)

---

## 你将搭建什么

```
你写 Markdown 文章
      │  git push
      ▼
  GitHub 仓库 ───────▶ Cloudflare Pages（自动用 Hugo 构建）
                              │
                              ▼
                   全球边缘节点托管 + 免费 HTTPS + 你的域名
                              │
                              ▼
                        读者飞快地打开你的博客
```

整条链路里，**你唯一要做的日常操作就是"写文章 + git push"**，其余全自动。

---

## 先记住：三个最容易翻车的点

这三个坑能劝退一半新手，**先在脑子里记一下**，正文到对应步骤会再提醒：

1. **必须设 `HUGO_VERSION` 环境变量。** Cloudflare 构建机自带的 Hugo 版本很老（v2 构建系统默认 `0.118.2`，v1 是远古的 `0.54.0`），不设的话现代主题会构建失败。→ [阶段四](#阶段四部署到-cloudflare-pages)
2. **主题用 HTTPS 链接装。** 主题以 git submodule 方式安装时，地址必须是 `https://github.com/...`，**不能用 `git@github.com:`（SSH）**，否则 Cloudflare 拉不到主题、构建报 submodule 错误。→ [阶段二](#阶段二本地把博客搭出来)
3. **`baseURL` 要填成你真实的网址（带结尾斜杠）。** 填错（比如留着 `localhost` 或 `example.org`）会导致**线上页面没有样式 / CSS 404**。→ [阶段二](#阶段二本地把博客搭出来) 设初值，[阶段五](#阶段五绑定你自己的域名) 改成你的域名。

---

## 阶段一·安装 Hugo 与 Git

### 1.1 安装 Hugo（Extended 扩展版）

> **为什么要"Extended 扩展版"**：它内置 SCSS/Sass 编译器，很多主题需要。虽然 PaperMod 自带编译好的 CSS、用普通版也行，但**扩展版是万能选择**，一律装它不会错。

**Windows（PowerShell，推荐 winget）：**
```powershell
winget install Hugo.Hugo.Extended
```
其他 Windows 包管理器任选其一也可：
```powershell
scoop install hugo-extended      # 如果你用 Scoop
choco install hugo-extended      # 如果你用 Chocolatey（需管理员 PowerShell）
```

**macOS：**
```bash
brew install hugo                # Homebrew 装的默认就是扩展版
```

**Linux：** 推荐从 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载 `hugo_extended_..._linux-amd64.deb` 安装；或 `brew install hugo`。
> ⚠️ 别用 `apt install hugo`——发行版仓库里常是**非扩展版且版本很旧**。

### 1.2 安装 Git

- **Windows**：`winget install Git.Git`（或到 [git-scm.com](https://git-scm.com) 下载安装）。
- **macOS**：`brew install git`（或 `xcode-select --install`）。
- **Linux**：`sudo apt install git`。

### 1.3 验证安装

**关掉并重开一个新的终端**（让 PATH 生效），运行：
```powershell
hugo version
git --version
```
能看到版本号即成功。`hugo version` 输出里应包含 `extended`，且版本号在 **0.15x / 0.16x** 这个区间（2025–2026 的当前系列）。**记下这个版本号**（比如 `0.153.0`），阶段四要用到。

---

## 阶段二·本地把博客搭出来

### 2.1 创建站点

挑个放代码的目录（示例用 `F:\blog`），创建站点骨架：
```powershell
hugo new site myblog
cd myblog
```
看一眼生成的目录结构：
```
myblog/
├── archetypes/      # 新文章模板
├── assets/          # 需要被处理的资源（SCSS 等）
├── content/         # 你的文章就放这里 ★
├── data/
├── i18n/
├── layouts/         # 自定义模板（覆盖主题用）
├── static/          # 原样拷贝的静态文件（图片等）
├── themes/          # 主题放这里 ★
└── hugo.toml        # 站点配置文件 ★（新版叫 hugo.toml，不是 config.toml）
```
> 💡 现代 Hugo 生成的配置文件叫 **`hugo.toml`**（自 v0.110 起）。若你在老教程里看到 `config.toml` 别慌，两者都能用，Hugo 会优先读 `hugo.toml`。

### 2.2 初始化 Git 并安装 PaperMod 主题

主题用 **git submodule** 安装，所以**先初始化 git 仓库**，再加子模块：
```powershell
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
```
> ⚠️ **坑点 2**：上面的地址是 `https://...`，**务必保持 HTTPS**。如果你习惯改成 `git@github.com:...`（SSH），Cloudflare 构建时会因为没有你的 SSH 密钥而拉不到主题、部署失败。
>
> 检查一下：打开自动生成的 `.gitmodules` 文件，确认里面的 `url =` 是 `https://...`。

### 2.3 写配置文件 `hugo.toml`

用编辑器（VS Code、记事本均可）打开 `hugo.toml`，**全部替换**成下面这份可直接用的配置（注释说明了哪些是可选的，按需改）：

```toml
# —— 坑点 3：baseURL 先填占位的 pages.dev，阶段五再改成你的域名；结尾必须带 / ——
baseURL = 'https://myblog.pages.dev/'
title   = '我的博客'
theme   = 'PaperMod'

languageCode    = 'zh-cn'
hasCJKLanguage  = true      # 中文必备：让字数统计/摘要/阅读时长按中文字符正确计算
enableRobotsTXT = true      # 自动生成 robots.txt（利于 SEO）
buildDrafts     = false     # 构建时不发布草稿
buildFuture     = false     # 不发布"未来日期"的文章

[pagination]
  pagerSize = 10            # 每页文章数（Hugo 0.128+ 用这个写法）

[minify]
  minifyOutput = true       # 输出压缩后的 HTML（省流量、更快）

[markup.highlight]
  noClasses = false         # 配合主题做代码高亮（明暗主题自适应）

[params]
  env             = 'production'
  description     = '一个用 Hugo + Cloudflare 搭建的博客'
  defaultTheme    = 'auto'  # 跟随系统明暗
  ShowReadingTime = true
  ShowShareButtons = true
  ShowCodeCopyButtons = true   # 代码块一键复制
  ShowToc         = true       # 文章目录
  ShowBreadCrumbs = true

  # 首页顶部的问候语（可选）
  [params.homeInfoParams]
    Title   = '你好 👋'
    Content = '欢迎来到我的博客，这里记录我的折腾与思考。'

  # 社交图标（可选，按需增删）
  [[params.socialIcons]]
    name = 'github'
    url  = 'https://github.com/你的用户名'

# —— 顶部导航菜单 ——
[[menu.main]]
  name   = '文章'
  url    = '/posts/'
  weight = 10
[[menu.main]]
  name   = '标签'
  url    = '/tags/'
  weight = 20
[[menu.main]]
  name   = '关于'
  url    = '/about/'
  weight = 30
```

### 2.4 写第一篇文章

```powershell
hugo new content posts/hello-world.md
```
这会在 `content/posts/` 下生成文件。打开它，内容大致如下（front matter 可能是 `+++`(TOML) 或 `---`(YAML) 格式，取决于你的 Hugo 版本，都正常）：

````toml
+++
title = '你好，世界'
date = 2026-05-30T10:00:00+08:00
draft = true
+++

这是我的第一篇博客文章。

## 一个小标题

正文用 **Markdown** 写就行，支持代码块：

```python
print("Hello, Cloudflare!")
```
````

> 关键字段：把 **`draft = true` 改成 `draft = false`** 这篇才会被正式发布。可以再加一行 `tags = ['随笔', '建站']` 给文章打标签。

顺手再建个"关于"页面（对应菜单里的 `/about/`）：
```powershell
hugo new content about.md
```
编辑它，把 `draft` 设为 `false`，写两句自我介绍。

### 2.5 本地预览

```powershell
hugo server -D
```
`-D` 表示**连草稿一起预览**。终端会显示一个地址（默认 `http://localhost:1313/`），浏览器打开就能看到博客了。**保持这个命令运行**，你改动文章/配置时页面会自动热刷新。看完按 `Ctrl + C` 停止。

> ✅ 到这里，一个能本地跑的博客就成型了。下面让全世界都能访问它。

---

## 阶段三·推送到 GitHub

### 3.1 在 GitHub 创建一个空仓库

登录 [github.com](https://github.com) → 右上角 **+ → New repository**：
- **Repository name**：如 `myblog`
- 选 **Public（公开）**（用免费的 giscus 评论、giscus 需要公开仓库；纯部署其实公开/私有都行）
- ⚠️ **不要**勾选 "Add a README / .gitignore / license"（保持空仓库，避免首次推送冲突）
- 点 **Create repository**

### 3.2 添加 `.gitignore`

在项目根目录新建文件 `.gitignore`，内容：
```gitignore
/public/
/resources/_gen/
.hugo_build.lock
```
（这些是构建产物和缓存，不需要进仓库——Cloudflare 会自己构建。）

### 3.3 提交并推送

把 GitHub 页面上显示的仓库地址替换到下面（注意是 **https** 地址）：
```powershell
git add .
git commit -m "初始化我的 Hugo 博客"
git branch -M main
git remote add origin https://github.com/你的用户名/myblog.git
git push -u origin main
```
> 首次推送会要求登录 GitHub（浏览器授权或输入凭据）。

### 3.4 确认主题也推上去了

刷新 GitHub 仓库页面，进入 `themes/` 目录——你应该看到 **`PaperMod @ 某个commit`** 这样的子模块引用（带一个箭头图标）。**如果 `themes/PaperMod` 是空的**，说明子模块没提交成功，Cloudflare 构建会因为"没有主题"而失败，回到 [2.2](#22-初始化-git-并安装-papermod-主题) 重做子模块步骤。

---

## 阶段四·部署到 Cloudflare Pages

### 4.1 连接 Git 仓库

1. 登录 [Cloudflare 仪表盘](https://dash.cloudflare.com) → **Workers & Pages** → **Create application** → **Pages** 选项卡 → **Connect to Git**。
2. 授权并选择刚才的 GitHub 仓库 `myblog` → **Begin setup**。

### 4.2 填写构建配置（关键一步）

在 **Set up builds and deployments** 里：

| 配置项 | 填什么 |
|---|---|
| **Framework preset（框架预设）** | 选 **Hugo** |
| **Build command（构建命令）** | `hugo` |
| **Build output directory（输出目录）** | `public` |

然后**务必**展开 **Environment variables（环境变量）**，新增一个：

| 变量名 | 值 |
|---|---|
| **`HUGO_VERSION`** | 你本地 `hugo version` 看到的版本，例如 `0.153.0` |

> ⚠️ **坑点 1 就在这里**：不设 `HUGO_VERSION`，Cloudflare 会用它自带的老版本 Hugo（默认 `0.118.2`，老构建系统甚至是 `0.54.0`），现代主题会直接构建失败或样式错乱。**填一个和本地一致的较新版本**最稳。
>
> 另外建议确认你的项目用的是 **v2 构建系统**（Ubuntu 22.04），新项目默认就是 v2，一般无需改动。

### 4.3 部署

点 **Save and Deploy**。构建日志会实时滚动（拉取代码 → 拉取子模块主题 → 跑 `hugo` → 全球分发），约几十秒。成功后你会得到一个网址：

**`https://myblog.pages.dev`** —— 打开它，你的博客已经在全球上线了 🎉

> 如果这一步报错，**先看构建日志最后几行**，再对照 [阶段八·报错排查](#阶段八常见报错排查)。

---

## 阶段五·绑定你自己的域名

> 前提：你的域名**已经接入 Cloudflare**（即 nameserver 已指向 Cloudflare、状态 Active）。还没做的话，按主指南第 2 部分操作，5 分钟搞定。

### 5.1 在 Pages 里添加自定义域名

1. 进入你的 Pages 项目 → **Custom domains** 选项卡 → **Set up a domain**。
2. 输入你想用的域名，例如 `blog.example.com`（子域）或 `example.com`（根域）→ 按提示 **Continue / Activate**。
   - 域名已在 Cloudflare 托管时，**CNAME 会自动加好**，证书也自动签发，等一两分钟即可。
3. 想要 `www` 和根域都能访问，就把两个都加进来（或在其中一个上做重定向统一到另一个）。

> ⚠️ 一定走这个 **Set up a domain** 流程；只在 DNS 里手动加 CNAME 而不在 Pages 里绑定，会报 **522**。

### 5.2 把 `baseURL` 改成你的域名（坑点 3）

回到代码，编辑 `hugo.toml` 第一行：
```toml
baseURL = 'https://blog.example.com/'    # 换成你的真实域名，结尾保留 /
```
然后推送，触发自动重新部署：
```powershell
git add hugo.toml
git commit -m "更新 baseURL 为自定义域名"
git push
```
> **为什么必须改**：PaperMod 用 `baseURL` 生成 CSS/JS/链接的绝对地址。如果 `baseURL` 还停留在占位的 `myblog.pages.dev` 或 `localhost`，在你的自定义域名下访问就会**加载不到样式（页面变成纯文字）**。

等部署完成，访问 `https://blog.example.com`，确认：有锁标（HTTPS 正常）、样式正常、点链接不 404。**完工！**

---

## 阶段六·日常写作工作流

以后发一篇新文章，只有三步：

```powershell
# 1. 新建文章
hugo new content posts/my-second-post.md

# 2. 编辑它：写正文，并把 draft 改成 false
#    （写作时可随时 hugo server -D 本地预览）

# 3. 推送 → Cloudflare 自动构建上线（约 1 分钟后生效）
git add .
git commit -m "发布：我的第二篇文章"
git push
```

几个实用习惯：
- **草稿**：保留 `draft = true`，文章不会上线，但本地 `hugo server -D` 能预览。
- **预览部署**：从非 `main` 分支推送，Cloudflare 会生成一个独立预览网址（不影响正式站），适合大改版前试效果。
- **定时发布**：把 `date` 设成未来时间 + 配置里 `buildFuture = false`，文章到点后下次构建才出现（配合 Cron 触发 可做到自动定时）。

---

## 阶段七·进阶优化

### 7.1 接入 Cloudflare Web Analytics（免费、隐私友好、无需 Cookie）

最简单的方式：在 **Pages 项目 → Metrics / Web Analytics** 里为该项目**启用 Web Analytics**，Cloudflare 会自动给页面注入统计脚本，**不用改一行代码**。之后就能看到访问量、访客地区、Core Web Vitals 性能指标——而且不打扰访客、不收集隐私。

### 7.2 加评论系统 giscus（基于 GitHub Discussions，免费无广告）

比 Disqus 干净，评论直接存到你的 GitHub Discussions 里：

1. 你的博客仓库需为 **Public**，并在仓库 **Settings → Features** 勾选 **Discussions**。
2. 安装 [giscus GitHub App](https://github.com/apps/giscus) 到该仓库。
3. 打开 [giscus.app](https://giscus.app)，填入你的仓库，选好映射方式和分类，它会生成一段 `<script ...>`，记下其中的 `data-repo`、`data-repo-id`、`data-category`、`data-category-id`。
4. 在项目里新建文件 **`layouts/partials/comments.html`**，粘贴 giscus 片段（把占位换成你的值）：
   ```html
   <script src="https://giscus.app/client.js"
           data-repo="你的用户名/myblog"
           data-repo-id="填你的"
           data-category="Announcements"
           data-category-id="填你的"
           data-mapping="pathname"
           data-theme="preferred_color_scheme"
           data-lang="zh-CN"
           crossorigin="anonymous" async>
   </script>
   ```
5. 在 `hugo.toml` 的 `[params]` 下加一行 **`comments = true`**，推送即可。PaperMod 会在文章页自动渲染这个评论分区。

### 7.3 SEO 与订阅（PaperMod 已基本内置）

- **RSS 订阅**：Hugo 自动生成 `index.xml`，读者可用 RSS 阅读器订阅。
- **站点地图 sitemap.xml**：自动生成，配合 `enableRobotsTXT = true` 利于被搜索引擎收录。
- **社交分享卡片（Open Graph / Twitter Card）**：PaperMod 自带，分享到社交平台有预览图文。给文章 front matter 加 `cover` 字段可设封面图。

### 7.4 自定义 404 页

Hugo/PaperMod 会生成 `404.html`，而 Cloudflare Pages **自动**用它作为找不到页面时的响应——无需额外配置。想自定义就在 `layouts/404.html` 覆盖。

### 7.5 改了文章但线上没变？清缓存

正常情况下每次部署 Cloudflare 会更新内容。若你改了 Cloudflare 侧的缓存规则导致旧页面残留，可到 **域名 → Caching → Configuration → Purge Everything** 清一次缓存。

### 7.6 想跟随官方未来主线？也可部署到 Workers

主指南提到官方正把重心转向 **Workers + 静态资源**。Hugo 博客同样可以这么部署：本地 `hugo` 生成 `public/` 后，用
```powershell
npx wrangler pages deploy public
```
直接上传（无需 GitHub），或用 Workers 的静态资源托管把 `public` 作为 `assets.directory`。对一个"Git 自动部署的博客"而言，**Pages + Git 仍是当前最顺手的路径**，本教程主线推荐它；了解 Workers 路线见主指南 第 3.4 节。

---

## 阶段八·常见报错排查

| 现象 | 大概率原因 | 解决 |
|---|---|---|
| **线上能打开但没样式 / CSS、JS 报 404** | `baseURL` 填错（还是 localhost/pages.dev/example.org） | 把 `hugo.toml` 的 `baseURL` 改成你真实的访问域名（带结尾 `/`），重新 push（坑点 3） |
| **构建失败，日志含 `submodule` / `could not read Username`** | 主题子模块用了 SSH 地址，或主题没推上 GitHub | 确认 `.gitmodules` 里是 `https://` 地址；确认 GitHub 上 `themes/PaperMod` 非空（坑点 2） |
| **构建失败，日志是模板/函数报错（如 `function "xxx" not defined`）** | Cloudflare 用了过老的 Hugo | 设 `HUGO_VERSION` 环境变量为较新版本（坑点 1） |
| **页面打开但主题区域空白 / 报 "found no layout"** | 本地 `themes/PaperMod` 是空的（子模块没初始化） | 本地跑 `git submodule update --init --recursive` 后重新提交 |
| **文章不显示** | `draft = true`，或 `date` 是未来时间 | 把 `draft` 改 `false`；检查日期；本地用 `hugo server -D` 验证 |
| **绑定自定义域名后报 522** | 没走 Pages 的 Set up a domain 流程，只手动加了 CNAME | 在 Pages 项目 **Custom domains** 里正式绑定 |
| **`hugo` 命令找不到** | 安装后没重开终端 / PATH 没生效 | 关掉终端重开；或重装并勾选"加入 PATH" |
| **分页 `[pagination]` 不生效或报错** | `HUGO_VERSION` 低于 0.128 | 升级 `HUGO_VERSION`；或改用旧写法 `paginate = 10` |

> 排查通用心法：**先看 Cloudflare 构建日志的最后 15 行**，90% 的错误原因都写在那里。

---

## 附录·完整命令速查

从零到上线，一条龙（Windows PowerShell；macOS/Linux 把第一条安装命令换成对应方式即可）：

```powershell
# ① 装工具（装完重开终端）
winget install Hugo.Hugo.Extended
winget install Git.Git
hugo version            # 记下版本号，例如 0.153.0
git --version

# ② 本地建站 + 装主题（注意 HTTPS 地址）
hugo new site myblog
cd myblog
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
#   —— 编辑 hugo.toml（见 2.3）——

# ③ 写第一篇 + 本地预览
hugo new content posts/hello-world.md
#   —— 编辑文章，draft 改成 false ——
hugo server -D          # 浏览器开 http://localhost:1313 ，看完 Ctrl+C

# ④ 推到 GitHub（先在网页建好空仓库）
#   —— 新建 .gitignore（见 3.2）——
git add .
git commit -m "初始化我的 Hugo 博客"
git branch -M main
git remote add origin https://github.com/你的用户名/myblog.git
git push -u origin main

# ⑤ 到 Cloudflare Pages：Connect to Git → 选仓库
#    Framework preset = Hugo，Build command = hugo，Output = public
#    环境变量 HUGO_VERSION = 0.153.0   ← 别忘了！
#    Save and Deploy → 得到 myblog.pages.dev

# ⑥ 绑定域名后，把 hugo.toml 的 baseURL 改成你的域名再 push

# —— 之后每发一篇文章 ——
hugo new content posts/new-post.md
git add . ; git commit -m "新文章" ; git push
```

### 上线检查清单

- [ ] `hugo version` 显示 `extended` 且版本是 0.15x/0.16x
- [ ] `.gitmodules` 里主题地址是 `https://`（不是 `git@`）
- [ ] GitHub 上 `themes/PaperMod` 目录**非空**
- [ ] Cloudflare Pages 设了 **`HUGO_VERSION`** 环境变量
- [ ] 构建命令 `hugo`、输出目录 `public`
- [ ] `myblog.pages.dev` 能正常打开且有样式
- [ ] 绑定域名后，`hugo.toml` 的 **`baseURL` 已改成你的域名**（带 `/`）
- [ ] 访问自定义域名：HTTPS 正常、样式正常、链接不 404
- [ ] （可选）开了 Web Analytics、配了 giscus 评论

---

> 🎯 **回顾**：你用 Hugo 写文章 → 推到 GitHub → Cloudflare Pages 自动构建并全球托管 → 套上自己的域名和免费 HTTPS。三个核心要点别忘——**HUGO_VERSION、submodule 用 HTTPS、baseURL 填对**。剩下的，就是好好写博客了。✍️
>
> 想深入了解每个环节背后的 Cloudflare 能力（缓存、安全、分析、Workers 全栈），回到 《Cloudflare 完全指南》。
