---
title: "Tokenization and Embedding"
date: 2025-05-29

categories: [learning note]
tags: ["LLM"]
keywords: ["LLM"]

description: "What is tokenization and embedding"
---

## Questions 阅读资料并解释什么是 tokenization 和 embedding
在自然语言处理（NLP）和大语言模型（LLM）中，Tokenization（分词）和Embedding（嵌入）是两个关键步骤，它们将原始文本转换为模型可以理解和处理的形式。

#### 🧩 Tokenization（分词）
定义：Tokenization 是将原始文本拆分成更小的单元（称为 tokens）的过程，这些 tokens 可以是单词、子词、字符，甚至是符号。

目的：将文本转换为模型可处理的基本单位。

常见方法：
- 空格分词：基于空格将文本拆分成单词。
- WordPiece：将词分解为更小的子词单元，常用于 BERT 模型。
- Byte-Pair Encoding (BPE)：基于频率合并字符或子词，常用于 GPT 系列模型。

示例：
原始文本：“It's over 9000!”

- 空格分词：["It's", "over", "9000!"]
- WordPiece：["It", "'", "s", "over", "9000", "!"]
- BPE：["It", "'s", " over", " 9000", "!"]

每个 token 通常会被映射到一个唯一的数字 ID，供模型使用。

#### 🧠 Embedding（嵌入）
定义：Embedding 是将 tokens 映射到连续向量空间的过程，使得模型能够捕捉词汇的语义和上下文信息。
目的：将离散的 token ID 转换为模型可以处理的连续向量。

实现方式：
- 词袋模型（Bag of Words）：统计词频，忽略词序。
- TF-IDF（词频-逆文档频率）：考虑词频和逆文档频率的加权。
- Word2Vec、GloVe：基于上下文训练词向量。
- Transformer 模型中的嵌入层：如 BERT、GPT 等模型使用嵌入层将 token ID 转换为向量。

示例：
对于 token ID 序列 [101, 2009, 1005, 1055, 2058, 7706, 2692, 999, 102]，嵌入层将其转换为对应的向量表示，供模型进一步处理。

### 🔄 总结
| 步骤           | 输入          | 输出       | 作用                    |
|--------------|-------------|----------|-----------------------|
| Tokenization | 原始文本        | token 序列 | 将文本拆分为基本处理单元          |
| Embedding    | token ID 序列 | 向量表示     | 将 token 转换为连续向量，供模型处理 |

这两个步骤共同构建了 NLP 模型处理文本的基础，使得模型能够理解和生成自然语言。

## 参考
- [Build-A-Large-Language-Model](https://github.com/nufy323/Build-A-Large-Language-Model-CN/blob/main/cn-Book/2.%E5%A4%84%E7%90%86%E6%96%87%E6%9C%AC%E6%95%B0%E6%8D%AE.md)
- [LLM Embeddings Explained](https://huggingface.co/spaces/hesamation/primer-llm-embedding?section=torch.nn.embedding)