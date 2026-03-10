# Data Agent - LangGraph 重构架构图

> 基于 Python + LangGraph 的新一代数据查询智能体架构
> 更新时间：2026-03-10

---

## 📊 架构概览

```mermaid
graph TB
    subgraph "API Layer"
        A[FastAPI Routes]
    end

    subgraph "Graph Layer - LangGraph Workflow"
        B[Main Graph]
        C[Subscription Subgraph]
    end

    subgraph "Node Layer - State Processors"
        D[Input Nodes]
        E[Rewrite Nodes]
        F[Retrieval Nodes]
        G[Query Nodes]
        H[Output Nodes]
        I[Subscription Nodes]
    end

    subgraph "Agent Layer - LLM Agents"
        J[Question Classifier]
        K[Question Rewriter]
        L[Report Matcher]
        M[Data Analyzer]
        N[Subscription Agent]
    end

    subgraph "Service Layer - External APIs"
        O[LLM Service]
        P[Elasticsearch Service]
        Q[Grafana Service]
        R[Database Service]
        S[Subscription Service]
    end

    subgraph "Utils Layer"
        T[Time Parser]
        U[URL Parser]
        V[Data Processor]
        W[Validators]
    end

    subgraph "Models Layer - Type Definitions"
        X[AgentState]
        Y[Enums]
        Z[Input/Process/Output Models]
    end

    subgraph "Config Layer"
        AA[Settings]
        AB[Prompts]
    end

    A --> B
    B --> C
    B --> D
    D --> E
    E --> F
    F --> G
    G --> H
    B --> I
    D -.-> J
    E -.-> K
    F -.-> L
    G -.-> M
    I -.-> N
    J --> O
    K --> O
    L --> O
    M --> O
    N --> O
    F --> P
    G --> Q
    G --> R
    I --> S
    E --> T
    G --> U
    G --> V
    D --> W
    B --> X
    X --> Y
    X --> Z
    O --> AA
    K --> AB
```

---

## 🔄 LangGraph 工作流详细流程

### 主工作流（Main Graph）

```mermaid
graph TD
    START([用户输入]) --> classify_query[1. Classify Query<br/>问题分类]

    classify_query --> route_type{Query Type?}

    route_type -->|data| rewrite_question[2. Rewrite Question<br/>问题改写]
    route_type -->|chat| handle_chat[Chat Response<br/>闲聊回复]
    route_type -->|subscription| subscription_flow[Subscription Subgraph<br/>订阅流程]

    rewrite_question --> parse_params[3. Parse Parameters<br/>参数解析]

    parse_params --> retrieve_reports[4. Retrieve Reports<br/>报表检索 Top70]

    retrieve_reports --> filter_reports[5. Filter Reports<br/>报表筛选 Top5]

    filter_reports --> match_reports[6. Match Reports LLM<br/>LLM精确匹配]

    match_reports --> check_match{Reports<br/>Matched?}

    check_match -->|yes| query_data[7. Query Data<br/>数据查询]
    check_match -->|no| no_match_response[No Match Response<br/>未找到报表]

    query_data --> check_success{Query<br/>Success?}

    check_success -->|yes| analyze_data[8. Analyze Data<br/>GPT-4 分析]
    check_success -->|no| error_response[Error Response<br/>查询失败]

    analyze_data --> generate_response[9. Generate Response<br/>生成回复]

    handle_chat --> END([返回结果])
    no_match_response --> END
    error_response --> END
    generate_response --> END
    subscription_flow --> END

    style START fill:#e1f5e1
    style END fill:#ffe1e1
    style classify_query fill:#e3f2fd
    style rewrite_question fill:#e3f2fd
    style parse_params fill:#e3f2fd
    style retrieve_reports fill:#fff3e0
    style filter_reports fill:#fff3e0
    style match_reports fill:#fff3e0
    style query_data fill:#f3e5f5
    style analyze_data fill:#f3e5f5
    style generate_response fill:#e8f5e9
```

### 订阅子图（Subscription Subgraph）

```mermaid
graph TD
    SUB_START([订阅请求]) --> detect_sub[Detect Subscription<br/>订阅检测]

    detect_sub --> classify_intent[Classify Intent<br/>意图分类]

    classify_intent --> route_intent{Intent Type?}

    route_intent -->|new| config_sub[Configure Subscription<br/>配置订阅]
    route_intent -->|modify| modify_sub[Modify Subscription<br/>修改订阅]
    route_intent -->|confirm| confirm_sub[Confirm Subscription<br/>确认订阅]
    route_intent -->|cancel| cancel_sub[Cancel Subscription<br/>取消订阅]

    config_sub --> create_task[Create Task<br/>创建订阅任务]
    modify_sub --> update_task[Update Task<br/>更新任务]

    create_task --> gen_response[Generate Response<br/>生成回复]
    update_task --> gen_response
    confirm_sub --> gen_response
    cancel_sub --> gen_response

    gen_response --> SUB_END([返回订阅信息])

    style SUB_START fill:#e1f5e1
    style SUB_END fill:#ffe1e1
    style detect_sub fill:#e1bee7
    style classify_intent fill:#e1bee7
    style config_sub fill:#ce93d8
    style create_task fill:#ba68c8
    style gen_response fill:#e8f5e9
```

---

## 🎯 AgentState 数据流图

```mermaid
graph LR
    subgraph "Input Context"
        A1[user_query]
        A2[history_data]
        A3[current_time]
    end

    subgraph "Query State"
        B1[query_type]
        B2[is_data_question]
        B3[show_data_flag]
    end

    subgraph "Rewrite Result"
        C1[target_index_info]
        C2[analy_target]
        C3[user_input_modified]
    end

    subgraph "Retrieval Result"
        D1[candidate_reports]
        D2[filtered_reports]
        D3[matched_reports_final]
    end

    subgraph "Query Result"
        E1[panel_base_data_list]
        E2[flattened_data]
        E3[analysis_result]
        E4[query_success]
    end

    subgraph "Output Context"
        F1[final_response]
        F2[conversation_record]
        F3[status]
    end

    A1 --> B1
    A1 --> C1
    A2 --> C3
    B1 --> C1
    C1 --> D1
    C3 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> F1
    E4 --> F3

    style A1 fill:#e3f2fd
    style C1 fill:#fff3e0
    style D3 fill:#f3e5f5
    style E3 fill:#e8f5e9
    style F1 fill:#ffebee
```

---

## 📦 模块依赖关系图

```mermaid
graph TD
    subgraph "Layer 0: Configuration"
        L0A[config/settings.py]
        L0B[config/prompts.py]
    end

    subgraph "Layer 1: Type Definitions"
        L1A[models/enums.py]
        L1B[models/input_models.py]
        L1C[models/process_models.py]
        L1D[models/output_models.py]
        L1E[models/subscription_models.py]
        L1F[models/state.py]
    end

    subgraph "Layer 2: Utilities"
        L2A[utils/time_parser.py]
        L2B[utils/url_parser.py]
        L2C[utils/data_processor.py]
        L2D[utils/validators.py]
        L2E[utils/logger.py]
    end

    subgraph "Layer 3: Services"
        L3A[services/llm_service.py]
        L3B[services/elasticsearch_service.py]
        L3C[services/grafana_service.py]
        L3D[services/database_service.py]
        L3E[services/subscription_service.py]
    end

    subgraph "Layer 4: Agents"
        L4A[agents/question_classifier.py]
        L4B[agents/question_rewriter.py]
        L4C[agents/report_matcher.py]
        L4D[agents/data_analyzer.py]
        L4E[agents/subscription_agent.py]
    end

    subgraph "Layer 5: Nodes"
        L5A[nodes/input_nodes.py]
        L5B[nodes/rewrite_nodes.py]
        L5C[nodes/retrieval_nodes.py]
        L5D[nodes/query_nodes.py]
        L5E[nodes/output_nodes.py]
        L5F[nodes/subscription_nodes.py]
    end

    subgraph "Layer 6: Graph"
        L6A[graph/edges.py]
        L6B[graph/main_graph.py]
        L6C[graph/subscription_subgraph.py]
        L6D[graph/checkpointer.py]
    end

    subgraph "Layer 7: API"
        L7A[api/schemas.py]
        L7B[api/routes.py]
    end

    subgraph "Layer 8: Application"
        L8[main.py]
    end

    L0A --> L1F
    L0B --> L4B
    L0B --> L4D

    L1A --> L1B
    L1A --> L1C
    L1B --> L1F
    L1C --> L1F
    L1D --> L1F
    L1E --> L1F

    L1F --> L2C
    L1F --> L3A

    L2A --> L5B
    L2B --> L5D
    L2C --> L5D
    L2D --> L5A
    L2E --> L3A

    L0A --> L3A
    L0A --> L3B
    L0A --> L3C
    L0A --> L3D
    L1F --> L3A

    L3A --> L4A
    L3A --> L4B
    L3A --> L4C
    L3A --> L4D
    L3A --> L4E
    L3B --> L5C
    L3C --> L5D
    L3D --> L5D
    L3E --> L5F

    L4A --> L5A
    L4B --> L5B
    L4C --> L5C
    L4D --> L5D
    L4E --> L5F

    L1F --> L5A
    L1F --> L5B
    L1F --> L5C
    L1F --> L5D
    L1F --> L5E
    L1F --> L5F

    L5A --> L6B
    L5B --> L6B
    L5C --> L6B
    L5D --> L6B
    L5E --> L6B
    L5F --> L6C
    L1F --> L6A
    L6A --> L6B
    L6C --> L6B

    L6B --> L7B
    L1F --> L7A
    L7A --> L7B

    L6B --> L8
    L7B --> L8

    style L0A fill:#e8eaf6
    style L1F fill:#e3f2fd
    style L3A fill:#fff3e0
    style L4D fill:#f3e5f5
    style L6B fill:#e8f5e9
    style L8 fill:#ffebee
```

---

## 🔧 核心组件说明

### 1. AgentState（状态模型）

**核心职责**：维护整个工作流的状态数据

| 子模型 | 字段数 | 关键字段 | Producer | Consumer |
|--------|-------|---------|----------|----------|
| **InputContext** | 5 | user_query, history_data, current_time | START节点 | 分类、改写节点 |
| **QueryState** | 4 | query_type, is_data_question, show_data_flag | 分类节点 | 路由边 |
| **RewriteResult** | 7 | target_index_info, analy_target, user_input_modified | 改写节点 | 检索、查询节点 |
| **RetrievalResult** | 3 | candidate_reports, filtered_reports, matched_reports_final | 检索节点 | 查询节点 |
| **QueryResult** | 6 | panel_base_data_list, flattened_data, analysis_result | 查询、分析节点 | 输出节点 |
| **SubscriptionState** | 5 | subscription_intent, subscription_config, task_id | 订阅节点 | 订阅子图 |
| **OutputContext** | 3 | final_response, conversation_record, status | 输出节点 | API Layer |

**总字段数**：33个（相比原Dify 80+变量减少60%）

---

### 2. Node Functions（节点函数）

**设计原则**：
- ✅ 无状态（Stateless）：只接收AgentState，返回State更新字典
- ✅ 单一职责：每个节点只做一件事
- ✅ 可测试性：纯函数，易于单元测试
- ✅ 可组合性：节点之间通过State解耦

**节点分类**：

| 类别 | 节点数 | 文件 | 说明 |
|-----|-------|------|------|
| **输入节点** | 1 | input_nodes.py | 问题分类 |
| **改写节点** | 2 | rewrite_nodes.py | 问题改写、参数解析 |
| **检索节点** | 3 | retrieval_nodes.py | 报表检索、筛选、匹配 |
| **查询节点** | 2 | query_nodes.py | 数据查询、分析 |
| **输出节点** | 3 | output_nodes.py | 回复生成、聊天回复、错误处理 |
| **订阅节点** | 7 | subscription_nodes.py | 订阅检测、分类、配置、CRUD |

**总节点数**：18个（相比原Dify 80+节点减少78%）

---

### 3. Conditional Edges（条件边）

**路由决策函数**：

```python
# edges.py 中的5个路由函数

def route_by_query_type(state) -> Literal["data", "chat", "subscription"]:
    """根据问题类型路由"""
    pass

def route_by_data_availability(state) -> Literal["analyze", "error"]:
    """根据数据可用性路由"""
    pass

def route_by_report_match(state) -> Literal["query", "no_match"]:
    """根据报表匹配结果路由"""
    pass

def route_by_subscription_status(state) -> Literal["continue", "end"]:
    """根据订阅状态路由"""
    pass

def should_show_data(state) -> Literal["show", "hide"]:
    """是否显示详细数据"""
    pass
```

---

### 4. LLM Agents（智能体）

**统一接口**：所有Agent继承`BaseAgent`

```python
class BaseAgent(ABC):
    def __init__(self, llm_service: LLMService):
        self.llm = llm_service

    @abstractmethod
    def invoke(self, state: AgentState) -> Dict[str, Any]:
        """处理State，返回State更新"""
        pass
```

**Agent列表**：

| Agent | 模型 | 输入 | 输出 | 对应原Dify节点 |
|-------|------|------|------|---------------|
| **QuestionClassifier** | DeepSeek-V3.1 | user_query | query_type | 1762787668706 |
| **QuestionRewriter** | GPT-4 | user_query, history | target_index_info | 1757317387157等4个 |
| **ReportMatcher** | DeepSeek-V3.1 | filtered_reports, target_index_info | matched_reports | 1762508830468 |
| **DataAnalyzer** | GPT-4.1 + Code Interpreter | flattened_data, analy_target | analysis_result | 1758989941400 |
| **SubscriptionAgent** | GPT-4 | user_query, history | subscription_config | 订阅相关26+节点 |

---

## 📈 性能对比

| 指标 | Dify Workflow | LangGraph 重构 | 改进 |
|-----|--------------|---------------|------|
| **节点数量** | 80+ | 18 | ⬇️ 78% |
| **变量数量** | 50+ | 33 (AgentState字段) | ⬇️ 34% |
| **if-else判断** | 20+ | 5 (条件边) | ⬇️ 75% |
| **代码行数** | 4000+ (Code节点) | ~2000 (模块化) | ⬇️ 50% |
| **平均响应时间** | 8-12s | 4-6s (预期) | ⬇️ 50% |
| **可维护性** | ⭐⭐ | ⭐⭐⭐⭐⭐ | +150% |
| **可测试性** | ⭐ | ⭐⭐⭐⭐⭐ | +400% |

---

## 🛠️ 技术栈

| 层级 | 技术 | 版本 | 用途 |
|-----|------|------|------|
| **工作流框架** | LangGraph | ≥0.0.60 | 状态图编排 |
| **LLM框架** | LangChain | ≥0.1.0 | LLM调用封装 |
| **类型验证** | Pydantic | ≥2.5.0 | 数据模型 |
| **API框架** | FastAPI | ≥0.108.0 | REST API |
| **LLM提供商** | OpenAI + DeepSeek | - | GPT-4.1 + DeepSeek-V3.1 |
| **搜索引擎** | Elasticsearch | ≥8.11.0 | 报表检索 |
| **数据库** | ClickHouse + MySQL | - | 数据查询 |
| **可视化** | Grafana API | - | 报表管理 |
| **日志** | Loguru | ≥0.7.0 | 结构化日志 |

---

## 🚀 实施路径

### 第1轮：基础设施层（3-4天）✅ 已完成骨架
- [x] 项目骨架创建
- [ ] config/ 模块实现
- [ ] models/ 模块实现
- [ ] utils/ 模块实现

### 第2轮：服务层（3-4天）
- [ ] services/llm_service.py
- [ ] services/elasticsearch_service.py
- [ ] services/grafana_service.py
- [ ] services/database_service.py

### 第3轮：Agent层（4-5天）
- [ ] agents/question_classifier.py
- [ ] agents/question_rewriter.py
- [ ] agents/report_matcher.py
- [ ] agents/data_analyzer.py

### 第4轮：节点层（4-5天）
- [ ] nodes/input_nodes.py
- [ ] nodes/rewrite_nodes.py
- [ ] nodes/retrieval_nodes.py
- [ ] nodes/query_nodes.py

### 第5轮：图层（3-4天）
- [ ] graph/edges.py
- [ ] graph/main_graph.py
- [ ] graph/subscription_subgraph.py

### 第6轮：API与集成（3-4天）
- [ ] api/routes.py
- [ ] api/schemas.py
- [ ] main.py
- [ ] 端到端测试

---

## 📝 更新日志

- **2026-03-10**: 创建新架构图，基于LangGraph重构设计
- **2026-03-10**: 完成项目骨架创建（48个文件）
- **待更新**: 各模块实施完成后更新此架构图

---

## 🔗 相关文档

- [原Dify架构图](./架构图.md)
- [AgentState设计详情](../C:\Users\Administrator\.claude\plans\effervescent-crunching-book.md)
- [项目README](../README.md)
- [实施计划](../C:\Users\Administrator\.claude\plans\effervescent-crunching-book.md)

---

**本架构图将随项目进展持续更新** 🔄
