# SASE和ZTP业务融合平台升级路径分析

本文档通过可视化图表分析SASE和ZTP业务在不同平台间的融合升级路径，重点展示两种主要的迁移策略及其工作量对比。

## 当前业务部署状态

首先展示当前业务在各平台的部署现状：

```mermaid
graph TB
    %% 图例
    classDef readyPlatform fill:#90EE90,stroke:#006400,stroke-width:2px
    classDef currentPlatform fill:#FFE4B5,stroke:#FF8C00,stroke-width:2px
    classDef targetPlatform fill:#87CEEB,stroke:#4682B4,stroke-width:2px
    
    subgraph CURRENT[当前部署状态]
        XOS[XOS平台]
        APEX[APEX平台]
        VC[VC平台，基础平台提供的虚拟集群能力]
        
        SASE_XOS[SASE#64;XOS<br/>P包已交付，正在运行，正迭代开发]
        ZTP_APEX[ZTP#64;APEX<br/>已就绪，正迭代开发]
        
        XOS --> SASE_XOS
        APEX --> ZTP_APEX
    end
    
    class SASE_XOS currentPlatform
    class ZTP_APEX readyPlatform
    class APEX,VC targetPlatform
```

当前状态显示SASE业务部署在XOS平台，而ZTP业务已在APEX平台就绪，形成了业务分散部署的现状。

## 业务融合升级路径对比

针对SASE和ZTP业务融合需求，存在两条可行的升级路径：

```mermaid
graph TD
    %% 图例和样式定义
    classDef startNode fill:#FFE4B5,stroke:#FF8C00,stroke-width:3px
    classDef path1Node fill:#90EE90,stroke:#006400,stroke-width:2px
    classDef path2Node fill:#87CEEB,stroke:#4682B4,stroke-width:2px
    classDef finalNode fill:#DDA0DD,stroke:#8B008B,stroke-width:3px
    classDef workload fill:#F0F8FF,stroke:#000080,stroke-width:1px,stroke-dasharray: 5 5
    
    START[SASE#64;XOS<br/>当前状态]
    
    %% 路径一：APEX统一平台
    subgraph PATH1[路径一：APEX平台统一融合]
        P1_STEP1[SASE#64;XOS --> SASE#64;APEX<br/>原地升级]
        P1_WORK1[升级包适配：300人天<br/>原地升级服务：600人天]
        P1_STEP2[ZTP#64;APEX直接开启<br/>0人天]
        P1_FINAL[APEX平台统一<br/>SASE+ZTP融合]
        
        P1_STEP1 --> P1_WORK1
        P1_WORK1 --> P1_STEP2
        P1_STEP2 --> P1_FINAL
    end
    
    %% 路径二：VC平台部署
    subgraph PATH2[路径二：VC平台部署方案]
        P2_STEP1[SASE#64;XOS --> （SASE#64;XOS）#64;VC<br/>VC平台迁移]
        P2_WORK1[SASE迁移工作量：140人天]
        P2_STEP2[ZTP#64;APEX --> （ZTP#64;APEX）#64;VC<br/>VC平台迁移]
        P2_WORK2[ZTP迁移工作量：300人天]
        P2_FINAL[VC平台统一<br/>SASE+ZTP融合]
        
        P2_STEP1 --> P2_WORK1
        P2_STEP2 --> P2_WORK2
        P2_WORK1 --> P2_FINAL
        P2_WORK2 --> P2_FINAL
    end
    
    START --> P1_STEP1
    START --> P2_STEP1
    START --> P2_STEP2
    
    class START startNode
    class P1_STEP1,P1_STEP2,P1_FINAL path1Node
    class P2_STEP1,P2_STEP2,P2_FINAL path2Node
    class P1_WORK1,P2_WORK1,P2_WORK2 workload
```

## 详细工作量分解分析

为了更清晰地展示各路径的工作量构成，以下图表详细分析了每个升级步骤的工作内容：

```mermaid
graph LR
    %% 图例定义
    classDef highCost fill:#FFB6C1,stroke:#DC143C,stroke-width:2px
    classDef mediumCost fill:#F0E68C,stroke:#DAA520,stroke-width:2px
    classDef lowCost fill:#98FB98,stroke:#32CD32,stroke-width:2px
    classDef zeroCost fill:#E6E6FA,stroke:#9370DB,stroke-width:2px
    
    subgraph WORKLOAD[工作量对比分析]
        subgraph P1_DETAIL[路径一详细工作量]
            P1_TASK1[SASE适配APEX<br/>业务改造<br/>300人天]
            P1_TASK2[原地升级服务<br/>工作流编排<br/>600人天]
            P1_TASK3[ZTP应用开启<br/>0人天]
            P1_TOTAL[路径一总计<br/>900人天]
            
            P1_TASK1 --> P1_TOTAL
            P1_TASK2 --> P1_TOTAL
            P1_TASK3 --> P1_TOTAL
        end
        
        subgraph P2_DETAIL[路径二详细工作量]
            P2_TASK1[SASE迁移至VC<br/>140人天]
            P2_TASK2[ZTP迁移至VC<br/>300人天]
            P2_TOTAL[路径二总计<br/>440人天]
            
            P2_TASK1 --> P2_TOTAL
            P2_TASK2 --> P2_TOTAL
        end
    end
    
    class P1_TASK2 highCost
    class P1_TASK1,P2_TASK2 mediumCost
    class P2_TASK1 lowCost
    class P1_TASK3 zeroCost
    class P1_TOTAL highCost
    class P2_TOTAL mediumCost
```

## 升级时序流程分析

以下时序图展示了两种路径的具体执行时序和关键节点：

```mermaid
sequenceDiagram
    %% 参与者定义
    participant XOS as XOS平台
    participant APEX as APEX平台
    participant VC as VC平台
    participant SASE as SASE业务
    participant ZTP as ZTP业务
    
    %% 路径一时序
    Note over XOS,ZTP: 路径一：APEX平台统一方案
    
    XOS->>APEX: 1.1 SASE业务迁移准备
    Note right of APEX: 业务改造 300人天
    
    XOS->>APEX: 1.2 原地升级执行
    Note right of APEX: 升级服务 600人天
    
    APEX->>SASE: 1.3 SASE业务完成迁移
    APEX->>ZTP: 1.4 ZTP业务直接开启
    Note right of ZTP: 0人天 即时开启
    
    SASE->>ZTP: 1.5 业务融合完成
    
    %% 路径二时序
    Note over XOS,ZTP: 路径二：VC平台部署方案
    
    XOS->>VC: 2.1 SASE业务迁移
    Note right of VC: 140人天
    
    APEX->>VC: 2.2 ZTP业务迁移
    Note right of VC: 300人天
    
    VC->>SASE: 2.3 SASE部署完成
    VC->>ZTP: 2.4 ZTP部署完成
    
    SASE->>ZTP: 2.5 业务融合完成
```

## 路径选择建议与风险分析

通过上述图表分析，可以得出以下关键结论：

**路径一 APEX统一方案**的优势在于最终实现了在单一APEX平台上的完整业务融合，ZTP业务无需额外迁移工作。但其总工作量较高（900人天），主要集中在原地升级服务的复杂性上。

**路径二 VC平台方案**的优势在于总工作量相对较少（440人天），迁移风险相对可控。但需要同时处理两个业务的平台迁移，协调复杂度较高。

基于工作量和风险平衡考虑，建议优先考虑路径二，同时制定详细的迁移时序规划，确保SASE和ZTP业务的平滑过渡和最终融合。
