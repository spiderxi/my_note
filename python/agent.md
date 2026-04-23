# 1. 常识
*agent主要由哪些模块组成?*
```
🌟 Planning: 将任务拆分为todo list, 并支持反思(Self-Reflection)
🌟 Memory: 分为短期记忆和长期记忆
🌟 Tools: 如skills和MCP
🌟 LLM

🌙 短期记忆通过滑动上下文窗口 / 摘要实现, 长期记忆主要通过向量数据库实现(RAG机制)
🌙 反思主要通过ReAct(Reasoning + Acting)实现, 即按照Thought -> Action -> Observation 的范式循环交互
```

*Skill由什么组成？*
```
🌟 SKILL.md:        给LLM看的技能说明书（简明扼要）
🌟 scripts/:        可执行脚本（Python/Shell/JS 等）
🌟 references/:     给LLM看的详细参考资料（API文档、示例等）
```

*MCP是作用什么?*
```
MCP是LLM集成Tool的通信协议(类似 HTTP 之于 Web), 由服务提供者提供MCP Server(本质是一个适配器)
```

*如何部署本地LLM?*
```
使用LM Studio或Ollama等开源工具
```

*当前(2026Q2)Agent主流开发框架有哪些?*
```
🌟 langchain / langgraph

🌙 langchain链式调用封装度高, 适合中小型agent; langgraph更底层并支持图控流程
```

# 2. Agent 各模块

## IO模块
*提示词工程本质上是什么?*
```
各种提示词工程, 其实质是控制模型在预测时能看到的前文上下文

🌙 最终LLM看到的上下文, 实际上是拼接后的对话文案(含分隔符Token), LLM自回归地预测下一个token:
<分隔toekn> system: 你是 Kimi <分隔toekn> 
<分隔toekn> user: 1 + 1 = ? <分隔toekn> 
<分隔toekn> assistant:  <分隔toekn> 

🌙 提示词工程中常用的方式是提示词模板
```

*怎么解析LLM的输出?*
```
在提示词中明确让LLM以特定格式输出, 例如按JSON格式输入, 然后用解析库解析

🌙 最好在提示词中给出输入输出示例
```

## Memory
*短期记忆怎么实现?*
```
将 滚动对话历史/对话历史摘要 放到LLM上下文中

🌙 最终的提示词往往是"{系统提示词} + {工具schema} + {对话历史} + {用户输入}"
🌙 当前(2026Q2)主流LLM上下文窗口100K~1M Token
```

*长期记忆怎么实现?*
```
RAG: 将知识文档/长期记忆存储到向量数据库中, 用户使用时匹配向量, 将匹配到的向量对应的文档补充到LLM上下文中
```

## Tool
*LLM怎么调用工具?*
```
🌟 方式一: 在提示词中添加工具schema并明确要求以JSON格式表示调用，由agent解析输出结果
🌟 方式二: 部分LLM原生支持Tool Calling

🌙 原生Tool Calling在API层面，会有单独的入参（工具列表）和出参（工具调用）
```

*原生Tool Calling怎么实现的?*
```
通过SFT(Supervised Fine-Tuning，监督微调)让LLM学会工具调用能力。
LLM返回的原始token中包含特殊的工具token，这些token是人工加入词表后通过训练习得的。

🌙 LLM的训练过程分为两个阶段: 
    1. 预训练: 使用广义语言数据（无特殊token，培养泛化能力）
    2. SFT: 使用带特殊token的工具调用数据集训练
🌙 SFT阶段训练语料中，包含大量工具调用相关的特殊token样本
🌙 推理引擎可配置约束解码(Constraint Decoding)，强制LLM输出符合指定Schema的工具调用格式
```

*推理引擎和LLM的关系?*
```
🌟 LLM本身是一组文件（权重参数+配置+词表），描述神经网络的结构和参数
🌟 推理引擎是LLM的"运行时"，主要作用有:
* 将模型加载到GPU显存中
* 将矩阵运算编译为高效的GPU kernel（如CUDA）
* KV Cache管理，避免重复计算
* 负责推理全流程: 
    文本 → Tokenize → 嵌入 → 前向传播获取Logits（未归一化的概率分布向量） 
    → 根据Logits和采样策略（temperature/top-p等）决定输出Token
```


