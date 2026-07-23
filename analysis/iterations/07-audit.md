# R7 — 三类审计（垂直 / 水平 / 连接）

**日期**：2026-07-23
**触发**：R6 完成后自我审计，"感到完成"时按 long-range-task-execution 规范必须审计而非停止。
**产出**：`reports/07-audit.md`（622 行，9 节）

## 执行的 3 类审计

### 1. 垂直审计（跨文件 ↔ inventory 一致性）
抽样 12 文件 60 个字段值，错误率 1.7%（仅 G1：Cursor_Prompt.md has_xml 应为 Y 而非 N）。

### 2. 水平审计（同文件内字段间一致性）
检查 inventory.csv 内字段间一致性，未发现新 bug。

### 3. 连接审计（R1-R6 报告间一致性）
发现 4 类 gap：
- G2: R2/R3/R4 声明"55 文件"，实际 66
- G3: R2/R3/R4 声明"27 vendor"，实际 25
- G4: R2 §Top 10 表 "Windsurf_Pools.md" 应为 "Windsurf_Tools.md"
- G5: inventory notes 11 处特性漏标（`<think>` / MCP / Computer Use）

### 4. 反向核对审计（声明但未强制的契约）
7 类特性标注一致性：PLINIVS/`<policy>`/Channels/Pop Quizzes 100% 一致；`<think>`/MCP/Computer Use 11 处漏标。

## 已立即修复

| ID | 修复 | 状态 |
|---|---|---|
| G1 | inventory.csv:41 Cursor_Prompt.md has_xml `N` → `Y`，notes 追加 `<user_query>` 标签说明 | ✅ 已修复 |

## 未自行修复（留 R8 处理）

| ID | 原因 |
|---|---|
| G2/G3 | 批量替换易破坏报告内其他引用链 |
| G4 | 单点拼写错误，R5/R6 已用正确名称 |
| G5 | notes 自由文本字段过度编辑风险 |

## 6 子目标核验

| 子目标 | 深度 | 证据 |
|---|---|---|
| 6. Consistency | 深 | 3 类审计 + 反向核对，揭示 5 gap 并立即修复 G1 |

## 下一轮（R8）触发条件

R7 留下 G2-G5 待修复。"反思而不重启 = 没反思" — 必须立即执行 R8 修复，而非停止。
