# R10 — 连接型迭代：bytes 补全的传播修复

**日期**：2026-07-23
**触发**：R9 修复了 inventory.csv 的 36 个空 bytes（G6），但修复未传播到引用 inventory 数据的报告层。连接型迭代：数据层修复 → 报告层必须同步。
**产出**：02-structural.md Top 10 表 bytes 回填 + 01-inventory.md 36 条 bytes 回填 + 本记录

## 执行的修复

### G12（中严重度，已修复）— R2 Top 10 复杂度表 5 条 bytes 为空

发现：R2 §Top 10 表中 5 个条目 bytes 列显示 "—"，因 R2 时 inventory.csv 这些文件 bytes 为空。

| 排名 | 文件 | 修复前 | 修复后 |
|---|---|---|---|
| 2 | DEVIN/Devin2_09-08-2025.md | — | 50815 |
| 7 | CLINE/Cline.md | — | 47221 |
| 8 | MANUS/Manus_Prompt + Manus_Functions | — | 39922 |
| 9 | LOVABLE/Lovable_2.0.txt | — | 16704 |
| 10 | WINDSURF/Windsurf_Tools + Windsurf_Prompt | — | 33760 |

### G13（中严重度，已修复）— 01-inventory.md 36 条 bytes 为空

发现：01-inventory.md 的人类可读清单中 36 个文件的 bytes 列显示 "—"，对应 inventory.csv 的 36 个空 bytes。涉及两种表格格式：
- 7 列格式（含日期列）：ANTHROPIC/OPENAI/XAI/MOONSHOT/DEVIN/SAMEDEV/FACTORY/DIA/LOVABLE/VERCEL V0/PERPLEXITY/MISTRAL/BRAVE/MINIMAX — 21 条
- 6 列格式（无日期列）：CURSOR/WINDSURF/CLINE/REPLIT/MANUS/BOLT/MULTION/HUME/CLUELY — 15 条

修复：用 Python 脚本从 inventory.csv 读取 bytes 值，按 vendor+file 匹配回填。首批脚本仅匹配 7 列格式（len(parts)>=9），遗漏 15 条 6 列格式条目；修正条件为 len(parts)>=8 后全部修复。

验证：Python 脚本扫描确认 01-inventory.md 中无剩余空 bytes。

## 连接型迭代的意义

R9 修复了数据层（inventory.csv），但两个报告层（02-structural.md Top 10 表、01-inventory.md 清单）仍引用旧的空值。这是典型的"连接型 gap"——两个独立完成的层（数据 vs 报告）的接口未同步。

**教训**：数据层修复后，必须 grep 所有引用该字段的报告层，确保传播完整。R9 仅验证了 data/ 目录内的一致性（inventory ↔ size_stats ↔ vendor_stats），未检查 reports/ 目录中对 bytes 的引用。

## 修复汇总

| ID | 严重度 | 描述 | 状态 |
|---|---|---|---|
| G12 | 中 | R2 Top 10 表 5 条 bytes 为空 | ✅ 已修复 |
| G13 | 中 | 01-inventory.md 36 条 bytes 为空 | ✅ 已修复 |

## 6 子目标核验

| 子目标 | 深度 | 变化 |
|---|---|---|
| 1. Definition | 深 | G13 修复使 01-inventory.md 从 45% bytes 完整 → 100% |
| 6. Consistency | 深 | 数据层与报告层 bytes 完全同步，零 "—" 残留 |

## 新发现（bytes 补全后暴露的定量洞察）

bytes/line 密度比（R5 未分析，因 inventory.csv bytes 不完整）：
- **Replit_Functions.md**：10,393 bytes/line（极端离群值——2 行单行 JSON schema，20,786 字节）
- 排除 Replit 后范围：0.6-268 bytes/line
- **高密度（>150）**：UserStyle_Modes(268)、GROK-4.20(182)、Claude_4(175)、Claude-Design(174)、Muse_Spark(173)——信息密集型 prompt
- **低密度（<50）**：ChatKit_Docs(43)、ChatGPT_4o(65)——对话式、注释多

此洞察可作为后续分析的种子，本轮不展开（避免 scope creep）。
