# WeChatAI

WeChatAI 是一个运行在 LSPosed 环境中的微信 AI 回复模块。

本仓库只提供已经编译、签名和混淆的 APK 发布包，不包含项目源代码。

## 功能

- 通过兼容 OpenAI Chat Completions API 的服务生成回复
- 支持私聊回复
- 支持群聊回复和群聊 `@` 回复
- 支持私聊、群聊白名单
- 支持关键词过滤
- 支持引用消息回复
- 支持会话上下文
- 支持图片消息分析（默认关闭）
- 支持图片大小和请求体大小限制
- 支持日志级别和日志文件滚动

## 环境要求

- Android 8.0（API 26）或更高版本
- 已安装并正常工作的 LSPosed
- 已安装微信

微信内部类名和实现可能随版本变化。当前版本的兼容性以实际测试结果为准，不保证兼容所有微信版本。

## 安装

1. 从 [Releases](https://github.com/wxmyyds/WeChatAI-Releases/releases) 下载最新 APK。
2. 安装 APK。
3. 打开 LSPosed Manager。
4. 在模块列表中启用 `WeChat AI Bot`。
5. 将微信（`com.tencent.mm`）加入作用域。
6. 强制停止并重新打开微信。
7. 按下面的说明编辑配置文件。

如果模块没有生效，请确认 LSPosed 已正常工作，并检查作用域是否选择了微信。

## 配置文件

模块配置文件位于：

```text
/data/data/com.tencent.mm/WeChatAI/config.jsonc
```

文件需要使用 root、ADB shell 或其他具有相应权限的文件管理方式访问。修改配置后，重启微信以确保配置生效。

配置文件是 JSONC 格式，支持使用 `//` 和 `/* ... */` 注释。

## 配置示例

下面是一个最小私聊配置示例。请将 `api_key` 替换为你自己的 API Key，不要把真实 Key 分享给其他人。

```jsonc
{
  "ai_enabled": true,
  "api_url": "https://api.openai.com/v1/chat/completions",
  "api_key": "YOUR_API_KEY",
  "model": "gpt-3.5-turbo",
  "system_prompt": "请简洁、准确地回答问题。",
  "model_context_chars": 32000,
  "reply_delay": 2000,
  "reply_friend_enabled": true,
  "reply_group_enabled": false,
  "reply_only_at": true,
  "reply_with_quote": false,
  "image_enabled": false,
  "image_max_bytes": 10485760,
  "image_max_request_bytes": 16777216,
  "image_use_mid": false,
  "friend_whitelist": "wxid_example",
  "group_whitelist": "",
  "filter_keywords": "",
  "log_level": "INFO",
  "request": {
    "messages": "{{messages}}",
    "model": "{{model}}",
    "temperature": 1,
    "max_completion_tokens": 2048,
    "top_p": 1,
    "stream": false,
    "reasoning_effort": "medium",
    "stop": null
  }
}
```

## 主要配置项

| 配置项 | 说明 |
| --- | --- |
| `ai_enabled` | 是否启用 AI 回复。启用时必须填写 `api_key`。 |
| `api_url` | OpenAI 兼容接口地址，只支持 `http` 或 `https`。 |
| `api_key` | API 密钥。请勿公开或提交到仓库。 |
| `model` | 使用的模型名称。 |
| `system_prompt` | 系统提示词，为空时不发送 system 消息。 |
| `model_context_chars` | 每个会话的上下文字符预算。`0` 表示不使用历史消息；有效非零范围为 `1000-4000000`。接近预算时会自动压缩历史。 |
| `reply_delay` | 回复延迟，单位为毫秒，范围为 `0-60000`。 |
| `max_response_bytes` | API 响应体最大字节数，范围为 `65536-16777216`。 |
| `reply_friend_enabled` | 是否回复私聊消息。 |
| `reply_group_enabled` | 是否回复群聊消息。 |
| `reply_only_at` | 群聊中是否只回复被 `@` 的消息。 |
| `reply_with_quote` | 是否使用微信引用消息功能。 |
| `image_enabled` | 是否允许将微信图片上传给视觉 AI，默认关闭。关闭时不会处理图片消息。 |
| `image_max_bytes` | 单张图片原文件最大字节数，范围为 `1-67108864`。超过后不上传。 |
| `image_max_request_bytes` | Base64 编码后的图片最大字节数，范围为 `1-134217728`。超过后不上传。 |
| `image_use_mid` | 原图未就绪时是否让微信通过官方接口单次下载中图，默认关闭。启用后仍使用正式图片路径读取，不使用缩略图。 |
| `friend_whitelist` | 私聊白名单，多个微信 ID 使用英文逗号分隔。 |
| `group_whitelist` | 群聊白名单，多个群 ID 使用英文逗号分隔。 |
| `filter_keywords` | 关键词过滤，多个关键词使用英文逗号分隔；为空表示不启用。 |
| `log_level` | 日志级别：`DEBUG`、`INFO`、`WARN`、`ERROR`、`NONE`。 |

## 图片消息

图片分析默认关闭。确认第三方 AI 服务允许上传图片，并了解其隐私政策后，再启用：

```jsonc
"image_enabled": true,
"image_max_bytes": 10485760,
"image_max_request_bytes": 16777216,
"image_use_mid": false
```

图片读取使用微信的正式图片存储路径和 VFS 接口，不使用缩略图作为回退。`image_use_mid` 只控制原图未就绪时是否执行一次官方中图下载；它不会启用轮询、重复下载或重复处理同一条消息。图片路径仍未就绪时，模块会回复“图片暂未下载完成”。

`image_enabled` 控制是否上传图片，`image_use_mid` 只控制是否允许这一次下载，两者互不替代。图片超过任一大小限制时，模块会回复“图片过大”。

## 白名单和关键词

建议先使用白名单进行小范围测试：

```jsonc
"friend_whitelist": "wxid_a,wxid_b",
"group_whitelist": "123456789@chatroom"
```

当 `filter_keywords` 不为空时，消息内容只要匹配其中任意一个关键词才会触发回复：

```jsonc
"filter_keywords": "帮助,问答,AI"
```

## 日志

日志文件位于：

```text
/data/data/com.tencent.mm/WeChatAI/WeChatAI.log
```

默认日志级别为 `INFO`。排查问题时可以临时改为：

```jsonc
"log_level": "DEBUG"
```

日志文件超过约 2 MB 后会自动滚动，并保留一个 `.1` 备份文件。日志会对微信 ID 和服务器 ID 做摘要处理，不应包含 API Key。

排查完成后建议恢复为 `INFO`，避免产生过多日志。

## 校验下载文件

每个正式 Release 同时提供 `SHA256SUMS.txt`。下载 APK 和校验文件后，在 Linux 或 macOS 上执行：

```bash
sha256sum -c SHA256SUMS.txt
```

Windows PowerShell 可以执行：

```powershell
Get-FileHash .\WeChatAIBot-v1.0.0.apk -Algorithm SHA256
```

将计算出的哈希值与 `SHA256SUMS.txt` 中的值进行比较。

## 常见问题

### 模块已启用但没有回复

请依次检查：

1. LSPosed 作用域是否包含微信。
2. `ai_enabled` 是否为 `true`。
3. `api_key` 是否有效。
4. `api_url` 是否可以访问。
5. 当前会话是否在对应白名单中。
6. `reply_friend_enabled` 或 `reply_group_enabled` 是否开启。
7. 修改配置后是否重启了微信。

### 群聊没有回复

请检查：

```jsonc
"reply_group_enabled": true
```

如果 `reply_only_at` 为 `true`，还需要在群聊中 `@` 机器人账号。

### 如何关闭模块

将配置中的 `ai_enabled` 改为 `false`，或在 LSPosed Manager 中禁用模块并重启微信。

### 如何停止发送 API 请求

将以下配置改为：

```jsonc
"ai_enabled": false
```

## 安全和免责声明

- AI 生成内容可能不准确，请自行甄别。
- 不要在系统提示词、聊天内容或配置中放入不必要的敏感信息。
- API Key 由用户自行保管，项目不会为用户保管或提供 API Key。
- 使用第三方 AI 服务时，消息内容可能会发送到对应服务商，请先了解其隐私政策。
- 请遵守当地法律法规、微信服务条款和第三方 AI 服务条款。
- 使用本模块造成的账号、数据或其他风险由使用者自行承担。

## 发布信息

- 当前正式版本：[v1.0.2](https://github.com/wxmyyds/WeChatAI-Releases/releases/tag/v1.0.2)
- APK：[WeChatAIBot-v1.0.2.apk](https://github.com/wxmyyds/WeChatAI-Releases/releases/download/v1.0.2/WeChatAIBot-v1.0.2.apk)
- 校验文件：[SHA256SUMS.txt](https://github.com/wxmyyds/WeChatAI-Releases/releases/download/v1.0.2/SHA256SUMS.txt)

本仓库只提供编译产物。源代码维护在私有仓库中。

## 第三方声明

```text
WeChatAI Third-Party Notices
============================

This file lists the principal third-party components used by WeChatAI.
The corresponding upstream license terms remain authoritative.

1. libxposed API 102.0.0
   License: Apache License 2.0
   Source: https://github.com/libxposed/api
   License: https://www.apache.org/licenses/LICENSE-2.0

2. OkHttp 4.12.0
   License: Apache License 2.0
   Source: https://github.com/square/okhttp
   License: https://www.apache.org/licenses/LICENSE-2.0

3. Gson 2.10.1
   License: Apache License 2.0
   Source: https://github.com/google/gson
   License: https://www.apache.org/licenses/LICENSE-2.0

4. DexKit 2.2.0
   License boundary: the upstream repository root is Apache License 2.0;
   the DexKit Core component carries LGPL-3.0 license information.
   Source: https://github.com/LuckyPray/DexKit
   Root license: https://github.com/LuckyPray/DexKit/blob/master/LICENSE
   Core license: https://github.com/LuckyPray/DexKit/tree/master/Core
   Published metadata: https://central.sonatype.com/artifact/org.luckypray/dexkit/2.2.0

5. JUnit 4.13.2 (test dependency)
   License: Eclipse Public License 1.0
   Source: https://github.com/junit-team/junit4
   License: https://www.eclipse.org/legal/epl-v10.html

Transitive dependencies may have additional copyright and license notices.
This notice is provided for attribution and does not replace the license
texts distributed by the respective upstream projects.
```
