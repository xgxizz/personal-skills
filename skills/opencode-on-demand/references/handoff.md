# 报告与长任务的交接约定

用于测试报告、批量评测、资料汇总等需要落盘或续作的任务。简单一次性问答不必创建这些文件。以下字段和目录是本技能的轻量约定，不是 OpenCode 原生 API。

## 请求

在任务专属目录保存 request.md（或等价 JSON），包含：

- task_id 与本轮 request_id；
- 目标、已确定的口径和相关输入路径；
- 执行工作目录、允许写入的目录与操作范围；
- 输出要求和必要自检；
- 超时/停止条件；
- 要求直接完成，不等待 Codex 对中间步骤逐个确认。

测试报告应说明只报告还是同时修复；默认不因发现失败而修改主流程。评测应约定样本、指标、耗时起止点、token 口径和失败如何计数；无法取得的指标标为 unavailable，不填成 0。这些口径在交接前确定，由 OpenCode 执行和汇报，Codex 不再独立重测。

长提示优先通过当前 CLI 确认支持的文件附件或输入方式传递；不要假定社区包装器的 stdin 支持等于本机 CLI 原生支持。直接传参时使用进程参数数组或可靠的原生 shell 引用，不能将任务文本当 shell 代码拼接。

## 返回

让 OpenCode 在任务目录写 result.json，并在最终回复给出简短摘要和该路径。推荐形状：

```json
{
  "task_id": "test-report-001",
  "request_id": "round-1",
  "status": "completed",
  "summary": "测试报告已生成，发现 2 项失败。",
  "artifacts": [{"path": "<绝对产物路径>", "kind": "report"}],
  "findings": {"passed": 18, "failed": 2},
  "checks": ["已记录命令、退出码和失败摘要"],
  "limitations": []
}
```

status 使用 completed、partial 或 failed，表示交付完成情况。测试失败数放在 findings 中，不能因此自动把交付判为 failed。无需强迫每个任务填充所有可选字段。

可额外使用 needs_context 与 blocked：前者返回具体缺失信息，调用方补充已有上下文后续接；后者返回阻碍原因及已完成部分。completed 可以附 limitations，无需为了消除所有顾虑自动复核。保持状态映射简单，不要求每次返回完整状态机。

长批次任务可增加 progress.md，写明输入路径及版本、已完成批次、剩余批次、产物位置和继续方式。执行者更新进度，调用方只用来恢复，不重读原材料核验。上游输入改变时标注哪些既有结果不再适用；输入未变则复用已完成批次。任务预算或超时将到时，优先保存可用部分并返回 partial，不无限探索，也不自行降低用户明确规定的样本数。

调用方记录 task_id、request_id、工作目录、会话 ID、实际模型、进程退出码和墙钟耗时；可获取时另记 token、缓存和供应商报告费用。不可获取的统计记为 unavailable，不为一次普通报告额外开启计费探测或新模型调用。

结果完整性只看本轮身份匹配、状态和必要交付项。不要使用共享 outbox 中上一轮文件，也不要因为某些可选指标不可用重做整项任务。partial 可直接向主流程传递已完成部分并标注缺失；仅在缺失妨碍本次交付时要求补齐。

## 日志与恢复

stdout/stderr 保存到本任务日志，优先读取最终结果。只有失败或交付缺失时，检索错误标记并读短尾部。没有结果时才按明确会话 ID 查询或导出；不读取全部历史。导出结构和统计字段按本机版本处理。

超时不等于取消；停止或确认原进程终止后再重试。保持同一任务的明确会话绑定；不要使用“最近会话”。源输入未变化、交付已完成时直接使用已有结果，不为提高置信度重新运行。

## 参考与取舍（2026-10-18 查阅）

- [delegate-agent](https://github.com/treygoff24/delegate-agent)：跨 CLI 的有界任务封装、关联工作合并、超时及结果记录。借鉴交接方式；不安装它，该项目未声称完成 Windows 测试，且 OpenCode 参数示例与本机 v2 有差异。
- [Codex 主导的 OpenCode 工作流](https://github.com/formiat/multi-agent-orchestration)：任务文件、会话映射和产物交接。借鉴持久化方式；其默认反复审查、只许复用已有会话及自动提交会话元数据均不采用。
- [OpenCode 会话与传输实现](https://github.com/formiat/multi-agent-orchestration/blob/main/codex/skills/.shared/delegated-agent/providers/opencode/session.md)：短结果优先、按 ID 续接、同会话串行。实现中的旧命令和 JSON 结构不可直接搬到本机。
- [threadle 跨工具交接](https://docs.threadle.sh/sessions/handoff/)：把交接上下文整理成目标、状态、决策、相关文件和下一步。这里只用轻量任务摘要，不引入其额外模型摘要、服务或工作流依赖。
- [OpenCode v2 官方命令](https://opencode.ai/v2/docs/cli/commands/)：非交互调用、JSON 输出、文件附件及 session export。具体支持仍以本机帮助为准。

这些是社区实现和设计经验，并非性能或准确率的独立实证。“合并关联工作、短结果回传”的选择用于降低本地调用和上下文开销；保留用户明确指定才调用、免费模型、直接采纳结果的偏好。

### 热门项目补充

以下为 2026-10-18 查阅时 GitHub 页面显示的约数，仅表示关注度，不代表性能、兼容性或准确率保证。只提炼设计，不安装这些框架。主体任务规范仍由本技能及用户指令决定。

| 项目与关注度 | 已查看的一手资料 | 本技能的取舍 |
|---|---|---|
| [Superpowers](https://github.com/obra/superpowers)，约 30 万星 | [委派工作流](https://github.com/obra/superpowers/blob/main/skills/subagent-driven-development/SKILL.md) | 借鉴精准上下文、完成/需上下文/阻塞的分流；忽略其默认独立审核与反复修复要求。角色和进度用于交付，不创建额外审查代理。 |
| [OmO / oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)，约 7 万星 | 项目 README；旧 oh-my-opencode 地址现重定向至此，当前主线已使用 Pi 分支运行时 | 借鉴按需加载、长工作保存记录和减少无效等待；不将当前 OmO 当成已安装 OpenCode 的兼容插件，不启用自动广泛委派或海量代理。 |
| [PAL MCP](https://github.com/BeehiveInnovations/pal-mcp-server)，约 1.18 万星 | [clink 文档](https://github.com/BeehiveInnovations/pal-mcp-server/blob/main/docs/tools/clink.md) | 借鉴外部 CLI 作为工具、角色提示、文件引用和续接 ID；仅回传结果。没有据此宣称其预置支持本机 OpenCode，也不复制其放宽权限参数。 |
| [GSD Core](https://github.com/open-gsd/gsd-core)，约 1.06 万星 | README 的独立执行上下文及 STATE/CONTEXT 产物设计 | 借鉴主上下文保持简短、长任务进度持久化与恢复；不引入其完整规划、验证、发布流程。旧 gsd-build/get-shit-done 仓库约 6.43 万星但已归档，本次按其指向查阅当前 GSD Core。 |

综合后采用的是“明确授权的单角色工具调用 + 结果文件 + 可续作进度”。这是一种针对本用户偏好的组合，不是以上项目共同认可的免审核标准，也没有经过新的耗时基准测试。
