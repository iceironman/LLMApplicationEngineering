# 1.Promptfoo项目概况
> 仓库地址：https://github.com/promptfoo/promptfoo
> ⭐ Stars：约 **24.5k**（业务应用评估类框架中Star数最高，区别于基座benchmark项目lm‑evaluation‑harness）
## 1.1项目定位
Promptfoo 面向**大模型应用系统评估**（模型应用工程范畴），不是基座大模型跑分工具。聚焦Prompt、RAG、Agent业务应用的离线评测、A/B对比、红队安全测试，适配从PoC原型到准生产阶段，支持嵌入CI/CD流水线做回归评估。

> 区分：`lm‑evaluation‑harness`(13.8k⭐) 是**基座模型基准跑分工具**，用来评测大模型底座能力，**不面向业务RAG/Agent应用评估**，不属于模型应用工程评估范畴。

## 核心能力（对应教程知识点）
1. **自定义业务评测集**：YAML配置编写评测用例，支持正向样本、边界case、BadCase；
2. **AI‑as‑Judge裁判模型评估**：内置LLM‑as‑Judge打分，支持自定义打分维度；
3. **Prompt / RAG / Agent A/B测试**：多套提示词、多版本知识库、多模型做对比评估；
4. **红队测试（Prompt注入、越狱攻击）**：自动化发现安全风险；
5. **CI/CD集成**：GitHub Action接入流水线，变更提示词/知识库自动跑评测，设置质量门禁；
6. **Web可视化报告**：直观查看各个用例得分、失败case明细；
7. 兼容OpenAI、Anthropic、Ollama本地模型、国产大模型API。

## 适用场景（教程可以直接摘抄）
✅适合：
- 业务Prompt迭代回归测试
- RAG知识库系统评测
- Agent原型系统评估
- 上线前红队安全测试
- CI流水线自动评估（评估驱动开发）

❌不擅长：
- 生产环境线上流量采样评估（这部分优先选Langfuse）
- 大规模长链路Trace链路追踪

## 简单最小示例配置片段（YAML）
```yaml
prompts:
  - file://prompts/system_prompt.txt
  - file://prompts/system_prompt_v2.txt

tests:
  - vars:
      query: "模型应用工程的核心命题是什么？"
    assert:
      - type: llm‑judge
        value: "回答必须提到概率性输出转化为可控业务系统"
```

## 在你的教程第3章可以写入的说明
> **⭐最高Star业务评估开源项目：promptfoo**
> 该项目是面向大模型**应用层**的评测工具，而非基座benchmark。支持自定义业务评测集、AI‑as‑Judge裁判评估、Prompt/RAG/Agent的A/B对比、红队安全测试，可集成CI/CD实现评估驱动开发。适合离线回归评测；线上抽样评估、全链路Trace能力较弱，可以搭配Langfuse做线上反馈闭环。

### 同类对比参考（方便教程表格）
|项目|Star|定位|侧重点|
|---|---|---|---|
|promptfoo|24.5k|业务应用评估|离线评测、A/B、红队、CI门禁|
|DeepEval|17.8k|业务应用评估|Python单元测试式评估、G‑Eval指标|
|RAGAS|15.4k|RAG专项|RAG忠实度、召回、精确率指标|
|lm‑evaluation‑harness|13.8k|基座Benchmark|底座模型跑分，不面向业务应用|
|Langfuse|33.5k|可观测+评估|**线上链路追踪、线上抽样评估、人在回路反馈闭环**|

> 补充：Langfuse star更高，但它**主定位是可观测/线上反馈平台**，离线批量评测不是它的核心；promptfoo专门聚焦**离线评估流水线**，属于纯评估工具，所以选promptfoo作为“模型应用评估”代表项目。

如果你需要，我可以把这段整理成Markdown小节直接插入你的第三章“拓展阅读：开源评估工具”。
