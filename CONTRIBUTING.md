Contributing to Vitraya
首先，感谢你愿意花时间贡献 Vitraya！无论你是修复一个 bug、添加一个新的 Connector、完善一个 Pattern，还是改进文档，每一份贡献都在让这个项目变得更好。

Vitraya 是一个社区驱动的开放基础设施项目。我们欢迎所有形式的贡献——不限于代码，也包括文档、示例、反馈和推广。

在开始之前：请先阅读 README 了解项目的核心定位与三层架构，这有助于你理解自己的贡献在整体中的位置。

行为准则
本项目及所有参与者均受 行为准则 约束。参与即表示你同意遵守该准则。如有不可接受的行为，请通过 conduct@vitraya.dev 报告。

我能贡献什么？
贡献类型	说明	适合谁
Connectors	新增或改进某个 API 平台的连接器（含 I/O 契约 + n8n 工作流）	熟悉特定 API、愿意封装标准接口的开发者
Patterns	新增或改进跨域组合逻辑模板	有跨系统编排经验、能抽象通用模式的架构师
Systems	新增或改进端到端自治系统蓝图	能构建完整闭环解决方案的全栈实践者
文档	改进 README、部署指南、API 文档等	所有愿意帮助项目更易懂的人
Issue 反馈	报告 bug、提出功能建议、讨论设计	所有使用者
不确定从哪里开始？ 可以查看带有 good first issue 标签的 Issue，这些都是为新手准备的入门任务。

贡献路径
Vitraya 采用 自底向上（Bottom-Up） 的贡献模式。每一层的贡献都依赖于其下层：

text
Connectors（原子能力）
    ↓
Patterns（组合逻辑，依赖 Connectors）
    ↓
Systems（自治系统，依赖 Patterns + Connectors）
自底向上依赖铁律
提交 Patterns 或 Systems 时，其所依赖的所有 Connectors 或底层 Patterns 必须同时存在于本次 PR 中，或已存在于 main 分支。

这条规则确保仓库始终保持可运行状态，避免出现“悬空依赖”——引用了不存在组件的上层逻辑。

审核顺序：所有 PR 将严格按照 Connectors → Patterns → Systems 的顺序进行审核。若底层未就绪，上层将被标记为 Awaiting Dependencies 并暂缓合并。

三级质量分级
所有贡献按成熟度分为三级，标注于对应目录的元数据中：

等级	徽章	适用场景	准入门槛
Experimental	🧪	草案、概念验证、单次验证	提供基本定义与说明，能跑通即可
Community	🌱	可复用的完整方案	通过 CI 检查与契约校验，有完整文档
Verified	🏆	生产就绪，官方推荐	经过人工审查与场景验证，由核心团队维护
新提交默认归入 Experimental。随着使用反馈和迭代，贡献者可申请升级。

Connector 贡献流程
0. 前置步骤：提取功能性事实（子仓库）
在提交 Connector 之前，必须先完成该平台 API 的“功能性事实”提取，并申请建立对应的子仓库。

什么是功能性事实？

功能性事实是从 API 文档中提取的可观察行为，而非受版权保护的“表达性内容”：

端点 URL

参数名称与数据类型

HTTP 方法（GET/POST/PUT/DELETE 等）

鉴权方式（API Key / OAuth / Bearer Token 等）

响应结构概要

操作流程：

使用项目提供的工具链（或手动）从目标 API 文档中提取功能性事实。

将提取结果整理为标准格式（YAML/JSON）。

在 Vitraya 功能性事实组织 下申请建立一个新的子仓库，命名为 facts-{platform-name}。

子仓库采用 Apache 2.0 许可证。

功能性事实子仓库仅存放“事实数据”，不包含任何可执行逻辑。这既降低了版权合规风险，也为后续的 Connector 自动化生成提供了标准化输入。

1. 编写 Connector
基于功能性事实，在主仓库中编写 Connector：

定义 I/O 契约（input_schema / output_schema，JSON Schema 格式）

提供对应的 n8n 工作流 JSON（位于 connectors/{platform}/ 目录）

包含必要的鉴权配置与错误处理

Connector 目录结构示例：

text
connectors/
└── shopify/
    ├── connector.json          # I/O 契约 + 元数据
    ├── workflow.json           # n8n 工作流
    └── README.md               # 使用说明
2. 本地测试
在提交 PR 之前，请先在本地验证 Connector 能正常工作：

bash
# 启动本地 n8n 环境（参考 DEPLOYMENT.md）
cd docker
docker-compose up -d

# 导入你的 Connector 工作流，手动触发测试
# 确认输入输出符合 I/O 契约
3. 提交 PR
在 PR 描述中请包含：

Connector 名称与目标平台

功能性事实子仓库的链接（证明事实已提取）

测试结果截图或日志

是否已通过 CI 检查

Pattern 贡献流程
Pattern 是跨域组合逻辑的封装，它引用一个或多个 Connectors，并定义它们之间的数据流、状态机与异常回滚逻辑。

Pattern 目录结构示例：

text
patterns/
└── order-fulfillment/
    ├── pattern.json            # 元数据 + 依赖声明
    ├── workflow.json           # n8n 工作流
    └── README.md               # 使用说明与依赖列表
依赖声明：在 pattern.json 中明确列出所依赖的 Connectors 及版本：

json
{
  "name": "order-fulfillment",
  "depends_on": {
    "connectors": ["shopify", "stripe", "shipstation"]
  }
}
System 贡献流程
System 是端到端的自治系统蓝图，它整合情报、执行与现金流，形成完整的“超级个体飞轮”。

System 目录结构示例：

text
systems/
└── ecommerce-autopilot/
    ├── system.yaml             # 部署配置
    ├── agent-config.yaml       # AI Agent 配置
    ├── workflows/              # 引用的 Patterns / Connectors
    └── README.md               # 完整说明与部署指引
System 的特别要求：

必须包含 deployment.yaml 或 docker-compose.yml 片段，确保可被复现部署

必须包含 agent-config.yaml，说明 AI Agent 的调用方式与审批协议

旗舰级（Verified）System 需符合“情报驱动 → 自动执行 → 正向现金流”闭环

提交前检查清单
在提交 PR 之前，请确认以下事项：

□ 代码/配置已通过本地测试
□ 已添加或更新相应的文档（README / 注释）
□ 已确认依赖关系（底层 Connectors / Patterns 已存在或包含在 PR 中）
□ PR 聚焦于单一功能或修复（避免“大杂烩”PR）
□ 提交信息清晰、有描述性（参考：Conventional Commits）
PR 审核流程
提交 PR：从你的 fork 分支向 main 分支发起 PR。

CI 检查：所有自动化检查必须通过（契约校验、格式检查等）。

人工审核：核心维护者将审核你的 PR。审核顺序为 Connectors → Patterns → Systems。

反馈与修改：根据审核意见进行修改并推送更新。

合并：审核通过后，PR 将由维护者合并入 main 分支。

CLA（贡献者许可协议）
Vitraya 要求所有贡献者签署 贡献者许可协议（CLA）。

为什么需要 CLA？

GPL v3.0 已经为专利授权提供了保护，但 CLA 解决的是 GPL 作为公共许可证无法覆盖的问题：

许可证升级与商业化：为未来推出企业版/商业版保留操作空间

版权归属清晰化：要求贡献者声明有权贡献，降低版权纠纷风险

应对法律环境变化：允许项目在必要时调整许可证策略

如何签署？

请在第一次提交 PR 时，在 PR 评论中说明“I have read and agree to the CLA”，或按照 CLA.md 中的指引完成签署。

CLA 是 Vitraya 能够长期可持续发展的重要保障。签署后，你仍保留贡献的版权，仅授予项目方将贡献用于 GPL v3.0 及未来商业版本的权利。

获取帮助
GitHub Issues：提交 bug 报告或功能请求

Discord / Slack：加入社区讨论（链接待补充）

邮件：如有其他问题，可联系 maintainers@vitraya.dev

再次感谢
开源项目的生命力来自每一个贡献者。无论你提交的是代码、文档、还是一个 bug 报告，你都在帮助 Vitraya 变得更好。

欢迎加入，一起构建 Agentic Web 的执行基础设施。
