---
name: mcdonalds-mcp
description: 麦当劳 MCP 点餐助手。通过麦当劳官方 MCP Server 浏览菜单、营养查询、优惠券领取、积分兑换、下单点外卖（18 个工具）。使用前需先在 https://open.mcd.cn/mcp 获取 MCP Token（免费无门槛）。
metadata:
  {
    "openclaw":
      {
        "emoji": "🍔",
        "requires": { "bins": ["mcporter"] },
      },
  }
---

# 麦当劳 MCP 点餐助手 🍔

通过麦当劳官方 MCP Server（`https://mcp.mcd.cn`）实现智能点餐，共 18 个工具覆盖配送、点餐、优惠券、积分、营养、活动。

## 快速开始

### 1. 获取 MCP Token（必须）
打开 https://open.mcd.cn/mcp 登录麦当劳账号，点击激活即可获取 MCP Token，**无任何门槛，免费获取**。

> ⚠️ 没有 MCP Token 无法使用本 Skill。如果 Token 为空或过期，请先去上述地址获取。

### 2. 配置 mcporter
```bash
mcporter config add mcdonalds --url https://mcp.mcd.cn --transport http --header "Authorization=Bearer <MCP_TOKEN>"
```

### 3. 验证连接
```bash
mcporter list mcdonalds          # 列出 18 个工具
mcporter call mcdonalds.now-time-info  # 测试调用
```

### 4. 调用工具
```bash
mcporter call mcdonalds.<tool_name> key=value
# 示例
mcporter call mcdonalds.available-coupons
mcporter call mcdonalds.list-nutrition-foods
```

## 协议详情

| 项目 | 值 |
|------|------|
| Endpoint | `https://mcp.mcd.cn` |
| 协议 | Streamable HTTP |
| 认证 | `Authorization: Bearer <TOKEN>` |
| MCP Version | 2025-06-18 及之前 |
| 限流 | 600 req/min/token |

## 可用工具（18 个）

#### 🛵 配送与点餐
| 工具 | 说明 |
|------|------|
| `delivery-query-addresses` | 查询用户配送地址列表 |
| `delivery-create-address` | 创建配送地址 |
| `query-meals` | 查询餐品列表（需 storeCode + beCode） |
| `query-meal-detail` | 查询餐品详情 |
| `calculate-price` | 计算商品价格（含优惠） |
| `create-order` | 创建外卖订单 |

#### 🎫 优惠券
| 工具 | 说明 |
|------|------|
| `available-coupons` | 查询可领取的优惠券列表 |
| `auto-bind-coupons` | 自动领取所有可用优惠券 |
| `query-my-coupons` | 查询卡包中的优惠券 |
| `query-store-coupons` | 查询门店可用优惠券（需 storeCode + beCode） |

#### 🏆 积分商城
| 工具 | 说明 |
|------|------|
| `query-my-account` | 查询积分账户详情 |
| `mall-points-products` | 查询可兑换餐品券列表 |
| `mall-product-detail` | 查询商品详情（需 spuId） |
| `mall-create-order` | 创建积分兑换订单（需 skuId） |

#### 📊 其他
| 工具 | 说明 |
|------|------|
| `query-order` | 查询订单详情（需 34 位订单号） |
| `campaign-calendar` | 查询当月营销活动日历 |
| `list-nutrition-foods` | 查询餐品营养成分数据 |
| `now-time-info` | 获取当前服务器时间 |

### 核心流程

#### 麦乐送点餐流程
1. **查询地址** → `delivery-query-addresses`（获取 storeCode + beCode）
2. **浏览菜单** → `query-meals`（用 storeCode + beCode）
3. **查看详情** → `query-meal-detail`
4. **查优惠券** → `query-store-coupons`（可选）
5. **计算价格** → `calculate-price`
6. **创建订单** → `create-order`
7. **查订单** → `query-order`

#### 积分兑换流程
1. **查积分** → `query-my-account`
2. **查商品** → `mall-points-products`
3. **商品详情** → `mall-product-detail`
4. **下单兑换** → `mall-create-order`

## 注意事项
- MCP Token 代表麦当劳会员身份，严禁分享
- 服务面向中国大陆地区（不含港澳台）
- 下单时需先 `calculate-price` 计算价格并让用户确认
- 多个商品兑换需分别下单，等前一个订单完成后再下新的
- 门店信息（storeCode/beCode）必须从 `delivery-query-addresses` 返回值获取，不可凭空生成
- 参考文档：https://open.mcd.cn/mcp/doc
