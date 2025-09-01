# AI 架构设计提示 - 源自 Eskil Steenberg 的《大型软件项目架构》讲座

借鉴 Eskil Steenberg 传奇般的系统编程讲座，利用 AI 驱动的架构原则，将任何代码库转变为模块化、可替换的“黑盒”。

## 🎯 这是什么

三个专门设计的 AI 提示，用于训练 Claude、ChatGPT 和 Cursor 等模型，使其能够以下列思维方式进行思考：

- **黑盒接口 (Black box interfaces)** - 模块之间清晰的 API
- **可替换组件 (Replaceable components)** - 如果你无法理解一个模块，就重写它
- **恒定开发速度 (Constant velocity)** - 今天写5行代码，胜过未来修改1行
- **单一职责 (Single responsibility)** - 一个模块，一个人负责

## 🔥 真实成果

我曾使用这些提示，完整重构了 [Mentis](https://github.com/Alexanderdunlop/mentis) 项目——一个因 DOM 操作 bug 而饱受困扰的 React @提及组件。AI 建议我采用一个黑盒化的 DOM 接口，现在它已经可以零开销地支持多种前端框架。

**之前**: 混乱的 DOM 操作，每次 React 更新都会出问题。
**之后**: 清晰的接口，同时支持 React、Vue、Svelte 和原生 JS。

## 📁 包含内容

- **`claude-code-prompt.md`** - 用于实际的开发与代码重构
- **`claude-prompt.md`** - 用于架构规划与系统设计
- **`cursor-prompt.md`** - 用于调试与测试策略
- **`eskil-transcript.txt`** - Eskil 讲座完整文字稿（一小时纯金内容）
- **`usage-guide.md`** - 如何结合 AI 上下文工具使用这些提示

## 💼 实操案例

为了帮助你更好地理解这些原则在实践中的应用，我们提供了一系列详细的实操案例。

➡️ **[点击这里查看实操案例 (CASE_STUDIES.md)](./CASE_STUDIES.md)**

这些案例将向你展示如何从零开始，将一个混乱的系统重构为清晰、模块化的架构。

## 🚀 快速上手

1. **克隆本仓库**

```bash
git clone git@github.com:Alexanderdunlop/ai-architecture-prompts.git
```

2. **选择你的 AI 工具和相应的提示**

- Claude Code → 使用 `claude-code-prompt.md`
- Claude → 使用 `claude-prompt.md`
- Cursor → 使用 `cursor-prompt.md`

3. **提取你的代码上下文** (推荐)

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
- [原始博文](medium-link)
- [Mentis 项目](https://github.com/Alexanderdunlop/mentis)
- [我是如何将任何 GitHub 仓库转为完美的 AI 上下文的](https://medium.com/vibe-coding/how-i-turn-any-github-repo-into-perfect-ai-context-game-changer-71919d497531)

## 🤝 贡献

对这些提示有改进建议？或者在有趣的项目上尝试过它们？欢迎提交 PR！

---

_本项目与 Anthropic、Eskil Steenberg 或提及的任何工具均无关联。这些是在真实开发工作中经过实战检验的提示。_
