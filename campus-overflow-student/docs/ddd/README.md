# CampusOverflow 领域驱动设计（DDD）文档

本目录是 Sprint 0 事件风暴工作坊（T0-06，PB-03）的产出，用于统一团队的沟通语言与编码边界。

| 文件 | 内容 | 对应指导书 |
| --- | --- | --- |
| [01-context-map.md](01-context-map.md) | 子域划分、限界上下文、上下文映射图与集成关系 | 3.1、3.2 |
| [02-ubiquitous-language.md](02-ubiquitous-language.md) | 统一语言词汇表（术语 / 代码名 / 定义与规则） | 3.3 |
| [03-events.md](03-events.md) | 领域事件清单：发布者、订阅者、集成方式 | 3.5 |
| [04-event-storming.md](04-event-storming.md) | 事件风暴过程记录：事件→命令→聚合→策略→边界 | 3.4、3.6 |

## 使用约定

1. 代码中的类名、方法名、错误信息、Issue 标题、站会发言，一律使用 `02-ubiquitous-language.md` 中的词汇；发现新词先补文档再写代码。
2. 跨上下文只允许依赖对方的 `*-api` 模块（查询接口 + DTO + 事件），禁止 import 其他上下文的 `domain` / `application` 类。
3. 修改 `co-shared-kernel` 需全组评审。
4. 本文档在 S0 出初版，S3 定稿；每次结构调整（新增聚合、新增事件、调整上下文边界）都要同步更新并走 PR。

## 变更记录

| 日期 | 版本 | 变更 | 作者 |
| --- | --- | --- | --- |
| Sprint 0 | v0.1 | 初版，对照指导书第 3 章完成 | 全组 |
