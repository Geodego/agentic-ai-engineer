# Multi-Step Worflows

## Table of contents

- [1. Output Parsers](#1-output-parsers)
  - [1.1 String Parser](#11-string-parser)
  - [1.2 Other Parsers](#12-other-parsers)
    - [Datetime](#datetime)
    - [Boolean](#boolean)
- [2. Structured](#2-structured)
  - [2.1 Dict Schema](#21-dict-schema)
  - [2.2 Pydantic](#22-pydantic)
- [3. Dealing with Errors](#3-dealing-with-errors)
  - [3.1 Fixing Parser](#31-fixing-parser)

```python
from typing import List
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_classic.output_parsers.datetime import DatetimeOutputParser
from langchain_classic.output_parsers.boolean import BooleanOutputParser
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.exceptions import OutputParserException
from langchain_classic.output_parsers import OutputFixingParser
from dotenv import load_dotenv
```

```python
load_dotenv()
```

```python
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.0,
)
```

## 1. Output Parsers

### 1.1 String Parser

```python
llm.invoke("hello")
```

```python
llm.invoke("hello").content
```

```python
parser = StrOutputParser()
```

```python
parser.invoke(
    llm.invoke("hello")
)
```

### 1.2 Other Parsers

#### Datetime

```python
llm.invoke(
    "Output a random datetime in %Y-%m-%dT%H:%M:%S.%fZ. "
    "Don't say anything else"
)
```

```python
parser = DatetimeOutputParser()
```

```python
parser.invoke(
    llm.invoke(
        "Output a random datetime in %Y-%m-%dT%H:%M:%S.%fZ. "
        "Don't say anything else"
    )
)
```

#### Boolean

```python
llm.invoke(
    "Are you an AI? YES or NO only"
)
```

```python
parser = BooleanOutputParser()
```

```python
parser.invoke(
    input=llm.invoke(
        "Are you an AI? YES or NO only"
    )
)
```

```python
parser.invoke(
    input=llm.invoke(
        "Are you Human? YES or NO only"
    )
)
```

## 2. Structured

### 2.1 Dict Schema

```python
from typing_extensions import Annotated, TypedDict

class UserInfo(TypedDict):
    """User's info."""
    name: Annotated[str, "", "User's name. Defaults to ''"]
    country: Annotated[str, "", "Where the user lives. Defaults to ''"]
```

```python
llm_with_structure = llm.with_structured_output(UserInfo)
```

```python
llm_with_structure.invoke(
    "My name is Henrique, and I am from Brazil"
)
```

```python
llm_with_structure.invoke(
    "The sky is blue"
)
```

```python
llm_with_structure.invoke(
    "Hello, my name is the same as the capital of the U.S.  "
    "But I'm from a country where we usually associate with kangaroos"
)
```

### 2.2 Pydantic

```python
from pydantic import BaseModel, Field

class PydanticUserInfo(BaseModel):
    """User's info."""
    name: Annotated[str, Field(description="User's name. Defaults to ''", default=None)]
    country: Annotated[str, Field(description="Where the user lives. Defaults to ''", default=None, )]
```

```python
llm_with_structure = llm.with_structured_output(PydanticUserInfo)
```

```python
structured_output = llm_with_structure.invoke("The sky is blue")
```

```python
structured_output
```

```python
print(structured_output.name)
```

```python
print(structured_output.country)
```

```python
structured_output = llm_with_structure.invoke(
    "Hello, my name is the same as the capital of the U.S.  "
    "But I'm from a country where we usually associate with kangaroos"
)
```

```python
structured_output
```

## 3. Dealing with Errors

```python
class Performer(BaseModel):
    """Filmography info about an actor/actress"""
    name: Annotated[str, Field(description="name of an actor/actress")]
    film_names: Annotated[List[str], Field(description="list of names of films they starred in")]
```

```python
llm_with_structure = llm.with_structured_output(Performer)
```

```python
response = llm_with_structure.invoke(
    "Generate the filmography for Scarlett Johansson. Top 5 only"
)
response
```

### 3.1 Fixing Parser

```python
response.model_dump_json()
```

```python
parser = PydanticOutputParser(pydantic_object=Performer)
```

```python
parser.parse(response.model_dump_json())
```

```python
misformatted_result = "{'name': 'Scarlett Johansson', 'film_names': ['The Avengers']}"
```

```python
try:
    parser.parse(misformatted_result)
except OutputParserException as e:
    print(e)
```

```python
new_parser = OutputFixingParser.from_llm(parser=parser, llm=llm)
```

```python
new_parser.parse(misformatted_result)
```

```python

```
