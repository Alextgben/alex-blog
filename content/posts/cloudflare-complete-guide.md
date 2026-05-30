+++
title = 'Cloudflare 完全指南：从域名托管到网站部署'
date = 2026-05-30T14:00:00+08:00
draft = false
tags = ['Cloudflare', 'CDN', 'DNS', '建站', '教程']
categories = ['建站']
summary = '系统的 Cloudflare 中文指南：托管域名、部署网站，并讲透仪表盘里的域名、构建、保护、连接、观察五大模块。'
ShowToc = true
TocOpen = false

+++





> 一份面向中文用户的 Cloudflare 系统教程。既讲**概念**，也给**可照做的步骤**；既覆盖**入门**（托管域名、上线第一个网站），也包含**进阶**（开发者平台、安全策略、网络隧道、可观测性）。重点讲透仪表盘里的五大板块：**域名（Domains）/ 构建（Build）/ 保护（Protect）/ 连接（Connect）/ 观察（Observe）**。
>
> **阅读建议**：第一次用 Cloudflare，请按 第 1→2→3 部分顺序读，30 分钟内即可让域名上 Cloudflare 并部署一个网站。需要查具体功能时，直接跳到对应模块章节。
>
> ⚠️ **关于时效**：Cloudflare 迭代很快，**仪表盘的菜单名、板块标题会做 A/B 测试**，**套餐价格 2025 年有上调**。本文菜单名以官方英文为准并附中文，价格等关键数字请在动手时以 [cloudflare.com/plans](https://www.cloudflare.com/plans/) 和官方文档 [developers.cloudflare.com](https://developers.cloudflare.com/) 的最新版为准。本文标注 ⚠️ 处即为"请以官方最新为准"。

---

## 目录

- [第 1 部分　认识 Cloudflare](#第-1-部分认识-cloudflare)
  - [1.1 Cloudflare 到底是什么](#11-cloudflare-到底是什么)
  - [1.2 它的核心原理：边缘网络与反向代理](#12-它的核心原理边缘网络与反向代理)
  - [1.3 产品全景：五大板块](#13-产品全景五大板块)
  - [1.4 注册账户与仪表盘导航（两层结构）](#14-注册账户与仪表盘导航两层结构)
- [第 2 部分　入门·托管你的域名（域名 / Domains 模块）](#第-2-部分入门托管你的域名域名--domains-模块)
  - [2.1 完整接入流程（Onboarding 七步）](#21-完整接入流程onboarding-七步)
  - [2.2 DNS 记录管理](#22-dns-记录管理)
  - [2.3 核心概念：橙云 vs 灰云（代理 vs 仅 DNS）](#23-核心概念橙云-vs-灰云代理-vs-仅-dns)
  - [2.4 SSL/TLS 入门：免费证书与加密模式](#24-ssltls-入门免费证书与加密模式)
  - [2.5 DNSSEC、子域名与高级 DNS](#25-dnssec子域名与高级-dns)
  - [2.6 Cloudflare Registrar：按成本价注册/转入域名](#26-cloudflare-registrar按成本价注册转入域名)
- [第 3 部分　入门·部署你的网站（构建 / Build 模块）](#第-3-部分入门部署你的网站构建--build-模块)
  - [3.1 先搞清楚：Pages 还是 Workers？](#31-先搞清楚pages-还是-workers)
  - [3.2 路线 A：Pages + Git（最推荐的零运维方案）](#32-路线-apages--git最推荐的零运维方案)
  - [3.3 路线 B：Pages 直接上传（拖拽 / Wrangler）](#33-路线-bpages-直接上传拖拽--wrangler)
  - [3.4 路线 C：Workers + 静态资源（官方新项目首选）](#34-路线-cworkers--静态资源官方新项目首选)
  - [3.5 绑定你自己的域名](#35-绑定你自己的域名)
  - [3.6 Wrangler CLI 速查](#36-wrangler-cli-速查)
- [第 4 部分　进阶·构建（Build）开发者平台全景](#第-4-部分进阶构建build开发者平台全景)
- [第 5 部分　保护（Protect）：安全模块详解](#第-5-部分保护protect安全模块详解)
- [第 6 部分　连接（Connect）：网络模块详解](#第-6-部分连接connect网络模块详解)
- [第 7 部分　观察（Observe）：分析与日志模块详解](#第-7-部分观察observe分析与日志模块详解)
- [第 8 部分　进阶补充：性能、缓存与规则引擎](#第-8-部分进阶补充性能缓存与规则引擎)
- [第 9 部分　套餐与价格](#第-9-部分套餐与价格)
- [第 10 部分　排错 FAQ：最常见的几个坑](#第-10-部分排错-faq最常见的几个坑)
- [第 11 部分　学习路径与官方资源](#第-11-部分学习路径与官方资源)
- [附录 A　命令速查表](#附录-a命令速查表)
- [附录 B　新手上线检查清单](#附录-b新手上线检查清单)
- [附录 C　术语表](#附录-c术语表)

---

## 第 1 部分　认识 Cloudflare

### 1.1 Cloudflare 到底是什么

一句话：**Cloudflare 是一张覆盖全球的网络，你把域名/流量交给它，它就在"用户"和"你的服务器"之间充当一道又快又安全的中间层。**

它最初以 **CDN（内容分发网络）+ DDoS 防护 + 免费 DNS** 出名，如今已扩展成 Cloudflare 自称的"**连接云（connectivity cloud）**"，能力覆盖五个方向：

- **更快**：全球边缘节点缓存与就近加速（CDN、Argo）。
- **更安全**：免费且不限量的 DDoS 防护、Web 应用防火墙（WAF）、机器人管理。
- **更省心**：免费 SSL 证书、免费权威 DNS、按成本价的域名注册。
- **能开发**：在它的边缘网络上直接跑代码、存数据、托管整站（Workers / Pages / R2 / D1…）。
- **能组网**：用隧道、零信任把你的内网、家庭服务器、办公室安全连起来（Tunnel / Zero Trust / Magic WAN）。

对个人和中小团队最有吸引力的一点：**上面绝大多数核心功能在免费套餐里就能用**——免费 DNS、免费 CDN、免费 SSL、免费 DDoS 防护、免费静态网站托管。

> ⚠️ **中国大陆访问者请先读这条**：Cloudflare 的免费版/Pro 版在**中国大陆的访问体验通常不稳定**（延迟偏高、偶有运营商干扰），因为标准 anycast 节点不在境内。要给大陆用户良好体验，需要 Cloudflare 的**中国网络（China Network）**——它属于 **Enterprise** 套餐，且要求网站**已完成 ICP 备案**。结论：拿 Cloudflare 练手、做面向海外/港澳台的站点完全没问题；但若做**主要面向大陆**的正式站点，请先评估这一点，别等上线才发现访问慢。

### 1.2 它的核心原理：边缘网络与反向代理

理解 Cloudflare，只需要抓住一个机制：**反向代理 + Anycast 任播**。

```
        【没有 Cloudflare】
  用户 ────────直连────────▶ 你的服务器(真实IP暴露)

        【接入 Cloudflare 并开启"代理"】
  用户 ──▶ 最近的 Cloudflare 边缘节点 ──▶ 你的服务器(真实IP隐藏)
            │
            ├─ 缓存静态内容（更快）
            ├─ 拦截攻击 / WAF / DDoS（更安全）
            ├─ 终止 HTTPS（免费证书）
            └─ 记录分析数据（可观测）
```

- **Anycast 任播**：Cloudflare 在全球数百个城市有数据中心，它们对外用同一批 IP。用户的请求会被自动路由到**物理上离他最近**的节点，天然分散了流量、抗住了 DDoS。
- **反向代理**：开启代理后，全世界看到的是 Cloudflare 的 IP，而不是你服务器的真实 IP。于是缓存、防火墙、证书、分析这些能力才有机会"插队"到中间生效。

> 🔑 **记住这条主线**：Cloudflare 的几乎所有"魔法"，都依赖于流量**真正经过它的边缘**。这就是后面 [2.3 橙云 vs 灰云](#23-核心概念橙云-vs-灰云代理-vs-仅-dns) 为什么是全篇最重要的概念。

### 1.3 产品全景：五大板块

你提到的 **域名 / Observe / 构建 / 保护 / 连接**，正是 Cloudflare 官方的**产品分类法**（也是它"连接云"叙事的五根支柱）。下表是全景速查，后续每个板块都有专门章节：

| 板块（中/英） | 定位 | 代表产品 | 本文章节 |
|---|---|---|---|
| **域名 Domains** | 把域名接进来、管 DNS、买/转域名 | DNS、Registrar、域名接入 | [第 2 部分](#第-2-部分入门托管你的域名域名--domains-模块) |
| **构建 Build** | 在边缘开发与托管应用 | Workers、Pages、R2、D1、KV、Workers AI… | [第 3](#第-3-部分入门部署你的网站构建--build-模块) / [第 4 部分](#第-4-部分进阶构建build开发者平台全景) |
| **保护 Protect** | 安全防护 | WAF、DDoS、Bot 管理、Turnstile、Page Shield、Zero Trust | [第 5 部分](#第-5-部分保护protect安全模块详解) |
| **连接 Connect** | 网络与零信任组网 | Tunnel、WARP、Access/Gateway、Spectrum、Magic WAN | [第 6 部分](#第-6-部分连接connect网络模块详解) |
| **观察 Observe** | 分析与日志 | Web Analytics、流量分析、Logpush、Health Checks | [第 7 部分](#第-7-部分观察observe分析与日志模块详解) |

> ⚠️ **重要澄清**：Build / Observe / Protect / Connect / Domains 是 Cloudflare 的**概念分类**，非常适合用来组织学习。但**仪表盘左侧实际显示的栏目名**会随版本和 A/B 测试变化，你看到的可能是 `Analytics & Logs`、`Compute & AI`、`Storage & Databases`、`Security`、`Network` 等更细的标题，而**零信任（Zero Trust / Cloudflare One）有独立的控制台**（`one.dash.cloudflare.com`）。看不到完全一致的字样是正常的，认准"功能"而非"字面菜单名"。

### 1.4 注册账户与仪表盘导航（两层结构）

**注册**：访问 [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)，用邮箱注册并验证，**强烈建议立刻开启两步验证（2FA）**——你的域名解析权将托管在这个账户里，一旦被盗后果严重。

**仪表盘最关键的结构认知：两层导航。** 这是新手最容易迷路的地方：

```
账户（Account）
  └── 区域 / 域名 A（Zone）
  └── 区域 / 域名 B（Zone）
  └── ...
```

- **账户级（Account level）**：登录后、**还没点进某个具体域名**时看到的界面。这里放的是"影响整个账户或跨多个域名"的产品，例如：**Workers & Pages、R2、Security Center、Bulk Redirects、Lists、Members（成员）、Billing（账单）、Notifications（通知）、Audit Log（审计日志）**。
- **区域级 / 域名级（Zone level）**：**点进某个具体域名之后**看到的界面。这里放的是"只影响这一个域名"的设置，例如：**DNS、SSL/TLS、Caching（缓存）、Security（安全/WAF）、Rules（规则）、Speed、Analytics、Load Balancing**。

切换方式：左上角有一个**域名选择器**（可搜索、可把常用域名加星标），点进去就是区域级；点 **"Account Home / 账户主页"** 回到账户级。

> 💡 术语对齐：**一个账户（account）= 一个登录身份**；**一个区域（zone）= 一个域名**。文档里说的"zone settings"就是"某个域名的设置"。

---

## 第 2 部分　入门·托管你的域名（域名 / Domains 模块）

本部分回答两个问题：**怎样把一个已经买好的域名接入 Cloudflare**，以及**域名（Domains）模块都能做什么**。

> 前提：你已经在某个注册商（阿里云、腾讯云、GoDaddy、Namecheap、Spaceship 等）**买好了一个域名**。如果还没买，可直接跳到 [2.6 用 Cloudflare Registrar 买域名](#26-cloudflare-registrar按成本价注册转入域名)。

### 2.1 完整接入流程（Onboarding 七步）

这套流程叫 **Full setup（完整接入）**，本质是：**把域名的"权威 DNS"从原注册商换成 Cloudflare**。换完之后，全世界查询你域名的解析，都由 Cloudflare 回答，于是它才能在中间施展所有能力。

**第 1 步　添加域名**
登录仪表盘 → 进入 **Domains / Account Home** → 点 **"Add a domain / 添加域名"**（也叫 Onboard a domain）。
输入**根域名**（如 `example.com`），**不要**带 `https://`、**不要**带 `www.`。→ 点 **Continue / 继续**。

**第 2 步　选择套餐**
选 **Free / Pro / Business / Enterprise**。**新手直接选 Free 即可**——免费套餐已包含 DNS、CDN 代理、Universal SSL、基础 DDoS 与 WAF。后续随时可升级。

**第 3 步　扫描并核对 DNS 记录（关键，最容易出事）**
Cloudflare 会**自动扫描**你域名当前的 DNS 记录（A、CNAME、MX、TXT 等）并导入。

> ⚠️ **官方明确说明：自动扫描"不保证找全"所有记录。** 这是新手翻车 No.1 的原因。**在切换 nameserver 之前，请对照你原注册商的 DNS 面板，逐条核对**，特别是：
> - **MX 记录 + 邮件相关的 TXT（SPF/DKIM/DMARC）**——漏了会导致**邮件收发瘫痪**；
> - 各种子域名解析、第三方服务的验证 TXT 记录。
> 漏掉的记录请在这一步手动补齐。自动扫描只在接入时提供，事后没有"再扫一次"按钮。

每条记录可以设置**代理状态**（橙云=代理 / 灰云=仅 DNS，详见 [2.3](#23-核心概念橙云-vs-灰云代理-vs-仅-dns)）。核对无误 → **Continue**。

**第 4 步　拿到 Cloudflare 分配的 nameserver**
Cloudflare 会给你**两个专属的 nameserver**，形如：
```
abby.ns.cloudflare.com
will.ns.cloudflare.com
```
（每个账户分到的名字不同，记下这两个。）

**第 5 步　去原注册商替换 nameserver**
登录你买域名的**注册商后台** → 找到 **Nameservers / DNS 服务器 / 域名服务器** 设置 → **删除原有的 nameserver**，**替换成 Cloudflare 给的这两个**。
> ⚠️ 如果原注册商开启了 **DNSSEC，请先在注册商处关闭它**，否则切换期间域名可能直接解析失败、整站打不开。

**第 6 步　等待激活**
状态会从 **Pending（待处理）→ Active（已激活）**。激活后你会收到**确认邮件**。
- **时间**：通常**几分钟到几小时**，官方上限说法**最长 24 小时**（老文档写 72 小时）。
- 可在域名概览页点 **"Check nameservers now / 立即检查"** 主动催一下。

**第 7 步　收尾检查 SSL**
激活后去 **SSL/TLS** 看一眼加密模式（见 [2.4](#24-ssltls-入门免费证书与加密模式)），避免出现"重定向循环"或证书告警。

#### Full setup vs Partial（CNAME）setup

| | **Full setup（完整接入）** | **Partial / CNAME setup（部分接入）** |
|---|---|---|
| 适用套餐 | 所有套餐（含 Free） | **仅 Business / Enterprise** |
| 权威 DNS 在谁手里 | **Cloudflare** | **保留你原来的 DNS 服务商** |
| 怎么接 | 改 nameserver | 只把**指定子域**用 CNAME 指向 `<主机名>.cdn.cloudflare.net` |
| 能力 | 完整 | 受限（apex 根域通常无法代理、DNS 层无 DDoS 防护） |

新手与个人站**一律用 Full setup**。Partial 是给"DNS 必须留在别处、只想给个别子域套上 Cloudflare"的大客户用的。

> 🔄 **想撤回 / 退出 Cloudflare 怎么办？** 接入本质只是"把 nameserver 指向了 Cloudflare"，随时可逆。要迁回原服务商：**先在新的 DNS 服务商处把所有解析记录原样建好**，再到注册商把 nameserver 改回去即可（同样记得先关掉 Cloudflare 侧的 DNSSEC）。**切勿**在没有备份解析记录的情况下直接删掉 Cloudflare 上的 zone，否则域名会短暂无法解析、邮件网站一起中断。

### 2.2 DNS 记录管理

接入后，在 **域名 → DNS → Records** 里管理解析。常用记录类型：

| 类型 | 作用 | 能否代理（橙云） |
|---|---|---|
| **A** | 域名 → IPv4 地址 | ✅ 可 |
| **AAAA** | 域名 → IPv6 地址 | ✅ 可 |
| **CNAME** | 域名 → 另一个域名（别名） | ✅ 可 |
| **MX** | 指定收邮件的邮件服务器 | ❌ 否 |
| **TXT** | 任意文本：域名验证、**SPF/DKIM/DMARC** 邮件认证 | ❌ 否 |
| **NS** | 把某个子域委派给别的 nameserver | ❌ 否 |
| **CAA** | 限定哪些证书机构(CA)能给你签证书 | ❌ 否 |
| **SRV** | 指定服务的主机+端口（VoIP、IM 等） | ❌ 否 |
| **HTTPS/SVCB** | 现代连接提示（HTTP/3 等） | — |

操作要点：
- 每条记录含 **Type / Name / Content(目标) / Proxy(代理状态) / TTL**。**一旦开启代理，TTL 由 Cloudflare 自动接管**（显示为 Auto）。
- 支持**批量导入/导出**（BIND 区域文件格式），适合迁移大量记录。
- 关于邮件：**MX 记录本身没有橙云/灰云开关**（邮件不是 HTTP 流量，不存在"代理"一说）。真正要注意的是 MX 指向的那台**邮件主机的 A 记录**必须保持**灰云（仅 DNS）**，否则邮件端口连不通。

> 💡 **顺带一提：Cloudflare Email Routing（免费）**。如果你只想要一个 `you@yourdomain.com` 的"门面邮箱"、把来信**转发到你的 Gmail/QQ 邮箱**，又不想自建邮件服务器，可在 **域名 → Email → Email Routing** 一键开启。它会自动配好所需的 MX/TXT 记录，支持多个转发地址和 catch-all。注意它只负责**接收与转发**，对外发信仍需借助其他服务。

### 2.3 核心概念：橙云 vs 灰云（代理 vs 仅 DNS）

**这是整个 Cloudflare 里最重要的一个开关。** 每条 A/AAAA/CNAME 记录前面都有一朵云：

| | 🟠 **橙云 = Proxied（代理）** | ⚪ **灰云 = DNS only（仅 DNS）** |
|---|---|---|
| 流量是否经过 Cloudflare | **经过**它的边缘 | **直连**你的服务器 |
| 对外暴露的 IP | Cloudflare 的 IP（**隐藏真实 IP**） | **你服务器的真实 IP** |
| CDN 缓存 | ✅ 有 | ❌ 无 |
| WAF / DDoS / Bot 防护 | ✅ 生效 | ❌ 不生效 |
| 免费 SSL（边缘终止） | ✅ 自动 | ❌ 需自理 |
| HTTP 层分析数据 | ✅ 完整 | ❌ 几乎没有 |

**什么时候用橙云**：你的**网站 / Web 服务**（HTTP/HTTPS），想要加速、隐藏源站、上防护——**默认就该开橙云**。

**什么时候必须用灰云**：那条记录**不是网页流量**，例如：
- **邮件服务器**（MX 指向的主机）；
- **SSH / FTP / 数据库 / 游戏服务器**等非 HTTP 服务；
- 某些**第三方验证**用的解析。

> ⚠️ **灰云的代价**：你服务器的真实 IP 会暴露在公网 DNS 里，攻击者可以**绕过 Cloudflare 直接打你的源站**。如果一个本可代理的网站记录被设成灰云，等于白白放弃了 Cloudflare 的所有防护。上线后**务必确认你的网站记录是橙云**。

### 2.4 SSL/TLS 入门：免费证书与加密模式

**Universal SSL（通用 SSL）**：域名激活后，Cloudflare 会**自动、免费**给你签发受信任的 DV 证书，覆盖**根域 + 一级子域**（即 `example.com` 和 `*.example.com`，如 `www.example.com`），并**自动续期**。所有套餐都有。更深层子域（如 `a.b.example.com`）需要 **Total TLS** 或 **Advanced Certificate Manager**。

**加密模式（SSL/TLS → Overview）**——决定 "Cloudflare ↔ 你的源站" 这段怎么加密：

| 模式 | 浏览器↔CF | CF↔源站 | 校验源站证书 | 评价 |
|---|---|---|---|---|
| **Off** | ❌ | ❌ | — | 全程明文，**别用** |
| **Flexible** | ✅ HTTPS | ❌ 明文 | — | 仅在源站完全不能上 TLS 时凑合用；⚠️ 官方正逐步淘汰，**不安全** |
| **Full** | ✅ | ✅ | ❌ 不校验 | 可接受自签名证书，较好 |
| **Full (Strict)** | ✅ | ✅ | ✅ 校验 | **官方推荐、最安全** |

> ✅ **新手最佳实践**：给源站装一张证书（Let's Encrypt，或 Cloudflare 免费的 **Origin CA 证书**），然后把模式设为 **Full (Strict)**。
> 部署在 Cloudflare Pages/Workers 上的网站不用操心这一段（边缘内部已加密）。

**Always Use HTTPS（强制 HTTPS）**：在 **SSL/TLS → Edge Certificates** 打开，把所有 HTTP 请求在边缘自动跳转到 HTTPS。
> ⚠️ 打开它之后，**不要再在源站服务器上做一遍 HTTP→HTTPS 跳转**，否则会"无限重定向循环"。同页还有 **HSTS、Automatic HTTPS Rewrites、最低 TLS 版本** 等安全选项。

### 2.5 DNSSEC、子域名与高级 DNS

- **DNSSEC**：给 DNS 响应加密签名，防止解析被劫持/投毒。在 **DNS → Settings → Enable DNSSEC** 一键开启，Cloudflare 生成一条 **DS 记录**，你需要把它**手动填回原注册商**完成信任链。**例外**：若域名就在 **Cloudflare Registrar**（及 `.ch`/`.cz`），DS 记录会**自动**配好。
- **子域名**：直接加一条 Name 为子域（如 `blog`、`api`、`www`）的 A/AAAA/CNAME 即可，每个子域可独立选择橙云/灰云。
- **更深委派**：用 NS 记录把某个子域整体交给别的 DNS 托管。

### 2.6 Cloudflare Registrar：按成本价注册/转入域名

Cloudflare 自己也是**域名注册商**，最大卖点是 **"零加价（at-cost）"**：

- **注册价和续费价都等于注册局批发价 + ICANN 规费，Cloudflare 不赚差价**——没有"首年 1 元、续费 100 元"的套路，没有强制搭售。
- 支持 **390+ 个 TLD**（`.com`/`.net`/`.org`/`.io`/`.ai`/`.shop` 等，`.ai`、`.shop` 于 2025 年 3 月加入）。

**两个重要前提**：
1. **要先把域名接入 Cloudflare 的 DNS**（即先做完 [2.1](#21-完整接入流程onboarding-七步) 的 Full setup），Registrar 才能用——它和 Cloudflare DNS 是绑定的。
2. **转入（Transfer-in）有 ICANN 规则限制**：域名需注册/上次转移**满 60 天**；近 60 天没改过注册人信息。

**转入步骤（概要）**：
1. 确认域名已在 Cloudflare **Active**，并先**关闭 DNSSEC**。
2. 在**原注册商**：**解锁域名**、**获取转移授权码（EPP/Auth Code）**。
3. Cloudflare：**Domain Registration → Transfer Domains** → 选域名 → **填授权码** → 添加付款方式 → 填写注册人信息。
4. 邮箱里**批准**转移授权信。
- **时长**：实际操作约 30 分钟，整体完成最长约 **5–10 天**（取决于原注册商）。多数转入会**免费延长 1 年**有效期。

---

## 第 3 部分　入门·部署你的网站（构建 / Build 模块）

域名接好后，就该让它指向一个真正的网站了。Cloudflare 提供了**自带托管 + 全球加速 + 免费 SSL** 的一体化方案，**不需要你自己买服务器**。

### 3.1 先搞清楚：Pages 还是 Workers？

这是 2025 年最需要讲清楚的一个变化，否则容易被过时教程误导：

> ⭐ **官方当前明确建议：新项目从 Workers（配合"静态资源 / Static Assets"）开始，而不是 Pages。**
> 官方原话大意：*"既然 Workers 现在既能托管静态资源、又能做服务端渲染，你应该从 Workers 起步。Pages 会继续支持，但今后所有的投入、优化和新功能都集中在 Workers。"*

要点：
- **Pages 没有被废弃**，现有 Pages 项目继续正常运行、无需迁移；但它进入了"**只维护、不再加新功能**"的状态。
- **历史脉络**：2023 年 Pages 与 Workers 宣布融合；2024 年 9 月起 Workers 支持**静态资源托管**，从此一个 Workers 项目就能同时搞定"静态站 + 服务端渲染 + 后端 API"。
- **Pages 目前仍独有的少数能力**：原生文件路由、Early Hints、自定义分支别名，以及**在"DNS 不在 Cloudflare"的域名上挂自定义域名**（这是选 Pages 的唯一实际理由）。
- **已过时、遇到请忽略**：`Workers Sites`（被静态资源取代）、`wrangler publish`（改用 `wrangler deploy`）。

**给新手的实用建议**：
- 想**最省事、纯静态站/博客、用 GitHub 自动部署** → 走 **[路线 A：Pages + Git](#32-路线-apages--git最推荐的零运维方案)**（生态成熟、界面友好，依然是上手最快的）。
- 想**跟随官方未来主线、或要写后端逻辑/全栈** → 走 **[路线 C：Workers + 静态资源](#34-路线-cworkers--静态资源官方新项目首选)**。
- 域名的 **DNS 不在 Cloudflare**（即你**没做**第 2 部分的接入、域名仍托管在别处的 DNS——个人站较少见）→ 这种情况挂自定义域名**只能用 Pages**（Workers 的自定义域名要求该域名是 Cloudflare 上的 zone）。

### 3.2 路线 A：Pages + Git（最推荐的零运维方案）

**效果**：把代码推到 GitHub/GitLab，Cloudflare 自动构建并全球部署；以后每次 `git push` 自动更新，PR 还会生成预览链接。

1. 仪表盘 → **Workers & Pages** → **Create application / 创建应用** → **Pages** 选项卡 → **Connect to Git / 连接到 Git**。
2. 授权 GitHub 或 GitLab（GitLab 需对仓库有 **Maintainer 及以上**角色）→ 选择仓库 → **Begin setup**。
3. 在 **Set up builds and deployments** 填写：
   - **Project name**：决定你的默认网址 `<项目名>.pages.dev`。
   - **Production branch（生产分支）**：通常 `main`。其他分支会自动生成**预览部署**。
   - **Framework preset（框架预设）**：选你的框架，会自动带出下面两项。
   - **Build command（构建命令）**：如 `npm run build`（纯 HTML 无需构建可留空）。
   - **Build output directory（输出目录）**：构建产物目录，如 `dist` 或 `public`。
   - （可选）**Environment variables（环境变量）**：构建期用的键值对。
4. **Save and Deploy**。看着日志跑完（装依赖 → 构建 → 全球分发），几十秒后站点就上线在 `https://<项目名>.pages.dev`。

**常见框架的配置速查**：

| 框架 | Build command | Output directory |
|---|---|---|
| React (Vite) / Vue / Astro / Nuxt | `npm run build` | `dist` |
| **Hugo** | `hugo` | `public` |
| Gatsby | `npx gatsby build` | `public` |
| SvelteKit | `npm run build` | `.svelte-kit/cloudflare` |
| Next.js（在 Pages 上） | `npx @cloudflare/next-on-pages@1` | `.vercel/output/static` |
| 纯静态 HTML | （留空） | `/`（仓库根或站点目录） |

> 📘 **想要一份完全照做的端到端实操？** 见同目录下的[《实战：用 Hugo 搭建博客并部署到 Cloudflare》](/posts/build-blog-with-hugo-and-cloudflare/)——从装 Hugo、套主题、写第一篇文章，到推 GitHub、Pages 自动部署、绑定自己的域名，全程命令可复制粘贴。

> 💡 Pages 免费额度很大方：**站点数不限、静态资源的请求与带宽不限量**、每月 **500 次构建**、最多 100 个自定义域名。个人博客、文档站、企业官网完全够用。
> ⚠️ **Next.js 注意**：在 Pages 上需用 `@cloudflare/next-on-pages` 适配器；但官方更推荐在 **Workers** 上用 **OpenNext（`@opennextjs/cloudflare`）** 部署 Next.js，与"新项目用 Workers"一致。

### 3.3 路线 B：Pages 直接上传（拖拽 / Wrangler）

没有 Git、或只想把一个**已经构建好的文件夹**丢上去：

**拖拽方式**：**Workers & Pages → Create application → Pages → Upload assets** → 填项目名 → 把构建好的文件夹拖进上传框 → **Deploy site**。

**命令行方式**：
```bash
npx wrangler login                       # 浏览器授权登录
npx wrangler pages project create        # 交互式创建项目
npx wrangler pages deploy ./dist         # 把 dist 目录部署到生产
npx wrangler pages deploy ./dist --branch=preview   # 部署到预览环境
```
> ⚠️ 限制：Wrangler 上传单项目最多 **20,000 个文件**、单文件 ≤ **25 MiB**；网页拖拽最多 **1,000 个文件**。另外 **Direct Upload 项目事后无法改成 Git 集成**（需重建项目）。

### 3.4 路线 C：Workers + 静态资源（官方新项目首选）

用脚手架 **C3（`create-cloudflare`）** 一条命令起项目：

```bash
npm create cloudflare@latest -- my-site
```
交互里选：**Template → Static site（纯静态）** 或 **SSR / full-stack app（带后端）**；语言 TypeScript/JavaScript 随意。然后：

```bash
cd my-site
npx wrangler dev        # 本地预览，默认 http://localhost:8787
npx wrangler deploy     # 部署到 <worker名>.<你的子域>.workers.dev
```

**核心配置 `wrangler.jsonc`**（官方现在主推带注释的 jsonc，`wrangler.toml` 也支持）：
```jsonc
{
  "name": "my-site",
  "compatibility_date": "2026-05-29",   // 建议改成你创建项目当天的日期
  "main": "src/index.ts",        // 后端/SSR 入口；纯静态站可省略
  "assets": {
    "directory": "./dist",        // 静态资源目录（必填）
    "binding": "ASSETS",          // 可选：脚本里用 env.ASSETS.fetch() 取资源
    "not_found_handling": "single-page-application"  // SPA 用这个；普通站用 "404-page"
  }
}
```

路由规则：请求**命中静态文件就直接返回该文件、不跑脚本**；未命中才进入 Worker 脚本（可做 SSR / API）。SPA 模式下未命中会回 `index.html`（200）。

> 💰 **强力卖点**：**静态资源请求免费且不限量**，不消耗 Workers 请求额度；只有真正触发脚本（如 SSR）才按 Workers 计费。

### 3.5 绑定你自己的域名

默认网址是 `*.pages.dev` 或 `*.workers.dev`。把它换成你自己的域名：

**Pages 绑定自定义域名**：
1. 进入该 Pages 项目 → **Custom domains** 选项卡 → **Set up a domain**。
2. 输入域名（`example.com` 或 `www.example.com`）→ 按提示确认。
   - 域名**已在 Cloudflare 托管**：CNAME 会**自动添加**。
   - 域名**不在 Cloudflare**（DNS 在别处）：去你的 DNS 服务商加一条 `CNAME` 指向 `<项目名>.pages.dev`。
3. 想 `www` 和根域都能访问，就把两者都加为自定义域名（或在其中之一做重定向统一）。

> ⚠️ **常见 522 错误**：一定要走 Pages 面板里的 **"Set up a domain"** 流程；如果只在 DNS 处手动加 CNAME 而没在 Pages 里确认绑定，会报 522。

**Workers 绑定自定义域名**：
进入该 Worker → **Settings → Domains & Routes → Add → Custom Domain** → 输入域名。Cloudflare 会**自动建 DNS 记录并签发证书**。
- **Custom Domain**（整个域名/子域所有路径都给这个 Worker）vs **Route**（按路径匹配，如 `example.com/api/*`）。
- ⚠️ Workers 的自定义域名**要求该域名是 Cloudflare 上的 zone**；DNS 不在 Cloudflare 的域名只能用 Pages。

### 3.6 Wrangler CLI 速查

`wrangler` 是和 Workers/Pages 打交道的官方命令行工具（当前 4.x）：

```bash
# 创建新项目（C3 脚手架，会顺带装好 wrangler）
npm create cloudflare@latest
npm create cloudflare@latest -- my-app --framework=react   # 直接指定框架

npx wrangler login            # 浏览器 OAuth 登录
npx wrangler whoami           # 看当前登录账户
npx wrangler dev              # 本地开发服务器
npx wrangler dev --remote     # 在 Cloudflare 边缘上远程运行（Pages 不支持）
npx wrangler deploy           # 部署 Worker（含静态资源）到生产

# Pages 专用
npx wrangler pages project create
npx wrangler pages deploy <目录>
```
> 遇到老教程里的 `wrangler publish` 一律换成 `wrangler deploy`。

---

## 第 4 部分　进阶·构建（Build）开发者平台全景

Cloudflare 的"构建"远不止托管网页——它是一整套**在边缘运行的 serverless 平台**，让你不用管服务器就能写后端、存数据、跑 AI。下面每个产品配一句定位 + 典型用途：

**计算 / 运行时**
- **Workers**：全球边缘 serverless 运行时（基于 V8 isolate，**冷启动近乎为零**、不按"等待时间"计费）。用途：API、SSR、中间件、整套全栈应用。**新项目首选。**
- **Pages**：面向前端/全栈的部署平台（Git CI/CD、预览部署、Pages Functions）。现处"仅维护"。用途：现有静态/Jamstack 站。
- **Workers for Platforms**：让你为**你的终端用户**部署/运行他们自己的代码（多租户隔离）。用途：建站平台、可让客户写自定义逻辑的 SaaS。

**存储 / 数据库**
- **R2**：对象存储，**零出口流量费（zero egress）**、兼容 S3 API。用途：图片/视频/备份/大文件、数据湖、给 AI 喂数据。**省钱杀手锏。**
- **D1**：基于 SQLite 的 serverless 关系型数据库，与 Workers 原生集成。用途：中小应用的用户/内容数据。
- **KV**：全球分布、最终一致的键值存储，**读取极快**。用途：配置、功能开关、会话、缓存（读多写少）。
- **Durable Objects**："计算+强一致存储合一"的特殊 Worker，每个对象全局**唯一实例**，自带持久化与 WebSocket。用途：聊天室、协同编辑、多人游戏房间、实时状态协调、有状态 AI agent。
- **Hyperdrive**：给你**已有的外部数据库**（Postgres/MySQL）做连接池 + 全球缓存加速，让 Workers 访问远端库像本地一样快。

**消息 / 任务**
- **Queues**：托管消息队列，可靠投递、批处理、重试、死信，**无出口费**。用途：异步任务、削峰解耦、后台批处理。
- **Workflows**：在 Workers 上构建**持久化多步骤流程**（durable execution），自动重试、可暂停/恢复、可长时间运行。用途：文档/图像处理流水线、订单审批流、ETL。
- **Cron Triggers**：给 Worker 设**定时调度**（cron 表达式）。⚠️ 仅 Workers 支持。用途：定时抓取、清理、报表、缓存预热。

**AI**
- **Workers AI**：在 Cloudflare 的 **GPU 边缘网络**上直接跑 AI 模型（大语言模型、文生图、Embedding、语音等）。用途：聊天/补全、RAG、图像生成、内容审核。
- **Vectorize**：托管**向量数据库**。用途：配合 Workers AI 做语义搜索、RAG 检索、推荐。
- **AI Gateway**：给你调用各家 AI 的请求做**观测、缓存、限流、治理**。
- 相关还有 **Stream**（视频流）、**Images**（图片存储/优化）、**Realtime**（实时音视频，原 Calls）、**Containers**（serverless 容器，跑塞不进 isolate 的重负载）、**Pipelines**（实时数据流写入 R2）。

> 🧭 **进阶组合示例**：一个"带评论的博客全栈应用"可以这样搭——前端静态资源放 Workers 静态托管；评论 API 用 Workers 脚本；评论数据存 **D1**；图片存 **R2**；热门文章列表缓存进 **KV**；"每天凌晨生成订阅摘要"用 **Cron Triggers + Queues + Workflows**；"智能摘要/相关推荐"用 **Workers AI + Vectorize**。全部 serverless、零服务器运维。

---

## 第 5 部分　保护（Protect）：安全模块详解

"保护"对应仪表盘里的 **Security（安全）** 板块（2024–2025 做过改版，现把安全总览、分析、规则、资产发现统一到一处），以及独立控制台里的 **Zero Trust**。下面按"是什么 / 在哪 / 哪个套餐"梳理。

### WAF（Web 应用防火墙）　位置：Security → WAF
按规则拦截 SQL 注入、XSS、路径穿越等攻击：
- **Free Managed Ruleset（免费托管规则集）**：抵御高危、广泛被利用的漏洞——**免费套餐唯一的托管规则集**。
- **Cloudflare Managed Ruleset + OWASP Core Ruleset**：更全面的托管防护与 OWASP 评分模型——**Pro/Business/Enterprise**。
- **Custom rules（自定义规则）**：按 IP、路径、请求头、请求体等条件自定义"拦截/质询/放行"——**所有套餐**可用（高阶字段如 WAF attack score、恶意上传检测偏向付费）。
- **Rate limiting（限速规则）**：超过阈值就限流——**Free 仅 1 条**；Pro/Business 更多；**Advanced Rate Limiting 为 Enterprise 加购**。

### DDoS 防护　位置：Security → DDoS
**全套餐（含 Free）自动、不限量地防护 L3–L7 的 DDoS 攻击**——这是 Cloudflare 的招牌能力。高级自适应防护（结合国家、UA、机器学习评分）属 **Advanced DDoS（Enterprise 加购）**。

### 机器人管理　位置：Security → Bots
| 产品 | 套餐 | 能力 |
|---|---|---|
| **Bot Fight Mode** | Free | 一个开关，挑战已识别的恶意机器人 |
| **Super Bot Fight Mode** | Pro / Business | 按机器人类别配置动作、保护静态资源、看分析 |
| **Bot Management** | Enterprise（加购） | 机器学习**逐请求打分(1–99)**、自定义规则、精细分析 |

> 🤖 **AI 爬虫控制（2024–2025 热点）**：**"Block AI bots"一键开关全套餐可用**（含 Free），可一键拦截 AI 爬虫；还有 **AI Crawl Control（原 AI Audit）** 能看清哪些 AI 在抓你的内容，甚至支持 **pay-per-crawl（按抓取收费）**。

### Turnstile（人机验证，CAPTCHA 替代品）
免费、隐私友好的"无感"人机验证，多数情况下用户**看不到拼图**。**不要求你的网站托管在 Cloudflare**，任何站点都能嵌。免费额度：每账户 **20 个 widget、验证次数不限**。

### Page Shield（客户端 / 供应链安全）　位置：Security → Page Shield
监控页面加载的**第三方 JavaScript**，防"Magecart"式供应链攻击（被篡改的脚本在浏览器里窃取支付/表单数据）。提供脚本/连接/Cookie 监控、恶意脚本检测、CSP 式白名单。**全套餐有基础脚本监控**，深度功能随套餐递增。

### SSL/TLS 安全设置　位置：SSL/TLS → Edge Certificates
除了 [2.4](#24-ssltls-入门免费证书与加密模式) 的加密模式，这里还有 **HSTS、最低 TLS 版本、TLS 1.3、Automatic HTTPS Rewrites、证书透明度监控** 等。

### Security Center（安全中心，账户级）
把安全产品 + 威胁情报 + 配置体检整合到一处：**Security Insights**（扫 DNS/SSL/WAF/Access 配置找隐患）、**Attack Surface Management**（发现暴露在公网的资产）、**Investigate Portal**（查 IP/域名情报）、**Brand Protection**（检测仿冒/抢注域名，beta）。**全套餐可用。**

### 其他
- **API Shield**：保护 REST/GraphQL API（Schema 校验、mTLS 客户端证书、滥用检测）——多为 **Enterprise 加购**。
- **Email Security（原 Area 1）**：在 **Zero Trust 控制台**里，AI + 威胁情报扫描每封入站邮件，防钓鱼/恶意软件/BEC 商务邮件诈骗——**主要为付费/企业级**。
- **Access & Gateway**：零信任，归在 [第 6 部分·连接](#第-6-部分连接connect网络模块详解) 讲。

> ✅ **新手安全起步配置**（Free 套餐即可）：① 加密模式设 **Full (Strict)** + 打开 **Always Use HTTPS**；② 确认 **DDoS** 与 **Free WAF 托管规则**已生效（默认开）；③ 打开 **Bot Fight Mode** 和 **Block AI bots**；④ 给登录/评论等表单接入 **Turnstile**；⑤ 用 **Security Center** 跑一次配置体检。

---

## 第 6 部分　连接（Connect）：网络模块详解

"连接"是把**你的服务器、内网、办公室、家庭设备**安全接入 Cloudflare 网络的一组产品。其中**零信任（Zero Trust / Cloudflare One）有独立控制台** `one.dash.cloudflare.com`。

### ⭐ Cloudflare Tunnel（最适合"家里/内网自托管"）

**痛点**：想把家里/公司内网的服务（NAS、博客、Home Assistant…）发布到公网，传统做法要**开路由器端口映射 + 暴露公网 IP**，既危险又常因**没有公网 IP / 运营商 NAT** 而做不到。

**Tunnel 的解法**：在你的服务器上跑一个轻量守护进程 **`cloudflared`**，它**只向外**主动建立一条加密隧道连到 Cloudflare。之后外部访问通过这条已建立的隧道反向回到你的服务。**不需要公网 IP、不需要开放任何入站端口、源站攻击面几乎为零**——你甚至可以把防火墙设成"禁止一切入站"。

**可照做的步骤（本地管理的命名隧道，Linux/macOS 示例）**：
```bash
# 1. 安装 cloudflared（各平台装法见官方文档；示例）
#    macOS:          brew install cloudflared
#    Debian/Ubuntu:  从 Cloudflare 包仓库 apt 安装 cloudflared

# 2. 授权到你的 Cloudflare 账户（浏览器里选择你的域名）
cloudflared tunnel login

# 3. 创建命名隧道（生成 UUID 与凭据文件 ~/.cloudflared/<UUID>.json）
cloudflared tunnel create my-tunnel

# 4. 把公网域名解析到这条隧道（自动在 DNS 里加好 CNAME）
cloudflared tunnel route dns my-tunnel app.example.com

# 5. 写 ingress 规则到 ~/.cloudflared/config.yml：
#    --------------------------------------------------
#    tunnel: my-tunnel
#    credentials-file: /root/.cloudflared/<UUID>.json
#    ingress:
#      - hostname: app.example.com
#        service: http://localhost:8080
#      - service: http_status:404         # 兜底规则，必须有
#    --------------------------------------------------

# 6. 先前台跑通验证
cloudflared tunnel run my-tunnel

# 7. 验证 OK 后，装成开机自启的后台服务
cloudflared service install
```
之后访问 `https://app.example.com`，流量就会经 Cloudflare 边缘、通过隧道回到你本地的 `localhost:8080`——**全程没有开放任何入站端口**。

管理方式二选一：上面是**本地配置文件 `config.yml`**（可控、适合 IaC）；新手也可改用 **Dashboard 远程管理**（在 Zero Trust 控制台里点选 hostname 与 service，免写 YAML）。**免费、全套餐可用**；只想临时演示可用 **TryCloudflare**：`cloudflared tunnel --url http://localhost:8080`，秒得一个临时 `*.trycloudflare.com` 网址，免账户、免配置。

> 🏠 **黄金组合：Tunnel + Access**。用 Tunnel 把家里的服务安全暴露出去，再用 Access 要求"必须用指定邮箱/Google 账号登录才能访问"——这是"无公网 IP 在家自托管，还要保证只有自己能访问"的标准答案。

### WARP / 1.1.1.1
- **1.1.1.1**：Cloudflare 免费、注重隐私的**公共 DNS 解析器**（支持 DoH/DoT）。
- **WARP**：类 VPN 客户端（基于 WireGuard），把设备流量加密走 Cloudflare 网络；企业场景里它是把设备纳入零信任策略的**设备代理**（现整合进 **Cloudflare One Agent** 品牌）。

### Zero Trust / Cloudflare One（SASE）
Cloudflare 的 **SASE 安全访问平台**，用"身份 + 最小权限"取代传统 VPN/边界。核心组件：
- **Access**（ZTNA）：对内部应用/SaaS 做**身份感知访问**，每次请求都校验，可对接 Google/Okta/Azure AD 等身份源。
- **Gateway**（SWG）：DNS/HTTP/网络层过滤与检查。
- 还有 **CASB、DLP（数据防泄露）、Browser Isolation（远程浏览器隔离）、Email Security**。
- **价格**：**50 用户以内免费**，超出约 **$7/席/月**（按量）或企业定制。⚠️ 以官方零信任套餐页为准。

### 其他网络产品
- **Spectrum**：把 DDoS 防护和代理扩展到**任意 TCP/UDP 应用**（SSH、RDP、游戏服、邮件等），隐藏源站 IP。**付费套餐**（Pro/Business 限部分协议，Enterprise 全端口）。
- **Argo Smart Routing**：实时选**最快、最不拥堵的路径**走 Cloudflare 私有骨干网，提速。**按量加购**。
- **Load Balancing（负载均衡）**：多源站之间分发流量，含**健康检查、故障转移、地理/延迟调度、会话保持**。**按量加购**。
- **Magic Transit / Magic WAN**：面向企业的**网络层（L3）DDoS 防护**与 **SD-WAN/网络即服务**，把整个数据中心、分支机构、云接入 Cloudflare。**Enterprise。**
- **Cloudflare for SaaS（自定义主机名）**：让 **SaaS 厂商**把 Cloudflare 的 SSL/CDN/安全延伸到**客户自己的域名**上（客户用 `app.客户域名.com` 指向你的服务）。

---

## 第 7 部分　观察（Observe）：分析与日志模块详解

"观察"对应仪表盘里的 **Analytics & Logs（分析与日志）**，帮你看清"谁在访问、快不快、有没有被攻击、出没出错"。

- **Web Analytics**：**免费、隐私优先**的网站分析。**不用 Cookie、不抽样**，统计 100% 页面访问，含 **Core Web Vitals 核心性能指标**。靠一段 JS 探针工作，**不要求网站托管在 Cloudflare**——是"隐私友好版 Google Analytics"。
- **账户 / 区域 Analytics**：内置仪表盘，覆盖**流量、安全事件、性能、缓存命中、DNS、Workers** 等。全套餐可用，**数据保留时长和粒度随套餐升级而增加**。
- **Logs（日志）**：
  - **Logpush**：把详细日志**批量推送**到 R2/S3/GCS/Azure 或 Splunk/Datadog 等（zone HTTP 日志主要面向 Enterprise）。
  - **Logpull**：用 API 拉取 HTTP 请求日志（Enterprise，较老）。
  - **Instant Logs**：在仪表盘/CLI **实时**看请求流（Enterprise）。
  - 注意：**Workers 的日志（Trace Events Logpush）在 Workers 付费版即可用**，不必买企业版 zone。
- **Workers 可观测性**：**Workers Logs**（自动收集、Query Builder 查询）、`wrangler tail`（near-real-time 实时日志）、**Traces**（导出 OpenTelemetry 到 Honeycomb/Grafana/Sentry 等）、**Analytics Engine**（从 Worker 写自定义时序数据并用 SQL 查询）。
- **Health Checks（健康检查）**：从 Cloudflare 边缘**主动探测**你的源站 IP/主机，宕机近实时告警（即使只有一个源站、没上负载均衡也能用）。**Pro/Business/Enterprise**。
- **Trace**：模拟一个请求，看它依次被哪些规则/产品（WAF、重定向、缓存、Workers）处理——**排查配置神器**。
- **Network Analytics / GraphQL Analytics API**：L3/4 流量与 DDoS 可视化（Magic Transit/Spectrum/企业）；GraphQL API 则是所有分析数据的统一出口，可对接 Grafana 做自定义大盘。

---

## 第 8 部分　进阶补充：性能、缓存与规则引擎

这部分不属于你点名的五大板块，但对"托管域名 + 部署网站"极其实用，放在区域级设置里（Caching / Rules / Speed）。

### 缓存（Caching）
开启橙云代理后，Cloudflare 会在边缘**缓存静态资源**（图片、CSS、JS 等），减轻源站压力、加速访问。
- **Cache Rules（缓存规则）**：按 URL/文件类型等精细控制"缓存什么、缓存多久（Edge TTL）、是否绕过缓存"。是现在推荐的缓存控制方式。
- **Tiered Cache（分层缓存）**：让边缘节点之间先互相找缓存，进一步降低回源率。
- 需要更新时可在 **Caching → Configuration** 做 **Purge Cache（清除缓存）**（可全部清或按 URL 清）。

### 规则引擎（Rules）
Cloudflare 把老的 **Page Rules（页面规则）** 逐步替换为更强的**模块化 Rules**（老页面规则仍可用但功能受限，新项目建议用下面这些）：
- **Redirect Rules**：URL 重定向（含批量重定向 Bulk Redirects），如 `http→https`、`naked 域→www`。
- **Transform Rules**：改写 URL、增删改请求/响应头。
- **Origin Rules**：改写回源时的 Host、端口、SNI 等。
- **Configuration Rules**：对匹配的请求动态调整 Cloudflare 设置。
- **Compression Rules / Cache Rules**：压缩与缓存策略。
- **Snippets**：在边缘跑一小段 JavaScript（像轻量版 Workers），用来实现规则做不到的自定义逻辑。

### 速度优化（Speed）
- **Polish**（图片压缩，Pro+）、**Mirage**（移动端图片优化）、**Rocket Loader**（异步加载 JS）、**Early Hints**、**HTTP/3、0-RTT、Brotli 压缩**等。
- ⚠️ 注意：**Auto Minify（自动压缩 HTML/CSS/JS）已于 2024 年下线**——官方建议在你自己的构建流程里做压缩。遇到老教程让你"开 Auto Minify"，忽略即可。

> ⚠️ 本部分功能名与归属位置 Cloudflare 仍在持续调整，请以仪表盘当前的 **Caching / Rules / Speed** 实际选项为准。

---

## 第 9 部分　套餐与价格

> ⚠️ **2025 年价格有上调**（约十二年来首次），且新增**年付约 8 折**。以下为参考，动手前请核对 [cloudflare.com/plans](https://www.cloudflare.com/plans/)。

**域名（zone）套餐**：

| 套餐 | 月付 | 年付（折合/月） | 适合 |
|---|---|---|---|
| **Free** | **$0** | $0 | 个人站、博客、练手 |
| **Pro** | **约 $25** | 约 $20 | 专业个人 / 小型站点 |
| **Business** | **约 $250** | 约 $200 | 中小企业（含 100% 在线 SLA） |
| **Enterprise** | 定制 | 定制 | 大型组织、专属支持 |

所有套餐都白送：免费 DNS、不限量 DDoS、CDN、Universal SSL、基础 WAF、Bot Fight Mode、Block AI bots。**Pro** 加全套 WAF 托管规则 + OWASP + Super Bot Fight Mode + 图片优化；**Business** 加 SLA + 高级 Page Shield + 自定义 SSL；**Enterprise** 加 Bot Management、高级限速、高级 DDoS、日志推送、专属支持。

**开发者平台价格要点**：
- **Workers**：免费版每天 10 万次请求；付费版 **$5/月起**，含每月 1000 万次请求（超出 $0.30/百万）。**2024 起只按 CPU 时间计费**（等待 I/O 不计费），**永不收出口带宽费**。
- **R2**：存储约 **$0.015/GB·月**，**出口流量 $0 免费**（对比 S3 的最大优势）；免费额度 10GB 存储。
- **Pages**：免费额度很大（站点/带宽不限、每月 500 次构建、最多 100 个自定义域名）。
- **Registrar**：**按成本价**，注册/续费零加价。
- **Turnstile / Zero Trust**：Turnstile 免费（20 个 widget）；Zero Trust **50 用户内免费**，超出约 $7/席/月。

---

## 第 10 部分　排错 FAQ：最常见的几个坑

| 现象 | 大概率原因 | 解决 |
|---|---|---|
| **接入后邮件收不到/发不出** | 自动扫描漏了 **MX / SPF·DKIM·DMARC(TXT)** 记录 | 对照原 DNS 补齐记录；MX 主机保持**灰云** |
| **ERR_TOO_MANY_REDIRECTS（无限重定向）** | 加密模式选了 **Flexible**，源站又强制跳 HTTPS | 改 **Full (Strict)**；源站别再做 HTTP→HTTPS 跳转 |
| **绑定自定义域名后报 522** | 只在 DNS 加了 CNAME，没在 Pages 面板**确认绑定** | 走 Pages 项目里的 **Set up a domain** 流程 |
| **521 / 站点打不开** | 源站宕机或防火墙挡了 Cloudflare 回源 IP | 确认源站存活；放行 Cloudflare IP 段 |
| **改了内容但页面不更新** | 边缘**缓存**还在 | **Purge Cache** 清缓存；用 Cache Rules 调 TTL |
| **改完 nameserver 域名打不开** | 切换前没关原注册商的 **DNSSEC** | 在注册商关闭 DNSSEC，等待重新生效 |
| **网站没被加速/没防护** | 网站记录被设成了**灰云** | 把网站的 A/CNAME 记录切回**橙云** |
| **nameserver 一直不激活** | 注册商处没改对/未保存，或仍在传播 | 核对两个 ns 拼写；等待（最长 24h）；点"立即检查" |

---

## 第 11 部分　学习路径与官方资源

**推荐学习顺序**：
1. **第 1 天**：注册账户 + 开 2FA → 按 [第 2 部分](#第-2-部分入门托管你的域名域名--domains-模块) 把一个域名接入 → 设好 SSL（Full Strict + Always Use HTTPS）。
2. **第 2 天**：按 [第 3 部分](#第-3-部分入门部署你的网站构建--build-模块) 用 Pages 或 Workers 部署一个静态站 → 绑定自定义域名。
3. **第 3 天**：按 [第 5 部分末尾](#第-5-部分保护protect安全模块详解) 配齐基础安全 → 在 [第 7 部分](#第-7-部分观察observe分析与日志模块详解) 打开 Web Analytics 看数据。
4. **进阶**：玩 Workers + D1/R2/KV 写个小全栈应用（[第 4 部分](#第-4-部分进阶构建build开发者平台全景)）；用 Tunnel + Access 把家里的服务安全发布（[第 6 部分](#第-6-部分连接connect网络模块详解)）。

**官方资源**（均为权威一手来源）：
- 文档总站：[developers.cloudflare.com](https://developers.cloudflare.com/)
- 域名接入：[developers.cloudflare.com/fundamentals/manage-domains/add-site/](https://developers.cloudflare.com/fundamentals/manage-domains/add-site/)
- Pages 上手：[developers.cloudflare.com/pages/](https://developers.cloudflare.com/pages/)
- Workers 上手：[developers.cloudflare.com/workers/](https://developers.cloudflare.com/workers/)
- Workers 静态资源：[developers.cloudflare.com/workers/static-assets/](https://developers.cloudflare.com/workers/static-assets/)
- SSL 模式：[developers.cloudflare.com/ssl/origin-configuration/ssl-modes/](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/)
- Cloudflare Tunnel：[developers.cloudflare.com/cloudflare-one/connections/connect-networks/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- 套餐价格：[cloudflare.com/plans](https://www.cloudflare.com/plans/)
- 学习中心：[cloudflare.com/learning](https://www.cloudflare.com/learning/)
- 状态页（故障查询）：[cloudflarestatus.com](https://www.cloudflarestatus.com/)

---

## 附录 A　命令速查表

```bash
# —— 创建项目 ——
npm create cloudflare@latest                 # 交互式脚手架（C3）
npm create cloudflare@latest -- my-app --framework=react

# —— 登录与身份 ——
npx wrangler login                           # 浏览器授权登录
npx wrangler whoami                          # 查看当前账户
npx wrangler logout

# —— 开发与部署（Workers）——
npx wrangler dev                             # 本地开发（localhost:8787）
npx wrangler dev --remote                    # 边缘远程开发
npx wrangler deploy                          # 部署到生产（含静态资源）

# —— Pages ——
npx wrangler pages project create
npx wrangler pages deploy ./dist             # 部署到生产
npx wrangler pages deploy ./dist --branch=preview

# —— Cloudflare Tunnel ——
cloudflared tunnel login                     # 授权（浏览器选域名）
cloudflared tunnel create my-tunnel          # 创建命名隧道
cloudflared tunnel route dns my-tunnel app.example.com   # 把域名指向隧道
cloudflared tunnel run my-tunnel             # 前台运行（验证）
cloudflared service install                  # 装成开机自启服务
cloudflared tunnel --url http://localhost:8080   # 或：TryCloudflare 临时隧道（免账户）
```

## 附录 B　新手上线检查清单

- [ ] 账户已开启 **两步验证（2FA）**
- [ ] 域名状态为 **Active**，两个 Cloudflare nameserver 已在注册商生效
- [ ] **DNS 记录核对完整**（尤其 MX + SPF/DKIM/DMARC，邮件没坏）
- [ ] 网站记录是 **橙云**；邮件/SSH 等非网页记录是 **灰云**
- [ ] SSL 加密模式 = **Full (Strict)**，**Always Use HTTPS** 已开
- [ ] 网站已部署（Pages/Workers），**自定义域名通过面板正确绑定**（不是只加了 CNAME）
- [ ] 浏览器访问 `https://你的域名` 正常、有锁标、无重定向循环
- [ ] 基础安全：DDoS/WAF 默认开、**Bot Fight Mode** + **Block AI bots** 已开
- [ ] **Web Analytics** 已开启，能看到访问数据

## 附录 C　术语表

| 术语 | 含义 |
|---|---|
| **Zone（区域）** | Cloudflare 里的"一个域名"及其全部设置 |
| **Nameserver（NS）** | 负责回答你域名解析的权威服务器；接入即把它换成 Cloudflare 的 |
| **代理 / 橙云 (Proxied)** | 流量经过 Cloudflare，启用缓存/防护/隐藏源站 |
| **仅 DNS / 灰云 (DNS only)** | Cloudflare 只解析、不代理，流量直连源站 |
| **源站 (Origin)** | 你真正放网站/服务的服务器 |
| **回源 (Origin pull)** | Cloudflare 边缘未命中缓存时，去源站取内容 |
| **边缘 (Edge)** | Cloudflare 遍布全球的就近数据中心节点 |
| **Universal SSL** | Cloudflare 免费自动签发并续期的证书 |
| **Wrangler** | 操作 Workers/Pages 的官方命令行工具 |
| **C3** | `create-cloudflare`，一键创建 Workers/Pages 项目的脚手架 |
| **Zero Trust / Cloudflare One** | Cloudflare 的零信任/SASE 平台，独立控制台 |

---

> 📌 **最后提醒**：Cloudflare 的仪表盘界面、菜单名、套餐价格更新频繁。当本文与你看到的实际界面不一致时，**以官方文档与你登录后的真实仪表盘为准**——但本文讲的**底层概念与操作主线**（接入域名 = 换 nameserver、橙云=代理、Full Strict + 强制 HTTPS、Pages/Workers 部署、Tunnel 自托管）是稳定不变的，掌握它们就能以不变应万变。
>
> 祝你玩得顺利！🚀
