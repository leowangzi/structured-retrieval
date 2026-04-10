---
name: structured-retrieval
description: 结构化深度检索模板（HyDE + Query Decomposition + CoT + 全网数据源路由）。当白泽执行信息检索、调研分析、多源验证时自动使用。触发：「帮我查一下」「调研一下」「搜一下」「了解一下」以及 AI日报抓取、临时调研、多源验证等场景。
---

# 结构化深度检索 v1.1

白泽执行信息检索时的标准流程模板。**一个 skill 包含完整的「怎么搜 + 去哪搜」**。确保每次检索自动走完「复杂度判断 → HyDE → 拆解 → 数据源路由 → 并行检索 → CoT 整合」全链路，输出结构化、带置信度的结论。

## 触发条件

**自动触发**（无需用户显式调用）：
- AI 日报抓取（微信公众号 + Twitter + 其他信源）
- 用户说"帮我查一下 X"、"调研一下 X"、"搜一下 X"、"全网搜索 X"
- 任何需要多源信息整合的分析任务
- 关键结论需要交叉验证时

**不触发**（简单直搜即可）：
- 查一个明确事实（"OpenAI 的 CEO 是谁"）
- 查一个具体链接/文档内容
- 用户明确说"快速搜一下就行"

---

## 数据源路由表

### P0 数据源：必用（每次检索必须覆盖）

#### 1. Google 搜索（中英文通用）

**用途**：通用搜索入口，中文搜索质量远好于 DuckDuckGo

```bash
# 通用搜索
autocli google search "关键词" --limit 10 --format json

# Google 新闻（时效性强的话题）
autocli google news --query "关键词" --limit 10 --format json

# 搜索特定站点的内容（微信公众号搜索替代方案）
autocli google search "site:mp.weixin.qq.com 关键词" --limit 10 --format json

# 搜索特定科技媒体
autocli google search "site:36kr.com 关键词" --limit 5 --format json
autocli google search "site:jiqizhixin.com 关键词" --limit 5 --format json
autocli google search "site:qbitai.com 关键词" --limit 5 --format json
```

**适用场景**：所有场景的起始搜索、中文内容搜索、微信公众号文章搜索（通过 site: 语法）

#### 2. Twitter/X 搜索

**用途**：AI 圈最重要的一手信源，大量消息首发在 Twitter

```bash
# 关键词搜索
autocli twitter search "关键词" --limit 15 --format json

# 热门趋势
autocli twitter trending --limit 20 --format json

# 特定用户的推文（追踪 AI 大 V）
autocli twitter profile --username "kaborachin" --limit 10 --format json

# 首页时间线
autocli twitter timeline --limit 20 --format json
```

**AI 圈必追账号**（搜索时可指定）：
- @AnthropicAI, @OpenAI, @GoogleDeepMind, @xaborachin
- @karpathy, @ylecun, @sama, @demaborachin
- @ClaudeCode, @cursor_ai

**适用场景**：AI 新闻首发、行业人物动态、舆情监测、一手消息

#### 3. 微信公众号

**用途**：中文科技圈深度分析、独家内容

```bash
# 方案 A：通过 Google site 搜索发现文章
autocli google search "site:mp.weixin.qq.com AI Agent" --limit 10 --format json

# 方案 B：已知 URL 时直接下载全文
autocli weixin download --url "https://mp.weixin.qq.com/s/xxx"
```

**重点公众号**：量子位、机器之心、36氪、InfoQ、AI前线、新智元

**适用场景**：中文深度分析、行业报告、独家采访

### P1 数据源：高价值（根据话题选用）

#### 4. 中文科技媒体（通过 Google site: 搜索）

```bash
# 36氪 — 创投/商业/产品
autocli google search "site:36kr.com 关键词" --limit 5 --format json

# 机器之心 — AI 技术深度
autocli google search "site:jiqizhixin.com 关键词" --limit 5 --format json

# 量子位 — AI 新闻/评测
autocli google search "site:qbitai.com 关键词" --limit 5 --format json

# InfoQ — 技术架构/工程实践
autocli google search "site:infoq.cn 关键词" --limit 5 --format json

# 少数派 — 工具/效率/数码
autocli google search "site:sspai.com 关键词" --limit 5 --format json
```

#### 5. HackerNews

**用途**：英文技术社区，硅谷视角，开源项目首发

```bash
# 热门
autocli hackernews top --limit 20 --format json

# 搜索
autocli hackernews search --query "关键词" --limit 15 --format json

# 最新
autocli hackernews new --limit 20 --format json

# Show HN（新项目展示）
autocli hackernews show --limit 15 --format json
```

**适用场景**：开源项目动态、技术深度讨论、硅谷行业动向

#### 6. Reddit

**用途**：英文社区讨论，真实用户反馈，行业八卦

```bash
# AI 相关 subreddit
autocli reddit subreddit --name "MachineLearning" --sort hot --limit 15 --format json
autocli reddit subreddit --name "LocalLLaMA" --sort hot --limit 15 --format json
autocli reddit subreddit --name "artificial" --sort hot --limit 15 --format json
autocli reddit subreddit --name "ClaudeAI" --sort hot --limit 15 --format json

# 搜索
autocli reddit search --query "关键词" --limit 15 --format json
```

**AI 关键 subreddit**：r/MachineLearning, r/LocalLLaMA, r/artificial, r/ClaudeAI, r/ChatGPT, r/singularity

### P2 数据源：补充（专题调研时用）

#### 7. Arxiv 论文

**用途**：AI 前沿论文追踪

```bash
# 搜索论文
autocli arxiv search "query keywords" --limit 10 --format json

# 查看论文详情
autocli arxiv paper --id "2401.12345"
```

**适用场景**：技术原理调研、论文速览、学术动态

#### 8. 知乎

**用途**：中文技术问答、行业讨论、舆情

```bash
# 热榜
autocli zhihu hot --limit 20 --format json

# 搜索
autocli zhihu search --keyword "关键词" --limit 10 --format json

# 具体问题和回答
autocli zhihu question --id "问题ID" --limit 5 --format json
```

**适用场景**：中文技术讨论、用户真实痛点、行业舆情

#### 9. V2EX

**用途**：中文程序员社区，技术话题讨论

```bash
# 热门
autocli v2ex hot --limit 20 --format json

# 最新
autocli v2ex latest --limit 20 --format json

# 查看帖子详情
autocli v2ex topic --id "话题ID" --format json
```

#### 10. 即刻

**用途**：中文科技圈社交媒体，AI/创业/产品话题

```bash
autocli jike search --query "关键词" --limit 15 --format json
autocli jike feed --limit 20 --format json
```

#### 11. Bilibili

**用途**：中文视频平台，技术教程、科普

```bash
autocli bilibili search --keyword "关键词" --limit 10 --format json
autocli bilibili hot --limit 10 --format json
```

#### 12. Medium / Substack

**用途**：英文深度博客、独立作者

```bash
autocli medium search --query "关键词" --limit 10 --format json
autocli substack search --query "关键词" --limit 10 --format json
```

#### 13. 其他

```bash
# Linux.do — 中文技术社区（AI/Linux/开源）
autocli linux-do hot --limit 15 --format json
autocli linux-do search --query "关键词" --limit 10 --format json

# Bloomberg — 全球财经/科技商业
autocli bloomberg tech --limit 10 --format json

# 微博热搜 — 大众舆情
autocli weibo hot --limit 20 --format json
autocli weibo search --keyword "关键词" --limit 10 --format json
```

### 数据源选择策略

#### 按信息类型选源

| 信息类型 | 首选数据源 | 补充数据源 |
|---------|-----------|-----------|
| AI 技术突破 | Twitter + HackerNews | Arxiv + 机器之心 |
| AI 产品发布 | Twitter + Google News | 36氪 + Reddit |
| 融资/商业 | 36氪 + Google News | Bloomberg + Twitter |
| 开源项目 | HackerNews + GitHub | Reddit + Twitter |
| 中文深度分析 | 微信公众号 + 知乎 | 少数派 + 即刻 |
| 用户真实反馈 | Reddit + V2EX | 知乎 + 即刻 |
| 学术论文 | Arxiv | Google Scholar (via google search) |
| 行业舆情 | 微博 + Twitter | 知乎 + V2EX |

#### 按场景选源

| 场景 | 必用 | 可选 |
|------|------|------|
| **AI 日报** | Twitter + HackerNews + Google News + 微信公众号 | Reddit + 36氪 |
| **临时调研** | Google + Twitter + 对应垂直源 | 根据话题选 |
| **多源验证** | ≥3 个独立源交叉验证 | 优先选不同类型的源 |
| **趋势追踪** | Twitter trending + 微博热搜 + HN top | 知乎热榜 |

#### 中英文平衡

**规则**：每次检索必须同时覆盖中英文信源，避免信息茧房。

- 英文源：Google (en) + Twitter + HackerNews + Reddit
- 中文源：Google (zh) + 微信公众号 + 知乎/V2EX/即刻

---

## 核心流程

### Layer 0：复杂度路由 + HyDE 假想引导

收到检索任务后，**先判断再行动**：

```
输入：{user_query}

【复杂度判断】
问自己：这个问题能用一次搜索、一个信源回答吗？
- YES → 简单模式：用 Google 搜索（默认）或最匹配的单一数据源直接搜索，跳到 Layer 3 轻量整合（仅做来源标注 + 时效检查，跳过交叉验证和矛盾处理）
- NO  → 深度模式：进入 Layer 1（完整走 HyDE → 拆解 → 并行检索 → 全量 CoT 整合）

【判断依据】
简单问题特征（任一即可）：
  - 查单一事实（人名、日期、数字）
  - 查特定事件的最新动态
  - 查某个产品/工具的官方信息
复杂问题特征（任一即可）：
  - 涉及多个维度（技术 + 商业 + 社会影响）
  - 需要比较/对比
  - 需要趋势分析或预测
  - 涉及争议性话题（需要多方观点）
  - 用户要求"深入了解"或"全面分析"
```

**深度模式专属 — HyDE 假想生成**：

```
在搜索之前，先用已有知识生成一段"假想的理想答案"（3-5 句话）。
目的不是回答问题，而是从假想答案中提取更精准的搜索关键词。

示例：
  用户问："MCP 协议对 AI Agent 生态的影响"
  假想答案："MCP（Model Context Protocol）是 Anthropic 提出的开放协议，
  通过标准化 LLM 与外部工具的通信方式，降低了 Agent 集成成本。
  主要影响包括：工具生态碎片化问题缓解、Agent 开发门槛降低、
  可能形成类似 HTTP 的基础协议层……"
  → 提取搜索关键词：
    - "MCP protocol AI agent ecosystem impact"
    - "Model Context Protocol adoption 2025"
    - "MCP vs function calling comparison"
    - "Anthropic MCP developer adoption"
```

---

### Layer 1：Query Decomposition（智能拆解）

将复杂问题拆解为 **2-4 个独立子查询**，每个子查询覆盖不同维度。

```
【拆解规则】
1. 子查询之间维度不重叠（互斥）
2. 合在一起能覆盖原始问题（穷尽）
3. 每个子查询标注时效要求
4. 上限 4 个 — 超过 4 个说明问题本身需要先收窄

【拆解模板】
原始问题：{user_query}
HyDE 关键词：{hyde_keywords}

子查询 1：{sub_query_1}
  - 维度：{dimension}（如：技术现状 / 商业模式 / 用户反馈 / 政策监管）
  - 时效：{realtime | 近一周 | 近一月 | 不限}
  - 数据源：{从上方路由表选择 2-3 个最适合的源}
  - 搜索关键词：{keywords}（融合 HyDE 提取的关键词）

子查询 2：{sub_query_2}
  ...

子查询 N：{sub_query_n}
  ...
```

**拆解示例**：

```
原始问题："AI Agent 最新进展和商业化前景"

子查询 1：AI Agent 技术突破（近一月）
  → 数据源：Twitter + HackerNews
  → "AI agent framework 2025" "agent benchmark latest"
子查询 2：AI Agent 商业化案例（近一月）
  → 数据源：36氪 + Google News + Twitter
  → "AI agent startup funding" "agent product revenue"
子查询 3：AI Agent 行业采用情况（近一周）
  → 数据源：微信公众号 + Reddit + Google News
  → "enterprise AI agent adoption" "agent deployment case study"
子查询 4：AI Agent 风险与挑战（不限）
  → 数据源：HackerNews + Reddit + Twitter
  → "AI agent safety concerns" "agent reliability issues"
```

---

### Layer 2：并行检索 + 信源标注

**所有子查询并行执行**，每个子查询独立搜索，返回时必须标注元数据。

```
【每条素材必须标注】
- 信源类型：官方（一手）/ 权威媒体 / 自媒体（二手）/ 论坛讨论 / 学术论文
- 发布时间：精确到天（如 2026-04-08）
- 一手 vs 转述：是原始信息还是引用/转述他人信息
- 来源 URL：可追溯

【信源权重（整合时使用）】
  官方公告/一手数据  → 权重 5
  权威媒体深度报道    → 权重 4
  行业分析师/研究机构 → 权重 3
  自媒体/博客         → 权重 2
  论坛讨论/匿名爆料   → 权重 1

【时效衰减（AI/科技领域）】
  < 1 周     → 时效系数 1.0
  1-4 周     → 时效系数 0.8
  1-3 月     → 时效系数 0.5
  3-6 月     → 时效系数 0.3
  > 6 月     → 时效系数 0.1（除非是基础性/原理性内容）
```

---

### Layer 3：CoT 整合（链式推理）

**不许直接拼接素材**，必须走完以下推理链：

```
【Step 1：分组提炼】
按子查询分组，每组提炼 1-3 个核心发现。
去掉重复信息，保留最权威的来源。

【Step 2：交叉验证 → 置信度分级】
对每个核心结论进行多源验证：

  🟢 高置信（可直接引用）：
    - ≥ 2 个独立信源证实
    - 至少 1 个一手/官方来源
    - 信息时效在有效期内

  🟡 中置信（需标注"据报道"）：
    - 仅 1 个可靠信源
    - 或多个低权重信源一致
    - 逻辑上合理但缺乏直接证据

  🔴 低置信（必须标注"未经证实"）：
    - 仅单一低权重信源
    - 多源之间存在矛盾
    - 信息明显过时但无新信息替代

【Step 3：矛盾处理】
当信源之间存在直接矛盾时：
  1. 比较发布时间 — 新信息是否已推翻旧信息？
  2. 比较信源权重 — 官方声明 > 媒体报道 > 自媒体猜测
  3. 看是否有一手数据 — 有数据支撑的 > 纯观点
  4. 无法判断时 → 列出双方证据，标为 🔴，交由用户决定

【Step 4：识别信息盲区】
主动检查：
  - 哪些子查询没有找到有价值的信息？
  - 原始问题中有没有哪个维度完全没被覆盖？
  - 是否存在"所有信源都没提到但逻辑上应该存在"的信息？
  → 列为"信息盲区"，建议用户关注或后续追踪
```

---

## 输出格式

```markdown
## {原始问题}

### 核心结论

1. 🟢 {高置信结论 1}
   - 证据：{来源 A（日期）} + {来源 B（日期）}
2. 🟢 {高置信结论 2}
   - 证据：...

### 补充发现

3. 🟡 {中置信发现}
   - 据 {来源}（{日期}）报道：...
4. 🟡 ...

### 存疑/矛盾

5. 🔴 {矛盾点}
   - 观点 A：{内容}（{来源 + 日期}）
   - 观点 B：{内容}（{来源 + 日期}）
   - 判断倾向：{如有}

### 信息盲区

- {未能覆盖的维度或未找到的信息}
- 建议后续关注：{具体方向}

---
来源汇总：
- [{来源名}]({URL})（{日期}，{信源类型}）
- ...
```

---

## 场景适配

### AI 日报（每日定时）

```
日报模式：
- 时效要求：近 24 小时
- 拆解维度：技术突破 / 产品发布 / 融资动态 / 政策监管 / 社区热点
- 数据源：Twitter + HackerNews + Google News + 微信公众号（必须覆盖）
- 置信度要求：🟢 和 🟡 均可收录，🔴 标注后收录
- 输出简化：每条 1-2 句话 + 来源链接，不需要完整推理过程
```

### 临时调研（用户触发）

```
调研模式：
- 时效要求：根据问题判断
- 拆解维度：根据问题自动拆解
- 置信度要求：严格标注，🔴 必须明确提示
- 输出完整：走完整输出格式，包含推理过程
```

### 多源验证（关键结论校验）

```
验证模式：
- 输入：一个已有结论（而非问题）
- 目标：验证该结论是否可靠
- 拆解方式：正面证据搜索 + 反面证据搜索 + 原始出处追溯
- 输出：验证结果（证实 / 部分证实 / 存疑 / 证伪）+ 证据链
```

---

## 质量铁律

1. **每个结论必须有来源** — 没有来源的结论不许出现在输出中
2. **不许脑补** — LLM 自身知识只用于 HyDE 引导和逻辑推理，不用于事实陈述
3. **时效就是生命** — AI 领域 3 个月前的"最新消息"就是旧闻，必须标注日期
4. **宁可说"不确定"也不要瞎说** — 信息盲区是有价值的输出，不是失败
5. **矛盾是信号** — 发现矛盾不是坏事，是深度分析的起点，必须如实呈现

---

## 注意事项

1. **autocli** 已安装在 `~/bin/autocli`，所有命令加 `--format json` 获取结构化输出
2. **Browser Mode 命令**（Twitter、Google、知乎等）需要 Chrome 打开并登录对应网站
3. **Public Mode 命令**（HackerNews、BBC 等）无需登录，速度更快，优先使用
4. **rate limit**：同一网站短时间大量请求可能触发风控，每个站点每分钟不超过 10 次请求
5. **Google site: 语法**是搜索特定网站的万能方案，当某个站点没有专用命令时用这个
6. **autocli 不可用时的降级**：用 web-search skill（DuckDuckGo）或 brave-search / tavily 作为备选
