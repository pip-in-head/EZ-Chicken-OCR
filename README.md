# EZ Chicken + OCR 🐔

M-SEC（多账户自动签到脚本，基于 [crazy0x70/EZ-Checkin](https://github.com/crazy0x70/EZ-Checkin) 改造，核心改动：**将远程云码验证码识别替换为纯本地 OCR（ddddocr）**，无需付费 API 即可自动识别验证码登录。

## ✨ 与原版的区别

| 项目 | 原版 EZ-Checkin | 本版 EZ Chicken + OCR |
|------|----------------|----------------------|
| 验证码识别 | 云码远程 API（需付费 token） | **本地 ddddocr**（完全免费离线） |
| 识别策略 | 单次远程请求 | **5种预处理 × 2引擎 = 多策略投票** |
| 运行模式 | 内置 APScheduler 长期驻留 | 运行一次即退出，配合 crontab |
| CAPTCHA_TOKEN | 必填 | **可选项**（不填=纯本地模式） |
| 网络依赖 | 必须联网 | **仅签到需联网，OCR 完全离线** |

## 🚀 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

> 首次运行时 ddddocr 会自动下载模型文件（约 10MB），请确保网络畅通。

### 2. 配置账户

复制示例配置文件并修改：

```bash
cp config_example.json config.json
```

编辑 `config.json`：

```json
{
  "user1": {
    "username": "你的用户名",
    "password": "你的密码",
    "Authorization": ""
  },
  "user2": {
    "username": "另一个用户名",
    "password": "另一个密码",
    "Authorization": ""
  },
  "LARK_WEBHOOK": "飞书通知webhook（可选）"
}
```

> **💡 `CAPTCHA_TOKEN` 现在是可选项**。不配置则使用纯本地 OCR；配置后本地 OCR 失败时自动回退到云码。

### 3. 运行

```bash
python main.py
```

脚本会执行一次签到然后退出。

### 4. 设置每日自动运行

**Linux / macOS（crontab）：**

```bash
# 每天 08:00 自动签到
0 8 * * * cd /path/to/EZ-Chicken-OCR && python3 main.py >> checkin.log 2>&1
```

**Windows（任务计划程序）：**

创建一个每天 08:00 触发的基本任务，操作为 `python main.py`，起始目录设为脚本所在文件夹。

## 🧠 验证码识别原理

### 本地 OCR 流程

```
验证码图片（160×80 PNG）
    │
    ├── 灰度+对比度增强 ──→ 普通引擎 + Beta引擎
    ├── 二值化（阈值80）  ──→ 普通引擎 + Beta引擎
    ├── 反色+对比度增强  ──→ 普通引擎 + Beta引擎
    ├── 锐化              ──→ 普通引擎 + Beta引擎
    └── 灰度原图          ──→ 普通引擎 + Beta引擎
    │
    └── 投票：出现次数最多的5位结果
         │
         ├── 置信度 ≥ 40% 且票数 ≥ 3 → ✅ 采纳
         └── 置信度 < 40% 或票数 < 3 → ❌ 自动重试
```

5 种预处理策略 × 2 个 OCR 引擎（普通 + Beta）= 最多 10 个识别结果参与投票，确保识别准确率。

## 📁 命令行参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--config-file` | 配置文件路径 | `./config.json` |
| `--lark` | 飞书/Lark Webhook | - |
| `--feishu` | 飞书 Webhook | - |
| `--tz` | 时区 | `Asia/Shanghai` |

## 📜 致谢

本项目基于 [crazy0x70/EZ-Checkin](https://github.com/crazy0x70/EZ-Checkin)（MIT License）改造而来。

原作者 [crazy0x70](https://github.com/crazy0x70) 实现了完整的多账户签到、飞书通知和智能 Token 管理机制，本项目在此基础上将验证码识别从远程云码 API 替换为纯本地 OCR（ddddocr）方案，并且移除了内置的 APScheduler 定时任务，改为运行一次即退出的模式，方便用户自行配合系统定时任务使用。

## 📄 License

MIT License — 与原项目保持一致。
