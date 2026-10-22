# 发布验证记录


<!-- source-publication-version -->
> 配套源码已同步至 [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52)；[实际验收与边界](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions)。下方“未提交／未推送”仅保留验收时快照，不表示当前发布状态。


<!-- runtime-followup:start -->
## 2026-10-22 续批更新

续批完整/race各407/0/0，离线入口206个Go事件+7个Python测试；新增12轮12003任务全部完成。真实上游种子只有预期失败/已知修复对照，模型调用0。时钟回退仍未修复。 详见配套源码本地 `reports/runtime-followup-20261022`；尚未推送，展示仓库无软件 CI。
<!-- runtime-followup:end -->

> 本轮已实测系统/数据库时钟同步倒退，候选原因和待授权对照已记录；未擅自改时间服务。

前批本地完整/race各338 PASS、关键恢复十轮220 PASS，均0 FAIL/0 SKIP；数据库无关153 PASS。真实Docker4场景与双容器接管通过。Live为12合成任务/11请求、3382/4201 Token、估价$0.0060558，非账单；Live绑定其实际较早二进制。物理双主机、断电、真实开源缺陷泛化、长期容量和历史时钟根因仍未验证。没有提交或推送。


## 2026-10-22 本地升级：实现与验收范围

本节是当前本地工作树；此前各日期的表格、BLOCKED 和“尚未实现”是历史快照。未提交或推送，本页不继承旧远端 CI。

| 方向 | 本轮交付 | 证据与边界 |
|---|---|---|
| 多文件补丁 | 同一逻辑轮次最多 8 个白名单 Go 文件，逐文件 before/after 摘要、全清单预检、统一验证 | 真实 Claim + 5 个 SIGKILL 窗口，含第一个 Rename 后的混合状态；多个 Rename 与 SQL 仍非原子操作 |
| 真实沙箱 | 固定镜像摘要、非 root、只读根、无网络、资源上限、超时；独立按身份清理孤儿 | 已运行真实 Docker；不是恶意内核逃逸证明，docker 组具有管理员级权限 |
| 模型恢复 | 请求意图、已观察响应与 Usage 持久化，恢复先复用响应；未知结果拒绝自动重发 | 3 个实际 SIGKILL 窗口；显式再授权最多 3 个逻辑尝试，物理费用预算另行约束，不保证 provider Exactly-once |
| 真实评测 | 冻结 12 项人工合成任务，High、deepseek-flash、单轮 | 11 次真实调用，3382 输入 / 4201 输出 Token；9 个新修复、1 个恢复对照通过独立断言，1 个不可修复对照按预期失败、1 个基线通过且零调用 |
| OTel | OTLP HTTP → Collector → Jaeger / Prometheus，耗时与错误指标、有界导出重试 | 实际读取 1 条新 Trace / 3 个 Span、指标和 firing 告警；受控错误，无外部联系人，SIGKILL 仍可能丢失缓冲数据 |
| 基准 | 真实 PG 固定等待、持续到达，以及真实 Worktree + Go test 负载 | 使用单调计时，保留数据库墙钟原值与异常计数；具体轮次见机器结果，不代表 LLM 吞吐或生产容量 |
| 可迁移 Worktree | 基线 commit + 有界全文件制品 + 持久化补丁链；新 Claim 下重建身份并重验 | 两个独立容器目录和第三个制品服务已验证；A SIGKILL、自然到期、B 真实 Epoch2 接管；只有一台物理电脑 |
| 输入与生命周期 | 显式 sealed-vendor / sealed-workspace 离线输入策略；终态 Worktree dry-run/受保护退役；数据库与角色恢复实验 | sealed 输入已完成 4 个真实沙箱场景；Workspace 限根内模块，不支持任意依赖图；数据库恢复只使用新建合成库 |

真实调用收到的型号字段为 `deepseek-flash`，这是服务端别名，不是不可变模型权重快照。2026-10-22 官方文档将它路由至 DeepSeek-V4.1-Flash-1017。峰时 cache-miss 上界估价 **$0.0060558**，不是供应商账单；本轮授权上限 $2，历史预算没有挪用。原始响应不公开推理内容或 Key。

仍未验证：第二台物理主机、网络分区/断电、任意不可信大型仓库、真实开源缺陷总体修复率、独立人工判定、长时生产压力/SLA，以及历史两条数据库负时长的根因。单调计时没有消除 Lease 对数据库墙钟的依赖。没有自动提交、推送或对外发布。

本仓库仅文档；完整可执行证据保存在配套源码本地 reports，尚未推送。


> 本文描述配套源码实现；命令在源码目录运行。展示仓库只有文档，不运行软件 CI。本次为本地对齐稿，尚未推送。

[English](../validation.md) | [简体中文](validation.md) | [中文首页](../../README.zh-CN.md)

## 历史系统评审快照 · 2026-10-20

全回归和 race 各 260 个测试/子测试事件通过，独立测试 138，通过且无 FAIL/SKIP；645c88e 的远端 PG、崩溃和 Capture 入口也通过。见[结果与范围](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/docs/review-20261020/RESULTS.md)、[本地记录](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/reports/review-20261020/verification.json)、[确切远端运行](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/reports/review-20261020/remote-ci.json)。本轮未重新测量覆盖率或付费模型质量，以下旧覆盖率不当作当前值。

## 历史发布快照 · 2026-10-17

2026-10-17 在独立发布目录执行验收，环境为 Linux/WSL、Go 1.27.1、
PostgreSQL 16。没有提供真实模型 API Key，只使用两个固定专用测试数据库。

| 验证项 | 通过条目 | FAIL | SKIP |
| --- | ---: | ---: | ---: |
| 全项目普通测试 | 226 | 0 | 0 |
| 全项目竞态测试 | 226 | 0 | 0 |
| 全项目覆盖率测试 | 226 | 0 | 0 |
| repair20 / durable / repair19 十轮回归 | 690 | 0 | 0 |
| 不依赖数据库的测试 | 133 | 0 | 0 |
| Mock Demo | 1 | 0 | 0 |
| 四状态矩阵与 Day20 崩溃 Demo | 6 | 0 | 0 |

条目数包含父测试、子测试与崩溃辅助入口。
格式、构建与 go vet 均通过。验收拒绝测试用例 SKIP；
没有测试文件的包不混算为被跳过的测试。

## Day21 复核

新采集的 12 个确定性试验严格评分全部通过。
已提交的 summary 与 Markdown 报告也逐字节重算一致。
原始场景、观测记录与 400/1403 ms 的历史 P50/P95 保持不变；
新采集延迟单独记录，不将它替换为通用基准。

新脚本输出到独立外部目录，不删除或覆盖已提交 JSONL。
场景、分母、计时边界与样本限制见[评测方法](evaluation.md)。

## 语句覆盖率

| 包 | 覆盖率 |
| --- | ---: |
| cmd/day21-eval | 55.6% |
| cmd/day8-server | 87.3% |
| distributed | 66.3% |
| durable | 81.2% |
| eval16 | 89.4% |
| eval21 | 94.9% |
| executor | 93.1% |
| httpapi | 58.2% |
| internal/testdb | 35.3% |
| observability | 93.9% |
| persistence | 44.9% |
| reliability15 | 85.7% |
| repair17 | 83.3% |
| repair18 | 82.5% |
| repair19 | 66.9% |
| repair20 | 39.2% |
| repodiag | 84.7% |
| runtimecore | 94.3% |

internal/integration20 只有测试，没有生产语句。
无单元测试的命令包显示零覆盖率或没有测试文件。
这些数值只描述本次记录，不代表性能或生产可靠性。

## 远程 CI 与边界

实际远程结果以 [Actions](https://github.com/Loz888/durable-coding-agent-runtime-source/actions) 为准。
CI 通过 TCP 连接 PostgreSQL 16，准备两个专用测试库，应用 001–009 迁移，
执行全量普通与竞态测试、真实 Mock/崩溃 Demo 和 Day21 采集与报告重算。
本地 WSL 使用对应 Unix Socket 连接，不能用本地通过冒充远程成功。

本仓库已公开，未选择软件 License。上表本地结果来自独立发布目录；
相同实现与测试文件通过文件同步进入公开仓库，没有导入私有 Git 历史。
原始开发工程保持只读；本地报告、原始日志、凭据和备份均保留在 Git 之外。
上方 Actions 链接展示本公开仓库自己的 CI 结果。

Day19 Rename、Day20 最终检查点前 SIGKILL、Day21 已验证结果接管属于不同实验；
不宣称沙箱、任意外部副作用恰好一次或跨主机 Worktree 恢复。

## 本地工程验收 · 2026-10-21（未发布）

配套源码在标准库独立 Go 模块范围内实现受控候选全树快照、版本化 `verification_input_digest` 与末次 Claim 后的受限收尾。历史判定缺少摘要时不能直接复用。收尾复核身份、完整输入、状态与 Checkpoint CAS，不增加执行 Claim/Epoch、修复轮数或模型次数；取消/Sweep 竞争和回滚有专项回归。

最新 Worker 启动修正后的最终源码完整回归/race 各为 **301 PASS、0 FAIL、0 SKIP**（含父/子测试事件）。核心五包十轮回归 **1,360 PASS、0 FAIL、0 SKIP**（`-p 1 -count=10 -timeout 1800s`；repair20、durable、repair19、internal/integration20、reliability15）。准确命令与最终源码指纹保存在配套源码本地 `docs/review-20261020/RESULTS.md` 和 `reports/review-20261020/engineering-20261021.json`。先停旧 Worker，应用迁移 **001–011**，再启动新 Worker；不承诺混用旧、新 Worker 的滚动升级。

本段是源码的本地验收，不是展示仓库软件 CI 或远端 CI。容器适配器已有代码，实际引擎隔离仍未验收；历史真实单样本、Day21 Mock 与旧统计保持各自日期、版本和分母。配套新增文件尚未推送，故不链接尚不存在的 GitHub 路径。

此前十轮回归使用 600 秒超时，600.715 秒后退出码为 1，记录 1,007 个通过事件并出现 durable 包失败，不能算验收通过。后续以 `-p 1 -timeout 1800s` 保留原五个包及全部十轮，在 804.374 秒完成、1,360 个事件通过。两份原始记录分开保留在 Git 外，没有减少测试场景或放宽断言。

## 本地扩展批次1 · 2026-10-21（尚未推送）

配套源码已新增可选OpenTelemetry SDK/本地NDJSON追踪、真实PostgreSQL合成吞吐量基准，以及冻结20场景Scripted提案/独立Oracle评测。最新全量/race各309 PASS、0 FAIL/SKIP；新增链路十轮race为90 PASS、纯本地入口147 PASS。计数含父/子测试与辅助入口，非场景种数。本展示仓库仍没有软件CI。

基准12轮共2400任务全部完成，失败/额外领取0；合成callback固定20ms，不代表模型吞吐或生产容量。20/20预期行为符合包含失败对照；17个可见测试通过，16个Oracle通过，1个预设过拟合被Oracle拒绝。新真实模型调用0。真实容器仍BLOCKED，多文件和跨主机尚未实现。证据在配套源码本地`reports/runtime-extension-20261021/verification.json`，未发布，故不提供尚不存在的远端链接。

启用追踪前应用源码001–012迁移。保留下面各有日期的301/260等历史快照；它们不覆盖本批新增代码。

最终基准的主要计时是同一进程Go单调时钟；2条数据库墙钟负值保留，根因未确认。桌面试验波动明显，不挑最好一轮，不解释为SLA。新建私有PG集群仅用于两个专用测试库，夹具资源已清理；原数据库不升级。本地沙箱修复仅证明候选目录可被非root UID读取；没有真实引擎安全验收。

## 剩余工作标注（2026-10-21）

截至2026-10-21，以下是**本地未发布版本**的状态；“基础已完成”不表示整个方向已完成。

| 工作项 | 状态 | 剩余内容 |
|---|---|---|
| 多文件补丁 | **尚未实现** | 多文件变更清单、逐文件摘要、部分写入后的恢复、统一验证和故障矩阵 |
| 沙箱隔离 | **已有适配代码，真实验收阻塞** | Docker真实隔离/资源/超时验收；Worker SIGKILL后的孤儿容器回收尚未完成 |
| 更广泛修复评测 | **离线基础已完成，待扩展** | 当前20个冻结合成场景使用脚本提案；真实仓库、真实模型、独立人工判定与费用统计仍待补 |
| OpenTelemetry | **本地追踪已完成，运维接入待补** | OTLP Collector/后端、Metrics、告警与导出故障验证；SIGKILL可丢缓冲span |
| 实测吞吐量 | **合成基准已完成，容量验收待补** | 两条数据库墙钟负时长根因未确认；真实工作负载、持续/突发压力、锁竞争和恢复耗时仍需验证 |
| 跨主机Worktree恢复 | **尚未实现，也未实测** | 可迁移制品、目标主机目录重建/身份验证及双主机接管；单机容器模拟不能代替双主机证据 |

详细验收门槛在配套源码本地`docs/pilot/IMPROVEMENT_OUTLINE.zh-CN.md`。本节不新增验收结果，不为尚未发布路径创建远端链接。
