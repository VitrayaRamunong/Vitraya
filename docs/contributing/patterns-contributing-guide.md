# Pattern 贡献指南

## 什么是 Pattern

Pattern 是跨域组合逻辑的封装，它引用一个或多个 Connectors，定义它们之间的数据流、状态机与异常回滚逻辑。

> Pattern 是 Vitraya 三层架构的中间层，是对“业务场景的抽象模板”。

## 依赖铁律

提交 Pattern 时，其所引用的所有 Connectors 必须满足以下条件之一：
- 已存在于 `main` 分支
- 包含在本次 PR 的 `connectors/` 目录下

**不允许引用不存在的 Connector。**

## 目录结构

```text
patterns/
└── {pattern-name}/
├── pattern.json # 元数据 + 依赖声明
├── workflow.json # n8n 工作流
└── README.md # 使用说明与依赖列表
```

## pattern.json 规范

```json
{
  "name": "order-fulfillment",
  "version": "1.0.0",
  "depends_on": {
    "connectors": ["shopify-orders", "stripe-payment", "shipstation-shipment"]
  },
  "state_machine": {
    "states": ["init", "payment", "shipment", "done", "failed"],
    "transitions": { ... }
  },
  "error_handling": {
    "retry": { "max_attempts": 3, "backoff": "exponential" },
    "rollback": { "enabled": true, "actions": [...] }
  }
}
```

## 本地测试
与 Connector 类似，导入 n8n 后测试完整流程，确保所有依赖的 Connector 已加载。

## 提交 PR
PR 描述中必须包含：
- 依赖的 Connectors 清单及来源（已有或本次新增）
- 测试结果截图或日志

## 质量升级
| 当前等级	| 升级条件 |
|---------|----------|
| 🧪 Experimental → 🌱 Community |	通过 CI 检查 + 有完整文档 + 收到 2 个独立使用反馈 |
| 🌱 Community → 🏆 Verified	| 经核心团队人工审查 + 被至少 1 个 Verified System 引用 |
