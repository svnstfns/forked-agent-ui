# Agno Framework Features and Configuration Guide

## Introduction

This guide covers all the features available in the Agno framework and how to configure them properly, with special attention to integration with development environments like Cursor IDE.

## Core Features

### 1. Agent Communication

Agno provides multiple ways for agents to communicate and coordinate:

#### Direct Agent-to-Agent Communication
```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

# Create specialized agents
researcher = Agent(
    name="Researcher",
    model=OpenAIChat(id="gpt-4o"),
    tools=[WebSearch()],
    description="Research specialist"
)

writer = Agent(
    name="Writer",
    model=OpenAIChat(id="gpt-4o"),
    tools=[FileWriter()],
    description="Content writer"
)

# Agents can pass information through shared context
research_result = researcher.run("Research topic X")
writer.run(f"Write article based on: {research_result}")
```

#### Team-Based Communication
```python
from agno.team import Team

# Team with collaborative mode
team = Team(
    name="Content Team",
    agents=[researcher, writer],
    mode="collaborate",  # Agents discuss and reach consensus
    description="Research and content creation team"
)

# Team handles communication internally
result = team.run("Create comprehensive article on AI")
```

**Communication Modes:**
- **Route**: Leader agent routes tasks to specialized agents
- **Coordinate**: Agents work sequentially, passing results
- **Collaborate**: Agents discuss via shared context to reach decisions

#### Shared Memory Communication
```python
# Agents can share knowledge through team memory
team = Team(
    agents=[agent1, agent2, agent3],
    memory=TeamMemory(shared=True),
    storage=PostgresStorage()
)

# All agents access same memory pool
team.run("Collaborate on project")
```

### 2. Persistent Knowledge

Agno provides comprehensive knowledge management through RAG (Retrieval-Augmented Generation):

#### Knowledge Base Setup
```python
from agno.knowledge import AgentKnowledge
from agno.vectordb.pgvector import PgVector2
from agno.embedder.openai import OpenAIEmbedder

knowledge = AgentKnowledge(
    vector_db=PgVector2(
        db_url="postgresql://localhost:5432/agno_db",
        collection="my_knowledge",
        embedder=OpenAIEmbedder(model="text-embedding-3-small"),
        search_type="hybrid"  # Combines semantic + keyword search
    ),
    # Data sources
    sources=[
        "docs/*.pdf",                    # Local PDFs
        "https://example.com/docs",      # Web pages
        "s3://bucket/documents/",        # S3 bucket
    ],
    # Chunking strategy
    chunk_size=512,
    chunk_overlap=50,
    # Metadata filtering
    filters={"category": "technical", "year": 2024}
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    knowledge=knowledge,
    description="Agent with knowledge base"
)
```

#### Supported Vector Databases
- **PgVector** (PostgreSQL): Production-ready, scalable
- **Pinecone**: Cloud-native, managed service
- **Qdrant**: High-performance, open-source
- **ChromaDB**: Lightweight, embedded
- **Weaviate**: GraphQL API, hybrid search

#### Document Readers
```python
from agno.document import PDFReader, WebReader, CSVReader

knowledge = AgentKnowledge(
    readers={
        "pdf": PDFReader(),
        "web": WebReader(),
        "csv": CSVReader()
    },
    sources=[
        "reports/*.pdf",
        "https://blog.example.com",
        "data/*.csv"
    ]
)
```

#### Embedding Models
- **OpenAI**: `text-embedding-3-small`, `text-embedding-3-large`
- **Mistral**: `mistral-embed`
- **Gemini**: `models/embedding-001`
- **Cohere**: `embed-english-v3.0`

#### RAG Configuration
```python
knowledge = AgentKnowledge(
    vector_db=PgVector2(
        search_type="hybrid",      # "semantic", "keyword", or "hybrid"
        similarity_threshold=0.7,   # Minimum similarity score
        top_k=5                     # Number of results to retrieve
    ),
    rerank=True,                    # Re-rank results for better relevance
    context_window=4000             # Max tokens for context
)
```

### 3. Improved Context Loading

Agno provides advanced context management features:

#### Session-Based Context
```python
from agno.storage.postgres import PostgresStorage

storage = PostgresStorage(
    db_url="postgresql://localhost:5432/agno",
    table_name="agent_sessions"
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    storage=storage,
    add_history_to_messages=True,  # Include conversation history
    num_history_responses=10        # Last 10 exchanges
)

# Context persists across runs
agent.run("Hello", session_id="user-123")
agent.run("What did I just say?", session_id="user-123")  # Remembers "Hello"
```

#### Context Compression
```python
from agno.memory import AgentMemory

memory = AgentMemory(
    compress_context=True,
    compression_ratio=0.5,         # Compress to 50% of original size
    preserve_recent=5              # Keep last 5 messages uncompressed
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    memory=memory
)
```

#### Session Summaries
```python
memory = AgentMemory(
    create_session_summary=True,   # Automatically summarize sessions
    update_session_summary=True,   # Update summary as conversation progresses
    summary_model=OpenAIChat(id="gpt-4o-mini")  # Use cheaper model for summaries
)
```

#### User Memory
```python
memory = AgentMemory(
    create_user_memories=True,     # Track user preferences and facts
    update_user_memories=True,     # Update memories over time
    user_memory_db=PostgresStorage()
)

# Agent remembers user preferences across sessions
agent.run("I prefer technical explanations", session_id="user-123")
# Later...
agent.run("Explain quantum computing", session_id="user-123")
# Uses remembered preference for technical style
```

#### Context Window Management
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    context_window=128000,          # GPT-4o's context window
    smart_truncation=True,          # Intelligently truncate old messages
    preserve_system_messages=True   # Keep system prompts
)
```

### 4. Advanced Memory Features

#### Memory Types
```python
from agno.memory import AgentMemory

memory = AgentMemory(
    # Short-term memory (conversation history)
    enable_short_term=True,
    short_term_limit=20,           # Last 20 messages
    
    # Long-term memory (persistent facts)
    enable_long_term=True,
    long_term_db=PostgresStorage(),
    
    # User preferences
    create_user_memories=True,
    
    # Session summaries
    create_session_summary=True,
    
    # Memory retrieval
    retrieve_relevant_memories=True,
    max_memories_to_retrieve=5
)
```

#### Memory Operations
```python
# Store memory
agent.memory.store(
    content="User prefers concise answers",
    memory_type="user_preference",
    user_id="user-123"
)

# Retrieve memories
memories = agent.memory.retrieve(
    query="user communication style",
    user_id="user-123",
    limit=5
)

# Update memory
agent.memory.update(
    memory_id="mem-456",
    content="Updated preference: technical and concise"
)

# Delete memory
agent.memory.delete(memory_id="mem-456")
```

### 5. Reasoning Capabilities

Agno includes built-in reasoning tools:

#### Reasoning Tool
```python
from agno.tools.reasoning import Reasoning

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[Reasoning()],
    reasoning_model=OpenAIChat(id="o1-preview"),  # Dedicated reasoning model
    show_reasoning_steps=True
)

# Agent will show step-by-step reasoning
response = agent.run("Solve complex problem X")
```

#### Chain-of-Thought
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    enable_reasoning=True,
    reasoning_config={
        "show_steps": True,
        "max_reasoning_steps": 10,
        "confidence_threshold": 0.8
    }
)
```

### 6. Tool Ecosystem

#### Pre-built Tools
```python
from agno.tools import (
    # Web & Search
    WebSearch, DuckDuckGo, GoogleSearch,
    
    # Finance
    YFinance, AlphaVantage,
    
    # Code & Development
    PythonTools, ShellTools, GitHubTools,
    
    # Documents
    PDFTools, ExcelTools, CSVTools,
    
    # Communication
    EmailTools, SlackTools,
    
    # Databases
    PostgresTools, MongoTools, SQLiteTools,
    
    # Reasoning
    Reasoning, MathTools
)

agent = Agent(
    tools=[
        WebSearch(),
        YFinance(),
        PythonTools(),
        Reasoning()
    ]
)
```

#### Custom Tools
```python
from agno.tools import Tool

def calculate_fibonacci(n: int) -> int:
    """Calculate the nth Fibonacci number.
    
    Args:
        n: The position in Fibonacci sequence
        
    Returns:
        The Fibonacci number at position n
    """
    if n <= 1:
        return n
    return calculate_fibonacci(n-1) + calculate_fibonacci(n-2)

# Create tool from function
fib_tool = Tool(
    function=calculate_fibonacci,
    name="fibonacci_calculator",
    description="Calculates Fibonacci numbers"
)

agent = Agent(
    tools=[fib_tool]
)
```

#### Tool with API Integration
```python
from agno.tools import Tool
import requests

class WeatherTool(Tool):
    def __init__(self):
        super().__init__(
            name="get_weather",
            description="Get current weather for a location"
        )
    
    def execute(self, location: str) -> dict:
        """Get weather data for location."""
        api_key = os.getenv("WEATHER_API_KEY")
        url = f"https://api.weather.com/v1/location/{location}"
        response = requests.get(url, params={"apikey": api_key})
        return response.json()

agent = Agent(tools=[WeatherTool()])
```

### 7. Streaming Support

#### Real-time Streaming
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    stream=True
)

# Stream responses token-by-token
for chunk in agent.run("Tell me a story", stream=True):
    print(chunk.content, end="", flush=True)
```

#### Server-Sent Events (SSE)
```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.post("/agent/run")
async def run_agent(message: str):
    agent = Agent(model=OpenAIChat(id="gpt-4o"), stream=True)
    
    async def event_generator():
        for chunk in agent.run(message, stream=True):
            yield f"data: {chunk.json()}\n\n"
    
    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

### 8. Multi-modal Support

#### Image Generation
```python
from agno.tools import DallE

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[DallE()],
    enable_multimodal=True
)

response = agent.run("Generate an image of a sunset over mountains")
# response.images contains generated images
```

#### Image Understanding
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o-vision"),
    enable_vision=True
)

response = agent.run(
    message="What's in this image?",
    images=["https://example.com/image.jpg"]
)
```

#### Audio Support
```python
from agno.tools import WhisperTool, TTSTool

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[WhisperTool(), TTSTool()],
    enable_audio=True
)

# Transcribe audio
response = agent.run(
    message="Transcribe this audio",
    audio=["audio.mp3"]
)

# Generate speech
response = agent.run(
    message="Convert this to speech",
    generate_audio=True
)
```

### 9. Workflow Orchestration

#### Deterministic Workflows
```python
from agno.workflow import Workflow, RunResponse

class DataProcessingWorkflow(Workflow):
    def __init__(self):
        super().__init__(
            agents=[data_fetcher, processor, analyzer],
            storage=PostgresStorage(),
            session_state=True
        )
    
    def run(self, session_id: str, message: str) -> RunResponse:
        # Step 1: Fetch data
        data = self.data_fetcher.run(message, session_id=session_id)
        self.save_state({"raw_data": data}, session_id)
        
        # Step 2: Process
        processed = self.processor.run(data, session_id=session_id)
        self.save_state({"processed_data": processed}, session_id)
        
        # Step 3: Analyze
        analysis = self.analyzer.run(processed, session_id=session_id)
        
        return RunResponse(content=analysis, session_id=session_id)

workflow = DataProcessingWorkflow()
result = workflow.run(session_id="wf-123", message="Process dataset X")
```

### 10. Monitoring and Observability

#### Metrics Collection
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    enable_metrics=True,
    metrics_config={
        "track_latency": True,
        "track_token_usage": True,
        "track_tool_calls": True,
        "track_errors": True
    }
)

# Access metrics
metrics = agent.get_metrics(session_id="user-123")
print(f"Total tokens: {metrics.total_tokens}")
print(f"Average latency: {metrics.avg_latency}ms")
```

#### Logging
```python
import logging

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    logger=logging.getLogger("agno.agent"),
    log_level=logging.DEBUG
)
```

## Configuration for Cursor IDE

### 1. Project Setup

#### Create Virtual Environment
```bash
# Create virtual environment
python3 -m venv .venv

# Activate (Unix/macOS)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Or use uv (faster)
pip install uv
uv venv .venv --python=python3.12
source .venv/bin/activate
```

#### Install Agno
```bash
# Install latest version
pip install -U agno

# Install with specific extras
pip install "agno[postgres,openai,anthropic]"

# Development installation
git clone https://github.com/agno-agi/agno.git
cd agno
pip install -e ".[dev]"
```

### 2. Cursor IDE Configuration

#### Python Interpreter Setup
1. Open Command Palette (`Cmd/Ctrl + Shift + P`)
2. Type "Python: Select Interpreter"
3. Choose your virtual environment (`.venv`)

#### Workspace Settings (`.vscode/settings.json`)
```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  
  "python.formatting.provider": "black",
  "python.formatting.blackArgs": [
    "--line-length", "100"
  ],
  
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.linting.flake8Enabled": true,
  "python.linting.mypyEnabled": true,
  "python.linting.lintOnSave": true,
  
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.rulers": [100]
  }
}
```

#### Environment Variables (`.env`)
```bash
# LLM API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GROQ_API_KEY=gsk_...

# Database
DATABASE_URL=postgresql://localhost:5432/agno_db

# AgentOS
OS_SECURITY_KEY=your-secure-token

# Vector DB
PINECONE_API_KEY=...
QDRANT_URL=http://localhost:6333

# Development
AGNO_DEBUG=true
LOG_LEVEL=DEBUG
```

#### Load Environment in Python
```python
# Use python-dotenv
from dotenv import load_dotenv
load_dotenv()

import os
openai_key = os.getenv("OPENAI_API_KEY")
```

### 3. Development Tooling

#### Install Development Tools
```bash
# Formatting
pip install black isort

# Linting
pip install pylint flake8 mypy

# Testing
pip install pytest pytest-asyncio pytest-cov

# Type checking
pip install pyright

# Environment management
pip install python-dotenv
```

#### Pre-commit Hooks (`.pre-commit-config.yaml`)
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3.12
  
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
  
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: ['--max-line-length=100']
```

### 4. Cursor AI Features Configuration

#### Create Cursor Rules

**Modern Approach (Recommended):** Create `.cursor/rules/agno-framework.mdc`:

```markdown
---
description: "Agno Framework development guidelines"
globs: ["**/*.py"]
alwaysApply: true
---

# Agno Framework Development Rules

## Project Context
- This is an Agno framework project
- Use Python 3.12+
- Follow PEP 8 style guide with 100 character line length
- Use type hints for all function signatures

## Agno-Specific Guidelines
- Always import from agno package directly: `from agno.agent import Agent`
- Use declarative configuration for agents, teams, and workflows
- Implement proper error handling for tool calls
- Add docstrings to custom tools
- Use environment variables for API keys (never hardcode)
- Prefer async operations when available
- Use sessions for stateful interactions

## Code Style
- Use Black formatter (100 char line length)
- Use isort for import organization
- Type hint all functions
- Add docstrings (Google style)

## Testing
- Write tests for custom tools
- Test with mock LLM responses
- Verify session persistence
- Check error handling

## Security
- Never commit API keys
- Use .env for secrets
- Validate tool inputs
- Sanitize user inputs
```

**Legacy Support:** Alternatively, create `.cursorrules` (root level):
```
# Agno Framework Development Rules

See .cursor/rules/*.mdc for modern, scoped rules.

## Core Guidelines
- Python 3.12+ with type hints
- Agno framework patterns (Agent, Team, Workflow)
- Use environment variables for secrets
- Follow PEP 8 with 100 char lines
```

**Note:** Use either `.cursor/rules/*.mdc` (modern) OR `.cursorrules` (legacy), not both.

#### Cursor Composer Settings
```json
{
  "cursor.composer.enabled": true,
  "cursor.composer.model": "gpt-4",
  "cursor.composer.autoSuggest": true,
  "cursor.aiExplanations": true
}
```

### 5. Example Project Structure

```
my-agno-project/
├── .env                          # Environment variables
├── .gitignore                    # Git ignore patterns
├── llms.txt                      # Documentation index for @Docs
├── agno-project.code-workspace   # Cursor workspace file
├── .venv/                        # Virtual environment
├── .cursor/
│   └── rules/                    # Cursor IDE rules (modern)
│       ├── agno-framework.mdc    # Core framework rules
│       ├── testing.mdc           # Testing guidelines
│       └── tools.mdc             # Tool development rules
├── .vscode/
│   └── settings.json             # IDE settings
├── docs/                         # Project documentation
│   ├── AGNO_ARCHITECTURE.md
│   ├── AGNO_FEATURES_AND_CONFIGURATION.md
│   └── AGNO_RESERVED_TERMS.md
├── agents/
│   ├── __init__.py
│   ├── researcher.py             # Research agent
│   ├── writer.py                 # Writer agent
│   └── analyzer.py               # Analyzer agent
├── teams/
│   ├── __init__.py
│   └── content_team.py           # Team configuration
├── workflows/
│   ├── __init__.py
│   └── data_pipeline.py          # Workflow definitions
├── tools/
│   ├── __init__.py
│   └── custom_tools.py           # Custom tool implementations
├── knowledge/
│   └── documents/                # Knowledge base documents
├── tests/
│   ├── test_agents.py
│   ├── test_teams.py
│   └── test_workflows.py
├── main.py                       # Entry point
├── requirements.txt              # Dependencies
└── README.md
```

### 6. Testing Configuration

#### pytest Configuration (`pytest.ini`)
```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --cov=agents
    --cov=teams
    --cov=workflows
    --cov-report=html
    --cov-report=term
```

#### Example Test
```python
import pytest
from agno.agent import Agent
from agno.models.openai import OpenAIChat

@pytest.fixture
def test_agent():
    return Agent(
        model=OpenAIChat(id="gpt-4o"),
        name="TestAgent"
    )

def test_agent_response(test_agent):
    response = test_agent.run("Hello")
    assert response is not None
    assert len(response.content) > 0

@pytest.mark.asyncio
async def test_agent_async(test_agent):
    response = await test_agent.arun("Hello async")
    assert response is not None
```

## Best Practices

### 1. Agent Configuration
- Use descriptive names and descriptions
- Configure appropriate context windows
- Enable only needed features
- Set reasonable timeouts

### 2. Memory Management
- Clear old sessions periodically
- Use summaries for long conversations
- Monitor memory usage in production
- Implement user memory cleanup

### 3. Knowledge Base
- Update embeddings when documents change
- Use appropriate chunk sizes (512-1024 tokens)
- Implement metadata for filtering
- Monitor vector database performance

### 4. Tool Development
- Keep tools focused and single-purpose
- Add comprehensive docstrings
- Implement error handling
- Validate inputs
- Return structured data

### 5. Production Deployment
- Use workflows for critical paths
- Implement proper authentication
- Enable monitoring and logging
- Set up error alerting
- Use database pooling
- Implement rate limiting

## Summary

The Agno framework provides comprehensive features for building production-grade AI agents:

- **Communication**: Agent-to-agent, teams, shared memory
- **Knowledge**: RAG, vector databases, document ingestion
- **Context**: Session management, compression, summaries
- **Memory**: Short-term, long-term, user preferences
- **Reasoning**: Built-in tools, chain-of-thought
- **Tools**: 100+ pre-built, easy custom integration
- **Streaming**: Real-time responses, SSE support
- **Multi-modal**: Images, audio, video
- **Workflows**: Deterministic orchestration
- **Monitoring**: Metrics, logging, observability

Proper configuration in Cursor IDE ensures optimal development experience with features like auto-completion, type checking, formatting, and AI assistance.
