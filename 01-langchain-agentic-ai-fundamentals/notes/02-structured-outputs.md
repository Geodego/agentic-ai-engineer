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

Function calling provides a more robust solution. Here, the model is given a
JSON schema and asked to call a specific function. This strongly constrains the
shape and types of the response, making the model more predictable and useful
in software systems. The application should still validate the result because
schema compliance does not guarantee that field values are factually correct.

### 1.4 Modeling Complex Data with Pydantic

When dealing with more complex outputs—like lists of tasks or nested objects—typed JSON becomes essential. Python’s `Pydantic` library enables schema enforcement through class definitions. For example:

```python
class ActionItem(BaseModel):
  title: str
  due_date: datetime
  owner: str
  status: Literal["open", "closed"]
```

This lets an application validate the generated output before using it. If the
model's response fails validation, the system can retry, rephrase the request,
or handle the error safely.

### 1.5 Designing for Reliability

Agents aren’t perfect and may return malformed data. Structured output mechanisms help catch and manage these errors. Using output parsers, schemas, and retry logic builds resilience into AI systems.

Structured outputs are foundational for agents that do more than talk. They enable integration, automation, and reliability—turning language models into powerful components of real-world workflows.

## 2. Structured Output Parsing with LangChain

The exercise uses a temperature-zero chat model configuration so that repeated
calls are less variable:

```python
from typing import List

from dotenv import load_dotenv
from langchain_classic.output_parsers.boolean import BooleanOutputParser
from langchain_classic.output_parsers.datetime import DatetimeOutputParser
from langchain_classic.output_parsers import OutputFixingParser
from langchain_core.exceptions import OutputParserException
from langchain_core.output_parsers import PydanticOutputParser, StrOutputParser
from langchain_openai import ChatOpenAI

load_dotenv()

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.0,
)
```

The examples below focus on the shape and type of each result. The exact text
or facts returned by an LLM may differ between calls, even at temperature zero.

### 2.1 Basic String Parsing

Calling `.invoke()` on the chat model returns an `AIMessage`, which includes
the generated content and response metadata:

```python
message = llm.invoke("hello")
text = message.content
```

Accessing `.content` extracts the response text directly. A `StrOutputParser`
performs the same transformation as a reusable LangChain runnable:

```python
parser = StrOutputParser()
text = parser.invoke(llm.invoke("hello"))
```

### 2.2 Datetime Parsing

A `DatetimeOutputParser` converts a correctly formatted response into a Python
`datetime` object. The prompt and parser must agree on the format:

```python
parser = DatetimeOutputParser()
message = llm.invoke(
    "Output a random datetime in %Y-%m-%dT%H:%M:%S.%fZ. "
    "Don't say anything else"
)
datetime_obj = parser.invoke(message)
```

For example, the string `2023-10-05T14:23:45.123456Z` becomes
`datetime.datetime(2023, 10, 5, 14, 23, 45, 123456)`. Extra prose or a
different date format can cause parsing to fail.

### 2.3 Boolean Parsing

A `BooleanOutputParser` converts `YES` and `NO` responses into Python booleans.
The restrictive prompt helps the model return a value the parser accepts:

```python
parser = BooleanOutputParser()
is_ai = parser.invoke(llm.invoke("Are you an AI? YES or NO only"))
is_human = parser.invoke(llm.invoke("Are you Human? YES or NO only"))
```

In the exercise, these calls return `True` and `False`, respectively.

### 2.4 TypedDict Parsing

`with_structured_output()` can use a `TypedDict` as the expected response
schema. Field descriptions tell the model what to extract, while the defaults
specify what to return when the input does not contain the requested details:

```python
from typing_extensions import Annotated, TypedDict

class UserInfo(TypedDict):
    """User's info."""

    name: Annotated[str, "", "User's name. Defaults to ''"]
    country: Annotated[str, "", "Where the user lives. Defaults to ''"]

llm_with_structure = llm.with_structured_output(UserInfo)
```

For this LangChain conversion, the values are interpreted positionally as:

```python
Annotated[field_type, default_value, description]
```

This interpretation is specific to LangChain's conversion of `TypedDict`
structured-output schemas. Python's `Annotated` type only attaches arbitrary
metadata to the underlying type; it does not define what that metadata means.
Other libraries may interpret or ignore the metadata differently.

The notebook demonstrates direct extraction, defaults, and inference:

```python
llm_with_structure.invoke(
    "My name is Henrique, and I am from Brazil"
)
# {'name': 'Henrique', 'country': 'Brazil'}

llm_with_structure.invoke("The sky is blue")
# {'name': '', 'country': ''}

llm_with_structure.invoke(
    "Hello, my name is the same as the capital of the U.S.  "
    "But I'm from a country where we usually associate with kangaroos"
)
# {'name': 'Washington', 'country': 'Australia'}
```

The last result is inferred rather than copied from the input. Structured
output controls the response shape; it does not restrict the model to literal
extraction or guarantee factual correctness.

### 2.5 Pydantic Parsing

Pydantic models add runtime construction and validation and return model
instances with attribute access. `Field` descriptions become part of the
schema presented to the model:

```python
from pydantic import BaseModel, Field

class PydanticUserInfo(BaseModel):
    """User's info."""

    name: Annotated[
        str,
        Field(description="User's name. Defaults to ''", default=None),
    ]
    country: Annotated[
        str,
        Field(description="Where the user lives. Defaults to ''", default=None),
    ]

llm_with_structure = llm.with_structured_output(PydanticUserInfo)
```

The return value is a `PydanticUserInfo` instance rather than a dictionary:

```python
structured_output = llm_with_structure.invoke("The sky is blue")
# PydanticUserInfo(name='', country='')

print(structured_output.name)
print(structured_output.country)
```

The fields can be accessed as attributes. In the exercise, both are empty
strings when the prompt contains no user information. The same capital and
kangaroo prompt from the `TypedDict` example produces
`PydanticUserInfo(name='Washington', country='Australia')`.

### 2.6 Parsing Complex Structures

A schema can contain collection fields. Here, `film_names` must be a list of
strings:

```python
class Performer(BaseModel):
    """Filmography info about an actor/actress."""

    name: Annotated[str, Field(description="name of an actor/actress")]
    film_names: Annotated[
        List[str],
        Field(description="list of names of films they starred in"),
    ]

llm_with_structure = llm.with_structured_output(Performer)
response = llm_with_structure.invoke(
    "Generate the filmography for Scarlett Johansson. Top 5 only"
)
```

`response` is a validated `Performer` instance. It can be serialized as JSON:

```python
json_response = response.model_dump_json()
```

`PydanticOutputParser` handles the inverse operation by parsing a JSON string
into a `Performer` instance:

```python
parser = PydanticOutputParser(pydantic_object=Performer)
performer = parser.parse(json_response)
```

The schema verifies the result's shape and types, but it does not verify that
the generated filmography is complete or accurate.

### 2.7 Handling Parsing Errors

`PydanticOutputParser` expects valid JSON matching its Pydantic model. The
exercise deliberately supplies a Python-style dictionary string with single
quotes, which is not valid JSON:

```python
misformatted_result = (
    "{'name': 'Scarlett Johansson', "
    "'film_names': ['The Avengers']}"
)

try:
    parser.parse(misformatted_result)
except OutputParserException as e:
    print(e)
```

The parser raises `OutputParserException` and reports an invalid JSON output.
Catch this exception at workflow boundaries so the application can retry,
repair, or reject the response explicitly.

### 2.8 Fixing Misformatted Outputs Automatically

`OutputFixingParser` uses another LLM call to try to repair a response that the
original parser rejected. It wraps the existing `PydanticOutputParser`:

```python
new_parser = OutputFixingParser.from_llm(parser=parser, llm=llm)
corrected_output = new_parser.parse(misformatted_result)
```

In the exercise, this produces a valid `Performer` instance. However, the
repairing model also adds film titles that were not present in the malformed
input. An output-fixing parser can therefore change semantic content, not just
punctuation or quoting. Validate important values after repair, and remember
that this fallback incurs another model call.

### 2.9 Conclusion

Structured output parsing transforms model responses into predictable Python
values. Simple parsers handle strings, datetimes, and booleans;
`with_structured_output()` maps responses to `TypedDict` or Pydantic schemas;
and `PydanticOutputParser` validates JSON against a model. Repair parsers can
recover from malformed syntax, but their output still requires semantic and
factual validation before it is trusted downstream.
