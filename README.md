# Synapse Skill

AI Agent skill for multi-agent orchestration with configurable models and role-based workflows.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)](https://github.com/jangyoung/synapse-skill)

## Overview

Synapse is a distributed AI agent orchestration system that coordinates three specialized services with configurable models:

| Agent | Role | Default Model | Capabilities |
|-------|------|---------------|--------------|
| **Claude** | Orchestrator/Planner | claude-sonnet-4 | Task planning, architecture design, code generation |
| **Gemini** | Analyst/Reviewer | gemini-2.5-pro | Large context analysis (>200k), code review |
| **Codex** | Executor | gpt-5.2 | Sandboxed command execution, builds, tests |

## Features

- **Multi-Model Support**: Configure different models for each service
- **Role-Based Assignment**: Assign specific models to specific tasks
- **Agentic Workflows**: Support for parallel, pipeline, and swarm patterns
- **Claude Swarms Compatible**: Similar orchestration patterns to Claude's TeammateTool

## Installation

### Prerequisites

- Docker and Docker Compose
- [Synapse](https://github.com/jangyoung/Synapse) services running

### Quick Start

```bash
# 1. Clone Synapse (the backend services)
git clone https://github.com/jangyoung/Synapse.git ~/Synapse
cd ~/Synapse
docker compose up -d

# 2. Clone synapse-skill
git clone https://github.com/jangyoung/synapse-skill.git
cd synapse-skill

# 3. Install for your AI agent
```

### For OpenCode

```bash
ln -s $(pwd) ~/.config/opencode/skills/synapse
```

### For Claude Code

```bash
mkdir -p ~/.agents/skills
ln -s $(pwd) ~/.agents/skills/synapse
```

### For Other Agents

Copy `SKILL.md` to your agent's skill directory or reference it directly.

## Configuration

### Environment Variables

```bash
# Custom Synapse location (default: ~/Synapse)
export SYNAPSE_HOME="$HOME/Synapse"

# Gateway URL (default: http://localhost:8000)
export SYNAPSE_GATEWAY_URL="http://localhost:8000"
```

### Model Configuration

Each service supports model selection via the `model` parameter:

```bash
# Claude with specific model
curl -X POST http://localhost:8000/api/v1/claude/plan \
  -d '{"task": "Build API", "model": "claude-opus-4"}'

# Gemini with specific model
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "...", "model": "gemini-3-pro-preview"}'

# Codex with specific model
curl -X POST http://localhost:8000/api/v1/codex/execute \
  -d '{"command": "pytest", "model": "gpt-5.2-mini"}'
```

### Role-Based Configuration

```json
{
  "model_config": {
    "planner": "claude-sonnet-4",
    "analyst": "gemini-2.5-pro",
    "coder": "claude-sonnet-4",
    "reviewer": "gemini-3-pro-preview",
    "executor": "gpt-5.2"
  }
}
```

## Agentic Workflow Patterns

### Parallel Specialists

Multiple reviewers work simultaneously:

```bash
# Security, performance, and simplicity reviews in parallel
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "...", "review_type": "security"}' &
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "...", "review_type": "performance"}' &
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "...", "review_type": "simplicity"}' &
wait
```

### Pipeline (Sequential)

```
Research (Gemini) → Plan (Claude) → Implement (Claude) → Test (Codex) → Review (Gemini)
```

### Swarm (Self-Organizing)

Workers claim tasks from a shared pool:

```bash
curl -X POST http://localhost:8000/api/v1/workflow \
  -d '{
    "task": "Review all files in src/",
    "mode": "swarm",
    "workers": 3
  }'
```

## API Quick Reference

| Endpoint | Purpose | Key Parameters |
|----------|---------|----------------|
| `POST /api/v1/claude/plan` | Create task plan | `task`, `constraints`, `model` |
| `POST /api/v1/claude/code` | Generate code | `description`, `language`, `model` |
| `POST /api/v1/gemini/analyze` | Analyze content | `content`, `analysis_type`, `model` |
| `POST /api/v1/gemini/review` | Review code | `code`, `language`, `review_type`, `model` |
| `POST /api/v1/codex/execute` | Execute command | `command`, `timeout`, `model` |
| `POST /api/v1/workflow` | Full pipeline | `task`, `workflow_type`, `model_config` |

## Available Models

### Claude
- `claude-sonnet-4` - Balanced (default)
- `claude-opus-4` - Maximum capability
- `claude-haiku-3.5` - Fast responses

### Gemini
- `gemini-2.5-pro` - Large context (default)
- `gemini-3-pro-preview` - Latest flagship
- `gemini-3-flash` - Speed-critical
- `gemini-2.5-flash` - Cost-efficient

### Codex
- `gpt-5.2` - Flagship (default)
- `gpt-5.2-mini` - Cost-efficient
- `gpt-5.1-thinking` - Complex reasoning

## Troubleshooting

### Services Not Running

```bash
# Check status
docker ps | grep synaps

# Start services
cd ~/Synapse && docker compose up -d

# Check health
curl http://localhost:8000/health
```

### View Logs

```bash
docker logs synaps-gateway
docker logs synaps-claude
docker logs synaps-gemini
docker logs synaps-codex
```

## Documentation

See [SKILL.md](./SKILL.md) for complete API documentation, workflow patterns, and usage examples.

## Related Projects

- [Synapse](https://github.com/jangyoung/Synapse) - The backend orchestration system
- [Oh My Claude Code](https://github.com/ohmyclaudecode/ohmyclaudecode) - Multi-agent orchestration framework
- [Claude Swarm SKILL.md](https://gist.github.com/kieranklaassen/4f2aba89594a4aea4ad64d753984b2ea) - TeammateTool orchestration patterns

## License

MIT
