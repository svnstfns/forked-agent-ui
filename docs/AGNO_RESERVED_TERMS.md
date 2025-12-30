# Agno Framework Reserved Terms and Concepts

## Introduction

This document provides a comprehensive reference of all reserved terms, concepts, and architectural patterns used in the Agno framework. Understanding these terms is essential for effective development with Agno.

## Core Entities

### Agent

**Definition**: The foundational, autonomous unit in Agno that executes tasks using LLM models, tools, memory, and knowledge.

**Key Characteristics:**
- Independent execution capability
- Equipped with specific tools and capabilities
- Maintains conversation context through sessions
- Can access persistent knowledge bases
- Generates responses with optional reasoning steps

**Usage:**
```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agent = Agent(
    name="MyAgent",                    # Agent identifier
    model=OpenAIChat(id="gpt-4o"),    # LLM model
    description="Helpful assistant",   # Agent purpose
    instructions=["Be concise"],       # Behavioral guidelines
    tools=[tool1, tool2],              # Available tools
    memory=AgentMemory(),              # Memory system
    knowledge=AgentKnowledge(),        # Knowledge base
    storage=AgentStorage()             # Session storage
)
```

**Attributes:**
- `name` (str): Human-readable agent identifier
- `agent_id` (str): Unique system identifier
- `model` (Model): LLM configuration
- `description` (str): Agent's purpose and capabilities
- `instructions` (list[str]): Behavioral guidelines
- `tools` (list[Tool]): Available tools/functions
- `memory` (AgentMemory): Memory management
- `knowledge` (AgentKnowledge): Knowledge base
- `storage` (Storage): Session persistence
- `reasoning` (bool): Enable reasoning capabilities
- `markdown` (bool): Format responses in markdown
- `debug_mode` (bool): Enable detailed logging

**Methods:**
- `run(message, session_id=None, stream=False)`: Execute agent with message
- `arun()`: Async execution
- `print_response()`: Execute and print formatted response
- `get_session()`: Retrieve session history
- `clear_memory()`: Clear agent memory

---

### Team

**Definition**: A collection of agents coordinated under a team leader to accomplish larger or more complex tasks through collaboration.

**Team Modes:**
1. **Route**: Leader assigns tasks to specialized agents
2. **Coordinate**: Agents work sequentially, passing results
3. **Collaborate**: Agents discuss and reach consensus

**Usage:**
```python
from agno.team import Team

team = Team(
    name="ResearchTeam",
    agents=[researcher, writer, editor],
    mode="collaborate",              # Coordination mode
    leader=team_leader,              # Optional team leader
    description="Content creation",
    memory=TeamMemory(shared=True),  # Shared memory
    storage=TeamStorage(),           # Team session storage
    instructions=["Quality first"]
)
```

**Attributes:**
- `name` (str): Team identifier
- `team_id` (str): Unique system identifier
- `agents` (list[Agent]): Team member agents
- `mode` (str): "route" | "coordinate" | "collaborate"
- `leader` (Agent): Team leader agent
- `description` (str): Team purpose
- `memory` (TeamMemory): Shared memory system
- `storage` (Storage): Team session storage
- `show_progress` (bool): Display team coordination progress

**Methods:**
- `run(message, session_id=None)`: Execute team task
- `arun()`: Async team execution
- `add_agent(agent)`: Add agent to team
- `remove_agent(agent_id)`: Remove agent from team
- `get_team_session()`: Get team session history

---

### Workflow

**Definition**: A deterministic, stateful orchestration of agents and teams for production-grade, multi-step processes with persistent state management.

**Key Characteristics:**
- Deterministic execution (same inputs → same outputs)
- State persistence across steps
- Intermediate result caching
- Standard Python control flow
- Batch and streaming support
- Full auditability

**Usage:**
```python
from agno.workflow import Workflow, RunResponse

class DataPipeline(Workflow):
    def __init__(self):
        super().__init__(
            agents=[fetcher, processor, analyzer],
            storage=WorkflowStorage(),
            session_state=True,
            cache_intermediate=True
        )
    
    def run(self, session_id: str, message: str) -> RunResponse:
        # Step 1: Fetch
        data = self.fetcher.run(message, session_id=session_id)
        self.save_state({"data": data}, session_id)
        
        # Step 2: Process
        processed = self.processor.run(data, session_id=session_id)
        self.save_state({"processed": processed}, session_id)
        
        # Step 3: Analyze
        result = self.analyzer.run(processed, session_id=session_id)
        
        return RunResponse(content=result, session_id=session_id)
```

**Attributes:**
- `name` (str): Workflow identifier
- `workflow_id` (str): Unique system identifier
- `agents` (list[Agent]): Agents in workflow
- `storage` (Storage): Workflow state storage
- `session_state` (bool): Enable session persistence
- `cache_intermediate` (bool): Cache intermediate results

**Methods:**
- `run(session_id, message)`: Execute workflow
- `save_state(state, session_id)`: Save workflow state
- `load_state(session_id)`: Load saved state
- `get_workflow_history()`: Get execution history

---

## Memory System

### Memory

**Definition**: The system for managing both short-term (conversational) and long-term (persistent) information across agent interactions.

**Memory Types:**

#### 1. Short-term Memory
Tracks recent conversation history within a session.

```python
memory = AgentMemory(
    enable_short_term=True,
    short_term_limit=20  # Last 20 messages
)
```

#### 2. Long-term Memory
Persists facts, preferences, and knowledge across sessions.

```python
memory = AgentMemory(
    enable_long_term=True,
    long_term_db=PostgresStorage(),
    retrieve_relevant_memories=True
)
```

#### 3. User Memory
Tracks user-specific preferences and information.

```python
memory = AgentMemory(
    create_user_memories=True,
    update_user_memories=True,
    user_memory_db=PostgresStorage()
)
```

#### 4. Session Summary
Compressed summary of session history.

```python
memory = AgentMemory(
    create_session_summary=True,
    update_session_summary=True,
    summary_model=OpenAIChat(id="gpt-4o-mini")
)
```

**Attributes:**
- `db` (Storage): Memory database
- `create_user_memories` (bool): Track user facts
- `create_session_summary` (bool): Create summaries
- `update_user_memories` (bool): Update stored memories
- `update_session_summary` (bool): Update summaries
- `compress_context` (bool): Compress old messages

**Methods:**
- `store(content, memory_type, user_id)`: Store new memory
- `retrieve(query, user_id, limit)`: Retrieve relevant memories
- `update(memory_id, content)`: Update existing memory
- `delete(memory_id)`: Delete memory
- `clear(user_id)`: Clear user memories

---

### Session

**Definition**: A persistent conversation context identified by a unique session ID, maintaining state across multiple interactions.

**Session Types:**

#### AgentSession
```python
from agno.storage import AgentSession

session = AgentSession(
    session_id="user-123-session",
    agent_id="agent-456",
    user_id="user-123",
    storage=PostgresStorage()
)
```

#### TeamSession
```python
from agno.storage import TeamSession

team_session = TeamSession(
    session_id="team-session-789",
    team_id="team-abc",
    storage=PostgresStorage()
)
```

#### WorkflowSession
```python
from agno.storage import WorkflowSession

workflow_session = WorkflowSession(
    session_id="workflow-xyz-123",
    workflow_id="workflow-xyz",
    state_persistence=True
)
```

**Attributes:**
- `session_id` (str): Unique session identifier
- `session_name` (str): Human-readable name
- `created_at` (int): Creation timestamp
- `updated_at` (int): Last update timestamp
- `user_id` (str): Associated user
- `messages` (list): Conversation history
- `metadata` (dict): Additional session data

**Methods:**
- `add_message(message)`: Add message to session
- `get_messages(limit)`: Retrieve messages
- `clear()`: Clear session history
- `delete()`: Delete session

---

## Knowledge System

### Knowledge

**Definition**: Structured information repositories that agents can search and retrieve from, typically implemented through RAG (Retrieval-Augmented Generation).

**Usage:**
```python
from agno.knowledge import AgentKnowledge
from agno.vectordb.pgvector import PgVector2

knowledge = AgentKnowledge(
    vector_db=PgVector2(
        db_url="postgresql://localhost:5432/db",
        collection="my_docs",
        search_type="hybrid"
    ),
    sources=[
        "docs/*.pdf",
        "https://example.com/docs",
        "s3://bucket/files/"
    ],
    readers={
        "pdf": PDFReader(),
        "web": WebReader(),
        "csv": CSVReader()
    },
    chunk_size=512,
    chunk_overlap=50
)
```

**Components:**

#### Vector Database
Stores and retrieves document embeddings:
- **PgVector**: PostgreSQL-based, production-ready
- **Pinecone**: Cloud-native managed service
- **Qdrant**: High-performance, open-source
- **ChromaDB**: Lightweight, embedded
- **Weaviate**: GraphQL API, hybrid search

#### Document Readers
Extract and parse content:
- **PDFReader**: PDF document parsing
- **WebReader**: Web page scraping
- **CSVReader**: CSV file parsing
- **JSONReader**: JSON document parsing
- **TextReader**: Plain text files

#### Embedder
Converts text to vector embeddings:
- **OpenAIEmbedder**: OpenAI embedding models
- **MistralEmbedder**: Mistral embeddings
- **GeminiEmbedder**: Google Gemini embeddings
- **CohereEmbedder**: Cohere embeddings

**Attributes:**
- `vector_db` (VectorDB): Vector database instance
- `sources` (list[str]): Data sources
- `readers` (dict): Document readers by type
- `embedder` (Embedder): Embedding model
- `chunk_size` (int): Document chunk size
- `chunk_overlap` (int): Overlap between chunks
- `search_type` (str): "semantic" | "keyword" | "hybrid"

**Methods:**
- `load()`: Load documents into knowledge base
- `search(query, top_k)`: Search for relevant documents
- `add_document(doc)`: Add single document
- `delete_document(doc_id)`: Remove document
- `update_document(doc_id, content)`: Update document

---

### Guidelines

**Definition**: Instructions and behavioral rules that guide agent responses and decision-making. Also referred to as "instructions" or "system prompts."

**Usage:**
```python
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "Be concise and technical",
        "Always cite sources",
        "Prioritize accuracy over speed",
        "Use markdown formatting",
        "Ask clarifying questions when uncertain"
    ],
    # Alternative: Single instruction string
    instruction="You are a helpful technical assistant."
)
```

**Best Practices:**
- Keep instructions clear and specific
- Order by priority (most important first)
- Avoid conflicting instructions
- Test with various inputs
- Update based on agent behavior

---

## Tools and Functions

### Tool

**Definition**: A function or capability that agents can invoke to perform actions beyond text generation.

**Built-in Tool Categories:**
- **Web & Search**: WebSearch, DuckDuckGo, GoogleSearch
- **Finance**: YFinance, AlphaVantage
- **Code**: PythonTools, ShellTools, GitHubTools
- **Documents**: PDFTools, ExcelTools, CSVTools
- **Communication**: EmailTools, SlackTools
- **Databases**: PostgresTools, MongoTools
- **Reasoning**: Reasoning, MathTools

**Custom Tool Definition:**
```python
from agno.tools import Tool

def custom_calculator(operation: str, a: float, b: float) -> float:
    """Perform arithmetic operations.
    
    Args:
        operation: Operation to perform (add, subtract, multiply, divide)
        a: First number
        b: Second number
    
    Returns:
        Result of the operation
    """
    if operation == "add":
        return a + b
    elif operation == "subtract":
        return a - b
    elif operation == "multiply":
        return a * b
    elif operation == "divide":
        return a / b if b != 0 else float('inf')
    else:
        raise ValueError(f"Unknown operation: {operation}")

# Create tool
calc_tool = Tool(
    function=custom_calculator,
    name="calculator",
    description="Performs basic arithmetic"
)

agent = Agent(tools=[calc_tool])
```

**Tool Attributes:**
- `name` (str): Tool identifier
- `description` (str): What the tool does
- `function` (callable): Actual implementation
- `parameters` (dict): Expected parameters
- `required` (list[str]): Required parameters

---

### Toolkit

**Definition**: A collection of related tools grouped together for a specific domain or functionality.

**Usage:**
```python
from agno.toolkit import Toolkit

class DataAnalysisToolkit(Toolkit):
    def __init__(self):
        super().__init__(name="data_analysis")
        
        self.register(self.load_data)
        self.register(self.analyze_data)
        self.register(self.visualize_data)
    
    def load_data(self, file_path: str) -> dict:
        """Load data from file."""
        # Implementation
        pass
    
    def analyze_data(self, data: dict) -> dict:
        """Analyze dataset."""
        # Implementation
        pass
    
    def visualize_data(self, data: dict) -> str:
        """Create visualizations."""
        # Implementation
        pass

agent = Agent(tools=[DataAnalysisToolkit()])
```

---

## Model Configuration

### Model

**Definition**: Configuration for the LLM (Large Language Model) that powers agent reasoning and response generation.

**Usage:**
```python
from agno.models.openai import OpenAIChat
from agno.models.anthropic import Claude
from agno.models.groq import Groq
from agno.models.ollama import Ollama

# OpenAI
model = OpenAIChat(
    id="gpt-4o",                    # Model identifier
    temperature=0.7,                # Randomness (0-2)
    max_tokens=4096,                # Max response length
    top_p=0.9,                      # Nucleus sampling
    frequency_penalty=0.0,          # Repetition penalty
    presence_penalty=0.0            # Topic diversity
)

# Anthropic
model = Claude(id="claude-3-sonnet-20240229")

# Groq
model = Groq(id="llama-3.1-70b-versatile")

# Ollama (local)
model = Ollama(id="llama3:latest")
```

**Attributes:**
- `id` (str): Model identifier
- `provider` (str): Provider name
- `temperature` (float): Response randomness
- `max_tokens` (int): Maximum response length
- `top_p` (float): Nucleus sampling parameter
- `frequency_penalty` (float): Repetition penalty
- `presence_penalty` (float): Topic diversity penalty

---

## Storage and Persistence

### Storage

**Definition**: Backend systems for persisting agent sessions, memory, knowledge, and workflow state.

**Supported Storage Types:**

#### SQLite (Development)
```python
from agno.storage.sqlite import SqliteStorage

storage = SqliteStorage(
    db_file="agno.db",
    table_name="sessions"
)
```

#### PostgreSQL (Production)
```python
from agno.storage.postgres import PostgresStorage

storage = PostgresStorage(
    db_url="postgresql://localhost:5432/agno",
    table_name="agent_sessions",
    schema="public"
)
```

#### MongoDB
```python
from agno.storage.mongo import MongoStorage

storage = MongoStorage(
    db_url="mongodb://localhost:27017",
    database="agno",
    collection="sessions"
)
```

**Methods:**
- `create(session)`: Create new session
- `read(session_id)`: Retrieve session
- `update(session_id, data)`: Update session
- `delete(session_id)`: Delete session
- `list(user_id)`: List user sessions

---

## Event System

### RunEvent

**Definition**: Event types emitted during agent, team, or workflow execution for real-time monitoring and streaming.

**Event Types:**

#### Agent Events
- `RunStarted`: Agent begins processing
- `RunContent`: Agent generates content (streaming)
- `RunCompleted`: Agent finishes successfully
- `RunError`: Agent encounters error
- `RunCancelled`: Run cancelled by user

#### Tool Events
- `ToolCallStarted`: Tool execution begins
- `ToolCallCompleted`: Tool execution finishes

#### Memory Events
- `MemoryUpdateStarted`: Memory update begins
- `MemoryUpdateCompleted`: Memory update finishes

#### Reasoning Events
- `ReasoningStarted`: Reasoning process begins
- `ReasoningStep`: Individual reasoning step
- `ReasoningCompleted`: Reasoning finished

#### Team Events
- `TeamRunStarted`: Team task begins
- `TeamRunContent`: Team generates content
- `TeamRunCompleted`: Team task finishes
- `TeamRunError`: Team encounters error
- `TeamToolCallStarted`: Team tool execution begins
- `TeamToolCallCompleted`: Team tool execution finishes
- `TeamReasoningStarted`: Team reasoning begins
- `TeamReasoningStep`: Team reasoning step
- `TeamReasoningCompleted`: Team reasoning finishes
- `TeamMemoryUpdateStarted`: Team memory update begins
- `TeamMemoryUpdateCompleted`: Team memory update finishes

**Usage:**
```python
for event in agent.run(message, stream=True):
    if event.event == RunEvent.RunContent:
        print(event.content, end="")
    elif event.event == RunEvent.ToolCallStarted:
        print(f"Calling tool: {event.tool.name}")
    elif event.event == RunEvent.RunCompleted:
        print("Done!")
```

---

## Additional Concepts

### Context

**Definition**: The complete set of information available to an agent during execution, including conversation history, system prompts, tool outputs, and retrieved knowledge.

**Context Components:**
- System messages (instructions)
- Conversation history
- Tool call results
- Retrieved knowledge (RAG)
- User memories
- Session summaries

**Context Management:**
```python
agent = Agent(
    context_window=128000,          # Maximum context size
    smart_truncation=True,          # Intelligently remove old messages
    preserve_system_messages=True,  # Keep instructions
    add_history_to_messages=True,   # Include conversation history
    num_history_responses=10        # Number of past exchanges
)
```

---

### RAG (Retrieval-Augmented Generation)

**Definition**: A technique that combines information retrieval from a knowledge base with LLM generation to produce grounded, fact-based responses.

**RAG Process:**
1. User query received
2. Query embedded into vector
3. Relevant documents retrieved from vector DB
4. Retrieved context added to LLM prompt
5. LLM generates response using context
6. Response returned to user

**Configuration:**
```python
knowledge = AgentKnowledge(
    vector_db=PgVector2(search_type="hybrid"),
    top_k=5,                    # Retrieve top 5 results
    similarity_threshold=0.7,    # Minimum similarity score
    rerank=True                  # Re-rank results
)
```

---

### Reasoning

**Definition**: The process of breaking down complex problems into logical steps, showing the agent's thought process.

**Types:**
1. **Chain-of-Thought**: Step-by-step reasoning
2. **Tree-of-Thought**: Multiple reasoning paths explored
3. **Reflection**: Agent reviews and refines its reasoning

**Configuration:**
```python
agent = Agent(
    reasoning_model=OpenAIChat(id="o1-preview"),  # Dedicated reasoning model
    show_reasoning_steps=True,
    enable_reasoning=True,
    reasoning_config={
        "max_steps": 10,
        "confidence_threshold": 0.8
    }
)
```

---

### Streaming

**Definition**: Real-time, token-by-token delivery of agent responses instead of waiting for complete generation.

**Benefits:**
- Faster perceived response time
- Better user experience
- Early error detection
- Progress visibility

**Implementation:**
```python
# Streaming enabled
for chunk in agent.run("Question", stream=True):
    print(chunk.content, end="", flush=True)
```

---

## Summary Table

| Term | Category | Description |
|------|----------|-------------|
| **Agent** | Core Entity | Autonomous unit with tools, memory, knowledge |
| **Team** | Core Entity | Coordinated group of agents |
| **Workflow** | Core Entity | Deterministic orchestration |
| **Memory** | Storage | Short and long-term information persistence |
| **Session** | Storage | Conversation context with unique ID |
| **Knowledge** | Data | RAG-based document repositories |
| **Guidelines** | Configuration | Behavioral instructions for agents |
| **Tool** | Function | Executable capability for agents |
| **Toolkit** | Function | Collection of related tools |
| **Model** | Configuration | LLM configuration and settings |
| **Storage** | Persistence | Backend database for sessions/memory |
| **RunEvent** | System | Event types during execution |
| **Context** | Runtime | Complete information available to agent |
| **RAG** | Technique | Retrieval-augmented generation |
| **Reasoning** | Capability | Logical problem breakdown |
| **Streaming** | Delivery | Real-time response delivery |

---

## Reserved Keywords

When developing with Agno, avoid using these as variable names:

- `agent`, `agents`
- `team`, `teams`
- `workflow`, `workflows`
- `memory`, `memories`
- `session`, `sessions`
- `knowledge`, `knowledge_base`
- `tool`, `tools`, `toolkit`, `toolkits`
- `model`, `models`
- `storage`, `db`
- `instruction`, `instructions`, `guidelines`
- `context`, `prompt`
- `event`, `events`
- `run`, `response`
- `stream`, `streaming`
- `reasoning`
- `embedder`, `embedding`
- `vector_db`, `vectordb`

---

## Additional Resources

- **Official Documentation**: https://docs.agno.com
- **API Reference**: https://docs.agno.com/reference
- **Examples**: https://github.com/agno-agi/agno/tree/main/cookbook
- **Community**: https://discord.gg/agno
