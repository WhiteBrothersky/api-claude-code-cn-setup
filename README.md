# api-claude-code-cn-setup

国内接入 Claude Code 的配置记录：环境变量怎么设、常见报错怎么排查、Cherry Studio 等 OpenAI 兼容客户端怎么填。

在线指南（可直接照着操作）：<https://whitebrothersky.github.io/api-claude-code-cn-setup/>

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

## 在 WorkBuddy 里接入

WorkBuddy 支持接入自定义模型，全程图形界面操作，不用手改配置文件。配置（含 API Key）只保存在本地，不上传云端。

### 入口路径

![WorkBuddy 配置入口](./docs/images/workbuddy-entry.svg)

1. 对话界面底部点击**模型选择器**（显示当前模型名的位置）
2. 在模型列表底部找到 **「配置自定义模型」**，进入设置页的「模型」
3. 右上角点 **「+ 添加模型」**

### 填写表单

![WorkBuddy 添加模型](./docs/images/workbuddy-add-model.svg)

| 字段      | 填什么                                                |
| ------- | -------------------------------------------------- |
| 提供商     | 选预设厂商会自动带出接口地址，只补 Key 即可；接中转服务选 **「自定义 / Custom」** |
| 接口地址    | `https://<你的中转地址>/v1`                              |
| API Key | 中转站控制台里创建的 Key                                     |
| 模型名称    | 按站内模型清单填，大小写敏感                                     |

> **最容易踩的坑**：一般客户端填到 `/v1` 就可以（客户端自己拼路径），不同客户端可能格式不一样。

### 高级开关怎么勾

| 开关    | 什么时候开                                    |
| ----- | ---------------------------------------- |
| 工具调用  | 模型本身支持 Function Calling 才勾，不支持勾了也没效果     |
| 图片输入  | 必须是多模态视觉模型，纯文本模型勾了会报错                    |
| 思考模式  | 模型支持推理链路才勾（如 DeepSeek Reasoner 一类）       |
| 自定义协议 | 接口地址不是标准 `/chat/completions` 结尾时勾，跳过自动补全 |
| 上下文窗口 | 不清楚就留空，用默认值                              |

保存后模型会出现在选择器列表里，选中即可对话。

> 接入自定义模型**不消耗 WorkBuddy 积分**，产生的费用由你直接向中转服务方支付。

### 进阶：直接改配置文件

需要批量配置或脚本化管理时，可以编辑本地配置文件（路径以本机实际安装为准，通常在用户目录下的 `.workbuddy/models.json`）：

```json
{
  "models": [
    {
      "id": "claude-sonnet-4",
      "name": "Claude（中转）",
      "vendor": "Custom",
      "apiKey": "sk-你的密钥",
      "url": "https://<你的中转地址>/v1",
      "supportsToolCall": true,
      "supportsImages": false
    }
  ]
}
```

> 这个文件是**明文 JSON**，别提交到公开仓库，也别发给别人。不再使用时记得清理里面的 Key。

---

## 其他客户端速查

| 客户端                 | 配置位置                        | 接口地址填法              |
| ------------------- | --------------------------- | ------------------- |
| Cherry Studio       | 设置 → 模型服务 → 添加 OpenAI 兼容提供商 | `https://<中转地址>/v1` |
| NextChat / LobeChat | 设置 → 自定义接口                  | `https://<中转地址>/v1` |
| 沉浸式翻译               | 设置 → 翻译服务 → 自定义 / OpenAI 兼容 | `https://<中转地址>/v1` |
| Claude Code         | 环境变量 `ANTHROPIC_BASE_URL`   | 见上文快速配置             |

各客户端版本迭代较快，菜单名称可能有出入，以你实际界面为准。填完发一条测试消息验证连通性再开始用。

---

## 几点提醒

1. **先小额验证**。任何第三方中转都存在服务变动、限流降智的可能，先用最小额度跑一到两天再决定是否加量。
2. **不要把重要业务绑死在单一渠道**，保留备用方案。
3. **不要提交敏感内容**。公司代码、用户数据、内部凭证不要通过第三方中转提交。
4. **定期核实用量**，留意后台统计与实际消耗是否一致。

---

## Topics

`claude-code` `anthropic` `openai-api` `llm` `cherry-studio` `api-guide` `开发者工具`
