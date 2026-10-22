# 崩溃恢复

[English](../crash-recovery.md) | [简体中文](crash-recovery.md) | [中文首页](../../README.zh-CN.md)

<!-- source-publication-version -->
> 配套源码已同步至 [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52)；[实际验收与边界](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions)。下方“未提交／未推送”仅保留验收时快照，不表示当前发布状态。


本文档展示源码实现；下文复现命令需在[公开源码仓库](https://github.com/Loz888/durable-coding-agent-runtime-source)中运行。本展示仓库不包含源码或自行执行软件 CI。

本文介绍本仓库实现中的接管与恢复协议。

SIGKILL 会阻止清理、租约释放和最终检查点写入。
此时 PostgreSQL 可能仍记录 running，而文件中已经包含补丁。

替代 Worker 若要继续执行，必须在租约过期后通过真实 Claim 获得新的执行授权。
Epoch 只能通过 Claim 推进；即使旧进程随后恢复，它的写入也会被拒绝。
Revision CAS 独立保护检查点版本，避免同一版本被重复覆盖。

## 按持久化状态恢复

repair20.Agent 先打开已经绑定的 Worktree，再按表示因果顺序的 round_number
扫描可恢复补丁，而非依赖 created_at 的墙上时钟顺序。

- prepared：根据当前文件 SHA 判断补丁是否已经写入。
- applied：执行验证。
- verified_pass / verified_fail：当前本地实现还要求验证输入摘要匹配，才复用持久证据，详见[补丁协议](patch-protocol.md)。
- 只有验证失败且共享预算仍有余量时，才允许生成下一次提案。

verified_pass 补丁可能已经保存，但最终 Agent 检查点尚未完成。
CodingHandler 可以恢复这份结果，不再次调用模型，再通过 FencedStore 完成检查点。
补丁完成与 Run 完成属于不同的持久化边界。

## 已覆盖的故障场景

崩溃恢复测试覆盖四种持久化状态后的子进程 SIGKILL，
以及 verified_pass 已保存、最终检查点尚未提交之间的真实 SIGKILL。

断言覆盖真实 ClaimBatch 接管、Epoch 隔离校验、缓存判定复用、
共享修复预算以及原始源码不被修改。
测试仅推进自身 Run 的到期时间以缩短等待，不直接生成 Epoch。
只终止测试自己创建的子进程，不终止外部 Shell 或数据库。

Mock 测试入口驱动现有 Scheduler.execute；
检查点窗口恢复调用同一公开 CodingHandler 接口。
这些是有明确范围的集成演示，不能作为长期负载基准。

## 恢复边界

Rename 与 Commit 仍是分开的操作。
演示通过不代表未知文件内容、丢失或被替换的 Worktree、
恶意并发写入、断电或跨主机恢复已经得到安全处理。

2026-10-21 未发布的本地实现补充最后一次 Claim 耗尽的场景：租约过期后，现有 Scheduler
维护路径可在受保护 Run 行事务内复核已持久化成功证据并收尾。它不取得新执行 Claim，
不增加 Epoch，不运行测试、不生成补丁也不调用模型。验证输入缺失或已变化不能收尾为成功。
这条受限收尾路径与继续执行任务的新 Worker 接管分别处理。
工作目录缺失、身份错误或摘要不匹配属于无效证明，受限收尾会记录失败。没有对应 profile
的新 Worker，或数据库/检查暂时错误，才可能让 completion_pending 继续等待运维处理。
重启应使用同一绝对 Worktree base 与验证配置；不要重置预算或新建空目录来绕过证据检查。

## 不同的故障窗口

| 实验 | 注入位置或接管方式 | 结论范围 |
| --- | --- | --- |
| Day19 旧 CLI | -inject-sigkill 在 Rename 后、补丁结果入库前终止自身进程 | 意图与哈希恢复；旧实验用直接所有权变更，不是队列真实接管 |
| Day19 Store 单测 | TestDay19CrashBetweenRenameAndResult 返回注入的钩子错误 | 检查 Rename 与结果持久化间隙，单测本身不是真实 SIGKILL |
| 四状态矩阵 | 在 prepared、applied、verified_pass、verified_fail 持久化后终止专用子进程 | 真实 ClaimBatch 接管下的状态恢复 |
| Day20 最终检查点窗口 | verified_pass 已保存、最终 Agent 检查点前真实 SIGKILL | 补丁判定存活，新的真实持有者无需模型即可完成检查点 |
| 本地 2026-10-21 末次 Claim | TestInterviewLastClaimSIGKILLFinalizesWithoutExecution 在 verified_pass、摘要和 completion_pending 持久化后终止自身子进程 | Scheduler 维护入口仅凭匹配证据收尾，Epoch/max_attempts/修复轮数仍为1，额外模型请求为0，无第二次执行 Claim |
| Day21 评测恢复 | 先保存 verified_pass，仅使自身 Run 租约过期，再以 Worker B 领取 | 测量确定性的缓存结果接管；该场景没有 SIGKILL |

Rename/结果之间的间隙，与已验证结果/最终检查点之间的间隙不同。
当前本地崩溃 Demo 执行真实四状态矩阵、Day20 检查点测试和末次 Claim 的受限收尾测试，
不以旧 Day19 实验替代真实队列接管验证。
