# Product Decision Graph Implementation Documentation

## Overview
This implementation demonstrates a sophisticated product decision-making system using PydanticAI and Pydantic Graph. The system creates a debate-style evaluation of products, where pro and con agents argue about a product's merits, followed by a final decision on whether to buy or skip the product.

## Framework Understanding

### PydanticAI
PydanticAI is a powerful agent framework built by the Pydantic team that provides:
- Model-agnostic support (OpenAI, Anthropic, Gemini, Ollama, etc.)
- Type-safe agent development
- Structured responses using Pydantic models
- Dependency injection system
- Streaming capabilities
- Graph support through Pydantic Graph

### Pydantic Graph
Pydantic Graph is a companion library that enables:
- Definition of complex workflows using typing hints
- Async graph and state machine capabilities
- State management across nodes
- Type-safe graph execution

## Implementation Components

### Data Models

1. **Product**
```python
@dataclass
class Product:
    name: str
    url: str
    keywords: list[str]
    num_rounds: int = 0
```
Represents the product being evaluated with its basic information.

2. **Decision**
```python
@dataclass
class Decision:
    sentiment: int
    decision: Choice  # buy or skip
    explanation: str
```
Stores the final decision about a product with sentiment analysis.

3. **Argument**
```python
@dataclass
class Argument:
    sentiment: int
    body: str
```
Represents pro or con arguments about the product.

### Agents

1. **Pro Agent**
- Purpose: Generates positive arguments for the product
- Uses: Product search with positive sentiment
- Model: OpenAI GPT-4

2. **Con Agent**
- Purpose: Generates negative arguments against the product
- Uses: Product search with negative sentiment
- Model: OpenAI GPT-4

3. **Reasoning Agent**
- Purpose: Reviews arguments and makes final decision
- Model: Deepseek R1 (via Ollama)
- Output: Decision with sentiment and explanation

### Graph Nodes

1. **ModeratorNode**
- Controls the debate flow
- Alternates between pro and con agents
- Triggers final decision after ROUNDS complete

2. **ProNode**
- Executes pro_agent
- Generates positive arguments
- Updates state with new messages

3. **ConNode**
- Executes con_agent
- Generates negative arguments
- Updates state with new messages

4. **DecisionNode**
- Formats final decision
- Uses decision_format_agent
- Returns structured Decision object

### Tools

1. **product_search**
```python
def product_search(ctx: RunContext[str], sentiment: str) -> str:
    """Searches for product reviews with specified sentiment"""
```
- Uses Tavily API for targeted review search
- Limited to first 3 rounds per agent

2. **get_product_details**
```python
def get_product_details(ctx: RunContext[str]) -> str:
    """Scrapes product information from URL"""
```
- Uses BeautifulSoup for web scraping
- Provides product context to agents

## Data Pipeline Flow

1. **Input Processing**
   - Product information (name, URL, keywords) is provided
   - State object is initialized

2. **Debate Phase**
   - ModeratorNode controls flow
   - Alternates between ProNode and ConNode
   - Each node:
     * Searches for reviews
     * Gets product details
     * Generates arguments
     * Updates state

3. **Decision Phase**
   - Reasoning agent reviews all arguments
   - Generates sentiment and decision
   - Decision node formats final output

4. **Output**
   - Structured Decision object with:
     * Buy/Skip recommendation
     * Sentiment score
     * Detailed explanation

## Dependencies
- OpenAI API (for GPT-4)
- Ollama (for Deepseek R1)
- Tavily API (for search)
- BeautifulSoup4 (for web scraping)
- Pydantic and PydanticAI
- Colorama (for terminal output)

## Error Handling and Limitations
- Limited to 3 rounds of search per agent
- Web scraping depends on target site structure
- API rate limits may apply (Tavily, OpenAI)
- Requires valid API keys for external services

## Best Practices
1. Keep API keys in environment variables
2. Use async/await for efficient execution
3. Implement proper error handling
4. Monitor API usage and rate limits
5. Validate input data before processing

## Future Improvements
1. Add caching for product searches
2. Implement retry mechanisms for API calls
3. Add more product data sources
4. Enhance sentiment analysis
5. Add user feedback loop
