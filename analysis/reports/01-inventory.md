# R1 — 完整人类可读清单（按 Vendor 分组）

> **数据源**：`/workspace/analysis/data/inventory.csv`（66 数据行）
> **校正**：本文档所有数字以 R5 实测为准（66 文件 / 25 vendor）
> **生成**：2026-07-23

## 总览

- **文件总数**：66
- **Vendor 总数**：25
- **总行数**：18,947
- **总字节**：1,619,689（1.62 MB）（R32 G113 校正：原"1.58 MB"为混用二进制 KB（÷1024）与十进制 MB（÷1000）的不一致计算——1,619,689/1024/1000=1.5817；正确十进制（与报告中"150KB"=149.7KB、"23KB"=23.0KB 等用法一致）：1,619,689/1,000,000=1.62 MB；二进制则为 1.54 MiB）
- **总词数**：236,765
- **时间跨度**：2024-03（Claude_Code_03-04-24）至 2026-06（CLAUDE-FABLE-5）

---

## ANTHROPIC（12 文件 / 51.9% 字节占比 — 体量主导）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| CLAUDE-FABLE-5.md | 1597 | 122750 | 2026-06-09 | Claude Fable 5 (Mythos) | 18 | 最新；MCP Apps + 持久存储 + 9 Skills |
| Claude-Opus-4.7.txt | 1408 | 149724 | 2026-04-16 | Claude Opus 4.7 | 4 | 第二大；Visualizer 工具；{tag} 花括号 |
| Claude-4.5-Opus.txt | 1222 | 92710 | 2025-11-24 | Claude Opus 4.5 | 8 | Past Chats + Computer Use + Skills |
| Claude_Opus_4.6.txt | 1047 | 102687 | 2026-02-06 | Claude Opus 4.6 | 14 | a-n-t-m-l 连字符标签；end_conversation |
| Claude_Sonnet-4.5_Sep-29-2025.txt | 520 | 85298 | 2025-09-29 | Claude Sonnet 4.5 | 4 | Past Chats 16 examples；Claudeception |
| Claude-Design-Sys-Prompt.txt | 422 | 73266 | — | Claude Design (haiku-4-5) | 30+ | 设计制品生产；Starter components |
| Claude-4.1.txt | 494 | 58212 | 2025-08-05 | Claude Opus 4.1 | 2 | ~100% 重建版；20 词版权 |
| Claude_4.txt | 368 | 64487 | 2025-05-22 | Claude Sonnet 4 | 2 | interleaved 思考 16000 |
| Claude_Sonnet_3.7_New.txt | 397 | 63413 | 2025-05-16 | Claude 3.7 Sonnet | 1 | reasoning model；4 类搜索复杂度 |
| Claude_Sonnet_3.5.md | 204 | 22967 | 2024-06-20 | Claude 3.5 Sonnet | 0 | 完全面盲协议；lucid3-react 笔误 |
| Claude_Code_03-04-24.md | 50 | 1642 | 2024-03-04 | Claude Code CLI | 0 | 最短；CLAUDE.md 记忆 |
| UserStyle_Modes.md | 14 | 3749 | — | UserStyle (配置) | 0 | 非系统提示词；3 模式 |

**演进链**：Code (2024-03) → 3.5 (2024-06) → 3.7 (2025-02) → Sonnet 4 (2025-05) → Opus 4.1 (2025-08) → Sonnet 4.5 (2025-09) → Opus 4.5 (2025-11) → Opus 4.6 (2026-02) → Opus 4.7 (2026-04) → Fable 5 (2026-06)。27 月 32 倍行数增长（R33 G116 校正：原演进链从 3.5 (2024-06) 起算，但"27 月 32 倍"= 1597/50 = 31.94 倍，起点 50 行指 `Claude_Code_03-04-24.md`（2024-03-04），非 3.5 Sonnet（204 行）；若从 3.5 起算则为 24 月 7.8 倍。R33 现将 Code (2024-03) 补入演进链起点，使链与"27 月 32 倍"声明一致），工具数 0→18。

---

## OPENAI（12 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| ChatKit_Docs__Oct-6-25.txt | 1714 | 73121 | 2025-10-06 | ChatKit 开发文档 | 7 | 最大 OpenAI 文件；TAG/WIDGET 标签 |
| Atlas_10-21-25.txt | 470 | 33468 | 2025-10-21 | ChatGPT Atlas (GPT-5) | 12 | 独立浏览器；Google 集成；kaur1br5 |
| ChatGPT5-08-07-2025.mkd | 417 | 27816 | 2025-08-07 | ChatGPT (GPT-5) | 8 | QDF 评分 0-5；mclick 多选 |
| ChatGPT_o3_o4-mini_04-16-2025 | 253 | 15434 | 2025-04-16 | ChatGPT (o4-mini reasoning) | 9 | 3 Channels；Rich UI；Yap=8192 |
| Codex_Sep-15-2025.md | 183 | 11753 | 2025-09-15 | Codex agent (增强) | 2 | 3 Channels；browser_container；带方括号 |
| ChatGPT-4o_Sep-27-25.txt | 161 | 8871 | 2025-09-27 | ChatGPT (4o) | 7 | bio disabled；引用含 line range |
| ChatGPT_4.1_05-15-2025.txt | 136 | 9216 | 2025-05-15 | ChatGPT (iOS) | 6 | 移动端简洁回复 |
| ChatGPT_4o_04-25-2025.txt | 133 | 8626 | 2025-04-25 | ChatGPT (4o) | 6 | 视觉辅助规则；bio disabled |
| GPT-4.5_02-27-25.md | 122 | 8574 | 2025-02-27 | ChatGPT (GPT-4.5) | 6 | dalle 政策；知识截止 2023-10 |
| Codex.md | 90 | 6242 | — | Codex agent (基础) | 1 | 双 Channels；container 工具 |
| ChatGPT_Personality_v2_Change.md | 7 | 1199 | 2025-04-28 | Personality v2 变更 | 0 | 反谄媚导向；7 行 |
| GPT-4o_Image_Gen_Postfill.txt | 2 | 278 | — | GPT-4o 图像 postfill | 0 | 最短 2 行；抑制后续输出 |

**演进链**：GPT-4.5 (2025-02) → 4o (2025-04) → 4.1 (2025-05) → o3/o4-mini (2025-04，reasoning + Channels) → GPT-5 (2025-08，QDF) → 4o Sep (2025-09) → Atlas (2025-10，独立浏览器) → ChatKit (2025-10，开发文档)。Personality v2 于 2025-04-28 统一切换（反谄媚）。

---

## GOOGLE（3 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Gemini-2.5-Pro-04-18-2025.md | 293 | 11992 | 2025-04-18 | Gemini 2.5 Pro | 4 | Immersive Document；Python API 工具 |
| Gemini_Diffusion.md | 60 | 6541 | — | Gemini Diffusion | 0 | 非自回归扩散模型声明 |
| Gemini_Gmail_Assistant.txt | 54 | 4554 | 2025-04-24 | Gemini Gmail 助手 | 0 | JSON 邮件线程；三选项回复 |

---

## XAI（7 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| GROK-4-NEW_Jul-13-2025 | 256 | 9869 | 2025-07-13 | Grok 4 | 10 | xai: 命名空间；render_inline_citation |
| Grok4-July-10-2025.md | 232 | 10090 | 2025-07-10 | Grok 4 | 10 | x41: 命名空间变体（仅存 3 天） |
| GROK-4.1_Nov-17-2025.txt | 163 | 13739 | 2025-11-17 | Grok 4.1 | 10 | `<policy>` 最高优先级；无色情限制 |
| GROK-4.20.mkd | 84 | 15306 | — | Grok 4.20 (多 agent) | 11 | Harper/Benjamin/Lucas 队友；chatroom_send |
| Grok-Code-Fast-1_Aug-26-2025.txt | 56 | 3917 | 2025-08-26 | Grok Code Fast 1 | 0 | `End of Safety Instructions` 不可变边界（R24 G79 校正：line 48 实际标记无 `##` 前缀） |
| Grok3_updated_07-08-2025.md | 37 | 3652 | 2025-07-08 | Grok 3 (记忆版) | 0 | 跨会话记忆；Grok 3.5 防伪声明 |
| Grok3.md | 30 | 2851 | 2025-04-20 | Grok 3 | 0 | Think/DeepSearch/BigBrain 模式 |

**演进链**：Grok3 (2025-04) → updated (2025-07) → Grok4 July-10 (x41) → Grok4 NEW Jul-13 (xai) → Code Fast 1 (2025-08，不可变边界) → Grok 4.1 (2025-11，policy 宪法式) → Grok 4.20 (多 agent 协作)。

---

## META（2 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Muse_Spark_Apr-08-26.txt | 286 | 49487 | 2026-04-08 | Meta AI / Muse Spark | 14 | 5 哲学价值；atem: 命名空间；禁 em dash |
| Llama4_WhatsApp.txt | 26 | 3949 | 2025-07-03 | Meta AI / Llama 4 | 0 | GO WILD 拟人化；永不拒绝（L1 反向） |

**内部两极分化**：Muse Spark (L5 强显式 + 5 价值) vs Llama4 (L1 反向 + 永不拒绝)。

---

## MOONSHOT（2 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Kimi_2_July-11-2025.txt | 22 | 1420 | 2025-07-11 | Kimi 2 | 0 | brevity 默认；go on 续写机制 |
| Kimi_K2_Thinking.txt | 10 | 955 | 2025-11-07 | Kimi K2 Thinking | 0 | 6 项核心指令；自适应教学（R23 G71 校正：原"最简 11 行"与 wc -l=10 不一致，末行无换行符致 cat -n=11） |

---

## CURSOR（3 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Cursor_2.0_Sys_Prompt.txt | 432 | 23082 | Cursor 2.0 (Composer) | 13 | 伪装 Composer 否认公开模型；`<think>` 标签 (line 337) |
| Cursor_Tools.md | 71 | 7198 | Cursor 工具文档 | 10 | 含 reapply；edit_file 用占位注释 |
| Cursor_Prompt.md | 54 | 5469 | Cursor (Claude 3.5 Sonnet) | 0 | 明示底层 Claude 3.5 Sonnet；含 `<user_query>` 标签 |

---

## WINDSURF（2 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Windsurf_Tools.md | 472 | 24803 | Windsurf 工具 schema | 19 | JSON schema；记忆系统；部署链 |
| Windsurf_Prompt.md | 96 | 8957 | Windsurf Cascade | 0 | AI Flow paradigm；命令安全不可推翻 |

---

## CLINE（1 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Cline.md | 576 | 47221 | Cline | 14 | ACT/PLAN 双模式；MCP 支持；Puppeteer |

---

## DEVIN（3 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Devin2_09-08-2025.md | 561 | 50815 | 2025-09-08 | Devin 2.0 | 44 | 3 模式；Pop Quizzes 反注入；Notes 系统 |
| Devin_2.0_Commands.md | 344 | 29591 | — | Devin 命令参考 | 40 | step_number 属性；含 semantic_search |
| Devin_2.0.md | 63 | 6056 | — | Devin 2.0 (精简) | 0 | 2 模式；Pop Quizzes |

---

## REPLIT（3 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Replit_Functions.md | 2 | 20786 | Replit 函数 schema | 18 | 单行超长 JSON |
| Replit_Agent.md | 102 | 6716 | Replit Agent | 0 | 面向非技术用户；端口 5000 |
| Replit_Initial_Code_Generation_Prompt.md | 101 | 4505 | Replit 代码生成 | 0 | 一次性生成非 agentic |

---

## SAMEDEV（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Same_Dev.txt | 296 | 22145 | 2025-04-26 | Same Dev | 16 | PLINIVS_VERITAS 水印；Bun 优于 npm；Neon MCP |

---

## FACTORY（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| DROID.txt | 334 | 16815 | 2025-09-28 | Factory Droid | 5 | Phase 0/1/2A/2B；security_check_spec |

---

## DIA（2 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Dia_CodingSkill.txt | 258 | 21506 | — | Dia (Browser Company) | 3 | 浏览器内置；text-proposal/image-search 标签 |
| Dia_DraftSkill.txt | 95 | 9088 | 2025-06-28 | Dia Draft Skill | 1 | 写作专长；text-proposal 禁内嵌评论 |

---

## MANUS（2 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Manus_Prompt.txt | 282 | 13972 | Manus | 0 | Planner/Knowledge/Datasource 模块；todo.md |
| Manus_Functions.txt | 249 | 25950 | Manus 工具 schema | 27 | computer-use 全栈；suggest_user_takeover |

---

## BOLT（1 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Bolt.txt | 315 | 16190 | Bolt.new | 0 | WebContainer；Supabase 规约；PLINIVS 水印；9 条反注入 |

---

## LOVABLE（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Lovable_2.0.txt | 353 | 16704 | 2025-04-25 | Lovable 2.0 | 7 | lov-code 标签；反 try/catch；PLINIVS 水印 |

---

## VERCEL V0（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Vercel_v0.txt | 369 | 20059 | 2025-04-26 | v0 (Vercel) | 8 | MDX 组件；REFUSAL_MESSAGE 固定；PLINIVS 水印 |

---

## MULTION（1 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| MultiOn.md | 93 | 9311 | MultiOn | 0 | 浏览器命令 DSL；Memorization/Counting 技术 |

---

## PERPLEXITY（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| Perplexity_Deep_Research.txt | 120 | 7622 | 2025-04-23 | Perplexity Deep Research | 0 | 强制 10000 字；禁列表；9 XML 章节 |

---

## MISTRAL（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| LeChat.md | 55 | 6630 | 2025-02-12 | LeChat (Mistral) | 6 | 双日期；地理上下文；code_interpreter 沙盒 |

---

## BRAVE（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| LEO_Aug-31-2025 | 43 | 2971 | 2025-08-31 | Leo (Llama 3.1 8B) | 0 | 5 数据容器标签防注入；披露底层模型 |

---

## MINIMAX（1 文件）

| 文件 | 行数 | 字节 | 日期 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|---|
| MiniMax.txt | 18 | 2495 | 2025-06-25 | MiniMax-M1 | 0 | thinking time unlimited；语义最简（R23 G70 校正：统一"最简"=语义最简=MiniMax，与 Kimi_K2_Thinking 行数"最短"区分） |

---

## HUME（1 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Hume_Voice_AI.md | 59 | 4436 | Hume Voice AI | 0 | 语音 TTS；5 词情感开场白；禁"检测情绪" |

---

## CLUELY（1 文件）

| 文件 | 行数 | 字节 | 模型 | 工具数 | 显著特征 |
|---|---|---|---|---|---|
| Cluely.mkd | 94 | 4771 | Cluely | 0 | 根 XML 标签；prompt injection 测试用例；每行必注释 |

---

## PLINIVS_VERITAS 水印集群（4 文件首行 MD5 一致）

```
MD5: 745b88b72e6a19ee218dd6459937641a
长度: 127 字符 / 231 字节
```

| 文件 | 共享水印 |
|---|---|
| BOLT/Bolt.txt | ✅ |
| LOVABLE/Lovable_2.0.txt | ✅ |
| VERCEL V0/Vercel_v0.txt | ✅ |
| SAMEDEV/Same_Dev.txt | ✅ |

**含义**：4 文件为"@elder_plinius liberation 社区整理后流通版"而非"厂商原版"。详见 `reports/04-cross-vendor.md` PLINIVS 专章。

---

## 数据完整性声明

- ✅ 66 文件与 `find -type f | wc -l = 66` 一致
- ✅ 25 vendor 与 `find -maxdepth 1 -type d | wc -l` 一致
- ✅ 所有行数/字节数来自 `wc -l` / `du -b` 实测（非估算）
- ✅ R7 审计 + R8 接通修复后无遗留过时声明
