# Agno Framework Setup for Cursor IDE

## Introduction

This guide provides step-by-step instructions for setting up the Agno framework in Cursor IDE, configuring your development environment, and leveraging Cursor's AI features for Agno development.

## Prerequisites

### System Requirements
- **Python**: 3.10 or higher (3.12+ recommended)
- **Cursor IDE**: Latest version from https://cursor.com
- **Git**: For version control
- **Package Manager**: pip or uv (uv recommended for faster installs)

### Recommended System Specs
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 10GB free space for Python environments
- **OS**: macOS, Linux, or Windows

## Step 1: Install Python and Package Manager

### macOS/Linux

```bash
# Check Python version
python3 --version

# Install Python 3.12 if needed (macOS with Homebrew)
brew install python@3.12

# Install uv (modern, fast package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Windows

```powershell
# Check Python version
python --version

# Install Python from https://www.python.org/downloads/
# Or use Scoop
scoop install python

# Install uv
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## Step 2: Create Project Structure

```bash
# Create project directory
mkdir my-agno-project
cd my-agno-project

# Initialize git
git init

# Create directory structure
mkdir -p agents teams workflows tools tests knowledge/documents
touch .env .gitignore README.md
```

## Step 3: Create Virtual Environment

### Using venv (Built-in)

```bash
# Create virtual environment
python3 -m venv .venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate
```

### Using uv (Recommended)

```bash
# Create virtual environment with uv
uv venv .venv --python=python3.12

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate
```

## Step 4: Install Agno Framework

```bash
# Basic installation
pip install -U agno

# Or with uv (faster)
uv pip install -U agno

# Install with extras
pip install "agno[openai,anthropic,postgres]"

# Install development tools
pip install black isort pylint mypy pytest pytest-asyncio python-dotenv
```

## Step 5: Configure Cursor IDE

### Open Project in Cursor

```bash
# Open Cursor in project directory
cursor .
```

### Select Python Interpreter

1. Open Command Palette: `Cmd/Ctrl + Shift + P`
2. Type: "Python: Select Interpreter"
3. Choose: `.venv/bin/python` (or `.venv\Scripts\python.exe` on Windows)

### Configure Workspace Settings

Create `.vscode/settings.json`:

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  
  "python.formatting.provider": "black",
  "python.formatting.blackArgs": [
    "--line-length",
    "100"
  ],
  
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.linting.flake8Enabled": false,
  "python.linting.mypyEnabled": true,
  "python.linting.lintOnSave": true,
  
  "python.analysis.typeCheckingMode": "basic",
  "python.analysis.autoImportCompletions": true,
  
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true,
    "source.fixAll": true
  },
  
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.rulers": [100],
    "editor.tabSize": 4
  },
  
  "files.exclude": {
    "**/__pycache__": true,
    "**/*.pyc": true,
    ".pytest_cache": true,
    ".mypy_cache": true,
    "*.egg-info": true
  },
  
  "cursor.composer.enabled": true,
  "cursor.aiExplanations": true,
  "cursor.autoComplete": true
}
```

## Step 6: Configure Environment Variables

Create `.env` file:

```bash
# LLM API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GROQ_API_KEY=gsk_...
GOOGLE_API_KEY=...

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/agno_db
MONGO_URI=mongodb://localhost:27017/agno_db

# Vector Database
PINECONE_API_KEY=...
PINECONE_ENVIRONMENT=...
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=...

# AgentOS Configuration
OS_SECURITY_KEY=your-secure-token-here
AGENTOS_URL=http://localhost:7777

# Development Settings
AGNO_DEBUG=true
LOG_LEVEL=DEBUG
PYTHON_ENV=development

# Model Configuration
DEFAULT_MODEL=gpt-4o
DEFAULT_TEMPERATURE=0.7
DEFAULT_MAX_TOKENS=4096
```

Create `.env.example` (for sharing):

```bash
# Copy .env.example to .env and fill in your values
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
DATABASE_URL=postgresql://localhost:5432/agno_db
OS_SECURITY_KEY=
```

## Step 7: Configure .gitignore

Create or update `.gitignore`:

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual Environment
.venv/
venv/
ENV/
env/

# Environment Variables
.env
.env.local

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/
.hypothesis/

# Type checking
.mypy_cache/
.pytype/
.pyre/

# Jupyter
.ipynb_checkpoints
*.ipynb

# Logs
*.log
logs/

# Database
*.db
*.sqlite
*.sqlite3

# OS
.DS_Store
Thumbs.db

# Agno specific
knowledge/vectors/
knowledge/cache/
sessions/
checkpoints/
```

## Step 8: Create Cursor Rules

Create `.cursorrules` file to guide Cursor's AI:

```
# Agno Framework Development Rules

## Project Context
This is an Agno framework project for building AI agents with memory, knowledge, and reasoning capabilities.

## Language and Framework
- Python 3.12+
- Agno framework (latest version)
- Use type hints for all functions
- Follow PEP 8 with 100 character line length

## Agno-Specific Patterns

### Imports
Always import from agno package directly:
```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools import Tool
from agno.knowledge import AgentKnowledge
```

### Agent Creation
Use declarative configuration:
```python
agent = Agent(
    name="AgentName",
    model=OpenAIChat(id="gpt-4o"),
    description="Clear description",
    instructions=["Guideline 1", "Guideline 2"],
    tools=[tool1, tool2],
    memory=AgentMemory(),
    knowledge=AgentKnowledge(),
    storage=PostgresStorage()
)
```

### Error Handling
Always wrap agent runs in try-except:
```python
try:
    response = agent.run(message, session_id=session_id)
except Exception as e:
    logger.error(f"Agent error: {e}")
    # Handle error appropriately
```

### Tool Development
- Add comprehensive docstrings
- Validate all inputs
- Return structured data
- Handle errors gracefully

### Security
- Never commit API keys or secrets
- Use environment variables for all credentials
- Validate user inputs in tools
- Sanitize file paths

### Testing
- Write tests for custom tools
- Mock LLM responses in tests
- Test session persistence
- Verify error handling

### Code Style
- Use Black formatter (100 char line length)
- Use isort for import organization
- Add type hints to all functions
- Write Google-style docstrings

### Best Practices
- Keep agents focused and single-purpose
- Use teams for multi-agent coordination
- Implement workflows for production systems
- Enable streaming for better UX
- Use sessions for stateful interactions
- Implement proper logging

## Common Patterns

### Basic Agent
```python
agent = Agent(
    name="Helper",
    model=OpenAIChat(id="gpt-4o"),
    instructions=["Be helpful and concise"]
)
response = agent.run("User question")
```

### Agent with Tools
```python
agent = Agent(
    tools=[WebSearch(), Calculator()],
    show_tool_calls=True
)
```

### Agent with Knowledge
```python
knowledge = AgentKnowledge(
    vector_db=PgVector2(db_url=os.getenv("DATABASE_URL")),
    sources=["docs/*.pdf"]
)
agent = Agent(knowledge=knowledge)
```

### Agent with Memory
```python
memory = AgentMemory(
    create_user_memories=True,
    create_session_summary=True
)
agent = Agent(memory=memory, storage=PostgresStorage())
```

### Team Creation
```python
team = Team(
    name="ResearchTeam",
    agents=[researcher, writer],
    mode="collaborate"
)
```

## Documentation Standards
- Add module-level docstrings
- Document all public functions
- Include usage examples
- Document environment variables needed

## Performance Considerations
- Use async operations where available
- Implement caching for expensive operations
- Monitor token usage
- Set appropriate context windows

## When to Use What
- **Agent**: Single-purpose tasks
- **Team**: Multi-agent collaboration
- **Workflow**: Deterministic, production processes
- **Memory**: Stateful conversations
- **Knowledge**: RAG and document search
- **Tools**: External capabilities
```

## Step 9: Install Cursor Extensions

1. Open Extensions panel: `Cmd/Ctrl + Shift + X`

2. Install these extensions:
   - **Python** (Microsoft) - Python language support
   - **Pylance** (Microsoft) - Fast Python language server
   - **Black Formatter** (Microsoft) - Code formatting
   - **isort** (Microsoft) - Import sorting
   - **Python Debugger** (Microsoft) - Debugging
   - **autoDocstring** (Nils Werner) - Generate docstrings
   - **Better Comments** - Enhanced comment highlighting
   - **Error Lens** - Inline error display

## Step 10: Create Project Files

### Create `requirements.txt`

```txt
# Core Framework
agno>=2.0.0

# LLM Providers
openai>=1.0.0
anthropic>=0.20.0
groq>=0.5.0

# Database
psycopg2-binary>=2.9.0
pymongo>=4.6.0
sqlalchemy>=2.0.0

# Vector Databases
pgvector>=0.2.0
pinecone-client>=3.0.0
qdrant-client>=1.7.0

# Utilities
python-dotenv>=1.0.0
pydantic>=2.0.0
requests>=2.31.0

# Development
black>=24.0.0
isort>=5.13.0
pylint>=3.0.0
mypy>=1.8.0
pytest>=8.0.0
pytest-asyncio>=0.23.0
pytest-cov>=4.1.0
```

### Create `main.py`

```python
"""
Main entry point for Agno application.
"""
import os
from dotenv import load_dotenv

from agno.agent import Agent
from agno.models.openai import OpenAIChat

# Load environment variables
load_dotenv()


def main():
    """Run the main agent."""
    # Create agent
    agent = Agent(
        name="Assistant",
        model=OpenAIChat(id="gpt-4o"),
        instructions=["Be helpful and concise"],
        markdown=True,
        debug_mode=True
    )
    
    # Run agent
    agent.print_response("Hello! Tell me about yourself.")


if __name__ == "__main__":
    main()
```

### Create `agents/researcher.py`

```python
"""
Research agent for information gathering.
"""
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools import DuckDuckGo


def create_researcher() -> Agent:
    """Create a research agent.
    
    Returns:
        Configured research agent
    """
    return Agent(
        name="Researcher",
        model=OpenAIChat(id="gpt-4o"),
        tools=[DuckDuckGo()],
        instructions=[
            "Research thoroughly using web search",
            "Cite all sources",
            "Prioritize recent and authoritative sources",
            "Provide comprehensive information"
        ],
        description="Specialized research agent",
        markdown=True
    )
```

### Create `tests/test_agents.py`

```python
"""
Tests for Agno agents.
"""
import pytest
from agents.researcher import create_researcher


def test_researcher_creation():
    """Test that researcher agent is created correctly."""
    agent = create_researcher()
    
    assert agent is not None
    assert agent.name == "Researcher"
    assert len(agent.tools) > 0


@pytest.mark.asyncio
async def test_researcher_run():
    """Test researcher agent execution."""
    agent = create_researcher()
    
    # Test with simple query
    response = agent.run("What is Python?")
    
    assert response is not None
    assert len(response.content) > 0
```

## Step 11: Configure Testing

Create `pytest.ini`:

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --strict-markers
    --cov=agents
    --cov=teams
    --cov=workflows
    --cov=tools
    --cov-report=html
    --cov-report=term
    --cov-report=xml
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
```

Create `conftest.py` for test fixtures:

```python
"""
Pytest configuration and fixtures.
"""
import pytest
import os
from dotenv import load_dotenv

# Load test environment
load_dotenv(".env.test")


@pytest.fixture(scope="session")
def test_api_key():
    """Provide test API key."""
    return os.getenv("OPENAI_API_KEY")


@pytest.fixture
def mock_agent():
    """Create a mock agent for testing."""
    from agno.agent import Agent
    from agno.models.openai import OpenAIChat
    
    return Agent(
        name="TestAgent",
        model=OpenAIChat(id="gpt-4o"),
        instructions=["Be helpful"]
    )
```

## Step 12: Use Cursor AI Features

### 1. Code Completion

Cursor provides intelligent code completion:

```python
# Type "agent = " and Cursor will suggest:
agent = Agent(
    name="",  # Cursor suggests appropriate agent names
    model=OpenAIChat(id="gpt-4o"),  # Auto-completes common patterns
    # Continue typing for more suggestions
)
```

### 2. Chat with Cursor (Cmd/Ctrl + L)

Ask Cursor AI questions:
- "How do I create an agent with memory?"
- "Show me how to implement a custom tool"
- "What's the best way to use RAG with Agno?"
- "Help me debug this error"

### 3. Inline Editing (Cmd/Ctrl + K)

Select code and press `Cmd/Ctrl + K`:
- "Add error handling"
- "Add type hints"
- "Add docstring"
- "Refactor this function"
- "Add logging"

### 4. Composer (Cmd/Ctrl + I)

Open Composer for multi-file edits:
- "Create a team of agents for research"
- "Add tests for all agents"
- "Set up a workflow for data processing"

### 5. Terminal Integration

Use Cursor's terminal with AI help:
```bash
# Type comment, get command suggestion
# Run my tests → cursor suggests: pytest tests/
# Install dependencies → cursor suggests: pip install -r requirements.txt
```

## Step 13: Running and Testing

### Run Your Agent

```bash
# Activate virtual environment
source .venv/bin/activate

# Run main script
python main.py

# Run specific agent
python -m agents.researcher
```

### Run Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov

# Run specific test file
pytest tests/test_agents.py

# Run with verbose output
pytest -v

# Run and generate HTML coverage report
pytest --cov --cov-report=html
```

### Format Code

```bash
# Format with Black
black .

# Sort imports
isort .

# Run both
black . && isort .
```

### Lint Code

```bash
# Run pylint
pylint agents/ teams/ workflows/

# Run mypy for type checking
mypy agents/ teams/ workflows/

# Run flake8
flake8 agents/ teams/ workflows/
```

## Step 14: Debugging in Cursor

### Configure Launch Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "env": {
        "PYTHONPATH": "${workspaceFolder}"
      },
      "envFile": "${workspaceFolder}/.env"
    },
    {
      "name": "Python: Main",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/main.py",
      "console": "integratedTerminal",
      "envFile": "${workspaceFolder}/.env"
    },
    {
      "name": "Python: Pytest",
      "type": "python",
      "request": "launch",
      "module": "pytest",
      "args": ["-v"],
      "console": "integratedTerminal",
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

### Set Breakpoints

1. Click left of line number to set breakpoint
2. Press `F5` to start debugging
3. Use debug toolbar to step through code

## Quick Reference Commands

### Environment Management
```bash
source .venv/bin/activate          # Activate venv (Unix/macOS)
.venv\Scripts\activate             # Activate venv (Windows)
deactivate                         # Deactivate venv
pip list                           # List installed packages
pip freeze > requirements.txt      # Save dependencies
```

### Agno Commands
```bash
pip install -U agno                # Update Agno
agentos run --port 7777           # Run AgentOS locally
agentos deploy                     # Deploy to cloud
```

### Git Commands
```bash
git add .                          # Stage changes
git commit -m "message"            # Commit changes
git push                           # Push to remote
```

### Testing Commands
```bash
pytest                             # Run all tests
pytest -v                          # Verbose output
pytest --cov                       # With coverage
pytest -k "test_name"             # Run specific test
pytest --pdb                       # Debug on failure
```

## Troubleshooting

### Issue: Python interpreter not found
**Solution**: 
1. Ensure virtual environment is activated
2. Select correct interpreter in Cursor
3. Reload Cursor window (`Cmd/Ctrl + Shift + P` → "Reload Window")

### Issue: Import errors
**Solution**:
```bash
# Ensure packages are installed
pip install -r requirements.txt

# Check PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:${PWD}"
```

### Issue: API key errors
**Solution**:
1. Check `.env` file exists and has correct keys
2. Verify keys are not expired
3. Ensure `python-dotenv` is installed
4. Load env with `load_dotenv()` at start of script

### Issue: Database connection errors
**Solution**:
1. Verify database is running
2. Check DATABASE_URL format
3. Test connection separately
4. Check firewall/network settings

## Additional Resources

- **Agno Documentation**: https://docs.agno.com
- **Cursor Documentation**: https://cursor.com/docs
- **Agno Examples**: https://github.com/agno-agi/agno/tree/main/cookbook
- **Cursor Community**: https://forum.cursor.com
- **Agno Discord**: https://discord.gg/agno

## Next Steps

1. ✅ Complete this setup guide
2. 📖 Read Agno Architecture documentation
3. 🧪 Run example agents
4. 🛠️ Create your first custom tool
5. 👥 Build a team of agents
6. 🔄 Implement a workflow
7. 📚 Add knowledge base with RAG
8. 🚀 Deploy to production with AgentOS

Happy coding with Agno and Cursor! 🎉
