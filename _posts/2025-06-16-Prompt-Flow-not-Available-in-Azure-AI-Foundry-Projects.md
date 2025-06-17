---
title: Prompt Flow not Available in Azure AI Foundry Projects
date: 2025-06-16 19:03:00 -0500
categories: [GenAI, Prompt Flow]
tags: [prompt flow, logic apps, semantic kernel, lang chain, orchestration]
image:
  path: /assets/img/posts/2025-06-16-Prompt-Flow-not-Available-in-Azure-AI-Foundry-Projects.png
---

Recently, I have received several inquiries about the current status of Azure Prompt Flow, especially from partners who've noticed its absence from certain project types within Azure AI Foundry. To clarify: Prompt Flow remains fully available and actively supported—but [now only within **Hub-type projects** in Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/flow-develop#prerequisites) and Azure Machine Learning. In these contexts, Prompt Flow continues to shine by enabling intuitive, visual orchestration of Large Language Models (LLMs), prompt experiments, Python tooling, detailed tracing, version control, and collaborative workflows.

However, when working with **non-Hub AI Foundry projects**, Prompt Flow is no longer directly accessible. Naturally, this raises the question: "What alternatives do we have?"

## A Shift Towards Agent-Centric Frameworks

When Prompt Flow isn't an option, many teams have successfully transitioned to frameworks like **Semantic Kernel**, **LangChain**, and **AutoGen**. These frameworks provide robust mechanisms for defining workflows and intelligent agents directly in code. They deliver substantial flexibility, scalability, detailed traceability, and powerful integrations with memory management, external tools, and dynamic planning.

This shift isn't merely technical—it's part of a broader evolution towards **agent-first architectures**. Modern AI-driven solutions increasingly leverage agents that handle context awareness, dynamic planning, stateful interactions, and adaptive behaviors, aligning perfectly with what these frameworks provide.

## The Role of Logic Apps

Logic Apps frequently come up as a suggested alternative. It's true—Logic Apps can orchestrate workflows visually and integrate seamlessly with Azure services and external systems. However, it's important to clarify its role: Logic Apps excels in integration, automation, and event coordination, but doesn't provide native support for prompt design, agent memory management, or experimentation workflows that are core to Prompt Flow.

Rather than replacing Prompt Flow, Logic Apps fits better as a **complementary component**. It can act as a **global orchestrator** or **sub-orchestrator** that triggers or coordinates intelligent agents built with Semantic Kernel or LangChain. This allows Logic Apps to bridge business events and enterprise systems with the agent's internal logic, without handling the core intelligence layer itself.

## A Practical, Hybrid Approach

In practice, many successful teams adopt a hybrid strategy: leveraging Prompt Flow's visual capabilities during early prototyping and experimentation within Hub-type projects. Once validated, the logic and prompts are transitioned to code-based agents for production use. Logic Apps then takes on the role of an integration layer, enabling communication between those agents and the broader ecosystem of services.

## Conclusion

The future isn't about choosing one tool exclusively—it's about combining them strategically. Prompt Flow remains a powerful tool within its supported environments. Agent-centric frameworks like Semantic Kernel and LangChain offer depth and flexibility for code-first, scalable solutions. And Logic Apps enables smooth, low-code integration across the enterprise. By aligning these tools thoughtfully, teams can build robust, intelligent, and future-ready AI applications.
