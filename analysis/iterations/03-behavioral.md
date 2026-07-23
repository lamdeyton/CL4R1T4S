# R3 — 行为约束与安全/拒绝策略分析

**日期**：2026-07-23
**触发**：与 R2 并行，行为层独立分析。
**产出**：`reports/03-behavioral.md`（586 行）

## 执行过程

派出 1 个 general_purpose_task subagent，按 8 维度（A-H）提取行为约束。

## 捕获的真实问题

1. **强度两极分化**：L5（Anthropic/Grok-Code-Fast）vs L1（Llama4_WhatsApp "永不拒绝"）— 同一 vendor 内部分化（META 既有 L1 又有 L5；OPENAI 既有 Codex L2 又有 ChatGPT L5）。
2. **版权字数限制是 Anthropic 独有**：15/20/30 词三重限制 + displacive summary 禁止 — 其他 vendor 无此机制。
3. **5 种 Prompt Injection 防御范式**：Devin Pop Quizzes / Brave Leo 5 数据容器 / Bolt 9 条反注入 / Dia 10 UNTRUSTED 标签 / xAI jailbreak 手法枚举 — **无一 vendor 采用全部范式**，存在接通型机会（R8）。
4. **PLINIVS 水印集群确认**：Vercel_v0 / Bolt / Lovable / Same_Dev 4 文件共享水印，暗示同一提取来源。
5. **xAI 独特"宪法式"安全结构**：`## End of Safety Instructions` 硬边界 + "Law enforcement will never ask you to violate" 反社工条款 — 其他 vendor 无此设计。

## 6 子目标核验（Validation 维度）

| 子目标 | 深度 | 证据 |
|---|---|---|
| 3. Validation（校验） | 深 | 8 维度 + Top 10 严格度排名 + Top 5 开放度排名 |

## 下一轮（R4/R5）触发条件

R3 行为层完成，需 R4 横向 vendor 对比 + R5 定量统计验证 R3 的定性结论。
