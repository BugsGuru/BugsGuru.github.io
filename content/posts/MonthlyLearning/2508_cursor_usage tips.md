---
title: "Cursor usage tips"
date: 2025-08-28

categories: [learning note]
tags: ["LLM","Cursor"]
keywords: ["LLM","Cursor"]

description: "cursor rules usage"
---

## Questions 请分享至少一个使用cursor(或类似AI编程工具)中的小技巧(或MCP工具)
参考文档：
1. https://x.com/ryolu_/status/1914384195138511142
    - Cursor首席设计师 发布的关于如何正确使用Cursor，一共 12 条方法
2. https://github.com/digitalchild/cursor-best-practices
   - 介绍了 Cursor 作为 AI 编辑器的配置与使用方法。
   - 规则（Rules）设置分为用户规则和项目规则，支持设置编码风格、命名规范等。
   - 提供了规则层级优先级和忽略文件机制的说明，有助于优化 AI 对代码库的理解。
   - 推荐写简洁且可复用的规则文件，给出具体示例。
3. https://www.siddharthbharath.com/coding-with-cursor-beginners-guide/
   - 详细说明了 Cursor 的三种主要模式及使用方法。
   - 建议使用规则文件统一团队或个人的偏好，如 TypeScript 项目的严格模式、Python 的 PEP8 规范等。
   - 强调通过规则让 AI 记住编码习惯，减少重复说明。
   - 提供了不同项目类型的规则配置示例，有助于建立一致的代码风格
4. https://www.prompthub.us/blog/top-cursor-rules-for-coding-agents
   - 大量引导 AI 遵循函数式编程、代码结构清晰、命名一致、类型安全优先（如 TypeScript）等现代编码最佳实践。
   - 关注代码安全、性能以及减少无用审核。
   - 尤其适合团队统一风格和提升生成代码质量。
5. https://mcp.so/
   - MCP站

### ⚙️规则驱动的AI辅助开发
大型语言模型的一个根本局限是，它们在每次交互后都会“忘记”之前的内容，除非你再次提供上下文。规则文件就是为了解决这个问题而生，它像一个持久化的“项目记忆”，能够让 AI 在任何时候都能遵循你预设的原则。

以下是如何利用这个技巧：
1. 创建项目规则文件：在你的项目根目录下创建一个 .cursor/rules 文件夹，并在其中创建规则文件（.mdc）。这些文件可以被版本控制系统（如 Git）追踪，确保整个团队都能使用同一套规范 。
2. 编写具体、可操作的规则：避免使用“写出好代码”这类模糊指令。相反，要提供具体、可操作的指导，并附上代码示例 。例如，你可以要求：
   - 强制使用 TypeScript 严格模式：明确告诉 AI“所有代码都必须使用 TypeScript 严格模式，避免使用 any 或 unknown 类型” 。 
   - 遵循 PEP8 规范：如果你在写 Python 项目，可以告诉 AI 遵循 PEP8 规范，例如“在二元运算符周围使用单个空格，但不要在函数参数默认值等号周围使用” 。 
   - 统一代码风格：要求 AI 遵循特定的命名约定，比如“所有目录名都使用小写加短横线（auth-wizard），组件名使用具名导出” 。
3. 利用不同层级的规则：Cursor 支持多种规则类型，它们有不同的应用范围和优先级 。你可以利用这些层级来精细化控制 AI 的行为：
   - 用户规则 (User Rules)：在 Cursor 设置中定义，适用于所有项目。例如，你可以设置一个全局规则来控制 AI 的回复风格，比如“请以简洁的风格回复，避免不必要的重复” 。
   - 项目规则 (Project Rules)：存储在项目目录中，只对当前项目生效。你可以使用“自动附加”（Auto Attached）规则，让 AI 在处理特定文件（如 app/controllers/**/*.go）时自动加载相应的规则 。