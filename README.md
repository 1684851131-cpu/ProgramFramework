[README.md](https://github.com/user-attachments/files/31625553/README.md)
[README.md](https://github.com/user-attachments/files/31625241/README.md)
[README.md](https://github.com/user-attachments/files/31619458/README.md)
[README.md](https://github.com/user-attachments/files/31619127/README.md)
# Make your agent smarter, Chain Retail WeChat Mall Mini Program · Tutorial from Scratch（Agent Skill）

一份**教程型 AI Agent Skill**：把"连锁门店 + 线上商城"微信小程序的完整全栈架构，
浓缩成一份可直接执行的复刻教程。给 AI 编程助手（Hermes / Claude Code / Codex 等）
装上它，就能从零搭出一个生产级的同类系统。

> 纯教程，不含任何真实业务数据、账号、密钥。# 连锁零售微信商城小程序 · 从零复刻教程（Agent Skill）

> **English version below · 英文版见本文档下半部分**

一份**教程型 AI Agent Skill**：把"连锁门店 + 线上商城"微信小程序的完整全栈架构，
浓缩成一份可直接执行的复刻教程。给 AI 编程助手（Hermes / Claude Code / Codex 等）
装上它，就能从零搭出一个生产级的同类系统。

**纯教程，不含任何真实业务数据、账号、密钥。**

---

## 目录

- [这个系统长什么样](#这个系统长什么样)
- [目标系统的功能全景](#目标系统的功能全景)
- [教程覆盖内容](#教程覆盖内容)
- [教程结构（8 步）](#教程结构8-步)
- [使用方式](#使用方式)
- [适用场景](#适用场景)
- [文件清单](#文件清单)
- [License](#license)

## 这个系统长什么样

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储
                → 微信支付V3 / 同城跑腿API / 订阅消息 / 内容安全
H5品牌站+管理后台(React+Vite) → PHP中转 → 云函数网关
门店Windows打票机 ← 本地打印服务(8899) ← 后台发现新订单自动出票
```

**技术选型**：原生微信小程序（非 Taro/uni-app）+ 微信云开发 CloudBase
（云函数 + 云数据库 + 云存储），无自建后端服务器；H5 用 React 18 + Vite +
Tailwind 构建为静态站，配合一个轻量 PHP 中转对接云函数网关。

**为什么是这个栈**：单开发者/小团队可维护；云开发免运维；前端无构建直接传
微信开发者工具；门店打印走 Windows 本地服务，不依赖云。

## 目标系统的功能全景

| 端 | 功能 |
|---|---|
| C 端小程序 | 多门店定位选购、双级分类+拼音搜索、限时价/整箱规格、跨店购物车、优惠券、微信支付、自提/配送/同城跑腿、订单状态机、核销码自提、评价（内容安全审核）、会员 3 级、积分商城、收藏/足迹、退换货、榜单/活动/公告 |
| 门店端（小程序内） | 店员 30s 轮询接单、备货/配送状态流转、核销码+拍照核销、店长管库存/打烊/店员/退货审核 |
| 超管后台（H5） | 商品 CRUD + CSV 批量导入导出、订单/销售报表、库存、退货、店员、门店开关、公告/活动/广告/榜单、优惠券、积分商品、提货卡全生命周期（制卡/激活/核销/流水/导出）、发票 |
| 周边系统 | H5 品牌站（门店地图）、服务号消息通知、订阅消息、门店热敏小票自动打印 |

## 教程覆盖内容

| 模块 | 内容 |
|---|---|
| 架构 | 目录骨架、系统拓扑、技术选型与理由 |
| 云函数 | 30 个函数清单 + 每个的 action 路由表（后台/商品/库存/支付/订单/店员/退货/积分/评价/跑腿/订阅消息/提货卡/发票……），统一 `exports.main → switch(action)` 写法约定 |
| 数据库 | 21 个集合设计及权限要点（订单仅创建者可读写 → 店员端必须云函数中转） |
| 业务规则 | 价格三分支、库存语义、跨店购物车、权限模型、会员积分、退款幂等、评价防重、图片永久直链、跑腿计费云端复算、提货卡状态机、拼音搜索幂等增强、公开接口 pub* 模式 |
| 页面 | 31 个页面分组规格（5 Tab + 交易链 + 个人中心 + 门店管理端 + 隐藏页） |
| 周边 | H5 品牌站+管理后台、门店 ESC/POS 热敏打印服务（含 Windows 运维坑） |
| 部署 | 云函数依赖顺序、环境变量分组清单、微信支付 V3、订阅消息、服务器注意事项 |
| 避坑 | 15+ 条真实踩坑记录（wxss 括号配平、云库 limit 100 分页、_.in 限 20、bat 中文 GBK、8899 端口冲突……） |

## 教程结构（8 步）

1. **目录骨架** — app.json 要点与全局层
2. **云函数** — 30 个函数的 action 清单与职责
3. **云数据库** — 21 集合与权限设计
4. **核心业务规则** — 14 条"改一处全盘皆错"的规则
5. **页面** — 31 页分组规格
6. **H5 品牌站 + 管理后台** — React 构建与 PHP 中转
7. **门店打印服务** — ESC/POS 自动出票
8. **部署与配置** — 环境变量、支付、上线检查

## 使用方式

**作为 Hermes Skill**：把 `SKILL.md` 放到 `skills/wechat/wechat-chain-mall-rebuild/`
目录下，对 Agent 说"照教程搭一个连锁商城小程序"即可触发。

**作为 Claude Code / Codex Skill**：放入项目的 `.claude/skills/` 或 `.agents/skills/`
目录，同样按名称调用。

**纯人工阅读**：直接读 `SKILL.md`，按"第一步~第八步"顺序执行即可。

## 适用场景

- 想从零做一个多门店零售商城小程序的开发者
- 需要把现有小程序项目克隆到新 AppID / 新云环境的团队
- 学习"小程序 + 云开发 + H5 后台 + 本地打印"全栈架构的参考样本
- 作为 AI 编码 Agent 的领域知识包，避免每次重新解释业务规则

## 文件清单

```
SKILL.md      教程本体（中文，正本）
SKILL.en.md   English version of the skill
README.md     仓库说明（中英双语，本文件）
```

## 直接喂给 Agent 的提示词（复制即用）

把下面这段连同 `SKILL.md` 一起丢给任意 AI 编程助手（Hermes / Claude Code /
Codex / Cursor……），它就能开工：

```
你是一个微信小程序全栈工程师。请严格按照附件 SKILL.md 的教程，
从零搭建一个"连锁门店 + 线上商城"微信小程序（云开发版）。

要求：
1. 严格按教程第一步~第八步的顺序执行，每完成一步向我汇报，经我确认后再继续
2. 30 个云函数按教程给的依赖顺序编写，统一 exports.main → switch(action) 结构
3. 14 条核心业务规则逐条落实，不得自行简化或"优化"
4. 业务数据（商品/门店/卡库）先问我拿，拿不到就建空集合+示例数据占位，
   并在交付清单里标注哪些是占位
5. 密钥/AppID/商户号等敏感配置只留环境变量占位符，由我自行填入
6. 每批页面写完后说明如何在微信开发者工具里编译验证
7. 避开教程"易错点"清单里的所有坑

先从第一步"目录骨架 + app.json + 全局层"开始。
```

要**复制一套已有项目**（场景：你手上有一套正在运营的同类小程序，想用新的
AppID + 新云环境再开一套一模一样的），改用这段：

```
请阅读附件 SKILL.md 教程。我手上有一套正在运营的同类微信小程序
（老项目，路径在 <老项目路径>），现在要用新的 AppID + 新云环境
复制一套一模一样的新系统。请按以下顺序执行：
1. 先通读老项目的目录结构，与教程的目录/云函数/集合清单对比，列出差异给我确认
2. 从老项目导出商品/门店/会员等业务数据，导入新云环境（商品用
   importProducts 或 CSV 管线，门店写进 stores.js）
3. 老项目的云函数代码逐函数迁入新环境（保留全部 action 和环境变量名），
   全局替换三处标识：AppID、云环境 ID、图片直链域名
4. 31 个页面逐个迁入，每迁 5 个在微信开发者工具编译一次
5. 全链路联调：下单→支付→接单→核销→退款→积分→评价
所有密钥/商户号/账号列成清单让我手动填，不要写进代码或文档。
```

## License

MIT

---
---

# Chain-Retail WeChat Mall Mini Program · From-Scratch Rebuild Tutorial (Agent Skill)

A **tutorial-style AI Agent Skill** that distills the complete full-stack
architecture of a "chain stores + online mall" WeChat mini program into a
directly executable rebuild tutorial. Install it into any AI coding assistant
(Hermes / Claude Code / Codex …) and it can build a production-grade system of
the same kind from zero.

**Pure tutorial — contains no real business data, accounts, or secrets.**

## Table of Contents

- [What the System Looks Like](#what-the-system-looks-like)
- [Feature Overview](#feature-overview)
- [What the Tutorial Covers](#what-the-tutorial-covers)
- [Tutorial Structure (8 Steps)](#tutorial-structure-8-steps)
- [Usage](#usage)
- [Use Cases](#use-cases)
- [Files](#files)
- [License](#license-1)

## What the System Looks Like

```
Customer (WeChat Mini Program) → Cloud Functions (30) → Cloud DB (21 collections) + Cloud Storage
                               → WeChat Pay V3 / local courier API / subscribe messages / content safety
H5 brand site + admin (React+Vite) → PHP relay → cloud function gateway
In-store Windows receipt printer ← local print service (8899) ← auto-prints new orders
```

**Tech choices**: native WeChat Mini Program (no Taro/uni-app) + WeChat
CloudBase (cloud functions + cloud DB + cloud storage) — no self-hosted
backend; the H5 side is React 18 + Vite + Tailwind built to a static site with
a thin PHP relay to the cloud-function gateway.

**Why this stack**: maintainable by a solo dev or small team; CloudBase is
ops-free; the mini program has no build step (straight into WeChat DevTools);
in-store printing runs as a Windows-local service with no cloud dependency.

## Feature Overview

| Side | Features |
|---|---|
| Customer mini program | Multi-store location-based shopping, two-level categories + pinyin search, flash pricing / case specs, cross-store cart, coupons, WeChat Pay, pickup / delivery / local courier, order state machine, verification-code pickup, reviews (content-safety checked), 3-tier membership, points mall, favorites/history, returns & exchanges, rankings/activities/notices |
| Store console (in mini program) | Staff 30s polling for new orders, prep/delivery status flow, verification code + photo confirmation, manager inventory/close-store/staff/return-audit |
| Super admin (H5) | Product CRUD + CSV bulk import/export, orders/sales reports, inventory, returns, staff, store open/close, notices/activities/ads/rankings, coupons, points products, gift-card full lifecycle (create/activate/redeem/transactions/export), invoices |
| Peripheral systems | H5 brand site (store map), service-account notifications, subscribe messages, in-store thermal receipt auto-printing |

## What the Tutorial Covers

| Module | Contents |
|---|---|
| Architecture | Directory skeleton, system topology, tech choices and rationale |
| Cloud functions | 30 functions with full action routing tables (admin / products / inventory / payments / orders / staff / returns / points / reviews / courier / subscribe messages / gift cards / invoices …), unified `exports.main → switch(action)` convention |
| Database | 21-collection design and permission essentials (orders are creator-only → staff consoles must go through cloud functions) |
| Business rules | Three-branch pricing, inventory semantics, cross-store cart, permission model, membership points, idempotent refunds, review dedup, permanent image URLs, server-side courier pricing, gift-card state machine, idempotent pinyin indexing, pub* public-endpoint pattern |
| Pages | Specs for 31 pages grouped by flow (5 Tabs + transaction chain + personal center + store consoles + hidden pages) |
| Peripherals | H5 brand site + admin, in-store ESC/POS thermal print service (incl. Windows ops pitfalls) |
| Deployment | Cloud-function dependency order, env-var group checklist, WeChat Pay V3, subscribe messages, server notes |
| Pitfalls | 15+ hard-earned lessons (wxss brace balancing, limit-100 pagination, _.in limit 20, GBK .bat encoding, port-8899 conflicts …) |

## Tutorial Structure (8 Steps)

1. **Directory skeleton** — app.json essentials and the global layer
2. **Cloud functions** — action lists and responsibilities of all 30
3. **Cloud database** — 21 collections and permission design
4. **Core business rules** — 14 "change one thing, break everything" rules
5. **Pages** — 31 pages in grouped specs
6. **H5 brand site + admin** — React build and PHP relay
7. **In-store print service** — ESC/POS auto receipts
8. **Deployment & configuration** — env vars, payments, launch checklist

## Usage

**As a Hermes skill**: place `SKILL.md` (or `SKILL.en.md`) under
`skills/wechat/wechat-chain-mall-rebuild/`, then tell the agent
"build a chain-mall mini program following the tutorial".

**As a Claude Code / Codex skill**: drop it into the project's
`.claude/skills/` or `.agents/skills/` directory and invoke by name.

**Plain reading**: open `SKILL.md` / `SKILL.en.md` and follow Step 1 → Step 8.

## Use Cases

- Developers building a multi-store retail mall mini program from scratch
- Teams cloning an existing mini program to a new AppID / cloud environment
- A reference implementation for learning the "mini program + CloudBase +
  H5 admin + local printing" full-stack architecture
- A domain-knowledge pack for AI coding agents, so business rules never need
  re-explaining

## Files

```
SKILL.md      The tutorial (Chinese, canonical)
SKILL.en.md   English version of the skill
README.md     This file (bilingual)
```

## Ready-to-Paste Agent Prompts

Drop the following together with `SKILL.en.md` into any AI coding assistant
(Hermes / Claude Code / Codex / Cursor …) and it can start immediately:

```
You are a WeChat Mini Program full-stack engineer. Strictly follow the
attached SKILL.en.md tutorial to build a "chain stores + online mall"
WeChat mini program (CloudBase edition) from scratch.

Requirements:
1. Execute Step 1 through Step 8 in order; report after each step and wait
   for my confirmation before continuing
2. Write all 30 cloud functions in the dependency order given by the
   tutorial, using the unified exports.main → switch(action) structure
3. Implement all 14 core business rules verbatim — no simplification or
   "optimization" of your own
4. Ask me for business data (products/stores/cards) first; if unavailable,
   create empty collections + placeholder sample data and mark them as
   placeholders in the delivery notes
5. Leave secrets (keys/AppID/merchant ID) as environment-variable
   placeholders for me to fill in
6. After each batch of pages, explain how to compile and verify in WeChat
   DevTools
7. Avoid every pitfall in the tutorial's pitfall list

Start with Step 1: "directory skeleton + app.json + global layer".
```

To **replicate an existing project** (the scenario: you have a similar mini
program already running in production, and want an identical copy under a NEW
AppID + new cloud environment), use this one:

```
Read the attached SKILL.en.md tutorial. I have a similar WeChat mini
program already in production (the old project, at <old-project-path>),
and I want an identical copy under a NEW AppID + NEW cloud environment.
Execute in this order:
1. Read the old project's directory structure first, diff it against the
   tutorial's directory / cloud-function / collection lists, and show me
   the differences for confirmation
2. Export business data (products/stores/members) from the old project and
   import into the new environment (products via importProducts or the CSV
   pipeline; stores go into stores.js)
3. Migrate the old cloud functions one by one (preserve ALL actions and
   env-var names); globally replace three identifiers: AppID, cloud-env ID,
   image direct-link domain
4. Migrate the 31 pages one by one, compiling in WeChat DevTools every 5
5. End-to-end test: order → pay → accept → verify → refund → points → review
List all secrets (keys/merchant IDs/accounts) for me to fill in manually —
never write them into code or docs.
```

## License

MIT

# 连锁零售微信商城小程序 · 从零复刻教程（Agent Skill）

> **English version below · 英文版见本文档下半部分**

一份**教程型 AI Agent Skill**：把"连锁门店 + 线上商城"微信小程序的完整全栈架构，
浓缩成一份可直接执行的复刻教程。给 AI 编程助手（Hermes / Claude Code / Codex 等）
装上它，就能从零搭出一个生产级的同类系统。

**纯教程，不含任何真实业务数据、账号、密钥。**

---

## 目录

- [这个系统长什么样](#这个系统长什么样)
- [目标系统的功能全景](#目标系统的功能全景)
- [教程覆盖内容](#教程覆盖内容)
- [教程结构（8 步）](#教程结构8-步)
- [使用方式](#使用方式)
- [适用场景](#适用场景)
- [文件清单](#文件清单)
- [License](#license)

## 这个系统长什么样

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储
                → 微信支付V3 / 同城跑腿API / 订阅消息 / 内容安全
H5品牌站+管理后台(React+Vite) → PHP中转 → 云函数网关
门店Windows打票机 ← 本地打印服务(8899) ← 后台发现新订单自动出票
```

**技术选型**：原生微信小程序（非 Taro/uni-app）+ 微信云开发 CloudBase
（云函数 + 云数据库 + 云存储），无自建后端服务器；H5 用 React 18 + Vite +
Tailwind 构建为静态站，配合一个轻量 PHP 中转对接云函数网关。

**为什么是这个栈**：单开发者/小团队可维护；云开发免运维；前端无构建直接传
微信开发者工具；门店打印走 Windows 本地服务，不依赖云。

## 目标系统的功能全景

| 端 | 功能 |
|---|---|
| C 端小程序 | 多门店定位选购、双级分类+拼音搜索、限时价/整箱规格、跨店购物车、优惠券、微信支付、自提/配送/同城跑腿、订单状态机、核销码自提、评价（内容安全审核）、会员 3 级、积分商城、收藏/足迹、退换货、榜单/活动/公告 |
| 门店端（小程序内） | 店员 30s 轮询接单、备货/配送状态流转、核销码+拍照核销、店长管库存/打烊/店员/退货审核 |
| 超管后台（H5） | 商品 CRUD + CSV 批量导入导出、订单/销售报表、库存、退货、店员、门店开关、公告/活动/广告/榜单、优惠券、积分商品、提货卡全生命周期（制卡/激活/核销/流水/导出）、发票 |
| 周边系统 | H5 品牌站（门店地图）、服务号消息通知、订阅消息、门店热敏小票自动打印 |

## 教程覆盖内容

| 模块 | 内容 |
|---|---|
| 架构 | 目录骨架、系统拓扑、技术选型与理由 |
| 云函数 | 30 个函数清单 + 每个的 action 路由表（后台/商品/库存/支付/订单/店员/退货/积分/评价/跑腿/订阅消息/提货卡/发票……），统一 `exports.main → switch(action)` 写法约定 |
| 数据库 | 21 个集合设计及权限要点（订单仅创建者可读写 → 店员端必须云函数中转） |
| 业务规则 | 价格三分支、库存语义、跨店购物车、权限模型、会员积分、退款幂等、评价防重、图片永久直链、跑腿计费云端复算、提货卡状态机、拼音搜索幂等增强、公开接口 pub* 模式 |
| 页面 | 31 个页面分组规格（5 Tab + 交易链 + 个人中心 + 门店管理端 + 隐藏页） |
| 周边 | H5 品牌站+管理后台、门店 ESC/POS 热敏打印服务（含 Windows 运维坑） |
| 部署 | 云函数依赖顺序、环境变量分组清单、微信支付 V3、订阅消息、服务器注意事项 |
| 避坑 | 15+ 条真实踩坑记录（wxss 括号配平、云库 limit 100 分页、_.in 限 20、bat 中文 GBK、8899 端口冲突……） |

## 教程结构（8 步）

1. **目录骨架** — app.json 要点与全局层
2. **云函数** — 30 个函数的 action 清单与职责
3. **云数据库** — 21 集合与权限设计
4. **核心业务规则** — 14 条"改一处全盘皆错"的规则
5. **页面** — 31 页分组规格
6. **H5 品牌站 + 管理后台** — React 构建与 PHP 中转
7. **门店打印服务** — ESC/POS 自动出票
8. **部署与配置** — 环境变量、支付、上线检查

## 使用方式

**作为 Hermes Skill**：把 `SKILL.md` 放到 `skills/wechat/wechat-chain-mall-rebuild/`
目录下，对 Agent 说"照教程搭一个连锁商城小程序"即可触发。

**作为 Claude Code / Codex Skill**：放入项目的 `.claude/skills/` 或 `.agents/skills/`
目录，同样按名称调用。

**纯人工阅读**：直接读 `SKILL.md`，按"第一步~第八步"顺序执行即可。

## 适用场景

- 想从零做一个多门店零售商城小程序的开发者
- 需要把现有小程序项目克隆到新 AppID / 新云环境的团队
- 学习"小程序 + 云开发 + H5 后台 + 本地打印"全栈架构的参考样本
- 作为 AI 编码 Agent 的领域知识包，避免每次重新解释业务规则

## 文件清单

```
SKILL.md      教程本体（中文，正本）
SKILL.en.md   English version of the skill
README.md     仓库说明（中英双语，本文件）
```

## 直接喂给 Agent 的提示词（复制即用）

把下面这段连同 `SKILL.md` 一起丢给任意 AI 编程助手（Hermes / Claude Code /
Codex / Cursor……），它就能开工：

```
你是一个微信小程序全栈工程师。请严格按照附件 SKILL.md 的教程，
从零搭建一个"连锁门店 + 线上商城"微信小程序（云开发版）。

要求：
1. 严格按教程第一步~第八步的顺序执行，每完成一步向我汇报，经我确认后再继续
2. 30 个云函数按教程给的依赖顺序编写，统一 exports.main → switch(action) 结构
3. 14 条核心业务规则逐条落实，不得自行简化或"优化"
4. 业务数据（商品/门店/卡库）先问我拿，拿不到就建空集合+示例数据占位，
   并在交付清单里标注哪些是占位
5. 密钥/AppID/商户号等敏感配置只留环境变量占位符，由我自行填入
6. 每批页面写完后说明如何在微信开发者工具里编译验证
7. 避开教程"易错点"清单里的所有坑

先从第一步"目录骨架 + app.json + 全局层"开始。
```

要克隆已有项目时，改用这段：

```
请阅读附件 SKILL.md 教程。我已有一个同架构的微信小程序项目，
现在要把它的业务数据灌进按教程搭好的新骨架里。
原项目在 <原项目路径>，请按以下顺序执行：
1. 对比原项目与教程的目录/云函数/集合清单，列出差异
2. 从原项目导出商品/门店/会员等数据，导入新云环境
3. 把原项目的云函数代码逐函数迁入（保留全部 action）
4. 31 个页面逐个迁入，每迁 5 个在开发者工具编译一次
5. 全链路联调：下单→支付→接单→核销→退款→积分→评价
有任何敏感配置（密钥/账号）都列清单让我手动填，不要写进代码。
```

## License

MIT

---
---

# Chain-Retail WeChat Mall Mini Program · From-Scratch Rebuild Tutorial (Agent Skill)

A **tutorial-style AI Agent Skill** that distills the complete full-stack
architecture of a "chain stores + online mall" WeChat mini program into a
directly executable rebuild tutorial. Install it into any AI coding assistant
(Hermes / Claude Code / Codex …) and it can build a production-grade system of
the same kind from zero.

**Pure tutorial — contains no real business data, accounts, or secrets.**

## Table of Contents

- [What the System Looks Like](#what-the-system-looks-like)
- [Feature Overview](#feature-overview)
- [What the Tutorial Covers](#what-the-tutorial-covers)
- [Tutorial Structure (8 Steps)](#tutorial-structure-8-steps)
- [Usage](#usage)
- [Use Cases](#use-cases)
- [Files](#files)
- [License](#license-1)

## What the System Looks Like

```
Customer (WeChat Mini Program) → Cloud Functions (30) → Cloud DB (21 collections) + Cloud Storage
                               → WeChat Pay V3 / local courier API / subscribe messages / content safety
H5 brand site + admin (React+Vite) → PHP relay → cloud function gateway
In-store Windows receipt printer ← local print service (8899) ← auto-prints new orders
```

**Tech choices**: native WeChat Mini Program (no Taro/uni-app) + WeChat
CloudBase (cloud functions + cloud DB + cloud storage) — no self-hosted
backend; the H5 side is React 18 + Vite + Tailwind built to a static site with
a thin PHP relay to the cloud-function gateway.

**Why this stack**: maintainable by a solo dev or small team; CloudBase is
ops-free; the mini program has no build step (straight into WeChat DevTools);
in-store printing runs as a Windows-local service with no cloud dependency.

## Feature Overview

| Side | Features |
|---|---|
| Customer mini program | Multi-store location-based shopping, two-level categories + pinyin search, flash pricing / case specs, cross-store cart, coupons, WeChat Pay, pickup / delivery / local courier, order state machine, verification-code pickup, reviews (content-safety checked), 3-tier membership, points mall, favorites/history, returns & exchanges, rankings/activities/notices |
| Store console (in mini program) | Staff 30s polling for new orders, prep/delivery status flow, verification code + photo confirmation, manager inventory/close-store/staff/return-audit |
| Super admin (H5) | Product CRUD + CSV bulk import/export, orders/sales reports, inventory, returns, staff, store open/close, notices/activities/ads/rankings, coupons, points products, gift-card full lifecycle (create/activate/redeem/transactions/export), invoices |
| Peripheral systems | H5 brand site (store map), service-account notifications, subscribe messages, in-store thermal receipt auto-printing |

## What the Tutorial Covers

| Module | Contents |
|---|---|
| Architecture | Directory skeleton, system topology, tech choices and rationale |
| Cloud functions | 30 functions with full action routing tables (admin / products / inventory / payments / orders / staff / returns / points / reviews / courier / subscribe messages / gift cards / invoices …), unified `exports.main → switch(action)` convention |
| Database | 21-collection design and permission essentials (orders are creator-only → staff consoles must go through cloud functions) |
| Business rules | Three-branch pricing, inventory semantics, cross-store cart, permission model, membership points, idempotent refunds, review dedup, permanent image URLs, server-side courier pricing, gift-card state machine, idempotent pinyin indexing, pub* public-endpoint pattern |
| Pages | Specs for 31 pages grouped by flow (5 Tabs + transaction chain + personal center + store consoles + hidden pages) |
| Peripherals | H5 brand site + admin, in-store ESC/POS thermal print service (incl. Windows ops pitfalls) |
| Deployment | Cloud-function dependency order, env-var group checklist, WeChat Pay V3, subscribe messages, server notes |
| Pitfalls | 15+ hard-earned lessons (wxss brace balancing, limit-100 pagination, _.in limit 20, GBK .bat encoding, port-8899 conflicts …) |

## Tutorial Structure (8 Steps)

1. **Directory skeleton** — app.json essentials and the global layer
2. **Cloud functions** — action lists and responsibilities of all 30
3. **Cloud database** — 21 collections and permission design
4. **Core business rules** — 14 "change one thing, break everything" rules
5. **Pages** — 31 pages in grouped specs
6. **H5 brand site + admin** — React build and PHP relay
7. **In-store print service** — ESC/POS auto receipts
8. **Deployment & configuration** — env vars, payments, launch checklist

## Usage

**As a Hermes skill**: place `SKILL.md` (or `SKILL.en.md`) under
`skills/wechat/wechat-chain-mall-rebuild/`, then tell the agent
"build a chain-mall mini program following the tutorial".

**As a Claude Code / Codex skill**: drop it into the project's
`.claude/skills/` or `.agents/skills/` directory and invoke by name.

**Plain reading**: open `SKILL.md` / `SKILL.en.md` and follow Step 1 → Step 8.

## Use Cases

- Developers building a multi-store retail mall mini program from scratch
- Teams cloning an existing mini program to a new AppID / cloud environment
- A reference implementation for learning the "mini program + CloudBase +
  H5 admin + local printing" full-stack architecture
- A domain-knowledge pack for AI coding agents, so business rules never need
  re-explaining

## Files

```
SKILL.md      The tutorial (Chinese, canonical)
SKILL.en.md   English version of the skill
README.md     This file (bilingual)
```

## Ready-to-Paste Agent Prompts

Drop the following together with `SKILL.en.md` into any AI coding assistant
(Hermes / Claude Code / Codex / Cursor …) and it can start immediately:

```
You are a WeChat Mini Program full-stack engineer. Strictly follow the
attached SKILL.en.md tutorial to build a "chain stores + online mall"
WeChat mini program (CloudBase edition) from scratch.

Requirements:
1. Execute Step 1 through Step 8 in order; report after each step and wait
   for my confirmation before continuing
2. Write all 30 cloud functions in the dependency order given by the
   tutorial, using the unified exports.main → switch(action) structure
3. Implement all 14 core business rules verbatim — no simplification or
   "optimization" of your own
4. Ask me for business data (products/stores/cards) first; if unavailable,
   create empty collections + placeholder sample data and mark them as
   placeholders in the delivery notes
5. Leave secrets (keys/AppID/merchant ID) as environment-variable
   placeholders for me to fill in
6. After each batch of pages, explain how to compile and verify in WeChat
   DevTools
7. Avoid every pitfall in the tutorial's pitfall list

Start with Step 1: "directory skeleton + app.json + global layer".
```

To clone an existing project instead, use this one:

```
Read the attached SKILL.en.md tutorial. I already have a WeChat mini
program project with the same architecture, and I want to migrate its
business data into the new skeleton built from the tutorial.
The original project is at <original-project-path>. Execute in order:
1. Diff the original project against the tutorial's directory / cloud
   function / collection lists and report the differences
2. Export products/stores/membership data from the original project and
   import into the new cloud environment
3. Migrate the cloud functions one by one (preserve ALL actions)
4. Migrate the 31 pages one by one, compiling in DevTools every 5 pages
5. End-to-end test: order → pay → accept → verify → refund → points → review
List every sensitive configuration (keys/accounts) for me to fill in
manually — never write them into code.
```

## License

MIT

## 这个系统长什么样

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储# 连锁零售微信商城小程序 · 从零复刻教程（Agent Skill）

> **English version below · 英文版见本文档下半部分**

一份**教程型 AI Agent Skill**：把"连锁门店 + 线上商城"微信小程序的完整全栈架构，
浓缩成一份可直接执行的复刻教程。给 AI 编程助手（Hermes / Claude Code / Codex 等）
装上它，就能从零搭出一个生产级的同类系统。

**纯教程，不含任何真实业务数据、账号、密钥。**

---

## 目录

- [这个系统长什么样](#这个系统长什么样)
- [目标系统的功能全景](#目标系统的功能全景)
- [教程覆盖内容](#教程覆盖内容)
- [教程结构（8 步）](#教程结构8-步)
- [使用方式](#使用方式)
- [适用场景](#适用场景)
- [文件清单](#文件清单)
- [License](#license)

## 这个系统长什么样

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储
                → 微信支付V3 / 同城跑腿API / 订阅消息 / 内容安全
H5品牌站+管理后台(React+Vite) → PHP中转 → 云函数网关
门店Windows打票机 ← 本地打印服务(8899) ← 后台发现新订单自动出票
```

**技术选型**：原生微信小程序（非 Taro/uni-app）+ 微信云开发 CloudBase
（云函数 + 云数据库 + 云存储），无自建后端服务器；H5 用 React 18 + Vite +
Tailwind 构建为静态站，配合一个轻量 PHP 中转对接云函数网关。

**为什么是这个栈**：单开发者/小团队可维护；云开发免运维；前端无构建直接传
微信开发者工具；门店打印走 Windows 本地服务，不依赖云。

## 目标系统的功能全景

| 端 | 功能 |
|---|---|
| C 端小程序 | 多门店定位选购、双级分类+拼音搜索、限时价/整箱规格、跨店购物车、优惠券、微信支付、自提/配送/同城跑腿、订单状态机、核销码自提、评价（内容安全审核）、会员 3 级、积分商城、收藏/足迹、退换货、榜单/活动/公告 |
| 门店端（小程序内） | 店员 30s 轮询接单、备货/配送状态流转、核销码+拍照核销、店长管库存/打烊/店员/退货审核 |
| 超管后台（H5） | 商品 CRUD + CSV 批量导入导出、订单/销售报表、库存、退货、店员、门店开关、公告/活动/广告/榜单、优惠券、积分商品、提货卡全生命周期（制卡/激活/核销/流水/导出）、发票 |
| 周边系统 | H5 品牌站（门店地图）、服务号消息通知、订阅消息、门店热敏小票自动打印 |

## 教程覆盖内容

| 模块 | 内容 |
|---|---|
| 架构 | 目录骨架、系统拓扑、技术选型与理由 |
| 云函数 | 30 个函数清单 + 每个的 action 路由表（后台/商品/库存/支付/订单/店员/退货/积分/评价/跑腿/订阅消息/提货卡/发票……），统一 `exports.main → switch(action)` 写法约定 |
| 数据库 | 21 个集合设计及权限要点（订单仅创建者可读写 → 店员端必须云函数中转） |
| 业务规则 | 价格三分支、库存语义、跨店购物车、权限模型、会员积分、退款幂等、评价防重、图片永久直链、跑腿计费云端复算、提货卡状态机、拼音搜索幂等增强、公开接口 pub* 模式 |
| 页面 | 31 个页面分组规格（5 Tab + 交易链 + 个人中心 + 门店管理端 + 隐藏页） |
| 周边 | H5 品牌站+管理后台、门店 ESC/POS 热敏打印服务（含 Windows 运维坑） |
| 部署 | 云函数依赖顺序、环境变量分组清单、微信支付 V3、订阅消息、服务器注意事项 |
| 避坑 | 15+ 条真实踩坑记录（wxss 括号配平、云库 limit 100 分页、_.in 限 20、bat 中文 GBK、8899 端口冲突……） |

## 教程结构（8 步）

1. **目录骨架** — app.json 要点与全局层
2. **云函数** — 30 个函数的 action 清单与职责
3. **云数据库** — 21 集合与权限设计
4. **核心业务规则** — 14 条"改一处全盘皆错"的规则
5. **页面** — 31 页分组规格
6. **H5 品牌站 + 管理后台** — React 构建与 PHP 中转
7. **门店打印服务** — ESC/POS 自动出票
8. **部署与配置** — 环境变量、支付、上线检查

## 使用方式

**作为 Hermes Skill**：把 `SKILL.md` 放到 `skills/wechat/wechat-chain-mall-rebuild/`
目录下，对 Agent 说"照教程搭一个连锁商城小程序"即可触发。

**作为 Claude Code / Codex Skill**：放入项目的 `.claude/skills/` 或 `.agents/skills/`
目录，同样按名称调用。

**纯人工阅读**：直接读 `SKILL.md`，按"第一步~第八步"顺序执行即可。

## 适用场景

- 想从零做一个多门店零售商城小程序的开发者
- 需要把现有小程序项目克隆到新 AppID / 新云环境的团队
- 学习"小程序 + 云开发 + H5 后台 + 本地打印"全栈架构的参考样本
- 作为 AI 编码 Agent 的领域知识包，避免每次重新解释业务规则

## 文件清单

```
SKILL.md      教程本体（中文，正本）
SKILL.en.md   English version of the skill
README.md     仓库说明（中英双语，本文件）
```

## License

MIT

---
---

# Chain-Retail WeChat Mall Mini Program · From-Scratch Rebuild Tutorial (Agent Skill)

A **tutorial-style AI Agent Skill** that distills the complete full-stack
architecture of a "chain stores + online mall" WeChat mini program into a
directly executable rebuild tutorial. Install it into any AI coding assistant
(Hermes / Claude Code / Codex …) and it can build a production-grade system of
the same kind from zero.

**Pure tutorial — contains no real business data, accounts, or secrets.**

## Table of Contents

- [What the System Looks Like](#what-the-system-looks-like)
- [Feature Overview](#feature-overview)
- [What the Tutorial Covers](#what-the-tutorial-covers)
- [Tutorial Structure (8 Steps)](#tutorial-structure-8-steps)
- [Usage](#usage)
- [Use Cases](#use-cases)
- [Files](#files)
- [License](#license-1)

## What the System Looks Like

```
Customer (WeChat Mini Program) → Cloud Functions (30) → Cloud DB (21 collections) + Cloud Storage
                               → WeChat Pay V3 / local courier API / subscribe messages / content safety
H5 brand site + admin (React+Vite) → PHP relay → cloud function gateway
In-store Windows receipt printer ← local print service (8899) ← auto-prints new orders
```

**Tech choices**: native WeChat Mini Program (no Taro/uni-app) + WeChat
CloudBase (cloud functions + cloud DB + cloud storage) — no self-hosted
backend; the H5 side is React 18 + Vite + Tailwind built to a static site with
a thin PHP relay to the cloud-function gateway.

**Why this stack**: maintainable by a solo dev or small team; CloudBase is
ops-free; the mini program has no build step (straight into WeChat DevTools);
in-store printing runs as a Windows-local service with no cloud dependency.

## Feature Overview

| Side | Features |
|---|---|
| Customer mini program | Multi-store location-based shopping, two-level categories + pinyin search, flash pricing / case specs, cross-store cart, coupons, WeChat Pay, pickup / delivery / local courier, order state machine, verification-code pickup, reviews (content-safety checked), 3-tier membership, points mall, favorites/history, returns & exchanges, rankings/activities/notices |
| Store console (in mini program) | Staff 30s polling for new orders, prep/delivery status flow, verification code + photo confirmation, manager inventory/close-store/staff/return-audit |
| Super admin (H5) | Product CRUD + CSV bulk import/export, orders/sales reports, inventory, returns, staff, store open/close, notices/activities/ads/rankings, coupons, points products, gift-card full lifecycle (create/activate/redeem/transactions/export), invoices |
| Peripheral systems | H5 brand site (store map), service-account notifications, subscribe messages, in-store thermal receipt auto-printing |

## What the Tutorial Covers

| Module | Contents |
|---|---|
| Architecture | Directory skeleton, system topology, tech choices and rationale |
| Cloud functions | 30 functions with full action routing tables (admin / products / inventory / payments / orders / staff / returns / points / reviews / courier / subscribe messages / gift cards / invoices …), unified `exports.main → switch(action)` convention |
| Database | 21-collection design and permission essentials (orders are creator-only → staff consoles must go through cloud functions) |
| Business rules | Three-branch pricing, inventory semantics, cross-store cart, permission model, membership points, idempotent refunds, review dedup, permanent image URLs, server-side courier pricing, gift-card state machine, idempotent pinyin indexing, pub* public-endpoint pattern |
| Pages | Specs for 31 pages grouped by flow (5 Tabs + transaction chain + personal center + store consoles + hidden pages) |
| Peripherals | H5 brand site + admin, in-store ESC/POS thermal print service (incl. Windows ops pitfalls) |
| Deployment | Cloud-function dependency order, env-var group checklist, WeChat Pay V3, subscribe messages, server notes |
| Pitfalls | 15+ hard-earned lessons (wxss brace balancing, limit-100 pagination, _.in limit 20, GBK .bat encoding, port-8899 conflicts …) |

## Tutorial Structure (8 Steps)

1. **Directory skeleton** — app.json essentials and the global layer
2. **Cloud functions** — action lists and responsibilities of all 30
3. **Cloud database** — 21 collections and permission design
4. **Core business rules** — 14 "change one thing, break everything" rules
5. **Pages** — 31 pages in grouped specs
6. **H5 brand site + admin** — React build and PHP relay
7. **In-store print service** — ESC/POS auto receipts
8. **Deployment & configuration** — env vars, payments, launch checklist

## Usage

**As a Hermes skill**: place `SKILL.md` (or `SKILL.en.md`) under
`skills/wechat/wechat-chain-mall-rebuild/`, then tell the agent
"build a chain-mall mini program following the tutorial".

**As a Claude Code / Codex skill**: drop it into the project's
`.claude/skills/` or `.agents/skills/` directory and invoke by name.

**Plain reading**: open `SKILL.md` / `SKILL.en.md` and follow Step 1 → Step 8.

## Use Cases

- Developers building a multi-store retail mall mini program from scratch
- Teams cloning an existing mini program to a new AppID / cloud environment
- A reference implementation for learning the "mini program + CloudBase +
  H5 admin + local printing" full-stack architecture
- A domain-knowledge pack for AI coding agents, so business rules never need
  re-explaining

## Files

```
SKILL.md      The tutorial (Chinese, canonical)
SKILL.en.md   English version of the skill
README.md     This file (bilingual)
```

## License

MIT

                → 微信支付V3 / 同城跑腿API / 订阅消息 / 内容安全
H5品牌站+管理后台(React+Vite) → PHP中转 → 云函数网关
门店Windows打票机 ← 本地打印服务(8899) ← 后台发现新订单自动出票
```

## 教程覆盖内容

| 模块 | 内容 |
|---|---|
| 架构 | 目录骨架、系统拓扑、技术选型（原生小程序 + 云开发，无自建后端） |
| 云函数 | 30 个函数清单 + 每个的 action 路由表（后台/商品/库存/支付/订单/店员/退货/积分/评价/跑腿/订阅消息/提货卡/发票……） |
| 数据库 | 21 个集合设计及权限要点（订单仅创建者可读写 → 店员端必须云函数中转） |
| 业务规则 | 价格三分支、库存语义、跨店购物车、权限模型、会员积分、退款幂等、评价防重、图片直链、跑腿计费、提货卡状态机、拼音搜索 |
| 页面 | 31 个页面分组规格（5 Tab + 交易链 + 个人中心 + 门店管理端 + 隐藏页） |
| 周边 | H5 品牌站+管理后台、门店 ESC/POS 热敏打印服务 |
| 部署 | 云函数依赖顺序、环境变量分组、支付 V3、订阅消息、服务器注意事项 |
| 避坑 | 15+ 条真实踩坑记录（wxss 括号、limit 100 分页、_.in 限 20、GBK 编码、端口冲突……） |

## 使用方式

**作为 Hermes Skill**：把 `SKILL.md` 放到 `skills/wechat/wechat-chain-mall-rebuild/`
目录下，对 Agent 说"照教程搭一个连锁商城小程序"即可触发。

**作为 Claude Code / Codex Skill**：放入项目的 `.claude/skills/` 或 `.agents/skills/`
目录，同样按名称调用。

**纯人工阅读**：直接读 `SKILL.md`，按"第一步~第八步"顺序执行。

## 文件

```
SKILL.md   教程本体（唯一文件，自包含）
```

## License

MIT
