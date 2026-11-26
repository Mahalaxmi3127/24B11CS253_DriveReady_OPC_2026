# Day 3 – Stationery Assistant with LangChain Prompt Chains

## Overview
This notebook rebuilds the Campus Stationery Store assistant using **LangChain**. It creates a reusable prompt template and chain, tests the assistant with fixed questions, compares model temperatures, and classifies tasks as plain code, a single LLM call, or an agent.

## Learning Objectives
- Connect LangChain's `ChatOpenAI` to Groq's OpenAI-compatible API.
- Write a structured system prompt with ROLE, FACTS, RULES, and FORMAT.
- Build a chain with `ChatPromptTemplate`, `MessagesPlaceholder`, and `StrOutputParser`.
- Run repeatable test questions against the assistant.
- Observe how temperature affects response variation.
- Decide when a task needs plain code, one LLM call, or an agent.
- Test conversation memory using the `history` placeholder.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain`, `langchain-openai` | Prompt templates, chains, and the chat model wrapper |
| `langchain_core` | `ChatPromptTemplate`, `MessagesPlaceholder`, `StrOutputParser` |
| Google Colab `userdata` | Reads the API key (`GROQ_KEY2`) from Colab Secrets |
| `openai/gpt-oss-20b` | Chat model served through Groq |

## Notebook Walkthrough

### 1. Starting code
The notebook begins with the earlier chatbot code (product dictionary, `get_price()`, and a chat loop). That cell expects `client` and `MODEL` from the Day 2 notebook, so the LangChain cells below are the main Day 3 work.

### 2. Install and import
```python
!pip install -q langchain langchain-openai
```

### 3. Connect to Groq
```python
llm = ChatOpenAI(
    model="openai/gpt-oss-20b",
    api_key=userdata.get("GROQ_KEY2"),
    base_url="https://api.groq.com/openai/v1",
    temperature=0.7
)
```
Add a Colab secret named `GROQ_KEY2` and enable notebook access.

### 4. System prompt
- **Facts:** Notebook ₹60, Pen ₹10, Pencil ₹5, School Bag ₹500, Eraser ₹5; orders take 15 minutes to prepare.
- **Rules:** use only the given facts, state exact prices, never invent prices, reply "I'll check with the store staff." when information is missing, calculate totals correctly, never promise delivery times, and offer no discounts.
- **Format:** under 50 words, in friendly English or Telugu matching the customer.

### 5. Prompt and chain
```python
prompt = ChatPromptTemplate.from_messages([
    ("system", SYSTEM),
    MessagesPlaceholder("history"),
    ("human", "{question}")
])
chain = prompt | llm | StrOutputParser()
```

### 6. Test questions
`run_tests()` sends these questions to the chain:
- What products are available?
- What is the price of a notebook?
- What is the price of 5 pens?
- Do you have a laptop?
- How long do orders take to prepare?

### 7. Temperature experiment
The model is recreated with `temperature=1.0` and the tests run twice. Comparing the two runs shows how higher temperature can change wording between runs.

### 8. Choosing the right approach
| Job | Best fit |
|---|---|
| Calculate the total price of 5 notebooks | Plain code |
| Answer customer questions about stationery products | One LLM call |
| Check stock availability using a store tool | Agent |

### 9. Memory check
The final cell asks a series of questions starting with "Hi, my name is Ravi." and ending with "What is my name?". Each call passes `"history": []`, so the model receives no earlier messages and cannot recall the name. To keep memory, append each question and answer to `history` and pass it into the chain.

## How to Run
1. Open the notebook in Google Colab.
2. Add `GROQ_KEY2` under Secrets and enable notebook access.
3. Run the install and import cells.
4. Run the cells that create `llm`, `SYSTEM`, `prompt`, and `chain`.
5. Run the test cells, then the temperature experiment, job classification, and memory check.

## Key Takeaways
- A chain (`prompt | llm | parser`) makes the assistant reusable and easy to test.
- Fixed test questions help check that the assistant follows its rules.
- Lower temperature gives more consistent answers; higher temperature gives more variety.
- Use plain code for exact math, one LLM call for language tasks, and an agent when tools are needed.
- The model remembers only the messages passed in `history`.

## Notes
This is a demonstration assistant. It has no stock database, payment system, or ordering tools.
