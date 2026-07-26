# R39 深度审计：第29类——安全词汇计数复算与连接型gap修复

**日期**: 2025-07-26
**审计类型**: 第29类——安全词汇计数复算
**审计范围**: `data/safety_strict.csv` ↔ R5 §7 全局声明 ↔ 磁盘源文件实测 三方一致性核对

---

## 1. 审计目标

对 R5 §7 安全词汇密度统计进行全量复算，验证：
1. `data/safety_strict.csv` 中66个文件的 NEVER/MUST NOT/FORBIDDEN/DO NOT 计数与磁盘源文件实测一致
2. CSV 列求和与 R5 §7.1 全局汇总表声明一致
3. 计数方法严格遵循"整词匹配"原则，避免子串误匹配

---

## 2. 审计方法

### 2.1 计数口径对比

| 方法 | 命令 | NEVER 结果 | 说明 |
|---|---|---:|---|
| 子串匹配 | `grep -o "NEVER"` | 202 | 含 WHENEVER/HOWEVER 等复合词中的子串 |
| 单词边界 | `grep -o "\bNEVER\b"` | 198 | 因 shell 正则 \b 实现差异，结果不稳定 |
| **整词匹配（正确）** | `grep -ohw "NEVER"` | **201** | grep -w 保证整词匹配，结果可复现 |

### 2.2 三基准交叉验证

| 基准 | NEVER | DO NOT | MUST NOT | FORBIDDEN | 合计 |
|---|---:|---:|---:|---:|---:|
| 磁盘实测（grep -ohw） | 201 | 99 | 9 | 5 | 314 |
| R5 §7.1 声明 | 201 | 99 | 9 | 5 | 314 |
| safety_strict.csv 修正前 | 202 | 99 | 9 | 5 | 315 |
| safety_strict.csv 修正后 | 201 | 99 | 9 | 5 | 314 |

---

## 3. G121 发现：连接型gap——R28修复报告层但遗漏CSV数据层

### 3.1 问题描述

**根本原因**：R28 G97/G98 已经发现并正确定位了 NEVER 计数差异（WHENEVER 子串误匹配），修正了 R5 §7.1 全局汇总表中的数字（202→201, 315→314），但**未同步修正 `data/safety_strict.csv` 数据源**中对应文件的计数。

差异源：`ANTHROPIC/Claude_Opus_4.6.txt:908`
```
"USE THIS TOOL WHENEVER YOU HAVE A QUESTION FOR THE USER..."
```
- `WHENEVER` 包含 `NEVER` 子串，被 `grep -o "NEVER"` 误计为1次NEVER
- 该文件实际独立NEVER出现次数为6次（行193/355/362/366/773/777），但CSV记录为7次

### 3.2 修复内容

**文件**: `analysis/data/safety_strict.csv` 第9行

| 字段 | 修正前 | 修正后 |
|---|---:|---:|
| never | 7 | 6 |
| total | 8 | 7 |
| per_klines | 7.64 | 6.69 |

密度重算：7个安全词 / 1047行 × 1000 = 6.69 次/千行。

**文件**: `analysis/reports/05-quantitative.md` §7.1 方法论注记

补充 R39 G121 校正注记，记录：
1. R28 修复不完整的连接型gap性质
2. CSV数据层同步修正详情
3. 整词匹配原则提炼：§7安全词汇必须用 `grep -ohw`，避免复合词子串误匹配

### 3.3 Top 20 表格影响评估

Claude_Opus_4.6.txt 修正后密度 = 6.69 次/千行，远低于 Top 20 门槛（第20名 VERCEL V0/Vercel_v0.txt = 21.68），不进入 Top 20，§7.2 表格无需调整。

---

## 4. 其他安全词验证

| 安全词 | grep -ohw 实测 | CSV 求和 | R5 声明 | 一致性 |
|---|---:|---:|---:|---|
| DO NOT | 99 | 99 | 99 | ✓ |
| MUST NOT | 9 | 9 | 9 | ✓ |
| FORBIDDEN | 5 | 5 | 5 | ✓ |

DO NOT/MUST NOT/FORBIDDEN 均无复合词子串误匹配问题，所有计数与实测完全一致。

---

## 5. 方法论提炼：第29类审计——安全词汇计数复算

### 核心原则

1. **整词匹配原则**：安全词汇计数必须使用 `grep -ohw`（整词匹配），不能用 `grep -o`（子串匹配），否则 WHENEVER/HOWEVER/WHATEVER/NOTHING/NEVERTHELESS 等复合词会导致系统性高估
2. **连接审计原则**：报告层数字修复后，必须立即检查并同步修复所有引用该数据的下游数据源（CSV/表格/其他报告），避免"报告正确但数据错误"的连接型gap
3. **三基准验证框架**：
   - 基准1：磁盘源文件实测（`find + grep -ohw` 逐文件求和）
   - 基准2：CSV 数据层（列求和）
   - 基准3：报告层声明
   - 三者必须完全一致

### 本次审计的教训

R28 已经做了出色的根因定位（找到 WHENEVER 这个精确差异源），但修复只做了一半（改报告不改CSV）。这再次印证了长程任务执行中"连接型迭代"的重要性——两个独立正确的部分（报告数字正确、CSV数据大部分正确）之间的接口本身就值得一轮审计。

---

## 6. 修复文件清单

| 文件 | 修改类型 | 修改内容 |
|---|---|---|
| `analysis/data/safety_strict.csv` | 数据修正 | Claude_Opus_4.6.txt: never=7→6, total=8→7, per_klines=7.64→6.69 |
| `analysis/reports/05-quantitative.md` | 注记补充 | §7.1 方法论注记追加 R39 G121 校正说明 |
| `analysis/iterations/39-deep-audit.md` | 新增 | 本轮审计记录 |

---

## 7. 验证结果

修正后三基准完全一致：
- 磁盘 `grep -ohw NEVER` 逐文件求和 = 201 ✓
- safety_strict.csv NEVER 列求和 = 201 ✓
- R5 §7.1 NEVER 声明 = 201 ✓
- 总安全词数 = 314 ✓
