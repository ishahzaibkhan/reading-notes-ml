# AI Agents in LangChain

## Introduction to AI Agents

### Problem: Traditional Trip Planning
When planning a trip (e.g., Delhi to Goa, May 1-7), users face multiple challenges:

**Manual Steps Required:**
- **Flight/Train Booking**: Navigate websites like MakeMyTrip, GoIbibo, or IRCTC; fill forms with departure, destination, dates, and class; search and compare options for both onward and return journeys
- **Hotel Booking**: Specify location, check-in/check-out dates, number of guests; review prices, ratings, and amenities like breakfast
- **Itinerary Planning**: Research top attractions, build travel plans based on preferences, book local transportation and tickets

**Key Challenges:**
- **Time-Consuming**: Takes several days due to research, decision-making, and data aggregation
- **Complex**: Requires gathering data from multiple sources and making interconnected decisions
- **Hectic and Stressful**: Demanding process for most users
- **Accessibility Issues**: Difficult for older adults (60+), less tech-savvy individuals, or those with limited education
- **Unnatural Interaction**: Current websites don't offer intuitive, human-like interactions

### Solution: AI Agent as Chat Interface

An AI agent transforms this experience into a simple conversation:

**User Input:** "Can you create a budget travel itinerary from Delhi to Goa from First to Seventh May?"

**Agent's Process:**

1. **Intent Understanding**
   - Identifies: Delhi to Goa, specific dates, 7-day duration, budget preference
   - Formulates internal goal: "Plan complete itinerary + optimize cost" and "Plan affordable travel, stay, local movement, and activities over 7 days"

2. **Travel Options**
   - Uses IRCTC API or flight APIs to search options
   - Compares prices, duration, availability internally
   - Suggests: "Goa Express, sleeper/3AC class" with pricing
   - User confirms: "book 3AC ticket"

3. **Accommodation**
   - Queries hotel APIs with filters: budget (<₹700-₹1000/night), proximity to beaches, good reviews
   - Suggests: "A dorm room at The Hosteller for ₹650/night"
   - User approves

4. **Local Travel**
   - Recommends: "Scooter rentals are most cost-effective, ₹300/day for 6 days = ₹1800"
   - User agrees

5. **Itinerary Planning**
   - Uses APIs or knowledge base for popular attractions
   - Creates day-wise activities: beaches/forts (May 2nd), churches (May 3rd), South Goa (May 4th)
   - User finalizes

6. **Return Travel**
   - Offers options: train (₹800/₹1500) or early morning flight (₹2800)
   - User selects: "train se hi aana hai"

7. **Final Automation**
   - Presents Budget Summary: ~₹14,000 total (train, stay, scooter+fuel, food, sightseeing)
   - Upon approval, uses stored payment data and APIs to complete all bookings automatically
   - Sends invoices via email, adds events to calendar (train departure, hotel check-in), sets reminders

**Key Benefits:**
- Maintains context and remembers user preferences throughout
- Adapts based on new inputs
- Seamless experience with minimal user effort
- Effective across various domains (customer support, education, etc.)
- Makes interactions accessible and natural for all users

## What is an AI Agent?

### Technical Definition

An AI agent is an intelligent system that:
- Receives a high-level goal from a user
- Autonomously plans, decides, and executes a sequence of actions
- Uses external tools, APIs, or knowledge sources
- Maintains context and reasons over multiple steps
- Adapts to new information and optimizes for intended outcomes

**Key Principle:** Users tell the agent *what* to do, not *how* to do it.

### AI Agent vs. LLM

**AI Agent = LLM + Tools**

- **LLM (Reasoning Engine)**: Provides reasoning capability, helps think, make decisions, and understand natural language; good at reasoning but cannot perform actions
- **Tools (Action Performers)**: Enable actions like hitting APIs, making database changes, performing searches; allow interaction with the external world

### Core Characteristics of AI Agents

1. **Goal-Driven**: User specifies objectives, not methods
2. **Self-Planning**: Breaks down problems and executes steps autonomously
3. **Tool Awareness**: Knows available tools (APIs, databases) and when to use them
4. **Context Maintenance**: Remembers conversation history and ultimate goals
5. **Adaptability**: Re-plans based on new information (e.g., public holidays)
6. **Improved User Experience**: Offers natural, accessible system interactions
7. **Autonomy**: Performs tasks independently
8. **Transparency**: Shows thought process (thought trace) to users
9. **Optimization**: Aims for best possible outcomes

## Building an AI Agent in LangChain

### Prerequisites and Setup

**Requirements:**
- OpenAI API Key
- Libraries: `langchain-openai`, `langchain-community`, `duckduckgo-search`

**Basic Components:**
```python
from langchain_community.tools import DuckDuckGoSearchRun
from langchain_openai import ChatOpenAI

# Define Tool
search_tool = DuckDuckGoSearchRun()
# Test: search_tool.invoke("Top news in India today")

# Define LLM
llm = ChatOpenAI()
# Test: llm.invoke("Hi")
```

### Creating the Agent

**Import Required Components:**
```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub
```

**Pull ReAct Prompt:**
```python
react_prompt = hub.pull("hwchase17/react")
```
- ReAct is a common design pattern (Reasoning + Action) for AI agents
- Created by Harrison Chase (LangChain's creator)
- Available from LangChain Hub (repository for pre-defined prompts)

**Create Agent:**
```python
agent = create_react_agent(
    llm=llm,
    tools=[search_tool],
    prompt=react_prompt
)
```

**Agent vs. Agent Executor:**
- **Agent**: The "thinker" who plans, breaks down problems, and decides which tool to use
- **Agent Executor**: The "doer" who executes the agent's decisions (e.g., actually hits APIs)

**Create Agent Executor:**
```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=[search_tool]
)
```

### Invoking the Agent
```python
agent_executor.invoke(
    {"input": "your query"},
    verbose=True  # Shows thought process
)
```

**Example 1: Travel Query**
- Query: "What are the three ways to reach Goa from Delhi?"
- Thought: "I should search for the most common ways to reach Goa from Delhi..."
- Action: `duck_duck_go_search`
- Action Input: "three ways to reach Goa from Delhi"
- Observation: Search results about air, train, road
- Final Answer: "The three common ways to reach Goa from Delhi are Air, Train and Road."

**Example 2: Sports Query**
- Query: "Give a preview for today's IPL match between CSK and PBKS."
- Process: Uses `duck_duck_go_search` to find preview
- Final Answer: Provides match preview

## The ReAct Design Pattern

### What is ReAct?

**ReAct = Reasoning + Acting**

A design pattern that allows language models to interleave internal reasoning with external actions in a structured, multi-step process.

**ReAct vs. Single-Turn LLM:**
- Typical LLM: Send query → Get response (single-turn)
- ReAct: Multi-step, structured process combining reasoning and acting

### The ReAct Loop

ReAct operates in a continuous loop of three steps until the final answer is achieved:

1. **Thought**: Agent's internal reasoning, deciding the next logical step
2. **Action**: Specific tool to use with its input
3. **Observation**: Result/output received from the tool

**Example: "Population of the Capital of France?"**

**First Iteration:**
- Thought: "To answer this, I first need to find the capital of France."
- Action: Search Tool
- Action Input: "Capital of France"
- Observation: "Paris"

**Second Iteration:**
- Thought: "Now I know the capital is Paris. I need to find the population of Paris."
- Action: Search Tool
- Action Input: "Population of Paris"
- Observation: "2.1 million"

**Third Iteration:**
- Thought: "I now know both the capital and its population. I can answer the question."
- Final Answer: "Paris is the capital of France and it has a population of 2.1 million." (Loop breaks)

**Key Advantages:**
- Marries reasoning (LLM's intelligence) with acting (external tools)
- Transparent: entire thought trace visible to users (with `verbose=True`)
- Ideal for multi-step problems and tasks requiring tool usage

## Improving the Agent: Adding Multiple Tools

### Adding a Calculator Tool
```python
from langchain_core.tools import tool

@tool
def calculator(expression: str) -> str:
    """Useful for when you need to answer questions about math"""
    return str(eval(expression))
```

**Key Elements:**
- `@tool` decorator registers function as LangChain tool
- Name: "Calculator"
- Description helps LLM decide when to use the tool
- Uses Python's `eval()` to compute expressions

### Updating the Agent
```python
tools = [search_tool, calculator]

agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=react_prompt
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools
)
```

### Enhanced Agent Demonstrations

**Query 1: Pure Calculation**
- Input: "What is 100 * 345 * 2 + 100?"
- Thought: "I need to perform a calculation. I should use the Calculator tool."
- Action: Calculator
- Action Input: `100 * 345 * 2 + 100`
- Observation: 69100
- Final Answer: 69100

**Query 2: Combined Search and Calculation**
- Input: "What is the capital of France and what is 100 * 345 * 2 + 100?"
- Agent recognizes two distinct sub-problems
- Identifies "capital of France" requires `search_tool`
- Identifies mathematical expression requires `calculator`
- Intelligently orchestrates both tools
- Combines results into single, comprehensive answer
- Demonstrates multi-step reasoning and autonomous tool selection

## High-Level Flow of Agent Creation

1. **Create Tools**: Define all available tools (search, calculator, APIs, etc.)
2. **Create LLM**: Initialize the language model
3. **Create Agent**: Use `create_react_agent()` with LLM, tools, and design pattern prompt (e.g., ReAct)
4. **Create Agent Executor**: Use `AgentExecutor()` with agent and tools
5. **Invoke**: Call `agent_executor.invoke()` with query; agent performs tasks autonomously

## Summary

AI agents transform user experiences by:
- Accepting high-level goals instead of step-by-step instructions
- Autonomously planning and executing complex, multi-step tasks
- Intelligently selecting and using appropriate tools
- Maintaining context and adapting to new information
- Providing transparency through visible thought processes
- Making technology accessible to all users through natural conversation

The ReAct pattern is fundamental to building effective AI agents, enabling them to reason through problems while taking concrete actions using external tools.
