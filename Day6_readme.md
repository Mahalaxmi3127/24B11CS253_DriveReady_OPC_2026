# Day 6 – Multi-Agent Stationery Store Assistant with LangGraph

## Overview
This notebook builds a multi-agent assistant for **Campus Stationery Store**. An LLM classifies each customer message by intent and a LangGraph workflow routes it to the right agent: an FAQ agent for prices, stock, timings, and policies, or an order agent for placing orders. Greetings and unrelated messages receive a fixed welcome reply.

## Learning Objectives
- Define tools with the `@tool` decorator.
- Create separate agents with their own tools and system prompts using `create_agent`.
- Classify user intent with structured output (`with_structured_output`) and a Pydantic model.
- Define shared workflow state with `TypedDict`.
- Build a graph with nodes, edges, and conditional routing using LangGraph.
- Visualize the graph and test it with sample customer messages.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain`, `langchain-groq` | Agents, tools, and Groq chat model integration |
| `langgraph` | Builds the routing workflow (`StateGraph`) |
| `pydantic` | `Intent` model for structured classification output |
| Google Colab `userdata` | Reads the API key from Colab Secrets |
| `openai/gpt-oss-120b` | Chat model used by the agents and classifier (temperature 0.2) |

## Notebook Walkthrough

### 1. Install and connect the model
```python
!pip install -q langchain langchain-groq langgraph pydantic
```
The Groq key is read from the Colab secret `GROQ_KEY2` and set as `GROQ_API_KEY`. The model is created with `temperature=0.2` and `max_retries=3`.

### 2. Store data
- `ITEMS`: notebook ₹50, pen ₹10, pencil ₹5, eraser ₹5, geometry box ₹80, file ₹25, marker ₹20, calculator ₹350.
- `HOURS`: Monday–Saturday 9 AM to 7 PM; Sunday 10 AM to 2 PM.
- `orders`: an empty list that stores saved orders.

### 3. Tools
| Tool | What it does |
|---|---|
| `get_price(item, quantity)` | Returns the unit price and total, or lists available items if the item is unknown |
| `check_stock(item)` | Checks whether the item is listed in inventory |
| `is_open(day)` | Returns the opening hours for a weekday |
| `save_order(customer, item, quantity)` | Validates the order, calculates the total, and appends it to `orders` |
| `search_faq(question)` | Matches keywords in the question to store policies and returns up to 3 answers |

### 4. FAQ data
`FAQS` covers delivery, payment, returns, exchange, discount, bulk orders, timing, location, and refunds. `search_faq` maps keywords such as "deliver", "upi", or "refund" to the matching policy and replies "No matching policy found. Please ask store staff." when nothing matches.

### 5. Two agents
| Agent | Tools | Purpose |
|---|---|---|
| `faq_agent` | `search_faq`, `get_price`, `check_stock`, `is_open` | Answers questions about policies, prices, stock, and timings |
| `order_agent` | `get_price`, `check_stock`, `save_order` | Takes orders: confirms name, item, and quantity, saves only complete orders, and repeats the total |

Both prompts keep replies under 60 words. The FAQ prompt tells the agent never to invent facts and to direct unknown questions to store staff. `ask_agent()` sends one message to an agent and returns the final text reply.

### 6. Intent classification
```python
class Intent(BaseModel):
    intent: Literal["order", "faq", "other"]

classifier = llm.with_structured_output(Intent)
```
- `order`: a purchase request
- `faq`: a price or policy question
- `other`: a greeting or unrelated message

`handle()` routes a message by intent without LangGraph and is tested with "How much do 3 notebooks cost?"

### 7. LangGraph workflow
The shared state is `StoreState` with `message`, `intent`, and `reply`.

| Node | Action |
|---|---|
| `classify` | Detects the intent |
| `faq` | Runs the FAQ agent |
| `order` | Runs the order agent |
| `other` | Returns a welcome message |

The graph flow:
```
START → classify → (conditional route) → faq / order / other → END
```
`route()` returns the intent, and the conditional edges send the message to the matching node. `graph.compile()` builds the workflow, and `draw_mermaid_png()` displays it.

### 8. Test messages
The notebook runs these through the graph and prints the intent and reply for each:
- "Hello!"
- "How much do 4 pens cost?"
- "Do you have notebooks?"
- "What are Sunday hours?"
- "Do you deliver?"
- "I am Anu, order 2 geometry boxes."

### 9. View saved orders
The final cell prints every entry in the `orders` list.

## How to Run
1. Open the notebook in Google Colab.
2. Add your Groq API key as a Colab secret named `GROQ_KEY2` and enable notebook access.
3. Run the install and model cells.
4. Run the cells for tools, FAQs, agents, the intent classifier, the graph state, and the graph.
5. Run the graph display cell to view the workflow diagram.
6. Run the test cell, then the final cell to see saved orders.

## Key Takeaways
- Splitting work between specialized agents keeps each prompt and tool set focused.
- Structured output gives a reliable intent label for routing.
- LangGraph's conditional edges turn the intent into a clear workflow.
- Tools handle exact lookups, calculations, and saving data.
- Each message is handled on its own, so the agents have no memory between messages.

## Notes
- Orders are stored only in the in-memory `orders` list and disappear when the notebook session ends.
- `check_stock` only confirms an item is listed; it does not track quantities.
- The notebook has no saved outputs, so the test replies are not shown.
