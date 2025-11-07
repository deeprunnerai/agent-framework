# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Agent Framework** - a comprehensive framework for designing and building intelligent, autonomous agents for software ecosystems. It provides design methodology, architectural patterns, reusable components, and templates for creating agents that are autonomous, proactive, adaptive, and intelligent.

### Key Distinction: Agent vs Service

- **Services**: Stateless request/response, explicit invocation, fixed behavior
- **Agents**: Autonomous action, proactive behavior, adaptive learning, intelligent decisions

## Architecture & Structure

The framework is organized into five main areas:

1. **Design Guides** (`*.md` in root)
   - `FRAMEWORK_SUMMARY.md` - **📋 ONE-PAGE VISUAL SUMMARY** - Quick reference card with core principles
   - `CORE_AGENT_MECHANICS.md` - **Core agent loop (Goal → Plan → Act → Observe → Reflect), Model+Tools+Environment composition, agent empathy (debugging from agent's POV), evaluation harnesses, and prompt engineering**
   - `AGENT_DESIGN_GUIDE.md` - 6-phase design methodology from problem definition to implementation (includes tool design & verification)
   - `DECISION_TREE.md` - Step-by-step decision framework for architectural choices
   - `PATTERNS.md` - Reusable architectural patterns (Core Agent Loop, lifecycle, communication, resilience, multi-tenancy)
   - `MULTI_AGENT_PATTERNS.md` - **Multi-agent systems: when to introduce multiple agents, communication patterns, self-improving tools, and safety considerations**
   - `INTELLIGENCE_LAYERS.md` - Guide for adding intelligence capabilities (risk scoring, behavioral analysis, anomaly detection)

2. **Templates** (`templates/`)
   - `passive-observer/` - For Level 2 observer agents (monitoring, tracking, non-blocking)
   - `active-controller/` - For Level 2-4 controller agents (enforcement, blocking, policy decisions)
   - `hybrid/` - Combined patterns for agents that both observe and control

3. **Components** (`components/`)
   - `sdk/` - Client-side SDK components (event buffer, transmitter, lifecycle, instrumentation)
   - `service/` - Server-side service components (registration, auth, ingestion, multi-tenancy)
   - `intelligence/` - Intelligence layer components (risk scoring, behavioral analysis, anomaly detection)

4. **Examples** (`examples/`)
   - Real integration examples from production agents (costing-agent, auth-agent)

## Agent Taxonomy

### By Autonomy Level
- **Level 1**: Instrumented (passive observation only)
- **Level 2**: Reactive (automated collection, self-registration, buffer management)
- **Level 3**: Proactive (pattern detection, anomaly alerting, automated reporting)
- **Level 4**: Autonomous (independent decisions, behavioral learning, autonomous threat response)
- **Level 5**: Collaborative (multi-agent coordination, emergent behavior)

### By Purpose
- **Observer Agents**: Monitor/report, non-blocking, async (e.g., costing-agent, metrics-agent)
- **Controller Agents**: Enforce policies, block/allow actions, sync (e.g., auth-agent, rate-limit-agent)
- **Optimizer Agents**: Improve performance, resource allocation, auto-scaling
- **Coordinator Agents**: Workflow management, multi-service coordination

## Core Patterns

All agents follow these patterns:

1. **Hybrid Deployment**: Central service (aggregation, intelligence) + Distributed SDKs (embedded in apps)
2. **Multi-Tenancy**: Organization → Projects → Deployments/Instances → Data
3. **Resilient Communication**: Retry with exponential backoff, circuit breaker, graceful degradation, timeout handling
4. **Observable & Explainable**: Structured logging, metrics, audit trails, decision reasoning

## When Working on Agents

### Design Process
1. **Problem Definition**: What problem? Why not a service? Who are stakeholders? Success metrics?
2. **Agent Classification**: Determine autonomy level (1-5) and purpose (observer/controller/optimizer/coordinator)
3. **Architecture Selection**: Passive observer, active controller, or hybrid?
4. **Component Selection**: Choose from sdk/, service/, intelligence/ components
5. **Interface Design**: SDK API, service endpoints, data models
6. **Implementation Planning**: Break down into phases

### Core Agent Loop Pattern

Every agent operates on this fundamental cycle:

```
GOAL → PLAN → ACT → OBSERVE → REFLECT → (EXIT or LOOP BACK)
```

**Key Principles:**
- **Goal**: Define clear success criteria (tests pass, metrics threshold met)
- **Plan**: Determine next action based on current state
- **Act**: Execute tools/actions
- **Observe**: Collect feedback (results, metrics, errors)
- **Reflect**: Analyze if goal achieved, adjust strategy
- **Exit**: Goal achieved or max iterations reached

**Implementation Requirements:**
- Explicit `maxIterations` to prevent infinite loops
- Measurable `successCriteria` for goal verification
- Tool design with clear interfaces, safety constraints, and fast feedback
- Loop metrics tracking (iterations, convergence rate, cost per goal)

See `CORE_AGENT_MECHANICS.md` for detailed implementation patterns and `PATTERNS.md` for the Core Agent Loop Pattern.

### Template Selection Decision Tree
- **No decisions, just tracking** → Use `passive-observer/` template
- **Makes decisions, blocks flow** → Use `active-controller/` template
- **Both observation and control** → Use `hybrid/` template

### Performance Targets

**Passive Observer Agents**:
- Tracking overhead: < 1ms per event
- Flush latency: < 500ms
- Memory usage: < 50MB for SDK
- Throughput: 10,000 events/sec

**Active Controller Agents** (critical - in hot path):
- Validation latency: p95 < 50ms
- Decision latency: p95 < 200ms
- Cache hit rate: > 95%
- Throughput: 10,000 req/sec per instance
- Availability: 99.9%+

### Agent-Specific Operational Metrics

Beyond traditional performance metrics, track these agent behavior metrics:

**Loop Efficiency:**
- Average iterations to goal: < 5
- Convergence rate: > 95% (% of goals achieved)
- Max iterations reached count (how often agent hits limit)

**Accuracy:**
- True positive rate: > 99% (real threats caught)
- False positive rate: < 1% (false alarms)
- Precision and recall

**Cost:**
- LLM API cost per goal achieved
- Tool invocations per goal
- Total operational cost per day

**Improvement Over Time:**
- Accuracy trend (week-over-week)
- False positive trend (should decrease)
- Model updates and policy adjustments per week

### Incremental Development Path

**Don't build Level 4 on day one!** Follow this path:

1. **Week 1-2: Level 2 (Rule-Based)** → Simple if/then logic, ship quickly
2. **Week 3-4: Level 3 (Pattern Detection)** → Add behavioral analysis, A/B test
3. **Week 5-8: Level 4 (Autonomous)** → Risk scoring, autonomous actions, gradual rollout
4. **Month 3+: Optimize** → Refine models, improve loop efficiency, reduce cost

Start simple, validate, then add complexity incrementally.

### Security Considerations for Controller Agents
- Authentication required for all endpoints
- Audit logging (immutable, all decisions)
- Secrets management (never log tokens)
- **Fail-closed by default** (deny on error) - only fail-open for non-critical agents
- Rate limiting on validation endpoints
- Input validation to prevent injection attacks

## Technology Stack Recommendations

### Node.js Agents
- Runtime: Node.js 20+
- Language: TypeScript (strict mode)
- Framework: Fastify (performance) or Express (familiarity)
- Testing: Jest, Supertest

### Python Agents
- Runtime: Python 3.10+
- Framework: FastAPI (async/await support)
- Database: PostgreSQL via SQLAlchemy
- Testing: pytest, pytest-asyncio

### Common Infrastructure
- Database: PostgreSQL 15+ (JSONB, UUID, time-series with TimescaleDB)
- Cache: Redis (sessions, rate limiting, distributed locks)
- Queue: Bull/BullMQ (Node.js) or Celery (Python)
- Deployment: Docker + Kubernetes

## Design Principles

When building agents:
1. **Autonomy First** - Agent acts independently
2. **Resilience Built-In** - Handle failures gracefully
3. **Explainability Required** - Every decision must be traceable with reasoning and audit trail
4. **Privacy Conscious** - Protect sensitive data
5. **Performance Critical** - Optimize for low latency (especially controller agents)
6. **Security by Default** - Never compromise on security
7. **Observable Always** - Metrics, logs, traces for all actions

## Documentation Workflow

When working on design guides or patterns:
- The `*.md` files in root are interconnected - changes to one may require updates to others
- `README.md` is the entry point - keep it in sync with other guides
- Templates reference patterns from `PATTERNS.md` and intelligence from `INTELLIGENCE_LAYERS.md`
- Always maintain the agent taxonomy and decision trees when adding new agent types

## Common Workflows

### Adding a New Agent Type
1. Define problem and classification using `AGENT_DESIGN_GUIDE.md`
2. Choose appropriate template from `templates/`
3. Select components from `components/` library
4. Document in `README.md` under "Examples of Agents to Build"
5. Extract reusable patterns to `PATTERNS.md` if novel

### Updating Architecture Patterns
1. Update `PATTERNS.md` with new pattern
2. Update relevant template READMEs if pattern applies
3. Update `AGENT_DESIGN_GUIDE.md` if it affects design methodology
4. Add examples to demonstrate pattern usage

### Working with Intelligence Layers
Reference `INTELLIGENCE_LAYERS.md` for:
- Risk scoring engines (Level 3-4 agents)
- Behavioral analysis patterns
- Anomaly detection implementations
- Decision engine architectures
