# Personal Skills

个人 Codex Skill 集合。当前包含 `opencode-on-demand`：只在用户明确指定时，把有界旁支任务交给本机 OpenCode，并直接使用其结果。

## 使用规则

- 默认由 Codex 自己工作，只有用户明确指定才调用 OpenCode。
- 可在用户已配置且确认免费的模型之间自行选择；用户指定模型时遵循指定。
- OpenCode 整项执行并自检；Codex 默认不复核、不重复计算、不重复跑测试。
- 检查调用状态与交付完整性，保存结果及续接会话；失败有限重试。
- 适用于测试报告、评测、资料整理等可接受小范围误差的旁支流程。

## 安装

下载并解压 [`dist/opencode-on-demand.zip`](dist/opencode-on-demand.zip)，将其中 `opencode-on-demand` 文件夹放到自己的 Codex skills 目录（通常是 `~/.codex/skills/`）。已有同名 Skill 时先比较内容并保留自己的配置。

该 Skill 配置了 `allow_implicit_invocation: false`。为明确触发，可使用：

```text
$opencode-on-demand 请把这个项目的测试运行和报告整理交给 OpenCode，直接返回报告和失败项。
```

本机 OpenCode 必须已安装并配置好模型。Windows 示例路径来自 v2.0.32，其他安装位置和版本应按实际 CLI 帮助调整。本仓库不附带认证、模型密钥或自动安装程序。

## 文件

- [Skill 入口](skills/opencode-on-demand/SKILL.md)
- [任务交接约定及社区参考](skills/opencode-on-demand/references/handoff.md)
- [三轮评测报告](reports/codex-opencode-benchmark-2026-10-18.md)
- [逐题数据与抽样记录](reports/data/)
- [可直接下载的 Skill ZIP](dist/opencode-on-demand.zip)

评测支持的结论有限：本次短题和 HumanEval 随机 10 题中，免复核工具调用更快；不证明复杂项目能力，也不证明整个主聊天节省了同等比例的 token。
