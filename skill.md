---
name: wechat-chain-mall-rebuild
description: Use when 从零搭建/复刻连锁零售微信商城小程序(云开发全栈教程).
---

# 连锁零售微信商城小程序 · 一次建成级复刻教程（云开发版）

> 目标：照本教程可以**一次性建成**整个系统，不需要再追问细节。
> 本教程只含结构、契约、规则与流程；**不含任何真实业务数据/账号/密钥**，
> 数据在第八步由你导入。

═══════════════════════════════════════════
第 0 章 · 系统总览与技术选型
═══════════════════════════════════════════

```
用户(微信小程序) → 云函数(30个) → 云数据库(21集合) + 云存储(图片)
                → 微信支付V3 / 同城跑腿API / 订阅消息 / 内容安全
H5品牌站+管理后台(React+Vite静态站) → 服务器PHP中转 → 云函数网关(h5admin→adminApi)
门店Windows打票机 ← 本地打印服务(127.0.0.1:8899) ← H5后台轮询到新订单即POST打印
```

- 小程序：**原生**（非 Taro/uni-app），无构建步骤，直接进微信开发者工具
- 后端：微信云开发（云函数 Node.js + 云数据库 MongoDB 风格 + 云存储）
- H5：React 18 + Vite + Tailwind v4，构建产物发静态服务器 + 一个 PHP 中转文件
- 打印：门店 Windows 机器跑 Node 服务，ESC/POS 直驱热敏打印机

═══════════════════════════════════════════
第 1 章 · 目录骨架（逐目录建）
═══════════════════════════════════════════

```
app.js / app.json / app.wxss / project.config.json / sitemap.json
pages/           31 个页面，每页一个目录 xxx/{xxx.js, xxx.wxml, xxx.wxss, xxx.json}
components/      qty-modal/  loading-lottie/
utils/           data.js stores.js price.js cloud.js membership.js
                 pinyin-search.js imgutil.js images.js avatars.js
                 qqmap-wx-jssdk.js(腾讯地图SDK) libs/pinyin-pro.js(拼音库vendor)
services/        cart.js coupons.js
cloudfunctions/  30 个云函数，每个一个目录 xxx/{index.js, package.json}
h5-app/          React 后台源码（src/ 下 App.tsx + context/ + components/ + data/）
admin/           遗留静态后台（可选，备份用途）
print-server/    打印服务（print-server.js + 若干 bat/ps1）
images/          本地占位图 placeholder.jpg 等
```

═══════════════════════════════════════════
第 2 章 · 全局三件套（app.json / app.wxss / app.js）
═══════════════════════════════════════════

## 2.1 app.json（照此结构写全）

- pages 数组 31 项，顺序与名称：
  index, category, membership, profile, wine-detail, ranking, cart, coupons,
  favorites, to-receive, to-ship, to-pay, address, browse-history, points,
  edit-profile, return-exchange, return-audit, checkout, order-track,
  points-mall, store-staff, store-admin, order-detail, uupt-track, invite,
  service-bind, activity, stores, review-write, review-list
  （全部加 "pages/<name>/<name>" 前缀）
- 隐藏页 invoice-apply / invoice-list **不写进 pages**（隐藏功能，勿接线）
- window：navigationBarTitleText=品牌名，navigationBarBackgroundColor=品牌蓝纯色
  （例如 #0284c7，**系统导航必须纯色**），navigationBarTextStyle=white，
  backgroundColor=#f5f6fa，navigationStyle=custom
- tabBar 5 项：首页/分类/购物车/会员/我的；color=#94a3b8，selectedColor=#1e3a8a，
  backgroundColor=#ffffff
- usingComponents: { "qty-modal": "/components/qty-modal/qty-modal" }
- lazyCodeLoading: "requiredComponents"
- permission: { "scope.userLocation": { "desc": "用于首页定位与附近门店推荐" } }
- requiredPrivateInfos: ["chooseLocation", "getLocation", "chooseAddress"]
- packOptions.ignore: docs 目录与大文件
- sitemapLocation: sitemap.json

## 2.2 app.wxss（全局设计变量）

```
--font-sans: -apple-system, BlinkMacSystemFont, 'Inter', system-ui, sans-serif
--font-serif: 'Noto Serif SC', Georgia, serif      /* 会员卡/榜单标题 */
--color-primary: #1e3a8a        /* 品牌深蓝：Tab选中/主按钮 */
--color-primary-container: #1e40af
页面背景 #faf9fa；正文 #1b1c1d；次要文字 #434653；错误 #ba1a1a
基础字号 32rpx；行高 1.5；全局 box-sizing: border-box
```
视觉规范：首页/分类/详情=白卡片+圆角16~24rpx+轻阴影电商风；
会员卡横幅=孟菲斯风（几何色块+圆点+撞色）；榜单页=波普叙事风
（品牌蓝主色+网点纹理+黑边硬阴影+撞色）。图片一律放 overflow:hidden 容器；
购物车角标=position:absolute + 父容器 overflow:hidden。

## 2.3 app.js（全局层）

globalData 字段（逐个建）：
  defaultImage, usePlaceholderImages(false), activeScreen('HOME'),
  isLoggedIn, hasUserInfo, username, avatarUrl, bio, phone,
  favorites[], cartItems[], cardBanner(null), customizingItem(null),
  wineProducts(=data.js 静态兜底), menuItems, coupons[], appliedCouponId(null),
  addresses[], historyItems[], points(0), pointsTransactions[],
  dineInMode(null), systemInfo, statusBarHeight(0),
  activeStore(null 当前门店), nearbyStore(null 定位门店),
  manualStore(false 手选优先于定位)

必须实现的方法：
  onLaunch（初始化云 wx.cloud.init、读系统信息算 statusBarHeight、
    checkLoginStatus、loadCloudData 拉云端商品覆盖本地、locateNearestStore、
    checkPendingOrders、_handleInvite 处理分享进入）
  getStockStore()/setStockStore()  — 生效门店：手选 > 定位 > 默认首店
  checkLoginStatus / requireLogin / doLogin / doPhoneLogin / _handleLoginSuccess / logout
  loadUserData / loadCloudData / notifyDataReady
  getMembershipInfo / addPoints
  toggleFavorite / isFavorited
  addToHistory / removeHistoryItem / clearAllHistory
  onPageNotFound

登录流程：wx.login 静默拿 code → getOpenid 云函数换 openid → 存 Storage
（key 例如 'openid'）→ users 集合 upsert。手机号登录走 phoneLogin 云函数
（解密 getPhoneNumber）。users 查询必须兼容 _openid 两种字段名。

═══════════════════════════════════════════
第 3 章 · 数据结构契约（逐字段，照抄）
═══════════════════════════════════════════

## 3.1 商品 wine_products（静态 data.js 兜底 + 云端真源）
```
id: 'wine-001' | 'tea-001'        字符串主键，品类前缀
name: string
price: number                     会员价（生效价默认分支）
originalPrice: number             划线原价（云端）
flashPrice: number                限时价（云端，0=无）
flashStart / flashEnd: string     限时时间窗（'YYYY-MM-DD HH:mm' 或 datetime-local 格式）
packSpecs: [{ name:'6瓶装', label, size:6, price }]   整箱规格（可选）
image: [fileID...]                主图数组 ≤20（轮播）
images: [fileID...]               详情图数组 ≤20（与主图分离）
category: 'Baijiu|Tea|Red|Beverage|Beer|Fruit|Huangjiu|White|Shochu|Whisky'
tag: string                       营销角标（热销/整箱/清仓/珍藏）
description: string
sales: number                     销量（云端）
banner: boolean                   是否进首页轮播（云端）
sort: number
```
商品规模参考：约 190 款（白酒/茶叶/红酒/饮料/啤酒/果酒/黄酒/白兰地/烧酒/威士忌）。

## 3.2 门店 stores（utils/stores.js 唯一数据源）
```
{ id:'s1', name:'品牌名+商圈名+店', address, phone, lat, lng, hours:'08:00-22:00' }
```
约 16 家。注意：id 允许历史跳号（如 s1-s5 后 s11-s21），**勿重排**；
H5 端 data/initialData.ts 需与此同步。

## 3.3 购物车条目（services/cart.js）
```
{ productId, name, price, originalPrice, points, image, quantity,
  type:'wine'|..., remarks, packSpec(null|{size,label,price}), storeId,
  key: productId + '_' + 规格size + '_' + storeId }
```
堆叠规则：同店+同商品+同规格+同备注 → 合并数量。旧数据无 storeId 归默认首店。

## 3.4 订单 orders（createOrder 时规范化冗余字段）
```
{ ...items 透传,
  items: [{ productId, name, price, quantity, points, image, packSpec, size }],
  deliveryMethod: 'pickup'|'delivery'|'uupt',
  deliveryLabel: '自提'|'配送',          // 冗余，后台直接可读
  totalQuantity: number,                // 总件数（冗余）
  itemNames: '名称(规格)×N、…',          // 冗余平铺，控制台友好
  phone, addressText,                   // 冗余自 address
  address: { region[], community, detail, phone, name },
  customerOpenid,                       // 冗余（_openid 为系统字段）
  subtotal, shippingFee, discount, couponId, total, pointsTotal,
  status: 'pending_pay'|'pending_ship'|'in_transit'|'completed'|'cancelled',
  payStatus, payTime:null, shipTime:null, createTime/createdAt(同值),
  trackingNo:'', carrier:'',
  verifyCode, verifyTime,               // 自提核销
  uuptOrderCode,                        // 跑腿单号
  refundStatus, refundAmount,           // 退款状态机（幂等）
  images: []                            // 核销拍照
}
```
权限：orders 集合设"仅创建者可读写"→ 店员/店长接单必须走
staffApi.listOrders / orderApi.getDetail 云函数中转。

## 3.5 其余集合字段要点
- users：_openid、phone、username、avatarUrl、bio、points(累计积分)、coupons
- staff：openid、storeId、role('admin'|'staff')、nickname、phone、password、
  enabled、canSeed
- inventory：{ productId, storeId, stock(按瓶), updatedAt }
- coupon_templates：{ title, type('满减'|'折扣'|'无门槛'), threshold, value,
  total, claimed, startTime, endTime, enabled, scope }；
  用户券状态：claimable(可领)/available(已领)/used(已用)
- points_products：{ name, points, image, stock, enabled }
- points_log：{ userId, delta(±), type, orderId, remark, createdTime }
- reviews：{ orderId, productId, userId, rating(1-5), content, tags[], images[],
  reply, createdTime }，一单一品一条
- return_requests：{ orderId, storeId, userId, reason, type('return'|'exchange'),
  method('门店'|'快递'|'跑腿'), status('pending'|'approved'|'rejected'|'received'),
  courierFee, images[], address, createdTime, refundId }
- rankings：{ title, desc, type, items[{productId,name,sales,rank}], enabled }
- ads：{ title, image, productId, position('首页轮播'|'首页广告'|'弹窗广告'|'清仓专区'),
  enabled, startTime, endTime }
- notices / activities：{ title, content/image, enabled, createdTime }
  （activities 另有 themeStyle、goods[]）
- card_banner：{ image, link }（首页会员卡上方横幅+年度榜单图）
- cards：{ cardNo, pwd(加密), kindId, value, status('未激活'|'已激活'|'已核销'|'已冻结'),
  activateTime, storeId, holderOpenid }
- card_kinds：{ name, faceValue, price, total, sold }
- card_trades：{ cardNo, storeId, orderId, amount, time, operator }
- invoices：{ userId, orderId, title, titleType('个人'|'企业'), taxNo, email,
  status('待审核'|'已核验'|'已驳回'), createdTime }
- store_settings：{ storeId, closeTimes[], openTimes[], updatedAt }

═══════════════════════════════════════════
第 4 章 · utils/services 逐文件函数签名
═══════════════════════════════════════════

## utils/price.js（价格唯一出口，全端必走）
```
getEffectivePrice(p) → { price, isFlash, memberPrice }
  flashPrice>0 且 now∈[flashStart,flashEnd] → 用 flashPrice, isFlash=true
  （时间窗解析兼容 'YYYY-MM-DD HH:mm' 与 datetime-local 的 T 分隔）
getOriginalPrice(p) → originalPrice>生效价 ? originalPrice : 0
```

## utils/membership.js
```
MEMBERSHIP_TIERS = [普通会员(0分,灰渐变), VIP(100001分,深蓝渐变), 钻石(1500001分,紫渐变)]
  每级: { name,title,minPoints,gradient,borderColor,textColor,metaColor,glowColor,
         logoIcon,pointColor,rarity,rarityBadgeClass,description,privileges[] }
getMembershipDetails(points) → { activeTier, nextTier }
getTierGradientStyle(tier) → css 渐变字符串
calcPoints(items) → 指定高价品类(名称关键词匹配) 1元=1分；其余 1元=5分；floor 取整
```

## utils/cloud.js（云数据库统一出口，自动按 _openid 隔离）
```
ENV 常量；getDB()
收藏: uploadFavorites(全量替换) loadFavorites addFavorite removeFavorite
足迹: addHistory loadHistory removeHistory clearHistory
地址: loadAddresses addAddress updateAddress removeAddress
订单: createOrder(data→见3.4规范化) loadOrders(status) updateOrderStatus
用户: createOrUpdateUser
券: loadUserCoupons claimCoupon markCouponUsed
积分: addPointsLog loadPointsLog
退货: addReturnRequest loadReturnRequests
购物车: syncCart loadCart
库存: loadInventory getStock adjustStock setStock deductStocks batchGetStock
```

## utils/imgutil.js（图片直链，与云端同套逻辑）
```
encPath(p)        中文/空格 URL 编码；已编码(含%XX)或纯 ASCII 原样返回
cloudIdToHttps(id)  cloud:// → https://<appid>.tcb.qcloud.la/<path> 永久直链
  兼容 cloud://<appid>/<path> 与 cloud://<envid>.<appid>/<path>（取末段为 appid）
  容忍路径内嵌 .tcb.qcloud.la/ 的脏数据；失败原样返回，image binderror 兜底
```

## utils/pinyin-search.js
```
enhanceProducts(products)  元素级幂等(p._py 存在则跳过)：生成
  p._py  = 全拼小写无空格（非中文原样保留）
  p._pys = 首字母小写
搜索匹配 name / _py / _pys 三字段。vendor 库 libs/pinyin-pro.js 免构建。
云端商品缓存 10 分钟 + refreshGoodsIfStale(60s 节流)。
```

## utils/stores.js / images.js / avatars.js
```
stores.js: STORES 数组（3.2）+ 按距离排序工具
images.js: DEFAULT_IMAGE = '/images/placeholder.jpg'
avatars.js: 8 个系统头像云路径数组 SYSTEM_AVATARS
```

## services/cart.js（跨店购物车）
```
currentStoreId()  生效店=手选>定位>默认首店
getCartItems(storeId?)  addToCart(item)（3.3堆叠规则）
removeCartItem / removeCartItemByKey / clearCart / getCartCount / updateCartBadge
```

## services/coupons.js
```
syncCouponStatus()  云记录同步本地 claimable/available/used 三态
claimCoupon / 可用性过滤（门槛/有效期/已用）
```

═══════════════════════════════════════════
第 5 章 · 云函数逐个规格（30 个）
═══════════════════════════════════════════

统一写法：`exports.main = async (event) => { switch(event.action){...} }`，
返回 `{ ok:true, data }` / `{ ok:false, err }`。部署顺序见第 8 章。

## ★ adminApi（后台核心，最大）
鉴权 auth(event)：
  超管 = 专用账号(环境变量 ADMIN_SUPER_ACCOUNT) + ADMIN_SUPER_PASSWORD → role:'super'
  管理员/店员 = 手机号+密码查 staff 集合 → role:'admin'|'staff', storeId
权限矩阵（勿改）：super=全店+店员管理；卡种/卡列表/广告/券/积分商品/商品/位置/
网页组仅 super；绑店 admin=本店订单/退货/库存/打烊/提货卡/打票机；无店 admin 判
super；面板强制本店。
actions（按组建）：
  登录: login
  商品: listProducts | h5Goods(分页) | h5GoodsSave | h5GoodsDelete | h5GoodsToggle |
       h5GoodsExport(CSV中文表头，导出即导入模板) | h5GoodsImport(按id匹配更新，
       无删除语义) | h5GoodsImages | h5SetProductImage | uploadImage | uploadFile
  订单: h5Orders | h5OrderDetail | h5OrderOp | h5OrdersExport | h5SalesReport | h5CloseTimes
  退货: h5Returns | h5ReturnOp
  库存: h5Stock | h5StockSet | h5StockUpdate
  店员: h5Staff | h5StaffAdd | h5StaffUpdate | h5StaffRemove | h5StaffCandidates(已登录小程序的候选用户) | h5StaffMigrate
  门店: h5Stores | h5SetCloseTime | h5SetOpenTime
  内容: h5Notices/h5NoticeSave/h5NoticeDelete | listActivities/saveActivity/
       deleteActivity/toggleActivity | h5Ads/h5AdsSave/h5AdsDelete/h5AdsToggle |
       h5RankingList/h5RankingSave/h5RankingDelete
  公开读(放鉴权前): pubNotices | pubActivities | pubAds | pubRankings(联查商品)
  券: listCoupons | saveCoupon | deleteCoupon | toggleCoupon
  积分商品: pointsList | pointsSave | pointsDelete | pointsToggle
  提货卡: cardLogin | cardKindList | cardKindSave | cardKindDelete |
       cardBatchCreate(批量制卡) | cardList(卡号搜索用后端正则进where) |
       cardBatchDelete | cardBatchSetPwd | cardActivate | cardUnactivate |
       cardCancel | cardReset | cardSetPassword | cardVerify | cardShip |
       cardStores | cardComplete | cardDelete | cardSubmitTrade | cardTradesList |
       cardTradeDelete | cardExport | cardExportTrades | cardMigrate |
       h5CardBannerGet | h5CardBannerSave
  发票: h5InvoiceList | h5InvoiceIssue | h5InvoiceDelete
  其他: debug | adMigrateImages
环境变量：ADMIN_SUPER_ACCOUNT / ADMIN_SUPER_PASSWORD / ADMIN_SUPER_KEY /
  ADMIN_H5_PASSWORD / CARD_ENC_KEY
**新 action 一律放主 try 块内**，否则部署后 404。

## ★ productApi（商品，客户端）
list(公开分页+筛选) / get(详情) / add / update / delete / banner(首页轮播)。
管理操作校验 ADMIN_PASSWORD 环境变量。内含 cloudIdToHttps（同 imgutil）。

## ★ inventoryApi（库存）
get / list / batchGet / set / adjust / addBack(退货回库) / deduct /
deductBatch(批量，任一不足整单拒绝) / findStores(按商品查有货门店)。
语义：0=无货，>0=有货，无记录=售罄。

## ★ payApi（微信支付 V3）
actions: query(查单/付后确认) | refund(退款) | courierFee(配送费) | queryCourierFee
退款规则：未发货全额含运费；已发货退商品不退运费；扣回已发积分；
refundStatus 状态机 + outRefundNo 唯一 = 幂等。
环境变量：MCHID / WXPAY_APPID / WXPAY_SERIAL_NO / WXPAY_API_V3_KEY

## ★ payNotify（支付回调，微信直调，无 action）
V3 验签(证书序列号+APIv3密钥) → 解密 resource → 改单 paid → deductBatch 扣库存
→ calcPoints 加积分写 points_log → 发店员新订单订阅消息 →（H5 轮询到即打印）。

## ★ orderApi / staffApi（店员侧）
orderApi: getDetail(仅本店店员或本人) / listByStore / setVerifyCode / uploadImages
staffApi: add/remove/toggle/list/setStore/setNickname/myStore/
  listOrders(本店订单，店员端30s轮询用)/updateOrder(状态流转)/
  canSeed(种子权限)/seedAdmin(种管理员)
订单状态机：pending_pay → pending_ship → in_transit → completed（+cancelled/退款分支）

## ★ pointsApi / reviewsApi / returnApi
pointsApi: list / exchange(验积分) / myExchanges / logPoints / myLogs /
  deductForOrder / refundForOrder
reviewsApi: submit(一单一品防重) / byProduct / checkOrder /
  msgCheck+mediaCheck+riskCheck(内容安全，仅 risky 拦)；展示评分=(5+平均分)/2
returnApi: list / approve / reject / receive(收货后退款)

## ★ uuptApi + uuptCallback（同城跑腿）
uuptApi actions: price(询价) / create / quickCreate / cancel / cancelFee /
  account / rechargeUrl / detail / track / dispatchOrder / gratuity / sync
签名按官方 OpenAPI（appId+appKey+时间戳）。计费规则（起步价+基础公里+
超程单价+封顶）**云端复算**，前端只展示。退货走跑腿=用户预付，勿到付。
环境变量：UUPT_APP_ID / UUPT_APP_KEY / UUPT_BASE / UUPT_CALLBACK_URL / UUPT_OPEN_ID

## ★ serviceApi + sendSubscribeMsg（通知）
serviceApi: bind/getBind/getOAuthUrl/oauth/genQrCode/sendTemplate/
  notifyNewOrder/notifyReject；经服务器 PHP 中转（IP 白名单）。
  环境变量：SERVICE_APPID/SERVICE_APPSECRET/SERVICE_TOKEN/SERVICE_BASE/SEND_HOST
sendSubscribeMsg: userOrderStatus/userRefund/userAppointment/staffNewOrder/
  staffReturn/staffAppointment。环境变量：MINI_APPID/MINI_APPSECRET。
触发点：下单→staffNewOrder；状态变→userOrderStatus；退款→双端。

## 工具类云函数
storeApi(getCloseTimes/setCloseTime) | invoiceApi(myAvailableOrders/submit/myList/
h5Issue/h5List，环境变量 ADMIN_PWD) | getOpenid(wxContext 直返) |
phoneLogin(解密手机号→users upsert) | mpVerify(内容安全占位) |
reverseGeocode(经纬度→地址，首页定位用) | syncData | importProducts/batchImport
(批量导商品: import/count/reset) | priceUpdate(批量改价/限时窗) |
ordersBackfill(历史回填) | fixSpaceFiles(修云存储含空格路径: scan/fixOne/updateDB) |
initDB(建集合索引) | h5admin(HTTP 网关，转发 adminApi，供 PHP 中转调用)

═══════════════════════════════════════════
第 6 章 · 业务规则（改一处全盘皆错）
═══════════════════════════════════════════
1. 价格三分支：限时价(时间窗)>会员价；划线价仅当原价>生效价；整箱=packSpecs。
   显示/加购/下单/结算全走 utils/price.js。
2. 库存：0 无货/>0 有货/无记录售罄；扣减只在支付回调 deductBatch（不足整单拒）；
   整箱按瓶扣=数量×规格size；生效店=手选>定位>默认首店。
3. 运费：自提 0；配送 小计≥200 免运费，否则 6 元（距离计费可扩展）；
   结算页显示"还差 X 元免运费"(freeShipGap=200-小计)。
4. 购物车跨店：条目绑 storeId、key=商品+规格+店；一店一结（分组链式下单）；
   切店 onShow 同步；旧无店数据归首店。
5. 订单状态机：pending_pay→pending_ship→in_transit→completed（+cancelled/退款）。
   自提单核销码 verifyCode + 拍照 images。
6. 权限：见 adminApi 权限矩阵，勿"修正"。
7. 积分：高价品类关键词 1元=1分，其余 1元=5分；等级 0/100001/1500001 三档；
   退款扣回。
8. 退款：未发货全额含运费，已发货退商品不退运费；幂等。
9. 评价：一单一品一条；内容安全仅 risky 拦；评分=(5+平均)/2。
10. 图片：全部转 tcb.qcloud.la 永久直链+encPath 编码；主图/详情各≤20；
    编辑未传图保留原图；404=重传；451=云开发欠费冻结(充值恢复)。
11. 跑腿：云端复算运费；退货跑腿用户预付。
12. 公开接口 pub*（公告/活动/广告/榜单）放鉴权前；前端切 tab 拉取，失败回落静态。
13. 提货卡：卡号搜索后端正则进 where；卡密环境变量加密；
    状态机 未激活→已激活→已核销/已冻结。
14. 拼音搜索：元素级幂等 _py；云缓存 10min+60s 节流。

═══════════════════════════════════════════
第 7 章 · 31 页逐页规格
═══════════════════════════════════════════
（每页 = 目录 pages/<name>/ 下 4 件套；每页 data 带 cartCount 角标，onShow 刷新；
自定义导航页用 app.globalData.statusBarHeight 算顶部高度）

## Tab 页
1. **index 首页**：定位(腾讯SDK+reverseGeocode)→activeStore/nearbyStore；
   分类宫格、banner 轮播(pubAds 首页轮播位)、热销、榜单入口、
   弹窗广告(showAdPopup/adImage/adProductId→详情)、加购飞入动画
   (buyItem/cartGif/flyGoods)、库存 batchGet→stockMap
2. **category 分类**：门店切换半屏(showStoreSheet，按距离排序)、
   左右双级分类(mainCategories/subCategories)、拼音搜索 keyword、
   筛选(sortBy/priceMin/priceMax/showFilter)、结果计数、直达结算
3. **cart 购物车**：跨店分组、勾选(selectedIds/selectAll)、合计
   (totalPrice/pointsTotal/availableCoupons/appliedCoupon/discountedTotal)、
   自提弹窗 showPickupModal、库存 list 刷新
4. **membership 会员**：孟菲斯风等级卡(getTierGradientStyle)、积分进度条
   progressWidth、等级路线图 showRoadmap、领券区 claimableCoupons
5. **profile 我的**：登录态/角色徽章(isStaff/isAdmin/canSeed)、系统头像选择
   弹窗(showAvatarModal+sysAvatars)、功能入口矩阵(订单/券/积分/收藏/足迹/
   地址/退货/榜单/店员端/店长端)

## 交易链
6. **wine-detail 详情**：主图轮播=image 全量、详情图 images 分离展示、
   收藏心形动画、packSpecs 整箱选择、多店库存 storeStocks(findStores)、
   售罄弹窗 soldOutModal(soldOutStores)、评价聚合(reviewAvg/featuredReview)、
   加购/立即购买
7. **checkout 结算**：按店分组链式 createOrder；积分商品 pointsApi
   deductForOrder；地址选择/新增；券选择(usable/unusable)；满200免运费
   freeShipGap；配送方式(自提/配送/跑腿+uuptApi price 询价)；备注；提交→to-pay
8. **to-pay 待付款**：订单列表+倒计时、wx.requestPayment 拉起
   (payApi query 确认)、取消订单、支付成功 deductBatch+staffNewOrder 订阅消息
9. **to-ship / to-receive**：按状态列表、退款入口(payApi refund)
10. **order-detail 详情**：状态机展示 statusText/deliveryLabel、verifyCode、
    追评入口(reviewsApi checkOrder)、跑腿轨迹(uuptApi track→uuptSteps/uuptMap)
11. **order-track / uupt-track**：订单跟踪列表；跑腿地图轨迹
    (markers/polyline/driverName/driverMobile/etaText)

## 个人/营销
12. **address**：地址 CRUD+表单校验(phoneValid/formErrors)+省市区选择
13. **coupons**：tabs 可用/不可用/已领，claimCoupon
14. **points**：流水列表、等级进度、明细弹窗
15. **points-mall**：pointsApi list/exchange
16. **favorites / browse-history**：列表+batchGet 刷库存；足迹分组今天/更早+清空
17. **edit-profile**：昵称/头像/简介编辑
18. **invite**：邀请海报图展示
19. **activity**：活动页(themeStyle 主题化、actGoods)
20. **stores**：门店列表+关键词搜索
21. **ranking 榜单**：波普叙事风(品牌蓝+网点+黑边硬阴影+撞色)、
    rankTabs 多榜单、pubRankings 公开读、banner 兜底
22. **review-write**：星级动画、标签、图片上传(mediaCheck)、msgCheck/riskCheck 后提交
23. **review-list**：byProduct 列表+平均评分(公式见规则9)
24. **return-exchange**：类型(退/换)、原因、方式(门店/快递/跑腿-预付询价)、
    申诉图上传、我的申请列表
25. **return-audit**(店长)：approve/reject/receive+refund

## 管理端
26. **store-staff 店员端**：myStore→30s 轮询 listOrders；tab(待备货/配送中/
    已完成)；updateOrder 状态流转；核销码输入；订阅消息开关
27. **store-admin 店长端**：多 tab(订单/库存/店员/打烊/退货/商品)；
    店员 CRUD(seedAdmin/canSeed)；库存 adjust；打烊 setCloseTime；
    核销 setVerifyCode+uploadImages
28. **service-bind**：web-view 内嵌服务号授权绑定页

## 隐藏页（不注册 app.json）
29. **invoice-apply / invoice-list**：invoiceApi myAvailableOrders/submit/myList

═══════════════════════════════════════════
第 8 章 · 组件 / H5 / 打印 / 部署 / 数据导入
═══════════════════════════════════════════

## 组件
- qty-modal：规格+数量选择弹窗（app.json 全局注册）
- loading-lottie：Lottie 加载动画组件

## H5 品牌站+后台（h5-app）
React18+Vite+Tailwind v4，单页多 tab 懒加载：Home/Notice/Activities/
Stores(地图 JS API，2D/3D)/Admin/Footer(ICP 备案)。
API 层：fetch → 服务器 PHP 中转文件 → h5admin 云函数 → adminApi。
（PHP 文件只在服务器，不在仓库，部署勿覆盖。）
登录：超管=专用账号(手机号留空)；管理员/店员=手机号+密码。
角色：super 见全局模块，店长只见本店。
后台发现新订单→POST http://127.0.0.1:8899/print；提示音走本机。

## 打印服务（print-server）
Node http 服务，端口 8899：
  POST /print { _id, storeId, ... } → 按门店映射选打印机 → ESC/POS → 出票
  GET /printers | GET/POST /config(门店→打印机映射 {default, map:{s1:'...'}}) |
  POST /test | GET /status | POST /autodetect
运维：开机自启+保活脚本；先停旧服务防 8899 冲突；bat 中文存 GBK；
ps1 用 Unrestricted 执行策略；附带杀软白名单指引。

## 部署（按序）
1. 开小程序+云开发环境；替换全局 AppID/环境ID/直链域名
2. 云函数按依赖序部署：getOpenid→initDB→productApi→inventoryApi→adminApi→
   payApi/payNotify→orderApi/staffApi→pointsApi→reviewsApi→returnApi→
   serviceApi/sendSubscribeMsg→uuptApi/uuptCallback→invoiceApi→其余工具函数
3. 环境变量分组配置（值部署时填，勿入代码）：
   adminApi 超管组 / payApi+payNotify 支付四件套 / uuptApi 跑腿五件套 /
   serviceApi 服务号组 / sendSubscribeMsg 小程序组 / 各后台口令
4. 建 21 集合；orders 设"仅创建者可读写"
5. 微信支付 V3：商户私钥+序列号+APIv3 密钥；回调路由→payNotify
6. 订阅消息模板 ID 配 sendSubscribeMsg
7. H5 构建部署+PHP 中转；地图 Key 注入页面
8. 打印服务装门店、配映射
9. **数据导入**：商品 Excel→importProducts 管线（或 h5GoodsImport CSV，
   按 id 匹配更新、无删除语义、导出即模板）；门店写 stores.js+同步 H5；
   商品图传云存储(主图/详情各≤20/款)；提货卡 cardBatchCreate 批量制卡
10. 全链路联调：下单→支付→扣库存→店员接单→核销→退款→积分→评价→
    榜单→提货卡→跑腿

═══════════════════════════════════════════
第 9 章 · 避坑清单（血泪）
═══════════════════════════════════════════
- wxss 改前重读磁盘+括号配平；删样式块精确匹配
- 测试改真实数据先记原值，用完还原
- 门店 id 历史跳号勿重排
- React context 漏 value 传参→页面白屏（tsc 过运行时崩），加字段后 grep value 段
- 云库客户端 limit 100→skip 分页；_.in 限 20→分批；users 查 openid 兼容 _openid
- adminApi 新 action 放主 try 内，否则部署后 404
- 本地打印/提示音走 127.0.0.1 仅门店本机有效
- 451=云开发欠费冻结，充值恢复，非代码 bug
- 商品 CSV 导入=按 id 匹配更新、无删除语义，导出文件即导入模板

═══════════════════════════════════════════
第 10 章 · 验收清单
═══════════════════════════════════════════
- 开发者工具每批改动即编译；真机全链路截图
- 后台用超管+店长两角色各验一遍权限边界
- 支付/退款/跑腿沙箱跑通再上线
- 页面：31 注册页+2 隐藏页；Tab 5 个；角标/弹窗/动画逐项过
