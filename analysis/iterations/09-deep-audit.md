# R9 — 深度审计与数据完整性修复

**日期**：2026-07-23
**触发**：用户要求"继续复盘，深度优化迭代"。R8 声称"可以诚实停止"，但按 long-range-task-execution 规范"复盘而不重启 = 没复盘"，必须立即执行审计+修复。
**产出**：inventory.csv bytes 补全 + 06-synthesis.md 残留修正 + 05-quantitative.md 方法论注记 + 本记录

## 执行的三类审计

### 1. 垂直审计（跨文件一致性）

**G6（高严重度，已修复）— inventory.csv bytes 字段不完整**

发现：inventory.csv 有 36/66 文件 bytes 字段为空（R1 subagent 无 shell 用 `≈` 估算，主 agent 仅用 `du -b` 填充了 30 个文件）。导致：
- inventory.csv bytes 总和 = 1,168,079（缺 36 文件）
- size_stats.csv bytes 总和 = 1,619,689（全 66 文件，正确）
- 差异：451,610 字节（28% 数据缺失）

修复：从 size_stats.csv 提取 66 文件的 bytes 值，用 awk 两步 join 填充 inventory.csv 的 36 个空 bytes 字段。

验证：
- 修复后 inventory.csv bytes 总和 = 1,619,689 ✅（与 size_stats.csv 一致）
- 空 bytes 字段数 = 0 ✅
- vendor_stats.csv 与 inventory.csv 派生值 diff = 空 ✅（G7 一并修复）

**G7（中严重度，已修复）— vendor_stats.csv 与 inventory.csv 派生值不匹配**

发现：vendor_stats.csv 的 bytes 聚合（正确，来自 size_stats.csv）与 inventory.csv 派生的 vendor bytes 聚合不匹配——36 个文件的空 bytes 导致 17 个 vendor 的派生 bytes = 0。

修复：G6 修复后自动解决。验证 diff = 空。

### 2. 连接审计（报告间一致性）

**G8（中严重度，已修复）— 06-synthesis.md "27 vendor" 残留**

发现：R8 声称修复了 R2/R3/R4/R5 的"27 vendor"声明，但 06-synthesis.md 仍有 2 处残留：
- line 634："R5 §1.4 显示 27 vendor 中已有 25 个" — 自相矛盾（27 是误差值，25 才是真实值）
- line 515："仅扫描 27 vendor 目录" — 应为 25

修复：
- line 634 → "当前数据集覆盖 25 个 vendor。"
- line 515 → "仅扫描 25 vendor 目录"

**G9（低严重度，已记录非 bug）— monthly_trend.csv 仅覆盖 28/66 文件**

发现：monthly_trend.csv sum_bytes=1,008,289（仅 28 文件），而非 1,619,689（66 文件）。

验证：inventory.csv 中仅 28 个文件有 content_date，24 个文件完全无日期。R5 line 413 已明确记录"28 个文件有已知日期"。**已记录的限制，非 bug**。

### 3. 定量声明验证审计

**PLINIVS MD5 验证 ✅**
- 4 文件首行 MD5 = `745b88b72e6a19ee218dd6459937641a` — 与报告声明完全一致

**安全词频验证 ✅**
- `do not` = 668（grep -ioh）— 与 R5 §2.3 声明一致
- `must not` = 20、`forbidden` = 8 — 均一致

**ANTHROPIC 字节占比验证 ✅**
- 840,905 / 1,619,689 = 51.9% — 与报告声明一致

**G10（中严重度，已文档化）— Pearson r 方法论不透明**

发现：R5 §4.4 声称"工具数 vs 文件大小 Pearson r=+0.10"，但：
- n=40（仅 tools_count > 0 的文件）→ r=+0.1037 ✅（与声明一致）
- n=66（全部文件）→ r=+0.3543（弱-中正相关）

报告未说明 n=40 的含义及排除标准。这不是计算错误，而是方法论透明度 gap。

修复：在 R5 §4.4 添加"方法论注记（R9 补充）"，说明 n=40/n=66 的区别及选择理由。同步在 06-synthesis.md §3.6 添加 R9 注记指针。

**G11（低严重度，已文档化）— safety_strict.csv 与 §2.3 计数方法差异未文档化**

发现：safety_strict.csv（§7）与 word_freq（§2.3）的计数方法不同：
- §2.3：`grep -ioh`（case-insensitive，按出现次数计数）→ `do not`=668、`never`=584
- §7：`grep -c`（case-sensitive 大写，按匹配行数计数）→ `DO NOT`=99、`NEVER`=202

两者回答不同问题（全局语料词频 vs 命令式硬约束语气强度），但报告未明确说明方法差异。

修复：在 R5 §7.1 表后添加"方法论注记（R9 补充）"，说明两节计数方法的区别。

## 修复汇总

| ID | 严重度 | 描述 | 状态 |
|---|---|---|---|
| G6 | 高 | inventory.csv 36 文件 bytes 为空 | ✅ 已修复 |
| G7 | 中 | vendor_stats 与 inventory 派生值不匹配 | ✅ 随 G6 修复 |
| G8 | 中 | 06-synthesis.md "27 vendor" 残留 2 处 | ✅ 已修复 |
| G9 | 低 | monthly_trend 仅覆盖 28/66 文件 | ⚠️ 已记录限制 |
| G10 | 中 | Pearson r 方法论不透明 | ✅ 已文档化 |
| G11 | 低 | safety_strict vs §2.3 方法差异 | ✅ 已文档化 |

## 数据一致性最终状态

| 指标 | inventory.csv | size_stats.csv | vendor_stats.csv | 一致 |
|---|---:|---:|---:|---|
| 总字节 | 1,619,689 | 1,619,689 | 1,619,689 | ✅ |
| 总行数 | 18,947 | 18,947 | 18,947 | ✅ |
| 总词数 | — | 236,765 | 236,765 | ✅ |
| 文件数 | 66 | 66 | 66 | ✅ |
| vendor 数 | 25 | — | 25 | ✅ |

## 6 子目标核验

| 子目标 | 深度 | 变化 |
|---|---|---|
| 1. Definition | 深 | G6 修复使 inventory.csv bytes 从 55% 完整 → 100% 完整 |
| 2. Derivation | 深 | G10 文档化使 Pearson r 方法论透明 |
| 3. Validation | 深 | 本轮验证了 PLINIVS MD5、安全词频、字节占比、Pearson r 四项定量声明 |
| 6. Consistency | 深 | G6-G8 修复后，所有数据文件与报告数字完全一致 |

## 下一轮（R10）触发条件

R9 修复了数据完整性 gap，但未深化分析本身。R10 应做连接型迭代：
- 检查 inventory.csv bytes 补全后是否有新的定量洞察
- 验证 R2 Top 10 复杂度排名是否与 size_stats.csv 一致
- 检查是否有其他报告间的分析缺口
