# workbuddy.md

> 本文件记录使用腾讯 WorkBuddy 开发本项目的对话上下文，用于核验是否符合联动活动奖励条件。

---

## 项目信息

| 项目 | 内容 |
|------|------|
| 项目名称 | 麦麦积分榨汁机 · McDonald's Points Maximizer |
| 参赛赛事 | 2026 麦当劳程序员创意开发大赛 |
| 开发工具 | 腾讯 WorkBuddy 智能设计助手 |
| MCP Server | 麦当劳中国官方 MCP（`https://mcp.mcd.cn`） |

---

## 开发过程中的 WorkBuddy 使用记录

### 阶段一：赛事规则研究

- 通过对话让 WorkBuddy 检索并解析官方赛事规则、排行榜现状与 MCP 工具清单
- WorkBuddy 拉取 GitHub 官方仓库内容，识别出关键信息：**排名仅按 GitHub Star 数计算**
- 据此调整策略：从「拼技术」转向「快速做出可用项目 + 重点投入传播」

### 阶段二：项目结构设计

- 与 WorkBuddy 讨论并确定选题方向
- 从多个候选（积分榨汁机/ 聚会策划官 / 聪明点餐官）中选定「麦麦积分榨汁机」，理由：
  - 痛点最普遍（积分闲置是麦门用户的共性问题）
  - 可调用的 MCP 工具最丰富（覆盖 5 条业务线）
  - 输出「体检报告」形式天然适合截图传播

### 阶段三：文档撰写

以下文件均在 WorkBuddy 中完成撰写与迭代：

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 核心技能定义、工作流、输出格式、合规边界 |
| `README.md` | 项目介绍、安装方法、使用示例、目标用户 |
| `MCP_INTEGRATION.md` | MCP 工具清单、3 条调用流程、业务价值分析 |
| `references/tool-reference.md` | 工具速查表、并行调用策略、返回格式差异说明 |
| `CONTEST_DECLARATION.md` | 官方文件直接下载，内容未做任何修改 |
| `mcp-config.example.json` | 脱敏配置示例 |
| `SECURITY.md` | Token 安全说明 |

### 阶段四：MCP 联调与实测 🔥

这是本项目最有价值的部分 —— **WorkBuddy 实际连接麦当劳 MCP 并运行了真实数据查询**。

**配置过程**：在WorkBuddy 指导下将 MCP 配置写入本地 `~/.workbuddy/mcp.json`，启用 `mcd-mcp` 连接器。

**实测调用记录**：

| 工具 | 实测结果 |
|------|---------|
| `now-time-info` | 返回 `2026-10-09 15:41:51`，GMT+08:00 |
| `query-my-account` | 可用积分 0、累计 0、本月将过期 0 |
| `query-my-coupons` | 2 张可用券（新人专享美式/奶铁 7 折），有效期至 11-07 |
| `available-coupons` | 9 张可领取 |
| `query-lottery-info` | 麦麦积分抽奖进行中，24 积分/次，10 种奖品 |
| `mall-points-products` | 50 个可兑换项（26 个 0 积分活动 + 24 个需积分） |
| `campaign-calendar` | 当月活动列表，含 PEACEMINUSONE 联名 |

**实测暴露并修复的两个设计缺陷**（这是联调的真正价值）：

1. 🔥 **临期积分有三个字段**，原设计只考虑了「本月将过期」，遗漏了 `nextMouthExpirePoint` 与`lastMouthExpirePoint`。
   其中 `lastMouthExpirePoint > 0` 是**最强的痛点信号**，已修订进 SKILL.md，要求置顶强调。

2. 🔥 **`query-my-coupons` 返回 Markdown 文本而非结构化 JSON**，有效期字段格式为
   `2026-10-09 00:00-2026-11-07 23:59 05:00-23:59`，
   需正则解析出结束日期与每日可用时段。已把解析规则写入 tool-reference.md。

> 这两个问题**只有真实调用才会暴露**。文档层面的设计无法发现接口返回格式的真实形态。

### 阶段五：产出与推广

- 基于**真实查询数据**生成《样例输出_积分体检报告.md》
- 制定推广策略，产出《推广文案包.md》（朋友圈 5 条 + 微信群 2 版 + 16 天节奏表）
- 完成合规自检：文件齐全性、Token 泄露检查、声明文件与官方原文一致性校验

---

## 产出物清单

| 文件 | 用途 |
|------|------|
| `SKILL.md` | 核心技能定义 |
| `README.md` | 项目文档 |
| `MCP_INTEGRATION.md` | MCP 集成说明 |
| `references/tool-reference.md` | 工具参考 |
| `CONTEST_DECLARATION.md` | 参赛声明（官方原文） |
| `mcp-config.example.json` | 脱敏配置示例 |
| `SECURITY.md` | 安全说明 |
| 样例输出_积分体检报告.md | 真实数据产出的报告样例 |
| MCP联调验证记录.md | 完整联调记录 |
| 推广文案包.md | 传播素材 |

---

## 声明

本项目确实使用腾讯 WorkBuddy 智能体完成开发：涵盖赛事调研、选题设计、文档撰写、MCP 配置与真实联调、缺陷修复、报告生成与推广策划全过程。

项目数据均来自麦当劳中国官方 MCP 服务的真实返回，未使用任何模拟或编造数据。

本项目为参赛作品，非麦当劳官方产品，与麦当劳中国无隶属关系。