# System 贡献指南

## 什么是 System

System 是端到端的自治系统蓝图，它整合情报、执行与现金流，形成完整的“超级个体飞轮”。

> System 是 Vitraya 三层架构的顶层，是对“完整商业闭环”的具象化。

## 依赖铁律

提交 System 时，其所引用的所有 Patterns 和 Connectors 必须满足以下条件之一：
- 已存在于 `main` 分支
- 包含在本次 PR 中

## 目录结构

```text
systems/
└── {system-name}/
├── system.yaml # 部署配置
├── agent-config.yaml # AI Agent 配置
├── workflows/ # 引用的 Patterns / Connectors（符号链接或复制）
└── README.md # 完整说明与部署指引
```

## system.yaml 规范

```yaml
name: ecommerce-autopilot
version: 1.0.0
depends_on:
  patterns:
    - order-fulfillment
    - inventory-sync
    - pricing-optimizer
  connectors:
    - shopify-orders
    - stripe-payment

deployment:
  compose: true
  ports:
    - "5678:5678"

flywheel:
  info_driven: true
  auto_execution: true
  cashflow_positive: true
```

## agent-config.yaml 规范

```yaml
agent:
  framework: openclaw
  entry_workflow: main-workflow
  approval_gateway:
    enabled: true
    channels: ["slack", "email"]
  reporting:
    frequency: daily
    metrics: ["execution_count", "success_rate", "cashflow"]

state_machine:
  type: langgraph
  persistence: postgresql
```

## 特别要求

- Verified / Flagship 级别的 System 必须符合“情报驱动 → 自动执行 → 正向现金流”闭环
- 必须提供完整的部署指引，确保用户能够复现

## 提交 PR

PR 描述中必须包含：

- 依赖清单（Patterns + Connectors）及来源
- 系统运行截图或演示
- 是否已通过本地完整测试

## 质量升级
| 当前等级	| 升级条件 |
|---------|----------|
| 🧪 Experimental → 🌱 Community	| 端到端可部署 + 有完整文档 |
| 🌱 Community → 🏆 Verified	| 符合“情报 → 执行 → 现金流”闭环 + 经核心团队审查 + 有真实落地案例 |
