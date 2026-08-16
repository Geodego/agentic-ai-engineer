# Multi-Step Worflows in LangChain

## Table of contents

- [1. `Multi-Step Worflows`](#1-multi-step-worflows)
  - [1.1 The Evolution from Chains to Runnables in LangChain](#11-the-evolution-from-chains-to-runnables-in-langchain)
  - [1.2 Runnables: The New Standard](#12-runnables-the-new-standard)
    - [1.2.1 What Can Runnables Do?](#121-what-can-runnables-do)
    - [1.2.2 Common Runnable Input and Output Types](#122-common-runnable-input-and-output-types)
  - [1.3 LCEL: The Declarative Approach to Chains](#13-lcel-the-declarative-approach-to-chains)
  - [1.4 Final Thoughts](#14-final-thoughts)

## 1. `Multi-Step Worflows`

### 1.1 The Evolution from Chains to Runnables in LangChain

LangChain originally introduced Chains, which allowed developers to build sequential workflows by passing outputs from one step as inputs to the next. Over time, these legacy Chain classes have been deprecated in favor of more flexible and powerful approaches:

- LCEL (LangChain Expression Language) – A declarative way to compose AI workflows.
- LangGraph – A framework for agentic workflows with complex state management.

### 1.2 Runnables: The New Standard

The `Runnable` interface is now the core building block of LangChain. It standardizes how components—such as LLMs, output parsers, retrievers, and agent workflows—are executed and composed.

#### 1.2.1 What Can Runnables Do?

- Invoke – Process a single input into an output.
- Batch – Handle multiple inputs at once.
- Stream – Output data in chunks for real-time processing.
- Inspect – Access input, output, and configuration details.
- Compose – Chain multiple Runnables together for complex workflows.

Example of invoking a `Runnable` with custom configuration:

```python
some_runnable.invoke(
        some_input,
        config={
            'run_name': 'my_run',
            'tags': ['tag1', 'tag2'],
            'metadata': {'key': 'value'}
        }
)
```

#### 1.2.2 Common Runnable Input and Output Types

| Component | Input Type | Output Type |
| --- | --- | --- |
| `Prompt` | Dictionary | `PromptValue` |
| `ChatModel` | Single string, list of chat messages or a `PromptValue` | `ChatMessage` |
| `LLM` | Single string, list of chat messages or a `PromptValue` | String |
| `OutputParser` | The output of an `LLM` or `ChatModel` | Depends on the parser |
| `Retriever` | Single string | List of Documents |
| `Tool` | Single string or dictionary, depending on the tool | Depends on the tool |

### 1.3 LCEL: The Declarative Approach to Chains

LCEL (LangChain Expression Language) enables composing Runnables efficiently using a syntax similar to Linux pipes:

```python
chain = prompt | llm | output_parser
```

Instead of manually managing execution, LCEL automatically optimizes the workflow, making it easier to build scalable AI applications.

### 1.4 Final Thoughts

The shift from legacy Chains to Runnables and LCEL provides greater flexibility, efficiency, and composability. Developers can now build complex AI pipelines with less boilerplate code, focusing on defining workflows rather than managing execution.
