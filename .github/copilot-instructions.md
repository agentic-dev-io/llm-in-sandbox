# Copilot Coding Agent Instructions for llm-in-sandbox

## Project Overview

**LLM-in-Sandbox** is a lightweight Python framework that enables Large Language Models (LLMs) to explore within a Docker-based code sandbox (virtual computer) to build general-purpose agents. The project focuses on eliciting general agentic intelligence beyond just coding tasks - including scientific reasoning, long-context understanding, video production, travel planning, and more.

- **Paper**: arXiv:2601.16206
- **Version**: 0.1.0 (Beta)
- **License**: Apache 2.0
- **Project Page**: https://llm-in-sandbox.github.io

## Technology Stack

### Core Technologies
- **Language**: Python 3.10, 3.11, 3.12 (Python 3.10+ required)
- **Build System**: Hatchling (PEP 517/518 compliant)
- **Container Runtime**: Docker (required for sandbox execution)
- **CLI Framework**: Python Fire

### Key Dependencies
- `litellm>=1.0.0` - LLM API abstraction layer for OpenAI, Anthropic, vLLM, SGLang
- `docker>=6.0.0` - Docker SDK for Python
- `pyyaml>=6.0` - YAML configuration parsing
- `fire>=0.5.0` - CLI generation
- `pydantic>=2.0.0` - Data validation and settings management
- `rich>=13.0.0` - Terminal formatting and output

### Development Tools
- `pytest>=7.0.0` - Testing framework (no tests in repo currently)
- `black>=23.0.0` - Code formatting (line length: 100)
- `ruff>=0.1.0` - Fast Python linter (select: E, F, W, I)

## Repository Structure

```
llm-in-sandbox/
├── llm_in_sandbox/              # Main package
│   ├── __init__.py              # Package initialization, exports main classes
│   ├── agent.py                 # Agent class with LLM interaction loop
│   ├── action.py                # Action data structures
│   ├── cli.py                   # CLI entry point (llm-in-sandbox command)
│   ├── docker_runtime.py        # Docker container lifecycle management
│   ├── observation.py           # Observation data structures
│   ├── tools.py                 # Tool definitions (bash, editor, submit)
│   ├── trajectory.py            # Execution history tracking
│   └── config/
│       ├── general.yaml         # Default system/instance prompts
│       └── runtime.example.yaml # Example runtime config
├── docker/
│   └── Dockerfile               # Sandbox image (Python 3.11-slim + tools)
├── examples/                    # 10+ diverse task examples
│   ├── 01_travel_planning/
│   ├── 02_poster_design/
│   ├── 03_video_creation/
│   └── ... (7 more examples)
├── pyproject.toml               # Package metadata & build config
├── config.yaml                  # Default LLM configuration
├── README.md                    # Main documentation
└── LICENSE                      # Apache 2.0

Ignored artifacts: output/, venv/, .venv/, test_env, __pycache__/, *.pyc, dist/, demo_page/
```

## Installation & Setup

### Prerequisites
- Python 3.10 or higher
- Docker installed and running (https://docs.docker.com/engine/install/)

### Installation Methods

**From PyPI (when published):**
```bash
pip install llm-in-sandbox
```

**From Source (Development):**
```bash
git clone https://github.com/llm-in-sandbox/llm-in-sandbox.git
cd llm-in-sandbox
pip install -e .                 # Basic install
pip install -e ".[dev]"          # With dev dependencies
```

**Note**: The package requires dependencies to be installed before the CLI works. If you get `ModuleNotFoundError: No module named 'litellm'`, ensure all dependencies are installed with `pip install -e .`

### Docker Image Setup

The default Docker image (`cdx123/llm-in-sandbox:v0.1`) is automatically pulled on first run. To build a custom image:

```bash
llm-in-sandbox build
# Or with custom name
llm-in-sandbox build --image_name my-custom-image:v1 --force
```

## CLI Usage

### Main Commands

1. **`llm-in-sandbox run`** - Run an agent task
2. **`llm-in-sandbox build`** - Build Docker image

### Common Run Patterns

**With Cloud LLM Service:**
```bash
llm-in-sandbox run \
    --query "Your task description" \
    --llm_name "openai/gpt-4" \
    --llm_base_url "http://your-api-server/v1" \
    --api_key "your-api-key"
```

**With Self-Hosted Model (vLLM):**
```bash
# Start vLLM server first
vllm serve Qwen/Qwen3-Coder-30B-A3B-Instruct \
    --served-model-name qwen3_coder \
    --enable-auto-tool-choice \
    --tool-call-parser qwen3_coder

# Run agent
llm-in-sandbox run \
    --query "write a hello world in python" \
    --llm_name qwen3_coder \
    --llm_base_url "http://localhost:8000/v1"
```

**With Input/Output Files:**
```bash
llm-in-sandbox run \
    --query "$(cat prompt.txt)" \
    --input_dir ./input \
    --output_dir ./my_output \
    --llm_name "your-model" \
    --llm_base_url "http://localhost:8000/v1"
```

**With Custom Prompt Config:**
```bash
llm-in-sandbox run \
    --query "$(cat prompt.txt)" \
    --prompt_config ./custom_prompts.yaml \
    --llm_name "your-model"
```

### Key Parameters

| Parameter | Description | Default | Notes |
|-----------|-------------|---------|-------|
| `--query` | Task description | *required* | Can use `$(cat file.txt)` |
| `--llm_name` | Model name | *required* | Auto-prefixed with `openai/` if needed |
| `--llm_base_url` | API endpoint | `LLM_BASE_URL` env var | Required for local servers |
| `--api_key` | API key | `OPENAI_API_KEY` env var | Not needed for local servers |
| `--docker_image` | Container image | `cdx123/llm-in-sandbox:v0.1` | |
| `--input_dir` | Input files directory | None | Mounted at `/testbed/input` |
| `--output_dir` | Output directory | `./output` | Timestamped subdirs created |
| `--prompt_config` | Prompt template YAML | `./config/general.yaml` | |
| `--max_steps` | Max conversation turns | 100 | |
| `--temperature` | Sampling temperature | 1.0 | |
| `--extra_body` | Extra JSON for API calls | None | e.g., `'{"chat_template_kwargs": {"thinking": True}}'` |

Run `llm-in-sandbox run --help` for all available parameters.

## Configuration Files

### Runtime Settings (`config.yaml`, `llm-in-sandbox.yaml`)

Set default values to avoid repeating CLI args. Checked in this order:
1. `--settings` path (explicit)
2. `LLM_IN_SANDBOX_CONFIG` environment variable
3. `./llm-in-sandbox.yaml` or `./llm_in_sandbox.yaml` (current directory)
4. `~/.llm-in-sandbox/config.yaml` or `~/.llm-in-sandbox.yaml` (home directory)

Example:
```yaml
llm_name: gpt-4
llm_base_url: http://localhost:8000/v1
api_key: your-api-key
prompt_config: ./my_prompts.yaml
```

### Prompt Configuration (`general.yaml`)

Defines system and instance prompts for the agent. Key sections:
- `system_prompt`: Instructs the agent on its role, tools, and workflow
- `instance_prompt`: Template with `{problem_statement}` and `{working_dir}` placeholders

**Important system prompt conventions:**
- Agent works in `/testbed` directory
- Input files are at `/testbed/input`
- **ALL outputs must go to `/testbed/output`**
- Agent must **write code to files and execute them** (no hardcoding answers)
- Three tools: `str_replace_editor` (file operations), `execute_bash` (run commands), `submit` (finish task)
- Anti-hardcoding: No large comment blocks for thinking, no hardcoded answers like `answer = "A"`

## Coding Conventions & Patterns

### Code Style
- **Line Length**: 100 characters (enforced by black and ruff)
- **Python Version**: Target Python 3.10 as minimum (use compatible syntax)
- **Formatting**: Use `black` for consistent formatting
- **Linting**: Use `ruff` with rules E, F, W, I (errors, pyflakes, warnings, import order)

### Code Patterns

1. **Logging**: Use Rich console and custom logger with `RichHandler`
   ```python
   from rich.console import Console
   from rich.panel import Panel
   console = Console()
   
   from .agent import get_logger
   logger = get_logger("MyModule")
   logger.info("Message")
   ```

2. **Docker Operations**: All runtime execution happens in Docker containers
   - Container lifecycle: `DockerRuntime.__init__` → operations → `close()`
   - Commands run via `runtime.run(command)` or `runtime.run_with_output(command)`
   - File transfers: `copy_dir_to_container()`, `copy_from_container()`

3. **LLM Integration**: Uses `litellm` for unified API interface
   ```python
   import litellm
   litellm.drop_params = True  # Ignore unsupported params
   ```

4. **Configuration Loading**: YAML files for prompts and settings
   ```python
   import yaml
   with open(config_path, "r") as f:
       config = yaml.safe_load(f)
   ```

5. **CLI with Fire**: Simple command routing
   ```python
   import fire
   fire.Fire({
       "run": run_agent_query,
       "build": build_docker_image,
   })
   ```

6. **Path Handling**: Use `pathlib.Path` for cross-platform compatibility
   ```python
   from pathlib import Path
   output_dir = Path(output_dir) / timestamp
   output_dir.mkdir(parents=True, exist_ok=True)
   ```

### Tools Definition Pattern

Tools follow OpenAI function calling format (see `tools.py`):
```python
tool = {
    "type": "function",
    "function": {
        "name": "tool_name",
        "description": "Tool description...",
        "parameters": {
            "type": "object",
            "properties": {
                "param_name": {
                    "type": "string",
                    "description": "Parameter description"
                }
            },
            "required": ["param_name"]
        }
    }
}
```

## Building & Testing

### Building the Package
```bash
# Development install (editable)
pip install -e .
pip install -e ".[dev]"  # With dev tools

# Build wheel
pip install build
python -m build
```

### Building Docker Image
```bash
llm-in-sandbox build
# Or manually
docker build -t llm-in-sandbox:v0.1 -f docker/Dockerfile docker/
```

### Code Quality
```bash
# Format code
black llm_in_sandbox/

# Lint code
ruff check llm_in_sandbox/

# Fix auto-fixable issues
ruff check --fix llm_in_sandbox/
```

### Testing

**Note**: No test files currently exist in the repository. When adding tests:
- Use `pytest` as the test framework
- Place tests in a `tests/` directory at repo root
- Follow naming: `test_*.py` for files, `test_*()` for functions
- Install dev dependencies: `pip install -e ".[dev]"`
- Run tests: `pytest tests/`

## Common Issues & Solutions

### 1. Docker Image Not Found
**Error**: `Docker image 'cdx123/llm-in-sandbox:v0.1' not found`
**Solution**: The CLI automatically pulls the image on first run. If that fails:
```bash
docker pull cdx123/llm-in-sandbox:v0.1
# Or build locally
llm-in-sandbox build
```

### 2. Module Import Errors
**Error**: `ModuleNotFoundError: No module named 'litellm'`
**Cause**: Dependencies not installed
**Solution**:
```bash
pip install -e .
# Or reinstall dependencies
pip install litellm docker pyyaml fire pydantic rich
```

### 3. API Key Issues
**Error**: API authentication failures
**Solution**: Set environment variables or use `--api_key`:
```bash
export OPENAI_API_KEY="your-key"
export ANTHROPIC_API_KEY="your-key"
# Or for custom endpoints
export LLM_BASE_URL="http://localhost:8000/v1"
```

For local servers without authentication, the CLI sets dummy keys automatically.

### 4. Model Name Auto-Prefixing
**Behavior**: Model names without provider prefix get `openai/` added automatically
**Example**: `qwen3_coder` becomes `openai/qwen3_coder`
**Control**: Explicitly prefix with `openai/`, `anthropic/`, `azure/`, or `hosted_vllm/`

### 5. Docker Permission Issues
**Error**: Permission denied accessing Docker
**Solution**: Ensure user is in `docker` group:
```bash
sudo usermod -aG docker $USER
# Then log out and back in
```

### 6. Extra Body JSON Parsing
**Error**: `Invalid extra_body JSON`
**Issue**: JSON parsing for `--extra_body` parameter
**Solution**: Use proper JSON string with escaped quotes:
```bash
--extra_body '{"chat_template_kwargs": {"thinking": true}}'
```
Note: String `'true'` is auto-converted to boolean `True`.

## Output Structure

Each run creates a timestamped output directory:
```
output/YYYYMMDD_HHMMSS/
├── files/                      # Container output files
│   ├── answer.txt              # Final answer (if applicable)
│   └── [other output files]    # Task-specific outputs
└── trajectory.json             # Complete execution history
```

## Development Workflow

### Making Changes

1. **Clone and setup**:
   ```bash
   git clone https://github.com/llm-in-sandbox/llm-in-sandbox.git
   cd llm-in-sandbox
   pip install -e ".[dev]"
   ```

2. **Make changes** to Python files in `llm_in_sandbox/`

3. **Format and lint**:
   ```bash
   black llm_in_sandbox/
   ruff check --fix llm_in_sandbox/
   ```

4. **Test manually** (no automated tests currently):
   ```bash
   # Test CLI
   llm-in-sandbox build
   llm-in-sandbox run --query "test task" --llm_name test-model
   ```

5. **Update documentation** if changing CLI, config format, or adding features

### Adding New Features

- **New tools**: Add to `tools.py` following the function calling format
- **Configuration options**: Update `AgentArgs` dataclass and CLI parameters
- **Prompt templates**: Modify `llm_in_sandbox/config/general.yaml`
- **Docker image changes**: Update `docker/Dockerfile` and rebuild

## Examples

The `examples/` directory contains 10+ diverse task demonstrations. Each example typically has:
- `prompt.txt` - Task description
- `input/` - Input files (if needed)
- `prompt_config.yaml` - Custom prompts (for examples 05-10)

To run an example:
```bash
cd examples/01_travel_planning
llm-in-sandbox run \
    --query "$(cat prompt.txt)" \
    --llm_name "your-model" \
    --llm_base_url "http://localhost:8000/v1"
```

See [examples/README.md](../examples/README.md) for full details.

## Important Notes for Coding Agents

### When Working on This Repository

1. **Always check Docker is available**: Many operations require Docker runtime
2. **Install dependencies before testing**: Run `pip install -e .` after cloning
3. **Respect the sandbox architecture**: All agent execution happens in isolated containers
4. **Follow the prompt conventions**: If modifying prompts, maintain the anti-hardcoding philosophy
5. **Test with actual LLM endpoints**: The framework requires a working LLM API to test fully
6. **Output to correct directories**: Container outputs go to `/testbed/output`, which gets copied to host

### When Making Code Changes

1. **Maintain Python 3.10+ compatibility**: Don't use Python 3.11+ exclusive features
2. **Keep line length at 100**: Use black formatting
3. **Update both code and docs**: CLI changes should be reflected in README.md
4. **Test Docker integration**: Changes to `docker_runtime.py` need container testing
5. **Preserve tool interfaces**: The agent relies on specific tool function signatures

### When Debugging Issues

1. **Check Docker logs**: Container failures may not show full error details
2. **Verify LLM connectivity**: Test the LLM endpoint independently first
3. **Inspect trajectory.json**: Contains full execution history for debugging
4. **Use Rich console output**: Provides colored, structured terminal output
5. **Check environment variables**: API keys and base URLs may be loaded from env

## Useful Commands Reference

```bash
# Installation
pip install -e .                              # Basic install
pip install -e ".[dev]"                       # With dev tools

# Docker
llm-in-sandbox build                          # Build default image
llm-in-sandbox build --force                  # Force rebuild
docker pull cdx123/llm-in-sandbox:v0.1       # Pull pre-built image

# Running
llm-in-sandbox run --query "task" --llm_name model  # Basic run
llm-in-sandbox run --help                     # Show all options

# Code quality
black llm_in_sandbox/                         # Format code
ruff check llm_in_sandbox/                    # Lint code
ruff check --fix llm_in_sandbox/             # Auto-fix issues

# Development
python -m llm_in_sandbox.cli run ...         # Direct module invocation
git log --oneline -20                         # Recent changes
```

## References

- **GitHub Repository**: https://github.com/llm-in-sandbox/llm-in-sandbox
- **Project Website**: https://llm-in-sandbox.github.io
- **Research Paper**: https://arxiv.org/abs/2601.16206
- **Docker Documentation**: https://docs.docker.com/
- **LiteLLM Docs**: https://docs.litellm.ai/

---

**Last Updated**: 2026-01-28  
**Document Version**: 1.0  
**Repository Version**: 0.1.0
