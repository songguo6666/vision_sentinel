# Vision Sentinel

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Windows%2010%2F11-blue" alt="Platform">
  <img src="https://img.shields.io/badge/Python-3.12%2B-green" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="License">
</p>

## 📖 项目简介

**Vision Sentinel** 是一个为 [OpenClaw](https://github.com/openclaw/openclaw) 设计的本地视觉感知技能（Skill），专为 Windows 10/11 环境的自动化 fallback 感知而构建。

当主要的结构化通道（DOM、UIA、MSAA）无法解释当前 UI 状态时，Vision Sentinel 提供本地视觉辅助，确保自动化可靠性。

## 🎯 核心功能

### 感知通道

| 通道 | 描述 |
|------|------|
| **Window Meta** | 窗口元数据（标题、进程、位置） |
| **UIA** | UI Automation 可访问性树 |
| **MSAA** | Microsoft Active Accessibility |
| **OCR** | 光学字符识别（多后端支持） |
| **Icons** | 图标模板匹配 |
| **Diff** | 截图差异分析 |
| **Local VLM** | 本地视觉语言模型（可选） |
| **Cloud** | 云端推理（按需） |

### 触发场景

- `structured_failure` - 结构化通道无有效候选
- `new_window` - 新窗口或模态框出现
- `post_action_verify` - 操作后状态验证
- `high_risk_precheck` - 高风险操作预检
- `unknown_canvas` - 自绘或 Canvas 类界面
- `manual_probe` - 手动探查

## 🏗️ 架构

```
┌─────────────────────────────────────────────────────┐
│                   OpenClaw Agent                     │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                Vision Sentinel                        │
│                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────┐  │
│  │   Observe   │    │   Verify    │    │ Warmup  │  │
│  │             │    │   Action    │    │         │  │
│  └──────┬──────┘    └──────┬──────┘    └────┬────┘  │
│         │                  │                  │       │
│         ▼                  ▼                  ▼       │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Fallback Chain                     │ │
│  │                                                 │ │
│  │  Window Meta → UIA → MSAA → OCR → Icons       │ │
│  │            → Diff → Local VLM → Cloud         │ │
│  │                                                 │ │
│  └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

## 🚀 快速开始

### 安装依赖

```bash
# 创建虚拟环境
uv venv .venv
source .venv/bin/activate  # Linux/Mac
# 或 .venv\Scripts\activate  # Windows

# 安装项目
uv pip install -e .
```

### 配置

编辑 `skill.yaml` 自定义感知行为：

```yaml
perception:
  ocr:
    enabled: true
    backend: rapidocr  # rapidocr, easyocr, tesseract
  vlm:
    enabled: true
    model: llava       # llava, minimonkey
  capture:
    default_region: focused_window
```

## 📡 API

### `observe`

观察当前 UI 状态，用于未知或不确定的场景。

```json
// 输入
{
  "reason": "structured_failure",
  "target": "main_window",
  "roi": null,
  "budget": 5000,
  "context": {}
}

// 输出
{
  "status": "ok",
  "state_hash": "9f01869cb39b2be1",
  "elements": [...],
  "uncertainty": [],
  "cloud_payload": {...}
}
```

### `verify_action`

验证操作后的状态变化。

```json
// 输入
{
  "reason": "post_action_verify",
  "last_action": {"type": "click", "target": "save_button"},
  "pre_action_state_hash": "abc123",
  "verification_deadline_ms": 3000
}
```

### `warmup`

预热运行时，初始化缓存和模型。

### `shutdown`

释放资源，清理缓存。

## ⚙️ 配置选项

| 参数 | 默认值 | 描述 |
|------|--------|------|
| `cheap_path_budget_ms` | 1200 | 廉价路径超时 |
| `ocr_path_budget_ms` | 2500 | OCR 路径超时 |
| `vlm_path_budget_ms` | 5000 | 本地 VLM 超时 |
| `hard_timeout_ms` | 6000 | 硬超时 |
| `captures_per_request` | 2 | 每次请求截图数 |
| `cloud_escalation` | false | 默认禁用云端上报 |

## 🔒 隐私

- 所有证据默认本地存储
- 默认不上传完整桌面截图
- 优先使用结构化摘要和 ROI 引用
- 云端上报需要显式启用

## 📦 项目结构

```
vision_sentinel/
├── src/vision_sentinel/
│   ├── api.py              # 公共 API
│   ├── capture.py         # 截图模块
│   ├── config.py          # 配置管理
│   ├── runtime.py         # 运行时
│   ├── perception/        # 感知模块
│   │   ├── browser.py     # 浏览器感知
│   │   ├── ocr.py         # OCR
│   │   ├── icons.py       # 图标识别
│   │   ├── uia.py         # UIA 后端
│   │   └── msaa.py        # MSAA 后端
│   ├── windows/           # Windows 特定
│   └── vlm/              # VLM 适配器
├── tests/                 # 测试
└── SKILL.md              # 技能定义
```

## 🛠️ 技术栈

- **Python** 3.12+
- **RapidOCR** / EasyOCR / Tesseract - OCR 引擎
- **LLaVA** / MiniMonkey - 本地 VLM
- **pywin32** - Windows API
- **UIAutomationCore** - UI 自动化

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

<p align="center">Built with ❤️ for OpenClaw</p>