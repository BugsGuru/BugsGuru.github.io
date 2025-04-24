---
title: "LLM Decoding Strategy"
date: 2025-04-24

categories: [learning note]
tags: ["LLM"]
keywords: ["LLM"]

description: "Decoding Strategy for Large Language Model (LLM)"
---

## Questions
#### 一、以下是几个目前大语言模型常用的解码策略(一般在调用模型的API中可以设置)，请解释它们的基本原理及使用方式
贪心搜索 (Greedy search)、波束搜索 (Beam search)、Top-K 采样 (Top-K sampling) 以及 Top-p 采样 (Top-p sampling)

#### 二、请解释 温度(temperature) 的基本原理及使用方式


## Answer
### 一、解码策略基本原理及使用方式
#### 贪心搜索（Greedy Search）
📌 原理：

每一步都选择概率最高的下一个 token（词/子词）。

📈 特点：
- 最快，计算简单； 
- 可能错过全局最优解； 
- 生成的文本往往比较“机械”或者“无趣”。

🧠 举个例子：

输入：“The weather is” 模型预测："sunny"（90%），"cloudy"（5%），"rainy"（5%）

→ 贪心搜索会 直接选择 "sunny"


#### 波束搜索（Beam Search）
📌 原理：

同时保留 k 个最有可能的候选序列（beam width），每一步扩展所有候选，然后选出得分最高的 k 个序列继续。

📈 特点：
- 比贪心更好，能找到更优的整体结果； 
- 但会带来重复、啰嗦问题（比如重复短语）； 
- 计算成本高于贪心。

🔧 参数：

beam width（波束宽度）常用值：3、5、10

🧠 类比：

像是在走迷宫，不止走一条路，而是并行尝试几条路，最后选得分最高的那条。


#### Top-K 采样
📌 原理：

每一步只从概率最高的 K 个 token 中随机采样一个。

📈 特点：
- 增加多样性（比贪心/beam 更随机）； 
- 控制生成的多样性和质量； 
- 如果 K 选得太小，会像贪心；太大，会生成胡话。

🔧 参数：

K 常见值：10, 40, 50

🧠 举个例子：
输入：“I love to eat” 预测的下一词：["pizza"(0.5), "sushi"(0.2), "pasta"(0.15), ...]
→ Top-K=3，只在前三中随机采样。


#### Top-p 采样（又称 Nucleus Sampling）
📌 原理： 

从累积概率加起来超过 p 的 token 列表中随机采样。相比 Top-K 更灵活：不是定量选，而是定概率。

📈 特点：
- 自适应地控制采样集合大小； 
- 在不牺牲质量的前提下提高多样性； 
- 被认为比 Top-K 更自然、现代。

🔧 参数：

p 常见值：0.8 ~ 0.95

🧠 举个例子：

如果概率列表为： ["pizza"(0.5), "sushi"(0.3), "burger"(0.1), "salad"(0.05), ...]

→ Top-p=0.9，则只从 "pizza", "sushi", "burger" 中随机采样。

### 二、温度(temperature) 的基本原理及使用方式
📌 原理简介：

温度是在采样前，对模型输出的概率分布进行缩放的一个系数，它不会改变概率的排序，但会影响概率差距的大小。

🔥 不同 Temperature 值的效果

| 温度 T 值  | 效果描述                         |
|---------|------------------------------|
| T = 1.0 | 原始概率分布，默认设置                  |
| T < 1.0 | 概率分布变“尖锐”，高概率的 token 更突出     |
| T > 1.0 | 概率分布变“平滑”，低概率的 token 更有机会被选中 |
| T → 0   | 趋近于贪心搜索（argmax）              |
| T → ∞   | 所有 token 几乎等概率（纯随机）          |

🧠 举个例子
假设模型在某一步输出以下概率：

```
Token logits:     [5.0, 3.0, 1.0]
Softmax (T=1.0):  [0.84, 0.11, 0.05]
Softmax (T=0.5):  [0.96, 0.03, 0.01] ← 更“贪心”
Softmax (T=1.5):  [0.68, 0.19, 0.13] ← 更“随机”
```
可以看到，温度越高，分布越“平”，更容易出现意想不到的词；越低，生成越保守但更可靠。

🔧 实际使用方式（以 Hugging Face Transformers 为例）
```python
model.generate(
    input_ids,
    do_sample=True,          # 开启采样（非贪心）
    temperature=0.8,         # 设置温度
    top_p=0.9,               # 可选：与 top-p 一起用
    max_new_tokens=50,
)
```
通常建议：

- 生成“逻辑性强”的文本 → temperature=0.7 ~ 0.9
- 想要更有创造力的写作 → temperature=1.0 ~ 1.5

## 参考
- [How to generate text](https://huggingface.co/blog/how-to-generate)
- [top_p诞生的论文](https://ar5iv.labs.arxiv.org/html/1904.09751?_immersive_translate_auto_translate=1)