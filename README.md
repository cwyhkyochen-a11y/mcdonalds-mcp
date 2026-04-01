# 🍔 麦当劳 MCP 点餐助手

通过[麦当劳官方 MCP Server](https://open.mcd.cn/mcp)实现智能点餐，支持浏览菜单、查询营养、领取优惠券、积分兑换、下单外卖。

> 一个 [OpenClaw](https://github.com/openclaw/openclaw) Skill，基于 `mcporter` CLI 调用麦当劳 MCP Server。

## 功能概览

| 分类 | 能力 |
|------|------|
| 🛵 配送点餐 | 查询地址、浏览菜单、查看详情、计算价格、创建外卖订单 |
| 🎫 优惠券 | 查询可用优惠券、一键领取、查看卡包、门店优惠券 |
| 🏆 积分商城 | 查积分、兑换商品、查看详情、创建兑换订单 |
| 📊 其他 | 订单查询、营销日历、营养成分、服务器时间 |

共 **18 个工具**，覆盖麦当劳会员体系核心功能。

## 前置条件

1. **[OpenClaw](https://docs.openclaw.ai)** 已安装并运行
2. **[mcporter](https://github.com/openclaw/mcporter)** CLI 已安装
3. **麦当劳 MCP Token** — 免费获取，无门槛

## 快速开始

### 第一步：获取 MCP Token

打开 [https://open.mcd.cn/mcp](https://open.mcd.cn/mcp)，登录麦当劳账号，点击激活即可获取 Token。

> ⚠️ 没有 Token 无法使用。如果 Token 过期，请重新获取。

### 第二步：配置 mcporter

```bash
mcporter config add mcdonalds \
  --url https://mcp.mcd.cn \
  --transport http \
  --header "Authorization=Bearer <你的MCP_TOKEN>"
```

### 第三步：验证连接

```bash
# 列出所有可用工具（应显示 18 个）
mcporter list mcdonalds

# 测试调用
mcporter call mcdonalds.now-time-info
```

### 第四步：安装 Skill

```bash
clawhub install mcdonalds-mcp
```

## 使用示例

### 麦乐送点餐

```bash
# 1. 查询配送地址（获取 storeCode + beCode + addressId）
mcporter call mcdonalds.delivery-query-addresses

# 2. 浏览菜单
mcporter call mcdonalds.query-meals storeCode=<STORE> beCode=<BE>

# 3. 查看菜品详情（促销商品可能查不到，用 query-meals 替代即可）
mcporter call mcdonalds.query-meal-detail storeCode=<STORE> beCode=<BE> code=<CODE>

# 4. 查看门店优惠券
mcporter call mcdonalds.query-store-coupons storeCode=<STORE> beCode=<BE>

# 5. 计算价格（⚠️ 必须用 --args 传完整 JSON，items 为对象数组）
mcporter call mcdonalds.calculate-price --args '{
  "storeCode": "<STORE>",
  "beCode": "<BE>",
  "addressId": "<ADDRESS_ID>",
  "items": [{"productCode": "<PRODUCT_CODE>", "quantity": 1}]
}'

# 6. 创建订单（参数与 calculate-price 完全相同）
mcporter call mcdonalds.create-order --args '{
  "storeCode": "<STORE>",
  "beCode": "<BE>",
  "addressId": "<ADDRESS_ID>",
  "items": [{"productCode": "<PRODUCT_CODE>", "quantity": 1}]
}'

# 7. 查询订单状态
mcporter call mcdonalds.query-order --args '{"orderId":"<订单号>"}'
```

### 优惠券

```bash
# 查看可领取的优惠券
mcporter call mcdonalds.available-coupons

# 一键领取所有可用优惠券
mcporter call mcdonalds.auto-bind-coupons

# 查看卡包中的优惠券
mcporter call mcdonalds.query-my-coupons
```

### 积分兑换

```bash
# 查看积分余额
mcporter call mcdonalds.query-my-account

# 浏览可兑换商品
mcporter call mcdonalds.mall-points-products

# 查看商品详情
mcporter call mcdonalds.mall-product-detail spuId=<SPU_ID>

# 下单兑换
mcporter call mcdonalds.mall-create-order skuId=<SKU_ID>
```

## 协议详情

| 项目 | 值 |
|------|------|
| Endpoint | `https://mcp.mcd.cn` |
| 协议 | Streamable HTTP |
| 认证 | `Authorization: Bearer <TOKEN>` |
| MCP Version | 2025-06-18 及之前 |
| 限流 | 600 req/min/token |

## 工具列表

### 🛵 配送与点餐

| 工具 | 说明 |
|------|------|
| `delivery-query-addresses` | 查询用户配送地址列表 |
| `delivery-create-address` | 创建配送地址 |
| `query-meals` | 查询餐品列表（需 storeCode + beCode） |
| `query-meal-detail` | 查询餐品详情 |
| `calculate-price` | 计算商品价格（含优惠） |
| `create-order` | 创建外卖订单 |

### 🎫 优惠券

| 工具 | 说明 |
|------|------|
| `available-coupons` | 查询可领取的优惠券列表 |
| `auto-bind-coupons` | 自动领取所有可用优惠券 |
| `query-my-coupons` | 查询卡包中的优惠券 |
| `query-store-coupons` | 查询门店可用优惠券 |

### 🏆 积分商城

| 工具 | 说明 |
|------|------|
| `query-my-account` | 查询积分账户详情 |
| `mall-points-products` | 查询可兑换餐品券列表 |
| `mall-product-detail` | 查询商品详情 |
| `mall-create-order` | 创建积分兑换订单 |

### 📊 其他

| 工具 | 说明 |
|------|------|
| `query-order` | 查询订单详情（需 34 位订单号） |
| `campaign-calendar` | 查询当月营销活动日历 |
| `list-nutrition-foods` | 查询餐品营养成分数据 |
| `now-time-info` | 获取当前服务器时间 |

## 常见问题

### `calculate-price` / `create-order` 报"缺少参数"

这两个工具**必须用 `--args` 传完整 JSON**，字段名是 `items`（不是 `productList` 或 `products`），且需包含 `storeCode`、`beCode`、`addressId` 三个顶层字段。

```bash
# ✅ 正确
mcporter call mcdonalds.calculate-price --args '{"storeCode":"1450555","beCode":"145055502","addressId":"xxx","items":[{"productCode":"9900013722","quantity":1}]}'

# ❌ 错误 — key=value 格式会被当成字符串
mcporter call mcdonalds.calculate-price storeCode=1450555 items='[{"productCode":"xxx","quantity":1}]'
```

### `query-meal-detail` 返回"未匹配到商品"

部分促销/限时商品（买一送一套餐等）不在详情库中。用 `query-meals` 获取商品名和价格即可，不依赖 `query-meal-detail`。

### 支付链接变成扫码页

`create-order` 返回的 `payH5Url` 目前为 `scanToPay` 扫码页。手机上打开可能自动调起支付；也可在麦当劳 App「我的订单」中直接支付。

## 注意事项

- MCP Token 代表麦当劳会员身份，**严禁分享**
- 服务面向中国大陆地区（不含港澳台）
- 下单前务必先 `calculate-price` 确认价格
- 多商品兑换需分别下单，等前一个完成后再下新的
- 门店信息（storeCode / beCode）必须从 `delivery-query-addresses` 返回值获取，**不可凭空生成**
- 官方文档：[open.mcd.cn/mcp/doc](https://open.mcd.cn/mcp/doc)

## 相关链接

- [OpenClaw](https://github.com/openclaw/openclaw) — AI Agent 框架
- [ClawHub](https://clawhub.com) — Skill 市场
- [mcporter](https://github.com/openclaw/mcporter) — MCP 调用工具
- [麦当劳 MCP 官方文档](https://open.mcd.cn/mcp/doc)

## License

MIT
