Deployment Guide
本文档面向希望在生产环境或长期运行场景中部署 Vitraya 的用户。如果你只是想快速体验，请直接参考 README 中的 Quick Start 章节。

架构概览
Vitraya 的推荐部署架构如下：

text
┌─────────────────────────────────────────────────────────────────┐
│                     外部流量 / Webhook                          │
│                  （Shopify、Stripe、...）                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │ Tailscale Funnel / ngrok
┌───────────────────────────▼─────────────────────────────────────┐
│                    Tailscale（公网暴露层）                       │
│              将本地 n8n Webhook 暴露为公网 Endpoint              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                        n8n（执行引擎）                           │
│     - 工作流编排、API 调用、状态机、异常回滚                     │
│     - 通过 MCP Server 封装 Connectors 暴露给 AI Agent           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
┌───────────▼───────────┐ ┌─▼─────────────┐ ┌─▼─────────────────┐
│     PostgreSQL         │ │    Redis      │ │   MCP Server      │
│    （状态持久化）       │ │  （缓存/队列） │ │ （AI 调用入口）    │
└───────────────────────┘ └───────────────┘ └───────────────────┘
前置条件
Docker 与 Docker Compose（推荐版本 ≥ 2.0）

Git（用于克隆仓库）

Tailscale（可选，用于公网 Webhook 暴露）

至少 2GB 可用内存（生产环境建议 4GB+）

环境变量
在部署前，请创建 .env 文件并配置以下变量：

变量名	默认值	说明
N8N_PORT	5678	n8n Web 界面与 API 端口
N8N_PROTOCOL	http	生产环境应设置为 https
N8N_ENCRYPTION_KEY	（必填）	用于加密凭据，生产环境务必设置，建议使用 openssl rand -hex 32 生成
N8N_BASIC_AUTH_USER	（可选）	启用基本身份验证的用户名
N8N_BASIC_AUTH_PASSWORD	（可选）	启用基本身份验证的密码
N8N_METRICS	false	是否开启 Prometheus 指标暴露
DB_POSTGRESDB_HOST	postgres	PostgreSQL 数据库主机
DB_POSTGRESDB_DATABASE	n8n	数据库名称
DB_POSTGRESDB_USER	n8n	数据库用户名
DB_POSTGRESDB_PASSWORD	（必填）	数据库密码
DB_POSTGRESDB_PORT	5432	数据库端口
REDIS_HOST	redis	Redis 主机
REDIS_PORT	6379	Redis 端口
REDIS_PASSWORD	（可选）	Redis 密码
GENERIC_TIMEZONE	UTC	系统时区，建议设置为 Asia/Shanghai
安全提示：对于敏感信息（如密码、密钥），可以使用 _FILE 后缀从文件中读取，例如 DB_POSTGRESDB_PASSWORD_FILE=/run/secrets/db_password，避免在环境变量中明文暴露。

Docker Compose 完整配置
以下是一个完整的 docker-compose.yml，包含核心执行环境（n8n + PostgreSQL + Redis）及可选的 AI 编排层（OpenClaw / LangGraph）：

yaml
version: '3.8'

services:
  ## ============================================================
  ## 核心执行环境（必须）
  ## ============================================================

  postgres:
    image: postgres:16
    restart: unless-stopped
    container_name: vitraya-postgres
    environment:
      - POSTGRES_USER=${DB_POSTGRESDB_USER:-n8n}
      - POSTGRES_PASSWORD=${DB_POSTGRESDB_PASSWORD}
      - POSTGRES_DB=${DB_POSTGRESDB_DATABASE:-n8n}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_POSTGRESDB_USER:-n8n}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    container_name: vitraya-redis
    command: redis-server --requirepass ${REDIS_PASSWORD:-}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    container_name: vitraya-n8n
    environment:
      - N8N_PORT=${N8N_PORT:-5678}
      - N8N_PROTOCOL=${N8N_PROTOCOL:-http}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER:-}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD:-}
      - N8N_METRICS=${N8N_METRICS:-false}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=${DB_POSTGRESDB_HOST:-postgres}
      - DB_POSTGRESDB_DATABASE=${DB_POSTGRESDB_DATABASE:-n8n}
      - DB_POSTGRESDB_USER=${DB_POSTGRESDB_USER:-n8n}
      - DB_POSTGRESDB_PASSWORD=${DB_POSTGRESDB_PASSWORD}
      - DB_POSTGRESDB_PORT=${DB_POSTGRESDB_PORT:-5432}
      - REDIS_HOST=${REDIS_HOST:-redis}
      - REDIS_PORT=${REDIS_PORT:-6379}
      - REDIS_PASSWORD=${REDIS_PASSWORD:-}
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE:-UTC}
      - WEBHOOK_URL=${WEBHOOK_URL:-}
    ports:
      - "${N8N_PORT:-5678}:5678"
    volumes:
      - n8n_data:/home/node/.n8n
      - ../connectors:/home/node/.n8n/connectors
      - ../patterns:/home/node/.n8n/patterns
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${N8N_PORT:-5678}/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3

  ## ============================================================
  ## AI 编排层（可选）
  ## ============================================================

  openclaw:
    image: openclaw/openclaw:latest
    restart: unless-stopped
    container_name: vitraya-openclaw
    profiles:
      - ai
    ports:
      - "${OPENCLAW_PORT:-8080}:8080"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - OPENCLAW_PORT=8080
      - OPENCLAW_DEFAULT_MODEL=${OPENCLAW_DEFAULT_MODEL:-claude-3-5-sonnet-20241022}
      - N8N_MCP_URL=${N8N_MCP_URL:-http://n8n:5678/mcp}
    volumes:
      - openclaw_data:/app/data
      - ../connectors:/app/connectors:ro
    depends_on:
      n8n:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  langgraph:
    image: vitraya/langgraph-agent:latest
    restart: unless-stopped
    container_name: vitraya-langgraph
    profiles:
      - ai
    ports:
      - "${LANGGRAPH_PORT:-8123}:8123"
    environment:
      - LANGGRAPH_ENV=production
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://${DB_POSTGRESDB_USER}:${DB_POSTGRESDB_PASSWORD}@postgres:5432/${DB_POSTGRESDB_DATABASE}
      - N8N_MCP_URL=${N8N_MCP_URL:-http://n8n:5678/mcp}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      n8n:
        condition: service_healthy

volumes:
  postgres_data:
  redis_data:
  n8n_data:
  openclaw_data:
注意：LangGraph 镜像需要先通过 langgraph build 构建，详见下文 LangGraph 部署 章节。OpenClaw 使用 profiles: ai 标签，启动时需加 --profile ai 参数。

快速启动
1. 克隆仓库
bash
git clone https://github.com/VitrayaRamunong/Vitraya.git
cd Vitraya/docker
2. 配置环境变量
bash
cp .env.example .env
# 编辑 .env，至少设置 N8N_ENCRYPTION_KEY 和 DB_POSTGRESDB_PASSWORD
3. 启动核心执行环境
bash
# 仅启动核心服务（n8n + PostgreSQL + Redis）
docker-compose up -d
4. 启动 AI 编排层（可选）
bash
# 启动核心 + AI 服务
docker-compose --profile ai up -d
5. 验证服务状态
bash
docker-compose ps
所有服务应显示为 Up 状态。

6. 访问 n8n 控制台
打开浏览器访问 http://localhost:5678，创建管理员账号。导入 connectors/ 目录中的工作流即可开始使用。

暴露公网 Webhook（Tailscale）
如果你需要接收外部平台的实时回调（如 Shopify 订单通知、Stripe 支付事件），需要将 n8n 的 Webhook 端点暴露到公网。

推荐使用 Tailscale Funnel，它免费、易用、自带 HTTPS。

步骤
1. 安装并登录 Tailscale

bash
# 安装 Tailscale（以 Linux 为例）
curl -fsSL https://tailscale.com/install.sh | sh

# 登录并认证
tailscale up
2. 启动 Funnel 暴露 n8n 端口

bash
tailscale funnel --bg --http=5678 localhost:5678
这会将本地 localhost:5678 暴露为一个公网 HTTPS 地址，例如：
https://your-machine-name.tailnet-name.ts.net

3. 配置 n8n 的 Webhook URL

在 .env 中添加：

text
WEBHOOK_URL=https://your-machine-name.tailnet-name.ts.net
然后重启 n8n 服务：

bash
docker-compose restart n8n
⚠️ 安全警告：Funnel 将服务暴露至公网，请确保：

启用 n8n 的基本身份验证（N8N_BASIC_AUTH_USER / N8N_BASIC_AUTH_PASSWORD）

不要暴露未加密或未认证的本地服务

生产环境建议在 n8n 前加一层反向代理（如 Caddy）做 TLS 终结和额外的鉴权

AI 编排层部署
OpenClaw
OpenClaw 是 Vitraya 官方推荐的 AI 长时自主运行框架。

1. 配置 API Key

在 .env 文件中添加你的模型提供商 API Key（至少配置一个）：

bash
ANTHROPIC_API_KEY=sk-ant-xxx
# 或
OPENAI_API_KEY=sk-xxx
2. 可选配置

bash
OPENCLAW_PORT=8080
OPENCLAW_DEFAULT_MODEL=claude-3-5-sonnet-20241022
N8N_MCP_URL=http://n8n:5678/mcp
3. 启动

bash
docker-compose --profile ai up -d openclaw
4. 验证

访问 http://localhost:8080 查看 OpenClaw Gateway 状态。

5. 配置 n8n MCP Server

在 n8n 中创建一个工作流，添加 MCP Server Trigger 节点，配置该节点暴露你需要的 Connectors 作为 Tools。OpenClaw 会自动通过 N8N_MCP_URL 发现并调用这些 Tools。

详细的配置选项（如多模型支持、渠道配置、Skill 管理）请参阅 docs/deploy/openclaw.md。

LangGraph（可选）
LangGraph 提供了有状态持久化与有环工作流能力，适合需要多轮推理、记忆回溯的复杂决策场景。

1. 准备 LangGraph 应用

确保你的 LangGraph 项目根目录包含 langgraph.json 配置文件，用于指定依赖、图和环境变量。

2. 构建 Docker 镜像

bash
# 在 LangGraph 项目根目录执行
langgraph build -t vitraya/langgraph-agent:latest
3. 启动

bash
docker-compose --profile ai up -d langgraph
4. 验证

访问 http://localhost:8123 查看 LangGraph 服务状态。

LangGraph 的完整部署选项（包括 Kubernetes、云 SaaS 等）请参阅 LangGraph 官方文档及 docs/deploy/langgraph.md。

生产环境建议
安全
设置 N8N_ENCRYPTION_KEY：使用 openssl rand -hex 32 生成随机密钥，切勿使用默认值

启用基本身份验证：设置 N8N_BASIC_AUTH_USER 和 N8N_BASIC_AUTH_PASSWORD

使用 HTTPS：设置 N8N_PROTOCOL=https

定期备份：备份 PostgreSQL 数据卷和 n8n_data 卷中的工作流文件

性能
内存分配：为 n8n 容器分配至少 2GB 内存

数据库连接池：根据并发工作流数量调整 PostgreSQL 的 max_connections

历史执行记录：启用 EXECUTIONS_DATA_PRUNE 自动清理旧记录，避免数据库膨胀

Worker 模式：高并发场景下，可配置独立的 n8n Worker 容器处理队列任务

高可用（Kubernetes）
对于需要弹性扩缩容或零停机部署的场景，n8n 官方提供了成熟的 Helm Chart，支持在 K8s 集群中一键部署。

部署时，只需将 Vitraya 的 connectors/ 和 patterns/ 目录通过 ConfigMap 挂载至 n8n 容器的 /home/node/.n8n/ 目录下即可。

参考：n8n Kubernetes 官方文档

故障排查
n8n 无法启动
检查日志：

bash
docker-compose logs n8n
常见原因：

N8N_ENCRYPTION_KEY 未设置或格式不正确

PostgreSQL 未就绪（检查 healthcheck 状态）

端口 5678 被占用

Webhook 无法接收外部回调
确认 WEBHOOK_URL 已正确设置为公网地址

确认 Tailscale Funnel 正在运行：tailscale funnel status

检查外部平台是否将回调发送到了正确的 URL

Redis 连接失败
确认 REDIS_PASSWORD 与 Redis 容器的 requirepass 一致

检查 Redis 容器是否正常运行：docker-compose logs redis

OpenClaw 无法连接 n8n MCP
确认 N8N_MCP_URL 在 OpenClaw 容器中可访问（使用服务名 n8n，而非 localhost）

确认 n8n 中已创建包含 MCP Server Trigger 的工作流并处于激活状态

检查网络：docker-compose exec openclaw curl http://n8n:5678/healthz

参考链接
n8n 官方文档

n8n 环境变量完整列表

Tailscale Funnel 文档

MCP 协议规范

OpenClaw 官方文档

LangGraph 官方文档

