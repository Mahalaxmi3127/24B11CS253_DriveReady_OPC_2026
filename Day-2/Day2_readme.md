# Day 2 – Campus Stationery Store Chatbot with Conversation Memory

## Overview
This notebook builds a command-line chatbot for **Campus Stationery Store** using Groq's OpenAI-compatible API. It combines a plain Python price function, a system prompt, conversation history, a sliding memory window, and error handling.

## Learning Objectives
- Connect to Groq using the `openai` client with a custom `base_url`.
- Store product prices in a Python dictionary and calculate totals in code.
- Write a system prompt with role, facts, rules, and format.
- Keep conversation history so the bot can remember details such as a customer's name.
- Limit memory with a sliding window (`MAX_TURNS`).
- Handle API errors such as rate limits and connection failures.
- Use a decorator to log function results.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `openai` | Client used to call Groq's OpenAI-compatible endpoint |
| `json`, `functools` | Format product data and build the logging decorator |
| `getpass` | Enter the API key without displaying it |
| `openai/gpt-oss-20b` | Chat model used by the chatbot |

## Notebook Walkthrough

### 1. Product data and system prompt
```python
PRODUCTS = {"notebook": 60, "pen": 10, "pencil": 5, "school bag": 500, "eraser": 5}
```
Prices are in rupees. The system prompt gives the assistant its role, the price list, and its rules: never promise delivery times, say "I'll check with the owner" when unsure, remember customer names and orders, and keep replies under 50 words.

### 2. Logging decorator and `get_price()`
```python
@log_call
def get_price(item: str, qty: int = 1) -> int:
    return PRODUCTS[item.lower().strip()] * qty
```
`log_call` prints each call's result. Totals are calculated by code, not by the model: 5 notebooks × ₹60 = **₹300**.

### 3. Chat function with memory
- `history` stores the system message and every user and assistant message.
- `MAX_TURNS = 8` limits how many recent messages are sent to the model.
- The system message is always included at the start of each request.

### 4. Error handling
The first version catches `RateLimitError` and `APIConnectionError` and returns a friendly message. The second version uses a general `except Exception` that returns the error text.

### 5. API connection
```python
client = OpenAI(api_key=key, base_url="https://api.groq.com/openai/v1")
client.models.list()
```
The key is entered with `getpass`, and a successful `models.list()` call confirms the connection.

### 6. Chat loop
A `while True` loop reads input and exits on `bye`, `exit`, or `quit`.

Sample run from the notebook:
```
You: What is my name?
Bot: I'm sorry, but I don't have that information. Could you please let me know your name?
You: Hi, my name is Ravi
Bot: Hello, Ravi! How can I help you with your stationery needs today?
```

## How to Run
1. Open the notebook in Google Colab.
2. Run the cell that installs `openai`, enters your Groq API key, and creates `client`.
3. Run the chatbot cell and type messages at the `You:` prompt.
4. Type `bye`, `exit`, or `quit` to stop.

Note: `chat()` uses `client`, so the connection cell must run before the first chatbot message is sent.

## Key Takeaways
- Use plain code for exact calculations such as totals.
- A system prompt with facts and rules keeps answers consistent.
- The model remembers only what is included in `history`.
- A sliding window keeps requests short.
- Handle API errors so the chatbot fails gracefully.

## Notes
This is a demonstration chatbot. It does not manage stock, take payments, or place real orders.
