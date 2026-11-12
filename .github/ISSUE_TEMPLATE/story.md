---
name: 用户故事（Story）
about: 提交一条 Product Backlog 用户故事（Lab2 起按此格式）
title: "[US-XX] 作为<角色>，我希望<能力>，以便<价值>"
labels: "type: story"
assignees: ""
---

## 用户故事

作为 **<角色：学生 / 教师 / 访客>**，
我希望 **<能力>**，
以便 **<价值>**。

## 验收标准（Gherkin）

```gherkin
Feature: <特性名>
  Scenario: 快乐路径
    Given <前置条件>
    When <操作>
    Then <可观察的结果>

  Scenario: 边界
    Given ...
    When ...
    Then ...

  Scenario: 异常
    Given ...
    When ...
    Then ...
```

## INVEST 自评

- [ ] Independent（与其他故事无强依赖）
- [ ] Negotiable（细节可讨论，不是死需求）
- [ ] Valuable（对用户有可感知价值）
- [ ] Estimable（团队能估点）
- [ ] Small（一个 Sprint 内可完成）
- [ ] Testable（有明确的通过/失败判定）

## 估算与归属

| 项 | 内容 |
| --- | --- |
| Story Point | <1 / 2 / 3 / 5 / 8> |
| 优先级 | <P0 必须有 / P1 应该有 / P2 可以有> |
| 所属 Sprint | Sprint <N> |
| 限界上下文 | <identity / qa / reputation / discovery> |
| 关联 ADR | <ADR-00X> |

## DoD 检查（合入前逐项勾）

- [ ] 代码经 PR 合入 `main`，至少 1 名非作者 LGTM
- [ ] 对应 Gherkin 场景全部有自动化测试
- [ ] ArchUnit 六条约束通过
- [ ] 错误码遵循 `docs/错误码.md` 分段
- [ ] 契约变更已同步 OpenAPI
