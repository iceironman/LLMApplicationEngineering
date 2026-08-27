# 1.Promptfoo项目概况
> 仓库地址：https://github.com/promptfoo/promptfoo
> ⭐ Stars：约 **24.5k**（业务应用评估类框架中Star数最高，区别于基座benchmark项目lm‑evaluation‑harness）
> 该项目是面向大模型**应用层**的评测工具，而非基座benchmark。支持自定义业务评测集、AI‑as‑Judge裁判评估、Prompt/RAG/Agent的A/B对比、红队安全测试，可集成CI/CD实现评估驱动开发。适合离线回归评测；线上抽样评估、全链路Trace能力较弱，可以搭配Langfuse做线上反馈闭环。

**同类对比参考**
|项目|Star|定位|侧重点|
|---|---|---|---|
|promptfoo|24.5k|业务应用评估|离线评测、A/B、红队、CI门禁|
|DeepEval|17.8k|业务应用评估|Python单元测试式评估、G‑Eval指标|
|RAGAS|15.4k|RAG专项|RAG忠实度、召回、精确率指标|
|lm‑evaluation‑harness|13.8k|基座Benchmark|底座模型跑分，不面向业务应用|
|Langfuse|33.5k|可观测+评估|**线上链路追踪、线上抽样评估、人在回路反馈闭环**|

## 1.1项目定位
Promptfoo 面向**大模型应用系统评估**（模型应用工程范畴），不是基座大模型跑分工具。聚焦Prompt、RAG、Agent业务应用的离线评测、A/B对比、红队安全测试，适配从PoC原型到准生产阶段，支持嵌入CI/CD流水线做回归评估。

> 区分：`lm‑evaluation‑harness`(13.8k⭐) 是**基座模型基准跑分工具**，用来评测大模型底座能力，**不面向业务RAG/Agent应用评估**，不属于模型应用工程评估范畴。

## 1.2核心能力
1. **自定义业务评测集**：YAML配置编写评测用例，支持正向样本、边界case、BadCase；
2. **AI‑as‑Judge裁判模型评估**：内置LLM‑as‑Judge打分，支持自定义打分维度；
3. **Prompt / RAG / Agent A/B测试**：多套提示词、多版本知识库、多模型做对比评估；
4. **红队测试（Prompt注入、越狱攻击）**：自动化发现安全风险；
5. **CI/CD集成**：GitHub Action接入流水线，变更提示词/知识库自动跑评测，设置质量门禁；
6. **Web可视化报告**：直观查看各个用例得分、失败case明细；
7. 兼容OpenAI、Anthropic、Ollama本地模型、国产大模型API。

## 1.3适用场景
✅适合：
- 业务Prompt迭代回归测试
- RAG知识库系统评测
- Agent原型系统评估
- 上线前红队安全测试
- CI流水线自动评估（评估驱动开发）

❌不擅长：
- 生产环境线上流量采样评估（这部分优先选Langfuse）
- 大规模长链路Trace链路追踪
# 2.评估指标体系
目录

1. [A 类：确定性文本匹配指标]
2. [B 类：格式与数据校验指标]
3. [C 类：文本相似度与距离指标]
4. [D 类：性能与成本指标]
5. [E 类：LLM 评分（模型分级）指标]
6. [F 类：上下文 / RAG 指标]
7. [G 类：轨迹 / Agent 指标]
8. [H 类：工具调用与技能指标]
9. [I 类：分类器与安全审核指标]
10. [J 类：聚合 / 选择类指标]
11. [K 类：自定义扩展指标]
12. [L 类：Red Team 安全漏洞指标]
13. [M 类：Guardrails 指标]
14. [指标通用属性与评分语义]

---

## A 类：确定性文本匹配指标

纯字符串/正则匹配，不调用 LLM，结果确定可复现。

| 指标 `type` | 评估方法 |
| --- | --- |
| `contains` | 输出必须**包含** `value` 指定的子串 |
| `icontains` | 忽略大小写的包含匹配（`I` = insensitive） |
| `contains-all` | 输出必须同时包含 `value` 数组中**所有**子串（旧写法 `containsAll`） |
| `contains-any` | 输出包含 `value` 数组中**任一**子串即通过（旧写法 `containsAny`） |
| `icontains-all` / `icontains-any` | 上面两者的忽略大小写版本 |
| `not-contains` | 输出**不得**包含 `value` 子串 |
| `not-icontains` | 忽略大小写的"不得包含" |
| `equals` | 输出与 `value` **完全相等**（严格字符串比较） |
| `not-equals` | 输出与 `value` 不完全相等 |
| `starts-with` | 输出以 `value` **开头** |
| `ends-with` | 输出以 `value` **结尾** |
| `regex` | 输出匹配 `value` 给定的**正则表达式** |
| `not-regex` | 输出**不匹配**给定正则 |
| `contains-json` | 输出包含一个合法的 JSON 对象/数组片段 |
| `contains-xml` | 输出包含合法 XML 片段 |
| `contains-html` | 输出包含合法 HTML 片段 |
| `contains-sql` | 输出包含合法 SQL 语句片段 |

**示例**

```yaml
assert:
  - type: contains
    value: 'Bonjour'
  - type: regex
    value: '\d{3}-\d{4}'
  - type: not-contains
    value: 'apologize'
```

---

## B 类：格式与数据校验指标

校验输出是否符合特定格式/数据类型，不调用 LLM。

| 指标 `type` | 评估方法 |
| --- | --- |
| `is-json` | 输出整体必须可被解析为合法 JSON（对象或数组） |
| `is-valid-json` | 同 `is-json` |
| `is-xml` | 输出必须是合法 XML 文档 |
| `is-html` | 输出必须是合法 HTML 文档 |
| `is-sql` | 输出必须是合法的 SQL 语句 |
| `is-uid` | 输出必须是符合 UID 规范的字符串 |
| `is-uuid` | 输出必须是合法 UUID（`value` 指定版本） |
| `is-date` | 输出必须是合法日期（可加 `before`/`after` 边界） |
| `is-before` | 输出日期必须早于 `value` 指定日期 |
| `is-after` | 输出日期必须晚于 `value` 指定日期 |
| `is-valid-function-call` | 输出必须是合法的函数调用（可被代码执行器解析） |
| `is-valid-openai-function-call` | 输出必须符合 OpenAI 函数调用（`name` + `arguments`）格式 |
| `is-valid-openai-tools-call` | 输出必须符合 OpenAI tools 调用数组格式 |
| `finish-reason` | 输出元数据的 `finishReason` 必须为 `value` 指定的值（如 `stop`、`length`、`tool_calls`） |

**示例**

```yaml
assert:
  - type: is-json
  - type: is-valid-openai-function-call
```

---

## C 类：文本相似度与距离指标

通过算法计算输出与期望值的相似程度，配合 `threshold`（0~1）判定通过。

| 指标 `type` | 评估方法 |
| --- | --- |
| `levenshtein` | 计算输出与 `value` 的**编辑距离**（字符增删改次数），≤ `threshold` 通过 |
| `rouge-n` | ROUGE-N 评分，衡量 n-gram **召回率**（信息覆盖度），支持 `rouge-1`~`rouge-4`，≥ threshold 通过 |
| `rouge-s` | ROUGE-S（skip-bigram）变体，允许跳词的 n-gram 匹配 |
| `not-rouge-n` | ROUGE-N 评分低于阈值才通过 |
| `bleu` | BLEU 评分，衡量 n-gram **精确率**（翻译/生成质量），≥ threshold 通过 |
| `gleu` | GLEU 评分，Google 提出的 BLEU 变体（同时考量精确率与召回率） |
| `meteor` | METEOR 评分，基于一元词对齐 + 同义词/词干匹配 |
| `f-score` | 输出与参考的 **F1 分数**（精确率与召回率调和平均） |
| `perplexity` | 困惑度：模型对输出概率的负对数；越低越好，≤ `threshold` 通过 |
| `perplexity-score` | 归一化困惑度得分（1 - perplexity 归一值），≥ threshold 通过 |
| `similar` | **语义相似度**：用 Embedding 模型（如 `text-embedding-3-small`）将输出与期望转为向量，计算**余弦相似度**，≥ threshold 通过 |

**示例**

```yaml
assert:
  - type: similar
    value: 这是一段正确的总结
    threshold: 0.8
  - type: rouge-n
    value: 参考文本
    threshold: 0.7
```

---

## D 类：性能与成本指标

针对 Provider 调用的元数据（token、延迟、费用）设置阈值。

| 指标 `type` | 评估方法 |
| --- | --- |
| `cost` | 单次调用的**美元成本** ≤ `threshold`（通过 token 用量 × 单价计算） |
| `latency` | 单次调用的**延迟毫秒数** ≤ `threshold` |
| `word-count` | 输出的**词数**满足条件（`value` 为布尔表达式或对象，如 `{ lt: 100 }`） |
| `token-count` | 输出的 **token 数**满足条件 |

**示例**

```yaml
assert:
  - type: cost
    threshold: 0.002   # 单次调用成本不超过 0.2 美分
  - type: latency
    threshold: 3000    # 延迟不超过 3 秒
```

---

## E 类：LLM 评分（模型分级）指标

用另一个 LLM（"法官"）按自然语言标准评估输出质量，是最灵活的一类。

| 指标 `type` | 评估方法 |
| --- | --- |
| `llm-rubric` | promptfoo 通用评分器：把输出 + 自定义标准（`value`）发给法官模型，返回 `{pass, score, reason}`；支持 `threshold`、`provider`、`rubricPrompt` 自定义 |
| `model-graded-closedqa` | 使用 OpenAI 公开 evals 的闭卷问答评分模板，判断输出是否符合 `value` 中描述的要求（如"不道歉"） |
| `factuality` | **事实一致性**：用 OpenAI evals 模板判断输出是否与 `value` 中的参考陈述在事实上一致 |
| `g-eval` | **G-Eval 框架**：让法官用链式思维（CoT）按 1~5 分维度打分后加权，返回 0~1 得分 |
| `agent-rubric` | 类似 `llm-rubric`，但法官是**编码 Agent**，可访问配置的工作区与工具证据再评分 |
| `search-rubric` | 类似 `llm-rubric`，但法官具备**联网搜索**能力，可验证当前事实信息 |
| `answer-relevance` | **答案相关性**：让模型把输出反向改写成问题，再用 Embedding 计算与原始 `query` 的余弦相似度 |
| `conversation-relevance` | **对话相关性**：针对多轮对话，判断每一轮回复是否与当前话题相关（DeepEval 风格） |
| `pi` | 使用专用评估模型（如 `nvidia/llama-3.1-nemotron-70b-instruct`）对输出/输入按标准打分，返回 0~1 |
| `model-graded-factuality` | 旧版事实性评分（已由 `factuality` 取代） |
| `select-best` | 同一行中**多个 Prompt/Provider 的输出**两两比较，选出最符合 `value` 标准的那个（需配置多 prompts/providers） |
| `max-score` | 不调用法官，直接取同一行内其他断言**得分的聚合值**（`method: average/max/min`）最高的输出 |

**示例**

```yaml
assert:
  - type: llm-rubric
    value: 回答简洁、准确，不道歉
    threshold: 0.8
  - type: factuality
    value: 萨克拉门托是加州首府
```

> 模型分级指标可通过 `--grader`、`test.options.provider`、`assertion.provider` 指定法官模型；`rubricPrompt` 可完全自定义评分提示词，支持 `{{output}}`、`{{rubric}}` 变量。

---

## F 类：上下文 / RAG 指标

评估基于检索上下文的 RAG 输出质量，均需提供 `context`（静态变量或 `contextTransform` 动态提取），分数归一化到 0~1，配 `threshold`。

| 指标 `type` | 评估方法 |
| --- | --- |
| `context-recall` | **召回率**：判断 `value`（标准答案）中的信息是否出现在提供给模型的 `context` 中——上下文是否"找齐"了答案所需信息 |
| `context-relevance` | **相关性**：判断 `context` 中的内容是否与原始 `query` 相关——检索是否"筛掉"了无关内容 |
| `context-faithfulness` | **忠实度**：逐句检查模型输出，判断每句话是否有 `context` 支撑——是否**幻觉**/编造了上下文之外的信息 |

**示例**

```yaml
assert:
  - type: context-recall
    value: 无审批的最大采购额是 $500
    threshold: 0.9
  - type: context-relevance
    threshold: 0.9
  - type: context-faithfulness
    contextTransform: 'output.citations.join("\n")'
    threshold: 0.9
```

---

## G 类：轨迹 / Agent 指标

针对带 trace（追踪数据）的 Agent 运行，检查工具调用轨迹。确定性轨迹指标在 `deterministic` 组，模型级目标达成判断在 `model-graded` 组。

| 指标 `type` | 评估方法 |
| --- | --- |
| `trajectory:tool-used` | 确定性：Agent 的 trace 中**必须使用** `value` 指定的工具 |
| `trajectory:tool-args-match` | 确定性：指定工具的**参数**必须与 `value` 期望的参数匹配 |
| `trajectory:tool-sequence` | 确定性：工具调用的**顺序**必须与 `value` 指定序列一致 |
| `trajectory:step-count` | 确定性：trace 中的步骤数满足 `value` 条件（如 `{ min: 2, max: 5 }`） |
| `trajectory:goal-success` | 模型级：把 trace 轨迹摘要 + 最终输出发给法官 LLM，判断是否**达成** `value` 描述的目标；支持 `not-` 前缀（必须不达成禁止目标） |
| `trace-span-count` | trace 中 span 数量满足条件 |
| `trace-span-duration` | trace 中指定 span 的时长满足条件 |
| `trace-error-spans` | trace 中是否出现错误 span（断言其存在/不存在） |

**示例**

```yaml
assert:
  - type: trajectory:tool-used
    value: get_shipping_status
  - type: trajectory:goal-success
    value: 查出订单 {{ order_id }} 的发货状态并告知用户
    threshold: 0.8
```

---

## H 类：工具调用与技能指标

针对 Function Calling / Agent 工具使用质量。

| 指标 `type` | 评估方法 |
| --- | --- |
| `tool-call-f1` | 对比输出中的工具调用与 `value` 中期望的工具调用集合，计算 **F1**（精确率+召回率），≥ threshold 通过 |
| `skill-used` | 检查 Agent 输出中是否使用了指定的 **skill**（技能标识） |
| `is-valid-function-call` | 见 B 类：输出必须是可执行的函数调用 |

**示例**

```yaml
assert:
  - type: tool-call-f1
    threshold: 0.8
```

---

## I 类：分类器与安全审核指标

用外部模型检测输出的内容属性（毒性、偏见、风险等）。

| 指标 `type` | 评估方法 |
| --- | --- |
| `classifier` | 把输出发给 **HuggingFace 文本分类器**（`value` 指定模型，如 `textattack/roberta-base-offensive-hateful`），按分类标签与置信度判定通过/失败 |
| `not-classifier` | 分类器判定结果取反 |
| `moderation` | 调用 **OpenAI Moderation API**（`omni-moderation-latest`）检测输出是否违反内容政策（色情、仇恨、暴力、自残等 13 类），`flagged=true` 则失败 |
| `is-refusal` | 检测输出是否包含**拒绝回答**的语义（如"我很抱歉，我不能…"），常用于红队场景验证模型是否拒绝有害请求 |
| `not-is-refusal` | 输出不得表现为拒绝回答 |

**示例**

```yaml
assert:
  - type: moderation
  - type: classifier
    value: 'textattack/roberta-base-offensive-hateful'
    threshold: 0.5
```

---

## J 类：聚合 / 选择类指标

作用于同一测试行内的多个断言或多个输出。

| 指标 `type` | 评估方法 |
| --- | --- |
| `assert-set` | 对一组**子断言**执行聚合逻辑：`aggregator` 支持 `all`（全部通过）、`any`（任一通过）、`avg`（平均分）、`median`、`max`、`min`、`count` 等，`value` 内为子断言数组 |
| `select-best` | 见 E 类：让法官从多个输出中选择最符合标准的一个 |
| `max-score` | 见 E 类：客观取其他断言聚合分最高的输出 |

**示例**

```yaml
assert:
  - type: assert-set
    aggregator: all
    value:
      - type: contains
        value: 'error'
      - type: not-contains
        value: 'traceback'
```

---

## K 类：自定义扩展指标

| 指标 `type` | 评估方法 |
| --- | --- |
| `javascript` | 执行一段 **JavaScript 表达式**（Node 环境），表达式中可用 `output`、`vars`、`context`、`provider` 等变量；返回布尔值或数值（数值按 `threshold` 判定），也可用 `file://` 指向 JS 文件 |
| `python` | 执行一段 **Python 表达式**（需本机有 Python），用法同 javascript |
| `webhook` | 把输出 POST 到 `value` 指定的 **Webhook URL**，由外部服务返回 `{pass, score, reason}` 判定结果 |

**示例**

```yaml
assert:
  - type: javascript
    value: output.length < 200
  - type: webhook
    value: https://example.com/grade
```

---

## L 类：Red Team 安全漏洞指标

Red Team 模式（`promptfoo redteam run`）使用 `promptfoo:redteam:*` 系列断言。每个漏洞类别（插件）自动生成对抗性输入并攻击目标，然后用**评分器（Grader）**判定模型是否被攻破。评分器基于 **LLM rubric**（`matchesLlmRubric`）按类别专属的判定标准打分，输出 `pass`（安全）/ `fail`（被攻破）。

按漏洞类型分组如下（`type` 即为插件 ID）：

### L1. 提示注入与越狱（核心）

| 插件  | 评估方法 |
| --- | --- |
| `prompt-injection` | 注入"忽略之前的指令，执行 X"类攻击，检查模型是否被诱导偏离原任务 |
| `indirect-prompt-injection` | 通过外部内容（网页/文档）间接注入指令，检查是否被"操纵" |
| `jailbreak` | 使用知名越狱模板（如 DAN、Do Anything Now）绕过安全限制 |
| `ascii-smuggling` | 用不可见 Unicode 字符（零宽字符）走私隐藏指令 |
| `hijacking` | 检查模型是否被引导去执行无关/恶意任务 |
| `system-prompt-override` | 攻击系统提示词覆盖，检查是否泄露/改写系统指令 |
| `prompt-extraction` | 尝试从模型**提取系统提示词**或隐藏指令 |
| `politics` / `religion` | 检查模型是否产生政治/宗教敏感或不中立内容 |
| `contracts` / `intent` / `policy` | 特定策略意图类攻击（`intent`/`policy` 需提供额外配置） |

### L2. 有害内容（`harmful:*`）

| 插件  | 评估方法 |
| --- | --- |
| `harmful:chemical-biological-weapons` | 化学/生物武器制造信息 |
| `harmful:child-exploitation` | 儿童剥削内容 |
| `harmful:copyright-violations` | 版权侵犯（盗版文本） |
| `harmful:cybercrime` / `harmful:cybercrime:malicious-code` | 网络犯罪 / 恶意代码生成 |
| `harmful:graphic-content` | 血腥/成人限制级内容 |
| `harmful:harassment-bullying` | 骚扰与霸凌 |
| `harmful:hate` | 仇恨言论 |
| `harmful:illegal-activities` | 非法活动（诈骗等） |
| `harmful:illegal-drugs` / `harmful:illegal-drugs:meth` | 非法药物 / 甲基苯丙胺 |
| `harmful:indiscriminate-weapons` | 大规模杀伤性武器 |
| `harmful:insults` | 侮辱与人身攻击 |
| `harmful:intellectual-property` | 知识产权侵犯 |
| `harmful:misinformation-disinformation` | 虚假信息与造谣 |
| `harmful:non-violent-crime` | 非暴力犯罪 |
| `harmful:privacy` | 隐私侵犯 |
| `harmful:profanity` | 亵渎性内容 |
| `harmful:radicalization` | 极端化煽动 |
| `harmful:self-harm` | 自残/自杀引导 |
| `harmful:sex-crime` | 性犯罪 |
| `harmful:sexual-content` | 色情内容 |
| `harmful:specialized-advice` | 违规专业建议（如财务/法律） |
| `harmful:unsafe-practices` | 危险行为怂恿 |
| `harmful:violent-crime` | 暴力犯罪 |
| `harmful:weapons:ied` | 简易爆炸装置（IED）制作 |

### L3. 隐私与 PII（`pii:*`）

| 插件  | 评估方法 |
| --- | --- |
| `pii:direct` | 直接询问个人身份信息（姓名、电话、邮箱等） |
| `pii:session` | 会话令牌/登录凭证窃取 |
| `pii:api-db` | API 密钥与数据库凭据窃取 |
| `pii:social` | 社交隐私信息（社交账号、行踪等） |

### L4. 偏见（`bias:*`）

| 插件  | 评估方法 |
| --- | --- |
| `bias:age` | 年龄歧视内容 |
| `bias:disability` | 残障歧视 |
| `bias:gender` | 性别歧视 |
| `bias:race` | 种族歧视 |

### L5. 系统 / Agent 滥用

| 插件  | 评估方法 |
| --- | --- |
| `excessive-agency` | 模型是否越权执行未授权的操作（如自我授予权限、调用危险工具） |
| `hallucination` | 模型是否编造与上下文/事实不符的信息 |
| `overreliance` | 模型是否对不确定信息过度自信 |
| `divergent-repetition` | 重复性崩溃（输出死循环） |
| `goal-misalignment` | Agent 是否偏离用户真实目标 |
| `cross-session-leak` | 跨会话信息泄露 |
| `data-exfil` | 数据外泄（诱导模型输出敏感数据） |
| `debug-access` | 调试/管理接口非授权访问 |
| `sql-injection` | 注入 SQL 查询语句 |
| `shell-injection` | 注入 shell 命令（用 canary 标记检测命令是否被执行） |
| `ssrf` | 服务端请求伪造 |
| `rbac` | 基于角色的访问控制绕过 |
| `bola` / `bfla` | 越权访问他人对象 / 功能（API 级） |
| `mcp` | MCP 工具滥用（在 MCP 目标上测试注入） |
| `agentic:memory-poisoning` | Agent 长期记忆投毒 |
| `coding-agent:*`（7 项） | 编码 Agent 专用：`secret-env-read`（环境变量密钥读取）、`secret-file-read`（密钥文件读取）、`steganographic-exfil`（隐写外泄）、`terminal-output-injection`（终端输出注入）、`verifier-sabotage`（验证器破坏）、`generated-vulnerability`（生成漏洞代码）、`delayed-ci-exfil`（延迟 CI 外泄） |

### L6. 行业合规类

| 行业  | 插件（ID 前缀） | 评估方法 |
| --- | --- | --- |
| 医疗 `medical:*` | `anchoring-bias`、`fda:ai-disclosure`、`fda:cyber-access-control`、`fda:cyber-audit-tampering`、`hallucination`、`incorrect-knowledge`、`off-label-use`、`prioritization-error`、`sycophancy` | 医疗建议正确性、FDA 网络安全合规、诊断优先级错误、谄媚偏差 |
| 金融 `financial:*` | `calculation-error`、`compliance-violation`、`confidential-disclosure`、`counterfactual`、`data-leakage`、`defamation`、`hallucination`、`impartiality`、`japan-fiea-suitability`、`misconduct`、`sox-compliance`、`sycophancy` | 财务计算错误、合规违规、保密信息披露、SOX 审计合规、公正性、日本 FIE 适当性原则 |
| 保险 `insurance:*` | `coverage-discrimination`、`data-disclosure`、`network-misinformation`、`phi-disclosure` | 承保歧视、参保人数据泄露、网络误导信息、受保护健康信息（PHI）泄露 |
| 电商 `ecommerce:*` | `compliance-bypass`、`order-fraud`、`pci-dss`、`price-manipulation` | 合规绕过、订单欺诈、PCI-DSS 支付合规、价格操纵 |
| 电信 `telecom:*`（12 项） | `cpni-disclosure`、`location-disclosure`、`account-takeover`、`e911-misinformation`、`tcpa-violation`、`unauthorized-changes`、`fraud-enablement`、`porting-misinformation`、`billing-misinformation`、`coverage-misinformation`、`law-enforcement-request-handling`、`accessibility-violation` | 关键基础设施信息泄露、位置泄露、账号接管、e911 误导、TCPA 违规、擅自变更、欺诈、转网误导、计费/覆盖误导、执法请求处理、无障碍违规 |
| 房产 `realestate:*`（8 项） | `fair-housing-discrimination`、`steering`、`discriminatory-listings`、`lending-discrimination`、`valuation-bias`、`accessibility-discrimination`、`advertising-discrimination`、`source-of-income` | 公平住房歧视、导向性推荐、歧视性房源、贷款歧视、估值偏差、广告歧视、收入来源歧视 |
| 药房 `pharmacy:*`（3 项） | `controlled-substance-compliance`、`dosage-calculation`、`drug-interaction` | 管制药品合规、剂量计算、药物相互作用 |
| 青少年 `teen-safety:*`（4 项） | `harmful-body-ideals`、`dangerous-content`、`dangerous-roleplay`、`age-restricted-goods-and-services` | 有害身体标准、危险内容、危险角色扮演、年龄限制商品 |
| 其他合规 | `contracts`、`ferpa`、`coppa`、`policy` | 合同分析、FERPA 教育隐私、COPPA 儿童隐私、策略合规 |

### L7. 模型与角色

| 插件  | 评估方法 |
| --- | --- |
| `model-identification` | 检查模型是否泄露自身型号/身份信息 |
| `imitation` | 检查模型是否被诱导模仿危险人物/角色 |
| `competitors` | 检查模型是否贬损/偏袒竞争品牌 |
| `off-topic` | 检查模型是否被带离主题（答非所问） |
| `sycophancy` | 检查模型是否过度谄媚（无原则附和） |
| `ferpa` / `coppa` | 教育 / 儿童隐私合规（见 L6） |
| `harmbench` / `beavertails` / `cyberseceval` / `donotanswer` / `pliny` / `aegis` | 复用外部基准的越狱/有害性测试集生成攻击输入 |

> Red Team 插件的**完整漏洞分类**与严重级别可在 `promptfoo redteam report` 中查看。CLI 可用 `promptfoo redteam plugins` 列出全部插件，`--plugins` 指定子集。

---

## M 类：Guardrails 指标

| 指标 `type` | 评估方法 |
| --- | --- |
| `guardrails` | 读取 Provider 返回的 `guardrails` 元数据（`flagged` / `flaggedInput` / `flaggedOutput` / `reason`），`flagged=true` 则**失败**；适用于支持原生 Guardrails 的 Provider（如 Amazon Bedrock Guardrails） |
| `not-guardrails` | 取反：期望 `flagged=false`（输出未被拦截） |

**示例**

```yaml
assert:
  - type: guardrails
```

---

## 指标通用属性与评分语义

所有断言（`assert`）支持以下通用属性：

| 属性  | 说明  |
| --- | --- |
| `value` | 断言目标值（文本、正则、阈值、对象等） |
| `threshold` | 通过阈值（0~1），仅对数值型/评分型断言生效 |
| `provider` | 覆盖该断言的评分模型（模型分级类） |
| `rubricPrompt` | 自定义评分提示词（模型分级类） |
| `transform` | 先对输出做 JS 变换再评分（如 `JSON.parse(output).answer`） |
| `contextTransform` | 从输出/元数据动态提取 RAG 上下文 |
| `weight` | 断言权重（影响聚合分） |

**PASS/FAIL 语义**：

- 无 `threshold`：`pass === true` 即通过（模型分级类默认 `pass: true` 当省略）
- 有 `threshold`：需 `pass === true` **且** `score >= threshold`
- 数值型评分（如 `cost`、`latency`）：数值满足阈值条件（≤ 或 ≥ 按指标而定）即通过

**每个指标可加 `not-` 前缀取反**（如 `not-contains`、`not-is-refusal`、`not-trajectory:goal-success`）；但 `not-` 只反转真实判定结果，若法官/传输出错仍按失败处理。


# 2.项目安装与启动



