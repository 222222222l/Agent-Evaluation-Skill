# Agent Evaluation：组织级协作评测附录

2026-11-23 · 评测规范草案，不代表新评分器已支持字段。核心仍为 **客观定量、硬门槛、人工偏好分层**。

## 为什么更新

多 Agent 运行可能“每个子 Agent 都答得不错，但总体任务变慢、变贵且丢消息”。因此需要以 **独立验收的业务工作项** 为分母，完整记录协调开销与失败，不应只比较局部文本分数。

## 基线与公平性

| 组别 | 机制 | 必须相同 |
|---|---|---|
| A | 单 Agent + 相同工具 | 冻结输入/业务真值、模型与 token 定价、权限、上下文、预算上限、截止时间 |
| B | 固定任务 DAG | 同上 |
| C | 共享板 + 定向消息 + 休眠/唤醒 | 同上 |
| D | 有限动态重规划 | 同上；仅在 C 发现调度瓶颈后使用 |

- 每种设置随机配对运行，按独立/串行/冲突/人工审批分层；原始失败、超时、拒绝、重复投递保留在分母。
- 主要指标：`accepted_goodput = independently_accepted_work_units / wall_clock_hours`；`full_cost_per_accepted_unit`（模型、工具、验证、协调、重试、人工、基础设施），0 个成功时成本/合格项标记未定义，并报告总支出。
- 辅助指标：P50/P95 完成时间、消息遗漏/重复、副作用重复次数、失联恢复率、首次验收通过率、返工次数、协调 Token /总 Token、人工处理时间。
- **硬门槛**：无授权写回、跨项目越权、陈旧证据被接受、不可恢复的遗失任务、验收者与提交者同一权限身份等失败不得被高平均分抵消。
- **人工层**：视觉审美、真实游戏趣味性、业务决策易读性保留人审；模型评审不能代替人类体验。

## 事件/轨迹证据最小字段

`runId, taskId, projectId, mode, runSeed, traceId, baseRevision, eventSeq, actorRole, recipient, eventKind, idempotencyKey, artifactHash, verifierRunId, acceptance, toolCost, tokenCost, retryCost, humanMinutes, elapsedMs, failureType`。

将字段映射为现有 JSON/JSONL 的扩展元数据，**不直接破坏**当前 `score_runs.py` 的兼容 schema；缺字段明确为缺失，不当作 0。先在 OrgOS 冻结夹具中验证汇总脚本的设计，再决定是否增添组织级 scorer。

## 验收计划

EV-SW0 锁定现有 self-test；EV-SW1 增加三种协作负例（消息遗漏、旧验收、重复副作用）及事件轨迹范例；EV-SW2 加入 A/B/C 配对报告模板；EV-SW3 再在保持兼容下实现自动评分。所有指标附样本数、方法、置信区间、负结果及适用范围。**机制的存在不是效率提升证据。**
