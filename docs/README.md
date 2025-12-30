# Agno Framework Documentation

This directory contains comprehensive documentation for the Agno framework, covering architecture, features, configuration, and development setup.

## 📚 Documentation Files

### 1. [Architecture Overview](./AGNO_ARCHITECTURE.md)
**Comprehensive guide to Agno's architecture and design**

Topics covered:
- Core architecture principles and the 5 levels of agentic systems
- Framework layers and components
- Agent, Team, and Workflow structures
- Memory and knowledge systems
- AgentOS runtime
- Event-driven architecture
- Data flow and integration points
- Performance characteristics
- Deployment models
- Best practices

**Best for**: Understanding how Agno works internally, architectural decisions, and system design.

---

### 2. [Features and Configuration Guide](./AGNO_FEATURES_AND_CONFIGURATION.md)
**Detailed guide to all Agno features and how to configure them**

Topics covered:
- Agent communication patterns
- Persistent knowledge and RAG implementation
- Improved context loading
- Advanced memory features
- Reasoning capabilities
- Tool ecosystem (100+ tools)
- Streaming support
- Multi-modal capabilities (images, audio, video)
- Workflow orchestration
- Monitoring and observability
- Cursor IDE configuration
- Development tooling setup

**Best for**: Learning what Agno can do and how to configure specific features for your use case.

---

### 3. [Reserved Terms and Concepts](./AGNO_RESERVED_TERMS.md)
**Complete reference of Agno terminology and reserved terms**

Topics covered:
- Core entities: Agent, Team, Workflow
- Memory system types and management
- Session management (Agent, Team, Workflow)
- Knowledge base components
- Guidelines and instructions
- Tools and toolkits
- Model configuration
- Storage and persistence
- Event system (RunEvent types)
- Additional concepts: Context, RAG, Reasoning, Streaming
- Reserved keywords to avoid

**Best for**: Quick reference when coding, understanding terminology, and avoiding naming conflicts.

---

### 4. [Cursor IDE Setup Guide](./AGNO_CURSOR_SETUP.md)
**Step-by-step setup guide for Agno development in Cursor IDE**

Topics covered:
- Prerequisites and system requirements
- Python and package manager installation
- Project structure creation
- Virtual environment setup
- Agno framework installation
- Cursor IDE configuration
- Environment variables setup
- Git configuration
- **Cursor workspace files (.code-workspace)**
- **Modern .cursor/rules/*.mdc files**
- **llms.txt for @Docs integration**
- **Using Cursor AI features (Chat, Composer, @Docs)**
- Extension recommendations
- Project files and templates
- Testing configuration
- Debugging setup
- Troubleshooting common issues

**Best for**: Getting started with Agno development, setting up your IDE, and leveraging Cursor's AI features.

---

## 🖱️ Cursor IDE Integration

This documentation is optimized for **Cursor IDE**, featuring:

### Cursor-Specific Features
- **`.code-workspace` file**: Multi-root workspace configuration with AI indexing
- **`.cursor/rules/*.mdc` files**: Modern, scoped development rules
- **`llms.txt`**: Project documentation index for @Docs integration
- **AI-aware settings**: Configured for optimal Cursor AI assistance

### Key Cursor Terminology
- **Workspace**: Your project environment with folders, settings, and AI context
- **@Docs**: Cursor directive to reference documentation (use with llms.txt)
- **Rules**: Guidelines in `.cursor/rules/*.mdc` that inform AI suggestions
- **Indexing**: Cursor's process of understanding your codebase for AI features
- **Composer**: Multi-file AI editing tool (`Cmd/Ctrl + I`)
- **Chat**: AI assistant with project context (`Cmd/Ctrl + L`)

### Using This Documentation in Cursor
1. Open workspace file: `cursor agno-project.code-workspace`
2. In Cursor Chat, type: `@Docs llms.txt`
3. Ask questions: `@Docs how do I create an agent with RAG?`
4. Let rules guide your coding with auto-suggestions

---

## 🚀 Quick Start

### For New Users
1. Start with [Cursor IDE Setup Guide](./AGNO_CURSOR_SETUP.md) to set up your environment
2. Read [Architecture Overview](./AGNO_ARCHITECTURE.md) to understand the framework
3. Review [Reserved Terms](./AGNO_RESERVED_TERMS.md) for quick reference
4. Dive into [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md) to build your application

### For Experienced Developers
1. Skim [Architecture Overview](./AGNO_ARCHITECTURE.md) for high-level understanding
2. Use [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md) as a cookbook
3. Keep [Reserved Terms](./AGNO_RESERVED_TERMS.md) open for quick reference
4. Refer to [Cursor Setup](./AGNO_CURSOR_SETUP.md) for IDE optimization

### For Architects
1. Deep dive into [Architecture Overview](./AGNO_ARCHITECTURE.md)
2. Review [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md) for capabilities
3. Plan implementation using best practices from all guides

---

## 📖 Documentation Overview

### Architecture Documentation
The architecture documentation explains the **5 levels of agentic systems** in Agno:

1. **Level 1: Tools + Instructions** - Basic agents with tool integration
2. **Level 2: + Knowledge + Storage** - Persistent knowledge management
3. **Level 3: + Memory + Reasoning** - Long-term memory and reasoning
4. **Level 4: + Teams** - Multi-agent coordination
5. **Level 5: + Workflows** - Deterministic orchestration

### Key Framework Layers
- **Core Agent Framework**: Orchestration and lifecycle management
- **Model Integration Layer**: 23+ LLM providers
- **Tools Ecosystem**: 100+ pre-built toolkits
- **Knowledge & Search**: RAG and vector database integration
- **Execution & State Persistence**: Session and memory management

### Performance Highlights
- ⚡ **3 microseconds** agent instantiation
- 💾 **~5KB** memory footprint per agent
- 🚀 **500x faster** than comparable frameworks
- 📊 Real-time token-by-token streaming

---

## 🎯 Common Use Cases

### Building a Simple Agent
```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    instructions=["Be helpful and concise"]
)

response = agent.run("Hello!")
```
→ See [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md#agent-configuration)

### Adding Knowledge (RAG)
```python
from agno.knowledge import AgentKnowledge
from agno.vectordb.pgvector import PgVector2

knowledge = AgentKnowledge(
    vector_db=PgVector2(db_url="postgresql://localhost/db"),
    sources=["docs/*.pdf"]
)

agent = Agent(knowledge=knowledge)
```
→ See [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md#2-persistent-knowledge)

### Creating a Team
```python
from agno.team import Team

team = Team(
    name="ResearchTeam",
    agents=[researcher, writer, editor],
    mode="collaborate"
)

result = team.run("Create an article on AI")
```
→ See [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md#1-agent-communication)

### Building a Workflow
```python
from agno.workflow import Workflow

class DataPipeline(Workflow):
    def run(self, session_id, message):
        data = self.fetcher.run(message)
        processed = self.processor.run(data)
        return self.analyzer.run(processed)
```
→ See [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md#9-workflow-orchestration)

---

## 🔑 Key Concepts Quick Reference

| Concept | Description | Documentation |
|---------|-------------|---------------|
| **Agent** | Autonomous AI entity with tools and memory | [Reserved Terms](./AGNO_RESERVED_TERMS.md#agent) |
| **Team** | Coordinated group of agents | [Reserved Terms](./AGNO_RESERVED_TERMS.md#team) |
| **Workflow** | Deterministic orchestration | [Reserved Terms](./AGNO_RESERVED_TERMS.md#workflow) |
| **Memory** | Short and long-term persistence | [Reserved Terms](./AGNO_RESERVED_TERMS.md#memory) |
| **Session** | Conversation context | [Reserved Terms](./AGNO_RESERVED_TERMS.md#session) |
| **Knowledge** | RAG-based document repositories | [Reserved Terms](./AGNO_RESERVED_TERMS.md#knowledge) |
| **Tool** | Executable capability | [Reserved Terms](./AGNO_RESERVED_TERMS.md#tool) |
| **RAG** | Retrieval-augmented generation | [Reserved Terms](./AGNO_RESERVED_TERMS.md#rag-retrieval-augmented-generation) |

---

## 🛠️ Development Workflow

### 1. Setup Phase
- Follow [Cursor Setup Guide](./AGNO_CURSOR_SETUP.md)
- Configure environment variables
- Install dependencies

### 2. Learning Phase
- Read [Architecture Overview](./AGNO_ARCHITECTURE.md)
- Understand [Reserved Terms](./AGNO_RESERVED_TERMS.md)
- Explore example projects

### 3. Development Phase
- Use [Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md) as cookbook
- Implement agents, teams, or workflows
- Add tools, memory, and knowledge

### 4. Testing Phase
- Write unit tests for custom tools
- Test agents with mock responses
- Verify session persistence

### 5. Deployment Phase
- Use AgentOS for production
- Enable authentication
- Set up monitoring

---

## 📝 Code Examples

All documentation includes practical code examples. Here are some file locations:

- **Basic Agent Examples**: Features Guide, Section "Core Features"
- **Team Examples**: Features Guide, Section "Agent Communication"
- **Workflow Examples**: Features Guide, Section "Workflow Orchestration"
- **Custom Tool Examples**: Features Guide, Section "Tool Ecosystem"
- **RAG Setup Examples**: Features Guide, Section "Persistent Knowledge"
- **Memory Configuration**: Features Guide, Section "Advanced Memory Features"
- **Testing Examples**: Cursor Setup Guide, Section "Create Project Files"

---

## 🔗 External Resources

### Official Links
- **Official Documentation**: https://docs.agno.com
- **GitHub Repository**: https://github.com/agno-agi/agno
- **PyPI Package**: https://pypi.org/project/agno/
- **Community Examples**: https://github.com/agno-agi/agno/tree/main/cookbook
- **Discord Community**: https://discord.gg/agno

### Additional Learning
- **DeepWiki**: https://deepwiki.com/agno-agi/agno-docs
- **Zread Deep Dives**: https://zread.ai/agno-agi/agno
- **YouTube Tutorials**: Search for "Agno framework tutorials"

---

## 🤝 Contributing

These documentation files are part of the forked-agent-ui project. Contributions and improvements are welcome!

### How to Contribute
1. Fork the repository
2. Make your changes to documentation
3. Test code examples
4. Submit a pull request

### Documentation Guidelines
- Keep examples practical and tested
- Use clear, concise language
- Include code snippets where helpful
- Link between related concepts
- Update table of contents when adding sections

---

## 📄 License

This documentation is part of the forked-agent-ui project and follows the same license.

---

## ❓ Getting Help

### For Documentation Issues
- Open an issue in the repository
- Tag with "documentation" label

### For Agno Framework Questions
- Check official documentation at https://docs.agno.com
- Ask in Discord community
- Search GitHub issues

### For Cursor IDE Questions
- Visit https://cursor.com/docs
- Check Cursor community forum

---

## 📅 Last Updated

Documentation created: December 30, 2024

**Note**: These documents are based on Agno v2.x. For v1.x documentation, check the v1 branch or legacy documentation.

---

## 🎓 Learning Path

### Beginner Path (1-2 days)
1. ✅ Setup environment ([Cursor Setup](./AGNO_CURSOR_SETUP.md))
2. 📖 Read architecture basics ([Architecture](./AGNO_ARCHITECTURE.md) - first 3 sections)
3. 🎯 Build your first agent ([Features Guide](./AGNO_FEATURES_AND_CONFIGURATION.md) - Basic Agent)
4. 🧪 Run and test your agent
5. 📚 Add tools to your agent

### Intermediate Path (3-5 days)
1. ✅ Complete beginner path
2. 🧠 Add memory and sessions
3. 📊 Implement knowledge base with RAG
4. 👥 Create a team of agents
5. 🔧 Build custom tools
6. 🧪 Write comprehensive tests

### Advanced Path (1-2 weeks)
1. ✅ Complete intermediate path
2. 🔄 Implement production workflows
3. 📈 Add monitoring and metrics
4. 🚀 Deploy with AgentOS
5. 🔐 Configure authentication
6. ⚡ Optimize performance
7. 🏗️ Design multi-agent systems

---

Happy building with Agno! 🚀
