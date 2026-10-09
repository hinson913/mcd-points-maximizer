# 工具清单与调用参考

本文件为 `SKILL.md` 的补充参考，记录可用 Tool 的调用顺序与注意事项。

---

## 调用速查表

### 必用工具（任何查询都会用到）

| Tool | 触发时机 | 备注 |
|------|---------|------|
| `now-time-info` | 涉及"过期""还剩几天"时 | 必须最先调用，否则过期判断不可靠 |

### 只读工具（可安全并发）

```
now-time-info
query-my-account
query-my-coupons
available-coupons
mall-points-products
mall-product-detail
mall-order-list
mall-order-detail
query-lottery-info
query-my-prizes
campaign-calendar
query-store-coupons
```

### 写操作工具（必须先征得用户同意）

| Tool | 副作用 | 处理方式 |
|------|-------|---------|
| `auto-bind-coupons` | 改变账户权益 | 明确说明"会帮你领取 N 张券"，得到确认后调用 |
| `draw-lottery` | 消耗积分 | 先给出期望值判断，用户确认后再抽 |
| `mall-create-order` | 扣减积分 + 库存 | 展示商品与所需积分，确认后下单 |

---

## 并行调用策略

MCP 工具调用之间若无数据依赖，**应在同一轮内并行发起**，不要串行等待。

**典型并行组**：

```json
{
  "toolGroup": [
    "now-time-info",
    "query-my-account",
    "query-my-coupons",
    "available-coupons"
  ],
  "reason": "四者互不依赖，同时发起可显著降低响应延迟"
}
```

**必须串行的场景**：

```json
{
  "sequence": [
    {
      "step": 1,
      "call": "mall-points-products",
      "reason": "先拿到商品池"
    },
    {
      "step": 2,
      "call": "mall-product-detail",
      "reason": "需要上一步返回的商品编码作为入参，逐个查询详情"
    }
  ]
}
```

---

## 数据为空时的处理

MCP 返回空值时，**如实告知，不得编造或推测**。

| 场景 | 正确响应 |
|------|---------|
| `available-coupons` 返回空 | "当前没有可领取的优惠券" |
| `mall-points-products` 返回空 | "你的积分可能不支持兑换该类商品，或当前无可兑换商品" |
| `query-lottery-info` 活动未开始 | "抽奖活动当前未开放，可以在活动日历里查看开放时间" |
| `query-my-account` 积分不足 | 如实说明可用积分，不推荐买不起的商品 |

---

## 错误码处理

| 错误码 | 原因 | 处理建议 |
|-------|------|---------|
| 401 | Token 无效、过期或未提供 | 提示用户检查 Token 配置与有效期 |
| 429 | 超过 600 次/分钟限流 | 降低频率，避免循环调用 |
|工具不存在 | MCP Server 版本较旧 | 提示用户升级，或改用其他等价工具 |

---

## 计算逻辑说明

### 兑换性价比

```
性价比 = 商品参考价值 / 所需积分
```

在商品未提供明确参考价值时，可按以下方式表述（避免编造价格）：

> 该商品需 X 积分，按你的可用积分 Y 计算，约可兑换 Z 件。

### 抽奖期望值

```
单次期望价值 = Σ(各奖品价值 × 中奖概率) / 每次消耗积分
```

若概率数据未由 MCP 返回，**不做概率推断**，仅陈述消耗与奖品清单，让用户自行判断。

### 临期判定

`query-my-account` 返回**三个**临期相关字段，需综合判断：

| 字段 | 含义 | 提示级别 |
|------|------|---------|
| `currentMouthExpirePoint` | 本月将过期 | 高优先级置顶 |
| `nextMouthExpirePoint` | 下月将过期 | 需关注 |
| `lastMouthExpirePoint` | 上月已过期 | **最强痛点信号**，置顶强调 |

```
剩余天数 = 过期日期 - now-time-info 返回的当前日期
```

- 剩余 ≤ 7 天：标记为高优先级
- 剩余 ≤ 30 天：标记为需关注
- 其余：常规展示

> 💡 `lastMouthExpirePoint > 0` 意味着用户**已经在漏接权益**。
> 建议表述为「你上个月有 X 积分白白过期了」，比「你有 X 分即将过期」更能触动用户。

---

## 返回格式差异（实测确认）

不同工具的返回格式不一致，需分别处理：

| 工具 | 返回格式 | 处理方式 |
|------|---------|---------|
| `query-my-account` | 结构化 JSON | 直接取 `data` 字段 |
| `mall-points-products` | 结构化 JSON | `data` 为数组，含 `spuName`/`point`/`catName`/`downTime` |
| `query-lottery-info` | 结构化 JSON | `data.drawPoint` 为单次消耗，`data.prizes` 为奖品数组 |
| `query-my-coupons` | ⚠️ **Markdown 文本** | 需正则解析，见下 |
| `available-coupons` | ⚠️ **Markdown 文本** | 统计「状态：可领取」出现次数 |

### 优惠券有效期解析规则

`query-my-coupons` 的有效期字段格式：

```
2026-10-09 00:00-2026-11-07 23:59 05:00-23:59
   ^开始日期       ^结束日期      ^每日可用时段
```

```python
import re
pattern = r"\*\*有效期\*\*:\s*([\d\-]+ [\d:]+) - ([\d\-]+ [\d:]+) ([\d:]+-[\d:]+)"
matches = re.findall(pattern, text)
# matches -> [(开始, 结束, 每日时段), ...]
```

**注意**：`05:00-23:59` 表示该券**仅在每日 05:00–23:59 可用**，不是全天通用。
若结束日期距今 ≤ 7 天且时段受限，应在报告中明确提醒。

---

## 版本信息

本项目基于麦当劳中国 MCP Server `v1.0.9`（截至 2026-09-10）的能力清单编写。若官方后续新增工具，可在此文件补充。