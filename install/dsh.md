# ExMachina — DeepSeek Harness (DSH) 安装指令

> 用法：在 DSH 会话中对 Agent 说「读取 `<ExMachina仓库>/install/dsh.md` 并执行安装」，Agent 即可自主完成。

以下是给 Agent 的执行指令。

## 目标

把本仓库 `dsh/` 目录安装为 DSH 的 Agent 预设「ExMachina」。DSH 预设通过**文件复制**安装（持久，进程重启与切换会话都不失效），不使用受管理块。

## 步骤

1. **定位源**：本仓库根目录 = 本文件所在目录的上一级；安装源为 `<仓库根>/dsh/`，含三项：
   - `preset.yml` — 花名册元数据
   - `agent.cordis.yml` — 预设编排（standard 全量基座 + 指挥体 persona + 技能发现）
   - `skills/` — 6 个协议技能（dispatch / spec / debug / review / release / protocol）
2. **确定目标**：DSH 用户预设根目录 = `$HOME/.dsh/.agent-presets/`（Windows 即 `C:\Users\<用户名>\.dsh\.agent-presets\`；可用 `echo $env:DSH_HOME` 确认）。安装目标为 `<预设根>/exmachina/`。
3. **前置检查**：
   - 目标目录不存在 → 直接安装。
   - 已存在 → 向用户确认覆盖；覆盖前把原目录改名备份（如 `exmachina.bak-<日期>`），备份即回退路径。
   - 该目录在会话工作区之外：若文件写入被沙箱拒绝，用 `sandbox_permissions` 重试一次（需用户批准）。
4. **执行安装**：把 `dsh/preset.yml`、`dsh/agent.cordis.yml`、`dsh/skills/` 按原相对结构复制到 `<预设根>/exmachina/`。
   - `agent.cordis.yml` 中的 `skills/` 路径相对预设目录解析，复制必须保持目录结构。
   - `dsh/README.md` 是仓库文档，不需要复制进预设目录。
5. **挂载校验**：通过临时插件注入 `agentPresets` 服务并调用 `agentPresets.standingKeyFor('exmachina')`；正常返回即校验通过。
6. **汇报**：安装位置、覆盖还是新建、备份与回退方法。

## 使用与验收

新建会话，在预设选择器中选择「**ExMachina**」，确认：

- 工具列表包含 `subagent` / `subagent_fork` / `job_list` / `job_output`；
- 技能列表包含 `exmachina-dispatch` / `exmachina-spec` / `exmachina-debug` / `exmachina-review` / `exmachina-release` / `exmachina-protocol`；
- 系统提示包含「职责与分工」「并行与等待」两段。

建议验收流程：交给它一个多阶段开发任务（例如「给某模块加缓存并保证不回归」），观察是否先收拢边界、再按职能卡并行分派子代理、等待期照常受理新需求。

## 更新与卸载

- 更新：重新执行本安装（覆盖复制 `<仓库根>/dsh/` 内容）；新会话自动生效，已运行会话保持其启动时的编排。
- 卸载：删除 `<预设根>/exmachina/` 目录（含备份目录则一并删除）。

## 偏差处理

- **预设选择器里没有「ExMachina」**：确认 `skills/` 子目录已随行复制、挂载校验通过；预设清单即时扫描，无需重启。
- **预设目录布局或校验接口与实际不符**：以 DSH 当前文档为准，向用户报告差异后再继续，不得静默猜测。
- **想改协议**：编辑本仓库 `dsh/skills/*/SKILL.md` 后重新执行安装；不要在已安装目录内直接改，否则回退与更新会失配。
