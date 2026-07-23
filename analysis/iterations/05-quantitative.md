# R5 — 定量文本分析

**日期**：2026-07-23
**触发**：R4 完成，需定量验证定性结论。
**产出**：`reports/05-quantitative.md`（586 行）+ 6 个中间数据文件存 `data/`

## 执行过程

派出 1 个 general_purpose_task subagent，强制要求用 shell 命令实际计算。

## 捕获的真实问题（关键！）

1. **R1 文件数计数 bug**（已修复）— subagent 通过 `find -type f | wc -l` 实际算出 **66 文件**，与 R1 迭代记录中"55 文件"声明冲突。主 agent 验证确认 66 为真。已修正 `iterations/01-inventory.md`。**这是 R5 的最大价值** — 防止错误数字继承到 R6-R8。
2. **命令替换陷阱** — `$(cat ...)` 会截断大内容导致 grep 计数为 0，subagent 自动改用直接 `grep -rioh` 跨 vendor 目录。
3. **工作区 grep 误计** — 全工作区 grep 会误计入 `analysis/` 报告自身，subagent 已限定 vendor 目录。

## 关键定量发现

1. **总量**：66 文件 / 18,947 行 / 1,619,689 字节(1.58MB) / 236,765 词
2. **ANTHROPIC 主导**：12 文件独占 51.9% 字节
3. **Top 5 高频词**：user(2,590) → claude(2,205) → search(1,616) → tool(1,524) → type(1,431)
4. **`do not`(668) 远超 `must not`(20)/`forbidden`(8)** — 厂商偏好软约束
5. **`jailbreak` 仅 3 文件且全为 XAI**；`injection` 仅 5 文件（4 Anthropic）
6. **PLINIVS 水印**：4 文件第 1 行逐字节一致，MD5 `745b88b72e6a19ee218dd6459937641a`，127 字符/231 字节
7. **工具数 vs 文件大小 Pearson r=+0.10** — 几乎无相关（推翻"工具多文件就大"直觉）
8. **安全严格度 Top 3**：Cursor_Prompt(166.67/千行) / Windsurf(125) / Bolt(98.41)
9. **时间趋势**：平均字节 2024-06→2026-06 增长 5.3×；安全词密度 2025-06 峰值 44.25 → 2026-06 降至 9.39（后期转向长篇 contextual 约束）

## 6 子目标核验（Derivation + Validation）

| 子目标 | 深度 | 证据 |
|---|---|---|
| 2. Derivation | 深 | 工具数 vs 字节相关性、时间趋势、词频分布 |
| 3. Validation | 深 | shell 命令实际计算，所有数字可复现 |

## 下一轮（R6）触发条件

R1-R5 全部完成，需 R6 综合：trends / insights / recommendations。
