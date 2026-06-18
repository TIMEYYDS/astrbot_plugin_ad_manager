# AstrBot 广告管理插件 (Ad Manager)

[![AstrBot](https://img.shields.io/badge/AstrBot-Plugin-blue)](https://docs.astrbot.app/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

一个基于 [AstrBot](https://github.com/Soulter/AstrBot) 的 Telegram 群组广告自动管理插件。  
使用 AI 识别广告消息，并根据配置自动执行警告、删除、禁言或踢出拉黑等操作，帮助您高效维护群组秩序。

---

## ✨ 功能特点

- **智能识别**：支持通过 AI 模型（如 Qwen、ChatGPT 等）分析消息内容，判断是否为广告。
- **灵活处置**：可组合执行多种动作：
  - `warn` – 发送警告（@ 用户）
  - `delete` – 删除违规消息
  - `mute` – 禁言用户（可设定时长）
  - `kick` – 踢出用户并自动加入黑名单
- **黑白名单**：
  - **白名单**：用户不受检测（如管理员、信任成员）
  - **黑名单**：被踢出的用户自动加入，后续消息直接跳过检测
- **配置多样**：支持 WebUI 可视化配置和群内命令动态调整，修改即生效。
- **管理员控制**：管理命令（`/ad_config`、`/ad_whitelist`、`/ad_blacklist`）仅对 AstrBot 全局管理员（`admins_id`）开放。
- **轻量降级**：当 AI 服务不可用时，自动降级为简单规则匹配（联系方式、链接等）。
- **仅限 Telegram**：专门针对 Telegram 适配器设计，不影响其他平台。

---

## 📦 安装

### 方法一：通过 Git 克隆（推荐）
```bash
cd /path/to/AstrBot/data/plugins
git clone https://github.com/TIMEYYDS/astrbot_plugin_ad_manager.git
```

### 方法二：下载 ZIP 压缩包

1. 从 Releases 下载最新版本。
2. 解压到 `data/plugins/astrbot_plugin_ad_manager` 目录。

### 依赖安装

```bash
pip install httpx
```

在 AstrBot WebUI 中启用插件即可。

---

## ⚙️ 配置

### WebUI 配置项（`_conf_schema.json`）

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `enabled_groups` | 启用的群组 ID 列表（留空则对所有群生效） | `[]` |
| `ai_provider_id` | AI 提供商 ID（如 `sg/Qwen/Qwen3.5-35B-A3B`），留空则使用简单规则 | `""` |
| `bot_token` | **必填**：你的 Telegram Bot Token | `""` |
| `sensitivity` | 广告判定敏感度（0.1~0.95），越高越严格 | `0.6` |
| `action` | 处理动作，可选组合：`warn`, `delete`, `delete_warn`, `mute`, `mute_delete`, `kick`, `kick_delete`, `kick_warn`, `kick_delete_warn` 等 | `"warn"` |
| `mute_duration` | 禁言时长（秒），仅当动作包含 `mute` 时生效 | `300` |
| `warn_message` | 警告消息内容（支持 @ 用户） | `"⚠️ 检测到您的消息可能包含广告内容。"` |
| `exclude_keywords` | 包含这些关键词的消息跳过检测（如官方公告） | `["官方", "公告", "管理员"]` |
| `exclude_users` | 白名单用户 ID 列表（不受检测） | `[]` |
| `blacklist_users` | 黑名单用户 ID 列表（不受检测，由踢出自动添加或手动管理） | `[]` |

### 管理员配置

插件管理命令（`/ad_config`、`/ad_whitelist`、`/ad_blacklist`）仅允许 AstrBot 全局管理员执行。请将您的 Telegram 用户 ID 添加到 `data/cmd_config.json` 的 `admins_id` 列表中：

```json
{
    "admins_id": ["7520986490", "6627073553"]
}
```

或使用 AstrBot 命令 `/op <用户ID>` 添加管理员。

---

## 🚀 使用示例

### 场景 1：仅警告 + 删除消息
- 设置 `action = "delete_warn"`
- **效果**：检测到广告时，先 @ 用户发送警告，再删除违规消息。

### 场景 2：禁言 + 删除
- 设置 `action = "mute_delete"`，`mute_duration = 600`
- **效果**：检测到广告时，删除消息并将用户禁言 10 分钟。

### 场景 3：踢出并拉黑（严厉）
- 设置 `action = "kick_delete_warn"`
- **效果**：检测到广告时，发送警告、删除消息、踢出用户并加入黑名单（后续该用户的所有消息都会被忽略）。

### 群内管理命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/ad_config show` | 查看当前全部配置 | `/ad_config show` |
| `/ad_config set <key> <value>` | 修改配置项（立即生效并持久化） | `/ad_config set action kick_delete_warn` |
| `/ad_whitelist add <user_id>` | 将用户加入白名单 | `/ad_whitelist add 123456789` |
| `/ad_whitelist remove <user_id>` | 从白名单移除用户 | `/ad_whitelist remove 123456789` |
| `/ad_whitelist list` | 列出白名单用户 | `/ad_whitelist list` |
| `/ad_blacklist add <user_id>` | 手动将用户加入黑名单 | `/ad_blacklist add 123456789` |
| `/ad_blacklist remove <user_id>` | 从黑名单移除用户 | `/ad_blacklist remove 123456789` |
| `/ad_blacklist list` | 列出黑名单用户 | `/ad_blacklist list` |

> **注意**：`user_id` 为 Telegram 数字 ID，可从机器人消息日志中获取。

---

## 🔧 开发与扩展

### 修改 AI 检测提示词
您可以在 `main.py` 的 `_is_advertisement` 方法中调整 prompt 内容，以适应不同场景或更精准的判断。

### 自定义动作
当前支持的动作组合已在 `_handle_ad` 中实现，您可以根据需要添加新的动作（例如记录日志、统计次数等）。

---

## 🤝 贡献
欢迎提交 Issue 和 Pull Request。  
请确保代码符合 AstrBot 插件开发规范。

---

## 📄 许可证
MIT © TIMEYYDS

---

## 📞 支持
- **官方文档**：[AstrBot Docs](https://docs.astrbot.app/)
- **问题反馈**：请在本仓库提交 [Issue](https://github.com/TIMEYYDS/astrbot_plugin_ad_manager/issues)
- **仓库地址**：https://github.com/TIMEYYDS/astrbot_plugin_ad_manager

---

**Enjoy! 🎉
欢迎提交 Issue 和 Pull Request。
请确保代码符合 AstrBot 插件开发规范。

---

📄 许可证

MIT © TIMEYYDS

---

📞 支持

· 官方文档：AstrBot Docs
· 问题反馈：请在本仓库提交 Issue
· 仓库地址：https://github.com/TIMEYYDS/astrbot_plugin_ad_manager

---

Enjoy! 🎉

```
