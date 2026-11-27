# Day 4 – Stationery Store AI Agent with Tools and Google Sheets

## Overview
This notebook builds a tool-using AI agent for **Campus Stationery Store**. The agent uses LangChain and a Groq-hosted model to answer product questions, calculate prices, check store hours, and save confirmed orders to a Google Sheet.

## Learning Objectives
- Connect LangChain's `ChatGroq` to a Groq-hosted model.
- Authenticate Google Colab with Google Sheets using `gspread`.
- Create a spreadsheet and write order records to it.
- Validate order data with a Pydantic model.
- Define tools with the `@tool` decorator.
- Create an agent with `create_agent` that chooses the right tool for each request.
- Keep conversation history across turns.
- Read saved orders back from the sheet.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain`, `langchain-groq` | Agent creation and Groq chat model integration |
| `gspread` | Reads and writes Google Sheets |
| `google.colab` (`userdata`, `auth`) | Colab Secrets and Google account authentication |
| `pydantic` | `Order` model that validates order data |
| `openai/gpt-oss-20b` | Chat model used by the agent (temperature 0.2) |

## Notebook Walkthrough

### 1. Install and connect the model
```python
!pip install -q langchain langchain-groq gspread
```
The API key is read from the Colab secret `GROQ_KEY2`. The model is created with `temperature=0.2` and `max_retries=3`.

### 2. Connect Google Sheets
`auth.authenticate_user()` signs in to Google, and `gspread.authorize(creds)` connects the notebook to Sheets.

### 3. Create the orders sheet
A spreadsheet named **Campus Stationery Orders** is created with the header row:

`name | item | qty | date | total`

### 4. Product data
```python
MENU = {"notebook": 60, "pen": 10, "pencil": 5, "school bag": 500, "eraser": 5}
```
Prices are in rupees. `HOURS` is an empty dictionary, so store hours are not available.

### 5. `Order` model
| Field | Rule |
|---|---|
| `name` | Customer's first name |
| `item` | One of: notebook, pen, pencil, school bag, eraser |
| `qty` | Integer, at least 1 |
| `date` | Pickup date as the customer said it |

### 6. Agent tools
| Tool | What it does |
|---|---|
| `get_price(item, qty)` | Returns the total price, e.g. `5 x notebook = Rs 300`, or lists available products if the item is unknown |
| `is_open(day)` | Looks up store hours; replies "I'll check with the store staff." when hours are not available |
| `log_order(name, item, qty, date)` | Validates the order with `Order`, calculates the total, and appends a row to the Google Sheet |

### 7. System prompt
The prompt defines the assistant's role, product prices, and 10 rules, including:
- Use `get_price` for prices and `is_open` for store timings.
- Never invent prices, products, or opening hours.
- Collect name, product, quantity, and pickup date before using `log_order`.
- Save an order only when the customer clearly asks to place it, then read it back with the total.
- Never promise an exact delivery time or offer discounts.
- Keep replies under 60 words, in English or Telugu to match the customer.

### 8. Create the agent
```python
agent = create_agent(
    model=llm,
    tools=[get_price, is_open, log_order],
    system_prompt=SYSTEM
)
```

### 9. Chat function
`chat(text)` sends the saved `history` plus the new message to the agent, updates `history` with the returned messages, and returns the last reply. If an error occurs, it prints the error and returns a polite retry message.

### 10. Test conversations
The notebook tests:
- "What products are available?"
- "How much are 5 notebooks?"
- "Are you open on Sunday?"
- "My name is Ravi. Please place an order for 5 notebooks for 30 November."
- "What is the total price?"
- "Show me my order details."

### 11. Verify saved orders
```python
records = ws.get_all_records()
for record in records:
    print(record)
```
This prints each saved order from the Google Sheet.

## How to Run
1. Open the notebook in Google Colab.
2. Add your Groq key as a Colab secret named `GROQ_KEY2` and enable notebook access.
3. Run the install and model cells.
4. Run the Google authentication cell and allow access to your Google account.
5. Run the cells in order to create the sheet, tools, system prompt, agent, and `chat()` function.
6. Run the test conversations, then the final cell to view saved orders.

## Key Takeaways
- Tools let an agent perform exact actions such as price calculation and saving data.
- A Pydantic model checks order data before it is saved.
- The system prompt controls when each tool should be used.
- Passing `history` back to the agent gives it memory across turns.
- Store hours stay unavailable until `HOURS` is filled in.

## Notes
- The notebook sets `GROQ_KEY2` as an environment variable, but `ChatGroq` reads `GROQ_API_KEY` by default. If the model connection fails, set `os.environ["GROQ_API_KEY"]` to the same key or pass `api_key=` to `ChatGroq`.
- The last two code cells are identical.
- A new spreadsheet is created each time the sheet-creation cell runs.
- This is a demonstration agent. It does not manage stock or take payments.
