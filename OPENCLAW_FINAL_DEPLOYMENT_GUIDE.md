# OpenClaw 完整部署教程 (最终版)

> 从零开始搭建个人 AI 助手 - 包含多模型、TuriX-CUA 桌面自动化、Google Workspace 集成
> 
> **适用版本**：OpenClaw 2026.3.2+
> **更新时间**：2026-03-04
> **作者**：Joel

---

## 📋 目录

1. [环境准备](#1-环境准备)
2. [OpenClaw 安装与基础配置](#2-openclaw-安装与基础配置)
3. [多模型配置](#3-多模型配置)
4. [Telegram 渠道配置](#4-telegram-渠道配置)
5. [技能安装](#5-技能安装)
6. [长期记忆配置](#6-长期记忆配置)
7. [TuriX-CUA Windows 桌面自动化](#7-turix-cua-windows-桌面自动化)
8. [GitHub 集成](#8-github-集成)
9. [Google Workspace (gog) 集成](#9-google-workspace-gog-集成)
10. [完整配置文件](#10-完整配置文件)
11. [常用命令速查](#11-常用命令速查)
12. [使用示例](#12-使用示例)
13. [故障排除](#13-故障排除)

---

## 1. 环境准备

### 1.1 系统要求

| 项目 | 要求 | 说明 |
|------|------|------|
| 操作系统 | Windows 10/11 (64位) | macOS/Linux 也支持 |
| Node.js | 18+ | JavaScript 运行时 |
| Python | 3.12 | 通过 Anaconda 安装 |
| 浏览器 | Chrome | 推荐用于浏览器控制 |
| Anaconda | 最新版 | Python 环境管理 |

### 1.2 安装步骤

**Node.js：**
- 官网下载：https://nodejs.org
- 验证：`node -v`

**Anaconda：**
- 官网下载：https://www.anaconda.com/download
- 验证：`conda --version`

---

## 2. OpenClaw 安装与基础配置

### 2.1 一键安装

```powershell
# Windows PowerShell
powershell -Command "iwr -useb https://openclaw.ai/install.ps1 | iex"
```

**验证安装：**
```powershell
openclaw --version
# 输出: 2026.3.2 或更高版本
```

### 2.2 初始化

```powershell
openclaw onboard
```
选择 `local` 模式

### 2.3 启动服务

```powershell
openclaw gateway start
openclaw status
```

### 2.4 基础配置

```powershell
# 开启完整工具权限
openclaw config set tools.profile "full"

# 设置心跳间隔（每2小时主动检查）
openclaw config set agents.defaults.heartbeat.every "2h"

# 允许附件上传（用于图片识别）
openclaw config set tools.sessions_spawn.attachments.enabled true
```

---

## 3. 多模型配置

### 3.1 API Key 申请地址

| 服务 | 申请地址 | 主要用途 |
|------|----------|----------|
| **MiniMax** | https://www.minimaxi.com | 默认对话、推理 |
| **阿里云百炼** | https://dashscope.console.aliyun.com | 多模型、支持视觉 |
| **TuriX** | https://turixapi.io | Windows 桌面自动化 |
| **Tavily** | https://tavily.com | 网页搜索 |
| **GitHub** | https://github.com/settings/tokens | 代码管理 |
| **Google Cloud** | https://console.cloud.google.com | Gmail/日历等 |

### 3.2 模型配置

编辑 `C:\Users\你的用户名\.openclaw\openclaw.json`

在 `models.providers` 中添加：

```json
{
  "providers": {
    "minimax-cn": {
      "baseUrl": "https://api.minimaxi.com/anthropic",
      "apiKey": "你的MiniMax_API_Key",
      "api": "anthropic-messages",
      "models": [
        {
          "id": "MiniMax-M2.5",
          "name": "MiniMax M2.5",
          "reasoning": true,
          "contextWindow": 200000,
          "maxTokens": 8192
        }
      ]
    },
    "bailian": {
      "baseUrl": "https://coding.dashscope.aliyuncs.com/v1",
      "apiKey": "你的阿里云_API_Key",
      "api": "openai-completions",
      "models": [
        {
          "id": "qwen3.5-plus",
          "name": "qwen3.5-plus",
          "vision": true,
          "input": ["text", "image"],
          "contextWindow": 1000000,
          "maxTokens": 65536
        },
        {
          "id": "qwen3-coder-plus",
          "name": "qwen3-coder-plus",
          "contextWindow": 1000000,
          "maxTokens": 65536
        },
        {
          "id": "qwen3-max-2026-01-23",
          "name": "qwen3-max-2026-01-23",
          "contextWindow": 262144,
          "maxTokens": 65536
        },
        {
          "id": "kimi-k2.5",
          "name": "kimi-k2.5",
          "vision": true,
          "contextWindow": 262144,
          "maxTokens": 32768
        }
      ]
    }
  }
}
```

### 3.3 设置默认模型

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.5"
      }
    }
  }
}
```

### 3.4 环境变量配置

```powershell
openclaw config set env.TAVILY_API_KEY "你的TavilyKey"
```

### 3.5 重启服务

```powershell
openclaw gateway restart
```

---

## 4. Telegram 渠道配置

### 4.1 创建 Bot

1. 打开 Telegram
2. 搜索 **@BotFather**
3. 发送 `/newbot`
4. 设置名称和用户名
5. 保存 **Bot Token**

### 4.2 配置

```powershell
openclaw config set channels.telegram.botToken "你的BotToken"
openclaw config set channels.telegram.dmPolicy "pairing"
openclaw config set channels.telegram.groupPolicy "allowlist"
openclaw gateway restart
```

---

## 5. 技能安装

### 5.1 使用 ClawHub

```powershell
# 搜索技能
clawhub search <关键词>

# 安装技能
clawhub install <技能名>
```

### 5.2 已安装技能 (15个)

| # | 技能名称 | 功能 | 用途 |
|---|----------|------|------|
| 1 | tavily-search | 网页搜索 | `搜索 xxx` |
| 2 | weather | 天气查询 | `北京天气` |
| 3 | video-frames | 视频处理 | 提取视频帧 |
| 4 | healthcheck | 安全检查 | `健康检查` |
| 5 | coding | 代码辅助 | 写代码 |
| 6 | adaptive-reasoning | 自适应推理 | 复杂推理 |
| 7 | deep-thinking | 深度思考 | 深入分析 |
| 8 | proactive-agent | 主动代理 | 主动执行 |
| 9 | multi-agent-collaboration | 多代理协作 | 团队协作 |
| 10 | agent-team-orchestration | 代理编排 | 任务编排 |
| 11 | tg-voice-whisper | 语音识别 | Telegram 语音转文字 |
| 12 | obsidian-task | Obsidian 任务 | (macOS/Linux) |
| 13 | openclaw-github-assistant | GitHub 管理 | 查看仓库、Issue |
| 14 | turix-windows | Windows 桌面自动化 | 控制桌面应用 |
| 15 | gog | Google Workspace | Gmail/日历/Drive |

---

## 6. 长期记忆配置

### 6.1 创建 memory 目录

```powershell
mkdir C:\Users\你的用户名\openclaw\workspace\memory
```

### 6.2 创建 MEMORY.md

在 `workspace` 目录下创建 `MEMORY.md` 文件，记录重要信息。

### 6.3 确认启用

```powershell
openclaw config get hooks.internal.entries.session-memory
# 输出应为: {"enabled": true}
```

---

## 7. TuriX-CUA Windows 桌面自动化

### 7.1 安装步骤

**步骤 1：克隆代码**
```powershell
git clone --branch multi-agent-windows --depth 1 https://github.com/TurixAI/TuriX-CUA.git C:\Users\你的用户名\openclaw\workspace\turix-cua-windows
```

**步骤 2：创建 Python 环境**
```powershell
conda create -n turix_env python=3.12 -y
conda activate turix_env
```

**步骤 3：安装依赖**
```powershell
cd C:\Users\你的用户名\openclaw\workspace\turix-cua-windows
pip install -r requirements.txt
pip install pywin32>=306 psutil>=5.9.0 oss2
```

### 7.2 配置

编辑 `examples\config.json`：

```json
{
  "logging_level": "INFO",
  "output_dir": ".turix_tmp",
  "cleanup_previous_runs": true,

  "brain_llm": {
    "provider": "turix",
    "api_key": "你的TuriX_API_Key",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },

  "actor_llm": {
    "provider": "turix",
    "api_key": "你的TuriX_API_Key",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },

  "memory_llm": {
    "provider": "turix",
    "api_key": "你的TuriX_API_Key",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },

  "planner_llm": {
    "provider": "turix",
    "api_key": "你的TuriX_API_Key",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },

  "agent": {
    "task": "打开微信，给老婆发送消息：test",
    "memory_budget": 2000,
    "summary_memory_budget": 8000,
    "use_search": false,
    "use_skills": false,
    "max_actions_per_step": 5,
    "max_steps": 20,
    "force_stop_hotkey": "ctrl+shift+2"
  }
}
```

### 7.3 运行

```powershell
conda activate turix_env
cd C:\Users\你的用户名\openclaw\workspace\turix-cua-windows
python examples/main.py
```

### 7.4 任务示例

| 任务 | task 配置 |
|------|----------|
| 打开记事本 | `"打开记事本"` |
| 打开微信 | `"打开微信"` |
| 发微信消息 | `"打开微信，找到微信号『XXX』，发送消息『你好』"` |

### 7.5 注意事项

1. **管理员权限**：首次运行需要管理员权限
2. **停止快捷键**：`Ctrl+Shift+2` 强制停止

---

## 8. GitHub 集成

### 8.1 申请 Token

1. 打开 https://github.com/settings/tokens
2. 生成 new token (classic)
3. 勾选 `repo` 权限
4. 复制 Token

### 8.2 配置

```powershell
openclaw config set env.GITHUB_TOKEN "ghp_你的Token"
openclaw config set env.GITHUB_USERNAME "你的GitHub用户名"
openclaw gateway restart
```

### 8.3 功能

- 列出仓库
- 查看 CI 状态
- 创建 Issue
- 搜索代码

---

## 9. Google Workspace (gog) 集成

### 9.1 安装 gog CLI

**下载：**
```powershell
curl.exe -L -o gog.zip https://github.com/steipete/gogcli/releases/download/v0.11.0/gogcli_0.11.0_windows_amd64.zip
```

**解压：**
```powershell
Expand-Archive gog.zip -DestinationPath gog -Force
```

### 9.2 创建 Google OAuth 凭据

1. 打开 https://console.cloud.google.com/apis/credentials
2. 创建 OAuth 客户端 ID（桌面应用）
3. 下载 JSON 文件
4. 重命名为 `client_secret.json`

### 9.3 配置 client_secret.json

编辑 `client_secret.json`，添加回调地址：

```json
{
  "installed": {
    "client_id": "你的client_id",
    "client_secret": "你的client_secret",
    "redirect_uris": [
      "http://localhost",
      "http://localhost/oauth2/callback",
      "http://127.0.0.1",
      "http://127.0.0.1/oauth2/callback"
    ]
  }
}
```

### 9.4 授权

```powershell
# 设置凭据
gog.exe auth credentials client_secret.json

# 添加账户（会打开浏览器授权）
gog.exe auth add 你的邮箱@gmail.com --services gmail,calendar,drive,contacts,sheets,docs --force-consent
```

### 9.5 常用命令

| 命令 | 功能 |
|------|------|
| `gog.exe gmail search 'newer_than:7d'` | 搜索邮件 |
| `gog.exe gmail send --to xxx --subject "Hi" --body "Hello"` | 发送邮件 |
| `gog.exe calendar events` | 查看日历 |
| `gog.exe drive search "query"` | 搜索 Drive |
| `gog.exe contacts list` | 查看联系人 |

---

## 10. 完整配置文件

### 10.1 openclaw.json

```json
{
  "meta": {
    "lastTouchedVersion": "2026.3.2"
  },
  "env": {
    "TAVILY_API_KEY": "你的TavilyKey",
    "GITHUB_TOKEN": "你的GitHubToken",
    "GITHUB_USERNAME": "你的GitHub用户名"
  },
  "browser": {
    "defaultProfile": "chrome"
  },
  "auth": {
    "profiles": {
      "minimax-cn:default": {
        "provider": "minimax-cn",
        "mode": "api_key"
      },
      "qwen:default": {
        "provider": "qwen",
        "mode": "api_key"
      }
    }
  },
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "你的MiniMaxKey",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "MiniMax-M2.5",
            "name": "MiniMax M2.5",
            "reasoning": true,
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      },
      "bailian": {
        "baseUrl": "https://coding.dashscope.aliyuncs.com/v1",
        "apiKey": "你的阿里云Key",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3.5-plus",
            "name": "qwen3.5-plus",
            "vision": true,
            "input": ["text", "image"],
            "contextWindow": 1000000,
            "maxTokens": 65536
          },
          {
            "id": "qwen3-coder-plus",
            "name": "qwen3-coder-plus",
            "contextWindow": 1000000,
            "maxTokens": 65536
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.5"
      },
      "models": {
        "minimax-cn/MiniMax-M2.5": {"alias": "Minimax"},
        "bailian/qwen3.5-plus": {}
      },
      "workspace": "C:\\Users\\你的用户名\\.openclaw\\workspace",
      "heartbeat": {"every": "2h"}
    }
  },
  "tools": {
    "profile": "full",
    "sessions_spawn": {
      "attachments": {"enabled": true}
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "你的BotToken",
      "groupPolicy": "allowlist"
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback"
  }
}
```

### 10.2 TuriX config.json

```json
{
  "logging_level": "INFO",
  "brain_llm": {
    "provider": "turix",
    "api_key": "你的TuriXKey",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },
  "actor_llm": {
    "provider": "turix",
    "api_key": "你的TuriXKey",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },
  "memory_llm": {
    "provider": "turix",
    "api_key": "你的TuriXKey",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },
  "planner_llm": {
    "provider": "turix",
    "api_key": "你的TuriXKey",
    "base_url": "https://turixapi.io/v1",
    "model_name": "turix-large-windows"
  },
  "agent": {
    "task": "打开微信，给老婆发送消息：test",
    "max_steps": 20
  }
}
```

---

## 11. 常用命令速查

### OpenClaw
```powershell
openclaw status                  # 查看状态
openclaw gateway start          # 启动
openclaw gateway restart        # 重启
openclaw config set <key> <value>  # 设置配置
```

### ClawHub
```powershell
clawhub search <关键词>   # 搜索技能
clawhub install <技能名>  # 安装技能
```

### TuriX
```powershell
conda activate turix_env
cd turix-cua-windows
python examples/main.py
```

### gog
```powershell
gog.exe gmail search 'query'
gog.exe gmail send --to x@x.com --subject "Hi" --body "Hello"
gog.exe calendar events
```

---

## 12. 使用示例

### 12.1 对话使用

| 场景 | 模型 | 说明 |
|------|------|------|
| 日常对话 | MiniMax-M2.5 | 默认 |
| 图片识别 | qwen3.5-plus | 发送图片 |
| 代码任务 | qwen3-coder-plus | 写代码 |

### 12.2 技能使用

```
用户: 搜索 OpenClaw 教程
用户: 北京天气怎么样
用户: 列出我的 GitHub 仓库
```

### 12.3 自动化任务

**TuriX (桌面自动化)：**
```powershell
# 编辑 config.json 中的 task，然后运行
python examples/main.py
```

**gog (Google Workspace)：**
```powershell
# 发送邮件
gog.exe gmail send --to 513994881@qq.com --subject "我爱你" --body "我爱你"
```

---

## 13. 故障排除

### 13.1 模型调用失败
- 检查 API Key 是否正确
- 检查余额

### 13.2 Telegram 无法连接
- 检查 Bot Token
- 检查网络

### 13.3 TuriX 无法打开应用
- 以管理员身份运行
- 检查 Windows 权限

### 13.4 gog 授权失败
- 确认 client_secret.json 中的 redirect_uris 正确
- 在 Google Cloud 添加 http://127.0.0.1/oauth2/callback

---

## ✅ 快速部署清单

- [ ] 安装 Node.js
- [ ] 安装 Anaconda
- [ ] 安装 OpenClaw
- [ ] 申请 MiniMax API Key
- [ ] 申请阿里云百炼 API Key
- [ ] 申请 TuriX API Key
- [ ] 申请 Tavily API Key
- [ ] 申请 GitHub Token
- [ ] 创建 Google OAuth 凭据
- [ ] 配置 openclaw.json
- [ ] 安装技能 (15个)
- [ ] 配置 Telegram Bot
- [ ] 配置长期记忆
- [ ] 安装 TuriX-CUA
- [ ] 安装 gog CLI
- [ ] 授权 Google Workspace

---

## 📞 支持与资源

| 资源 | 链接 |
|------|------|
| OpenClaw 官网 | https://openclaw.ai |
| OpenClaw 文档 | https://docs.openclaw.ai |
| Discord 社区 | https://discord.com/invite/clawd |
| GitHub | https://github.com/openclaw/openclaw |
| TuriX-CUA | https://github.com/TurixAI/TuriX-CUA |
| ClawHub | https://clawhub.com |

---

## 📝 当前配置摘要

| 项目 | 状态 |
|------|------|
| 默认模型 | MiniMax-M2.5 |
| 可用模型 | 8+ 个 (MiniMax, 阿里云百炼) |
| Telegram | ✅ 已配置 |
| 技能数 | 15 个 |
| 长期记忆 | ✅ 已启用 |
| TuriX-CUA Windows | ✅ 已配置 |
| GitHub | ✅ 已配置 |
| Google Workspace | ✅ 已配置 (已测试发送邮件) |

---

*文档版本：最终版*
*更新时间：2026-03-04*
*作者：Joel*
