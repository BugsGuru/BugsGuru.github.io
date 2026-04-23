---
title: "Claude Code 对话压缩机制：MicroCompact、AutoCompact、ManualCompact"
date: 2026-04-23
categories: [learning note]
tags: ["Claude Code"]
keywords: ["MicroCompact", "AutoCompact", "ManualCompact", "Token"]
description: "基于 claude-code 源码梳理三种对话压缩机制的触发时机与分层设计原因"
---

## 背景

在长会话里，Agent 会不断积累历史消息、工具调用结果和中间思考，最终逼近模型上下文窗口上限。  
`claude-code` 为了解决这个问题，没有只做一种“统一压缩”，而是设计了多层机制：

- **MicroCompact**：轻量、优先、尽量不动核心对话语义
- **AutoCompact**：系统自动触发的总结式压缩
- **ManualCompact**：用户显式触发的 `/compact`

这三层组合起来，本质是“先局部瘦身，再自动兜底，最后用户可控介入”。

---

## 1) Claude Code 的对话压缩机制整体是怎样的？

从源码看，压缩链路并不是单点触发，而是分布在请求前后与命令层：

1. **请求前的轻量压缩（MicroCompact）**  
   在 `microCompact.ts` 中先尝试清理“可压缩工具结果”（如 file read/shell/grep/glob/web fetch/search 等历史输出），减少噪声 token。

2. **接近阈值时的自动压缩（AutoCompact）**  
   在 `autoCompact.ts` 中根据 token 使用量与阈值比较，超过阈值后触发自动压缩流程（优先 session memory compact，不行再走 `compactConversation` 生成摘要替换长历史）。

3. **用户手动压缩（ManualCompact）**  
   `/compact` 命令在 `src/commands/compact/compact.ts` 中实现，可由用户主动触发，还支持附加摘要指令（例如“总结时更关注 bug 线索”）。

---

## 2) 三种压缩方式分别在什么情况下触发？

## 2.1 MicroCompact 触发时机

### A. 时间驱动触发（time-based）

`microCompact.ts` 里有 `evaluateTimeBasedTrigger()`：  
当满足以下条件时触发：

- time-based 配置启用
- `querySource` 是主线程会话
- 当前距离上一条 assistant 消息的时间间隔超过 `gapThresholdMinutes`

触发后会**清空较旧 tool_result 内容**，只保留最近 `keepRecent` 条，清空内容替换为：
`[Old tool result content cleared]`。

这类压缩不依赖 LLM，总成本低，适合在“会话间隔长、缓存可能过期”的场景先做一次清道夫式瘦身。

### B. 缓存编辑触发（cached microcompact）

当特性开关、模型支持、主线程条件都满足时，会走 `cachedMicrocompactPath()`：

- 先登记可压缩工具结果
- 达到配置阈值后，生成 `cache_edits` 删除旧工具结果引用
- 不直接改本地 message 内容，而是在 API 层做缓存编辑

目标是：**在尽量不破坏缓存命中的前提下，删掉高 token、低语义价值的历史工具输出**。

---

## 2.2 AutoCompact 触发时机

`autoCompact.ts` 中，核心判断在 `shouldAutoCompact()`：

- 自动压缩功能开启（未被 `DISABLE_COMPACT` 或 `DISABLE_AUTO_COMPACT` 禁用，且用户配置允许）
- 当前 query source 不是禁止递归的特殊 source（如 `session_memory`、`compact`）
- `tokenCountWithEstimation(messages)` 超过自动阈值

阈值来自：

- `getEffectiveContextWindowSize(model)`（上下文窗口减去摘要输出预留）
- 再减 `AUTOCOMPACT_BUFFER_TOKENS = 13_000`

也就是说，AutoCompact 是“**快触顶时系统自动介入**”。

另外，源码还有一个重要保护：

- 连续失败计数（`consecutiveFailures`）
- 失败超过 `MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES = 3` 后触发熔断，不再反复尝试

目的是避免在不可恢复超长上下文里持续浪费 API 调用。

---

## 2.3 ManualCompact 触发时机

ManualCompact 由用户显式输入 `/compact` 触发（`src/commands/compact/index.ts` + `src/commands/compact/compact.ts`）：

- 用户主动要求压缩上下文
- 可附带自定义压缩说明（`/compact [instructions]`）
- 命令实现里会先尝试 session memory compact
- 再尝试 microcompact 预清理，最后走完整 `compactConversation`

所以 ManualCompact 的本质是：**把压缩控制权交给用户，在用户感知到“上下文快乱了/快满了”时手动干预**。

---

## 3) 为什么要设计成这种层次结构？

我理解这个分层的核心，是在四个目标之间做平衡：**成本、稳定性、上下文质量、可控性**。

### 3.1 先轻后重，控制成本

- MicroCompact 优先删除“工具输出垃圾体积”，通常不需要 LLM 重写摘要
- 只有在逼近阈值时才动用 AutoCompact 的总结式压缩

这能减少“每回合都做大摘要”的成本。

### 3.2 自动兜底，防止会话硬崩

- AutoCompact 持续监控 token 占用
- 逼近上限自动触发，避免 prompt 直接超长失败
- 熔断机制避免失败风暴

这是一层可靠性保障。

### 3.3 手动入口，保留用户意图优先级

- 用户有时更清楚“该保留什么、不该保留什么”
- `/compact` 允许在关键节点人工指定摘要方向

对“高价值上下文”的保留更可控。

### 3.4 缓存友好与语义安全的折中

- cached microcompact 尽量通过 cache edit 处理，减少缓存失效影响
- time-based microcompact 在缓存本就可能冷掉时再直接改内容

说明它不是只追求“压得越狠越好”，而是兼顾缓存命中、响应速度与语义完整度。

---

## 小结

`claude-code` 的三层压缩策略可以概括成一句话：  
**MicroCompact 负责“日常减脂”，AutoCompact 负责“临界抢救”，ManualCompact 负责“用户主导的精细手术”。**

这种架构比单一压缩策略更稳，因为它把“什么时候压、压多狠、谁来决定”拆开处理了。

---

## 参考

- [codeaashu/claude-code](https://github.com/codeaashu/claude-code)
- [microCompact.ts](https://github.com/codeaashu/claude-code/blob/main/src/services/compact/microCompact.ts)
- [autoCompact.ts](https://github.com/codeaashu/claude-code/blob/main/src/services/compact/autoCompact.ts)
- [compact command index](https://github.com/codeaashu/claude-code/blob/main/src/commands/compact/index.ts)
- [compact command implementation](https://github.com/codeaashu/claude-code/blob/main/src/commands/compact/compact.ts)
