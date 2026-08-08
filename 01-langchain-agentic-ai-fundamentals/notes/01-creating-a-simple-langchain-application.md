# Creating a Simple LangChain Application

## Table of contents

- [1. Introduction to LangChain](#1-introduction-to-langchain)
  - [1.1 Why Use LangChain?](#11-why-use-langchain)
  - [1.2 Core Components](#12-core-components)
    - [Models](#models)
    - [Messages](#messages)
  - [1.3 Final Thoughts](#13-final-thoughts)
- [2. Chat history and prompt templates](#2-chat-history-and-prompt-templates)
  - [2.1 Building Stateful Interactions with LLMs](#21-building-stateful-interactions-with-llms)
  - [2.2 Structure of a Conversation](#22-structure-of-a-conversation)
  - [2.3 Few-Shot Prompting for Better Responses](#23-few-shot-prompting-for-better-responses)
  - [2.4 Prompt Templates](#24-prompt-templates)
    - [2.4.1 Basic Prompt Templates](#241-basic-prompt-templates)
    - [2.4.2 ChatPromptTemplates](#242-chatprompttemplates)
    - [2.4.3 Few-Shot Prompt Templates](#243-few-shot-prompt-templates)
  - [2.5 Final Thoughts](#25-final-thoughts)

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
- Uses lightweight integration packages like `langchain-openai` or `langchain-anthropic`.
- Simplifies model interaction with the `invoke()` method, handling messages efficiently.

#### Messages

LangChain standardizes communication with models through message objects:

- `HumanMessage` – Represents user input.
- `AIMessage` – Represents model responses.
- `SystemMessage` – Provides additional instructions.
- `ToolMessage` – Used for function calling.

Example:

```python
llm.invoke([HumanMessage(content="What’s the capital of Brazil?")])
```

For convenience, LangChain automatically converts text inputs into the correct format.

### 1.3 Final Thoughts

LangChain provides a powerful and flexible foundation for LLM-based applications. Its structured approach simplifies model interactions, message handling, and system integration, allowing developers to build AI solutions efficiently. Understanding LLMs, messages, and workflows is key to making the most of LangChain’s capabilities.

## 2. Chat history and prompt templates

### 2.1 Building Stateful Interactions with LLMs

LLMs are stateless, meaning they do not remember previous interactions unless past messages are explicitly provided. To build cohesive conversations, applications must manage chat history and supply relevant context with each request.

### 2.2 Structure of a Conversation

A conversation typically consists of three key message types:

- `SystemMessage` – Sets the context for the interaction.

  ```python
  SystemMessage("You are a geography tutor")
  ```

- `HumanMessage` – Represents user input.

  ```python
  HumanMessage("What's the capital of Brazil?")
  ```

- `AIMessage` – Represents the model’s response.
- `ToolMessage` – Requests a tool invocation (for agent-based workflows).

A conversation is structured as a list of messages, which is then passed to the model.

```python
messages = [
        SystemMessage("You are a geography tutor"),
        HumanMessage("What's the capital of Brazil?")
]

llm.invoke(messages)
```

### 2.3 Few-Shot Prompting for Better Responses

By programmatically structuring chat history, developers can create examples of ideal interactions, guiding the model toward better responses.

This technique, called few-shot prompting, improves performance by providing examples of desired behavior.

- More examples enhance response quality.
- Larger prompts increase costs and latency.

To manage these trade-offs, LangChain provides the `FewShotPromptTemplate`, but first, understanding prompt templates is essential.

### 2.4 Prompt Templates

#### 2.4.1 Basic Prompt Templates

Define a single-variable input prompt with a placeholder.

```python
prompt = PromptTemplate(
    input_variables=["topic"],
    template="Tell me a joke about {topic}"
)
```

The prompt can be formatted using `.format()` or `.invoke()`:

```python
formatted_prompt = prompt.format(topic="Python")
llm.invoke(prompt.invoke({"topic": "Python"}))
```

#### 2.4.2 ChatPromptTemplates

Define prompts for structured conversations.

```python
template = ChatPromptTemplate([
        ("system", "You are a helpful AI bot. Your name is {name}."),
        ("human", "Hello, how are you doing?"),
        ("ai", "I'm doing well, thanks!"),
        ("human", "{user_input}"),
])
```

#### 2.4.3 Few-Shot Prompt Templates

Combines multiple structured examples to guide the LLM’s reasoning.

Components:

- `examples`: List of example dictionaries (`input` → `thought` → `output`).
- `example_prompt`: A `PromptTemplate` defining the format for each example.
- `suffix`: The actual question for the current prompt.

```python
examples = [
    {"input": "A train leaves City A...", "thought": "...", "output": "2 hours"},
    {"input": "A store applies a 20% discount...", "thought": "...", "output": "..."},
    ...
]

example_prompt = PromptTemplate(
    input_variables=["input", "thought", "output"],
    template="Question: {input}\nThought: {thought}\nResponse: {output}"
)

few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="Question: {input}",
    input_variables=["input"]
)

llm.invoke(few_shot_prompt.invoke({"input": "If today is Wednesday, what day will it be in 10 days?"}))
```

Each time this template is invoked, LangChain places the formatted few-shot examples before the current user question so they provide context for that request.

The result shows the model following the reasoning steps provided in the examples and outputting: "Saturday."

### 2.5 Final Thoughts

By managing chat history, structuring prompts, and leveraging few-shot learning, developers can build ChatGPT-like applications with better responses and task-specific optimizations. These tools provide the foundation for more powerful and interactive AI applications.
