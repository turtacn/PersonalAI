## 《如何用一个 Prompt 把 Claude 变成系统工程天才——从传奇系统工程师演讲到 AI 驱动的项目架构实践》

**导语**

最近一位 C 语言系统工程师 Eskil Steenberg (看起来已然是非常成熟的程序员) 的演讲，彻底改变了许多人设计软件架构的方式。让许多人彻底重塑了思维，并由此打造了几条 AI Prompt，期望成为大家每次重构或启动新项目时最得力的伙伴。

<img src="images/one-prompt-changes-everything.png"  width="100%"/>

视频中，演讲者Eskil Steenberg通过他开发视频编辑器、数字医疗系统和战斗机软件的经验，总结出了一套软件架构实操方法；主要针对在开发大型软件项目时，如何通过模块化、平台抽象化、插件化等方式来构建一个可靠、可扩展、可维护的软件架构。其主要内容包括：

* **模块化**：将大型项目分解成小的、独立的“黑匣子”模块，每个模块都通过API暴露其功能，这样开发人员就可以在不了解其内部实现的情况下使用它们 [[07:46](http://www.youtube.com/watch?v=sSpULGNHyoI&t=466)]。
* **平台抽象化**：将所有第三方或特定平台的代码用自己的包装器进行封装，以隔离外部变化带来的影响，并更好地控制功能和未来开发 [[11:42](http://www.youtube.com/watch?v=sSpULGNHyoI&t=702)]。
* **思考“原始”数据类型**：识别应用程序操作的核心“原始”数据类型或概念 [[27:37](http://www.youtube.com/watch?v=sSpULGNHyoI&t=1657)]。
* **插件架构**：实现插件架构，将不同的功能（如视频格式、效果或传感器）由独立的插件来处理 [[34:51](http://www.youtube.com/watch?v=sSpULGNHyoI&t=2091)]。
* **格式设计**：定义用于存储或交换信息的格式（例如，API、文件格式、协议、编程语言），并平衡语义和结构，以实现代码的重用和简化 [[01:03:59](http://www.youtube.com/watch?v=sSpULGNHyoI&t=3839)]。
* **广泛的工具**：开发一套用于记录、回放、模拟和可视化的工具，以方便开发、测试和协作 [[56:20](http://www.youtube.com/watch?v=sSpULGNHyoI&t=3380)]。

**视频原文金句精选**

* “Instead of focusing on preventing AI from creating complex code, I can now break everything up into perfect ‘black boxes’…”
  → 不再害怕 AI 生成复杂代码，我可以将系统拆分为任何开发者（包括 AI）都能理解、替换的“黑盒”模块。

* “Constant developer velocity — regardless of project size.”
  → 确保开发者持续高速推进工作——无论项目规模大小。

* “One module, one person — one person should be able to do everything (given enough time).”
  → “单模块、一人负责”原则 —— 在足够时间内，一个人应能独立承担整个模块的设计与维护。

* “Everything should be replaceable — if you can’t understand it, you can rewrite it easily.”
  → 系统的每个部分都应具备可替换性——无法理解的模块，必须易于重写。

* “It’s faster to write five lines of code today than to write one line today and then have to edit it in the future.”
  → 今天写五行可能冗余的代码，胜过今天写一行将来还得改。

---

## 分析与验证

1. **Eskil Steenberg 的讲座内容验证**

   Eskil 在讲座《Architecting LARGE Software Projects》中确实阐释了将复杂项目拆分为可由个人完成的小模块、清晰接口与高可替换性的系统架构策略 ([YouTube][1], [Geeky Gadgets][2])。这些原则在许多关键应用领域（如医疗平台或战斗机系统）中具有高可参考价值 ([Geeky Gadgets][2])。

2. **GitHub “AI Architecture Prompts” 项目**

   在 Personal.AI 上，该项目将上述原则具体化为 AI Prompt，可以指导 Claude、ChatGPT、Cursor 等 AI 工具进行模块化设计、可替换组件开发、保持连贯开发节奏等操作 ([GitHub][3])。

3. **系统工程原则背景支持**

   Eskil 的这些观点与现代系统工程的核心理念一致，即强调“模块化、解耦、高内聚、低耦合”的设计哲学，与 INCOSE 等专业组织提出的系统工程原则高度契合 ([SEBoK][4], [ieeesystemscouncil.org][5])。

---

## 增强

---

### 如何用一个 Prompt 把 Claude 打造为系统工程大师级助手

**从传奇专家的系统工程演讲，到 AI 引领下的革命性开发实践**

回顾上个月，偶然看到 C 语言系统工程师 **Eskil Steenberg** 的讲座《Architecting LARGE Software Projects》。他构建从 3D 引擎到网络游戏的系统经验，仿佛照亮了我的开发黑暗——

> **“保持稳定的开发速度；单模块、一人负责；模块可替换；黑盒接口；写五行取代未来不断修改一行”**
> 这些核心原则令我恍然大悟：我面对的挑战远不及他设计战斗机系统时的复杂程度，但需要的是同样的思考方式。

于是，将此讲座转录后融入 AI 中，设计出三个 Prompt 工具：

* **Claude Code Prompt** — 直接操刀开发、重构
* **Claude Prompt** — 用于计划与架构设计
* **Cursor Prompt** — 专注调试与测试策略

搭配上下文提取工具（如 `repomix`、`onefilellm`），这些 Prompts 可以适配任何语言、框架，并引导 AI 用**系统规模**思维去分析问题 ([GitHub][3])。

在许多创意项目中，我将整个项目上下文与架构 Prompt 交给 Claude，结果令人惊艳——Claude 的架构建议的创意，叹为观止。

---

## 为什么这很重要：AI时代的软件架构新范式

* **模块化设计 = 可替换性：** 遵循 Eskil 所言，“一旦无法理解，我们得能轻易重写它”。以这种方式建构的系统，不再因 AI 随机生成复杂代码而头疼。
* **稳定开发节奏：** 先写五行胜过未来花时改一行，降低上下文切换成本，提高效率。
* **黑盒接口 + 清晰契约：** 每个模块都通过简洁 API 交互，易于理解、测试、替换。
* **AI友好架构：** Prompt 常驻意识形态，让 Claude / ChatGPT / Cursor 成为懂架构的助手，而不仅是代码生成器。


所有 Prompts、使用指南、演讲原文、示例项目打包上传至 GitHub——实用性已通过数周实战验证，无愧“创新工作流”的称号 ([GitHub][3])。

---

### 总结寄语

Eskil Steenberg 的系统工程思维，让我试着去想如何重新定义开发方式。在 AI 横空出世的时代，我们不再被复杂代码束缚：通过“黑盒 + 可替换模块 + 连续开发节奏”，AI 不是带来混乱，而是一种让系统更可控、更强大的力量。

如果你愿意挑战思维边界，我推荐你：

1. 观看 Eskil 的讲演视频，体验核心思想。
2. 参考 [Personal.AI-Architecure Prompts](https://github.com/turtacn/PersonalAI/blob/master/architecture-prompts)，感受 Prompt 在项目中的实际应用。
3. 尝试将这种方法用于新项目或重构老项目，看看 AI 与系统工程结合后的魔力。

---

希望这个软文，既有行业背景、理论支撑和实践路径，能让人更**深深信服**AI的前景，也激发更多人尝试 AI 驱动的系统工程方法。

* [1]: https://www.youtube.com/watch?v=sSpULGNHyoI&utm_source=chatgpt.com "Architecting LARGE software projects. - YouTube"
* [2]: https://www.geeky-gadgets.com/building-large-robust-software-systems/?utm_source=chatgpt.com "How to Build Scalable and Maintainable Software Systems - Geeky Gadgets"
* [3]: https://github.com/turtacn/PersonalAI/blob/master/architecture-prompts "GitHub - turtacn/PersonalAI/.../architecture-prompts"
* [4]: https://sebokwiki.org/wiki/Systems_Engineering_Principles?utm_source=chatgpt.com "Systems Engineering Principles - SEBoK"
* [5]: https://ieeesystemscouncil.org/files/ieeesyscouncil/2023-10/Systems%20Engineering%20Principles.pdf?utm_source=chatgpt.com "SYSTEMS ENGINEERING PRINCIPLES"
