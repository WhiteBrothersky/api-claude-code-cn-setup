# claude-code-cn-setup

国内接入 Claude Code 的配置记录：环境变量怎么设、常见报错怎么排查、Cherry Studio 等 OpenAI 兼容客户端怎么填。

在线指南（可直接照着操作）：<https://whitebrothersky.github.io/api-guide/>

---

## 快速配置

### 1. 获取一个 API Key

在中转服务控制台新建 API Key 并复制保存。所有客户端共用同一个 Key，建议按用途分开建，方便定位是哪一端在消耗额度。

### 2. 配置环境变量

macOS / Linux：

```bash
echo 'export ANTHROPIC_BASE_URL=https://<你的中转地址>/v1' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY=<你的key>' >> ~/.zshrc
source ~/.zshrc
```

Windows（PowerShell）：

```powershell
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://<你的中转地址>/v1", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "<你的key>", "User")
```

Windows 下设置后**必须重开终端窗口**才会生效。

### 3. 验证

```bash
claude "写一个函数，输入字符串返回反转结果"
```

能正常返回即配置成功。

---

## 常见报错对照

| 现象                    | 可能原因              | 处理方式                     |
| --------------------- | ----------------- | ------------------------ |
| 401 Unauthorized      | Key 错误或复制时带了空格/换行 | 重新复制，注意 trim             |
| 404 model not found   | 模型名与中转站提供的列表不一致   | 按站内模型清单填实际名称             |
| 429 Too Many Requests | 触发限流              | 降低并发，或确认额度是否用尽           |
| 连接超时                  | 中转地址填错，或服务不可用     | 核对 Base URL 是否带 `/v1` 后缀 |
| 环境变量不生效               | 只作用于当前 shell      | 写入 shell 配置或重启终端         |

---

## OpenAI 兼容客户端

Cherry Studio、NextChat、LobeChat、沉浸式翻译等客户端，在「API 地址」填 `https://<你的中转地址>/v1`，在「API Key」填你的 Key，模型按站内清单选择即可。

---

## 几点提醒

1. **先小额验证**。任何第三方中转都存在服务变动、限流降智的可能，先用最小额度跑一到两天再决定是否加量。
2. **不要把重要业务绑死在单一渠道**，保留备用方案。
3. **不要提交敏感内容**。公司代码、用户数据、内部凭证不要通过第三方中转提交。
4. **定期核实用量**，留意后台统计与实际消耗是否一致。

---

## Topics

`claude-code` `anthropic` `openai-api` `llm` `cherry-studio` `api-guide`
