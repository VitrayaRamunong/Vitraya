# Vitraya: The Open Execution Infrastructure for Agentic Web

[![License](https://img.shields.io/github/license/VitrayaRamunong/Vitraya)](LICENSE)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-blue)](https://modelcontextprotocol.io)
[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1-brightgreen)](https://www.openapis.org/)

> **连接信息，是互联网的过去；连接执行，是互联网的未来。**
>
> **Vitraya**旨在构建 AI 时代的“执行能力注册表与组合引擎”。通过将繁杂的 API 封装为标准化的 Connectors（原子能力）、Patterns（组合逻辑）和 Systems（自治系统），我们让 AI Agent 能够像调用函数一样调用数字世界（甚至物理世界）的执行能力。

---

## 🎯 核心定位：超越“自动化工具”

本项目不是又一个 n8n 节点库，而是一个**横跨能力层、编排层与系统层的全栈开放执行基础设施**。

- **能力层 (Capability)**：提供标准 I/O 契约的 Connectors，原生支持 MCP 协议（供 AI Agent 直接调用）及 OpenAPI 3.1 导出（供其他自动化平台导入）。
- **编排层 (Orchestration)**：封装跨域业务逻辑的 Patterns，通过状态机与异常回滚保证执行的确定性，并以能力标签标注其生产就绪度。
- **系统层 (System)**：整合情报、执行与现金流的“超级个体飞轮”蓝图，支持长时 AI 自主运行 + 人类审批。

## 🔌 开放协议：MCP 与 OpenAPI

Vitraya 的能力通过两层标准化协议对外暴露：

- **MCP（Model Context Protocol）**：面向 AI Agent。所有 Connectors 可封装为 MCP Tools，LangGraph、OpenClaw、CrewAI 等框架可直接发现与调用。
- **OpenAPI 3.1**：面向自动化平台与开发者。一键导出标准 Spec，可导入 Zapier、Make、Postman、Dify 等任意支持 OpenAPI 的系统。

**关于执行运行时**：n8n 是 Vitraya 官方维护的参考运行时（Reference Runtime）。我们选择它，是因为它在开源工作流引擎中具备最强的确定性执行能力与可视化调试体验。但 Vitraya 的核心资产是 I/O 契约，而非绑定任何特定引擎的工作流——通过 OpenAPI 导出，你可以将 Vitraya 的能力集成到任何支持 HTTP 调用的平台。

## 📖 哲学与缘起

### 🌍 为什么是现在？执行互联网的结构性必然

早期互联网解决的是​​“信息的连接与共享”​​，而当前正在发生的，正是​​“执行能力的连接与共享”​​。这不仅是趋势，更是技术、经济与架构演进叠加下的​​结构性必然​​：

- 🏗️ **​软件架构演进​**​：单体 → 微服务 → API 化 → Agent 可调用。软件正从“给人操作”转向“给 AI 调度”。
- 💰 **​经济 ROI 刚性​**​：跨系统手动对接成本极高，AI 编排将边际执行成本趋近于零。自动化执行的付费意愿已被市场验证。
- 🌐 **​物理-数字融合​**​：IoT、机器人、工业设备全面暴露接口。AI Agent 正在成为物理世界的“数字神经末梢”。
- 📡 **​协议收敛效应​**​：历史证明，一旦开放协议形成标准，开发者生态会自我强化，最终覆盖封闭体系。

执行互联网的本质，是将“数字世界的操作权限”标准化、可发现、可组合，并由AI作为调度中枢。**Vitraya**的 Connectors → Patterns → Systems 架构，正是为这一范式铺设的​​能力分层轨道​​。

### 🔗 Agentic Web 亟需 API 连接生态

当前的 AI Agent 拥有强大的推理与规划能力，却缺乏​**​标准化、可组合、可观测的执行边界​**​。海量 API 仍是“给人看的文档”，而非“给机器读的契约”。参数格式不统一、鉴权逻辑碎片化、错误码无规范、副作用无约束，导致 Agent 在落地时陷入“胶水代码泥潭”。Agentic Web 的规模化，亟需一座将“意图”转化为“确定性动作”的中间件。

### 🌊 "More is different"，系统涌现

当 API Connectors 被标准化注册并达到一定规模和多样性时，量变将引发质变。这不是单纯的网络效应，而是组合创新（Combinatorial Innovation）——就像自然界中，原子种类足够多之后，分子结构的可能性呈指数级增长。孤立环境中从未连接过的 API 组合逻辑将被发现，从而涌现出全新的业务范式。

### 🌀 正反馈飞轮：系统的动力来源

飞轮的本质是正反馈循环——系统的输出成为下一轮运转的输入，每一次循环都在增强系统本身。

在宏观层面，Vitraya 的愿景是驱动一个“超级个体飞轮”：
系统产生正向现金流 → 支撑个体发展 → AI分身扩展能力边界 → 情报工作流开辟新领域 → 新领域被连接入系统 → 系统产生新的现金流……

### 🔄 从文档到契约：工具链驱动递归自举

我们希望构建 `API 文档爬取 → 功能性事实提取 → 契约化 Connector 生成 → 沙箱验证 → 自动注册` 的完整工具链，从而降低新API连接的实现门槛。随着更多patterns和systems的涌现，其中的经验可以沉淀为agent的skill，配合自动化工具链，我们的愿景是实现AI自动化系统的AI自动化搭建。

## 🚀 Quick Start

### 前置条件
- 已安装 Docker 和 Docker Compose
- （可选）已安装 Tailscale 用于公网 Webhook 暴露

### 1. 克隆仓库
```bash
git clone https://github.com/VitrayaRamunong/Vitraya.git
cd Vitraya/docker
```

### 2. 启动核心执行环境（n8n + PostgreSQL + Redis）

#### 2.1. 准备环境变量文件
```bash
cp .env.example .env
cp n8n/.env.n8n.example n8n/.env.n8n
cp n8n/.env.runners.example n8n/.env.runners
cp postgres/.env.postgres.example postgres/.env.postgres
cp redis/.env.redis.example redis/.env.redis
```
#### 2.2. 填充必要的环境变量

编辑每个 .env.{service} 文件，至少填入以下必需的配置值：
- n8n/.env.n8n：n8n密钥、数据库密码等
- n8n/.env.runners：n8n runners的token、需要的modules或python packages
- postgres/.env.postgres：POSTGRES_PASSWORD（设置一个强密码）
- redis/.env.redis：REDIS_PASSWORD（设置一个强密码）

**注意**：docker根目录的.env文件只包含以下环境变量：
- `COMPOSE_PROFILES`--用于控制启动哪些服务组合。默认只启动以上核心执行环境，如需其他，请按需修改。
- `xx-IMAGE`--用于控制各组件的版本。

#### 2.3. 一键启动
```bash
docker compose up -d
```

### 3. 访问 n8n 控制台
打开浏览器访问 http://localhost:5678，创建管理员账号。

### 4. 导入 Connectors
在 n8n 界面中，选择 工作流 → 导入，从 connectors/ 目录中选择需要的 API 连接器。

### 5. 连接 AI Agent（可选）
如果你希望让 AI 长时自主调用这些 Connectors：
- 参考 DEPLOYMENT.md 中的 OpenClaw / LangGraph 配置章节
- 或直接使用 n8n 内置的 Webhook 触发执行
>更多配置选项（数据库调优、公网暴露、生产环境部署）请参阅 DEPLOYMENT.md。

## 📦 Dependencies & Deployment

Vitraya 推荐使用 **Docker Compose** 一键启动核心执行环境，兼容 Linux / macOS / Windows。

### 核心依赖

| 服务 | 用途 | 备注 |
|------|------|------|
| **n8n** | 确定性执行引擎 & 工作流运行时 | Vitraya 的官方参考运行时 |
| **PostgreSQL** | n8n 的状态持久化存储 | 生产环境必备，替代默认的 SQLite |
| **Redis** | 缓存与任务队列 | 提升 n8n 多实例/定时任务可靠性 |
| **Tailscale** | 公网 Webhook 暴露 | 用于接收外部平台的回调（如 Shopify 订单通知） |

### 可选 AI 编排层

| 框架 | 用途 | 推荐场景 |
|------|------|----------|
| **OpenClaw** | 长时自主运行 AI Agent | 持续监控、周期执行、主动汇报（官方推荐，熟悉度高） |
| **LangGraph** | 有状态持久化 + 有环工作流 | 复杂决策循环、需要记忆与回溯的 Agent 任务（优势互补） |

完整部署架构、环境变量配置与生产环境调优，请参阅：

- 完整部署指南 (DEPLOYMENT.md)
- 各依赖详细部署文档 (docs/deployment/)

## 🤝 Contributing
Vitraya 是一个社区驱动的开放基础设施项目。我们欢迎所有形式的贡献，无论你是提交一个新的 Connector、完善一个 Pattern，还是改进文档。

### 三层贡献路径
|层级|贡献形式|准入门槛|
|----|-------|--------|
|Connectors	|新增 API 连接器（含 I/O 契约）|	提供 input_schema / output_schema + n8n 工作流 JSON|
|Patterns	|跨域组合逻辑模板|	引用已有 Connectors，提供状态机与异常回滚设计|
|Systems	|端到端自治系统蓝图|	引用已有 Patterns，提供部署配置与 AI 审批协议|
### 自底向上依赖铁律
提交 Patterns 或 Systems 时，其所依赖的所有 Connectors 或底层 Patterns 必须同时存在于本次 PR 中，或已存在于 main 分支。审核将严格按照 Connectors → Patterns → Systems 的顺序进行。
### 质量分级
所有贡献按成熟度分为三级，标注于对应目录的元数据中：

| 等级 | 徽章 | 说明 |
|------|-----|------|
|Experimental | 🧪 |	草案或单次验证，尚未经过完整测试 |
|Community | 🌱	| 可复用方案，通过基础 CI 检查与契约校验 |
|Verified | 🏆	 | 生产就绪，经过人工审查与场景验证，由核心团队维护 |

>完整的贡献指南、PR 模板与本地测试流程，请参阅 CONTRIBUTING.md。
## ⚠️ Disclaimer
Vitraya 是一个开源社区项目，所有 Connectors、Patterns 与 Systems 均由社区贡献者维护。

- **API 合规**：本项目仅提取 API 的功能性事实（端点、参数、数据类型、HTTP 方法），不复制受版权保护的描述性文档内容。贡献者有责任确保其提交内容符合目标平台的开发者协议。
- **实验性风险**：标记为 Experimental 或 Community 的组件未经全面生产验证，使用前请在沙箱环境中充分测试。
- **执行责任**：Vitraya 提供执行能力，但不对自动化操作的结果（包括但不限于资金损失、数据丢失、平台违规）承担责任。使用者应自行评估风险并配置适当的审批与熔断机制。

## 📄 License

Vitraya 采用多许可证策略：
- 核心仓库（本仓库）使用 **GPL v3.0**。
- 功能性事实子仓库使用 **Apache 2.0**。
- 贡献者需签署 **CLA**。

详见 [LICENSE](LICENSE)、[CLA.md](CLA.md) 与 [LEGAL.md](LEGAL.md)。

## 📜 What "Vitraya" Means

**Vitraya** 这个名字源自电影《阿凡达》中的 **"Vitraya"**（灵魂之树）——潘多拉星球上连接万物、承载记忆与能量的神圣存在。

在我们的语境中，它隐喻着：
- **连接**：如同灵魂之树连接潘多拉的所有生命，Vitraya 连接数字世界的所有 API 与执行能力。
- **记忆**：如同灵魂之树承载祖先的记忆，Vitraya 的 Patterns 与 Systems 沉淀着社区的最佳实践与执行经验。
- **涌现**：如同灵魂之树下生命网络的协同演化，当 Connectors 足够丰富时，系统将自发涌现出超越单一 API 的新价值。

*（若您更熟悉梵语词源，Vitraya 亦与 "Vi-traya"（三重视野）相通——恰好暗合项目的三层架构：Connectors、Patterns、Systems。）*
