# 文档导航

基线版本：`0.2`；建立/更新日期：2026-10-08；状态：复用与开发分工文档已完善，运行验证待开展。

本文档集将用户附件里的调研、复用方案、开发阶段与命名方案转为实施依据。附件原文完整保留，当前方案取其中后续明确的“优先复用官方内核”路线。本轮独立复用方案另行归档；不改写旧附件或历史工作记录。

## 建议阅读顺序

1. [项目定义与范围](project-overview.md)：项目是什么、首期交付什么。
2. [调研与证据](research.md)：官方信息、历史推断及尚需验证的部分。
3. [复用与开发边界总表](reuse-development-matrix.md)与[技术路线](technical-route.md)：统一分工、逐系统处理方式与选型。
4. [技术架构](architecture.md)：在线、内容、平台和发布边界。
5. [实现路线图](roadmap.md)与[任务清单](backlog.md)：按依赖实施。
6. [当前进度](progress/README.md)与[工作日志](progress/worklog.md)：继续工作前必读。

## 技术设计

| 文档 | 解决的问题 |
|---|---|
| [官方运行时接入](runtime-integration.md) | 安装清点、版本能力、适配边界、探针 |
| [游戏内容与业务分工](gameplay-content.md) | 官方机制、内容配置、自有规则、UI 与资源的实施工作包 |
| [Lua 与客户端](lua-client.md) | 模块、生命周期、GUI、多端与脚本边界 |
| [数据与存储](data-storage.md) | 表格、ID、迁移、订单与数据所有权 |
| [消息与安全](protocol-security.md) | 业务契约、鉴权、校验、限流和审计 |
| [外围平台服务](platform-services.md) | 账号、支付、发货、GM、CDK、统计 |
| [工具链与发布](tooling-release.md) | 校验、构建、BOM、灰度和回滚 |
| [部署与运维](deployment-operations.md) | Windows 区服、网络、备份、恢复与故障 |
| [测试与验收](testing.md) | mock 与实测、场景、性能、里程碑证据 |
| [风险与待决事项](risks-and-questions.md) | 外部依赖、风险控制、默认假设 |
| [架构决策](decisions/README.md) | 已采纳路线和待验证选型 |

## 跟踪与模板

- [进度维护说明](progress/README.md#维护规则)；仓库强制规则见 [AGENTS.md](../AGENTS.md)。
- [工作记录模板](templates/work-record.md)。
- [功能分工与验收模板](templates/feature-scope.md)：先证明复用差异，再确定最小开发范围。
- [版本 BOM 模板](templates/release-bom.example.yaml)。
- [能力验证矩阵](templates/capability-matrix.csv)。
- [发布检查单](templates/release-checklist.md)。
- [本轮官方内核同构方案](reference/runtime-reuse-proposal.md)：2026-10-08 用户补充的完整复用方案，原文逐字节归档。
- [附件原始方案](reference/original-proposal.md)与[来源/哈希记录](reference/README.md)：历史资料，不是当前进度。

## 文档状态说明

“采纳”代表设计方向已确定；“拟定”代表实施时可以根据证据调整；“待验证”代表依赖锁定版本或环境；“已实现”必须关联任务与验证记录。所有设计中的自有 `ER.*` API、HTTP 路径、数据表和目录均是拟建契约，当前没有对应实现。

变更设计时同步路线图、任务、决策和进度。供应商资料冲突时记录分支与版本，不用旧文档覆盖新环境实测。引用链接不可访问时保留出处和访问结果。
