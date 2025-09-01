# AI 架构设计提示 - 源自 Eskil Steenberg 的《大型软件项目架构》讲座

借鉴 Eskil Steenberg 传奇般的系统编程讲座，利用 AI 驱动的架构原则，将任何代码库转变为模块化、可替换的“黑盒”。

## 🎯 这是什么

三个专门设计的 AI 提示，用于训练 Claude、ChatGPT 和 Cursor 等模型，使其能够以下列思维方式进行思考：

- **黑盒接口 (Black box interfaces)** - 模块之间清晰的 API
- **可替换组件 (Replaceable components)** - 如果你无法理解一个模块，就重写它
- **恒定开发速度 (Constant velocity)** - 今天写5行代码，胜过未来修改1行
- **单一职责 (Single responsibility)** - 一个模块，一个人负责

## 🔥 真实成果

我曾使用这些提示，完整重构了 [guocedb](https://github.com/turtacn/guocedb) 项目——一个利用多个开源组件，去创造另一个开源组件的思路。

```text guocedb-prompt 
You are an expert Senior Software Architect and a seasoned Open Source Project Lead. Your task is to design and generate the complete project structure and foundational files for a new, ambitious open-source project called "guocedb". Your output must be professional, meticulously detailed, well-structured, and ready for a development team to start working on. The tone should be inspiring for potential contributors.

特别要求：深刻分析下面核心输入中 <开源项目参考部分> 中的开源项目和 <需求描述部分> 中的技术需求描述，要求全面技术覆盖到每一个技术需求项；ps: `guocedb-cli` is short for command line `guocedb`；根据项目实现难度和质量分析，从golang和python中选择一个主要项目语言，可通过各种脚本和其他命令集成

Project Name: guocedb GitHub Repository: github.com/turtacn/guocedb


核心输入：
1.<开源项目参考部分> {

假定你是关系数据库mysql和golang顶级大师，需求是极其熟悉并通过两个开源项目，构建一个新的开源项目:
    - [A MySQL-compatible relational database with a storage agnostic query engine. Implemented in pure Go.] https://github.com/dolthub/go-mysql-server
    - [fast key-value db in go] https://github.com/hypermodeinc/badger
    <可洞察搜索相关，e.g tidb >
}

2. <需求描述部分> {

```原文-markdown
    目标是从0-1重新构建一个从内存数据库（go-mysql-server）演进成mysql协议兼容的通用型集中数据库（guocedb），借助badger的kv持久化存储能力，基本通用型集中数据库的功能和能力需要具备，支持一般业务系统的集成开发。
    guocedb架构采用分层设计
    - 接口层提供多样化的数据访问和管理接口，包括SQL以及数据生命周期管理API
    - 计算层包含查询优化器、向量化执行引擎、分布式调度器、执行器、事务管理器，并通过服务网格实现弹性伸缩
    - 存储层通过抽象存储层（SAL）解耦计算与存储，支持插件式的存储引擎（MDD/MDI/KVD，badger）
    - 维护层监控数据库运行状态、性能指标和日志
    - 安全层提供身份认证、访问控制、数据加解密、防篡改、漏洞管理和审计机制
    - guocedb项目目录严格按照如下的结构，可以新增，不可修改；(假设项目地址 https://github.com/turtacn/guocedb）
            ```plaintext
            guocedb/
            ├── api/                      # 外部 API 定义 (External API definitions)
            │   ├── protobuf/             # gRPC 服务定义 (gRPC service definitions)
            │   │   └── mgmt/             # 管理 API (Management API)
            │   │       └── v1/
            │   │           └── management.proto # 管理服务 proto 文件 (Management service proto file)
            │   └── rest/                 # RESTful API (未来扩展 / Future extension)
            │       └── rest.go           # REST API 占位符 (REST API placeholder)
            ├── cmd/                      # 项目可执行文件入口 (Project executables entry points)
            │   ├── guocedb-cli/          # 命令行客户端 (Command-line client)
            │   │   └── main.go
            │   └── guocedb-server/       # 数据库服务端 (Database server)
            │       └── main.go
            ├── common/                   # 通用基础包 (Common base packages)
            │   ├── config/               # 配置加载与管理 (Configuration loading and management)
            │   │   └── config.go
            │   ├── constants/            # 全局常量定义 (Global constants definition)
            │   │   └── constants.go
            │   ├── errors/               # 统一错误定义与处理 (Unified error definition and handling)
            │   │   └── errors.go
            │   ├── log/                  # 统一日志接口与实现 (Unified logging interface and implementation)
            │   │   └── logger.go
            │   └── types/                # 基础数据类型 (Basic data types)
            │       ├── enum/             # 全局枚举类型 (Global enumeration types)
            │       │   └── enum.go
            │       └── value/            # SQL 值类型 (SQL value types)
            │           └── value.go
            ├── compute/                  # 计算层 (Compute Layer)
            │   ├── analyzer/             # SQL 查询分析器 (SQL query analyzer)
            │   │   └── analyzer.go       # 分析器实现/包装 (Analyzer implementation/wrapper)
            │   ├── catalog/              # 元数据目录管理 (Metadata catalog management)
            │   │   ├── catalog.go        # Catalog 接口定义 (Catalog interface definition)
            │   │   ├── memory/           # 内存 Catalog 实现 (In-memory Catalog implementation)
            │   │   │   └── memory_catalog.go
            │   │   └── persistent/       # 持久化 Catalog 实现 (Persistent Catalog implementation)
            │   │       └── persistent_catalog.go
            │   ├── executor/             # 查询执行引擎 (Query execution engine)
            │   │   ├── engine.go         # 执行引擎接口与基础实现 (Engine interface and base implementation)
            │   │   └── vector/           # 向量化执行相关 (Vectorized execution related)
            │   │       └── vector.go     # 向量化占位符 (Vectorization placeholder)
            │   ├── optimizer/            # 查询优化器 (Query optimizer)
            │   │   └── optimizer.go      # 优化器实现/包装 (Optimizer implementation/wrapper)
            │   ├── parser/               # SQL 解析器 (SQL parser)
            │   │   └── parser.go         # 解析器实现/包装 (Parser implementation/wrapper)
            │   ├── plan/                 # 查询计划节点 (Query plan nodes)
            │   │   └── plan.go           # 计划节点定义 (Plan node definitions)
            │   ├── scheduler/            # 分布式调度器 (Distributed scheduler)
            │   │   └── scheduler.go      # 调度器占位符 (Scheduler placeholder)
            │   └── transaction/          # 事务管理器 (Transaction manager)
            │       └── manager.go        # 事务管理器接口与实现 (Txn manager interface and implementation)
            ├── interfaces/               # 核心抽象接口定义 (Core abstraction interface definitions)
            │   └── storage.go            # 存储抽象层接口 (Storage Abstraction Layer interface)
            ├── internal/                 # 项目内部实现细节 (Internal implementation details)
            │   ├── encoding/             # 内部数据编码/解码 (Internal data encoding/decoding)
            │   │   └── encoding.go       # 内部编码工具 (Internal encoding utilities)
            │   └── utils/                # 内部工具函数 (Internal utility functions)
            │       └── utils.go          # 内部辅助函数 (Internal helper functions)
            ├── maintenance/              # 维护层 (Maintenance Layer)
            │   ├── diagnostic/           # 诊断工具 (Diagnostic tools)
            │   │   └── diagnostic.go     # 诊断功能实现 (Diagnostic features implementation)
            │   ├── metrics/              # 性能指标收集 (Performance metrics collection)
            │   │   └── metrics.go        # 指标收集实现 (Metrics collection implementation)
            │   └── status/               # 状态报告 (Status reporting)
            │       └── status.go         # 状态报告实现 (Status reporting implementation)
            ├── network/                  # 网络层 (Networking Layer)
            │   ├── mesh/                 # 服务网格集成 (Service mesh integration)
            │   │   └── mesh.go           # 服务网格占位符 (Service mesh placeholder)
            │   └── server/               # 基础网络服务 (Basic network server)
            │       └── server.go         # 网络服务基础结构 (Network server infrastructure)
            ├── protocol/                 # 数据库协议实现 (Database protocol implementations)
            │   └── mysql/                # MySQL 协议处理 (MySQL protocol handling)
            │       ├── auth.go           # MySQL 认证处理 (MySQL authentication handling)
            │       ├── connection.go     # 连接处理 (Connection handling)
            │       ├── handler.go        # GMS Handler 实现 (GMS Handler implementation)
            │       └── server.go         # MySQL 协议服务启动器 (MySQL protocol server starter)
            ├── security/                 # 安全层 (Security Layer)
            │   ├── audit/                # 审计日志 (Audit logging)
            │   │   └── audit.go          # 审计实现 (Audit implementation)
            │   ├── authn/                # 身份认证 (Authentication)
            │   │   └── authn.go          # 认证实现 (Authentication implementation)
            │   ├── authz/                # 访问控制 (Authorization)
            │   │   └── authz.go          # 授权实现 (Authorization implementation)
            │   └── crypto/               # 数据加解密 (Data encryption/decryption)
            │       └── crypto.go         # 加密实现 (Encryption implementation)
            ├── storage/                  # 存储层 (Storage Layer)
            │   ├── engines/              # 存储引擎实现 (Storage engine implementations)
            │   │   ├── badger/           # Badger KV 存储引擎 (Badger KV storage engine)
            │   │   │   ├── badger.go     # Badger 引擎适配器 (Badger engine adapter)
            │   │   │   ├── database.go   # 数据库级别操作 (Database level operations)
            │   │   │   ├── encoding.go   # Badger 特定的 K/V 编码 (Badger specific K/V encoding)
            │   │   │   ├── iterator.go   # 数据迭代器 (Data iterator)
            │   │   │   ├── table.go      # 表级别操作 (Table level operations)
            │   │   │   └── transaction.go # 事务适配 (Transaction adaptation)
            │   │   ├── kvd/              # KVD 引擎占位符 (KVD engine placeholder)
            │   │   │   └── kvd.go
            │   │   ├── mdd/              # MDD 引擎占位符 (MDD engine placeholder)
            │   │   │   └── mdd.go
            │   │   └── mdi/              # MDI 引擎占位符 (MDI engine placeholder)
            │   │       └── mdi.go
            │   └── sal/                  # 存储抽象层实现 (Storage Abstraction Layer implementation)
            │       └── adapter.go        # 适配器实现 (Adapter implementation)
            ├── docs/                     # 文档 (Documentation)
            │   └── architecture.md       # 架构设计文档 (Architecture design document)
            ├── scripts/                  # 构建、部署、测试脚本 (Build, deploy, test scripts)
            │   ├── build.sh
            │   └── test.sh
            ├── configs/                  # 配置文件示例 (Configuration examples)
            │   └── config.yaml.example
            ├── test/                     # 测试代码 (Test code)
            │   ├── integration/          # 集成测试 (Integration tests)
            │   │   └── integration_test.go
            │   └── unit/                 # 单元测试 (Unit tests)
            │       └── unit_test.go
            ├── .gitignore
            ├── go.mod
            ├── go.sum
            ├── LICENSE                   # 选择合适的开源许可证 (Choose an appropriate open-source license)
            └── README.md
            ```
    ``` 
    
    }
    
    特别地，因为mysql-like的数据库无论是理论、技术还是实现都已经非常多了，所以当前最优先的思路可以考虑MVP的价值达成，同时保证简单和易于扩展。
    [必须完整] MVP with BadgerDB backend
    [必须完整] MySQL protocol compatibility
    [接口定义] Distributed transaction support
    [接口定义] Service mesh integration
    [接口定义] Additional storage engines (MDD/MDI/KVD)
    [接口定义] Advanced query optimization
    [接口定义] Kubernetes operator
```


## 📁 包含内容

- **`claude-code-prompt.md`** - 用于实际的开发与代码重构
- **`claude-prompt.md`** - 用于架构规划与系统设计
- **`cursor-prompt.md`** - 用于调试与测试策略
- **`eskil-transcript.txt`** - Eskil 讲座完整文字稿（一小时纯金内容）
- **`usage-guide.md`** - 如何结合 AI 上下文工具使用这些提示

## 💼 实操案例

为了帮助你更好地理解这些原则在实践中的应用，这里提供了一系列详细的实操案例。

➡️ **[点击这里查看实操案例 (CASE_STUDIES.md)](./CASE_STUDIES.md)**

这些案例将向你展示如何从零开始，将一个混乱的系统重构为清晰、模块化的架构。

## 🚀 快速上手

1. **选择你的 AI 工具和相应的提示**

- Claude Code → 使用 `claude-code-prompt.md`
- Claude → 使用 `claude-prompt.md`
- Cursor → 使用 `cursor-prompt.md`

2. **提取你的代码上下文** (推荐)

```bash
# 对于 JavaScript/TypeScript 项目
npx repomix --include "src/**" --output context.xml

# 对于 Python 项目
python onefilellm.py ./src/ --output context.xml
```

4. **应用提示**

- 将提示内容粘贴到你的 AI 工具中
- 附上你的代码上下文
- 请求进行架构分析

## 💡 最佳实践

### 对于新项目

首先使用“规划提示” (`claude-prompt.md`) 来确立你的黑盒架构，然后用“开发提示” (`claude-code-prompt.md`) 来实现。

### 对于重构现有代码

1. 一次只专注于一个文件夹/模块
2. 使用上下文提取工具进行精确控制
3. 让 AI 识别出可以黑盒化的机会
4. 增量式地实现重构

### 黄金组合

这些提示与 AI 上下文工具结合使用效果最佳：

- [repomix](https://github.com/yamadashy/repomix) (JS/TS)
- [onefilellm](https://github.com/jimmc414/onefilellm) (Python)

## 📖 核心原则 (源自 Eskil)

> "今天写五行代码，比今天写一行、未来再回来修改要快得多。"

- **恒定的开发者速度**，不受项目规模影响
- **一个模块，一个人** - 实现完全的自主权和理解
- **一切皆可替换** - 你可以重写任何模块化组件
- **黑盒接口** - 清晰的 API 隐藏实现细节
- **可复用模块** - 可以在不同项目中使用的组件

## 💡 核心理念解析

为了更好地理解这些原则，以下是一些关键概念的通俗解释：

- **黑盒 (Black Box)**: 想象一个你不需要知道内部工作原理的设备，比如你的微波炉。你只需要知道按哪些按钮（接口），它就会帮你加热食物（功能）。在软件中，一个“黑盒”模块就是这样一个单元：你调用它的 API，它完成它的工作，你完全不必关心它内部的代码是如何实现的。这使得系统更容易理解和维护。

- **可替换性 (Replaceability)**: 如果你的微波炉坏了，你可以买一个不同品牌的新微波炉换上，只要你知道如何使用新按钮，它同样能加热食物。软件的可替换性意味着，如果一个模块写得很糟糕或技术过时，你可以从零开始重写一个新的模块，只要新模块遵循相同的 API 接口，整个系统就不会受到任何影响。

- **软件原生体 (Primitive)**: 这是你的系统中最核心、最基本的数据单元。就像乐高积木中的基础砖块，或者文字中的字母。在视频编辑器中，“原生体”可能是一个“视频片段”；在电商系统中，可能是“用户”或“订单”。定义好你的原生体，然后围绕它来构建所有功能，会让你的系统设计更加一致和强大。

## 🎬 原始来源

观看 Eskil Steenberg 的完整讲座: [Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)

这位传奇人物使用这些原则，完全用 C 语言构建了 3D 引擎、网络游戏和各种复杂系统。

## 🛠️ 示例

### 之前 (混乱)

```javascript
// DOM 操作散布在各个 React 组件中
const MentionInput = () => {
  const handleClick = (e) => {
    // 直接的 DOM 操作
    const selection = window.getSelection();
    const range = selection.getRangeAt(0);
    // 50多行光标定位逻辑...
  };
};
```

### 之后 (黑盒接口)

```javascript
// 清晰的接口 - 实现细节被隐藏
interface DOMAdapter {
  insertMention(mention: Mention, position: number): void;
  getCursorPosition(): number;
  updateContent(content: string): void;
}

// 现在可以支持 React, Vue, Svelte, 原生 JS
const MentionInput = ({ adapter }: { adapter: DOMAdapter }) => {
  const handleClick = () => adapter.insertMention(mention, position);
};
```

## 🔗 相关资源

- [Eskil 的视频讲座：Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)
- [GuoceDB 项目](https://github.com/turtacn/guocedb)
- [我是如何将任何 GitHub 仓库转为完美的 AI 上下文的](https://medium.com/vibe-coding/how-i-turn-any-github-repo-into-perfect-ai-context-game-changer-71919d497531)

---

_本项目与 Anthropic、Eskil Steenberg 或提及的任何工具均无关联，均为公共知识、经验的分享与提示。

