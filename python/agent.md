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