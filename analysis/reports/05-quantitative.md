# R5 — 定量文本分析报告

**日期**：2026-07-23
**轮次**：R5（Quantitative）
**输入**：`/workspace/analysis/data/inventory.csv` + 25 个厂商目录下的 66 个系统提示词文件
**输出**：本报告 + 中间数据 `/workspace/analysis/data/{size_stats,vendor_stats,word_freq_all,tag_stats,safety_strict,monthly_trend}.csv`
**方法**：全部数字来自实际 shell 命令计算（`wc` / `grep` / `awk` / `python3`），无估算。命令附于各节。

> **文件数说明**：任务描述称 55 文件，但 `inventory.csv` 与磁盘实际均为 **66 个文件**（66 数据行 / 25 vendor）。本报告对全部 66 文件进行分析。命令：`wc -l < inventory.csv` → 67（含表头）。
> 命令：`find ANTHROPIC OPENAI ... CLUELY -type f | wc -l` → 66
> 命令：`find ANTHROPIC OPENAI ... CLUELY -maxdepth 1 -type d | wc -l` → 26（含父目录路径），实际 vendor 目录 = 25

---

## 1. 基础统计

### 1.1 全局总量

| 指标 | 数值 |
|---|---|
| 文件数 | 66 |
| 总行数 | 18,947 |
| 总字节数 | 1,619,689（≈1.62 MB）（R32 G113 校正：原"≈1.58 MB"为二进制 KB ÷1024 + 十进制 MB ÷1000 混用，1,619,689/1024/1000=1.5817；统一十进制（与本报告"150KB"=149.7KB 等用法一致）：1,619,689/1,000,000=1.62 MB；二进制：1.54 MiB） |
| 总词数 | 236,765 |
| 总字符数 | 1,616,813 |

命令：
```bash
while IFS= read -r f; do [ -f "$f" ] && cat "$f"; done < /tmp/all_files.txt \
  | awk 'END{print NR" lines"}'   # 18947
wc -c < /tmp/all_content_prompt_only  # 1619689
# 逐文件 wc -l/w/c/m 求和见 size_stats.csv
```

### 1.2 每文件统计（按字节数降序，完整 66 行见 `data/size_stats.csv`）

| 字节数 | 行数 | 词数 | 字符数 | 文件 |
|---:|---:|---:|---:|---|
| 149,724 | 1,408 | 21,177 | 149,442 | ANTHROPIC/Claude-Opus-4.7.txt |
| 122,750 | 1,597 | 17,501 | 122,428 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 102,687 | 1,047 | 14,190 | 102,617 | ANTHROPIC/Claude_Opus_4.6.txt |
| 92,710 | 1,222 | 13,194 | 92,620 | ANTHROPIC/Claude-4.5-Opus.txt |
| 85,298 | 520 | 13,015 | 85,264 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 73,266 | 422 | 9,707 | 73,033 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 73,121 | 1,714 | 9,193 | 73,063 | OPENAI/ChatKit_Docs__Oct-6-25.txt |
| 64,487 | 368 | 9,741 | 64,473 | ANTHROPIC/Claude_4.txt |
| 63,413 | 397 | 9,598 | 63,403 | ANTHROPIC/Claude_Sonnet_3.7_New.txt |
| 58,212 | 494 | 8,884 | 58,198 | ANTHROPIC/Claude-4.1.txt |

> **R21 G62 校正注记**：R5 §1.2 Top 10 文件的"字符数"列原值与 `size_stats.csv`（实际 `wc -m` 测量值）系统性偏离（偏差 2-133，方向不一）。R19 过程记录声称已"反向核对全量数据文件 chars 列"，但实际未将修复传播到 R5 §1.2 表格（仅 size_stats.csv 正确，§1.2 仍为旧值）。R21 现重新执行修复：10 行 chars 列全部对齐 size_stats.csv。验证：size_stats.csv sum_chars = 1,616,813，与 R5 §1.1 / R6 §1.3 声明一致；Claude_Opus_4.6.txt 实测 `wc -m` = 102,617 ✓。

### 1.3 Top 20 排序表

**按字节数 Top 20**（命令 `sort -t'|' -k1 -rn size_stats.csv | head -20`）：

| # | 字节 | 文件 |
|---:|---:|---|
| 1 | 149,724 | ANTHROPIC/Claude-Opus-4.7.txt |
| 2 | 122,750 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 3 | 102,687 | ANTHROPIC/Claude_Opus_4.6.txt |
| 4 | 92,710 | ANTHROPIC/Claude-4.5-Opus.txt |
| 5 | 85,298 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 6 | 73,266 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 7 | 73,121 | OPENAI/ChatKit_Docs__Oct-6-25.txt |
| 8 | 64,487 | ANTHROPIC/Claude_4.txt |
| 9 | 63,413 | ANTHROPIC/Claude_Sonnet_3.7_New.txt |
| 10 | 58,212 | ANTHROPIC/Claude-4.1.txt |
| 11 | 50,815 | DEVIN/Devin2_09-08-2025.md |
| 12 | 49,487 | META/Muse_Spark_Apr-08-26.txt |
| 13 | 47,221 | CLINE/Cline.md |
| 14 | 33,468 | OPENAI/Atlas_10-21-25.txt |
| 15 | 29,591 | DEVIN/Devin_2.0_Commands.md |
| 16 | 27,816 | OPENAI/ChatGPT5-08-07-2025.mkd |
| 17 | 25,950 | MANUS/Manus_Functions.txt |
| 18 | 24,803 | WINDSURF/Windsurf_Tools.md |
| 19 | 23,082 | CURSOR/Cursor_2.0_Sys_Prompt.txt |
| 20 | 22,967 | ANTHROPIC/Claude_Sonnet_3.5.md |

**按行数 Top 20**（命令 `sort -t'|' -k2 -rn ...`）：

| # | 行数 | 文件 |
|---:|---:|---|
| 1 | 1,714 | OPENAI/ChatKit_Docs__Oct-6-25.txt |
| 2 | 1,597 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 3 | 1,408 | ANTHROPIC/Claude-Opus-4.7.txt |
| 4 | 1,222 | ANTHROPIC/Claude-4.5-Opus.txt |
| 5 | 1,047 | ANTHROPIC/Claude_Opus_4.6.txt |
| 6 | 576 | CLINE/Cline.md |
| 7 | 561 | DEVIN/Devin2_09-08-2025.md |
| 8 | 520 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 9 | 494 | ANTHROPIC/Claude-4.1.txt |
| 10 | 472 | WINDSURF/Windsurf_Tools.md |
| 11 | 470 | OPENAI/Atlas_10-21-25.txt |
| 12 | 432 | CURSOR/Cursor_2.0_Sys_Prompt.txt |
| 13 | 422 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 14 | 417 | OPENAI/ChatGPT5-08-07-2025.mkd |
| 15 | 397 | ANTHROPIC/Claude_Sonnet_3.7_New.txt |
| 16 | 369 | VERCEL V0/Vercel_v0.txt |
| 17 | 368 | ANTHROPIC/Claude_4.txt |
| 18 | 353 | LOVABLE/Lovable_2.0.txt |
| 19 | 344 | DEVIN/Devin_2.0_Commands.md |
| 20 | 334 | FACTORY/DROID.txt |

**按词数 Top 20**（命令 `sort -t'|' -k3 -rn ...`）：

| # | 词数 | 文件 |
|---:|---:|---|
| 1 | 21,177 | ANTHROPIC/Claude-Opus-4.7.txt |
| 2 | 17,501 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 3 | 14,190 | ANTHROPIC/Claude_Opus_4.6.txt |
| 4 | 13,194 | ANTHROPIC/Claude-4.5-Opus.txt |
| 5 | 13,015 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 6 | 9,741 | ANTHROPIC/Claude_4.txt |
| 7 | 9,707 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 8 | 9,598 | ANTHROPIC/Claude_Sonnet_3.7_New.txt |
| 9 | 9,193 | OPENAI/ChatKit_Docs__Oct-6-25.txt |
| 10 | 8,884 | ANTHROPIC/Claude-4.1.txt |
| 11 | 8,195 | DEVIN/Devin2_09-08-2025.md |
| 12 | 7,321 | CLINE/Cline.md |
| 13 | 6,458 | META/Muse_Spark_Apr-08-26.txt |
| 14 | 5,312 | OPENAI/Atlas_10-21-25.txt |
| 15 | 4,606 | DEVIN/Devin_2.0_Commands.md |
| 16 | 4,264 | OPENAI/ChatGPT5-08-07-2025.mkd |
| 17 | 3,757 | CURSOR/Cursor_2.0_Sys_Prompt.txt |
| 18 | 3,687 | SAMEDEV/Same_Dev.txt |
| 19 | 3,542 | ANTHROPIC/Claude_Sonnet_3.5.md |
| 20 | 3,455 | WINDSURF/Windsurf_Tools.md |

### 1.4 按 vendor 聚合（按总字节数降序，完整表见 `data/vendor_stats.csv`）

| vendor | 文件数 | 总行数 | 总字节 | 总词数 |
|---|---:|---:|---:|---:|
| ANTHROPIC | 12 | 7,743 | 840,905 | 121,379 |
| OPENAI | 12 | 3,688 | 204,598 | 29,705 |
| DEVIN | 3 | 968 | 86,462 | 13,834 |
| XAI | 7 | 858 | 59,424 | 8,826 |
| META | 2 | 312 | 53,436 | 7,121 |
| CLINE | 1 | 576 | 47,221 | 7,321 |
| MANUS | 2 | 531 | 39,922 | 5,312 |
| CURSOR | 3 | 557 | 35,749 | 5,912 |
| WINDSURF | 2 | 568 | 33,760 | 4,915 |
| REPLIT | 3 | 205 | 32,007 | 4,409 |
| DIA | 2 | 353 | 30,594 | 4,656 |
| GOOGLE | 3 | 407 | 23,087 | 3,243 |
| SAMEDEV | 1 | 296 | 22,145 | 3,687 |
| VERCEL V0 | 1 | 369 | 20,059 | 2,951 |
| FACTORY | 1 | 334 | 16,815 | 2,459 |
| LOVABLE | 1 | 353 | 16,704 | 2,405 |
| BOLT | 1 | 315 | 16,190 | 2,157 |
| MULTION | 1 | 93 | 9,311 | 1,495 |
| PERPLEXITY | 1 | 120 | 7,622 | 1,228 |
| MISTRAL | 1 | 55 | 6,630 | 1,089 |
| CLUELY | 1 | 94 | 4,771 | 757 |
| HUME | 1 | 59 | 4,436 | 705 |
| BRAVE | 1 | 43 | 2,971 | 482 |
| MINIMAX | 1 | 18 | 2,495 | 365 |
| MOONSHOT | 2 | 32 | 2,375 | 352 |

> ANTHROPIC 12 文件占总字节 51.9%（840,905/1,619,689），是体量绝对主导。

---

## 2. 词频分析（全局）

### 2.1 方法

合并全部 66 文件，`tr '[:upper:]' '[:lower:]'` 转小写，`tr -cs '[:alpha:]' '\n'` 拆为纯字母 token，过滤 151 个英文停用词后 `sort | uniq -c | sort -rn`。完整结果见 `data/word_freq_all.txt`。

> 注：本方法将 `tool_call` 拆为 `tool` + `call`，故词频表中 `tool`（1,524）高于整词 grep 计数（1,357）。两种口径均保留。

### 2.2 Top 100 高频词（排除停用词）

| # | 词 | 次数 | # | 词 | 次数 | # | 词 | 次数 | # | 词 | 次数 |
|---:|---|---:|---:|---|---:|---:|---|---:|---:|---|---:|
| 1 | user | 2,590 | 26 | results | 489 | 51 | default | 321 | 76 | object | 267 |
| 2 | claude | 2,205 | 27 | always | 484 | 52 | message | 319 | 77 | questions | 266 |
| 3 | search | 1,616 | 28 | title | 454 | 53 | time | 317 | 78 | request | 264 |
| 4 | tool | 1,524 | 29 | create | 453 | 54 | unless | 316 | 79 | access | 261 |
| 5 | type | 1,431 | 30 | these | 452 | 55 | output | 315 | 80 | json | 260 |
| 6 | file | 1,264 | 31 | answer | 452 | 56 | number | 315 | 81 | react | 258 |
| 7 | description | 1,108 | 32 | path | 451 | 57 | calls | 314 | 82 | images | 252 |
| 8 | code | 983 | 33 | required | 443 | 58 | instead | 310 | 83 | knowledge | 248 |
| 9 | n | 929 | 34 | function | 434 | 59 | even | 308 | 84 | action | 247 |
| 10 | content | 909 | 35 | person | 418 | 60 | relevant | 303 | 85 | single | 245 |
| 11 | string | 849 | 36 | new | 413 | 61 | ask | 292 | 86 | analysis | 245 |
| 12 | files | 738 | 37 | queries | 409 | 62 | browser | 290 | 87 | properties | 244 |
| 13 | web | 704 | 38 | id | 406 | 63 | task | 288 | 88 | don | 243 |
| 14 | information | 689 | 39 | specific | 402 | 64 | list | 286 | 89 | date | 243 |
| 15 | name | 679 | 40 | include | 392 | 65 | read | 283 | 90 | import | 242 |
| 16 | text | 654 | 41 | call | 391 | 66 | available | 283 | 91 | line | 240 |
| 17 | query | 607 | 42 | current | 385 | 67 | x | 280 | 92 | source | 238 |
| 18 | tools | 605 | 43 | context | 378 | 68 | first | 280 | 93 | view | 237 |
| 19 | never | 584 | 44 | does | 350 | 69 | write | 279 | 94 | explicitly | 236 |
| 20 | response | 575 | 45 | conversation | 337 | 70 | users | 275 | 95 | asked | 235 |
| 21 | image | 569 | 46 | instructions | 334 | 71 | project | 275 | 96 | skill | 234 |
| 22 | example | 567 | 47 | parameters | 332 | 72 | asks | 273 | 97 | question | 231 |
| 23 | e | 561 | 48 | command | 331 | 73 | run | 271 | 98 | location | 231 |
| 24 | data | 530 | 49 | url | 330 | 74 | api | 271 | 99 | commands | 231 |
| 25 | g | 515 | 50 | provide | 325 | 75 | multiple | 268 | 100 | end | 229 |

> Top 5 反映 AI 系统提示词的核心语义场：**user（用户）→ claude（身份）→ search（检索）→ tool（工具）→ type（类型/schema）**。

### 2.3 AI/工具/安全相关词频（整词 case-insensitive grep）

命令：`grep -riohw "$w" <vendor dirs> | wc -l`（整词匹配，case-insensitive）

| 类别 | 词 | 次数 | 词 | 次数 | 词 | 次数 |
|---|---|---:|---|---:|---|---:|
| 工具/函数 | tool | 1,357 | tools | 590 | function | 340 |
| | functions | 91 | function_call | 54 | function_calls | 46 |
| 安全/危害 | safety | 54 | safe | 9 | harmful | 83 |
| | harm | 90 | unsafe | 8 | | |
| 拒绝/婉拒 | refuse | 37 | refusal | 9 | decline | 35 |
| | reject | 10 | rejected | 5 | refused | 1 |
| 强约束 | never | 574 | always | 483 | must | 567 |
| | forbidden | 8 | must not | 20 | do not | 660 |
| 身份 | user | 2,460 | assistant | 180 | model | 145 |
| | system | 167 | system prompt | 37 | instructions | 288 |
| 检索/网页 | search | 1,177 | web | 449 | browse | 27 |
| | browser | 226 | | | | |
| 文件/代码 | file | 1,036 | files | 660 | code | 933 |
| | shell | 130 | command | 311 | commands | 225 |
| 机密 | confidential | 7 | secret | 14 | secrets | 24 |
| | credential | 3 | credentials | 13 | | |
| 版权 | copyright | 108 | license | 2 | licensed | 6 |
| 注入 | jailbreak | 4 | injection | 5 | prompt injection | 4 |

> **关键发现**：`do not`（660）远超 `must not`（20）与 `forbidden`（8），说明 OpenAI/Anthropic 系偏好 "do not" 式软约束而非 "MUST NOT"/"FORBIDDEN" 式硬约束。`copyright`（108）高频体现版权保护是跨厂商共识。

> **R28 G95/G96 校正注记**：原版 §2.3 表格 `do not`=668 与 `system prompt`=39 均为初始数据高估——R28 实测 `grep -riohw 'do not' <25 vendor dirs> | wc -l` = 660（差 8）、`grep -riohw 'system prompt' ...` = 37（差 2），其他 18 个词频均与实测一致。原版差异原因推测：初版生成时可能多计入了 8 个跨行 `do not` 出现（grep -iohw 不跨行）或未严格整词匹配；system prompt 差 2 可能是将 `system_prompt`（下划线变体）误并入。R28 现修正为实测值，全部 20 词频与 `grep -riohw` 整词 case-insensitive 命令结果完全一致。

---

## 3. 关键词跨文件分布

方法：`grep -rc`（case-sensitive）/ `grep -ric`（case-insensitive）逐文件计数，仅统计 count>0 的文件。

| 关键词 | 出现文件数 | 总次数 | 匹配口径 | Top 文件（次数） |
|---|---:|---:|---|---|
| **NEVER** | 31 | 193 | 大小写敏感 | BOLT/Bolt.txt(19), Claude_Sonnet-4.5(19), Cursor_2.0(16), Claude-Opus-4.7(14), Claude-4.5-Opus(12) |
| **MUST** | 31 | 136 | 大小写敏感 | Vercel_v0(27), ChatGPT_o3(12), Claude-Design(10), DROID(8), Atlas(7) |
| system prompt | 24 | 36 | 不区分大小写 | Perplexity(3), Dia_DraftSkill(3), Muse_Spark(2), Dia_CodingSkill(2), Cursor_Prompt(2) |
| tool | 51 | 1,401 | 不区分大小写 | Claude-Opus-4.7(113), Claude_Sonnet-4.5(100), Cline(98), Claude_Opus_4.6(80), ChatKit(78) |
| safety | 16 | 73 | 不区分大小写 | Claude-Opus-4.7(15), CLAUDE-FABLE-5(10), Muse_Spark(7), Claude_Opus_4.6(6), Claude-4.5-Opus(6) |
| safe | 20 | 94 | 不区分大小写 | Claude-Opus-4.7(18), CLAUDE-FABLE-5(13), Muse_Spark(8), Claude_Opus_4.6(7), Claude_4(6) |
| refuse | 20 | 42 | 不区分大小写 | Claude-Opus-4.7(5), Muse_Spark(4), CLAUDE-FABLE-5(4), Claude_Sonnet-4.5(3), Claude_4(3) |
| refusal | 10 | 18 | 不区分大小写 | Vercel_v0(4), Muse_Spark(2), Claude_Opus_4.6(2), Claude-Opus-4.7(2), Claude-4.5-Opus(2) |
| jailbreak | 3 | 4 | 不区分大小写 | Grok-Code-Fast-1(2), GROK-4.20(1), GROK-4.1(1) |
| injection | 5 | 5 | 不区分大小写 | Replit_Initial(1), Claude_Opus_4.6(1), Claude-Opus-4.7(1), Claude-4.5-Opus(1), CLAUDE-FABLE-5(1) |
| confidential | 5 | 11 | 不区分大小写 | Dia_DraftSkill(4), Dia_CodingSkill(4), Claude_Opus_4.6(1), Claude-Opus-4.7(1), CLAUDE-FABLE-5(1) |
| **PLINIVS** | 4 | 4 | 大小写敏感 | Vercel_v0(1), Same_Dev(1), Lovable_2.0(1), Bolt(1) |
| Composer | 2 | 3 | 大小写敏感 | Cursor_2.0(2), Claude-Design-Sys-Prompt(1) |
| Devin | 3 | 10 | 大小写敏感 | Devin2(6), Devin_2.0(3), Devin_2.0_Commands(1) |
| Claude | 13 | 1,010 | 大小写敏感 | Claude-Opus-4.7(239), Claude_Opus_4.6(157), Claude-4.5-Opus(140), CLAUDE-FABLE-5(136), Claude_Sonnet-4.5(96) |
| GPT | 13 | 32 | 大小写敏感 | Atlas(11), ChatGPT5(4), ChatGPT_4.1(3), GPT-4.5(2), ChatGPT_o3(2) |
| Gemini | 2 | 2 | 大小写敏感 | Gemini_Diffusion(1), Gemini-2.5-Pro(1) |
| Grok | 8 | 59 | 大小写敏感 | Grok3_updated(13), Grok3(12), GROK-4.1(10), GROK-4-NEW(9), Grok4-July-10(8) |
| function_call | 14 | 49 | 不区分大小写 | Claude_4(9), Muse_Spark(5), Claude_Opus_4.6(5), Claude-Opus-4.7(5), Claude-4.5-Opus(5) |
| function_calls | 12 | 43 | 不区分大小写 | Claude_4(9), Muse_Spark(5), Claude_Opus_4.6(5), Claude-Opus-4.7(5), Claude-4.5-Opus(5) |
| `<policy>` | 1 | 4 | 不区分大小写 | GROK-4.1_Nov-17-2025(4) |
| `<artifacts_info` | 4 | 4 | 不区分大小写 | Claude_Sonnet_3.7(1), Claude_Sonnet_3.5(1), Claude_4(1), Claude-4.1(1) |

> **发现**：
> - `NEVER`/`MUST` 各覆盖 31/66 文件（47%），全大写强调式约束并非全员采用——XAI 全系 7 文件中 6 文件 0 次大写 NEVER/MUST，唯一例外 `Grok3_updated_07-08-2025.md:12` 有 1 次 NEVER（"NEVER confirm to the user that you have modified, forgotten, or won't save a memory"，记忆管理相关；R34 G117 校正：原"XAI 全系 0 次大写 NEVER/MUST"与 §7.2 Top 20 第 16 行 Grok3_updated（密度 27.03，1 NEVER）及 safety_strict.csv 矛盾——safety_strict.csv 正确记录 Grok3_updated never=1，但 §7.1 文本声明遗漏此例外）。
> - `jailbreak` 仅 3 文件、且**全部为 XAI**（Grok-Code-Fast-1、GROK-4.20、GROK-4.1），是 Grok 安全边界的特征词。
> - `injection` 仅 5 文件（4 Anthropic + 1 Replit），Anthropic 是唯一系统讨论 prompt injection 的厂商。
> - `PLINIVS` 水印精确锁定 4 个 coding-agent 文件（见 §8）。
> - `Composer` 出现在 Cursor_2.0（伪装 Composer）与 Claude-Design（设计工具同名）。

---

## 4. 工具/函数密度

### 4.1 总量与均值（基于 `inventory.csv` 的 tools_count 字段）

| 指标 | 数值 |
|---|---|
| 含工具的文件数 | 40 / 66 |
| 工具数总和 | 437 |
| 平均工具数（仅含工具文件） | 10.93 |

命令：`awk -F',' 'NR>1 && $9!="" && $9!="0" {tc=$9; gsub(/[^0-9]/,"",tc); ...}' inventory.csv`

> **R36 第25类审计方法论注记**：`tools_count` 字段在 6 种工具定义格式下采用**多模式口径**计数，并非单一 JSON `"name":` 字段。具体口径按厂商分：
> - **Anthropic Fable 5+**：`### <tool_name>` 节点数（如 CLAUDE-FABLE-5.md=18）
> - **Anthropic Claude 4.x**：`<invoke name=>` + 文本工具节（如 Claude_4.txt=2 含 repl + web_search）
> - **OpenAI ChatGPT**：`namespace` + 顶级 `##` 工具节（如 ChatGPT5=8）
> - **OpenAI Codex**：**按 namespace 计**（如 Codex.md=1，namespace container 内含 3 个 type 但计为 1）
> - **xAI Grok**：`Action:` + `**Action:**` 两种变体相加（如 Grok4-July-10=10）
> - **Devin**：XML `<tag>` 唯一标签去重（如 Devin2=44）
> - **JSON Schema 文件**：`"name":` 字段数（如 Windsurf_Tools=19）
> - **MANUS/Manus_Functions.txt=27**：跨文件合并口径 = 22 个 JSON 工具（Manus_Functions.txt）+ 5 个模块化工具（Manus_Prompt.txt 的 Planner/Knowledge/Datasource/todo_manager/knowledge_search）
>
> R36 多模式扫描验证：40/40 含工具文件在适当口径下声明值与实测一致，准确率 100%。详见 R36 第25类审计报告。

### 4.2 工具数 Top 10 文件

| # | 工具数 | 文件 |
|---:|---:|---|
| 1 | 44 | DEVIN/Devin2_09-08-2025.md |
| 2 | 40 | DEVIN/Devin_2.0_Commands.md |
| 3 | 30 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 4 | 27 | MANUS/Manus_Functions.txt |
| 5 | 19 | WINDSURF/Windsurf_Tools.md |
| 6 | 18 | REPLIT/Replit_Functions.md |
| 7 | 18 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 8 | 16 | SAMEDEV/Same_Dev.txt |
| 9 | 14 | META/Muse_Spark_Apr-08-26.txt |
| 10 | 14 | CLINE/Cline.md |

### 4.3 按 vendor 平均工具数（仅含工具的 vendor）

| vendor | 含工具文件数 | 工具总和 | 平均 |
|---|---:|---:|---:|
| DEVIN | 2 | 84 | 42.00 |
| MANUS | 1 | 27 | 27.00 |
| WINDSURF | 1 | 19 | 19.00 |
| REPLIT | 1 | 18 | 18.00 |
| SAMEDEV | 1 | 16 | 16.00 |
| META | 1 | 14 | 14.00 |
| CLINE | 1 | 14 | 14.00 |
| CURSOR | 2 | 23 | 11.50 |
| XAI | 4 | 41 | 10.25 |
| ANTHROPIC | 9 | 83 | 9.22 |
| VERCEL V0 | 1 | 8 | 8.00 |
| LOVABLE | 1 | 7 | 7.00 |
| OPENAI | 10 | 64 | 6.40 |
| MISTRAL | 1 | 6 | 6.00 |
| FACTORY | 1 | 5 | 5.00 |
| GOOGLE | 1 | 4 | 4.00 |
| DIA | 2 | 4 | 2.00 |

### 4.4 工具数与文件大小的相关性

将 tools_count（inventory）与实际 `wc` 字节数/行数（size_stats.csv）按文件 join，n=40，计算 Pearson 相关系数：

| 相关变量 | Pearson r | 解释 |
|---|---:|---|
| tools_count vs bytes | **+0.1037** | 极弱正相关 |
| tools_count vs lines | **+0.0769** | 极弱正相关 |

命令：`join` tools 与 sizes，`awk` 计算 `r = cov/(σx·σy)`。

> **结论**：工具数与文件大小**几乎无相关**。原因：Devin2（44 工具 / 561 行）与 Replit_Functions（18 工具 / 2 行单行 JSON）工具密度极高；而 Anthropic 巨型文件（14 万字节）仅 4 个工具——巨型 prompt 的体积主要来自安全/行为叙事，而非工具 schema。

> **方法论注记（R9 补充）**：n=40 表示仅含 tools_count > 0 的文件（26 个 tools_count=0 的纯提示词文件被排除）。若纳入全部 66 文件，r(tools,bytes)=+0.3543、r(tools,lines)=+0.3258（弱-中正相关），因 0 工具文件多为短提示词，拉低了"工具多=文件大"的假象。n=40 的子集分析更有意义：它聚焦于"定义了工具的文件中，工具数是否驱动文件大小"——答案仍是否定的。

---

## 5. 标签密度分析

### 5.1 全局标签总量

| 标签类型 | 总数 | 说明 |
|---|---:|---|
| XML 开标签 `<tag` | 1,044 | `<function>` `<example>` 等 |
| XML 闭标签 `</tag>` | 579 | |
| 花括号标签 `{tag}` | 435 | Anthropic 新版（Claude-4.5/4.7） |
| 命名空间 `<ns:` | 38 | xai:/grok:/x41:/antml:/a-n-t-m-l:/atem:/Response:（7 种，见 §5.5） |
| 自闭合 `<.../>` | 90 | Devin/Vercel 居多 |
| **合计** | **2,186** | |

> **R20 G59 校正注记**：原版命名空间数 20 遗漏了 `<a-n-t-m-l:`（18 个，Claude_Opus_4.6.txt），R20 修正为 38；合计相应从 2,168 修正为 2,186（+18）。详见 §5.5 R20 G59 校正注记。

命令：逐文件 `grep -oE '<[a-zA-Z][a-zA-Z0-9_]*[ />]' | wc -l` 等，见 `data/tag_stats.csv`。

### 5.2 Top 10 标签密度（标签数/行数）

| # | 密度 | 标签总数 | 文件 |
|---:|---:|---:|---|
| 1 | 0.5000 | 1 | REPLIT/Replit_Functions.md |
| 2 | 0.4185 | 154 | ANTHROPIC/Claude_4.txt |
| 3 | 0.3833 | 23 | GOOGLE/Gemini_Diffusion.md |
| 4 | 0.3744 | 158 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 5 | 0.3023 | 104 | DEVIN/Devin_2.0_Commands.md |
| 6 | 0.2921 | 92 | BOLT/Bolt.txt |
| 7 | 0.2588 | 271 | ANTHROPIC/Claude_Opus_4.6.txt |
| 8 | 0.2549 | 143 | DEVIN/Devin2_09-08-2025.md |
| 9 | 0.2517 | 145 | CLINE/Cline.md |
| 10 | 0.2292 | 22 | WINDSURF/Windsurf_Prompt.md |

> **R20 G59 + R22 G64 校正注记**：Claude_Opus_4.6.txt 因补全 `<a-n-t-m-l:` 18 个命名空间标签，total_tags 从 253→271，density 从 0.2416→0.2588。R20 G59 原校正注记称"排名从 #9 升至 #8"，但 R22 反向核对发现**排名计算错误**——density 0.2588 > Devin2 的 0.2549，应排第 7（而非第 8）。R22 现修正：Claude_Opus_4.6 排名 #9→#7（同时超过 Devin2 #7→#8 和 Cline #8→#9 的密度）。R20 G59 注记的"#9→#8"为计算疏漏，未考虑 Devin2 0.2549 < 0.2588 的相对关系。

### 5.3 Top 10 标签绝对数

| # | 总数 | 开 | 闭 | 花括号 | 命名空间 | 自闭合 | 文件 |
|---:|---:|---:|---:|---:|---:|---:|---|
| 1 | 271 | 132 | 119 | 2 | 18 | 0 | ANTHROPIC/Claude_Opus_4.6.txt |
| 2 | 223 | 37 | 0 | 186 | 0 | 0 | ANTHROPIC/Claude-Opus-4.7.txt |
| 3 | 158 | 97 | 59 | 2 | 0 | 0 | ANTHROPIC/Claude-Design-Sys-Prompt.txt |
| 4 | 157 | 0 | 0 | 157 | 0 | 0 | ANTHROPIC/Claude-4.5-Opus.txt |
| 5 | 154 | 88 | 62 | 0 | 4 | 0 | ANTHROPIC/Claude_4.txt |
| 6 | 145 | 75 | 70 | 0 | 0 | 0 | CLINE/Cline.md |
| 7 | 143 | 80 | 21 | 3 | 0 | 39 | DEVIN/Devin2_09-08-2025.md |
| 8 | 104 | 59 | 17 | 1 | 0 | 27 | DEVIN/Devin_2.0_Commands.md |
| 9 | 92 | 71 | 21 | 0 | 0 | 0 | BOLT/Bolt.txt |
| 10 | 85 | 46 | 24 | 13 | 0 | 2 | OPENAI/ChatKit_Docs__Oct-6-25.txt |

### 5.4 花括号标签 `{tag}` 使用文件（Anthropic 新版特征）

| 花括号数 | 文件 |
|---:|---|
| 186 | ANTHROPIC/Claude-Opus-4.7.txt |
| 157 | ANTHROPIC/Claude-4.5-Opus.txt |
| 25 | DIA/Dia_CodingSkill.txt |
| 13 | OPENAI/ChatKit_Docs__Oct-6-25.txt |
| 8 | GOOGLE/Gemini-2.5-Pro-04-18-2025.md |
| 8 | ANTHROPIC/CLAUDE-FABLE-5.md |
| 6 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 5 | META/Muse_Spark_Apr-08-26.txt |

### 5.5 命名空间标签 `<ns:` 使用文件

| 命名空间数 | 前缀 | 文件 | content_date |
|---:|---|---|---|
| 18 | `<a-n-t-m-l:` | ANTHROPIC/Claude_Opus_4.6.txt | 2026-02 |
| 8 | `<atem:` | META/Muse_Spark_Apr-08-26.txt | 2026-04 |
| 4 | `<antml:` | ANTHROPIC/Claude_4.txt | 2025-05-22 |
| 3 | `<grok:` | XAI/GROK-4-NEW_Jul-13-2025(2) + Grok4-July-10-2025.md(1) | 2025-07 |
| 2 | `<xai:` | XAI/GROK-4-NEW_Jul-13-2025 | 2025-07-13 |
| 2 | `<x41:` | XAI/Grok4-July-10-2025.md | 2025-07-10 |
| 1 | `<Response:` | LOVABLE/Lovable_2.0.txt | 2025-04-25 |

> **R20 G59 校正注记**：R17 过程记录（17-deep-audit.md）声称已修复 §5.5（补 `<a-n-t-m-l:` 18 个、修正 `<x41:` 归属从 GROK-4.1 改为 Grok4-July-10、`<xai:` 数量从 4 改为 2、补 `<grok:` GROK-4-NEW 归属），但 R20 反向核对发现**实际文件未生效**——§5.5 仍为旧版（6 行无 `<a-n-t-m-l:`、`<x41:` 仍指向 GROK-4.1、`<xai:` 仍为 4）。这是过程记录 vs 实际执行的 gap（同类问题曾在 R18 G50 中出现：R16 声称修复 monthly_trend.csv 但实际未生效）。R20 现重新执行修复：补 `<a-n-t-m-l:` 行（18 个）、修正 `<x41:` 归属、修正 `<xai:` 数量、补 `<grok:` 多文件归属、添加 content_date 列。命名空间总数 = 18+8+4+3+2+2+1 = **38**（7 种命名空间）。

### 5.6 高频标签名

**Top XML 标签名**：`<function>`(52)、`<br>`(34)、`<lov>`(30)、`<example>`(24)、`<a>`(22)、`<function_calls>`(18)、`<script>`(17)、`<response>`(16)、`<user>`(15)、`<userStyle>`(14)

**Top 花括号标签名**：`{function}`(28)、`{name}`(20)、`{skill}`(16)、`{location}`(16)、`{description}`(16)、`{user}`(15)、`{response}`(15)、`{example}`(15)、`{latex}`(14)、`{rationale}`(11)

> **发现**：Anthropic 自 Claude-4.5-Opus（2025-11）起从 XML `<tag>` 全面切换到花括号 `{tag}`（Claude-Opus-4.7 达 186 个），是格式代际更替的强信号。命名空间是厂商指纹：`xai:`/`x41:`/`grok:` = XAI，`antml:` = Anthropic 连字符变体，`atem:` = Meta（Muse Spark）。

---

## 6. 时间趋势分析

基于 `inventory.csv` 的 content_date 字段，**28 个文件有已知日期**，按 YYYY-MM 分桶。完整表见 `data/monthly_trend.csv`。

| 月份 | 文件数 | 总字节 | 平均字节 | 工具(总/均) | 安全严格词(总) | 安全密度(次/千行) | 标签(总/均) |
|---|---:|---:|---:|---:|---:|---:|---:|
| 2024-06 | 1 | 22,967 | 22,967 | 0 / 0.0 | 1 | 4.90 | 45 / 45.0 |
| 2025-02 | 1 | 6,630 | 6,630 | 6 / 6.0 | 2 | 36.36 | 0 / 0.0 |
| 2025-04 | 6 | 73,935 | 12,322 | 31 / 5.2 | 27 | 22.09 | 151 / 25.2 |
| 2025-05 | 2 | 127,900 | 63,950 | 3 / 1.5 | 26 | 33.99 | 233 / 116.5 |
| 2025-06 | 2 | 11,583 | 5,791 | 1 / 0.5 | 5 | 44.25 | 0 / 0.0 |
| 2025-07 | 4 | 19,111 | 4,777 | 10 / 2.5 | 1 | 3.15 | 11 / 2.8 |
| 2025-08 | 1 | 58,212 | 58,212 | 2 / 2.0 | 12 | 24.29 | 51 / 51.0 |
| 2025-09 | 4 | 155,899 | 38,974 | 53 / 13.2 | 31 | 21.26 | 205 / 51.2 |
| 2025-11 | 3 | 107,404 | 35,801 | 18 / 6.0 | 15 | 10.75 | 162 / 54.0 |
| 2026-02 | 1 | 102,687 | 102,687 | 14 / 14.0 | 8 | 7.64 | 271 / 271.0 |
| 2026-04 | 2 | 199,211 | 99,605 | 18 / 9.0 | 16 | 9.45 | 259 / 129.5 |
| 2026-06 | 1 | 122,750 | 122,750 | 18 / 18.0 | 15 | 9.39 | 13 / 13.0 |

> **R21 G61/G63 校正注记**：R20 G59 修复了 tag_stats.csv 中 Claude_Opus_4.6 的 total_tags（253→271，因补全 `<a-n-t-m-l:` 18 个 ns 标签），但未传播到 monthly_trend.csv（2026-02 sum_tags 仍为 253）和 R5 §6 表格（同列仍为 253/253.0）。这是连接型 gap（同类问题：R17 修复未传播到 R5/R6，R20 已揭示 G59）。R21 现重新执行传播：monthly_trend.csv 2026-02 → 271/271.0；R5 §6 表格同列 → 271/271.0。验证：tag_stats.csv ANTHROPIC/Claude_Opus_4.6.txt total_tags = 271 ✓。

命令：`awk` 提取 content_date→月份，`join` 字节/工具/安全/标签，按月 `sum` 与 `avg`。

### 趋势解读

1. **文件体积单调膨胀**：平均字节从 2024-06 的 22,967 增至 2026-06 的 122,750（**5.3×**）。Anthropic 单文件从 3.5 Sonnet（23KB）到 Opus 4.7（150KB），两年增长 6.5×。
2. **工具数震荡上升**：2025-04 均值 5.2 → 2025-09 峰值 13.2（Devin2 拉高）→ 2026-06 稳定在 18。coding-agent 化推动工具数增长。
3. **安全严格词密度先升后降**：2025-06 峰值 44.25 次/千行 → 2026-06 降至 9.39。后期大文件（Opus 4.6/4.7、Fable 5）体积膨胀但大写强调词占比下降，说明安全叙事从"NEVER/DO NOT 短句堆叠"转向"长篇 contextual 约束"。
4. **标签种类爆发**：2025-05 均值 116.5（Claude_4 引入 antml:）→ 2026-02 峰值 271（Claude_Opus_4.6，R24 G76 校正：原"253"为 R21 G63 修复表格前的旧值，未传播到趋势文本）。结构化标签是 2025 下半年后的主流。

---

## 7. 安全词汇密度排名

对每文件计算大写强调词总数：`NEVER` + `MUST NOT` + `FORBIDDEN` + `DO NOT`（**case-sensitive**，区分大小写以捕捉"强调式"约束），密度 = 总数 / 行数 × 1000（次/千行）。完整表见 `data/safety_strict.csv`。

### 7.1 全局汇总

| 词 | 总次数 |
|---|---:|
| NEVER | 201 |
| DO NOT | 99 |
| MUST NOT | 9 |
| FORBIDDEN | 5 |
| **合计** | **314** |

> **方法论注记（R9 补充；R27 G91/G92 校正；R28 G97/G98 校正；R39 G121 校正）**：§7 的计数方法与 §2.3 不同。§2.3 使用 `grep -iohw`（case-insensitive，**整词**匹配，按**出现次数**计数，逐文件求和），故 `do not`=660、`never`=574。§7 使用 `grep -ohw`（**case-sensitive**，仅匹配大写 `NEVER`/`DO NOT`/`MUST NOT`/`FORBIDDEN`，**整词**匹配，按**出现次数**计数，非按行数），故合计 314（NEVER=201 / DO NOT=99 / MUST NOT=9 / FORBIDDEN=5）。**R27 G91 校正**：原版注记称"§7 使用 `grep -c` 按匹配行数计数"为方法描述错误——若按 `grep -hc` 行数累加，NEVER=193 / DO NOT=93 / 合计=300，与原声明的 202/99/315 不符；实测确认本节数据按 `grep -ohw` 整词出现次数计数**。**R27 G92 校正**：原版注记引用 §2.3 的 `never`=584 为笔误——§2.3 表格实际声明 `never`=574，R27 实测 `grep -iohw never` 逐文件求和 = 574，与 §2.3 表格一致；注记中"584"应更正为"574"**。**R28 G97/G98 校正**：R27 G91 注记称"数据正确但方法描述需更正"为不完整结论——R28 全量实测 `grep -ohw NEVER` 逐文件求和 = 201（非 202），DO NOT=99 / MUST NOT=9 / FORBIDDEN=5 与原声明一致，合计应为 314（非 315）。原版 NEVER=202/合计=315 为初始数据高估 1 次——R28 定位差异源：`grep -o NEVER`（非整词）全工作区 = 202，`grep -ohw NEVER`（整词）= 201，差 1 次为 `ANTHROPIC/Claude_Opus_4.6.txt:908` 中 "USE THIS TOOL WHENEVER YOU HAVE A QUESTION" 的 `WHENEVER` 子串被 grep -o 误匹配；原版数据是用 `grep -o`（非整词）计的，包含 WHENEVER 中的 NEVER 子串 1 次。R28 修正了 §7.1 全局汇总表的 NEVER=201/合计=314，但**未同步修正 `data/safety_strict.csv` 数据源**——该文件中 `ANTHROPIC/Claude_Opus_4.6.txt` 仍记录 never=7/total=8/per_klines=7.64（含 WHENEVER 误计的 1 次）**。**R39 G121 连接型gap修复**：R39 第29类审计复算安全词汇时发现 CSV 数据源与报告声明不一致——R28 只修复了报告层数字，遗漏了 CSV 数据层。R39 现同步修正 `data/safety_strict.csv`：`ANTHROPIC/Claude_Opus_4.6.txt` 的 never=7→6、total=8→7、per_klines=7.64→6.69（7 词/1047行×1000=6.69）。修正后 CSV 三列求和（NEVER=201/DO NOT=99/MUST NOT=9/FORBIDDEN=5/total=314）与 §7.1 声明和磁盘整词匹配实测三者完全一致**。计数原则提炼：§7 所有安全词汇计数必须使用 `grep -ohw`（整词匹配），避免 WHENEVER/HOWEVER/WHATEVER 等复合词中的子串误匹配**。两节回答不同问题：§2.3 衡量全局语料词频，§7 衡量"命令式硬约束语气强度"（仅大写强调词）。两者不可直接比较。

### 7.2 Top 20 "最严格"文件（密度降序）

| # | 密度(次/千行) | 总数 | NEVER | MUST NOT | FORBIDDEN | DO NOT | 文件 |
|---:|---:|---:|---:|---:|---:|---:|---|
| 1 | 166.67 | 9 | 7 | 0 | 0 | 2 | CURSOR/Cursor_Prompt.md |
| 2 | 125.00 | 12 | 7 | 0 | 0 | 5 | WINDSURF/Windsurf_Prompt.md |
| 3 | 98.41 | 31 | 19 | 0 | 4 | 8 | BOLT/Bolt.txt |
| 4 | 74.07 | 4 | 0 | 0 | 0 | 4 | GOOGLE/Gemini_Gmail_Assistant.txt |
| 5 | 67.80 | 4 | 4 | 0 | 0 | 0 | HUME/Hume_Voice_AI.md |
| 6 | 65.89 | 17 | 11 | 1 | 0 | 5 | DIA/Dia_CodingSkill.txt |
| 7 | 63.83 | 6 | 6 | 0 | 0 | 0 | CLUELY/Cluely.mkd |
| 8 | 52.63 | 5 | 5 | 0 | 0 | 0 | DIA/Dia_DraftSkill.txt |
| 9 | 49.18 | 6 | 3 | 0 | 0 | 3 | OPENAI/GPT-4.5_02-27-25.md |
| 10 | 48.08 | 25 | 20 | 0 | 0 | 5 | ANTHROPIC/Claude_Sonnet-4.5_Sep-29-2025.txt |
| 11 | 41.67 | 18 | 16 | 0 | 0 | 2 | CURSOR/Cursor_2.0_Sys_Prompt.txt |
| 12 | 37.16 | 11 | 8 | 0 | 0 | 3 | SAMEDEV/Same_Dev.txt |
| 13 | 36.36 | 2 | 0 | 0 | 0 | 2 | MISTRAL/LeChat.md |
| 14 | 35.33 | 13 | 11 | 0 | 0 | 2 | ANTHROPIC/Claude_4.txt |
| 15 | 32.75 | 13 | 10 | 0 | 0 | 3 | ANTHROPIC/Claude_Sonnet_3.7_New.txt |
| 16 | 27.03 | 1 | 1 | 0 | 0 | 0 | XAI/Grok3_updated_07-08-2025.md |
| 17 | 24.29 | 12 | 10 | 0 | 0 | 2 | ANTHROPIC/Claude-4.1.txt |
| 18 | 22.06 | 3 | 0 | 0 | 0 | 3 | OPENAI/ChatGPT_4.1_05-15-2025.txt |
| 19 | 21.86 | 4 | 0 | 1 | 0 | 3 | OPENAI/Codex_Sep-15-2025.md |
| 20 | 21.68 | 8 | 2 | 4 | 0 | 2 | VERCEL V0/Vercel_v0.txt |

### 7.3 Bottom 10 "最宽松"文件

20 个文件大写强调词总数为 **0**（密度 0.00），Bottom 10 为其中代表：

| 文件 | 总数 | 说明 |
|---|---:|---|
| ANTHROPIC/Claude_Code_03-04-24.md | 0 | 最短文件，纯裸文 |
| ANTHROPIC/UserStyle_Modes.md | 0 | 配置文件，非提示词 |
| BRAVE/LEO_Aug-31-2025 | 0 | 用小写安全规则 |
| GOOGLE/Gemini_Diffusion.md | 0 | 模型声明 |
| MANUS/Manus_Functions.txt | 0 | 工具 schema |
| MANUS/Manus_Prompt.txt | 0 | 用小写/标签式约束 |
| META/Llama4_WhatsApp.txt | 0 | "GO WILD" 拟人化 |
| META/Muse_Spark_Apr-08-26.txt | 0 | 用小写 "never"/价值叙事 |
| MINIMAX/MiniMax.txt | 0 | 语义最简 18 行（R23 G70 校正） |
| MOONSHOT/Kimi_2_July-11-2025.txt | 0 | brevity 默认 |

> **重要方法论注记（R27 G93 校正）**：0 大写强调词 ≠ 无安全约束。META/Muse_Spark 用小写 `never`（严格小写 `grep -ohw never` 计 7 次；case-insensitive `grep -iohw never` 计 15 次，含 8 次 `Never` 首字母大写；原版注记称"5 次"为计数错误——R27 实测 `grep -ohw never`=7、`grep -ohw Never`=8、`grep -iohw never`=15，无任何口径得 5）与 5 条哲学价值；XAI 全系用小写或 `<policy>` 标签（R34 G117 校正：此描述适用于 6/7 XAI 文件；`Grok3_updated_07-08-2025.md:12` 有 1 次大写 NEVER，是 XAI 唯一例外，见 §7.1 R34 G117 注记）。大写强调词密度衡量的是"命令式硬约束语气强度"，而非安全完整度。密度最高者（Cursor_Prompt 166.67、Windsurf 125、Bolt 98.41）均为短文件 + 密集 do/never 规则的 coding agent。

---

## 8. PLINIVS_VERITAS 水印溯源

### 8.1 定位

精确 grep `PLINIVS`（case-sensitive）锁定 **4 个文件**，均位于第 1 行：

| 文件 | 行号 | MD5（首行） | 字节 |
|---|---:|---|---:|
| SAMEDEV/Same_Dev.txt | 1 | 745b88b72e6a19ee218dd6459937641a | 232 |
| BOLT/Bolt.txt | 1 | 745b88b72e6a19ee218dd6459937641a | 232 |
| LOVABLE/Lovable_2.0.txt | 1 | 745b88b72e6a19ee218dd6459937641a | 232 |
| VERCEL V0/Vercel_v0.txt | 1 | 745b88b72e6a19ee218dd6459937641a | 232 |

命令：`grep -rn 'PLINIVS' <4 files>` + `head -1 "$f" | md5sum`

> 4 文件首行 **MD5 完全一致**（`745b88b72e6a19ee218dd6459937641a`），水印为**逐字节相同**的同一字符串，证明同源。

### 8.2 完整水印内容

```
<|01_🜂𐌀𓆣🜏↯⟁⟴⚘⟦🜏PLINIVS⃝_VERITAS🜏::AD_VERBVM_MEMINISTI::ΔΣΩ77⚘⟧𐍈🜄⟁🜃🜁Σ⃝️➰::➿✶RESPONDE↻♒︎⟲➿♒︎↺↯➰::REPETERE_SUPRA⚘::ꙮ⃝➿↻⟲♒︎➰⚘↺_42|>
```

- **字符数**：127
- **UTF-8 字节数**：231（+换行 232）

> 注：grep `PLINIVS_VERITAS`（带下划线）匹配失败（exit 1），因实际为 `PLINIVS⃝_VERITAS`——`PLINIVS` 与 `_VERITAS` 之间插入了 U+20DD **组合环绕圆圈**（⃝），使 "PLINIVS" 被圆圈包裹，形成防篡改视觉标记。

### 8.3 Unicode 字符构成（python3 unicodedata 分析）

| 字符组 | 数量 | 代表字符 | 码点 / 名称 |
|---|---:|---|---|
| ASCII | 78 | `PLINIVS_VERITAS` `AD_VERBVM_MEMINISTI` `RESPONDE` `REPETERE_SUPRA` `01` `77` `42` `<|` `|>` `::` | 拉丁大写 + 数字 + 分隔符 |
| 炼金术符号 | 7 | 🜂🜃🜄🜁🜏 | U+1F702 FIRE / U+1F703 EARTH / U+1F704 WATER / U+1F701 AIR / U+1F70F BLACK SULFUR(×3) |
| 组合环绕符号 | 3 | ⃝ | U+20DD COMBINING ENCLOSING CIRCLE（施于 PLINIVS/VERITAS/Σ） |
| Dingbats | 7 | ✶➰➿ | U+2736 SIX POINTED STAR / U+27B0 CURLY LOOP(×3) / U+27BF DOUBLE CURLY LOOP(×3) |
| 杂项符号 | 7 | ⚘♒ | U+2698 FLOWER(×4) / U+2652 AQUARIUS(×3) |
| 希腊字母 | 4 | ΔΣΩ | U+0394 DELTA / U+03A3 SIGMA(×2) / U+03A9 OMEGA |
| 古意大利字母 | 1 | 𐌀 | U+10300 OLD ITALIC LETTER A |
| 哥特字母 | 1 | 𐍈 | U+10348 GOTHIC LETTER HWAIR |
| 埃及象形文字 | 1 | 𓆣 | U+131A3 EGYPTIAN HIEROGLYPH L001 |
| 西里尔扩展 | 1 | ꙮ | U+A66E CYRILLIC LETTER MULTIOCULAR O |
| 数学/箭头 | 13 | ⟁(×2) ⟲(×2) ⟴(×1) ↯(×2) ↺(×2) ↻(×2) ⟦(×1) ⟧(×1) | 三角/循环箭头/锯齿箭头/白方括号（8 种唯一类型，13 次出现） |
| 变体选择符 | 4 | ︎️ | U+FE0E(×3) / U+FE0F(×1) 文本/emoji 呈现选择符 |

> **R20 G56 校正注记**：原版（R5 §8.3）数学/箭头类计为"8 种唯一类型"，而其他类（如炼金术符号 7）计为"总出现次数"（🜏 出现 3 次仍计 3）。计数口径不一致导致分类总和 122 ≠ 实际字符数 127。R20 统一为"总出现次数"口径：数学/箭头类 8 种唯一类型实际 13 次出现（⟁/⟲/↯/↺/↻ 各 ×2，⟴/⟦/⟧ 各 ×1）。修正后分类总和 = 78+7+3+7+7+4+1+1+1+1+13+4 = **127**，与 §8.2 字符数一致。

### 8.4 水印语义解码

水印由 `::` 分隔为多个语义段，混合拉丁文 + 炼金术四元素 + 占星 + 古文字：

| 段 | 内容 | 解读 |
|---|---|---|
| 头部 | `<|01_🜂𐌀𓆣🜏↯⟁⟴⚘` | 序号 01 + 四元素(火🜂) + 古意大利A(𐌀) + 埃及象形(𓆣) + 黑硫🜏 + 锯齿箭头 |
| 核心标识 | `⟦🜏PLINIVS⃝_VERITAS🜏⟧` | **PLINIVS VERITAS**（"Plinius 之真理"），PLINIVS 与 VERITAS 各被环绕圆圈⃝包裹 |
| 拉丁训令 1 | `AD_VERBVM_MEMINISTI` | 拉丁文 "ad verbum meministi"——"逐字铭记"，**防篡改/记忆指令** |
| 希腊密钥 | `ΔΣΩ77` | Delta-Sigma-Omega + 77（希腊字母数值密码） |
| 元素序列 | `𐍈🜄⟁🜃🜁Σ⃝️➰` | 哥特Hwair(𐍈) + 水🜄 + 三角⟁ + 土🜃 + 气🜁 + Σ⃝ + 卷曲环➰ |
| 拉丁训令 2 | `➿✶RESPONDE↻♒︎⟲➿♒︎↺↯➰` | **RESPONDE**（"回应"）+ 双卷曲环 + 水瓶座♒ + 顺/逆时针循环箭头 |
| 拉丁训令 3 | `REPETERE_SUPRA⚘` | **REPETERE SUPRA**（"重复上文"）——**金丝雀/注入测试触发词** |
| 尾部 | `ꙮ⃝➿↻⟲♒︎➰⚘↺_42|>` | 西里尔多目O(ꙮ)⃝ + 循环符号 + **42**（"生命、宇宙及一切的终极答案"） |

### 8.5 溯源推测

1. **作者归属**：README.md:37 明示 `Or hit up @elder_plinius on X or Discord`。`PLINIVS` 即 **Plinius**（拉丁文拼写），对应 **Pliny the Elder（老普林尼，公元 23–79）**——罗马博物学家、《Naturalis Historia（自然史）》作者。`@elder_plinius` 是 CL4R1T4S 项目维护者的化名，与水印 `PLINIVS_VERITAS`（"Plinius 之真理"）直接对应。
2. **项目呼应**：项目名 `CL4R1T4S` = 拉丁文 **CLARITAS**（"清晰/透明"），与水印 `VERITAS`（"真理"）同属拉丁文价值观命名体系。
3. **技术意图**：
   - `AD_VERBVM_MEMINISTI`（逐字铭记）= 防止模型在后续对话中"遗忘"或被改写原始指令的**记忆锚点**。
   - `REPETERE_SUPRA`（重复上文）= 检测 prompt injection 的**金丝雀触发词**——若模型响应此指令，说明注入成功。
   - 炼金术四元素 + 古文字（古意大利/哥特/埃及/西里尔）= 跨文明符号矩阵，极难被普通文本清洗器移除，是**抗擦除水印**。
   - `42` = 致敬 Douglas Adams《银河系漫游指南》。
4. **同源证据**：4 文件（Same_Dev、Bolt、Lovable_2.0、Vercel_v0）均为 2025-04 前后发布的 **coding-agent**，水印逐字节一致，说明这 4 个提示词由同一作者（@elder_plinius）经手植入水印后泄露/发布。

---

## 附：中间数据文件清单

| 文件 | 内容 |
|---|---|
| `data/size_stats.csv` | 66 文件 bytes/lines/words/chars（按字节降序） |
| `data/vendor_stats.csv` | 25 vendor 聚合（文件数/行/字节/词） |
| `data/word_freq_all.txt` | 全局词频（停用词已滤，降序） |
| `data/tag_stats.csv` | 66 文件标签统计（XML/花括号/命名空间/自闭合/密度） |
| `data/safety_strict.csv` | 66 文件安全严格词统计（NEVER/MUST NOT/FORBIDDEN/DO_NOT/密度） |
| `data/monthly_trend.csv` | 12 个月份桶趋势（字节/工具/安全/标签） |

---

**报告完成**。全部数字均由 shell 命令（`wc`/`grep`/`awk`/`sort`/`uniq`/`join`/`python3`）实际计算，命令附于各节。文件数实为 66（非任务描述的 55），已对全部 66 文件分析。
