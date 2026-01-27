---
name: synapse
description: "Multi-AI Agent Orchestration System with configurable models and role-based workflows. Use when you need to coordinate multiple AI agents (Claude, Gemini, Codex) for complex tasks like planning, code generation, analysis, review, and execution. Supports agentic workflow patterns: parallel specialists, pipeline, and swarm orchestration. Triggers: 'orchestrate agents', 'multi-agent workflow', 'plan and execute', 'code review pipeline', 'run synapse', 'agentic workflow'."
license: MIT
metadata:
  author: jangyoung
  version: "2.0.0"
  tags:
    - ai-orchestration
    - multi-agent
    - agentic-workflow
    - code-generation
    - code-review
    - swarm
---

# Synapse AI Agent Orchestration Skill

Synapse is a distributed AI agent system that orchestrates three specialized services with configurable models and role-based workflows.

## Agent Roles

| Agent | Role | Capabilities | Default Model |
|-------|------|--------------|---------------|
| **Claude** | Orchestrator/Planner | Task planning, architecture design, code generation | claude-sonnet-4 |
| **Gemini** | Analyst/Reviewer | Large context analysis (>200k), code review, security audits | gemini-2.5-pro |
| **Codex** | Executor | Command execution, build processes, automated testing | gpt-5.2 |

## When to Apply

Use this skill when:
- Breaking down complex tasks into actionable multi-step plans
- Generating implementation code from descriptions or specifications
- Analyzing large codebases or content (especially >200k tokens via Gemini)
- Getting comprehensive code reviews with quality scoring
- Running sandboxed shell commands for builds, tests, or automation
- Executing full plan-analyze-code-review-execute workflows
- Orchestrating parallel or pipeline-based multi-agent workflows

**Do NOT use when:**
- Simple questions that don't require code generation
- Tasks that only need local file operations
- When Docker/Synapse services are not available

## Prerequisites

**Docker must be running with Synapse services active.**

```bash
# Clone Synapse (if not already)
git clone https://github.com/jangyoung/Synapse.git ~/Synapse
cd ~/Synapse

# Start services
docker compose up -d

# Verify all services are healthy
curl -s http://localhost:8000/health
# Expected: {"status": "healthy", "service": "gateway"}
```

## Installation

### Global Installation (Recommended)

```bash
# Clone synapse-skill
git clone https://github.com/jangyoung/synapse-skill.git

# For OpenCode
ln -s $(pwd)/synapse-skill ~/.config/opencode/skills/synapse

# For Claude Code / Other Agents
mkdir -p ~/.agents/skills
ln -s $(pwd)/synapse-skill ~/.agents/skills/synapse
```

### Environment Variables (Optional)

```bash
# Custom Synapse location
export SYNAPSE_HOME="$HOME/Synapse"

# Gateway URL (default: http://localhost:8000)
export SYNAPSE_GATEWAY_URL="http://localhost:8000"
```

---

## Model Configuration

### Claude Service Models

| Model | Best For | Context | Cost |
|-------|----------|---------|------|
| `claude-sonnet-4` | Task planning, code generation, architecture | 200K | $3/$15 |
| `claude-opus-4` | Complex reasoning, detailed analysis | 200K | $15/$75 |
| `claude-haiku-3.5` | Fast responses, simple tasks | 200K | $0.25/$1.25 |

### Gemini Service Models

| Model | Best For | Context | Cost |
|-------|----------|---------|------|
| `gemini-3-pro-preview` | Complex reasoning, coding, agentic tasks | 1M | $2-4/M |
| `gemini-3-flash` | Sub-second latency, speed-critical | 1M | Lower |
| `gemini-2.5-pro` | Large context analysis, code review | 1M | $1.25/M |
| `gemini-2.5-flash` | Cost-efficient, high-volume | 1M | $0.15/M |

### Codex Service Models

| Model | Best For | Context | Cost |
|-------|----------|---------|------|
| `gpt-5.2` | Software engineering, agentic workflows | 400K | $1.25/$10 |
| `gpt-5.2-mini` | Cost-efficient coding (4x more usage) | 400K | $0.25/$2 |
| `gpt-5.1-thinking` | Ultra-complex reasoning | 400K | Higher |

---

## Agentic Workflow Patterns

### Pattern 1: Parallel Specialists

Multiple specialists review code simultaneously:

```
┌─────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                      │
│                   (You / Claude)                     │
└────────────┬───────────┬───────────┬────────────────┘
             │           │           │
     ┌───────▼───┐ ┌─────▼─────┐ ┌───▼───────┐
     │ Security  │ │Performance│ │ Simplicity│
     │ Reviewer  │ │ Reviewer  │ │ Reviewer  │
     │ (Gemini)  │ │ (Gemini)  │ │ (Gemini)  │
     └───────────┘ └───────────┘ └───────────┘
```

**Workflow:**
```bash
# 1. Spawn parallel reviews
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "<code>", "language": "python", "review_type": "security"}'

curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "<code>", "language": "python", "review_type": "performance"}'

curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d '{"code": "<code>", "language": "python", "review_type": "simplicity"}'

# 2. Aggregate results and synthesize
```

### Pattern 2: Pipeline (Sequential Dependencies)

Each stage depends on the previous:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Research │───▶│   Plan   │───▶│Implement │───▶│   Test   │───▶│  Review  │
│ (Gemini) │    │ (Claude) │    │ (Claude) │    │ (Codex)  │    │ (Gemini) │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

**Workflow:**
```bash
# Step 1: Research (Gemini analyzes requirements)
RESEARCH=$(curl -X POST http://localhost:8000/api/v1/gemini/analyze \
  -d '{"content": "<requirements>", "analysis_type": "documentation"}')

# Step 2: Plan (Claude creates implementation plan)
PLAN=$(curl -X POST http://localhost:8000/api/v1/claude/plan \
  -d "{\"task\": \"Implement based on: $RESEARCH\", \"constraints\": [\"python\", \"tests\"]}")

# Step 3: Implement (Claude generates code)
CODE=$(curl -X POST http://localhost:8000/api/v1/claude/code \
  -d "{\"description\": \"$PLAN\", \"language\": \"python\"}")

# Step 4: Test (Codex executes tests)
TEST=$(curl -X POST http://localhost:8000/api/v1/codex/execute \
  -d '{"command": "python -m pytest tests/", "timeout": 60}')

# Step 5: Review (Gemini reviews final code)
REVIEW=$(curl -X POST http://localhost:8000/api/v1/gemini/review \
  -d "{\"code\": \"$CODE\", \"language\": \"python\"}")
```

### Pattern 3: Swarm (Self-Organizing)

Workers grab available tasks from a pool:

```
                    ┌─────────────┐
                    │  Task Pool  │
                    │ ┌─┬─┬─┬─┬─┐ │
                    │ │1│2│3│4│5│ │
                    │ └─┴─┴─┴─┴─┘ │
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           │               │               │
     ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
     │  Worker 1 │   │  Worker 2 │   │  Worker 3 │
     │  (Claude) │   │  (Gemini) │   │  (Codex)  │
     │ claims #1 │   │ claims #2 │   │ claims #3 │
     └───────────┘   └───────────┘   └───────────┘
```

**Workflow:**
```bash
# Create task pool via workflow endpoint
curl -X POST http://localhost:8000/api/v1/workflow \
  -d '{
    "task": "Review all files in src/",
    "mode": "swarm",
    "workers": 3,
    "constraints": ["parallel", "auto-assign"]
  }'
```

### Pattern 4: Full Workflow (End-to-End)

Use the workflow endpoint for automatic orchestration:

```bash
curl -X POST http://localhost:8000/api/v1/workflow \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Build a user authentication module",
    "constraints": ["FastAPI", "JWT", "PostgreSQL"],
    "workflow_type": "pipeline",
    "model_config": {
      "planner": "claude-sonnet-4",
      "analyst": "gemini-2.5-pro",
      "coder": "claude-sonnet-4",
      "executor": "gpt-5.2"
    }
  }'
```

---

## Role-Based Model Assignment

### Configuration Schema

```json
{
  "roles": {
    "planner": {
      "service": "claude",
      "model": "claude-sonnet-4",
      "description": "Creates task plans and architecture designs"
    },
    "analyst": {
      "service": "gemini",
      "model": "gemini-2.5-pro",
      "description": "Analyzes content and performs code reviews"
    },
    "coder": {
      "service": "claude",
      "model": "claude-sonnet-4",
      "description": "Generates implementation code"
    },
    "reviewer": {
      "service": "gemini",
      "model": "gemini-3-pro-preview",
      "description": "Performs comprehensive code reviews"
    },
    "executor": {
      "service": "codex",
      "model": "gpt-5.2",
      "description": "Executes commands and runs tests"
    }
  }
}
```

### Role Selection by Task Type

| Task Type | Primary Role | Model | Endpoint |
|-----------|--------------|-------|----------|
| Task breakdown | Planner | claude-sonnet-4 | `/api/v1/claude/plan` |
| Code generation | Coder | claude-sonnet-4 | `/api/v1/claude/code` |
| Large context analysis | Analyst | gemini-2.5-pro | `/api/v1/gemini/analyze` |
| Security review | Reviewer | gemini-3-pro-preview | `/api/v1/gemini/review` |
| Build & test | Executor | gpt-5.2 | `/api/v1/codex/execute` |

---

## API Reference

### Base URL
```
http://localhost:8000
```

### Endpoints Summary

| Service | Method | Endpoint | Purpose |
|---------|--------|----------|---------|
| System | GET | `/health` | Gateway health check |
| System | GET | `/metrics` | Pool and load balancer stats |
| Claude | GET | `/api/v1/claude/health` | Service health |
| Claude | POST | `/api/v1/claude/plan` | Create task plan |
| Claude | POST | `/api/v1/claude/code` | Generate code |
| Gemini | GET | `/api/v1/gemini/health` | Service health |
| Gemini | POST | `/api/v1/gemini/analyze` | Analyze content |
| Gemini | POST | `/api/v1/gemini/review` | Review code |
| Codex | GET | `/api/v1/codex/health` | Service health |
| Codex | POST | `/api/v1/codex/execute` | Execute command |
| Workflow | POST | `/api/v1/workflow` | Full pipeline |

---

## Quick Reference

| Use Case | Service | Endpoint | Key Fields |
|----------|---------|----------|------------|
| Break down task | Claude | `/api/v1/claude/plan` | `task`, `constraints`, `model` |
| Generate code | Claude | `/api/v1/claude/code` | `description`, `language`, `model` |
| Analyze codebase | Gemini | `/api/v1/gemini/analyze` | `content`, `analysis_type`, `model` |
| Code review | Gemini | `/api/v1/gemini/review` | `code`, `language`, `review_type`, `model` |
| Run command | Codex | `/api/v1/codex/execute` | `command`, `timeout`, `model` |
| Full pipeline | Workflow | `/api/v1/workflow` | `task`, `constraints`, `workflow_type`, `model_config` |

---

## Running Tasks

### 1. Health Check (Always Start Here)

```bash
# Check all services
curl -s http://localhost:8000/api/v1/claude/health
curl -s http://localhost:8000/api/v1/gemini/health
curl -s http://localhost:8000/api/v1/codex/health
```

Expected response:
```json
{"status": "SERVING", "version": "1.0.0", "uptime_seconds": 123}
```

### 2. Create a Plan (Claude)

```bash
curl -X POST http://localhost:8000/api/v1/claude/plan \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Build a REST API with authentication",
    "constraints": ["use FastAPI", "JWT tokens", "PostgreSQL"],
    "model": "claude-sonnet-4"
  }'
```

### 3. Generate Code (Claude)

```bash
curl -X POST http://localhost:8000/api/v1/claude/code \
  -H "Content-Type: application/json" \
  -d '{
    "description": "A function to validate email addresses using regex",
    "language": "python",
    "model": "claude-sonnet-4"
  }'
```

### 4. Analyze Content (Gemini)

```bash
curl -X POST http://localhost:8000/api/v1/gemini/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "content": "<large_codebase_content>",
    "analysis_type": "code",
    "model": "gemini-2.5-pro"
  }'
```

### 5. Review Code (Gemini)

```bash
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -H "Content-Type: application/json" \
  -d '{
    "code": "def add(a, b):\n    return a + b",
    "language": "python",
    "review_type": "comprehensive",
    "model": "gemini-3-pro-preview"
  }'
```

### 6. Execute Command (Codex)

```bash
curl -X POST http://localhost:8000/api/v1/codex/execute \
  -H "Content-Type: application/json" \
  -d '{
    "command": "python -m pytest tests/",
    "working_dir": "/tmp",
    "timeout": 60,
    "model": "gpt-5.2"
  }'
```

**Allowed Commands:** `echo`, `ls`, `pwd`, `date`, `cat`, `head`, `tail`, `wc`, `grep`, `find`, `python`, `pip`, `npm`, `node`, `git`, `make`

---

## Common Workflows

### Code Development Workflow

```bash
# 1. Create plan
curl -X POST http://localhost:8000/api/v1/claude/plan \
  -H "Content-Type: application/json" \
  -d '{"task": "Implement user authentication", "model": "claude-sonnet-4"}'

# 2. Generate code
curl -X POST http://localhost:8000/api/v1/claude/code \
  -H "Content-Type: application/json" \
  -d '{"description": "JWT authentication middleware", "language": "python"}'

# 3. Review code
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -H "Content-Type: application/json" \
  -d '{"code": "<generated_code>", "language": "python", "model": "gemini-3-pro-preview"}'

# 4. Run tests
curl -X POST http://localhost:8000/api/v1/codex/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "python -m pytest tests/", "timeout": 60}'
```

### Code Analysis Workflow

```bash
# 1. Read file content
FILE_CONTENT=$(cat myfile.py | jq -Rs .)

# 2. Analyze with large context model
curl -X POST http://localhost:8000/api/v1/gemini/analyze \
  -H "Content-Type: application/json" \
  -d "{\"content\": $FILE_CONTENT, \"analysis_type\": \"code\", \"model\": \"gemini-2.5-pro\"}"

# 3. Review with detailed model
curl -X POST http://localhost:8000/api/v1/gemini/review \
  -H "Content-Type: application/json" \
  -d "{\"code\": $FILE_CONTENT, \"language\": \"python\", \"model\": \"gemini-3-pro-preview\"}"
```

---

## Error Handling

### Service Unavailable (503)
```json
{"detail": "Service not ready"}
```
**Solution:** Check if Docker containers are running:
```bash
docker ps | grep synaps
```

### Connection Refused
**Solution:** Start Synapse services:
```bash
cd ~/Synapse  # or $SYNAPSE_HOME
docker compose up -d
```

### Command Not Allowed (Codex)
```json
{"success": false, "stderr": "Command not allowed"}
```
**Solution:** Use only allowed commands (echo, ls, pwd, python, etc.)

---

## Troubleshooting

### Check Service Status

```bash
# All containers running?
docker ps --format "table {{.Names}}\t{{.Status}}" | grep synaps

# Gateway health
curl -s http://localhost:8000/health

# Individual service health
curl -s http://localhost:8000/api/v1/claude/health
curl -s http://localhost:8000/api/v1/gemini/health
curl -s http://localhost:8000/api/v1/codex/health
```

### Restart Services

```bash
cd ~/Synapse  # or $SYNAPSE_HOME
docker compose down
docker compose up -d
```

### View Logs

```bash
docker logs synaps-gateway
docker logs synaps-claude
docker logs synaps-gemini
docker logs synaps-codex
```

---

## Service Ports

| Service | Port | Protocol |
|---------|------|----------|
| Gateway | 8000 | HTTP/REST |
| Claude | 5011 | gRPC |
| Gemini | 5012 | gRPC |
| Codex | 5013 | gRPC |
| Prometheus | 9090 | HTTP |
| Grafana | 3000 | HTTP |

---

## Integration with AI Agents

This skill can be used by any AI coding agent that supports HTTP requests:

- **OpenCode/Claude Code**: Use bash tool with curl commands
- **Cursor**: Use terminal or HTTP client extensions
- **GitHub Copilot**: Use inline curl in terminal
- **Windsurf**: Use built-in terminal

### Example Agent Integration

```
User: "Create a plan to build a web scraper"

Agent uses Synapse skill:
1. curl POST /api/v1/claude/plan with task and model config
2. Parse response and present steps to user
3. Offer to generate code for each step
4. Execute tests via Codex
5. Review final code via Gemini
```

### Swarm Mode Integration

For complex multi-file tasks, use swarm orchestration:

```
User: "Review all authentication files for security issues"

Agent orchestrates:
1. Spawn parallel Gemini reviewers for each file
2. Aggregate findings
3. Generate summary report
4. Propose fixes via Claude
```
