# Synapse Skill

AI Agent skill for multi-agent orchestration with configurable models and role-based workflows.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-2.2.0-blue.svg)](https://github.com/akillness/synapse-skill)
[![OpenCode](https://img.shields.io/badge/OpenCode-Compatible-orange.svg)](https://opencode.ai)
[![Claude 4.5](https://img.shields.io/badge/Claude-4.5-purple.svg)](https://anthropic.com)
[![Cursor Compatible](https://img.shields.io/badge/Cursor-Compatible-green.svg)](https://cursor.sh)

## Overview

Synapse is a distributed AI agent orchestration system that coordinates three specialized services with configurable models (Updated January 2026):

| Agent | Role | Default Model | Capabilities |
|-------|------|---------------|--------------|
| **Claude** | Orchestrator/Planner | claude-sonnet-4.5 | Task planning, architecture design, code generation |
| **Gemini** | Analyst/Reviewer | gemini-3-pro-preview | Large context analysis (>1M tokens), code review |
| **Codex** | Executor | gpt-5.2 | Sandboxed command execution, builds, tests |

## Features

- **Multi-Model Support**: Configure different models for each service (Claude 4.5, Gemini 3, GPT-5.2)
- **Role-Based Assignment**: Assign specific models to specific tasks
- **Agentic Workflows**: Support for parallel, pipeline, and swarm patterns
- **Claude Swarms Compatible**: Similar orchestration patterns to Claude's TeammateTool
- **OpenCode Compatible**: `oh-my-opencode.json` agent configuration + skill integration
- **Cursor IDE Compatible**: `.cursor/rules/` integration for Cursor agents
- **Claude Code Compatible**: CLAUDE.md integration for Claude Code workflows

## Installation

### Universal Installation (Recommended)

Works with **all CLI terminals** (Claude Code, Cursor, OpenCode, etc.):

```bash
npx skills add https://github.com/akillness/synapse-skill --skill
```

This automatically installs the skill for your AI agent environment.

### Prerequisites

- Docker and Docker Compose
- [Synapse](https://github.com/akillness/Synapse) services running

### Quick Start

```bash
# 1. Clone Synapse (the backend services)
git clone https://github.com/akillness/Synapse.git ~/Synapse
cd ~/Synapse
docker compose up -d

# 2. Clone synapse-skill
git clone https://github.com/akillness/synapse-skill.git
cd synapse-skill

# 3. Install for your AI agent
```

### For OpenCode

```bash
# 1. Symlink the skill
ln -s $(pwd) ~/.config/opencode/skills/synapse

# 2. (Optional) Add agent configuration to oh-my-opencode.json
cat >> ~/.config/opencode/oh-my-opencode.json << 'EOF'
{
  "agents": {
    "synapse-planner": { "model": "google/antigravity-claude-sonnet-4-5" },
    "synapse-analyst": { "model": "google/antigravity-gemini-3-pro-high" },
    "synapse-reviewer": { "model": "google/antigravity-gemini-3-pro-high" }
  }
}
EOF
```

**OpenCode Trigger Phrases:**
- "orchestrate agents" → Synapse workflow
- "multi-agent workflow" → Parallel/pipeline execution
- "run synapse" → Full workflow

### For Claude Code

```bash
mkdir -p ~/.agents/skills
ln -s $(pwd) ~/.agents/skills/synapse
```

### For Cursor IDE

Create `.cursor/rules/synapse.mdc`:

```markdown
---
description: Synapse AI Agent Orchestration
globs: ["**/*"]
alwaysApply: true
---

When orchestrating multi-agent tasks, use Synapse endpoints:
- Plan: POST http://localhost:8000/api/v1/claude/plan
- Code: POST http://localhost:8000/api/v1/claude/code
- Review: POST http://localhost:8000/api/v1/gemini/review
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
# Claude 4.5 with specific model
curl -X POST http://localhost:8000/api/v1/claude/plan \
  -d '{"task": "Build API", "model": "claude-opus-4.5"}'

# Gemini 3 with specific model
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "...", "model": "gemini-3-pro-preview"}'

# GPT-5.2 with specific model
curl -X POST http://localhost:8000/api/v1/codex/execute \
  -d '{"command": "pytest", "model": "gpt-5.2-mini"}'
```

### Role-Based Configuration

```json
{
  "model_config": {
    "planner": "claude-sonnet-4.5",
    "analyst": "gemini-3-pro-preview",
    "coder": "claude-sonnet-4.5",
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

## Available Models (2026)

### Claude 4.5
| Model | API ID | Best For | Cost |
|-------|--------|----------|------|
| `claude-opus-4.5` | `claude-opus-4-5-20251101` | Production code, sophisticated agents | $5/$25 per M |
| `claude-sonnet-4.5` | `claude-sonnet-4-5-20250929` | Task planning, code generation (default) | $3/$15 per M |
| `claude-haiku-4.5` | `claude-haiku-4-5-20251201` | Fast responses, 90% Sonnet perf | $0.80/$4 per M |

### Gemini 3
| Model | Best For | Cost |
|-------|----------|------|
| `gemini-3-pro-preview` | Complex reasoning, 76.2% SWE-bench (default) | $2-4/M |
| `gemini-3-flash` | Sub-second latency | Lower |
| `gemini-2.5-pro` | Large context analysis | $1.25/M |
| `gemini-2.5-flash` | Cost-efficient | $0.15/M |

### GPT-5.2 (Codex)
| Model | Best For | Cost |
|-------|----------|------|
| `gpt-5.2` | Software engineering (default) | $1.25/$10 per M |
| `gpt-5.2-mini` | Cost-efficient (4x usage) | $0.25/$2 per M |
| `gpt-5.1-thinking` | Ultra-complex reasoning | Higher |

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

- [Synapse](https://github.com/akillness/Synapse) - The backend orchestration system
- [Oh My Claude Code](https://github.com/ohmyclaudecode/ohmyclaudecode) - Multi-agent orchestration framework
- [Claude Swarm SKILL.md](https://gist.github.com/kieranklaassen/4f2aba89594a4aea4ad64d753984b2ea) - TeammateTool orchestration patterns
- [KERNEL for Cursor](https://github.com/ariaxhan/kernel-cursor) - Self-evolving Cursor configuration system
- [KERNEL for Claude](https://github.com/ariaxhan/kernel-claude) - Self-evolving Claude Code configuration

## License

MIT
