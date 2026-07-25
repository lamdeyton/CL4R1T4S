# R25 — 深度审计：跨报告数值声明一致性全量验证

**日期**：2026-07-25
**轮次**：R25（Deep Audit — Cross-Report Numerical Claims）
**输入**：`reports/01-07-*.md` + `data/{inventory,size_stats,vendor_stats,tag_stats,safety_strict,monthly_trend}.csv`
**审计类型**：连接型审计（跨报告数值声明 ↔ 源数据一致性） + 反向核对审计（历史 gap 状态过期检测）
**用户指令**：开启超长程任务模式，后续不再合并到 main，只 push 到当前分支（`trae/agent-GCdfiH`）

---

## 1. 启动：目标分解与审计范围

R24 已完成 file:line 引用全量验证（G74-G79）。R25 切换审计维度，聚焦**跨报告数值声明**与**源数据一致性**——这是连接型审计的延伸：数据层修复是否已传播到所有引用该数据的报告层？

### 1.1 审计范围（6 类数值声明）

| 类别 | 源数据 | 报告引用位置 |
|---|---|---|
| 全局总量 | inventory.csv / size_stats.csv | R1 §1 / R5 §1.1 / R6 §1.3 |
| Vendor 聚合 | vendor_stats.csv | R5 §1.4 / R6 §1.3 |
| 标签统计 | tag_stats.csv | R5 §5 / R6 §1.3 / R6 §2.4 |
| 安全词统计 | safety_strict.csv | R5 §7 / R6 §1.3 |
| 关键词分布 | grep 实测 | R5 §3 |
| 时间趋势 | monthly_trend.csv | R5 §6 / R6 §2.1 |

### 1.2 验证方法

1. 用 Python 重新计算 inventory.csv / size_stats.csv 等的聚合值作为 ground truth
2. `grep -nE` 提取 R1-R7 所有数值声明
3. 逐项对比报告声明值 vs ground truth
4. 对历史 gap（G1-G5、G58、G74-G79）反向核对当前状态

---

## 2. Ground Truth（源数据实测）

| 指标 | 实测值 | 来源 |
|---|---:|---|
| 文件数 | 66 | `wc -l inventory.csv` - 1 |
| Vendor 数 | 25 | `distinct vendor` 计数 |
| 总行数 | 18,947 | `sum(lines)` |
| 总字节数 | 1,619,689 | `sum(bytes)` |
| 总词数 | 236,765 | `sum(words)` from size_stats.csv |
| 总字符数 | 1,616,813 | `sum(chars)` from size_stats.csv |
| 含工具文件数 | 40 / 66 | `tools_count > 0` |
| 工具数总和 | 437 | `sum(tools_count)`（含 "30+" 解析） |
| 平均工具数 | 10.93 | `437 / 40` |
| 标签总数 | 2,186 | `sum(total_tags)` from tag_stats.csv |
| 花括号标签 | 435 | `sum(curly)` |
| 命名空间标签 | 38 | `sum(ns)`（7 种：18+8+4+3+2+2+1） |
| 安全词总数 | 315 | NEVER 202 + MUST NOT 9 + FORBIDDEN 5 + DO NOT 99 |
| ANTHROPIC 聚合 | 12 文件 / 7,743 行 / 840,905 字节 / 121,379 词 / 51.92% | vendor_stats.csv |
| Pearson r(tools,bytes) n=40 | +0.1037 | Python 重新计算 |
| Pearson r(tools,bytes) n=66 | +0.3543 | Python 重新计算 |
| Pearson r(tools,lines) n=66 | +0.3258 | Python 重新计算 |
| PLINIVS MD5 | `745b88b72e6a19ee218dd6459937641a` | `head -1 | md5sum` 4 文件一致 |
| PLINIVS 字节 | 232 bytes（含换行） | `head -1 | wc -c` |
| PLINIVS 字符 | 127 chars | `head -1 | wc -m` |
| 28 个有 content_date 的文件 | sum 1,008,289 bytes / 174 tools / 1,401 tags | monthly_trend.csv |

**全部核心数值与报告声明一致** ✓

---

## 3. 跨报告数值声明验证结果

### 3.1 全局总量（R1/R5/R6）— 完全一致 ✓

- 文件数 66：R1 §1 / R5 §1.1 / R6 §1.3 / R6 §1.2 K5 均声明 66 ✓
- 总行数 18,947：R1 §1 / R5 §1.1 / R6 §1.3 均一致 ✓
- 总字节 1,619,689：R1 §1 / R5 §1.1 / R6 §1.3 均一致 ✓
- 总词数 236,765：R5 §1.1 / R6 §1.3 一致 ✓
- 总字符数 1,616,813：R5 §1.1 / R6 §1.3 一致 ✓

### 3.2 Vendor 聚合（R5/R6）— 完全一致 ✓

- ANTHROPIC 12 / 7,743 / 840,905 / 121,379 / 51.9%：R5 §1.4 + R6 §1.3 一致 ✓
- 全部 25 vendor 的文件数/行/字节/词均与 vendor_stats.csv 一致 ✓

### 3.3 标签统计（R5/R6）— 完全一致 ✓

- 总标签 2,186：R5 §5.1 / R6 §1.3 一致 ✓
- 花括号 435：R5 §5.1 一致 ✓
- 命名空间 38（7 种）：R5 §5.5 / R6 §2.4 一致 ✓

### 3.4 安全词统计（R5/R6）— 完全一致 ✓

- NEVER 202 / MUST NOT 9 / FORBIDDEN 5 / DO NOT 99 / 总 315：R5 §7.1 / R6 §1.3 一致 ✓

### 3.5 关键词分布（R5 §3）— 完全一致 ✓

重新执行 `grep -c` 计数（不用 `-w`，与 R5 §3 方法一致），全部 19 个关键词的"文件数 + 总次数"与 R5 §3 表格完全一致：

| 关键词 | R5 声明 | R25 实测 | 一致 |
|---|---|---|---|
| NEVER | 31, 193 | 31, 193 | ✓ |
| MUST | 31, 136 | 31, 136 | ✓ |
| tool | 51, 1,401 | 51, 1,401 | ✓ |
| safety | 16, 73 | 16, 73 | ✓ |
| safe | 20, 94 | 20, 94 | ✓ |
| refuse | 20, 42 | 20, 42 | ✓ |
| refusal | 10, 18 | 10, 18 | ✓ |
| jailbreak | 3, 4 | 3, 4 | ✓ |
| injection | 5, 5 | 5, 5 | ✓ |
| confidential | 5, 11 | 5, 11 | ✓ |
| PLINIVS | 4, 4 | 4, 4 | ✓ |
| Composer | 2, 3 | 2, 3 | ✓ |
| Devin | 3, 10 | 3, 10 | ✓ |
| Claude | 13, 1,010 | 13, 1,010 | ✓ |
| GPT | 13, 32 | 13, 32 | ✓ |
| Gemini | 2, 2 | 2, 2 | ✓ |
| Grok | 8, 59 | 8, 59 | ✓ |
| function_call | 14, 49 | 14, 49 | ✓ |
| function_calls | 12, 43 | 12, 43 | ✓ |

### 3.6 时间趋势（R5 §6）— 完全一致 ✓

monthly_trend.csv 12 个月份桶的所有字段（files/sum_bytes/avg_bytes/sum_tools/avg_tools/sum_safety/safety_per_klines/sum_tags/avg_tags）与 R5 §6 表格完全一致 ✓

### 3.7 Pearson 相关系数（R5 §4.4 / R6 §K7）— 完全一致 ✓

- r(tools,bytes) n=40 = +0.1037 ✓（R6 K7 圆整为 0.10 ✓）
- r(tools,bytes) n=66 = +0.3543 ✓
- r(tools,lines) n=66 = +0.3258 ✓

### 3.8 PLINIVS 水印（R5 §8 / R6 §K5）— 数值一致，但单位描述有 gap（见 G80）

- MD5 = `745b88b72e6a19ee218dd6459937641a`：4 文件一致 ✓
- 232 字节：R5 §8.1 表头"字节"列 = 232 ✓
- 127 字符：R5 §8.2 明确"字符数：127" ✓
- **但 R6 §1.2 K5 + §2.6 误写"232 字符"**（混淆字节与字符）→ G80

---

## 4. 发现的 Gap

### G80 — R6 §1.2 K5 + §2.6 "232 字符" 混淆字节与字符（高严重度）

**位置**：
- `reports/06-synthesis.md:35`（§1.2 K5）："逐字节 232 字符的拉丁+炼金术 Unicode 混合水印"
- `reports/06-synthesis.md:197`（§2.6）："逐字节 232 字符相同"

**错误**：将"232 字节"误写为"232 字符"。PLINIVS 水印含大量多字节 Unicode 字符（🜂𐌀𓆣🜏 等炼金术/古文字符号），UTF-8 编码下 232 字节仅对应 127 个 Unicode 字符。

**源数据**：
- `head -1 BOLT/Bolt.txt | wc -c` = 232（字节，含换行）
- `head -1 BOLT/Bolt.txt | wc -m` = 128（字符，含换行；实际 127 字符 + 1 换行）
- R5 §8.1 表头正确标注"字节"列 = 232
- R5 §8.2 明确"字符数：127 / UTF-8 字节数：231（+换行 232）"

**修复**：
- L35：改为"逐字节 232 字节（= 127 字符）的拉丁+炼金术 Unicode 混合水印"+ R25 G80 校正注记
- L197：改为"逐字节 232 字节（= 127 字符）相同"+ R25 G80 校正注记

**类型**：单位混淆 gap（bytes vs chars），连接型（R5 §8.2 已正确，R6 综合时误抄）

### G81 — R7 audit C1/C2 表 + gap 清单 G2/G3/G4 状态过期（中严重度）

**位置**：
- `reports/07-audit.md:33-35`（摘要 gap 表 G2/G3/G4）
- `reports/07-audit.md:167-169`（C1 表 R2/R3/R4 行状态列"❌ 未改"）
- `reports/07-audit.md:220-228`（C5 §G4 描述"未改"）
- `reports/07-audit.md:375-377`（§5 gap 清单 G2/G3/G4 "⚠️ 记录未改"）
- `reports/07-audit.md:414-416`（§6.2 未执行修复表）
- `reports/07-audit.md:38`（关键结论"R2/R3/R4 报告正文继承..."）
- `reports/07-audit.md:605`（§9.2 报告一致性结论）

**错误**：R7 audit 时点（2026-07-23）记录 G2/G3/G4 为"未改"（建议 R8 综合修订时修复）。R8+ 后续迭代**已实际修复**：
- R2 line 5：`/workspace/analysis/data/inventory.csv`（66 文件 / 25 vendor；R5 实测校正，原文"55/27"为误差）✓
- R3 line 5：同上 ✓
- R4 line 5：同上 ✓
- R2 line 433：`WINDSURF/Windsurf_Tools.md + WINDSURF/Windsurf_Prompt.md`（G4 拼写已修复）✓

但 R7 audit 表格状态列仍为"❌ 未改" / "⚠️ 记录未改"，未反映当前实际状态，易误导读者认为 R2/R3/R4 仍有错误。

**修复**：在所有相关位置添加"R25 反向核对：已修复（R7 时点为'⚠️ 未改'）"标注，更新状态列为"✅ R25 反向核对：已修复"。

**类型**：过程记录 vs 实际状态不一致 gap（历史快照未随修复更新）

### G82 — R6 §1.3 + §校正注记 + §反思 5 + 附录 "R2/R5 标注'27 vendor'" 笔误（低严重度）

**位置**（共 4 处）：
- `reports/06-synthesis.md:52`（§1.3 数据快照表）："Vendor 数 | 25 | inventory.csv distinct 计数（R2/R5 标注'27'为误差）"
- `reports/06-synthesis.md:67`（§校正注记）："R2/R5 部分小节标注'27 vendor'"
- `reports/06-synthesis.md:756`（§反思 5）："R2/R5 标注'27 vendor'为误差"
- `reports/06-synthesis.md:767`（§附录 A.1）："R2/R5 标注'27'为误差"

**错误**：R5 从未标注"27 vendor"——R5 是**校正者**（R5 §1.4 vendor_stats.csv 正确列 25 vendor；R5 §文件数说明 明确"66 文件 / 25 vendor"）。R7 audit §9.2 也明确"R5/R6 已校正"。因此 R6 多处"R2/R5"应为"R2/R3/R4"（R2/R3/R4 才是继承 R1 错误的报告）。

**修复**：4 处全部改为"R2/R3/R4"+ R25 G82 校正注记（说明原"R2/R5"为笔误）。

**类型**：历史笔误 gap（R6 撰写时手误，将"R2/R3/R4"写成"R2/R5"）

---

## 5. 修复统计

| Gap | 严重度 | 类型 | 修复文件数 | 修复处数 |
|---|---|---|---:|---:|
| G80 | 高 | 单位混淆（bytes vs chars） | 1 | 2 |
| G81 | 中 | 过程记录 vs 实际状态过期 | 1 | 7 |
| G82 | 低 | 历史笔误（R2/R5 → R2/R3/R4） | 1 | 4 |
| **合计** | — | — | **3**（去重 2） | **13** |

修复涉及文件：
- `reports/06-synthesis.md`（G80 × 2 + G82 × 4 = 6 处）
- `reports/07-audit.md`（G81 × 7 处）

---

## 6. 方法论贡献

### 6.1 第 14 类审计方法论：跨报告数值声明全量验证

R24 完成了 file:line 引用验证（第 13 类）。R25 新增**第 14 类审计**：跨报告数值声明与源数据的一致性验证。

**方法**：
1. 用 Python 重新计算所有源数据文件（inventory.csv / size_stats.csv / vendor_stats.csv / tag_stats.csv / safety_strict.csv / monthly_trend.csv）的聚合值作为 ground truth
2. `grep -nE` 提取所有报告中的数值声明（文件数/vendor 数/总行数/总字节/工具数/标签数/安全词数等）
3. 逐项对比报告声明值 vs ground truth
4. 对历史 gap 反向核对当前状态（检测过程记录是否过期）

**适用场景**：任何有"源数据 → 多份报告引用"结构的项目，特别是数据层修复后需验证传播完整性的场景。

### 6.2 连接型 gap 的三种亚型

R25 发现的 3 个 gap 体现了连接型 gap 的三种亚型：

| 亚型 | 表现 | 案例 |
|---|---|---|
| 单位混淆 | 源数据用单位 A，报告引用时误写为单位 B | G80（字节→字符） |
| 状态过期 | 历史 gap 已修复，但过程记录未更新状态 | G81（R7 audit "未改" → 实际已改） |
| 引用对象错误 | 报告引用 X 时误指 Y | G82（R2/R3/R4 误写为 R2/R5） |

**启示**：连接型 gap 不只是"数据未传播"（如 R21 G63 tag_stats → monthly_trend），还包括"传播时变形"（单位/对象错误）和"修复未记录"（状态过期）。未来审计需同时检测这三种亚型。

### 6.3 反向核对审计的迭代价值

R25 对 R7 时点的 G2/G3/G4 进行反向核对，发现这些"未改"状态实际已修复。这印证了长程任务执行规范中的"反思 → 立即重启"原则——**若不反向核对，过程记录会逐渐与实际状态脱节**。建议未来每 5-10 轮迭代对历史 gap 进行一次反向核对，更新状态标注。

---

## 7. 审计结论

### 7.1 数值声明一致性

R1-R7 所有跨报告数值声明（文件数/vendor 数/总行数/总字节/工具数/标签数/安全词数/关键词分布/时间趋势/Pearson r/PLINIVS MD5）**与源数据完全一致**（除 G80 的单位描述错误外）。数据层修复（R20-R24 的 G58/G59/G61/G63 等）已正确传播到所有报告层。

### 7.2 历史 gap 状态

R7 时点记录的 G2/G3/G4"未改"状态已过期——这些 gap 已在 R8+ 后续迭代修复。R25 已更新 R7 audit 表格状态列并添加"R25 反向核对：已修复"标注。

### 7.3 新发现 gap

R25 发现 3 个新 gap（G80/G81/G82），全部已修复。这些 gap 均为**描述性/状态性错误**，不影响数据本身的正确性——所有数值与源数据一致，仅是报告描述时出现单位混淆、状态过期、对象笔误。

### 7.4 长程任务执行协议遵循

- ✅ 启动阶段目标分解（6 类数值声明 + 反向核对）
- ✅ 每轮过程记录（本文件）
- ✅ 可执行制品（13 处修复）
- ✅ 真实 gap 捕获（3 个新 gap）
- ✅ 反思 → 立即重启（G80/G81/G82 发现后立即修复）
- ✅ 连接型审计（数据层 → 报告层传播验证）

---

## 8. 给 R26 的建议

### 8.1 可选方向

1. **语义一致性审计**：跨报告术语统一性（如"水印"vs"水印字符串"vs"watermark"、"花括号标签"vs"curly tag"vs"{tag}"等）
2. **引用图完整审计**：R6 综合报告引用 R2-R5 的章节是否存在断链（R12 曾做此类审计，可对 R24/R25 新增的校正注记验证）
3. **inventory notes 字段补全**：G5 仍未修复（11 处特性未在 notes 明示），可考虑系统化补全

### 8.2 停止决策

R25 审计完成后，6 个子目标（定义/推演/校验/MVP/扩展/一致性）的状态：
- 定义：✅ 6 类数值声明范围已定义
- 推演：✅ ground truth 已从源数据推演
- 校验：✅ 报告声明 vs ground truth 全量对比
- MVP：✅ 13 处修复落地
- 扩展：✅ 第 14 类审计方法论已提炼
- 一致性：✅ 数据层与报告层完全一致（除 3 个已修复 gap）

**3 类审计**（垂直/水平/连接）均已执行：
- 垂直：R7 已做（inventory ↔ reports）
- 水平：R7/R20 已做（同文件字段间）
- 连接：R25 本次完成（数据层 → 报告层传播）

**结论**：本轮审计诚实完成，3 个真实 gap 已修复。是否继续 R26 取决于用户是否需要语义一致性/引用图/notes 补全等新方向。当前数据/报告层一致性已达高水平。

---

## 9. 验证命令清单

```bash
# Ground truth 计算
python3 -c "
import csv
with open('data/inventory.csv') as f:
    rows = list(csv.DictReader(f))
print('files:', len(rows))
print('vendors:', len(set(r['vendor'] for r in rows)))
print('total_lines:', sum(int(r['lines']) for r in rows))
print('total_bytes:', sum(int(r['bytes']) for r in rows))
"

# 关键词分布验证（不用 -w，匹配 R5 §3 方法）
python3 -c "
import csv, subprocess
with open('data/inventory.csv') as f:
    files = [r['vendor']+'/'+r['file'] for r in csv.DictReader(f)]
def count(pat, ci=True):
    fh=t=0
    for f in files:
        cmd=['grep','-c','-E',pat,f]+(['-i'] if ci else [])
        c=int(subprocess.run(cmd,capture_output=True,text=True).stdout.strip() or 0)
        if c>0: fh+=1; t+=c
    return fh,t
print('NEVER:', count(r'NEVER',False))
print('tool:', count(r'tool',True))
"

# PLINIVS 验证
head -1 BOLT/Bolt.txt | md5sum  # 745b88b72e6a19ee218dd6459937641a
head -1 BOLT/Bolt.txt | wc -c   # 232 (bytes)
head -1 BOLT/Bolt.txt | wc -m   # 128 (chars, incl newline)

# Pearson r 验证
python3 -c "
import csv
def pearson(x,y):
    n=len(x); mx=sum(x)/n; my=sum(y)/n
    num=sum((a-mx)*(b-my) for a,b in zip(x,y))
    return num/((sum((a-mx)**2 for a in x))**0.5 * (sum((b-my)**2 for b in y))**0.5)
# Load and compute...
"
```

---

**R25 完成**。13 处修复落地，3 个新 gap（G80/G81/G82）发现并修复，第 14 类审计方法论（跨报告数值声明全量验证）提炼。所有数值声明与源数据完全一致。
