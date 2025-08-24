# Proposal：Kubernetes AI Conformance for Agentic AI Platforms (CNCF SIG Draft)

## Executive Summary
This draft proposes an enterprise-grade conformance profile for Kubernetes focused on Agentic AI platforms. It covers essential areas: accelerator resource model and scheduling (DRA), inference ingress and Gateway API, distributed training/job queueing (JobSet/Kueue), workload identity and sandboxing, observability and auditability, and portability across clusters. The aim is an actionable, automatable conformance test suite that can be used by vendors and end-users to validate that their Kubernetes environment meets production requirements for Agentic AI.

---

## Scope and Objectives
* **Scope**: Kubernetes clusters hosting Agentic AI components such as model servers, vector DBs, tool gateways, sandbox runtimes, and API gateways.  
* **Goal**: Provide executable test cases (YAML/Markdown), CI pipeline templates, and pass/fail criteria for a multi-level conformance profile.

---

## Conformance Levels
* **Level 0 (Basic)**: Basic component deployment and metrics exposure.  
* **Level 1 (Production)**: DRA/device attributes, Gateway API, JobSet or Kueue, Workload Identity, basic sandboxing.  
* **Level 2 (Enterprise)**: Multi-cluster portability, accelerator consistency tests, advanced sandboxing/confidential computing, multi-tenant isolation, and automated regression suites.

---

## Test Checklist (Selected Items)
A. **Resource and Scheduling (DRA)**  
* Validate nodes expose accelerator attributes: vendor, model, memory, numa_zone.  
* Validate topology-aware scheduling places pods with specific NVLink/NUMA requirements.

B. **Accelerator Consistency**  
* Run inference across different accelerator types and verify output consistency and performance baselines.

C. **Inference Gateway (Gateway API)**  
* Validate traffic routing, canary/gray release, and KV cache-aware routing behavior.

D. **Distributed Job Scheduling (JobSet/Kueue)**  
* Validate gang scheduling for distributed training; ensure simultaneously starting replicas.

E. **Security and Sandboxing**  
* Validate workload identity tokens, sandbox enforcement for untrusted code, and egress restrictions.

F. **Observability & Audit**  
* Validate Prometheus metrics (per-accelerator utilization, memory) and OpenTelemetry traces are present and traceable through plan→action→tool→result.

G. **Portability**  
* Validate tool CRDs & MCP descriptors move across clusters without modification.

H. **Benchmark Integration**  
* Integrate WebArena, AgentBench, SWE-bench for functional and performance regression.

---

## Minimal CI Pipeline (Pseudo YAML)
```yaml
name: ai-conformance-pipeline
on: [push]
jobs:
  prepare:
    steps: install tools, provision test cluster
  smoke:
    steps: deploy core stack, run smoke tests
  conformance:
    matrix: [dra, gateway, jobset, sandbox, metrics, portability]
    steps: run conformance tests, upload results
  benchmark:
    steps: run webarena/agentbench workloads, compare baselines
  report:
    steps: aggregate and publish results
````

---

## Pass/Fail Rules

* **MUST** tests are blocking; any failure blocks Level 1 certification.
* **SHOULD** tests are advisory and become blocking at Level 2 if repeated failures occur.
* **Performance regression threshold**: if key metric degrades by more than 5% vs baseline, mark as failure.

---

## Integration with Community Benchmarks

* Adopt WebArena / AgentBench / SWE-bench as functional/performance benchmarks to complement conformance tests.
* Coordinate with CNCF AI Conformance repo and SIGs to harmonize test cases and tooling. \[CNCF repo references]

---

## References

1. CNCF Blog: Help Us Build the Kubernetes Conformance for AI，[https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/](https://www.cncf.io/blog/2025/08/01/help-us-build-the-kubernetes-conformance-for-ai/)

2. CNCF AI Conformance GitHub，[https://github.com/cncf/ai-conformance](https://github.com/cncf/ai-conformance)

3. WebArena，[https://webarena.dev/](https://webarena.dev/)

4. AgentBench (arXiv)，[https://arxiv.org/abs/2308.03688](https://arxiv.org/abs/2308.03688)

5. GAIA benchmark，[https://arxiv.org/abs/2311.12983](https://arxiv.org/abs/2311.12983)

6. SWE-bench，[https://www.swebench.com/](https://www.swebench.com/)