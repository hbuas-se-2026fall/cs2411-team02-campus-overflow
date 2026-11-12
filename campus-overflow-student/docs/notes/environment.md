# 环境验证记录（S0 · Lab0）

> 执行人：蔡锦莉（@Vespersea）｜ 机器：LAPTOP-1LNHK3AH（Windows）｜ 记录时间：2026-11-12
> 结论先行：**环境尚未就绪**，`mvn clean test` 在本机跑不起来。缺 JDK 21、Maven、Docker、IntelliJ IDEA。Git 与 Node 已达标。

## 1. 前置条件核对（对照 Lab1《环境与前置准备》）

| 工具 | 要求 | 本机实测 | 结论 |
| --- | --- | --- | --- |
| Git | 2.40+ | `git version 2.45.1.windows.1`（`C:\Program Files\Git\cmd\git.exe`，未加入 PATH） | ✅ 达标，但需把 `C:\Program Files\Git\cmd` 加进 PATH |
| JDK | 21 | 仅有 `corretto-1.8.0_492`、`openjdk-26`（`.jdks`）、`jre1.8.0_481`（`Program Files\Java\latest`） | ❌ **无 21**；26 可直接跑但 Lombok/字节码插件风险未知，不作为正式环境 |
| Maven | 3.9+ | `mvn` 不在 PATH，全盘常见路径无 `mvn.cmd` | ❌ 未安装 |
| Node.js | 20+ | `node v22.23.2` | ✅ 达标 |
| Docker | Docker Desktop | `docker` 命令不存在，无 Docker Desktop | ❌ 未安装（影响 Testcontainers 集成测试与 `deploy/docker-compose.yml`） |
| IDE | IntelliJ IDEA / VS Code | 仅 PyCharm Community 2024.1.5 | ❌ 需补装 IDEA（后端）或 VS Code |
| GitHub 账号 | 已加入课程组织 | 已登录 `hbuas-se-2026fall/cs2411-team02-campus-overflow`，对仓库有管理权限 | ✅ |

## 2. 仓库状态

```
本地路径：C:\Users\caijl\OneDrive\桌面\CampusOverflow
origin  → https://github.com/Vespersea/CampusOverflow.git            （个人仓库）
target  → https://github.com/hbuas-se-2026fall/cs2411-team02-campus-overflow.git  （团队仓库）
分支    → develop（跟踪 target/develop）、feature/update-project（当前）、main（跟踪 origin/main）
```

发现的问题：

1. `main` 跟踪的是**个人仓库** origin，不是团队仓库 target——从本地 `main` 推送会推到个人仓库，容易造成"以为合到团队 main 了"的错觉。
2. 项目代码整体嵌在 `campus-overflow-student/` 子目录，而 `.github/workflows` 也在子目录里 → GitHub 只读仓库根 `.github/workflows`，**Actions 从未运行过**。
3. 提交信息不规范（`自我介绍`、`第一次提交(更改README)`），不符合章程 5.2 的 Conventional Commits 约定。

## 3. 待办安装清单（把环境补齐后本记录才闭环）

```powershell
# 1) JDK 21（Temurin 或 Microsoft OpenJDK 均可，装完设 JAVA_HOME）
winget install EclipseAdoptium.Temurin.21.JDK

# 2) Maven
winget install Apache.Maven

# 3) Docker Desktop（跑 Testcontainers 与 compose 需要）
winget install Docker.DockerDesktop

# 4) IDEA Community（后端）
winget install JetBrains.IntelliJIDEA.Community

# 5) 把 Git 加入 PATH（当前新窗口里 git 不可用）
# 系统环境变量 Path 追加：C:\Program Files\Git\cmd
```

安装完成后按下面顺序验证，**每一步留下截图**（Lab0 要求的"环境验证截图"）：

```powershell
git --version                 # 期望 2.40+
java -version                 # 期望 21.x
mvn -v                        # 期望 3.9+，且最上面一行显示 java version: 21
docker version                # 期望 Client/Server 都返回版本
node -v && npm -v             # 期望 Node 20+ / npm 10+

cd backend
mvn -q clean test             # 期望：107 个 TODO 相关的失败 —— 这是"起点是红的"，属预期
cd ..\frontend
npm install
npm run typecheck             # 期望通过
```

## 4. 本次已完成的验证（用现有工具链能做的部分）

| 项 | 方法 | 结果 |
| --- | --- | --- |
| 仓库可克隆、工作区干净 | `git status --short` | 无输出，工作区干净 |
| 远程可达 | `git ls-remote target`（登录态下的 HTTPS） | 可列出 refs |
| 模块结构完整 | 读 `backend/pom.xml` 的 `<modules>` | 6 个父模块，展开 27 个 Maven 模块 |
| 架构约束存在 | 读 `ArchitectureTest.java` | FF-1 ~ FF-6 共 6 条可执行规则 |
| 前端依赖声明 | 读 `frontend/package.json` | React 18 + Vite，脚本含 lint / typecheck / build |
| 构建可执行性 | `mvn` / `docker` 探测 | **不可执行**，见第 1 节 |

## 5. 如实说明

Lab1 的提交物里没有要求"必须在本机跑通全部测试"，但 S0 的验收要求"跑通环境"。本文档的作用是把"没跑通"这件事变成可交接的事实：谁在什么机器上、缺什么、补什么、怎么验证。等 JDK 21 + Maven 装好，第 3 节的命令与截图补齐后，本记录更新为 v2.0 并附 4 张验证截图（`docs/submissions/lab0-env-*.png`）。
