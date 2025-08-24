# Kubernetes AI Conformance 草案（面向 Agentic AI 平台的企业级评测标准白皮书草案）

## 概要（Overview）
本白皮书草案旨在为运行 Agentic AI 平台的 Kubernetes 环境定义一套企业级评测标准（AI Conformance），重点覆盖加速器资源暴露与调度、网络与推理流量管理、作业排队与分布式训练、工作负载安全原语、可观测与审计、以及多集群/跨云可移植性。目标是形成一套可执行、可自动化回归的测试清单，便于平台供应商和企业自测并最终对接 CNCF 的 Conformance 体系。

---

## 目标与范围（Scope）
* **覆盖对象**：在 Kubernetes 上运行的 Agentic AI 平台与其支撑组件（模型服务、向量 DB、工具网关、沙箱、API 网关等）。  
* **非覆盖对象**：上层业务逻辑（具体 Agent 策略）本身的正确性测试，留给业务基准（WebArena、AgentBench）评测。  
* **输出**：可执行测试清单（YAML/Markdown 模板）、最小回归 CI 流水线示例、以及评分/通过准则。

---

## Conformance 层级（Conformance Levels）
1. **Level 0（基础）**：K8s 能安装并运行 AI 工具链基础组件，暴露 Prometheus 格式基本指标。  
2. **Level 1（生产）**：支持 DRA/设备属性、Gateway API、JobSet 或 Kueue；Workload Identity；基本安全沙箱。  
3. **Level 2（企业级）**：支持多集群迁移、GPU/TPU/FPGA 一致性验证、高级沙箱/机密计算、多租户隔离、自动化回归套件。  

---

## 测试与检查清单（Executable Checklist）
下面为可直接纳入 CI 的测试项（每项附“目的”“通过准则”“示例模板”）。

### A. 资源与调度（DRA / Device Model）
**目的**：验证 K8s 能以描述性资源模型暴露加速器属性并支持拓扑感知调度。  
**必测项（MUST）**：
1. 节点能通过 DRA 或类似接口暴露以下属性：`accelerator.vendor`、`accelerator.model`、`accelerator.memory`、`accelerator.numa_zone`。  
   * 通过准则：在任意节点上查询资源描述，并能对 pod 设置 `nodeSelector`/`resourceRequests` 匹配到目标设备。  
2. 拓扑感知调度：提交需要 NVLink/同 NUMA Node 特性的 pod，验证被调度到满足拓扑约束的节点。  
   * 通过准则：pod 在规定时间（例如 120s）内被调度到合乎约束的节点。

**示例 YAML（资源声明片段）**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-dra
spec:
  containers:
  - name: trainer
    image: busybox
    resources:
      requests:
        accelerator.example.com/vendor: "nvidia"
        accelerator.example.com/model: "a100"
````

### B. 加速器一致性测试（GPU/TPU/FPGA）

**目的**：验证同一请求在不同节点/不同硬件上的推理输出一致性与性能基线。
**必测项（MUST）**：

* 在至少两类加速器上运行相同模型的推理任务，比较输出一致性（数值容许误差阈值）及 P95 延迟、吞吐（QPS）。
* 通过准则：功能一致（误差在允许范围内），延迟/吞吐满足 SLA（例如 P95 ≤ 1s 或自定义标准）。

### C. 推理入口与流量管理（Gateway API）

**目的**：验证 Gateway API 能对推理流量做智能路由、流量控制与灰度。
**必测项（MUST）**：

* 基本路由规则生效（基于 Header、路径或 KV 缓存命中率）。
* 流量切分与 Canary 发布（灰度）成功。
* 通过准则：在 95% 请求中路由决策正确，灰度发布回滚成功。

### D. 批处理与作业排队（JobSet / Kueue）

**目的**：验证分布式训练作业的队列、Gang Scheduling 与资源配额功能。
**必测项（SHOULD）**：

* 提交一个跨节点的分布式训练 Job，验证 Gang scheduling 能否确保同时启动所有 Replica。
* 通过准则：作业在资源准备就绪后 60s 内同时启动。

### E. 安全与沙箱（Workload Identity / Sandboxing）

**目的**：验证 Agent 运行时的最小权限、安全隔离与沙箱能力。
**必测项（MUST）**：

* Workload Identity 生效：Pod 通过 SPIFFE 风格 token 获取临时凭据并调用内网服务。
* 沙箱隔离：代码解释器容器在 Kata/gVisor/Firecracker 下运行并受 egress 控制。
* 通过准则：尝试越权访问失败；沙箱内任意外联在白名单外被阻断。

### F. 可观测与审计（Metrics / Traces / Audit）

**目的**：验证 Prometheus 指标端点、OpenTelemetry traces 以及审计日志可用性。
**必测项（MUST）**：

* 模型服务与加速器暴露的 metrics（如 per-accelerator utilization、memory usage）能被抓取并入库。
* 关键链路（计划→动作→工具调用→结果）有 trace 可追溯。
* 通过准则：指标采集覆盖率 ≥ 95%，任一请求的 trace 可追溯并含工具调用详情。

### G. 多集群/迁移（Portability）

**目的**：验证在不同认证集群间 Agent 工件（工具、MCP 描述、CRD）零修改迁移能力。
**必测项（SHOULD）**：

* 将 Tool CRD 从 cluster A 导出并在 cluster B 导入，验证 Agent 能发现并调用。
* 通过准则：调用成功且输出一致（误差阈内）。

### H. 性能回归（Benchmark Integration）

**目的**：将 WebArena / AgentBench / 自定义场景作为回归基准，验证平台在变更后性能无明显退化。
**必测项（MUST）**：

* 在 CI 中运行一组代表性任务（交互类 Agent、RAG fetch、工具调用等），并与基线比较 P95、成功率、Cost/Task。
* 通过准则：P95/成功率在允许变动范围内（例如不劣化超过 5%）。

---

## 可执行测试模板（示例清单与评分）

> 下列 Markdown/YAML 用于 CI 自动化、测试登记与结果上报。

### conformance/tests/01-dra-check.yaml

```yaml
# 简化示例：DRA 资源曝光与拓扑测试
test: dra_resource_expose
level: MUST
description: 验证节点资源属性是否以 DRA 风格暴露
steps:
  - name: query_node_resources
    cmd: kubectl get nodes -o jsonpath='{.items[*].status.allocatable}'
  - name: validate_attrs
    cmd: ./validate_dra_attrs.sh
pass_criteria:
  - attr_present: ["accelerator.vendor","accelerator.model","accelerator.memory"]
  - max_time_s: 120
```

### conformance/tests/05-gateway-canary.md

```markdown
# Gateway API Canary 发布测试
level: MUST
description: 验证基于 Gateway API 的灰度发布与回滚
steps:
  1. 部署 v1 服务并设置 Gateway 路由
  2. 部署 v2 并将 10% 流量路由到 v2
  3. 触发流量并检查响应版本
  4. 回滚到 v1，检查流量恢复
pass:
  - canary_success_rate >= 90%
  - rollback_complete == true
```

---

## 最小回归评测流水线（CI Pipeline）

下面给出一个最小化的 CI 流水线示例，供团队直接落地（可用 GitHub Actions / Tekton / Jenkins 实现）。关键步骤为：环境准备 → 功能健康检查 → DRA / Gateway / JobSet / 沙箱 / 可观测 测试 → 基准回归（WebArena/AgentBench）→ 报告发布。

### Pipeline（伪 YAML）

```yaml
name: ai-conformance-pipeline
on: [push]
jobs:
  prepare:
    runs-on: ubuntu-latest
    steps:
      - run: install tools (kubectl, kustomize, helm)
      - run: provision cluster (kind/minikube or cloud test cluster)
  smoke:
    needs: prepare
    steps:
      - run: deploy core stack (k8s + apis + gateway + metrics)
      - run: smoke tests (basic pod scheduling)
  conformance:
    needs: smoke
    strategy:
      matrix: [dra, gateway, jobset, sandbox, metrics, portability]
    steps:
      - run: execute conformance/tests/${{matrix}}.*
      - run: upload results to artifact store
  benchmark:
    needs: conformance
    steps:
      - run: execute webarena/agentbench workloads
      - run: compare to baseline metrics
  report:
    needs: benchmark
    steps:
      - run: aggregate results
      - run: publish report (HTML / JSON)
```

---

## 评分规则与通过准则（Scoring）

* **MUST 项目**：任一失败导致该平台无法通过 Level 1（或设定 Level）。
* **SHOULD 项目**：建议达成，允许作为降级警告或改进项。
* **性能基线**：使用 CI 中的基准测试结果与历史基线比对；若关键指标退化 ≥ 5%，触发阻断。

---

## 与现有社区/基准的对接（Benchmarks）

* **集成 WebArena / AgentBench / SWE-bench** 作为业务层基准测试（Agent 行为成功率、工具调用成功率等）。\[2]\[3]\[4]
* **与 CNCF AI Conformance GitHub（草案）对齐**，把本白皮书中的测试作为“Agentic AI 扩展套件”。\[5]\[6]

---

## 参考资料（References）

1. CNCF，Help Us Build the Kubernetes Conformance for AI，[https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/](https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/)

2. CNCF AI Conformance（GitHub），[https://github.com/cncf/ai-conformance](https://github.com/cncf/ai-conformance)

3. WebArena，[https://webarena.dev/](https://webarena.dev/)

4. AgentBench，arXiv:2308.03688，[https://arxiv.org/abs/2308.03688](https://arxiv.org/abs/2308.03688)

5. GAIA 基准，arXiv:2311.12983，[https://arxiv.org/abs/2311.12983](https://arxiv.org/abs/2311.12983)

6. SWE-bench，[https://www.swebench.com/](https://www.swebench.com/)

7. 关于 DRA、Gateway API、JobSet 的 K8s SIG 文档汇编与社区讨论（见 CNCF 仓库链接）。