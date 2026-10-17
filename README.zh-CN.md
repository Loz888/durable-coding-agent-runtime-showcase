# Durable Coding Agent Runtime · 项目介绍

[English](README.md) | [简体中文](README.zh-CN.md)

一个使用 Go 构建的 Agent 工程项目，关注执行状态持久化、PostgreSQL 调度、
租约校验，以及受控代码修复中的崩溃恢复。

**本仓库提供设计文档与验证结果摘要。**
实现源码、测试、数据库迁移和可执行 Demo 已在[源码仓库](https://github.com/Loz888/durable-coding-agent-runtime-source)公开。
本仓库仍专注于文档阅读；运行说明见[源码仓库中文 README](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/README.zh-CN.md)。

## 项目解决什么问题

Worker 修改文件后，可能在记录完成状态之前崩溃。
接手的 Worker 需要确认之前已经发生了什么，重新取得执行权限，
并在不依赖上一个进程内存的情况下完成后续工作。

实现采用以下方式处理这个窗口：写文件前持久化 Patch Intent（补丁意图），
恢复时核对文件哈希，持有有效 Lease（租约）和 Epoch（执行代次）时才允许保存结果。
补丁完成与 Agent Run 完成是两个独立的持久化边界。

## 已实现的能力

下面介绍的是实现中已具备的能力：

- PostgreSQL 队列，支持按优先级和可用时间原子领取任务。
- 租约心跳、Worker Epoch 隔离校验，以及检查点 Revision CAS（版本比较并交换）。
- 在同一个现有 Scheduler 中调度诊断与代码修复两条路径。
- 使用 Git Worktree 隔离修改，并校验基线提交与工作区身份。
- 持久化 Patch Intent/Result，覆盖四种补丁状态的崩溃恢复。
- 分别限制领取次数、显式重试次数和逻辑修复轮次。
- 使用确定性的 Mock 提案，完成受控的单文件 Go 修复。
- 真实 PostgreSQL、并发持有者和自建子进程 SIGKILL 测试。

项目未声明未经测量的吞吐量或延迟指标。

## 系统架构

```mermaid
flowchart TD
    Submit[提交一次 Run] --> Queue[(PostgreSQL 队列)]
    Queue --> Worker[调度器 / Worker]
    Worker --> Lease[租约心跳与 Epoch 校验]
    Worker --> Diagnosis[诊断路径]
    Worker --> Coding[代码修复路径]
    Diagnosis --> Tools[模型决策与工具调用]
    Tools --> Checkpoint[检查点版本 CAS]
    Coding --> Proposer[Mock 或模型提案]
    Coding --> Patch[(补丁意图 / 结果)]
    Patch --> Worktree[受管理的 Git Worktree]
    Worktree --> Verify[运行 Go 测试]
    Verify --> Patch
    Coding --> Checkpoint
    Checkpoint --> Queue
```

## 阅读导航

| 文档 | 主要内容 |
| --- | --- |
| [系统架构](docs/zh-CN/architecture.md) | 组件职责、持久化边界与 Worktree 归属 |
| [执行生命周期](docs/zh-CN/execution-lifecycle.md) | 领取、心跳、检查点与重试流程 |
| [崩溃恢复](docs/zh-CN/crash-recovery.md) | Worker 退出后的接管与状态核对 |
| [补丁协议](docs/zh-CN/patch-protocol.md) | 补丁意图、文件哈希、受保护写入与缓存判定 |
| [设计取舍](docs/zh-CN/design-decisions.md) | 技术选择、代价与适用边界 |
| [测试策略](docs/zh-CN/testing.md) | 实现中的测试场景与验收规则 |
| [验证结果](docs/zh-CN/validation.md) | 已记录的结果及公开文档的验证范围 |

## 修复演示流程

以下文字描述了已在实现上执行的受控实验。
公开展示版使用文字流程介绍实验，不提供可执行的环境搭建步骤。

1. 创建临时 Git 仓库，放入单文件 Divide 函数的失败测试。
2. 提交一次 Run，通过真实队列领取操作取得任务。
3. 进入现有 Scheduler 执行路径，启动租约心跳并准备受管理的 Worktree。
4. 获取确定性的 Mock 提案，在应用补丁之前先持久化 Patch Intent。
5. 在 Worktree 中运行 Go 测试，保存验证结果。
6. 保存经过执行权限校验的 Agent 检查点，并验证原始仓库未被修改、
   已完成的 Run 不会再次被领取。

Mock 路径不需要真实模型 API Key。该实验记录的结果是修复场景通过；
可运行源码已单独公开；本文档仓库不分发原始执行输出。

## 崩溃恢复演示流程

```mermaid
sequenceDiagram
    participant A as Worker A / 自建子进程
    participant DB as PostgreSQL
    participant WT as 受管理的 Worktree
    participant B as Worker B
    A->>DB: 领取 Run，取得 Epoch
    A->>DB: 持久化 Patch Intent
    A->>WT: 应用补丁并运行测试
    A->>DB: 保存 verified_pass
    Note over A: 最终 Agent 检查点之前被 SIGKILL
    Note over DB: 测试仅推进自身 Run 的租约到期时间
    B->>DB: 通过真实队列领取，取得新 Epoch
    B->>DB: 恢复已持久化的补丁判定
    B->>DB: 通过权限校验与检查点 CAS，记录 succeeded
```

实现测试还分别在 prepared、applied、verified_pass 和 verified_fail 状态后，
终止测试自己创建的子进程。接管时的 Epoch 来自真实队列领取。
为了缩短等待，测试只推进自身 Run 的到期时间，不直接修改 Epoch 来模拟接管。

这些是有明确范围的集成测试场景，不能代替长期生产负载测试。

## 验证结果概览

实现的已记录验证环境为 Linux/WSL、Go 1.27.1 和 PostgreSQL 16，
未使用真实模型 API Key：

| 已记录的测试集 | 通过的测试条目 | FAIL | SKIP |
| --- | ---: | ---: | ---: |
| 全项目测试 | 207 | 0 | 0 |
| 全项目竞态检测 | 207 | 0 | 0 |
| 不依赖数据库的测试集 | 127 | 0 | 0 |

条目数包含父测试、子测试和子进程辅助测试入口。
当前软件 CI 状态见[源码仓库 Actions](https://github.com/Loz888/durable-coding-agent-runtime-source/actions)。
以上是实现工程的历史验证记录，并非这个文档仓库执行的 CI 结果；
仅凭本仓库无法独立复现这些软件测试。详见[验证结果说明](docs/zh-CN/validation.md)。

## 能力范围与限制

- 当前支持受控的单文件 Go 修复；任意多文件仓库修复属于后续工作。
- 仅适用于可信仓库：测试继承 Worker 的操作系统权限，尚未实现不可信代码沙箱。
- 文件系统 Rename 与 SQL Commit 仍是两个独立操作。
- 恢复依赖同一个有效的本地 Worktree；跨主机重建属于后续工作。
- 不保证任意外部副作用都恰好发生一次。
- 租约判断依赖数据库墙上时钟，需要合理设置 TTL。
- Worktree 身份校验以可信写入者模型为前提。

## 后续计划

多文件补丁、沙箱隔离、更广泛的修复评估、OpenTelemetry、
实测吞吐量基准和跨主机 Worktree 恢复，均属于后续工作。

实现源码见上方独立源码仓库；本项目尚未选择软件 License。
