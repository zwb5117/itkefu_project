# 多智能体电脑售后客服系统

基于 FastAPI + OpenAI Agents SDK 构建的 **多智能体电脑售后客服系统**，通过主调度智能体（Orchestrator）协调多个专业子智能体，为用户提供 IT 技术支持、服务站查询、地图导航、实时资讯等一站式智能服务。

## 项目架构

```
用户请求 → 主调度智能体（Orchestrator）
                ├── 技术专家智能体（Technical Agent）  → 技术问答 / 故障排查 / 实时资讯
                └── 全能业务智能体（Service Agent）    → 服务站查询 / POI 导航 / 地理位置服务
```

- **后端框架**：FastAPI + Uvicorn（异步 Web 服务，支持 SSE 流式响应）
- **AI 引擎**：OpenAI Agents SDK（多智能体编排 / 工具调用）
- **模型支持**：硅基流动（Qwen3-32B）/ 阿里百炼（Qwen3-Max）
- **外部工具**：MCP 协议集成（联网搜索 + 百度地图）
- **数据存储**：MySQL（会话管理 / 用户记忆）
- **知识库**：支持外部知识库服务对接（backend/knowledge，进行自主知识库构建）

## 项目结构

```
itkefu_project/
├── backend/app/
│   ├── api/                  # FastAPI 路由入口
│   │   ├── main.py           # 应用启动入口
│   │   └── routers.py        # API 路由定义（对话 / 会话查询）
│   ├── config/               # 配置管理（pydantic-settings）
│   ├── infrastructure/       # 基础设施层
│   │   ├── ai/               # AI 客户端 / Prompt 加载
│   │   ├── database/         # MySQL 连接池
│   │   ├── logging/          # 日志系统
│   │   └── tools/            # 工具层（本地工具 / MCP 工具）
│   ├── multi_agent/          # 多智能体定义
│   │   ├── orchestrator_agent.py   # 主调度智能体
│   │   ├── technical_agent.py      # 技术专家智能体
│   │   ├── service_agent.py        # 全能业务智能体
│   │   └── agent_factory.py        # 工具注册 & 路由
│   ├── prompts/              # 智能体 Prompt 模板
│   ├── repositories/         # 数据持久层
│   ├── schemas/              # 请求/响应模型
│   ├── services/             # 业务服务层
│   ├── tests/                # 测试用例
│   └── user_memories/        # 用户会话记忆（JSON）
└── README.md
```

## 运行指南

### 环境要求

- Python 3.11+
- MySQL 5.7+
- 至少一个 LLM API Key（硅基流动 或 阿里百炼）

### 1. 克隆项目

```bash
git clone <repo-url>
cd itkefu_project
```

### 2. 安装依赖

```bash
cd backend/app
pip install -r requirements.txt
```

### 3. 配置环境变量

编辑 `backend/app/.env` 文件，填入你的 API Key 和数据库信息：

```env
# LLM 配置（至少配置一个）
SF_API_KEY=你的硅基流动API_KEY
SF_BASE_URL=https://api.siliconflow.cn/v1
MAIN_MODEL_NAME=Qwen/Qwen3-32B

AL_BAILIAN_API_KEY=你的阿里百炼API_KEY
AL_BAILIAN_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
SUB_MODEL_NAME=qwen3-max

# MySQL 配置
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=你的数据库密码
MYSQL_DATABASE=its

# MCP 联网搜索
DASHSCOPE_BASE_URL=https://dashscope.aliyuncs.com/api/v1/mcps/WebSearch/sse

# 百度地图
BAIDUMAP_AK=你的百度地图AK

# 知识库服务（你需要backend/knowledge自己去构建）
KNOWLEDGE_BASE_URL=http://127.0.0.1:8001
```

### 4. 初始化数据库

确保 MySQL 已启动，并创建数据库：

```sql
CREATE DATABASE IF NOT EXISTS its DEFAULT CHARSET utf8mb4;
```

### 5. 启动服务

```bash
# 方式一：直接运行
cd backend/app
python api/main.py

# 方式二：使用 uvicorn
cd backend/app
uvicorn api.main:create_fast_api --host 127.0.0.1 --port 8000 --reload
```

服务启动后访问 `http://127.0.0.1:8000`。

### 6. 测试接口

```bash
# 对话接口（SSE 流式响应）
curl -X POST http://127.0.0.1:8000/api/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "我的电脑无法开机怎么办？",
    "context": {"user_id": "test_user"},
    "flag": true
  }'

# 查询用户会话历史
curl -X POST http://127.0.0.1:8000/api/user_sessions \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test_user"}'
```

## API 接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/query` | POST | 智能体对话（SSE 流式响应） |
| `/api/user_sessions` | POST | 获取用户历史会话记忆 |

## 功能示例

| 场景 | 示例输入 |
|------|----------|
| 技术问答 | "为什么 Windows 删除文件后在回收站找不到？" |
| 服务站查询 | "帮我找最近的维修站" |
| POI 导航 | "导航去天安门广场" |
| 实时资讯 | "今天 AI 圈发生了什么事？" |
| 多跳任务 | "查一下今天天气，如果下雨就帮我找一家最近的服务站" |
