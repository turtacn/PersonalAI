# Agentic AI 基础设施与平台（Agentic AI Infrastructure & Platform）

## 摘要（Executive Summary）
本文件整合行业主流厂商与社区对“**Agentic AI**”, “**On-Premise**”目标的洞察，分析并提出一套面向企业级落地的技术路线和可测化基线。目标是给出既能快速落地又具前瞻性的设计：支持可移植的工具协议（MCP/OpenAPI），基于 Kubernetes 的混合云/本地平台，完整的治理与可观测能力，以及面向 AI 原生的 K8s 能力清单（Kubernetes AI Conformance 方向）。结合现实痛点、行业案例与可执行建议，便于技术决策者直接采用。

---

## 一、核心问题（Core Problems）
1. **工具和 Agent 生态碎片化，移植性差**  
   多厂商与自研实现采用不同协议、不同封装，导致工具复用困难、迁移成本高（来源对比见 [1][9]）。

2. **企业级治理与安全未成体系**  
   没有统一的 Agent 身份/最小权限/审计/沙箱规范，企业难以在合规环境下放开自治执行权限（见 [5][9]）。

3. **混合云/本地落地难：硬件、调度与数据主权**  
   实时性与数据主权要求促成本地化部署，但本地环境在 GPU/网络/存储一体化上差异大，运维成本高（见 [5][6][8]）。

4. **Kubernetes 对 AI 原生工作负载支持欠缺**  
   传统 K8s 在资源表达、拓扑感知、组调度、加速器一致性、沙箱与机密计算等方面需要扩展，影响可移植性与性能保证（详见 CNCF 草案段落 [5][6]）。

5. **缺乏统一可量化基线与回归测试**  
   当前平台评估常用指标不一致，缺少行业公认的 Agentic AI 企业级基线（参考基准：WebArena、AgentBench、SWE-bench）[2][3][4]。

---

## 二、基本思路（Design Principles）
1. **协议化、可自描述的工具层**  
   使用 MCP（Model Context Protocol）与 OpenAPI 互补：MCP 解决 Agent-to-Agent/Tool 层的上下文与可移植性，OpenAPI 作为 HTTP/REST 兼容描述。

2. **Kubernetes 为统一控制平面（但需扩展）**  
   K8s 负责容器、加速器、网络、存储调度，通过扩展（DRA、Kueue、JobSet、Gateway API、KServe 等）实现 AI 原生特性。

3. **治理（API 管理 + 身份）与可观测（OpenTelemetry）内嵌**  
   API Gateway（含策略/限流/审计）+ Workload Identity（SPIFFE/SPIRE 风格）+ 全链路 Trace/Metric/Logging。

4. **混合云/本地优先：数据主权与实时性**  
   允许“本地优先”部署：关键数据及推理保留在本地，非敏感或批量训练可上云。

5. **分层抽象以降低复杂性**  
   分为 硬件层 → 平台层（K8s）→ 中间件（MCP/OpenAPI/Tool Gateway）→ Agent 层 → 安全治理层，层间契约清晰，便于替换。

---

## 三、预期效果与行业基线（Expected Outcomes and Baselines）
> 以下基线整合了各方建议并取中短期可实现的目标（企业级生产环境目标）。

### 性能（Performance）
* 工具调用延迟（API） ≤ 100 毫秒；Agent 端到端响应（交互类） ≤ 1 秒（P95）。[来源：统一建议，参见多份原文]
* 多模态 RAG 检索单次 ≤ 1.5 秒（当地向量索引与高性能存储）。
* 单实例/单 Agent 并发管理能力 ≥ 1000（水平可扩展）。  

### 可用性（Availability）
* 平台控制面与关键服务 SLA ≥ 99.95%（企业级）。  
* 支持熔断、隔离与降级（单工具失败不致使全局停摆）。

### 安全（Security）
* Agent 与工具强制身份绑定（OAuth/OBO/短时凭据/Workload Identity）。  
* 100% 调用链审计（输入/工具调用/输出/决策），审计数据可回放。  
* 沙箱运行高风险工具（代码解释器、浏览器）并启用 egress 控制。

### 扩展性（Scalability）
* 工具自描述（MCP/OpenAPI），实现“零修改”跨集群迁移。  
* 支持跨集群/多云部署、工具注册与版本化。  

### 硬件（Hardware）
* 必需支持异构资源调度（GPU/TPU/FPGA/CPU），并暴露单卡利用率、显存使用等指标（Prometheus 格式）。  
* 拓扑感知调度、MIG 与 vGPU 支持，RDMA/NVLink 等网络加速集成（见 CNCF AI Conformance 要求）。

---

## 四、参考架构（Architecture Overview）

下面的架构图（Mermaid）展示硬件、Kubernetes 平台、治理中间件与 Agent 层如何协同（图下有要点说明）。

%% 图例：颜色用于区分层级，图中英文在中文后以全角括号标注
```mermaid
flowchart LR
    %% 模块命名规则示例"大写缩写[中文名称（English Term）]"
    subgraph HW[硬件层（Hardware Layer）]
        A1[CPU 节点池（CPU Pool）]
        A2[GPU／TPU 加速集群（Accelerators）]
        A3[高速存储（NVMe / GPUDirect）]
        A4[高速网络（RDMA / NVLink）]
    end

    subgraph PL[平台层（Platform Layer）]
        B1[Kubernetes 集群（Kubernetes Cluster）]
        B2[动态资源分配（DRA）]
        B3[AI 感知调度器（Topology-Aware Scheduler）]
        B4[服务网格与 API 网关（Service Mesh / API Gateway）]
    end

    subgraph MW[中间件与治理（Middleware & Governance）]
        C1[MCP 工具注册（MCP Tool Registry）]
        C2[工具网关（Tool Gateway）]
        C3[API 管理（API Management）]
        C4[身份与鉴权（Workload Identity）]
        C5[可观测（OpenTelemetry / Metrics）]
        C6[审计与策略引擎（Policy Engine / OPA）]
    end

    subgraph APP[应用与 Agent 层（Application & Agent）]
        D1[AI Agent 运行时（Agent Runtime）]
        D2[内置工具（Built-In Tools）]
        D3[企业自定义工具（Custom Tools）]
        D4[向量数据库（Vector DB / RAG）]
    end

    subgraph SEC[安全与合规（Security & Compliance）]
        E1[沙箱运行时（Sandbox Runtime）]
        E2[审计仓库（Audit Store）]
        E3[合规控制（Policy / Controls）]
    end

    %% 链接关系
    A1 --> B1
    A2 --> B1
    A3 --> B1
    A4 --> B1

    B1 --> B2
    B2 --> B3
    B4 --> C3

    D1 --> C1
    D1 --> C2
    D1 --> D4
    D2 --> C1
    D3 --> C1

    C2 --> C3
    C3 --> C4
    C3 --> C5
    C5 --> E2
    C6 --> E3

    E1 --> D1
    C4 --> D1
````

**图解要点**

1. 硬件层提供异构资源，需被 DRA 等机制细粒度描述与发现。
2. Kubernetes 作为控制平面，但要通过 DRA、拓扑感知调度器与 JobSet/Kueue 等扩展支持 AI 工作负载。
3. 中间件层（MCP、Tool Gateway、API 管理、Workload Identity、可观测）是企业治理与审计的核心。
4. 安全沙箱对不可信执行（代码解释器、受控浏览）是必须项，审计仓库保证可回放与追责。

---

## 五、关键技术需求与业务价值矩阵（Tech. & Value）
下面以表格形式（每个维度一张表）系统呈现 **技术需求 → 业务价值 → 可量化基线 / SLO → 简单易用建议 → 稳定可靠实践 → 快速落地建议**。此章可作为技术评审/路线图决策的单独章节直接使用，并便于转为任务卡（Jira）或评估清单。表格内容基于业界实践与 CNCF / Benchmarks / 各平台趋势综述\[2]\[3]\[5]\[10]\[11]，并优先体现“简单易用”与“稳定可靠”两条主线。

---

### 计算（Compute）

| 编号 | 技术需求（What）                               | 业务价值（Why）                 | 可量化基线 / SLO（How measured）                    | 简单易用建议（Developer UX）                              | 稳定可靠实践（Ops / Risk）                 | 快速落地建议（0–3 个月）                            |
| -: | ---------------------------------------- | ------------------------- | -------------------------------------------- | ------------------------------------------------- | ---------------------------------- | ----------------------------------------- |
| C1 | 异构加速器统一登记与属性描述（GPU/TPU/FPGA，含 MIG、驱动、拓扑） | 精准调度、减少性能抖动，提升资源利用率与预测性成本 | 每卡显存使用率、Per-accelerator 利用率；目标：显存利用率报警阈值 70% | 提供资源模板（如 `accelerator: nvidia/a100,mig:2`），隐藏复杂参数 | 使用 DRA 或兼容层暴露属性；设备指标暴露到 Prometheus | 部署 GPU exporter + DevicePlugin；在控制台显示拓扑视图 |
| C2 | 拓扑感知调度（NUMA、NVLink、PCIe 亲和）              | 降低跨节点通信延迟，提升训练/推理吞吐       | 分布式训练通信延迟、JobSet 启动成功率；Gang job 成功率 ≥ 99%    | 模板化配置分布式作业（一键 gang-schedule）                      | 使用 Kueue/JobSet 或调度扩展，做亲和策略和回退     | 在测试集群运行小规模 gang 作业并记录基线                   |
| C3 | 模型预热与 fast-warm 池（减少冷启动）                 | 降低交互延迟、改善用户体验             | P95 推理延迟；目标：交互场景 P95 ≤ 1s（实时场景 ≤ 50ms）       | 提供“预热”开关给部署模板                                     | 维护 warm pool 自动回收与容量告警             | 为重要模型配置 warm pool，测量冷/热启动差异               |
| C4 | 弹性伸缩（GPU-aware autoscaler，基于队列长度/利用率）    | 应对突发流量，控制成本               | scale-up latency、scale-to-zero 成功率           | 抽象为策略（如基于 QPS 或 GPU 使用率）供产品选择                     | 阈值保护、冷却窗口、防抖逻辑                     | 部署基于 GPU 指标的简单 HPA/KEDA 脚本                |
| C5 | 多租户隔离（资源 / IO / QoS）                     | 保护关键业务、满足合规与计费            | 资源争抢导致的 SLO 违约率；隔离生效率 100%                   | 提供租户级资源配额模版                                       | QoS 类别、SR-IOV、IOPS 配额、监控告警         | 先启用 Namespace-level ResourceQuota 与限流策略   |

---

### 存储（Storage）

| 编号 | 技术需求（What）                           | 业务价值（Why）              | 可量化基线 / SLO                                | 简单易用建议                  | 稳定可靠实践                       | 快速落地建议                         |
| -: | ------------------------------------ | ---------------------- | ------------------------------------------ | ----------------------- | ---------------------------- | ------------------------------ |
| S1 | 分层存储（热 NVMe / 温 SSD / 冷 对象存储）与自动分层策略 | 保证 RAG/向量检索实时性同时控制存储成本 | 向量检索 P95 延迟；目标交互场景 ≤ 100–300ms，批量场景 ≤ 1.5s | 提供数据生命周期模板（热/温/冷）       | 自动化迁移任务与回滚策略，定期一致性校验         | 部署本地向量库（Milvus/FAISS），建立冷热迁移脚本 |
| S2 | GPUDirect / NVMe 本地高速通道支持            | 减少主机间数据拷贝、降低训练网络带宽压力   | IO 带宽与延迟指标；目标 NVMe 带宽瓶颈未成为瓶颈               | 抽象为“数据盘类型”选择            | 使用 CSI 与性能校验脚本，做压力测试         | 为关键节点配置 NVMe 并做热路径基准           |
| S3 | 向量数据库低延迟检索与水平扩展                      | 支撑大规模 RAG、相似度检索        | 单查询延迟、吞吐（QPS）；目标：P95 ≤ 200–300ms           | 提供统一 VectorStore API 封装 | 异常分片重分布、备份快照策略               | 部署单节点演示版 Milvus 并接入检索 API      |
| S4 | 不可变审计日志与合规存储（WORM）                   | 满足数据主权、审计与取证需求         | 审计覆盖率 100%；审计查询可用性 SLA ≥ 99.95%            | 将审计导出做成一键功能             | 写入不可变对象存储并定期快照               | 把关键审计流先写入对象存储并验证回放             |
| S5 | 模型权重分发与版本控制（安全签名）                    | 快速回滚与合规性验证             | 模型部署回滚时间、签名校验率                             | 提供模型 registry UI（版本、签名） | 使用 sigstore/cosign；CI 集成签名校验 | 先接入简单模型仓库并用 cosign 做签名验证       |

---

### 网络（Network）

| 编号 | 技术需求（What）                    | 业务价值（Why）               | 可量化基线 / SLO                        | 简单易用建议                     | 稳定可靠实践                                 | 快速落地建议                        |
| -: | ----------------------------- | ----------------------- | ---------------------------------- | -------------------------- | -------------------------------------- | ----------------------------- |
| N1 | 高性能网络（RDMA / SR-IOV / 多网卡）    | 降低主机间延迟，提升分布式训练效率       | 内网延迟、RDMA 吞吐、重传率；目标内网延迟尽可能 <1ms    | 提供网络剖面选择（低延迟 / 公网 / 存储）    | Multus + SR-IOV，网络隔离与流控                | 启用 Multus 并测试 SR-IOV 模板       |
| N2 | Gateway API 作为推理入口（智能路由、缓存感知） | 灰度、流控、根据缓存命中率路由，减少不必要推理 | Gateway 决策延迟 P95 ≤ 10ms；路由成功率 100% | 将路由策略以配置项暴露给 SRE           | 使用 Gateway API + Service Mesh 做 N/S 加固 | 将推理入口先迁移到 Gateway API 并配置简单路由 |
| N3 | 服务网格与 mTLS（东西向安全）             | 增强服务间身份验证、流量策略及链路可观测    | mTLS 覆盖率 100%；流量镜像无性能显著下降          | 提供开关式启用（dev off / prod on） | Istio/Linkerd 配置稳定策略与熔断                | 在灰度环境启用 mTLS 并监测性能影响          |
| N4 | egress 控制与域名白名单（Agent 外联安全）   | 限制未经授权外联，防止数据泄露         | 外部请求被拒率、白名单误阻率                     | 内置默认白名单模板，便于快速上手           | 结合工具网关做审计与脱敏                           | 将高风险容器加入严格 egress 策略          |
| N5 | 网络 QoS 与带宽配额                  | 保证关键路径不被挤占（推理优先级）       | 带宽争抢导致的 SLO 违约率；关键流量丢包率            | 提供“优先级剖面”选择                | 在网络层实现带宽配额与限速                          | 配置关键 POD 的带宽限额并压测             |

---

### 安全（Security）

|   编号 | 技术需求（What）                          | 业务价值（Why）         | 可量化基线 / SLO           | 简单易用建议            | 稳定可靠实践                                | 快速落地建议                         |
| ---: | ----------------------------------- | ----------------- | --------------------- | ----------------- | ------------------------------------- | ------------------------------ |
| Sec1 | Workload Identity（OIDC/SPIFFE）与短时凭据 | 最小权限、可审计，避免长期凭据泄露 | 身份绑定比例 100%；凭据轮转频率    | 将凭据申请做成自助式短时令牌    | 强制所有服务使用 Workload Identity            | 先在控制面启用 OIDC 并迁移关键服务           |
| Sec2 | 沙箱化执行（代码解释器/浏览器隔离）                  | 降低不可信代码与外联风险      | 沙箱覆盖率、外联被阻断比率         | 将沙箱作为模板一键启用       | 使用 gVisor/Kata/Firecracker 并限制 egress | 把 Code Interpreter POD 迁移到沙箱运行 |
| Sec3 | 工具网关白名单、脱敏与速率限制                     | 控制工具调用风险、保护凭据     | 非白名单调用被阻断率 100%       | 默认启用最小白名单，提供扩展流程  | 日志脱敏与审计链绑定到请求 ID                      | 建立工具白名单与审计 pipeline            |
| Sec4 | 全链路审计与可回放（计划→工具→结果）                 | 可溯源、合规取证与异常回放     | 审计完整率 100%；回放成功率 100% | 提供审计回放 UI（按请求 ID） | 审计写入不可变存储并定期备份                        | 把 Agent 操作日志先写入审计桶并验证回放        |
| Sec5 | 供应链安全（镜像签名、依赖扫描）                    | 防止恶意/被篡改镜像进入生产    | 签名验证通过率 100%；漏洞修复时间   | 强制 CI 签名并在部署时校验   | sigstore/cosign + SBOM + 定期扫描         | 在 CI 中加入 cosign 校验并阻断未签名镜像     |
| Sec6 | Human-in-the-loop 策略（高风险操作人工复核）     | 风险控制、合规性与责任链      | 高风险操作人工审批通过率与平均延时     | 把审批嵌入工作流并提供模拟器    | 审批日志纳入审计并绑定请求上下文                      | 定义高风险动作清单并启用审批流程               |

---

### 横向要点（Cost、可观测与可运维性）

| 编号 | 技术需求（What）                                | 业务价值（Why）          | 可量化基线 / SLO          | 建议                                      |
| -: | ----------------------------------------- | ------------------ | -------------------- | --------------------------------------- |
| X1 | 成本/容量治理（Cost per Successful Task）         | 控制运营成本、支持定价与计费     | 成本指标每周可视化，预算超额告警     | 将 Cost metric 与任务/tenant 关联并入 Dashboard |
| X2 | 全链路可观测（OpenTelemetry、Tracing、Metrics）     | 快速定位故障、回放审计、SLO 报告 | Trace 覆盖率、SLO 违约告警时间 | Trace 一键追踪 plan→tool→result，暴露至控制台      |
| X3 | 自动化回滚与降级策略                                | 在异常时保证系统韧性         | 自动降级成功率、回滚时间         | 定义有损降级策略（禁外联→只读→人工复核）                   |
| X4 | 基准化测试与 CI 门禁（WebArena/AgentBench + 自定义用例） | 保证版本质量、避免回归        | 基准覆盖率、基线得分不低于历史分位    | 将基准测试纳入 CI Pipeline 并在发布前运行             |

---

### 优先级建议（简单版）

| 优先级 | 关键动作（示例）                                              | 目标时间    |
| --: | ----------------------------------------------------- | ------- |
|   高 | API Gateway + Workload Identity + 基本审计 + 向量 DB 演示     | 短期  |
|   中 | GPU metrics + GPU-aware autoscaler + Gateway API 推理入口 | 中期  |
|   低 | DRA / 拓扑感知调度 / RDMA / JobSet/Kueue 全面上线               | 长期 |

---

### 要点速览

* **按维度分解技术点**，同时用“简单易用 → 稳定可靠”两条线并行推进，能在短期内交付可用且合规的 On-Prem Agentic AI 平台基础能力；长期逐步引入 DRA、拓扑感知和高性能网络存储以实现规模化与成本优化。
* **度量与基线必须从一开始就建立**（显存利用率、P95 延迟、审计覆盖率、成本/任务），并把基准化测试纳入 CI（WebArena/AgentBench + 自建场景）。
* **快速落地路线（0–3 个月）**：优先把治理（身份、API、审计）与向量检索/推理入口做好，这是确保可用性与合规的“最低可用企业级平台”。

---

## 六、关键技术路线与成熟度评估（Roadmap & Maturity）

按“可短期落地/中期演进/长期前瞻”分类列出关键技术及其成熟度与落地建议。

| 关键点                              |       可行性 / 成熟度 | 建议落地路径                                                          |
| -------------------------------- | --------------: | --------------------------------------------------------------- |
| MCP 工具协议                         |        可行（早期采纳） | 先在内部 Tool Gateway 采用 MCP 标注工具元数据，逐步对外互操作。                       |
| OpenAPI 封装                       |              成熟 | 兼做外部工具适配层，便于传统 API 工具上接入。                                       |
| API 管理 + Workload Identity       |              成熟 | 使用 APIM（Azure API Management / Kong / Ambassador）+ SPIFFE 风格身份。 |
| DRA（Dynamic Resource Allocation） | 即将成熟（K8s v1.34） | 跟踪 K8s 版本，规划在 1.34+ 上启用 DRA；而在短期使用设备插件并做抽象兼容。                   |
| Kueue / JobSet                   |        发展中、社区认可 | 用于分布式训练、批作业调度，作为 Job 的增强替代。                                     |
| 沙箱运行时（gVisor/Kata/Firecracker）   |           成熟/可选 | 对高风险工具（代码执行、浏览）强制隔离。                                            |
| 向量数据库 + RAG 管道                   |              成熟 | 在本地部署向量 DB 加速检索并保证数据主权。                                         |
| 全链路可观测（OpenTelemetry）            |              成熟 | 指标、trace、日志必须贯穿计划→执行→工具调用→结果。                                   |

---

## 七、行业案例（精选、细化）

> 以下案例用于说明在现实场景中采用上述架构的效果与对比证据（含来源引用）。

### 案例 A：某大型金融机构（合规本地化推理）

* **问题**：交易风控需在毫秒级完成本地推理，数据不能外发（合规）。
* **方案**：在本地 K8s 集群部署 vLLM/Triton 推理，API 网关结合 Workload Identity，沙箱化执行可疑自动操作；向量索引用 NVMe 本地存储。
* **效果**：端到端延迟从云端 200ms 降至本地 30–50ms，满足交易超低延迟要求，并实现数据不出境审计（与多家厂商实践相符，参见 \[5]\[8]）。

### 案例 B：制造业（C3 AI 风格的模型驱动平台）

* **问题**：海量设备时序数据，需要在 PB 级存储与数百万数字孪生基础上做预测性维护。
* **方案**：构建强类型的企业模型层，K8s 支撑 PB 存储与批量训练，JobSet/Kueue 管理分布式训练作业。
* **效果**：比传统自建数据湖缩短项目上线周期 6–12 个月（参见 C3 AI 公开案例与分析 \[9]）。

### 案例 C：SaaS 平台供应商（Agent 化客服）

* **问题**：希望在多租户场景提供 Agentic API，同时隔离租户数据与计算资源。
* **方案**：多租户隔离采用 Namespaces + OPA/Kyverno 策略 + Workload Identity，借助 MCP 描述工具能力，并采集统一指标提供 Command Center。
* **效果**：降低误操作风险，提升租户隔离度与运维可视性（参考 Salesforce / ServiceNow 的实践 \[9]）。

---

## 八、实施建议（Practical Steps）

1. **短期**

   * 建立 Tool Gateway，统一 OpenAPI 描述；开始对关键工具做 MCP 元数据补充。
   * 部署 OpenTelemetry，保证对 Agent 行为的 Trace。
   * 将高风险执行（浏览器、代码运行）迁入现成沙箱（gVisor/Kata）。

2. **中期**

   * 升级 K8s 版本并评估 DRA（若支持，则逐步启用）。
   * 在 CI 中加入 WebArena/AgentBench 等基准的回归测试（见 docs2 的流水线示例）。
   * 建立审计仓库与合规报告流水线。

3. **长期**

   * 与社区协作实现 Kubernetes AI Conformance 自测套件对接。
   * 构建跨集群/跨云的工具注册中心与 Marketplace。
   * 完备 AI SLA 管理（自动化告警与回滚策略）。

---

## 九、趋势预测（Trends & Outlook）

1. **Kubernetes 将成为 AI 基础设施的事实控制面，但需要 AI-native 扩展（DRA、JobSet、Kueue、Gateway API）**。
2. **合规与数据主权推动本地化/混合云优先架构**；离线/air-gap 环境将成为金融、医疗等行业常态。
3. **工具协议化（MCP）与 Agent-to-Agent 标准将加速生态互操作**，但从协议到行业采纳需 12–24 个月。
4. **自动化基准与 Conformance 流水线（如 CNCF AI Conformance）会成为采购/上线门槛**。

---

## 十、结论（Conclusion）

要把 Agentic AI 从 PoC 推向企业生产，必须在协议（MCP/OpenAPI）、平台（K8s 扩展）、治理（API 管理、身份、审计）和硬件（拓扑感知、异构调度）四方面同时发力。短期落地靠 Tool Gateway 与可观测，中期依赖 K8s v1.34+DRA 等能力，长期以行业共识（Conformance）固化可移植与 SLA。本文给出的基线、架构与实施步阵，旨在帮助技术团队在 6–12 个月内完成从试点到可复制生产的转变。

---

## 参考资料（References）

1. CrewAI，The Leading Multi-Agent Platform，[https://www.crewai.com/](https://www.crewai.com/)

2. WebArena，A Realistic Web Environment for Building Autonomous Agents，[https://webarena.dev/](https://webarena.dev/)

3. GAIA: a benchmark for General AI Assistants，arXiv:2311.12983，[https://arxiv.org/abs/2311.12983](https://arxiv.org/abs/2311.12983)

4. SWE-bench Leaderboards，[https://www.swebench.com/](https://www.swebench.com/)

5. CNCF，Help Us Build the Kubernetes Conformance for AI，CNCF Blog（2025），[https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/](https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/)

6. CNCF AI Conformance（GitHub），[https://github.com/cncf/ai-conformance](https://github.com/cncf/ai-conformance)

7. AgentBench: Evaluating LLMs as Agents，arXiv:2308.03688，[https://arxiv.org/abs/2308.03688](https://arxiv.org/abs/2308.03688)

8. Towards Stable Large-Scale Benchmarking on Tool Learning，arXiv:2403.07714，[https://arxiv.org/html/2403.07714v1](https://arxiv.org/html/2403.07714v1)

9. C3 AI / SimplAI / Vendor 公共资料汇编与新闻稿（行业厂商实践分析，参见厂商官网及相关新闻稿）。

10. K8s 相关技术（JobSet / Kueue，DRA 动态资源分配）社区讨论与实现文档。

11. 各云厂商与平台能力公告（如 AWS Bedrock AgentCore、OpenAI Agent 模式、Microsoft Copilot Agent）—行业发布与白皮书汇总。
