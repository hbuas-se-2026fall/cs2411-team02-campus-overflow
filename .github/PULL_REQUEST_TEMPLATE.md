## 变更目的

<!-- 一句话说清这个 PR 解决什么问题，不要写"修改了一些文件" -->

## 关联

- 用户故事 / 任务：Closes #<issue 编号>
- 所属 Sprint：Sprint <N>
- 相关 ADR：<ADR-00X / 无>

## 变更类型

- [ ] 新功能（feat）
- [ ] 缺陷修复（fix）
- [ ] 重构（refactor，无行为变化）
- [ ] 文档（docs）
- [ ] 测试（test）
- [ ] 构建 / CI（chore）

## 改动范围

| 模块 | 说明 |
| --- | --- |
| `<模块名>` | <改了什么> |

## 验证

```bash
# 贴真实输出，不要只写"已测试"
mvn -pl <模块> -Dtest=<测试类> test
```

- [ ] 本地测试通过
- [ ] ArchUnit 架构约束通过
- [ ] 领域层覆盖率未低于门禁
- [ ] 前端 `npm run typecheck && npm run build` 通过（涉及前端时）

## 评审自检

- [ ] 领域层没有引入 Spring / JPA / Jackson 依赖（FF-1）
- [ ] 没有跨上下文直接 import 别人的 domain / application（FF-2）
- [ ] 控制器里没有业务判断（业务在领域对象里）
- [ ] 契约变更已同步 OpenAPI，前端字段已确认
- [ ] 错误码使用约定的分段与前缀
- [ ] 新增/修改的逻辑有对应测试，没有放宽已有断言
- [ ] 提交信息符合 Conventional Commits

## 截图（涉及 UI 时必填）

<!-- 贴前后对比 -->

## Review 记录

- 评审人：@<用户名>
- 结论：<LGTM / 需修改（列出具体文件和行号）>
