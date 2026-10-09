#环境变量占位符说明

本项目**不包含任何真实Token、密钥或账号凭证**。

## 获取你自己的 MCP Token

1. 打开 https://open.mcd.cn/mcp
2. 右上角点击「登录」，使用手机号完成验证
3. 登录后右上角变为「控制台」，点击打开控制台弹窗
4. 点击「激活」按钮申请 MCP Token
5. 阅读并同意服务协议，复制 Token

## 配置方式

将 `mcp-config.example.json` 复制为你的本地 MCP 配置，把 `${MCD_MCP_TOKEN}` 替换为你的实际 Token。

推荐使用环境变量注入（更安全）：

```bash
export MCD_MCP_TOKEN="你的实际Token"
```

或在 WorkBuddy 中通过【自定义连接器】→【配置MCP】填入：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

⚠️ Token 请妥善保管，避免泄露。Token 无效或过期会返回 401，超出频率限制会返回 429（每 Token 每分钟最多 600 次请求）。