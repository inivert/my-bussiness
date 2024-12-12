# Cogito Documentation

## Overview
CogitoForge is an intelligent multi-agent development environment that orchestrates AI collaborators to transform ideas into code. By leveraging multiple specialized AI agents, CogitoForge streamlines the entire development process from project planning through code generation and review, creating a seamless workflow for turning concepts into reality.

## Features
- Multi-agent orchestration
- Dynamic API key management
- File system operations
- Task division and coordination
- Code generation and review
- Project structure creation

## Requirements

### System Requirements
- Python 3.9+
- pip package manager
- Internet connection
- Valid API keys for chosen AI services

### Dependencies
```
anthropic>=0.3.0
python-dotenv>=1.0.0
rich>=13.0.0
pathlib>=1.0.1
typer>=0.9.0
pydantic>=2.0.0
```

## Project Structure
```
cogito/
├── .env                    # API keys and configuration
├── README.md              # Project documentation
├── requirements.txt       # Project dependencies
├── setup.py              # Package setup file
├── src/
│   ├── __init__.py
│   ├── main.py           # Entry point
│   ├── config/
│   │   ├── __init__.py
│   │   └── settings.py   # Configuration management
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base.py      # Base agent class
│   │   ├── architect.py  # Project planning agent
│   │   ├── builder.py    # Code generation agent
│   │   └── inspector.py  # Code review agent
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── forge_ops.py  # File operations
│   │   └── cli.py       # CLI interface
│   └── core/
│       ├── __init__.py
│       └── synapse.py    # Agent coordination logic
└── tests/
    ├── __init__.py
    └── test_agents/
        └── test_synapse.py
```

## Setup Instructions

1. Clone the repository
```bash
git clone https://github.com/yourusername/cogitoforge.git
cd cogitoforge
```

2. Create and activate virtual environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies
```bash
pip install -r requirements.txt
```

4. Create .env file
```bash
touch .env
```

## Configuration
Create a `.env` file in the root directory with your API keys:
```
ANTHROPIC_API_KEY=your_key_here
CODESEEK_API_KEY=your_key_here
REVIEWER_API_KEY=your_key_here
```

## Usage

1. Start the forge:
```bash
python src/main.py
```

2. Follow the interactive prompts:
   - Select number of agents
   - Choose agent types
   - Provide API keys if not configured
   - Define your coding task
   - Select programming languages

3. CogitoForge will:
   - Create project structure
   - Generate code
   - Perform review
   - Save output in designated folder

## Program Flow

1. **Initialization**
   - Load configuration
   - Validate environment
   - Initialize CLI interface

2. **Agent Setup**
   - Collect user preferences
   - Validate API keys
   - Initialize selected agents

3. **Task Processing**
   - Get user requirements
   - Plan project structure
   - Divide tasks
   - Generate code
   - Review output

4. **Output Management**
   - Create output directory
   - Save generated files
   - Provide summary report

## Security Considerations

- API keys are stored locally in `.env`
- Keys are never logged or exposed
- Temporary files are cleaned up
- Input validation for all user data

## Error Handling

CogitoForge includes comprehensive error handling for:
- Invalid API keys
- Network failures
- File system errors
- Invalid input
- Agent failures

## Development Guidelines

1. **Code Style**
   - Follow PEP 8
   - Use type hints
   - Document all functions
   - Include error handling

2. **Testing**
   - Write unit tests
   - Mock API calls
   - Test file operations
   - Verify agent coordination

3. **Contributing**
   - Fork repository
   - Create feature branch
   - Submit pull request
   - Include tests

## Troubleshooting

Common issues and solutions:

1. **API Key Issues**
   - Verify key validity
   - Check .env file format
   - Ensure proper permissions

2. **File Operation Errors**
   - Check write permissions
   - Verify path existence
   - Clean temporary files

3. **Agent Failures**
   - Check API quotas
   - Verify network connection
   - Review error logs

## Support

For issues and feature requests:
- Create GitHub issue
- Include error logs
- Provide reproduction steps
- Specify environment details

## License
MIT License


[Previous sections remain the same until License, then add:]

## Implementation Guide

### Phase 1: Core Architecture Setup

1. **Project Initialization**
```python
# src/main.py
from typing import List, Dict, Any
from pathlib import Path
from rich.console import Console
from rich.prompt import Prompt

class Cogito:
    def __init__(self):
        self.console = Console()
        self.config = self.load_configuration()
        self.agents = {}
        self.workspace = Path("./workspace")
```

2. **Configuration Management**
```python
# src/config/settings.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

    ANTHROPIC_API_KEY: str = ""
    CODESEEK_API_KEY: str = ""
    REVIEWER_API_KEY: str = ""
    DEFAULT_MODEL: str = "claude-3-sonnet"
    WORKSPACE_DIR: str = "./workspace"
```

3. **Base Agent Interface**
```python
# src/agents/base.py
from abc import ABC, abstractmethod
from typing import Dict, Any

class Agent(ABC):
    def __init__(self, api_key: str, model: str):
        self.api_key = api_key
        self.model = model
        self.context = {}

    @abstractmethod
    async def process(self, task: Dict[str, Any]) -> Dict[str, Any]:
        pass

    @abstractmethod
    async def validate_response(self, response: Dict[str, Any]) -> bool:
        pass
```

### Phase 2: Agent Implementation

1. **Architect Agent (Project Planning)**
```python
# src/agents/architect.py
from .base import Agent
import anthropic

class Architect(Agent):
    def __init__(self, api_key: str, model: str = "claude-3-sonnet"):
        super().__init__(api_key, model)
        self.client = anthropic.Client(api_key=api_key)
        self.system_prompt = """
        You are a software architect specializing in:
        - Project structure design
        - Directory organization
        - Task breakdown
        - Dependency management
        
        Provide detailed plans including:
        1. Directory structure
        2. File organization
        3. Module dependencies
        4. Implementation steps
        """

    async def process(self, task: Dict[str, Any]) -> Dict[str, Any]:
        response = await self.client.messages.create(
            model=self.model,
            system=self.system_prompt,
            messages=[{"role": "user", "content": task["requirements"]}]
        )
        return self._parse_response(response)
```

2. **Builder Agent (Code Generation)**
```python
# src/agents/builder.py
class Builder(Agent):
    def __init__(self, api_key: str, model: str = "claude-3-sonnet"):
        super().__init__(api_key, model)
        self.system_prompt = """
        You are a code generation specialist focusing on:
        - Clean code implementation
        - Best practices
        - Documentation
        - Error handling
        
        Generate code following:
        1. Project specifications
        2. Language-specific conventions
        3. Design patterns
        4. Testing requirements
        """
```

3. **Inspector Agent (Code Review)**
```python
# src/agents/inspector.py
class Inspector(Agent):
    def __init__(self, api_key: str, model: str = "claude-3-sonnet"):
        super().__init__(api_key, model)
        self.system_prompt = """
        You are a code review specialist focusing on:
        - Code quality
        - Performance optimization
        - Security considerations
        - Best practices adherence
        
        Review and provide:
        1. Issues found
        2. Improvement suggestions
        3. Security concerns
        4. Performance optimizations
        """
```

### Phase 3: Core Logic Implementation

1. **Synapse (Agent Coordination)**
```python
# src/core/synapse.py
from typing import List, Dict
from ..agents import Architect, Builder, Inspector

class Synapse:
    def __init__(self, agents: Dict[str, Agent]):
        self.agents = agents
        self.task_queue = []
        self.results = {}

    async def orchestrate(self, task: Dict[str, Any]) -> Dict[str, Any]:
        # 1. Get project structure from Architect
        plan = await self.agents['architect'].process(task)
        
        # 2. Break down into subtasks
        subtasks = self._create_subtasks(plan)
        
        # 3. Assign to Builder
        code_modules = await self._parallel_process(
            self.agents['builder'], 
            subtasks
        )
        
        # 4. Send to Inspector
        review = await self.agents['inspector'].process({
            'code': code_modules,
            'requirements': task['requirements']
        })
        
        return self._compile_results(plan, code_modules, review)
```

2. **File Operations**
```python
# src/utils/forge_ops.py
from pathlib import Path
import shutil
import json

class ForgeOps:
    def __init__(self, workspace: Path):
        self.workspace = workspace
        self.workspace.mkdir(exist_ok=True)

    def create_project_structure(self, structure: Dict[str, Any]):
        for path, content in structure.items():
            full_path = self.workspace / path
            if isinstance(content, dict):
                full_path.mkdir(parents=True, exist_ok=True)
                self._create_files(full_path, content)
            else:
                full_path.write_text(content)
```

### Phase 4: CLI Interface

```python
# src/utils/cli.py
import typer
from rich.console import Console
from rich.prompt import Prompt
from rich.panel import Panel

app = typer.Typer()
console = Console()

@app.command()
def setup():
    """Initial setup and configuration"""
    console.print(Panel("Welcome to Cogito Setup"))
    api_keys = _collect_api_keys()
    _save_configuration(api_keys)

@app.command()
def create():
    """Start a new project"""
    requirements = _get_project_requirements()
    languages = _get_language_preferences()
    agents = _initialize_agents()
    
    with console.status("Processing..."):
        result = agents.orchestrate({
            'requirements': requirements,
            'languages': languages
        })
```

### Phase 5: Implementation Steps

1. **Setup Development Environment**
   - Create virtual environment
   - Install dependencies
   - Setup configuration files
   - Initialize workspace

2. **Implement Core Components**
   - Create base Agent class
   - Implement Synapse coordinator
   - Setup file operations
   - Create configuration management

3. **Implement Agents**
   - Architect agent for planning
   - Builder agent for coding
   - Inspector agent for review
   - Test each agent individually

4. **Develop CLI Interface**
   - User input collection
   - Configuration management
   - Progress display
   - Error handling

5. **Integration and Testing**
   - Connect all components
   - Test full workflow
   - Add logging
   - Error recovery

### Data Flow

1. **User Input → System**
   ```json
   {
     "task": "Create a web scraper",
     "language": "python",
     "requirements": ["async support", "error handling"],
     "output_format": "module"
   }
   ```

2. **Architect → Builder**
   ```json
   {
     "structure": {
       "src": {
         "scraper.py": null,
         "utils": {
           "parser.py": null,
           "error_handling.py": null
         }
       },
       "tests": {
         "test_scraper.py": null
       }
     },
     "dependencies": [
       "aiohttp",
       "beautifulsoup4"
     ]
   }
   ```

3. **Builder → Inspector**
   ```json
   {
     "files": {
       "src/scraper.py": "code content...",
       "src/utils/parser.py": "code content...",
       "src/utils/error_handling.py": "code content..."
     },
     "tests": {
       "test_scraper.py": "test content..."
     }
   }
   ```

4. **Inspector → System**
   ```json
   {
     "status": "approved",
     "suggestions": [
       {
         "file": "src/scraper.py",
         "line": 45,
         "suggestion": "Add timeout handling"
       }
     ],
     "security_concerns": [],
     "performance_notes": []
   }
   ```

### Error Handling

1. **API Failures**
```python
class APIError(Exception):
    def __init__(self, agent_type: str, message: str):
        self.agent_type = agent_type
        self.message = message
        super().__init__(f"{agent_type} API Error: {message}")

async def retry_with_backoff(func, max_retries=3):
    for i in range(max_retries):
        try:
            return await func()
        except Exception as e:
            if i == max_retries - 1:
                raise
            await asyncio.sleep(2 ** i)
```

2. **File System Errors**
```python
class FileOpsError(Exception):
    pass

def safe_file_operation(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except OSError as e:
            raise FileOpsError(f"File operation failed: {str(e)}")
    return wrapper
```

### Testing Strategy

1. **Unit Tests**
```python
# tests/test_agents/test_architect.py
import pytest
from src.agents.architect import Architect

@pytest.mark.asyncio
async def test_architect_planning():
    architect = Architect(api_key="test_key")
    result = await architect.process({
        "requirements": "Create a web scraper"
    })
    assert "structure" in result
    assert "dependencies" in result
```

2. **Integration Tests**
```python
# tests/test_integration.py
async def test_full_workflow():
    cogito = Cogito()
    result = await cogito.process_task({
        "task": "Create a web scraper",
        "language": "python"
    })
    assert result["status"] == "complete"
    assert Path("workspace/src/scraper.py").exists()
```

### Success Criteria

1. **Project Planning**
   - Clear directory structure
   - Appropriate file organization
   - Complete dependency list
   - Implementation roadmap

2. **Code Generation**
   - Functional code
   - Proper error handling
   - Documentation
   - Test coverage
   - Best practices followed

3. **Code Review**
   - Security check passed
   - Performance optimized
   - Code style consistent
   - No critical issues

4. **User Experience**
   - Clear progress indication
   - Error messages helpful
   - Configuration simple
   - Output organized

This implementation guide provides the foundation for building Cogito. Each component is designed to be modular and extensible, allowing for future improvements and additional features. The system uses asynchronous operations where possible for better performance and provides comprehensive error handling and logging.

Remember to implement proper testing at each stage and maintain documentation as the system grows. The modular design allows for easy replacement of components or addition of new agent types in the future.