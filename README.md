[README.md](https://github.com/user-attachments/files/31619127/README.md)
# Make your agent smarter, Chain Retail WeChat Mall Mini Program · Tutorial from Scratch（Agent Skill）

一份**教程型 AI Agent Skill**：把"连锁门店 + 线上商城"微信小程序的完整全栈架构，
浓缩成一份可直接执行的复刻教程。给 AI 编程助手（Hermes / Claude Code / Codex 等）
装上它，就能从零搭出一个生产级的同类系统。

> 纯教程，不含任何真实业务数据、账号、密钥。

## 这个系统长什么样

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储
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
