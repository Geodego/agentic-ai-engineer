# Structured Outputs

## Table of contents

- [1. Structured outputs](#1-structured-outputs)
  - [1.1 Structured Outputs: Making AI Responses Actionable](#11-structured-outputs-making-ai-responses-actionable)
  - [1.2 Why Prompting Isn’t Enough](#12-why-prompting-isnt-enough)
  - [1.3 Output Parsers and Function Calling](#13-output-parsers-and-function-calling)
  - [1.4 Modeling Complex Data with Pydantic](#14-modeling-complex-data-with-pydantic)
  - [1.5 Designing for Reliability](#15-designing-for-reliability)
- [2. Structured Output Parsing with LangChain](#2-structured-output-parsing-with-langchain)
  - [2.1 Basic String Parsing](#21-basic-string-parsing)
  - [2.2 Datetime Parsing](#22-datetime-parsing)
  - [2.3 Boolean Parsing](#23-boolean-parsing)
  - [2.4 TypedDict Parsing](#24-typeddict-parsing)
  - [2.5 Pydantic Parsing](#25-pydantic-parsing)
  - [2.6 Parsing Complex Structures](#26-parsing-complex-structures)
  - [2.7 Handling Parsing Errors](#27-handling-parsing-errors)
  - [2.8 Fixing Misformatted Outputs Automatically](#28-fixing-misformatted-outputs-automatically)
  - [2.9 Conclusion](#29-conclusion)

## 1. Structured outputs

### 1.1 Structured Outputs: Making AI Responses Actionable

Agents are more powerful when they return structured, machine-readable outputs like JSON instead of just plain text. Structured responses allow agents to integrate with other systems, power automation, and trigger actions in workflows. For example, rather than summarizing a customer call in text, an agent can extract structured fields like:

```json
{
  "issue_type": "login_problem",
  "urgency": "high",
  "customer_email": "jane@mail.com"
}
```

This format can be directly used in ticketing systems, dashboards, or alerts. Free-form text may be readable, but structured data is what enables downstream tools to act.

### 1.2 Why Prompting Isn’t Enough

A common early method was to prompt the model to output JSON, such as: “Return the answer as JSON with fields: `issue_type`, `urgency`, `customer_email`.” While this sometimes works, models often return vague or invalid data, like `"urgency": "very"` or `"email": "none found"`, which causes errors when code tries to use it. LLMs are trained for natural language, not strict type rules.

### 1.3 Output Parsers and Function Calling

To improve reliability, output parsers can validate whether the model's response conforms to a defined schema. If valid, the output is parsed; if not, fallback strategies can be applied.

Function calling provides an even more robust solution. Here, the model is given a JSON schema and asked to call a specific function. The model must output a structured function call object that matches the schema. This guarantees correct format and types, turning the model into something more predictable and usable in software systems.

### 1.4 Modeling Complex Data with Pydantic

When dealing with more complex outputs—like lists of tasks or nested objects—typed JSON becomes essential. Python’s `Pydantic` library enables schema enforcement through class definitions. For example:

```python
class ActionItem(BaseModel):
  title: str
  due_date: datetime
  owner: str
  status: Literal["open", "closed"]
```

This ensures that agents generate only valid outputs. If the model's response fails validation, the system can retry, rephrase the request, or handle the error safely.

### 1.5 Designing for Reliability

Agents aren’t perfect and may return malformed data. Structured output mechanisms help catch and manage these errors. Using output parsers, schemas, and retry logic builds resilience into AI systems.

Structured outputs are foundational for agents that do more than talk. They enable integration, automation, and reliability—turning language models into powerful components of real-world workflows.

## 2. Structured Output Parsing with LangChain

### 2.1 Basic String Parsing

By default, calling `.invoke()` on an LLM returns an `AIMessage` object.
To extract the raw text, access `ai_message.content`.

```python
response = llm.invoke("Hello there.")
raw_text = response.content
```

Alternatively, a `StrOutputParser` can be used to transform the output cleanly.

```python
parser = StrOutputParser()
text = parser.invoke(response)
```

### 2.2 Datetime Parsing

A `DatetimeOutputParser` is used when you need to convert LLM output into a Python `datetime` object.
The LLM is prompted to produce a date in a specific format.

```python
parser = DatetimeOutputParser()
datetime_obj = parser.invoke(response)
```

### 2.3 Boolean Parsing

A `BooleanOutputParser` converts "yes" or "no" responses into Python `True` or `False`.

Example:

Content: "yes" → `True`
Content: "no" → `False`

```python
parser = BooleanOutputParser()
result = parser.invoke(AIMessage(content="yes"))
```

### 2.4 TypedDict Parsing

LangChain supports using `TypedDict` to define the structure of expected output.

```python
class UserInfo(TypedDict):
  name: str
  country: str
```

Using `with_structured_output(UserInfo)`, the model is guided to format its response accordingly.

Examples:

Input: "My name is Henrique and I am from Brazil." → `{ "name": "Henrique", "country": "Brazil" }`

If no relevant info is found, defaults are used.

### 2.5 Pydantic Parsing

For more robust parsing and validation, Pydantic models are used.

```python
class UserInfo(BaseModel):
  name: str
  country: str
```

Pydantic models provide automatic type checking and better error handling.

```python
parsed = llm.with_structured_output(UserInfo).invoke("My name is Washington and I am from Australia.")
```

If the LLM output is properly structured, parsing succeeds.
If missing information, fields default to empty strings or `None`, based on model configuration.

### 2.6 Parsing Complex Structures

A more complex example is parsing a list of films (filmography) for an actor using a Pydantic model.

```python
class Performer(BaseModel):
  name: str
  film_names: List[str]
```

Asking for "Scarlett Johansson filmography" returns the correct structured object with movie names.

### 2.7 Handling Parsing Errors

Sometimes the LLM outputs poorly formatted JSON or semi-structured text.
If parsing fails (e.g., bad quotes, wrong format), an `OutputParserException` is raised.

```python
try:
  parser.invoke(bad_output)
except OutputParserException as e:
  print("Parsing error caught!")
```

### 2.8 Fixing Misformatted Outputs Automatically

LangChain provides an `OutputFixingParser`.

This parser:

- Detects format errors.
- Attempts to reformat the output using the LLM itself.

```python
fixing_parser = OutputFixingParser.from_llm(parser, llm)
corrected_output = fixing_parser.invoke(misformatted_output)
```

This enables parsing even from imperfect LLM outputs, making workflows much more reliable.

### 2.9 Conclusion

Structured output parsing transforms unstructured LLM responses into reliable Python objects.
TypedDicts and Pydantic models improve structure and validation.
Parsers combined with automatic fixing allow workflows to handle imperfect LLM behavior gracefully.
