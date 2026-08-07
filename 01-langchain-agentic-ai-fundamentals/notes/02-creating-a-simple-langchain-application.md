# Creating a Simple LangChain Application

## 1. Introduction to LangChain
LangChain is a framework for building applications powered by large language models (LLMs). It simplifies LLM integration, making it easier to develop AI-driven solutions like chatbots, retrieval-augmented generation (RAG) systems, document summarization tools, and autonomous agents.

### 1.1 Why Use LangChain?
LangChain provides abstractions that connect LLMs with various tools, including:

- Cloud storage services for managing data.
- Web scraping tools for fetching real-time information.
- Vector databases for efficient search and retrieval.

This flexibility allows developers to focus on building AI solutions rather than handling complex integrations.

### 1.2 Core Components
LangChain supports multiple programming languages, including Python, JavaScript, and TypeScript. This lesson focuses on the Python libraries.

#### Models
- LangChain supports multiple LLM providers, such as OpenAI and Anthropic.
- Uses lightweight integration packages like langchain-openai or langchain-anthropic.
- Simplifies model interaction with the invoke() method, handling messages efficiently.

#### Messages
LangChain standardizes communication with models through message objects:

- HumanMessage – Represents user input.
- AIMessage – Represents model responses.
- SystemMessage – Provides additional instructions.
- ToolMessage – Used for function calling.

Example:

```python
llm.invoke([HumanMessage(content="What’s the capital of Brazil?")])
```

For convenience, LangChain automatically converts text inputs into the correct format.

#### Final Thoughts
LangChain provides a powerful and flexible foundation for LLM-based applications. Its structured approach simplifies model interactions, message handling, and system integration, allowing developers to build AI solutions efficiently. Understanding LLMs, messages, and workflows is key to making the most of LangChain’s capabilities.
