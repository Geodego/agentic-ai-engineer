# Multi-Step Worflows in LangChain

## Table of contents

- [Related notebooks](#related-notebooks)
- [1. `Multi-Step Worflows`](#1-multi-step-worflows)
  - [1.1 The Evolution from Chains to Runnables in LangChain](#11-the-evolution-from-chains-to-runnables-in-langchain)
  - [1.2 Runnables: The New Standard](#12-runnables-the-new-standard)
    - [1.2.1 What Can Runnables Do?](#121-what-can-runnables-do)
    - [1.2.2 Common Runnable Input and Output Types](#122-common-runnable-input-and-output-types)
    - [1.2.3 Chaining Invocations](#123-chaining-invocations)
    - [1.2.4 Runtime Configuration and Inspection](#124-runtime-configuration-and-inspection)
    - [1.2.5 Composing Runnables](#125-composing-runnables)
    - [1.2.6 Turning Functions into Runnables](#126-turning-functions-into-runnables)
    - [1.2.7 Parallel Runnables](#127-parallel-runnables)
  - [1.3 LCEL: The Declarative Approach to Chains](#13-lcel-the-declarative-approach-to-chains)
  - [1.4 Final Thoughts](#14-final-thoughts)

## Related notebooks

- [Demo: LCEL and Runnables](../demos/03_LCEL.ipynb) — worked examples of composing and inspecting Runnables.
- [Exercise: Multi-Step Workflow](../exercises/03_multi_step_workflow.ipynb) — build an AI business-advisor workflow using LCEL.

Both notebooks contain cells that may call the OpenAI API and incur usage charges.

## 1. `Multi-Step Worflows`

### 1.1 The Evolution from Chains to Runnables in LangChain

LangChain originally introduced Chains, which allowed developers to build sequential workflows by passing outputs from one step as inputs to the next. Over time, these legacy Chain classes have been deprecated in favor of more flexible and powerful approaches:

- LCEL (LangChain Expression Language) – A declarative way to compose AI workflows.
- LangGraph – A framework for agentic workflows with complex state management.

### 1.2 Runnables: The New Standard

The `Runnable` interface is now the core building block of LangChain. It standardizes how components—such as LLMs, output parsers, retrievers, and agent workflows—are executed and composed.

#### 1.2.1 What Can Runnables Do?

- `invoke(input)` – Process a single input into an output.
- `batch(inputs)` – Process multiple inputs and return their results in the
  same order.
- `stream(input)` – Yield pieces of the result as they become available.
- Inspect – Access input, output, and configuration schemas.
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

Compatible input and output types are important when Runnables are composed:
each component must be able to accept the result produced by the previous
component.

### 1.2.3 Chaining Invocations

A prompt, chat model, and string output parser can be called individually and
nested to create a three-stage workflow:

```python
from dotenv import load_dotenv
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableLambda, RunnableParallel, RunnableSequence
from langchain_core.tracers.context import collect_runs
from langchain_openai import ChatOpenAI

load_dotenv()

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.0,
)
prompt = PromptTemplate(
    template="Tell me a joke about {topic}"
)
parser = StrOutputParser()

parser.invoke(
    llm.invoke(
        prompt.invoke(
            {"topic": "Python"}
        )
    )
)
```

`load_dotenv()` expects `OPENAI_API_KEY` to be available through the local
environment. Invoking `ChatOpenAI` makes a request to the OpenAI API and may
incur usage charges.

The nested invocation passes a value through three stages:

1. The input dictionary is formatted by `PromptTemplate` into a prompt value.
2. `ChatOpenAI` sends that prompt to the model and returns an `AIMessage`.
3. `StrOutputParser` extracts the message content as a plain string.

The parser makes the final result easier to display and pass into components
that expect text rather than a LangChain message object.

### 1.2.4 Runtime Configuration and Inspection

Every Runnable exposes methods for inspecting its input, output, and
configuration schemas:

```python
for runnable in [prompt, llm, parser]:
    print(runnable.get_input_schema())
    print(runnable.get_output_schema())
    print(runnable.config_schema())
```

The optional `config` dictionary attaches operational information such as a
run name, tags, and metadata. It does not change the prompt or model response.
For example, `collect_runs()` can capture an invocation locally so its trace
details can be inspected:

```python
with collect_runs() as run_collection:
    result = llm.invoke(
        "Hello",
        config={
            "run_name": "demo_run",
            "tags": ["demo", "lcel"],
            "metadata": {"lesson": 2},
        },
    )

run_collection.traced_runs
```

### 1.2.5 Composing Runnables

`RunnableSequence` connects components from left to right. Each component
receives the previous component's output:

```python
chain = RunnableSequence(prompt, llm, parser)
result = chain.invoke({"topic": "Python"})
```

The composed chain supports the same common execution methods as its
components. It can stream one input or batch several inputs:

```python
for chunk in chain.stream({"topic": "Python"}):
    print(chunk, end="", flush=True)

results = chain.batch([
    {"topic": "Python"},
    {"topic": "Data"},
    {"topic": "Machine Learning"},
])
```

The chain's structure can also be inspected as a graph:

```python
chain.get_graph().print_ascii()
```

### 1.2.6 Turning Functions into Runnables

`RunnableLambda` adapts a Python callable to the Runnable interface so it can
be invoked or composed like other LangChain components:

```python
def double(x: int) -> int:
    return 2 * x


runnable = RunnableLambda(double)
runnable.invoke(2)
```

### 1.2.7 Parallel Runnables

`RunnableParallel` sends the same input to each branch. The branches can run
independently, and their results are collected in a dictionary whose keys
match the names supplied to the Runnable:

```python
parallel_chain = RunnableParallel(
    double=RunnableLambda(lambda x: x * 2),
    triple=RunnableLambda(lambda x: x * 3),
)

parallel_chain.invoke(3)
```

Its structure is also available through the graph inspection interface:

```python
parallel_chain.get_graph().print_ascii()
```

### 1.3 LCEL: The Declarative Approach to Chains

LCEL (LangChain Expression Language) overloads the pipe operator (`|`) to
compose Runnables. Therefore, `RunnableSequence(prompt, llm, parser)` and
`prompt | llm | parser` create equivalent prompt-to-model-to-parser pipelines:

```python
chain = prompt | llm | parser
result = chain.invoke({"topic": "computer"})
```

Instead of manually nesting each invocation, the pipeline defines the workflow
once and retains the Runnable interface for invocation, batching, streaming,
inspection, and further composition. LCEL automatically optimizes the composed
workflow, reducing boilerplate and making it easier to build scalable AI
applications.

### 1.4 Final Thoughts

The shift from legacy Chains to Runnables and LCEL provides greater flexibility, efficiency, and composability. Developers can now build complex AI pipelines with less boilerplate code, focusing on defining workflows rather than managing execution.
