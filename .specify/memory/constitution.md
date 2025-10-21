<!--
═══════════════════════════════════════════════════════════════════════════════
SYNC IMPACT REPORT
═══════════════════════════════════════════════════════════════════════════════
Version Change: 1.0.0 → 1.1.0
Amendment Type: MINOR (new principle added + collaboration rules adapted for solo dev)
Ratification: 2025-10-21
Last Amended: 2025-10-21

Modified Principles:
  - ADDED: VIII. Commit Discipline (frequent commits for history tracking)

Removed/Modified Sections:
  - Development Workflow > Code Review Requirements: Removed peer review requirement,
    adapted to solo development self-review discipline
  - Development Workflow > Complexity Management: Removed team consensus requirement,
    kept documentation requirement
  - Governance > Amendment Process: Removed team consensus requirement for solo dev
  - Governance > Compliance Review: Adapted pull request language for solo workflow

Principles (v1.1.0):
  1. Root Cause Analysis (NON-NEGOTIABLE)
  2. Modern Tooling & Dependency Management
  3. Documentation-Driven Development
  4. Test-First Development (NON-NEGOTIABLE)
  5. LLM Integration Patterns
  6. Risk Management & Safety
  7. Observability & Monitoring
  8. Commit Discipline [NEW]

Template Synchronization Status:
  ✅ plan-template.md - Already aligned (no updates needed)
  ✅ spec-template.md - Already aligned (no updates needed)
  ✅ tasks-template.md - Needs update to reference Commit Discipline principle
  ✅ README.md - Needs update to add 8th principle

Follow-up Actions:
  - Update tasks-template.md to mention commit frequency
  - Update README.md to list 8 principles instead of 7

Version Bump Rationale:
  - MINOR (not MAJOR): No backward incompatible changes to existing principles
  - MINOR (not PATCH): New principle added (Commit Discipline)
  - Collaboration adaptations are clarifications, not principle changes

Notes:
  - Solo development workflow emphasized
  - Self-discipline and documentation still maintained
  - Commit frequency added to support better history tracking
═══════════════════════════════════════════════════════════════════════════════
-->

# Sigma Bot Constitution

## Core Principles

### I. Root Cause Analysis (NON-NEGOTIABLE)

When encountering any problem, issue, or bug, you MUST identify and address the root cause rather than implementing workarounds or superficial fixes.

**Rules:**

- Every problem requires a documented root cause analysis
- Workarounds are explicitly forbidden unless temporarily required for production incidents (must be followed by proper fix within 48 hours)
- "Quick fixes" that don't address underlying issues must be rejected
- When uncertain about root causes, investigation continues until clarity is achieved

**Rationale:** Trading bots operate with real financial consequences. Surface-level fixes compound technical debt and can mask critical systemic issues that lead to financial losses. Root cause discipline ensures system reliability and prevents cascading failures.

### II. Modern Tooling & Dependency Management

All Python dependencies MUST be managed using `uv`. The project MUST use current, maintained versions of all tools and libraries.

**Rules:**

- `uv` is the exclusive package manager (no pip, poetry, or conda)
- Dependencies must specify minimum versions that are actively supported
- Monthly dependency audits to check for updates, security patches, and deprecations
- New tools/libraries require justification for "latest stable" version selection
- Legacy/unmaintained packages require explicit approval and migration plan

**Rationale:** Modern tooling provides better performance, security, and developer productivity. uv offers fast, reliable dependency resolution. Staying current reduces security vulnerabilities and ensures access to latest bug fixes and features.

### III. Documentation-Driven Development

When facing uncertainty about library usage, API design, or best practices, you MUST consult official documentation using Context7 or equivalent tools before implementation.

**Rules:**

- Unknown library features require Context7 documentation lookup before coding
- API design decisions must reference official framework documentation
- Implementation patterns should follow documented best practices from authoritative sources
- "I think it works like this" is not acceptable—verify with official docs
- Document all architectural decisions with references to source documentation

**Rationale:** Trading systems require correctness. Assumptions lead to subtle bugs. Official documentation provides authoritative, tested guidance. This principle reduces bugs from misunderstood APIs and ensures implementations follow proven patterns.

### IV. Test-First Development (NON-NEGOTIABLE)

All feature development MUST follow Test-Driven Development (TDD): write tests, get approval on test scenarios, verify tests fail, then implement.

**Rules:**

- Tests written BEFORE implementation code
- Stakeholder/self-approval required on test scenarios before implementation begins
- Initial test run must demonstrate failures (red state)
- Implementation proceeds only after test failures confirmed
- Red-Green-Refactor cycle strictly enforced
- Contract tests required for all external integrations (exchanges, LLM APIs, data feeds)
- Integration tests required for critical trading logic paths

**Rationale:** Financial systems demand correctness. TDD ensures requirements are testable, understood, and verified before code is written. Pre-approved test scenarios align expectations. Failed-first discipline prevents false positives from bad tests.

### V. LLM Integration Patterns

All LLM integrations MUST follow structured patterns with proper error handling, cost controls, and fallback strategies.

**Rules:**

- LLM calls must have explicit timeout limits
- Implement token usage tracking and budget limits per operation
- All LLM responses require validation before use in trading decisions
- Structured output formats (JSON schemas) preferred over free-text parsing
- Fallback logic required for LLM service failures
- LLM decision rationale must be logged for audit trails
- Never pass sensitive credentials or API keys to LLM contexts

**Rationale:** LLMs are probabilistic and can fail or produce invalid outputs. Trading decisions require deterministic validation. Cost control prevents runaway expenses. Structured patterns ensure safe, auditable LLM integration.

### VI. Risk Management & Safety

All trading operations MUST implement multiple layers of safety controls to prevent catastrophic losses.

**Rules:**

- Position size limits enforced at code level (not just configuration)
- Maximum loss per trade and per day must be hardcoded limits
- Kill switch mechanism required for emergency shutdown
- Dry-run mode mandatory for all new strategies (paper trading first)
- Real money trading requires explicit manual approval flag
- Rate limiting on order placement to prevent runaway execution
- Balance checks before every trade execution
- Audit log for every trading decision and execution

**Rationale:** Trading bots can lose significant money very quickly if bugs occur. Defense-in-depth safety controls prevent single points of failure. Hardcoded limits prevent configuration errors. Audit trails enable post-mortem analysis and regulatory compliance.

### VII. Observability & Monitoring

All system operations MUST be observable through structured logging, metrics, and tracing.

**Rules:**

- Structured logging (JSON format) required for all components
- Every trading decision must be logged with full context (reasoning, signals, market data)
- Metrics collection for: API latency, LLM costs, trade performance, system health
- Distributed tracing for request flows across services
- Error tracking with stack traces and context
- Performance monitoring for critical paths (market data ingestion, order execution)
- Log levels: DEBUG for development, INFO for production operations, ERROR for actionable issues

**Rationale:** Trading systems require real-time visibility for debugging, performance optimization, and audit compliance. Structured observability enables rapid incident response and post-trade analysis. Cannot fix what you cannot see.

### VIII. Commit Discipline

Every valid, working unit of modification MUST be committed to version control to maintain clear, traceable project history.

**Rules:**

- Commit after each logical, working unit of change (feature component, bug fix, refactoring)
- Each commit must represent a functional state (code runs without breaking)
- Commit messages must clearly describe what changed and why
- Follow conventional commit format: `type(scope): description`
  - Types: feat, fix, docs, refactor, test, chore
  - Example: `feat(trading): add position size limit validation`
- Never commit broken/non-functional code to main branch
- Commit before switching contexts or starting new features
- WIP commits allowed on feature branches, but squash before merging to main

**Rationale:** Frequent, atomic commits create a detailed project history that enables easy debugging, rollback, and understanding of evolution. For solo development, commits serve as checkpoints and documentation of decision-making process. Clear history is essential for troubleshooting trading issues and understanding when bugs were introduced.

## Technology Stack Requirements

**Language**: Python 3.11+ (latest stable Python version)

**Package Management**: uv (exclusive, mandatory)

**Testing Framework**: pytest with pytest-asyncio for async code

**Logging**: structlog for structured JSON logging

**LLM Integration**:

- OpenAI SDK for GPT models
- Anthropic SDK for Claude models
- LangChain only if multi-provider abstraction required (justify first)

**Trading Libraries**:

- ccxt for exchange integrations
- pandas for data analysis
- numpy for numerical computations

**Async Runtime**: asyncio for concurrent operations

**Type Checking**: mypy with strict mode enabled

**Code Quality**: ruff for linting and formatting

**Environment Management**:

- .env files for local development
- Environment variables for production secrets
- Never commit secrets to version control

## Development Workflow

### Self-Review Requirements

As a solo developer, maintain discipline through self-review before finalizing changes:

- Verify Constitution compliance for all code changes
- Test coverage required for all new code paths
- Root cause documentation required for bug fixes
- LLM integration changes require extra scrutiny for safety controls
- Use commit history and git diff to review your own changes

### Testing Gates

**Before Commit** (for main branch):

- All tests pass (unit, integration, contract)
- Type checking passes (mypy strict mode)
- Linting passes (ruff)
- Test coverage meets threshold (≥80% for trading logic, ≥60% overall)

**Before Production Deployment**:

- Paper trading validation (minimum 7 days for new strategies)
- Performance profiling completed
- Observability validated (logs, metrics, traces)
- Risk limits tested and verified
- Kill switch tested and operational

### Complexity Management

When adding complexity (new patterns, abstractions, dependencies):

1. Document why simpler alternatives are insufficient
2. Ensure complexity is justified by measurable benefit
3. Document the decision in code comments or ADR (Architecture Decision Record)
4. Provide migration path if introducing breaking changes

## Governance

**Constitution Authority**: This constitution supersedes all other project practices, guidelines, and conventions. In case of conflict, constitution principles take precedence.

**Amendment Process**:

1. Proposed amendments must be documented with rationale
2. Version must be incremented following semantic versioning
3. Amendment history preserved in this document
4. Migration plan required for breaking changes

**Compliance Review**:

- Self-review all changes against Constitution compliance before committing to main
- Periodic constitution reviews to ensure continued relevance
- Principles may be clarified but core intent must be preserved
- Non-compliance requires explicit justification and documentation

**Versioning Policy**:

- MAJOR: Backward incompatible changes (principle removal/redefinition)
- MINOR: New principles added or material expansions
- PATCH: Clarifications, wording improvements, non-semantic fixes

**Development Guidance**:
Runtime development guidance and agent-specific instructions should be maintained in separate files (e.g., `.specify/memory/agent-guidance.md`) and must align with this constitution.

**Version**: 1.1.0 | **Ratified**: 2025-10-21 | **Last Amended**: 2025-10-21
