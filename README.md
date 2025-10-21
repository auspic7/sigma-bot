# Sigma Bot

An automated trading bot leveraging LLM capabilities for intelligent market analysis and decision-making.

## Project Status

🚧 **Project Initialization** - Constitution and development framework established.

## Core Principles

This project follows strict engineering principles defined in [`.specify/memory/constitution.md`](.specify/memory/constitution.md):

1. **Root Cause Analysis** (NON-NEGOTIABLE) - No workarounds, only proper fixes
2. **Modern Tooling** - Python 3.11+ with `uv` package management
3. **Documentation-Driven** - Use Context7/official docs for all integrations
4. **Test-First Development** (NON-NEGOTIABLE) - TDD workflow strictly enforced
5. **LLM Integration Patterns** - Structured, safe, and cost-controlled
6. **Risk Management & Safety** - Multiple layers of trading safeguards
7. **Observability** - Comprehensive logging, metrics, and tracing
8. **Commit Discipline** - Frequent, atomic commits for clear history tracking

## Technology Stack

- **Language**: Python 3.11+
- **Package Manager**: uv (exclusive, mandatory)
- **Testing**: pytest with pytest-asyncio
- **Type Checking**: mypy (strict mode)
- **Linting**: ruff
- **Logging**: structlog (JSON format)
- **Trading**: ccxt (exchange integrations)
- **LLM**: OpenAI SDK, Anthropic SDK

## Getting Started

### Prerequisites

- Python 3.11 or higher
- uv package manager ([installation guide](https://github.com/astral-sh/uv))

### Installation

```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone the repository
git clone <repository-url>
cd sigma-bot

# Install dependencies
uv sync

# Run tests
uv run pytest

# Run type checking
uv run mypy src/

# Run linting
uv run ruff check .
```

## Project Structure

```
sigma-bot/
├── .specify/                 # Project governance and templates
│   ├── memory/
│   │   └── constitution.md  # Project constitution (principles & rules)
│   └── templates/           # Templates for specs, plans, tasks
├── src/                     # Source code (to be created)
├── tests/                   # Test suites (to be created)
│   ├── unit/
│   ├── integration/
│   └── contract/
├── specs/                   # Feature specifications (created by /speckit commands)
├── pyproject.toml          # Project configuration (to be created)
└── README.md               # This file
```

## Development Workflow

### 1. Feature Specification

Use `/speckit.spec` to create detailed feature specifications with:

- User scenarios and acceptance criteria
- Functional requirements
- Safety and risk controls (for trading features)
- Success metrics

### 2. Implementation Planning

Use `/speckit.plan` to create implementation plans with:

- Technical context and dependencies
- Constitution compliance checks
- Project structure
- Complexity justification (if needed)

### 3. Task Breakdown

Use `/speckit.tasks` to generate task lists organized by:

- User story priority (P1, P2, P3...)
- Dependencies and parallel execution opportunities
- Test-first workflow requirements

### 4. Implementation

Follow TDD workflow strictly:

1. Write tests first
2. Get stakeholder approval on test scenarios
3. Verify tests fail (red state)
4. Implement feature (green state)
5. Refactor as needed

## Safety Controls

For trading operations, the following safety mechanisms MUST be implemented:

- **Position Limits**: Hardcoded maximum position sizes
- **Loss Limits**: Per-trade and daily loss thresholds
- **Kill Switch**: Emergency shutdown mechanism
- **Dry-Run Mode**: Paper trading validation before real money
- **Rate Limiting**: Prevent runaway order execution
- **Audit Logging**: Every decision and execution logged

## LLM Integration Guidelines

When integrating LLM services:

- Always set explicit timeouts
- Implement token usage tracking and budgets
- Validate all LLM responses before use
- Prefer structured outputs (JSON schemas)
- Provide fallback logic for failures
- Never pass credentials to LLM contexts
- Log all LLM decision rationale for audits

## Development Guidelines

This is a solo development project. All code changes must:

1. Comply with the constitution principles
2. Self-review for constitution compliance before committing
3. Maintain test coverage (≥80% for trading logic, ≥60% overall)
4. Address root causes only - no workarounds
5. Document all architectural decisions
6. Use conventional commit messages for clear history
7. Commit after each valid, working unit of modification

## Governance

The project constitution supersedes all other practices. For details on:

- Amendment procedures
- Version management
- Compliance reviews

See [`.specify/memory/constitution.md`](.specify/memory/constitution.md#governance)

## License

[To be determined]

## Contact

[To be determined]
