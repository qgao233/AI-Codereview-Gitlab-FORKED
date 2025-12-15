# 🔍 AI-Codereview-Gitlab 项目解构分析

## 一、项目概述

这是一个 **基于大模型的自动化代码审查工具**，主要功能是：
- 监听 GitLab/GitHub/Gitea 的 Webhook 事件（MR/PR/Push）
- 使用 AI 大模型自动审查代码变更
- 将审查结果回写到对应的 MR/PR/Commit 评论中
- 支持消息推送到钉钉/企业微信/飞书
- 提供可视化 Dashboard 展示审查统计

---

## 二、项目架构

```
AI-Codereview-Gitlab/
├── api.py              # 🚀 Flask API 服务入口（核心）
├── ui.py               # 📊 Streamlit Dashboard 入口
├── biz/                # 📦 业务逻辑层
│   ├── cmd/            # 命令行工具（整库审查）
│   ├── entity/         # 数据实体类
│   ├── event/          # 事件管理器
│   ├── gitea/          # Gitea Webhook 处理
│   ├── github/         # GitHub Webhook 处理
│   ├── gitlab/         # GitLab Webhook 处理
│   ├── llm/            # LLM 客户端（工厂模式）
│   │   ├── client/     # 各大模型客户端实现
│   │   └── factory.py  # 客户端工厂
│   ├── queue/          # 任务队列处理
│   ├── service/        # 服务层（数据库操作）
│   └── utils/          # 工具类
│       ├── code_reviewer.py  # 代码审查核心逻辑
│       ├── im/         # 消息推送（钉钉/飞书/企微）
│       └── ...
├── conf/               # 配置文件
│   ├── .env            # 环境变量配置
│   └── prompt_templates.yml  # AI 提示词模板
├── data/               # SQLite 数据库存储
└── docker-compose.yml  # Docker 部署配置
```

---

## 三、核心工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        工作流程图                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GitLab/GitHub/Gitea                                            │
│       │                                                         │
│       ▼ (Webhook: MR/PR/Push)                                   │
│  ┌─────────────┐                                                │
│  │   api.py    │  Flask 服务接收 Webhook                        │
│  └──────┬──────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────────────────┐                                    │
│  │ webhook_handler.py      │  解析事件，获取代码变更(diff)       │
│  └──────────┬──────────────┘                                    │
│             │                                                   │
│             ▼                                                   │
│  ┌─────────────────────────┐                                    │
│  │  code_reviewer.py       │  构建 Prompt，调用 LLM 审查        │
│  └──────────┬──────────────┘                                    │
│             │                                                   │
│             ▼                                                   │
│  ┌─────────────────────────┐                                    │
│  │  llm/factory.py         │  工厂模式，支持多种 LLM 提供商     │
│  │  (DeepSeek/OpenAI/...)  │                                    │
│  └──────────┬──────────────┘                                    │
│             │                                                   │
│             ▼                                                   │
│  ┌─────────────────────────┐                                    │
│  │  回写评论到 MR/PR       │  + 发送消息到钉钉/飞书              │
│  │  + 保存到 SQLite        │                                    │
│  └─────────────────────────┘                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 四、项目复杂度评估

| 维度 | 评分 | 说明 |
|------|------|------|
| **代码量** | ⭐⭐ 中低 | 约 2000-3000 行 Python 代码 |
| **架构复杂度** | ⭐⭐ 中低 | 分层清晰，模块化良好 |
| **技术栈** | ⭐⭐ 中等 | Flask + Streamlit + SQLite + 多 LLM API |
| **业务逻辑** | ⭐⭐ 中等 | 多平台适配（GitLab/GitHub/Gitea） |
| **部署难度** | ⭐ 低 | Docker 一键部署 |

**总体评价：中低复杂度项目**，适合中级开发者快速上手。

---

## 五、核心文件说明

| 文件 | 作用 | 重要程度 |
|------|------|----------|
| `api.py` | Flask API 入口，处理 Webhook | ⭐⭐⭐⭐⭐ |
| `biz/queue/worker.py` | 核心业务处理（MR/Push 事件） | ⭐⭐⭐⭐⭐ |
| `biz/utils/code_reviewer.py` | 代码审查逻辑，调用 LLM | ⭐⭐⭐⭐⭐ |
| `biz/llm/factory.py` | LLM 客户端工厂 | ⭐⭐⭐⭐ |
| `biz/gitlab/webhook_handler.py` | GitLab API 交互 | ⭐⭐⭐⭐ |
| `biz/service/review_service.py` | 数据库操作（SQLite） | ⭐⭐⭐ |
| `conf/prompt_templates.yml` | AI 提示词配置 | ⭐⭐⭐ |
| `ui.py` | Dashboard 界面 | ⭐⭐ |

---

## 六、技术栈详解

### 后端框架
- **Flask 3.0.3** - 轻量级 Web 框架，处理 Webhook 请求
- **APScheduler 3.10.4** - 定时任务调度（日报生成）

### 前端/Dashboard
- **Streamlit 1.42.2** - 快速构建数据可视化界面
- **Matplotlib 3.10.1** - 图表绑制

### 数据存储
- **SQLite** - 轻量级数据库，存储审查记录
- **Pandas** - 数据处理和分析

### LLM 集成
- **OpenAI SDK** - 支持 OpenAI/DeepSeek/通义千问
- **ZhipuAI SDK** - 智谱 AI 支持
- **Ollama** - 本地大模型支持

### 代码托管平台
- **python-gitlab** - GitLab API 交互
- **requests** - GitHub/Gitea API 交互

### 消息推送
- 钉钉 Webhook
- 企业微信 Webhook
- 飞书 Webhook

---

## 七、快速入门指南

### 1. 环境准备

```bash
# 克隆项目
git clone <你的仓库地址>
cd AI-Codereview-Gitlab-FORKED

# 创建虚拟环境（推荐 Python 3.10+）
python -m venv venv

# Windows 激活虚拟环境
.\venv\Scripts\activate

# Linux/Mac 激活虚拟环境
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

### 2. 配置环境变量

```bash
# 复制配置模板
# Windows
copy conf\.env.dist conf\.env

# Linux/Mac
cp conf/.env.dist conf/.env
```

编辑 `conf/.env`，配置以下关键项：

```bash
# LLM 提供商配置（支持 zhipuai/openai/deepseek/qwen/ollama）
LLM_PROVIDER=deepseek

# DeepSeek API Key
DEEPSEEK_API_KEY=your_deepseek_api_key

# 或者 OpenAI API Key
OPENAI_API_KEY=your_openai_api_key
OPENAI_API_BASE=https://api.openai.com/v1

# GitLab 配置
GITLAB_URL=https://gitlab.com
GITLAB_ACCESS_TOKEN=your_gitlab_token

# 支持审查的文件类型
SUPPORTED_EXTENSIONS=.java,.py,.php,.yml,.vue,.go,.c,.cpp,.h,.js,.css,.md,.sql

# 审查风格（professional/sarcastic/gentle/humorous）
REVIEW_STYLE=professional

# 钉钉推送（可选）
DINGTALK_ENABLED=0
DINGTALK_WEBHOOK_URL=https://oapi.dingtalk.com/robot/send?access_token=xxx

# Dashboard 登录账号
DASHBOARD_USER=admin
DASHBOARD_PASSWORD=your_password
```

### 3. 启动服务

**方式一：本地启动**

```bash
# 启动 API 服务（监听 Webhook）
python api.py

# 另开终端，启动 Dashboard
streamlit run ui.py --server.port=5002 --server.address=0.0.0.0
```

**方式二：Docker 启动**

```bash
docker-compose up -d
```

### 4. 验证服务

- API 服务：访问 `http://localhost:5001`，显示 "The code review api server is running."
- Dashboard：访问 `http://localhost:5002`，看到登录页面

### 5. 配置 GitLab Webhook

1. 进入 GitLab 项目 → Settings → Webhooks
2. 添加 Webhook：
   - URL: `http://your-server:5001/review/webhook`
   - Secret Token: 你的 GitLab Access Token（可选）
   - 触发事件: 勾选 `Push Events` 和 `Merge Request Events`
3. 点击 "Add webhook"

---

## 八、代码阅读路径（推荐顺序）

按以下顺序阅读代码，可以快速理解项目：

### 第一阶段：理解入口和流程
1. **`api.py`** - 理解 Flask 路由和 Webhook 入口
2. **`biz/queue/worker.py`** - 理解核心业务处理流程

### 第二阶段：理解核心逻辑
3. **`biz/utils/code_reviewer.py`** - 理解 LLM 调用和代码审查逻辑
4. **`biz/llm/factory.py`** + `biz/llm/client/` - 理解多 LLM 适配
5. **`conf/prompt_templates.yml`** - 理解提示词设计

### 第三阶段：理解平台适配
6. **`biz/gitlab/webhook_handler.py`** - 理解 GitLab API 交互
7. **`biz/github/webhook_handler.py`** - 理解 GitHub 适配
8. **`biz/gitea/webhook_handler.py`** - 理解 Gitea 适配

### 第四阶段：理解辅助功能
9. **`biz/service/review_service.py`** - 理解数据存储
10. **`biz/utils/im/`** - 理解消息推送
11. **`ui.py`** - 理解 Dashboard 实现

---

## 九、扩展开发指南

### 添加新的 LLM 提供商

1. 在 `biz/llm/client/` 下新建客户端文件，如 `claude.py`
2. 继承 `BaseClient` 并实现 `completions` 方法
3. 在 `biz/llm/factory.py` 中注册新客户端

```python
# biz/llm/client/claude.py
from biz.llm.client.base import BaseClient

class ClaudeClient(BaseClient):
    def __init__(self):
        self.api_key = os.getenv("CLAUDE_API_KEY")
        
    def completions(self, messages: list) -> str:
        # 实现 Claude API 调用
        pass
```

### 自定义审查规则

修改 `conf/prompt_templates.yml`，调整提示词：

```yaml
code_review_prompt:
  system_prompt: |-
    你是一位资深的软件开发工程师...
    # 在这里添加你的自定义规则
    
  user_prompt: |-
    # 修改用户提示词格式
```

### 添加新的消息推送渠道

1. 在 `biz/utils/im/` 下新建推送类
2. 继承基类并实现 `send` 方法
3. 在 `notifier.py` 中注册

### 支持其他代码托管平台

参考 `biz/gitea/` 目录结构：
1. 创建 `webhook_handler.py` 处理 Webhook 解析
2. 在 `api.py` 中添加路由处理
3. 在 `worker.py` 中添加事件处理函数

---

## 十、常见问题

### Q1: 如何查看审查日志？

访问 Dashboard（默认 `http://localhost:5002`），登录后可查看所有审查记录。

### Q2: 如何对整个代码库进行审查？

```bash
python -m biz.cmd.review
```

### Q3: 支持哪些文件类型？

通过环境变量 `SUPPORTED_EXTENSIONS` 配置，默认支持：
`.java,.py,.php,.yml,.vue,.go,.c,.cpp,.h,.js,.css,.md,.sql`

### Q4: 如何切换审查风格？

修改环境变量 `REVIEW_STYLE`，支持：
- `professional` - 专业严谨
- `sarcastic` - 讽刺吐槽
- `gentle` - 温和建议
- `humorous` - 幽默风趣

---

## 十一、项目亮点

1. **多平台支持** - 同时支持 GitLab、GitHub、Gitea
2. **多 LLM 支持** - 工厂模式，轻松切换 LLM 提供商
3. **可配置提示词** - YAML 配置 + Jinja2 模板，灵活定制
4. **可视化 Dashboard** - Streamlit 快速构建，开箱即用
5. **消息推送** - 支持钉钉、企业微信、飞书
6. **Docker 部署** - 一键部署，简单易用

---

*文档生成时间：2025年12月*

