# Lab 0 · S0 环境与架构准备 —— 提交说明

> 团队：六边形小队（CampusOverflow）｜ 提交人：蔡锦莉（PO，@Vespersea）｜ 日期：2026-11-12
>
> 说明：课程实验单元从 Lab 1 开始（《CampusOverflow 实验部分》第 48 页起），没有独立的"Lab 0"章节。本仓库中 Lab 0 对应 **README 第 4 节的 S0（第 1–2 周）：跑通环境、读懂 Identity 参考实现与架构文档，交付"环境验证截图 + 架构理解笔记"**。

## 1. 交付物

| 编号 | 交付物 | 路径 | 状态 |
| --- | --- | --- | --- |
| 0-1 | 架构理解笔记 | [`docs/notes/architecture.md`](../notes/architecture.md) | ✅ 已完成（v1.0，含上下文、模块、分层、事件、6 条适应度函数、ADR 速览、Identity 导读、疑问清单） |
| 0-2 | 环境验证记录 | [`docs/notes/environment.md`](../notes/environment.md) | ✅ 已完成（实测版本 + 前置条件核对 + 待装清单 + 验证步骤） |
| 0-3 | 环境验证截图 | `docs/submissions/lab0-env-*.png` | ⬜ 待补（本机缺 JDK 21 / Maven / Docker，装好并跑通后补 4 张截图） |
| 0-4 | 本说明 | `docs/submissions/lab0.md` | ✅ |

## 2. 环境核对结论（详见 environment.md）

| 工具 | 要求 | 实测 | 结论 |
| --- | --- | --- | --- |
| Git | 2.40+ | 2.45.1 | ✅ |
| JDK | 21 | 仅 1.8 / 26 | ❌ 需装 21 |
| Maven | 3.9+ | 未安装 | ❌ |
| Node.js | 20+ | 22.23.2（npm 10.9.8） | ✅ |
| Docker | Desktop | 未安装 | ❌（影响 Testcontainers / compose） |
| IDE | IDEA / VS Code | 仅 PyCharm | ❌ |

**一句话结论**：仓库可克隆、可读、结构完整；但构建链在本机尚不可执行，`mvn clean test` 未跑通。补装清单与验证命令已写入 `environment.md` 第 3 节。

## 3. 架构理解要点（笔记里的五条结论）

1. **模块化单体的边界不是靠自觉，是靠 `ArchitectureTest` 里 FF-1 ~ FF-6 六条可执行规则**；越界的代码编译/测试直接红。
2. **27 个 Maven 模块 = 6 父模块 + 4 上下文 × 5 层 + 1 装配模块**，拆模块的唯一目的是让"越界依赖"在编译期就引不到。
3. **上下文之间只有两条通道**：同步查 `*-api` 发布语言、异步走"事务性发件箱 + 幂等消费"；禁止直接 import 别人的 domain/application。
4. **领域层必须能脱离 Spring 单独 new 出来跑测试**（FF-1 + ADR-008），JPA 实体只在 infrastructure 层。
5. **采纳一条回答会跨三个上下文**（Q&A 发事件 → Reputation 记账/徽章/悬赏 → Discovery 投影/通知），理解这条链路就等于理解整个系统的协作方式。

## 4. 发现并已登记的问题

| # | 问题 | 影响 | 处理 |
| --- | --- | --- | --- |
| 1 | 项目代码整体在 `campus-overflow-student/` 子目录，`.github/workflows` 也在其中 | 仓库根 Actions 页空白，**CI 从未运行** | 已登记，需团队决策是否把内容提到仓库根（影响全员，不在 Lab0/Lab1 内擅自动） |
| 2 | 本机无 JDK 21 / Maven / Docker | 无法本地验证测试与覆盖率门禁 | 已给安装清单，待补截图 |
| 3 | ADR 001/003/005/006 文件缺失，四个 ADR 文件名乱码，链接全断 | 架构文档可读性受损 | 已登记（Lab2/Lab3 文档债） |
| 4 | 根目录提交了 `.idea/` | IDE 配置混入仓库 | 已加入 `.gitignore` 并从索引移除（本地文件保留） |

## 5. 验收对应

- [x] 读懂 Identity 参考实现的分层结构（笔记第 8 节给出"由外向内读三遍"的读法，全队可按此复现）
- [x] 读懂架构文档与 ADR（笔记第 6、7 节）
- [x] 形成书面产出（架构理解笔记 v1.0）
- [ ] 环境验证截图（装机后补，见 environment.md 第 3 节命令）
