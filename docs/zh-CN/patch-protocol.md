# 补丁协议

[English](../patch-protocol.md) | [简体中文](patch-protocol.md) | [中文首页](../../README.zh-CN.md)

<!-- source-publication-version -->
> 配套源码已同步至 [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52)；[实际验收与边界](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions)。下方“未提交／未推送”仅保留验收时快照，不表示当前发布状态。


本文档展示源码实现；下文复现命令需在[公开源码仓库](https://github.com/Loz888/durable-coding-agent-runtime-source)中运行。本展示仓库不包含源码或自行执行软件 CI。

本文介绍本仓库实现中的补丁意图、写入与验证流程。

```text
prepared → applied → verified_pass / verified_fail

CurrentSHA == BeforeSHA → 经过权限校验后应用补丁
CurrentSHA == AfterSHA  → 恢复为已应用状态
其他情况               → 冲突，停止执行
```

## 准备补丁

Prepare 持久化路径、文件哈希、精确替换内容、准备时的 Epoch，
以及表示因果顺序的 round_number；在同一个事务中消耗一轮逻辑修复预算。

相同 Patch ID 与相同内容的重复请求保持幂等；
同一 ID 携带不同内容则产生冲突。
新的意图不能与 prepared、applied 或 verified_pass 状态的现有补丁重叠。
在 verified_fail 之后，下一次意图还需要剩余预算，
以及与前一个已知 AfterSHA 相符的文件状态。

## 受保护的应用

Store 锁定租约记录，并在取得锁之后重新检查 TTL。
ApplyGuarded 在 Rename 之前和结果提交之前分别校验执行权限。
已经持久化绑定的 Worktree 会拒绝旧式、不带保护的独立 Apply。

如果租约在 Rename 之后过期，旧持有者的结果提交会被拒绝，
prepared 意图仍保留，供下一位合法持有者恢复。

## 验证与结果缓存

VerifyGo 在长事务之外运行。
持久化测试判定前，再次验证租约、Worktree 绑定和当前文件哈希。

2026-10-21 未发布的本地实现还要求版本化 verification_input_digest 匹配受控候选全树、
测试输入与执行配置，才可复用缓存输出和退出码。摘要缺失或变化不能继承旧的通过判定；
重新验证需要有效执行授权。当前只支持标准库独立 Go 模块。
见[当前本地范围](validation.md#本地工程验收--2026-10-21未发布)。
旧数据表未持久化 DurationMS，因此缓存结果中的耗时不能当作新一轮测试的测量值。

## 身份边界

身份校验组合使用 Git 元数据中的随机令牌、基线、注册信息与路径检查。
它用于在可信写入者模型下防止意外路径复用；
不能防御拥有同等操作系统权限的恶意进程。
