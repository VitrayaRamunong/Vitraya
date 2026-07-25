# Connector 贡献指南

## 什么是 Connector

Connector 是将单个 API 平台封装为标准化“原子能力”的模块，包含：
- **I/O 契约**（JSON Schema）：定义输入与输出的数据结构
- **n8n 工作流**：可导入 n8n 执行的 JSON 文件
- **鉴权配置**：API Key / OAuth / Bearer Token 等

> Connector 是 Vitraya 三层架构的基础层。所有 Patterns 和 Systems 最终都依赖于 Connectors。

## 前置步骤：提取功能性事实（必须）

在提交 Connector 之前，**必须先完成目标平台 API 的“功能性事实”提取**，并申请建立对应的子仓库。

### 什么是功能性事实？

功能性事实是从 API 文档中提取的**可观察行为**：
- 端点 URL
- 参数名称与数据类型
- HTTP 方法
- 鉴权方式
- 响应结构概要

**不包含**：API 的描述性文本、示例代码、教程等可能受版权保护的“表达性内容”。

### 子仓库操作流程

1. 使用项目提供的工具链（或手动）从目标 API 文档中提取功能性事实
2. 将提取结果整理为标准格式（YAML/JSON）
3. 在 [Vitraya 功能性事实组织](https://github.com/VitrayaRamunong) 下申请建立新的子仓库，命名为 `facts-{platform-name}`
4. 子仓库采用 **Apache 2.0** 许可证

## 编写 Connector

### 目录结构

```text
connectors/
└── {platform}/
├── connector.json # I/O 契约 + 元数据
├── workflow.json # n8n 工作流
└── README.md # 使用说明
```

### connector.json 规范

```json
{
  "name": "shopify-orders",
  "version": "1.0.0",
  "platform": "shopify",
  "facts_repo": "https://github.com/VitrayaRamunong/facts-shopify",
  "input_schema": {
    "type": "object",
    "properties": {
      "shop": { "type": "string", "description": "店铺子域名" },
      "status": { "type": "string", "enum": ["open", "closed"] }
    },
    "required": ["shop"]
  },
  "output_schema": {
    "type": "array",
    "items": { "type": "object", "properties": {
      "order_id": { "type": "string" },
      "total": { "type": "number" }
    }}
  },
  "auth": {
    "type": "api_key",
    "header": "X-Shopify-Access-Token"
  }
}
```

### 本地测试

```bash
cd docker
docker-compose up -d
# 导入 workflow.json，在 n8n UI 中手动触发测试
```

## 提交 PR

PR 描述中必须包含：

- 功能性事实子仓库链接
- 测试结果截图或日志
- 是否已通过 CI 检查

## 质量升级

| 当前等级	 | 升级条件 |
| ---------| ---------|
|🧪 Experimental → 🌱 Community	| 通过 CI 检查 + 有完整文档 + 收到 2 个独立使用反馈 |
|🌱 Community → 🏆 Verified	| 经核心团队人工审查 + 场景验证 + 被至少 1 个 Pattern 引用 |

>升级需提交 Issue 申请，由核心团队评估。
