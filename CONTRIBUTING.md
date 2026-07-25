# Contributing to Vitraya

首先，感谢你愿意花时间贡献 Vitraya！

## 行为准则

本项目遵循 [行为准则](CODE_OF_CONDUCT.md)。参与即表示你同意遵守。

## 我能贡献什么？

| 贡献类型 | 详细指南 |
|----------|----------|
| **Connectors** | [docs/contributing/connectors.md](docs/contributing/connectors.md) |
| **Patterns** | [docs/contributing/patterns.md](docs/contributing/patterns.md) |
| **Systems** | [docs/contributing/systems.md](docs/contributing/systems.md) |
| **文档** | 直接编辑仓库中的 `.md` 文件，提交 PR |
| **Issue 反馈** | 提交 [GitHub Issue](https://github.com/VitrayaRamunong/Vitraya/issues) |

## 核心原则

### 自底向上依赖铁律

提交 `Patterns` 或 `Systems` 时，其所依赖的所有 `Connectors` 或底层 `Patterns` **必须同时存在于本次 PR 中，或已存在于 `main` 分支**。

审核顺序严格按 **Connectors → Patterns → Systems** 进行。

### 三级质量分级

所有贡献按成熟度分为三级：

| 等级 | 徽章 | 说明 |
|------|------|------|
| **Experimental** | 🧪 | 草案、概念验证 |
| **Community** | 🌱 | 可复用的完整方案 |
| **Verified** | 🏆 | 生产就绪，官方推荐 |

新提交默认归为 `Experimental`。详见各层级详细指南。

### CLA（贡献者许可协议）

所有贡献者需签署 CLA。详见 [CLA.md](CLA.md)。

## 提交前检查清单

- [ ] 已阅读对应层级的详细贡献指南
- [ ] 已通过本地测试
- [ ] 依赖关系已确认（底层已存在或包含在 PR 中）
- [ ] PR 聚焦于单一功能或修复
- [ ] 提交信息清晰、有描述性

## PR 审核流程

1. 提交 PR → 2. CI 检查 → 3. 人工审核（Connectors → Patterns → Systems 顺序） → 4. 反馈与修改 → 5. 合并

## 获取帮助

- **GitHub Issues**：提交 bug 或功能请求
- **Discord**：加入社区讨论（链接待补充）
