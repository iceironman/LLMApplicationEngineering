ls# 1.项目介绍
<div dig="center">
<img width="9798" height="4296" alt="how-does-ai-memory-work-gray" src="https://github.com/user-attachments/assets/ee8d14fd-d35e-4cd2-bd3f-6562eeb33814" />
</div>
项目地址：[https://www.zdoc.app/zh/topoteretes/cognee](https://www.zdoc.app/zh/topoteretes/cognee)

定位：**开源 AI 记忆平台，用来落地上下文工程范式**；不再手动拼接 prompt 字符串，由平台完成记忆摄入、知识图谱构建、动态检索、上下文组装、会话管理，把上下文工程的理论变成可运行代码。

前置理论回顾：
- 上下文工程核心，系统化管理送入模型上下文窗口全部信息；
- 包含系统指令、检索知识、对话历史、工具结果、示例、输出约束；
- 在有限 Token 预算，最大化信息密度与相关性。
  
Cognee 就是这套思想的开源工程实现。Cognee 是面向 AI 智能体的开源**AI 记忆平台**，Python 为主，同时支持 Rust、TypeScript 客户端。

- 核心能力：摄入多格式原始数据，自动构建**向量 + 知识图谱混合记忆**；
- 支持会话记忆与跨会话持久长期记忆；
- 根据用户查询自动召回高相关记忆片段，组装成高质量上下文交给大模型腾讯云。

Cognee 通过三段式流水线完成知识库构建：Ingestion 接入多源原始数据，Processing 执行分块、向量化、实体关系抽取，生成向量与知识图谱；Enrichment 完成知识推理与迭代优化。底层同时对接向量库、图谱库、关系库等多存储；上层提供 SDK、API、GQL 供业务调用。带*标记的功能属于开发中特性。
<img width="1148" height="586" alt="cognee_diagram" src="https://github.com/user-attachments/assets/cfc1cd0d-b722-4d99-88c1-0b8807c021d8" />

和上下文工程理论一一映射：
| 上下文工程理论模块 | Cognee 实现 |
| ---- | ---- |
| 动态上下文组装 | `recall()` 根据 query 自动召回记忆片段，输出可送入 LLM 的上下文文本 |
| Token 预算与信息密度 | 混合检索做相关性过滤，只返回高相关片段，避免把全部文档塞进 prompt，解决 Lost‑in‑the‑Middle 中间遗忘效应 |
| 会话 / 多轮历史管理 | `session_id` 区分会话，短期会话缓存 + 长期图谱持久化，实现对话记忆压缩与存储 |
| 私有知识接入 | RAG 能力，PDF / 文本 / CSV 等多源数据通过 `remember()` 摄入，构建私有知识记忆库 |
| 动态 Few‑shot、示例选择 | 基于语义检索，自动挑选相关历史案例送入上下文 |
| 来源可追溯 | 全部记忆携带文档溯源元数据，方便做输出校验、引用来源 |

>关键认知：Cognee**不是大模型，不是 Agent 框架**；它是独立的**记忆 / 上下文工程中间层**，可以对接 LangGraph、Claude Code、AutoGen 等各类 Agent 框架，专门负责 “怎么把外部资料、会话历史变成高质量模型输入上下文”，正好对应第四章上下文工程>的核心目标。
将用户输入转成图结构
![原理图：转成图结构](../assets/remember.svg)
召回过程
![原理图：召回过程](../assets/recall.svg)

# 2.核心三层存储架构
Cognee 采用三类存储协同工作，本地开发可以完全嵌入式运行，不需要额外部署数据库；生产环境可替换为 Postgres、Neo4j、PGVector、Qdrant 等组件。

 1. 关系存储（Relational Store）：SQLite/Postgres。存储文档元数据、切片、溯源信息、租户权限；记录记忆来自哪里，实现审计、可追溯。
 2. 向量存储（Vector Store）：LanceDB / PGVector / Qdrant。存储 embedding，做语义相似度检索。
 3. 图谱存储（Graph Store）：Kuzu / Neo4j。提取实体、实体之间关系，支持多跳推理检索，弥补纯向量 RAG 只能做相似度匹配的短板。
 4. 简单理解：向量找语义相似内容；知识图谱找实体之间关联；关系库保证来源、权限、审计。三者输出合并，筛选出高相关性片段，作为 LLM 的输入上下文。
# 3.四大核心 API（1.0 版本统一接口）
全部为异步 API，是做上下文工程的核心操作：

## 3.1.await cognee.remember(content, session_id=None)
记忆写入。支持原始文本、文件路径。
- 不传session_id：写入持久知识图谱长期记忆；
- 传入session_id：写入会话短期记忆，后台异步同步到图谱。
对应上下文工程：私有知识录入、会话对话历史存入记忆层。
## 3.2.await cognee.recall(query, session_id=None)
  
记忆召回。自动路由检索策略（向量检索 / 图遍历混合检索），返回已经过滤、排序完成的相关记忆片段。拿到的结果直接拼接到 Prompt 送入大模型。
>对应上下文工程：动态上下文组装、相关性过滤，提升信息密度，控制 Token 消耗。
## 3.3.await cognee.improve()
记忆自我优化。剪枝无效节点、强化高频实体关系权重，迭代优化知识图谱质量。
对应上下文工程：记忆持续迭代优化。
## 3.4 await cognee.forget(dataset="xxx")
删除记忆；支持按数据集、会话清除，实现记忆生命周期管理。
> 对应上下文工程：清理过期上下文数据。
# 4.Cognee实战
<div dig="center">
<img width="640" height="320" alt="cognee部署图" src="https://github.com/user-attachments/assets/7a0c9cc5-d5ff-4206-86e9-5188f404fcf7" />
</div>
## 4.1启动llama.cpp服务
服务端版本：0.1.1-dev
```bash
qixin@qixin:~/llama.cpp$ ./build/bin/llama-server --version
version: 0.1.1-dev (build 1, commit 01818e4)
built with GNU 15.2.0 for Linux x86_64
```
### 4.1.1 LLM模型服务：

```bash
qixin@qixin:~/llama.cpp$ ./build/bin/llama-server -m ./models/Qwen3-0.6B-Q8_0.gguf -c 8192 --host 0.0.0.0 --port 8080

```
**输出：**
```
0.00.004.584 I cmn  common_param: common_params_print_info: verbosity = 3 (adjust with the `-lv N` CLI arg)
0.00.018.653 W srv  llama_server: -----------------
0.00.018.658 W srv  llama_server: CORS is set to allow all origins ('*') and no API key is set
0.00.018.659 W srv  llama_server: this can be a security risk (cross-origin attacks)
0.00.018.659 W srv  llama_server: more info: https://github.com/ggml-org/llama.cpp/pull/25655
0.00.018.660 W srv  llama_server: -----------------
0.00.034.802 I srv    load_model: loading model './models/Qwen3-0.6B-Q8_0.gguf'
0.00.900.015 W load: control-looking token: 128247 '</s>' was not control-type; this is probably a bug in the model. its type will be overridden
0.07.206.009 I cmn          init: llama threadpool init, n_threads = 2
0.07.681.017 I srv    load_model: initializing, n_slots = 4, n_ctx_slot = 8192, kv_unified = 'true'
0.07.776.606 I srv  llama_server: model loaded
0.07.776.622 I srv  llama_server: listening on http://0.0.0.0:8080
0.07.776.623 W srv  llama_server: NOTICE: server default port will be changed to :9931 in a future release
0.07.776.624 W srv  llama_server:         ref: https://github.com/ggml-org/llama.cpp/pull/26508

```
**验证：**
<img width="1501" height="941" alt="image" src="https://github.com/user-attachments/assets/7ea09acd-a7d6-4996-8b4e-faed02dc2da1" />


### 4.1.2 embeding服务：
```bash
qixin@qixin:~/llama.cpp$ ./build/bin/llama-server -m ./models/Qwen3-Embedding-0.6B-Q8_0.gguf -c 2048 --host 0.0.0.0 --port 8081 --embeddings
```
**输出：**
```
0.00.006.116 I cmn  common_param: common_params_print_info: verbosity = 3 (adjust with the `-lv N` CLI arg)
0.00.006.136 W srv  llama_server: embeddings enabled with n_batch (2048) > n_ubatch (512)
0.00.006.137 W srv  llama_server: setting n_batch = n_ubatch = 512 to avoid assertion failure
0.00.006.374 W srv  llama_server: -----------------
0.00.006.378 W srv  llama_server: CORS is set to allow all origins ('*') and no API key is set
0.00.006.378 W srv  llama_server: this can be a security risk (cross-origin attacks)
0.00.006.379 W srv  llama_server: more info: https://github.com/ggml-org/llama.cpp/pull/25655
0.00.006.379 W srv  llama_server: -----------------
0.00.007.985 I srv    load_model: loading model './models/Qwen3-Embedding-0.6B-Q8_0.gguf'
0.00.738.618 W load: control-looking token: 128247 '</s>' was not control-type; this is probably a bug in the model. its type will be overridden
0.01.027.147 I cmn          init: llama threadpool init, n_threads = 2
0.01.395.697 I srv    load_model: initializing, n_slots = 4, n_ctx_slot = 2048, kv_unified = 'true'
0.01.410.178 I srv  llama_server: model loaded
0.01.410.187 I srv  llama_server: listening on http://0.0.0.0:8081
0.06.825.016 I slot get_availabl: id  3 | task -1 | selected slot by LRU, t_last = -1
0.06.825.037 I slot launch_slot_: id  3 | task 0 | processing task, is_child = 0
0.07.038.819 I slot      release: id  3 | task 0 | stop processing: n_tokens = 3, truncated = 0
```
**验证：向量维度默认1024**
<img width="974" height="891" alt="image" src="https://github.com/user-attachments/assets/40f0ca28-87be-4d4a-94a8-46ba1cd14a5c" />
## 4.2 cognee基本功能测试
### 4.2.1 安装cognee
**将cognee项目clone到本地**
```
git https://github.com/topoteretes/cognee.git
mkdir cognee_work
uv pip install -e .\cognee#导入cognee依赖包
```
**将cognee_work目录下的.env.template文件拷贝到cognee_work目录下，文件名改为.env**
### 4.2.2 配置cognee环境参数.env
**和cognee同级建一个cognee_work目录，在这个目录下用uv初始化虚拟环境**
LLM模型[Qwen3-0.6B-Q8_0.gguf]、EMBEDDING[Qwen3-Embedding-0.6B]模型、向量数据库[turso]和图数据库[networkx]。
```
###############################################################################
# TIER 1 — QUICK START
# Set this one variable and you're done. Everything else has working defaults.
# Default databases (SQLite, LanceDB, KuzuDB) are file-based, no setup needed.
###############################################################################
# llama.cpp 不校验 key，但 cognee 的 custom provider 要求非空，占位即可
LLM_API_KEY="sk-local-llama"

# 单用户本地部署：关闭多租户鉴权，避免 REQUIRE_AUTHENTICATION 被强制开启
ENABLE_BACKEND_ACCESS_CONTROL=false

# 跳过 LLM/embedding 连接预检（30s 超时对慢速 llama.cpp 偏短，端点可用性请
# 自行用 curl http://192.168.1.39:8080/v1/models 与 .../8081/v1/models 验证）
COGNEE_SKIP_CONNECTION_TEST=true #本地模型反应慢，不做连通性测试


###############################################################################
# TIER 2 — COMMON OVERRIDES (uncomment to customize)
# Most users only need a few of these.
###############################################################################

# -- LLM Provider & Model ----------------------------------------------------
# llama.cpp 服务（OpenAI 兼容端点）。model 字段由服务端忽略，可保留占位。
LLM_MODEL="openai/local-llm"
LLM_PROVIDER="custom"
LLM_ENDPOINT="http://192.168.1.39:8080/v1"
EMBEDDING_PROVIDER="openai_compatible"
EMBEDDING_MODEL="Qwen3-Embedding-0.6B"
EMBEDDING_ENDPOINT="http://192.168.1.39:8081/v1"
EMBEDDING_API_KEY=sk-dumy
EMBEDDING_DIMENSIONS=1024
HUGGINGFACE_TOKENIZER="Qwen/Qwen3-Embedding-0.6B"
```
修改.env参数后要，清除原理的cognee环境
```bash
rm -rf ~/.cognee
```
### 4.2.3 cognee测试
测试cognee的remember和recall接口
```python
"""
最简单的 cognee 端到端冒烟测试：
    remember()  把一段文字写入知识库（切块 -> LLM 实体/关系抽取 -> 图 -> 向量化）
    recall()    基于该知识检索问答

配置来源：本目录下的 .env（llama.cpp LLM 192.168.1.39:8080，embedding 192.168.1.39:8081）。

运行方法（先确保 cognee 已安装，例如：pip install -e ..\\cognee）：
    cd d:/032-Cognee/cognee_work
    python test_cognee.py
"""

import asyncio
import traceback
from pathlib import Path

import dotenv

# 显式加载本文件同目录的 .env，避免工作目录不同导致读不到配置
dotenv.load_dotenv(Path(__file__).resolve().parent / ".env", override=True)

import cognee  # noqa: E402
from cognee import SearchType  # noqa: E402

# 重复运行同一 dataset 会重复入库，需要彻底清空时放开下一行
# await cognee.forget(everything=True)

SAMPLE_TEXT = """\
Natural language processing (NLP) is an interdisciplinary subfield of
computer science and information retrieval. NLP helps machines understand,
interpret and generate human language.
"""


def print_config() -> None:
    """打印 .env 实际生效的关键配置，方便确认是否走 llama.cpp。"""
    from cognee.infrastructure.databases.vector.embeddings.config import (
        get_embedding_config,
    )
    from cognee.infrastructure.llm.config import get_llm_config

    llm = get_llm_config()
    emb = get_embedding_config()

    print("[config]")
    print(f"  LLM       : model={llm.llm_model!r} provider={llm.llm_provider!r} endpoint={llm.llm_endpoint!r}")
    print(f"  Embedding : model={emb.embedding_model!r} provider={emb.embedding_provider!r} "
          f"dims={emb.embedding_dimensions!r} endpoint={emb.embedding_endpoint!r}")


async def main() -> None:
    print_config()

    # 1) 入库：add + cognify（切块、LLM 抽取、构图、向量化）
    print("\n[1/2] remember() ...")
    result = await cognee.remember(
        SAMPLE_TEXT,
        dataset_name="smoke_test",
        self_improvement=False,  # 最小化 LLM 调用，只测主链路
    )
    print(f"  remember done -> {type(result).__name__}")

    # 2) 检索：基于刚才的知识回答
    print("\n[2/2] recall() ...")
    answers = await cognee.recall(
        query_text="What is NLP?",
        query_type=SearchType.GRAPH_COMPLETION,
        datasets=["smoke_test"],
    )

    if not answers:
        print("  recall returned no results.")
        return

    for i, item in enumerate(answers[:3], 1):
        text = getattr(item, "text", None) or str(item)
        print(f"  --- answer {i} ---")
        print(f"  {text}")

    print("\nSUCCESS: cognee smoke test passed.")


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except Exception:
        traceback.print_exc()
        raise SystemExit("\n测试失败，请检查上面的错误与 .env 配置。")

```
**输出**
```log
2026-09-04T01:35:35.234402 [info     ] Log file created at: C:\Users\qixin\.cognee\logs\2026-09-04_09-35-35.log [cognee.shared.logging_utils] log_file=C:\Users\qixin\.cognee\logs\2026-09-04_09-35-35.log

2026-09-04T01:35:35.234838 [warning  ] Cognee 1.0 changes: New API — remember/recall/forget/improve (V1 add/cognify/search still work). Session memory enabled by default (CACHING=false to disable). Multi-user access control on by default (ENABLE_BACKEND_ACCESS_CONTROL=false to disable). Agents (@cognee.agent) auto-verified on registration. See https://docs.cognee.ai/ [cognee.shared.logging_utils]

2026-09-04T01:35:35.235554 [info     ] Logging initialized            [cognee.shared.logging_utils] cognee_version=1.5.3-local database_path=D:\032-Cognee\cognee\cognee\.cognee_system\databases os_info='Windows 11 (10.0.26200)' python_version=3.14.3 structlog_version=25.5.0

2026-09-04T01:35:35.235764 [info     ] Database storage: D:\032-Cognee\cognee\cognee\.cognee_system\databases [cognee.shared.logging_utils]

2026-09-04T01:35:35.677570 [warning  ] REQUIRE_AUTHENTICATION=false is incompatible with ENABLE_BACKEND_ACCESS_CONTROL=true: multi-tenant mode requires authentication. Forcing REQUIRE_AUTHENTICATION=true. To disable auth for a single-user deployment, also set ENABLE_BACKEND_ACCESS_CONTROL=false. [get_authenticated_user]

2026-09-04T01:35:35.678048 [info     ] auth posture: authentication=required, multi_tenant=enabled (forced on by multi-tenant mode (REQUIRE_AUTHENTICATION=false was ignored)) [get_authenticated_user]
[config]
  LLM       : model='openai/local-llm' provider='custom' endpoint='http://192.168.1.39:8080/v1'
  Embedding : model='Qwen3-Embedding-0.6B' provider='openai_compatible' dims=1024 endpoint='http://192.168.1.39:8081/v1'

[1/2] remember() ...

setup plugin alembic.autogenerate.schemas

setup plugin alembic.autogenerate.tables

setup plugin alembic.autogenerate.types

setup plugin alembic.autogenerate.constraints

setup plugin alembic.autogenerate.defaults

setup plugin alembic.autogenerate.comments

setup plugin alembic.autogenerate.checkconstraint_byname

Using database: sqlite+aiosqlite:///D:\032-Cognee\cognee\cognee\.cognee_system\databases/cognee_db

Context impl SQLiteImpl.      

Will assume non-transactional DDL.

Relational migrations applied (target head).

2026-09-04T01:35:39.536243 [info     ] Skipping LLM/embedding connection tests (COGNEE_SKIP_CONNECTION_TEST is set). [cognee.shared.logging_utils]

2026-09-04T01:35:40.024654 [info     ] Log file created at: C:\Users\qixin\.cognee\logs\2026-09-04_09-35-35.log [cognee.shared.logging_utils] log_file=C:\Users\qixin\.cognee\logs\2026-09-04_09-35-35.log

2026-09-04T01:35:40.024959 [warning  ] Cognee 1.0 changes: New API — remember/recall/forget/improve (V1 add/cognify/search still work). Session memory enabled by default (CACHING=false to disable). Multi-user access control on by default (ENABLE_BACKEND_ACCESS_CONTROL=false to disable). Agents (@cognee.agent) auto-verified on registration. See https://docs.cognee.ai/ [cognee.shared.logging_utils]

2026-09-04T01:35:40.025197 [info     ] Logging initialized            [cognee.shared.logging_utils] cognee_version=1.5.3-local database_path=D:\032-Cognee\cognee\cognee\.cognee_system\databases os_info='Windows 11 (10.0.26200)' python_version=3.14.3 structlog_version=25.5.0

2026-09-04T01:35:40.025453 [info     ] Database storage: D:\032-Cognee\cognee\cognee\.cognee_system\databases [cognee.shared.logging_utils]

2026-09-04T01:35:40.029119 [warning  ] Could not load a matching tokenizer for embedding model 'Qwen3-Embedding-0.6B' (No module named 'transformers'). Falling back to TikToken, so token counts are approximate. Token counts drive chunk sizing and the --dry-run estimate, so a tokenizer that does not match the embedding model will mis-size chunks. Set HUGGINGFACE_TOKENIZER to a tokenizer matching your embedding model to fix this. [tokenizer_resolver]

2026-09-04T01:35:41.385091 [info     ] Pipeline run started: `a75de8a6-f247-5864-9015-c879f14686e3` [run_tasks_with_telemetry()]

2026-09-04T01:35:41.415197 [info     ] Coroutine task started: `classify_documents` [run_tasks_base]

2026-09-04T01:35:41.451964 [info     ] Async Generator task started: `extract_chunks_from_documents` [run_tasks_base]

2026-09-04T01:35:41.499374 [info     ] Coroutine task started: `extract_graph_and_summarize` [run_tasks_base]

2026-09-04T01:44:41.421526 [info     ] Coroutine task started: `add_data_points` [run_tasks_base]

2026-09-04T01:44:41.459423 [info     ] Completed graph extraction for DataPoint [cognee.shared.logging_utils] extra={'datapoint_id': 'c4637ee7-dd73-4f03-8fff-84835d884a02', 'nodes_extracted': 1, 'edges_extracted': 0}

2026-09-04T01:44:41.463995 [info     ] Completed graph extraction for DataPoint [cognee.shared.logging_utils] extra={'datapoint_id': 'd5c177cd-9a7c-50ae-b191-10e1dc7d810b', 'nodes_extracted': 1, 'edges_extracted': 0}

2026-09-04T01:44:41.464283 [info     ] Completed graph extraction for DataPoint [cognee.shared.logging_utils] extra={'datapoint_id': 'db1bf137-6f81-57fb-8018-e903894d30e7', 'nodes_extracted': 2, 'edges_extracted': 2}

2026-09-04T01:44:41.464494 [info     ] Completed graph extraction for DataPoint [cognee.shared.logging_utils] extra={'datapoint_id': 'da330436-aafa-526c-a301-8a18c07eb21c', 'nodes_extracted': 4, 'edges_extracted': 4}

2026-09-04T01:44:41.464753 [info     ] Completed graph extraction for DataPoint [cognee.shared.logging_utils] extra={'datapoint_id': 'f37c541f-7c43-50d6-b742-e47281cc5002', 'nodes_extracted': 5, 'edges_extracted': 5}

2026-09-04T01:45:04.188667 [info     ] Coroutine task completed: `add_data_points` [run_tasks_base]

2026-09-04T01:45:04.216254 [info     ] Coroutine task completed: `extract_graph_and_summarize` [run_tasks_base]

2026-09-04T01:45:04.245672 [info     ] Async Generator task completed: `extract_chunks_from_documents` [run_tasks_base]

2026-09-04T01:45:04.276715 [info     ] Coroutine task completed: `classify_documents` [run_tasks_base]

2026-09-04T01:45:04.306980 [info     ] Pipeline run completed: `a75de8a6-f247-5864-9015-c879f14686e3` [run_tasks_with_telemetry()]
  remember done -> RememberResult

[2/2] recall() ...#开始召回

2026-09-04T01:45:04.415371 [info     ] query_router: no patterns matched, default=HYBRID_COMPLETION query='What is NLP?' [query_router]

2026-09-04T01:45:04.415860 [info     ] Router override recorded: routed=HYBRID_COMPLETION, user_chose=GRAPH_COMPLETION (total=1) [query_router]

2026-09-04T01:45:07.037612 [info     ] Vector collection retrieval completed: Retrieved distances from 6 collections in 0.48s [cognee.shared.logging_utils]

2026-09-04T01:45:07.037956 [info     ] Retrieving ID-filtered graph from database. [CogneeGraph]

2026-09-04T01:45:07.053074 [info     ] ID-filtered retrieval: 5 nodes and 5 edges in 0.01s [cognee.shared.logging_utils]

2026-09-04T01:45:07.054330 [info     ] Graph projection completed: 5 nodes, 5 edges in 0.00s [CogneeGraph]

2026-09-04T01:45:07.054878 [info     ] Completed resolving edges to text [cognee.shared.logging_utils] extra={'node_count': 5, 'connection_count': 5}

2026-09-04T01:45:36.539567 [warning  ] Concurrent turn analysis failed open:  [session_concurrent_turn]

2026-09-04T01:50:49.516824 [info     ] recall: 1 results across sources=['graph'] (session=-) [recall]
  --- answer 1 ---
  NLP is an interdisciplinary subfield of computer science and information retrieval, supporting machine understanding, interpretation, and generation of human language.

SUCCESS: cognee smoke test passed.##测试工通过

Unclosed client session
client_session: <aiohttp.client.ClientSession object at 0x0000029EAA1B5160>

Unclosed connector
connections: ['deque([(<aiohttp.client_proto.ResponseHandler object at 0x0000029EEE591F90>, 7563.2680702)])']
connector: <aiohttp.connector.TCPConnector object at 0x0000029EAA1B52B0>
```
