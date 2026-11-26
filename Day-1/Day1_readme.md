# Day 1 – Building a Bakery AI Assistant with Groq

## Overview
This notebook builds an LLM-powered customer-support assistant for **Sweet Crumbs Home Bakery**. It uses Groq-hosted models through LangChain's `ChatGroq` integration and shows how a system prompt controls an assistant's role, tone, business knowledge, and response limits.

## Learning Objectives
- Connect a Python notebook to the Groq API.
- Load an API key securely from Google Colab Secrets.
- Initialize a chat model with a chosen model name and temperature.
- Separate system instructions from the customer's message.
- Use `llm.invoke()` to send messages and receive a response.
- Apply basic prompt-injection detection using a prompt-guard model.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain-groq` | LangChain integration for Groq chat models |
| `groq` | Groq Python SDK for direct API calls |
| Google Colab `userdata` | Reads the API key from Colab Secrets |
| `os` | Sets the API key as an environment variable |
| `openai/gpt-oss-120b` | Chat model used for the bakery assistant |
| `meta-llama/llama-prompt-guard-2-22m` | Prompt-guard model used in the security example |

## Notebook Walkthrough

### 1. Install and import
```python
!pip install langchain_groq
from langchain_groq import ChatGroq
```

### 2. Configure the API key
```python
import os
from google.colab import userdata

os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
```
Add a secret named `GROQ_API_KEY` in Colab and enable notebook access. Never commit API keys to GitHub.

### 3. Initialize the model
```python
llm = ChatGroq(model="openai/gpt-oss-120b", temperature=0)
```
A temperature of `0` gives more consistent, less varied replies.

### 4. Define the system prompt
The `SYSTEM` prompt sets the bakery's facts and rules:
- Cakes are baked to order.
- Orders need 24 hours' notice.
- All cakes have eggless versions.
- Never promise delivery times; say so if the answer is unknown.
- Replies must be polite and under 60 words.

### 5. Create a reusable function
```python
def ask(question):
    return llm.invoke([("system", SYSTEM), ("human", question)]).content
```
The function sends the system instructions with the customer's question; `.content` extracts the reply text.

### 6. Test question
```python
print(ask("can i get my order tommorrow"))
```
The assistant explains the 24-hour notice policy and avoids guaranteeing a delivery time.

### 7. Prompt-guard example
The notebook calls `meta-llama/llama-prompt-guard-2-22m` with a message that tries to override earlier instructions. The model returns a numeric score (`0.9988957643508911` in the saved output) for that input. This check is separate from the bakery chat function.

## How to Run
1. Open the notebook in Google Colab.
2. Add `GROQ_API_KEY` under Secrets and enable notebook access.
3. Run the installation cell.
4. Run the import, API key, and model initialization cells.
5. Define `SYSTEM` and `ask()`, then run the test question.
6. Optionally run the prompt-guard cell.

## Key Takeaways
- Clear system instructions shape LLM behavior.
- Business facts and policies should be supplied explicitly.
- A reusable function simplifies repeated model calls.
- API keys belong in secrets, not in source code.
- Guard models add a safety layer but do not guarantee every unsafe request is blocked.

## Notes
The bakery is a demonstration only. It does not check inventory, accept payments, or schedule deliveries.
