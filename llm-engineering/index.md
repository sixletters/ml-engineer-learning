---
layout: page
title: LLM Engineering & Agents
permalink: /llm-engineering/
---

Prompt engineering, tool calling, agents, RAG, and making LLM-based systems work reliably in production.

## Topics

| Topic | Depth |
|-------|-------|
| Prompt engineering: few-shot, chain-of-thought, structured outputs | Deep |
| Function/tool calling: schema design, error handling | Deep |
| LangGraph: state machines, conditional edges, memory, subgraphs | Deep |
| Pydantic for structured LLM outputs | Deep |
| RAG: chunking, embedding, retrieval, reranking | Deep |
| Observability: tracing LLM calls, cost tracking, latency | Deep |
| Evals: testing LLM behaviour systematically | Medium |

## Resources

- [Anthropic docs](https://docs.anthropic.com) — tool use and prompt engineering guides
- [LangGraph documentation](https://langchain-ai.github.io/langgraph/) — work through all how-to guides
- [Langfuse documentation](https://langfuse.com/docs) — observability patterns
- [Eugene Yan on LLM evals](https://eugeneyan.com/writing/llm-evaluations/)

## Key Exercises

- Build a structured extraction pipeline — give an LLM messy documents, define a Pydantic schema, use tool calling to force structured output. Measure extraction accuracy on 50 manually labelled examples
- Build a tool-using agent from scratch using the Anthropic SDK directly — no LangGraph yet. Implement the tool call loop yourself: call API → check for tool use → execute → send result → repeat. This makes LangGraph non-magical
- Reproduce a failure mode (looping, wrong output). Then add retry logic, output validation, and a fallback path. Observing failures firsthand is the fastest way to learn agent reliability
- Instrument an agent with Langfuse — every LLM call, tool call, input, output, and latency logged. Use the traces to find the most expensive step and optimise it
- Write an eval suite for an LLM task: 20 test cases with expected outputs, LLM-as-judge scorer, iterate on the prompt until the score improves
- Build a RAG pipeline: chunk documents, embed, store in Qdrant, retrieve on query, pass context to LLM. Measure answer quality with and without retrieval, then add a reranker and measure again

---

*Notes and exercises will be added below as I work through this section.*
